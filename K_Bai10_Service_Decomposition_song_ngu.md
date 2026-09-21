# Bài 10 — Service Decomposition & bounded context
### Bản song ngữ: tiếng Anh bám cấu trúc tiếng Việt

> **Cách dùng bài này:** đọc đoạn tiếng Việt trước → tự dịch trong đầu, không nhìn xuống → so sánh với đoạn tiếng Anh ngay bên dưới → **đọc to bản tiếng Anh hai lần**: lần một đọc chậm cho đúng từng âm, lần hai đọc liền mạch như đang nói với người phỏng vấn.

---

## ① Microservice thật sự là gì

**Tiếng Việt**

Bài này là gốc rễ sinh ra mọi bài phía trên, vì chính việc tách service tạo ra nhu cầu dùng Saga, Outbox và event. Trước hết chúng ta phải định nghĩa lại microservice cho đúng, vì rất nhiều người định nghĩa sai. Một microservice không phải là một service nhỏ, và nó cũng không phải là một thứ được đóng gói bằng Docker. Một microservice là một service tự trị được dựng quanh một năng lực nghiệp vụ, nó sở hữu dữ liệu của riêng nó, và nó được triển khai độc lập với các service khác. Nó giao tiếp với bên ngoài qua API hoặc qua event, và nó thậm chí có thể dùng một bộ công nghệ khác hẳn.

Vì vậy trọng tâm của định nghĩa nằm ở ranh giới và ở quyền tự trị, chứ không nằm ở kích thước hay công cụ. Khi bị hỏi trong phỏng vấn, chúng ta nên đưa ra đúng một phép thử để kiểm tra. Phép thử đó là chúng ta có triển khai được service này lên môi trường thật mà không phải triển khai kèm service nào khác hay không. Nếu câu trả lời là không, thì dù chúng ta có ba mươi container thì chúng ta vẫn chưa có microservice. Đây là cách nói ngắn gọn nhất và nó cho thấy chúng ta hiểu bản chất chứ không học thuộc định nghĩa.

**English (bám cấu trúc tiếng Việt)**

This lesson is the root that gives rise to all the lessons above, because it is the splitting of services that creates the need to use Sagas, Outboxes and events. First of all we have to define a microservice correctly again, because very many people define it wrongly. A microservice is not a small service, and it is also not something that is packaged with Docker. A microservice is an autonomous service built around one business capability, it owns its own data, and it is deployed independently of the other services. It communicates with the outside through an API or through events, and it can even use a completely different technology stack.

Therefore the focus of the definition lies in the boundary and in the autonomy, rather than lying in the size or the tooling. When we are asked in an interview, we should offer exactly one test to check it. That test is whether we can deploy this service to the real environment without having to deploy any other service alongside it. If the answer is no, then even if we have thirty containers we still do not have microservices. This is the shortest way of saying it and it shows that we understand the essence rather than having memorised a definition.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| gốc rễ sinh ra mọi bài phía trên | the root that gives rise to all the lessons above |
| chính việc tách service tạo ra nhu cầu | it is the splitting of services that creates the need |
| chúng ta phải định nghĩa lại cho đúng | we have to define it correctly again |
| một service tự trị được dựng quanh | an autonomous service built around |
| một năng lực nghiệp vụ | one business capability |
| được triển khai độc lập với | is deployed independently of |
| một bộ công nghệ khác hẳn | a completely different technology stack |
| trọng tâm của định nghĩa nằm ở | the focus of the definition lies in |
| chúng ta nên đưa ra đúng một phép thử | we should offer exactly one test |
| mà không phải triển khai kèm service nào khác | without having to deploy any other service alongside it |
| chúng ta hiểu bản chất | we understand the essence |

**Thuật ngữ cần nhớ**

- quyền tự trị → **autonomy**
- năng lực nghiệp vụ → **a business capability**
- triển khai độc lập → **independent deployment**
- sở hữu dữ liệu của riêng nó → **to own its own data**
- bộ công nghệ → **a technology stack**

---

## ② Tách theo nghiệp vụ, không tách theo tầng kỹ thuật

**Tiếng Việt**

