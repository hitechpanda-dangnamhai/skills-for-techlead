# Bài 1 — Vì sao cache · Các tầng cache · Khi nào KHÔNG cache
### Bản song ngữ: tiếng Anh bám cấu trúc tiếng Việt

> **Cách dùng bài này:** đọc đoạn tiếng Việt trước → tự dịch trong đầu → so với đoạn tiếng Anh bên dưới → **đọc to bản tiếng Anh hai lần**: lần một đọc chậm cho đúng, lần hai đọc trôi như đang nói với người phỏng vấn.

---

## Phần 1 — Cache là gì và tốc độ đến từ đâu

**Tiếng Việt**

Cache là một bản sao của dữ liệu được đặt ở nơi truy cập nhanh hơn nguồn gốc, để lần sau chúng ta không phải tính lại hoặc đọc lại từ chỗ chậm. Chúng ta có thể hình dung như thế này: bạn không chạy xuống thư viện mỗi lần cần một con số, bạn ghi con số đó lên một tờ giấy dán cạnh màn hình. Tờ giấy đó chính là cache. Nó nhanh vì nó ở gần và vì nó đã sẵn ở đó. Tốc độ của cache đến từ hai nguồn. Nguồn thứ nhất là locality, gồm temporal locality nghĩa là thứ vừa dùng thì dễ được dùng lại, và spatial locality nghĩa là thứ nằm gần thứ vừa dùng thì dễ được dùng tiếp. Nguồn thứ hai là chúng ta tránh được I/O và tính toán đắt, vì RAM nhanh hơn disk, DB, network hoặc một lời gọi LLM nhiều bậc độ lớn.

Đổi lại, chúng ta phải trả ba thứ. Chúng ta trả thêm bộ nhớ, chúng ta chấp nhận rủi ro stale nghĩa là bản sao lệch so với nguồn, và chúng ta thêm một thành phần nữa phải vận hành. Nếu chúng ta quên rằng cache là một cuộc đánh cược vào locality, chúng ta sẽ cache những thứ không ai đọc lại, và khi đó chúng ta chỉ tốn RAM mà không nhanh hơn. Hậu quả nếu làm sai còn nặng hơn thế: một bản sao cũ được trả về cho người dùng, và lỗi này rất khó tái hiện vì nó phụ thuộc vào thời điểm. Vì vậy chúng ta nên coi cache là một quyết định kiến trúc, không phải một tính năng bật cho chắc.

**English (bám cấu trúc tiếng Việt)**

A cache is a copy of data placed somewhere faster to access than the original source, so that next time we do not have to recompute it or read it again from the slow place. We can picture it like this: you do not run down to the library every time you need a number, you write that number on a piece of paper stuck next to your screen. That piece of paper is exactly the cache. It is fast because it is near and because it is already there. The speed of a cache comes from two sources. The first source is locality, which includes temporal locality, meaning that what we have just used is likely to be used again, and spatial locality, meaning that what sits near what we have just used is likely to be used next. The second source is that we avoid expensive I/O and computation, because RAM is faster than disk, the database, the network or an LLM call by many orders of magnitude.

In exchange, we have to pay three things. We pay extra memory, we accept the risk of stale data, which means the copy drifts away from the source, and we add one more component that we have to operate. If we forget that a cache is a bet on locality, we will cache things that nobody reads again, and in that case we only burn RAM without getting any faster. The consequence of getting this wrong is even heavier than that: an old copy is returned to the user, and this bug is very hard to reproduce because it depends on timing. Therefore we should treat a cache as an architectural decision, not as a feature we switch on just to be safe.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| đặt ở nơi truy cập nhanh hơn nguồn gốc | placed somewhere faster to access than the original source |
| chúng ta không phải tính lại | we do not have to recompute it |
| Chúng ta có thể hình dung như thế này | We can picture it like this |
| nó đã sẵn ở đó | it is already there |
| thứ vừa dùng thì dễ được dùng lại | what we have just used is likely to be used again |
| nhiều bậc độ lớn | by many orders of magnitude |
| Đổi lại, chúng ta phải trả | In exchange, we have to pay |
| bản sao lệch so với nguồn | the copy drifts away from the source |
| một cuộc đánh cược vào locality | a bet on locality |
| chỉ tốn RAM mà không nhanh hơn | only burn RAM without getting any faster |
| rất khó tái hiện | very hard to reproduce |
| một tính năng bật cho chắc | a feature we switch on just to be safe |

**Thuật ngữ cần nhớ**

- bản sao → **a copy**
- nguồn gốc, nguồn sự thật → **the original source** / **the source of truth**
- dữ liệu cũ, lệch nguồn → **stale data**
- bậc độ lớn → **order of magnitude**
- đánh đổi → **trade-off**

---

## Phần 2 — Các tầng cache trong một request

**Tiếng Việt**

