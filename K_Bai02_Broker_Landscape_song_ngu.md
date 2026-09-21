# Bài 2 — Broker Landscape & chọn broker
### Bản song ngữ: tiếng Anh bám cấu trúc tiếng Việt

> **Cách dùng bài này:** đọc đoạn tiếng Việt trước → tự dịch trong đầu, không nhìn xuống → so sánh với đoạn tiếng Anh ngay bên dưới → **đọc to bản tiếng Anh hai lần**: lần một đọc chậm cho đúng từng âm, lần hai đọc liền mạch như đang nói với người phỏng vấn.

---

## ① Hai mô hình lưu trữ: nhật ký và hàng đợi

**Tiếng Việt**

Bài trước trả lời câu hỏi vì sao chúng ta dùng bất đồng bộ, còn bài này trả lời câu hỏi chúng ta dùng cái gì để làm bất đồng bộ. Điểm đầu tiên chúng ta phải nắm là trên thị trường có hai mô hình lưu trữ khác nhau, chứ không phải hai thương hiệu khác nhau. Mô hình thứ nhất là hàng đợi truyền thống, và RabbitMQ là đại diện tiêu biểu. Chúng ta hình dung nó như một quầy phát thư: message tới quầy, consumer lấy đi, và broker xoá message ngay sau khi consumer báo là đã nhận xong. Đọc xong là mất, nên không ai đọc lại được nữa. Mô hình thứ hai là nhật ký ghi nối đuôi, và Kafka là đại diện tiêu biểu. Chúng ta hình dung nó như một cuộn băng ghi âm: message được ghi nối vào cuối, broker giữ lại theo thời hạn lưu trữ đã cấu hình, ví dụ bảy ngày, và bản thân consumer tự nhớ mình đã đọc tới đâu bằng một con số gọi là offset.

Sự khác nhau về lưu trữ này kéo theo mọi thứ còn lại. Vì Kafka giữ message lại, nhiều nhóm consumer có thể đọc cùng một topic một cách độc lập, và mỗi nhóm giữ offset riêng của mình. Vì offset nằm ở phía consumer, chúng ta có thể tua lại và xử lý lại toàn bộ lịch sử khi một service mới ra đời hoặc khi chúng ta vừa sửa một con bug. Vì RabbitMQ xoá message sau khi nhận được xác nhận, chúng ta không tua lại được, nhưng đổi lại broker biết chính xác message nào còn chưa xử lý xong. Nếu chúng ta chọn nhầm mô hình thì cái giá rất đắt: chọn Kafka cho một hàng đợi công việc thì chúng ta phải tự dựng lại phần độ ưu tiên và phần thử lại, còn chọn RabbitMQ cho phân tích dữ liệu thì chúng ta mất hẳn khả năng tua lại lịch sử.

**English (bám cấu trúc tiếng Việt)**

The previous lesson answered the question of why we use asynchronous work, while this lesson answers the question of what we use to do asynchronous work. The first point we must grasp is that in the market there are two different storage models, rather than two different brands. The first model is the traditional queue, and RabbitMQ is the typical representative. We picture it as a post counter: the message arrives at the counter, the consumer takes it away, and the broker deletes the message right after the consumer reports that it has finished receiving it. Once it has been read it is gone, so nobody can read it again. The second model is the append-only log, and Kafka is the typical representative. We picture it as a recording tape: the message is appended to the end, the broker keeps it according to the retention period we configured, for example seven days, and the consumer itself remembers how far it has read by a number called the offset.

This difference in storage drags everything else along with it. Because Kafka keeps the messages, many consumer groups can read the same topic independently, and each group holds its own offset. Because the offset sits on the consumer side, we can rewind and reprocess the whole history when a new service is born or when we have just fixed a bug. Because RabbitMQ deletes the message after it receives the acknowledgement, we cannot rewind, but in exchange the broker knows exactly which message is not finished yet. If we choose the wrong model then the price is very high: if we choose Kafka for a work queue then we have to rebuild the priority part and the retry part ourselves, and if we choose RabbitMQ for data analytics then we completely lose the ability to rewind the history.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| điểm đầu tiên chúng ta phải nắm | the first point we must grasp |
| chứ không phải hai thương hiệu khác nhau | rather than two different brands |
| đại diện tiêu biểu | the typical representative |
| chúng ta hình dung nó như | we picture it as |
| báo là đã nhận xong | reports that it has finished receiving it |
| đọc xong là mất | once it has been read it is gone |
| ghi nối vào cuối | appended to the end |
| thời hạn lưu trữ đã cấu hình | the retention period we configured |
| tự nhớ mình đã đọc tới đâu | remembers how far it has read |
| kéo theo mọi thứ còn lại | drags everything else along with it |
| tua lại và xử lý lại | rewind and reprocess |
| cái giá rất đắt | the price is very high |
| tự dựng lại phần độ ưu tiên | rebuild the priority part ourselves |

