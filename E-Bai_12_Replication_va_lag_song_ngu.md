# Bài 12 — Replication & lag
### Bản song ngữ: tiếng Anh bám cấu trúc tiếng Việt

> **Cách dùng bài này:** đọc đoạn tiếng Việt trước, tự dịch trong đầu, rồi mới so với đoạn tiếng Anh bên dưới. Chỗ nào lệch thì xem bảng ánh xạ cụm khó. Cuối cùng, **đọc to đoạn tiếng Anh hai lần** — một lần chậm để đúng âm, một lần với tốc độ nói thật.

---

## Phần 1 — Replication giải bài toán nào và không giải bài toán nào

**Tiếng Việt**

Replication nghĩa là chúng ta sao chép dữ liệu từ một node sang một hoặc nhiều node khác. Chúng ta làm việc này vì ba lý do, và chúng ta nên nói rõ cả ba lý do trong buổi phỏng vấn. Lý do thứ nhất là mở rộng khả năng đọc, vì chúng ta có thể chia tải đọc ra cho nhiều bản sao. Lý do thứ hai là tính sẵn sàng cao, vì khi máy chính chết thì chúng ta có thể nâng một bản sao lên thay thế. Lý do thứ ba là tách tải, vì chúng ta đẩy việc sao lưu và việc chạy báo cáo sang bản sao để máy chính chỉ lo phục vụ người dùng. Nhưng có một điều mà replication không giải quyết được, đó là mở rộng khả năng ghi. Mọi lệnh ghi vẫn phải đi về máy chính, vì vậy nếu điểm nghẽn của chúng ta nằm ở đường ghi thì thêm bản sao sẽ không giúp gì cả.

**English (bám cấu trúc tiếng Việt)**

Replication means that we copy the data from one node to one or more other nodes. We do this for three reasons, and we should state all three reasons clearly in an interview. The first reason is scaling reads, because we can spread the read load across many replicas. The second reason is high availability, because when the primary dies we can promote a replica to take its place. The third reason is separating the load, because we push backups and report queries over to a replica so that the primary only has to serve the users. But there is one thing that replication does not solve, namely scaling writes. Every write still has to go to the primary, therefore if our bottleneck is on the write path then adding replicas will not help at all.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| chúng ta nên nói rõ cả ba lý do | we should state all three reasons clearly |
| chia tải đọc ra cho nhiều bản sao | spread the read load across many replicas |
| khi máy chính chết | when the primary dies |
| nâng một bản sao lên thay thế | promote a replica to take its place |
| để máy chính chỉ lo phục vụ người dùng | so that the primary only has to serve the users |
| có một điều mà replication không giải quyết được | there is one thing that replication does not solve |
| mọi lệnh ghi vẫn phải đi về máy chính | every write still has to go to the primary |
| điểm nghẽn của chúng ta nằm ở đường ghi | our bottleneck is on the write path |
| thêm bản sao sẽ không giúp gì cả | adding replicas will not help at all |

**Thuật ngữ cần nhớ**

- máy chính → **the primary**
- bản sao → **the replica**
- mở rộng khả năng đọc → **scale reads**
- tính sẵn sàng cao → **high availability**
- nâng lên làm máy chính → **promote**

---

## Phần 2 — Sao chép đồng bộ và sao chép bất đồng bộ

**Tiếng Việt**

Chúng ta có hai kiểu sao chép, và chúng khác nhau ở thời điểm máy chính báo thành công cho ứng dụng. Với sao chép đồng bộ, máy chính chờ ít nhất một bản sao xác nhận đã nhận dữ liệu rồi nó mới báo commit thành công. Cách này bảo đảm rằng chúng ta không mất dữ liệu đã commit, nhưng nó làm mỗi lệnh ghi chậm thêm đúng bằng một vòng đi và về trên mạng. Nguy hiểm hơn, nếu bản sao chết hoặc mạng đứt, máy chính có thể bị chặn và cả hệ thống ngừng nhận ghi. Với sao chép bất đồng bộ, máy chính báo thành công ngay và nó gửi dữ liệu sang bản sao ở phía sau. Cách này nhanh hơn nhiều, nhưng nếu máy chính chết trước khi dữ liệu kịp sang, chúng ta sẽ mất những giao dịch cuối cùng khi chuyển đổi. Vì vậy, chúng ta thường chọn đồng bộ cho dữ liệu sống còn như tiền, và chúng ta chọn bất đồng bộ cho phần còn lại.

