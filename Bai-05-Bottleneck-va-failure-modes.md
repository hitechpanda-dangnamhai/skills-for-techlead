# Bài 5 — Bottleneck & failure modes ở production
### Bản song ngữ: tiếng Anh bám cấu trúc tiếng Việt

> **Cách dùng:** đọc đoạn tiếng Việt trước → tự dịch trong đầu → so với bản tiếng Anh bên dưới → **đọc to bản tiếng Anh hai lần**. Lần một đọc chậm cho đúng từ, lần hai đọc liền mạch như đang nói với người phỏng vấn. Bài này là bài mà người phỏng vấn dùng để phân biệt người đã vận hành hệ thống thật với người chỉ đọc sách, nên hãy luyện cho tới khi các cụm mô tả sự cố bật ra tự nhiên.

---

## Phần 1 — Soi điểm nghẽn ở đâu trước

**Tiếng Việt**

Khi người phỏng vấn hỏi điểm nghẽn của thiết kế mà chúng ta vừa vẽ là gì, họ đang kiểm tra xem chúng ta có nhìn được chỗ tải dồn hay không. Điểm nghẽn gần như luôn nằm ở một trong hai chỗ: tầng có mức đồng thời cao nhất, hoặc chỗ giữ trạng thái tập trung. Chỗ giữ trạng thái tập trung là nơi mà mọi request đều phải đi qua một điểm duy nhất, ví dụ đường ghi vào cơ sở dữ liệu hoặc một bộ cân bằng tải đơn lẻ. Vì vậy, khi nhìn một bản vẽ, chúng ta rà từ ngoài vào trong và tìm cái hộp mà mọi mũi tên đều chụm vào. Cái hộp đó là nghi phạm đầu tiên, và chúng ta nên nói tên nó ra trước khi người phỏng vấn phải hỏi thêm.

Điều quan trọng là chúng ta phải chỉ ra một điểm cụ thể kèm theo cách giảm tải cho nó, chứ không nói chung chung rằng hệ thống cần mở rộng. Câu nói cần mở rộng thêm không cho người nghe biết chúng ta hiểu gì, vì ai cũng nói được câu đó. Cách nói đúng là chúng ta gọi tên tầng, gọi tên chỉ số sẽ tăng trước, rồi nêu biện pháp rẻ nhất có thể thử đầu tiên. Ví dụ, chúng ta nói rằng đường ghi vào cơ sở dữ liệu sẽ bão hoà trước, chúng ta sẽ thấy điều đó qua thời gian chờ khoá tăng, và bước đầu tiên của chúng ta là gộp ghi theo lô. Nếu chúng ta chỉ nói chung chung, hậu quả là người phỏng vấn xếp chúng ta vào nhóm chưa từng phải cứu một hệ thống đang cháy.

**English (bám cấu trúc tiếng Việt)**

When the interviewer asks what the bottleneck of the design we have just drawn is, they are testing whether we can see where the load piles up. The bottleneck is almost always in one of two places: the layer with the highest concurrency, or the place that holds centralised state. A place that holds centralised state is where every request has to go through one single point, for example the write path into the database or a single load balancer. Therefore, when we look at a drawing, we scan from the outside inwards and look for the box that all the arrows converge on. That box is the first suspect, and we should name it before the interviewer has to ask further.

The important thing is that we must point to a specific place together with a way to reduce the load on it, rather than saying vaguely that the system needs to scale. The sentence "it needs to scale" tells the listener nothing about what we understand, because anybody can say that sentence. The correct way to speak is that we name the layer, name the metric that will rise first, and then give the cheapest measure we could try first. For example, we say that the write path into the database will saturate first, we will see that through the rising lock wait time, and our first step is to batch the writes. If we only speak in general terms, the consequence is that the interviewer puts us in the group who have never had to save a system on fire.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| chỗ tải dồn | where the load piles up |
| tầng có mức đồng thời cao nhất | the layer with the highest concurrency |
| chỗ giữ trạng thái tập trung | the place that holds centralised state |
| phải đi qua một điểm duy nhất | has to go through one single point |
| chúng ta rà từ ngoài vào trong | we scan from the outside inwards |
| cái hộp mà mọi mũi tên đều chụm vào | the box that all the arrows converge on |
| nghi phạm đầu tiên | the first suspect |
| gọi tên chỉ số sẽ tăng trước | name the metric that will rise first |
| sẽ bão hoà trước | will saturate first |
| gộp ghi theo lô | batch the writes |
| chưa từng phải cứu một hệ thống đang cháy | have never had to save a system on fire |

