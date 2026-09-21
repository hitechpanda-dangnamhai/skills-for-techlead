# Bài 14 — DDD (2): Aggregate · Invariant · Transaction Boundary · Repository · Domain/App Service · Domain Event · khi nào KHÔNG dùng DDD
### Bản song ngữ: tiếng Anh bám cấu trúc tiếng Việt

> **Cách dùng bài này:** đọc đoạn tiếng Việt trước, tự dịch trong đầu, rồi mới nhìn sang đoạn tiếng Anh để so. Sau khi so xong, **đọc to đoạn tiếng Anh hai lần** — lần một để nhìn chữ, lần hai để nghe chính mình. Phần bài nói ở cuối là bắt buộc; không nói ra thì bài này chỉ là bài đọc.

---

## Phần 1 — Tactical DDD và câu hỏi khi nào nó thành chi phí thừa

**Tiếng Việt**

Bài này là phần tactical của DDD, tức là các building block giúp chúng ta giữ tính nhất quán cho một domain phức tạp. Bài này cũng là phần judgment, bởi vì chúng ta phải trả lời được câu hỏi khi nào DDD đầy đủ trở thành chi phí thừa. Đây là đỉnh của mạch mô hình hoá nghiệp vụ mà chúng ta đã đi từ Bài 11. Nó cũng chuẩn bị trực tiếp cho phần judgment của Tech Lead ở Bài 15. Điều chúng ta cần nhớ xuyên suốt là phần strategic ở Bài 13 thì luôn dùng được, còn phần tactical ở bài này thì chúng ta chỉ dùng có chọn lọc.

**English (bám cấu trúc tiếng Việt)**

This lesson is the tactical part of DDD, that is, the building blocks that help us keep consistency for a complex domain. This lesson is also the judgment part, because we have to be able to answer the question of when full DDD becomes an unnecessary cost. This is the peak of the business modelling thread that we have been following since Lesson 11. It also prepares directly for the Tech Lead judgment part in Lesson 15. The thing we need to remember throughout is that the strategic part in Lesson 13 is always usable, while the tactical part in this lesson is something we only use selectively.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| giúp chúng ta giữ tính nhất quán | help us keep consistency |
| trở thành chi phí thừa | becomes an unnecessary cost |
| đỉnh của mạch mô hình hoá nghiệp vụ | the peak of the business modelling thread |
| mà chúng ta đã đi từ Bài 11 | that we have been following since Lesson 11 |
| chuẩn bị trực tiếp cho | prepares directly for |
| điều chúng ta cần nhớ xuyên suốt | the thing we need to remember throughout |
| chúng ta chỉ dùng có chọn lọc | we only use selectively |

**Thuật ngữ cần nhớ**

- tính nhất quán → **consistency**
- chi phí thừa → **unnecessary cost**
- mạch, luồng xuyên suốt → **thread**
- có chọn lọc → **selectively**
- viên gạch xây dựng → **building block**

---

## Phần 2 — Aggregate là pháo đài chỉ có một cổng

**Tiếng Việt**

Aggregate là một cụm object tạo thành một ranh giới nhất quán duy nhất. Aggregate Root là cửa duy nhất để bên ngoài truy cập và sửa mọi thứ nằm bên trong cụm đó. Ví dụ quen thuộc nhất là một đơn hàng chứa các dòng hàng, trong đó đơn hàng là root còn các dòng hàng nằm bên trong. Bên ngoài không được cầm thẳng một dòng hàng rồi sửa nó, mà bên ngoài phải đi qua đơn hàng. Ẩn dụ dễ nhớ là aggregate giống một pháo đài chỉ có một cổng, và mọi người ra vào đều phải qua cái cổng đó. Nếu chúng ta để nhiều cửa, thì không ai bảo đảm được rằng bên trong pháo đài luôn đúng luật.

**English (bám cấu trúc tiếng Việt)**

