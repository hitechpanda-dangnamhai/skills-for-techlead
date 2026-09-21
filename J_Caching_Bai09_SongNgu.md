# Bài 9 — Mở rộng Redis: replica · cluster · sentinel · RediSearch
### Bản song ngữ: tiếng Anh bám cấu trúc tiếng Việt

> **Cách dùng bài này:** đọc đoạn tiếng Việt trước → tự dịch trong đầu → so với đoạn tiếng Anh bên dưới → **đọc to bản tiếng Anh hai lần**: lần một đọc chậm cho đúng, lần hai đọc trôi như đang nói với người phỏng vấn.

---

## Phần 1 — Khi nào một node hết đủ và ba hướng mở rộng

**Tiếng Việt**

Ở Bài 7 chúng ta đã biết phần thực thi lệnh của Redis giới hạn ở khoảng một lõi. Vì vậy câu hỏi tiếp theo rất tự nhiên: khi một node không còn đủ nữa thì chúng ta làm gì. Một node hết đủ trong hai trường hợp: khi dữ liệu vượt quá RAM của máy, hoặc khi thông lượng vượt quá năng lực của một lõi và của băng thông mạng. Việc đầu tiên chúng ta phải làm là nhận diện đúng nút thắt, tức là xác định xem vấn đề nằm ở bộ nhớ, ở CPU hay ở mạng. Nếu chúng ta thêm node mà không biết nút thắt nằm ở đâu, chúng ta có thể tốn tiền mà không giải quyết được gì. Hướng thứ nhất là thêm bản sao, và hướng này giúp chúng ta mở rộng phần đọc đồng thời có tính sẵn sàng cao. Hướng thứ hai là chia dữ liệu ra nhiều node bằng cluster, và hướng này mở rộng cả phần ghi lẫn dung lượng.

Có một hướng thứ ba mà chúng ta cũng nên biết, đó là để client tự băm key và tự chọn node. Cách này từng rất phổ biến trước khi cluster xuất hiện, và nó vẫn còn được dùng trong một số hệ thống cũ. Nhược điểm của nó là mọi logic phân mảnh nằm trong client, vì vậy việc thêm hoặc bớt node trở nên rất khó. Với dự án mới, chúng ta nên dùng cluster hoặc dùng một dịch vụ quản lý sẵn. Cách trả lời tốt nhất trong phỏng vấn là nêu nút thắt trước, rồi mới nêu hướng mở rộng tương ứng.

**English (bám cấu trúc tiếng Việt)**

In Lesson 7 we learned that the command execution part of Redis is limited to about one core. Therefore the next question comes very naturally: what do we do when one node is no longer enough. One node stops being enough in two cases: when the data exceeds the RAM of the machine, or when the throughput exceeds the capacity of one core and of the network bandwidth. The first thing we have to do is identify the bottleneck correctly, that is to determine whether the problem lies in memory, in CPU or in the network. If we add nodes without knowing where the bottleneck lies, we may spend money without solving anything. The first direction is to add replicas, and this direction helps us scale the read side while also giving us high availability. The second direction is to split the data across many nodes with a cluster, and this direction scales both the write side and the capacity.

There is a third direction that we should also know, which is letting the client hash the key and pick the node itself. This way used to be very common before the cluster appeared, and it is still used in some legacy systems. Its drawback is that all the sharding logic sits in the client, therefore adding or removing a node becomes very hard. For a new project, we should use the cluster or use a managed service. The best way to answer in an interview is to name the bottleneck first, and only then name the matching scaling direction.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| câu hỏi tiếp theo rất tự nhiên | the next question comes very naturally |
| khi một node không còn đủ nữa | when one node is no longer enough |
| vượt quá năng lực của một lõi | exceeds the capacity of one core |
| nhận diện đúng nút thắt | identify the bottleneck correctly |
| tốn tiền mà không giải quyết được gì | spend money without solving anything |
| mở rộng phần đọc | scale the read side |
| mở rộng cả phần ghi lẫn dung lượng | scales both the write side and the capacity |
| để client tự băm key và tự chọn node | letting the client hash the key and pick the node itself |
| từng rất phổ biến trước khi cluster xuất hiện | used to be very common before the cluster appeared |
| một số hệ thống cũ | some legacy systems |
| mọi logic phân mảnh nằm trong client | all the sharding logic sits in the client |
| nêu nút thắt trước | name the bottleneck first |

