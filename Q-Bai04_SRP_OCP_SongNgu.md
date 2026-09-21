# Bài 4 — SOLID (1): Single Responsibility & Open/Closed
### Bản song ngữ: tiếng Anh bám cấu trúc tiếng Việt

> **Cách dùng bài này:** đọc đoạn tiếng Việt trước, tự dịch trong đầu, rồi mới nhìn sang đoạn tiếng Anh để so. Sau khi so xong, **đọc to đoạn tiếng Anh hai lần** — lần một để nhìn chữ, lần hai để nghe chính mình. Phần bài nói ở cuối là bắt buộc; không nói ra thì bài này chỉ là bài đọc.

---

## Phần 1 — SOLID là cách hệ thống hoá la bàn của Bài 1

**Tiếng Việt**

SOLID, do Robert C. Martin hệ thống hoá, chính là cách chúng ta biến chiếc la bàn ở Bài 1 thành năm quy tắc dùng được hằng ngày. Trong năm chữ đó, SRP và ISP kéo cohesion lên, còn OCP và DIP kéo coupling xuống. Bài này gói gọn hai chữ đầu là SRP và OCP, bởi vì đây là hai chữ mà chúng ta vi phạm nhiều nhất trong công việc thật. Bài 5 sẽ lo LSP và ISP, còn Bài 6 sẽ lo DIP cùng những đánh đổi khi chúng ta áp dụng cả bộ. Nếu chúng ta nắm chắc hai chữ trong bài này, chúng ta đã xử lý được phần lớn câu hỏi thiết kế ở vòng phỏng vấn đầu tiên.

**English (bám cấu trúc tiếng Việt)**

SOLID, systematised by Robert C. Martin, is exactly the way we turn the compass from Lesson 1 into five rules that we can use every day. Among those five letters, SRP and ISP pull cohesion up, while OCP and DIP push coupling down. This lesson covers the first two letters, which are SRP and OCP, because these are the two letters that we violate the most in real work. Lesson 5 will take care of LSP and ISP, while Lesson 6 will take care of DIP together with the trade-offs when we apply the whole set. If we have a firm grip on the two letters in this lesson, we have already handled most of the design questions in the first interview round.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| do Robert C. Martin hệ thống hoá | systematised by Robert C. Martin |
| biến chiếc la bàn … thành năm quy tắc | turn the compass … into five rules |
| trong năm chữ đó | among those five letters |
| bài này gói gọn hai chữ đầu | this lesson covers the first two letters |
| mà chúng ta vi phạm nhiều nhất | that we violate the most |
| sẽ lo LSP và ISP | will take care of LSP and ISP |
| khi chúng ta áp dụng cả bộ | when we apply the whole set |
| nếu chúng ta nắm chắc | if we have a firm grip on |
| ở vòng phỏng vấn đầu tiên | in the first interview round |

**Thuật ngữ cần nhớ**

- hệ thống hoá → **systematise**
- vi phạm → **violate**
- đánh đổi → **trade-off**
- nắm chắc → **have a firm grip on**
- vòng phỏng vấn → **interview round**

---

## Phần 2 — Gọi đúng tên năm chữ SOLID

**Tiếng Việt**

Trước hết chúng ta phải gọi đúng tên năm chữ, bởi vì đây là câu hỏi mở đầu rất hay gặp và trả lời sai thì chúng ta mất điểm ngay. Chữ S là Single Responsibility, tức là nguyên tắc một trách nhiệm. Chữ O là Open/Closed, tức là nguyên tắc mở để mở rộng và đóng để sửa đổi. Chữ L là Liskov Substitution, tức là nguyên tắc thay thế Liskov, còn chữ I là Interface Segregation, tức là nguyên tắc tách nhỏ interface. Chữ D là Dependency Inversion, tức là nguyên tắc đảo ngược phụ thuộc. Khi trả lời phỏng vấn, chúng ta nên đọc tên đầy đủ kèm một câu giải nghĩa cho mỗi chữ, chứ chúng ta không nên chỉ đọc trơn năm chữ cái.

**English (bám cấu trúc tiếng Việt)**

