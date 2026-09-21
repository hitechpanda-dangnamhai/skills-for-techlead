# Bài 9 — Case study: URL shortener / pastebin
### Bản song ngữ: tiếng Anh bám cấu trúc tiếng Việt

> **Cách dùng:** đọc đoạn tiếng Việt trước → tự dịch trong đầu → so với bản tiếng Anh bên dưới → **đọc to bản tiếng Anh hai lần**. Lần một đọc chậm cho đúng từ, lần hai đọc liền mạch như đang nói với người phỏng vấn. Đây là case study đầu tiên, nên các phần được xếp đúng theo thứ tự bạn sẽ nói trong bốn mươi lăm phút: làm rõ, ước lượng, vẽ tổng thể, rồi đào sâu.

---

## Phần 1 — Làm rõ yêu cầu và nhận ra bản chất của bài toán

**Tiếng Việt**

Về mặt chức năng, hệ thống này chỉ cần làm hai việc: nhận một đường dẫn dài rồi trả về một mã ngắn, và nhận một mã ngắn rồi chuyển hướng người dùng tới đường dẫn gốc. Ba tính năng thường được hỏi thêm là bí danh tự chọn, thời hạn sử dụng và thống kê lượt bấm. Về mặt phi chức năng, điều quan trọng nhất là lượt đọc nhiều hơn lượt ghi rất nhiều lần. Chúng ta cũng cần độ trễ chuyển hướng thật thấp, độ sẵn sàng cao, và bảo đảm không có hai đường dẫn nào nhận cùng một mã. Chúng ta nên chốt tỷ lệ đọc trên ghi ngay ở phút đầu tiên, vì chính con số đó quyết định toàn bộ kiến trúc.

Về bản chất, hệ thống này là một cuốn từ điển khổng lồ ánh xạ từ mã ngắn sang đường dẫn dài. Điều đáng nói là phần khó không nằm ở việc tạo ra mã, mà nằm ở việc tra ngược thật nhanh và thật nhiều lần. Một đường dẫn được tạo đúng một lần, nhưng nó có thể được bấm hàng triệu lần trong suốt vòng đời của nó. Vì vậy, mọi nỗ lực tối ưu của chúng ta phải dồn vào đường đọc, còn đường ghi thì chúng ta để đơn giản. Nếu chúng ta dành cả buổi để bàn về thuật toán sinh mã, hậu quả là chúng ta tối ưu đúng phần chiếm chưa tới một phần trăm lưu lượng.

**English (bám cấu trúc tiếng Việt)**

Functionally, this system only needs to do two things: take a long URL and return a short code, and take a short code and redirect the user to the original URL. The three features usually asked about on top are a custom alias, an expiry date and click analytics. Non-functionally, the most important thing is that reads outnumber writes by a very large factor. We also need very low redirect latency, high availability, and a guarantee that no two URLs receive the same code. We should settle the read-to-write ratio in the very first minute, because that number decides the whole architecture.

In essence, this system is a huge dictionary mapping from a short code to a long URL. The notable thing is that the hard part does not lie in generating the code, it lies in looking it up in reverse very fast and very many times. One URL is created exactly once, but it may be clicked millions of times over its lifetime. Therefore, all our optimisation effort has to go into the read path, while we keep the write path simple. If we spend the whole session discussing the code-generation algorithm, the consequence is that we optimise exactly the part that carries less than one percent of the traffic.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| chuyển hướng người dùng tới đường dẫn gốc | redirect the user to the original URL |
| bí danh tự chọn | a custom alias |
| lượt đọc nhiều hơn lượt ghi rất nhiều lần | reads outnumber writes by a very large factor |
| bảo đảm không có hai đường dẫn nào nhận cùng một mã | a guarantee that no two URLs receive the same code |
| chốt tỷ lệ đọc trên ghi ngay ở phút đầu tiên | settle the read-to-write ratio in the very first minute |
| một cuốn từ điển khổng lồ | a huge dictionary |
| tra ngược thật nhanh | looking it up in reverse very fast |
| trong suốt vòng đời của nó | over its lifetime |
| mọi nỗ lực tối ưu của chúng ta phải dồn vào | all our optimisation effort has to go into |
| chiếm chưa tới một phần trăm lưu lượng | carries less than one percent of the traffic |

