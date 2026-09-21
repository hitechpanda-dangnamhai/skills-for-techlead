# Bài 14 — Case study: Notification system đa kênh
### Bản song ngữ: tiếng Anh bám cấu trúc tiếng Việt

> **Cách dùng:** đọc đoạn tiếng Việt trước → tự dịch trong đầu → so với bản tiếng Anh bên dưới → **đọc to bản tiếng Anh hai lần**. Lần một đọc chậm cho đúng từ, lần hai đọc liền mạch như đang nói với người phỏng vấn. Đây là bài về độ tin cậy, nên phần lớn từ vựng cần luyện là từ mô tả lỗi và cách phục hồi sau lỗi.

---

## Phần 1 — Bưu điện đa kênh và bốn lời hứa

**Tiếng Việt**

Chúng ta có thể hình dung hệ thống thông báo như một bưu điện phục vụ nhiều kênh cùng lúc. Bưu điện đó nhận một yêu cầu dạng hãy gửi tin này cho người dùng kia, rồi nó chọn kênh phù hợp, có thể là thư điện tử, thông báo đẩy hoặc tin nhắn ngắn. Sau đó nó gọi tới nhà cung cấp bên ngoài, chờ kết quả, và thử lại nếu lần gửi đó thất bại. Trong suốt quá trình, nó phải tôn trọng những gì người dùng đã chọn, ví dụ họ không muốn nhận loại tin này hoặc không muốn bị làm phiền ban đêm. Nói cách khác, phần khó không nằm ở việc gọi API của nhà cung cấp, mà nằm ở mọi thứ bao quanh lời gọi đó.

Toàn bộ thiết kế xoay quanh bốn lời hứa mà chúng ta phải giữ cùng lúc. Lời hứa thứ nhất là không gửi trùng, vì người dùng nhận ba email giống hệt nhau sẽ mất niềm tin vào sản phẩm. Lời hứa thứ hai là không làm mất, vì một mã xác thực không tới nơi có thể chặn người dùng khỏi tài khoản của họ. Lời hứa thứ ba là không làm phiền, vì gửi quá nhiều sẽ khiến người dùng tắt thông báo và chúng ta mất kênh đó vĩnh viễn. Lời hứa thứ tư là không chặn hệ thống chính, vì một nhà cung cấp chậm không được phép làm chậm luồng đặt hàng. Nếu chúng ta nêu được bốn lời hứa này ngay đầu buổi, phần còn lại của thiết kế gần như tự suy ra.

**English (bám cấu trúc tiếng Việt)**

We can picture a notification system as a post office serving several channels at the same time. That post office receives a request of the form "please send this message to that user", then it picks the appropriate channel, which may be email, push notification or a short text message. After that it calls out to an external provider, waits for the result, and retries if that send attempt fails. Throughout the process, it has to respect what the user has chosen, for example that they do not want this type of message or do not want to be disturbed at night. In other words, the hard part does not lie in calling the provider's API, it lies in everything surrounding that call.

The whole design revolves around four promises that we have to keep at the same time. The first promise is not to send duplicates, because a user who receives three identical emails will lose trust in the product. The second promise is not to lose messages, because a verification code that does not arrive can lock a user out of their own account. The third promise is not to spam, because sending too much will make users turn notifications off and we lose that channel for good. The fourth promise is not to block the main system, because a slow provider must not be allowed to slow down the checkout flow. If we can state these four promises at the start of the session, the rest of the design almost derives itself.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| phục vụ nhiều kênh cùng lúc | serving several channels at the same time |
| chọn kênh phù hợp | picks the appropriate channel |
| gọi tới nhà cung cấp bên ngoài | calls out to an external provider |
| tôn trọng những gì người dùng đã chọn | respect what the user has chosen |
| không muốn bị làm phiền ban đêm | do not want to be disturbed at night |
| mọi thứ bao quanh lời gọi đó | everything surrounding that call |
| bốn lời hứa mà chúng ta phải giữ cùng lúc | four promises that we have to keep at the same time |
| sẽ mất niềm tin vào sản phẩm | will lose trust in the product |
| chặn người dùng khỏi tài khoản của họ | lock a user out of their own account |
| chúng ta mất kênh đó vĩnh viễn | we lose that channel for good |
| phần còn lại của thiết kế gần như tự suy ra | the rest of the design almost derives itself |

