# Bài 1 — Vì sao Messaging: sync vs async & coupling
### Bản song ngữ: tiếng Anh bám cấu trúc tiếng Việt

> **Cách dùng bài này:** đọc đoạn tiếng Việt trước → tự dịch trong đầu, không nhìn xuống → so sánh với đoạn tiếng Anh ngay bên dưới → **đọc to bản tiếng Anh hai lần**: lần một đọc chậm cho đúng từng âm, lần hai đọc liền mạch như đang nói với người phỏng vấn.

---

## ① Hai kiểu giao tiếp: gọi điện và gửi thư

**Tiếng Việt**

Khi hai service nói chuyện với nhau, chúng ta chỉ có hai kiểu cơ bản, và mọi thứ còn lại đều là biến thể của hai kiểu đó. Kiểu thứ nhất là gọi đồng bộ, và nó giống hệt một cuộc gọi điện thoại. Hai bên phải cùng online tại một thời điểm, người gọi nói xong thì đứng chờ, và người gọi chỉ làm được việc tiếp theo khi đầu bên kia trả lời. Kiểu thứ hai là gọi bất đồng bộ qua một broker, và nó giống việc gửi thư qua bưu điện. Chúng ta bỏ thư vào hòm, rồi quay đi làm việc khác ngay lập tức, còn người nhận sẽ lấy thư khi họ rảnh. Nếu người nhận đang đi vắng, lá thư vẫn nằm yên trong hòm và không mất đi. Bưu điện ở đây chính là broker, và giá trị lớn nhất của nó là tách rời thời gian giữa lúc gửi và lúc nhận.

Sự phân biệt này nghe đơn giản, nhưng nếu chúng ta không nắm chắc nó thì chúng ta sẽ đi sai theo một trong hai hướng. Hướng sai thứ nhất là nhét queue vào mọi chỗ, và khi đó hệ thống có thêm hạ tầng, thêm độ trễ, và thêm chỗ cho lỗi ẩn nấp. Hướng sai thứ hai là sợ bất đồng bộ và cố giữ mọi thứ đồng bộ, và khi đó một service chậm sẽ kéo sập cả chuỗi service gọi nó. Vì vậy câu hỏi gốc nhất của bài này không phải là broker chạy thế nào, mà là chúng ta thêm broker để giải bài toán gì và trả giá gì.

**English (bám cấu trúc tiếng Việt)**

When two services talk to each other, we only have two basic styles, and everything else is a variant of those two styles. The first style is the synchronous call, and it is exactly like a phone call. Both sides must be online at the same moment, the caller finishes speaking and then waits, and the caller can only do the next thing when the other end answers. The second style is the asynchronous call through a broker, and it is like sending a letter through the post office. We drop the letter into the box, then we turn away and do other work immediately, while the receiver will pick the letter up when they are free. If the receiver is away, the letter still sits quietly in the box and is not lost. The post office here is exactly the broker, and its biggest value is that it separates time between the moment of sending and the moment of receiving.

This distinction sounds simple, but if we do not hold it firmly then we will go wrong in one of two directions. The first wrong direction is to push a queue into every place, and then the system has more infrastructure, more latency, and more places for bugs to hide. The second wrong direction is to be afraid of asynchronous work and to try to keep everything synchronous, and then one slow service will drag down the whole chain of services that call it. Therefore the most basic question of this lesson is not how a broker runs, but what problem we add a broker to solve and what price we pay.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| mọi thứ còn lại đều là biến thể của | everything else is a variant of |
| nó giống hệt một cuộc gọi điện thoại | it is exactly like a phone call |
| tại một thời điểm | at the same moment |
| người gọi chỉ làm được việc tiếp theo khi | the caller can only do the next thing when |
| bỏ thư vào hòm | drop the letter into the box |
| khi họ rảnh | when they are free |
| tách rời thời gian giữa lúc gửi và lúc nhận | separates time between the moment of sending and the moment of receiving |
| nếu chúng ta không nắm chắc nó | if we do not hold it firmly |
| nhét queue vào mọi chỗ | push a queue into every place |
| thêm chỗ cho lỗi ẩn nấp | more places for bugs to hide |
| kéo sập cả chuỗi service gọi nó | drag down the whole chain of services that call it |
| trả giá gì | what price we pay |

**Thuật ngữ cần nhớ**

