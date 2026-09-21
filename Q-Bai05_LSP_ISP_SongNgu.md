# Bài 5 — SOLID (2): Liskov Substitution & Interface Segregation
### Bản song ngữ: tiếng Anh bám cấu trúc tiếng Việt

> **Cách dùng bài này:** đọc đoạn tiếng Việt trước, tự dịch trong đầu, rồi mới nhìn sang đoạn tiếng Anh để so. Sau khi so xong, **đọc to đoạn tiếng Anh hai lần** — lần một để nhìn chữ, lần hai để nghe chính mình. Phần bài nói ở cuối là bắt buộc; không nói ra thì bài này chỉ là bài đọc.

---

## Phần 1 — Hai chữ ít người nói được nhưng lộ rõ ai hiểu OOP

**Tiếng Việt**

LSP là bài kiểm tra để chúng ta biết một quan hệ kế thừa có hợp lệ hay không, và vì vậy nó nối thẳng vào Bài 2, nơi chúng ta đã học rằng nên ưu tiên composition hơn inheritance. ISP là SRP áp cho interface, nghĩa là nó kéo cohesion lên ở mức contract chứ không chỉ ở mức class. Hai chữ này thường bị bỏ qua khi người ta ôn SOLID, bởi vì chúng khó nói hơn SRP và OCP. Chính vì vậy chúng lộ rõ ai thật sự hiểu OOP và ai chỉ học thuộc năm chữ cái. Nếu chúng ta nói được hai chữ này bằng ví dụ của riêng mình, chúng ta tạo ra khác biệt rất lớn ở vòng phỏng vấn thiết kế.

**English (bám cấu trúc tiếng Việt)**

LSP is the test that lets us know whether an inheritance relationship is valid or not, and therefore it connects straight back to Lesson 2, where we learned that we should favour composition over inheritance. ISP is SRP applied to interfaces, which means that it pulls cohesion up at the contract level and not only at the class level. These two letters are often skipped when people revise SOLID, because they are harder to talk about than SRP and OCP. That is exactly why they show clearly who really understands OOP and who has only memorised five letters. If we can talk about these two letters with our own examples, we make a very large difference in the design interview round.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| bài kiểm tra để chúng ta biết … có hợp lệ hay không | the test that lets us know whether … is valid or not |
| nó nối thẳng vào Bài 2 | it connects straight back to Lesson 2 |
| SRP áp cho interface | SRP applied to interfaces |
| ở mức contract chứ không chỉ ở mức class | at the contract level and not only at the class level |
| thường bị bỏ qua khi người ta ôn SOLID | are often skipped when people revise SOLID |
| chúng khó nói hơn | they are harder to talk about |
| ai chỉ học thuộc năm chữ cái | who has only memorised five letters |
| bằng ví dụ của riêng mình | with our own examples |

**Thuật ngữ cần nhớ**

- hợp lệ → **valid**
- ôn lại → **revise**
- học thuộc → **memorise**
- mức, cấp → **level**
- khác biệt → **difference**

---

## Phần 2 — Phát biểu của LSP và ba ràng buộc đi kèm

**Tiếng Việt**

Phát biểu của LSP là một subtype phải thay thế được supertype mà nó không phá vỡ hành vi hoặc contract mà client trông đợi. Có ba ràng buộc hình thức đáng nhớ đi kèm phát biểu này. Ràng buộc thứ nhất là tiền điều kiện ở subtype không được mạnh hơn tiền điều kiện ở supertype, nghĩa là class con không được đòi hỏi nhiều hơn class cha. Ràng buộc thứ hai là hậu điều kiện ở subtype không được yếu hơn hậu điều kiện ở supertype, nghĩa là class con không được hứa ít hơn class cha. Ràng buộc thứ ba là invariant của class cha phải được giữ nguyên trong class con. Khi chúng ta nói ba ràng buộc này ra trong phỏng vấn, chúng ta cho thấy rằng chúng ta hiểu LSP như một bản hợp đồng, chứ chúng ta không hiểu nó như một câu khẩu hiệu.

