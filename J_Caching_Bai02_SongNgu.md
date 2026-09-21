# Bài 2 — Các chiến lược cache (Cache Strategies)
### Bản song ngữ: tiếng Anh bám cấu trúc tiếng Việt

> **Cách dùng bài này:** đọc đoạn tiếng Việt trước → tự dịch trong đầu → so với đoạn tiếng Anh bên dưới → **đọc to bản tiếng Anh hai lần**: lần một đọc chậm cho đúng, lần hai đọc trôi như đang nói với người phỏng vấn.

---

## Phần 1 — Khung tư duy: năm chiến lược khác nhau ở đâu

**Tiếng Việt**

Bài này cho chúng ta khung năm chiến lược đọc và ghi cache, và đó là bộ từ vựng chúng ta sẽ dùng suốt phần còn lại của mục caching. Năm chiến lược này chỉ khác nhau ở hai điểm, vì vậy chúng ta không cần học thuộc từng cái một cách rời rạc. Điểm khác nhau thứ nhất là ai chịu trách nhiệm đọc và ghi giữa cache và DB, app hay là chính tầng cache. Điểm khác nhau thứ hai là việc ghi xuống DB diễn ra đồng bộ hay bất đồng bộ. Nếu chúng ta nắm được hai trục đó, chúng ta có thể tự suy ra luồng của từng chiến lược ngay tại buổi phỏng vấn. Nếu chúng ta chỉ học thuộc tên, chúng ta sẽ lúng túng ngay khi người phỏng vấn hỏi ngược lại bằng một tình huống cụ thể.

Chúng ta cũng nên nhớ rằng năm cái tên này không phải là năm lựa chọn ngang hàng. Ba cái đầu là những cách làm an toàn và được dùng rộng rãi. Cái thứ tư là write-back, nó rất nhanh nhưng nó có thể làm mất dữ liệu. Cái thứ năm là refresh-ahead, nó là một tối ưu thêm vào chứ không phải một nền tảng để xây hệ thống. Vì vậy trong một câu trả lời tốt, chúng ta nên nêu đủ cả năm nhưng phải nói rõ cái nào là mặc định.

**English (bám cấu trúc tiếng Việt)**

This lesson gives us a framework of five read and write caching strategies, and that is the vocabulary we will use throughout the rest of the caching topic. These five strategies differ in only two points, therefore we do not need to memorise each one separately. The first difference is who is responsible for reading and writing between the cache and the database, the app or the cache layer itself. The second difference is whether the write down to the database happens synchronously or asynchronously. If we hold those two axes, we can work out the flow of each strategy ourselves right in the interview. If we only memorise the names, we will get stuck as soon as the interviewer asks back with a concrete scenario.

We should also remember that these five names are not five equal options. The first three are safe approaches that are widely used. The fourth one is write-back, it is very fast but it can lose data. The fifth one is refresh-ahead, it is an extra optimisation and not a foundation to build a system on. Therefore in a good answer, we should list all five but we must say clearly which one is the default.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| khung năm chiến lược đọc và ghi cache | a framework of five read and write caching strategies |
| bộ từ vựng chúng ta sẽ dùng suốt | the vocabulary we will use throughout |
| chỉ khác nhau ở hai điểm | differ in only two points |
| học thuộc từng cái một cách rời rạc | memorise each one separately |
| ai chịu trách nhiệm đọc và ghi | who is responsible for reading and writing |
| đồng bộ hay bất đồng bộ | synchronously or asynchronously |
| nếu chúng ta nắm được hai trục đó | if we hold those two axes |
| tự suy ra luồng của từng chiến lược | work out the flow of each strategy ourselves |
| chúng ta sẽ lúng túng ngay khi | we will get stuck as soon as |
| hỏi ngược lại bằng một tình huống cụ thể | asks back with a concrete scenario |
| năm lựa chọn ngang hàng | five equal options |
| một nền tảng để xây hệ thống | a foundation to build a system on |

**Thuật ngữ cần nhớ**

- chiến lược cache → **caching strategy**
- tầng cache → **the cache layer**
- đồng bộ → **synchronous** / **synchronously**
- bất đồng bộ → **asynchronous** / **asynchronously**
- mặc định → **the default**

---

## Phần 2 — Cache-aside, hay lazy loading

**Tiếng Việt**

Cache-aside, còn gọi là lazy loading, là chiến lược phổ biến nhất trong công nghiệp. Trong mô hình này, app tự quản lý cache, còn bản thân cache không biết gì về DB. Luồng đọc diễn ra như sau: app kiểm tra cache trước, nếu miss thì app đọc DB, ghi kết quả vào cache, rồi trả về cho người gọi, còn nếu hit thì app trả về luôn. Luồng ghi ngắn hơn nhiều: app cập nhật DB trước, sau đó app xoá key trong cache. Chúng ta xoá key chứ không cập nhật cache, và lý do chi tiết sẽ được nói ở Bài 4. Nhược điểm thứ nhất là lần đọc đầu tiên luôn miss, và chúng ta gọi tình trạng đó là cache lạnh. Nhược điểm thứ hai là vẫn tồn tại một cửa sổ stale giữa lúc dữ liệu đổi và lúc key bị xoá.

