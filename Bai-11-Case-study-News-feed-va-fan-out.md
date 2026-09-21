# Bài 11 — Case study: News feed & fan-out
### Bản song ngữ: tiếng Anh bám cấu trúc tiếng Việt

> **Cách dùng:** đọc đoạn tiếng Việt trước → tự dịch trong đầu → so với bản tiếng Anh bên dưới → **đọc to bản tiếng Anh hai lần**. Lần một đọc chậm cho đúng từ, lần hai đọc liền mạch như đang nói với người phỏng vấn. Bài này có một câu chuyện rõ ràng: một thiết kế đúng ở trường hợp thường lại vỡ khi gặp tài khoản triệu người theo dõi, và cách chúng ta kể câu chuyện đó chính là thứ được chấm.

---

## Phần 1 — Hai hướng trái ngược: đẩy và kéo

**Tiếng Việt**

Câu hỏi trung tâm của một bảng tin là khi một người đăng bài, làm sao những người theo dõi họ nhìn thấy bài đó thật nhanh. Có hai hướng trả lời hoàn toàn trái ngược nhau. Hướng thứ nhất là đẩy, nghĩa là ngay lúc đăng, chúng ta ghi bài đó vào hộp thư của từng người theo dõi. Hướng thứ hai là kéo, nghĩa là chúng ta không làm gì lúc đăng, và chỉ khi người dùng mở ứng dụng thì chúng ta mới gom bài từ những người mà họ theo dõi. Cả hai hướng đều đúng, và việc chọn hướng nào phụ thuộc vào hình dạng dữ liệu chứ không phụ thuộc vào sở thích.

Đánh đổi giữa hai hướng rất rõ ràng và chúng ta nên phát biểu nó thành một câu gọn. Hướng đẩy làm cho việc đọc cực nhanh, vì bảng tin đã được tính sẵn trước khi người dùng mở lên. Đổi lại, hướng đẩy làm việc ghi trở nên rất nặng, vì một bài đăng biến thành hàng nghìn lượt ghi, và nó tốn dung lượng vì cùng một mã bài được nhân bản ở nhiều nơi. Hướng kéo thì ngược lại: việc ghi rất nhẹ vì chúng ta chỉ lưu bài một lần, nhưng việc đọc chậm vì mỗi lần mở ứng dụng chúng ta phải gom và sắp xếp lại. Nói ngắn gọn, chúng ta chọn trả giá lúc ghi hay trả giá lúc đọc, và chúng ta chọn dựa trên tỷ lệ đọc trên ghi.

**English (bám cấu trúc tiếng Việt)**

The central question of a news feed is how, when one person posts, their followers see that post very fast. There are two completely opposite directions for answering. The first direction is push, which means that at the moment of posting, we write that post into the inbox of every single follower. The second direction is pull, which means we do nothing at posting time, and only when the user opens the app do we gather the posts from the people they follow. Both directions are correct, and choosing between them depends on the shape of the data rather than on preference.

The trade-off between the two directions is very clear and we should state it in one compact sentence. The push direction makes reading extremely fast, because the feed has already been computed before the user opens it. In exchange, the push direction makes writing very heavy, because one post turns into thousands of writes, and it costs storage because the same post id is duplicated in many places. The pull direction is the opposite: writing is very light because we store the post only once, but reading is slow because every time the app opens we have to gather and sort again. Put briefly, we choose whether to pay at write time or to pay at read time, and we choose based on the read-to-write ratio.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| những người theo dõi họ | their followers |
| hai hướng trả lời hoàn toàn trái ngược nhau | two completely opposite directions for answering |
| ghi bài đó vào hộp thư của từng người theo dõi | write that post into the inbox of every single follower |
| chúng ta mới gom bài từ những người mà họ theo dõi | we gather the posts from the people they follow |
| phụ thuộc vào hình dạng dữ liệu | depends on the shape of the data |
| đã được tính sẵn | has already been computed |
| một bài đăng biến thành hàng nghìn lượt ghi | one post turns into thousands of writes |
| được nhân bản ở nhiều nơi | is duplicated in many places |
| chúng ta phải gom và sắp xếp lại | we have to gather and sort again |
| trả giá lúc ghi hay trả giá lúc đọc | pay at write time or pay at read time |

