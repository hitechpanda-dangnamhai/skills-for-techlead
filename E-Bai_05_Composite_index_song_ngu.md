# Bài 5 — Composite index
### Bản song ngữ: tiếng Anh bám cấu trúc tiếng Việt

> **Cách dùng bài này:** đọc đoạn tiếng Việt trước, tự dịch trong đầu, rồi mới so với đoạn tiếng Anh bên dưới. Chỗ nào lệch thì xem bảng ánh xạ cụm khó. Cuối cùng, **đọc to đoạn tiếng Anh hai lần** — một lần chậm để đúng âm, một lần với tốc độ nói thật.

---

## Phần 1 — Composite index được đọc từ trái sang phải

**Tiếng Việt**

Composite index là một index đặt trên nhiều cột cùng một lúc, ví dụ một index đặt trên cặp cột a và b. Cách dễ nhất để hiểu nó là chúng ta hình dung một cuốn danh bạ được sắp xếp theo họ trước, rồi mới đến tên. Trong cuốn danh bạ đó, chúng ta tìm rất nhanh khi chúng ta biết họ, và chúng ta còn nhanh hơn nữa khi chúng ta biết cả họ lẫn tên. Nhưng nếu chúng ta chỉ biết tên mà không biết họ, cuốn danh bạ gần như vô dụng, vì chúng ta vẫn phải lật hết từng trang. Cơ sở dữ liệu hoạt động đúng như vậy, vì một composite index sắp các khoá theo cột đầu tiên trước, rồi mới sắp theo cột thứ hai bên trong từng nhóm. Vì lý do đó, chúng ta luôn phải đọc một composite index từ trái sang phải. Đây chính là kiến thức phân biệt người biết tạo index với người tạo đúng index.

**English (bám cấu trúc tiếng Việt)**

A composite index is an index built on several columns at the same time, for example an index built on the pair of columns a and b. The easiest way to understand it is that we picture a phone directory sorted by surname first, and only then by first name. In that directory, we search very fast when we know the surname, and we are even faster when we know both the surname and the first name. But if we know only the first name and not the surname, the directory is almost useless, because we still have to turn every page. A database works in exactly the same way, because a composite index sorts the keys by the first column first, and only then sorts by the second column inside each group. For that reason, we always have to read a composite index from left to right. This is exactly the knowledge that separates the person who knows how to create an index from the person who creates the right index.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| đặt trên nhiều cột cùng một lúc | built on several columns at the same time |
| chúng ta hình dung một cuốn danh bạ | we picture a phone directory |
| được sắp xếp theo họ trước, rồi mới đến tên | sorted by surname first, and only then by first name |
| chúng ta còn nhanh hơn nữa khi | we are even faster when |
| cuốn danh bạ gần như vô dụng | the directory is almost useless |
| chúng ta vẫn phải lật hết từng trang | we still have to turn every page |
| sắp các khoá theo cột đầu tiên trước | sorts the keys by the first column first |
| bên trong từng nhóm | inside each group |
| đọc … từ trái sang phải | read … from left to right |
| phân biệt người biết tạo index với người tạo đúng index | separates the person who knows how to create an index from the person who creates the right index |

**Thuật ngữ cần nhớ**

- index nhiều cột → **composite index / multicolumn index**
- danh bạ điện thoại → **phone directory**
- cặp cột → **pair of columns**
- nhóm giá trị → **group of values**
- từ trái sang phải → **from left to right**

---

## Phần 2 — Quy tắc leftmost prefix

**Tiếng Việt**

Quy tắc này được gọi là leftmost prefix, nghĩa là index chỉ được dùng khi truy vấn lọc theo một tiền tố tính từ bên trái. Với một index đặt trên ba cột a, b và c, một truy vấn lọc theo a sẽ dùng được index đó. Một truy vấn lọc theo cả a và b cũng dùng được, và một truy vấn lọc theo cả ba cột thì dùng được tốt nhất. Nhưng một truy vấn chỉ lọc theo b sẽ không tận dụng được index đó, vì các giá trị của b nằm rải rác trong từng nhóm a khác nhau. Tương tự, một truy vấn chỉ lọc theo c cũng không dùng được index, vì c còn nằm sâu hơn nữa. Hệ quả quan trọng là thứ tự cột có ý nghĩa, vì index trên cặp a và b không giống index trên cặp b và a. Nếu chúng ta tạo index sai thứ tự, index vẫn được tạo thành công, nhưng truy vấn nóng của chúng ta sẽ không bao giờ dùng tới nó.