**Thuật ngữ cần nhớ**

- nút thắt cổ chai → **a bottleneck**
- thông lượng → **throughput**
- bản sao → **a replica**
- phân mảnh dữ liệu → **sharding**
- hệ thống cũ để lại → **a legacy system**

---

## Phần 2 — Redis Cluster và mười sáu nghìn ba trăm tám mươi tư slot

**Tiếng Việt**

Redis Cluster chia không gian key thành mười sáu nghìn ba trăm tám mươi tư hash slot. Khi chúng ta gửi một key, Redis tính mã băm CRC16 của key đó rồi chia lấy dư cho tổng số slot. Kết quả của phép chia lấy dư chính là số hiệu slot, và mỗi node trong cụm chịu trách nhiệm giữ một dải slot nhất định. Nhờ lớp trung gian này, việc thêm hoặc bớt node chỉ là việc di chuyển một số slot từ node này sang node khác. Chúng ta gọi việc di chuyển đó là chia lại slot, và nó cho chúng ta quyền kiểm soát cách dữ liệu được phân bố. Nếu không có lớp slot mà băm thẳng key vào số node, thì mỗi lần đổi số node chúng ta sẽ phải xáo trộn gần như toàn bộ dữ liệu.

Chi tiết này rất đáng nói, vì nó cho thấy chúng ta hiểu vì sao thiết kế lại như vậy. Số slot là một hằng số, vì vậy ánh xạ từ key sang slot không bao giờ thay đổi. Thứ thay đổi chỉ là ánh xạ từ slot sang node, và ánh xạ đó rẻ hơn nhiều lần để cập nhật. Đây cũng chính là ý tưởng nằm sau các kỹ thuật băm nhất quán mà chúng ta gặp ở những hệ phân tán khác.

**English (bám cấu trúc tiếng Việt)**

Redis Cluster splits the key space into sixteen thousand three hundred and eighty-four hash slots. When we send a key, Redis computes the CRC16 hash of that key and then takes the remainder against the total number of slots. The result of that remainder operation is exactly the slot number, and each node in the cluster is responsible for holding a certain range of slots. Thanks to this intermediate layer, adding or removing a node is merely moving some slots from one node to another. We call that movement resharding, and it gives us control over how the data is distributed. If there were no slot layer and we hashed the key straight onto the number of nodes, then every time the node count changed we would have to shuffle almost all of the data.

This detail is well worth mentioning, because it shows that we understand why the design is the way it is. The slot count is a constant, therefore the mapping from key to slot never changes. The only thing that changes is the mapping from slot to node, and that mapping is many times cheaper to update. This is also exactly the idea behind the consistent hashing techniques that we meet in other distributed systems.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| chia không gian key thành | splits the key space into |
| chia lấy dư cho tổng số slot | takes the remainder against the total number of slots |
| chịu trách nhiệm giữ một dải slot nhất định | is responsible for holding a certain range of slots |
| nhờ lớp trung gian này | thanks to this intermediate layer |
| chỉ là việc di chuyển một số slot | is merely moving some slots |
| chia lại slot | resharding |
| cách dữ liệu được phân bố | how the data is distributed |
| băm thẳng key vào số node | hashed the key straight onto the number of nodes |
| xáo trộn gần như toàn bộ dữ liệu | shuffle almost all of the data |
| vì sao thiết kế lại như vậy | why the design is the way it is |
| rẻ hơn nhiều lần để cập nhật | many times cheaper to update |
| ý tưởng nằm sau | the idea behind |

**Thuật ngữ cần nhớ**

- khe băm → **a hash slot**
- phép chia lấy dư → **the remainder operation**
- chia lại phân mảnh → **resharding**
- băm nhất quán → **consistent hashing**
- ánh xạ → **a mapping**