Một request đi từ trình duyệt xuống tới DB sẽ xuyên qua nhiều tầng cache, và mỗi tầng đổi tốc độ lấy phạm vi chia sẻ và độ tươi. Tầng đầu tiên là browser cache, nó riêng cho từng người dùng, nhanh nhất, nhưng cũng dễ stale nhất vì chúng ta gần như không xoá được nó từ xa. Tầng thứ hai là CDN hoặc edge cache, nó nằm gần người dùng về mặt địa lý, vì vậy nó cắt được độ trễ giữa các vùng miền. Tầng thứ ba là reverse proxy như Nginx hoặc Varnish, nó cache nguyên response trước khi request chạm vào app. Tầng thứ tư là cache in-process của app, chúng ta gọi là L1, nó nằm ngay trong tiến trình nên không tốn một vòng mạng nào. Tầng thứ năm là distributed cache như Redis hoặc Memcached, chúng ta gọi là L2, nó được chia sẻ giữa mọi instance. Tầng cuối cùng là buffer pool của chính DB, vì DB cũng giữ page trong RAM.

Khi trả lời phỏng vấn, chúng ta nên đọc đúng thứ tự sáu tầng này, vì thứ tự chính là mạch của câu trả lời. Chúng ta cũng nên nói rõ rằng tầng nào càng gần người dùng thì độ tươi càng khó kiểm soát. Nếu chúng ta đặt một thứ hay đổi lên browser cache với thời gian sống dài, chúng ta sẽ không có cách nào thu hồi nó. Hậu quả là người dùng nhìn thấy giá cũ hoặc giao diện cũ trong nhiều ngày, trong khi log phía server hoàn toàn sạch. Vì vậy chúng ta thường đặt thứ hay đổi ở tầng gần app, và chỉ đẩy ra CDN những thứ ổn định hoặc những thứ có phiên bản trong tên tệp.

**English (bám cấu trúc tiếng Việt)**

A request that travels from the browser down to the database will pass through many cache layers, and each layer trades speed for sharing scope and freshness. The first layer is the browser cache, which is private to each user, the fastest, but also the easiest to go stale, because we can almost never delete it remotely. The second layer is the CDN or edge cache, which sits near the user geographically, therefore it cuts the latency between regions. The third layer is a reverse proxy such as Nginx or Varnish, which caches the whole response before the request touches the app. The fourth layer is the in-process cache of the app, which we call L1, and it sits right inside the process, so it does not cost a single network round trip. The fifth layer is a distributed cache such as Redis or Memcached, which we call L2, and it is shared between all instances. The last layer is the buffer pool of the database itself, because the database also keeps pages in RAM.

When we answer in an interview, we should recite these six layers in the right order, because the order is exactly the thread of the answer. We should also say clearly that the closer a layer sits to the user, the harder its freshness is to control. If we put something that changes often into the browser cache with a long lifetime, we will have no way to take it back. The consequence is that the user sees an old price or an old interface for many days, while the logs on the server side are completely clean. Therefore we usually put things that change often in the layer near the app, and we only push out to the CDN the things that are stable or the things that have a version in the file name.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| xuyên qua nhiều tầng cache | pass through many cache layers |
| đổi tốc độ lấy phạm vi chia sẻ và độ tươi | trades speed for sharing scope and freshness |
| riêng cho từng người dùng | private to each user |
| dễ stale nhất | the easiest to go stale |
| cắt được độ trễ giữa các vùng miền | cuts the latency between regions |
| trước khi request chạm vào app | before the request touches the app |
| không tốn một vòng mạng nào | does not cost a single network round trip |
| thứ tự chính là mạch của câu trả lời | the order is exactly the thread of the answer |
| tầng nào càng gần người dùng thì … càng khó | the closer a layer sits to the user, the harder … |
| không có cách nào thu hồi nó | we will have no way to take it back |
| log phía server hoàn toàn sạch | the logs on the server side are completely clean |
| có phiên bản trong tên tệp | have a version in the file name |

**Thuật ngữ cần nhớ**

- tầng cache → **cache layer** / **cache tier**
- độ tươi của dữ liệu → **freshness**
- một vòng mạng → **a network round trip**
- thời gian sống → **time to live (TTL)**
- thu hồi, làm mất hiệu lực → **invalidate**

---

## Phần 3 — Local cache và distributed cache

**Tiếng Việt**

Local cache nằm ngay trong tiến trình của ứng dụng, còn distributed cache nằm ở một service riêng mà mọi instance cùng gọi tới. Local cache cực nhanh vì nó không tốn một network hop nào, nhưng mỗi instance giữ một bản riêng nên các bản đó dễ lệch nhau. Distributed cache chậm hơn một chút vì mỗi lần đọc tốn một vòng mạng, nhưng đổi lại mọi instance nhìn thấy cùng một dữ liệu. Việc xoá cache đồng loạt cũng khác nhau rõ rệt giữa hai bên. Với local cache, chúng ta phải báo cho từng node, vì vậy việc xoá là khó và dễ sót. Với distributed cache, chúng ta chỉ xoá ở một nơi, vì vậy việc xoá là dễ và chắc chắn hơn. Đổi lại, distributed cache thêm một service nữa mà chúng ta phải làm sẵn sàng cao.