**English (bám cấu trúc tiếng Việt)**

This rule is called the leftmost prefix, which means that the index is only used when the query filters on a prefix counted from the left. With an index built on the three columns a, b and c, a query that filters on a will be able to use that index. A query that filters on both a and b can also use it, and a query that filters on all three columns uses it best. But a query that filters only on b will not be able to take advantage of that index, because the values of b lie scattered inside each different a group. In the same way, a query that filters only on c cannot use the index either, because c sits even deeper. The important consequence is that the column order matters, because an index on the pair a and b is not the same as an index on the pair b and a. If we create the index in the wrong order, the index will still be created successfully, but our hot query will never reach for it.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| một tiền tố tính từ bên trái | a prefix counted from the left |
| sẽ dùng được index đó | will be able to use that index |
| dùng được tốt nhất | uses it best |
| sẽ không tận dụng được | will not be able to take advantage of |
| nằm rải rác trong từng nhóm a khác nhau | lie scattered inside each different a group |
| tương tự | in the same way |
| c còn nằm sâu hơn nữa | c sits even deeper |
| hệ quả quan trọng là thứ tự cột có ý nghĩa | the important consequence is that the column order matters |
| không giống | is not the same as |
| index vẫn được tạo thành công | the index will still be created successfully |
| sẽ không bao giờ dùng tới nó | will never reach for it |

**Thuật ngữ cần nhớ**

- tiền tố bên trái nhất → **leftmost prefix**
- thứ tự cột → **column order**
- truy vấn nóng → **hot query**
- tận dụng → **take advantage of**
- nằm rải rác → **lie scattered**

---

## Phần 3 — Composite index xoá bỏ bước sắp xếp

**Tiếng Việt**

Giá trị lớn nhất của composite index không nằm ở việc lọc, mà nằm ở việc nó xoá bỏ được bước sắp xếp. Chúng ta hãy xét một truy vấn lọc a bằng một giá trị rồi sắp xếp kết quả theo b. Với một index trên cặp a và b, tất cả các dòng có cùng giá trị a nằm liền nhau, và bên trong nhóm đó chúng đã sẵn thứ tự theo b. Vì vậy, cơ sở dữ liệu chỉ cần đi dọc theo index và trả kết quả ra ngay, và nó không cần chạy một bước sắp xếp riêng. Bước sắp xếp là một trong những thao tác đắt nhất, vì nó phải giữ toàn bộ tập kết quả trong bộ nhớ, và khi tập đó quá lớn thì cơ sở dữ liệu phải ghi tạm ra đĩa. Nếu chúng ta chỉ có index trên riêng cột a, cơ sở dữ liệu vẫn lọc nhanh, nhưng nó phải sắp xếp lại toàn bộ kết quả trước khi trả về. Với một trang danh sách có phân trang và có sắp xếp, khác biệt này quyết định giữa một truy vấn mười mili giây và một truy vấn hai giây.

**English (bám cấu trúc tiếng Việt)**

The biggest value of a composite index does not lie in the filtering, but lies in the fact that it removes the sort step. Let us look at a query that filters a by one value and then sorts the result by b. With an index on the pair a and b, all the rows that have the same a value sit next to each other, and inside that group they are already in order by b. Therefore, the database only needs to walk along the index and hand the result out immediately, and it does not need to run a separate sort step. The sort step is one of the most expensive operations, because it has to hold the whole result set in memory, and when that set is too large the database has to spill it to disk. If we only have an index on the a column alone, the database still filters quickly, but it has to sort the whole result again before returning it. For a list page that has pagination and sorting, this difference decides between a ten millisecond query and a two second query.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| không nằm ở việc lọc, mà nằm ở việc | does not lie in the filtering, but lies in the fact that |
| chúng ta hãy xét một truy vấn | let us look at a query |
| nằm liền nhau | sit next to each other |
| chúng đã sẵn thứ tự theo b | they are already in order by b |
| đi dọc theo index | walk along the index |
| trả kết quả ra ngay | hand the result out immediately |
| một bước sắp xếp riêng | a separate sort step |
| giữ toàn bộ tập kết quả trong bộ nhớ | hold the whole result set in memory |
| ghi tạm ra đĩa | spill it to disk |
| khác biệt này quyết định giữa … và … | this difference decides between … and … |

