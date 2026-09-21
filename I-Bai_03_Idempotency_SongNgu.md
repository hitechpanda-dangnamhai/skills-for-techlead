# Bài 3 — Idempotency (góc concurrency / distributed)
### Bản song ngữ: tiếng Anh bám cấu trúc tiếng Việt

> **Cách dùng bài này:** đọc đoạn tiếng Việt trước → tự dịch trong đầu, không nhìn xuống → so với đoạn tiếng Anh bên dưới → **đọc to bản tiếng Anh hai lần**: lần một đọc chậm cho đúng từng âm, lần hai đọc trôi như đang nói với người phỏng vấn.

---

## ① Idempotent nghĩa là lặp lại không đổi kết quả

**Tiếng Việt**

Idempotent nghĩa là lặp lại nhiều lần mà kết quả không đổi. Chúng ta hãy nghĩ đến cái nút gọi thang máy: bấm năm lần thì vẫn chỉ gọi đúng một thang, chứ không gọi ra năm thang. Trong code, câu lệnh `SET balance = 100` lặp bao nhiêu lần thì số dư vẫn bằng một trăm. Nhưng câu lệnh `balance = balance + 10`, nếu nó chạy hai lần, sẽ làm chúng ta cộng nhầm gấp đôi. Sự khác nhau nằm ở chỗ một bên đặt một giá trị tuyệt đối, còn bên kia cộng thêm một lượng tương đối. Nếu chúng ta không phân biệt được hai loại này, chúng ta sẽ tính tiền khách hàng hai lần trong khi chúng ta vẫn tin rằng code của mình đúng.

**English (bám cấu trúc tiếng Việt)**

Idempotent means that repeating an action many times does not change the result. Let us think about the button that calls a lift: pressing it five times still calls exactly one lift, and does not call out five lifts. In code, the statement `SET balance = 100` can repeat any number of times and the balance still equals one hundred. But the statement `balance = balance + 10`, if it runs twice, will make us add double by mistake. The difference lies in the fact that one side sets an absolute value, while the other side adds a relative amount. If we cannot tell these two kinds apart, we will charge the customer twice while we still believe that our code is correct.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| lặp lại nhiều lần mà kết quả không đổi | repeating an action many times does not change the result |
| chúng ta hãy nghĩ đến | let us think about |
| bấm năm lần thì vẫn chỉ gọi đúng một thang | pressing it five times still calls exactly one lift |
| lặp bao nhiêu lần thì … vẫn | can repeat any number of times and … still |
| sẽ làm chúng ta cộng nhầm gấp đôi | will make us add double by mistake |
| sự khác nhau nằm ở chỗ | the difference lies in the fact that |
| đặt một giá trị tuyệt đối | sets an absolute value |
| cộng thêm một lượng tương đối | adds a relative amount |
| không phân biệt được hai loại này | cannot tell these two kinds apart |
| tính tiền khách hàng hai lần | charge the customer twice |

**Thuật ngữ cần nhớ**

- lặp lại không đổi kết quả → **idempotent**
- giá trị tuyệt đối → **an absolute value**
- lượng cộng thêm → **a relative amount** / **a delta**
- tính tiền hai lần → **to double-charge**

---

## ② Vì sao consumer bắt buộc phải idempotent

**Tiếng Việt**

Lý do chúng ta bắt buộc phải làm cho consumer idempotent là mạng và message queue chắc chắn sẽ giao trùng. Khi một request hết thời gian chờ, client không biết là server đã xử lý hay chưa, cho nên nó gửi lại. Khi một broker không nhận được xác nhận, nó cũng giao lại đúng cái message đó. Đây không phải là một lỗi hiếm gặp mà là hành vi mặc định, bởi vì hầu hết các hệ hàng đợi chọn ngữ nghĩa at-least-once để không bao giờ mất message. Đổi lại, chúng ta phải chấp nhận rằng consumer sẽ nhận trùng, và trách nhiệm chống trùng nằm ở phía chúng ta. Nếu chúng ta bỏ qua trách nhiệm này, chúng ta sẽ trừ tiền hai lần, gửi email hai lần, và tạo hai đơn hàng cho cùng một khách.

