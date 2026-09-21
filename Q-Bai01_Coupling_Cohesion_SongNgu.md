# Bài 1 — Hai thước đo gốc: Coupling & Cohesion · Encapsulation & Abstraction
### Bản song ngữ: tiếng Anh bám cấu trúc tiếng Việt

> **Cách dùng bài này:** đọc đoạn tiếng Việt trước, tự dịch trong đầu, rồi mới nhìn sang đoạn tiếng Anh để so. Sau khi so xong, **đọc to đoạn tiếng Anh hai lần** — lần một để nhìn chữ, lần hai để nghe chính mình. Phần bài nói ở cuối là bắt buộc; không nói ra thì bài này chỉ là bài đọc.

---

## Phần 1 — Vì sao hai thước đo này là nền tảng của mọi thứ

**Tiếng Việt**

Bài này là bài nền tảng nhất của cả mục Software Design. Mọi nguyên tắc SOLID, mọi pattern của GoF, và mọi quyết định kiến trúc sau này đều chỉ là phương tiện để đạt một mục tiêu duy nhất, đó là coupling lỏng và cohesion cao. Chúng ta nên xem hai thước đo này như một chiếc la bàn, vì chúng cho phép chúng ta đánh giá bất kỳ thiết kế nào, kể cả code do AI sinh ra. Khi người phỏng vấn hỏi "tại sao anh thiết kế như vậy", một câu trả lời tốt luôn quay về hai thước đo này và không dừng lại ở tên của pattern. Nếu chúng ta chỉ nhớ tên pattern mà không nói được nó đẩy coupling xuống hay kéo cohesion lên như thế nào, người phỏng vấn sẽ kết luận rằng chúng ta học vẹt. Vì vậy bài này đứng trước SOLID, bởi vì SOLID chính là cách hệ thống hoá hai thước đo này thành năm quy tắc dễ nhớ.

**English (bám cấu trúc tiếng Việt)**

This lesson is the most foundational lesson in the whole Software Design section. Every SOLID principle, every GoF pattern, and every architectural decision later on is only a means to reach a single goal, which is loose coupling and high cohesion. We should treat these two measures as a compass, because they let us judge any design, including the code that AI generates. When an interviewer asks "why did you design it this way", a good answer always goes back to these two measures and does not stop at the name of the pattern. If we only remember the name of a pattern but cannot say how it pushes coupling down or pulls cohesion up, the interviewer will conclude that we learned it by rote. Therefore this lesson comes before SOLID, because SOLID is exactly the way of systematising these two measures into five rules that are easy to remember.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| bài nền tảng nhất của cả mục | the most foundational lesson in the whole section |
| đều chỉ là phương tiện để đạt | is only a means to reach |
| một mục tiêu duy nhất, đó là | a single goal, which is |
| kể cả code do AI sinh ra | including the code that AI generates |
| luôn quay về hai thước đo này | always goes back to these two measures |
| không dừng lại ở tên của pattern | does not stop at the name of the pattern |
| đẩy coupling xuống hay kéo cohesion lên | pushes coupling down or pulls cohesion up |
| chúng ta học vẹt | we learned it by rote |
| hệ thống hoá … thành năm quy tắc | systematising … into five rules |

**Thuật ngữ cần nhớ**

- độ ghép → **coupling**
- độ cố kết → **cohesion**
- thước đo → **measure** / **metric**
- la bàn → **compass**
- học vẹt → **learn by rote**

---

## Phần 2 — Coupling là gì và hậu quả khi coupling cao

**Tiếng Việt**

Coupling, hay độ ghép, là mức độ mà một module phụ thuộc vào module khác và biết về chi tiết bên trong của module đó. Chúng ta có thể hình dung coupling như những sợi dây điện chạy giữa các hộp thiết bị. Càng nhiều dây thì chúng ta càng khó nhấc một hộp ra khỏi hệ thống, và khi chúng ta đụng vào một hộp thì cả mớ dây bị giật theo. Trong code, "bị giật theo" nghĩa là chúng ta sửa một class rồi phải sửa lan sang năm class khác, và mỗi lần sửa là một cơ hội làm hỏng thứ đang chạy tốt. Nếu chúng ta để coupling cao trong thời gian dài, tốc độ giao hàng của cả đội sẽ chậm dần, vì mọi thay đổi nhỏ đều biến thành một cuộc điều tra lớn. Đó là lý do người phỏng vấn quan tâm đến coupling nhiều hơn là quan tâm đến việc chúng ta gọi tên được bao nhiêu pattern.

