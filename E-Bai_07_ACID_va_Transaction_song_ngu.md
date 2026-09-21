# Bài 7 — ACID & Transaction
### Bản song ngữ: tiếng Anh bám cấu trúc tiếng Việt

> **Cách dùng bài này:** đọc đoạn tiếng Việt trước, tự dịch trong đầu, rồi mới so với đoạn tiếng Anh bên dưới. Chỗ nào lệch thì xem bảng ánh xạ cụm khó. Cuối cùng, **đọc to đoạn tiếng Anh hai lần** — một lần chậm để đúng âm, một lần với tốc độ nói thật.

---

## Phần 1 — Tính nguyên tử: hoặc tất cả, hoặc không gì cả

**Tiếng Việt**

Một transaction là một nhóm thao tác mà cơ sở dữ liệu đối xử như một đơn vị duy nhất. Ví dụ kinh điển là chúng ta chuyển một trăm nghìn đồng từ tài khoản A sang tài khoản B, trong đó chúng ta trừ tiền ở A và chúng ta cộng tiền ở B. Tính nguyên tử nghĩa là hai thao tác đó cùng xảy ra hoặc cùng không xảy ra, và không bao giờ có một trạng thái nằm ở giữa. Nếu hệ thống sập ngay sau khi trừ tiền ở A, cơ sở dữ liệu phải hoàn tác thao tác đó khi nó khởi động lại. Nếu chúng ta không có tính nguyên tử, một trăm nghìn đồng sẽ biến mất khỏi hệ thống, và không có cách nào tự động lấy lại được. Đây là lý do tính đúng đắn phải được tách hẳn khỏi tốc độ khi chúng ta thiết kế. Một hệ thống chậm thì làm khách hàng khó chịu, nhưng một hệ thống làm mất tiền thì làm công ty mất giấy phép.

**English (bám cấu trúc tiếng Việt)**

A transaction is a group of operations that the database treats as one single unit. The classic example is that we transfer one hundred thousand dong from account A to account B, in which we subtract the money at A and we add the money at B. Atomicity means that those two operations either both happen or both do not happen, and there is never a state sitting in between. If the system crashes right after the money is subtracted at A, the database has to undo that operation when it starts up again. If we do not have atomicity, one hundred thousand dong will disappear from the system, and there is no way to get it back automatically. This is the reason why correctness has to be kept completely separate from speed when we design. A slow system makes the customer unhappy, but a system that loses money makes the company lose its licence.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| mà cơ sở dữ liệu đối xử như một đơn vị duy nhất | that the database treats as one single unit |
| chúng ta trừ tiền ở A | we subtract the money at A |
| cùng xảy ra hoặc cùng không xảy ra | either both happen or both do not happen |
| một trạng thái nằm ở giữa | a state sitting in between |
| ngay sau khi trừ tiền ở A | right after the money is subtracted at A |
| phải hoàn tác thao tác đó | has to undo that operation |
| khi nó khởi động lại | when it starts up again |
| không có cách nào tự động lấy lại được | there is no way to get it back automatically |
| phải được tách hẳn khỏi tốc độ | has to be kept completely separate from speed |
| làm công ty mất giấy phép | makes the company lose its licence |

**Thuật ngữ cần nhớ**

- tính nguyên tử → **atomicity**
- hoàn tác → **undo / roll back**
- tính đúng đắn → **correctness**
- đơn vị duy nhất → **a single unit**
- sập hệ thống → **crash**

---

## Phần 2 — Tính nhất quán và tính cô lập trong ACID

**Tiếng Việt**

Chữ C trong ACID là tính nhất quán, và nó có nghĩa hẹp hơn nhiều so với điều mà người ta thường tưởng. Nó nói rằng sau khi transaction kết thúc, mọi ràng buộc và mọi bất biến của dữ liệu vẫn phải đúng. Trong ví dụ chuyển tiền, bất biến là tổng số tiền của cả hai tài khoản không được tự sinh ra và không được tự mất đi. Các ràng buộc mà cơ sở dữ liệu kiểm tra giúp chúng ta ở đây là khoá ngoại, ràng buộc duy nhất và ràng buộc kiểm tra giá trị. Chữ I là tính cô lập, và nó nói rằng các transaction chạy đồng thời phải cho ra kết quả như thể chúng đã chạy lần lượt. Nếu không có tính cô lập, hai người cùng rút tiền một lúc có thể cùng đọc được số dư cũ và cùng rút thành công. Mức độ cô lập trong thực tế có nhiều nấc khác nhau, và chúng ta sẽ nói kỹ về chúng trong bài tiếp theo.