**Thuật ngữ cần nhớ**

- hệ thống thông báo → **a notification system**
- kênh gửi → **a delivery channel**
- thông báo đẩy → **a push notification**
- nhà cung cấp → **a provider**
- gửi trùng → **a duplicate send**

---

## Phần 2 — Kiến trúc tối thiểu: hàng đợi ở giữa

**Tiếng Việt**

Kiến trúc tối thiểu gồm bốn khối nối tiếp nhau và chúng ta nên đọc chúng theo đúng thứ tự. Khối thứ nhất là điểm tiếp nhận, nơi các dịch vụ khác gửi yêu cầu thông báo tới bằng một lời gọi rất nhẹ. Khối thứ hai là hàng đợi, nơi yêu cầu nằm chờ và nhờ đó dịch vụ gọi không phải chờ nhà cung cấp. Khối thứ ba là các luồng xử lý theo từng kênh, mỗi luồng có một bộ chuyển đổi riêng vì mỗi nhà cung cấp có giao diện khác nhau. Khối thứ tư là cơ chế thử lại cùng với hàng đợi thư chết, nơi giữ lại những tin không thể gửi được sau nhiều lần cố gắng.

Quanh bốn khối đó, chúng ta còn cần ba dịch vụ phụ trợ mà nhiều ứng viên quên nhắc tới. Dịch vụ thứ nhất là dịch vụ khuôn mẫu, nơi dựng nội dung thật từ một khuôn mẫu và một tập biến. Dịch vụ thứ hai là dịch vụ tuỳ chọn người dùng, nơi lưu việc họ muốn nhận gì, qua kênh nào và vào giờ nào. Dịch vụ thứ ba là tầng quan sát, nơi theo dõi tỷ lệ gửi thành công, tỷ lệ trả về không tới nơi và tỷ lệ mở. Nếu chúng ta chỉ vẽ đường ống mà quên ba dịch vụ này, hậu quả là hệ thống chạy được nhưng không ai biết nó đang hoạt động tốt hay đang âm thầm mất tin.

**English (bám cấu trúc tiếng Việt)**

The minimal architecture consists of four blocks in a row and we should read them out in the right order. The first block is the ingestion point, where other services send notification requests in with a very light call. The second block is the queue, where the requests sit and wait, and thanks to which the calling service does not have to wait for the provider. The third block is the per-channel workers, each of which has its own adapter because every provider has a different interface. The fourth block is the retry mechanism together with a dead-letter queue, which holds the messages that cannot be delivered after many attempts.

Around those four blocks, we also need three supporting services that many candidates forget to mention. The first service is the template service, which builds the real content from a template and a set of variables. The second service is the user preference service, which stores what they want to receive, through which channel and at what hours. The third service is the observability layer, which tracks the successful delivery rate, the bounce rate and the open rate. If we only draw the pipeline and forget these three services, the consequence is that the system runs but nobody knows whether it is working well or quietly losing messages.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| bốn khối nối tiếp nhau | four blocks in a row |
| điểm tiếp nhận | the ingestion point |
| bằng một lời gọi rất nhẹ | with a very light call |
| nơi yêu cầu nằm chờ | where the requests sit and wait |
| mỗi luồng có một bộ chuyển đổi riêng | each has its own adapter |
| nơi giữ lại những tin không thể gửi được | which holds the messages that cannot be delivered |
| ba dịch vụ phụ trợ | three supporting services |
| dựng nội dung thật từ một khuôn mẫu và một tập biến | builds the real content from a template and a set of variables |
| tỷ lệ trả về không tới nơi | the bounce rate |
| đang âm thầm mất tin | quietly losing messages |

**Thuật ngữ cần nhớ**

- điểm tiếp nhận → **the ingestion point**
- bộ chuyển đổi nhà cung cấp → **a provider adapter**
- hàng đợi thư chết → **a dead-letter queue (DLQ)**
- dịch vụ khuôn mẫu → **the template service**
- khả năng quan sát → **observability**

