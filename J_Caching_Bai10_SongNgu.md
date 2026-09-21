# Bài 10 — Công việc chạy nền: BullMQ và Redis Streams
### Bản song ngữ: tiếng Anh bám cấu trúc tiếng Việt

> **Cách dùng bài này:** đọc đoạn tiếng Việt trước → tự dịch trong đầu → so với đoạn tiếng Anh bên dưới → **đọc to bản tiếng Anh hai lần**: lần một đọc chậm cho đúng, lần hai đọc trôi như đang nói với người phỏng vấn.

---

## Phần 1 — Ba lớp vấn đề mà hàng đợi công việc giải

**Tiếng Việt**

Hàng đợi công việc giải ba lớp vấn đề khác nhau, và chúng ta nên kể đủ cả ba khi được hỏi. Lớp thứ nhất là tách việc chậm ra khỏi request của người dùng, ví dụ chuyển mã video hoặc gửi email. Nhờ đó request HTTP không bị treo, và chúng ta trả về kết quả thành công ngay lập tức. Lớp thứ hai là làm phẳng những đợt tải tăng đột ngột: hàng đợi đóng vai trò một bộ đệm khi lượng việc vượt quá năng lực của hệ thống phía sau. Nếu không có bộ đệm đó, một đợt tải cao có thể làm sập chính dịch vụ mà chúng ta phụ thuộc. Lớp thứ ba là thử lại những việc thất bại tạm thời, thay vì để cả request của người dùng hỏng theo. Ngoài ba lớp đó, hàng đợi còn tách rời bên tạo việc và bên xử lý việc, vì vậy chúng ta mở rộng số worker một cách độc lập với số máy chủ web.

Cách nói này mạnh hơn hẳn cách nói rằng hàng đợi giúp hệ thống nhanh hơn. Nó cho thấy chúng ta nhìn hàng đợi như một công cụ kiểm soát rủi ro, chứ không chỉ như một mẹo tăng tốc. Trong phỏng vấn, chúng ta nên gắn mỗi lớp với một ví dụ cụ thể mà chúng ta từng gặp. Ví dụ càng cụ thể thì câu trả lời càng khó bị hỏi vặn.

**English (bám cấu trúc tiếng Việt)**

A job queue solves three different classes of problem, and we should list all three when we are asked. The first class is moving slow work out of the user's request, for example transcoding a video or sending an email. Thanks to that the HTTP request does not hang, and we return a successful result immediately. The second class is flattening sudden spikes in load: the queue acts as a buffer when the amount of work exceeds the capacity of the system behind it. Without that buffer, a burst of load can bring down the very service that we depend on. The third class is retrying work that has failed temporarily, instead of letting the user's whole request fail along with it. Besides those three classes, a queue also decouples the side that creates work from the side that processes work, therefore we scale the number of workers independently of the number of web servers.

This way of speaking is far stronger than saying that a queue makes the system faster. It shows that we see the queue as a tool for controlling risk, and not merely as a trick for going faster. In an interview, we should tie each class to a concrete example that we have met ourselves. The more concrete the example, the harder the answer is to pick apart.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| tách việc chậm ra khỏi request của người dùng | moving slow work out of the user's request |
| chuyển mã video | transcoding a video |
| request HTTP không bị treo | the HTTP request does not hang |
| làm phẳng những đợt tải tăng đột ngột | flattening sudden spikes in load |
| đóng vai trò một bộ đệm | acts as a buffer |
| vượt quá năng lực của hệ thống phía sau | exceeds the capacity of the system behind it |
| làm sập chính dịch vụ mà chúng ta phụ thuộc | bring down the very service that we depend on |
| thất bại tạm thời | failed temporarily |
| tách rời bên tạo việc và bên xử lý việc | decouples the side that creates work from the side that processes work |
| một công cụ kiểm soát rủi ro | a tool for controlling risk |
| chỉ như một mẹo tăng tốc | merely as a trick for going faster |
| càng khó bị hỏi vặn | the harder the answer is to pick apart |

**Thuật ngữ cần nhớ**

- hàng đợi công việc → **a job queue**
- đợt tải tăng đột ngột → **a spike in load**
- bộ đệm → **a buffer**
- tách rời phụ thuộc → **to decouple**
- tiến trình xử lý việc → **a worker**

---

## Phần 2 — BullMQ: hàng đợi chuẩn cho hệ Node

**Tiếng Việt**

