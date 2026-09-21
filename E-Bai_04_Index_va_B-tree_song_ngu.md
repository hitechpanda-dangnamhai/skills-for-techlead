# Bài 4 — Index & B-tree
### Bản song ngữ: tiếng Anh bám cấu trúc tiếng Việt

> **Cách dùng bài này:** đọc đoạn tiếng Việt trước, tự dịch trong đầu, rồi mới so với đoạn tiếng Anh bên dưới. Chỗ nào lệch thì xem bảng ánh xạ cụm khó. Cuối cùng, **đọc to đoạn tiếng Anh hai lần** — một lần chậm để đúng âm, một lần với tốc độ nói thật.

---

## Phần 1 — Index là mục lục, và mục lục thì phải trả tiền

**Tiếng Việt**

Chúng ta có thể hình dung một index giống như mục lục ở cuối một cuốn sách dày. Khi không có mục lục, muốn tìm một từ chúng ta phải lật từng trang, và trong cơ sở dữ liệu việc đó được gọi là quét toàn bảng. Khi có mục lục, chúng ta tra một lần rồi chúng ta nhảy thẳng tới đúng trang cần đọc. Index chỉ là một cấu trúc phụ nằm cạnh bảng, và nó tồn tại để giúp chúng ta tìm nhanh hơn. Nhưng cấu trúc phụ đó không tự nhiên mà có được, vì mỗi lần chúng ta thêm, sửa hoặc xoá một dòng, cơ sở dữ liệu phải cập nhật cả bảng lẫn mọi index liên quan. Vì vậy, càng nhiều index thì đường ghi càng đắt, và đây chính là sự đánh đổi kinh điển giữa tốc độ đọc và tốc độ ghi. Nếu chúng ta thêm index cho mọi cột cho chắc, chúng ta sẽ làm chậm mọi thao tác ghi và tốn thêm rất nhiều dung lượng đĩa mà không đổi lại được gì.

**English (bám cấu trúc tiếng Việt)**

We can picture an index like the table of contents at the back of a thick book. When there is no index at the back, in order to find a word we have to turn every page, and in a database that is called a full table scan. When there is one, we look it up once and then we jump straight to the right page that we need to read. An index is only a secondary structure sitting next to the table, and it exists to help us search faster. But that secondary structure does not come for free, because every time we insert, update or delete a row, the database has to update both the table and every related index. Therefore, the more indexes we have, the more expensive the write path becomes, and this is exactly the classic trade-off between read speed and write speed. If we add an index on every column just to be safe, we will slow down every write operation and use up a lot of extra disk space without getting anything in exchange.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| mục lục ở cuối một cuốn sách dày | the table of contents at the back of a thick book |
| chúng ta phải lật từng trang | we have to turn every page |
| chúng ta tra một lần | we look it up once |
| nhảy thẳng tới đúng trang cần đọc | jump straight to the right page that we need to read |
| một cấu trúc phụ nằm cạnh bảng | a secondary structure sitting next to the table |
| không tự nhiên mà có được | does not come for free |
| cập nhật cả bảng lẫn mọi index liên quan | update both the table and every related index |
| càng nhiều index thì đường ghi càng đắt | the more indexes we have, the more expensive the write path becomes |
| sự đánh đổi kinh điển | the classic trade-off |
| thêm index cho mọi cột cho chắc | add an index on every column just to be safe |
| mà không đổi lại được gì | without getting anything in exchange |

**Thuật ngữ cần nhớ**

- quét toàn bảng → **full table scan**
- cấu trúc phụ → **secondary structure**
- đường ghi → **the write path**
- sự đánh đổi đọc–ghi → **the read–write trade-off**
- dung lượng đĩa → **disk space**

---

## Phần 2 — Vì sao B+tree là lựa chọn mặc định

**Tiếng Việt**