- gọi đồng bộ → **synchronous call**
- gọi bất đồng bộ → **asynchronous call**
- độ trễ → **latency**
- hạ tầng → **infrastructure**
- lỗi lan theo chuỗi → **cascade failure**

---

## ② Năm lớp bài toán mà message queue giải

**Tiếng Việt**

Message queue giải năm lớp bài toán khác nhau, và chúng ta nên thuộc cả năm, vì người phỏng vấn hay đếm xem chúng ta kể được mấy cái. Lớp thứ nhất là gỡ phụ thuộc, nghĩa là producer không cần biết consumer là ai, nằm ở đâu và có bao nhiêu cái, producer chỉ cần biết tên của topic. Nhờ đó chúng ta thêm hoặc đổi consumer mà không phải đụng vào code của producer. Lớp thứ hai là bất đồng bộ để giảm độ trễ mà người dùng cảm nhận được, nghĩa là chúng ta trả response ngay sau khi nhận yêu cầu, còn phần việc nặng thì để worker làm sau. Ví dụ điển hình là gửi email xác nhận hoặc render video, vì người dùng không cần ngồi chờ hai việc đó. Lớp thứ ba là đệm tải, nghĩa là broker hấp thụ các cú tăng đột biến, và nếu mười nghìn request mỗi giây ập tới trong khi phía sau chỉ xử lý được một nghìn mỗi giây, queue sẽ giữ phần dư và phía sau rút ra theo nhịp của nó.

Lớp thứ tư là độ bền, nghĩa là message được ghi xuống đĩa, nên khi một consumer chết giữa chừng thì message không mất và sẽ được xử lý lại lúc consumer sống dậy. Lớp thứ năm là phát tán, nghĩa là một event đi tới nhiều consumer độc lập cùng một lúc, ví dụ một event đơn hàng đã tạo đi tới kho, tới email, tới analytics và tới audit. Điểm đáng giá của phát tán là chúng ta thêm consumer thứ năm mà không sửa một dòng nào bên phía producer. Nếu chúng ta chỉ nhớ mỗi lý do cho chạy nhanh hơn, hậu quả là chúng ta sẽ đặt queue sai chỗ và không giải thích được vì sao mình đặt nó ở đó. Tốc độ chỉ là một trong năm lý do, và nó thậm chí không phải lý do mạnh nhất.

**English (bám cấu trúc tiếng Việt)**

A message queue solves five different classes of problem, and we should know all five by heart, because interviewers often count how many of them we can list. The first class is decoupling, which means that the producer does not need to know who the consumers are, where they sit and how many there are, the producer only needs to know the name of the topic. Thanks to that, we add or change a consumer without having to touch the code of the producer. The second class is asynchronous work in order to reduce the latency that the user feels, which means that we return the response right after we receive the request, while the heavy work is left for a worker to do later. Typical examples are sending a confirmation email or rendering a video, because the user does not need to sit and wait for those two jobs. The third class is load levelling, which means that the broker absorbs the spikes, and if ten thousand requests per second arrive while the downstream side can only handle one thousand per second, the queue will hold the surplus and the downstream side will pull it out at its own pace.

The fourth class is durability, which means that the message is written down to disk, so when a consumer dies halfway the message is not lost and will be processed again when the consumer comes back to life. The fifth class is fan-out, which means that one event goes to many independent consumers at the same time, for example an order-created event goes to inventory, to email, to analytics and to audit. The valuable point of fan-out is that we add a fifth consumer without changing a single line on the producer side. If we only remember the one reason of making things run faster, the consequence is that we will put the queue in the wrong place and we will not be able to explain why we put it there. Speed is only one of the five reasons, and it is not even the strongest reason.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| chúng ta nên thuộc cả năm | we should know all five by heart |
| gỡ phụ thuộc | decoupling |
| nhờ đó | thanks to that |
| không phải đụng vào code của | without having to touch the code of |
| độ trễ mà người dùng cảm nhận được | the latency that the user feels |
| phần việc nặng thì để worker làm sau | the heavy work is left for a worker to do later |
| đệm tải | load levelling |
| hấp thụ các cú tăng đột biến | absorbs the spikes |
| giữ phần dư | hold the surplus |
| rút ra theo nhịp của nó | pull it out at its own pace |
| chết giữa chừng | dies halfway |
| sống dậy | comes back to life |
| không sửa một dòng nào | without changing a single line |
| đặt queue sai chỗ | put the queue in the wrong place |

