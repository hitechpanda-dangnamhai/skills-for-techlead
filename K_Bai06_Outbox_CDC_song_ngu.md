# Bài 6 — Dual-write, Transactional Outbox & CDC
### Bản song ngữ: tiếng Anh bám cấu trúc tiếng Việt

> **Cách dùng bài này:** đọc đoạn tiếng Việt trước → tự dịch trong đầu, không nhìn xuống → so sánh với đoạn tiếng Anh ngay bên dưới → **đọc to bản tiếng Anh hai lần**: lần một đọc chậm cho đúng từng âm, lần hai đọc liền mạch như đang nói với người phỏng vấn.

---

## ① Bài toán ghi kép và bốn kịch bản

**Tiếng Việt**

Bài này vá đúng cái bẫy mà chúng ta đã gài từ Bài 1, đó là dòng ghi cơ sở dữ liệu rồi ngay sau đó là dòng phát event. Vấn đề gốc nằm ở chỗ chúng ta phải ghi vào hai hệ thống khác nhau, nhưng hai hệ thống đó không nằm chung một transaction. Cơ sở dữ liệu có transaction của riêng nó, còn broker thì có cơ chế riêng của nó, và không ai điều phối hai bên. Vì vậy mỗi lần chạy qua đoạn code đó, chúng ta rơi vào một trong bốn kịch bản. Chúng ta nên thuộc cả bốn kịch bản này, vì người phỏng vấn thường yêu cầu liệt kê đủ.

Kịch bản thứ nhất là cơ sở dữ liệu commit thành công và việc phát event cũng thành công, và đây là trường hợp duy nhất chúng ta mong muốn. Kịch bản thứ hai là cơ sở dữ liệu commit thành công nhưng việc phát event thất bại, và hậu quả là chúng ta mất event. Khi đó đơn hàng nằm trong cơ sở dữ liệu nhưng kho, email và phân tích không hề biết đơn đó tồn tại. Kịch bản thứ ba là việc phát event thành công nhưng cơ sở dữ liệu lại quay lui, và hậu quả là chúng ta sinh ra một event ma. Khi đó các service phía sau xử lý một đơn hàng không hề tồn tại, và dữ liệu của họ lệch vĩnh viễn so với chúng ta. Kịch bản thứ tư là cả hai bên cùng thất bại, và may mắn là trạng thái vẫn nhất quán vì chưa có gì được làm.

**English (bám cấu trúc tiếng Việt)**

This lesson patches exactly the trap that we set back in Lesson 1, which is the line that writes to the database and then right after it the line that publishes the event. The root problem lies in the fact that we have to write into two different systems, but those two systems do not sit inside one transaction. The database has a transaction of its own, while the broker has a mechanism of its own, and nobody coordinates the two sides. Therefore every time we run through that piece of code, we fall into one of four scenarios. We should know all four scenarios by heart, because interviewers often ask us to list them all.

The first scenario is that the database commits successfully and publishing the event also succeeds, and this is the only case we actually want. The second scenario is that the database commits successfully but publishing the event fails, and the consequence is that we lose the event. At that point the order sits in the database but inventory, email and analytics have no idea that this order exists. The third scenario is that publishing the event succeeds but the database rolls back instead, and the consequence is that we create a ghost event. At that point the downstream services process an order that does not exist at all, and their data drifts away from ours permanently. The fourth scenario is that both sides fail together, and fortunately the state is still consistent because nothing has been done.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| vá đúng cái bẫy mà chúng ta đã gài | patches exactly the trap that we set |
| vấn đề gốc nằm ở chỗ | the root problem lies in the fact that |
| không ai điều phối hai bên | nobody coordinates the two sides |
| mỗi lần chạy qua đoạn code đó | every time we run through that piece of code |
| chúng ta nên thuộc cả bốn kịch bản | we should know all four scenarios by heart |
| trường hợp duy nhất chúng ta mong muốn | the only case we actually want |
| không hề biết đơn đó tồn tại | have no idea that this order exists |
| cơ sở dữ liệu lại quay lui | the database rolls back instead |
| chúng ta sinh ra một event ma | we create a ghost event |
| dữ liệu của họ lệch vĩnh viễn so với chúng ta | their data drifts away from ours permanently |
| may mắn là trạng thái vẫn nhất quán | fortunately the state is still consistent |

**Thuật ngữ cần nhớ**

- ghi kép → **the dual-write problem**
- quay lui giao dịch → **to roll back**
- event ma → **a ghost event**
- các service phía sau → **the downstream services**
- lệch dữ liệu dần theo thời gian → **data drift**

---

## ② Vì sao khối try/catch không cứu được

