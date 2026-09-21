# Bài 7 — Saga: orchestration vs choreography & compensation
### Bản song ngữ: tiếng Anh bám cấu trúc tiếng Việt

> **Cách dùng bài này:** đọc đoạn tiếng Việt trước → tự dịch trong đầu, không nhìn xuống → so sánh với đoạn tiếng Anh ngay bên dưới → **đọc to bản tiếng Anh hai lần**: lần một đọc chậm cho đúng từng âm, lần hai đọc liền mạch như đang nói với người phỏng vấn.

---

## ① Vì sao chúng ta cần Saga

**Tiếng Việt**

Trong kiến trúc microservices, mỗi service giữ cơ sở dữ liệu của riêng nó, nên chúng ta không hề có một transaction ACID chung cho cả một quy trình nghiệp vụ. Khi một luồng thanh toán đi qua ba service, chúng ta có ba transaction riêng biệt chứ không phải một. Chúng ta đã thấy ở bài trước rằng giao dịch hai pha giải được vấn đề trên lý thuyết nhưng khoá quá nhiều và chịu lỗi rất kém. Vì vậy ngành chọn một cách khác, đó là Saga, tức là một chuỗi các giao dịch cục bộ nối tiếp nhau. Mỗi bước commit độc lập ở service của chính nó, và nếu một bước thất bại thì chúng ta chạy các giao dịch bù trừ để hoàn tác những bước đã làm trước đó.

Chúng ta nên nói rõ Saga đánh đổi cái gì để lấy cái gì. Cái chúng ta bỏ đi là nhất quán mạnh tại mọi thời điểm, vì trong lúc saga đang chạy thì hệ thống đang ở một trạng thái nửa vời. Cái chúng ta nhận về là không có khoá toàn cục và không có một bộ điều phối duy nhất có thể làm kẹt tất cả. Vì đánh đổi này, Saga chỉ cho chúng ta nhất quán cuối cùng, và chúng ta phải thiết kế giao diện người dùng theo đúng thực tế đó. Nếu chúng ta quên điều này, hậu quả là người dùng nhìn thấy một trạng thái tạm thời rồi tưởng rằng hệ thống bị lỗi.

**English (bám cấu trúc tiếng Việt)**

In a microservices architecture, each service holds a database of its own, so we have no shared ACID transaction at all for one whole business process. When a checkout flow goes through three services, we have three separate transactions rather than one. We saw in the previous lesson that a two-phase commit solves the problem in theory but locks too much and tolerates failure very poorly. Therefore the industry chose a different way, which is the Saga, that is, a chain of local transactions following one another. Each step commits independently in its own service, and if one step fails then we run compensating transactions to undo the steps that were done before it.

We should state clearly what a Saga trades away and what it gets in return. What we give up is strong consistency at every moment, because while the saga is running the system sits in a half-finished state. What we get back is that there is no global lock and no single coordinator that can leave everyone stuck. Because of this trade-off, a Saga only gives us eventual consistency, and we have to design the user interface according to that reality. If we forget this, the consequence is that users see a temporary state and then assume that the system is broken.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| chúng ta không hề có một transaction ACID chung | we have no shared ACID transaction at all |
| cho cả một quy trình nghiệp vụ | for one whole business process |
| ba transaction riêng biệt chứ không phải một | three separate transactions rather than one |
| chịu lỗi rất kém | tolerates failure very poorly |
| một chuỗi các giao dịch cục bộ nối tiếp nhau | a chain of local transactions following one another |
| hoàn tác những bước đã làm trước đó | undo the steps that were done before it |
| đánh đổi cái gì để lấy cái gì | what it trades away and what it gets in return |
| hệ thống đang ở một trạng thái nửa vời | the system sits in a half-finished state |
| có thể làm kẹt tất cả | that can leave everyone stuck |
| theo đúng thực tế đó | according to that reality |
| rồi tưởng rằng hệ thống bị lỗi | and then assume that the system is broken |

**Thuật ngữ cần nhớ**

- giao dịch cục bộ → **a local transaction**
- giao dịch bù trừ → **a compensating transaction**
- quy trình nghiệp vụ → **a business process**
- nhất quán cuối cùng → **eventual consistency**
- một cơ sở dữ liệu cho mỗi service → **database per service**

---

## ② Orchestration: một bộ điều phối ra lệnh từng bước

**Tiếng Việt**

