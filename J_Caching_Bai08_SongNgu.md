# Bài 8 — Redis ngoài vai trò cache
### Bản song ngữ: tiếng Anh bám cấu trúc tiếng Việt

> **Cách dùng bài này:** đọc đoạn tiếng Việt trước → tự dịch trong đầu → so với đoạn tiếng Anh bên dưới → **đọc to bản tiếng Anh hai lần**: lần một đọc chậm cho đúng, lần hai đọc trôi như đang nói với người phỏng vấn.

---

## Phần 1 — Vì sao một hạ tầng làm được nhiều việc

**Tiếng Việt**

Redis không chỉ là một cache, và đây là điểm chúng ta nên nói ngay khi được hỏi về nó. Ba tính chất khiến nó trở thành một con dao đa năng: dữ liệu nằm trong bộ nhớ nên rất nhanh, các thao tác của nó có tính nguyên tử, và nó thường đã có sẵn trong hệ thống rồi. Nhờ ba tính chất đó, chúng ta dùng cùng một hạ tầng cho nhiều việc khác nhau. Chúng ta dùng nó làm bộ giới hạn tần suất, làm khoá phân tán, làm kho lưu phiên đăng nhập, làm bảng xếp hạng, làm kênh phát thông điệp, làm hàng đợi, và làm bộ đếm xấp xỉ. Tính chất thứ ba nghe có vẻ tầm thường nhưng nó lại quan trọng nhất trong thực tế: thêm một hệ thống mới luôn tốn công vận hành, còn tái dùng một hệ thống đã có thì gần như miễn phí. Tuy nhiên, mỗi use case đều có lưu ý riêng của nó. Lưu ý quan trọng nhất là chính sách đuổi key và chế độ lưu bền, vì chúng ta tuyệt đối không được để cache đuổi nhầm những thứ không được phép mất.

Đây cũng là cách chúng ta nên cấu trúc câu trả lời trong phỏng vấn. Chúng ta kể ra các use case trước, để cho thấy chúng ta biết bề rộng của công cụ. Sau đó chúng ta nói ngay về cạm bẫy chung, để cho thấy chúng ta cũng biết chiều sâu. Một người chỉ liệt kê được use case sẽ nghe giống người đọc tài liệu, còn một người nêu được cạm bẫy sẽ nghe giống người đã vận hành.

**English (bám cấu trúc tiếng Việt)**

Redis is not only a cache, and this is the point we should make as soon as we are asked about it. Three properties turn it into a Swiss army knife: the data sits in memory so it is very fast, its operations are atomic, and it is usually already there in the system. Thanks to those three properties, we use the same piece of infrastructure for many different jobs. We use it as a rate limiter, as a distributed lock, as a session store, as a leaderboard, as a message channel, as a queue, and as an approximate counter. The third property sounds trivial but in practice it is the most important one: adding a new system always costs operational effort, while reusing a system we already have is almost free. However, every use case has its own caveat. The most important caveat is the eviction policy and the persistence mode, because we must absolutely not let the cache evict the things that are not allowed to be lost.

This is also how we should structure our answer in an interview. We list the use cases first, in order to show that we know the breadth of the tool. Then we speak straight away about the common trap, in order to show that we know the depth as well. Someone who can only list use cases will sound like a person who has read the documentation, while someone who can name the traps will sound like a person who has operated it.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| điểm chúng ta nên nói ngay khi được hỏi | the point we should make as soon as we are asked |
| một con dao đa năng | a Swiss army knife |
| nó thường đã có sẵn trong hệ thống rồi | it is usually already there in the system |
| cùng một hạ tầng cho nhiều việc khác nhau | the same piece of infrastructure for many different jobs |
| nghe có vẻ tầm thường | sounds trivial |
| tốn công vận hành | costs operational effort |
| tái dùng một hệ thống đã có thì gần như miễn phí | reusing a system we already have is almost free |
| mỗi use case đều có lưu ý riêng của nó | every use case has its own caveat |
| không được phép mất | not allowed to be lost |
| để cho thấy chúng ta biết bề rộng của công cụ | in order to show that we know the breadth of the tool |
| sẽ nghe giống người đọc tài liệu | will sound like a person who has read the documentation |
| người nêu được cạm bẫy | someone who can name the traps |

