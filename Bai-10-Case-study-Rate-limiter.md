# Bài 10 — Case study: Rate limiter
### Bản song ngữ: tiếng Anh bám cấu trúc tiếng Việt

> **Cách dùng:** đọc đoạn tiếng Việt trước → tự dịch trong đầu → so với bản tiếng Anh bên dưới → **đọc to bản tiếng Anh hai lần**. Lần một đọc chậm cho đúng từ, lần hai đọc liền mạch như đang nói với người phỏng vấn. Đây là bài bộc lộ rõ nhất ai thật sự hiểu hệ phân tán, vì nó buộc chúng ta nói về tranh chấp và về trạng thái dùng chung.

---

## Phần 1 — Người gác cửa và cuốn sổ chung

**Tiếng Việt**

Bộ giới hạn tốc độ giống một người gác cửa đếm xem một người đã vào bao nhiêu lần trong phút này. Nếu chỉ có một cửa duy nhất, bài toán rất dễ, vì người gác cửa nhớ hết mọi thứ trong đầu. Vấn đề bắt đầu khi chúng ta có năm mươi cửa, tức là năm mươi máy chủ cùng phục vụ. Nếu mỗi cửa đếm riêng theo sổ của mình, thì tổng số lượt vào vượt xa giới hạn mà chúng ta định đặt ra. Vì vậy, bài toán thật không phải là chọn thuật toán đếm, mà là làm sao có một cuốn sổ chung và làm sao ghi vào sổ đó mà không tranh nhau.

Hai câu hỏi đó chính là hai câu hỏi cốt lõi của hệ phân tán, chỉ khác là chúng xuất hiện dưới lớp áo của một tính năng đơn giản. Câu hỏi thứ nhất là trạng thái nằm ở đâu, và câu trả lời thường là một kho trung tâm như Redis. Câu hỏi thứ hai là làm sao nhiều bên cùng sửa một giá trị mà không mất cập nhật, và câu trả lời là dùng thao tác nguyên tử. Nếu chúng ta nhận ra hai câu hỏi này ngay từ đầu, chúng ta đã đi trước phần lớn ứng viên. Nếu chúng ta chỉ nói về công thức đếm, hậu quả là chúng ta trả lời rất trôi chảy phần dễ và bỏ trống phần được chấm điểm.

**English (bám cấu trúc tiếng Việt)**

A rate limiter is like a doorkeeper counting how many times one person has come in during this minute. If there is only one door, the problem is very easy, because the doorkeeper remembers everything in their head. The problem starts when we have fifty doors, that is, fifty servers serving at the same time. If each door counts separately in its own book, then the total number of entries goes far beyond the limit we intended to set. Therefore, the real problem is not choosing a counting algorithm, it is how to have one shared book and how to write into that book without fighting over it.

Those two questions are exactly the two core questions of distributed systems, the only difference being that they show up dressed as a simple feature. The first question is where the state lives, and the answer is usually a central store such as Redis. The second question is how several parties can modify one value without losing an update, and the answer is to use atomic operations. If we recognise these two questions from the start, we are already ahead of most candidates. If we only talk about the counting formula, the consequence is that we answer the easy part very fluently and leave the graded part empty.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| người gác cửa | a doorkeeper |
| nhớ hết mọi thứ trong đầu | remembers everything in their head |
| đếm riêng theo sổ của mình | counts separately in its own book |
| vượt xa giới hạn mà chúng ta định đặt ra | goes far beyond the limit we intended to set |
| ghi vào sổ đó mà không tranh nhau | write into that book without fighting over it |
| dưới lớp áo của một tính năng đơn giản | dressed as a simple feature |
| trạng thái nằm ở đâu | where the state lives |
| mà không mất cập nhật | without losing an update |
| chúng ta đã đi trước phần lớn ứng viên | we are already ahead of most candidates |
| bỏ trống phần được chấm điểm | leave the graded part empty |

**Thuật ngữ cần nhớ**

- bộ giới hạn tốc độ → **a rate limiter**
- trạng thái dùng chung → **shared state**
- kho trung tâm → **a central store**
- thao tác nguyên tử → **an atomic operation**
- mất cập nhật → **a lost update**

