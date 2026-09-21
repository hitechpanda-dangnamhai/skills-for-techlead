# Bài 9 — PostgreSQL specifics
### Bản song ngữ: tiếng Anh bám cấu trúc tiếng Việt

> **Cách dùng bài này:** đọc đoạn tiếng Việt trước, tự dịch trong đầu, rồi mới so với đoạn tiếng Anh bên dưới. Chỗ nào lệch thì xem bảng ánh xạ cụm khó. Cuối cùng, **đọc to đoạn tiếng Anh hai lần** — một lần chậm để đúng âm, một lần với tốc độ nói thật.

---

## Phần 1 — JSONB và ranh giới của nó

**Tiếng Việt**

PostgreSQL có hai kiểu dữ liệu cho JSON, và chúng ta gần như luôn chọn kiểu jsonb. Kiểu json lưu nguyên văn bản gốc, vì vậy mỗi lần truy vấn, cơ sở dữ liệu phải phân tích lại chuỗi đó từ đầu. Kiểu jsonb lưu dữ liệu dưới dạng nhị phân đã được phân tích sẵn, nhờ đó chúng ta truy vấn nhanh hơn và chúng ta đánh index được lên nó. Đổi lại, jsonb tốn thêm một chút công lúc ghi và nó không giữ nguyên thứ tự khoá cũng như khoảng trắng của bản gốc. Kiểu jsonb hợp với dữ liệu bán cấu trúc, ví dụ cấu hình, cờ tính năng, và phần payload của sự kiện. Nhưng chúng ta không nên thay toàn bộ schema quan hệ bằng jsonb, vì làm như vậy chúng ta mất kiểu dữ liệu chặt, mất ràng buộc, và mất khả năng tối ưu của bộ lập kế hoạch. Quy tắc thực dụng là chúng ta để phần dữ liệu ổn định trong các cột quan hệ, và chúng ta chỉ đẩy phần linh hoạt vào một cột jsonb.

**English (bám cấu trúc tiếng Việt)**

PostgreSQL has two data types for JSON, and we almost always choose the jsonb type. The json type stores the original text as it is, therefore on every query the database has to parse that string again from the beginning. The jsonb type stores the data in a binary form that has already been parsed, thanks to which we query faster and we can build an index on it. In exchange, jsonb costs a bit more work at write time and it does not keep the original key order or the original whitespace. The jsonb type fits semi-structured data, for example configuration, feature flags, and the payload part of an event. But we should not replace the whole relational schema with jsonb, because doing that we lose strict data types, we lose constraints, and we lose the optimisation ability of the planner. The pragmatic rule is that we keep the stable part of the data in relational columns, and we only push the flexible part into one jsonb column.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| lưu nguyên văn bản gốc | stores the original text as it is |
| phải phân tích lại chuỗi đó từ đầu | has to parse that string again from the beginning |
| dưới dạng nhị phân đã được phân tích sẵn | in a binary form that has already been parsed |
| chúng ta đánh index được lên nó | we can build an index on it |
| tốn thêm một chút công lúc ghi | costs a bit more work at write time |
| nó không giữ nguyên thứ tự khoá | it does not keep the original key order |
| dữ liệu bán cấu trúc | semi-structured data |
| cờ tính năng | feature flags |
| chúng ta mất kiểu dữ liệu chặt | we lose strict data types |
| quy tắc thực dụng là | the pragmatic rule is |
| chúng ta chỉ đẩy phần linh hoạt vào | we only push the flexible part into |

**Thuật ngữ cần nhớ**

- dạng nhị phân → **binary form**
- phân tích cú pháp → **parse**
- dữ liệu bán cấu trúc → **semi-structured data**
- cờ tính năng → **feature flag**
- phần thân của sự kiện → **event payload**

---

## Phần 2 — Index GIN cho jsonb và giới hạn của nó

**Tiếng Việt**

Để truy vấn nhanh trên một cột jsonb, chúng ta cần một loại index khác với B-tree, và loại đó là GIN. GIN là một index đảo, nghĩa là nó lập chỉ mục cho từng thành phần bên trong tài liệu chứ nó không lập chỉ mục cho cả giá trị. Nhờ vậy, nó phục vụ rất tốt phép chứa, tức là câu hỏi liệu tài liệu này có chứa cặp khoá và giá trị kia hay không. Chúng ta thường dùng lớp toán tử jsonb_path_ops, vì nó tạo ra một index nhỏ hơn và nhanh hơn cho riêng phép chứa. Nhưng GIN có một giới hạn rõ ràng, vì nó không phục vụ tốt việc lọc theo khoảng và việc sắp xếp như B-tree trên một cột thường. Vì vậy, nếu một trường bên trong jsonb thường xuyên được lọc theo khoảng hoặc được dùng để sắp xếp, chúng ta nên kéo trường đó ra thành một cột riêng.

