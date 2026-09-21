# Bài 13 — Sharding & Partitioning
### Bản song ngữ: tiếng Anh bám cấu trúc tiếng Việt

> **Cách dùng bài này:** đọc đoạn tiếng Việt trước, tự dịch trong đầu, rồi mới so với đoạn tiếng Anh bên dưới. Chỗ nào lệch thì xem bảng ánh xạ cụm khó. Cuối cùng, **đọc to đoạn tiếng Anh hai lần** — một lần chậm để đúng âm, một lần với tốc độ nói thật.

---

## Phần 1 — Partition và shard khác nhau ở đâu

**Tiếng Việt**

Hai chữ partition và shard hay bị dùng lẫn với nhau, nhưng chúng nói về hai việc khác nhau. Partition nghĩa là chúng ta chia một bảng lớn thành nhiều bảng con nằm trong cùng một instance cơ sở dữ liệu. Về mặt logic, người dùng vẫn nhìn thấy một bảng duy nhất, và cơ sở dữ liệu tự biết dòng nào thuộc về bảng con nào. Shard nghĩa là chúng ta chia dữ liệu ra nhiều instance nằm trên nhiều máy khác nhau, vì vậy chúng ta có nhiều cơ sở dữ liệu độc lập. Sự khác biệt về độ khó là rất lớn, vì partition chỉ là một quyết định bên trong một cơ sở dữ liệu còn shard là một quyết định về kiến trúc của cả hệ thống. Cách nhớ đơn giản là partition chia phòng bên trong một ngôi nhà, còn shard thì chia người ở ra nhiều ngôi nhà khác nhau.

**English (bám cấu trúc tiếng Việt)**

The two words partition and shard are often mixed up with each other, but they talk about two different things. Partition means that we split one large table into many child tables sitting inside the same database instance. Logically, the user still sees one single table, and the database itself knows which row belongs to which child table. Shard means that we split the data across many instances sitting on many different machines, therefore we have many independent databases. The difference in difficulty is very large, because a partition is only a decision inside one database while a shard is a decision about the architecture of the whole system. A simple way to remember it is that partitioning divides the rooms inside one house, while sharding divides the residents across many different houses.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| hay bị dùng lẫn với nhau | are often mixed up with each other |
| chia một bảng lớn thành nhiều bảng con | split one large table into many child tables |
| về mặt logic | logically |
| cơ sở dữ liệu tự biết dòng nào thuộc về bảng con nào | the database itself knows which row belongs to which child table |
| nằm trên nhiều máy khác nhau | sitting on many different machines |
| sự khác biệt về độ khó là rất lớn | the difference in difficulty is very large |
| chỉ là một quyết định bên trong một cơ sở dữ liệu | is only a decision inside one database |
| cách nhớ đơn giản là | a simple way to remember it is that |
| chia người ở ra nhiều ngôi nhà khác nhau | divides the residents across many different houses |

**Thuật ngữ cần nhớ**

- chia bảng trong một máy → **partitioning**
- chia dữ liệu ra nhiều máy → **sharding**
- bảng con → **child table**
- độc lập → **independent**
- kiến trúc hệ thống → **system architecture**

---

## Phần 2 — Ba kiểu chia bảng và cách chọn

**Tiếng Việt**

PostgreSQL cho chúng ta ba kiểu chia bảng, và chúng ta chọn theo cách truy vấn cùng với phân bố dữ liệu. Chia theo khoảng nghĩa là mỗi bảng con giữ một khoảng giá trị liên tục, và kiểu này hợp nhất với dữ liệu theo thời gian. Chia theo danh sách nghĩa là mỗi bảng con giữ một tập giá trị rời rạc, ví dụ mỗi vùng địa lý một bảng con. Chia theo băm nghĩa là cơ sở dữ liệu băm khoá rồi phân dòng đều ra các bảng con, và kiểu này hợp khi chúng ta chỉ cần phân bố đều. Nếu chúng ta chọn kiểu chia không khớp với điều kiện lọc thường dùng, cơ sở dữ liệu vẫn phải đọc mọi bảng con và chúng ta không được lợi gì. Vì vậy, chúng ta luôn bắt đầu từ câu hỏi truy vấn nóng đang lọc theo cột nào, rồi chúng ta mới chọn kiểu chia.