**English (bám cấu trúc tiếng Việt)**

The letter C in ACID is consistency, and it has a much narrower meaning than what people usually assume. It says that after the transaction ends, every constraint and every invariant of the data must still hold. In the money transfer example, the invariant is that the total amount across both accounts must not create itself out of nothing and must not lose itself. The constraints that the database checks for us here are foreign keys, unique constraints and check constraints. The letter I is isolation, and it says that transactions running at the same time must produce a result as if they had run one after another. Without isolation, two people withdrawing money at the same moment could both read the old balance and both withdraw successfully. The isolation level in practice comes in several different steps, and we will talk about them in detail in the next lesson.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| có nghĩa hẹp hơn nhiều so với điều mà người ta thường tưởng | has a much narrower meaning than what people usually assume |
| mọi bất biến của dữ liệu vẫn phải đúng | every invariant of the data must still hold |
| không được tự sinh ra và không được tự mất đi | must not create itself out of nothing and must not lose itself |
| ràng buộc kiểm tra giá trị | check constraint |
| các transaction chạy đồng thời | transactions running at the same time |
| như thể chúng đã chạy lần lượt | as if they had run one after another |
| hai người cùng rút tiền một lúc | two people withdrawing money at the same moment |
| cùng đọc được số dư cũ | both read the old balance |
| có nhiều nấc khác nhau | comes in several different steps |
| chúng ta sẽ nói kỹ về chúng | we will talk about them in detail |

**Thuật ngữ cần nhớ**

- tính nhất quán → **consistency**
- bất biến → **invariant**
- tính cô lập → **isolation**
- mức cô lập → **isolation level**
- số dư tài khoản → **account balance**

---

## Phần 3 — Tính bền vững và write-ahead log

**Tiếng Việt**

Chữ D là tính bền vững, và nó nói rằng một khi transaction đã commit thì dữ liệu không được mất, kể cả khi máy chủ mất điện ngay sau đó. Cơ sở dữ liệu bảo đảm điều này bằng một kỹ thuật tên là write-ahead log. Trước khi thay đổi các trang dữ liệu thật, cơ sở dữ liệu ghi mô tả của thay đổi vào một tệp log tuần tự. Sau đó nó gọi fsync để buộc hệ điều hành đẩy dữ liệu từ bộ đệm xuống đĩa thật, và chỉ khi đó lệnh commit mới được coi là thành công. Nếu máy chủ sập, cơ sở dữ liệu đọc lại tệp log khi khởi động và nó phát lại mọi thay đổi đã commit. Cách này nhanh vì ghi tuần tự vào một tệp rẻ hơn nhiều so với ghi ngẫu nhiên vào hàng nghìn trang dữ liệu nằm rải rác. Chính tệp log này cũng là nền tảng cho việc nhân bản dữ liệu, vì một máy chủ khác chỉ cần đọc log và phát lại để bám theo máy chủ chính.

**English (bám cấu trúc tiếng Việt)**

The letter D is durability, and it says that once a transaction has committed the data must not be lost, even if the server loses power right afterwards. The database guarantees this with a technique called the write-ahead log. Before it changes the real data pages, the database writes a description of the change into a sequential log file. After that it calls fsync to force the operating system to push the data from the buffer down to the real disk, and only then is the commit command considered successful. If the server crashes, the database reads the log file again at start-up and it replays every committed change. This approach is fast because writing sequentially into one file is much cheaper than writing randomly into thousands of data pages lying scattered around. This very log file is also the foundation for replication, because another server only needs to read the log and replay it in order to follow the main server.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| một khi transaction đã commit | once a transaction has committed |
| kể cả khi máy chủ mất điện ngay sau đó | even if the server loses power right afterwards |
| một kỹ thuật tên là | a technique called |
| ghi mô tả của thay đổi vào một tệp log tuần tự | writes a description of the change into a sequential log file |
| buộc hệ điều hành đẩy dữ liệu từ bộ đệm xuống đĩa thật | force the operating system to push the data from the buffer down to the real disk |
| chỉ khi đó lệnh commit mới được coi là thành công | only then is the commit command considered successful |
| nó phát lại mọi thay đổi đã commit | it replays every committed change |
| nằm rải rác | lying scattered around |
| chính tệp log này cũng là nền tảng cho | this very log file is also the foundation for |
| để bám theo máy chủ chính | in order to follow the main server |

