# Bài 7 — Redis làm cache: bản chất bên trong
### Bản song ngữ: tiếng Anh bám cấu trúc tiếng Việt

> **Cách dùng bài này:** đọc đoạn tiếng Việt trước → tự dịch trong đầu → so với đoạn tiếng Anh bên dưới → **đọc to bản tiếng Anh hai lần**: lần một đọc chậm cho đúng, lần hai đọc trôi như đang nói với người phỏng vấn.

---

## Phần 1 — Vì sao Redis nhanh dù chỉ chạy một luồng

**Tiếng Việt**

Chúng ta bắt đầu bằng câu hỏi kinh điển: vì sao Redis nhanh trong khi nó chỉ thực thi lệnh trên một luồng duy nhất. Lý do thứ nhất là toàn bộ dữ liệu nằm trong bộ nhớ, vì vậy trên đường đi nóng không có thao tác đọc ghi đĩa nào cả. Lý do thứ hai là chính việc chạy một luồng: không có khoá, không có chuyển ngữ cảnh giữa các luồng, và mọi thao tác đều nguyên tử mà không cần khoá. Lý do thứ ba là Redis dùng cơ chế ghép kênh vào ra, ví dụ epoll, để phục vụ rất nhiều kết nối một cách hiệu quả. Lý do thứ tư là các cấu trúc dữ liệu được viết bằng C và được mã hoá rất gọn. Phần có thể chậm nằm ở mạng và ở việc lưu bền, chứ không nằm ở việc thực thi lệnh. Nói cách khác, chạy một luồng không phải là điểm yếu mà là một lựa chọn thiết kế có chủ đích.

Chúng ta nên bổ sung một câu cập nhật, vì nó cho thấy chúng ta theo dõi tình hình hiện tại. Từ Redis 6 trở đi đã có các luồng vào ra chuyên xử lý việc đọc ghi socket song song, và các phiên bản 8 cùng Valkey 8 còn mạnh hơn nữa. Vì vậy câu nói Redis chỉ dùng được một lõi không còn đúng tuyệt đối. Nhưng phần thực thi lệnh vẫn tuần tự, và chính điều đó giữ được tính nguyên tử mà chúng ta vừa nói.

**English (bám cấu trúc tiếng Việt)**

We start with the classic question: why is Redis fast while it executes commands on one single thread. The first reason is that all the data sits in memory, therefore on the hot path there is no disk read or write at all. The second reason is single-threaded execution itself: there are no locks, there is no context switching between threads, and every operation is atomic without needing a lock. The third reason is that Redis uses an I/O multiplexing mechanism, for example epoll, to serve a great many connections efficiently. The fourth reason is that the data structures are written in C and are encoded very compactly. The part that can be slow lies in the network and in persistence, instead of lying in command execution. In other words, running on one thread is not a weakness but a deliberate design choice.

We should add one sentence to bring this up to date, because it shows that we follow the current situation. From Redis 6 onwards there have been I/O threads dedicated to reading and writing sockets in parallel, and version 8 together with Valkey 8 goes further still. Therefore the statement that Redis can only use one core is no longer absolutely true. But command execution is still sequential, and that is exactly what preserves the atomicity we have just mentioned.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| trên đường đi nóng | on the hot path |
| không có thao tác đọc ghi đĩa nào cả | there is no disk read or write at all |
| chuyển ngữ cảnh giữa các luồng | context switching between threads |
| nguyên tử mà không cần khoá | atomic without needing a lock |
| cơ chế ghép kênh vào ra | an I/O multiplexing mechanism |
| được mã hoá rất gọn | encoded very compactly |
| chứ không nằm ở việc thực thi lệnh | instead of lying in command execution |
| một lựa chọn thiết kế có chủ đích | a deliberate design choice |
| bổ sung một câu cập nhật | add one sentence to bring this up to date |
| còn mạnh hơn nữa | goes further still |
| không còn đúng tuyệt đối | no longer absolutely true |
| chính điều đó giữ được tính nguyên tử | that is exactly what preserves the atomicity |

**Thuật ngữ cần nhớ**

- chạy một luồng → **single-threaded**
- đường đi nóng → **the hot path**
- chuyển ngữ cảnh → **a context switch**
- ghép kênh vào ra → **I/O multiplexing**
- tính nguyên tử → **atomicity**

---

## Phần 2 — Hệ quả vận hành của việc thực thi tuần tự

**Tiếng Việt**

