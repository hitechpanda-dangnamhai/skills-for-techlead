# Bài 7 — Consensus (Raft / Paxos / Quorum / Leader Election)
### Bản song ngữ: tiếng Anh bám cấu trúc tiếng Việt

> **Cách dùng bài này:** đọc đoạn tiếng Việt trước → tự dịch trong đầu, không nhìn xuống → so với đoạn tiếng Anh bên dưới → **đọc to bản tiếng Anh hai lần**: lần một đọc chậm cho đúng từng âm, lần hai đọc trôi như đang nói với người phỏng vấn.

---

## ① Consensus là bài toán gì và nó phục vụ việc gì

**Tiếng Việt**

Consensus là bài toán làm cho nhiều node cùng đồng ý trên một giá trị hoặc trên một thứ tự, ngay cả khi một số node chết hoặc mạng bị trễ. Ở đây chúng ta chỉ nói về lỗi kiểu crash, tức là node dừng hoạt động, chứ chúng ta không nói về node cố tình nói dối. Chúng ta cần consensus cho bốn việc chính. Việc thứ nhất là bầu chọn leader, việc thứ hai là nhân bản log hoặc trạng thái theo đúng một thứ tự, việc thứ ba là dựng một distributed lock thật sự đúng đắn, và việc thứ tư là làm kho cấu hình và điều phối như etcd. Nói cách khác, consensus là viên gạch nền của mọi hệ thuộc nhóm CP mà chúng ta đã học ở Bài 5. Nếu chúng ta không có consensus, chúng ta không có cách nào để cả cụm cùng biết chắc chắn ai đang là người quyết định.

**English (bám cấu trúc tiếng Việt)**

Consensus is the problem of making many nodes agree on one value or on one order, even when some nodes die or the network is delayed. Here we are only talking about crash faults, that is, a node stops working, and we are not talking about a node that deliberately lies. We need consensus for four main jobs. The first job is leader election, the second job is replicating the log or the state in exactly one order, the third job is building a distributed lock that is genuinely correct, and the fourth job is serving as a configuration and coordination store such as etcd. In other words, consensus is the foundation stone of every system in the CP group that we learned about in Lesson 5. If we do not have consensus, we have no way for the whole cluster to know for certain who is currently making the decisions.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| làm cho nhiều node cùng đồng ý trên một giá trị | making many nodes agree on one value |
| ngay cả khi một số node chết | even when some nodes die |
| lỗi kiểu crash | crash faults |
| node cố tình nói dối | a node that deliberately lies |
| theo đúng một thứ tự | in exactly one order |
| thật sự đúng đắn | genuinely correct |
| làm kho cấu hình và điều phối | serving as a configuration and coordination store |
| viên gạch nền của mọi hệ thuộc nhóm CP | the foundation stone of every system in the CP group |
| cả cụm cùng biết chắc chắn | the whole cluster to know for certain |
| ai đang là người quyết định | who is currently making the decisions |

**Thuật ngữ cần nhớ**

- sự đồng thuận → **consensus**
- lỗi do node chết → **a crash fault**
- cụm máy → **a cluster**
- kho điều phối → **a coordination store**
- nhân bản log → **log replication**

---

## ② Paxos và Raft nhìn ở mức thực dụng

**Tiếng Việt**

Paxos và Raft là hai thuật toán consensus nổi tiếng nhất, và chúng ta nên biết cách so sánh chúng ở mức thực dụng. Paxos đã được chứng minh là đúng, nhưng nó nổi tiếng là khó hiểu và khó hiện thực cho đúng. Raft được thiết kế với mục tiêu dễ hiểu, cho nên nó tách bài toán thành ba phần rõ ràng, gồm bầu chọn leader, nhân bản log, và các điều kiện an toàn. Ngày nay Raft là chuẩn trên thực tế, bởi vì etcd, Consul và nhiều hệ khác đều dùng nó. Có một mẹo phỏng vấn đáng nhớ ở đây, đó là chúng ta đừng diễn lại Paxos từng bước, mà chúng ta chỉ cần nói rằng Raft tách bài toán ra cho dễ hiểu và đã trở thành chuẩn thực tế, như vậy là đủ điểm.

