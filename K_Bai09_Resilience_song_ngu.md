# Bài 9 — Resilience: circuit breaker, bulkhead, retry, timeout
### Bản song ngữ: tiếng Anh bám cấu trúc tiếng Việt

> **Cách dùng bài này:** đọc đoạn tiếng Việt trước → tự dịch trong đầu, không nhìn xuống → so sánh với đoạn tiếng Anh ngay bên dưới → **đọc to bản tiếng Anh hai lần**: lần một đọc chậm cho đúng từng âm, lần hai đọc liền mạch như đang nói với người phỏng vấn.

---

## ① Lỗi lan theo chuỗi: kẻ thù chính

**Tiếng Việt**

Bất đồng bộ đã gỡ được coupling thời gian, nhưng mọi lời gọi đồng bộ còn lại trong hệ thống vẫn cần được bảo vệ. Chúng ta bắt đầu bằng cách hiểu kẻ thù chính, đó là lỗi lan theo chuỗi. Kịch bản luôn bắt đầu giống nhau, đó là một service phía sau trở nên chậm chứ chưa hẳn là chết hẳn. Bên gọi gửi request rồi ngồi chờ, và mỗi request đang chờ giữ lại một luồng cùng một kết nối. Khi số request chờ đủ nhiều, bể luồng và bể kết nối của bên gọi cạn sạch, nên chính bên gọi cũng ngừng phục vụ được ai.

Điểm nguy hiểm nằm ở chỗ sự cố lan ngược lên trên chứ không dừng lại tại chỗ. Service gọi bên gọi kia cũng chờ, rồi bể tài nguyên của nó cũng cạn, và cứ thế lan tới tận cổng vào. Vì vậy một độ trễ nhỏ ở một service phụ có thể khuếch đại thành một sự cố toàn hệ thống trong vài phút. Đây chính là lý do các công cụ trong bài này tồn tại, và tất cả chúng phục vụ đúng hai mục tiêu. Mục tiêu thứ nhất là hỏng thật nhanh thay vì hỏng thật chậm, còn mục tiêu thứ hai là hỏng thật êm thay vì hỏng thật gắt.

**English (bám cấu trúc tiếng Việt)**

Asynchronous work has removed temporal coupling, but every remaining synchronous call in the system still needs to be protected. We start by understanding the main enemy, which is cascade failure. The scenario always begins the same way, which is that a downstream service becomes slow rather than actually dying. The caller sends a request and then sits waiting, and each waiting request holds on to one thread together with one connection. When enough requests are waiting, the thread pool and the connection pool of the caller run dry, so the caller itself can no longer serve anybody either.

The dangerous point lies in the fact that the incident spreads upwards rather than stopping where it started. The service calling that caller also waits, then its resource pool runs dry too, and so it goes all the way up to the gateway. Therefore a small latency in one minor service can be amplified into a system-wide incident within a few minutes. This is exactly why the tools in this lesson exist, and all of them serve exactly two goals. The first goal is to fail fast instead of failing slowly, while the second goal is to fail softly instead of failing harshly.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| mọi lời gọi đồng bộ còn lại | every remaining synchronous call |
| hiểu kẻ thù chính | understanding the main enemy |
| chậm chứ chưa hẳn là chết hẳn | slow rather than actually dying |
| ngồi chờ | sits waiting |
| giữ lại một luồng cùng một kết nối | holds on to one thread together with one connection |
| bể luồng và bể kết nối cạn sạch | the thread pool and the connection pool run dry |
| sự cố lan ngược lên trên | the incident spreads upwards |
| cứ thế lan tới tận cổng vào | and so it goes all the way up to the gateway |
| có thể khuếch đại thành | can be amplified into |
| hỏng thật nhanh thay vì hỏng thật chậm | fail fast instead of failing slowly |
| hỏng thật êm thay vì hỏng thật gắt | fail softly instead of failing harshly |

**Thuật ngữ cần nhớ**

- lỗi lan theo chuỗi → **cascade failure**
- cạn kiệt tài nguyên → **resource exhaustion**
- bể luồng → **the thread pool**
- bể kết nối → **the connection pool**
- service phía sau → **the downstream service**

---

## ② Cầu dao ngắt mạch và ba trạng thái của nó

**Tiếng Việt**

