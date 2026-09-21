# Bài 16 — Phán đoán Tech Lead: hiểu để chỉ huy & kiểm tra
### Bản song ngữ: tiếng Anh bám cấu trúc tiếng Việt

> **Cách dùng:** đọc đoạn tiếng Việt trước → tự dịch trong đầu → so với bản tiếng Anh bên dưới → **đọc to bản tiếng Anh hai lần**. Lần một đọc chậm cho đúng từ, lần hai đọc liền mạch như đang nói với người phỏng vấn. Đây là bài cuối và cũng là bài đổi vai: từ người trả lời đúng thành người phán đoán đúng, nên phần lớn từ vựng ở đây là từ dùng để **đặt câu hỏi cho người khác**.

---

## Phần 1 — Đổi vai: từ trả lời đúng sang phán đoán đúng

**Tiếng Việt**

Mười lăm bài trước đã cho chúng ta các công cụ, còn bài này dạy chúng ta khi nào nên rút công cụ nào ra. Một kỹ sư dẫn dắt không cần tự tay gõ lại từng dòng mã của cả hệ thống. Điều họ cần là hai năng lực rất khác nhau so với năng lực của một người viết mã giỏi. Năng lực thứ nhất là phản biện một lựa chọn kiến trúc bằng những câu hỏi đúng chỗ. Năng lực thứ hai là dự đoán cái gì sẽ vỡ ở môi trường thật, ngay cả khi bản vẽ trông rất đẹp.

Sự khác biệt này lộ ra rõ nhất qua câu hỏi mà mỗi người đặt ra khi nhìn một thiết kế. Người mới hỏi thiết kế này hoạt động như thế nào, vì họ đang cố hiểu cơ chế. Người dẫn dắt hỏi thiết kế này sai ở đâu, đắt ở đâu, vỡ khi nào, và chúng ta đo bằng gì. Bốn câu hỏi đó không đòi hỏi chúng ta thuộc mọi chi tiết, chúng chỉ đòi hỏi chúng ta biết chỗ nào đáng nghi. Nói cách khác, chúng ta không nhớ để gõ, chúng ta hiểu để chỉ huy, và chúng ta tra cứu phần còn lại khi cần.

**English (bám cấu trúc tiếng Việt)**

The fifteen previous lessons gave us the tools, while this lesson teaches us when to reach for which tool. A leading engineer does not need to type out every line of the system's code themselves. What they need is two abilities quite different from the abilities of a strong coder. The first ability is to challenge an architectural choice with the right questions in the right places. The second ability is to predict what will break in the real environment, even when the drawing looks very nice.

This difference shows up most clearly through the question each person asks when they look at a design. A newcomer asks how this design works, because they are trying to understand the mechanism. A leader asks where this design is wrong, where it is expensive, when it will break, and what we measure it with. Those four questions do not require us to know every detail by heart, they only require us to know which places are suspicious. In other words, we do not memorise in order to type, we understand in order to lead, and we look up the rest when we need it.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| khi nào nên rút công cụ nào ra | when to reach for which tool |
| không cần tự tay gõ lại từng dòng mã | does not need to type out every line of code themselves |
| bằng những câu hỏi đúng chỗ | with the right questions in the right places |
| ngay cả khi bản vẽ trông rất đẹp | even when the drawing looks very nice |
| lộ ra rõ nhất qua câu hỏi mà mỗi người đặt ra | shows up most clearly through the question each person asks |
| vì họ đang cố hiểu cơ chế | because they are trying to understand the mechanism |
| chúng ta đo bằng gì | what we measure it with |
| biết chỗ nào đáng nghi | know which places are suspicious |
| chúng ta không nhớ để gõ | we do not memorise in order to type |
| chúng ta hiểu để chỉ huy | we understand in order to lead |

**Thuật ngữ cần nhớ**

- phản biện, chất vấn → **to challenge**
- phán đoán → **judgement**
- lựa chọn kiến trúc → **an architectural choice**
- môi trường thật → **the real environment**
- năng lực → **an ability** / **a capability**

---

## Phần 2 — Khung năm câu hỏi để phản biện một đề xuất công nghệ

**Tiếng Việt**

Khi ai đó đề xuất một công nghệ mới, chúng ta không phản đối và cũng không đồng ý ngay. Chúng ta hỏi năm câu theo thứ tự, và năm câu này áp dụng được cho mọi đề xuất chứ không riêng một lĩnh vực. Câu thứ nhất là bản chất của bài toán này là gì, ví dụ đây là tri thức hay thay đổi hay là một phong cách cố định. Câu thứ hai là công cụ được đề xuất có khớp với bản chất đó không, vì phần lớn quyết định sai là do lệch giữa hai thứ này. Câu thứ ba là chi phí thay đổi và chi phí vận hành sẽ ra sao, nghĩa là mỗi lần cập nhật thì chúng ta phải làm gì.

