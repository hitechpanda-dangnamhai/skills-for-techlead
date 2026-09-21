# Bài 12 — Distributed Tracing & Observability
### Bản song ngữ: tiếng Anh bám cấu trúc tiếng Việt

> **Cách dùng bài này:** đọc đoạn tiếng Việt trước → tự dịch trong đầu, không nhìn xuống → so sánh với đoạn tiếng Anh ngay bên dưới → **đọc to bản tiếng Anh hai lần**: lần một đọc chậm cho đúng từng âm, lần hai đọc liền mạch như đang nói với người phỏng vấn.

---

## ① Vì sao gỡ lỗi ở hệ phân tán khó hơn hẳn

**Tiếng Việt**

Đây là bài chốt của cả chủ đề, vì sau khi một request đi xuyên nhiều service và đi qua broker, chúng ta cần một cách nhìn xuyên toàn hệ. Trong một monolith, mọi việc xảy ra trong cùng một tiến trình, nên chúng ta mở một tệp log là thấy đủ câu chuyện. Trong hệ phân tán, cùng một request để lại dấu vết rải rác ở mười nơi khác nhau, và mỗi nơi chỉ giữ một mẩu. Tệ hơn nữa, các mẩu đó không có gì nối với nhau, nên chúng ta không dựng lại được đường đi từ đầu tới cuối. Khi có sự cố, chúng ta rơi vào cảnh mò kim đáy bể và đội trực phải đoán mò.

Theo dõi phân tán sinh ra để giải đúng bài toán đó. Ý tưởng gồm hai phần rất đơn giản và chúng ta nên nói được cả hai trong một câu. Phần thứ nhất là gắn cho mỗi request một mã duy nhất và mang mã đó đi cùng nó qua mọi chặng. Phần thứ hai là chia hành trình thành nhiều đoạn công việc, mỗi đoạn ghi lại thời điểm bắt đầu và thời lượng của nó. Nhờ hai phần đó, chúng ta dựng lại được một dòng thời gian đầy đủ và chỉ ra được chặng nào chậm hoặc chặng nào hỏng.

**English (bám cấu trúc tiếng Việt)**

This is the closing lesson of the whole topic, because after a request travels through many services and goes through a broker, we need a way to see across the whole system. In a monolith, everything happens inside the same process, so we open one log file and we see the full story. In a distributed system, the same request leaves traces scattered across ten different places, and each place keeps only one fragment. Even worse, those fragments have nothing linking them together, so we cannot rebuild the path from beginning to end. When an incident happens, we end up looking for a needle in a haystack and the on-call team has to guess.

Distributed tracing was born to solve exactly that problem. The idea has two very simple parts and we should be able to say both of them in one sentence. The first part is to attach a unique id to each request and to carry that id along with it through every hop. The second part is to divide the journey into many units of work, and each unit records its start time and its duration. Thanks to those two parts, we can rebuild a complete timeline and point out which hop is slow or which hop is failing.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| bài chốt của cả chủ đề | the closing lesson of the whole topic |
| một cách nhìn xuyên toàn hệ | a way to see across the whole system |
| chúng ta mở một tệp log là thấy đủ câu chuyện | we open one log file and we see the full story |
| để lại dấu vết rải rác ở mười nơi khác nhau | leaves traces scattered across ten different places |
| mỗi nơi chỉ giữ một mẩu | each place keeps only one fragment |
| không có gì nối với nhau | have nothing linking them together |
| chúng ta rơi vào cảnh mò kim đáy bể | we end up looking for a needle in a haystack |
| đội trực phải đoán mò | the on-call team has to guess |
| sinh ra để giải đúng bài toán đó | was born to solve exactly that problem |
| mang mã đó đi cùng nó qua mọi chặng | carry that id along with it through every hop |
| chia hành trình thành nhiều đoạn công việc | divide the journey into many units of work |
| chỉ ra được chặng nào chậm | point out which hop is slow |

**Thuật ngữ cần nhớ**

