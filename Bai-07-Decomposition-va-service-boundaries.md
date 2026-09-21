# Bài 7 — Decomposition & service boundaries
### Bản song ngữ: tiếng Anh bám cấu trúc tiếng Việt

> **Cách dùng:** đọc đoạn tiếng Việt trước → tự dịch trong đầu → so với bản tiếng Anh bên dưới → **đọc to bản tiếng Anh hai lần**. Lần một đọc chậm cho đúng từ, lần hai đọc liền mạch như đang nói với người phỏng vấn. Bài này nhiều từ trừu tượng về tổ chức, nên hãy chú ý các cụm chỉ quan hệ giữa các phần, ví dụ *cái gì thay đổi cùng nhau* và *gọi chéo liên tục*.

---

## Phần 1 — Ranh giới đúng là ranh giới ít phải họp

**Tiếng Việt**

Chúng ta chia dịch vụ giống như chia phòng ban trong một công ty. Nếu chúng ta chia theo chức năng nghiệp vụ, ví dụ kế toán, kho và bán hàng, thì mỗi phòng tự làm được việc của mình từ đầu tới cuối. Nếu chúng ta chia theo tầng kỹ thuật, ví dụ một phòng chuyên viết SQL và một phòng chuyên viết API, thì làm bất cứ việc gì cũng phải họp cả ba phòng. Vì vậy, một ranh giới tốt là ranh giới mà một thay đổi nghiệp vụ điển hình chỉ chạm vào một chỗ. Ngược lại, một ranh giới xấu bắt chúng ta phối hợp giữa nhiều đội cho mỗi việc nhỏ.

Cách kiểm tra nhanh nhất là chúng ta lấy ba yêu cầu nghiệp vụ gần đây rồi đếm xem mỗi yêu cầu chạm vào bao nhiêu dịch vụ. Nếu con số đó thường lớn hơn hai, ranh giới của chúng ta gần như chắc chắn đang sai. Đây là một phép thử rất mạnh vì nó dựa trên lịch sử thật của đội, chứ không dựa trên sơ đồ trên giấy. Chúng ta cũng nên hỏi ngược lại rằng có dịch vụ nào chưa bao giờ được triển khai một mình hay không. Nếu hai dịch vụ luôn đi cùng nhau trong mọi lần phát hành, thì trên thực tế chúng đã là một dịch vụ, chỉ khác là chúng ta đang trả thêm chi phí mạng cho ảo giác về sự tách biệt.

**English (bám cấu trúc tiếng Việt)**

We split services in the same way we split departments in a company. If we split by business function, for example accounting, warehouse and sales, then each department can do its own work from end to end. If we split by technical layer, for example one department that only writes SQL and one department that only writes APIs, then doing anything at all requires a meeting of all three departments. Therefore, a good boundary is a boundary where a typical business change touches only one place. On the contrary, a bad boundary forces us to coordinate across several teams for every small job.

The fastest way to check is that we take three recent business requests and count how many services each request touched. If that number is usually larger than two, our boundaries are almost certainly wrong. This is a very strong test because it is based on the real history of the team, not on a diagram on paper. We should also ask the reverse question, whether there is any service that has never been deployed on its own. If two services always travel together in every release, then in practice they are already one service, the only difference being that we are paying extra network cost for the illusion of separation.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| chia theo chức năng nghiệp vụ | split by business function |
| tự làm được việc của mình từ đầu tới cuối | can do its own work from end to end |
| làm bất cứ việc gì cũng phải họp cả ba phòng | doing anything at all requires a meeting of all three departments |
| một thay đổi nghiệp vụ điển hình | a typical business change |
| bắt chúng ta phối hợp giữa nhiều đội | forces us to coordinate across several teams |
| đếm xem mỗi yêu cầu chạm vào bao nhiêu dịch vụ | count how many services each request touched |
| dựa trên lịch sử thật của đội | based on the real history of the team |
| chưa bao giờ được triển khai một mình | has never been deployed on its own |
| luôn đi cùng nhau trong mọi lần phát hành | always travel together in every release |
| ảo giác về sự tách biệt | the illusion of separation |

**Thuật ngữ cần nhớ**