**English (bám cấu trúc tiếng Việt)**

The reason why we are obliged to make the consumer idempotent is that the network and the message queue will certainly deliver duplicates. When a request runs out of waiting time, the client does not know whether the server has processed it or not, therefore it sends it again. When a broker does not receive an acknowledgement, it also delivers exactly that message again. This is not a rare fault but the default behaviour, because most queue systems choose at-least-once semantics in order never to lose a message. In exchange, we have to accept that the consumer will receive duplicates, and the responsibility for stopping duplicates lies on our side. If we ignore this responsibility, we will take the money twice, send the email twice, and create two orders for the same customer.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| lý do chúng ta bắt buộc phải | the reason why we are obliged to |
| chắc chắn sẽ giao trùng | will certainly deliver duplicates |
| hết thời gian chờ | runs out of waiting time |
| không biết là server đã xử lý hay chưa | does not know whether the server has processed it or not |
| không nhận được xác nhận | does not receive an acknowledgement |
| không phải là một lỗi hiếm gặp mà là hành vi mặc định | not a rare fault but the default behaviour |
| chọn ngữ nghĩa at-least-once | choose at-least-once semantics |
| để không bao giờ mất message | in order never to lose a message |
| trách nhiệm chống trùng nằm ở phía chúng ta | the responsibility for stopping duplicates lies on our side |
| trừ tiền hai lần | take the money twice |

**Thuật ngữ cần nhớ**

- giao trùng → **to deliver duplicates**
- xác nhận đã nhận → **an acknowledgement**
- giao ít nhất một lần → **at-least-once delivery**
- ngữ nghĩa giao nhận → **delivery semantics**
- bản trùng → **a duplicate**

---

## ③ Exactly-once delivery là một ảo tưởng

**Tiếng Việt**

Nhiều người hỏi tại sao chúng ta không dùng thẳng exactly-once delivery cho đỡ mệt. Câu trả lời là exactly-once delivery không tồn tại trên một mạng không tin cậy, và điều này được minh họa bằng bài toán Two Generals. Bên gửi không bao giờ chắc chắn tuyệt đối rằng bên nhận đã nhận đúng một lần, bởi vì chính cái xác nhận cũng có thể bị mất. Cho nên thứ mà chúng ta thật sự đạt được có tên là effectively-once, và công thức của nó là at-least-once cộng với một consumer biết khử trùng. Ngay cả thứ mà Kafka gọi là exactly-once cũng là producer idempotent cộng transaction nội bộ, chứ nó không phá vỡ định lý này. Trong phỏng vấn, người nói được câu này sẽ lập tức nghe khác hẳn với người chỉ đọc tài liệu tiếp thị.

**English (bám cấu trúc tiếng Việt)**

Many people ask why we do not use exactly-once delivery directly so that life becomes easier. The answer is that exactly-once delivery does not exist over an unreliable network, and this is illustrated by the Two Generals' Problem. The sender is never absolutely sure that the receiver has received it exactly once, because the acknowledgement itself can also be lost. Therefore what we really achieve is called effectively-once, and its formula is at-least-once plus a consumer that knows how to deduplicate. Even the thing that Kafka calls exactly-once is an idempotent producer plus an internal transaction, and it does not break this theorem. In an interview, the person who can say this sentence will immediately sound completely different from the person who has only read the marketing material.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| cho đỡ mệt | so that life becomes easier |
| trên một mạng không tin cậy | over an unreliable network |
| điều này được minh họa bằng | this is illustrated by |
| không bao giờ chắc chắn tuyệt đối rằng | is never absolutely sure that |
| chính cái xác nhận cũng có thể bị mất | the acknowledgement itself can also be lost |
| thứ mà chúng ta thật sự đạt được | what we really achieve |
| một consumer biết khử trùng | a consumer that knows how to deduplicate |
| chứ nó không phá vỡ định lý này | and it does not break this theorem |
| sẽ lập tức nghe khác hẳn với | will immediately sound completely different from |
| người chỉ đọc tài liệu tiếp thị | the person who has only read the marketing material |