**English (bám cấu trúc tiếng Việt)**

Coupling is the degree to which one module depends on another module and knows about the internal details of that module. We can picture coupling as the electrical wires that run between boxes of equipment. The more wires there are, the harder it is for us to lift one box out of the system, and when we touch one box the whole bundle of wires gets pulled along. In code, "gets pulled along" means that we fix one class and then the fix spreads to five other classes, and every fix is a chance to break something that is running fine. If we let coupling stay high for a long time, the delivery speed of the whole team will slowly drop, because every small change turns into a big investigation. That is the reason interviewers care about coupling more than they care about how many patterns we can name.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| mức độ mà một module phụ thuộc vào | the degree to which one module depends on |
| biết về chi tiết bên trong của | knows about the internal details of |
| chúng ta có thể hình dung … như | we can picture … as |
| càng nhiều dây thì càng khó | the more wires there are, the harder it is |
| nhấc một hộp ra khỏi hệ thống | lift one box out of the system |
| cả mớ dây bị giật theo | the whole bundle of wires gets pulled along |
| phải sửa lan sang năm class khác | the fix spreads to five other classes |
| làm hỏng thứ đang chạy tốt | break something that is running fine |
| tốc độ giao hàng của cả đội sẽ chậm dần | the delivery speed of the whole team will slowly drop |
| biến thành một cuộc điều tra lớn | turns into a big investigation |

**Thuật ngữ cần nhớ**

- phụ thuộc vào → **depend on**
- chi tiết bên trong → **internal details**
- lan ra, lan sang → **spread to**
- tốc độ giao hàng → **delivery speed**
- gọi tên được → **can name**

---

## Phần 3 — Cohesion là gì và hậu quả khi cohesion thấp

**Tiếng Việt**

Cohesion, hay độ cố kết, là mức độ mà các phần bên trong một module thật sự thuộc về nhau và cùng phục vụ một mục đích. Ẩn dụ dễ nhớ nhất là một hộp dụng cụ: cohesion cao nghĩa là trong hộp chỉ có cờ-lê đúng bộ, còn cohesion thấp nghĩa là trong hộp lẫn cả cờ-lê, bánh mì và hoá đơn tiền điện. Cách kiểm tra nhanh là chúng ta hỏi xem các method trong một class có cùng phục vụ một trách nhiệm hay không. Nếu một class vừa tính giá, vừa gửi email, vừa ghi log ra file, thì class đó có cohesion thấp, bởi vì ba việc đó thay đổi vì ba lý do khác nhau. Hậu quả của cohesion thấp là không ai dám tái sử dụng class đó, vì lấy phần mình cần thì phải kéo theo cả phần mình không cần. Ý này chính là cầu nối sang nguyên tắc Single Responsibility ở Bài 4.

**English (bám cấu trúc tiếng Việt)**

Cohesion is the degree to which the parts inside one module truly belong together and serve one purpose. The metaphor that is easiest to remember is a toolbox: high cohesion means the box holds only wrenches from the right set, while low cohesion means the box holds a mix of wrenches, bread and an electricity bill. The quick way to check is that we ask whether the methods in a class all serve one responsibility. If a class calculates prices, sends emails, and writes logs to a file, then that class has low cohesion, because those three jobs change for three different reasons. The consequence of low cohesion is that nobody dares to reuse that class, because taking the part we need forces us to drag along the part we do not need. This idea is exactly the bridge to the Single Responsibility principle in Lesson 4.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| thật sự thuộc về nhau | truly belong together |
| ẩn dụ dễ nhớ nhất là | the metaphor that is easiest to remember is |
| cờ-lê đúng bộ | wrenches from the right set |
| trong hộp lẫn cả | the box holds a mix of |
| cách kiểm tra nhanh là chúng ta hỏi xem | the quick way to check is that we ask whether |
| cùng phục vụ một trách nhiệm | all serve one responsibility |
| thay đổi vì ba lý do khác nhau | change for three different reasons |
| không ai dám tái sử dụng | nobody dares to reuse |
| phải kéo theo cả phần mình không cần | forces us to drag along the part we do not need |
| chính là cầu nối sang | is exactly the bridge to |

**Thuật ngữ cần nhớ**