- ranh giới dịch vụ → **a service boundary**
- phân rã → **decomposition**
- chức năng nghiệp vụ → **a business function**
- tầng kỹ thuật → **a technical layer**
- lần phát hành → **a release**

---

## Phần 2 — Chia theo năng lực nghiệp vụ, không chia theo tầng kỹ thuật

**Tiếng Việt**

Nguyên tắc gốc là chúng ta chia theo năng lực nghiệp vụ và theo tốc độ thay đổi, chứ không chia theo tầng kỹ thuật. Nói ngắn gọn, cái gì thay đổi cùng nhau thì nên nằm cùng một chỗ. Trong ngôn ngữ của thiết kế hướng miền, mỗi mảnh như vậy là một ngữ cảnh có ranh giới, và bên trong ranh giới đó các khái niệm có nghĩa thống nhất. Ví dụ, từ đơn hàng trong ngữ cảnh bán hàng khác với từ đơn hàng trong ngữ cảnh giao vận, và chính sự khác nghĩa đó là dấu hiệu của một đường cắt tự nhiên. Khi chúng ta cắt đúng những đường tự nhiên này, mỗi dịch vụ tự chủ được một nghiệp vụ trọn vẹn.

Nếu chúng ta chia theo tầng, chúng ta tạo ra một hệ thống mà mọi thay đổi đều phải đi ngang qua tất cả các mảnh. Thêm một trường vào biểu mẫu đặt hàng sẽ buộc chúng ta sửa dịch vụ giao diện, sửa dịch vụ nghiệp vụ và sửa dịch vụ dữ liệu, rồi phối hợp ba lần phát hành. Đó chính là kiểu kiến trúc mà người ta gọi đùa là khối bùn phân tán, vì nó có đủ chi phí của hệ phân tán mà không có lợi ích nào của sự tự chủ. Hậu quả thực tế là tốc độ giao hàng của đội chậm lại, dù sơ đồ kiến trúc trông rất hiện đại. Vì vậy, trong phỏng vấn chúng ta nên nói rõ tiêu chí chia trước khi vẽ bất kỳ hộp dịch vụ nào.

**English (bám cấu trúc tiếng Việt)**

The root principle is that we split by business capability and by rate of change, rather than splitting by technical layer. Put briefly, the things that change together should sit in the same place. In the language of domain-driven design, each such piece is a bounded context, and inside that boundary the concepts have one consistent meaning. For example, the word order in the sales context is different from the word order in the shipping context, and that very difference in meaning is the sign of a natural seam. When we cut along these natural seams, each service owns one complete business capability.

If we split by layer, we create a system in which every change has to travel across all the pieces. Adding one field to the order form will force us to change the interface service, change the business service and change the data service, and then coordinate three releases. That is exactly the kind of architecture people jokingly call a distributed ball of mud, because it has all the costs of a distributed system without any of the benefits of autonomy. The practical consequence is that the team's delivery speed slows down, even though the architecture diagram looks very modern. Therefore, in an interview we should state our splitting criteria clearly before we draw any service box.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| theo tốc độ thay đổi | by rate of change |
| cái gì thay đổi cùng nhau thì nên nằm cùng một chỗ | the things that change together should sit in the same place |
| một ngữ cảnh có ranh giới | a bounded context |
| các khái niệm có nghĩa thống nhất | the concepts have one consistent meaning |
| dấu hiệu của một đường cắt tự nhiên | the sign of a natural seam |
| tự chủ được một nghiệp vụ trọn vẹn | owns one complete business capability |
| phải đi ngang qua tất cả các mảnh | has to travel across all the pieces |
| phối hợp ba lần phát hành | coordinate three releases |
| khối bùn phân tán | a distributed ball of mud |
| tốc độ giao hàng của đội chậm lại | the team's delivery speed slows down |

**Thuật ngữ cần nhớ**

- năng lực nghiệp vụ → **a business capability**
- ngữ cảnh có ranh giới → **a bounded context**
- thiết kế hướng miền → **domain-driven design (DDD)**
- tính tự chủ → **autonomy**
- độ dính kết → **coupling**

---