**Tiếng Việt**

Phản xạ đầu tiên của nhiều lập trình viên là bọc cả hai lệnh vào một khối try/catch rồi cho rằng như vậy là an toàn. Chúng ta phải giải thích rõ vì sao lập luận đó sai, vì đây là một câu phản biện rất hay gặp. Lý do thứ nhất là lệnh phát event không nằm trong transaction của cơ sở dữ liệu, nên lệnh quay lui của cơ sở dữ liệu không thu hồi được message đã rời khỏi ứng dụng. Một khi message đã tới broker, nó thuộc về broker, và không có cách nào để chúng ta gọi nó quay lại. Lý do thứ hai là tiến trình có thể chết ở đúng khoảnh khắc giữa hai lệnh, và khi tiến trình đã chết thì khối catch cũng không bao giờ chạy.

Còn một trường hợp tinh vi hơn mà chúng ta nên nêu ra để ghi điểm. Việc phát event có thể đã thành công ở phía broker nhưng gói tin xác nhận lại bị mất trên đường về. Ứng dụng của chúng ta nhìn thấy một lỗi, nên nó quay lui cơ sở dữ liệu, trong khi thật ra event đã nằm chắc chắn trong broker rồi. Đây chính xác là kịch bản event ma, và nó sinh ra ngay cả khi code của chúng ta xử lý lỗi rất cẩn thận. Kết luận là chúng ta không thể sửa bài toán này bằng cách viết code khéo hơn, mà chúng ta phải đổi sang một mẫu kiến trúc khác.

**English (bám cấu trúc tiếng Việt)**

The first reflex of many developers is to wrap both statements into one try/catch block and then assume that this is safe. We have to explain clearly why that argument is wrong, because this is a very common push-back question. The first reason is that the publish statement does not sit inside the database transaction, so the rollback of the database cannot take back a message that has already left the application. Once the message has reached the broker, it belongs to the broker, and there is no way for us to call it back. The second reason is that the process can die at exactly the moment between the two statements, and once the process has died the catch block never runs either.

There is one more subtle case that we should raise in order to score points. Publishing the event may already have succeeded on the broker side but the acknowledgement packet was lost on the way back. Our application sees an error, so it rolls the database back, while in reality the event is already sitting safely in the broker. This is exactly the ghost event scenario, and it arises even when our code handles errors very carefully. The conclusion is that we cannot fix this problem by writing cleverer code, but we have to switch to a different architectural pattern.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| bọc cả hai lệnh vào một khối try/catch | wrap both statements into one try/catch block |
| rồi cho rằng như vậy là an toàn | and then assume that this is safe |
| một câu phản biện rất hay gặp | a very common push-back question |
| không thu hồi được message đã rời khỏi ứng dụng | cannot take back a message that has already left the application |
| một khi message đã tới broker | once the message has reached the broker |
| không có cách nào để chúng ta gọi nó quay lại | there is no way for us to call it back |
| chết ở đúng khoảnh khắc giữa hai lệnh | die at exactly the moment between the two statements |
| một trường hợp tinh vi hơn | one more subtle case |
| bị mất trên đường về | was lost on the way back |
| event đã nằm chắc chắn trong broker rồi | the event is already sitting safely in the broker |
| bằng cách viết code khéo hơn | by writing cleverer code |

**Thuật ngữ cần nhớ**

- khối bắt lỗi → **a try/catch block**
- gói tin xác nhận → **the acknowledgement packet**
- thu hồi lại → **to take back**
- mẫu kiến trúc → **an architectural pattern**
- trường hợp tinh vi → **a subtle case**

---

## ③ Transactional Outbox: biến hai hệ thống thành một transaction

**Tiếng Việt**

Mẫu Transactional Outbox giải bài toán này bằng một ý tưởng rất gọn. Thay vì ghi vào cơ sở dữ liệu rồi phát lên broker, chúng ta ghi dữ liệu nghiệp vụ và ghi event vào một bảng tên là `outbox`, cả hai trong cùng một transaction của cơ sở dữ liệu. Vì cả hai lần ghi đều thuộc về một transaction ACID duy nhất, chúng nguyên tử với nhau một cách tự nhiên. Sau đó một tiến trình bất đồng bộ sẽ đọc bảng `outbox` và đẩy các event đó lên broker. Điểm mấu chốt là chúng ta đã chuyển bài toán từ hai hệ thống về đúng một transaction cơ sở dữ liệu.

