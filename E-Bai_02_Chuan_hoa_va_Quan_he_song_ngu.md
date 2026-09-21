# Bài 2 — Chuẩn hoá & Quan hệ (Normalization)
### Bản song ngữ: tiếng Anh bám cấu trúc tiếng Việt

> **Cách dùng bài này:** đọc đoạn tiếng Việt trước, tự dịch trong đầu, rồi mới so với đoạn tiếng Anh bên dưới. Chỗ nào lệch thì xem bảng ánh xạ cụm khó. Cuối cùng, **đọc to đoạn tiếng Anh hai lần** — một lần chậm để đúng âm, một lần với tốc độ nói thật.

---

## Phần 1 — Chuẩn hoá và một nguồn sự thật duy nhất

**Tiếng Việt**

Chuẩn hoá nghĩa là chúng ta loại bỏ dữ liệu trùng lặp, để mỗi mẩu thông tin chỉ nằm ở đúng một chỗ. Chỗ duy nhất đó được gọi là nguồn sự thật duy nhất, và mọi nơi khác chỉ trỏ tới nó bằng khoá ngoại. Mục tiêu thật sự của chuẩn hoá không phải là sự gọn gàng, mà là chống lại các bất thường khi thêm, sửa và xoá dữ liệu. Bất thường khi sửa xảy ra khi một khách hàng đổi địa chỉ, và chúng ta phải sửa cùng một địa chỉ ở năm mươi dòng khác nhau. Bất thường khi thêm xảy ra khi chúng ta không thể lưu một sản phẩm mới, chỉ vì chưa có đơn hàng nào chứa sản phẩm đó. Bất thường khi xoá xảy ra khi chúng ta xoá đơn hàng cuối cùng của một khách, và thông tin của khách đó biến mất theo. Nếu chúng ta phải sửa một sự thật ở nhiều chỗ, chúng ta chắc chắn sẽ quên một chỗ nào đó, và từ hôm ấy dữ liệu bắt đầu lệch.

Chúng ta có thể coi mỗi bản sao thừa của một sự thật là một cơ hội để dữ liệu tự mâu thuẫn với chính nó. Khi chỉ có một bản, dữ liệu không thể mâu thuẫn, vì không có gì để so lệch. Vì vậy, chuẩn hoá là cách rẻ nhất để mua sự đúng đắn ngay tại tầng thiết kế. Chúng ta chỉ nên rời bỏ nguyên tắc này khi có một lý do hiệu năng đo được, và bài này sẽ nói về lúc đó ở phần sau.

**English (bám cấu trúc tiếng Việt)**

Normalisation means that we remove duplicated data, so that each piece of information sits in exactly one place. That single place is called the single source of truth, and every other place only points to it with a foreign key. The real goal of normalisation is not tidiness, but protection against the anomalies when we insert, update and delete data. An update anomaly happens when a customer changes their address, and we have to fix the same address in fifty different rows. An insert anomaly happens when we cannot save a new product, only because no order contains that product yet. A delete anomaly happens when we delete the last order of a customer, and the information of that customer disappears with it. If we have to fix one fact in many places, we will certainly forget one of them, and from that day the data starts to drift apart.

We can treat every redundant copy of a fact as one chance for the data to contradict itself. When there is only one copy, the data cannot contradict itself, because there is nothing to disagree with. Therefore, normalisation is the cheapest way to buy correctness right at the design layer. We should only leave this principle when we have a measurable performance reason, and this lesson will talk about that moment in a later part.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| mỗi mẩu thông tin chỉ nằm ở đúng một chỗ | each piece of information sits in exactly one place |
| trỏ tới nó bằng khoá ngoại | points to it with a foreign key |
| không phải là sự gọn gàng, mà là | is not tidiness, but |
| chống lại các bất thường | protection against the anomalies |
| chỉ vì chưa có đơn hàng nào chứa sản phẩm đó | only because no order contains that product yet |
| thông tin của khách đó biến mất theo | the information of that customer disappears with it |
| dữ liệu bắt đầu lệch | the data starts to drift apart |
| mỗi bản sao thừa của một sự thật | every redundant copy of a fact |
| dữ liệu tự mâu thuẫn với chính nó | the data contradicts itself |
| không có gì để so lệch | there is nothing to disagree with |
| cách rẻ nhất để mua sự đúng đắn | the cheapest way to buy correctness |