An aggregate is a cluster of objects forming one single consistency boundary. The aggregate root is the only door through which the outside accesses and edits everything sitting inside that cluster. The most familiar example is an order holding order lines, in which the order is the root while the order lines sit inside. The outside is not allowed to take an order line directly and edit it, but the outside has to go through the order. The metaphor that is easy to remember is that an aggregate is like a fortress with only one gate, and everybody going in and out has to pass through that gate. If we leave many doors open, then nobody can guarantee that everything inside the fortress always follows the rules.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| một cụm object | a cluster of objects |
| tạo thành một ranh giới nhất quán duy nhất | forming one single consistency boundary |
| cửa duy nhất để bên ngoài truy cập | the only door through which the outside accesses |
| một đơn hàng chứa các dòng hàng | an order holding order lines |
| không được cầm thẳng một dòng hàng rồi sửa nó | is not allowed to take an order line directly and edit it |
| một pháo đài chỉ có một cổng | a fortress with only one gate |
| mọi người ra vào đều phải qua cái cổng đó | everybody going in and out has to pass through that gate |
| không ai bảo đảm được rằng | nobody can guarantee that |

**Thuật ngữ cần nhớ**

- cụm gốc tổng hợp → **aggregate**
- gốc của cụm → **aggregate root**
- ranh giới nhất quán → **consistency boundary**
- dòng hàng trong đơn → **order line**
- pháo đài → **fortress**

---

## Phần 3 — Tham chiếu aggregate khác bằng định danh

**Tiếng Việt**

Có một quy tắc rất quan trọng, đó là chúng ta tham chiếu tới aggregate khác bằng định danh, chứ chúng ta không nhúng cả object vào bên trong. Ví dụ một đơn hàng chỉ giữ mã khách hàng, chứ đơn hàng không giữ nguyên cả object khách hàng. Lý do thứ nhất là quy tắc này giữ được ranh giới, bởi vì mỗi aggregate chỉ chịu trách nhiệm cho phần bên trong của chính nó. Lý do thứ hai là nó tránh việc chúng ta vô tình nạp lên và sửa chéo sang một aggregate khác. Lý do thứ ba là nó cho phép chúng ta chấp nhận nhất quán sau cùng giữa hai phía. Nếu chúng ta nhúng cả object, chúng ta sẽ phải khiêng cả lâu đài mỗi lần chúng ta chỉ cần gọi tên một người.

**English (bám cấu trúc tiếng Việt)**

There is a very important rule, which is that we refer to another aggregate by identifier, and we do not embed the whole object inside. For example an order only holds the customer identifier, and the order does not hold the whole customer object. The first reason is that this rule preserves the boundary, because each aggregate is only responsible for what is inside itself. The second reason is that it avoids the case where we accidentally load and edit across into another aggregate. The third reason is that it lets us accept eventual consistency between the two sides. If we embed the whole object, we will have to carry the entire castle every time we only need to call one person's name.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| tham chiếu tới aggregate khác bằng định danh | refer to another aggregate by identifier |
| chúng ta không nhúng cả object vào bên trong | we do not embed the whole object inside |
| quy tắc này giữ được ranh giới | this rule preserves the boundary |
| chỉ chịu trách nhiệm cho phần bên trong của chính nó | is only responsible for what is inside itself |
| chúng ta vô tình nạp lên và sửa chéo | we accidentally load and edit across |
| chấp nhận nhất quán sau cùng | accept eventual consistency |
| chúng ta sẽ phải khiêng cả lâu đài | we will have to carry the entire castle |
| mỗi lần chúng ta chỉ cần gọi tên một người | every time we only need to call one person's name |

**Thuật ngữ cần nhớ**

- nhúng vào → **embed**
- giữ gìn, bảo toàn → **preserve**
- vô tình → **accidentally**
- nhất quán sau cùng → **eventual consistency**
- lâu đài → **castle**

---

## Phần 4 — Invariant và lý do mọi thay đổi phải đi qua root

**Tiếng Việt**

