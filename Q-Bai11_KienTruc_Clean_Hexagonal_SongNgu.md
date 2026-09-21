# Bài 11 — Kiến trúc (1): Layered · Clean · Hexagonal · Onion · Dependency Rule · Screaming
### Bản song ngữ: tiếng Anh bám cấu trúc tiếng Việt

> **Cách dùng bài này:** đọc đoạn tiếng Việt trước, tự dịch trong đầu, rồi mới nhìn sang đoạn tiếng Anh để so. Sau khi so xong, **đọc to đoạn tiếng Anh hai lần** — lần một để nhìn chữ, lần hai để nghe chính mình. Phần bài nói ở cuối là bắt buộc; không nói ra thì bài này chỉ là bài đọc.

---

## Phần 1 — Bài này là DIP được nâng lên cấp kiến trúc

**Tiếng Việt**

Bài này chính là DIP của Bài 6 được nâng lên cấp kiến trúc. Ý tưởng cốt lõi vẫn là tách domain ra khỏi hạ tầng và đảo chiều phụ thuộc để mọi mũi tên đều chĩa vào trong. Khi học xong, chúng ta sẽ đọc được vì sao một dòng import client của ORM nằm trong entity là sai, và vì sao interface của repository lại phải nằm ở tầng domain. Bài 12 sẽ lo phần nâng cao, gồm anemic model, CQRS và cách gỡ một monolith rối. Đây cũng là phần mà người phỏng vấn cho vị trí senior hỏi kỹ nhất, bởi vì nó cho thấy chúng ta có nghĩ ở cấp hệ thống hay chúng ta chỉ nghĩ ở cấp file.

**English (bám cấu trúc tiếng Việt)**

This lesson is exactly the DIP of Lesson 6 raised to the architecture level. The core idea is still to separate the domain from the infrastructure and to invert the dependencies so that every arrow points inward. When we finish, we will be able to read why one line that imports an ORM client inside an entity is wrong, and why the repository interface has to sit in the domain layer. Lesson 12 will take care of the advanced part, which includes the anaemic model, CQRS and how to untangle a messy monolith. This is also the part that interviewers for a senior position probe the most deeply, because it shows whether we think at the system level or we only think at the file level.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| được nâng lên cấp kiến trúc | raised to the architecture level |
| để mọi mũi tên đều chĩa vào trong | so that every arrow points inward |
| chúng ta sẽ đọc được vì sao | we will be able to read why |
| cách gỡ một monolith rối | how to untangle a messy monolith |
| người phỏng vấn … hỏi kỹ nhất | interviewers … probe the most deeply |
| nghĩ ở cấp hệ thống | think at the system level |
| chúng ta chỉ nghĩ ở cấp file | we only think at the file level |

**Thuật ngữ cần nhớ**

- hạ tầng → **infrastructure**
- chĩa vào trong → **point inward**
- gỡ rối → **untangle**
- khối một mảnh → **monolith**
- đào sâu, hỏi kỹ → **probe**

---

## Phần 2 — Kiến trúc phân tầng và điểm yếu của nó

**Tiếng Việt**

Kiến trúc phân tầng tách trách nhiệm thành ba tầng quen thuộc, đó là tầng trình bày, tầng nghiệp vụ và tầng dữ liệu. Quy tắc của nó là phụ thuộc chỉ đi một chiều xuống dưới, nghĩa là tầng trên gọi tầng dưới còn tầng dưới không được gọi ngược lên. Lợi ích rõ nhất là khi chúng ta đổi một tầng, ví dụ chúng ta đổi database, thì các tầng khác ít bị ảnh hưởng. Đây là kiến trúc phổ biến nhất, và nó cũng là kiến trúc mà hầu hết chúng ta bắt đầu. Tuy nhiên nó có một điểm yếu mà chúng ta phải nhìn thấy, đó là tầng nghiệp vụ vẫn phụ thuộc vào tầng dữ liệu theo đúng chiều mũi tên. Chính điểm yếu đó là thứ mà Clean Architecture sinh ra để sửa.