---

## Phần 3 — Vì sao không bao giờ gọi nhà cung cấp trong request chính

**Tiếng Việt**

Cách làm sai phổ biến nhất là gọi thẳng nhà cung cấp ngay bên trong request đang phục vụ người dùng. Cách đó có vẻ đơn giản và nó chạy tốt trong lúc phát triển, vì nhà cung cấp trả lời trong vài trăm mili giây. Vấn đề xuất hiện khi nhà cung cấp chậm đi hoặc ngừng phản hồi, và lúc đó request của chúng ta treo theo họ. Người dùng bấm đặt hàng rồi ngồi nhìn vòng quay chờ, dù đơn hàng của họ đã được ghi thành công từ lâu. Nói cách khác, chúng ta để một dịch vụ bên ngoài quyết định độ sẵn sàng của chính sản phẩm mình.

Hàng đợi ở giữa cắt đứt sự phụ thuộc đó một cách gọn gàng. Dịch vụ đặt hàng chỉ ghi một thông điệp vào hàng đợi rồi trả lời người dùng ngay lập tức, và việc đó tốn vài mili giây. Nếu nhà cung cấp đang hỏng, các thông điệp cứ nằm lại trong hàng đợi và được gửi đi khi nhà cung cấp khoẻ trở lại. Chúng ta cũng có thể theo dõi độ dài hàng đợi để biết mình đang tụt lại bao xa, và đó là một chỉ số cảnh báo rất tốt. Nếu chúng ta gọi đồng bộ, hậu quả là một sự cố của bên thứ ba biến thành một sự cố của chúng ta, và khách hàng sẽ đổ lỗi cho chúng ta chứ không cho họ.

**English (bám cấu trúc tiếng Việt)**

The most common wrong approach is to call the provider directly inside the request that is serving the user. That approach looks simple and it runs fine during development, because the provider answers within a few hundred milliseconds. The problem appears when the provider slows down or stops responding, and at that point our request hangs along with them. The user presses "place order" and then sits watching a loading spinner, even though their order was written successfully long ago. In other words, we let an external service decide the availability of our own product.

A queue in the middle cuts that dependency neatly. The ordering service only writes one message into the queue and then answers the user immediately, and that costs a few milliseconds. If the provider is down, the messages simply stay in the queue and are sent out when the provider becomes healthy again. We can also watch the queue length to know how far behind we are falling, and that is a very good warning metric. If we call synchronously, the consequence is that a third party's incident turns into our incident, and customers will blame us rather than them.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| gọi thẳng nhà cung cấp | call the provider directly |
| ngay bên trong request đang phục vụ người dùng | inside the request that is serving the user |
| nó chạy tốt trong lúc phát triển | it runs fine during development |
| request của chúng ta treo theo họ | our request hangs along with them |
| ngồi nhìn vòng quay chờ | sits watching a loading spinner |
| quyết định độ sẵn sàng của chính sản phẩm mình | decide the availability of our own product |
| cắt đứt sự phụ thuộc đó một cách gọn gàng | cuts that dependency neatly |
| cứ nằm lại trong hàng đợi | simply stay in the queue |
| mình đang tụt lại bao xa | how far behind we are falling |
| khách hàng sẽ đổ lỗi cho chúng ta | customers will blame us |

**Thuật ngữ cần nhớ**

- gọi đồng bộ → **a synchronous call**
- sự phụ thuộc → **a dependency**
- bên thứ ba → **a third party**
- độ dài hàng đợi → **the queue length**
- tụt lại phía sau → **to fall behind**

---

## Phần 4 — Giao ít nhất một lần nên phải khử trùng lặp

**Tiếng Việt**

