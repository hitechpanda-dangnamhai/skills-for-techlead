# Bài 11 — Cache ở tầng HTTP và CDN
### Bản song ngữ: tiếng Anh bám cấu trúc tiếng Việt

> **Cách dùng bài này:** đọc đoạn tiếng Việt trước → tự dịch trong đầu → so với đoạn tiếng Anh bên dưới → **đọc to bản tiếng Anh hai lần**: lần một đọc chậm cho đúng, lần hai đọc trôi như đang nói với người phỏng vấn.

---

## Phần 1 — Cache-Control và các chỉ thị chính

**Tiếng Việt**

Tầng cache xa nhất tính từ máy chủ của chúng ta chính là tầng HTTP, và nó được điều khiển gần như hoàn toàn bằng một header duy nhất. Header đó là Cache-Control, và chuẩn mô tả nó là RFC 9111. Chỉ thị max-age đặt thời gian sống tính theo giây cho cache phía client, tức là trình duyệt. Chỉ thị s-maxage đặt thời gian sống cho các cache dùng chung như CDN, và nó ghi đè lên max-age ở tầng đó. Chỉ thị private nghĩa là chỉ trình duyệt được phép lưu, còn CDN thì không được. Chỉ thị public nghĩa là các cache dùng chung cũng được phép lưu một bản sao. Hai chỉ thị còn lại, no-store và no-cache, là chỗ dễ nhầm nhất nên chúng ta sẽ nói riêng về chúng ở phần sau.

Điều quan trọng là chúng ta phải nghĩ theo từng loại nội dung, chứ không đặt một cấu hình chung cho tất cả. Với tệp tĩnh có phiên bản trong tên, chúng ta đặt public, đặt thời gian sống rất dài, và đánh dấu nó là bất biến. Với một API công khai trả về bằng phương thức GET, chúng ta đặt s-maxage ngắn và dựa thêm vào ETag. Với bất kỳ response nào chứa dữ liệu của một người dùng cụ thể, chúng ta đặt private hoặc cấm lưu hẳn.

**English (bám cấu trúc tiếng Việt)**

The cache layer that is furthest from our server is exactly the HTTP layer, and it is controlled almost entirely by one single header. That header is Cache-Control, and the standard describing it is RFC 9111. The max-age directive sets the time to live in seconds for the cache on the client side, that is the browser. The s-maxage directive sets the time to live for shared caches such as a CDN, and it overrides max-age at that layer. The private directive means that only the browser is allowed to store it, while the CDN is not. The public directive means that shared caches are allowed to store a copy as well. The two remaining directives, no-store and no-cache, are the easiest ones to confuse so we will talk about them separately in the next part.

The important thing is that we have to think per type of content, instead of setting one common configuration for everything. For a static file with a version in its name, we set public, we set a very long time to live, and we mark it as immutable. For a public API returned through the GET method, we set a short s-maxage and lean additionally on the ETag. For any response containing the data of one specific user, we set private or forbid storing it altogether.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| tầng cache xa nhất tính từ máy chủ của chúng ta | the cache layer that is furthest from our server |
| được điều khiển gần như hoàn toàn bằng | is controlled almost entirely by |
| chuẩn mô tả nó là | the standard describing it is |
| nó ghi đè lên max-age ở tầng đó | it overrides max-age at that layer |
| chỉ trình duyệt được phép lưu | only the browser is allowed to store it |
| chỗ dễ nhầm nhất | the easiest ones to confuse |
| nghĩ theo từng loại nội dung | think per type of content |
| một cấu hình chung cho tất cả | one common configuration for everything |
| đánh dấu nó là bất biến | mark it as immutable |
| dựa thêm vào ETag | lean additionally on the ETag |
| cấm lưu hẳn | forbid storing it altogether |

**Thuật ngữ cần nhớ**

- chỉ thị điều khiển cache → **a cache directive**
- cache dùng chung → **a shared cache**
- ghi đè lên → **to override**
- bất biến → **immutable**
- tệp tĩnh → **a static file**

---

## Phần 2 — no-cache khác no-store: cái bẫy kinh điển

**Tiếng Việt**

