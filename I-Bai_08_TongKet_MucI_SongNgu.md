# Tổng kết Mục I — Concurrency & Consistency
### Bản song ngữ: tiếng Anh bám cấu trúc tiếng Việt

> **Cách dùng bài này:** đây không phải bài học mới, mà là bài để **nói lại toàn bộ mục I**. Đọc đoạn tiếng Việt trước → tự dịch trong đầu → so với đoạn tiếng Anh → **đọc to bản tiếng Anh hai lần**. Phần bài nói ở cuối là dạng câu mà người phỏng vấn hay dùng để **mở đầu buổi**, cho nên đáng luyện kỹ hơn các bài trước.

---

## ① Sợi chỉ đỏ nối bảy bài

**Tiếng Việt**

Bảy bài của mục này không phải là bảy chủ đề rời rạc, mà chúng là bảy lát cắt của cùng một vấn đề. Vấn đề gốc là race condition, tức là khoảng trống giữa lúc đọc và lúc ghi. Khi vấn đề đó nằm gọn trong một database, chúng ta chống nó bằng lock và bằng idempotency. Khi nó vượt ra khỏi một database, chúng ta cần distributed lock, và muốn cái lock đó thật sự đúng thì chúng ta cần fencing token. Fencing token lại đòi hỏi một chuỗi số vừa đơn điệu tăng vừa bền vững, mà chỉ consensus mới sinh ra được chuỗi đó. Trong khi đó, ở tầng có nhiều bản sao dữ liệu, mọi đánh đổi về nhất quán được đóng khung bởi CAP và PACELC, rồi được chi tiết hóa bởi các consistency model.

**English (bám cấu trúc tiếng Việt)**

The seven lessons of this section are not seven separate topics, but they are seven slices of one and the same problem. The root problem is the race condition, that is, the gap between the moment of reading and the moment of writing. When that problem sits neatly inside one database, we fight it with locks and with idempotency. When it goes outside one database, we need a distributed lock, and if we want that lock to be genuinely correct then we need a fencing token. The fencing token in turn demands a sequence of numbers that both increases monotonically and is durable, and only consensus can produce that sequence. Meanwhile, at the layer where there are many copies of the data, every consistency trade-off is framed by CAP and PACELC, and is then made detailed by the consistency models.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| không phải là bảy chủ đề rời rạc | not seven separate topics |
| bảy lát cắt của cùng một vấn đề | seven slices of one and the same problem |
| vấn đề gốc là | the root problem is |
| chúng ta chống nó bằng | we fight it with |
| muốn cái lock đó thật sự đúng thì | if we want that lock to be genuinely correct then |
| lại đòi hỏi | in turn demands |
| vừa đơn điệu tăng vừa bền vững | both increases monotonically and is durable |
| chỉ consensus mới sinh ra được chuỗi đó | only consensus can produce that sequence |
| mọi đánh đổi về nhất quán được đóng khung bởi | every consistency trade-off is framed by |
| rồi được chi tiết hóa bởi | and is then made detailed by |

**Thuật ngữ cần nhớ**

- vấn đề gốc → **the root problem**
- đóng khung một đánh đổi → **to frame a trade-off**
- chuỗi số → **a sequence of numbers**
- bền vững → **durable**
- lát cắt → **a slice**

---

## ② Bốn bài đầu, mỗi bài một câu

**Tiếng Việt**

Nếu chúng ta phải tóm mỗi bài trong đúng một câu, bốn bài đầu sẽ như sau. Bài một nói rằng khoảng trống đọc-ghi sinh ra race, cho nên chúng ta bịt nó bằng một thao tác atomic chứ chúng ta không vá nó bằng timing. Bài hai nói rằng pessimistic là khóa trước rồi mới làm, còn optimistic là cứ làm rồi kiểm tra lúc commit, và chúng ta chọn theo tỉ lệ conflict đo được. Bài ba nói rằng mạng chắc chắn sẽ giao trùng nên consumer phải lặp-không-đổi, nhưng lặp song song thì vẫn cần thêm một phép claim atomic. Bài bốn nói rằng distributed lock là loại trừ lẫn nhau vượt khỏi một database, trong đó TTL chống deadlock còn fencing token chống holder zombie.