**Thuật ngữ cần nhớ**

- chuẩn hoá → **normalisation**
- nguồn sự thật duy nhất → **single source of truth**
- khoá ngoại → **foreign key**
- bất thường khi sửa / thêm / xoá → **update / insert / delete anomaly**
- dữ liệu trùng lặp → **duplicated data / redundancy**

---

## Phần 2 — Dạng chuẩn thứ nhất: mỗi ô một giá trị nguyên tử

**Tiếng Việt**

Dạng chuẩn thứ nhất yêu cầu mỗi ô trong bảng chỉ chứa một giá trị nguyên tử. Nguyên tử ở đây nghĩa là giá trị không thể tách nhỏ thêm nữa theo cách mà ứng dụng cần dùng. Ví dụ, chúng ta không được nhồi chuỗi "đỏ, xanh, vàng" vào một cột màu duy nhất, và chúng ta cũng không được nhồi danh sách sản phẩm vào một cột của bảng đơn hàng. Nếu chúng ta làm như vậy, cơ sở dữ liệu sẽ không thể index từng phần tử, và mọi truy vấn sẽ phải quét cả bảng rồi cắt chuỗi bằng tay. Hậu quả tiếp theo là chúng ta không thể đặt ràng buộc khoá ngoại lên từng phần tử, vì với cơ sở dữ liệu thì cả chuỗi chỉ là một đoạn văn bản. Cách sửa đúng là tách các phần tử ra thành một bảng con, ví dụ chúng ta tách order_item ra khỏi order, với một dòng cho mỗi sản phẩm trong đơn. Sau khi tách, chúng ta có thể đếm, lọc, JOIN và index trên từng dòng, và đó chính là thứ mà một cột chuỗi không bao giờ cho chúng ta.

**English (bám cấu trúc tiếng Việt)**

The first normal form requires that each cell in the table holds only one atomic value. Atomic here means that the value cannot be split any further in the way that the application needs to use it. For example, we must not stuff the string "red, green, yellow" into a single colour column, and we must not stuff a list of products into one column of the order table. If we do that, the database will not be able to index each element, and every query will have to scan the whole table and then cut the string by hand. The next consequence is that we cannot put a foreign key constraint on each element, because for the database the whole string is just a piece of text. The correct fix is to split the elements out into a child table, for example we split order_item out of order, with one row for each product in the order. After we split it, we can count, filter, JOIN and index on each row, and that is exactly the thing that a string column never gives us.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| mỗi ô trong bảng chỉ chứa một giá trị nguyên tử | each cell in the table holds only one atomic value |
| không thể tách nhỏ thêm nữa | cannot be split any further |
| theo cách mà ứng dụng cần dùng | in the way that the application needs to use it |
| nhồi chuỗi … vào một cột duy nhất | stuff the string … into a single column |
| quét cả bảng rồi cắt chuỗi bằng tay | scan the whole table and then cut the string by hand |
| đặt ràng buộc khoá ngoại lên từng phần tử | put a foreign key constraint on each element |
| cả chuỗi chỉ là một đoạn văn bản | the whole string is just a piece of text |
| tách … ra thành một bảng con | split … out into a child table |
| một dòng cho mỗi sản phẩm trong đơn | one row for each product in the order |
| đó chính là thứ mà … không bao giờ cho chúng ta | that is exactly the thing that … never gives us |

**Thuật ngữ cần nhớ**

- dạng chuẩn thứ nhất → **the first normal form (1NF)**
- giá trị nguyên tử → **atomic value**
- ô trong bảng → **cell**
- ràng buộc → **constraint**
- bảng con → **child table**
- quét toàn bảng → **full table scan**

---

## Phần 3 — Dạng chuẩn thứ hai: bỏ phụ thuộc bộ phận

**Tiếng Việt**