**Thuật ngữ cần nhớ**

- bảng tin → **a news feed**
- phát tán lúc ghi → **fan-out on write** / **push**
- gom lúc đọc → **fan-out on read** / **pull**
- người theo dõi → **a follower**
- tính sẵn → **precomputed**

---

## Phần 2 — Bài toán người nổi tiếng

**Tiếng Việt**

Hướng đẩy hoạt động rất tốt cho người dùng bình thường, nhưng nó vỡ hoàn toàn khi gặp một tài khoản có hàng triệu người theo dõi. Khi một tài khoản như vậy đăng một bài, chúng ta phải thực hiện hàng triệu lượt ghi chỉ cho một hành động duy nhất. Việc đó tạo ra một cơn bão ghi, làm nghẽn hàng đợi, và làm những người theo dõi đầu tiên thấy bài sau vài giây còn những người cuối cùng thấy bài sau vài phút. Tệ hơn nữa, nếu tài khoản đó đăng nhiều bài liên tiếp, các đợt phát tán chồng lên nhau và hệ thống không bao giờ đuổi kịp. Đây chính là bài toán người nổi tiếng, và nó là câu hỏi phân biệt ứng viên cấp cao với ứng viên trung cấp.

Lời giải được chấp nhận rộng rãi là một cách làm lai giữa hai hướng. Với người dùng bình thường, chúng ta vẫn đẩy bài vào bảng tin của từng người theo dõi như cũ. Với những tài khoản vượt một ngưỡng người theo dõi nhất định, chúng ta không phát tán gì cả, mà chỉ lưu bài vào dòng thời gian của chính tác giả. Khi một người mở ứng dụng, chúng ta lấy phần đã tính sẵn của họ, rồi kéo thêm bài mới từ những người nổi tiếng mà họ đang theo dõi, và trộn hai nguồn đó lại trước khi trả về. Bước trộn này là bước dễ quên nhất, và nếu chúng ta quên nó thì người dùng sẽ không bao giờ thấy bài của người nổi tiếng trong bảng tin của mình.

**English (bám cấu trúc tiếng Việt)**

The push direction works very well for ordinary users, but it breaks completely when it meets an account with millions of followers. When such an account publishes one post, we have to perform millions of writes for one single action. That creates a write storm, clogs up the queue, and makes the first followers see the post after a few seconds while the last ones see it after a few minutes. Worse still, if that account posts several times in a row, the fan-out waves pile on top of each other and the system never catches up. This is exactly the celebrity problem, and it is the question that separates a senior candidate from a mid-level one.

The widely accepted solution is a hybrid approach between the two directions. For ordinary users, we still push the post into each follower's feed as before. For accounts above a certain follower threshold, we do not fan out at all, we only store the post in the author's own timeline. When a person opens the app, we take their precomputed part, then pull the recent posts from the celebrities they follow, and merge the two sources before returning. This merge step is the easiest one to forget, and if we forget it then users will never see the celebrities' posts in their own feed.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| nó vỡ hoàn toàn khi gặp | it breaks completely when it meets |
| chỉ cho một hành động duy nhất | for one single action |
| tạo ra một cơn bão ghi | creates a write storm |
| làm nghẽn hàng đợi | clogs up the queue |
| tệ hơn nữa | worse still |
| các đợt phát tán chồng lên nhau | the fan-out waves pile on top of each other |
| hệ thống không bao giờ đuổi kịp | the system never catches up |
| một cách làm lai giữa hai hướng | a hybrid approach between the two directions |
| vượt một ngưỡng người theo dõi nhất định | above a certain follower threshold |
| dòng thời gian của chính tác giả | the author's own timeline |
| bước dễ quên nhất | the easiest step to forget |

**Thuật ngữ cần nhớ**

- bài toán người nổi tiếng → **the celebrity problem**
- cơn bão ghi → **a write storm**
- cách làm lai → **a hybrid approach**
- ngưỡng → **a threshold**
- trộn lúc đọc → **merge at read time**