Đây là cặp chỉ thị bị nhầm nhiều nhất trong toàn bộ chuẩn HTTP. Chỉ thị no-store nghĩa là cấm lưu, tức là không một cache nào được phép giữ lại bản sao của response. Chúng ta dùng nó cho dữ liệu nhạy cảm, ví dụ trang tài khoản hoặc dữ liệu thanh toán. Chỉ thị no-cache thì ngược lại: cache vẫn được phép lưu bản sao, nhưng nó phải hỏi lại máy chủ trước khi dùng bản đó. Nói cách khác, no-cache nghĩa là kiểm tra lại chứ không phải nghĩa là cấm lưu. Nếu chúng ta đặt no-cache cho dữ liệu nhạy cảm mà tưởng rằng mình đã cấm lưu, thì bản sao vẫn nằm trên đĩa của trình duyệt và nằm trên các cache trung gian.

Có một cách nhớ rất ngắn mà chúng ta nên dùng khi nói. No-store nghĩa là đừng giữ, còn no-cache nghĩa là giữ nhưng phải hỏi lại. Người phỏng vấn rất hay dùng cặp này để kiểm tra xem chúng ta đọc chuẩn hay chỉ đoán theo tên gọi. Nếu chúng ta trả lời đúng và còn nêu được hậu quả của việc nhầm, đó là một điểm cộng rõ rệt.

**English (bám cấu trúc tiếng Việt)**

This is the pair of directives that gets confused the most in the whole HTTP standard. The no-store directive means storing is forbidden, that is no cache at all is allowed to keep a copy of the response. We use it for sensitive data, for example an account page or payment data. The no-cache directive is the opposite: the cache is still allowed to store a copy, but it has to ask the server again before using that copy. In other words, no-cache means revalidate instead of meaning do not store. If we set no-cache on sensitive data while believing that we have forbidden storage, then the copy still sits on the browser's disk and sits on the intermediate caches.

There is a very short way to remember it that we should use when we speak. No-store means do not keep it, while no-cache means keep it but ask again. Interviewers very often use this pair to test whether we have read the standard or are merely guessing from the names. If we answer correctly and also name the consequence of getting it wrong, that is a clear plus point.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| bị nhầm nhiều nhất trong toàn bộ chuẩn HTTP | gets confused the most in the whole HTTP standard |
| không một cache nào được phép giữ lại bản sao | no cache at all is allowed to keep a copy |
| dữ liệu nhạy cảm | sensitive data |
| nó phải hỏi lại máy chủ trước khi dùng bản đó | it has to ask the server again before using that copy |
| nghĩa là kiểm tra lại chứ không phải nghĩa là cấm lưu | means revalidate instead of meaning do not store |
| mà tưởng rằng mình đã cấm lưu | while believing that we have forbidden storage |
| nằm trên các cache trung gian | sits on the intermediate caches |
| giữ nhưng phải hỏi lại | keep it but ask again |
| chỉ đoán theo tên gọi | merely guessing from the names |
| nêu được hậu quả của việc nhầm | name the consequence of getting it wrong |

**Thuật ngữ cần nhớ**

- cấm lưu → **no-store**
- lưu nhưng phải kiểm tra lại → **no-cache**
- dữ liệu nhạy cảm → **sensitive data**
- cache trung gian → **an intermediate cache**
- kiểm tra lại → **to revalidate**

---

## Phần 3 — ETag và cơ chế kiểm tra lại

**Tiếng Việt**

ETag là một dấu vân tay của nội dung mà máy chủ gắn kèm vào response. Lần sau, client gửi lại giá trị đó trong header If-None-Match để hỏi rằng nội dung có thay đổi hay không. Nếu nội dung chưa đổi, máy chủ trả về mã ba trăm linh bốn và nó không gửi kèm phần thân. Nhờ đó chúng ta tiết kiệm được băng thông, vì phần thân thường lớn hơn phần header rất nhiều lần. Chúng ta gọi cơ chế này là kiểm tra lại, và nó cho chúng ta độ tươi cao hơn so với việc chỉ dựa vào thời gian sống. Một cơ chế tương đương là cặp Last-Modified và If-Modified-Since, nhưng nó dựa trên thời điểm sửa đổi thay vì dựa trên nội dung. So với việc chỉ đặt max-age, cách này vừa tươi hơn vừa rẻ, vì một lần kiểm tra lại thành công gần như không tốn dữ liệu.

Chúng ta nên nêu rõ rằng việc kiểm tra lại vẫn tốn một vòng đi về trên mạng. Vì vậy cách tối ưu thường là kết hợp: đặt một khoảng max-age ngắn để không phải hỏi liên tục, rồi dựa vào ETag khi khoảng đó hết. Với tệp tĩnh có phiên bản trong tên, chúng ta thậm chí không cần kiểm tra lại chút nào. Lý do là tên tệp đã đổi mỗi khi nội dung đổi, nên bản cũ không bao giờ sai.