Index mặc định trong hầu hết cơ sở dữ liệu quan hệ là B+tree, và có lý do rất rõ ràng cho lựa chọn đó. B+tree là một cây cân bằng, nghĩa là mọi lá đều nằm ở cùng một độ sâu, vì vậy chi phí tìm kiếm luôn ổn định. Nó phục vụ được bốn kiểu truy vấn cùng một lúc: so sánh bằng, tìm theo khoảng, sắp xếp, và tìm theo tiền tố của chuỗi. Các lá của B+tree được nối với nhau thành một danh sách, nhờ đó cơ sở dữ liệu quét một khoảng chỉ bằng cách đi dọc theo lá. Ngoài ra, mỗi nút của cây được thiết kế vừa với một block đĩa, vì vậy số lần đọc đĩa cho một lần tra cứu rất ít. Hash index thì ngược lại, vì nó chỉ trả lời được câu hỏi so sánh bằng và nó không giữ bất kỳ thứ tự nào. Nếu chúng ta chọn hash index cho một cột hay được lọc theo khoảng ngày, chúng ta sẽ mất khả năng dùng index ngay khi truy vấn đầu tiên có dấu lớn hơn.

**English (bám cấu trúc tiếng Việt)**

The default index in most relational databases is the B+tree, and there is a very clear reason for that choice. A B+tree is a balanced tree, which means that all the leaves sit at the same depth, therefore the cost of a search is always stable. It serves four kinds of query at the same time: equality comparison, range lookup, sorting, and prefix search on a string. The leaves of a B+tree are linked together into a list, thanks to which the database scans a range simply by walking along the leaves. In addition, each node of the tree is designed to fit one disk block, therefore the number of disk reads for one lookup is very small. A hash index is the opposite, because it can only answer the equality question and it does not keep any ordering. If we choose a hash index for a column that is often filtered by date range, we will lose the ability to use the index as soon as the first query contains a greater-than sign.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| có lý do rất rõ ràng cho lựa chọn đó | there is a very clear reason for that choice |
| mọi lá đều nằm ở cùng một độ sâu | all the leaves sit at the same depth |
| chi phí tìm kiếm luôn ổn định | the cost of a search is always stable |
| nó phục vụ được bốn kiểu truy vấn cùng một lúc | it serves four kinds of query at the same time |
| tìm theo tiền tố của chuỗi | prefix search on a string |
| được nối với nhau thành một danh sách | are linked together into a list |
| chỉ bằng cách đi dọc theo lá | simply by walking along the leaves |
| được thiết kế vừa với một block đĩa | is designed to fit one disk block |
| nó không giữ bất kỳ thứ tự nào | it does not keep any ordering |
| ngay khi truy vấn đầu tiên có dấu lớn hơn | as soon as the first query contains a greater-than sign |

**Thuật ngữ cần nhớ**

- cây cân bằng → **balanced tree**
- lá của cây → **leaf / leaves**
- so sánh bằng → **equality comparison**
- tìm theo khoảng → **range lookup**
- tiền tố → **prefix**
- block đĩa → **disk block**

---

## Phần 3 — Clustered index và cái bẫy khi chuyển từ MySQL sang PostgreSQL

**Tiếng Việt**

Một index được gọi là clustered khi nó quyết định thứ tự vật lý của các dòng trên đĩa. Trong MySQL với engine InnoDB, khoá chính chính là clustered index, vì vậy các dòng nằm trên đĩa theo đúng thứ tự của khoá chính. Điều này làm cho việc quét theo khoảng của khoá chính rất nhanh, nhưng nó cũng làm cho mọi index phụ phải trỏ gián tiếp qua khoá chính. PostgreSQL không có clustered index thật, vì PostgreSQL lưu các dòng trong một cấu trúc gọi là heap, và thứ tự trong heap không được bảo đảm. PostgreSQL có lệnh CLUSTER, nhưng lệnh đó chỉ sắp xếp lại bảng đúng một lần, và nó không tự duy trì trật tự khi dữ liệu mới được ghi thêm. Vì vậy, nếu chúng ta mang giả định của MySQL sang PostgreSQL, chúng ta sẽ chờ đợi một hành vi mà PostgreSQL không hề có. Đây là loại câu hỏi phân biệt người đã đọc tài liệu với người chỉ nghe kể lại.