**Thuật ngữ cần nhớ**

- con dao đa năng → **a Swiss army knife**
- hạ tầng → **infrastructure**
- lưu ý cần cẩn trọng → **a caveat**
- chính sách đuổi key → **the eviction policy**
- chế độ lưu bền → **the persistence mode**

---

## Phần 2 — Bộ giới hạn tần suất nhìn từ góc kho lưu trữ

**Tiếng Việt**

Cách đơn giản nhất để giới hạn tần suất là dùng một bộ đếm với cửa sổ cố định. Chúng ta tạo một key gắn với người dùng và với số hiệu của cửa sổ thời gian, rồi chúng ta tăng bộ đếm mỗi lần có request. Lệnh tăng là nguyên tử, vì vậy nhiều instance của app có thể cùng đếm mà không giẫm lên nhau. Ngay khi bộ đếm đạt giá trị một, chúng ta đặt thời hạn cho key để nó tự biến mất lúc cửa sổ kết thúc. Nếu chúng ta quên đặt thời hạn đó, key sẽ tích tụ vô hạn và bộ nhớ sẽ phình lên theo số người dùng. Nếu chúng ta cần cửa sổ trượt thay vì cửa sổ cố định, chúng ta dùng sorted set hoặc một script Lua để đếm theo dấu thời gian.

Chúng ta nên nói rõ rằng bài này chỉ chạm tới góc kho lưu trữ của bài toán. Việc chọn thuật toán, ví dụ gáo token hay cửa sổ trượt, và việc đặt bộ giới hạn ở cổng vào hay ở trong app, là những chủ đề riêng. Điều quan trọng ở đây là trạng thái đếm phải được chia sẻ giữa mọi instance, vì nếu mỗi instance đếm riêng thì giới hạn thật sẽ nhân lên theo số instance. Đó chính là lý do chúng ta đặt bộ đếm ở Redis chứ không đặt trong bộ nhớ của tiến trình.

**English (bám cấu trúc tiếng Việt)**

The simplest way to limit the rate is to use a counter with a fixed window. We create a key tied to the user and to the number of the time window, and then we increment the counter on every request. The increment command is atomic, therefore many instances of the app can count at the same time without stepping on each other. As soon as the counter reaches the value one, we set an expiry on the key so that it disappears by itself when the window ends. If we forget to set that expiry, the keys will pile up without limit and the memory will swell up in proportion to the number of users. If we need a sliding window instead of a fixed window, we use a sorted set or a Lua script to count by timestamp.

We should say clearly that this lesson only touches the storage corner of the problem. Choosing the algorithm, for example a token bucket or a sliding window, and placing the limiter at the gateway or inside the app, are separate topics. What matters here is that the counting state has to be shared between all instances, because if each instance counts on its own then the real limit is multiplied by the number of instances. That is exactly the reason we put the counter in Redis instead of putting it in the memory of the process.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| một bộ đếm với cửa sổ cố định | a counter with a fixed window |
| gắn với số hiệu của cửa sổ thời gian | tied to the number of the time window |
| mà không giẫm lên nhau | without stepping on each other |
| ngay khi bộ đếm đạt giá trị một | as soon as the counter reaches the value one |
| key sẽ tích tụ vô hạn | the keys will pile up without limit |
| phình lên theo số người dùng | swell up in proportion to the number of users |
| đếm theo dấu thời gian | count by timestamp |
| chỉ chạm tới góc kho lưu trữ của bài toán | only touches the storage corner of the problem |
| đặt bộ giới hạn ở cổng vào hay ở trong app | placing the limiter at the gateway or inside the app |
| trạng thái đếm phải được chia sẻ | the counting state has to be shared |
| giới hạn thật sẽ nhân lên theo số instance | the real limit is multiplied by the number of instances |

**Thuật ngữ cần nhớ**

- bộ giới hạn tần suất → **a rate limiter**
- cửa sổ cố định → **a fixed window**
- cửa sổ trượt → **a sliding window**
- gáo token → **a token bucket**
- dấu thời gian → **a timestamp**