Nếu chúng ta chọn sai, hậu quả xuất hiện đúng lúc hệ thống đang chạy tốt nhất. Ví dụ, chúng ta để cấu hình phân quyền trong local cache với thời gian sống mười phút, rồi chúng ta thu hồi quyền của một người dùng. Node A đã xoá bản của nó, nhưng node B vẫn giữ bản cũ, vì vậy người dùng đó vẫn vào được trong vài phút, tuỳ theo load balancer đẩy họ về máy nào. Lỗi này gần như không tái hiện được trên máy của lập trình viên, vì trên máy đó chỉ có một tiến trình. Vì vậy nguyên tắc của chúng ta rất ngắn: thứ gì cần đúng trên toàn cụm thì phải nằm ở L2, không nằm ở L1.

**English (bám cấu trúc tiếng Việt)**

A local cache sits right inside the process of the application, while a distributed cache sits in a separate service that all instances call to. A local cache is extremely fast because it does not cost a single network hop, but each instance keeps its own copy, so those copies easily drift apart. A distributed cache is a little slower because each read costs one network round trip, but in exchange all instances see the same data. Clearing the cache everywhere at once is also clearly different between the two sides. With a local cache, we have to tell every node, therefore the clearing is hard and easy to miss. With a distributed cache, we only delete in one place, therefore the clearing is easy and more reliable. In exchange, a distributed cache adds one more service that we have to make highly available.

If we choose wrongly, the consequence shows up exactly when the system is running at its best. For example, we keep the permission configuration in a local cache with a time to live of ten minutes, and then we revoke the rights of one user. Node A has deleted its copy, but node B still keeps the old copy, therefore that user can still get in for a few minutes, depending on which machine the load balancer sends them to. This bug is almost impossible to reproduce on a developer's machine, because on that machine there is only one process. Therefore our rule is very short: whatever needs to be correct across the whole cluster must sit in L2, not in L1.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| nằm ngay trong tiến trình của ứng dụng | sits right inside the process of the application |
| mọi instance cùng gọi tới | that all instances call to |
| không tốn một network hop nào | does not cost a single network hop |
| các bản đó dễ lệch nhau | those copies easily drift apart |
| xoá cache đồng loạt | clearing the cache everywhere at once |
| khó và dễ sót | hard and easy to miss |
| chúng ta phải làm sẵn sàng cao | we have to make highly available |
| hậu quả xuất hiện đúng lúc | the consequence shows up exactly when |
| thu hồi quyền của một người dùng | revoke the rights of one user |
| tuỳ theo load balancer đẩy họ về máy nào | depending on which machine the load balancer sends them to |
| gần như không tái hiện được | almost impossible to reproduce |
| thứ gì cần đúng trên toàn cụm | whatever needs to be correct across the whole cluster |

**Thuật ngữ cần nhớ**

- cache cục bộ, trong tiến trình → **local (in-process) cache**
- cache phân tán → **distributed cache**
- sẵn sàng cao → **high availability**
- lệch nhau, tách nhau ra → **drift apart** / **diverge**
- toàn cụm → **across the whole cluster**

---

## Phần 4 — Mô hình nhiều tầng L1 cộng L2

**Tiếng Việt**

Trong thực tế, hầu hết hệ thống lớn dùng cả hai tầng cùng lúc, và chúng ta gọi đó là multi-tier. L1 là local cache dùng cho hot key, nó cắt độ trễ và giảm tải cho L2. L2 là Redis, nó đóng vai trò nguồn chia sẻ cho mọi instance. Luồng đọc rất đơn giản: chúng ta hỏi L1 trước, nếu miss thì chúng ta hỏi L2, nếu vẫn miss thì chúng ta đọc DB rồi điền ngược lên cả hai tầng. Rủi ro của mô hình này là L1 ở mỗi node có thể stale theo cách khác nhau khi L2 hoặc DB thay đổi. Chúng ta xử lý rủi ro đó bằng ba cách: đặt TTL rất ngắn cho L1, phát broadcast invalidate qua pub/sub, hoặc chấp nhận một mức lệch nhỏ nếu dữ liệu không nhạy cảm.

Con số cụ thể làm câu trả lời của chúng ta đáng tin hơn nhiều. Ví dụ, chúng ta đặt L1 với TTL năm giây và sức chứa mười nghìn key, còn L2 với TTL năm phút. Với cấu hình đó, mức lệch tối đa giữa hai node là năm giây, và đó là con số chúng ta có thể đem ra thoả thuận với bên nghiệp vụ. Nếu chúng ta kéo TTL của L1 lên mười phút cho nhanh hơn, chúng ta đã biến một tối ưu nhỏ thành một nguồn bug kéo dài. Vì vậy quy tắc là: L1 luôn phải có TTL ngắn hơn L2 nhiều lần.