Invariant là một quy tắc nghiệp vụ luôn luôn phải đúng đối với một aggregate. Ví dụ dễ hình dung là tổng giá trị của các dòng hàng không được vượt quá hạn mức của đơn. Vì mọi thay đổi đều đi qua root, root có đúng một chỗ để kiểm tra và để giữ tính nhất quán bên trong. Cụ thể là khi ai đó thêm một dòng hàng, chính đơn hàng tính lại tổng rồi ném lỗi nếu tổng vượt quá hạn mức. Đây chính là lý do encapsulation ở Bài 1 lại quan trọng đến thế, bởi vì chúng ta không cho phép ai sửa thẳng vào ruột của aggregate. Nếu chúng ta để lộ danh sách bên trong ra ngoài, thì bất kỳ ai cũng thêm được một dòng hàng mà họ không đi qua bước kiểm tra.

**English (bám cấu trúc tiếng Việt)**

An invariant is a business rule that must always be true for an aggregate. An example that is easy to picture is that the total value of the order lines must not exceed the credit limit of the order. Because every change goes through the root, the root has exactly one place to check and to keep the consistency inside. Concretely, when somebody adds an order line, the order itself recomputes the total and then throws an error if the total exceeds the limit. This is exactly the reason why the encapsulation in Lesson 1 matters so much, because we do not allow anybody to edit straight into the guts of the aggregate. If we expose the inner list to the outside, then anybody at all can add an order line without going through the checking step.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| luôn luôn phải đúng đối với một aggregate | must always be true for an aggregate |
| không được vượt quá hạn mức của đơn | must not exceed the credit limit of the order |
| root có đúng một chỗ để kiểm tra | the root has exactly one place to check |
| chính đơn hàng tính lại tổng | the order itself recomputes the total |
| lại quan trọng đến thế | matters so much |
| sửa thẳng vào ruột của aggregate | edit straight into the guts of the aggregate |
| nếu chúng ta để lộ danh sách bên trong ra ngoài | if we expose the inner list to the outside |
| mà họ không đi qua bước kiểm tra | without going through the checking step |

**Thuật ngữ cần nhớ**

- bất biến nghiệp vụ → **invariant**
- vượt quá → **exceed**
- tính lại → **recompute**
- ruột, phần bên trong → **the guts**
- để lộ ra → **expose**

---

## Phần 5 — Ranh giới aggregate gần trùng với ranh giới transaction

**Tiếng Việt**

Ranh giới của aggregate gần như trùng với ranh giới của transaction. Nói cụ thể hơn, một transaction chỉ nên sửa đúng một aggregate. Khi một thao tác nghiệp vụ buộc phải đổi nhiều aggregate, chúng ta không nên gói tất cả vào trong một transaction khổng lồ. Thay vào đó chúng ta dùng nhất quán sau cùng, dùng domain event, hoặc dùng saga để điều phối nhiều bước. Ở đây có một đánh đổi mà chúng ta phải nói được, đó là ranh giới to thì gây tranh chấp và khoá nhiều, còn ranh giới nhỏ thì lại sinh ra nhiều thao tác xuyên aggregate. Cách cân là chúng ta nhìn vào invariant thật sự, bởi vì những gì phải luôn đúng cùng lúc thì mới nên nằm chung một ranh giới.

**English (bám cấu trúc tiếng Việt)**

The boundary of an aggregate almost coincides with the boundary of a transaction. To say it more concretely, one transaction should only edit exactly one aggregate. When a business operation is forced to change several aggregates, we should not wrap everything inside one enormous transaction. Instead we use eventual consistency, we use domain events, or we use a saga to coordinate the several steps. There is a trade-off here that we have to be able to state, which is that a large boundary causes contention and heavy locking, while a small boundary instead produces many cross-aggregate operations. The way to balance it is that we look at the real invariants, because only the things that must be true at the same time should sit inside one boundary.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| gần như trùng với | almost coincides with |
| chỉ nên sửa đúng một aggregate | should only edit exactly one aggregate |
| buộc phải đổi nhiều aggregate | is forced to change several aggregates |
| một transaction khổng lồ | one enormous transaction |
| để điều phối nhiều bước | to coordinate the several steps |
| ranh giới to thì gây tranh chấp và khoá nhiều | a large boundary causes contention and heavy locking |
| sinh ra nhiều thao tác xuyên aggregate | produces many cross-aggregate operations |
| những gì phải luôn đúng cùng lúc | the things that must be true at the same time |