Dạng chuẩn thứ hai chỉ có ý nghĩa khi bảng đã đạt dạng chuẩn thứ nhất và khi bảng dùng một khoá chính gồm nhiều cột. Quy tắc là mọi cột không thuộc khoá phải phụ thuộc vào toàn bộ khoá, chứ không phụ thuộc vào một phần của khoá. Nếu một cột chỉ phụ thuộc vào một phần, chúng ta gọi đó là phụ thuộc bộ phận, và bảng chưa đạt dạng chuẩn thứ hai. Ví dụ, bảng order_item có khoá chính gồm order_id và product_id, nhưng tên sản phẩm chỉ phụ thuộc vào product_id. Vì vậy, tên sản phẩm không được nằm trong bảng order_item, mà phải nằm trong bảng product. Nếu chúng ta để nó nằm sai chỗ, tên sản phẩm sẽ bị lặp lại ở mọi dòng đơn hàng có sản phẩm đó, và khi đổi tên sản phẩm chúng ta sẽ phải sửa hàng nghìn dòng. Cách nhận ra lỗi này rất nhanh: chúng ta nhìn từng cột, chúng ta tự hỏi cột này phụ thuộc vào cái gì, rồi chúng ta so câu trả lời với khoá chính.

**English (bám cấu trúc tiếng Việt)**

The second normal form only makes sense when the table has already reached the first normal form and when the table uses a primary key made of several columns. The rule is that every non-key column must depend on the whole key, and must not depend on a part of the key. If a column depends on only a part, we call that a partial dependency, and the table has not reached the second normal form. For example, the order_item table has a primary key made of order_id and product_id, but the product name depends only on product_id. Therefore, the product name must not sit in the order_item table, but must sit in the product table. If we let it sit in the wrong place, the product name will be repeated in every order row that contains that product, and when the product is renamed we will have to fix thousands of rows. The way to spot this mistake is very quick: we look at each column, we ask ourselves what this column depends on, then we compare the answer with the primary key.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| chỉ có ý nghĩa khi | only makes sense when |
| một khoá chính gồm nhiều cột | a primary key made of several columns |
| mọi cột không thuộc khoá | every non-key column |
| phụ thuộc vào toàn bộ khoá | depend on the whole key |
| phụ thuộc vào một phần của khoá | depend on a part of the key |
| chúng ta gọi đó là phụ thuộc bộ phận | we call that a partial dependency |
| để nó nằm sai chỗ | let it sit in the wrong place |
| bị lặp lại ở mọi dòng đơn hàng | be repeated in every order row |
| cách nhận ra lỗi này rất nhanh | the way to spot this mistake is very quick |
| chúng ta tự hỏi cột này phụ thuộc vào cái gì | we ask ourselves what this column depends on |

**Thuật ngữ cần nhớ**

- dạng chuẩn thứ hai → **the second normal form (2NF)**
- khoá chính → **primary key**
- cột không thuộc khoá → **non-key column**
- phụ thuộc bộ phận → **partial dependency**
- khoá kép → **composite key**

---

## Phần 4 — Dạng chuẩn thứ ba: bỏ phụ thuộc bắc cầu

**Tiếng Việt**

Dạng chuẩn thứ ba yêu cầu một cột không thuộc khoá thì không được phụ thuộc gián tiếp vào khoá thông qua một cột không thuộc khoá khác. Kiểu phụ thuộc này được gọi là phụ thuộc bắc cầu. Ví dụ kinh điển là bảng khách hàng chứa cả mã bưu chính và tên thành phố, trong đó tên thành phố phụ thuộc vào mã bưu chính chứ không phụ thuộc vào mã khách hàng. Trong trường hợp đó, chúng ta nên tách mã bưu chính và thành phố ra một bảng riêng, và bảng khách hàng chỉ giữ lại mã bưu chính. Nếu chúng ta không tách, hai khách hàng có cùng mã bưu chính có thể mang hai tên thành phố khác nhau, và cơ sở dữ liệu không có cách nào ngăn chuyện đó. Ba dạng chuẩn này nên được hiểu theo nghĩa, chứ không nên học thuộc lòng như định nghĩa trong sách giáo khoa. Cách diễn đạt ngắn nhất mà chúng ta có thể dùng trong buổi phỏng vấn là: dạng chuẩn thứ nhất bắt giá trị phải nguyên tử, dạng chuẩn thứ hai bỏ phụ thuộc bộ phận, và dạng chuẩn thứ ba bỏ phụ thuộc bắc cầu.

**English (bám cấu trúc tiếng Việt)**

