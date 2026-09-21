# Bài 4 — Tính nhất quán giữa cache và DB
### Bản song ngữ: tiếng Anh bám cấu trúc tiếng Việt

> **Cách dùng bài này:** đọc đoạn tiếng Việt trước → tự dịch trong đầu → so với đoạn tiếng Anh bên dưới → **đọc to bản tiếng Anh hai lần**: lần một đọc chậm cho đúng, lần hai đọc trôi như đang nói với người phỏng vấn.

---

## Phần 1 — Bản chất của vấn đề: chúng ta đang ghi vào hai nơi

**Tiếng Việt**

Điểm khởi đầu của bài này là một sự thật đơn giản: cache và DB là hai kho lưu trữ tách biệt. Khi chúng ta cập nhật hai kho, thao tác đó không có tính nguyên tử, vì vậy luôn tồn tại một khoảng thời gian mà một bên đã đổi còn bên kia thì chưa. Chúng ta gọi tình huống này là dual-write, nghĩa là ghi vào hai nơi. Mục tiêu thực tế của chúng ta không phải là xoá bỏ cửa sổ stale đó, vì điều đó là bất khả thi. Mục tiêu thực tế là thu nhỏ cửa sổ đó và kiểm soát được nó. Điều này nối thẳng với hai khái niệm chúng ta đã biết, đó là CAP và nhất quán cuối cùng. Nói ngắn gọn, không tồn tại một giải pháp vừa nhất quán tuyệt đối vừa miễn phí.

Trong phỏng vấn, chúng ta nên mở đầu câu trả lời bằng chính câu vừa nói. Nếu chúng ta hứa nhất quán tuyệt đối, người phỏng vấn sẽ lập tức đưa ra một kịch bản làm vỡ lời hứa đó. Nếu chúng ta thừa nhận cửa sổ stale ngay từ đầu, cuộc trò chuyện sẽ chuyển sang phần chúng ta muốn nói, đó là cách thu nhỏ và kiểm soát cửa sổ. Đó cũng chính là chỗ một tech lead nói khác với một lập trình viên mới vào nghề.

**English (bám cấu trúc tiếng Việt)**

The starting point of this lesson is a simple truth: the cache and the database are two separate stores. When we update two stores, that operation is not atomic, therefore there is always a period of time in which one side has changed while the other side has not. We call this situation a dual write, which means writing into two places. Our realistic goal is not to remove that stale window, because that is impossible. Our realistic goal is to shrink that window and to keep it under control. This connects directly to two concepts we already know, namely CAP and eventual consistency. In short, there is no solution that is both absolutely consistent and free.

In an interview, we should open our answer with exactly the sentence we have just said. If we promise absolute consistency, the interviewer will immediately bring out a scenario that breaks that promise. If we admit the stale window right from the start, the conversation will move on to the part we want to talk about, which is how to shrink and control the window. That is also the place where a tech lead speaks differently from a developer new to the trade.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| điểm khởi đầu của bài này | the starting point of this lesson |
| hai kho lưu trữ tách biệt | two separate stores |
| thao tác đó không có tính nguyên tử | that operation is not atomic |
| luôn tồn tại một khoảng thời gian mà | there is always a period of time in which |
| vì điều đó là bất khả thi | because that is impossible |
| thu nhỏ cửa sổ đó | shrink that window |
| kiểm soát được nó | keep it under control |
| nối thẳng với hai khái niệm | connects directly to two concepts |
| vừa nhất quán tuyệt đối vừa miễn phí | both absolutely consistent and free |
| đưa ra một kịch bản làm vỡ lời hứa đó | bring out a scenario that breaks that promise |
| thừa nhận cửa sổ stale ngay từ đầu | admit the stale window right from the start |
| một lập trình viên mới vào nghề | a developer new to the trade |

**Thuật ngữ cần nhớ**

- ghi vào hai nơi → **a dual write**
- có tính nguyên tử → **atomic**
- cửa sổ dữ liệu cũ → **the stale window**
- nhất quán cuối cùng → **eventual consistency**
- kho lưu trữ → **a store**

---

## Phần 2 — Xoá cache chứ không cập nhật cache

**Tiếng Việt**

Khi có một lệnh ghi, chúng ta nên xoá key trong cache chứ không nên cập nhật giá trị trong cache. Lý do thứ nhất là cập nhật cache rất dễ gặp điều kiện tranh chấp. Nếu hai tiến trình cùng ghi, thứ tự chúng ghi vào cache có thể ngược với thứ tự chúng ghi vào DB, và khi đó giá trị cũ sẽ đè lên giá trị mới. Kết quả là cache giữ một giá trị sai một cách bền vững, chứ không phải sai trong vài giây rồi tự khỏi. Lý do thứ hai là chúng ta tốn công ghi một giá trị mà có thể sẽ không ai đọc tới. Ngược lại, nếu chúng ta xoá key thì lần đọc kế tiếp sẽ nạp lại từ DB, và nó chắc chắn lấy được giá trị mới nhất. Cách này đơn giản hơn và ít chỗ sai hơn, vì vậy nó là lựa chọn mặc định.

