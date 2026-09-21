# Bài 1 — SQL vs NoSQL & Chọn mô hình dữ liệu
### Bản song ngữ: tiếng Anh bám cấu trúc tiếng Việt

> **Cách dùng bài này:** đọc đoạn tiếng Việt trước, tự dịch trong đầu, rồi mới so với đoạn tiếng Anh bên dưới. Chỗ nào lệch thì xem bảng ánh xạ cụm khó. Cuối cùng, **đọc to đoạn tiếng Anh hai lần** — một lần chậm để đúng âm, một lần với tốc độ nói thật.

---

## Phần 1 — Ba câu hỏi phải đặt trước khi chọn cơ sở dữ liệu

**Tiếng Việt**

Khi chọn cơ sở dữ liệu, chúng ta không nên hỏi "SQL hay NoSQL nhanh hơn", vì đó là một câu hỏi sai ngay từ đầu. Tốc độ không phải là thuộc tính của một loại cơ sở dữ liệu, mà là kết quả của việc ghép đúng công cụ với đúng khối lượng công việc. Thay vào đó, chúng ta phải hỏi ba thứ. Thứ nhất là access pattern: hệ thống đọc và ghi theo kiểu nào, đọc theo id hay đọc theo khoảng, ghi lác đác hay ghi ồ ạt. Thứ hai là yêu cầu nhất quán: nếu dữ liệu sai một chút trong vài giây thì hậu quả là gì. Thứ ba là mô hình dữ liệu: các thực thể có nhiều quan hệ chéo nhau và cần JOIN, hay mỗi bản ghi là một document khép kín. Nếu chúng ta bỏ qua ba câu hỏi này và chọn theo trào lưu, chúng ta sẽ phải trả giá bằng một cuộc migration đắt đỏ sáu tháng sau.

Chúng ta có thể nghĩ về việc chọn cơ sở dữ liệu giống như việc chọn phương tiện đi lại. Không có "chiếc xe nhanh nhất" một cách tuyệt đối, chỉ có chiếc xe hợp với quãng đường và món hàng cần chở. Một chiếc xe tải thua xe máy trong ngõ hẹp, nhưng thắng tuyệt đối khi phải chở hai tấn hàng. Trong buổi phỏng vấn, người hỏi muốn nghe chúng ta suy luận từ yêu cầu ra lựa chọn, chứ không muốn nghe tên sản phẩm. Vì vậy, chúng ta nên mở câu trả lời bằng access pattern, và chỉ nêu tên cơ sở dữ liệu ở cuối.

**English (bám cấu trúc tiếng Việt)**

When we choose a database, we should not ask "which one is faster, SQL or NoSQL", because that is a wrong question from the start. Speed is not a property of a database type, but a result of matching the right tool with the right workload. Instead, we must ask three things. The first one is the access pattern: how the system reads and writes, whether it reads by id or reads by range, whether it writes now and then or writes in huge volume. The second one is the consistency requirement: if the data is slightly wrong for a few seconds, what the consequence is. The third one is the data model: whether the entities have many cross relationships and need JOINs, or whether each record is a self-contained document. If we skip these three questions and choose by trend, we will pay for it with an expensive migration six months later.

We can think about choosing a database in the same way as choosing a means of transport. There is no "fastest vehicle" in an absolute sense, there is only the vehicle that fits the distance and the goods we need to carry. A truck loses to a motorbike in a narrow alley, but it wins completely when it has to carry two tons of goods. In an interview, the person asking wants to hear us reason from the requirements to the choice, and does not want to hear a product name. Therefore, we should open our answer with the access pattern, and only name the database at the end.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| đó là một câu hỏi sai ngay từ đầu | that is a wrong question from the start |
| kết quả của việc ghép đúng công cụ với đúng khối lượng công việc | a result of matching the right tool with the right workload |
| ghi lác đác hay ghi ồ ạt | whether it writes now and then or writes in huge volume |
| sai một chút trong vài giây | slightly wrong for a few seconds |
| có nhiều quan hệ chéo nhau | have many cross relationships |
| một document khép kín | a self-contained document |
| chọn theo trào lưu | choose by trend |
| trả giá bằng một cuộc migration đắt đỏ | pay for it with an expensive migration |
| một cách tuyệt đối | in an absolute sense |
| suy luận từ yêu cầu ra lựa chọn | reason from the requirements to the choice |
| chỉ nêu tên … ở cuối | only name … at the end |