**Thuật ngữ cần nhớ**

- điểm nghẽn → **a bottleneck**
- mức đồng thời → **concurrency**
- trạng thái tập trung → **centralised state**
- bão hoà → **to saturate** / **saturation**
- gộp theo lô → **to batch**

---

## Phần 2 — Cơ sở dữ liệu thường là điểm nghẽn đầu tiên

**Tiếng Việt**

Trong đa số hệ thống, cơ sở dữ liệu là thứ chạm trần trước tiên, vì mọi đường đọc và đường ghi cuối cùng đều đổ về đó. Khi chúng ta phát hiện cơ sở dữ liệu là điểm nghẽn, chúng ta nên đi theo một thang biện pháp từ rẻ tới đắt. Nấc đầu tiên là thêm chỉ mục cho đúng truy vấn, vì rất nhiều vấn đề hiệu năng chỉ là một truy vấn quét toàn bảng. Nấc thứ hai là đặt một tầng cache phía trước để chặn bớt lượt đọc lặp lại. Nấc thứ ba là thêm bản sao chỉ đọc, nấc thứ tư là phân mảnh, và nấc cuối cùng là tách đường đọc khỏi đường ghi bằng mô hình CQRS hoặc bằng hàng đợi.

Sai lầm phổ biến là nhảy thẳng vào phân mảnh ngay khi tải tăng, vì phân mảnh nghe có vẻ là câu trả lời của người giỏi. Thực tế thì phân mảnh rất đắt về mặt vận hành, vì chúng ta phải chọn khoá phân mảnh, phải cân bằng lại khi lệch, và phải sống thiếu phép nối bảng giữa các mảnh. Đa số trường hợp được giải quyết gọn ở hai nấc đầu tiên, tức là chỉ mục và cache, với chi phí gần bằng không. Vì vậy, trong phỏng vấn chúng ta nên đọc rõ cả thang biện pháp rồi mới nói chúng ta chọn nấc nào và vì sao. Nếu chúng ta phân mảnh quá sớm, hậu quả là đội của chúng ta gánh một hệ thống phức tạp gấp ba trong khi vấn đề thật chỉ là một chỉ mục còn thiếu.

**English (bám cấu trúc tiếng Việt)**

In most systems, the database is the thing that hits its ceiling first, because every read path and every write path eventually pours into it. When we find that the database is the bottleneck, we should follow a ladder of measures from cheap to expensive. The first rung is to add an index for the right query, because a great many performance problems are just one query doing a full table scan. The second rung is to put a cache layer in front in order to block some of the repeated reads. The third rung is to add read replicas, the fourth rung is sharding, and the last rung is to separate the read path from the write path with the CQRS pattern or with a queue.

A common mistake is to jump straight to sharding as soon as the load rises, because sharding sounds like the answer of a strong engineer. In reality sharding is very expensive operationally, because we have to choose a shard key, we have to rebalance when it becomes skewed, and we have to live without joins across shards. Most cases are solved neatly at the first two rungs, that is, indexes and caching, at a cost close to zero. Therefore, in an interview we should read out the whole ladder of measures and only then say which rung we choose and why. If we shard too early, the consequence is that our team carries a system three times more complex while the real problem was just a missing index.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| là thứ chạm trần trước tiên | is the thing that hits its ceiling first |
| cuối cùng đều đổ về đó | eventually pours into it |
| một thang biện pháp từ rẻ tới đắt | a ladder of measures from cheap to expensive |
| nấc đầu tiên là | the first rung is |
| chặn bớt lượt đọc lặp lại | block some of the repeated reads |
| tách đường đọc khỏi đường ghi | separate the read path from the write path |
| nghe có vẻ là câu trả lời của người giỏi | sounds like the answer of a strong engineer |
| phải sống thiếu phép nối bảng giữa các mảnh | have to live without joins across shards |
| được giải quyết gọn ở hai nấc đầu tiên | are solved neatly at the first two rungs |
| với chi phí gần bằng không | at a cost close to zero |
| một hệ thống phức tạp gấp ba | a system three times more complex |

**Thuật ngữ cần nhớ**

- chỉ mục → **an index**
- bản sao chỉ đọc → **a read replica**
- khoá phân mảnh → **a shard key**
- tách đọc ghi → **CQRS (command query responsibility segregation)**
- truy vấn chậm → **a slow query**

