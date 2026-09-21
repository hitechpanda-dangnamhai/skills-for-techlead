# Bài 8 — Event Sourcing & CQRS
### Bản song ngữ: tiếng Anh bám cấu trúc tiếng Việt

> **Cách dùng bài này:** đọc đoạn tiếng Việt trước → tự dịch trong đầu, không nhìn xuống → so sánh với đoạn tiếng Anh ngay bên dưới → **đọc to bản tiếng Anh hai lần**: lần một đọc chậm cho đúng từng âm, lần hai đọc liền mạch như đang nói với người phỏng vấn.

---

## ① Lưu chuyện đã xảy ra thay vì lưu trạng thái hiện tại

**Tiếng Việt**

Event Sourcing và CQRS cũng thuộc họ hướng sự kiện, nhưng chúng nằm ở tầng lưu trữ trạng thái chứ không nằm ở tầng giao tiếp giữa các service. Trong cách làm quen thuộc, chúng ta lưu trạng thái hiện tại và mỗi lần thay đổi thì chúng ta ghi đè lên giá trị cũ. Trong Event Sourcing, chúng ta lưu một chuỗi event bất biến được ghi nối đuôi, và chúng ta không bao giờ sửa một event đã ghi. Trạng thái hiện tại không được lưu trực tiếp nữa, mà nó được dựng lại bằng cách tua lại toàn bộ chuỗi event từ đầu. Nói ngắn gọn, chúng ta lưu chuyện đã xảy ra chứ không lưu kết quả cuối cùng.

Cách này cho chúng ta ba thứ mà cách thông thường không có. Thứ nhất là dấu vết kiểm toán đầy đủ, vì mọi thay đổi đều còn nguyên trong lịch sử. Thứ hai là khả năng du hành thời gian, nghĩa là chúng ta dựng lại được trạng thái tại bất kỳ thời điểm nào trong quá khứ. Thứ ba là chúng ta dựng được một cách nhìn hoàn toàn mới từ lịch sử cũ, dù cách nhìn đó chưa hề tồn tại lúc dữ liệu được ghi. Đổi lại, chúng ta trả giá bằng sự phức tạp, bằng việc truy vấn trạng thái hiện tại trở nên khó hơn, và bằng nhất quán cuối cùng ở phía đọc.

**English (bám cấu trúc tiếng Việt)**

Event Sourcing and CQRS also belong to the event-driven family, but they sit at the state storage layer rather than at the communication layer between services. In the familiar way of working, we store the current state and every time something changes we overwrite the old value. In Event Sourcing, we store a chain of immutable events that are appended one after another, and we never modify an event that has been written. The current state is no longer stored directly, but it is rebuilt by replaying the whole chain of events from the beginning. In short, we store what has happened rather than storing the final result.

This way gives us three things that the ordinary way does not have. The first is a complete audit trail, because every change remains intact in the history. The second is the ability to travel in time, which means that we can rebuild the state at any moment in the past. The third is that we can build a completely new view out of the old history, even though that view did not exist at all when the data was written. In exchange, we pay with complexity, with querying the current state becoming harder, and with eventual consistency on the reading side.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| chúng thuộc họ hướng sự kiện | they belong to the event-driven family |
| ở tầng lưu trữ trạng thái | at the state storage layer |
| trong cách làm quen thuộc | in the familiar way of working |
| chúng ta ghi đè lên giá trị cũ | we overwrite the old value |
| một chuỗi event bất biến được ghi nối đuôi | a chain of immutable events that are appended |
| nó được dựng lại bằng cách tua lại | it is rebuilt by replaying |
| chúng ta lưu chuyện đã xảy ra | we store what has happened |
| mọi thay đổi đều còn nguyên trong lịch sử | every change remains intact in the history |
| khả năng du hành thời gian | the ability to travel in time |
| một cách nhìn hoàn toàn mới từ lịch sử cũ | a completely new view out of the old history |
| chúng ta trả giá bằng sự phức tạp | we pay with complexity |

**Thuật ngữ cần nhớ**

