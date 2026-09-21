# Bài 3 — Invalidation · TTL · Thiết kế key · Negative caching
### Bản song ngữ: tiếng Anh bám cấu trúc tiếng Việt

> **Cách dùng bài này:** đọc đoạn tiếng Việt trước → tự dịch trong đầu → so với đoạn tiếng Anh bên dưới → **đọc to bản tiếng Anh hai lần**: lần một đọc chậm cho đúng, lần hai đọc trôi như đang nói với người phỏng vấn.

---

## Phần 1 — Vì sao invalidation là việc khó

**Tiếng Việt**

Có một câu đùa nổi tiếng trong ngành: hai việc khó nhất trong khoa học máy tính là đặt tên biến, cache invalidation, và lỗi lệch một đơn vị. Câu đùa đó chỉ đúng một nửa, vì vấn đề thật sự nằm ở hai câu hỏi rất cụ thể. Câu hỏi thứ nhất là khi nào chúng ta phải xoá, và câu hỏi thứ hai là chúng ta phải xoá cái gì. Với một bản ghi đơn lẻ thì hai câu hỏi đó dễ trả lời, nhưng với dữ liệu dẫn xuất thì mọi thứ khó hẳn lên. Một dòng dữ liệu thay đổi có thể làm sai một danh sách, một con số tổng hợp, và cả một trang đã render. Nếu chúng ta xoá thiếu thì người dùng nhận dữ liệu cũ, còn nếu chúng ta xoá thừa thì chúng ta phá mất hiệu quả của cache. Ngoài ra chúng ta còn phải phối hợp giữa nhiều tiến trình ghi và nhiều instance, vì mỗi nơi đều có thể thay đổi cùng một nguồn.

Đây là lý do chúng ta không nên trả lời câu hỏi này bằng một câu ngắn trong buổi phỏng vấn. Người phỏng vấn muốn nghe chúng ta phân biệt giữa dữ liệu gốc và dữ liệu dẫn xuất. Họ cũng muốn nghe chúng ta thừa nhận rằng không có lời giải hoàn hảo, mà chỉ có các mức đánh đổi. Vì vậy cách trả lời tốt là nêu từng công cụ, nêu điểm mù của từng công cụ, rồi nói chúng ta ghép chúng lại với nhau như thế nào.

**English (bám cấu trúc tiếng Việt)**

There is a famous joke in our industry: the two hardest things in computer science are naming variables, cache invalidation, and off-by-one errors. That joke is only half true, because the real problem lies in two very concrete questions. The first question is when we have to delete, and the second question is what we have to delete. For a single record those two questions are easy to answer, but for derived data everything becomes much harder. One row that changes can make a list wrong, an aggregate number wrong, and a whole rendered page wrong as well. If we delete too little then users receive stale data, and if we delete too much then we destroy the effectiveness of the cache. On top of that we still have to coordinate between many writing processes and many instances, because each of them can change the same source.

This is why we should not answer this question with one short sentence in an interview. The interviewer wants to hear us distinguish between source data and derived data. They also want to hear us admit that there is no perfect solution, but only levels of trade-off. Therefore the good way to answer is to name each tool, name the blind spot of each tool, and then say how we combine them together.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| lỗi lệch một đơn vị | off-by-one errors |
| câu đùa đó chỉ đúng một nửa | that joke is only half true |
| vấn đề thật sự nằm ở | the real problem lies in |
| với dữ liệu dẫn xuất thì mọi thứ khó hẳn lên | for derived data everything becomes much harder |
| một con số tổng hợp | an aggregate number |
| nếu chúng ta xoá thiếu | if we delete too little |
| phá mất hiệu quả của cache | destroy the effectiveness of the cache |
| phối hợp giữa nhiều tiến trình ghi | coordinate between many writing processes |
| thừa nhận rằng không có lời giải hoàn hảo | admit that there is no perfect solution |
| chỉ có các mức đánh đổi | only levels of trade-off |
| nêu điểm mù của từng công cụ | name the blind spot of each tool |
| ghép chúng lại với nhau | combine them together |

**Thuật ngữ cần nhớ**

- làm mất hiệu lực cache → **cache invalidation**
- dữ liệu dẫn xuất → **derived data**
- dữ liệu tổng hợp → **aggregate data**
- điểm mù → **a blind spot**
- lỗi lệch một đơn vị → **an off-by-one error**

---

## Phần 2 — TTL, xoá tường minh, và lưới an toàn

**Tiếng Việt**

