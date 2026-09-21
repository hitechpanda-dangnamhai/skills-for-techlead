# Bài 4 — Distributed Locks & Fencing Token
### Bản song ngữ: tiếng Anh bám cấu trúc tiếng Việt

> **Cách dùng bài này:** đọc đoạn tiếng Việt trước → tự dịch trong đầu, không nhìn xuống → so với đoạn tiếng Anh bên dưới → **đọc to bản tiếng Anh hai lần**: lần một đọc chậm cho đúng từng âm, lần hai đọc trôi như đang nói với người phỏng vấn.

---

## ① Distributed lock là gì và khi nào chúng ta thật sự cần

**Tiếng Việt**

Distributed lock là cơ chế loại trừ lẫn nhau xuyên qua nhiều tiến trình, nhiều instance và nhiều service. Chúng ta cần nó khi tài nguyên cần bảo vệ không nằm gọn trong một transaction của một database, ví dụ như một lời gọi API bên ngoài, một cron job chỉ được phép chạy đúng một lần, hoặc một thao tác ghi trải trên nhiều store khác nhau. Ở Bài 2, câu lệnh `SELECT ... FOR UPDATE` đã đủ bởi vì mọi thứ nằm trong cùng một database, cho nên chính database đóng vai trò trọng tài. Nhưng khi phạm vi vượt ra ngoài database đó, chúng ta không còn trọng tài nào nữa, cho nên chúng ta phải dựng một điểm khóa chung ở bên ngoài, thường là Redis, ZooKeeper hoặc etcd. Điều quan trọng cần nhận ra ngay từ đầu là distributed lock khó hơn row lock rất nhiều, bởi vì bây giờ chúng ta phải tự lo về hạn sử dụng, về node chết, và về đồng hồ lệch nhau. Nếu chúng ta bê nguyên tư duy row lock sang môi trường phân tán, chúng ta sẽ viết ra một cái lock trông thì đúng nhưng vỡ dưới tải thật.

**English (bám cấu trúc tiếng Việt)**

A distributed lock is a mechanism of mutual exclusion across many processes, many instances and many services. We need it when the resource that we have to protect does not sit neatly inside one transaction of one database, for example a call out to an external API, a cron job that is allowed to run exactly once, or a write operation that spreads across several different stores. In Lesson 2, the statement `SELECT ... FOR UPDATE` was enough because everything sat inside the same database, therefore the database itself played the role of the referee. But when the scope goes outside that database, we no longer have any referee, therefore we have to set up a shared locking point on the outside, usually Redis, ZooKeeper or etcd. The important thing to realise right from the start is that a distributed lock is far harder than a row lock, because now we have to look after expiry, dead nodes, and clocks that drift apart, all by ourselves. If we carry the row-lock mindset straight over into a distributed environment, we will write a lock that looks correct but breaks under real load.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| loại trừ lẫn nhau xuyên qua nhiều tiến trình | mutual exclusion across many processes |
| không nằm gọn trong một transaction | does not sit neatly inside one transaction |
| một lời gọi API bên ngoài | a call out to an external API |
| trải trên nhiều store khác nhau | spreads across several different stores |
| đóng vai trò trọng tài | played the role of the referee |
| khi phạm vi vượt ra ngoài | when the scope goes outside |
| dựng một điểm khóa chung ở bên ngoài | set up a shared locking point on the outside |
| chúng ta phải tự lo về | we have to look after … all by ourselves |
| đồng hồ lệch nhau | clocks that drift apart |
| bê nguyên tư duy row lock sang | carry the row-lock mindset straight over into |
| trông thì đúng nhưng vỡ dưới tải thật | looks correct but breaks under real load |

**Thuật ngữ cần nhớ**

- khóa phân tán → **a distributed lock**
- loại trừ lẫn nhau → **mutual exclusion**
- xuyên nhiều instance → **across instances**
- hạn sử dụng của khóa → **lock expiry**
- đồng hồ lệch → **clock drift** / **clock skew**

---

## ② Lock Redis tối thiểu và cách nhả lock cho đúng

**Tiếng Việt**