**English (bám cấu trúc tiếng Việt)**

If we have to sum up each lesson in exactly one sentence, the first four lessons go as follows. Lesson one says that the read-write gap gives birth to the race, therefore we close it with an atomic operation and we do not patch it with timing. Lesson two says that pessimistic means locking first and doing the work afterwards, while optimistic means just doing the work and checking at commit time, and we choose according to the conflict rate that we have measured. Lesson three says that the network will certainly deliver duplicates so the consumer must be idempotent, but repeating in parallel still needs an atomic claim as well. Lesson four says that a distributed lock is mutual exclusion beyond one database, in which the TTL stops the deadlock while the fencing token stops the zombie holder.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| tóm mỗi bài trong đúng một câu | sum up each lesson in exactly one sentence |
| bốn bài đầu sẽ như sau | the first four lessons go as follows |
| sinh ra race | gives birth to the race |
| chúng ta không vá nó bằng timing | we do not patch it with timing |
| khóa trước rồi mới làm | locking first and doing the work afterwards |
| cứ làm rồi kiểm tra lúc commit | just doing the work and checking at commit time |
| theo tỉ lệ conflict đo được | according to the conflict rate that we have measured |
| lặp-không-đổi | idempotent |
| vượt khỏi một database | beyond one database |
| trong đó TTL chống deadlock | in which the TTL stops the deadlock |

**Thuật ngữ cần nhớ**

- khoảng trống đọc-ghi → **the read-write gap**
- tỉ lệ xung đột → **the conflict rate**
- giành quyền atomic → **an atomic claim**
- loại trừ lẫn nhau → **mutual exclusion**
- người giữ khóa zombie → **a zombie holder**

---

## ③ Ba bài cuối, và tất cả quy về hai ý

**Tiếng Việt**

Ba bài còn lại cũng có thể tóm gọn theo đúng cách đó. Bài năm nói rằng khi mạng chia cắt thì chúng ta chọn đúng hay chọn sống, còn khi mạng lành thì chúng ta vẫn phải chọn nhanh hay chọn đúng, và lựa chọn này là theo từng thao tác. Bài sáu nói rằng nhất quán là một phổ từ mạnh xuống yếu, cho nên chúng ta chọn mức yếu nhất mà nghiệp vụ vẫn đúng. Bài bảy nói rằng đa số cấm được split-brain, rằng Raft là chuẩn thực tế, và rằng consensus đắt nên nó chỉ thuộc về tầng điều khiển. Nhìn từ trên xuống, cả bảy bài quy về đúng hai ý: một là quorum, tức là ai được quyền quyết định, và hai là atomicity, tức là quyết định đó có bị chen ngang hay không.

**English (bám cấu trúc tiếng Việt)**

The three remaining lessons can also be summed up in exactly that way. Lesson five says that when the network splits we choose to be right or to stay alive, while when the network is healthy we still have to choose to be fast or to be right, and this choice is per operation. Lesson six says that consistency is a spectrum from strong down to weak, therefore we choose the weakest level at which the business is still correct. Lesson seven says that the majority forbids split-brain, that Raft is the de facto standard, and that consensus is expensive so it belongs only to the control plane. Looking at it from above, all seven lessons come down to exactly two ideas: one is the quorum, that is, who has the right to decide, and two is atomicity, that is, whether that decision can be interrupted in the middle.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| tóm gọn theo đúng cách đó | summed up in exactly that way |
| chúng ta chọn đúng hay chọn sống | we choose to be right or to stay alive |
| lựa chọn này là theo từng thao tác | this choice is per operation |
| một phổ từ mạnh xuống yếu | a spectrum from strong down to weak |
| đa số cấm được split-brain | the majority forbids split-brain |
| nó chỉ thuộc về tầng điều khiển | it belongs only to the control plane |
| nhìn từ trên xuống | looking at it from above |
| quy về đúng hai ý | come down to exactly two ideas |
| ai được quyền quyết định | who has the right to decide |
| có bị chen ngang hay không | whether that decision can be interrupted in the middle |

**Thuật ngữ cần nhớ**

