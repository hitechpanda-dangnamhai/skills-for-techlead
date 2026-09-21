# Bài 6 — Eviction và bộ nhớ
### Bản song ngữ: tiếng Anh bám cấu trúc tiếng Việt

> **Cách dùng bài này:** đọc đoạn tiếng Việt trước → tự dịch trong đầu → so với đoạn tiếng Anh bên dưới → **đọc to bản tiếng Anh hai lần**: lần một đọc chậm cho đúng, lần hai đọc trôi như đang nói với người phỏng vấn.

---

## Phần 1 — Eviction khác expiration ở chỗ nào

**Tiếng Việt**

Chúng ta cần tách bạch hai khái niệm hay bị gộp làm một. Expiration là việc key tự hết hạn theo đúng TTL mà chúng ta đã đặt. Eviction là việc Redis chủ động xoá key khi bộ nhớ chạm ngưỡng maxmemory, để lấy chỗ cho dữ liệu mới. Hai cơ chế này độc lập với nhau, vì vậy eviction có thể xoá cả những key chưa hết hạn. Đây chính là chỗ nhiều lập trình viên bị sốc: họ đặt TTL một giờ, nhưng key biến mất chỉ sau mười phút. Câu trả lời là key đó đã bị đuổi vì bộ nhớ đầy, chứ không phải vì nó hết hạn. Nếu chúng ta không biết điều này, chúng ta sẽ đi tìm bug ở sai chỗ trong nhiều giờ đồng hồ.

Hệ quả thực tế của việc này rất nghiêm trọng nếu chúng ta hiểu sai. Chúng ta không bao giờ được coi Redis là một nơi mà dữ liệu chắc chắn còn nguyên cho tới hết TTL. Mọi đoạn code đọc cache đều phải xử lý được trường hợp key biến mất sớm hơn dự kiến. Nói cách khác, cache miss phải là một nhánh bình thường của luồng xử lý, chứ không phải một trường hợp ngoại lệ.

**English (bám cấu trúc tiếng Việt)**

We need to separate two concepts that are often merged into one. Expiration is the key running out by itself according to exactly the TTL that we set. Eviction is Redis actively deleting keys when the memory reaches the maxmemory threshold, in order to make room for new data. These two mechanisms are independent of each other, therefore eviction can delete keys that have not expired yet. This is exactly where many developers get a shock: they set a TTL of one hour, but the key disappears after only ten minutes. The answer is that the key was evicted because the memory was full, and not because it expired. If we do not know this, we will hunt for the bug in the wrong place for hours on end.

The practical consequence of this is very serious if we get it wrong. We must never treat Redis as a place where the data is guaranteed to stay intact until the TTL runs out. Every piece of code that reads the cache has to handle the case where the key disappears earlier than expected. In other words, a cache miss must be a normal branch of the flow, and not an exceptional case.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| hai khái niệm hay bị gộp làm một | two concepts that are often merged into one |
| key tự hết hạn theo đúng TTL | the key running out by itself according to exactly the TTL |
| khi bộ nhớ chạm ngưỡng | when the memory reaches the threshold |
| để lấy chỗ cho dữ liệu mới | in order to make room for new data |
| độc lập với nhau | independent of each other |
| chỗ nhiều lập trình viên bị sốc | where many developers get a shock |
| key đó đã bị đuổi vì bộ nhớ đầy | the key was evicted because the memory was full |
| đi tìm bug ở sai chỗ trong nhiều giờ đồng hồ | hunt for the bug in the wrong place for hours on end |
| chắc chắn còn nguyên cho tới hết TTL | guaranteed to stay intact until the TTL runs out |
| biến mất sớm hơn dự kiến | disappears earlier than expected |
| một nhánh bình thường của luồng xử lý | a normal branch of the flow |
| một trường hợp ngoại lệ | an exceptional case |

**Thuật ngữ cần nhớ**

- đuổi key khi hết bộ nhớ → **eviction**
- hết hạn theo thời gian → **expiration**
- ngưỡng → **a threshold**
- còn nguyên vẹn → **intact**
- lấy chỗ cho → **to make room for**

---

## Phần 2 — Tám chính sách eviction và cách chọn

**Tiếng Việt**