Kiểu điều phối thứ nhất gọi là orchestration, và trong kiểu này chúng ta có một bộ điều phối trung tâm. Bộ điều phối biết toàn bộ quy trình, nên nó gọi từng bước theo thứ tự và nó chờ kết quả của mỗi bước. Nó cũng lưu trạng thái của saga, nghĩa là nó biết chúng ta đang ở bước nào và bước nào đã hoàn tất. Khi một bước thất bại, chính bộ điều phối là bên quyết định gọi những hành động bù trừ nào và gọi theo thứ tự nào. Nhờ trạng thái tập trung như vậy, chúng ta nhìn thấy toàn bộ luồng ở một chỗ và chúng ta viết kiểm thử dễ hơn nhiều.

Cái giá của kiểu này là mỗi service phải biết tới bộ điều phối, nên mức gỡ phụ thuộc thấp hơn. Ngoài ra chính bộ điều phối trở thành một thành phần mà chúng ta phải làm cho bền, vì nếu nó chết giữa chừng thì saga phải chạy tiếp được sau khi nó sống lại. Vì lý do đó, ngày nay chúng ta hiếm khi tự viết bộ điều phối kèm bảng trạng thái và một tác vụ định giờ. Thay vào đó chúng ta dùng các công cụ thực thi bền như Temporal hoặc Camunda, vì chúng đã lo sẵn phần thử lại, phần hẹn giờ và phần hiển thị trạng thái. Đây là một điểm cập nhật đáng nói, vì xu hướng những năm gần đây nghiêng hẳn về quy trình được điều phối bằng công cụ.

**English (bám cấu trúc tiếng Việt)**

The first coordination style is called orchestration, and in this style we have one central orchestrator. The orchestrator knows the whole process, so it calls each step in order and it waits for the result of each step. It also stores the state of the saga, which means that it knows which step we are on and which steps have completed. When one step fails, the orchestrator itself is the side that decides which compensating actions to call and in which order to call them. Thanks to such centralised state, we see the whole flow in one place and we write tests far more easily.

The price of this style is that each service has to know about the orchestrator, so the level of decoupling is lower. Besides, the orchestrator itself becomes a component that we have to make durable, because if it dies halfway then the saga must be able to continue after it comes back to life. For that reason, nowadays we rarely write an orchestrator ourselves together with a state table and a scheduled task. Instead we use durable execution tools such as Temporal or Camunda, because they already take care of the retry part, the timer part and the state visibility part. This is an update worth mentioning, because the trend of recent years leans clearly towards processes that are orchestrated by a tool.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| kiểu điều phối thứ nhất | the first coordination style |
| nó gọi từng bước theo thứ tự | it calls each step in order |
| chúng ta đang ở bước nào | which step we are on |
| chính bộ điều phối là bên quyết định | the orchestrator itself is the side that decides |
| nhờ trạng thái tập trung như vậy | thanks to such centralised state |
| chúng ta viết kiểm thử dễ hơn nhiều | we write tests far more easily |
| mức gỡ phụ thuộc thấp hơn | the level of decoupling is lower |
| một thành phần mà chúng ta phải làm cho bền | a component that we have to make durable |
| chúng ta hiếm khi tự viết | we rarely write ... ourselves |
| một tác vụ định giờ | a scheduled task |
| chúng đã lo sẵn phần thử lại | they already take care of the retry part |
| xu hướng những năm gần đây nghiêng hẳn về | the trend of recent years leans clearly towards |

**Thuật ngữ cần nhớ**

- bộ điều phối trung tâm → **a central orchestrator**
- trạng thái của saga → **the saga state**
- thực thi bền vững → **durable execution**
- khả năng nhìn thấy trạng thái → **state visibility**
- tác vụ định giờ → **a scheduled task**

---

## ③ Choreography: mỗi service tự phản ứng theo event

**Tiếng Việt**

Kiểu điều phối thứ hai gọi là choreography, và trong kiểu này không hề có ai đứng ra chỉ huy. Mỗi service làm xong phần việc của mình thì phát ra một event, rồi các service khác nghe event đó và tự quyết định phải làm gì tiếp. Chuỗi công việc hình thành từ chính các phản ứng nối tiếp nhau, chứ không hình thành từ một kịch bản viết sẵn ở một nơi. Nhờ cách này, các service gỡ phụ thuộc với nhau ở mức cao nhất, vì không ai phải biết tên của ai. Đây cũng là cách tự nhiên nhất khi đội đã quen làm việc theo hướng sự kiện.