**English (bám cấu trúc tiếng Việt)**

The layered architecture splits responsibilities into three familiar layers, which are the presentation layer, the business layer and the data layer. Its rule is that dependencies only go one way downward, which means that the upper layer calls the lower layer while the lower layer is not allowed to call back up. The clearest benefit is that when we change one layer, for example we change the database, the other layers are barely affected. This is the most common architecture, and it is also the architecture that most of us start with. However it has one weakness that we have to see, which is that the business layer still depends on the data layer along the direction of the arrow. That weakness is exactly the thing that Clean Architecture was born to fix.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| tách trách nhiệm thành ba tầng quen thuộc | splits responsibilities into three familiar layers |
| phụ thuộc chỉ đi một chiều xuống dưới | dependencies only go one way downward |
| tầng dưới không được gọi ngược lên | the lower layer is not allowed to call back up |
| các tầng khác ít bị ảnh hưởng | the other layers are barely affected |
| kiến trúc mà hầu hết chúng ta bắt đầu | the architecture that most of us start with |
| một điểm yếu mà chúng ta phải nhìn thấy | one weakness that we have to see |
| theo đúng chiều mũi tên | along the direction of the arrow |
| là thứ mà Clean Architecture sinh ra để sửa | is exactly the thing that Clean Architecture was born to fix |

**Thuật ngữ cần nhớ**

- kiến trúc phân tầng → **layered architecture**
- tầng trình bày → **presentation layer**
- một chiều → **one way**
- điểm yếu → **weakness**
- bị ảnh hưởng → **be affected**

---

## Phần 3 — The Dependency Rule: mọi phụ thuộc trỏ vào trong

**Tiếng Việt**

The Dependency Rule của Clean Architecture nói rằng phụ thuộc chỉ được trỏ vào trong, tức là trỏ về phía policy và domain. Điều đó có nghĩa là phần domain và phần nghiệp vụ không được phụ thuộc vào framework, không phụ thuộc vào database, và không phụ thuộc vào tầng web. Lý do rất thực tế, bởi vì domain là phần ổn định nhất và cũng là phần có giá trị nhất của hệ thống. Khi chúng ta tách domain khỏi các chi tiết kỹ thuật, chúng ta test nó dễ hơn và chúng ta thay hạ tầng mà không phải đụng đến nghiệp vụ. Đây chính là khác biệt cốt lõi giữa một kiến trúc phân tầng ngây thơ và một kiến trúc clean, bởi vì ở kiểu ngây thơ thì nghiệp vụ trỏ sang dữ liệu, còn ở kiểu clean thì dữ liệu trỏ ngược về nghiệp vụ qua một interface. Nếu chúng ta nói được đúng một câu này trong phỏng vấn, chúng ta đã trả lời trọn phần lý thuyết của Clean Architecture.

**English (bám cấu trúc tiếng Việt)**

The Dependency Rule of Clean Architecture says that dependencies are only allowed to point inward, that is, to point towards the policy and the domain. That means that the domain part and the business part must not depend on the framework, must not depend on the database, and must not depend on the web layer. The reason is very practical, because the domain is the most stable part and it is also the most valuable part of the system. When we separate the domain from the technical details, we test it more easily and we replace the infrastructure without having to touch the business. This is exactly the core difference between a naive layered architecture and a clean architecture, because in the naive kind the business points at the data, while in the clean kind the data points back at the business through an interface. If we can say exactly this one sentence in an interview, we have fully answered the theory part of Clean Architecture.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| chỉ được trỏ vào trong | are only allowed to point inward |
| trỏ về phía policy và domain | to point towards the policy and the domain |
| không được phụ thuộc vào framework | must not depend on the framework |
| phần ổn định nhất và cũng là phần có giá trị nhất | the most stable part and also the most valuable part |
| mà không phải đụng đến nghiệp vụ | without having to touch the business |
| một kiến trúc phân tầng ngây thơ | a naive layered architecture |
| dữ liệu trỏ ngược về nghiệp vụ qua một interface | the data points back at the business through an interface |
| chúng ta đã trả lời trọn phần lý thuyết | we have fully answered the theory part |