Cầu dao ngắt mạch hoạt động đúng như cái cầu dao điện trong nhà chúng ta, và nó có ba trạng thái. Trạng thái thứ nhất là đóng, nghĩa là mọi lời gọi vẫn đi qua bình thường trong khi cầu dao lặng lẽ đếm số lỗi. Khi tỷ lệ lỗi vượt ngưỡng đã đặt, cầu dao chuyển sang trạng thái mở. Ở trạng thái mở, cầu dao không gọi xuống service phía sau nữa mà trả lỗi ngay lập tức, hoặc trả một giá trị dự phòng. Sau một khoảng nguội, cầu dao chuyển sang trạng thái nửa mở và cho vài request thử đi qua, rồi nó đóng lại nếu các request đó thành công và mở tiếp nếu chúng vẫn hỏng.

Chúng ta nên giải thích được vì sao trạng thái mở lại có ích, vì đó là phần cốt lõi. Lợi ích thứ nhất là bên gọi không còn giữ luồng để chờ một service đang ốm, nên tài nguyên của chính nó được bảo toàn. Lợi ích thứ hai là service phía sau được nghỉ, vì nó không phải nhận thêm lưu lượng trong lúc nó đang cố hồi phục. Nói cách khác, cầu dao vừa bảo vệ bên gọi vừa bảo vệ bên bị gọi cùng một lúc. Nếu chúng ta không có cầu dao, hậu quả là chúng ta vừa tự treo mình vừa dìm luôn cái service đang ngắc ngoải.

**English (bám cấu trúc tiếng Việt)**

A circuit breaker works exactly like the electrical breaker in our house, and it has three states. The first state is closed, which means that every call still passes through normally while the breaker quietly counts the failures. When the failure rate goes beyond the threshold we have set, the breaker moves to the open state. In the open state, the breaker no longer calls down to the downstream service but returns an error immediately, or returns a fallback value. After a cooldown period, the breaker moves to the half-open state and lets a few trial requests through, and then it closes again if those requests succeed and opens again if they still fail.

We should be able to explain why the open state is useful, because that is the core part. The first benefit is that the caller no longer holds threads waiting for a sick service, so its own resources are preserved. The second benefit is that the downstream service gets a rest, because it does not have to take more traffic while it is trying to recover. In other words, the breaker protects the caller and protects the callee at the same time. If we do not have a breaker, the consequence is that we both hang ourselves up and drown the service that is already gasping.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| đúng như cái cầu dao điện trong nhà chúng ta | exactly like the electrical breaker in our house |
| trong khi cầu dao lặng lẽ đếm số lỗi | while the breaker quietly counts the failures |
| vượt ngưỡng đã đặt | goes beyond the threshold we have set |
| trả một giá trị dự phòng | returns a fallback value |
| sau một khoảng nguội | after a cooldown period |
| cho vài request thử đi qua | lets a few trial requests through |
| vì đó là phần cốt lõi | because that is the core part |
| giữ luồng để chờ một service đang ốm | holds threads waiting for a sick service |
| service phía sau được nghỉ | the downstream service gets a rest |
| trong lúc nó đang cố hồi phục | while it is trying to recover |
| chúng ta vừa tự treo mình vừa dìm luôn | we both hang ourselves up and drown |
| cái service đang ngắc ngoải | the service that is already gasping |

**Thuật ngữ cần nhớ**

- cầu dao ngắt mạch → **a circuit breaker**
- đóng, mở, nửa mở → **closed, open, half-open**
- tỷ lệ lỗi → **the failure rate**
- khoảng nguội → **the cooldown period**
- hỏng nhanh, trả lỗi ngay → **to fail fast**

---

## ③ Không có timeout thì cầu dao không bao giờ nhảy

**Tiếng Việt**

Đây là chi tiết mà người phỏng vấn dùng để phân biệt người hiểu thật với người học thuộc. Một cầu dao chỉ chuyển sang trạng thái mở khi nó đếm được đủ số lỗi, nên nó cần các lời gọi hỏng phải được ghi nhận là hỏng. Nếu chúng ta không đặt thời gian chờ tối đa, một request chậm vô hạn sẽ không bao giờ bị tính là lỗi. Khi đó cầu dao vẫn nằm im ở trạng thái đóng, trong khi các luồng của chúng ta lần lượt bị giữ hết. Nói cách khác, thời gian chờ tối đa chính là thứ biến chậm thành lỗi đếm được.

Từ đó chúng ta rút ra một nguyên tắc rất gọn cho mọi lời gọi qua mạng. Mọi lời gọi ra ngoài đều phải có thời gian chờ tối đa, và con số đó phải được chọn theo phân vị cao của độ trễ thực tế chứ không phải chọn bừa. Ba công cụ gồm thời gian chờ, cầu dao và thử lại luôn phải đi cùng nhau như một bộ ba. Nếu chúng ta chỉ có cầu dao mà thiếu thời gian chờ, cầu dao đó gần như vô dụng. Nếu chúng ta chỉ có thời gian chờ mà thiếu cầu dao, chúng ta vẫn tiếp tục ném request vào một service đã chết.