---

## Phần 3 — Bảng tin lưu ở đâu và lưu cái gì

**Tiếng Việt**

Chúng ta lưu bảng tin đã tính sẵn trong một kho nhanh như Redis, mỗi người dùng một danh sách riêng. Điều quan trọng là chúng ta chỉ lưu mã của bài viết, chứ không lưu nội dung của bài viết trong danh sách đó. Cách này tiết kiệm bộ nhớ rất nhiều, vì một mã chỉ chiếm vài byte trong khi một bài viết có thể chiếm vài kilobyte. Quan trọng hơn, khi tác giả sửa nội dung hoặc xoá bài, chúng ta không phải viết lại bảng tin của hàng nghìn người. Lúc phục vụ, chúng ta lấy danh sách mã rồi nạp nội dung thật từ kho bài viết, và bước đó thường được gọi là làm đầy nội dung.

Chúng ta cũng giới hạn độ dài của mỗi bảng tin, ví dụ chỉ giữ vài trăm mục gần nhất. Lý do là gần như không ai cuộn quá vài chục bài trong một phiên, nên việc giữ toàn bộ lịch sử trong bộ nhớ là lãng phí. Nếu một người dùng cuộn sâu hơn phần đã tính sẵn, chúng ta rơi về cách kéo và gom từ cơ sở dữ liệu, và điều đó chấp nhận được vì trường hợp này rất hiếm. Cách trình bày ghi điểm là chúng ta nói rõ rằng bộ nhớ nhanh chỉ phục vụ phần nóng, còn phần lạnh thì để ở kho rẻ hơn. Nếu chúng ta lưu toàn bộ nội dung cho mọi người dùng, hậu quả là chi phí bộ nhớ tăng theo cấp số nhân trong khi phần lớn dữ liệu đó không bao giờ được đọc tới.

**English (bám cấu trúc tiếng Việt)**

We store the precomputed feed in a fast store such as Redis, one separate list per user. The important thing is that we only store the post ids, we do not store the post content inside that list. This way saves a lot of memory, because one id takes a few bytes while one post may take a few kilobytes. More importantly, when the author edits the content or deletes the post, we do not have to rewrite the feeds of thousands of people. At serving time, we take the list of ids and then load the real content from the post store, and that step is usually called hydration.

We also limit the length of each feed, for example keeping only a few hundred of the most recent items. The reason is that almost nobody scrolls past a few dozen posts in one session, so keeping the whole history in memory is wasteful. If a user scrolls deeper than the precomputed part, we fall back to pulling and gathering from the database, and that is acceptable because this case is very rare. The way of presenting that scores is that we say clearly that the fast memory only serves the hot part, while the cold part stays in a cheaper store. If we store the whole content for every user, the consequence is that the memory cost grows exponentially while most of that data is never read at all.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| mỗi người dùng một danh sách riêng | one separate list per user |
| chúng ta chỉ lưu mã của bài viết | we only store the post ids |
| khi tác giả sửa nội dung hoặc xoá bài | when the author edits the content or deletes the post |
| viết lại bảng tin của hàng nghìn người | rewrite the feeds of thousands of people |
| nạp nội dung thật từ kho bài viết | load the real content from the post store |
| bước đó thường được gọi là làm đầy nội dung | that step is usually called hydration |
| giữ vài trăm mục gần nhất | keeping a few hundred of the most recent items |
| cuộn sâu hơn phần đã tính sẵn | scrolls deeper than the precomputed part |
| chúng ta rơi về cách kéo | we fall back to pulling |
| bộ nhớ nhanh chỉ phục vụ phần nóng | the fast memory only serves the hot part |

**Thuật ngữ cần nhớ**

- kho bảng tin → **the feed store**
- làm đầy nội dung → **hydration**
- cắt bớt danh sách → **to trim**
- rơi về phương án dự phòng → **to fall back**
- dữ liệu nóng và lạnh → **hot and cold data**

---

## Phần 4 — Xếp hạng không chỉ theo thời gian

**Tiếng Việt**

