# Bài 4 — Building blocks lõi
### Bản song ngữ: tiếng Anh bám cấu trúc tiếng Việt

> **Cách dùng:** đọc đoạn tiếng Việt trước → tự dịch trong đầu → so với bản tiếng Anh bên dưới → **đọc to bản tiếng Anh hai lần**. Lần một đọc chậm cho đúng từ, lần hai đọc liền mạch như đang nói với người phỏng vấn. Bài này là bộ "hộp lego", nên mục tiêu khi nói là **gọi đúng tên** một kỹ thuật trong hai giây, chứ không phải mô tả vòng vo.

---

## Phần 1 — Mạng phân phối nội dung

**Tiếng Việt**

Mạng phân phối nội dung là một lớp cache đặt gần người dùng, và nó giữ bản sao của nội dung tĩnh ở nhiều điểm biên trên thế giới. Nhờ đó, chúng ta giảm được độ trễ cho người dùng ở xa và giảm được tải cho máy chủ gốc cùng một lúc. Chúng ta đặt lên mạng phân phối nội dung những thứ bất biến như ảnh, video, tệp JavaScript và tệp CSS. Chúng ta không đặt lên đó dữ liệu cá nhân, dữ liệu động hoặc dữ liệu nhạy cảm, vì bản sao có thể bị phục vụ nhầm cho người khác. Phần khó của mạng phân phối nội dung không nằm ở lúc đặt vào, mà nằm ở lúc lấy ra.

Chúng ta phải quyết định thời gian sống cho mỗi loại nội dung, và phải có cách xoá bản cũ khi nội dung thay đổi. Nếu thời gian sống quá dài mà không có cách xoá, người dùng sẽ nhìn thấy phiên bản cũ trong nhiều giờ sau khi chúng ta đã phát hành bản mới. Cách làm phổ biến và an toàn là gắn số phiên bản vào đường dẫn của tệp, ví dụ chúng ta đổi tên từ app.js thành app.v3.js. Khi đường dẫn đổi thì bản cũ không còn được ai gọi tới nữa, nên chúng ta không cần xoá bản cũ một cách vội vàng. Nếu chúng ta bỏ qua chuyện này, hậu quả là mỗi lần phát hành đều kéo theo một đợt báo lỗi từ người dùng đang giữ tệp cũ trong trình duyệt.

**English (bám cấu trúc tiếng Việt)**

A content delivery network is a cache layer placed close to the users, and it keeps copies of static content at many edge locations around the world. Thanks to that, we reduce the latency for far-away users and reduce the load on the origin server at the same time. We put on the content delivery network the things that are immutable, such as images, videos, JavaScript files and CSS files. We do not put personal data, dynamic data or sensitive data there, because a copy could be served to the wrong person. The hard part of a content delivery network does not lie in putting things in, it lies in taking things out.

We have to decide the time to live for each type of content, and we have to have a way to remove the old copy when the content changes. If the time to live is too long and there is no way to remove it, users will see the old version for many hours after we have already shipped the new one. A common and safe practice is to put a version number into the file path, for example we rename app.js into app.v3.js. When the path changes, nobody calls the old copy any more, so we do not need to remove the old copy in a hurry. If we skip this, the consequence is that every release brings a wave of bug reports from users who are still holding the old file in their browser.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| một lớp cache đặt gần người dùng | a cache layer placed close to the users |
| những thứ bất biến | the things that are immutable |
| bản sao có thể bị phục vụ nhầm cho người khác | a copy could be served to the wrong person |
| không nằm ở lúc đặt vào, mà nằm ở lúc lấy ra | does not lie in putting things in, it lies in taking things out |
| thời gian sống | the time to live |
| cách xoá bản cũ | a way to remove the old copy |
| gắn số phiên bản vào đường dẫn của tệp | put a version number into the file path |
| không còn được ai gọi tới nữa | nobody calls it any more |
| xoá bản cũ một cách vội vàng | remove the old copy in a hurry |
| kéo theo một đợt báo lỗi | brings a wave of bug reports |

**Thuật ngữ cần nhớ**

- mạng phân phối nội dung → **a content delivery network (CDN)**
- điểm biên → **an edge location**
- nội dung tĩnh → **static content**
- thời gian sống → **time to live (TTL)**
- vô hiệu hoá cache → **cache invalidation**
- bất biến → **immutable**

---

## Phần 2 — Kho đối tượng so với cơ sở dữ liệu

**Tiếng Việt**

Kho đối tượng là nơi để chứa tệp lớn, vì nó rẻ, bền và mở rộng gần như không giới hạn. Cơ sở dữ liệu thì không phải là nơi để chứa tệp lớn, mà chỉ nên giữ siêu dữ liệu cùng với đường dẫn trỏ tới tệp. Nói cách khác, chúng ta không nhồi ảnh và video vào một cột nhị phân trong cơ sở dữ liệu. Nếu chúng ta làm vậy, cơ sở dữ liệu sẽ phình rất nhanh, chi phí tăng theo, và mọi thao tác sao lưu hay di chuyển đều trở nên chậm chạp. Đây là một trong những lỗi tốn kém nhất mà một đội nhỏ có thể mắc trong năm đầu tiên.