**English (bám cấu trúc tiếng Việt)**

Paxos and Raft are the two most famous consensus algorithms, and we ought to know how to compare them at a pragmatic level. Paxos has been proved correct, but it is famous for being hard to understand and hard to implement correctly. Raft was designed with understandability as its goal, therefore it splits the problem into three clear parts, namely leader election, log replication, and the safety conditions. Nowadays Raft is the de facto standard, because etcd, Consul and many other systems all use it. There is an interview tip worth remembering here, which is that we should not act out Paxos step by step, but we only need to say that Raft splits the problem up to make it understandable and has become the de facto standard, and that is enough to score.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| ở mức thực dụng | at a pragmatic level |
| đã được chứng minh là đúng | has been proved correct |
| nó nổi tiếng là khó hiểu | it is famous for being hard to understand |
| với mục tiêu dễ hiểu | with understandability as its goal |
| tách bài toán thành ba phần rõ ràng | splits the problem into three clear parts |
| các điều kiện an toàn | the safety conditions |
| chuẩn trên thực tế | the de facto standard |
| một mẹo phỏng vấn đáng nhớ | an interview tip worth remembering |
| chúng ta đừng diễn lại Paxos từng bước | we should not act out Paxos step by step |
| như vậy là đủ điểm | and that is enough to score |

**Thuật ngữ cần nhớ**

- thuật toán → **an algorithm**
- chuẩn trên thực tế → **the de facto standard**
- tính dễ hiểu → **understandability**
- điều kiện an toàn → **a safety condition**
- hiện thực cho đúng → **to implement correctly**

---

## ③ Quorum, đa số, và lý do chọn số node lẻ

**Tiếng Việt**

Quorum nghĩa là chúng ta chỉ chấp nhận một quyết định khi có đa số node đồng ý, và đa số ở đây bằng N chia hai rồi cộng một. Chỉ phe đa số mới được phép bầu leader hoặc commit một lệnh ghi, còn phe thiểu số thì phải dừng lại. Lý do rất đơn giản, đó là trong cùng một cụm không thể tồn tại hai phe đa số cùng lúc, cho nên không thể có hai leader hợp lệ. Từ đó chúng ta suy ra quy tắc chọn số node lẻ, bởi vì khả năng chịu lỗi bằng phần nguyên của N chia hai. Cụ thể, ba node chịu được một node chết, năm node chịu được hai node chết, và bảy node chịu được ba node chết. Nếu chúng ta dùng bốn node, chúng ta vẫn chỉ chịu được một node chết giống hệt như ba node, cho nên chúng ta tốn thêm một máy mà chúng ta không được thêm gì cả.

**English (bám cấu trúc tiếng Việt)**

A quorum means that we only accept a decision when a majority of the nodes agree, and the majority here equals N divided by two and then plus one. Only the majority side is allowed to elect a leader or to commit a write, while the minority side has to stop. The reason is very simple, which is that inside the same cluster two majority sides cannot exist at the same time, therefore two valid leaders cannot exist. From that we derive the rule of choosing an odd number of nodes, because the fault tolerance equals the integer part of N divided by two. Specifically, three nodes tolerate one dead node, five nodes tolerate two dead nodes, and seven nodes tolerate three dead nodes. If we use four nodes, we still tolerate only one dead node exactly like three nodes, therefore we spend one extra machine and we gain nothing at all.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| chỉ chấp nhận một quyết định khi | only accept a decision when |
| N chia hai rồi cộng một | N divided by two and then plus one |
| chỉ phe đa số mới được phép | only the majority side is allowed to |
| phe thiểu số thì phải dừng lại | the minority side has to stop |
| không thể tồn tại hai phe đa số cùng lúc | two majority sides cannot exist at the same time |
| từ đó chúng ta suy ra quy tắc | from that we derive the rule |
| khả năng chịu lỗi | the fault tolerance |
| phần nguyên của N chia hai | the integer part of N divided by two |
| ba node chịu được một node chết | three nodes tolerate one dead node |
| chúng ta không được thêm gì cả | we gain nothing at all |

**Thuật ngữ cần nhớ**

