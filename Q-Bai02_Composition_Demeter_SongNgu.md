# Bài 2 — Composition > Inheritance · Fragile Base Class · Law of Demeter · Program-to-interface
### Bản song ngữ: tiếng Anh bám cấu trúc tiếng Việt

> **Cách dùng bài này:** đọc đoạn tiếng Việt trước, tự dịch trong đầu, rồi mới nhìn sang đoạn tiếng Anh để so. Sau khi so xong, **đọc to đoạn tiếng Anh hai lần** — lần một để nhìn chữ, lần hai để nghe chính mình. Phần bài nói ở cuối là bắt buộc; không nói ra thì bài này chỉ là bài đọc.

---

## Phần 1 — Bốn nguyên tắc giảm coupling dùng hằng ngày

**Tiếng Việt**

Bài 1 đã cho chúng ta một chiếc la bàn, đó là cặp coupling và cohesion. Bài này dạy bốn nguyên tắc giảm coupling mà chúng ta dùng hằng ngày, và cả bốn nguyên tắc đều là tiền đề để hiểu SOLID lẫn các pattern của GoF. Nguyên tắc thứ nhất là ưu tiên composition hơn inheritance. Nguyên tắc thứ hai là hiểu vì sao inheritance dễ tạo ra coupling chặt, qua vấn đề fragile base class. Nguyên tắc thứ ba là Law of Demeter, tức là nguyên tắc hiểu biết tối thiểu. Nguyên tắc thứ tư là "program to an interface", nghĩa là chúng ta phụ thuộc vào contract chứ không phụ thuộc vào class cụ thể.

**English (bám cấu trúc tiếng Việt)**

Lesson 1 gave us a compass, which is the pair of coupling and cohesion. This lesson teaches four principles for reducing coupling that we use every day, and all four principles are the groundwork for understanding both SOLID and the GoF patterns. The first principle is to favour composition over inheritance. The second principle is to understand why inheritance easily creates tight coupling, through the fragile base class problem. The third principle is the Law of Demeter, that is, the principle of least knowledge. The fourth principle is "program to an interface", which means that we depend on a contract and not on a concrete class.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| đã cho chúng ta một chiếc la bàn | gave us a compass |
| bốn nguyên tắc giảm coupling | four principles for reducing coupling |
| mà chúng ta dùng hằng ngày | that we use every day |
| đều là tiền đề để hiểu | are the groundwork for understanding |
| ưu tiên composition hơn inheritance | favour composition over inheritance |
| dễ tạo ra coupling chặt | easily creates tight coupling |
| qua vấn đề fragile base class | through the fragile base class problem |
| tức là nguyên tắc hiểu biết tối thiểu | that is, the principle of least knowledge |
| phụ thuộc vào contract chứ không phụ thuộc vào class cụ thể | depend on a contract and not on a concrete class |

**Thuật ngữ cần nhớ**

- ưu tiên A hơn B → **favour A over B**
- tiền đề, nền để đi tiếp → **groundwork**
- coupling chặt → **tight coupling**
- hiểu biết tối thiểu → **least knowledge**
- bản giao kèo giữa hai bên code → **contract**

---

## Phần 2 — Vì sao inheritance ghép chặt hơn composition

**Tiếng Việt**

Inheritance, viết trong code là `class B extends A`, tạo ra quan hệ "is-a" và đồng thời tạo ra coupling rất chặt. Lý do là class B không chỉ phụ thuộc vào contract của A, mà còn phụ thuộc vào cả cách A hiện thực bên trong. Vì vậy, khi ai đó đổi A, class B có thể vỡ một cách ngầm, nghĩa là vỡ mà không có lỗi biên dịch nào báo cho chúng ta biết. Composition thì ngược lại, nó tạo ra quan hệ "has-a", trong đó B chứa một thành phần và uỷ quyền công việc cho thành phần đó. Cách này linh hoạt hơn, vì chúng ta ghép hoặc đổi hành vi ngay lúc runtime, và chúng ta test được từng mảnh riêng biệt. Nếu chúng ta chọn inheritance chỉ để tái dùng vài dòng code, chúng ta đang trả một cái giá rất đắt, đó là buộc hai class vào nhau vĩnh viễn.

**English (bám cấu trúc tiếng Việt)**

