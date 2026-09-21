# Bài 3 — Kafka Internals I: partition, consumer group, offset
### Bản song ngữ: tiếng Anh bám cấu trúc tiếng Việt

> **Cách dùng bài này:** đọc đoạn tiếng Việt trước → tự dịch trong đầu, không nhìn xuống → so sánh với đoạn tiếng Anh ngay bên dưới → **đọc to bản tiếng Anh hai lần**: lần một đọc chậm cho đúng từng âm, lần hai đọc liền mạch như đang nói với người phỏng vấn.

---

## ① Topic, partition, offset, broker, cluster

**Tiếng Việt**

Kafka là broker thống trị mảng event streaming, nên hai bài về Kafka chính là trục xương sống của cả chủ đề messaging. Chúng ta bắt đầu bằng bốn khái niệm nền, và nếu chúng ta nắm chắc bốn cái này thì Kafka không còn là một hộp đen nữa. Topic là một chủ đề, ví dụ topic tên `orders` chứa mọi event liên quan tới đơn hàng. Mỗi topic được chia thành nhiều partition, và chúng ta hình dung mỗi partition là một cuộn băng ghi nối đuôi chạy song song với các cuộn khác. Mỗi message rơi vào đúng một partition và nhận một offset, tức là một số thứ tự tăng dần bên trong chính partition đó. Các partition được đặt trên nhiều broker, tức là nhiều máy chủ, và tập hợp các broker đó tạo thành một cluster.

Có một chi tiết nhỏ nhưng người phỏng vấn rất hay hỏi, đó là offset không duy nhất trên toàn topic. Offset số năm của partition không và offset số năm của partition một là hai message hoàn toàn khác nhau. Offset chỉ duy nhất bên trong một partition, và nó chỉ có nghĩa khi chúng ta nói kèm tên topic và số hiệu partition. Nếu chúng ta quên điều này, hậu quả là chúng ta lưu lại mỗi con số offset để đánh dấu tiến độ, rồi sau một lần khởi động lại chúng ta nhảy tới đúng vị trí sai trên đúng partition sai. Vì vậy một vị trí đọc đầy đủ luôn gồm ba phần, gồm tên topic, số hiệu partition, và offset.

**English (bám cấu trúc tiếng Việt)**

Kafka is the broker that dominates the event streaming area, so the two lessons about Kafka are exactly the backbone of the whole messaging topic. We start with four foundational concepts, and if we hold these four firmly then Kafka is no longer a black box. A topic is a subject, for example the topic named `orders` holds every event related to orders. Each topic is split into many partitions, and we picture each partition as an append-only tape running in parallel with the other tapes. Each message falls into exactly one partition and receives an offset, that is, a sequence number that increases inside that partition itself. The partitions are placed on many brokers, that is, many servers, and the set of those brokers forms a cluster.

There is a small detail that interviewers ask about very often, which is that the offset is not unique across the whole topic. Offset number five of partition zero and offset number five of partition one are two completely different messages. The offset is only unique inside one partition, and it only has a meaning when we say it together with the topic name and the partition number. If we forget this, the consequence is that we save only the offset number to mark our progress, and then after one restart we jump to exactly the wrong position on exactly the wrong partition. Therefore a complete reading position always has three parts, which are the topic name, the partition number, and the offset.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| broker thống trị mảng event streaming | the broker that dominates the event streaming area |
| trục xương sống của cả chủ đề | the backbone of the whole topic |
| nếu chúng ta nắm chắc bốn cái này | if we hold these four firmly |
| không còn là một hộp đen nữa | is no longer a black box |
| chúng ta hình dung mỗi partition là | we picture each partition as |
| chạy song song với các cuộn khác | running in parallel with the other tapes |
| một số thứ tự tăng dần | a sequence number that increases |
| offset không duy nhất trên toàn topic | the offset is not unique across the whole topic |
| nó chỉ có nghĩa khi chúng ta nói kèm | it only has a meaning when we say it together with |
| để đánh dấu tiến độ | to mark our progress |
| nhảy tới đúng vị trí sai | jump to exactly the wrong position |
| một vị trí đọc đầy đủ | a complete reading position |