- lưu trạng thái bằng chuỗi sự kiện → **event sourcing**
- bất biến, không sửa được → **immutable**
- tua lại chuỗi event → **to replay the event stream**
- dựng lại trạng thái → **to rebuild the state**
- du hành thời gian → **time travel**

---

## ② Điểm khác cốt lõi so với nhật ký kiểm toán

**Tiếng Việt**

Rất nhiều người nhầm Event Sourcing với một bảng nhật ký kiểm toán thông thường, nên chúng ta phải phân biệt được hai thứ đó chỉ trong một câu. Trong một hệ thống có nhật ký kiểm toán thông thường, bảng trạng thái vẫn là nguồn sự thật, còn nhật ký chỉ là một sản phẩm phụ ghi thêm bên cạnh. Nếu bảng nhật ký đó bị xoá, hệ thống vẫn chạy bình thường, vì trạng thái không hề phụ thuộc vào nó. Trong Event Sourcing thì ngược lại hoàn toàn, vì chính chuỗi event mới là nguồn sự thật duy nhất. Nếu chúng ta mất chuỗi event, chúng ta mất luôn dữ liệu, vì không còn gì để dựng lại trạng thái nữa.

Vì vậy câu hỏi phân biệt hai thứ này chỉ có một, đó là ai là nguồn sự thật. Chúng ta nên trả lời đúng câu đó trong phỏng vấn thay vì mô tả dài dòng về cấu trúc bảng. Sự khác biệt này cũng kéo theo hệ quả về kỷ luật kỹ thuật mà đội phải giữ. Trong Event Sourcing, chúng ta không được phép sửa một event đã ghi, kể cả khi chúng ta phát hiện ra nó sai. Cách chữa đúng là ghi thêm một event mới để đính chính, y hệt cách một kế toán viên ghi bút toán điều chỉnh chứ không tẩy xoá sổ cũ.

**English (bám cấu trúc tiếng Việt)**

Very many people confuse Event Sourcing with an ordinary audit log table, so we must be able to tell those two apart in only one sentence. In a system that has an ordinary audit log, the state table is still the source of truth, while the log is only a by-product written alongside it. If that log table is deleted, the system still runs normally, because the state does not depend on it at all. In Event Sourcing it is completely the opposite, because the event stream itself is the only source of truth. If we lose the event stream, we lose the data as well, because there is nothing left to rebuild the state from.

Therefore the question that separates these two things is only one, which is who the source of truth is. We should answer exactly that question in an interview instead of describing the table structure at length. This difference also brings a consequence about the engineering discipline that the team has to keep. In Event Sourcing, we are not allowed to modify an event that has been written, even when we discover that it is wrong. The correct cure is to write one more event as a correction, exactly the way an accountant writes an adjusting entry rather than rubbing out the old book.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| phân biệt được hai thứ đó chỉ trong một câu | tell those two apart in only one sentence |
| một sản phẩm phụ ghi thêm bên cạnh | a by-product written alongside it |
| trạng thái không hề phụ thuộc vào nó | the state does not depend on it at all |
| ngược lại hoàn toàn | completely the opposite |
| không còn gì để dựng lại trạng thái nữa | there is nothing left to rebuild the state from |
| ai là nguồn sự thật | who the source of truth is |
| thay vì mô tả dài dòng về | instead of describing ... at length |
| kỷ luật kỹ thuật mà đội phải giữ | the engineering discipline that the team has to keep |
| kể cả khi chúng ta phát hiện ra nó sai | even when we discover that it is wrong |
| ghi thêm một event mới để đính chính | write one more event as a correction |
| chứ không tẩy xoá sổ cũ | rather than rubbing out the old book |

**Thuật ngữ cần nhớ**

- nguồn sự thật → **the source of truth**
- nhật ký kiểm toán → **an audit log**
- sản phẩm phụ → **a by-product**
- bút toán điều chỉnh → **an adjusting entry**
- chuỗi event của một thực thể → **an event stream**

---

## ③ Snapshot và yêu cầu tất định của hàm apply

**Tiếng Việt**