**English (bám cấu trúc tiếng Việt)**

The statement of LSP is that a subtype must be able to replace the supertype without breaking the behaviour or the contract that the client expects. There are three formal constraints worth remembering that come with this statement. The first constraint is that the precondition in the subtype must not be stronger than the precondition in the supertype, which means that the child class must not demand more than the parent class. The second constraint is that the postcondition in the subtype must not be weaker than the postcondition in the supertype, which means that the child class must not promise less than the parent class. The third constraint is that the invariant of the parent class must be kept unchanged in the child class. When we say these three constraints out loud in an interview, we show that we understand LSP as a contract, and we do not understand it as a slogan.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| phải thay thế được supertype | must be able to replace the supertype |
| mà nó không phá vỡ hành vi | without breaking the behaviour |
| mà client trông đợi | that the client expects |
| ba ràng buộc hình thức đáng nhớ | three formal constraints worth remembering |
| không được mạnh hơn | must not be stronger than |
| không được đòi hỏi nhiều hơn | must not demand more than |
| không được hứa ít hơn | must not promise less than |
| phải được giữ nguyên | must be kept unchanged |
| khi chúng ta nói … ra trong phỏng vấn | when we say … out loud in an interview |
| chứ chúng ta không hiểu nó như một câu khẩu hiệu | and we do not understand it as a slogan |

**Thuật ngữ cần nhớ**

- kiểu con / kiểu cha → **subtype** / **supertype**
- ràng buộc → **constraint**
- tiền điều kiện → **precondition**
- hậu điều kiện → **postcondition**
- bất biến → **invariant**
- khẩu hiệu → **slogan**

---

## Phần 3 — Hình vuông kế thừa hình chữ nhật: đúng toán, sai hành vi

**Tiếng Việt**

Ví dụ vi phạm kinh điển nhất là một class `Square` kế thừa từ class `Rectangle`. Trong hình học, một hình vuông đúng là một hình chữ nhật, cho nên quan hệ này nghe rất hợp lý. Vấn đề xuất hiện khi chúng ta đặt chiều rộng cho một hình vuông, bởi vì hình vuông buộc phải đổi luôn chiều cao để giữ tính chất của nó. Mọi đoạn code viết dựa trên contract rằng đặt chiều rộng thì chiều cao không đổi sẽ vỡ khi nó nhận vào một hình vuông. Đây chính là chỗ chúng ta thấy một quan hệ đúng về mặt toán học nhưng sai về mặt hành vi trong code. Hậu quả rất khó chịu, bởi vì code vẫn biên dịch được và lỗi chỉ hiện ra lúc chạy, ở một chỗ nằm rất xa nơi chúng ta tạo ra object.

**English (bám cấu trúc tiếng Việt)**

The most classic violation example is a `Square` class that inherits from a `Rectangle` class. In geometry, a square really is a rectangle, so this relationship sounds very reasonable. The problem appears when we set the width of a square, because the square is forced to change the height as well in order to keep its property. Every piece of code written on the contract that setting the width leaves the height unchanged will break when it receives a square. This is exactly the place where we see a relationship that is correct in mathematics but wrong in behaviour inside the code. The consequence is very unpleasant, because the code still compiles and the error only shows up at runtime, in a place that sits very far from where we created the object.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| ví dụ vi phạm kinh điển nhất | the most classic violation example |
| quan hệ này nghe rất hợp lý | this relationship sounds very reasonable |
| buộc phải đổi luôn chiều cao | is forced to change the height as well |
| để giữ tính chất của nó | in order to keep its property |
| mọi đoạn code viết dựa trên contract rằng | every piece of code written on the contract that |
| đúng về mặt toán học nhưng sai về mặt hành vi | correct in mathematics but wrong in behaviour |
| hậu quả rất khó chịu | the consequence is very unpleasant |
| lỗi chỉ hiện ra lúc chạy | the error only shows up at runtime |
| nằm rất xa nơi chúng ta tạo ra object | sits very far from where we created the object |