First of all we have to name the five letters correctly, because this is a very common opening question and if we answer it wrongly we lose points immediately. The letter S is Single Responsibility, that is, the principle of one responsibility. The letter O is Open/Closed, that is, the principle of being open for extension and closed for modification. The letter L is Liskov Substitution, that is, the Liskov substitution principle, while the letter I is Interface Segregation, that is, the principle of splitting interfaces into small ones. The letter D is Dependency Inversion, that is, the principle of inverting dependencies. When we answer in an interview, we should say the full name together with one explaining sentence for each letter, and we should not just recite the five letters bare.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| trước hết chúng ta phải gọi đúng tên | first of all we have to name … correctly |
| câu hỏi mở đầu rất hay gặp | a very common opening question |
| chúng ta mất điểm ngay | we lose points immediately |
| mở để mở rộng và đóng để sửa đổi | open for extension and closed for modification |
| nguyên tắc tách nhỏ interface | the principle of splitting interfaces into small ones |
| nguyên tắc đảo ngược phụ thuộc | the principle of inverting dependencies |
| kèm một câu giải nghĩa cho mỗi chữ | together with one explaining sentence for each letter |
| chỉ đọc trơn năm chữ cái | just recite the five letters bare |

**Thuật ngữ cần nhớ**

- trách nhiệm → **responsibility**
- mở rộng → **extension**
- sửa đổi → **modification**
- tách nhỏ, phân tách → **segregation**
- đảo ngược → **inversion**

---

## Phần 3 — Phát biểu chuẩn của SRP: một lý do để thay đổi

**Tiếng Việt**

SRP không phải là "một class chỉ làm một việc", và đây là chỗ mà rất nhiều ứng viên trả lời hụt. Phát biểu chuẩn là một class chỉ nên có một lý do để thay đổi, nghĩa là nó chỉ nên phục vụ một actor hay một nhóm người dùng nghiệp vụ. Lý do thay đổi không phải là số lượng method, cho nên một class có mười method vẫn thoả SRP nếu cả mười method cùng phục vụ một actor. Cách kiểm tra thực tế là chúng ta hỏi ai sẽ là người yêu cầu sửa class này, và nếu câu trả lời gồm ba nhóm người khác nhau thì class đó đang vi phạm. SRP thật ra chính là cohesion ở cấp class, và nó là vũ khí chính của chúng ta để chống God Object. Nếu chúng ta hiểu SRP theo kiểu "một việc", chúng ta sẽ tách máy móc thành hàng chục class vụn, và lúc đó chúng ta lại rơi vào over-engineering như Bài 3 đã cảnh báo.

**English (bám cấu trúc tiếng Việt)**

SRP is not "a class only does one thing", and this is the point where very many candidates answer short. The standard statement is that a class should only have one reason to change, which means that it should only serve one actor or one group of business users. The reason to change is not the number of methods, so a class with ten methods still satisfies SRP if all ten methods serve one actor. The practical way to check is that we ask who will be the person asking for a change in this class, and if the answer includes three different groups of people then that class is violating the principle. SRP is in fact cohesion at the class level, and it is our main weapon against the God Object. If we understand SRP as "one thing", we will split mechanically into dozens of tiny classes, and at that point we fall into over-engineering again as Lesson 3 warned.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| chỗ mà rất nhiều ứng viên trả lời hụt | the point where very many candidates answer short |
| phát biểu chuẩn là | the standard statement is that |
| một lý do để thay đổi | one reason to change |
| một nhóm người dùng nghiệp vụ | one group of business users |
| vẫn thoả SRP nếu | still satisfies SRP if |
| ai sẽ là người yêu cầu sửa class này | who will be the person asking for a change in this class |
| class đó đang vi phạm | that class is violating the principle |
| vũ khí chính của chúng ta để chống | our main weapon against |
| tách máy móc thành hàng chục class vụn | split mechanically into dozens of tiny classes |
| như Bài 3 đã cảnh báo | as Lesson 3 warned |

**Thuật ngữ cần nhớ**

- bên liên quan, người đặt yêu cầu → **actor** / **stakeholder**
- ứng viên → **candidate**
- thoả mãn (nguyên tắc) → **satisfy**
- class ôm mọi thứ → **God Object**
- vụn, nhỏ li ti → **tiny**