**English (bám cấu trúc tiếng Việt)**

In order to query quickly on a jsonb column, we need a different kind of index from the B-tree, and that kind is GIN. GIN is an inverted index, which means that it indexes each element inside the document and it does not index the value as a whole. Thanks to that, it serves containment very well, that is the question of whether this document contains that key and value pair. We usually use the jsonb_path_ops operator class, because it produces a smaller and faster index for containment alone. But GIN has one clear limitation, because it does not serve range filtering and sorting as well as a B-tree on an ordinary column. Therefore, if a field inside the jsonb is regularly filtered by range or is used for sorting, we should pull that field out into its own column.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| một loại index khác với B-tree | a different kind of index from the B-tree |
| một index đảo | an inverted index |
| nó lập chỉ mục cho từng thành phần bên trong tài liệu | it indexes each element inside the document |
| nó không lập chỉ mục cho cả giá trị | it does not index the value as a whole |
| nó phục vụ rất tốt phép chứa | it serves containment very well |
| liệu tài liệu này có chứa cặp khoá và giá trị kia hay không | whether this document contains that key and value pair |
| lớp toán tử | the operator class |
| cho riêng phép chứa | for containment alone |
| có một giới hạn rõ ràng | has one clear limitation |
| trên một cột thường | on an ordinary column |
| kéo trường đó ra thành một cột riêng | pull that field out into its own column |

**Thuật ngữ cần nhớ**

- index đảo → **inverted index**
- phép chứa → **containment**
- lớp toán tử → **operator class**
- trường dữ liệu → **field**
- cột thường → **ordinary column**

---

## Phần 3 — MVCC để lại rác, và VACUUM là người dọn

**Tiếng Việt**

Như chúng ta đã nói ở bài trước, MVCC tạo ra một phiên bản mới mỗi lần cập nhật, và phiên bản cũ vẫn nằm lại trong bảng. Những phiên bản cũ mà không còn ai nhìn thấy được gọi là dead tuple, và chúng chính là rác mà MVCC để lại. VACUUM là tiến trình dọn thứ rác đó, và nó thu hồi phần không gian mà dead tuple đang chiếm để bảng tái sử dụng. VACUUM còn cập nhật visibility map, tức là bản đồ đánh dấu những trang mà mọi transaction đều nhìn thấy toàn bộ. Bản đồ này rất quan trọng, vì index-only scan chỉ hoạt động được khi cơ sở dữ liệu biết chắc rằng nó không cần kiểm tra lại bảng. PostgreSQL có autovacuum chạy nền để làm việc này một cách tự động, nhưng chúng ta vẫn phải theo dõi và chỉnh tham số cho những bảng ghi rất nhiều. Nếu autovacuum không theo kịp tốc độ ghi, dead tuple sẽ tích tụ nhanh hơn tốc độ dọn, và hệ thống sẽ trượt dần vào tình trạng phình bảng.

**English (bám cấu trúc tiếng Việt)**

As we said in the previous lesson, MVCC creates a new version on every update, and the old version still stays behind in the table. The old versions that nobody can see any more are called dead tuples, and they are exactly the rubbish that MVCC leaves behind. VACUUM is the process that cleans up that rubbish, and it reclaims the space the dead tuples are occupying so that the table can reuse it. VACUUM also updates the visibility map, that is the map marking the pages that every transaction can see in full. This map is very important, because an index-only scan can only work when the database knows for sure that it does not need to check the table again. PostgreSQL has autovacuum running in the background to do this work automatically, but we still have to watch it and tune the parameters for the tables that are written to very heavily. If autovacuum cannot keep up with the write rate, the dead tuples will pile up faster than the cleaning rate, and the system will slide gradually into a bloated state.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| phiên bản cũ vẫn nằm lại trong bảng | the old version still stays behind in the table |
| mà không còn ai nhìn thấy | that nobody can see any more |
| rác mà MVCC để lại | the rubbish that MVCC leaves behind |
| nó thu hồi phần không gian mà dead tuple đang chiếm | it reclaims the space the dead tuples are occupying |
| để bảng tái sử dụng | so that the table can reuse it |
| bản đồ đánh dấu những trang mà mọi transaction đều nhìn thấy toàn bộ | the map marking the pages that every transaction can see in full |
| biết chắc rằng | knows for sure that |
| chúng ta vẫn phải theo dõi và chỉnh tham số | we still have to watch it and tune the parameters |
| không theo kịp tốc độ ghi | cannot keep up with the write rate |
| sẽ trượt dần vào tình trạng phình bảng | will slide gradually into a bloated state |

