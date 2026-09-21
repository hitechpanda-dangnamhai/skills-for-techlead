# Bài 15 — E-commerce Checkout: chống bán quá số lượng & thanh toán bất biến
### Bản song ngữ: tiếng Anh bám cấu trúc tiếng Việt

> **Cách dùng:** đọc đoạn tiếng Việt trước → tự dịch trong đầu → so với bản tiếng Anh bên dưới → **đọc to bản tiếng Anh hai lần**. Lần một đọc chậm cho đúng từ, lần hai đọc liền mạch như đang nói với người phỏng vấn. Đây là case study duy nhất trong giáo trình mà chúng ta chọn nhất quán mạnh, nên hãy chú ý cách phát biểu lý do của lựa chọn đó.

---

## Phần 1 — Vì sao thanh toán là bài toán ngược với bảng tin

**Tiếng Việt**

Ở bảng tin và ở hệ thông báo, chúng ta luôn chọn nhất quán cuối cùng vì trễ vài giây không gây thiệt hại. Ở luồng thanh toán thì ngược lại hoàn toàn, vì một câu trả lời sai về tiền hoặc về tồn kho tạo ra thiệt hại thật. Nếu chúng ta bán quá số hàng đang có, chúng ta phải huỷ đơn của một khách đã trả tiền và xin lỗi họ. Nếu chúng ta trừ tiền hai lần, khách hàng mất niềm tin ngay lập tức và có thể báo cáo với ngân hàng. Vì vậy, ở đây chúng ta chọn nhất quán mạnh một cách có chủ đích, và chúng ta chấp nhận trả giá bằng độ trễ cùng thông lượng.

Câu này là câu chúng ta nên nói sớm trong buổi phỏng vấn, vì nó chứng minh chúng ta không áp một công thức cho mọi bài. Người phỏng vấn thường hỏi vì sao ở bài trước chúng ta chấp nhận dữ liệu cũ mà ở bài này lại không. Câu trả lời tốt là chúng ta gắn mức nhất quán với hậu quả nghiệp vụ chứ không gắn với sở thích kỹ thuật. Với bảng tin, hậu quả của việc sai là một người xem thấy bài chậm ba giây. Với thanh toán, hậu quả của việc sai là một khoản tiền chạy sai chỗ, và không có cách nào sửa nó mà không có người phải xin lỗi.

**English (bám cấu trúc tiếng Việt)**

In the news feed and in the notification system, we always choose eventual consistency because a delay of a few seconds causes no damage. In the checkout flow it is the complete opposite, because one wrong answer about money or about stock creates real damage. If we sell more items than we actually have, we have to cancel the order of a customer who has already paid and apologise to them. If we charge twice, the customer loses trust immediately and may report us to their bank. Therefore, here we choose strong consistency deliberately, and we accept paying the price in latency and in throughput.

This is a sentence we should say early in the interview, because it proves that we do not apply one formula to every problem. Interviewers often ask why in the previous exercise we accepted stale data while in this one we do not. A good answer is that we tie the consistency level to the business consequence rather than tying it to a technical preference. For the feed, the consequence of being wrong is that one viewer sees a post three seconds late. For the checkout, the consequence of being wrong is that a sum of money goes to the wrong place, and there is no way to fix it without somebody having to apologise.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| ngược lại hoàn toàn | the complete opposite |
| tạo ra thiệt hại thật | creates real damage |
| bán quá số hàng đang có | sell more items than we actually have |
| một khách đã trả tiền | a customer who has already paid |
| có thể báo cáo với ngân hàng | may report us to their bank |
| một cách có chủ đích | deliberately |
| chúng ta không áp một công thức cho mọi bài | we do not apply one formula to every problem |
| gắn mức nhất quán với hậu quả nghiệp vụ | tie the consistency level to the business consequence |
| một khoản tiền chạy sai chỗ | a sum of money goes to the wrong place |
| không có cách nào sửa nó mà không có người phải xin lỗi | no way to fix it without somebody having to apologise |

**Thuật ngữ cần nhớ**

- luồng thanh toán → **the checkout flow**
- bán quá số lượng → **overselling**
- tồn kho → **stock** / **inventory**
- nhất quán mạnh → **strong consistency**
- hậu quả nghiệp vụ → **the business consequence**

---

## Phần 2 — Khoảng trống giữa đọc và ghi

**Tiếng Việt**

