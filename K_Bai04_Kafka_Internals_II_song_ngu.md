# Bài 4 — Kafka Internals II: replication, rebalance, retention, EOS, lag
### Bản song ngữ: tiếng Anh bám cấu trúc tiếng Việt

> **Cách dùng bài này:** đọc đoạn tiếng Việt trước → tự dịch trong đầu, không nhìn xuống → so sánh với đoạn tiếng Anh ngay bên dưới → **đọc to bản tiếng Anh hai lần**: lần một đọc chậm cho đúng từng âm, lần hai đọc liền mạch như đang nói với người phỏng vấn.

---

## ① Nhân bản và ISR

**Tiếng Việt**

Bài trước dựng nền về partition và offset, còn bài này hoàn thiện phần vận hành Kafka. Chúng ta bắt đầu bằng nhân bản, vì đó là cách Kafka giữ cho dữ liệu không mất khi một máy chết. Mỗi partition có một bản leader và một số bản follower, và tổng số bản đó gọi là hệ số nhân bản. Producer và consumer chỉ làm việc với bản leader, còn các follower thì liên tục kéo dữ liệu từ leader về để chép lại. Tập hợp các bản đang theo kịp leader được gọi là ISR, tức là các bản sao đang đồng bộ. Một follower tụt quá xa sẽ bị đẩy ra khỏi ISR, và nó chỉ được quay lại khi nó đuổi kịp.

Chúng ta cần hiểu vì sao ISR lại quan trọng đến vậy. ISR chính là danh sách những bản sao đủ điều kiện được bầu lên làm leader mới. Khi leader chết, bộ điều khiển của cluster sẽ chọn một follower trong ISR lên thay, rồi producer và consumer tự kết nối lại vào leader mới. Nếu chúng ta cho phép bầu một bản sao nằm ngoài ISR lên làm leader, chúng ta sẽ mất những message mà bản sao đó chưa kịp chép. Vì vậy ISR vừa là thước đo sức khoẻ của một partition, vừa là người gác cổng cho việc bầu leader.

**English (bám cấu trúc tiếng Việt)**

The previous lesson built the foundation about partitions and offsets, while this lesson completes the operational side of Kafka. We start with replication, because that is how Kafka keeps the data from being lost when a machine dies. Each partition has one leader copy and a number of follower copies, and the total number of those copies is called the replication factor. Producers and consumers only work with the leader copy, while the followers continuously pull data from the leader in order to copy it. The set of copies that are keeping up with the leader is called the ISR, that is, the in-sync replicas. A follower that falls too far behind will be pushed out of the ISR, and it is only allowed back when it catches up.

We need to understand why the ISR is so important. The ISR is exactly the list of replicas that are eligible to be elected as the new leader. When the leader dies, the controller of the cluster will pick a follower inside the ISR to take over, and then producers and consumers reconnect to the new leader by themselves. If we allow a replica outside the ISR to be elected as the leader, we will lose the messages that this replica has not managed to copy. Therefore the ISR is both a measure of the health of a partition and the gatekeeper for leader election.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| hoàn thiện phần vận hành | completes the operational side |
| giữ cho dữ liệu không mất | keeps the data from being lost |
| tổng số bản đó gọi là | the total number of those copies is called |
| kéo dữ liệu từ leader về để chép lại | pull data from the leader in order to copy it |
| các bản đang theo kịp leader | the copies that are keeping up with the leader |
| tụt quá xa | falls too far behind |
| chỉ được quay lại khi nó đuổi kịp | is only allowed back when it catches up |
| đủ điều kiện được bầu lên | eligible to be elected |
| lên thay | to take over |
| chưa kịp chép | has not managed to copy |
| thước đo sức khoẻ | a measure of the health |
| người gác cổng | the gatekeeper |

**Thuật ngữ cần nhớ**

- nhân bản → **replication**
- bản chính và bản theo → **leader and follower**
- các bản sao đang đồng bộ → **in-sync replicas (ISR)**
- hệ số nhân bản → **the replication factor**
- bầu leader mới → **leader election**

---

## ② acks, min.insync.replicas và đánh đổi giữa bền và sẵn sàng

**Tiếng Việt**

Nhân bản một mình vẫn chưa đủ, vì chúng ta còn phải nói cho producer biết nó phải chờ tới đâu. Cấu hình `acks` quyết định chính điều đó. Nếu chúng ta đặt `acks` bằng `all`, producer chỉ coi là ghi thành công khi mọi bản sao trong ISR đều đã ghi xong. Cấu hình thứ hai là `min.insync.replicas`, tức là số bản sao đồng bộ tối thiểu để broker chấp nhận một lần ghi. Khi ISR tụt xuống dưới ngưỡng đó, broker sẽ từ chối ghi và producer nhận về một lỗi, thay vì âm thầm ghi vào một bản duy nhất rồi mất dữ liệu lúc leader chết.