**Thuật ngữ cần nhớ**

- hình chữ nhật / hình vuông → **rectangle** / **square**
- chiều rộng / chiều cao → **width** / **height**
- hình học → **geometry**
- tính chất → **property**
- biên dịch → **compile**

---

## Phần 4 — Đà điểu kế thừa chim: "là một loại của" không đủ

**Tiếng Việt**

Ví dụ thứ hai cũng nổi tiếng không kém là một class `Ostrich` kế thừa từ class `Bird` có method `fly`. Con đà điểu không bay được, cho nên người viết code thường cho method `fly` của nó ném ra một exception. Khi một client cầm trong tay một object kiểu `Bird` và gọi `fly`, chương trình vỡ dù client đã làm đúng theo contract. Bài học rút ra là quan hệ phân loại ngoài đời thật không giống quan hệ thay thế về hành vi trong phần mềm. Nói cách khác, "là một loại của" không tự động kéo theo "hành xử được như là". Khi hai thứ đó không đi cùng nhau, chúng ta bỏ inheritance và chúng ta chuyển sang composition, đúng như Bài 2 đã khuyên.

**English (bám cấu trúc tiếng Việt)**

The second example, which is no less famous, is an `Ostrich` class that inherits from a `Bird` class that has a `fly` method. An ostrich cannot fly, so the person who writes the code usually makes its `fly` method throw an exception. When a client holds an object of type `Bird` and calls `fly`, the program breaks although the client did exactly what the contract said. The lesson we draw is that a classification relationship in real life is not the same as a behavioural substitution relationship in software. In other words, "is a kind of" does not automatically carry with it "behaves like a". When those two things do not go together, we drop inheritance and we move to composition, exactly as Lesson 2 advised.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| cũng nổi tiếng không kém | which is no less famous |
| ném ra một exception | throw an exception |
| khi một client cầm trong tay một object kiểu `Bird` | when a client holds an object of type `Bird` |
| dù client đã làm đúng theo contract | although the client did exactly what the contract said |
| bài học rút ra là | the lesson we draw is that |
| quan hệ phân loại ngoài đời thật | a classification relationship in real life |
| quan hệ thay thế về hành vi | a behavioural substitution relationship |
| nói cách khác | in other words |
| không tự động kéo theo | does not automatically carry with it |
| khi hai thứ đó không đi cùng nhau | when those two things do not go together |

**Thuật ngữ cần nhớ**

- đà điểu → **ostrich**
- ngoại lệ, lỗi ném ra → **exception**
- phân loại → **classification**
- thay thế → **substitution**
- hành xử như là → **behaves like a**

---

## Phần 5 — Override rồi ném lỗi "không hỗ trợ" là mùi LSP

**Tiếng Việt**

Có một câu hỏi bẫy mà chúng ta nên chuẩn bị sẵn, đó là câu "tôi override một method rồi ném ra lỗi không hỗ trợ, mà code vẫn biên dịch được đấy thôi". Câu trả lời của chúng ta là biên dịch được không có nghĩa là thiết kế đúng. Một method override chỉ để ném lỗi không hỗ trợ là dấu hiệu rất rõ rằng subtype không thay thế được supertype. Nó nói cho chúng ta biết rằng phân cấp kế thừa này đã sai ngay từ chỗ chúng ta đặt method đó vào class cha. Cách sửa có hai hướng, một là chúng ta tách interface để mỗi loại chỉ ký đúng phần nó làm được, hai là chúng ta chuyển sang composition. Nếu chúng ta để nguyên, mọi client đều phải viết thêm kiểm tra kiểu trước khi gọi, và chính những kiểm tra kiểu đó lại phá luôn OCP mà chúng ta vừa học ở Bài 4.

**English (bám cấu trúc tiếng Việt)**