## Phần 3 — Bốn dấu hiệu cho thấy ranh giới bị vẽ sai

**Tiếng Việt**

Có bốn dấu hiệu giúp chúng ta nhận ra một ranh giới đã bị vẽ sai, và chúng ta nên thuộc cả bốn. Dấu hiệu thứ nhất là hai dịch vụ luôn được sửa và triển khai cùng nhau, tức là chúng chưa bao giờ tách rời trên thực tế. Dấu hiệu thứ hai là chúng gọi chéo nhau liên tục cho một thao tác đơn giản, và người ta gọi kiểu giao tiếp đó là lắm lời. Dấu hiệu thứ ba là chúng ta thường xuyên cần một giao dịch trải qua cả hai dịch vụ để giữ dữ liệu đúng. Dấu hiệu thứ tư là hai dịch vụ dùng chung một cơ sở dữ liệu, vì lúc đó chúng dính vào nhau qua lược đồ dù mã nguồn đã tách.

Khi thấy các dấu hiệu này, phản ứng đúng là gộp lại hoặc vẽ lại ranh giới, chứ không phải thêm công cụ để chịu đựng chúng. Một ví dụ kinh điển là chúng ta tách dịch vụ Đơn hàng và dịch vụ Dòng hàng thành hai dịch vụ riêng. Hai thứ đó không phải là hai năng lực nghiệp vụ, chúng chỉ là hai thực thể trong cùng một ngữ cảnh đặt hàng. Vì vậy chúng luôn đổi cùng nhau, luôn gọi nhau và luôn cần giao dịch chung, và việc tách chúng chỉ mang lại chi phí mạng. Nếu chúng ta cố sống chung với ranh giới sai đó, hậu quả là chúng ta phải dựng cả một bộ máy phức tạp để giải quyết một vấn đề mà lẽ ra không nên tồn tại.

**English (bám cấu trúc tiếng Việt)**

There are four signs that help us recognise a boundary that has been drawn wrongly, and we should know all four by heart. The first sign is that two services are always changed and deployed together, that is, they have never actually been separate in practice. The second sign is that they call each other constantly for one simple operation, and people call that style of communication chatty. The third sign is that we frequently need a transaction spanning both services in order to keep the data correct. The fourth sign is that two services share one database, because at that point they are stuck to each other through the schema even though the code has been split.

When we see these signs, the correct reaction is to merge them or redraw the boundary, rather than to add tools in order to endure them. A classic example is that we split an Order service and an Order Item service into two separate services. Those two things are not two business capabilities, they are just two entities inside the same ordering context. Therefore they always change together, always call each other and always need a shared transaction, and splitting them brings nothing but network cost. If we try to live with that wrong boundary, the consequence is that we have to build a whole complex machinery to solve a problem that should not have existed.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| chưa bao giờ tách rời trên thực tế | have never actually been separate in practice |
| gọi chéo nhau liên tục | call each other constantly |
| người ta gọi kiểu giao tiếp đó là lắm lời | people call that style of communication chatty |
| một giao dịch trải qua cả hai dịch vụ | a transaction spanning both services |
| chúng dính vào nhau qua lược đồ | they are stuck to each other through the schema |
| thêm công cụ để chịu đựng chúng | add tools in order to endure them |
| chúng chỉ là hai thực thể | they are just two entities |
| chỉ mang lại chi phí mạng | brings nothing but network cost |
| cố sống chung với ranh giới sai đó | try to live with that wrong boundary |
| một vấn đề mà lẽ ra không nên tồn tại | a problem that should not have existed |

**Thuật ngữ cần nhớ**

- lắm lời → **chatty**
- gọi chéo → **a cross-service call**
- dùng chung cơ sở dữ liệu → **a shared database**
- thực thể → **an entity**
- mẫu phản diện → **an anti-pattern**

---

## Phần 4 — Mỗi dịch vụ một cơ sở dữ liệu

**Tiếng Việt**