Inheritance, written in code as `class B extends A`, creates an "is-a" relationship and at the same time creates very tight coupling. The reason is that class B does not only depend on the contract of A, but also depends on the way A is implemented inside. Therefore, when someone changes A, class B can break silently, which means it breaks without any compile error telling us about it. Composition is the opposite, it creates a "has-a" relationship, in which B holds a component and delegates the work to that component. This way is more flexible, because we combine or swap behaviour right at runtime, and we can test each piece separately. If we choose inheritance only to reuse a few lines of code, we are paying a very high price, which is tying the two classes together forever.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| viết trong code là | written in code as |
| và đồng thời tạo ra | and at the same time creates |
| không chỉ … mà còn | does not only … but also |
| cách A hiện thực bên trong | the way A is implemented inside |
| vỡ một cách ngầm | break silently |
| không có lỗi biên dịch nào báo cho chúng ta biết | without any compile error telling us about it |
| uỷ quyền công việc cho | delegates the work to |
| ghép hoặc đổi hành vi ngay lúc runtime | combine or swap behaviour right at runtime |
| test được từng mảnh riêng biệt | test each piece separately |
| trả một cái giá rất đắt | paying a very high price |
| buộc hai class vào nhau vĩnh viễn | tying the two classes together forever |

**Thuật ngữ cần nhớ**

- kế thừa → **inheritance**
- ghép thành phần → **composition**
- uỷ quyền → **delegate**
- vỡ ngầm, hỏng âm thầm → **break silently**
- lỗi biên dịch → **compile error**
- linh hoạt → **flexible**

---

## Phần 3 — Ba điều kiện để inheritance vẫn là lựa chọn đúng

**Tiếng Việt**

Chúng ta không nên bài trừ inheritance, bởi vì nguyên tắc ở đây là "ưu tiên", chứ không phải "cấm". Inheritance vẫn là lựa chọn đúng khi ba điều kiện cùng được thoả mãn. Điều kiện thứ nhất là quan hệ "is-a" phải thật, nghĩa là class con đúng là một loại của class cha trong ngôn ngữ nghiệp vụ, chứ không phải chỉ tình cờ dùng chung code. Điều kiện thứ hai là class con phải thoả nguyên tắc Liskov ở Bài 5, nghĩa là chúng ta thay class cha bằng class con ở mọi nơi mà chương trình vẫn chạy đúng. Điều kiện thứ ba là phân cấp phải ổn định, nghĩa là chúng ta không dự kiến phải thêm hoặc đổi các tầng kế thừa liên tục. Nếu một trong ba điều kiện không được thoả, chúng ta nên quay về composition, vì cái giá của việc gỡ kế thừa ra sau này cao hơn nhiều so với cái lợi trước mắt.

**English (bám cấu trúc tiếng Việt)**

We should not ban inheritance, because the principle here is "favour", and not "forbid". Inheritance is still the right choice when three conditions are satisfied together. The first condition is that the "is-a" relationship must be real, which means that the child class truly is a kind of the parent class in the business language, and not just a case of sharing code by accident. The second condition is that the child class must satisfy the Liskov principle in Lesson 5, which means that we replace the parent class with the child class everywhere and the program still runs correctly. The third condition is that the hierarchy must be stable, which means that we do not expect to add or change the inheritance layers all the time. If one of the three conditions is not satisfied, we should go back to composition, because the price of pulling the inheritance apart later is much higher than the benefit we get right now.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| chúng ta không nên bài trừ | we should not ban |
| "ưu tiên", chứ không phải "cấm" | "favour", and not "forbid" |
| khi ba điều kiện cùng được thoả mãn | when three conditions are satisfied together |
| đúng là một loại của | truly is a kind of |
| chỉ tình cờ dùng chung code | just a case of sharing code by accident |
| thay A bằng B ở mọi nơi | replace A with B everywhere |
| phân cấp phải ổn định | the hierarchy must be stable |
| không dự kiến phải thêm hoặc đổi | do not expect to add or change |
| gỡ kế thừa ra sau này | pulling the inheritance apart later |
| cái lợi trước mắt | the benefit we get right now |

**Thuật ngữ cần nhớ**

- cấm → **forbid**
- điều kiện được thoả → **a condition is satisfied**
- class con / class cha → **child class** / **parent class**
- phân cấp → **hierarchy**
- tầng kế thừa → **inheritance layer**