---

## Phần 3 — Khoá nóng và phân vùng nóng

**Tiếng Việt**

Khoá nóng là tình huống mà một khoá hoặc một mảnh nhận phần tải lớn hơn hẳn các mảnh còn lại. Nguyên nhân thường là dữ liệu tự nhiên bị lệch, ví dụ một người nổi tiếng có hàng triệu người theo dõi, hoặc một sản phẩm đang được bán chạy đột biến. Nguyên nhân thứ hai là chúng ta chọn khoá phân mảnh xấu, ví dụ chúng ta phân mảnh theo quốc gia trong khi phần lớn người dùng ở cùng một nước. Chúng ta phát hiện khoá nóng bằng cách nhìn phân phối tải giữa các mảnh, và dấu hiệu điển hình là độ trễ p99 của đúng một mảnh cao vọt lên. Nếu chúng ta chỉ nhìn con số trung bình toàn cụm, chúng ta sẽ không thấy gì cả, vì một mảnh cháy bị các mảnh rảnh làm loãng đi.

Có vài cách chữa, và chúng ta nên nói được từng cách kèm theo cái giá của nó. Cách thứ nhất là chọn khoá phân mảnh tốt hơn, tức là khoá có độ phân tán cao như mã người dùng thay vì mã quốc gia. Cách thứ hai là thêm muối vào khoá, nghĩa là chúng ta gắn thêm một hậu tố ngẫu nhiên để trải một khoá nóng ra nhiều mảnh, đổi lại việc đọc phải gom từ nhiều nơi. Cách thứ ba là dành riêng một cache cho những khoá nóng, vì số khoá nóng thường rất ít nên cache chúng rất hiệu quả. Cách thứ tư là nhân bản dữ liệu nóng ra nhiều bản chỉ đọc để chia đều lượt đọc. Nếu chúng ta bỏ qua khoá nóng, hậu quả là chúng ta thêm máy mãi mà độ trễ đuôi vẫn không giảm, vì nút thắt nằm ở một khoá duy nhất chứ không nằm ở tổng dung lượng.

**English (bám cấu trúc tiếng Việt)**

A hot key is the situation where one key or one shard takes a much larger share of the load than the remaining shards. The cause is usually that the data is naturally skewed, for example a celebrity with millions of followers, or a product that is suddenly selling extremely well. The second cause is that we chose a bad shard key, for example we shard by country while most of the users are in the same country. We detect a hot key by looking at the load distribution across the shards, and the typical sign is that the p99 latency of exactly one shard shoots up. If we only look at the average number across the cluster, we will see nothing at all, because one burning shard is diluted by the idle shards.

There are several fixes, and we should be able to say each one together with its price. The first way is to choose a better shard key, that is, a key with high cardinality such as the user id instead of the country code. The second way is to add salt to the key, which means we attach a random suffix in order to spread one hot key across many shards, in exchange the read has to gather from several places. The third way is to dedicate a cache to the hot keys, because the number of hot keys is usually very small so caching them is very effective. The fourth way is to replicate the hot data into several read-only copies in order to spread the reads evenly. If we ignore the hot key, the consequence is that we keep adding machines while the tail latency does not come down, because the knot is at one single key rather than in the total capacity.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| nhận phần tải lớn hơn hẳn | takes a much larger share of the load |
| dữ liệu tự nhiên bị lệch | the data is naturally skewed |
| đang được bán chạy đột biến | is suddenly selling extremely well |
| nhìn phân phối tải giữa các mảnh | looking at the load distribution across the shards |
| cao vọt lên | shoots up |
| bị các mảnh rảnh làm loãng đi | is diluted by the idle shards |
| khoá có độ phân tán cao | a key with high cardinality |
| thêm muối vào khoá | add salt to the key |
| gắn thêm một hậu tố ngẫu nhiên | attach a random suffix |
| việc đọc phải gom từ nhiều nơi | the read has to gather from several places |
| độ trễ đuôi vẫn không giảm | the tail latency does not come down |

**Thuật ngữ cần nhớ**

- khoá nóng → **a hot key**
- phân vùng nóng → **a hot partition**
- lệch tải → **skew**
- độ phân tán → **cardinality**
- thêm muối → **salting**
- độ trễ đuôi → **tail latency**

---

## Phần 4 — Cạn pool kết nối và lỗi lan dây chuyền

**Tiếng Việt**

