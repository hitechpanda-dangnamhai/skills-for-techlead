# Bài 11 — Connection pooling
### Bản song ngữ: tiếng Anh bám cấu trúc tiếng Việt

> **Cách dùng bài này:** đọc đoạn tiếng Việt trước, tự dịch trong đầu, rồi mới so với đoạn tiếng Anh bên dưới. Chỗ nào lệch thì xem bảng ánh xạ cụm khó. Cuối cùng, **đọc to đoạn tiếng Anh hai lần** — một lần chậm để đúng âm, một lần với tốc độ nói thật.

---

## Phần 1 — Vì sao chúng ta cần một pool kết nối

**Tiếng Việt**

Mỗi lần ứng dụng mở một kết nối mới tới PostgreSQL, chi phí không hề nhỏ. Trước hết, hai bên phải bắt tay TCP, rồi phải chạy qua bước xác thực, và có thể phải thiết lập cả TLS. Sau đó, máy chủ phải tạo hẳn một tiến trình mới ở phía backend cho riêng kết nối đó. Toàn bộ chuỗi việc này có thể tốn vài chục mili giây, và nó lặp lại cho từng request nếu chúng ta không làm gì cả. Pool giải quyết chuyện đó bằng cách giữ sẵn một số kết nối đã mở và cho các request mượn lại lần lượt. Nhờ vậy, chúng ta giảm được độ trễ của từng request và chúng ta bảo vệ cơ sở dữ liệu khỏi việc bị quá tải vì số kết nối. Nếu chúng ta không có pool, hệ thống vẫn chạy tốt ở môi trường thử nghiệm, nhưng nó sẽ sụp ngay trong đợt tải cao đầu tiên.

**English (bám cấu trúc tiếng Việt)**

Every time the application opens a new connection to PostgreSQL, the cost is not small at all. First of all, the two sides have to do a TCP handshake, then they have to go through the authentication step, and they may also have to set up TLS. After that, the server has to create a whole new process on the backend side just for that connection. This whole chain of work can cost a few tens of milliseconds, and it repeats for every request if we do nothing at all. A pool solves that by keeping a number of already open connections and lending them out to requests one after another. Thanks to that, we reduce the latency of each request and we protect the database from being overloaded by the number of connections. If we have no pool, the system still runs fine in the test environment, but it will collapse during the first heavy load.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| chi phí không hề nhỏ | the cost is not small at all |
| hai bên phải bắt tay TCP | the two sides have to do a TCP handshake |
| chạy qua bước xác thực | go through the authentication step |
| tạo hẳn một tiến trình mới ở phía backend | create a whole new process on the backend side |
| toàn bộ chuỗi việc này | this whole chain of work |
| nếu chúng ta không làm gì cả | if we do nothing at all |
| giữ sẵn một số kết nối đã mở | keeping a number of already open connections |
| cho các request mượn lại lần lượt | lending them out to requests one after another |
| bị quá tải vì số kết nối | being overloaded by the number of connections |
| nó sẽ sụp ngay trong đợt tải cao đầu tiên | it will collapse during the first heavy load |

**Thuật ngữ cần nhớ**

- bắt tay giao thức → **handshake**
- xác thực → **authentication**
- tiến trình phía máy chủ → **backend process**
- độ trễ → **latency**
- quá tải → **overloaded**

---

## Phần 2 — Pool nhỏ mà đúng tốt hơn pool lớn

**Tiếng Việt**

Câu hỏi tiếp theo là pool nên lớn bao nhiêu, và ở đây có một hiểu lầm rất phổ biến. Nhiều người nghĩ rằng pool càng lớn thì hệ thống càng phục vụ được nhiều người, nhưng điều đó sai. Khi số kết nối đang hoạt động vượt quá năng lực của cơ sở dữ liệu, máy chủ tốn thời gian cho việc chuyển ngữ cảnh và cho tranh chấp khoá nhiều hơn là cho công việc thật. Kết quả là thông lượng giảm xuống trong khi độ trễ của mọi truy vấn đều tăng lên. Một công thức kinh nghiệm thường dùng là hai lần số nhân của CPU cộng thêm số ổ đĩa hiệu dụng, nhưng chúng ta phải coi đó là điểm khởi đầu chứ không phải là câu trả lời cuối cùng. Cách đúng là chúng ta đo thông lượng thật ở vài mức pool khác nhau, và chúng ta chọn theo năng lực của cơ sở dữ liệu chứ chúng ta không chọn theo số kết nối mà ứng dụng muốn có.

