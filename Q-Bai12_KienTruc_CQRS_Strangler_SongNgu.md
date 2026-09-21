# Bài 12 — Kiến trúc (2): Anemic Model · Application/Domain Service · CQRS · Refactor Big-Ball-of-Mud
### Bản song ngữ: tiếng Anh bám cấu trúc tiếng Việt

> **Cách dùng bài này:** đọc đoạn tiếng Việt trước, tự dịch trong đầu, rồi mới nhìn sang đoạn tiếng Anh để so. Sau khi so xong, **đọc to đoạn tiếng Anh hai lần** — lần một để nhìn chữ, lần hai để nghe chính mình. Phần bài nói ở cuối là bắt buộc; không nói ra thì bài này chỉ là bài đọc.

---

## Phần 1 — Bốn câu hỏi khó của phần judgment kiến trúc

**Tiếng Việt**

Bài này là phần nâng cao của kiến trúc, và nó cũng chính là phần judgment. Chúng ta sẽ trả lời bốn câu hỏi khó, và câu thứ nhất là khi nào một kiến trúc clean đầy đủ mới đáng bỏ công làm. Câu thứ hai là anemic model có thật sự sai hay nó là một lựa chọn có chủ đích. Câu thứ ba là khi nào chúng ta nên tách phần đọc ra khỏi phần ghi bằng CQRS. Câu thứ tư là làm sao refactor một monolith rối mà chúng ta không đập đi xây lại từ đầu.

**English (bám cấu trúc tiếng Việt)**

This lesson is the advanced part of architecture, and it is also exactly the judgment part. We will answer four hard questions, and the first one is when a full clean architecture is actually worth the effort. The second one is whether the anaemic model is really wrong or whether it is a deliberate choice. The third one is when we should separate the read side from the write side with CQRS. The fourth one is how to refactor a messy monolith without tearing it down and rebuilding it from scratch.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| nó cũng chính là phần judgment | it is also exactly the judgment part |
| mới đáng bỏ công làm | is actually worth the effort |
| hay nó là một lựa chọn có chủ đích | or whether it is a deliberate choice |
| tách phần đọc ra khỏi phần ghi | separate the read side from the write side |
| một monolith rối | a messy monolith |
| đập đi xây lại từ đầu | tearing it down and rebuilding it from scratch |

**Thuật ngữ cần nhớ**

- đáng bỏ công → **worth the effort**
- có chủ đích → **deliberate**
- phía đọc / phía ghi → **the read side** / **the write side**
- rối, bừa bộn → **messy**
- làm lại từ đầu → **from scratch**

---

## Phần 2 — Anemic Domain Model và lý do Fowler gọi nó là anti-pattern

**Tiếng Việt**

Anemic Domain Model là tình trạng entity chỉ còn là một cái túi chứa getter và setter, còn mọi logic thì dồn hết vào service. Martin Fowler coi đây là một anti-pattern, và lý do của ông rất rõ ràng. Khi entity không giữ hành vi nào, chúng ta mất luôn encapsulation, tức là chúng ta mất đúng thứ mà chúng ta đã học ở Bài 1. Hậu quả cụ thể là không còn chỗ nào bảo vệ invariant, cho nên bất kỳ service nào cũng đặt được object vào một trạng thái sai. DDD thì muốn điều ngược lại, nghĩa là hành vi và invariant phải nằm ngay bên trong domain object. Ví dụ dễ nhớ là số dư tài khoản nên bị trừ bởi chính method rút tiền của tài khoản, chứ nó không nên bị gán từ một service bên ngoài.

**English (bám cấu trúc tiếng Việt)**