Cách nói ngắn gọn nhất cho phần này là hai công thức. Luồng đọc là: kiểm tra, miss, đọc DB, ghi cache. Luồng ghi là: ghi DB trước, rồi xoá key. Nếu chúng ta đảo thứ tự và xoá key trước khi ghi DB, một request khác có thể chen vào giữa và nạp lại giá trị cũ vào cache. Hậu quả là dữ liệu cũ nằm trong cache cho tới khi hết TTL, và đó là loại lỗi rất khó tìm vì nó chỉ xảy ra khi hai request trùng đúng thời điểm.

**English (bám cấu trúc tiếng Việt)**

Cache-aside, also called lazy loading, is the most common strategy in the industry. In this model, the app manages the cache itself, while the cache knows nothing about the database. The read flow goes as follows: the app checks the cache first, if it misses then the app reads the database, writes the result into the cache, and returns it to the caller, and if it hits then the app returns it straight away. The write flow is much shorter: the app updates the database first, and then the app deletes the key in the cache. We delete the key instead of updating the cache, and the detailed reason will be discussed in Lesson 4. The first drawback is that the first read always misses, and we call that situation a cold cache. The second drawback is that there is still a stale window between the moment the data changes and the moment the key is deleted.

The shortest way to say this part is two formulas. The read flow is: check, miss, read the database, write the cache. The write flow is: write the database first, then delete the key. If we reverse the order and delete the key before writing the database, another request can slip in between and load the old value back into the cache. The consequence is that stale data sits in the cache until the TTL runs out, and that is the kind of bug that is very hard to find because it only happens when two requests land at exactly the same moment.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| còn gọi là | also called |
| app tự quản lý cache | the app manages the cache itself |
| cache không biết gì về DB | the cache knows nothing about the database |
| trả về cho người gọi | returns it to the caller |
| app trả về luôn | the app returns it straight away |
| xoá key trong cache | deletes the key in the cache |
| lần đọc đầu tiên luôn miss | the first read always misses |
| cache lạnh | a cold cache |
| một cửa sổ stale | a stale window |
| đảo thứ tự | reverse the order |
| chen vào giữa | slip in between |
| nạp lại giá trị cũ vào cache | load the old value back into the cache |
| cho tới khi hết TTL | until the TTL runs out |
| trùng đúng thời điểm | land at exactly the same moment |

**Thuật ngữ cần nhớ**

- nạp lười, nạp khi cần → **lazy loading**
- luồng đọc, luồng ghi → **the read flow**, **the write flow**
- cache lạnh → **a cold cache**
- xoá key, làm mất hiệu lực → **delete the key** / **invalidate**
- điều kiện tranh chấp → **a race condition**

---

## Phần 3 — Read-through và write-through

**Tiếng Việt**

Read-through rất giống cache-aside nhưng trách nhiệm được dời sang một chỗ khác. Trong read-through, chính tầng cache hoặc thư viện cache tự đọc DB khi gặp miss, còn app chỉ nói chuyện với cache mà thôi. Ưu điểm là code phía app gọn hơn, vì app không phải viết đi viết lại đoạn kiểm tra rồi điền cache. Nhược điểm là chúng ta cần một provider hoặc một lớp trừu tượng hỗ trợ việc đó, ví dụ một thư viện cache có cơ chế loader. Write-through thì giải quyết phía ghi: mỗi lần ghi, hệ thống ghi đồng bộ vào cả cache lẫn DB thông qua tầng cache. Nhờ đó cache luôn tươi, và các lần đọc sau đó nhất quán hơn hẳn so với cache-aside. Đổi lại, độ trễ của thao tác ghi cao hơn, và cache sẽ chứa cả những dữ liệu không bao giờ được đọc lại, nghĩa là chúng ta trả tiền RAM cho thứ không ai dùng.

Có một điểm chúng ta phải cẩn thận khi trình bày write-through. Nhiều người nói rằng write-through cho tính nhất quán tuyệt đối, nhưng điều đó không đúng trong môi trường phân tán. Cache vẫn có thể lệch khi có nhiều cache node, khi có replica của DB, hoặc khi tồn tại một đường ghi khác không đi qua tầng cache. Nếu chúng ta khẳng định quá mạnh trong buổi phỏng vấn, người phỏng vấn sẽ hỏi ngay về đường ghi thứ hai đó. Vì vậy chúng ta nên nói rằng write-through làm cache tươi hơn, chứ không nói rằng nó bảo đảm đúng tuyệt đối.