Chúng ta có hai cơ chế gốc để kiểm soát dữ liệu cũ. Cơ chế thứ nhất là hết hạn theo TTL: chúng ta đặt một thời hạn cho key, và key tự biến mất khi hết thời hạn đó. Cơ chế này rất đơn giản, nó tự dọn dẹp, và mức stale tối đa đúng bằng TTL mà chúng ta đặt. Cơ chế thứ hai là xoá tường minh: chúng ta xoá key ngay khi có một lệnh ghi. Cơ chế này cho dữ liệu tươi hơn, nhưng nó đòi hỏi chúng ta bắt được mọi đường ghi và xoá đúng key. Trong thực tế chúng ta kết hợp cả hai: xoá tường minh để có độ tươi, cộng thêm TTL để làm lưới an toàn. Nhờ lưới an toàn đó, nếu chúng ta lỡ bỏ sót một đường ghi thì dữ liệu cũ cũng chỉ sống tối đa một khoảng TTL chứ không sống vĩnh viễn.

Việc chọn TTL ngắn hay dài cũng là một đánh đổi rất rõ ràng. TTL ngắn cho dữ liệu tươi hơn, nhưng nó làm tỉ lệ hit giảm xuống và làm tải trên DB tăng lên. TTL dài cho hiệu quả cache cao hơn, nhưng nó giữ dữ liệu cũ lâu hơn. Chúng ta chọn dựa trên hai yếu tố: mức stale mà nghiệp vụ chấp nhận được, và chi phí của một lần miss. Có một cái bẫy chúng ta phải tránh tuyệt đối, đó là bỏ hẳn TTL và chỉ dựa vào xoá thủ công. Nếu chỉ một đường ghi quên gọi lệnh xoá, key đó sẽ cũ mãi mãi và không có gì cứu chúng ta cả.

**English (bám cấu trúc tiếng Việt)**

We have two basic mechanisms to control stale data. The first mechanism is TTL-based expiry: we set a deadline on the key, and the key disappears by itself when that deadline runs out. This mechanism is very simple, it cleans up after itself, and the maximum staleness is exactly the TTL that we set. The second mechanism is explicit invalidation: we delete the key as soon as there is a write. This mechanism gives fresher data, but it requires us to catch every write path and to delete the right key. In practice we combine both: explicit invalidation for freshness, plus a TTL to act as a safety net. Thanks to that safety net, if we happen to miss one write path then the stale data only lives for at most one TTL window instead of living forever.

Choosing a short or a long TTL is also a very clear trade-off. A short TTL gives fresher data, but it drives the hit ratio down and drives the load on the database up. A long TTL gives better cache effectiveness, but it holds stale data for longer. We choose based on two factors: the staleness the business can accept, and the cost of one miss. There is one trap we must avoid completely, which is dropping the TTL entirely and relying only on manual deletion. If just one write path forgets to call the delete, that key will be stale forever and nothing will save us.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| hết hạn theo TTL | TTL-based expiry |
| đặt một thời hạn cho key | set a deadline on the key |
| nó tự dọn dẹp | it cleans up after itself |
| mức stale tối đa đúng bằng | the maximum staleness is exactly |
| xoá tường minh | explicit invalidation |
| bắt được mọi đường ghi | catch every write path |
| để làm lưới an toàn | to act as a safety net |
| nếu chúng ta lỡ bỏ sót một đường ghi | if we happen to miss one write path |
| chỉ sống tối đa một khoảng TTL | only lives for at most one TTL window |
| làm tỉ lệ hit giảm xuống | drives the hit ratio down |
| mức stale mà nghiệp vụ chấp nhận được | the staleness the business can accept |
| bỏ hẳn TTL và chỉ dựa vào xoá thủ công | dropping the TTL entirely and relying only on manual deletion |
| không có gì cứu chúng ta cả | nothing will save us |

**Thuật ngữ cần nhớ**

- hết hạn → **expiry** / **to expire**
- xoá tường minh → **explicit invalidation**
- lưới an toàn → **a safety net** / **a TTL backstop**
- đường ghi → **a write path**
- mức cũ của dữ liệu → **staleness**

---

## Phần 3 — Thiết kế cache key

**Tiếng Việt**

Thiết kế key là phần dễ bị xem nhẹ nhất, nhưng nó lại là chỗ sinh ra những lỗi nghiêm trọng nhất. Nguyên tắc chỉ gói trong một câu: key phải xác định duy nhất kết quả, vì vậy nó phải chứa mọi tham số ảnh hưởng tới đầu ra. Những chiều hay bị quên là định danh người dùng hoặc tenant, ngôn ngữ, bộ lọc, và phiên bản của schema. Một key tồi kinh điển là key chỉ có mỗi chữ products, không kèm tenant, không kèm bộ lọc, không kèm ngôn ngữ. Với một key như vậy, một tenant có thể nhận về dữ liệu của tenant khác, và đó không còn là lỗi hiệu năng nữa mà là lỗi rò rỉ dữ liệu. Một key tốt phải ghi rõ tenant, loại dữ liệu, ngôn ngữ, bộ lọc và số phiên bản. Chúng ta viết chúng thành một chuỗi có namespace rõ ràng, và các phần được ngăn cách bằng dấu hai chấm.