The third normal form requires that a non-key column must not depend on the key indirectly through another non-key column. This kind of dependency is called a transitive dependency. The classic example is a customer table that holds both the postcode and the city name, in which the city name depends on the postcode and does not depend on the customer id. In that case, we should split the postcode and the city out into a separate table, and the customer table only keeps the postcode. If we do not split them, two customers with the same postcode can carry two different city names, and the database has no way to stop that. These three normal forms should be understood by their meaning, and should not be learned by heart like a definition in a textbook. The shortest wording that we can use in an interview is: the first normal form forces values to be atomic, the second normal form removes partial dependencies, and the third normal form removes transitive dependencies.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| phụ thuộc gián tiếp vào khoá | depend on the key indirectly |
| thông qua một cột không thuộc khoá khác | through another non-key column |
| kiểu phụ thuộc này được gọi là | this kind of dependency is called |
| ví dụ kinh điển là | the classic example is |
| mã bưu chính | the postcode |
| tách … ra một bảng riêng | split … out into a separate table |
| chỉ giữ lại | only keeps |
| có thể mang hai tên thành phố khác nhau | can carry two different city names |
| không có cách nào ngăn chuyện đó | has no way to stop that |
| hiểu theo nghĩa | understood by their meaning |
| học thuộc lòng | learned by heart |
| cách diễn đạt ngắn nhất | the shortest wording |

**Thuật ngữ cần nhớ**

- dạng chuẩn thứ ba → **the third normal form (3NF)**
- phụ thuộc bắc cầu → **transitive dependency**
- mã bưu chính → **postcode** (Anh) / **zip code** (Mỹ)
- tách ra bảng riêng → **split into a separate table**
- học thuộc lòng → **learn by heart**

---

## Phần 5 — Denormalize có chủ đích và cái giá của nó

**Tiếng Việt**

Chúng ta denormalize khi hệ thống đọc rất nhiều và khi phép JOIN trở nên đắt đỏ. Cách làm là chúng ta nhân bản một phần dữ liệu sang chỗ đọc, để câu truy vấn không phải đi qua nhiều bảng nữa. Ví dụ, chúng ta thêm cột total_cents vào bảng order, để trang danh sách đơn hàng không phải cộng dồn các dòng order_item ở mỗi lần đọc. Đổi lại, chúng ta phải chấp nhận ba cái giá: đường ghi trở nên phức tạp hơn, dữ liệu có nguy cơ lệch nhau, và chúng ta phải tự lo việc đồng bộ. Có hai cách giữ cho dữ liệu nhân bản không lệch. Cách thứ nhất là chúng ta cập nhật bản gốc và bản sao trong cùng một transaction, hoặc chúng ta dùng trigger để cơ sở dữ liệu tự làm việc đó. Cách thứ hai là chúng ta chấp nhận nhất quán cuối cùng và chạy một job đồng bộ định kỳ, nhưng chúng ta phải ý thức rõ rủi ro mà mình vừa nhận.

Điểm mấu chốt là mỗi lần denormalize, chúng ta đưa ra một lời hứa với hệ thống. Lời hứa đó là chúng ta sẽ tự giữ cho hai bản dữ liệu khớp nhau mãi mãi, vì cơ sở dữ liệu không còn làm việc đó thay chúng ta nữa. Nếu chúng ta denormalize bừa bãi để chạy nhanh trước mắt, chúng ta sẽ tích luỹ nợ kỹ thuật dưới dạng dữ liệu lệch. Vì vậy, chúng ta chỉ nên denormalize sau khi đã đo được rằng phép JOIN đúng là nút thắt cổ chai.

**English (bám cấu trúc tiếng Việt)**

We denormalize when the system reads a lot and when the JOIN becomes expensive. The way to do it is that we duplicate a part of the data into the read side, so that the query does not have to go through many tables any more. For example, we add a total_cents column to the order table, so that the order list page does not have to add up the order_item rows on every read. In exchange, we have to accept three costs: the write path becomes more complex, the data is at risk of drifting apart, and we have to take care of the synchronisation ourselves. There are two ways to keep the duplicated data from drifting. The first way is that we update the original and the copy in the same transaction, or we use a trigger so that the database does that job by itself. The second way is that we accept eventual consistency and run a periodic sync job, but we must be fully aware of the risk that we have just taken on.