**Thuật ngữ cần nhớ**

- tính bền vững → **durability**
- nhật ký ghi trước → **write-ahead log (WAL)**
- ghi tuần tự → **sequential write**
- phát lại → **replay**
- nhân bản dữ liệu → **replication**

---

## Phần 4 — Bẫy thuật ngữ: chữ C của ACID khác chữ C của CAP

**Tiếng Việt**

Có một cái bẫy thuật ngữ mà người phỏng vấn rất thích dùng, đó là chữ C trong ACID và chữ C trong CAP. Hai chữ này viết giống hệt nhau nhưng chúng nói về hai chuyện hoàn toàn khác nhau. Chữ C trong ACID nói về ràng buộc dữ liệu bên trong một cơ sở dữ liệu, nghĩa là mọi bất biến vẫn đúng sau mỗi transaction. Chữ C trong CAP nói về việc mọi node trong một hệ phân tán đều nhìn thấy giá trị mới nhất, và tên chính xác của tính chất đó là linearizability. Vì vậy, một hệ thống có thể có ACID đầy đủ trên từng node mà vẫn không có tính nhất quán theo nghĩa của CAP. Nếu chúng ta lẫn hai khái niệm này khi trả lời, người phỏng vấn sẽ kết luận rằng chúng ta học thuộc chữ viết tắt chứ chưa hiểu nội dung.

**English (bám cấu trúc tiếng Việt)**

There is a terminology trap that interviewers really like to use, namely the letter C in ACID and the letter C in CAP. These two letters are written exactly the same but they talk about two completely different things. The C in ACID talks about the data constraints inside one database, which means that every invariant still holds after each transaction. The C in CAP talks about every node in a distributed system seeing the newest value, and the precise name of that property is linearizability. Therefore, a system can have full ACID on each node and still not have consistency in the CAP sense. If we mix these two concepts up when we answer, the interviewer will conclude that we have memorised the acronym without understanding the content.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| một cái bẫy thuật ngữ | a terminology trap |
| viết giống hệt nhau | are written exactly the same |
| hai chuyện hoàn toàn khác nhau | two completely different things |
| tên chính xác của tính chất đó là | the precise name of that property is |
| có ACID đầy đủ trên từng node | have full ACID on each node |
| theo nghĩa của CAP | in the CAP sense |
| nếu chúng ta lẫn hai khái niệm này | if we mix these two concepts up |
| học thuộc chữ viết tắt | have memorised the acronym |
| chứ chưa hiểu nội dung | without understanding the content |

**Thuật ngữ cần nhớ**

- chữ viết tắt → **acronym**
- tính tuyến tính hoá → **linearizability**
- hệ phân tán → **distributed system**
- giá trị mới nhất → **the newest value**
- nhầm lẫn hai khái niệm → **mix up two concepts**

---

## Phần 5 — Transaction phải ngắn: mở muộn, commit sớm

**Tiếng Việt**

Một transaction chạy lâu là một trong những nguồn sự cố phổ biến nhất trong production. Lý do thứ nhất là nó giữ khoá trên các dòng mà nó đã chạm vào, vì vậy mọi transaction khác muốn ghi vào cùng dòng đó đều phải xếp hàng chờ. Lý do thứ hai là nó giữ một snapshot cũ, và cơ sở dữ liệu không được phép dọn những phiên bản dữ liệu cũ hơn snapshot đó. Hậu quả là bảng và index phình lên, hiện tượng này được gọi là bloat, và nó làm mọi truy vấn khác chậm dần đi. Lý do thứ ba là transaction càng dài thì càng có nhiều cơ hội để hai transaction khoá chéo nhau và tạo ra deadlock. Quy tắc thực hành rất ngắn gọn, đó là chúng ta mở transaction muộn nhất có thể và chúng ta commit sớm nhất có thể. Cụ thể, chúng ta chuẩn bị mọi dữ liệu và mọi tính toán trước khi mở transaction, chứ chúng ta không bọc cả một request trong một transaction cho chắc.

