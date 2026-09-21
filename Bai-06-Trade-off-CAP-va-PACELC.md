# Bài 6 — Trade-off articulation, CAP & PACELC
### Bản song ngữ: tiếng Anh bám cấu trúc tiếng Việt

> **Cách dùng:** đọc đoạn tiếng Việt trước → tự dịch trong đầu → so với bản tiếng Anh bên dưới → **đọc to bản tiếng Anh hai lần**. Lần một đọc chậm cho đúng từ, lần hai đọc liền mạch như đang nói với người phỏng vấn. Đây là bài quyết định việc bạn được xếp vào mức senior hay mức trung cấp, vì nó không kiểm tra kiến thức mà kiểm tra **cách phát biểu một quyết định**.

---

## Phần 1 — Không có phương án tốt nhất, chỉ có phương án phù hợp nhất

**Tiếng Việt**

Trong thiết kế hệ thống, không tồn tại phương án tốt nhất, mà chỉ tồn tại phương án phù hợp nhất với yêu cầu đã chốt. Mọi lựa chọn đều cắt đi một thứ để đổi lấy một thứ khác, và việc của chúng ta là nói rõ cả hai vế đó. Một kỹ sư trung cấp thường nói rằng phương án A tốt hơn phương án B, và câu đó gần như luôn rỗng. Một kỹ sư dẫn dắt thì nói rằng với yêu cầu này, tôi chọn A và tôi chấp nhận mất B. Sự khác biệt nằm ở chỗ vế thứ hai cho thấy chúng ta biết mình đang trả giá gì.

Cách nói này cũng bảo vệ chúng ta trong buổi phỏng vấn. Khi chúng ta tuyên bố một phương án là tốt hơn, người phỏng vấn sẽ đưa ra một tình huống mà phương án đó thua, và chúng ta rơi vào thế phải chống đỡ. Khi chúng ta gắn lựa chọn với một yêu cầu cụ thể, người phỏng vấn muốn phản biện thì phải đổi yêu cầu trước, và lúc đó chúng ta chỉ cần đổi lựa chọn theo. Nói cách khác, chúng ta không bảo vệ một công nghệ, chúng ta bảo vệ một cách suy luận. Nếu chúng ta trình bày như thể có một câu trả lời đúng phổ quát, hậu quả là mọi câu hỏi tiếp theo đều biến thành một cuộc tranh cãi mà chúng ta không thắng được.

**English (bám cấu trúc tiếng Việt)**

In system design, there is no best option, there is only the option that fits the agreed requirements best. Every choice cuts away one thing in exchange for another thing, and our job is to state both sides clearly. A mid-level engineer usually says that option A is better than option B, and that sentence is almost always empty. A leading engineer says that for this requirement, I choose A and I accept losing B. The difference lies in the fact that the second half shows that we know what we are paying.

This way of speaking also protects us during the interview. When we declare that one option is better, the interviewer will bring up a situation where that option loses, and we fall into a defensive position. When we tie the choice to a specific requirement, an interviewer who wants to push back has to change the requirement first, and at that point we simply change our choice accordingly. In other words, we are not defending a technology, we are defending a line of reasoning. If we present things as if there were one universally correct answer, the consequence is that every following question turns into an argument that we cannot win.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| yêu cầu đã chốt | the agreed requirements |
| cắt đi một thứ để đổi lấy một thứ khác | cuts away one thing in exchange for another thing |
| nói rõ cả hai vế đó | state both sides clearly |
| câu đó gần như luôn rỗng | that sentence is almost always empty |
| tôi chấp nhận mất B | I accept losing B |
| chúng ta rơi vào thế phải chống đỡ | we fall into a defensive position |
| thì phải đổi yêu cầu trước | has to change the requirement first |
| chúng ta chỉ cần đổi lựa chọn theo | we simply change our choice accordingly |
| chúng ta bảo vệ một cách suy luận | we are defending a line of reasoning |
| một câu trả lời đúng phổ quát | a universally correct answer |

**Thuật ngữ cần nhớ**

- đánh đổi → **a trade-off**
- phù hợp nhất → **the best fit**
- phát biểu rõ ràng → **to articulate**
- yêu cầu đã chốt → **the agreed requirements**
- cách suy luận → **a line of reasoning**

---

## Phần 2 — Định lý CAP và vì sao câu "chọn hai trong ba" gây hiểu lầm

**Tiếng Việt**