**Thuật ngữ cần nhớ**

- phân mảnh của một topic → **a partition**
- số thứ tự trong partition → **the offset**
- một máy chủ Kafka → **a broker**
- cụm máy chủ → **a cluster**
- ghi nối đuôi → **append-only**

---

## ② Partition vừa là đơn vị song song vừa là đơn vị thứ tự

**Tiếng Việt**

Điểm quan trọng nhất của bài này là partition mang hai vai trò cùng một lúc. Vai trò thứ nhất là đơn vị song song, nghĩa là số partition quyết định tối đa bao nhiêu luồng có thể cùng tiêu thụ một topic trong một nhóm. Vai trò thứ hai là đơn vị thứ tự, nghĩa là Kafka chỉ bảo đảm thứ tự bên trong một partition và không bảo đảm gì giữa các partition với nhau. Nói cách khác, Kafka không hề có thứ tự toàn cục trên một topic có nhiều partition. Nếu ai đó hỏi làm sao để có thứ tự toàn cục, câu trả lời trung thực là chúng ta phải dùng đúng một partition, và khi đó chúng ta mất sạch khả năng song song.

Hậu quả của việc hiểu sai điểm này vừa nặng vừa khó phát hiện. Giả sử một đơn hàng phát ra ba event theo thứ tự tạo, thanh toán và huỷ, nhưng ba event đó rơi vào ba partition khác nhau. Ba consumer khác nhau sẽ đọc chúng cùng một lúc, và không có gì bảo đảm event huỷ được xử lý sau event thanh toán. Kết quả là chúng ta huỷ một đơn chưa từng được thanh toán, hoặc chúng ta thanh toán một đơn đã bị huỷ. Loại lỗi này không xuất hiện khi tải nhẹ, vì lúc đó mọi thứ tình cờ chạy đúng thứ tự, và nó chỉ nổ ra khi hệ thống thật sự bận.

**English (bám cấu trúc tiếng Việt)**

The most important point of this lesson is that a partition carries two roles at the same time. The first role is the unit of parallelism, which means that the number of partitions decides at most how many threads can consume one topic within one group. The second role is the unit of ordering, which means that Kafka only guarantees the order inside one partition and guarantees nothing between the partitions themselves. In other words, Kafka has no global ordering at all on a topic that has many partitions. If somebody asks how to get global ordering, the honest answer is that we have to use exactly one partition, and then we lose all our ability to run in parallel.

The consequence of misunderstanding this point is both serious and hard to detect. Suppose one order emits three events in the order of created, paid and cancelled, but those three events fall into three different partitions. Three different consumers will read them at the same time, and there is nothing that guarantees the cancelled event is processed after the paid event. The result is that we cancel an order that has never been paid, or we pay for an order that has already been cancelled. This kind of bug does not appear under light load, because at that time everything happens to run in the right order, and it only blows up when the system is really busy.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| mang hai vai trò cùng một lúc | carries two roles at the same time |
| đơn vị song song | the unit of parallelism |
| quyết định tối đa bao nhiêu luồng | decides at most how many threads |
| không bảo đảm gì giữa các partition với nhau | guarantees nothing between the partitions themselves |
| nói cách khác | in other words |
| không hề có thứ tự toàn cục | has no global ordering at all |
| chúng ta mất sạch khả năng song song | we lose all our ability to run in parallel |
| vừa nặng vừa khó phát hiện | both serious and hard to detect |
| không có gì bảo đảm | there is nothing that guarantees |
| mọi thứ tình cờ chạy đúng thứ tự | everything happens to run in the right order |
| nó chỉ nổ ra khi hệ thống thật sự bận | it only blows up when the system is really busy |

**Thuật ngữ cần nhớ**

- thứ tự toàn cục → **global ordering**
- thứ tự bên trong một partition → **per-partition ordering**
- đơn vị song song → **the unit of parallelism**
- bảo đảm → **to guarantee**
- tải nhẹ / tải nặng → **light load / heavy load**