The Anaemic Domain Model is the situation where the entity is only a bag holding getters and setters, while all the logic is pushed into the service. Martin Fowler sees this as an anti-pattern, and his reason is very clear. When the entity holds no behaviour, we also lose encapsulation, that is, we lose exactly the thing that we learned in Lesson 1. The concrete consequence is that there is no place left protecting the invariant, so any service at all can put the object into a wrong state. DDD wants the opposite, which means that the behaviour and the invariant have to sit right inside the domain object. The example that is easy to remember is that the account balance should be reduced by the account's own withdraw method, and it should not be assigned from a service outside.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| chỉ còn là một cái túi chứa getter và setter | is only a bag holding getters and setters |
| mọi logic thì dồn hết vào service | all the logic is pushed into the service |
| lý do của ông rất rõ ràng | his reason is very clear |
| khi entity không giữ hành vi nào | when the entity holds no behaviour |
| không còn chỗ nào bảo vệ invariant | there is no place left protecting the invariant |
| bất kỳ service nào cũng đặt được object vào một trạng thái sai | any service at all can put the object into a wrong state |
| phải nằm ngay bên trong domain object | have to sit right inside the domain object |
| bị trừ bởi chính method rút tiền của tài khoản | reduced by the account's own withdraw method |
| không nên bị gán từ một service bên ngoài | should not be assigned from a service outside |

**Thuật ngữ cần nhớ**

- mô hình domain thiếu máu → **anaemic domain model**
- cái túi chứa → **a bag holding**
- dồn vào → **push into**
- gán giá trị → **assign**
- rút tiền → **withdraw**

---

## Phần 3 — Anemic vẫn là lựa chọn hợp lệ khi domain mỏng

**Tiếng Việt**

Tuy nhiên chúng ta không nên nói rằng anemic model luôn luôn sai. Đây là một đánh đổi có chủ đích, và trong nhiều tình huống nó hoàn toàn chấp nhận được. Với một hệ thống CRUD đơn giản, nơi mà nghiệp vụ chỉ gồm tạo, đọc, sửa và xoá, việc dựng một domain model giàu hành vi thường không đem lại gì. Với một đội đã quen kiểu layered service, entity mỏng cộng với service dày lại dễ đọc hơn và dễ bàn giao hơn. Rất nhiều đội dùng Nest đang làm đúng như vậy, và họ làm rất hiệu quả cho những domain mỏng. Cách trả lời phỏng vấn tốt là chúng ta gọi đúng tên anti-pattern trước, rồi sau đó chúng ta nêu điều kiện mà nó vẫn là một lựa chọn hợp lý.

**English (bám cấu trúc tiếng Việt)**

However we should not say that the anaemic model is always wrong. This is a deliberate trade-off, and in many situations it is completely acceptable. For a simple CRUD system, where the business only consists of creating, reading, updating and deleting, building a behaviour-rich domain model usually brings nothing. For a team that is already used to the layered service style, a thin entity together with a thick service is easier to read and easier to hand over. Very many teams using Nest are doing exactly this, and they are doing it very effectively for thin domains. The good way to answer in an interview is that we name the anti-pattern correctly first, and after that we state the conditions under which it is still a reasonable choice.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| chúng ta không nên nói rằng … luôn luôn sai | we should not say that … is always wrong |
| nó hoàn toàn chấp nhận được | it is completely acceptable |
| nơi mà nghiệp vụ chỉ gồm | where the business only consists of |
| một domain model giàu hành vi | a behaviour-rich domain model |
| thường không đem lại gì | usually brings nothing |
| entity mỏng cộng với service dày | a thin entity together with a thick service |
| dễ bàn giao hơn | easier to hand over |
| chúng ta gọi đúng tên anti-pattern trước | we name the anti-pattern correctly first |
| nêu điều kiện mà nó vẫn là một lựa chọn hợp lý | state the conditions under which it is still a reasonable choice |

**Thuật ngữ cần nhớ**

- chấp nhận được → **acceptable**
- mỏng / dày → **thin** / **thick**
- bàn giao → **hand over**
- hợp lý → **reasonable**
- hiệu quả → **effectively**

---

## Phần 4 — Ba vai: Controller, Application Service và Domain Service

**Tiếng Việt**