---

## Phần 4 — Ví dụ vi phạm SRP kinh điển và cách tách

**Tiếng Việt**

Chúng ta hãy xem ví dụ vi phạm kinh điển nhất, đó là class `UserService`. Class này vừa validate dữ liệu theo luật nghiệp vụ, vừa lưu người dùng xuống database, vừa gửi email chào mừng. Ba việc đó thay đổi vì ba lý do hoàn toàn khác nhau, bởi vì luật nghiệp vụ đổi khi phòng sản phẩm đổi ý, database đổi khi đội hạ tầng đổi công nghệ, còn phần mail đổi khi công ty đổi nhà cung cấp. Cách sửa là chúng ta tách thành `UserValidator`, `UserRepository` và `EmailSender`, rồi chúng ta để `UserService` chỉ đóng vai điều phối. Lợi ích thấy ngay là chúng ta test và bảo trì từng phần một cách độc lập, và việc đổi nhà cung cấp mail không đụng đến logic validate. Nếu chúng ta để nguyên như cũ, thì mỗi lần đổi nhà cung cấp mail chúng ta lại phải chạy lại toàn bộ test của phần nghiệp vụ, và rủi ro làm hỏng thứ đang chạy tăng lên một cách vô lý.

**English (bám cấu trúc tiếng Việt)**

Let us look at the most classic violation example, which is the `UserService` class. This class validates data according to business rules, saves the user down to the database, and sends a welcome email at the same time. Those three jobs change for three completely different reasons, because the business rules change when the product department changes its mind, the database changes when the infrastructure team changes technology, while the mail part changes when the company changes provider. The fix is that we split it into `UserValidator`, `UserRepository` and `EmailSender`, and then we let `UserService` play only the coordinating role. The immediate benefit is that we test and maintain each part independently, and changing the mail provider does not touch the validation logic. If we leave it as it was, then every time we change the mail provider we have to run the whole test suite of the business part again, and the risk of breaking something that is running goes up for no good reason.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| ví dụ vi phạm kinh điển nhất | the most classic violation example |
| lưu người dùng xuống database | saves the user down to the database |
| khi phòng sản phẩm đổi ý | when the product department changes its mind |
| khi đội hạ tầng đổi công nghệ | when the infrastructure team changes technology |
| chỉ đóng vai điều phối | play only the coordinating role |
| test và bảo trì từng phần một cách độc lập | test and maintain each part independently |
| không đụng đến logic validate | does not touch the validation logic |
| nếu chúng ta để nguyên như cũ | if we leave it as it was |
| chạy lại toàn bộ test của phần nghiệp vụ | run the whole test suite of the business part again |
| tăng lên một cách vô lý | goes up for no good reason |

**Thuật ngữ cần nhớ**

- lưu trữ lâu dài → **persistence**
- kho chứa dữ liệu → **repository**
- điều phối → **coordinate** / **orchestrate**
- hạ tầng → **infrastructure**
- bộ test → **test suite**
- bảo trì → **maintain**

---

## Phần 5 — OCP: thêm hành vi mà không cưa lại code đang chạy

**Tiếng Việt**

OCP nói rằng một module nên mở để mở rộng nhưng đóng để sửa đổi. Nói cụ thể hơn, chúng ta thêm hành vi mới mà chúng ta không phải sửa lại phần code cũ đang chạy ổn định. Chúng ta đạt được điều đó nhờ abstraction và polymorphism, ví dụ nhờ một interface, một Strategy, hoặc một bảng ánh xạ từ loại sang hàm xử lý. Lý do nguyên tắc này quan trọng rất thực tế, bởi vì sửa code cũ luôn mang theo rủi ro regression lên những thứ đang chạy tốt. Thêm một class mới thì an toàn hơn nhiều, vì phần code cũ không hề bị đụng đến nên nó không thể vỡ theo cách mới. Trong một hệ thống có hàng nghìn bài test, khác biệt giữa hai cách làm này chính là khác biệt giữa một lần release yên tâm và một đêm trực sự cố.