Câu thứ tư là có cách nào đơn giản hơn đạt được khoảng tám mươi phần trăm kết quả không. Câu hỏi này rất mạnh vì nó buộc người đề xuất so sánh với một phương án cơ sở, thay vì chỉ mô tả phương án họ thích. Câu thứ năm là chúng ta đo bằng chỉ số nào để biết lựa chọn này đúng hay sai. Nếu một đề xuất không có chỉ số đi kèm, thì đó không phải là một quyết định kỹ thuật mà là một niềm tin. Năm câu này hỏi hết chưa tới hai phút, nhưng chúng lọc được phần lớn những đề xuất chạy theo xu hướng.

**English (bám cấu trúc tiếng Việt)**

When somebody proposes a new technology, we neither object nor agree straight away. We ask five questions in order, and these five questions apply to every proposal rather than to one field only. The first question is what the nature of this problem is, for example whether this is knowledge that changes often or a fixed style. The second question is whether the proposed tool matches that nature, because most wrong decisions come from a mismatch between these two things. The third question is what the cost of change and the cost of operation will be, meaning what we have to do on each update.

The fourth question is whether there is a simpler way that achieves about eighty percent of the result. This question is very powerful because it forces the proposer to compare against a baseline, rather than only describing the option they like. The fifth question is which metric we measure in order to know whether this choice is right or wrong. If a proposal comes with no metric attached, then it is not a technical decision but a belief. These five questions take less than two minutes to ask, but they filter out most of the proposals that are merely chasing a trend.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| chúng ta không phản đối và cũng không đồng ý ngay | we neither object nor agree straight away |
| bản chất của bài toán này là gì | what the nature of this problem is |
| tri thức hay thay đổi | knowledge that changes often |
| do lệch giữa hai thứ này | from a mismatch between these two things |
| chi phí thay đổi và chi phí vận hành | the cost of change and the cost of operation |
| đạt được khoảng tám mươi phần trăm kết quả | achieves about eighty percent of the result |
| so sánh với một phương án cơ sở | compare against a baseline |
| không có chỉ số đi kèm | comes with no metric attached |
| không phải là một quyết định kỹ thuật mà là một niềm tin | is not a technical decision but a belief |
| những đề xuất chạy theo xu hướng | proposals that are merely chasing a trend |

**Thuật ngữ cần nhớ**

- đề xuất → **a proposal**
- bản chất bài toán → **the nature of the problem**
- lệch, không khớp → **a mismatch**
- phương án cơ sở → **a baseline**
- chỉ số đo → **a metric**

---

## Phần 3 — Một ví dụ áp khung: fine-tune so với truy hồi

**Tiếng Việt**

Giả sử có người đề xuất tinh chỉnh một mô hình ngôn ngữ để làm trợ lý trả lời theo tài liệu nội bộ. Nghe qua thì có vẻ hiện đại, nhưng chúng ta áp khung năm câu vào và vấn đề lộ ra ngay ở câu đầu tiên. Bản chất bài toán ở đây là tri thức hay thay đổi, vì tài liệu nội bộ được sửa hằng tuần. Trong khi đó, tinh chỉnh là việc nướng tri thức vào trọng số của mô hình, nên mỗi lần tài liệu đổi là chúng ta phải huấn luyện lại. Vậy là công cụ không khớp bản chất, và đó đã đủ để chúng ta dừng lại và đề xuất hướng khác.

Hướng đúng ở đây là truy hồi rồi sinh, tức là chúng ta để tài liệu ở bên ngoài mô hình. Khi có câu hỏi, chúng ta tìm những đoạn tài liệu liên quan rồi đưa chúng vào ngữ cảnh cho mô hình trả lời. Nhờ vậy, việc cập nhật tài liệu chỉ là cập nhật chỉ mục chứ không phải huấn luyện lại, và câu trả lời trích được nguồn cụ thể. Chúng ta cũng nên nói rõ khi nào thì tinh chỉnh mới thật sự hợp lý, để lời phản biện không nghe như phủ định sạch trơn. Tinh chỉnh hợp khi chúng ta cần đổi phong cách, đổi định dạng đầu ra, hoặc dạy một kỹ năng ổn định, chứ không hợp để nhồi tri thức hay thay đổi.