Redis hiện có tám chính sách eviction, và chúng ta nên nhớ chúng theo hai trục thay vì học thuộc tám cái tên. Trục thứ nhất là phạm vi xét: allkeys nghĩa là xét mọi key, còn volatile nghĩa là chỉ xét những key có đặt TTL. Trục thứ hai là tiêu chí chọn nạn nhân: lâu nhất không được dùng, ít được dùng nhất, ngẫu nhiên, hoặc sắp hết hạn nhất. Ghép hai trục lại, chúng ta có bảy chính sách, cộng thêm một chính sách đặc biệt là noeviction. Với noeviction, Redis không xoá gì cả và nó trả về lỗi hết bộ nhớ khi chúng ta cố ghi thêm. Mặc định của nhiều dịch vụ quản lý là volatile-lru, ví dụ ElastiCache và Memorystore. Với một instance thuần làm cache, chúng ta nên chọn allkeys-lru hoặc allkeys-lfu. Với một instance giữ dữ liệu không được phép mất, chúng ta chọn noeviction và bật lưu bền.

Có một cái bẫy trong nhóm volatile mà chúng ta phải nói ra. Nhóm volatile chỉ đụng tới những key có TTL, vì vậy nếu trong instance tồn tại nhiều key không có TTL thì Redis có thể không giải phóng được gì cả. Khi đó chúng ta rơi vào tình trạng bộ nhớ đầy nhưng eviction không có nạn nhân nào để xoá. Vì vậy nếu chúng ta chọn nhóm volatile, chúng ta phải bảo đảm rằng mọi key đều được đặt TTL. Nếu chúng ta không chắc chắn về điều đó, nhóm allkeys an toàn hơn cho một instance làm cache.

**English (bám cấu trúc tiếng Việt)**

Redis currently has eight eviction policies, and we should remember them along two axes instead of learning eight names by heart. The first axis is the scope of consideration: allkeys means considering every key, while volatile means considering only the keys that have a TTL set. The second axis is the criterion for choosing the victim: the least recently used, the least frequently used, random, or the closest to expiry. Putting the two axes together, we get seven policies, plus one special policy which is noeviction. With noeviction, Redis deletes nothing at all and it returns an out-of-memory error when we try to write more. The default of many managed services is volatile-lru, for example ElastiCache and Memorystore. For an instance that is purely a cache, we should choose allkeys-lru or allkeys-lfu. For an instance holding data that must not be lost, we choose noeviction and turn on persistence.

There is a trap inside the volatile group that we have to say out loud. The volatile group only touches the keys that have a TTL, therefore if there are many keys without a TTL in the instance then Redis may not be able to free anything at all. In that case we fall into a state where the memory is full but eviction has no victim to delete. Therefore if we choose the volatile group, we have to make sure that every key has a TTL set. If we are not certain about that, the allkeys group is safer for an instance that serves as a cache.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| nhớ chúng theo hai trục | remember them along two axes |
| thay vì học thuộc tám cái tên | instead of learning eight names by heart |
| phạm vi xét | the scope of consideration |
| tiêu chí chọn nạn nhân | the criterion for choosing the victim |
| lâu nhất không được dùng | the least recently used |
| sắp hết hạn nhất | the closest to expiry |
| ghép hai trục lại | putting the two axes together |
| khi chúng ta cố ghi thêm | when we try to write more |
| một instance thuần làm cache | an instance that is purely a cache |
| dữ liệu không được phép mất | data that must not be lost |
| có thể không giải phóng được gì cả | may not be able to free anything at all |
| eviction không có nạn nhân nào để xoá | eviction has no victim to delete |

**Thuật ngữ cần nhớ**

- chính sách eviction → **an eviction policy**
- key có đặt TTL → **a volatile key**
- lâu nhất không dùng → **least recently used (LRU)**
- ít được dùng nhất → **least frequently used (LFU)**
- lưu bền → **persistence**

---

## Phần 3 — LRU và LFU: hai tiêu chí, hai kết quả

**Tiếng Việt**

LRU đuổi key lâu nhất không được dùng, vì vậy nó xét theo tính gần đây. LFU đuổi key ít được dùng nhất, vì vậy nó xét theo tần suất. Hai tiêu chí này cho kết quả khác hẳn nhau trong một tình huống rất cụ thể. Giả sử một con crawler quét qua toàn bộ key trong vài phút, và mỗi key chỉ được đọc đúng một lần. Với LRU, tất cả những key vừa bị quét đều được coi là vừa mới dùng, vì vậy chúng đẩy các key nóng thật ra ngoài. Chúng ta gọi hiện tượng đó là ô nhiễm cache, và nó làm tỉ lệ hit tụt hẳn dù không có gì hỏng hóc. Với LFU, những key chỉ được đọc một lần vẫn có tần suất thấp, vì vậy chúng bị đuổi trước và các key nóng được giữ lại.

