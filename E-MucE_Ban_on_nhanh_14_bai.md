# Mục E — Bản ôn nhanh trước phỏng vấn
### Tổng hợp 14 bài · song ngữ · dùng trong 30 phút cuối trước buổi phỏng vấn

> **Cách dùng:** đây không phải tài liệu học lần đầu, mà là tài liệu **gọi lại**. Che cột tiếng Anh, đọc câu tiếng Việt, tự nói ra bản tiếng Anh, rồi mới mở ra so. Phần cuối là mười bốn đề nói — mỗi ngày lấy hai đề, nói không nhìn giấy.

---

## PHẦN 1 — Mười bốn mô hình ghi nhớ

### Bài 1 — SQL vs NoSQL

**Tiếng Việt.** Chúng ta chọn cơ sở dữ liệu theo access pattern, theo yêu cầu nhất quán và theo mô hình quan hệ, chứ không theo tốc độ. PostgreSQL là lựa chọn mặc định, và mọi lựa chọn lệch khỏi nó phải có một lý do đo được.

**English.** We choose a database by access pattern, by consistency requirement and by the relationship model, and not by speed. PostgreSQL is the default choice, and every choice that moves away from it must have a measurable reason.

### Bài 2 — Chuẩn hoá

**Tiếng Việt.** Chúng ta chuẩn hoá để dữ liệu đúng, và chúng ta denormalize để đọc nhanh. Mỗi lần denormalize là một lời hứa rằng chúng ta sẽ tự giữ cho hai bản dữ liệu khớp nhau.

**English.** We normalise so that the data is correct, and we denormalize so that reads are fast. Every denormalisation is a promise that we will keep the two copies of the data matching each other ourselves.

### Bài 3 — Data modeling

**Tiếng Việt.** Kiểu dữ liệu là một hợp đồng vĩnh viễn, vì vậy chúng ta lưu tiền bằng số nguyên, chúng ta lưu thời gian bằng timestamptz, và chúng ta chọn khoá chính tuần tự hoặc sắp theo thời gian. Nếu chúng ta chọn sai ngay từ đầu, việc sửa về sau sẽ rất đắt.

**English.** The data type is a permanent contract, therefore we store money as an integer, we store time as timestamptz, and we choose a primary key that is sequential or time-ordered. If we choose wrongly from the start, fixing it later will be very expensive.

### Bài 4 — Index & B-tree

**Tiếng Việt.** Index là việc chúng ta đổi tốc độ ghi để lấy tốc độ đọc. Mỗi index phải trả lời được câu hỏi truy vấn nào đang dùng nó, và nếu không có ai dùng thì chúng ta xoá nó đi.

**English.** An index is us trading write speed for read speed. Every index must be able to answer the question of which query is using it, and if nobody is using it then we drop it.

### Bài 5 — Composite index

**Tiếng Việt.** Composite index được đọc từ trái sang phải, vì vậy chúng ta đặt cột so sánh bằng lên trước và đặt cột khoảng hoặc cột sắp xếp ra sau. Sau đó chúng ta bỏ đi mọi index mà tập cột của nó chỉ là tiền tố của một index khác.

**English.** A composite index is read from left to right, therefore we put the equality columns first and put the range column or the sort column last. After that we drop every index whose column set is only a prefix of another index.

### Bài 6 — N+1 query

**Tiếng Việt.** N+1 là hiện tượng chúng ta gọi cơ sở dữ liệu bên trong một vòng lặp, vì vậy chúng ta gom các lời gọi đó lại bằng mệnh đề IN, bằng JOIN hoặc bằng DataLoader. Nhưng gom thành một câu JOIN khổng lồ cũng là một lỗi, vì vậy chúng ta phải đo chứ chúng ta không đoán.

**English.** N+1 is the situation where we call the database inside a loop, therefore we group those calls together with an IN clause, with a JOIN or with DataLoader. But grouping them into one giant JOIN is also a mistake, therefore we have to measure and we do not guess.