Một distributed lock tối thiểu trên Redis được viết bằng đúng một câu lệnh, đó là `SET key <token> NX PX <ttl>`. Cờ `NX` nghĩa là chỉ set khi key chưa tồn tại, cho nên đây là một phép claim atomic và chỉ đúng một client thắng. Cờ `PX` đặt thời gian sống, cho nên nếu client đang giữ lock chết giữa chừng thì lock vẫn tự nhả và chúng ta không rơi vào deadlock vĩnh viễn. Giá trị mà chúng ta ghi vào không phải là một chuỗi tùy ý, mà là một token sở hữu duy nhất, thường là một UUID sinh ngẫu nhiên. Redis hiện đại đã gộp việc set và việc đặt hạn vào cùng một lệnh atomic, cho nên chúng ta tuyệt đối không tách thành `SETNX` rồi `EXPIRE`, bởi vì nếu tiến trình chết giữa hai lệnh thì lock sẽ không bao giờ hết hạn.

Khi nhả lock, chúng ta không được gọi `DEL` thẳng. Lý do là lock của chúng ta có thể đã hết hạn và một client khác đã chiếm được nó, cho nên một lệnh `DEL` thẳng sẽ xóa nhầm lock của người khác. Chúng ta phải so token sở hữu trước rồi mới xóa, tức là chúng ta chỉ xóa nếu giá trị hiện tại đúng bằng token của mình. Phép so sánh và phép xóa đó phải nằm trong cùng một thao tác atomic, và trên Redis chúng ta làm việc này bằng một script Lua. Nếu chúng ta tách thành `GET` rồi `DEL`, chúng ta lại rơi vào đúng cái TOCTOU của Bài 1, chỉ khác là lần này hậu quả là mất luôn tính loại trừ lẫn nhau.

**English (bám cấu trúc tiếng Việt)**

A minimal distributed lock on Redis is written with exactly one statement, which is `SET key <token> NX PX <ttl>`. The `NX` flag means set only if the key does not exist yet, therefore this is an atomic claim and exactly one client wins. The `PX` flag sets the time to live, therefore if the client that is holding the lock dies halfway through, the lock still releases itself and we do not fall into a permanent deadlock. The value that we write in is not an arbitrary string, but a unique ownership token, usually a randomly generated UUID. Modern Redis has merged the set and the expiry into the same atomic command, therefore we absolutely do not split it into `SETNX` and then `EXPIRE`, because if the process dies between the two commands then the lock will never expire.

When we release the lock, we must not call `DEL` directly. The reason is that our lock may already have expired and another client may already have taken it, therefore a direct `DEL` command will delete somebody else's lock by mistake. We have to compare the ownership token first and only then delete, that is, we delete only if the current value equals our own token exactly. That comparison and that deletion must sit inside the same atomic operation, and on Redis we do this with a Lua script. If we split it into `GET` and then `DEL`, we fall right back into the same TOCTOU of Lesson 1, the only difference being that this time the consequence is losing mutual exclusion altogether.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| chỉ set khi key chưa tồn tại | set only if the key does not exist yet |
| chỉ đúng một client thắng | exactly one client wins |
| chết giữa chừng | dies halfway through |
| lock vẫn tự nhả | the lock still releases itself |
| deadlock vĩnh viễn | a permanent deadlock |
| không phải là một chuỗi tùy ý | not an arbitrary string |
| một token sở hữu duy nhất | a unique ownership token |
| đã gộp … vào cùng một lệnh atomic | has merged … into the same atomic command |
| xóa nhầm lock của người khác | delete somebody else's lock by mistake |
| chỉ xóa nếu giá trị hiện tại đúng bằng token của mình | delete only if the current value equals our own token exactly |
| chỉ khác là lần này hậu quả là | the only difference being that this time the consequence is |

**Thuật ngữ cần nhớ**

- đặt nếu chưa tồn tại → **set if not exists**
- thời gian sống → **time to live (TTL)**
- token sở hữu → **an ownership token**
- nhả khóa → **to release a lock**
- vùng tới hạn → **the critical section**

---

## ③ Chọn TTL là một quyết định đánh đổi

**Tiếng Việt**

