# Bài 3 — Quyết định data modeling thực chiến
### Bản song ngữ: tiếng Anh bám cấu trúc tiếng Việt

> **Cách dùng bài này:** đọc đoạn tiếng Việt trước, tự dịch trong đầu, rồi mới so với đoạn tiếng Anh bên dưới. Chỗ nào lệch thì xem bảng ánh xạ cụm khó. Cuối cùng, **đọc to đoạn tiếng Anh hai lần** — một lần chậm để đúng âm, một lần với tốc độ nói thật.

---

## Phần 1 — Kiểu dữ liệu là một hợp đồng vĩnh viễn

**Tiếng Việt**

Có một nhóm quyết định trong thiết kế dữ liệu mà chúng ta đưa ra rất nhanh ở tuần đầu tiên, nhưng chúng ta phải sống với nó trong nhiều năm. Nhóm đó gồm kiểu của khoá chính, cách chúng ta xoá một bản ghi, cách chúng ta lưu tiền và cách chúng ta lưu thời gian. Những quyết định này âm thầm gây hậu quả lớn, vì hệ thống vẫn chạy bình thường trong sáu tháng đầu và chỉ lộ vấn đề khi dữ liệu đã lớn. Chúng ta có thể coi kiểu dữ liệu là một hợp đồng vĩnh viễn giữa chúng ta và hệ thống. Nếu chúng ta ký sai hợp đồng đó ngay từ đầu, chi phí sửa về sau sẽ rất cao, vì chúng ta phải đổi kiểu của một cột trên bảng đang có hàng trăm triệu dòng. Vì vậy, người phỏng vấn hay hỏi đúng nhóm câu này, bởi vì câu trả lời của chúng ta để lộ độ chín của kinh nghiệm.

**English (bám cấu trúc tiếng Việt)**

There is a group of decisions in data design that we make very quickly in the first week, but that we have to live with for many years. That group includes the type of the primary key, the way we delete a record, the way we store money and the way we store time. These decisions quietly cause big consequences, because the system still runs normally for the first six months and only shows the problem when the data has grown large. We can treat the data type as a permanent contract between us and the system. If we sign that contract wrongly from the start, the cost of fixing it later will be very high, because we have to change the type of a column on a table that already holds hundreds of millions of rows. Therefore, an interviewer often asks exactly this group of questions, because our answer reveals the maturity of our experience.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| chúng ta phải sống với nó trong nhiều năm | we have to live with it for many years |
| âm thầm gây hậu quả lớn | quietly cause big consequences |
| chỉ lộ vấn đề khi dữ liệu đã lớn | only shows the problem when the data has grown large |
| một hợp đồng vĩnh viễn | a permanent contract |
| nếu chúng ta ký sai hợp đồng đó ngay từ đầu | if we sign that contract wrongly from the start |
| chi phí sửa về sau | the cost of fixing it later |
| bảng đang có hàng trăm triệu dòng | a table that already holds hundreds of millions of rows |
| người phỏng vấn hay hỏi đúng nhóm câu này | an interviewer often asks exactly this group of questions |
| để lộ độ chín của kinh nghiệm | reveals the maturity of our experience |

**Thuật ngữ cần nhớ**

- kiểu dữ liệu → **data type**
- khoá chính → **primary key**
- bản ghi → **record / row**
- hậu quả → **consequence**
- độ chín nghề nghiệp → **maturity**

---

## Phần 2 — Hệ giao dịch và hệ phân tích không được ở chung một chỗ

**Tiếng Việt**