Chúng ta phục vụ tệp cho người dùng qua mạng phân phối nội dung, hoặc qua một đường dẫn có chữ ký với thời hạn ngắn. Đường dẫn có chữ ký cho phép trình duyệt tải thẳng từ kho đối tượng, nên tệp không phải đi vòng qua máy chủ ứng dụng của chúng ta. Nhờ đó, máy chủ ứng dụng không phải gánh băng thông của tệp lớn, và nó tập trung vào việc xử lý logic nghiệp vụ. Nếu chúng ta cho tệp đi vòng qua ứng dụng, hậu quả là một vài lượt tải video có thể chiếm hết số luồng xử lý của tiến trình. Khi đó, các request nhẹ và quan trọng phải xếp hàng chờ phía sau một lượt tải tệp nặng, và cả hệ thống trông như đang chết dù CPU vẫn rảnh.

**English (bám cấu trúc tiếng Việt)**

Object storage is the place to hold large files, because it is cheap, durable and scales almost without limit. A database, on the other hand, is not the place to hold large files, it should only keep the metadata together with the path that points to the file. In other words, we do not stuff images and videos into a binary column in the database. If we do that, the database will grow very fast, the cost will grow with it, and every backup or migration operation will become slow. This is one of the most expensive mistakes a small team can make in its first year.

We serve files to users through a content delivery network, or through a signed URL with a short expiry. A signed URL lets the browser download straight from the object storage, so the file does not have to go around through our application server. Thanks to that, the application server does not have to carry the bandwidth of large files, and it focuses on handling the business logic. If we let files go around through the application, the consequence is that a few video downloads can take up all the worker threads of the process. At that point, light and important requests have to queue up behind one heavy file download, and the whole system looks as if it is dying even though the CPU is still idle.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| nơi để chứa tệp lớn | the place to hold large files |
| mở rộng gần như không giới hạn | scales almost without limit |
| đường dẫn trỏ tới tệp | the path that points to the file |
| nhồi ảnh và video vào một cột nhị phân | stuff images and videos into a binary column |
| cơ sở dữ liệu sẽ phình rất nhanh | the database will grow very fast |
| một đường dẫn có chữ ký với thời hạn ngắn | a signed URL with a short expiry |
| không phải đi vòng qua máy chủ ứng dụng | does not have to go around through our application server |
| chiếm hết số luồng xử lý | take up all the worker threads |
| phải xếp hàng chờ phía sau | have to queue up behind |
| trông như đang chết dù CPU vẫn rảnh | looks as if it is dying even though the CPU is still idle |

**Thuật ngữ cần nhớ**

- kho đối tượng → **object storage**
- siêu dữ liệu → **metadata**
- dữ liệu nhị phân lớn → **a blob**
- đường dẫn có chữ ký → **a presigned URL** / **a signed URL**
- luồng xử lý → **a worker thread**
- bền vững → **durable**

---

## Phần 3 — Băm nhất quán và nút ảo

**Tiếng Việt**

Khi chúng ta có nhiều node cache hoặc nhiều mảnh dữ liệu, chúng ta cần một luật để quyết định khoá nào thuộc về node nào. Cách ngây thơ là lấy giá trị băm chia lấy dư cho số node, nhưng cách này hỏng ngay khi số node thay đổi. Nếu chúng ta đổi số node từ ba lên bốn, thì gần như mọi khoá đều nhảy sang một node khác. Hậu quả là toàn bộ cache trở nên vô dụng cùng một lúc, và cơ sở dữ liệu phía sau nhận một cơn bão truy vấn ngay lập tức. Đây chính là kiểu sự cố xảy ra đúng vào lúc chúng ta thêm máy để cứu hệ thống đang quá tải.

Băm nhất quán giải quyết vấn đề này bằng cách xếp cả node lẫn khoá lên một vòng băm. Mỗi khoá thuộc về node đầu tiên nằm sau nó theo chiều kim đồng hồ trên vòng đó. Khi chúng ta thêm hoặc bớt một node, chỉ phần khoá nằm giữa node mới và node kề trước phải chuyển chỗ, còn phần lớn khoá vẫn giữ nguyên node cũ. Với bốn node thì chúng ta chỉ phải chuyển khoảng một phần tư số khoá, thay vì gần như toàn bộ như cách chia lấy dư. Đây là nền tảng của các cụm cache và cụm phân mảnh hiện đại, ví dụ Redis Cluster, Cassandra và cách phân vùng của DynamoDB.