Bảng tin sớm nhất chỉ sắp xếp theo thời gian, nhưng các sản phẩm hiện nay đều xếp hạng theo mức độ liên quan. Khi đó, mỗi bài viết cần một điểm số được tính từ nhiều đặc trưng, ví dụ mức độ thân thiết với tác giả, chủ đề mà người dùng hay tương tác, và độ mới của bài. Việc tính điểm này quá nặng để làm trong lúc phục vụ request, nên chúng ta tách nó ra thành một dịch vụ xếp hạng riêng. Dịch vụ đó tính điểm theo lô ở phía sau, và lưu kết quả sẵn để lúc đọc chúng ta chỉ việc lấy ra. Nhờ vậy, đường phục vụ vẫn nhanh dù mô hình xếp hạng có phức tạp đến đâu.

Ở đây có một đánh đổi mà chúng ta nên nói ra: giữa tính tức thời và chất lượng xếp hạng. Nếu chúng ta tính điểm sẵn hoàn toàn, bảng tin sẽ rất nhanh nhưng nó không phản ứng kịp với những gì vừa xảy ra vài phút trước. Nếu chúng ta tính điểm ngay lúc đọc, thứ hạng sẽ tươi mới nhưng độ trễ tăng và chi phí tính toán cũng tăng theo. Cách phổ biến là chia hai giai đoạn: một bước chọn ứng viên rẻ và rộng chạy trước, rồi một bước xếp hạng tinh hơn chỉ chạy trên vài trăm ứng viên đó. Nếu chúng ta cố xếp hạng toàn bộ kho bài viết trong mỗi request, hậu quả là chúng ta xây một hệ thống rất thông minh nhưng không ai đủ kiên nhẫn chờ nó trả lời.

**English (bám cấu trúc tiếng Việt)**

The earliest feeds only sorted by time, but products nowadays all rank by relevance. In that case, each post needs a score computed from many features, for example how close the user is to the author, the topics the user usually engages with, and how recent the post is. Computing this score is too heavy to do while serving a request, so we split it out into a separate ranking service. That service computes scores in batches at the back, and stores the results ready so that at read time we simply take them out. Thanks to that, the serving path stays fast no matter how complex the ranking model becomes.

There is a trade-off here that we should say out loud: between freshness and ranking quality. If we precompute the scores completely, the feed will be very fast but it will not react in time to what happened a few minutes ago. If we compute the scores at read time, the ranking will be fresh but the latency goes up and the compute cost goes up with it. The common approach is to split it into two stages: a cheap and broad candidate selection step runs first, and then a finer ranking step runs on only those few hundred candidates. If we try to rank the entire post store on every request, the consequence is that we build a very clever system that nobody is patient enough to wait for.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| xếp hạng theo mức độ liên quan | rank by relevance |
| một điểm số được tính từ nhiều đặc trưng | a score computed from many features |
| mức độ thân thiết với tác giả | how close the user is to the author |
| chủ đề mà người dùng hay tương tác | the topics the user usually engages with |
| quá nặng để làm trong lúc phục vụ request | too heavy to do while serving a request |
| tính điểm theo lô ở phía sau | computes scores in batches at the back |
| giữa tính tức thời và chất lượng xếp hạng | between freshness and ranking quality |
| nó không phản ứng kịp | it does not react in time |
| một bước chọn ứng viên rẻ và rộng | a cheap and broad candidate selection step |
| không ai đủ kiên nhẫn chờ nó trả lời | nobody is patient enough to wait for it |

**Thuật ngữ cần nhớ**

- dịch vụ xếp hạng → **a ranking service**
- mức độ liên quan → **relevance**
- đặc trưng → **a feature**
- chọn ứng viên → **candidate selection**
- tính tươi mới → **freshness**

---

## Phần 5 — Mức nhất quán mà bảng tin cần

**Tiếng Việt**

