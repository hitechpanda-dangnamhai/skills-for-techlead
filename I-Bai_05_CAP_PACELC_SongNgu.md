# Bài 5 — CAP & PACELC
### Bản song ngữ: tiếng Anh bám cấu trúc tiếng Việt

> **Cách dùng bài này:** đọc đoạn tiếng Việt trước → tự dịch trong đầu, không nhìn xuống → so với đoạn tiếng Anh bên dưới → **đọc to bản tiếng Anh hai lần**: lần một đọc chậm cho đúng từng âm, lần hai đọc trôi như đang nói với người phỏng vấn.

---

## ① Phát biểu CAP cho đúng và gỡ hiểu lầm "chọn hai trong ba"

**Tiếng Việt**

CAP phát biểu rằng khi mạng bị chia cắt, một hệ phân tán không thể vừa nhất quán vừa sẵn sàng, cho nên chúng ta phải chọn một trong hai. Nhiều người nhớ định lý này dưới dạng "chọn hai trong ba", và cách nhớ đó gây ra một hiểu lầm nghiêm trọng. Lý do là trong một hệ phân tán thật, chữ P không phải là một lựa chọn mà là một điều kiện bắt buộc, bởi vì mạng chắc chắn sẽ chia cắt sớm hay muộn. Chúng ta không thể tuyên bố rằng mình bỏ P để lấy cả C lẫn A, bởi vì bỏ P nghĩa là giả vờ rằng mạng không bao giờ hỏng. Cho nên cách phát biểu đúng là khi partition xảy ra, chúng ta chọn giữ tính nhất quán hay giữ khả năng phục vụ. Khi mạng lành, hệ hoàn toàn có thể vừa nhất quán vừa sẵn sàng, và điều đó không hề mâu thuẫn với định lý.

**English (bám cấu trúc tiếng Việt)**

CAP states that when the network is partitioned, a distributed system cannot be both consistent and available, therefore we have to choose one of the two. Many people remember this theorem in the form "pick two out of three", and that way of remembering causes a serious misunderstanding. The reason is that in a real distributed system, the letter P is not a choice but a compulsory condition, because the network will certainly split sooner or later. We cannot declare that we drop P in order to take both C and A, because dropping P means pretending that the network never fails. Therefore the correct way to state it is that when a partition happens, we choose to keep consistency or to keep the ability to serve. When the network is healthy, the system can perfectly well be both consistent and available, and that does not contradict the theorem at all.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| khi mạng bị chia cắt | when the network is partitioned |
| nhớ định lý này dưới dạng | remember this theorem in the form |
| gây ra một hiểu lầm nghiêm trọng | causes a serious misunderstanding |
| không phải là một lựa chọn mà là một điều kiện bắt buộc | is not a choice but a compulsory condition |
| sớm hay muộn | sooner or later |
| chúng ta không thể tuyên bố rằng | we cannot declare that |
| giả vờ rằng mạng không bao giờ hỏng | pretending that the network never fails |
| giữ khả năng phục vụ | keep the ability to serve |
| khi mạng lành | when the network is healthy |
| không hề mâu thuẫn với định lý | does not contradict the theorem at all |

**Thuật ngữ cần nhớ**

- chia cắt mạng → **a network partition**
- khả năng sẵn sàng phục vụ → **availability**
- tính nhất quán → **consistency**
- điều kiện bắt buộc → **a compulsory condition**
- định lý → **a theorem**

---

## ② Chữ C trong CAP không phải chữ C trong ACID

**Tiếng Việt**