**English (bám cấu trúc tiếng Việt)**

Read-through is very similar to cache-aside but the responsibility is moved to a different place. In read-through, the cache layer or the cache library itself reads the database when there is a miss, while the app only talks to the cache. The advantage is that the code on the app side is tidier, because the app does not have to write the check-then-fill block over and over. The drawback is that we need a provider or an abstraction layer that supports it, for example a cache library with a loader mechanism. Write-through, on the other hand, solves the write side: on every write, the system writes synchronously into both the cache and the database through the cache layer. Thanks to that the cache is always fresh, and the reads after it are far more consistent than with cache-aside. In exchange, the latency of the write operation is higher, and the cache will also hold data that is never read again, which means we pay for RAM on something nobody uses.

There is one point we have to be careful about when we present write-through. Many people say that write-through gives absolute consistency, but that is not true in a distributed environment. The cache can still drift when there are several cache nodes, when there are database replicas, or when there is another write path that does not go through the cache layer. If we claim it too strongly in an interview, the interviewer will immediately ask about that second write path. Therefore we should say that write-through keeps the cache fresher, instead of saying that it guarantees absolute correctness.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| trách nhiệm được dời sang một chỗ khác | the responsibility is moved to a different place |
| app chỉ nói chuyện với cache mà thôi | the app only talks to the cache |
| code phía app gọn hơn | the code on the app side is tidier |
| viết đi viết lại đoạn kiểm tra rồi điền cache | write the check-then-fill block over and over |
| một lớp trừu tượng hỗ trợ việc đó | an abstraction layer that supports it |
| thì giải quyết phía ghi | on the other hand, solves the write side |
| ghi đồng bộ vào cả cache lẫn DB | writes synchronously into both the cache and the database |
| nhất quán hơn hẳn so với | far more consistent than |
| trả tiền RAM cho thứ không ai dùng | pay for RAM on something nobody uses |
| có một điểm chúng ta phải cẩn thận | there is one point we have to be careful about |
| một đường ghi khác không đi qua tầng cache | another write path that does not go through the cache layer |
| nếu chúng ta khẳng định quá mạnh | if we claim it too strongly |
| bảo đảm đúng tuyệt đối | guarantees absolute correctness |

**Thuật ngữ cần nhớ**

- đọc xuyên qua cache → **read-through**
- ghi xuyên qua cache → **write-through**
- lớp trừu tượng → **an abstraction layer**
- bộ nạp dữ liệu → **a loader**
- bản sao chép của DB → **a database replica**

---

## Phần 4 — Write-back: nhanh nhưng có thể mất dữ liệu

**Tiếng Việt**

Write-back, còn gọi là write-behind, ghi vào cache trước rồi mới đẩy xuống DB sau theo kiểu bất đồng bộ hoặc theo lô. Thao tác ghi vì thế cực nhanh, và chúng ta còn gộp được nhiều lần ghi vào một lần chạm DB. Nhưng rủi ro của nó là rủi ro chí mạng: nếu cache chết trước khi dữ liệu được đẩy xuống, phần dữ liệu đó mất hẳn. Không có cách nào tái tạo lại nó, vì DB chưa bao giờ nhìn thấy nó. Vì vậy write-back chỉ hợp khi chúng ta chấp nhận mất mát nhỏ, ví dụ đếm lượt xem hoặc thu thập số liệu vận hành. Ngay cả trong trường hợp đó, chúng ta vẫn nên bật persistence và replication cho tầng cache. Với bất kỳ thứ gì liên quan tới tiền, tới đơn hàng hoặc tới quyền truy cập, chúng ta không dùng write-back.

Câu hỏi kiểm tra rất đơn giản và chúng ta nên hỏi nó mỗi lần review code. Nếu Redis chết ngay sau lệnh ghi này, chúng ta có mất dữ liệu không thể tái tạo hay không. Nếu câu trả lời là có, chiến lược này bị cấm ở đây. Nếu câu trả lời là không, ví dụ chúng ta chỉ mất vài lượt xem video, thì write-back là một lựa chọn hợp lý.

**English (bám cấu trúc tiếng Việt)**

Write-back, also called write-behind, writes into the cache first and only then pushes it down to the database later, in an asynchronous way or in batches. The write operation is therefore extremely fast, and we can also merge many writes into a single touch on the database. But its risk is a fatal one: if the cache dies before the data is pushed down, that piece of data is lost for good. There is no way to rebuild it, because the database has never seen it. Therefore write-back only fits when we accept a small amount of loss, for example counting views or collecting operational metrics. Even in that case, we should still turn on persistence and replication for the cache layer. For anything related to money, to orders or to access rights, we do not use write-back.