**English (bám cấu trúc tiếng Việt)**

Suppose somebody proposes fine-tuning a language model to build an assistant that answers from internal documents. It sounds modern at first, but we apply the five-question framework and the problem shows up right at the first question. The nature of the problem here is knowledge that changes often, because the internal documents are edited every week. Meanwhile, fine-tuning is the act of baking knowledge into the model's weights, so every time the documents change we have to retrain. So the tool does not match the nature, and that alone is enough for us to stop and propose a different direction.

The right direction here is retrieval followed by generation, that is, we keep the documents outside the model. When a question comes, we find the relevant passages of the documents and then put them into the context for the model to answer from. Thanks to that, updating the documents is only updating an index rather than retraining, and the answer can cite a specific source. We should also say clearly when fine-tuning really is the right fit, so that our challenge does not sound like a flat rejection. Fine-tuning fits when we need to change the style, change the output format, or teach a stable skill, but it does not fit for stuffing in knowledge that changes often.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| nghe qua thì có vẻ hiện đại | it sounds modern at first |
| vấn đề lộ ra ngay ở câu đầu tiên | the problem shows up right at the first question |
| việc nướng tri thức vào trọng số của mô hình | the act of baking knowledge into the model's weights |
| chúng ta phải huấn luyện lại | we have to retrain |
| đủ để chúng ta dừng lại và đề xuất hướng khác | enough for us to stop and propose a different direction |
| đưa chúng vào ngữ cảnh cho mô hình trả lời | put them into the context for the model to answer from |
| câu trả lời trích được nguồn cụ thể | the answer can cite a specific source |
| không nghe như phủ định sạch trơn | does not sound like a flat rejection |
| dạy một kỹ năng ổn định | teach a stable skill |
| nhồi tri thức hay thay đổi | stuffing in knowledge that changes often |

**Thuật ngữ cần nhớ**

- tinh chỉnh mô hình → **fine-tuning**
- truy hồi rồi sinh → **retrieval-augmented generation (RAG)**
- trọng số mô hình → **the model weights**
- bịa thông tin → **to hallucinate**
- trích nguồn → **to cite a source**

---

## Phần 4 — Vì sao "đúng trên giấy" thường vỡ ở môi trường thật

**Tiếng Việt**

Mọi thiết kế trên bảng trắng đều ngầm giả định ba điều rất đẹp về thế giới. Giả định thứ nhất là tải phân bố đều, giả định thứ hai là mạng luôn hoạt động, và giả định thứ ba là dữ liệu phân bố cân đối. Môi trường thật thì ngược lại cả ba: tải dồn thành gai, mạng thỉnh thoảng đứt, và dữ liệu luôn lệch về một phía. Vì vậy, việc của một kỹ sư dẫn dắt là nhìn một sơ đồ đẹp rồi chỉ ra đúng chỗ nó sẽ vỡ. Đây là năng lực khó dạy nhất, vì nó thường đến từ việc đã từng phải thức dậy lúc ba giờ sáng.

Điều may mắn là danh sách các tử huyệt không dài, và chúng ta có thể học thuộc. Chúng ta nên nhớ mười thứ: khoá nóng, khởi động nguội, đổ xô vào cache, cạn pool kết nối, phân mảnh mạng, cơn bão thử lại, chi phí ở tải thật, thiếu khả năng quan sát, thiếu đường lùi khi triển khai, và lệch đồng hồ giữa các máy. Với mỗi thứ, chúng ta nên thuộc một dấu hiệu sớm và một cách đo. Ví dụ, khoá nóng lộ ra qua độ trễ p99 của đúng một mảnh, còn cạn pool lộ ra qua thời gian chờ lấy kết nối. Nếu chúng ta ném mười câu hỏi này vào bất kỳ thiết kế nào, chúng ta gần như luôn tìm ra ít nhất ba điểm đáng bàn.

**English (bám cấu trúc tiếng Việt)**

Every whiteboard design silently assumes three very pretty things about the world. The first assumption is that the load is evenly spread, the second assumption is that the network always works, and the third assumption is that the data is evenly distributed. The real environment is the opposite on all three: the load arrives in spikes, the network occasionally breaks, and the data is always skewed to one side. Therefore, the job of a leading engineer is to look at a nice diagram and then point out exactly where it will break. This is the hardest ability to teach, because it usually comes from having had to wake up at three in the morning.