Một vòng băm với mỗi node chỉ có một điểm thì vẫn phân phối không đều, vì các điểm rơi ngẫu nhiên và tạo ra những khoảng dài ngắn khác nhau. Vì vậy chúng ta rải mỗi node vật lý thành nhiều điểm trên vòng, và mỗi điểm đó gọi là một nút ảo. Càng nhiều nút ảo thì các khoảng càng đều, nên tải chia đều hơn và chúng ta tránh được node nóng. Ngoài ra, khi một node chết, phần khoá của nó được chia lại cho nhiều node khác thay vì dồn hết sang một node kề bên. Nhờ đó việc cân bằng lại diễn ra mượt hơn và không tạo ra một điểm nóng mới ngay sau sự cố.

**English (bám cấu trúc tiếng Việt)**

When we have many cache nodes or many data shards, we need a rule to decide which key belongs to which node. The naive way is to take the hash value modulo the number of nodes, but this way breaks as soon as the number of nodes changes. If we change the number of nodes from three to four, then almost every key jumps to a different node. The consequence is that the whole cache becomes useless at the same moment, and the database behind it takes a storm of queries immediately. This is exactly the kind of incident that happens right when we are adding machines to save an overloaded system.

Consistent hashing solves this problem by placing both the nodes and the keys on a hash ring. Each key belongs to the first node that sits after it clockwise on that ring. When we add or remove one node, only the keys lying between the new node and the previous neighbouring node have to move, while most of the keys stay on their old node. With four nodes we only have to move about a quarter of the keys, instead of almost all of them as with the modulo way. This is the foundation of modern cache clusters and shard clusters, for example Redis Cluster, Cassandra and the partitioning scheme of DynamoDB.

A hash ring where each node has only one point still distributes unevenly, because the points fall at random and create gaps of different lengths. Therefore we spread each physical node into many points on the ring, and each of those points is called a virtual node. The more virtual nodes there are, the more even the gaps become, so the load is spread more evenly and we avoid a hot node. In addition, when one node dies, its share of the keys is split among many other nodes instead of piling onto a single neighbour. Thanks to that, the rebalancing happens more smoothly and it does not create a new hot spot right after the incident.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| khoá nào thuộc về node nào | which key belongs to which node |
| lấy giá trị băm chia lấy dư cho số node | take the hash value modulo the number of nodes |
| hỏng ngay khi số node thay đổi | breaks as soon as the number of nodes changes |
| gần như mọi khoá đều nhảy sang một node khác | almost every key jumps to a different node |
| nhận một cơn bão truy vấn | takes a storm of queries |
| nằm sau nó theo chiều kim đồng hồ | sits after it clockwise |
| phải chuyển chỗ | have to move |
| các điểm rơi ngẫu nhiên | the points fall at random |
| tạo ra những khoảng dài ngắn khác nhau | create gaps of different lengths |
| dồn hết sang một node kề bên | piling onto a single neighbour |
| không tạo ra một điểm nóng mới | does not create a new hot spot |

**Thuật ngữ cần nhớ**

- băm nhất quán → **consistent hashing**
- vòng băm → **the hash ring**
- chia lấy dư → **modulo**
- nút ảo → **a virtual node**
- node nóng / điểm nóng → **a hot node** / **a hot spot**
- phân mảnh → **a shard** / **partitioning**

---

## Phần 4 — Bộ lọc Bloom

**Tiếng Việt**

Bộ lọc Bloom là một cấu trúc dữ liệu xác suất, và nó chỉ trả lời hai câu: có thể có, hoặc chắc chắn không. Câu trả lời chắc chắn không thì luôn đúng, còn câu trả lời có thể có thì đôi khi sai. Nói theo thuật ngữ, bộ lọc Bloom có dương tính giả nhưng không có âm tính giả. Đổi lại cho sự thiếu chắc chắn đó, nó tốn rất ít bộ nhớ so với việc giữ toàn bộ tập khoá. Vì vậy chúng ta dùng nó như một lớp lọc đặt phía trước, để tránh những lần đi xuống cơ sở dữ liệu một cách vô ích.

Ứng dụng phổ biến nhất là chống hiện tượng xuyên cache, tức là khi ai đó liên tục hỏi những khoá không tồn tại. Những khoá đó không nằm trong cache, nên mọi lần hỏi đều đập thẳng xuống cơ sở dữ liệu và làm nó kiệt sức. Với một bộ lọc Bloom ở phía trước, chúng ta chặn được phần lớn các khoá không tồn tại ngay tại tầng cache. Các ứng dụng khác cũng theo cùng một dạng, ví dụ kiểm tra nhanh xem một tên đăng nhập đã có người dùng chưa, hoặc kiểm tra xem một đường dẫn đã được thu thập chưa.

