# Bài 2 — Optimistic vs Pessimistic Locking
### Bản song ngữ: tiếng Anh bám cấu trúc tiếng Việt

> **Cách dùng bài này:** đọc đoạn tiếng Việt trước → tự dịch trong đầu, không nhìn xuống → so với đoạn tiếng Anh bên dưới → **đọc to bản tiếng Anh hai lần**: lần một đọc chậm cho đúng từng âm, lần hai đọc trôi như đang nói với người phỏng vấn.

---

## ① Hai triết lý đối lập

**Tiếng Việt**

Optimistic locking và pessimistic locking là hai triết lý đối lập nhau về cùng một câu hỏi: chúng ta có tin rằng conflict hay xảy ra hay không. Pessimistic nghĩa là bi quan, tức là chúng ta giả định conflict xảy ra thường xuyên, cho nên chúng ta khóa dữ liệu trước, và sau đó chúng ta mới làm việc, còn ai đến sau thì phải chờ. Cách này giống như mượn chìa khóa phòng họp: chúng ta giữ chìa trong tay, và những người khác đứng ngoài cửa cho đến khi chúng ta trả chìa. Optimistic nghĩa là lạc quan, tức là chúng ta giả định conflict rất hiếm, cho nên chúng ta để mọi người chạy song song và chỉ kiểm tra vào lúc commit xem có ai sửa chen vào giữa hay không. Nếu có người sửa chen vào, giao dịch của chúng ta thất bại và chúng ta phải làm lại từ đầu. Cách này giống như sửa một tài liệu Google Doc rồi đến lúc lưu mới nhận được thông báo rằng người khác vừa sửa và chúng ta cần tải lại. Điểm cần nhớ là cả hai cách đều đúng, và việc chọn sai không làm hỏng dữ liệu mà làm hỏng hiệu năng.

**English (bám cấu trúc tiếng Việt)**

Optimistic locking and pessimistic locking are two opposite philosophies about the same question: do we believe that conflicts happen often or not. Pessimistic means being pessimistic, that is, we assume that conflicts happen often, therefore we lock the data first, and after that we do the work, while whoever comes later has to wait. This way is like borrowing the key of a meeting room: we hold the key in our hand, and other people stand outside the door until we give the key back. Optimistic means being optimistic, that is, we assume that conflicts are very rare, therefore we let everybody run in parallel and we only check at commit time whether somebody has edited in the middle. If somebody has edited in the middle, our transaction fails and we have to do the work again from the start. This way is like editing a Google Doc and then, at the moment of saving, receiving a message that another person has just edited it and that we need to reload. The point to remember is that both ways are correct, and choosing the wrong one does not corrupt the data but ruins the performance.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| hai triết lý đối lập nhau về cùng một câu hỏi | two opposite philosophies about the same question |
| chúng ta giả định conflict xảy ra thường xuyên | we assume that conflicts happen often |
| và sau đó chúng ta mới làm việc | and after that we do the work |
| ai đến sau thì phải chờ | whoever comes later has to wait |
| mượn chìa khóa phòng họp | borrowing the key of a meeting room |
| cho đến khi chúng ta trả chìa | until we give the key back |
| chỉ kiểm tra vào lúc commit | only check at commit time |
| có ai sửa chen vào giữa hay không | whether somebody has edited in the middle |
| làm lại từ đầu | do the work again from the start |
| không làm hỏng dữ liệu mà làm hỏng hiệu năng | does not corrupt the data but ruins the performance |

**Thuật ngữ cần nhớ**

- khóa bi quan → **a pessimistic lock**
- khóa lạc quan → **an optimistic lock**
- xung đột ghi → **a write conflict**
- lúc commit → **at commit time**
- chạy song song → **to run in parallel**
- hiệu năng → **performance**

---

## ② Optimistic được hiện thực bằng cột version

**Tiếng Việt**

