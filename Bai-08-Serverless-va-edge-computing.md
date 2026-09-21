# Bài 8 — Serverless & edge computing
### Bản song ngữ: tiếng Anh bám cấu trúc tiếng Việt

> **Cách dùng:** đọc đoạn tiếng Việt trước → tự dịch trong đầu → so với bản tiếng Anh bên dưới → **đọc to bản tiếng Anh hai lần**. Lần một đọc chậm cho đúng từ, lần hai đọc liền mạch như đang nói với người phỏng vấn. Bài này có nhiều tên sản phẩm, nhưng điều được chấm là **tiêu chí lựa chọn**, nên hãy luyện kỹ các cụm so sánh và các cụm nói về chi phí.

---

## Phần 1 — Trực giác: trả tiền theo cuốc

**Tiếng Việt**

Chúng ta có thể hình dung serverless như việc đi taxi theo cuốc thay vì tự nuôi một chiếc xe. Khi không có cuốc nào, chúng ta không trả đồng nào, và đó chính là ý nghĩa của việc thu nhỏ về không. Khi có cuốc đầu tiên sau một thời gian dài không đi, chúng ta phải chờ xe chạy tới, và đó chính là khởi động nguội. Nếu chúng ta phải chở hàng đường dài và liên tục, thì tự nuôi một chiếc xe rẻ hơn nhiều so với gọi taxi cả ngày. Điện toán biên thì giống việc đặt một quầy phục vụ nhỏ ngay cạnh khách hàng, và quầy đó chỉ làm được những việc nhẹ.

Phép ẩn dụ này hữu ích vì nó gói gọn cả ba đánh đổi chính chỉ trong một hình ảnh. Thứ nhất, chúng ta đổi chi phí cố định lấy chi phí theo lượt dùng. Thứ hai, chúng ta đổi khả năng kiểm soát môi trường chạy lấy việc ít phải vận hành. Thứ ba, chúng ta đổi độ trễ ổn định lấy sự linh hoạt về quy mô. Trong phỏng vấn, chúng ta nên mở đầu bằng ba vế đổi này rồi mới đi vào chi tiết kỹ thuật, vì nó cho thấy chúng ta nhìn serverless như một lựa chọn kinh tế chứ không phải như một xu hướng.

**English (bám cấu trúc tiếng Việt)**

We can picture serverless as taking a taxi per ride instead of owning a car ourselves. When there is no ride, we pay nothing, and that is exactly the meaning of scaling to zero. When the first ride comes after a long period without travelling, we have to wait for the car to arrive, and that is exactly a cold start. If we have to carry goods over long distances continuously, then owning a car ourselves is much cheaper than calling taxis all day. Edge computing is like putting a small service counter right next to the customer, and that counter can only do light work.

This metaphor is useful because it packs all three main trade-offs into one picture. First, we trade a fixed cost for a pay-per-use cost. Second, we trade control over the runtime environment for having little to operate. Third, we trade stable latency for flexibility in scale. In an interview, we should open with these three trades and only then go into the technical details, because it shows that we look at serverless as an economic choice rather than as a trend.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| đi taxi theo cuốc | taking a taxi per ride |
| thay vì tự nuôi một chiếc xe | instead of owning a car ourselves |
| việc thu nhỏ về không | scaling to zero |
| chúng ta phải chờ xe chạy tới | we have to wait for the car to arrive |
| chở hàng đường dài và liên tục | carry goods over long distances continuously |
| một quầy phục vụ nhỏ ngay cạnh khách hàng | a small service counter right next to the customer |
| gói gọn cả ba đánh đổi chính | packs all three main trade-offs |
| chúng ta đổi chi phí cố định lấy chi phí theo lượt dùng | we trade a fixed cost for a pay-per-use cost |
| khả năng kiểm soát môi trường chạy | control over the runtime environment |
| như một lựa chọn kinh tế chứ không phải như một xu hướng | as an economic choice rather than as a trend |

**Thuật ngữ cần nhớ**

- hàm như dịch vụ → **function as a service (FaaS)**
- thu nhỏ về không → **scale to zero**
- khởi động nguội → **a cold start**
- trả theo lượt dùng → **pay per use**
- môi trường chạy → **the runtime**