---

## Phần 3 — Bảng xếp hạng bằng sorted set

**Tiếng Việt**

Sorted set là cấu trúc gần như sinh ra để làm bảng xếp hạng. Mỗi phần tử được gắn một điểm số, và Redis giữ chúng luôn ở trạng thái đã sắp xếp theo điểm số đó. Việc cộng điểm cho một người chơi có chi phí theo hàm lô-ga-rít của số phần tử, chứ không phải sắp xếp lại toàn bộ tập hợp. Việc lấy mười người dẫn đầu chỉ là đọc một dải liên tiếp, vì vậy nó rất nhanh dù bảng có hàng triệu người. Chúng ta cũng lấy được thứ hạng của một người cụ thể bằng một lệnh duy nhất, và đó là thứ mà cơ sở dữ liệu quan hệ làm rất tốn kém. Nếu chúng ta làm việc này trong DB, mỗi lần hiển thị bảng xếp hạng chúng ta phải sắp xếp lại hoặc phải duy trì một chỉ mục nặng.

Cái giá của cách làm này là bảng xếp hạng nằm trong bộ nhớ, vì vậy chúng ta phải nghĩ tới chuyện mất dữ liệu. Nếu điểm số là dữ liệu nghiệp vụ thật, chúng ta vẫn phải ghi nó xuống DB và coi sorted set là bản dựng sẵn để đọc. Nếu điểm số chỉ có ý nghĩa trong một mùa giải ngắn, chúng ta có thể chấp nhận giữ nó hoàn toàn trong Redis nhưng phải bật lưu bền. Đây đúng là kiểu câu hỏi tiếp theo mà người phỏng vấn hay đưa ra sau khi chúng ta khoe sorted set.

**English (bám cấu trúc tiếng Việt)**

A sorted set is almost a structure born to serve as a leaderboard. Each element is attached to a score, and Redis keeps them permanently in a state sorted by that score. Adding points for one player costs on the order of the logarithm of the number of elements, instead of re-sorting the whole set. Taking the top ten players is merely reading one contiguous range, therefore it is very fast even when the board has millions of people. We can also get the rank of one specific person with a single command, and that is something a relational database does very expensively. If we did this work in the database, every time we display the leaderboard we would have to re-sort or have to maintain a heavy index.

The price of this approach is that the leaderboard sits in memory, therefore we have to think about losing the data. If the score is genuine business data, we still have to write it down to the database and treat the sorted set as a prebuilt copy for reading. If the score only has meaning within one short season, we can accept keeping it entirely in Redis but we have to turn persistence on. This is exactly the kind of follow-up question interviewers like to raise after we have shown off the sorted set.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| gần như sinh ra để làm bảng xếp hạng | almost a structure born to serve as a leaderboard |
| giữ chúng luôn ở trạng thái đã sắp xếp | keeps them permanently in a state sorted |
| chi phí theo hàm lô-ga-rít của số phần tử | costs on the order of the logarithm of the number of elements |
| chứ không phải sắp xếp lại toàn bộ tập hợp | instead of re-sorting the whole set |
| chỉ là đọc một dải liên tiếp | is merely reading one contiguous range |
| bằng một lệnh duy nhất | with a single command |
| phải duy trì một chỉ mục nặng | have to maintain a heavy index |
| coi sorted set là bản dựng sẵn để đọc | treat the sorted set as a prebuilt copy for reading |
| chỉ có ý nghĩa trong một mùa giải ngắn | only has meaning within one short season |
| kiểu câu hỏi tiếp theo | the kind of follow-up question |
| sau khi chúng ta khoe sorted set | after we have shown off the sorted set |

**Thuật ngữ cần nhớ**

- tập hợp có sắp xếp → **a sorted set**
- điểm số → **a score**
- thứ hạng → **a rank**
- dải liên tiếp → **a contiguous range**
- chỉ mục → **an index**

---

## Phần 4 — Kho lưu phiên đăng nhập

**Tiếng Việt**