Có một câu ngắn chúng ta nên thuộc để nói trong phỏng vấn. Xoá là một thao tác không mang thông tin, vì vậy hai lệnh xoá liên tiếp cho cùng một kết quả. Cập nhật là một thao tác mang thông tin, vì vậy thứ tự của chúng quyết định kết quả cuối cùng. Chính sự khác biệt đó làm cho xoá an toàn hơn trong một hệ thống có nhiều tiến trình chạy song song.

**English (bám cấu trúc tiếng Việt)**

When there is a write, we should delete the key in the cache instead of updating the value in the cache. The first reason is that updating the cache runs into race conditions very easily. If two processes write at the same time, the order in which they write to the cache can be the reverse of the order in which they write to the database, and then the old value will overwrite the new value. The result is that the cache holds a wrong value in a durable way, instead of being wrong for a few seconds and then healing by itself. The second reason is that we spend effort writing a value that possibly nobody will ever read. On the other hand, if we delete the key then the next read will load it again from the database, and it is guaranteed to get the newest value. This way is simpler and has fewer places to go wrong, therefore it is the default choice.

There is a short sentence we should learn by heart to say in an interview. A delete is an operation that carries no information, therefore two deletes in a row give the same result. An update is an operation that carries information, therefore their order decides the final result. It is exactly that difference that makes deleting safer in a system with many processes running in parallel.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| rất dễ gặp điều kiện tranh chấp | runs into race conditions very easily |
| thứ tự chúng ghi vào cache có thể ngược với | the order in which they write to the cache can be the reverse of |
| giá trị cũ sẽ đè lên giá trị mới | the old value will overwrite the new value |
| sai một cách bền vững | wrong in a durable way |
| sai trong vài giây rồi tự khỏi | wrong for a few seconds and then healing by itself |
| chúng ta tốn công ghi | we spend effort writing |
| nó chắc chắn lấy được giá trị mới nhất | it is guaranteed to get the newest value |
| ít chỗ sai hơn | has fewer places to go wrong |
| chúng ta nên thuộc | we should learn by heart |
| một thao tác không mang thông tin | an operation that carries no information |
| hai lệnh xoá liên tiếp | two deletes in a row |
| chạy song song | running in parallel |

**Thuật ngữ cần nhớ**

- điều kiện tranh chấp → **a race condition**
- đè lên, ghi chồng → **to overwrite**
- lặp lại vẫn cho cùng kết quả → **idempotent**
- chạy song song → **running in parallel**
- lựa chọn mặc định → **the default choice**

---

## Phần 3 — Thứ tự thao tác và hai kiểu tranh chấp

**Tiếng Việt**

Bây giờ chúng ta xét thứ tự giữa hai thao tác. Cách thứ nhất là xoá cache trước rồi mới ghi DB, và cách này có một race rất xấu. Tiến trình ghi xoá cache xong, một tiến trình đọc gặp miss và đi đọc DB, nhưng lúc đó DB vẫn còn giá trị cũ vì lệnh ghi chưa commit. Tiến trình đọc đó nạp giá trị cũ vào cache, sau đó lệnh ghi mới commit, và cache giữ giá trị cũ cho tới khi hết TTL. Cách thứ hai là ghi DB trước rồi mới xoá cache, và cách này an toàn hơn nhiều. Nó vẫn còn một race hiếm: tiến trình đọc đọc DB trước khi lệnh ghi commit, rồi nó ghi giá trị cũ vào cache sau khi lệnh xoá đã chạy xong. Xác suất của race này thấp hơn hẳn, vì tiến trình đọc phải rơi đúng vào một khe rất hẹp giữa hai thời điểm.

Kết luận của chúng ta gồm hai vế và chúng ta phải nói đủ cả hai. Vế thứ nhất là ghi DB trước rồi xoá cache sau là lựa chọn tiêu chuẩn, và đó cũng là cách AWS mô tả mẫu cache-aside. Vế thứ hai là cách đó không kín tuyệt đối, vì vậy chúng ta vẫn cần TTL làm lưới an toàn. Nếu chúng ta chỉ nói vế thứ nhất, người phỏng vấn sẽ hỏi tiếp cho tới khi chúng ta phải thừa nhận vế thứ hai.

**English (bám cấu trúc tiếng Việt)**