There is a trap question that we should prepare in advance, which is the question "I override a method and then throw a not-supported error, and the code still compiles, doesn't it". Our answer is that compiling does not mean that the design is correct. An overridden method that only throws a not-supported error is a very clear sign that the subtype cannot replace the supertype. It tells us that this inheritance hierarchy was already wrong at the point where we put that method into the parent class. The fix has two directions, one is that we split the interface so that each type only signs the part it can do, and two is that we move to composition. If we leave it as it is, every client has to write an extra type check before calling, and those very type checks then break the OCP that we have just learned in Lesson 4.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| một câu hỏi bẫy mà chúng ta nên chuẩn bị sẵn | a trap question that we should prepare in advance |
| mà code vẫn biên dịch được đấy thôi | and the code still compiles, doesn't it |
| biên dịch được không có nghĩa là thiết kế đúng | compiling does not mean that the design is correct |
| là dấu hiệu rất rõ rằng | is a very clear sign that |
| đã sai ngay từ chỗ | was already wrong at the point where |
| mỗi loại chỉ ký đúng phần nó làm được | each type only signs the part it can do |
| nếu chúng ta để nguyên | if we leave it as it is |
| phải viết thêm kiểm tra kiểu trước khi gọi | has to write an extra type check before calling |
| chính những kiểm tra kiểu đó lại phá luôn | those very type checks then break |

**Thuật ngữ cần nhớ**

- ghi đè → **override** / **an overridden method**
- lỗi không hỗ trợ → **a not-supported error**
- phân cấp kế thừa → **inheritance hierarchy**
- ký (vào hợp đồng) → **sign**
- kiểm tra kiểu → **type check**

---

## Phần 6 — ISP: đừng bắt client phụ thuộc vào thứ nó không dùng

**Tiếng Việt**

ISP nói rằng một client không nên bị buộc phụ thuộc vào những method mà nó không dùng. Một interface quá to, mà người ta gọi là fat interface, chính là một cái mùi cần xử lý. Ví dụ thường gặp là một interface `Worker` chứa cả method `work` lẫn method `eat`. Khi chúng ta có một class `RobotWorker`, class đó buộc phải hiện thực method `eat` với thân hàm rỗng, và điều đó rõ ràng là sai. Cách sửa là chúng ta tách interface thành `Workable` và `Eatable`, rồi mỗi class chỉ ký đúng phần nó thật sự làm. ISP vì vậy chính là SRP áp cho interface, nghĩa là chúng ta gom đúng một nhóm hành vi gắn kết vào một contract, chứ chúng ta không nhồi mọi thứ vào cùng một chỗ.

**English (bám cấu trúc tiếng Việt)**

ISP says that a client should not be forced to depend on the methods that it does not use. An interface that is too big, which people call a fat interface, is exactly a smell that needs to be handled. The example we meet often is a `Worker` interface that holds both the `work` method and the `eat` method. When we have a `RobotWorker` class, that class is forced to implement the `eat` method with an empty body, and that is clearly wrong. The fix is that we split the interface into `Workable` and `Eatable`, and then each class only signs the part it truly does. ISP is therefore SRP applied to interfaces, which means that we gather exactly one group of cohesive behaviour into one contract, and we do not stuff everything into the same place.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| không nên bị buộc phụ thuộc vào | should not be forced to depend on |
| một interface quá to | an interface that is too big |
| chính là một cái mùi cần xử lý | is exactly a smell that needs to be handled |
| ví dụ thường gặp là | the example we meet often is |
| chứa cả … lẫn … | holds both … and … |
| với thân hàm rỗng | with an empty body |
| điều đó rõ ràng là sai | that is clearly wrong |
| chỉ ký đúng phần nó thật sự làm | only signs the part it truly does |
| một nhóm hành vi gắn kết | one group of cohesive behaviour |
| chúng ta không nhồi mọi thứ vào cùng một chỗ | we do not stuff everything into the same place |

**Thuật ngữ cần nhớ**

- interface quá to → **fat interface**
- bị ép buộc → **be forced to**
- thân hàm rỗng → **an empty body**
- gắn kết → **cohesive**
- nhồi nhét → **stuff**