Hệ quả trực tiếp của việc thực thi tuần tự là một lệnh chậm sẽ chặn tất cả mọi người. Nếu một lệnh có độ phức tạp tuyến tính chạy trên một tập lớn, mọi client khác phải xếp hàng chờ nó xong. Ví dụ kinh điển nhất là lệnh liệt kê toàn bộ key theo mẫu, và chúng ta phải thay nó bằng lệnh quét theo con trỏ. Lệnh quét theo con trỏ trả về từng phần nhỏ, vì vậy nó không chặn luồng duy nhất. Tương tự, chúng ta tránh mọi thao tác trên key quá lớn, và chúng ta dùng lệnh xoá bất đồng bộ thay cho lệnh xoá thường. Chúng ta cũng phải giữ cho script Lua thật ngắn, vì một script chạy lâu cũng chặn y như một lệnh chậm. Cuối cùng, vì phần thực thi giới hạn ở khoảng một lõi, cách mở rộng đúng là chia dữ liệu ra nhiều shard chứ không phải mua một máy mạnh hơn.

Đây là chỗ chúng ta nên kể một câu chuyện thật nếu chúng ta có. Một lệnh liệt kê toàn bộ key chạy trên môi trường production có thể làm độ trễ của cả hệ thống tăng vọt trong vài giây. Điều đáng sợ là biểu đồ sẽ cho thấy Redis vẫn khoẻ, vì nó không hề bị lỗi mà nó chỉ đang bận. Vì vậy chúng ta nên cấm hẳn lệnh đó trên production, và nhiều dịch vụ quản lý cũng cho phép chúng ta tắt các lệnh nguy hiểm.

**English (bám cấu trúc tiếng Việt)**

The direct consequence of sequential execution is that one slow command will block everybody. If a command with linear complexity runs on a large set, every other client has to queue up and wait for it to finish. The most classic example is the command that lists every key matching a pattern, and we have to replace it with the cursor-based scan command. The cursor-based scan command returns one small chunk at a time, therefore it does not block the single thread. Similarly, we avoid every operation on a key that is too large, and we use the asynchronous delete command instead of the ordinary delete command. We also have to keep Lua scripts really short, because a long-running script blocks just like a slow command. Finally, because execution is limited to about one core, the correct way to scale is to split the data across many shards instead of buying a more powerful machine.

This is where we should tell a real story if we have one. A command that lists every key, run on the production environment, can make the latency of the whole system shoot up for several seconds. The frightening thing is that the dashboard will show Redis as healthy, because it is not failing at all, it is merely busy. Therefore we should ban that command outright on production, and many managed services also let us disable the dangerous commands.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| một lệnh chậm sẽ chặn tất cả mọi người | one slow command will block everybody |
| có độ phức tạp tuyến tính | with linear complexity |
| phải xếp hàng chờ nó xong | has to queue up and wait for it to finish |
| liệt kê toàn bộ key theo mẫu | lists every key matching a pattern |
| trả về từng phần nhỏ | returns one small chunk at a time |
| chặn y như một lệnh chậm | blocks just like a slow command |
| chia dữ liệu ra nhiều shard | split the data across many shards |
| chứ không phải mua một máy mạnh hơn | instead of buying a more powerful machine |
| làm độ trễ của cả hệ thống tăng vọt | make the latency of the whole system shoot up |
| điều đáng sợ là | the frightening thing is that |
| nó không hề bị lỗi mà nó chỉ đang bận | it is not failing at all, it is merely busy |
| cấm hẳn lệnh đó trên production | ban that command outright on production |

**Thuật ngữ cần nhớ**

- thực thi tuần tự → **sequential execution**
- độ phức tạp tuyến tính → **linear complexity**
- lệnh quét theo con trỏ → **the cursor-based scan command**
- chia thành nhiều shard → **to shard**
- tắt lệnh nguy hiểm → **to disable dangerous commands**

---

## Phần 3 — Redis so với Memcached

**Tiếng Việt**

Memcached và Redis giải cùng một bài toán nhưng theo hai triết lý khác nhau. Memcached rất đơn giản, nó chạy nhiều luồng, và nó chỉ lưu chuỗi theo cặp khoá và giá trị. Redis thì giàu tính năng hơn hẳn: nó có nhiều cấu trúc dữ liệu, có lưu bền, có nhân bản, có phát và nhận thông điệp, và có chạy script. Chúng ta chọn Memcached khi chúng ta chỉ cần một cache thuần rất đơn giản và muốn tận dụng nhiều lõi cho chuỗi. Chúng ta chọn Redis khi chúng ta cần cấu trúc dữ liệu, cần tính sẵn sàng cao, hoặc cần lưu bền. Trong phần lớn dự án mới hiện nay, người ta chọn Redis, hoặc chọn các bản thay thế như Valkey và DragonflyDB.