**Thuật ngữ cần nhớ**

- trùng với → **coincide with**
- tranh chấp tài nguyên → **contention**
- khoá dữ liệu → **locking**
- chuỗi giao dịch phân tán → **saga**
- xuyên nhiều aggregate → **cross-aggregate**

---

## Phần 6 — Repository trong DDD nói bằng ngôn ngữ domain

**Tiếng Việt**

Repository trong DDD khác với một DAO hoặc một lớp bọc ORM thông thường. Nó hoạt động như một bộ sưu tập của các aggregate root, và nó giấu hoàn toàn phần lưu trữ. Nó trả về domain object, chứ nó không để lộ entity của ORM ra bên ngoài. Interface của nó được thiết kế theo nhu cầu của domain, ví dụ chúng ta viết một method tìm các đơn đang hoạt động của một khách hàng. Chúng ta không thiết kế interface theo khả năng của database, ví dụ chúng ta không viết một method nhận vào một câu truy vấn thô. Nói ngắn gọn, repository phải nói bằng ngôn ngữ của domain, đúng tinh thần Ubiquitous Language ở Bài 13.

**English (bám cấu trúc tiếng Việt)**

A repository in DDD is different from a DAO or an ordinary ORM wrapper. It behaves like a collection of aggregate roots, and it hides the persistence completely. It returns domain objects, and it does not expose ORM entities to the outside. Its interface is designed around the needs of the domain, for example we write a method that finds the active orders of a customer. We do not design the interface around the capabilities of the database, for example we do not write a method that takes in a raw query. In short, the repository must speak in the language of the domain, exactly in the spirit of the Ubiquitous Language in Lesson 13.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| một lớp bọc ORM thông thường | an ordinary ORM wrapper |
| nó hoạt động như một bộ sưu tập | it behaves like a collection |
| nó giấu hoàn toàn phần lưu trữ | it hides the persistence completely |
| nó không để lộ entity của ORM ra bên ngoài | it does not expose ORM entities to the outside |
| được thiết kế theo nhu cầu của domain | is designed around the needs of the domain |
| theo khả năng của database | around the capabilities of the database |
| một câu truy vấn thô | a raw query |
| nói ngắn gọn | in short |

**Thuật ngữ cần nhớ**

- kho chứa aggregate → **repository**
- bộ sưu tập → **collection**
- lớp bọc → **wrapper**
- truy vấn thô → **raw query**
- khả năng → **capability**

---

## Phần 7 — Đặt side effect đúng tầng, và phát domain event

**Tiếng Việt**

Chúng ta nhắc lại ranh giới giữa Domain Service và Application Service, vì đây là chỗ hay sai nhất khi làm DDD. Domain Service chứa rule nghiệp vụ thuần liên quan tới nhiều aggregate, và nó không chạm vào hạ tầng. Application Service điều phối use case, quản transaction, và gọi ra bên ngoài qua các port. Ví dụ kinh điển là việc gửi email xác nhận sau khi đặt đơn, và đó là một side effect thuộc về hạ tầng. Vì vậy việc gửi email phải nằm ở Application Service và phải đi qua một port, chứ nó tuyệt đối không nằm bên trong domain.