---

## Phần 2 — Khi nào serverless hợp và khi nào không hợp

**Tiếng Việt**

Serverless hợp nhất với ba dạng tải, và chúng ta nên gọi tên cả ba. Dạng thứ nhất là tải gai, tức là lưu lượng lúc rất cao lúc gần bằng không, ví dụ một điểm nhận webhook. Dạng thứ hai là tải hướng sự kiện, ví dụ mỗi lần có tệp mới tải lên thì chúng ta tạo ảnh thu nhỏ. Dạng thứ ba là những việc chạy theo lịch, ví dụ một tác vụ tổng hợp dữ liệu chạy lúc hai giờ sáng. Điểm chung của ba dạng này là chúng ta không cần một máy chủ nằm chờ suốt ngày, nên việc thu nhỏ về không tiết kiệm thật.

Serverless không hợp với bốn tình huống, và chúng ta cũng nên nói thẳng ra. Thứ nhất là tải đều và cao liên tục, vì lúc đó chi phí theo lượt gọi vượt xa chi phí của một cụm máy chủ thường trú. Thứ hai là những hệ rất nhạy với độ trễ đuôi, vì khởi động nguội làm hỏng ngân sách độ trễ ở phân vị cao. Thứ ba là những tác vụ chạy dài, vì các nền tảng đều đặt giới hạn thời gian cho một lần gọi. Thứ tư là những dịch vụ cần giữ trạng thái trong bộ nhớ, ví dụ một máy chủ trò chơi giữ kết nối lâu dài. Nếu chúng ta ép serverless vào bốn tình huống này, hậu quả là chúng ta vừa trả nhiều tiền hơn vừa nhận độ trễ tệ hơn.

**English (bám cấu trúc tiếng Việt)**

Serverless fits three shapes of load best, and we should name all three. The first shape is spiky load, that is, traffic that is very high at times and near zero at other times, for example a webhook endpoint. The second shape is event-driven load, for example every time a new file is uploaded we generate a thumbnail. The third shape is work that runs on a schedule, for example a data aggregation job that runs at two in the morning. What these three shapes have in common is that we do not need a server sitting and waiting all day, so scaling to zero saves real money.

Serverless does not fit four situations, and we should say those out loud as well. The first is a steady and continuously high load, because at that point the per-invocation cost far exceeds the cost of a cluster of long-running servers. The second is systems that are very sensitive to tail latency, because cold starts ruin the latency budget at the high percentiles. The third is long-running tasks, because every platform puts a time limit on a single invocation. The fourth is services that need to keep state in memory, for example a game server holding long-lived connections. If we force serverless into these four situations, the consequence is that we both pay more money and get worse latency.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| ba dạng tải | three shapes of load |
| tải gai | spiky load |
| lúc rất cao lúc gần bằng không | very high at times and near zero at other times |
| tải hướng sự kiện | event-driven load |
| những việc chạy theo lịch | work that runs on a schedule |
| một máy chủ nằm chờ suốt ngày | a server sitting and waiting all day |
| chi phí theo lượt gọi vượt xa | the per-invocation cost far exceeds |
| rất nhạy với độ trễ đuôi | very sensitive to tail latency |
| đặt giới hạn thời gian cho một lần gọi | put a time limit on a single invocation |
| giữ kết nối lâu dài | holding long-lived connections |
| chúng ta vừa trả nhiều tiền hơn vừa nhận độ trễ tệ hơn | we both pay more money and get worse latency |

**Thuật ngữ cần nhớ**

- tải gai → **spiky load**
- hướng sự kiện → **event-driven**
- lượt gọi hàm → **an invocation**
- ảnh thu nhỏ → **a thumbnail**
- dịch vụ thường trú → **a long-running service**

---

## Phần 3 — Khởi động nguội và cách giảm

**Tiếng Việt**