Now we look at the order between the two operations. The first way is to delete the cache first and only then write the database, and this way has a very bad race. The writing process finishes deleting the cache, a reading process gets a miss and goes to read the database, but at that moment the database still holds the old value because the write has not committed. That reading process loads the old value into the cache, then the write finally commits, and the cache holds the old value until the TTL runs out. The second way is to write the database first and only then delete the cache, and this way is much safer. It still has a rare race: the reading process reads the database before the write commits, and then it writes the old value into the cache after the delete has already run. The probability of this race is far lower, because the reading process has to land exactly in a very narrow slot between two moments.

Our conclusion has two halves and we must say both of them. The first half is that writing the database first and deleting the cache afterwards is the standard choice, and that is also the way AWS describes the cache-aside pattern. The second half is that this way is not completely airtight, therefore we still need a TTL as a safety net. If we only say the first half, the interviewer will keep asking until we have to admit the second half.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| bây giờ chúng ta xét thứ tự giữa hai thao tác | now we look at the order between the two operations |
| cách này có một race rất xấu | this way has a very bad race |
| vì lệnh ghi chưa commit | because the write has not committed |
| nạp giá trị cũ vào cache | loads the old value into the cache |
| cho tới khi hết TTL | until the TTL runs out |
| nó vẫn còn một race hiếm | it still has a rare race |
| sau khi lệnh xoá đã chạy xong | after the delete has already run |
| xác suất của race này thấp hơn hẳn | the probability of this race is far lower |
| rơi đúng vào một khe rất hẹp | land exactly in a very narrow slot |
| kết luận của chúng ta gồm hai vế | our conclusion has two halves |
| không kín tuyệt đối | not completely airtight |
| người phỏng vấn sẽ hỏi tiếp cho tới khi | the interviewer will keep asking until |

**Thuật ngữ cần nhớ**

- xác nhận giao dịch → **to commit**
- tiến trình đọc, tiến trình ghi → **the reading process**, **the writing process**
- xác suất → **the probability**
- kín tuyệt đối → **airtight**
- lưới an toàn → **a safety net**

---

## Phần 4 — Hai cách giảm race: xoá hai lần và ghi có phiên bản

**Tiếng Việt**

Có hai cách phổ biến để giảm cái race vừa nói. Cách thứ nhất là xoá hai lần có trễ, nghĩa là chúng ta xoá cache một lần trước khi ghi DB và một lần nữa sau khi ghi, trong đó lần thứ hai được hoãn lại một khoảng ngắn. Lần xoá thứ hai dọn giúp chúng ta giá trị cũ mà một tiến trình đọc có thể đã kịp nạp vào trong cửa sổ tranh chấp. Điểm yếu của cách này là chọn khoảng hoãn rất khó: nếu chúng ta đặt một mili giây, lần xoá thứ hai chạy trước khi tiến trình đọc kịp ghi giá trị cũ, vì vậy nó không dọn được gì cả. Nếu chúng ta đặt khoảng hoãn dài hơn thì chúng ta lại kéo dài cửa sổ stale. Vì vậy xoá hai lần chỉ làm giảm xác suất chứ không triệt tiêu vấn đề. Ngoài ra, nếu tiến trình chết giữa chừng thì lần xoá thứ hai biến mất, nên trong production chúng ta nên đẩy nó vào một hàng đợi thay vì dùng một bộ hẹn giờ nằm trong bộ nhớ.

Cách thứ hai chắc chắn hơn, đó là gắn số phiên bản vào giá trị khi ghi vào cache. Chúng ta chỉ cho phép ghi đè nếu số phiên bản mới lớn hơn hoặc bằng số phiên bản đang nằm trong cache. Nhờ vậy một tiến trình đọc chậm không thể ghi đè giá trị cũ lên giá trị mới, vì phiên bản của nó nhỏ hơn. Chúng ta phải thực hiện phép so sánh và phép ghi trong cùng một thao tác nguyên tử, ví dụ bằng một script Lua chạy trên Redis. Nếu chúng ta kiểm tra rồi mới ghi bằng hai lệnh riêng biệt, chúng ta lại tạo ra đúng cái race mà chúng ta đang muốn chặn. Với những đường đi thật sự quan trọng, cách đúng nhất vẫn là không cache giá trị quyết định và đọc thẳng từ nguồn.

**English (bám cấu trúc tiếng Việt)**

There are two common ways to reduce the race we have just described. The first way is the delayed double delete, which means we delete the cache once before writing the database and once more after writing, where the second one is postponed by a short interval. The second delete cleans up for us the old value that a reading process may have managed to load in during the race window. The weakness of this way is that choosing the interval is very hard: if we set one millisecond, the second delete runs before the reading process has time to write the old value, therefore it cleans up nothing at all. If we set a longer interval then we stretch out the stale window instead. Therefore the double delete only reduces the probability instead of eliminating the problem. On top of that, if the process dies halfway then the second delete disappears, so in production we should push it onto a queue instead of using a timer that sits in memory.