Đưa session vào Redis giải quyết một vấn đề kinh điển của việc mở rộng ngang. Khi session nằm trong bộ nhớ của từng instance, chúng ta buộc phải ghim người dùng vào một máy cố định. Việc ghim đó cản trở cân bằng tải, và nó làm mất session khi một node chết. Khi session nằm ở Redis, mọi instance đều nhìn thấy cùng một dữ liệu, vì vậy load balancer được tự do chuyển người dùng đi bất cứ đâu. Nhưng session là thứ không được phép mất trong thời gian nó còn hiệu lực, và đó là điểm khác biệt lớn so với dữ liệu cache. Vì vậy chúng ta không bao giờ để session nằm chung với cache trong một instance dùng chính sách đuổi mọi key.

Cấu hình đúng gồm ba phần và chúng ta nên nói đủ cả ba. Thứ nhất, thời gian sống của key phải khớp với thời gian hết hạn của phiên đăng nhập. Thứ hai, chính sách của instance đó phải là không đuổi key, và chúng ta nên bật lưu bền. Thứ ba, instance giữ session nên tách hẳn khỏi instance làm cache, vì hai vai trò này cần hai cấu hình trái ngược nhau.

**English (bám cấu trúc tiếng Việt)**

Moving sessions into Redis solves a classic problem of scaling horizontally. When the session sits in the memory of each instance, we are forced to pin the user to a fixed machine. That pinning gets in the way of load balancing, and it loses the session when a node dies. When the session sits in Redis, all instances see the same data, therefore the load balancer is free to send the user anywhere. But a session is something that must not be lost while it is still valid, and that is the big difference compared with cache data. Therefore we never let sessions sit together with the cache in an instance that uses a policy evicting all keys.

The correct configuration has three parts and we should say all three. First, the time to live of the key has to match the expiry time of the login session. Second, the policy of that instance has to be no eviction, and we should turn persistence on. Third, the instance holding sessions should be separated completely from the instance serving as a cache, because these two roles need two opposite configurations.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| một vấn đề kinh điển của việc mở rộng ngang | a classic problem of scaling horizontally |
| ghim người dùng vào một máy cố định | pin the user to a fixed machine |
| cản trở cân bằng tải | gets in the way of load balancing |
| được tự do chuyển người dùng đi bất cứ đâu | is free to send the user anywhere |
| trong thời gian nó còn hiệu lực | while it is still valid |
| điểm khác biệt lớn so với dữ liệu cache | the big difference compared with cache data |
| một instance dùng chính sách đuổi mọi key | an instance that uses a policy evicting all keys |
| phải khớp với thời gian hết hạn của phiên đăng nhập | has to match the expiry time of the login session |
| tách hẳn khỏi instance làm cache | separated completely from the instance serving as a cache |
| hai cấu hình trái ngược nhau | two opposite configurations |

**Thuật ngữ cần nhớ**

- kho lưu phiên đăng nhập → **a session store**
- ghim người dùng vào một máy → **a sticky session**
- mở rộng ngang → **to scale horizontally**
- còn hiệu lực → **still valid**
- không đuổi key → **no eviction**

---

## Phần 5 — Khoá phân tán: nhận diện cạm bẫy

**Tiếng Việt**

Redis hay được dùng làm khoá phân tán, và đây là chỗ chúng ta phải cẩn thận nhất. Cách làm cơ bản là đặt một key chỉ khi nó chưa tồn tại, kèm theo một thời hạn và một token ngẫu nhiên để nhận dạng người đang giữ khoá. Khi giải phóng, chúng ta phải so sánh token rồi mới xoá, và hai bước đó phải nằm trong một script Lua để bảo đảm tính nguyên tử. Nếu chúng ta xoá thẳng mà không so sánh, chúng ta có thể xoá nhầm khoá của một tiến trình khác vừa mới giành được nó. Nhưng ngay cả khi làm đúng như vậy, vẫn còn những cạm bẫy sâu hơn về thời hạn của khoá, về việc chuyển đổi khi node chính chết, và về token bảo vệ. Toàn bộ phần phân tích đó nằm ở mục về khoá phân tán, vì vậy ở đây chúng ta chỉ cần nhận diện rằng bài toán này không hề đơn giản.