Khi review code, chúng ta nên đọc key trước khi đọc logic. Câu hỏi kiểm tra là: hai người dùng khác nhau có thể tạo ra cùng một key này hay không. Nếu câu trả lời là có, mà kết quả của họ lại phải khác nhau, thì chúng ta vừa tìm ra một lỗ rò dữ liệu. Đây là lỗi mà công cụ AI mắc phải rất thường xuyên, vì nó hay sinh key theo tên hàm chứ không theo tham số.

**English (bám cấu trúc tiếng Việt)**

Key design is the part that is most easily underestimated, but it is also the place that produces the most serious bugs. The principle fits in a single sentence: the key must uniquely identify the result, therefore it must contain every parameter that affects the output. The dimensions that are often forgotten are the user or tenant identifier, the language, the filter, and the version of the schema. A classic bad key is a key that only has the word products, with no tenant, no filter and no language. With a key like that, one tenant can receive the data of another tenant, and that is no longer a performance bug but a data leak. A good key must spell out the tenant, the type of data, the language, the filter and the version number. We write them as a string with a clear namespace, and the parts are separated by colons.

When we review code, we should read the key before we read the logic. The check question is: can two different users produce this same key. If the answer is yes, while their results are supposed to be different, then we have just found a data leak. This is a mistake that AI tools make very often, because they tend to generate the key from the function name instead of from the parameters.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| dễ bị xem nhẹ nhất | most easily underestimated |
| chỗ sinh ra những lỗi nghiêm trọng nhất | the place that produces the most serious bugs |
| nguyên tắc chỉ gói trong một câu | the principle fits in a single sentence |
| xác định duy nhất kết quả | uniquely identify the result |
| mọi tham số ảnh hưởng tới đầu ra | every parameter that affects the output |
| những chiều hay bị quên | the dimensions that are often forgotten |
| chỉ có mỗi chữ products | only has the word products |
| không còn là lỗi hiệu năng nữa mà là | no longer a performance bug but |
| phải ghi rõ | must spell out |
| ngăn cách bằng dấu hai chấm | separated by colons |
| chúng ta vừa tìm ra một lỗ rò dữ liệu | we have just found a data leak |
| sinh key theo tên hàm chứ không theo tham số | generate the key from the function name instead of from the parameters |

**Thuật ngữ cần nhớ**

- thiết kế key → **key design**
- khách hàng dùng chung hệ thống → **a tenant**
- ngôn ngữ và vùng → **locale**
- không gian tên → **a namespace**
- rò rỉ dữ liệu → **a data leak**

---

## Phần 4 — Key có phiên bản và cách vô hiệu cả nhóm

**Tiếng Việt**

Khi chúng ta đổi định dạng dữ liệu, chúng ta không nên đi xoá từng key một. Cách tốt hơn là nhúng một số phiên bản vào chính key, hoặc đổi prefix của cả nhóm key đó. Chúng ta chỉ cần tăng số phiên bản lên một đơn vị, và ngay lập tức toàn bộ bản cũ trở nên vô hiệu, vì không còn ai đọc vào key cũ nữa. Các key cũ sẽ tự rơi rụng theo TTL hoặc theo cơ chế đuổi key, nên chúng ta không phải dọn chúng bằng tay. Cách này còn tránh được điều kiện tranh chấp kiểu xoá xong rồi đọc lại, vì thật ra chúng ta không xoá gì cả. Cùng một ý tưởng đó giải luôn bài toán dữ liệu tổng hợp: khi một phần tử đổi, rất nhiều key dẫn xuất bị ảnh hưởng, ví dụ danh sách, số đếm và từng trang. Thay vì truy từng key một, chúng ta gắn thẻ cho cả nhóm hoặc tăng phiên bản của cả namespace, và chúng ta cũng có thể đặt riêng một TTL ngắn cho các key tổng hợp.

Số phiên bản đó nên nằm ở một chỗ cấu hình duy nhất. Nếu chúng ta rải nó khắp code, việc tăng phiên bản sẽ trở thành một lần sửa mười chỗ, và chắc chắn chúng ta sẽ sót một chỗ. Trong thế giới tệp tĩnh, chúng ta đã quen với đúng ý tưởng này rồi: tên tệp chứa mã băm của nội dung. Đổi nội dung nghĩa là đổi tên tệp, vì vậy CDN buộc phải lấy bản mới mà chúng ta không phải gọi một lệnh xoá nào.