**English (bám cấu trúc tiếng Việt)**

We have two kinds of replication, and they differ in the moment when the primary reports success to the application. With synchronous replication, the primary waits for at least one replica to confirm that it has received the data before it reports a successful commit. This way guarantees that we do not lose committed data, but it makes each write slower by exactly one network round trip. More dangerously, if the replica dies or the network breaks, the primary can be blocked and the whole system stops accepting writes. With asynchronous replication, the primary reports success immediately and it sends the data over to the replica behind the scenes. This way is much faster, but if the primary dies before the data manages to get across, we will lose the last transactions during the failover. Therefore, we usually choose synchronous for critical data such as money, and we choose asynchronous for everything else.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| khác nhau ở thời điểm máy chính báo thành công | differ in the moment when the primary reports success |
| chờ ít nhất một bản sao xác nhận | waits for at least one replica to confirm |
| rồi nó mới báo commit thành công | before it reports a successful commit |
| chậm thêm đúng bằng một vòng đi và về trên mạng | slower by exactly one network round trip |
| nguy hiểm hơn | more dangerously |
| máy chính có thể bị chặn | the primary can be blocked |
| nó gửi dữ liệu sang bản sao ở phía sau | it sends the data over to the replica behind the scenes |
| trước khi dữ liệu kịp sang | before the data manages to get across |
| dữ liệu sống còn như tiền | critical data such as money |

**Thuật ngữ cần nhớ**

- sao chép đồng bộ → **synchronous replication**
- sao chép bất đồng bộ → **asynchronous replication**
- xác nhận → **confirm / acknowledge**
- chuyển đổi khi sự cố → **failover**
- dữ liệu sống còn → **critical data**

---

## Phần 3 — Độ trễ sao chép và bẫy đọc lại thứ vừa ghi

**Tiếng Việt**

Bản sao luôn chạy chậm hơn máy chính một chút, và khoảng chậm đó được gọi là độ trễ sao chép. Ở điều kiện bình thường, độ trễ này chỉ vài mili giây, nhưng nó có thể vọt lên vài giây khi có một lệnh ghi lớn hoặc khi mạng chậm. Vấn đề xuất hiện khi người dùng ghi một thứ lên máy chính rồi ngay lập tức đọc lại từ một bản sao. Lúc đó, bản sao chưa nhận được thay đổi, vì vậy người dùng nhìn thấy dữ liệu cũ và họ nghĩ rằng thao tác của mình đã thất bại. Trong sách vở, tính chất bị vi phạm ở đây được gọi là đọc thấy chính thứ mình vừa ghi. Đây là lỗi rất khó tái hiện ở môi trường thử nghiệm, vì ở đó độ trễ gần như bằng không và mọi thứ trông hoàn hảo.

**English (bám cấu trúc tiếng Việt)**

A replica always runs a little behind the primary, and that gap is called replication lag. Under normal conditions, this lag is only a few milliseconds, but it can jump up to a few seconds when there is a large write or when the network is slow. The problem shows up when a user writes something to the primary and then immediately reads it back from a replica. At that point, the replica has not received the change yet, therefore the user sees the old data and they think that their action has failed. In the textbooks, the property being violated here is called reading your own writes. This is a bug that is very hard to reproduce in a test environment, because there the lag is nearly zero and everything looks perfect.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| chạy chậm hơn máy chính một chút | runs a little behind the primary |
| khoảng chậm đó được gọi là độ trễ sao chép | that gap is called replication lag |
| ở điều kiện bình thường | under normal conditions |
| nó có thể vọt lên vài giây | it can jump up to a few seconds |
| rồi ngay lập tức đọc lại từ một bản sao | and then immediately reads it back from a replica |
| họ nghĩ rằng thao tác của mình đã thất bại | they think that their action has failed |
| tính chất bị vi phạm ở đây | the property being violated here |
| đọc thấy chính thứ mình vừa ghi | reading your own writes |
| rất khó tái hiện ở môi trường thử nghiệm | very hard to reproduce in a test environment |
| mọi thứ trông hoàn hảo | everything looks perfect |

