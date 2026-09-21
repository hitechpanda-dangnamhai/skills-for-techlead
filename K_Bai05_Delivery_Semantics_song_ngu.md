# Bài 5 — Delivery Semantics & Idempotent Consumer
### Bản song ngữ: tiếng Anh bám cấu trúc tiếng Việt

> **Cách dùng bài này:** đọc đoạn tiếng Việt trước → tự dịch trong đầu, không nhìn xuống → so sánh với đoạn tiếng Anh ngay bên dưới → **đọc to bản tiếng Anh hai lần**: lần một đọc chậm cho đúng từng âm, lần hai đọc liền mạch như đang nói với người phỏng vấn.

---

## ① Ba mức bảo đảm giao hàng

**Tiếng Việt**

Bài này là bài lõi về độ tin cậy, vì mọi câu hỏi về Outbox, Saga và khả năng chống chịu đều dựa lên nó. Chúng ta bắt đầu bằng ba mức bảo đảm giao hàng, và chúng ta nên định nghĩa được cả ba chỉ trong ba câu ngắn. Mức thứ nhất là giao nhiều nhất một lần, nghĩa là chúng ta gửi đúng một lần rồi thôi, nên message có thể mất nhưng không bao giờ bị trùng. Mức thứ hai là giao ít nhất một lần, nghĩa là chúng ta gửi lại cho tới khi chắc chắn bên kia đã nhận, nên message không mất nhưng có thể bị trùng. Mức thứ ba là giao đúng một lần, nghĩa là mỗi message được xử lý chính xác một lần, không mất và cũng không trùng.

Mức thứ ba nghe hấp dẫn nhất, nhưng chúng ta phải nói thẳng rằng nó gần như bất khả thi khi đi qua biên mạng. Lý do là bên gửi không bao giờ biết chắc gói tin xác nhận có tới nơi hay không, nên nó chỉ có hai lựa chọn là gửi lại hoặc không gửi lại. Gửi lại thì có thể trùng, còn không gửi lại thì có thể mất, và không hề có lựa chọn thứ ba. Vì vậy trong thực tế chúng ta chọn giao ít nhất một lần rồi cộng thêm một lớp chống trùng ở phía nhận. Kết quả của cách ghép đó gọi là hiệu quả một lần, và đó là mức tốt nhất mà một hệ phân tán đạt được.

**English (bám cấu trúc tiếng Việt)**

This lesson is the core lesson about reliability, because every question about the Outbox, the Saga and resilience rests on it. We start with the three levels of delivery guarantee, and we should be able to define all three in only three short sentences. The first level is at-most-once, which means that we send exactly once and then stop, so the message may be lost but is never duplicated. The second level is at-least-once, which means that we send again until we are sure the other side has received it, so the message is not lost but may be duplicated. The third level is exactly-once, which means that each message is processed precisely one time, neither lost nor duplicated.

The third level sounds the most attractive, but we have to say plainly that it is almost impossible across a network boundary. The reason is that the sender never knows for sure whether the acknowledgement packet arrived or not, so it only has two choices, which are to send again or not to send again. Sending again may duplicate, while not sending again may lose, and there is no third choice at all. Therefore in practice we choose at-least-once and then add a deduplication layer on the receiving side. The result of that combination is called effectively-once, and it is the best level that a distributed system can reach.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| bài lõi về độ tin cậy | the core lesson about reliability |
| đều dựa lên nó | rests on it |
| chỉ trong ba câu ngắn | in only three short sentences |
| chúng ta gửi đúng một lần rồi thôi | we send exactly once and then stop |
| cho tới khi chắc chắn bên kia đã nhận | until we are sure the other side has received it |
| không mất và cũng không trùng | neither lost nor duplicated |
| nghe hấp dẫn nhất | sounds the most attractive |
| gần như bất khả thi | almost impossible |
| khi đi qua biên mạng | across a network boundary |
| không hề có lựa chọn thứ ba | there is no third choice at all |
| cộng thêm một lớp chống trùng | add a deduplication layer |
| mức tốt nhất mà một hệ phân tán đạt được | the best level that a distributed system can reach |

**Thuật ngữ cần nhớ**

- mức bảo đảm giao hàng → **the delivery guarantee**
- giao nhiều nhất một lần → **at-most-once**
- giao ít nhất một lần → **at-least-once**
- giao đúng một lần → **exactly-once**
- hiệu quả một lần → **effectively-once**

---

## ② Vì sao consumer bắt buộc phải bất biến khi lặp

**Tiếng Việt**