- thuộc về nhau → **belong together**
- hộp dụng cụ → **toolbox**
- trách nhiệm → **responsibility**
- tái sử dụng → **reuse**
- hậu quả → **consequence**
- cầu nối sang → **bridge to**

---

## Phần 4 — Coupling là một cái thang, không phải một công tắc

**Tiếng Việt**

Coupling không phải là một công tắc bật tắt, mà là một cái thang có nhiều bậc. Bậc chặt nhất là khi module của chúng ta phụ thuộc trực tiếp vào một kiểu cụ thể, ví dụ khi code nghiệp vụ gọi thẳng vào class `StripeClient`. Bậc lỏng hơn là khi chúng ta phụ thuộc vào một abstraction, ví dụ một interface tên là `PaymentGateway`, và class Stripe chỉ là một hiện thực của interface đó. Bậc lỏng nhất là khi hai bên không gọi nhau nữa mà nói chuyện qua message hoặc event, vì bên gửi thậm chí không cần biết ai đang nghe. Giảm coupling, vì vậy, chính là leo lên cái thang abstraction này từng bậc một. Nếu chúng ta bỏ qua cái thang và giữ phụ thuộc vào kiểu cụ thể, thì việc đổi nhà cung cấp thanh toán sẽ buộc chúng ta mở lại toàn bộ code nghiệp vụ.

**English (bám cấu trúc tiếng Việt)**

Coupling is not an on-off switch, but a ladder with many steps. The tightest step is when our module depends directly on a concrete type, for example when the business code calls straight into the `StripeClient` class. A looser step is when we depend on an abstraction, for example an interface named `PaymentGateway`, and the Stripe class is only one implementation of that interface. The loosest step is when the two sides no longer call each other but talk through a message or an event, because the sender does not even need to know who is listening. Reducing coupling, therefore, is exactly climbing up this abstraction ladder one step at a time. If we skip the ladder and keep depending on a concrete type, then changing the payment provider will force us to open up the whole business code again.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| một công tắc bật tắt | an on-off switch |
| một cái thang có nhiều bậc | a ladder with many steps |
| bậc chặt nhất là khi | the tightest step is when |
| phụ thuộc trực tiếp vào một kiểu cụ thể | depends directly on a concrete type |
| gọi thẳng vào | calls straight into |
| chỉ là một hiện thực của | is only one implementation of |
| bên gửi thậm chí không cần biết ai đang nghe | the sender does not even need to know who is listening |
| leo lên … từng bậc một | climbing up … one step at a time |
| bỏ qua cái thang | skip the ladder |
| buộc chúng ta mở lại toàn bộ | force us to open up the whole |

**Thuật ngữ cần nhớ**

- kiểu cụ thể → **concrete type**
- hiện thực (của interface) → **implementation**
- bậc, nấc thang → **step** / **rung**
- nhà cung cấp → **provider** / **vendor**
- thang abstraction → **abstraction ladder**

---

## Phần 5 — Hai thước đo không tự động đánh đổi nhau

**Tiếng Việt**

Nhiều người tưởng rằng coupling và cohesion là một cặp đánh đổi, nghĩa là kéo cái này lên thì cái kia phải xuống. Điều đó không đúng. Một thiết kế tốt kéo cohesion lên và đẩy coupling xuống cùng một lúc, bởi vì khi chúng ta gom đúng những thứ thuộc về nhau vào một chỗ thì số dây phải chạy ra ngoài tự nhiên giảm đi. Ngược lại, một thiết kế tệ thường có coupling cao và cohesion thấp cùng lúc, và hình dạng quen thuộc nhất của nó là God Object, tức là một class ôm mọi thứ và ai cũng phải gọi vào nó. Khi chúng ta gặp một class dài hai nghìn dòng mà mọi module đều import, chúng ta đang nhìn thấy cả hai thước đo cùng hỏng. Cách chữa không phải là cắt class đó làm đôi cho ngắn lại, mà là tách theo trục thay đổi, nghĩa là gom những phần luôn thay đổi cùng nhau vào cùng một module.

**English (bám cấu trúc tiếng Việt)**

