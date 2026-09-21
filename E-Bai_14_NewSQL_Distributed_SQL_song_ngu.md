# Bài 14 — NewSQL / Distributed SQL
### Bản song ngữ: tiếng Anh bám cấu trúc tiếng Việt

> **Cách dùng bài này:** đọc đoạn tiếng Việt trước, tự dịch trong đầu, rồi mới so với đoạn tiếng Anh bên dưới. Chỗ nào lệch thì xem bảng ánh xạ cụm khó. Cuối cùng, **đọc to đoạn tiếng Anh hai lần** — một lần chậm để đúng âm, một lần với tốc độ nói thật.

---

## Phần 1 — NewSQL phá bỏ sự lựa chọn giữa SQL và khả năng mở rộng

**Tiếng Việt**

Trong nhiều năm, chúng ta bị buộc phải chọn giữa hai thứ tốt. Một bên là cơ sở dữ liệu quan hệ với SQL, với quan hệ giữa các bảng và với transaction ACID, nhưng nó chỉ mở rộng khả năng ghi trong giới hạn của một máy. Bên kia là NoSQL với khả năng mở rộng ngang rất tốt, nhưng chúng ta phải hi sinh phép JOIN và hi sinh transaction trên nhiều dòng. NewSQL, mà ngày nay người ta thường gọi là distributed SQL, ra đời để phá bỏ sự lựa chọn đó. Nó hứa cho chúng ta cả ba thứ cùng một lúc, đó là SQL đầy đủ, bảo đảm ACID, và khả năng mở rộng ngang trên nhiều node. Để làm được điều đó, nó tự lo phần chia mảnh, tự lo phần nhân bản, và tự lo phần giao dịch phân tán mà trước đây chúng ta phải viết tay.

**English (bám cấu trúc tiếng Việt)**

For many years, we were forced to choose between two good things. On one side there is the relational database with SQL, with relationships between tables and with ACID transactions, but it only scales its write capacity within the limits of one machine. On the other side there is NoSQL with very good horizontal scalability, but we have to sacrifice the JOIN and sacrifice transactions across many rows. NewSQL, which people nowadays usually call distributed SQL, was born in order to break that choice. It promises us all three things at the same time, namely full SQL, ACID guarantees, and horizontal scalability across many nodes. In order to do that, it takes care of the sharding itself, takes care of the replication itself, and takes care of the distributed transactions that we previously had to write by hand.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| chúng ta bị buộc phải chọn giữa hai thứ tốt | we were forced to choose between two good things |
| trong giới hạn của một máy | within the limits of one machine |
| chúng ta phải hi sinh phép JOIN | we have to sacrifice the JOIN |
| mà ngày nay người ta thường gọi là | which people nowadays usually call |
| ra đời để phá bỏ sự lựa chọn đó | was born in order to break that choice |
| nó hứa cho chúng ta cả ba thứ cùng một lúc | it promises us all three things at the same time |
| nó tự lo phần chia mảnh | it takes care of the sharding itself |
| mà trước đây chúng ta phải viết tay | that we previously had to write by hand |

**Thuật ngữ cần nhớ**

- cơ sở dữ liệu quan hệ → **relational database**
- mở rộng ngang → **horizontal scalability**
- hi sinh, đánh đổi → **sacrifice**
- bảo đảm ACID → **ACID guarantees**
- giao dịch phân tán → **distributed transaction**

---

## Phần 2 — Bức tranh sản phẩm và điểm mạnh riêng của từng cái tên

**Tiếng Việt**

Chúng ta nên biết vài cái tên chính, nhưng chúng ta phải luôn kiểm chứng lại vì thị trường này đổi rất nhanh. CockroachDB tương thích giao thức của PostgreSQL, nó nhắm vào hệ giao dịch trải nhiều vùng, và nó đặt mức serializable làm mặc định. YugabyteDB cũng tương thích PostgreSQL, nó tách lớp truy vấn khỏi lớp lưu trữ, và nó thường được so sánh trực tiếp với CockroachDB. TiDB thì tương thích MySQL, và điểm mạnh riêng của nó là chạy được cả tải giao dịch lẫn tải phân tích trên cùng một hệ. Google Spanner là sản phẩm được quản lý hoàn toàn trên nền tảng của Google, và nó dùng đồng hồ nguyên tử để đồng bộ thời gian. Amazon Aurora DSQL là hướng distributed SQL không máy chủ và nó tích hợp sâu với hệ sinh thái của AWS. Khi nói về nhóm này trong buổi phỏng vấn, chúng ta nên nêu điểm mạnh riêng của từng sản phẩm thay vì chỉ đọc tên chúng ra.