**Thuật ngữ cần nhớ**

- giao đúng một lần → **exactly-once delivery**
- hiệu quả như một lần → **effectively-once**
- khử trùng lặp → **to deduplicate**
- mạng không tin cậy → **an unreliable network**
- định lý → **a theorem**

---

## ④ Dedup key phải được claim một cách atomic

**Tiếng Việt**

Chỗ để khử trùng phải là một điểm claim atomic, chứ không phải là một phép kiểm tra rồi hành động. Cách chuẩn là chúng ta tạo một bảng lưu các key đã xử lý, và chúng ta đặt unique constraint trên chính cột key đó. Khi một message đến, chúng ta chạy `INSERT INTO processed_events (event_id) VALUES ($1) ON CONFLICT (event_id) DO NOTHING RETURNING event_id`. Nếu câu lệnh trả về một hàng, chúng ta là người thắng và chúng ta được phép xử lý tiếp. Nếu nó không trả về hàng nào, message này đã được xử lý trước đó và chúng ta chỉ cần trả về hai trăm rồi bỏ qua.

Ngược lại, cách viết `SELECT` xem key đã có chưa rồi mới `INSERT` là một cái bẫy rất quen thuộc. Đó chính là check-then-act, tức là đúng cái TOCTOU mà chúng ta đã học ở Bài 1. Hai message trùng nhau đến cùng lúc sẽ cùng đọc thấy chưa có, cho nên cả hai cùng đi tiếp và cả hai cùng gây side-effect. Điểm cần nhớ là chỉ có ràng buộc của database mới quyết định được ai thắng một cách atomic, còn tầng ứng dụng thì không quyết định được.

**English (bám cấu trúc tiếng Việt)**

The place where we deduplicate must be an atomic claim point, and not a check followed by an action. The standard way is that we create a table which stores the keys that have been processed, and we put a unique constraint on that key column itself. When a message arrives, we run `INSERT INTO processed_events (event_id) VALUES ($1) ON CONFLICT (event_id) DO NOTHING RETURNING event_id`. If the statement returns one row, we are the winner and we are allowed to carry on processing. If it returns no row, this message has been processed before and we only need to return two hundred and skip it.

On the other hand, writing a `SELECT` to see whether the key is already there and only then an `INSERT` is a very familiar trap. That is exactly check-then-act, that is, exactly the TOCTOU that we learned in Lesson 1. Two duplicate messages that arrive at the same time will both read that it is not there, therefore both of them carry on and both of them cause the side effect. The point to remember is that only a constraint of the database can decide the winner atomically, while the application layer cannot decide it.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| một điểm claim atomic | an atomic claim point |
| chứ không phải là một phép kiểm tra rồi hành động | and not a check followed by an action |
| một bảng lưu các key đã xử lý | a table which stores the keys that have been processed |
| trên chính cột key đó | on that key column itself |
| chúng ta là người thắng | we are the winner |
| được phép xử lý tiếp | are allowed to carry on processing |
| rồi bỏ qua | and skip it |
| xem key đã có chưa rồi mới `INSERT` | to see whether the key is already there and only then an `INSERT` |
| một cái bẫy rất quen thuộc | a very familiar trap |
| cùng đọc thấy chưa có | both read that it is not there |
| chỉ có ràng buộc của database mới quyết định được ai thắng | only a constraint of the database can decide the winner |

**Thuật ngữ cần nhớ**

- giành quyền xử lý một cách atomic → **to claim atomically**
- khóa khử trùng → **a dedup key**
- ràng buộc duy nhất → **a unique constraint**
- tác dụng phụ → **a side effect**
- kiểm-tra-rồi-hành-động → **check-then-act**

---

## ⑤ Thao tác tự idempotent và thao tác phải làm cho idempotent

**Tiếng Việt**