**Thuật ngữ cần nhớ**

- kiểu truy cập → **access pattern**
- yêu cầu nhất quán → **consistency requirement**
- khối lượng công việc → **workload**
- đọc theo khoảng → **read by range**
- chuyển đổi hệ dữ liệu → **migration**

---

## Phần 2 — Bốn họ NoSQL và câu hỏi truy cập của mỗi họ

**Tiếng Việt**

NoSQL không phải là một loại cơ sở dữ liệu, mà là một cái tên chung cho bốn họ rất khác nhau. Họ thứ nhất là document, lưu dữ liệu dưới dạng JSON lồng nhau, và tối ưu cho việc đọc hoặc ghi trọn một aggregate theo id, ví dụ MongoDB, dùng cho catalog sản phẩm và hồ sơ người dùng. Họ thứ hai là key-value, ánh xạ một khoá sang một giá trị, và tối ưu cho get và set theo khoá, ví dụ Redis và DynamoDB, dùng cho cache, session và bộ đếm. Họ thứ ba là wide-column, nhóm các cột theo partition key, và tối ưu cho việc ghi cực lớn vào từng partition, ví dụ Cassandra và ScyllaDB, dùng cho time-series, feed và log. Họ thứ tư là graph, lưu node và edge, và tối ưu cho việc duyệt quan hệ nhiều bậc, ví dụ Neo4j, dùng cho mạng xã hội, phát hiện gian lận và gợi ý. Bốn họ này giải quyết bốn bài toán khác nhau, vì vậy chúng ta không thể so sánh chúng bằng một con số duy nhất. Nếu chúng ta gọi chung tất cả là "NoSQL" trong buổi phỏng vấn, người nghe sẽ kết luận rằng chúng ta chưa từng vận hành một hệ thống thật.

Cách nhớ đơn giản nhất là gắn mỗi họ với một câu hỏi truy cập. Document trả lời "cho tôi trọn đối tượng này", key-value trả lời "cho tôi giá trị của khoá này", wide-column trả lời "cho tôi dải bản ghi trong partition này", còn graph trả lời "cho tôi những node cách node này ba bước". Khi chúng ta nghe một yêu cầu, chúng ta nên dịch nó thành một trong bốn câu hỏi trên trước. Sau đó, tên cơ sở dữ liệu sẽ tự hiện ra, và lý do lựa chọn cũng tự có sẵn.

**English (bám cấu trúc tiếng Việt)**

NoSQL is not one type of database, but a common name for four very different families. The first family is document, which stores data as nested JSON, and is optimised for reading or writing a whole aggregate by id, for example MongoDB, used for product catalogs and user profiles. The second family is key-value, which maps one key to one value, and is optimised for get and set by key, for example Redis and DynamoDB, used for cache, session and counters. The third family is wide-column, which groups columns by a partition key, and is optimised for very large writes into each partition, for example Cassandra and ScyllaDB, used for time-series, feeds and logs. The fourth family is graph, which stores nodes and edges, and is optimised for walking relationships across many hops, for example Neo4j, used for social networks, fraud detection and recommendation. These four families solve four different problems, therefore we cannot compare them with a single number. If we call all of them "NoSQL" in an interview, the listener will conclude that we have never operated a real system.

The simplest way to remember them is to attach each family to an access question. Document answers "give me this whole object", key-value answers "give me the value of this key", wide-column answers "give me the range of records in this partition", and graph answers "give me the nodes that are three steps away from this node". When we hear a requirement, we should translate it into one of the four questions above first. After that, the name of the database will appear by itself, and the reason for the choice will also be ready.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| một cái tên chung cho bốn họ rất khác nhau | a common name for four very different families |
| JSON lồng nhau | nested JSON |
| đọc hoặc ghi trọn một aggregate theo id | reading or writing a whole aggregate by id |
| ánh xạ một khoá sang một giá trị | maps one key to one value |
| ghi cực lớn vào từng partition | very large writes into each partition |
| duyệt quan hệ nhiều bậc | walking relationships across many hops |
| phát hiện gian lận | fraud detection |
| so sánh chúng bằng một con số duy nhất | compare them with a single number |
| chưa từng vận hành một hệ thống thật | have never operated a real system |
| gắn mỗi họ với một câu hỏi truy cập | attach each family to an access question |
| tên … sẽ tự hiện ra | the name … will appear by itself |