**English (bám cấu trúc tiếng Việt)**

PostgreSQL gives us three kinds of partitioning, and we choose according to the query pattern together with the data distribution. Range partitioning means that each child table holds one continuous range of values, and this kind fits time-based data best. List partitioning means that each child table holds one discrete set of values, for example one child table per geographic region. Hash partitioning means that the database hashes the key and then spreads the rows evenly across the child tables, and this kind fits when we only need an even distribution. If we choose a partitioning kind that does not match the filter condition we normally use, the database still has to read every child table and we gain nothing. Therefore, we always start from the question of which column the hot query is filtering on, and only then do we choose the partitioning kind.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| theo cách truy vấn cùng với phân bố dữ liệu | according to the query pattern together with the data distribution |
| một khoảng giá trị liên tục | one continuous range of values |
| kiểu này hợp nhất với dữ liệu theo thời gian | this kind fits time-based data best |
| một tập giá trị rời rạc | one discrete set of values |
| mỗi vùng địa lý một bảng con | one child table per geographic region |
| phân dòng đều ra các bảng con | spreads the rows evenly across the child tables |
| không khớp với điều kiện lọc thường dùng | does not match the filter condition we normally use |
| chúng ta không được lợi gì | we gain nothing |
| truy vấn nóng đang lọc theo cột nào | which column the hot query is filtering on |

**Thuật ngữ cần nhớ**

- chia theo khoảng → **range partitioning**
- chia theo danh sách → **list partitioning**
- chia theo băm → **hash partitioning**
- phân bố dữ liệu → **data distribution**
- rời rạc → **discrete**

---

## Phần 3 — Lợi ích thật của partition nằm ở khâu vận hành

**Tiếng Việt**

Lợi ích đầu tiên của việc chia bảng là cắt tỉa phân vùng, nghĩa là bộ lập kế hoạch chỉ đọc những bảng con thật sự chứa dữ liệu cần tìm. Nếu chúng ta chia theo tháng và truy vấn chỉ hỏi một ngày, cơ sở dữ liệu sẽ bỏ qua toàn bộ các tháng khác. Nhưng lợi ích lớn hơn nhiều lại nằm ở khâu vận hành chứ nó không nằm ở tốc độ đọc. Khi chúng ta muốn xoá dữ liệu cũ hơn một năm, chúng ta chỉ cần tách hoặc xoá nguyên một bảng con, và việc đó gần như tức thì. Nếu chúng ta dùng lệnh DELETE cho hàng trăm triệu dòng, chúng ta sẽ tạo ra một lượng dead tuple khổng lồ và bảng sẽ phình lên. Ngoài ra, chúng ta có thể chạy VACUUM và tạo index theo từng bảng con, vì vậy mỗi lần bảo trì chỉ chạm vào một phần nhỏ của dữ liệu. Chính vì hai lợi ích vận hành này, chia bảng theo thời gian gần như luôn là lựa chọn đúng cho bảng log và bảng sự kiện.

**English (bám cấu trúc tiếng Việt)**

The first benefit of partitioning is partition pruning, which means that the planner only reads the child tables that really contain the data we are looking for. If we partition by month and the query only asks about one day, the database will skip all the other months. But the much bigger benefit lies in operations and it does not lie in read speed. When we want to delete data older than one year, we only need to detach or drop a whole child table, and that is almost instant. If we use a DELETE statement for hundreds of millions of rows, we will create a huge amount of dead tuples and the table will grow fat. In addition, we can run VACUUM and build indexes per child table, therefore each maintenance run only touches a small part of the data. Precisely because of these two operational benefits, partitioning by time is almost always the right choice for log tables and event tables.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| cắt tỉa phân vùng | partition pruning |
| những bảng con thật sự chứa dữ liệu cần tìm | the child tables that really contain the data we are looking for |
| sẽ bỏ qua toàn bộ các tháng khác | will skip all the other months |
| lợi ích lớn hơn nhiều lại nằm ở khâu vận hành | the much bigger benefit lies in operations |
| tách hoặc xoá nguyên một bảng con | detach or drop a whole child table |
| việc đó gần như tức thì | that is almost instant |
| một lượng dead tuple khổng lồ | a huge amount of dead tuples |
| mỗi lần bảo trì chỉ chạm vào một phần nhỏ của dữ liệu | each maintenance run only touches a small part of the data |
| chính vì hai lợi ích vận hành này | precisely because of these two operational benefits |