**Thuật ngữ cần nhớ**

- bước sắp xếp → **sort step**
- tập kết quả → **result set**
- ghi tạm ra đĩa → **spill to disk**
- phân trang → **pagination**
- mili giây → **millisecond**

---

## Phần 4 — Equality đặt trước, range và sort đặt sau

**Tiếng Việt**

Nguyên tắc sắp thứ tự cột rất ngắn: cột dùng để so sánh bằng thì đặt trước, còn cột dùng cho khoảng hoặc cho sắp xếp thì đặt sau. Lý do là một cột được lọc theo khoảng sẽ cắt đứt khả năng dùng các cột đứng sau nó trong index. Khi chúng ta lọc a theo khoảng, các giá trị của b bên trong khoảng đó không còn liền mạch nữa, vì mỗi giá trị a lại có dãy b riêng của nó. Ngược lại, khi chúng ta lọc a bằng đúng một giá trị, cơ sở dữ liệu bước vào đúng một nhóm, và bên trong nhóm đó cột b vẫn giữ nguyên thứ tự. Ví dụ, với một truy vấn lọc status bằng một giá trị và lọc created_at lớn hơn một mốc, chúng ta nên tạo index trên cặp status và created_at. Nếu chúng ta đảo lại thành cặp created_at và status, cơ sở dữ liệu sẽ phải quét cả một khoảng thời gian rộng rồi lọc status bằng tay trên từng dòng. Đây là lỗi rất hay gặp, vì người viết thường đặt cột thời gian lên đầu theo thói quen.

**English (bám cấu trúc tiếng Việt)**

The rule for ordering the columns is very short: the columns used for equality go first, while the columns used for a range or for sorting go last. The reason is that a column filtered by a range cuts off the ability to use the columns standing after it in the index. When we filter a by a range, the values of b inside that range are no longer continuous, because each a value has its own b sequence. On the other hand, when we filter a by exactly one value, the database steps into exactly one group, and inside that group the b column still keeps its order. For example, with a query that filters status by one value and filters created_at greater than a point in time, we should create an index on the pair status and created_at. If we flip it into the pair created_at and status, the database will have to scan a wide range of time and then filter status by hand on each row. This is a very common mistake, because the person writing it usually puts the time column first out of habit.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| nguyên tắc sắp thứ tự cột rất ngắn | the rule for ordering the columns is very short |
| cột dùng để so sánh bằng thì đặt trước | the columns used for equality go first |
| sẽ cắt đứt khả năng dùng các cột đứng sau nó | cuts off the ability to use the columns standing after it |
| không còn liền mạch nữa | are no longer continuous |
| mỗi giá trị a lại có dãy b riêng của nó | each a value has its own b sequence |
| cơ sở dữ liệu bước vào đúng một nhóm | the database steps into exactly one group |
| lọc created_at lớn hơn một mốc | filters created_at greater than a point in time |
| nếu chúng ta đảo lại thành | if we flip it into |
| lọc status bằng tay trên từng dòng | filter status by hand on each row |
| theo thói quen | out of habit |

**Thuật ngữ cần nhớ**

- lọc bằng giá trị chính xác → **equality filter**
- lọc theo khoảng → **range filter**
- cắt đứt → **cut off**
- liền mạch → **continuous**
- theo chiều giảm dần → **in descending order**

---

## Phần 5 — Hai index đơn không thay được một composite index

**Tiếng Việt**

Một hiểu lầm phổ biến là hai index đơn, một cái trên a và một cái trên b, cộng lại thì tương đương một composite index trên cặp a và b. Điều này không đúng, mặc dù cơ sở dữ liệu vẫn có cách kết hợp hai index đơn với nhau. Cách kết hợp đó được gọi là bitmap-AND, trong đó cơ sở dữ liệu quét từng index, dựng hai bản đồ bit, rồi giao hai bản đồ đó với nhau. Cách này chạy được, nhưng nó tốn thêm bộ nhớ, nó tốn thêm một bước dựng bitmap, và nó không cho chúng ta thứ tự sẵn để bỏ bước sắp xếp. Một composite index đúng thứ tự thì đi thẳng tới đúng vùng dữ liệu chỉ bằng một lần tra cứu. Vì vậy, khi một cặp cột thường xuyên xuất hiện cùng nhau trong cùng một truy vấn, chúng ta nên tạo một composite index thay vì tạo hai index rời.