BullMQ là thư viện hàng đợi phổ biến nhất cho Node, và nó chạy trên chính Redis mà chúng ta đã có. Nó là bản kế nhiệm của thư viện Bull cũ, nó được viết bằng TypeScript, và bên trong nó tận dụng Redis Streams. Nó cung cấp sẵn những thứ mà chúng ta thường phải tự viết. Nó có các trạng thái của công việc, gồm đang chờ, đang chạy, bị hoãn, đã xong và thất bại. Nó có cơ chế thử lại kèm khoảng lùi, có hoãn theo thời gian, có mức ưu tiên, có giới hạn tần suất, và có công việc lặp lại theo lịch. Nó còn hỗ trợ luồng cha con giữa các công việc, và có một bảng điều khiển để chúng ta quan sát hàng đợi.

Chúng ta nên nói thêm một câu về vận hành, vì đó là chỗ nhiều người quên. Worker phải chạy trong một tiến trình riêng, chứ không nằm chung với tiến trình phục vụ HTTP. Nếu chúng ta để chung, một công việc nặng sẽ chiếm mất tài nguyên của những request đang cần trả lời nhanh. Bảng điều khiển cũng phải có xác thực, vì nó cho phép xem và thao tác lên toàn bộ công việc trong hệ thống.

**English (bám cấu trúc tiếng Việt)**

BullMQ is the most popular queue library for Node, and it runs on the very Redis that we already have. It is the successor of the old Bull library, it is written in TypeScript, and internally it makes use of Redis Streams. It provides out of the box the things that we would usually have to write ourselves. It has job states, including waiting, active, delayed, completed and failed. It has a retry mechanism with a backoff, it has delays over time, it has priorities, it has rate limiting, and it has repeatable jobs on a schedule. It also supports parent-and-child flows between jobs, and it has a dashboard for us to observe the queue.

We should add one sentence about operations, because that is the place many people forget. The worker has to run in a separate process, instead of sitting together with the process serving HTTP. If we put them together, one heavy job will take up the resources of the requests that need a fast answer. The dashboard also has to have authentication, because it allows viewing and acting on every job in the system.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| chạy trên chính Redis mà chúng ta đã có | runs on the very Redis that we already have |
| bản kế nhiệm của thư viện Bull cũ | the successor of the old Bull library |
| bên trong nó tận dụng | internally it makes use of |
| cung cấp sẵn | provides out of the box |
| những thứ mà chúng ta thường phải tự viết | the things that we would usually have to write ourselves |
| thử lại kèm khoảng lùi | retry with a backoff |
| công việc lặp lại theo lịch | repeatable jobs on a schedule |
| luồng cha con giữa các công việc | parent-and-child flows between jobs |
| chỗ nhiều người quên | the place many people forget |
| chiếm mất tài nguyên của những request | take up the resources of the requests |
| thao tác lên toàn bộ công việc trong hệ thống | acting on every job in the system |

**Thuật ngữ cần nhớ**

- bản kế nhiệm → **the successor**
- trạng thái công việc → **a job state**
- khoảng lùi giữa các lần thử → **a backoff**
- mức ưu tiên → **priority**
- xác thực → **authentication**

---

## Phần 3 — Stream so với hàng đợi dựa trên danh sách

**Tiếng Việt**

Có hai cách làm hàng đợi trên Redis, và chúng khác nhau ở mức nền tảng. Cách thứ nhất là dùng danh sách, với lệnh đẩy vào một đầu và lệnh lấy ra ở đầu kia có chờ. Cách này rất đơn giản, nhưng khi một thông điệp bị lấy ra thì nó biến mất hẳn khỏi hàng đợi. Cách thứ hai là dùng stream, tức là một nhật ký chỉ ghi thêm, trong đó mỗi thông điệp mang một định danh riêng. Vì thông điệp vẫn nằm lại trong nhật ký, chúng ta có thể đọc lại lịch sử khi cần. Stream còn cho phép nhiều nhóm consumer đọc độc lập với nhau trên cùng một dòng dữ liệu. Quan trọng nhất là stream có sẵn cơ chế xác nhận đã xử lý, có danh sách công việc đang chờ xác nhận, và có cách thu hồi công việc của một worker đã chết.