---

## Phần 2 — Bốn thuật toán và đặc tính của chúng

**Tiếng Việt**

Thuật toán phổ biến nhất là bình chứa thẻ, trong đó chúng ta hình dung một cái bình được đổ thẻ đều đặn theo thời gian. Mỗi request lấy đi một thẻ, và nếu bình rỗng thì request bị từ chối. Cách này cho phép một đợt dồn ngắn tới bằng dung tích bình, đồng thời vẫn giữ tốc độ trung bình đúng như chúng ta cấu hình. Thuật toán thứ hai là cửa sổ cố định, trong đó chúng ta đếm trong từng phút rồi đặt lại bộ đếm về không. Cách này đơn giản nhất nhưng nó có một lỗ hổng ở ranh giới cửa sổ.

Lỗ hổng đó gọi là đợt dồn ở ranh giới, và chúng ta nên mô tả nó bằng một ví dụ cụ thể. Nếu giới hạn là một trăm request mỗi phút, một người có thể gửi một trăm request vào giây cuối của phút này và một trăm request nữa vào giây đầu của phút sau. Kết quả là hai trăm request đi qua trong khoảng hai giây, tức là gấp đôi ý định của chúng ta. Thuật toán thứ ba là nhật ký cửa sổ trượt, trong đó chúng ta lưu dấu thời gian của mọi request nên độ chính xác là tuyệt đối, đổi lại bộ nhớ tốn rất nhiều. Thuật toán thứ tư là bộ đếm cửa sổ trượt, trong đó chúng ta nội suy giữa hai cửa sổ liền kề, và nó dung hoà được độ chính xác với chi phí bộ nhớ.

**English (bám cấu trúc tiếng Việt)**

The most common algorithm is the token bucket, in which we picture a bucket being filled with tokens steadily over time. Each request takes one token away, and if the bucket is empty then the request is rejected. This way allows a short burst up to the capacity of the bucket, while it still keeps the average rate exactly as we configured it. The second algorithm is the fixed window, in which we count within each minute and then reset the counter to zero. This way is the simplest one but it has a hole at the window boundary.

That hole is called the boundary burst, and we should describe it with a concrete example. If the limit is one hundred requests per minute, one person can send one hundred requests in the last second of this minute and one hundred more in the first second of the next minute. The result is that two hundred requests pass through within about two seconds, that is, double what we intended. The third algorithm is the sliding window log, in which we store the timestamp of every request so the accuracy is absolute, in exchange the memory cost is very high. The fourth algorithm is the sliding window counter, in which we interpolate between two adjacent windows, and it balances accuracy against memory cost.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| bình chứa thẻ | the token bucket |
| được đổ thẻ đều đặn theo thời gian | being filled with tokens steadily over time |
| nếu bình rỗng thì request bị từ chối | if the bucket is empty then the request is rejected |
| một đợt dồn ngắn tới bằng dung tích bình | a short burst up to the capacity of the bucket |
| đặt lại bộ đếm về không | reset the counter to zero |
| một lỗ hổng ở ranh giới cửa sổ | a hole at the window boundary |
| vào giây cuối của phút này | in the last second of this minute |
| gấp đôi ý định của chúng ta | double what we intended |
| chúng ta nội suy giữa hai cửa sổ liền kề | we interpolate between two adjacent windows |
| dung hoà được độ chính xác với chi phí bộ nhớ | balances accuracy against memory cost |

**Thuật ngữ cần nhớ**

- bình chứa thẻ → **the token bucket**
- cửa sổ cố định → **a fixed window**
- cửa sổ trượt → **a sliding window**
- đợt dồn ở ranh giới → **a boundary burst**
- dung tích → **capacity**

---

## Phần 3 — Vì sao bộ đếm cục bộ trên từng máy là sai

**Tiếng Việt**

