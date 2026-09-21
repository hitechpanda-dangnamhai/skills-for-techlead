# Bài 6 — SOLID (3): Dependency Inversion & DI · SOLID trong thực chiến
### Bản song ngữ: tiếng Anh bám cấu trúc tiếng Việt

> **Cách dùng bài này:** đọc đoạn tiếng Việt trước, tự dịch trong đầu, rồi mới nhìn sang đoạn tiếng Anh để so. Sau khi so xong, **đọc to đoạn tiếng Anh hai lần** — lần một để nhìn chữ, lần hai để nghe chính mình. Phần bài nói ở cuối là bắt buộc; không nói ra thì bài này chỉ là bài đọc.

---

## Phần 1 — DIP là chữ đắt giá nhất, và bài này đóng phần SOLID bằng đánh đổi

**Tiếng Việt**

DIP là chữ đắt giá nhất trong năm chữ SOLID, bởi vì nó là nền của kiến trúc Clean và Hexagonal mà chúng ta sẽ học ở Bài 11. Nó cũng là lý do tồn tại của DI container trong các framework như NestJS hay Spring. Bài này còn đóng lại phần SOLID bằng một câu chuyện về đánh đổi, đó là SOLID cũng có thể trở thành over-engineering. Chúng ta sẽ học cách phản hồi một pull request áp SOLID một cách máy móc, và đây là kỹ năng nối thẳng sang phần judgment ở Bài 15. Với một vị trí senior, cách chúng ta nói về giới hạn của một nguyên tắc thường quan trọng hơn cách chúng ta đọc thuộc nguyên tắc đó.

**English (bám cấu trúc tiếng Việt)**

DIP is the most valuable letter among the five SOLID letters, because it is the ground of the Clean and Hexagonal architectures that we will study in Lesson 11. It is also the reason why the DI container exists in frameworks such as NestJS or Spring. This lesson also closes the SOLID part with a story about trade-offs, which is that SOLID can also turn into over-engineering. We will learn how to respond to a pull request that applies SOLID mechanically, and this is a skill that connects straight to the judgment part in Lesson 15. For a senior position, the way we talk about the limits of a principle is usually more important than the way we recite that principle.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| chữ đắt giá nhất trong năm chữ | the most valuable letter among the five letters |
| nó là nền của | it is the ground of |
| lý do tồn tại của | the reason why … exists |
| đóng lại phần SOLID bằng một câu chuyện về đánh đổi | closes the SOLID part with a story about trade-offs |
| cũng có thể trở thành over-engineering | can also turn into over-engineering |
| áp SOLID một cách máy móc | applies SOLID mechanically |
| nối thẳng sang phần judgment | connects straight to the judgment part |
| cách chúng ta nói về giới hạn của một nguyên tắc | the way we talk about the limits of a principle |
| cách chúng ta đọc thuộc nguyên tắc đó | the way we recite that principle |

**Thuật ngữ cần nhớ**

- đắt giá, có giá trị → **valuable**
- khung nền, nền tảng → **the ground** / **the foundation**
- yêu cầu gộp code → **pull request**
- giới hạn → **limit**
- đọc thuộc → **recite**

---

## Phần 2 — Hai vế của DIP và việc đảo mũi tên phụ thuộc

**Tiếng Việt**

DIP gồm hai vế, và chúng ta nên đọc đủ cả hai vế khi trả lời phỏng vấn. Vế thứ nhất nói rằng module cấp cao không được phụ thuộc vào module cấp thấp, mà cả hai phải cùng phụ thuộc vào abstraction. Vế thứ hai nói rằng abstraction không phụ thuộc vào chi tiết, còn chi tiết thì phụ thuộc vào abstraction. Module cấp cao ở đây là phần logic nghiệp vụ, tức là lý do vì sao ứng dụng tồn tại, còn module cấp thấp là các chi tiết kỹ thuật như Postgres hay SMTP. Điều mà DIP làm là nó đảo chiều mũi tên phụ thuộc, bởi vì thay vì để nghiệp vụ trỏ thẳng vào database, chúng ta để nghiệp vụ trỏ vào một interface còn phần hiện thực database cũng trỏ vào chính interface đó. Nếu chúng ta không đảo mũi tên này, thì công nghệ lưu trữ sẽ quyết định hình dạng của logic nghiệp vụ, và đó là điều ngược với thứ tự đúng.

