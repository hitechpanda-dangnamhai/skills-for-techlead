# Bài 6 — N+1 query
### Bản song ngữ: tiếng Anh bám cấu trúc tiếng Việt

> **Cách dùng bài này:** đọc đoạn tiếng Việt trước, tự dịch trong đầu, rồi mới so với đoạn tiếng Anh bên dưới. Chỗ nào lệch thì xem bảng ánh xạ cụm khó. Cuối cùng, **đọc to đoạn tiếng Anh hai lần** — một lần chậm để đúng âm, một lần với tốc độ nói thật.

---

## Phần 1 — N+1 là gì và nó sinh ra từ đâu

**Tiếng Việt**

N+1 là lỗi hiệu năng phổ biến nhất ở tầng ứng dụng, và gần như mọi hệ thống dùng ORM đều từng mắc phải nó. Tên gọi này mô tả đúng những gì xảy ra, vì chúng ta chạy một câu truy vấn để lấy một danh sách gồm N phần tử, rồi chúng ta chạy thêm N câu truy vấn nữa để lấy dữ liệu liên quan cho từng phần tử. Tổng cộng là N cộng một lần đi tới cơ sở dữ liệu chỉ cho một request duy nhất. Nguyên nhân gần như luôn giống nhau, đó là chúng ta gọi một quan hệ được nạp lười ngay bên trong một vòng lặp. Ví dụ, chúng ta lấy về năm mươi đơn hàng, rồi trong vòng lặp chúng ta đọc tên khách hàng của từng đơn. Mỗi lần chúng ta chạm vào thuộc tính khách hàng, ORM âm thầm bắn thêm một câu truy vấn, và người viết code không hề nhìn thấy điều đó trong mã nguồn. Chính vì nó vô hình ở tầng mã nguồn, N+1 thường chỉ lộ ra khi lượng dữ liệu thật lớn lên trong production.

**English (bám cấu trúc tiếng Việt)**

N+1 is the most common performance problem at the application layer, and almost every system that uses an ORM has run into it at some point. The name describes exactly what happens, because we run one query to fetch a list of N items, then we run N more queries to fetch the related data for each item. In total that is N plus one trips to the database for just one single request. The cause is almost always the same, namely that we touch a lazily loaded relationship right inside a loop. For example, we fetch fifty orders, then inside the loop we read the customer name of each order. Every time we touch the customer property, the ORM quietly fires one more query, and the person writing the code does not see that at all in the source. Precisely because it is invisible at the source level, N+1 usually only shows up when the real data grows large in production.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| gần như mọi hệ thống … đều từng mắc phải nó | almost every system … has run into it at some point |
| tên gọi này mô tả đúng những gì xảy ra | the name describes exactly what happens |
| chúng ta chạy thêm N câu truy vấn nữa | we run N more queries |
| tổng cộng là | in total that is |
| chỉ cho một request duy nhất | for just one single request |
| một quan hệ được nạp lười | a lazily loaded relationship |
| ngay bên trong một vòng lặp | right inside a loop |
| ORM âm thầm bắn thêm một câu truy vấn | the ORM quietly fires one more query |
| không hề nhìn thấy điều đó trong mã nguồn | does not see that at all in the source |
| chính vì nó vô hình ở tầng mã nguồn | precisely because it is invisible at the source level |
| chỉ lộ ra khi lượng dữ liệu thật lớn lên | only shows up when the real data grows large |

**Thuật ngữ cần nhớ**

- nạp lười → **lazy loading**
- nạp sẵn → **eager loading**
- quan hệ giữa hai bảng → **relationship**
- vòng lặp → **loop**
- mã nguồn → **source code**

---

## Phần 2 — Vì sao N+1 đắt: chi phí nằm ở số lần đi và về

**Tiếng Việt**