**Thuật ngữ cần nhớ**

- rút gọn đường dẫn → **URL shortening**
- chuyển hướng → **to redirect** / **a redirect**
- bí danh tự chọn → **a custom alias**
- ánh xạ → **a mapping**
- nặng về đọc → **read-heavy**

---

## Phần 2 — Ước lượng nhanh để chốt quy mô

**Tiếng Việt**

Chúng ta giả sử hệ thống lưu một trăm triệu đường dẫn và mỗi bản ghi chiếm khoảng năm trăm byte kể cả siêu dữ liệu. Nhân hai con số đó ra khoảng năm mươi gigabyte, và nếu chúng ta giữ ba bản sao thì tổng khoảng một trăm năm mươi gigabyte. Con số này nhỏ một cách đáng ngạc nhiên, nên toàn bộ tập dữ liệu có thể nằm gọn trong một cụm cơ sở dữ liệu bình thường. Đây là một kết luận rất đáng nói ra, vì nó cho phép chúng ta bỏ qua việc phân mảnh ở giai đoạn đầu. Nói cách khác, dữ liệu không phải là vấn đề của bài toán này.

Bây giờ chúng ta tính đường đọc, vì đó mới là chỗ áp lực nằm. Nếu mỗi đường dẫn được bấm khoảng năm mươi lần trong một năm, chúng ta có năm tỷ lượt chuyển hướng mỗi năm. Chia cho khoảng ba mươi mốt triệu giây trong một năm, chúng ta được khoảng một trăm sáu mươi lượt chuyển hướng mỗi giây ở mức trung bình. Nhân thêm hệ số ba cho giờ cao điểm, chúng ta được khoảng năm trăm lượt mỗi giây, và con số đó vẫn rất dễ chịu. Kết luận rút ra là chúng ta không cần một kiến trúc đồ sộ, chúng ta chỉ cần một tầng cache tốt và một đường chuyển hướng ngắn.

**English (bám cấu trúc tiếng Việt)**

We assume the system stores one hundred million URLs and each record takes about five hundred bytes including the metadata. Multiplying those two numbers gives about fifty gigabytes, and if we keep three replicas then the total is about one hundred and fifty gigabytes. This number is surprisingly small, so the entire dataset can fit comfortably inside one ordinary database cluster. This is a conclusion very much worth saying out loud, because it lets us skip sharding in the early stage. In other words, the data is not the problem in this exercise.

Now we calculate the read path, because that is where the pressure actually sits. If each URL is clicked about fifty times in one year, we have five billion redirects per year. Dividing by about thirty-one million seconds in a year, we get about one hundred and sixty redirects per second on average. Multiplying by a factor of three for the peak hours, we get about five hundred per second, and that number is still very comfortable. The conclusion we draw is that we do not need a massive architecture, we only need a good cache layer and a short redirect path.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| mỗi bản ghi chiếm khoảng năm trăm byte | each record takes about five hundred bytes |
| nhân hai con số đó ra | multiplying those two numbers gives |
| nhỏ một cách đáng ngạc nhiên | surprisingly small |
| có thể nằm gọn trong | can fit comfortably inside |
| một kết luận rất đáng nói ra | a conclusion very much worth saying out loud |
| cho phép chúng ta bỏ qua việc phân mảnh | lets us skip sharding |
| đó mới là chỗ áp lực nằm | that is where the pressure actually sits |
| con số đó vẫn rất dễ chịu | that number is still very comfortable |
| kết luận rút ra là | the conclusion we draw is |
| một kiến trúc đồ sộ | a massive architecture |

**Thuật ngữ cần nhớ**

- bản ghi → **a record**
- tập dữ liệu → **the dataset**
- lượt chuyển hướng mỗi giây → **redirects per second**
- hệ số đỉnh → **the peak factor**
- vừa đủ chứa → **to fit**

---

## Phần 3 — Thiết kế tổng thể và tối ưu đường đọc

**Tiếng Việt**