The check question is very simple and we should ask it every time we review code. If Redis dies right after this write, do we lose data that cannot be rebuilt or not. If the answer is yes, this strategy is banned here. If the answer is no, for example we only lose a few video views, then write-back is a reasonable choice.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| rồi mới đẩy xuống DB sau | and only then pushes it down to the database later |
| theo kiểu bất đồng bộ hoặc theo lô | in an asynchronous way or in batches |
| gộp nhiều lần ghi vào một lần chạm DB | merge many writes into a single touch on the database |
| rủi ro của nó là rủi ro chí mạng | its risk is a fatal one |
| mất hẳn | lost for good |
| không có cách nào tái tạo lại nó | there is no way to rebuild it |
| chấp nhận mất mát nhỏ | accept a small amount of loss |
| thu thập số liệu vận hành | collecting operational metrics |
| ngay cả trong trường hợp đó | even in that case |
| quyền truy cập | access rights |
| chiến lược này bị cấm ở đây | this strategy is banned here |
| một lựa chọn hợp lý | a reasonable choice |

**Thuật ngữ cần nhớ**

- ghi trễ, ghi hoãn → **write-back** / **write-behind**
- đẩy xuống, xả xuống nguồn → **flush down to the source**
- theo lô → **in batches**
- lưu bền → **persistence**
- nhân bản → **replication**

---

## Phần 5 — Chống miss chủ động: refresh-ahead và làm ấm cache

**Tiếng Việt**

Refresh-ahead là cách chúng ta chủ động làm mới một key nóng trước khi nó hết hạn, dựa trên dự đoán rằng nó sắp được đọc tiếp. Nhờ đó không người dùng nào phải gánh lần miss và cú tăng độ trễ ngay tại thời điểm key hết hạn. Hạn chế của nó là chúng ta có thể làm mới nhầm những key không còn ai đọc nữa, và như vậy chúng ta đốt tài nguyên vô ích. Vì vậy refresh-ahead chỉ nên áp dụng cho một tập key nóng nhỏ và đã được đo đạc rõ ràng, chứ không áp dụng đại trà. Một vấn đề họ hàng với nó là cold start: sau mỗi lần deploy hoặc restart, cache rỗng, và một loạt miss sẽ đập thẳng vào DB. Chúng ta giảm rủi ro đó bằng warming, nghĩa là nạp trước các key nóng vào cache trước khi cho instance nhận traffic.

Chúng ta cần nói rõ giới hạn của warming, vì đó là chỗ phân biệt người đã vận hành thật với người chỉ đọc lý thuyết. Thứ nhất, chúng ta không bao giờ biết hết danh sách key nóng, vì vậy warming chỉ che được một phần. Thứ hai, warming không cứu chúng ta khỏi hiện tượng hàng loạt key hết hạn cùng một lúc về sau. Muốn xử lý chuyện đó, chúng ta phải rắc thêm một độ lệch ngẫu nhiên vào TTL, và Bài 5 sẽ nói kỹ về nó. Vì vậy warming là một lớp giảm đau, chứ không phải một lời giải trọn vẹn.

**English (bám cấu trúc tiếng Việt)**

Refresh-ahead is the way we actively refresh a hot key before it expires, based on the prediction that it is about to be read again. Thanks to that no user has to carry the miss and the latency spike right at the moment the key expires. Its limitation is that we may refresh the wrong keys, ones that nobody reads any more, and in that way we burn resources for nothing. Therefore refresh-ahead should only be applied to a small set of hot keys that has been clearly measured, and not applied across the board. A related problem is the cold start: after every deploy or restart, the cache is empty, and a burst of misses will hit the database directly. We reduce that risk with warming, which means we preload the hot keys into the cache before we let the instance take traffic.

We need to state the limits of warming clearly, because that is what separates a person who has really operated a system from a person who has only read the theory. First, we never know the full list of hot keys, therefore warming only covers a part of it. Second, warming does not save us from the phenomenon of a mass of keys expiring at the same time later on. To handle that, we have to sprinkle a random offset into the TTL, and Lesson 5 will talk about it in detail. Therefore warming is a painkiller layer, and not a complete solution.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| chủ động làm mới một key nóng | actively refresh a hot key |
| dựa trên dự đoán rằng | based on the prediction that |
| gánh lần miss và cú tăng độ trễ | carry the miss and the latency spike |
| làm mới nhầm những key | refresh the wrong keys |
| đốt tài nguyên vô ích | burn resources for nothing |
| không áp dụng đại trà | not applied across the board |
| một vấn đề họ hàng với nó | a related problem |
| một loạt miss sẽ đập thẳng vào DB | a burst of misses will hit the database directly |
| nạp trước các key nóng | preload the hot keys |
| trước khi cho instance nhận traffic | before we let the instance take traffic |
| phân biệt người đã vận hành thật với người chỉ đọc lý thuyết | separates a person who has really operated a system from a person who has only read the theory |
| hàng loạt key hết hạn cùng một lúc | a mass of keys expiring at the same time |
| rắc thêm một độ lệch ngẫu nhiên vào TTL | sprinkle a random offset into the TTL |
| một lớp giảm đau | a painkiller layer |