**Thuật ngữ cần nhớ**

- gỡ phụ thuộc → **decoupling**
- đệm tải → **load levelling** / **load-levelling buffer**
- cú tăng đột biến → **spike**
- độ bền → **durability**
- phát tán một tới nhiều → **fan-out**
- phía sau / phía hạ nguồn → **the downstream side**

---

## ③ Coupling: cái được gỡ và cái mới sinh ra

**Tiếng Việt**

Bây giờ chúng ta nói về coupling, vì đây là chỗ người phỏng vấn hay đào sâu nhất trong chủ đề này. Coupling thời gian nghĩa là hai bên phải cùng online tại một thời điểm, và một lời gọi đồng bộ luôn mang loại coupling này. Bất đồng bộ gỡ được coupling thời gian, vì consumer có thể đang tắt mà vẫn nhận được message sau đó. Coupling không gian nghĩa là hai bên phải biết địa chỉ của nhau, ví dụ phải biết IP hoặc phải biết đúng instance nào đang sống. Bất đồng bộ chỉ gỡ được một phần coupling không gian, vì producer vẫn phải biết tên topic, nhưng nó không cần biết máy nào đang consume.

Tuy nhiên chúng ta phải nói thẳng phần còn lại, vì đây là điểm phân biệt một người senior với một người vừa đọc xong tài liệu. Bất đồng bộ tạo ra hai loại coupling mới. Loại thứ nhất là coupling vào schema của event, vì mọi consumer đều đọc theo đúng hình dạng dữ liệu mà producer phát ra. Loại thứ hai là coupling vào chính broker, vì cả hệ thống bây giờ dựa vào một mảnh hạ tầng chung và mảnh đó phải luôn sống. Không có bữa trưa miễn phí ở đây, chúng ta đổi coupling thời gian và coupling không gian để lấy coupling vào hợp đồng dữ liệu. Nếu chúng ta quên điều này, hậu quả là một ngày nào đó có người đổi một trường trong event và làm hỏng bốn consumer mà họ chưa từng gặp.

**English (bám cấu trúc tiếng Việt)**

Now we talk about coupling, because this is the place where interviewers dig the deepest in this topic. Temporal coupling means that both sides must be online at the same moment, and a synchronous call always carries this kind of coupling. Asynchronous work removes temporal coupling, because the consumer can be down and still receive the message afterwards. Spatial coupling means that both sides must know the address of each other, for example they must know the IP or they must know exactly which instance is alive. Asynchronous work only removes a part of spatial coupling, because the producer still has to know the topic name, but it does not need to know which machine is consuming.

However we must say the rest out loud, because this is the point that separates a senior person from a person who has just finished reading the documentation. Asynchronous work creates two new kinds of coupling. The first kind is coupling to the schema of the event, because every consumer reads according to the exact shape of the data that the producer emits. The second kind is coupling to the broker itself, because the whole system now leans on one shared piece of infrastructure and that piece must always be alive. There is no free lunch here, we trade temporal coupling and spatial coupling in exchange for coupling to the data contract. If we forget this, the consequence is that one day someone changes one field in the event and breaks four consumers that they have never met.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| chỗ người phỏng vấn hay đào sâu nhất | the place where interviewers dig the deepest |
| luôn mang loại coupling này | always carries this kind of coupling |
| có thể đang tắt mà vẫn nhận được | can be down and still receive |
| đúng instance nào đang sống | exactly which instance is alive |
| chúng ta phải nói thẳng phần còn lại | we must say the rest out loud |
| người vừa đọc xong tài liệu | a person who has just finished reading the documentation |
| đúng hình dạng dữ liệu mà producer phát ra | the exact shape of the data that the producer emits |
| dựa vào một mảnh hạ tầng chung | leans on one shared piece of infrastructure |
| không có bữa trưa miễn phí | there is no free lunch |
| đổi A để lấy B | trade A in exchange for B |
| làm hỏng bốn consumer mà họ chưa từng gặp | breaks four consumers that they have never met |

**Thuật ngữ cần nhớ**

- coupling thời gian → **temporal coupling**
- coupling không gian → **spatial coupling**
- hình dạng dữ liệu của event → **the event schema**
- hợp đồng dữ liệu → **the data contract**
- sự đánh đổi → **the trade-off**