**English (bám cấu trúc tiếng Việt)**

An ETag is a fingerprint of the content that the server attaches to the response. Next time, the client sends that value back in the If-None-Match header to ask whether the content has changed or not. If the content has not changed, the server returns the code three hundred and four and it does not send the body along. Thanks to that we save bandwidth, because the body is usually many times larger than the headers. We call this mechanism revalidation, and it gives us higher freshness compared with relying on the time to live alone. An equivalent mechanism is the pair Last-Modified and If-Modified-Since, but it is based on the modification time instead of being based on the content. Compared with setting max-age alone, this way is both fresher and cheaper, because a successful revalidation costs almost no data.

We should state clearly that a revalidation still costs one round trip on the network. Therefore the optimal approach is usually a combination: set a short max-age so that we do not have to ask constantly, and then lean on the ETag once that period runs out. For a static file with a version in its name, we do not even need to revalidate at all. The reason is that the file name has already changed whenever the content changed, so an old copy is never wrong.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| một dấu vân tay của nội dung | a fingerprint of the content |
| gắn kèm vào response | attaches to the response |
| nó không gửi kèm phần thân | it does not send the body along |
| lớn hơn phần header rất nhiều lần | many times larger than the headers |
| so với việc chỉ dựa vào thời gian sống | compared with relying on the time to live alone |
| dựa trên thời điểm sửa đổi | based on the modification time |
| vừa tươi hơn vừa rẻ | both fresher and cheaper |
| gần như không tốn dữ liệu | costs almost no data |
| vẫn tốn một vòng đi về trên mạng | still costs one round trip on the network |
| để không phải hỏi liên tục | so that we do not have to ask constantly |
| chúng ta thậm chí không cần kiểm tra lại chút nào | we do not even need to revalidate at all |
| bản cũ không bao giờ sai | an old copy is never wrong |

**Thuật ngữ cần nhớ**

- dấu vân tay nội dung → **a content fingerprint**
- kiểm tra lại → **revalidation**
- phần thân response → **the body**
- băng thông → **bandwidth**
- thời điểm sửa đổi → **the modification time**

---

## Phần 4 — CDN giải bài toán mà Redis không giải được

**Tiếng Việt**

CDN giải một bài toán mà Redis không giải được, đó là khoảng cách địa lý. Redis nằm trong trung tâm dữ liệu của chúng ta, vì vậy một người dùng ở châu Âu vẫn phải chờ gói tin đi vòng qua nửa vòng trái đất. CDN đặt bản sao ở các điểm hiện diện gần người dùng, vì vậy nó cắt thẳng phần độ trễ do khoảng cách gây ra. Lợi ích thứ hai là nó giảm tải cho máy chủ gốc, vì phần lớn request không bao giờ đi tới nơi đó. Chúng ta đặt lên CDN các tệp tĩnh như ảnh, mã JavaScript và CSS, cùng với những response GET công khai. Còn Redis vẫn giữ vai trò cache cho dữ liệu động và cho trạng thái được chia sẻ giữa các instance.

Cách trả lời tốt là gọi tên hai loại độ trễ khác nhau. Loại thứ nhất đến từ việc tính toán và đọc dữ liệu, và Redis giải quyết loại này. Loại thứ hai đến từ quãng đường vật lý mà gói tin phải đi, và chỉ có CDN mới giải quyết được. Nếu chúng ta gộp hai loại đó làm một, chúng ta sẽ chọn nhầm công cụ cho đúng vấn đề.

**English (bám cấu trúc tiếng Việt)**

A CDN solves a problem that Redis cannot solve, which is geographical distance. Redis sits inside our own data centre, therefore a user in Europe still has to wait for the packets to travel halfway around the world. A CDN places copies at points of presence near the user, therefore it cuts away exactly the latency caused by distance. The second benefit is that it takes load off the origin server, because most requests never reach that place. We put onto the CDN the static files such as images, JavaScript and CSS, together with the public GET responses. Redis, meanwhile, keeps its role as the cache for dynamic data and for state shared between instances.