Vì giao ít nhất một lần là mặc định trong thực tế, chúng ta phải chấp nhận rằng broker sẽ giao lại message. Có ít nhất bốn tình huống làm cho việc đó xảy ra. Tình huống thứ nhất là consumer xử lý quá lâu và hết thời gian chờ. Tình huống thứ hai là nhóm consumer chia lại partition, nên phần việc đang dở bị chuyển sang một máy khác. Tình huống thứ ba là consumer chết trước khi kịp commit. Tình huống thứ tư là chính producer thử lại vì nó không nhận được xác nhận. Trong cả bốn tình huống, phần tác dụng phụ của chúng ta sẽ chạy hai lần.

Vì vậy consumer bắt buộc phải bất biến khi lặp, và đây không phải là một lựa chọn tuỳ thích. Cách làm trong messaging là gắn cho mỗi message một mã sự kiện duy nhất, rồi lưu lại những mã đã xử lý vào một bảng chống trùng. Khi chúng ta thấy lại một mã đã có trong bảng, chúng ta bỏ qua message đó thay vì chạy lại tác dụng phụ. Cách này khác với khoá bất biến trong HTTP, vì ở đó mã nằm trong header của request còn ở đây mã nằm trong chính message. Nếu chúng ta bỏ lớp chống trùng đi, hậu quả rất cụ thể, đó là một lần giao lại là kho bị trừ hai lần và khách bị tính tiền hai lần.

**English (bám cấu trúc tiếng Việt)**

Because at-least-once is the default in practice, we have to accept that the broker will redeliver messages. There are at least four situations that make that happen. The first situation is that the consumer processes for too long and the waiting time runs out. The second situation is that the consumer group reassigns the partitions, so the unfinished work is moved to another machine. The third situation is that the consumer dies before it manages to commit. The fourth situation is that the producer itself retries because it did not receive an acknowledgement. In all four situations, our side-effect part will run twice.

Therefore the consumer must be idempotent, and this is not an optional choice. The way to do it in messaging is to attach a unique event id to each message, and then store the ids we have processed into a deduplication table. When we see again an id that is already in the table, we skip that message instead of running the side effect again. This way differs from the idempotency key in HTTP, because there the key sits in the request header while here the id sits in the message itself. If we remove the deduplication layer, the consequence is very concrete, which is that one redelivery means stock is taken down twice and the customer is charged twice.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| là mặc định trong thực tế | is the default in practice |
| bốn tình huống làm cho việc đó xảy ra | four situations that make that happen |
| hết thời gian chờ | the waiting time runs out |
| phần việc đang dở | the unfinished work |
| trước khi kịp commit | before it manages to commit |
| đây không phải là một lựa chọn tuỳ thích | this is not an optional choice |
| gắn cho mỗi message một mã sự kiện duy nhất | attach a unique event id to each message |
| bảng chống trùng | the deduplication table |
| chúng ta bỏ qua message đó | we skip that message |
| mã nằm trong chính message | the id sits in the message itself |
| hậu quả rất cụ thể | the consequence is very concrete |
| kho bị trừ hai lần | stock is taken down twice |

**Thuật ngữ cần nhớ**

- bất biến khi lặp → **idempotent**
- giao lại → **to redeliver**
- mã sự kiện duy nhất → **a unique event id**
- bảng chống trùng → **a deduplication table**
- tác dụng phụ → **a side effect**

---

## ③ Xác nhận đúng lúc, và cái bẫy thời gian ẩn

**Tiếng Việt**

Quy tắc về thời điểm xác nhận chỉ gồm một câu, đó là chúng ta xác nhận sau khi đã xử lý xong và đã ghi xong tác dụng phụ. Nếu chúng ta xác nhận sớm rồi tiến trình chết ở giữa, message coi như đã xong nhưng công việc thật thì chưa bao giờ chạy. Nếu chúng ta không xác nhận, hoặc chúng ta báo lỗi, broker sẽ giao lại message đó. Ở RabbitMQ, việc báo lỗi đưa message quay lại hàng đợi hoặc đẩy nó sang hàng đợi thư chết, tuỳ theo cấu hình. Ở Kafka, chúng ta không cần làm gì đặc biệt cả, chúng ta chỉ đơn giản là không commit offset. Câu thần chú mà chúng ta nên nhớ là xác nhận nghĩa là tôi đã xử lý xong, chứ không phải tôi đã nhận được.