Cách viết ngây thơ là chúng ta đọc số tồn kho, kiểm tra xem nó có lớn hơn không, rồi ghi lại số đã trừ. Đoạn mã đó đọc lên nghe rất hợp lý, và nó chạy đúng mọi lần khi chỉ có một người mua. Vấn đề nằm ở khoảng trống giữa lúc đọc và lúc ghi, vì trong khoảng đó một request khác có thể chen vào. Hai request cùng đọc thấy còn một món, cả hai cùng thấy điều kiện thoả mãn, và cả hai cùng ghi số không. Kết quả là chúng ta bán hai món trong khi kho chỉ có một, và đây chính là hiện tượng mất cập nhật kinh điển.

Điều nguy hiểm là lỗi này gần như không bao giờ xuất hiện khi chúng ta kiểm thử bằng tay. Muốn nó xảy ra, hai request phải rơi vào đúng cùng một khoảnh khắc rất ngắn, nên xác suất ở tải thấp là rất nhỏ. Nhưng ở tải cao, khoảnh khắc đó lặp lại hàng nghìn lần mỗi phút, nên chuyện hiếm trở thành chuyện thường. Đây là lý do các sự cố bán quá số lượng luôn xuất hiện đúng vào ngày khuyến mãi lớn nhất. Nói cách khác, chúng ta không gặp lỗi này vì hệ thống hỏng, chúng ta gặp nó vì hệ thống cuối cùng đã đủ đông khách.

**English (bám cấu trúc tiếng Việt)**

The naive way to write it is that we read the stock number, check whether it is greater than zero, and then write back the decremented number. That piece of code reads very reasonably, and it runs correctly every time when only one person is buying. The problem lies in the gap between the moment of reading and the moment of writing, because within that gap another request can slip in. Two requests both read that there is one item left, both see the condition satisfied, and both write zero. The result is that we sell two items while the warehouse only has one, and this is exactly the classic lost update phenomenon.

The dangerous thing is that this bug almost never shows up when we test by hand. For it to happen, two requests have to fall into exactly the same very short moment, so the probability at low load is tiny. But at high load, that moment repeats thousands of times per minute, so the rare thing becomes the normal thing. This is the reason why overselling incidents always appear on exactly the biggest sale day. In other words, we do not meet this bug because the system broke, we meet it because the system has finally become busy enough.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| cách viết ngây thơ | the naive way to write it |
| ghi lại số đã trừ | write back the decremented number |
| đoạn mã đó đọc lên nghe rất hợp lý | that piece of code reads very reasonably |
| khoảng trống giữa lúc đọc và lúc ghi | the gap between the moment of reading and the moment of writing |
| một request khác có thể chen vào | another request can slip in |
| cả hai cùng thấy điều kiện thoả mãn | both see the condition satisfied |
| hiện tượng mất cập nhật kinh điển | the classic lost update phenomenon |
| xác suất ở tải thấp là rất nhỏ | the probability at low load is tiny |
| chuyện hiếm trở thành chuyện thường | the rare thing becomes the normal thing |
| hệ thống cuối cùng đã đủ đông khách | the system has finally become busy enough |

**Thuật ngữ cần nhớ**

- tranh chấp → **a race condition**
- mất cập nhật → **a lost update**
- đọc sửa ghi → **read-modify-write**
- khoảng trống → **the gap**
- kho hàng → **the warehouse**

---

## Phần 3 — Ba cách trừ tồn kho một cách nguyên tử

**Tiếng Việt**

Cách thứ nhất và cũng là cách mặc định là đưa điều kiện vào ngay trong câu lệnh cập nhật. Chúng ta viết một câu trừ tồn kho kèm điều kiện tồn kho phải lớn hơn không, rồi chúng ta đọc số dòng bị ảnh hưởng. Nếu số dòng bằng một thì chúng ta giữ được hàng, còn nếu bằng không thì hàng đã hết và chúng ta từ chối. Cách này không còn khoảng trống nào giữa kiểm tra và ghi, vì cả hai nằm trong cùng một câu lệnh mà cơ sở dữ liệu thực hiện nguyên tử. Đây là lựa chọn rẻ nhất và đúng nhất cho phần lớn trường hợp, nên chúng ta nên nêu nó trước tiên.

Cách thứ hai là khoá bi quan, tức là chúng ta chọn dòng dữ liệu kèm mệnh đề khoá để giữ nó tới hết giao dịch. Chúng ta dùng cách này khi logic phức tạp, ví dụ phải kiểm tra nhiều sản phẩm trong một combo trước khi quyết định. Đổi lại, khoá được giữ lâu hơn nên thông lượng giảm, và một sản phẩm nóng có thể tạo ra hàng dài các giao dịch xếp hàng chờ. Cách thứ ba là đẩy điểm tranh chấp sang Redis và dùng lệnh giảm nguyên tử của nó. Cách này cực nhanh vì Redis xử lý tuần tự, nhưng chúng ta phải đồng bộ ngược về cơ sở dữ liệu và phải có kế hoạch cho tình huống Redis chết.