Ba lệnh cốt lõi rất đáng nhớ, vì chúng ta hay phải gọi tên chúng. Lệnh thứ nhất thêm thông điệp vào stream, lệnh thứ hai đọc theo nhóm consumer, và lệnh thứ ba xác nhận rằng thông điệp đã được xử lý xong. Nhóm consumer chia thông điệp giữa các worker, vì vậy hai worker không bao giờ nhận cùng một thông điệp trong điều kiện bình thường. Nhóm consumer cũng theo dõi những thông điệp đã phát ra nhưng chưa được xác nhận, và chính danh sách đó cho chúng ta khả năng giao ít nhất một lần. Nếu chúng ta tự viết hàng đợi bằng danh sách, chúng ta sẽ phải tự cài lại toàn bộ những thứ này và gần như chắc chắn sẽ cài sai một chỗ nào đó.

**English (bám cấu trúc tiếng Việt)**

There are two ways to build a queue on Redis, and they differ at a fundamental level. The first way is to use a list, with a command that pushes onto one end and a command that pops from the other end with waiting. This way is very simple, but when a message is taken out then it disappears from the queue for good. The second way is to use a stream, that is an append-only log in which every message carries its own identifier. Because the message stays behind in the log, we can read the history again when we need to. A stream also lets several consumer groups read independently of each other on the same flow of data. Most importantly, a stream comes with an acknowledgement mechanism, with a list of jobs waiting to be acknowledged, and with a way to reclaim the jobs of a worker that has died.

The three core commands are well worth remembering, because we often have to name them. The first command adds a message to the stream, the second command reads through a consumer group, and the third command acknowledges that the message has been processed. A consumer group splits the messages between the workers, therefore two workers never receive the same message under normal conditions. A consumer group also tracks the messages that have been handed out but not yet acknowledged, and it is exactly that list which gives us at-least-once delivery. If we write the queue ourselves with a list, we will have to reimplement all of this and we will almost certainly get one part of it wrong.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| khác nhau ở mức nền tảng | differ at a fundamental level |
| đẩy vào một đầu … lấy ra ở đầu kia có chờ | pushes onto one end … pops from the other end with waiting |
| nó biến mất hẳn khỏi hàng đợi | it disappears from the queue for good |
| một nhật ký chỉ ghi thêm | an append-only log |
| mang một định danh riêng | carries its own identifier |
| vì thông điệp vẫn nằm lại trong nhật ký | because the message stays behind in the log |
| đọc độc lập với nhau | read independently of each other |
| danh sách công việc đang chờ xác nhận | a list of jobs waiting to be acknowledged |
| thu hồi công việc của một worker đã chết | reclaim the jobs of a worker that has died |
| trong điều kiện bình thường | under normal conditions |
| đã phát ra nhưng chưa được xác nhận | have been handed out but not yet acknowledged |
| tự cài lại toàn bộ những thứ này | reimplement all of this |

**Thuật ngữ cần nhớ**

- nhật ký chỉ ghi thêm → **an append-only log**
- nhóm consumer → **a consumer group**
- xác nhận đã xử lý → **to acknowledge**
- đọc lại lịch sử → **to replay the history**
- giao ít nhất một lần → **at-least-once delivery**

---

## Phần 4 — Không mất công việc khi worker chết

**Tiếng Việt**

Câu hỏi quan trọng nhất về hàng đợi là chuyện gì xảy ra khi một worker chết giữa chừng. Để không mất công việc, worker phải nhận công việc bằng một thao tác nguyên tử và phải đánh dấu rằng nó đang xử lý. Worker chỉ xác nhận sau khi đã làm xong, chứ không xác nhận ngay lúc vừa nhận. Nếu worker chết trước khi xác nhận, công việc đó vẫn nằm trong danh sách chờ xác nhận. Sau một khoảng thời gian, hệ thống sẽ thu hồi công việc đó và giao lại cho một worker khác. BullMQ gọi những công việc như vậy là công việc bị treo, và nó có cơ chế tự thu hồi chúng. Hệ quả trực tiếp là chúng ta chỉ có bảo đảm giao ít nhất một lần, chứ không có bảo đảm giao đúng một lần.

Vì vậy mọi consumer của chúng ta bắt buộc phải lặp lại được mà không gây hại. Cách phổ biến nhất là lưu một dấu hiệu rằng công việc mang định danh này đã được xử lý rồi. Một cách khác là dùng khoá chống trùng do chính nhà cung cấp dịch vụ bên ngoài hỗ trợ. Điều chúng ta tuyệt đối không được làm là giả định rằng hệ thống bảo đảm mỗi công việc chạy đúng một lần.