Điều làm cho N+1 trở nên đắt không phải là bản thân từng câu truy vấn, vì mỗi câu trong số đó thường rất nhanh. Điều làm cho nó đắt là số lần đi và về giữa ứng dụng và cơ sở dữ liệu. Mỗi lần đi và về đều tốn thời gian mạng, tốn một lần lấy kết nối từ pool, và tốn công phân tích lại câu lệnh. Nếu một lần đi và về mất một mili giây, thì năm mươi mốt lần sẽ mất năm mươi mốt mili giây chỉ riêng cho phần chờ đợi. Con số đó nghe có vẻ nhỏ, nhưng nó nhân lên theo số người dùng đồng thời, và nó chiếm giữ kết nối trong suốt khoảng thời gian đó. Hậu quả cuối cùng là pool kết nối cạn kiệt, và những request hoàn toàn không liên quan cũng bắt đầu chậm theo.

**English (bám cấu trúc tiếng Việt)**

The thing that makes N+1 expensive is not each query itself, because each one of them is usually very fast. The thing that makes it expensive is the number of round trips between the application and the database. Every round trip costs network time, costs one checkout of a connection from the pool, and costs the work of parsing the statement again. If one round trip takes one millisecond, then fifty-one of them will take fifty-one milliseconds for the waiting alone. That number sounds small, but it multiplies with the number of concurrent users, and it holds a connection for that whole period. The final consequence is that the connection pool runs dry, and requests that are completely unrelated also start to slow down.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| điều làm cho N+1 trở nên đắt | the thing that makes N+1 expensive |
| không phải là bản thân từng câu truy vấn | is not each query itself |
| số lần đi và về | the number of round trips |
| tốn một lần lấy kết nối từ pool | costs one checkout of a connection from the pool |
| tốn công phân tích lại câu lệnh | costs the work of parsing the statement again |
| chỉ riêng cho phần chờ đợi | for the waiting alone |
| con số đó nghe có vẻ nhỏ | that number sounds small |
| nó nhân lên theo số người dùng đồng thời | it multiplies with the number of concurrent users |
| nó chiếm giữ kết nối | it holds a connection |
| pool kết nối cạn kiệt | the connection pool runs dry |
| những request hoàn toàn không liên quan | requests that are completely unrelated |

**Thuật ngữ cần nhớ**

- một lần đi và về → **round trip**
- bể kết nối → **connection pool**
- người dùng đồng thời → **concurrent users**
- phân tích câu lệnh → **parse the statement**
- cạn kiệt → **run dry / be exhausted**

---

## Phần 3 — Phát hiện bằng đo, không phát hiện bằng đoán

**Tiếng Việt**

Chúng ta không nên đoán rằng một endpoint đang bị N+1, mà chúng ta phải đo để biết chắc. Cách đơn giản nhất là chúng ta bật ghi log truy vấn ở môi trường phát triển, rồi chúng ta đếm số câu truy vấn cho một request. Nếu chúng ta thấy năm mươi mốt câu truy vấn cho một trang danh sách năm mươi dòng, chúng ta đã có câu trả lời ngay lập tức. Trong production, chúng ta dùng công cụ giám sát hiệu năng ứng dụng, vì nó vẽ ra toàn bộ các span của một request theo trục thời gian. Trên biểu đồ đó, N+1 hiện lên rất đặc trưng, đó là một hàng dài các thanh nhỏ giống hệt nhau nằm nối tiếp nhau. Một cách khác là chúng ta mở pg_stat_statements và tìm câu truy vấn có số lần gọi cao bất thường so với số request. Dấu hiệu chắc chắn nhất là cùng một dạng câu lệnh được gọi lặp đi lặp lại, và chúng chỉ khác nhau ở giá trị tham số.

**English (bám cấu trúc tiếng Việt)**

We should not guess that an endpoint is suffering from N+1, but we have to measure in order to know for sure. The simplest way is that we turn on query logging in the development environment, then we count the number of queries for one request. If we see fifty-one queries for a list page of fifty rows, we already have the answer immediately. In production, we use an application performance monitoring tool, because it draws all the spans of one request along a time axis. On that chart, N+1 shows up in a very characteristic way, namely as a long row of identical small bars sitting one after another. Another way is that we open pg_stat_statements and look for the query whose call count is unusually high compared with the number of requests. The most certain sign is the same statement shape being called over and over again, and they differ only in the parameter values.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| chúng ta phải đo để biết chắc | we have to measure in order to know for sure |
| bật ghi log truy vấn | turn on query logging |
| chúng ta đã có câu trả lời ngay lập tức | we already have the answer immediately |
| vẽ ra toàn bộ các span của một request theo trục thời gian | draws all the spans of one request along a time axis |
| hiện lên rất đặc trưng | shows up in a very characteristic way |
| một hàng dài các thanh nhỏ giống hệt nhau | a long row of identical small bars |
| nằm nối tiếp nhau | sitting one after another |
| số lần gọi cao bất thường | a call count that is unusually high |
| dấu hiệu chắc chắn nhất | the most certain sign |
| cùng một dạng câu lệnh | the same statement shape |
| chỉ khác nhau ở giá trị tham số | differ only in the parameter values |