**English (bám cấu trúc tiếng Việt)**

The first way, and also the default way, is to put the condition inside the update statement itself. We write one statement that decrements the stock together with a condition that the stock must be greater than zero, and then we read the number of affected rows. If the number of rows is one then we have secured the item, while if it is zero then the item is sold out and we refuse. This way leaves no gap at all between the check and the write, because both sit inside one statement that the database executes atomically. This is the cheapest and most correct option for most cases, so we should mention it first.

The second way is pessimistic locking, that is, we select the data row with a locking clause in order to hold it until the end of the transaction. We use this way when the logic is complex, for example when we have to check several products in a bundle before deciding. In exchange, the lock is held for longer so the throughput drops, and one hot product can create a long line of transactions queuing up. The third way is to move the point of contention over to Redis and use its atomic decrement command. This way is extremely fast because Redis processes sequentially, but we have to synchronise back to the database and we have to have a plan for the situation where Redis dies.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| đưa điều kiện vào ngay trong câu lệnh cập nhật | put the condition inside the update statement itself |
| số dòng bị ảnh hưởng | the number of affected rows |
| thì chúng ta giữ được hàng | then we have secured the item |
| hàng đã hết và chúng ta từ chối | the item is sold out and we refuse |
| không còn khoảng trống nào giữa kiểm tra và ghi | leaves no gap at all between the check and the write |
| khoá bi quan | pessimistic locking |
| kèm mệnh đề khoá | with a locking clause |
| hàng dài các giao dịch xếp hàng chờ | a long line of transactions queuing up |
| đẩy điểm tranh chấp sang Redis | move the point of contention over to Redis |
| phải có kế hoạch cho tình huống Redis chết | have to have a plan for the situation where Redis dies |

**Thuật ngữ cần nhớ**

- cập nhật có điều kiện → **a conditional update**
- số dòng bị ảnh hưởng → **affected rows**
- khoá bi quan → **pessimistic locking**
- khoá lạc quan → **optimistic locking**
- điểm tranh chấp → **the point of contention**

---

## Phần 4 — Giữ chỗ có thời hạn

**Tiếng Việt**

Giữa lúc người dùng bấm mua và lúc họ trả tiền xong có một khoảng thời gian, và khoảng đó là nguồn của rất nhiều rắc rối. Nếu chúng ta trừ hẳn tồn kho ngay lúc thêm vào giỏ hàng, những người bỏ giỏ sẽ khoá hàng lại vĩnh viễn. Nếu chúng ta không trừ gì cả cho tới lúc thanh toán xong, hai người có thể cùng thanh toán cho món hàng cuối cùng. Lời giải là giữ chỗ có thời hạn, nghĩa là chúng ta chuyển số lượng từ trạng thái còn bán sang trạng thái đã giữ, kèm một thời điểm hết hạn. Nhờ vậy, hàng được bảo vệ cho người đang trả tiền nhưng không bị khoá mãi mãi.

Về mô hình dữ liệu, chúng ta nên tách rõ ba trạng thái thay vì chỉ giữ một con số. Trạng thái thứ nhất là số còn bán được, trạng thái thứ hai là số đang được giữ chỗ, và trạng thái thứ ba là số đã bán hẳn. Khi thanh toán thành công, chúng ta chuyển từ đã giữ sang đã bán, và đó là một phép chuyển chứ không phải một phép trừ mới. Khi hết hạn hoặc người dùng huỷ, chúng ta trả số lượng về lại phần còn bán được, và đó chính là một hành động bù trừ. Chúng ta cần một tác vụ quét các giữ chỗ quá hạn, hoặc dùng cơ chế tự hết hạn của Redis, để không ai phải nhớ nhả hàng bằng tay.

**English (bám cấu trúc tiếng Việt)**

Between the moment the user presses buy and the moment they finish paying there is a period of time, and that period is the source of a great deal of trouble. If we deduct the stock outright at the moment of adding to the cart, the people who abandon their cart will lock the items away forever. If we deduct nothing at all until the payment finishes, two people may both pay for the very last item. The solution is a reservation with an expiry, which means we move the quantity from the available state into the reserved state, together with an expiry time. Thanks to that, the item is protected for the person who is paying but it is not locked away forever.