---

## Phần 3 — Lệnh đa key và thẻ băm

**Tiếng Việt**

Cluster đặt ra một ràng buộc mà nhiều người chỉ phát hiện khi đã lên production. Một lệnh đụng tới nhiều key chỉ chạy được khi tất cả các key đó nằm trên cùng một slot. Điều đó nghĩa là lệnh đọc nhiều key một lượt, transaction đa key và script Lua đa key đều bị giới hạn. Nếu các key nằm ở những slot khác nhau, Redis từ chối lệnh và trả về lỗi chéo slot. Giải pháp là thẻ băm, tức là phần nằm trong cặp ngoặc nhọn ở trong tên key. Khi có thẻ băm, Redis chỉ băm phần nằm trong ngoặc chứ không băm toàn bộ tên key, vì vậy các key mang cùng một thẻ luôn rơi vào cùng một slot. Nhờ đó chúng ta ép được hồ sơ và cài đặt của cùng một người dùng nằm chung một node.

Điểm quan trọng là chúng ta phải tính chuyện này ngay từ lúc thiết kế key. Nếu chúng ta chỉ thêm thẻ băm sau khi hệ thống đã chạy, chúng ta phải di trú toàn bộ tên key và việc đó rất tốn kém. Nhưng chúng ta cũng không được lạm dụng thẻ băm, vì nếu mọi key đều mang cùng một thẻ thì tất cả dữ liệu dồn về một slot duy nhất. Khi đó chúng ta đã vô hiệu hoá chính việc phân mảnh mà chúng ta vừa dựng lên. Nguyên tắc là chỉ gom theo thực thể, ví dụ theo một người dùng hoặc theo một đơn hàng, và chỉ gom khi chúng ta thật sự cần thao tác nguyên tử trên nhóm đó.

**English (bám cấu trúc tiếng Việt)**

The cluster imposes a constraint that many people only discover once they are already in production. A command that touches several keys can only run when all of those keys sit on the same slot. That means the command that reads several keys in one go, multi-key transactions and multi-key Lua scripts are all limited. If the keys sit in different slots, Redis refuses the command and returns a cross-slot error. The solution is the hash tag, that is the part inside the curly brackets within the key name. When there is a hash tag, Redis hashes only the part inside the brackets instead of hashing the whole key name, therefore keys carrying the same tag always fall into the same slot. Thanks to that we can force the profile and the settings of the same user to sit on one node.

The important point is that we have to plan for this right from the moment we design the keys. If we only add hash tags after the system is already running, we have to migrate all the key names and that is very expensive. But we must not abuse hash tags either, because if every key carries the same tag then all the data piles onto one single slot. In that case we have disabled the very sharding that we have just built. The principle is to group only by entity, for example by one user or by one order, and to group only when we truly need an atomic operation on that group.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| đặt ra một ràng buộc | imposes a constraint |
| chỉ phát hiện khi đã lên production | only discover once they are already in production |
| một lệnh đụng tới nhiều key | a command that touches several keys |
| đọc nhiều key một lượt | reads several keys in one go |
| trả về lỗi chéo slot | returns a cross-slot error |
| phần nằm trong cặp ngoặc nhọn | the part inside the curly brackets |
| các key mang cùng một thẻ | keys carrying the same tag |
| ép được hồ sơ và cài đặt … nằm chung một node | force the profile and the settings … to sit on one node |
| ngay từ lúc thiết kế key | right from the moment we design the keys |
| chúng ta cũng không được lạm dụng thẻ băm | we must not abuse hash tags either |
| dồn về một slot duy nhất | piles onto one single slot |
| vô hiệu hoá chính việc phân mảnh | disabled the very sharding |
| chỉ gom theo thực thể | group only by entity |

**Thuật ngữ cần nhớ**

- thẻ băm → **a hash tag**
- ràng buộc → **a constraint**
- lỗi chéo slot → **a cross-slot error**
- ngoặc nhọn → **curly brackets**
- thực thể → **an entity**