Nhược điểm lớn nhất của kiểu này là chúng ta khó nhìn thấy toàn bộ luồng. Không có một chỗ nào trong mã nguồn mô tả quy trình từ đầu tới cuối, nên muốn hiểu luồng thì chúng ta phải đọc rải rác qua nhiều kho mã. Khi có sự cố, chúng ta cũng khó trả lời câu hỏi đơn giản nhất, đó là đơn hàng này đang kẹt ở bước nào. Ngoài ra, một saga dài viết theo kiểu này rất dễ sinh ra các vòng lặp event mà không ai nhận ra. Vì vậy chúng ta chỉ nên chọn choreography khi số bước còn ít và khi luồng event vẫn còn dễ vẽ ra trên một trang giấy.

**English (bám cấu trúc tiếng Việt)**

The second coordination style is called choreography, and in this style there is nobody at all standing up to command. Each service finishes its own piece of work and then emits an event, and the other services listen to that event and decide for themselves what to do next. The chain of work forms out of the reactions following one another, rather than forming out of a script written in advance in one place. Thanks to this way, the services are decoupled from each other at the highest level, because nobody has to know anybody's name. This is also the most natural way when the team is already used to working in an event-driven direction.

The biggest disadvantage of this style is that we find it hard to see the whole flow. There is no single place in the source code that describes the process from beginning to end, so if we want to understand the flow then we have to read scattered across many repositories. When an incident happens, we also find it hard to answer the simplest question, which is which step this order is stuck at. Besides, a long saga written in this style very easily produces event loops that nobody notices. Therefore we should only choose choreography when the number of steps is still small and when the event flow is still easy to draw on one sheet of paper.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| không hề có ai đứng ra chỉ huy | there is nobody at all standing up to command |
| tự quyết định phải làm gì tiếp | decide for themselves what to do next |
| hình thành từ chính các phản ứng nối tiếp nhau | forms out of the reactions following one another |
| một kịch bản viết sẵn ở một nơi | a script written in advance in one place |
| không ai phải biết tên của ai | nobody has to know anybody's name |
| đội đã quen làm việc theo hướng sự kiện | the team is already used to working in an event-driven direction |
| chúng ta khó nhìn thấy toàn bộ luồng | we find it hard to see the whole flow |
| đọc rải rác qua nhiều kho mã | read scattered across many repositories |
| đơn hàng này đang kẹt ở bước nào | which step this order is stuck at |
| các vòng lặp event mà không ai nhận ra | event loops that nobody notices |
| dễ vẽ ra trên một trang giấy | easy to draw on one sheet of paper |

**Thuật ngữ cần nhớ**

- phối hợp theo event, không có nhạc trưởng → **choreography**
- phát ra một event → **to emit an event**
- theo hướng sự kiện → **event-driven**
- kho mã nguồn → **a repository**
- vòng lặp event → **an event loop**

---

## ④ Chọn giữa hai kiểu như thế nào

**Tiếng Việt**

Nhiều người trả lời câu hỏi này một cách máy móc, đó là saga đơn giản thì dùng choreography còn saga phức tạp thì dùng orchestration. Chúng ta nên trả lời sâu hơn thế, vì đây là một câu hỏi phân biệt trình độ. Tiêu chí thứ nhất là mức tải nhận thức mà đội phải gánh khi đọc hiểu luồng. Tiêu chí thứ hai là mức độ chúng ta cần nhìn thấy trạng thái khi vận hành, đặc biệt là lúc có sự cố lúc hai giờ sáng. Tiêu chí thứ ba là quy trình có nhiều nhánh điều kiện hay không, vì nhánh điều kiện viết bằng event rất khó theo dõi.

Từ ba tiêu chí đó, chúng ta rút ra một hướng dẫn thực dụng. Nếu saga chỉ có hai tới bốn bước, luồng đi thẳng, và đội đã quen với hướng sự kiện, choreography là lựa chọn gọn nhẹ. Nếu saga có năm bước trở lên, có nhánh điều kiện, hoặc bộ phận vận hành cần tra cứu trạng thái từng đơn, chúng ta chọn orchestration. Một điều nữa mà chúng ta nên nói ra là các hệ thống lớn thường dùng cả hai kiểu cùng lúc, chứ không chọn một kiểu cho toàn công ty. Câu chốt nên là chúng ta chọn theo mức độ cần nhìn thấy, chứ không chọn theo sở thích kiến trúc.

**English (bám cấu trúc tiếng Việt)**

Many people answer this question mechanically, which is that a simple saga uses choreography while a complex saga uses orchestration. We should answer more deeply than that, because this is a question that separates levels of experience. The first criterion is the cognitive load that the team has to carry when they read and understand the flow. The second criterion is how much we need to see the state during operations, especially at the moment of an incident at two in the morning. The third criterion is whether the process has many conditional branches or not, because conditional branches written with events are very hard to follow.