The key point is that every time we denormalize, we make a promise to the system. That promise is that we will keep the two copies of the data matching each other forever, because the database no longer does that job for us. If we denormalize carelessly in order to be fast in the short term, we will build up technical debt in the form of drifted data. Therefore, we should only denormalize after we have measured that the JOIN really is the bottleneck.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| khi phép JOIN trở nên đắt đỏ | when the JOIN becomes expensive |
| nhân bản một phần dữ liệu sang chỗ đọc | duplicate a part of the data into the read side |
| không phải cộng dồn … ở mỗi lần đọc | does not have to add up … on every read |
| chúng ta phải chấp nhận ba cái giá | we have to accept three costs |
| đường ghi trở nên phức tạp hơn | the write path becomes more complex |
| có nguy cơ lệch nhau | is at risk of drifting apart |
| tự lo việc đồng bộ | take care of the synchronisation ourselves |
| giữ cho dữ liệu nhân bản không lệch | keep the duplicated data from drifting |
| chạy một job đồng bộ định kỳ | run a periodic sync job |
| ý thức rõ rủi ro mà mình vừa nhận | be fully aware of the risk that we have just taken on |
| chúng ta đưa ra một lời hứa với hệ thống | we make a promise to the system |
| denormalize bừa bãi để chạy nhanh trước mắt | denormalize carelessly in order to be fast in the short term |
| tích luỹ nợ kỹ thuật | build up technical debt |
| nút thắt cổ chai | the bottleneck |

**Thuật ngữ cần nhớ**

- phi chuẩn hoá có chủ đích → **deliberate denormalisation**
- đường ghi → **the write path**
- đồng bộ → **synchronisation / keep in sync**
- nhất quán cuối cùng → **eventual consistency**
- nợ kỹ thuật → **technical debt**
- nút thắt cổ chai → **bottleneck**

---

## Phần 6 — Ba cách lưu dữ liệu tính sẵn và độ tươi cần thiết

**Tiếng Việt**

Khi chúng ta cần một con số tính sẵn, chúng ta có ba lựa chọn, và chúng ta chọn theo độ tươi mà nghiệp vụ cần. Lựa chọn thứ nhất là materialized view, tức là một bảng kết quả được cơ sở dữ liệu tính sẵn và làm mới định kỳ. Nó hợp với báo cáo, vì báo cáo chấp nhận dữ liệu hơi cũ, và nó không làm nặng đường ghi. Lựa chọn thứ hai là một cột denormalized nằm ngay trong bảng gốc, được cập nhật cùng lúc với dữ liệu gốc. Nó cho chúng ta con số theo thời gian thực, nhưng nó làm cho mỗi lần ghi tốn kém hơn. Lựa chọn thứ ba là cache với thời gian sống, và chúng ta chấp nhận rằng trong khoảng thời gian đó người dùng có thể nhìn thấy số cũ. Nếu chúng ta chọn sai theo hướng quá tươi, chúng ta trả giá bằng thông lượng ghi, còn nếu chúng ta chọn sai theo hướng quá cũ, chúng ta trả giá bằng niềm tin của người dùng.

**English (bám cấu trúc tiếng Việt)**

When we need a pre-computed number, we have three options, and we choose according to the freshness that the business needs. The first option is a materialized view, that is a result table which the database computes in advance and refreshes periodically. It fits reports, because a report accepts slightly old data, and it does not put weight on the write path. The second option is a denormalized column sitting right inside the original table, which is updated at the same time as the original data. It gives us the number in real time, but it makes every write more expensive. The third option is a cache with a time to live, and we accept that within that period the user may see an old number. If we choose wrongly towards too fresh, we pay with write throughput, and if we choose wrongly towards too old, we pay with the trust of the user.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| một con số tính sẵn | a pre-computed number |
| theo độ tươi mà nghiệp vụ cần | according to the freshness that the business needs |
| được cơ sở dữ liệu tính sẵn và làm mới định kỳ | which the database computes in advance and refreshes periodically |
| báo cáo chấp nhận dữ liệu hơi cũ | a report accepts slightly old data |
| nó không làm nặng đường ghi | it does not put weight on the write path |
| nằm ngay trong bảng gốc | sitting right inside the original table |
| được cập nhật cùng lúc với dữ liệu gốc | is updated at the same time as the original data |
| theo thời gian thực | in real time |
| thời gian sống | a time to live |
| người dùng có thể nhìn thấy số cũ | the user may see an old number |
| chọn sai theo hướng quá tươi | choose wrongly towards too fresh |
| chúng ta trả giá bằng niềm tin của người dùng | we pay with the trust of the user |

**Thuật ngữ cần nhớ**