**Thuật ngữ cần nhớ**

- ghi log truy vấn → **query logging**
- công cụ giám sát hiệu năng ứng dụng → **APM tool**
- đoạn theo dõi trong một request → **span**
- số lần gọi → **call count**
- giá trị tham số → **parameter value**

---

## Phần 4 — Cách sửa thứ nhất: JOIN và cái bẫy nhân dòng

**Tiếng Việt**

Cách sửa đầu tiên là chúng ta nạp sẵn quan hệ bằng một câu JOIN, và cách này thường được ORM gọi là eager loading. Ưu điểm rõ ràng là chúng ta chỉ đi tới cơ sở dữ liệu đúng một lần cho cả trang. Nhưng cách này có một cái bẫy tên là nhân dòng, và cái bẫy đó xuất hiện khi chúng ta JOIN nhiều quan hệ một-nhiều cùng lúc. Nếu một đơn hàng có năm dòng hàng và ba lần thanh toán, một câu JOIN cả hai bảng sẽ trả về mười lăm dòng cho đúng một đơn hàng. Ứng dụng sau đó phải gộp lại các dòng trùng, và lượng dữ liệu truyền qua mạng phình lên một cách vô ích. Ngoài ra, JOIN thường kéo về nhiều cột hơn mức chúng ta cần, và đó chính là hiện tượng lấy thừa dữ liệu. Vì vậy, JOIN là lựa chọn tốt cho quan hệ nhiều-một, nhưng nó trở nên nguy hiểm khi chúng ta ghép nhiều nhánh một-nhiều.

**English (bám cấu trúc tiếng Việt)**

The first fix is that we load the relationship in advance with a JOIN, and this approach is usually called eager loading by the ORM. The obvious advantage is that we go to the database exactly once for the whole page. But this approach has a trap named row multiplication, and that trap appears when we JOIN several one-to-many relationships at the same time. If one order has five line items and three payments, a JOIN across both tables will return fifteen rows for exactly one order. The application then has to collapse the duplicated rows, and the amount of data travelling over the network grows fat for no good reason. In addition, a JOIN usually drags back more columns than we need, and that is exactly the over-fetching problem. Therefore, a JOIN is a good choice for a many-to-one relationship, but it becomes dangerous when we combine several one-to-many branches.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| chúng ta nạp sẵn quan hệ | we load the relationship in advance |
| ưu điểm rõ ràng là | the obvious advantage is that |
| một cái bẫy tên là nhân dòng | a trap named row multiplication |
| quan hệ một-nhiều | a one-to-many relationship |
| năm dòng hàng và ba lần thanh toán | five line items and three payments |
| phải gộp lại các dòng trùng | has to collapse the duplicated rows |
| phình lên một cách vô ích | grows fat for no good reason |
| kéo về nhiều cột hơn mức chúng ta cần | drags back more columns than we need |
| hiện tượng lấy thừa dữ liệu | the over-fetching problem |
| khi chúng ta ghép nhiều nhánh một-nhiều | when we combine several one-to-many branches |

**Thuật ngữ cần nhớ**

- nhân dòng → **row multiplication**
- dòng trùng lặp → **duplicated rows**
- lấy thừa dữ liệu → **over-fetching**
- quan hệ nhiều-một → **many-to-one relationship**
- dòng hàng trong đơn → **line item**

---

## Phần 5 — Cách sửa thứ hai: gom theo lô bằng mệnh đề IN

**Tiếng Việt**