- theo từng thao tác → **per operation**
- phổ nhất quán → **the consistency spectrum**
- tầng điều khiển → **the control plane**
- tính nguyên tử → **atomicity**
- bị chen ngang → **to be interrupted**

---

## ④ Hai chữ C, và hai chữ "serial"

**Tiếng Việt**

Bảng dễ nhầm bắt đầu bằng hai chữ C. Chữ C trong CAP là sự đồng ý giữa các bản sao, tức là linearizability, và nó nói về việc mọi replica cùng cho ra một câu trả lời. Chữ C trong ACID là bất biến của một transaction, ví dụ khóa ngoại, ràng buộc kiểm tra, hoặc một quy tắc nghiệp vụ. Hai khái niệm này nằm trên hai trục khác nhau, cho nên một hệ có thể tuân thủ ACID ở từng node mà vẫn chỉ nhất quán cuối cùng trên phạm vi toàn cục.

Cặp thứ hai là linearizability và serializability, và đây là cặp bị lẫn nhiều nhất. Linearizability nói về một đối tượng đơn lẻ và về thứ tự thời gian thực, tức là nó nói về độ mới. Serializability nói về nhiều đối tượng và về thứ tự của các transaction, và nó không bắt buộc phải khớp với thời gian thực. Khi một hệ đạt được cả hai điều kiện, chúng ta gọi đó là strict serializable.

**English (bám cấu trúc tiếng Việt)**

The table of easily confused pairs begins with the two letters C. The letter C in CAP is the agreement between the copies, that is, linearizability, and it talks about every replica giving out the same answer. The letter C in ACID is the invariant of a transaction, for example a foreign key, a check constraint, or a business rule. These two concepts sit on two different axes, therefore a system can comply with ACID on each node while it is still only eventually consistent at the global scope.

The second pair is linearizability and serializability, and this is the pair that gets mixed up the most. Linearizability talks about a single object and about real-time order, that is, it talks about recency. Serializability talks about many objects and about the order of transactions, and it is not required to match real time. When a system achieves both conditions, we call that strict serializable.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| bảng dễ nhầm | the table of easily confused pairs |
| sự đồng ý giữa các bản sao | the agreement between the copies |
| mọi replica cùng cho ra một câu trả lời | every replica giving out the same answer |
| nằm trên hai trục khác nhau | sit on two different axes |
| tuân thủ ACID ở từng node | comply with ACID on each node |
| cặp bị lẫn nhiều nhất | the pair that gets mixed up the most |
| nó nói về độ mới | it talks about recency |
| không bắt buộc phải khớp với thời gian thực | is not required to match real time |
| khi một hệ đạt được cả hai điều kiện | when a system achieves both conditions |

**Thuật ngữ cần nhớ**

- bất biến → **an invariant**
- sự đồng ý giữa các bản sao → **replica agreement**
- độ mới của dữ liệu → **recency**
- tuần tự nghiêm ngặt → **strict serializable**
- trục → **an axis**

---

## ⑤ TTL so với fencing token, và lock cho hiệu quả so với lock cho tính đúng đắn

**Tiếng Việt**

Cặp thứ ba là TTL và fencing token, hai thứ trông giống nhau nhưng chúng giải hai bài toán khác nhau. TTL làm cho lock tự nhả khi người giữ nó chết, cho nên nó chống deadlock vĩnh viễn. Fencing token chặn một tiến trình đã mất lock nhưng vẫn tưởng rằng mình còn giữ, cho nên nó chống việc ghi đè từ holder zombie. Nếu chúng ta chỉ có TTL mà không có fencing, một cú pause dài hơn TTL sẽ phá vỡ tính loại trừ lẫn nhau.

Cặp thứ tư là lock cho hiệu quả và lock cho tính đúng đắn. Lock cho hiệu quả chỉ tránh làm trùng việc, cho nên chạy trùng một lần thì tốn tài nguyên chứ nó không hỏng gì, và một Redis đơn node là đủ. Lock cho tính đúng đắn thì sai một lần là mất tiền hoặc hỏng dữ liệu, cho nên chúng ta cần một consensus store cộng với fencing token. Câu hỏi mà chúng ta phải tự đặt ra trước mỗi lần dùng lock là hỏng thì mất gì.