Câu hỏi tiếp theo là chúng ta lấy gì làm tiêu chí để vẽ ranh giới giữa các service. Câu trả lời đúng là chúng ta tách theo năng lực nghiệp vụ, kèm theo cân nhắc về tốc độ thay đổi và về đội nào sở hữu phần nào. Mục tiêu cuối cùng rất cụ thể, đó là mỗi service phải thay đổi được một cách độc lập với các service còn lại. Nếu một yêu cầu nghiệp vụ mới chỉ chạm vào một service, chúng ta đã vẽ ranh giới đúng. Nếu nó chạm vào bốn service cùng lúc, ranh giới của chúng ta đang sai ở đâu đó.

Cách tách sai phổ biến nhất là tách theo tầng kỹ thuật, ví dụ một service cho giao diện, một service cho logic và một service cho dữ liệu. Cách này nhìn qua thì gọn gàng, vì nó phản chiếu đúng sơ đồ kiến trúc mà chúng ta hay vẽ trên bảng. Nhưng mỗi tính năng nghiệp vụ đều đi xuyên qua cả ba tầng, nên mỗi lần làm tính năng là chúng ta phải sửa cả ba service. Hậu quả là ba service đó phải triển khai cùng nhau và không bao giờ rời nhau ra được. Khi đó chúng ta gánh toàn bộ chi phí của hệ phân tán mà không nhận được một chút lợi ích tự trị nào.

**English (bám cấu trúc tiếng Việt)**

The next question is what we take as the criterion for drawing the boundaries between services. The right answer is that we split by business capability, together with considerations about the rate of change and about which team owns which part. The final goal is very concrete, which is that each service must be able to change independently of the remaining services. If a new business requirement touches only one service, we have drawn the boundary correctly. If it touches four services at the same time, our boundaries are wrong somewhere.

The most common wrong way of splitting is to split by technical layer, for example one service for the interface, one service for the logic and one service for the data. This way looks tidy at first glance, because it mirrors exactly the architecture diagram that we often draw on the whiteboard. But every business feature goes through all three layers, so every time we build a feature we have to modify all three services. The consequence is that those three services have to be deployed together and can never be pulled apart. At that point we carry the entire cost of a distributed system without receiving any benefit of autonomy at all.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| lấy gì làm tiêu chí để vẽ ranh giới | what we take as the criterion for drawing the boundaries |
| cân nhắc về tốc độ thay đổi | considerations about the rate of change |
| đội nào sở hữu phần nào | which team owns which part |
| chỉ chạm vào một service | touches only one service |
| ranh giới của chúng ta đang sai ở đâu đó | our boundaries are wrong somewhere |
| tách theo tầng kỹ thuật | to split by technical layer |
| nhìn qua thì gọn gàng | looks tidy at first glance |
| phản chiếu đúng sơ đồ kiến trúc | mirrors exactly the architecture diagram |
| không bao giờ rời nhau ra được | can never be pulled apart |
| gánh toàn bộ chi phí của hệ phân tán | carry the entire cost of a distributed system |
| không nhận được một chút lợi ích tự trị nào | without receiving any benefit of autonomy at all |

**Thuật ngữ cần nhớ**

- tầng kỹ thuật → **a technical layer**
- tốc độ thay đổi → **the rate of change**
- đội sở hữu → **the owning team**
- vẽ ranh giới → **to draw a boundary**
- sơ đồ kiến trúc → **an architecture diagram**

---

## ③ Bounded context và ngôn ngữ của từng miền

**Tiếng Việt**

Khái niệm giúp chúng ta vẽ ranh giới cho chuẩn là bounded context, và nó đến từ thiết kế hướng miền nghiệp vụ. Một bounded context là phạm vi mà trong đó một mô hình dữ liệu và một bộ từ vựng giữ nguyên ý nghĩa. Ví dụ kinh điển là từ khách hàng, vì trong bộ phận bán hàng nó nghĩa là một cơ hội có tên và số điện thoại. Nhưng trong bộ phận thanh toán, cùng từ đó lại nghĩa là một pháp nhân có địa chỉ xuất hoá đơn và hạn mức tín dụng. Hai mô hình này không nên bị nhét chung vào một bảng, vì chúng thay đổi theo hai nhịp hoàn toàn khác nhau.

Vì vậy chúng ta cố gắng đặt ranh giới service trùng với ranh giới bounded context. Khi hai ranh giới trùng nhau, mỗi đội sở hữu trọn vẹn mô hình của mình và họ đổi mô hình mà không phải xin phép ai. Khi hai ranh giới lệch nhau, chúng ta bắt đầu thấy những cuộc họp dài về việc thêm một trường vào bảng khách hàng. Trong trường hợp đó, coupling không nằm ở mã nguồn mà nằm ở ý nghĩa của dữ liệu, và loại coupling này khó gỡ hơn nhiều. Nếu hai bên buộc phải trao đổi, chúng ta định nghĩa rõ một hợp đồng dịch giữa hai mô hình thay vì để chúng rò rỉ vào nhau.