Fortunately, the list of fatal weak points is not long, and we can learn it by heart. We should remember ten of them: hot keys, cold starts, cache stampedes, connection pool exhaustion, network partitions, retry storms, cost at real load, missing observability, missing rollback paths on deployment, and clock skew between machines. For each of them, we should know one early sign and one way to measure it. For example, a hot key reveals itself through the p99 latency of exactly one shard, while pool exhaustion reveals itself through the connection acquisition wait time. If we throw these ten questions at any design, we almost always find at least three points worth discussing.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| ngầm giả định ba điều rất đẹp về thế giới | silently assumes three very pretty things about the world |
| tải dồn thành gai | the load arrives in spikes |
| dữ liệu luôn lệch về một phía | the data is always skewed to one side |
| chỉ ra đúng chỗ nó sẽ vỡ | point out exactly where it will break |
| năng lực khó dạy nhất | the hardest ability to teach |
| đã từng phải thức dậy lúc ba giờ sáng | having had to wake up at three in the morning |
| danh sách các tử huyệt không dài | the list of fatal weak points is not long |
| thiếu đường lùi khi triển khai | missing rollback paths on deployment |
| lệch đồng hồ giữa các máy | clock skew between machines |
| lộ ra qua thời gian chờ lấy kết nối | reveals itself through the connection acquisition wait time |

**Thuật ngữ cần nhớ**

- điểm tử huyệt → **a fatal weak point**
- giả định ngầm → **a silent assumption**
- lệch đồng hồ → **clock skew**
- đường lùi → **a rollback path**
- dấu hiệu sớm → **an early sign**

---

## Phần 5 — Chi phí là một tiêu chí thiết kế, không phải chuyện của kế toán

**Tiếng Việt**

Một kiến trúc có thể hoàn toàn đúng về mặt kỹ thuật mà vẫn là một kiến trúc tồi. Lý do là nó chạy được nhưng hoá đơn hằng tháng lớn gấp mười lần mức mà công ty chịu được. Ba khoản hay bị bỏ quên là chi phí dữ liệu đi ra, chi phí lưu lượng giữa các vùng sẵn sàng, và chi phí của tính toán không máy chủ ở tải đều. Cả ba khoản này đều không xuất hiện trong lúc chúng ta thử nghiệm, vì lúc đó lưu lượng còn rất nhỏ. Chúng chỉ hiện ra ở tháng đầu tiên sau khi sản phẩm thành công, và lúc đó việc sửa đã rất đắt.

Vì vậy, chúng ta nên đưa chi phí thành một tiêu chí ngay trong buổi thiết kế, chứ không để nó cho bộ phận tài chính phát hiện sau. Cách làm đơn giản là với mỗi thành phần, chúng ta ước lượng chi phí ở mức tải dự kiến chứ không ở mức tải hiện tại. Chúng ta cũng nên hỏi một câu rất hữu ích: nếu lưu lượng tăng gấp mười, khoản nào tăng gấp mười theo. Nếu câu trả lời là gần như mọi khoản, thì kiến trúc của chúng ta không có tính kinh tế theo quy mô. Trong phỏng vấn cho vị trí dẫn dắt, việc chủ động nhắc tới chi phí là một tín hiệu rất mạnh, vì nó cho thấy chúng ta nghĩ như người phải ký duyệt ngân sách.

**English (bám cấu trúc tiếng Việt)**

An architecture can be completely correct technically and still be a bad architecture. The reason is that it works but the monthly bill is ten times larger than what the company can bear. The three items most often forgotten are egress cost, the cost of traffic between availability zones, and the cost of serverless compute at steady load. All three of these items fail to appear while we are testing, because the traffic is still very small then. They only show up in the first month after the product succeeds, and by then fixing it is very expensive.

Therefore, we should make cost a criterion right inside the design session, rather than leaving it for the finance department to discover later. The simple way to do it is that for each component, we estimate the cost at the expected load rather than at the current load. We should also ask one very useful question: if the traffic grows ten times, which items grow ten times with it. If the answer is almost every item, then our architecture has no economies of scale. In an interview for a lead position, volunteering the cost topic is a very strong signal, because it shows that we think like the person who has to sign off the budget.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| hoàn toàn đúng về mặt kỹ thuật | completely correct technically |
| lớn gấp mười lần mức mà công ty chịu được | ten times larger than what the company can bear |
| ba khoản hay bị bỏ quên | the three items most often forgotten |
| lưu lượng giữa các vùng sẵn sàng | traffic between availability zones |
| đều không xuất hiện trong lúc chúng ta thử nghiệm | fail to appear while we are testing |
| lúc đó việc sửa đã rất đắt | by then fixing it is very expensive |
| để nó cho bộ phận tài chính phát hiện sau | leaving it for the finance department to discover later |
| khoản nào tăng gấp mười theo | which items grow ten times with it |
| không có tính kinh tế theo quy mô | has no economies of scale |
| người phải ký duyệt ngân sách | the person who has to sign off the budget |