Có một biến thể của cùng vấn đề này mà chúng ta gặp ở các hàng đợi kiểu SQS. Khi một consumer lấy message ra, broker cho nó một khoảng thời gian độc quyền để xử lý, và người ta gọi khoảng đó là thời gian ẩn. Nếu consumer xử lý lâu hơn khoảng thời gian đó, message sẽ hiện lại cho người khác lấy, dù người đầu tiên vẫn đang làm dở. Khi đó hai worker cùng xử lý một message tại cùng một thời điểm, và mọi giả định về việc chỉ có một người chạy đều sai. Cách chữa gồm ba việc, đó là làm cho phần xử lý bất biến khi lặp, giữ vòng lặp chính thật ngắn, và đẩy các việc nặng sang một luồng khác.

**English (bám cấu trúc tiếng Việt)**

The rule about the moment of acknowledgement consists of one sentence, which is that we acknowledge after we have finished processing and have finished writing the side effect. If we acknowledge early and then the process dies in the middle, the message counts as done but the real work has never run. If we do not acknowledge, or we report a failure, the broker will redeliver that message. In RabbitMQ, reporting a failure puts the message back into the queue or pushes it to the dead-letter queue, depending on the configuration. In Kafka, we do not need to do anything special at all, we simply do not commit the offset. The mantra we should remember is that an acknowledgement means I have finished processing, rather than I have received it.

There is a variant of this same problem that we meet in SQS-style queues. When a consumer takes a message out, the broker gives it an exclusive period of time to process, and people call that period the visibility timeout. If the consumer processes for longer than that period, the message becomes visible again for somebody else to take, even though the first person is still working on it. At that point two workers process one message at the same moment, and every assumption about only one runner being active is wrong. The cure consists of three things, which are to make the processing part idempotent, to keep the main loop really short, and to push the heavy jobs over to another flow.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| thời điểm xác nhận | the moment of acknowledgement |
| message coi như đã xong | the message counts as done |
| công việc thật thì chưa bao giờ chạy | the real work has never run |
| hoặc chúng ta báo lỗi | or we report a failure |
| đưa message quay lại hàng đợi | puts the message back into the queue |
| tuỳ theo cấu hình | depending on the configuration |
| câu thần chú | the mantra |
| một khoảng thời gian độc quyền | an exclusive period of time |
| message sẽ hiện lại cho người khác lấy | the message becomes visible again for somebody else to take |
| dù người đầu tiên vẫn đang làm dở | even though the first person is still working on it |
| mọi giả định ... đều sai | every assumption ... is wrong |
| giữ vòng lặp chính thật ngắn | keep the main loop really short |

**Thuật ngữ cần nhớ**

- xác nhận đã xử lý → **to acknowledge (ack)**
- báo lỗi để trả message về → **to nack**
- thời gian ẩn của message → **the visibility timeout**
- quyền xử lý tạm thời → **a lease**
- hàng đợi thư chết → **the dead-letter queue**

---

## ④ Chống trùng phải nguyên tử cùng với tác dụng phụ

**Tiếng Việt**

Bây giờ chúng ta nói tới chi tiết mà code do AI sinh ra hay sai nhất, đó là tính nguyên tử của lớp chống trùng. Nhiều người viết đúng ý tưởng nhưng lại đặt hai bước vào hai transaction tách rời. Bước một là kiểm tra và ghi mã sự kiện vào bảng đã xử lý, còn bước hai là chạy phần cập nhật nghiệp vụ. Nếu tiến trình chết ở giữa hai bước đó, chúng ta rơi đúng vào cái bẫy mà chúng ta định tránh. Vì vậy chúng ta phải đặt cả hai bước trong cùng một transaction của cơ sở dữ liệu, để hoặc cả hai cùng có, hoặc cả hai cùng không.

Cách viết chuẩn rất gọn nếu chúng ta dùng một cơ sở dữ liệu quan hệ. Chúng ta tạo một bảng có khoá chính là mã sự kiện, rồi chèn mã đó kèm mệnh đề bỏ qua khi trùng. Nếu lệnh chèn thật sự thêm được một dòng thì đây là lần đầu, nên chúng ta chạy phần cập nhật nghiệp vụ ngay trong cùng transaction. Nếu lệnh chèn không thêm được dòng nào thì message này đã được xử lý rồi, nên chúng ta bỏ qua và đi tiếp. Một cách khác cũng hợp lệ là đặt ràng buộc duy nhất ngay trên bảng nghiệp vụ, để chính cơ sở dữ liệu từ chối bản trùng thay vì chúng ta tự kiểm tra bằng tay.

**English (bám cấu trúc tiếng Việt)**