The second way is more reliable, and it is to attach a version number to the value when we write it into the cache. We only allow an overwrite if the new version number is greater than or equal to the version number currently in the cache. Thanks to that a slow reading process cannot overwrite the old value on top of the new value, because its version is smaller. We have to perform the comparison and the write inside the same atomic operation, for example with a Lua script running on Redis. If we check and only then write with two separate commands, we create exactly the race that we are trying to block. For the paths that really matter, the most correct way is still not to cache the deciding value and to read straight from the source.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| xoá hai lần có trễ | the delayed double delete |
| lần thứ hai được hoãn lại một khoảng ngắn | the second one is postponed by a short interval |
| dọn giúp chúng ta giá trị cũ | cleans up for us the old value |
| có thể đã kịp nạp vào | may have managed to load in |
| chọn khoảng hoãn rất khó | choosing the interval is very hard |
| nó không dọn được gì cả | it cleans up nothing at all |
| chúng ta lại kéo dài cửa sổ stale | we stretch out the stale window instead |
| chỉ làm giảm xác suất chứ không triệt tiêu vấn đề | only reduces the probability instead of eliminating the problem |
| nếu tiến trình chết giữa chừng | if the process dies halfway |
| một bộ hẹn giờ nằm trong bộ nhớ | a timer that sits in memory |
| gắn số phiên bản vào giá trị | attach a version number to the value |
| lớn hơn hoặc bằng | greater than or equal to |
| kiểm tra rồi mới ghi bằng hai lệnh riêng biệt | check and only then write with two separate commands |
| không cache giá trị quyết định | not to cache the deciding value |

**Thuật ngữ cần nhớ**

- xoá hai lần có trễ → **delayed double delete**
- hoãn lại → **to postpone**
- so sánh rồi mới ghi → **compare-and-set (CAS)**
- thao tác nguyên tử → **an atomic operation**
- hàng đợi công việc → **a job queue**

---

## Phần 5 — Write-through vẫn có thể lệch trong môi trường phân tán

**Tiếng Việt**

Nhiều người tin rằng cứ dùng write-through là hết chuyện, nhưng niềm tin đó không đứng vững trong môi trường phân tán. Write-through chỉ bảo đảm cache và DB khớp nhau khi mọi lệnh ghi cùng đi qua một tầng cache duy nhất. Trong thực tế chúng ta thường có nhiều cache node, và mỗi node có thể được cập nhật ở những thời điểm khác nhau. Chúng ta cũng thường đọc từ read replica của DB, mà replica thì luôn chậm hơn primary một khoảng. Và gần như hệ thống nào cũng tồn tại ít nhất một đường ghi không đi qua tầng cache. Vì vậy tính nhất quán của chúng ta phụ thuộc vào cơ chế nhân bản và phụ thuộc vào mọi đường ghi, chứ không phụ thuộc vào tên của chiến lược.

Đây là một câu hỏi bẫy rất hay gặp và chúng ta nên chuẩn bị sẵn. Nếu người phỏng vấn nói rằng write-through cho nhất quán tuyệt đối, chúng ta không nên gật đầu. Chúng ta nên hỏi lại rằng hệ thống có bao nhiêu cache node và có đọc từ replica hay không. Chính câu hỏi ngược đó cho thấy chúng ta đã vận hành hệ thống thật chứ không chỉ đọc tài liệu.

**English (bám cấu trúc tiếng Việt)**

Many people believe that using write-through settles the matter, but that belief does not hold up in a distributed environment. Write-through only guarantees that the cache and the database match when every write goes through one single cache layer. In practice we usually have several cache nodes, and each node can be updated at different moments. We also usually read from a read replica of the database, and a replica is always behind the primary by some interval. And almost every system has at least one write path that does not go through the cache layer. Therefore our consistency depends on the replication mechanism and depends on every write path, instead of depending on the name of the strategy.

This is a trap question we meet very often and we should be prepared for it. If the interviewer says that write-through gives absolute consistency, we should not nod along. We should ask back how many cache nodes the system has and whether it reads from a replica. It is exactly that question back which shows that we have operated a real system instead of only reading the documentation.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| cứ dùng write-through là hết chuyện | using write-through settles the matter |
| niềm tin đó không đứng vững | that belief does not hold up |
| khi mọi lệnh ghi cùng đi qua một tầng cache duy nhất | when every write goes through one single cache layer |
| ở những thời điểm khác nhau | at different moments |
| luôn chậm hơn primary một khoảng | is always behind the primary by some interval |
| gần như hệ thống nào cũng tồn tại | almost every system has |
| chứ không phụ thuộc vào tên của chiến lược | instead of depending on the name of the strategy |
| một câu hỏi bẫy rất hay gặp | a trap question we meet very often |
| chúng ta không nên gật đầu | we should not nod along |
| chúng ta nên hỏi lại rằng | we should ask back |
| chính câu hỏi ngược đó cho thấy | it is exactly that question back which shows |