The good way to answer is to name two different kinds of latency. The first kind comes from computing and reading data, and Redis solves this kind. The second kind comes from the physical distance the packets have to travel, and only a CDN can solve that. If we merge those two kinds into one, we will pick the wrong tool for the actual problem.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| khoảng cách địa lý | geographical distance |
| đi vòng qua nửa vòng trái đất | travel halfway around the world |
| các điểm hiện diện gần người dùng | points of presence near the user |
| cắt thẳng phần độ trễ do khoảng cách gây ra | cuts away exactly the latency caused by distance |
| giảm tải cho máy chủ gốc | takes load off the origin server |
| phần lớn request không bao giờ đi tới nơi đó | most requests never reach that place |
| trạng thái được chia sẻ giữa các instance | state shared between instances |
| gọi tên hai loại độ trễ khác nhau | name two different kinds of latency |
| quãng đường vật lý mà gói tin phải đi | the physical distance the packets have to travel |
| chọn nhầm công cụ cho đúng vấn đề | pick the wrong tool for the actual problem |

**Thuật ngữ cần nhớ**

- mạng phân phối nội dung → **a content delivery network (CDN)**
- điểm hiện diện → **a point of presence**
- máy chủ gốc → **the origin server**
- biên mạng → **the edge**
- dữ liệu động → **dynamic data**

---

## Phần 5 — Vì sao GraphQL trên POST khó cache

**Tiếng Việt**

Cache HTTP hoạt động dựa trên địa chỉ và phương thức của request. Phương thức GET được coi là an toàn và có thể cache, vì nó không làm thay đổi trạng thái ở máy chủ. GraphQL thì thường dùng một địa chỉ duy nhất với phương thức POST, và nội dung truy vấn nằm trong phần thân. Vì cache không nhìn vào phần thân, mọi truy vấn khác nhau đều trông giống hệt nhau đối với CDN. Hệ quả là chúng ta mất gần như toàn bộ lợi ích của cache HTTP. Hai cách lấy lại lợi ích đó là dùng cache đã chuẩn hoá ở phía client, hoặc dùng truy vấn được lưu sẵn rồi gửi qua GET.

Cách nói này cho thấy chúng ta hiểu vì sao, chứ không chỉ thuộc kết luận. Vấn đề không nằm ở GraphQL, mà nằm ở việc phần phân biệt request bị đẩy vào chỗ mà cache không đọc được. Nếu chúng ta đưa phần phân biệt đó trở lại địa chỉ, ví dụ bằng một mã băm của truy vấn, thì cache HTTP lại hoạt động bình thường. Đây cũng chính là ý tưởng đứng sau kỹ thuật truy vấn được lưu sẵn.

**English (bám cấu trúc tiếng Việt)**

HTTP caching works on the basis of the address and the method of the request. The GET method is considered safe and cacheable, because it does not change the state on the server. GraphQL, on the other hand, usually uses one single address with the POST method, and the query content sits in the body. Because the cache does not look into the body, all the different queries look exactly alike to the CDN. The consequence is that we lose almost all of the benefit of HTTP caching. Two ways to win that benefit back are using a normalised cache on the client side, or using persisted queries sent through GET.

This way of speaking shows that we understand why, instead of only having memorised the conclusion. The problem does not lie in GraphQL, but lies in the fact that the distinguishing part of the request has been pushed into a place the cache cannot read. If we bring that distinguishing part back into the address, for example through a hash of the query, then HTTP caching works normally again. This is also exactly the idea behind the persisted query technique.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| dựa trên địa chỉ và phương thức của request | on the basis of the address and the method of the request |
| được coi là an toàn và có thể cache | is considered safe and cacheable |
| nội dung truy vấn nằm trong phần thân | the query content sits in the body |
| trông giống hệt nhau đối với CDN | look exactly alike to the CDN |
| chúng ta mất gần như toàn bộ lợi ích | we lose almost all of the benefit |
| hai cách lấy lại lợi ích đó | two ways to win that benefit back |
| truy vấn được lưu sẵn | persisted queries |
| chứ không chỉ thuộc kết luận | instead of only having memorised the conclusion |
| phần phân biệt request | the distinguishing part of the request |
| bị đẩy vào chỗ mà cache không đọc được | has been pushed into a place the cache cannot read |
| đưa phần phân biệt đó trở lại địa chỉ | bring that distinguishing part back into the address |

**Thuật ngữ cần nhớ**

- có thể cache được → **cacheable**
- phương thức HTTP → **an HTTP method**
- cache đã chuẩn hoá → **a normalised cache**
- truy vấn được lưu sẵn → **a persisted query**
- mã băm của truy vấn → **a hash of the query**

---

## Phần 6 — Xoá cache trên CDN, địa chỉ có phiên bản, và header Vary