- khung nhìn vật chất hoá → **materialized view**
- làm mới định kỳ → **refresh periodically**
- độ tươi của dữ liệu → **data freshness**
- dữ liệu cũ / ôi → **stale data**
- thời gian sống → **time to live (TTL)**
- thông lượng ghi → **write throughput**

---

## Phần 7 — Hệ giao dịch chuẩn hoá, kho phân tích thì ngược lại

**Tiếng Việt**

Hệ giao dịch trực tuyến và kho phân tích có hai mục tiêu ngược nhau, vì vậy chúng có hai kiểu thiết kế ngược nhau. Hệ giao dịch trực tuyến tối ưu cho việc ghi và cho tính nhất quán, vì vậy chúng ta chuẩn hoá bảng ở đó. Kho phân tích tối ưu cho việc đọc và cho các phép tổng hợp trên hàng tỉ dòng, vì vậy chúng ta denormalize ở đó. Thiết kế phổ biến trong kho phân tích là star schema, gồm một bảng fact ở giữa và các bảng dimension xung quanh. Bảng fact giữ các sự kiện đo được như từng dòng bán hàng, còn các bảng dimension giữ ngữ cảnh như sản phẩm, khách hàng và thời gian. Cách sắp xếp này giảm số phép JOIN cần thiết, và nhờ đó một câu truy vấn tổng hợp chạy nhanh hơn nhiều. Trong ngành, hai thế giới này được tách hẳn ra: hệ giao dịch chạy trên PostgreSQL hoặc MySQL, kho phân tích chạy trên Snowflake, BigQuery hoặc Redshift, và một pipeline ETL hoặc ELT chuyển dữ liệu từ bên này sang bên kia.

**English (bám cấu trúc tiếng Việt)**

An online transactional system and an analytical warehouse have two opposite goals, therefore they have two opposite design styles. An online transactional system optimises for writing and for consistency, therefore we normalise the tables there. An analytical warehouse optimises for reading and for aggregations over billions of rows, therefore we denormalize there. The common design in an analytical warehouse is the star schema, made of one fact table in the middle and the dimension tables around it. The fact table holds the measurable events such as each line of a sale, while the dimension tables hold the context such as product, customer and time. This arrangement reduces the number of JOINs needed, and thanks to that an aggregation query runs much faster. In the industry, these two worlds are kept fully apart: the transactional system runs on PostgreSQL or MySQL, the analytical warehouse runs on Snowflake, BigQuery or Redshift, and an ETL or ELT pipeline moves the data from one side to the other.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| có hai mục tiêu ngược nhau | have two opposite goals |
| tối ưu cho việc ghi và cho tính nhất quán | optimises for writing and for consistency |
| các phép tổng hợp trên hàng tỉ dòng | aggregations over billions of rows |
| gồm một bảng fact ở giữa | made of one fact table in the middle |
| các bảng dimension xung quanh | the dimension tables around it |
| các sự kiện đo được | the measurable events |
| từng dòng bán hàng | each line of a sale |
| giữ ngữ cảnh như … | hold the context such as … |
| cách sắp xếp này giảm số phép JOIN cần thiết | this arrangement reduces the number of JOINs needed |
| hai thế giới này được tách hẳn ra | these two worlds are kept fully apart |
| chuyển dữ liệu từ bên này sang bên kia | moves the data from one side to the other |

**Thuật ngữ cần nhớ**

- hệ giao dịch trực tuyến → **online transactional system (OLTP)**
- kho phân tích → **analytical warehouse (OLAP)**
- phép tổng hợp → **aggregation**
- bảng sự kiện và bảng chiều → **fact table and dimension table**
- đường ống dữ liệu → **data pipeline**

---

## Phần 8 — Khi người review nói "bảng này chưa chuẩn 3NF"

**Tiếng Việt**