**English (bám cấu trúc tiếng Việt)**

A common misunderstanding is that two single-column indexes, one on a and one on b, added together are equivalent to a composite index on the pair a and b. This is not true, although the database does have a way to combine two single-column indexes with each other. That way of combining is called a bitmap-AND, in which the database scans each index, builds two bit maps, and then intersects those two maps with each other. This way does work, but it costs extra memory, it costs an extra step to build the bitmaps, and it does not give us a ready order that would remove the sort step. A composite index in the right order goes straight to the right area of data with only one lookup. Therefore, when a pair of columns regularly appears together in the same query, we should create one composite index instead of creating two separate indexes.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| một hiểu lầm phổ biến là | a common misunderstanding is that |
| cộng lại thì tương đương | added together are equivalent to |
| mặc dù cơ sở dữ liệu vẫn có cách | although the database does have a way |
| dựng hai bản đồ bit | builds two bit maps |
| giao hai bản đồ đó với nhau | intersects those two maps with each other |
| cách này chạy được, nhưng | this way does work, but |
| nó không cho chúng ta thứ tự sẵn | it does not give us a ready order |
| đi thẳng tới đúng vùng dữ liệu | goes straight to the right area of data |
| chỉ bằng một lần tra cứu | with only one lookup |
| thường xuyên xuất hiện cùng nhau | regularly appears together |
| thay vì tạo hai index rời | instead of creating two separate indexes |

**Thuật ngữ cần nhớ**

- index một cột → **single-column index**
- kết hợp hai index bằng bitmap → **bitmap-AND**
- phép giao → **intersect / intersection**
- tương đương → **equivalent**
- hiểu lầm → **misunderstanding**

---

## Phần 6 — Đi từ một truy vấn nóng ra một index đúng

**Tiếng Việt**

Trong thực tế, chúng ta luôn bắt đầu từ câu truy vấn nóng chứ chúng ta không bắt đầu từ bảng. Giả sử truy vấn nóng của chúng ta lọc tenant_id bằng một giá trị, lọc status bằng một giá trị, rồi sắp xếp theo created_at giảm dần. Chúng ta gạch chân hai cột so sánh bằng, đó là tenant_id và status, và chúng ta đặt hai cột đó lên đầu index. Sau đó chúng ta đặt cột sắp xếp là created_at vào vị trí cuối cùng, và chúng ta khai báo luôn chiều giảm dần cho nó. Kết quả là một index trên bộ ba tenant_id, status và created_at giảm dần, và index này phục vụ cả phần lọc lẫn phần sắp xếp chỉ trong một lần đi. Nếu sau này có thêm một truy vấn cần vài cột nữa để hiển thị, chúng ta thêm các cột đó bằng mệnh đề INCLUDE để đạt index-only scan, chứ chúng ta không tạo thêm một index mới.

**English (bám cấu trúc tiếng Việt)**

In practice, we always start from the hot query and we do not start from the table. Suppose our hot query filters tenant_id by one value, filters status by one value, and then sorts by created_at in descending order. We underline the two equality columns, which are tenant_id and status, and we put those two columns at the front of the index. After that we put the sort column, which is created_at, in the last position, and we declare the descending direction for it right there. The result is an index on the triple tenant_id, status and created_at descending, and this index serves both the filtering part and the sorting part in a single pass. If later another query needs a few more columns for display, we add those columns with the INCLUDE clause in order to reach an index-only scan, and we do not create one more index.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| chúng ta luôn bắt đầu từ câu truy vấn nóng | we always start from the hot query |
| giả sử | suppose |
| sắp xếp theo created_at giảm dần | sorts by created_at in descending order |
| chúng ta gạch chân hai cột so sánh bằng | we underline the two equality columns |
| đặt hai cột đó lên đầu index | put those two columns at the front of the index |
| vào vị trí cuối cùng | in the last position |
| chúng ta khai báo luôn chiều giảm dần cho nó | we declare the descending direction for it right there |
| trên bộ ba | on the triple |
| chỉ trong một lần đi | in a single pass |
| vài cột nữa để hiển thị | a few more columns for display |