**Thuật ngữ cần nhớ**

- chi phí dữ liệu đi ra → **egress cost**
- vùng sẵn sàng → **an availability zone**
- tải dự kiến → **the expected load**
- tính kinh tế theo quy mô → **economies of scale**
- ngân sách → **a budget**

---

## Phần 6 — Khả năng quan sát và đường lùi là bắt buộc

**Tiếng Việt**

Một hệ thống không có khả năng quan sát là một hệ thống mà chúng ta chỉ biết nó hỏng khi khách hàng gọi điện. Vì vậy, chúng ta nên coi chỉ số, nhật ký và truy vết là phần bắt buộc của thiết kế, chứ không phải phần thêm vào sau. Câu hỏi kiểm tra rất đơn giản: nếu thành phần này bắt đầu chậm lại lúc hai giờ sáng, chúng ta nhìn vào đâu để biết. Nếu không ai trả lời được câu đó, thiết kế chưa hoàn chỉnh dù mọi hộp trên bản vẽ đều đúng. Chúng ta cũng nên gắn một cảnh báo cho mỗi giả định quan trọng, để khi giả định sai thì chúng ta biết trước khách hàng.

Đường lùi cũng quan trọng ngang như vậy, và nó thường bị bỏ quên trong phần di trú dữ liệu. Một thay đổi lược đồ chạy vào giờ cao điểm có thể khoá bảng và làm ngừng dịch vụ, dù bản thân câu lệnh hoàn toàn hợp lệ. Cách an toàn là chúng ta triển khai theo nhiều bước tương thích ngược, ví dụ thêm cột mới trước, ghi vào cả hai chỗ, rồi mới bỏ cột cũ sau. Chúng ta cũng nên triển khai dần cho một phần nhỏ người dùng trước, rồi mở rộng khi các chỉ số vẫn khoẻ. Nếu chúng ta triển khai một lượt cho tất cả và không có đường lùi, hậu quả là mọi lỗi nhỏ đều biến thành một sự cố toàn hệ thống.

**English (bám cấu trúc tiếng Việt)**

A system without observability is a system where we only learn that it is broken when a customer phones us. Therefore, we should treat metrics, logs and traces as a mandatory part of the design, not as a part added afterwards. The test question is very simple: if this component starts slowing down at two in the morning, where do we look to find out. If nobody can answer that, the design is not complete even though every box on the drawing is correct. We should also attach an alert to each important assumption, so that when the assumption turns out wrong we know before the customer does.

The rollback path matters just as much, and it is usually forgotten in the data migration part. A schema change run at peak hours can lock a table and stop the service, even though the statement itself is perfectly valid. The safe way is that we deploy in several backwards-compatible steps, for example adding the new column first, writing to both places, and only dropping the old column afterwards. We should also roll out gradually to a small portion of users first, and then widen it when the metrics still look healthy. If we deploy to everybody at once with no rollback path, the consequence is that every small bug turns into a system-wide incident.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| chỉ biết nó hỏng khi khách hàng gọi điện | only learn that it is broken when a customer phones us |
| chứ không phải phần thêm vào sau | not as a part added afterwards |
| chúng ta nhìn vào đâu để biết | where do we look to find out |
| dù mọi hộp trên bản vẽ đều đúng | even though every box on the drawing is correct |
| chúng ta biết trước khách hàng | we know before the customer does |
| bị bỏ quên trong phần di trú dữ liệu | forgotten in the data migration part |
| có thể khoá bảng và làm ngừng dịch vụ | can lock a table and stop the service |
| nhiều bước tương thích ngược | several backwards-compatible steps |
| rồi mới bỏ cột cũ sau | and only dropping the old column afterwards |
| biến thành một sự cố toàn hệ thống | turns into a system-wide incident |

**Thuật ngữ cần nhớ**

- khả năng quan sát → **observability**
- truy vết → **a trace**
- di trú dữ liệu → **a data migration**
- tương thích ngược → **backwards-compatible**
- triển khai dần → **a gradual rollout**

---

## Phần 7 — Phản biện câu "chạy ổn ở môi trường thử nghiệm rồi"

**Tiếng Việt**