**Thuật ngữ cần nhớ**

- nhật ký ghi nối đuôi → **append-only log**
- vị trí đọc → **the offset**
- thời hạn lưu trữ → **the retention period**
- nhóm consumer → **a consumer group**
- tua lại và xử lý lại → **to replay** / **to reprocess**
- xác nhận đã xử lý → **an acknowledgement (ack)**

---

## ② Exchange và định tuyến trong RabbitMQ

**Tiếng Việt**

Điểm mạnh lớn nhất của RabbitMQ nằm ở khả năng định tuyến, và cơ chế cụ thể của nó là exchange. Producer không gửi thẳng message vào một queue, mà gửi vào một exchange. Exchange nhìn vào kiểu của chính nó và nhìn vào các binding đã được khai báo, rồi nó quyết định đưa message vào không queue nào, một queue, hoặc nhiều queue cùng lúc. Chìa khoá để quyết định là routing key, tức là một chuỗi ngắn mà producer gắn kèm theo mỗi message. Nhờ cách tách này, producer không cần biết có bao nhiêu queue đang tồn tại, và chúng ta thêm một người nhận mới chỉ bằng cách khai báo thêm một binding.

Chúng ta cần thuộc bốn kiểu exchange, vì người phỏng vấn hỏi câu này rất thường xuyên. Kiểu fanout đưa message tới mọi queue đã bind và bỏ qua routing key hoàn toàn. Kiểu direct chỉ đưa message tới queue nào có binding key khớp chính xác với routing key. Kiểu topic cho phép khớp theo ký tự đại diện, ví dụ mẫu `order.*.created` sẽ khớp với `order.vn.created`. Kiểu headers không nhìn routing key mà khớp theo các header đính kèm message. Nếu chúng ta bỏ qua exchange và cho producer gửi thẳng vào từng queue, hậu quả là producer phải biết toàn bộ danh sách người nhận, và chúng ta quay lại đúng kiểu coupling mà bất đồng bộ đáng lẽ phải gỡ.

**English (bám cấu trúc tiếng Việt)**

The biggest strength of RabbitMQ lies in its routing ability, and its concrete mechanism is the exchange. The producer does not send the message straight into a queue, but sends it into an exchange. The exchange looks at its own type and looks at the bindings that have been declared, then it decides to put the message into no queue, one queue, or many queues at the same time. The key for that decision is the routing key, that is, a short string that the producer attaches to every message. Thanks to this separation, the producer does not need to know how many queues exist, and we add a new receiver only by declaring one more binding.

We need to know the four exchange types by heart, because interviewers ask this question very often. The fanout type puts the message into every queue that has been bound and ignores the routing key completely. The direct type only puts the message into the queue whose binding key matches the routing key exactly. The topic type allows matching by wildcard, for example the pattern `order.*.created` will match `order.vn.created`. The headers type does not look at the routing key but matches according to the headers attached to the message. If we skip the exchange and let the producer send straight into each queue, the consequence is that the producer has to know the whole list of receivers, and we come back to exactly the kind of coupling that asynchronous work was supposed to remove.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| nằm ở khả năng định tuyến | lies in its routing ability |
| không gửi thẳng vào | does not send it straight into |
| các binding đã được khai báo | the bindings that have been declared |
| đưa message vào không queue nào | put the message into no queue |
| chìa khoá để quyết định | the key for that decision |
| một chuỗi ngắn gắn kèm theo mỗi message | a short string attached to every message |
| nhờ cách tách này | thanks to this separation |
| chỉ bằng cách khai báo thêm một binding | only by declaring one more binding |
| khớp chính xác với | matches ... exactly |
| khớp theo ký tự đại diện | matching by wildcard |
| toàn bộ danh sách người nhận | the whole list of receivers |
| đáng lẽ phải gỡ | was supposed to remove |

**Thuật ngữ cần nhớ**

- bộ định tuyến message → **an exchange**
- khoá định tuyến → **the routing key**
- ràng buộc queue vào exchange → **a binding**
- ký tự đại diện → **a wildcard**
- phát tới mọi queue → **fanout**

---

## ③ Đẩy và kéo: ai là bên chủ động

**Tiếng Việt**

Hai mô hình còn khác nhau ở chỗ ai là bên chủ động chuyển message. RabbitMQ đi theo kiểu đẩy, nghĩa là broker chủ động đẩy message xuống consumer ngay khi có message. Cách này cho độ trễ thấp, vì message không phải nằm chờ một vòng hỏi nào cả. Nhưng nó mang một rủi ro rất rõ ràng, đó là nếu consumer xử lý chậm hơn tốc độ đẩy thì consumer sẽ bị ngộp và bộ nhớ của nó phình lên. Vì vậy chúng ta phải đặt giới hạn prefetch, tức là số message tối đa được phép đang bay tới consumer mà chưa được xác nhận.