Mỗi dịch vụ giữ một pool kết nối tới cơ sở dữ liệu, và pool đó có số lượng hữu hạn. Khi các truy vấn chậm lại, các kết nối bị giữ lâu hơn, nên request mới phải xếp hàng chờ được cấp một kết nối. Thời gian chờ đó cộng vào độ trễ của người dùng, và khi hàng chờ dài ra thì các request bắt đầu hết thời gian chờ hàng loạt. Điều nguy hiểm là dịch vụ gọi tới chúng ta cũng đang chờ, nên chúng cũng cạn pool theo, và sự cố lan ngược lên toàn hệ thống. Đây là cơ chế của lỗi lan dây chuyền, và nó là nguyên nhân của rất nhiều sự cố lớn ngoài đời thật.

Chúng ta phòng bằng ba lớp, và chúng ta nên nói được cả ba. Thứ nhất, chúng ta đặt kích thước pool theo giới hạn kết nối thật của cơ sở dữ liệu, chứ không đặt pool thật to cho chắc. Pool quá lớn không làm hệ nhanh hơn, nó chỉ đẩy nhiều truy vấn xuống cơ sở dữ liệu hơn cho tới khi cơ sở dữ liệu gục. Thứ hai, chúng ta tách pool theo loại tải, để một luồng báo cáo nặng không nuốt hết kết nối của luồng giao dịch quan trọng. Thứ ba, chúng ta đặt cầu dao ngắt mạch, tức là khi tỷ lệ lỗi hoặc độ trễ vượt ngưỡng thì chúng ta cắt sớm và trả lỗi ngay thay vì để mọi thứ treo. Cắt sớm nghe có vẻ tàn nhẫn, nhưng nó giữ cho phần còn lại của hệ thống sống sót.

**English (bám cấu trúc tiếng Việt)**

Each service holds a pool of connections to the database, and that pool has a finite size. When the queries slow down, the connections are held for longer, so new requests have to queue up waiting to be given a connection. That waiting time adds to the user's latency, and when the queue grows long the requests start to time out in large numbers. The dangerous thing is that the services calling us are waiting too, so they run out of pool as well, and the incident spreads back up through the whole system. This is the mechanism of a cascading failure, and it is the cause of a great many large outages in real life.

We defend with three layers, and we should be able to say all three. First, we set the pool size according to the real connection limit of the database, rather than setting a very large pool just to be safe. A pool that is too large does not make the system faster, it only pushes more queries down to the database until the database collapses. Second, we split the pool by workload type, so that one heavy reporting path does not swallow all the connections of the important transactional path. Third, we put in a circuit breaker, that is, when the error rate or the latency crosses a threshold we cut early and return an error straight away instead of letting everything hang. Cutting early sounds cruel, but it keeps the rest of the system alive.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| có số lượng hữu hạn | has a finite size |
| các kết nối bị giữ lâu hơn | the connections are held for longer |
| xếp hàng chờ được cấp một kết nối | queue up waiting to be given a connection |
| hết thời gian chờ hàng loạt | time out in large numbers |
| chúng cũng cạn pool theo | they run out of pool as well |
| lan ngược lên toàn hệ thống | spreads back up through the whole system |
| không đặt pool thật to cho chắc | not setting a very large pool just to be safe |
| cho tới khi cơ sở dữ liệu gục | until the database collapses |
| nuốt hết kết nối của luồng giao dịch quan trọng | swallow all the connections of the important transactional path |
| vượt ngưỡng | crosses a threshold |
| cắt sớm nghe có vẻ tàn nhẫn | cutting early sounds cruel |

**Thuật ngữ cần nhớ**

- pool kết nối → **a connection pool**
- cạn kiệt / bão hoà pool → **pool saturation**
- lỗi lan dây chuyền → **a cascading failure**
- cầu dao ngắt mạch → **a circuit breaker**
- vách ngăn → **a bulkhead**
- hết thời gian chờ → **to time out**

---

## Phần 5 — Cache stampede và cách chặn

**Tiếng Việt**

Cache stampede xảy ra khi nhiều khoá cùng hết hạn tại một thời điểm, và toàn bộ request trượt cache cùng lúc đổ xuống cơ sở dữ liệu. Tình huống này hay xảy ra sau khi chúng ta triển khai lại dịch vụ, vì cache trong tiến trình bị xoá sạch và mọi thứ bắt đầu lại từ số không. Nó cũng hay xảy ra vào đầu giờ làm việc, khi cache đã nguội qua đêm còn người dùng thì ập vào cùng một lúc. Nhiều đội tưởng rằng thêm cache là xong chuyện, rồi ngạc nhiên khi hệ thống vẫn sập đều đặn lúc tám giờ sáng. Nguyên nhân không phải là cache vô dụng, mà là chúng ta chưa xử lý thời điểm cache nguội.