Khi trả lời câu này, chúng ta nên tránh nói rằng cái này tốt hơn cái kia. Cách nói tốt hơn là nêu ra điều kiện: nếu tải của chúng ta thuần là chuỗi và cần nhiều lõi, thì Memcached vẫn rất mạnh. Nếu chúng ta cần bảng xếp hạng, cần hàng đợi, hoặc cần thao tác nguyên tử phía máy chủ, thì Redis tiết kiệm cho chúng ta rất nhiều code. Cách nói theo điều kiện luôn được đánh giá cao hơn cách nói theo sở thích.

**English (bám cấu trúc tiếng Việt)**

Memcached and Redis solve the same problem but with two different philosophies. Memcached is very simple, it runs on multiple threads, and it only stores strings as key-value pairs. Redis is far richer in features: it has many data structures, it has persistence, it has replication, it has publishing and receiving of messages, and it has scripting. We choose Memcached when we only need a very simple pure cache and we want to make use of multiple cores for strings. We choose Redis when we need data structures, when we need high availability, or when we need persistence. In most new projects today, people choose Redis, or they choose the alternatives such as Valkey and DragonflyDB.

When we answer this question, we should avoid saying that one is better than the other. The better way to speak is to state the condition: if our load is purely strings and needs many cores, then Memcached is still very strong. If we need a leaderboard, need a queue, or need atomic operations on the server side, then Redis saves us a great deal of code. Speaking in terms of conditions is always valued more highly than speaking in terms of preference.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| theo hai triết lý khác nhau | with two different philosophies |
| lưu chuỗi theo cặp khoá và giá trị | stores strings as key-value pairs |
| giàu tính năng hơn hẳn | far richer in features |
| phát và nhận thông điệp | publishing and receiving of messages |
| muốn tận dụng nhiều lõi | want to make use of multiple cores |
| các bản thay thế | the alternatives |
| tránh nói rằng cái này tốt hơn cái kia | avoid saying that one is better than the other |
| cách nói tốt hơn là nêu ra điều kiện | the better way to speak is to state the condition |
| tiết kiệm cho chúng ta rất nhiều code | saves us a great deal of code |
| cách nói theo điều kiện | speaking in terms of conditions |
| được đánh giá cao hơn | is valued more highly |

**Thuật ngữ cần nhớ**

- chạy nhiều luồng → **multi-threaded**
- cặp khoá và giá trị → **a key-value pair**
- phát và đăng ký nhận → **publish and subscribe (pub/sub)**
- chạy script phía máy chủ → **server-side scripting**
- bản thay thế → **an alternative**

---

## Phần 4 — Chọn đúng cấu trúc dữ liệu

**Tiếng Việt**

Chọn đúng cấu trúc dữ liệu là cách rẻ nhất để cắt bớt logic trong app. Chuỗi dùng cho giá trị đã được tuần tự hoá, ví dụ một khối JSON, và dùng cho bộ đếm qua lệnh tăng. Hash dùng cho một đối tượng có nhiều trường, và nhờ đó chúng ta đọc hoặc ghi từng trường mà không phải lấy cả đối tượng. Sorted set dùng cho bảng xếp hạng, cho giới hạn tần suất theo thời gian, và cho việc lấy N phần tử đứng đầu. Set dùng cho một tập phần tử không trùng nhau và cho việc kiểm tra một phần tử có thuộc tập hay không. List dùng cho một hàng đợi đơn giản, với lệnh đẩy vào đầu và lệnh lấy ra ở cuối có chờ. Stream dùng cho nhật ký sự kiện và cho công việc chạy nền, và Bài 10 sẽ nói kỹ về nó. Bitmap và HyperLogLog dùng cho việc đánh dấu và cho việc đếm xấp xỉ số phần tử duy nhất, và Bài 8 sẽ nói kỹ về chúng.

Nguyên tắc của chúng ta rất ngắn: đừng làm trong app cái mà Redis đã làm sẵn. Nếu chúng ta lấy về mười nghìn phần tử rồi tự sắp xếp trong app để tìm mười người dẫn đầu, chúng ta vừa tốn băng thông vừa tốn bộ nhớ. Sorted set làm đúng việc đó ngay trong máy chủ với chi phí thấp hơn nhiều lần. Đây cũng là một câu hỏi review rất tốt: cấu trúc chúng ta chọn có hợp với thao tác chúng ta cần hay không.

**English (bám cấu trúc tiếng Việt)**