**Thuật ngữ cần nhớ**

- độ trễ sao chép → **replication lag**
- dữ liệu cũ | ôi → **stale data**
- đọc thấy thứ mình vừa ghi → **read-your-own-writes**
- tái hiện lỗi → **reproduce a bug**
- vi phạm → **violate**

---

## Phần 4 — Bốn cách xử lý bài toán đọc ngay sau khi ghi

**Tiếng Việt**

Chúng ta có bốn cách xử lý, và chúng ta chọn theo yêu cầu nhất quán của từng lần đọc chứ chúng ta không chọn một lần cho cả hệ thống. Cách thứ nhất là chúng ta định tuyến những lần đọc quan trọng về thẳng máy chính, ví dụ màn hình hiện ra ngay sau khi người dùng bấm lưu. Cách thứ hai là chúng ta ghim một phiên người dùng vào máy chính trong một khoảng thời gian ngắn sau khi họ vừa ghi. Cách thứ ba là chúng ta ghi lại vị trí log tại thời điểm ghi, rồi chúng ta chờ bản sao phát lại tới đúng vị trí đó trước khi đọc. Cách thứ tư là chúng ta trả về kết quả từ cache của chính lần ghi đó, vì ứng dụng đã biết giá trị mới rồi. Điểm mấu chốt là chúng ta phân loại từng lần đọc, vì một danh sách báo cáo chấp nhận dữ liệu cũ vài giây trong khi màn hình xác nhận thanh toán thì không chấp nhận.

**English (bám cấu trúc tiếng Việt)**

We have four ways to handle it, and we choose according to the consistency requirement of each read and we do not choose once for the whole system. The first way is that we route the important reads straight to the primary, for example the screen that appears right after the user presses save. The second way is that we pin a user session to the primary for a short period after they have just written. The third way is that we record the log position at the moment of the write, then we wait for the replica to replay up to that exact position before we read. The fourth way is that we return the result from the cache of that same write, because the application already knows the new value. The key point is that we classify each read, because a report list accepts data that is a few seconds old while a payment confirmation screen does not accept it.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| chúng ta không chọn một lần cho cả hệ thống | we do not choose once for the whole system |
| định tuyến những lần đọc quan trọng về thẳng máy chính | route the important reads straight to the primary |
| màn hình hiện ra ngay sau khi người dùng bấm lưu | the screen that appears right after the user presses save |
| ghim một phiên người dùng vào máy chính | pin a user session to the primary |
| trong một khoảng thời gian ngắn sau khi họ vừa ghi | for a short period after they have just written |
| ghi lại vị trí log tại thời điểm ghi | record the log position at the moment of the write |
| chờ bản sao phát lại tới đúng vị trí đó | wait for the replica to replay up to that exact position |
| điểm mấu chốt là chúng ta phân loại từng lần đọc | the key point is that we classify each read |
| chấp nhận dữ liệu cũ vài giây | accepts data that is a few seconds old |

**Thuật ngữ cần nhớ**

- định tuyến → **route**
- phiên ghim vào một node → **sticky session**
- vị trí trong log → **log position (LSN)**
- phát lại → **replay**
- yêu cầu nhất quán → **consistency requirement**

---

## Phần 5 — Failover và rủi ro split-brain

**Tiếng Việt**