---

## ④ Command và event

**Tiếng Việt**

Command và event nhìn qua thì giống nhau, vì cả hai đều là một message đi qua broker, nhưng ngữ nghĩa của chúng khác hẳn nhau. Một command mang nghĩa hãy làm việc X, tức là nó ra lệnh cho một bên khác. Một event mang nghĩa việc X đã xảy ra, tức là nó chỉ thông báo một sự thật. Vì vậy chúng ta đặt tên command ở dạng mệnh lệnh, ví dụ ChargePayment, và đặt tên event ở dạng quá khứ, ví dụ PaymentCharged. Một command có đúng một chủ sở hữu xử lý nó, còn một event thì bất kỳ ai quan tâm cũng nhận được.

Sự khác nhau này kéo theo hệ quả về coupling, và đây chính là phần người phỏng vấn muốn nghe. Với command, producer biết ai sẽ xử lý, nên coupling cao hơn nhưng chúng ta kiểm soát luồng tốt hơn. Với event, producer không quan tâm ai xử lý, nên mức gỡ phụ thuộc cao nhất nhưng không ai chịu trách nhiệm một cách rõ ràng. Hậu quả nếu chúng ta lạm dụng event là logic nghiệp vụ bị rải ra khắp nơi, và sáu tháng sau không ai vẽ lại được luồng đầy đủ. Chúng ta có một quy tắc chọn rất gọn: nếu chúng ta cần một bên cụ thể làm một việc cụ thể thì hãy dùng command, còn nếu chúng ta chỉ muốn kể lại một sự thật đã xảy ra thì hãy dùng event.

**English (bám cấu trúc tiếng Việt)**

A command and an event look the same at first glance, because both of them are a message going through the broker, but their meanings are completely different. A command carries the meaning of please do X, that is, it gives an order to another side. An event carries the meaning of X has happened, that is, it only announces a fact. Therefore we name a command in the imperative form, for example ChargePayment, and we name an event in the past form, for example PaymentCharged. A command has exactly one owner that handles it, while an event is received by anybody who cares.

This difference brings a consequence about coupling, and this is exactly the part that interviewers want to hear. With a command, the producer knows who will handle it, so the coupling is higher but we control the flow better. With an event, the producer does not care who handles it, so the level of decoupling is the highest but nobody takes responsibility in a clear way. The consequence if we overuse events is that the business logic gets scattered everywhere, and six months later nobody can draw the full flow again. We have a very short rule for choosing: if we need one specific side to do one specific job then we should use a command, and if we only want to report a fact that has happened then we should use an event.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| nhìn qua thì giống nhau | look the same at first glance |
| ngữ nghĩa của chúng khác hẳn nhau | their meanings are completely different |
| nó ra lệnh cho một bên khác | it gives an order to another side |
| nó chỉ thông báo một sự thật | it only announces a fact |
| đặt tên ở dạng mệnh lệnh | name it in the imperative form |
| đặt tên ở dạng quá khứ | name it in the past form |
| bất kỳ ai quan tâm cũng nhận được | it is received by anybody who cares |
| kéo theo hệ quả về | brings a consequence about |
| không ai chịu trách nhiệm một cách rõ ràng | nobody takes responsibility in a clear way |
| logic nghiệp vụ bị rải ra khắp nơi | the business logic gets scattered everywhere |
| không ai vẽ lại được luồng đầy đủ | nobody can draw the full flow again |

**Thuật ngữ cần nhớ**

- mệnh lệnh → **a command**
- sự kiện đã xảy ra → **an event**
- dạng mệnh lệnh → **the imperative form**
- chủ sở hữu duy nhất → **a single owner**
- lạm dụng → **to overuse**

---

## ⑤ Điểm-tới-điểm và xuất bản–đăng ký

**Tiếng Việt**

Sau khi phân biệt được command và event, chúng ta cần phân biệt thêm hai kiểu phân phối message. Kiểu điểm-tới-điểm nghĩa là mỗi message chỉ tới đúng một consumer, dù có bao nhiêu consumer đang cùng lắng nghe. Đây là mô hình hàng đợi công việc, ví dụ mười worker cùng rút job render video ra làm, và mỗi job chỉ được làm đúng một lần. Kiểu xuất bản và đăng ký nghĩa là mỗi message tới mọi thuê bao đang đăng ký, tức là nó phát tán rộng. Ví dụ một event OrderPlaced đi tới bốn service khác nhau, và mỗi service làm phần việc riêng của nó.