Chúng ta nên nói rõ mẫu này loại bỏ được kịch bản nào, vì đó là phần người phỏng vấn muốn nghe. Kịch bản mất event bị loại bỏ, vì nếu đơn hàng đã được commit thì dòng event cũng đã được commit ngay bên cạnh nó. Kịch bản event ma cũng bị loại bỏ, vì nếu transaction quay lui thì dòng event biến mất cùng với đơn hàng. Nói cách khác, event sống và chết cùng với dữ liệu nghiệp vụ, và đó chính là bảo đảm mà chúng ta cần. Điều duy nhất còn lại là event có thể được phát nhiều hơn một lần, và chúng ta sẽ xử lý phần đó ở phía nhận.

**English (bám cấu trúc tiếng Việt)**

The Transactional Outbox pattern solves this problem with a very neat idea. Instead of writing into the database and then publishing to the broker, we write the business data and write the event into a table called `outbox`, both of them inside the same database transaction. Because both writes belong to one single ACID transaction, they are atomic with each other naturally. After that an asynchronous process will read the `outbox` table and push those events up to the broker. The crucial point is that we have moved the problem from two systems back to exactly one database transaction.

We should say clearly which scenarios this pattern eliminates, because that is the part interviewers want to hear. The lost-event scenario is eliminated, because if the order has been committed then the event row has been committed right next to it. The ghost-event scenario is eliminated too, because if the transaction rolls back then the event row disappears together with the order. In other words, the event lives and dies together with the business data, and that is exactly the guarantee we need. The only thing left is that the event may be published more than once, and we will handle that part on the receiving side.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| bằng một ý tưởng rất gọn | with a very neat idea |
| thay vì ghi vào ... rồi phát lên | instead of writing into ... and then publishing to |
| cả hai trong cùng một transaction | both of them inside the same transaction |
| nguyên tử với nhau một cách tự nhiên | atomic with each other naturally |
| điểm mấu chốt là | the crucial point is that |
| chúng ta đã chuyển bài toán từ ... về ... | we have moved the problem from ... back to ... |
| mẫu này loại bỏ được kịch bản nào | which scenarios this pattern eliminates |
| dòng event cũng đã được commit ngay bên cạnh nó | the event row has been committed right next to it |
| biến mất cùng với đơn hàng | disappears together with the order |
| event sống và chết cùng với dữ liệu nghiệp vụ | the event lives and dies together with the business data |
| điều duy nhất còn lại là | the only thing left is that |

**Thuật ngữ cần nhớ**

- mẫu hộp thư gửi có transaction → **the Transactional Outbox pattern**
- dữ liệu nghiệp vụ → **the business data**
- nguyên tử với nhau → **atomic with each other**
- dòng event trong bảng → **the event row**
- bảo đảm không mất event → **the no-loss guarantee**

---

## ④ Đẩy outbox ra broker bằng tiến trình quét bảng

**Tiếng Việt**

Cách đơn giản nhất để đẩy event ra broker là dùng một tiến trình quét bảng theo chu kỳ. Tiến trình đó chạy đều đặn, ví dụ mỗi hai giây một lần, và nó chọn ra những dòng chưa được gửi. Với mỗi dòng, nó phát event lên broker rồi đánh dấu dòng đó là đã gửi. Ưu điểm rõ nhất của cách này là chúng ta không phải thêm bất kỳ hạ tầng nào, vì tất cả chỉ là một công việc chạy nền trong chính ứng dụng. Vì vậy đây thường là lựa chọn hợp lý cho một đội nhỏ hoặc cho giai đoạn đầu của hệ thống.

Tuy nhiên chúng ta phải nêu ra hai nhược điểm, vì người phỏng vấn luôn hỏi về mặt trái. Nhược điểm thứ nhất là độ trễ, vì event phải chờ tới lượt quét tiếp theo mới được phát đi. Nhược điểm thứ hai là tải lên cơ sở dữ liệu, vì chúng ta liên tục chạy các truy vấn quét ngay cả khi không có việc gì để làm. Ngoài ra, nếu chúng ta chạy nhiều bản của tiến trình quét cùng lúc, hai bản có thể cùng chọn một dòng và phát event hai lần. Vì vậy chúng ta phải khoá dòng khi chọn, ví dụ bằng cách chọn kèm mệnh đề khoá và bỏ qua dòng đang bị khoá.

**English (bám cấu trúc tiếng Việt)**

The simplest way to push events out to the broker is to use a process that polls the table on a cycle. That process runs regularly, for example once every two seconds, and it picks out the rows that have not been sent. For each row, it publishes the event to the broker and then marks that row as sent. The clearest advantage of this way is that we do not have to add any infrastructure at all, because all of it is just a background job inside the application itself. Therefore this is often a reasonable choice for a small team or for the early stage of a system.

