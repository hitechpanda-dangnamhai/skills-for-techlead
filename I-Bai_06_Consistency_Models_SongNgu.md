# Bài 6 — Consistency Models & Eventual Consistency
### Bản song ngữ: tiếng Anh bám cấu trúc tiếng Việt

> **Cách dùng bài này:** đọc đoạn tiếng Việt trước → tự dịch trong đầu, không nhìn xuống → so với đoạn tiếng Anh bên dưới → **đọc to bản tiếng Anh hai lần**: lần một đọc chậm cho đúng từng âm, lần hai đọc trôi như đang nói với người phỏng vấn.

---

## ① Nhất quán là một phổ, không phải một công tắc

**Tiếng Việt**

Tính nhất quán không phải là một công tắc bật hoặc tắt, mà nó là một phổ gồm nhiều mức. Xếp từ mạnh xuống yếu, chúng ta có linearizable, rồi sequential, rồi causal, và cuối cùng là eventual. Mức càng mạnh thì các node càng phải phối hợp với nhau nhiều hơn trước khi trả lời, cho nên mức càng mạnh thì càng chậm và càng đắt. Nguyên tắc thiết kế của chúng ta là chọn mức yếu nhất mà nghiệp vụ vẫn đúng, chứ không phải chọn mức mạnh nhất mà hệ chịu được. Nếu chúng ta chọn quá mạnh, chúng ta trả tiền bằng độ trễ và bằng khả năng phục vụ mà chúng ta không nhận thêm giá trị nghiệp vụ nào. Nếu chúng ta chọn quá yếu, người dùng sẽ nhìn thấy dữ liệu sai ở đúng những chỗ mà họ không được phép nhìn thấy dữ liệu sai.

**English (bám cấu trúc tiếng Việt)**

Consistency is not a switch that is on or off, but it is a spectrum made of many levels. Ordered from strong down to weak, we have linearizable, then sequential, then causal, and finally eventual. The stronger the level is, the more the nodes have to coordinate with each other before they answer, therefore the stronger the level is, the slower and the more expensive it becomes. Our design principle is to choose the weakest level at which the business is still correct, and not to choose the strongest level that the system can bear. If we choose too strong, we pay with latency and with the ability to serve while we gain no extra business value. If we choose too weak, the users will see wrong data in exactly the places where they are not allowed to see wrong data.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| một công tắc bật hoặc tắt | a switch that is on or off |
| một phổ gồm nhiều mức | a spectrum made of many levels |
| xếp từ mạnh xuống yếu | ordered from strong down to weak |
| mức càng mạnh thì … càng | the stronger the level is, the more … |
| phối hợp với nhau | coordinate with each other |
| chọn mức yếu nhất mà nghiệp vụ vẫn đúng | choose the weakest level at which the business is still correct |
| mức mạnh nhất mà hệ chịu được | the strongest level that the system can bear |
| chúng ta trả tiền bằng độ trễ | we pay with latency |
| không nhận thêm giá trị nghiệp vụ nào | we gain no extra business value |
| ở đúng những chỗ mà họ không được phép | in exactly the places where they are not allowed |

**Thuật ngữ cần nhớ**

- phổ nhất quán → **the consistency spectrum**
- sự phối hợp giữa các node → **coordination**
- mức nhất quán → **a consistency level**
- yếu nhất mà vẫn đúng → **the weakest level that is still correct**
- giá trị nghiệp vụ → **business value**

---

## ② Linearizability và cái giá của nó

**Tiếng Việt**

Linearizability là mức mạnh nhất trong phổ, và định nghĩa của nó khá gọn. Mỗi thao tác được coi như xảy ra tức thời tại một điểm nào đó nằm giữa lúc client gọi và lúc client nhận được kết quả. Hệ tôn trọng thứ tự thời gian thực, cho nên nếu một lệnh ghi kết thúc trước khi một lệnh đọc bắt đầu, lệnh đọc đó bắt buộc phải nhìn thấy lệnh ghi kia. Nói ngắn gọn, hệ hành xử như thể chỉ tồn tại một bản sao dữ liệu duy nhất, dù bên dưới có năm replica. Cái giá phải trả là mỗi thao tác cần một vòng đi tới leader hoặc tới đa số replica, cho nên linearizability luôn luôn đắt về độ trễ.