Khi máy chính chết, hệ thống phải nâng một bản sao lên làm máy chính mới, và quá trình đó được gọi là failover. Việc này nghe đơn giản nhưng nó ẩn chứa một rủi ro nghiêm trọng tên là split-brain. Split-brain xảy ra khi máy chính cũ thật ra vẫn còn sống, ví dụ nó chỉ bị mất mạng tạm thời, và nó vẫn tưởng mình là máy chính. Lúc đó chúng ta có hai node cùng nhận lệnh ghi, và dữ liệu của hai bên sẽ phân kỳ theo hai hướng không thể gộp lại. Để phòng chuyện này, hệ thống dùng quorum, nghĩa là chỉ nhóm nào chiếm đa số mới có quyền bầu ra máy chính mới. Hệ thống cũng dùng fencing, nghĩa là nó chủ động cắt máy chính cũ ra khỏi mạng hoặc khỏi kho lưu trữ trước khi nó nâng máy mới lên. Trong thực tế, chúng ta hiếm khi tự viết cơ chế này, mà chúng ta dùng Patroni hoặc một dịch vụ được quản lý sẵn.

**English (bám cấu trúc tiếng Việt)**

When the primary dies, the system has to promote a replica to become the new primary, and that process is called failover. This sounds simple but it hides a serious risk named split-brain. Split-brain happens when the old primary is in fact still alive, for example it has only lost the network temporarily, and it still believes that it is the primary. At that point we have two nodes both accepting writes, and the data on the two sides will diverge in two directions that cannot be merged back together. To prevent this, the system uses a quorum, which means that only the group holding the majority has the right to elect a new primary. The system also uses fencing, which means that it actively cuts the old primary off from the network or from the storage before it promotes the new one. In practice, we rarely write this mechanism ourselves, but we use Patroni or a managed service.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| nó ẩn chứa một rủi ro nghiêm trọng | it hides a serious risk |
| thật ra vẫn còn sống | is in fact still alive |
| nó chỉ bị mất mạng tạm thời | it has only lost the network temporarily |
| nó vẫn tưởng mình là máy chính | it still believes that it is the primary |
| hai node cùng nhận lệnh ghi | two nodes both accepting writes |
| sẽ phân kỳ theo hai hướng không thể gộp lại | will diverge in two directions that cannot be merged back together |
| chỉ nhóm nào chiếm đa số mới có quyền | only the group holding the majority has the right |
| nó chủ động cắt máy chính cũ ra khỏi mạng | it actively cuts the old primary off from the network |
| chúng ta hiếm khi tự viết cơ chế này | we rarely write this mechanism ourselves |

**Thuật ngữ cần nhớ**

- chuyển đổi khi sự cố → **failover**
- não phân đôi → **split-brain**
- phân kỳ dữ liệu → **diverge**
- đa số / nhóm đủ túc số → **majority / quorum**
- cô lập node hỏng → **fencing**

---

## Phần 6 — Sao chép vật lý và sao chép logic

**Tiếng Việt**

PostgreSQL có hai kiểu sao chép ở mức cơ chế, và chúng phục vụ hai mục đích khác nhau. Sao chép vật lý truyền thẳng dòng WAL sang bản sao, vì vậy bản sao là một bản giống hệt đến từng byte. Cách này nhanh và đơn giản, nhưng nó bắt hai bên phải chạy cùng một phiên bản chính và nó sao chép toàn bộ cụm. Sao chép logic thì giải mã WAL thành các thay đổi ở mức dòng, rồi nó gửi những thay đổi đó theo từng bảng đã đăng ký. Nhờ vậy, chúng ta chọn lọc được bảng nào cần sao chép, và hai bên có thể chạy hai phiên bản khác nhau. Đổi lại, sao chép logic tốn nhiều tài nguyên hơn và nó có những giới hạn riêng, ví dụ về cách nó xử lý thay đổi cấu trúc bảng.

**English (bám cấu trúc tiếng Việt)**

PostgreSQL has two kinds of replication at the mechanism level, and they serve two different purposes. Physical replication streams the WAL directly across to the replica, therefore the replica is a copy that is identical down to every byte. This way is fast and simple, but it forces both sides to run the same major version and it replicates the whole cluster. Logical replication instead decodes the WAL into row-level changes, then it sends those changes for each subscribed table. Thanks to that, we can select which tables need to be replicated, and the two sides can run two different versions. In exchange, logical replication costs more resources and it has its own limitations, for example in the way it handles schema changes.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| ở mức cơ chế | at the mechanism level |
| truyền thẳng dòng WAL sang bản sao | streams the WAL directly across to the replica |
| giống hệt đến từng byte | identical down to every byte |
| nó bắt hai bên phải chạy cùng một phiên bản chính | it forces both sides to run the same major version |
| nó sao chép toàn bộ cụm | it replicates the whole cluster |
| giải mã WAL thành các thay đổi ở mức dòng | decodes the WAL into row-level changes |
| theo từng bảng đã đăng ký | for each subscribed table |
| chúng ta chọn lọc được bảng nào cần sao chép | we can select which tables need to be replicated |
| nó có những giới hạn riêng | it has its own limitations |
| cách nó xử lý thay đổi cấu trúc bảng | the way it handles schema changes |