Many people think that coupling and cohesion are a trade-off pair, which means that pulling one up must push the other down. That is not correct. A good design pulls cohesion up and pushes coupling down at the same time, because when we gather the things that belong together into one place, the number of wires that have to run outside naturally drops. On the other hand, a bad design usually has high coupling and low cohesion at the same time, and its most familiar shape is the God Object, that is, one class that holds everything and that everybody has to call into. When we meet a class of two thousand lines that every module imports, we are looking at both measures failing together. The cure is not to cut that class in half so that it becomes shorter, but to split it along the axis of change, which means gathering the parts that always change together into the same module.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| nhiều người tưởng rằng | many people think that |
| một cặp đánh đổi | a trade-off pair |
| kéo cái này lên thì cái kia phải xuống | pulling one up must push the other down |
| gom đúng những thứ thuộc về nhau vào một chỗ | gather the things that belong together into one place |
| số dây phải chạy ra ngoài tự nhiên giảm đi | the number of wires that have to run outside naturally drops |
| hình dạng quen thuộc nhất của nó | its most familiar shape |
| một class ôm mọi thứ | one class that holds everything |
| cả hai thước đo cùng hỏng | both measures failing together |
| cách chữa không phải là … mà là | the cure is not to … but to |
| tách theo trục thay đổi | split it along the axis of change |

**Thuật ngữ cần nhớ**

- đánh đổi → **trade-off**
- ngược lại → **on the other hand**
- cách chữa → **the cure** / **the fix**
- cắt làm đôi → **cut in half**
- trục thay đổi → **axis of change**

---

## Phần 6 — Encapsulation thật sự là bảo vệ invariant

**Tiếng Việt**

Encapsulation, hay đóng gói, là việc che giấu và bảo vệ trạng thái bên trong của một object, và chỉ cho phép thay đổi trạng thái đó qua các hành vi hợp lệ. Mục đích thật sự của nó là giữ invariant, tức là giữ những điều luôn phải đúng về object đó. Ở đây có một hiểu lầm rất phổ biến mà chúng ta phải gọi tên: để tất cả field là private rồi sinh một getter và một setter cho từng field không phải là encapsulation. Cách làm đó chỉ phơi state ra qua cửa sau, vì bên ngoài vẫn đặt được giá trị tuỳ ý, và chúng ta mất hoàn toàn khả năng bảo vệ bất biến. Phép thử tốt nhất là chúng ta tự hỏi liệu người khác có thể đặt object vào một trạng thái không hợp lệ từ bên ngoài hay không. Ví dụ, nếu câu lệnh gán số dư bằng âm một trăm chạy được thì encapsulation đã rò, nhưng nếu bên ngoài buộc phải gọi hành vi rút tiền và hành vi đó tự kiểm tra số dư thì encapsulation mới thật sự đứng vững.

**English (bám cấu trúc tiếng Việt)**

Encapsulation is the act of hiding and protecting the internal state of an object, and only allowing that state to be changed through valid behaviour. Its real purpose is to keep invariants, that is, to keep the things that must always be true about that object. Here there is a very common misunderstanding that we have to name: making all fields private and then generating one getter and one setter for each field is not encapsulation. That approach only exposes the state through the back door, because the outside can still set any value it likes, and we completely lose the ability to protect the invariant. The best test is that we ask ourselves whether other people can put the object into an invalid state from the outside. For example, if the statement that assigns the balance to minus one hundred runs successfully then encapsulation has leaked, but if the outside is forced to call the withdraw behaviour and that behaviour checks the balance itself then encapsulation really holds.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| che giấu và bảo vệ trạng thái bên trong | hiding and protecting the internal state |
| qua các hành vi hợp lệ | through valid behaviour |
| giữ những điều luôn phải đúng về | keep the things that must always be true about |
| một hiểu lầm rất phổ biến mà chúng ta phải gọi tên | a very common misunderstanding that we have to name |
| phơi state ra qua cửa sau | exposes the state through the back door |
| đặt được giá trị tuỳ ý | can set any value it likes |
| mất hoàn toàn khả năng bảo vệ | completely lose the ability to protect |
| phép thử tốt nhất là chúng ta tự hỏi liệu | the best test is that we ask ourselves whether |
| encapsulation đã rò | encapsulation has leaked |
| encapsulation mới thật sự đứng vững | encapsulation really holds |

**Thuật ngữ cần nhớ**

- đóng gói → **encapsulation**
- bất biến, điều luôn đúng → **invariant**
- trạng thái không hợp lệ → **invalid state**
- phơi ra → **expose**
- rò rỉ → **leak**
- rút tiền → **withdraw**