**English (bám cấu trúc tiếng Việt)**

This is the detail that interviewers use to separate the person who truly understands from the person who has memorised. A breaker only moves to the open state when it counts enough failures, so it needs the failed calls to be recorded as failures. If we do not set a maximum waiting time, an infinitely slow request will never be counted as a failure. At that point the breaker still sits still in the closed state, while our threads are held up one after another. In other words, the maximum waiting time is exactly the thing that turns slowness into a countable failure.

From that we draw a very short principle for every call over the network. Every outbound call must have a maximum waiting time, and that number must be chosen according to a high percentile of the real latency rather than picked at random. The three tools, namely the timeout, the breaker and the retry, must always go together as a trio. If we only have the breaker but lack the timeout, that breaker is nearly useless. If we only have the timeout but lack the breaker, we keep throwing requests at a service that is already dead.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| phân biệt người hiểu thật với người học thuộc | separate the person who truly understands from the person who has memorised |
| các lời gọi hỏng phải được ghi nhận là hỏng | the failed calls to be recorded as failures |
| một request chậm vô hạn | an infinitely slow request |
| vẫn nằm im ở trạng thái đóng | still sits still in the closed state |
| lần lượt bị giữ hết | are held up one after another |
| biến chậm thành lỗi đếm được | turns slowness into a countable failure |
| mọi lời gọi ra ngoài | every outbound call |
| theo phân vị cao của độ trễ thực tế | according to a high percentile of the real latency |
| chứ không phải chọn bừa | rather than picked at random |
| như một bộ ba | as a trio |
| gần như vô dụng | nearly useless |
| ném request vào một service đã chết | throwing requests at a service that is already dead |

**Thuật ngữ cần nhớ**

- thời gian chờ tối đa → **a timeout**
- lời gọi ra ngoài → **an outbound call**
- phân vị độ trễ → **a latency percentile**
- lỗi đếm được → **a countable failure**
- điều kiện tiên quyết → **a precondition**

---

## ④ Vách ngăn: cô lập tài nguyên theo từng bên phía sau

**Tiếng Việt**

Vách ngăn là một ẩn dụ mượn từ ngành đóng tàu, nơi thân tàu được chia thành nhiều khoang kín. Khi một khoang bị thủng và ngập nước, các khoang còn lại vẫn khô, nên con tàu vẫn nổi. Áp dụng vào phần mềm, chúng ta chia bể luồng và bể kết nối thành từng ngăn riêng cho từng service phía sau. Khi một service phía sau treo, chỉ ngăn dành riêng cho nó bị cạn, còn các ngăn khác vẫn phục vụ bình thường. Nhờ đó một dịch vụ gợi ý bị hỏng không thể nuốt hết tài nguyên chung và làm chết luôn luồng thanh toán.

Chúng ta nên nói rõ vách ngăn khác cầu dao ở đâu, vì hai thứ này rất hay bị gộp làm một. Cầu dao chặn các lời gọi đi tới một bên đang hỏng, nên nó làm việc ở tầng quyết định gọi hay không gọi. Vách ngăn thì cô lập tài nguyên, nên nó làm việc ở tầng phân bổ luồng và kết nối. Hai công cụ này bổ trợ nhau chứ không thay thế nhau, và một hệ nghiêm túc thường dùng cả hai. Nếu chúng ta chỉ có cầu dao, thì trong khoảng thời gian trước lúc cầu dao kịp nhảy, tài nguyên chung vẫn có thể bị hút cạn.

**English (bám cấu trúc tiếng Việt)**

The bulkhead is a metaphor borrowed from shipbuilding, where the hull of a ship is divided into many sealed compartments. When one compartment is holed and floods with water, the remaining compartments stay dry, so the ship still floats. Applying this to software, we divide the thread pool and the connection pool into separate compartments for each downstream service. When one downstream service hangs, only the compartment dedicated to it runs dry, while the other compartments still serve normally. Thanks to that, a broken recommendation service cannot swallow all the shared resources and kill the payment flow as well.