- số phiếu tối thiểu → **a quorum**
- phe đa số → **the majority side**
- phe thiểu số → **the minority side**
- khả năng chịu lỗi → **fault tolerance**
- số lẻ → **an odd number**

---

## ④ Raft bầu leader như thế nào

**Tiếng Việt**

Cơ chế bầu leader của Raft có thể mô tả bằng trực giác mà không cần đến công thức. Leader liên tục phát tín hiệu nhịp tim cho các follower để chứng tỏ rằng nó vẫn còn sống. Nếu một follower không nghe thấy nhịp tim trong khoảng thời gian chờ bầu cử, nó tự chuyển thành ứng viên, nó tăng số nhiệm kỳ lên một, và nó đi xin phiếu bầu. Ứng viên nào nhận được đa số phiếu sẽ trở thành leader mới, và nó bắt đầu phát nhịp tim của chính mình. Số nhiệm kỳ ở đây đóng vai trò một đồng hồ logic, bởi vì nhiệm kỳ cao hơn nghĩa là thông tin mới hơn. Nhờ con số đó, một node có thể nhận ra ngay rằng nó đang nghe một leader đã cũ, và nó từ chối làm theo.

**English (bám cấu trúc tiếng Việt)**

The leader election mechanism of Raft can be described by intuition without needing any formula. The leader continuously sends a heartbeat signal to the followers in order to prove that it is still alive. If a follower does not hear the heartbeat within the election timeout, it turns itself into a candidate, it raises the term number by one, and it goes and asks for votes. Whichever candidate receives a majority of the votes becomes the new leader, and it starts sending its own heartbeat. The term number here plays the role of a logical clock, because a higher term means newer information. Thanks to that number, a node can recognise straight away that it is listening to a leader that has become stale, and it refuses to follow.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| mô tả bằng trực giác mà không cần đến công thức | described by intuition without needing any formula |
| phát tín hiệu nhịp tim | sends a heartbeat signal |
| để chứng tỏ rằng nó vẫn còn sống | in order to prove that it is still alive |
| trong khoảng thời gian chờ bầu cử | within the election timeout |
| nó tự chuyển thành ứng viên | it turns itself into a candidate |
| nó đi xin phiếu bầu | it goes and asks for votes |
| ứng viên nào nhận được đa số phiếu | whichever candidate receives a majority of the votes |
| đóng vai trò một đồng hồ logic | plays the role of a logical clock |
| nó đang nghe một leader đã cũ | it is listening to a leader that has become stale |
| nó từ chối làm theo | it refuses to follow |

**Thuật ngữ cần nhớ**

- tín hiệu nhịp tim → **a heartbeat**
- thời gian chờ bầu cử → **the election timeout**
- ứng viên → **a candidate**
- nhiệm kỳ → **a term**
- đồng hồ logic → **a logical clock**

---

## ⑤ Split-brain, hành vi khi partition, và việc thoái vị khi mạng lành

**Tiếng Việt**

Split-brain là tình huống hai node cùng tưởng rằng mình là leader, và cả hai cùng nhận những lệnh ghi mâu thuẫn nhau. Đây là loại lỗi tệ nhất trong hệ phân tán, bởi vì dữ liệu hỏng một cách âm thầm chứ hệ không hề báo lỗi. Consensus chặn tình huống này bằng chính quy tắc đa số mà chúng ta vừa nói. Chỉ phe nào giữ được đa số mới được bầu leader và mới được commit, còn phe thiểu số thì đứng im. Vì trong một cụm chỉ có thể có một phe đa số, hệ bảo đảm rằng không bao giờ tồn tại hai leader hợp lệ cùng lúc.

Khi mạng bị chia cắt, hành vi của cụm rất dễ mô tả. Phe có đa số sẽ bầu ra leader mới và tiếp tục phục vụ như bình thường. Phe thiểu số không đạt được quorum, cho nên nó không commit được gì và nó phải chặn request lại hoặc trả về dữ liệu cũ tùy theo cấu hình. Khi mạng lành trở lại, leader cũ ở phe thiểu số sẽ nhìn thấy một số nhiệm kỳ cao hơn từ phe kia, cho nên nó tự động thoái vị và đồng bộ lại log của mình. Đây chính là cơ chế nằm bên dưới câu nói "hệ CP từ chối phục vụ phe thiểu số" mà chúng ta đã gặp ở Bài 5.