Bây giờ chúng ta phân định ba vai rất hay bị trộn lẫn với nhau. Controller là một adapter vào ra, nghĩa là nó nhận request và trả response, và nó phải mỏng. Application Service, còn gọi là use case service, lo việc điều phối luồng, quản lý transaction, và gọi ra bên ngoài qua các port. Điều quan trọng là Application Service không chứa rule nghiệp vụ lõi, bởi vì nó chỉ sắp xếp thứ tự các bước. Domain Service chứa rule nghiệp vụ liên quan đến nhiều aggregate, và nó tuyệt đối không chạm vào hạ tầng. Ví dụ trong một use case chuyển tiền, Application Service mở transaction rồi lấy hai tài khoản ra, còn Domain Service kiểm tra rằng hai tài khoản phải khác nhau và số tiền phải dương. Chúng ta cũng nên nói thêm rằng cách phân định này là quy ước của cộng đồng, chứ nó không phải một chuẩn tuyệt đối.

**English (bám cấu trúc tiếng Việt)**

Now let us separate three roles that are very often mixed up with each other. The controller is an input-output adapter, which means that it receives the request and returns the response, and it has to be thin. The application service, also called the use case service, takes care of orchestrating the flow, managing the transaction, and calling out through the ports. The important thing is that the application service does not hold the core business rules, because it only arranges the order of the steps. The domain service holds the business rules that involve several aggregates, and it absolutely does not touch the infrastructure. For example in a money transfer use case, the application service opens the transaction and then fetches the two accounts, while the domain service checks that the two accounts must be different and that the amount must be positive. We should also add that this way of separating is a community convention, and it is not an absolute standard.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| ba vai rất hay bị trộn lẫn với nhau | three roles that are very often mixed up with each other |
| một adapter vào ra | an input-output adapter |
| lo việc điều phối luồng | takes care of orchestrating the flow |
| gọi ra bên ngoài qua các port | calling out through the ports |
| nó chỉ sắp xếp thứ tự các bước | it only arranges the order of the steps |
| liên quan đến nhiều aggregate | that involve several aggregates |
| nó tuyệt đối không chạm vào hạ tầng | it absolutely does not touch the infrastructure |
| số tiền phải dương | the amount must be positive |
| là quy ước của cộng đồng | is a community convention |
| không phải một chuẩn tuyệt đối | is not an absolute standard |

**Thuật ngữ cần nhớ**

- vai trò → **role**
- điều phối → **orchestrate**
- giao dịch, phiên ghi → **transaction**
- cụm gốc tổng hợp → **aggregate**
- quy ước → **convention**

---

## Phần 5 — CQRS tách model đọc khỏi model ghi

**Tiếng Việt**

CQRS tách model ghi ra khỏi model đọc. Model ghi có nhiệm vụ giữ invariant, cho nên nó được thiết kế theo nghiệp vụ và nó thường rất chặt chẽ. Model đọc có nhiệm vụ trả dữ liệu thật nhanh, cho nên nó được tối ưu cho truy vấn và nó thường ở dạng đã dựng sẵn. Cách tách này đáng làm khi phần đọc và phần ghi rất khác nhau, hoặc khi hai phía chịu tải rất khác nhau, ví dụ đọc thì cực nhiều còn ghi thì phức tạp. Ví dụ thực tế là trang sản phẩm được đọc liên tục, trong khi luồng đặt hàng lại phải giữ rất nhiều ràng buộc.

Chúng ta phải nói rõ rằng CQRS là thừa khi hệ thống chỉ là CRUD đơn giản. Trong trường hợp đó, việc duy trì hai model chỉ làm tăng chi phí đồng bộ mà nó không đem lại lợi ích nào. Có một hiểu lầm rất phổ biến nữa mà chúng ta phải gỡ, đó là nhiều người nghĩ CQRS bắt buộc phải đi kèm Event Sourcing. Hai khái niệm đó độc lập với nhau, bởi vì CQRS chỉ nói về việc tách phần đọc khỏi phần ghi, còn Event Sourcing nói về cách lưu trạng thái dưới dạng một chuỗi sự kiện. Khi phỏng vấn, việc tách bạch hai khái niệm này là một điểm cộng rất rõ.

**English (bám cấu trúc tiếng Việt)**