**English (bám cấu trúc tiếng Việt)**

DIP has two halves, and we should state both halves fully when we answer in an interview. The first half says that high-level modules must not depend on low-level modules, but both of them must depend on abstractions. The second half says that abstractions do not depend on details, while details depend on abstractions. The high-level module here is the business logic, that is, the reason why the application exists, while the low-level module is the technical details such as Postgres or SMTP. What DIP does is that it inverts the direction of the dependency arrow, because instead of letting the business point straight at the database, we let the business point at an interface while the database implementation also points at that same interface. If we do not invert this arrow, then the storage technology will decide the shape of the business logic, and that is the opposite of the correct order.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| DIP gồm hai vế | DIP has two halves |
| nên đọc đủ cả hai vế | should state both halves fully |
| module cấp cao / module cấp thấp | high-level modules / low-level modules |
| lý do vì sao ứng dụng tồn tại | the reason why the application exists |
| nó đảo chiều mũi tên phụ thuộc | it inverts the direction of the dependency arrow |
| thay vì để nghiệp vụ trỏ thẳng vào database | instead of letting the business point straight at the database |
| cũng trỏ vào chính interface đó | also points at that same interface |
| công nghệ lưu trữ sẽ quyết định hình dạng của | the storage technology will decide the shape of |
| ngược với thứ tự đúng | the opposite of the correct order |

**Thuật ngữ cần nhớ**

- vế (của một phát biểu) → **half** / **part**
- cấp cao / cấp thấp → **high-level** / **low-level**
- đảo chiều → **invert**
- mũi tên phụ thuộc → **dependency arrow**
- công nghệ lưu trữ → **storage technology**

---

## Phần 3 — Port và adapter qua ví dụ `OrderService`

**Tiếng Việt**

Chúng ta hãy nhìn một ví dụ rất cụ thể là `OrderService`. Đây là module cấp cao, và nó phụ thuộc vào một interface tên là `OrderRepository`, và người ta gọi một interface như vậy là một port. Class `PostgresOrderRepository` là module cấp thấp, và nó hiện thực cái port đó, cho nên người ta gọi nó là một adapter. Khi công ty quyết định chuyển từ Postgres sang Mongo, chúng ta chỉ thêm một adapter mới, và chúng ta không đụng một dòng nào trong `OrderService`. Khi viết test, chúng ta truyền vào một `InMemoryOrderRepository`, nhờ đó bài test chạy trong vài mili giây mà nó không cần một database thật. Điểm quan trọng nhất trong cách sắp xếp này là interface do phía nghiệp vụ định nghĩa, chứ nó không do phía hạ tầng định nghĩa.

**English (bám cấu trúc tiếng Việt)**

Let us look at a very concrete example, which is `OrderService`. This is the high-level module, and it depends on an interface named `OrderRepository`, and people call an interface like this a port. The `PostgresOrderRepository` class is the low-level module, and it implements that port, so people call it an adapter. When the company decides to move from Postgres to Mongo, we only add a new adapter, and we do not touch a single line in `OrderService`. When we write tests, we pass in an `InMemoryOrderRepository`, thanks to that the test runs in a few milliseconds and it does not need a real database. The most important point in this arrangement is that the interface is defined by the business side, and it is not defined by the infrastructure side.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| người ta gọi một interface như vậy là một port | people call an interface like this a port |
| nó hiện thực cái port đó | it implements that port |
| chúng ta không đụng một dòng nào trong | we do not touch a single line in |
| chạy trong vài mili giây | runs in a few milliseconds |
| điểm quan trọng nhất trong cách sắp xếp này | the most important point in this arrangement |
| do phía nghiệp vụ định nghĩa | is defined by the business side |
| nó không do phía hạ tầng định nghĩa | it is not defined by the infrastructure side |

**Thuật ngữ cần nhớ**

- cổng vào ra của nghiệp vụ → **port**
- bộ chuyển tiếp → **adapter**
- cách sắp xếp → **arrangement**
- phía nghiệp vụ → **the business side**
- mili giây → **millisecond**

---

## Phần 4 — DIP khác DI: bẫy phỏng vấn kinh điển

**Tiếng Việt**