**English (bám cấu trúc tiếng Việt)**

The next question is how large the pool should be, and here there is a very common misunderstanding. Many people think that the larger the pool is, the more users the system can serve, but that is wrong. When the number of active connections goes beyond the capacity of the database, the server spends more time on context switching and on lock contention than on the real work. The result is that throughput goes down while the latency of every query goes up. A rule of thumb that is often used is twice the number of CPU cores plus the number of effective spindles, but we have to treat that as a starting point and not as the final answer. The correct way is that we measure the real throughput at a few different pool sizes, and we choose according to the capacity of the database and we do not choose according to the number of connections that the application would like to have.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| pool nên lớn bao nhiêu | how large the pool should be |
| pool càng lớn thì hệ thống càng phục vụ được nhiều người | the larger the pool is, the more users the system can serve |
| vượt quá năng lực của cơ sở dữ liệu | goes beyond the capacity of the database |
| tốn thời gian cho … nhiều hơn là cho công việc thật | spends more time on … than on the real work |
| một công thức kinh nghiệm thường dùng | a rule of thumb that is often used |
| số ổ đĩa hiệu dụng | the number of effective spindles |
| chúng ta phải coi đó là điểm khởi đầu | we have to treat that as a starting point |
| ở vài mức pool khác nhau | at a few different pool sizes |
| số kết nối mà ứng dụng muốn có | the number of connections that the application would like to have |

**Thuật ngữ cần nhớ**

- kích thước pool → **pool size**
- năng lực xử lý → **capacity**
- chuyển ngữ cảnh → **context switching**
- tranh chấp khoá → **lock contention**
- công thức kinh nghiệm → **rule of thumb**

---

## Phần 3 — PgBouncer và ba chế độ gom kết nối

**Tiếng Việt**

PgBouncer là trình gom kết nối phổ biến nhất cho PostgreSQL, và nó có ba chế độ hoạt động. Chế độ session giữ một kết nối cho suốt phiên làm việc của client, vì vậy nó an toàn nhất nhưng nó tái dùng kém nhất. Chế độ transaction trả kết nối về pool ngay sau mỗi transaction, vì vậy nó tái dùng tốt nhất và nó là lựa chọn mặc định cho một ứng dụng web. Chế độ statement trả kết nối về ngay sau mỗi câu lệnh, vì vậy nó chặt nhất và nó rất ít khi được dùng. Với chế độ transaction, một kết nối thật tới cơ sở dữ liệu có thể phục vụ hàng chục client khác nhau trong cùng một giây. Đó chính là lý do một pooler cho phép hàng nghìn kết nối từ phía ứng dụng trong khi nó chỉ giữ vài chục kết nối thật tới cơ sở dữ liệu.

**English (bám cấu trúc tiếng Việt)**

PgBouncer is the most popular connection pooler for PostgreSQL, and it has three working modes. Session mode holds one connection for the whole working session of the client, therefore it is the safest but it reuses connections the worst. Transaction mode returns the connection to the pool right after each transaction, therefore it reuses connections the best and it is the default choice for a web application. Statement mode returns the connection right after each statement, therefore it is the strictest and it is very rarely used. With transaction mode, one real connection to the database can serve dozens of different clients within the same second. That is exactly the reason why a pooler allows thousands of connections from the application side while it only holds a few dozen real connections to the database.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| trình gom kết nối phổ biến nhất | the most popular connection pooler |
| ba chế độ hoạt động | three working modes |
| suốt phiên làm việc của client | the whole working session of the client |
| nó tái dùng kém nhất | it reuses connections the worst |
| trả kết nối về pool ngay sau mỗi transaction | returns the connection to the pool right after each transaction |
| nó rất ít khi được dùng | it is very rarely used |
| hàng chục client khác nhau trong cùng một giây | dozens of different clients within the same second |
| đó chính là lý do | that is exactly the reason why |
| trong khi nó chỉ giữ vài chục kết nối thật | while it only holds a few dozen real connections |

