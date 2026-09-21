# Bài 8 — Isolation · MVCC · Locking · Deadlock
### Bản song ngữ: tiếng Anh bám cấu trúc tiếng Việt

> **Cách dùng bài này:** đọc đoạn tiếng Việt trước, tự dịch trong đầu, rồi mới so với đoạn tiếng Anh bên dưới. Chỗ nào lệch thì xem bảng ánh xạ cụm khó. Cuối cùng, **đọc to đoạn tiếng Anh hai lần** — một lần chậm để đúng âm, một lần với tốc độ nói thật.

---

## Phần 1 — Bốn mức cô lập theo độ chặt

**Tiếng Việt**

Chuẩn SQL định nghĩa bốn mức cô lập, và chúng được xếp từ lỏng đến chặt. Mức lỏng nhất là read uncommitted, tiếp theo là read committed, rồi đến repeatable read, và chặt nhất là serializable. Mỗi mức chặt hơn sẽ ngăn được thêm một loại bất thường, nhưng nó cũng đòi hỏi cơ sở dữ liệu làm thêm việc. Vì vậy, mức cô lập là một cần gạt đánh đổi giữa tính đúng đắn và thông lượng. Chúng ta không nên nhớ bốn cái tên này như một danh sách rời rạc, mà chúng ta nên nhớ chúng cùng với loại bất thường mà chúng ngăn được. Đó cũng chính là cách mà người phỏng vấn muốn nghe chúng ta trình bày.

**English (bám cấu trúc tiếng Việt)**

The SQL standard defines four isolation levels, and they are ordered from loose to strict. The loosest level is read uncommitted, next is read committed, then comes repeatable read, and the strictest is serializable. Each stricter level prevents one more kind of anomaly, but it also requires the database to do more work. Therefore, the isolation level is a trade-off lever between correctness and throughput. We should not remember these four names as a disconnected list, but we should remember them together with the kind of anomaly that they prevent. That is also exactly the way an interviewer wants to hear us present it.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| chúng được xếp từ lỏng đến chặt | they are ordered from loose to strict |
| mức lỏng nhất là | the loosest level is |
| rồi đến | then comes |
| mỗi mức chặt hơn sẽ ngăn được thêm một loại bất thường | each stricter level prevents one more kind of anomaly |
| nó cũng đòi hỏi cơ sở dữ liệu làm thêm việc | it also requires the database to do more work |
| một cần gạt đánh đổi | a trade-off lever |
| như một danh sách rời rạc | as a disconnected list |
| cùng với loại bất thường mà chúng ngăn được | together with the kind of anomaly that they prevent |
| cách mà người phỏng vấn muốn nghe chúng ta trình bày | the way an interviewer wants to hear us present it |

**Thuật ngữ cần nhớ**

- mức cô lập → **isolation level**
- bất thường khi chạy đồng thời → **anomaly**
- chặt / lỏng → **strict / loose**
- thông lượng → **throughput**
- chuẩn SQL → **the SQL standard**

---

## Phần 2 — Ba bất thường và mức nào ngăn được chúng

**Tiếng Việt**

Bất thường thứ nhất là dirty read, nghĩa là một transaction đọc được dữ liệu của một transaction khác chưa commit. Nếu transaction kia sau đó rollback, chúng ta đã ra quyết định dựa trên một con số chưa bao giờ tồn tại. Mức read committed ngăn được bất thường này, và đó là lý do gần như không ai dùng read uncommitted trong thực tế. Bất thường thứ hai là non-repeatable read, nghĩa là chúng ta đọc cùng một dòng hai lần trong cùng một transaction và chúng ta nhận được hai giá trị khác nhau. Chuyện này xảy ra vì một transaction khác đã commit một thay đổi vào giữa hai lần đọc, và mức repeatable read ngăn được nó. Bất thường thứ ba là phantom read, nghĩa là chúng ta chạy cùng một điều kiện lọc hai lần và chúng ta nhận được số dòng khác nhau. Lần này không phải một dòng bị sửa, mà là một dòng mới được thêm vào hoặc bị xoá đi, và chỉ mức serializable mới ngăn được hoàn toàn.