Khi đã tách dịch vụ thật, chúng ta cho mỗi dịch vụ giữ cơ sở dữ liệu riêng và không cho dịch vụ khác đọc thẳng vào đó. Lợi ích đầu tiên là tính tự chủ, vì một đội đổi lược đồ của mình mà không phải xin phép đội khác. Lợi ích thứ hai là triển khai độc lập, vì thay đổi bên trong không rò rỉ ra ngoài qua bảng dữ liệu. Nếu chúng ta cho nhiều dịch vụ dùng chung một cơ sở dữ liệu, chúng ta tạo ra một sự dính kết ngầm mà không ai nhìn thấy trong mã nguồn. Khi đó, một lần đổi tên cột có thể làm hỏng một dịch vụ mà tác giả của thay đổi thậm chí chưa từng đọc mã.

Cái giá của lựa chọn này khá lớn và chúng ta phải nói ra. Thứ nhất, chúng ta mất phép nối bảng giữa hai miền dữ liệu, nên việc ghép dữ liệu chuyển lên tầng ứng dụng hoặc chuyển sang một khung nhìn đọc riêng. Thứ hai, chúng ta phải đồng bộ dữ liệu giữa các dịch vụ, thường bằng sự kiện hoặc bằng luồng thu thập thay đổi từ nhật ký giao dịch. Thứ ba, tính nhất quán giữa các dịch vụ trở thành nhất quán cuối cùng, nên giao diện phải chịu được vài giây dữ liệu chưa khớp. Nếu chúng ta quên chuẩn bị cho ba điều này, hậu quả là mỗi báo cáo tổng hợp đều biến thành một dự án nhỏ, và người dùng thì thấy hai màn hình hiển thị hai con số khác nhau.

**English (bám cấu trúc tiếng Việt)**

Once we have really split the services, we let each service keep its own database and we do not let another service read directly into it. The first benefit is autonomy, because one team changes its own schema without having to ask permission from another team. The second benefit is independent deployment, because internal changes do not leak outside through the data tables. If we let several services share one database, we create a hidden coupling that nobody can see in the source code. At that point, one column rename can break a service whose code the author of the change has never even read.

The price of this choice is quite large and we have to say it out loud. First, we lose joins between two data domains, so joining data moves up to the application layer or moves into a separate read model. Second, we have to synchronise data between the services, usually through events or through a change-data-capture stream from the transaction log. Third, consistency between services becomes eventual consistency, so the interface has to tolerate a few seconds of mismatched data. If we forget to prepare for these three things, the consequence is that every aggregate report turns into a small project, and users see two screens showing two different numbers.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| không cho dịch vụ khác đọc thẳng vào đó | we do not let another service read directly into it |
| mà không phải xin phép đội khác | without having to ask permission from another team |
| không rò rỉ ra ngoài qua bảng dữ liệu | do not leak outside through the data tables |
| một sự dính kết ngầm | a hidden coupling |
| một lần đổi tên cột | one column rename |
| chuyển sang một khung nhìn đọc riêng | moves into a separate read model |
| phải chịu được vài giây dữ liệu chưa khớp | has to tolerate a few seconds of mismatched data |
| mỗi báo cáo tổng hợp đều biến thành một dự án nhỏ | every aggregate report turns into a small project |
| hai màn hình hiển thị hai con số khác nhau | two screens showing two different numbers |

**Thuật ngữ cần nhớ**

- mỗi dịch vụ một cơ sở dữ liệu → **database per service**
- triển khai độc lập → **independent deployment**
- khung nhìn đọc → **a read model**
- thu thập thay đổi dữ liệu → **change data capture (CDC)**
- sự kiện miền → **a domain event**

---

## Phần 5 — Saga thay cho giao dịch phân tán

**Tiếng Việt**

Khi một nghiệp vụ phải chạm vào hai dịch vụ, chúng ta không dùng giao dịch phân tán kiểu cam kết hai pha. Cách đó mong manh và chậm trong hệ phân tán, vì nó bắt mọi bên giữ khoá trong lúc chờ một bên điều phối, và nếu bên điều phối chết thì các bên còn lại treo. Thay vào đó, chúng ta dùng saga, tức là chúng ta chia nghiệp vụ thành một chuỗi bước cục bộ, và mỗi bước tự cam kết ngay trong dịch vụ của nó. Nếu một bước ở giữa thất bại, chúng ta không quay lui theo kiểu cơ sở dữ liệu, mà chúng ta chạy các hành động bù trừ để đảo ngược những bước đã xong. Ví dụ, chúng ta đã trừ tồn kho rồi thanh toán thất bại, thì bước bù trừ là trả lại số lượng đã trừ.