---

## Phần 7 — Abstraction phơi "cái gì", encapsulation giấu "như thế nào"

**Tiếng Việt**

Abstraction, hay trừu tượng hoá, là việc phơi ra "cái gì" và giấu đi "như thế nào". Khi chúng ta định nghĩa một interface, chúng ta đang nói cho người dùng biết họ gọi được những gì, nhưng chúng ta không nói cho họ biết bên trong chạy ra sao. Câu phân biệt ngắn nhất mà chúng ta nên thuộc lòng là câu này: abstraction quyết định cái gì lộ ra, còn encapsulation bảo vệ cái được giấu đi. Hai khái niệm này là hai mặt của cùng một đồng xu, nhưng chúng không đồng nhất, và người phỏng vấn hay dùng đúng điểm này để tách người hiểu sâu khỏi người chỉ học thuộc. Nếu chúng ta gộp hai khái niệm làm một, chúng ta sẽ thiết kế interface theo thói quen chứ không theo mục đích, và kết quả là interface lộ ra cả những chi tiết lẽ ra phải giấu. Một dấu hiệu điển hình là interface có tên rất trừu tượng nhưng lại chứa method mang tên của công nghệ bên dưới, ví dụ một method nhắc thẳng tên nhà cung cấp.

**English (bám cấu trúc tiếng Việt)**

Abstraction is the act of exposing "what" and hiding "how". When we define an interface, we are telling the user what they can call, but we are not telling them how things run inside. The shortest distinguishing sentence that we should learn by heart is this one: abstraction decides what is exposed, while encapsulation protects what is hidden. These two concepts are two sides of the same coin, but they are not identical, and interviewers often use exactly this point to separate people who understand deeply from people who only memorise. If we merge the two concepts into one, we will design interfaces out of habit and not out of purpose, and the result is an interface that exposes even the details which should have been hidden. A typical sign is an interface that has a very abstract name but contains a method named after the technology underneath, for example a method that mentions the provider's name directly.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| phơi ra "cái gì" và giấu đi "như thế nào" | exposing "what" and hiding "how" |
| họ gọi được những gì | what they can call |
| bên trong chạy ra sao | how things run inside |
| câu phân biệt ngắn nhất | the shortest distinguishing sentence |
| chúng ta nên thuộc lòng | we should learn by heart |
| hai mặt của cùng một đồng xu | two sides of the same coin |
| nhưng chúng không đồng nhất | but they are not identical |
| tách người hiểu sâu khỏi người chỉ học thuộc | separate people who understand deeply from people who only memorise |
| theo thói quen chứ không theo mục đích | out of habit and not out of purpose |
| những chi tiết lẽ ra phải giấu | the details which should have been hidden |
| mang tên của công nghệ bên dưới | named after the technology underneath |

**Thuật ngữ cần nhớ**

- trừu tượng hoá → **abstraction**
- lộ ra → **be exposed**
- đồng nhất → **identical**
- dấu hiệu điển hình → **a typical sign**
- bên dưới → **underneath**

---

## Phần 8 — Tách càng nhiều càng tốt? Và cách áp dụng vào một module thật

**Tiếng Việt**

Một câu hỏi bẫy hay gặp là câu này: nếu tách module giúp giảm coupling, vậy chúng ta tách càng nhiều càng tốt phải không? Câu trả lời là không. Coupling bằng không là điều không thể đạt được và cũng vô dụng, bởi vì các phần của hệ thống vẫn phải nói chuyện với nhau thì phần mềm mới chạy. Mục tiêu của chúng ta là coupling lỏng, chứ không phải coupling bằng không. Khi chúng ta tách quá tay, chúng ta chỉ chuyển coupling từ trong code ra ngoài mạng, và bây giờ mỗi lời gọi hàm đơn giản trở thành một lời gọi qua mạng có thể lỗi. Nguyên tắc an toàn là chúng ta tách theo trục thay đổi thật, nghĩa là hai phần luôn phải sửa cùng nhau thì nên nằm cùng nhau.