We should say clearly where the bulkhead differs from the breaker, because these two things are very often merged into one. The breaker blocks the calls going to a side that is failing, so it works at the layer of deciding whether to call or not to call. The bulkhead isolates resources, so it works at the layer of allocating threads and connections. These two tools complement each other rather than replacing each other, and a serious system usually uses both. If we only have the breaker, then during the period before the breaker manages to trip, the shared resources can still be drained.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| một ẩn dụ mượn từ ngành đóng tàu | a metaphor borrowed from shipbuilding |
| thân tàu được chia thành nhiều khoang kín | the hull of a ship is divided into many sealed compartments |
| bị thủng và ngập nước | is holed and floods with water |
| con tàu vẫn nổi | the ship still floats |
| ngăn dành riêng cho nó | the compartment dedicated to it |
| nuốt hết tài nguyên chung | swallow all the shared resources |
| rất hay bị gộp làm một | very often merged into one |
| ở tầng quyết định gọi hay không gọi | at the layer of deciding whether to call or not to call |
| ở tầng phân bổ luồng và kết nối | at the layer of allocating threads and connections |
| bổ trợ nhau chứ không thay thế nhau | complement each other rather than replacing each other |
| trước lúc cầu dao kịp nhảy | before the breaker manages to trip |
| vẫn có thể bị hút cạn | can still be drained |

**Thuật ngữ cần nhớ**

- vách ngăn cô lập tài nguyên → **a bulkhead**
- khoang kín → **a sealed compartment**
- cô lập → **isolation**
- phân bổ tài nguyên → **resource allocation**
- cầu dao nhảy → **the breaker trips**

---

## ⑤ Thử lại đúng cách và cơn bão thử lại

**Tiếng Việt**

Thử lại là công cụ dễ dùng sai nhất trong cả bộ, vì nó trông vô hại nhưng lại có thể làm sập hệ thống. Cách thử lại ngây thơ là thử ngay lập tức, thử với mọi loại lỗi, và mọi client cùng thử tại cùng một thời điểm. Khi một service phía sau đang ngộp, cách làm đó dội thêm một lượng lưu lượng gấp nhiều lần lên chính nó. Người ta gọi hiện tượng này là cơn bão thử lại, và nó biến một sự cố nhỏ thành một sự cố chết hẳn. Vì vậy chúng ta phải nói rằng thử lại chỉ an toàn khi nó đi kèm ba điều kiện.

Điều kiện thứ nhất là giãn cách theo cấp số nhân, nghĩa là mỗi lần thử sau đều chờ lâu hơn lần trước. Điều kiện thứ hai là thêm một chút ngẫu nhiên vào khoảng chờ, để hàng nghìn client không cùng gõ cửa tại đúng một mili giây. Điều kiện thứ ba là đặt hạn mức thử lại, nghĩa là chúng ta giới hạn tỷ lệ request được phép là request thử lại. Ngoài ba điều kiện đó, chúng ta chỉ thử lại các lỗi tạm thời và chỉ thử lại các thao tác bất biến khi lặp. Nếu chúng ta thử lại một lệnh tính tiền không có khoá bất biến, hậu quả là khách bị trừ tiền nhiều lần cho một đơn hàng.

**English (bám cấu trúc tiếng Việt)**

Retry is the tool that is most easily misused in the whole set, because it looks harmless but can bring the system down. The naive way of retrying is to retry immediately, to retry on every kind of failure, and to have every client retry at the same moment. When a downstream service is already overwhelmed, that approach pours several times more traffic onto it. People call this phenomenon a retry storm, and it turns a small incident into an incident that kills the service outright. Therefore we have to say that retrying is only safe when it comes with three conditions.

The first condition is exponential backoff, which means that each later attempt waits longer than the previous one. The second condition is adding a bit of randomness into the waiting time, so that thousands of clients do not knock on the door at exactly the same millisecond. The third condition is setting a retry budget, which means that we limit the proportion of requests that are allowed to be retries. Besides those three conditions, we only retry transient failures and we only retry operations that are idempotent. If we retry a charge command with no idempotency key, the consequence is that the customer is charged several times for one order.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| dễ dùng sai nhất trong cả bộ | most easily misused in the whole set |
| nó trông vô hại | it looks harmless |
| cách thử lại ngây thơ | the naive way of retrying |
| dội thêm một lượng lưu lượng gấp nhiều lần | pours several times more traffic |
| một sự cố chết hẳn | an incident that kills the service outright |
| chỉ an toàn khi nó đi kèm ba điều kiện | is only safe when it comes with three conditions |
| mỗi lần thử sau đều chờ lâu hơn lần trước | each later attempt waits longer than the previous one |
| không cùng gõ cửa tại đúng một mili giây | do not knock on the door at exactly the same millisecond |
| đặt hạn mức thử lại | setting a retry budget |
| giới hạn tỷ lệ request được phép là request thử lại | limit the proportion of requests that are allowed to be retries |
| khách bị trừ tiền nhiều lần cho một đơn hàng | the customer is charged several times for one order |