Định lý CAP nói về ba tính chất của một hệ phân tán: tính nhất quán, tính sẵn sàng và khả năng chịu phân mảnh mạng. Cách phát biểu phổ biến là chúng ta chỉ được chọn hai trong ba, và cách phát biểu đó gây hiểu lầm nghiêm trọng. Lý do là phân mảnh mạng không phải là một lựa chọn của chúng ta, mà là một sự kiện luôn có thể xảy ra khi các máy nói chuyện qua mạng. Cáp đứt, chuyển mạch hỏng, một vùng mất kết nối, và không kỹ sư nào ngăn được những chuyện đó. Vì vậy, phát biểu đúng hơn là khi phân mảnh xảy ra, chúng ta phải chọn giữa nhất quán và sẵn sàng.

Chọn nhất quán nghĩa là chúng ta từ chối phục vụ khi không chắc dữ liệu đúng, và người dùng nhận về lỗi thay vì nhận về câu trả lời sai. Chọn sẵn sàng nghĩa là chúng ta vẫn trả lời, nhưng chúng ta chấp nhận rằng hai phía của phân mảnh có thể thấy hai phiên bản dữ liệu khác nhau. Đây là một quyết định nghiệp vụ chứ không phải một quyết định kỹ thuật thuần tuý, nên chúng ta phải hỏi phía sản phẩm. Với một giao dịch chuyển tiền, trả lỗi thì phiền nhưng trả sai thì thành thảm hoạ. Với một bộ đếm lượt xem, trả một con số hơi cũ thì không ai chết, nên chúng ta chọn tiếp tục phục vụ.

**English (bám cấu trúc tiếng Việt)**

The CAP theorem talks about three properties of a distributed system: consistency, availability and partition tolerance. The common way of stating it is that we can only pick two out of three, and that way of stating it is seriously misleading. The reason is that a network partition is not a choice of ours, it is an event that can always happen when machines talk over a network. Cables get cut, switches break, one region loses connectivity, and no engineer can prevent those things. Therefore, the more correct statement is that when a partition happens, we have to choose between consistency and availability.

Choosing consistency means that we refuse to serve when we are not sure the data is correct, and the user gets an error instead of getting a wrong answer. Choosing availability means that we still answer, but we accept that the two sides of the partition may see two different versions of the data. This is a business decision rather than a purely technical decision, so we have to ask the product side. For a money transfer, returning an error is annoying but returning a wrong result becomes a disaster. For a view counter, returning a slightly old number does not kill anyone, so we choose to keep serving.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| khả năng chịu phân mảnh mạng | partition tolerance |
| cách phát biểu đó gây hiểu lầm nghiêm trọng | that way of stating it is seriously misleading |
| một sự kiện luôn có thể xảy ra | an event that can always happen |
| một vùng mất kết nối | one region loses connectivity |
| khi phân mảnh xảy ra | when a partition happens |
| từ chối phục vụ khi không chắc dữ liệu đúng | refuse to serve when we are not sure the data is correct |
| hai phía của phân mảnh | the two sides of the partition |
| một quyết định kỹ thuật thuần tuý | a purely technical decision |
| trả lỗi thì phiền nhưng trả sai thì thành thảm hoạ | returning an error is annoying but returning a wrong result becomes a disaster |
| thì không ai chết | does not kill anyone |

**Thuật ngữ cần nhớ**

- định lý CAP → **the CAP theorem**
- phân mảnh mạng → **a network partition**
- khả năng chịu phân mảnh → **partition tolerance**
- tính sẵn sàng → **availability**
- tính nhất quán → **consistency**

---

## Phần 3 — PACELC, cách nói chính xác hơn CAP

**Tiếng Việt**

PACELC mở rộng CAP và mô tả thực tế chính xác hơn, nên chúng ta nên dùng nó khi muốn gây ấn tượng đúng cách. Cách đọc của PACELC là: nếu có phân mảnh thì chọn giữa nhất quán và sẵn sàng, còn nếu không có phân mảnh thì chọn giữa nhất quán và độ trễ. Vế thứ hai là phần mà CAP bỏ sót, và nó quan trọng hơn trong đời sống hằng ngày. Lý do là phân mảnh mạng thì hiếm, còn việc phải chờ nhiều bản sao xác nhận thì xảy ra ở từng request. Nói cách khác, ngay cả lúc mạng hoàn toàn khoẻ, tính nhất quán mạnh vẫn bắt chúng ta trả bằng độ trễ.