---

## Phần 4 — Fragile base class: con vỡ vì cha đổi

**Tiếng Việt**

Fragile base class là tình huống class con vỡ chỉ vì class cha đổi chi tiết bên trong, dù chúng ta không hề đụng đến class con. Ví dụ kinh điển là một class đếm số phần tử được thêm vào, class này kế thừa `ArrayList` và override method `add` để tăng bộ đếm. Vấn đề nằm ở chỗ method `addAll` của class cha gọi nội bộ vào `add`, cho nên mỗi phần tử bị đếm hai lần và tổng số đếm sai gấp đôi. Điều nguy hiểm hơn nằm ở chiều ngược lại: một ngày nào đó class cha đổi cách `addAll` chạy bên trong, và class con của chúng ta vỡ dù không ai sửa một dòng nào trong nó. Bài học rút ra là inheritance để lộ phần bên trong của class cha theo kiểu hộp trắng, còn composition giữ class kia như một hộp đen. Vì vậy, khi chúng ta không kiểm soát được class cha, ví dụ khi nó nằm trong một thư viện của bên thứ ba, chúng ta gần như luôn nên chọn composition.

**English (bám cấu trúc tiếng Việt)**

The fragile base class is the situation where the child class breaks only because the parent class changes its internal details, although we never touch the child class. The classic example is a class that counts the number of elements added, this class inherits `ArrayList` and overrides the `add` method to increase the counter. The problem lies in the fact that the `addAll` method of the parent class calls `add` internally, therefore each element is counted twice and the total count is wrong by a factor of two. The more dangerous thing lies in the opposite direction: one day the parent class changes the way `addAll` runs inside, and our child class breaks although nobody edits a single line in it. The lesson we draw is that inheritance exposes the inside of the parent class in a white-box way, while composition keeps the other class as a black box. Therefore, when we do not control the parent class, for example when it sits in a third-party library, we should almost always choose composition.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| là tình huống class con vỡ chỉ vì | is the situation where the child class breaks only because |
| dù chúng ta không hề đụng đến | although we never touch |
| ví dụ kinh điển là | the classic example is |
| override method `add` để tăng bộ đếm | overrides the `add` method to increase the counter |
| vấn đề nằm ở chỗ | the problem lies in the fact that |
| gọi nội bộ vào | calls … internally |
| tổng số đếm sai gấp đôi | the total count is wrong by a factor of two |
| điều nguy hiểm hơn nằm ở chiều ngược lại | the more dangerous thing lies in the opposite direction |
| dù không ai sửa một dòng nào trong nó | although nobody edits a single line in it |
| bài học rút ra là | the lesson we draw is that |
| khi chúng ta không kiểm soát được | when we do not control |
| thư viện của bên thứ ba | a third-party library |

**Thuật ngữ cần nhớ**

- lớp cha mong manh → **fragile base class**
- ghi đè → **override**
- bộ đếm → **counter**
- hộp trắng / hộp đen → **white box** / **black box**
- thư viện bên thứ ba → **third-party library**

---

## Phần 5 — Law of Demeter và vụ trật đường ray

**Tiếng Việt**

Law of Demeter còn được gọi là nguyên tắc hiểu biết tối thiểu, và cách nói dễ nhớ nhất của nó là "chỉ nói chuyện với bạn bè trực tiếp". Một câu vi phạm điển hình là `order.getCustomer().getAddress().getCity()`, và người ta gọi chuỗi gọi kiểu này là một vụ trật đường ray. Vấn đề của nó là câu lệnh đó phơi ra cấu trúc nội bộ nhiều cấp, cho nên đoạn code gọi bị ghép chặt với cả chuỗi object phía sau. Khi ai đó đổi cấu trúc của `Address`, mọi chỗ với tay vào sâu như vậy đều vỡ cùng một lúc. Chúng ta nên nhớ rằng dấu hiệu nhận biết rất đơn giản, đó là một chuỗi dấu chấm dài quá một cấp. Đây cũng là lỗi mà công cụ AI hay sinh ra nhất, vì AI viết theo cách ngắn nhất để lấy được dữ liệu chứ không quan tâm đến coupling.

**English (bám cấu trúc tiếng Việt)**