**English (bám cấu trúc tiếng Việt)**

In practice, most large systems use both layers at the same time, and we call that multi-tier. L1 is the local cache used for hot keys, it cuts the latency and takes load off L2. L2 is Redis, it plays the role of the shared source for all instances. The read flow is very simple: we ask L1 first, if it misses then we ask L2, if it still misses then we read the database and fill the value back up into both layers. The risk of this model is that L1 on each node can go stale in different ways when L2 or the database changes. We handle that risk in three ways: set a very short TTL for L1, broadcast an invalidation over pub/sub, or accept a small amount of drift if the data is not sensitive.

Concrete numbers make our answer far more credible. For example, we set L1 with a TTL of five seconds and a capacity of ten thousand keys, and L2 with a TTL of five minutes. With that configuration, the maximum drift between two nodes is five seconds, and that is a number we can put on the table with the business side. If we push the TTL of L1 up to ten minutes to be faster, we have turned a small optimisation into a long-running source of bugs. Therefore the rule is: L1 must always have a TTL many times shorter than L2.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| dùng cả hai tầng cùng lúc | use both layers at the same time |
| giảm tải cho L2 | takes load off L2 |
| đóng vai trò nguồn chia sẻ | plays the role of the shared source |
| điền ngược lên cả hai tầng | fill the value back up into both layers |
| stale theo cách khác nhau | go stale in different ways |
| phát broadcast invalidate qua pub/sub | broadcast an invalidation over pub/sub |
| chấp nhận một mức lệch nhỏ | accept a small amount of drift |
| làm câu trả lời đáng tin hơn nhiều | make our answer far more credible |
| sức chứa mười nghìn key | a capacity of ten thousand keys |
| đem ra thoả thuận với bên nghiệp vụ | put on the table with the business side |
| một nguồn bug kéo dài | a long-running source of bugs |
| ngắn hơn … nhiều lần | many times shorter than … |

**Thuật ngữ cần nhớ**

- nhiều tầng → **multi-tier**
- key nóng, bị gọi rất nhiều → **hot key**
- giảm tải → **take load off** / **offload**
- phát tán lệnh xoá cache → **broadcast an invalidation**
- sức chứa → **capacity**

---

## Phần 5 — Khi nào chúng ta KHÔNG nên cache

**Tiếng Việt**

Phần mà nhiều người bỏ qua là câu hỏi ngược: khi nào chúng ta không nên cache. Trường hợp thứ nhất là dữ liệu đổi liên tục hoặc chỉ được đọc một lần, vì khi đó tỉ lệ hit rất thấp và cache chỉ tốn RAM rồi sinh thêm stale. Trường hợp thứ hai là những đường đi cần tính nhất quán mạnh tuyệt đối, ví dụ số dư tiền, tồn kho và hạn mức, vì đọc một giá trị cũ ở đó nghĩa là ra một quyết định sai. Trường hợp thứ ba là hệ thống ghi nhiều đọc ít, vì chúng ta sẽ phải xoá cache liên tục mà hiếm khi hit. Trường hợp thứ tư là dữ liệu rẻ để tính lại, vì cache không cứu được gì mà lại thêm một chỗ có thể sai. Nguyên tắc chung của chúng ta là chỉ cache khi đo được lợi ích. Cache thêm cho chắc là cách phổ biến nhất để đẻ ra bug stale.

Cách nói này rất mạnh trong phỏng vấn, vì nó cho thấy chúng ta nghĩ về rủi ro chứ không chỉ nghĩ về tốc độ. Khi ai đó đề nghị cache tồn kho để trang sản phẩm nhanh hơn, chúng ta nên tách bài toán ra làm hai phần. Phần công khai và ổn định như tên sản phẩm và mô tả thì chúng ta cache mạnh. Phần cần đúng tuyệt đối như số lượng còn lại thì chúng ta đọc thẳng DB tại bước thanh toán. Nhờ đó chúng ta vẫn nhanh ở chỗ người dùng cảm nhận được, mà không bán mất món hàng chúng ta không còn.

**English (bám cấu trúc tiếng Việt)**

The part that many people skip is the reverse question: when should we not cache. The first case is data that changes constantly or is read only once, because then the hit ratio is very low and the cache only burns RAM and then produces more stale data. The second case is paths that need absolutely strong consistency, for example money balances, stock levels and credit limits, because reading an old value there means making a wrong decision. The third case is a write-heavy and read-light system, because we will have to delete the cache constantly while we rarely hit it. The fourth case is data that is cheap to recompute, because the cache saves nothing and yet adds one more place that can be wrong. Our general rule is to cache only when we can measure the benefit. Caching just to be safe is the most common way to breed stale bugs.