Bảng tin là một trong số ít hệ thống mà chúng ta có thể nói thẳng rằng nhất quán cuối cùng là đủ. Nếu một người dùng thấy bài của bạn mình chậm ba giây, không có thiệt hại nào xảy ra và thường thì không ai nhận ra. Điều này khác hẳn với tiền bạc hay vé máy bay, nơi một câu trả lời cũ tạo ra sai lệch thật về nghiệp vụ. Vì vậy, chúng ta ưu tiên độ trễ thấp cùng tính sẵn sàng cao, và chúng ta chấp nhận rằng các bản sao hội tụ sau vài giây. Đây là chỗ chúng ta nên chủ động phát biểu lựa chọn, vì nó cho thấy chúng ta gắn mức nhất quán với hậu quả nghiệp vụ chứ không chọn theo mặc định.

Tuy nhiên, có một ngoại lệ mà chúng ta nên nêu để câu trả lời được cân bằng. Người dùng rất nhạy với chính hành động của họ, nên nếu họ vừa đăng một bài mà không thấy nó xuất hiện ngay, họ sẽ nghĩ hệ thống bị lỗi. Cách xử lý phổ biến là chúng ta chèn bài của chính người đó vào bảng tin ngay tại giao diện, trong lúc quá trình phát tán vẫn chạy ở phía sau. Nguyên tắc này thường được gọi là đọc thấy chính cái mình vừa ghi, và nó chỉ áp dụng cho tác giả chứ không cho tất cả. Nếu chúng ta không xử lý riêng trường hợp này, hậu quả là bộ phận hỗ trợ nhận được rất nhiều báo lỗi về những bài viết bị mất mà thật ra chúng chỉ đang trên đường.

**English (bám cấu trúc tiếng Việt)**

The news feed is one of the few systems where we can say straight out that eventual consistency is enough. If a user sees their friend's post three seconds late, no damage occurs and usually nobody notices. This is completely different from money or plane tickets, where one stale answer creates a real business discrepancy. Therefore, we prioritise low latency together with high availability, and we accept that the replicas converge after a few seconds. This is the place where we should volunteer our choice, because it shows that we tie the consistency level to the business consequence rather than choosing by default.

However, there is one exception that we should mention so that the answer stays balanced. Users are very sensitive to their own actions, so if they have just posted something and do not see it appear immediately, they will think the system is broken. The common way to handle this is that we insert the person's own post into the feed right at the interface, while the fan-out process still runs at the back. This principle is usually called read-your-own-writes, and it applies only to the author rather than to everybody. If we do not handle this case separately, the consequence is that the support team receives a lot of bug reports about missing posts which are in fact merely on their way.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| chúng ta có thể nói thẳng rằng | we can say straight out that |
| không có thiệt hại nào xảy ra | no damage occurs |
| tạo ra sai lệch thật về nghiệp vụ | creates a real business discrepancy |
| các bản sao hội tụ sau vài giây | the replicas converge after a few seconds |
| chúng ta nên chủ động phát biểu lựa chọn | we should volunteer our choice |
| rất nhạy với chính hành động của họ | very sensitive to their own actions |
| chèn bài của chính người đó vào bảng tin ngay tại giao diện | insert the person's own post into the feed right at the interface |
| đọc thấy chính cái mình vừa ghi | read-your-own-writes |
| những bài viết bị mất | missing posts |
| mà thật ra chúng chỉ đang trên đường | which are in fact merely on their way |

**Thuật ngữ cần nhớ**

- nhất quán cuối cùng → **eventual consistency**
- đọc thấy cái mình vừa ghi → **read-your-own-writes**
- sai lệch → **a discrepancy**
- hội tụ → **to converge**
- cập nhật lạc quan trên giao diện → **an optimistic update**

---

## Phần 6 — Phân trang bằng con trỏ, không dùng độ lệch

**Tiếng Việt**

Với cuộn vô tận, chúng ta phải dùng con trỏ chứ không dùng độ lệch, và đây là chi tiết mà nhiều người bỏ qua. Phân trang theo độ lệch nghĩa là chúng ta bảo cơ sở dữ liệu bỏ qua một nghìn dòng đầu rồi lấy hai mươi dòng tiếp theo. Cách này chậm dần khi người dùng cuộn sâu, vì cơ sở dữ liệu vẫn phải duyệt qua đúng một nghìn dòng đó trước khi bỏ chúng đi. Tệ hơn, nếu có bài mới được chèn lên đầu trong lúc người dùng đang cuộn, toàn bộ danh sách dịch xuống một vị trí. Kết quả là người dùng thấy một bài xuất hiện hai lần, hoặc một bài bị nhảy qua hoàn toàn.