Cách sửa thứ hai là gom theo lô, trong đó chúng ta chạy đúng hai câu truy vấn thay vì N cộng một. Câu thứ nhất lấy danh sách, còn câu thứ hai lấy toàn bộ quan hệ bằng một mệnh đề IN chứa các id mà chúng ta vừa thu thập. Sau đó, ứng dụng ghép hai tập kết quả lại với nhau trong bộ nhớ, và chúng ta thường làm việc đó bằng một map từ id sang đối tượng. Cách này thường là điểm cân bằng tốt nhất, vì nó tránh được cả việc đi lại nhiều lần lẫn việc nhân dòng. Chúng ta chỉ cần chú ý một điểm, đó là danh sách id không được quá dài, vì một mệnh đề IN chứa hàng chục nghìn phần tử sẽ làm bộ lập kế hoạch chậm hẳn đi. Trong trường hợp đó, chúng ta chia danh sách thành từng lô nhỏ, ví dụ mỗi lô một nghìn id.

**English (bám cấu trúc tiếng Việt)**

The second fix is batching, in which we run exactly two queries instead of N plus one. The first one fetches the list, while the second one fetches all the relationships with an IN clause containing the ids that we have just collected. After that, the application stitches the two result sets together in memory, and we usually do that with a map from id to object. This approach is usually the best balance point, because it avoids both the repeated round trips and the row multiplication. We only need to watch one thing, namely that the id list must not be too long, because an IN clause containing tens of thousands of elements will slow the planner down noticeably. In that case, we split the list into small batches, for example one thousand ids per batch.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| gom theo lô | batching |
| chúng ta chạy đúng hai câu truy vấn | we run exactly two queries |
| các id mà chúng ta vừa thu thập | the ids that we have just collected |
| ghép hai tập kết quả lại với nhau trong bộ nhớ | stitches the two result sets together in memory |
| một map từ id sang đối tượng | a map from id to object |
| điểm cân bằng tốt nhất | the best balance point |
| tránh được cả … lẫn … | avoids both … and … |
| chúng ta chỉ cần chú ý một điểm | we only need to watch one thing |
| sẽ làm bộ lập kế hoạch chậm hẳn đi | will slow the planner down noticeably |
| chia danh sách thành từng lô nhỏ | split the list into small batches |

**Thuật ngữ cần nhớ**

- gom theo lô → **batching**
- mệnh đề IN → **the IN clause**
- tập kết quả → **result set**
- ghép dữ liệu trong bộ nhớ → **stitch data in memory**
- mỗi lô → **per batch**

---

## Phần 6 — Cách sửa thứ ba: DataLoader cho GraphQL

**Tiếng Việt**

GraphQL rất dễ dính N+1, vì các resolver được gọi lồng nhau theo từng trường của từng node. Khi client hỏi năm mươi bài viết kèm theo tác giả, resolver tác giả sẽ chạy năm mươi lần một cách hoàn toàn độc lập. Giải pháp chuẩn cho vấn đề này là DataLoader, một thư viện do Facebook tạo ra chính vì lý do đó. DataLoader gom tất cả các yêu cầu phát sinh trong cùng một nhịp của vòng lặp sự kiện, rồi nó gọi cơ sở dữ liệu đúng một lần cho cả lô. Nó cũng giữ một bộ nhớ đệm sống trong phạm vi một request, vì vậy hai node cùng trỏ tới một tác giả sẽ chỉ tốn một lần nạp. Điều quan trọng khi chúng ta viết hàm nạp lô là chúng ta phải trả kết quả đúng theo thứ tự của danh sách id đầu vào, nếu không dữ liệu sẽ bị gán nhầm cho sai node.

**English (bám cấu trúc tiếng Việt)**