**Thuật ngữ cần nhớ**

- sao chép vật lý → **physical replication**
- sao chép logic → **logical replication**
- truyền luồng → **stream**
- cụm máy chủ → **cluster**
- đăng ký nhận thay đổi → **subscribe**

---

## Phần 7 — Sao chép logic dùng cho CDC và nâng cấp không downtime

**Tiếng Việt**

Chính khả năng chạy chéo phiên bản làm cho sao chép logic trở thành công cụ nâng cấp gần như không có thời gian chết. Chúng ta dựng một cụm mới ở phiên bản mới, chúng ta cho nó bám theo cụm cũ bằng sao chép logic, và chúng ta chờ nó đuổi kịp. Sau đó chúng ta dừng ghi trong vài giây, chúng ta kiểm tra hai bên đã khớp nhau, rồi chúng ta chuyển ứng dụng sang cụm mới. Ứng dụng thứ hai của sao chép logic là thu thập thay đổi dữ liệu, và người ta thường gọi tắt nó là CDC. Một công cụ như Debezium đọc luồng thay đổi đó rồi đẩy từng sự kiện sang hệ thống tìm kiếm, sang kho phân tích hoặc sang một message broker. Cách này tốt hơn hẳn việc quét bảng theo lịch, vì chúng ta lấy được mọi thay đổi theo đúng thứ tự mà chúng ta không làm nặng thêm cho cơ sở dữ liệu.

**English (bám cấu trúc tiếng Việt)**

It is exactly the ability to run across versions that makes logical replication a tool for upgrades with almost no downtime. We build a new cluster on the new version, we let it follow the old cluster through logical replication, and we wait for it to catch up. After that we stop writes for a few seconds, we check that the two sides match each other, and then we switch the application over to the new cluster. The second use of logical replication is change data capture, and people usually shorten it to CDC. A tool such as Debezium reads that change stream and then pushes each event over to a search system, to an analytical warehouse or to a message broker. This way is clearly better than scanning tables on a schedule, because we get every change in the right order and we do not put extra weight on the database.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| chính khả năng chạy chéo phiên bản | it is exactly the ability to run across versions |
| gần như không có thời gian chết | with almost no downtime |
| chúng ta cho nó bám theo cụm cũ | we let it follow the old cluster |
| chúng ta chờ nó đuổi kịp | we wait for it to catch up |
| chúng ta kiểm tra hai bên đã khớp nhau | we check that the two sides match each other |
| chuyển ứng dụng sang cụm mới | switch the application over to the new cluster |
| người ta thường gọi tắt nó là | people usually shorten it to |
| đọc luồng thay đổi đó | reads that change stream |
| việc quét bảng theo lịch | scanning tables on a schedule |
| chúng ta không làm nặng thêm cho cơ sở dữ liệu | we do not put extra weight on the database |

**Thuật ngữ cần nhớ**

- thu thập thay đổi dữ liệu → **change data capture (CDC)**
- thời gian chết → **downtime**
- đuổi kịp → **catch up**
- chuyển sang hệ mới → **cut over**
- luồng thay đổi → **change stream**

---

## Phần 8 — Chẩn đoán khi thêm bản sao mà hệ thống không nhanh hơn

**Tiếng Việt**