Việc chọn giá trị TTL là một quyết định đánh đổi thật sự, chứ không phải là một con số tùy tiện. Nếu TTL quá ngắn, lock có thể hết hạn ngay giữa lúc chúng ta đang ở trong vùng tới hạn, cho nên hai client cùng tin rằng mình đang giữ lock. Nếu TTL quá dài, khi client giữ lock chết thì tài nguyên bị kẹt rất lâu và không ai làm gì được cho đến khi hạn trôi qua. Cách xử lý đúng là chúng ta đo thời gian thực tế của vùng tới hạn thay vì đoán, rồi chúng ta đặt TTL rộng hơn con số đo được một khoảng an toàn. Với những công việc chạy dài, chúng ta thêm một watchdog để gia hạn lease định kỳ trong lúc công việc vẫn đang chạy. Và một nguyên tắc bắt buộc là chúng ta không gọi một API bên ngoài chậm trong lúc đang giữ một lock có TTL ngắn, bởi vì đó là công thức chắc chắn dẫn tới hai holder cùng lúc.

**English (bám cấu trúc tiếng Việt)**

Choosing the TTL value is a real trade-off decision, and not an arbitrary number. If the TTL is too short, the lock may expire right in the middle of the moment when we are inside the critical section, therefore two clients both believe that they are holding the lock. If the TTL is too long, when the client holding the lock dies then the resource is stuck for a very long time and nobody can do anything until the deadline passes. The correct way to handle it is that we measure the real duration of the critical section instead of guessing, and then we set the TTL wider than the measured number by a safety margin. For jobs that run long, we add a watchdog in order to renew the lease periodically while the job is still running. And a compulsory rule is that we do not call a slow external API while we are holding a lock with a short TTL, because that is the sure recipe for two holders at the same time.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| một quyết định đánh đổi thật sự | a real trade-off decision |
| một con số tùy tiện | an arbitrary number |
| hết hạn ngay giữa lúc | expire right in the middle of the moment when |
| tài nguyên bị kẹt rất lâu | the resource is stuck for a very long time |
| cho đến khi hạn trôi qua | until the deadline passes |
| đo thời gian thực tế … thay vì đoán | measure the real duration … instead of guessing |
| rộng hơn con số đo được một khoảng an toàn | wider than the measured number by a safety margin |
| gia hạn lease định kỳ | renew the lease periodically |
| trong lúc công việc vẫn đang chạy | while the job is still running |
| công thức chắc chắn dẫn tới hai holder cùng lúc | the sure recipe for two holders at the same time |

**Thuật ngữ cần nhớ**

- hết hạn → **to expire**
- quyền thuê có hạn → **a lease**
- gia hạn lease → **lease renewal**
- bộ canh gia hạn → **a watchdog**
- khoảng an toàn → **a safety margin**

---

## ④ Process pause và failover — hai kẻ thù mà TTL không xử lý được

**Tiếng Việt**

Kẻ thù nguy hiểm nhất của distributed lock là những khoảng dừng của tiến trình. Một đợt thu gom rác kiểu stop-the-world, một lần bị scheduler của hệ điều hành treo, hoặc một cú hoán đổi bộ nhớ đều có thể làm tiến trình đứng im lâu hơn TTL. Trong lúc đó lock hết hạn, một client khác chiếm được nó, và client đó bắt đầu làm việc. Rồi tiến trình cũ tỉnh dậy, và nó vẫn tưởng rằng nó đang giữ lock, cho nên nó ghi tiếp như chưa có chuyện gì xảy ra. Đây chính là điều mà TTL không thể chặn được, bởi vì một tiến trình vừa tỉnh dậy không có cách nào biết rằng nó đã mất lock.

Một rủi ro khác nằm ở chính kiến trúc master và replica của Redis. Việc nhân bản dữ liệu ở đây là bất đồng bộ, cho nên một lock vừa được ghi trên master có thể chưa kịp sang replica. Nếu master chết đúng lúc đó và replica được đưa lên thay, replica sẽ không hề biết đến cái lock kia, cho nên một client thứ hai sẽ chiếm được lock một cách hoàn toàn hợp lệ. Kết quả là hai client cùng giữ một lock mà không bên nào làm gì sai cả, và đây chính là động cơ đã đẻ ra thuật toán Redlock.

**English (bám cấu trúc tiếng Việt)**

The most dangerous enemy of a distributed lock is the pauses of the process. A stop-the-world garbage collection, one moment of being suspended by the scheduler of the operating system, or a memory swap can all make the process stand still for longer than the TTL. During that time the lock expires, another client takes it, and that client starts doing the work. Then the old process wakes up, and it still thinks that it is holding the lock, therefore it carries on writing as if nothing had happened. This is exactly what the TTL cannot block, because a process that has just woken up has no way of knowing that it has lost the lock.