Vì vậy cách chọn của chúng ta phụ thuộc vào mẫu truy cập chứ không phụ thuộc vào sở thích. Nếu tải của chúng ta chủ yếu là người dùng thật đọc đi đọc lại một tập nhỏ, LRU đủ tốt và nó rẻ hơn. Nếu chúng ta thường xuyên bị các đợt quét một lần làm nhiễu, LFU là lựa chọn đúng hơn. Trong Redis, LFU còn có cơ chế suy giảm theo thời gian, nhờ đó một key từng nóng trong quá khứ không được giữ lại mãi mãi.

**English (bám cấu trúc tiếng Việt)**

LRU evicts the key that has gone the longest without being used, therefore it looks at recency. LFU evicts the key that is used the least, therefore it looks at frequency. These two criteria give completely different results in one very concrete situation. Suppose a crawler sweeps through all the keys in a few minutes, and each key is read exactly once. With LRU, all the keys that have just been swept count as recently used, therefore they push the genuinely hot keys out. We call that phenomenon cache pollution, and it makes the hit ratio drop sharply even though nothing is broken. With LFU, the keys that are read only once still have a low frequency, therefore they are evicted first and the hot keys are kept.

Therefore our choice depends on the access pattern instead of depending on personal taste. If our load is mostly real users reading a small set over and over, LRU is good enough and it is cheaper. If we are regularly disturbed by one-off sweeps, LFU is the more correct choice. In Redis, LFU also has a decay mechanism over time, thanks to which a key that was hot in the past is not kept forever.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| key lâu nhất không được dùng | the key that has gone the longest without being used |
| nó xét theo tính gần đây | it looks at recency |
| cho kết quả khác hẳn nhau | give completely different results |
| quét qua toàn bộ key | sweeps through all the keys |
| đều được coi là vừa mới dùng | all count as recently used |
| đẩy các key nóng thật ra ngoài | push the genuinely hot keys out |
| ô nhiễm cache | cache pollution |
| dù không có gì hỏng hóc | even though nothing is broken |
| phụ thuộc vào mẫu truy cập | depends on the access pattern |
| đọc đi đọc lại một tập nhỏ | reading a small set over and over |
| bị các đợt quét một lần làm nhiễu | disturbed by one-off sweeps |
| cơ chế suy giảm theo thời gian | a decay mechanism over time |

**Thuật ngữ cần nhớ**

- tính gần đây → **recency**
- tần suất → **frequency**
- ô nhiễm cache → **cache pollution**
- suy giảm dần → **decay**
- mẫu truy cập → **the access pattern**

---

## Phần 4 — Vì sao LRU của Redis chỉ là xấp xỉ

**Tiếng Việt**

Một chi tiết ít người biết là LRU và LFU trong Redis đều chỉ là xấp xỉ. Redis không giữ một danh sách toàn cục xếp theo thứ tự sử dụng, vì làm như vậy tốn rất nhiều bộ nhớ và tốn cả CPU. Thay vào đó, Redis lấy mẫu ngẫu nhiên một vài key rồi chọn nạn nhân tốt nhất trong số đó. Tham số maxmemory-samples điều khiển số key được lấy mẫu, và giá trị mặc định là năm. Nếu chúng ta tăng tham số này lên, kết quả sẽ gần với LRU thật hơn nhưng chúng ta tốn thêm CPU. Đây là một đánh đổi kinh điển giữa độ chính xác và chi phí, và Redis đã chọn nghiêng về phía chi phí thấp.

Chúng ta nên nói rõ vì sao Redis từ chối làm LRU thật. Một LRU thật cần một danh sách liên kết hai chiều gắn với mọi key, và cấu trúc đó tự nó ngốn thêm bộ nhớ cho từng phần tử. Với một hệ thống mà mục tiêu chính là nhét được càng nhiều dữ liệu vào RAM càng tốt, cái giá đó không đáng. Chi tiết này rất đáng nói trong phỏng vấn, vì nó cho thấy chúng ta hiểu được lý do đứng sau một quyết định thiết kế.

**English (bám cấu trúc tiếng Việt)**

A detail that few people know is that LRU and LFU in Redis are both only approximations. Redis does not keep a global list ordered by usage, because doing so costs a great deal of memory and costs CPU as well. Instead, Redis samples a few keys at random and then picks the best victim among them. The maxmemory-samples parameter controls the number of keys that are sampled, and the default value is five. If we raise this parameter, the result will be closer to true LRU but we spend more CPU. This is a classic trade-off between accuracy and cost, and Redis has chosen to lean towards low cost.