- theo dõi phân tán → **distributed tracing**
- khả năng quan sát → **observability**
- dòng thời gian → **a timeline**
- một chặng trong hành trình → **a hop**
- đội trực sự cố → **the on-call team**

---

## ② Trace và span: bản đồ của một request

**Tiếng Việt**

Chúng ta cần dùng đúng hai từ chuyên môn khi nói về chủ đề này, và người phỏng vấn nghe rất kỹ hai từ đó. Từ thứ nhất là trace, tức là toàn bộ hành trình của đúng một request từ lúc vào tới lúc ra. Từ thứ hai là span, tức là một đơn vị công việc bên trong hành trình đó, ví dụ một lời gọi HTTP hoặc một truy vấn cơ sở dữ liệu. Mỗi span có một span cha, có thời lượng riêng, và có các nhãn mô tả nó làm gì. Vì vậy một trace thực chất là một cây các span, chứ không phải một danh sách phẳng.

Cấu trúc cây này chính là thứ giúp chúng ta đọc được nguyên nhân chỉ trong vài giây. Khi mở một trace chậm, chúng ta nhìn ngay ra span nào chiếm phần lớn thời lượng của cả cây. Nếu span đó là một truy vấn cơ sở dữ liệu, chúng ta biết vấn đề nằm ở chỉ mục chứ không nằm ở mạng. Nếu span đó là một lời gọi sang service khác, chúng ta biết phải đi hỏi đội nào và mang theo bằng chứng gì. Nếu không có cấu trúc này, một cuộc điều tra như vậy thường mất cả buổi chiều và kết thúc bằng một phỏng đoán.

**English (bám cấu trúc tiếng Việt)**

We need to use exactly two technical words when we talk about this topic, and interviewers listen very carefully for those two words. The first word is a trace, that is, the entire journey of exactly one request from the moment it enters to the moment it leaves. The second word is a span, that is, one unit of work inside that journey, for example one HTTP call or one database query. Each span has a parent span, has its own duration, and has tags describing what it does. Therefore a trace is in fact a tree of spans, rather than a flat list.

This tree structure is exactly the thing that lets us read the cause in only a few seconds. When we open a slow trace, we see immediately which span takes up most of the duration of the whole tree. If that span is a database query, we know the problem lies in the indexing rather than lying in the network. If that span is a call over to another service, we know which team to go and ask and what evidence to bring along. Without this structure, an investigation like that usually takes a whole afternoon and ends with a guess.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| người phỏng vấn nghe rất kỹ hai từ đó | interviewers listen very carefully for those two words |
| từ lúc vào tới lúc ra | from the moment it enters to the moment it leaves |
| một đơn vị công việc bên trong hành trình đó | one unit of work inside that journey |
| có các nhãn mô tả nó làm gì | has tags describing what it does |
| một cây các span chứ không phải một danh sách phẳng | a tree of spans rather than a flat list |
| đọc được nguyên nhân chỉ trong vài giây | read the cause in only a few seconds |
| chiếm phần lớn thời lượng của cả cây | takes up most of the duration of the whole tree |
| nằm ở chỉ mục chứ không nằm ở mạng | lies in the indexing rather than lying in the network |
| phải đi hỏi đội nào | which team to go and ask |
| mang theo bằng chứng gì | what evidence to bring along |
| kết thúc bằng một phỏng đoán | ends with a guess |

**Thuật ngữ cần nhớ**

- toàn bộ hành trình một request → **a trace**
- một đơn vị công việc → **a span**
- span cha và span con → **parent span and child span**
- thời lượng → **duration**
- nhãn mô tả → **a tag**

---

## ③ Truyền ngữ cảnh, và chỗ trace hay bị đứt

**Tiếng Việt**