**Thuật ngữ cần nhớ**

- quy tắc phụ thuộc → **the Dependency Rule**
- chính sách, luật lõi → **policy**
- ngây thơ, sơ khai → **naive**
- ổn định → **stable**
- lý thuyết → **theory**

---

## Phần 4 — Hexagonal: port, adapter, driving và driven

**Tiếng Việt**

Kiến trúc Hexagonal còn có một tên khác là Ports and Adapters, và cái tên thứ hai mô tả cơ chế của nó chính xác hơn. Port là một interface do chính domain định nghĩa, và nó mô tả thứ mà domain cần từ thế giới bên ngoài. Adapter là phần hiện thực cụ thể nối domain ra thế giới đó. Người ta chia adapter thành hai loại, loại thứ nhất là driving adapter, tức là thứ gọi vào ứng dụng, ví dụ controller HTTP, giao diện dòng lệnh, hoặc chính bài test. Loại thứ hai là driven adapter, tức là thứ mà ứng dụng gọi ra, ví dụ database, dịch vụ gửi mail, hoặc hàng đợi. Điểm đắt giá nhất của kiểu kiến trúc này là phần lõi không biết adapter nào đang cắm vào nó.

**English (bám cấu trúc tiếng Việt)**

The Hexagonal architecture also has another name, which is Ports and Adapters, and the second name describes its mechanism more accurately. A port is an interface defined by the domain itself, and it describes what the domain needs from the outside world. An adapter is the concrete implementation that connects the domain out to that world. People divide adapters into two kinds, the first kind is the driving adapter, that is, the thing that calls into the application, for example an HTTP controller, a command-line interface, or the test itself. The second kind is the driven adapter, that is, the thing that the application calls out to, for example the database, the mail service, or the queue. The most valuable point of this kind of architecture is that the core does not know which adapter is plugged into it.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| mô tả cơ chế của nó chính xác hơn | describes its mechanism more accurately |
| do chính domain định nghĩa | defined by the domain itself |
| thứ mà domain cần từ thế giới bên ngoài | what the domain needs from the outside world |
| phần hiện thực cụ thể nối domain ra thế giới đó | the concrete implementation that connects the domain out to that world |
| thứ gọi vào ứng dụng | the thing that calls into the application |
| giao diện dòng lệnh | a command-line interface |
| thứ mà ứng dụng gọi ra | the thing that the application calls out to |
| điểm đắt giá nhất của kiểu kiến trúc này | the most valuable point of this kind of architecture |
| adapter nào đang cắm vào nó | which adapter is plugged into it |

**Thuật ngữ cần nhớ**

- cổng do domain định nghĩa → **port**
- bộ nối bên ngoài → **adapter**
- phía gọi vào → **driving** / **primary**
- phía được gọi ra → **driven** / **secondary**
- cắm vào → **plug into**

---

## Phần 5 — Clean, Onion và Hexagonal là cùng một tư tưởng

**Tiếng Việt**

Rất nhiều người tranh cãi xem Clean, Onion và Hexagonal khác nhau ra sao. Về bản chất, ba cái tên đó cùng nói về một tư tưởng, đó là tách domain khỏi hạ tầng và đảo chiều phụ thuộc vào trong. Chúng chỉ khác nhau ở thuật ngữ, ở cách vẽ sơ đồ, và ở chỗ mỗi tác giả nhấn mạnh điều gì. Vì vậy chúng ta không nên tranh cãi về tên gọi, mà chúng ta nên nắm cho chắc cái nguyên tắc chung. Trong phỏng vấn, một câu trả lời gọn kiểu ba cái tên khác nhau cho cùng một nguyên tắc thường được đánh giá cao hơn một bài diễn giải dài.

**English (bám cấu trúc tiếng Việt)**