Hệ giao dịch trực tuyến phục vụ rất nhiều giao dịch nhỏ, và thứ nó cần là độ trễ thấp trên từng câu lệnh. Vì mục tiêu đó, chúng ta chuẩn hoá bảng và chúng ta giữ cho mỗi câu truy vấn chỉ chạm vào một lượng dòng nhỏ. Hệ phân tích thì ngược lại, vì nó chạy các phép tổng hợp lớn và thứ nó cần là thông lượng, chứ không phải độ trễ. Vì mục tiêu đó, hệ phân tích thường denormalize và thường lưu dữ liệu theo cột thay vì theo dòng. Sai lầm phổ biến nhất là chúng ta chạy một báo cáo nặng thẳng trên cơ sở dữ liệu giao dịch chính. Khi đó, một câu truy vấn quét toàn bảng sẽ chiếm hết bộ nhớ đệm, và mọi người dùng thật sẽ thấy trang web chậm hẳn đi trong vài phút. Cách làm đúng là chúng ta đẩy dữ liệu sang một kho phân tích riêng, hoặc ít nhất chúng ta chạy báo cáo trên một bản sao đọc.

**English (bám cấu trúc tiếng Việt)**

An online transactional system serves a very large number of small transactions, and what it needs is low latency on each statement. For that goal, we normalise the tables and we keep every query touching only a small number of rows. An analytical system is the opposite, because it runs large aggregations and what it needs is throughput, not latency. For that goal, an analytical system usually denormalizes and usually stores the data by column instead of by row. The most common mistake is that we run a heavy report directly on the main transactional database. At that point, one query that scans the whole table will take over the entire cache, and all the real users will see the website slow down badly for a few minutes. The correct approach is that we push the data into a separate analytical warehouse, or at least we run the report on a read replica.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| thứ nó cần là độ trễ thấp trên từng câu lệnh | what it needs is low latency on each statement |
| giữ cho mỗi câu truy vấn chỉ chạm vào một lượng dòng nhỏ | keep every query touching only a small number of rows |
| hệ phân tích thì ngược lại | an analytical system is the opposite |
| lưu dữ liệu theo cột thay vì theo dòng | store the data by column instead of by row |
| chạy một báo cáo nặng thẳng trên | run a heavy report directly on |
| chiếm hết bộ nhớ đệm | take over the entire cache |
| trang web chậm hẳn đi trong vài phút | the website slows down badly for a few minutes |
| đẩy dữ liệu sang một kho phân tích riêng | push the data into a separate analytical warehouse |
| hoặc ít nhất | or at least |

**Thuật ngữ cần nhớ**

- độ trễ thấp → **low latency**
- thông lượng → **throughput**
- lưu theo cột → **columnar storage**
- quét toàn bảng → **full table scan**
- bản sao đọc → **read replica**

---

## Phần 3 — Khoá chính tuần tự: gọn, nhanh, nhưng dễ đoán

**Tiếng Việt**

Lựa chọn kinh điển cho khoá chính là một số nguyên tự tăng, ví dụ kiểu bigserial hoặc cột identity trong PostgreSQL. Ưu điểm thứ nhất là nó gọn, vì mỗi khoá chỉ chiếm tám byte. Ưu điểm thứ hai quan trọng hơn, vì các giá trị tăng dần nên mỗi bản ghi mới luôn được chèn vào cuối cây B-tree. Nhờ đó, index ít bị phân mảnh, và cơ sở dữ liệu chỉ phải giữ nóng vài trang cuối trong bộ nhớ. Nhược điểm thứ nhất là khoá này để lộ thứ tự, vì bất kỳ ai nhìn thấy id bằng một nghìn cũng đoán được rằng chúng ta có khoảng một nghìn bản ghi. Nhược điểm thứ hai là khi chúng ta gộp dữ liệu từ nhiều nguồn, các dãy số sẽ đụng nhau và chúng ta phải đánh số lại. Vì vậy, số tự tăng là lựa chọn tốt cho một hệ thống có một nguồn ghi duy nhất, nhưng nó bắt đầu vướng khi hệ thống trở nên phân tán.

**English (bám cấu trúc tiếng Việt)**