On the data model, we should clearly separate three states rather than keeping only one number. The first state is the quantity still available for sale, the second state is the quantity currently reserved, and the third state is the quantity already sold. When the payment succeeds, we move from reserved to sold, and that is a transfer rather than a new deduction. When the reservation expires or the user cancels, we return the quantity back to the available part, and that is exactly a compensating action. We need a job that sweeps the expired reservations, or we use Redis's own expiry mechanism, so that nobody has to remember to release the stock by hand.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| nguồn của rất nhiều rắc rối | the source of a great deal of trouble |
| trừ hẳn tồn kho ngay lúc thêm vào giỏ hàng | deduct the stock outright at the moment of adding to the cart |
| những người bỏ giỏ | the people who abandon their cart |
| khoá hàng lại vĩnh viễn | lock the items away forever |
| cùng thanh toán cho món hàng cuối cùng | both pay for the very last item |
| giữ chỗ có thời hạn | a reservation with an expiry |
| số còn bán được | the quantity still available for sale |
| đó là một phép chuyển chứ không phải một phép trừ mới | that is a transfer rather than a new deduction |
| một tác vụ quét các giữ chỗ quá hạn | a job that sweeps the expired reservations |
| không ai phải nhớ nhả hàng bằng tay | nobody has to remember to release the stock by hand |

**Thuật ngữ cần nhớ**

- giữ chỗ → **a reservation**
- còn bán được → **available**
- đang giữ chỗ → **reserved**
- đã bán → **sold**
- bỏ giỏ hàng → **cart abandonment**

---

## Phần 5 — Tính bất biến khi lặp lại trong thanh toán

**Tiếng Việt**

Mạng có thể hết thời gian chờ, người dùng có thể bấm nút trả tiền hai lần, và máy khách có thể tự động thử lại. Trong cả ba trường hợp, cùng một ý định trả tiền có thể tới máy chủ nhiều lần, và nếu chúng ta xử lý ngây thơ thì khách bị trừ tiền hai lần. Cách chặn chuẩn của ngành là khoá bất biến, trong đó máy khách sinh ra một khoá duy nhất cho mỗi ý định trả tiền. Trước khi gọi cổng thanh toán, máy chủ ghi khoá đó vào một kho với điều kiện chỉ ghi nếu chưa tồn tại. Nếu khoá là mới thì chúng ta thực hiện thu tiền rồi lưu kết quả gắn với khoá đó, còn nếu khoá đã có thì chúng ta trả lại đúng kết quả cũ và không thu tiền lần nữa.

Điểm quan trọng cần nói rõ là khoá phải đại diện cho ý định chứ không đại diện cho lần gọi. Nếu máy khách sinh khoá mới mỗi lần bấm, thì bấm hai lần vẫn tạo ra hai giao dịch và cơ chế trở nên vô dụng. Vì vậy, khoá thường được sinh một lần khi màn hình thanh toán mở ra, rồi giữ nguyên cho mọi lần thử lại của cùng màn hình đó. Chúng ta cũng nên chuyển tiếp khoá đó xuống cổng thanh toán, vì các cổng nghiêm túc đều nhận một tiêu đề bất biến. Nhờ vậy, lớp bảo vệ tồn tại ở cả phía chúng ta lẫn phía họ, và mọi request làm thay đổi tiền đều an toàn khi bị lặp lại.

**English (bám cấu trúc tiếng Việt)**

The network may time out, the user may press the pay button twice, and the client may retry automatically. In all three cases, the same payment intention may reach the server several times, and if we handle it naively then the customer is charged twice. The industry-standard way to block this is an idempotency key, in which the client generates one unique key for each payment intention. Before calling the payment gateway, the server writes that key into a store under the condition that it only writes if the key does not exist yet. If the key is new then we perform the charge and store the result attached to that key, while if the key already exists then we return exactly the old result and we do not charge again.

The important point to state clearly is that the key must represent the intention rather than represent the call. If the client generates a new key on every press, then pressing twice still creates two transactions and the mechanism becomes useless. Therefore, the key is usually generated once when the payment screen opens, and then kept unchanged for every retry from that same screen. We should also forward that key down to the payment gateway, because every serious gateway accepts an idempotency header. Thanks to that, the protection layer exists both on our side and on theirs, and every request that changes money is safe when it is repeated.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| người dùng có thể bấm nút trả tiền hai lần | the user may press the pay button twice |
| nếu chúng ta xử lý ngây thơ | if we handle it naively |
| cách chặn chuẩn của ngành | the industry-standard way to block this |
| một khoá duy nhất cho mỗi ý định trả tiền | one unique key for each payment intention |
| lưu kết quả gắn với khoá đó | store the result attached to that key |
| trả lại đúng kết quả cũ | return exactly the old result |
| khoá phải đại diện cho ý định chứ không đại diện cho lần gọi | the key must represent the intention rather than the call |
| cơ chế trở nên vô dụng | the mechanism becomes useless |
| giữ nguyên cho mọi lần thử lại | kept unchanged for every retry |
| an toàn khi bị lặp lại | safe when it is repeated |