Chữ C trong CAP có một nghĩa rất hẹp, đó là linearizability. Linearizability nghĩa là mọi lệnh đọc đều nhìn thấy lệnh ghi mới nhất, tức là hệ hành xử như thể chỉ có đúng một bản sao dữ liệu duy nhất. Đây không phải là chữ C trong ACID, mặc dù hai chữ được viết giống hệt nhau, và đây là cái bẫy kinh điển nhất của chủ đề này. Chữ C trong ACID nói về việc một transaction phải giữ được các ràng buộc và các bất biến của dữ liệu, ví dụ khóa ngoại, ràng buộc kiểm tra, hoặc một quy tắc nghiệp vụ. Hai khái niệm này trực giao với nhau, cho nên một hệ hoàn toàn có thể tuân thủ ACID ở từng node mà vẫn chỉ nhất quán cuối cùng ở phạm vi toàn cục. Nếu chúng ta lẫn hai chữ C này trong phỏng vấn, người phỏng vấn sẽ kết luận ngay rằng chúng ta học thuộc lòng chứ chưa thật sự hiểu.

**English (bám cấu trúc tiếng Việt)**

The letter C in CAP has a very narrow meaning, which is linearizability. Linearizability means that every read sees the most recent write, that is, the system behaves as though there is exactly one single copy of the data. This is not the letter C in ACID, although the two letters are written exactly the same, and this is the most classic trap of this topic. The letter C in ACID is about a transaction having to preserve the constraints and the invariants of the data, for example a foreign key, a check constraint, or a business rule. These two concepts are orthogonal to each other, therefore a system can perfectly well comply with ACID on each node while it is still only eventually consistent at the global scope. If we mix up these two letters C in an interview, the interviewer will conclude straight away that we have learned by rote and have not really understood.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| có một nghĩa rất hẹp | has a very narrow meaning |
| nhìn thấy lệnh ghi mới nhất | sees the most recent write |
| như thể chỉ có đúng một bản sao dữ liệu duy nhất | as though there is exactly one single copy of the data |
| mặc dù hai chữ được viết giống hệt nhau | although the two letters are written exactly the same |
| cái bẫy kinh điển nhất của chủ đề này | the most classic trap of this topic |
| các ràng buộc và các bất biến của dữ liệu | the constraints and the invariants of the data |
| trực giao với nhau | orthogonal to each other |
| tuân thủ ACID ở từng node | comply with ACID on each node |
| nhất quán cuối cùng ở phạm vi toàn cục | eventually consistent at the global scope |
| chúng ta học thuộc lòng chứ chưa thật sự hiểu | we have learned by rote and have not really understood |

**Thuật ngữ cần nhớ**

- tính tuyến tính hóa → **linearizability**
- bất biến dữ liệu → **an invariant**
- khóa ngoại → **a foreign key**
- trực giao → **orthogonal**
- nhất quán cuối cùng → **eventually consistent**

---

## ③ CP và AP khác nhau ở hành vi khi partition

**Tiếng Việt**

Cách phân biệt CP và AP không nằm ở lúc mạng lành, mà nằm ở hành vi của hệ đúng vào lúc partition xảy ra. Một hệ CP sẽ từ chối hoặc chặn request lại và trả về lỗi, bởi vì nó thà không phục vụ còn hơn phục vụ một câu trả lời sai. Một hệ AP vẫn tiếp tục phục vụ, nhưng nó có thể trả về dữ liệu cũ hoặc nhận những lệnh ghi xung đột nhau ở hai phía. Nói cách khác, CP chọn đúng còn AP chọn sống, và cả hai lựa chọn đều hợp lý tùy theo bài toán nghiệp vụ. Nếu chúng ta chọn nhầm, hậu quả không phải là hệ chạy chậm, mà là hệ trả sai số dư cho khách hàng, hoặc là hệ ngừng bán hàng trong lúc mạng nội bộ có sự cố.

Về ví dụ cụ thể, các hệ thường được xếp vào nhóm CP gồm etcd, ZooKeeper, HBase, CockroachDB, Spanner, và MongoDB khi chúng ta đặt write concern ở mức majority. Các hệ thường được xếp vào nhóm AP gồm Cassandra khi chạy ở consistency level thấp, DynamoDB với chế độ đọc nhất quán cuối cùng, CouchDB, và cả hệ thống tên miền DNS. Nhưng chúng ta phải nói thêm một câu quan trọng, đó là rất nhiều database ngày nay điều chỉnh được, cho nên cái nhãn không cố định. MongoDB nghiêng về CP hay AP là tùy vào write concern, còn Cassandra cho phép chọn mức nhất quán cho từng câu truy vấn. Vì vậy khi trả lời phỏng vấn, chúng ta nên nói kèm cấu hình chứ chúng ta không đọc thuộc lòng một danh sách.