**Thuật ngữ cần nhớ**

- bản sao chỉ để đọc → **a read replica**
- máy chính → **the primary**
- cơ chế nhân bản → **the replication mechanism**
- môi trường phân tán → **a distributed environment**
- câu hỏi bẫy → **a trap question**

---

## Phần 6 — Khi nào chấp nhận stale, khi nào đọc thẳng nguồn

**Tiếng Việt**

Câu hỏi cuối cùng luôn là chúng ta chấp nhận sai tới mức nào. Với những thứ chỉ để hiển thị phụ, ví dụ số lượt xem hoặc khối gợi ý sản phẩm, dữ liệu cũ vài giây không gây hại gì cả. Với những quyết định liên quan tới tiền, tới tồn kho hoặc tới quyền truy cập, chúng ta phải đọc thẳng từ nguồn hoặc bỏ hẳn cache trên đường đi đó. Nguyên tắc chọn rất rõ ràng: chúng ta chọn theo hậu quả của việc sai, chứ không chọn theo mức độ tiện lợi. Một ví dụ cụ thể là bước trừ tồn kho khi khách đặt hàng, và ở bước đó chúng ta không được tin vào cache. Chúng ta đọc và trừ trực tiếp trên DB hoặc trên một kho có thao tác nguyên tử, vì bán một món hàng không còn tồn tại là một thiệt hại thật.

Cách trình bày này giúp chúng ta tránh được một cái bẫy rất phổ biến. Cái bẫy đó là trả lời theo kiểu cache toàn bộ rồi tìm cách vá lỗi sau. Câu trả lời tốt hơn là chia các đường đi theo hậu quả, rồi áp chính sách khác nhau cho từng nhóm. Người phỏng vấn đánh giá cao cách chia này, vì nó cho thấy chúng ta nghĩ theo rủi ro chứ không nghĩ theo công nghệ.

**English (bám cấu trúc tiếng Việt)**

The final question is always how much wrongness we accept. For things that are only secondary display, for example view counts or the block of product recommendations, data that is a few seconds old does no harm at all. For decisions related to money, to stock levels or to access rights, we have to read straight from the source or drop the cache entirely on that path. The rule for choosing is very clear: we choose according to the consequence of being wrong, instead of choosing according to how convenient it is. A concrete example is the step of decrementing stock when a customer places an order, and at that step we must not trust the cache. We read and decrement directly on the database or on a store with atomic operations, because selling an item that no longer exists is a real loss.

This way of presenting helps us avoid a very common trap. That trap is answering in the style of caching everything and then looking for ways to patch the bugs afterwards. The better answer is to split the paths according to consequence, and then apply a different policy to each group. Interviewers value this split highly, because it shows that we think in terms of risk instead of thinking in terms of technology.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| chúng ta chấp nhận sai tới mức nào | how much wrongness we accept |
| chỉ để hiển thị phụ | only secondary display |
| không gây hại gì cả | does no harm at all |
| bỏ hẳn cache trên đường đi đó | drop the cache entirely on that path |
| chọn theo hậu quả của việc sai | choose according to the consequence of being wrong |
| bước trừ tồn kho khi khách đặt hàng | the step of decrementing stock when a customer places an order |
| chúng ta không được tin vào cache | we must not trust the cache |
| là một thiệt hại thật | is a real loss |
| cache toàn bộ rồi tìm cách vá lỗi sau | caching everything and then looking for ways to patch the bugs afterwards |
| chia các đường đi theo hậu quả | split the paths according to consequence |
| áp chính sách khác nhau cho từng nhóm | apply a different policy to each group |
| nghĩ theo rủi ro chứ không nghĩ theo công nghệ | think in terms of risk instead of thinking in terms of technology |

**Thuật ngữ cần nhớ**

- hiển thị phụ → **secondary display**
- trừ đi, giảm đi → **to decrement**
- quyền truy cập → **access rights**
- vá lỗi → **to patch**
- chính sách → **a policy**

---

## Phần 7 — Người dùng phải thấy được thay đổi của chính mình

**Tiếng Việt**