---

## Phần 4 — Sentinel so với Cluster

**Tiếng Việt**

Sentinel và Cluster giải hai bài toán khác nhau, nhưng người ta rất hay nhầm chúng với nhau. Sentinel giải bài toán tính sẵn sàng cao cho một cụm gồm một máy chính và các bản sao, và nó không hề chia dữ liệu. Nó theo dõi máy chính, và khi máy chính chết thì nó bầu một bản sao lên làm máy chính mới. Cluster giải bài toán phân mảnh, tức là chia dữ liệu ra nhiều node, và nó cũng có cơ chế chuyển đổi dự phòng cho từng mảnh. Chúng ta chọn Sentinel khi dữ liệu vẫn vừa trong một node và chúng ta chỉ cần khả năng tự phục hồi. Chúng ta chọn Cluster khi dữ liệu hoặc lượng ghi đã vượt quá năng lực của một node.

Một cách trả lời gọn là gắn mỗi công cụ với một câu hỏi. Sentinel trả lời câu hỏi ai sẽ làm máy chính khi máy chính hiện tại chết. Cluster trả lời câu hỏi dữ liệu này nằm ở node nào. Nếu chúng ta phân biệt được hai câu hỏi đó, chúng ta sẽ không bao giờ chọn nhầm công cụ.

**English (bám cấu trúc tiếng Việt)**

Sentinel and Cluster solve two different problems, but people confuse them with each other very often. Sentinel solves the problem of high availability for a group made of one primary and its replicas, and it does not split the data at all. It watches the primary, and when the primary dies it promotes one replica to become the new primary. Cluster solves the problem of sharding, that is splitting the data across many nodes, and it also has a failover mechanism for each shard. We choose Sentinel when the data still fits in one node and we only need the ability to recover by itself. We choose Cluster when the data or the write volume has already exceeded the capacity of one node.

A compact way to answer is to tie each tool to one question. Sentinel answers the question of who will be the primary when the current primary dies. Cluster answers the question of which node this data sits on. If we can tell those two questions apart, we will never pick the wrong tool.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| người ta rất hay nhầm chúng với nhau | people confuse them with each other very often |
| một cụm gồm một máy chính và các bản sao | a group made of one primary and its replicas |
| nó không hề chia dữ liệu | it does not split the data at all |
| bầu một bản sao lên làm máy chính mới | promotes one replica to become the new primary |
| cơ chế chuyển đổi dự phòng cho từng mảnh | a failover mechanism for each shard |
| khi dữ liệu vẫn vừa trong một node | when the data still fits in one node |
| khả năng tự phục hồi | the ability to recover by itself |
| lượng ghi đã vượt quá năng lực của một node | the write volume has already exceeded the capacity of one node |
| gắn mỗi công cụ với một câu hỏi | tie each tool to one question |
| chúng ta sẽ không bao giờ chọn nhầm công cụ | we will never pick the wrong tool |

**Thuật ngữ cần nhớ**

- máy chính → **the primary**
- bầu lên làm máy chính → **to promote to primary**
- chuyển đổi dự phòng → **failover**
- tính sẵn sàng cao → **high availability**
- tự phục hồi → **to recover by itself**

---

## Phần 5 — Nhân bản bất đồng bộ: cache chịu được, khoá thì không

**Tiếng Việt**

Cơ chế nhân bản của Redis là bất đồng bộ, và đây là điều chúng ta bắt buộc phải hiểu. Khi máy chính xác nhận một lệnh ghi cho client, dữ liệu đó có thể chưa kịp sang bản sao. Nếu máy chính chết ngay lúc đó và một bản sao được bầu lên thay thế, những lệnh ghi gần nhất sẽ biến mất. Với dữ liệu cache, hậu quả thường chấp nhận được, vì mất một vài key chỉ làm tăng số lần miss. Với khoá phân tán, hậu quả lại rất nghiêm trọng: một client giữ khoá trên máy chính cũ, còn một client khác giành được chính khoá đó trên máy chính mới. Khi đó hai client cùng tin rằng mình đang độc quyền, và toàn bộ mục đích của khoá bị phá vỡ. Đây chính là điểm phê bình nổi tiếng nhắm vào thuật toán Redlock, và phần phân tích đầy đủ nằm ở mục về khoá phân tán.