**English (bám cấu trúc tiếng Việt)**

The first anomaly is the dirty read, which means that one transaction reads the data of another transaction that has not committed. If that other transaction then rolls back, we have made a decision based on a number that never existed. The read committed level prevents this anomaly, and that is the reason why almost nobody uses read uncommitted in practice. The second anomaly is the non-repeatable read, which means that we read the same row twice inside the same transaction and we get two different values. This happens because another transaction has committed a change in between the two reads, and the repeatable read level prevents it. The third anomaly is the phantom read, which means that we run the same filter condition twice and we get a different number of rows. This time it is not a row being modified, but a new row being inserted or deleted, and only the serializable level prevents it completely.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| của một transaction khác chưa commit | of another transaction that has not committed |
| chúng ta đã ra quyết định dựa trên | we have made a decision based on |
| một con số chưa bao giờ tồn tại | a number that never existed |
| gần như không ai dùng … trong thực tế | almost nobody uses … in practice |
| chúng ta đọc cùng một dòng hai lần | we read the same row twice |
| đã commit một thay đổi vào giữa hai lần đọc | has committed a change in between the two reads |
| chạy cùng một điều kiện lọc hai lần | run the same filter condition twice |
| lần này không phải một dòng bị sửa, mà là | this time it is not a row being modified, but |
| chỉ … mới ngăn được hoàn toàn | only … prevents it completely |

**Thuật ngữ cần nhớ**

- đọc dữ liệu bẩn → **dirty read**
- đọc lại không giống lần đầu → **non-repeatable read**
- đọc thấy dòng ma → **phantom read**
- quay lui → **roll back**
- điều kiện lọc → **filter condition**

---

## Phần 3 — PostgreSQL không hành xử đúng y như chuẩn

**Tiếng Việt**

PostgreSQL không hành xử đúng y như chuẩn, và chúng ta cần biết hai điểm khác biệt. Điểm thứ nhất là mức mặc định của PostgreSQL là read committed, chứ không phải một mức chặt hơn. Điều đó có nghĩa là mỗi câu lệnh trong transaction nhìn thấy một ảnh chụp mới, vì vậy hai câu SELECT liên tiếp có thể cho ra hai kết quả khác nhau. Điểm thứ hai là mức mà PostgreSQL gọi là repeatable read thực chất là snapshot isolation, và nó mạnh hơn mức tối thiểu mà chuẩn yêu cầu. Nhờ đó, PostgreSQL ngăn được phần lớn phantom read ngay ở mức repeatable read, trong khi chuẩn chỉ bắt buộc điều đó ở mức serializable. Nếu chúng ta bê nguyên kiến thức từ một cơ sở dữ liệu khác sang, chúng ta sẽ mô tả sai hành vi thật của hệ thống mà mình đang vận hành.

**English (bám cấu trúc tiếng Việt)**

PostgreSQL does not behave exactly like the standard, and we need to know two differences. The first difference is that the default level of PostgreSQL is read committed, and not a stricter level. That means that each statement inside the transaction sees a fresh snapshot, therefore two consecutive SELECT statements can give two different results. The second difference is that the level PostgreSQL calls repeatable read is in fact snapshot isolation, and it is stronger than the minimum that the standard requires. Thanks to that, PostgreSQL prevents most phantom reads already at the repeatable read level, while the standard only requires that at the serializable level. If we carry knowledge over from another database as it is, we will describe the real behaviour of the system we are running wrongly.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| không hành xử đúng y như chuẩn | does not behave exactly like the standard |
| mức mặc định của PostgreSQL | the default level of PostgreSQL |
| nhìn thấy một ảnh chụp mới | sees a fresh snapshot |
| hai câu SELECT liên tiếp | two consecutive SELECT statements |
| mức mà PostgreSQL gọi là … thực chất là | the level PostgreSQL calls … is in fact |
| mạnh hơn mức tối thiểu mà chuẩn yêu cầu | stronger than the minimum that the standard requires |
| trong khi chuẩn chỉ bắt buộc điều đó ở | while the standard only requires that at |
| bê nguyên kiến thức từ … sang | carry knowledge over from … as it is |
| mô tả sai hành vi thật | describe the real behaviour wrongly |
| hệ thống mà mình đang vận hành | the system we are running |