Để mọi span thuộc về cùng một trace, chúng ta phải truyền mã trace qua mọi biên mà request đi qua. Với lời gọi HTTP, chuẩn hiện nay là một header có tên `traceparent` theo đặc tả của W3C. Bên gọi bơm mã trace vào header đó, còn bên nhận đọc header ra và tạo span con nối vào đúng span cha. Phần này thường được các thư viện tự động lo, nên nhiều đội bật thư viện lên là thấy trace hiện ra ngay. Chính vì dễ như vậy nên rất nhiều người tưởng rằng công việc đã xong.

Chỗ trace hay bị đứt nhất lại nằm ở broker, và đây là điểm chúng ta nên chủ động nêu ra trong phỏng vấn. Khi producer phát một message, ngữ cảnh trace không tự đi theo message trừ khi chúng ta chủ động đính nó vào header của message. Nếu chúng ta quên bước đó, consumer sẽ mở một trace hoàn toàn mới, nên hành trình bị cắt làm đôi tại hàng đợi. Trên bảng điều khiển, chúng ta sẽ thấy mọi trace đều kết thúc đúng ở chỗ phát message, và các consumer thì có những trace rời rạc không cha không mẹ. Cách chữa gồm hai bước, đó là bơm ngữ cảnh vào header message ở phía phát, và trích ngữ cảnh đó ra rồi mở span con ở phía nhận.

**English (bám cấu trúc tiếng Việt)**

For every span to belong to the same trace, we have to propagate the trace id across every boundary that the request crosses. For HTTP calls, the current standard is a header named `traceparent` following the W3C specification. The caller injects the trace id into that header, while the receiver reads the header out and creates a child span linked to the right parent span. This part is usually handled automatically by the libraries, so many teams switch the library on and see traces appear straight away. Precisely because it is that easy, very many people assume the work is finished.

The place where the trace most often breaks is at the broker, and this is a point we should raise ourselves in an interview. When a producer publishes a message, the trace context does not travel with the message unless we deliberately attach it into the message headers. If we forget that step, the consumer will open a completely new trace, so the journey is cut in half at the queue. On the dashboard, we will see every trace ending exactly at the point of publishing, while the consumers have separate traces with no parents at all. The cure consists of two steps, which are injecting the context into the message headers on the publishing side, and extracting that context and opening a child span on the receiving side.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| qua mọi biên mà request đi qua | across every boundary that the request crosses |
| theo đặc tả của W3C | following the W3C specification |
| bên gọi bơm mã trace vào header đó | the caller injects the trace id into that header |
| nối vào đúng span cha | linked to the right parent span |
| bật thư viện lên là thấy trace hiện ra ngay | switch the library on and see traces appear straight away |
| chính vì dễ như vậy nên | precisely because it is that easy |
| chỗ trace hay bị đứt nhất | the place where the trace most often breaks |
| trừ khi chúng ta chủ động đính nó vào | unless we deliberately attach it into |
| hành trình bị cắt làm đôi tại hàng đợi | the journey is cut in half at the queue |
| những trace rời rạc không cha không mẹ | separate traces with no parents at all |
| trích ngữ cảnh đó ra | extracting that context |

**Thuật ngữ cần nhớ**

- truyền ngữ cảnh qua các chặng → **context propagation**
- bơm vào và trích ra → **to inject and to extract**
- header của message → **the message headers**
- đặc tả chuẩn → **a specification**
- trace bị đứt → **a broken trace**

---

## ④ Correlation id, trace id và log có cấu trúc

**Tiếng Việt**

Nhiều người nhầm mã tương quan với mã trace, nên chúng ta nên phân biệt hai thứ cho rõ. Mã tương quan là mã mà chúng ta gắn vào mọi dòng log thuộc cùng một luồng nghiệp vụ. Nhờ nó, chúng ta lọc log ra và đọc được toàn bộ câu chuyện dưới dạng chữ, kể cả những chi tiết mà trace không ghi. Mã trace thì phục vụ hệ thống theo dõi và cho chúng ta hình dạng cùng thời lượng của hành trình. Hai mã này bổ trợ nhau chứ không thay thế nhau, và cách làm tốt nhất là đưa cả hai vào mọi dòng log.