The Law of Demeter is also called the principle of least knowledge, and the way of saying it that is easiest to remember is "only talk to your direct friends". A typical violating line is `order.getCustomer().getAddress().getCity()`, and people call this kind of call chain a train wreck. Its problem is that the statement exposes the internal structure over several levels, therefore the calling code is tightly coupled to the whole chain of objects behind it. When someone changes the structure of `Address`, every place that reaches in that deeply breaks at the same time. We should remember that the warning sign is very simple, it is a chain of dots that is longer than one level. This is also the mistake that AI tools generate most often, because AI writes in the shortest way to get the data and does not care about coupling.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| còn được gọi là | is also called |
| cách nói dễ nhớ nhất của nó | the way of saying it that is easiest to remember |
| chỉ nói chuyện với bạn bè trực tiếp | only talk to your direct friends |
| một câu vi phạm điển hình | a typical violating line |
| người ta gọi chuỗi gọi kiểu này là | people call this kind of call chain |
| một vụ trật đường ray | a train wreck |
| phơi ra cấu trúc nội bộ nhiều cấp | exposes the internal structure over several levels |
| mọi chỗ với tay vào sâu như vậy | every place that reaches in that deeply |
| dấu hiệu nhận biết rất đơn giản | the warning sign is very simple |
| một chuỗi dấu chấm dài quá một cấp | a chain of dots that is longer than one level |
| viết theo cách ngắn nhất để lấy được dữ liệu | writes in the shortest way to get the data |

**Thuật ngữ cần nhớ**

- vi phạm → **violate** / **violation**
- chuỗi gọi → **call chain**
- vụ trật đường ray → **train wreck**
- với tay vào sâu → **reach in deeply**
- dấu hiệu nhận biết → **warning sign**

---

## Phần 6 — "Tell, Don't Ask": đẩy hành vi về nơi có dữ liệu

**Tiếng Việt**

Hướng sửa cho vụ trật đường ray là "hãy ra lệnh, đừng hỏi", tức là chúng ta thêm một method ở đúng tầng thay vì với tay vào lấy dữ liệu. Thay cho `order.getCustomer().getAddress().getCity()`, chúng ta viết `order.shippingCity()`, và bên trong thì `Order` tự hỏi `Customer` của nó. `Customer` lại bọc `Address` của nó bằng một method nhỏ, cho nên không ai bên ngoài cần biết `Address` có hình dạng gì. Nhờ đó, khi cấu trúc bên trong thay đổi, chúng ta chỉ sửa một chỗ duy nhất là chính class sở hữu dữ liệu đó. Nguyên tắc chung là chúng ta nên đẩy hành vi về nơi có dữ liệu, chứ không kéo dữ liệu về nơi có hành vi. Nếu chúng ta làm ngược lại trong thời gian dài, các class sẽ dần trở thành những túi dữ liệu rỗng và toàn bộ logic sẽ dồn vào tầng service, và đó chính là mô hình thiếu máu ở Bài 12.

**English (bám cấu trúc tiếng Việt)**

The fix for the train wreck is "tell, don't ask", that is, we add a method at the right level instead of reaching in to take the data. Instead of `order.getCustomer().getAddress().getCity()`, we write `order.shippingCity()`, and inside it `Order` asks its own `Customer`. `Customer` in turn wraps its own `Address` with a small method, therefore nobody outside needs to know what shape `Address` has. Thanks to that, when the internal structure changes, we edit only one single place, which is the very class that owns that data. The general principle is that we should push behaviour to where the data is, and not pull the data to where the behaviour is. If we do the opposite for a long time, the classes will slowly turn into empty bags of data and all the logic will pile up in the service layer, and that is exactly the anemic model in Lesson 12.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| hướng sửa cho | the fix for |
| hãy ra lệnh, đừng hỏi | tell, don't ask |
| thêm một method ở đúng tầng | add a method at the right level |
| thay vì với tay vào lấy dữ liệu | instead of reaching in to take the data |
| `Customer` lại bọc `Address` của nó | `Customer` in turn wraps its own `Address` |
| không ai bên ngoài cần biết | nobody outside needs to know |
| `Address` có hình dạng gì | what shape `Address` has |
| chỉ sửa một chỗ duy nhất | edit only one single place |
| chính class sở hữu dữ liệu đó | the very class that owns that data |
| đẩy hành vi về nơi có dữ liệu | push behaviour to where the data is |
| những túi dữ liệu rỗng | empty bags of data |
| toàn bộ logic sẽ dồn vào tầng service | all the logic will pile up in the service layer |