**Thuật ngữ cần nhớ**

- cắt tỉa phân vùng → **partition pruning**
- tách phân vùng ra khỏi bảng cha → **detach a partition**
- gần như tức thì → **almost instant**
- bảo trì → **maintenance**
- bảng sự kiện → **event table**

---

## Phần 4 — Chọn shard key và cái bẫy shard nóng

**Tiếng Việt**

Khi chúng ta thật sự phải shard, quyết định quan trọng nhất là chọn shard key. Shard key quyết định mỗi dòng sẽ đi về node nào, và một lựa chọn sai sẽ theo chúng ta suốt nhiều năm. Sai lầm kinh điển là shard theo ngày tạo, vì khi đó mọi lệnh ghi trong hôm nay đều dồn về đúng một node. Node đó trở thành shard nóng, nó chịu toàn bộ tải ghi, trong khi các node cũ thì gần như nằm im. Kết quả là chúng ta đã trả cái giá của một hệ phân tán mà chúng ta không hề nhận được khả năng mở rộng. Lựa chọn tốt hơn là băm theo một khoá có nhiều giá trị và được truy cập đều, ví dụ mã khách thuê hoặc mã người dùng. Chúng ta cũng phải kiểm tra phân bố thật của dữ liệu, vì một khách thuê chiếm bốn mươi phần trăm lưu lượng sẽ lại tạo ra một shard nóng mới.

**English (bám cấu trúc tiếng Việt)**

When we really have to shard, the most important decision is choosing the shard key. The shard key decides which node each row will go to, and a wrong choice will follow us for many years. The classic mistake is sharding by the creation date, because then every write during today lands on exactly one node. That node becomes a hot shard, it takes the whole write load, while the older nodes sit almost idle. The result is that we have paid the price of a distributed system while we have not received any scalability at all. A better choice is to hash on a key that has many values and is accessed evenly, for example the tenant id or the user id. We also have to check the real distribution of the data, because one tenant taking forty percent of the traffic will create a new hot shard again.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| khi chúng ta thật sự phải shard | when we really have to shard |
| quyết định mỗi dòng sẽ đi về node nào | decides which node each row will go to |
| sẽ theo chúng ta suốt nhiều năm | will follow us for many years |
| mọi lệnh ghi trong hôm nay đều dồn về đúng một node | every write during today lands on exactly one node |
| các node cũ thì gần như nằm im | the older nodes sit almost idle |
| chúng ta đã trả cái giá của một hệ phân tán | we have paid the price of a distributed system |
| mà chúng ta không hề nhận được khả năng mở rộng | while we have not received any scalability at all |
| một khoá có nhiều giá trị và được truy cập đều | a key that has many values and is accessed evenly |
| chiếm bốn mươi phần trăm lưu lượng | taking forty percent of the traffic |

**Thuật ngữ cần nhớ**

- khoá phân mảnh → **shard key**
- mảnh bị dồn tải → **hot shard**
- nằm im, không tải → **sit idle**
- khả năng mở rộng → **scalability**
- khách thuê trong hệ đa khách → **tenant**

---

## Phần 5 — Truy vấn xuyên shard là phần ác mộng

**Tiếng Việt**