Bây giờ chúng ta hãy đặt tất cả những điều trên vào một ví dụ cụ thể là module thanh toán. Module này chỉ nên phụ thuộc vào abstraction `PaymentGateway`, chứ không phụ thuộc vào class cụ thể của một nhà cung cấp, nhờ đó việc đổi từ Stripe sang Adyen không đụng đến business logic. Bên trong module, chúng ta gom đúng những thứ liên quan đến thanh toán để cohesion cao, và chúng ta không nhét logic gửi email vào đó. Các công ty lớn làm đúng một việc này ở cấp tổ chức: họ chia code và service quanh các bounded context nghiệp vụ để cohesion cao, rồi cho các service nói chuyện với nhau qua API hoặc event để coupling lỏng. Khái niệm "two-pizza team" của Amazon thực ra là cohesion ở cấp tổ chức, vì một đội nhỏ sở hữu trọn một domain. Nếu chúng ta chia đội theo tầng kỹ thuật thay vì theo domain, mọi tính năng sẽ phải đi qua ba đội, và coupling giữa người với người còn đắt hơn coupling giữa các class.

**English (bám cấu trúc tiếng Việt)**

A common trap question is this one: if splitting modules helps to reduce coupling, then should we split as much as possible? The answer is no. Zero coupling is impossible to reach and is also useless, because the parts of the system still have to talk to each other for the software to run. Our goal is loose coupling, and not zero coupling. When we split too far, we only move the coupling from inside the code out onto the network, and now every simple function call becomes a network call that can fail. The safe principle is that we split along the real axis of change, which means that two parts that always have to be edited together should sit together.

Now let us put everything above into one concrete example, which is the payment module. This module should only depend on the `PaymentGateway` abstraction, and not depend on the concrete class of one provider, thanks to that switching from Stripe to Adyen does not touch the business logic. Inside the module, we gather exactly the things related to payment so that cohesion is high, and we do not stuff the email-sending logic in there. Big companies do exactly this one thing at the organisational level: they split code and services around business bounded contexts so that cohesion is high, and then let the services talk to each other through APIs or events so that coupling is loose. Amazon's "two-pizza team" idea is in fact cohesion at the organisational level, because one small team owns a whole domain. If we split teams by technical layer instead of by domain, every feature will have to go through three teams, and the coupling between people is even more expensive than the coupling between classes.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| một câu hỏi bẫy hay gặp | a common trap question |
| tách càng nhiều càng tốt phải không? | should we split as much as possible? |
| là điều không thể đạt được và cũng vô dụng | is impossible to reach and is also useless |
| thì phần mềm mới chạy | for the software to run |
| khi chúng ta tách quá tay | when we split too far |
| chuyển coupling từ trong code ra ngoài mạng | move the coupling from inside the code out onto the network |
| luôn phải sửa cùng nhau thì nên nằm cùng nhau | that always have to be edited together should sit together |
| nhờ đó … không đụng đến | thanks to that … does not touch |
| không nhét logic gửi email vào đó | do not stuff the email-sending logic in there |
| ở cấp tổ chức | at the organisational level |
| sở hữu trọn một domain | owns a whole domain |
| còn đắt hơn | is even more expensive than |

**Thuật ngữ cần nhớ**

- câu hỏi bẫy → **trap question**
- tách quá tay → **split too far**
- lời gọi qua mạng → **network call**
- tầng kỹ thuật → **technical layer**
- sở hữu → **own**
- đắt (về chi phí) → **expensive**

---

## Mô hình ghi nhớ

**Tiếng Việt**

Coupling là số dây nối chạy ra bên ngoài, còn cohesion là độ "đúng bộ" của những thứ nằm bên trong. Mọi pattern và mọi nguyên tắc sau này chỉ là cách rút bớt dây ra ngoài và gom đúng bộ ở bên trong.

**English (bám cấu trúc tiếng Việt)**

Coupling is the number of wires that run to the outside, while cohesion is how well the things sitting inside "match as a set". Every pattern and every principle later on is only a way to pull out fewer wires and to gather the right set inside.

---

## Bảng thuật ngữ tổng hợp