Kafka đi theo kiểu kéo, nghĩa là consumer tự gọi lên broker để lấy message theo nhịp của chính nó. Cách này tạo ra kiểm soát ngược một cách tự nhiên, vì consumer chậm thì đơn giản là gọi thưa hơn và không có ai ép nó cả. Cách này cũng dễ gom lô, vì mỗi lần gọi consumer có thể lấy về một lô lớn thay vì lấy từng message một. Đổi lại, chúng ta trả thêm một nhịp chờ giữa lúc message tới broker và lúc consumer gọi lên hỏi. Khi bị hỏi trong phỏng vấn, chúng ta nên chốt bằng một câu ngắn: đẩy thì được độ trễ, còn kéo thì được sự an toàn dưới tải nặng.

**English (bám cấu trúc tiếng Việt)**

The two models also differ in who is the active side that moves the message. RabbitMQ follows the push style, which means that the broker actively pushes the message down to the consumer as soon as there is a message. This way gives low latency, because the message does not have to sit and wait for any polling round. But it carries a very clear risk, which is that if the consumer processes more slowly than the push rate then the consumer will be overwhelmed and its memory will swell up. Therefore we have to set a prefetch limit, that is, the maximum number of messages that are allowed to be in flight towards the consumer without being acknowledged.

Kafka follows the pull style, which means that the consumer calls up to the broker itself to fetch messages at its own pace. This way creates backpressure naturally, because a slow consumer simply calls less often and nobody forces it. This way is also easy to batch, because on each call the consumer can fetch back a large batch instead of fetching one message at a time. In exchange, we pay one extra beat of waiting between the moment the message arrives at the broker and the moment the consumer calls up to ask. When we are asked in an interview, we should close with one short sentence: push buys us latency, while pull buys us safety under heavy load.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| ai là bên chủ động | who is the active side |
| ngay khi có message | as soon as there is a message |
| nằm chờ một vòng hỏi | sit and wait for a polling round |
| nó mang một rủi ro rất rõ ràng | it carries a very clear risk |
| consumer sẽ bị ngộp | the consumer will be overwhelmed |
| bộ nhớ của nó phình lên | its memory will swell up |
| đang bay tới consumer mà chưa được xác nhận | in flight towards the consumer without being acknowledged |
| theo nhịp của chính nó | at its own pace |
| gọi thưa hơn | calls less often |
| không có ai ép nó cả | nobody forces it |
| lấy từng message một | fetching one message at a time |
| trả thêm một nhịp chờ | pay one extra beat of waiting |
| đẩy thì được độ trễ | push buys us latency |

**Thuật ngữ cần nhớ**

- kiểu đẩy và kiểu kéo → **push style and pull style**
- kiểm soát ngược → **backpressure**
- giới hạn số message chưa xác nhận → **the prefetch limit**
- đang bay, chưa xử lý xong → **in flight**
- gom lô → **to batch**

---

## ④ SQS, SNS và mẫu phát tán có vùng đệm riêng

**Tiếng Việt**

Nếu hệ thống của chúng ta chạy trên AWS và chúng ta muốn vận hành ít nhất có thể, hai dịch vụ cần biết là SQS và SNS. SQS bản chuẩn là một hàng đợi có thông lượng rất cao, nó bảo đảm giao ít nhất một lần và nó chỉ cố giữ thứ tự chứ không hứa hẹn gì về thứ tự. SQS bản FIFO thì giữ đúng thứ tự và có khử trùng lặp dựa trên mã nhóm message cùng mã khử trùng lặp, nhưng đổi lại thông lượng của nó bị giới hạn. SNS không phải là hàng đợi mà là một kênh xuất bản, nó phát một message tới nhiều bên đăng ký cùng lúc, ví dụ tới SQS, tới Lambda, hoặc tới một endpoint HTTP. Chúng ta nên nhớ rằng SNS lo phần phát tán còn SQS lo phần giữ hàng, và hai việc đó không phải là một.

Từ đó sinh ra một mẫu kiến trúc kinh điển mà chúng ta nên gọi tên được ngay trong phỏng vấn, đó là SNS phát ra rồi nhiều SQS cùng nhận. Mỗi consumer có một hàng đợi riêng của mình, nên nó có vùng đệm riêng và nhịp xử lý riêng. Nhờ đó một consumer chậm không kéo các consumer khác chậm theo, và một consumer chết cũng không làm mất message của những bên còn lại. Nếu chúng ta cho cả bốn consumer dùng chung một hàng đợi, hậu quả là chúng tranh nhau message và mỗi message chỉ có đúng một bên nhận được. Đây là lỗi rất hay gặp khi một người cần phát tán nhưng lại cấu hình theo kiểu chia việc.