This way of speaking is very strong in an interview, because it shows that we think about risk and not only about speed. When someone proposes caching the stock level to make the product page faster, we should split the problem into two parts. The public and stable part, such as the product name and the description, we cache aggressively. The part that must be exactly correct, such as the remaining quantity, we read straight from the database at the checkout step. Thanks to that, we are still fast where the user can feel it, without selling an item that we no longer have.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| câu hỏi ngược | the reverse question |
| tỉ lệ hit rất thấp | the hit ratio is very low |
| sinh thêm stale | produces more stale data |
| tính nhất quán mạnh tuyệt đối | absolutely strong consistency |
| số dư tiền, tồn kho và hạn mức | money balances, stock levels and credit limits |
| ghi nhiều đọc ít | write-heavy and read-light |
| rẻ để tính lại | cheap to recompute |
| thêm một chỗ có thể sai | adds one more place that can be wrong |
| chỉ cache khi đo được lợi ích | cache only when we can measure the benefit |
| cache thêm cho chắc | caching just to be safe |
| đẻ ra bug stale | breed stale bugs |
| tách bài toán ra làm hai phần | split the problem into two parts |
| bán mất món hàng chúng ta không còn | selling an item that we no longer have |

**Thuật ngữ cần nhớ**

- tính nhất quán mạnh → **strong consistency**
- tỉ lệ hit → **hit ratio**
- ghi nhiều → **write-heavy**
- tính lại → **recompute**
- tồn kho → **stock level** / **inventory**

---

## Phần 6 — Đo xem cache có đáng hay không

**Tiếng Việt**

Để biết cache có đáng hay không, chúng ta phải đo bốn thứ. Chúng ta đo tỉ lệ hit, đo độ trễ ở phân vị năm mươi và phân vị chín mươi chín, đo mức giảm tải trên nguồn nghĩa là DB giảm được bao nhiêu truy vấn mỗi giây, và đo chi phí trên mỗi request. Cái bẫy lớn nhất ở đây là tỉ lệ hit cao chưa chắc đã tốt. Nếu chúng ta chỉ cache những thứ rẻ và ít được gọi, con số hit ratio sẽ rất đẹp mà chẳng bảo vệ được gì cả. Vì vậy chúng ta phải gắn cache vào đúng thứ chúng ta muốn bảo vệ, thường là DB khỏi sập và độ trễ đuôi ở p99. Chúng ta không tối ưu hit ratio một cách mù quáng.

Có một tình huống thách đố hay được hỏi: team thêm Redis trước DB nhưng p99 không cải thiện chút nào. Lý do thứ nhất có thể là tỉ lệ hit thấp, vì dữ liệu đổi liên tục hoặc chỉ được đọc một lần. Lý do thứ hai có thể là nút thắt không nằm ở DB mà nằm ở phần tính toán, phần chuyển đổi dữ liệu hoặc phần mạng. Lý do thứ ba có thể là ở đuôi phân bố, các request miss vẫn đập thẳng vào DB, và chính chúng tạo ra p99. Kết luận của chúng ta rất ngắn: phải đo trước khi thêm một tầng.

**English (bám cấu trúc tiếng Việt)**

To know whether a cache is worth it or not, we have to measure four things. We measure the hit ratio, we measure the latency at the fiftieth percentile and the ninety-ninth percentile, we measure how much load is taken off the source, meaning how many queries per second the database saves, and we measure the cost per request. The biggest trap here is that a high hit ratio is not necessarily good. If we only cache things that are cheap and rarely called, the hit ratio number will look very nice while it protects nothing at all. Therefore we have to tie the cache to the exact thing we want to protect, usually the database from falling over and the tail latency at p99. We do not optimise the hit ratio blindly.

There is a challenge scenario that gets asked often: the team adds Redis in front of the database but p99 does not improve at all. The first reason may be a low hit ratio, because the data changes constantly or is read only once. The second reason may be that the bottleneck does not sit in the database but sits in the computation, in the data serialisation or in the network. The third reason may be that at the tail of the distribution, the requests that miss still hit the database directly, and it is exactly those that create p99. Our conclusion is very short: we must measure before we add a layer.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| cache có đáng hay không | whether a cache is worth it or not |
| phân vị chín mươi chín | the ninety-ninth percentile |
| mức giảm tải trên nguồn | how much load is taken off the source |
| truy vấn mỗi giây | queries per second |
| chi phí trên mỗi request | the cost per request |
| chưa chắc đã tốt | is not necessarily good |
| chẳng bảo vệ được gì cả | protects nothing at all |
| DB khỏi sập | the database from falling over |
| độ trễ đuôi | the tail latency |
| một cách mù quáng | blindly |
| nút thắt không nằm ở DB | the bottleneck does not sit in the database |
| ở đuôi phân bố | at the tail of the distribution |
| đập thẳng vào DB | hit the database directly |