Khởi động nguội xảy ra ở lần gọi đầu tiên hoặc sau một khoảng dài không có lưu lượng, khi nền tảng phải dựng lại môi trường chạy từ đầu. Nền tảng phải tải mã của chúng ta, khởi tạo môi trường ngôn ngữ, rồi chạy phần khởi tạo của ứng dụng trước khi xử lý request. Tất cả những việc đó cộng thêm vào độ trễ của đúng người dùng xui xẻo đến đầu tiên. Với tải bất đồng bộ, ví dụ một sự kiện tải tệp hoặc một tác vụ theo lịch, chúng ta thường chịu được khoảng chờ này. Với một giao diện người dùng nhạy về độ trễ, khoảng chờ đó lại xuất hiện đúng ở phân vị 99 và làm hỏng chỉ tiêu.

Chúng ta có vài cách giảm, và chúng ta nên nói kèm chi phí của từng cách. Cách thứ nhất là mua sẵn một mức đồng thời được cấp phát trước, nghĩa là nền tảng giữ sẵn một số bản đã khởi tạo, nhưng chúng ta trả tiền cố định suốt hai mươi bốn giờ. Cách thứ hai là dùng cơ chế chụp ảnh môi trường đã khởi tạo, ví dụ SnapStart trên các môi trường chạy được hỗ trợ, và cách này thường miễn phí nên đáng thử trước. Cách thứ ba là giữ ấm bằng cách gọi định kỳ, tuy nhiên cách này chỉ giúp cho mức đồng thời thấp. Cách thứ tư là chọn môi trường chạy nhẹ và giảm số thư viện nạp lúc khởi tạo, vì phần lớn thời gian khởi động nguội nằm ở đó. Nếu chúng ta mua đồng thời cấp phát trước cho mọi hàm, hậu quả là chúng ta mất chính lợi ích kinh tế mà serverless hứa hẹn.

**English (bám cấu trúc tiếng Việt)**

A cold start happens on the first invocation or after a long period with no traffic, when the platform has to build the runtime environment again from scratch. The platform has to load our code, initialise the language environment, and then run the application's initialisation part before it handles the request. All of that work adds to the latency of exactly the unlucky user who arrives first. For asynchronous load, for example a file upload event or a scheduled job, we can usually tolerate this wait. For a latency-sensitive user interface, that wait instead shows up right at the 99th percentile and ruins the target.

We have several ways to reduce it, and we should state the cost of each way alongside. The first way is to buy a pre-allocated level of concurrency, which means the platform keeps a number of already-initialised copies ready, but we pay a fixed price around the clock. The second way is to use a snapshot of the already-initialised environment, for example SnapStart on the supported runtimes, and this way is usually free so it is worth trying first. The third way is to keep things warm by calling on a schedule, however this way only helps at low concurrency. The fourth way is to choose a light runtime and reduce the number of libraries loaded at initialisation, because most of the cold-start time sits there. If we buy pre-allocated concurrency for every function, the consequence is that we lose the very economic benefit that serverless promised.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| dựng lại môi trường chạy từ đầu | build the runtime environment again from scratch |
| chạy phần khởi tạo của ứng dụng | run the application's initialisation part |
| đúng người dùng xui xẻo đến đầu tiên | exactly the unlucky user who arrives first |
| chúng ta thường chịu được khoảng chờ này | we can usually tolerate this wait |
| xuất hiện đúng ở phân vị 99 | shows up right at the 99th percentile |
| một mức đồng thời được cấp phát trước | a pre-allocated level of concurrency |
| chúng ta trả tiền cố định suốt hai mươi bốn giờ | we pay a fixed price around the clock |
| chụp ảnh môi trường đã khởi tạo | a snapshot of the already-initialised environment |
| giữ ấm bằng cách gọi định kỳ | keep things warm by calling on a schedule |
| số thư viện nạp lúc khởi tạo | the number of libraries loaded at initialisation |
| chính lợi ích kinh tế mà serverless hứa hẹn | the very economic benefit that serverless promised |

**Thuật ngữ cần nhớ**

- khởi động nguội → **a cold start**
- đồng thời cấp phát trước → **provisioned concurrency**
- giữ ấm → **keep-warm**
- ảnh chụp nhanh → **a snapshot**
- phân vị 99 → **the 99th percentile**

---

## Phần 4 — Không giữ trạng thái, phù du, và bài toán bùng nổ kết nối

**Tiếng Việt**