**English (bám cấu trúc tiếng Việt)**

OCP says that a module should be open for extension but closed for modification. To say it more concretely, we add new behaviour and we do not have to edit the old code that is running stably. We achieve that thanks to abstraction and polymorphism, for example thanks to an interface, a Strategy, or a map from the type to the handler function. The reason this principle matters is very practical, because editing old code always carries the risk of a regression on the things that are running well. Adding a new class is much safer, because the old code is not touched at all so it cannot break in a new way. In a system with thousands of tests, the difference between these two ways of working is exactly the difference between a calm release and a night on call fixing an incident.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| mở để mở rộng nhưng đóng để sửa đổi | open for extension but closed for modification |
| nói cụ thể hơn | to say it more concretely |
| phần code cũ đang chạy ổn định | the old code that is running stably |
| một bảng ánh xạ từ loại sang hàm xử lý | a map from the type to the handler function |
| luôn mang theo rủi ro regression | always carries the risk of a regression |
| không hề bị đụng đến | is not touched at all |
| nó không thể vỡ theo cách mới | it cannot break in a new way |
| khác biệt giữa một lần release yên tâm | the difference between a calm release |
| một đêm trực sự cố | a night on call fixing an incident |

**Thuật ngữ cần nhớ**

- đa hình → **polymorphism**
- lỗi hồi quy (làm hỏng thứ đang chạy) → **regression**
- ổn định → **stably** / **stable**
- trực sự cố → **on call**
- sự cố → **incident**

---

## Phần 6 — Mùi vi phạm OCP: rẽ nhánh theo loại

**Tiếng Việt**

Dấu hiệu vi phạm OCP dễ nhận nhất là một chuỗi điều kiện dạng nếu loại là A thì làm thế này, còn nếu loại là B thì làm thế kia. Vấn đề nằm ở chỗ mỗi khi xuất hiện một loại mới, chúng ta lại phải mở hàm cũ ra và sửa nó. Người ta gọi hình dạng này là switch-on-type, và nó báo hiệu rằng thiết kế đang thiếu một abstraction. Cách trị là chúng ta chuyển sang polymorphism, ví dụ mỗi loại giá trở thành một class riêng biết tự trả lời rằng nó áp dụng cho loại nào và nó tính ra giá bao nhiêu. Một cách trị nhẹ hơn là chúng ta dùng một bảng ánh xạ từ tên loại sang hàm xử lý, và khi đó việc thêm một loại mới chỉ là thêm một dòng đăng ký. Sau khi chuyển xong, class điều phối chỉ còn việc tìm ra luật phù hợp rồi gọi luật đó, và bản thân class điều phối cũng thoả SRP vì nó không còn chứa từng luật cụ thể nữa.

**English (bám cấu trúc tiếng Việt)**

The easiest sign of an OCP violation to spot is a chain of conditions in the form of if the type is A then do this, else if the type is B then do that. The problem lies in the fact that every time a new type appears, we have to open the old function again and edit it. People call this shape switch-on-type, and it signals that the design is missing an abstraction. The cure is that we move to polymorphism, for example each price type becomes its own class that knows how to answer which type it applies to and what price it computes. A lighter cure is that we use a map from the type name to the handler function, and then adding a new type is only adding one registration line. After the move is finished, the coordinating class only has to find the matching rule and then call that rule, and the coordinating class itself also satisfies SRP because it no longer holds each concrete rule.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| dấu hiệu … dễ nhận nhất | the easiest sign … to spot |
| một chuỗi điều kiện dạng | a chain of conditions in the form of |
| vấn đề nằm ở chỗ | the problem lies in the fact that |
| mở hàm cũ ra và sửa nó | open the old function again and edit it |
| nó báo hiệu rằng thiết kế đang thiếu | it signals that the design is missing |
| biết tự trả lời rằng nó áp dụng cho loại nào | knows how to answer which type it applies to |
| một cách trị nhẹ hơn | a lighter cure |
| chỉ là thêm một dòng đăng ký | is only adding one registration line |
| tìm ra luật phù hợp rồi gọi luật đó | find the matching rule and then call that rule |
| không còn chứa từng luật cụ thể nữa | no longer holds each concrete rule |