Đây là bẫy phỏng vấn kinh điển mà chúng ta phải chuẩn bị thật kỹ. DIP là một nguyên tắc, và nó nói rằng chúng ta phải phụ thuộc vào abstraction. DI, tức là Dependency Injection, chỉ là một kỹ thuật, và nó nói về việc cung cấp dependency từ bên ngoài vào qua constructor, qua setter, hoặc qua một container. Có DI thì chưa chắc đã có DIP, bởi vì nếu chúng ta inject thẳng một class cụ thể như `StripeClient` thì chúng ta vẫn đang phụ thuộc vào chi tiết. Muốn đạt DIP, chúng ta phải inject qua một interface hoặc một abstraction, chứ chúng ta không chỉ cần chuyển việc khởi tạo ra bên ngoài. Khi phỏng vấn, việc nói được đúng câu rằng inject một class cụ thể vẫn là DI nhưng chưa phải DIP là một trong những cách nhanh nhất để cho thấy chúng ta hiểu sâu.

**English (bám cấu trúc tiếng Việt)**

This is the classic interview trap that we have to prepare for very carefully. DIP is a principle, and it says that we must depend on abstractions. DI, that is, Dependency Injection, is only a technique, and it is about supplying a dependency from the outside through the constructor, through a setter, or through a container. Having DI does not necessarily mean having DIP, because if we inject a concrete class such as `StripeClient` directly then we are still depending on a detail. To achieve DIP, we have to inject through an interface or an abstraction, and we do not only need to move the creation to the outside. In an interview, being able to say exactly the sentence that injecting a concrete class is still DI but is not yet DIP is one of the fastest ways to show that we understand deeply.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| mà chúng ta phải chuẩn bị thật kỹ | that we have to prepare for very carefully |
| chỉ là một kỹ thuật | is only a technique |
| cung cấp dependency từ bên ngoài vào | supplying a dependency from the outside |
| có DI thì chưa chắc đã có DIP | having DI does not necessarily mean having DIP |
| nếu chúng ta inject thẳng một class cụ thể | if we inject a concrete class … directly |
| chuyển việc khởi tạo ra bên ngoài | move the creation to the outside |
| việc nói được đúng câu rằng | being able to say exactly the sentence that |
| một trong những cách nhanh nhất để cho thấy | one of the fastest ways to show |

**Thuật ngữ cần nhớ**

- tiêm phụ thuộc → **dependency injection**
- kỹ thuật (cách làm) → **technique**
- bộ chứa quản lý phụ thuộc → **container**
- khởi tạo → **creation** / **instantiation**
- chưa chắc đã → **does not necessarily mean**

---

## Phần 5 — Vì sao DIP làm cho code dễ test

**Tiếng Việt**

Lý do DIP làm cho code dễ test nằm ở chỗ phụ thuộc trỏ vào abstraction chứ nó không trỏ vào thứ thật. Khi viết test, chúng ta thay hiện thực thật bằng một test double, tức là bằng một mock, một stub hoặc một fake. Nhờ đó chúng ta tách phần nghiệp vụ ra khỏi phần vào ra thật, ví dụ chúng ta tách nó khỏi database và khỏi network. Kết quả là unit test chạy nhanh và cho kết quả xác định, bởi vì nó không phụ thuộc vào mạng chậm hay vào dữ liệu còn sót lại từ lần chạy trước. Nếu chúng ta không có seam để cắm test double vào, thì mỗi bài test buộc phải dựng cả hạ tầng lên, và bộ test sẽ chậm tới mức cả đội bỏ luôn việc chạy nó. Đây là hậu quả rất thật, bởi vì một bộ test mà không ai chạy thì cũng như không có bộ test nào.

**English (bám cấu trúc tiếng Việt)**

The reason DIP makes code easy to test lies in the fact that the dependency points at an abstraction and it does not point at the real thing. When we write tests, we replace the real implementation with a test double, that is, with a mock, a stub or a fake. Thanks to that we separate the business part from the real input and output, for example we separate it from the database and from the network. The result is that unit tests run fast and give deterministic results, because they do not depend on a slow network or on data left over from the previous run. If we do not have a seam to plug a test double into, then every test has to stand up the whole infrastructure, and the test suite will get so slow that the whole team gives up running it. This is a very real consequence, because a test suite that nobody runs is the same as having no test suite at all.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| nằm ở chỗ | lies in the fact that |
| nó không trỏ vào thứ thật | it does not point at the real thing |
| thay hiện thực thật bằng | replace the real implementation with |
| tách phần nghiệp vụ ra khỏi phần vào ra thật | separate the business part from the real input and output |
| cho kết quả xác định | give deterministic results |
| dữ liệu còn sót lại từ lần chạy trước | data left over from the previous run |
| seam để cắm test double vào | a seam to plug a test double into |
| chậm tới mức cả đội bỏ luôn việc chạy nó | so slow that the whole team gives up running it |
| cũng như không có bộ test nào | is the same as having no test suite at all |