Hầu hết hàng đợi chỉ bảo đảm giao ít nhất một lần, nghĩa là cùng một thông điệp có thể được giao lại nhiều lần. Điều đó xảy ra khi một luồng xử lý chết sau khi đã làm việc nhưng trước khi kịp báo đã xử lý xong. Còn một trường hợp tinh vi hơn ở phía nhà cung cấp: họ đã gửi email thành công rồi mới hết thời gian chờ khi trả lời chúng ta. Chúng ta thấy lỗi nên chúng ta thử lại, và người dùng nhận email lần thứ hai dù cả hai bên đều làm đúng phần của mình. Đây chính là nguyên nhân của lời phàn nàn kinh điển rằng tôi nhận ba email giống hệt nhau.

Cách chữa là chúng ta gán cho mỗi thông báo một khoá bất biến trước khi đưa vào hàng đợi. Khoá đó thường được ghép từ mã người dùng, tên sự kiện và một chữ ký của nội dung, nên cùng một ý định sẽ luôn cho cùng một khoá. Trước khi gửi, luồng xử lý ghi khoá đó vào một kho khử trùng lặp với điều kiện chỉ ghi nếu chưa tồn tại. Nếu ghi thành công thì đây là lần đầu và chúng ta gửi, còn nếu ghi thất bại thì tin này đã được xử lý và chúng ta bỏ qua. Chúng ta cũng nên dùng khoá bất biến của chính nhà cung cấp nếu họ hỗ trợ, vì như vậy lớp bảo vệ nằm ở cả hai phía.

**English (bám cấu trúc tiếng Việt)**

Most queues only guarantee at-least-once delivery, which means the same message may be delivered again several times. That happens when a worker dies after it has done the work but before it manages to report that the work is finished. There is also a subtler case on the provider's side: they have already sent the email successfully and only then time out while replying to us. We see an error so we retry, and the user receives the email a second time even though both sides did their own part correctly. This is exactly the cause of the classic complaint that I received three identical emails.

The fix is that we give each notification an idempotency key before we put it into the queue. That key is usually assembled from the user id, the event name and a signature of the content, so the same intention always produces the same key. Before sending, the worker writes that key into a deduplication store under the condition that it only writes if the key does not exist yet. If the write succeeds then this is the first time and we send, while if the write fails then this message has already been handled and we skip it. We should also use the provider's own idempotency key if they support one, because then the protection layer sits on both sides.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| bảo đảm giao ít nhất một lần | guarantee at-least-once delivery |
| trước khi kịp báo đã xử lý xong | before it manages to report that the work is finished |
| một trường hợp tinh vi hơn | a subtler case |
| rồi mới hết thời gian chờ khi trả lời chúng ta | and only then time out while replying to us |
| dù cả hai bên đều làm đúng phần của mình | even though both sides did their own part correctly |
| một chữ ký của nội dung | a signature of the content |
| cùng một ý định sẽ luôn cho cùng một khoá | the same intention always produces the same key |
| với điều kiện chỉ ghi nếu chưa tồn tại | under the condition that it only writes if it does not exist yet |
| tin này đã được xử lý và chúng ta bỏ qua | this message has already been handled and we skip it |
| lớp bảo vệ nằm ở cả hai phía | the protection layer sits on both sides |

**Thuật ngữ cần nhớ**

- khoá bất biến → **an idempotency key**
- kho khử trùng lặp → **a deduplication store**
- giao lại → **redelivery**
- chỉ ghi nếu chưa có → **write if not exists**
- trạng thái giao tin → **delivery status**

---

## Phần 5 — Thử lại có kỷ luật và hàng đợi thư chết

**Tiếng Việt**

Khi nhà cung cấp trả lỗi, chúng ta thử lại, nhưng chúng ta phải thử lại một cách có kỷ luật. Cách sai là thử lại ngay lập tức và thử mãi, vì khi nhà cung cấp đang quá tải thì hàng nghìn lượt thử lại càng làm họ chết sâu hơn. Cách đúng là lùi dần theo cấp số nhân, nghĩa là chờ một giây, rồi hai giây, rồi bốn giây, và cứ thế nhân đôi. Chúng ta còn cộng thêm một khoảng ngẫu nhiên vào mỗi lần chờ, để các luồng xử lý không cùng quay lại tại một thời điểm. Nếu thiếu phần ngẫu nhiên này, mọi tin lỗi cùng một lúc sẽ tạo ra những đợt sóng thử lại đều đặn đập vào nhà cung cấp.