**English (bám cấu trúc tiếng Việt)**

The concept that helps us draw the boundaries properly is the bounded context, and it comes from domain-driven design. A bounded context is the scope within which one data model and one set of vocabulary keep the same meaning. The classic example is the word customer, because in the sales department it means an opportunity with a name and a phone number. But in the billing department, the same word means a legal entity with an invoicing address and a credit limit. These two models should not be stuffed together into one table, because they change at two completely different rhythms.

Therefore we try to place the service boundary so that it coincides with the bounded context boundary. When the two boundaries coincide, each team fully owns its own model and they change that model without having to ask anyone's permission. When the two boundaries are misaligned, we start seeing long meetings about adding one field to the customer table. In that case, the coupling does not sit in the source code but sits in the meaning of the data, and this kind of coupling is far harder to remove. If the two sides must exchange data, we define a clear translation contract between the two models instead of letting them leak into each other.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| giúp chúng ta vẽ ranh giới cho chuẩn | helps us draw the boundaries properly |
| thiết kế hướng miền nghiệp vụ | domain-driven design |
| phạm vi mà trong đó | the scope within which |
| giữ nguyên ý nghĩa | keep the same meaning |
| một pháp nhân | a legal entity |
| hạn mức tín dụng | a credit limit |
| không nên bị nhét chung vào một bảng | should not be stuffed together into one table |
| thay đổi theo hai nhịp hoàn toàn khác nhau | change at two completely different rhythms |
| trùng với ranh giới bounded context | coincides with the bounded context boundary |
| mà không phải xin phép ai | without having to ask anyone's permission |
| khi hai ranh giới lệch nhau | when the two boundaries are misaligned |
| để chúng rò rỉ vào nhau | letting them leak into each other |

**Thuật ngữ cần nhớ**

- phạm vi mô hình nhất quán → **a bounded context**
- thiết kế hướng miền → **domain-driven design (DDD)**
- ngôn ngữ chung của miền → **the ubiquitous language**
- pháp nhân → **a legal entity**
- hợp đồng dịch giữa hai mô hình → **a translation contract**

---

## ④ Mỗi service một cơ sở dữ liệu: lợi ích và cái giá

**Tiếng Việt**

Nguyên tắc mỗi service sở hữu một cơ sở dữ liệu riêng là điều kiện bắt buộc để có quyền tự trị thật sự. Nếu nhiều service dùng chung một cơ sở dữ liệu, schema của cơ sở dữ liệu đó trở thành một hợp đồng ngầm giữa tất cả các bên. Khi một đội đổi tên một cột, họ có thể làm hỏng ba service mà họ chưa từng đọc mã nguồn. Ngoài ra, việc dùng chung khiến chúng ta không triển khai độc lập được nữa, vì mọi thay đổi schema đều phải được phối hợp giữa nhiều đội. Vì hai lý do đó, dùng chung cơ sở dữ liệu là một anti-pattern dù nó rất tiện lúc ban đầu.

Tuy nhiên chúng ta phải trung thực về cái giá phải trả khi tách cơ sở dữ liệu. Cái giá thứ nhất là chúng ta mất khả năng nối bảng giữa hai miền dữ liệu, nên một báo cáo đơn giản bỗng cần gọi qua nhiều service. Cái giá thứ hai là chúng ta mất transaction chung, nên chúng ta phải dùng Saga và chấp nhận nhất quán cuối cùng như đã học ở Bài 7. Cái giá thứ ba là dữ liệu bị lặp lại ở vài nơi, nên chúng ta phải giữ cho các bản sao đó đồng bộ với nhau. Một người senior phải nói ra cả hai vế, vì nói mỗi phần lợi ích thì nghe giống một khẩu hiệu chứ không giống một quyết định kỹ thuật.

**English (bám cấu trúc tiếng Việt)**

The principle that each service owns its own database is a mandatory condition for having real autonomy. If many services share one database, the schema of that database becomes an implicit contract between all the sides. When one team renames a column, they may break three services whose source code they have never read. Besides, sharing makes it impossible for us to deploy independently any more, because every schema change has to be coordinated between several teams. For those two reasons, sharing a database is an anti-pattern even though it is very convenient at the beginning.