**Thuật ngữ cần nhớ**

- họ cơ sở dữ liệu → **database family**
- khoá phân mảnh → **partition key**
- hồ sơ người dùng → **user profile**
- bộ đếm → **counter**
- duyệt quan hệ → **traverse relationships**
- dải bản ghi → **range of records**

---

## Phần 3 — Vì sao hệ quan hệ vẫn là lựa chọn mặc định

**Tiếng Việt**

Hệ quan hệ như PostgreSQL và MySQL cho chúng ta bốn thứ cùng một lúc: quan hệ giữa các bảng, phép JOIN, bảo đảm ACID mạnh, và một schema tường minh. Thứ quý nhất trong bốn thứ đó là schema tường minh, vì nó buộc chúng ta phải nói rõ dữ liệu trông như thế nào ngay tại tầng cơ sở dữ liệu. Ưu điểm lớn nhất của hệ quan hệ là nó không ép chúng ta đoán trước access pattern. Khi sản phẩm đổi hướng và một truy vấn mới xuất hiện, chúng ta chỉ cần viết một câu JOIN mới và thêm một index, và chúng ta không cần thiết kế lại toàn bộ kho dữ liệu. Vì vậy, PostgreSQL nên là lựa chọn mặc định cho hệ thống giao dịch trực tuyến, và mọi lựa chọn lệch khỏi nó phải có một lý do đo được. Nếu chúng ta rời khỏi hệ quan hệ quá sớm, chúng ta sẽ phải tự viết lại bằng tay những thứ mà cơ sở dữ liệu vốn cho không: kiểm tra ràng buộc, transaction, và toàn vẹn tham chiếu.

Có một niềm tin cũ rằng muốn scale thì phải đổi sang NoSQL. Niềm tin này đã lỗi thời, vì PostgreSQL hiện nay có JSONB cho thuộc tính linh hoạt, có partition cho bảng lớn, và có replica cho tải đọc. Trong phần lớn trường hợp, ba thứ này đã đủ để đi rất xa. Đổi cơ sở dữ liệu là một việc rất tốn kém, vì chúng ta phải viết lại tầng truy cập dữ liệu, chuyển toàn bộ dữ liệu cũ, và chạy song song hai hệ trong nhiều tháng. Vì vậy, chúng ta nên coi việc đổi cơ sở dữ liệu là phương án cuối cùng, chứ không phải phương án đầu tiên.

**English (bám cấu trúc tiếng Việt)**

Relational databases such as PostgreSQL and MySQL give us four things at the same time: relationships between tables, the JOIN operation, strong ACID guarantees, and an explicit schema. The most valuable one among those four is the explicit schema, because it forces us to state clearly what the data looks like right at the database layer. The biggest advantage of a relational database is that it does not force us to guess the access pattern in advance. When the product changes direction and a new query appears, we only need to write a new JOIN and add an index, and we do not need to redesign the whole data store. Therefore, PostgreSQL should be the default choice for an online transactional system, and every choice that moves away from it must have a measurable reason. If we leave the relational world too early, we will have to rewrite by hand the things that the database normally gives us for free: constraint checking, transactions, and referential integrity.

There is an old belief that if we want to scale, we have to switch to NoSQL. This belief is out of date, because PostgreSQL today has JSONB for flexible attributes, partitioning for large tables, and replicas for read load. In most cases, these three things are already enough to go very far. Changing the database is a very expensive job, because we have to rewrite the data access layer, move all the old data, and run two systems in parallel for many months. Therefore, we should treat changing the database as the last option, and not as the first option.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| cho chúng ta bốn thứ cùng một lúc | give us four things at the same time |
| một schema tường minh | an explicit schema |
| nó buộc chúng ta phải nói rõ | it forces us to state clearly |
| ngay tại tầng cơ sở dữ liệu | right at the database layer |
| không ép chúng ta đoán trước | does not force us to guess in advance |
| khi sản phẩm đổi hướng | when the product changes direction |
| mọi lựa chọn lệch khỏi nó | every choice that moves away from it |
| một lý do đo được | a measurable reason |
| tự viết lại bằng tay | rewrite by hand |
| những thứ … vốn cho không | the things … normally gives us for free |
| niềm tin này đã lỗi thời | this belief is out of date |
| chạy song song hai hệ | run two systems in parallel |