**Thuật ngữ cần nhớ**

- rẽ nhánh theo loại → **switch-on-type**
- báo hiệu → **signal**
- đăng ký → **register** / **registration**
- luật phù hợp → **the matching rule**
- thiếu abstraction → **a missing abstraction**

---

## Phần 7 — OCP không có nghĩa là không bao giờ sửa code

**Tiếng Việt**

Chúng ta phải nói rõ rằng OCP không có nghĩa là chúng ta không bao giờ được sửa code. Nó nói rằng chúng ta thiết kế điểm mở rộng tại đúng cái trục thực sự biến động của bài toán. Vì vậy câu hỏi bẫy "để đạt OCP thì cứ tạo interface cho mọi thứ cho chắc, đúng không" phải được trả lời là không. Nếu chúng ta mở mọi thứ, chúng ta rơi thẳng vào over-engineering, và những interface chỉ có đúng một hiện thực sẽ làm code khó đọc mà không đem lại lợi ích nào. Cách chọn đúng chỗ để mở là chúng ta nhìn lại lịch sử thay đổi của repository và tìm xem chỗ nào đã bị sửa nhiều lần nhất. Chỗ đã biến động trong quá khứ nhiều khả năng vẫn biến động trong tương lai, và đó mới là nơi xứng đáng có một điểm mở rộng.

**English (bám cấu trúc tiếng Việt)**

We have to say clearly that OCP does not mean that we are never allowed to edit code. It says that we design extension points at exactly the axis that really moves in the problem. Therefore the trap question "to achieve OCP we should just create an interface for everything to be safe, right" has to be answered with a no. If we open everything, we fall straight into over-engineering, and interfaces that have exactly one implementation will make the code harder to read without bringing any benefit. The way to choose the right place to open is that we look back at the change history of the repository and find which place has been edited the most times. A place that moved in the past will most likely still move in the future, and that is the place which deserves an extension point.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| chúng ta phải nói rõ rằng | we have to say clearly that |
| chúng ta không bao giờ được sửa code | we are never allowed to edit code |
| đúng cái trục thực sự biến động | exactly the axis that really moves |
| cho mọi thứ cho chắc, đúng không | for everything to be safe, right |
| phải được trả lời là không | has to be answered with a no |
| chúng ta rơi thẳng vào | we fall straight into |
| mà không đem lại lợi ích nào | without bringing any benefit |
| nhìn lại lịch sử thay đổi của repository | look back at the change history of the repository |
| chỗ nào đã bị sửa nhiều lần nhất | which place has been edited the most times |
| là nơi xứng đáng có một điểm mở rộng | is the place which deserves an extension point |

**Thuật ngữ cần nhớ**

- điểm mở rộng → **extension point**
- trục biến động → **the axis that moves**
- lịch sử thay đổi → **change history**
- xứng đáng → **deserve**
- lợi ích → **benefit**

---

## Phần 8 — Áp dụng vào cổng thanh toán, kiến trúc plugin, và soát code AI

**Tiếng Việt**

Chúng ta hãy đặt hai nguyên tắc này vào một ví dụ rất thật là một cổng thanh toán hỗ trợ nhiều nhà cung cấp. Thay vì viết một chuỗi điều kiện kiểm tra xem nhà cung cấp có phải là Stripe hay không, chúng ta định nghĩa một interface `PaymentProvider` và chúng ta cho mỗi nhà cung cấp một class riêng. Khi công ty ký hợp đồng với nhà cung cấp thứ tư, chúng ta chỉ thêm một class mới và đăng ký nó, chứ chúng ta không đụng vào phần điều phối. Ở quy mô lớn hơn, kiến trúc plugin chính là OCP, ví dụ extension của VS Code, loader của webpack, hoặc middleware của Express. Trong các hệ thống đó, phần lõi được đóng lại còn phần mở rộng đi qua một interface công khai, và nhờ đó hàng nghìn người mở rộng được sản phẩm mà không ai phải sửa lõi.