Cách làm này đòi hỏi chúng ta suy nghĩ về nghiệp vụ chứ không chỉ về kỹ thuật. Với mỗi bước, chúng ta phải trả lời được câu hỏi rằng nếu bước này đã xong mà bước sau hỏng thì chúng ta bù trừ như thế nào. Có những hành động không đảo ngược được, ví dụ chúng ta đã gửi thư điện tử cho khách, nên chúng ta phải sắp xếp lại thứ tự để đưa các bước không đảo ngược xuống cuối. Chúng ta cũng phải làm cho mỗi bước bất biến khi lặp lại, vì hàng đợi có thể giao cùng một thông điệp hai lần. Nếu chúng ta bỏ qua tính bất biến này, hậu quả là một khách hàng bị trừ tiền hai lần, và đó là loại lỗi mà không lời xin lỗi nào chữa lành hoàn toàn.

**English (bám cấu trúc tiếng Việt)**

When one business flow has to touch two services, we do not use a distributed transaction of the two-phase commit kind. That approach is fragile and slow in a distributed system, because it makes every party hold locks while waiting for a coordinator, and if the coordinator dies then the remaining parties hang. Instead, we use a saga, that is, we break the flow into a chain of local steps, and each step commits on its own inside its own service. If a step in the middle fails, we do not roll back in the database sense, we run compensating actions in order to undo the steps that have already finished. For example, we have already deducted the inventory and then the payment fails, so the compensating step is to return the quantity we deducted.

This approach requires us to think about the business rather than only about the technology. For each step, we must be able to answer the question of how we compensate if this step has finished but the next step breaks. There are actions that cannot be undone, for example we have already sent an email to the customer, so we have to reorder the steps in order to push the irreversible ones to the end. We also have to make each step idempotent, because the queue may deliver the same message twice. If we skip this idempotency, the consequence is that one customer is charged twice, and that is the kind of bug which no apology fully heals.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| giao dịch phân tán kiểu cam kết hai pha | a distributed transaction of the two-phase commit kind |
| mong manh và chậm | fragile and slow |
| giữ khoá trong lúc chờ một bên điều phối | hold locks while waiting for a coordinator |
| một chuỗi bước cục bộ | a chain of local steps |
| mỗi bước tự cam kết ngay | each step commits on its own |
| chúng ta không quay lui theo kiểu cơ sở dữ liệu | we do not roll back in the database sense |
| các hành động bù trừ | compensating actions |
| có những hành động không đảo ngược được | there are actions that cannot be undone |
| đưa các bước không đảo ngược xuống cuối | push the irreversible ones to the end |
| giao cùng một thông điệp hai lần | deliver the same message twice |
| không lời xin lỗi nào chữa lành hoàn toàn | no apology fully heals |

**Thuật ngữ cần nhớ**

- cam kết hai pha → **two-phase commit (2PC)**
- hành động bù trừ → **a compensating action**
- bất biến khi lặp lại → **idempotent**
- quay lui → **to roll back**
- bên điều phối → **a coordinator**

---

## Phần 6 — Khối đơn mô-đun là điểm khởi đầu tốt

**Tiếng Việt**

Khối đơn mô-đun là cách chúng ta lấy được lợi ích của ranh giới rõ ràng mà chưa phải trả chi phí của hệ phân tán. Chúng ta giữ mọi thứ trong một lần triển khai duy nhất, nhưng chúng ta chia mã nguồn thành các mô-đun theo đúng các ngữ cảnh nghiệp vụ. Quy tắc quan trọng nhất là các mô-đun chỉ gọi nhau qua giao diện công khai, và không mô-đun nào được đụng vào bảng dữ liệu của mô-đun khác. Nếu chúng ta giữ được quy tắc đó, mỗi mô-đun trở thành một đường cắt đã sẵn sàng để bê ra thành dịch vụ riêng. Nhờ vậy, việc tách sau này mất vài ngày chứ không mất vài tháng.