Nếu mỗi lần đọc trạng thái mà chúng ta phải tua lại từ event đầu tiên, chi phí sẽ tăng theo tuổi của hệ thống. Một tài khoản chạy năm năm có thể tích tới hàng trăm nghìn event, và không ai muốn tua lại từng ấy chỉ để hiển thị số dư. Vì vậy chúng ta lưu snapshot, tức là một bản ghi trạng thái tại một vị trí nhất định trong chuỗi. Khi cần dựng lại, chúng ta nạp snapshot gần nhất rồi chỉ tua lại các event nằm sau nó. Đánh đổi ở đây rất rõ, đó là chúng ta tốn thêm chỗ lưu trữ để đổi lấy tốc độ dựng lại.

Có một yêu cầu kỹ thuật đi kèm mà người mới rất hay bỏ sót, đó là hàm áp dụng event phải thuần và tất định. Thuần nghĩa là nó chỉ nhận trạng thái cũ và một event rồi trả về trạng thái mới, không đọc thêm gì từ bên ngoài. Tất định nghĩa là với cùng đầu vào, nó luôn cho ra cùng đầu ra, bất kể chúng ta chạy nó lúc nào. Nếu hàm đó gọi giờ hệ thống hoặc gọi một số ngẫu nhiên, chúng ta sẽ dựng ra hai trạng thái khác nhau từ cùng một chuỗi event. Khi đó snapshot và bản tua lại sẽ lệch nhau, và chúng ta mất hoàn toàn niềm tin vào dữ liệu của mình.

**English (bám cấu trúc tiếng Việt)**

If every time we read the state we have to replay from the very first event, the cost will grow with the age of the system. An account running for five years may accumulate hundreds of thousands of events, and nobody wants to replay that many just to display a balance. Therefore we store a snapshot, that is, a record of the state at a certain position in the stream. When we need to rebuild, we load the nearest snapshot and then only replay the events that sit after it. The trade-off here is very clear, which is that we spend extra storage in exchange for rebuild speed.

There is a technical requirement that comes with it and that newcomers very often miss, which is that the function applying an event must be pure and deterministic. Pure means that it only takes the old state and one event and then returns the new state, without reading anything else from outside. Deterministic means that with the same input, it always produces the same output, no matter when we run it. If that function calls the system clock or calls a random number, we will build two different states from the same event stream. At that point the snapshot and the replayed version will drift apart, and we lose all trust in our own data.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| chi phí sẽ tăng theo tuổi của hệ thống | the cost will grow with the age of the system |
| có thể tích tới hàng trăm nghìn event | may accumulate hundreds of thousands of events |
| chỉ để hiển thị số dư | just to display a balance |
| tại một vị trí nhất định trong chuỗi | at a certain position in the stream |
| chúng ta nạp snapshot gần nhất | we load the nearest snapshot |
| tốn thêm chỗ lưu trữ để đổi lấy tốc độ | spend extra storage in exchange for speed |
| người mới rất hay bỏ sót | newcomers very often miss |
| phải thuần và tất định | must be pure and deterministic |
| không đọc thêm gì từ bên ngoài | without reading anything else from outside |
| bất kể chúng ta chạy nó lúc nào | no matter when we run it |
| gọi giờ hệ thống | calls the system clock |
| sẽ lệch nhau | will drift apart |

**Thuật ngữ cần nhớ**

- bản chụp trạng thái → **a snapshot**
- hàm thuần → **a pure function**
- tất định → **deterministic**
- gộp dần các event thành trạng thái → **to fold the events**
- tốc độ dựng lại → **rebuild speed**

---

## ④ CQRS: tách mô hình ghi khỏi mô hình đọc

**Tiếng Việt**