Có một dạng lỗi mà người dùng cảm nhận rất rõ, đó là họ không nhìn thấy thay đổi của chính mình. Người dùng vừa sửa hồ sơ xong, họ tải lại trang, nhưng họ lại đọc trúng bản cũ nằm trong cache. Về mặt kỹ thuật thì cửa sổ stale chỉ kéo dài vài giây, nhưng về mặt trải nghiệm thì người dùng nghĩ rằng hệ thống đã mất dữ liệu của họ. Chúng ta gọi tính chất cần bảo đảm ở đây là đọc được thứ mình vừa ghi. Cách sửa thứ nhất là xoá cache ngay sau lệnh ghi của chính người dùng đó. Cách sửa thứ hai là cho riêng người dùng đó đi vòng qua cache trong một khoảng ngắn sau khi họ ghi, ví dụ năm giây.

Chúng ta nên nói rõ rằng đây là một bảo đảm có phạm vi hẹp. Chúng ta chỉ bảo đảm cho chính người vừa ghi, chứ không bảo đảm cho tất cả người dùng khác. Nhờ phạm vi hẹp đó, chi phí của cách làm này rất thấp và chúng ta không phải hy sinh tỉ lệ hit trên toàn hệ thống. Đây là một ví dụ đẹp về việc chọn đúng mức bảo đảm cho đúng nhóm người dùng.

**English (bám cấu trúc tiếng Việt)**

There is a kind of bug that users feel very clearly, which is that they do not see their own change. The user has just finished editing their profile, they reload the page, but they read the old copy sitting in the cache. Technically the stale window only lasts a few seconds, but in terms of experience the user thinks that the system has lost their data. We call the property we need to guarantee here read-your-own-writes. The first fix is to delete the cache right after the write of that particular user. The second fix is to let that one user bypass the cache for a short interval after they write, for example five seconds.

We should say clearly that this is a guarantee with a narrow scope. We only guarantee it for the person who has just written, instead of guaranteeing it for all the other users. Thanks to that narrow scope, the cost of this approach is very low and we do not have to sacrifice the hit ratio across the whole system. This is a nice example of choosing the right level of guarantee for the right group of users.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| một dạng lỗi mà người dùng cảm nhận rất rõ | a kind of bug that users feel very clearly |
| họ không nhìn thấy thay đổi của chính mình | they do not see their own change |
| bản cũ nằm trong cache | the old copy sitting in the cache |
| về mặt kỹ thuật thì … nhưng về mặt trải nghiệm thì | technically … but in terms of experience |
| tính chất cần bảo đảm ở đây | the property we need to guarantee here |
| ngay sau lệnh ghi của chính người dùng đó | right after the write of that particular user |
| đi vòng qua cache | bypass the cache |
| một bảo đảm có phạm vi hẹp | a guarantee with a narrow scope |
| chúng ta không phải hy sinh tỉ lệ hit | we do not have to sacrifice the hit ratio |
| chọn đúng mức bảo đảm cho đúng nhóm người dùng | choosing the right level of guarantee for the right group of users |

**Thuật ngữ cần nhớ**

- đọc được thứ mình vừa ghi → **read-your-own-writes**
- đi vòng qua cache → **to bypass the cache**
- bảo đảm → **a guarantee**
- phạm vi → **scope**
- hy sinh, đánh đổi mất → **to sacrifice**

---

## Phần 8 — CDC và outbox: cách đồng bộ đáng tin cậy trong production

**Tiếng Việt**

Cách đồng bộ đáng tin cậy nhất trong production là bắt thay đổi từ chính commit log của DB. Chúng ta dùng một công cụ CDC, ví dụ Debezium, để đọc log đó và phát ra một sự kiện tương ứng với mỗi lần commit. Một consumer nghe các sự kiện đó và xoá những key cache bị ảnh hưởng. Ưu điểm lớn nhất là mọi commit đều kích hoạt việc xoá, kể cả những lệnh ghi không đi qua app của chúng ta. Nhờ đó chúng ta không còn gặp tình huống commit thành công nhưng lệnh xoá cache bị quên hoặc bị lỗi. Mẫu outbox giải quyết cùng một vấn đề theo một cách khác: chúng ta ghi sự kiện vào một bảng nằm trong cùng transaction với dữ liệu, rồi một tiến trình chuyển tiếp sẽ đọc bảng đó và phát sự kiện đi. Vì sự kiện và dữ liệu được commit cùng nhau, chúng ta không bao giờ mất sự kiện.

Khi review code cache, chúng ta nên kiểm tra ba thứ theo đúng thứ tự sau đây. Thứ nhất, đoạn code này xoá cache hay cập nhật cache, vì cập nhật là dấu hiệu của một lỗi sắp xảy ra. Thứ hai, DB được ghi trước hay cache được xoá trước, vì thứ tự ngược lại sinh ra dữ liệu cũ bền vững. Thứ ba, có đường ghi nào không đi qua đoạn code này hay không, vì nếu có thì chúng ta cần tới CDC. Công cụ AI thường sai ở hai điểm đầu tiên, vì vậy ba câu hỏi này nên trở thành phản xạ của chúng ta.