We should say clearly why Redis refuses to do true LRU. A true LRU needs a doubly linked list attached to every key, and that structure by itself eats extra memory for each element. For a system whose main goal is to fit as much data into RAM as possible, that price is not worth it. This detail is well worth mentioning in an interview, because it shows that we understand the reasoning behind a design decision.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| một chi tiết ít người biết | a detail that few people know |
| đều chỉ là xấp xỉ | are both only approximations |
| một danh sách toàn cục xếp theo thứ tự sử dụng | a global list ordered by usage |
| lấy mẫu ngẫu nhiên một vài key | samples a few keys at random |
| chọn nạn nhân tốt nhất trong số đó | picks the best victim among them |
| gần với LRU thật hơn | closer to true LRU |
| chọn nghiêng về phía chi phí thấp | chosen to lean towards low cost |
| một danh sách liên kết hai chiều | a doubly linked list |
| tự nó ngốn thêm bộ nhớ cho từng phần tử | by itself eats extra memory for each element |
| nhét được càng nhiều dữ liệu vào RAM càng tốt | fit as much data into RAM as possible |
| cái giá đó không đáng | that price is not worth it |
| lý do đứng sau một quyết định thiết kế | the reasoning behind a design decision |

**Thuật ngữ cần nhớ**

- xấp xỉ → **an approximation** / **approximated**
- lấy mẫu → **to sample**
- nạn nhân bị đuổi → **the victim**
- danh sách liên kết hai chiều → **a doubly linked list**
- độ chính xác → **accuracy**

---

## Phần 5 — Đừng trộn cache với kho dữ liệu bền

**Tiếng Việt**

Đây là lỗi cấu hình nguy hiểm nhất trong bài này. Nhiều đội dùng chung một Redis cho cả ba việc: làm cache, giữ session, và giữ hàng đợi công việc. Nếu instance đó đặt chính sách nhóm allkeys, Redis được phép xoá bất kỳ key nào khi bộ nhớ đầy. Nghĩa là nó có thể xoá một session đang hoạt động hoặc một job chưa được xử lý, chỉ vì cần chỗ cho một trang sản phẩm. Hậu quả là người dùng bị đăng xuất giữa chừng, hoặc một công việc biến mất mà không ai biết. Lỗi này rất khó lần ra, vì nó chỉ xuất hiện khi bộ nhớ đầy, tức là đúng lúc hệ thống đang bận nhất. Cách sửa đúng là tách thành hai instance riêng: một instance làm cache với allkeys-lru, và một instance giữ dữ liệu bền với noeviction cộng với lưu bền.

Chúng ta nên biến điều này thành một câu hỏi review cố định. Câu hỏi đó là: trong instance này có key nào không được phép mất hay không. Nếu có, mà instance đó lại đang bật eviction, thì chúng ta đang chờ một sự cố xảy ra. Công cụ AI rất hay gợi ý dùng chung một Redis cho mọi việc vì như vậy tiện hơn, và đây đúng là chỗ chúng ta phải chặn lại.

**English (bám cấu trúc tiếng Việt)**

This is the most dangerous configuration mistake in this lesson. Many teams share one Redis for all three jobs: serving as a cache, holding sessions, and holding the job queue. If that instance is set to an allkeys policy, Redis is allowed to delete any key at all when the memory is full. That means it can delete an active session or a job that has not been processed yet, just because it needs room for a product page. The consequence is that a user is logged out halfway through, or a job disappears without anybody knowing. This bug is very hard to track down, because it only shows up when the memory is full, that is exactly when the system is at its busiest. The correct fix is to split it into two separate instances: one instance as the cache with allkeys-lru, and one instance holding durable data with noeviction plus persistence.

We should turn this into a standing review question. That question is: is there any key in this instance that must not be lost. If there is, while that instance has eviction switched on, then we are waiting for an incident to happen. AI tools very often suggest sharing one Redis for everything because it is more convenient, and this is exactly where we have to stop them.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| lỗi cấu hình nguy hiểm nhất | the most dangerous configuration mistake |
| dùng chung một Redis cho cả ba việc | share one Redis for all three jobs |
| được phép xoá bất kỳ key nào | is allowed to delete any key at all |
| một job chưa được xử lý | a job that has not been processed yet |
| chỉ vì cần chỗ cho một trang sản phẩm | just because it needs room for a product page |
| bị đăng xuất giữa chừng | logged out halfway through |
| rất khó lần ra | very hard to track down |
| đúng lúc hệ thống đang bận nhất | exactly when the system is at its busiest |
| tách thành hai instance riêng | split it into two separate instances |
| một câu hỏi review cố định | a standing review question |
| chúng ta đang chờ một sự cố xảy ra | we are waiting for an incident to happen |
| đây đúng là chỗ chúng ta phải chặn lại | this is exactly where we have to stop them |