**English (bám cấu trúc tiếng Việt)**

An index is called clustered when it decides the physical order of the rows on disk. In MySQL with the InnoDB engine, the primary key is itself the clustered index, therefore the rows sit on disk in exactly the order of the primary key. This makes a range scan on the primary key very fast, but it also makes every secondary index point indirectly through the primary key. PostgreSQL does not have a true clustered index, because PostgreSQL stores the rows in a structure called a heap, and the order inside the heap is not guaranteed. PostgreSQL has the CLUSTER command, but that command only reorders the table exactly once, and it does not maintain the order by itself when new data is written afterwards. Therefore, if we carry a MySQL assumption over to PostgreSQL, we will expect a behaviour that PostgreSQL simply does not have. This is the kind of question that separates the person who has read the documentation from the person who has only heard it second-hand.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| quyết định thứ tự vật lý của các dòng trên đĩa | decides the physical order of the rows on disk |
| khoá chính chính là clustered index | the primary key is itself the clustered index |
| phải trỏ gián tiếp qua khoá chính | point indirectly through the primary key |
| một cấu trúc gọi là heap | a structure called a heap |
| thứ tự trong heap không được bảo đảm | the order inside the heap is not guaranteed |
| chỉ sắp xếp lại bảng đúng một lần | only reorders the table exactly once |
| nó không tự duy trì trật tự | it does not maintain the order by itself |
| mang giả định của MySQL sang PostgreSQL | carry a MySQL assumption over to PostgreSQL |
| một hành vi mà PostgreSQL không hề có | a behaviour that PostgreSQL simply does not have |
| người chỉ nghe kể lại | the person who has only heard it second-hand |

**Thuật ngữ cần nhớ**

- thứ tự vật lý → **physical order**
- index phụ → **secondary index**
- vùng lưu dòng không sắp thứ tự → **heap**
- quét theo khoảng → **range scan**
- giả định → **assumption**

---

## Phần 4 — Covering index và index-only scan

**Tiếng Việt**

Bình thường, cơ sở dữ liệu tra index để tìm vị trí của dòng, rồi nó phải đi đọc bảng để lấy các cột còn lại. Bước đi đọc bảng đó tốn thêm một lần truy cập ngẫu nhiên cho mỗi dòng, và với nhiều dòng thì chi phí này rất lớn. Nếu bản thân index đã chứa đủ mọi cột mà câu truy vấn cần, cơ sở dữ liệu có thể trả lời ngay từ index và bỏ hẳn bước đọc bảng. Cách đọc này được gọi là index-only scan, và một index như vậy được gọi là covering index. Trong PostgreSQL, chúng ta thêm các cột chỉ để đọc bằng mệnh đề INCLUDE, và những cột đó nằm ở lá của cây chứ không tham gia vào việc sắp thứ tự. Nhờ đó, index không phình lên ở các nút bên trong, nhưng câu truy vấn vẫn lấy đủ dữ liệu mà không cần chạm vào bảng. Nếu chúng ta bỏ qua kỹ thuật này, một truy vấn báo cáo chạy trên hàng triệu dòng sẽ mất phần lớn thời gian ở bước đi đọc bảng, chứ không phải ở bước tra index.

**English (bám cấu trúc tiếng Việt)**