CQRS là ý tưởng tách mô hình ghi ra khỏi mô hình đọc, và nó độc lập với Event Sourcing. Mô hình ghi nhận các lệnh, kiểm tra quy tắc nghiệp vụ, và nó được tối ưu cho tính đúng đắn chứ không phải cho tốc độ truy vấn. Mô hình đọc phục vụ các câu truy vấn, và nó được tối ưu cho đúng những cách mà giao diện muốn lấy dữ liệu. Hai mô hình này có thể nằm trong hai bảng khác nhau, hai kho dữ liệu khác nhau, hoặc thậm chí hai công nghệ khác nhau. Nhờ tách như vậy, chúng ta thay đổi cách hiển thị mà không phải bẻ cong mô hình nghiệp vụ ở phía ghi.

Chúng ta nên nói rõ vì sao Event Sourcing và CQRS thường đi cùng nhau, nhưng lại không bắt buộc phải đi cùng. Chúng đi cùng nhau vì một kho event rất khó truy vấn trực tiếp, nên chúng ta phải chiếu chuỗi event ra các bảng đọc phù hợp với từng màn hình. Tuy nhiên chúng ta hoàn toàn có thể dùng CQRS mà không dùng Event Sourcing, ví dụ hệ ghi vẫn là các bảng quan hệ bình thường còn hệ đọc là một chỉ mục tìm kiếm. Ngược lại, chúng ta cũng có thể dùng Event Sourcing cho một aggregate nhỏ mà không dựng cả một kiến trúc đọc riêng. Vì vậy khi trả lời phỏng vấn, chúng ta nên nhấn mạnh rằng đây là hai khái niệm độc lập thường được dùng chung.

**English (bám cấu trúc tiếng Việt)**

CQRS is the idea of separating the write model from the read model, and it is independent of Event Sourcing. The write model receives the commands, checks the business rules, and it is optimised for correctness rather than for query speed. The read model serves the queries, and it is optimised for exactly the ways the interface wants to fetch the data. These two models can sit in two different tables, two different data stores, or even two different technologies. Thanks to that separation, we change the way we display things without having to bend the business model on the write side.

We should say clearly why Event Sourcing and CQRS often go together, but are not required to go together. They go together because an event store is very hard to query directly, so we have to project the event stream out into read tables that suit each screen. However we can perfectly well use CQRS without using Event Sourcing, for example the write side is still ordinary relational tables while the read side is a search index. Conversely, we can also use Event Sourcing for one small aggregate without building a whole separate read architecture. Therefore when we answer in an interview, we should stress that these are two independent concepts that are often used together.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| tách mô hình ghi ra khỏi mô hình đọc | separating the write model from the read model |
| nó độc lập với | it is independent of |
| được tối ưu cho tính đúng đắn | optimised for correctness |
| đúng những cách mà giao diện muốn lấy dữ liệu | exactly the ways the interface wants to fetch the data |
| hai kho dữ liệu khác nhau | two different data stores |
| mà không phải bẻ cong mô hình nghiệp vụ | without having to bend the business model |
| nhưng lại không bắt buộc phải đi cùng | but are not required to go together |
| chiếu chuỗi event ra các bảng đọc | project the event stream out into read tables |
| chúng ta hoàn toàn có thể dùng | we can perfectly well use |
| ngược lại | conversely |
| hai khái niệm độc lập thường được dùng chung | two independent concepts that are often used together |

**Thuật ngữ cần nhớ**

- tách lệnh ghi khỏi câu đọc → **CQRS**
- mô hình ghi và mô hình đọc → **the write model and the read model**
- kho lưu event → **the event store**
- chiếu dữ liệu ra bảng đọc → **to project**
- chỉ mục tìm kiếm → **a search index**

---

## ⑤ Projection chạy sau, nên người dùng có thể chưa thấy ngay

**Tiếng Việt**

Trong kiến trúc này, mô hình đọc được cập nhật một cách bất đồng bộ sau khi mô hình ghi đã commit. Vì vậy luôn tồn tại một khoảng trễ giữa lúc dữ liệu được ghi và lúc dữ liệu xuất hiện ở phía đọc. Khoảng trễ đó thường chỉ vài chục mili giây, nhưng nó vẫn đủ để người dùng bấm lưu rồi tải lại trang mà chưa thấy thay đổi. Nếu chúng ta không lường trước, người dùng sẽ bấm lưu thêm lần nữa và chúng ta sinh ra bản ghi trùng. Đây là một lỗi trải nghiệm rất phổ biến, và nó xuất phát từ thiết kế chứ không phải từ một con bug.