**English (bám cấu trúc tiếng Việt)**

The way to tell CP and AP apart does not lie in the time when the network is healthy, but lies in the behaviour of the system at exactly the moment when a partition happens. A CP system will refuse or block the request and return an error, because it would rather not serve than serve a wrong answer. An AP system carries on serving, but it may return old data or accept writes that conflict with each other on the two sides. In other words, CP chooses to be right while AP chooses to stay alive, and both choices are reasonable depending on the business problem. If we choose wrongly, the consequence is not that the system runs slowly, but that the system returns the wrong balance to the customer, or that the system stops selling while the internal network has an incident.

As for concrete examples, the systems usually placed in the CP group include etcd, ZooKeeper, HBase, CockroachDB, Spanner, and MongoDB when we set the write concern at the majority level. The systems usually placed in the AP group include Cassandra when it runs at a low consistency level, DynamoDB with eventually consistent reads, CouchDB, and even the domain name system DNS. But we have to add one important sentence, which is that a great many databases nowadays are tunable, therefore the label is not fixed. Whether MongoDB leans towards CP or AP depends on the write concern, while Cassandra allows us to choose the consistency level for each individual query. Therefore when we answer in an interview, we should speak with the configuration attached and we do not recite a list by heart.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| cách phân biệt CP và AP | the way to tell CP and AP apart |
| đúng vào lúc partition xảy ra | at exactly the moment when a partition happens |
| thà không phục vụ còn hơn phục vụ một câu trả lời sai | would rather not serve than serve a wrong answer |
| những lệnh ghi xung đột nhau ở hai phía | writes that conflict with each other on the two sides |
| CP chọn đúng còn AP chọn sống | CP chooses to be right while AP chooses to stay alive |
| tùy theo bài toán nghiệp vụ | depending on the business problem |
| trong lúc mạng nội bộ có sự cố | while the internal network has an incident |
| thường được xếp vào nhóm CP | usually placed in the CP group |
| rất nhiều database ngày nay điều chỉnh được | a great many databases nowadays are tunable |
| cho nên cái nhãn không cố định | therefore the label is not fixed |
| chúng ta nên nói kèm cấu hình | we should speak with the configuration attached |
| không đọc thuộc lòng một danh sách | do not recite a list by heart |

**Thuật ngữ cần nhớ**

- mức nhất quán → **a consistency level**
- mức bảo đảm khi ghi → **a write concern**
- điều chỉnh được → **tunable**
- dữ liệu cũ → **stale data**
- sự cố → **an incident**

---

## ④ PACELC — vế Else mới là phần mô tả đời thường

**Tiếng Việt**

PACELC là một cách phát biểu đầy đủ hơn CAP, và đây là phần mà người phỏng vấn dùng để nhận ra ứng viên đã đọc sâu. Vế đầu nói rằng nếu có partition, chúng ta chọn giữa availability và consistency, và vế này chính là CAP cũ. Vế sau nói rằng nếu không có partition, chúng ta vẫn phải chọn giữa latency và consistency. Điểm sáng của PACELC nằm ở chỗ vế sau mô tả hơn chín mươi chín phần trăm thời gian hoạt động của hệ, tức là những lúc mạng hoàn toàn bình thường. Lý do là muốn có nhất quán mạnh thì mỗi thao tác phải đi một vòng tới leader hoặc tới đa số replica rồi mới trả lời, cho nên nó chậm hơn ngay cả khi không có sự cố nào. CAP hoàn toàn bỏ qua chuyện này, còn PACELC vá đúng vào chỗ đó. Theo cách phân loại này, MongoDB với write concern majority thường được xếp là PC cộng EC, còn Cassandra ở mức mặc định ONE là PA cộng EL, và khi chúng ta chuyển sang QUORUM cho cả đọc lẫn ghi thì nó trở thành PC cộng EC.