**Thuật ngữ cần nhớ**

- cô lập theo ảnh chụp → **snapshot isolation**
- mức mặc định → **the default level**
- liên tiếp → **consecutive**
- hành vi → **behaviour**
- mức tối thiểu → **the minimum**

---

## Phần 4 — MVCC là trái tim của PostgreSQL

**Tiếng Việt**

MVCC là trái tim của PostgreSQL, và tên đầy đủ của nó là điều khiển đồng thời bằng nhiều phiên bản. Ý tưởng là mỗi dòng dữ liệu không bị ghi đè tại chỗ, mà mỗi lần cập nhật sẽ tạo ra một phiên bản mới của dòng đó. Mỗi phiên bản mang hai dấu hệ thống, một dấu ghi transaction đã tạo ra nó và một dấu ghi transaction đã xoá nó. Khi một transaction bắt đầu, nó nhận một ảnh chụp, và nó chỉ nhìn thấy những phiên bản hợp lệ đối với ảnh chụp đó. Nhờ cơ chế này, người đọc không cần lấy khoá đọc, và đó là lý do người đọc không chặn người ghi còn người ghi cũng không chặn người đọc. Chỉ khi hai người ghi cùng chạm vào một dòng thì họ mới thật sự đụng nhau và phải chờ nhau. Cái giá của MVCC là các phiên bản cũ tích tụ lại theo thời gian, vì vậy cơ sở dữ liệu phải chạy VACUUM để dọn chúng, và chúng ta sẽ nói kỹ về việc này trong bài sau.

**English (bám cấu trúc tiếng Việt)**

MVCC is the heart of PostgreSQL, and its full name is multi-version concurrency control. The idea is that each data row is not overwritten in place, but each update creates a new version of that row. Each version carries two system marks, one mark recording the transaction that created it and one mark recording the transaction that deleted it. When a transaction starts, it receives a snapshot, and it only sees the versions that are valid for that snapshot. Thanks to this mechanism, a reader does not need to take a read lock, and that is the reason why a reader does not block a writer and a writer does not block a reader either. Only when two writers touch the same row do they actually collide and have to wait for each other. The price of MVCC is that the old versions pile up over time, therefore the database has to run VACUUM to clean them, and we will talk about this in detail in the next lesson.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| tên đầy đủ của nó là | its full name is |
| không bị ghi đè tại chỗ | is not overwritten in place |
| mang hai dấu hệ thống | carries two system marks |
| một dấu ghi transaction đã tạo ra nó | one mark recording the transaction that created it |
| những phiên bản hợp lệ đối với ảnh chụp đó | the versions that are valid for that snapshot |
| nhờ cơ chế này | thanks to this mechanism |
| người đọc không chặn người ghi | a reader does not block a writer |
| họ mới thật sự đụng nhau | they actually collide |
| các phiên bản cũ tích tụ lại theo thời gian | the old versions pile up over time |
| cái giá của MVCC là | the price of MVCC is |

**Thuật ngữ cần nhớ**

- điều khiển đồng thời nhiều phiên bản → **multi-version concurrency control (MVCC)**
- ghi đè tại chỗ → **overwrite in place**
- khoá đọc → **read lock**
- dọn phiên bản cũ → **VACUUM**
- tích tụ → **pile up**

---

## Phần 5 — Serializable là cơ chế lạc quan và nó cần vòng lặp thử lại

**Tiếng Việt**

Mức serializable trong PostgreSQL được cài đặt theo hướng lạc quan, và tên kỹ thuật của nó là serializable snapshot isolation. Cơ sở dữ liệu không khoá trước để ngăn xung đột, mà nó theo dõi các phụ thuộc giữa những transaction đang chạy. Nếu nó phát hiện một vòng phụ thuộc có thể dẫn tới kết quả không tuần tự hoá được, nó sẽ huỷ một trong các transaction đó. Transaction bị huỷ nhận về lỗi serialization_failure, và cơ sở dữ liệu không tự chạy lại giúp chúng ta. Vì vậy, khi chúng ta dùng mức serializable, ứng dụng bắt buộc phải có một vòng lặp thử lại. Nếu chúng ta quên viết vòng lặp đó, hệ thống sẽ ném lỗi thẳng ra người dùng đúng vào những lúc tải cao nhất.