Choosing the right data structure is the cheapest way to cut logic out of the app. A string is used for a serialised value, for example a block of JSON, and for a counter through the increment command. A hash is used for an object with many fields, and thanks to that we read or write individual fields without having to fetch the whole object. A sorted set is used for a leaderboard, for rate limiting over time, and for taking the top N elements. A set is used for a collection of elements with no duplicates and for checking whether an element belongs to the collection or not. A list is used for a simple queue, with a command that pushes onto the head and a command that pops from the tail with waiting. A stream is used for an event log and for background jobs, and Lesson 10 will talk about it in detail. Bitmaps and HyperLogLog are used for flagging and for approximate counting of unique elements, and Lesson 8 will talk about them in detail.

Our principle is very short: do not do in the app what Redis already does for us. If we fetch ten thousand elements and then sort them in the app to find the top ten, we waste both bandwidth and memory. A sorted set does exactly that job inside the server at a cost that is many times lower. This is also a very good review question: does the structure we chose fit the operation we need.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| cắt bớt logic trong app | cut logic out of the app |
| giá trị đã được tuần tự hoá | a serialised value |
| qua lệnh tăng | through the increment command |
| đọc hoặc ghi từng trường | read or write individual fields |
| mà không phải lấy cả đối tượng | without having to fetch the whole object |
| lấy N phần tử đứng đầu | taking the top N elements |
| một tập phần tử không trùng nhau | a collection of elements with no duplicates |
| đẩy vào đầu … lấy ra ở cuối có chờ | pushes onto the head … pops from the tail with waiting |
| đếm xấp xỉ số phần tử duy nhất | approximate counting of unique elements |
| đừng làm trong app cái mà Redis đã làm sẵn | do not do in the app what Redis already does for us |
| vừa tốn băng thông vừa tốn bộ nhớ | waste both bandwidth and memory |
| với chi phí thấp hơn nhiều lần | at a cost that is many times lower |

**Thuật ngữ cần nhớ**

- tập hợp có sắp xếp → **a sorted set**
- bảng xếp hạng → **a leaderboard**
- kiểm tra thuộc tập → **a membership check**
- không trùng nhau, duy nhất → **unique** / **no duplicates**
- đếm xấp xỉ → **approximate counting**

---

## Phần 5 — Lưu bền: ảnh chụp và nhật ký ghi

**Tiếng Việt**

Redis có hai cơ chế lưu bền và chúng ta phải phân biệt chúng cho rõ. RDB là ảnh chụp định kỳ toàn bộ dữ liệu, vì vậy tệp gọn và việc khôi phục rất nhanh. Nhược điểm của RDB là chúng ta mất toàn bộ thay đổi kể từ ảnh chụp cuối cùng. AOF là nhật ký ghi lại từng lệnh ghi, vì vậy nó bền hơn nhiều nhưng tệp lớn hơn và tốc độ chậm hơn. AOF có nhiều mức đồng bộ xuống đĩa, ví dụ mỗi giây một lần hoặc mỗi lệnh một lần, và mỗi mức đổi độ bền lấy hiệu năng. Với một instance thuần làm cache, chúng ta có thể tắt hẳn lưu bền để chạy nhanh nhất, vì dữ liệu luôn tái tạo được từ DB.

Nhưng với một instance giữ session hoặc giữ hàng đợi công việc, chúng ta bắt buộc phải bật lưu bền. Đây chính là lý do chúng ta tách hai vai trò ra hai instance như đã nói ở Bài 6. Nếu chúng ta gộp chúng lại, chúng ta buộc phải chọn một cấu hình sai cho ít nhất một trong hai vai trò. Cách trả lời này cho thấy chúng ta nghĩ theo vai trò của dữ liệu chứ không nghĩ theo mặc định của công cụ.

**English (bám cấu trúc tiếng Việt)**

Redis has two persistence mechanisms and we have to tell them apart clearly. RDB is a periodic snapshot of all the data, therefore the file is compact and restoring is very fast. The drawback of RDB is that we lose all the changes made since the last snapshot. AOF is a log that records every write command, therefore it is far more durable but the file is larger and the speed is slower. AOF has several levels of syncing to disk, for example once every second or once per command, and each level trades durability for performance. For an instance that is purely a cache, we can turn persistence off entirely to run as fast as possible, because the data can always be rebuilt from the database.

But for an instance holding sessions or holding the job queue, we are obliged to turn persistence on. This is exactly the reason we split the two roles into two instances as we said in Lesson 6. If we merge them together, we are forced to choose a wrong configuration for at least one of the two roles. This way of answering shows that we think in terms of the role of the data instead of thinking in terms of the tool's defaults.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| ảnh chụp định kỳ toàn bộ dữ liệu | a periodic snapshot of all the data |
| tệp gọn và việc khôi phục rất nhanh | the file is compact and restoring is very fast |
| kể từ ảnh chụp cuối cùng | since the last snapshot |
| nhật ký ghi lại từng lệnh ghi | a log that records every write command |
| nhiều mức đồng bộ xuống đĩa | several levels of syncing to disk |
| đổi độ bền lấy hiệu năng | trades durability for performance |
| tắt hẳn lưu bền | turn persistence off entirely |
| chúng ta bắt buộc phải bật | we are obliged to turn on |
| chúng ta buộc phải chọn một cấu hình sai | we are forced to choose a wrong configuration |
| nghĩ theo vai trò của dữ liệu | think in terms of the role of the data |
| mặc định của công cụ | the tool's defaults |