Chúng ta có thể dùng PACELC để đọc tính cách của một kho dữ liệu chỉ bằng hai chữ cái. Những hệ như DynamoDB hoặc Cassandra nghiêng về sẵn sàng khi phân mảnh và nghiêng về độ trễ thấp khi bình thường, nên chúng thuộc nhóm PA và EL. Những hệ như Spanner nghiêng về nhất quán ở cả hai tình huống, nên chúng thuộc nhóm PC và EC, và chúng trả giá bằng độ trễ ghi cao hơn. Khi chúng ta phát biểu được như vậy trong phỏng vấn, chúng ta cho thấy mình đọc kho dữ liệu qua lăng kính đánh đổi chứ không qua danh tiếng. Nếu chúng ta chỉ nói rằng Cassandra mở rộng tốt, hậu quả là câu trả lời của chúng ta nghe giống một câu quảng cáo hơn là một phân tích.

**English (bám cấu trúc tiếng Việt)**

PACELC extends CAP and describes reality more accurately, so we should use it when we want to make the right impression. The way to read PACELC is: if there is a partition then choose between consistency and availability, else if there is no partition then choose between consistency and latency. The second half is the part that CAP leaves out, and it matters more in everyday life. The reason is that network partitions are rare, while having to wait for several replicas to confirm happens on every single request. In other words, even when the network is completely healthy, strong consistency still makes us pay in latency.

We can use PACELC to read the personality of a datastore with just two letters. Systems like DynamoDB or Cassandra lean towards availability during a partition and lean towards low latency in normal times, so they belong to the PA and EL group. Systems like Spanner lean towards consistency in both situations, so they belong to the PC and EC group, and they pay the price with higher write latency. When we can state it that way in an interview, we show that we read a datastore through the lens of trade-offs rather than through its reputation. If we only say that Cassandra scales well, the consequence is that our answer sounds more like an advertisement than an analysis.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| mô tả thực tế chính xác hơn | describes reality more accurately |
| gây ấn tượng đúng cách | make the right impression |
| là phần mà CAP bỏ sót | is the part that CAP leaves out |
| quan trọng hơn trong đời sống hằng ngày | matters more in everyday life |
| chờ nhiều bản sao xác nhận | wait for several replicas to confirm |
| ngay cả lúc mạng hoàn toàn khoẻ | even when the network is completely healthy |
| đọc tính cách của một kho dữ liệu | read the personality of a datastore |
| nghiêng về sẵn sàng khi phân mảnh | leans towards availability during a partition |
| qua lăng kính đánh đổi | through the lens of trade-offs |
| nghe giống một câu quảng cáo | sounds like an advertisement |

**Thuật ngữ cần nhớ**

- kho dữ liệu → **a datastore**
- bản sao → **a replica**
- độ trễ ghi → **write latency**
- xác nhận → **to acknowledge** / **an acknowledgement**
- lăng kính → **a lens**

---

## Phần 4 — Nhất quán mạnh so với nhất quán cuối cùng

**Tiếng Việt**

Nhất quán mạnh nghĩa là ngay sau khi một lần ghi thành công, mọi lần đọc tiếp theo đều thấy giá trị mới. Nhất quán cuối cùng nghĩa là các bản sao sẽ hội tụ về cùng một giá trị sau một khoảng thời gian, nhưng trong khoảng đó người đọc có thể thấy giá trị cũ. Chúng ta chọn nhất quán mạnh cho tiền bạc, cho tồn kho và cho đặt vé, vì ở những chỗ đó một câu trả lời cũ tạo ra bán quá số lượng. Chúng ta chọn nhất quán cuối cùng cho bảng tin, cho lượt thích và cho bộ đếm lượt xem, vì trễ vài giây thì không ai nhận ra. Khi trả lời, chúng ta phải nêu ví dụ cụ thể như vậy, chứ không nói lý thuyết suông.

Mỗi hướng đều có cái giá mà chúng ta phải nói ra. Nhất quán mạnh làm tăng độ trễ, vì lần ghi phải chờ đủ số bản sao xác nhận trước khi trả về. Nó cũng làm giảm tính sẵn sàng khi có phân mảnh, vì hệ thống thà từ chối còn hơn trả lời sai. Nhất quán cuối cùng thì nhanh và luôn sẵn sàng, nhưng nó bắt tầng ứng dụng xử lý những tình huống khó chịu, ví dụ người dùng vừa sửa hồ sơ xong lại thấy dữ liệu cũ. Nếu chúng ta chọn nhất quán cuối cùng mà không thiết kế cho tình huống đó, hậu quả là đội hỗ trợ nhận về những báo lỗi mà kỹ sư không tài nào tái hiện được.

**English (bám cấu trúc tiếng Việt)**