**English (bám cấu trúc tiếng Việt)**

We should know a few of the main names, but we must always verify them again because this market changes very fast. CockroachDB is compatible with the PostgreSQL wire protocol, it aims at transactional systems spread across many regions, and it sets serializable as the default level. YugabyteDB is also PostgreSQL compatible, it separates the query layer from the storage layer, and it is often compared directly with CockroachDB. TiDB on the other hand is MySQL compatible, and its own strength is that it can run both transactional load and analytical load on the same system. Google Spanner is a fully managed product on Google's platform, and it uses atomic clocks to synchronise time. Amazon Aurora DSQL is the serverless direction of distributed SQL and it integrates deeply with the AWS ecosystem. When we talk about this group in an interview, we should state the particular strength of each product instead of just reading their names out.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| chúng ta phải luôn kiểm chứng lại | we must always verify them again |
| vì thị trường này đổi rất nhanh | because this market changes very fast |
| tương thích giao thức của PostgreSQL | compatible with the PostgreSQL wire protocol |
| nó nhắm vào hệ giao dịch trải nhiều vùng | it aims at transactional systems spread across many regions |
| nó tách lớp truy vấn khỏi lớp lưu trữ | it separates the query layer from the storage layer |
| điểm mạnh riêng của nó là | its own strength is that |
| được quản lý hoàn toàn | fully managed |
| nó tích hợp sâu với hệ sinh thái | it integrates deeply with the ecosystem |
| thay vì chỉ đọc tên chúng ra | instead of just reading their names out |

**Thuật ngữ cần nhớ**

- tương thích giao thức → **wire compatible**
- tải giao dịch và tải phân tích → **transactional and analytical load**
- được quản lý hoàn toàn → **fully managed**
- hệ sinh thái → **ecosystem**
- kiểm chứng lại → **verify**

---

## Phần 3 — Đồng thuận Raft là cách giữ nhất quán mạnh

**Tiếng Việt**

Câu hỏi thú vị nhất là làm sao một hệ trải trên nhiều máy vẫn giữ được tính nhất quán mạnh. Câu trả lời là thuật toán đồng thuận, và thuật toán phổ biến nhất trong nhóm sản phẩm này là Raft. Dữ liệu được chia thành các dải, và mỗi dải có vài bản sao đặt trên các node khác nhau. Trong mỗi nhóm bản sao, một node được bầu làm trưởng nhóm, và mọi lệnh ghi vào dải đó đều phải đi qua trưởng nhóm. Trưởng nhóm chỉ coi một lệnh ghi là thành công khi đa số bản sao đã ghi nó vào log của mình. Nhờ quy tắc đa số này, hệ thống vẫn phục vụ được và vẫn không mất dữ liệu khi một số node chết, vì nhóm còn lại tự bầu ra một trưởng nhóm mới.

**English (bám cấu trúc tiếng Việt)**

The most interesting question is how a system spread across many machines still keeps strong consistency. The answer is a consensus algorithm, and the most common algorithm in this group of products is Raft. The data is split into ranges, and each range has several replicas placed on different nodes. Within each replica group, one node is elected as the leader, and every write into that range has to go through the leader. The leader only treats a write as successful when a majority of the replicas have written it into their own log. Thanks to this majority rule, the system still serves traffic and still loses no data when some nodes die, because the remaining group elects a new leader by itself.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| làm sao một hệ trải trên nhiều máy vẫn giữ được | how a system spread across many machines still keeps |
| thuật toán đồng thuận | a consensus algorithm |
| dữ liệu được chia thành các dải | the data is split into ranges |
| vài bản sao đặt trên các node khác nhau | several replicas placed on different nodes |
| một node được bầu làm trưởng nhóm | one node is elected as the leader |
| chỉ coi một lệnh ghi là thành công khi | only treats a write as successful when |
| đa số bản sao đã ghi nó vào log của mình | a majority of the replicas have written it into their own log |
| nhờ quy tắc đa số này | thanks to this majority rule |
| nhóm còn lại tự bầu ra một trưởng nhóm mới | the remaining group elects a new leader by itself |

**Thuật ngữ cần nhớ**