**English (bám cấu trúc tiếng Việt)**

Split-brain is the situation where two nodes both think that they are the leader, and both of them accept writes that contradict each other. This is the worst kind of fault in a distributed system, because the data is corrupted silently while the system reports no error at all. Consensus blocks this situation with the very majority rule that we have just described. Only the side that holds the majority may elect a leader and may commit, while the minority side stands still. Because inside one cluster there can only be one majority side, the system guarantees that two valid leaders never exist at the same time.

When the network is partitioned, the behaviour of the cluster is very easy to describe. The side with the majority will elect a new leader and will carry on serving as normal. The minority side does not reach a quorum, therefore it cannot commit anything and it has to block the requests or return stale data depending on the configuration. When the network heals, the old leader on the minority side will see a higher term number from the other side, therefore it steps down automatically and synchronises its log again. This is exactly the mechanism that sits underneath the sentence "a CP system refuses to serve the minority side" that we met in Lesson 5.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| hai node cùng tưởng rằng mình là leader | two nodes both think that they are the leader |
| những lệnh ghi mâu thuẫn nhau | writes that contradict each other |
| dữ liệu hỏng một cách âm thầm | the data is corrupted silently |
| bằng chính quy tắc đa số mà chúng ta vừa nói | with the very majority rule that we have just described |
| phe thiểu số thì đứng im | the minority side stands still |
| tiếp tục phục vụ như bình thường | will carry on serving as normal |
| không đạt được quorum | does not reach a quorum |
| tùy theo cấu hình | depending on the configuration |
| khi mạng lành trở lại | when the network heals |
| nó tự động thoái vị | it steps down automatically |
| cơ chế nằm bên dưới câu nói | the mechanism that sits underneath the sentence |

**Thuật ngữ cần nhớ**

- não chia đôi, hai leader → **split-brain**
- mâu thuẫn nhau → **to contradict each other**
- hỏng dữ liệu → **to corrupt data**
- thoái vị → **to step down**
- mạng lành lại → **the network heals**

---

## ⑥ Consensus là nơi duy nhất sinh được fencing token

**Tiếng Việt**

Bây giờ chúng ta nối bài này ngược về Bài 4. Một fencing token cần hai tính chất, đó là nó phải đơn điệu tăng và nó phải sống sót qua việc node chết. Nếu chúng ta đếm token trên một node duy nhất, con số sẽ mất khi node đó chết, cho nên chúng ta không có khả năng chịu lỗi. Nếu chúng ta để nhiều node tự đếm riêng, các con số sẽ lệch nhau và chuỗi token mất hết ý nghĩa. Chỉ consensus mới cho chúng ta một chuỗi thứ tự vừa nhất quán vừa bền vững, và đó là lý do ZooKeeper cấp sẵn zxid còn etcd cấp sẵn revision.

Trong hạ tầng thật, etcd là ví dụ dễ thấy nhất của consensus đang chạy. Kubernetes lưu toàn bộ trạng thái của cụm trong etcd, và etcd dùng Raft để bảo đảm nhất quán mạnh cùng với cơ chế theo dõi thay đổi. Nhiều hệ khác cũng đi theo hướng này, ví dụ Consul dùng Raft cho điều phối, còn Kafka đã chuyển từ ZooKeeper sang KRaft là Raft nội bộ của chính nó. Điều đáng nhớ là chúng ta gần như không bao giờ tự viết consensus, mà chúng ta chỉ cần dùng đúng API bầu leader và lease của những hệ đã được kiểm chứng.

**English (bám cấu trúc tiếng Việt)**

Now we connect this lesson back to Lesson 4. A fencing token needs two properties, which are that it must increase monotonically and it must survive the death of a node. If we count the token on one single node, the number will be lost when that node dies, therefore we have no fault tolerance. If we let many nodes count separately on their own, the numbers will drift apart and the token sequence loses all its meaning. Only consensus gives us an ordering sequence that is both consistent and durable, and that is the reason why ZooKeeper provides the zxid and etcd provides the revision.