Normally, the database looks up the index to find the position of the row, then it has to go and read the table to get the remaining columns. That step of going to read the table costs one extra random access for each row, and with many rows this cost is very large. If the index itself already contains all the columns that the query needs, the database can answer straight from the index and drop the table read step completely. This way of reading is called an index-only scan, and an index like that is called a covering index. In PostgreSQL, we add the read-only columns with the INCLUDE clause, and those columns sit in the leaves of the tree and do not take part in the ordering. Thanks to that, the index does not grow fat in the inner nodes, but the query still gets all the data without touching the table. If we ignore this technique, a report query running over millions of rows will spend most of its time in the table read step, and not in the index lookup step.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| tra index để tìm vị trí của dòng | looks up the index to find the position of the row |
| đi đọc bảng để lấy các cột còn lại | go and read the table to get the remaining columns |
| tốn thêm một lần truy cập ngẫu nhiên cho mỗi dòng | costs one extra random access for each row |
| chứa đủ mọi cột mà câu truy vấn cần | contains all the columns that the query needs |
| trả lời ngay từ index | answer straight from the index |
| bỏ hẳn bước đọc bảng | drop the table read step completely |
| không tham gia vào việc sắp thứ tự | do not take part in the ordering |
| index không phình lên ở các nút bên trong | the index does not grow fat in the inner nodes |
| mà không cần chạm vào bảng | without touching the table |
| sẽ mất phần lớn thời gian ở bước … | will spend most of its time in the … step |

**Thuật ngữ cần nhớ**

- quét chỉ dùng index → **index-only scan**
- index phủ đủ cột → **covering index**
- truy cập ngẫu nhiên → **random access**
- nút bên trong / nút lá → **inner node / leaf node**
- mệnh đề → **clause**

---

## Phần 5 — Selectivity: vì sao index trên cột boolean thường vô dụng

**Tiếng Việt**

Một index chỉ hữu ích khi nó loại được phần lớn dữ liệu ra khỏi kết quả. Khả năng lọc này được gọi là selectivity, và nó gắn liền với cardinality, tức là số lượng giá trị khác nhau trong cột. Cột email có cardinality rất cao, vì gần như mỗi dòng mang một giá trị riêng, vì vậy index trên cột đó lọc rất tốt. Cột is_active kiểu boolean có cardinality bằng hai, vì vậy một truy vấn lọc theo cột đó vẫn phải lấy về phần lớn bảng. Trong tình huống ấy, bộ lập kế hoạch truy vấn sẽ tính toán và kết luận rằng quét tuần tự cả bảng còn rẻ hơn là nhảy ngẫu nhiên hàng triệu lần. Vì vậy, cơ sở dữ liệu sẽ bỏ qua index của chúng ta, và index đó chỉ còn làm chậm đường ghi mà không giúp gì cho đường đọc. Nếu chín mươi chín phần trăm số dòng đang active nhưng chúng ta lại hay tìm những dòng không active, một partial index sẽ hữu ích hơn nhiều so với một index đầy đủ.

**English (bám cấu trúc tiếng Việt)**

An index is only useful when it removes most of the data from the result. This filtering power is called selectivity, and it is tied to cardinality, that is the number of distinct values in the column. The email column has very high cardinality, because almost every row carries its own value, therefore an index on that column filters very well. The is_active boolean column has a cardinality of two, therefore a query that filters on that column still has to bring back most of the table. In that situation, the query planner will do the arithmetic and conclude that scanning the whole table sequentially is cheaper than jumping randomly millions of times. Therefore, the database will ignore our index, and that index will only slow down the write path without helping the read path at all. If ninety-nine percent of the rows are active but we often look for the rows that are not active, a partial index will be far more useful than a full index.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| loại được phần lớn dữ liệu ra khỏi kết quả | removes most of the data from the result |
| khả năng lọc này được gọi là | this filtering power is called |
| số lượng giá trị khác nhau trong cột | the number of distinct values in the column |
| gần như mỗi dòng mang một giá trị riêng | almost every row carries its own value |
| vẫn phải lấy về phần lớn bảng | still has to bring back most of the table |
| bộ lập kế hoạch truy vấn sẽ tính toán | the query planner will do the arithmetic |
| quét tuần tự cả bảng còn rẻ hơn | scanning the whole table sequentially is cheaper |
| nhảy ngẫu nhiên hàng triệu lần | jumping randomly millions of times |
| cơ sở dữ liệu sẽ bỏ qua index của chúng ta | the database will ignore our index |
| mà không giúp gì cho đường đọc | without helping the read path at all |

**Thuật ngữ cần nhớ**