From those three criteria, we draw out a practical guideline. If the saga has only two to four steps, the flow runs straight, and the team is already used to the event-driven direction, choreography is the lighter choice. If the saga has five steps or more, has conditional branches, or the operations team needs to look up the state of each order, we choose orchestration. One more thing we should say out loud is that large systems often use both styles at the same time, rather than choosing one style for the whole company. The closing sentence should be that we choose according to how much visibility we need, rather than choosing according to an architectural preference.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| một cách máy móc | mechanically |
| một câu hỏi phân biệt trình độ | a question that separates levels of experience |
| mức tải nhận thức mà đội phải gánh | the cognitive load that the team has to carry |
| mức độ chúng ta cần nhìn thấy trạng thái | how much we need to see the state |
| nhiều nhánh điều kiện | many conditional branches |
| chúng ta rút ra một hướng dẫn thực dụng | we draw out a practical guideline |
| luồng đi thẳng | the flow runs straight |
| lựa chọn gọn nhẹ | the lighter choice |
| cần tra cứu trạng thái từng đơn | needs to look up the state of each order |
| chứ không chọn một kiểu cho toàn công ty | rather than choosing one style for the whole company |
| chọn theo sở thích kiến trúc | choosing according to an architectural preference |

**Thuật ngữ cần nhớ**

- tải nhận thức → **cognitive load**
- khả năng nhìn thấy khi vận hành → **operational visibility**
- nhánh điều kiện → **a conditional branch**
- tiêu chí lựa chọn → **a selection criterion**
- hướng dẫn thực dụng → **a practical guideline**

---

## ⑤ Bù trừ không phải là quay lui

**Tiếng Việt**

Phần khó nhất của saga nằm ở chỗ bù trừ, và đây cũng là chỗ mà nhiều người hiểu sai ngay từ đầu. Quay lui là hành động bên trong một cơ sở dữ liệu duy nhất, và sau khi quay lui thì mọi thứ đúng như chưa từng xảy ra. Bù trừ thì khác hẳn, vì nó là một hành động nghiệp vụ ngược diễn ra trong một thế giới đã thay đổi. Khi chúng ta đã trừ kho, đã tính tiền và đã gửi email, chúng ta không thể xoá những sự kiện đó khỏi lịch sử. Vì vậy hoàn tiền không có nghĩa là chưa từng tính tiền, bởi vì phí giao dịch, thời gian chờ và bản ghi lịch sử vẫn còn nguyên đó.

Từ nhận thức đó, chúng ta rút ra ba nguyên tắc khi thiết kế bù trừ. Nguyên tắc thứ nhất là mỗi bước phải có một hành động nghiệp vụ ngược tương ứng, ví dụ nhả chỗ đã giữ hoặc hoàn tiền, chứ không phải xoá bản ghi. Nguyên tắc thứ hai là chúng ta chạy các hành động bù trừ theo thứ tự ngược với thứ tự đã thực hiện. Nguyên tắc thứ ba là mỗi hành động bù trừ phải bất biến khi lặp, vì nó cũng chạy dưới ngữ nghĩa giao ít nhất một lần như mọi thứ khác. Nếu chúng ta viết bù trừ theo kiểu xoá bản ghi, hậu quả là chúng ta phá mất dấu vết kiểm toán và bộ phận tài chính sẽ không đối soát được.

**English (bám cấu trúc tiếng Việt)**

The hardest part of a saga lies in the compensation, and this is also the place where many people misunderstand from the very start. A rollback is an action inside one single database, and after the rollback everything is exactly as if it had never happened. Compensation is completely different, because it is a reverse business action taking place in a world that has already changed. When we have taken stock down, charged the money and sent the email, we cannot erase those events from history. Therefore a refund does not mean that we never charged, because the transaction fee, the waiting time and the history record are all still sitting there.

Out of that understanding, we draw three principles for designing compensation. The first principle is that each step must have a matching reverse business action, for example releasing a held seat or refunding the money, rather than deleting the record. The second principle is that we run the compensating actions in the reverse order of the order in which they were performed. The third principle is that each compensating action must be idempotent, because it also runs under at-least-once semantics like everything else. If we write compensation as deleting records, the consequence is that we destroy the audit trail and the finance team will not be able to reconcile.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| nhiều người hiểu sai ngay từ đầu | many people misunderstand from the very start |
| đúng như chưa từng xảy ra | exactly as if it had never happened |
| một hành động nghiệp vụ ngược | a reverse business action |
| trong một thế giới đã thay đổi | in a world that has already changed |
| xoá những sự kiện đó khỏi lịch sử | erase those events from history |
| vẫn còn nguyên đó | are all still sitting there |
| từ nhận thức đó | out of that understanding |
| nhả chỗ đã giữ | releasing a held seat |
| theo thứ tự ngược với thứ tự đã thực hiện | in the reverse order of the order in which they were performed |
| phá mất dấu vết kiểm toán | destroy the audit trail |
| sẽ không đối soát được | will not be able to reconcile |