Một cách làm hấp dẫn là để mỗi máy chủ giữ bộ đếm trong bộ nhớ của chính nó, vì như thế không tốn lượt gọi mạng nào. Cách này nhanh thật, nhưng nó phá vỡ chính điều mà chúng ta đang cố bảo đảm. Lý do là mỗi máy chỉ nhìn thấy phần lưu lượng mà bộ cân bằng tải gửi tới nó, nên nó không biết gì về phần còn lại. Với năm mươi máy, mỗi máy cho qua đúng giới hạn của nó, và tổng số lượt cho qua lớn gấp khoảng năm mươi lần ý định ban đầu. Đây là kiểu lỗi mà mã nguồn nhìn hoàn toàn đúng và kiểm thử đơn vị vẫn xanh, nhưng hệ thống thì sai ở quy mô thật.

Chúng ta có hai hướng chữa, và mỗi hướng nằm ở một chỗ khác nhau trên trục đánh đổi. Hướng thứ nhất là đặt toàn bộ trạng thái ở một kho trung tâm, nên mọi máy đọc và ghi vào cùng một chỗ và con số luôn đúng. Hướng thứ hai là giữ bộ đếm cục bộ cho nhanh, nhưng đồng bộ về kho trung tâm sau mỗi khoảng ngắn, ví dụ mỗi một trăm mili giây. Hướng thứ hai chấp nhận một sai số nhỏ trong khoảng giữa hai lần đồng bộ, đổi lại nó giảm mạnh số lượt gọi tới kho trung tâm. Điều quan trọng khi trình bày là chúng ta gọi tên đây là một bài toán về tính nhất quán, chứ không phải một bài toán về tốc độ.

**English (bám cấu trúc tiếng Việt)**

One tempting approach is to let each server keep the counter in its own memory, because that way costs no network calls at all. This approach is genuinely fast, but it breaks the very thing we are trying to guarantee. The reason is that each machine only sees the portion of traffic that the load balancer sends to it, so it knows nothing about the rest. With fifty machines, each machine lets through exactly its own limit, and the total number let through is about fifty times larger than the original intention. This is the kind of bug where the source code looks completely correct and the unit tests are still green, but the system is wrong at real scale.

We have two directions for fixing it, and each direction sits at a different place on the trade-off axis. The first direction is to put all the state in a central store, so every machine reads and writes into the same place and the number is always correct. The second direction is to keep a local counter for speed, but to synchronise it back to the central store after each short interval, for example every one hundred milliseconds. The second direction accepts a small error in the gap between two synchronisations, in exchange it sharply reduces the number of calls to the central store. The important thing when we present this is that we name it as a consistency problem, not as a speed problem.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| một cách làm hấp dẫn | one tempting approach |
| không tốn lượt gọi mạng nào | costs no network calls at all |
| nó phá vỡ chính điều mà chúng ta đang cố bảo đảm | it breaks the very thing we are trying to guarantee |
| phần lưu lượng mà bộ cân bằng tải gửi tới nó | the portion of traffic that the load balancer sends to it |
| lớn gấp khoảng năm mươi lần ý định ban đầu | about fifty times larger than the original intention |
| kiểm thử đơn vị vẫn xanh | the unit tests are still green |
| sai ở quy mô thật | wrong at real scale |
| trong khoảng giữa hai lần đồng bộ | in the gap between two synchronisations |
| giảm mạnh số lượt gọi tới kho trung tâm | sharply reduces the number of calls to the central store |
| chúng ta gọi tên đây là một bài toán về tính nhất quán | we name it as a consistency problem |

**Thuật ngữ cần nhớ**

- bộ đếm cục bộ → **a local counter**
- đồng bộ định kỳ → **periodic synchronisation**
- sai số → **an error margin**
- quy mô thật → **real scale**
- kiểm thử đơn vị → **a unit test**

---

## Phần 4 — Tranh chấp và thao tác nguyên tử

**Tiếng Việt**

Ngay cả khi đã có kho trung tâm, chúng ta vẫn có thể làm sai nếu chúng ta đọc rồi sửa rồi ghi thành ba bước rời rạc. Giả sử hai máy cùng đọc giá trị năm, cả hai cùng cộng một trong bộ nhớ, rồi cả hai cùng ghi lại giá trị sáu. Kết quả là hai request chỉ làm bộ đếm tăng một, và chúng ta vừa để mất một lần cập nhật. Đây gọi là tranh chấp, và nó xuất hiện không thường xuyên nên nó rất khó bị bắt trong kiểm thử. Ở lưu lượng cao, chuyện hiếm này lại xảy ra hàng nghìn lần mỗi phút.