Cách trả lời an toàn nhất trong phỏng vấn là thừa nhận độ khó thay vì tỏ ra tự tin. Chúng ta nói rằng chúng ta biết cách cài đặt cơ bản, và chúng ta cũng biết vì sao cách cơ bản đó chưa đủ. Chúng ta nêu ra hai rủi ro cụ thể: khoá hết hạn trong lúc công việc vẫn còn đang chạy, và hai tiến trình cùng tin rằng mình đang giữ khoá sau một lần chuyển đổi. Đây cũng là chỗ chúng ta tuyệt đối không nên để công cụ AI tự viết một cách làm riêng.

**English (bám cấu trúc tiếng Việt)**

Redis is often used as a distributed lock, and this is the place where we have to be most careful. The basic approach is to set a key only when it does not exist yet, together with an expiry and a random token to identify whoever is holding the lock. When releasing, we have to compare the token and only then delete, and those two steps have to sit inside a Lua script to guarantee atomicity. If we delete outright without comparing, we may delete by mistake the lock of another process that has just won it. But even when we do it exactly like that, there are still deeper traps around the expiry of the lock, around the failover when the primary node dies, and around the fencing token. That whole analysis belongs to the topic on distributed locks, therefore here we only need to recognise that this problem is not simple at all.

The safest way to answer in an interview is to admit the difficulty instead of acting confident. We say that we know the basic implementation, and we also know why that basic implementation is not enough. We name two concrete risks: the lock expiring while the work is still running, and two processes both believing that they hold the lock after a failover. This is also the place where we absolutely should not let an AI tool write its own approach.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| đặt một key chỉ khi nó chưa tồn tại | set a key only when it does not exist yet |
| để nhận dạng người đang giữ khoá | to identify whoever is holding the lock |
| so sánh token rồi mới xoá | compare the token and only then delete |
| nếu chúng ta xoá thẳng mà không so sánh | if we delete outright without comparing |
| của một tiến trình khác vừa mới giành được nó | of another process that has just won it |
| những cạm bẫy sâu hơn | deeper traps |
| việc chuyển đổi khi node chính chết | the failover when the primary node dies |
| bài toán này không hề đơn giản | this problem is not simple at all |
| thừa nhận độ khó thay vì tỏ ra tự tin | admit the difficulty instead of acting confident |
| trong lúc công việc vẫn còn đang chạy | while the work is still running |
| cùng tin rằng mình đang giữ khoá | both believing that they hold the lock |

**Thuật ngữ cần nhớ**

- khoá phân tán → **a distributed lock**
- giải phóng khoá → **to release the lock**
- mã nhận dạng ngẫu nhiên → **a random token**
- chuyển đổi dự phòng → **failover**
- token bảo vệ theo thứ tự → **a fencing token**

---

## Phần 6 — HyperLogLog và bitmap: đếm xấp xỉ

**Tiếng Việt**

Bài toán đếm số người dùng duy nhất là một ví dụ rất đẹp về đánh đổi. Nếu chúng ta dùng một tập hợp thường, chúng ta phải lưu mọi định danh, vì vậy bộ nhớ tăng tuyến tính theo số người dùng. Với một trang có mười triệu lượt khách mỗi ngày, cách đó tốn rất nhiều RAM chỉ để trả lời một con số. HyperLogLog ước lượng số phần tử duy nhất bằng một lượng bộ nhớ cố định và rất nhỏ, với sai số khoảng dưới một phần trăm. Chúng ta thêm phần tử vào bằng một lệnh, và chúng ta đọc con số ước lượng bằng một lệnh khác. Cái giá là chúng ta không thể liệt kê lại các phần tử, và con số thì không chính xác tuyệt đối. Bitmap thì hợp cho trường hợp định danh người dùng là số nguyên liên tiếp, ví dụ đánh dấu ai đã hoạt động trong ngày hôm nay.