**English (bám cấu trúc tiếng Việt)**

The most important question about a queue is what happens when a worker dies halfway through. So that we do not lose the job, the worker has to claim the job with an atomic operation and has to mark that it is processing it. The worker only acknowledges after it has finished, instead of acknowledging the moment it receives the job. If the worker dies before acknowledging, that job still sits in the list waiting to be acknowledged. After a period of time, the system will reclaim that job and hand it to another worker. BullMQ calls jobs like that stalled jobs, and it has a mechanism to reclaim them automatically. The direct consequence is that we only have an at-least-once guarantee, instead of having an exactly-once guarantee.

Therefore every consumer of ours is obliged to be repeatable without doing harm. The most common way is to store a marker saying that the job carrying this identifier has already been processed. Another way is to use a deduplication key supported by the external provider itself. The thing we must absolutely not do is assume that the system guarantees each job runs exactly once.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| chết giữa chừng | dies halfway through |
| nhận công việc bằng một thao tác nguyên tử | claim the job with an atomic operation |
| chứ không xác nhận ngay lúc vừa nhận | instead of acknowledging the moment it receives the job |
| vẫn nằm trong danh sách chờ xác nhận | still sits in the list waiting to be acknowledged |
| thu hồi công việc đó và giao lại | reclaim that job and hand it to |
| công việc bị treo | stalled jobs |
| hệ quả trực tiếp là | the direct consequence is that |
| bảo đảm giao đúng một lần | an exactly-once guarantee |
| bắt buộc phải lặp lại được mà không gây hại | is obliged to be repeatable without doing harm |
| lưu một dấu hiệu rằng | store a marker saying that |
| khoá chống trùng | a deduplication key |
| điều chúng ta tuyệt đối không được làm | the thing we must absolutely not do |

**Thuật ngữ cần nhớ**

- nhận lấy công việc → **to claim a job**
- công việc bị treo → **a stalled job**
- thu hồi lại → **to reclaim**
- lặp lại không gây hại → **idempotent**
- khoá chống trùng → **a deduplication key**

---

## Phần 5 — Hàng đợi thư chết, giới hạn thử lại và khoảng lùi

**Tiếng Việt**

Việc thử lại phải có giới hạn, nếu không chúng ta sẽ tạo ra một cơn bão thử lại. Khi một công việc thất bại quá số lần cho phép, chúng ta đẩy nó sang hàng đợi thư chết để điều tra sau. Nhờ đó công việc hỏng không quay vòng mãi mãi và không chiếm mất năng lực xử lý của những công việc khác. Khoảng thời gian giữa các lần thử lại phải tăng dần, thường theo cấp số nhân, và nên có thêm một lượng ngẫu nhiên. Lượng ngẫu nhiên đó rất quan trọng, vì nếu hàng nghìn công việc cùng thử lại tại đúng một thời điểm thì chúng ta lại tự tạo ra một đợt tải dồn. Và vì mỗi lần thử lại là một lần chạy lại, công việc bắt buộc phải lặp lại được mà không gây hại.

Hàng đợi thư chết không phải là nơi để quên công việc đi. Chúng ta phải có cảnh báo khi số công việc trong đó tăng lên, vì đó là dấu hiệu một thứ gì đó đang hỏng có hệ thống. Chúng ta cũng cần một cách để phát lại những công việc đó sau khi đã sửa nguyên nhân. Nếu thiếu hai thứ này, hàng đợi thư chết chỉ là một cái thùng rác được đặt cho một cái tên sang trọng.

**English (bám cấu trúc tiếng Việt)**

Retrying has to be limited, otherwise we will create a retry storm. When a job has failed more than the allowed number of times, we push it into the dead letter queue to investigate later. Thanks to that a broken job does not go round forever and does not take up the processing capacity of the other jobs. The interval between retries has to grow, usually exponentially, and it should have a random amount added on. That random amount is very important, because if thousands of jobs all retry at exactly the same moment then we create a pile-up of load ourselves. And because every retry is another run, the job is obliged to be repeatable without doing harm.