Another risk lies in the master and replica architecture of Redis itself. The replication of data here is asynchronous, therefore a lock that has just been written on the master may not have reached the replica yet. If the master dies at exactly that moment and the replica is promoted in its place, the replica will know nothing about that lock, therefore a second client will take the lock completely legitimately. The result is that two clients hold one lock while neither side has done anything wrong, and this is exactly the motivation that gave birth to the Redlock algorithm.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| những khoảng dừng của tiến trình | the pauses of the process |
| một đợt thu gom rác kiểu stop-the-world | a stop-the-world garbage collection |
| bị scheduler của hệ điều hành treo | being suspended by the scheduler of the operating system |
| làm tiến trình đứng im lâu hơn TTL | make the process stand still for longer than the TTL |
| nó ghi tiếp như chưa có chuyện gì xảy ra | it carries on writing as if nothing had happened |
| không có cách nào biết rằng nó đã mất lock | has no way of knowing that it has lost the lock |
| việc nhân bản dữ liệu ở đây là bất đồng bộ | the replication of data here is asynchronous |
| chưa kịp sang replica | may not have reached the replica yet |
| replica được đưa lên thay | the replica is promoted in its place |
| một cách hoàn toàn hợp lệ | completely legitimately |
| động cơ đã đẻ ra thuật toán Redlock | the motivation that gave birth to the Redlock algorithm |

**Thuật ngữ cần nhớ**

- khoảng dừng tiến trình → **a process pause**
- thu gom rác → **garbage collection**
- nhân bản bất đồng bộ → **asynchronous replication**
- chuyển đổi dự phòng → **failover**
- được đưa lên thay → **to be promoted**

---

## ⑤ Fencing token — thứ mà TTL không làm được

**Tiếng Việt**

Fencing token là câu trả lời đúng cho vấn đề holder zombie, và đây là phần lõi phân biệt một senior với một tech lead thật sự. Fencing token là một con số đơn điệu tăng được cấp cho mỗi lần acquire lock: lần này là ba mươi ba, lần sau là ba mươi tư, rồi ba mươi lăm. Mỗi khi ghi vào tài nguyên, client phải mang theo token của mình, và tài nguyên đó ghi nhớ token lớn nhất mà nó từng nhìn thấy. Nếu một lệnh ghi đến với token nhỏ hơn token lớn nhất đã thấy, tài nguyên từ chối lệnh ghi đó. Nhờ vậy, một tiến trình zombie tỉnh dậy sau cú pause sẽ bị chặn lại ngay tại tầng lưu trữ, dù bản thân nó vẫn tin rằng nó đang giữ lock. Điều kiện để cơ chế này hoạt động là tài nguyên phải biết kiểm tra token, cho nên fencing không phải là thứ chúng ta bật lên ở phía lock, mà là thứ chúng ta phải thiết kế ở cả hai đầu.

**English (bám cấu trúc tiếng Việt)**

A fencing token is the correct answer to the zombie holder problem, and this is the core part that separates a senior from a real tech lead. A fencing token is a monotonically increasing number that is issued for every lock acquisition: this time it is thirty-three, next time it is thirty-four, then thirty-five. Every time it writes to the resource, the client has to carry its own token, and that resource remembers the largest token that it has ever seen. If a write arrives with a token smaller than the largest token already seen, the resource rejects that write. Thanks to that, a zombie process that wakes up after the pause will be blocked right at the storage layer, although it itself still believes that it is holding the lock. The condition for this mechanism to work is that the resource must know how to check the token, therefore fencing is not something that we switch on at the lock side, but something that we have to design at both ends.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| câu trả lời đúng cho vấn đề holder zombie | the correct answer to the zombie holder problem |
| phần lõi phân biệt một senior với một tech lead thật sự | the core part that separates a senior from a real tech lead |
| một con số đơn điệu tăng | a monotonically increasing number |
| được cấp cho mỗi lần acquire lock | that is issued for every lock acquisition |
| phải mang theo token của mình | has to carry its own token |
| token lớn nhất mà nó từng nhìn thấy | the largest token that it has ever seen |
| tài nguyên từ chối lệnh ghi đó | the resource rejects that write |
| bị chặn lại ngay tại tầng lưu trữ | will be blocked right at the storage layer |
| điều kiện để cơ chế này hoạt động | the condition for this mechanism to work |
| chúng ta phải thiết kế ở cả hai đầu | we have to design at both ends |