Điều làm nên sức mạnh của câu trả lời này là chúng ta nói rõ khi nào không nên dùng. Nếu con số được dùng cho hoá đơn hoặc cho báo cáo tài chính, chúng ta không dùng ước lượng. Nếu chúng ta cần biết chính xác những ai đã ghé thăm, chúng ta cũng không dùng HyperLogLog, vì nó không lưu danh sách. Nhưng với các chỉ số vận hành và các bảng thống kê, sai số dưới một phần trăm là hoàn toàn chấp nhận được.

**English (bám cấu trúc tiếng Việt)**

The problem of counting unique users is a very nice example of a trade-off. If we use an ordinary set, we have to store every identifier, therefore the memory grows linearly with the number of users. For a site with ten million visits a day, that approach costs a great deal of RAM just to answer one number. HyperLogLog estimates the number of unique elements with a fixed and very small amount of memory, with an error of roughly under one percent. We add elements with one command, and we read the estimated number with another command. The price is that we cannot list the elements back, and the number is not absolutely accurate. A bitmap, on the other hand, suits the case where the user identifier is a consecutive integer, for example flagging who has been active today.

What makes this answer strong is that we state clearly when we should not use it. If the number is used for an invoice or for a financial report, we do not use an estimate. If we need to know exactly who has visited, we do not use HyperLogLog either, because it does not keep the list. But for operational metrics and statistics dashboards, an error under one percent is completely acceptable.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| một ví dụ rất đẹp về đánh đổi | a very nice example of a trade-off |
| chúng ta phải lưu mọi định danh | we have to store every identifier |
| bộ nhớ tăng tuyến tính theo số người dùng | the memory grows linearly with the number of users |
| chỉ để trả lời một con số | just to answer one number |
| một lượng bộ nhớ cố định và rất nhỏ | a fixed and very small amount of memory |
| sai số khoảng dưới một phần trăm | an error of roughly under one percent |
| chúng ta không thể liệt kê lại các phần tử | we cannot list the elements back |
| là số nguyên liên tiếp | is a consecutive integer |
| đánh dấu ai đã hoạt động trong ngày hôm nay | flagging who has been active today |
| điều làm nên sức mạnh của câu trả lời này | what makes this answer strong |
| chúng ta không dùng ước lượng | we do not use an estimate |
| hoàn toàn chấp nhận được | completely acceptable |

**Thuật ngữ cần nhớ**

- số phần tử duy nhất → **cardinality**
- ước lượng → **to estimate** / **an estimate**
- sai số → **the margin of error**
- số nguyên liên tiếp → **a consecutive integer**
- đánh dấu → **to flag**

---

## Phần 7 — Góc tech lead: người dùng bị đăng xuất một cách ngẫu nhiên

**Tiếng Việt**

Tình huống thách đố của bài này rất dễ gặp và rất khó chẩn đoán. Một đội dùng chung một Redis với chính sách đuổi mọi key cho ba việc: cache, session đăng nhập, và bộ đếm giới hạn tần suất. Người dùng thỉnh thoảng bị đăng xuất một cách ngẫu nhiên, và không ai tìm ra quy luật. Nguyên nhân là khi bộ nhớ đầy, chính sách đó được phép đuổi cả key session, vì với Redis thì mọi key đều như nhau. Người dùng mất session giữa chừng, và họ bị đăng xuất trong khi log của ứng dụng thì hoàn toàn sạch. Cách sửa đúng là tách session sang một instance riêng với chính sách không đuổi key và có bật lưu bền. Cách sửa tối thiểu là bảo đảm key session không nằm trong nhóm bị đuổi, nhưng tách instance vẫn là lựa chọn sạch sẽ hơn.

Chúng ta có thể rút cả bài này thành một câu hỏi review duy nhất. Câu hỏi đó là: trong instance này, key nào không được phép mất. Với mỗi key thuộc nhóm đó, chúng ta kiểm tra xem chính sách hiện tại có thể xoá nó hay không, và chế độ lưu bền có bảo vệ nó hay không. Công cụ AI rất hay nhét mọi thứ vào một Redis duy nhất với cấu hình của cache, vì vậy đây là chỗ chúng ta phải rà thật kỹ.

**English (bám cấu trúc tiếng Việt)**