The dead letter queue is not a place to forget jobs in. We have to have an alert when the number of jobs in it goes up, because that is the sign that something is failing systematically. We also need a way to replay those jobs after we have fixed the cause. Without these two things, the dead letter queue is merely a bin that has been given a grand name.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| một cơn bão thử lại | a retry storm |
| thất bại quá số lần cho phép | failed more than the allowed number of times |
| hàng đợi thư chết | the dead letter queue |
| không quay vòng mãi mãi | does not go round forever |
| chiếm mất năng lực xử lý | take up the processing capacity |
| tăng dần, thường theo cấp số nhân | grow, usually exponentially |
| một lượng ngẫu nhiên | a random amount |
| chúng ta lại tự tạo ra một đợt tải dồn | we create a pile-up of load ourselves |
| mỗi lần thử lại là một lần chạy lại | every retry is another run |
| một thứ gì đó đang hỏng có hệ thống | something is failing systematically |
| phát lại những công việc đó | replay those jobs |
| một cái thùng rác được đặt cho một cái tên sang trọng | a bin that has been given a grand name |

**Thuật ngữ cần nhớ**

- cơn bão thử lại → **a retry storm**
- hàng đợi thư chết → **a dead letter queue (DLQ)**
- lùi theo cấp số nhân → **exponential backoff**
- lượng ngẫu nhiên thêm vào → **jitter**
- cảnh báo → **an alert**

---

## Phần 6 — Khi nào BullMQ không còn đủ

**Tiếng Việt**

Chúng ta cũng phải biết lúc nào công cụ hiện tại không còn đủ nữa. Chúng ta chuyển sang Kafka khi chúng ta cần một dòng sự kiện được lưu bền lâu dài và cần đọc lại lịch sử sau nhiều ngày hoặc nhiều tuần. Chúng ta cũng chuyển sang Kafka khi thông lượng rất lớn và khi nhiều nhóm consumer độc lập cùng tiêu thụ một dòng dữ liệu. Chúng ta chọn RabbitMQ khi nhu cầu chính là định tuyến phức tạp giữa các hàng đợi. Còn BullMQ hợp nhất với một hàng đợi công việc trong hệ Node, khi chúng ta cần thử lại và hoãn, và khi chúng ta muốn giữ hạ tầng thật nhẹ vì đã có sẵn Redis. Cách chọn này dựa trên nhu cầu chứ không dựa trên độ nổi tiếng của công cụ.

Một câu trả lời trưởng thành nên nêu cả cái giá của việc chuyển sang Kafka. Kafka thêm một hệ thống phân tán nữa mà chúng ta phải vận hành, phải theo dõi và phải nâng cấp. Với một đội nhỏ, cái giá vận hành đó thường lớn hơn lợi ích mà chúng ta thu được. Vì vậy chúng ta chỉ chuyển khi có một nhu cầu cụ thể mà công cụ hiện tại không đáp ứng nổi.

**English (bám cấu trúc tiếng Việt)**

We also have to know when the current tool is no longer enough. We move to Kafka when we need a stream of events stored durably for a long time and need to read the history again after many days or many weeks. We also move to Kafka when the throughput is very large and when several independent consumer groups consume the same flow of data. We choose RabbitMQ when the main requirement is complex routing between queues. BullMQ, meanwhile, fits best for a job queue in the Node world, when we need retries and delays, and when we want to keep the infrastructure really light because we already have Redis. This way of choosing is based on the requirement instead of being based on how famous the tool is.

A mature answer should also state the price of moving to Kafka. Kafka adds one more distributed system that we have to operate, have to monitor and have to upgrade. For a small team, that operational price is usually larger than the benefit we get back. Therefore we only move when there is a concrete requirement that the current tool cannot meet.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| lúc nào công cụ hiện tại không còn đủ nữa | when the current tool is no longer enough |
| một dòng sự kiện được lưu bền lâu dài | a stream of events stored durably for a long time |
| cùng tiêu thụ một dòng dữ liệu | consume the same flow of data |
| định tuyến phức tạp giữa các hàng đợi | complex routing between queues |
| hợp nhất với một hàng đợi công việc trong hệ Node | fits best for a job queue in the Node world |
| giữ hạ tầng thật nhẹ | keep the infrastructure really light |
| không dựa trên độ nổi tiếng của công cụ | not based on how famous the tool is |
| một câu trả lời trưởng thành | a mature answer |
| phải vận hành, phải theo dõi và phải nâng cấp | have to operate, have to monitor and have to upgrade |
| cái giá vận hành đó thường lớn hơn lợi ích | that operational price is usually larger than the benefit |
| một nhu cầu cụ thể mà công cụ hiện tại không đáp ứng nổi | a concrete requirement that the current tool cannot meet |