**Tiếng Việt**

Khi nội dung đổi, chúng ta có hai lựa chọn để cache không trả về bản cũ. Lựa chọn thứ nhất là gọi lệnh xoá trên CDN, nhưng việc xoá đó phải lan ra rất nhiều điểm hiện diện trên toàn cầu. Việc lan đó tốn thời gian và thường tốn cả tiền, vì vậy chúng ta không nên dựa vào nó cho mỗi lần triển khai. Lựa chọn thứ hai là đưa phiên bản vào chính địa chỉ, ví dụ nhúng mã băm của nội dung vào tên tệp. Khi nội dung đổi thì tên tệp đổi theo, vì vậy CDN buộc phải lấy bản mới mà chúng ta không cần xoá gì cả. Header Vary là một công cụ khác: nó nói cho cache biết rằng response còn phụ thuộc vào một số header của request. Đặt Vary quá rộng thì cache bị chia vụn và tỉ lệ trúng tụt xuống, còn thiếu Vary cho dữ liệu riêng thì chúng ta rò cache giữa những người dùng.

Vì vậy nguyên tắc của chúng ta là đặt Vary vừa đủ, chứ không đặt cho chắc. Với dữ liệu thay đổi theo kiểu nén, chúng ta khai báo theo đúng header tương ứng. Với dữ liệu thay đổi theo người dùng, cách an toàn hơn là đánh dấu private hoặc cấm lưu hẳn. Dựa vào Vary theo header xác thực là một cách làm mong manh, vì chỉ cần một tầng trung gian bỏ qua nó là chúng ta rò dữ liệu.

**English (bám cấu trúc tiếng Việt)**

When the content changes, we have two options so that the cache does not return the old copy. The first option is to call a purge on the CDN, but that purge has to spread out to a great many points of presence around the world. That spreading costs time and usually costs money as well, therefore we should not rely on it for every deployment. The second option is to put the version into the address itself, for example by embedding a hash of the content into the file name. When the content changes the file name changes with it, therefore the CDN is forced to fetch the new copy while we do not need to purge anything. The Vary header is another tool: it tells the cache that the response also depends on certain headers of the request. Setting Vary too broadly means the cache is fragmented and the hit ratio falls, while missing Vary on private data means we leak the cache between users.

Therefore our principle is to set Vary just enough, instead of setting it to be safe. For data that changes according to the compression type, we declare exactly the matching header. For data that changes according to the user, the safer way is to mark it private or to forbid storing it altogether. Relying on Vary with the authentication header is a fragile approach, because it only takes one intermediate layer ignoring it for us to leak data.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| gọi lệnh xoá trên CDN | call a purge on the CDN |
| phải lan ra rất nhiều điểm hiện diện | has to spread out to a great many points of presence |
| cho mỗi lần triển khai | for every deployment |
| nhúng mã băm của nội dung vào tên tệp | embedding a hash of the content into the file name |
| tên tệp đổi theo | the file name changes with it |
| mà chúng ta không cần xoá gì cả | while we do not need to purge anything |
| cache bị chia vụn | the cache is fragmented |
| chúng ta rò cache giữa những người dùng | we leak the cache between users |
| đặt Vary vừa đủ, chứ không đặt cho chắc | set Vary just enough, instead of setting it to be safe |
| một cách làm mong manh | a fragile approach |
| chỉ cần một tầng trung gian bỏ qua nó | it only takes one intermediate layer ignoring it |

**Thuật ngữ cần nhớ**

- xoá cache trên CDN → **to purge**
- lan ra khắp nơi → **to spread out** / **to propagate**
- địa chỉ có gắn phiên bản → **a versioned URL**
- bị chia vụn → **fragmented**
- mong manh, dễ vỡ → **fragile**

---

## Phần 7 — Cache response cá nhân hoá: bẫy nguy hiểm nhất

**Tiếng Việt**

Đây là bẫy nguy hiểm nhất của cả bài, vì hậu quả của nó là rò rỉ dữ liệu chứ không phải là chậm. Nếu một response chứa dữ liệu riêng của một người mà chúng ta lại đánh dấu là public, thì cache dùng chung sẽ lưu bản của người đó. Người tiếp theo gọi cùng địa chỉ sẽ nhận đúng bản sao đó, vì CDN không phân biệt hai người dùng qua header xác thực theo mặc định. Cách làm đúng có ba hướng và chúng ta nên nêu đủ cả ba. Hướng thứ nhất là đánh dấu private hoặc cấm lưu cho mọi thứ có tính cá nhân. Hướng thứ hai là tách phần công khai ra khỏi phần riêng, cache phần công khai và để phần riêng được dựng ở phía client hoặc được ghép ở biên.