**Thuật ngữ cần nhớ**

- làm mới trước hạn → **refresh-ahead**
- khởi động lạnh → **cold start**
- làm ấm cache, nạp trước → **cache warming** / **preload**
- cú tăng độ trễ → **a latency spike**
- độ lệch ngẫu nhiên → **jitter** / **a random offset**

---

## Phần 6 — Vì sao cache-aside là mặc định của cả ngành

**Tiếng Việt**

Bây giờ chúng ta so sánh cache-aside với write-through trên bốn tiêu chí. Về độ tươi khi đọc, cache-aside có một cửa sổ stale, còn write-through thì tươi hơn. Về độ trễ khi ghi, cache-aside bình thường vì nó chỉ ghi DB, còn write-through cao hơn vì nó ghi vào cả hai nơi. Về chuyện cache chết, cache-aside chỉ suy giảm chứ không sập, vì app đọc thẳng DB và chỉ phải chịu thêm nhiều lần miss. Write-through thì buộc app phụ thuộc vào tầng cache nhiều hơn. Về độ phức tạp, cache-aside đơn giản, còn write-through cần một tầng cache hỗ trợ sẵn. Bốn dòng so sánh này là bộ khung tốt để trả lời bất kỳ câu hỏi nào về hai chiến lược đó.

Có ba lý do khiến cache-aside trở thành mặc định trong công nghiệp. Thứ nhất, nó đơn giản, nên đội ngũ nào cũng đọc hiểu được luồng chỉ trong vài phút. Thứ hai, nó có tính chống chịu tốt, vì khi cache down thì hệ thống suy giảm chứ không chết. Thứ ba, nó không khoá chúng ta vào một nhà cung cấp cụ thể, vì phần logic nằm trong app chứ không nằm trong thư viện. Hai nhược điểm của nó, tức là lần miss đầu tiên và cửa sổ stale, đều chấp nhận được nếu chúng ta đặt TTL hợp lý.

**English (bám cấu trúc tiếng Việt)**

Now we compare cache-aside with write-through on four criteria. On read freshness, cache-aside has a stale window, while write-through is fresher. On write latency, cache-aside is normal because it only writes the database, while write-through is higher because it writes into both places. On what happens when the cache dies, cache-aside only degrades instead of falling over, because the app reads the database directly and only has to take more misses. Write-through, on the other hand, makes the app depend on the cache layer much more. On complexity, cache-aside is simple, while write-through needs a cache layer that already supports it. These four lines of comparison are a good frame for answering any question about those two strategies.

There are three reasons why cache-aside has become the default in the industry. First, it is simple, so any team can read and understand the flow in just a few minutes. Second, it is resilient, because when the cache goes down the system degrades instead of dying. Third, it does not lock us into one specific provider, because the logic sits in the app and not in a library. Its two drawbacks, that is the first miss and the stale window, are both acceptable if we set a sensible TTL.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| trên bốn tiêu chí | on four criteria |
| về độ tươi khi đọc | on read freshness |
| chỉ suy giảm chứ không sập | only degrades instead of falling over |
| chỉ phải chịu thêm nhiều lần miss | only has to take more misses |
| buộc app phụ thuộc vào tầng cache nhiều hơn | makes the app depend on the cache layer much more |
| cần một tầng cache hỗ trợ sẵn | needs a cache layer that already supports it |
| bộ khung tốt để trả lời | a good frame for answering |
| đội ngũ nào cũng đọc hiểu được luồng | any team can read and understand the flow |
| nó có tính chống chịu tốt | it is resilient |
| không khoá chúng ta vào một nhà cung cấp cụ thể | does not lock us into one specific provider |
| đều chấp nhận được nếu | are both acceptable if |
| đặt TTL hợp lý | set a sensible TTL |

**Thuật ngữ cần nhớ**

- tiêu chí → **criteria**
- suy giảm chứ không sập → **degrade instead of failing**
- có tính chống chịu → **resilient**
- bị khoá vào nhà cung cấp → **vendor lock-in**
- độ phức tạp → **complexity**

---

## Phần 7 — Cache ở mức nào: bản ghi thô, kết quả tính, hay mảnh giao diện

**Tiếng Việt**