**Thuật ngữ cần nhớ**

- ảnh chụp dữ liệu → **a snapshot**
- tệp nhật ký chỉ ghi thêm → **an append-only file (AOF)**
- độ bền dữ liệu → **durability**
- khôi phục → **to restore**
- đồng bộ xuống đĩa → **to sync to disk**

---

## Phần 6 — Pipelining khác transaction ở chỗ nào

**Tiếng Việt**

Rất nhiều người nhầm hai khái niệm này, vì vậy phân biệt được chúng là một điểm cộng. Pipelining nghĩa là chúng ta gộp nhiều lệnh rồi gửi chúng đi trong một lượt, để cắt bớt số vòng đi về trên mạng. Nó cải thiện thông lượng rất mạnh, nhưng nó không bảo đảm tính nguyên tử, vì lệnh của client khác vẫn có thể xen vào giữa. MULTI và EXEC thì ngược lại: chúng nhóm các lệnh thành một khối nguyên tử, và không lệnh nào của người khác chen vào giữa khối đó. Nhưng chúng ta phải nói rõ một điểm: khối này không có cơ chế quay lui như transaction trong SQL. Nếu một lệnh gặp lỗi lúc chạy, các lệnh còn lại vẫn được thực thi bình thường. Vì vậy hai công cụ này phục vụ hai mục đích khác nhau: pipeline tối ưu thông lượng, còn MULTI bảo đảm tính nguyên tử.

Câu hỏi bẫy hay gặp là hỏi chúng ta có dùng transaction để quay lui được hay không. Câu trả lời đúng là không, và đó là điểm khác biệt cốt lõi so với cơ sở dữ liệu quan hệ. Nếu chúng ta cần một logic nguyên tử có điều kiện, chúng ta nên viết một script Lua ngắn. Script chạy trên máy chủ như một khối, vì vậy nó cho chúng ta thứ mà MULTI không cho được.

**English (bám cấu trúc tiếng Việt)**

Very many people confuse these two concepts, therefore being able to tell them apart is a plus point. Pipelining means that we batch many commands and then send them off in one go, in order to cut down the number of round trips on the network. It improves throughput a great deal, but it does not guarantee atomicity, because commands from other clients can still slip in between. MULTI and EXEC are the opposite: they group the commands into one atomic block, and no command from anybody else cuts into the middle of that block. But we have to state one point clearly: this block has no rollback mechanism like a transaction in SQL. If one command hits an error at run time, the remaining commands are still executed normally. Therefore these two tools serve two different purposes: the pipeline optimises throughput, while MULTI guarantees atomicity.

The trap question we meet often is whether we can use a transaction to roll back. The correct answer is no, and that is the core difference compared with a relational database. If we need atomic logic with a condition in it, we should write a short Lua script. The script runs on the server as one block, therefore it gives us the thing that MULTI cannot give.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| phân biệt được chúng là một điểm cộng | being able to tell them apart is a plus point |
| gộp nhiều lệnh rồi gửi chúng đi trong một lượt | batch many commands and then send them off in one go |
| cắt bớt số vòng đi về trên mạng | cut down the number of round trips on the network |
| lệnh của client khác vẫn có thể xen vào giữa | commands from other clients can still slip in between |
| nhóm các lệnh thành một khối nguyên tử | group the commands into one atomic block |
| chen vào giữa khối đó | cuts into the middle of that block |
| không có cơ chế quay lui | has no rollback mechanism |
| gặp lỗi lúc chạy | hits an error at run time |
| phục vụ hai mục đích khác nhau | serve two different purposes |
| điểm khác biệt cốt lõi | the core difference |
| một logic nguyên tử có điều kiện | atomic logic with a condition in it |
| thứ mà MULTI không cho được | the thing that MULTI cannot give |

**Thuật ngữ cần nhớ**

- gộp lệnh gửi một lượt → **pipelining**
- vòng đi về trên mạng → **a network round trip**
- thông lượng → **throughput**
- quay lui giao dịch → **rollback**
- khối nguyên tử → **an atomic block**

---

## Phần 7 — Script Lua và kỷ luật đặt TTL

**Tiếng Việt**