**Thuật ngữ cần nhớ**

- dòng sự kiện → **an event stream**
- lưu bền lâu dài → **stored durably**
- định tuyến → **routing**
- theo dõi hệ thống → **to monitor**
- chi phí vận hành → **the operational cost**

---

## Phần 7 — Tách Redis của hàng đợi khỏi Redis của cache

**Tiếng Việt**

Đây là điểm nối trực tiếp với Bài 6, và nó lặp lại vì nó quan trọng. Cache cần được phép đuổi key, vì vậy nó dùng chính sách đuổi mọi key khi bộ nhớ đầy. Công việc trong hàng đợi thì không được phép mất, vì vậy nó cần chính sách không đuổi key và cần bật lưu bền. Nếu chúng ta đặt hai thứ đó chung một instance, chúng ta gặp hai rủi ro ngược chiều nhau. Rủi ro thứ nhất là cơ chế đuổi key xoá nhầm một công việc chưa được xử lý. Rủi ro thứ hai là hàng đợi phình lên trong một đợt tải cao và đẩy toàn bộ dữ liệu cache ra ngoài.

Ngoài ra, hai vai trò này có kiểu tải và kiểu hỏng hoàn toàn khác nhau. Cache chịu tải đọc rất cao và hỏng theo kiểu suy giảm nhẹ, còn hàng đợi chịu tải ghi và hỏng theo kiểu mất việc. Vì vậy chúng ta nên tách thành hai instance với hai chuỗi kết nối khác nhau ngay từ đầu. Việc tách sau khi hệ thống đã chạy luôn tốn kém hơn nhiều so với việc tách ngay từ ngày đầu tiên.

**English (bám cấu trúc tiếng Việt)**

This is the point that connects directly back to Lesson 6, and it comes up again because it matters. A cache needs to be allowed to evict keys, therefore it uses a policy that evicts all keys when the memory is full. Jobs in a queue, on the other hand, must not be lost, therefore they need a no-eviction policy and need persistence turned on. If we put those two things in one instance, we run into two risks pointing in opposite directions. The first risk is that the eviction mechanism deletes by mistake a job that has not been processed. The second risk is that the queue swells up during a burst of load and pushes all the cache data out.

Besides that, these two roles have completely different load patterns and failure modes. A cache takes a very high read load and fails by degrading gently, while a queue takes a write load and fails by losing work. Therefore we should split them into two instances with two different connection strings right from the start. Splitting them after the system is already running is always far more expensive than splitting them on day one.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| điểm nối trực tiếp với Bài 6 | the point that connects directly back to Lesson 6 |
| nó lặp lại vì nó quan trọng | it comes up again because it matters |
| cần được phép đuổi key | needs to be allowed to evict keys |
| hai rủi ro ngược chiều nhau | two risks pointing in opposite directions |
| xoá nhầm một công việc chưa được xử lý | deletes by mistake a job that has not been processed |
| phình lên trong một đợt tải cao | swells up during a burst of load |
| đẩy toàn bộ dữ liệu cache ra ngoài | pushes all the cache data out |
| kiểu tải và kiểu hỏng | load patterns and failure modes |
| hỏng theo kiểu suy giảm nhẹ | fails by degrading gently |
| hai chuỗi kết nối khác nhau | two different connection strings |
| tách ngay từ ngày đầu tiên | splitting them on day one |

**Thuật ngữ cần nhớ**

- chính sách không đuổi key → **a no-eviction policy**
- kiểu hỏng → **a failure mode**
- chuỗi kết nối → **a connection string**
- suy giảm nhẹ → **to degrade gently**
- lưu bền → **persistence**

---

## Phần 8 — Góc tech lead: worker chết ngay trước khi xác nhận

**Tiếng Việt**

Tình huống thách đố của bài này rất đơn giản nhưng rất sâu. Một worker gửi email thành công, rồi nó chết ngay trước khi kịp xác nhận công việc. Vì công việc chưa được xác nhận, hệ thống coi nó là chưa xong và giao lại cho một worker khác. Kết quả là người dùng nhận được email lần thứ hai, và đó chính là hệ quả của bảo đảm giao ít nhất một lần. Cách sửa không nằm ở hàng đợi mà nằm ở chính consumer của chúng ta. Chúng ta lưu lại một dấu hiệu rằng công việc mang định danh này đã gửi rồi, và chúng ta kiểm tra dấu hiệu đó trước khi gửi. Cách khác là dùng khoá chống trùng mà nhà cung cấp email hỗ trợ, để chính họ chặn bản sao thứ hai.