Optimistic locking được hiện thực bằng một cột `version` trên mỗi hàng, hoặc bằng một cột timestamp đóng vai trò tương tự. Khi chúng ta đọc hàng đó ra, chúng ta giữ lại giá trị version cũ trong tay. Khi chúng ta ghi xuống, chúng ta đưa version cũ vào mệnh đề `WHERE` và đồng thời tăng version lên một, tức là `UPDATE t SET ..., version = version + 1 WHERE id = $1 AND version = $2`. Nếu số hàng bị ảnh hưởng bằng một, không ai chen vào giữa và bản ghi của chúng ta đã được lưu. Nếu số hàng bị ảnh hưởng bằng không, có người đã tăng version trước chúng ta, và đó chính là tín hiệu conflict. Đây thực chất là phép compare-and-swap ở mức một hàng dữ liệu, chỉ khác là phép so sánh được database thực hiện thay cho chúng ta. Nếu chúng ta quên kiểm tra số hàng bị ảnh hưởng, chúng ta sẽ tưởng rằng mọi thứ đã thành công trong khi thay đổi của chúng ta bị rơi mất hoàn toàn.

Ở đây có một cái bẫy mà nhiều đội mắc phải khi dùng ORM. TypeORM cung cấp `@VersionColumn`, và cột này tự tăng mỗi lần chúng ta gọi `save()` trên một entity đầy đủ. Nhưng nếu chúng ta gọi `repository.update()` để sửa một phần, phép kiểm tra version có thể không được kích hoạt, và chúng ta lại dính lost update dù nhìn vào code thì tưởng đã an toàn. Vì vậy chúng ta phải đọc tài liệu chính thức của đúng phiên bản đang dùng, và chúng ta phải tự viết một bài test chạy đồng thời để chứng minh rằng conflict thật sự bị chặn.

**English (bám cấu trúc tiếng Việt)**

Optimistic locking is implemented with a `version` column on every row, or with a timestamp column that plays a similar role. When we read that row out, we keep the old version value in our hand. When we write it down, we put the old version into the `WHERE` clause and at the same time we raise the version by one, that is, `UPDATE t SET ..., version = version + 1 WHERE id = $1 AND version = $2`. If the number of affected rows equals one, nobody has slipped into the middle and our record has been saved. If the number of affected rows equals zero, somebody has raised the version before us, and that is exactly the signal of a conflict. This is in fact a compare-and-swap at the level of a single row, the only difference being that the comparison is carried out by the database on our behalf. If we forget to check the number of affected rows, we will assume that everything has succeeded while our change has been dropped completely.

Here there is a trap that many teams fall into when they use an ORM. TypeORM provides `@VersionColumn`, and this column raises itself every time we call `save()` on a full entity. But if we call `repository.update()` to change one part only, the version check may not be triggered, and we get hit by a lost update again although the code looks safe when we look at it. Therefore we must read the official documentation of the exact version we are using, and we must write a concurrent test ourselves in order to prove that conflicts are really blocked.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| đóng vai trò tương tự | plays a similar role |
| giữ lại giá trị version cũ trong tay | keep the old version value in our hand |
| đưa version cũ vào mệnh đề WHERE | put the old version into the `WHERE` clause |
| không ai chen vào giữa | nobody has slipped into the middle |
| đó chính là tín hiệu conflict | that is exactly the signal of a conflict |
| ở mức một hàng dữ liệu | at the level of a single row |
| chỉ khác là | the only difference being that |
| được database thực hiện thay cho chúng ta | is carried out by the database on our behalf |
| thay đổi của chúng ta bị rơi mất hoàn toàn | our change has been dropped completely |
| một cái bẫy mà nhiều đội mắc phải | a trap that many teams fall into |
| có thể không được kích hoạt | may not be triggered |
| dù nhìn vào code thì tưởng đã an toàn | although the code looks safe when we look at it |

**Thuật ngữ cần nhớ**

- cột phiên bản → **a version column**
- so-sánh-rồi-tráo-đổi → **compare-and-swap (CAS)**
- mệnh đề WHERE → **the `WHERE` clause**
- bản ghi → **a record**
- một bài test chạy đồng thời → **a concurrent test**

---

## ③ ABA problem và vì sao version thắng

**Tiếng Việt**