Đường ghi rất đơn giản: người dùng gửi đường dẫn dài tới dịch vụ tạo mã, dịch vụ xin một mã mới, ghi cặp mã và đường dẫn xuống cơ sở dữ liệu, rồi trả mã về. Đường đọc mới là phần chúng ta chăm chút, và nó chỉ gồm ba bước. Bước một, chúng ta tra mã trong cache và trả về ngay nếu có. Bước hai, nếu cache trượt, chúng ta đọc từ một bản sao chỉ đọc rồi nạp kết quả vào cache. Bước ba, chúng ta trả về mã chuyển hướng cho trình duyệt, và toàn bộ việc này nên xong trong vài mili giây.

Tỷ lệ trúng cache ở bài toán này thường rất cao, vì lưu lượng bấm tuân theo phân phối đuôi dài với một nhóm nhỏ đường dẫn cực nóng. Trong thực tế, một phần trăm số đường dẫn có thể chiếm quá nửa tổng số lượt bấm, nên một cache khiêm tốn cũng chặn được phần lớn lưu lượng. Chúng ta còn có thể đẩy việc chuyển hướng ra tận các điểm biên, vì mã chuyển hướng là dữ liệu công khai và không thay đổi. Khi làm vậy, người dùng ở châu Á không phải đi vòng qua máy chủ ở châu Âu chỉ để nhận một câu trả lời ba mươi byte. Nếu chúng ta để mọi lượt chuyển hướng đi thẳng xuống cơ sở dữ liệu, hậu quả là chúng ta biến một tra cứu khoá đơn giản thành nút thắt của cả hệ thống.

**English (bám cấu trúc tiếng Việt)**

The write path is very simple: the user sends the long URL to the code-creation service, the service asks for a new code, writes the code and URL pair down into the database, and then returns the code. The read path is the part we polish, and it consists of only three steps. Step one, we look the code up in the cache and return immediately if it is there. Step two, if the cache misses, we read from a read replica and then load the result into the cache. Step three, we return the redirect code to the browser, and all of this should finish within a few milliseconds.

The cache hit rate in this problem is usually very high, because click traffic follows a long-tail distribution with a small group of extremely hot URLs. In practice, one percent of the URLs may account for more than half of all the clicks, so even a modest cache blocks most of the traffic. We can also push the redirect out to the edge locations themselves, because a redirect code is public data and it does not change. When we do that, a user in Asia does not have to travel around through a server in Europe just to receive a thirty-byte answer. If we let every redirect go straight down to the database, the consequence is that we turn a simple key lookup into the bottleneck of the whole system.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| đường đọc mới là phần chúng ta chăm chút | the read path is the part we polish |
| tra mã trong cache | look the code up in the cache |
| nếu cache trượt | if the cache misses |
| tỷ lệ trúng cache | the cache hit rate |
| phân phối đuôi dài | a long-tail distribution |
| có thể chiếm quá nửa tổng số lượt bấm | may account for more than half of all the clicks |
| một cache khiêm tốn | a modest cache |
| đẩy việc chuyển hướng ra tận các điểm biên | push the redirect out to the edge locations |
| không phải đi vòng qua máy chủ ở châu Âu | does not have to travel around through a server in Europe |
| một tra cứu khoá đơn giản | a simple key lookup |

**Thuật ngữ cần nhớ**

- tỷ lệ trúng cache → **the cache hit rate**
- trượt cache → **a cache miss**
- bản sao chỉ đọc → **a read replica**
- phân phối đuôi dài → **a long-tail distribution**
- tra cứu khoá → **a key lookup**

---

## Phần 4 — Sinh mã ngắn: đếm rồi đổi cơ số, hay băm

**Tiếng Việt**

Có hai hướng sinh mã ngắn, và mỗi hướng đổi một thứ lấy một thứ khác. Hướng thứ nhất là chúng ta lấy một số nguyên tăng dần rồi đổi nó sang cơ số sáu mươi hai, dùng chữ số cùng chữ cái thường và chữ cái hoa. Cách này cho mã rất gọn và không bao giờ trùng, vì mỗi số chỉ ứng với đúng một mã. Nhược điểm là mã sinh ra theo thứ tự, nên người ngoài đoán được mã kế tiếp và có thể duyệt qua toàn bộ kho đường dẫn. Hướng thứ hai là chúng ta băm đường dẫn hoặc sinh chuỗi ngẫu nhiên, nhờ đó mã khó đoán, nhưng chúng ta phải kiểm tra trùng trước khi ghi.