Ở đây có một đánh đổi mà chúng ta phải nói rõ trong phỏng vấn. Chờ nhiều bản sao hơn thì độ bền cao hơn, nhưng độ trễ tăng lên và khả năng ghi giảm xuống. Nói cụ thể hơn, khi cluster mất bớt node thì chúng ta chủ động chọn dừng ghi thay vì chấp nhận rủi ro mất dữ liệu. Cấu hình kinh điển cho một topic quan trọng là hệ số nhân bản bằng ba, số bản đồng bộ tối thiểu bằng hai, và `acks` bằng `all`. Nếu chúng ta để `acks` bằng `1` cho nhanh, hậu quả là một lần leader chết ngay sau khi ghi sẽ nuốt mất những message vừa được xác nhận, trong khi producer thì đã tưởng mọi thứ đều ổn.

**English (bám cấu trúc tiếng Việt)**

Replication alone is still not enough, because we also have to tell the producer how far it must wait. The `acks` setting decides exactly that. If we set `acks` to `all`, the producer only counts the write as successful when every replica in the ISR has finished writing. The second setting is `min.insync.replicas`, that is, the minimum number of in-sync replicas for the broker to accept one write. When the ISR drops below that threshold, the broker will reject the write and the producer receives an error back, instead of silently writing into a single copy and then losing the data when the leader dies.

Here there is a trade-off that we must state clearly in an interview. Waiting for more replicas means higher durability, but the latency goes up and the ability to write goes down. To put it more concretely, when the cluster loses some nodes we deliberately choose to stop writing instead of accepting the risk of losing data. The classic configuration for an important topic is a replication factor of three, a minimum of two in-sync replicas, and `acks` set to `all`. If we set `acks` to `1` for speed, the consequence is that one leader death right after a write will swallow the messages that have just been acknowledged, while the producer has already assumed that everything is fine.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| nói cho producer biết nó phải chờ tới đâu | tell the producer how far it must wait |
| quyết định chính điều đó | decides exactly that |
| chỉ coi là ghi thành công khi | only counts the write as successful when |
| tụt xuống dưới ngưỡng đó | drops below that threshold |
| producer nhận về một lỗi | the producer receives an error back |
| thay vì âm thầm ghi vào một bản duy nhất | instead of silently writing into a single copy |
| nói cụ thể hơn | to put it more concretely |
| chúng ta chủ động chọn dừng ghi | we deliberately choose to stop writing |
| chấp nhận rủi ro mất dữ liệu | accepting the risk of losing data |
| sẽ nuốt mất những message vừa được xác nhận | will swallow the messages that have just been acknowledged |
| producer thì đã tưởng mọi thứ đều ổn | the producer has already assumed that everything is fine |

**Thuật ngữ cần nhớ**

- ngưỡng tối thiểu → **the minimum threshold**
- từ chối một lần ghi → **to reject a write**
- độ bền đổi lấy khả năng sẵn sàng → **durability traded for availability**
- xác nhận đã ghi → **acknowledged**
- cấu hình kinh điển → **the classic configuration**

---

## ③ Rebalance: từ dừng cả thế giới sang chuyển dần

**Tiếng Việt**

Rebalance là quá trình một nhóm consumer chia lại các partition cho nhau. Nó được kích hoạt khi một consumer tham gia, khi một consumer rời đi hoặc chết, hoặc khi số partition thay đổi. Trong kiểu rebalance cũ, tất cả consumer đều phải trả lại toàn bộ partition rồi mới nhận phần mới, nên cả nhóm ngừng xử lý trong suốt khoảng thời gian đó. Người ta gọi kiểu đó là dừng cả thế giới, và hậu quả trực tiếp là độ trễ tiêu thụ tăng vọt mỗi lần chúng ta triển khai một phiên bản mới. Nguy hiểm hơn nữa, nếu một consumer chưa kịp commit trước khi bị thu hồi partition, phần việc đó sẽ được làm lại bởi một consumer khác.

Đây là chỗ mà kiến thức cũ cần được cập nhật, và người phỏng vấn rất thích chi tiết này. Kafka đã chuyển sang kiểu rebalance hợp tác, nghĩa là nhóm chỉ chuyển đúng những partition cần chuyển và giữ nguyên phần còn lại. Giao thức nhóm mới đã chính thức ổn định từ Kafka phiên bản bốn, và consumer bật nó bằng cấu hình giao thức nhóm. Vì vậy khi bị hỏi rebalance có phải là dừng cả thế giới hay không, chúng ta nên trả lời rằng điều đó đúng với kiểu cũ, còn hiện nay ngành đang chuyển sang kiểu chuyển dần. Câu trả lời như vậy cho thấy chúng ta theo dõi công nghệ chứ không chỉ đọc lại tài liệu cũ.