**English (bám cấu trúc tiếng Việt)**

Linearizability is the strongest level in the spectrum, and its definition is fairly compact. Every operation is treated as happening instantly at some point that lies between the moment the client calls and the moment the client receives the result. The system respects real-time order, therefore if a write finishes before a read begins, that read is obliged to see the other write. Put briefly, the system behaves as though only one single copy of the data exists, although underneath there are five replicas. The price to pay is that every operation needs a round trip to the leader or to the majority of replicas, therefore linearizability is always expensive in terms of latency.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| định nghĩa của nó khá gọn | its definition is fairly compact |
| được coi như xảy ra tức thời | is treated as happening instantly |
| tại một điểm nào đó nằm giữa | at some point that lies between |
| tôn trọng thứ tự thời gian thực | respects real-time order |
| lệnh đọc đó bắt buộc phải nhìn thấy | that read is obliged to see |
| nói ngắn gọn | put briefly |
| như thể chỉ tồn tại một bản sao dữ liệu duy nhất | as though only one single copy of the data exists |
| dù bên dưới có năm replica | although underneath there are five replicas |
| cái giá phải trả là | the price to pay is that |
| đắt về độ trễ | expensive in terms of latency |

**Thuật ngữ cần nhớ**

- thứ tự thời gian thực → **real-time order**
- độ mới của dữ liệu → **recency**
- một vòng đi và về → **a round trip**
- bản sao dữ liệu → **a copy of the data**
- cái giá phải trả → **the price to pay**

---

## ③ Linearizability khác serializability ở trục nào

**Tiếng Việt**

Cái bẫy hay gặp nhất của chủ đề này là chúng ta lẫn linearizability với serializability, bởi vì hai từ nhìn na ná nhau. Linearizability là thuộc tính về một đối tượng đơn lẻ và về thứ tự thời gian thực, tức là nó nói về độ mới của dữ liệu. Serializability là thuộc tính về nhiều đối tượng và về transaction, tức là nó nói rằng kết quả cuối cùng phải tương đương với một thứ tự tuần tự nào đó của các transaction. Điểm mấu chốt là serializability không bắt buộc thứ tự tuần tự đó phải khớp với thời gian thực, cho nên một hệ serializable vẫn có thể cho chúng ta đọc ra dữ liệu cũ. Khi một hệ thỏa mãn cả hai điều kiện, chúng ta gọi nó là strict serializable, và đó là mức bảo đảm mạnh nhất mà một database phân tán có thể cung cấp. Nếu chúng ta nói rõ được hai trục này trong phỏng vấn, đó thường là câu trả lời tách chúng ta ra khỏi phần lớn ứng viên.

**English (bám cấu trúc tiếng Việt)**

The most common trap of this topic is that we mix up linearizability with serializability, because the two words look rather alike. Linearizability is a property about a single object and about real-time order, that is, it talks about the recency of the data. Serializability is a property about many objects and about transactions, that is, it says that the final result must be equivalent to some sequential order of the transactions. The key point is that serializability does not require that sequential order to match real time, therefore a serializable system can still let us read old data. When a system satisfies both conditions, we call it strict serializable, and that is the strongest guarantee that a distributed database can offer. If we can state these two axes clearly in an interview, that is usually the answer that separates us from most candidates.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| hai từ nhìn na ná nhau | the two words look rather alike |
| thuộc tính về một đối tượng đơn lẻ | a property about a single object |
| nó nói về độ mới của dữ liệu | it talks about the recency of the data |
| phải tương đương với một thứ tự tuần tự nào đó | must be equivalent to some sequential order |
| điểm mấu chốt là | the key point is that |
| không bắt buộc … phải khớp với thời gian thực | does not require … to match real time |
| vẫn có thể cho chúng ta đọc ra dữ liệu cũ | can still let us read old data |
| khi một hệ thỏa mãn cả hai điều kiện | when a system satisfies both conditions |
| mức bảo đảm mạnh nhất | the strongest guarantee |
| nói rõ được hai trục này | state these two axes clearly |

**Thuật ngữ cần nhớ**

- tính khả tuần tự → **serializability**
- tuần tự nghiêm ngặt → **strict serializable**
- thuộc tính đơn đối tượng → **a single-object property**
- tương đương → **equivalent**
- mức bảo đảm → **a guarantee**

---