---

## Phần 7 — Tách `Repository` và bài học interface nhỏ từ Go

**Tiếng Việt**

Chúng ta hãy áp ISP vào một ví dụ rất thật, đó là interface `Repository`. Nhiều dự án có một interface khổng lồ chứa cả thao tác đọc, thao tác ghi, thao tác ghi hàng loạt, thao tác stream và cả phần cache. Cách tốt hơn là chúng ta tách nó thành `ReadRepository` và `WriteRepository`, nhờ đó một service chỉ đọc dữ liệu sẽ chỉ phụ thuộc vào phần đọc. Việc tách này cũng chính là tiền đề cho CQRS mà chúng ta sẽ học ở Bài 12. Hệ sinh thái Go còn đi xa hơn nữa, bởi vì Go khuyến khích interface nhỏ, và Rob Pike có một nhận xét nổi tiếng rằng interface càng to thì abstraction càng yếu. Hai interface `io.Reader` và `io.Writer`, mỗi cái chỉ có đúng một method, là ví dụ về ISP ở mức cực đoan nhưng cực kỳ hiệu quả.

**English (bám cấu trúc tiếng Việt)**

Let us apply ISP to a very real example, which is the `Repository` interface. Many projects have a huge interface that holds read operations, write operations, bulk write operations, stream operations and the cache part as well. The better way is that we split it into `ReadRepository` and `WriteRepository`, thanks to that a service that only reads data will only depend on the read part. This split is also the ground for CQRS that we will study in Lesson 12. The Go ecosystem goes even further, because Go encourages small interfaces, and Rob Pike made a famous remark that the bigger an interface gets, the weaker its abstraction becomes. The two interfaces `io.Reader` and `io.Writer`, each of which has exactly one method, are an example of ISP at an extreme level but they are extremely effective.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| một interface khổng lồ chứa cả | a huge interface that holds |
| thao tác ghi hàng loạt | bulk write operations |
| cách tốt hơn là chúng ta tách nó thành | the better way is that we split it into |
| chỉ phụ thuộc vào phần đọc | will only depend on the read part |
| cũng chính là tiền đề cho | is also the ground for |
| còn đi xa hơn nữa | goes even further |
| có một nhận xét nổi tiếng rằng | made a famous remark that |
| interface càng to thì abstraction càng yếu | the bigger an interface gets, the weaker its abstraction becomes |
| ở mức cực đoan nhưng cực kỳ hiệu quả | at an extreme level but extremely effective |

**Thuật ngữ cần nhớ**

- thao tác → **operation**
- hàng loạt → **bulk**
- hệ sinh thái → **ecosystem**
- nhận xét → **remark**
- cực đoan → **extreme**

---

## Phần 8 — Sửa đúng bằng cách tách theo hành vi, và soát code AI

**Tiếng Việt**

Chúng ta hãy quay lại ví dụ hình học để thấy cách sửa đúng trông như thế nào. Thay vì cho `Square` kế thừa `Rectangle`, chúng ta định nghĩa một interface `Shape` chỉ yêu cầu đúng một khả năng, đó là tính được diện tích. Sau đó `Rectangle` và `Square` cùng hiện thực `Shape`, nhưng hai class đó không kế thừa lẫn nhau. Nguyên tắc chung là chúng ta tách abstraction theo hành vi mà client cần, chứ chúng ta không tách theo cách phân loại của toán học hay của đời thường. Khi chúng ta làm như vậy, mọi client đều thay thế được hình này bằng hình kia mà không có gì vỡ, và đó chính là lúc LSP được thoả mãn.

Hai chữ trong bài này cũng là bộ lọc rất hiệu quả khi chúng ta soát code do AI sinh ra. AI rất hay dựng phân cấp kế thừa theo lối phân loại đời thường, ví dụ nó cho `Penguin` kế thừa `Bird` có method `fly`. AI cũng rất hay sinh ra interface to nhồi nhét, bởi vì gộp mọi thứ vào một chỗ trông có vẻ tiện. Khi review, chúng ta hỏi một câu duy nhất cho LSP, đó là subtype này có thay thế được supertype ở mọi chỗ đang dùng hay không. Với ISP, chúng ta đếm xem có client nào phải hiện thực method rỗng hoặc phải ném lỗi không hỗ trợ hay không, và nếu có thì chúng ta yêu cầu tách interface.