Now we come to the detail that AI-generated code gets wrong most often, which is the atomicity of the deduplication layer. Many people write the idea correctly but put the two steps into two separate transactions. Step one is to check and write the event id into the processed table, while step two is to run the business update. If the process dies between those two steps, we fall exactly into the trap that we intended to avoid. Therefore we have to put both steps inside the same database transaction, so that either both of them exist, or neither of them does.

The standard way of writing it is very short if we use a relational database. We create a table whose primary key is the event id, and then we insert that id together with a clause that skips on conflict. If the insert really adds one row then this is the first time, so we run the business update right inside the same transaction. If the insert adds no row at all then this message has already been processed, so we skip it and move on. Another equally valid way is to put a unique constraint right on the business table, so that the database itself rejects the duplicate instead of us checking by hand.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| tính nguyên tử của lớp chống trùng | the atomicity of the deduplication layer |
| viết đúng ý tưởng nhưng lại đặt | write the idea correctly but put |
| hai transaction tách rời | two separate transactions |
| chúng ta rơi đúng vào cái bẫy | we fall exactly into the trap |
| mà chúng ta định tránh | that we intended to avoid |
| hoặc cả hai cùng có, hoặc cả hai cùng không | either both of them exist, or neither of them does |
| kèm mệnh đề bỏ qua khi trùng | together with a clause that skips on conflict |
| thật sự thêm được một dòng | really adds one row |
| chúng ta bỏ qua và đi tiếp | we skip it and move on |
| một cách khác cũng hợp lệ | another equally valid way |
| ràng buộc duy nhất | a unique constraint |
| thay vì chúng ta tự kiểm tra bằng tay | instead of us checking by hand |

**Thuật ngữ cần nhớ**

- tính nguyên tử → **atomicity**
- bảng lưu event đã xử lý → **the processed-events table**
- ràng buộc duy nhất → **a unique constraint**
- bỏ qua khi trùng khoá → **on conflict do nothing**
- mẫu hộp thư đến → **the Inbox pattern**

---

## ⑤ Thứ tự theo khoá và nghịch lý giữa mở rộng và thứ tự

**Tiếng Việt**

Phần tiếp theo là thứ tự, và câu hỏi đúng luôn là thứ tự trong phạm vi nào. Trong hầu hết các nghiệp vụ, chúng ta không cần thứ tự toàn cục, chúng ta chỉ cần thứ tự cho từng thực thể. Ở Kafka, chúng ta đạt được điều đó bằng cách định tuyến theo khoá, vì cùng một khoá luôn rơi vào cùng một partition. Ở RabbitMQ, chúng ta dùng exchange băm nhất quán, hoặc chúng ta dựng một hàng đợi riêng cho mỗi khoá. Ở SQS kiểu FIFO, chúng ta dùng mã nhóm message để giữ thứ tự bên trong từng nhóm.

Từ đây sinh ra một nghịch lý mà người phỏng vấn rất thích hỏi. Khi độ trễ tiêu thụ tăng, phản xạ của chúng ta là thêm consumer và tăng số partition cho nhanh hơn. Nhưng càng nhiều luồng chạy song song thì thứ tự toàn cục càng vỡ, và khách hàng bắt đầu thấy trạng thái đơn hàng nhảy lung tung. Cách dung hoà là chấp nhận rằng thứ tự chỉ cần đúng bên trong một khoá, còn giữa các khoá khác nhau thì chúng ta cho chạy song song thoải mái. Nói cách khác, chúng ta không chọn giữa thứ tự và tốc độ, mà chúng ta chọn đúng phạm vi cho thứ tự.

**English (bám cấu trúc tiếng Việt)**

The next part is ordering, and the right question is always ordering within which scope. In most business domains, we do not need global ordering, we only need ordering for each entity. In Kafka, we achieve that by routing on the key, because the same key always falls into the same partition. In RabbitMQ, we use a consistent-hash exchange, or we set up a separate queue for each key. In FIFO-style SQS, we use the message group id to keep the ordering inside each group.

Out of this comes a paradox that interviewers really like to ask about. When consumer lag rises, our reflex is to add consumers and raise the partition count so that things go faster. But the more threads run in parallel, the more the global ordering breaks, and customers start to see the order status jumping around. The way to reconcile it is to accept that the ordering only has to be right inside one key, while between different keys we let things run in parallel freely. In other words, we do not choose between ordering and speed, but we choose the right scope for the ordering.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| câu hỏi đúng luôn là | the right question is always |
| thứ tự trong phạm vi nào | ordering within which scope |
| trong hầu hết các nghiệp vụ | in most business domains |
| định tuyến theo khoá | routing on the key |
| exchange băm nhất quán | a consistent-hash exchange |
| giữ thứ tự bên trong từng nhóm | keep the ordering inside each group |
| từ đây sinh ra một nghịch lý | out of this comes a paradox |
| phản xạ của chúng ta là | our reflex is |
| càng nhiều luồng chạy song song thì càng vỡ | the more threads run in parallel, the more it breaks |
| trạng thái đơn hàng nhảy lung tung | the order status jumping around |
| cách dung hoà | the way to reconcile it |
| chúng ta chọn đúng phạm vi cho thứ tự | we choose the right scope for the ordering |