---

## ③ Cái giá của việc tăng số partition

**Tiếng Việt**

Vì partition là trần song song, phản xạ đầu tiên của nhiều đội là tăng số partition lên cho nhanh hơn. Việc này đúng là làm tăng thông lượng, nhưng nó kèm theo ba cái giá mà chúng ta phải nói ra. Cái giá thứ nhất là chúng ta càng chia nhỏ thì chúng ta càng xa thứ tự toàn cục. Cái giá thứ hai là mỗi partition thêm một tập tin trên đĩa, thêm kết nối, và thêm chi phí quản lý cho cluster. Cái giá thứ ba, và cũng là cái nguy hiểm nhất, là việc đổi số partition làm đổi luôn cách ánh xạ từ key sang partition.

Chúng ta cần hiểu cơ chế đằng sau cái giá thứ ba, vì đây là một câu hỏi bẫy rất hay gặp. Producer chọn partition bằng cách lấy giá trị băm của key rồi chia lấy dư cho số partition. Khi chúng ta đổi số partition từ sáu lên mười hai, phép chia lấy dư cho ra kết quả khác, nên các message cùng một key có thể rơi vào partition khác so với trước. Lúc đó các event cũ của một đơn hàng nằm ở partition này còn các event mới nằm ở partition kia, và hai partition đó lại được đọc song song. Vì vậy chúng ta mất thứ tự theo key ngay trong giai đoạn chuyển đổi, và khách hàng sẽ báo lỗi trước khi chúng ta kịp nhận ra. Kết luận thực dụng là chúng ta không đổi số partition một cách tuỳ tiện khi hệ thống đang dựa vào thứ tự theo key.

**English (bám cấu trúc tiếng Việt)**

Because the partition is the ceiling of parallelism, the first reflex of many teams is to raise the number of partitions so that things go faster. This does indeed raise throughput, but it comes with three prices that we must say out loud. The first price is that the more finely we split, the further we move away from global ordering. The second price is that each partition adds one more file on disk, more connections, and more management cost for the cluster. The third price, and also the most dangerous one, is that changing the number of partitions changes the mapping from key to partition as well.

We need to understand the mechanism behind the third price, because this is a trap question that comes up very often. The producer chooses the partition by taking the hash value of the key and then taking the remainder after dividing by the number of partitions. When we change the number of partitions from six to twelve, the remainder gives a different result, so the messages with the same key can fall into a different partition compared to before. At that point the old events of one order sit in this partition while the new events sit in that partition, and those two partitions are read in parallel again. Therefore we lose the ordering by key right in the transition period, and the customer will report the bug before we have time to notice it. The practical conclusion is that we do not change the number of partitions carelessly when the system is relying on ordering by key.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| trần song song | the ceiling of parallelism |
| phản xạ đầu tiên của nhiều đội | the first reflex of many teams |
| việc này đúng là làm tăng thông lượng | this does indeed raise throughput |
| ba cái giá mà chúng ta phải nói ra | three prices that we must say out loud |
| chúng ta càng chia nhỏ thì càng xa | the more finely we split, the further we move away |
| cách ánh xạ từ key sang partition | the mapping from key to partition |
| một câu hỏi bẫy rất hay gặp | a trap question that comes up very often |
| lấy giá trị băm của key | taking the hash value of the key |
| chia lấy dư cho số partition | taking the remainder after dividing by the number of partitions |
| so với trước | compared to before |
| ngay trong giai đoạn chuyển đổi | right in the transition period |
| trước khi chúng ta kịp nhận ra | before we have time to notice it |
| một cách tuỳ tiện | carelessly |

**Thuật ngữ cần nhớ**

- giá trị băm → **the hash value**
- phép chia lấy dư → **the modulo operation**
- trần / giới hạn trên → **the ceiling**
- giai đoạn chuyển đổi → **the transition period**
- ánh xạ từ khoá sang partition → **the key-to-partition mapping**