**Thuật ngữ cần nhớ**

- kho dữ liệu bền → **a durable store**
- hàng đợi công việc → **the job queue**
- phiên đăng nhập → **a session**
- sự cố vận hành → **an incident**
- lần ra nguyên nhân → **to track down**

---

## Phần 6 — Tỉ lệ hit tụt sau khi eviction bắt đầu chạy

**Tiếng Việt**

Có một triệu chứng vận hành mà chúng ta nên nhận ra ngay: tỉ lệ hit tụt mạnh ngay sau khi eviction bắt đầu hoạt động. Chẩn đoán thường gặp nhất là tập dữ liệu đang hoạt động lớn hơn bộ nhớ khả dụng. Khi đó Redis liên tục đuổi ra chính những key sắp được dùng lại, và chúng ta gọi hiện tượng này là cache thrash. Chúng ta xác nhận chẩn đoán đó bằng hai chỉ số: số key bị đuổi rất cao trong khi số lần trúng cache lại thấp. Có bốn hướng xử lý: tăng bộ nhớ, giảm bớt thứ chúng ta cache, tách dữ liệu nóng khỏi dữ liệu lạnh, hoặc chia thêm shard. Điều quan trọng là chúng ta phải đo trước khi sửa, vì bốn hướng đó có chi phí rất khác nhau.

Cách nói này rất mạnh trong phỏng vấn, vì nó gắn triệu chứng với chỉ số cụ thể. Chúng ta không nói mơ hồ rằng cache hoạt động không tốt. Chúng ta nói rằng số key bị đuổi tăng, tỉ lệ trúng giảm, và tập dữ liệu hoạt động vượt quá bộ nhớ. Một người phỏng vấn có kinh nghiệm sẽ nhận ra ngay rằng chúng ta đã từng nhìn vào một bảng chỉ số thật.

**English (bám cấu trúc tiếng Việt)**

There is an operational symptom that we should recognise immediately: the hit ratio drops sharply right after eviction starts working. The most common diagnosis is that the working set is larger than the available memory. In that case Redis constantly evicts exactly the keys that are about to be used again, and we call this phenomenon cache thrashing. We confirm that diagnosis with two metrics: the number of evicted keys is very high while the number of cache hits is low. There are four directions to handle it: increase the memory, reduce what we cache, separate hot data from cold data, or add more shards. The important thing is that we have to measure before we fix, because those four directions have very different costs.

This way of speaking is very strong in an interview, because it ties the symptom to concrete metrics. We do not say vaguely that the cache is not working well. We say that evicted keys are rising, the hit ratio is falling, and the working set exceeds the memory. An experienced interviewer will recognise straight away that we have actually looked at a real metrics dashboard.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| một triệu chứng vận hành | an operational symptom |
| tập dữ liệu đang hoạt động | the working set |
| lớn hơn bộ nhớ khả dụng | larger than the available memory |
| đuổi ra chính những key sắp được dùng lại | evicts exactly the keys that are about to be used again |
| chúng ta xác nhận chẩn đoán đó bằng hai chỉ số | we confirm that diagnosis with two metrics |
| giảm bớt thứ chúng ta cache | reduce what we cache |
| tách dữ liệu nóng khỏi dữ liệu lạnh | separate hot data from cold data |
| đo trước khi sửa | measure before we fix |
| nó gắn triệu chứng với chỉ số cụ thể | it ties the symptom to concrete metrics |
| chúng ta không nói mơ hồ rằng | we do not say vaguely that |
| vượt quá bộ nhớ | exceeds the memory |
| một bảng chỉ số thật | a real metrics dashboard |

**Thuật ngữ cần nhớ**

- tập dữ liệu đang hoạt động → **the working set**
- đuổi qua đuổi lại liên tục → **cache thrashing**
- chẩn đoán → **a diagnosis**
- chỉ số đo → **a metric**
- bảng theo dõi → **a dashboard**

---

## Phần 7 — Dữ liệu phụ trội và phân mảnh bộ nhớ

**Tiếng Việt**