**Thuật ngữ cần nhớ**

- trình gom kết nối → **connection pooler**
- chế độ theo phiên → **session mode**
- chế độ theo transaction → **transaction mode**
- chế độ theo câu lệnh → **statement mode**
- tái dùng → **reuse**

---

## Phần 4 — Cái bẫy của chế độ transaction: trạng thái phiên

**Tiếng Việt**

Chế độ transaction tái dùng rất tốt, nhưng nó đi kèm một điều kiện mà chúng ta phải nhớ. Điều kiện đó là ứng dụng không được giữ bất kỳ trạng thái nào bên trong phiên kết nối. Lý do rất đơn giản, vì transaction tiếp theo của chúng ta có thể chạy trên một kết nối thật hoàn toàn khác. Vì vậy, những thứ như biến phiên đặt bằng lệnh SET, advisory lock, và một số kiểu prepared statement sẽ không hoạt động như chúng ta mong đợi. Lỗi này rất khó tìm, vì nó chỉ xuất hiện khi có tải và nó biến mất khi chúng ta thử lại một mình. Cách xử lý là chúng ta đọc tài liệu của pooler và của driver, rồi chúng ta bật đúng cấu hình cho prepared statement thay vì tắt hẳn tính năng đó.

**English (bám cấu trúc tiếng Việt)**

Transaction mode reuses connections very well, but it comes with one condition that we have to remember. That condition is that the application must not hold any state inside the connection session. The reason is very simple, because our next transaction may run on a completely different real connection. Therefore, things like session variables set with the SET command, advisory locks, and some kinds of prepared statement will not work the way we expect. This bug is very hard to find, because it only shows up under load and it disappears when we try it again on our own. The way to handle it is that we read the documentation of the pooler and of the driver, then we turn on the right configuration for prepared statements instead of switching that feature off completely.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| nó đi kèm một điều kiện mà chúng ta phải nhớ | it comes with one condition that we have to remember |
| không được giữ bất kỳ trạng thái nào bên trong phiên kết nối | must not hold any state inside the connection session |
| có thể chạy trên một kết nối thật hoàn toàn khác | may run on a completely different real connection |
| biến phiên đặt bằng lệnh SET | session variables set with the SET command |
| sẽ không hoạt động như chúng ta mong đợi | will not work the way we expect |
| nó chỉ xuất hiện khi có tải | it only shows up under load |
| nó biến mất khi chúng ta thử lại một mình | it disappears when we try it again on our own |
| chúng ta bật đúng cấu hình cho | we turn on the right configuration for |
| thay vì tắt hẳn tính năng đó | instead of switching that feature off completely |

**Thuật ngữ cần nhớ**

- trạng thái phiên → **session state**
- biến phiên → **session variable**
- khoá cố vấn → **advisory lock**
- câu lệnh chuẩn bị sẵn → **prepared statement**
- chỉ xuất hiện khi có tải → **only shows up under load**

---

## Phần 5 — Serverless và cơn bão kết nối

**Tiếng Việt**

Kiến trúc serverless tạo ra một vấn đề rất đặc thù với kết nối cơ sở dữ liệu. Mỗi instance của hàm chạy trong một môi trường riêng, vì vậy mỗi instance mở pool riêng của chính nó. Khi lưu lượng tăng và nền tảng mở rộng ngang lên hàng trăm instance, số kết nối cũng bùng nổ theo. Cơ sở dữ liệu sẽ chạm trần kết nối chỉ trong vài giây, và mọi instance mới sẽ nhận lỗi ngay lúc khởi động. Cách giải quyết là chúng ta đặt một pooler bên ngoài giữa hàm và cơ sở dữ liệu, ví dụ PgBouncer hoặc một proxy do nhà cung cấp quản lý. Pooler đó chịu toàn bộ cơn bão kết nối từ phía hàm, còn phía cơ sở dữ liệu thì chỉ nhìn thấy một số ít kết nối ổn định.

**English (bám cấu trúc tiếng Việt)**