Có bốn biện pháp, và chúng thường được dùng cùng nhau chứ không thay thế nhau. Biện pháp thứ nhất là gộp các lần trượt cùng một khoá thành một lần tính duy nhất, nghĩa là chỉ một request đi xuống cơ sở dữ liệu còn những request khác chờ kết quả đó. Biện pháp thứ hai là rải ngẫu nhiên thời gian sống, ví dụ cộng thêm mười tới hai mươi phần trăm ngẫu nhiên, để các khoá không hết hạn cùng một giây. Biện pháp thứ ba là trả bản cũ trong lúc làm mới ở phía sau, nhờ đó người dùng không phải chờ lần tính lại. Biện pháp thứ tư là làm ấm cache trước giờ cao điểm, tức là chúng ta chủ động nạp những khoá quan trọng trước khi người dùng tới. Nếu chúng ta không làm gì cả, hậu quả là chính cơ chế cache trở thành thứ tạo ra đợt tải dồn định kỳ đánh sập hệ thống.

**English (bám cấu trúc tiếng Việt)**

A cache stampede happens when many keys expire at the same moment, and all the cache misses pour down onto the database at once. This situation often happens after we redeploy the service, because the in-process cache is wiped clean and everything starts again from zero. It also often happens at the start of the working day, when the cache has gone cold overnight while the users arrive all at the same time. Many teams think that adding a cache is the end of the story, and then they are surprised when the system still goes down regularly at eight in the morning. The cause is not that the cache is useless, it is that we have not handled the moment when the cache is cold.

There are four measures, and they are usually used together rather than as substitutes for each other. The first measure is to collapse the misses on the same key into one single computation, which means only one request goes down to the database while the other requests wait for that result. The second measure is to spread the time to live randomly, for example by adding a random ten to twenty percent, so that the keys do not expire in the same second. The third measure is to serve the stale copy while refreshing in the background, thanks to which the user does not have to wait for the recomputation. The fourth measure is to warm the cache before the peak hours, that is, we actively load the important keys before the users arrive. If we do nothing at all, the consequence is that the caching mechanism itself becomes the thing that creates a periodic load spike which knocks the system down.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| nhiều khoá cùng hết hạn tại một thời điểm | many keys expire at the same moment |
| toàn bộ request trượt cache cùng lúc đổ xuống | all the cache misses pour down at once |
| bị xoá sạch | is wiped clean |
| bắt đầu lại từ số không | starts again from zero |
| cache đã nguội qua đêm | the cache has gone cold overnight |
| tưởng rằng thêm cache là xong chuyện | think that adding a cache is the end of the story |
| gộp các lần trượt cùng một khoá thành một lần tính duy nhất | collapse the misses on the same key into one single computation |
| rải ngẫu nhiên thời gian sống | spread the time to live randomly |
| trả bản cũ trong lúc làm mới ở phía sau | serve the stale copy while refreshing in the background |
| làm ấm cache trước giờ cao điểm | warm the cache before the peak hours |
| đợt tải dồn định kỳ | a periodic load spike |

**Thuật ngữ cần nhớ**

- đổ xô vào cache → **a cache stampede** / **a thundering herd**
- trượt cache → **a cache miss**
- gộp về một lần gọi → **single-flight**
- rải ngẫu nhiên thời hạn → **jittered TTL**
- trả bản cũ trong lúc làm mới → **stale-while-revalidate**
- làm ấm trước → **to pre-warm**

---

## Phần 6 — Đo trước, đừng đoán

**Tiếng Việt**

Khi ai đó báo rằng hệ thống chậm ở môi trường thật, phản xạ đúng của chúng ta là đo trước chứ không sửa trước. Chúng ta mở bảng theo dõi, nhìn độ trễ p99 theo từng tầng, rồi tìm tầng nào là nơi thời gian bị tiêu nhiều nhất. Sau khi đã khoanh được tầng, chúng ta soi tiếp các chỉ số bên trong tầng đó, ví dụ độ sâu hàng đợi, mức dùng CPU, mức bão hoà pool kết nối và danh sách truy vấn chậm. Truy vết phân tán rất có ích ở bước này, vì nó cho chúng ta thấy một request đã đi qua những đâu và dừng lại ở đâu lâu nhất. Chỉ khi có bằng chứng, chúng ta mới đưa ra giả thuyết và mới sửa.