**Thuật ngữ cần nhớ**

- thứ tự theo từng thực thể → **per-entity ordering**
- phạm vi của thứ tự → **the ordering scope**
- băm nhất quán → **consistent hashing**
- mã nhóm message → **the message group id**
- nghịch lý giữa hai mục tiêu → **the paradox between the two goals**

---

## ⑥ Message độc và hàng đợi thư chết

**Tiếng Việt**

Message độc là message luôn thất bại dù chúng ta thử lại bao nhiêu lần đi nữa. Nguyên nhân thường là dữ liệu bị hỏng, hoặc một con bug nằm trong chính hàm xử lý. Nếu chúng ta thử lại vô hạn, message đó sẽ kẹt mãi ở đầu hàng và làm nghẽn cả luồng. Vì vậy chúng ta phải đặt một số lần thử lại tối đa, và sau khi vượt ngưỡng thì chúng ta đẩy message sang hàng đợi thư chết. Hàng đợi thư chết không sửa được gì cả, nhưng nó tách message hỏng ra khỏi luồng chính để chúng ta điều tra sau.

Có hai chi tiết nữa mà một người senior luôn nói ra. Chi tiết thứ nhất là các lần thử lại phải giãn ra theo cấp số nhân và phải kèm một chút ngẫu nhiên. Nếu chúng ta thử lại ngay lập tức và đều đặn, chúng ta chỉ đang tự tấn công cái dịch vụ đang ốm của chính mình. Chi tiết thứ hai là chúng ta phải đặt cảnh báo trên số lượng message nằm trong hàng đợi thư chết. Một hàng đợi thư chết mà không ai nhìn thì cũng chỉ là một chỗ để mất dữ liệu một cách lịch sự.

**English (bám cấu trúc tiếng Việt)**

A poison message is a message that always fails no matter how many times we retry. The cause is usually broken data, or a bug sitting in the handler function itself. If we retry infinitely, that message will be stuck at the head of the line forever and will clog the whole flow. Therefore we have to set a maximum number of retries, and after it passes the threshold we push the message over to the dead-letter queue. The dead-letter queue does not fix anything at all, but it separates the broken message out of the main flow so that we can investigate later.

There are two more details that a senior person always says out loud. The first detail is that the retries must spread out exponentially and must come with a bit of randomness. If we retry immediately and at a steady rhythm, we are only attacking our own sick service. The second detail is that we have to set an alert on the number of messages sitting in the dead-letter queue. A dead-letter queue that nobody looks at is only a place to lose data politely.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| dù chúng ta thử lại bao nhiêu lần đi nữa | no matter how many times we retry |
| một con bug nằm trong chính hàm xử lý | a bug sitting in the handler function itself |
| kẹt mãi ở đầu hàng | stuck at the head of the line forever |
| làm nghẽn cả luồng | clog the whole flow |
| sau khi vượt ngưỡng | after it passes the threshold |
| không sửa được gì cả | does not fix anything at all |
| tách message hỏng ra khỏi luồng chính | separates the broken message out of the main flow |
| giãn ra theo cấp số nhân | spread out exponentially |
| kèm một chút ngẫu nhiên | with a bit of randomness |
| tự tấn công cái dịch vụ đang ốm của chính mình | attacking our own sick service |
| mà không ai nhìn | that nobody looks at |
| mất dữ liệu một cách lịch sự | lose data politely |

**Thuật ngữ cần nhớ**

- message luôn lỗi → **a poison message**
- giãn thời gian thử lại theo cấp số nhân → **exponential backoff**
- thêm ngẫu nhiên vào khoảng chờ → **jitter**
- số lần thử tối đa → **the retry limit**
- cảnh báo vận hành → **an operational alert**

---

## ⑦ Chặn đầu hàng và topic thử lại

**Tiếng Việt**