Một câu hỏi hay gặp là vì sao một gigabyte dữ liệu lại chiếm hơn một gigabyte RAM. Lý do thứ nhất là mỗi key đều có phần dữ liệu phụ đi kèm, gồm siêu dữ liệu, thông tin hết hạn và các con trỏ. Lý do thứ hai là Redis mã hoá dữ liệu bên trong theo những cách khác nhau, tuỳ kiểu dữ liệu và tuỳ kích thước. Lý do thứ ba là bộ cấp phát bộ nhớ sinh ra phân mảnh, nghĩa là có những khoảng trống không dùng được nằm rải rác. Vì vậy quy tắc vận hành của chúng ta là đặt maxmemory ở khoảng bảy mươi tới tám mươi phần trăm RAM của máy, chứ không đặt sát mép. Chúng ta theo dõi ba con số trong phần thông tin bộ nhớ: bộ nhớ đã dùng, bộ nhớ mà hệ điều hành nhìn thấy, và tỉ lệ phân mảnh giữa hai con số đó.

Phần đệm đó không phải là lãng phí, mà là chỗ dành cho những việc khác. Redis cần bộ nhớ cho bộ đệm sao chép sang replica, cho bộ đệm ghi nhật ký, và cho các bộ đệm phía client. Nếu chúng ta đặt maxmemory bằng đúng toàn bộ RAM, chính hệ điều hành sẽ giết tiến trình Redis khi những bộ đệm đó phình ra. Đây là kiểu sự cố hay xảy ra lúc nửa đêm, vì vậy chúng ta nên chừa phần đệm ngay từ đầu.

**English (bám cấu trúc tiếng Việt)**

A question we meet often is why one gigabyte of data takes more than one gigabyte of RAM. The first reason is that every key carries extra data with it, including metadata, expiry information and pointers. The second reason is that Redis encodes the data internally in different ways, depending on the data type and on the size. The third reason is that the memory allocator produces fragmentation, meaning that there are unusable gaps scattered around. Therefore our operational rule is to set maxmemory at around seventy to eighty percent of the machine's RAM, instead of setting it right at the edge. We track three numbers in the memory information section: the memory used, the memory that the operating system sees, and the fragmentation ratio between those two numbers.

That headroom is not waste, but room set aside for other things. Redis needs memory for the replication buffer to the replicas, for the log-writing buffer, and for the client-side buffers. If we set maxmemory to exactly the whole of the RAM, the operating system itself will kill the Redis process when those buffers swell up. This is the kind of incident that tends to happen in the middle of the night, therefore we should leave the headroom right from the start.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| mỗi key đều có phần dữ liệu phụ đi kèm | every key carries extra data with it |
| mã hoá dữ liệu bên trong | encodes the data internally |
| tuỳ kiểu dữ liệu và tuỳ kích thước | depending on the data type and on the size |
| bộ cấp phát bộ nhớ | the memory allocator |
| những khoảng trống không dùng được nằm rải rác | unusable gaps scattered around |
| chứ không đặt sát mép | instead of setting it right at the edge |
| bộ nhớ mà hệ điều hành nhìn thấy | the memory that the operating system sees |
| phần đệm đó không phải là lãng phí | that headroom is not waste |
| chỗ dành cho những việc khác | room set aside for other things |
| khi những bộ đệm đó phình ra | when those buffers swell up |
| kiểu sự cố hay xảy ra lúc nửa đêm | the kind of incident that tends to happen in the middle of the night |
| chừa phần đệm ngay từ đầu | leave the headroom right from the start |

**Thuật ngữ cần nhớ**

- phần dữ liệu phụ trội → **overhead**
- siêu dữ liệu → **metadata**
- phân mảnh bộ nhớ → **memory fragmentation**
- bộ cấp phát bộ nhớ → **the memory allocator**
- phần đệm dung lượng → **headroom**

---

## Phần 8 — Góc tech lead: khi Redis từ chối ghi vì hết bộ nhớ

**Tiếng Việt**

Tình huống thách đố của bài này rất hay gặp trong thực tế. Chúng ta đặt TTL một giờ cho mọi key cache, chọn chính sách noeviction, và giới hạn bộ nhớ ở hai gigabyte. Một ngày nọ, app bắt đầu báo lỗi rằng lệnh ghi không được phép vì đã hết bộ nhớ. Nguyên nhân là key tích tụ nhanh hơn tốc độ mà TTL dọn chúng đi, vì vậy bộ nhớ chạm trần. Với chính sách noeviction, Redis từ chối mọi lệnh ghi thay vì đuổi bớt key, và app của chúng ta nhận lỗi. Cách sửa gồm ba hướng có thể làm cùng lúc: đổi sang allkeys-lru để cho phép đuổi key, tăng bộ nhớ, và giảm bớt thứ chúng ta cache hoặc rút ngắn TTL. Bài học rút ra là một instance làm cache thì không nên dùng noeviction.