In real infrastructure, etcd is the most visible example of consensus in action. Kubernetes stores the entire state of the cluster in etcd, and etcd uses Raft in order to guarantee strong consistency together with a mechanism for watching changes. Many other systems go in the same direction, for example Consul uses Raft for coordination, while Kafka has moved from ZooKeeper to KRaft, which is its own internal Raft. The thing worth remembering is that we almost never write consensus ourselves, but we only need to use the leader election and lease APIs of systems that have already been proven.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| chúng ta nối bài này ngược về | we connect this lesson back to |
| nó phải sống sót qua việc node chết | it must survive the death of a node |
| chúng ta không có khả năng chịu lỗi | we have no fault tolerance |
| để nhiều node tự đếm riêng | let many nodes count separately on their own |
| chuỗi token mất hết ý nghĩa | the token sequence loses all its meaning |
| vừa nhất quán vừa bền vững | both consistent and durable |
| ví dụ dễ thấy nhất của consensus đang chạy | the most visible example of consensus in action |
| cơ chế theo dõi thay đổi | a mechanism for watching changes |
| đi theo hướng này | go in the same direction |
| những hệ đã được kiểm chứng | systems that have already been proven |

**Thuật ngữ cần nhớ**

- bền vững, không mất khi khởi động lại → **durable**
- chuỗi thứ tự → **an ordering sequence**
- hạ tầng → **infrastructure**
- theo dõi thay đổi → **to watch for changes**
- quyền thuê có hạn → **a lease**

---

## ⑦ Consensus rất đắt — tách tầng điều khiển khỏi tầng dữ liệu

**Tiếng Việt**

Điểm cuối cùng, và cũng là điểm quan trọng nhất về mặt phán đoán, là consensus rất đắt. Mỗi quyết định đều cần một vòng đi tới đa số node, cộng thêm một lần ghi log xuống đĩa, cho nên độ trễ cao và thông lượng bị giới hạn. Vì vậy chúng ta chỉ đặt consensus ở tầng điều khiển, tức là cho metadata, cho việc điều phối, và cho việc bầu chọn leader. Chúng ta không đặt nó vào tầng dữ liệu, tức là chúng ta không cho mọi lệnh ghi của ứng dụng chạy qua nó. Nguyên tắc của một tech lead là tách rõ tầng điều khiển với tầng dữ liệu, rồi đặt consensus đúng chỗ chứ không nhét nó vào đường đi nóng. Nếu ai đó đề nghị đẩy mọi lệnh ghi qua etcd để có nhất quán mạnh, chúng ta phải chặn lại, bởi vì đó là trường hợp đúng khái niệm nhưng sai chỗ đặt.

**English (bám cấu trúc tiếng Việt)**

The final point, and also the most important point in terms of judgement, is that consensus is very expensive. Every decision needs a round trip to the majority of the nodes, plus one write of the log down to disk, therefore the latency is high and the throughput is limited. Therefore we only place consensus at the control plane, that is, for metadata, for coordination, and for leader election. We do not place it at the data plane, that is, we do not let every write of the application run through it. The rule of a tech lead is to separate the control plane from the data plane clearly, and then to place consensus in the right spot rather than squeezing it into the hot path. If somebody proposes pushing every write through etcd in order to get strong consistency, we have to block it, because that is a case of being right about the concept but wrong about where to put it.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| điểm quan trọng nhất về mặt phán đoán | the most important point in terms of judgement |
| một lần ghi log xuống đĩa | one write of the log down to disk |
| thông lượng bị giới hạn | the throughput is limited |
| chúng ta chỉ đặt consensus ở tầng điều khiển | we only place consensus at the control plane |
| chúng ta không cho mọi lệnh ghi chạy qua nó | we do not let every write run through it |
| tách rõ tầng điều khiển với tầng dữ liệu | separate the control plane from the data plane clearly |
| đặt consensus đúng chỗ | place consensus in the right spot |
| chứ không nhét nó vào đường đi nóng | rather than squeezing it into the hot path |
| nếu ai đó đề nghị | if somebody proposes |
| chúng ta phải chặn lại | we have to block it |
| đúng khái niệm nhưng sai chỗ đặt | right about the concept but wrong about where to put it |