CQRS separates the write model from the read model. The write model has the job of keeping the invariants, so it is designed around the business and it is usually very strict. The read model has the job of returning data very fast, so it is optimised for queries and it is usually in a pre-built shape. This separation is worth doing when the read side and the write side are very different, or when the two sides carry very different loads, for example reads are enormous while writes are complex. A real example is that the product page is read constantly, while the ordering flow has to keep a great many constraints.

We have to say clearly that CQRS is unnecessary when the system is only simple CRUD. In that case, maintaining two models only raises the synchronisation cost and it does not bring any benefit. There is one more very common misunderstanding that we have to clear up, which is that many people think CQRS must come together with Event Sourcing. Those two concepts are independent of each other, because CQRS only talks about separating the read side from the write side, while Event Sourcing talks about storing the state as a sequence of events. In an interview, keeping these two concepts apart is a very clear plus point.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| có nhiệm vụ giữ invariant | has the job of keeping the invariants |
| nó được thiết kế theo nghiệp vụ | it is designed around the business |
| nó được tối ưu cho truy vấn | it is optimised for queries |
| ở dạng đã dựng sẵn | in a pre-built shape |
| khi hai phía chịu tải rất khác nhau | when the two sides carry very different loads |
| đọc thì cực nhiều còn ghi thì phức tạp | reads are enormous while writes are complex |
| chỉ làm tăng chi phí đồng bộ | only raises the synchronisation cost |
| một hiểu lầm rất phổ biến nữa mà chúng ta phải gỡ | one more very common misunderstanding that we have to clear up |
| lưu trạng thái dưới dạng một chuỗi sự kiện | storing the state as a sequence of events |
| việc tách bạch hai khái niệm này | keeping these two concepts apart |

**Thuật ngữ cần nhớ**

- truy vấn → **query**
- dựng sẵn, hình chiếu dữ liệu → **projection** / **pre-built**
- tải → **load**
- đồng bộ → **synchronisation**
- nguồn sự kiện → **event sourcing**

---

## Phần 6 — Clean đầy đủ cho một microservice CRUD mỏng có đáng không

**Tiếng Việt**

Một câu hỏi rất thực tế là có đáng áp kiến trúc clean đầy đủ cho một microservice CRUD mỏng hay không. Chúng ta phải nhìn thẳng vào chi phí, bởi vì hexagonal thêm tầng, thêm interface, và thêm phần ánh xạ dữ liệu giữa các tầng. Chi phí của port, adapter và mapping chỉ đáng bỏ ra khi domain đủ phức tạp hoặc đủ biến động. Với một service chỉ đọc ghi vài bảng, cách làm đó là over-engineering đúng nghĩa. Chúng ta quyết định dựa trên ba yếu tố, đó là độ phức tạp của nghiệp vụ, vòng đời dự kiến của service, và năng lực cùng thói quen của đội. Câu trả lời tệ nhất trong phỏng vấn là chúng ta nói rằng lúc nào cũng nên dùng clean architecture, bởi vì câu đó cho thấy chúng ta chưa từng trả giá cho nó.

**English (bám cấu trúc tiếng Việt)**

A very practical question is whether it is worth applying a full clean architecture to a thin CRUD microservice. We have to look the cost straight in the face, because hexagonal adds layers, adds interfaces, and adds data mapping between the layers. The cost of ports, adapters and mapping is only worth paying when the domain is complex enough or volatile enough. For a service that only reads and writes a few tables, that approach is over-engineering in the true sense. We decide based on three factors, which are the complexity of the business, the expected lifetime of the service, and the skills and habits of the team. The worst answer in an interview is that we say clean architecture should always be used, because that sentence shows that we have never paid its price.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| có đáng áp … hay không | whether it is worth applying … or not |
| chúng ta phải nhìn thẳng vào chi phí | we have to look the cost straight in the face |
| thêm phần ánh xạ dữ liệu giữa các tầng | adds data mapping between the layers |
| chỉ đáng bỏ ra khi | is only worth paying when |
| đủ phức tạp hoặc đủ biến động | complex enough or volatile enough |
| là over-engineering đúng nghĩa | is over-engineering in the true sense |
| vòng đời dự kiến của service | the expected lifetime of the service |
| năng lực cùng thói quen của đội | the skills and habits of the team |
| chúng ta chưa từng trả giá cho nó | we have never paid its price |