Very many people argue about how Clean, Onion and Hexagonal differ from each other. In essence, those three names talk about one and the same idea, which is separating the domain from the infrastructure and inverting the dependencies inward. They only differ in terminology, in the way the diagram is drawn, and in what each author emphasises. Therefore we should not argue about the names, but we should get a firm grip on the shared principle. In an interview, a short answer such as three different names for the same principle is usually valued more highly than a long explanation.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| tranh cãi xem … khác nhau ra sao | argue about how … differ from each other |
| về bản chất | in essence |
| cùng nói về một tư tưởng | talk about one and the same idea |
| ở cách vẽ sơ đồ | in the way the diagram is drawn |
| ở chỗ mỗi tác giả nhấn mạnh điều gì | in what each author emphasises |
| nắm cho chắc cái nguyên tắc chung | get a firm grip on the shared principle |
| được đánh giá cao hơn một bài diễn giải dài | is valued more highly than a long explanation |

**Thuật ngữ cần nhớ**

- về bản chất → **in essence**
- thuật ngữ → **terminology**
- nhấn mạnh → **emphasise**
- tác giả → **author**
- diễn giải → **explanation**

---

## Phần 6 — Vì sao port nằm ở domain còn adapter nằm ở hạ tầng

**Tiếng Việt**

Bây giờ chúng ta trả lời câu hỏi vì sao interface của repository lại nằm ở domain còn phần hiện thực lại nằm ở hạ tầng. Lý do là domain phải sở hữu abstraction, bởi vì chính domain mới biết nó cần gì từ kho dữ liệu. Khi hạ tầng hiện thực cái port đó, mũi tên phụ thuộc trỏ từ ngoài vào trong, và đó chính là DIP ở cấp kiến trúc. Domain không import bất cứ thứ gì từ hạ tầng, mà nó chỉ import abstraction của chính nó. Ví dụ cụ thể là file `OrderRepository` nằm trong thư mục domain, còn file `PrismaOrderRepository` nằm trong thư mục hạ tầng và nó hiện thực cái interface kia. Khi công ty đổi từ Prisma sang TypeORM, chúng ta chỉ thêm một adapter mới, và toàn bộ phần domain vẫn im lặng.

**English (bám cấu trúc tiếng Việt)**

Now let us answer the question of why the repository interface sits in the domain while the implementation sits in the infrastructure. The reason is that the domain has to own the abstraction, because only the domain knows what it needs from the data store. When the infrastructure implements that port, the dependency arrow points from the outside inward, and that is exactly DIP at the architecture level. The domain does not import anything from the infrastructure, but it only imports its own abstraction. A concrete example is that the `OrderRepository` file sits in the domain folder, while the `PrismaOrderRepository` file sits in the infrastructure folder and it implements that interface. When the company moves from Prisma to TypeORM, we only add a new adapter, and the whole domain stays silent.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| domain phải sở hữu abstraction | the domain has to own the abstraction |
| chính domain mới biết nó cần gì | only the domain knows what it needs |
| từ kho dữ liệu | from the data store |
| mũi tên phụ thuộc trỏ từ ngoài vào trong | the dependency arrow points from the outside inward |
| nó chỉ import abstraction của chính nó | it only imports its own abstraction |
| nằm trong thư mục domain | sits in the domain folder |
| toàn bộ phần domain vẫn im lặng | the whole domain stays silent |

**Thuật ngữ cần nhớ**

- kho dữ liệu → **data store**
- sở hữu → **own**
- thư mục → **folder**
- hiện thực (interface) → **implement**
- im lặng, không đổi → **stay silent**

---

## Phần 7 — Vì sao một dòng import ORM trong entity là sai

**Tiếng Việt**