Trong buổi review thiết kế, sẽ có người nói rằng bảng của chúng ta chưa đạt dạng chuẩn thứ ba. Chúng ta cần phân biệt hai trường hợp trước khi trả lời. Trường hợp thứ nhất là vấn đề thật, tức là dữ liệu bị trùng lặp ngoài ý muốn, không ai chịu trách nhiệm đồng bộ, và bất thường khi sửa đã bắt đầu xuất hiện. Trường hợp thứ hai là giáo điều, tức là thiết kế đã lệch chuẩn một cách có chủ đích vì hiệu năng, và chúng ta đã có cơ chế giữ đồng bộ rõ ràng. Dạng chuẩn thứ ba là một mặc định tốt, nhưng nó không phải là mục tiêu tự thân. Vì vậy, chúng ta nên trả lời bằng access pattern và bằng số đo, chứ không nên trả lời bằng tên của dạng chuẩn. Nếu chúng ta trả lời bằng tên của dạng chuẩn, người nghe sẽ nghĩ rằng chúng ta đang đọc lại sách giáo khoa, chứ không phải đang bảo vệ một thiết kế thật.

**English (bám cấu trúc tiếng Việt)**

In a design review, someone will say that our table has not reached the third normal form. We need to separate two cases before we answer. The first case is a real problem, that is the data is duplicated unintentionally, nobody takes responsibility for the synchronisation, and update anomalies have started to appear. The second case is dogma, that is the design has moved away from the standard deliberately for performance, and we already have a clear mechanism to keep things in sync. The third normal form is a good default, but it is not a goal in itself. Therefore, we should answer with the access pattern and with measurements, and we should not answer with the name of the normal form. If we answer with the name of the normal form, the listener will think that we are reciting a textbook, and not defending a real design.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| trong buổi review thiết kế | in a design review |
| chúng ta cần phân biệt hai trường hợp | we need to separate two cases |
| dữ liệu bị trùng lặp ngoài ý muốn | the data is duplicated unintentionally |
| không ai chịu trách nhiệm đồng bộ | nobody takes responsibility for the synchronisation |
| đã bắt đầu xuất hiện | have started to appear |
| lệch chuẩn một cách có chủ đích | has moved away from the standard deliberately |
| một cơ chế giữ đồng bộ rõ ràng | a clear mechanism to keep things in sync |
| một mặc định tốt | a good default |
| nó không phải là mục tiêu tự thân | it is not a goal in itself |
| trả lời bằng số đo | answer with measurements |
| chúng ta đang đọc lại sách giáo khoa | we are reciting a textbook |
| bảo vệ một thiết kế thật | defending a real design |

**Thuật ngữ cần nhớ**

- buổi review thiết kế → **design review**
- có chủ đích → **deliberate / deliberately**
- mục tiêu tự thân → **a goal in itself**
- cơ chế → **mechanism**
- số đo → **measurements**

---

## Mô hình ghi nhớ

**Tiếng Việt**

Chúng ta chuẩn hoá để dữ liệu đúng, và chúng ta denormalize để đọc nhanh. Mỗi lần denormalize là một lời hứa rằng chúng ta sẽ tự giữ cho hai bản dữ liệu khớp nhau.

**English (bám cấu trúc tiếng Việt)**

We normalise so that the data is correct, and we denormalize so that reads are fast. Every denormalisation is a promise that we will keep the two copies of the data matching each other ourselves.

---

## Bảng thuật ngữ tổng hợp