**Thuật ngữ cần nhớ**

- token chặn ghi cũ → **a fencing token**
- người giữ khóa đã chết lâm sàng → **a zombie holder**
- lần giành khóa → **a lock acquisition**
- từ chối lệnh ghi → **to reject a write**
- tầng lưu trữ → **the storage layer**

---

## ⑥ Redlock, phê phán của Kleppmann, và ranh giới efficiency – correctness

**Tiếng Việt**

Redlock là thuật toán mà tác giả của Redis đề xuất để lock an toàn hơn trên nhiều node. Ý tưởng là chúng ta acquire lock trên N node Redis độc lập, tức là các node không nhân bản cho nhau, và chúng ta chỉ coi là thành công khi giành được đa số trong khoảng thời gian nhỏ hơn TTL. Martin Kleppmann phê phán thuật toán này ở hai điểm. Điểm thứ nhất là Redlock không tự sinh ra fencing token, cho nên nó không giải được bài toán holder zombie. Điểm thứ hai là nó dựa trên các giả định về thời gian, ví dụ đồng hồ không lệch và không có pause dài, mà những giả định đó không đúng trong thực tế. Tài liệu của Redis hiện nay cũng khuyên người đọc xem kỹ phần phản biện và khuyên chúng ta hiện thực fencing token.

Từ cuộc tranh luận đó, cả hai phía đi đến một kết luận thực dụng mà chúng ta nên thuộc. Nếu lock chỉ dùng cho hiệu quả, nghĩa là chạy trùng thì tốn tài nguyên chứ không hỏng gì, thì một Redis đơn node với `SET NX` là đủ. Nếu lock dùng cho tính đúng đắn, nghĩa là sai một lần là mất tiền hoặc hỏng dữ liệu, thì chúng ta phải dùng một hệ consensus như ZooKeeper hoặc etcd, và chúng ta bắt buộc phải có fencing token. Các hệ đó dựa trên consensus và cung cấp sẵn thứ tự đơn điệu cùng với ephemeral node hoặc lease, cho nên lock tự nhả khi session chết và token thứ tự gần như đã có sẵn. Đổi lại, chúng nặng hơn và chậm hơn Redis rất nhiều, cho nên chúng ta không đặt chúng vào đường đi nóng của dữ liệu.

**English (bám cấu trúc tiếng Việt)**

Redlock is the algorithm that the author of Redis proposed in order to lock more safely across many nodes. The idea is that we acquire the lock on N independent Redis nodes, that is, nodes that do not replicate to each other, and we only count it as a success when we win the majority within a period shorter than the TTL. Martin Kleppmann criticises this algorithm on two points. The first point is that Redlock does not generate a fencing token by itself, therefore it does not solve the zombie holder problem. The second point is that it rests on assumptions about time, for example that clocks do not drift and that there are no long pauses, and those assumptions are not true in practice. The documentation of Redis nowadays also advises the reader to look carefully at the criticism and advises us to implement a fencing token.

Out of that debate, both sides arrive at a pragmatic conclusion that we ought to learn by heart. If the lock is used only for efficiency, meaning that running twice wastes resources but breaks nothing, then a single-node Redis with `SET NX` is enough. If the lock is used for correctness, meaning that being wrong once loses money or corrupts data, then we have to use a consensus system such as ZooKeeper or etcd, and we are obliged to have a fencing token. Those systems rest on consensus and provide a monotonic ordering together with ephemeral nodes or leases, therefore the lock releases itself when the session dies and the ordering token is almost already there. In exchange, they are far heavier and far slower than Redis, therefore we do not put them on the hot path of the data.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| tác giả của Redis đề xuất | the author of Redis proposed |
| các node không nhân bản cho nhau | nodes that do not replicate to each other |
| chỉ coi là thành công khi giành được đa số | only count it as a success when we win the majority |
| phê phán thuật toán này ở hai điểm | criticises this algorithm on two points |
| nó dựa trên các giả định về thời gian | it rests on assumptions about time |
| những giả định đó không đúng trong thực tế | those assumptions are not true in practice |
| khuyên người đọc xem kỹ phần phản biện | advises the reader to look carefully at the criticism |
| một kết luận thực dụng mà chúng ta nên thuộc | a pragmatic conclusion that we ought to learn by heart |
| chạy trùng thì tốn tài nguyên chứ không hỏng gì | running twice wastes resources but breaks nothing |
| sai một lần là mất tiền hoặc hỏng dữ liệu | being wrong once loses money or corrupts data |
| chúng ta không đặt chúng vào đường đi nóng của dữ liệu | we do not put them on the hot path of the data |