Để lọc và tổng hợp được, log của chúng ta phải có cấu trúc chứ không phải là những câu chữ tự do. Điều đó nghĩa là mỗi dòng log là một đối tượng dữ liệu với các trường rõ ràng, thường là định dạng JSON. Khi log đã có cấu trúc, chúng ta truy vấn được theo mã trace, theo mã người dùng, hoặc theo mã đơn hàng chỉ trong một câu lệnh. Nếu log chỉ là chữ tự do, chúng ta phải viết biểu thức chính quy cho từng dịp và kết quả thì không đáng tin. Vì vậy log có cấu trúc là điều kiện cần để nối được thế giới log với thế giới trace.

**English (bám cấu trúc tiếng Việt)**

Many people confuse the correlation id with the trace id, so we should distinguish these two things clearly. The correlation id is the id that we attach to every log line belonging to the same business flow. Thanks to it, we filter the logs out and read the whole story in text form, including the details that the trace does not record. The trace id serves the tracing system and gives us the shape and the duration of the journey. These two ids complement each other rather than replacing each other, and the best practice is to put both of them into every log line.

In order to filter and aggregate, our logs must be structured rather than being free-form sentences. That means that each log line is a data object with clear fields, usually in JSON format. When the logs are structured, we can query by trace id, by user id, or by order id in a single statement. If the logs are only free text, we have to write a regular expression for each occasion and the result is not trustworthy. Therefore structured logging is a necessary condition for connecting the world of logs with the world of traces.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| nhầm A với B | confuse A with B |
| mọi dòng log thuộc cùng một luồng nghiệp vụ | every log line belonging to the same business flow |
| chúng ta lọc log ra | we filter the logs out |
| dưới dạng chữ | in text form |
| kể cả những chi tiết mà trace không ghi | including the details that the trace does not record |
| hình dạng cùng thời lượng của hành trình | the shape and the duration of the journey |
| cách làm tốt nhất là | the best practice is |
| chứ không phải là những câu chữ tự do | rather than being free-form sentences |
| chỉ trong một câu lệnh | in a single statement |
| viết biểu thức chính quy cho từng dịp | write a regular expression for each occasion |
| là điều kiện cần để nối được | is a necessary condition for connecting |

**Thuật ngữ cần nhớ**

- mã tương quan → **a correlation id**
- log có cấu trúc → **structured logging**
- trường dữ liệu → **a field**
- truy vấn và tổng hợp → **to query and aggregate**
- điều kiện cần → **a necessary condition**

---

## ⑤ OpenTelemetry và chuyện thoát khỏi khoá nhà cung cấp

**Tiếng Việt**

Trước đây mỗi nhà cung cấp công cụ giám sát lại đưa ra một bộ thư viện riêng để gắn vào mã nguồn. Hậu quả là khi chúng ta muốn đổi nhà cung cấp, chúng ta phải sửa lại phần gắn thư viện ở hàng trăm chỗ. Đó chính là khoá nhà cung cấp, và nó khiến việc đàm phán lại hợp đồng trở nên rất khó khăn. OpenTelemetry ra đời để tách phần sinh dữ liệu ra khỏi phần lưu trữ và hiển thị dữ liệu. Chúng ta gắn thư viện trung lập một lần, rồi chúng ta xuất dữ liệu tới bất kỳ hệ lưu trữ nào mà chúng ta chọn.

Chúng ta nên phân biệt rõ vai trò của hai lớp này khi trả lời phỏng vấn. OpenTelemetry là chuẩn và bộ thư viện sinh ra trace, số liệu đo và log ngay bên trong ứng dụng của chúng ta. Jaeger, Tempo hoặc các sản phẩm thương mại là nơi lưu trữ những dữ liệu đó và cho chúng ta giao diện để tra cứu. Nói cách khác, một bên tạo dữ liệu còn bên kia chứa dữ liệu, và ranh giới đó chính là thứ bảo vệ chúng ta. Nhờ ranh giới này, khi giá của một nhà cung cấp tăng gấp đôi, chúng ta đổi đích xuất dữ liệu chứ không phải viết lại ứng dụng.