**English (bám cấu trúc tiếng Việt)**

If our system runs on AWS and we want to operate as little as possible, the two services we need to know are SQS and SNS. Standard SQS is a queue with very high throughput, it guarantees at-least-once delivery and it only tries to keep the order rather than promising anything about the order. FIFO SQS keeps the exact order and has deduplication based on the message group id together with the deduplication id, but in exchange its throughput is limited. SNS is not a queue but a publishing channel, it broadcasts one message to many subscribers at the same time, for example to SQS, to Lambda, or to an HTTP endpoint. We should remember that SNS takes care of the fan-out part while SQS takes care of the holding part, and those two jobs are not the same one.

Out of that comes a classic architectural pattern that we should be able to name straight away in an interview, which is SNS publishing out and then many SQS queues receiving. Each consumer has a queue of its own, so it has its own buffer and its own processing pace. Thanks to that, one slow consumer does not drag the other consumers into being slow, and one dead consumer does not lose the messages of the remaining sides either. If we let all four consumers share one queue, the consequence is that they compete for the messages and each message is received by exactly one side. This is a very common mistake when somebody needs fan-out but configures it in the work-splitting style.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| vận hành ít nhất có thể | operate as little as possible |
| nó chỉ cố giữ thứ tự | it only tries to keep the order |
| không hứa hẹn gì về thứ tự | rather than promising anything about the order |
| khử trùng lặp | deduplication |
| dựa trên mã nhóm message | based on the message group id |
| lo phần phát tán | takes care of the fan-out part |
| hai việc đó không phải là một | those two jobs are not the same one |
| từ đó sinh ra | out of that comes |
| gọi tên được ngay | be able to name straight away |
| một hàng đợi riêng của mình | a queue of its own |
| không kéo các consumer khác chậm theo | does not drag the other consumers into being slow |
| chúng tranh nhau message | they compete for the messages |
| cấu hình theo kiểu chia việc | configures it in the work-splitting style |

**Thuật ngữ cần nhớ**

- giao ít nhất một lần → **at-least-once delivery**
- khử trùng lặp → **deduplication**
- vùng đệm riêng → **its own buffer**
- bên đăng ký nhận → **a subscriber**
- dịch vụ do nhà cung cấp vận hành → **a managed service**

---

## ⑤ Độ bền: ghi đĩa, nhân bản và xác nhận

**Tiếng Việt**

Bây giờ chúng ta nói về độ bền, vì đây là ranh giới giữa một broker chạy được và một broker dám tin. Một broker bền phải làm ba việc. Việc thứ nhất là ghi message xuống đĩa thay vì chỉ giữ nó trong bộ nhớ. Việc thứ hai là nhân bản message sang nhiều node trước khi coi là đã nhận, vì một bản duy nhất trên một đĩa duy nhất vẫn mất khi máy đó chết. Việc thứ ba là trả một xác nhận về cho producer, để producer biết chắc broker đã thật sự giữ message chứ không phải chỉ vừa nhận vào bộ đệm.

Mỗi broker có tên gọi riêng cho các cơ chế này, và chúng ta nên nói đúng tên khi trả lời phỏng vấn. Ở Kafka, chúng ta đặt `acks` bằng `all`, nghĩa là producer chờ tới khi tất cả các bản sao đang đồng bộ đều đã ghi xong. Ở RabbitMQ, cơ chế tương ứng gọi là publisher confirms, và chúng ta còn phải đánh dấu message là persistent để nó thật sự được ghi xuống đĩa. Nếu chúng ta thiếu các cơ chế này, hậu quả là message bay mất một cách âm thầm đúng vào lúc một node chết, và chúng ta thậm chí không biết mình đã mất cái gì. Đây là loại lỗi tệ nhất, vì nó không để lại một dòng log lỗi nào cả.

**English (bám cấu trúc tiếng Việt)**

Now we talk about durability, because this is the line between a broker that runs and a broker we dare to trust. A durable broker must do three things. The first thing is to write the message down to disk instead of only keeping it in memory. The second thing is to replicate the message to several nodes before treating it as received, because a single copy on a single disk is still lost when that machine dies. The third thing is to return an acknowledgement back to the producer, so that the producer knows for sure that the broker has really held the message rather than having just taken it into a buffer.