Trên Kafka có một cái bẫy mà rất nhiều đội vấp phải, đó là chặn đầu hàng. Nguyên nhân là mỗi partition được xử lý một cách tuần tự bởi đúng một consumer. Nếu chúng ta thử lại ngay tại chỗ và ngồi chờ một message khó, mọi message nằm sau nó trong cùng partition đều phải chờ theo. Một message hỏng duy nhất có thể làm đứng cả một partition trong nhiều giờ, trong khi các partition khác vẫn chạy bình thường. Vì vậy chúng ta không nên thử lại tại chỗ với thời gian chờ dài trên một broker kiểu nhật ký.

Cách làm đúng là đẩy message lỗi sang một topic thử lại riêng, rồi commit offset của message đó ở luồng chính. Một tiến trình khác sẽ đọc topic thử lại đó và xử lý muộn hơn, với thời gian chờ tăng dần qua từng bậc. Nhờ vậy luồng chính không bao giờ bị chặn, còn message khó thì vẫn có cơ hội thành công. Ngoài ra chúng ta phải phân biệt lỗi tạm thời với lỗi vĩnh viễn, vì hai loại này cần cách xử lý khác nhau. Lỗi tạm thời như mất kết nối thì đáng để thử lại, còn lỗi vĩnh viễn như dữ liệu sai định dạng thì nên đi thẳng ra hàng đợi thư chết.

**English (bám cấu trúc tiếng Việt)**

On Kafka there is a trap that very many teams fall into, which is head-of-line blocking. The cause is that each partition is processed sequentially by exactly one consumer. If we retry in place and sit waiting for one difficult message, every message behind it in the same partition has to wait as well. One single broken message can stall a whole partition for many hours, while the other partitions still run normally. Therefore we should not retry in place with long waits on a log-style broker.

The right way is to push the failed message over to a separate retry topic, and then commit the offset of that message in the main flow. Another process will read that retry topic and handle it later, with the waiting time growing step by step. Thanks to that, the main flow is never blocked, while the difficult message still has a chance to succeed. Besides, we have to distinguish a transient failure from a permanent failure, because these two kinds need different handling. A transient failure such as a lost connection is worth retrying, while a permanent failure such as badly formatted data should go straight to the dead-letter queue.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| một cái bẫy mà rất nhiều đội vấp phải | a trap that very many teams fall into |
| chặn đầu hàng | head-of-line blocking |
| được xử lý một cách tuần tự | is processed sequentially |
| thử lại ngay tại chỗ | retry in place |
| mọi message nằm sau nó | every message behind it |
| làm đứng cả một partition | stall a whole partition |
| với thời gian chờ tăng dần qua từng bậc | with the waiting time growing step by step |
| luồng chính không bao giờ bị chặn | the main flow is never blocked |
| vẫn có cơ hội thành công | still has a chance to succeed |
| phân biệt A với B | distinguish A from B |
| đáng để thử lại | worth retrying |
| dữ liệu sai định dạng | badly formatted data |
| đi thẳng ra hàng đợi thư chết | go straight to the dead-letter queue |

**Thuật ngữ cần nhớ**

- chặn đầu hàng → **head-of-line blocking**
- xử lý tuần tự → **sequential processing**
- topic dành cho việc thử lại → **a retry topic**
- lỗi tạm thời → **a transient failure**
- lỗi vĩnh viễn → **a permanent failure**

---

## ⑧ Hiệu quả một lần trên toàn tuyến

**Tiếng Việt**

Cuối cùng chúng ta ghép mọi thứ lại để trả lời câu hỏi khó nhất, đó là làm sao đạt hiệu quả một lần trên toàn tuyến. Câu trả lời gồm ba mảnh, và không mảnh nào là một cái công tắc thần kỳ. Mảnh thứ nhất nằm ở phía producer, nơi chúng ta dùng producer bất biến khi lặp hoặc mẫu Outbox để việc phát event không mất và không trùng. Mảnh thứ hai nằm ở broker, nơi chúng ta cấu hình đủ bền và chấp nhận ngữ nghĩa giao ít nhất một lần. Mảnh thứ ba nằm ở consumer, nơi chúng ta chống trùng theo mã sự kiện và ghi vào cơ sở dữ liệu một cách nguyên tử cùng với việc lưu dấu đã xử lý.

Khi ai đó hỏi có cấu hình nào bật một phát là được exactly-once hay không, chúng ta nên trả lời theo hai bước. Bước một, chúng ta thừa nhận rằng có những cơ chế mang tên đó, nhưng chúng chỉ có hiệu lực bên trong một hệ thống duy nhất. Bước hai, chúng ta chỉ ra rằng ngay khi có một tác dụng phụ đi ra bên ngoài, ví dụ một lần ghi cơ sở dữ liệu hoặc một lời gọi API, bảo đảm đó hết hiệu lực. Vì vậy hiệu quả một lần là một chuỗi cơ chế mà chúng ta tự dựng, chứ không phải một lá cờ nằm trong tệp cấu hình. Nói được điều này chính là dấu hiệu rõ nhất phân biệt một người senior với một người vừa đọc xong tài liệu.