**English (bám cấu trúc tiếng Việt)**

A rebalance is the process in which a consumer group divides the partitions among themselves again. It is triggered when a consumer joins, when a consumer leaves or dies, or when the number of partitions changes. In the old rebalance style, all consumers had to give back all their partitions before they received the new share, so the whole group stopped processing during that whole period. People call that style stop-the-world, and the direct consequence is that consumer lag jumps up every time we deploy a new version. Even more dangerously, if a consumer has not managed to commit before its partitions are revoked, that piece of work will be done again by another consumer.

This is the place where old knowledge needs to be updated, and interviewers really like this detail. Kafka has moved to the cooperative rebalance style, which means that the group only moves exactly the partitions that need to be moved and keeps the rest untouched. The new group protocol has been officially stable since Kafka version four, and a consumer turns it on through the group protocol setting. Therefore when we are asked whether a rebalance is stop-the-world or not, we should answer that this was true for the old style, while nowadays the industry is moving to the incremental style. An answer like that shows that we follow the technology rather than only re-reading old documentation.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| chia lại các partition cho nhau | divides the partitions among themselves again |
| nó được kích hoạt khi | it is triggered when |
| trả lại toàn bộ partition | give back all their partitions |
| ngừng xử lý trong suốt khoảng thời gian đó | stopped processing during that whole period |
| độ trễ tiêu thụ tăng vọt | consumer lag jumps up |
| nguy hiểm hơn nữa | even more dangerously |
| chưa kịp commit | has not managed to commit |
| trước khi bị thu hồi partition | before its partitions are revoked |
| giữ nguyên phần còn lại | keeps the rest untouched |
| đã chính thức ổn định từ | has been officially stable since |
| ngành đang chuyển sang kiểu chuyển dần | the industry is moving to the incremental style |
| chứ không chỉ đọc lại tài liệu cũ | rather than only re-reading old documentation |

**Thuật ngữ cần nhớ**

- chia lại partition → **a rebalance**
- bị thu hồi → **to be revoked**
- dừng cả thế giới → **stop-the-world**
- kiểu hợp tác, chuyển dần → **cooperative / incremental**
- triển khai phiên bản mới → **to deploy a new version**

---

## ④ Retention và compaction: lịch sử hay trạng thái

**Tiếng Việt**

Vòng đời dữ liệu trong Kafka có hai chính sách dọn dẹp, và chúng ta phải chọn đúng một cái cho mỗi topic. Chính sách thứ nhất là giữ theo thời hạn, nghĩa là broker xoá những message cũ hơn một khoảng thời gian hoặc vượt quá một dung lượng. Đây là lựa chọn mặc định, và nó hợp với những topic ghi lại dòng sự kiện thông thường. Chính sách thứ hai là nén theo khoá, nghĩa là broker chỉ giữ lại bản ghi mới nhất của mỗi khoá và xoá các bản cũ hơn có cùng khoá. Với chính sách này, topic biến thành một ảnh chụp trạng thái hiện tại của từng khoá.

Cách chọn rất gọn nếu chúng ta hỏi đúng một câu, đó là chúng ta cần lịch sử đầy đủ hay chỉ cần trạng thái hiện tại. Nếu chúng ta cần dựng lại toàn bộ chuỗi sự kiện, ví dụ với một sổ cái tài chính, chúng ta giữ theo thời hạn và đặt thời hạn thật dài. Nếu chúng ta chỉ cần biết số dư mới nhất của mỗi tài khoản để một service khởi động và nạp lại trạng thái, chúng ta dùng nén theo khoá. Nếu chúng ta chọn nhầm và bật nén cho một topic sổ cái, hậu quả là chúng ta mất vĩnh viễn các bút toán trung gian và không bao giờ đối soát lại được. Ngược lại, nếu chúng ta để một topic trạng thái chạy theo thời hạn, service khởi động sẽ phải đọc hàng triệu bản ghi cũ chỉ để lấy ra vài nghìn giá trị cuối cùng.

**English (bám cấu trúc tiếng Việt)**

The data lifecycle in Kafka has two cleanup policies, and we have to choose exactly one of them for each topic. The first policy is retention by time, which means that the broker deletes the messages that are older than a period of time or that go beyond a size limit. This is the default choice, and it suits the topics that record an ordinary stream of events. The second policy is compaction by key, which means that the broker only keeps the newest record of each key and deletes the older records that have the same key. With this policy, the topic turns into a snapshot of the current state of each key.