Có một câu hỏi thách đố rất hay gặp, đó là chúng ta nhìn thấy một dòng import client của Prisma nằm ngay trong một entity của domain, và người phỏng vấn hỏi chúng ta sai ở chỗ nào. Cái sai đầu tiên là domain đã rò rỉ phụ thuộc sang ORM, nghĩa là nó đang phụ thuộc vào một chi tiết hạ tầng. Cái sai thứ hai là mũi tên phụ thuộc bị đảo ngược so với Dependency Rule, bởi vì bây giờ phần lõi lại trỏ ra ngoài. Hậu quả thứ nhất là chúng ta rất khó test entity đó nếu chúng ta không dựng database lên, và chúng ta cũng rất khó thay công nghệ lưu trữ. Hậu quả thứ hai là logic nghiệp vụ bị trộn lẫn với logic lưu trữ, trong khi hai thứ đó vốn thay đổi vì hai lý do khác nhau. Cách sửa gồm hai bước, bước một là chúng ta định nghĩa một repository port ở domain, và bước hai là chúng ta viết một adapter Prisma ở tầng ngoài.

**English (bám cấu trúc tiếng Việt)**

There is a challenge question that we meet very often, which is that we see one line importing the Prisma client right inside a domain entity, and the interviewer asks us where the mistake is. The first mistake is that the domain has leaked a dependency onto the ORM, which means that it is depending on an infrastructure detail. The second mistake is that the dependency arrow is inverted compared with the Dependency Rule, because now the core points outward. The first consequence is that we find it very hard to test that entity if we do not stand a database up, and we also find it very hard to replace the storage technology. The second consequence is that the business logic gets mixed with the persistence logic, while those two things change for two different reasons. The fix has two steps, step one is that we define a repository port in the domain, and step two is that we write a Prisma adapter in the outer layer.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| người phỏng vấn hỏi chúng ta sai ở chỗ nào | the interviewer asks us where the mistake is |
| domain đã rò rỉ phụ thuộc sang ORM | the domain has leaked a dependency onto the ORM |
| bị đảo ngược so với Dependency Rule | is inverted compared with the Dependency Rule |
| bây giờ phần lõi lại trỏ ra ngoài | now the core points outward |
| nếu chúng ta không dựng database lên | if we do not stand a database up |
| bị trộn lẫn với logic lưu trữ | gets mixed with the persistence logic |
| vốn thay đổi vì hai lý do khác nhau | change for two different reasons |
| cách sửa gồm hai bước | the fix has two steps |

**Thuật ngữ cần nhớ**

- rò rỉ phụ thuộc → **leak a dependency**
- chi tiết hạ tầng → **infrastructure detail**
- lưu trữ lâu dài → **persistence**
- tầng ngoài → **the outer layer**
- công nghệ lưu trữ → **storage technology**

---

## Phần 8 — Screaming Architecture và cách bắt máy kiểm tra hộ

**Tiếng Việt**

Screaming Architecture là ý tưởng của Robert C. Martin về cách đặt tên thư mục. Ông cho rằng cấu trúc thư mục nên hét lên domain và use case, ví dụ chúng ta nhìn thấy thư mục `orders` và thư mục `billing`. Cấu trúc thư mục không nên hét lên tên framework hay tên tầng, ví dụ `controllers`, `services` và `models`. Lợi ích là khi một người mới mở cây thư mục lên, họ biết ngay hệ thống này làm gì, chứ họ không chỉ biết hệ thống được viết bằng gì. Cách tổ chức này cũng khớp với kiểu module theo tính năng mà nhiều framework hiện đại khuyến khích. Nếu thư mục của chúng ta chỉ nói về framework, thì mọi dự án trong công ty sẽ trông giống hệt nhau dù chúng phục vụ những nghiệp vụ rất khác nhau.

Điều cuối cùng và cũng thực tế nhất là chúng ta phải biến Dependency Rule thành thứ mà máy kiểm tra được. Chúng ta dùng công cụ như `dependency-cruiser` hoặc một plugin lint về ranh giới module để chặn việc thư mục domain import thư mục hạ tầng ngay trong CI. Việc này quan trọng vì đây đúng là chỗ mà AI viết đúng cú pháp nhưng sai kiến trúc, bởi vì AI hay nhét thẳng ORM hoặc lời gọi HTTP vào entity vì làm vậy thì code chạy được. Khi review, chúng ta hỏi một câu duy nhất, đó là domain có import bất cứ thứ gì từ hạ tầng hay không. Nếu câu trả lời là có, chúng ta yêu cầu tách port và adapter, và chúng ta thêm luôn một luật lint để lần sau máy bắt hộ chúng ta.