Một câu hỏi rất hay được hỏi ở vòng senior là tại sao chúng ta dùng cột version thay vì so sánh trực tiếp với giá trị cũ. Lý do nằm ở ABA problem. Chúng ta hãy giả sử một giá trị đi từ A sang B rồi quay về A trong lúc chúng ta đang tính toán. Nếu chúng ta chỉ so sánh với giá trị cũ, chúng ta sẽ thấy nó vẫn bằng A và chúng ta sẽ kết luận rằng không có gì thay đổi. Nhưng thật ra đã có hai lần ghi xen vào giữa, cho nên giả định của chúng ta về trạng thái hiện tại đã sai. Một cột version đơn điệu tăng phát hiện được tình huống này, vì version đã nhảy từ mười lên mười hai dù giá trị nghiệp vụ quay về đúng như cũ. Hậu quả nếu chúng ta bỏ qua điểm này là chúng ta ghi đè lên công việc của người khác mà không hề hay biết, và loại lỗi đó gần như không thể tìm ra bằng log thông thường.

**English (bám cấu trúc tiếng Việt)**

A question that is asked very often in a senior round is why we use a version column instead of comparing directly with the old value. The reason lies in the ABA problem. Let us suppose that a value goes from A to B and then comes back to A while we are doing our computation. If we only compare with the old value, we will see that it still equals A and we will conclude that nothing has changed. But in reality there have been two writes in between, therefore our assumption about the current state is wrong. A monotonically increasing version column can detect this situation, because the version has jumped from ten to twelve although the business value has come back exactly as before. The consequence, if we ignore this point, is that we overwrite somebody else's work without being aware of it at all, and that kind of bug is almost impossible to find with ordinary logs.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| một câu hỏi rất hay được hỏi ở vòng senior | a question that is asked very often in a senior round |
| lý do nằm ở | the reason lies in |
| chúng ta hãy giả sử | let us suppose that |
| rồi quay về A trong lúc chúng ta đang tính toán | and then comes back to A while we are doing our computation |
| chúng ta sẽ kết luận rằng không có gì thay đổi | we will conclude that nothing has changed |
| đã có hai lần ghi xen vào giữa | there have been two writes in between |
| đơn điệu tăng | monotonically increasing |
| dù giá trị nghiệp vụ quay về đúng như cũ | although the business value has come back exactly as before |
| ghi đè lên công việc của người khác | overwrite somebody else's work |
| mà không hề hay biết | without being aware of it at all |

**Thuật ngữ cần nhớ**

- giá trị nghiệp vụ → **the business value**
- đơn điệu tăng → **monotonically increasing**
- ghi đè → **to overwrite**
- trạng thái hiện tại → **the current state**

---

## ④ Pessimistic bằng `FOR UPDATE` và biến thể `SKIP LOCKED`

**Tiếng Việt**

Pessimistic locking được hiện thực bằng chính lệnh khóa của database, phổ biến nhất là `SELECT ... FOR UPDATE`. Lệnh này khóa hàng ở chế độ độc quyền cho đến khi transaction kết thúc, cho nên transaction khác đụng vào hàng đó phải xếp hàng chờ. Trong khoảng thời gian giữ khóa, chúng ta có thể đọc, tính toán và ghi mà không ai chen vào giữa. Đổi lại, chúng ta đã biến các thao tác song song thành tuần tự trên đúng hàng đó, và thông lượng trên hàng đó bị giới hạn bởi độ dài của transaction. Vì vậy nguyên tắc bắt buộc là chúng ta giữ transaction thật ngắn và chúng ta tuyệt đối không gọi API bên ngoài trong lúc đang giữ khóa.

Một biến thể rất hữu ích là `SELECT ... FOR UPDATE SKIP LOCKED`, dùng để giải bài toán hàng đợi công việc nằm ngay trong database. Khi nhiều worker cùng quét bảng job, mặc định mỗi worker sẽ chờ đúng cái hàng mà worker khác đang khóa, và chúng ta được một đám worker đứng xếp hàng để tranh một công việc duy nhất. Với `SKIP LOCKED`, mỗi worker bỏ qua những hàng đang bị khóa và nhận ngay công việc kế tiếp còn rảnh. Nhờ đó mỗi worker lấy được một job khác nhau và mức tranh chấp giảm xuống gần như bằng không. Đây chính là nền tảng của nhiều hệ job queue xây trực tiếp trên PostgreSQL mà không cần thêm một message broker riêng.

**English (bám cấu trúc tiếng Việt)**