The way to choose is very short if we ask exactly one question, which is whether we need the full history or only the current state. If we need to rebuild the whole chain of events, for example with a financial ledger, we keep by retention and we set the period really long. If we only need to know the newest balance of each account so that a service can start up and load the state again, we use compaction by key. If we choose wrongly and turn compaction on for a ledger topic, the consequence is that we permanently lose the intermediate entries and we can never reconcile them again. On the other hand, if we leave a state topic running on retention, a service that starts up will have to read millions of old records only to pull out a few thousand final values.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| vòng đời dữ liệu | the data lifecycle |
| chính sách dọn dẹp | cleanup policy |
| vượt quá một dung lượng | go beyond a size limit |
| ghi lại dòng sự kiện thông thường | record an ordinary stream of events |
| nén theo khoá | compaction by key |
| bản ghi mới nhất của mỗi khoá | the newest record of each key |
| một ảnh chụp trạng thái hiện tại | a snapshot of the current state |
| dựng lại toàn bộ chuỗi sự kiện | rebuild the whole chain of events |
| sổ cái tài chính | a financial ledger |
| nạp lại trạng thái | load the state again |
| mất vĩnh viễn các bút toán trung gian | permanently lose the intermediate entries |
| không bao giờ đối soát lại được | can never reconcile them again |
| chỉ để lấy ra vài nghìn giá trị cuối cùng | only to pull out a few thousand final values |

**Thuật ngữ cần nhớ**

- giữ theo thời hạn → **retention**
- nén theo khoá → **log compaction**
- ảnh chụp trạng thái → **a snapshot of the state**
- sổ cái → **a ledger**
- đối soát → **to reconcile**

---

## ⑤ Exactly-once trong Kafka được ghép từ hai mảnh

**Tiếng Việt**

Bây giờ chúng ta nói về exactly-once, vì đây là chủ đề bị hiểu sai nhiều nhất trong cả mục này. Kafka đạt được nó bằng cách ghép hai mảnh riêng biệt lại với nhau. Mảnh thứ nhất là producer bất biến khi lặp, nghĩa là mỗi producer có một mã định danh và mỗi message có một số thứ tự, nên broker nhận ra và bỏ qua bản trùng khi producer thử lại. Mảnh thứ hai là transaction, nghĩa là chúng ta gói một chuỗi đọc, xử lý và ghi thành một đơn vị nguyên tử, để hoặc cả lô cùng được commit, hoặc không có gì được commit. Ở các đời Kafka mới, phần producer bất biến khi lặp đã được bật sẵn theo mặc định, nhưng chúng ta vẫn nên kiểm chứng lại theo từng thư viện client.

Chúng ta nên nêu rõ vì sao hai mảnh này giải hai bài toán khác nhau. Producer bất biến khi lặp chỉ chống trùng ở chặng từ producer tới broker, ví dụ khi mạng chập chờn và producer phải gửi lại. Transaction thì chống trạng thái nửa vời ở chặng xử lý, ví dụ khi một tiến trình đọc từ topic này, tính toán, rồi ghi sang topic kia và commit offset. Nếu chúng ta chỉ bật một trong hai mảnh, chúng ta vẫn còn một nửa bài toán chưa được giải. Vì vậy khi trả lời phỏng vấn, chúng ta nên gọi tên cả hai mảnh chứ không chỉ nói mỗi chữ exactly-once.

**English (bám cấu trúc tiếng Việt)**

Now we talk about exactly-once, because this is the topic that is misunderstood the most in this whole section. Kafka achieves it by joining two separate pieces together. The first piece is the idempotent producer, which means that each producer has an identifier and each message has a sequence number, so the broker recognises and drops the duplicate when the producer retries. The second piece is transactions, which means that we wrap a chain of read, process and write into one atomic unit, so that either the whole batch is committed together, or nothing is committed. In the newer Kafka versions, the idempotent producer part is already turned on by default, but we should still verify it again for each client library.