Có một cái bẫy mà nhiều người rơi vào khi mới dùng bộ lọc Bloom. Bộ lọc nói rằng tên đăng nhập john có thể đã tồn tại, và chúng ta từ chối đăng ký ngay lập tức. Cách xử lý đó sai, vì có thể có không có nghĩa là chắc chắn có. Khi bộ lọc nói có thể có, chúng ta phải đi xác nhận lại ở cơ sở dữ liệu trước khi ra quyết định. Chúng ta chỉ được tin tuyệt đối vào một câu trả lời duy nhất, đó là câu chắc chắn không.

**English (bám cấu trúc tiếng Việt)**

A Bloom filter is a probabilistic data structure, and it only gives two answers: possibly present, or definitely not present. The answer "definitely not present" is always correct, while the answer "possibly present" is sometimes wrong. In technical terms, a Bloom filter has false positives but it has no false negatives. In exchange for that lack of certainty, it uses very little memory compared with keeping the whole key set. Therefore we use it as a filter layer placed in front, in order to avoid pointless trips down to the database.

The most common application is to prevent cache penetration, that is, when someone repeatedly asks for keys that do not exist. Those keys are not in the cache, so every question goes straight down to the database and wears it out. With a Bloom filter in front, we block most of the non-existent keys right at the cache layer. Other applications follow the same shape, for example checking quickly whether a username is already taken, or checking whether a URL has already been crawled.

There is a trap that many people fall into when they first use a Bloom filter. The filter says that the username john may already exist, and we reject the registration immediately. That handling is wrong, because "possibly present" does not mean "definitely present". When the filter says "possibly present", we have to go and confirm again in the database before we make a decision. We are only allowed to trust one single answer absolutely, and that is the answer "definitely not present".

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| một cấu trúc dữ liệu xác suất | a probabilistic data structure |
| có thể có, hoặc chắc chắn không | possibly present, or definitely not present |
| đổi lại cho sự thiếu chắc chắn đó | in exchange for that lack of certainty |
| một lớp lọc đặt phía trước | a filter layer placed in front |
| những lần đi xuống cơ sở dữ liệu một cách vô ích | pointless trips down to the database |
| đập thẳng xuống cơ sở dữ liệu | goes straight down to the database |
| làm nó kiệt sức | wears it out |
| một tên đăng nhập đã có người dùng chưa | whether a username is already taken |
| đã được thu thập chưa | has already been crawled |
| chúng ta phải đi xác nhận lại | we have to go and confirm again |
| chỉ được tin tuyệt đối vào | only allowed to trust absolutely |

**Thuật ngữ cần nhớ**

- bộ lọc Bloom → **a Bloom filter**
- dương tính giả → **a false positive**
- âm tính giả → **a false negative**
- xuyên cache → **cache penetration**
- xác suất → **probabilistic**
- khử trùng lặp → **deduplication**

---

## Phần 5 — Chỉ mục địa lý cho truy vấn lân cận

**Tiếng Việt**

Khi bài toán yêu cầu tìm những điểm gần một vị trí, chúng ta không được quét toàn bảng rồi tính khoảng cách cho từng điểm. Cách quét toàn bảng có chi phí tuyến tính theo số bản ghi, nên nó chết ngay khi dữ liệu lên tới hàng triệu điểm. Thay vào đó, chúng ta biến toạ độ hai chiều thành một khoá một chiều có thể đánh chỉ mục, bằng geohash, cây tứ phân hoặc thư viện S2. Ý tưởng chung là chia bản đồ thành các ô lưới, rồi gán cho mỗi ô một mã chuỗi. Hai điểm gần nhau trên bản đồ thì có phần đầu của mã giống nhau, nên chúng ta tìm theo tiền tố thay vì tính khoảng cách.

Khi truy vấn, chúng ta lấy ô chứa vị trí người dùng cùng với các ô kề bên, rồi chỉ tính khoảng cách trong phạm vi các ô đó. Bước lấy thêm ô kề bên là bắt buộc, vì người dùng có thể đứng sát mép của một ô. Nếu chúng ta quên các ô kề bên, hậu quả là hệ thống bỏ sót đúng những tài xế hoặc những cửa hàng gần nhất, và người dùng thấy kết quả vô lý. Chúng ta cũng phải chọn kích thước ô cho hợp với mật độ, vì ô quá lớn thì mỗi truy vấn đọc quá nhiều điểm còn ô quá nhỏ thì phải đọc quá nhiều ô. Đây là loại đánh đổi mà người phỏng vấn rất thích nghe chúng ta nói ra thành lời.

**English (bám cấu trúc tiếng Việt)**

When the problem asks us to find the points near a location, we must not scan the whole table and then compute the distance for each point. A full table scan has a cost that is linear in the number of records, so it dies as soon as the data reaches millions of points. Instead, we turn the two-dimensional coordinates into a one-dimensional key that can be indexed, using geohash, a quad-tree or the S2 library. The general idea is to divide the map into grid cells, and then give each cell a string code. Two points that are near each other on the map share the same beginning of the code, so we search by prefix instead of computing distances.