**Thuật ngữ cần nhớ**

- khoá bất biến → **an idempotency key**
- cổng thanh toán → **the payment gateway**
- thu tiền → **to charge**
- ý định trả tiền → **a payment intention**
- hoàn tiền → **a refund**

---

## Phần 6 — Saga thay cho cam kết hai pha, và mẫu hộp thư đi

**Tiếng Việt**

Một lượt thanh toán chạm vào nhiều dịch vụ: tồn kho, thanh toán, đơn hàng và giao vận. Chúng ta muốn tất cả cùng thành công hoặc tất cả cùng được huỷ, và cách kinh điển trên lý thuyết là cam kết hai pha. Cách đó hoạt động trên giấy nhưng nó khoá tài nguyên xuyên nhiều dịch vụ, nó chậm, và nó treo khi bên điều phối chết giữa chừng. Vì vậy, ở quy mô lớn gần như không ai dùng nó, và chúng ta thay bằng saga. Saga là một chuỗi bước cục bộ, mỗi bước tự cam kết trong dịch vụ của nó, và mỗi bước có một hành động bù trừ tương ứng nếu bước sau thất bại.

Chuỗi bù trừ chạy ngược lại theo đúng thứ tự đã làm. Nếu thu tiền thất bại, chúng ta nhả phần tồn kho đã giữ. Nếu tạo đơn hàng thất bại sau khi đã thu tiền, chúng ta hoàn tiền rồi nhả tồn kho. Có một chỗ nứt tinh vi giữa việc ghi cơ sở dữ liệu và việc bắn thông điệp ra ngoài, vì máy chủ có thể chết đúng giữa hai việc đó. Chúng ta bịt chỗ nứt này bằng mẫu hộp thư đi, nghĩa là chúng ta ghi sự kiện vào một bảng hộp thư đi trong cùng giao dịch với thay đổi nghiệp vụ, rồi một tiến trình chuyển tiếp đọc bảng đó và phát lên đường truyền thông điệp.

**English (bám cấu trúc tiếng Việt)**

One checkout touches several services: inventory, payment, orders and shipping. We want them all to succeed together or all to be cancelled together, and the classic textbook way is two-phase commit. That way works on paper but it locks resources across several services, it is slow, and it hangs when the coordinator dies halfway. Therefore, at large scale almost nobody uses it, and we replace it with a saga. A saga is a chain of local steps, each step commits on its own inside its own service, and each step has a corresponding compensating action if a later step fails.

The compensation chain runs backwards in exactly the order things were done. If the charge fails, we release the stock we had reserved. If creating the order fails after the money has been taken, we refund the payment and then release the stock. There is a subtle crack between writing to the database and publishing the message outwards, because the server may die exactly between those two actions. We seal this crack with the outbox pattern, which means we write the event into an outbox table inside the same transaction as the business change, and then a relay process reads that table and publishes onto the message bus.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| cách kinh điển trên lý thuyết | the classic textbook way |
| nó khoá tài nguyên xuyên nhiều dịch vụ | it locks resources across several services |
| nó treo khi bên điều phối chết giữa chừng | it hangs when the coordinator dies halfway |
| gần như không ai dùng nó | almost nobody uses it |
| mỗi bước tự cam kết trong dịch vụ của nó | each step commits on its own inside its own service |
| chạy ngược lại theo đúng thứ tự đã làm | runs backwards in exactly the order things were done |
| chúng ta nhả phần tồn kho đã giữ | we release the stock we had reserved |
| một chỗ nứt tinh vi | a subtle crack |
| chúng ta bịt chỗ nứt này | we seal this crack |
| một tiến trình chuyển tiếp đọc bảng đó | a relay process reads that table |

**Thuật ngữ cần nhớ**

- cam kết hai pha → **two-phase commit (2PC)**
- hành động bù trừ → **a compensating action**
- mẫu hộp thư đi → **the outbox pattern**
- tiến trình chuyển tiếp → **a relay**
- đường truyền thông điệp → **the message bus**