**Thuật ngữ cần nhớ**

- phiên bản chết → **dead tuple**
- thu hồi không gian → **reclaim space**
- bản đồ hiển thị → **visibility map**
- dọn rác tự động → **autovacuum**
- chỉnh tham số → **tune the parameters**

---

## Phần 4 — Bloat và cái giá của VACUUM FULL

**Tiếng Việt**

Khi dead tuple tích tụ, bảng và index phình to hơn nhiều so với lượng dữ liệu sống thật, và hiện tượng đó được gọi là bloat. Điều bất ngờ với nhiều người là VACUUM thường không trả không gian đó về cho hệ điều hành. Nó chỉ đánh dấu phần không gian đó là trống để chính bảng đó dùng lại cho các dòng mới. Muốn thật sự thu nhỏ tệp trên đĩa, chúng ta phải chạy VACUUM FULL, và lệnh này viết lại toàn bộ bảng sang một tệp mới. Cái giá là VACUUM FULL đòi một khoá độc quyền trên bảng, vì vậy không ai đọc hay ghi được trong suốt thời gian nó chạy. Vì lý do đó, chúng ta không bao giờ chạy VACUUM FULL vào giờ cao điểm, và với bảng lớn chúng ta nên nghĩ tới việc phân mảnh bảng để xoá cả mảnh cũ thay vì xoá từng dòng.

**English (bám cấu trúc tiếng Việt)**

When dead tuples pile up, the table and the indexes grow far bigger than the amount of live data, and that phenomenon is called bloat. The surprising thing for many people is that VACUUM usually does not give that space back to the operating system. It only marks that space as free so that the same table can reuse it for new rows. If we want to really shrink the file on disk, we have to run VACUUM FULL, and this command rewrites the whole table into a new file. The price is that VACUUM FULL demands an exclusive lock on the table, therefore nobody can read or write during the whole time it runs. For that reason, we never run VACUUM FULL at peak hours, and for a large table we should think about partitioning the table so that we can drop a whole old partition instead of deleting row by row.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| so với lượng dữ liệu sống thật | than the amount of live data |
| hiện tượng đó được gọi là bloat | that phenomenon is called bloat |
| điều bất ngờ với nhiều người là | the surprising thing for many people is that |
| không trả không gian đó về cho hệ điều hành | does not give that space back to the operating system |
| nó chỉ đánh dấu phần không gian đó là trống | it only marks that space as free |
| muốn thật sự thu nhỏ tệp trên đĩa | if we want to really shrink the file on disk |
| viết lại toàn bộ bảng sang một tệp mới | rewrites the whole table into a new file |
| đòi một khoá độc quyền trên bảng | demands an exclusive lock on the table |
| trong suốt thời gian nó chạy | during the whole time it runs |
| vào giờ cao điểm | at peak hours |
| xoá cả mảnh cũ thay vì xoá từng dòng | drop a whole old partition instead of deleting row by row |

**Thuật ngữ cần nhớ**

- phình bảng do rác tích tụ → **bloat**
- dữ liệu sống → **live data**
- khoá độc quyền → **exclusive lock**
- giờ cao điểm → **peak hours**
- phân mảnh bảng → **table partitioning**

---

## Phần 5 — XID wraparound và cơ chế tự bảo vệ

**Tiếng Việt**