Hai nguyên tắc này cũng là bộ lọc rất tốt khi chúng ta soát code do AI sinh ra. AI rất hay sinh ra một service ôm hết mọi việc, và nó cũng rất hay sinh ra switch-on-type, bởi vì cách đó chạy được và trông ngắn gọn. Khi review, chúng ta hỏi hai câu ngắn: class này có mấy lý do để thay đổi, và có chỗ rẽ nhánh theo loại nào nên trở thành polymorphism hay không. Nếu class có nhiều hơn một lý do để thay đổi, chúng ta yêu cầu tách theo actor chứ chúng ta không tách theo số dòng. Nếu có switch-on-type nằm trên một trục mà chúng ta biết chắc sẽ còn thêm loại mới, chúng ta yêu cầu chuyển sang Strategy ngay từ bây giờ.

**English (bám cấu trúc tiếng Việt)**

Let us put these two principles into a very real example, which is a payment gateway that supports many providers. Instead of writing a chain of conditions that checks whether the provider is Stripe or not, we define a `PaymentProvider` interface and we give each provider its own class. When the company signs a contract with the fourth provider, we only add a new class and register it, and we do not touch the coordinating part. At a larger scale, plugin architecture is exactly OCP, for example VS Code extensions, webpack loaders, or Express middleware. In those systems, the core is closed while the extensions go through a public interface, and thanks to that thousands of people can extend the product without anyone having to edit the core.

These two principles are also a very good filter when we review code that AI generates. AI very often produces a service that holds every job, and it also very often produces switch-on-type, because that way runs correctly and looks short. When we review, we ask two short questions: how many reasons to change does this class have, and is there any branching by type that should become polymorphism or not. If the class has more than one reason to change, we ask for a split by actor and we do not split by line count. If there is a switch-on-type sitting on an axis where we know for sure that new types will keep coming, we ask for a move to Strategy right now.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| một cổng thanh toán hỗ trợ nhiều nhà cung cấp | a payment gateway that supports many providers |
| kiểm tra xem … có phải là … hay không | checks whether … is … or not |
| chúng ta cho mỗi nhà cung cấp một class riêng | we give each provider its own class |
| khi công ty ký hợp đồng với | when the company signs a contract with |
| ở quy mô lớn hơn | at a larger scale |
| phần lõi được đóng lại | the core is closed |
| mà không ai phải sửa lõi | without anyone having to edit the core |
| một service ôm hết mọi việc | a service that holds every job |
| tách theo actor chứ … không tách theo số dòng | a split by actor and … do not split by line count |
| mà chúng ta biết chắc sẽ còn thêm loại mới | where we know for sure that new types will keep coming |

**Thuật ngữ cần nhớ**

- cổng thanh toán → **payment gateway**
- ký hợp đồng → **sign a contract**
- quy mô → **scale**
- phần lõi → **the core**
- phần mở rộng → **extension** / **plugin**

---

## Mô hình ghi nhớ

**Tiếng Việt**

Với SRP, mỗi class chỉ nên có một ông chủ để phục vụ, nghĩa là chỉ có một lý do để thay đổi. Với OCP, chúng ta thêm hành vi bằng cách thêm class mới, chứ chúng ta không cưa lại phần code đang chạy.

**English (bám cấu trúc tiếng Việt)**

With SRP, each class should only have one boss to serve, which means only one reason to change. With OCP, we add behaviour by adding a new class, and we do not saw into the code that is already running.

---

## Bảng thuật ngữ tổng hợp