Domain event là một sự kiện mô tả rằng có việc gì đó đã xảy ra trong domain, ví dụ một đơn hàng đã được đặt. Chúng ta phát sự kiện đó ra để những phần khác phản ứng lại một cách bất đồng bộ. Cách này giảm coupling rất mạnh, bởi vì một aggregate không gọi trực tiếp sang aggregate khác hay sang context khác. Đổi lại, chúng ta chấp nhận nhất quán sau cùng thay vì nhất quán tức thì. Về cơ chế, đây chính là Observer và Pub/Sub ở Bài 9 được đưa lên cấp nghiệp vụ.

**English (bám cấu trúc tiếng Việt)**

Let us restate the line between the domain service and the application service, because this is the place where people get it wrong the most in DDD. The domain service holds pure business rules involving several aggregates, and it does not touch the infrastructure. The application service coordinates the use case, manages the transaction, and calls out through the ports. The classic example is sending a confirmation email after an order is placed, and that is a side effect belonging to the infrastructure. Therefore sending the email has to sit in the application service and has to go through a port, and it absolutely does not sit inside the domain.

A domain event is an event describing that something has happened in the domain, for example an order has been placed. We publish that event so that other parts react to it asynchronously. This way reduces coupling very strongly, because one aggregate does not call directly into another aggregate or into another context. In exchange, we accept eventual consistency instead of immediate consistency. As for the mechanism, this is exactly the Observer and Pub/Sub from Lesson 9 lifted to the business level.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| đây là chỗ hay sai nhất khi làm DDD | this is the place where people get it wrong the most in DDD |
| rule nghiệp vụ thuần | pure business rules |
| gửi email xác nhận sau khi đặt đơn | sending a confirmation email after an order is placed |
| một side effect thuộc về hạ tầng | a side effect belonging to the infrastructure |
| nó tuyệt đối không nằm bên trong domain | it absolutely does not sit inside the domain |
| mô tả rằng có việc gì đó đã xảy ra | describing that something has happened |
| chúng ta phát sự kiện đó ra | we publish that event |
| phản ứng lại một cách bất đồng bộ | react to it asynchronously |
| đổi lại, chúng ta chấp nhận | in exchange, we accept |
| được đưa lên cấp nghiệp vụ | lifted to the business level |

**Thuật ngữ cần nhớ**

- tác dụng phụ, việc phụ → **side effect**
- email xác nhận → **confirmation email**
- phát sự kiện → **publish an event**
- nhất quán tức thì → **immediate consistency**
- cơ chế → **mechanism**

---

## Phần 8 — Khi nào KHÔNG nên dùng DDD đầy đủ, và soát code AI

**Tiếng Việt**

Bây giờ là câu hỏi judgment quan trọng nhất của cả hai bài DDD. Phần tactical có chi phí học và chi phí triển khai rất cao, cho nên chúng ta không nên áp nó ở mọi nơi. Chúng ta không nên áp đầy đủ khi hệ thống chỉ là CRUD đơn giản, khi nghiệp vụ có rất ít rule, hoặc khi đội quá nhỏ để duy trì mức phức tạp đó. Nó đáng làm khi nghiệp vụ phức tạp, khi có nhiều invariant phải giữ, khi ngôn ngữ domain giàu, và khi yêu cầu thay đổi liên tục. Điểm phân biệt mà chúng ta phải nói rõ trong phỏng vấn là strategic khác tactical, bởi vì bounded context thì luôn hữu ích còn aggregate và repository thì chỉ nên dùng có chọn lọc. Nếu ai đó đề nghị áp full DDD tactical cho một service nội bộ chỉ đọc ghi vài bảng, chúng ta nên phản biện bằng chính chi phí bảo trì.

Với code do AI sinh ra, bài này có bốn thứ cần soát. Thứ nhất là AI hay nhúng cả object thay vì tham chiếu bằng định danh. Thứ hai là AI hay nhét side effect như gửi email vào thẳng bên trong domain. Thứ ba là AI hay để repository trả về entity của ORM, và như vậy phần lưu trữ rò ngược vào domain. Thứ tư là AI hay đề xuất áp DDD đầy đủ cho một service CRUD, cho nên chúng ta phải hỏi lại rằng ở đây chúng ta có thật sự cần DDD hay không.