---

## ④ Consumer group và quy tắc một partition một consumer

**Tiếng Việt**

Consumer group là cách Kafka cho nhiều tiến trình chia nhau công việc của một topic. Quy tắc vàng chỉ gồm một câu, đó là mỗi partition được gán cho đúng một consumer trong cùng một nhóm. Từ quy tắc đó chúng ta suy ra ngay hệ quả thứ nhất, đó là nếu số consumer nhiều hơn số partition thì những consumer thừa sẽ ngồi không. Vì vậy khi chúng ta muốn tiêu thụ nhanh hơn, chúng ta phải tăng số partition trước, còn nếu chỉ thêm máy thì chúng ta không giải quyết được gì cả. Đây chính là chỗ nhiều đội đốt tiền hạ tầng mà không nhanh lên được một giây nào.

Hệ quả thứ hai liên quan tới việc nhiều nhóm khác nhau cùng đọc một topic. Hai nhóm khác nhau đọc hoàn toàn độc lập với nhau, và mỗi nhóm giữ offset riêng của mình. Nhờ đó nhóm của service kho và nhóm của service phân tích cùng nhận đủ mọi message mà không tranh nhau. Đây chính là cách Kafka làm được xuất bản và đăng ký, vì một nhóm đóng vai một người đăng ký, còn các consumer bên trong nhóm chỉ là cách chúng ta chia việc. Nếu chúng ta vô tình đặt hai service khác nhau vào cùng một mã nhóm, hậu quả là mỗi service chỉ nhận được một nửa số message và không có ai báo lỗi cả.

**English (bám cấu trúc tiếng Việt)**

A consumer group is the way Kafka lets many processes share the work of one topic. The golden rule consists of only one sentence, which is that each partition is assigned to exactly one consumer within the same group. From that rule we immediately derive the first consequence, which is that if the number of consumers is larger than the number of partitions then the surplus consumers will sit idle. Therefore when we want to consume faster, we have to raise the number of partitions first, and if we only add machines then we solve nothing at all. This is exactly the place where many teams burn infrastructure money without getting one second faster.

The second consequence relates to many different groups reading the same topic. Two different groups read completely independently of each other, and each group holds its own offset. Thanks to that, the group of the inventory service and the group of the analytics service both receive every message without competing with each other. This is exactly how Kafka achieves publish and subscribe, because one group plays the role of one subscriber, while the consumers inside the group are only the way we split the work. If we accidentally put two different services into the same group id, the consequence is that each service receives only half of the messages and nobody reports an error at all.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| cho nhiều tiến trình chia nhau công việc | lets many processes share the work |
| quy tắc vàng chỉ gồm một câu | the golden rule consists of only one sentence |
| được gán cho đúng một consumer | is assigned to exactly one consumer |
| chúng ta suy ra ngay hệ quả thứ nhất | we immediately derive the first consequence |
| những consumer thừa sẽ ngồi không | the surplus consumers will sit idle |
| chúng ta không giải quyết được gì cả | we solve nothing at all |
| đốt tiền hạ tầng | burn infrastructure money |
| không nhanh lên được một giây nào | without getting one second faster |
| độc lập với nhau | independently of each other |
| mà không tranh nhau | without competing with each other |
| một nhóm đóng vai một người đăng ký | one group plays the role of one subscriber |
| nếu chúng ta vô tình đặt | if we accidentally put |

**Thuật ngữ cần nhớ**

- nhóm tiêu thụ → **a consumer group**
- được gán cho → **to be assigned to**
- ngồi không → **to sit idle**
- mã nhóm → **the group id**
- dư thừa → **surplus**

---

## ⑤ Key quyết định partition, và vì thế key quyết định thứ tự

**Tiếng Việt**