**English (bám cấu trúc tiếng Việt)**

A long-running transaction is one of the most common sources of incidents in production. The first reason is that it holds locks on the rows it has touched, therefore every other transaction that wants to write to the same row has to queue up and wait. The second reason is that it holds an old snapshot, and the database is not allowed to clean up the data versions that are older than that snapshot. The consequence is that tables and indexes grow fat, this phenomenon is called bloat, and it makes every other query gradually slower. The third reason is that the longer a transaction is, the more chances there are for two transactions to lock across each other and create a deadlock. The practical rule is very short, namely that we open the transaction as late as possible and we commit as early as possible. Concretely, we prepare all the data and all the calculations before we open the transaction, and we do not wrap a whole request inside one transaction just to be safe.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| một transaction chạy lâu | a long-running transaction |
| nguồn sự cố phổ biến nhất | the most common source of incidents |
| các dòng mà nó đã chạm vào | the rows it has touched |
| đều phải xếp hàng chờ | has to queue up and wait |
| không được phép dọn | is not allowed to clean up |
| hiện tượng này được gọi là bloat | this phenomenon is called bloat |
| làm mọi truy vấn khác chậm dần đi | makes every other query gradually slower |
| càng dài thì càng có nhiều cơ hội để | the longer it is, the more chances there are for |
| hai transaction khoá chéo nhau | two transactions lock across each other |
| mở transaction muộn nhất có thể | open the transaction as late as possible |
| chúng ta không bọc cả một request trong một transaction | we do not wrap a whole request inside one transaction |

**Thuật ngữ cần nhớ**

- transaction chạy lâu → **long-running transaction**
- ảnh chụp dữ liệu → **snapshot**
- phình bảng do bản cũ chưa dọn → **bloat**
- khoá chết → **deadlock**
- xếp hàng chờ → **queue up**

---

## Phần 6 — Savepoint và khi nào chúng ta không cần transaction tường minh

**Tiếng Việt**

Đôi khi chúng ta muốn huỷ một phần của transaction mà vẫn giữ những gì đã làm trước đó. Công cụ cho việc này là savepoint, tức là một điểm đánh dấu bên trong transaction mà chúng ta có thể quay về. Chúng ta đặt một savepoint, chúng ta chạy một bước có thể thất bại, và nếu bước đó hỏng thì chúng ta rollback về savepoint chứ chúng ta không huỷ cả transaction. Cần lưu ý rằng PostgreSQL có subtransaction nhưng nó không cho phép một transaction con commit độc lập với transaction cha. Ở chiều ngược lại, chúng ta cũng nên biết khi nào chúng ta không cần một transaction tường minh. Một câu lệnh đơn lẻ đã tự nó là nguyên tử nhờ chế độ autocommit, vì vậy chúng ta chỉ cần transaction tường minh khi nhiều thao tác phải nguyên tử cùng với nhau.

**English (bám cấu trúc tiếng Việt)**

Sometimes we want to cancel a part of a transaction while still keeping what we have done before that. The tool for this is the savepoint, that is a marker inside the transaction that we can go back to. We set a savepoint, we run a step that may fail, and if that step goes wrong then we roll back to the savepoint and we do not cancel the whole transaction. We should note that PostgreSQL has subtransactions but it does not allow a child transaction to commit independently of the parent transaction. In the opposite direction, we should also know when we do not need an explicit transaction. A single statement is already atomic by itself thanks to autocommit mode, therefore we only need an explicit transaction when several operations have to be atomic together.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| huỷ một phần của transaction | cancel a part of a transaction |
| mà vẫn giữ những gì đã làm trước đó | while still keeping what we have done before that |
| một điểm đánh dấu bên trong transaction | a marker inside the transaction |
| mà chúng ta có thể quay về | that we can go back to |
| một bước có thể thất bại | a step that may fail |
| nếu bước đó hỏng | if that step goes wrong |
| cần lưu ý rằng | we should note that |
| commit độc lập với transaction cha | commit independently of the parent transaction |
| ở chiều ngược lại | in the opposite direction |
| đã tự nó là nguyên tử | is already atomic by itself |
| phải nguyên tử cùng với nhau | have to be atomic together |