**Thuật ngữ cần nhớ**

- cơn bão thử lại → **a retry storm**
- giãn cách theo cấp số nhân → **exponential backoff**
- thêm ngẫu nhiên vào khoảng chờ → **jitter**
- hạn mức thử lại → **a retry budget**
- khoá chống trùng cho lệnh ghi → **an idempotency key**

---

## ⑥ Giá trị dự phòng và suy giảm êm ái

**Tiếng Việt**

Hỏng nhanh mới chỉ là một nửa câu chuyện, vì người dùng không quan tâm chúng ta hỏng nhanh hay hỏng chậm. Nửa còn lại là chúng ta trả về cái gì cho người dùng khi service phía sau đã chết. Chúng ta có bốn lựa chọn quen thuộc, và chúng ta nên chọn theo từng tính năng cụ thể. Lựa chọn thứ nhất là trả dữ liệu đã lưu tạm dù nó hơi cũ, và điều này thường tốt hơn nhiều so với một trang lỗi. Lựa chọn thứ hai là trả một giá trị mặc định, lựa chọn thứ ba là trả kết quả một phần, và lựa chọn thứ tư là nhận yêu cầu rồi xếp vào hàng đợi để xử lý sau.

Nguyên tắc bao trùm ở đây là chúng ta hạ cấp tính năng phụ để giữ tính năng lõi. Nếu dịch vụ gợi ý cá nhân hoá chết, chúng ta trả về danh sách sản phẩm phổ biến thay vì trả lỗi năm trăm cho cả trang. Nếu dịch vụ tính điểm thưởng chết, chúng ta vẫn cho khách hoàn tất đơn hàng và cộng điểm bù sau. Người dùng gần như không nhận ra sự khác biệt, trong khi doanh thu của chúng ta không dừng lại một giây nào. Nếu chúng ta không thiết kế phần này, hậu quả là một tính năng phụ hoàn toàn không quan trọng lại có quyền đánh sập cả trang thanh toán.

**English (bám cấu trúc tiếng Việt)**

Failing fast is only half of the story, because users do not care whether we fail fast or fail slowly. The other half is what we return to the user when the downstream service has died. We have four familiar choices, and we should choose according to each individual feature. The first choice is to return cached data even though it is slightly old, and this is usually far better than an error page. The second choice is to return a default value, the third choice is to return a partial result, and the fourth choice is to accept the request and then put it into a queue to be handled later.

The overarching principle here is that we degrade the secondary features in order to keep the core features. If the personalised recommendation service dies, we return a list of popular products instead of returning a five hundred error for the whole page. If the loyalty points service dies, we still let the customer complete the order and add the points afterwards. Users barely notice the difference, while our revenue does not stop for a single second. If we do not design this part, the consequence is that a completely unimportant secondary feature gets the right to bring down the whole checkout page.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| mới chỉ là một nửa câu chuyện | is only half of the story |
| dữ liệu đã lưu tạm dù nó hơi cũ | cached data even though it is slightly old |
| tốt hơn nhiều so với một trang lỗi | far better than an error page |
| trả kết quả một phần | return a partial result |
| xếp vào hàng đợi để xử lý sau | put it into a queue to be handled later |
| nguyên tắc bao trùm ở đây | the overarching principle here |
| chúng ta hạ cấp tính năng phụ | we degrade the secondary features |
| cộng điểm bù sau | add the points afterwards |
| gần như không nhận ra sự khác biệt | barely notice the difference |
| không dừng lại một giây nào | does not stop for a single second |
| lại có quyền đánh sập cả trang thanh toán | gets the right to bring down the whole checkout page |

**Thuật ngữ cần nhớ**

- giá trị dự phòng → **a fallback**
- suy giảm êm ái → **graceful degradation**
- dữ liệu cũ trong bộ nhớ đệm → **stale cached data**
- kết quả một phần → **a partial result**
- tính năng lõi và tính năng phụ → **core features and secondary features**

---

## ⑦ Kiểm soát ngược khi bên gửi nhanh hơn bên nhận

**Tiếng Việt**