- khả năng lọc của index → **selectivity**
- số giá trị khác nhau → **cardinality**
- giá trị phân biệt → **distinct value**
- bộ lập kế hoạch truy vấn → **query planner**
- quét tuần tự → **sequential scan**

---

## Phần 6 — Có index mà vẫn quét toàn bảng

**Tiếng Việt**

Một câu hỏi phỏng vấn rất hay là: chúng ta đã có index trên cột đó rồi, vậy vì sao truy vấn vẫn quét toàn bảng. Khái niệm cần dùng ở đây là sargable, nghĩa là điều kiện lọc được viết theo cách mà cơ sở dữ liệu có thể dùng được index. Nguyên nhân thứ nhất là chúng ta bọc cột trong một hàm, ví dụ chúng ta viết where lower của email bằng một giá trị, trong khi index lại nằm trên cột email nguyên bản. Nguyên nhân thứ hai là kiểu dữ liệu không khớp, vì khi cơ sở dữ liệu phải ép kiểu cột thì nó không còn so sánh trực tiếp với index nữa. Nguyên nhân thứ ba là mẫu tìm kiếm bắt đầu bằng ký tự đại diện, vì lúc đó cây B+tree không biết phải bắt đầu tìm từ đâu. Nguyên nhân thứ tư là selectivity quá thấp, và nguyên nhân thứ năm là thống kê của bảng đã cũ nên bộ lập kế hoạch ước lượng sai số dòng. Nguyên nhân thứ sáu là điều kiện OR trải trên nhiều cột, vì khi đó cơ sở dữ liệu phải gộp nhiều đường truy cập và nó có thể chọn quét toàn bảng cho đơn giản.

**English (bám cấu trúc tiếng Việt)**

A very good interview question is: we already have an index on that column, so why does the query still scan the whole table. The concept we need here is sargable, which means that the filter condition is written in a way that the database can actually use the index. The first cause is that we wrap the column inside a function, for example we write where lower of email equals a value, while the index sits on the raw email column. The second cause is a data type mismatch, because when the database has to cast the column it no longer compares directly against the index. The third cause is a search pattern that starts with a wildcard character, because at that point the B+tree does not know where it should start searching. The fourth cause is that the selectivity is too low, and the fifth cause is that the table statistics are stale so the planner estimates the row count wrongly. The sixth cause is an OR condition spread across several columns, because in that case the database has to combine several access paths and it may choose a full scan for simplicity.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| vậy vì sao truy vấn vẫn quét toàn bảng | so why does the query still scan the whole table |
| khái niệm cần dùng ở đây là | the concept we need here is |
| được viết theo cách mà cơ sở dữ liệu có thể dùng được index | is written in a way that the database can actually use the index |
| chúng ta bọc cột trong một hàm | we wrap the column inside a function |
| trên cột email nguyên bản | on the raw email column |
| kiểu dữ liệu không khớp | a data type mismatch |
| khi cơ sở dữ liệu phải ép kiểu cột | when the database has to cast the column |
| mẫu tìm kiếm bắt đầu bằng ký tự đại diện | a search pattern that starts with a wildcard character |
| không biết phải bắt đầu tìm từ đâu | does not know where it should start searching |
| thống kê của bảng đã cũ | the table statistics are stale |
| ước lượng sai số dòng | estimates the row count wrongly |
| gộp nhiều đường truy cập | combine several access paths |
| chọn quét toàn bảng cho đơn giản | choose a full scan for simplicity |

**Thuật ngữ cần nhớ**

- điều kiện dùng được index → **sargable condition**
- ép kiểu → **cast / type coercion**
- ký tự đại diện đứng đầu → **leading wildcard**
- thống kê bảng → **table statistics**
- đường truy cập → **access path**

---

## Phần 7 — Partial index và expression index

**Tiếng Việt**