## ④ Eventual consistency không hứa một mốc thời gian nào

**Tiếng Việt**

Eventual consistency có một định nghĩa rất dễ bị nhớ sai. Định nghĩa đúng là nếu chúng ta ngừng ghi, các replica cuối cùng sẽ hội tụ về cùng một giá trị. Định nghĩa này không hứa bất kỳ mốc thời gian cụ thể nào, cho nên câu nói "eventual nghĩa là đồng bộ trong vòng hai giây" là sai về bản chất. Trong thực tế, khoảng lệch thường nằm trong tầm vài mili-giây đến vài giây, và nó bị chặn bởi độ trễ nhân bản cùng với tình trạng của mạng. Vì vậy khi trả lời phỏng vấn, chúng ta phát biểu điều kiện hội tụ trước, rồi chúng ta mới nói con số quan sát được như một dữ kiện vận hành.

**English (bám cấu trúc tiếng Việt)**

Eventual consistency has a definition that is very easy to remember wrongly. The correct definition is that if we stop writing, the replicas will eventually converge on the same value. This definition does not promise any specific point in time, therefore the sentence "eventual means synchronised within two seconds" is wrong in its very nature. In practice, the gap usually lies in the range of a few milliseconds to a few seconds, and it is bounded by the replication lag together with the state of the network. Therefore when we answer in an interview, we state the convergence condition first, and only then we mention the observed number as an operational fact.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| rất dễ bị nhớ sai | very easy to remember wrongly |
| nếu chúng ta ngừng ghi | if we stop writing |
| cuối cùng sẽ hội tụ về cùng một giá trị | will eventually converge on the same value |
| không hứa bất kỳ mốc thời gian cụ thể nào | does not promise any specific point in time |
| là sai về bản chất | is wrong in its very nature |
| nằm trong tầm vài mili-giây đến vài giây | lies in the range of a few milliseconds to a few seconds |
| bị chặn bởi độ trễ nhân bản | is bounded by the replication lag |
| chúng ta phát biểu điều kiện hội tụ trước | we state the convergence condition first |
| con số quan sát được | the observed number |
| như một dữ kiện vận hành | as an operational fact |

**Thuật ngữ cần nhớ**

- nhất quán cuối cùng → **eventual consistency**
- hội tụ → **to converge**
- độ trễ nhân bản → **replication lag**
- bị chặn bởi → **to be bounded by**
- dữ kiện vận hành → **an operational fact**

---

## ⑤ Ba bảo đảm theo phiên làm việc

**Tiếng Việt**

Giữa hai đầu của phổ có một nhóm bảo đảm rất thực dụng, gọi là các bảo đảm theo phiên làm việc. Bảo đảm đầu tiên là read-your-own-writes, nghĩa là một người dùng luôn đọc thấy chính những gì họ vừa ghi. Bảo đảm này bị vi phạm khi lệnh đọc được định tuyến sang một replica chưa kịp cập nhật, và người dùng sẽ kêu rằng họ vừa sửa xong mà không thấy gì thay đổi. Cách chữa là chúng ta định tuyến lệnh đọc của chính người đó về primary trong một cửa sổ ngắn sau khi ghi, hoặc chúng ta gửi kèm một version token để client chờ replica bắt kịp. Điều đáng nhớ là hiện tượng này thường không phải là lỗi ghi, mà nó là lỗi định tuyến đọc.

Bảo đảm thứ hai là monotonic reads, nghĩa là lần đọc sau không được nhìn thấy trạng thái cũ hơn lần đọc trước. Khi thiếu bảo đảm này, người dùng sẽ thấy thời gian chạy ngược, tức là họ tải lại trang và dữ liệu vừa xuất hiện bỗng biến mất, bởi vì hai lần đọc rơi vào hai replica lệch nhau. Cách chữa là chúng ta ghim phiên làm việc vào một replica cố định, hoặc chúng ta so version trước khi hiển thị. Bảo đảm thứ ba là causal consistency, nghĩa là hệ giữ đúng thứ tự nhân quả, cho nên một câu trả lời không bao giờ xuất hiện trước tin nhắn gốc. Mức này yếu hơn linearizable nhưng nó đủ dùng cho chat và cho bình luận, còn eventual thuần túy thì có thể đảo ngược thứ tự của những sự kiện liên quan nhau.

**English (bám cấu trúc tiếng Việt)**