**English (bám cấu trúc tiếng Việt)**

The serializable level in PostgreSQL is implemented in an optimistic way, and its technical name is serializable snapshot isolation. The database does not lock in advance in order to prevent conflicts, but it tracks the dependencies between the transactions that are running. If it detects a dependency cycle that could lead to a result which is not serialisable, it will abort one of those transactions. The aborted transaction gets back a serialization_failure error, and the database does not re-run it for us. Therefore, when we use the serializable level, the application is required to have a retry loop. If we forget to write that loop, the system will throw the error straight out to the user exactly at the moments of highest load.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| được cài đặt theo hướng lạc quan | is implemented in an optimistic way |
| không khoá trước để ngăn xung đột | does not lock in advance in order to prevent conflicts |
| nó theo dõi các phụ thuộc giữa những transaction đang chạy | it tracks the dependencies between the transactions that are running |
| một vòng phụ thuộc | a dependency cycle |
| dẫn tới kết quả không tuần tự hoá được | lead to a result which is not serialisable |
| nó sẽ huỷ một trong các transaction đó | it will abort one of those transactions |
| cơ sở dữ liệu không tự chạy lại giúp chúng ta | the database does not re-run it for us |
| ứng dụng bắt buộc phải có một vòng lặp thử lại | the application is required to have a retry loop |
| ném lỗi thẳng ra người dùng | throw the error straight out to the user |
| đúng vào những lúc tải cao nhất | exactly at the moments of highest load |

**Thuật ngữ cần nhớ**

- lạc quan / bi quan → **optimistic / pessimistic**
- vòng phụ thuộc → **dependency cycle**
- huỷ transaction → **abort a transaction**
- vòng lặp thử lại → **retry loop**
- lỗi tuần tự hoá → **serialization failure**

---

## Phần 6 — Lost update và ba cách chặn nó

**Tiếng Việt**

Lost update là tình huống hai transaction cùng đọc một giá trị rồi cùng ghi đè lên nhau. Ví dụ, hai request cùng đọc tồn kho bằng mười, cả hai cùng tính ra chín, và cả hai cùng ghi số chín xuống. Kết quả là chúng ta đã bán hai món hàng nhưng chúng ta chỉ trừ đi một đơn vị tồn kho. Cách chặn thứ nhất là chúng ta viết một câu cập nhật nguyên tử, tức là chúng ta trừ trực tiếp trên cột và chúng ta kèm điều kiện tồn kho phải lớn hơn không. Cách chặn thứ hai là khoá bi quan bằng SELECT FOR UPDATE, trong đó chúng ta khoá dòng ngay khi đọc để transaction khác phải chờ. Cách chặn thứ ba là khoá lạc quan bằng một cột version, trong đó câu cập nhật chỉ thành công nếu version vẫn đúng như lúc chúng ta đọc. Với cách thứ ba, chúng ta phải kiểm tra số dòng bị ảnh hưởng, và nếu số đó bằng không thì chúng ta đọc lại rồi thử lại.

**English (bám cấu trúc tiếng Việt)**