- thuật toán đồng thuận → **consensus algorithm**
- dải dữ liệu → **range**
- nhóm bản sao → **replica group**
- bầu trưởng nhóm → **leader election**
- đa số → **majority**

---

## Phần 4 — Spanner và cách dùng thời gian để sắp thứ tự

**Tiếng Việt**

Google Spanner giải bài toán theo một hướng khác, và hướng đó dựa trên thời gian. Trong một hệ phân tán, đồng hồ của các máy không bao giờ khớp nhau một cách tuyệt đối, vì vậy chúng ta rất khó nói giao dịch nào đã xảy ra trước. Google trang bị đồng hồ nguyên tử và máy thu tín hiệu vệ tinh cho các trung tâm dữ liệu, rồi họ xây một dịch vụ thời gian tên là TrueTime. Điểm đặc biệt là TrueTime không trả về một mốc thời gian duy nhất, mà nó trả về một khoảng chắc chắn chứa thời điểm thật. Khi commit, Spanner chờ hết khoảng bất định đó rồi nó mới trả kết quả, nhờ vậy thứ tự của các giao dịch là chắc chắn trên phạm vi toàn cầu. Cái giá là mỗi lần commit phải chờ thêm vài mili giây, và đó là ví dụ rõ nhất cho việc nhất quán mạnh luôn được mua bằng thời gian.

**English (bám cấu trúc tiếng Việt)**

Google Spanner solves the problem in a different direction, and that direction is based on time. In a distributed system, the clocks of the machines never match each other absolutely, therefore it is very hard for us to say which transaction happened first. Google equips its data centres with atomic clocks and satellite receivers, then they built a time service named TrueTime. The special point is that TrueTime does not return one single timestamp, but it returns an interval that certainly contains the real moment. At commit time, Spanner waits out that uncertainty interval before it returns the result, thanks to which the order of the transactions is certain on a global scale. The price is that each commit has to wait a few extra milliseconds, and that is the clearest example of strong consistency always being bought with time.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| giải bài toán theo một hướng khác | solves the problem in a different direction |
| không bao giờ khớp nhau một cách tuyệt đối | never match each other absolutely |
| chúng ta rất khó nói giao dịch nào đã xảy ra trước | it is very hard for us to say which transaction happened first |
| trang bị … cho các trung tâm dữ liệu | equips its data centres with … |
| máy thu tín hiệu vệ tinh | satellite receivers |
| một khoảng chắc chắn chứa thời điểm thật | an interval that certainly contains the real moment |
| chờ hết khoảng bất định đó | waits out that uncertainty interval |
| trên phạm vi toàn cầu | on a global scale |
| nhất quán mạnh luôn được mua bằng thời gian | strong consistency is always bought with time |

**Thuật ngữ cần nhớ**

- đồng hồ nguyên tử → **atomic clock**
- mốc thời gian → **timestamp**
- khoảng bất định → **uncertainty interval**
- chờ trước khi trả kết quả commit → **commit wait**
- nhất quán mạnh → **strong consistency**

---

## Phần 5 — Ba khoản chi phí mà chúng ta phải nói ra

**Tiếng Việt**

Distributed SQL nghe rất hấp dẫn, nhưng nó có ba khoản chi phí mà chúng ta phải nói ra. Khoản thứ nhất là độ trễ, vì một giao dịch chạm vào nhiều node phải chờ đồng thuận qua mạng. Nếu các bản sao nằm ở nhiều vùng địa lý, mỗi lần đồng thuận có thể tốn hàng chục mili giây, và con số đó cộng thẳng vào từng lệnh ghi. Khoản thứ hai là vận hành, vì chúng ta phải hiểu thêm về dải dữ liệu, về chính sách đặt bản sao và về hành vi của hệ thống khi mất một vùng. Khoản thứ ba là tiền, vì các hệ này thường đòi nhiều node và giá của chúng cao hơn hẳn một cơ sở dữ liệu được quản lý thông thường. Điều đáng nói là rất nhiều cụm trong thực tế không hề cần chạy trên nhiều vùng. Trong những trường hợp đó, chúng ta trả cả ba khoản chi phí trên mà chúng ta không dùng tới thứ mà mình đã mua.

**English (bám cấu trúc tiếng Việt)**