**English (bám cấu trúc tiếng Việt)**

In the past each monitoring vendor put out its own set of libraries to embed into the source code. The consequence was that when we wanted to change vendor, we had to rework the library embedding in hundreds of places. That is exactly vendor lock-in, and it makes renegotiating a contract very difficult. OpenTelemetry came about in order to separate the part that produces the data from the part that stores and displays the data. We embed the neutral library once, and then we export the data to whichever storage system we choose.

We should clearly distinguish the roles of these two layers when we answer in an interview. OpenTelemetry is the standard and the set of libraries that produce traces, metrics and logs right inside our application. Jaeger, Tempo or the commercial products are where those data are stored and where we get an interface to look them up. In other words, one side creates the data while the other side holds the data, and that boundary is exactly the thing that protects us. Thanks to this boundary, when the price of one vendor doubles, we change the export destination rather than rewriting the application.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| lại đưa ra một bộ thư viện riêng | put out its own set of libraries |
| gắn vào mã nguồn | to embed into the source code |
| sửa lại phần gắn thư viện ở hàng trăm chỗ | rework the library embedding in hundreds of places |
| việc đàm phán lại hợp đồng | renegotiating a contract |
| ra đời để tách A ra khỏi B | came about in order to separate A from B |
| chúng ta gắn thư viện trung lập một lần | we embed the neutral library once |
| bất kỳ hệ lưu trữ nào mà chúng ta chọn | whichever storage system we choose |
| ngay bên trong ứng dụng của chúng ta | right inside our application |
| cho chúng ta giao diện để tra cứu | gives us an interface to look them up |
| ranh giới đó chính là thứ bảo vệ chúng ta | that boundary is exactly the thing that protects us |
| chúng ta đổi đích xuất dữ liệu | we change the export destination |

**Thuật ngữ cần nhớ**

- bộ chuẩn trung lập cho quan sát → **OpenTelemetry**
- khoá vào một nhà cung cấp → **vendor lock-in**
- việc gắn thư viện đo đạc vào mã → **instrumentation**
- xuất dữ liệu sang hệ khác → **to export**
- hệ lưu trữ phía sau → **a backend**

---

## ⑥ Lấy mẫu: quyết ở đầu luồng hay cuối luồng

**Tiếng Việt**

Giữ lại toàn bộ trace của mọi request nghe rất hấp dẫn, nhưng chi phí lưu trữ và băng thông sẽ tăng rất nhanh. Vì vậy chúng ta lấy mẫu, nghĩa là chúng ta chỉ giữ lại một phần trong tổng số trace. Cách thứ nhất là quyết định ngay ở đầu luồng, tức là ngay khi request vừa vào hệ thống. Cách này rất rẻ và rất đơn giản, vì chúng ta chỉ cần một phép quay số ngẫu nhiên ở chặng đầu tiên. Nhưng nó có một điểm yếu rõ ràng, đó là lúc quyết định thì chúng ta chưa biết request này rồi sẽ hỏng hay sẽ chậm.

Cách thứ hai là quyết định ở cuối luồng, tức là sau khi toàn bộ trace đã chạy xong. Nhờ vậy chúng ta giữ lại đúng những trace thú vị nhất, gồm các trace bị lỗi và các trace chậm bất thường. Cái giá là hệ thống phải giữ tạm mọi span trong bộ nhớ đệm cho tới khi trace kết thúc, nên chi phí hạ tầng cao hơn. Vì vậy lựa chọn giữa hai cách chính là một đánh đổi giữa chi phí và khả năng giữ lại bằng chứng quan trọng. Trong thực tế, nhiều đội bắt đầu bằng cách rẻ, rồi chuyển sang cách kia khi họ đã thấm cảm giác mất đúng cái trace mà họ cần.