Cách chữa là chúng ta biến ba bước đó thành một thao tác nguyên tử duy nhất mà kho dữ liệu bảo đảm. Với phép cộng đơn giản, chúng ta dùng lệnh tăng sẵn có của Redis, vì lệnh đó chạy trọn vẹn trên máy chủ Redis. Với logic phức tạp hơn, ví dụ bình chứa thẻ phải đọc số thẻ, tính lượng đổ thêm rồi trừ đi một, chúng ta gói toàn bộ vào một đoạn kịch bản Lua. Redis chạy đoạn kịch bản đó như một khối duy nhất, nên không có máy nào chen vào giữa được. Trong phỏng vấn, việc chúng ta chủ động nêu tranh chấp rồi nói cách xử lý là một điểm cộng lớn, còn việc lờ nó đi là một cờ đỏ rõ ràng.

**English (bám cấu trúc tiếng Việt)**

Even when we already have a central store, we can still get it wrong if we read then modify then write as three separate steps. Suppose two machines both read the value five, both add one in memory, and then both write back the value six. The result is that two requests only move the counter up by one, and we have just lost one update. This is called a race condition, and it appears infrequently so it is very hard to catch in testing. At high traffic, this rare event instead happens thousands of times per minute.

The fix is that we turn those three steps into one single atomic operation that the datastore guarantees. For simple addition, we use the built-in increment command of Redis, because that command runs entirely on the Redis server. For more complex logic, for example a token bucket that has to read the token count, compute the refill amount and then subtract one, we wrap the whole thing into a Lua script. Redis runs that script as one single block, so no machine can slip in between. In an interview, volunteering the race condition and then saying how we handle it is a big plus, while ignoring it is a clear red flag.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| đọc rồi sửa rồi ghi thành ba bước rời rạc | read then modify then write as three separate steps |
| cả hai cùng ghi lại giá trị sáu | both write back the value six |
| chúng ta vừa để mất một lần cập nhật | we have just lost one update |
| nó xuất hiện không thường xuyên | it appears infrequently |
| chuyện hiếm này lại xảy ra hàng nghìn lần mỗi phút | this rare event instead happens thousands of times per minute |
| lệnh tăng sẵn có | the built-in increment command |
| chạy trọn vẹn trên máy chủ Redis | runs entirely on the Redis server |
| tính lượng đổ thêm rồi trừ đi một | compute the refill amount and then subtract one |
| không có máy nào chen vào giữa được | no machine can slip in between |
| một cờ đỏ rõ ràng | a clear red flag |

**Thuật ngữ cần nhớ**

- tranh chấp → **a race condition**
- đọc sửa ghi → **read-modify-write**
- lệnh tăng → **the increment command**
- kịch bản Lua → **a Lua script**
- tính nguyên tử → **atomicity**

---

## Phần 5 — Khi kho trung tâm chết: mở cổng hay đóng cổng

**Tiếng Việt**

Chúng ta phải trả lời được câu hỏi điều gì xảy ra khi Redis không phản hồi, vì đó là câu hỏi mà người phỏng vấn rất hay hỏi. Có hai hành vi, và cả hai đều hợp lệ tuỳ theo bối cảnh. Hành vi thứ nhất là mở cổng khi lỗi, nghĩa là chúng ta cho mọi request đi qua vì chúng ta ưu tiên tính sẵn sàng của dịch vụ. Hành vi thứ hai là đóng cổng khi lỗi, nghĩa là chúng ta chặn hết vì chúng ta ưu tiên bảo vệ phần phía sau. Điểm mấu chốt là hành vi khi lỗi phải là một quyết định có chủ đích, chứ không phải một tai nạn do chúng ta quên xử lý ngoại lệ.