Pessimistic locking is implemented with the locking statement of the database itself, and the most common one is `SELECT ... FOR UPDATE`. This statement locks the row in exclusive mode until the transaction ends, therefore another transaction that touches that row has to queue up and wait. During the period when we hold the lock, we can read, compute and write without anybody slipping into the middle. In exchange, we have turned parallel operations into sequential ones on exactly that row, and the throughput on that row is limited by the length of the transaction. Therefore the compulsory rule is that we keep the transaction really short and we absolutely do not call an external API while we are holding the lock.

A very useful variant is `SELECT ... FOR UPDATE SKIP LOCKED`, which is used to solve the problem of a job queue that sits right inside the database. When many workers scan the job table together, by default each worker will wait for exactly the row that another worker is locking, and we end up with a crowd of workers queueing up to fight over a single job. With `SKIP LOCKED`, each worker skips the rows that are being locked and takes the next free job straight away. Thanks to that, each worker gets a different job and the contention drops down to almost zero. This is exactly the foundation of many job queue systems built directly on PostgreSQL without needing a separate message broker.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| khóa hàng ở chế độ độc quyền | locks the row in exclusive mode |
| đụng vào hàng đó phải xếp hàng chờ | touches that row has to queue up and wait |
| trong khoảng thời gian giữ khóa | during the period when we hold the lock |
| đổi lại | in exchange |
| biến các thao tác song song thành tuần tự | turned parallel operations into sequential ones |
| bị giới hạn bởi độ dài của transaction | is limited by the length of the transaction |
| nguyên tắc bắt buộc | the compulsory rule |
| trong lúc đang giữ khóa | while we are holding the lock |
| một đám worker đứng xếp hàng để tranh một công việc duy nhất | a crowd of workers queueing up to fight over a single job |
| nhận ngay công việc kế tiếp còn rảnh | takes the next free job straight away |
| mức tranh chấp giảm xuống gần như bằng không | the contention drops down to almost zero |
| mà không cần thêm một message broker riêng | without needing a separate message broker |

**Thuật ngữ cần nhớ**

- khóa độc quyền → **an exclusive lock**
- xếp hàng chờ → **to queue up and wait**
- hàng đợi công việc → **a job queue**
- bỏ qua hàng đang bị khóa → **to skip locked rows**
- mức tranh chấp → **contention**

---

## ⑤ Deadlock và thói quen lạm dụng `FOR UPDATE`

**Tiếng Việt**

Pessimistic lock mang theo một rủi ro riêng của nó là deadlock. Deadlock xảy ra khi hai transaction khóa hai hàng theo thứ tự ngược nhau, và mỗi bên chờ đúng cái mà bên kia đang giữ. Cách phòng đơn giản và hiệu quả nhất là chúng ta luôn khóa theo một thứ tự nhất quán, ví dụ chúng ta luôn khóa hàng có id nhỏ trước. Ngoài ra chúng ta giữ transaction ngắn và chúng ta khóa ở mức chi tiết nhất có thể, tức là khóa từng hàng chứ không khóa cả bảng. Nếu chúng ta bỏ qua các nguyên tắc này, database sẽ tự phát hiện deadlock và hủy một trong hai transaction, và người dùng nhận được một lỗi rất khó hiểu.

Sai lầm phổ biến thứ hai là thói quen thêm `FOR UPDATE` vào mọi câu đọc cho chắc. Cách làm này serialize những chỗ vốn không cần serialize, cho nên chúng ta mất tính song song mà không đổi lại được chút tính đúng đắn nào. Tệ hơn nữa, mỗi khóa được giữ đến hết transaction, và mỗi transaction chiếm một connection trong pool. Khi tải tăng lên, chúng ta sẽ thấy pool cạn, request xếp hàng ở tầng kết nối, và cả hệ thống chậm lại dù database gần như rảnh rỗi. Vì vậy chúng ta chỉ khóa đúng điểm nóng nơi thật sự có đọc-sửa-ghi, chứ chúng ta không khóa theo thói quen.

**English (bám cấu trúc tiếng Việt)**

A pessimistic lock carries its own risk, which is deadlock. A deadlock happens when two transactions lock two rows in the opposite order, and each side waits for exactly what the other side is holding. The simplest and most effective way to prevent it is that we always lock in a consistent order, for example we always lock the row with the smaller id first. Besides that, we keep transactions short and we lock at the most granular level possible, that is, we lock each row and not the whole table. If we ignore these rules, the database will detect the deadlock by itself and will abort one of the two transactions, and the user receives a very confusing error.