**Thuật ngữ cần nhớ**

- tỉ lệ hit → **hit ratio**
- độ trễ đuôi → **tail latency**
- nút thắt cổ chai → **bottleneck**
- truy vấn mỗi giây → **queries per second (QPS)**
- chuyển đổi dữ liệu để truyền → **serialisation**

---

## Phần 7 — Cache so với tính sẵn và materialized view

**Tiếng Việt**

Cache và tính sẵn giải cùng một bài toán nhưng theo hai hướng ngược nhau. Cache là lười, nghĩa là nó chỉ tính khi có người hỏi, vì vậy nó có thể miss và lần miss đó phải trả giá bằng độ trễ. Materialized view là chủ động, nghĩa là chúng ta tính sẵn từ trước, vì vậy dữ liệu luôn có mặt nhưng chúng ta tốn chi phí ghi và chi phí làm mới. Chúng ta chọn giữa hai cách dựa trên hai yếu tố: độ tươi mà nghiệp vụ yêu cầu, và mẫu đọc của hệ thống. Nếu lượng truy vấn phân tán rộng và khó đoán, cache thường hợp hơn vì chúng ta không thể tính sẵn mọi thứ. Nếu chỉ có vài báo cáo nặng được xem đi xem lại, materialized view thường hợp hơn vì chúng ta trả tiền một lần cho nhiều lượt đọc.

Trong phỏng vấn, chúng ta nên nói rõ chi phí của mỗi hướng thay vì chỉ gọi tên chúng. Với cache, chi phí nằm ở lần miss đầu tiên và ở cửa sổ stale. Với materialized view, chi phí nằm ở việc làm mới, và nếu chúng ta làm mới quá thưa thì dữ liệu cũng cũ y như cache. Nhờ cách trình bày đó, người phỏng vấn thấy rằng chúng ta hiểu cả hai đều là đánh đổi, chứ không phải một bên luôn thắng.

**English (bám cấu trúc tiếng Việt)**

A cache and precomputation solve the same problem but in two opposite directions. A cache is lazy, meaning that it only computes when somebody asks, therefore it can miss and that miss has to be paid for in latency. A materialized view is eager, meaning that we compute it in advance, therefore the data is always there but we pay a write cost and a refresh cost. We choose between the two approaches based on two factors: the freshness that the business requires, and the read pattern of the system. If the volume of queries is widely spread and hard to predict, a cache usually fits better because we cannot precompute everything. If there are only a few heavy reports that are looked at over and over, a materialized view usually fits better because we pay once for many reads.

In an interview, we should state the cost of each direction clearly instead of only naming them. With a cache, the cost sits in the first miss and in the stale window. With a materialized view, the cost sits in the refreshing, and if we refresh too rarely then the data is just as old as in a cache. Thanks to that way of presenting it, the interviewer sees that we understand both are trade-offs, and not that one side always wins.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| theo hai hướng ngược nhau | in two opposite directions |
| cache là lười | a cache is lazy |
| phải trả giá bằng độ trễ | has to be paid for in latency |
| chúng ta tính sẵn từ trước | we compute it in advance |
| chi phí làm mới | a refresh cost |
| mẫu đọc của hệ thống | the read pattern of the system |
| phân tán rộng và khó đoán | widely spread and hard to predict |
| được xem đi xem lại | are looked at over and over |
| trả tiền một lần cho nhiều lượt đọc | pay once for many reads |
| cửa sổ stale | the stale window |
| làm mới quá thưa | refresh too rarely |
| cũ y như cache | just as old as in a cache |
| không phải một bên luôn thắng | not that one side always wins |

**Thuật ngữ cần nhớ**

- tính sẵn → **precompute** / **precomputation**
- lười, theo nhu cầu → **lazy**
- chủ động, tính trước → **eager**
- làm mới → **refresh**
- cửa sổ dữ liệu cũ → **the stale window**

---

## Phần 8 — Góc nhìn tech lead: chỗ AI hay chọn sai

**Tiếng Việt**

Phần cuối là góc nhìn của một tech lead, vì đây là chỗ chúng ta tạo ra giá trị khác với một lập trình viên bình thường. Công cụ AI thường mặc định rằng thêm cache là nhanh hơn, vì vậy nó bê Redis vào mọi đường đi, kể cả tồn kho và tiền. Khi chúng ta review một đoạn code như vậy, câu hỏi đầu tiên chúng ta phải đặt là đường đi này có chịu được dữ liệu cũ hay không. Nếu câu trả lời là không, chúng ta bỏ cache ở đó, hoặc chúng ta tách phần công khai ra khỏi phần cần đúng tuyệt đối. Chúng ta cũng phải kiểm tra key đã chứa định danh người dùng hay chưa, vì dùng chung một key cho mọi người là cách nhanh nhất để rò dữ liệu giữa các tài khoản. Cuối cùng, chúng ta hỏi TTL được đặt bằng bao nhiêu và ai sẽ xoá key khi dữ liệu thay đổi.