Between the two ends of the spectrum there is a group of very practical guarantees, called the session guarantees. The first guarantee is read-your-own-writes, which means that a user always reads exactly what they have just written. This guarantee is violated when the read is routed to a replica that has not caught up yet, and the user will complain that they have just finished editing but see nothing changed. The cure is that we route the reads of that particular person to the primary within a short window after the write, or we send along a version token so that the client waits for the replica to catch up. The thing worth remembering is that this phenomenon is usually not a write bug, but it is a read routing bug.

The second guarantee is monotonic reads, which means that a later read must not see a state older than an earlier read. When this guarantee is missing, the user will see time running backwards, that is, they reload the page and the data that has just appeared suddenly disappears, because the two reads land on two replicas that are out of step. The cure is that we pin the session to a fixed replica, or we compare versions before displaying. The third guarantee is causal consistency, which means that the system keeps the cause-and-effect order right, therefore a reply never appears before the original message. This level is weaker than linearizable but it is enough for chat and for comments, while pure eventual can reverse the order of events that are related to each other.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| các bảo đảm theo phiên làm việc | the session guarantees |
| bảo đảm này bị vi phạm khi | this guarantee is violated when |
| một replica chưa kịp cập nhật | a replica that has not caught up yet |
| họ vừa sửa xong mà không thấy gì thay đổi | they have just finished editing but see nothing changed |
| định tuyến lệnh đọc của chính người đó về primary | route the reads of that particular person to the primary |
| để client chờ replica bắt kịp | so that the client waits for the replica to catch up |
| nó là lỗi định tuyến đọc | it is a read routing bug |
| người dùng sẽ thấy thời gian chạy ngược | the user will see time running backwards |
| hai replica lệch nhau | two replicas that are out of step |
| ghim phiên làm việc vào một replica cố định | pin the session to a fixed replica |
| giữ đúng thứ tự nhân quả | keeps the cause-and-effect order right |
| có thể đảo ngược thứ tự của những sự kiện liên quan nhau | can reverse the order of events that are related to each other |

**Thuật ngữ cần nhớ**

- bảo đảm theo phiên → **a session guarantee**
- đọc thấy ghi của chính mình → **read-your-own-writes**
- đọc đơn điệu → **monotonic reads**
- nhất quán nhân quả → **causal consistency**
- định tuyến đọc → **read routing**
- ghim phiên → **session stickiness**

---

## ⑥ Công thức quorum R cộng W lớn hơn N

**Tiếng Việt**

Công thức quorum là thứ chúng ta phải thuộc, và nó chỉ gồm đúng một bất đẳng thức. Gọi N là tổng số replica, W là số node phải xác nhận cho một lệnh ghi, và R là số node phải trả lời cho một lệnh đọc. Nếu R cộng W lớn hơn N, tập node đọc và tập node ghi bắt buộc phải giao nhau ít nhất một node, cho nên lệnh đọc chắc chắn chạm được một node đã nhận bản ghi mới nhất. Với N bằng ba, cấu hình W bằng hai và R bằng hai là mức cân bằng, còn W bằng ba và R bằng một thì đọc rất nhanh nhưng ghi chậm. Ngược lại, W bằng một và R bằng ba thì ghi rất nhanh nhưng đọc chậm, còn W bằng một và R bằng một thì tổng chỉ bằng hai và nhỏ hơn ba, cho nên chúng ta không có bảo đảm gì và chúng ta có thể đọc phải dữ liệu cũ. Điều quan trọng cần nói thêm là quorum cho chúng ta chọn chi phí nằm ở phía đọc hay ở phía ghi, chứ nó không làm biến mất chi phí đó.

**English (bám cấu trúc tiếng Việt)**