### Bài 7 — ACID & Transaction

**Tiếng Việt.** Transaction là một đơn vị hoặc thành công trọn vẹn hoặc không có gì xảy ra, và nó phải ngắn. Chúng ta không nhốt lời gọi mạng bên trong transaction, và với nghiệp vụ trải nhiều service thì chúng ta dùng saga cùng outbox thay vì two-phase commit.

**English.** A transaction is a unit that either succeeds completely or nothing happens at all, and it has to be short. We do not lock a network call inside a transaction, and for a business flow that spreads across many services we use a saga together with an outbox instead of two-phase commit.

### Bài 8 — Isolation & MVCC

**Tiếng Việt.** MVCC cho phép người đọc làm việc trên một ảnh chụp riêng, vì vậy người đọc không chặn người ghi. Với các điểm nóng, chúng ta dùng câu cập nhật nguyên tử hoặc dùng FOR UPDATE cục bộ, và chúng ta không nâng serializable cho toàn hệ thống cho lành.

**English.** MVCC lets a reader work on its own snapshot, therefore a reader does not block a writer. For hot spots, we use an atomic update statement or we use FOR UPDATE locally, and we do not raise serializable for the whole system just to be safe.

### Bài 9 — PostgreSQL specifics

**Tiếng Việt.** MVCC tạo ra rác dưới dạng dead tuple nên VACUUM phải dọn, và mỗi kết nối là một tiến trình nên chúng ta thêm trình gom kết nối chứ chúng ta không nâng max_connections. Khi một truy vấn chậm đột ngột, chúng ta chạy ANALYZE trước, vì thống kê cũ làm cho bộ lập kế hoạch bị mù.

**English.** MVCC creates rubbish in the form of dead tuples so VACUUM has to clean it up, and each connection is a process so we add a connection pooler and we do not raise max_connections. When a query slows down suddenly, we run ANALYZE first, because stale statistics make the planner blind.

### Bài 10 — Query optimization

**Tiếng Việt.** Chúng ta đo trước rồi mới sửa, vì vậy chúng ta dùng pg_stat_statements để tìm thủ phạm theo tổng thời gian, rồi chúng ta dùng EXPLAIN ANALYZE để lấy kế hoạch thật. Khi số dòng ước lượng lệch xa số dòng thật, nghi phạm đầu tiên của chúng ta là thống kê cũ.

**English.** We measure first and only then fix, therefore we use pg_stat_statements to find the culprit by total time, then we use EXPLAIN ANALYZE to get the real plan. When the estimated row count is far off the actual row count, our first suspect is stale statistics.

### Bài 11 — Connection pooling

**Tiếng Việt.** Pool tồn tại để tái dùng kết nối, và một pool nhỏ mà đúng thì tốt hơn một pool lớn. Tổng của kích thước pool nhân với số instance phải lọt dưới max_connections, và khi pool cạn thì chúng ta sửa truy vấn trước chứ chúng ta không tăng pool.

**English.** A pool exists in order to reuse connections, and a small but correct pool is better than a large one. The total of the pool size multiplied by the number of instances must sit under max_connections, and when the pool runs out we fix the queries first and we do not raise the pool.

### Bài 12 — Replication & lag

**Tiếng Việt.** Bản sao giải bài toán đọc và bài toán sẵn sàng cao, nhưng nó không giải bài toán ghi. Vì luôn có độ trễ, chúng ta phải đưa những lần đọc ngay sau khi ghi về máy chính, và sao chép bất đồng bộ là việc chúng ta đổi an toàn để lấy tốc độ.

**English.** Replicas solve the read problem and the high availability problem, but they do not solve the write problem. Because there is always lag, we have to send the reads that come right after a write to the primary, and asynchronous replication is us trading safety for speed.

### Bài 13 — Sharding & Partitioning