We should spell out why these two pieces solve two different problems. The idempotent producer only prevents duplicates on the leg from the producer to the broker, for example when the network is flaky and the producer has to send again. Transactions prevent a half-finished state on the processing leg, for example when a process reads from this topic, computes, then writes to that topic and commits the offset. If we only turn on one of the two pieces, we still have half of the problem unsolved. Therefore when we answer in an interview, we should name both pieces rather than only saying the words exactly-once.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| bị hiểu sai nhiều nhất | is misunderstood the most |
| ghép hai mảnh riêng biệt lại với nhau | joining two separate pieces together |
| bỏ qua bản trùng | drops the duplicate |
| gói một chuỗi đọc, xử lý và ghi | wrap a chain of read, process and write |
| thành một đơn vị nguyên tử | into one atomic unit |
| hoặc cả lô cùng được commit | either the whole batch is committed together |
| đã được bật sẵn theo mặc định | is already turned on by default |
| chúng ta nên nêu rõ | we should spell out |
| chặng từ producer tới broker | the leg from the producer to the broker |
| khi mạng chập chờn | when the network is flaky |
| trạng thái nửa vời | a half-finished state |
| chúng ta vẫn còn một nửa bài toán chưa được giải | we still have half of the problem unsolved |

**Thuật ngữ cần nhớ**

- producer bất biến khi lặp → **the idempotent producer**
- số thứ tự chống trùng → **the sequence number**
- giao dịch nguyên tử → **an atomic transaction**
- bản trùng → **a duplicate**
- đọc, xử lý rồi ghi → **read-process-write**

---

## ⑥ Biên của exactly-once và lý do consumer vẫn phải chống trùng

**Tiếng Việt**

Điều quan trọng nhất về exactly-once là biên của nó, và đây chính là chỗ chúng ta ghi điểm với người phỏng vấn. Mọi bảo đảm của Kafka chỉ có hiệu lực bên trong Kafka. Khi consumer của chúng ta ghi vào một cơ sở dữ liệu bên ngoài hoặc gọi một API bên ngoài, hành động đó nằm ngoài transaction của Kafka. Nếu tiến trình chết sau khi ghi vào cơ sở dữ liệu nhưng trước khi commit, message sẽ được giao lại và chúng ta ghi thêm một lần nữa. Vì vậy điều tốt nhất chúng ta đạt được ở biên ngoài là hiệu quả một lần, nghĩa là hệ thống có thể xử lý lại nhưng kết quả cuối cùng chỉ tính một lần.

Khi một đồng nghiệp bật transaction rồi tuyên bố rằng bây giờ ghi vào cơ sở dữ liệu cũng exactly-once nên bỏ được phần chống trùng ở consumer, chúng ta phải phản biện ngay. Chúng ta hỏi lại đúng một câu, đó là cơ sở dữ liệu kia có tham gia vào transaction của Kafka hay không. Câu trả lời luôn là không, vì hai hệ thống này không chia sẻ một transaction chung. Từ đó chúng ta chỉ ra rằng chỉ cần một lần giao lại là hàng bị trừ kho hai lần hoặc khách bị tính tiền hai lần. Kết luận là chúng ta vẫn phải giữ khoá chống trùng hoặc dùng lệnh ghi kiểu chèn-hoặc-cập-nhật ở phía consumer, và transaction không miễn cho chúng ta việc đó.

**English (bám cấu trúc tiếng Việt)**

The most important thing about exactly-once is its boundary, and this is exactly where we score points with the interviewer. Every guarantee of Kafka is only in force inside Kafka. When our consumer writes into an external database or calls an external API, that action sits outside the Kafka transaction. If the process dies after writing into the database but before the commit, the message will be delivered again and we write one more time. Therefore the best thing we can achieve at the outer boundary is effectively-once, which means that the system may reprocess but the final result only counts once.

When a colleague turns transactions on and then declares that writing into the database is now exactly-once as well, so the duplicate protection in the consumer can be dropped, we have to push back straight away. We ask back exactly one question, which is whether that database takes part in the Kafka transaction or not. The answer is always no, because these two systems do not share one common transaction. From there we point out that only one redelivery is enough for stock to be taken down twice or for a customer to be charged twice. The conclusion is that we still have to keep a deduplication key or use an insert-or-update write on the consumer side, and transactions do not excuse us from that.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| biên của nó | its boundary |
| chỗ chúng ta ghi điểm | where we score points |
| chỉ có hiệu lực bên trong Kafka | only in force inside Kafka |
| hành động đó nằm ngoài transaction | that action sits outside the transaction |
| message sẽ được giao lại | the message will be delivered again |
| hiệu quả một lần | effectively-once |
| kết quả cuối cùng chỉ tính một lần | the final result only counts once |
| rồi tuyên bố rằng | and then declares that |
| chúng ta phải phản biện ngay | we have to push back straight away |
| có tham gia vào ... hay không | takes part in ... or not |
| chỉ cần một lần giao lại là | only one redelivery is enough for |
| khách bị tính tiền hai lần | a customer to be charged twice |
| không miễn cho chúng ta việc đó | do not excuse us from that |

**Thuật ngữ cần nhớ**