**Thuật ngữ cần nhớ**

- hệ quan hệ → **relational database**
- bảo đảm ACID mạnh → **strong ACID guarantees**
- toàn vẹn tham chiếu → **referential integrity**
- tầng truy cập dữ liệu → **data access layer**
- hệ thống giao dịch trực tuyến → **online transactional system (OLTP)**
- tải đọc → **read load**

---

## Phần 4 — "Schema-less" không có nghĩa là khỏi thiết kế schema

**Tiếng Việt**

Nhiều người hiểu chữ "schema-less" theo nghĩa là chúng ta không cần thiết kế schema nữa. Cách hiểu này sai, vì schema không biến mất, nó chỉ dời lên tầng ứng dụng. Dữ liệu vẫn có hình dạng, và code vẫn phải biết hình dạng đó để đọc được. Điều duy nhất thay đổi là ai chịu trách nhiệm bảo vệ hình dạng ấy: trước đây là cơ sở dữ liệu, bây giờ là chúng ta. Nếu chúng ta bỏ qua trách nhiệm này, chúng ta sẽ gặp schema drift, nghĩa là sau hai năm cùng một collection chứa năm phiên bản document khác nhau. Hậu quả tiếp theo là chúng ta phải tự viết code kiểm tra dữ liệu ở mọi chỗ đọc, và bộ tối ưu truy vấn cũng khó làm việc vì nó không biết trước một trường có mặt hay không. Vì vậy, khi dùng một cơ sở dữ liệu document, chúng ta vẫn phải viết schema ra giấy, đặt phiên bản cho document, và bật validation ở tầng cơ sở dữ liệu nếu có.

**English (bám cấu trúc tiếng Việt)**

Many people understand the word "schema-less" as meaning that we no longer need to design a schema. This understanding is wrong, because the schema does not disappear, it only moves up to the application layer. The data still has a shape, and the code still has to know that shape in order to read it. The only thing that changes is who takes responsibility for protecting that shape: before it was the database, now it is us. If we skip this responsibility, we will meet schema drift, which means that after two years the same collection holds five different versions of the document. The next consequence is that we have to write validation code ourselves at every read site, and the query optimiser also has a hard time working because it does not know in advance whether a field is present or not. Therefore, when we use a document database, we still have to write the schema down, put a version on the document, and turn on validation at the database layer if it exists.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| hiểu chữ … theo nghĩa là | understand the word … as meaning that |
| nó chỉ dời lên tầng ứng dụng | it only moves up to the application layer |
| dữ liệu vẫn có hình dạng | the data still has a shape |
| ai chịu trách nhiệm bảo vệ hình dạng ấy | who takes responsibility for protecting that shape |
| bỏ qua trách nhiệm này | skip this responsibility |
| chứa năm phiên bản document khác nhau | holds five different versions of the document |
| ở mọi chỗ đọc | at every read site |
| khó làm việc | has a hard time working |
| nó không biết trước một trường có mặt hay không | it does not know in advance whether a field is present or not |
| viết schema ra giấy | write the schema down |
| đặt phiên bản cho document | put a version on the document |

**Thuật ngữ cần nhớ**

- trôi dạt lược đồ → **schema drift**
- tầng ứng dụng → **application layer**
- kiểm tra hợp lệ → **validation**
- bộ tối ưu truy vấn → **query optimiser**
- trường dữ liệu → **field**

---

## Phần 5 — Câu "NoSQL không có ACID" đã lỗi thời

**Tiếng Việt**

Câu nói "NoSQL không có ACID" từng đúng cách đây mười năm, nhưng bây giờ nó đã lỗi thời. MongoDB hỗ trợ transaction ACID trên nhiều document từ phiên bản 4.0 với replica set, và từ phiên bản 4.2 với cụm sharded. Câu trả lời chính xác trong buổi phỏng vấn là "tuỳ cơ sở dữ liệu", chứ không phải "NoSQL thì không có". Tuy nhiên, chúng ta phải nói thêm rằng khả năng này đi kèm chi phí và giới hạn, vì một transaction trải trên nhiều node làm tăng độ trễ và giữ khoá lâu hơn. Nói cách khác, tính năng đó có tồn tại, nhưng nó không tự nhiên mà có được. Nếu chúng ta thiết kế một hệ thống mà mọi thao tác đều cần transaction nhiều document, đó là dấu hiệu cho thấy chúng ta đang dùng sai loại cơ sở dữ liệu.