The classic choice for a primary key is an auto-increment integer, for example the bigserial type or an identity column in PostgreSQL. The first advantage is that it is compact, because each key takes only eight bytes. The second advantage is more important, because the values increase steadily, so every new record is always inserted at the end of the B-tree. Thanks to that, the index suffers little fragmentation, and the database only has to keep the last few pages hot in memory. The first drawback is that this key reveals the order, because anyone who sees an id equal to one thousand can guess that we have about one thousand records. The second drawback is that when we merge data from many sources, the number ranges will collide and we will have to renumber everything. Therefore, an auto-increment number is a good choice for a system that has a single write source, but it starts to get in the way when the system becomes distributed.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| một số nguyên tự tăng | an auto-increment integer |
| nó gọn | it is compact |
| các giá trị tăng dần | the values increase steadily |
| được chèn vào cuối cây B-tree | is inserted at the end of the B-tree |
| index ít bị phân mảnh | the index suffers little fragmentation |
| giữ nóng vài trang cuối trong bộ nhớ | keep the last few pages hot in memory |
| khoá này để lộ thứ tự | this key reveals the order |
| cũng đoán được rằng | can guess that |
| các dãy số sẽ đụng nhau | the number ranges will collide |
| chúng ta phải đánh số lại | we will have to renumber everything |
| nó bắt đầu vướng | it starts to get in the way |

**Thuật ngữ cần nhớ**

- số tự tăng → **auto-increment**
- phân mảnh index → **index fragmentation**
- trang dữ liệu → **page**
- một nguồn ghi duy nhất → **a single write source**
- hệ phân tán → **a distributed system**

---

## Phần 4 — Vì sao UUID phiên bản bốn làm hỏng cây B-tree

**Tiếng Việt**

Nhiều người chọn UUID phiên bản bốn làm khoá chính vì họ nghĩ rằng như vậy là chắc chắn hơn. UUID phiên bản bốn là một giá trị ngẫu nhiên hoàn toàn, và chính tính ngẫu nhiên đó gây ra vấn đề. Vì các giá trị mới rơi rải rác khắp không gian khoá, mỗi lần chèn sẽ chạm vào một trang khác nhau của cây B-tree. Khi một trang đã đầy mà chúng ta vẫn phải chèn vào giữa, cơ sở dữ liệu buộc phải tách trang đó ra làm đôi. Hậu quả là index bị phân mảnh, index phình to hơn nhiều so với mức cần thiết, và tốc độ chèn giảm dần theo thời gian. Ngoài ra, một khoá UUID chiếm mười sáu byte, tức là gấp đôi một số nguyên tám byte, và cái giá này bị nhân lên ở mọi index phụ trỏ tới khoá chính. Nếu chúng ta chọn UUID phiên bản bốn cho một bảng ghi nặng, chúng ta sẽ thấy hiệu năng chèn xuống dốc đúng vào lúc hệ thống bắt đầu thành công.

**English (bám cấu trúc tiếng Việt)**

Many people choose UUID version four as the primary key because they think that this way is safer. UUID version four is a completely random value, and that randomness itself causes the problem. Because the new values fall scattered all over the key space, each insert will touch a different page of the B-tree. When a page is already full and we still have to insert into the middle, the database is forced to split that page into two. The consequence is that the index becomes fragmented, the index grows far bigger than necessary, and the insert speed drops gradually over time. In addition, a UUID key takes sixteen bytes, which is twice an eight-byte integer, and this cost is multiplied across every secondary index that points to the primary key. If we choose UUID version four for a write-heavy table, we will see the insert performance go downhill exactly at the moment when the system starts to succeed.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| vì họ nghĩ rằng như vậy là chắc chắn hơn | because they think that this way is safer |
| chính tính ngẫu nhiên đó gây ra vấn đề | that randomness itself causes the problem |
| rơi rải rác khắp không gian khoá | fall scattered all over the key space |
| chạm vào một trang khác nhau | touch a different page |
| cơ sở dữ liệu buộc phải tách trang đó ra làm đôi | the database is forced to split that page into two |
| phình to hơn nhiều so với mức cần thiết | grows far bigger than necessary |
| tốc độ chèn giảm dần theo thời gian | the insert speed drops gradually over time |
| cái giá này bị nhân lên ở mọi index phụ | this cost is multiplied across every secondary index |
| một bảng ghi nặng | a write-heavy table |
| hiệu năng chèn xuống dốc | the insert performance goes downhill |
| đúng vào lúc hệ thống bắt đầu thành công | exactly at the moment when the system starts to succeed |