Chúng ta có ba cách xử lý và chúng ta nên chọn theo từng màn hình cụ thể. Cách thứ nhất là cập nhật giao diện một cách lạc quan, nghĩa là chúng ta hiển thị kết quả mong đợi ngay sau khi lệnh ghi được chấp nhận. Cách thứ hai là cho những màn hình bắt buộc phải tươi đọc thẳng từ phía ghi, và chúng ta chấp nhận đánh đổi về hiệu năng ở đúng chỗ đó. Cách thứ ba là nói thật với người dùng bằng một trạng thái đang xử lý, để họ hiểu rằng hệ thống đã nhận yêu cầu. Điều quan trọng là chúng ta thiết kế cho độ trễ chứ không giả vờ rằng độ trễ không tồn tại.

**English (bám cấu trúc tiếng Việt)**

In this architecture, the read model is updated asynchronously after the write model has committed. Therefore there is always a delay between the moment the data is written and the moment the data appears on the read side. That delay is usually only a few tens of milliseconds, but it is still enough for the user to press save and then reload the page without seeing the change. If we do not anticipate it, the user will press save one more time and we create a duplicate record. This is a very common experience bug, and it comes from the design rather than from a bug in the code.

We have three ways to handle it and we should choose per individual screen. The first way is to update the interface optimistically, which means that we display the expected result right after the write command is accepted. The second way is to let the screens that must be fresh read straight from the write side, and we accept the performance trade-off at exactly that place. The third way is to tell the user honestly with a processing state, so that they understand the system has received the request. The important thing is that we design for the delay rather than pretending that the delay does not exist.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| luôn tồn tại một khoảng trễ giữa | there is always a delay between |
| vài chục mili giây | a few tens of milliseconds |
| bấm lưu rồi tải lại trang | press save and then reload the page |
| nếu chúng ta không lường trước | if we do not anticipate it |
| chúng ta sinh ra bản ghi trùng | we create a duplicate record |
| nó xuất phát từ thiết kế | it comes from the design |
| chọn theo từng màn hình cụ thể | choose per individual screen |
| cập nhật giao diện một cách lạc quan | update the interface optimistically |
| những màn hình bắt buộc phải tươi | the screens that must be fresh |
| nói thật với người dùng | tell the user honestly |
| chúng ta thiết kế cho độ trễ | we design for the delay |
| giả vờ rằng độ trễ không tồn tại | pretending that the delay does not exist |

**Thuật ngữ cần nhớ**

- bảng đọc được dựng từ event → **a projection**
- độ trễ của projection → **projection lag**
- cập nhật giao diện lạc quan → **an optimistic update**
- dữ liệu tươi mới → **fresh data**
- trạng thái đang xử lý → **a processing state**

---

## ⑥ Schema tiến hoá khi event là bất biến

**Tiếng Việt**

Chi phí dài hạn mà ít đội lường trước được nằm ở việc schema của event phải tiến hoá theo thời gian. Trong hệ thông thường, chúng ta chạy một lệnh chuyển đổi dữ liệu và bảng cũ trở thành bảng mới. Trong Event Sourcing, chúng ta không được phép làm như vậy, vì event đã ghi là bất biến và lịch sử không được viết lại. Vì thế chúng ta phải gắn số phiên bản cho từng loại event ngay từ ngày đầu tiên. Khi cấu trúc thay đổi, chúng ta viết một bộ nâng phiên bản để chuyển event phiên bản cũ thành phiên bản mới ngay lúc đọc.