Cách nhận diện nhu cầu rất gọn, và chúng ta nên trả lời được trong một câu khi bị hỏi. Nếu bài toán là chia việc cho nhiều máy cùng làm, chúng ta cần điểm-tới-điểm. Nếu bài toán là báo cho nhiều bên cùng biết, chúng ta cần xuất bản và đăng ký. Nếu chúng ta chọn nhầm và dùng xuất bản–đăng ký cho một việc chỉ nên làm một lần, hậu quả là bốn consumer cùng trừ kho cho một đơn hàng. Ngược lại, nếu chúng ta dùng điểm-tới-điểm cho một sự kiện mà nhiều bên cần biết, các bên còn lại sẽ không bao giờ nghe thấy gì.

**English (bám cấu trúc tiếng Việt)**

After we can tell a command and an event apart, we need to tell apart two more styles of message delivery. The point-to-point style means that each message goes to exactly one consumer, no matter how many consumers are listening at the same time. This is the work queue model, for example ten workers pull video rendering jobs out to do, and each job is done exactly once. The publish and subscribe style means that each message goes to every subscriber that is subscribed, that is, it broadcasts widely. For example one OrderPlaced event goes to four different services, and each service does its own piece of work.

The way to recognise the need is very short, and we should be able to answer it in one sentence when we are asked. If the problem is to split work across many machines, we need point-to-point. If the problem is to tell many sides at the same time, we need publish and subscribe. If we choose wrongly and use publish and subscribe for a job that should only be done once, the consequence is that four consumers all take stock down for one order. On the other hand, if we use point-to-point for an event that many sides need to know, the remaining sides will never hear anything.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| phân biệt được A và B | tell A and B apart |
| dù có bao nhiêu consumer đang cùng lắng nghe | no matter how many consumers are listening at the same time |
| mô hình hàng đợi công việc | the work queue model |
| rút job ra làm | pull jobs out to do |
| mỗi job chỉ được làm đúng một lần | each job is done exactly once |
| thuê bao đang đăng ký | a subscriber that is subscribed |
| cách nhận diện nhu cầu | the way to recognise the need |
| chia việc cho nhiều máy cùng làm | split work across many machines |
| nếu chúng ta chọn nhầm | if we choose wrongly |
| cùng trừ kho cho một đơn hàng | all take stock down for one order |
| các bên còn lại sẽ không bao giờ nghe thấy gì | the remaining sides will never hear anything |

**Thuật ngữ cần nhớ**

- điểm-tới-điểm → **point-to-point**
- xuất bản và đăng ký → **publish and subscribe** (**pub/sub**)
- hàng đợi công việc → **work queue**
- thuê bao → **a subscriber**
- phát tán rộng → **to broadcast**

---

## ⑥ Khi nào chúng ta KHÔNG nên dùng queue

**Tiếng Việt**

Có một anti-pattern rất phổ biến, đó là cho mọi lời gọi nội bộ đi qua queue chỉ vì nghe có vẻ scalable. Chúng ta nên nói thẳng rằng bất đồng bộ chỉ đáng giá khi có ít nhất một lý do nằm trong năm lớp bài toán ở trên. Khi người dùng cần đọc lại ngay thứ mình vừa ghi, ví dụ đặt hàng xong là muốn xem đơn hàng ngay, việc nhét queue vào chỉ làm mọi thứ tệ đi. Chúng ta thêm độ trễ, thêm một mảnh hạ tầng phải vận hành, thêm nhất quán cuối cùng, và thêm rất nhiều công sức gỡ lỗi. Vì vậy mặc định phải là đồng bộ, còn bất đồng bộ là một quyết định có chủ đích và chúng ta phải nêu được lý do của nó.