---

## Phần 7 — Bán hàng chớp nhoáng

**Tiếng Việt**

Bán hàng chớp nhoáng là tình huống một trăm nghìn người giành một nghìn món trong đúng một giây. Toàn bộ lưu lượng đó dồn vào một dòng dữ liệu duy nhất, nên chúng ta gặp lại bài toán khoá nóng ở mức khắc nghiệt nhất. Nếu mọi request đều đánh thẳng vào cơ sở dữ liệu, các giao dịch sẽ xếp hàng chờ khoá và cơ sở dữ liệu sẽ gục trong vài giây. Điều cần nhận ra là chín mươi chín phần trăm số người sẽ thua cuộc, nên việc để họ chờ trong hàng khoá là hoàn toàn lãng phí. Nguyên tắc thiết kế là chúng ta phải từ chối người thua thật nhanh và chỉ để người thắng đi tiếp.

Cách làm gồm bốn bước, và chúng ta nên đọc chúng theo thứ tự. Bước một, chúng ta đặt một phòng chờ trước cửa và thả người vào từ từ bằng vé, để đám đông không đập vào hệ thống cùng một lúc. Bước hai, chúng ta nạp trước số lượng tồn kho vào Redis và dùng lệnh giảm nguyên tử, nên mỗi request chỉ tốn một thao tác trong bộ nhớ. Bước ba, ai giảm ra kết quả không âm thì được giữ chỗ, còn những người còn lại nhận ngay câu trả lời hết hàng. Bước bốn, chúng ta đẩy danh sách người thắng xuống cơ sở dữ liệu qua hàng đợi, nhờ đó cơn sóng ghi được làm phẳng thành một dòng đều. Cuối cùng, chúng ta cần một tác vụ đối soát định kỳ giữa Redis và cơ sở dữ liệu, cùng một kế hoạch khôi phục cho trường hợp Redis chết giữa đợt bán.

**English (bám cấu trúc tiếng Việt)**

A flash sale is the situation where one hundred thousand people compete for one thousand items within exactly one second. All of that traffic converges on one single data row, so we meet the hot key problem again at its most brutal. If every request hits the database directly, the transactions will queue up waiting for the lock and the database will collapse within a few seconds. The thing to realise is that ninety-nine percent of the people are going to lose, so making them wait in a lock queue is completely wasteful. The design principle is that we have to reject the losers very fast and only let the winners carry on.

The approach has four steps, and we should read them out in order. Step one, we put a waiting room in front of the door and let people in gradually with tokens, so that the crowd does not hit the system all at once. Step two, we preload the stock quantity into Redis and use the atomic decrement command, so each request costs only one operation in memory. Step three, whoever decrements to a non-negative result gets the reservation, while the remaining people receive an immediate sold-out answer. Step four, we push the list of winners down to the database through a queue, thanks to which the wave of writes is flattened into a steady stream. Finally, we need a periodic reconciliation job between Redis and the database, together with a recovery plan for the case where Redis dies mid-sale.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| giành một nghìn món trong đúng một giây | compete for one thousand items within exactly one second |
| dồn vào một dòng dữ liệu duy nhất | converges on one single data row |
| ở mức khắc nghiệt nhất | at its most brutal |
| sẽ gục trong vài giây | will collapse within a few seconds |
| việc để họ chờ trong hàng khoá là hoàn toàn lãng phí | making them wait in a lock queue is completely wasteful |
| từ chối người thua thật nhanh | reject the losers very fast |
| thả người vào từ từ bằng vé | let people in gradually with tokens |
| nạp trước số lượng tồn kho vào Redis | preload the stock quantity into Redis |
| giảm ra kết quả không âm | decrements to a non-negative result |
| cơn sóng ghi được làm phẳng thành một dòng đều | the wave of writes is flattened into a steady stream |
| một tác vụ đối soát định kỳ | a periodic reconciliation job |

**Thuật ngữ cần nhớ**

- bán hàng chớp nhoáng → **a flash sale**
- phòng chờ → **a waiting room**
- nạp trước → **to preload**
- thất bại nhanh → **to fail fast**
- đối soát → **reconciliation**

---

## Phần 8 — Phản biện câu nói "giao dịch cơ sở dữ liệu là đủ"

**Tiếng Việt**