**Tiếng Việt.** Partition chia một bảng bên trong một máy, còn shard chia dữ liệu ra nhiều máy, vì vậy shard là cửa cuối cùng sau khi index, bản sao đọc, cache và partition đã hết đường. Một shard key sai sẽ tạo ra shard nóng và những truy vấn xuyên shard rất đắt.

**English.** Partitioning splits one table inside one machine, while sharding splits the data across many machines, therefore sharding is the last door after indexes, read replicas, caching and partitioning have run out. A wrong shard key will create a hot shard and cross-shard queries that are very expensive.

### Bài 14 — NewSQL

**Tiếng Việt.** NewSQL cho chúng ta SQL, bảo đảm ACID và khả năng mở rộng ngang nhờ thuật toán đồng thuận, nhưng nó thu phí bằng độ trễ, bằng công vận hành và bằng tiền. Chúng ta chỉ chọn nó khi thật sự cần nhất quán mạnh trên nhiều vùng, còn nếu chỉ chạy trong một vùng thì PostgreSQL được quản lý là lựa chọn đúng.

**English.** NewSQL gives us SQL, ACID guarantees and horizontal scalability thanks to a consensus algorithm, but it charges us in latency, in operational work and in money. We only choose it when we really need strong consistency across many regions, and if we only run in one region then managed PostgreSQL is the right choice.

---

## PHẦN 2 — Ba mươi cụm ánh xạ dùng lại được ở mọi bài

Đây là những cụm xuất hiện lặp lại xuyên suốt mười bốn bài. Thuộc ba mươi cụm này là đủ để nối ý trong hầu hết câu trả lời phỏng vấn.

| Tiếng Việt | Tiếng Anh |
|---|---|
| nó không tự nhiên mà có được | it does not come for free |
| đổi lại | in exchange |
| phần khó nằm ở lúc | the hard part lies in the moment when |
| nhìn qua thì tiện | at first glance this looks convenient |
| một lý do đo được | a measurable reason |
| chúng ta đo chứ chúng ta không đoán | we measure and we do not guess |
| dựa trên số liệu chứ không dựa trên cảm tính | based on numbers and not based on gut feeling |
| nếu làm sai thì hậu quả là | if we get this wrong, the consequence is |
| chỉ lộ ra khi dữ liệu lớn lên | only shows up when the data grows large |
| nó chỉ xuất hiện khi có tải | it only shows up under load |
| rất khó tái hiện ở môi trường thử nghiệm | very hard to reproduce in a test environment |
| chỉ che triệu chứng chứ không chữa gốc | only hides the symptom and does not cure the root |
| nghi phạm đầu tiên của chúng ta là | our first suspect is |
| chúng ta xử lý riêng từng điểm nóng | we handle each hot spot separately |
| cho chắc | just to be safe |
| đây là một hướng sai | this is the wrong direction |
| lý do nằm ở kiến trúc của | the reason lies in the architecture of |
| vượt quá khả năng của một máy duy nhất | goes beyond the capacity of one single machine |
| thêm bao nhiêu … cũng vô ích | adding any number of … is useless |
| là phương án cuối cùng | is the last option |
| chúng ta đi lần lượt qua các bậc thang rẻ hơn | we go through the cheaper steps one by one |
| khi tất cả những bậc đó đã hết đường | when all of those steps have run out |
| chúng ta không chạy theo giải pháp thời thượng | we do not chase the fashionable solution |
| một khoản thuế vĩnh viễn | a permanent tax |
| chúng ta phải chấp nhận ba cái giá | we have to accept three costs |
| mỗi lần … là một lời hứa | every time we …, we make a promise |
| tuỳ theo yêu cầu của từng trường hợp | it depends on the requirement of each case |
| chúng ta phải nói thêm rằng | we must add that |
| đó là dấu hiệu cho thấy | that is a sign that |
| công cụ đang được dùng sai bài toán | the tool is being used on the wrong problem |

---