**Thuật ngữ cần nhớ**

- đa số → **the majority**
- giả định về thời gian → **timing assumptions**
- khóa vì hiệu quả → **a lock for efficiency**
- khóa vì tính đúng đắn → **a lock for correctness**
- node tạm, mất khi mất session → **an ephemeral node**
- đường đi nóng → **the hot path**

---

## ⑦ Khi nào nên tránh distributed lock, và bài toán cron chạy một lần

**Tiếng Việt**

Lời khuyên cuối cùng, và cũng là lời khuyên đáng giá nhất trong phỏng vấn, là chúng ta nên tránh distributed lock nếu còn cách khác. Nếu bài toán giải được bằng một thao tác atomic trong database, bằng một unique constraint, bằng idempotency, hoặc bằng cách chia partition theo key, thì chúng ta chọn những cách đó trước. Lý do là distributed lock có quá nhiều thứ để làm sai, ví dụ TTL đặt nhầm, failover làm mất lock, thiếu fencing, và đồng hồ lệch nhau. Khi một tech lead nhìn thấy distributed lock trong một bản thiết kế, phản xạ đúng là hỏi tại sao chúng ta không giải được bằng một trong các cách rẻ hơn. Chúng ta chỉ dùng nó khi thật sự cần loại trừ lẫn nhau trên nhiều tài nguyên và không còn cách nào khác.

Một tình huống rất hay gặp là một cron job phải chạy đúng một lần dù ứng dụng có nhiều instance. Nhiều đội giải bài này bằng cách giả định rằng họ chỉ deploy một instance, và giả định đó sẽ vỡ ngay ngày đầu tiên họ scale lên. Cách làm đúng là chúng ta dùng leader election, hoặc một distributed lock với TTL ngắn kèm gia hạn, hoặc một advisory lock của database. Một cách khác rất gọn là chúng ta tạo một hàng duy nhất theo khung giờ chạy, và chúng ta để unique constraint quyết định instance nào được chạy. Cách cuối cùng này thường là tốt nhất, bởi vì nó biến một bài toán khóa phân tán thành một bài toán ràng buộc trong database mà chúng ta đã biết cách xử lý.

**English (bám cấu trúc tiếng Việt)**

The final piece of advice, and also the most valuable piece of advice in an interview, is that we should avoid a distributed lock if there is another way. If the problem can be solved with an atomic operation in the database, with a unique constraint, with idempotency, or by partitioning by key, then we choose those ways first. The reason is that a distributed lock has too many things to get wrong, for example a badly set TTL, a failover that loses the lock, missing fencing, and clocks that drift apart. When a tech lead sees a distributed lock in a design document, the correct reflex is to ask why we cannot solve it with one of the cheaper ways. We only use it when we really need mutual exclusion across several resources and there is no other way left.

A situation that comes up very often is a cron job that has to run exactly once even though the application has many instances. Many teams solve this problem by assuming that they only deploy one instance, and that assumption will break on the very first day they scale up. The correct way is that we use leader election, or a distributed lock with a short TTL plus renewal, or an advisory lock of the database. Another very neat way is that we create a single row per scheduled slot, and we let the unique constraint decide which instance is allowed to run. This last way is usually the best, because it turns a distributed locking problem into a database constraint problem that we already know how to handle.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| lời khuyên đáng giá nhất trong phỏng vấn | the most valuable piece of advice in an interview |
| nếu còn cách khác | if there is another way |
| bằng cách chia partition theo key | by partitioning by key |
| có quá nhiều thứ để làm sai | has too many things to get wrong |
| phản xạ đúng là hỏi tại sao | the correct reflex is to ask why |
| không còn cách nào khác | there is no other way left |
| giả định đó sẽ vỡ ngay ngày đầu tiên họ scale lên | that assumption will break on the very first day they scale up |
| một hàng duy nhất theo khung giờ chạy | a single row per scheduled slot |
| để unique constraint quyết định instance nào được chạy | let the unique constraint decide which instance is allowed to run |
| biến một bài toán khóa phân tán thành | turns a distributed locking problem into |
| mà chúng ta đã biết cách xử lý | that we already know how to handle |