**English (bám cấu trúc tiếng Việt)**

Let us go back to the geometry example to see what the correct fix looks like. Instead of letting `Square` inherit `Rectangle`, we define a `Shape` interface that requires exactly one capability, which is being able to compute an area. After that `Rectangle` and `Square` both implement `Shape`, but those two classes do not inherit from each other. The general principle is that we split abstractions by the behaviour that the client needs, and we do not split by the classification of mathematics or of everyday life. When we work this way, every client can replace one shape with the other shape and nothing breaks, and that is exactly the moment when LSP is satisfied.

The two letters in this lesson are also a very effective filter when we review code that AI generates. AI very often builds inheritance hierarchies along everyday classification lines, for example it lets `Penguin` inherit `Bird` which has a `fly` method. AI also very often produces big stuffed interfaces, because putting everything into one place looks convenient. When we review, we ask a single question for LSP, which is whether this subtype can replace the supertype in every place where it is used. For ISP, we count whether there is any client that has to implement an empty method or has to throw a not-supported error, and if there is one then we ask for the interface to be split.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| để thấy cách sửa đúng trông như thế nào | to see what the correct fix looks like |
| chỉ yêu cầu đúng một khả năng | requires exactly one capability |
| hai class đó không kế thừa lẫn nhau | those two classes do not inherit from each other |
| theo hành vi mà client cần | by the behaviour that the client needs |
| của toán học hay của đời thường | of mathematics or of everyday life |
| mà không có gì vỡ | and nothing breaks |
| đó chính là lúc LSP được thoả mãn | that is exactly the moment when LSP is satisfied |
| theo lối phân loại đời thường | along everyday classification lines |
| interface to nhồi nhét | big stuffed interfaces |
| ở mọi chỗ đang dùng | in every place where it is used |

**Thuật ngữ cần nhớ**

- khả năng → **capability**
- diện tích → **area**
- hình dạng → **shape**
- chim cánh cụt → **penguin**
- hiệu quả → **effective**

---

## Mô hình ghi nhớ

**Tiếng Việt**

Với LSP, class con phải đóng thế được class cha mà khán giả không nhận ra sự thay đổi. Với ISP, chúng ta đừng bắt ai ký hợp đồng cho phần việc mà họ không làm.

**English (bám cấu trúc tiếng Việt)**

With LSP, the child class must be able to stand in for the parent class without the audience noticing the change. With ISP, we should not make anyone sign a contract for work that they do not do.

---

## Bảng thuật ngữ tổng hợp