Strong consistency means that right after one write succeeds, every following read sees the new value. Eventual consistency means that the replicas will converge to the same value after some time, but during that window a reader may see the old value. We choose strong consistency for money, for inventory and for ticket booking, because in those places one stale answer creates overselling. We choose eventual consistency for the feed, for likes and for the view counter, because a delay of a few seconds is something nobody notices. When we answer, we must give concrete examples like these, rather than talking pure theory.

Each direction has a price that we have to say out loud. Strong consistency increases latency, because the write has to wait for enough replicas to confirm before it returns. It also reduces availability during a partition, because the system would rather refuse than answer incorrectly. Eventual consistency is fast and always available, but it makes the application layer handle awkward situations, for example a user who has just edited their profile and then sees the old data. If we choose eventual consistency without designing for that situation, the consequence is that the support team receives bug reports which the engineers cannot reproduce at all.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| ngay sau khi một lần ghi thành công | right after one write succeeds |
| sẽ hội tụ về cùng một giá trị | will converge to the same value |
| trong khoảng đó | during that window |
| tạo ra bán quá số lượng | creates overselling |
| trễ vài giây thì không ai nhận ra | a delay of a few seconds is something nobody notices |
| không nói lý thuyết suông | rather than talking pure theory |
| chờ đủ số bản sao xác nhận | wait for enough replicas to confirm |
| thà từ chối còn hơn trả lời sai | would rather refuse than answer incorrectly |
| những tình huống khó chịu | awkward situations |
| kỹ sư không tài nào tái hiện được | which the engineers cannot reproduce at all |

**Thuật ngữ cần nhớ**

- nhất quán mạnh → **strong consistency**
- nhất quán cuối cùng → **eventual consistency**
- hội tụ → **to converge**
- bán quá số lượng → **overselling**
- đọc phải dữ liệu cũ → **a stale read**

---

## Phần 5 — Bốn trục luôn kéo nhau: độ trễ, thông lượng, chi phí, nhất quán

**Tiếng Việt**

Có bốn trục mà mọi quyết định lớn đều chạm vào, đó là độ trễ, thông lượng, chi phí và tính nhất quán. Cải thiện một trục thường phải hy sinh một trục khác, nên chúng ta không được mơ một phương án tốt trên cả bốn. Nhất quán mạnh làm độ trễ tăng, vì chúng ta phải chờ xác nhận từ nhiều nơi. Cache làm độ trễ giảm rất nhiều, nhưng nó tạo ra dữ liệu cũ nên nó ăn vào trục nhất quán. Thêm bản sao làm tính sẵn sàng tăng, nhưng nó làm chi phí tăng và làm việc đồng bộ phức tạp hơn.

Việc của một kỹ sư dẫn dắt là xếp thứ tự ưu tiên cho bốn trục theo nhu cầu nghiệp vụ, chứ không phải tối ưu cả bốn. Chúng ta hỏi phía sản phẩm rằng nếu buộc phải mất một thứ thì họ chịu mất thứ nào. Với một sàn thương mại điện tử, họ thường chịu độ trễ cao hơn nhưng không chịu bán quá số lượng. Với một nền tảng video, họ chịu con số lượt xem xấp xỉ nhưng không chịu người xem phải chờ. Nếu chúng ta không ép ra được thứ tự ưu tiên này, hậu quả là mọi cuộc họp thiết kế đều quay vòng, vì mỗi người bảo vệ một trục khác nhau mà không ai sai.

**English (bám cấu trúc tiếng Việt)**

There are four axes that every big decision touches, and they are latency, throughput, cost and consistency. Improving one axis usually means sacrificing another axis, so we must not dream of an option that is good on all four. Strong consistency makes latency go up, because we have to wait for confirmation from several places. A cache makes latency go down a lot, but it creates stale data so it eats into the consistency axis. Adding replicas makes availability go up, but it makes the cost go up and makes the synchronisation work more complex.

The job of a leading engineer is to rank the four axes according to the business needs, rather than to optimise all four. We ask the product side which one they are willing to lose if they are forced to lose something. For an e-commerce marketplace, they usually accept higher latency but they do not accept overselling. For a video platform, they accept an approximate view count but they do not accept viewers having to wait. If we cannot force this ranking out of them, the consequence is that every design meeting goes round in circles, because each person defends a different axis and nobody is wrong.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| mọi quyết định lớn đều chạm vào | every big decision touches |
| thường phải hy sinh một trục khác | usually means sacrificing another axis |
| chúng ta không được mơ | we must not dream of |
| nó ăn vào trục nhất quán | it eats into the consistency axis |
| làm việc đồng bộ phức tạp hơn | makes the synchronisation work more complex |
| xếp thứ tự ưu tiên cho bốn trục | rank the four axes |
| nếu buộc phải mất một thứ | if they are forced to lose something |
| con số lượt xem xấp xỉ | an approximate view count |
| ép ra được thứ tự ưu tiên này | force this ranking out of them |
| mọi cuộc họp thiết kế đều quay vòng | every design meeting goes round in circles |