Distributed SQL sounds very attractive, but it has three costs that we have to say out loud. The first cost is latency, because a transaction touching many nodes has to wait for consensus over the network. If the replicas sit in many geographic regions, each round of consensus can cost tens of milliseconds, and that number adds directly onto every write. The second cost is operations, because we have to understand more about data ranges, about replica placement policies and about how the system behaves when a region is lost. The third cost is money, because these systems usually demand many nodes and their price is clearly higher than an ordinary managed database. What is worth saying is that a great many clusters in practice do not need to run across many regions at all. In those cases, we pay all three of the costs above while we never use the thing that we have bought.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| nghe rất hấp dẫn | sounds very attractive |
| ba khoản chi phí mà chúng ta phải nói ra | three costs that we have to say out loud |
| phải chờ đồng thuận qua mạng | has to wait for consensus over the network |
| con số đó cộng thẳng vào từng lệnh ghi | that number adds directly onto every write |
| chính sách đặt bản sao | replica placement policies |
| hành vi của hệ thống khi mất một vùng | how the system behaves when a region is lost |
| cao hơn hẳn một cơ sở dữ liệu được quản lý thông thường | clearly higher than an ordinary managed database |
| điều đáng nói là | what is worth saying is that |
| chúng ta không dùng tới thứ mà mình đã mua | we never use the thing that we have bought |

**Thuật ngữ cần nhớ**

- độ trễ → **latency**
- vùng địa lý → **geographic region**
- chính sách đặt dữ liệu → **placement policy**
- chi phí vận hành → **operational cost**
- cơ sở dữ liệu được quản lý → **managed database**

---

## Phần 6 — Ba tiêu chí để quyết định

**Tiếng Việt**

Vì vậy, chúng ta cần một bộ tiêu chí rõ ràng để quyết định, chứ chúng ta không quyết định theo cảm giác hiện đại. Tiêu chí thứ nhất là chúng ta có thật sự cần nhất quán mạnh trên nhiều vùng địa lý hay không. Tiêu chí thứ hai là tải ghi của chúng ta có thật sự vượt quá khả năng của một máy duy nhất hay không. Tiêu chí thứ ba là chúng ta có bị ràng buộc pháp lý về nơi lưu trữ dữ liệu hay không, ví dụ dữ liệu của người dùng châu Âu phải nằm trong châu Âu. Nếu câu trả lời cho cả ba tiêu chí đều là không, thì một cơ sở dữ liệu PostgreSQL được quản lý cùng với vài bản sao đọc sẽ đơn giản hơn và rẻ hơn nhiều. Nếu ít nhất một câu trả lời là có, và đặc biệt nếu tiêu chí thứ ba là có, thì distributed SQL trở thành một lựa chọn nghiêm túc.

**English (bám cấu trúc tiếng Việt)**

Therefore, we need a clear set of criteria in order to decide, and we do not decide by the feeling of being modern. The first criterion is whether we really need strong consistency across many geographic regions or not. The second criterion is whether our write load really goes beyond the capacity of one single machine or not. The third criterion is whether we are bound by legal rules about where the data is stored or not, for example the data of European users has to stay inside Europe. If the answer to all three criteria is no, then a managed PostgreSQL database together with a few read replicas will be much simpler and much cheaper. If at least one answer is yes, and especially if the third criterion is yes, then distributed SQL becomes a serious option.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| một bộ tiêu chí rõ ràng để quyết định | a clear set of criteria in order to decide |
| chúng ta không quyết định theo cảm giác hiện đại | we do not decide by the feeling of being modern |
| có thật sự cần … hay không | whether we really need … or not |
| vượt quá khả năng của một máy duy nhất | goes beyond the capacity of one single machine |
| chúng ta có bị ràng buộc pháp lý về nơi lưu trữ dữ liệu | whether we are bound by legal rules about where the data is stored |
| phải nằm trong châu Âu | has to stay inside Europe |
| nếu câu trả lời cho cả ba tiêu chí đều là không | if the answer to all three criteria is no |
| trở thành một lựa chọn nghiêm túc | becomes a serious option |

**Thuật ngữ cần nhớ**

- tiêu chí → **criterion / criteria**
- nơi lưu trữ dữ liệu theo luật → **data residency**
- ràng buộc pháp lý → **legal rules / regulation**
- bản sao đọc → **read replica**
- lựa chọn nghiêm túc → **a serious option**

---

## Phần 7 — Cách phản biện đề xuất "chuyển sang distributed SQL cho hiện đại"

**Tiếng Việt**