**English (bám cấu trúc tiếng Việt)**

The third pair is the TTL and the fencing token, two things that look alike but they solve two different problems. The TTL makes the lock release itself when the one holding it dies, therefore it stops a permanent deadlock. The fencing token blocks a process that has lost the lock but still thinks that it is holding it, therefore it stops the overwrite from a zombie holder. If we only have the TTL and do not have fencing, a pause longer than the TTL will break mutual exclusion.

The fourth pair is a lock for efficiency and a lock for correctness. A lock for efficiency only avoids doing duplicate work, therefore running twice wastes resources but it breaks nothing, and a single-node Redis is enough. A lock for correctness means that being wrong once loses money or corrupts data, therefore we need a consensus store plus a fencing token. The question that we have to put to ourselves before every use of a lock is what we lose when it goes wrong.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| hai thứ trông giống nhau | two things that look alike |
| làm cho lock tự nhả | makes the lock release itself |
| khi người giữ nó chết | when the one holding it dies |
| vẫn tưởng rằng mình còn giữ | still thinks that it is holding it |
| sẽ phá vỡ tính loại trừ lẫn nhau | will break mutual exclusion |
| chỉ tránh làm trùng việc | only avoids doing duplicate work |
| nó không hỏng gì | it breaks nothing |
| sai một lần là mất tiền | being wrong once loses money |
| câu hỏi mà chúng ta phải tự đặt ra | the question that we have to put to ourselves |
| hỏng thì mất gì | what we lose when it goes wrong |

**Thuật ngữ cần nhớ**

- thời gian sống của khóa → **the lock TTL**
- token chặn ghi cũ → **a fencing token**
- khóa vì hiệu quả → **a lock for efficiency**
- khóa vì tính đúng đắn → **a lock for correctness**
- kho đồng thuận → **a consensus store**

---

## ⑥ Ba cặp còn lại

**Tiếng Việt**

Cặp thứ năm là at-least-once và exactly-once. At-least-once là thứ có thật, và chính nó buộc consumer phải idempotent. Exactly-once delivery là một ảo tưởng trên mạng không tin cậy, cho nên thứ chúng ta thật sự đạt được là effectively-once, tức là at-least-once cộng với khử trùng. Nếu ai đó nói rằng hệ của họ giao đúng một lần, chúng ta nên hỏi họ khử trùng ở chỗ nào.

Hai cặp cuối ngắn hơn nhưng chúng vẫn hay bị hỏi. Optimistic là kiểm tra lúc commit rồi thử lại, còn pessimistic là khóa trước rồi chờ, và dưới mức tranh chấp cao thì optimistic có thể tệ hơn. Paxos đúng nhưng khó, còn Raft dễ hiểu và đã thành chuẩn thực tế, cho nên trong phỏng vấn chúng ta nói về Raft chứ chúng ta không diễn lại Paxos. Nhớ được sáu cặp này nghĩa là chúng ta đã nắm phần lớn những chỗ mà người phỏng vấn dùng để phân loại ứng viên.

**English (bám cấu trúc tiếng Việt)**

The fifth pair is at-least-once and exactly-once. At-least-once is the thing that is real, and it is precisely what forces the consumer to be idempotent. Exactly-once delivery is an illusion over an unreliable network, therefore what we really achieve is effectively-once, that is, at-least-once plus deduplication. If somebody says that their system delivers exactly once, we ought to ask them where they deduplicate.

The last two pairs are shorter but they still get asked often. Optimistic means checking at commit time and then retrying, while pessimistic means locking first and then waiting, and under high contention optimistic can be worse. Paxos is correct but hard, while Raft is understandable and has become the de facto standard, therefore in an interview we talk about Raft and we do not act out Paxos. Remembering these six pairs means that we have grasped most of the places that interviewers use in order to sort candidates.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| là thứ có thật | is the thing that is real |
| chính nó buộc consumer phải idempotent | it is precisely what forces the consumer to be idempotent |
| một ảo tưởng trên mạng không tin cậy | an illusion over an unreliable network |
| chúng ta nên hỏi họ khử trùng ở chỗ nào | we ought to ask them where they deduplicate |
| chúng vẫn hay bị hỏi | they still get asked often |
| dưới mức tranh chấp cao | under high contention |
| đã thành chuẩn thực tế | has become the de facto standard |
| chúng ta không diễn lại Paxos | we do not act out Paxos |
| nghĩa là chúng ta đã nắm phần lớn | means that we have grasped most of |
| dùng để phân loại ứng viên | use in order to sort candidates |