A lost update is the situation where two transactions both read one value and then both overwrite each other. For example, two requests both read the stock as ten, both calculate nine, and both write the number nine down. The result is that we have sold two items but we have only subtracted one unit of stock. The first way to prevent it is that we write an atomic update statement, that is we subtract directly on the column and we attach the condition that the stock must be greater than zero. The second way to prevent it is a pessimistic lock with SELECT FOR UPDATE, in which we lock the row at the moment we read it so that other transactions have to wait. The third way to prevent it is an optimistic lock with a version column, in which the update statement only succeeds if the version is still the same as when we read it. With the third way, we have to check the number of affected rows, and if that number is zero then we read again and retry.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| cùng ghi đè lên nhau | both overwrite each other |
| cả hai cùng tính ra chín | both calculate nine |
| chúng ta chỉ trừ đi một đơn vị tồn kho | we have only subtracted one unit of stock |
| một câu cập nhật nguyên tử | an atomic update statement |
| chúng ta trừ trực tiếp trên cột | we subtract directly on the column |
| chúng ta kèm điều kiện | we attach the condition |
| chúng ta khoá dòng ngay khi đọc | we lock the row at the moment we read it |
| chỉ thành công nếu version vẫn đúng như lúc chúng ta đọc | only succeeds if the version is still the same as when we read it |
| số dòng bị ảnh hưởng | the number of affected rows |
| chúng ta đọc lại rồi thử lại | we read again and retry |

**Thuật ngữ cần nhớ**

- mất bản cập nhật → **lost update**
- cập nhật nguyên tử → **atomic update**
- khoá bi quan → **pessimistic lock**
- khoá lạc quan bằng cột version → **optimistic lock with a version column**
- tồn kho → **stock / inventory**

---

## Phần 7 — Deadlock: vì sao xảy ra và cách phòng

**Tiếng Việt**

Deadlock xảy ra khi hai transaction khoá hai tài nguyên theo hai thứ tự ngược nhau. Transaction thứ nhất khoá bảng A rồi chờ bảng B, trong khi transaction thứ hai khoá bảng B rồi chờ bảng A. Không ai nhả khoá trước, vì vậy cả hai sẽ chờ nhau mãi mãi nếu không có ai can thiệp. PostgreSQL phát hiện tình huống này sau một khoảng chờ và nó huỷ một transaction làm nạn nhân, còn transaction kia thì được chạy tiếp. Cách phòng hiệu quả nhất là chúng ta luôn khoá các tài nguyên theo cùng một thứ tự trong toàn bộ mã nguồn. Ngoài ra, chúng ta giữ transaction thật ngắn và chúng ta thu hẹp phạm vi khoá, vì hai việc đó làm giảm hẳn cơ hội xảy ra deadlock.

**English (bám cấu trúc tiếng Việt)**

A deadlock happens when two transactions lock two resources in two opposite orders. The first transaction locks table A then waits for table B, while the second transaction locks table B then waits for table A. Neither one releases its lock first, therefore both will wait for each other forever if nobody steps in. PostgreSQL detects this situation after a waiting period and it aborts one transaction as the victim, while the other transaction is allowed to continue. The most effective prevention is that we always lock the resources in the same order across the whole codebase. In addition, we keep transactions really short and we narrow the lock scope, because those two things clearly reduce the chance of a deadlock happening.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| theo hai thứ tự ngược nhau | in two opposite orders |
| rồi chờ bảng B | then waits for table B |
| không ai nhả khoá trước | neither one releases its lock first |
| sẽ chờ nhau mãi mãi nếu không có ai can thiệp | will wait for each other forever if nobody steps in |
| sau một khoảng chờ | after a waiting period |
| nó huỷ một transaction làm nạn nhân | it aborts one transaction as the victim |
| được chạy tiếp | is allowed to continue |
| theo cùng một thứ tự trong toàn bộ mã nguồn | in the same order across the whole codebase |
| chúng ta thu hẹp phạm vi khoá | we narrow the lock scope |
| làm giảm hẳn cơ hội xảy ra deadlock | clearly reduce the chance of a deadlock happening |

**Thuật ngữ cần nhớ**

- khoá chết → **deadlock**
- tài nguyên → **resource**
- nhả khoá → **release a lock**
- nạn nhân bị huỷ → **the victim transaction**
- phạm vi khoá → **lock scope**

---

## Phần 8 — Đừng nâng serializable toàn cục cho lành

**Tiếng Việt**