Mỗi transaction trong PostgreSQL được đánh một số định danh dài ba mươi hai bit. Vì con số đó có giới hạn, sau khoảng hai tỉ transaction nó sẽ quay vòng trở lại từ đầu. Nếu điều đó xảy ra mà các dòng cũ chưa được đóng băng, cơ sở dữ liệu sẽ không còn phân biệt được đâu là dòng cũ và đâu là dòng mới. Hậu quả sẽ là dữ liệu cũ đột nhiên biến mất khỏi tầm nhìn, và đó là một thảm hoạ không sửa được. Để tự bảo vệ, PostgreSQL theo dõi tuổi của số định danh và nó chuyển cơ sở dữ liệu sang chế độ chỉ đọc trước khi tình huống đó xảy ra. Việc phòng ngừa nằm ở chỗ autovacuum phải chạy đủ để đóng băng các dòng cũ trước khi số định danh cạn. Ở quy mô lớn, chúng ta phải đặt cảnh báo trên tuổi của số định danh, vì phát hiện muộn nghĩa là hệ thống ngừng nhận ghi ngay giữa giờ làm việc.

**English (bám cấu trúc tiếng Việt)**

Every transaction in PostgreSQL is given an identifier that is thirty-two bits long. Because that number has a limit, after about two billion transactions it will wrap around back to the beginning. If that happens while the old rows have not been frozen, the database will no longer be able to tell which row is old and which row is new. The consequence would be that old data suddenly disappears from view, and that is a disaster that cannot be fixed. To protect itself, PostgreSQL tracks the age of the identifier and it switches the database into read-only mode before that situation happens. The prevention lies in the fact that autovacuum has to run enough to freeze the old rows before the identifier runs out. At large scale, we have to set an alert on the age of the identifier, because finding out late means the system stops accepting writes right in the middle of working hours.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| được đánh một số định danh dài ba mươi hai bit | is given an identifier that is thirty-two bits long |
| nó sẽ quay vòng trở lại từ đầu | it will wrap around back to the beginning |
| mà các dòng cũ chưa được đóng băng | while the old rows have not been frozen |
| không còn phân biệt được đâu là dòng cũ và đâu là dòng mới | no longer be able to tell which row is old and which row is new |
| đột nhiên biến mất khỏi tầm nhìn | suddenly disappears from view |
| một thảm hoạ không sửa được | a disaster that cannot be fixed |
| để tự bảo vệ | to protect itself |
| chuyển … sang chế độ chỉ đọc | switches … into read-only mode |
| việc phòng ngừa nằm ở chỗ | the prevention lies in the fact that |
| trước khi số định danh cạn | before the identifier runs out |
| phát hiện muộn nghĩa là | finding out late means |

**Thuật ngữ cần nhớ**

- quay vòng số định danh transaction → **XID wraparound**
- đóng băng dòng cũ → **freeze old rows**
- chế độ chỉ đọc → **read-only mode**
- tuổi của số định danh → **the age of the identifier**
- đặt cảnh báo → **set an alert**

---

## Phần 6 — Nâng max_connections không phải là mở rộng

**Tiếng Việt**

Khi ứng dụng báo lỗi hết kết nối, phản xạ đầu tiên của nhiều người là nâng tham số max_connections lên một nghìn. Đây là một hướng sai, và lý do nằm ở kiến trúc của PostgreSQL. Mỗi kết nối trong PostgreSQL là một tiến trình riêng ở phía máy chủ, và mỗi tiến trình đó tốn bộ nhớ cùng với thời gian chuyển ngữ cảnh của CPU. Khi số kết nối vượt quá khả năng của phần cứng, hệ thống không chạy nhanh hơn mà nó chạy chậm hẳn đi. Cách đúng là chúng ta đặt một trình gom kết nối ở phía trước, ví dụ PgBouncer, và trình đó dồn hàng nghìn kết nối của ứng dụng vào một số ít kết nối thật. Chúng ta sẽ nói kỹ về cách cấu hình trình gom kết nối trong bài về connection pooling.

**English (bám cấu trúc tiếng Việt)**

When the application reports a connection exhaustion error, the first reflex of many people is to raise the max_connections parameter to one thousand. This is the wrong direction, and the reason lies in the architecture of PostgreSQL. Each connection in PostgreSQL is a separate process on the server side, and each of those processes costs memory together with CPU context switching time. When the number of connections goes beyond what the hardware can handle, the system does not run faster but it runs clearly slower. The correct approach is that we put a connection pooler in front, for example PgBouncer, and that pooler funnels thousands of application connections into a small number of real connections. We will talk in detail about how to configure a connection pooler in the lesson on connection pooling.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| khi ứng dụng báo lỗi hết kết nối | when the application reports a connection exhaustion error |
| phản xạ đầu tiên của nhiều người | the first reflex of many people |
| đây là một hướng sai | this is the wrong direction |
| lý do nằm ở kiến trúc của PostgreSQL | the reason lies in the architecture of PostgreSQL |
| một tiến trình riêng ở phía máy chủ | a separate process on the server side |
| thời gian chuyển ngữ cảnh của CPU | CPU context switching time |
| vượt quá khả năng của phần cứng | goes beyond what the hardware can handle |
| nó chạy chậm hẳn đi | it runs clearly slower |
| đặt một trình gom kết nối ở phía trước | put a connection pooler in front |
| dồn hàng nghìn kết nối vào một số ít kết nối thật | funnels thousands of connections into a small number of real connections |