However we must raise two disadvantages, because interviewers always ask about the downside. The first disadvantage is latency, because the event has to wait for the next polling round before it is published. The second disadvantage is load on the database, because we continuously run scanning queries even when there is nothing to do. Besides, if we run several copies of the polling process at the same time, two copies may pick the same row and publish the event twice. Therefore we have to lock the row when we select it, for example by selecting with a locking clause and skipping the rows that are already locked.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| một tiến trình quét bảng theo chu kỳ | a process that polls the table on a cycle |
| nó chọn ra những dòng chưa được gửi | it picks out the rows that have not been sent |
| đánh dấu dòng đó là đã gửi | marks that row as sent |
| chúng ta không phải thêm bất kỳ hạ tầng nào | we do not have to add any infrastructure at all |
| một công việc chạy nền | a background job |
| cho giai đoạn đầu của hệ thống | for the early stage of a system |
| người phỏng vấn luôn hỏi về mặt trái | interviewers always ask about the downside |
| chờ tới lượt quét tiếp theo | wait for the next polling round |
| ngay cả khi không có việc gì để làm | even when there is nothing to do |
| hai bản có thể cùng chọn một dòng | two copies may pick the same row |
| chọn kèm mệnh đề khoá | selecting with a locking clause |
| bỏ qua dòng đang bị khoá | skipping the rows that are already locked |

**Thuật ngữ cần nhớ**

- tiến trình quét và phát → **a polling publisher**
- công việc chạy nền → **a background job**
- mặt trái, nhược điểm → **the downside**
- khoá dòng khi chọn → **to lock the row on select**
- bỏ qua dòng đang khoá → **skip locked**

---

## ⑤ CDC: đọc thẳng nhật ký giao dịch của cơ sở dữ liệu

**Tiếng Việt**

Cách thứ hai và cũng là cách hiện đại hơn là bắt thay đổi dữ liệu, thường được gọi tắt là CDC. Ý tưởng là chúng ta không truy vấn bảng nữa, mà chúng ta đọc thẳng nhật ký giao dịch mà cơ sở dữ liệu vốn đã ghi ra. Ở PostgreSQL, nhật ký đó là write-ahead log, còn ở MySQL thì nó là binary log. Một công cụ như Debezium sẽ theo dõi nhật ký này và biến mỗi thay đổi thành một event gửi lên broker gần như tức thì. Nhờ vậy chúng ta có độ trễ rất thấp mà không phải chạy các truy vấn quét lặp đi lặp lại.

Chúng ta nên nêu rõ vì sao cách này tốt hơn việc quét bảng để đồng bộ dữ liệu giữa các hệ thống. Lý do thứ nhất là nhật ký giao dịch ghi lại mọi thay đổi, kể cả những thay đổi do một tập lệnh chạy bên ngoài ứng dụng gây ra. Lý do thứ hai là chúng ta không bao giờ bỏ sót một bản cập nhật nào, trong khi cách quét theo cột thời gian có thể bỏ sót nếu hai lần ghi rơi vào cùng một mốc thời gian. Lý do thứ ba là tải lên cơ sở dữ liệu nhẹ hơn nhiều, vì đọc nhật ký rẻ hơn quét bảng liên tục. Cái giá phải trả là chúng ta thêm một mảnh hạ tầng phải vận hành, và chúng ta phải bật chế độ nhật ký ở mức logic trên cơ sở dữ liệu.

**English (bám cấu trúc tiếng Việt)**

The second way, and also the more modern one, is change data capture, which is usually shortened to CDC. The idea is that we no longer query the table, but we read straight from the transaction log that the database already writes out anyway. In PostgreSQL that log is the write-ahead log, while in MySQL it is the binary log. A tool such as Debezium will follow this log and turn every change into an event sent up to the broker almost instantly. Thanks to that we get very low latency without having to run scanning queries over and over again.

We should state clearly why this way is better than polling a table in order to synchronise data between systems. The first reason is that the transaction log records every change, including the changes caused by a script running outside the application. The second reason is that we never miss a single update, while the way that polls by a timestamp column can miss updates if two writes land on the same timestamp. The third reason is that the load on the database is much lighter, because reading the log is cheaper than scanning the table continuously. The price we pay is that we add one more piece of infrastructure to operate, and we have to turn on logical-level logging on the database.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| bắt thay đổi dữ liệu | change data capture |
| thường được gọi tắt là | usually shortened to |
| chúng ta không truy vấn bảng nữa | we no longer query the table |
| mà cơ sở dữ liệu vốn đã ghi ra | that the database already writes out anyway |
| sẽ theo dõi nhật ký này | will follow this log |
| gần như tức thì | almost instantly |
| chạy các truy vấn quét lặp đi lặp lại | run scanning queries over and over again |
| kể cả những thay đổi do một tập lệnh chạy bên ngoài | including the changes caused by a script running outside |
| chúng ta không bao giờ bỏ sót | we never miss |
| nếu hai lần ghi rơi vào cùng một mốc thời gian | if two writes land on the same timestamp |
| cái giá phải trả là | the price we pay is that |
| bật chế độ nhật ký ở mức logic | turn on logical-level logging |