Có một câu rất hay gặp từ phía quản lý: hệ thống đã chạy ổn ở môi trường thử nghiệm một tuần rồi, cứ đưa lên môi trường thật thôi. Chúng ta không nên bác bỏ thẳng, mà nên chỉ ra chính xác những gì môi trường thử nghiệm không tái hiện được. Thứ nhất, tải ở đó nhỏ, nên khoá nóng, phân vùng lệch và cạn pool kết nối chưa có cơ hội lộ ra. Thứ hai, ở đó không có lưu lượng thật, nên khởi động nguội, đổ xô vào cache và cơn bão thử lại chưa từng xảy ra. Thứ ba, dữ liệu ở đó sạch và nhỏ, nên những truy vấn chậm trên bảng lớn vẫn đang nhanh một cách giả tạo.

Thứ tư, môi trường thử nghiệm chưa từng trải qua một lần mạng đứt, nên lựa chọn giữa nhất quán và sẵn sàng chưa bị thử thách. Kết luận đúng là việc chạy ổn ở đó chỉ chứng minh rằng không có lỗi hiển nhiên, chứ không chứng minh hệ thống chịu được tải và sự cố thật. Sau khi nói xong, chúng ta phải đưa ra một đề xuất chứ không dừng ở việc phản đối. Đề xuất của chúng ta gồm ba phần: chạy kiểm thử tải ở mức đỉnh dự kiến, triển khai dần cho một phần nhỏ người dùng, và bảo đảm có sẵn đường lùi cùng bảng theo dõi trước khi mở rộng. Cách trả lời này cho thấy chúng ta không cản trở tiến độ, chúng ta chỉ đang bảo vệ nó khỏi một sự cố lớn.

**English (bám cấu trúc tiếng Việt)**

There is a sentence we hear very often from the management side: the system has been running fine in staging for a week, let us just push it to production. We should not reject it outright, we should instead point out exactly what staging cannot reproduce. First, the load there is small, so hot keys, skewed partitions and pool exhaustion have had no chance to reveal themselves. Second, there is no real traffic there, so cold starts, cache stampedes and retry storms have never happened. Third, the data there is clean and small, so the queries that are slow on large tables are still artificially fast.

Fourth, the staging environment has never lived through a network partition, so the choice between consistency and availability has not been tested. The correct conclusion is that running fine there only proves that there are no obvious bugs, it does not prove that the system can survive real load and real incidents. After saying that, we have to put forward a proposal rather than stopping at the objection. Our proposal has three parts: run a load test at the expected peak, roll out gradually to a small portion of users, and make sure a rollback path and a dashboard are in place before we widen it. This way of answering shows that we are not blocking progress, we are only protecting it from a large incident.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| chúng ta không nên bác bỏ thẳng | we should not reject it outright |
| những gì môi trường thử nghiệm không tái hiện được | exactly what staging cannot reproduce |
| chưa có cơ hội lộ ra | have had no chance to reveal themselves |
| vẫn đang nhanh một cách giả tạo | are still artificially fast |
| chưa từng trải qua một lần mạng đứt | has never lived through a network partition |
| chưa bị thử thách | has not been tested |
| chỉ chứng minh rằng không có lỗi hiển nhiên | only proves that there are no obvious bugs |
| chúng ta phải đưa ra một đề xuất | we have to put forward a proposal |
| chúng ta không cản trở tiến độ | we are not blocking progress |
| bảo vệ nó khỏi một sự cố lớn | protecting it from a large incident |

**Thuật ngữ cần nhớ**

- môi trường thử nghiệm → **staging**
- môi trường thật → **production**
- kiểm thử tải → **a load test**
- triển khai thí điểm → **a canary release**
- sự cố → **an incident**

---

## Phần 8 — Mười lăm bài trước trở thành mười lăm câu hỏi kiểm tra

**Tiếng Việt**

Từ bây giờ, mỗi bài chúng ta đã học không còn là kiến thức để nhớ, mà trở thành một câu hỏi để ném vào thiết kế của người khác. Từ bài về quy trình, chúng ta hỏi phạm vi và giả định đã được làm rõ chưa, hay đội đã nhảy vào vẽ hộp ngay. Từ bài về ước lượng, chúng ta hỏi con số tải và dung lượng đã được tính chưa, hay mọi người đang nói chắc là ổn. Từ ba bài về linh kiện mở rộng, chúng ta hỏi điểm nghẽn nằm ở đâu và thành phần nào là điểm chết duy nhất. Từ bài về đánh đổi, chúng ta hỏi chỗ nào cần nhất quán mạnh và chỗ nào chấp nhận nhất quán cuối cùng.