## PHẦN 3 — Bảng phát âm: bốn mươi từ dễ sai nhất

Chỉ liệt kê những từ mà người Việt hay đọc sai. Đọc to từng từ hai lần trước khi vào phỏng vấn.

| Từ | Ghi chú phát âm (giọng Anh) |
|---|---|
| cache | /kæʃ/ — đọc y hệt "cash"; **không** đọc "ca-chê" |
| queue | /kjuː/ — đọc y hệt chữ cái "Q" |
| schema | /ˈskiːmə/ — "**SKII**-mờ", trọng âm âm đầu |
| tuple | Anh /ˈtjuːpl/ — "**TIU**-pl", không đọc "tớp-lê" |
| parse | /pɑːz/ — đuôi /z/, không đọc "pat" |
| suite | /swiːt/ — đọc y hệt "sweet" |
| route | Anh /ruːt/ — "rut"; người Mỹ đọc /raʊt/ |
| heap | /hiːp/ — âm **h** phải bật |
| bloat | /bləʊt/ — vần với "boat" |
| stale | /steɪl/ — "stêi-l" |
| clause | /klɔːz/ — "clô-z", đuôi /z/ |
| leaf / leaves | /liːf/ · /liːvz/ — số nhiều đổi **f** thành **v** |
| debt | /det/ — chữ **b** câm |
| column | chữ **n** cuối câm: "**CO**-lơm" |
| foreign | chữ **g** câm: "**FO**-rin" |
| write / rewrite | chữ **w** câm |
| wrap | chữ **w** câm: "ræp" |
| acknowledge | ơk-**NO**-lij — chữ **w** câm |
| acquire | ơ-**KWAI**-ơ — chữ **c** câm |
| weigh | /weɪ/ — **gh** câm |
| hours | chữ **h** câm: "au-ơz" |
| thumb | /θʌm/ — chữ **b** câm, "th" đầu lưỡi |
| symptom | **SIM**-tơm — chữ **p** gần như câm |
| throughput | /ˈθruːpʊt/ — "th" đầu lưỡi, không thành "trút" |
| threshold | **THRESH**-hould — "th" đầu lưỡi |
| atomicity | a-tơ-**MI**-sơ-ti — trọng âm 3 |
| anomaly | ơ-**NO**-mơ-li — trọng âm 2 |
| isolation | ai-sơ-**LAY**-shơn — âm đầu là "ai" |
| serializable | si-ri-ơ-**LAI**-zơ-bl |
| cardinality | car-đi-**NA**-lơ-ti — trọng âm 3 |
| selectivity | se-lek-**TI**-vơ-ti |
| latency | **LAY**-tơn-si, không đọc "la-ten-si" |
| migration | my-**GRAY**-shơn — âm đầu là "my" |
| integer | **IN**-tơ-jơ — chữ **g** đọc /dʒ/ |
| unique | yu-**NEEK** — trọng âm 2 |
| archive | **AR**-kaiv — không đọc "ạc-chiv" |
| mechanism | **ME**-cơ-ni-zơm — "ch" đọc /k/ |
| architecture | **AR**-ki-tek-chơ — "ch" trong *arch* đọc /k/ |
| algorithm | **AL**-gơ-ri-đơm — trọng âm 1 |
| idempotent | i-**DEM**-pơ-tơnt — trọng âm 2 |
| estimate | động từ **ES**-ti-meit; danh từ **ES**-ti-mơt |
| suspect | danh từ **SUS**-pect; động từ sơs-**PECT** |
| beta | Anh **BEE**-tơ, không đọc "bê-ta" |

---

## PHẦN 4 — Mười bốn đề nói, mỗi bài một đề

Đây là mười bốn đề chọn lọc, ưu tiên dạng phản biện và bảo vệ quan điểm — dạng phân biệt senior với mid-level. Nói **60–90 giây** mỗi đề, **không nhìn giấy**, bật ghi âm.