When we query, we take the cell containing the user's position together with the neighbouring cells, and then we only compute distances within those cells. The step of taking the neighbouring cells is compulsory, because the user may be standing right at the edge of a cell. If we forget the neighbouring cells, the consequence is that the system misses exactly the nearest drivers or the nearest shops, and the user sees a result that makes no sense. We also have to choose the cell size to match the density, because cells that are too large make each query read too many points while cells that are too small make us read too many cells. This is the kind of trade-off that interviewers really like to hear us put into words.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| quét toàn bảng | scan the whole table / a full table scan |
| chi phí tuyến tính theo số bản ghi | a cost that is linear in the number of records |
| biến toạ độ hai chiều thành một khoá một chiều | turn the two-dimensional coordinates into a one-dimensional key |
| chia bản đồ thành các ô lưới | divide the map into grid cells |
| có phần đầu của mã giống nhau | share the same beginning of the code |
| tìm theo tiền tố | search by prefix |
| các ô kề bên | the neighbouring cells |
| đứng sát mép của một ô | standing right at the edge of a cell |
| hệ thống bỏ sót đúng những tài xế gần nhất | the system misses exactly the nearest drivers |
| chọn kích thước ô cho hợp với mật độ | choose the cell size to match the density |

**Thuật ngữ cần nhớ**

- chỉ mục địa lý → **a geo index**
- truy vấn lân cận → **a nearby query**
- cây tứ phân → **a quad-tree**
- ô lưới → **a grid cell**
- tiền tố → **a prefix**
- toạ độ → **coordinates**

---

## Phần 6 — Hàng đợi thông điệp

**Tiếng Việt**

Chúng ta thêm hàng đợi thông điệp khi muốn tách rời bên gửi khỏi bên xử lý. Bên gửi chỉ cần đẩy một thông điệp vào hàng đợi rồi trả lời người dùng ngay, còn bên xử lý lấy thông điệp ra và làm việc nặng ở phía sau. Hàng đợi còn đóng vai trò bộ đệm, nên nó hấp thụ được những đợt tải dồn đột ngột thay vì để chúng đập thẳng vào dịch vụ. Ngoài ra, hàng đợi cho chúng ta cơ chế thử lại một cách tự nhiên khi một lần xử lý thất bại. Những dấu hiệu cho thấy nên thêm hàng đợi là các tác vụ nặng và chậm mà người dùng không cần chờ, ví dụ gửi thư điện tử, chuyển mã video, hoặc phát tán bài viết tới người theo dõi.

Hàng đợi không phải là món quà miễn phí, và chúng ta phải nói được cái giá của nó. Thứ nhất, nó thêm độ trễ, vì công việc không còn xong ngay trong request nữa. Thứ hai, nó đưa hệ thống về trạng thái nhất quán cuối cùng, nên người dùng có thể thấy dữ liệu chưa cập nhật trong vài giây. Thứ ba, nó thêm một hệ thống nữa phải vận hành, phải theo dõi độ trễ tồn đọng và phải xử lý những thông điệp chết. Nếu chúng ta thêm hàng đợi mà không chuẩn bị cho ba điều đó, hậu quả là chúng ta đổi một vấn đề hiệu năng lấy một vấn đề vận hành khó thấy hơn nhiều.

**English (bám cấu trúc tiếng Việt)**

We add a message queue when we want to decouple the sender from the processor. The sender only needs to push a message into the queue and answer the user right away, while the processor takes the message out and does the heavy work at the back. The queue also acts as a buffer, so it absorbs sudden bursts of load instead of letting them hit the service directly. In addition, the queue gives us a retry mechanism in a natural way when one attempt at processing fails. The signals that we should add a queue are heavy and slow tasks that the user does not need to wait for, for example sending emails, transcoding videos, or fanning out a post to the followers.

A queue is not a free gift, and we must be able to say its price. First, it adds latency, because the work no longer finishes inside the request. Second, it moves the system to eventual consistency, so a user may see data that is not updated yet for a few seconds. Third, it adds one more system to operate, to monitor for backlog lag and to handle dead messages. If we add a queue without preparing for those three things, the consequence is that we trade a performance problem for an operational problem that is much harder to see.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| tách rời bên gửi khỏi bên xử lý | decouple the sender from the processor |
| trả lời người dùng ngay | answer the user right away |
| làm việc nặng ở phía sau | does the heavy work at the back |
| đóng vai trò bộ đệm | acts as a buffer |
| hấp thụ được những đợt tải dồn đột ngột | absorbs sudden bursts of load |
| đập thẳng vào dịch vụ | hit the service directly |
| phát tán bài viết tới người theo dõi | fanning out a post to the followers |
| không phải là món quà miễn phí | is not a free gift |
| theo dõi độ trễ tồn đọng | monitor for backlog lag |
| những thông điệp chết | dead messages |
| đổi một vấn đề hiệu năng lấy một vấn đề vận hành | trade a performance problem for an operational problem |