**English (bám cấu trúc tiếng Việt)**

Finally we put everything together to answer the hardest question, which is how to achieve effectively-once end to end. The answer consists of three pieces, and none of the pieces is a magic switch. The first piece sits on the producer side, where we use an idempotent producer or the Outbox pattern so that publishing events is neither lost nor duplicated. The second piece sits at the broker, where we configure it durably enough and accept at-least-once semantics. The third piece sits at the consumer, where we deduplicate by event id and write into the database atomically together with recording the processed mark.

When somebody asks whether there is a setting we can switch on once to get exactly-once, we should answer in two steps. Step one, we admit that there are mechanisms carrying that name, but they are only in force inside one single system. Step two, we point out that as soon as there is a side effect going outside, for example one database write or one API call, that guarantee stops being in force. Therefore effectively-once is a chain of mechanisms that we build ourselves, rather than a flag sitting in a configuration file. Being able to say this is exactly the clearest sign that separates a senior person from a person who has just finished reading the documentation.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| ghép mọi thứ lại | put everything together |
| trên toàn tuyến | end to end |
| không mảnh nào là một cái công tắc thần kỳ | none of the pieces is a magic switch |
| việc phát event không mất và không trùng | publishing events is neither lost nor duplicated |
| cấu hình đủ bền | configure it durably enough |
| lưu dấu đã xử lý | recording the processed mark |
| bật một phát là được | switch on once to get |
| chúng ta thừa nhận rằng | we admit that |
| chỉ có hiệu lực bên trong một hệ thống duy nhất | only in force inside one single system |
| ngay khi có một tác dụng phụ đi ra bên ngoài | as soon as there is a side effect going outside |
| bảo đảm đó hết hiệu lực | that guarantee stops being in force |
| một lá cờ nằm trong tệp cấu hình | a flag sitting in a configuration file |

**Thuật ngữ cần nhớ**

- trên toàn tuyến → **end to end**
- công tắc thần kỳ → **a magic switch**
- chuỗi cơ chế → **a chain of mechanisms**
- mẫu hộp thư gửi → **the Outbox pattern**
- dấu đã xử lý → **the processed mark**

---

## ⑨ Mô hình ghi nhớ

**Tiếng Việt**

Mặc định là giao ít nhất một lần, nên consumer bắt buộc phải bất biến khi lặp bằng cách chống trùng theo mã sự kiện một cách nguyên tử, và chúng ta luôn xác nhận sau khi xử lý xong. Thứ tự chỉ cần đúng bên trong một khoá, việc thử lại nên đi qua một topic riêng, message luôn hỏng thì ra hàng đợi thư chết, và hiệu quả một lần là một chuỗi cơ chế chứ không phải một lá cờ.

**English (bám cấu trúc tiếng Việt)**

The default is at-least-once, so the consumer must be idempotent by deduplicating on the event id atomically, and we always acknowledge after we have finished processing. Ordering only has to be right inside one key, retries should go through a separate topic, a message that always breaks goes to the dead-letter queue, and effectively-once is a chain of mechanisms rather than a flag.

---

## ⑩ Bảng thuật ngữ tổng hợp