**Thuật ngữ cần nhớ**

- hành động nghiệp vụ ngược → **a reverse business action**
- bù trừ theo nghĩa nghiệp vụ → **semantic compensation**
- hoàn tiền → **to refund**
- nhả phần đã giữ → **to release a reservation**
- dấu vết kiểm toán → **the audit trail**

---

## ⑥ Saga thiếu chữ I của ACID

**Tiếng Việt**

Một saga có tính nguyên tử ở mức nghiệp vụ và có nhất quán cuối cùng, nhưng nó không hề có tính cô lập. Lý do là giữa các bước, trạng thái trung gian lộ ra cho mọi giao dịch khác cùng nhìn thấy. Một người dùng khác có thể đọc một đơn hàng đang ở trạng thái chờ và tưởng rằng đơn đó đã chắc chắn. Tệ hơn nữa, một luồng khác có thể đọc chính dữ liệu mà lát nữa sẽ bị bù trừ, rồi ra quyết định dựa trên dữ liệu đó. Đây chính là lý do vì sao bán vượt kho là lỗi kinh điển của các saga viết vội.

Chúng ta không xoá được vấn đề này, nhưng chúng ta giảm nhẹ nó bằng bốn kỹ thuật. Kỹ thuật thứ nhất và cũng quan trọng nhất là khoá theo nghĩa nghiệp vụ, tức là chúng ta đặt bản ghi vào một trạng thái như đang giữ hoặc đang chờ, để các luồng khác biết mà tránh. Kỹ thuật thứ hai là gắn số phiên bản cho bản ghi, để một luồng ghi đè lên dữ liệu cũ sẽ bị phát hiện. Kỹ thuật thứ ba là dùng các phép cập nhật giao hoán, ví dụ trừ đi một lượng thay vì ghi đè một con số tuyệt đối. Kỹ thuật thứ tư là đọc lại và kiểm tra lại ngay trước khi ra quyết định cuối cùng, thay vì tin vào dữ liệu đã đọc từ vài giây trước.

**English (bám cấu trúc tiếng Việt)**

A saga has atomicity at the business level and has eventual consistency, but it has no isolation at all. The reason is that between the steps, the intermediate state is exposed for every other transaction to see. Another user may read an order that is in a pending state and assume that this order is already certain. Even worse, another flow may read exactly the data that will be compensated shortly afterwards, and then make a decision based on that data. This is exactly why overselling stock is the classic bug of hastily written sagas.

We cannot erase this problem, but we reduce it with four techniques. The first technique, and also the most important one, is the semantic lock, that is, we put the record into a state such as held or pending, so that other flows know to stay away. The second technique is to attach a version number to the record, so that a flow overwriting old data will be detected. The third technique is to use commutative updates, for example subtracting an amount instead of overwriting an absolute number. The fourth technique is to re-read and re-check right before we make the final decision, instead of trusting data that we read a few seconds earlier.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| nó không hề có tính cô lập | it has no isolation at all |
| trạng thái trung gian lộ ra | the intermediate state is exposed |
| cho mọi giao dịch khác cùng nhìn thấy | for every other transaction to see |
| tưởng rằng đơn đó đã chắc chắn | assume that this order is already certain |
| tệ hơn nữa | even worse |
| dữ liệu mà lát nữa sẽ bị bù trừ | the data that will be compensated shortly afterwards |
| bán vượt kho | overselling stock |
| các saga viết vội | hastily written sagas |
| để các luồng khác biết mà tránh | so that other flows know to stay away |
| một luồng ghi đè lên dữ liệu cũ sẽ bị phát hiện | a flow overwriting old data will be detected |
| các phép cập nhật giao hoán | commutative updates |
| ghi đè một con số tuyệt đối | overwriting an absolute number |

**Thuật ngữ cần nhớ**

- tính cô lập → **isolation**
- khoá theo nghĩa nghiệp vụ → **a semantic lock**
- trạng thái đang giữ → **a held state**
- phép cập nhật giao hoán → **a commutative update**
- bán vượt kho → **overselling**

---

## ⑦ Khi chính bước bù trừ cũng thất bại

**Tiếng Việt**