Phân trang theo con trỏ tránh được cả hai vấn đề trên. Thay vì nói bỏ qua một nghìn dòng, chúng ta nói lấy hai mươi bài cũ hơn bài có mã này. Cơ sở dữ liệu nhảy thẳng tới vị trí đó bằng chỉ mục, nên chi phí không tăng theo độ sâu cuộn. Ngoài ra, việc chèn bài mới ở đầu danh sách không ảnh hưởng gì tới con trỏ, vì con trỏ neo vào một mục cụ thể chứ không neo vào một vị trí đếm được. Trong phỏng vấn, chúng ta nên nói rõ rằng con trỏ thường là cặp gồm dấu thời gian và mã bài, vì chỉ dấu thời gian thôi sẽ mơ hồ khi hai bài trùng thời điểm.

**English (bám cấu trúc tiếng Việt)**

For infinite scrolling, we have to use a cursor rather than an offset, and this is a detail that many people skip. Offset pagination means that we tell the database to skip the first thousand rows and then take the next twenty rows. This way gets slower as the user scrolls deeper, because the database still has to walk through exactly those thousand rows before throwing them away. Worse, if a new post is inserted at the top while the user is scrolling, the whole list shifts down by one position. The result is that the user sees one post appear twice, or one post skipped over completely.

Cursor pagination avoids both of the problems above. Instead of saying skip a thousand rows, we say take twenty posts older than the post with this id. The database jumps straight to that position using the index, so the cost does not grow with the scroll depth. In addition, inserting a new post at the top of the list does not affect the cursor at all, because the cursor is anchored to a specific item rather than to a countable position. In an interview, we should say clearly that the cursor is usually a pair made of a timestamp and a post id, because a timestamp alone would be ambiguous when two posts share the same moment.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| cuộn vô tận | infinite scrolling |
| bỏ qua một nghìn dòng đầu | skip the first thousand rows |
| chậm dần khi người dùng cuộn sâu | gets slower as the user scrolls deeper |
| trước khi bỏ chúng đi | before throwing them away |
| toàn bộ danh sách dịch xuống một vị trí | the whole list shifts down by one position |
| một bài bị nhảy qua hoàn toàn | one post skipped over completely |
| lấy hai mươi bài cũ hơn bài có mã này | take twenty posts older than the post with this id |
| nhảy thẳng tới vị trí đó bằng chỉ mục | jumps straight to that position using the index |
| không tăng theo độ sâu cuộn | does not grow with the scroll depth |
| con trỏ neo vào một mục cụ thể | the cursor is anchored to a specific item |
| sẽ mơ hồ khi hai bài trùng thời điểm | would be ambiguous when two posts share the same moment |

**Thuật ngữ cần nhớ**

- phân trang → **pagination**
- con trỏ → **a cursor**
- độ lệch → **an offset**
- cuộn vô tận → **infinite scroll**
- neo vào → **to anchor to**

---

## Phần 7 — Ghép tất cả lại thành một đường đi hoàn chỉnh

**Tiếng Việt**

Chúng ta nên kể lại toàn bộ thiết kế bằng cách đi theo một bài viết từ lúc sinh ra tới lúc xuất hiện trên màn hình. Khi tác giả bấm đăng, dịch vụ bài viết lưu nội dung vào kho chính rồi trả về ngay cho người dùng, và nó không chờ việc phát tán. Cùng lúc đó, nó gửi một thông điệp vào hàng đợi, và một luồng xử lý phía sau quyết định đẩy hay không đẩy dựa trên số người theo dõi của tác giả. Nếu tác giả là người dùng thường, luồng đó ghi mã bài vào danh sách của từng người theo dõi rồi cắt bớt danh sách cho gọn. Nếu tác giả là người nổi tiếng, luồng đó không làm gì cả ngoài việc bảo đảm bài đã nằm trong dòng thời gian của tác giả.