**English (bám cấu trúc tiếng Việt)**

The most reliable way to stay in sync in production is to capture the changes from the commit log of the database itself. We use a CDC tool, for example Debezium, to read that log and emit an event corresponding to each commit. A consumer listens to those events and deletes the cache keys that are affected. The biggest advantage is that every commit triggers the deletion, including the writes that do not go through our app. Thanks to that we no longer run into the situation where the commit succeeds but the cache delete is forgotten or fails. The outbox pattern solves the same problem in a different way: we write the event into a table that sits in the same transaction as the data, and then a relay process reads that table and publishes the event out. Because the event and the data are committed together, we never lose an event.

When we review caching code, we should check three things in exactly the following order. First, does this code delete the cache or update the cache, because an update is the sign of a bug about to happen. Second, is the database written first or is the cache deleted first, because the reverse order produces durable stale data. Third, is there any write path that does not go through this code, because if there is then we need CDC. AI tools usually get the first two points wrong, therefore these three questions should become our reflex.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| bắt thay đổi từ chính commit log của DB | capture the changes from the commit log of the database itself |
| phát ra một sự kiện tương ứng với mỗi lần commit | emit an event corresponding to each commit |
| những key cache bị ảnh hưởng | the cache keys that are affected |
| mọi commit đều kích hoạt việc xoá | every commit triggers the deletion |
| bị quên hoặc bị lỗi | is forgotten or fails |
| nằm trong cùng transaction với dữ liệu | sits in the same transaction as the data |
| một tiến trình chuyển tiếp | a relay process |
| chúng ta không bao giờ mất sự kiện | we never lose an event |
| theo đúng thứ tự sau đây | in exactly the following order |
| dấu hiệu của một lỗi sắp xảy ra | the sign of a bug about to happen |
| thứ tự ngược lại sinh ra dữ liệu cũ bền vững | the reverse order produces durable stale data |
| nên trở thành phản xạ của chúng ta | should become our reflex |

**Thuật ngữ cần nhớ**

- bắt thay đổi dữ liệu → **change data capture (CDC)**
- nhật ký commit → **the commit log**
- mẫu hộp thư đi → **the outbox pattern**
- tiến trình chuyển tiếp → **a relay process**
- phản xạ → **a reflex**

---

## Mô hình ghi nhớ

**Tiếng Việt**

Cache và DB là hai kho, vì vậy luôn có một cửa sổ stale mà chúng ta chỉ thu nhỏ được chứ không xoá được. Chúng ta xoá cache chứ không cập nhật, ghi DB trước rồi xoá cache sau, giữ TTL làm lưới an toàn, đọc thẳng từ nguồn cho tiền và quyền, và dùng CDC hoặc outbox khi cần đồng bộ đáng tin cậy.

**English (bám cấu trúc tiếng Việt)**

The cache and the database are two stores, therefore there is always a stale window that we can only shrink instead of remove. We delete the cache instead of updating it, we write the database first and delete the cache afterwards, we keep a TTL as a safety net, we read straight from the source for money and rights, and we use CDC or the outbox when we need reliable synchronisation.

---

## Bảng thuật ngữ tổng hợp