GraphQL falls into N+1 very easily, because the resolvers are called in a nested way for each field of each node. When the client asks for fifty posts together with their authors, the author resolver will run fifty times completely independently. The standard solution for this problem is DataLoader, a library that Facebook created for exactly that reason. DataLoader collects all the requests that arise within the same tick of the event loop, then it calls the database exactly once for the whole batch. It also keeps a cache that lives within the scope of one request, therefore two nodes pointing at the same author will only cost one load. The important thing when we write the batch loading function is that we have to return the results in exactly the order of the incoming id list, otherwise the data will be attached to the wrong node.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| rất dễ dính N+1 | falls into N+1 very easily |
| được gọi lồng nhau theo từng trường của từng node | are called in a nested way for each field of each node |
| kèm theo tác giả | together with their authors |
| một cách hoàn toàn độc lập | completely independently |
| một thư viện do Facebook tạo ra chính vì lý do đó | a library that Facebook created for exactly that reason |
| các yêu cầu phát sinh trong cùng một nhịp của vòng lặp sự kiện | the requests that arise within the same tick of the event loop |
| một bộ nhớ đệm sống trong phạm vi một request | a cache that lives within the scope of one request |
| sẽ chỉ tốn một lần nạp | will only cost one load |
| hàm nạp lô | the batch loading function |
| dữ liệu sẽ bị gán nhầm cho sai node | the data will be attached to the wrong node |

**Thuật ngữ cần nhớ**

- hàm phân giải trường → **resolver**
- lồng nhau → **nested**
- nhịp của vòng lặp sự kiện → **a tick of the event loop**
- bộ nhớ đệm theo request → **per-request cache**
- giữ đúng thứ tự → **preserve the order**

---

## Phần 7 — Nạp sẵn tất cả cũng là một lỗi

**Tiếng Việt**

Sau khi bị N+1 một lần, nhiều đội quay sang bật eager loading cho mọi quan hệ để cho chắc. Đây là một phản ứng thái quá, và nó tạo ra một nhóm vấn đề khác. Vấn đề thứ nhất là chúng ta lấy thừa rất nhiều dữ liệu mà màn hình không bao giờ hiển thị. Vấn đề thứ hai là các câu JOIN trở nên nặng và chúng trả về những dòng trùng lặp như chúng ta đã nói ở trên. Vấn đề thứ ba là bộ nhớ của tiến trình ứng dụng bị ngốn, vì mỗi request nạp về một cây đối tượng lớn hơn nhiều so với nhu cầu thật. Vì vậy, nạp lười vẫn là lựa chọn đúng cho những quan hệ mà chúng ta hiếm khi cần tới, và chúng ta chỉ nạp sẵn đúng những quan hệ mà màn hình thật sự dùng.

**English (bám cấu trúc tiếng Việt)**

After being hit by N+1 once, many teams switch to turning eager loading on for every relationship just to be safe. This is an overreaction, and it creates a different group of problems. The first problem is that we over-fetch a lot of data that the screen never displays. The second problem is that the JOINs become heavy and they return duplicated rows as we said above. The third problem is that the memory of the application process gets eaten up, because each request loads an object tree that is far larger than the real need. Therefore, lazy loading is still the right choice for the relationships that we rarely need, and we only eagerly load exactly the relationships that the screen actually uses.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| quay sang bật eager loading cho mọi quan hệ | switch to turning eager loading on for every relationship |
| để cho chắc | just to be safe |
| đây là một phản ứng thái quá | this is an overreaction |
| nó tạo ra một nhóm vấn đề khác | it creates a different group of problems |
| mà màn hình không bao giờ hiển thị | that the screen never displays |
| như chúng ta đã nói ở trên | as we said above |
| bộ nhớ của tiến trình ứng dụng bị ngốn | the memory of the application process gets eaten up |
| một cây đối tượng lớn hơn nhiều so với nhu cầu thật | an object tree that is far larger than the real need |
| những quan hệ mà chúng ta hiếm khi cần tới | the relationships that we rarely need |
| đúng những quan hệ mà màn hình thật sự dùng | exactly the relationships that the screen actually uses |

**Thuật ngữ cần nhớ**

- phản ứng thái quá → **overreaction**
- cây đối tượng → **object tree**
- tiến trình ứng dụng → **application process**
- hiếm khi → **rarely**
- nhu cầu thật → **the real need**

---

## Phần 8 — Đo chứ đừng giáo điều, và đặt ngân sách truy vấn

**Tiếng Việt**