Điều làm cho shard trở nên khó không phải là việc chia dữ liệu, mà là những truy vấn phải đi qua nhiều shard. Khi một câu truy vấn không có shard key trong điều kiện lọc, hệ thống phải gửi câu hỏi tới mọi node rồi gom kết quả lại. Kiểu thực thi này được gọi là phát tán và thu gom, và độ trễ của nó bằng độ trễ của node chậm nhất. Tệ hơn nữa là phép JOIN, vì hai bảng nằm trên hai node khác nhau thì không thể nối với nhau một cách tự nhiên. Tệ nhất là transaction, vì một transaction trải trên nhiều node kéo chúng ta trở lại bài toán giao dịch phân tán mà chúng ta đã cố tránh. Vì vậy, mục tiêu khi thiết kế shard key là làm cho gần như mọi truy vấn nóng chỉ chạm vào đúng một shard.

**English (bám cấu trúc tiếng Việt)**

The thing that makes sharding hard is not splitting the data, but the queries that have to go across many shards. When a query does not have the shard key in its filter condition, the system has to send the question to every node and then gather the results back. This kind of execution is called scatter and gather, and its latency equals the latency of the slowest node. Worse than that is the JOIN, because two tables sitting on two different nodes cannot be joined together naturally. Worst of all is the transaction, because a transaction spread across many nodes drags us back to the distributed transaction problem that we tried to avoid. Therefore, the goal when we design the shard key is to make almost every hot query touch exactly one shard.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| điều làm cho shard trở nên khó | the thing that makes sharding hard |
| những truy vấn phải đi qua nhiều shard | the queries that have to go across many shards |
| gửi câu hỏi tới mọi node rồi gom kết quả lại | send the question to every node and then gather the results back |
| kiểu thực thi này được gọi là | this kind of execution is called |
| độ trễ của nó bằng độ trễ của node chậm nhất | its latency equals the latency of the slowest node |
| tệ hơn nữa là | worse than that is |
| không thể nối với nhau một cách tự nhiên | cannot be joined together naturally |
| kéo chúng ta trở lại bài toán … mà chúng ta đã cố tránh | drags us back to the … problem that we tried to avoid |
| chỉ chạm vào đúng một shard | touch exactly one shard |

**Thuật ngữ cần nhớ**

- truy vấn xuyên nhiều mảnh → **cross-shard query**
- phát tán và thu gom → **scatter and gather**
- giao dịch phân tán → **distributed transaction**
- node chậm nhất → **the slowest node**
- điều kiện lọc → **filter condition**

---

## Phần 6 — Consistent hashing và bài toán thêm bớt node

**Tiếng Việt**

Cách chia đơn giản nhất là lấy giá trị băm chia lấy dư cho số node, nhưng cách này có một khuyết điểm nghiêm trọng. Khi chúng ta thêm hoặc bớt một node, số dư của gần như mọi khoá đều đổi, vì vậy chúng ta phải di chuyển gần như toàn bộ dữ liệu. Consistent hashing giải quyết đúng vấn đề đó bằng cách đặt cả node lẫn khoá lên một vòng tròn băm. Mỗi khoá thuộc về node đầu tiên mà chúng ta gặp khi đi theo chiều kim đồng hồ trên vòng tròn đó. Khi một node được thêm vào, chỉ những khoá nằm trong cung mà node đó phụ trách mới phải di chuyển, còn phần còn lại thì đứng yên. Trong thực tế, người ta còn dùng node ảo để phân bố đều hơn, vì nếu không thì một node có thể vô tình phụ trách một cung quá rộng.

**English (bám cấu trúc tiếng Việt)**

The simplest way to split is to take the hash value modulo the number of nodes, but this way has one serious weakness. When we add or remove one node, the remainder of almost every key changes, therefore we have to move almost all of the data. Consistent hashing solves exactly that problem by placing both the nodes and the keys on a hash ring. Each key belongs to the first node that we meet when we walk clockwise around that ring. When one node is added, only the keys lying in the arc that this node takes over have to move, while the rest stay where they are. In practice, people also use virtual nodes for a more even distribution, because otherwise one node could accidentally take over an arc that is too wide.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| lấy giá trị băm chia lấy dư cho số node | take the hash value modulo the number of nodes |
| có một khuyết điểm nghiêm trọng | has one serious weakness |
| số dư của gần như mọi khoá đều đổi | the remainder of almost every key changes |
| bằng cách đặt cả node lẫn khoá lên một vòng tròn băm | by placing both the nodes and the keys on a hash ring |
| khi đi theo chiều kim đồng hồ | when we walk clockwise |
| những khoá nằm trong cung mà node đó phụ trách | the keys lying in the arc that this node takes over |
| còn phần còn lại thì đứng yên | while the rest stay where they are |
| người ta còn dùng node ảo | people also use virtual nodes |
| vì nếu không thì | because otherwise |
| vô tình phụ trách một cung quá rộng | accidentally take over an arc that is too wide |