The challenge scenario of this lesson is very easy to run into and very hard to diagnose. A team shares one Redis with a policy evicting all keys for three jobs: the cache, the login sessions, and the rate-limit counters. Users are sometimes logged out at random, and nobody can find the pattern. The cause is that when the memory is full, that policy is allowed to evict the session keys as well, because to Redis every key is the same. The user loses their session halfway through, and they are logged out while the application logs are completely clean. The correct fix is to move the sessions to a separate instance with a no-eviction policy and with persistence turned on. The minimal fix is to make sure the session keys are not in the group that can be evicted, but separating the instances is still the cleaner choice.

We can boil this whole lesson down into one single review question. That question is: in this instance, which keys are not allowed to be lost. For each key in that group, we check whether the current policy can delete it, and whether the persistence mode protects it. AI tools very often stuff everything into one single Redis with a cache configuration, therefore this is the place where we have to look really carefully.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| rất dễ gặp và rất khó chẩn đoán | very easy to run into and very hard to diagnose |
| bị đăng xuất một cách ngẫu nhiên | logged out at random |
| không ai tìm ra quy luật | nobody can find the pattern |
| với Redis thì mọi key đều như nhau | to Redis every key is the same |
| mất session giữa chừng | loses their session halfway through |
| log của ứng dụng thì hoàn toàn sạch | the application logs are completely clean |
| cách sửa tối thiểu | the minimal fix |
| vẫn là lựa chọn sạch sẽ hơn | is still the cleaner choice |
| key nào không được phép mất | which keys are not allowed to be lost |
| chế độ lưu bền có bảo vệ nó hay không | whether the persistence mode protects it |
| nhét mọi thứ vào một Redis duy nhất | stuff everything into one single Redis |
| chúng ta phải rà thật kỹ | we have to look really carefully |

**Thuật ngữ cần nhớ**

- bị đăng xuất → **to be logged out**
- chẩn đoán → **to diagnose**
- quy luật lặp lại → **a pattern**
- cách sửa tối thiểu → **the minimal fix**
- rà soát kỹ → **to look carefully**

---

## Mô hình ghi nhớ

**Tiếng Việt**

Redis là một con dao đa năng nhờ tốc độ, nhờ thao tác nguyên tử và nhờ nó đã có sẵn, vì vậy chúng ta dùng nó cho giới hạn tần suất, khoá phân tán, session, bảng xếp hạng, kênh thông điệp và bộ đếm xấp xỉ. Nhưng những thứ không được phép mất phải có chính sách riêng hoặc instance riêng, vì chúng ta không được để cache đuổi nhầm chúng.

**English (bám cấu trúc tiếng Việt)**

Redis is a Swiss army knife thanks to its speed, thanks to its atomic operations and thanks to it already being there, therefore we use it for rate limiting, distributed locks, sessions, leaderboards, message channels and approximate counters. But the things that are not allowed to be lost must have their own policy or their own instance, because we must not let the cache evict them by mistake.

---

## Bảng thuật ngữ tổng hợp