Có một câu nói rất hay gặp trong các buổi thiết kế: chỉ cần dùng giao dịch của cơ sở dữ liệu là đủ chống bán quá số lượng. Câu đó đúng một nửa, vì giao dịch quả thật giải quyết được tranh chấp bên trong một cơ sở dữ liệu duy nhất. Nhưng nó thiếu ba thứ, và chúng ta nên nêu đủ ba thứ đó khi phản biện. Thứ nhất, với một sản phẩm cực nóng, các giao dịch xếp hàng nối đuôi nhau và thông lượng sụp xuống đúng vào lúc chúng ta cần nó nhất. Thứ hai, tiền nằm ở một dịch vụ khác, nên một giao dịch của cơ sở dữ liệu chúng ta không thể bao trùm cả việc thu tiền.

Thứ ba, việc giữ hàng trong lúc người dùng nhập thẻ là một bài toán về thời gian, và giao dịch không giải được bài toán thời gian. Một giao dịch mở suốt mười lăm phút chờ người dùng gõ số thẻ là điều không ai chấp nhận, vì nó giữ khoá và làm chết cơ sở dữ liệu. Vì vậy, kết luận đúng là giao dịch là điều kiện cần chứ không phải điều kiện đủ. Chúng ta vẫn dùng giao dịch cho phần trừ tồn kho, nhưng chúng ta cần giữ chỗ có thời hạn cho khoảng chờ, cần khoá bất biến cho phần tiền, và cần saga cho phần nhiều dịch vụ. Cách trả lời này ghi điểm vì nó không bác bỏ hoàn toàn ý kiến của đồng nghiệp, mà chỉ ra chính xác phần nào đúng và phần nào còn thiếu.

**English (bám cấu trúc tiếng Việt)**

There is a sentence that comes up very often in design sessions: a database transaction alone is enough to prevent overselling. That sentence is half right, because a transaction really does solve contention inside one single database. But it is missing three things, and we should state all three when we push back. First, for an extremely hot product, the transactions queue up one behind another and the throughput collapses at exactly the moment when we need it most. Second, the money sits in another service, so one transaction in our database cannot cover the act of taking the payment.

Third, holding the stock while the user types in their card is a problem about time, and a transaction cannot solve a problem about time. A transaction held open for fifteen minutes waiting for a user to type card numbers is something nobody accepts, because it holds locks and kills the database. Therefore, the correct conclusion is that a transaction is a necessary condition rather than a sufficient one. We still use a transaction for the stock deduction, but we need a reservation with an expiry for the waiting period, we need an idempotency key for the money part, and we need a saga for the multi-service part. This way of answering scores because it does not reject the colleague's idea entirely, it points out exactly which part is right and which part is still missing.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| câu đó đúng một nửa | that sentence is half right |
| giải quyết được tranh chấp bên trong một cơ sở dữ liệu duy nhất | solves contention inside one single database |
| nó thiếu ba thứ | it is missing three things |
| xếp hàng nối đuôi nhau | queue up one behind another |
| đúng vào lúc chúng ta cần nó nhất | at exactly the moment when we need it most |
| không thể bao trùm cả việc thu tiền | cannot cover the act of taking the payment |
| trong lúc người dùng nhập thẻ | while the user types in their card |
| là điều kiện cần chứ không phải điều kiện đủ | is a necessary condition rather than a sufficient one |
| không bác bỏ hoàn toàn ý kiến của đồng nghiệp | does not reject the colleague's idea entirely |
| phần nào đúng và phần nào còn thiếu | which part is right and which part is still missing |

**Thuật ngữ cần nhớ**

- giao dịch → **a transaction**
- tranh giành tài nguyên → **contention**
- điều kiện cần → **a necessary condition**
- điều kiện đủ → **a sufficient condition**
- thời gian chờ khoá → **lock wait time**

---

## Mô hình ghi nhớ

**Tiếng Việt**

Chúng ta đóng khoảng trống giữa đọc và ghi bằng thao tác nguyên tử, chúng ta giữ chỗ có thời hạn, chúng ta thu tiền đúng một lần bằng khoá bất biến, và với nhiều dịch vụ thì chúng ta bù trừ bằng saga chứ không khoá chéo bằng cam kết hai pha.

**English (bám cấu trúc tiếng Việt)**

We close the gap between reading and writing with an atomic operation, we reserve with an expiry, we charge exactly once with an idempotency key, and across several services we compensate with a saga rather than locking across them with two-phase commit.

---

## Bảng thuật ngữ tổng hợp