Bên cạnh đó chúng ta nên giữ hai thói quen giúp giảm đau về sau. Thói quen thứ nhất là dùng schema lỏng ở phía đọc, nghĩa là hàm áp dụng bỏ qua những trường mà nó không biết thay vì ném lỗi. Thói quen thứ hai là thêm trường mới ở dạng tuỳ chọn kèm giá trị mặc định, đúng như quy tắc chúng ta đã học ở Bài 2. Nếu một ngày nào đó chúng ta thật sự phải viết lại lịch sử, chúng ta nên coi đó là một dự án di trú có kế hoạch chứ không phải một lệnh cập nhật chạy vội. Hậu quả của việc sửa event tại chỗ là mọi snapshot, mọi projection và mọi báo cáo cũ đều không còn khớp với nhau nữa.

**English (bám cấu trúc tiếng Việt)**

The long-term cost that few teams anticipate lies in the fact that the event schema has to evolve over time. In an ordinary system, we run a data migration statement and the old table becomes the new table. In Event Sourcing, we are not allowed to do that, because an event that has been written is immutable and history must not be rewritten. That is why we have to attach a version number to each event type from the very first day. When the structure changes, we write an upcaster to convert an event of the old version into the new version at the moment of reading.

Alongside that we should keep two habits that reduce the pain later. The first habit is to use a loose schema on the reading side, which means that the apply function ignores the fields it does not know instead of throwing an error. The second habit is to add new fields as optional with a default value, exactly like the rule we learned in Lesson 2. If one day we really have to rewrite the history, we should treat that as a planned migration project rather than an update statement run in a hurry. The consequence of editing events in place is that every snapshot, every projection and every old report no longer matches one another.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| chi phí dài hạn mà ít đội lường trước được | the long-term cost that few teams anticipate |
| lịch sử không được viết lại | history must not be rewritten |
| ngay từ ngày đầu tiên | from the very first day |
| một bộ nâng phiên bản | an upcaster |
| ngay lúc đọc | at the moment of reading |
| hai thói quen giúp giảm đau về sau | two habits that reduce the pain later |
| bỏ qua những trường mà nó không biết | ignores the fields it does not know |
| một dự án di trú có kế hoạch | a planned migration project |
| một lệnh cập nhật chạy vội | an update statement run in a hurry |
| việc sửa event tại chỗ | editing events in place |
| không còn khớp với nhau nữa | no longer matches one another |

**Thuật ngữ cần nhớ**

- gắn phiên bản cho event → **to version an event**
- bộ nâng phiên bản khi đọc → **an upcaster**
- schema lỏng → **a loose schema**
- di trú dữ liệu → **a data migration**
- viết lại lịch sử → **to rewrite history**

---

## ⑦ Khi nào chúng ta KHÔNG nên dùng Event Sourcing

**Tiếng Việt**

Phần cuối cùng và cũng là phần quan trọng nhất về mặt phán đoán là biết khi nào không nên dùng Event Sourcing. Với một ứng dụng chủ yếu là tạo, đọc, sửa, xoá, Event Sourcing thêm rất nhiều phức tạp mà không mang lại lợi ích tương xứng. Chúng ta sẽ phải gánh nhất quán cuối cùng, gánh việc gắn phiên bản cho event, gánh snapshot, và gánh việc dựng lại các bảng đọc. Đội cũng phải học một mô hình tư duy mới, và mọi người mới vào sẽ mất nhiều tuần mới quen. Vì vậy chúng ta chỉ dùng Event Sourcing khi lịch sử và khả năng kiểm toán thật sự là yêu cầu nghiệp vụ, ví dụ với sổ cái tài chính, kế toán, hoặc hệ thống đặt chỗ.

Khi một đồng nghiệp đề xuất chuyển toàn bộ ứng dụng quản lý nhân sự sang Event Sourcing để có kiểm toán, chúng ta nên phản biện theo ba bước. Bước một, chúng ta hỏi lại yêu cầu thật là gì, vì nếu yêu cầu chỉ là biết ai đã sửa gì lúc nào thì một bảng nhật ký kiểm toán đã đủ. Bước hai, chúng ta nêu ra cái giá thật của Event Sourcing, gồm nhất quán cuối cùng, phiên bản event và việc dựng lại. Bước ba, chúng ta đề xuất áp dụng có chọn lọc, nghĩa là chỉ dùng Event Sourcing cho những aggregate thật sự cần tua lại lịch sử. Câu chốt nên là chúng ta chọn theo cán cân chi phí và lợi ích, chứ không chọn vì một mẫu kiến trúc đang được nhắc tới nhiều.