Cách làm này đặc biệt hợp với đội nhỏ và với sản phẩm ở giai đoạn sớm, khi ranh giới nghiệp vụ còn chưa rõ. Ở giai đoạn đó, chúng ta vẽ ranh giới sai là chuyện bình thường, và sửa một ranh giới sai trong một khối mã là việc dễ. Ngược lại, sửa một ranh giới sai giữa hai dịch vụ đã tách là việc rất tốn kém, vì chúng ta phải dời dữ liệu và phải phối hợp hai lần triển khai. Nói cách khác, khối đơn mô-đun cho chúng ta quyền đổi ý với giá rẻ. Nếu chúng ta tách quá sớm, hậu quả là chúng ta đóng băng một quyết định vào lúc chúng ta hiểu về miền nghiệp vụ ít nhất.

**English (bám cấu trúc tiếng Việt)**

A modular monolith is how we get the benefits of clear boundaries without yet paying the cost of a distributed system. We keep everything in one single deployment, but we split the source code into modules that follow the business contexts exactly. The most important rule is that modules call each other only through a public interface, and no module is allowed to touch another module's data tables. If we can hold that rule, each module becomes a seam that is already prepared to be lifted out into its own service. Thanks to that, splitting later takes a few days rather than a few months.

This approach suits a small team and an early-stage product particularly well, when the business boundaries are still unclear. At that stage, drawing a wrong boundary is a normal thing, and fixing a wrong boundary inside one codebase is easy work. On the contrary, fixing a wrong boundary between two services that have already been split is very expensive work, because we have to move data and we have to coordinate two deployments. In other words, a modular monolith gives us the right to change our mind cheaply. If we split too early, the consequence is that we freeze a decision at the moment when we understand the business domain the least.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| mà chưa phải trả chi phí của hệ phân tán | without yet paying the cost of a distributed system |
| trong một lần triển khai duy nhất | in one single deployment |
| theo đúng các ngữ cảnh nghiệp vụ | that follow the business contexts exactly |
| không mô-đun nào được đụng vào | no module is allowed to touch |
| một đường cắt đã sẵn sàng để bê ra | a seam that is already prepared to be lifted out |
| vẽ ranh giới sai là chuyện bình thường | drawing a wrong boundary is a normal thing |
| là việc rất tốn kém | is very expensive work |
| cho chúng ta quyền đổi ý với giá rẻ | gives us the right to change our mind cheaply |
| chúng ta đóng băng một quyết định | we freeze a decision |
| vào lúc chúng ta hiểu về miền nghiệp vụ ít nhất | at the moment when we understand the business domain the least |

**Thuật ngữ cần nhớ**

- khối đơn mô-đun → **a modular monolith**
- giao diện công khai → **a public interface**
- đường cắt → **a seam**
- miền nghiệp vụ → **the business domain**
- kho mã nguồn → **a codebase**

---

## Phần 7 — Tách dần bằng mẫu cây bóp cổ và quy luật Conway

**Tiếng Việt**

Khi cần tách một khối đơn đã lớn, chúng ta không viết lại từ đầu mà chúng ta tách dần theo mẫu cây bóp cổ. Cách làm là chúng ta đặt một lớp mặt tiền phía trước khối đơn, để mọi lưu lượng đi vào đều qua lớp đó. Sau đó chúng ta chuyển từng năng lực nghiệp vụ ra một dịch vụ mới, rồi trỏ lớp mặt tiền sang dịch vụ mới cho đúng phần đó. Chúng ta lặp lại nhiều vòng, và phần còn lại của khối đơn nhỏ dần cho tới khi nó biến mất hoặc chỉ còn phần ít thay đổi. Cách này an toàn hơn viết lại từ đầu, vì mỗi vòng đều nhỏ, đều đảo ngược được và đều giữ hệ thống chạy liên tục.