**English (bám cấu trúc tiếng Việt)**

Keeping the whole trace of every request sounds very attractive, but the storage cost and the bandwidth will rise very fast. Therefore we sample, which means that we only keep a portion of the total number of traces. The first way is to decide right at the head of the flow, that is, right when the request has just entered the system. This way is very cheap and very simple, because we only need one random draw at the first hop. But it has one clear weakness, which is that at the moment of deciding we do not yet know whether this request will fail or will be slow.

The second way is to decide at the tail of the flow, that is, after the whole trace has finished running. Thanks to that, we keep exactly the most interesting traces, including the traces that failed and the traces that were unusually slow. The price is that the system has to hold every span temporarily in a buffer until the trace ends, so the infrastructure cost is higher. Therefore the choice between the two ways is exactly a trade-off between cost and the ability to keep important evidence. In practice, many teams start with the cheap way, and then move to the other one once they have felt what it is like to lose exactly the trace they needed.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| chi phí lưu trữ và băng thông | the storage cost and the bandwidth |
| chúng ta chỉ giữ lại một phần | we only keep a portion |
| quyết định ngay ở đầu luồng | decide right at the head of the flow |
| một phép quay số ngẫu nhiên | one random draw |
| nó có một điểm yếu rõ ràng | it has one clear weakness |
| chưa biết request này rồi sẽ hỏng hay sẽ chậm | do not yet know whether this request will fail or will be slow |
| các trace chậm bất thường | the traces that were unusually slow |
| giữ tạm mọi span trong bộ nhớ đệm | hold every span temporarily in a buffer |
| khả năng giữ lại bằng chứng quan trọng | the ability to keep important evidence |
| bắt đầu bằng cách rẻ | start with the cheap way |
| khi họ đã thấm cảm giác mất đúng cái trace mà họ cần | once they have felt what it is like to lose exactly the trace they needed |

**Thuật ngữ cần nhớ**

- lấy mẫu → **sampling**
- quyết ở đầu luồng → **head-based sampling**
- quyết ở cuối luồng → **tail-based sampling**
- băng thông → **bandwidth**
- bộ nhớ đệm tạm → **a buffer**

---

## ⑦ Ba trụ cột và quy trình khoanh vùng khi cảnh báo nổ

**Tiếng Việt**

Khả năng quan sát đứng trên ba trụ cột, và mỗi trụ cột trả lời một câu hỏi khác nhau. Số liệu đo trả lời câu hỏi có gì bất thường và bất thường ở đâu, ví dụ tỷ lệ lỗi hoặc độ trễ ở phân vị chín mươi chín. Trace trả lời câu hỏi request đi qua đâu và chặng nào chậm hoặc hỏng. Log trả lời câu hỏi vì sao chặng đó hỏng, vì nó chứa chi tiết mà hai trụ cột kia không giữ. Nhiều người dùng lẫn lộn ba thứ này, ví dụ họ cố cảnh báo dựa trên log hoặc cố tìm nguyên nhân bằng số liệu đo.

Vì vậy chúng ta nên thuộc một quy trình gồm ba bước và nói ra được nó khi bị hỏi về cách xử lý sự cố. Bước một, số liệu đo phát hiện vấn đề và bắn cảnh báo, vì nó rẻ nên chúng ta thu thập nó liên tục. Bước hai, chúng ta mở trace để khoanh vùng chặng có vấn đề, thay vì đoán mò xem service nào có lỗi. Bước ba, chúng ta lọc log theo mã trace của chặng đó để đọc chi tiết và tìm ra nguyên nhân gốc. Khi một đồng nghiệp nói rằng đội đã bật tự động đo đạc và có bảng điều khiển đẹp nên coi như xong phần quan sát, chúng ta nên hỏi lại một câu rất cụ thể, đó là hãy mở cho tôi xem một trace đi từ cổng vào tới tận consumer cuối cùng. Nếu trace đó dừng ở chỗ phát message, thì chúng ta chưa có khả năng quan sát mà chỉ mới có một bảng điều khiển đẹp.