Một tình huống rất thực là có người trong đội đề xuất chuyển sang CockroachDB để hệ thống có khả năng mở rộng tốt hơn. Chúng ta không nên bác bỏ ngay, mà chúng ta nên hỏi ba câu tương ứng với ba tiêu chí ở trên. Nếu công ty chỉ chạy trong một vùng và chỉ có một trăm nghìn người dùng, thì lý do thật sự thường chỉ là một truy vấn chậm chưa được tối ưu. Chúng ta cũng nên nhắc rằng chữ tương thích ở đây là tương thích giao thức, chứ nó không phải là tương thích hoàn toàn. Phần lớn câu SQL sẽ chạy y như trên PostgreSQL, nhưng một số stored procedure, một số truy vấn vào bảng hệ thống và vài trường hợp biên vẫn cần sửa. Vì vậy, lời khuyên thực dụng là chúng ta viết SQL chuẩn, chúng ta tránh những tính năng quá đặc thù, và chúng ta chạy thử toàn bộ bộ kiểm thử trước khi cam kết. Cách trả lời này cho thấy chúng ta hiểu công cụ, hiểu chi phí, và biết đặt câu hỏi trước khi đưa ra kết luận.

**English (bám cấu trúc tiếng Việt)**

A very real situation is that someone on the team proposes moving to CockroachDB so that the system scales better. We should not reject it straight away, but we should ask three questions matching the three criteria above. If the company only runs in one region and only has one hundred thousand users, then the real reason is usually just one slow query that has not been tuned. We should also point out that the word compatible here means wire compatible, and it does not mean fully compatible. Most SQL statements will run exactly as they do on PostgreSQL, but some stored procedures, some queries against system tables and a few edge cases will still need fixing. Therefore, the pragmatic advice is that we write standard SQL, we avoid the features that are too specific, and we run the whole test suite before we commit to the move. This way of answering shows that we understand the tool, we understand the cost, and we know how to ask questions before we draw a conclusion.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| có người trong đội đề xuất chuyển sang | someone on the team proposes moving to |
| chúng ta không nên bác bỏ ngay | we should not reject it straight away |
| ba câu tương ứng với ba tiêu chí ở trên | three questions matching the three criteria above |
| lý do thật sự thường chỉ là | the real reason is usually just |
| một truy vấn chậm chưa được tối ưu | one slow query that has not been tuned |
| chứ nó không phải là tương thích hoàn toàn | and it does not mean fully compatible |
| một số truy vấn vào bảng hệ thống | some queries against system tables |
| vài trường hợp biên vẫn cần sửa | a few edge cases will still need fixing |
| chạy thử toàn bộ bộ kiểm thử trước khi cam kết | run the whole test suite before we commit to the move |
| biết đặt câu hỏi trước khi đưa ra kết luận | know how to ask questions before we draw a conclusion |

**Thuật ngữ cần nhớ**

- bác bỏ → **reject**
- trường hợp biên → **edge case**
- bộ kiểm thử → **test suite**
- bảng hệ thống → **system table**
- đưa ra kết luận → **draw a conclusion**

---

## Mô hình ghi nhớ

**Tiếng Việt**

NewSQL cho chúng ta SQL, bảo đảm ACID và khả năng mở rộng ngang nhờ thuật toán đồng thuận, nhưng nó thu phí bằng độ trễ, bằng công vận hành và bằng tiền. Chúng ta chỉ chọn nó khi thật sự cần nhất quán mạnh trên nhiều vùng, còn nếu chỉ chạy trong một vùng thì PostgreSQL được quản lý là lựa chọn đúng.

**English (bám cấu trúc tiếng Việt)**

NewSQL gives us SQL, ACID guarantees and horizontal scalability thanks to a consensus algorithm, but it charges us in latency, in operational work and in money. We only choose it when we really need strong consistency across many regions, and if we only run in one region then managed PostgreSQL is the right choice.

---

## Bảng thuật ngữ tổng hợp