**English (bám cấu trúc tiếng Việt)**

PACELC is a fuller way of stating CAP, and this is the part that interviewers use to spot a candidate who has read deeply. The first half says that if there is a partition, we choose between availability and consistency, and this half is exactly the old CAP. The second half says that if there is no partition, we still have to choose between latency and consistency. The bright point of PACELC lies in the fact that the second half describes more than ninety-nine per cent of the operating time of the system, that is, the times when the network is completely normal. The reason is that in order to have strong consistency, every operation has to make a round trip to the leader or to the majority of replicas before it answers, therefore it is slower even when there is no incident at all. CAP ignores this completely, while PACELC patches exactly that spot. Under this classification, MongoDB with a majority write concern is usually placed as PC plus EC, while Cassandra at the default level ONE is PA plus EL, and when we switch to QUORUM for both reads and writes it becomes PC plus EC.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| một cách phát biểu đầy đủ hơn | a fuller way of stating |
| nhận ra ứng viên đã đọc sâu | spot a candidate who has read deeply |
| vế này chính là CAP cũ | this half is exactly the old CAP |
| điểm sáng của PACELC nằm ở chỗ | the bright point of PACELC lies in the fact that |
| hơn chín mươi chín phần trăm thời gian hoạt động | more than ninety-nine per cent of the operating time |
| đi một vòng tới leader | make a round trip to the leader |
| rồi mới trả lời | before it answers |
| ngay cả khi không có sự cố nào | even when there is no incident at all |
| CAP hoàn toàn bỏ qua chuyện này | CAP ignores this completely |
| PACELC vá đúng vào chỗ đó | PACELC patches exactly that spot |
| theo cách phân loại này | under this classification |

**Thuật ngữ cần nhớ**

- độ trễ → **latency**
- một vòng đi và về → **a round trip**
- nhất quán mạnh → **strong consistency**
- đa số bản sao → **the majority of replicas**
- cách phân loại → **a classification**

---

## ⑤ "Chúng tôi chọn availability" chưa phải là câu trả lời

**Tiếng Việt**

Nói rằng "chúng tôi chọn availability" là một câu khẩu hiệu, chứ đó chưa phải là câu trả lời của một người senior. Người phỏng vấn giỏi sẽ hỏi ngay ba con số. Thứ nhất, cửa sổ dữ liệu cũ mà nghiệp vụ chấp nhận được là bao lâu, hai giây hay hai phút. Thứ hai, tỉ lệ xung đột dự kiến là bao nhiêu, và thứ ba, chúng ta hòa giải xung đột bằng cách nào khi mạng lành trở lại.

Về cách hòa giải xung đột, chúng ta có mấy lựa chọn quen thuộc và mỗi lựa chọn mang một cái giá riêng. Cách đơn giản nhất là last-write-wins, tức là chúng ta lấy bản ghi có timestamp lớn hơn, nhưng cách này âm thầm vứt bỏ một lệnh ghi hợp lệ. Cách thứ hai là vector clock hoặc version vector, thứ giúp chúng ta phát hiện được rằng hai lệnh ghi là đồng thời chứ không phải nối tiếp nhau. Cách thứ ba là CRDT, tức là những cấu trúc dữ liệu tự hợp nhất được một cách đúng đắn mà không cần con người quyết định. Và cách cuối cùng là để tầng ứng dụng tự hợp nhất theo luật nghiệp vụ, cách này linh hoạt nhất nhưng nó cũng tốn công nhất.

**English (bám cấu trúc tiếng Việt)**