- hiệu quả một lần → **effectively-once**
- giao lại → **redelivery**
- khoá chống trùng → **a deduplication key**
- ghi kiểu chèn hoặc cập nhật → **an upsert**
- tác dụng phụ ra hệ thống ngoài → **an external side effect**

---

## ⑦ Partition nóng và lệch dữ liệu

**Tiếng Việt**

Lỗi vận hành thứ nhất là partition nóng, và nguyên nhân gần như luôn là khoá bị phân bố lệch. Nếu chúng ta chọn khoá theo mã khách hàng doanh nghiệp, một khách hàng lớn sẽ đẩy phần lớn lưu lượng vào đúng một partition. Khi đó một consumer làm việc quá tải còn các consumer khác gần như rảnh rỗi, và tổng thông lượng của nhóm bị chặn bởi cái chậm nhất. Chúng ta nhận ra triệu chứng này khá dễ, vì độ trễ tiêu thụ chỉ tăng ở một partition trong khi các partition khác vẫn sạch. Người ít kinh nghiệm hay kết luận nhầm rằng cluster thiếu tài nguyên rồi thêm consumer, nhưng consumer mới không giúp được gì vì partition nóng vẫn chỉ có đúng một người đọc.

Chúng ta có ba cách xử lý, và cả ba đều là đánh đổi chứ không có cách nào miễn phí. Cách thứ nhất là dùng khoá ghép, ví dụ ghép mã khách hàng với mã thực thể, để lưu lượng trải rộng hơn. Cách thứ hai là tách hẳn khách hàng lớn sang một topic riêng hoặc một cluster riêng, để họ không ảnh hưởng tới phần còn lại. Cách thứ ba là bỏ khoá đi và chấp nhận mất thứ tự, nếu nghiệp vụ thật sự không cần thứ tự. Điểm mà chúng ta phải nói ra là cả ba cách đều động tới thứ tự hoặc động tới chi phí vận hành, nên đây là một đánh đổi giữa thứ tự và cân bằng tải.

**English (bám cấu trúc tiếng Việt)**

The first operational failure is the hot partition, and the cause is almost always a key that is unevenly distributed. If we choose the key by the business customer id, one large customer will push most of the traffic into exactly one partition. At that point one consumer works while it is overloaded and the other consumers are nearly idle, and the total throughput of the group is capped by the slowest one. We recognise this symptom fairly easily, because consumer lag only rises on one partition while the other partitions stay clean. Less experienced people often conclude wrongly that the cluster lacks resources and then add consumers, but the new consumers cannot help because the hot partition still has exactly one reader.

We have three ways to handle it, and all three are trade-offs rather than anything free. The first way is to use a composite key, for example joining the customer id with the entity id, so that the traffic spreads out more widely. The second way is to separate the large customer completely into its own topic or its own cluster, so that they do not affect the rest. The third way is to drop the key and accept losing the ordering, if the business truly does not need the ordering. The point we must say out loud is that all three ways touch either the ordering or the operational cost, so this is a trade-off between ordering and load balancing.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| khoá bị phân bố lệch | a key that is unevenly distributed |
| đẩy phần lớn lưu lượng vào đúng một partition | push most of the traffic into exactly one partition |
| làm việc quá tải | works while it is overloaded |
| bị chặn bởi cái chậm nhất | is capped by the slowest one |
| chúng ta nhận ra triệu chứng này khá dễ | we recognise this symptom fairly easily |
| trong khi các partition khác vẫn sạch | while the other partitions stay clean |
| kết luận nhầm rằng | conclude wrongly that |
| cluster thiếu tài nguyên | the cluster lacks resources |
| khoá ghép | a composite key |
| tách hẳn ... sang một topic riêng | separate ... completely into its own topic |
| chấp nhận mất thứ tự | accept losing the ordering |
| cả ba cách đều động tới | all three ways touch |

**Thuật ngữ cần nhớ**

- partition nóng → **a hot partition**
- lệch dữ liệu → **data skew**
- khoá ghép → **a composite key**
- quá tải → **overloaded**
- cân bằng tải → **load balancing**

---

## ⑧ Consumer lag: nhịp tim của lớp messaging

**Tiếng Việt**

Lỗi vận hành thứ hai là độ trễ tiêu thụ, và chúng ta nên coi nó là nhịp tim của lớp messaging. Nó được tính bằng offset mới nhất trên partition trừ đi offset đã commit của nhóm, nên nó chính là số message còn chưa được xử lý. Nếu con số này dao động quanh một mức ổn định thì hệ thống đang theo kịp. Nếu nó tăng đều đặn thì consumer đang chậm hơn producer, và khoảng cách đó sẽ không bao giờ tự đóng lại. Vì vậy chúng ta phải đặt cảnh báo trên chỉ số này, chứ không chờ tới lúc người dùng báo rằng dữ liệu bị trễ.