Chúng ta nên gói bài này thành hai câu hỏi review cố định. Câu thứ nhất là công việc này chạy hai lần thì có gây hại hay không. Câu thứ hai là hàng đợi này có nằm trên một instance đang bật cơ chế đuổi key hay không. Công cụ AI thường giả định rằng hệ thống bảo đảm chạy đúng một lần, vì vậy nó viết consumer không hề chống trùng. Đây là loại lỗi chỉ lộ ra khi có sự cố thật, tức là đúng lúc chúng ta ít muốn phát hiện nó nhất.

**English (bám cấu trúc tiếng Việt)**

The challenge scenario of this lesson is very simple but very deep. A worker sends the email successfully, and then it dies right before it can acknowledge the job. Because the job has not been acknowledged, the system treats it as unfinished and hands it to another worker. The result is that the user receives the email a second time, and that is exactly the consequence of the at-least-once guarantee. The fix does not lie in the queue but lies in our own consumer. We store a marker saying that the job carrying this identifier has already been sent, and we check that marker before sending. Another way is to use a deduplication key that the email provider supports, so that they themselves block the second copy.

We should wrap this lesson into two standing review questions. The first is whether this job does harm if it runs twice. The second is whether this queue sits on an instance that has the eviction mechanism switched on. AI tools usually assume that the system guarantees exactly-once execution, therefore they write consumers with no deduplication at all. This is the kind of bug that only shows up during a real incident, that is exactly when we least want to discover it.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| rất đơn giản nhưng rất sâu | very simple but very deep |
| ngay trước khi kịp xác nhận công việc | right before it can acknowledge the job |
| hệ thống coi nó là chưa xong | the system treats it as unfinished |
| nhận được email lần thứ hai | receives the email a second time |
| cách sửa không nằm ở hàng đợi mà nằm ở | the fix does not lie in the queue but lies in |
| chúng ta kiểm tra dấu hiệu đó trước khi gửi | we check that marker before sending |
| để chính họ chặn bản sao thứ hai | so that they themselves block the second copy |
| hai câu hỏi review cố định | two standing review questions |
| chạy hai lần thì có gây hại hay không | does harm if it runs twice |
| consumer không hề chống trùng | consumers with no deduplication at all |
| chỉ lộ ra khi có sự cố thật | only shows up during a real incident |
| đúng lúc chúng ta ít muốn phát hiện nó nhất | exactly when we least want to discover it |

**Thuật ngữ cần nhớ**

- chống trùng lặp → **deduplication**
- sự cố thật → **a real incident**
- bảo đảm chạy đúng một lần → **an exactly-once guarantee**
- chưa hoàn thành → **unfinished**
- câu hỏi review cố định → **a standing review question**

---

## Mô hình ghi nhớ

**Tiếng Việt**

Hàng đợi tách việc chậm khỏi request, làm phẳng đợt tải và cho phép thử lại, vì vậy chúng ta dùng BullMQ chạy trên Redis Streams cho hệ Node. Nhưng bảo đảm ở đây chỉ là giao ít nhất một lần, nên mọi công việc phải lặp lại được mà không gây hại, và hàng đợi phải nằm ở một instance riêng không bật cơ chế đuổi key.

**English (bám cấu trúc tiếng Việt)**

A queue moves slow work out of the request, flattens spikes in load and allows retries, therefore we use BullMQ running on Redis Streams for the Node world. But the guarantee here is only at-least-once, so every job must be repeatable without doing harm, and the queue must sit on a separate instance with the eviction mechanism switched off.

---

## Bảng thuật ngữ tổng hợp