**Thuật ngữ cần nhớ**

- tầng điều khiển → **the control plane**
- tầng dữ liệu → **the data plane**
- đường đi nóng → **the hot path**
- siêu dữ liệu → **metadata**
- ghi xuống đĩa → **to write down to disk**

---

## Mô hình ghi nhớ

**Tiếng Việt**

Consensus là việc nhiều node đồng ý với nhau dù có node chết, và nó dựa trên đa số nên chúng ta dùng số node lẻ để cấm split-brain. Raft là phiên bản dễ hiểu của Paxos và đã thành chuẩn thực tế, nhưng nó đúng mà đắt, cho nên chúng ta chỉ đặt nó ở tầng điều khiển và tránh xa đường đi nóng của dữ liệu.

**English (bám cấu trúc tiếng Việt)**

Consensus is many nodes agreeing with each other even when nodes die, and it rests on the majority so we use an odd number of nodes in order to forbid split-brain. Raft is the understandable version of Paxos and has become the de facto standard, but it is correct and expensive, therefore we only place it at the control plane and keep it away from the hot path of the data.

---

## Bảng thuật ngữ tổng hợp

| Tiếng Việt | Tiếng Anh | Ghi chú phát âm (Anh–Anh) |
|---|---|---|
| sự đồng thuận | **consensus** | *kơn-SEN-sợs* — trọng âm giữa, không đọc "con-sen-sút" |
| lỗi do node chết | **a crash fault** | *fault* /fɔːlt/ "FOOLT" — nguyên âm dài /ɔː/, cụm cuối **-lt** phải bật ra |
| cụm máy | **a cluster** | *KLAS-tơ* — cụm **kl** đầu phải rõ |
| kho điều phối | **a coordination store** | *kou-o-di-NÊI-shợn* — trọng âm âm thứ tư |
| nhân bản log | **log replication** | *re-pli-KÊI-shợn* — trọng âm áp chót |
| thuật toán | **an algorithm** | *AL-gơ-ri-đợm* /ˈælɡərɪðəm/ — đuôi có âm /ð/ |
| chuẩn trên thực tế | **the de facto standard** | *đây là tiếng Latin*: *đei FAK-tou* /deɪ ˈfæktəʊ/ |
| tính dễ hiểu | **understandability** | *an-đơ-stan-đơ-BI-lơ-ti* — trọng âm rơi vào **-BI-** |
| điều kiện an toàn | **a safety condition** | |
| hiện thực cho đúng | **to implement correctly** | *IM-pli-mợnt* (động từ) — trọng âm đầu |
| số phiếu tối thiểu | **a quorum** | *KWO-rợm* — có âm /kw/ đầu |
| phe đa số | **the majority side** | *mơ-DJO-rơ-ti* — trọng âm âm thứ hai |
| phe thiểu số | **the minority side** | *mai-NO-rơ-ti* — âm đầu là /maɪ/, khác `majority` |
| khả năng chịu lỗi | **fault tolerance** | *TO-lơ-rợns* — trọng âm đầu, cụm cuối **-ns** |
| số lẻ | **an odd number** | *odd* /ɒd/ — nguyên âm ngắn, đuôi /d/ phải bật |
| phần nguyên | **the integer part** | *IN-ti-djơ* — chữ `g` đọc /dʒ/ |
| tín hiệu nhịp tim | **a heartbeat** | *HAAT-biit* — câm chữ **r** trong `heart` (giọng Anh) |
| thời gian chờ bầu cử | **the election timeout** | *i-LEK-shợn* — trọng âm giữa |
| ứng viên | **a candidate** | Anh: *KAN-di-đợt* /ˈkændɪdət/ — âm cuối yếu, không đọc "can-đi-đêit" |
| nhiệm kỳ | **a term** | /tɜːm/ — nguyên âm dài /ɜː/ |
| đồng hồ logic | **a logical clock** | *LO-dji-kợl* — chữ `g` đọc /dʒ/ |
| lá phiếu | **a vote** | /vəʊt/ — âm đầu /v/ phải cắn môi, không thành /b/ |
| não chia đôi, hai leader | **split-brain** | cụm **spl-** ở đầu rất khó, tập chậm: *s-pl-it* |
| mâu thuẫn nhau | **to contradict each other** | *kon-trơ-DIKT* — trọng âm cuối, cụm **-kt** phải bật |
| hỏng dữ liệu | **to corrupt data** | *kơ-RAPT* — trọng âm sau |
| thoái vị | **to step down** | |
| mạng lành lại | **the network heals** | *heal* /hiːl/ — khác `hell`, nguyên âm dài |
| bền vững qua khởi động lại | **durable** | Anh: *DYUA-rơ-bợl* /ˈdjʊərəbl/ — có âm /dj/ |
| chuỗi thứ tự | **an ordering sequence** | *SII-kwợns* — trọng âm đầu, có âm /kw/ |
| hạ tầng | **infrastructure** | *IN-frơ-strak-tsơ* — trọng âm đầu, cụm **-str-** ở giữa |
| theo dõi thay đổi | **to watch for changes** | *watch* /wɒtʃ/ — đuôi /tʃ/ |
| quyền thuê có hạn | **a lease** | /liːs/ — đuôi /s/, đừng lẫn với `least` |
| tầng điều khiển | **the control plane** | *kơn-TROUL* — trọng âm sau |
| tầng dữ liệu | **the data plane** | *data* Anh = *DÊI-tơ* |
| đường đi nóng | **the hot path** | *path* Anh = /pɑːθ/ — nguyên âm dài, đuôi **th** |
| siêu dữ liệu | **metadata** | Anh: *ME-tơ-đêi-tơ* — trọng âm đầu |
| ghi xuống đĩa | **to write down to disk** | *write* câm chữ **w** |
| độ trễ | **latency** | *LÊI-tợn-si* — trọng âm đầu |
| thông lượng | **throughput** | *THRUU-put* — âm **th** và cụm **thr** |
| phán đoán | **judgement** | *DJADJ-mợnt* — hai âm /dʒ/ trong một từ |
| cố tình | **deliberately** | *đi-LI-bơ-rợt-li* — trọng âm âm thứ hai |