However we have to be honest about the price we pay when we split the databases. The first price is that we lose the ability to join tables across two data domains, so a simple report suddenly needs to call across several services. The second price is that we lose the shared transaction, so we have to use Sagas and accept eventual consistency as we learned in Lesson 7. The third price is that data is duplicated in a few places, so we have to keep those copies synchronised with each other. A senior person must state both sides, because stating only the benefits sounds like a slogan rather than an engineering decision.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| là điều kiện bắt buộc để có | is a mandatory condition for having |
| một hợp đồng ngầm giữa tất cả các bên | an implicit contract between all the sides |
| họ chưa từng đọc mã nguồn | whose source code they have never read |
| phải được phối hợp giữa nhiều đội | has to be coordinated between several teams |
| dù nó rất tiện lúc ban đầu | even though it is very convenient at the beginning |
| chúng ta phải trung thực về cái giá phải trả | we have to be honest about the price we pay |
| mất khả năng nối bảng | lose the ability to join tables |
| một báo cáo đơn giản bỗng cần | a simple report suddenly needs |
| giữ cho các bản sao đó đồng bộ với nhau | keep those copies synchronised with each other |
| phải nói ra cả hai vế | must state both sides |
| nghe giống một khẩu hiệu | sounds like a slogan |

**Thuật ngữ cần nhớ**

- mỗi service một cơ sở dữ liệu → **database per service**
- hợp đồng ngầm → **an implicit contract**
- nối bảng xuyên miền → **a cross-domain join**
- dữ liệu bị lặp → **duplicated data**
- mẫu sai nên tránh → **an anti-pattern**

---

## ⑤ Monolith phân tán: tệ hơn cả monolith

**Tiếng Việt**

Anti-pattern nguy hiểm nhất khi tách service là monolith phân tán, và nó rất hay xảy ra ở lần chuyển đổi đầu tiên. Đó là tình trạng chúng ta có nhiều service trên sơ đồ nhưng chúng vẫn dính chặt vào nhau trong thực tế. Dấu hiệu rõ nhất chỉ gồm một câu, đó là chúng ta không thể triển khai một service mà không triển khai kèm service khác. Các dấu hiệu đi kèm gồm chuỗi gọi đồng bộ dài, dùng chung một thư viện nội bộ cho mọi thứ, và dùng chung một cơ sở dữ liệu. Trong tình trạng đó, mỗi thay đổi nhỏ đều lan ra nhiều kho mã và nhiều lịch phát hành.

Lý do chúng ta gọi nó là tệ hơn cả monolith rất dễ giải thích. Với một monolith thật, chúng ta ít nhất còn có transaction chung, gọi hàm trong bộ nhớ, và một lần triển khai duy nhất. Với một monolith phân tán, chúng ta mất hết những thứ đó nhưng lại nhận thêm độ trễ mạng, lỗi từng phần và độ phức tạp vận hành. Nói cách khác, chúng ta trả toàn bộ hoá đơn của kiến trúc phân tán mà không mang về món hàng nào. Nếu một hệ thống rơi vào tình trạng này, cách chữa thường là gộp bớt các service lại theo bounded context chứ không phải tách thêm.

**English (bám cấu trúc tiếng Việt)**

The most dangerous anti-pattern when splitting services is the distributed monolith, and it very often happens at the first migration attempt. It is the condition in which we have many services on the diagram but they are still tightly stuck to each other in reality. The clearest sign consists of only one sentence, which is that we cannot deploy one service without deploying another service alongside it. The accompanying signs include long synchronous call chains, sharing one internal library for everything, and sharing one database. In that condition, every small change spreads out across many repositories and many release schedules.

The reason we call it worse than a monolith is very easy to explain. With a real monolith, we at least still have a shared transaction, function calls in memory, and one single deployment. With a distributed monolith, we lose all of those things but additionally receive network latency, partial failures and operational complexity. In other words, we pay the entire bill of a distributed architecture without bringing home any of the goods. If a system falls into this condition, the cure is usually to merge some services back together along bounded contexts rather than to split further.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| rất hay xảy ra ở lần chuyển đổi đầu tiên | very often happens at the first migration attempt |
| chúng vẫn dính chặt vào nhau trong thực tế | they are still tightly stuck to each other in reality |
| dấu hiệu rõ nhất chỉ gồm một câu | the clearest sign consists of only one sentence |
| các dấu hiệu đi kèm gồm | the accompanying signs include |
| chuỗi gọi đồng bộ dài | long synchronous call chains |
| lan ra nhiều kho mã và nhiều lịch phát hành | spreads out across many repositories and many release schedules |
| chúng ta ít nhất còn có | we at least still have |
| lỗi từng phần | partial failures |
| trả toàn bộ hoá đơn của kiến trúc phân tán | pay the entire bill of a distributed architecture |
| mà không mang về món hàng nào | without bringing home any of the goods |
| gộp bớt các service lại | merge some services back together |