**Thuật ngữ cần nhớ**

- vật thế thân trong test → **test double**
- bản giả ghi lại lời gọi → **mock** / **stub** / **fake**
- vào ra → **input and output (I/O)**
- xác định, luôn ra cùng kết quả → **deterministic**
- chỗ cắm để test → **seam**

---

## Phần 6 — Nối cả năm chữ SOLID về lại coupling và cohesion

**Tiếng Việt**

Bây giờ chúng ta hãy nối cả năm chữ SOLID về lại chiếc la bàn ở Bài 1. SRP và ISP kéo cohesion lên, bởi vì SRP gom đúng một trách nhiệm vào một class, còn ISP giữ cho interface gọn đúng theo nhu cầu của client. OCP và DIP kéo coupling xuống, bởi vì cả hai đều bắt chúng ta phụ thuộc vào abstraction và đều tránh việc sửa lan sang code cũ. LSP đứng ở giữa như một điều kiện cần, bởi vì nếu subtype không thay thế được supertype thì mọi abstraction ở trên đều mất giá trị. Mục tiêu cuối cùng của cả năm chữ vẫn là loose coupling và high cohesion, chứ năm chữ đó không phải là mục tiêu tự thân. Khi trả lời phỏng vấn theo cách này, chúng ta cho thấy mình nhìn SOLID như một phương tiện, chứ chúng ta không nhìn nó như một danh sách phải học thuộc.

**English (bám cấu trúc tiếng Việt)**

Now let us connect all five SOLID letters back to the compass from Lesson 1. SRP and ISP pull cohesion up, because SRP gathers exactly one responsibility into one class, while ISP keeps the interface tight to the needs of the client. OCP and DIP push coupling down, because both of them make us depend on abstractions and both avoid changes spreading into old code. LSP stands in the middle as a necessary condition, because if a subtype cannot replace its supertype then every abstraction above it loses its value. The final goal of all five letters is still loose coupling and high cohesion, and those five letters are not a goal in themselves. When we answer an interview this way, we show that we see SOLID as a means, and we do not see it as a list that has to be memorised.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| nối cả năm chữ … về lại chiếc la bàn | connect all five letters … back to the compass |
| giữ cho interface gọn đúng theo nhu cầu của client | keeps the interface tight to the needs of the client |
| tránh việc sửa lan sang code cũ | avoid changes spreading into old code |
| đứng ở giữa như một điều kiện cần | stands in the middle as a necessary condition |
| thì mọi abstraction ở trên đều mất giá trị | then every abstraction above it loses its value |
| không phải là mục tiêu tự thân | are not a goal in themselves |
| nhìn SOLID như một phương tiện | see SOLID as a means |
| một danh sách phải học thuộc | a list that has to be memorised |

**Thuật ngữ cần nhớ**

- điều kiện cần → **a necessary condition**
- mất giá trị → **lose its value**
- mục tiêu tự thân → **a goal in itself**
- phương tiện → **a means**
- lan ra → **spread**

---

## Phần 7 — SOLID cũng có thể là over-engineering

**Tiếng Việt**

Chúng ta phải nói thẳng rằng SOLID cũng có thể trở thành over-engineering. Với một script chạy một lần, một công cụ nội bộ nhỏ, hoặc một bản thử nghiệm rồi sẽ bị vứt đi, việc thêm interface và thêm tầng gián tiếp chỉ làm chậm công việc mà nó không đem lại gì. Trường hợp nguy hiểm hơn là chúng ta tạo abstraction quá sớm khi chúng ta chưa nhìn rõ trục biến thiên thật của bài toán, bởi vì lúc đó chúng ta rất dễ chọn nhầm chỗ để mở. Nguyên tắc thực dụng là chúng ta chỉ áp dụng khi đã đau thật, chứ chúng ta không áp dụng theo kiểu giáo điều. Một cách nói an toàn trong phỏng vấn là chúng ta xem SOLID như thuốc, bởi vì đúng liều thì khỏi bệnh còn quá liều thì ngộ độc. Người phỏng vấn cho vị trí senior thường chờ đúng câu trả lời có sắc thái này, chứ họ không chờ một lời ca ngợi SOLID vô điều kiện.