Sau một số lần thử nhất định, ví dụ năm lần, chúng ta ngừng thử và đẩy thông điệp vào hàng đợi thư chết. Hàng đợi đó không phải là thùng rác, mà là một nơi giữ bằng chứng để con người xem xét sau. Chúng ta nên đặt cảnh báo trên tốc độ tăng của hàng đợi này, vì nó tăng đột ngột thường là dấu hiệu sớm nhất của một sự cố ở phía nhà cung cấp. Chúng ta cũng nên phân biệt lỗi tạm thời với lỗi vĩnh viễn, vì một địa chỉ email không tồn tại thì thử lại bao nhiêu lần cũng vô ích. Nếu chúng ta thử lại vô hạn cho mọi loại lỗi, hậu quả là hàng đợi của chúng ta bị tắc bởi những tin không bao giờ gửi được, và những tin hợp lệ phải xếp hàng phía sau chúng.

**English (bám cấu trúc tiếng Việt)**

When the provider returns an error, we retry, but we have to retry in a disciplined way. The wrong way is to retry immediately and to retry forever, because when the provider is overloaded then thousands of retries only push them further down. The right way is exponential backoff, which means waiting one second, then two seconds, then four seconds, and doubling each time. We also add a random amount to each wait, so that the workers do not all come back at the same moment. Without this randomness, all the failed messages at one moment will create regular waves of retries hammering the provider.

After a certain number of attempts, for example five, we stop trying and push the message into the dead-letter queue. That queue is not a rubbish bin, it is a place that keeps the evidence for a human to review later. We should set an alert on the growth rate of this queue, because a sudden rise in it is usually the earliest sign of an incident on the provider's side. We should also distinguish temporary errors from permanent errors, because an email address that does not exist is useless to retry however many times we try. If we retry infinitely for every kind of error, the consequence is that our queue gets clogged by messages that can never be delivered, and the valid messages have to queue up behind them.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| thử lại một cách có kỷ luật | retry in a disciplined way |
| càng làm họ chết sâu hơn | only push them further down |
| và cứ thế nhân đôi | and doubling each time |
| để các luồng xử lý không cùng quay lại tại một thời điểm | so that the workers do not all come back at the same moment |
| những đợt sóng thử lại đều đặn | regular waves of retries |
| không phải là thùng rác | is not a rubbish bin |
| một nơi giữ bằng chứng để con người xem xét sau | a place that keeps the evidence for a human to review later |
| dấu hiệu sớm nhất của một sự cố | the earliest sign of an incident |
| thử lại bao nhiêu lần cũng vô ích | is useless to retry however many times |
| bị tắc bởi những tin không bao giờ gửi được | gets clogged by messages that can never be delivered |

**Thuật ngữ cần nhớ**

- lùi dần theo cấp số nhân → **exponential backoff**
- ngẫu nhiên hoá độ trễ → **jitter**
- lỗi tạm thời → **a transient error**
- lỗi vĩnh viễn → **a permanent error**
- cảnh báo → **an alert**

---

## Phần 6 — Tách chính sách khỏi việc giao tin

**Tiếng Việt**

Chúng ta nên tách rõ hai câu hỏi khác nhau: có nên gửi hay không, và nếu gửi thì gửi thế nào. Câu hỏi thứ nhất thuộc về chính sách, và nó gồm việc người dùng đã tắt loại thông báo này chưa, hiện có đang trong khung giờ yên tĩnh của họ không, và chúng ta đã gửi cho họ quá nhiều trong hôm nay chưa. Câu hỏi thứ hai thuộc về việc giao tin, và nó gồm việc dựng nội dung, chọn nhà cung cấp và xử lý lỗi. Chúng ta luôn áp chính sách trước khi chạm tới nhà cung cấp, vì gửi rồi mới hối tiếc là chuyện không thể sửa. Cách tách này cũng giúp chúng ta kiểm thử phần chính sách một cách độc lập, không cần dựng cả đường ống lên.