Khi sếp hoặc một đồng nghiệp nói rằng cứ đẩy tất cả qua Kafka cho scalable, chúng ta nên phản biện theo ba bước. Bước một, chúng ta hỏi lại lời gọi này có cần phản hồi ngay hay không, vì nếu có thì bất đồng bộ sẽ làm hỏng trải nghiệm người dùng. Bước hai, chúng ta chỉ ra cái giá phải trả, gồm độ trễ, hạ tầng, nhất quán cuối cùng và chi phí gỡ lỗi. Bước ba, chúng ta đề nghị chỉ chuyển sang bất đồng bộ những chỗ có lý do rõ ràng, ví dụ chỗ cần đệm tải hoặc chỗ cần phát tán. Từ scalable tự nó không phải một lý do, vì nó không nói cho chúng ta biết cái gì đang là nút thắt.

**English (bám cấu trúc tiếng Việt)**

There is a very common anti-pattern, which is to send every internal call through a queue just because it sounds scalable. We should say plainly that asynchronous work is only worth it when there is at least one reason among the five classes of problem above. When the user needs to read back immediately what they have just written, for example they place an order and want to see that order right away, pushing a queue in only makes everything worse. We add latency, we add one more piece of infrastructure to operate, we add eventual consistency, and we add a lot of debugging effort. Therefore the default must be synchronous, while asynchronous is a deliberate decision and we have to state the reason for it.

When our boss or a colleague says that we should push everything through Kafka to be scalable, we should push back in three steps. Step one, we ask back whether this call needs an immediate response or not, because if it does then asynchronous work will damage the user experience. Step two, we point out the price we pay, which includes latency, infrastructure, eventual consistency and debugging cost. Step three, we propose to move to asynchronous only the places that have a clear reason, for example the place that needs load levelling or the place that needs fan-out. The word scalable by itself is not a reason, because it does not tell us what the bottleneck is.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| chỉ vì nghe có vẻ scalable | just because it sounds scalable |
| chúng ta nên nói thẳng rằng | we should say plainly that |
| chỉ đáng giá khi | is only worth it when |
| đọc lại ngay thứ mình vừa ghi | read back immediately what they have just written |
| việc nhét queue vào chỉ làm mọi thứ tệ đi | pushing a queue in only makes everything worse |
| một quyết định có chủ đích | a deliberate decision |
| chúng ta nên phản biện theo ba bước | we should push back in three steps |
| chúng ta hỏi lại | we ask back |
| làm hỏng trải nghiệm người dùng | damage the user experience |
| chỉ ra cái giá phải trả | point out the price we pay |
| tự nó không phải một lý do | by itself is not a reason |
| cái gì đang là nút thắt | what the bottleneck is |

**Thuật ngữ cần nhớ**

- đọc lại ngay sau khi ghi → **read-after-write**
- quyết định có chủ đích → **a deliberate decision**
- phản biện lại → **to push back**
- nút thắt → **the bottleneck**
- công sức gỡ lỗi → **debugging effort**

---

## ⑦ Bất đồng bộ làm đổi mô hình nhất quán

**Tiếng Việt**

Điều cuối cùng và cũng dễ quên nhất là bất đồng bộ làm đổi mô hình nhất quán của hệ thống. Khi chúng ta biến bước gửi email thành bất đồng bộ, phần đó chuyển từ nhất quán mạnh sang nhất quán cuối cùng. Người dùng nhìn thấy dòng chữ đặt hàng thành công trước khi email thật sự được gửi đi. Vì vậy chúng ta phải thiết kế trạng thái trung gian trong dữ liệu, ví dụ PENDING rồi mới tới CONFIRMED. Chúng ta cũng phải có cơ chế thử lại và một cách báo lỗi muộn cho người dùng, vì lỗi bây giờ xảy ra sau khi response đã trả về. Sai lầm chết người ở đây là coi publish xong là xong, trong khi publish chỉ có nghĩa là chúng ta đã ghi nhận yêu cầu.

Còn một cái bẫy nữa nằm ngay trong đoạn code mà ai cũng viết ở lần đầu. Chúng ta ghi đơn hàng xuống cơ sở dữ liệu, rồi ngay dòng sau chúng ta publish event lên broker. Đoạn code này chạy được, nhưng nó sai về kiến trúc, vì hai lần ghi đó không nằm trong cùng một transaction. Nếu tiến trình chết ở giữa hai dòng đó, đơn hàng đã nằm trong cơ sở dữ liệu nhưng event thì không bao giờ được phát ra. Đây gọi là bài toán ghi kép, và chúng ta sẽ vá nó bằng Outbox ở Bài 6. Bây giờ chúng ta chỉ cần nhớ một điều: code chạy được không có nghĩa là kiến trúc đúng.