| Tiếng Việt | Tiếng Anh | Ghi chú phát âm |
|---|---|---|
| chuẩn hoá | normalisation | nor-mơ-lai-**ZAY**-shun; Anh viết *-isation* |
| phi chuẩn hoá | denormalisation | di-nor-mơ-lai-**ZAY**-shun |
| nguồn sự thật duy nhất | single source of truth | *truth* — âm **th** cuối, đầu lưỡi chạm răng |
| bất thường dữ liệu | anomaly | /əˈnɒməli/ — trọng âm 2: a-**NO**-mơ-li |
| bất thường khi sửa / thêm / xoá | update / insert / delete anomaly | |
| dữ liệu trùng lặp | redundancy | trọng âm 2: re-**DUN**-dan-si |
| lược đồ dữ liệu | schema | /ˈskiːmə/ — "**SKII**-mờ", đừng đọc "sờ-kê-ma" |
| dạng chuẩn thứ nhất | the first normal form (1NF) | |
| giá trị nguyên tử | atomic value | trọng âm 2: a-**TO**-mic |
| ô trong bảng | cell | |
| ràng buộc | constraint | /kənˈstreɪnt/ — cụm /str/ đọc liền, không chèn nguyên âm |
| khoá ngoại | foreign key | *foreign* /ˈfɒrən/ — "**FO**-rin", chữ **g** câm |
| khoá chính | primary key | Anh /ˈpraɪməri/ — "**PRAI**-mơ-ri" |
| khoá kép | composite key | trọng âm 1: **COM**-pơ-zit |
| cột không thuộc khoá | non-key column | *column* — chữ **n** cuối câm: "**CO**-lơm" |
| phụ thuộc bộ phận | partial dependency | *partial* /ˈpɑːʃl/ — "ti" đọc /ʃ/; de-**PEN**-den-si |
| phụ thuộc bắc cầu | transitive dependency | trọng âm 1: **TRAN**-si-tiv |
| mã bưu chính | postcode / zip code | |
| bảng con | child table | |
| quét toàn bảng | full table scan | *scan* — đuôi /-n/, đừng nuốt |
| toàn vẹn tham chiếu | referential integrity | re-fe-**REN**-shal · in-**TEG**-ri-ti |
| khung nhìn vật chất hoá | materialized view | mơ-**TIA**-ri-ơ-laizd — trọng âm 2 |
| làm mới | refresh | trọng âm 2: ri-**FRESH** |
| độ tươi của dữ liệu | data freshness | |
| dữ liệu cũ / ôi | stale data | /steɪl/ — "stêi-l", không đọc "sờ-ta-lê" |
| thời gian sống | time to live (TTL) | |
| bộ nhớ đệm | cache | /kæʃ/ — đọc y hệt "cash"; **không** đọc "ca-chê" |
| thông lượng ghi | write throughput | *throughput* /ˈθruːpʊt/ — âm **th** đầu lưỡi, không thành "trút" |
| đường ghi | the write path | *write* — chữ **w** câm |
| đồng bộ | synchronisation / sync | **SIN**-crơ-nai-zay-shun; *sync* đọc như "sink" |
| nhất quán cuối cùng | eventual consistency | i-**VEN**-chu-ơl · con-**SIS**-ten-si |
| nút thắt cổ chai | bottleneck | |
| nợ kỹ thuật | technical debt | *debt* /det/ — chữ **b** câm, đọc như "det" |
| kích hoạt tự động trong DB | trigger | |
| công việc chạy nền định kỳ | periodic job | pe-ri-**O**-dic |
| hệ giao dịch trực tuyến | online transactional system (OLTP) | tran-**SAC**-tion-al |
| kho phân tích | analytical warehouse (OLAP) | a-nơ-**LY**-ti-cal · *warehouse* — "**WEA**-hauz", có bật **h** |
| phép tổng hợp | aggregation | a-gri-**GAY**-shun |
| bảng sự kiện | fact table | |
| bảng chiều | dimension table | di-**MEN**-shun |
| lược đồ hình sao | star schema | |
| đường ống dữ liệu | data pipeline | |
| buổi review thiết kế | design review | |
| có chủ đích | deliberate | trọng âm 2: di-**LI**-bơ-rơt |
| cơ chế | mechanism | **ME**-cơ-ni-zơm — "ch" đọc /k/ |
| mục tiêu tự thân | a goal in itself | |
| rủi ro | risk | đuôi /-sk/ phải bật ra, không thành "rít" |

---

## Bài nói

> **Cách luyện:** bật ghi âm, nói **60–90 giây** cho mỗi đề, **không nhìn tài liệu**. Mỗi bài nói phải dùng **ít nhất 6 thuật ngữ** trong bảng trên. Nghe lại bản ghi một lần, đánh dấu chỗ ngập ngừng, rồi nói lại đề đó lần thứ hai.

1. Explain to a junior engineer what normalisation is actually protecting us from, using the three kinds of anomaly as your examples.

2. A colleague has designed an `orders` table with a `product_list` text column holding "12,45,78". Explain what is wrong with that design and how you would fix it.

3. A reviewer says: "This table is not in third normal form, so the design is wrong." Explain when that criticism is a real problem and when it is just dogma, and how you would answer in the review.

4. Someone on your team wants to add a denormalized `total_cents` column to the order table so the list page loads faster. Explain what you would ask for before you approve it, and what promise the team is making by adding it.

5. Describe what happens over the next two years to a system where the same customer address is stored in three different tables and nobody owns the synchronisation.

6. When would you choose a materialized view over a denormalized column, and when would you choose a cache instead of both?

7. Explain to a data analyst why the transactional database is normalised while the analytical warehouse uses a star schema, and why we do not simply run the reports on the transactional database.