The second common mistake is the habit of adding `FOR UPDATE` to every read just to be safe. This practice serialises the places that never needed to be serialised, therefore we lose parallelism without getting any extra correctness in exchange. Even worse, every lock is held until the end of the transaction, and every transaction takes up a connection in the pool. When the load goes up, we will see the pool run dry, requests queueing at the connection layer, and the whole system slowing down although the database is almost idle. Therefore we only lock at the exact hot spot where there really is a read-modify-write, and we do not lock out of habit.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| mang theo một rủi ro riêng của nó | carries its own risk |
| khóa hai hàng theo thứ tự ngược nhau | lock two rows in the opposite order |
| chờ đúng cái mà bên kia đang giữ | waits for exactly what the other side is holding |
| khóa theo một thứ tự nhất quán | lock in a consistent order |
| ở mức chi tiết nhất có thể | at the most granular level possible |
| hủy một trong hai transaction | abort one of the two transactions |
| thêm `FOR UPDATE` vào mọi câu đọc cho chắc | adding `FOR UPDATE` to every read just to be safe |
| những chỗ vốn không cần serialize | the places that never needed to be serialised |
| chúng ta sẽ thấy pool cạn | we will see the pool run dry |
| dù database gần như rảnh rỗi | although the database is almost idle |
| chúng ta không khóa theo thói quen | we do not lock out of habit |

**Thuật ngữ cần nhớ**

- khóa chết → **a deadlock**
- thứ tự khóa nhất quán → **a consistent lock order**
- mức chi tiết của khóa → **lock granularity**
- cạn pool kết nối → **to exhaust the connection pool**
- điểm nóng → **a hot spot**

---

## ⑥ Chọn cái nào, và cái bẫy ngược

**Tiếng Việt**

Việc chọn giữa hai loại lock phải dựa trên workload thật, chứ không dựa trên cảm tính. Khi mức tranh chấp thấp, khi tải nghiêng về đọc, và khi việc thử lại rẻ hoặc idempotent, chúng ta chọn optimistic. Khi mức tranh chấp cao, khi nhiều transaction cùng ghi vào một hàng nóng, và khi việc thử lại đắt, chúng ta chọn pessimistic. Nguyên tắc của một tech lead là chúng ta đo tỉ lệ conflict trên production rồi mới quyết định. Nếu chúng ta không đo, chúng ta chỉ đang đoán, và một quyết định đúng nhờ may mắn vẫn là một quyết định mà chúng ta không bảo vệ được trong phỏng vấn.

Có một cái bẫy ngược mà nhiều người không ngờ tới: dưới mức tranh chấp cao, optimistic có thể tệ hơn pessimistic. Khi hàng chục transaction cùng đụng vào một hàng, phần lớn trong số đó sẽ trượt phép kiểm tra version và phải thử lại. Các lần thử lại đó lại tiếp tục đụng nhau, cho nên chúng ta rơi vào tình trạng retry dồn dập, gần giống livelock, nghĩa là hệ thống rất bận nhưng rất ít việc được hoàn thành. Pessimistic trong tình huống đó tuy phải xếp hàng, nhưng mỗi transaction tiến đều và không có công sức nào bị vứt đi. Đây chính là câu trả lời phân biệt người senior với người mới học, bởi vì optimistic không phải lúc nào cũng nhanh hơn.

**English (bám cấu trúc tiếng Việt)**

The choice between the two kinds of lock must be based on the real workload, and not on gut feeling. When the contention is low, when the load leans towards reads, and when a retry is cheap or idempotent, we choose optimistic. When the contention is high, when many transactions write into one hot row, and when a retry is expensive, we choose pessimistic. The rule of a tech lead is that we measure the conflict rate on production and only then we decide. If we do not measure, we are only guessing, and a decision that is right by luck is still a decision that we cannot defend in an interview.