Câu hỏi tiếp theo không phải là cache thế nào, mà là cache cái gì. Chúng ta có thể cache một bản ghi thô lấy từ DB, cache một kết quả đã tính xong, hoặc cache một mảnh giao diện đã render. Càng gần kết quả cuối cùng thì chúng ta càng cắt được nhiều công tính toán. Nhưng càng gần kết quả cuối cùng thì cache càng dễ vỡ, vì chỉ cần một phần nhỏ bên trong thay đổi là chúng ta phải xoá cả mảnh đó. Mảnh đã render cũng dễ bị trùng lặp, vì cùng một dữ liệu có thể xuất hiện trong nhiều mảnh khác nhau. Ngược lại, cache ở mức thấp như bản ghi thô thì tái dùng được nhiều, nhưng chúng ta vẫn phải tính lại phần logic phía trên. Chúng ta chọn mức nào dựa trên hai yếu tố: chi phí tính lại và tần suất thay đổi.

Một ví dụ cụ thể làm câu trả lời rõ ràng hơn nhiều. Trên một trang sản phẩm, chúng ta cache bản ghi sản phẩm ở mức thấp, vì nhiều màn hình khác nhau cùng dùng nó. Chúng ta cache khối gợi ý sản phẩm liên quan ở mức kết quả đã tính, vì nó tốn nhiều công tính mà lại đổi chậm. Chúng ta không cache cả trang đã render nếu trang đó chứa giá cá nhân hoá, vì khi đó mỗi người dùng cần một bản riêng và tỉ lệ tái dùng gần như bằng không.

**English (bám cấu trúc tiếng Việt)**

The next question is not how we cache, but what we cache. We can cache a raw row taken from the database, cache a result that has already been computed, or cache a rendered interface fragment. The closer we get to the final result, the more computation we cut away. But the closer we get to the final result, the more fragile the cache becomes, because as soon as one small part inside it changes we have to delete that whole fragment. A rendered fragment is also easy to duplicate, because the same data can appear in many different fragments. On the other hand, caching at a low level such as a raw row is highly reusable, but we still have to recompute the logic above it. We choose which level based on two factors: the cost of recomputing and the frequency of change.

A concrete example makes the answer far clearer. On a product page, we cache the product row at the low level, because many different screens use it. We cache the block of related product recommendations at the computed-result level, because it costs a lot of computation while it changes slowly. We do not cache the whole rendered page if that page contains personalised prices, because in that case every user needs their own copy and the reuse rate is almost zero.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| không phải là cache thế nào, mà là cache cái gì | not how we cache, but what we cache |
| một bản ghi thô lấy từ DB | a raw row taken from the database |
| một mảnh giao diện đã render | a rendered interface fragment |
| càng gần kết quả cuối cùng thì càng cắt được nhiều công tính toán | the closer we get to the final result, the more computation we cut away |
| cache càng dễ vỡ | the more fragile the cache becomes |
| chỉ cần một phần nhỏ bên trong thay đổi | as soon as one small part inside it changes |
| dễ bị trùng lặp | easy to duplicate |
| tái dùng được nhiều | highly reusable |
| tính lại phần logic phía trên | recompute the logic above it |
| tần suất thay đổi | the frequency of change |
| tốn nhiều công tính mà lại đổi chậm | costs a lot of computation while it changes slowly |
| giá cá nhân hoá | personalised prices |
| tỉ lệ tái dùng gần như bằng không | the reuse rate is almost zero |

**Thuật ngữ cần nhớ**

- bản ghi thô → **a raw row**
- kết quả đã tính sẵn → **a computed result**
- mảnh giao diện → **a fragment**
- dễ vỡ → **fragile**
- tái dùng → **reuse** / **reusable**

---

## Phần 8 — Góc tech lead: bắt lỗi write-back ở luồng thanh toán

**Tiếng Việt**

Đây là chỗ một tech lead phải bắt lỗi, và nó cũng chính là câu thách đố kinh điển của bài này. Công cụ AI sinh ra một đoạn code dùng write-back cho dữ liệu thanh toán, với lý do là làm như vậy thì nhanh hơn. Đây là một lỗi kiến trúc chứ không phải một lỗi cú pháp, vì đoạn code đó chạy hoàn hảo trong bản demo. Vấn đề nằm ở chỗ write-back có thể mất lệnh ghi khi cache chết, và ở luồng thanh toán thì mất một lệnh ghi nghĩa là mất một giao dịch tiền. Không ai chấp nhận rủi ro đó, dù độ trễ có đẹp đến đâu đi nữa. Cách sửa là chuyển sang write-through, hoặc đơn giản hơn là ghi DB trước rồi mới xoá cache theo kiểu cache-aside. Nhanh không bao giờ bù được cho việc mất tiền của khách hàng.