Vấn đề tiếp theo xuất hiện khi bên gửi tạo việc nhanh hơn bên nhận xử lý được. Nếu không có gì chặn lại, vùng đệm ở giữa sẽ phình lên cho tới khi nó ngốn hết bộ nhớ và làm tiến trình chết. Trong hệ dùng broker, cùng hiện tượng đó xuất hiện dưới dạng độ trễ tiêu thụ tăng vô hạn. Vì vậy chúng ta cần kiểm soát ngược, tức là một cách để bên nhận nói cho bên gửi biết rằng nó đang quá tải. Đây chính là lý do các broker kiểu kéo an toàn hơn dưới tải nặng, như chúng ta đã học ở Bài 2.

Chúng ta có bốn công cụ thực dụng cho phần này. Công cụ thứ nhất là giới hạn số việc được xử lý đồng thời ở phía nhận, ví dụ đặt giới hạn prefetch hoặc giới hạn mức song song. Công cụ thứ hai là dùng hàng đợi có sức chứa hữu hạn, để hệ thống buộc phải ra quyết định khi hàng đầy. Công cụ thứ ba là chủ động loại bỏ bớt tải, ví dụ trả về mã lỗi quá nhiều yêu cầu cho một phần lưu lượng. Công cụ thứ tư là mở rộng số consumer, nhưng chúng ta nhớ rằng cách này bị chặn bởi số partition. Điều tệ nhất chúng ta có thể làm là để vùng đệm không giới hạn, vì khi đó hệ thống sẽ chết một cách bất ngờ thay vì chậm lại một cách có kiểm soát.

**English (bám cấu trúc tiếng Việt)**

The next problem appears when the sending side creates work faster than the receiving side can process. If nothing stops it, the buffer in the middle will swell up until it eats all the memory and kills the process. In a system using a broker, the same phenomenon appears in the form of consumer lag rising without limit. Therefore we need backpressure, that is, a way for the receiving side to tell the sending side that it is overloaded. This is exactly why pull-based brokers are safer under heavy load, as we learned in Lesson 2.

We have four practical tools for this part. The first tool is to limit the number of jobs processed concurrently on the receiving side, for example setting a prefetch limit or a concurrency limit. The second tool is to use a queue with a finite capacity, so that the system is forced to make a decision when the queue is full. The third tool is to shed load deliberately, for example returning a too-many-requests status code for part of the traffic. The fourth tool is to scale the number of consumers, but we remember that this way is capped by the partition count. The worst thing we can do is to leave the buffer unbounded, because then the system will die suddenly instead of slowing down in a controlled way.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| bên gửi tạo việc nhanh hơn bên nhận xử lý được | the sending side creates work faster than the receiving side can process |
| nếu không có gì chặn lại | if nothing stops it |
| ngốn hết bộ nhớ | eats all the memory |
| tăng vô hạn | rising without limit |
| nói cho bên gửi biết rằng nó đang quá tải | tell the sending side that it is overloaded |
| số việc được xử lý đồng thời | the number of jobs processed concurrently |
| hàng đợi có sức chứa hữu hạn | a queue with a finite capacity |
| buộc phải ra quyết định khi hàng đầy | is forced to make a decision when the queue is full |
| chủ động loại bỏ bớt tải | shed load deliberately |
| bị chặn bởi số partition | is capped by the partition count |
| để vùng đệm không giới hạn | to leave the buffer unbounded |
| chậm lại một cách có kiểm soát | slowing down in a controlled way |

**Thuật ngữ cần nhớ**

- kiểm soát ngược → **backpressure**
- giới hạn mức song song → **a concurrency limit**
- sức chứa hữu hạn → **a finite capacity**
- chủ động bỏ bớt tải → **load shedding**
- không giới hạn → **unbounded**

---

## ⑧ Dùng sai cầu dao, và vì sao hàng đợi không miễn cho chúng ta việc này

**Tiếng Việt**

Một người senior phải nói được cả những chỗ không nên dùng các công cụ này. Nếu chúng ta đặt ngưỡng quá chặt, cầu dao sẽ nhảy oan khi hệ thống chỉ vừa hắt hơi một cái. Nếu chúng ta đặt ngưỡng quá lỏng, lỗi vẫn lọt qua đủ lâu để kịp làm cạn bể luồng. Nếu chúng ta có cầu dao nhưng không có giá trị dự phòng, người dùng vẫn nhận lỗi và chúng ta chỉ đổi một lỗi chậm thành một lỗi nhanh. Ngoài ra, việc bọc cầu dao quanh một lời gọi hàm trong cùng tiến trình là hoàn toàn thừa, vì cầu dao chỉ dành cho các lời gọi qua mạng có độ trễ đáng kể.