Hai loại index đặc biệt giúp chúng ta xử lý đúng những vấn đề vừa nêu ở trên. Partial index là index chỉ phủ một phần của bảng, và phần đó được xác định bằng một mệnh đề WHERE ngay lúc tạo index. Ví dụ, nếu chín mươi lăm phần trăm đơn hàng đã đóng và chúng ta chỉ truy vấn đơn hàng đang mở, chúng ta chỉ nên index những dòng đang mở. Một index như vậy nhỏ hơn nhiều, nó nằm gọn trong bộ nhớ, và nó khiến mỗi lần tra cứu rẻ hơn hẳn. Expression index là index đặt trên kết quả của một biểu thức thay vì trên cột nguyên bản, ví dụ chúng ta index trên lower của email để khớp đúng câu truy vấn đang chạy. Cả hai loại này đều theo cùng một nguyên tắc, đó là chúng ta index đúng thứ mà truy vấn thật sự hỏi, chứ chúng ta không index cả bảng cho chắc.

**English (bám cấu trúc tiếng Việt)**

Two special kinds of index help us handle exactly the problems mentioned above. A partial index is an index that covers only a part of the table, and that part is defined by a WHERE clause right at the moment we create the index. For example, if ninety-five percent of the orders are already closed and we only query the open orders, we should index only the open rows. An index like that is much smaller, it fits comfortably in memory, and it makes each lookup clearly cheaper. An expression index is an index built on the result of an expression instead of on the raw column, for example we index on lower of email in order to match the query that is actually running. Both of these kinds follow the same principle, namely that we index exactly what the query really asks for, and we do not index the whole table just to be safe.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| những vấn đề vừa nêu ở trên | exactly the problems mentioned above |
| chỉ phủ một phần của bảng | covers only a part of the table |
| được xác định bằng một mệnh đề WHERE | is defined by a WHERE clause |
| ngay lúc tạo index | right at the moment we create the index |
| chúng ta chỉ nên index những dòng đang mở | we should index only the open rows |
| nó nằm gọn trong bộ nhớ | it fits comfortably in memory |
| khiến mỗi lần tra cứu rẻ hơn hẳn | makes each lookup clearly cheaper |
| đặt trên kết quả của một biểu thức | built on the result of an expression |
| để khớp đúng câu truy vấn đang chạy | in order to match the query that is actually running |
| index đúng thứ mà truy vấn thật sự hỏi | index exactly what the query really asks for |

**Thuật ngữ cần nhớ**

- index một phần bảng → **partial index**
- index trên biểu thức → **expression index**
- cột nguyên bản → **the raw column**
- nằm gọn trong bộ nhớ → **fit in memory**
- lần tra cứu → **lookup**

---

## Phần 8 — Dọn index thừa dựa trên số liệu, không dựa trên cảm tính

**Tiếng Việt**

Một tình huống rất thật là hệ thống có mười hai index trên một bảng và tốc độ ghi chậm dần theo từng tháng. Việc đầu tiên chúng ta làm không phải là đoán, mà là mở bảng thống kê pg_stat_user_indexes ra xem. Bảng đó cho chúng ta cột idx_scan, tức là số lần mỗi index thật sự được dùng kể từ lần đặt lại thống kê gần nhất. Những index có idx_scan bằng không là ứng viên đầu tiên để xoá, vì không có truy vấn nào đang dùng chúng. Nhóm thứ hai là các index thừa, tức là index mà tập cột của nó chỉ là tiền tố của một composite index khác đã tồn tại. Sau khi liệt kê hai nhóm đó, chúng ta ước lượng mức khuếch đại ghi mà mỗi index gây ra, rồi chúng ta quyết định dựa trên số liệu chứ không dựa trên cảm tính. Chúng ta cũng phải kiểm tra xem index nào đang phục vụ một ràng buộc duy nhất hoặc một khoá ngoại, vì xoá nhầm những index đó sẽ phá vỡ ràng buộc dữ liệu.

Khi tạo index trên môi trường production, chúng ta nên dùng lệnh CREATE INDEX CONCURRENTLY. Lệnh này xây index mà không khoá bảng cho việc ghi, vì vậy người dùng vẫn dùng được hệ thống trong lúc index đang được tạo. Đổi lại, lệnh này chạy lâu hơn và nó không được phép chạy bên trong một transaction, vì vậy chúng ta phải tách nó ra khỏi migration thông thường.