**English (bám cấu trúc tiếng Việt)**

When we change the data format, we should not go and delete each key one by one. The better way is to embed a version number into the key itself, or to change the prefix of that whole group of keys. We only need to bump the version number by one, and immediately the entire old set becomes invalid, because nobody reads the old keys any more. The old keys will fall away by themselves through the TTL or through the eviction mechanism, so we do not have to clean them up by hand. This way also avoids the race condition of deleting and then reading again, because in fact we delete nothing. That same idea also solves the problem of aggregate data: when one element changes, many derived keys are affected, for example lists, counts and individual pages. Instead of chasing each key one by one, we tag the whole group or bump the version of the whole namespace, and we can also set a separate short TTL for the aggregate keys.

That version number should live in one single configuration place. If we scatter it all over the code, bumping the version becomes an edit in ten places, and we will certainly miss one of them. In the world of static files, we are already used to exactly this idea: the file name contains a hash of the content. Changing the content means changing the file name, therefore the CDN is forced to fetch the new version while we do not have to call a single delete command.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| đi xoá từng key một | go and delete each key one by one |
| nhúng một số phiên bản vào chính key | embed a version number into the key itself |
| tăng số phiên bản lên một đơn vị | bump the version number by one |
| toàn bộ bản cũ trở nên vô hiệu | the entire old set becomes invalid |
| tự rơi rụng theo TTL | fall away by themselves through the TTL |
| dọn chúng bằng tay | clean them up by hand |
| điều kiện tranh chấp kiểu xoá xong rồi đọc lại | the race condition of deleting and then reading again |
| thay vì truy từng key một | instead of chasing each key one by one |
| gắn thẻ cho cả nhóm | tag the whole group |
| nằm ở một chỗ cấu hình duy nhất | live in one single configuration place |
| nếu chúng ta rải nó khắp code | if we scatter it all over the code |
| tên tệp chứa mã băm của nội dung | the file name contains a hash of the content |
| CDN buộc phải lấy bản mới | the CDN is forced to fetch the new version |

**Thuật ngữ cần nhớ**

- key có gắn phiên bản → **a versioned key**
- tăng số phiên bản → **to bump the version**
- vô hiệu hoá cả nhóm → **cache busting**
- cơ chế đuổi key → **eviction**
- mã băm của nội dung → **a content hash**

---

## Phần 5 — Xoá cache dựa trên sự kiện

**Tiếng Việt**

Thay vì gọi lệnh xoá cache ngay trong đoạn code ghi, chúng ta có thể phát ra một sự kiện khi DB thay đổi. Một consumer sẽ nghe sự kiện đó và chịu trách nhiệm xoá cache. Lợi ích thứ nhất là chúng ta tách được các mối quan tâm: đoạn code nghiệp vụ chỉ lo ghi dữ liệu, nó không phải nhớ xoá cache. Lợi ích thứ hai còn quan trọng hơn: chúng ta bắt được cả những lệnh ghi không đi qua app, ví dụ một batch job hoặc một service khác ghi thẳng vào DB. Để làm việc này cho chắc chắn, chúng ta thường dùng mẫu outbox hoặc dùng CDC, nghĩa là đọc thay đổi trực tiếp từ log của DB. Nếu không có hai thứ đó, chúng ta sẽ gặp tình huống ghi DB thành công nhưng phát sự kiện thất bại, và khi đó cache không bao giờ được xoá. Bài 4 và mục K sẽ nói kỹ hơn về outbox và về CDC.

Chúng ta nên nói rõ cái giá của hướng này khi trình bày. Chúng ta thêm một đường xử lý bất đồng bộ, vì vậy việc xoá cache không còn xảy ra tức thì mà xảy ra sau một độ trễ nhỏ. Chúng ta cũng phải xử lý trường hợp một sự kiện đến hai lần, nên thao tác xoá phải an toàn khi bị lặp lại. Đổi lại, chúng ta không còn phải tin vào giả định rằng mọi lệnh ghi đều đi qua app, và giả định đó gần như luôn sai trong một hệ thống lớn.

**English (bám cấu trúc tiếng Việt)**