**Thuật ngữ cần nhớ**

- giá trị ngẫu nhiên → **a random value**
- không gian khoá → **key space**
- tách trang → **page split**
- index phụ → **secondary index**
- bảng ghi nặng → **write-heavy table**

---

## Phần 5 — UUID phiên bản bảy và ULID: cách dung hoà

**Tiếng Việt**

May mắn là chúng ta không còn phải chọn giữa hai cực đó nữa, vì đã có UUID phiên bản bảy và ULID. Hai định dạng này đặt phần dấu thời gian ở đầu khoá, còn phần ngẫu nhiên thì nằm ở phía sau. Nhờ vậy, các khoá mới sinh ra vẫn tăng dần theo thời gian, và chúng vẫn được chèn vào cuối cây B-tree. Đồng thời, chúng ta vẫn giữ được hai lợi ích của UUID, đó là sinh khoá ngay ở phía ứng dụng mà không cần hỏi cơ sở dữ liệu, và người ngoài không đoán được số lượng bản ghi. PostgreSQL 18 đã hỗ trợ hàm uuidv7 ngay trong cơ sở dữ liệu, nhưng chúng ta nên kiểm tra lại tài liệu chính thức cho đúng phiên bản mà mình đang dùng. Vì vậy, quy tắc thực dụng là chúng ta dùng số tự tăng cho hệ thống một nguồn ghi, và chúng ta dùng UUID phiên bản bảy hoặc ULID cho hệ thống nhiều nguồn ghi.

**English (bám cấu trúc tiếng Việt)**

Luckily we no longer have to choose between those two extremes, because UUID version seven and ULID now exist. These two formats put the timestamp part at the front of the key, while the random part sits at the back. Thanks to that, the newly generated keys still increase over time, and they are still inserted at the end of the B-tree. At the same time, we still keep the two benefits of a UUID, namely generating the key right on the application side without asking the database, and outsiders cannot guess the number of records. PostgreSQL 18 supports the uuidv7 function inside the database itself, but we should check the official documentation again for the exact version that we are using. Therefore, the pragmatic rule is that we use an auto-increment number for a system with a single write source, and we use UUID version seven or ULID for a system with many write sources.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| chúng ta không còn phải chọn giữa hai cực đó nữa | we no longer have to choose between those two extremes |
| đặt phần dấu thời gian ở đầu khoá | put the timestamp part at the front of the key |
| phần ngẫu nhiên thì nằm ở phía sau | the random part sits at the back |
| các khoá mới sinh ra | the newly generated keys |
| đó là | namely |
| sinh khoá ngay ở phía ứng dụng | generating the key right on the application side |
| mà không cần hỏi cơ sở dữ liệu | without asking the database |
| người ngoài không đoán được | outsiders cannot guess |
| kiểm tra lại tài liệu chính thức | check the official documentation again |
| quy tắc thực dụng là | the pragmatic rule is |

**Thuật ngữ cần nhớ**

- sắp theo thời gian → **time-ordered**
- dấu thời gian → **timestamp**
- sinh ở phía ứng dụng → **generated on the application side**
- không cần đi vòng tới cơ sở dữ liệu → **without a database round trip**
- tài liệu chính thức → **the official documentation**

---

## Phần 6 — Xoá mềm và ba cái bẫy của nó

**Tiếng Việt**

Xoá mềm nghĩa là chúng ta không xoá dòng dữ liệu thật, mà chúng ta chỉ đánh dấu nó bằng một cột deleted_at. Cách này giữ lại lịch sử, và nó cho phép chúng ta khôi phục dữ liệu khi người dùng xoá nhầm. Nhưng nó kéo theo ba cái bẫy mà chúng ta phải nói ra trong buổi phỏng vấn. Bẫy thứ nhất là bảng phình lên mãi mãi, vì dữ liệu chết vẫn nằm chung với dữ liệu sống và vẫn chiếm chỗ trong mọi index. Bẫy thứ hai là mọi câu truy vấn đều phải thêm điều kiện lọc deleted_at is null, và chỉ cần một chỗ quên là người dùng sẽ nhìn thấy dữ liệu đã xoá. Bẫy thứ ba là ràng buộc duy nhất bị vướng, vì một email đã xoá vẫn chiếm chỗ và vẫn chặn người dùng đăng ký lại bằng chính email đó. Cách xử lý bẫy thứ ba là chúng ta tạo một unique index bộ phận, tức là ràng buộc duy nhất chỉ áp dụng cho những dòng còn sống.