Khi gặp vấn đề về đồng thời, phản xạ đầu tiên của nhiều đội là nâng mức cô lập lên serializable cho cả cơ sở dữ liệu. Cách làm này nghe có vẻ an toàn, nhưng nó không hề miễn phí. Chi phí thứ nhất là số lần huỷ transaction tăng lên, và mỗi lần huỷ đều kéo theo một lần chạy lại toàn bộ công việc. Chi phí thứ hai là thông lượng của cả hệ thống giảm xuống, kể cả ở những phần vốn không hề có tranh chấp. Cách làm đúng là chúng ta giữ mức mặc định cho toàn hệ thống và chúng ta xử lý riêng từng điểm nóng. Ví dụ, với việc trừ tồn kho hoặc trừ ví điểm, chúng ta dùng một câu cập nhật nguyên tử hoặc chúng ta dùng FOR UPDATE trên đúng những dòng đó. Chúng ta chỉ chọn serializable cho những nghiệp vụ đòi hỏi bảo đảm rất mạnh, ví dụ đặt vé hoặc chuyển khoản ngân hàng, và khi đó chúng ta luôn viết kèm một vòng lặp thử lại.

**English (bám cấu trúc tiếng Việt)**

When we meet a concurrency problem, the first reflex of many teams is to raise the isolation level to serializable for the whole database. This approach sounds safe, but it is not free at all. The first cost is that the number of transaction aborts goes up, and every abort drags along a full re-run of the work. The second cost is that the throughput of the whole system goes down, even in the parts that have no contention at all. The correct approach is that we keep the default level for the whole system and we handle each hot spot separately. For example, for subtracting stock or subtracting a points wallet, we use an atomic update statement or we use FOR UPDATE on exactly those rows. We only choose serializable for the business flows that demand very strong guarantees, for example ticket booking or a bank transfer, and in that case we always write a retry loop along with it.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| phản xạ đầu tiên của nhiều đội | the first reflex of many teams |
| nâng mức cô lập lên serializable | raise the isolation level to serializable |
| nghe có vẻ an toàn | sounds safe |
| nó không hề miễn phí | it is not free at all |
| mỗi lần huỷ đều kéo theo một lần chạy lại toàn bộ công việc | every abort drags along a full re-run of the work |
| kể cả ở những phần vốn không hề có tranh chấp | even in the parts that have no contention at all |
| chúng ta xử lý riêng từng điểm nóng | we handle each hot spot separately |
| trên đúng những dòng đó | on exactly those rows |
| đòi hỏi bảo đảm rất mạnh | demand very strong guarantees |
| chúng ta luôn viết kèm một vòng lặp thử lại | we always write a retry loop along with it |

**Thuật ngữ cần nhớ**

- tranh chấp tài nguyên → **contention**
- điểm nóng → **hot spot**
- chạy lại → **re-run / retry**
- bảo đảm mạnh → **strong guarantee**
- đặt vé → **ticket booking**

---

## Mô hình ghi nhớ

**Tiếng Việt**

MVCC cho phép người đọc làm việc trên một ảnh chụp riêng, vì vậy người đọc không chặn người ghi. Với các điểm nóng, chúng ta dùng câu cập nhật nguyên tử hoặc dùng FOR UPDATE cục bộ, và chúng ta không nâng serializable cho toàn hệ thống cho lành.

**English (bám cấu trúc tiếng Việt)**

MVCC lets a reader work on its own snapshot, therefore a reader does not block a writer. For hot spots, we use an atomic update statement or we use FOR UPDATE locally, and we do not raise serializable for the whole system just to be safe.

---

## Bảng thuật ngữ tổng hợp