**English (bám cấu trúc tiếng Việt)**

Observability stands on three pillars, and each pillar answers a different question. Metrics answer the question of whether something is abnormal and where it is abnormal, for example the error rate or the latency at the ninety-ninth percentile. Traces answer the question of where the request went and which hop is slow or failing. Logs answer the question of why that hop failed, because they hold the details that the other two pillars do not keep. Many people mix these three things up, for example they try to alert based on logs or try to find the cause using metrics.

Therefore we should know a three-step procedure by heart and be able to say it out loud when we are asked about incident handling. Step one, metrics detect the problem and fire the alert, because they are cheap so we collect them continuously. Step two, we open a trace to narrow down the problematic hop, instead of guessing which service has the fault. Step three, we filter the logs by the trace id of that hop in order to read the details and find the root cause. When a colleague says that the team has switched on automatic instrumentation and has a beautiful dashboard so observability can be considered done, we should ask back one very concrete question, which is please show me one trace going from the gateway all the way to the last consumer. If that trace stops at the point of publishing, then we do not have observability yet but only have a beautiful dashboard.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| đứng trên ba trụ cột | stands on three pillars |
| có gì bất thường và bất thường ở đâu | whether something is abnormal and where it is abnormal |
| độ trễ ở phân vị chín mươi chín | the latency at the ninety-ninth percentile |
| chứa chi tiết mà hai trụ cột kia không giữ | hold the details that the other two pillars do not keep |
| nhiều người dùng lẫn lộn ba thứ này | many people mix these three things up |
| bắn cảnh báo | fire the alert |
| khoanh vùng chặng có vấn đề | narrow down the problematic hop |
| thay vì đoán mò xem service nào có lỗi | instead of guessing which service has the fault |
| lọc log theo mã trace | filter the logs by the trace id |
| coi như xong phần quan sát | observability can be considered done |
| đi từ cổng vào tới tận consumer cuối cùng | going from the gateway all the way to the last consumer |
| chỉ mới có một bảng điều khiển đẹp | only have a beautiful dashboard |

**Thuật ngữ cần nhớ**

- ba trụ cột → **the three pillars**
- số liệu đo → **metrics**
- tỷ lệ lỗi → **the error rate**
- bảng điều khiển → **a dashboard**
- nguyên nhân gốc → **the root cause**

---

## ⑧ Mô hình ghi nhớ

**Tiếng Việt**

Một mã trace đi xuyên suốt cộng với một span cho mỗi chặng chính là tấm bản đồ của một request, và chúng ta không được để trace đứt ở hàng đợi vì phải truyền ngữ cảnh qua header của message. Khi cảnh báo nổ, số liệu đo phát hiện, trace khoanh vùng, còn log soi chi tiết; chúng ta dùng OpenTelemetry để không bị khoá vào một nhà cung cấp.

**English (bám cấu trúc tiếng Việt)**

One trace id running all the way through plus one span for each hop is exactly the map of a request, and we must not let the trace break at the queue because we have to propagate the context through the message headers. When an alert fires, metrics detect, traces narrow down, and logs examine the details; we use OpenTelemetry so that we are not locked into one vendor.

---

## ⑨ Bảng thuật ngữ tổng hợp