Chúng ta chọn độ dài mã dựa trên không gian địa chỉ mà chúng ta cần. Với cơ số sáu mươi hai và bảy ký tự, chúng ta có khoảng ba nghìn năm trăm tỷ tổ hợp, và con số đó dư sức cho một trăm triệu đường dẫn. Trong phỏng vấn, cách trả lời mạnh là chúng ta nói rằng chúng ta chọn hướng đếm rồi đổi cơ số cho các đường dẫn công khai, và chọn hướng ngẫu nhiên cho các đường dẫn riêng tư. Chúng ta cũng nói rằng nếu tính đoán được là một rủi ro thật, chúng ta trộn thêm một phép hoán vị lên số đếm trước khi đổi cơ số. Nếu chúng ta dùng băm mà quên kiểm tra trùng, hậu quả là một ngày nào đó một người dùng bấm vào mã của mình và rơi vào trang của người khác.

**English (bám cấu trúc tiếng Việt)**

There are two directions for generating a short code, and each direction trades one thing for another. The first direction is that we take an increasing integer and convert it into base sixty-two, using digits together with lowercase letters and uppercase letters. This way gives a very compact code and it never collides, because each number maps to exactly one code. The drawback is that the codes come out in order, so an outsider can guess the next code and can walk through the entire URL store. The second direction is that we hash the URL or generate a random string, thanks to which the code is hard to guess, but we have to check for collisions before writing.

We choose the code length based on the address space we need. With base sixty-two and seven characters, we have about three and a half trillion combinations, and that number is far more than enough for one hundred million URLs. In an interview, the strong way to answer is that we say we choose the counter-then-base-conversion direction for public URLs, and we choose the random direction for private URLs. We also say that if guessability is a real risk, we mix in a permutation over the counter before we convert the base. If we use hashing but forget the collision check, the consequence is that one day a user clicks on their own code and lands on somebody else's page.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| đổi nó sang cơ số sáu mươi hai | convert it into base sixty-two |
| chữ cái thường và chữ cái hoa | lowercase letters and uppercase letters |
| không bao giờ trùng | it never collides |
| mã sinh ra theo thứ tự | the codes come out in order |
| duyệt qua toàn bộ kho đường dẫn | walk through the entire URL store |
| không gian địa chỉ | the address space |
| dư sức cho | far more than enough for |
| nếu tính đoán được là một rủi ro thật | if guessability is a real risk |
| trộn thêm một phép hoán vị lên số đếm | mix in a permutation over the counter |
| rơi vào trang của người khác | lands on somebody else's page |

**Thuật ngữ cần nhớ**

- cơ số sáu mươi hai → **base sixty-two**
- va chạm mã → **a collision**
- không gian địa chỉ → **the address space**
- đoán được → **guessable**
- chuỗi ngẫu nhiên → **a random string**

---

## Phần 5 — Sinh số định danh duy nhất trong hệ phân tán

**Tiếng Việt**

Nếu chúng ta dùng số đếm, chúng ta phải trả lời được câu hỏi số đếm đó đến từ đâu khi có nhiều máy cùng chạy. Cách ngây thơ là dùng cột tự tăng của một cơ sở dữ liệu duy nhất, nhưng cách đó biến chính cơ sở dữ liệu đó thành nút thắt và thành điểm chết duy nhất. Cách thứ nhất tốt hơn là cấp phát theo dải, nghĩa là mỗi máy xin trước một dải mười nghìn số rồi tự cấp phát cục bộ trong dải đó. Nhờ vậy, chúng ta chỉ chạm vào bộ đếm trung tâm một lần cho mười nghìn mã, và các máy chạy độc lập gần như hoàn toàn. Cái giá là các mã sẽ có khoảng trống khi một máy chết giữa chừng và mang theo phần dải chưa dùng.