Each broker has its own name for these mechanisms, and we should say the right name when we answer in an interview. In Kafka, we set `acks` to `all`, which means that the producer waits until all the in-sync replicas have finished writing. In RabbitMQ, the corresponding mechanism is called publisher confirms, and we also have to mark the message as persistent so that it is really written down to disk. If we lack these mechanisms, the consequence is that messages fly away silently at exactly the moment when a node dies, and we do not even know what we have lost. This is the worst kind of failure, because it does not leave a single line of error log behind.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| ranh giới giữa A và B | the line between A and B |
| một broker dám tin | a broker we dare to trust |
| trước khi coi là đã nhận | before treating it as received |
| một bản duy nhất trên một đĩa duy nhất | a single copy on a single disk |
| biết chắc | knows for sure |
| chứ không phải chỉ vừa nhận vào bộ đệm | rather than having just taken it into a buffer |
| các bản sao đang đồng bộ | the in-sync replicas |
| cơ chế tương ứng gọi là | the corresponding mechanism is called |
| đánh dấu message là persistent | mark the message as persistent |
| bay mất một cách âm thầm | fly away silently |
| chúng ta thậm chí không biết mình đã mất cái gì | we do not even know what we have lost |
| không để lại một dòng log lỗi nào cả | does not leave a single line of error log behind |

**Thuật ngữ cần nhớ**

- độ bền dữ liệu → **durability**
- nhân bản sang nhiều node → **to replicate across nodes**
- bản sao đang đồng bộ → **in-sync replica**
- xác nhận từ broker cho producer → **publisher confirms**
- ghi xuống đĩa, không mất khi restart → **persistent**

---

## ⑥ Broker có phải là điểm chết duy nhất không

**Tiếng Việt**

Một câu hỏi rất hay gặp là broker có trở thành điểm chết duy nhất hoặc nút thắt hay không, và câu trả lời trung thực là có thể. Khi chúng ta đặt broker vào giữa mọi luồng, mọi service đều dựa vào nó, nên nếu nó sập thì cả hệ dừng theo. Vì vậy chúng ta phải chạy broker dưới dạng một cụm chứ không phải một máy đơn lẻ. Ở Kafka, chúng ta đặt hệ số nhân bản ít nhất là ba, để hệ vẫn sống khi mất một node. Ở RabbitMQ, cơ chế tương ứng hiện nay là quorum queue, tức là hàng đợi được nhân bản và quyết định theo đa số.

Ngoài chuyện nhân bản, chúng ta cần thêm bốn thứ nữa thì mới ngủ yên được. Thứ nhất là chia dữ liệu thành nhiều partition hoặc nhiều shard, để chúng ta mở rộng ngang thay vì phải mua một máy to hơn. Thứ hai là phía client phải biết thử lại và tự kết nối lại, vì một cụm khoẻ vẫn có những giây chuyển vai trò giữa các node. Thứ ba là mẫu Outbox ở phía producer, mà chúng ta sẽ học ở Bài 6, để event không mất khi broker tạm thời sập. Thứ tư là giám sát độ trễ tiêu thụ và dung lượng đĩa, vì đĩa đầy là nguyên nhân làm sập broker phổ biến nhất trong thực tế.

**English (bám cấu trúc tiếng Việt)**

A very common question is whether the broker becomes a single point of failure or a bottleneck, and the honest answer is that it can. When we put the broker in the middle of every flow, every service leans on it, so if it goes down then the whole system stops with it. Therefore we have to run the broker as a cluster rather than a single machine. In Kafka, we set the replication factor to at least three, so that the system stays alive when we lose one node. In RabbitMQ, the corresponding mechanism nowadays is the quorum queue, that is, a queue that is replicated and decides by majority.

Besides replication, we need four more things before we can sleep well. The first is to split the data into many partitions or many shards, so that we scale horizontally instead of having to buy a bigger machine. The second is that the client side must know how to retry and to reconnect by itself, because even a healthy cluster still has those seconds of handing roles over between nodes. The third is the Outbox pattern on the producer side, which we will learn in Lesson 6, so that events are not lost when the broker is temporarily down. The fourth is to monitor consumer lag and disk usage, because a full disk is the most common cause of a broker going down in practice.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| câu trả lời trung thực là có thể | the honest answer is that it can |
| mọi service đều dựa vào nó | every service leans on it |
| cả hệ dừng theo | the whole system stops with it |
| dưới dạng một cụm chứ không phải một máy đơn lẻ | as a cluster rather than a single machine |
| hệ số nhân bản | the replication factor |
| quyết định theo đa số | decides by majority |
| thì mới ngủ yên được | before we can sleep well |
| mở rộng ngang | scale horizontally |
| những giây chuyển vai trò giữa các node | those seconds of handing roles over between nodes |
| tạm thời sập | temporarily down |
| độ trễ tiêu thụ | consumer lag |
| nguyên nhân phổ biến nhất trong thực tế | the most common cause in practice |

**Thuật ngữ cần nhớ**

- điểm chết duy nhất → **a single point of failure (SPOF)**
- hệ số nhân bản → **the replication factor**
- hàng đợi quyết theo đa số → **a quorum queue**
- kết nối lại → **to reconnect**
- độ trễ tiêu thụ → **consumer lag**

---