Saying that "we choose availability" is a slogan, and that is not yet the answer of a senior person. A good interviewer will immediately ask for three numbers. First, how long is the window of stale data that the business can accept, two seconds or two minutes. Second, what is the expected conflict rate, and third, how do we reconcile the conflicts when the network comes back healthy.

As for the way to reconcile conflicts, we have a few familiar options and each option carries its own price. The simplest one is last-write-wins, that is, we take the record with the larger timestamp, but this way silently throws away a legitimate write. The second one is a vector clock or a version vector, which helps us detect that two writes are concurrent and not one after the other. The third one is CRDT, that is, data structures that can merge themselves correctly without a human deciding. And the last one is to let the application layer merge by the business rules, this way is the most flexible but it is also the most laborious.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| là một câu khẩu hiệu | is a slogan |
| sẽ hỏi ngay ba con số | will immediately ask for three numbers |
| cửa sổ dữ liệu cũ | the window of stale data |
| tỉ lệ xung đột dự kiến | the expected conflict rate |
| khi mạng lành trở lại | when the network comes back healthy |
| mỗi lựa chọn mang một cái giá riêng | each option carries its own price |
| âm thầm vứt bỏ một lệnh ghi hợp lệ | silently throws away a legitimate write |
| là đồng thời chứ không phải nối tiếp nhau | are concurrent and not one after the other |
| tự hợp nhất được một cách đúng đắn | can merge themselves correctly |
| nó cũng tốn công nhất | it is also the most laborious |

**Thuật ngữ cần nhớ**

- cửa sổ dữ liệu cũ → **the staleness window**
- hòa giải xung đột → **conflict resolution**
- ghi sau thắng → **last-write-wins**
- đồng hồ vector → **a vector clock**
- kiểu dữ liệu tự hợp nhất → **a CRDT**

---

## ⑥ Partition không nhị phân, và đừng mang CAP vào chỗ không thuộc về nó

**Tiếng Việt**

Trong thực tế, partition không phải là một trạng thái bật hoặc tắt rõ ràng như trong lý thuyết. Node A có thể coi một lần timeout là bằng chứng rằng node B đã chết, trong khi node B vẫn đang chạy hoàn toàn bình thường. Khi đó chính các node trong hệ cũng không thống nhất được với nhau về việc có partition hay không. Cho nên chúng ta nên nhớ rằng CAP là một mô hình đơn giản hóa, và nó hữu ích để suy nghĩ chứ nó không mô tả đầy đủ thực tế.

Một sai lầm khác là chúng ta mang CAP vào những chỗ không thuộc phạm vi của nó. CAP chỉ nói về hệ phân tán có nhiều bản sao dữ liệu, cho nên một PostgreSQL đơn lẻ không hề chọn CP hay AP. Nếu người phỏng vấn hỏi rằng database đơn node của chúng ta là CP hay AP, câu trả lời đúng là chúng ta chỉ ra rằng câu hỏi đó đặt sai chỗ. Ngoài ra, phần lớn ứng dụng vừa và nhỏ không cần dùng đến lập luận CAP để ra quyết định. Khi một ứng viên mang CAP vào mọi câu trả lời, đó thường là dấu hiệu của học vẹt chứ đó không phải là dấu hiệu của hiểu sâu.

**English (bám cấu trúc tiếng Việt)**

In practice, a partition is not a clearly on or off state as it is in theory. Node A may treat one timeout as evidence that node B has died, while node B is still running completely normally. At that point the nodes in the system themselves cannot even agree with each other about whether there is a partition or not. Therefore we should remember that CAP is a simplified model, and it is useful for thinking but it does not describe reality fully.