| Tiếng Việt | Tiếng Anh | Ghi chú phát âm |
|---|---|---|
| khả năng quan sát | observability | ob-zə-və-**BI**-li-ty, sáu âm tiết |
| theo dõi phân tán | distributed tracing | *tracing* = **TRAY**-sing |
| dòng thời gian | a timeline | |
| một chặng trong hành trình | a hop | |
| đội trực sự cố | the on-call team | |
| toàn bộ hành trình một request | a trace | |
| một đơn vị công việc | a span | |
| span cha / span con | parent span / child span | *child* /tʃaɪld/ — số nhiều *children* đọc "CHIL-drən" |
| thời lượng | duration | dyu-**RAY**-shn, giọng Anh có /dj/ |
| nhãn mô tả | a tag | |
| truyền ngữ cảnh qua các chặng | context propagation | prop-ə-**GAY**-shn |
| bơm vào và trích ra | to inject and to extract | *inject* = in-**JEKT**; *extract* (động từ) = iks-**TRACT** |
| header của message | the message headers | |
| đặc tả chuẩn | a specification | spe-si-fi-**KAY**-shn, năm âm tiết |
| trace bị đứt | a broken trace | |
| mã tương quan | a correlation id | co-rə-**LAY**-shn |
| log có cấu trúc | structured logging | *structured* = **STRUC**-chəd, cụm *str-* phải bật rõ |
| trường dữ liệu | a field | /fiːld/ — nguyên âm dài |
| truy vấn và tổng hợp | to query and aggregate | *query* = **KWIƏ**-ri; *aggregate* (động từ) = **A**-gri-gate |
| biểu thức chính quy | a regular expression | iks-**PRE**-shn |
| điều kiện cần | a necessary condition | **NE**-sə-sə-ri, trọng âm âm đầu |
| bộ chuẩn trung lập cho quan sát | OpenTelemetry | o-pən-tə-**LE**-mə-tri |
| khoá vào một nhà cung cấp | vendor lock-in | *vendor* = **VEN**-də |
| gắn thư viện đo đạc vào mã | instrumentation | in-strə-men-**TAY**-shn, năm âm tiết |
| xuất dữ liệu sang hệ khác | to export | động từ = iks-**PORT**; danh từ = **EX**-port |
| hệ lưu trữ phía sau | a backend | |
| lấy mẫu | sampling | **SAM**-pling |
| quyết ở đầu luồng | head-based sampling | |
| quyết ở cuối luồng | tail-based sampling | |
| băng thông | bandwidth | **BAND**-width, âm cuối là /θ/ |
| bộ nhớ đệm tạm | a buffer | **BU**-fə |
| bằng chứng | evidence | **E**-vi-dəns, danh từ không đếm được |
| ba trụ cột | the three pillars | *pillar* = **PI**-lə; *three* — âm *th* /θ/ |
| số liệu đo | metrics | **ME**-trics |
| tỷ lệ lỗi | the error rate | |
| phân vị | a percentile | pə-**SEN**-tail |
| bảng điều khiển | a dashboard | |
| bắn cảnh báo | to fire an alert | *alert* = ə-**LERT** |
| khoanh vùng vấn đề | to narrow down the problem | *narrow* = **NA**-rəʊ |
| nguyên nhân gốc | the root cause | |
| chi phí lưu trữ | the storage cost | **STOR**-ij |

---

## ⑩ Bài nói

> **Cách luyện:** bật ghi âm, nói **60–90 giây** cho mỗi đề, **không nhìn tài liệu**. Mỗi bài nói phải dùng ít nhất **sáu thuật ngữ** trong bảng trên. Nói xong thì nghe lại một lượt, đánh dấu chỗ mình ngập ngừng, rồi nói lại chính đề đó thêm một lần nữa.

1. Explain to a junior developer why debugging one slow request is much harder in microservices than in a monolith, and describe what tracing adds.

2. Describe the relationship between a trace and a span, and walk through how you would use a trace tree to find where three seconds went.

3. Your team has auto-instrumentation switched on, but every trace ends at the point where the order service publishes to Kafka. Explain what is happening and how you would fix it.

4. A colleague says the correlation id in the logs makes tracing unnecessary. Explain where you agree and where you would push back.

5. Explain what problem OpenTelemetry solves, and describe the difference between the instrumentation layer and the backend.

6. When would you choose tail-based sampling over head-based sampling, and what cost would you have to justify to your team?

7. An alert has just fired for a rise in checkout errors. Walk through your three steps, naming which pillar you use at each step and what you expect it to tell you.