Câu thần chú của cả mục này rất đáng nhớ: chúng ta không học thuộc để gõ lệnh Redis, chúng ta hiểu để chỉ huy và để kiểm tra. Chỉ huy nghĩa là chọn đúng chiến lược và đúng chính sách cho từng đường đi. Kiểm tra nghĩa là bắt được lúc AI hoặc đồng đội chọn sai, trước khi thứ đó lên production. Nếu chúng ta chỉ nhớ cú pháp, chúng ta sẽ bị thay thế bởi công cụ sinh code. Nếu chúng ta nắm được đánh đổi, chúng ta là người quyết định công cụ đó được dùng ở đâu.

**English (bám cấu trúc tiếng Việt)**

The last part is the point of view of a tech lead, because this is where we create value that is different from an ordinary developer. AI tools usually assume by default that adding a cache is faster, therefore they drop Redis into every path, including stock levels and money. When we review a piece of code like that, the first question we have to ask is whether this path can tolerate stale data or not. If the answer is no, we remove the cache there, or we separate the public part from the part that must be exactly correct. We also have to check whether the key already contains the user identifier or not, because sharing one key for everybody is the fastest way to leak data between accounts. Finally, we ask what the TTL is set to and who will delete the key when the data changes.

The mantra of this whole topic is well worth remembering: we do not memorise in order to type Redis commands, we understand in order to direct and to verify. To direct means to choose the right strategy and the right policy for each path. To verify means to catch the moment when the AI or a teammate chooses wrongly, before that thing goes to production. If we only remember the syntax, we will be replaced by code-generation tools. If we hold the trade-offs, we are the person who decides where that tool is used.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| chỗ chúng ta tạo ra giá trị khác với | where we create value that is different from |
| bê Redis vào mọi đường đi | drop Redis into every path |
| có chịu được dữ liệu cũ hay không | can tolerate stale data or not |
| tách phần công khai ra khỏi phần cần đúng tuyệt đối | separate the public part from the part that must be exactly correct |
| key đã chứa định danh người dùng hay chưa | whether the key already contains the user identifier or not |
| rò dữ liệu giữa các tài khoản | leak data between accounts |
| ai sẽ xoá key khi dữ liệu thay đổi | who will delete the key when the data changes |
| câu thần chú | the mantra |
| học thuộc để gõ lệnh | memorise in order to type commands |
| hiểu để chỉ huy và để kiểm tra | understand in order to direct and to verify |
| trước khi thứ đó lên production | before that thing goes to production |
| bị thay thế bởi công cụ sinh code | be replaced by code-generation tools |

**Thuật ngữ cần nhớ**

- đường đi, luồng xử lý → **path**
- chịu được, chấp nhận được → **tolerate**
- rò rỉ dữ liệu → **data leak**
- chính sách → **policy**
- đưa lên production → **ship to production**

---

## Mô hình ghi nhớ

**Tiếng Việt**

Cache là một bản sao ở gần và nhanh, nó đặt cược vào locality, và chúng ta trả cho nó bằng RAM cộng với rủi ro stale. Vì vậy chúng ta không cache thứ rẻ để tính lại, thứ đổi liên tục, hoặc thứ phải đúng tuyệt đối.

**English (bám cấu trúc tiếng Việt)**

A cache is a copy that is near and fast, it bets on locality, and we pay for it with RAM plus the risk of stale data. Therefore we do not cache things that are cheap to recompute, things that change constantly, or things that must be exactly correct.

---

## Bảng thuật ngữ tổng hợp