**Thuật ngữ cần nhớ**

- hàng đợi thông điệp → **a message queue**
- tách rời → **to decouple**
- bộ đệm → **a buffer**
- đợt tải dồn → **a burst**
- tồn đọng → **the backlog**
- hàng đợi thư chết → **a dead-letter queue**

---

## Phần 7 — Tìm kiếm toàn văn

**Tiếng Việt**

Khi yêu cầu là tìm kiếm trong văn bản, chúng ta không dùng câu lệnh LIKE với dấu phần trăm ở hai đầu. Kiểu truy vấn đó không dùng được chỉ mục, nên cơ sở dữ liệu phải quét tuyến tính qua mọi bản ghi. Thay vào đó, chúng ta thêm một hệ tìm kiếm dựa trên chỉ mục ngược, ví dụ Elasticsearch hoặc OpenSearch. Chỉ mục ngược lưu ánh xạ từ mỗi từ tới danh sách tài liệu chứa từ đó, nên việc tìm trở thành tra bảng thay vì quét bảng. Ngoài tốc độ, hệ tìm kiếm còn cho chúng ta xếp hạng theo độ liên quan, thứ mà cơ sở dữ liệu quan hệ không làm tốt.

Cái giá của lựa chọn này nằm ở việc đồng bộ dữ liệu và ở chi phí vận hành. Chúng ta phải đưa dữ liệu từ cơ sở dữ liệu nguồn sang chỉ mục tìm kiếm, và hai bên phải không lệch nhau quá lâu. Cách an toàn là dùng một luồng thu thập thay đổi từ nhật ký giao dịch, hoặc dùng bảng hộp thư đi trong cùng giao dịch ghi. Chúng ta nên tránh cách ghi kép bằng tay, tức là ứng dụng tự ghi vào cả hai nơi, vì một trong hai lần ghi có thể thất bại. Nếu chúng ta chọn ghi kép, hậu quả là dữ liệu lệch dần theo thời gian, và không ai biết chính xác từ lúc nào kết quả tìm kiếm bắt đầu sai.

**English (bám cấu trúc tiếng Việt)**

When the requirement is to search inside text, we do not use a LIKE statement with a percent sign at both ends. That kind of query cannot use an index, so the database has to scan linearly through every record. Instead, we add a search system based on an inverted index, for example Elasticsearch or OpenSearch. An inverted index stores a mapping from each word to the list of documents containing that word, so searching becomes a table lookup rather than a table scan. Besides the speed, a search system also gives us ranking by relevance, which a relational database does not do well.

The price of this choice lies in synchronising the data and in the operational cost. We have to move the data from the source database into the search index, and the two sides must not drift apart for too long. The safe way is to use a change-data-capture stream from the transaction log, or to use an outbox table inside the same write transaction. We should avoid the manual dual-write approach, that is, the application writing into both places itself, because one of the two writes may fail. If we choose dual writes, the consequence is that the data drifts gradually over time, and nobody knows exactly from which moment the search results started to be wrong.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| với dấu phần trăm ở hai đầu | with a percent sign at both ends |
| không dùng được chỉ mục | cannot use an index |
| quét tuyến tính qua mọi bản ghi | scan linearly through every record |
| lưu ánh xạ từ mỗi từ tới danh sách tài liệu | stores a mapping from each word to the list of documents |
| tra bảng thay vì quét bảng | a table lookup rather than a table scan |
| xếp hạng theo độ liên quan | ranking by relevance |
| hai bên phải không lệch nhau quá lâu | the two sides must not drift apart for too long |
| một luồng thu thập thay đổi từ nhật ký giao dịch | a change-data-capture stream from the transaction log |
| bảng hộp thư đi trong cùng giao dịch ghi | an outbox table inside the same write transaction |
| cách ghi kép bằng tay | the manual dual-write approach |
| dữ liệu lệch dần theo thời gian | the data drifts gradually over time |

**Thuật ngữ cần nhớ**

- tìm kiếm toàn văn → **full-text search**
- chỉ mục ngược → **an inverted index**
- độ liên quan → **relevance**
- thu thập thay đổi dữ liệu → **change data capture (CDC)**
- mẫu hộp thư đi → **the outbox pattern**
- ghi kép → **a dual write**

---

## Phần 8 — Cache đặt ở nhiều tầng

**Tiếng Việt**

Cache không nằm ở một chỗ duy nhất, mà nằm rải trên một chuỗi tầng từ người dùng vào tới cơ sở dữ liệu. Chuỗi đó gồm cache của trình duyệt, mạng phân phối nội dung ở biên, cổng API, cache cục bộ trong tiến trình ứng dụng, cache phân tán như Redis, và cuối cùng là cache bên trong chính cơ sở dữ liệu. Mỗi tầng cache giảm tải cho tầng nằm dưới nó, nên hiệu quả cộng dồn rất lớn. Nhưng mỗi tầng cũng thêm một bản sao của sự thật, và mỗi bản sao là một chỗ có thể bị cũ. Vì vậy, số tầng cache mà chúng ta thêm vào phải tương xứng với khả năng chúng ta kiểm soát tính cũ của dữ liệu.