Script Lua cho phép chúng ta chạy nhiều thao tác một cách nguyên tử ngay trên máy chủ. Ví dụ điển hình là giải phóng một khoá phân tán: chúng ta so sánh token rồi mới xoá, và hai bước đó phải nằm trong cùng một khối. Một ví dụ khác là giới hạn tần suất: chúng ta đếm rồi kiểm tra ngưỡng, và hai bước đó cũng phải nguyên tử. Nếu chúng ta tách chúng thành hai lệnh riêng, chúng ta tạo ra đúng cái race kiểm tra rồi mới hành động. Rủi ro của Lua là một script chạy lâu sẽ chặn luồng duy nhất, vì vậy chúng ta phải giữ script thật ngắn. Chuyển sang chủ đề TTL, chúng ta đặt thời hạn ngay trong lệnh ghi bằng tuỳ chọn tính theo giây hoặc theo mili giây, hoặc đặt sau bằng một lệnh riêng. Lỗi phổ biến nhất là quên đặt TTL, và khi đó key sống mãi, bộ nhớ phình lên, và dữ liệu cũ nằm lại vô thời hạn.

Chúng ta nên đặt ra một quy tắc cứng cho cả đội. Quy tắc đó là mọi giá trị cache đều phải có TTL, trừ khi có một lý do rõ ràng được ghi lại. Nếu một key cần sống mãi, thì nhiều khả năng nó không phải là dữ liệu cache mà là dữ liệu nghiệp vụ. Khi đó nó nên nằm ở một instance khác, hoặc nên nằm hẳn trong DB.

**English (bám cấu trúc tiếng Việt)**

A Lua script lets us run several operations atomically right on the server. The typical example is releasing a distributed lock: we compare the token and only then delete, and those two steps have to sit inside the same block. Another example is rate limiting: we count and then check the threshold, and those two steps also have to be atomic. If we split them into two separate commands, we create exactly the check-then-act race. The risk of Lua is that a long-running script will block the single thread, therefore we have to keep the script really short. Moving on to the topic of TTL, we set the deadline right inside the write command with an option counted in seconds or in milliseconds, or we set it afterwards with a separate command. The most common mistake is forgetting to set a TTL, and then the key lives forever, the memory swells up, and stale data stays there indefinitely.

We should set a hard rule for the whole team. That rule is that every cache value must have a TTL, unless there is a clear reason that has been written down. If a key needs to live forever, then most likely it is not cache data but business data. In that case it should sit in a different instance, or it should sit in the database altogether.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| chạy nhiều thao tác một cách nguyên tử | run several operations atomically |
| chúng ta so sánh token rồi mới xoá | we compare the token and only then delete |
| phải nằm trong cùng một khối | have to sit inside the same block |
| chúng ta đếm rồi kiểm tra ngưỡng | we count and then check the threshold |
| đúng cái race kiểm tra rồi mới hành động | exactly the check-then-act race |
| giữ script thật ngắn | keep the script really short |
| chuyển sang chủ đề TTL | moving on to the topic of TTL |
| đặt sau bằng một lệnh riêng | set it afterwards with a separate command |
| key sống mãi, bộ nhớ phình lên | the key lives forever, the memory swells up |
| nằm lại vô thời hạn | stays there indefinitely |
| một lý do rõ ràng được ghi lại | a clear reason that has been written down |
| nên nằm hẳn trong DB | should sit in the database altogether |

**Thuật ngữ cần nhớ**

- khoá phân tán → **a distributed lock**
- ngưỡng giới hạn → **a threshold**
- kiểm tra rồi mới hành động → **check-then-act**
- quy tắc cứng → **a hard rule**
- vô thời hạn → **indefinitely**

---

## Phần 8 — Góc tech lead: cùng một key cho mọi người dùng

**Tiếng Việt**

Thách đố của bài này là một lỗi mà công cụ AI tạo ra rất thường xuyên. Đoạn code trả về một đối tượng đã tuần tự hoá, nhưng nó dùng chung một key cho mọi người dùng. Lỗi ở đây là key thiếu các chiều phân biệt, ví dụ người dùng, tenant, ngôn ngữ và quyền. Hậu quả là dữ liệu của người này được trả về cho người khác, và đó vừa là lỗi đúng đắn vừa là lỗi bảo mật. Chỗ chúng ta phải kiểm tra rất cụ thể: key có chứa đủ mọi bối cảnh ảnh hưởng tới kết quả hay không. Đây chính là điểm chúng ta đã học ở Bài 3, và nó lặp lại ở đây vì nó quan trọng đến mức đó. Một lỗi hiệu năng chỉ làm chậm hệ thống, còn lỗi này làm rò dữ liệu giữa các tài khoản.