| Tiếng Việt | Tiếng Anh | Ghi chú phát âm |
|---|---|---|
| hệ thống hoá | systematise | trọng âm đầu: SYS-te-ma-tise |
| vi phạm | violate / violation | động từ *violate* trọng âm đầu: VI-o-late |
| đánh đổi | trade-off | |
| nắm chắc | have a firm grip on | *firm* /fɜːm/ — "phơm", không đọc "phi-ơm" |
| vòng phỏng vấn | interview round | |
| trách nhiệm | responsibility | trọng âm âm thứ tư: res-pon-si-BIL-i-ty |
| mở rộng | extension | trọng âm âm hai: ex-TEN-sion |
| sửa đổi | modification | trọng âm áp chót: mo-di-fi-CA-tion |
| tách nhỏ, phân tách | segregation | trọng âm áp chót: se-gre-GA-tion |
| đảo ngược | inversion | trọng âm âm hai: in-VER-sion |
| nguyên tắc thay thế Liskov | Liskov substitution | *Liskov* đọc "LIS-kov"; *substitution* trọng âm áp chót: sub-sti-TU-tion |
| bên liên quan, người đặt yêu cầu | actor / stakeholder | |
| ứng viên | candidate | /ˈkændɪdət/ — CAN-di-date, đuôi đọc nhẹ "-đợt" |
| thoả mãn | satisfy | |
| class ôm mọi thứ | God Object | |
| vụn, nhỏ li ti | tiny | /ˈtaɪni/ — "TAI-ni", không đọc "ti-ni" |
| lưu trữ lâu dài | persistence | trọng âm âm hai: per-SIS-tence |
| kho chứa dữ liệu | repository | trọng âm âm hai: re-PO-si-to-ry |
| điều phối | coordinate / orchestrate | *orchestrate* /ˈɔːkɪstreɪt/ — âm **ch** đọc như "k": OR-kes-trate |
| hạ tầng | infrastructure | trọng âm đầu: IN-fra-struc-ture; cụm **-str-** phải bật |
| bộ test | test suite | *suite* /swiːt/ — đọc giống "sweet", không đọc theo mặt chữ |
| bảo trì | maintain / maintenance | danh từ *maintenance* trọng âm đầu: MAIN-te-nance |
| đa hình | polymorphism | trọng âm âm ba: po-ly-MOR-phism |
| lỗi hồi quy | regression | trọng âm âm hai: re-GRE-ssion |
| ổn định | stable / stably | |
| trực sự cố | on call | |
| sự cố | incident | trọng âm đầu: IN-ci-dent |
| rẽ nhánh theo loại | switch-on-type | *switch* — đuôi **-tch** phải bật, không nuốt |
| báo hiệu | signal | chữ **g** không đọc rời: "SIG-nồ" |
| đăng ký | register / registration | |
| thiếu abstraction | a missing abstraction | *abstraction* — cụm **-bstr-** khó, tập đọc chậm |
| điểm mở rộng | extension point | |
| trục biến động | the axis that moves | *axis* /ˈæksɪs/ — đuôi **-ks-** phải bật rõ |
| lịch sử thay đổi | change history | |
| xứng đáng | deserve | trọng âm cuối: de-SERVE |
| lợi ích | benefit | trọng âm đầu: BE-ne-fit |
| cổng thanh toán | payment gateway | |
| quy mô | scale | |
| phần lõi | the core | |
| phần mở rộng, plugin | extension / plugin | |
| lớp trung gian xử lý request | middleware | |
| rủi ro | risk | đuôi **-sk** phải bật rõ, không đọc thành "rít" |

---

## Bài nói

> **Cách làm:** ghi âm lại giọng của mình, mỗi câu nói liên tục **60–90 giây**, **không nhìn tài liệu** khi nói. Mỗi bài nói phải dùng ít nhất **6 thuật ngữ** trong bảng ở trên. Nghe lại bản ghi âm một lần, ghi ra hai chỗ mình bị vấp, rồi nói lại lần hai.

1. Name the five SOLID letters and explain each one in a single sentence, as if the interviewer had just asked you the opening question.
2. Explain to a junior developer why "a class does one thing" is a weak definition of SRP, and what question you ask instead to decide whether a class should be split.
3. A colleague says: "To be safe with OCP, let us put an interface in front of every class in this service." Explain why you would push back, and how you would decide where an extension point is actually worth it.
4. Take a `UserService` that validates, saves to the database, and sends email. Describe how you would split it and what each new class would own.
5. Describe what happens to your release risk when you add a fifth payment provider by editing an existing `if/else` chain, compared with adding a new class behind an interface.
6. When would you accept a `switch` on type instead of introducing polymorphism? Give the trade-off in both directions.
7. You are reviewing a service that an AI tool generated, and it handles four notification channels inside one method. Describe out loud the two questions you would ask and what change you would request.