Chúng ta chọn tầng dựa trên cách dữ liệu được đọc, chứ không cache mọi thứ cho nhanh. Dữ liệu dùng chung cho mọi người và ít thay đổi thì hợp với mạng phân phối nội dung ở biên. Dữ liệu riêng của từng người mà nóng thì hợp với một cache phân tán như Redis. Dữ liệu cực nóng và dùng cục bộ thì hợp với cache trong tiến trình, với điều kiện chúng ta chấp nhận một khoảng cũ rất ngắn. Nếu chúng ta cache bừa mà không có chiến lược làm mới, hậu quả là chúng ta tạo ra những lỗi dữ liệu cũ, và loại lỗi đó khó chịu hơn nhiều so với việc hệ thống chạy chậm.

**English (bám cấu trúc tiếng Việt)**

A cache does not sit in one single place, it is spread across a chain of layers from the user down to the database. That chain includes the browser cache, the content delivery network at the edge, the API gateway, the local cache inside the application process, a distributed cache such as Redis, and finally the cache inside the database itself. Each cache layer reduces the load on the layer below it, so the effect adds up to a lot. But each layer also adds one more copy of the truth, and each copy is a place that can go stale. Therefore, the number of cache layers we add has to match our ability to control the staleness of the data.

We choose the layer based on how the data is read, we do not cache everything just to make it fast. Data that is shared by everyone and changes rarely fits the content delivery network at the edge. Data that belongs to each person and is hot fits a distributed cache such as Redis. Data that is extremely hot and used locally fits an in-process cache, on the condition that we accept a very short window of staleness. If we cache carelessly without a refresh strategy, the consequence is that we create stale-data bugs, and that kind of bug is far more annoying than a system that runs slowly.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| nằm rải trên một chuỗi tầng | is spread across a chain of layers |
| giảm tải cho tầng nằm dưới nó | reduces the load on the layer below it |
| hiệu quả cộng dồn rất lớn | the effect adds up to a lot |
| thêm một bản sao của sự thật | adds one more copy of the truth |
| một chỗ có thể bị cũ | a place that can go stale |
| phải tương xứng với khả năng chúng ta kiểm soát | has to match our ability to control |
| dựa trên cách dữ liệu được đọc | based on how the data is read |
| với điều kiện chúng ta chấp nhận | on the condition that we accept |
| nếu chúng ta cache bừa | if we cache carelessly |
| khó chịu hơn nhiều so với việc hệ thống chạy chậm | far more annoying than a system that runs slowly |

**Thuật ngữ cần nhớ**

- cache nhiều tầng → **multi-layer caching**
- cache trong tiến trình → **an in-process cache**
- cache phân tán → **a distributed cache**
- cổng API → **an API gateway**
- tính cũ của dữ liệu → **staleness**
- kiểu đọc → **the read pattern**

---

## Mô hình ghi nhớ

**Tiếng Việt**

Chúng ta đừng phát minh lại hộp lego. Tệp lớn thì đưa vào kho đối tượng cộng với mạng phân phối nội dung, chia khoá thì dùng băm nhất quán, hỏi tồn tại thì dùng bộ lọc Bloom, tìm điểm gần thì dùng geohash, tìm văn bản thì dùng chỉ mục ngược, và việc nặng thì đẩy vào hàng đợi.

**English (bám cấu trúc tiếng Việt)**

We should not reinvent the lego bricks. Large files go into object storage plus a content delivery network, splitting keys uses consistent hashing, asking about existence uses a Bloom filter, finding nearby points uses geohash, searching text uses an inverted index, and heavy work goes into a queue.

---

## Bảng thuật ngữ tổng hợp