Chúng ta cũng nên cân nhắc một lựa chọn khác, đó là chuyển dữ liệu chết sang một bảng lưu trữ riêng. Cách này giữ cho bảng chính gọn nhẹ, và nó vẫn giữ lịch sử cho việc kiểm toán. Trong ngành tài chính, người ta gần như không bao giờ xoá cứng, và họ dùng nhật ký kiểm toán để ghi lại mọi thay đổi. Điểm quan trọng là chúng ta chọn xoá mềm một cách có chủ đích cho từng bảng, chứ chúng ta không bật nó mặc định cho mọi bảng.

**English (bám cấu trúc tiếng Việt)**

Soft delete means that we do not delete the real data row, but we only mark it with a deleted_at column. This approach keeps the history, and it lets us restore the data when a user deletes something by mistake. But it drags along three traps that we have to name out loud in an interview. The first trap is that the table grows forever, because the dead data still sits together with the live data and still takes space in every index. The second trap is that every query has to add the filter condition deleted_at is null, and one forgotten place is enough for a user to see deleted data. The third trap is that the unique constraint gets in the way, because a deleted email still occupies the slot and still blocks the user from signing up again with that same email. The way to handle the third trap is that we create a partial unique index, that is a unique constraint which applies only to the live rows.

We should also consider another option, namely moving the dead data into a separate archive table. This approach keeps the main table small and light, and it still keeps the history for auditing. In the finance industry, people almost never hard delete, and they use an audit log to record every change. The important point is that we choose soft delete deliberately for each table, and we do not turn it on by default for every table.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| chúng ta chỉ đánh dấu nó bằng một cột | we only mark it with a column |
| khôi phục dữ liệu khi người dùng xoá nhầm | restore the data when a user deletes something by mistake |
| nó kéo theo ba cái bẫy | it drags along three traps |
| mà chúng ta phải nói ra trong buổi phỏng vấn | that we have to name out loud in an interview |
| bảng phình lên mãi mãi | the table grows forever |
| dữ liệu chết vẫn nằm chung với dữ liệu sống | the dead data still sits together with the live data |
| chỉ cần một chỗ quên là | one forgotten place is enough for |
| vẫn chiếm chỗ | still occupies the slot |
| chặn người dùng đăng ký lại | blocks the user from signing up again |
| chỉ áp dụng cho những dòng còn sống | applies only to the live rows |
| chuyển dữ liệu chết sang một bảng lưu trữ riêng | moving the dead data into a separate archive table |
| giữ cho bảng chính gọn nhẹ | keeps the main table small and light |
| chúng ta không bật nó mặc định cho mọi bảng | we do not turn it on by default for every table |

**Thuật ngữ cần nhớ**

- xoá mềm / xoá cứng → **soft delete / hard delete**
- ràng buộc duy nhất → **unique constraint**
- index duy nhất bộ phận → **partial unique index**
- bảng lưu trữ → **archive table**
- nhật ký kiểm toán → **audit log**

---

## Phần 7 — Lưu tiền và lưu thời gian cho đúng

**Tiếng Việt**