**Thuật ngữ cần nhớ**

- ánh xạ dữ liệu → **data mapping**
- hay biến động → **volatile**
- yếu tố → **factor**
- vòng đời dự kiến → **expected lifetime**
- trả giá → **pay the price**

---

## Phần 7 — Lộ trình refactor một monolith rối, từng bước một

**Tiếng Việt**

Bây giờ chúng ta bàn về một monolith đã trở thành big ball of mud, nghĩa là các tầng trộn lẫn hết vào nhau. Điều đầu tiên phải nói là chúng ta không viết lại toàn bộ, bởi vì một cuộc rewrite kiểu big bang mang rủi ro cực cao. Lộ trình an toàn gồm bốn bước, và chúng ta đi tuần tự từng bước một. Bước một là chúng ta viết characterization test để chụp lại hành vi hiện tại, và bộ test đó chính là lưới an toàn của cả quá trình. Bước hai là chúng ta bọc các ranh giới bằng interface để tạo ra seam, tức là tạo ra chỗ mà chúng ta cắt và thay thế được. Bước ba là chúng ta tách domain ra dần và đẩy phần vào ra sang các adapter.

**English (bám cấu trúc tiếng Việt)**

Now let us talk about a monolith that has become a big ball of mud, which means that the layers are all mixed into each other. The first thing to say is that we do not rewrite the whole thing, because a big-bang rewrite carries an extremely high risk. The safe road map has four steps, and we go through them one step at a time. Step one is that we write characterization tests to capture the current behaviour, and that test suite is exactly the safety net of the whole process. Step two is that we wrap the boundaries with interfaces in order to create seams, that is, to create places where we can cut and replace. Step three is that we split the domain out gradually and push the input and output parts into adapters.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| các tầng trộn lẫn hết vào nhau | the layers are all mixed into each other |
| chúng ta không viết lại toàn bộ | we do not rewrite the whole thing |
| mang rủi ro cực cao | carries an extremely high risk |
| lộ trình an toàn gồm bốn bước | the safe road map has four steps |
| chúng ta đi tuần tự từng bước một | we go through them one step at a time |
| để chụp lại hành vi hiện tại | to capture the current behaviour |
| bọc các ranh giới bằng interface | wrap the boundaries with interfaces |
| chỗ mà chúng ta cắt và thay thế được | places where we can cut and replace |
| tách domain ra dần | split the domain out gradually |

**Thuật ngữ cần nhớ**

- đống bùn nhão → **big ball of mud**
- viết lại một phát → **big-bang rewrite**
- lộ trình → **road map**
- chỗ cắt để thay thế → **seam**
- dần dần → **gradually**

---

## Phần 8 — Strangler fig, và ba thứ cần soát trong code AI

**Tiếng Việt**

Bước bốn là chúng ta áp dụng strangler fig, và đây là bước quyết định thành bại của cả lộ trình. Ý tưởng là chúng ta viết mọi tính năng mới theo kiến trúc mới, rồi chúng ta siết dần phần cũ lại cho tới khi nó bị thay hết. Cái tên này do Martin Fowler đặt, và nó lấy hình ảnh một loài cây bóp nghẹt thân cây chủ rồi chiếm chỗ của thân cây đó. Nhiều công ty lớn đã tách monolith theo đúng cách này, nghĩa là họ định tuyến tính năng mới sang service mới trong khi phần cũ vẫn chạy song song. Lợi ích lớn nhất là hệ thống luôn ở trạng thái chạy được, cho nên chúng ta dừng lại được ở bất kỳ bước nào mà chúng ta không mất tất cả.