Có một câu giáo điều rất hay được nhắc lại, đó là một câu JOIN lớn luôn nhanh hơn nhiều câu truy vấn nhỏ. Câu này không đúng trong mọi trường hợp, và đây chính là chỗ để chúng ta thể hiện tư duy của một người senior. Một câu JOIN lớn nhân dòng, nó truyền nhiều dữ liệu hơn qua mạng, và kết quả của nó rất khó cache vì nó gắn với một tổ hợp tham số cụ thể. Nó cũng chạm vào nhiều bảng cùng một lúc, vì vậy nó giữ khoá trên một phạm vi rộng hơn. Ngược lại, vài câu truy vấn nhỏ có thể dùng đúng index, mỗi câu trả về ít dữ liệu, và kết quả của từng câu dễ được cache lại và dùng chung. Vì vậy, quy tắc đúng là chúng ta đo cả hai phương án trên dữ liệu thật, chứ chúng ta không chọn theo niềm tin. Ở cấp tech lead, chúng ta còn nên đặt một ngân sách số câu truy vấn cho mỗi request, và chúng ta cho hệ thống cảnh báo mỗi khi một endpoint vượt quá ngân sách đó.

**English (bám cấu trúc tiếng Việt)**

There is a piece of dogma that gets repeated a lot, namely that one large JOIN is always faster than many small queries. This is not true in every case, and this is exactly the place where we show the thinking of a senior person. A large JOIN multiplies rows, it sends more data over the network, and its result is very hard to cache because it is tied to one specific combination of parameters. It also touches many tables at the same time, therefore it holds locks over a wider scope. On the other hand, a few small queries can use exactly the right index, each one returns little data, and the result of each one is easy to cache and to share. Therefore, the correct rule is that we measure both options on real data, and we do not choose by belief. At tech lead level, we should also set a budget for the number of queries per request, and we make the system alert us every time an endpoint goes over that budget.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| một câu giáo điều rất hay được nhắc lại | a piece of dogma that gets repeated a lot |
| câu này không đúng trong mọi trường hợp | this is not true in every case |
| chỗ để chúng ta thể hiện tư duy của một người senior | the place where we show the thinking of a senior person |
| nó gắn với một tổ hợp tham số cụ thể | it is tied to one specific combination of parameters |
| nó giữ khoá trên một phạm vi rộng hơn | it holds locks over a wider scope |
| dễ được cache lại và dùng chung | easy to cache and to share |
| chúng ta đo cả hai phương án trên dữ liệu thật | we measure both options on real data |
| chúng ta không chọn theo niềm tin | we do not choose by belief |
| đặt một ngân sách số câu truy vấn cho mỗi request | set a budget for the number of queries per request |
| mỗi khi một endpoint vượt quá ngân sách đó | every time an endpoint goes over that budget |

**Thuật ngữ cần nhớ**

- giáo điều → **dogma**
- tổ hợp tham số → **combination of parameters**
- phạm vi giữ khoá → **lock scope**
- ngân sách truy vấn → **query budget**
- cảnh báo → **alert**

---

## Mô hình ghi nhớ

**Tiếng Việt**

N+1 là hiện tượng chúng ta gọi cơ sở dữ liệu bên trong một vòng lặp, vì vậy chúng ta gom các lời gọi đó lại bằng mệnh đề IN, bằng JOIN hoặc bằng DataLoader. Nhưng gom thành một câu JOIN khổng lồ cũng là một lỗi, vì vậy chúng ta phải đo chứ chúng ta không đoán.

**English (bám cấu trúc tiếng Việt)**

N+1 is the situation where we call the database inside a loop, therefore we group those calls together with an IN clause, with a JOIN or with DataLoader. But grouping them into one giant JOIN is also a mistake, therefore we have to measure and we do not guess.

---

## Bảng thuật ngữ tổng hợp