| Tiếng Việt | Tiếng Anh | Ghi chú phát âm (Anh-Anh) |
|---|---|---|
| hàng đợi công việc | a job queue | queue /kjuː/ — đọc đúng như chữ cái **Q** |
| tiến trình xử lý việc | a worker | |
| bên tạo việc và bên xử lý việc | the producer and the consumer | consumer — Anh-Anh kən-**SYOO**-mə |
| tách rời phụ thuộc | to decouple | dee-**KUP**-əl |
| chuyển mã | to transcode | |
| đợt tải tăng đột ngột | a spike in load | |
| một đợt dồn | a burst | /bɜːst/ — kết thúc bằng cụm /st/ |
| bộ đệm | a buffer | **BUF**-ə |
| bản kế nhiệm | the successor | sək-**SES**-ə — trọng âm âm tiết thứ hai |
| trạng thái công việc | a job state | |
| bị hoãn | delayed | di-**LAYD** |
| mức ưu tiên | priority | prai-**OR**-ə-ti — âm đầu là /praɪ/ |
| theo lịch | on a schedule | Anh-Anh **SHED**-yool — khác Mỹ "sked-jool" |
| bảng điều khiển | a dashboard | |
| xác thực | authentication | aw-then-ti-**KAY**-shən — âm /θ/ ở giữa |
| số việc chạy song song | concurrency | kən-**KUR**-ən-si |
| nhật ký chỉ ghi thêm | an append-only log | ə-**PEND** — trọng âm cuối |
| nhóm consumer | a consumer group | |
| xác nhận đã xử lý | to acknowledge | ək-**NOL**-ij — chữ **k** đầu câm |
| đọc lại lịch sử | to replay the history | |
| giao ít nhất một lần | at-least-once delivery | |
| bảo đảm chạy đúng một lần | an exactly-once guarantee | guarantee — ga-rən-**TEE**, trọng âm cuối |
| nhận lấy công việc | to claim a job | |
| công việc bị treo | a stalled job | stalled /stɔːld/ — vần với "called" |
| thu hồi lại | to reclaim | ri-**KLAYM** |
| lặp lại không gây hại | idempotent | ai-**DEM**-pə-tənt |
| chống trùng lặp | deduplication | dee-dyoo-pli-**KAY**-shən |
| khoá chống trùng | a deduplication key | |
| cơn bão thử lại | a retry storm | |
| hàng đợi thư chết | a dead letter queue (DLQ) | |
| lùi theo cấp số nhân | exponential backoff | ek-spə-**NEN**-shəl |
| lượng ngẫu nhiên thêm vào | jitter | **JIT**-ə |
| cảnh báo | an alert | ə-**LERT** |
| dòng sự kiện | an event stream | i-**VENT** |
| lưu bền lâu dài | stored durably | **DYOO**-rə-bli — Anh-Anh có /dj/ |
| thông lượng | throughput | **THROO**-put — âm /θ/ đầu lưỡi |
| định tuyến | routing | Anh-Anh **ROO**-ting — khác Mỹ "rau-ting" |
| theo dõi hệ thống | to monitor | **MON**-i-tə |
| chi phí vận hành | the operational cost | |
| hạ tầng | infrastructure | **IN**-frə-struk-chə |
| chính sách không đuổi key | a no-eviction policy | eviction — i-**VIK**-shən |
| kiểu hỏng | a failure mode | |
| chuỗi kết nối | a connection string | |
| suy giảm nhẹ | to degrade gently | di-**GREID** |
| lưu bền | persistence | pə-**SIS**-təns |
| sự cố thật | a real incident | **IN**-si-dənt |
| chưa hoàn thành | unfinished | |
| bộ nhớ đệm | cache | /kæʃ/ — đọc y hệt "cash" |

---

## Bài nói

> **Cách luyện:** bật ghi âm, nói **60–90 giây** cho mỗi đề, **không nhìn tài liệu**. Mỗi bài nói phải dùng ít nhất **sáu thuật ngữ** trong bảng trên. Nghe lại một lần, đánh dấu chỗ bạn ngập ngừng, rồi nói lại đúng đề đó thêm một lần nữa.

1. **Explain to a junior** the three classes of problem a job queue solves, giving one concrete example for each.

2. **Explain the difference** between a list-based queue and Redis Streams, and say exactly what a consumer group gives you that a list cannot.

3. **A teammate wrote a consumer** that assumes each job runs exactly once, because the queue library "handles that". Explain why you would push back and what you would ask them to add.

4. **Describe what happens** when a worker dies in the middle of a job, and how the system gets that job back to another worker.

5. **Someone on your team wants** unlimited retries so that no job is ever lost. Explain why you would push back and what you would configure instead.

6. **When is BullMQ not enough** and you would move to Kafka? Include the price of that move in your answer.

7. **A colleague put the queue** on the same Redis as the cache with an evict-all-keys policy, to save on cost. Explain the two opposite risks and what you would set up instead.