Từ hai bài về ranh giới và tính toán, chúng ta hỏi dịch vụ được cắt theo nghiệp vụ hay theo tầng kỹ thuật. Từ bảy case study, chúng ta hỏi thiết kế đã có đủ những mẫu lõi chưa, ví dụ thao tác nguyên tử, tính bất biến khi lặp lại, phát tán và khử trùng lặp. Khi ghép tất cả lại, chúng ta có một bộ câu hỏi đủ để rà soát gần như mọi thiết kế trong ngành. Điều đáng nói là bộ câu hỏi này không đòi hỏi chúng ta là người viết ra hệ thống đó. Đây chính là ý nghĩa của câu chúng ta mang theo vào phòng phỏng vấn: chúng ta không nhớ để gõ, chúng ta hiểu để chỉ huy, và chúng ta tra cứu phần còn lại.

**English (bám cấu trúc tiếng Việt)**

From now on, each lesson we have learned is no longer knowledge to remember, it becomes a question to throw at somebody else's design. From the lesson on process, we ask whether the scope and the assumptions have been clarified, or whether the team jumped straight into drawing boxes. From the lesson on estimation, we ask whether the load and capacity numbers have been worked out, or whether everyone is saying it will probably be fine. From the three lessons on scaling primitives, we ask where the bottleneck sits and which component is the single point of failure. From the lesson on trade-offs, we ask where strong consistency is needed and where eventual consistency is acceptable.

From the two lessons on boundaries and compute, we ask whether the services are cut along business lines or along technical layers. From the seven case studies, we ask whether the design has all the core patterns, for example atomic operations, idempotency, fan-out and deduplication. When we put them all together, we have a set of questions sufficient to review almost any design in the industry. The notable thing is that this set of questions does not require us to be the person who wrote that system. This is exactly the meaning of the sentence we carry into the interview room: we do not memorise in order to type, we understand in order to lead, and we look up the rest.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| không còn là kiến thức để nhớ | is no longer knowledge to remember |
| một câu hỏi để ném vào thiết kế của người khác | a question to throw at somebody else's design |
| hay đội đã nhảy vào vẽ hộp ngay | or whether the team jumped straight into drawing boxes |
| mọi người đang nói chắc là ổn | everyone is saying it will probably be fine |
| thành phần nào là điểm chết duy nhất | which component is the single point of failure |
| cắt theo nghiệp vụ hay theo tầng kỹ thuật | cut along business lines or along technical layers |
| đủ để rà soát gần như mọi thiết kế trong ngành | sufficient to review almost any design in the industry |
| không đòi hỏi chúng ta là người viết ra hệ thống đó | does not require us to be the person who wrote that system |
| câu chúng ta mang theo vào phòng phỏng vấn | the sentence we carry into the interview room |

**Thuật ngữ cần nhớ**

- buổi rà soát thiết kế → **a design review**
- mẫu lõi → **a core pattern**
- danh sách kiểm → **a checklist**
- rà soát → **to review**
- điểm chết duy nhất → **a single point of failure**

---

## Mô hình ghi nhớ

**Tiếng Việt**

Người mới hỏi thiết kế này làm thế nào, còn kỹ sư dẫn dắt hỏi thiết kế này sai ở đâu, đắt ở đâu, vỡ khi nào và đo bằng gì. Chúng ta không nhớ để gõ, chúng ta hiểu để chỉ huy, và chúng ta tra cứu phần còn lại.

**English (bám cấu trúc tiếng Việt)**

A newcomer asks how this design works, while a leading engineer asks where this design is wrong, where it is expensive, when it will break and what we measure it with. We do not memorise in order to type, we understand in order to lead, and we look up the rest.

---

## Bảng thuật ngữ tổng hợp