The quorum formula is something we have to know by heart, and it consists of exactly one inequality. Let N be the total number of replicas, W be the number of nodes that must acknowledge a write, and R be the number of nodes that must answer a read. If R plus W is greater than N, the set of read nodes and the set of write nodes are obliged to overlap on at least one node, therefore the read is certain to touch a node that has received the newest write. With N equal to three, the configuration W equal to two and R equal to two is the balanced level, while W equal to three and R equal to one makes reads very fast but writes slow. On the other hand, W equal to one and R equal to three makes writes very fast but reads slow, while W equal to one and R equal to one gives a total of only two which is smaller than three, therefore we have no guarantee and we may read stale data. The important thing to add is that a quorum lets us choose whether the cost sits on the read side or on the write side, and it does not make that cost disappear.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| nó chỉ gồm đúng một bất đẳng thức | it consists of exactly one inequality |
| gọi N là tổng số replica | let N be the total number of replicas |
| số node phải xác nhận cho một lệnh ghi | the number of nodes that must acknowledge a write |
| bắt buộc phải giao nhau ít nhất một node | are obliged to overlap on at least one node |
| chắc chắn chạm được một node đã nhận bản ghi mới nhất | is certain to touch a node that has received the newest write |
| là mức cân bằng | is the balanced level |
| tổng chỉ bằng hai và nhỏ hơn ba | gives a total of only two which is smaller than three |
| chúng ta không có bảo đảm gì | we have no guarantee |
| chi phí nằm ở phía đọc hay ở phía ghi | whether the cost sits on the read side or on the write side |
| nó không làm biến mất chi phí đó | it does not make that cost disappear |

**Thuật ngữ cần nhớ**

- số phiếu tối thiểu → **a quorum**
- bất đẳng thức → **an inequality**
- giao nhau → **to overlap** / **to intersect**
- xác nhận một lệnh ghi → **to acknowledge a write**
- đọc phải dữ liệu cũ → **a stale read**

---

## ⑦ Gắn mức vào nghiệp vụ, và phản biện lối nghĩ "cứ strong cho chắc"

**Tiếng Việt**

Việc gắn mức nhất quán vào nghiệp vụ phải dựa trên hậu quả của một lần sai, chứ nó không dựa trên cảm giác an toàn. Số dư ví và các thao tác tiền cần mức mạnh, bởi vì một con số sai chính là một khoản tiền sai. Bộ đếm lượt thích có thể để ở mức eventual, bởi vì lệch vài giây thì không ai bị thiệt hại. Trạng thái đơn hàng thường cần mức mạnh cho những trạng thái mang tính quyết định, còn các trường hiển thị phụ thì để yếu hơn cũng được. Khi database cho phép chỉnh mức theo từng truy vấn, lợi ích là chúng ta trả đúng chi phí cho đúng nhu cầu, nhưng rủi ro là một lập trình viên đặt nhầm mức ở một luồng quan trọng và sinh ra loại bug rất khó tái hiện.

Vì vậy chúng ta phải phản biện lại lối nghĩ "cứ chọn strong cho chắc". Nhất quán mạnh làm tăng độ trễ, làm giảm khả năng phục vụ khi có partition, và làm giảm thông lượng của cả hệ. Rất nhiều đường đi trong ứng dụng không cần đến mức đó, cho nên đặt strong làm mặc định toàn cục là một quyết định đắt mà ít người kiểm chứng lại. Cuối cùng, chúng ta nên nhớ rằng độ trễ nhân bản là nguyên nhân chung của cả một lớp lỗi, gồm đọc phải dữ liệu cũ, mất read-your-writes, và đọc không đơn điệu. Cách xử lý là chúng ta đọc từ primary cho các luồng nhạy cảm, ghim phiên khi cần, dùng version token để chờ, và giám sát độ trễ nhân bản để biết cửa sổ dữ liệu cũ thật sự là bao nhiêu.

**English (bám cấu trúc tiếng Việt)**

Attaching a consistency level to the business must rest on the consequence of being wrong once, and it does not rest on a feeling of safety. The wallet balance and money operations need a strong level, because a wrong number is exactly a wrong amount of money. A like counter can be left at the eventual level, because being a few seconds out harms nobody. The order status usually needs a strong level for the statuses that are decisive, while the secondary display fields can be left weaker. When the database allows the level to be tuned per query, the benefit is that we pay the right cost for the right need, but the risk is that a developer sets the wrong level on an important flow and produces the kind of bug that is very hard to reproduce.