**Thuật ngữ cần nhớ**

- giao ít nhất một lần → **at-least-once delivery**
- hiệu quả như một lần → **effectively-once**
- khử trùng lặp → **deduplication**
- mức tranh chấp → **contention**
- chuẩn trên thực tế → **the de facto standard**

---

## ⑦ Cách ôn, và câu thần chú của cả mục

**Tiếng Việt**

Về cách ôn, chúng ta nên dùng lịch ngắt quãng thay vì đọc lại một lượt từ đầu đến cuối. Hôm nay chúng ta ôn bài một và bài hai, ngày mai chúng ta ôn từ bài một đến bài bốn và nhớ lại các bẫy nặng nhất. Sau ba ngày chúng ta ôn toàn bộ cùng với bảng dễ nhầm, và sau một tuần chúng ta tự phỏng vấn mình bằng cách trộn lẫn các câu hỏi. Điều quan trọng là chúng ta phải nói ra miệng chứ chúng ta không chỉ đọc thầm, bởi vì mục tiêu của chúng ta là nói được chứ không phải là nhận ra được.

Câu thần chú xuyên suốt mục này là chúng ta không nhớ để gõ, chúng ta hiểu để chỉ huy, và chúng ta tra cứu phần còn lại. Concurrency chính là nơi mà một trợ lý AI viết ra code đúng cú pháp nhưng sai hành vi nhiều nhất. Lý do là bug chỉ hiện ra dưới tải thật, cho nên đoạn code đó vượt qua được test đơn giản mà nó vẫn sai ở production. Mỗi bài trong mục này đều có ít nhất một điểm mà AI dễ sai, ví dụ khử trùng bằng cách `SELECT` rồi `INSERT`, hoặc dùng distributed lock trên Redis cho việc trừ tiền. Giá trị của chúng ta nằm đúng ở chỗ đó, tức là ở khả năng kiểm tra và chỉ huy chứ nó không nằm ở tốc độ gõ.

**English (bám cấu trúc tiếng Việt)**

As for the way of revising, we should use a spaced schedule instead of reading through once from beginning to end. Today we revise lesson one and lesson two, tomorrow we revise from lesson one to lesson four and recall the heaviest traps. After three days we revise everything together with the table of confusable pairs, and after a week we interview ourselves by mixing the questions up. The important thing is that we have to speak out loud and we do not merely read silently, because our goal is to be able to say it and not to be able to recognise it.

The mantra running through this section is that we do not memorise in order to type, we understand in order to direct, and we look up the rest. Concurrency is precisely the area where an AI assistant writes code with correct syntax but wrong behaviour the most often. The reason is that the bug only shows up under real load, therefore that piece of code passes the simple tests while it is still wrong on production. Every lesson in this section has at least one point where the AI easily goes wrong, for example deduplicating by means of a `SELECT` and then an `INSERT`, or using a distributed lock on Redis for taking money. Our value lies exactly at that spot, that is, in the ability to check and to direct, and it does not lie in typing speed.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| lịch ngắt quãng | a spaced schedule |
| đọc lại một lượt từ đầu đến cuối | reading through once from beginning to end |
| nhớ lại các bẫy nặng nhất | recall the heaviest traps |
| tự phỏng vấn mình bằng cách trộn lẫn các câu hỏi | interview ourselves by mixing the questions up |
| nói ra miệng chứ không chỉ đọc thầm | speak out loud and not merely read silently |
| nói được chứ không phải là nhận ra được | to be able to say it and not to be able to recognise it |
| câu thần chú xuyên suốt mục này | the mantra running through this section |
| đúng cú pháp nhưng sai hành vi | with correct syntax but wrong behaviour |
| bug chỉ hiện ra dưới tải thật | the bug only shows up under real load |
| giá trị của chúng ta nằm đúng ở chỗ đó | our value lies exactly at that spot |
| nó không nằm ở tốc độ gõ | it does not lie in typing speed |