Another mistake is that we bring CAP into places that do not fall within its scope. CAP only talks about a distributed system that has several copies of the data, therefore a single PostgreSQL does not choose CP or AP at all. If the interviewer asks whether our single-node database is CP or AP, the correct answer is that we point out that the question is misplaced. Besides that, most small and medium applications do not need to use CAP reasoning in order to make a decision. When a candidate brings CAP into every answer, that is usually a sign of rote learning and it is not a sign of deep understanding.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| một trạng thái bật hoặc tắt rõ ràng | a clearly on or off state |
| coi một lần timeout là bằng chứng rằng | treat one timeout as evidence that |
| không thống nhất được với nhau về việc có partition hay không | cannot even agree with each other about whether there is a partition or not |
| một mô hình đơn giản hóa | a simplified model |
| hữu ích để suy nghĩ | useful for thinking |
| những chỗ không thuộc phạm vi của nó | places that do not fall within its scope |
| không hề chọn CP hay AP | does not choose CP or AP at all |
| câu hỏi đó đặt sai chỗ | the question is misplaced |
| không cần dùng đến lập luận CAP | do not need to use CAP reasoning |
| dấu hiệu của học vẹt | a sign of rote learning |

**Thuật ngữ cần nhớ**

- bằng chứng → **evidence**
- mô hình đơn giản hóa → **a simplified model**
- phạm vi áp dụng → **the scope of application**
- đặt sai chỗ → **misplaced**
- học vẹt → **rote learning**

---

## ⑦ Nhất quán là lựa chọn theo từng thao tác

**Tiếng Việt**

Nhận thức quan trọng nhất của bài này là tính nhất quán được chọn theo từng thao tác, chứ nó không phải là một cái nhãn dán cho cả hệ thống. Một hệ thương mại điện tử hoàn toàn có thể vừa nhất quán mạnh ở luồng tiền, vừa nhất quán cuối cùng ở luồng phân tích, và đó là thiết kế đúng chứ đó không phải là mâu thuẫn. Số dư ví và trạng thái thanh toán nên chọn CP, bởi vì chúng ta thà báo lỗi cho khách còn hơn hiển thị sai số tiền. Ngược lại, bộ đếm lượt xem, danh sách gợi ý và feed nên chọn AP, bởi vì lệch vài giây thì không ai thiệt hại còn ngừng phục vụ thì mất doanh thu. Việc của chúng ta là ghi rõ chính sách nhất quán cho từng nhóm API trong tài liệu kiến trúc, rồi viết test cho những luồng nhạy cảm để bắt lỗi đặt nhầm mức. Nếu chúng ta để một mức mặc định duy nhất áp cho mọi truy vấn, sớm muộn sẽ có một luồng tiền chạy ở mức quá yếu, và đó là loại bug rất lâu mới lộ ra.

**English (bám cấu trúc tiếng Việt)**

The most important realisation of this lesson is that consistency is chosen per operation, and it is not a label stuck onto the whole system. An e-commerce system can perfectly well be both strongly consistent on the money flow and eventually consistent on the analytics flow, and that is a correct design and it is not a contradiction. The wallet balance and the payment status should choose CP, because we would rather return an error to the customer than display the wrong amount of money. On the other hand, the view counter, the recommendation list and the feed should choose AP, because being a few seconds out harms nobody while stopping the service loses revenue. Our job is to write the consistency policy for each group of APIs clearly in the architecture document, and then to write tests for the sensitive flows in order to catch a level that has been set wrongly. If we let one single default level apply to every query, sooner or later there will be a money flow running at too weak a level, and that is the kind of bug that only shows up a very long time later.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| được chọn theo từng thao tác | is chosen per operation |
| một cái nhãn dán cho cả hệ thống | a label stuck onto the whole system |
| ở luồng tiền … ở luồng phân tích | on the money flow … on the analytics flow |
| đó là thiết kế đúng chứ đó không phải là mâu thuẫn | that is a correct design and it is not a contradiction |
| thà báo lỗi cho khách còn hơn hiển thị sai số tiền | would rather return an error to the customer than display the wrong amount of money |
| lệch vài giây thì không ai thiệt hại | being a few seconds out harms nobody |
| ngừng phục vụ thì mất doanh thu | stopping the service loses revenue |
| ghi rõ chính sách nhất quán cho từng nhóm API | write the consistency policy for each group of APIs clearly |
| để bắt lỗi đặt nhầm mức | in order to catch a level that has been set wrongly |
| một mức mặc định duy nhất áp cho mọi truy vấn | one single default level applying to every query |
| loại bug rất lâu mới lộ ra | the kind of bug that only shows up a very long time later |