Serverless architecture creates a very specific problem with database connections. Each instance of the function runs in its own environment, therefore each instance opens its own pool. When the traffic grows and the platform scales out to hundreds of instances, the number of connections explodes as well. The database will hit the connection ceiling within just a few seconds, and every new instance will get an error right at start-up. The solution is that we put an external pooler between the function and the database, for example PgBouncer or a proxy managed by the provider. That pooler takes the whole connection storm from the function side, while the database side only sees a small number of stable connections.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| tạo ra một vấn đề rất đặc thù | creates a very specific problem |
| chạy trong một môi trường riêng | runs in its own environment |
| mở pool riêng của chính nó | opens its own pool |
| nền tảng mở rộng ngang lên hàng trăm instance | the platform scales out to hundreds of instances |
| số kết nối cũng bùng nổ theo | the number of connections explodes as well |
| sẽ chạm trần kết nối | will hit the connection ceiling |
| nhận lỗi ngay lúc khởi động | get an error right at start-up |
| một proxy do nhà cung cấp quản lý | a proxy managed by the provider |
| chịu toàn bộ cơn bão kết nối | takes the whole connection storm |
| chỉ nhìn thấy một số ít kết nối ổn định | only sees a small number of stable connections |

**Thuật ngữ cần nhớ**

- mở rộng ngang → **scale out**
- trần kết nối → **the connection ceiling**
- cơn bão kết nối → **connection storm**
- pooler bên ngoài → **external pooler**
- nhà cung cấp dịch vụ → **the provider**

---

## Phần 6 — Phép tính trần mà một tech lead luôn phải làm

**Tiếng Việt**

Có một phép tính rất đơn giản mà một tech lead phải luôn làm trong đầu. Tổng số kết nối tới cơ sở dữ liệu bằng kích thước pool của mỗi instance nhân với số instance đang chạy. Con số đó phải nhỏ hơn hoặc bằng max_connections trừ đi phần dự phòng. Phần dự phòng dành cho kết nối của superuser để chúng ta còn vào được khi có sự cố, và nó cũng dành cho các tiến trình nội bộ như nhân bản dữ liệu. Ví dụ, nếu mỗi instance có pool hai mươi và chúng ta chạy mười instance, tổng là hai trăm, và con số đó lọt dưới ba trăm trừ hai mươi. Điểm nguy hiểm nằm ở chỗ tự động mở rộng, vì một cấu hình an toàn với hai instance sẽ vỡ trận khi hệ thống mở lên bốn mươi instance. Vì vậy, chúng ta luôn tính theo số instance tối đa mà chính sách tự động mở rộng cho phép, chứ chúng ta không tính theo số instance hiện tại.

**English (bám cấu trúc tiếng Việt)**

There is a very simple calculation that a tech lead always has to do in their head. The total number of connections to the database equals the pool size of each instance multiplied by the number of instances that are running. That number must be less than or equal to max_connections minus the reserved part. The reserved part is for the superuser connection so that we can still get in when there is an incident, and it is also for internal processes such as replication. For example, if each instance has a pool of twenty and we run ten instances, the total is two hundred, and that number sits under three hundred minus twenty. The dangerous point lies in autoscaling, because a configuration that is safe with two instances will break down when the system scales up to forty instances. Therefore, we always calculate with the maximum number of instances that the autoscaling policy allows, and we do not calculate with the current number of instances.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| một phép tính rất đơn giản | a very simple calculation |
| phải luôn làm trong đầu | always has to do in their head |
| nhân với số instance đang chạy | multiplied by the number of instances that are running |
| phải nhỏ hơn hoặc bằng | must be less than or equal to |
| trừ đi phần dự phòng | minus the reserved part |
| để chúng ta còn vào được khi có sự cố | so that we can still get in when there is an incident |
| con số đó lọt dưới | that number sits under |
| điểm nguy hiểm nằm ở chỗ tự động mở rộng | the dangerous point lies in autoscaling |
| sẽ vỡ trận khi hệ thống mở lên bốn mươi instance | will break down when the system scales up to forty instances |
| số instance tối đa mà chính sách tự động mở rộng cho phép | the maximum number of instances that the autoscaling policy allows |

**Thuật ngữ cần nhớ**

- trần kết nối tối đa → **max_connections**
- phần dự phòng → **the reserved part**
- tự động mở rộng → **autoscaling**
- nhân bản dữ liệu → **replication**
- vỡ trận → **break down**

---