| Tiếng Việt | Tiếng Anh | Ghi chú phát âm |
|---|---|---|
| mạng phân phối nội dung | a content delivery network (CDN) | |
| điểm biên | an edge location | |
| nội dung tĩnh | static content | |
| thời gian sống | time to live (TTL) | |
| vô hiệu hoá cache | cache invalidation | *cache* /kæʃ/ — đọc đúng như "cash", không đọc "ca-chê" |
| bất biến | immutable | i-**MIU**-tơ-bợl, trọng âm âm thứ hai |
| kho đối tượng | object storage | |
| siêu dữ liệu | metadata | Anh thường đọc "**ME**-tơ-đây-tơ", trọng âm đầu |
| dữ liệu nhị phân lớn | a blob | |
| đường dẫn có chữ ký | a presigned URL | *signed* /saɪnd/ — chữ **g câm**, đọc "sain-d" |
| luồng xử lý | a worker thread | *thread* /θred/ — âm **th** đầu, không đọc thành "tred" |
| bền vững | durable | Anh /ˈdjʊərəbl/ — "**DIU**-ơ-rơ-bợl", trọng âm đầu |
| băm nhất quán | consistent hashing | *consistent* — cần-**SIS**-tần, trọng âm âm thứ hai |
| vòng băm | the hash ring | |
| chia lấy dư | modulo | "**MO**-điu-lâu", trọng âm đầu |
| nút ảo | a virtual node | *virtual* — "**VƠ**-chu-ợl", trọng âm đầu |
| node nóng | a hot node / a hot spot | |
| phân mảnh | a shard / partitioning | *shard* /ʃɑːd/ — âm đầu là "sh" không phải "s" |
| cân bằng lại | to rebalance | |
| bộ lọc Bloom | a Bloom filter | |
| dương tính giả | a false positive | |
| âm tính giả | a false negative | |
| xuyên cache | cache penetration | *penetration* — pe-nơ-**TRÂY**-shợn, trọng âm âm thứ ba |
| xác suất | probabilistic | pro-bơ-bi-**LIS**-tịc, trọng âm âm thứ tư |
| khử trùng lặp | deduplication | |
| chỉ mục địa lý | a geo index | *geo-* đọc "JI-âu", không đọc "gê-ô" |
| truy vấn lân cận | a nearby query | *query* /ˈkwɪəri/ — "KUY-ơ-ri" |
| cây tứ phân | a quad-tree | *quad* /kwɒd/ — "quod" |
| ô lưới | a grid cell | |
| tiền tố | a prefix | "**PRI**-fix", trọng âm đầu |
| toạ độ | coordinates | câu-**O**-đi-nợts, trọng âm âm thứ hai; âm cuối **-ts** phải bật ra |
| quét toàn bảng | a full table scan | |
| hàng đợi thông điệp | a message queue | *queue* /kjuː/ — đọc như chữ "Q", bốn chữ cái cuối câm |
| tách rời | to decouple | đi-**CA**-pợl, trọng âm âm thứ hai |
| bộ đệm | a buffer | |
| đợt tải dồn | a burst | âm cuối **-st** phải bật ra |
| phát tán | fan-out | |
| chuyển mã | to transcode | |
| tồn đọng | the backlog | |
| hàng đợi thư chết | a dead-letter queue | *dead* /ded/ — đọc ngắn, không kéo dài thành "đi-ét" |
| nhất quán cuối cùng | eventual consistency | |
| tìm kiếm toàn văn | full-text search | *search* /sɜːtʃ/ — âm cuối **-ch**, không thành "sớt" |
| chỉ mục ngược | an inverted index | |
| độ liên quan | relevance | "**RE**-lơ-vợns", trọng âm đầu |
| thu thập thay đổi dữ liệu | change data capture (CDC) | |
| mẫu hộp thư đi | the outbox pattern | |
| ghi kép | a dual write | *dual* /ˈdjuːəl/ — "DIU-ợl", nghe gần giống *jewel* |
| nhật ký giao dịch | the transaction log | |
| cache nhiều tầng | multi-layer caching | |
| cache trong tiến trình | an in-process cache | |
| cache phân tán | a distributed cache | |
| cổng API | an API gateway | |
| tính cũ của dữ liệu | staleness | *stale* /steɪl/ — "stêi-l", có âm **st** đầu rõ |
| kiểu đọc | the read pattern | |
| thông lượng | throughput | âm **th** /θ/ đầu, "THRU-put" |

---

## Bài nói

> **Cách luyện:** bật ghi âm, nói **60–90 giây** cho mỗi đề, **không nhìn tài liệu** khi nói. Mỗi bài phải dùng ít nhất **sáu thuật ngữ** trong bảng trên. Nghe lại bản ghi và đánh dấu chỗ nào bạn phải dừng lại tìm từ — chính chỗ đó là cụm cần học lại từ bảng ánh xạ.

1. **Explain to a junior engineer** why `hash % N` is a bad way to spread keys across cache nodes, and what happens to the database the moment you add one node.

2. **A colleague says:** *"The Bloom filter says this username exists, so we can reject the registration straight away."* **Explain what is wrong with that**, and say which of the two answers you are allowed to trust.

3. **Someone on your team wants to** store user-uploaded videos in a binary column in PostgreSQL because it keeps everything in one place. **Explain why you would push back**, and describe the architecture you would propose instead.

4. **Describe what happens when** a search feature is built with `LIKE '%keyword%'` and the table grows to ten million rows. Say what you would add, and what new problem that addition creates.

5. **Describe what happens when** a nearby-driver query forgets to include the neighbouring grid cells, from the point of view of a user standing at the edge of a cell.

6. **When would you choose** an in-process cache over a distributed cache like Redis, and when would you choose the CDN instead? Use the read pattern to justify each choice.

7. **Explain to a product manager** why adding a message queue makes the upload feel instant but also means the result may not appear straight away.