Việc tôn trọng khung giờ yên tĩnh có một chi tiết dễ sai mà chúng ta nên nói ra. Chúng ta không được vứt bỏ thông báo rơi vào giờ yên tĩnh, mà nên xếp nó lại để gửi vào đầu khung giờ cho phép. Tuy nhiên, chúng ta phải chừa ngoại lệ cho những tin mang tính giao dịch, ví dụ mã xác thực hoặc cảnh báo bảo mật, vì những tin đó luôn khẩn cấp. Chúng ta cũng nên gộp nhiều thông báo nhỏ thành một bản tóm tắt khi tần suất quá dày, vì mười thông báo rời rạc gây khó chịu hơn nhiều so với một thông báo tổng hợp. Nếu chúng ta bỏ qua tầng chính sách, hậu quả là tỷ lệ người dùng tắt hẳn thông báo tăng lên, và chúng ta mất một kênh mà sau này không mua lại được bằng tiền.

**English (bám cấu trúc tiếng Việt)**

We should clearly separate two different questions: whether we should send at all, and if we send, how we send. The first question belongs to policy, and it covers whether the user has turned this notification type off, whether we are currently inside their quiet hours, and whether we have already sent them too much today. The second question belongs to delivery, and it covers building the content, choosing the provider and handling errors. We always apply the policy before we touch the provider, because sending first and regretting afterwards is something that cannot be undone. This separation also lets us test the policy part independently, without having to stand up the whole pipeline.

Respecting quiet hours has one easily-missed detail that we should say out loud. We must not throw away a notification that falls inside quiet hours, we should instead queue it up to be sent at the start of the allowed window. However, we have to make an exception for transactional messages, for example verification codes or security alerts, because those messages are always urgent. We should also group many small notifications into one digest when the frequency becomes too dense, because ten separate notifications are far more annoying than one combined one. If we skip the policy layer, the consequence is that the rate of users turning notifications off entirely goes up, and we lose a channel that no amount of money can buy back later.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| có nên gửi hay không | whether we should send at all |
| thuộc về chính sách | belongs to policy |
| đang trong khung giờ yên tĩnh của họ | inside their quiet hours |
| gửi rồi mới hối tiếc là chuyện không thể sửa | sending first and regretting afterwards cannot be undone |
| không cần dựng cả đường ống lên | without having to stand up the whole pipeline |
| một chi tiết dễ sai | an easily-missed detail |
| xếp nó lại để gửi vào đầu khung giờ cho phép | queue it up to be sent at the start of the allowed window |
| những tin mang tính giao dịch | transactional messages |
| gộp nhiều thông báo nhỏ thành một bản tóm tắt | group many small notifications into one digest |
| sau này không mua lại được bằng tiền | that no amount of money can buy back later |

**Thuật ngữ cần nhớ**

- chính sách so với giao tin → **policy versus delivery**
- từ chối nhận → **to opt out**
- khung giờ yên tĩnh → **quiet hours**
- bản tóm tắt gộp → **a digest**
- tin giao dịch → **a transactional message**

---

## Phần 7 — Phát tán tới hàng triệu người cùng lúc

**Tiếng Việt**

Có những sự kiện chạm tới rất nhiều người cùng một lúc, ví dụ một người phát trực tiếp có năm triệu người theo dõi vừa lên sóng. Chúng ta tuyệt đối không gửi từng thông báo ngay trong request của sự kiện đó, vì request sẽ hết thời gian chờ từ lâu trước khi xong. Cách đúng là chúng ta ghi một sự kiện duy nhất vào hàng đợi, rồi để một tầng phát tán tách riêng nở nó ra thành hàng triệu thông báo. Tầng phát tán đó mở rộng ngang bằng cách thêm luồng xử lý, và chúng ta có thể tăng số luồng khi biết trước sẽ có sự kiện lớn. Nhờ đó, việc tạo sự kiện luôn nhanh, còn việc giao tin thì kéo dài bao lâu tuỳ vào năng lực thật.