## Phần 7 — "Pool exhausted": sửa gốc trước, đừng tăng pool

**Tiếng Việt**

Triệu chứng thường gặp nhất là ứng dụng báo lỗi hết thời gian chờ khi nó xin một kết nối từ pool. Phản xạ sai là chúng ta lập tức tăng kích thước pool lên, vì cách đó chỉ che triệu chứng chứ nó không chữa gốc. Nguyên nhân gốc thứ nhất là có truy vấn chậm hoặc có transaction chạy lâu đang giữ kết nối quá lâu. Nguyên nhân gốc thứ hai là lỗi N+1, vì một request bắn ra năm mươi mốt câu truy vấn sẽ giữ kết nối lâu gấp nhiều lần một request bình thường. Nguyên nhân gốc thứ ba là rò rỉ kết nối, nghĩa là code lấy kết nối ra nhưng quên trả nó về pool khi có lỗi. Chúng ta kiểm tra ba nguyên nhân đó trước, và chỉ khi cả ba đều sạch thì chúng ta mới xét tới khả năng pool thật sự quá nhỏ. Nếu chúng ta tăng pool mà không sửa gốc, chúng ta chỉ chuyển điểm nghẽn từ ứng dụng sang cơ sở dữ liệu, và lần sập sau sẽ nặng hơn lần này.

**English (bám cấu trúc tiếng Việt)**

The most common symptom is that the application reports a timeout error when it asks for a connection from the pool. The wrong reflex is that we immediately raise the pool size, because that approach only hides the symptom and it does not cure the root. The first root cause is that there is a slow query or a long-running transaction holding a connection for too long. The second root cause is the N+1 problem, because one request that fires fifty-one queries will hold a connection many times longer than a normal request. The third root cause is a connection leak, which means that the code takes a connection out but forgets to return it to the pool when an error happens. We check those three causes first, and only when all three are clean do we consider the possibility that the pool really is too small. If we raise the pool without fixing the root, we only move the bottleneck from the application to the database, and the next crash will be worse than this one.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| triệu chứng thường gặp nhất | the most common symptom |
| khi nó xin một kết nối từ pool | when it asks for a connection from the pool |
| phản xạ sai là | the wrong reflex is that |
| chỉ che triệu chứng chứ nó không chữa gốc | only hides the symptom and it does not cure the root |
| đang giữ kết nối quá lâu | holding a connection for too long |
| một request bắn ra năm mươi mốt câu truy vấn | one request that fires fifty-one queries |
| lâu gấp nhiều lần một request bình thường | many times longer than a normal request |
| quên trả nó về pool khi có lỗi | forgets to return it to the pool when an error happens |
| chỉ khi cả ba đều sạch | only when all three are clean |
| chúng ta chỉ chuyển điểm nghẽn từ … sang … | we only move the bottleneck from … to … |
| lần sập sau sẽ nặng hơn lần này | the next crash will be worse than this one |

**Thuật ngữ cần nhớ**

- cạn pool → **pool exhausted**
- hết thời gian chờ | quá hạn → **timeout**
- nguyên nhân gốc → **root cause**
- rò rỉ kết nối → **connection leak**
- điểm nghẽn → **bottleneck**

---

## Mô hình ghi nhớ

**Tiếng Việt**

Pool tồn tại để tái dùng kết nối, và một pool nhỏ mà đúng thì tốt hơn một pool lớn. Tổng của kích thước pool nhân với số instance phải lọt dưới max_connections, và khi pool cạn thì chúng ta sửa truy vấn trước chứ chúng ta không tăng pool.

**English (bám cấu trúc tiếng Việt)**

A pool exists in order to reuse connections, and a small but correct pool is better than a large one. The total of the pool size multiplied by the number of instances must sit under max_connections, and when the pool runs out we fix the queries first and we do not raise the pool.

---

## Bảng thuật ngữ tổng hợp