Ở chiều đọc, khi người dùng kéo bảng tin xuống, chúng ta lấy phần đã tính sẵn của họ trước. Sau đó chúng ta kéo thêm bài mới từ những người nổi tiếng mà họ theo dõi, trộn hai nguồn lại, rồi áp điểm xếp hạng đã tính sẵn để sắp thứ tự. Tiếp theo chúng ta nạp nội dung thật cho khoảng hai mươi mã đầu tiên và trả về kèm một con trỏ cho lần cuộn sau. Toàn bộ đường đọc này chỉ chạm vào bộ nhớ nhanh và một lần nạp nội dung, nên nó xong trong vài chục mili giây. Nếu chúng ta trình bày được đúng hai đường đi này một cách mạch lạc, chúng ta đã trả lời trọn vẹn phần lớn những gì người phỏng vấn muốn nghe.

**English (bám cấu trúc tiếng Việt)**

We should retell the whole design by following one post from the moment it is born to the moment it appears on a screen. When the author presses publish, the post service saves the content into the main store and returns to the user immediately, and it does not wait for the fan-out. At the same time, it sends a message into the queue, and a worker at the back decides whether to push or not to push based on the author's follower count. If the author is an ordinary user, that worker writes the post id into each follower's list and then trims the list to keep it short. If the author is a celebrity, that worker does nothing except making sure the post is already in the author's timeline.

On the read side, when the user pulls the feed down, we take their precomputed part first. Then we pull the recent posts from the celebrities they follow, merge the two sources, and apply the precomputed ranking scores in order to sort them. Next we load the real content for about the first twenty ids and return it together with a cursor for the next scroll. This whole read path only touches the fast memory and one content load, so it finishes within a few tens of milliseconds. If we can present exactly these two paths coherently, we have fully answered most of what the interviewer wants to hear.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| đi theo một bài viết từ lúc sinh ra | following one post from the moment it is born |
| khi tác giả bấm đăng | when the author presses publish |
| nó không chờ việc phát tán | it does not wait for the fan-out |
| quyết định đẩy hay không đẩy | decides whether to push or not to push |
| rồi cắt bớt danh sách cho gọn | and then trims the list to keep it short |
| không làm gì cả ngoài việc bảo đảm | does nothing except making sure |
| khi người dùng kéo bảng tin xuống | when the user pulls the feed down |
| áp điểm xếp hạng đã tính sẵn | apply the precomputed ranking scores |
| kèm một con trỏ cho lần cuộn sau | together with a cursor for the next scroll |
| trình bày được đúng hai đường đi này một cách mạch lạc | present exactly these two paths coherently |

**Thuật ngữ cần nhớ**

- đường ghi và đường đọc → **the write path and the read path**
- luồng xử lý nền → **a background worker**
- số người theo dõi → **the follower count**
- dòng thời gian → **a timeline**
- kho chính → **the main store**

---

## Mô hình ghi nhớ

**Tiếng Việt**

Chúng ta đẩy cho người dùng thường, chúng ta kéo cho người nổi tiếng, và chúng ta trộn hai nguồn lúc đọc. Chúng ta lưu mã bài rồi nạp nội dung sau, chúng ta phân trang bằng con trỏ chứ không bằng độ lệch, và chúng ta chấp nhận nhất quán cuối cùng vì trễ vài giây không gây hại.

**English (bám cấu trúc tiếng Việt)**

We push for ordinary users, we pull for celebrities, and we merge the two sources at read time. We store post ids and load the content later, we paginate with a cursor rather than with an offset, and we accept eventual consistency because a few seconds of delay does no harm.

---

## Bảng thuật ngữ tổng hợp