There is a reverse trap that many people do not expect: under high contention, optimistic can be worse than pessimistic. When dozens of transactions touch one row together, most of them will fail the version check and will have to retry. Those retries then keep colliding with each other, therefore we fall into a state of retry storm, close to livelock, which means that the system is very busy but very little work is completed. Pessimistic in that situation does have to queue, but every transaction makes steady progress and no effort is thrown away. This is exactly the answer that separates a senior person from a beginner, because optimistic is not always faster.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| chứ không dựa trên cảm tính | and not on gut feeling |
| tải nghiêng về đọc | the load leans towards reads |
| một hàng nóng | one hot row |
| đo tỉ lệ conflict trên production rồi mới quyết định | measure the conflict rate on production and only then we decide |
| một quyết định đúng nhờ may mắn | a decision that is right by luck |
| chúng ta không bảo vệ được trong phỏng vấn | that we cannot defend in an interview |
| một cái bẫy ngược mà nhiều người không ngờ tới | a reverse trap that many people do not expect |
| sẽ trượt phép kiểm tra version | will fail the version check |
| lại tiếp tục đụng nhau | keep colliding with each other |
| rơi vào tình trạng retry dồn dập | fall into a state of retry storm |
| mỗi transaction tiến đều | every transaction makes steady progress |
| không có công sức nào bị vứt đi | no effort is thrown away |

**Thuật ngữ cần nhớ**

- tỉ lệ xung đột → **the conflict rate**
- bão thử lại → **a retry storm**
- tình trạng bận mà không xong việc → **livelock**
- tiến triển đều → **steady progress**
- lặp lại không đổi kết quả → **idempotent**

---

## ⑦ Thiết kế retry an toàn, và khi nào lock là thừa

**Tiếng Việt**

Nếu chúng ta chọn optimistic, chúng ta phải thiết kế phần retry một cách nghiêm túc. Số lần thử lại phải có trần, ví dụ ba lần, chứ chúng ta không được retry vô hạn. Mỗi lần thử lại phải đọc lại state mới nhất rồi tính toán lại, bởi vì tính lại trên dữ liệu cũ sẽ hỏng ngay ở lần ghi tiếp theo. Giữa các lần thử, chúng ta chờ một khoảng có backoff và có jitter, để các retry không đồng pha rồi lại đụng nhau đúng cùng một lúc. Khi hết số lần thử, chúng ta trả về một lỗi rõ ràng cho người dùng, thay vì chúng ta để họ chờ mãi mà không biết chuyện gì đang xảy ra.

Cuối cùng, chúng ta cần biết khi nào lock là thừa. Với một bộ đếm hoặc một trường stock đơn giản, câu lệnh `UPDATE ... SET stock = stock - 1 WHERE stock > 0` đã đủ và gọn hơn cột version rất nhiều. Cột version chỉ đáng dùng khi chúng ta sửa nhiều trường cùng lúc, hoặc khi chúng ta cần báo conflict ra tận giao diện cho người dùng biết. Và nếu ứng dụng chạy nhiều instance mà đã dùng optimistic version, chúng ta không cần thêm distributed lock, bởi vì phép kiểm tra version ở database vốn đã là một điểm đồng bộ atomic xuyên mọi instance. Nhận thức cốt lõi của bài này là một ràng buộc trong database cũng chính là một cơ chế điều khiển đồng thời.

**English (bám cấu trúc tiếng Việt)**

If we choose optimistic, we have to design the retry part seriously. The number of retries must have a ceiling, for example three times, and we must not retry endlessly. Every retry must read the newest state again and then compute again, because computing again on old data will break at the very next write. Between the attempts, we wait for an interval with backoff and with jitter, so that the retries do not come into phase and collide with each other at exactly the same moment. When the attempts run out, we return a clear error to the user, instead of letting them wait forever without knowing what is going on.

Finally, we need to know when a lock is unnecessary. For a simple counter or a simple stock field, the statement `UPDATE ... SET stock = stock - 1 WHERE stock > 0` is already enough and is far neater than a version column. A version column is only worth using when we change many fields at the same time, or when we need to report the conflict all the way out to the interface so that the user knows. And if the application runs on many instances but already uses an optimistic version, we do not need to add a distributed lock, because the version check in the database is already an atomic synchronisation point across every instance. The core realisation of this lesson is that a constraint in the database is itself a concurrency control mechanism.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| số lần thử lại phải có trần | the number of retries must have a ceiling |
| chúng ta không được retry vô hạn | we must not retry endlessly |
| sẽ hỏng ngay ở lần ghi tiếp theo | will break at the very next write |
| để các retry không đồng pha | so that the retries do not come into phase |
| khi hết số lần thử | when the attempts run out |
| để họ chờ mãi mà không biết chuyện gì đang xảy ra | letting them wait forever without knowing what is going on |
| khi nào lock là thừa | when a lock is unnecessary |
| gọn hơn cột version rất nhiều | far neater than a version column |
| chỉ đáng dùng khi | is only worth using when |
| báo conflict ra tận giao diện | report the conflict all the way out to the interface |
| một điểm đồng bộ atomic xuyên mọi instance | an atomic synchronisation point across every instance |
| nhận thức cốt lõi của bài này | the core realisation of this lesson |