Bây giờ chúng ta trả lời câu hỏi producer quyết định message đi vào partition nào. Nếu message có key, producer băm key đó rồi chia lấy dư cho số partition, nên cùng một key luôn đi vào cùng một partition. Nếu message không có key, producer rải các message ra các partition khá đều, thường theo kiểu xoay vòng hoặc theo lô dính. Vì vậy key chính là công cụ để chúng ta giữ thứ tự theo từng thực thể. Nếu chúng ta đặt key bằng mã đơn hàng, mọi event của một đơn sẽ nằm cùng một partition và được xử lý đúng thứ tự, trong khi các đơn khác nhau vẫn trải đều ra nhiều partition và chạy song song.

Đây là câu trả lời chuẩn cho câu hỏi làm sao vừa giữ được thứ tự vừa mở rộng được. Chúng ta không cần thứ tự toàn cục, chúng ta chỉ cần thứ tự trong phạm vi một thực thể. Vì vậy chúng ta chọn key theo đúng thực thể mà nghiệp vụ quan tâm, ví dụ mã đơn hàng, mã tài khoản, hoặc mã người dùng. Nếu chúng ta chọn key quá thô, ví dụ đặt key bằng tên quốc gia, hậu quả là một partition gánh chín mươi phần trăm lưu lượng còn các partition khác nằm không. Hiện tượng này gọi là lệch partition, và nó biến một cụm mười hai partition thành một cụm một partition trên thực tế.

**English (bám cấu trúc tiếng Việt)**

Now we answer the question of how the producer decides which partition a message goes into. If the message has a key, the producer hashes that key and then takes the remainder after dividing by the number of partitions, so the same key always goes into the same partition. If the message has no key, the producer spreads the messages across the partitions fairly evenly, usually in a round-robin way or in sticky batches. Therefore the key is exactly the tool we use to keep ordering per entity. If we set the key to the order id, every event of one order will sit in the same partition and be processed in the right order, while different orders still spread evenly across many partitions and run in parallel.

This is the standard answer to the question of how to keep ordering and scale out at the same time. We do not need global ordering, we only need ordering within the scope of one entity. Therefore we choose the key according to exactly the entity that the business cares about, for example the order id, the account id, or the user id. If we choose a key that is too coarse, for example setting the key to the country name, the consequence is that one partition carries ninety percent of the traffic while the other partitions sit empty. This phenomenon is called partition skew, and it turns a twelve-partition cluster into a one-partition cluster in practice.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| producer băm key đó | the producer hashes that key |
| rải các message ra các partition khá đều | spreads the messages across the partitions fairly evenly |
| theo kiểu xoay vòng | in a round-robin way |
| theo lô dính | in sticky batches |
| giữ thứ tự theo từng thực thể | keep ordering per entity |
| trải đều ra nhiều partition | spread evenly across many partitions |
| vừa giữ được thứ tự vừa mở rộng được | keep ordering and scale out at the same time |
| trong phạm vi một thực thể | within the scope of one entity |
| thực thể mà nghiệp vụ quan tâm | the entity that the business cares about |
| chọn key quá thô | choose a key that is too coarse |
| gánh chín mươi phần trăm lưu lượng | carries ninety percent of the traffic |
| nằm không | sit empty |
| trên thực tế | in practice |

**Thuật ngữ cần nhớ**

- khoá phân mảnh → **the partition key**
- thứ tự theo từng thực thể → **per-entity ordering**
- xoay vòng → **round-robin**
- lệch partition → **partition skew**
- mở rộng ngang → **to scale out**

---

## ⑥ Commit offset trước hay sau khi xử lý

**Tiếng Việt**

Phần cuối cùng về cơ chế là commit offset, và đây là chỗ mà code do AI sinh ra hay sai nhất. Commit offset nghĩa là consumer báo cho Kafka biết mình đã đi tới đâu, để khi nó khởi động lại thì nó biết phải đọc tiếp từ chỗ nào. Nếu chúng ta commit trước khi xử lý xong và tiến trình chết ở giữa, message đó coi như đã được đọc nhưng thật ra nó chưa hề được xử lý. Đó chính là ngữ nghĩa giao nhiều nhất một lần, và nó có nghĩa là chúng ta chấp nhận mất message. Nếu chúng ta commit sau khi xử lý xong và tiến trình chết trước lúc commit, message đó sẽ được đọc lại và được xử lý thêm một lần nữa. Đó là ngữ nghĩa giao ít nhất một lần, và nó đòi hỏi phần xử lý của chúng ta phải chịu được việc chạy lại.