**Thuật ngữ cần nhớ**

- monolith phân tán → **a distributed monolith**
- dính chặt vào nhau → **tightly coupled**
- lịch phát hành → **a release schedule**
- lỗi từng phần → **a partial failure**
- gộp lại → **to merge back together**

---

## ⑥ Service quá mịn và độ mịn là một đánh đổi

**Tiếng Việt**

Ở đầu bên kia của phổ, chúng ta gặp anti-pattern service quá mịn, tức là chúng ta chia nhỏ quá mức cần thiết. Dấu hiệu là một thao tác nghiệp vụ đơn giản phải gọi qua rất nhiều service mới hoàn thành. Khi đó độ trễ của từng chặng cộng dồn lại, và một hành động lẽ ra mất năm mươi mili giây thì mất tới nửa giây. Ngoài ra mỗi lời gọi thêm vào là một cơ hội hỏng thêm, nên xác suất thành công của cả chuỗi giảm rất nhanh. Chi phí vận hành cũng tăng theo, vì mỗi service đều cần theo dõi, cảnh báo, phát hành và trực sự cố.

Cách nhận biết dễ nhất là nhìn vào lịch sử thay đổi và vào cách các service gọi nhau. Nếu hai service luôn được sửa cùng nhau trong cùng một pull request, chúng gần như chắc chắn thuộc về một service duy nhất. Nếu hai service luôn gọi nhau đồng bộ và không bao giờ hoạt động riêng lẻ, chúng cũng nên được gộp lại. Vì vậy chúng ta phải nói rõ rằng độ mịn là một đánh đổi chứ không phải một cuộc đua, và nhỏ hơn không tự động tốt hơn. Câu hỏi đúng không phải là chúng ta có bao nhiêu service, mà là mỗi service có thay đổi được một mình hay không.

**English (bám cấu trúc tiếng Việt)**

At the other end of the spectrum, we meet the anti-pattern of services that are too fine-grained, that is, we split things up more than necessary. The sign is that a simple business operation has to call through very many services before it completes. At that point the latency of each hop adds up, and an action that should take fifty milliseconds ends up taking half a second. Besides, each extra call is one more opportunity to fail, so the success probability of the whole chain drops very quickly. The operational cost also rises accordingly, because every service needs monitoring, alerting, releasing and on-call duty.

The easiest way to recognise it is to look at the change history and at the way the services call each other. If two services are always modified together in the same pull request, they almost certainly belong to one single service. If two services always call each other synchronously and never operate separately, they should also be merged back together. Therefore we have to state clearly that granularity is a trade-off rather than a race, and smaller is not automatically better. The right question is not how many services we have, but whether each service can change on its own.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| ở đầu bên kia của phổ | at the other end of the spectrum |
| chia nhỏ quá mức cần thiết | split things up more than necessary |
| độ trễ của từng chặng cộng dồn lại | the latency of each hop adds up |
| lẽ ra mất năm mươi mili giây | should take fifty milliseconds |
| một cơ hội hỏng thêm | one more opportunity to fail |
| xác suất thành công của cả chuỗi | the success probability of the whole chain |
| trực sự cố | on-call duty |
| nhìn vào lịch sử thay đổi | look at the change history |
| luôn được sửa cùng nhau | are always modified together |
| không bao giờ hoạt động riêng lẻ | never operate separately |
| một đánh đổi chứ không phải một cuộc đua | a trade-off rather than a race |
| có thay đổi được một mình hay không | whether it can change on its own |

**Thuật ngữ cần nhớ**

- service quá mịn → **a nano-service**
- độ mịn khi chia → **granularity**
- một chặng gọi mạng → **a network hop**
- gọi nhau quá nhiều → **chatty communication**
- chi phí vận hành → **operational cost**

---

## ⑦ Ba cách để service A lấy dữ liệu của service B