Một câu hỏi rất hay ở vòng phỏng vấn senior là điều gì xảy ra khi chính hành động bù trừ cũng thất bại. Chúng ta phải thừa nhận ngay rằng chuyện đó hoàn toàn có thể xảy ra, vì hành động bù trừ cũng chỉ là một lời gọi qua mạng như mọi lời gọi khác. Vì vậy mỗi hành động bù trừ phải thử lại được và phải bất biến khi lặp, đúng như các bước thuận. Ngoài ra chúng ta phải lưu trạng thái của saga một cách bền vững, để hệ thống biết nó đã bù trừ tới bước nào rồi. Nếu chúng ta giữ trạng thái đó chỉ trong bộ nhớ, một lần khởi động lại là chúng ta mất dấu và không ai biết saga đang dở ở đâu.

Nếu hành động bù trừ vẫn thất bại sau nhiều lần thử, chúng ta không được phép im lặng bỏ qua. Chúng ta đẩy công việc đó sang hàng đợi thư chết, phát cảnh báo, và chấp nhận rằng cần một người can thiệp bằng tay. Điều quan trọng nhất là hệ thống phải kết thúc ở một trạng thái được ghi nhận rõ ràng, ví dụ là đã thất bại và cần xử lý, chứ không phải một trạng thái nửa vời không tên. Một saga kẹt im lặng ở giữa chừng là tình huống tệ nhất, vì tiền của khách đã bị giữ mà không ai trong đội biết chuyện đó. Vì vậy chúng ta luôn thiết kế đường thoát cuối cùng cho con người, chứ không giả định rằng máy sẽ tự sửa được mọi thứ.

**English (bám cấu trúc tiếng Việt)**

A very good question in a senior interview round is what happens when the compensating action itself also fails. We have to admit straight away that this can completely happen, because a compensating action is also just a call over the network like every other call. Therefore each compensating action must be retryable and must be idempotent, exactly like the forward steps. Besides, we have to store the state of the saga durably, so that the system knows which step it has already compensated. If we keep that state only in memory, one restart is enough for us to lose track and for nobody to know where the saga is unfinished.

If the compensating action still fails after many attempts, we are not allowed to skip it silently. We push that job over to the dead-letter queue, raise an alert, and accept that a person needs to step in by hand. The most important thing is that the system must end in a state that is clearly recorded, for example failed and needing attention, rather than an unnamed half-finished state. A saga stuck silently in the middle is the worst situation, because the customer's money is being held and nobody on the team knows about it. Therefore we always design a final escape route for humans, rather than assuming that machines will fix everything by themselves.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| chúng ta phải thừa nhận ngay rằng | we have to admit straight away that |
| cũng chỉ là một lời gọi qua mạng | is also just a call over the network |
| đúng như các bước thuận | exactly like the forward steps |
| lưu trạng thái một cách bền vững | store the state durably |
| chúng ta mất dấu | we lose track |
| không ai biết saga đang dở ở đâu | nobody knows where the saga is unfinished |
| chúng ta không được phép im lặng bỏ qua | we are not allowed to skip it silently |
| cần một người can thiệp bằng tay | a person needs to step in by hand |
| một trạng thái nửa vời không tên | an unnamed half-finished state |
| tiền của khách đã bị giữ | the customer's money is being held |
| đường thoát cuối cùng cho con người | a final escape route for humans |
| máy sẽ tự sửa được mọi thứ | machines will fix everything by themselves |

**Thuật ngữ cần nhớ**

- có thể thử lại → **retryable**
- lưu bền vững → **to store durably**
- can thiệp bằng tay → **manual intervention**
- kẹt im lặng → **stuck silently**
- cần được chú ý xử lý → **needing attention**

---

## ⑧ Bốn nền tảng bắt buộc và ba chỗ chết người

**Tiếng Việt**

Dù chúng ta chọn orchestration hay choreography, có bốn nền tảng mà mọi saga đều phải có. Nền tảng thứ nhất là tính nguyên tử tại biên của mỗi bước, nghĩa là ghi cơ sở dữ liệu và phát event phải đi cùng nhau qua mẫu Outbox. Nền tảng thứ hai là consumer bất biến khi lặp, vì mọi thứ đều chạy dưới ngữ nghĩa giao ít nhất một lần. Nền tảng thứ ba là các hành động bù trừ được viết tường minh, kèm hàng đợi thư chết cho trường hợp xấu nhất. Nền tảng thứ tư là khả năng quan sát, gồm một mã tương quan đi xuyên toàn luồng và một cách tra cứu trạng thái saga.

