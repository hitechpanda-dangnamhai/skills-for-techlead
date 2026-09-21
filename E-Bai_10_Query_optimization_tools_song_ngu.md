# Bài 10 — Query optimization tools
### Bản song ngữ: tiếng Anh bám cấu trúc tiếng Việt

> **Cách dùng bài này:** đọc đoạn tiếng Việt trước, tự dịch trong đầu, rồi mới so với đoạn tiếng Anh bên dưới. Chỗ nào lệch thì xem bảng ánh xạ cụm khó. Cuối cùng, **đọc to đoạn tiếng Anh hai lần** — một lần chậm để đúng âm, một lần với tốc độ nói thật.

---

## Phần 1 — Đo trước, sửa sau

**Tiếng Việt**

Khi một truy vấn chậm, việc đầu tiên chúng ta làm không phải là thêm index, mà là đo. Quy trình chuẩn gồm bốn bước, và chúng ta luôn đi theo đúng thứ tự đó. Bước thứ nhất là chúng ta tìm ra truy vấn nào đang thật sự tốn thời gian, bằng pg_stat_statements hoặc bằng nhật ký truy vấn chậm. Bước thứ hai là chúng ta chạy EXPLAIN ANALYZE trên truy vấn đó để lấy kế hoạch thực thi thật. Bước thứ ba là chúng ta đọc kế hoạch đó và tìm bước tốn nhiều thời gian nhất, thường là một phép quét, một phép nối hoặc một phép sắp xếp. Bước thứ tư là chúng ta thêm index hoặc viết lại truy vấn, rồi chúng ta đo lại để chứng minh rằng thay đổi đó có tác dụng. Nếu chúng ta bỏ qua ba bước đầu và nhảy thẳng vào bước cuối, chúng ta sẽ tạo ra những index không ai dùng và chúng ta sẽ trả thuế ghi cho chúng mãi mãi.

**English (bám cấu trúc tiếng Việt)**

When a query is slow, the first thing we do is not to add an index, but to measure. The standard process has four steps, and we always go through them in exactly that order. The first step is that we find out which query is really eating the time, using pg_stat_statements or using the slow query log. The second step is that we run EXPLAIN ANALYZE on that query in order to get the real execution plan. The third step is that we read that plan and look for the step that costs the most time, usually a scan, a join or a sort. The fourth step is that we add an index or rewrite the query, then we measure again in order to prove that the change actually works. If we skip the first three steps and jump straight to the last one, we will create indexes that nobody uses and we will pay the write tax for them forever.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| việc đầu tiên chúng ta làm không phải là …, mà là | the first thing we do is not to …, but to |
| chúng ta luôn đi theo đúng thứ tự đó | we always go through them in exactly that order |
| truy vấn nào đang thật sự tốn thời gian | which query is really eating the time |
| nhật ký truy vấn chậm | the slow query log |
| để lấy kế hoạch thực thi thật | in order to get the real execution plan |
| tìm bước tốn nhiều thời gian nhất | look for the step that costs the most time |
| chúng ta đo lại để chứng minh rằng | we measure again in order to prove that |
| nếu chúng ta bỏ qua ba bước đầu | if we skip the first three steps |
| nhảy thẳng vào bước cuối | jump straight to the last one |
| trả thuế ghi cho chúng mãi mãi | pay the write tax for them forever |

**Thuật ngữ cần nhớ**

- kế hoạch thực thi → **execution plan**
- nhật ký truy vấn chậm → **slow query log**
- viết lại truy vấn → **rewrite the query**
- đo lại → **measure again**
- thuế ghi → **write tax**

---

## Phần 2 — EXPLAIN chỉ ước lượng, EXPLAIN ANALYZE chạy thật

**Tiếng Việt**

Lệnh EXPLAIN và lệnh EXPLAIN ANALYZE nghe giống nhau nhưng chúng làm hai việc khác nhau. Lệnh EXPLAIN chỉ hỏi bộ lập kế hoạch xem nó định làm gì, và nó không chạy truy vấn. Vì vậy, mọi con số mà EXPLAIN đưa ra đều là ước lượng dựa trên thống kê. Lệnh EXPLAIN ANALYZE thì chạy truy vấn thật, và nó báo về số dòng thật cùng với thời gian thật của từng bước. Chính vì nó chạy thật, chúng ta phải rất cẩn thận khi dùng nó với một câu INSERT, UPDATE hoặc DELETE. Cách an toàn là chúng ta bọc câu lệnh trong một transaction rồi chúng ta rollback, nhờ đó chúng ta lấy được kế hoạch thật mà dữ liệu không hề bị đổi.