| Tiếng Việt | Tiếng Anh | Ghi chú phát âm |
|---|---|---|
| mức bảo đảm giao hàng | delivery guarantee | *guarantee* = ga-ran-**TEE**, trọng âm âm cuối |
| ngữ nghĩa giao hàng | delivery semantics | si-**MAN**-tics, trọng âm âm giữa |
| giao nhiều nhất một lần | at-most-once | |
| giao ít nhất một lần | at-least-once | |
| giao đúng một lần | exactly-once | |
| hiệu quả một lần | effectively-once | |
| độ tin cậy | reliability | ri-lai-ə-**BI**-li-ty, trọng âm áp chót |
| biên mạng | network boundary | *boundary* = **BOUN**-də-ri, trọng âm âm đầu |
| bất biến khi lặp | idempotent | i-**DEM**-po-tent, trọng âm âm thứ hai |
| giao lại | to redeliver | ri-di-**LI**-və |
| bản trùng | a duplicate | danh từ **DU**-pli-kət; động từ **DU**-pli-kate |
| chống trùng | deduplication | di-du-pli-**KEY**-shn, năm âm tiết |
| mã sự kiện duy nhất | a unique event id | *unique* = yu-**NEEK**, không đọc "iu-ních" |
| tác dụng phụ | a side effect | |
| xác nhận đã xử lý | to acknowledge (ack) | ək-**NO**-lij — **không** tách rời phần *know* |
| trả message về vì lỗi | to nack | đọc /næk/, ghép từ *negative acknowledgement* |
| thời gian ẩn của message | visibility timeout | vi-zi-**BI**-li-ty, trọng âm áp chót |
| quyền xử lý tạm thời | a lease | /liːs/ — "LIS", đuôi /s/ chứ không phải /z/ |
| tính nguyên tử | atomicity | a-tə-**MI**-si-ty — trọng âm âm thứ ba, khác *atomic* (ə-**TOM**-ic) |
| giao dịch cơ sở dữ liệu | a database transaction | *transaction* = tran-**ZAK**-shn, chữ *s* đọc /z/ |
| ràng buộc duy nhất | a unique constraint | *constraint* = kən-**STRAINT**, cụm *-nstr-* phải bật rõ |
| bỏ qua khi trùng khoá | on conflict do nothing | |
| mẫu hộp thư đến | the Inbox pattern | |
| mẫu hộp thư gửi | the Outbox pattern | |
| thứ tự theo từng thực thể | per-entity ordering | *entity* = **EN**-ti-ty |
| phạm vi của thứ tự | the ordering scope | |
| băm nhất quán | consistent hashing | |
| mã nhóm message | the message group id | |
| nghịch lý | a paradox | **PA**-ra-dox, trọng âm âm đầu |
| message luôn lỗi | a poison message | *poison* /ˈpɔɪzn/ — "POI-zợn", hai âm tiết |
| hàng đợi thư chết | the dead-letter queue (DLQ) | *queue* /kjuː/ — đọc y hệt chữ cái **Q** |
| giãn thời gian thử lại theo cấp số nhân | exponential backoff | ek-spo-**NEN**-shl, trọng âm âm thứ ba |
| thêm ngẫu nhiên vào khoảng chờ | jitter | **JI**-tơ, chữ *j* đọc /dʒ/ |
| ngưỡng | threshold | âm *th* /θ/, đọc "THRESH-hold" |
| cảnh báo vận hành | an operational alert | *alert* = ə-**LERT**, trọng âm âm sau |
| chặn đầu hàng | head-of-line blocking | |
| xử lý tuần tự | sequential processing | si-**KWEN**-shl, trọng âm âm giữa |
| topic dành cho việc thử lại | a retry topic | *retry* = ri-**TRAI**, trọng âm âm sau |
| lỗi tạm thời | a transient failure | **TRAN**-zi-ənt, trọng âm âm đầu, chữ *s* đọc /z/ |
| lỗi vĩnh viễn | a permanent failure | **PER**-mə-nənt, trọng âm âm đầu |
| trên toàn tuyến | end to end | |
| chuỗi cơ chế | a chain of mechanisms | *mechanism* = **ME**-cə-ni-zm, chữ *ch* đọc /k/ |

---

## ⑪ Bài nói

> **Cách luyện:** bật ghi âm, nói **60–90 giây** cho mỗi đề, **không nhìn tài liệu**. Mỗi bài nói phải dùng ít nhất **sáu thuật ngữ** trong bảng trên. Nói xong thì nghe lại một lượt, đánh dấu chỗ mình ngập ngừng, rồi nói lại chính đề đó thêm một lần nữa.

1. Explain to a junior developer the three delivery guarantees, and explain why the industry settled on at-least-once plus deduplication rather than chasing exactly-once.

2. A colleague says the consumer does not need deduplication because the handler only sets a flag to true, so running it twice changes nothing. Explain where that reasoning holds and where it breaks.

3. Someone on your team wants to acknowledge the message as soon as it is received, so that the broker sees the consumer as fast and healthy. Explain why you would push back.

4. Describe exactly what goes wrong when the deduplication check and the business update sit in two separate transactions, and describe how you would write it instead.

5. The team added consumers to bring lag down, and now customers report that order statuses appear out of sequence. Describe the paradox and describe the fix you would propose.

6. Describe what head-of-line blocking is on Kafka, and walk through how a retry topic changes the behaviour when one message keeps failing.

7. When would you send a message straight to the dead-letter queue instead of retrying it, and what would you put in place so that the dead-letter queue does not become a quiet data loss?