**Thuật ngữ cần nhớ**

- bọc lại → **wrap**
- sở hữu dữ liệu → **own the data**
- túi dữ liệu → **bag of data**
- dồn vào, chất đống → **pile up**
- mô hình thiếu máu → **anemic model**

---

## Phần 7 — "Program to an interface" không phải là từ khoá `interface`

**Tiếng Việt**

Nguyên tắc thứ tư nói rằng chúng ta hãy lập trình hướng tới một interface, chứ không hướng tới một implementation. Ở đây có một cái bẫy mà rất nhiều người mắc phải, đó là nguyên tắc này không đồng nghĩa với từ khoá `interface` trong ngôn ngữ. Ý thật sự của nó là chúng ta phụ thuộc vào một contract hoặc một kiểu trừu tượng, chứ không phụ thuộc vào một kiểu cụ thể. Chúng ta có thể hiện thực contract đó bằng một abstract class, hoặc thậm chí bằng duck typing trong JavaScript và TypeScript, và nguyên tắc vẫn được giữ nguyên. Mục tiêu cuối cùng là tách client ra khỏi class cụ thể, để chúng ta thay implementation mà không phải sửa client. Lợi ích thấy ngay là chúng ta có một khe để test, bởi vì trong unit test chúng ta truyền vào một object giả thoả contract, và bài test chạy được mà không cần gọi mạng thật.

**English (bám cấu trúc tiếng Việt)**

The fourth principle says that we should program towards an interface, and not towards an implementation. Here there is a trap that many people fall into, which is that this principle does not mean the `interface` keyword in the language. Its real meaning is that we depend on a contract or an abstract type, and not on a concrete type. We can implement that contract with an abstract class, or even with duck typing in JavaScript and TypeScript, and the principle is still kept. The final goal is to separate the client from the concrete class, so that we swap the implementation without having to edit the client. The benefit we see immediately is that we have a seam for testing, because in a unit test we pass in a fake object that satisfies the contract, and the test runs without needing a real network call.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| lập trình hướng tới | program towards |
| một cái bẫy mà rất nhiều người mắc phải | a trap that many people fall into |
| không đồng nghĩa với từ khoá | does not mean the keyword |
| ý thật sự của nó là | its real meaning is that |
| một kiểu trừu tượng | an abstract type |
| và nguyên tắc vẫn được giữ nguyên | and the principle is still kept |
| tách client ra khỏi class cụ thể | separate the client from the concrete class |
| lợi ích thấy ngay là | the benefit we see immediately is |
| một khe để test | a seam for testing |
| một object giả thoả contract | a fake object that satisfies the contract |
| mà không cần gọi mạng thật | without needing a real network call |

**Thuật ngữ cần nhớ**

- từ khoá → **keyword**
- kiểu trừu tượng → **abstract type**
- khe để test, điểm chèn giả → **test seam**
- object giả → **fake** / **stub**
- bên gọi, phía dùng → **the client**

---

## Phần 8 — Ví dụ thật và danh sách soát code

**Tiếng Việt**

Chúng ta hãy đặt bốn nguyên tắc trên vào một ví dụ cụ thể là việc ghi log. Thay vì viết một chuỗi kế thừa kiểu `JsonLogger` kế thừa `Logger` kế thừa `BaseWriter`, chúng ta dùng composition và truyền một `logger` vào service qua constructor. Đối tượng `logger` đó chỉ cần thoả interface `Logger`, cho nên việc đổi từ logger ra console sang logger ra file chỉ là đổi thành phần được inject vào. Ở cấp thiết kế ngôn ngữ, Go đã cố tình bỏ hẳn inheritance và chỉ giữ composition cùng interface ngầm, và đó là một minh chứng mạnh cho nguyên tắc này. React cũng khuyến nghị chúng ta dùng composition thay vì kế thừa component. Khi hai hệ sinh thái lớn cùng chọn một hướng, chúng ta nên coi đó là mặc định hợp lý chứ không phải một sở thích cá nhân.