Khi độ trễ tiêu thụ tăng, chúng ta xử lý theo thứ tự từ rẻ tới đắt. Trước hết chúng ta thêm consumer, nhưng cách này chỉ hiệu quả tới mức bằng số partition. Sau đó chúng ta tăng số partition, và lúc đó chúng ta phải nhớ lại bài toán ánh xạ khoá ở bài trước. Tiếp theo chúng ta tối ưu chính phần xử lý, ví dụ gom lô các lệnh ghi thay vì ghi từng bản một. Cuối cùng chúng ta đẩy những việc nặng và chậm sang một luồng bất đồng bộ khác, để vòng lặp tiêu thụ chính luôn ngắn.

**English (bám cấu trúc tiếng Việt)**

The second operational failure is consumer lag, and we should treat it as the heartbeat of the messaging layer. It is computed as the latest offset on the partition minus the committed offset of the group, so it is exactly the number of messages that are not processed yet. If this number swings around a stable level then the system is keeping up. If it rises steadily then the consumers are slower than the producers, and that gap will never close by itself. Therefore we have to set an alert on this metric, rather than waiting until users report that the data is late.

When consumer lag rises, we handle it in order from cheap to expensive. First of all we add consumers, but this way is only effective up to the level of the partition count. After that we raise the number of partitions, and at that point we have to recall the key mapping problem from the previous lesson. Next we optimise the processing part itself, for example batching the write statements instead of writing one record at a time. Finally we push the heavy and slow jobs over to another asynchronous flow, so that the main consumption loop always stays short.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| nhịp tim của lớp messaging | the heartbeat of the messaging layer |
| offset mới nhất trừ đi offset đã commit | the latest offset minus the committed offset |
| dao động quanh một mức ổn định | swings around a stable level |
| hệ thống đang theo kịp | the system is keeping up |
| nếu nó tăng đều đặn | if it rises steadily |
| khoảng cách đó sẽ không bao giờ tự đóng lại | that gap will never close by itself |
| đặt cảnh báo trên chỉ số này | set an alert on this metric |
| chứ không chờ tới lúc người dùng báo | rather than waiting until users report |
| theo thứ tự từ rẻ tới đắt | in order from cheap to expensive |
| chỉ hiệu quả tới mức bằng số partition | only effective up to the level of the partition count |
| gom lô các lệnh ghi | batching the write statements |
| vòng lặp tiêu thụ chính luôn ngắn | the main consumption loop always stays short |

**Thuật ngữ cần nhớ**

- độ trễ tiêu thụ → **consumer lag**
- chỉ số theo dõi → **a metric**
- cảnh báo → **an alert**
- theo kịp → **to keep up**
- gom lô → **to batch**

---

## ⑨ Mô hình ghi nhớ

**Tiếng Việt**

Độ bền bằng ba bản sao, hai bản đồng bộ tối thiểu và `acks=all`, trong đó ISR đóng vai người gác cổng; rebalance đang chuyển từ dừng cả thế giới sang chuyển dần. Exactly-once chỉ có hiệu lực bên trong Kafka, nên ra ngoài chúng ta vẫn phải tự chống trùng, và độ trễ tiêu thụ chính là nhịp tim mà chúng ta phải theo dõi.

**English (bám cấu trúc tiếng Việt)**

Durability equals three replicas, a minimum of two in-sync replicas and `acks=all`, in which the ISR plays the role of the gatekeeper; rebalancing is moving from stop-the-world to incremental. Exactly-once is only in force inside Kafka, so beyond that boundary we still have to prevent duplicates ourselves, and consumer lag is exactly the heartbeat that we have to watch.

---

## ⑩ Bảng thuật ngữ tổng hợp