| Tiếng Việt | Tiếng Anh | Ghi chú phát âm |
|---|---|---|
| luồng thanh toán | the checkout flow | |
| bán quá số lượng | overselling | |
| tồn kho | stock / inventory | *inventory* Anh — "**IN**-vần-tri", trọng âm đầu, ba âm tiết |
| nhất quán mạnh | strong consistency | *consistency* — cần-**SIS**-tần-si |
| hậu quả nghiệp vụ | the business consequence | *consequence* — "**CON**-sị-kwợns", trọng âm đầu |
| tranh chấp | a race condition | |
| mất cập nhật | a lost update | |
| đọc sửa ghi | read-modify-write | |
| kho hàng | the warehouse | "**UE**-hau-s", chữ *w* đầu đọc rõ |
| cập nhật có điều kiện | a conditional update | |
| số dòng bị ảnh hưởng | affected rows | *row* /rəʊ/ — "râu", không đọc "rao" |
| khoá bi quan | pessimistic locking | *pessimistic* — pe-sị-**MIS**-tịc, trọng âm âm thứ ba |
| khoá lạc quan | optimistic locking | op-tị-**MIS**-tịc, trọng âm âm thứ ba |
| điểm tranh chấp | the point of contention | *contention* — cần-**TEN**-shợn |
| tranh giành tài nguyên | contention | |
| giữ chỗ | a reservation | re-zơ-**VÂY**-shợn, trọng âm âm thứ ba |
| còn bán được | available | ơ-**VÂY**-lơ-bợl, trọng âm âm thứ hai |
| đang giữ chỗ | reserved | |
| đã bán | sold | âm cuối **-ld** phải bật ra |
| bỏ giỏ hàng | cart abandonment | *abandonment* — ơ-**BAN**-đợn-mợnt |
| hết hạn | to expire / expiry | Anh *expiry* — ịc-**SPAI**-ơ-ri |
| khoá bất biến | an idempotency key | ai-đem-**PO**-tần-si |
| cổng thanh toán | the payment gateway | |
| thu tiền | to charge | |
| ý định trả tiền | a payment intention | |
| hoàn tiền | a refund | danh từ nhấn đầu "**RI**-fănd"; động từ nhấn sau |
| cam kết hai pha | two-phase commit (2PC) | *phase* /feɪz/ — "phêiz" |
| hành động bù trừ | a compensating action | "**COM**-pần-sêi-ting", trọng âm đầu |
| mẫu hộp thư đi | the outbox pattern | |
| tiến trình chuyển tiếp | a relay | danh từ nhấn đầu: "**RI**-lêi" |
| đường truyền thông điệp | the message bus | |
| giao dịch | a transaction | |
| quay lui | to roll back | |
| bán hàng chớp nhoáng | a flash sale | |
| phòng chờ | a waiting room | |
| nạp trước | to preload | |
| thất bại nhanh | to fail fast | |
| đối soát | reconciliation | re-cơn-si-li-**ÂY**-shợn, trọng âm áp chót |
| làm phẳng | to flatten | |
| khoá nóng | a hot key | |
| điều kiện cần | a necessary condition | *necessary* — "**NE**-sơ-sơ-ri", trọng âm đầu |
| điều kiện đủ | a sufficient condition | sơ-**FI**-shợnt, trọng âm âm thứ hai |
| thời gian chờ khoá | lock wait time | |
| thông lượng | throughput | âm **th** /θ/ đầu, "THRU-put" |

---

## Bài nói

> **Cách luyện:** bật ghi âm, nói **60–90 giây** cho mỗi đề, **không nhìn tài liệu** khi nói. Mỗi bài phải dùng ít nhất **sáu thuật ngữ** trong bảng trên. Đề số 6 là đề phản biện quan trọng nhất của cả giáo trình — hãy luyện tới khi bạn nói được cả ba lý do mà không phải dừng lại nghĩ.

1. **Explain to a junior engineer** why reading the stock, checking it is above zero, and then writing the decremented value causes overselling, and what you would write instead.

2. **Explain to a junior engineer** why a checkout flow takes the opposite consistency decision from a news feed, and what you are paying for that choice.

3. **Describe what happens when** a customer adds the last item to their cart, spends twelve minutes entering their card details, and then abandons the page. Walk through the reservation lifecycle.

4. **Describe what happens when** the network times out after the payment gateway has already taken the money, and explain how your design prevents a second charge.

5. **When would you choose** a conditional update over pessimistic locking, and when would you move the whole thing into Redis? State the trade-off in each direction.

6. **A colleague says:** *"A database transaction is enough to prevent overselling — we don't need reservations or idempotency keys."* **Explain what is wrong with that**, and give the three specific things a transaction cannot cover.

7. **Describe what happens when** one hundred thousand people press buy on one thousand units at the same second, following both a winner and a loser through your design.