Cách làm ngược lại là đoán rồi vá, và đó là cách mà rất nhiều đội tiêu hết một tuần mà không khá hơn. Phản xạ phổ biến nhất là thêm máy chủ, nhưng thêm máy chủ không cứu được khoá nóng, không cứu được pool cạn và không cứu được khoá bị giữ lâu trong cơ sở dữ liệu. Trong ba trường hợp đó, thêm máy chỉ làm tăng số kết nối đi xuống cơ sở dữ liệu, nên nó đẩy hệ thống xuống nhanh hơn. Vì vậy, một kỹ sư dẫn dắt phải là người dừng cuộc tranh cãi lại và hỏi một câu duy nhất: chúng ta có số liệu nào cho thấy điều đó không. Nếu chúng ta không dựng được thói quen này trong đội, hậu quả là mọi sự cố đều được xử lý theo cảm tính, và không ai học được gì sau mỗi lần sập.

**English (bám cấu trúc tiếng Việt)**

When someone reports that the system is slow in the real environment, our correct reflex is to measure first rather than to fix first. We open the dashboard, look at the p99 latency for each layer, and then find which layer is the place where the time is spent the most. After we have narrowed it down to a layer, we look further at the metrics inside that layer, for example the queue depth, the CPU usage, the connection pool saturation and the list of slow queries. Distributed tracing is very useful at this step, because it shows us where a request has travelled and where it stopped for the longest. Only when we have evidence do we form a hypothesis and only then do we fix.

The opposite approach is to guess and then patch, and that is the approach in which many teams burn a whole week without getting any better. The most common reflex is to add servers, but adding servers does not save us from a hot key, it does not save us from pool saturation and it does not save us from locks held for a long time in the database. In those three cases, adding machines only increases the number of connections going down to the database, so it pushes the system down faster. Therefore, a leading engineer has to be the person who stops the argument and asks one single question: do we have any data showing that. If we cannot build this habit in the team, the consequence is that every incident is handled by gut feeling, and nobody learns anything after each outage.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| phản xạ đúng của chúng ta | our correct reflex |
| đo trước chứ không sửa trước | measure first rather than fix first |
| nơi thời gian bị tiêu nhiều nhất | the place where the time is spent the most |
| sau khi đã khoanh được tầng | after we have narrowed it down to a layer |
| độ sâu hàng đợi | the queue depth |
| dừng lại ở đâu lâu nhất | where it stopped for the longest |
| chỉ khi có bằng chứng | only when we have evidence |
| đoán rồi vá | guess and then patch |
| tiêu hết một tuần mà không khá hơn | burn a whole week without getting any better |
| khoá bị giữ lâu trong cơ sở dữ liệu | locks held for a long time in the database |
| chúng ta có số liệu nào cho thấy điều đó không | do we have any data showing that |

**Thuật ngữ cần nhớ**

- khả năng quan sát → **observability**
- truy vết phân tán → **distributed tracing**
- độ sâu hàng đợi → **queue depth**
- dựa trên số liệu → **data-driven**
- giả thuyết → **a hypothesis**
- sự cố ngừng dịch vụ → **an outage**

---

## Phần 7 — Cách nói về điểm nghẽn trong phỏng vấn

**Tiếng Việt**

Người phỏng vấn thường không mong chúng ta biết trước sự cố nào sẽ xảy ra, họ mong chúng ta có một cách nói có cấu trúc. Cấu trúc bốn phần rất hiệu quả và dễ nhớ: chỗ nào nghẽn, chúng ta thấy nó qua chỉ số nào, chúng ta chữa thế nào, và chúng ta trả giá gì. Ví dụ, chúng ta nói rằng đường ghi vào cơ sở dữ liệu sẽ nghẽn trước, chúng ta thấy nó qua thời gian chờ khoá và qua độ sâu hàng đợi. Sau đó chúng ta nói rằng chúng ta gộp ghi theo lô và đẩy phần không khẩn cấp sang hàng đợi, đổi lại người dùng sẽ thấy dữ liệu chậm vài giây. Bốn câu đó nói mất chưa tới ba mươi giây, nhưng chúng cho thấy chúng ta suy nghĩ như người vận hành hệ thống thật.