1. **(Bài 1)** A colleague says: "MongoDB is schema-less, so we do not need to design a schema for this service." Explain what is wrong with that.

2. **(Bài 2)** A reviewer says: "This table is not in third normal form, so the design is wrong." Explain when that criticism is a real problem and when it is just dogma.

3. **(Bài 3)** A colleague says: "We should store prices as a float, it is simpler and the rounding is close enough." Explain what is wrong with that and what you would use instead.

4. **(Bài 4)** A colleague says: "The query is slow, so let us just add an index on every column in the WHERE clause." Explain why you would push back and what you would do first.

5. **(Bài 5)** Someone has created an index on `(created_at, status)` for the query `WHERE status = ? AND created_at > ?`. Explain why you would push back and what order you would use instead.

6. **(Bài 6)** A teammate insists that one big JOIN is always faster than several small queries. Explain when that is not true and how you would settle the argument.

7. **(Bài 7)** Someone opens a transaction, calls the payment gateway inside it, and commits afterwards. Explain why you would push back and what you would propose instead.

8. **(Bài 8)** A colleague says: "Let us set the whole database to serializable, then we never have to think about concurrency again." Explain why you would push back and what you would do instead.

9. **(Bài 9)** A colleague says: "We keep hitting connection limits, so let us set max_connections to one thousand." Explain why you would push back and what you would propose instead.

10. **(Bài 10)** A query that was fast last month is now slow, and nobody has deployed anything. Talk your team through your list of suspects and how you would rule each one out.

11. **(Bài 11)** Your service autoscales from two to forty instances and each instance has a pool of thirty, while the database allows three hundred connections. Explain the problem and describe two ways to fix it.

12. **(Bài 12)** You have added replicas, the latency has not improved, and some pages show wrong data. Talk an interviewer through your diagnosis, step by step.

13. **(Bài 13)** Someone says the orders table is getting large, so the team should start sharding this quarter. Walk them through the cheaper steps you would take first and what evidence would change your mind.

14. **(Bài 14)** A colleague proposes moving your single-region product to CockroachDB "so we are ready to scale". Explain the questions you would ask before agreeing, and what you would probably recommend instead.

---

## PHẦN 5 — Lịch bảy ngày trước phỏng vấn

| Ngày | Việc làm |
|---|---|
| **Ngày 1** | Đọc lại mười bốn mô hình ghi nhớ ở Phần 1, che tiếng Anh và tự nói. Nói đề 1 và đề 2. |
| **Ngày 2** | Học thuộc ba mươi cụm ở Phần 2. Nói đề 3 và đề 4. |
| **Ngày 3** | Đọc to bốn mươi từ ở Phần 3, ghi âm và nghe lại. Nói đề 5 và đề 6. |
| **Ngày 4** | Nói đề 7, đề 8 và đề 9. Sau mỗi đề, tự đánh dấu chỗ ngập ngừng rồi nói lại lần hai. |
| **Ngày 5** | Nói đề 10, đề 11 và đề 12. Trộn thêm hai đề của ngày 1. |
| **Ngày 6** | Nói đề 13 và đề 14. Đọc lại toàn bộ Phần 1 một lượt. |
| **Ngày 7** | Chọn ngẫu nhiên năm đề bất kỳ và nói liền một mạch, không chuẩn bị. |

> **Quy tắc vàng:** gọi lại trước, mở đáp án sau. Đọc lại gần như vô dụng — chỉ có nói ra thành tiếng mới khắc được.

> **Lưu ý kiểm chứng:** những chỗ nhắc tới phiên bản PostgreSQL, lịch hết vòng đời và danh sách sản phẩm distributed SQL đều bám mốc mà giáo trình chốt. Trước buổi phỏng vấn, hãy kiểm tra lại trên trang chính thức, và trong buổi nói hãy dùng chính câu tiếng Anh đã học: *"as of the middle of this year"* và *"we should check the official documentation again for the exact version"*.