Ở quy mô này, nhà cung cấp trở thành ràng buộc chính chứ không phải hệ thống của chúng ta. Mỗi nhà cung cấp đều có hạn mức số tin mỗi giây, và vượt hạn mức thì họ chặn hoặc tính thêm tiền. Vì vậy chúng ta gộp tin theo lô cho từng kênh và bóp nhịp gửi cho khớp với hạn mức đã ký. Chúng ta cũng nên trừu tượng hoá nhiều nhà cung cấp sau cùng một giao diện, để khi một bên hỏng thì chúng ta chuyển sang bên còn lại mà không phải sửa mã. Cuối cùng, chúng ta nên chấp nhận rằng một đợt phát tán lớn sẽ trải ra trong vài phút, và đó là điều hoàn toàn bình thường miễn là chúng ta nói trước với phía sản phẩm.

**English (bám cấu trúc tiếng Việt)**

There are events that reach a great many people at the same time, for example a streamer with five million followers who has just gone live. We must absolutely not send each notification inside the request of that event, because the request would time out long before it finishes. The correct way is that we write one single event into the queue, and then let a separate fan-out layer expand it into millions of notifications. That fan-out layer scales horizontally by adding workers, and we can raise the worker count when we know a large event is coming. Thanks to that, creating the event is always fast, while delivering the messages takes as long as the real capacity allows.

At this scale, the provider becomes the main constraint rather than our own system. Every provider has a quota of messages per second, and going over the quota means they either throttle us or charge us more. Therefore we batch the messages per channel and throttle our sending rate to match the quota we have signed for. We should also abstract several providers behind one common interface, so that when one of them fails we switch to the other without having to change code. Finally, we should accept that a large fan-out will spread out over several minutes, and that is completely normal as long as we tell the product side in advance.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| chạm tới rất nhiều người cùng một lúc | reach a great many people at the same time |
| vừa lên sóng | has just gone live |
| tuyệt đối không gửi từng thông báo ngay trong request | must absolutely not send each notification inside the request |
| nở nó ra thành hàng triệu thông báo | expand it into millions of notifications |
| khi biết trước sẽ có sự kiện lớn | when we know a large event is coming |
| kéo dài bao lâu tuỳ vào năng lực thật | takes as long as the real capacity allows |
| trở thành ràng buộc chính | becomes the main constraint |
| bóp nhịp gửi cho khớp với hạn mức đã ký | throttle our sending rate to match the quota we have signed for |
| trừu tượng hoá nhiều nhà cung cấp sau cùng một giao diện | abstract several providers behind one common interface |
| sẽ trải ra trong vài phút | will spread out over several minutes |

**Thuật ngữ cần nhớ**

- tầng phát tán → **the fan-out layer**
- hạn mức nhà cung cấp → **the provider quota**
- gộp theo lô → **batching**
- chuyển dự phòng → **failover**
- ràng buộc → **a constraint**

---

## Mô hình ghi nhớ

**Tiếng Việt**

Chúng ta đặt hàng đợi ở giữa, chúng ta áp chính sách trước khi giao tin, chúng ta khử trùng lặp bằng khoá bất biến, và chúng ta giữ những tin hỏng trong hàng đợi thư chết. Chúng ta không bao giờ gọi nhà cung cấp một cách đồng bộ ngay trong request chính.

**English (bám cấu trúc tiếng Việt)**

We put a queue in the middle, we apply the policy before delivery, we deduplicate with an idempotency key, and we keep the failed messages in a dead-letter queue. We never call a provider synchronously inside the main request.

---

## Bảng thuật ngữ tổng hợp