| Tiếng Việt | Tiếng Anh | Ghi chú phát âm |
|---|---|---|
| mức cô lập | isolation level | *isolation* ai-sơ-**LAY**-shơn — âm đầu là "ai" |
| bất thường khi chạy đồng thời | anomaly | ơ-**NO**-mơ-li — trọng âm 2 |
| chuẩn SQL | the SQL standard | |
| chặt / lỏng | strict / loose | *loose* /luːs/ đuôi /s/, khác *lose* /luːz/ |
| thông lượng | throughput | /ˈθruːpʊt/ — âm **th** đầu lưỡi |
| đọc dữ liệu bẩn | dirty read | |
| đọc lại không giống lần đầu | non-repeatable read | ri-**PEE**-tơ-bl |
| đọc thấy dòng ma | phantom read | **FAN**-tơm — "ph" đọc /f/ |
| có thể tuần tự hoá | serializable | si-ri-ơ-**LAI**-zơ-bl |
| cô lập theo ảnh chụp | snapshot isolation | |
| liên tiếp | consecutive | cơn-**SE**-kiu-tiv — trọng âm 2 |
| hành vi | behaviour | bi-**HEIV**-yơ; Anh viết *-our* |
| điều khiển đồng thời nhiều phiên bản | multi-version concurrency control | *concurrency* cơn-**CU**-rơn-si |
| ghi đè tại chỗ | overwrite in place | *write* — chữ **w** câm |
| phiên bản của dòng | row version | |
| khoá đọc | read lock | |
| dọn phiên bản cũ | VACUUM | **VA**-kiu-ơm — trọng âm 1 |
| tích tụ | pile up | |
| cơ chế | mechanism | **ME**-cơ-ni-zơm — "ch" đọc /k/ |
| lạc quan / bi quan | optimistic / pessimistic | op-ti-**MIS**-tic · pe-si-**MIS**-tic |
| vòng phụ thuộc | dependency cycle | *cycle* **SAI**-kl |
| huỷ transaction | abort a transaction | ơ-**BORT** — trọng âm 2 |
| vòng lặp thử lại | retry loop | |
| lỗi tuần tự hoá | serialization failure | **FEI**-li-ơ |
| mất bản cập nhật | lost update | |
| cập nhật nguyên tử | atomic update | ơ-**TO**-mic — trọng âm 2 |
| khoá bi quan | pessimistic lock | |
| khoá lạc quan | optimistic lock | |
| cột đánh số phiên bản | version column | *column* — chữ **n** cuối câm |
| tồn kho | stock / inventory | *inventory* Anh **IN**-vơn-tơ-ri — trọng âm 1 |
| số dòng bị ảnh hưởng | affected rows | ơ-**FEC**-tid |
| khoá chết | deadlock | |
| tài nguyên | resource | Anh /rɪˈzɔːs/ — ri-**ZORS**, trọng âm 2 |
| lấy khoá / nhả khoá | acquire a lock / release a lock | *acquire* ơ-**KWAI**-ơ — chữ **c** câm |
| nạn nhân bị huỷ | the victim transaction | **VIC**-tim |
| phạm vi khoá | lock scope | |
| tranh chấp tài nguyên | contention | cơn-**TEN**-shơn |
| điểm nóng | hot spot | |
| bảo đảm mạnh | strong guarantee | *guarantee* trọng âm cuối: ga-rơn-**TEE** |
| đặt vé | ticket booking | |
| ví điểm | points wallet | **WO**-lit |
| can thiệp | step in / intervene | *intervene* in-tơ-**VEEN** |
| rủi ro | risk | đuôi /-sk/ phải bật ra, không thành "rít" |

---

## Bài nói

> **Cách luyện:** bật ghi âm, nói **60–90 giây** cho mỗi đề, **không nhìn tài liệu**. Mỗi bài nói phải dùng **ít nhất 6 thuật ngữ** trong bảng trên. Nghe lại bản ghi một lần, đánh dấu chỗ ngập ngừng, rồi nói lại đề đó lần thứ hai.

1. Explain the three concurrency anomalies to a junior engineer and say which isolation level stops each one.

2. A colleague says: "Let us set the whole database to serializable, then we never have to think about concurrency again." Explain why you would push back and what you would do instead.

3. Describe what happens inside PostgreSQL when one transaction reads a row while another transaction is updating the same row, and explain why the reader does not have to wait.

4. Two requests are both selling the last item in stock. Walk your team through three different ways to make sure the stock never goes negative, and say which one you would pick.

5. A teammate has written `SELECT` then `UPDATE` to decrement a counter and says it works fine in testing. Explain what will go wrong in production and why testing did not catch it.

6. Describe how a deadlock forms between two transactions and explain what the database does about it.

7. When would you choose a pessimistic lock over an optimistic version column, and when would the opposite be the better choice?