Khi một đồng nghiệp đưa ra một saga thanh toán chạy tốt trên luồng thuận nhưng thỉnh thoảng tính tiền hai lần và bán vượt kho, chúng ta nên chỉ ra ba nguyên nhân gốc quen thuộc. Nguyên nhân thứ nhất là các bước không bất biến khi lặp, nên một lần giao lại là khách bị tính tiền lần nữa. Nguyên nhân thứ hai là thiếu khoá theo nghĩa nghiệp vụ, nên trạng thái chờ bị đọc như thể nó đã chắc chắn và kho bị bán vượt. Nguyên nhân thứ ba là event được phát thẳng trong cùng khối lệnh với transaction thay vì đi qua Outbox, nên thỉnh thoảng một bước bị mất hẳn. Ba lỗi này gần như luôn xuất hiện cùng nhau, vì chúng đều bắt nguồn từ việc chỉ kiểm thử luồng thuận.

**English (bám cấu trúc tiếng Việt)**

Whether we choose orchestration or choreography, there are four foundations that every saga must have. The first foundation is atomicity at the boundary of each step, which means that the database write and the event publish must go together through the Outbox pattern. The second foundation is an idempotent consumer, because everything runs under at-least-once semantics. The third foundation is compensating actions that are written out explicitly, together with a dead-letter queue for the worst case. The fourth foundation is observability, consisting of a correlation id that travels through the whole flow and a way to look up the saga state.

When a colleague presents a checkout saga that runs well on the happy path but occasionally charges twice and oversells stock, we should point out three familiar root causes. The first cause is that the steps are not idempotent, so one redelivery means the customer is charged again. The second cause is a missing semantic lock, so the pending state is read as if it were already certain and the stock is oversold. The third cause is that the event is published straight inside the same block as the transaction instead of going through the Outbox, so occasionally one step is lost entirely. These three bugs almost always appear together, because they all originate from testing only the happy path.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| dù chúng ta chọn A hay B | whether we choose A or B |
| tính nguyên tử tại biên của mỗi bước | atomicity at the boundary of each step |
| phải đi cùng nhau | must go together |
| được viết tường minh | written out explicitly |
| cho trường hợp xấu nhất | for the worst case |
| một mã tương quan đi xuyên toàn luồng | a correlation id that travels through the whole flow |
| chạy tốt trên luồng thuận | runs well on the happy path |
| ba nguyên nhân gốc quen thuộc | three familiar root causes |
| bị đọc như thể nó đã chắc chắn | is read as if it were already certain |
| trong cùng khối lệnh với transaction | inside the same block as the transaction |
| một bước bị mất hẳn | one step is lost entirely |
| chúng đều bắt nguồn từ | they all originate from |

**Thuật ngữ cần nhớ**

- luồng thuận, mọi thứ đều ổn → **the happy path**
- nguyên nhân gốc → **the root cause**
- mã tương quan → **a correlation id**
- khả năng quan sát → **observability**
- nền tảng bắt buộc → **a mandatory foundation**

---

## ⑨ Mô hình ghi nhớ

**Tiếng Việt**

Saga là nhiều giao dịch cục bộ nối với nhau bằng event, và khi có lỗi thì chúng ta chạy hành động nghiệp vụ ngược chứ không phải quay lui. Nền tảng bắt buộc gồm Outbox, tính bất biến khi lặp, bù trừ tường minh và khả năng quan sát; chúng ta dùng orchestration khi cần nhìn rõ, và dùng choreography khi cần gỡ phụ thuộc.

**English (bám cấu trúc tiếng Việt)**

A saga is many local transactions linked to each other by events, and when there is a failure we run a reverse business action rather than a rollback. The mandatory foundations are the Outbox, idempotency, explicit compensation and observability; we use orchestration when we need visibility, and we use choreography when we need decoupling.

---

## ⑩ Bảng thuật ngữ tổng hợp