**English (bám cấu trúc tiếng Việt)**

The last thing, and also the thing that is easiest to forget, is that asynchronous work changes the consistency model of the system. When we turn the email-sending step into an asynchronous one, that part moves from strong consistency to eventual consistency. The user sees the line order placed successfully before the email is really sent out. Therefore we must design an intermediate state in the data, for example PENDING first and only then CONFIRMED. We must also have a retry mechanism and a way to report a late failure to the user, because the failure now happens after the response has been returned. The deadly mistake here is to treat publish as done, while publish only means that we have recorded the request.

There is one more trap sitting right inside the piece of code that everybody writes on the first attempt. We write the order down into the database, and then on the very next line we publish an event to the broker. This piece of code runs, but it is wrong architecturally, because those two writes do not sit inside the same transaction. If the process dies between those two lines, the order is already in the database but the event is never emitted. This is called the dual-write problem, and we will patch it with the Outbox pattern in Lesson 6. For now we only need to remember one thing: code that runs does not mean the architecture is right.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| điều dễ quên nhất | the thing that is easiest to forget |
| biến bước gửi email thành bất đồng bộ | turn the email-sending step into an asynchronous one |
| chuyển từ nhất quán mạnh sang nhất quán cuối cùng | moves from strong consistency to eventual consistency |
| trạng thái trung gian | an intermediate state |
| cơ chế thử lại | a retry mechanism |
| cách báo lỗi muộn | a way to report a late failure |
| sai lầm chết người | the deadly mistake |
| coi publish xong là xong | to treat publish as done |
| chúng ta đã ghi nhận yêu cầu | we have recorded the request |
| một cái bẫy nằm ngay trong đoạn code | a trap sitting right inside the piece of code |
| nó sai về kiến trúc | it is wrong architecturally |
| bài toán ghi kép | the dual-write problem |
| vá nó bằng | patch it with |

**Thuật ngữ cần nhớ**

- mô hình nhất quán → **the consistency model**
- nhất quán cuối cùng → **eventual consistency**
- trạng thái trung gian → **an intermediate state**
- thử lại → **retry**
- ghi kép → **dual write**
- phát ra một event → **to emit an event**

---

## ⑧ Mô hình ghi nhớ

**Tiếng Việt**

Đồng bộ là gọi điện, cả hai bên phải cùng online; bất đồng bộ là gửi thư, broker giữ hộ chúng ta. Queue không phải để chạy nhanh hơn, mà để gỡ phụ thuộc, đệm tải, giữ bền và phát tán; và mỗi lần chúng ta chuyển sang bất đồng bộ, chúng ta đổi nhất quán mạnh lấy nhất quán cuối cùng, nên phải luôn có một trạng thái PENDING.

**English (bám cấu trúc tiếng Việt)**

Synchronous is a phone call, both sides must be online at once; asynchronous is a letter, the broker holds it for us. A queue is not there to run faster, but to decouple, to level the load, to keep things durable and to fan out; and every time we move to asynchronous, we trade strong consistency for eventual consistency, so we must always have a PENDING state.

---

## ⑨ Bảng thuật ngữ tổng hợp