Chúng ta không bao giờ được lưu tiền bằng kiểu số thực dấu phẩy động. Lý do là số thực nhị phân không biểu diễn chính xác được những giá trị thập phân đơn giản, ví dụ số không phẩy một. Sai số này rất nhỏ ở một phép tính, nhưng nó tích luỹ qua hàng triệu giao dịch và cuối cùng bảng cân đối sẽ lệch vài xu. Cách đúng là chúng ta dùng kiểu numeric, hoặc chúng ta lưu tiền dưới dạng số nguyên theo đơn vị nhỏ nhất, ví dụ theo xu. Với thời gian, chúng ta nên dùng timestamptz thay vì timestamp, vì timestamptz lưu giá trị theo giờ quốc tế và biết chuyển đổi khi đọc ra. Nếu chúng ta dùng timestamp không có múi giờ, một sự kiện được ghi ở Việt Nam và được đọc ở Anh sẽ hiển thị lệch bảy tiếng, và không ai biết con số nào mới đúng. Cả hai lỗi này rất khó sửa về sau, vì lúc đó chúng ta không còn biết những giá trị cũ đã được ghi theo quy ước nào.

**English (bám cấu trúc tiếng Việt)**

We must never store money with a floating-point type. The reason is that a binary floating-point number cannot represent simple decimal values exactly, for example the number zero point one. This error is very small in one calculation, but it accumulates across millions of transactions and in the end the balance sheet will be off by a few cents. The correct way is that we use the numeric type, or we store money as an integer in the smallest unit, for example in cents. For time, we should use timestamptz instead of timestamp, because timestamptz stores the value in universal time and knows how to convert it on the way out. If we use timestamp without a time zone, an event that is written in Vietnam and is read in Britain will show up seven hours off, and nobody will know which number is the right one. Both of these mistakes are very hard to fix later, because by then we no longer know which convention the old values were written under.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| kiểu số thực dấu phẩy động | a floating-point type |
| không biểu diễn chính xác được | cannot represent exactly |
| sai số này rất nhỏ ở một phép tính | this error is very small in one calculation |
| nó tích luỹ qua hàng triệu giao dịch | it accumulates across millions of transactions |
| bảng cân đối sẽ lệch vài xu | the balance sheet will be off by a few cents |
| theo đơn vị nhỏ nhất | in the smallest unit |
| lưu giá trị theo giờ quốc tế | stores the value in universal time |
| biết chuyển đổi khi đọc ra | knows how to convert it on the way out |
| sẽ hiển thị lệch bảy tiếng | will show up seven hours off |
| không ai biết con số nào mới đúng | nobody will know which number is the right one |
| đã được ghi theo quy ước nào | which convention they were written under |

**Thuật ngữ cần nhớ**

- số thực dấu phẩy động → **floating-point number**
- sai số làm tròn → **rounding error**
- đơn vị nhỏ nhất của tiền tệ → **minor unit**
- múi giờ → **time zone**
- giờ quốc tế → **UTC / universal time**

---

## Phần 8 — Review một migration trước khi duyệt

**Tiếng Việt**

Ngày nay, một công cụ sinh mã có thể tạo ra một migration chạy được chỉ trong vài giây. Nhưng "chạy được" không có nghĩa là "đúng kiến trúc", và đây chính là chỗ mà một tech lead phải can thiệp. Công cụ thường chọn hàm gen_random_uuid, tức là UUID phiên bản bốn, thường chọn kiểu text cho mọi thứ, và thường chọn timestamp không có múi giờ. Vì vậy, chúng ta nên có một danh sách kiểm tra năm điểm trước khi chúng ta duyệt một migration. Chúng ta hỏi lần lượt như sau: index có khớp với access pattern thật không, kiểu dữ liệu cho tiền và khoá và thời gian có đúng không, các ràng buộc nullable và khoá ngoại và check đã đủ chưa, migration có khoá bảng quá lâu không, và chúng ta có quay lui được không. Điểm thứ tư là điểm đáng sợ nhất, vì một câu lệnh thêm cột kèm giá trị mặc định trên bảng năm trăm triệu dòng có thể buộc cơ sở dữ liệu viết lại toàn bộ bảng. Cách an toàn là chúng ta thêm một cột cho phép null trước, rồi chúng ta lấp dữ liệu theo từng lô nhỏ, và cuối cùng chúng ta mới thêm ràng buộc.

**English (bám cấu trúc tiếng Việt)**