Một câu hỏi thách đố hay gặp là chúng ta đã thêm bản sao nhưng độ trễ không cải thiện, và thỉnh thoảng hệ thống còn trả về dữ liệu sai. Việc đầu tiên chúng ta kiểm tra là độ trễ sao chép, vì nếu bản sao đang chậm nhiều giây thì mọi lần đọc trên đó đều cho ra dữ liệu cũ. Việc thứ hai là chúng ta xem những lần đọc nào đang bị định tuyến nhầm sang bản sao trong khi chúng cần dữ liệu mới nhất. Việc thứ ba là chúng ta đối chiếu từng nhóm truy vấn với yêu cầu nhất quán của nghiệp vụ tương ứng. Nếu độ trễ vẫn không cải thiện, rất có thể điểm nghẽn thật của chúng ta nằm ở đường ghi chứ nó không nằm ở đường đọc. Trong trường hợp đó, thêm bao nhiêu bản sao cũng vô ích, và chúng ta phải nghĩ tới phân mảnh dữ liệu hoặc tối ưu chính các lệnh ghi. Đây là chỗ mà một người senior thể hiện rõ, vì câu trả lời đúng là chỉ ra rằng công cụ đang được dùng sai bài toán.

**English (bám cấu trúc tiếng Việt)**

A common challenge question is that we have added replicas but the latency has not improved, and now and then the system even returns wrong data. The first thing we check is the replication lag, because if the replica is running several seconds behind then every read on it gives back old data. The second thing is that we look at which reads are being routed to the replica by mistake while they need the newest data. The third thing is that we match each group of queries against the consistency requirement of the corresponding business flow. If the latency still does not improve, it is very likely that our real bottleneck is on the write path and it is not on the read path. In that case, adding any number of replicas is useless, and we have to think about sharding the data or optimising the writes themselves. This is where a senior person clearly stands out, because the right answer is to point out that the tool is being used on the wrong problem.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| một câu hỏi thách đố hay gặp | a common challenge question |
| thỉnh thoảng hệ thống còn trả về dữ liệu sai | now and then the system even returns wrong data |
| nếu bản sao đang chậm nhiều giây | if the replica is running several seconds behind |
| đang bị định tuyến nhầm sang bản sao | are being routed to the replica by mistake |
| trong khi chúng cần dữ liệu mới nhất | while they need the newest data |
| đối chiếu từng nhóm truy vấn với | match each group of queries against |
| của nghiệp vụ tương ứng | of the corresponding business flow |
| thêm bao nhiêu bản sao cũng vô ích | adding any number of replicas is useless |
| đây là chỗ mà một người senior thể hiện rõ | this is where a senior person clearly stands out |
| công cụ đang được dùng sai bài toán | the tool is being used on the wrong problem |

**Thuật ngữ cần nhớ**

- định tuyến nhầm → **routed by mistake**
- nghiệp vụ tương ứng → **the corresponding business flow**
- phân mảnh dữ liệu → **sharding**
- vô ích → **useless**
- thể hiện rõ | nổi bật → **stand out**

---

## Mô hình ghi nhớ

**Tiếng Việt**

Bản sao giải bài toán đọc và bài toán sẵn sàng cao, nhưng nó không giải bài toán ghi. Vì luôn có độ trễ, chúng ta phải đưa những lần đọc ngay sau khi ghi về máy chính, và sao chép bất đồng bộ là việc chúng ta đổi an toàn để lấy tốc độ.

**English (bám cấu trúc tiếng Việt)**

Replicas solve the read problem and the high availability problem, but they do not solve the write problem. Because there is always lag, we have to send the reads that come right after a write to the primary, and asynchronous replication is us trading safety for speed.

---

## Bảng thuật ngữ tổng hợp