**English (bám cấu trúc tiếng Việt)**

The EXPLAIN command and the EXPLAIN ANALYZE command sound the same but they do two different things. The EXPLAIN command only asks the planner what it intends to do, and it does not run the query. Therefore, every number that EXPLAIN gives us is an estimate based on the statistics. The EXPLAIN ANALYZE command does run the query for real, and it reports the actual row counts together with the actual time of each step. Precisely because it runs for real, we have to be very careful when we use it with an INSERT, UPDATE or DELETE statement. The safe way is that we wrap the statement in a transaction and then we roll back, thanks to which we get the real plan while the data is not changed at all.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| nghe giống nhau nhưng chúng làm hai việc khác nhau | sound the same but they do two different things |
| chỉ hỏi bộ lập kế hoạch xem nó định làm gì | only asks the planner what it intends to do |
| mọi con số mà EXPLAIN đưa ra | every number that EXPLAIN gives us |
| là ước lượng dựa trên thống kê | is an estimate based on the statistics |
| thì chạy truy vấn thật | does run the query for real |
| số dòng thật cùng với thời gian thật của từng bước | the actual row counts together with the actual time of each step |
| chính vì nó chạy thật | precisely because it runs for real |
| chúng ta bọc câu lệnh trong một transaction | we wrap the statement in a transaction |
| mà dữ liệu không hề bị đổi | while the data is not changed at all |

**Thuật ngữ cần nhớ**

- ước lượng → **estimate**
- số liệu thật đo được → **actual**
- bọc trong transaction → **wrap in a transaction**
- quay lui → **roll back**
- bộ lập kế hoạch → **the planner**

---

## Phần 3 — Bốn kiểu quét trong kế hoạch thực thi

**Tiếng Việt**

Trong một kế hoạch thực thi, chúng ta sẽ gặp bốn kiểu quét chính, và mỗi kiểu nói cho chúng ta một điều khác nhau. Seq Scan nghĩa là cơ sở dữ liệu đọc tuần tự cả bảng, và điều này hợp lý khi bảng nhỏ hoặc khi truy vấn cần phần lớn số dòng. Index Scan nghĩa là cơ sở dữ liệu tra index rồi lấy từng dòng tương ứng từ heap, và nó hợp khi số dòng cần lấy rất ít. Bitmap Index Scan nằm ở giữa hai kiểu trên, vì cơ sở dữ liệu gom trước danh sách các trang cần đọc rồi mới đọc heap một lượt theo thứ tự. Kiểu này cũng chính là cách để cơ sở dữ liệu kết hợp nhiều index cho cùng một truy vấn. Index Only Scan là kiểu tốt nhất, vì mọi cột cần thiết đã nằm sẵn trong index nên cơ sở dữ liệu không phải chạm vào heap. Chúng ta cần nhớ rằng Index Only Scan chỉ hoạt động khi visibility map đủ mới, và đó là một lý do nữa để autovacuum chạy đều đặn.

**English (bám cấu trúc tiếng Việt)**

In an execution plan, we will meet four main scan types, and each type tells us something different. Seq Scan means that the database reads the whole table sequentially, and this makes sense when the table is small or when the query needs most of the rows. Index Scan means that the database looks up the index and then fetches each matching row from the heap, and it fits when the number of rows to fetch is very small. Bitmap Index Scan sits between the two types above, because the database first collects the list of pages it needs to read and only then reads the heap in one pass in order. This type is also exactly how the database combines several indexes for the same query. Index Only Scan is the best type, because all the needed columns already sit in the index so the database does not have to touch the heap. We need to remember that Index Only Scan only works when the visibility map is fresh enough, and that is one more reason for autovacuum to run regularly.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| mỗi kiểu nói cho chúng ta một điều khác nhau | each type tells us something different |
| đọc tuần tự cả bảng | reads the whole table sequentially |
| điều này hợp lý khi | this makes sense when |
| lấy từng dòng tương ứng từ heap | fetches each matching row from the heap |
| nằm ở giữa hai kiểu trên | sits between the two types above |
| gom trước danh sách các trang cần đọc | first collects the list of pages it needs to read |
| rồi mới đọc heap một lượt theo thứ tự | and only then reads the heap in one pass in order |
| kết hợp nhiều index cho cùng một truy vấn | combines several indexes for the same query |
| không phải chạm vào heap | does not have to touch the heap |
| khi visibility map đủ mới | when the visibility map is fresh enough |