**Thuật ngữ cần nhớ**

- bắt thay đổi dữ liệu → **change data capture (CDC)**
- nhật ký giao dịch → **the transaction log**
- nhật ký ghi trước → **the write-ahead log (WAL)**
- bỏ sót bản cập nhật → **to miss an update**
- đồng bộ dữ liệu giữa hai hệ thống → **to synchronise data between systems**

---

## ⑥ Inbox: nửa còn lại của hiệu quả một lần

**Tiếng Việt**

Outbox giải quyết được vế không mất event, nhưng nó không giải quyết vế không trùng event. Lý do rất đơn giản, đó là tiến trình đẩy có thể phát một dòng lên broker rồi chết trước khi kịp đánh dấu dòng đó là đã gửi. Khi nó sống lại, nó sẽ thấy dòng đó vẫn chưa được đánh dấu và nó phát lại một lần nữa. Vì vậy Outbox cho chúng ta ngữ nghĩa giao ít nhất một lần ở phía phát, đúng như những gì chúng ta đã học ở bài trước. Phần còn lại phải được xử lý ở phía nhận, và mẫu tương ứng gọi là Inbox.

Mẫu Inbox chính là hình ảnh đối xứng của mẫu Outbox. Ở phía nhận, consumer ghi mã sự kiện vừa nhận vào một bảng `inbox`, và nó làm việc đó trong cùng transaction với phần cập nhật nghiệp vụ. Nếu mã sự kiện đó đã tồn tại, consumer biết rằng mình đã xử lý rồi, nên nó bỏ qua và không chạy lại tác dụng phụ. Nhờ cặp Outbox và Inbox, chúng ta có một câu tóm tắt rất dễ nhớ cho phỏng vấn: Outbox lo phần không mất, còn Inbox lo phần không trùng. Hai nửa đó ghép lại chính là hiệu quả một lần trên toàn tuyến.

**English (bám cấu trúc tiếng Việt)**

The Outbox solves the no-loss half, but it does not solve the no-duplicate half. The reason is very simple, which is that the publishing process may push a row up to the broker and then die before it manages to mark that row as sent. When it comes back to life, it will see that the row is still not marked and it publishes it one more time. Therefore the Outbox gives us at-least-once semantics on the sending side, exactly as we learned in the previous lesson. The remaining part has to be handled on the receiving side, and the matching pattern is called the Inbox.

The Inbox pattern is exactly the mirror image of the Outbox pattern. On the receiving side, the consumer writes the event id it has just received into an `inbox` table, and it does that inside the same transaction as the business update. If that event id already exists, the consumer knows that it has already processed this event, so it skips it and does not run the side effect again. Thanks to the Outbox and Inbox pair, we have a summary sentence that is very easy to remember for interviews: the Outbox takes care of the no-loss half, while the Inbox takes care of the no-duplicate half. Those two halves put together are exactly effectively-once from end to end.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| vế không mất event | the no-loss half |
| chết trước khi kịp đánh dấu | die before it manages to mark |
| khi nó sống lại | when it comes back to life |
| vẫn chưa được đánh dấu | is still not marked |
| đúng như những gì chúng ta đã học | exactly as we learned |
| mẫu tương ứng gọi là | the matching pattern is called |
| hình ảnh đối xứng của | the mirror image of |
| nó bỏ qua và không chạy lại tác dụng phụ | it skips it and does not run the side effect again |
| một câu tóm tắt rất dễ nhớ | a summary sentence that is very easy to remember |
| hai nửa đó ghép lại chính là | those two halves put together are exactly |

**Thuật ngữ cần nhớ**

- mẫu hộp thư đến → **the Inbox pattern**
- hình ảnh đối xứng → **the mirror image**
- không mất và không trùng → **no loss and no duplicates**
- phía phát và phía nhận → **the sending side and the receiving side**
- cặp mẫu bổ trợ nhau → **a complementary pair of patterns**

---

## ⑦ Vận hành bảng outbox

**Tiếng Việt**