**Thuật ngữ cần nhớ**

- tiến trình phía máy chủ → **backend process**
- chuyển ngữ cảnh → **context switching**
- trình gom kết nối → **connection pooler**
- hết kết nối → **connection exhaustion**
- phần cứng → **hardware**

---

## Phần 7 — Thống kê cũ làm cho bộ lập kế hoạch bị mù

**Tiếng Việt**

Bộ lập kế hoạch của PostgreSQL không nhìn thấy dữ liệu thật khi nó chọn kế hoạch, mà nó chỉ nhìn vào thống kê. Thống kê đó mô tả số dòng, số giá trị khác nhau, và phân bố giá trị trong từng cột. Dựa trên các con số này, bộ lập kế hoạch ước lượng mỗi bước sẽ trả về bao nhiêu dòng, rồi nó chọn kế hoạch rẻ nhất. Nếu thống kê đã cũ hoặc bị lệch, ước lượng sẽ sai, và bộ lập kế hoạch sẽ chọn một kế hoạch tồi mà nó tưởng là tốt. Đây chính là lý do một truy vấn đang chạy nhanh bỗng nhiên chậm hẳn ngay sau khi chúng ta nạp thêm mười triệu dòng. Cách xử lý là chúng ta chạy lệnh ANALYZE trên bảng đó để cập nhật thống kê, và PostgreSQL cũng có autoanalyze chạy nền. Vì vậy, mỗi khi một truy vấn chậm đột ngột mà không có ai đổi code, nghi phạm đầu tiên của chúng ta luôn là thống kê cũ.

**English (bám cấu trúc tiếng Việt)**

The PostgreSQL planner does not see the real data when it chooses a plan, but it only looks at the statistics. Those statistics describe the number of rows, the number of distinct values, and the distribution of values in each column. Based on these numbers, the planner estimates how many rows each step will return, then it picks the cheapest plan. If the statistics are stale or skewed, the estimate will be wrong, and the planner will pick a bad plan that it believes is a good one. This is exactly the reason why a query that was running fast suddenly becomes much slower right after we load ten million more rows. The way to handle it is that we run the ANALYZE command on that table to update the statistics, and PostgreSQL also has autoanalyze running in the background. Therefore, whenever a query slows down suddenly and nobody has changed the code, our first suspect is always stale statistics.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| không nhìn thấy dữ liệu thật khi nó chọn kế hoạch | does not see the real data when it chooses a plan |
| phân bố giá trị trong từng cột | the distribution of values in each column |
| ước lượng mỗi bước sẽ trả về bao nhiêu dòng | estimates how many rows each step will return |
| nó chọn kế hoạch rẻ nhất | it picks the cheapest plan |
| nếu thống kê đã cũ hoặc bị lệch | if the statistics are stale or skewed |
| một kế hoạch tồi mà nó tưởng là tốt | a bad plan that it believes is a good one |
| bỗng nhiên chậm hẳn | suddenly becomes much slower |
| ngay sau khi chúng ta nạp thêm mười triệu dòng | right after we load ten million more rows |
| mà không có ai đổi code | and nobody has changed the code |
| nghi phạm đầu tiên của chúng ta | our first suspect |

**Thuật ngữ cần nhớ**

- thống kê bảng → **table statistics**
- phân bố giá trị → **value distribution**
- ước lượng → **estimate**
- kế hoạch thực thi → **execution plan**
- bị lệch → **skewed**

---

## Phần 8 — Chọn phiên bản PostgreSQL cho một dự án mới

**Tiếng Việt**