**English (bám cấu trúc tiếng Việt)**

The last part, and also the most important part in terms of judgement, is knowing when not to use Event Sourcing. For an application that is mainly create, read, update and delete, Event Sourcing adds a great deal of complexity without bringing a matching benefit. We will have to carry eventual consistency, carry the versioning of events, carry snapshots, and carry the rebuilding of the read tables. The team also has to learn a new mental model, and every new joiner will need many weeks to get used to it. Therefore we only use Event Sourcing when history and auditability are genuinely a business requirement, for example with a financial ledger, with accounting, or with a booking system.

When a colleague proposes moving the whole HR application to Event Sourcing in order to have auditing, we should push back in three steps. Step one, we ask back what the real requirement is, because if the requirement is only to know who changed what and when then an audit log table is already enough. Step two, we lay out the real price of Event Sourcing, which includes eventual consistency, event versioning and rebuilding. Step three, we propose applying it selectively, which means using Event Sourcing only for the aggregates that genuinely need to replay their history. The closing sentence should be that we choose according to the balance of cost and benefit, rather than choosing because of an architectural pattern that is being talked about a lot.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| quan trọng nhất về mặt phán đoán | the most important part in terms of judgement |
| chủ yếu là tạo, đọc, sửa, xoá | mainly create, read, update and delete |
| mà không mang lại lợi ích tương xứng | without bringing a matching benefit |
| chúng ta sẽ phải gánh | we will have to carry |
| mọi người mới vào | every new joiner |
| thật sự là yêu cầu nghiệp vụ | are genuinely a business requirement |
| chúng ta hỏi lại yêu cầu thật là gì | we ask back what the real requirement is |
| chúng ta nêu ra cái giá thật | we lay out the real price |
| áp dụng có chọn lọc | applying it selectively |
| cán cân chi phí và lợi ích | the balance of cost and benefit |
| đang được nhắc tới nhiều | that is being talked about a lot |

**Thuật ngữ cần nhớ**

- làm phức tạp quá mức cần thiết → **over-engineering**
- khả năng kiểm toán → **auditability**
- yêu cầu nghiệp vụ → **a business requirement**
- áp dụng có chọn lọc → **to apply selectively**
- cán cân chi phí và lợi ích → **the cost-benefit balance**

---

## ⑧ Mô hình ghi nhớ

**Tiếng Việt**

Event Sourcing lưu chuyện đã xảy ra dưới dạng event bất biến, và trạng thái hiện tại là kết quả của việc tua lại; snapshot giúp tua nhanh hơn. CQRS tách phía đọc khỏi phía ghi và không bắt buộc đi cùng Event Sourcing, còn Event Sourcing chỉ đáng dùng khi lịch sử và kiểm toán là yêu cầu thật.

**English (bám cấu trúc tiếng Việt)**

Event Sourcing stores what has happened as immutable events, and the current state is the result of replaying them; a snapshot helps us replay faster. CQRS separates the read side from the write side and is not required to go with Event Sourcing, while Event Sourcing is only worth using when history and auditing are a real requirement.

---

## ⑨ Bảng thuật ngữ tổng hợp