**English (bám cấu trúc tiếng Việt)**

A very real situation is a system with twelve indexes on one table where the write speed gets slower month by month. The first thing we do is not to guess, but to open the pg_stat_user_indexes statistics view and look at it. That view gives us the idx_scan column, that is the number of times each index has actually been used since the last statistics reset. The indexes with idx_scan equal to zero are the first candidates for removal, because no query is using them. The second group is the redundant indexes, that is an index whose column set is only a prefix of another composite index that already exists. After we have listed those two groups, we estimate the write amplification that each index causes, then we decide based on numbers and not based on gut feeling. We also have to check which index is serving a unique constraint or a foreign key, because dropping those indexes by mistake will break the data constraints.

When we create an index in a production environment, we should use the CREATE INDEX CONCURRENTLY command. This command builds the index without locking the table for writes, therefore users can still use the system while the index is being built. In exchange, this command runs longer and it is not allowed to run inside a transaction, therefore we have to keep it out of the ordinary migration.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| tốc độ ghi chậm dần theo từng tháng | the write speed gets slower month by month |
| việc đầu tiên chúng ta làm không phải là đoán | the first thing we do is not to guess |
| kể từ lần đặt lại thống kê gần nhất | since the last statistics reset |
| là ứng viên đầu tiên để xoá | are the first candidates for removal |
| tập cột của nó chỉ là tiền tố của | its column set is only a prefix of |
| mức khuếch đại ghi | the write amplification |
| dựa trên số liệu chứ không dựa trên cảm tính | based on numbers and not based on gut feeling |
| xoá nhầm những index đó | dropping those indexes by mistake |
| sẽ phá vỡ ràng buộc dữ liệu | will break the data constraints |
| mà không khoá bảng cho việc ghi | without locking the table for writes |
| trong lúc index đang được tạo | while the index is being built |
| nó không được phép chạy bên trong một transaction | it is not allowed to run inside a transaction |

**Thuật ngữ cần nhớ**

- index thừa → **redundant index**
- khuếch đại ghi → **write amplification**
- ứng viên để xoá → **candidate for removal**
- cảm tính → **gut feeling**
- môi trường production → **production environment**

---

## Mô hình ghi nhớ

**Tiếng Việt**

Index là việc chúng ta đổi tốc độ ghi để lấy tốc độ đọc. Mỗi index phải trả lời được câu hỏi truy vấn nào đang dùng nó, và nếu không có ai dùng thì chúng ta xoá nó đi.

**English (bám cấu trúc tiếng Việt)**

An index is us trading write speed for read speed. Every index must be able to answer the question of which query is using it, and if nobody is using it then we drop it.

---

## Bảng thuật ngữ tổng hợp