Một kỹ năng thực tế là chúng ta nhìn vào một thao tác và biết ngay nó thuộc loại nào. Loại thứ nhất là thao tác tự nó đã idempotent, tức là các phép đặt giá trị tuyệt đối như `PUT state = X`, `SET balance = 100`, hoặc `DELETE id = 5`. Chạy chúng một lần hay mười lần thì trạng thái cuối cùng vẫn giống hệt nhau. Loại thứ hai là thao tác phải được làm cho idempotent, tức là các phép cộng thêm và các phép tạo mới như cộng mười điểm hoặc `INSERT order`. Với loại thứ hai, chúng ta phải gắn thêm một dedup key hoặc một phép kiểm tra version, bởi vì bản thân thao tác không có cách nào tự nhận ra rằng nó đang bị lặp. Nếu chúng ta xếp nhầm một phép tạo mới vào nhóm thứ nhất, chúng ta sẽ phát hiện ra sai lầm khi khách hàng gọi lên và hỏi tại sao họ bị tính tiền hai lần.

**English (bám cấu trúc tiếng Việt)**

A practical skill is that we look at an operation and know straight away which kind it belongs to. The first kind is an operation that is already idempotent by itself, that is, absolute set operations such as `PUT state = X`, `SET balance = 100`, or `DELETE id = 5`. Running them once or ten times leaves the final state exactly the same. The second kind is an operation that has to be made idempotent, that is, increment operations and create operations such as adding ten points or `INSERT order`. For the second kind, we have to attach a dedup key or a version check, because the operation itself has no way to recognise that it is being repeated. If we wrongly put a create operation into the first group, we will discover the mistake when the customer rings up and asks why they have been charged twice.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| biết ngay nó thuộc loại nào | know straight away which kind it belongs to |
| tự nó đã idempotent | already idempotent by itself |
| các phép đặt giá trị tuyệt đối | absolute set operations |
| trạng thái cuối cùng vẫn giống hệt nhau | leaves the final state exactly the same |
| phải được làm cho idempotent | has to be made idempotent |
| các phép cộng thêm và các phép tạo mới | increment operations and create operations |
| chúng ta phải gắn thêm | we have to attach |
| không có cách nào tự nhận ra rằng nó đang bị lặp | has no way to recognise that it is being repeated |
| nếu chúng ta xếp nhầm … vào nhóm thứ nhất | if we wrongly put … into the first group |
| khi khách hàng gọi lên và hỏi | when the customer rings up and asks |

**Thuật ngữ cần nhớ**

- tự nhiên đã idempotent → **naturally idempotent**
- phép đặt tuyệt đối → **an absolute set**
- phép cộng thêm → **an increment operation**
- phép tạo mới → **a create operation**
- kiểm tra phiên bản → **a version check**

---

## ⑥ Bẫy chí mạng: idempotent nhưng vẫn race khi chạy song song

**Tiếng Việt**

Đây là điểm quan trọng nhất của cả bài, và đây cũng là chỗ nhiều người trả lời sai trong phỏng vấn. Một handler idempotent vẫn có thể tạo ra hai đơn hàng khi hai bản retry chạy song song với cùng một key. Lý do là idempotency chỉ hứa rằng lặp lại tuần tự thì không sao, chứ nó không hứa gì về việc chạy đồng thời. Hai request cùng key đến trong cùng một mili-giây sẽ cùng thấy rằng key chưa tồn tại, cho nên cả hai cùng ghi và cả hai cùng gây side-effect. Nếu chúng ta muốn chặn tình huống này, chúng ta phải có một phép claim atomic, thường là unique constraint, để database tuần tự hóa hai request đó lại. Kết luận cần thuộc là idempotency và concurrency control phải đi cùng nhau, bởi vì một mình idempotency không đủ.

**English (bám cấu trúc tiếng Việt)**