## ⑦ Schema tiến hoá: event là một hợp đồng API

**Tiếng Việt**

Điểm cuối cùng về mặt cơ chế là schema tiến hoá, và đây là chỗ nhiều đội trả giá đắt sau một hai năm. Chúng ta phải coi mỗi event là một hợp đồng API, chứ không phải một cục JSON nội bộ muốn sửa thế nào cũng được. Lý do rất đơn giản, đó là chúng ta không biết hết ai đang đọc event của mình, và nếu chúng ta dùng Kafka thì họ còn có thể đọc lại các event cũ từ nhiều tháng trước. Vì vậy chúng ta nên dùng một schema registry để đăng ký và kiểm tra hình dạng của event, kèm theo số phiên bản. Registry sẽ chặn ngay lúc build những thay đổi phá vỡ tương thích, thay vì để chúng nổ ở môi trường thật.

Các quy tắc thực hành thì rất ít và rất dễ nhớ. Khi thêm một trường, chúng ta luôn thêm nó ở dạng tuỳ chọn và kèm một giá trị mặc định. Chúng ta không bao giờ đổi ý nghĩa của một trường đã có, vì consumer cũ vẫn hiểu trường đó theo nghĩa cũ. Phía consumer phải bỏ qua những trường lạ mà nó chưa biết, thay vì ném lỗi và chết. Khi chúng ta buộc phải thay đổi lớn, chúng ta phát hành một phiên bản event mới và cho hai phiên bản chạy song song trong một thời gian. Nếu chúng ta bỏ qua các quy tắc này, hậu quả là một lần deploy nhỏ ở phía producer làm chết bốn consumer cùng lúc vào hai giờ sáng.

**English (bám cấu trúc tiếng Việt)**

The last point about mechanisms is schema evolution, and this is the place where many teams pay a high price after one or two years. We must treat every event as an API contract, rather than an internal blob of JSON that we can change however we want. The reason is very simple, which is that we do not know all the people who are reading our events, and if we use Kafka then they can even read old events from many months ago again. Therefore we should use a schema registry to register and check the shape of the event, together with a version number. The registry will block at build time the changes that break compatibility, instead of letting them blow up in the real environment.

The practical rules are very few and very easy to remember. When we add a field, we always add it as optional and with a default value. We never change the meaning of an existing field, because old consumers still understand that field with the old meaning. The consumer side must ignore the strange fields that it does not know yet, instead of throwing an error and dying. When we are forced to make a big change, we release a new version of the event and let the two versions run side by side for a while. If we skip these rules, the consequence is that one small deploy on the producer side kills four consumers at the same time at two in the morning.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| trả giá đắt sau một hai năm | pay a high price after one or two years |
| muốn sửa thế nào cũng được | that we can change however we want |
| chúng ta không biết hết ai đang đọc | we do not know all the people who are reading |
| kèm theo số phiên bản | together with a version number |
| chặn ngay lúc build | block at build time |
| phá vỡ tương thích | break compatibility |
| để chúng nổ ở môi trường thật | letting them blow up in the real environment |
| ở dạng tuỳ chọn và kèm một giá trị mặc định | as optional and with a default value |
| bỏ qua những trường lạ | ignore the strange fields |
| thay vì ném lỗi và chết | instead of throwing an error and dying |
| cho hai phiên bản chạy song song | let the two versions run side by side |

**Thuật ngữ cần nhớ**

- sự tiến hoá của schema → **schema evolution**
- kho đăng ký schema → **a schema registry**
- tương thích ngược → **backward compatibility**
- trường tuỳ chọn → **an optional field**
- gắn phiên bản cho event → **to version an event**

---

## ⑧ Chọn broker là chọn mô hình, không phải chọn thương hiệu

**Tiếng Việt**

Bây giờ chúng ta gom mọi thứ lại thành một bộ tiêu chí chọn, vì đây chính là câu hỏi thật trong phỏng vấn. Nếu bài toán cần tua lại, cần lịch sử, cần nhiều consumer độc lập, cần thông lượng cực lớn, hoặc cần đưa dữ liệu sang phân tích, chúng ta chọn Kafka. Nếu bài toán cần định tuyến phức tạp, cần độ ưu tiên cho từng message, hoặc chỉ là một hàng đợi công việc thuần tuý, chúng ta chọn RabbitMQ. Nếu bài toán cần xuất bản siêu nhẹ với độ trễ rất thấp, ví dụ ở tầng điều khiển hoặc ở thiết bị biên, chúng ta chọn NATS. Nếu chúng ta đang ở trên AWS và muốn vận hành ít, chúng ta chọn SQS cho phần hàng đợi và SNS cho phần phát tán.