Vì vậy chúng ta phải phân loại dữ liệu theo mức chịu đựng chứ không phân loại theo công cụ. Nếu mất vài lệnh ghi cuối cùng chỉ làm chậm hệ thống, dùng Redis là hoàn toàn ổn. Nếu mất vài lệnh ghi cuối cùng phá vỡ tính đúng đắn, chúng ta cần một hệ thống có đồng thuận thật sự. Cách phân loại này cũng áp dụng khi chúng ta quyết định có đọc từ bản sao hay không, vì bản sao luôn chậm hơn máy chính một khoảng.

**English (bám cấu trúc tiếng Việt)**

The replication mechanism of Redis is asynchronous, and this is something we absolutely have to understand. When the primary acknowledges a write to the client, that data may not have reached the replica yet. If the primary dies at exactly that moment and a replica is promoted to take its place, the most recent writes will disappear. For cache data, the consequence is usually acceptable, because losing a few keys only increases the number of misses. For a distributed lock, the consequence is very serious: one client holds the lock on the old primary, while another client wins that same lock on the new primary. In that case both clients believe that they have exclusive access, and the entire purpose of the lock is broken. This is exactly the famous criticism aimed at the Redlock algorithm, and the full analysis belongs to the topic on distributed locks.

Therefore we have to classify data by how much loss it can tolerate instead of classifying it by tool. If losing the last few writes only makes the system slower, using Redis is completely fine. If losing the last few writes breaks correctness, we need a system with real consensus. This classification also applies when we decide whether to read from a replica or not, because a replica is always behind the primary by some interval.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| đây là điều chúng ta bắt buộc phải hiểu | this is something we absolutely have to understand |
| máy chính xác nhận một lệnh ghi cho client | the primary acknowledges a write to the client |
| có thể chưa kịp sang bản sao | may not have reached the replica yet |
| những lệnh ghi gần nhất sẽ biến mất | the most recent writes will disappear |
| chỉ làm tăng số lần miss | only increases the number of misses |
| giành được chính khoá đó trên máy chính mới | wins that same lock on the new primary |
| cùng tin rằng mình đang độc quyền | both believe that they have exclusive access |
| toàn bộ mục đích của khoá bị phá vỡ | the entire purpose of the lock is broken |
| điểm phê bình nổi tiếng nhắm vào | the famous criticism aimed at |
| phân loại dữ liệu theo mức chịu đựng | classify data by how much loss it can tolerate |
| phá vỡ tính đúng đắn | breaks correctness |
| một hệ thống có đồng thuận thật sự | a system with real consensus |

**Thuật ngữ cần nhớ**

- nhân bản bất đồng bộ → **asynchronous replication**
- xác nhận đã ghi → **to acknowledge a write**
- độ trễ nhân bản → **replication lag**
- quyền truy cập độc quyền → **exclusive access**
- đồng thuận → **consensus**

---

## Phần 6 — RediSearch và các module mở rộng

**Tiếng Việt**

Redis còn có các module mở rộng khả năng truy vấn vượt xa mô hình khoá và giá trị. RediSearch cho phép chúng ta tạo chỉ mục phụ, tìm kiếm toàn văn, tìm kiếm theo vector và tính toán tổng hợp ngay trên dữ liệu đang nằm trong Redis. Lợi ích lớn nhất là chúng ta không phải chuyển dữ liệu sang một hệ thống khác chỉ để tìm kiếm. Nhưng chúng ta phải nói rõ giới hạn của nó: module này không thay thế được một công cụ tìm kiếm đầy đủ khi nhu cầu tìm kiếm thật sự lớn. Với một kho tài liệu lớn và nhu cầu xếp hạng phức tạp, một hệ thống chuyên dụng vẫn là lựa chọn đúng. Từ phiên bản tám, nhiều module trong số này đã được tích hợp thẳng vào phần lõi.