**English (bám cấu trúc tiếng Việt)**

Screaming Architecture is Robert C. Martin's idea about how to name folders. He argues that the folder structure should scream the domain and the use cases, for example we see an `orders` folder and a `billing` folder. The folder structure should not scream the name of the framework or the name of the layer, for example `controllers`, `services` and `models`. The benefit is that when a new person opens the folder tree, they know right away what this system does, and they do not only know what the system was written with. This way of organising also matches the feature-module style that many modern frameworks encourage. If our folders only talk about the framework, then every project in the company will look exactly the same although they serve very different businesses.

The last point and also the most practical one is that we have to turn the Dependency Rule into something a machine can check. We use tools such as `dependency-cruiser` or a lint plugin about module boundaries in order to block the domain folder from importing the infrastructure folder right inside CI. This matters because this is exactly the place where AI writes correct syntax but wrong architecture, because AI often puts an ORM or an HTTP call straight into an entity since doing so makes the code run. When we review, we ask a single question, which is whether the domain imports anything from the infrastructure or not. If the answer is yes, we ask for the port and the adapter to be separated, and we also add a lint rule so that next time the machine catches it for us.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| về cách đặt tên thư mục | about how to name folders |
| cấu trúc thư mục nên hét lên domain | the folder structure should scream the domain |
| khi một người mới mở cây thư mục lên | when a new person opens the folder tree |
| họ không chỉ biết hệ thống được viết bằng gì | they do not only know what the system was written with |
| kiểu module theo tính năng | the feature-module style |
| trông giống hệt nhau | look exactly the same |
| thành thứ mà máy kiểm tra được | into something a machine can check |
| một plugin lint về ranh giới module | a lint plugin about module boundaries |
| viết đúng cú pháp nhưng sai kiến trúc | writes correct syntax but wrong architecture |
| để lần sau máy bắt hộ chúng ta | so that next time the machine catches it for us |

**Thuật ngữ cần nhớ**

- cây thư mục → **folder tree**
- module theo tính năng → **feature module**
- ranh giới module → **module boundary**
- luật lint → **lint rule**
- tích hợp liên tục → **continuous integration (CI)**

---

## Mô hình ghi nhớ

**Tiếng Việt**

Domain nằm ở lõi và nó không biết gì về thế giới bên ngoài, cho nên mọi mũi tên phụ thuộc đều chĩa vào trong. Port là thứ domain cần, adapter là thứ thế giới cắm vào, và cây thư mục phải hét lên nghiệp vụ chứ không hét lên tên framework.

**English (bám cấu trúc tiếng Việt)**

The domain sits at the core and it knows nothing about the outside world, therefore every dependency arrow points inward. The port is what the domain needs, the adapter is what the world plugs in, and the folder tree must scream the business and not scream the name of the framework.

---

## Bảng thuật ngữ tổng hợp