| Tiếng Việt | Tiếng Anh | Ghi chú phát âm |
|---|---|---|
| bảng tin | a news feed | |
| phát tán | fan-out | |
| phát tán lúc ghi | fan-out on write / push | |
| gom lúc đọc | fan-out on read / pull | |
| người theo dõi | a follower | |
| số người theo dõi | the follower count | |
| tính sẵn | precomputed | |
| bài toán người nổi tiếng | the celebrity problem | *celebrity* — sơ-**LE**-brơ-ti, trọng âm âm thứ hai |
| cơn bão ghi | a write storm | *write* — chữ **w câm**, đọc như *right* |
| cách làm lai | a hybrid approach | *hybrid* — "**HAI**-brịd", trọng âm đầu |
| ngưỡng | a threshold | âm **th** /θ/ đầu, "THRESH-hâuld" |
| trộn lúc đọc | merge at read time | |
| kho bảng tin | the feed store | |
| làm đầy nội dung | hydration | hai-**ĐRÂY**-shợn, trọng âm âm thứ hai |
| cắt bớt danh sách | to trim | |
| rơi về phương án dự phòng | to fall back | |
| dữ liệu nóng và lạnh | hot and cold data | |
| dịch vụ xếp hạng | a ranking service | |
| mức độ liên quan | relevance | "**RE**-lơ-vợns", trọng âm đầu |
| đặc trưng | a feature | |
| chọn ứng viên | candidate selection | *candidate* — "**CAN**-đi-đợt", trọng âm đầu |
| tính tươi mới | freshness | |
| nhất quán cuối cùng | eventual consistency | *eventual* — i-**VEN**-chu-ợl, trọng âm âm thứ hai |
| đọc thấy cái mình vừa ghi | read-your-own-writes | |
| sai lệch | a discrepancy | đis-**CRE**-pợn-si, trọng âm âm thứ hai |
| hội tụ | to converge | cần-**VƠJ**, trọng âm âm thứ hai |
| cập nhật lạc quan | an optimistic update | |
| phân trang | pagination | pa-jị-**NÂY**-shợn, trọng âm âm thứ ba |
| con trỏ | a cursor | "**CƠ**-sơ", trọng âm đầu |
| độ lệch | an offset | danh từ nhấn đầu: "**O**-fset" |
| cuộn vô tận | infinite scroll | *infinite* — "**IN**-fơ-nợt", trọng âm đầu, không đọc "in-FAI-nait" |
| neo vào | to anchor to | *anchor* /ˈæŋkə/ — "**ANG**-cơ", chữ **ch** đọc là /k/ |
| đường ghi và đường đọc | the write path and the read path | *path* có âm **th** cuối |
| luồng xử lý nền | a background worker | |
| dòng thời gian | a timeline | |
| kho chính | the main store | |
| hàng đợi | a queue | /kjuː/ — đọc như chữ "Q" |
| dấu thời gian | a timestamp | âm cuối **-mp** phải bật ra |
| chỉ mục | an index | số nhiều kỹ thuật thường dùng *indexes* |
| khoá nóng | a hot key | |

---

## Bài nói

> **Cách luyện:** bật ghi âm, nói **60–90 giây** cho mỗi đề, **không nhìn tài liệu** khi nói. Mỗi bài phải dùng ít nhất **sáu thuật ngữ** trong bảng trên. Đề số 7 là đề tổng hợp — hãy dùng nó như một buổi diễn tập trình bày trọn vẹn thiết kế.

1. **Explain to a junior engineer** the difference between fan-out on write and fan-out on read, and say which cost each one moves and where it moves it to.

2. **A colleague says:** *"We'll fan out every post to every follower, it keeps the read path simple."* **Explain what is wrong with that**, using an account with five million followers as your example.

3. **Someone on your team wants to** store the full post content in each follower's feed so that reads need no extra lookup. **Explain why you would push back**, and describe what happens when an author edits a post.

4. **Describe what happens when** a user scrolls an infinite feed built on offset pagination while new posts keep arriving at the top.

5. **When would you choose** eventual consistency for a feed, and where would you make an exception? Explain the read-your-own-writes case in your answer.

6. **Explain to a product manager** why the ranking model cannot run on every post at request time, and what the two-stage approach gives them instead.

7. **Describe what happens when** a post is published, following it all the way from the publish button to a follower's screen, including the celebrity case.