This is the most important point of the whole lesson, and this is also the place where many people answer wrongly in an interview. An idempotent handler can still create two orders when two retries run in parallel with the same key. The reason is that idempotency only promises that repeating sequentially is fine, and it promises nothing about running at the same time. Two requests with the same key that arrive within the same millisecond will both see that the key does not exist yet, therefore both of them write and both of them cause the side effect. If we want to block this situation, we must have an atomic claim, usually a unique constraint, so that the database serialises those two requests. The conclusion to learn by heart is that idempotency and concurrency control have to go together, because idempotency on its own is not enough.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| chỗ nhiều người trả lời sai trong phỏng vấn | the place where many people answer wrongly in an interview |
| khi hai bản retry chạy song song với cùng một key | when two retries run in parallel with the same key |
| chỉ hứa rằng lặp lại tuần tự thì không sao | only promises that repeating sequentially is fine |
| nó không hứa gì về việc chạy đồng thời | it promises nothing about running at the same time |
| đến trong cùng một mili-giây | arrive within the same millisecond |
| cùng thấy rằng key chưa tồn tại | both see that the key does not exist yet |
| nếu chúng ta muốn chặn tình huống này | if we want to block this situation |
| để database tuần tự hóa hai request đó lại | so that the database serialises those two requests |
| kết luận cần thuộc là | the conclusion to learn by heart is that |
| một mình idempotency không đủ | idempotency on its own is not enough |

**Thuật ngữ cần nhớ**

- chạy song song → **to run in parallel**
- lặp lại tuần tự → **to repeat sequentially**
- tuần tự hóa → **to serialise**
- giành quyền một cách atomic → **an atomic claim**
- điều khiển đồng thời → **concurrency control**

---

## ⑦ Claim trước, side-effect sau, và outbox pattern

**Tiếng Việt**

Thứ tự thực hiện trong handler cũng quan trọng không kém bản thân cơ chế. Chúng ta phải claim key trước, và chỉ khi thắng claim thì chúng ta mới gây side-effect. Nếu chúng ta làm ngược lại, tức là trừ tiền trước rồi mới ghi key, thì một bản trùng đến giữa hai bước sẽ trừ tiền thêm một lần nữa. Lý tưởng nhất là chúng ta gói phép claim và phần ghi dữ liệu vào cùng một transaction, để hoặc cả hai cùng thành công hoặc cả hai cùng bị hủy. Nếu chúng ta để hai thao tác đó ở hai transaction khác nhau, hệ thống sẽ có một khoảng thời gian mà key đã được ghi nhưng công việc thì chưa được làm.

Khi side-effect là một lời gọi sang service bên ngoài, chúng ta không thể rollback nó cùng với transaction được. Trong tình huống đó, chúng ta dùng outbox pattern, tức là chúng ta ghi ý định gửi vào cùng transaction với dữ liệu, và một tiến trình relay sẽ đọc bảng outbox rồi gửi đi sau. Nhờ đó event không bị mất khi transaction thành công, và event cũng không bị gửi khi transaction thất bại. Và ở tầng dữ liệu, cột version của Bài 2 cũng chính là một dạng idempotency theo trạng thái, bởi vì một lệnh ghi lặp lại với version cũ sẽ bị từ chối.

**English (bám cấu trúc tiếng Việt)**

The order of the steps inside the handler is no less important than the mechanism itself. We must claim the key first, and only when we win the claim do we cause the side effect. If we do the opposite, that is, take the money first and only then write the key, a duplicate that arrives between the two steps will take the money one more time. The ideal is that we wrap the claim and the data write into the same transaction, so that either both of them succeed or both of them are cancelled. If we leave those two operations in two different transactions, the system will have a period during which the key has been written but the work has not been done.

When the side effect is a call out to an external service, we cannot roll it back together with the transaction. In that situation, we use the outbox pattern, that is, we write the intention to send into the same transaction as the data, and a relay process will read the outbox table and send it later. Thanks to that, the event is not lost when the transaction succeeds, and the event is also not sent when the transaction fails. And at the data layer, the version column of Lesson 2 is also a form of idempotency by state, because a repeated write with an old version will be rejected.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| quan trọng không kém bản thân cơ chế | no less important than the mechanism itself |
| chỉ khi thắng claim thì chúng ta mới gây side-effect | only when we win the claim do we cause the side effect |
| nếu chúng ta làm ngược lại | if we do the opposite |
| một bản trùng đến giữa hai bước | a duplicate that arrives between the two steps |
| gói … vào cùng một transaction | wrap … into the same transaction |
| hoặc cả hai cùng thành công hoặc cả hai cùng bị hủy | either both of them succeed or both of them are cancelled |
| sẽ có một khoảng thời gian mà | will have a period during which |
| một lời gọi sang service bên ngoài | a call out to an external service |
| chúng ta ghi ý định gửi | we write the intention to send |
| một tiến trình relay | a relay process |
| một dạng idempotency theo trạng thái | a form of idempotency by state |