| Tiếng Việt | Tiếng Anh | Ghi chú phát âm (Anh-Anh) |
|---|---|---|
| ghi vào hai nơi | a dual write | dual — **DYOO**-əl, Anh-Anh có /dj/ |
| có tính nguyên tử | atomic | ə-**TOM**-ik — trọng âm âm tiết thứ hai |
| tính nguyên tử | atomicity | at-ə-**MIS**-ə-ti — trọng âm rơi vào âm tiết thứ ba |
| kho lưu trữ | a store | |
| cửa sổ dữ liệu cũ | the stale window | stale /steɪl/ — "xtêi-l" |
| nhất quán cuối cùng | eventual consistency | i-**VEN**-chu-əl kən-**SIS**-tən-si |
| bộ nhớ đệm | cache | /kæʃ/ — đọc y hệt "cash" |
| điều kiện tranh chấp | a race condition | |
| đè lên, ghi chồng | to overwrite | əʊ-və-**RAIT** — trọng âm cuối; **w** trong *write* câm |
| lặp lại vẫn cho cùng kết quả | idempotent | ai-**DEM**-pə-tənt — trọng âm âm tiết thứ hai |
| chạy song song | running in parallel | parallel — **PA**-rə-lel, trọng âm âm tiết đầu |
| xác nhận giao dịch | to commit | kə-**MIT** — trọng âm cuối |
| giao dịch | a transaction | tran-**ZAK**-shən — âm giữa là /z/ |
| xác suất | the probability | prob-ə-**BIL**-ə-ti |
| kín tuyệt đối | airtight | |
| lưới an toàn | a safety net / a TTL backstop | |
| thời gian sống | time to live (TTL) | đọc rời từng chữ cái: "tee-tee-el" |
| xoá hai lần có trễ | delayed double delete | |
| hoãn lại | to postpone | pəʊst-**PƏUN** — trọng âm cuối |
| khoảng thời gian | an interval | **IN**-tə-vəl — trọng âm âm tiết đầu |
| so sánh rồi mới ghi | compare-and-set (CAS) | |
| thao tác nguyên tử | an atomic operation | |
| hàng đợi công việc | a job queue | queue /kjuː/ — đọc đúng như chữ cái **Q** |
| bộ hẹn giờ | a timer | |
| bản sao chỉ để đọc | a read replica | **REP**-li-kə — trọng âm đầu |
| máy chính | the primary | Anh-Anh **PRAI**-mə-ri, ba âm tiết |
| cơ chế nhân bản | the replication mechanism | mechanism — **MEK**-ə-niz-əm, "ch" đọc thành /k/ |
| môi trường phân tán | a distributed environment | in-**VAI**-rən-mənt — có âm /n/ ở giữa |
| câu hỏi bẫy | a trap question | |
| hiển thị phụ | secondary display | **SEK**-ən-dri — Anh-Anh nuốt bớt âm |
| trừ đi, giảm đi | to decrement | **DEK**-ri-mənt — trọng âm âm tiết đầu |
| tồn kho | stock level / inventory | inventory — Anh-Anh **IN**-vən-tri, ba âm tiết |
| quyền truy cập | access rights | **AK**-ses — trọng âm âm tiết đầu |
| vá lỗi | to patch | /pætʃ/ — kết thúc bằng /tʃ/ |
| chính sách | a policy | |
| hồ sơ người dùng | a profile | Anh-Anh **PRƏU**-fail — trọng âm âm tiết đầu |
| đọc được thứ mình vừa ghi | read-your-own-writes | |
| đi vòng qua cache | to bypass the cache | |
| bảo đảm | a guarantee | ga-rən-**TEE** — trọng âm rơi vào âm tiết cuối |
| phạm vi | scope | /skəʊp/ — kết thúc bằng /p/ rõ |
| hy sinh, đánh đổi mất | to sacrifice | **SAK**-ri-fais — âm cuối là /s/, không phải /z/ |
| tỉ lệ hit | hit ratio | ratio — **RAY**-shi-əʊ |
| bắt thay đổi dữ liệu | change data capture (CDC) | capture — **KAP**-chə |
| nhật ký commit | the commit log | |
| phát ra sự kiện | to emit an event | i-**MIT**; event i-**VENT** |
| bên tiêu thụ sự kiện | a consumer | Anh-Anh kən-**SYOO**-mə — có /sj/ |
| mẫu hộp thư đi | the outbox pattern | |
| tiến trình chuyển tiếp | a relay process | danh từ **REE**-lay, động từ ri-**LAY** |
| kích hoạt | to trigger | **TRIG**-ə |
| phản xạ | a reflex | **REE**-fleks — kết thúc bằng cụm /ks/ |
| dữ liệu cũ bền vững | durable stale data | durable — Anh-Anh **DYOO**-rə-bl |

---

## Bài nói

> **Cách luyện:** bật ghi âm, nói **60–90 giây** cho mỗi đề, **không nhìn tài liệu**. Mỗi bài nói phải dùng ít nhất **sáu thuật ngữ** trong bảng trên. Nghe lại một lần, đánh dấu chỗ bạn ngập ngừng, rồi nói lại đúng đề đó thêm một lần nữa.

1. **Explain to a junior** why a cache and a database can never be perfectly consistent for free, and what our realistic goal is instead.

2. **A colleague's pull request** updates the cache with the new value on every write, because "it saves one database read later". Explain why you would push back.

3. **Walk an interviewer through** both orderings — delete the cache then write the database, and write the database then delete the cache. Describe the race in each one and say which you would ship.

4. **A teammate says** the delayed double delete with a one-millisecond delay closes the race completely. Explain what is wrong with that, and what you would use when you really need certainty.

5. **A colleague claims** that write-through gives perfect consistency, so it is safe to cache the account balance. Explain why you would push back, and describe what you would do for that path instead.

6. **Describe read-your-own-writes**: how caching breaks it, how the user experiences the bug, and two ways to preserve the guarantee.

7. **Explain how CDC or the outbox pattern** keeps a cache in sync, and why it is more reliable than deleting the cache inside the application code.