Mặc định an toàn mà chúng ta nên chọn là xử lý xong rồi mới commit. Lý do là nếu xử lý trùng thì chúng ta còn vá được bằng cách làm cho hàm xử lý bất biến khi lặp, còn nếu mất dữ liệu thì chúng ta không vá được nữa. Kafka lưu offset của các nhóm trong một topic nội bộ, nên bản thân offset cũng bền như mọi message khác. Chúng ta sẽ học kỹ phần chịu được chạy lại ở Bài 5, vì nó là gốc của mọi câu hỏi về độ tin cậy. Bây giờ chúng ta chỉ cần nhớ rằng commit nghĩa là tôi đã xử lý xong, chứ không phải tôi đã nhận được.

**English (bám cấu trúc tiếng Việt)**

The last part about mechanisms is the offset commit, and this is the place where AI-generated code goes wrong most often. Committing the offset means that the consumer tells Kafka how far it has gone, so that when it restarts it knows where it has to read on from. If we commit before we finish processing and the process dies in the middle, that message counts as read but in reality it has never been processed. That is exactly the at-most-once semantics, and it means that we accept losing messages. If we commit after we finish processing and the process dies before the commit, that message will be read again and processed one more time. That is the at-least-once semantics, and it requires our processing part to survive being run again.

The safe default that we should choose is to process first and only then commit. The reason is that if we process twice then we can still patch it by making the handler function unchanged under repetition, but if we lose data then we cannot patch it any more. Kafka stores the offsets of the groups in an internal topic, so the offset itself is as durable as every other message. We will study the part about surviving a re-run carefully in Lesson 5, because it is the root of every question about reliability. For now we only need to remember that a commit means I have finished processing, rather than I have received it.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| code do AI sinh ra hay sai nhất | where AI-generated code goes wrong most often |
| báo cho Kafka biết mình đã đi tới đâu | tells Kafka how far it has gone |
| nó biết phải đọc tiếp từ chỗ nào | it knows where it has to read on from |
| chết ở giữa | dies in the middle |
| coi như đã được đọc | counts as read |
| thật ra nó chưa hề được xử lý | in reality it has never been processed |
| chúng ta chấp nhận mất message | we accept losing messages |
| phải chịu được việc chạy lại | must survive being run again |
| mặc định an toàn | the safe default |
| bất biến khi lặp | unchanged under repetition |
| chúng ta không vá được nữa | we cannot patch it any more |
| bền như mọi message khác | as durable as every other message |

**Thuật ngữ cần nhớ**

- ghi nhận vị trí đã xử lý → **to commit the offset**
- giao nhiều nhất một lần → **at-most-once semantics**
- giao ít nhất một lần → **at-least-once semantics**
- chịu được chạy lại nhiều lần → **idempotent**
- topic nội bộ → **an internal topic**

---

## ⑦ Bẫy commit tự động

**Tiếng Việt**

Còn một cái bẫy nữa mà chúng ta nên chủ động nói ra trong phỏng vấn, đó là chế độ commit tự động. Khi chúng ta bật cấu hình commit tự động, thư viện sẽ commit offset theo một chu kỳ thời gian, ví dụ mỗi năm giây một lần. Điều nguy hiểm là chu kỳ đó không liên quan gì tới việc chúng ta đã xử lý xong hay chưa. Nếu hàm xử lý của chúng ta chạy bất đồng bộ và mất mười giây, thư viện có thể đã commit offset trong khi công việc thật vẫn còn dang dở. Khi đó chúng ta rơi vào ngữ nghĩa giao nhiều nhất một lần một cách âm thầm, và chúng ta mất message mà không có một dấu hiệu nào trên log.