**English (bám cấu trúc tiếng Việt)**

Now comes the most important judgment question of both DDD lessons. The tactical part has a very high learning cost and a very high implementation cost, so we should not apply it everywhere. We should not apply it in full when the system is only simple CRUD, when the business has very few rules, or when the team is too small to maintain that level of complexity. It is worth doing when the business is complex, when there are many invariants to keep, when the domain language is rich, and when the requirements change constantly. The distinction that we have to state clearly in an interview is that strategic differs from tactical, because the bounded context is always useful while aggregates and repositories should only be used selectively. If somebody proposes applying full tactical DDD to an internal service that only reads and writes a few tables, we should push back with the maintenance cost itself.

For code that AI generates, this lesson has four things that need review. The first is that AI often embeds the whole object instead of referring by identifier. The second is that AI often puts side effects such as sending email straight inside the domain. The third is that AI often lets the repository return ORM entities, and in that way the persistence leaks back into the domain. The fourth is that AI often proposes applying full DDD to a CRUD service, so we have to ask back whether we really need DDD here or not.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| câu hỏi judgment quan trọng nhất của cả hai bài | the most important judgment question of both lessons |
| chi phí học và chi phí triển khai rất cao | a very high learning cost and a very high implementation cost |
| khi đội quá nhỏ để duy trì mức phức tạp đó | when the team is too small to maintain that level of complexity |
| khi ngôn ngữ domain giàu | when the domain language is rich |
| khi yêu cầu thay đổi liên tục | when the requirements change constantly |
| chỉ nên dùng có chọn lọc | should only be used selectively |
| chúng ta nên phản biện bằng chính chi phí bảo trì | we should push back with the maintenance cost itself |
| phần lưu trữ rò ngược vào domain | the persistence leaks back into the domain |
| chúng ta phải hỏi lại rằng | we have to ask back whether |

**Thuật ngữ cần nhớ**

- chi phí triển khai → **implementation cost**
- yêu cầu (nghiệp vụ) → **requirement**
- phân biệt, điểm khác → **distinction**
- phản biện, đẩy lại → **push back**
- chi phí bảo trì → **maintenance cost**

---

## Mô hình ghi nhớ

**Tiếng Việt**

Aggregate là một pháo đài chỉ có một cổng, và cái cổng đó là root bảo vệ mọi invariant bên trong. Ra ngoài thì chúng ta gọi tên bằng định danh chứ chúng ta không khiêng cả lâu đài, một transaction chỉ nên đụng một pháo đài, và khi cần nhiều pháo đài thì chúng ta gửi thư bằng domain event.

**English (bám cấu trúc tiếng Việt)**

An aggregate is a fortress with only one gate, and that gate is the root protecting every invariant inside. Going outside, we call names by identifier and we do not carry the entire castle, one transaction should only touch one fortress, and when we need several fortresses then we send letters through domain events.

---

## Bảng thuật ngữ tổng hợp