**English (bám cấu trúc tiếng Việt)**

The saying "NoSQL has no ACID" used to be true ten years ago, but now it is out of date. MongoDB has supported ACID transactions across many documents since version 4.0 with a replica set, and since version 4.2 with a sharded cluster. The accurate answer in an interview is "it depends on the database", and not "NoSQL does not have it". However, we must add that this ability comes with a cost and with limits, because a transaction that spreads across many nodes increases latency and holds locks for longer. In other words, that feature does exist, but it does not come for free. If we design a system where every operation needs a multi-document transaction, that is a sign that we are using the wrong type of database.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| từng đúng cách đây mười năm | used to be true ten years ago |
| bây giờ nó đã lỗi thời | now it is out of date |
| transaction ACID trên nhiều document | ACID transactions across many documents |
| tuỳ cơ sở dữ liệu | it depends on the database |
| chúng ta phải nói thêm rằng | we must add that |
| đi kèm chi phí và giới hạn | comes with a cost and with limits |
| trải trên nhiều node | spreads across many nodes |
| giữ khoá lâu hơn | holds locks for longer |
| nó không tự nhiên mà có được | it does not come for free |
| đó là dấu hiệu cho thấy | that is a sign that |
| dùng sai loại cơ sở dữ liệu | using the wrong type of database |

**Thuật ngữ cần nhớ**

- giao dịch nhiều document → **multi-document transaction**
- cụm phân mảnh → **sharded cluster**
- độ trễ → **latency**
- giữ khoá → **hold a lock**
- lỗi thời → **out of date / outdated**

---

## Phần 6 — Embed hay reference khi mô hình hoá trong MongoDB

**Tiếng Việt**

Khi mô hình hoá một quan hệ trong MongoDB, chúng ta phải chọn giữa embed và reference. Chúng ta nên embed khi dữ liệu con luôn được đọc cùng dữ liệu cha, và khi số lượng phần tử con bị chặn trên. Chúng ta nên reference khi số lượng phần tử con không bị chặn, khi dữ liệu con được chia sẻ giữa nhiều cha, hoặc khi dữ liệu con được cập nhật độc lập. Ví dụ, một người dùng có vài địa chỉ thì nên embed, còn một người dùng có hàng nghìn bình luận thì phải reference. Đổi lại, embed cho chúng ta tốc độ đọc vì chúng ta chỉ cần một lần đi tới cơ sở dữ liệu, nhưng document sẽ phình ra và MongoDB giới hạn mỗi document ở mười sáu megabyte. Reference giữ cho document nhỏ gọn, nhưng chúng ta phải "join tay" ở tầng ứng dụng hoặc dùng lookup, và điều này dễ dẫn đến vấn đề N+1. Nếu chúng ta embed một danh sách không bị chặn, hệ thống sẽ chạy tốt trong sáu tháng đầu rồi chết đột ngột khi một document chạm giới hạn.

**English (bám cấu trúc tiếng Việt)**

When we model a relationship in MongoDB, we have to choose between embedding and referencing. We should embed when the child data is always read together with the parent data, and when the number of child elements is bounded. We should reference when the number of child elements is unbounded, when the child data is shared between many parents, or when the child data is updated independently. For example, a user who has a few addresses should be embedded, while a user who has thousands of comments must be referenced. In exchange, embedding gives us read speed because we only need one trip to the database, but the document will grow fat and MongoDB limits each document to sixteen megabytes. Referencing keeps the document small, but we have to "join by hand" at the application layer or use a lookup, and this easily leads to the N+1 problem. If we embed a list that is not bounded, the system will run fine for the first six months and then die suddenly when one document hits the limit.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| chọn giữa embed và reference | choose between embedding and referencing |
| được đọc cùng dữ liệu cha | is read together with the parent data |
| số lượng phần tử con bị chặn trên | the number of child elements is bounded |
| được chia sẻ giữa nhiều cha | is shared between many parents |
| được cập nhật độc lập | is updated independently |
| chỉ cần một lần đi tới cơ sở dữ liệu | we only need one trip to the database |
| document sẽ phình ra | the document will grow fat |
| giữ cho document nhỏ gọn | keeps the document small |
| "join tay" ở tầng ứng dụng | "join by hand" at the application layer |
| dễ dẫn đến vấn đề N+1 | easily leads to the N+1 problem |
| chết đột ngột khi một document chạm giới hạn | die suddenly when one document hits the limit |