Mỗi lượt gọi hàm chạy trong một môi trường không giữ trạng thái và có vòng đời rất ngắn. Hệ quả trực tiếp là mỗi lượt gọi có thể tự mở một kết nối tới cơ sở dữ liệu, và khi lưu lượng bùng lên thì hàng nghìn lượt gọi mở kết nối cùng lúc. Cơ sở dữ liệu có giới hạn số kết nối tối đa, nên nó bắt đầu từ chối kết nối hoặc treo dưới sức nặng đó. Đây là lý do một hàm chạy hoàn hảo trong lúc kiểm thử lại đánh sập cơ sở dữ liệu khi lưu lượng thật tăng vọt. Điều nguy hiểm là lỗi này không lộ ra ở quy mô nhỏ, nên chúng ta chỉ gặp nó vào đúng ngày đông khách nhất.

Cách chữa là chúng ta đặt một lớp trung gian quản lý pool kết nối dùng chung, ví dụ một proxy cơ sở dữ liệu do nhà cung cấp quản lý. Hàm của chúng ta nối tới proxy đó, còn proxy giữ một pool ổn định phía sau và chia lại cho các lượt gọi. Bên trong ứng dụng, chúng ta không được tự mở một pool lớn nữa, vì như thế là gộp hai lớp pool chồng lên nhau và làm mọi thứ tệ hơn. Chúng ta cũng nên đặt một giới hạn đồng thời dành riêng cho hàm, để một cơn tăng đột biến không thể vượt quá khả năng chịu đựng của cơ sở dữ liệu. Nếu chúng ta bỏ qua lớp proxy này, hậu quả là chính lớp tính toán co giãn vô hạn của chúng ta trở thành vũ khí phá huỷ tầng dữ liệu.

**English (bám cấu trúc tiếng Việt)**

Each function invocation runs in an environment that is stateless and has a very short lifetime. The direct consequence is that each invocation may open its own connection to the database, and when the traffic bursts then thousands of invocations open connections at the same time. The database has a limit on the maximum number of connections, so it starts refusing connections or hanging under that weight. This is the reason why a function that runs perfectly during testing then knocks the database down when the real traffic spikes. The dangerous thing is that this bug does not show up at small scale, so we only meet it on exactly the busiest day.

The fix is that we put in an intermediate layer that manages a shared connection pool, for example a managed database proxy from the provider. Our function connects to that proxy, while the proxy keeps a stable pool behind it and hands connections back out to the invocations. Inside the application, we must not open a large pool of our own any more, because that stacks two pool layers on top of each other and makes everything worse. We should also set a reserved concurrency limit for the function, so that one sudden surge cannot go beyond what the database can bear. If we skip this proxy layer, the consequence is that our infinitely elastic compute layer itself becomes the weapon that destroys the data tier.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| có vòng đời rất ngắn | has a very short lifetime |
| khi lưu lượng bùng lên | when the traffic bursts |
| nó bắt đầu từ chối kết nối hoặc treo | it starts refusing connections or hanging |
| dưới sức nặng đó | under that weight |
| khi lưu lượng thật tăng vọt | when the real traffic spikes |
| không lộ ra ở quy mô nhỏ | does not show up at small scale |
| chia lại cho các lượt gọi | hands connections back out to the invocations |
| gộp hai lớp pool chồng lên nhau | stacks two pool layers on top of each other |
| một giới hạn đồng thời dành riêng | a reserved concurrency limit |
| vượt quá khả năng chịu đựng của cơ sở dữ liệu | go beyond what the database can bear |
| trở thành vũ khí phá huỷ tầng dữ liệu | becomes the weapon that destroys the data tier |

**Thuật ngữ cần nhớ**

- phù du, vòng đời ngắn → **ephemeral**
- bùng nổ kết nối → **connection blow-up**
- proxy cơ sở dữ liệu → **a database proxy**
- pool dùng chung → **a shared pool**
- giới hạn đồng thời dành riêng → **reserved concurrency**

---

## Phần 5 — Điện toán biên: đặt gì và không đặt gì ở đó

**Tiếng Việt**