| Tiếng Việt | Tiếng Anh | Ghi chú phát âm |
|---|---|---|
| tính nhất quán | consistency | trọng âm âm hai: con-SIS-ten-cy |
| chi phí thừa | unnecessary cost | *unnecessary* trọng âm âm hai: un-NE-ce-ssa-ry |
| mạch, luồng xuyên suốt | thread | /θred/ — có âm **th**, vần với "red" |
| có chọn lọc | selectively | trọng âm âm hai: se-LEC-tive-ly |
| viên gạch xây dựng | building block | |
| cụm gốc tổng hợp | aggregate | danh từ /ˈægrɪgət/ — AG-gre-gợt, đuôi đọc nhẹ |
| gốc của cụm | aggregate root | |
| ranh giới nhất quán | consistency boundary | *boundary* /ˈbaʊndri/ — BOUN-dry |
| dòng hàng trong đơn | order line | |
| pháo đài | fortress | trọng âm đầu: FOR-tress |
| nhúng vào | embed | trọng âm cuối: em-BED |
| giữ gìn, bảo toàn | preserve | trọng âm cuối: pre-SERVE |
| vô tình | accidentally | trọng âm âm ba: ac-ci-DEN-tal-ly |
| nhất quán sau cùng | eventual consistency | *eventual* trọng âm âm hai: e-VEN-tu-al — nghĩa là "sau cùng", KHÔNG phải "có thể xảy ra" |
| lâu đài | castle | /ˈkɑːsl/ — chữ **t** câm, đọc "CAA-sồ" |
| bất biến nghiệp vụ | invariant | trọng âm âm hai: in-VA-ri-ant |
| vượt quá | exceed | trọng âm cuối: ex-CEED |
| tính lại | recompute | |
| ruột, phần bên trong | the guts | |
| để lộ ra | expose | trọng âm cuối: ex-POSE, đuôi đọc **-z** |
| trùng với | coincide with | trọng âm cuối: co-in-CIDE |
| tranh chấp tài nguyên | contention | trọng âm âm hai: con-TEN-tion |
| khoá dữ liệu | locking | |
| chuỗi giao dịch phân tán | saga | /ˈsɑːgə/ — "SAA-gơ" |
| xuyên nhiều aggregate | cross-aggregate | |
| kho chứa aggregate | repository | trọng âm âm hai: re-PO-si-to-ry |
| bộ sưu tập | collection | trọng âm âm hai: co-LLEC-tion |
| lớp bọc | wrapper | chữ **w** câm: "RA-pơ" |
| truy vấn thô | raw query | *query* /ˈkwɪəri/ — "KWI-ri" |
| khả năng | capability | trọng âm âm ba: ca-pa-BI-li-ty |
| tác dụng phụ, việc phụ | side effect | |
| email xác nhận | confirmation email | *confirmation* trọng âm áp chót: con-fir-MA-tion |
| phát sự kiện | publish an event | |
| nhất quán tức thì | immediate consistency | *immediate* trọng âm âm hai: im-ME-di-ate |
| cơ chế | mechanism | /ˈmekənɪzəm/ — ME-cha-nism, **ch** đọc như "k" |
| chi phí triển khai | implementation cost | *implementation* trọng âm áp chót: im-ple-men-TA-tion |
| yêu cầu (nghiệp vụ) | requirement | trọng âm âm hai: re-QUIRE-ment |
| phân biệt, điểm khác | distinction | trọng âm âm hai: dis-TINC-tion |
| phản biện, đẩy lại | push back | |
| chi phí bảo trì | maintenance cost | *maintenance* trọng âm đầu: MAIN-te-nance |

---

## Bài nói

> **Cách làm:** ghi âm lại giọng của mình, mỗi câu nói liên tục **60–90 giây**, **không nhìn tài liệu** khi nói. Mỗi bài nói phải dùng ít nhất **6 thuật ngữ** trong bảng ở trên. Nghe lại bản ghi âm một lần, ghi ra hai chỗ mình bị vấp, rồi nói lại lần hai.

1. Explain what an aggregate and an aggregate root are, and why every change has to go through the root.
2. Explain why you reference another aggregate by identifier instead of embedding the whole object, and what you give up in exchange.
3. Describe what happens when a business operation has to change three aggregates at once, and how you would design it without one enormous transaction.
4. A colleague puts the confirmation email inside the `Order` entity because "it happens right after the order is placed". Explain why you would push back and where that code belongs.
5. Explain how a DDD repository differs from a DAO, and give one method name that speaks the domain language rather than the database language.
6. Someone proposes full tactical DDD for a small internal CRUD service "so that it is done properly". Explain why you would push back, and say which part of DDD you would still keep.
7. You are reviewing AI-generated domain code for an ordering system. Describe out loud the four things you would check and what you would ask to be changed first.