| Tiếng Việt | Tiếng Anh | Ghi chú phát âm |
|---|---|---|
| độ ghép | coupling | /ˈkʌplɪŋ/ — đọc là "CẤP-ling", không đọc theo mặt chữ thành "cau-pling" |
| độ cố kết | cohesion | /kəʊˈhiːʒən/ — trọng âm rơi vào âm giữa: co-HE-sion; đuôi đọc như "-zhần" |
| gắn kết (tính từ) | cohesive | /kəʊˈhiːsɪv/ — co-HE-sive, âm giữa dài |
| ghép lỏng | loose coupling | *loose* /luːs/ đọc đuôi **-s** rõ, không đọc thành /luːz/ |
| thước đo | measure / metric | |
| la bàn | compass | /ˈkʌmpəs/ — "CÂM-pợs", không đọc "com-pát" |
| học vẹt | learn by rote | *rote* /rəʊt/ đọc giống "route" kiểu Anh |
| phụ thuộc vào | depend on | |
| chi tiết bên trong | internal details | |
| lan sang | spread to | |
| tốc độ giao hàng | delivery speed | |
| thuộc về nhau | belong together | |
| hộp dụng cụ | toolbox | |
| cờ-lê | wrench | /rentʃ/ — chữ **w** câm, đọc là "ren-chờ" |
| trách nhiệm | responsibility | trọng âm ở âm thứ tư: res-pon-si-BIL-i-ty |
| tái sử dụng | reuse | |
| hậu quả | consequence | trọng âm đầu: CON-se-quence |
| kiểu cụ thể | concrete type | *concrete* /ˈkɒŋkriːt/ — trọng âm đầu: CON-crete |
| hiện thực (của interface) | implementation | trọng âm áp chót: im-ple-men-TA-tion |
| giao diện, hợp đồng gọi hàm | interface | trọng âm đầu: IN-ter-face |
| nhà cung cấp | provider / vendor | |
| thang abstraction | abstraction ladder | *abstraction* /æbˈstrækʃən/ — cụm phụ âm **-bstr-** khó, tập đọc chậm |
| đánh đổi | trade-off | |
| trục thay đổi | axis of change | *axis* /ˈæksɪs/ — đuôi **-ks-** phải bật rõ; số nhiều là *axes* /ˈæksiːz/ |
| cách chữa | the cure / the fix | |
| đóng gói | encapsulation | trọng âm áp chót: en-cap-su-LA-tion |
| bất biến | invariant | /ɪnˈveəriənt/ — trọng âm âm thứ hai: in-VA-ri-ant |
| trạng thái không hợp lệ | invalid state | *invalid* (tính từ) trọng âm âm hai: in-VAL-id |
| phơi ra | expose | |
| rò rỉ | leak | |
| rút tiền | withdraw | có âm **th**: with-DRAW, đặt lưỡi giữa hai hàm răng |
| trừu tượng hoá | abstraction | xem ở trên |
| đồng nhất | identical | trọng âm âm hai: i-DEN-ti-cal |
| bên dưới | underneath | có âm **th** cuối: un-der-NEATH |
| câu hỏi bẫy | trap question | |
| lời gọi qua mạng | network call | *network* — đuôi **-rk** phải bật, không nuốt |
| tầng kỹ thuật | technical layer | |
| ngữ cảnh nghiệp vụ được khoanh vùng | bounded context | *bounded* /ˈbaʊndɪd/ — hai âm tiết, đọc rõ đuôi **-ded** |
| lĩnh vực nghiệp vụ | domain | trọng âm âm hai: do-MAIN |
| ngưỡng, mức chịu | threshold | có âm **th** đầu: THRESH-old |

---

## Bài nói

> **Cách làm:** ghi âm lại giọng của mình, mỗi câu nói liên tục **60–90 giây**, **không nhìn tài liệu** khi nói. Mỗi bài nói phải dùng ít nhất **6 thuật ngữ** trong bảng ở trên. Nghe lại bản ghi âm một lần, và ghi ra hai chỗ mình bị vấp để nói lại lần hai.

1. Explain to a junior developer what coupling and cohesion are, using your own metaphor, and say why we want coupling to be loose and cohesion to be high.
2. A colleague says: "I made every field private and added a getter and a setter for each one, so this class is properly encapsulated." Explain what is wrong with that, and describe the test you would use instead.
3. Someone on your team wants to split one service into eight small services because they believe that less coupling is always better. Explain why you would push back, and what you would propose instead.
4. Describe what happens, step by step, when a payment module depends on a concrete `StripeClient` class and the company decides to move to Adyen.
5. Say one sentence that separates abstraction from encapsulation, then give a concrete example of an interface that leaks the "how" instead of exposing only the "what".
6. When would you choose event-based communication over a direct interface call, and when would you not? Give the trade-off in both directions.
7. You have just received a class generated by an AI tool. Describe out loud how you would judge its cohesion and its coupling in under one minute, and what single question you would ask about its invariants.