Điện toán biên nghĩa là chúng ta chạy một ít logic ngay tại các điểm biên của mạng phân phối nội dung, tức là rất gần người dùng. Nhờ đó, chúng ta cắt được một vòng đi về tới vùng gốc, và với người dùng ở xa thì khoản tiết kiệm đó lên tới cả trăm mili giây. Những việc hợp với biên đều là việc nhẹ và không cần nhiều dữ liệu, ví dụ kiểm tra một token xác thực, viết lại đường dẫn, chia nhánh cho thử nghiệm A và B, hoặc chọn phiên bản nội dung theo vùng địa lý. Điểm chung là chúng chỉ cần dữ liệu đầu vào của request và một ít cấu hình. Vì vậy chúng chạy xong trong vài mili giây và không cần hỏi cơ sở dữ liệu.

Chúng ta không đặt ở biên những thứ nặng, những thứ giữ trạng thái, hoặc những thứ cần nhiều dữ liệu từ vùng gốc. Lý do là môi trường biên bị giới hạn về CPU và về thời gian chạy, nên một tác vụ nặng sẽ bị cắt giữa chừng. Lý do thứ hai là biên nằm xa cơ sở dữ liệu nguồn, nên mỗi lần hỏi dữ liệu lại phải đi ngược về vùng gốc và chúng ta mất luôn lợi thế khoảng cách. Nói cách khác, chúng ta đặt logic nhẹ cùng với cache ở biên, và giữ dữ liệu cùng logic nặng ở vùng gốc. Nếu chúng ta đẩy nghiệp vụ nặng ra biên, hậu quả là chúng ta có một hệ thống khó gỡ lỗi, chạy ở hàng trăm nơi, mà vẫn chậm vì nó liên tục gọi ngược về trung tâm.

**English (bám cấu trúc tiếng Việt)**

Edge computing means that we run a little logic right at the edge locations of the content delivery network, that is, very close to the users. Thanks to that, we cut out one round trip to the origin region, and for far-away users that saving reaches a hundred milliseconds or more. The work that suits the edge is all light work that does not need much data, for example checking an authentication token, rewriting a path, branching for an A and B test, or picking a content variant by geographic region. What they have in common is that they only need the request's input data and a little configuration. Therefore they finish within a few milliseconds and do not need to ask the database.

We do not put at the edge the things that are heavy, the things that hold state, or the things that need a lot of data from the origin region. The reason is that the edge environment is limited in CPU and in execution time, so a heavy task will be cut off halfway. The second reason is that the edge sits far from the source database, so every data lookup has to travel back to the origin region and we lose the distance advantage entirely. In other words, we put light logic together with a cache at the edge, and we keep the data together with the heavy logic at the origin. If we push heavy business logic out to the edge, the consequence is that we have a system which is hard to debug, runs in hundreds of places, and is still slow because it keeps calling back to the centre.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| chúng ta cắt được một vòng đi về | we cut out one round trip |
| khoản tiết kiệm đó lên tới cả trăm mili giây | that saving reaches a hundred milliseconds or more |
| việc nhẹ và không cần nhiều dữ liệu | light work that does not need much data |
| viết lại đường dẫn | rewriting a path |
| chia nhánh cho thử nghiệm A và B | branching for an A and B test |
| chọn phiên bản nội dung theo vùng địa lý | picking a content variant by geographic region |
| sẽ bị cắt giữa chừng | will be cut off halfway |
| phải đi ngược về vùng gốc | has to travel back to the origin region |
| chúng ta mất luôn lợi thế khoảng cách | we lose the distance advantage entirely |
| nó liên tục gọi ngược về trung tâm | it keeps calling back to the centre |

**Thuật ngữ cần nhớ**

- điện toán biên → **edge computing**
- vùng gốc → **the origin region**
- xác thực → **authentication**
- viết lại đường dẫn → **URL rewriting**
- cá nhân hoá → **personalisation**

---

## Phần 6 — Chi phí và sự lệ thuộc nhà cung cấp

**Tiếng Việt**