Chúng ta chọn theo hậu quả của từng hướng đối với nghiệp vụ cụ thể. Với một API công khai phục vụ hàng triệu người, mở cổng thường đúng hơn, vì việc chặn toàn bộ người dùng gây thiệt hại lớn hơn việc để lọt vài đợt lạm dụng. Với một điểm cuối nhạy cảm như đăng nhập hay thanh toán, đóng cổng thường đúng hơn, vì một cuộc tấn công dò mật khẩu không giới hạn nguy hiểm hơn việc tạm ngừng dịch vụ. Cách trả lời mạnh là chúng ta nói rằng câu trả lời là tuỳ, rồi lập tức nêu tiêu chí để chọn. Nếu chúng ta đưa ra một câu trả lời cứng nhắc cho mọi trường hợp, hậu quả là chúng ta cho thấy mình chưa từng phải cân nhắc rủi ro thật của một hệ thống đang chạy.

**English (bám cấu trúc tiếng Việt)**

We must be able to answer the question of what happens when Redis does not respond, because that is a question interviewers ask very often. There are two behaviours, and both of them are valid depending on the context. The first behaviour is to fail open, which means we let every request through because we prioritise the availability of the service. The second behaviour is to fail closed, which means we block everything because we prioritise protecting what sits behind. The key point is that the behaviour on failure has to be a deliberate decision, not an accident caused by us forgetting to handle the exception.

We choose according to the consequence of each direction for the specific business. For a public API serving millions of people, failing open is usually more correct, because blocking all users does more damage than letting a few waves of abuse slip through. For a sensitive endpoint such as login or payment, failing closed is usually more correct, because an unlimited password-guessing attack is more dangerous than pausing the service. The strong way to answer is that we say the answer is "it depends", and then immediately give the criteria for choosing. If we give one rigid answer for every case, the consequence is that we show we have never had to weigh the real risk of a running system.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| khi Redis không phản hồi | when Redis does not respond |
| mở cổng khi lỗi | to fail open |
| đóng cổng khi lỗi | to fail closed |
| ưu tiên bảo vệ phần phía sau | prioritise protecting what sits behind |
| một quyết định có chủ đích | a deliberate decision |
| do chúng ta quên xử lý ngoại lệ | caused by us forgetting to handle the exception |
| để lọt vài đợt lạm dụng | letting a few waves of abuse slip through |
| một cuộc tấn công dò mật khẩu không giới hạn | an unlimited password-guessing attack |
| rồi lập tức nêu tiêu chí để chọn | and then immediately give the criteria for choosing |
| một câu trả lời cứng nhắc cho mọi trường hợp | one rigid answer for every case |

**Thuật ngữ cần nhớ**

- mở cổng khi lỗi → **to fail open**
- đóng cổng khi lỗi → **to fail closed**
- lạm dụng → **abuse**
- điểm cuối nhạy cảm → **a sensitive endpoint**
- xử lý ngoại lệ → **exception handling**

---

## Phần 6 — Khi chính kho trung tâm trở thành điểm nghẽn

**Tiếng Việt**

Ở thông lượng rất lớn, chính Redis cũng trở thành điểm nghẽn hoặc trở thành một khoá nóng. Trường hợp điển hình là một người dùng cực lớn, ví dụ một đối tác gọi API hàng trăm nghìn lần mỗi giây, và mọi lượt gọi đều chạm vào đúng một khoá. Khoá đó nằm trên đúng một mảnh của cụm, nên mảnh đó cháy trong khi các mảnh khác nhàn rỗi. Chúng ta không giải được chuyện này bằng cách thêm máy, vì vấn đề nằm ở một khoá chứ không nằm ở tổng dung lượng. Đây đúng là chế độ hỏng mà chúng ta đã học ở bài về điểm nghẽn, chỉ khác là lần này nó xuất hiện ngay trong lớp bảo vệ của chúng ta.