**Thuật ngữ cần nhớ**

- băm nhất quán → **consistent hashing**
- phép chia lấy dư → **modulo**
- vòng tròn băm → **hash ring**
- theo chiều kim đồng hồ → **clockwise**
- node ảo → **virtual node**

---

## Phần 7 — Lộ trình mở rộng theo bậc thang, shard là cửa cuối

**Tiếng Việt**

Câu hỏi thách đố ở cấp tech lead thường có dạng bảng đơn hàng đã hai tỉ dòng và tải thì vẫn đang tăng. Câu trả lời sai là chúng ta nói ngay rằng cần shard, vì shard làm cho độ phức tạp của hệ thống tăng lên rất nhiều. Câu trả lời đúng bắt đầu bằng việc đo, vì chúng ta phải biết điểm nghẽn thật đang nằm ở đọc, ở ghi hay ở một truy vấn cụ thể. Sau khi đo, chúng ta đi lần lượt qua các bậc thang rẻ hơn, đó là tối ưu index và truy vấn, nâng cấu hình máy, chia bảng, thêm bản sao đọc, và thêm cache. Chỉ khi tất cả những bậc đó đã hết đường, và khi điểm nghẽn nằm rõ ràng ở khả năng ghi của một máy duy nhất, chúng ta mới nói tới shard. Ngay cả lúc đó, chúng ta vẫn nên cân nhắc một lớp hạ tầng làm sẵn việc shard thay vì tự viết trong mã ứng dụng. Cách trình bày theo bậc thang này chính là thứ mà người phỏng vấn muốn nghe, vì nó cho thấy chúng ta cân nhắc chi phí chứ chúng ta không chạy theo giải pháp thời thượng.

**English (bám cấu trúc tiếng Việt)**

The challenge question at tech lead level usually takes the form of an orders table that already has two billion rows while the load is still growing. The wrong answer is that we say straight away that we need to shard, because sharding makes the complexity of the system go up enormously. The right answer starts with measuring, because we have to know whether the real bottleneck is on reads, on writes or on one specific query. After measuring, we go through the cheaper steps one by one, namely tuning the indexes and the queries, scaling the machine up, partitioning the table, adding read replicas, and adding a cache. Only when all of those steps have run out, and when the bottleneck clearly sits in the write capacity of one single machine, do we talk about sharding. Even at that point, we should still consider an infrastructure layer that does the sharding for us instead of writing it ourselves in the application code. This way of presenting it as a ladder is exactly what the interviewer wants to hear, because it shows that we weigh the cost and we do not chase the fashionable solution.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| thường có dạng | usually takes the form of |
| tải thì vẫn đang tăng | the load is still growing |
| chúng ta nói ngay rằng cần shard | we say straight away that we need to shard |
| làm cho độ phức tạp của hệ thống tăng lên rất nhiều | makes the complexity of the system go up enormously |
| đi lần lượt qua các bậc thang rẻ hơn | go through the cheaper steps one by one |
| nâng cấu hình máy | scaling the machine up |
| khi tất cả những bậc đó đã hết đường | when all of those steps have run out |
| ngay cả lúc đó | even at that point |
| một lớp hạ tầng làm sẵn việc shard | an infrastructure layer that does the sharding for us |
| chúng ta không chạy theo giải pháp thời thượng | we do not chase the fashionable solution |

**Thuật ngữ cần nhớ**

- lộ trình mở rộng → **scaling path**
- nâng cấu hình máy → **scale up (vertical scaling)**
- độ phức tạp → **complexity**
- hết đường → **run out of options**
- cân nhắc chi phí → **weigh the cost**

---

## Phần 8 — Đẩy việc shard xuống tầng hạ tầng