**Thuật ngữ cần nhớ**

- thông lượng → **throughput**
- trục → **an axis** (số nhiều: **axes**)
- hy sinh → **to sacrifice**
- xấp xỉ → **approximate**
- xếp thứ tự ưu tiên → **to rank** / **to prioritise**

---

## Phần 6 — SQL so với NoSQL

**Tiếng Việt**

Chúng ta quyết định giữa SQL và NoSQL dựa trên bốn thứ: kiểu truy cập dữ liệu, yêu cầu về nhất quán, mô hình dữ liệu và quy mô ghi. Chúng ta không quyết định dựa trên câu nói rằng NoSQL nhanh hơn, vì câu đó không có nghĩa nếu không gắn với một khối lượng công việc cụ thể. Với phần lớn hệ thống, một cơ sở dữ liệu quan hệ như Postgres là lựa chọn mặc định an toàn. Nó cho chúng ta giao dịch, phép nối bảng, ràng buộc toàn vẹn và một ngôn ngữ truy vấn mà cả đội đã biết. Chúng ta chỉ rời khỏi mặc định đó khi có một lý do cụ thể mà chúng ta gọi tên được.

Những lý do hợp lệ thường là quy mô ghi cực lớn vượt quá một máy chủ, hoặc lược đồ dữ liệu thay đổi liên tục, hoặc mẫu truy cập đơn giản kiểu khoá và giá trị. Khi trả lời, chúng ta nên nói cả cái mất, vì nhiều kho NoSQL không cho phép nối bảng và không cho giao dịch trải nhiều bản ghi. Điều đó nghĩa là phần logic ghép dữ liệu chuyển từ cơ sở dữ liệu lên tầng ứng dụng, và đội của chúng ta phải viết nhiều mã hơn để giữ dữ liệu đúng. Một cách trả lời rất mạnh là chúng ta nói rằng chúng ta bắt đầu với Postgres, và chúng ta sẽ tách phần có quy mô ghi lớn sang một kho chuyên biệt khi số liệu cho thấy cần. Nếu chúng ta chọn NoSQL từ đầu chỉ vì nó hiện đại, hậu quả là chúng ta mất những đảm bảo miễn phí mà lẽ ra chúng ta được hưởng.

**English (bám cấu trúc tiếng Việt)**

We decide between SQL and NoSQL based on four things: the data access pattern, the consistency requirement, the data model and the write scale. We do not decide based on the sentence that NoSQL is faster, because that sentence means nothing unless it is tied to a specific workload. For most systems, a relational database such as Postgres is the safe default choice. It gives us transactions, joins, integrity constraints and a query language that the whole team already knows. We only leave that default when there is a specific reason that we can name.

The valid reasons are usually a very large write scale that goes beyond one server, or a data schema that changes constantly, or a simple key-value access pattern. When we answer, we should also say what we lose, because many NoSQL stores do not allow joins and do not allow transactions spanning several records. That means the data-joining logic moves from the database up to the application layer, and our team has to write more code to keep the data correct. A very strong way to answer is to say that we start with Postgres, and that we will split off the high-write part into a specialised store when the numbers show that we need to. If we choose NoSQL from day one just because it is modern, the consequence is that we give up guarantees which we would otherwise have had for free.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| kiểu truy cập dữ liệu | the data access pattern |
| câu đó không có nghĩa nếu không gắn với | that sentence means nothing unless it is tied to |
| lựa chọn mặc định an toàn | the safe default choice |
| ràng buộc toàn vẹn | integrity constraints |
| một lý do cụ thể mà chúng ta gọi tên được | a specific reason that we can name |
| vượt quá một máy chủ | goes beyond one server |
| giao dịch trải nhiều bản ghi | transactions spanning several records |
| chuyển từ cơ sở dữ liệu lên tầng ứng dụng | moves from the database up to the application layer |
| khi số liệu cho thấy cần | when the numbers show that we need to |
| những đảm bảo miễn phí | guarantees for free |

**Thuật ngữ cần nhớ**

- cơ sở dữ liệu quan hệ → **a relational database**
- mẫu truy cập → **an access pattern**
- lược đồ → **a schema**
- phép nối bảng → **a join**
- ràng buộc → **a constraint**
- khoá và giá trị → **key-value**

---

## Phần 7 — Khối đơn so với vi dịch vụ