| Tiếng Việt | Tiếng Anh | Ghi chú phát âm |
|---|---|---|
| nạp lười | lazy loading | |
| nạp sẵn | eager loading | *eager* /ˈiːɡə/ — "**II**-gơ", nguyên âm dài |
| quan hệ giữa hai bảng | relationship | ri-**LAY**-shơn-ship |
| quan hệ một-nhiều / nhiều-một | one-to-many / many-to-one | |
| vòng lặp | loop | |
| mã nguồn | source code | *source* /sɔːs/ — "sos", không đọc "sua-xơ" |
| một lần đi và về | round trip | |
| bể kết nối | connection pool | |
| người dùng đồng thời | concurrent users | cơn-**CU**-rơnt — trọng âm 2 |
| phân tích câu lệnh | parse the statement | *parse* /pɑːz/ — đuôi /z/ |
| cạn kiệt | run dry / be exhausted | *exhausted* ig-**ZOS**-tid — "x" đọc /gz/ |
| ghi log truy vấn | query logging | *query* Anh /ˈkwɪəri/ — "**KWIƠ**-ri" |
| công cụ giám sát hiệu năng ứng dụng | APM tool | đọc từng chữ: "ây-pi-**EM**" |
| đoạn theo dõi trong một request | span | |
| số lần gọi | call count | |
| giá trị tham số | parameter value | pơ-**RA**-mi-tơ — trọng âm 2 |
| đặc trưng | characteristic | ca-rơc-tơ-**RIS**-tic; "ch" đọc /k/ |
| nhân dòng | row multiplication | mul-ti-pli-**CAY**-shơn |
| dòng trùng lặp | duplicated rows | **DIU**-pli-kei-tid |
| lấy thừa dữ liệu | over-fetching | *fetch* — đuôi /-tʃ/ phải bật |
| dòng hàng trong đơn | line item | |
| tích Descartes (nhân chéo hai bảng) | cartesian product | car-**TEE**-zi-ơn |
| gom theo lô | batching | *batch* /bætʃ/; số nhiều *batches* đuôi /-ɪz/ |
| mệnh đề IN | the IN clause | *clause* /klɔːz/ — "clô-z", đuôi /z/ |
| tập kết quả | result set | |
| ghép dữ liệu trong bộ nhớ | stitch data in memory | *stitch* — cụm /st-/ đọc liền |
| hàm phân giải trường | resolver | ri-**ZOL**-vơ — trọng âm 2 |
| lồng nhau | nested | **NES**-tid |
| nhịp của vòng lặp sự kiện | a tick of the event loop | |
| bộ nhớ đệm theo request | per-request cache | *cache* /kæʃ/ — đọc y hệt "cash" |
| giữ đúng thứ tự | preserve the order | pri-**ZERV** — trọng âm 2 |
| thư viện | library | Anh /ˈlaɪbrəri/ — "**LAI**-brơ-ri" |
| phản ứng thái quá | overreaction | ou-vơ-ri-**AC**-shơn |
| cây đối tượng | object tree | danh từ **OB**-jekt — trọng âm 1 |
| tiến trình ứng dụng | application process | |
| hiếm khi | rarely | **REA**-li |
| giáo điều | dogma | **DOG**-mơ |
| tổ hợp tham số | combination of parameters | com-bi-**NAY**-shơn |
| phạm vi giữ khoá | lock scope | |
| ngân sách truy vấn | query budget | *budget* **BU**-jit — "g" đọc /dʒ/ |
| cảnh báo | alert | ơ-**LERT** — trọng âm 2 |
| bộ lập kế hoạch truy vấn | query planner | |
| rủi ro | risk | đuôi /-sk/ phải bật ra, không thành "rít" |

---

## Bài nói

> **Cách luyện:** bật ghi âm, nói **60–90 giây** cho mỗi đề, **không nhìn tài liệu**. Mỗi bài nói phải dùng **ít nhất 6 thuật ngữ** trong bảng trên. Nghe lại bản ghi một lần, đánh dấu chỗ ngập ngừng, rồi nói lại đề đó lần thứ hai.

1. Explain to a junior engineer what an N+1 query is, using an ORM example, and explain why the cost is not in the queries themselves.

2. A colleague says: "We fixed N+1 by turning on eager loading for every relationship in the model." Explain what is wrong with that and what you would do instead.

3. A teammate insists that one big JOIN is always faster than several small queries. Explain when that is not true and how you would settle the argument.

4. Describe what happens when a JOIN across two one-to-many relationships runs, and describe what the application has to do with the result.

5. A GraphQL endpoint returning fifty posts with their authors is slow. Walk your team through how you would diagnose it and how DataLoader would fix it.

6. When would you choose batching with an IN clause over a JOIN, and when would the IN clause itself become the problem?

7. Explain to a product manager why the team wants to set a limit on the number of database queries per request, and what happens if nobody watches that number.