| Tiếng Việt | Tiếng Anh | Ghi chú phát âm |
|---|---|---|
| hợp lệ | valid | trọng âm đầu: VA-lid |
| ôn lại | revise | trọng âm cuối: re-VISE, đuôi đọc "-vaiz" |
| học thuộc | memorise | trọng âm đầu: ME-mo-rise |
| khác biệt | difference | /ˈdɪfrəns/ — hai âm tiết khi nói nhanh: "DIF-rợns" |
| kiểu con / kiểu cha | subtype / supertype | *super-* người Anh đọc "SOO-pơ" |
| ràng buộc | constraint | /kənˈstreɪnt/ — cụm **-str-** và đuôi **-nt** phải bật |
| tiền điều kiện | precondition | trọng âm ở *-di-*: pre-con-DI-tion |
| hậu điều kiện | postcondition | post-con-DI-tion |
| bất biến | invariant | trọng âm âm hai: in-VA-ri-ant |
| khẩu hiệu | slogan | trọng âm đầu: SLO-gan |
| hình chữ nhật | rectangle | trọng âm đầu: REC-tan-gle |
| hình vuông | square | /skweə/ — cụm **skw-** đầu từ khó, tập đọc chậm |
| chiều rộng | width | /wɪdθ/ — đuôi **-dth** rất khó, đặt lưỡi giữa răng ở cuối |
| chiều cao | height | /haɪt/ — đọc là "hait", KHÔNG có âm **th** ở cuối |
| hình học | geometry | trọng âm âm hai: ge-O-me-try |
| tính chất | property | trọng âm đầu: PRO-per-ty |
| biên dịch | compile | trọng âm cuối: com-PILE |
| đà điểu | ostrich | /ˈɒstrɪtʃ/ — OS-trich, đuôi **-tch** bật rõ |
| chim cánh cụt | penguin | /ˈpeŋgwɪn/ — PEN-gwin, có âm "gw" ở giữa |
| ngoại lệ, lỗi ném ra | exception | trọng âm âm hai: ex-CEP-tion |
| ném (lỗi) | throw | có âm **th** đầu, đặt lưỡi giữa hai hàm răng |
| phân loại | classification | trọng âm áp chót: clas-si-fi-CA-tion |
| thay thế | substitution | trọng âm áp chót: sub-sti-TU-tion |
| hành xử như là | behaves like a | *behaviour* trọng âm âm hai: be-HA-viour |
| ghi đè | override / overridden | trọng âm cuối: o-ver-RIDE / o-ver-RID-den |
| phân cấp kế thừa | inheritance hierarchy | *hierarchy* /ˈhaɪərɑːki/ — HI-er-ar-chy |
| ký (hợp đồng) | sign | chữ **g** câm: đọc là "sain" |
| kiểm tra kiểu | type check | |
| interface quá to | fat interface | |
| bị ép buộc | be forced to | |
| thân hàm rỗng | an empty body | |
| gắn kết | cohesive | trọng âm âm hai: co-HE-sive |
| nhồi nhét | stuff | đuôi **-ff** đọc rõ, không nuốt |
| thao tác | operation | trọng âm áp chót: o-pe-RA-tion |
| hàng loạt | bulk | đuôi **-lk** phải bật, đọc "bâlk" |
| hệ sinh thái | ecosystem | trọng âm đầu: E-co-sys-tem |
| nhận xét | remark | trọng âm cuối: re-MARK |
| cực đoan | extreme | trọng âm cuối: ex-TREME |
| khả năng | capability | trọng âm âm ba: ca-pa-BI-li-ty |
| diện tích | area | /ˈeəriə/ — "AIR-ri-ơ", trọng âm đầu |
| hiệu quả | effective | trọng âm âm hai: ef-FEC-tive |

---

## Bài nói

> **Cách làm:** ghi âm lại giọng của mình, mỗi câu nói liên tục **60–90 giây**, **không nhìn tài liệu** khi nói. Mỗi bài nói phải dùng ít nhất **6 thuật ngữ** trong bảng ở trên. Nghe lại bản ghi âm một lần, ghi ra hai chỗ mình bị vấp, rồi nói lại lần hai.

1. State the Liskov Substitution Principle in your own words, and add the three formal constraints about preconditions, postconditions and invariants.
2. Explain to a junior developer why `Square extends Rectangle` is a problem, even though a square really is a rectangle in geometry.
3. A colleague says: "I overrode the method and threw a not-supported error, and it compiles fine, so the design is acceptable." Explain what is wrong with that argument and what you would change.
4. Describe what happens to a client that receives an `Ostrich` through a `Bird` reference and calls `fly`, and say who is at fault in that design.
5. Someone on your team wants one big `Repository` interface with read, write, bulk and cache methods, because "it is all data access anyway". Explain why you would push back and how you would split it.
6. When would you keep an inheritance hierarchy instead of moving to composition? Give the trade-off in both directions.
7. You are reviewing an AI-generated class hierarchy for shapes and animals. Describe out loud the one question you ask for LSP and the one question you ask for ISP, and what you do with the answers.