Instead of calling the cache delete right inside the writing code, we can emit an event when the database changes. A consumer will listen to that event and take responsibility for deleting the cache. The first benefit is that we separate the concerns: the business code only cares about writing data, it does not have to remember to delete the cache. The second benefit is even more important: we also catch the writes that do not go through the app, for example a batch job or another service writing straight into the database. To do this reliably, we usually use the outbox pattern or use CDC, which means reading the changes directly from the database log. Without those two things, we will run into the situation where the database write succeeds but publishing the event fails, and then the cache is never deleted. Lesson 4 and topic K will talk in more detail about the outbox and about CDC.

We should state the price of this direction clearly when we present it. We add an asynchronous processing path, therefore the cache deletion no longer happens instantly but happens after a small delay. We also have to handle the case where one event arrives twice, so the delete operation must be safe when it is repeated. In exchange, we no longer have to trust the assumption that every write goes through the app, and that assumption is almost always wrong in a large system.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| ngay trong đoạn code ghi | right inside the writing code |
| phát ra một sự kiện | emit an event |
| chịu trách nhiệm xoá cache | take responsibility for deleting the cache |
| tách được các mối quan tâm | separate the concerns |
| nó không phải nhớ xoá cache | it does not have to remember to delete the cache |
| để làm việc này cho chắc chắn | to do this reliably |
| đọc thay đổi trực tiếp từ log của DB | reading the changes directly from the database log |
| ghi DB thành công nhưng phát sự kiện thất bại | the database write succeeds but publishing the event fails |
| nói rõ cái giá của hướng này | state the price of this direction clearly |
| không còn xảy ra tức thì | no longer happens instantly |
| an toàn khi bị lặp lại | safe when it is repeated |
| tin vào giả định rằng | trust the assumption that |

**Thuật ngữ cần nhớ**

- xoá cache dựa trên sự kiện → **event-based invalidation**
- bên tiêu thụ sự kiện → **a consumer**
- mẫu hộp thư đi → **the outbox pattern**
- bắt thay đổi dữ liệu → **change data capture (CDC)**
- an toàn khi lặp lại → **idempotent**

---

## Phần 6 — Negative caching

**Tiếng Việt**

Negative caching nghĩa là chúng ta cache cả kết quả không tồn tại. Chúng ta lưu một dấu hiệu rỗng cho key đó, thay vì không lưu gì cả. Mục đích là chặn những request mang định danh rác cứ liên tục đập thẳng vào DB, vì với chúng thì cache luôn miss. Nếu chúng ta không làm việc này, một kẻ tấn công chỉ cần bắn hàng loạt id không có thật là DB của chúng ta phải gánh toàn bộ. Nhưng negative caching có một rủi ro rất rõ: nếu dữ liệu đó được tạo ra ngay sau đó, dấu hiệu rỗng trong cache sẽ che mất dữ liệu thật. Vì vậy chúng ta luôn đặt TTL rất ngắn cho dấu hiệu rỗng, thường chỉ vài chục giây. Và chúng ta phải xoá dấu hiệu rỗng đó ngay tại lúc dữ liệu mới được tạo.

Có một chi tiết nhỏ nhưng quan trọng trong lúc cài đặt. Chúng ta phải phân biệt được ba trạng thái: key không có trong cache, key có và chứa dấu hiệu rỗng, và key có và chứa dữ liệu thật. Nếu chúng ta dùng chung một giá trị cho hai trạng thái đầu, mọi lần negative hit sẽ bị hiểu nhầm thành miss và toàn bộ lợi ích biến mất. Bài 5 sẽ nối tiếp phần này khi nói về hiện tượng xuyên thủng cache và về bộ lọc Bloom.

**English (bám cấu trúc tiếng Việt)**

Negative caching means that we cache the non-existent result as well. We store an empty marker for that key, instead of storing nothing at all. The purpose is to block the requests carrying junk identifiers that keep hitting the database directly, because for them the cache always misses. If we do not do this, an attacker only needs to fire a flood of ids that do not exist and our database has to carry all of it. But negative caching has one very clear risk: if that data is created right afterwards, the empty marker in the cache will hide the real data. Therefore we always set a very short TTL on the empty marker, usually only a few tens of seconds. And we have to delete that empty marker at the very moment the new data is created.

There is a small but important detail during implementation. We must be able to distinguish three states: the key is not in the cache, the key exists and holds an empty marker, and the key exists and holds real data. If we use the same value for the first two states, every negative hit will be misread as a miss and the whole benefit disappears. Lesson 5 will continue this part when it talks about the cache penetration problem and about Bloom filters.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| cache cả kết quả không tồn tại | cache the non-existent result as well |
| một dấu hiệu rỗng | an empty marker |
| thay vì không lưu gì cả | instead of storing nothing at all |
| định danh rác | junk identifiers |
| cứ liên tục đập thẳng vào DB | keep hitting the database directly |
| bắn hàng loạt id không có thật | fire a flood of ids that do not exist |
| DB của chúng ta phải gánh toàn bộ | our database has to carry all of it |
| sẽ che mất dữ liệu thật | will hide the real data |
| vài chục giây | a few tens of seconds |
| ngay tại lúc dữ liệu mới được tạo | at the very moment the new data is created |
| bị hiểu nhầm thành miss | be misread as a miss |
| toàn bộ lợi ích biến mất | the whole benefit disappears |