Câu nói rằng serverless luôn rẻ hơn là một câu sai, và chúng ta nên biết cách bác bỏ nó bằng số. Serverless rẻ khi tải gai, vì chúng ta chỉ trả tiền cho thời gian thật sự chạy. Nhưng khi tải trở nên đều và cao, tổng số lượt gọi nhân với giá mỗi lượt vượt qua chi phí thuê một cụm máy chủ cố định. Điểm mà hai đường chi phí cắt nhau gọi là điểm giao, và việc của chúng ta là ước lượng xem hệ thống của mình nằm ở phía nào của điểm đó. Cách trả lời mạnh trong phỏng vấn là chúng ta nói rằng chúng ta sẽ tính chi phí ở mức tải dự kiến trước khi chọn, chứ không chọn theo cảm giác.

Chi phí thứ hai không nằm trên hoá đơn, đó là sự lệ thuộc vào nhà cung cấp. Khi chúng ta viết hàm gắn chặt với các dịch vụ riêng của một nhà cung cấp, việc chuyển sang nhà cung cấp khác sau này rất tốn công. Chúng ta giảm rủi ro đó bằng cách giữ phần logic nghiệp vụ thuần khiết, tách khỏi lớp vỏ của nền tảng. Nói cách khác, hàm xử lý sự kiện chỉ nên là một lớp mỏng gọi vào phần nghiệp vụ có thể kiểm thử độc lập. Nếu chúng ta trộn lẫn hai thứ đó, hậu quả là chúng ta không những khó chuyển nhà cung cấp, mà còn không kiểm thử được nghiệp vụ nếu không dựng cả nền tảng lên.

**English (bám cấu trúc tiếng Việt)**

The sentence that serverless is always cheaper is a wrong sentence, and we should know how to refute it with numbers. Serverless is cheap for spiky load, because we only pay for the time that actually runs. But when the load becomes steady and high, the total number of invocations multiplied by the price per invocation passes the cost of renting a fixed cluster of servers. The point where the two cost lines cross is called the crossover point, and our job is to estimate which side of that point our system sits on. The strong way to answer in an interview is that we say we will compute the cost at the expected load before choosing, rather than choosing by feel.

The second cost does not appear on the bill, and that is vendor lock-in. When we write functions tied tightly to one provider's own services, moving to another provider later is very laborious. We reduce that risk by keeping the business logic pure, separated from the platform's shell. In other words, the event handler function should only be a thin layer that calls into the business part which can be tested independently. If we mix those two things together, the consequence is that we are not only stuck with the provider, but we also cannot test the business logic without standing up the whole platform.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| bác bỏ nó bằng số | refute it with numbers |
| chỉ trả tiền cho thời gian thật sự chạy | only pay for the time that actually runs |
| tổng số lượt gọi nhân với giá mỗi lượt | the total number of invocations multiplied by the price per invocation |
| điểm mà hai đường chi phí cắt nhau | the point where the two cost lines cross |
| nằm ở phía nào của điểm đó | which side of that point it sits on |
| ở mức tải dự kiến | at the expected load |
| không nằm trên hoá đơn | does not appear on the bill |
| gắn chặt với các dịch vụ riêng của một nhà cung cấp | tied tightly to one provider's own services |
| tách khỏi lớp vỏ của nền tảng | separated from the platform's shell |
| một lớp mỏng gọi vào phần nghiệp vụ | a thin layer that calls into the business part |

**Thuật ngữ cần nhớ**

- điểm giao chi phí → **the cost crossover point**
- lệ thuộc nhà cung cấp → **vendor lock-in**
- tải dự kiến → **the expected load**
- hàm xử lý → **a handler**
- kiểm thử độc lập → **to test in isolation**

---

## Phần 7 — Khung quyết định giữa serverless và container

**Tiếng Việt**

Khi ai đó hỏi chúng ta nên dùng serverless hay dùng container, chúng ta trả lời bằng sáu tiêu chí thay vì bằng một sở thích. Tiêu chí thứ nhất là hình dạng tải, tức là tải gai hay tải đều. Tiêu chí thứ hai là ngân sách độ trễ, đặc biệt ở phân vị cao nơi khởi động nguội lộ ra. Tiêu chí thứ ba là mức độ phức tạp vận hành mà đội chúng ta gánh nổi. Tiêu chí thứ tư là chi phí tại mức tải dự kiến, tiêu chí thứ năm là nhu cầu giữ trạng thái, và tiêu chí thứ sáu là mức lệ thuộc nhà cung cấp mà công ty chấp nhận được.