| Tiếng Việt | Tiếng Anh | Ghi chú phát âm |
|---|---|---|
| hạ tầng | infrastructure | trọng âm đầu: IN-fra-struc-ture, cụm **-str-** phải bật |
| chĩa vào trong | point inward | |
| gỡ rối | untangle | trọng âm âm hai: un-TAN-gle |
| khối một mảnh | monolith | trọng âm đầu: MO-no-lith, đuôi có âm **th** |
| đào sâu, hỏi kỹ | probe | /prəʊb/ — "prâub", đuôi **-b** bật nhẹ |
| kiến trúc phân tầng | layered architecture | *layered* hai âm tiết: "LAY-ơd"; *architecture* — AR-chi-tec-ture, **ch** đầu đọc như "k" |
| tầng trình bày | presentation layer | *presentation* trọng âm áp chót: pre-sen-TA-tion |
| một chiều | one way | |
| điểm yếu | weakness | |
| bị ảnh hưởng | be affected | *affect* /əˈfekt/ — đừng lẫn với *effect* |
| quy tắc phụ thuộc | the Dependency Rule | *dependency* trọng âm âm hai: de-PEN-den-cy |
| chính sách, luật lõi | policy | trọng âm đầu: PO-li-cy |
| ngây thơ, sơ khai | naive | /naɪˈiːv/ — nai-EEV, trọng âm cuối |
| ổn định | stable | |
| lý thuyết | theory | /ˈθɪəri/ — có âm **th** đầu: "THI-ơ-ri" |
| kiến trúc lục giác | hexagonal | trọng âm âm hai: hex-A-go-nal |
| kiến trúc củ hành | onion | /ˈʌnjən/ — đọc là "AN-yơn", KHÔNG đọc "o-ni-on" |
| cổng do domain định nghĩa | port | |
| bộ nối bên ngoài | adapter | trọng âm âm hai: a-DAP-ter |
| phía gọi vào | driving / primary | *primary* Anh đọc /ˈpraɪməri/ — PRI-ma-ry |
| phía được gọi ra | driven / secondary | *driven* /ˈdrɪvn/ — "DRI-vn", nguyên âm ngắn |
| cắm vào | plug into | |
| giao diện dòng lệnh | command-line interface | |
| về bản chất | in essence | *essence* trọng âm đầu: E-ssence |
| thuật ngữ | terminology | trọng âm âm ba: ter-mi-NO-lo-gy |
| nhấn mạnh | emphasise | trọng âm đầu: EM-pha-sise |
| tác giả | author | /ˈɔːθə/ — có âm **th**: "AW-thơ" |
| diễn giải | explanation | trọng âm áp chót: ex-pla-NA-tion |
| kho dữ liệu | data store | |
| thư mục | folder | |
| hiện thực (interface) | implement | trọng âm đầu: IM-ple-ment |
| rò rỉ phụ thuộc | leak a dependency | |
| lưu trữ lâu dài | persistence | trọng âm âm hai: per-SIS-tence |
| tầng ngoài | the outer layer | |
| công nghệ lưu trữ | storage technology | *technology* trọng âm âm hai: tech-NO-lo-gy |
| cây thư mục | folder tree | |
| module theo tính năng | feature module | *feature* /ˈfiːtʃə/ — "FII-chơ" |
| ranh giới module | module boundary | *boundary* /ˈbaʊndri/ — BOUN-dry |
| luật lint | lint rule | |
| tích hợp liên tục | continuous integration | *continuous* trọng âm âm hai: con-TI-nu-ous |
| hét lên | scream | cụm **scr-** đầu từ khó, tập đọc chậm |

---

## Bài nói

> **Cách làm:** ghi âm lại giọng của mình, mỗi câu nói liên tục **60–90 giây**, **không nhìn tài liệu** khi nói. Mỗi bài nói phải dùng ít nhất **6 thuật ngữ** trong bảng ở trên. Nghe lại bản ghi âm một lần, ghi ra hai chỗ mình bị vấp, rồi nói lại lần hai.

1. State the Dependency Rule in your own words, and explain why the domain must not depend on the framework or the database.
2. Explain to a junior developer what a port and an adapter are, and give one driving adapter and one driven adapter from a system you have worked on.
3. A colleague imports the Prisma client inside a domain entity and says: "It works, and it saves a whole layer of code." Explain exactly what is wrong and what you would ask for instead.
4. Someone on your team wants to spend a meeting arguing whether the project follows Clean, Onion or Hexagonal. Explain why you would redirect that discussion and what you would focus on instead.
5. Describe what changes in the codebase when the company moves from Prisma to TypeORM in a properly layered hexagonal design.
6. When would you keep a simple layered architecture instead of going full hexagonal? Give the trade-off in both directions.
7. Your team keeps merging AI-generated code that imports infrastructure into the domain. Describe out loud what you would put in place so that the machine catches this before review.