**Tiếng Việt**

Sau khi tách cơ sở dữ liệu, câu hỏi thực tế nhất là service A lấy dữ liệu của service B bằng cách nào. Cách thứ nhất là gọi đồng bộ sang B ngay lúc cần, và cách này cho chúng ta dữ liệu tươi nhất. Đổi lại, A chỉ sống được khi B còn sống, độ trễ cộng thêm vào mỗi request, và chúng ta mở đường cho lỗi lan theo chuỗi. Cách thứ hai là nhân bản dữ liệu qua event, nghĩa là A giữ một bản sao cục bộ được cập nhật từ các event mà B phát ra. Cách này gỡ phụ thuộc rất tốt và đọc rất nhanh, nhưng dữ liệu chỉ nhất quán cuối cùng và chúng ta phải chấp nhận dữ liệu bị lặp.

Cách thứ ba là dựng một mô hình đọc riêng phục vụ cho các câu truy vấn xuyên nhiều service. Cách này cho truy vấn rất nhanh, nhưng nó thêm hạ tầng và nó cũng chỉ nhất quán cuối cùng. Điều quan trọng nhất là không có cách nào là mặc định đúng cho mọi tình huống. Chúng ta chọn dựa trên ba câu hỏi, gồm dữ liệu cần tươi tới mức nào, chúng ta đọc nó thường xuyên tới mức nào, và chúng ta chịu được bao nhiêu khi B chết. Ví dụ với một đơn hàng, chúng ta thường lưu tên món và giá tại thời điểm đặt, vì đó là dữ liệu lịch sử chứ không phải dữ liệu hiện tại của danh mục.

**English (bám cấu trúc tiếng Việt)**

After we split the databases, the most practical question is how service A gets the data of service B. The first way is to call synchronously over to B at the moment it is needed, and this way gives us the freshest data. In exchange, A can only live while B is alive, the latency adds onto every request, and we open the road for cascade failure. The second way is to replicate the data through events, which means that A keeps a local copy that is updated from the events that B emits. This way decouples very well and reads very fast, but the data is only eventually consistent and we have to accept duplicated data.

The third way is to build a separate read model serving the queries that span several services. This way makes queries very fast, but it adds infrastructure and it is also only eventually consistent. The most important thing is that no way is the correct default for every situation. We choose based on three questions, namely how fresh the data needs to be, how often we read it, and how much we can tolerate when B is down. For example with an order, we usually store the item name and the price at the moment of ordering, because that is historical data rather than the current data of the catalogue.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| câu hỏi thực tế nhất | the most practical question |
| ngay lúc cần | at the moment it is needed |
| dữ liệu tươi nhất | the freshest data |
| chỉ sống được khi B còn sống | can only live while B is alive |
| chúng ta mở đường cho lỗi lan theo chuỗi | we open the road for cascade failure |
| giữ một bản sao cục bộ | keeps a local copy |
| gỡ phụ thuộc rất tốt | decouples very well |
| các câu truy vấn xuyên nhiều service | the queries that span several services |
| không có cách nào là mặc định đúng | no way is the correct default |
| chúng ta chịu được bao nhiêu khi B chết | how much we can tolerate when B is down |
| tại thời điểm đặt | at the moment of ordering |
| dữ liệu lịch sử chứ không phải dữ liệu hiện tại | historical data rather than current data |

**Thuật ngữ cần nhớ**

- ghép dữ liệu bằng cách gọi API → **API composition**
- nhân bản dữ liệu qua event → **data replication through events**
- bản sao cục bộ → **a local copy**
- độ tươi của dữ liệu → **data freshness**
- dữ liệu tại thời điểm giao dịch → **point-in-time data**

---

## ⑧ Thoát khỏi monolith bằng cách siết dần

**Tiếng Việt**

Câu hỏi cuối cùng là chúng ta đi từ một monolith sẵn có tới kiến trúc mới bằng con đường nào. Cách được khuyến nghị gọi là siết dần, và tên gọi này mượn hình ảnh cây đa siết chết cây chủ. Chúng ta đặt một lớp định tuyến ở phía trước, thường là một cổng vào hoặc một proxy, rồi chúng ta tách từng năng lực nghiệp vụ ra thành service mới. Với mỗi năng lực đã tách xong, chúng ta chuyển dần lưu lượng từ monolith sang service mới và theo dõi kỹ. Monolith vẫn chạy song song trong suốt quá trình, và nó chỉ được gỡ bỏ khi phần cuối cùng đã được rút ra.