Chúng ta có thể gói cả bài này thành hai câu hỏi khi review cấu hình. Câu thứ nhất là instance này là cache hay là kho dữ liệu bền, vì câu trả lời quyết định chính sách eviction. Câu thứ hai là chính sách hiện tại có thể xoá nhầm thứ gì, vì đó là một rủi ro thật chứ không phải một rủi ro trên giấy. Hai câu hỏi đó rất ngắn, nhưng chúng chặn được phần lớn sự cố liên quan tới bộ nhớ.

**English (bám cấu trúc tiếng Việt)**

The challenge scenario of this lesson comes up very often in real life. We set a TTL of one hour on every cache key, choose the noeviction policy, and cap the memory at two gigabytes. One day, the app starts reporting an error saying that the write command is not allowed because the memory has run out. The cause is that keys pile up faster than the rate at which the TTL clears them away, therefore the memory hits the ceiling. With the noeviction policy, Redis refuses every write instead of evicting some keys, and our app receives the error. The fix has three directions that can be done at the same time: switch to allkeys-lru so that eviction is allowed, increase the memory, and reduce what we cache or shorten the TTL. The lesson we take away is that an instance serving as a cache should not use noeviction.

We can wrap this whole lesson into two questions when we review a configuration. The first is whether this instance is a cache or a durable store, because the answer decides the eviction policy. The second is what the current policy could delete by mistake, because that is a real risk and not a risk on paper. Those two questions are very short, but they block most of the incidents related to memory.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| giới hạn bộ nhớ ở hai gigabyte | cap the memory at two gigabytes |
| lệnh ghi không được phép | the write command is not allowed |
| key tích tụ nhanh hơn tốc độ mà TTL dọn chúng đi | keys pile up faster than the rate at which the TTL clears them away |
| bộ nhớ chạm trần | the memory hits the ceiling |
| từ chối mọi lệnh ghi | refuses every write |
| ba hướng có thể làm cùng lúc | three directions that can be done at the same time |
| rút ngắn TTL | shorten the TTL |
| gói cả bài này thành hai câu hỏi | wrap this whole lesson into two questions |
| có thể xoá nhầm thứ gì | could delete by mistake |
| một rủi ro trên giấy | a risk on paper |
| chặn được phần lớn sự cố liên quan tới bộ nhớ | block most of the incidents related to memory |

**Thuật ngữ cần nhớ**

- hết bộ nhớ → **out of memory (OOM)**
- giới hạn ở mức → **to cap at**
- chạm trần → **to hit the ceiling**
- tích tụ lại → **to pile up**
- rủi ro trên giấy → **a risk on paper**

---

## Mô hình ghi nhớ

**Tiếng Việt**

Khi bộ nhớ chạm trần, Redis đuổi key theo chính sách đã đặt, và nó xoá cả những key chưa hết TTL. Vì vậy một instance thuần làm cache nên dùng allkeys-lru, còn dữ liệu không được phép mất phải nằm ở một instance riêng với noeviction và lưu bền.

**English (bám cấu trúc tiếng Việt)**

When the memory hits the ceiling, Redis evicts keys according to the policy we set, and it deletes keys that have not expired yet as well. Therefore an instance serving purely as a cache should use allkeys-lru, while data that must not be lost has to sit in a separate instance with noeviction and persistence.

---

## Bảng thuật ngữ tổng hợp