**Thuật ngữ cần nhớ**

- quét tuần tự → **sequential scan (Seq Scan)**
- quét theo index → **index scan**
- quét theo bản đồ bit → **bitmap index scan**
- quét chỉ dùng index → **index-only scan**
- một lượt đọc → **one pass**

---

## Phần 4 — Khi số dòng ước lượng lệch xa số dòng thật

**Tiếng Việt**

Điều quan trọng nhất khi đọc một kế hoạch là chúng ta so số dòng ước lượng với số dòng thật ở từng bước. Nếu bộ lập kế hoạch ước lượng hai dòng nhưng bước đó thực tế trả về chín trăm nghìn dòng, đó là một dấu hiệu rất xấu. Lý do là bộ lập kế hoạch chọn thuật toán dựa trên con số ước lượng, vì vậy một ước lượng sai sẽ kéo theo một lựa chọn sai. Ví dụ, nó sẽ chọn nested loop vì nó tưởng chỉ có vài dòng, trong khi lẽ ra nó phải chọn hash join cho chín trăm nghìn dòng. Nguyên nhân phổ biến nhất của sai lệch này là thống kê đã cũ, và cách xử lý đầu tiên là chúng ta chạy ANALYZE. Nguyên nhân thứ hai là các cột có tương quan với nhau, ví dụ thành phố và mã bưu chính, vì bộ lập kế hoạch mặc định coi chúng độc lập và nhân xác suất của chúng với nhau.

**English (bám cấu trúc tiếng Việt)**

The most important thing when we read a plan is that we compare the estimated row count with the actual row count at each step. If the planner estimates two rows but that step actually returns nine hundred thousand rows, that is a very bad sign. The reason is that the planner picks its algorithm based on the estimated number, therefore a wrong estimate will drag a wrong choice along with it. For example, it will pick a nested loop because it thinks there are only a few rows, while it should have picked a hash join for nine hundred thousand rows. The most common cause of this gap is that the statistics are stale, and the first thing we do about it is that we run ANALYZE. The second cause is that the columns are correlated with each other, for example the city and the postcode, because by default the planner treats them as independent and multiplies their probabilities together.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| chúng ta so số dòng ước lượng với số dòng thật | we compare the estimated row count with the actual row count |
| đó là một dấu hiệu rất xấu | that is a very bad sign |
| chọn thuật toán dựa trên con số ước lượng | picks its algorithm based on the estimated number |
| sẽ kéo theo một lựa chọn sai | will drag a wrong choice along with it |
| vì nó tưởng chỉ có vài dòng | because it thinks there are only a few rows |
| trong khi lẽ ra nó phải chọn | while it should have picked |
| nguyên nhân phổ biến nhất của sai lệch này | the most common cause of this gap |
| các cột có tương quan với nhau | the columns are correlated with each other |
| mặc định coi chúng độc lập | by default treats them as independent |
| nhân xác suất của chúng với nhau | multiplies their probabilities together |

**Thuật ngữ cần nhớ**

- số dòng ước lượng / số dòng thật → **estimated rows / actual rows**
- khoảng lệch → **gap**
- tương quan giữa các cột → **column correlation**
- độc lập → **independent**
- xác suất → **probability**

---

## Phần 5 — Ba thuật toán nối và ý nghĩa của chúng

**Tiếng Việt**