Việc chọn phiên bản PostgreSQL cho một dự án mới cũng là một câu hỏi phỏng vấn, vì nó cho thấy chúng ta có theo dõi vòng đời sản phẩm hay không. Mỗi phiên bản chính của PostgreSQL được hỗ trợ trong năm năm, và sau đó nó ngừng nhận cả bản vá bảo mật. Tính đến giữa năm hai nghìn không trăm hai mươi sáu, lựa chọn an toàn cho một dự án mới là phiên bản mười bảy hoặc phiên bản mười tám. Chúng ta nên tránh phiên bản mười bốn vì nó sẽ hết vòng đời vào tháng mười một năm nay, và chúng ta phải tránh mọi phiên bản từ mười ba trở xuống vì chúng đã hết vòng đời. Chúng ta cũng không bao giờ chạy một bản beta trên production, dù bản đó có tính năng mà chúng ta rất muốn dùng. Điều quan trọng cuối cùng là chúng ta luôn kiểm tra lại trên trang chủ của PostgreSQL, vì lịch vòng đời thay đổi theo thời gian và câu trả lời hôm nay sẽ cũ đi sau vài tháng.

**English (bám cấu trúc tiếng Việt)**

Choosing the PostgreSQL version for a new project is also an interview question, because it shows whether we follow the product lifecycle or not. Each major version of PostgreSQL is supported for five years, and after that it stops receiving even security patches. As of the middle of the year two thousand and twenty-six, the safe choice for a new project is version seventeen or version eighteen. We should avoid version fourteen because it reaches end of life in November this year, and we must avoid every version from thirteen downwards because they have already reached end of life. We also never run a beta version in production, even if that version has a feature that we really want to use. The last important point is that we always check again on the PostgreSQL website, because the lifecycle schedule changes over time and today's answer will be out of date in a few months.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| nó cho thấy chúng ta có theo dõi … hay không | it shows whether we follow … or not |
| mỗi phiên bản chính | each major version |
| nó ngừng nhận cả bản vá bảo mật | it stops receiving even security patches |
| tính đến giữa năm … | as of the middle of the year … |
| nó sẽ hết vòng đời vào tháng mười một năm nay | it reaches end of life in November this year |
| mọi phiên bản từ mười ba trở xuống | every version from thirteen downwards |
| dù bản đó có tính năng mà chúng ta rất muốn dùng | even if that version has a feature that we really want to use |
| chúng ta luôn kiểm tra lại trên trang chủ | we always check again on the website |
| lịch vòng đời thay đổi theo thời gian | the lifecycle schedule changes over time |
| câu trả lời hôm nay sẽ cũ đi sau vài tháng | today's answer will be out of date in a few months |

**Thuật ngữ cần nhớ**

- phiên bản chính → **major version**
- vòng đời sản phẩm → **product lifecycle**
- hết vòng đời → **end of life (EOL)**
- bản vá bảo mật → **security patch**
- bản thử nghiệm → **beta version**

---

## Mô hình ghi nhớ

**Tiếng Việt**

MVCC tạo ra rác dưới dạng dead tuple nên VACUUM phải dọn, và mỗi kết nối là một tiến trình nên chúng ta thêm trình gom kết nối chứ chúng ta không nâng max_connections. Khi một truy vấn chậm đột ngột, chúng ta chạy ANALYZE trước, vì thống kê cũ làm cho bộ lập kế hoạch bị mù.

**English (bám cấu trúc tiếng Việt)**

MVCC creates rubbish in the form of dead tuples so VACUUM has to clean it up, and each connection is a process so we add a connection pooler and we do not raise max_connections. When a query slows down suddenly, we run ANALYZE first, because stale statistics make the planner blind.

---

## Bảng thuật ngữ tổng hợp