Cuối cùng, khi một đồng nghiệp nói rằng hệ thống đã dùng hàng đợi nên không cần lo về khả năng chống chịu nữa, chúng ta nên phản biện bằng bốn điểm cụ thể. Điểm thứ nhất là consumer của chúng ta vẫn gọi đồng bộ xuống các service khác, nên chỗ đó vẫn cần thời gian chờ và cầu dao. Điểm thứ hai là message độc và hàng đợi thư chết vẫn phải được xử lý, vì hàng đợi không tự sửa dữ liệu hỏng. Điểm thứ ba là độ trễ tiêu thụ vẫn phải được giám sát, vì một consumer chậm cũng là một sự cố dù không ai thấy lỗi năm trăm. Điểm thứ tư là chính broker cũng có thể sập, và tính bất biến khi lặp vẫn bắt buộc như trước. Kết luận là hàng đợi gỡ được coupling thời gian, nhưng nó không gỡ được trách nhiệm thiết kế cho lỗi.

**English (bám cấu trúc tiếng Việt)**

A senior person must also be able to say where these tools should not be used. If we set the threshold too tight, the breaker will trip unfairly when the system has merely sneezed once. If we set the threshold too loose, failures still slip through long enough to drain the thread pool. If we have a breaker but no fallback value, users still receive an error and we have only changed a slow error into a fast error. Besides, wrapping a breaker around a function call inside the same process is completely pointless, because a breaker is only meant for network calls with meaningful latency.

Finally, when a colleague says that the system already uses a queue so we no longer need to worry about resilience, we should push back with four concrete points. The first point is that our consumers still call synchronously down to other services, so that place still needs timeouts and breakers. The second point is that poison messages and the dead-letter queue still have to be handled, because a queue does not fix broken data by itself. The third point is that consumer lag still has to be monitored, because a slow consumer is also an incident even though nobody sees a five hundred error. The fourth point is that the broker itself can go down, and idempotency is still mandatory as before. The conclusion is that a queue removes temporal coupling, but it does not remove our responsibility to design for failure.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| cả những chỗ không nên dùng | where these tools should not be used |
| cầu dao sẽ nhảy oan | the breaker will trip unfairly |
| hệ thống chỉ vừa hắt hơi một cái | the system has merely sneezed once |
| lỗi vẫn lọt qua đủ lâu | failures still slip through long enough |
| chỉ đổi một lỗi chậm thành một lỗi nhanh | only changed a slow error into a fast error |
| là hoàn toàn thừa | is completely pointless |
| chỉ dành cho các lời gọi qua mạng | only meant for network calls |
| phản biện bằng bốn điểm cụ thể | push back with four concrete points |
| hàng đợi không tự sửa dữ liệu hỏng | a queue does not fix broken data by itself |
| dù không ai thấy lỗi năm trăm | even though nobody sees a five hundred error |
| vẫn bắt buộc như trước | is still mandatory as before |
| trách nhiệm thiết kế cho lỗi | our responsibility to design for failure |

**Thuật ngữ cần nhớ**

- nhảy oan, kích hoạt sai → **to trip unfairly**
- ngưỡng quá chặt hoặc quá lỏng → **too tight or too loose a threshold**
- vô nghĩa, thừa → **pointless**
- gọi trong cùng tiến trình → **an in-process call**
- thiết kế cho tình huống lỗi → **to design for failure**

---

## ⑨ Mô hình ghi nhớ

**Tiếng Việt**

Thứ tự công cụ luôn là thời gian chờ, rồi cầu dao, rồi vách ngăn, rồi thử lại có giãn cách và ngẫu nhiên, và cuối cùng là giá trị dự phòng. Một service chậm có thể kéo sập cả hệ thống, nên mục tiêu của chúng ta là hỏng thật nhanh và hỏng thật êm; hàng đợi không cứu chúng ta khỏi việc đó.

**English (bám cấu trúc tiếng Việt)**

The order of the tools is always the timeout, then the circuit breaker, then the bulkhead, then the retry with backoff and jitter, and finally the fallback. One slow service can bring the whole system down, so our goal is to fail fast and to fail softly; a queue does not save us from that.

---

## ⑩ Bảng thuật ngữ tổng hợp