| Tiếng Việt | Tiếng Anh | Ghi chú phát âm (Anh-Anh) |
|---|---|---|
| đuổi key khi hết bộ nhớ | eviction | i-**VIK**-shən — trọng âm âm tiết thứ hai |
| hết hạn theo thời gian | expiration | ek-spi-**RAY**-shən |
| bộ nhớ đệm | cache | /kæʃ/ — đọc y hệt "cash" |
| ngưỡng | a threshold | **THRESH**-həuld — âm /θ/ đầu lưỡi |
| còn nguyên vẹn | intact | in-**TAKT** — trọng âm cuối, cụm /kt/ rõ |
| lấy chỗ cho | to make room for | |
| chính sách eviction | an eviction policy | |
| key có đặt TTL | a volatile key | Anh-Anh **VOL**-ə-tail — đuôi /aɪl/, khác Mỹ |
| lâu nhất không dùng | least recently used (LRU) | đọc rời chữ cái: "el-ar-yoo" |
| ít được dùng nhất | least frequently used (LFU) | **FREE**-kwənt-li |
| ngẫu nhiên | random | |
| không đuổi key | noeviction | |
| tính gần đây | recency | **REE**-sən-si |
| tần suất | frequency | **FREE**-kwən-si |
| ô nhiễm cache | cache pollution | pə-**LOO**-shən |
| con bọ quét web | a crawler | **KRAW**-lə |
| suy giảm dần | decay | di-**KAY** — trọng âm cuối |
| mẫu truy cập | the access pattern | **AK**-ses — trọng âm âm tiết đầu |
| xấp xỉ | an approximation | ə-prok-si-**MAY**-shən |
| lấy mẫu | to sample | |
| nạn nhân bị đuổi | the victim | **VIK**-tim |
| danh sách liên kết hai chiều | a doubly linked list | **DUB**-li — "ou" đọc thành /ʌ/ |
| độ chính xác | accuracy | **AK**-yə-rə-si |
| tham số | a parameter | pə-**RAM**-i-tə — trọng âm âm tiết thứ hai |
| lưu bền | persistence | pə-**SIS**-təns |
| kho dữ liệu bền | a durable store | **DYOO**-rə-bl — Anh-Anh có /dj/ |
| hàng đợi công việc | the job queue | queue /kjuː/ — đọc đúng như chữ cái **Q** |
| phiên đăng nhập | a session | **SESH**-ən |
| sự cố vận hành | an incident | **IN**-si-dənt |
| lần ra nguyên nhân | to track down | |
| tập dữ liệu đang hoạt động | the working set | |
| đuổi qua đuổi lại liên tục | cache thrashing | **THRASH**-ing — âm /θ/, không phải "trát" |
| chẩn đoán | a diagnosis | dai-əg-**NƏU**-sis — số nhiều *diagnoses* |
| chỉ số đo | a metric | **MET**-rik |
| bảng theo dõi | a dashboard | |
| dữ liệu nóng và dữ liệu lạnh | hot data and cold data | |
| mảnh dữ liệu | a shard | /ʃɑːd/ — bắt đầu bằng /ʃ/ |
| phần dữ liệu phụ trội | overhead | əʊ-və-**HED** — trọng âm cuối |
| siêu dữ liệu | metadata | **MET**-ə-day-tə |
| con trỏ | a pointer | |
| phân mảnh bộ nhớ | memory fragmentation | frag-men-**TAY**-shən |
| bộ cấp phát bộ nhớ | the memory allocator | **AL**-ə-kay-tə |
| phần đệm dung lượng | headroom | |
| bộ đệm | a buffer | **BUF**-ə |
| hệ điều hành | the operating system | |
| hết bộ nhớ | out of memory (OOM) | đọc rời chữ cái: "oh-oh-em" |
| giới hạn ở mức | to cap at | |
| chạm trần | to hit the ceiling | **SEE**-ling |
| tích tụ lại | to pile up | |
| rủi ro trên giấy | a risk on paper | risk — kết thúc bằng cụm /sk/, đừng nuốt âm |
| tỉ lệ hit | hit ratio | ratio — **RAY**-shi-əʊ |

---

## Bài nói

> **Cách luyện:** bật ghi âm, nói **60–90 giây** cho mỗi đề, **không nhìn tài liệu**. Mỗi bài nói phải dùng ít nhất **sáu thuật ngữ** trong bảng trên. Nghe lại một lần, đánh dấu chỗ bạn ngập ngừng, rồi nói lại đúng đề đó thêm một lần nữa.

1. **Explain to a junior** the difference between expiration and eviction, and why a key with a one-hour TTL can disappear after ten minutes.

2. **A colleague says:** *"I set a one-hour TTL, so the value is guaranteed to be there for an hour."* Explain why you would push back, and what their code has to handle instead.

3. **Explain when you would choose LFU over LRU**, using a crawler sweeping every key as your example.

4. **Someone on your team wants** one Redis instance for the cache, the sessions and the job queue, all on allkeys-lru, because it is cheaper. Explain why you would push back and what you would set up instead.

5. **Your hit ratio dropped sharply** right after eviction started. Describe how you would diagnose it, which two metrics you would look at, and the options you would weigh.

6. **Explain why one gigabyte of data** takes more than one gigabyte of RAM, and say what maxmemory you would set on a machine and why.

7. **A teammate configured** the cache instance with noeviction so that nothing would ever be lost, and now the app gets out-of-memory errors on writes. Explain what happened and how you would fix it.