| Tiếng Việt | Tiếng Anh | Ghi chú phát âm |
|---|---|---|
| chuỗi giao dịch nghiệp vụ | a saga | **SAH**-gə, trọng âm âm đầu |
| giao dịch cục bộ | a local transaction | *transaction* = tran-**ZAK**-shn, chữ *s* đọc /z/ |
| giao dịch bù trừ | a compensating transaction | **COM**-pen-say-ting, trọng âm âm đầu |
| quy trình nghiệp vụ | a business process | *business* — hai âm tiết "BIZ-nis" |
| một cơ sở dữ liệu cho mỗi service | database per service | |
| điều phối tập trung | orchestration | or-kes-**TRAY**-shn, chữ *ch* đọc /k/ |
| bộ điều phối | an orchestrator | **OR**-kes-tray-tə, trọng âm âm đầu |
| phối hợp không có nhạc trưởng | choreography | co-ri-**O**-grə-fi, chữ *ch* đọc /k/, trọng âm âm thứ ba |
| trạng thái của saga | the saga state | |
| thực thi bền vững | durable execution | *durable* = **DUR**-ə-bl, khác *durability* (du-rə-**BI**-li-ty) |
| tác vụ định giờ | a scheduled task | giọng Anh *schedule* /ˈʃedjuːl/ — "SHED-yul", không phải "SKED-jul" |
| phát ra một event | to emit an event | i-**MIT**, trọng âm âm sau |
| theo hướng sự kiện | event-driven | |
| kho mã nguồn | a repository | ri-**PO**-zi-tə-ri, trọng âm âm thứ hai |
| tải nhận thức | cognitive load | **COG**-ni-tiv, trọng âm âm đầu |
| khả năng nhìn thấy khi vận hành | operational visibility | vi-zi-**BI**-li-ty |
| nhánh điều kiện | a conditional branch | *branch* giọng Anh /brɑːntʃ/ — nguyên âm dài |
| tiêu chí lựa chọn | a selection criterion | *criterion* = crai-**TIƏ**-ri-ən; số nhiều là *criteria* |
| hành động nghiệp vụ ngược | a reverse business action | *reverse* = ri-**VERS**, trọng âm âm sau |
| bù trừ theo nghĩa nghiệp vụ | semantic compensation | si-**MAN**-tic, trọng âm âm giữa |
| quay lui | a rollback | |
| hoàn tiền | to refund | động từ = ri-**FUND**; danh từ = **RE**-fund |
| nhả phần đã giữ | to release a reservation | *reservation* = re-zə-**VAY**-shn |
| dấu vết kiểm toán | the audit trail | *audit* = **OR**-dit, giọng Anh /ˈɔːdɪt/ |
| đối soát | to reconcile | **RE**-con-cile, đuôi đọc /saɪl/ |
| tính cô lập | isolation | ai-sə-**LAY**-shn, âm đầu là /aɪ/ |
| khoá theo nghĩa nghiệp vụ | a semantic lock | |
| trạng thái đang giữ | a held state | |
| phép cập nhật giao hoán | a commutative update | cə-**MYU**-tə-tiv, trọng âm âm thứ hai |
| bán vượt kho | overselling | |
| số phiên bản của bản ghi | a version number | giọng Anh *version* /ˈvɜːʃn/ — "VER-shn" |
| có thể thử lại | retryable | ri-**TRAI**-ə-bl |
| can thiệp bằng tay | manual intervention | in-tə-**VEN**-shn |
| hàng đợi thư chết | the dead-letter queue | *queue* /kjuː/ — đọc y hệt chữ cái **Q** |
| bất biến khi lặp | idempotent | i-**DEM**-po-tent, trọng âm âm thứ hai |
| luồng thuận | the happy path | |
| nguyên nhân gốc | the root cause | |
| mã tương quan | a correlation id | co-rə-**LAY**-shn |
| khả năng quan sát | observability | ob-zə-və-**BI**-li-ty, sáu âm tiết |
| nền tảng bắt buộc | a mandatory foundation | **MAN**-də-tə-ri, trọng âm âm đầu |

---

## ⑪ Bài nói

> **Cách luyện:** bật ghi âm, nói **60–90 giây** cho mỗi đề, **không nhìn tài liệu**. Mỗi bài nói phải dùng ít nhất **sáu thuật ngữ** trong bảng trên. Nói xong thì nghe lại một lượt, đánh dấu chỗ mình ngập ngừng, rồi nói lại chính đề đó thêm một lần nữa.

1. Explain to a junior developer why a checkout flow across three services cannot use one ACID transaction, and describe what a saga does instead.

2. Walk through a checkout saga step by step: reserve inventory, charge payment, confirm order. Describe what happens when the payment step fails.

3. A colleague says compensation is just a rollback across services, so they plan to delete the payment record when the saga fails. Explain what is wrong with that.

4. Someone on your team wants choreography for a nine-step onboarding flow with several conditional branches, because it is more decoupled. Explain why you would push back.

5. Describe why a saga has no isolation, and describe how you would use a semantic lock to stop the inventory being oversold.

6. Describe what you would put in place for the case where the compensating action itself keeps failing, and explain why silence is the worst outcome.

7. When would you reach for a durable execution tool such as Temporal rather than writing your own orchestrator with a state table and a scheduled job?