Bộ lập kế hoạch có ba thuật toán nối, và nó chọn theo kích thước dữ liệu, theo index sẵn có và theo việc dữ liệu đã được sắp xếp hay chưa. Nested loop duyệt từng dòng của bên ngoài rồi tra bên trong cho mỗi dòng, vì vậy nó rất tốt khi một bên nhỏ và bên kia có index. Nhưng nếu bên ngoài lớn, nested loop trở thành thảm hoạ, vì chi phí của nó tăng theo tích của hai bên. Hash join dựng một bảng băm từ bên nhỏ rồi quét bên lớn đúng một lượt, và nó là lựa chọn tốt khi cả hai bên đều lớn và chưa được sắp xếp. Merge join đi song song trên hai bên đã sắp xếp, vì vậy nó rất rẻ khi dữ liệu vốn đã có thứ tự nhờ index. Chúng ta không chọn thuật toán thay cho bộ lập kế hoạch, nhưng khi nhìn thấy một nested loop chạy trên hàng triệu dòng, chúng ta biết ngay rằng ước lượng đã sai ở đâu đó.

**English (bám cấu trúc tiếng Việt)**

The planner has three join algorithms, and it chooses according to the size of the data, according to the available indexes and according to whether the data is already sorted or not. A nested loop walks through each row of the outer side and then looks up the inner side for each row, therefore it is very good when one side is small and the other side has an index. But if the outer side is large, a nested loop becomes a disaster, because its cost grows with the product of the two sides. A hash join builds a hash table from the small side and then scans the large side exactly once, and it is a good choice when both sides are large and not sorted. A merge join walks in parallel along two sides that are already sorted, therefore it is very cheap when the data already has an order thanks to an index. We do not choose the algorithm on behalf of the planner, but when we see a nested loop running over millions of rows, we know immediately that an estimate has gone wrong somewhere.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| theo việc dữ liệu đã được sắp xếp hay chưa | according to whether the data is already sorted or not |
| duyệt từng dòng của bên ngoài | walks through each row of the outer side |
| rồi tra bên trong cho mỗi dòng | and then looks up the inner side for each row |
| nested loop trở thành thảm hoạ | a nested loop becomes a disaster |
| chi phí của nó tăng theo tích của hai bên | its cost grows with the product of the two sides |
| dựng một bảng băm từ bên nhỏ | builds a hash table from the small side |
| quét bên lớn đúng một lượt | scans the large side exactly once |
| đi song song trên hai bên đã sắp xếp | walks in parallel along two sides that are already sorted |
| vốn đã có thứ tự nhờ index | already has an order thanks to an index |
| chúng ta không chọn thuật toán thay cho bộ lập kế hoạch | we do not choose the algorithm on behalf of the planner |
| ước lượng đã sai ở đâu đó | an estimate has gone wrong somewhere |

**Thuật ngữ cần nhớ**

- thuật toán nối → **join algorithm**
- vòng lặp lồng nhau → **nested loop**
- nối bằng bảng băm → **hash join**
- nối trộn hai bên đã sắp xếp → **merge join**
- bên ngoài / bên trong → **the outer side / the inner side**

---

## Phần 6 — pg_stat_statements: nhìn tổng thời gian, không nhìn câu chậm nhất

**Tiếng Việt**

Công cụ pg_stat_statements gom các truy vấn theo mẫu, nghĩa là những câu chỉ khác nhau ở giá trị tham số sẽ được tính chung vào một dòng. Với mỗi mẫu, nó cho chúng ta số lần gọi, tổng thời gian thực thi và thời gian trung bình. Sai lầm phổ biến là chúng ta sắp xếp theo thời gian trung bình rồi đi sửa câu chậm nhất. Cách đúng là chúng ta sắp xếp theo tổng thời gian, vì thủ phạm thật của tải hệ thống thường không phải là câu chậm nhất. Một câu chạy hết năm mili giây nhưng được gọi một triệu lần sẽ tốn nhiều thời gian hơn hẳn một câu chạy hết ba giây nhưng chỉ được gọi một lần. Vì vậy, chúng ta luôn hỏi câu nào đang ăn nhiều thời gian nhất của cả cơ sở dữ liệu, chứ chúng ta không hỏi câu nào chậm nhất.

**English (bám cấu trúc tiếng Việt)**