**Thuật ngữ cần nhớ**

- điểm lưu trong transaction → **savepoint**
- transaction con → **subtransaction**
- tường minh → **explicit**
- tự động commit → **autocommit**
- quay lui về điểm lưu → **roll back to the savepoint**

---

## Phần 7 — Anti-pattern: gọi API bên ngoài bên trong transaction

**Tiếng Việt**

Một anti-pattern rất hay gặp là chúng ta mở transaction, rồi chúng ta gọi một API bên ngoài ngay bên trong transaction đó. Ví dụ, chúng ta mở transaction, chúng ta gọi cổng thanh toán mất hai giây, rồi chúng ta mới commit. Trong suốt hai giây đó, transaction vẫn giữ khoá trên các dòng và nó vẫn chiếm một kết nối trong pool. Nếu cổng thanh toán chậm hoặc treo, chúng ta không chỉ mất một request, mà chúng ta làm cạn pool kết nối cho cả hệ thống. Ngoài ra, lời gọi mạng có thể thành công trong khi transaction lại rollback, và lúc đó dữ liệu của chúng ta lệch với hệ thống bên ngoài. Cách sửa là chúng ta đẩy mọi tác dụng phụ ra ngoài transaction, và chúng ta thực hiện chúng sau khi commit thành công. Nếu tác dụng phụ đó bắt buộc phải xảy ra, chúng ta ghi nó vào một bảng outbox trong cùng transaction, rồi một worker riêng sẽ đọc bảng đó và gửi đi.

**English (bám cấu trúc tiếng Việt)**

A very common anti-pattern is that we open a transaction, then we call an external API right inside that transaction. For example, we open the transaction, we call the payment gateway which takes two seconds, and then we commit. During those whole two seconds, the transaction still holds locks on the rows and it still occupies one connection in the pool. If the payment gateway is slow or hangs, we do not only lose one request, but we drain the connection pool for the entire system. In addition, the network call may succeed while the transaction rolls back, and at that point our data is out of step with the external system. The fix is that we push every side effect out of the transaction, and we perform them after a successful commit. If that side effect absolutely must happen, we write it into an outbox table within the same transaction, then a separate worker will read that table and send it out.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| một anti-pattern rất hay gặp | a very common anti-pattern |
| ngay bên trong transaction đó | right inside that transaction |
| chúng ta gọi cổng thanh toán mất hai giây | we call the payment gateway which takes two seconds |
| trong suốt hai giây đó | during those whole two seconds |
| nó vẫn chiếm một kết nối trong pool | it still occupies one connection in the pool |
| nếu cổng thanh toán chậm hoặc treo | if the payment gateway is slow or hangs |
| chúng ta làm cạn pool kết nối cho cả hệ thống | we drain the connection pool for the entire system |
| dữ liệu của chúng ta lệch với hệ thống bên ngoài | our data is out of step with the external system |
| đẩy mọi tác dụng phụ ra ngoài transaction | push every side effect out of the transaction |
| nếu tác dụng phụ đó bắt buộc phải xảy ra | if that side effect absolutely must happen |

**Thuật ngữ cần nhớ**

- cách làm phản mẫu → **anti-pattern**
- cổng thanh toán → **payment gateway**
- tác dụng phụ → **side effect**
- bảng gửi đi → **outbox table**
- tiến trình chạy nền → **worker**

---

## Phần 8 — Nghiệp vụ trải nhiều service: Saga và Outbox thay cho 2PC

**Tiếng Việt**