| Tiếng Việt | Tiếng Anh | Ghi chú phát âm |
|---|---|---|
| mục lục | table of contents | |
| quét toàn bảng | full table scan | *scan* — đuôi /-n/ đừng nuốt |
| quét tuần tự | sequential scan | si-**KWEN**-shơl |
| cấu trúc phụ | secondary structure | **SE**-cơn-đơ-ri · *structure* — cụm /str/ đọc liền |
| đường ghi / đường đọc | the write path / the read path | *write* — chữ **w** câm |
| sự đánh đổi | trade-off | |
| dung lượng đĩa | disk space | |
| cây cân bằng | balanced tree | **BA**-lơnst |
| lá của cây | leaf / leaves | /liːf/ · /liːvz/ — số nhiều đổi **f** thành **v** |
| nút bên trong | inner node | |
| so sánh bằng | equality comparison | i-**KWO**-lơ-ti — trọng âm 2 |
| tìm theo khoảng | range lookup | |
| tiền tố | prefix | **PREE**-fix — trọng âm 1 |
| block đĩa | disk block | |
| thứ tự vật lý | physical order | *physical* /ˈfɪzɪkl/ — "ph" đọc /f/ |
| index phụ | secondary index | |
| vùng lưu dòng không sắp thứ tự | heap | /hiːp/ — "hiip", âm **h** phải bật |
| được bảo đảm | guaranteed | ga-rơn-**TEED** — trọng âm cuối |
| giả định | assumption | ơ-**SUMP**-shơn |
| hành vi | behaviour | bi-**HEIV**-yơ; Anh viết *-our* |
| quét chỉ dùng index | index-only scan | |
| index phủ đủ cột | covering index | |
| truy cập ngẫu nhiên | random access | |
| mệnh đề | clause | /klɔːz/ — "clô-z", đuôi /z/ |
| khả năng lọc của index | selectivity | se-lek-**TI**-vơ-ti |
| số giá trị khác nhau | cardinality | car-đi-**NA**-lơ-ti — trọng âm 3 |
| giá trị phân biệt | distinct value | dis-**TINKT** — đuôi /-ŋkt/ khó, bật nhẹ |
| bộ lập kế hoạch truy vấn | query planner | *query* Anh /ˈkwɪəri/ — "**KWIƠ**-ri" |
| ước lượng | estimate | động từ **ES**-ti-meit; danh từ **ES**-ti-mơt |
| điều kiện dùng được index | sargable condition | **SAR**-gơ-bl — trọng âm 1 |
| ép kiểu | cast / type coercion | *coercion* cơ-**ER**-shơn |
| ký tự đại diện đứng đầu | leading wildcard | |
| thống kê bảng | table statistics | stơ-**TIS**-tics — trọng âm 2 |
| dữ liệu cũ / ôi | stale | /steɪl/ — "stêi-l" |
| đường truy cập | access path | *access* danh từ trọng âm 1: **AC**-cess |
| index một phần bảng | partial index | *partial* /ˈpɑːʃl/ — "ti" đọc /ʃ/ |
| index trên biểu thức | expression index | ex-**PRE**-shơn |
| cột nguyên bản | the raw column | *column* — chữ **n** cuối câm |
| lần tra cứu | lookup | |
| index thừa | redundant index | ri-**DUN**-đơnt — trọng âm 2 |
| khuếch đại ghi | write amplification | am-pli-fi-**CAY**-shơn |
| đồng thời, không khoá bảng | concurrently | cơn-**CU**-rơnt-li — trọng âm 2 |
| ràng buộc duy nhất | unique constraint | *unique* trọng âm 2: yu-**NEEK** |
| khoá ngoại | foreign key | *foreign* — chữ **g** câm: "**FO**-rin" |
| cảm tính | gut feeling | |
| môi trường production | production environment | en-**VAI**-rơn-mơnt |
| ứng viên để xoá | candidate for removal | **CAN**-đi-đơt |

---

## Bài nói

> **Cách luyện:** bật ghi âm, nói **60–90 giây** cho mỗi đề, **không nhìn tài liệu**. Mỗi bài nói phải dùng **ít nhất 6 thuật ngữ** trong bảng trên. Nghe lại bản ghi một lần, đánh dấu chỗ ngập ngừng, rồi nói lại đề đó lần thứ hai.

1. Explain to a junior engineer why adding an index speeds up reads but slows down writes, and how you decide whether a new index is worth it.

2. A colleague says: "The query is slow, so let us just add an index on every column in the WHERE clause." Explain why you would push back and what you would do first.

3. A teammate has created an index on a boolean `is_active` column and complains that the database refuses to use it. Explain what is happening inside the planner and what you would suggest instead.

4. Describe what happens step by step when a query can be answered by an index-only scan, and what changes when the index does not cover every column.

5. Someone reports that a query on `lower(email)` is slow even though there is an index on `email`. Walk them through the cause and through two different ways to fix it.

6. When would you choose a partial index over a full index, and when would that choice backfire?

7. Your table has twelve indexes and writes are getting slower every month. Explain to your team how you would decide which indexes to drop, and what you would check before dropping any of them.