Hướng thứ ba, nếu chúng ta thật sự cần cache cả phần riêng, là đưa định danh người dùng vào chính khoá cache. Cách này giống hệt nguyên tắc thiết kế key mà chúng ta đã học ở Bài 3 và đã nhắc lại ở Bài 7. Nói cách khác, cùng một lỗi xuất hiện ở ba tầng khác nhau, chỉ đổi tên gọi mà thôi. Ở tầng ứng dụng nó là key thiếu chiều phân biệt, còn ở tầng HTTP nó là response cá nhân hoá bị đánh dấu công khai.

**English (bám cấu trúc tiếng Việt)**

This is the most dangerous trap of the whole lesson, because its consequence is a data leak instead of slowness. If a response contains the private data of one person while we mark it as public, then the shared cache will store that person's copy. The next person calling the same address will receive exactly that copy, because a CDN does not distinguish two users through the authentication header by default. The correct approach has three directions and we should name all three. The first direction is to mark private or forbid storage for everything that is personal. The second direction is to separate the public part from the private part, cache the public part and let the private part be rendered on the client side or be assembled at the edge.

The third direction, if we truly need to cache the private part as well, is to put the user identifier into the cache key itself. This way is exactly the same as the key design principle we learned in Lesson 3 and repeated in Lesson 7. In other words, the same mistake appears at three different layers, merely changing its name. At the application layer it is a key missing a distinguishing dimension, while at the HTTP layer it is a personalised response marked as public.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| hậu quả của nó là rò rỉ dữ liệu chứ không phải là chậm | its consequence is a data leak instead of slowness |
| cache dùng chung sẽ lưu bản của người đó | the shared cache will store that person's copy |
| không phân biệt hai người dùng qua header xác thực | does not distinguish two users through the authentication header |
| theo mặc định | by default |
| mọi thứ có tính cá nhân | everything that is personal |
| tách phần công khai ra khỏi phần riêng | separate the public part from the private part |
| được dựng ở phía client hoặc được ghép ở biên | be rendered on the client side or be assembled at the edge |
| đưa định danh người dùng vào chính khoá cache | put the user identifier into the cache key itself |
| chỉ đổi tên gọi mà thôi | merely changing its name |
| key thiếu chiều phân biệt | a key missing a distinguishing dimension |
| response cá nhân hoá bị đánh dấu công khai | a personalised response marked as public |

**Thuật ngữ cần nhớ**

- response cá nhân hoá → **a personalised response**
- rò rỉ dữ liệu → **a data leak**
- header xác thực → **the authentication header**
- ghép ở biên → **assembled at the edge**
- định danh người dùng → **the user identifier**

---

## Phần 8 — Góc tech lead: khép lại cả giáo trình bằng năm cái bẫy

**Tiếng Việt**

Tình huống thách đố của bài cuối rất ngắn nhưng rất đắt. Một đội đặt public và max-age ba trăm giây cho địa chỉ trả về hồ sơ của người đang đăng nhập. Vài ngày sau, một số người dùng báo rằng họ nhìn thấy thông tin của người khác. Nguyên nhân là cache dùng chung lưu response của người dùng thứ nhất, rồi trả chính response đó cho người dùng thứ hai gọi cùng địa chỉ. Cách sửa là đánh dấu private và cấm lưu, hoặc tối thiểu là private kèm khai báo Vary theo header xác thực. Đây là lỗi thuộc nhóm đúng đắn và bảo mật, chứ không thuộc nhóm hiệu năng. Vì vậy nó là loại lỗi mà một tech lead bắt buộc phải bắt được ngay trong lúc review.

Khép lại giáo trình, chúng ta nên thuộc năm cái bẫy mà công cụ AI hay tạo ra. Bẫy thứ nhất là dùng write-back cho dữ liệu tiền, và hậu quả là mất giao dịch khi cache chết. Bẫy thứ hai là dùng chung một key cho mọi người dùng, và hậu quả là rò dữ liệu giữa các tài khoản. Bẫy thứ ba là cập nhật cache thay vì xoá, hoặc làm sai thứ tự giữa việc ghi DB và việc xoá cache, và hậu quả là dữ liệu cũ tồn tại bền vững. Bẫy thứ tư là dùng một Redis duy nhất cho cả cache, session và hàng đợi với chính sách đuổi mọi key. Bẫy thứ năm là đánh dấu public cho một response cá nhân hoá, và đó chính là bài học của chính phần này.