Chúng ta có bốn cách giảm tải, và chúng thường được dùng kết hợp. Cách thứ nhất là giữ bộ đếm cục bộ rồi đồng bộ về Redis mỗi khoảng ngắn, nhờ đó số lượt gọi giảm đi hàng chục lần. Cách thứ hai là chia khoá thành nhiều khoá con rồi cộng lại khi cần, để tải trải ra nhiều mảnh thay vì dồn vào một chỗ. Cách thứ ba là cấp một khoá riêng cùng một hạn mức riêng cho những người dùng cực lớn, vì họ là thiểu số nhưng chiếm phần lớn lưu lượng. Cách thứ tư là đặt bộ giới hạn ở cổng API hoặc ở biên, để chặn sớm và không cho lưu lượng xấu đi sâu vào bên trong. Nếu chúng ta bỏ qua bước này, hậu quả là lớp phòng vệ của chúng ta sập trước cả thứ mà nó đang bảo vệ.

**English (bám cấu trúc tiếng Việt)**

At very high throughput, Redis itself also becomes a bottleneck or becomes a hot key. The typical case is one extremely large user, for example a partner calling the API hundreds of thousands of times per second, and every call touches exactly one key. That key sits on exactly one shard of the cluster, so that shard burns while the other shards sit idle. We cannot solve this by adding machines, because the problem lies in one key rather than in the total capacity. This is exactly the failure mode we learned in the lesson on bottlenecks, the only difference being that this time it appears right inside our own protection layer.

We have four ways to reduce the load, and they are usually used in combination. The first way is to keep a local counter and then synchronise to Redis every short interval, thanks to which the number of calls drops by tens of times. The second way is to split the key into several sub-keys and add them up when needed, so that the load spreads across many shards instead of piling into one place. The third way is to give a dedicated key together with a dedicated quota to the extremely large users, because they are a minority but they account for most of the traffic. The fourth way is to put the limiter at the API gateway or at the edge, in order to block early and stop bad traffic from travelling deep inside. If we skip this step, the consequence is that our defence layer collapses before the thing it is defending does.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| trở thành một khoá nóng | becomes a hot key |
| một người dùng cực lớn | one extremely large user |
| mảnh đó cháy trong khi các mảnh khác nhàn rỗi | that shard burns while the other shards sit idle |
| vấn đề nằm ở một khoá chứ không nằm ở tổng dung lượng | the problem lies in one key rather than in the total capacity |
| ngay trong lớp bảo vệ của chúng ta | right inside our own protection layer |
| chia khoá thành nhiều khoá con rồi cộng lại | split the key into several sub-keys and add them up |
| cấp một khoá riêng cùng một hạn mức riêng | give a dedicated key together with a dedicated quota |
| họ là thiểu số nhưng chiếm phần lớn lưu lượng | they are a minority but they account for most of the traffic |
| chặn sớm | block early |
| sập trước cả thứ mà nó đang bảo vệ | collapses before the thing it is defending does |

**Thuật ngữ cần nhớ**

- khoá nóng → **a hot key**
- hạn mức riêng → **a dedicated quota**
- cổng API → **the API gateway**
- chia khoá → **key splitting**
- lớp phòng vệ → **a defence layer**

---

## Phần 7 — Nhiều tầng giới hạn và cách trả về cho khách gọi

**Tiếng Việt**

Trong thực tế, chúng ta hiếm khi chỉ đặt một mức giới hạn duy nhất. Chúng ta thường đặt giới hạn theo địa chỉ IP để chặn các nguồn lạ, theo người dùng để bảo đảm chia sẻ công bằng, theo từng điểm cuối để bảo vệ những đường đắt tiền, và một giới hạn toàn cục để bảo vệ toàn hệ thống. Một request phải qua được tất cả các tầng thì mới được phục vụ, và tầng nào chặt nhất sẽ là tầng quyết định. Chúng ta cũng có thể siết tự động khi phần phía sau đang quá tải, và cách làm đó gọi là giới hạn thích ứng. Nhờ vậy, hệ thống tự bảo vệ mình mà không cần ai đó thức dậy lúc hai giờ sáng.