Cách trả lời an toàn là đặt module vào đúng vị trí của nó trên phổ lựa chọn. Chúng ta nói rằng nó rất hợp khi dữ liệu đã nằm sẵn trong Redis và nhu cầu truy vấn ở mức vừa phải. Chúng ta cũng nói rằng nó không phải là lý do để chúng ta bỏ hẳn một công cụ tìm kiếm chuyên dụng. Cách nói theo phổ luôn nghe chín chắn hơn cách nói rằng công cụ này thay được công cụ kia.

**English (bám cấu trúc tiếng Việt)**

Redis also has modules that extend the query capability far beyond the key-value model. RediSearch lets us create secondary indexes, full-text search, vector search and aggregation right on the data already sitting in Redis. The biggest benefit is that we do not have to move the data into another system just in order to search it. But we have to state its limit clearly: this module does not replace a full search engine when the search requirement is genuinely large. For a large document store and complex ranking requirements, a dedicated system is still the right choice. From version eight onwards, many of these modules have been integrated straight into the core.

The safe way to answer is to place the module in its proper position on the spectrum of options. We say that it fits very well when the data already sits in Redis and the query requirement is moderate. We also say that it is not a reason for us to drop a dedicated search engine entirely. Speaking in terms of a spectrum always sounds more mature than saying that one tool replaces another.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| mở rộng khả năng truy vấn vượt xa | extend the query capability far beyond |
| tạo chỉ mục phụ | create secondary indexes |
| tính toán tổng hợp | aggregation |
| chỉ để tìm kiếm | just in order to search it |
| chúng ta phải nói rõ giới hạn của nó | we have to state its limit clearly |
| khi nhu cầu tìm kiếm thật sự lớn | when the search requirement is genuinely large |
| một hệ thống chuyên dụng | a dedicated system |
| được tích hợp thẳng vào phần lõi | integrated straight into the core |
| đúng vị trí của nó trên phổ lựa chọn | its proper position on the spectrum of options |
| nhu cầu truy vấn ở mức vừa phải | the query requirement is moderate |
| bỏ hẳn một công cụ tìm kiếm chuyên dụng | drop a dedicated search engine entirely |
| nghe chín chắn hơn | sounds more mature |

**Thuật ngữ cần nhớ**

- chỉ mục phụ → **a secondary index**
- tìm kiếm toàn văn → **full-text search**
- tính toán tổng hợp → **aggregation**
- chuyên dụng → **dedicated**
- phổ lựa chọn → **a spectrum of options**

---

## Phần 7 — Góc tech lead: lỗi chéo slot lần đầu lên cluster

**Tiếng Việt**

Tình huống thách đố của bài này xảy ra ngay lần đầu một đội chuyển sang cluster. Đoạn code gọi một lệnh đọc nhiều key cho hai người dùng khác nhau, và nó nhận về lỗi chéo slot. Nguyên nhân là hai key đó băm ra hai slot khác nhau, vì vậy lệnh đa key bị từ chối. Cách sửa sai là ép mọi key về cùng một slot bằng một thẻ băm chung, vì làm như vậy chúng ta mất luôn lợi ích của việc phân mảnh. Cách sửa đúng gồm hai phần tuỳ theo nhu cầu thật sự. Nếu chúng ta thật sự cần thao tác nguyên tử trên một nhóm key thuộc cùng một thực thể, chúng ta dùng thẻ băm theo thực thể đó. Nếu chúng ta chỉ cần tra cứu nhiều người dùng khác nhau, chúng ta gộp nhiều lệnh đọc đơn lẻ vào một pipeline thay vì dùng một lệnh đa key.

Chúng ta nên biến điều này thành một mục kiểm tra khi review code chạy trên cluster. Câu hỏi là: mọi key trong một lệnh đa key có nằm cùng slot hay không. Công cụ AI thường viết lệnh đa key một cách rất tự nhiên, vì nó không biết hệ thống đang chạy trên cluster. Đây là loại lỗi chỉ lộ ra lúc chạy thật, vì vậy chúng ta phải bắt nó ngay ở khâu review.