**English (bám cấu trúc tiếng Việt)**

We have to say plainly that SOLID can also turn into over-engineering. For a script that runs once, a small internal tool, or an experiment that will be thrown away later, adding interfaces and adding layers of indirection only slows the work down and it brings nothing. The more dangerous case is that we create an abstraction too early when we do not yet see the real axis of change of the problem, because at that moment we very easily pick the wrong place to open. The pragmatic principle is that we only apply it when we have felt real pain, and we do not apply it in a dogmatic way. A safe way to say it in an interview is that we see SOLID as medicine, because the right dose cures the illness while an overdose poisons the patient. Interviewers for a senior position usually wait for exactly this kind of nuanced answer, and they do not wait for unconditional praise of SOLID.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| chúng ta phải nói thẳng rằng | we have to say plainly that |
| một bản thử nghiệm rồi sẽ bị vứt đi | an experiment that will be thrown away later |
| chỉ làm chậm công việc | only slows the work down |
| khi chúng ta chưa nhìn rõ trục biến thiên thật | when we do not yet see the real axis of change |
| rất dễ chọn nhầm chỗ để mở | very easily pick the wrong place to open |
| chỉ áp dụng khi đã đau thật | only apply it when we have felt real pain |
| không áp dụng theo kiểu giáo điều | do not apply it in a dogmatic way |
| đúng liều thì khỏi bệnh còn quá liều thì ngộ độc | the right dose cures the illness while an overdose poisons the patient |
| câu trả lời có sắc thái này | this kind of nuanced answer |
| một lời ca ngợi SOLID vô điều kiện | unconditional praise of SOLID |

**Thuật ngữ cần nhớ**

- công cụ nội bộ → **an internal tool**
- vứt đi → **throw away**
- thực dụng → **pragmatic**
- giáo điều → **dogmatic**
- liều lượng / quá liều → **dose** / **overdose**
- có sắc thái, có cân nhắc → **nuanced**

---

## Phần 8 — Phản hồi một review "interface một-một", và soát code AI

**Tiếng Việt**

Chúng ta hãy xử lý một tình huống review rất hay gặp. Một bạn junior tạo một interface riêng cho mọi class trong service, mỗi interface chỉ có đúng một hiện thực, và bạn ấy giải thích rằng làm như vậy là cho đúng SOLID. Đây là over-abstraction, và nó vi phạm YAGNI mà chúng ta đã học ở Bài 3. Một interface chỉ thật sự có giá trị khi nó có ít nhất hai hiện thực, hoặc khi chúng ta cần đảo chiều phụ thuộc, hoặc khi chúng ta cần một seam thật để test. Cách phản hồi tốt không phải là chê thẳng, mà là dẫn dắt bằng câu hỏi, ví dụ chúng ta hỏi rằng interface này sắp có hiện thực thứ hai nào chưa, và bài test hiện tại có cần một seam ở chỗ này không. Khi bạn ấy tự trả lời hai câu hỏi đó, bạn ấy tự nhìn ra vấn đề, và cách này giữ được động lực học của bạn ấy tốt hơn nhiều so với một lời chê.

Chính hai lỗi này cũng là thứ chúng ta hay gặp khi soát code do AI sinh ra. AI rất hay inject một class cụ thể vào constructor, và như vậy nó dừng lại ở DI mà nó không đạt DIP. AI cũng rất hay đẻ ra những interface một-một thừa thãi, bởi vì mẫu code đó xuất hiện rất dày trong dữ liệu huấn luyện. Khi review, chúng ta hỏi hai câu: phụ thuộc này có trỏ vào một abstraction hay không, và interface này có một lý do tồn tại thật hay không. Nếu câu thứ nhất trả lời là không, chúng ta yêu cầu tách port và adapter, còn nếu câu thứ hai trả lời là không, chúng ta yêu cầu xoá bớt interface đi.