**Thuật ngữ cần nhớ**

- truy vấn nóng → **hot query**
- chiều giảm dần → **descending order**
- một lần đi | một lượt quét → **a single pass**
- cột để hiển thị → **display column**
- mệnh đề bao thêm cột → **the INCLUDE clause**

---

## Phần 7 — Bộ index tối thiểu và khoản thuế ghi vĩnh viễn

**Tiếng Việt**

Việc của một tech lead không phải là tối ưu từng index riêng lẻ, mà là tối ưu cả tập index của một bảng. Bước đầu tiên là chúng ta liệt kê các truy vấn thật đang chạy, rồi chúng ta gom chúng lại theo tiền tố bên trái mà chúng cần. Bước thứ hai là chúng ta loại bỏ mọi index mà tập cột của nó chỉ là tiền tố của một index khác đã có. Ví dụ, nếu bảng đã có index trên bộ ba a, b và c, thì index riêng trên a là thừa, và index trên cặp a và b cũng thừa. Chúng ta chỉ giữ lại các index ngắn hơn khi chúng phục vụ một chiều sắp xếp khác, hoặc khi chúng nhỏ hơn nhiều và nằm gọn trong bộ nhớ cho một truy vấn cực nóng. Lý do phải làm việc này là mỗi index thừa là một khoản thuế ghi vĩnh viễn, vì hệ thống trả tiền cho nó ở mọi lần thêm, sửa và xoá. Nếu chúng ta không dọn định kỳ, sau hai năm bảng sẽ có hơn mười index, và không ai còn nhớ index nào đang phục vụ truy vấn nào.

**English (bám cấu trúc tiếng Việt)**

The job of a tech lead is not to optimise each index on its own, but to optimise the whole index set of a table. The first step is that we list the real queries that are running, then we group them by the left prefix that they need. The second step is that we remove every index whose column set is only a prefix of another index that already exists. For example, if the table already has an index on the triple a, b and c, then a standalone index on a is redundant, and an index on the pair a and b is redundant too. We only keep the shorter indexes when they serve a different sort direction, or when they are much smaller and fit in memory for an extremely hot query. The reason we have to do this work is that every redundant index is a permanent write tax, because the system pays for it on every insert, update and delete. If we do not clean up periodically, after two years the table will have more than ten indexes, and nobody will remember any more which index is serving which query.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| tối ưu từng index riêng lẻ | optimise each index on its own |
| cả tập index của một bảng | the whole index set of a table |
| gom chúng lại theo tiền tố bên trái mà chúng cần | group them by the left prefix that they need |
| mà tập cột của nó chỉ là tiền tố của | whose column set is only a prefix of |
| index riêng trên a là thừa | a standalone index on a is redundant |
| phục vụ một chiều sắp xếp khác | serve a different sort direction |
| cho một truy vấn cực nóng | for an extremely hot query |
| một khoản thuế ghi vĩnh viễn | a permanent write tax |
| hệ thống trả tiền cho nó ở mọi lần … | the system pays for it on every … |
| nếu chúng ta không dọn định kỳ | if we do not clean up periodically |
| không ai còn nhớ index nào đang phục vụ truy vấn nào | nobody will remember which index is serving which query |

**Thuật ngữ cần nhớ**

- bộ index tối thiểu → **a minimal index set**
- index thừa → **redundant index**
- thuế ghi → **write tax**
- dọn dẹp định kỳ → **clean up periodically**
- chiều sắp xếp → **sort direction**

---

## Mô hình ghi nhớ

**Tiếng Việt**

Composite index được đọc từ trái sang phải, vì vậy chúng ta đặt cột so sánh bằng lên trước và đặt cột khoảng hoặc cột sắp xếp ra sau. Sau đó chúng ta bỏ đi mọi index mà tập cột của nó chỉ là tiền tố của một index khác.

**English (bám cấu trúc tiếng Việt)**

A composite index is read from left to right, therefore we put the equality columns first and put the range column or the sort column last. After that we drop every index whose column set is only a prefix of another index.