**Thuật ngữ cần nhớ**

- theo từng thao tác → **per operation**
- luồng nghiệp vụ → **a business flow**
- doanh thu → **revenue**
- tài liệu kiến trúc → **the architecture document**
- luồng nhạy cảm → **a sensitive flow**

---

## Mô hình ghi nhớ

**Tiếng Việt**

CAP nói rằng khi mạng chia cắt, chúng ta chọn đúng hay chọn sống; PACELC nói thêm rằng khi mạng lành, chúng ta vẫn phải chọn nhanh hay chọn đúng. Và lựa chọn này là theo từng thao tác, chứ nó không phải là một cái nhãn dán cho cả hệ thống.

**English (bám cấu trúc tiếng Việt)**

CAP says that when the network splits, we choose to be right or we choose to stay alive; PACELC adds that when the network is healthy, we still have to choose to be fast or to be right. And this choice is per operation, and it is not a label stuck onto the whole system.

---

## Bảng thuật ngữ tổng hợp

| Tiếng Việt | Tiếng Anh | Ghi chú phát âm (Anh–Anh) |
|---|---|---|
| chia cắt mạng | **a network partition** | *paa-TI-shợn* — trọng âm giữa |
| khả năng sẵn sàng phục vụ | **availability** | *ơ-vei-lơ-BI-lơ-ti* — sáu âm tiết, trọng âm rơi vào **-BI-** |
| tính nhất quán | **consistency** | *kơn-SIS-tợn-si* — trọng âm âm thứ hai |
| điều kiện bắt buộc | **a compulsory condition** | *kơm-PAL-sơ-ri* — trọng âm âm thứ hai |
| định lý | **a theorem** | *THI-ơ-rợm* /ˈθɪərəm/ — âm **th**, không đọc "thê-ô-rem" |
| tính tuyến tính hóa | **linearizability** | *li-ni-ơ-rai-zơ-BI-lơ-ti* — bảy âm tiết, tập tách chậm từng cụm |
| bất biến dữ liệu | **an invariant** | *in-VAIR-ri-ợnt* — trọng âm âm thứ hai |
| khóa ngoại | **a foreign key** | *foreign* /ˈfɒrən/ — câm chữ **g**, đọc "FO-rợn" |
| ràng buộc | **a constraint** | *kơn-STRÊINT* — cụm **str** và cụm cuối **-nt** |
| trực giao | **orthogonal** | *o-THO-gơ-nợl* — có âm **th** /θ/, trọng âm âm thứ hai |
| nhất quán cuối cùng | **eventually consistent** | *i-VEN-tsu-ơ-li* — trọng âm âm thứ hai |
| mức nhất quán | **a consistency level** | |
| mức bảo đảm khi ghi | **a write concern** | *write* câm chữ **w**; *concern* = *kơn-SƠƠN*, trọng âm sau |
| điều chỉnh được | **tunable** | *TYUU-nơ-bợl* — kiểu Anh có âm /tj/ |
| dữ liệu cũ | **stale data** | *stale* /steɪl/; *data* Anh = *DÊI-tơ* |
| sự cố | **an incident** | *IN-si-dợnt* — trọng âm đầu |
| câu truy vấn | **a query** | Anh: *KWIA-ri* /ˈkwɪəri/ — không đọc "quê-ri" |
| độ trễ | **latency** | *LÊI-tợn-si* — trọng âm đầu, âm đầu là /eɪ/ |
| một vòng đi và về | **a round trip** | |
| nhất quán mạnh | **strong consistency** | |
| đa số bản sao | **the majority of replicas** | *replica* Anh = *RE-pli-kơ*, trọng âm đầu |
| cách phân loại | **a classification** | *kla-si-fi-KÊI-shợn* — trọng âm áp chót |
| cửa sổ dữ liệu cũ | **the staleness window** | |
| hòa giải xung đột | **conflict resolution** | *re-zơ-LUU-shợn* — trọng âm áp chót, chữ `s` đọc /z/ |
| ghi sau thắng | **last-write-wins** | |
| đồng hồ vector | **a vector clock** | *VEK-tơ* — trọng âm đầu |
| kiểu dữ liệu tự hợp nhất | **a CRDT** | đọc rời từng chữ cái: *si-aa-di-TI* |
| dấu thời gian | **a timestamp** | cụm **-mp-** ở giữa phải bật ra |
| đồng thời | **concurrent** | *kơn-KA-rợnt* — trọng âm giữa |
| hợp nhất | **to merge** | /mɜːdʒ/ — đuôi /dʒ/ |
| tốn công | **laborious** | *lơ-BO-ri-ợs* — trọng âm âm thứ hai |
| bằng chứng | **evidence** | *E-vi-dợns* — trọng âm đầu, cụm cuối **-ns** |
| mô hình đơn giản hóa | **a simplified model** | *SIM-pli-faid* — trọng âm đầu |
| phạm vi áp dụng | **the scope of application** | *scope* /skəʊp/ — cụm **sk** đầu và **-p** cuối đều phải rõ |
| đặt sai chỗ | **misplaced** | *mis-PLÊIST* — cụm cuối **-st** |
| học vẹt | **rote learning** | *rote* /rəʊt/ — đọc như "wrote" |
| theo từng thao tác | **per operation** | *o-pơ-RÊI-shợn* — trọng âm âm thứ ba |
| luồng nghiệp vụ | **a business flow** | *business* /ˈbɪznəs/ — hai âm tiết khi nói, không đọc "bi-di-nét" |
| doanh thu | **revenue** | *RE-vơ-nyuu* — trọng âm đầu, kiểu Anh có /nj/ |
| tài liệu kiến trúc | **the architecture document** | *AA-ki-tek-tsơ* — `ch` đọc /k/, trọng âm đầu |
| luồng nhạy cảm | **a sensitive flow** | *SEN-sơ-tiv* — trọng âm đầu |
| phân tích dữ liệu | **analytics** | *a-nơ-LI-tiks* — trọng âm âm thứ ba, cụm cuối **-ks** |