Khi một đồng nghiệp nói rằng cứ bật commit tự động cho gọn code, chúng ta nên phản biện theo hai bước. Bước một, chúng ta hỏi lại rằng nếu tiến trình bị giết giữa chừng thì hệ thống chấp nhận mất bao nhiêu message. Bước hai, nếu câu trả lời là không được mất message nào, thì commit tự động đã bị loại ngay từ đầu và chúng ta phải commit thủ công. Đúng là commit thủ công tốn thêm vài dòng code và buộc chúng ta phải nghĩ về tính bất biến khi lặp. Nhưng vài dòng code đó rẻ hơn rất nhiều so với một buổi tối đi truy tìm những đơn hàng đã biến mất.

**English (bám cấu trúc tiếng Việt)**

There is one more trap that we should bring up ourselves in an interview, which is the auto-commit mode. When we turn on the auto-commit setting, the library will commit the offset on a time cycle, for example once every five seconds. The dangerous thing is that this cycle has nothing to do with whether we have finished processing or not. If our handler function runs asynchronously and takes ten seconds, the library may already have committed the offset while the real work is still unfinished. At that point we fall into at-most-once semantics silently, and we lose messages without a single sign in the log.

When a colleague says that we should just turn on auto-commit to keep the code tidy, we should push back in two steps. Step one, we ask back how many messages the system accepts losing if the process is killed halfway. Step two, if the answer is that we must not lose any message, then auto-commit is ruled out from the start and we have to commit manually. It is true that manual commit costs a few extra lines of code and forces us to think about being unchanged under repetition. But those few lines of code are far cheaper than one evening spent hunting for orders that have disappeared.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| chúng ta nên chủ động nói ra | we should bring up ourselves |
| theo một chu kỳ thời gian | on a time cycle |
| không liên quan gì tới việc | has nothing to do with whether |
| công việc thật vẫn còn dang dở | the real work is still unfinished |
| một cách âm thầm | silently |
| không có một dấu hiệu nào trên log | without a single sign in the log |
| cho gọn code | to keep the code tidy |
| chúng ta hỏi lại rằng | we ask back how |
| bị giết giữa chừng | is killed halfway |
| đã bị loại ngay từ đầu | is ruled out from the start |
| tốn thêm vài dòng code | costs a few extra lines of code |
| một buổi tối đi truy tìm | one evening spent hunting for |

**Thuật ngữ cần nhớ**

- commit tự động → **auto-commit**
- commit thủ công → **manual commit**
- xử lý xong rồi mới commit → **process-then-commit**
- bị loại từ đầu → **ruled out from the start**
- dang dở → **unfinished**

---

## ⑧ Mô hình ghi nhớ

**Tiếng Việt**

Một partition là một cuộn băng có thứ tự, và trong một nhóm thì đúng một consumer đọc nó; key quyết định partition, nên key cũng quyết định thứ tự. Commit có nghĩa là tôi đã xử lý xong, vì vậy chúng ta luôn đặt commit sau phần xử lý.

**English (bám cấu trúc tiếng Việt)**

One partition is one ordered tape, and within one group exactly one consumer reads it; the key decides the partition, so the key also decides the ordering. A commit means I have finished processing, therefore we always put the commit after the processing part.

---

## ⑨ Bảng thuật ngữ tổng hợp