Với code do AI sinh ra, bài này có ba thứ cần soát rất kỹ. Thứ nhất là AI hay sinh ra entity rỗng rồi dồn hết logic vào service, nghĩa là nó tạo ra anemic model theo mặc định. Thứ hai là AI hay đề xuất CQRS hoặc Event Sourcing cho một service CRUD mỏng, và lời đề xuất đó nghe rất kêu nhưng nó rất tốn kém. Thứ ba là khi chúng ta hỏi cách xử lý một monolith rối, AI hay đề xuất viết lại từ đầu. Khi review, chúng ta hỏi hai câu, đó là domain này có thật sự cần mức phức tạp đó hay không, và các rule nghiệp vụ đã nằm đúng tầng hay chưa.

**English (bám cấu trúc tiếng Việt)**

Step four is that we apply the strangler fig, and this is the step that decides whether the whole road map succeeds or fails. The idea is that we write every new feature in the new architecture, and then we tighten our grip on the old part until it is completely replaced. This name was given by Martin Fowler, and it takes the image of a tree that strangles the host trunk and then takes the place of that trunk. Many large companies have split their monoliths in exactly this way, which means that they route new features to the new service while the old part still runs in parallel. The biggest benefit is that the system is always in a working state, so we can stop at any step and we do not lose everything.

For code that AI generates, this lesson has three things that need very careful review. The first is that AI often produces empty entities and then pushes all the logic into the service, which means that it creates an anaemic model by default. The second is that AI often proposes CQRS or Event Sourcing for a thin CRUD service, and that proposal sounds very impressive but it is very expensive. The third is that when we ask how to deal with a messy monolith, AI often proposes rewriting from scratch. When we review, we ask two questions, which are whether this domain really needs that level of complexity, and whether the business rules already sit in the right layer.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| bước quyết định thành bại của cả lộ trình | the step that decides whether the whole road map succeeds or fails |
| chúng ta siết dần phần cũ lại | we tighten our grip on the old part |
| cho tới khi nó bị thay hết | until it is completely replaced |
| nó lấy hình ảnh một loài cây bóp nghẹt thân cây chủ | it takes the image of a tree that strangles the host trunk |
| họ định tuyến tính năng mới sang service mới | they route new features to the new service |
| phần cũ vẫn chạy song song | the old part still runs in parallel |
| luôn ở trạng thái chạy được | is always in a working state |
| lời đề xuất đó nghe rất kêu | that proposal sounds very impressive |
| có thật sự cần mức phức tạp đó hay không | whether it really needs that level of complexity |

**Thuật ngữ cần nhớ**

- cây bóp nghẹt → **strangler fig**
- định tuyến → **route**
- song song → **in parallel**
- đề xuất → **proposal**
- mặc định → **by default**

---

## Mô hình ghi nhớ

**Tiếng Việt**

Controller phải mỏng, Application Service điều phối và giữ transaction, Domain Service giữ rule liên nhiều aggregate, còn entity phải giữ hành vi để chúng ta không rơi vào anemic model. Khi gặp một monolith rối, chúng ta siết dần theo kiểu strangler fig, chứ chúng ta không đập đi xây lại.

**English (bám cấu trúc tiếng Việt)**

The controller must be thin, the application service orchestrates and holds the transaction, the domain service holds the rules across several aggregates, while the entity must hold behaviour so that we do not fall into the anaemic model. When we meet a messy monolith, we tighten our grip gradually in the strangler fig way, and we do not tear it down and rebuild it.

---

## Bảng thuật ngữ tổng hợp