| Tiếng Việt | Tiếng Anh | Ghi chú phát âm |
|---|---|---|
| dạng nhị phân | binary form | **BAI**-nơ-ri — âm đầu là "bai" |
| phân tích cú pháp | parse | /pɑːz/ — đuôi /z/, không đọc "pat" |
| dữ liệu bán cấu trúc | semi-structured data | *structured* — cụm /str/ đọc liền |
| cờ tính năng | feature flag | **FEE**-chơ |
| phần thân của sự kiện | event payload | i-**VENT** — trọng âm 2 |
| lược đồ quan hệ | relational schema | *schema* /ˈskiːmə/ — "**SKII**-mờ" |
| index đảo | inverted index | in-**VER**-tid |
| phép chứa | containment | cơn-**TEIN**-mơnt — trọng âm 2 |
| lớp toán tử | operator class | **O**-pơ-rây-tơ |
| trường dữ liệu | field | /fiːld/ — nguyên âm dài |
| cột thường | ordinary column | **OR**-đi-nơ-ri; *column* có **n** cuối câm |
| phiên bản chết | dead tuple | *tuple* Anh /ˈtjuːpl/ — "**TIU**-pl", không đọc "tớp-lê" |
| thu hồi không gian | reclaim space | ri-**CLEIM** |
| bản đồ hiển thị | visibility map | vi-zơ-**BI**-lơ-ti |
| dọn rác tự động | autovacuum | *vacuum* **VA**-kiu-ơm — trọng âm 1 |
| chỉnh tham số | tune the parameters | pơ-**RA**-mi-tơ — trọng âm 2 |
| quét chỉ dùng index | index-only scan | |
| phình bảng do rác tích tụ | bloat | /bləʊt/ — "blợut", vần với "boat" |
| dữ liệu sống | live data | tính từ *live* /laɪv/ — "laiv" |
| khoá độc quyền | exclusive lock | ex-**CLU**-siv |
| viết lại toàn bộ | rewrite | *write* — chữ **w** câm |
| giờ cao điểm | peak hours | *hours* — chữ **h** câm: "au-ơz" |
| phân mảnh bảng | table partitioning | par-**TI**-shơn-ing |
| số định danh transaction | transaction ID (XID) | |
| quay vòng | wrap around | *wrap* — chữ **w** câm: "ræp" |
| đóng băng dòng cũ | freeze old rows | /friːz/ — đuôi /z/ |
| chế độ chỉ đọc | read-only mode | |
| thảm hoạ | disaster | Anh đi-**ZAS**-tơ — trọng âm 2 |
| đặt cảnh báo | set an alert | ơ-**LERT** |
| tiến trình phía máy chủ | backend process | Anh **PROU**-ses |
| chuyển ngữ cảnh | context switching | **CON**-text — trọng âm 1 |
| trình gom kết nối | connection pooler | |
| hết kết nối | connection exhaustion | ig-**ZOS**-chơn — "x" đọc /gz/ |
| phần cứng | hardware | |
| kiến trúc | architecture | **AR**-ki-tek-chơ — "ch" trong *arch* đọc /k/ |
| thống kê bảng | table statistics | stơ-**TIS**-tics — trọng âm 2 |
| phân bố giá trị | value distribution | dis-tri-**BIU**-shơn |
| ước lượng | estimate | động từ **ES**-ti-meit; danh từ **ES**-ti-mơt |
| kế hoạch thực thi | execution plan | ek-si-**KIU**-shơn |
| bị lệch | skewed | /skjuːd/ — "skiu-đ" |
| cũ, không còn tươi | stale | /steɪl/ — "stêi-l" |
| nghi phạm | suspect | danh từ **SUS**-pect; động từ sơs-**PECT** |
| phiên bản chính | major version | **MEI**-jơ |
| vòng đời sản phẩm | product lifecycle | |
| hết vòng đời | end of life (EOL) | |
| bản vá bảo mật | security patch | si-**KIU**-rơ-ti |
| bản thử nghiệm | beta version | Anh **BEE**-tơ, không đọc "bê-ta" |

---

## Bài nói

> **Cách luyện:** bật ghi âm, nói **60–90 giây** cho mỗi đề, **không nhìn tài liệu**. Mỗi bài nói phải dùng **ít nhất 6 thuật ngữ** trong bảng trên. Nghe lại bản ghi một lần, đánh dấu chỗ ngập ngừng, rồi nói lại đề đó lần thứ hai.

1. Explain to a junior engineer why VACUUM exists in PostgreSQL and what would happen to a busy table if autovacuum stopped running.

2. A colleague says: "We keep hitting connection limits, so let us set max_connections to one thousand." Explain why you would push back and what you would propose instead.

3. Someone on your team wants to store the whole product model in a single jsonb column "so the schema never blocks us again". Explain where you agree with them and where you would draw the line.

4. Describe what happens between running VACUUM and running VACUUM FULL on a bloated table, and explain why you would not run the second one during the day.

5. A query that ran in twenty milliseconds yesterday now takes four seconds, and nobody has deployed any code. Walk your team through your first three checks.

6. Explain what XID wraparound is and why PostgreSQL would rather stop accepting writes than let it happen.

7. When would you choose a GIN index on a jsonb column over pulling the field out into its own B-tree indexed column?