**Thuật ngữ cần nhớ**

- chia phân vùng theo khóa → **to partition by key**
- bầu chọn thủ lĩnh → **leader election**
- khóa tư vấn của database → **a database advisory lock**
- khung giờ đã lên lịch → **a scheduled slot**
- bản thiết kế → **a design document**

---

## Mô hình ghi nhớ

**Tiếng Việt**

Distributed lock là loại trừ lẫn nhau trên phạm vi vượt khỏi một database; TTL chống được deadlock nhưng không chống được holder zombie, và chỉ fencing token với con số đơn điệu tăng mới làm được điều đó. Lock cho hiệu quả thì dùng Redis, lock cho tính đúng đắn thì dùng consensus cộng fencing, còn tốt nhất là chúng ta tránh lock nếu còn một cách atomic khác.

**English (bám cấu trúc tiếng Việt)**

A distributed lock is mutual exclusion over a scope that goes beyond one database; the TTL can stop a deadlock but cannot stop a zombie holder, and only a fencing token with a monotonically increasing number can do that. A lock for efficiency uses Redis, a lock for correctness uses consensus plus fencing, and the best thing is that we avoid a lock if there is another atomic way.

---

## Bảng thuật ngữ tổng hợp

| Tiếng Việt | Tiếng Anh | Ghi chú phát âm (Anh–Anh) |
|---|---|---|
| khóa phân tán | **a distributed lock** | *dis-TRI-byu-tid* — trọng âm âm thứ hai |
| loại trừ lẫn nhau | **mutual exclusion** | *MYUU-tsu-ợl ik-SKLUU-zhợn* — `exclusion` có âm /ʒ/, khác `exclusive` có /s/ |
| trọng tài | **a referee** | *re-fơ-RII* — trọng âm rơi vào âm cuối |
| hạn sử dụng của khóa | **lock expiry** | *ik-SPAI-ơ-ri* — trọng âm âm thứ hai |
| đồng hồ lệch | **clock drift** / **clock skew** | *skew* /skjuː/ — có âm /kj/, không đọc "sờ-kiu-ơ" |
| đặt nếu chưa tồn tại | **set if not exists** | |
| thời gian sống | **time to live (TTL)** | |
| token sở hữu | **an ownership token** | *TOU-kợn* — không đọc "tô-ken" |
| chuỗi tùy ý | **an arbitrary string** | *AA-bi-trơ-ri* /ˈɑːbɪtrəri/ — trọng âm đầu, bốn âm tiết |
| nhả khóa | **to release a lock** | *ri-LIIS* — đuôi /s/, không phải /z/ |
| vùng tới hạn | **the critical section** | *KRI-ti-kợl* — trọng âm đầu |
| hết hạn | **to expire** | *ik-SPAI-ơ* — trọng âm âm thứ hai |
| quyền thuê có hạn | **a lease** | /liːs/ đuôi /s/; đừng lẫn với `least` |
| gia hạn lease | **lease renewal** | *ri-NYUU-ợl* — có âm /nj/ kiểu Anh |
| bộ canh gia hạn | **a watchdog** | *WOTCH-dog* — âm /w/ tròn môi |
| khoảng an toàn | **a safety margin** | *MAA-djin* — `g` đọc /dʒ/ |
| khoảng dừng tiến trình | **a process pause** | *process* Anh = *PROU-ses*; *pause* /pɔːz/ đuôi /z/ |
| thu gom rác | **garbage collection** | *GAA-bidj* /ˈɡɑːbɪdʒ/ — đuôi /dʒ/, không đọc "gar-bết" |
| bộ lập lịch | **the scheduler** | Anh: *SHE-dyu-lơ* /ˈʃedjuːlə/ — bắt đầu bằng /ʃ/, khác Mỹ "SKE-" |
| nhân bản bất đồng bộ | **asynchronous replication** | *ei-SING-krơ-nợs* — trọng âm âm thứ hai |
| chuyển đổi dự phòng | **failover** | |
| được đưa lên thay | **to be promoted** | *prơ-MOU-tid* |
| token chặn ghi cũ | **a fencing token** | |
| người giữ khóa "zombie" | **a zombie holder** | *ZOM-bi* /ˈzɒmbi/ — trọng âm đầu, không đọc "dôm-bi" |
| đơn điệu tăng | **monotonically increasing** | *mo-nơ-TON-ik-li* — trọng âm âm thứ ba |
| lần giành khóa | **a lock acquisition** | *a-kwi-ZI-shợn* — trọng âm áp chót, có âm /kw/ |
| giành khóa | **to acquire** | *ơ-KWAI-ơ* — trọng âm sau, âm /kw/ |
| từ chối lệnh ghi | **to reject a write** | *write* câm chữ **w** |
| tầng lưu trữ | **the storage layer** | *STO-ridj* — đuôi /dʒ/ |
| thuật toán | **an algorithm** | *AL-gơ-ri-đợm* /ˈælɡərɪðəm/ — đuôi có âm /ð/, không phải /t/ |
| đa số | **the majority** | *mơ-DJO-rơ-ti* — trọng âm âm thứ hai |
| số đại biểu tối thiểu | **a quorum** | *KWO-rợm* — âm /kw/ đầu |
| giả định về thời gian | **timing assumptions** | *ơ-SAMP-shợn* — cụm **-mpsh-** khó, nói chậm |
| phê phán, phản biện | **criticism** | *KRI-ti-si-zợm* — trọng âm đầu, đuôi /zəm/ |
| khóa vì hiệu quả | **a lock for efficiency** | *i-FI-shợn-si* — trọng âm âm thứ hai |
| khóa vì tính đúng đắn | **a lock for correctness** | cụm cuối **-ctness** khó: *kơ-REKT-nợs* |
| đồng thuận | **consensus** | *kơn-SEN-sợs* — trọng âm giữa |
| node tạm theo session | **an ephemeral node** | *i-FE-mơ-rợl* /ɪˈfemərəl/ — trọng âm âm thứ hai |
| đường đi nóng | **the hot path** | *path* Anh = /pɑːθ/ — nguyên âm dài, đuôi **th** |
| chia phân vùng theo khóa | **to partition by key** | *paa-TI-shợn* — trọng âm giữa |
| bầu chọn thủ lĩnh | **leader election** | *i-LEK-shợn* — trọng âm giữa |
| khóa tư vấn của database | **a database advisory lock** | *ơd-VAI-zơ-ri* — trọng âm âm thứ hai |
| khung giờ đã lên lịch | **a scheduled slot** | *SHE-dyuuld* — kiểu Anh, đuôi **-led** đọc /ld/ |
| bản thiết kế | **a design document** | |