| Tiếng Việt | Tiếng Anh | Ghi chú phát âm |
|---|---|---|
| đáng bỏ công | worth the effort | *effort* trọng âm đầu: E-ffort |
| có chủ đích | deliberate | tính từ /dɪˈlɪbərət/ — de-LI-be-rate, đuôi đọc nhẹ "-rợt" |
| phía đọc / phía ghi | the read side / the write side | *write* — chữ **w** câm, đọc là "rait" |
| rối, bừa bộn | messy | |
| làm lại từ đầu | from scratch | cụm **scr-** đầu và đuôi **-tch** đều phải bật |
| mô hình domain thiếu máu | anaemic domain model | /əˈniːmɪk/ — "ơ-NII-mic", trọng âm âm hai |
| cái túi chứa | a bag holding | |
| dồn vào | push into | |
| gán giá trị | assign | trọng âm cuối: a-SSIGN, chữ **g** câm |
| rút tiền | withdraw | có âm **th** ở giữa: with-DRAW |
| chấp nhận được | acceptable | trọng âm âm hai: ac-CEP-ta-ble |
| mỏng / dày | thin / thick | cả hai đều bắt đầu bằng âm **th** không rung |
| bàn giao | hand over | |
| hợp lý | reasonable | trọng âm đầu: REA-so-na-ble |
| hiệu quả | effectively | trọng âm âm hai: ef-FEC-tive-ly |
| vai trò | role | /rəʊl/ — đọc giống "roll" |
| điều phối | orchestrate | /ˈɔːkɪstreɪt/ — OR-kes-trate, **ch** đọc như "k" |
| giao dịch, phiên ghi | transaction | trọng âm âm hai: tran-SAC-tion |
| cụm gốc tổng hợp | aggregate | danh từ /ˈægrɪgət/ — AG-gre-gợt; động từ đuôi đọc "-geit" |
| quy ước | convention | trọng âm âm hai: con-VEN-tion |
| truy vấn | query | /ˈkwɪəri/ — "KWI-ri", cụm **qu-** đọc "kw" |
| hình chiếu dữ liệu | projection | trọng âm âm hai: pro-JEC-tion |
| tải | load | |
| đồng bộ | synchronisation | syn-chro-ni-SA-tion, **ch** đọc như "k" |
| nguồn sự kiện | event sourcing | *event* trọng âm cuối: e-VENT |
| ánh xạ dữ liệu | data mapping | |
| hay biến động | volatile | Anh đọc /ˈvɒlətaɪl/ — VO-la-tile, đuôi vần với "tile" |
| yếu tố | factor | |
| vòng đời dự kiến | expected lifetime | |
| trả giá | pay the price | |
| đống bùn nhão | big ball of mud | |
| viết lại một phát | big-bang rewrite | |
| lộ trình | road map | |
| chỗ cắt để thay thế | seam | /siːm/ — đọc giống "seem" |
| dần dần | gradually | trọng âm đầu: GRA-du-al-ly |
| cây bóp nghẹt | strangler fig | *strangler* — cụm **str-** khó: "STRANG-glơ" |
| định tuyến | route | người Anh đọc /ruːt/ giống "root"; người Mỹ đọc /raʊt/ |
| song song | in parallel | *parallel* trọng âm đầu: PA-ra-llel |
| đề xuất | proposal | trọng âm âm hai: pro-PO-sal |
| mặc định | by default | *default* trọng âm cuối: de-FAULT |

---

## Bài nói

> **Cách làm:** ghi âm lại giọng của mình, mỗi câu nói liên tục **60–90 giây**, **không nhìn tài liệu** khi nói. Mỗi bài nói phải dùng ít nhất **6 thuật ngữ** trong bảng ở trên. Nghe lại bản ghi âm một lần, ghi ra hai chỗ mình bị vấp, rồi nói lại lần hai.

1. Explain what an anaemic domain model is, why Fowler calls it an anti-pattern, and when you would still accept it on purpose.
2. Draw the line between a controller, an application service and a domain service, using a money transfer as your example.
3. A colleague proposes CQRS and Event Sourcing for a small internal CRUD service "so that it scales later". Explain why you would push back and what you would ask them first.
4. Explain what CQRS actually separates, and why it does not have to come with Event Sourcing.
5. Someone on your team says the legacy monolith is beyond saving and wants a full rewrite. Explain why you would push back and describe the incremental path you would propose instead.
6. When is a full hexagonal architecture worth its cost, and when would you deliberately stay with a simple layered service? Give the trade-off in both directions.
7. You are reviewing AI-generated code where every entity is a plain data holder and every rule lives in the service. Describe out loud what you would ask about, and which one or two rules you would move first.