| Tiếng Việt | Tiếng Anh | Ghi chú phát âm |
|---|---|---|
| phản biện, chất vấn | to challenge | "**CHA**-lịnj", trọng âm đầu |
| phán đoán | judgement | "**JĂJ**-mần", trọng âm đầu |
| lựa chọn kiến trúc | an architectural choice | *architectural* — a-ki-**TEK**-chơ-rợl, chữ *ch* đầu đọc /k/ |
| năng lực | an ability / a capability | |
| đề xuất | a proposal | prơ-**PÂU**-zợl, trọng âm âm thứ hai |
| bản chất bài toán | the nature of the problem | *nature* — "**NÂY**-chơ" |
| lệch, không khớp | a mismatch | |
| phương án cơ sở | a baseline | |
| chỉ số đo | a metric | |
| tinh chỉnh mô hình | fine-tuning | |
| truy hồi rồi sinh | retrieval-augmented generation (RAG) | *retrieval* — ri-**TRII**-vợl, trọng âm âm thứ hai |
| trọng số mô hình | the model weights | *weights* — chữ *gh* câm, âm cuối **-ts** phải bật |
| bịa thông tin | to hallucinate | hơ-**LU**-sị-nêit, trọng âm âm thứ hai |
| trích nguồn | to cite a source | *cite* /saɪt/ — đọc như *site* |
| chỉ mục | an index | |
| điểm tử huyệt | a fatal weak point | *fatal* — "**FÂY**-tợl" |
| giả định ngầm | a silent assumption | *assumption* — ơ-**SĂMP**-shợn |
| lệch đồng hồ | clock skew | *skew* /skjuː/ — "skiu" |
| đường lùi | a rollback path | *path* có âm **th** cuối |
| dấu hiệu sớm | an early sign | *sign* — chữ **g câm**, đọc "sain" |
| khoá nóng | a hot key | |
| khởi động nguội | a cold start | |
| đổ xô vào cache | a cache stampede | *stampede* — stam-**PIID**, trọng âm âm cuối |
| cạn pool kết nối | connection pool exhaustion | *exhaustion* — ịg-**ZOS**-chợn, chữ *h* không đọc rõ |
| phân mảnh mạng | a network partition | |
| cơn bão thử lại | a retry storm | |
| chi phí dữ liệu đi ra | egress cost | "**II**-gres", trọng âm đầu |
| vùng sẵn sàng | an availability zone | |
| tải dự kiến | the expected load | |
| tính kinh tế theo quy mô | economies of scale | *economies* — i-**CO**-nơ-miz, trọng âm âm thứ hai |
| ngân sách | a budget | "**BĂ**-jịt", trọng âm đầu |
| khả năng quan sát | observability | ợb-zơ-vơ-**BI**-li-ti, trọng âm âm thứ tư |
| truy vết | a trace | |
| di trú dữ liệu | a data migration | *migration* — mai-**GRÂY**-shợn |
| tương thích ngược | backwards-compatible | *compatible* — cợm-**PA**-tơ-bợl, trọng âm âm thứ hai |
| triển khai dần | a gradual rollout | *gradual* — "**GRA**-ju-ợl" |
| môi trường thử nghiệm | staging | |
| môi trường thật | production | |
| kiểm thử tải | a load test | |
| triển khai thí điểm | a canary release | *canary* — cơ-**NE**-ri, trọng âm âm thứ hai |
| sự cố | an incident | "**IN**-sị-đợnt", trọng âm đầu |
| buổi rà soát thiết kế | a design review | |
| mẫu lõi | a core pattern | |
| danh sách kiểm | a checklist | |
| điểm chết duy nhất | a single point of failure | |
| kỹ thuật gây lỗi có chủ đích | chaos engineering | *chaos* /ˈkeɪɒs/ — "KÂY-os", chữ *ch* đọc /k/ |
| báo cáo sau sự cố | a postmortem | pâust-**MO**-tợm, trọng âm âm thứ hai |

---

## Bài nói

> **Cách luyện:** bật ghi âm, nói **60–90 giây** cho mỗi đề, **không nhìn tài liệu** khi nói. Mỗi bài phải dùng ít nhất **sáu thuật ngữ** trong bảng trên. Đây là bài cuối, nên hãy dùng nó như một buổi tổng duyệt: các đề dưới đây đều là dạng câu hỏi thật ở vòng phỏng vấn cho vị trí dẫn dắt.

1. **A colleague says:** *"Let's fine-tune a model on our internal documentation so the assistant can answer support questions."* **Explain what is wrong with that**, and describe what you would build instead and when fine-tuning would actually be the right call.

2. **A manager says:** *"It has run fine in staging for a week, let's ship it to production on Monday."* **Explain why you would push back**, and say what you would ask for before agreeing.

3. **Explain to a junior engineer** the five questions you ask whenever someone proposes a new technology, and why the question about a simpler alternative matters most.

4. **Describe what happens when** a design that assumes an even load meets real traffic where one percent of the keys carry half the requests. Name the metric that would reveal it first.

5. **Explain to a product manager** why cost has to be a design criterion rather than something finance discovers two months later.

6. **When would you choose** to block a release, and when would you let it go out with a known risk? Describe how you would frame that decision to the team.

7. **Someone hands you a microservices diagram** you have never seen before and gives you three minutes. **Describe the first three questions you would ask** and say what each answer would tell you.