**Tiếng Việt**

Trong quá khứ, các đội thường tự viết logic shard ngay trong mã ứng dụng. Cách đó buộc mọi lập trình viên phải nhớ shard key, phải tự gom kết quả, và phải tự xử lý giao dịch phân tán. Ngày nay, chúng ta có thể đẩy phần lớn công việc đó xuống tầng hạ tầng. Với PostgreSQL, chúng ta có Citus, một extension biến một cụm Postgres thành một cơ sở dữ liệu phân tán. Với MySQL, vai trò tương đương thuộc về Vitess, và đó là hệ thống đã chạy cho những sản phẩm rất lớn. Lợi ích là ứng dụng vẫn nói SQL như bình thường, còn việc định tuyến và gom kết quả thì do hạ tầng lo, và chúng ta sẽ nói tiếp về hướng này trong bài cuối cùng.

**English (bám cấu trúc tiếng Việt)**

In the past, teams usually wrote the sharding logic themselves right inside the application code. That approach forced every developer to remember the shard key, to gather the results themselves, and to handle distributed transactions themselves. Nowadays, we can push most of that work down into the infrastructure layer. For PostgreSQL, we have Citus, an extension that turns a Postgres cluster into a distributed database. For MySQL, the equivalent role belongs to Vitess, and that is a system which has run for some very large products. The benefit is that the application still speaks plain SQL, while the routing and the result gathering are handled by the infrastructure, and we will talk further about this direction in the final lesson.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| tự viết logic shard ngay trong mã ứng dụng | wrote the sharding logic themselves right inside the application code |
| cách đó buộc mọi lập trình viên phải nhớ | that approach forced every developer to remember |
| đẩy phần lớn công việc đó xuống tầng hạ tầng | push most of that work down into the infrastructure layer |
| biến một cụm Postgres thành một cơ sở dữ liệu phân tán | turns a Postgres cluster into a distributed database |
| vai trò tương đương thuộc về | the equivalent role belongs to |
| đã chạy cho những sản phẩm rất lớn | has run for some very large products |
| ứng dụng vẫn nói SQL như bình thường | the application still speaks plain SQL |
| thì do hạ tầng lo | are handled by the infrastructure |
| chúng ta sẽ nói tiếp về hướng này | we will talk further about this direction |

**Thuật ngữ cần nhớ**

- tầng hạ tầng → **the infrastructure layer**
- phần mở rộng của Postgres → **extension**
- cụm máy chủ → **cluster**
- vai trò tương đương → **the equivalent role**
- định tuyến → **routing**

---

## Mô hình ghi nhớ

**Tiếng Việt**

Partition chia một bảng bên trong một máy, còn shard chia dữ liệu ra nhiều máy, vì vậy shard là cửa cuối cùng sau khi index, bản sao đọc, cache và partition đã hết đường. Một shard key sai sẽ tạo ra shard nóng và những truy vấn xuyên shard rất đắt.

**English (bám cấu trúc tiếng Việt)**

Partitioning splits one table inside one machine, while sharding splits the data across many machines, therefore sharding is the last door after indexes, read replicas, caching and partitioning have run out. A wrong shard key will create a hot shard and cross-shard queries that are very expensive.

---

## Bảng thuật ngữ tổng hợp