**Thuật ngữ cần nhớ**

- nhúng dữ liệu con vào cha → **embed**
- tham chiếu bằng khoá → **reference**
- bị chặn trên / không bị chặn → **bounded / unbounded**
- dữ liệu cha và dữ liệu con → **parent data and child data**
- một lần đi tới cơ sở dữ liệu → **one round trip to the database**

---

## Phần 7 — Wide-column và tư duy "truy vấn trước, bảng sau"

**Tiếng Việt**

Wide-column bắt chúng ta đảo ngược thứ tự thiết kế, vì ở đây truy vấn đến trước còn bảng đến sau. Chúng ta liệt kê các truy vấn đã biết trước, rồi chúng ta tạo một bảng riêng phục vụ cho từng truy vấn. Chúng ta chấp nhận ghi trùng cùng một dữ liệu vào nhiều bảng, và chúng ta không chuẩn hoá. Cách làm này thắng PostgreSQL khi lượng ghi cực lớn, khi hệ thống trải trên nhiều vùng địa lý, và khi mọi truy vấn đều đi qua partition key. Nhưng nó thua ngay lập tức khi xuất hiện một truy vấn mới không nằm trong danh sách ban đầu, vì chúng ta không thể JOIN để cứu vãn tình thế. Vì vậy, chúng ta chỉ nên chọn wide-column khi access pattern đã ổn định và chúng ta tin rằng nó sẽ không đổi.

**English (bám cấu trúc tiếng Việt)**

Wide-column forces us to reverse the design order, because here the query comes first and the table comes second. We list the queries that we already know in advance, then we create a separate table serving each query. We accept writing the same data into many tables, and we do not normalise. This approach beats PostgreSQL when the write volume is very large, when the system spreads across many geographic regions, and when every query goes through the partition key. But it loses immediately when a new query appears that is not in the original list, because we cannot JOIN to save the situation. Therefore, we should only choose wide-column when the access pattern is already stable and we believe that it will not change.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| đảo ngược thứ tự thiết kế | reverse the design order |
| truy vấn đến trước còn bảng đến sau | the query comes first and the table comes second |
| các truy vấn đã biết trước | the queries that we already know in advance |
| một bảng riêng phục vụ cho từng truy vấn | a separate table serving each query |
| chấp nhận ghi trùng cùng một dữ liệu | accept writing the same data |
| khi lượng ghi cực lớn | when the write volume is very large |
| trải trên nhiều vùng địa lý | spreads across many geographic regions |
| nó thua ngay lập tức | it loses immediately |
| không nằm trong danh sách ban đầu | is not in the original list |
| để cứu vãn tình thế | to save the situation |
| khi access pattern đã ổn định | when the access pattern is already stable |

**Thuật ngữ cần nhớ**

- thiết kế theo truy vấn → **query-first design**
- ghi trùng dữ liệu → **data duplication**
- chuẩn hoá → **normalise / normalisation**
- lượng ghi → **write volume**
- nhiều vùng địa lý → **multi-region**

---

## Phần 8 — Polyglot persistence và cái giá phải trả

**Tiếng Việt**