---

## Bài nói

> **Cách luyện:** bật ghi âm, nói **60–90 giây** cho mỗi đề, **không nhìn tài liệu**. Mỗi bài nói phải dùng ít nhất **sáu thuật ngữ** trong bảng trên. Nói xong nghe lại một lần, đánh dấu chỗ nào bị vấp hoặc phát âm sai, rồi nói lại đề đó một lần nữa.

1. **Explain to a junior developer** when a row lock in the database is enough and when the team genuinely needs a distributed lock. Give one concrete example of each.

2. **Describe what happens**, step by step, when a lock holder is paused by garbage collection for eight seconds while the lock TTL is five seconds. Say why the TTL does not save the system and what does.

3. **A colleague says:** *"We release the lock with `DEL`, and that is fine because we only delete our own key."* Explain what is wrong with that claim and how you would rewrite the release path.

4. **Someone on your team wants to guard a money transfer with a Redis lock** and says Redlock makes it safe. Explain why you would push back, what Kleppmann's two criticisms are, and what you would use instead.

5. **When would you accept a single-node Redis lock**, and when would you insist on a consensus store such as etcd or ZooKeeper? Frame your answer around efficiency versus correctness.

6. **Explain to a product manager**, without code, why the nightly report job sometimes runs twice after the team scaled the service to four instances, and what options exist to fix it.

7. **A design document proposes a distributed lock around every write to the orders table.** Explain why you would challenge that design, and which cheaper mechanisms you would ask the author to consider first.

---

> 🔚 **Hết Bài 4.** Gõ `Làm Bài 5` để sang *CAP & PACELC*.