| Tiếng Việt | Tiếng Anh | Ghi chú phát âm |
|---|---|---|
| khả năng chống chịu | resilience | ri-**ZI**-li-əns, chữ *s* đọc /z/, trọng âm âm thứ hai |
| lỗi lan theo chuỗi | cascade failure | *cascade* = cas-**CADE**, trọng âm âm sau |
| cạn kiệt tài nguyên | resource exhaustion | *exhaustion* = ig-**ZOR**-schn, chữ *h* gần như câm |
| bể luồng | the thread pool | *thread* — âm *th* /θ/ + đuôi /d/: "thred" |
| bể kết nối | the connection pool | |
| service phía sau | the downstream service | |
| cầu dao ngắt mạch | a circuit breaker | *circuit* = **SER**-kit, chữ *c* thứ hai đọc /k/ |
| đóng / mở / nửa mở | closed / open / half-open | *half* /hɑːf/ — chữ **l** câm |
| tỷ lệ lỗi | the failure rate | |
| khoảng nguội | the cooldown period | |
| hỏng nhanh | to fail fast | |
| hỏng êm | to fail soft | |
| thời gian chờ tối đa | a timeout | |
| lời gọi ra ngoài | an outbound call | |
| phân vị độ trễ | a latency percentile | *percentile* = pə-**SEN**-tail |
| điều kiện tiên quyết | a precondition | |
| vách ngăn cô lập tài nguyên | a bulkhead | **BULK**-head, trọng âm âm đầu |
| khoang kín | a sealed compartment | *compartment* = kəm-**PART**-mənt |
| cô lập | isolation | ai-sə-**LAY**-shn, âm đầu là /aɪ/ |
| cầu dao nhảy | the breaker trips | |
| cơn bão thử lại | a retry storm | *retry* = ri-**TRAI**, trọng âm âm sau |
| giãn cách theo cấp số nhân | exponential backoff | ek-spo-**NEN**-shl |
| thêm ngẫu nhiên vào khoảng chờ | jitter | **JI**-tə, chữ *j* đọc /dʒ/ |
| hạn mức thử lại | a retry budget | *budget* = **BU**-jit |
| lỗi tạm thời | a transient failure | **TRAN**-zi-ənt, chữ *s* đọc /z/ |
| khoá chống trùng | an idempotency key | ai-dem-**PO**-tən-si |
| giá trị dự phòng | a fallback | |
| suy giảm êm ái | graceful degradation | de-grə-**DAY**-shn |
| dữ liệu cũ trong bộ đệm | stale cached data | *cached* = "cashd"; *stale* /steɪl/ |
| kết quả một phần | a partial result | *partial* = **PAR**-shl, chữ *ti* đọc /ʃ/ |
| tính năng lõi / tính năng phụ | core features / secondary features | *secondary* = **SE**-cən-də-ri |
| kiểm soát ngược | backpressure | **BACK**-pressure |
| giới hạn mức song song | a concurrency limit | kən-**KUR**-ən-si |
| sức chứa hữu hạn | a finite capacity | *finite* = **FAI**-nait, âm đầu là /aɪ/ |
| chủ động bỏ bớt tải | load shedding | *shedding* — bật rõ /ʃ/ ở đầu |
| không giới hạn | unbounded | |
| nhảy oan | to trip unfairly | |
| vô nghĩa, thừa | pointless | |
| gọi trong cùng tiến trình | an in-process call | |
| thiết kế cho tình huống lỗi | to design for failure | |
| ngưỡng | threshold | âm *th* /θ/, đọc "THRESH-hold" |

---

## ⑪ Bài nói

> **Cách luyện:** bật ghi âm, nói **60–90 giây** cho mỗi đề, **không nhìn tài liệu**. Mỗi bài nói phải dùng ít nhất **sáu thuật ngữ** trong bảng trên. Nói xong thì nghe lại một lượt, đánh dấu chỗ mình ngập ngừng, rồi nói lại chính đề đó thêm một lần nữa.

1. Explain to a junior developer how one slow downstream service can take down an entire system, walking through what happens to the thread pool at each hop.

2. Describe the three states of a circuit breaker, and explain what the open state protects on the caller side and on the downstream side.

3. A colleague has added a circuit breaker but no timeouts, and says the breaker will catch the problem. Explain exactly why the breaker will never trip.

4. Someone on your team wants to retry every failed request three times immediately, to improve reliability. Explain why you would push back and what you would propose instead.

5. Describe the difference between a bulkhead and a circuit breaker, and explain why a serious system uses both.

6. The recommendation service is down. Describe how you would keep the product page working, and describe what the user would see.

7. A colleague argues that because everything now goes through Kafka, the team can stop worrying about resilience patterns. Explain the case against that.