Cách chúng ta trả lời cho khách gọi cũng quan trọng, vì nó quyết định họ cư xử thế nào sau đó. Chúng ta trả về mã bốn trăm hai mươi chín, và chúng ta kèm theo một header cho biết còn bao nhiêu lượt và khi nào hạn mức được đặt lại. Chúng ta nên kèm cả header nói rõ nên chờ bao lâu trước khi thử lại, để khách gọi không quay lại ngay lập tức. Nếu chúng ta chỉ trả lỗi trống không, hậu quả là các máy khách sẽ thử lại liên tục và biến một đợt quá tải nhẹ thành một cơn bão thử lại. Nói cách khác, một bộ giới hạn tốt không chỉ chặn, nó còn dạy cho phía bên kia biết cách cư xử.

**English (bám cấu trúc tiếng Việt)**

In practice, we rarely set only one single limit. We usually set a limit by IP address to block unknown sources, by user to ensure fair sharing, by individual endpoint to protect the expensive paths, and one global limit to protect the whole system. A request has to pass all the layers before it is served, and whichever layer is the strictest will be the deciding one. We can also tighten automatically when the part behind is overloaded, and that approach is called adaptive rate limiting. Thanks to that, the system protects itself without needing somebody to wake up at two in the morning.

The way we answer the caller also matters, because it decides how they behave afterwards. We return the four hundred and twenty-nine code, and we attach a header telling them how many calls are left and when the quota will be reset. We should also attach a header saying clearly how long they should wait before retrying, so that the caller does not come back immediately. If we only return a bare error, the consequence is that the clients will retry continuously and turn a mild overload into a retry storm. In other words, a good rate limiter does not only block, it also teaches the other side how to behave.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| để bảo đảm chia sẻ công bằng | to ensure fair sharing |
| bảo vệ những đường đắt tiền | protect the expensive paths |
| tầng nào chặt nhất sẽ là tầng quyết định | whichever layer is the strictest will be the deciding one |
| siết tự động khi phần phía sau đang quá tải | tighten automatically when the part behind is overloaded |
| mà không cần ai đó thức dậy lúc hai giờ sáng | without needing somebody to wake up at two in the morning |
| nó quyết định họ cư xử thế nào sau đó | it decides how they behave afterwards |
| khi nào hạn mức được đặt lại | when the quota will be reset |
| nên chờ bao lâu trước khi thử lại | how long they should wait before retrying |
| chỉ trả lỗi trống không | only return a bare error |
| một cơn bão thử lại | a retry storm |

**Thuật ngữ cần nhớ**

- giới hạn thích ứng → **adaptive rate limiting**
- chia sẻ công bằng → **fair sharing** / **fair use**
- cơn bão thử lại → **a retry storm**
- lùi dần theo cấp số nhân → **exponential backoff**
- hạn mức → **a quota**

---

## Mô hình ghi nhớ

**Tiếng Việt**

Chúng ta cần một cuốn sổ chung và một cách ghi sổ không tranh nhau. Vì vậy chúng ta đặt trạng thái ở kho trung tâm, chúng ta cập nhật bằng thao tác nguyên tử, chúng ta chọn hành vi khi lỗi một cách có chủ đích, và khi chính kho đó nóng lên thì chúng ta đếm cục bộ rồi đồng bộ định kỳ.

**English (bám cấu trúc tiếng Việt)**

We need one shared book and one way of writing into it without fighting. Therefore we put the state in a central store, we update with atomic operations, we choose the failure behaviour deliberately, and when that store itself gets hot we count locally and then synchronise periodically.

---

## Bảng thuật ngữ tổng hợp