Chúng ta nên giải thích được vì sao cách viết lại toàn bộ một lần lại nguy hiểm đến vậy. Lý do thứ nhất là một dự án viết lại thường kéo dài nhiều tháng trong khi nghiệp vụ vẫn tiếp tục thay đổi mỗi tuần. Lý do thứ hai là chúng ta phải viết lại cả những hành vi kỳ lạ mà không ai còn nhớ vì sao chúng tồn tại. Lý do thứ ba là chúng ta không có đường lùi, vì tới ngày chuyển đổi thì hoặc là tất cả cùng chạy, hoặc là tất cả cùng hỏng. Khi một đồng nghiệp đề xuất viết lại từ đầu cho sạch, chúng ta nên hỏi lại chúng ta sẽ giao được giá trị đầu tiên cho người dùng vào tháng nào, và câu trả lời thường làm cuộc thảo luận đổi hướng.

**English (bám cấu trúc tiếng Việt)**

The final question is which road we take to go from an existing monolith to the new architecture. The recommended approach is called strangling, and this name borrows the image of a fig tree strangling its host tree. We place a routing layer in front, usually a gateway or a proxy, and then we pull each business capability out into a new service. For each capability that has been pulled out, we gradually shift the traffic from the monolith over to the new service and watch it closely. The monolith keeps running in parallel throughout the process, and it is only removed when the last piece has been pulled out.

We should be able to explain why the approach of rewriting everything at once is so dangerous. The first reason is that a rewrite project usually stretches over many months while the business keeps changing every week. The second reason is that we have to rewrite even the strange behaviours that nobody remembers the reason for. The third reason is that we have no way back, because on the switchover day either everything runs together, or everything breaks together. When a colleague proposes a clean rewrite from scratch, we should ask back in which month we will deliver the first value to users, and the answer usually turns the discussion in another direction.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| bằng con đường nào | which road we take |
| cách được khuyến nghị gọi là | the recommended approach is called |
| mượn hình ảnh cây đa siết chết cây chủ | borrows the image of a fig tree strangling its host tree |
| chúng ta đặt một lớp định tuyến ở phía trước | we place a routing layer in front |
| chuyển dần lưu lượng | gradually shift the traffic |
| vẫn chạy song song trong suốt quá trình | keeps running in parallel throughout the process |
| kéo dài nhiều tháng | stretches over many months |
| những hành vi kỳ lạ mà không ai còn nhớ vì sao | the strange behaviours that nobody remembers the reason for |
| chúng ta không có đường lùi | we have no way back |
| tới ngày chuyển đổi | on the switchover day |
| viết lại từ đầu cho sạch | a clean rewrite from scratch |
| làm cuộc thảo luận đổi hướng | turns the discussion in another direction |

**Thuật ngữ cần nhớ**

- siết dần từng phần → **the Strangler Fig pattern**
- viết lại toàn bộ một lần → **a big-bang rewrite**
- chuyển dần lưu lượng → **to shift traffic gradually**
- ngày chuyển đổi → **the switchover day**
- đường lùi → **a way back / a rollback path**

---

## ⑨ Mô hình ghi nhớ

**Tiếng Việt**

Một service là một bounded context tự trị, có cơ sở dữ liệu riêng và triển khai được một mình; chúng ta tách theo nghiệp vụ chứ không tách theo tầng kỹ thuật. Tách quá chặt thì thành monolith phân tán còn tệ hơn monolith, tách quá mịn thì thành các service gọi nhau suốt ngày, và chúng ta thoát khỏi monolith bằng cách siết dần chứ không viết lại một lần.

**English (bám cấu trúc tiếng Việt)**

A service is an autonomous bounded context, with its own database and deployable on its own; we split by business rather than splitting by technical layer. Splitting too tightly becomes a distributed monolith that is worse than a monolith, splitting too finely becomes services that call each other all day long, and we escape the monolith by strangling it gradually rather than rewriting it at once.

---

## ⑩ Bảng thuật ngữ tổng hợp