**Thuật ngữ cần nhớ**

- mẫu hộp thư đi → **the outbox pattern**
- tiến trình chuyển tiếp → **a relay process**
- hoàn tác giao dịch → **to roll back a transaction**
- được-ăn-cả-ngã-về-không → **all-or-nothing**
- dịch vụ bên ngoài → **an external service**

---

## Mô hình ghi nhớ

**Tiếng Việt**

Mạng chắc chắn sẽ giao trùng, cho nên consumer bắt buộc phải lặp-không-đổi; nhưng idempotency chỉ cứu được trường hợp lặp tuần tự, còn lặp song song thì cần thêm một phép claim atomic. Và exactly-once delivery không tồn tại, thứ chúng ta thật sự có là at-least-once cộng với khử trùng, tức là effectively-once.

**English (bám cấu trúc tiếng Việt)**

The network will certainly deliver duplicates, therefore the consumer is obliged to be idempotent; but idempotency only saves the case of repeating sequentially, while repeating in parallel needs an atomic claim as well. And exactly-once delivery does not exist, what we really have is at-least-once plus deduplication, that is, effectively-once.

---

## Bảng thuật ngữ tổng hợp

| Tiếng Việt | Tiếng Anh | Ghi chú phát âm (Anh–Anh) |
|---|---|---|
| lặp lại không đổi kết quả | **idempotent** | *ai-DEM-pơ-tợnt* — trọng âm âm thứ hai, không đọc "i-đem-pô-ten" |
| tính chất lặp-không-đổi | **idempotency** | *ai-đem-PO-tơn-si* — trọng âm dịch sang âm thứ ba |
| giá trị tuyệt đối | **an absolute value** | *AB-sơ-luut* /ˈæbsəluːt/ — trọng âm đầu |
| lượng cộng thêm | **a delta** | |
| tính tiền hai lần | **to double-charge** | *charge* /tʃɑːdʒ/ — đầu /tʃ/, cuối /dʒ/, hai âm khác nhau |
| người tiêu thụ message | **a consumer** | Anh: *kơn-SYUU-mơ* /kənˈsjuːmə/ — có âm /sj/, khác Mỹ "kơn-SUU-mơ" |
| giao trùng | **to deliver duplicates** | *duplicate* danh từ = *DYUU-pli-kợt*; động từ = *DYUU-pli-kêit* |
| xác nhận đã nhận | **an acknowledgement** | *ơk-NO-lij-mợnt* — trọng âm âm thứ hai, chữ `c` đọc /k/ |
| hàng đợi | **a queue** | /kjuː/ đọc đúng như tên chữ cái **Q** |
| bộ môi giới message | **a message broker** | *BROU-kơ* /ˈbrəʊkə/ |
| giao ít nhất một lần | **at-least-once delivery** | |
| ngữ nghĩa giao nhận | **delivery semantics** | *si-MAN-tiks* — trọng âm giữa, cụm cuối **-ks** phải bật |
| giao đúng một lần | **exactly-once delivery** | *exactly* = *ig-ZAKT-li*, chữ `x` đọc /gz/; `once` /wʌns/ cụm cuối **-ns** |
| hiệu quả như một lần | **effectively-once** | |
| khử trùng lặp | **to deduplicate** | *dii-DYUU-pli-kêit* — trọng âm âm thứ hai |
| mạng không tin cậy | **an unreliable network** | *an-ri-LAI-ơ-bợl* — trọng âm âm thứ ba |
| định lý | **a theorem** | *THI-ơ-rợm* /ˈθɪərəm/ — âm **th**, không đọc "thê-ô-rem" |
| giành quyền một cách atomic | **an atomic claim** | *atomic* = *ơ-TOM-ik*; *claim* /kleɪm/ — cụm **cl** phải rõ |
| khóa khử trùng | **a dedup key** | |
| ràng buộc duy nhất | **a unique constraint** | *unique* = *yu-NIIK*; *constraint* = *kơn-STRÊINT* với cụm **str** |
| tác dụng phụ | **a side effect** | |
| kiểm-tra-rồi-hành-động | **check-then-act** | có âm **th** trong `then` |
| tự nhiên đã idempotent | **naturally idempotent** | *NA-tsrơ-li* /ˈnætʃrəli/ — ba âm tiết khi nói nhanh |
| phép đặt tuyệt đối | **an absolute set** | |
| phép cộng thêm | **an increment operation** | *IN-krơ-mợnt* — trọng âm đầu |
| phép tạo mới | **a create operation** | |
| kiểm tra phiên bản | **a version check** | *version* Anh = /ˈvɜːʃn/ "VƠƠ-shợn" với /ʃ/ |
| chạy song song | **to run in parallel** | *PA-rơ-lel* — trọng âm đầu, ba âm tiết |
| lặp lại tuần tự | **to repeat sequentially** | *si-KWEN-shợ-li* — trọng âm thứ hai, có âm /kw/ |
| tuần tự hóa | **to serialise** | Anh viết **-ise**; đọc *SIA-ri-ơ-laiz* |
| mili-giây | **a millisecond** | *MI-li-se-kợnd* — trọng âm đầu |
| điều khiển đồng thời | **concurrency control** | *kơn-KA-rợn-si* — trọng âm giữa |
| mẫu hộp thư đi | **the outbox pattern** | *pattern* Anh = *PA-tợn* /ˈpætn/ — không bật âm `r` cuối |
| tiến trình chuyển tiếp | **a relay process** | *relay* danh từ = *RII-lêi*; *process* Anh = *PROU-ses* |
| hoàn tác giao dịch | **to roll back a transaction** | *transaction* = *tran-ZAK-shợn*, chữ `s` đọc /z/ |
| được-ăn-cả-ngã-về-không | **all-or-nothing** | *nothing* /ˈnʌθɪŋ/ — âm **th** và đuôi **-ng** |
| dịch vụ bên ngoài | **an external service** | *ik-STƠƠ-nợl* — trọng âm giữa |
| ý định gửi | **the intention to send** | *in-TEN-shợn* — trọng âm giữa |
| tài liệu tiếp thị | **the marketing material** | *mơ-TIA-ri-ợl* — trọng âm âm thứ hai |