---

## Bài nói

> **Cách luyện:** bật ghi âm, nói **60–90 giây** cho mỗi đề, **không nhìn tài liệu**. Mỗi bài nói phải dùng ít nhất **sáu thuật ngữ** trong bảng trên. Nói xong nghe lại một lần, đánh dấu chỗ nào bị vấp hoặc phát âm sai, rồi nói lại đề đó một lần nữa.

1. **Explain to a junior developer** what CAP actually says, and why "pick two out of three" is a misleading way to remember it.

2. **A colleague says:** *"Our database is ACID compliant, so it gives us the C in CAP."* Explain what is wrong with that claim, and describe both meanings of the letter C precisely.

3. **Describe what happens**, step by step, to a CP system and to an AP system during the same ten-second network partition. Say what each one returns to the user and what state the data is in afterwards.

4. **Explain why PACELC is more useful than CAP** for day-to-day design decisions, and give one example of a system where the Else half changed a choice you would make.

5. **Someone on your team says:** *"We are an AP system, we chose availability."* Explain why you would push back on that as an architecture statement, and what three things you would ask them to quantify.

6. **When would you accept last-write-wins as a conflict resolution strategy**, and when would you insist on version vectors or a CRDT instead? Ground your answer in what the business loses if a write disappears.

7. **A design review proposes labelling the whole platform as "eventually consistent" to keep things simple.** Explain why you disagree, and how you would document consistency per operation instead.

---

> 🔚 **Hết Bài 5.** Gõ `Làm Bài 6` để sang *Consistency Models & Eventual Consistency*.