**English (bám cấu trúc tiếng Việt)**

Let us handle a review situation that we meet very often. A junior teammate creates a separate interface for every class in the service, each interface has exactly one implementation, and they explain that doing it this way is for the sake of SOLID. This is over-abstraction, and it violates the YAGNI that we learned in Lesson 3. An interface only truly has value when it has at least two implementations, or when we need to invert a dependency, or when we need a real seam for testing. The good way to respond is not to criticise directly, but to guide with questions, for example we ask whether this interface has a second implementation coming yet, and whether the current tests need a seam at this place. When they answer those two questions themselves, they see the problem themselves, and this way protects their motivation to learn much better than a piece of criticism does.

These two mistakes are exactly what we also meet often when we review code that AI generates. AI very often injects a concrete class into the constructor, and in that way it stops at DI and it does not reach DIP. AI also very often produces useless one-to-one interfaces, because that code pattern appears very densely in the training data. When we review, we ask two questions: does this dependency point at an abstraction or not, and does this interface have a real reason to exist or not. If the answer to the first question is no, we ask for a port and an adapter to be separated, while if the answer to the second question is no, we ask for some interfaces to be deleted.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| một tình huống review rất hay gặp | a review situation that we meet very often |
| bạn ấy giải thích rằng làm như vậy là cho đúng SOLID | they explain that doing it this way is for the sake of SOLID |
| chỉ thật sự có giá trị khi | only truly has value when |
| cách phản hồi tốt không phải là chê thẳng | the good way to respond is not to criticise directly |
| dẫn dắt bằng câu hỏi | guide with questions |
| sắp có hiện thực thứ hai nào chưa | has a second implementation coming yet |
| bạn ấy tự nhìn ra vấn đề | they see the problem themselves |
| giữ được động lực học của bạn ấy | protects their motivation to learn |
| nó dừng lại ở DI mà nó không đạt DIP | it stops at DI and it does not reach DIP |
| xuất hiện rất dày trong dữ liệu huấn luyện | appears very densely in the training data |
| chúng ta yêu cầu xoá bớt interface đi | we ask for some interfaces to be deleted |

**Thuật ngữ cần nhớ**

- trừu tượng hoá quá mức → **over-abstraction**
- chê, phê bình → **criticise** / **criticism**
- dẫn dắt → **guide**
- động lực → **motivation**
- dữ liệu huấn luyện → **training data**

---

## Mô hình ghi nhớ

**Tiếng Việt**

Với DIP, chúng ta đảo mũi tên phụ thuộc, nghĩa là phía nghiệp vụ định nghĩa interface còn phía hạ tầng cắm vào interface đó. DI chỉ là cách đưa đồ vào tận tay, và SOLID là thuốc, cho nên đúng liều thì khỏi bệnh còn quá liều thì ngộ độc.

**English (bám cấu trúc tiếng Việt)**

With DIP, we invert the dependency arrow, which means that the business side defines the interface while the infrastructure side plugs into that interface. DI is only the way of handing the thing over, and SOLID is medicine, therefore the right dose cures the illness while an overdose poisons the patient.

---

## Bảng thuật ngữ tổng hợp