**English (bám cấu trúc tiếng Việt)**

The challenge scenario of the final lesson is very short but very expensive. A team set public and a max-age of three hundred seconds on the address that returns the profile of the logged-in person. A few days later, some users reported that they were seeing other people's information. The cause is that the shared cache stored the response of the first user, and then returned that very response to the second user calling the same address. The fix is to mark it private and forbid storage, or at minimum private together with a Vary declaration on the authentication header. This is a bug in the correctness and security group, instead of being in the performance group. Therefore it is the kind of bug that a tech lead is obliged to catch right during review.

To close the curriculum, we should learn by heart the five traps that AI tools tend to produce. The first trap is using write-back for money data, and the consequence is losing transactions when the cache dies. The second trap is sharing one key for every user, and the consequence is leaking data between accounts. The third trap is updating the cache instead of deleting it, or getting the order wrong between writing the database and deleting the cache, and the consequence is stale data that persists durably. The fourth trap is using one single Redis for the cache, the sessions and the queue with a policy that evicts all keys. The fifth trap is marking a personalised response as public, and that is exactly the lesson of this very part.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| rất ngắn nhưng rất đắt | very short but very expensive |
| hồ sơ của người đang đăng nhập | the profile of the logged-in person |
| họ nhìn thấy thông tin của người khác | they were seeing other people's information |
| trả chính response đó cho người dùng thứ hai | returned that very response to the second user |
| tối thiểu là private kèm khai báo Vary | at minimum private together with a Vary declaration |
| thuộc nhóm đúng đắn và bảo mật | in the correctness and security group |
| bắt buộc phải bắt được ngay trong lúc review | is obliged to catch right during review |
| khép lại giáo trình | to close the curriculum |
| chúng ta nên thuộc năm cái bẫy | we should learn by heart the five traps |
| làm sai thứ tự giữa việc ghi DB và việc xoá cache | getting the order wrong between writing the database and deleting the cache |
| dữ liệu cũ tồn tại bền vững | stale data that persists durably |
| bài học của chính phần này | the lesson of this very part |

**Thuật ngữ cần nhớ**

- người đang đăng nhập → **the logged-in person**
- khai báo → **a declaration**
- nhóm lỗi bảo mật → **the security group of bugs**
- khép lại, kết thúc → **to close** / **to wrap up**
- thuộc lòng → **to learn by heart**

---

## Mô hình ghi nhớ

**Tiếng Việt**

Cache HTTP nằm ở biên và nó phân biệt các bản sao theo địa chỉ cộng với phương thức, vì vậy tệp tĩnh nên là public kèm phiên bản trong tên, còn API công khai thì dùng s-maxage kèm ETag. Dữ liệu riêng của người dùng phải là private hoặc bị cấm lưu, và chúng ta luôn nhớ rằng no-cache nghĩa là giữ nhưng phải hỏi lại, khác hẳn no-store nghĩa là đừng giữ.

**English (bám cấu trúc tiếng Việt)**

HTTP caching sits at the edge and it distinguishes copies by the address plus the method, therefore a static file should be public with a version in its name, while a public API uses s-maxage with an ETag. Private user data has to be private or forbidden from storage, and we always remember that no-cache means keep it but ask again, which is completely different from no-store meaning do not keep it.

---

## Bảng thuật ngữ tổng hợp