| Tiếng Việt | Tiếng Anh | Ghi chú phát âm |
|---|---|---|
| bắt tay giao thức | handshake | *hand* — âm **h** phải bật |
| xác thực | authentication | o-then-ti-**KAY**-shơn; "th" đầu lưỡi |
| tiến trình phía máy chủ | backend process | Anh **PROU**-ses |
| độ trễ | latency | /ˈleɪtənsi/ — "**LAY**-tơn-si" |
| thông lượng | throughput | /ˈθruːpʊt/ — âm **th** đầu lưỡi |
| quá tải | overloaded | ou-vơ-**LOU**-đid |
| kích thước pool | pool size | |
| năng lực xử lý | capacity | cơ-**PA**-sơ-ti — trọng âm 2 |
| chuyển ngữ cảnh | context switching | **CON**-text — trọng âm 1 |
| tranh chấp khoá | lock contention | cơn-**TEN**-shơn |
| công thức kinh nghiệm | rule of thumb | *thumb* /θʌm/ — chữ **b** câm, "th" đầu lưỡi |
| nhân xử lý | CPU core | /kɔː/ — "co" |
| ổ đĩa hiệu dụng | effective spindle | **SPIN**-đl |
| trình gom kết nối | connection pooler | |
| chế độ theo phiên | session mode | **SE**-shơn |
| chế độ theo transaction | transaction mode | tran-**SAC**-shơn |
| chế độ theo câu lệnh | statement mode | |
| tái dùng | reuse | ri-**YUZE** — đuôi /z/ |
| trạng thái phiên | session state | |
| biến phiên | session variable | **VEA**-ri-ơ-bl |
| khoá cố vấn | advisory lock | ơd-**VAI**-zơ-ri — trọng âm 2 |
| câu lệnh chuẩn bị sẵn | prepared statement | pri-**PEAD** |
| trình điều khiển kết nối | driver | **DRAI**-vơ |
| kiến trúc không máy chủ | serverless architecture | *architecture* **AR**-ki-tek-chơ — "ch" đọc /k/ |
| mở rộng ngang | scale out | |
| trần kết nối | the connection ceiling | *ceiling* /ˈsiːlɪŋ/ — "SII-ling", **c** đọc /s/ |
| cơn bão kết nối | connection storm | |
| pooler bên ngoài | external pooler | ex-**TER**-nơl |
| nhà cung cấp dịch vụ | the provider | prơ-**VAI**-đơ |
| phần dự phòng | the reserved part | ri-**ZERVD** |
| tài khoản quản trị cao nhất | superuser | |
| tự động mở rộng | autoscaling | |
| nhân bản dữ liệu | replication | re-pli-**CAY**-shơn |
| vỡ trận | break down | |
| triệu chứng | symptom | **SIM**-tơm — chữ **p** gần như câm |
| cạn pool | pool exhausted | ig-**ZOS**-tid — "x" đọc /gz/ |
| hết thời gian chờ | timeout | |
| xin một kết nối | acquire a connection | *acquire* ơ-**KWAI**-ơ — chữ **c** câm |
| nguyên nhân gốc | root cause | *root* Anh /ruːt/ — "rut" |
| rò rỉ kết nối | connection leak | /liːk/ — nguyên âm dài |
| trả kết nối về pool | release the connection | ri-**LEES** |
| điểm nghẽn | bottleneck | |
| sự cố | incident | **IN**-si-đơnt |

---

## Bài nói

> **Cách luyện:** bật ghi âm, nói **60–90 giây** cho mỗi đề, **không nhìn tài liệu**. Mỗi bài nói phải dùng **ít nhất 6 thuật ngữ** trong bảng trên. Nghe lại bản ghi một lần, đánh dấu chỗ ngập ngừng, rồi nói lại đề đó lần thứ hai.

1. Explain to a junior engineer why opening a new database connection for every request is expensive, and what a pool does about it.

2. A colleague says: "We are getting connection timeouts, so let us double the pool size on every instance." Explain why you would push back and what you would investigate first.

3. Your service autoscales from two to forty instances and each instance has a pool of thirty, while the database allows three hundred connections. Explain the problem to your team and describe two ways to fix it.

4. Describe what happens to a connection in PgBouncer transaction mode from the moment a request arrives to the moment it finishes.

5. A teammate wants to use session variables and advisory locks in a service that sits behind a transaction-mode pooler. Explain what will go wrong and why it will not show up in testing.

6. When would you choose session mode over transaction mode, and what do you give up by doing that?

7. Explain to a platform team why a serverless function needs an external pooler, and what would happen without one during a traffic spike.