Cách trình bày ghi điểm là chúng ta chia hệ thống ra rồi chọn khác nhau cho từng phần, thay vì chọn một thứ cho tất cả. Ví dụ, chúng ta đặt điểm nhận webhook và các tác vụ theo lịch trên serverless, vì chúng gai và bất đồng bộ. Cùng lúc đó, chúng ta đặt API chính phục vụ người dùng trên một cụm container, vì nó có tải đều và nhạy về độ trễ. Cách chia này cho thấy chúng ta quyết theo yêu cầu chứ không theo một phe công nghệ. Nếu chúng ta khăng khăng dùng một mô hình duy nhất cho mọi thứ, hậu quả là chúng ta ép một nửa hệ thống chạy trong điều kiện mà nó không được thiết kế để chạy.

**English (bám cấu trúc tiếng Việt)**

When someone asks us whether to use serverless or containers, we answer with six criteria instead of with a preference. The first criterion is the shape of the load, that is, whether it is spiky or steady. The second criterion is the latency budget, especially at the high percentiles where cold starts show up. The third criterion is the level of operational complexity that our team can carry. The fourth criterion is the cost at the expected load, the fifth criterion is the need to hold state, and the sixth criterion is the level of vendor lock-in that the company finds acceptable.

The way of presenting that scores is that we split the system up and then choose differently for each part, rather than choosing one thing for everything. For example, we put the webhook endpoint and the scheduled jobs on serverless, because they are spiky and asynchronous. At the same time, we put the main user-facing API on a container cluster, because it has a steady load and is latency-sensitive. This way of splitting shows that we decide by requirements rather than by a technology camp. If we insist on one single model for everything, the consequence is that we force half of the system to run in conditions it was not designed to run in.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| bằng sáu tiêu chí thay vì bằng một sở thích | with six criteria instead of with a preference |
| hình dạng tải | the shape of the load |
| nơi khởi động nguội lộ ra | where cold starts show up |
| mà đội chúng ta gánh nổi | that our team can carry |
| chọn khác nhau cho từng phần | choose differently for each part |
| thay vì chọn một thứ cho tất cả | rather than choosing one thing for everything |
| theo một phe công nghệ | by a technology camp |
| nếu chúng ta khăng khăng dùng một mô hình duy nhất | if we insist on one single model |
| chúng ta ép một nửa hệ thống | we force half of the system |
| điều kiện mà nó không được thiết kế để chạy | conditions it was not designed to run in |

**Thuật ngữ cần nhớ**

- cụm container → **a container cluster**
- ngân sách độ trễ → **the latency budget**
- phức tạp vận hành → **operational complexity**
- nhạy về độ trễ → **latency-sensitive**
- điểm cuối → **an endpoint**

---

## Mô hình ghi nhớ

**Tiếng Việt**

Serverless là trả tiền theo cuốc, nên nó hợp với tải gai. Vì môi trường không giữ trạng thái, kết nối trở thành bài toán và chúng ta cần một proxy, còn ở biên chúng ta chỉ đặt logic nhẹ, và chúng ta quyết theo yêu cầu chứ không theo xu hướng.

**English (bám cấu trúc tiếng Việt)**

Serverless is paying per ride, so it fits spiky load. Because the environment is stateless, connections become a problem and we need a proxy, while at the edge we only put light logic, and we decide by requirements rather than by trends.

---

## Bảng thuật ngữ tổng hợp