---

## Bài nói

> **Cách luyện:** bật ghi âm, nói **60–90 giây** cho mỗi đề, **không nhìn tài liệu**. Mỗi bài nói phải dùng ít nhất **sáu thuật ngữ** trong bảng trên. Nói xong nghe lại một lần, đánh dấu chỗ nào bị vấp hoặc phát âm sai, rồi nói lại đề đó một lần nữa.

1. **Explain to a junior developer** what problem consensus solves and name three things in your stack that quietly depend on it.

2. **Describe what happens**, step by step, to a five-node cluster during a partition that leaves three nodes on one side and two on the other, and then what happens when the network heals.

3. **A colleague asks:** *"Our cluster has five nodes and three of them went down — why can we not write any more? Surely two working nodes is better than none."* Explain why the system is behaving correctly.

4. **Explain why teams choose an odd number of nodes** for a consensus cluster. Use three, four and five nodes in your comparison.

5. **Someone proposes routing every application write through etcd** so the whole platform gets strong consistency. Explain why you would block that proposal, and where consensus does belong.

6. **When would you reach for etcd or ZooKeeper instead of a Redis lock?** Connect your answer to fencing tokens and to the difference between locking for efficiency and locking for correctness.

7. **An interviewer asks you to walk through Paxos.** Explain how you would answer at a senior level without reciting the algorithm, and what you would say about Raft instead.

---

> 🔚 **Hết Bài 7 — cũng là bài cuối của mục I.** Bảy bài đã nối thành một mạch: race là vấn đề gốc, lock và idempotency chống nó trong một database, distributed lock đưa nó ra phạm vi phân tán, fencing token đòi hỏi một chuỗi thứ tự bền vững, và consensus chính là thứ sinh ra chuỗi đó — trong khi CAP, PACELC và các consistency model đóng khung mọi đánh đổi ở tầng replica.