Khi một nghiệp vụ trải trên nhiều service và nhiều cơ sở dữ liệu, chúng ta không còn một transaction duy nhất nữa. Giải pháp cổ điển cho tình huống này là two-phase commit, trong đó một điều phối viên hỏi tất cả các bên rồi mới ra lệnh commit. Ngày nay chúng ta thường tránh two-phase commit, vì nó giữ khoá trong suốt cả hai pha và vì nó chịu lỗi rất kém. Nếu điều phối viên chết giữa chừng, các bên còn lại sẽ treo với khoá đang giữ và không ai biết phải làm gì tiếp theo. Cách làm hiện nay là saga, trong đó chúng ta chia nghiệp vụ thành nhiều bước cục bộ, và mỗi bước có một bước bù trừ để hoàn tác khi có lỗi. Saga thường đi cùng với outbox, vì chúng ta ghi sự kiện vào bảng outbox trong cùng transaction với dữ liệu, nhờ đó sự kiện không bao giờ bị mất. Đổi lại, chúng ta chấp nhận nhất quán cuối cùng, và chúng ta phải làm cho mỗi bước an toàn khi bị lặp lại bằng một idempotency key.

**English (bám cấu trúc tiếng Việt)**

When a business flow spreads across many services and many databases, we no longer have one single transaction. The classic solution for this situation is two-phase commit, in which one coordinator asks all the parties before it gives the order to commit. Nowadays we usually avoid two-phase commit, because it holds locks throughout both phases and because it handles failure very poorly. If the coordinator dies halfway through, the remaining parties will hang with the locks they are holding and nobody knows what to do next. The current approach is the saga, in which we split the business flow into many local steps, and each step has a compensating step to undo it when something goes wrong. A saga usually goes together with an outbox, because we write the event into the outbox table within the same transaction as the data, thanks to which the event is never lost. In exchange, we accept eventual consistency, and we have to make each step safe when it is repeated by using an idempotency key.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| khi một nghiệp vụ trải trên nhiều service | when a business flow spreads across many services |
| một điều phối viên hỏi tất cả các bên | one coordinator asks all the parties |
| rồi mới ra lệnh commit | before it gives the order to commit |
| nó giữ khoá trong suốt cả hai pha | it holds locks throughout both phases |
| nó chịu lỗi rất kém | it handles failure very poorly |
| nếu điều phối viên chết giữa chừng | if the coordinator dies halfway through |
| sẽ treo với khoá đang giữ | will hang with the locks they are holding |
| chia nghiệp vụ thành nhiều bước cục bộ | split the business flow into many local steps |
| một bước bù trừ để hoàn tác khi có lỗi | a compensating step to undo it when something goes wrong |
| nhờ đó sự kiện không bao giờ bị mất | thanks to which the event is never lost |
| làm cho mỗi bước an toàn khi bị lặp lại | make each step safe when it is repeated |

**Thuật ngữ cần nhớ**

- cam kết hai pha → **two-phase commit (2PC)**
- điều phối viên → **coordinator**
- bước bù trừ → **compensating step / compensating action**
- khoá chống lặp → **idempotency key**
- nhất quán cuối cùng → **eventual consistency**

---

## Mô hình ghi nhớ

**Tiếng Việt**

Transaction là một đơn vị hoặc thành công trọn vẹn hoặc không có gì xảy ra, và nó phải ngắn. Chúng ta không nhốt lời gọi mạng bên trong transaction, và với nghiệp vụ trải nhiều service thì chúng ta dùng saga cùng outbox thay vì two-phase commit.

**English (bám cấu trúc tiếng Việt)**

A transaction is a unit that either succeeds completely or nothing happens at all, and it has to be short. We do not lock a network call inside a transaction, and for a business flow that spreads across many services we use a saga together with an outbox instead of two-phase commit.

---

## Bảng thuật ngữ tổng hợp