The pg_stat_statements tool groups the queries by shape, which means that statements differing only in the parameter values are counted together into one row. For each shape, it gives us the call count, the total execution time and the mean time. The common mistake is that we sort by the mean time and then go and fix the slowest statement. The correct way is that we sort by the total time, because the real culprit behind the system load is usually not the slowest statement. A statement that takes five milliseconds but is called one million times will cost far more time than a statement that takes three seconds but is called only once. Therefore, we always ask which statement is eating the most time of the whole database, and we do not ask which statement is the slowest.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| gom các truy vấn theo mẫu | groups the queries by shape |
| chỉ khác nhau ở giá trị tham số | differing only in the parameter values |
| sẽ được tính chung vào một dòng | are counted together into one row |
| số lần gọi | the call count |
| tổng thời gian thực thi | the total execution time |
| rồi đi sửa câu chậm nhất | and then go and fix the slowest statement |
| thủ phạm thật của tải hệ thống | the real culprit behind the system load |
| sẽ tốn nhiều thời gian hơn hẳn | will cost far more time than |
| câu nào đang ăn nhiều thời gian nhất của cả cơ sở dữ liệu | which statement is eating the most time of the whole database |

**Thuật ngữ cần nhớ**

- mẫu câu truy vấn → **query shape**
- số lần gọi → **call count**
- tổng thời gian thực thi → **total execution time**
- thời gian trung bình → **mean time**
- thủ phạm → **culprit**

---

## Phần 7 — Truy vấn bỗng chậm dù không ai đổi gì

**Tiếng Việt**

Một tình huống kinh điển trong phỏng vấn là truy vấn bỗng nhiên chậm dù không ai đổi code và dữ liệu cũng không tăng đột biến. Nghi phạm thứ nhất là thống kê đã cũ, vì nó khiến bộ lập kế hoạch đổi sang một kế hoạch tồi hơn. Nghi phạm thứ hai là bloat, vì bảng và index phình lên khiến mỗi lần đọc phải chạm vào nhiều trang hơn. Nghi phạm thứ ba là kế hoạch bị lật, nghĩa là dữ liệu lớn dần cho tới lúc bộ lập kế hoạch thấy quét tuần tự rẻ hơn dùng index. Nghi phạm thứ tư là bộ nhớ đệm bị nguội sau một lần khởi động lại, vì lúc đó mọi lần đọc đều phải xuống tận đĩa. Nghi phạm thứ năm là tranh chấp khoá, nghĩa là truy vấn không hề chậm mà nó chỉ đang chờ một transaction khác. Chúng ta xác minh từng nghi phạm bằng EXPLAIN và bằng các khung nhìn thống kê, chứ chúng ta không sửa mò theo linh cảm.

**English (bám cấu trúc tiếng Việt)**

A classic interview situation is a query that suddenly becomes slow although nobody has changed the code and the data has not grown suddenly either. The first suspect is stale statistics, because they make the planner switch to a worse plan. The second suspect is bloat, because the table and the indexes have grown fat so each read has to touch more pages. The third suspect is a plan flip, which means that the data grew until the planner found a sequential scan cheaper than using the index. The fourth suspect is a cold cache after a restart, because at that point every read has to go all the way down to disk. The fifth suspect is lock contention, which means that the query is not slow at all but it is only waiting for another transaction. We verify each suspect with EXPLAIN and with the statistics views, and we do not fix things blindly by gut feeling.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| dù không ai đổi code | although nobody has changed the code |
| dữ liệu cũng không tăng đột biến | the data has not grown suddenly either |
| nó khiến bộ lập kế hoạch đổi sang một kế hoạch tồi hơn | they make the planner switch to a worse plan |
| mỗi lần đọc phải chạm vào nhiều trang hơn | each read has to touch more pages |
| kế hoạch bị lật | a plan flip |
| cho tới lúc bộ lập kế hoạch thấy … rẻ hơn | until the planner found … cheaper |
| bộ nhớ đệm bị nguội sau một lần khởi động lại | a cold cache after a restart |
| mọi lần đọc đều phải xuống tận đĩa | every read has to go all the way down to disk |
| truy vấn không hề chậm mà nó chỉ đang chờ | the query is not slow at all but it is only waiting |
| chúng ta không sửa mò theo linh cảm | we do not fix things blindly by gut feeling |

**Thuật ngữ cần nhớ**