Một bảng outbox chạy trong môi trường thật sẽ sinh ra ba việc vận hành mà chúng ta phải chuẩn bị trước. Việc thứ nhất là dọn dẹp, vì bảng này phình lên rất nhanh nếu chúng ta giữ lại mọi dòng đã gửi. Chúng ta nên xoá hoặc chuyển sang kho lưu trữ những dòng đã gửi cũ hơn một khoảng thời gian nhất định. Nếu chúng ta quên việc này, hậu quả là bảng outbox trở thành bảng lớn nhất trong cơ sở dữ liệu và mọi truy vấn quét đều chậm dần. Đây là loại sự cố xuất hiện sau sáu tháng, đúng lúc không ai còn nhớ ai đã dựng cái bảng đó.

Việc thứ hai là giữ thứ tự khi phát event ra broker. Chúng ta phải đọc bảng outbox theo đúng thứ tự ghi, ví dụ theo một cột số thứ tự tăng dần hoặc theo thời điểm tạo. Sau đó chúng ta đặt khoá phân mảnh theo mã thực thể, để mọi event của cùng một đơn hàng vẫn nằm cùng một partition như chúng ta đã học ở Bài 3. Việc thứ ba là giám sát lượng tồn đọng trong bảng outbox, và chúng ta nên coi nó ngang hàng với độ trễ tiêu thụ. Nếu số dòng chưa gửi tăng đều, nghĩa là tiến trình đẩy đang chết hoặc đang chậm, và chúng ta cần biết điều đó trước khi khách hàng biết.

**English (bám cấu trúc tiếng Việt)**

An outbox table running in a real environment creates three operational jobs that we have to prepare for in advance. The first job is cleaning up, because this table grows very fast if we keep every row that has been sent. We should delete or move to an archive the sent rows that are older than a certain period of time. If we forget this, the consequence is that the outbox table becomes the largest table in the database and every scanning query gets slower and slower. This is the kind of incident that shows up after six months, right when nobody remembers who built that table.

The second job is keeping the order when we publish events out to the broker. We have to read the outbox table in exactly the order it was written, for example by an increasing sequence column or by the creation time. After that we set the partition key by the entity id, so that every event of the same order still sits in the same partition as we learned in Lesson 3. The third job is monitoring the backlog in the outbox table, and we should treat it as equal in importance to consumer lag. If the number of unsent rows rises steadily, it means that the publishing process is dying or is running slowly, and we need to know that before the customer knows.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| ba việc vận hành mà chúng ta phải chuẩn bị trước | three operational jobs that we have to prepare for in advance |
| bảng này phình lên rất nhanh | this table grows very fast |
| chuyển sang kho lưu trữ | move to an archive |
| cũ hơn một khoảng thời gian nhất định | older than a certain period of time |
| mọi truy vấn quét đều chậm dần | every scanning query gets slower and slower |
| loại sự cố xuất hiện sau sáu tháng | the kind of incident that shows up after six months |
| đúng lúc không ai còn nhớ | right when nobody remembers |
| theo đúng thứ tự ghi | in exactly the order it was written |
| một cột số thứ tự tăng dần | an increasing sequence column |
| coi nó ngang hàng với | treat it as equal in importance to |
| tiến trình đẩy đang chết hoặc đang chậm | the publishing process is dying or is running slowly |
| trước khi khách hàng biết | before the customer knows |

**Thuật ngữ cần nhớ**

- lượng tồn đọng chưa gửi → **the backlog**
- dọn dẹp bảng → **to clean up the table**
- chuyển vào kho lưu trữ → **to archive**
- cột số thứ tự → **a sequence column**
- sự cố vận hành → **an incident**

---

## ⑧ Vì sao ngành né 2PC và chọn Saga cộng Outbox

**Tiếng Việt**

Câu hỏi kinh điển đi kèm bài này là vì sao chúng ta không dùng giao dịch phân tán hai pha cho gọn. Trong cách đó, một bộ điều phối bắt tất cả các bên chuẩn bị trước, rồi mới ra lệnh cho tất cả cùng commit. Về lý thuyết thì cách đó cho chúng ta nhất quán mạnh xuyên nhiều service, và nghe rất hấp dẫn. Nhưng trong thực tế nó có một điểm yếu chết người, đó là nếu bộ điều phối chết sau bước chuẩn bị thì tất cả các bên đều bị kẹt và giữ khoá. Khi các bên bị kẹt, khả năng phục vụ của cả hệ thống tụt xuống, và thông lượng cũng giảm theo vì mọi thứ đều phải chờ nhau.