Polyglot persistence nghĩa là chúng ta dùng nhiều loại kho dữ liệu trong cùng một hệ thống, mỗi loại cho một việc mà nó làm tốt nhất. Một kiến trúc điển hình gồm PostgreSQL cho bản ghi nghiệp vụ và tiền, Redis cho cache, Elasticsearch cho tìm kiếm, và khi cần thì chúng ta thêm Cassandra cho luồng sự kiện. Cách làm này cho chúng ta hiệu năng tốt trên từng loại truy vấn, nhưng nó không tự nhiên mà có được. Đổi lại, chúng ta phải vận hành nhiều hệ thống hơn, chúng ta phải giám sát nhiều thứ hơn, và chúng ta phải đồng bộ dữ liệu giữa các kho. Phần khó nằm ở lúc một kho đã cập nhật còn kho kia thì chưa, vì lúc đó người dùng nhìn thấy hai câu trả lời khác nhau cho cùng một câu hỏi. Trong ngành, nhiều sản phẩm rất lớn vẫn chạy lõi quan hệ cho giao dịch trực tuyến, Redis gần như là mặc định cho cache, và Discord từng chuyển kho tin nhắn từ MongoDB sang Cassandra rồi sang ScyllaDB. Vì vậy, chúng ta nên bắt đầu bằng một kho duy nhất, và chúng ta chỉ thêm kho mới khi có một con số đo được chứng minh rằng kho hiện tại không kham nổi.

**English (bám cấu trúc tiếng Việt)**

Polyglot persistence means that we use many types of data store in the same system, each type for the job that it does best. A typical architecture includes PostgreSQL for business records and money, Redis for cache, Elasticsearch for search, and when needed we add Cassandra for the event stream. This approach gives us good performance on each type of query, but it does not come for free. In exchange, we have to operate more systems, we have to monitor more things, and we have to keep the data in sync between the stores. The hard part lies in the moment when one store has been updated while the other one has not, because at that point the user sees two different answers to the same question. In the industry, many very large products still run a relational core for online transactions, Redis is almost the default for cache, and Discord once moved its message store from MongoDB to Cassandra and then to ScyllaDB. Therefore, we should start with a single store, and we only add a new store when there is a measurable number proving that the current store cannot cope.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| mỗi loại cho một việc mà nó làm tốt nhất | each type for the job that it does best |
| bản ghi nghiệp vụ và tiền | business records and money |
| luồng sự kiện | the event stream |
| nó không tự nhiên mà có được | it does not come for free |
| đổi lại | in exchange |
| đồng bộ dữ liệu giữa các kho | keep the data in sync between the stores |
| phần khó nằm ở lúc | the hard part lies in the moment when |
| một kho đã cập nhật còn kho kia thì chưa | one store has been updated while the other one has not |
| vẫn chạy lõi quan hệ | still run a relational core |
| một con số đo được chứng minh rằng | a measurable number proving that |
| kho hiện tại không kham nổi | the current store cannot cope |

**Thuật ngữ cần nhớ**

- dùng nhiều loại kho dữ liệu → **polyglot persistence**
- kho dữ liệu → **data store**
- giám sát → **monitor / monitoring**
- đồng bộ → **keep in sync**
- lõi quan hệ → **relational core**
- kho tin nhắn → **message store**

---

## Mô hình ghi nhớ

**Tiếng Việt**

Chúng ta chọn cơ sở dữ liệu theo access pattern, theo yêu cầu nhất quán và theo mô hình quan hệ, chứ không theo tốc độ. PostgreSQL là lựa chọn mặc định, và mọi lựa chọn lệch khỏi nó phải có một lý do đo được.

**English (bám cấu trúc tiếng Việt)**

We choose a database by access pattern, by consistency requirement and by the relationship model, and not by speed. PostgreSQL is the default choice, and every choice that moves away from it must have a measurable reason.

---

## Bảng thuật ngữ tổng hợp