**Thuật ngữ cần nhớ**

- ôn ngắt quãng → **spaced repetition**
- nói ra miệng → **to speak out loud**
- cú pháp → **syntax**
- hành vi → **behaviour**
- dưới tải thật → **under real load**

---

## Mô hình ghi nhớ

**Tiếng Việt**

Cả mục này quy về hai câu hỏi: ai được quyền quyết định, và quyết định đó có bị chen ngang hay không. Câu thứ nhất dẫn tới quorum và consensus, còn câu thứ hai dẫn tới atomicity, tức là atomic write, lock, idempotency và fencing token.

**English (bám cấu trúc tiếng Việt)**

This whole section comes down to two questions: who has the right to decide, and whether that decision can be interrupted in the middle. The first question leads to the quorum and to consensus, while the second question leads to atomicity, that is, to the atomic write, the lock, idempotency and the fencing token.

---

## Bảng thuật ngữ tổng hợp (xuyên suốt bảy bài)

| Tiếng Việt | Tiếng Anh | Ghi chú phát âm (Anh–Anh) |
|---|---|---|
| tình huống tranh chấp | **a race condition** | *kơn-DI-shợn* — đuôi `-tion` = /ʃən/ |
| khoảng trống đọc-ghi | **the read-write gap** | *write* câm chữ **w** |
| tính nguyên tử | **atomicity** | *a-tơ-MI-sơ-ti* — trọng âm âm thứ ba; khác `atomic` = *ơ-TOM-ik* |
| mất bản cập nhật | **lost update** | |
| khóa bi quan | **a pessimistic lock** | *pe-si-MIS-tik* — trọng âm áp chót |
| khóa lạc quan | **an optimistic lock** | *op-ti-MIS-tik* — trọng âm áp chót |
| tỉ lệ xung đột | **the conflict rate** | *conflict* danh từ = *KON-flikt* |
| mức tranh chấp | **contention** | *kơn-TEN-shợn* — trọng âm giữa |
| lặp-không-đổi | **idempotent** | *ai-DEM-pơ-tợnt* — trọng âm âm thứ hai |
| tính lặp-không-đổi | **idempotency** | *ai-đem-PO-tơn-si* — trọng âm dịch sang âm thứ ba |
| khử trùng lặp | **deduplication** | *dii-dyuu-pli-KÊI-shợn* — trọng âm áp chót |
| giao ít nhất một lần | **at-least-once delivery** | *once* /wʌns/ — cụm cuối **-ns** phải bật |
| hiệu quả như một lần | **effectively-once** | |
| ràng buộc duy nhất | **a unique constraint** | *kơn-STRÊINT* — cụm **str** và cụm cuối **-nt** |
| loại trừ lẫn nhau | **mutual exclusion** | *ik-SKLUU-zhợn* — có âm /ʒ/, khác `exclusive` có /s/ |
| khóa phân tán | **a distributed lock** | *dis-TRI-byu-tid* — trọng âm âm thứ hai |
| thời gian sống | **time to live (TTL)** | |
| token chặn ghi cũ | **a fencing token** | *TOU-kợn* — không đọc "tô-ken" |
| người giữ khóa zombie | **a zombie holder** | *ZOM-bi* — trọng âm đầu |
| khoảng dừng tiến trình | **a process pause** | *process* Anh = *PROU-ses*; *pause* đuôi /z/ |
| khóa vì hiệu quả | **a lock for efficiency** | *i-FI-shợn-si* — trọng âm âm thứ hai |
| khóa vì tính đúng đắn | **a lock for correctness** | cụm **-ctness** khó: *kơ-REKT-nợs* |
| chia cắt mạng | **a network partition** | *paa-TI-shợn* — trọng âm giữa |
| khả năng sẵn sàng phục vụ | **availability** | *ơ-vei-lơ-BI-lơ-ti* — trọng âm rơi vào **-BI-** |
| tính tuyến tính hóa | **linearizability** | tách chậm: *linear + ize + ability* |
| tính khả tuần tự | **serializability** | tách chậm: *serial + ize + ability* |
| tuần tự nghiêm ngặt | **strict serializable** | *strict* — cụm **str** đầu và **-ct** cuối |
| bất biến | **an invariant** | *in-VAIR-ri-ợnt* — trọng âm âm thứ hai |
| trục | **an axis** (số nhiều **axes**) | số ít *AK-sis*; số nhiều *AK-siiz* |
| độ mới của dữ liệu | **recency** | *RII-sợn-si* — trọng âm đầu |
| nhất quán cuối cùng | **eventual consistency** | *i-VEN-tsu-ợl* — trọng âm âm thứ hai |
| bảo đảm theo phiên | **a session guarantee** | *guarantee* = *ga-rơn-TII*, trọng âm rơi vào âm **cuối** |
| độ trễ nhân bản | **replication lag** | *re-pli-KÊI-shợn* — trọng âm áp chót |
| số phiếu tối thiểu | **a quorum** | *KWO-rợm* — âm /kw/ đầu |
| sự đồng thuận | **consensus** | *kơn-SEN-sợs* — trọng âm giữa |
| não chia đôi, hai leader | **split-brain** | cụm **spl-** rất khó, tập chậm: *s-pl-it* |
| bầu chọn thủ lĩnh | **leader election** | *i-LEK-shợn* — trọng âm giữa |
| chuẩn trên thực tế | **the de facto standard** | tiếng Latin: *đei FAK-tou* /deɪ ˈfæktəʊ/ |
| tầng điều khiển | **the control plane** | *kơn-TROUL* — trọng âm sau |
| tầng dữ liệu | **the data plane** | *data* Anh = *DÊI-tơ* |
| đường đi nóng | **the hot path** | *path* Anh = /pɑːθ/ — nguyên âm dài, đuôi **th** |
| thông lượng | **throughput** | *THRUU-put* — âm **th** và cụm **thr** |
| độ trễ | **latency** | *LÊI-tợn-si* — trọng âm đầu |
| cú pháp | **syntax** | *SIN-taks* — trọng âm đầu, cụm cuối **-ks** |
| hành vi | **behaviour** | Anh viết **-our**; đọc *bi-HÊI-vi-ơ*, trọng âm âm thứ hai |
| ôn ngắt quãng | **spaced repetition** | *re-pơ-TI-shợn* — trọng âm áp chót |
| phán đoán | **judgement** | *DJADJ-mợnt* — hai âm /dʒ/ trong một từ |