Khi một đồng nghiệp nói rằng cứ chọn Kafka vì ai cũng dùng, chúng ta nên phản biện bằng cách quay lại nhu cầu thật. Giả sử nhu cầu là chạy vài nghìn job render mỗi ngày, có độ ưu tiên và có thử lại sau một khoảng chờ. Kafka không có độ ưu tiên cho từng message, phần chờ và phần thử lại thì chúng ta phải tự dựng bằng các topic phụ, và partition là một đơn vị song song khá thô. Vài nghìn job mỗi ngày cũng chưa đến mức phải nuôi một cụm Kafka. Với nhu cầu đó, RabbitMQ hoặc một thư viện hàng đợi chạy trên Redis là lựa chọn gọn hơn nhiều. Câu chốt mà chúng ta nên nói ra là: chọn broker là chọn mô hình, không phải chọn thương hiệu.

**English (bám cấu trúc tiếng Việt)**

Now we gather everything into one set of selection criteria, because this is the real question in an interview. If the problem needs replay, needs history, needs many independent consumers, needs extremely high throughput, or needs to move data over to analytics, we choose Kafka. If the problem needs complex routing, needs a priority for each message, or is only a pure work queue, we choose RabbitMQ. If the problem needs extremely light publishing with very low latency, for example at the control plane or at edge devices, we choose NATS. If we are on AWS and we want to operate little, we choose SQS for the queue part and SNS for the fan-out part.

When a colleague says that we should just choose Kafka because everybody uses it, we should push back by going back to the real requirement. Suppose the requirement is to run a few thousand rendering jobs per day, with priority and with a retry after a waiting period. Kafka does not have a priority for each message, the waiting part and the retry part are things we have to build ourselves with extra topics, and a partition is quite a coarse unit of parallelism. A few thousand jobs per day is also not yet at the level where we must feed a Kafka cluster. For that requirement, RabbitMQ or a queue library running on Redis is a far tidier choice. The closing sentence we should say out loud is this: choosing a broker is choosing a model, not choosing a brand.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| gom mọi thứ lại thành một bộ tiêu chí chọn | gather everything into one set of selection criteria |
| đưa dữ liệu sang phân tích | move data over to analytics |
| chỉ là một hàng đợi công việc thuần tuý | is only a pure work queue |
| xuất bản siêu nhẹ | extremely light publishing |
| tầng điều khiển | the control plane |
| thiết bị biên | edge devices |
| quay lại nhu cầu thật | going back to the real requirement |
| thử lại sau một khoảng chờ | a retry after a waiting period |
| một đơn vị song song khá thô | quite a coarse unit of parallelism |
| chưa đến mức phải nuôi một cụm Kafka | not yet at the level where we must feed a Kafka cluster |
| là lựa chọn gọn hơn nhiều | is a far tidier choice |
| câu chốt mà chúng ta nên nói ra | the closing sentence we should say out loud |

**Thuật ngữ cần nhớ**

- bộ tiêu chí chọn → **selection criteria**
- hàng đợi công việc thuần tuý → **a pure work queue**
- độ ưu tiên cho từng message → **per-message priority**
- đơn vị song song → **a unit of parallelism**
- hàng đợi thư chết → **a dead-letter queue (DLQ)**

---

## ⑨ Mô hình ghi nhớ

**Tiếng Việt**

Kafka là một cuộn băng tua lại được, gồm nhật ký, offset và khả năng xử lý lại; RabbitMQ là một quầy phát thư có bộ định tuyến mạnh, gồm exchange, xác nhận và xoá sau khi xong. Chúng ta chọn theo mô hình mà bài toán cần, chứ không chọn theo thương hiệu mà người khác đang dùng.

**English (bám cấu trúc tiếng Việt)**

Kafka is a tape that can be rewound, made of a log, an offset and the ability to reprocess; RabbitMQ is a post counter with a strong router, made of exchanges, acknowledgements and deletion after the work is done. We choose according to the model that the problem needs, rather than choosing according to the brand that other people are using.

---

## ⑩ Bảng thuật ngữ tổng hợp