Cuối cùng, chúng ta nên biến bài này thành một danh sách soát code, đặc biệt khi code do AI sinh ra. Câu hỏi thứ nhất là quan hệ kế thừa ở đây có phải "is-a" thật hay không, hay nó chỉ là một cách tái dùng code cho nhanh. Câu hỏi thứ hai là trong file này có chuỗi dấu chấm nào dài quá một cấp hay không. Câu hỏi thứ ba là constructor đang nhận vào một abstraction hay đang tự tạo ra một class cụ thể bên trong. Nếu câu trả lời cho ba câu hỏi này đều xấu, code vẫn chạy được, nhưng chúng ta sẽ trả giá ở lần thay đổi tiếp theo.

**English (bám cấu trúc tiếng Việt)**

Let us put the four principles above into one concrete example, which is logging. Instead of writing an inheritance chain like `JsonLogger` extends `Logger` extends `BaseWriter`, we use composition and pass a `logger` into the service through the constructor. That `logger` object only needs to satisfy the `Logger` interface, therefore switching from a console logger to a file logger is only a matter of changing the component that is injected. At the level of language design, Go deliberately dropped inheritance completely and kept only composition together with implicit interfaces, and that is strong evidence for this principle. React also recommends that we use composition instead of component inheritance. When two big ecosystems choose the same direction, we should treat that as a reasonable default and not as a personal taste.

Finally, we should turn this lesson into a code review checklist, especially when the code is generated by AI. The first question is whether the inheritance relationship here is a real "is-a", or whether it is only a quick way to reuse code. The second question is whether there is any chain of dots in this file that is longer than one level. The third question is whether the constructor takes in an abstraction or creates a concrete class inside itself. If the answers to these three questions are all bad, the code still runs, but we will pay the price at the next change.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| một chuỗi kế thừa kiểu | an inheritance chain like |
| truyền một `logger` vào service qua constructor | pass a `logger` into the service through the constructor |
| chỉ là đổi thành phần được inject vào | is only a matter of changing the component that is injected |
| ở cấp thiết kế ngôn ngữ | at the level of language design |
| đã cố tình bỏ hẳn | deliberately dropped … completely |
| interface ngầm | implicit interfaces |
| một minh chứng mạnh cho | strong evidence for |
| coi đó là mặc định hợp lý | treat that as a reasonable default |
| chứ không phải một sở thích cá nhân | and not as a personal taste |
| biến bài này thành một danh sách soát code | turn this lesson into a code review checklist |
| chỉ là một cách tái dùng code cho nhanh | only a quick way to reuse code |
| chúng ta sẽ trả giá ở lần thay đổi tiếp theo | we will pay the price at the next change |

**Thuật ngữ cần nhớ**

- chuỗi kế thừa → **inheritance chain**
- tiêm phụ thuộc vào → **inject**
- ngầm định → **implicit**
- hệ sinh thái → **ecosystem**
- danh sách soát code → **code review checklist**
- trả giá → **pay the price**

---

## Mô hình ghi nhớ

**Tiếng Việt**

Chúng ta mặc định chọn "has-a" thay vì "is-a", chúng ta chỉ nói chuyện với hàng xóm trực tiếp, và chúng ta phụ thuộc vào contract chứ không phụ thuộc vào class. Ba thói quen đó là cách rẻ nhất để giữ coupling lỏng trong công việc hằng ngày.

**English (bám cấu trúc tiếng Việt)**

We choose "has-a" by default instead of "is-a", we only talk to our direct neighbours, and we depend on a contract and not on a class. Those three habits are the cheapest way to keep coupling loose in our daily work.

---

## Bảng thuật ngữ tổng hợp