**Thuật ngữ cần nhớ**

- cache kết quả rỗng → **negative caching**
- dấu hiệu rỗng → **an empty marker** / **a null marker**
- định danh rác → **a junk identifier**
- xuyên thủng cache → **cache penetration**
- bộ lọc Bloom → **a Bloom filter**

---

## Phần 7 — Trả bản cũ trong lúc làm mới

**Tiếng Việt**

Stale-while-revalidate là một kỹ thuật đơn giản nhưng hiệu quả rất cao. Khi một key vừa hết hạn, chúng ta vẫn trả bản cũ cho client ngay lập tức, đồng thời chúng ta cho một tiến trình nền đi làm mới nó. Nhờ đó không người dùng nào phải đứng chờ DB đúng vào thời điểm key hết hạn. Chúng ta chấp nhận một mức stale rất ngắn để đổi lấy việc xoá bỏ cú tăng độ trễ. Kỹ thuật này rất phổ biến ở tầng HTTP và tầng CDN, và Bài 11 sẽ nói kỹ về nó. Nhưng chúng ta cũng dùng được đúng ý tưởng đó ở cache bên trong app.

Chúng ta nên nêu điều kiện áp dụng thay vì nói rằng nó luôn tốt. Kỹ thuật này chỉ dùng được khi nghiệp vụ chấp nhận đọc một bản hơi cũ trong vài giây. Với số dư tiền hoặc với tồn kho tại bước thanh toán, chúng ta không dùng nó. Với danh mục sản phẩm, bảng xếp hạng hoặc trang chủ, đây gần như luôn là lựa chọn đúng.

**English (bám cấu trúc tiếng Việt)**

Stale-while-revalidate is a simple technique but one with very high effectiveness. When a key has just expired, we still return the old copy to the client immediately, and at the same time we let a background process go and refresh it. Thanks to that no user has to stand and wait for the database exactly at the moment the key expires. We accept a very short amount of staleness in exchange for removing the latency spike. This technique is very common at the HTTP layer and the CDN layer, and Lesson 11 will talk about it in detail. But we can also use exactly that idea in the cache inside the app.

We should state the condition for applying it instead of saying that it is always good. This technique can only be used when the business accepts reading a slightly old copy for a few seconds. For money balances or for stock levels at the checkout step, we do not use it. For a product catalogue, a leaderboard or a home page, this is almost always the right choice.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| đơn giản nhưng hiệu quả rất cao | simple but one with very high effectiveness |
| khi một key vừa hết hạn | when a key has just expired |
| trả bản cũ cho client ngay lập tức | return the old copy to the client immediately |
| cho một tiến trình nền đi làm mới nó | let a background process go and refresh it |
| không người dùng nào phải đứng chờ DB | no user has to stand and wait for the database |
| để đổi lấy việc xoá bỏ cú tăng độ trễ | in exchange for removing the latency spike |
| nêu điều kiện áp dụng | state the condition for applying it |
| đọc một bản hơi cũ trong vài giây | reading a slightly old copy for a few seconds |
| gần như luôn là lựa chọn đúng | is almost always the right choice |

**Thuật ngữ cần nhớ**

- trả bản cũ trong lúc làm mới → **stale-while-revalidate**
- tiến trình nền → **a background process**
- bảng xếp hạng → **a leaderboard**
- danh mục sản phẩm → **a product catalogue**
- cú tăng độ trễ → **a latency spike**

---

## Phần 8 — Góc tech lead: batch job ban đêm ghi thẳng vào DB

**Tiếng Việt**

Tình huống thách đố của bài này rất sát với thực tế. Chúng ta đặt TTL một giờ cho cache, và chúng ta gọi lệnh xoá cache mỗi lần app cập nhật dữ liệu. Rồi một batch job chạy ban đêm sửa thẳng vào DB mà không đi qua app. Kết quả là batch job đó không kích hoạt lệnh xoá nào, vì vậy cache giữ dữ liệu cũ tới một tiếng đồng hồ. Buổi sáng, người dùng nhìn thấy số liệu của ngày hôm qua, trong khi truy vấn thẳng vào DB thì lại thấy số liệu đúng. Cách sửa đúng là chuyển sang xoá cache dựa trên sự kiện hoặc dựa trên CDC, vì cách đó bắt được thay đổi từ chính log của DB. Cách sửa tạm thời là giảm TTL cho phần dữ liệu đó, nhưng nó chỉ làm nhẹ triệu chứng chứ không chữa nguyên nhân.