Chúng ta có thể gói cả bài thành bốn mục kiểm tra khi review code Redis. Mục thứ nhất là key có đủ chiều phân biệt hay không. Mục thứ hai là có lệnh chặn nào, ví dụ lệnh liệt kê toàn bộ key, hay không. Mục thứ ba là mọi giá trị cache có TTL hay không. Mục thứ tư là đoạn code này có coi MULTI như một transaction có quay lui hay không, vì đó là một hiểu lầm rất phổ biến.

**English (bám cấu trúc tiếng Việt)**

The challenge of this lesson is a mistake that AI tools produce very often. The code returns a serialised object, but it uses the same key for every user. The mistake here is that the key is missing the distinguishing dimensions, for example the user, the tenant, the language and the permissions. The consequence is that one person's data is returned to another person, and that is both a correctness bug and a security bug. The place we have to check is very concrete: does the key contain every piece of context that affects the result. This is exactly the point we learned in Lesson 3, and it comes back here because it is that important. A performance bug only makes the system slow, while this bug leaks data between accounts.

We can wrap this whole lesson into four check items when we review Redis code. The first item is whether the key has all the distinguishing dimensions. The second item is whether there is any blocking command, for example the command that lists every key. The third item is whether every cache value has a TTL. The fourth item is whether this code treats MULTI as a transaction with rollback, because that is a very common misunderstanding.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| một lỗi mà công cụ AI tạo ra rất thường xuyên | a mistake that AI tools produce very often |
| key thiếu các chiều phân biệt | the key is missing the distinguishing dimensions |
| vừa là lỗi đúng đắn vừa là lỗi bảo mật | both a correctness bug and a security bug |
| chỗ chúng ta phải kiểm tra rất cụ thể | the place we have to check is very concrete |
| mọi bối cảnh ảnh hưởng tới kết quả | every piece of context that affects the result |
| nó lặp lại ở đây vì nó quan trọng đến mức đó | it comes back here because it is that important |
| làm rò dữ liệu giữa các tài khoản | leaks data between accounts |
| gói cả bài thành bốn mục kiểm tra | wrap this whole lesson into four check items |
| có lệnh chặn nào hay không | whether there is any blocking command |
| có coi MULTI như một transaction có quay lui hay không | whether it treats MULTI as a transaction with rollback |
| một hiểu lầm rất phổ biến | a very common misunderstanding |

**Thuật ngữ cần nhớ**

- chiều phân biệt → **a distinguishing dimension**
- lỗi đúng đắn → **a correctness bug**
- lỗi bảo mật → **a security bug**
- mục kiểm tra khi review → **a review check item**
- hiểu lầm → **a misunderstanding**

---

## Mô hình ghi nhớ

**Tiếng Việt**

Redis nhanh nhờ dữ liệu nằm trong bộ nhớ và nhờ thực thi tuần tự, vì vậy mọi thao tác đều nguyên tử mà không cần khoá. Đổi lại, một lệnh chậm sẽ chặn tất cả mọi người, nên chúng ta dùng lệnh quét theo con trỏ, giữ script thật ngắn, chọn đúng cấu trúc dữ liệu, và luôn đặt TTL.

**English (bám cấu trúc tiếng Việt)**

Redis is fast thanks to the data sitting in memory and thanks to sequential execution, therefore every operation is atomic without needing a lock. In exchange, one slow command will block everybody, so we use the cursor-based scan command, keep scripts really short, choose the right data structure, and always set a TTL.

---

## Bảng thuật ngữ tổng hợp