**Tiếng Việt**

Vi dịch vụ không phải là lựa chọn mặc định, và câu nói rằng vi dịch vụ hiện đại nên chúng ta phải dùng là một câu sai. Mỗi lần chúng ta cắt một dịch vụ ra, chúng ta đổi một lời gọi hàm trong bộ nhớ thành một lời gọi qua mạng. Lời gọi qua mạng có thể chậm, có thể thất bại, và nó buộc chúng ta xử lý việc thử lại cùng với tính bất biến khi gọi lại. Chúng ta cũng mất khả năng dùng một giao dịch duy nhất cho hai bảng nằm ở hai dịch vụ khác nhau. Ngoài ra, mỗi dịch vụ thêm vào là thêm một quy trình triển khai, thêm bảng theo dõi và thêm một chỗ để gọi lúc hai giờ sáng.

Vì vậy, chúng ta chỉ tách khi có lý do rõ ràng, chứ không tách theo phong trào. Hai lý do được chấp nhận rộng rãi là nhu cầu mở rộng độc lập cho một phần có tải rất khác, và ranh giới đội ngũ khi nhiều nhóm giẫm chân nhau trên cùng một kho mã. Với một đội nhỏ ở giai đoạn sớm, khối đơn có cấu trúc mô-đun là lựa chọn tốt hơn nhiều. Chúng ta giữ mọi thứ trong một tiến trình, nhưng chúng ta dựng ranh giới rõ trong mã và bắt các mô-đun chỉ gọi nhau qua giao diện công khai. Nhờ đó, khi thật sự cần tách, mỗi mô-đun đã là một đường cắt sẵn sàng và việc tách chỉ mất vài ngày thay vì vài tháng.

**English (bám cấu trúc tiếng Việt)**

Microservices are not the default choice, and the sentence that microservices are modern so we must use them is a wrong sentence. Every time we cut out a service, we turn an in-memory function call into a call over the network. A call over the network can be slow, can fail, and it forces us to handle retries together with idempotency when the call is repeated. We also lose the ability to use a single transaction for two tables sitting in two different services. In addition, every service we add is one more deployment pipeline, one more dashboard and one more thing to be paged about at two in the morning.

Therefore, we only split when there is a clear reason, we do not split because it is fashionable. The two widely accepted reasons are the need to scale one part with a very different load independently, and team boundaries when several groups step on each other in the same codebase. For a small team at an early stage, a modular monolith is a much better choice. We keep everything in one process, but we build clear boundaries in the code and force the modules to call each other only through a public interface. Thanks to that, when we truly need to split, each module is already a ready-made seam and the split takes a few days instead of a few months.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| không tách theo phong trào | we do not split because it is fashionable |
| một lời gọi hàm trong bộ nhớ | an in-memory function call |
| tính bất biến khi gọi lại | idempotency when the call is repeated |
| mất khả năng dùng một giao dịch duy nhất | lose the ability to use a single transaction |
| một chỗ để gọi lúc hai giờ sáng | one more thing to be paged about at two in the morning |
| nhu cầu mở rộng độc lập | the need to scale independently |
| nhiều nhóm giẫm chân nhau trên cùng một kho mã | several groups step on each other in the same codebase |
| khối đơn có cấu trúc mô-đun | a modular monolith |
| chỉ gọi nhau qua giao diện công khai | call each other only through a public interface |
| một đường cắt sẵn sàng | a ready-made seam |

**Thuật ngữ cần nhớ**

- khối đơn → **a monolith**
- khối đơn mô-đun → **a modular monolith**
- vi dịch vụ → **microservices**
- tính bất biến khi lặp lại → **idempotency**
- đường cắt → **a seam**
- quy trình triển khai → **a deployment pipeline**

---

## Phần 8 — Khung bốn bước và cảnh báo về thiết kế thừa

**Tiếng Việt**

Khi được hỏi vì sao chọn A mà không chọn B, chúng ta trả lời theo bốn bước cố định. Bước một, chúng ta nêu các tiêu chí đang cân, ví dụ độ trễ, chi phí, tính nhất quán và độ phức tạp vận hành. Bước hai, chúng ta so sánh hai phương án trên từng tiêu chí, chứ không nói chung chung. Bước ba, chúng ta gắn lựa chọn với một yêu cầu đã chốt từ đầu buổi, ví dụ vì chúng ta cần nhất quán mạnh cho tồn kho. Bước bốn, chúng ta thừa nhận nhược điểm của phương án mình chọn và nói vì sao nhược điểm đó chấp nhận được. Bốn bước này nói gọn trong một phút, và chúng biến một câu trả lời cảm tính thành một lập luận có thể kiểm tra.