Bài học rút ra rất đáng nhớ và chúng ta nên nói thẳng nó ra trong buổi phỏng vấn. Chúng ta không được dựa vào giả định rằng mọi lệnh ghi đều đi qua app của chúng ta. Trong bất kỳ hệ thống nào sống đủ lâu, sẽ luôn có một script di trú, một công cụ quản trị, hoặc một service cũ ghi thẳng vào DB. Vì vậy chúng ta thiết kế cơ chế xoá cache dựa trên nguồn sự thật, chứ không dựa trên thiện chí của người viết code.

**English (bám cấu trúc tiếng Việt)**

The challenge scenario of this lesson is very close to real life. We set a TTL of one hour on the cache, and we call the cache delete every time the app updates the data. Then a batch job that runs at night edits the database directly without going through the app. The result is that this batch job does not trigger any delete, therefore the cache holds stale data for up to one hour. In the morning, users see yesterday's numbers, while querying the database directly shows the correct numbers. The correct fix is to move to event-based or CDC-based invalidation, because that way catches the change from the database log itself. The temporary fix is to lower the TTL for that piece of data, but it only eases the symptom instead of curing the cause.

The lesson we take away is well worth remembering and we should say it out loud in the interview. We must not rely on the assumption that every write goes through our app. In any system that lives long enough, there will always be a migration script, an admin tool, or an old service writing straight into the database. Therefore we design the invalidation mechanism around the source of truth, and not around the good intentions of whoever writes the code.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| rất sát với thực tế | very close to real life |
| sửa thẳng vào DB mà không đi qua app | edits the database directly without going through the app |
| không kích hoạt lệnh xoá nào | does not trigger any delete |
| tới một tiếng đồng hồ | for up to one hour |
| số liệu của ngày hôm qua | yesterday's numbers |
| bắt được thay đổi từ chính log của DB | catches the change from the database log itself |
| chỉ làm nhẹ triệu chứng chứ không chữa nguyên nhân | only eases the symptom instead of curing the cause |
| bài học rút ra | the lesson we take away |
| chúng ta nên nói thẳng nó ra | we should say it out loud |
| trong bất kỳ hệ thống nào sống đủ lâu | in any system that lives long enough |
| một script di trú | a migration script |
| dựa trên thiện chí của người viết code | around the good intentions of whoever writes the code |

**Thuật ngữ cần nhớ**

- tác vụ chạy theo lô → **a batch job**
- kích hoạt → **to trigger**
- triệu chứng và nguyên nhân → **the symptom and the cause**
- script di trú dữ liệu → **a migration script**
- nguồn sự thật → **the source of truth**

---

## Mô hình ghi nhớ

**Tiếng Việt**

Key phải chứa mọi thứ ảnh hưởng tới kết quả, và chúng ta luôn ghép xoá tường minh với một lưới an toàn TTL. Khi đổi định dạng thì chúng ta tăng số phiên bản chứ không xoá từng key, và chúng ta cache cả kết quả không tồn tại để chặn định danh rác.

**English (bám cấu trúc tiếng Việt)**

The key must contain everything that affects the result, and we always combine explicit invalidation with a TTL safety net. When we change the format we bump the version number instead of deleting each key, and we cache the non-existent result as well in order to block junk identifiers.

---

## Bảng thuật ngữ tổng hợp