| Tiếng Việt | Tiếng Anh | Ghi chú phát âm (Anh-Anh) |
|---|---|---|
| bộ nhớ đệm | cache | /kæʃ/ — đọc y hệt "cash" |
| chạy một luồng | single-threaded | thread /θred/ — âm /θ/, không phải "t" |
| đường đi nóng | the hot path | path Anh-Anh /pɑːθ/ — "paath" |
| chuyển ngữ cảnh | a context switch | **KON**-tekst — kết thúc bằng cụm /kst/ |
| ghép kênh vào ra | I/O multiplexing | **MUL**-ti-pleks-ing |
| tính nguyên tử | atomicity | at-ə-**MIS**-ə-ti — trọng âm âm tiết thứ ba |
| có tính nguyên tử | atomic | ə-**TOM**-ik |
| thực thi tuần tự | sequential execution | si-**KWEN**-shəl |
| độ phức tạp tuyến tính | linear complexity | **LIN**-i-ə |
| lệnh quét theo con trỏ | the cursor-based scan command | cursor — **KER**-sə |
| chia thành nhiều shard | to shard | /ʃɑːd/ — bắt đầu bằng /ʃ/ |
| tắt, vô hiệu hoá | to disable | dis-**AI**-bl |
| chạy nhiều luồng | multi-threaded | |
| cặp khoá và giá trị | a key-value pair | |
| phát và đăng ký nhận | publish and subscribe (pub/sub) | səb-**SKRAIB** |
| chạy script phía máy chủ | server-side scripting | |
| bản thay thế | an alternative | awl-**TER**-nə-tiv — trọng âm âm tiết thứ hai |
| tính sẵn sàng cao | high availability | ə-veil-ə-**BIL**-ə-ti |
| tuần tự hoá dữ liệu | to serialise | **SEER**-i-ə-laiz |
| bộ đếm | a counter | |
| tăng lên một | to increment | **IN**-kri-mənt |
| tập hợp có sắp xếp | a sorted set | |
| bảng xếp hạng | a leaderboard | |
| kiểm tra thuộc tập | a membership check | **MEM**-bə-ship |
| không trùng nhau, duy nhất | unique | yoo-**NEEK** — trọng âm cuối |
| đếm xấp xỉ | approximate counting | ə-**PROK**-si-mət |
| hàng đợi | a queue | /kjuː/ — đọc đúng như chữ cái **Q** |
| nhật ký sự kiện | an event log | i-**VENT** |
| ảnh chụp dữ liệu | a snapshot | |
| tệp nhật ký chỉ ghi thêm | an append-only file (AOF) | ə-**PEND** — trọng âm cuối |
| độ bền dữ liệu | durability | dyoo-rə-**BIL**-ə-ti — Anh-Anh có /dj/ |
| khôi phục | to restore | ri-**STOR** |
| đồng bộ xuống đĩa | to sync to disk | |
| lưu bền | persistence | pə-**SIS**-təns |
| gộp lệnh gửi một lượt | pipelining | **PAIP**-lain-ing |
| vòng đi về trên mạng | a network round trip | |
| thông lượng | throughput | **THROO**-put — âm /θ/ đầu lưỡi |
| giao dịch | a transaction | tran-**ZAK**-shən — âm giữa là /z/ |
| quay lui giao dịch | rollback | |
| khối nguyên tử | an atomic block | |
| khoá phân tán | a distributed lock | dis-**TRIB**-yoo-tid |
| ngưỡng giới hạn | a threshold | **THRESH**-həuld |
| kiểm tra rồi mới hành động | check-then-act | |
| điều kiện tranh chấp | a race condition | |
| quy tắc cứng | a hard rule | |
| vô thời hạn | indefinitely | in-**DEF**-i-nət-li |
| thời gian sống | time to live (TTL) | đọc rời chữ cái: "tee-tee-el" |
| chiều phân biệt | a distinguishing dimension | dis-**TING**-gwish-ing |
| khách hàng dùng chung hệ thống | a tenant | **TEN**-ənt |
| ngôn ngữ và vùng | locale | ləʊ-**KAAL** — trọng âm cuối |
| quyền | permissions | pə-**MISH**-ənz |
| lỗi đúng đắn | a correctness bug | |
| lỗi bảo mật | a security bug | si-**KYOO**-rə-ti |
| hiểu lầm | a misunderstanding | |
| dịch vụ quản lý sẵn | a managed service | **MAN**-ijd |

---

## Bài nói

> **Cách luyện:** bật ghi âm, nói **60–90 giây** cho mỗi đề, **không nhìn tài liệu**. Mỗi bài nói phải dùng ít nhất **sáu thuật ngữ** trong bảng trên. Nghe lại một lần, đánh dấu chỗ bạn ngập ngừng, rồi nói lại đúng đề đó thêm một lần nữa.

1. **Explain to a junior** why Redis is fast even though it executes commands on a single thread, and what part of the system can still be slow.

2. **A colleague ran the command** that lists every key on production to find some keys, and latency spiked across the whole service. Explain what happened, why the dashboard still showed Redis as healthy, and what they should use instead.

3. **A teammate says** that MULTI and EXEC work like a SQL transaction, so a failed command will roll the whole block back. Explain why you would push back and what you would use for conditional atomic logic.

4. **When would you still choose** Memcached over Redis today? Answer in terms of conditions rather than preference.

5. **Someone on your team wants** to fetch ten thousand members and sort them in the application to build a top-ten leaderboard. Explain why you would push back and what you would do instead.

6. **Explain the difference** between RDB and AOF, and say what you would configure for a pure cache instance versus a session store.

7. **You are reviewing AI-generated caching code** that uses the same key for every user. Explain what is wrong, what the consequence is, and the four things you check in any Redis review.