| Tiếng Việt | Tiếng Anh | Ghi chú phát âm |
|---|---|---|
| lưu trạng thái bằng chuỗi sự kiện | event sourcing | *sourcing* = **SOR**-sing |
| bất biến, không sửa được | immutable | i-**MYU**-tə-bl, trọng âm âm thứ hai |
| tua lại chuỗi event | to replay the event stream | |
| dựng lại trạng thái | to rebuild the state | |
| du hành thời gian | time travel | |
| nguồn sự thật | the source of truth | *truth* — âm *th* /θ/ ở cuối, phải bật lưỡi |
| nhật ký kiểm toán | an audit log | *audit* = **OR**-dit, giọng Anh /ˈɔːdɪt/ |
| khả năng kiểm toán | auditability | or-di-tə-**BI**-li-ty |
| sản phẩm phụ | a by-product | |
| bút toán điều chỉnh | an adjusting entry | *adjusting* = ə-**JUS**-ting, chữ *d* gần như không nghe thấy |
| chuỗi event của một thực thể | an event stream | |
| bản chụp trạng thái | a snapshot | |
| hàm thuần | a pure function | *pure* giọng Anh /pjʊə/ — "PIU-ơ" |
| tất định | deterministic | di-ter-mi-**NIS**-tic, trọng âm áp chót |
| gộp dần các event thành trạng thái | to fold the events | |
| tách lệnh ghi khỏi câu đọc | CQRS | đọc từng chữ cái: C-Q-R-S |
| mô hình ghi / mô hình đọc | the write model / the read model | *write* — chữ **w** câm hoàn toàn |
| kho lưu event | the event store | |
| chiếu dữ liệu ra bảng đọc | to project | động từ = prə-**JEKT**; danh từ = **PRO**-ject |
| bảng đọc dựng từ event | a projection | prə-**JEK**-shn |
| độ trễ của projection | projection lag | |
| chỉ mục tìm kiếm | a search index | số nhiều học thuật là *indices*, thông dụng là *indexes* |
| cập nhật giao diện lạc quan | an optimistic update | op-ti-**MIS**-tic |
| dữ liệu tươi mới | fresh data | giọng Anh *data* /ˈdeɪtə/ — "ĐÂY-tơ" |
| trạng thái đang xử lý | a processing state | giọng Anh *process* = **PRO**-cess /ˈprəʊses/ |
| gắn phiên bản cho event | to version an event | *version* giọng Anh /ˈvɜːʃn/ — "VER-shn" |
| bộ nâng phiên bản khi đọc | an upcaster | **UP**-cas-tə |
| schema lỏng | a loose schema | *schema* /ˈskiːmə/ — "SKI-mơ"; *loose* /luːs/ đuôi /s/, khác *lose* /luːz/ |
| di trú dữ liệu | a data migration | mai-**GRAY**-shn |
| viết lại lịch sử | to rewrite history | |
| thực thể gốc trong miền nghiệp vụ | an aggregate | danh từ = **A**-gri-gət, đuôi đọc nhẹ |
| tạo, đọc, sửa, xoá | CRUD | đọc thành một từ /krʌd/ |
| làm phức tạp quá mức | over-engineering | |
| yêu cầu nghiệp vụ | a business requirement | *business* — hai âm tiết "BIZ-nis" |
| áp dụng có chọn lọc | to apply selectively | |
| cán cân chi phí và lợi ích | the cost-benefit balance | |
| nhất quán cuối cùng | eventual consistency | i-**VEN**-chu-al, nghĩa là "rốt cuộc" |
| người mới vào đội | a new joiner | |

---

## ⑩ Bài nói

> **Cách luyện:** bật ghi âm, nói **60–90 giây** cho mỗi đề, **không nhìn tài liệu**. Mỗi bài nói phải dùng ít nhất **sáu thuật ngữ** trong bảng trên. Nói xong thì nghe lại một lượt, đánh dấu chỗ mình ngập ngừng, rồi nói lại chính đề đó thêm một lần nữa.

1. Explain to a junior developer what Event Sourcing changes about how state is stored, and describe what you gain and what you pay for it.

2. A colleague says their system already does Event Sourcing because it writes an audit row on every update. Explain the core difference and why the distinction matters.

3. Your team proposes moving the whole HR application to Event Sourcing so that everything is auditable. Explain why you would push back, and what you would recommend instead.

4. Describe why a snapshot is needed, and describe exactly what breaks if the apply function calls the system clock.

5. Explain the relationship between CQRS and Event Sourcing, and give one example of using CQRS with no Event Sourcing at all.

6. A user saves a record and then reloads the page but the change is not there yet. Describe what is happening and describe three ways you could design around it.

7. When would you accept the long-term cost of event versioning and upcasters, and how would you explain that cost to a product manager who only sees the audit benefit?