---

## Bài nói

> **Cách luyện:** bật ghi âm, nói **60–90 giây** cho mỗi đề, **không nhìn tài liệu**. Mỗi bài nói phải dùng ít nhất **sáu thuật ngữ** trong bảng trên. Nói xong nghe lại một lần, đánh dấu chỗ nào bị vấp hoặc phát âm sai, rồi nói lại đề đó một lần nữa.

1. **Explain to a junior developer** why a payment webhook handler must be idempotent, and what goes wrong in production if it is not. Walk through the retry path that creates the duplicate.

2. **A colleague says:** *"Our queue is configured for exactly-once delivery, so we do not need any deduplication in the consumer."* Explain what is wrong with that belief and what guarantee they actually have.

3. **Describe what happens**, step by step, when two copies of the same webhook arrive within the same millisecond at a handler that checks the key with a `SELECT` before inserting it. Name the failure mode.

4. **Someone on your team wants to charge the card first and record the idempotency key afterwards**, because "the charge is the important part". Explain why you would push back, and what ordering you would insist on.

5. **When would you say an operation is naturally idempotent**, and when does it have to be made idempotent? Give two examples of each from a real e-commerce system.

6. **Explain to a product manager**, without SQL, why the team cannot promise that a confirmation email will be sent exactly once, and what promise the team can make instead.

7. **A senior engineer argues** that the outbox pattern is over-engineering, and that calling the external payment service directly inside the transaction is simpler. Explain why you disagree, and what breaks in their design.

---

> 🔚 **Hết Bài 3.** Gõ `Làm Bài 4` để sang *Distributed Locks & Fencing Token*.