| Tiếng Việt | Tiếng Anh | Ghi chú phát âm (Anh-Anh) |
|---|---|---|
| bộ nhớ đệm | cache | /kæʃ/ — đọc y hệt "cash" |
| chỉ thị điều khiển cache | a cache directive | di-**REK**-tiv — trọng âm âm tiết thứ hai |
| cache dùng chung | a shared cache | |
| ghi đè lên | to override | əʊ-və-**RAID** — trọng âm cuối |
| bất biến | immutable | i-**MYOO**-tə-bl — trọng âm âm tiết thứ hai |
| tệp tĩnh | a static file | **STAT**-ik |
| thời gian sống | time to live (TTL) | đọc rời chữ cái: "tee-tee-el" |
| cấm lưu | no-store | |
| lưu nhưng phải kiểm tra lại | no-cache | |
| dữ liệu nhạy cảm | sensitive data | **SEN**-si-tiv |
| quyền riêng tư | privacy | Anh-Anh **PRIV**-ə-si — khác Mỹ "PRAI-və-si" |
| cache trung gian | an intermediate cache | in-tə-**MEE**-di-ət |
| kiểm tra lại | to revalidate / revalidation | ree-**VAL**-i-deit |
| dấu vân tay nội dung | a content fingerprint | |
| phần thân response | the body | |
| băng thông | bandwidth | **BAND**-width — kết thúc bằng /dθ/ |
| thời điểm sửa đổi | the modification time | mod-i-fi-**KAY**-shən |
| vòng đi về trên mạng | a network round trip | |
| mạng phân phối nội dung | a content delivery network (CDN) | di-**LIV**-ə-ri |
| điểm hiện diện | a point of presence | **PREZ**-əns |
| máy chủ gốc | the origin server | **OR**-i-jin — chữ **g** đọc thành /dʒ/ |
| biên mạng | the edge | /edʒ/ — kết thúc bằng /dʒ/ |
| khoảng cách địa lý | geographical distance | jee-ə-**GRAF**-i-kəl |
| độ trễ | latency | **LAY**-tən-si |
| dữ liệu động | dynamic data | dai-**NAM**-ik |
| gói tin | a packet | **PAK**-it |
| có thể cache được | cacheable | **KASH**-ə-bl |
| phương thức HTTP | an HTTP method | **METH**-əd — âm /θ/ ở giữa |
| cache đã chuẩn hoá | a normalised cache | **NOR**-mə-laizd |
| truy vấn được lưu sẵn | a persisted query | query — **KWEER**-ri |
| mã băm của nội dung | a hash of the content | |
| xoá cache trên CDN | to purge | /pɜːdʒ/ — kết thúc bằng /dʒ/ |
| lan ra khắp nơi | to propagate | **PROP**-ə-geit |
| lần triển khai | a deployment | di-**PLOI**-mənt |
| địa chỉ có gắn phiên bản | a versioned URL | |
| bị chia vụn | fragmented | frag-**MEN**-tid |
| mong manh, dễ vỡ | fragile | Anh-Anh **FRAJ**-ail — đuôi /aɪl/ |
| tỉ lệ hit | hit ratio | ratio — **RAY**-shi-əʊ |
| response cá nhân hoá | a personalised response | **PER**-sə-nə-laizd |
| rò rỉ dữ liệu | a data leak | |
| header xác thực | the authentication header | aw-then-ti-**KAY**-shən |
| ghép ở biên | assembled at the edge | ə-**SEM**-bld |
| dựng giao diện | to render | |
| định danh người dùng | the user identifier | ai-**DEN**-ti-fai-ə |
| chiều phân biệt | a distinguishing dimension | dis-**TING**-gwish-ing |
| người đang đăng nhập | the logged-in person | |
| khai báo | a declaration | dek-lə-**RAY**-shən |
| tính đúng đắn | correctness | |
| giao dịch | a transaction | tran-**ZAK**-shən — âm giữa là /z/ |
| dữ liệu cũ | stale data | stale /steɪl/ — "xtêi-l" |
| chính sách đuổi mọi key | an evict-all-keys policy | eviction — i-**VIK**-shən |
| thuộc lòng | to learn by heart | |

---

## Bài nói

> **Cách luyện:** bật ghi âm, nói **60–90 giây** cho mỗi đề, **không nhìn tài liệu**. Mỗi bài nói phải dùng ít nhất **sáu thuật ngữ** trong bảng trên. Nghe lại một lần, đánh dấu chỗ bạn ngập ngừng, rồi nói lại đúng đề đó thêm một lần nữa.

1. **Explain to a junior** the difference between max-age, s-maxage, private and public, giving one example of content for each.

2. **A colleague set no-cache** on the payment page, believing that nothing would then be stored anywhere. Explain why you would push back and what they should set instead.

3. **Explain how an ETag** and revalidation work, and say when they beat simply setting a long max-age.

4. **Explain what a CDN solves** that Redis cannot, by naming two different kinds of latency.

5. **Someone on your team wants** to purge the CDN on every deploy so that new assets go live. Explain why you would push back and what you would do instead.

6. **Why is GraphQL over POST** harder to cache than a REST GET endpoint, and how would you get that caching back?

7. **Users report seeing each other's profile data**, and the endpoint is marked public with a max-age of three hundred seconds. Walk through the diagnosis, the fix, and then list the five caching traps you check in any review.