| Tiếng Việt | Tiếng Anh | Ghi chú phát âm |
|---|---|---|
| kế thừa | inheritance | trọng âm âm hai: in-HER-i-tance, không đọc "IN-heritance" |
| ghép thành phần | composition | trọng âm áp chót: com-po-SI-tion |
| ưu tiên A hơn B | favour A over B | Anh viết *favour*, Mỹ viết *favor*; đọc /ˈfeɪvə/ |
| cấm | forbid | trọng âm âm hai: for-BID |
| coupling chặt | tight coupling | |
| tiền đề, nền để đi tiếp | groundwork | |
| bản giao kèo giữa hai bên code | contract | danh từ trọng âm đầu: CON-tract |
| uỷ quyền | delegate | động từ /ˈdelɪɡeɪt/ — DEL-e-gate, đuôi đọc như "gate" |
| linh hoạt | flexible | |
| vỡ ngầm | break silently | |
| lỗi biên dịch | compile error | *compile* /kəmˈpaɪl/ — trọng âm âm hai: com-PILE |
| class con / class cha | child class / parent class | |
| phân cấp | hierarchy | /ˈhaɪərɑːki/ — trọng âm đầu: HI-er-ar-chy, người Việt hay đọc sai thành "hi-ER-ar-chy" |
| điều kiện được thoả | a condition is satisfied | |
| lớp cha mong manh | fragile base class | *fragile* — người Anh đọc /ˈfrædʒaɪl/ "FRAJ-ai-l", người Mỹ đọc /ˈfrædʒəl/ |
| ghi đè | override | trọng âm cuối: o-ver-RIDE |
| bộ đếm | counter | |
| hộp trắng / hộp đen | white box / black box | |
| thư viện bên thứ ba | third-party library | *third* có âm **th**; *library* /ˈlaɪbrəri/ — "LAI-brơ-ri", không đọc "li-brơ-ry" |
| nguyên tắc hiểu biết tối thiểu | the principle of least knowledge | *least* /liːst/ — bật rõ đuôi **-st**; *knowledge* có **k** câm, đọc "NOL-ij" |
| vi phạm | violate / violation | *violate* trọng âm đầu: VI-o-late |
| chuỗi gọi | call chain | |
| vụ trật đường ray | train wreck | *wreck* /rek/ — chữ **w** câm, đọc là "rếch" |
| với tay vào sâu | reach in deeply | |
| dấu hiệu nhận biết | warning sign | |
| hãy ra lệnh, đừng hỏi | tell, don't ask | |
| bọc lại | wrap | /ræp/ — chữ **w** câm, đọc giống "rap" |
| sở hữu dữ liệu | own the data | |
| túi dữ liệu | bag of data | |
| dồn vào, chất đống | pile up | |
| mô hình thiếu máu | anemic model | /əˈniːmɪk/ — a-NEE-mic, trọng âm âm giữa |
| từ khoá | keyword | |
| kiểu trừu tượng | abstract type | *abstract* (tính từ) trọng âm đầu: AB-stract; cụm **-bstr-** phải đọc chậm |
| kiểu cụ thể | concrete type | trọng âm đầu: CON-crete |
| khe để test | test seam | *seam* /siːm/ — đọc giống "seem" |
| object giả | fake / stub | |
| bên gọi, phía dùng | the client | |
| hàm khởi tạo | constructor | trọng âm âm hai: con-STRUC-tor |
| tiêm phụ thuộc vào | inject | trọng âm âm hai: in-JECT |
| ngầm định | implicit | trọng âm âm hai: im-PLIC-it |
| hệ sinh thái | ecosystem | /ˈiːkəʊsɪstəm/ — trọng âm đầu: E-co-sys-tem |
| danh sách soát code | code review checklist | |
| trả giá | pay the price | |

---

## Bài nói

> **Cách làm:** ghi âm lại giọng của mình, mỗi câu nói liên tục **60–90 giây**, **không nhìn tài liệu** khi nói. Mỗi bài nói phải dùng ít nhất **6 thuật ngữ** trong bảng ở trên. Nghe lại bản ghi âm một lần, ghi ra hai chỗ mình bị vấp, rồi nói lại lần hai.

1. Explain to a junior developer why we favour composition over inheritance, and give one concrete case where inheritance is still the right choice.
2. A colleague says: "Composition is always better than inheritance, so we should never use `extends` in this codebase." Explain what is wrong with that statement and what rule you would give the team instead.
3. Describe the fragile base class problem step by step, using the counting-list example, and say what the child class author could not have predicted.
4. Someone on your team wants to keep `order.getCustomer().getAddress().getCity()` because "it works and it is only one line". Explain why you would push back, and show the refactoring you would ask for.
5. Explain what "program to an interface, not an implementation" really means, and why it is not the same as using the `interface` keyword.
6. Describe what happens to your unit tests when a service creates its own `HttpClient` inside the constructor instead of receiving it, and how a test seam changes that picture.
7. You are reviewing a class that an AI tool generated. Describe out loud the three questions you would ask about inheritance, call chains, and constructor dependencies, and what you would do with the answers.