| Tiếng Việt | Tiếng Anh | Ghi chú phát âm (Anh-Anh) |
|---|---|---|
| làm mất hiệu lực cache | cache invalidation | in-val-i-**DAY**-shən — trọng âm áp chót; cache = "cash" |
| xoá tường minh | explicit invalidation | ik-**SPLIS**-it — trọng âm âm tiết thứ hai |
| hết hạn | expiry / to expire | ik-**SPAI**-ri — trọng âm giữa, âm /aɪ/ |
| thời gian sống | time to live (TTL) | đọc rời từng chữ cái: "tee-tee-el" |
| lưới an toàn | a safety net / a TTL backstop | |
| mức cũ của dữ liệu | staleness | stale /steɪl/ — "xtêi-l", không phải "xtan" |
| đường ghi | a write path | write — chữ **w** câm; path Anh-Anh /pɑːθ/ |
| dữ liệu dẫn xuất | derived data | di-**RAIVD** — trọng âm âm tiết thứ hai |
| dữ liệu tổng hợp | aggregate data | danh từ **AG**-ri-gət, động từ **AG**-ri-geit |
| điểm mù | a blind spot | |
| lỗi lệch một đơn vị | an off-by-one error | |
| thiết kế key | key design | |
| khách hàng dùng chung hệ thống | a tenant | **TEN**-ənt — không đọc "ti-nần" |
| ngôn ngữ và vùng | locale | Anh-Anh ləʊ-**KAAL** — trọng âm cuối, không đọc "lô-kêu" |
| lược đồ dữ liệu | schema | **SKEE**-mə — "sch" đọc thành /sk/ |
| không gian tên | a namespace | |
| tiền tố | a prefix | **PREE**-fiks — trọng âm âm tiết đầu |
| dấu hai chấm | a colon | **KOH**-lən |
| rò rỉ dữ liệu | a data leak | |
| key có gắn phiên bản | a versioned key | |
| tăng số phiên bản | to bump the version | |
| vô hiệu hoá cả nhóm | cache busting | |
| cơ chế đuổi key | eviction | i-**VIK**-shən — trọng âm âm tiết thứ hai |
| mã băm của nội dung | a content hash | |
| gắn thẻ | to tag | |
| điều kiện tranh chấp | a race condition | |
| xoá cache dựa trên sự kiện | event-based invalidation | event — i-**VENT**, trọng âm cuối |
| bên tiêu thụ sự kiện | a consumer | Anh-Anh kən-**SYOO**-mə — có /sj/ |
| phát ra sự kiện | to emit an event | i-**MIT** |
| mẫu hộp thư đi | the outbox pattern | |
| bắt thay đổi dữ liệu | change data capture (CDC) | |
| an toàn khi lặp lại | idempotent | ai-**DEM**-pə-tənt — trọng âm âm tiết thứ hai |
| bất đồng bộ | asynchronous | ei-**SIN**-krə-nəs — "ch" đọc thành /k/ |
| giả định | an assumption | ə-**SUMP**-shən |
| cache kết quả rỗng | negative caching | |
| dấu hiệu rỗng | an empty marker / a null marker | null /nʌl/ — "nal", không phải "nu-lồ" |
| định danh rác | a junk identifier | ai-**DEN**-ti-fai-ə |
| xuyên thủng cache | cache penetration | pen-ə-**TRAY**-shən |
| kẻ tấn công | an attacker | ə-**TAK**-ə |
| bộ lọc Bloom | a Bloom filter | |
| trả bản cũ trong lúc làm mới | stale-while-revalidate | ree-**VAL**-i-deit |
| tiến trình nền | a background process | |
| bảng xếp hạng | a leaderboard | |
| danh mục sản phẩm | a product catalogue | **KAT**-ə-log — chính tả Anh có đuôi *-logue* |
| cú tăng độ trễ | a latency spike | latency — **LAY**-tən-si |
| tác vụ chạy theo lô | a batch job | batch /bætʃ/ — kết thúc bằng /tʃ/ |
| kích hoạt | to trigger | **TRIG**-ə |
| triệu chứng | a symptom | **SIMP**-təm — chữ **p** có phát âm |
| script di trú dữ liệu | a migration script | mai-**GRAY**-shən |
| nguồn sự thật | the source of truth | truth — âm /θ/ ở cuối, không thành "trút" |
| tỉ lệ hit | hit ratio | ratio — **RAY**-shi-əʊ |

---

## Bài nói

> **Cách luyện:** bật ghi âm, nói **60–90 giây** cho mỗi đề, **không nhìn tài liệu**. Mỗi bài nói phải dùng ít nhất **sáu thuật ngữ** trong bảng trên. Nghe lại một lần, đánh dấu chỗ bạn ngập ngừng, rồi nói lại đúng đề đó thêm một lần nữa.

1. **Explain to a junior** why cache invalidation is famously hard, using the difference between a single record and derived data.

2. **A colleague wants** to remove all TTLs from the cache, because the team already invalidates explicitly on every write. Explain why you would push back and what you would keep.

3. **You are reviewing a pull request** and the cache key is simply the name of the function. Explain to the author what is missing, what can go wrong, and what the key should contain.

4. **Someone on your team proposes** deleting every affected key one by one whenever the data format changes. Explain why you would push back, and describe the approach you would use instead.

5. **Describe what happens** when a nightly batch job writes straight into the database while your invalidation lives in the application code. Then explain how you would fix it properly.

6. **Explain what negative caching is**, why we need it, and the one risk it brings with it. Say exactly how you control that risk.

7. **When would you use** stale-while-revalidate, and when would you refuse to use it? Give one example on each side.