Mặt còn lại của cùng một kỹ năng là biết khi nào nên nói không với sự phức tạp. Nếu chúng ta thêm phân mảnh, vi dịch vụ, hàng đợi và nhiều vùng cho một sản phẩm mới có một trăm người dùng, đó là thiết kế thừa. Trong phỏng vấn cho vị trí dẫn dắt, thiết kế thừa bị trừ điểm nặng hơn là thiếu kiến thức, vì nó cho thấy chúng ta thiếu phán đoán. Nguyên tắc an toàn là chúng ta bắt đầu đơn giản và chỉ mở rộng khi yêu cầu thật sự đòi hỏi. Cách nói ghi điểm là chúng ta trình bày phương án đơn giản trước, rồi nói rõ chúng ta sẽ chuyển sang phương án phức tạp hơn khi chỉ số nào vượt ngưỡng nào.

**English (bám cấu trúc tiếng Việt)**

When we are asked why we chose A and not B, we answer in four fixed steps. Step one, we state the criteria we are weighing, for example latency, cost, consistency and operational complexity. Step two, we compare the two options on each criterion, rather than speaking in general terms. Step three, we tie the choice to a requirement that was agreed at the start of the session, for example because we need strong consistency for inventory. Step four, we admit the drawback of the option we chose and say why that drawback is acceptable. These four steps take about a minute to say, and they turn a gut-feeling answer into an argument that can be checked.

The other side of the same skill is knowing when to say no to complexity. If we add sharding, microservices, queues and multiple regions for a new product with one hundred users, that is over-engineering. In an interview for a lead position, over-engineering costs more points than missing knowledge, because it shows that we lack judgement. The safe principle is that we start simple and only scale when the requirements really demand it. The way of speaking that scores is that we present the simple option first, and then say clearly that we will move to the more complex option when which metric crosses which threshold.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| nêu các tiêu chí đang cân | state the criteria we are weighing |
| trên từng tiêu chí | on each criterion |
| gắn lựa chọn với một yêu cầu đã chốt | tie the choice to an agreed requirement |
| thừa nhận nhược điểm | admit the drawback |
| nói gọn trong một phút | take about a minute to say |
| một câu trả lời cảm tính | a gut-feeling answer |
| một lập luận có thể kiểm tra | an argument that can be checked |
| biết khi nào nên nói không với sự phức tạp | knowing when to say no to complexity |
| bị trừ điểm nặng hơn là thiếu kiến thức | costs more points than missing knowledge |
| khi chỉ số nào vượt ngưỡng nào | when which metric crosses which threshold |

**Thuật ngữ cần nhớ**

- tiêu chí → **a criterion** (số nhiều: **criteria**)
- độ phức tạp vận hành → **operational complexity**
- thiết kế thừa → **over-engineering**
- phán đoán → **judgement**
- nhược điểm → **a drawback**

---

## Mô hình ghi nhớ

**Tiếng Việt**

Không có phương án tốt nhất, chỉ có phương án phù hợp nhất. Chúng ta nêu tiêu chí, so sánh hai phương án, gắn với yêu cầu, rồi thừa nhận nhược điểm, và chúng ta nhớ rằng PACELC mô tả đúng hơn câu chọn hai trong ba của CAP.

**English (bám cấu trúc tiếng Việt)**

There is no best option, there is only the best-fitting option. We state the criteria, compare the two options, tie them to the requirements, and then admit the drawback, and we remember that PACELC describes reality better than the pick-two-out-of-three version of CAP.

---

## Bảng thuật ngữ tổng hợp