**Thuật ngữ cần nhớ**

- giãn cách tăng dần → **backoff**
- nhiễu ngẫu nhiên chống đồng pha → **jitter**
- số lần thử tối đa → **the maximum number of attempts**
- điều khiển đồng thời → **concurrency control**
- khóa phân tán → **a distributed lock**

---

## Mô hình ghi nhớ

**Tiếng Việt**

Pessimistic nghĩa là khóa trước rồi mới làm, cho nên người khác phải chờ; optimistic nghĩa là cứ làm rồi kiểm tra lúc commit, cho nên khi hỏng thì phải thử lại. Chúng ta chọn theo tỉ lệ conflict đo được, chứ không chọn theo cảm tính, và chúng ta nhớ rằng một ràng buộc trong database cũng đã là một cơ chế điều khiển đồng thời.

**English (bám cấu trúc tiếng Việt)**

Pessimistic means locking first and doing the work afterwards, therefore other people have to wait; optimistic means just doing the work and checking at commit time, therefore when it fails we have to retry. We choose according to the conflict rate that we have measured, and not according to gut feeling, and we remember that a constraint in the database is already a concurrency control mechanism.

---

## Bảng thuật ngữ tổng hợp

| Tiếng Việt | Tiếng Anh | Ghi chú phát âm (Anh–Anh) |
|---|---|---|
| khóa bi quan | **a pessimistic lock** | *pe-si-MIS-tik* — trọng âm áp chót |
| khóa lạc quan | **an optimistic lock** | *op-ti-MIS-tik* — trọng âm áp chót |
| triết lý | **philosophy** | *fi-LO-sơ-fi* — trọng âm âm thứ hai, `ph` = /f/ |
| xung đột ghi | **a write conflict** | *write* câm chữ **w**, đọc y hệt "right"; *conflict* danh từ = *KON-flikt*, động từ = *kợn-FLIKT* |
| giao dịch | **a transaction** | *tran-ZAK-shợn* — chữ `s` đọc /z/, không đọc /s/ |
| lúc commit | **at commit time** | *kơ-MIT* — trọng âm sau |
| cột phiên bản | **a version column** | *version* Anh = /ˈvɜːʃn/ "VƠƠ-shợn" với /ʃ/, không đọc "ver-zần"; *column* câm chữ **n** cuối |
| so-sánh-rồi-tráo-đổi | **compare-and-swap (CAS)** | *swap* /swɒp/ "xoóp", không đọc "sờ-goáp" |
| mệnh đề WHERE | **the `WHERE` clause** | *clause* /klɔːz/ — đuôi đọc /z/ |
| bản ghi | **a record** | danh từ = *RE-kơd*; động từ = *ri-KOD* — trọng âm đổi theo từ loại |
| bài test chạy đồng thời | **a concurrent test** | *kơn-KA-rợnt* /kənˈkʌrənt/ — trọng âm giữa |
| đơn điệu tăng | **monotonically increasing** | *mo-nơ-TON-ik-li* — trọng âm âm thứ ba |
| ghi đè | **to overwrite** | trọng âm chính ở *WRITE*, chữ **w** trong `write` vẫn câm |
| trạng thái hiện tại | **the current state** | *KA-rợnt* /ˈkʌrənt/ — không đọc "kiu-rần" |
| khóa độc quyền | **an exclusive lock** | *ik-SKLUU-siv* — cụm phụ âm **skl** phải bật ra |
| xếp hàng chờ | **to queue up and wait** | *queue* /kjuː/ đọc đúng như tên chữ cái **Q** |
| hàng đợi công việc | **a job queue** | |
| bỏ qua hàng đang bị khóa | **to skip locked rows** | *rows* kết thúc /z/; cụm **sk-** trong `skip` phải rõ |
| mức tranh chấp | **contention** | *kơn-TEN-shợn* — trọng âm giữa |
| thông lượng | **throughput** | *THRUU-put* /ˈθruːpʊt/ — âm **th** và cụm **thr** |
| khóa chết | **a deadlock** | *DED-lok* — `dead` = /ded/, không đọc "đi-át" |
| thứ tự khóa nhất quán | **a consistent lock order** | *kơn-SIS-tợnt* — trọng âm giữa |
| mức chi tiết của khóa | **lock granularity** | *gra-nyu-LA-rơ-ti* — trọng âm âm thứ tư |
| tuần tự hóa | **to serialise** | Anh viết **-ise**; đọc *SIA-ri-ơ-laiz* /ˈsɪəriəlaɪz/ |
| cạn pool kết nối | **to exhaust the connection pool** | *exhaust* = *ig-ZOST* /ɪɡˈzɔːst/ — câm chữ **h**, cụm cuối **-st** |
| điểm nóng | **a hot spot** | |
| rảnh rỗi | **idle** | /ˈaɪdl/ "AI-đợl", không đọc "i-đồ" |
| tỉ lệ xung đột | **the conflict rate** | |
| bão thử lại | **a retry storm** | |
| bận mà không xong việc | **livelock** | *LAIV-lok* — `live` ở đây đọc /laɪv/ |
| tiến triển đều | **steady progress** | *PROU-gres* (danh từ, Anh) — khác động từ *prơ-GRES* |
| lặp lại không đổi kết quả | **idempotent** | *ai-DEM-pơ-tợnt* — trọng âm âm thứ hai; từ hay đọc sai nhất trong mục này |
| giãn cách tăng dần | **backoff** | |
| nhiễu ngẫu nhiên | **jitter** | *DJI-tơ* — âm đầu /dʒ/ như trong "job" |
| số lần thử tối đa | **the maximum number of attempts** | *attempts* — cụm cuối **-mpts** rất khó, tập nói chậm: *ơ-TEMPTS* |
| điều khiển đồng thời | **concurrency control** | *kơn-KA-rợn-si* — trọng âm giữa |
| khóa phân tán | **a distributed lock** | *dis-TRI-byu-tid* — trọng âm âm thứ hai |
| điểm đồng bộ | **a synchronisation point** | *sing-krơ-nai-ZÊI-shợn* — trọng âm áp chót; Anh viết **-sation** |
| cảm tính | **gut feeling** | |
| ràng buộc trong database | **a database constraint** | *kơn-STRÊINT* — cụm **str** và cụm cuối **-nt** |