Vì vậy ngành đã chuyển sang cách khác, đó là chuỗi giao dịch cục bộ nối với nhau bằng event, tức là Saga, và dùng Outbox để phát event một cách tin cậy. Mỗi service chỉ commit transaction cục bộ của chính nó, nên không có ai khoá toàn cục và không ai kẹt vì một bộ điều phối chết. Đổi lại, chúng ta chấp nhận nhất quán cuối cùng, và chúng ta phải viết các bước bù trừ cho trường hợp một bước ở giữa thất bại. Khi một đồng nghiệp đề xuất dùng hai pha cho chắc, chúng ta nên hỏi lại một câu duy nhất, đó là hệ thống chịu được bao lâu khi các bên bị giữ khoá vì bộ điều phối chết. Câu trả lời hầu như luôn cho thấy Saga cộng Outbox là lựa chọn hợp lý hơn cho microservices.

**English (bám cấu trúc tiếng Việt)**

The classic question that comes with this lesson is why we do not use a two-phase distributed transaction for simplicity. In that approach, a coordinator makes all the participants prepare first, and only then orders all of them to commit together. In theory that approach gives us strong consistency across many services, and it sounds very attractive. But in practice it has one deadly weakness, which is that if the coordinator dies after the prepare step then all the participants are stuck and holding locks. When the participants are stuck, the serving capability of the whole system drops, and the throughput falls as well because everything has to wait for everything else.

Therefore the industry has moved to a different approach, which is a chain of local transactions linked by events, that is, the Saga, and it uses the Outbox to publish events reliably. Each service only commits its own local transaction, so nobody holds a global lock and nobody gets stuck because of a dead coordinator. In exchange, we accept eventual consistency, and we have to write compensating steps for the case where a step in the middle fails. When a colleague proposes using two phases to be safe, we should ask back exactly one question, which is how long the system can survive while the participants are held under lock because the coordinator has died. The answer almost always shows that the Saga plus the Outbox is the more sensible choice for microservices.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| giao dịch phân tán hai pha | a two-phase distributed transaction |
| bắt tất cả các bên chuẩn bị trước | makes all the participants prepare first |
| rồi mới ra lệnh cho tất cả cùng commit | and only then orders all of them to commit together |
| về lý thuyết thì | in theory |
| một điểm yếu chết người | one deadly weakness |
| tất cả các bên đều bị kẹt và giữ khoá | all the participants are stuck and holding locks |
| khả năng phục vụ tụt xuống | the serving capability drops |
| mọi thứ đều phải chờ nhau | everything has to wait for everything else |
| chuỗi giao dịch cục bộ nối với nhau bằng event | a chain of local transactions linked by events |
| không có ai khoá toàn cục | nobody holds a global lock |
| các bước bù trừ | compensating steps |
| hệ thống chịu được bao lâu | how long the system can survive |

**Thuật ngữ cần nhớ**

- giao dịch hai pha → **two-phase commit (2PC)**
- bộ điều phối → **the coordinator**
- các bên tham gia → **the participants**
- giao dịch cục bộ → **a local transaction**
- bước bù trừ → **a compensating step**

---

## ⑨ Mô hình ghi nhớ

**Tiếng Việt**

Chúng ta không ghi vào hai nơi nữa, mà chúng ta ghi dữ liệu nghiệp vụ và event vào bảng outbox trong một transaction cơ sở dữ liệu duy nhất, rồi để một tiến trình quét hoặc một công cụ CDC đẩy chúng đi. Outbox lo phần không mất, Inbox lo phần không trùng, và giao dịch hai pha đã nhường chỗ cho Saga cộng Outbox.

**English (bám cấu trúc tiếng Việt)**

We no longer write into two places, but we write the business data and the event into the outbox table inside one single database transaction, and then we let a polling process or a CDC tool push them out. The Outbox takes care of the no-loss half, the Inbox takes care of the no-duplicate half, and two-phase commit has given way to the Saga plus the Outbox.

---

## ⑩ Bảng thuật ngữ tổng hợp