Chúng ta cũng nên chủ động nêu một chế độ hỏng mà thiết kế hiện tại chưa xử lý, thay vì chờ người phỏng vấn chỉ ra. Việc tự chỉ ra chỗ yếu của mình không làm chúng ta yếu đi trong mắt người nghe, mà ngược lại nó cho thấy sự trung thực kỹ thuật. Một cách mở lời an toàn là chúng ta nói rằng thiết kế này ổn ở tải bình thường, nhưng nó có một điểm sẽ vỡ trước tiên. Sau đó chúng ta gọi tên điểm đó và nói chúng ta sẽ theo dõi chỉ số nào để biết mình đang tiến gần tới nó. Nếu chúng ta trình bày một thiết kế như thể nó không có nhược điểm nào, hậu quả là người phỏng vấn sẽ dành phần còn lại của buổi để tìm cách chứng minh điều ngược lại.

**English (bám cấu trúc tiếng Việt)**

Interviewers usually do not expect us to know in advance which incident will happen, they expect us to have a structured way of speaking. A four-part structure is very effective and easy to remember: where the bottleneck is, which metric shows it to us, how we fix it, and what price we pay. For example, we say that the write path into the database will be the bottleneck first, and we see it through the lock wait time and through the queue depth. Then we say that we batch the writes and push the non-urgent part onto a queue, in exchange the user will see the data a few seconds late. Those four sentences take less than thirty seconds to say, but they show that we think like someone who really operates systems.

We should also volunteer one failure mode that the current design does not handle yet, rather than waiting for the interviewer to point it out. Pointing out our own weak spot does not make us look weaker to the listener, on the contrary it shows technical honesty. A safe way to open is that we say this design is fine at normal load, but it has one point that will break first. Then we name that point and say which metric we would watch in order to know that we are getting close to it. If we present a design as if it had no drawbacks at all, the consequence is that the interviewer will spend the rest of the session trying to prove the opposite.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| một cách nói có cấu trúc | a structured way of speaking |
| chúng ta thấy nó qua chỉ số nào | which metric shows it to us |
| chúng ta trả giá gì | what price we pay |
| đẩy phần không khẩn cấp sang hàng đợi | push the non-urgent part onto a queue |
| nói mất chưa tới ba mươi giây | take less than thirty seconds to say |
| chủ động nêu một chế độ hỏng | volunteer one failure mode |
| tự chỉ ra chỗ yếu của mình | pointing out our own weak spot |
| sự trung thực kỹ thuật | technical honesty |
| một điểm sẽ vỡ trước tiên | one point that will break first |
| đang tiến gần tới nó | getting close to it |
| như thể nó không có nhược điểm nào | as if it had no drawbacks at all |

**Thuật ngữ cần nhớ**

- chế độ hỏng → **a failure mode**
- chỉ số → **a metric**
- thời gian chờ khoá → **lock wait time**
- ngưỡng cảnh báo → **an alert threshold**
- nhược điểm → **a drawback**

---

## Mô hình ghi nhớ

**Tiếng Việt**

Điểm nghẽn nằm ở chỗ trạng thái tập trung hoặc chỗ tải dồn. Chúng ta đo trước khi sửa, chúng ta thử nấc rẻ trước nấc đắt, và chúng ta chặn lỗi lan dây chuyền bằng cầu dao ngắt mạch.

**English (bám cấu trúc tiếng Việt)**

The bottleneck sits where the state is centralised or where the load piles up. We measure before we fix, we try the cheap rung before the expensive one, and we stop cascading failures with a circuit breaker.

---

## Bảng thuật ngữ tổng hợp