**English (bám cấu trúc tiếng Việt)**

The challenge scenario of this lesson happens the very first time a team moves onto a cluster. The code calls a command that reads several keys for two different users, and it receives a cross-slot error. The cause is that those two keys hash into two different slots, therefore the multi-key command is refused. The wrong fix is to force every key onto the same slot with one shared hash tag, because doing so loses the benefit of sharding altogether. The correct fix has two parts depending on the real requirement. If we truly need an atomic operation on a group of keys belonging to the same entity, we use a hash tag based on that entity. If we only need to look up several different users, we batch several individual read commands into a pipeline instead of using one multi-key command.

We should turn this into a check item when we review code that runs on a cluster. The question is: do all the keys in a multi-key command sit on the same slot. AI tools usually write multi-key commands very naturally, because they do not know that the system is running on a cluster. This is the kind of bug that only shows up at run time, therefore we have to catch it right at the review stage.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| ngay lần đầu một đội chuyển sang cluster | the very first time a team moves onto a cluster |
| nó nhận về lỗi chéo slot | it receives a cross-slot error |
| băm ra hai slot khác nhau | hash into two different slots |
| cách sửa sai là | the wrong fix is |
| chúng ta mất luôn lợi ích của việc phân mảnh | we lose the benefit of sharding altogether |
| tuỳ theo nhu cầu thật sự | depending on the real requirement |
| thuộc cùng một thực thể | belonging to the same entity |
| gộp nhiều lệnh đọc đơn lẻ vào một pipeline | batch several individual read commands into a pipeline |
| một mục kiểm tra khi review | a check item when we review |
| viết lệnh đa key một cách rất tự nhiên | write multi-key commands very naturally |
| loại lỗi chỉ lộ ra lúc chạy thật | the kind of bug that only shows up at run time |
| bắt nó ngay ở khâu review | catch it right at the review stage |

**Thuật ngữ cần nhớ**

- lệnh đa key → **a multi-key command**
- gộp lệnh gửi một lượt → **a pipeline**
- lúc chạy thật → **at run time**
- khâu review → **the review stage**
- mục kiểm tra → **a check item**

---

## Mô hình ghi nhớ

**Tiếng Việt**

Khi một node hết đủ, chúng ta thêm bản sao để mở rộng phần đọc, dùng cluster với mười sáu nghìn ba trăm tám mươi tư slot để mở rộng phần ghi và dung lượng, và dùng Sentinel khi chúng ta chỉ cần chuyển đổi dự phòng. Nhưng cơ chế nhân bản là bất đồng bộ, vì vậy cache chịu được việc mất vài lệnh ghi còn khoá phân tán thì không.

**English (bám cấu trúc tiếng Việt)**

When one node is no longer enough, we add replicas to scale the read side, we use a cluster with sixteen thousand three hundred and eighty-four slots to scale the write side and the capacity, and we use Sentinel when we only need failover. But the replication mechanism is asynchronous, therefore a cache can tolerate losing a few writes while a distributed lock cannot.

---

## Bảng thuật ngữ tổng hợp