| Tiếng Việt | Tiếng Anh | Ghi chú phát âm |
|---|---|---|
| đắt giá, có giá trị | valuable | /ˈvæljuəbl/ — VAL-yu-a-ble, trọng âm đầu |
| nền tảng | the ground / the foundation | *foundation* trọng âm áp chót: foun-DA-tion |
| yêu cầu gộp code | pull request | *request* trọng âm cuối: re-QUEST |
| giới hạn | limit | |
| đọc thuộc | recite | trọng âm cuối: re-CITE, đuôi đọc "-sait" |
| kiến trúc lục giác | Hexagonal architecture | *hexagonal* trọng âm âm hai: hex-A-go-nal |
| vế (của phát biểu) | half / part | *half* /hɑːf/ — chữ **l** câm, đọc "haaf" |
| cấp cao / cấp thấp | high-level / low-level | |
| đảo chiều | invert / inversion | *invert* trọng âm cuối: in-VERT |
| mũi tên phụ thuộc | dependency arrow | *dependency* trọng âm âm hai: de-PEN-den-cy; *arrow* /ˈærəʊ/ — A-rrow |
| công nghệ lưu trữ | storage technology | *storage* trọng âm đầu: STOR-age |
| cổng vào ra của nghiệp vụ | port | |
| bộ chuyển tiếp | adapter | trọng âm âm hai: a-DAP-ter |
| cách sắp xếp | arrangement | trọng âm âm hai: ar-RANGE-ment |
| hạ tầng | infrastructure | trọng âm đầu: IN-fra-struc-ture, cụm **-str-** phải bật |
| mili giây | millisecond | |
| tiêm phụ thuộc | dependency injection | *injection* trọng âm âm hai: in-JEC-tion |
| kỹ thuật (cách làm) | technique | trọng âm cuối: tech-NIQUE, đọc "tek-NIIK" |
| bộ chứa quản lý phụ thuộc | container | trọng âm âm hai: con-TAI-ner |
| khởi tạo | creation / instantiation | *instantiation* — in-stan-ti-A-tion, đọc chậm từng âm |
| chưa chắc đã | does not necessarily mean | *necessarily* trọng âm âm ba: ne-ce-SSA-ri-ly |
| vật thế thân trong test | test double | |
| bản giả trong test | mock / stub / fake | *stub* đuôi **-b** đọc nhẹ nhưng có bật |
| vào ra | input and output (I/O) | đọc từng chữ: "ai-OH" |
| luôn ra cùng kết quả | deterministic | trọng âm áp chót: de-ter-mi-NIS-tic |
| chỗ cắm để test | seam | /siːm/ — đọc giống "seem" |
| điều kiện cần | a necessary condition | *necessary* trọng âm đầu: NE-ce-ssa-ry |
| mục tiêu tự thân | a goal in itself | |
| phương tiện | a means | luôn có **-s** kể cả khi số ít, đọc /miːnz/ |
| lan ra | spread | /spred/ — vần với "bread", không đọc "spriit" |
| công cụ nội bộ | an internal tool | *internal* trọng âm âm hai: in-TER-nal |
| vứt đi | throw away | *throw* có âm **th**, đặt lưỡi giữa hai hàm răng |
| thực dụng | pragmatic | trọng âm âm hai: prag-MA-tic |
| giáo điều | dogmatic | trọng âm âm hai: dog-MA-tic |
| liều lượng / quá liều | dose / overdose | *dose* /dəʊs/ — đuôi đọc **-s**, không đọc /dəʊz/ |
| ngộ độc | poison | /ˈpɔɪzn/ — POI-zn |
| có sắc thái, có cân nhắc | nuanced | /ˈnjuːɑːnst/ — NEW-ahnst, người Việt hay bỏ đuôi **-nst** |
| ca ngợi | praise | /preɪz/ — đuôi đọc **-z** |
| trừu tượng hoá quá mức | over-abstraction | |
| chê, phê bình | criticise / criticism | *criticism* trọng âm đầu: CRI-ti-cism |
| dẫn dắt | guide | chữ **u** câm: đọc "gaid" |
| động lực | motivation | trọng âm áp chót: mo-ti-VA-tion |
| dữ liệu huấn luyện | training data | |

---

## Bài nói

> **Cách làm:** ghi âm lại giọng của mình, mỗi câu nói liên tục **60–90 giây**, **không nhìn tài liệu** khi nói. Mỗi bài nói phải dùng ít nhất **6 thuật ngữ** trong bảng ở trên. Nghe lại bản ghi âm một lần, ghi ra hai chỗ mình bị vấp, rồi nói lại lần hai.

1. State both halves of the Dependency Inversion Principle, and explain what "high-level" and "low-level" mean with an example from your own work.
2. A colleague says: "We use the Nest DI container everywhere, so our codebase already follows DIP." Explain what is wrong with that claim and what you would look for in the code to check it.
3. Explain to a junior developer why depending on an interface makes a service easy to unit test, and describe what the test looks like before and after the change.
4. Describe step by step what you would change to move an `OrderService` that imports a Postgres client directly onto a port and adapter design.
5. Someone on your team creates a one-to-one interface for every class "to follow SOLID". Describe out loud how you would respond in the pull request, using questions rather than criticism.
6. When would you deliberately not apply SOLID? Give the trade-off in both directions and name the kind of project where you would skip it.
7. You are reviewing AI-generated code for a payment service. Describe the two questions you would ask about the dependencies and the interfaces, and what change you would request in each case.