---

## Bài nói

> **Cách luyện:** bật ghi âm, **không nhìn tài liệu**. Đề 1 và đề 7 nói trong **hai phút**, các đề còn lại nói trong **60–90 giây**. Mỗi bài phải dùng ít nhất **tám thuật ngữ** trong bảng trên, vì đây là bài tổng kết. Nói xong nghe lại, đánh dấu chỗ vấp, rồi nói lại một lần nữa.

1. **The interviewer opens with:** *"Tell me what you know about concurrency and consistency in distributed systems."* Give a two-minute answer that walks the whole thread from the race condition through to consensus, without listing topics one by one.

2. **Explain the two meanings of the letter C** — the one in CAP and the one in ACID — to a mid-level engineer who has just used them interchangeably in a design review.

3. **A colleague says:** *"We have TTLs on all our distributed locks, so mutual exclusion is guaranteed."* Explain what is missing, and describe the exact scenario where their claim fails.

4. **Compare linearizability and serializability** as if you were correcting a whiteboard diagram. Say which axis each one lives on, and what strict serializability adds.

5. **Someone on your team says:** *"Our message broker gives us exactly-once delivery, so the consumers can stay simple."* Explain why you would push back, and what you would ask them to add.

6. **When would you choose an optimistic lock over a pessimistic one**, and when does that choice reverse? Explain what you would measure before deciding either way.

7. **A hiring manager asks:** *"With AI writing so much of the code now, what is your value as a senior engineer?"* Answer in two minutes, using concurrency as your concrete example of where generated code is syntactically correct but behaviourally wrong.

---

> 🔚 **Hết phần tổng kết Mục I.** Bảy bài cộng bản tổng kết này là một bộ hoàn chỉnh: đọc để hiểu, nói để giữ, và tra cứu phần còn lại khi cần.