Cách thứ hai là dùng lược đồ kiểu bông tuyết, trong đó một số sáu mươi tư bit được ghép từ dấu thời gian, mã máy và một số thứ tự nội bộ. Cách này không cần phối hợp toàn cục, vì mỗi máy chỉ cần biết mã của chính nó, và các số vẫn tăng dần theo thời gian nên chúng thân thiện với chỉ mục. Cách thứ ba là dựng một dịch vụ sinh khoá riêng, sinh sẵn hàng loạt mã chưa dùng và phát dần cho các máy khi cần. Khi trình bày, chúng ta nên nói rõ đánh đổi trung tâm của cả ba cách, đó là tính tuần tự đổi lấy tính phân tán và tính khó đoán. Nếu chúng ta giữ cột tự tăng ở một cơ sở dữ liệu duy nhất, hậu quả là mọi lần tạo mã đều phải đi qua một điểm, và điểm đó quyết định trần thông lượng của toàn hệ thống.

**English (bám cấu trúc tiếng Việt)**

If we use a counter, we have to be able to answer the question of where that counter comes from when many machines are running at once. The naive way is to use the auto-increment column of one single database, but that way turns that very database into a bottleneck and into a single point of failure. The first better way is range allocation, which means each machine requests a range of ten thousand numbers in advance and then allocates locally within that range. Thanks to that, we touch the central counter only once per ten thousand codes, and the machines run almost completely independently. The price is that the codes will have gaps when a machine dies halfway and takes the unused part of its range with it.

The second way is to use a snowflake-style scheme, in which a sixty-four-bit number is assembled from a timestamp, a machine id and an internal sequence number. This way needs no global coordination, because each machine only needs to know its own id, and the numbers still increase over time so they are index-friendly. The third way is to build a separate key generation service, which pre-generates batches of unused codes and hands them out to the machines as needed. When we present this, we should clearly state the central trade-off of all three ways, which is sequentiality traded for distribution and unguessability. If we keep the auto-increment column on one single database, the consequence is that every code creation has to pass through one point, and that point decides the throughput ceiling of the whole system.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| số đếm đó đến từ đâu | where that counter comes from |
| cột tự tăng | the auto-increment column |
| cấp phát theo dải | range allocation |
| xin trước một dải mười nghìn số | requests a range of ten thousand numbers in advance |
| chúng ta chỉ chạm vào bộ đếm trung tâm một lần | we touch the central counter only once |
| các mã sẽ có khoảng trống | the codes will have gaps |
| mang theo phần dải chưa dùng | takes the unused part of its range with it |
| được ghép từ dấu thời gian | is assembled from a timestamp |
| không cần phối hợp toàn cục | needs no global coordination |
| thân thiện với chỉ mục | index-friendly |
| trần thông lượng của toàn hệ thống | the throughput ceiling of the whole system |

**Thuật ngữ cần nhớ**

- định danh duy nhất → **a unique identifier**
- cấp phát theo dải → **range allocation**
- dấu thời gian → **a timestamp**
- số thứ tự → **a sequence number**
- dịch vụ sinh khoá → **a key generation service**

---

## Phần 6 — Mã 301 so với 302, và vì sao đếm phải bất đồng bộ

**Tiếng Việt**

Khi trả về chuyển hướng, chúng ta chọn giữa mã ba trăm linh một và mã ba trăm linh hai, và lựa chọn này ảnh hưởng trực tiếp tới việc đếm. Mã ba trăm linh một nghĩa là chuyển hướng vĩnh viễn, nên trình duyệt lưu lại và những lần sau nó đi thẳng mà không hỏi máy chủ của chúng ta nữa. Cách đó nhanh hơn cho người dùng và tốt cho tối ưu công cụ tìm kiếm, nhưng chúng ta mất hầu hết các lượt bấm sau lần đầu. Mã ba trăm linh hai nghĩa là chuyển hướng tạm thời, nên mỗi lượt bấm đều đi qua máy chủ và chúng ta đếm được đầy đủ. Vì vậy, chúng ta chọn theo nhu cầu thống kê chứ không chọn theo thói quen.