Therefore we have to argue back against the mindset of "just choose strong to be safe". Strong consistency raises latency, reduces the ability to serve when there is a partition, and lowers the throughput of the whole system. A great many paths in the application do not need that level, therefore making strong the global default is an expensive decision that few people check again. Finally, we should remember that replication lag is the common cause of a whole class of faults, including stale reads, the loss of read-your-writes, and non-monotonic reads. The way to handle it is that we read from the primary for the sensitive flows, pin the session when needed, use a version token to wait, and monitor the replication lag in order to know what the staleness window really is.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| dựa trên hậu quả của một lần sai | rest on the consequence of being wrong once |
| nó không dựa trên cảm giác an toàn | it does not rest on a feeling of safety |
| một con số sai chính là một khoản tiền sai | a wrong number is exactly a wrong amount of money |
| lệch vài giây thì không ai bị thiệt hại | being a few seconds out harms nobody |
| những trạng thái mang tính quyết định | the statuses that are decisive |
| các trường hiển thị phụ | the secondary display fields |
| chúng ta trả đúng chi phí cho đúng nhu cầu | we pay the right cost for the right need |
| phản biện lại lối nghĩ | argue back against the mindset of |
| một quyết định đắt mà ít người kiểm chứng lại | an expensive decision that few people check again |
| nguyên nhân chung của cả một lớp lỗi | the common cause of a whole class of faults |
| để biết cửa sổ dữ liệu cũ thật sự là bao nhiêu | in order to know what the staleness window really is |

**Thuật ngữ cần nhớ**

- hậu quả → **the consequence**
- mang tính quyết định → **decisive**
- chỉnh theo từng truy vấn → **to tune per query**
- mặc định toàn cục → **the global default**
- cửa sổ dữ liệu cũ → **the staleness window**

---

## Mô hình ghi nhớ

**Tiếng Việt**

Nhất quán là một phổ chứ không phải một công tắc, và mức càng mạnh thì càng cần phối hợp nhiều hơn nên càng đắt, cho nên chúng ta chọn mức yếu nhất mà nghiệp vụ vẫn đúng. Linearizability nói về độ mới của một đối tượng, còn serializability nói về thứ tự của nhiều transaction, và chúng ta đừng lẫn hai trục đó với nhau.

**English (bám cấu trúc tiếng Việt)**

Consistency is a spectrum and not a switch, and the stronger the level is, the more coordination it needs and therefore the more expensive it becomes, so we choose the weakest level at which the business is still correct. Linearizability talks about the recency of one object, while serializability talks about the order of many transactions, and we must not mix those two axes up with each other.

---

## Bảng thuật ngữ tổng hợp