| Tiếng Việt | Tiếng Anh | Ghi chú phát âm (Anh-Anh) |
|---|---|---|
| nút thắt cổ chai | a bottleneck | |
| thông lượng | throughput | **THROO**-put — âm /θ/ đầu lưỡi |
| băng thông | bandwidth | **BAND**-width — kết thúc bằng /dθ/ |
| dung lượng | capacity | kə-**PAS**-ə-ti |
| bản sao | a replica | **REP**-li-kə — trọng âm đầu, không đọc "ri-plai-ka" |
| nhân bản | replication | rep-li-**KAY**-shən |
| phân mảnh dữ liệu | sharding | shard /ʃɑːd/ — bắt đầu bằng /ʃ/ |
| hệ thống cũ để lại | a legacy system | **LEG**-ə-si |
| dịch vụ quản lý sẵn | a managed service | **MAN**-ijd |
| khe băm | a hash slot | |
| phép chia lấy dư | the remainder operation | ri-**MAIN**-də |
| chia lại phân mảnh | resharding | |
| băm nhất quán | consistent hashing | kən-**SIS**-tənt |
| ánh xạ | a mapping | |
| không gian key | the key space | |
| thẻ băm | a hash tag | |
| ràng buộc | a constraint | kən-**STRAINT** — kết thúc bằng cụm /nt/ |
| lỗi chéo slot | a cross-slot error | |
| ngoặc nhọn | curly brackets | **KER**-li |
| thực thể | an entity | **EN**-ti-ti |
| di trú dữ liệu | to migrate | Anh-Anh mai-**GRAYT** |
| lạm dụng | to abuse | động từ ə-**BYOOZ** (/z/), danh từ ə-**BYOOS** (/s/) |
| máy chính | the primary | **PRAI**-mə-ri — ba âm tiết |
| bầu lên làm máy chính | to promote to primary | prə-**MƏUT** |
| chuyển đổi dự phòng | failover | **FAIL**-əʊ-və |
| tính sẵn sàng cao | high availability | ə-veil-ə-**BIL**-ə-ti |
| tự phục hồi | to recover by itself | ri-**KUV**-ə — "co" đọc thành /kʌ/ |
| nhân bản bất đồng bộ | asynchronous replication | ei-**SIN**-krə-nəs |
| xác nhận đã ghi | to acknowledge a write | ək-**NOL**-ij — chữ **k** đầu câm |
| độ trễ nhân bản | replication lag | |
| quyền truy cập độc quyền | exclusive access | ik-**SKLOO**-siv |
| đồng thuận | consensus | kən-**SEN**-səs |
| tính đúng đắn | correctness | |
| chịu đựng được | to tolerate | **TOL**-ə-reit |
| chỉ mục phụ | a secondary index | **SEK**-ən-dri |
| tìm kiếm toàn văn | full-text search | search /sɜːtʃ/ — kết thúc bằng /tʃ/ |
| tìm kiếm theo vector | vector search | **VEK**-tə |
| tính toán tổng hợp | aggregation | ag-ri-**GAY**-shən |
| chuyên dụng | dedicated | **DED**-i-kay-tid |
| phổ lựa chọn | a spectrum of options | **SPEK**-trəm |
| công cụ tìm kiếm | a search engine | **EN**-jin |
| phần lõi | the core | |
| lệnh đa key | a multi-key command | |
| gộp lệnh gửi một lượt | a pipeline | **PAIP**-lain |
| lúc chạy thật | at run time | |
| khâu review | the review stage | ri-**VYOO** |
| mục kiểm tra | a check item | |
| bộ nhớ đệm | cache | /kæʃ/ — đọc y hệt "cash" |

---

## Bài nói

> **Cách luyện:** bật ghi âm, nói **60–90 giây** cho mỗi đề, **không nhìn tài liệu**. Mỗi bài nói phải dùng ít nhất **sáu thuật ngữ** trong bảng trên. Nghe lại một lần, đánh dấu chỗ bạn ngập ngừng, rồi nói lại đúng đề đó thêm một lần nữa.

1. **Explain to a junior** when one Redis node stops being enough, and the directions available to scale. Say what you would measure before choosing.

2. **Describe how Redis Cluster** maps a key to a node, and explain why the slot layer exists at all instead of hashing straight onto the node count.

3. **A colleague hit a cross-slot error** and wants to put one shared hash tag on every key so that the error goes away. Explain why you would push back and what you would do instead.

4. **Explain the difference** between Sentinel and Cluster by naming the one question each of them answers.

5. **A teammate says** that because Redis has replication, the replica always has the data, so a distributed lock stays safe across a failover. Explain what is wrong with that claim.

6. **When would you use RediSearch**, and when would you insist on a dedicated search engine? Answer in terms of a spectrum rather than a winner.

7. **You are reviewing code** for a service that is about to move onto Redis Cluster. Describe what you check, and what you would tell the author about multi-key commands.