Dù chọn mã nào, chúng ta cũng không được đếm ngay trong đường chuyển hướng bằng một câu lệnh cộng một vào cơ sở dữ liệu. Lý do là mọi lượt bấm của cùng một đường dẫn đều ghi vào đúng một dòng, nên dòng đó trở thành một dòng nóng và mọi lượt ghi phải xếp hàng chờ khoá. Cách đúng là chúng ta đẩy một sự kiện bấm vào hàng đợi rồi trả chuyển hướng về ngay lập tức. Một luồng xử lý phía sau sẽ gộp các sự kiện theo lô và cập nhật số liệu định kỳ, hoặc chúng ta tăng một bộ đếm trong bộ nhớ rồi ghi xuống theo lô. Nếu chúng ta đếm đồng bộ, hậu quả là một chiến dịch quảng cáo thành công sẽ tự tay đánh sập chính đường dẫn của nó.

**English (bám cấu trúc tiếng Việt)**

When we return the redirect, we choose between the three hundred and one code and the three hundred and two code, and this choice affects the counting directly. The three hundred and one code means a permanent redirect, so the browser caches it and on later occasions it goes straight through without asking our server again. That way is faster for the user and good for search engine optimisation, but we lose most of the clicks after the first one. The three hundred and two code means a temporary redirect, so every click passes through the server and we can count them all. Therefore, we choose according to the analytics need rather than according to habit.

Whichever code we choose, we must not do the counting inside the redirect path with a statement that adds one in the database. The reason is that every click on the same URL writes into exactly one row, so that row becomes a hot row and every write has to queue up waiting for the lock. The correct way is that we push a click event into a queue and return the redirect immediately. A worker behind it will group the events into batches and update the figures periodically, or we increment a counter in memory and then write it down in batches. If we count synchronously, the consequence is that a successful advertising campaign will knock down its own link with its own hands.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| chuyển hướng vĩnh viễn | a permanent redirect |
| trình duyệt lưu lại | the browser caches it |
| nó đi thẳng mà không hỏi máy chủ của chúng ta nữa | it goes straight through without asking our server again |
| chúng ta mất hầu hết các lượt bấm sau lần đầu | we lose most of the clicks after the first one |
| chúng ta đếm được đầy đủ | we can count them all |
| không chọn theo thói quen | not according to habit |
| ghi vào đúng một dòng | writes into exactly one row |
| trở thành một dòng nóng | becomes a hot row |
| phải xếp hàng chờ khoá | has to queue up waiting for the lock |
| gộp các sự kiện theo lô | group the events into batches |
| sẽ tự tay đánh sập chính đường dẫn của nó | will knock down its own link with its own hands |

**Thuật ngữ cần nhớ**

- chuyển hướng vĩnh viễn → **a permanent redirect (301)**
- chuyển hướng tạm thời → **a temporary redirect (302)**
- dòng nóng → **a hot row**
- sự kiện bấm → **a click event**
- gộp theo lô → **batching**

---

## Phần 7 — Ba tính năng thêm và chỗ chúng làm mọi thứ phức tạp

**Tiếng Việt**

Bí danh tự chọn nghe đơn giản nhưng nó chạm vào không gian khoá của cơ chế sinh mã tự động. Chúng ta phải kiểm tra tính duy nhất khi người dùng đăng ký bí danh, và chúng ta phải bảo đảm mã tự sinh không bao giờ đụng vào một bí danh đã có. Cách sạch nhất là chúng ta tách hai không gian ra, ví dụ mã tự sinh luôn có đúng bảy ký tự còn bí danh phải khác độ dài đó. Chúng ta cũng cần một danh sách từ cấm, để không ai đăng ký được những bí danh trùng với đường dẫn quản trị của chính hệ thống. Nếu chúng ta để hai không gian trộn vào nhau, hậu quả là một ngày nào đó bộ sinh mã tạo ra đúng một chuỗi mà người dùng đã giữ.

Thời hạn sử dụng kéo theo hai việc: chúng ta lưu thời điểm hết hạn cùng bản ghi, và chúng ta cần một tác vụ dọn dẹp chạy định kỳ. Chúng ta không nên xoá ngay tại lúc đọc, vì như thế biến đường đọc thành đường ghi và làm hỏng đúng phần nhạy cảm nhất. Thống kê thì phải nằm hoàn toàn ngoài đường chuyển hướng, chảy từ hàng đợi vào một kho dữ liệu phân tích riêng. Cách trình bày ghi điểm là chúng ta nói rằng chuyển hướng là nhiệm vụ chính còn đếm là việc phụ, nên đếm không bao giờ được phép làm chậm chuyển hướng. Nếu chúng ta nhét cả ba tính năng này vào đường nóng, hậu quả là chúng ta biến một hệ thống lẽ ra rất nhanh thành một hệ thống chậm vì những việc không ai nhìn thấy.