| Tiếng Việt | Tiếng Anh | Ghi chú phát âm |
|---|---|---|
| quyền tự trị | autonomy | or-**TO**-nə-mi, trọng âm âm thứ hai |
| tự trị (tính từ) | autonomous | or-**TO**-nə-məs |
| năng lực nghiệp vụ | a business capability | *business* — hai âm tiết "BIZ-nis" |
| triển khai độc lập | independent deployment | *deployment* = di-**PLOI**-mənt |
| bộ công nghệ | a technology stack | |
| tầng kỹ thuật | a technical layer | |
| tốc độ thay đổi | the rate of change | |
| vẽ ranh giới | to draw a boundary | *boundary* = **BOUN**-də-ri |
| sơ đồ kiến trúc | an architecture diagram | **AR**-ki-tek-chə; *diagram* = **DAI**-ə-gram |
| phạm vi mô hình nhất quán | a bounded context | |
| thiết kế hướng miền | domain-driven design (DDD) | *domain* = də-**MAIN**, trọng âm âm sau |
| ngôn ngữ chung của miền | the ubiquitous language | yu-**BI**-kwi-təs, trọng âm âm thứ hai |
| pháp nhân | a legal entity | *entity* = **EN**-ti-ty |
| hợp đồng dịch giữa hai mô hình | a translation contract | *contract* (danh từ) = **CON**-tract |
| mỗi service một cơ sở dữ liệu | database per service | |
| hợp đồng ngầm | an implicit contract | im-**PLI**-sit |
| nối bảng xuyên miền | a cross-domain join | |
| dữ liệu bị lặp | duplicated data | *duplicated* = **DU**-pli-kay-tid |
| mẫu sai nên tránh | an anti-pattern | giọng Anh *anti* /ˈænti/ — "AN-ti", không phải "AN-tai" |
| monolith phân tán | a distributed monolith | *monolith* = **MO**-nə-lith, âm cuối là /θ/ |
| dính chặt vào nhau | tightly coupled | *coupled* = **CU**-pəld, âm giữa là /ʌ/ |
| lịch phát hành | a release schedule | giọng Anh *schedule* /ˈʃedjuːl/ — "SHED-yul" |
| lỗi từng phần | a partial failure | *partial* = **PAR**-shl |
| gộp lại | to merge back together | |
| service quá mịn | a nano-service | |
| độ mịn khi chia | granularity | gra-nyu-**LA**-ri-ty, trọng âm âm thứ ba |
| một chặng gọi mạng | a network hop | |
| gọi nhau quá nhiều | chatty communication | **CHA**-ti |
| chi phí vận hành | operational cost | |
| trực sự cố | on-call duty | |
| ghép dữ liệu bằng cách gọi API | API composition | com-pə-**ZI**-shn |
| nhân bản dữ liệu qua event | data replication through events | rep-li-**KAY**-shn |
| bản sao cục bộ | a local copy | |
| độ tươi của dữ liệu | data freshness | giọng Anh *data* /ˈdeɪtə/ — "ĐÂY-tơ" |
| dữ liệu tại thời điểm giao dịch | point-in-time data | |
| siết dần từng phần | the Strangler Fig pattern | *strangler* = **STRANG**-glə, cụm *str-* phải bật rõ |
| viết lại toàn bộ một lần | a big-bang rewrite | |
| chuyển dần lưu lượng | to shift traffic gradually | *gradually* = **GRA**-ju-ə-li |
| ngày chuyển đổi | the switchover day | |
| đường lùi | a rollback path | |
| yêu cầu chạm vào nhiều service | a change that spans services | |

---

## ⑪ Bài nói

> **Cách luyện:** bật ghi âm, nói **60–90 giây** cho mỗi đề, **không nhìn tài liệu**. Mỗi bài nói phải dùng ít nhất **sáu thuật ngữ** trong bảng trên. Nói xong thì nghe lại một lượt, đánh dấu chỗ mình ngập ngừng, rồi nói lại chính đề đó thêm một lần nữa.

1. Explain to a junior developer what actually makes something a microservice, and give the one test you would apply to check it.

2. A colleague proposes splitting the system into a UI service, a business logic service and a data service. Explain why you would push back on those boundaries.

3. Describe what a bounded context is, using the word customer in sales and in billing, and explain how that shapes where you draw service boundaries.

4. Someone on your team wants two services to share one database so that reporting can use joins. Explain the case against it and what you would offer instead.

5. Your system has thirty services, but every release has to be deployed together and checkout calls twelve services synchronously. Describe your diagnosis and your first three moves.

6. Service A needs product names and prices for an order. Walk through the three ways it could get that data, and say which one you would choose and why.

7. When would you recommend the Strangler Fig approach over a rewrite, and how would you explain that choice to a stakeholder who wants a clean start?