| Tiếng Việt | Tiếng Anh | Ghi chú phát âm |
|---|---|---|
| sao chép dữ liệu | replication | re-pli-**CAY**-shơn |
| bản sao | replica | **REP**-li-cơ — trọng âm 1 |
| máy chính | the primary | Anh /ˈpraɪməri/ — "**PRAI**-mơ-ri" |
| mở rộng khả năng đọc | scale reads | |
| tính sẵn sàng cao | high availability | ơ-vei-lơ-**BI**-lơ-ti |
| nâng lên làm máy chính | promote | prơ-**MOUT** — trọng âm 2 |
| sao lưu | backup | |
| điểm nghẽn | bottleneck | |
| sao chép đồng bộ | synchronous replication | **SIN**-crơ-nơs — "ch" đọc /k/ |
| sao chép bất đồng bộ | asynchronous replication | ei-**SIN**-crơ-nơs |
| xác nhận đã nhận | acknowledge | ơk-**NO**-lij — chữ **w** câm |
| một vòng đi và về | a round trip | |
| chuyển đổi khi sự cố | failover | |
| dữ liệu sống còn | critical data | **CRI**-ti-cơl |
| độ trễ sao chép | replication lag | |
| dữ liệu cũ, ôi | stale data | /steɪl/ — "stêi-l" |
| đọc thấy thứ mình vừa ghi | read-your-own-writes | *write* — chữ **w** câm |
| tái hiện lỗi | reproduce a bug | ri-prơ-**DIUS** |
| vi phạm | violate | **VAI**-ơ-leit — trọng âm 1 |
| định tuyến | route | Anh /ruːt/ — "rut"; người Mỹ đọc /raʊt/ |
| phiên ghim vào một node | sticky session | |
| vị trí trong log | log position (LSN) | |
| phát lại | replay | ri-**PLAY** |
| yêu cầu nhất quán | consistency requirement | ri-**KWAI**-ơ-mơnt |
| não phân đôi | split-brain | *split* — cụm /spl-/ đọc liền |
| phân kỳ | diverge | dai-**VERJ** — đuôi /dʒ/ |
| gộp lại | merge back | /mɜːdʒ/ |
| đa số | majority | mơ-**JO**-rơ-ti — trọng âm 2 |
| nhóm đủ túc số | quorum | **KWO**-rơm — trọng âm 1 |
| cô lập node hỏng | fencing | |
| dịch vụ được quản lý sẵn | a managed service | **MA**-nijd |
| sao chép vật lý | physical replication | *physical* — "ph" đọc /f/ |
| sao chép logic | logical replication | **LO**-ji-cơl |
| truyền luồng | stream | cụm /str-/ đọc liền |
| cụm máy chủ | cluster | **CLUS**-tơ |
| đăng ký nhận thay đổi | subscribe | sơb-**SCRAIB** |
| thay đổi cấu trúc bảng | schema change | *schema* /ˈskiːmə/ — "**SKII**-mờ" |
| thu thập thay đổi dữ liệu | change data capture (CDC) | **CAP**-chơ |
| thời gian chết | downtime | |
| đuổi kịp | catch up | *catch* — đuôi /-tʃ/ phải bật |
| chuyển sang hệ mới | cut over | |
| bộ trung chuyển thông điệp | message broker | **BROU**-cơ |
| kho phân tích | analytical warehouse | *warehouse* — "**WEA**-hauz", có bật **h** |
| phân mảnh dữ liệu | sharding | *shard* /ʃɑːd/ — âm "sh" đầu |
| thể hiện rõ, nổi bật | stand out | |
| rủi ro | risk | đuôi /-sk/ phải bật ra |

---

## Bài nói

> **Cách luyện:** bật ghi âm, nói **60–90 giây** cho mỗi đề, **không nhìn tài liệu**. Mỗi bài nói phải dùng **ít nhất 6 thuật ngữ** trong bảng trên. Nghe lại bản ghi một lần, đánh dấu chỗ ngập ngừng, rồi nói lại đề đó lần thứ hai.

1. Explain to a junior engineer what replication gives you and what it does not give you.

2. A colleague says: "Writes are slow, so let us add two more read replicas." Explain why you would push back and what you would look at instead.

3. A user updates their profile, reloads the page, and sees the old values. Walk your team through the cause and through two different fixes.

4. Describe what happens during a failover, and describe how split-brain can occur if nothing protects against it.

5. When would you choose synchronous replication over asynchronous, and what do you accept by making that choice?

6. Your team needs to upgrade PostgreSQL across a major version with almost no downtime. Explain how you would use logical replication to do it.

7. You have added replicas, the latency has not improved, and some pages show wrong data. Talk an interviewer through your diagnosis, step by step.