**English (bám cấu trúc tiếng Việt)**

A custom alias sounds simple but it touches the key space of the automatic code generator. We have to check uniqueness when a user registers an alias, and we have to guarantee that a generated code never collides with an existing alias. The cleanest way is that we separate the two spaces, for example generated codes always have exactly seven characters while an alias must have a different length from that. We also need a blocklist of words, so that nobody can register an alias that clashes with the system's own administrative paths. If we let the two spaces mix together, the consequence is that one day the generator produces exactly the string a user already holds.

An expiry date brings two things with it: we store the expiry time alongside the record, and we need a cleanup job running periodically. We should not delete at read time, because that turns the read path into a write path and damages exactly the most sensitive part. Analytics has to sit completely outside the redirect path, flowing from the queue into a separate analytical datastore. The way of presenting that scores is that we say the redirect is the main duty while counting is a side job, so counting is never allowed to slow the redirect down. If we stuff all three features into the hot path, the consequence is that we turn a system which should be very fast into a slow one because of work nobody can see.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| chạm vào không gian khoá | touches the key space |
| kiểm tra tính duy nhất | check uniqueness |
| không bao giờ đụng vào một bí danh đã có | never collides with an existing alias |
| chúng ta tách hai không gian ra | we separate the two spaces |
| một danh sách từ cấm | a blocklist of words |
| trùng với đường dẫn quản trị | clashes with the administrative paths |
| một tác vụ dọn dẹp chạy định kỳ | a cleanup job running periodically |
| biến đường đọc thành đường ghi | turns the read path into a write path |
| chuyển hướng là nhiệm vụ chính còn đếm là việc phụ | the redirect is the main duty while counting is a side job |
| nhét cả ba tính năng này vào đường nóng | stuff all three features into the hot path |

**Thuật ngữ cần nhớ**

- không gian khoá → **the key space**
- tính duy nhất → **uniqueness**
- hết hạn → **expiry** / **to expire**
- tác vụ dọn dẹp → **a cleanup job**
- đường nóng → **the hot path**

---

## Mô hình ghi nhớ

**Tiếng Việt**

Đường dẫn được tạo một lần nhưng được đọc hàng triệu lần. Vì vậy chúng ta tối ưu đường đọc bằng cache và bản sao, chúng ta sinh định danh phân tán để không đụng nhau, và chúng ta đẩy việc đếm ra khỏi đường chuyển hướng.

**English (bám cấu trúc tiếng Việt)**

A URL is created once but read millions of times. Therefore we optimise the read path with a cache and replicas, we generate distributed identifiers so that they never collide, and we push the counting out of the redirect path.

---

## Bảng thuật ngữ tổng hợp