Chúng ta cũng nên nhớ quy luật Conway khi vẽ ranh giới, vì cơ cấu hệ thống có xu hướng phản chiếu cơ cấu tổ chức. Nếu công ty có ba đội độc lập, hệ thống sẽ tự nhiên trôi về ba mảnh dù chúng ta vẽ thế nào trên giấy. Vì vậy, cách khôn ngoan là chúng ta cho ranh giới dịch vụ trùng với ranh giới đội, để mỗi đội sở hữu trọn vẹn phần của mình. Nếu chúng ta cố áp một ranh giới đi ngược lại cách tổ chức đội, hậu quả là mọi thay đổi đều cần phối hợp giữa các đội, và ranh giới đó sẽ mòn dần rồi vỡ. Nói cách khác, chúng ta thiết kế tổ chức và thiết kế hệ thống trong cùng một lần suy nghĩ.

**English (bám cấu trúc tiếng Việt)**

When we need to split a monolith that has already grown large, we do not rewrite it from scratch, we split it gradually using the strangler fig pattern. The way it works is that we put a facade layer in front of the monolith, so that all incoming traffic goes through that layer. Then we move one business capability at a time out into a new service, and we point the facade at the new service for that part. We repeat this over many rounds, and the remaining part of the monolith shrinks until it disappears or until only the rarely-changing part is left. This way is safer than rewriting from scratch, because each round is small, each round is reversible and each round keeps the system running continuously.

We should also remember Conway's law when we draw boundaries, because the structure of a system tends to mirror the structure of the organisation. If the company has three independent teams, the system will naturally drift towards three pieces no matter how we draw it on paper. Therefore, the wise approach is that we let the service boundaries line up with the team boundaries, so that each team fully owns its own part. If we try to impose a boundary that goes against the way the teams are organised, the consequence is that every change needs coordination between teams, and that boundary will erode and then break. In other words, we design the organisation and design the system in the same act of thinking.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| chúng ta không viết lại từ đầu | we do not rewrite it from scratch |
| tách dần theo mẫu cây bóp cổ | split it gradually using the strangler fig pattern |
| đặt một lớp mặt tiền phía trước | put a facade layer in front |
| chuyển từng năng lực nghiệp vụ ra | move one business capability at a time out |
| nhỏ dần cho tới khi nó biến mất | shrinks until it disappears |
| đều đảo ngược được | is reversible |
| có xu hướng phản chiếu cơ cấu tổ chức | tends to mirror the structure of the organisation |
| sẽ tự nhiên trôi về ba mảnh | will naturally drift towards three pieces |
| cho ranh giới dịch vụ trùng với ranh giới đội | let the service boundaries line up with the team boundaries |
| ranh giới đó sẽ mòn dần rồi vỡ | that boundary will erode and then break |

**Thuật ngữ cần nhớ**

- mẫu cây bóp cổ → **the strangler fig pattern**
- lớp mặt tiền → **a facade layer**
- viết lại từ đầu → **a rewrite from scratch**
- quy luật Conway → **Conway's law**
- quyền sở hữu → **ownership**

---

## Mô hình ghi nhớ

**Tiếng Việt**

Chúng ta chia theo cái thay đổi cùng nhau, chứ không chia theo tầng kỹ thuật. Chúng ta bắt đầu bằng khối đơn mô-đun, và chúng ta chỉ tách thành dịch vụ riêng khi đường cắt đã rõ và khi có một lý do thật.

**English (bám cấu trúc tiếng Việt)**

We split by what changes together, we do not split by technical layer. We start with a modular monolith, and we only split into separate services when the seam is clear and when there is a real reason.

---

## Bảng thuật ngữ tổng hợp