| Tiếng Việt | Tiếng Anh | Ghi chú phát âm |
|---|---|---|
| đánh đổi | a trade-off | |
| phù hợp nhất | the best fit | |
| phát biểu rõ ràng | to articulate | a-**TI**-kiu-lêit (động từ), trọng âm âm thứ hai |
| yêu cầu đã chốt | the agreed requirements | *requirement* — ri-**KOAI**-ơ-mần, trọng âm âm thứ hai |
| cách suy luận | a line of reasoning | |
| định lý CAP | the CAP theorem | *theorem* — "**THI**-ơ-rợm", âm **th** /θ/ đầu |
| phân mảnh mạng | a network partition | *partition* — pa-**TI**-shợn, trọng âm âm thứ hai |
| khả năng chịu phân mảnh | partition tolerance | *tolerance* — "**TO**-lơ-rợns", trọng âm đầu |
| tính sẵn sàng | availability | a-vây-lơ-**BI**-li-ti, trọng âm âm thứ tư |
| tính nhất quán | consistency | cần-**SIS**-tần-si, trọng âm âm thứ hai |
| kho dữ liệu | a datastore | |
| bản sao | a replica | Anh /ˈreplɪkə/ — "**RE**-pli-cơ", trọng âm đầu |
| độ trễ | latency | /ˈleɪtənsi/ — "LÂY-tân-si", âm đầu là "lây" |
| xác nhận | to acknowledge | ợc-**NO**-lịj, chữ *k* đầu **câm**, trọng âm âm thứ hai |
| nhất quán mạnh | strong consistency | |
| nhất quán cuối cùng | eventual consistency | *eventual* — i-**VEN**-chu-ợl, trọng âm âm thứ hai |
| hội tụ | to converge | cần-**VƠJ**, trọng âm âm thứ hai |
| bán quá số lượng | overselling | |
| đọc phải dữ liệu cũ | a stale read | *stale* /steɪl/ — "stêi-l", có âm **st** đầu rõ |
| tồn kho | inventory | Anh /ˈɪnvəntri/ — "IN-vần-tri", trọng âm đầu, ba âm tiết |
| thông lượng | throughput | âm **th** /θ/ đầu, "THRU-put", không đọc thành "trú" |
| trục | an axis / axes | số ít "**AK**-sịs", số nhiều **axes** đọc "AK-si-iz" |
| hy sinh | to sacrifice | "**SA**-cri-fais", trọng âm đầu, đuôi đọc như *fice* |
| xấp xỉ | approximate | ơ-**PROK**-si-mợt (tính từ), trọng âm âm thứ hai |
| cơ sở dữ liệu quan hệ | a relational database | |
| mẫu truy cập | an access pattern | |
| lược đồ | a schema | /ˈskiːmə/ — "SKI-mơ", không đọc "sê-ma" |
| phép nối bảng | a join | |
| ràng buộc | a constraint | cần-**STRÂYNT**, có cụm phụ âm **-str** và **-nt** cuối |
| khoá và giá trị | key-value | |
| giao dịch | a transaction | |
| khối đơn | a monolith | "**MO**-nơ-lith", âm **th** cuối /θ/ |
| khối đơn mô-đun | a modular monolith | *modular* — "**MO**-điu-lơ", trọng âm đầu |
| vi dịch vụ | microservices | |
| tính bất biến khi lặp lại | idempotency | ai-**DEM**-pơ-tần-si, trọng âm âm thứ hai |
| đường cắt | a seam | /siːm/ — đọc như *seem* |
| quy trình triển khai | a deployment pipeline | |
| tiêu chí | a criterion / criteria | số ít "crai-**TI**-ri-ợn", số nhiều **criteria** "crai-TI-ri-ơ" |
| độ phức tạp vận hành | operational complexity | |
| thiết kế thừa | over-engineering | âu-vơ-en-jị-**NIA**, trọng âm rơi vào âm **cuối** |
| phán đoán | judgement | "**JĂJ**-mần", trọng âm đầu |
| nhược điểm | a drawback | |
| ngưỡng | a threshold | âm **th** /θ/ đầu, "THRESH-hâuld" |

---

## Bài nói

> **Cách luyện:** bật ghi âm, nói **60–90 giây** cho mỗi đề, **không nhìn tài liệu** khi nói. Mỗi bài phải dùng ít nhất **sáu thuật ngữ** trong bảng trên. Với mọi đề có chữ *choose* hoặc *why*, hãy ép mình đi đúng bốn bước ở Phần 8: tiêu chí → so sánh → gắn yêu cầu → thừa nhận nhược điểm.

1. **Explain to a junior engineer** why the phrase "CAP means pick two out of three" is misleading, and what the correct statement is.

2. **A colleague says:** *"NoSQL is faster than SQL, so let's use DynamoDB for the whole product."* **Explain what is wrong with that**, and say which four things you would look at before deciding.

3. **Someone on your team wants to** build the MVP with microservices, Kafka and two regions for a product that has one hundred users. **Explain why you would push back**, and describe what you would build instead and when you would revisit the decision.

4. **When would you choose** strong consistency over eventual consistency? Give one feature for each, and state the price you pay in both directions.

5. **Describe what happens when** a network partition hits a system that has chosen availability, from the point of view of two users on the two sides of the partition.

6. **Explain to a product manager** why you cannot have low latency, strong consistency and low cost all at the same time, and ask them to rank the three.

7. **Explain to a junior engineer** how you would classify Postgres, Cassandra and Spanner using PACELC, and what that classification tells you before you read any benchmark.