| Tiếng Việt | Tiếng Anh | Ghi chú phát âm |
|---|---|---|
| hàm như dịch vụ | function as a service (FaaS) | |
| thu nhỏ về không | scale to zero | |
| khởi động nguội | a cold start | âm cuối **-ld** và **-rt** đều phải bật ra |
| trả theo lượt dùng | pay per use | |
| môi trường chạy | the runtime | |
| tải gai | spiky load | *spiky* — "**SPAI**-ki" |
| hướng sự kiện | event-driven | |
| lượt gọi hàm | an invocation | in-vơ-**KÂY**-shợn, trọng âm âm thứ ba |
| ảnh thu nhỏ | a thumbnail | *thumb* — chữ **b câm**, đọc "thăm-nêil"; âm **th** /θ/ |
| dịch vụ thường trú | a long-running service | |
| lịch chạy | a schedule | Anh /ˈʃedjuːl/ — "**SHE**-điu-l"; Mỹ đọc "SKE-jul". Với khách Anh dùng "SHE-" |
| độ trễ đuôi | tail latency | *latency* — "LÂY-tân-si" |
| phân vị 99 | the 99th percentile | đọc "the ninety-ninth per-**CEN**-tile" |
| đồng thời cấp phát trước | provisioned concurrency | *provisioned* — prơ-**VI**-zhợnd, âm giữa là /ʒ/ |
| giữ ấm | keep-warm | |
| ảnh chụp nhanh | a snapshot | |
| phù du, vòng đời ngắn | ephemeral | i-**FE**-mơ-rợl, trọng âm âm thứ hai |
| bùng nổ kết nối | connection blow-up | |
| proxy cơ sở dữ liệu | a database proxy | |
| pool dùng chung | a shared pool | |
| giới hạn đồng thời dành riêng | reserved concurrency | *concurrency* — cần-**CA**-rợn-si, trọng âm âm thứ hai |
| tầng dữ liệu | the data tier | *tier* /tɪə/ — "ti-ơ", đọc gần giống *tear* (nước mắt) |
| điện toán biên | edge computing | |
| vùng gốc | the origin region | *origin* — "**O**-ri-jin", trọng âm đầu |
| xác thực | authentication | ô-then-ti-**KÂY**-shợn, trọng âm âm thứ tư; có âm **th** |
| viết lại đường dẫn | URL rewriting | |
| cá nhân hoá | personalisation | pơ-sơ-nơ-lai-**ZÂY**-shợn, trọng âm áp chót |
| vòng đi về | a round trip | |
| điểm giao chi phí | the cost crossover point | |
| lệ thuộc nhà cung cấp | vendor lock-in | *vendor* — "**VEN**-đơ", trọng âm đầu |
| tải dự kiến | the expected load | |
| hàm xử lý | a handler | |
| kiểm thử độc lập | to test in isolation | *isolation* — ai-sơ-**LÂY**-shợn, âm đầu là "ai" |
| cụm container | a container cluster | *container* — cần-**TÊI**-nơ, trọng âm âm thứ hai |
| ngân sách độ trễ | the latency budget | |
| phức tạp vận hành | operational complexity | |
| nhạy về độ trễ | latency-sensitive | |
| điểm cuối | an endpoint | |
| ngưỡng | a threshold | âm **th** /θ/ đầu, "THRESH-hâuld" |
| máy chủ | host | âm cuối **-st** phải bật ra, không thành "hâu" |

---

## Bài nói

> **Cách luyện:** bật ghi âm, nói **60–90 giây** cho mỗi đề, **không nhìn tài liệu** khi nói. Mỗi bài phải dùng ít nhất **sáu thuật ngữ** trong bảng trên. Với các đề so sánh, hãy nêu tiêu chí trước rồi mới nêu kết luận, đúng như khung ở Phần 7.

1. **Explain to a junior engineer** what a cold start is and why it matters much more for a user-facing API than for a nightly batch job.

2. **A colleague says:** *"Let's move everything to Lambda, it will be cheaper than our container cluster."* **Explain what is wrong with that**, and describe how you would check the claim before agreeing or refusing.

3. **Someone on your team wants to** open a connection pool of twenty connections inside each Lambda function so that queries are faster. **Explain why you would push back**, and describe what you would put in place instead.

4. **Describe what happens when** a serverless webhook handler that works perfectly in testing meets a real traffic spike of ten thousand events per minute, from the database's point of view.

5. **When would you choose** serverless over a container cluster? Give one workload for each choice from the same product, and state the criteria behind both decisions.

6. **Explain to a product manager** why the personalisation logic they want can run at the edge but the recommendation engine cannot.

7. **Explain to a junior engineer** how you would structure a serverless codebase so that vendor lock-in stays low and the business logic can still be tested without the cloud.