| Tiếng Việt | Tiếng Anh | Ghi chú phát âm |
|---|---|---|
| rút gọn đường dẫn | URL shortening | *URL* đọc từng chữ cái: "iu-a-el" |
| chuyển hướng | to redirect / a redirect | động từ nhấn âm cuối "ri-đi-**REKT**"; danh từ nhấn đầu |
| bí danh tự chọn | a custom alias | *alias* — "**ÂY**-li-ợs", trọng âm đầu, không đọc "a-li-át" |
| ánh xạ | a mapping | |
| nặng về đọc | read-heavy | |
| bản ghi | a record | danh từ nhấn đầu "**RE**-cợd"; động từ *record* nhấn âm sau |
| tập dữ liệu | the dataset | |
| hệ số đỉnh | the peak factor | *peak* /piːk/ — âm "i" dài, phân biệt với *pick* |
| tỷ lệ trúng cache | the cache hit rate | *cache* /kæʃ/ — đọc đúng như "cash" |
| trượt cache | a cache miss | |
| bản sao chỉ đọc | a read replica | *replica* Anh — "**RE**-pli-cơ", trọng âm đầu |
| phân phối đuôi dài | a long-tail distribution | |
| tra cứu khoá | a key lookup | |
| điểm biên | an edge location | |
| cơ số sáu mươi hai | base sixty-two | |
| va chạm mã | a collision | cơ-**LI**-zhợn, âm giữa là /ʒ/ |
| không gian địa chỉ | the address space | *address* Anh nhấn âm sau: "ơ-**DRES**" |
| đoán được | guessable | *guess* /ɡes/ — chữ **u câm**, đọc "ghét-sơ-bợl" |
| chuỗi ngẫu nhiên | a random string | |
| định danh duy nhất | a unique identifier | *unique* — iu-**NIIK**, trọng âm âm thứ hai |
| cấp phát theo dải | range allocation | *allocation* — a-lơ-**KÂY**-shợn, trọng âm âm thứ ba |
| cột tự tăng | the auto-increment column | *column* — "**CO**-lợm", chữ **n cuối câm** |
| dấu thời gian | a timestamp | âm cuối **-mp** phải bật ra |
| số thứ tự | a sequence number | *sequence* — "**SI**-kwợns", trọng âm đầu |
| dịch vụ sinh khoá | a key generation service | |
| tuần tự | sequential | si-**KWEN**-shợl, trọng âm âm thứ hai |
| thông lượng | throughput | âm **th** /θ/ đầu, "THRU-put" |
| chuyển hướng vĩnh viễn | a permanent redirect (301) | *permanent* — "**PƠ**-mơ-nợnt", trọng âm đầu |
| chuyển hướng tạm thời | a temporary redirect (302) | Anh /ˈtemprəri/ — "**TEM**-prơ-ri", ba âm tiết, trọng âm đầu |
| dòng nóng | a hot row | *row* /rəʊ/ — "râu", không đọc "rao" |
| sự kiện bấm | a click event | |
| gộp theo lô | batching | |
| hàng đợi | a queue | /kjuː/ — đọc như chữ "Q", bốn chữ cái cuối câm |
| không gian khoá | the key space | |
| tính duy nhất | uniqueness | |
| hết hạn | expiry / to expire | Anh *expiry* — ịc-**SPAI**-ơ-ri, trọng âm âm thứ hai |
| tác vụ dọn dẹp | a cleanup job | |
| đường nóng | the hot path | *path* có âm **th** cuối, không thành "pát" |
| danh sách chặn | a blocklist | |
| tối ưu công cụ tìm kiếm | search engine optimisation (SEO) | *search* /sɜːtʃ/ — âm cuối **-ch** |
| kho dữ liệu phân tích | an analytical datastore | *analytics* — a-nơ-**LI**-tics, trọng âm âm thứ ba |

---

## Bài nói

> **Cách luyện:** bật ghi âm, nói **60–90 giây** cho mỗi đề, **không nhìn tài liệu** khi nói. Mỗi bài phải dùng ít nhất **sáu thuật ngữ** trong bảng trên. Với case study, hãy tập nói theo đúng thứ tự bảy phần ở trên, vì đó cũng là thứ tự bạn sẽ trình bày trên bảng.

1. **Explain to a junior engineer** why a URL shortener is a read-heavy system, and what that single fact tells you about where to spend your design effort.

2. **A colleague says:** *"We'll just use `UPDATE links SET clicks = clicks + 1` in the redirect handler, it keeps the count exact."* **Explain what is wrong with that**, and describe what you would do instead and what accuracy you give up.

3. **Someone on your team wants to** use a single database with an auto-increment column to generate the short codes. **Explain why you would push back**, and describe two alternatives with their trade-offs.

4. **Describe what happens when** a marketing team ships a link that gets ten thousand clicks per second for one hour, and walk through the path a single click takes in your design.

5. **When would you choose** a 301 permanent redirect over a 302 temporary one? State what you gain and what you lose in each direction.

6. **Explain to a junior engineer** the trade-off between generating codes from a counter and generating them from a hash, and say which one you would pick for a link that must not be guessable.

7. **Explain to a product manager** why adding custom aliases is more work than it looks, using the key space and the blocklist to make the point.