| Tiếng Việt | Tiếng Anh | Ghi chú phát âm |
|---|---|---|
| điểm nghẽn | a bottleneck | |
| mức đồng thời | concurrency | cần-**CA**-rợn-si, trọng âm âm thứ hai |
| trạng thái tập trung | centralised state | |
| bão hoà | to saturate / saturation | *saturation* — sa-chơ-**RÂY**-shợn, trọng âm âm thứ ba |
| gộp theo lô | to batch | |
| chỉ mục | an index | |
| bản sao chỉ đọc | a read replica | *replica* Anh /ˈreplɪkə/ — "**RE**-pli-cơ", trọng âm đầu |
| phân mảnh | sharding | *shard* /ʃɑːd/ — âm đầu là "sh" không phải "s" |
| khoá phân mảnh | a shard key | |
| tách đọc ghi | CQRS | đọc từng chữ cái: "si-kiu-a-es" |
| truy vấn chậm | a slow query | *query* /ˈkwɪəri/ — "KUY-ơ-ri" |
| khoá nóng | a hot key | |
| phân vùng nóng | a hot partition | |
| lệch tải | skew | /skjuː/ — "skiu", không đọc "sờ-kiu-ơ" |
| độ phân tán | cardinality | ca-đi-**NA**-li-ti, trọng âm âm thứ ba |
| thêm muối | salting | |
| độ trễ đuôi | tail latency | *latency* /ˈleɪtənsi/ — "LÂY-tân-si" |
| bách phân vị 99 | the 99th percentile | đọc là "the ninety-ninth per-CEN-tile"; *p99* đọc "p ninety-nine" |
| pool kết nối | a connection pool | |
| bão hoà pool | pool saturation | |
| lỗi lan dây chuyền | a cascading failure | *cascading* — cơ-**SKÂY**-đing, trọng âm âm thứ hai |
| cầu dao ngắt mạch | a circuit breaker | *circuit* /ˈsɜːkɪt/ — "**SƠ**-kịt", chữ *u* không đọc |
| vách ngăn | a bulkhead | |
| hết thời gian chờ | to time out | |
| ngưỡng | a threshold | âm **th** /θ/ đầu, "THRESH-hâuld" |
| đổ xô vào cache | a cache stampede | *stampede* — stam-**PIID**, trọng âm âm **cuối** |
| bầy giẫm đạp | a thundering herd | *herd* /hɜːd/ — "hớđ", nghe gần giống *heard* |
| trượt cache | a cache miss | *cache* /kæʃ/ — đọc đúng như "cash" |
| gộp về một lần gọi | single-flight | |
| rải ngẫu nhiên thời hạn | jittered TTL | *jitter* — "**JI**-tơ", âm đầu là "j" |
| trả bản cũ trong lúc làm mới | stale-while-revalidate | *stale* /steɪl/ — "stêi-l" |
| làm ấm trước | to pre-warm | |
| khả năng quan sát | observability | ợb-zơ-vơ-**BI**-li-ti, trọng âm âm thứ tư |
| truy vết phân tán | distributed tracing | |
| độ sâu hàng đợi | queue depth | *queue* /kjuː/ đọc như chữ "Q"; *depth* có âm **th** cuối |
| dựa trên số liệu | data-driven | |
| giả thuyết | a hypothesis | hai-**PO**-thơ-sịs, trọng âm âm thứ hai; số nhiều *hypotheses* đọc "-si-iz" |
| sự cố ngừng dịch vụ | an outage | "**AU**-tịj", trọng âm đầu |
| chế độ hỏng | a failure mode | |
| chỉ số | a metric | |
| thời gian chờ khoá | lock wait time | |
| nhược điểm | a drawback | |
| khắc phục sự cố | troubleshooting | |
| máy chủ | host | âm cuối **-st** phải bật ra, không thành "hâu" |

---

## Bài nói

> **Cách luyện:** bật ghi âm, nói **60–90 giây** cho mỗi đề, **không nhìn tài liệu** khi nói. Mỗi bài phải dùng ít nhất **sáu thuật ngữ** trong bảng trên. Với các đề mô tả sự cố, hãy cố dùng đúng cấu trúc bốn phần ở Phần 7: chỗ nghẽn, chỉ số, cách chữa, cái giá.

1. **Explain to a junior engineer** how you decide where to look first for the bottleneck in a design, and why "we just need more servers" is not an answer.

2. **A colleague says:** *"We added Redis, so the database is safe now."* **Explain what is wrong with that**, using what happens at eight in the morning when the cache is cold.

3. **Someone on your team wants to** shard the database because the load has doubled this quarter. **Explain why you would push back**, and describe the cheaper rungs you would try first and what evidence would change your mind.

4. **Someone on your team wants to** raise the connection pool size from 20 to 200 to fix a timeout problem. **Explain why you would push back**, and say what you would measure instead.

5. **Describe what happens when** one shard holds a celebrity account with millions of followers. Say which metric reveals it and give two different fixes with their trade-offs.

6. **Describe what happens when** a downstream service slows down and every caller keeps waiting instead of failing fast. Explain how a circuit breaker changes that story.

7. **Explain to a manager**, without jargon, why the team needs half a day to add tracing before they can promise a fix for the slowness reported yesterday.