| Tiếng Việt | Tiếng Anh | Ghi chú phát âm (Anh–Anh) |
|---|---|---|
| phổ nhất quán | **the consistency spectrum** | *SPEK-trợm* — trọng âm đầu, cụm **sp-** và **-ktr-** phải rõ |
| sự phối hợp giữa các node | **coordination** | *kou-o-di-NÊI-shợn* — trọng âm âm thứ tư, năm âm tiết |
| mức nhất quán | **a consistency level** | *kơn-SIS-tợn-si* — trọng âm âm thứ hai |
| tuần tự | **sequential** | *si-KWEN-shợl* — trọng âm giữa, có âm /kw/ |
| nhân quả | **causal** | */ˈkɔːzl/* "KOO-zợl" — chữ `s` đọc /z/, không đọc "cau-sal" |
| nhất quán cuối cùng | **eventual consistency** | *i-VEN-tsu-ợl* — trọng âm âm thứ hai |
| tính tuyến tính hóa | **linearizability** | *li-ni-ơ-rai-zơ-BI-lơ-ti* — bảy âm tiết; tập tách: *linear + ize + ability* |
| tính khả tuần tự | **serializability** | *si-ri-ơ-lai-zơ-BI-lơ-ti* — tập tách: *serial + ize + ability* |
| tuần tự nghiêm ngặt | **strict serializable** | *strict* — cụm đầu **str** và cụm cuối **-ct** đều phải bật ra |
| thứ tự thời gian thực | **real-time order** | |
| độ mới của dữ liệu | **recency** | *RII-sợn-si* — trọng âm đầu |
| một vòng đi và về | **a round trip** | |
| tương đương | **equivalent** | *i-KWI-vơ-lợnt* — trọng âm âm thứ hai, có âm /kw/ |
| mức bảo đảm | **a guarantee** | *ga-rơn-TII* — trọng âm rơi vào âm **cuối** |
| trục | **an axis** (số nhiều **axes**) | số ít *AK-sis*; số nhiều *AK-siiz* — hai cách đọc khác nhau |
| hội tụ | **to converge** | *kơn-VƠƠDJ* — trọng âm sau, đuôi /dʒ/ |
| độ trễ nhân bản | **replication lag** | *replication* = *re-pli-KÊI-shợn*, trọng âm áp chót |
| bị chặn bởi | **to be bounded by** | |
| dữ kiện vận hành | **an operational fact** | *o-pơ-RÊI-shơ-nợl* — trọng âm âm thứ ba |
| bảo đảm theo phiên | **a session guarantee** | *SE-shợn* — trọng âm đầu |
| đọc thấy ghi của chính mình | **read-your-own-writes** | *write* câm chữ **w**, đọc như "right" |
| đọc đơn điệu | **monotonic reads** | *mo-nơ-TON-ik* — trọng âm âm thứ ba |
| nhất quán nhân quả | **causal consistency** | |
| định tuyến đọc | **read routing** | Anh: *route* /ruːt/ "ruut", **không** đọc /raʊt/ kiểu Mỹ |
| ghim phiên | **session stickiness** | |
| bắt kịp | **to catch up** | *catch* /kætʃ/ — đuôi /tʃ/ |
| lệch nhau | **out of step** | |
| số phiếu tối thiểu | **a quorum** | *KWO-rợm* — có âm /kw/ đầu |
| bất đẳng thức | **an inequality** | *i-ni-KWO-lơ-ti* — trọng âm âm thứ ba, có âm /kw/ |
| giao nhau | **to overlap** / **to intersect** | *in-tơ-SEKT* — trọng âm cuối, cụm **-kt** phải bật |
| xác nhận một lệnh ghi | **to acknowledge a write** | *ơk-NO-lidj* — trọng âm âm thứ hai, đuôi /dʒ/ |
| đọc phải dữ liệu cũ | **a stale read** | *stale* /steɪl/ |
| hậu quả | **the consequence** | *KON-si-kwợns* — trọng âm đầu, cụm cuối **-ns** |
| mang tính quyết định | **decisive** | *di-SAI-siv* — trọng âm giữa, chữ `s` giữa đọc /s/ |
| chỉnh theo từng truy vấn | **to tune per query** | *query* Anh = *KWIA-ri* /ˈkwɪəri/ |
| mặc định toàn cục | **the global default** | *di-FOLT* — trọng âm sau khi là danh từ kỹ thuật |
| cửa sổ dữ liệu cũ | **the staleness window** | |
| thông lượng | **throughput** | *THRUU-put* — âm **th** và cụm **thr** |
| tái hiện | **to reproduce** | *rep-rơ-DYUUS* — kiểu Anh có âm /dj/ |
| lối nghĩ | **a mindset** | |
| giám sát | **to monitor** | *MO-ni-tơ* — trọng âm đầu |

---

## Bài nói

> **Cách luyện:** bật ghi âm, nói **60–90 giây** cho mỗi đề, **không nhìn tài liệu**. Mỗi bài nói phải dùng ít nhất **sáu thuật ngữ** trong bảng trên. Nói xong nghe lại một lần, đánh dấu chỗ nào bị vấp hoặc phát âm sai, rồi nói lại đề đó một lần nữa.

1. **Explain to a junior developer** the consistency spectrum from strong to weak, and why each step towards the strong end costs more.

2. **Describe what happens**, step by step, when a user posts a comment, the write goes to the primary, and the reload is served by a lagging replica. Name the guarantee that was broken and two ways to restore it.

3. **A colleague says:** *"Our database is serializable, so every read is guaranteed to see the latest write."* Explain what is wrong with that claim, and define both properties precisely.

4. **Explain the quorum condition R plus W greater than N** to someone who has never seen it, using N equal to five. Give one configuration tuned for fast reads and one tuned for fast writes.

5. **Someone on your team proposes setting strong consistency as the global default** "so we never have to think about it again". Explain why you would push back, and what you would ask them to look at instead.

6. **When is causal consistency enough** and when do you need linearizability? Give one product feature for each, and say what the user would notice if you got it wrong.

7. **A user reports that data appears and then disappears when they refresh the page.** Explain to the team what is most likely happening, why it is not a write bug, and how you would fix it.

---

> 🔚 **Hết Bài 6.** Gõ `Làm Bài 7` để sang *Consensus (Raft / Paxos / Quorum / Leader Election)* — bài cuối của mục I.