| Tiếng Việt | Tiếng Anh | Ghi chú phát âm |
|---|---|---|
| kiểu truy cập | access pattern | *pattern* Anh /ˈpætn/ — "PÆT-tơn", không đọc "pat-tơn-nờ" |
| yêu cầu nhất quán | consistency requirement | trọng âm 2: con-**SIS**-ten-cy |
| khối lượng công việc | workload | |
| lược đồ dữ liệu | schema | /ˈskiːmə/ — "SKII-mờ", trọng âm âm đầu; đừng đọc "sờ-kê-ma" |
| lược đồ tường minh | explicit schema | ex-**PLI**-cit |
| trôi dạt lược đồ | schema drift | |
| truy vấn | query | Anh /ˈkwɪəri/ — "KWIƠ-ri" |
| bộ tối ưu truy vấn | query optimiser | trọng âm 1: **OP**-ti-mai-zơ |
| bộ nhớ đệm | cache | /kæʃ/ — đọc y hệt "cash"; **không** đọc "ca-chê" |
| aggregate (khối dữ liệu trọn vẹn) | aggregate | danh từ /ˈæɡrɪɡət/ "**AG**-ri-gợt"; động từ đọc đuôi /-eɪt/ |
| khoá phân mảnh | partition key | par-**TI**-tion, Anh /pɑːˈtɪʃn/ |
| cụm phân mảnh | sharded cluster | *shard* /ʃɑːd/ — âm "sh" đầu, đuôi /-dɪd/ rõ |
| độ trễ | latency | /ˈleɪtənsi/ — "**LAY**-tơn-si", không đọc "la-ten-si" |
| thông lượng ghi | write throughput | *throughput* /ˈθruːpʊt/ — âm **th** đầu lưỡi, không thành "trút" |
| toàn vẹn tham chiếu | referential integrity | re-fe-**REN**-tial · in-**TEG**-ri-ty |
| bảo đảm ACID mạnh | strong ACID guarantees | *guarantee* trọng âm cuối: ga-ran-**TEE** |
| giao dịch nhiều document | multi-document transaction | |
| chuyển đổi hệ dữ liệu | migration | my-**GRAY**-shun — âm đầu là "my", không phải "mi" |
| kiểm tra hợp lệ | validation | va-li-**DAY**-shun |
| kiến trúc | architecture | **AR**-ki-tek-chơ — "ch" trong *arch* đọc /k/ |
| thuộc tính | attribute | danh từ trọng âm 1: **AT**-tri-biut; động từ a-**TTRI**-bute |
| dùng nhiều loại kho dữ liệu | polyglot persistence | **PO**-ly-glot · per-**SIS**-tence |
| tầng truy cập dữ liệu | data access layer | |
| tầng ứng dụng | application layer | |
| hệ quan hệ | relational database | |
| lõi quan hệ | relational core | |
| hệ thống giao dịch trực tuyến | online transactional system (OLTP) | |
| bị chặn trên / không bị chặn | bounded / unbounded | |
| nhúng / tham chiếu | embed / reference | |
| tải đọc | read load | |
| bản sao đọc | read replica | *replica* trọng âm 1: **REP**-li-cơ |
| chuẩn hoá | normalise | Anh viết *-ise*: **NOR**-mơ-lai-z |
| ghi trùng dữ liệu | data duplication | du-pli-**CAY**-shun |
| thiết kế theo truy vấn | query-first design | |
| nhiều vùng địa lý | multi-region | |
| kho dữ liệu | data store | |
| luồng sự kiện | event stream | *stream* — cụm phụ âm /str-/ đọc liền, không chèn nguyên âm |
| rủi ro | risk | đuôi /-sk/ phải bật ra, không thành "rít" |
| giám sát | monitor | **MO**-ni-tơ |
| phát hiện gian lận | fraud detection | *fraud* /frɔːd/ — "phrôd" |
| duyệt quan hệ | traverse relationships | tra-**VERSE** |
| nhất quán cuối cùng | eventual consistency | e-**VEN**-tu-al |

---

## Bài nói

> **Cách luyện:** bật ghi âm, nói **60–90 giây** cho mỗi đề, **không nhìn tài liệu**. Mỗi bài nói phải dùng **ít nhất 6 thuật ngữ** trong bảng trên. Nghe lại bản ghi một lần, đánh dấu chỗ ngập ngừng, rồi nói lại đề đó lần thứ hai.

1. Explain to a junior engineer why "which one is faster, SQL or NoSQL" is the wrong question, and what three questions we should ask instead.

2. A colleague says: "MongoDB is schema-less, so we do not need to design a schema for this service." Explain what is wrong with that, and what will happen to the codebase after two years.

3. Someone on your team wants to move the main order database from PostgreSQL to Cassandra "so that it scales more easily". Explain why you would push back, and what you would measure before you agree.

4. Describe what happens over time when we embed an unbounded list of comments inside a user document in MongoDB, and describe how the failure shows up in production.

5. When would you choose a wide-column store over PostgreSQL, and when would that same choice become a problem?

6. A senior colleague says: "NoSQL databases do not support ACID, so we cannot use MongoDB for anything involving money." Explain what is accurate and what is out of date in that statement.

7. Explain to a product manager what polyglot persistence is, why it gives better performance, and what price the team pays for it in daily operation.