Chúng ta có thể rút phần này thành một quy tắc review ngắn. Với mỗi lệnh ghi có cache đứng phía trước, chúng ta hỏi dữ liệu này có tái tạo được hay không nếu cache biến mất. Nếu tái tạo được, ví dụ số đếm hoặc số liệu vận hành, chúng ta cho phép ghi bất đồng bộ. Nếu không tái tạo được, chúng ta bắt buộc ghi xuống nguồn trước, và cache chỉ được đứng sau nguồn chứ không được đứng trước nó.

**English (bám cấu trúc tiếng Việt)**

This is where a tech lead has to catch the mistake, and it is also the classic challenge question of this lesson. An AI tool produces a piece of code that uses write-back for payment data, with the reason that doing it this way is faster. This is an architectural mistake and not a syntax mistake, because that piece of code runs perfectly in the demo. The problem lies in the fact that write-back can lose a write when the cache dies, and in the payment flow losing a write means losing a money transaction. Nobody accepts that risk, no matter how nice the latency looks. The fix is to move to write-through, or more simply to write the database first and only then delete the cache in the cache-aside style. Being fast never makes up for losing the customer's money.

We can boil this part down into a short review rule. For every write that has a cache in front of it, we ask whether this data can be rebuilt or not if the cache disappears. If it can be rebuilt, for example counters or operational metrics, we allow an asynchronous write. If it cannot be rebuilt, we require the write to go down to the source first, and the cache is only allowed to sit behind the source, not in front of it.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| chỗ một tech lead phải bắt lỗi | where a tech lead has to catch the mistake |
| câu thách đố kinh điển | the classic challenge question |
| với lý do là làm như vậy thì nhanh hơn | with the reason that doing it this way is faster |
| chạy hoàn hảo trong bản demo | runs perfectly in the demo |
| vấn đề nằm ở chỗ | the problem lies in the fact that |
| mất lệnh ghi | lose a write |
| dù độ trễ có đẹp đến đâu đi nữa | no matter how nice the latency looks |
| nhanh không bao giờ bù được cho | being fast never makes up for |
| rút phần này thành một quy tắc review ngắn | boil this part down into a short review rule |
| có cache đứng phía trước | that has a cache in front of it |
| chúng ta bắt buộc ghi xuống nguồn trước | we require the write to go down to the source first |
| chỉ được đứng sau nguồn chứ không được đứng trước nó | is only allowed to sit behind the source, not in front of it |

**Thuật ngữ cần nhớ**

- lỗi kiến trúc → **an architectural mistake**
- giao dịch tiền → **a money transaction**
- luồng thanh toán → **the payment flow**
- tái tạo lại được → **can be rebuilt** / **recoverable**
- bộ đếm → **a counter**

---

## Mô hình ghi nhớ

**Tiếng Việt**

Cache-aside là mặc định, vì app tự quản và vì khi cache chết thì hệ thống chỉ suy giảm chứ không sập. Write-through thì tươi hơn nhưng ghi chậm hơn, còn write-back thì nhanh nhất nhưng có thể mất dữ liệu, vì vậy chúng ta cấm nó ở mọi luồng liên quan tới tiền.

**English (bám cấu trúc tiếng Việt)**

Cache-aside is the default, because the app manages it itself and because when the cache dies the system only degrades instead of falling over. Write-through is fresher but slower to write, while write-back is the fastest but it can lose data, therefore we ban it on every flow related to money.

---

## Bảng thuật ngữ tổng hợp