| Tiếng Việt | Tiếng Anh | Ghi chú phát âm |
|---|---|---|
| phân rã | decomposition | đi-com-pơ-**ZI**-shợn, trọng âm âm thứ tư |
| ranh giới dịch vụ | a service boundary | *boundary* — "**BAUN**-đơ-ri", trọng âm đầu |
| chức năng nghiệp vụ | a business function | *business* — hai âm tiết "**BIZ**-nịs", không đọc ba âm |
| tầng kỹ thuật | a technical layer | |
| lần phát hành | a release | ri-**LIIS**, trọng âm âm thứ hai |
| năng lực nghiệp vụ | a business capability | *capability* — kêi-pơ-**BI**-lơ-ti, trọng âm âm thứ ba |
| ngữ cảnh có ranh giới | a bounded context | |
| thiết kế hướng miền | domain-driven design (DDD) | *domain* — đơ-**MÊIN**, trọng âm âm thứ hai |
| tính tự chủ | autonomy | ô-**TO**-nơ-mi, trọng âm âm thứ hai |
| độ dính kết | coupling | "**CĂP**-ling" |
| độ gắn kết bên trong | cohesion | câu-**HII**-zhợn, âm giữa là /ʒ/ |
| lắm lời | chatty | |
| gọi chéo | a cross-service call | |
| dùng chung cơ sở dữ liệu | a shared database | |
| thực thể | an entity | "**EN**-tơ-ti", trọng âm đầu |
| mẫu phản diện | an anti-pattern | |
| mỗi dịch vụ một cơ sở dữ liệu | database per service | |
| triển khai độc lập | independent deployment | *independent* — in-đi-**PEN**-đợnt, trọng âm âm thứ ba |
| khung nhìn đọc | a read model | |
| thu thập thay đổi dữ liệu | change data capture (CDC) | |
| sự kiện miền | a domain event | |
| lược đồ | a schema | /ˈskiːmə/ — "SKI-mơ", không đọc "sê-ma" |
| phép nối bảng | a join | |
| cam kết hai pha | two-phase commit (2PC) | *phase* /feɪz/ — "phêiz", không đọc "pha-se" |
| hành động bù trừ | a compensating action | "**COM**-pần-sêi-ting", trọng âm đầu |
| bất biến khi lặp lại | idempotent | ai-**DEM**-pơ-tần, trọng âm âm thứ hai |
| quay lui | to roll back | |
| bên điều phối | a coordinator | câu-**O**-đi-nêi-tơ, trọng âm âm thứ hai |
| khối đơn | a monolith | "**MO**-nơ-lith", âm **th** cuối /θ/ |
| khối đơn mô-đun | a modular monolith | *modular* — "**MO**-điu-lơ", trọng âm đầu |
| giao diện công khai | a public interface | |
| đường cắt | a seam | /siːm/ — đọc như *seem* |
| miền nghiệp vụ | the business domain | |
| kho mã nguồn | a codebase | |
| mẫu cây bóp cổ | the strangler fig pattern | *strangler* — "**STRÂNG**-glơ", có cụm **str-** đầu |
| lớp mặt tiền | a facade layer | /fəˈsɑːd/ — "phơ-**SAAD**", chữ *c* đọc là "s" |
| viết lại từ đầu | a rewrite from scratch | *scratch* — âm cuối **-tch**, không thành "scrát" |
| quy luật Conway | Conway's law | |
| quyền sở hữu | ownership | |
| giao dịch | a transaction | |
| nhất quán cuối cùng | eventual consistency | *eventual* — i-**VEN**-chu-ợl, trọng âm âm thứ hai |

---

## Bài nói

> **Cách luyện:** bật ghi âm, nói **60–90 giây** cho mỗi đề, **không nhìn tài liệu** khi nói. Mỗi bài phải dùng ít nhất **sáu thuật ngữ** trong bảng trên. Với các đề phản biện, hãy nêu dấu hiệu cụ thể trước khi nêu kết luận — đó là cách nói của người đã tách dịch vụ thật.

1. **Explain to a junior engineer** why we split services by business capability instead of by technical layer, using a concrete change request to make the point.

2. **A colleague says:** *"Order and Order Item should be two separate microservices because they are two different entities."* **Explain what is wrong with that**, and name the signs you would look for to confirm it.

3. **Someone on your team wants to** let the reporting service read directly from the ordering service's database because it is faster to build. **Explain why you would push back**, and describe what you would offer instead.

4. **Describe what happens when** a checkout flow spans an inventory service and a payment service and the payment fails after the stock has been deducted. Walk through the compensating steps.

5. **When would you choose** a modular monolith over microservices, and what specific evidence would make you change that decision six months later?

6. **Explain to a manager** how you would break up a five-year-old monolith without a rewrite, and why the first release of that project will not remove any code.

7. **Explain to a junior engineer** what Conway's law says, and why you would take the team structure into account before drawing service boundaries.