---

## Bài nói

> **Cách luyện:** bật ghi âm, nói **60–90 giây** cho mỗi đề, **không nhìn tài liệu**. Mỗi bài nói phải dùng ít nhất **sáu thuật ngữ** trong bảng trên. Nói xong nghe lại một lần, đánh dấu chỗ nào bị vấp hoặc phát âm sai, rồi nói lại đề đó một lần nữa.

1. **Explain to a junior developer** the difference in philosophy between optimistic and pessimistic locking, and give one real feature in a product where each of them is the right choice.

2. **Describe what happens**, step by step, when two admins open the same order, both edit it, and both press save, in a system that uses a version column. Say exactly what the second admin sees and why.

3. **A colleague says:** *"Optimistic locking is lock-free, so it is always faster than `FOR UPDATE`."* Explain what is wrong with that claim, and describe the workload where the opposite is true.

4. **Someone on your team wants to add `FOR UPDATE` to every read query** in the checkout service "to be safe". Explain why you would push back, and what you would measure before agreeing to any locking at all.

5. **When would you choose a version column over a plain atomic conditional update**, and when is the version column simply unnecessary? Ground your answer in the number of fields and in what the user needs to be told.

6. **Explain why the ABA problem** makes value comparison unsafe, and why a monotonically increasing version solves it. Use a concrete example with numbers.

7. **A senior engineer proposes adding a distributed lock in Redis** because the service now runs on six instances, even though the table already has a version column. Explain why you disagree, and what the version check already guarantees across instances.

---

> 🔚 **Hết Bài 2.** Gõ `Làm Bài 3` để sang *Idempotency (góc concurrency/distributed)*.