| Tiếng Việt | Tiếng Anh | Ghi chú phát âm (Anh-Anh) |
|---|---|---|
| chiến lược cache | caching strategy | strategy — **STRAT**-ə-ji, trọng âm âm tiết đầu |
| cache tự quản bởi app | cache-aside | |
| nạp lười, nạp khi cần | lazy loading | lazy — **LAY**-zi, âm /eɪ/ rõ |
| đọc xuyên qua cache | read-through | âm /θ/ ở "through", không đọc thành "tru" |
| ghi xuyên qua cache | write-through | write — chữ **w** câm, đọc như "rait" |
| ghi trễ, ghi hoãn | write-back / write-behind | |
| làm mới trước hạn | refresh-ahead | |
| đồng bộ | synchronous | **SIN**-krə-nəs — "ch" đọc thành /k/, trọng âm đầu |
| bất đồng bộ | asynchronous | ei-**SIN**-krə-nəs — trọng âm âm tiết thứ hai |
| theo lô | in batches | batch — /bætʃ/, kết thúc bằng /tʃ/ |
| luồng đọc / luồng ghi | the read flow / the write flow | |
| người gọi hàm | the caller | |
| cache lạnh | a cold cache | cache /kæʃ/ — đọc y hệt "cash" |
| khởi động lạnh | cold start | |
| làm ấm cache | cache warming / preload | |
| cửa sổ dữ liệu cũ | a stale window | stale /steɪl/ — "xtêi-l", không phải "xtan" |
| làm mất hiệu lực | invalidate | in-**VAL**-i-deit — trọng âm âm tiết thứ hai |
| thời gian sống | time to live (TTL) | đọc rời từng chữ cái: "tee-tee-el" |
| điều kiện tranh chấp | a race condition | |
| lớp trừu tượng | an abstraction layer | ab-**STRAK**-shən — trọng âm âm tiết thứ hai |
| bộ nạp dữ liệu | a loader | |
| nhà cung cấp | a provider | prə-**VAI**-də — trọng âm giữa, âm /aɪ/ |
| bị khoá vào nhà cung cấp | vendor lock-in | |
| bản sao chép của DB | a database replica | **REP**-li-kə — trọng âm đầu, không đọc "ri-plai-ka" |
| nhân bản | replication | rep-li-**KAY**-shən — trọng âm áp chót |
| lưu bền | persistence | pə-**SIS**-təns — trọng âm giữa |
| đẩy xuống nguồn | flush down to the source | |
| mất hẳn dữ liệu | data loss | |
| tái tạo lại được | recoverable / can be rebuilt | ri-**KUV**-ər-ə-bl — "co" đọc thành /kʌ/ |
| giao dịch | transaction | tran-**ZAK**-shən — âm giữa là /z/, không phải /s/ |
| luồng thanh toán | the payment flow | |
| bộ đếm | a counter | |
| số liệu vận hành | operational metrics | metrics — **MET**-riks, kết thúc bằng cụm /ks/ |
| key nóng | a hot key | |
| cú tăng độ trễ | a latency spike | latency — **LAY**-tən-si |
| hết hạn | to expire / expiry | ik-**SPAI**-ri — trọng âm âm tiết thứ hai |
| độ lệch ngẫu nhiên | jitter / a random offset | |
| một loạt, một đợt dồn | a burst | /bɜːst/ — kết thúc bằng cụm /st/, đừng nuốt âm |
| suy giảm chứ không sập | degrade instead of failing | di-**GREID** — trọng âm âm tiết thứ hai |
| có tính chống chịu | resilient | ri-**ZIL**-i-ənt — trọng âm âm tiết thứ hai, âm /z/ |
| tiêu chí | criteria | krai-**TEER**-i-ə — số nhiều của *criterion* |
| độ phức tạp | complexity | kəm-**PLEK**-sə-ti |
| bản ghi thô | a raw row | row /rəʊ/ — vần với "go", không phải "rau" |
| kết quả đã tính sẵn | a computed result | |
| mảnh giao diện | a fragment | **FRAG**-mənt — trọng âm âm tiết đầu |
| dựng ra giao diện | to render | |
| dễ vỡ | fragile | Anh-Anh **FRAJ**-ail — âm cuối /aɪl/, khác Mỹ |
| tái dùng | reuse | động từ ri-**YOOZ** (/z/), danh từ ri-**YOOSS** (/s/) |
| tần suất thay đổi | the frequency of change | frequency — **FREE**-kwən-si |
| giá cá nhân hoá | personalised prices | |
| gợi ý sản phẩm | product recommendations | rek-ə-men-**DAY**-shənz |
| tình huống cụ thể | a concrete scenario | scenario — Anh-Anh sə-**NAA**-ri-əʊ |
| lỗi kiến trúc | an architectural mistake | ar-ki-**TEK**-chə-rəl |

---

## Bài nói

> **Cách luyện:** bật ghi âm, nói **60–90 giây** cho mỗi đề, **không nhìn tài liệu**. Mỗi bài nói phải dùng ít nhất **sáu thuật ngữ** trong bảng trên. Nghe lại một lần, đánh dấu chỗ bạn ngập ngừng, rồi nói lại đúng đề đó thêm một lần nữa.

1. **Explain to a junior** the read flow and the write flow of cache-aside, in your own words, and say why we delete the key instead of updating it.

2. **A colleague says:** *"Read-through and cache-aside are basically the same thing, so it does not matter which one we pick."* Explain what is actually different between them and when that difference matters.

3. **Someone on your team wants** to use write-back for the payment path, because it makes writes much faster in the load test. Explain why you would push back, and describe exactly what you would do instead.

4. **A teammate claims** that write-through gives absolutely strong consistency, so the cache can never be stale. Explain what is wrong with that claim.

5. **Explain why cache-aside** is the industrial default, even though it has a cold start and a stale window. Give three reasons and be ready to defend each one.

6. **Describe what happens** to your system in the first minute after a deploy, and explain what cache warming can and cannot do about it.

7. **When would you cache** a raw row, and when would you cache a rendered fragment? Give the trade-off in both directions and finish with a concrete example.