- nghi phạm → **suspect**
- kế hoạch bị lật → **plan flip**
- bộ nhớ đệm nguội → **cold cache**
- tranh chấp khoá → **lock contention**
- khung nhìn thống kê → **statistics view**

---

## Phần 8 — Bộ công cụ thường trực trên production

**Tiếng Việt**

Ở production, chúng ta không nên chờ tới lúc có sự cố rồi mới đi bật công cụ đo. Cấu hình tối thiểu gồm pg_stat_statements để nhìn toàn cảnh, và auto_explain để tự ghi lại kế hoạch của những câu vượt quá một ngưỡng thời gian. Chúng ta cũng nên thêm tuỳ chọn BUFFERS khi chạy EXPLAIN, vì nó cho biết truy vấn đọc bao nhiêu trang từ bộ nhớ đệm và bao nhiêu trang từ đĩa. Ngoài ra, chúng ta dựng một bảng theo dõi các truy vấn chậm để cả đội cùng nhìn thấy xu hướng theo thời gian. Điều quan trọng cuối cùng là sau mỗi lần sửa, chúng ta phải đo lại và chúng ta ghi con số trước và sau vào mô tả của pull request. Nhờ thói quen đó, đội của chúng ta tranh luận bằng số liệu chứ không tranh luận bằng cảm giác.

**English (bám cấu trúc tiếng Việt)**

In production, we should not wait until there is an incident before we go and turn on the measuring tools. The minimum configuration includes pg_stat_statements to see the whole picture, and auto_explain to record automatically the plan of any statement that goes over a time threshold. We should also add the BUFFERS option when we run EXPLAIN, because it tells us how many pages the query reads from the cache and how many pages it reads from disk. In addition, we build a dashboard tracking the slow queries so that the whole team can see the trend over time. The last important point is that after every fix, we have to measure again and we write the before and after numbers into the description of the pull request. Thanks to that habit, our team argues with numbers and does not argue with feelings.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| chờ tới lúc có sự cố rồi mới đi bật công cụ đo | wait until there is an incident before we go and turn on the measuring tools |
| để nhìn toàn cảnh | to see the whole picture |
| tự ghi lại kế hoạch của những câu vượt quá một ngưỡng thời gian | record automatically the plan of any statement that goes over a time threshold |
| vì nó cho biết | because it tells us |
| chúng ta dựng một bảng theo dõi các truy vấn chậm | we build a dashboard tracking the slow queries |
| để cả đội cùng nhìn thấy xu hướng theo thời gian | so that the whole team can see the trend over time |
| con số trước và sau | the before and after numbers |
| vào mô tả của pull request | into the description of the pull request |
| tranh luận bằng số liệu chứ không tranh luận bằng cảm giác | argues with numbers and does not argue with feelings |

**Thuật ngữ cần nhớ**

- ngưỡng thời gian → **time threshold**
- bảng theo dõi → **dashboard**
- xu hướng theo thời gian → **the trend over time**
- con số trước và sau → **before and after numbers**
- sự cố → **incident**

---

## Mô hình ghi nhớ

**Tiếng Việt**

Chúng ta đo trước rồi mới sửa, vì vậy chúng ta dùng pg_stat_statements để tìm thủ phạm theo tổng thời gian, rồi chúng ta dùng EXPLAIN ANALYZE để lấy kế hoạch thật. Khi số dòng ước lượng lệch xa số dòng thật, nghi phạm đầu tiên của chúng ta là thống kê cũ.

**English (bám cấu trúc tiếng Việt)**

We measure first and only then fix, therefore we use pg_stat_statements to find the culprit by total time, then we use EXPLAIN ANALYZE to get the real plan. When the estimated row count is far off the actual row count, our first suspect is stale statistics.

---

## Bảng thuật ngữ tổng hợp