---

## Bảng thuật ngữ tổng hợp

| Tiếng Việt | Tiếng Anh | Ghi chú phát âm |
|---|---|---|
| index nhiều cột | composite index | **COM**-pơ-zit — trọng âm 1, đuôi đọc /-zɪt/ |
| index nhiều cột (cách gọi của Postgres) | multicolumn index | *column* — chữ **n** cuối câm: "**CO**-lơm" |
| index một cột | single-column index | |
| tiền tố bên trái nhất | leftmost prefix | *prefix* **PREE**-fix — trọng âm 1 |
| thứ tự cột | column order | |
| danh bạ điện thoại | phone directory | Anh /dɪˈrektəri/ — đi-**REC**-tơ-ri |
| họ / tên | surname / first name | **SUR**-neim — trọng âm 1 |
| lọc bằng giá trị chính xác | equality filter | *equality* i-**KWO**-lơ-ti — trọng âm 2 |
| lọc theo khoảng | range filter | |
| cắt đứt | cut off | |
| liền mạch | continuous | cơn-**TI**-niu-ơs |
| chiều tăng dần / giảm dần | ascending / descending order | ơ-**SEN**-ding · đi-**SEN**-ding |
| bước sắp xếp | sort step | |
| tập kết quả | result set | |
| ghi tạm ra đĩa | spill to disk | |
| phân trang | pagination | pa-ji-**NAY**-shơn |
| mili giây | millisecond | **MI**-li-se-cơnd |
| kết hợp hai index bằng bitmap | bitmap-AND | |
| phép giao | intersect / intersection | in-tơ-**SEKT** — trọng âm cuối |
| tương đương | equivalent | i-**KWI**-vơ-lơnt — trọng âm 2 |
| hiểu lầm | misunderstanding | mis-ơn-đơ-**STAN**-ding |
| lần tra cứu | lookup | |
| truy vấn nóng | hot query | *query* Anh /ˈkwɪəri/ — "**KWIƠ**-ri" |
| một lượt quét | a single pass | |
| quét chỉ dùng index | index-only scan | *scan* — đuôi /-n/ đừng nuốt |
| mệnh đề bao thêm cột | the INCLUDE clause | *clause* /klɔːz/ — "clô-z", đuôi /z/ |
| bộ index tối thiểu | a minimal index set | **MI**-ni-mơl |
| index thừa | redundant index | ri-**DUN**-đơnt — trọng âm 2 |
| thuế ghi | write tax | *write* — chữ **w** câm |
| vĩnh viễn | permanent | **PER**-mơ-nơnt — trọng âm 1 |
| dọn dẹp định kỳ | clean up periodically | pia-ri-**O**-đik-li |
| thói quen | habit | **HA**-bit — âm **h** phải bật |
| khách thuê trong hệ đa khách | tenant | **TE**-nơnt, không đọc "ti-nơnt" |
| rủi ro | risk | đuôi /-sk/ phải bật ra, không thành "rít" |
| bộ lập kế hoạch truy vấn | query planner | |

---

## Bài nói

> **Cách luyện:** bật ghi âm, nói **60–90 giây** cho mỗi đề, **không nhìn tài liệu**. Mỗi bài nói phải dùng **ít nhất 6 thuật ngữ** trong bảng trên. Nghe lại bản ghi một lần, đánh dấu chỗ ngập ngừng, rồi nói lại đề đó lần thứ hai.

1. Explain to a junior engineer what the leftmost prefix rule is, using an everyday example rather than database terms.

2. A colleague says: "We already have an index on `a` and an index on `b`, so a composite index on `(a, b)` would be pointless." Explain what is wrong with that.

3. Someone on your team has created an index on `(created_at, status)` for the query `WHERE status = ? AND created_at > ?`. Explain why you would push back and what order you would use instead.

4. Describe what the database does differently when a query with `WHERE a = ? ORDER BY b` has an index on `(a, b)` compared with an index on `a` alone.

5. When would you keep a shorter index even though it is a prefix of a longer one, and when would you drop it?

6. Given four hot queries on the same table, explain how you would design a minimal index set and how you would justify each index to the team.

7. Explain to a product manager why the team cannot simply add an index every time a page feels slow, and what the hidden cost is.