| Tiếng Việt | Tiếng Anh | Ghi chú phát âm |
|---|---|---|
| gọi đồng bộ | synchronous | /ˈsɪŋkrənəs/ — "SING-cro-nợs", trọng âm đầu, **không** đọc "sin-CRÔ-nớt" |
| gọi bất đồng bộ | asynchronous | /eɪˈsɪŋkrənəs/ — "ây-SING-cro-nợs", trọng âm rơi vào âm thứ hai |
| hàng đợi | queue | /kjuː/ — đọc y hệt chữ cái **Q**, bốn chữ cái cuối câm hoàn toàn |
| bộ trung chuyển message | broker | |
| bên phát | producer | /prəˈdjuːsə/ — giọng Anh có âm /dj/: "prơ-DYU-sơ" |
| bên nhận | consumer | /kənˈsjuːmə/ — giọng Anh "kơn-SYU-mơ", không phải "kơn-SU-mơ" |
| gỡ phụ thuộc | decoupling | /diːˈkʌplɪŋ/ — "đi-CẤP-linh", âm giữa là /ʌ/ chứ không phải "cao" |
| coupling thời gian | temporal coupling | **TEM**-po-ral, trọng âm âm đầu |
| coupling không gian | spatial coupling | /ˈspeɪʃəl/ — "SPÂY-shợl", chữ *ti* đọc thành /ʃ/ |
| độ trễ | latency | /ˈleɪtənsi/ — "LÂY-tơn-si", âm đầu là /eɪ/ không phải /æ/ |
| thông lượng | throughput | âm *th* /θ/ + *-ough* đọc /uː/: "THRU-put" |
| độ bền | durability | du-ra-**BI**-li-ty — trọng âm nhảy sang âm thứ ba, khác với *durable* (**DUR**-a-ble) |
| phát tán một tới nhiều | fan-out | |
| cú tăng đột biến | spike | |
| đệm tải | load levelling | *levelling* viết hai chữ **l** theo chuẩn Anh |
| hạ tầng | infrastructure | **IN**-fra-struc-ture, trọng âm âm đầu |
| lược đồ dữ liệu | schema | /ˈskiːmə/ — "SKI-mơ", **không** đọc "sê-ma" hay "shê-ma" |
| hợp đồng dữ liệu | data contract | *contract* (danh từ) = **CON**-tract; động từ mới là con-**TRACT** |
| mệnh lệnh | command | com-**MAND**, giọng Anh /kəˈmɑːnd/ nguyên âm dài |
| dạng mệnh lệnh | imperative | im-**PE**-ra-tive, trọng âm âm thứ hai |
| sự kiện | event | |
| điểm-tới-điểm | point-to-point | |
| xuất bản và đăng ký | publish and subscribe (pub/sub) | *subscribe* /səbˈskraɪb/ — cụm *-scr-* phải bật rõ, không nuốt |
| thuê bao | subscriber | sub-**SCRI**-ber, trọng âm âm thứ hai |
| phát tán rộng | broadcast | giọng Anh /ˈbrɔːdkɑːst/ — "BROAD-cast" đuôi /-st/ phải bật |
| nhất quán cuối cùng | eventual consistency | *eventual* = i-**VEN**-chu-al, nghĩa là "rốt cuộc", không phải "có thể xảy ra" |
| trạng thái trung gian | intermediate state | in-ter-**ME**-di-ate; đuôi *-ate* của tính từ đọc nhẹ /ɪt/ |
| thử lại | retry | |
| ghi kép | dual write | *dual* /ˈdjuːəl/ — giọng Anh "DYU-ợl" |
| phát ra một event | emit an event | i-**MIT**, trọng âm âm thứ hai |
| nút thắt | bottleneck | |
| phản biện lại | push back | |
| đọc lại ngay sau khi ghi | read-after-write | |
| lỗi lan theo chuỗi | cascade failure | *cascade* = cas-**CADE**, trọng âm âm sau |
| sự đánh đổi | trade-off | |
| mẫu sai nên tránh | anti-pattern | giọng Anh *anti* đọc /ˈænti/ — "AN-ti", không phải "AN-tai" như giọng Mỹ |

---

## ⑩ Bài nói

> **Cách luyện:** bật ghi âm, nói **60–90 giây** cho mỗi đề, **không nhìn tài liệu**. Mỗi bài nói phải dùng ít nhất **sáu thuật ngữ** trong bảng trên. Nói xong thì nghe lại một lượt và đánh dấu những chỗ mình ngập ngừng, rồi nói lại chính đề đó một lần nữa.

1. Explain to a junior developer, using the phone call and the letter analogy, what actually changes when we move a step from a synchronous call to an asynchronous one through a broker.

2. A colleague says that the whole point of a message queue is to make the system faster. Explain what is wrong with that statement, and list the five classes of problem a queue really solves.

3. Your team lead wants to route every internal service call through Kafka so that the platform is "scalable". Explain why you would push back, and describe which calls you would keep synchronous.

4. Describe what happens to the consistency model when we make the confirmation email asynchronous, and describe what the user sees on the screen at each moment.

5. A senior colleague argues that asynchronous messaging removes coupling between services. Explain where you agree, and explain which two new kinds of coupling the broker introduces in exchange.

6. When would you choose a command over an event, and when would you choose an event over a command? Give one concrete example of each from an e-commerce checkout flow.

7. Describe the dual-write problem in your own words: walk through the two lines of code, the moment the process dies, and the state the system is left in afterwards.