| Tiếng Việt | Tiếng Anh | Ghi chú phát âm |
|---|---|---|
| tính nguyên tử | atomicity | a-tơ-**MI**-sơ-ti — trọng âm 3 |
| tính nhất quán | consistency | cơn-**SIS**-tơn-si |
| tính cô lập | isolation | ai-sơ-**LAY**-shơn — âm đầu là "ai" |
| tính bền vững | durability | điua-rơ-**BI**-lơ-ti |
| tính đúng đắn | correctness | cơ-**REC**-nợs |
| bất biến | invariant | in-**VEA**-ri-ơnt — trọng âm 2 |
| ràng buộc kiểm tra giá trị | check constraint | *constraint* /kənˈstreɪnt/ — cụm /str/ đọc liền |
| mức cô lập | isolation level | |
| số dư tài khoản | account balance | ơ-**COUNT** — trọng âm 2 |
| hoàn tác | undo / roll back | |
| xác nhận ghi | commit | cơ-**MIT** — trọng âm 2 |
| nhật ký ghi trước | write-ahead log (WAL) | *write* — chữ **w** câm; WAL đọc là "wal" |
| ghi tuần tự | sequential write | si-**KWEN**-shơl |
| ép ghi xuống đĩa | fsync | đọc là "ef-sink" |
| bộ đệm | buffer | **BU**-fơ |
| phát lại | replay | ri-**PLAY** |
| nhân bản dữ liệu | replication | re-pli-**CAY**-shơn |
| chữ viết tắt | acronym | **A**-crơ-nim — trọng âm 1 |
| tính tuyến tính hoá | linearizability | li-ni-ơ-rai-zơ-**BI**-lơ-ti |
| hệ phân tán | distributed system | dis-**TRI**-biu-tid |
| transaction chạy lâu | long-running transaction | tran-**SAC**-shơn |
| ảnh chụp dữ liệu | snapshot | |
| phình bảng do bản cũ chưa dọn | bloat | /bləʊt/ — "blợut", vần với "boat" |
| khoá chết | deadlock | |
| xếp hàng chờ | queue up | *queue* /kjuː/ — đọc y hệt chữ "Q" |
| sự cố production | incident | **IN**-si-đơnt — trọng âm 1 |
| điểm lưu trong transaction | savepoint | |
| transaction con | subtransaction | |
| tường minh | explicit | ex-**PLI**-cit |
| tự động commit | autocommit | |
| cách làm phản mẫu | anti-pattern | *pattern* Anh /ˈpætn/ — "**PÆT**-tơn" |
| cổng thanh toán | payment gateway | |
| treo, không phản hồi | hang | |
| làm cạn | drain | /dreɪn/ — "đrêin" |
| tác dụng phụ | side effect | |
| bảng gửi đi | outbox table | |
| tiến trình chạy nền | worker | |
| cam kết hai pha | two-phase commit (2PC) | *phase* /feɪz/ — "ph" đọc /f/, đuôi /z/ |
| điều phối viên | coordinator | cou-**OR**-đi-nây-tơ |
| bước bù trừ | compensating step | **COM**-pơn-sây-ting |
| khoá chống lặp | idempotency key | i-đem-**PO**-tơn-si; tính từ *idempotent* i-**DEM**-pơ-tơnt |
| nhất quán cuối cùng | eventual consistency | i-**VEN**-chu-ơl |
| chịu lỗi | fault tolerance | *fault* /fɔːlt/ — "l" phát âm nhẹ |
| bể kết nối | connection pool | |
| rủi ro | risk | đuôi /-sk/ phải bật ra, không thành "rít" |

---

## Bài nói

> **Cách luyện:** bật ghi âm, nói **60–90 giây** cho mỗi đề, **không nhìn tài liệu**. Mỗi bài nói phải dùng **ít nhất 6 thuật ngữ** trong bảng trên. Nghe lại bản ghi một lần, đánh dấu chỗ ngập ngừng, rồi nói lại đề đó lần thứ hai.

1. Explain the four letters of ACID to a junior engineer using a money transfer as your only example.

2. A colleague says: "ACID gives us consistency, so our distributed system is already consistent in the CAP sense." Explain what is wrong with that.

3. Someone on your team opens a transaction, calls the payment gateway inside it, and commits afterwards. Explain why you would push back and what you would propose instead.

4. Describe what the database does between the moment we type COMMIT and the moment the client gets a success response, and explain why a crash right after that does not lose the data.

5. Describe what happens to the rest of the system when one transaction stays open for ten minutes.

6. When would you use a savepoint instead of splitting the work into two separate transactions?

7. A team wants to use two-phase commit across three microservices. Explain why you would argue against it and how a saga with an outbox would solve the same problem.