| Tiếng Việt | Tiếng Anh | Ghi chú phát âm |
|---|---|---|
| cơ sở dữ liệu quan hệ | relational database | ri-**LAY**-shơn-ơl |
| mở rộng ngang | horizontal scalability | *horizontal* ho-ri-**ZON**-tơl — trọng âm 3 |
| hi sinh, đánh đổi | sacrifice | **SA**-cri-fais — trọng âm 1, đuôi /s/ |
| bảo đảm ACID | ACID guarantees | *guarantee* trọng âm cuối: ga-rơn-**TEE** |
| giao dịch phân tán | distributed transaction | dis-**TRI**-biu-tid |
| tương thích giao thức | wire compatible | *wire* /waɪə/ — "wai-ơ"; cơm-**PA**-tơ-bl |
| tải giao dịch | transactional load | tran-**SAC**-shơn-ơl |
| tải phân tích | analytical load | a-nơ-**LY**-ti-cơl |
| được quản lý hoàn toàn | fully managed | **MA**-nijd |
| hệ sinh thái | ecosystem | **EE**-cou-sis-tơm — trọng âm 1 |
| kiểm chứng lại | verify | **VE**-ri-fai — trọng âm 1 |
| thuật toán | algorithm | **AL**-gơ-ri-đơm — "th" cuối đọc /ð/ |
| đồng thuận | consensus | cơn-**SEN**-sơs — trọng âm 2 |
| dải dữ liệu | range | |
| nhóm bản sao | replica group | **REP**-li-cơ — trọng âm 1 |
| bầu trưởng nhóm | leader election | i-**LEC**-shơn |
| đa số | majority | mơ-**JO**-rơ-ti — trọng âm 2 |
| nhật ký ghi trước | write-ahead log | *write* — chữ **w** câm |
| đồng hồ nguyên tử | atomic clock | ơ-**TO**-mic — trọng âm 2 |
| mốc thời gian | timestamp | |
| khoảng bất định | uncertainty interval | ơn-**SER**-tơn-ti · **IN**-tơ-vơl |
| đồng bộ thời gian | synchronise time | **SIN**-crơ-naiz — "ch" đọc /k/ |
| trung tâm dữ liệu | data centre | Anh viết *centre*, đọc "**SEN**-tơ" |
| vệ tinh | satellite | **SA**-tơ-lait — trọng âm 1 |
| phạm vi toàn cầu | global scale | |
| nhất quán mạnh | strong consistency | cơn-**SIS**-tơn-si |
| độ trễ | latency | /ˈleɪtənsi/ — "**LAY**-tơn-si" |
| vùng địa lý | geographic region | ji-ơ-**GRA**-fic · **REE**-jơn |
| chính sách đặt dữ liệu | placement policy | **PLEIS**-mơnt |
| chi phí vận hành | operational cost | o-pơ-**RAY**-shơn-ơl |
| cơ sở dữ liệu được quản lý | managed database | |
| cụm máy chủ | cluster | **CLUS**-tơ |
| tiêu chí (số ít / số nhiều) | criterion / criteria | crai-**TIA**-ri-ơn / crai-**TIA**-ri-ơ |
| nơi lưu trữ dữ liệu theo luật | data residency | **RE**-zi-đơn-si |
| quy định pháp lý | regulation | re-giu-**LAY**-shơn |
| bản sao đọc | read replica | |
| không máy chủ | serverless | |
| bác bỏ | reject | động từ ri-**JECT**; danh từ **REE**-ject |
| đề xuất | propose | prơ-**POUZ** — đuôi /z/ |
| trường hợp biên | edge case | *edge* /edʒ/ — đuôi /dʒ/ |
| bộ kiểm thử | test suite | *suite* /swiːt/ — đọc y hệt "sweet" |
| bảng hệ thống | system table | |
| đưa ra kết luận | draw a conclusion | cơn-**CLU**-jơn |
| chạy theo trào lưu | chase the hype | *hype* /haɪp/ — "haip" |

---

## Bài nói

> **Cách luyện:** bật ghi âm, nói **60–90 giây** cho mỗi đề, **không nhìn tài liệu**. Mỗi bài nói phải dùng **ít nhất 6 thuật ngữ** trong bảng trên. Nghe lại bản ghi một lần, đánh dấu chỗ ngập ngừng, rồi nói lại đề đó lần thứ hai.

1. Explain to a junior engineer what problem distributed SQL was created to solve, and what people had to give up before it existed.

2. A colleague proposes moving your single-region product to CockroachDB "so we are ready to scale". Explain the questions you would ask before agreeing, and what you would probably recommend instead.

3. Describe how a write is committed in a system that uses Raft, and explain why the system survives when one node dies.

4. Explain what TrueTime does in Spanner and why a commit sometimes has to wait.

5. Your company must keep European customer data inside Europe while still running one product with strong consistency. Explain what you would propose and why.

6. When would you accept the extra latency of a distributed SQL database, and when would that latency be unacceptable?

7. A teammate says migration will be easy because the database is "PostgreSQL compatible". Explain what that phrase really covers and what you would test before committing to the move.