| Tiếng Việt | Tiếng Anh | Ghi chú phát âm |
|---|---|---|
| chia bảng trong một máy | partitioning | par-**TI**-shơn-ing |
| chia dữ liệu ra nhiều máy | sharding | *shard* /ʃɑːd/ — âm "sh" đầu, nguyên âm dài |
| bảng con | child table | *child* /tʃaɪld/ — đuôi /-ld/ đủ hai âm |
| độc lập | independent | in-đi-**PEN**-đơnt |
| kiến trúc hệ thống | system architecture | **AR**-ki-tek-chơ — "ch" trong *arch* đọc /k/ |
| chia theo khoảng | range partitioning | |
| chia theo danh sách | list partitioning | |
| chia theo băm | hash partitioning | |
| phân bố dữ liệu | data distribution | dis-tri-**BIU**-shơn |
| liên tục | continuous | cơn-**TI**-niu-ơs |
| rời rạc | discrete | đis-**CREET** — trọng âm 2 |
| vùng địa lý | geographic region | ji-ơ-**GRA**-fic · **REE**-jơn |
| cắt tỉa phân vùng | partition pruning | *pruning* **PRU**-ning |
| tách phân vùng | detach a partition | đi-**TATCH** — đuôi /-tʃ/ phải bật |
| xoá bảng | drop a table | |
| gần như tức thì | almost instant | **IN**-stơnt |
| phiên bản chết | dead tuple | *tuple* Anh /ˈtjuːpl/ — "**TIU**-pl" |
| phình bảng | bloat | /bləʊt/ — vần với "boat" |
| bảo trì | maintenance | **MEIN**-tơ-nơns — trọng âm 1 |
| khoá phân mảnh | shard key | |
| mảnh bị dồn tải | hot shard | |
| nằm im, không tải | sit idle | *idle* **AI**-đl |
| khả năng mở rộng | scalability | skei-lơ-**BI**-lơ-ti |
| khách thuê trong hệ đa khách | tenant | **TE**-nơnt, không đọc "ti-nơnt" |
| lưu lượng | traffic | |
| truy vấn xuyên nhiều mảnh | cross-shard query | *query* Anh /ˈkwɪəri/ — "**KWIƠ**-ri" |
| phát tán và thu gom | scatter and gather | **SCA**-tơ · **GA**-đơ (âm cuối /ð/) |
| giao dịch phân tán | distributed transaction | dis-**TRI**-biu-tid |
| băm nhất quán | consistent hashing | |
| phép chia lấy dư | modulo | **MO**-điu-lou |
| số dư | remainder | ri-**MEIN**-đơ |
| vòng tròn băm | hash ring | |
| theo chiều kim đồng hồ | clockwise | **CLOK**-waiz |
| cung tròn | arc | /ɑːk/ — "ak", chữ **c** đọc /k/ |
| node ảo | virtual node | **VER**-chu-ơl |
| lộ trình mở rộng | scaling path | |
| nâng cấu hình máy | scale up / vertical scaling | **VER**-ti-cơl |
| mở rộng ngang | scale out | |
| độ phức tạp | complexity | cơm-**PLEK**-sơ-ti |
| hết đường | run out of options | |
| cân nhắc chi phí | weigh the cost | *weigh* /weɪ/ — chữ **gh** câm |
| thời thượng | fashionable | **FA**-shơn-ơ-bl |
| tầng hạ tầng | the infrastructure layer | **IN**-frơ-strơc-chơ — trọng âm 1 |
| phần mở rộng | extension | ex-**TEN**-shơn |
| cụm máy chủ | cluster | **CLUS**-tơ |
| định tuyến | routing | Anh /ˈruːtɪŋ/ — "**RU**-ting" |
| rủi ro | risk | đuôi /-sk/ phải bật ra |

---

## Bài nói

> **Cách luyện:** bật ghi âm, nói **60–90 giây** cho mỗi đề, **không nhìn tài liệu**. Mỗi bài nói phải dùng **ít nhất 6 thuật ngữ** trong bảng trên. Nghe lại bản ghi một lần, đánh dấu chỗ ngập ngừng, rồi nói lại đề đó lần thứ hai.

1. Explain the difference between partitioning and sharding to a junior engineer, and say why one is far harder than the other.

2. A colleague proposes sharding the events table by creation date because "the data is naturally time-based". Explain why you would push back and what key you would suggest instead.

3. Someone on your team says the orders table is getting large, so the team should start sharding this quarter. Walk them through the cheaper steps you would take first and what evidence would change your mind.

4. Describe what happens when a query without the shard key runs against a sharded system.

5. Explain why `hash modulo N` is a poor sharding scheme, and describe how consistent hashing improves on it.

6. Your events table keeps only thirty days of data. Explain to the team how partitioning changes the way you delete the old data, and why that matters.

7. When would you use Citus or a distributed SQL product instead of writing sharding logic in the application, and what do you give up by doing that?