Nowadays, a code generation tool can produce a migration that runs in just a few seconds. But "it runs" does not mean "it is architecturally correct", and this is exactly the place where a tech lead has to step in. The tool usually picks the gen_random_uuid function, which is UUID version four, usually picks the text type for everything, and usually picks timestamp without a time zone. Therefore, we should have a five-point checklist before we approve a migration. We ask in turn as follows: whether the indexes match the real access pattern, whether the data types for money and keys and time are correct, whether the nullable and foreign key and check constraints are complete, whether the migration locks the table for too long, and whether we can roll back. The fourth point is the scariest one, because a statement that adds a column with a default value on a table of five hundred million rows can force the database to rewrite the whole table. The safe approach is that we add a nullable column first, then we backfill the data in small batches, and only at the end do we add the constraint.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| một công cụ sinh mã | a code generation tool |
| "chạy được" không có nghĩa là "đúng kiến trúc" | "it runs" does not mean "it is architecturally correct" |
| đây chính là chỗ mà … phải can thiệp | this is exactly the place where … has to step in |
| một danh sách kiểm tra năm điểm | a five-point checklist |
| chúng ta hỏi lần lượt như sau | we ask in turn as follows |
| có khớp với access pattern thật không | whether the indexes match the real access pattern |
| các ràng buộc … đã đủ chưa | whether the … constraints are complete |
| có khoá bảng quá lâu không | whether it locks the table for too long |
| chúng ta có quay lui được không | whether we can roll back |
| là điểm đáng sợ nhất | is the scariest one |
| buộc cơ sở dữ liệu viết lại toàn bộ bảng | force the database to rewrite the whole table |
| lấp dữ liệu theo từng lô nhỏ | backfill the data in small batches |

**Thuật ngữ cần nhớ**

- chuyển đổi lược đồ → **migration**
- danh sách kiểm tra → **checklist**
- khoá bảng → **lock the table**
- quay lui → **roll back / rollback**
- lấp dữ liệu → **backfill**
- theo từng lô → **in batches**

---

## Mô hình ghi nhớ

**Tiếng Việt**

Kiểu dữ liệu là một hợp đồng vĩnh viễn, vì vậy chúng ta lưu tiền bằng số nguyên, chúng ta lưu thời gian bằng timestamptz, và chúng ta chọn khoá chính tuần tự hoặc sắp theo thời gian. Nếu chúng ta chọn sai ngay từ đầu, việc sửa về sau sẽ rất đắt.

**English (bám cấu trúc tiếng Việt)**

The data type is a permanent contract, therefore we store money as an integer, we store time as timestamptz, and we choose a primary key that is sequential or time-ordered. If we choose wrongly from the start, fixing it later will be very expensive.

---

## Bảng thuật ngữ tổng hợp