| Tiếng Việt | Tiếng Anh | Ghi chú phát âm |
|---|---|---|
| hàng đợi | queue | /kjuː/ — đọc y hệt chữ cái **Q**, bốn chữ cuối câm hoàn toàn |
| nhật ký ghi nối đuôi | append-only log | *append* = a-**PEND**, trọng âm âm sau |
| vị trí đọc | offset | **OFF**-set, trọng âm âm đầu (danh từ) |
| thời hạn lưu trữ | retention | ri-**TEN**-shn, chữ *ti* đọc thành /ʃ/ |
| nhóm consumer | consumer group | *consumer* giọng Anh /kənˈsjuːmə/ — "kơn-SYU-mơ" |
| tua lại, xử lý lại | replay / reprocess | |
| xác nhận đã xử lý | acknowledgement (ack) | /əkˈnɒlɪdʒmənt/ — "ợc-NO-lij-mợnt"; **không** tách rời phần *know* |
| bộ định tuyến message | exchange | iks-**CHEYNJ**, trọng âm âm sau |
| khoá định tuyến | routing key | giọng Anh *routing* /ˈruːtɪŋ/ — "RU-ting", **không** đọc "RAO-ting" như giọng Mỹ |
| ràng buộc queue vào exchange | binding | |
| ký tự đại diện | wildcard | |
| phát tới mọi queue | fanout | |
| kiểu đẩy / kiểu kéo | push / pull | |
| kiểm soát ngược | backpressure | **BACK**-pressure, trọng âm âm đầu |
| giới hạn message chưa xác nhận | prefetch limit | **PRE**-fetch, đọc /ˈpriːfetʃ/ |
| đang bay, chưa xử lý xong | in flight | |
| gom lô | to batch | /bætʃ/ — nguyên âm ngắn, khác hẳn *beach* |
| thông lượng | throughput | âm *th* /θ/ + *-ough* đọc /uː/: "THRU-put" |
| giao ít nhất một lần | at-least-once delivery | |
| khử trùng lặp | deduplication | di-du-pli-**KEY**-shn, năm âm tiết, trọng âm áp chót |
| dịch vụ do nhà cung cấp vận hành | a managed service | *managed* = **MAN**-ijd, đuôi chỉ một âm /d/ nhẹ |
| bên đăng ký nhận | subscriber | sub-**SCRI**-ber, cụm *-scr-* phải bật rõ |
| độ bền dữ liệu | durability | du-ra-**BI**-li-ty — trọng âm âm thứ ba, khác *durable* (**DUR**-a-ble) |
| nhân bản | to replicate / replication | *replica* = **REP**-li-ka (trọng âm đầu); *replication* = rep-li-**KEY**-shn |
| bản sao đang đồng bộ | in-sync replica | |
| hệ số nhân bản | replication factor | |
| ghi xuống đĩa, không mất khi restart | persistent | pə-**SIS**-tənt, trọng âm giữa |
| phân mảnh dữ liệu | partition | pɑːˈtɪʃn — par-**TI**-shn, trọng âm âm giữa |
| điểm chết duy nhất | single point of failure (SPOF) | |
| hàng đợi quyết theo đa số | quorum queue | *quorum* /ˈkwɔːrəm/ — "KWO-rợm" |
| kết nối lại | to reconnect | |
| độ trễ tiêu thụ | consumer lag | |
| sự tiến hoá của schema | schema evolution | *schema* /ˈskiːmə/ — "SKI-mơ", **không** đọc "sê-ma" |
| kho đăng ký schema | schema registry | **RE**-gis-try, trọng âm âm đầu |
| tương thích ngược | backward compatibility | com-pat-i-**BI**-li-ty, sáu âm tiết, trọng âm áp chót |
| trường tuỳ chọn | an optional field | |
| độ ưu tiên cho từng message | per-message priority | *priority* /praɪˈɒrəti/ — âm đầu là "prai", không phải "pri" |
| đơn vị song song | a unit of parallelism | **PA**-ra-llel-ism, trọng âm âm đầu |
| hàng đợi thư chết | dead-letter queue (DLQ) | |
| tầng điều khiển | the control plane | |
| bộ tiêu chí chọn | selection criteria | *criteria* = crai-**TIƏ**-ri-a, số nhiều của *criterion* |

---

## ⑪ Bài nói

> **Cách luyện:** bật ghi âm, nói **60–90 giây** cho mỗi đề, **không nhìn tài liệu**. Mỗi bài nói phải dùng ít nhất **sáu thuật ngữ** trong bảng trên. Nói xong thì nghe lại một lượt, đánh dấu chỗ mình ngập ngừng, rồi nói lại chính đề đó thêm một lần nữa.

1. Explain to a junior developer the core difference between Kafka and RabbitMQ, using the recording tape and the post counter pictures, and say what that difference means for replay.

2. A colleague says that Kafka can replace RabbitMQ for everything, so the team should standardise on Kafka. Explain what is wrong with that claim, and name two requirements where you would still pick RabbitMQ.

3. Your team wants to run a few thousand video rendering jobs a day, with priorities and delayed retries, and someone proposes Kafka because everybody uses it. Explain why you would push back and what you would propose instead.

4. Describe what happens to a slow consumer under a push-based broker and under a pull-based broker, and explain which setting you would reach for first in each case.

5. Someone on your team wants four services to receive the same order event, so they point all four at one shared SQS queue. Explain what will actually happen and how you would fix the topology.

6. Describe the three mechanisms a broker relies on for durability, and describe exactly what is lost if the producer does not wait for an acknowledgement.

7. When would you introduce a schema registry, and how would you explain the compatibility rules to a team that has been changing event fields freely for a year?