| Tiếng Việt | Tiếng Anh | Ghi chú phát âm (Anh-Anh) |
|---|---|---|
| bộ nhớ đệm | cache | /kæʃ/ — đọc y hệt "cash", **không** đọc "ca-chê" |
| dữ liệu cũ, lệch nguồn | stale | /steɪl/ — "xtêi-l", âm /eɪ/, kết thúc bằng /l/ rõ |
| tính cục bộ | locality | lə-**KAL**-ə-ti — trọng âm rơi vào âm tiết thứ hai |
| theo thời gian | temporal | **TEM**-pə-rəl — trọng âm âm tiết đầu |
| theo không gian | spatial | /ˈspeɪ.ʃəl/ — "SPÂY-shồl", **không** đọc "spa-ti-al" |
| bậc độ lớn | order of magnitude | **MAG**-ni-tyood — Anh-Anh có /tj/, không phải "tood" |
| tầng cache | cache layer / cache tier | tier /tɪər/ — "ti-ờ", một âm tiết |
| độ tươi | freshness | |
| độ trễ | latency | **LAY**-tən-si — âm đầu /leɪ/, không phải "la" |
| độ trễ đuôi | tail latency | |
| phân vị | percentile | pə-**SEN**-tail — trọng âm giữa |
| thông lượng | throughput | **THROO**-put — âm /θ/ đầu lưỡi, không phải "t" hay "th-ờ" |
| vòng mạng | network round trip | |
| chặng mạng | network hop | |
| máy chủ trung gian | reverse proxy | |
| bộ nhớ đệm phân tán | distributed cache | |
| trong tiến trình | in-process | |
| sẵn sàng cao | high availability | ə-**VEIL**-ə-**BIL**-ə-ti — âm /veɪ/, không phải "a-va" |
| toàn cụm | across the whole cluster | |
| lệch nhau | drift apart / diverge | diverge — dai-**VERJ**, kết thúc bằng /dʒ/ |
| làm mất hiệu lực | invalidate | in-**VAL**-i-deit — trọng âm âm tiết thứ hai |
| thời gian sống | time to live (TTL) | đọc rời từng chữ cái: "tee-tee-el" |
| nhiều tầng | multi-tier | |
| key nóng | hot key | |
| giảm tải | take load off / offload | |
| sức chứa | capacity | kə-**PAS**-ə-ti — trọng âm âm tiết thứ hai |
| kiến trúc | architecture | **AR**-ki-tek-chə — âm cuối là /tʃə/, không phải "-tua" |
| tỉ lệ hit | hit ratio | ratio: **RAY**-shi-əʊ — không đọc "ra-ti-ô" |
| tính nhất quán mạnh | strong consistency | kən-**SIS**-tən-si |
| ghi nhiều đọc ít | write-heavy, read-light | write — âm **w** câm, đọc như "rait" |
| tính lại | recompute | |
| tồn kho | stock level / inventory | inventory — Anh-Anh **IN**-vən-tri, ba âm tiết |
| hạn mức | credit limit | |
| số dư | balance | |
| chịu được | tolerate | **TOL**-ə-reit — trọng âm âm tiết đầu |
| tái hiện lỗi | reproduce | ri-prə-**DYOOS** — Anh-Anh có /dj/ |
| thu hồi quyền | revoke the rights | |
| nút thắt cổ chai | bottleneck | |
| truy vấn mỗi giây | queries per second (QPS) | queries — **KWEER**-riz, /kw/ rõ ở đầu |
| chuyển đổi dữ liệu | serialisation | si-ri-ə-lai-**ZAY**-shən — trọng âm gần cuối |
| tính sẵn | precompute / precomputation | |
| khung nhìn hiện thực hoá | materialized view | mə-**TEER**-ri-ə-laizd — trọng âm âm tiết thứ hai |
| lười, theo nhu cầu | lazy | |
| chủ động, tính trước | eager | **EE**-gə — /iː/ dài |
| làm mới | refresh | |
| cửa sổ dữ liệu cũ | the stale window | |
| đường đi, luồng xử lý | path | Anh-Anh /pɑːθ/ — "paath", âm cuối /θ/ |
| chính sách | policy | |
| rò rỉ dữ liệu | data leak | |
| đưa lên production | ship to production | |
| đánh đổi | trade-off | |
| bước thanh toán | the checkout step | |
| máy chủ biên | edge cache | edge /edʒ/ — kết thúc bằng /dʒ/, không phải "ét" |
| bộ nhớ đệm Redis | Redis | **RED**-iss — trọng âm đầu, không phải "ri-đai-x" |
| Memcached | Memcached | "mem-**CASHED**" — phần giữa đọc như "cash" |
| máy chủ Nginx | Nginx | đọc là "engine-X" |

---

## Bài nói

> **Cách luyện:** bật ghi âm, nói **60–90 giây** cho mỗi đề, **không nhìn tài liệu**. Mỗi bài nói phải dùng ít nhất **sáu thuật ngữ** trong bảng trên. Nghe lại một lần, đánh dấu chỗ bạn ngập ngừng, rồi nói lại đúng đề đó thêm một lần nữa.

1. **Explain to a junior** why a cache is a bet on locality, and what exactly we pay in exchange for that bet.

2. **Describe what happens** when a request travels from the browser down to the database. Name the six cache layers in order and say what each one trades away.

3. **A colleague says:** *"Our hit ratio is ninety-nine percent, so our cache is clearly working well."* Explain what is wrong with that claim and what you would measure instead.

4. **Someone on your team wants** to cache the stock level of a product so that the product page loads faster. Explain why you would push back, and describe what you would do instead.

5. **A teammate proposes** raising the TTL of the L1 in-process cache from five seconds to ten minutes, because it improves latency in the load test. Explain why you would push back, and what failure you expect in production.

6. **When would you choose** a materialized view over a cache, and when would you choose the opposite? Describe the cost of each direction.

7. **Your team added Redis** in front of the database, but p99 latency did not improve at all. Describe how you would investigate, and give three possible reasons.