| Tiếng Việt | Tiếng Anh | Ghi chú phát âm |
|---|---|---|
| phân mảnh của một topic | partition | pɑːˈtɪʃn — par-**TI**-shn, trọng âm âm giữa |
| số thứ tự trong partition | offset | **OFF**-set, trọng âm âm đầu |
| một máy chủ Kafka | broker | |
| cụm máy chủ | cluster | |
| ghi nối đuôi | append-only | *append* = a-**PEND**, trọng âm âm sau |
| thứ tự toàn cục | global ordering | |
| thứ tự trong một partition | per-partition ordering | |
| đơn vị song song | unit of parallelism | **PA**-ra-llel-ism, trọng âm âm đầu |
| luồng xử lý | thread | âm *th* /θ/ + đuôi /d/: "thred", không phải "tret" |
| bảo đảm | to guarantee | ga-ran-**TEE**, trọng âm âm cuối |
| giá trị băm | hash value | |
| phép chia lấy dư | modulo | **MO**-du-lo, trọng âm âm đầu |
| trần, giới hạn trên | ceiling | /ˈsiːlɪŋ/ — "SI-linh", chữ *c* đọc /s/ |
| giai đoạn chuyển đổi | transition period | tran-**SI**-shn, chữ *ti* đọc /ʒ/ hoặc /ʃ/ |
| nhóm tiêu thụ | consumer group | *consumer* giọng Anh /kənˈsjuːmə/ — "kơn-SYU-mơ" |
| được gán cho | to be assigned to | *assigned* = ə-**SAIND**, chữ *g* câm |
| ngồi không | to sit idle | *idle* /ˈaɪdl/ — "AI-đợl", **không** đọc "i-đồ" |
| dư thừa | surplus | **SUR**-plus, trọng âm âm đầu |
| mã nhóm | group id | |
| khoá phân mảnh | partition key | |
| thứ tự theo từng thực thể | per-entity ordering | *entity* = **EN**-ti-ty, trọng âm âm đầu |
| xoay vòng | round-robin | |
| lệch partition | partition skew | *skew* /skjuː/ — đọc như chữ **Q** có thêm /sk/ ở đầu |
| mở rộng ngang | to scale out | |
| lưu lượng | traffic | |
| ghi nhận vị trí đã xử lý | to commit the offset | *commit* = cə-**MIT**, trọng âm âm sau, cả danh từ lẫn động từ |
| ngữ nghĩa giao hàng | delivery semantics | *semantics* = si-**MAN**-tics, trọng âm âm giữa |
| giao nhiều nhất một lần | at-most-once | |
| giao ít nhất một lần | at-least-once | |
| chịu được chạy lại nhiều lần | idempotent | i-**DEM**-po-tent, trọng âm âm thứ hai |
| topic nội bộ | internal topic | |
| độ tin cậy | reliability | ri-lai-a-**BI**-li-ty, trọng âm áp chót |
| commit tự động | auto-commit | |
| commit thủ công | manual commit | **MAN**-u-al, ba âm tiết |
| xử lý xong rồi mới commit | process-then-commit | *process* (động từ) giọng Anh = **PRO**-cess /ˈprəʊses/ |
| dang dở | unfinished | |
| bị loại từ đầu | ruled out from the start | đuôi cụm *-lt aʊt* phải bật rõ, không nuốt |
| khởi động lại | to restart | |
| bất đồng bộ | asynchronously | ây-**SING**-crơ-nợs-li, trọng âm âm thứ hai |

---

## ⑩ Bài nói

> **Cách luyện:** bật ghi âm, nói **60–90 giây** cho mỗi đề, **không nhìn tài liệu**. Mỗi bài nói phải dùng ít nhất **sáu thuật ngữ** trong bảng trên. Nói xong thì nghe lại một lượt, đánh dấu chỗ mình ngập ngừng, rồi nói lại chính đề đó thêm một lần nữa.

1. Explain to a junior developer how a topic, a partition, an offset and a consumer group relate to each other, and say why an offset on its own is not enough to describe a position.

2. A colleague says the team can double consumption speed by doubling the number of consumers in the group. Explain what is wrong with that, and say what you would change instead.

3. Your team raised the partition count from six to twelve last night, and this morning customers report that events for the same order are processed out of sequence. Describe what happened and why.

4. Describe how a producer decides which partition a message goes to, and explain how you would use the key to keep ordering per order while still processing many orders in parallel.

5. Someone proposes setting the key to the country name so that traffic is grouped neatly by region. Explain why you would push back, and describe what the load would look like afterwards.

6. Describe what happens if the process is killed after the commit but before the handler finishes, and describe what happens if the commit comes after the handler instead.

7. When would you accept at-least-once semantics rather than trying to avoid duplicates, and what would you require from the consumer code before you accepted it?