| Tiếng Việt | Tiếng Anh | Ghi chú phát âm |
|---|---|---|
| ghi kép | the dual-write problem | *dual* /ˈdjuːəl/ — giọng Anh "DYU-ợl" |
| quay lui giao dịch | to roll back | |
| event ma | a ghost event | *ghost* — chữ **h** câm, đọc /ɡəʊst/ |
| các service phía sau | the downstream services | |
| lệch dữ liệu dần | data drift | giọng Anh *data* /ˈdeɪtə/ — "ĐÂY-tơ" |
| kịch bản | a scenario | sə-**NAH**-ri-oh, giọng Anh /səˈnɑːriəʊ/ |
| khối bắt lỗi | a try/catch block | |
| gói tin xác nhận | the acknowledgement packet | ək-**NO**-lij-mənt, không tách rời phần *know* |
| thu hồi lại | to take back | |
| mẫu kiến trúc | an architectural pattern | ar-ki-**TEC**-chə-rəl, chữ *ch* đầu đọc /k/ |
| trường hợp tinh vi | a subtle case | *subtle* /ˈsʌtl/ — chữ **b** câm hoàn toàn |
| mẫu hộp thư gửi có transaction | the Transactional Outbox pattern | *transactional* = tran-**ZAK**-shə-nəl, chữ *s* đọc /z/ |
| dữ liệu nghiệp vụ | the business data | *business* — hai âm tiết "BIZ-nis", chữ *i* giữa câm |
| tính nguyên tử | atomicity | a-tə-**MI**-si-ty, khác *atomic* (ə-**TOM**-ic) |
| dòng dữ liệu trong bảng | a row | /rəʊ/ — vần với *go*, không phải "rao" |
| tiến trình quét và phát | a polling publisher | |
| công việc chạy nền | a background job | |
| mặt trái, nhược điểm | the downside | |
| khoá dòng khi chọn | to lock the row on select | |
| bỏ qua dòng đang khoá | skip locked | |
| bắt thay đổi dữ liệu | change data capture (CDC) | *capture* = **CAP**-chə, đuôi đọc /tʃə/ |
| nhật ký giao dịch | the transaction log | |
| nhật ký ghi trước | the write-ahead log (WAL) | *write* — chữ **w** câm hoàn toàn |
| nhật ký nhị phân | the binary log | **BAI**-nə-ri, âm đầu là /aɪ/ |
| bỏ sót bản cập nhật | to miss an update | |
| đồng bộ dữ liệu | to synchronise data | **SIN**-crə-nais, trọng âm âm đầu |
| mốc thời gian | a timestamp | |
| mẫu hộp thư đến | the Inbox pattern | |
| hình ảnh đối xứng | the mirror image | *mirror* = **MI**-rə, hai âm tiết |
| phía phát và phía nhận | the sending side and the receiving side | |
| cặp mẫu bổ trợ nhau | a complementary pair | com-plə-**MEN**-tə-ri, trọng âm âm thứ ba |
| lượng tồn đọng chưa gửi | the backlog | **BACK**-log, trọng âm âm đầu |
| chuyển vào kho lưu trữ | to archive | **AR**-kaiv, chữ *ch* đọc /k/ |
| cột số thứ tự | a sequence column | *column* — chữ **n** cuối câm, đọc "CO-lơm" |
| sự cố vận hành | an incident | **IN**-si-dənt, trọng âm âm đầu |
| giao dịch hai pha | two-phase commit (2PC) | *phase* /feɪz/ — đuôi đọc /z/ |
| bộ điều phối | the coordinator | co-**OR**-di-nay-tə, năm âm tiết |
| các bên tham gia | the participants | par-**TI**-si-pənts, trọng âm âm thứ hai |
| giao dịch cục bộ | a local transaction | |
| bước bù trừ | a compensating step | **COM**-pen-say-ting, trọng âm âm đầu |
| nhất quán cuối cùng | eventual consistency | i-**VEN**-chu-al, nghĩa là "rốt cuộc" |
| bị kẹt và giữ khoá | stuck and holding locks | |

---

## ⑪ Bài nói

> **Cách luyện:** bật ghi âm, nói **60–90 giây** cho mỗi đề, **không nhìn tài liệu**. Mỗi bài nói phải dùng ít nhất **sáu thuật ngữ** trong bảng trên. Nói xong thì nghe lại một lượt, đánh dấu chỗ mình ngập ngừng, rồi nói lại chính đề đó thêm một lần nữa.

1. Explain to a junior developer what the dual-write problem is, and walk through the four scenarios that can happen when the code inserts a row and then publishes an event.

2. A developer says the code is safe because the insert and the publish are inside the same try/catch, and there is a rollback in the catch. Explain exactly where that argument breaks.

3. Describe how the Transactional Outbox pattern removes the lost event and the ghost event, and describe what it deliberately does not remove.

4. When would you choose a polling publisher over CDC, and when would you choose CDC instead? Give the trade-off you would present to your team.

5. Someone on your team proposes a nightly job that polls the orders table by `updated_at` to sync into the search index. Explain why you would push back and what you would propose instead.

6. Explain how the Inbox pattern complements the Outbox, and explain why we still need it after the Outbox is in place.

7. A colleague argues for a two-phase commit across the order service and the payment service, because eventual consistency feels risky. Explain the case against it, and say what you would offer as the alternative.