| Tiếng Việt | Tiếng Anh | Ghi chú phát âm |
|---|---|---|
| hệ thống thông báo | a notification system | |
| kênh gửi | a delivery channel | *channel* — "**CHA**-nợl", trọng âm đầu |
| thông báo đẩy | a push notification | |
| nhà cung cấp | a provider | prơ-**VAI**-đơ, trọng âm âm thứ hai |
| gửi trùng | a duplicate send | *duplicate* tính từ/danh từ đọc "**DIU**-plị-cợt"; động từ đuôi "-kêit" |
| điểm tiếp nhận | the ingestion point | *ingestion* — in-**JES**-chợn |
| bộ chuyển đổi nhà cung cấp | a provider adapter | |
| hàng đợi thư chết | a dead-letter queue (DLQ) | *queue* /kjuː/ — đọc như chữ "Q" |
| dịch vụ khuôn mẫu | the template service | *template* — "**TEM**-plêit", trọng âm đầu |
| khả năng quan sát | observability | ợb-zơ-vơ-**BI**-li-ti, trọng âm âm thứ tư |
| tỷ lệ trả về không tới nơi | the bounce rate | |
| gọi đồng bộ | a synchronous call | *synchronous* — "**SING**-crơ-nợs", trọng âm đầu |
| sự phụ thuộc | a dependency | đi-**PEN**-đợn-si, trọng âm âm thứ hai |
| bên thứ ba | a third party | *third* có âm **th** /θ/ đầu |
| độ dài hàng đợi | the queue length | *length* có âm **-ngth** cuối, khó — đọc trọn "leng-th" |
| tụt lại phía sau | to fall behind | |
| khoá bất biến | an idempotency key | ai-đem-**PO**-tần-si, trọng âm âm thứ ba |
| kho khử trùng lặp | a deduplication store | |
| giao lại | redelivery | |
| trạng thái giao tin | delivery status | |
| ít nhất một lần | at-least-once | |
| lùi dần theo cấp số nhân | exponential backoff | *exponential* — ek-spơ-**NEN**-shợl |
| ngẫu nhiên hoá độ trễ | jitter | "**JI**-tơ", âm đầu là "j" |
| lỗi tạm thời | a transient error | *transient* Anh — "**TRAN**-zi-ợnt", trọng âm đầu |
| lỗi vĩnh viễn | a permanent error | *permanent* — "**PƠ**-mơ-nợnt", trọng âm đầu |
| cảnh báo | an alert | ơ-**LƠT**, trọng âm âm thứ hai |
| chính sách so với giao tin | policy versus delivery | |
| từ chối nhận | to opt out | *opt* — âm cuối **-pt** phải bật ra |
| khung giờ yên tĩnh | quiet hours | *quiet* hai âm tiết "**KUAI**-ợt", khác *quite* một âm tiết |
| bản tóm tắt gộp | a digest | danh từ nhấn đầu: "**DAI**-jest" |
| tin giao dịch | a transactional message | |
| tầng phát tán | the fan-out layer | |
| hạn mức nhà cung cấp | the provider quota | *quota* Anh — "**KUÂU**-tơ", trọng âm đầu |
| gộp theo lô | batching | |
| chuyển dự phòng | failover | |
| ràng buộc | a constraint | cần-**STRÂYNT**, có cụm **str-** và **-nt** cuối |
| bóp nhịp | to throttle | "**THRO**-tợl", âm **th** /θ/ đầu |
| mã xác thực | a verification code | |
| tồn đọng | the backlog | |

---

## Bài nói

> **Cách luyện:** bật ghi âm, nói **60–90 giây** cho mỗi đề, **không nhìn tài liệu** khi nói. Mỗi bài phải dùng ít nhất **sáu thuật ngữ** trong bảng trên. Đề số 3 là đề khó nhất trong bài — hãy luyện nó nhiều lần, vì nó buộc bạn giải thích một lỗi mà cả hai bên đều làm đúng.

1. **Explain to a junior engineer** the four promises a notification system has to keep, and say which one is hardest to keep and why.

2. **A colleague says:** *"We'll call the email provider directly in the checkout handler so the customer gets the receipt before the page reloads."* **Explain why you would push back**, and describe what the user sees on the day the provider is slow.

3. **Describe what happens when** the provider successfully sends an email but times out while replying to you, and explain how your design stops the customer receiving it three times.

4. **Explain to a junior engineer** why retries need exponential backoff and jitter, and what happens to the provider if you retry immediately and forever.

5. **Someone on your team wants to** drop any notification that falls inside a user's quiet hours to keep the code simple. **Explain why you would push back**, and describe the exceptions you would carve out.

6. **When would you choose** to send a digest instead of individual notifications? Give one concrete feature and state what the user gains and loses.

7. **Describe what happens when** a streamer with five million followers goes live, following one notification from the event to the phone, and say where the real constraint sits.