| Tiếng Việt | Tiếng Anh | Ghi chú phát âm |
|---|---|---|
| kiểu dữ liệu | data type | Anh /ˈdeɪtə/ — "**DAY**-tơ" |
| hợp đồng vĩnh viễn | a permanent contract | **PER**-mơ-nơnt · danh từ **CON**-tract (động từ con-**TRACT**) |
| độ chín nghề nghiệp | maturity | mơ-**TIU**-rơ-ti — trọng âm 2 |
| hệ giao dịch trực tuyến | online transactional system (OLTP) | tran-**SAC**-tion-al |
| hệ phân tích | analytical system (OLAP) | a-nơ-**LY**-ti-cal |
| độ trễ thấp | low latency | *latency* /ˈleɪtənsi/ — "**LAY**-tơn-si" |
| thông lượng | throughput | /ˈθruːpʊt/ — âm **th** đầu lưỡi, không thành "trút" |
| lưu theo cột | columnar storage | cơ-**LUM**-nơ; *column* có **n** cuối câm |
| quét toàn bảng | full table scan | *scan* — đuôi /-n/ đừng nuốt |
| bản sao đọc | read replica | **REP**-li-cơ — trọng âm 1 |
| khoá chính | primary key | Anh /ˈpraɪməri/ — "**PRAI**-mơ-ri" |
| số tự tăng | auto-increment | **IN**-cri-mơnt — trọng âm 1 |
| số nguyên | integer | **IN**-tơ-jơ — chữ **g** đọc /dʒ/, không đọc "in-tơ-gơ" |
| tuần tự | sequential | si-**KWEN**-shơl — "qu" đọc /kw/ |
| cây B-tree | B-tree | đọc là "bee-tree" |
| trang dữ liệu | page | |
| tách trang | page split | *split* — cụm /spl-/ đọc liền |
| phân mảnh index | index fragmentation | frag-mơn-**TAY**-shun |
| index phụ | secondary index | **SE**-cơn-đơ-ri |
| bảng ghi nặng | write-heavy table | *write* — chữ **w** câm |
| giá trị ngẫu nhiên | a random value | |
| không gian khoá | key space | |
| sắp theo thời gian | time-ordered | |
| dấu thời gian | timestamp | |
| định danh duy nhất toàn cục | UUID | đọc từng chữ: "yu-yu-ai-**DEE**" |
| không cần đi vòng tới DB | without a database round trip | |
| hệ phân tán | distributed system | dis-**TRI**-biu-tid |
| xoá mềm / xoá cứng | soft delete / hard delete | |
| ràng buộc | constraint | /kənˈstreɪnt/ — cụm /str/ đọc liền |
| ràng buộc duy nhất | unique constraint | *unique* trọng âm 2: yu-**NEEK** |
| index duy nhất bộ phận | partial unique index | *partial* /ˈpɑːʃl/ — "ti" đọc /ʃ/ |
| bảng lưu trữ | archive table | /ˈɑːkaɪv/ — "**AR**-kaiv", không đọc "ạc-chiv" |
| nhật ký kiểm toán | audit log | *audit* /ˈɔːdɪt/ — "**AW**-đit" |
| khôi phục | restore | ri-**STOR** — trọng âm 2 |
| số thực dấu phẩy động | floating-point number | |
| sai số làm tròn | rounding error | |
| độ chính xác | precision | pri-**SI**-zhơn — "si" cuối đọc /ʒ/ |
| kiểu số chính xác | numeric | niu-**ME**-rik — trọng âm 2 |
| đơn vị nhỏ nhất của tiền tệ | minor unit | |
| bảng cân đối | the balance sheet | |
| múi giờ | time zone | |
| giờ quốc tế | UTC / universal time | yu-ni-**VER**-sal |
| chuyển đổi lược đồ | migration | my-**GRAY**-shun — âm đầu là "my" |
| danh sách kiểm tra | checklist | |
| khoá bảng | lock the table | |
| quay lui | roll back / rollback | |
| lấp dữ liệu | backfill | |
| theo từng lô | in batches | *batches* — đuôi /-ɪz/ rõ |
| duyệt / phê duyệt | approve | ơ-**PROOV** — trọng âm 2 |
| rủi ro | risk | đuôi /-sk/ phải bật ra, không thành "rít" |

---

## Bài nói

> **Cách luyện:** bật ghi âm, nói **60–90 giây** cho mỗi đề, **không nhìn tài liệu**. Mỗi bài nói phải dùng **ít nhất 6 thuật ngữ** trong bảng trên. Nghe lại bản ghi một lần, đánh dấu chỗ ngập ngừng, rồi nói lại đề đó lần thứ hai.

1. Explain to a junior engineer why a random UUID as a primary key hurts insert performance, and what you would use instead.

2. A colleague says: "We should store prices as a float, it is simpler and the rounding is close enough." Explain what is wrong with that and what you would use instead.

3. Someone on your team wants to enable soft delete on every table by default "so that we never lose anything". Explain why you would push back, and what you would propose instead.

4. Describe what actually happens inside the database when a migration adds a column with a default value to a table of five hundred million rows, and describe the safe way to do the same change.

5. When would you choose an auto-increment integer key over UUID version seven, and when would you choose the opposite?

6. Explain to a product manager why the weekly report should not run on the main production database, and what it costs to move it somewhere else.

7. An AI tool has produced a migration and it runs without errors on the test database. Walk a teammate through the checks you would run before you approve it.