| Tiếng Việt | Tiếng Anh | Ghi chú phát âm (Anh-Anh) |
|---|---|---|
| con dao đa năng | a Swiss army knife | |
| đa năng, linh hoạt | versatile | Anh-Anh **VER**-sə-tail — đuôi /aɪl/, khác Mỹ |
| hạ tầng | infrastructure | **IN**-frə-struk-chə — trọng âm âm tiết đầu |
| lưu ý cần cẩn trọng | a caveat | **KAV**-i-at |
| chính sách đuổi key | the eviction policy | i-**VIK**-shən |
| chế độ lưu bền | the persistence mode | pə-**SIS**-təns |
| có tính nguyên tử | atomic | ə-**TOM**-ik |
| bộ giới hạn tần suất | a rate limiter | |
| cửa sổ cố định | a fixed window | |
| cửa sổ trượt | a sliding window | |
| gáo token | a token bucket | **TƏU**-kən — âm /əʊ/ dài |
| dấu thời gian | a timestamp | |
| cổng vào hệ thống | a gateway | |
| tăng lên một | to increment | **IN**-kri-mənt |
| tập hợp có sắp xếp | a sorted set | |
| bảng xếp hạng | a leaderboard | |
| điểm số | a score | |
| thứ hạng | a rank | |
| hàm lô-ga-rít | the logarithm | **LOG**-ə-ri-thəm — có âm /ð/ ở giữa |
| dải liên tiếp | a contiguous range | kən-**TIG**-yoo-əs |
| chỉ mục | an index | số nhiều thường dùng: *indexes* hoặc *indices* |
| kho lưu phiên đăng nhập | a session store | **SESH**-ən |
| ghim người dùng vào một máy | a sticky session | |
| mở rộng ngang | to scale horizontally | ho-ri-**ZON**-tə-li — trọng âm âm tiết thứ ba |
| cân bằng tải | load balancing | |
| còn hiệu lực | still valid | **VAL**-id |
| không đuổi key | no eviction | |
| khoá phân tán | a distributed lock | dis-**TRIB**-yoo-tid |
| giải phóng khoá | to release the lock | ri-**LEES** — âm cuối là /s/ |
| mã nhận dạng ngẫu nhiên | a random token | |
| chuyển đổi dự phòng | failover | **FAIL**-əʊ-və |
| token bảo vệ theo thứ tự | a fencing token | |
| số phần tử duy nhất | cardinality | kaa-di-**NAL**-ə-ti — trọng âm âm tiết thứ ba |
| duy nhất | unique | yoo-**NEEK** — trọng âm cuối |
| ước lượng | to estimate / an estimate | động từ **ES**-ti-meit, danh từ **ES**-ti-mət |
| sai số | the margin of error | **MAA**-jin |
| số nguyên | an integer | **IN**-ti-jə — chữ **g** đọc thành /dʒ/ |
| liên tiếp | consecutive | kən-**SEK**-yoo-tiv |
| đánh dấu | to flag | |
| bản đồ bit | a bitmap | |
| hoá đơn | an invoice | **IN**-vois |
| báo cáo tài chính | a financial report | fai-**NAN**-shəl — âm đầu là /faɪ/ |
| chỉ số vận hành | operational metrics | **MET**-riks |
| kênh phát thông điệp | a message channel | **CHAN**-əl |
| phát và đăng ký nhận | publish and subscribe (pub/sub) | səb-**SKRAIB** |
| hàng đợi | a queue | /kjuː/ — đọc đúng như chữ cái **Q** |
| bị đăng xuất | to be logged out | |
| chẩn đoán | to diagnose | **DAI**-əg-nəuz — động từ kết thúc bằng /z/ |
| quy luật lặp lại | a pattern | **PAT**-ən |
| cách sửa tối thiểu | the minimal fix | |
| thời gian sống | time to live (TTL) | đọc rời chữ cái: "tee-tee-el" |
| bộ nhớ đệm | cache | /kæʃ/ — đọc y hệt "cash" |

---

## Bài nói

> **Cách luyện:** bật ghi âm, nói **60–90 giây** cho mỗi đề, **không nhìn tài liệu**. Mỗi bài nói phải dùng ít nhất **sáu thuật ngữ** trong bảng trên. Nghe lại một lần, đánh dấu chỗ bạn ngập ngừng, rồi nói lại đúng đề đó thêm một lần nữa.

1. **Explain to a junior** why one Redis instance can serve so many different purposes, and what the common caveat is across all of them.

2. **Explain why a sorted set** is the right structure for a realtime leaderboard, and then raise the follow-up risk yourself before the interviewer does.

3. **A colleague wants** to keep sessions in each instance's own memory and turn on sticky sessions at the load balancer. Explain why you would push back and what you would propose instead.

4. **Describe how you would build** a rate limiter at the storage level, and explain exactly what breaks if each instance counts on its own.

5. **A teammate says** they will write a simple distributed lock with set-if-not-exists and delete this afternoon. Explain why you would push back, naming two concrete risks.

6. **When would you use HyperLogLog** instead of an ordinary set, and when would you refuse to use it? Give one example on each side.

7. **Users are being logged out at random** and everything shares one Redis with an evict-all-keys policy. Walk through your diagnosis and the fix you would ship.