| Tiếng Việt | Tiếng Anh | Ghi chú phát âm |
|---|---|---|
| bộ giới hạn tốc độ | a rate limiter | |
| người gác cửa | a doorkeeper | |
| trạng thái dùng chung | shared state | |
| kho trung tâm | a central store | |
| thao tác nguyên tử | an atomic operation | *atomic* — ơ-**TO**-mịc, trọng âm âm thứ hai |
| tính nguyên tử | atomicity | a-tơ-**MI**-sơ-ti, trọng âm âm thứ ba |
| mất cập nhật | a lost update | |
| bình chứa thẻ | the token bucket | |
| cửa sổ cố định | a fixed window | |
| cửa sổ trượt | a sliding window | |
| nhật ký cửa sổ trượt | a sliding window log | |
| đợt dồn ở ranh giới | a boundary burst | *burst* — âm cuối **-st** phải bật ra |
| dung tích | capacity | cơ-**PA**-sơ-ti, trọng âm âm thứ hai |
| đợt dồn | a burst | |
| nội suy | to interpolate | in-**TƠ**-pơ-lêit, trọng âm âm thứ hai |
| bộ đếm cục bộ | a local counter | |
| đồng bộ định kỳ | periodic synchronisation | *synchronisation* — sing-crơ-nai-**ZÂY**-shợn |
| sai số | an error margin | |
| quy mô thật | real scale | |
| kiểm thử đơn vị | a unit test | |
| tranh chấp | a race condition | |
| đọc sửa ghi | read-modify-write | |
| lệnh tăng | the increment command | *increment* — "**IN**-cri-mợnt", trọng âm đầu |
| kịch bản Lua | a Lua script | *script* — cụm **scr-** đầu và **-pt** cuối đều phải bật |
| mở cổng khi lỗi | to fail open | |
| đóng cổng khi lỗi | to fail closed | |
| lạm dụng | abuse | danh từ /əˈbjuːs/ "ơ-**BIUS**"; động từ đuôi đọc /z/ |
| điểm cuối nhạy cảm | a sensitive endpoint | |
| xử lý ngoại lệ | exception handling | |
| khoá nóng | a hot key | |
| mảnh dữ liệu | a shard | /ʃɑːd/ — âm đầu là "sh" không phải "s" |
| hạn mức | a quota | Anh /ˈkwəʊtə/ — "**KUÂU**-tơ", trọng âm đầu |
| hạn mức riêng | a dedicated quota | |
| cổng API | the API gateway | |
| chia khoá | key splitting | |
| lớp phòng vệ | a defence layer | Anh viết *defence*, đọc đi-**FENS**, trọng âm âm thứ hai |
| giới hạn thích ứng | adaptive rate limiting | *adaptive* — ơ-**DAP**-tịv, trọng âm âm thứ hai |
| chia sẻ công bằng | fair sharing / fair use | |
| cơn bão thử lại | a retry storm | |
| lùi dần theo cấp số nhân | exponential backoff | *exponential* — ek-spơ-**NEN**-shợl, trọng âm âm thứ ba |
| thông lượng | throughput | âm **th** /θ/ đầu, "THRU-put" |
| ngưỡng | a threshold | âm **th** /θ/ đầu, "THRESH-hâuld" |
| dấu thời gian | a timestamp | âm cuối **-mp** phải bật ra |

---

## Bài nói

> **Cách luyện:** bật ghi âm, nói **60–90 giây** cho mỗi đề, **không nhìn tài liệu** khi nói. Mỗi bài phải dùng ít nhất **sáu thuật ngữ** trong bảng trên. Bài này có nhiều đề phản biện, vì phần lớn giá trị của bạn ở vòng phỏng vấn nằm ở chỗ chỉ ra được cái sai một cách lịch sự và có căn cứ.

1. **Explain to a junior engineer** why a rate limiter running on fifty servers is really a distributed state problem, not a counting problem.

2. **A colleague says:** *"Let's keep the counter in each server's memory, it saves a Redis round trip on every request."* **Explain what is wrong with that**, and quantify how far off the limit the system would be.

3. **A colleague says:** *"We read the counter, add one, and write it back — that's fine because the window is a whole minute."* **Explain what is wrong with that**, and describe the two ways you would make it safe.

4. **Describe what happens when** a client sends one hundred requests in the last second of a minute and one hundred more in the first second of the next minute, under a fixed-window limiter.

5. **When would you choose** to fail open, and when would you choose to fail closed, if Redis becomes unreachable? Give one endpoint for each and state the risk you are accepting.

6. **Describe what happens when** a single partner account sends four hundred thousand requests per second through a limiter keyed by account id, and explain how you would spread that load.

7. **Explain to a junior engineer** why the response to a rate-limited request should include headers, and what goes wrong with clients if it does not.