| Tiếng Việt | Tiếng Anh | Ghi chú phát âm |
|---|---|---|
| kế hoạch thực thi | execution plan | ek-si-**KIU**-shơn |
| nhật ký truy vấn chậm | slow query log | *query* Anh /ˈkwɪəri/ — "**KWIƠ**-ri" |
| viết lại truy vấn | rewrite the query | *write* — chữ **w** câm |
| thuế ghi | write tax | |
| ước lượng | estimate | động từ **ES**-ti-meit; danh từ **ES**-ti-mơt |
| số liệu thật đo được | actual | **AC**-chu-ơl — "tu" đọc /tʃ/ |
| bọc trong transaction | wrap in a transaction | *wrap* — chữ **w** câm: "ræp" |
| quay lui | roll back | |
| bộ lập kế hoạch | the planner | |
| quét tuần tự | sequential scan | si-**KWEN**-shơl |
| quét theo index | index scan | *scan* — đuôi /-n/ đừng nuốt |
| quét theo bản đồ bit | bitmap index scan | |
| quét chỉ dùng index | index-only scan | |
| vùng lưu dòng | heap | /hiːp/ — âm **h** phải bật |
| bản đồ hiển thị | visibility map | vi-zơ-**BI**-lơ-ti |
| một lượt đọc | one pass | |
| khoảng lệch | gap | |
| tương quan giữa các cột | column correlation | co-rơ-**LAY**-shơn; *column* có **n** cuối câm |
| độc lập | independent | in-đi-**PEN**-đơnt |
| xác suất | probability | pro-bơ-**BI**-lơ-ti |
| thuật toán | algorithm | **AL**-gơ-ri-đơm — trọng âm 1, "th" cuối đọc /ð/ |
| thuật toán nối | join algorithm | |
| vòng lặp lồng nhau | nested loop | **NES**-tid |
| nối bằng bảng băm | hash join | |
| nối trộn | merge join | *merge* /mɜːdʒ/ — đuôi /dʒ/ |
| bên ngoài / bên trong | the outer side / the inner side | |
| song song | in parallel | **PA**-rơ-lel — trọng âm 1 |
| mẫu câu truy vấn | query shape | |
| số lần gọi | call count | |
| tổng thời gian thực thi | total execution time | |
| thời gian trung bình | mean time | /miːn/ — nguyên âm dài |
| thủ phạm | culprit | **CUL**-prit |
| tải hệ thống | system load | |
| nghi phạm | suspect | danh từ **SUS**-pect; động từ sơs-**PECT** |
| kế hoạch bị lật | plan flip | |
| bộ nhớ đệm nguội | cold cache | *cache* /kæʃ/ — đọc y hệt "cash" |
| tranh chấp khoá | lock contention | cơn-**TEN**-shơn |
| khung nhìn thống kê | statistics view | stơ-**TIS**-tics — trọng âm 2 |
| cũ, không còn tươi | stale | /steɪl/ — "stêi-l" |
| phình bảng do rác tích tụ | bloat | /bləʊt/ — vần với "boat" |
| ngưỡng | threshold | **THRESH**-hould — âm **th** đầu lưỡi |
| bảng theo dõi | dashboard | |
| xu hướng | trend | cụm /tr-/ đọc liền |
| sự cố | incident | **IN**-si-đơnt — trọng âm 1 |
| xác minh | verify | **VE**-ri-fai — trọng âm 1 |
| cảm tính | gut feeling | |

---

## Bài nói

> **Cách luyện:** bật ghi âm, nói **60–90 giây** cho mỗi đề, **không nhìn tài liệu**. Mỗi bài nói phải dùng **ít nhất 6 thuật ngữ** trong bảng trên. Nghe lại bản ghi một lần, đánh dấu chỗ ngập ngừng, rồi nói lại đề đó lần thứ hai.

1. Walk a junior engineer through your process for debugging a slow query, from the first step to the last.

2. A colleague says: "The page is slow, I am pretty sure we just need an index on that column." Explain why you would push back and what you would ask them to bring you first.

3. A teammate wants to run `EXPLAIN ANALYZE` on an `UPDATE` statement in production. Explain what will happen and how you would do it safely.

4. Describe the four scan types you can see in a plan and what each one tells you about the query.

5. In a plan, one step estimates two rows but actually returns nine hundred thousand. Explain what that means, why the plan around it is probably wrong, and what you would check first.

6. When would you sort `pg_stat_statements` by total time rather than by mean time, and what would you miss if you always used mean time?

7. A query that was fast last month is now slow, and nobody has deployed anything. Talk your team through your list of suspects and how you would rule each one out.