| Tiếng Việt | Tiếng Anh | Ghi chú phát âm |
|---|---|---|
| nhân bản | replication | rep-li-**KEY**-shn; *replica* = **REP**-li-ka, trọng âm âm đầu |
| bản chính / bản theo | leader / follower | *leader* /ˈliːdə/ — "LI-đơ", nguyên âm dài |
| các bản sao đang đồng bộ | in-sync replicas (ISR) | |
| hệ số nhân bản | replication factor | |
| bầu leader mới | leader election | i-**LEK**-shn, chữ đầu đọc /ɪ/ nhẹ |
| đủ điều kiện | eligible | **E**-li-ji-bl, chữ *g* đọc /dʒ/ |
| người gác cổng | gatekeeper | |
| ngưỡng tối thiểu | minimum threshold | *threshold* — âm *th* /θ/, đọc "THRESH-hold" |
| từ chối một lần ghi | to reject a write | *write* — chữ **w** câm hoàn toàn |
| xác nhận đã ghi | acknowledged | ək-**NO**-lijd, không tách rời phần *know* |
| khả năng sẵn sàng | availability | ə-vay-lə-**BI**-li-ty, sáu âm tiết |
| độ bền | durability | du-ra-**BI**-li-ty, trọng âm âm thứ ba |
| chia lại partition | rebalance | ri-**BA**-lance, trọng âm âm giữa |
| bị thu hồi | to be revoked | ri-**VOUKD**, đuôi /kt/ bật nhanh |
| dừng cả thế giới | stop-the-world | |
| hợp tác / chuyển dần | cooperative / incremental | *cooperative* = co-**O**-pə-rə-tiv, năm âm tiết |
| triển khai | to deploy | di-**PLOI**, trọng âm âm sau |
| vòng đời dữ liệu | data lifecycle | giọng Anh *data* /ˈdeɪtə/ — "ĐÂY-tơ" |
| chính sách dọn dẹp | cleanup policy | |
| giữ theo thời hạn | retention | ri-**TEN**-shn |
| nén theo khoá | log compaction | kəm-**PAK**-shn, trọng âm âm giữa |
| ảnh chụp trạng thái | snapshot | |
| sổ cái | ledger | **LE**-jer, chữ *g* đọc /dʒ/ |
| đối soát | to reconcile | **RE**-con-cile, trọng âm âm đầu, đuôi đọc /saɪl/ |
| producer bất biến khi lặp | idempotent producer | i-**DEM**-po-tent, trọng âm âm thứ hai |
| số thứ tự chống trùng | sequence number | **SEE**-kwəns, trọng âm âm đầu |
| giao dịch nguyên tử | atomic transaction | *atomic* = ə-**TOM**-ic; *transaction* = tran-**ZAK**-shn, chữ *s* đọc /z/ |
| bản trùng | a duplicate | danh từ = **DU**-pli-kət; động từ = **DU**-pli-kate |
| đọc, xử lý rồi ghi | read-process-write | |
| hiệu quả một lần | effectively-once | |
| giao lại | redelivery | ri-di-**LI**-və-ri |
| khoá chống trùng | deduplication key | di-du-pli-**KEY**-shn, năm âm tiết |
| ghi chèn hoặc cập nhật | an upsert | **UP**-sert, ghép từ *update* và *insert* |
| tác dụng phụ ra ngoài | an external side effect | *external* = iks-**TER**-nal |
| partition nóng | hot partition | *partition* = par-**TI**-shn |
| lệch dữ liệu | data skew | *skew* /skjuː/ — đọc như chữ **Q** thêm /sk/ ở đầu |
| khoá ghép | composite key | **COM**-po-sit, giọng Anh trọng âm âm đầu |
| quá tải | overloaded | |
| khách hàng doanh nghiệp trong hệ đa khách | a tenant | **TEN**-ənt, âm đầu ngắn, **không** đọc "TI-nant" |
| cân bằng tải | load balancing | |
| độ trễ tiêu thụ | consumer lag | |
| chỉ số theo dõi | a metric | **ME**-tric |
| cảnh báo | an alert | ə-**LERT**, trọng âm âm sau |
| theo kịp | to keep up | |
| gom lô | to batch | /bætʃ/ — nguyên âm ngắn, khác hẳn *beach* |

---

## ⑪ Bài nói

> **Cách luyện:** bật ghi âm, nói **60–90 giây** cho mỗi đề, **không nhìn tài liệu**. Mỗi bài nói phải dùng ít nhất **sáu thuật ngữ** trong bảng trên. Nói xong thì nghe lại một lượt, đánh dấu chỗ mình ngập ngừng, rồi nói lại chính đề đó thêm một lần nữa.

1. Explain to a junior developer what the ISR is, and describe step by step what happens inside the cluster when the leader of a partition dies.

2. A developer turns on Kafka transactions and tells the team that writes to the order database are now exactly-once, so the deduplication logic in the consumer can be removed. Explain what is wrong with that.

3. Someone proposes setting `acks` to one and dropping `min.insync.replicas`, because the current setup makes the checkout endpoint slower. Explain why you would push back, and what you would offer instead.

4. Describe the difference between retention and compaction, and describe how you would configure a ledger topic and an account-balance topic differently.

5. Consumer lag on one partition has been climbing all week while the other partitions are flat. Describe how you would diagnose that and what you suspect the root cause is.

6. When would you choose a composite key over a plain tenant id, and what do you give up when you make that change?

7. Describe what a rebalance does to a consumer group during a rolling deployment, and explain how the newer cooperative protocol changes that picture.
