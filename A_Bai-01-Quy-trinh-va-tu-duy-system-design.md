# Bài 1 — Quy trình & tư duy làm system design
### Bản song ngữ: tiếng Anh bám cấu trúc tiếng Việt

> **Cách dùng:** đọc đoạn tiếng Việt trước → tự dịch trong đầu → so với bản tiếng Anh bên dưới → **đọc to bản tiếng Anh hai lần**. Lần một đọc chậm cho đúng từ, lần hai đọc liền mạch như đang nói với người phỏng vấn.

---

## Phần 1 — Vì sao quy trình được chấm riêng, độc lập với kiến thức

**Tiếng Việt**

Một bài system design giống như việc chúng ta được giao xây một thành phố trong bốn mươi lăm phút. Người yếu lao ngay vào vẽ từng toà nhà, vì họ nghĩ rằng bản vẽ càng nhiều hộp thì càng chứng tỏ được năng lực. Người giỏi hỏi trước: thành phố này dành cho bao nhiêu dân, nằm ở đâu, và ưu tiên giao thông hay nhà ở. Những câu hỏi làm rõ đó định hình toàn bộ bản vẽ về sau. Vì lý do này, người phỏng vấn chấm quy trình một cách độc lập với kiến thức. Nếu chúng ta bỏ qua quy trình, chúng ta vẫn có thể trượt dù biết rất nhiều kỹ thuật. Ngược lại, một người có kiến thức vừa phải nhưng chạy đúng khung sáu bước thường được đánh giá cao hơn.

**English (bám cấu trúc tiếng Việt)**

A system design question is like being asked to build a city in forty-five minutes. A weak candidate rushes straight into drawing each building, because they think that the more boxes the drawing has, the more it proves their ability. A strong candidate asks first: how many people is this city for, where does it sit, and does it prioritise transport or housing. Those clarifying questions shape the whole drawing that comes after. For this reason, the interviewer scores the process independently of the knowledge. If we skip the process, we can still fail even though we know a lot of techniques. On the other hand, a person with moderate knowledge but who runs the six-step framework correctly is usually rated higher.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| chúng ta được giao xây một thành phố | we are asked to build a city |
| lao ngay vào vẽ từng toà nhà | rushes straight into drawing each building |
| càng nhiều hộp thì càng chứng tỏ được năng lực | the more boxes it has, the more it proves their ability |
| ưu tiên giao thông hay nhà ở | prioritise transport or housing |
| định hình toàn bộ bản vẽ về sau | shape the whole drawing that comes after |
| chấm quy trình một cách độc lập với kiến thức | score the process independently of the knowledge |
| dù biết rất nhiều kỹ thuật | even though we know a lot of techniques |
| chạy đúng khung sáu bước | runs the six-step framework correctly |
| được đánh giá cao hơn | is rated higher |

**Thuật ngữ cần nhớ**

- quy trình → **the process**
- ứng viên → **a candidate**
- người phỏng vấn → **the interviewer**
- khung sáu bước → **the six-step framework**
- câu hỏi làm rõ → **a clarifying question**

---

## Phần 2 — Bước một: làm rõ yêu cầu, và vì sao chốt phi chức năng trước

**Tiếng Việt**

Bước đầu tiên trong khung là làm rõ yêu cầu, và chúng ta chia yêu cầu thành hai nhóm. Nhóm thứ nhất là yêu cầu chức năng, tức là hệ thống làm gì, ví dụ đăng bài, theo dõi người khác, hoặc chuyển hướng một đường dẫn ngắn. Nhóm thứ hai là yêu cầu phi chức năng, gồm năm trục: quy mô, độ trễ, độ sẵn sàng, tính nhất quán và độ bền dữ liệu. Rất nhiều người chỉ hỏi nhóm thứ nhất rồi vẽ ngay, và đó là sai lầm phổ biến nhất ở vòng này. Chúng ta phải chốt nhóm phi chức năng trước, vì chính nhóm này định hình kiến trúc.

Nếu bài toán cần tính nhất quán mạnh, thiết kế sẽ khác hẳn so với khi chúng ta chấp nhận tính nhất quán cuối cùng. Một hệ thống tiền bạc hoặc tồn kho buộc phải nhất quán mạnh, vì vậy chúng ta không được đặt một hàng đợi bất đồng bộ vào giữa luồng trừ tiền. Ngược lại, một bảng tin mạng xã hội chấp nhận trễ vài giây, vì vậy chúng ta được phép đẩy việc nặng ra phía sau. Nếu chúng ta bỏ qua bước chốt này, hậu quả là chúng ta vẽ một hệ thống cho sai quy mô và sai mức nhất quán, rồi phải vẽ lại từ đầu khi người phỏng vấn nói ra con số thật. Chúng ta cũng nên hỏi rõ cái gì nằm ngoài phạm vi, vì phần nằm ngoài phạm vi bảo vệ chúng ta khỏi việc thiết kế thừa.

**English (bám cấu trúc tiếng Việt)**

The first step in the framework is to clarify the requirements, and we split the requirements into two groups. The first group is the functional requirements, that is, what the system does, for example posting, following other people, or redirecting a short link. The second group is the non-functional requirements, which cover five axes: scale, latency, availability, consistency and durability. Many people ask only about the first group and then start drawing, and that is the most common mistake in this round. We must settle the non-functional group first, because it is this group that shapes the architecture.

If the problem needs strong consistency, the design will be completely different from when we accept eventual consistency. A money system or an inventory system is forced to be strongly consistent, therefore we must not put an asynchronous queue in the middle of the payment flow. On the other hand, a social feed accepts a delay of a few seconds, therefore we are allowed to push the heavy work to the back. If we skip this settling step, the consequence is that we draw a system for the wrong scale and the wrong level of consistency, and then we have to draw again from the start when the interviewer says the real numbers out loud. We should also ask clearly what is out of scope, because the out-of-scope part protects us from over-engineering.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| chia yêu cầu thành hai nhóm | split the requirements into two groups |
| tức là hệ thống làm gì | that is, what the system does |
| gồm năm trục | cover five axes |
| rồi vẽ ngay | and then start drawing |
| chốt nhóm phi chức năng trước | settle the non-functional group first |
| chính nhóm này định hình kiến trúc | it is this group that shapes the architecture |
| khác hẳn so với khi | completely different from when |
| buộc phải nhất quán mạnh | is forced to be strongly consistent |
| đẩy việc nặng ra phía sau | push the heavy work to the back |
| phải vẽ lại từ đầu | have to draw again from the start |
| cái gì nằm ngoài phạm vi | what is out of scope |

**Thuật ngữ cần nhớ**

- yêu cầu chức năng → **functional requirements**
- yêu cầu phi chức năng → **non-functional requirements**
- tính nhất quán mạnh / cuối cùng → **strong consistency** / **eventual consistency**
- độ bền dữ liệu → **durability**
- nằm ngoài phạm vi → **out of scope** / **non-goals**
- thiết kế thừa → **over-engineering**

---

## Phần 3 — Bước hai: ước lượng, nhưng chỉ ước lượng con số dẫn tới quyết định

**Tiếng Việt**

Bước thứ hai là ước lượng nhanh trên giấy nháp, và chúng ta đi từ số người dùng hoạt động hằng ngày ra số truy vấn mỗi giây, rồi ra dung lượng lưu trữ và băng thông. Mục đích của bước này không phải là ra con số chính xác, mà là ra đúng bậc độ lớn để biện luận. Một triệu hay một tỷ là hai thế giới khác nhau, và chúng dẫn tới hai kiến trúc khác nhau. Chúng ta chỉ nên ước lượng những con số thật sự đổi một quyết định thiết kế. Ví dụ, tỷ lệ đọc trên ghi đổi quyết định về cache, còn số byte của một trường tên người dùng thì không đổi gì cả.

Nếu chúng ta ước lượng mọi thứ cho đủ bài, hậu quả là chúng ta đốt mất năm phút quý giá vào những con số trang trí. Người phỏng vấn sẽ thấy chúng ta bận rộn nhưng không tiến về phía trước. Cách nói an toàn là chúng ta nêu giả định thành tiếng, làm tròn số cho dễ tính, rồi nói ngay con số này dẫn tới quyết định gì. Nhờ đó, mỗi phép tính đều có một câu kết luận đi kèm, và buổi phỏng vấn vẫn chạy đúng nhịp.

**English (bám cấu trúc tiếng Việt)**

The second step is a quick estimate on scratch paper, and we go from the number of daily active users to the number of queries per second, then to the storage size and the bandwidth. The purpose of this step is not to produce an exact number, but to produce the right order of magnitude to reason with. One million or one billion are two different worlds, and they lead to two different architectures. We should only estimate the numbers that really change a design decision. For example, the read-to-write ratio changes the decision about the cache, while the number of bytes in a username field does not change anything at all.

If we estimate everything just to fill the answer, the consequence is that we burn five precious minutes on decorative numbers. The interviewer will see that we are busy but not moving forward. The safe way to speak is that we state our assumptions out loud, round the numbers so they are easy to compute, and then say straight away what decision this number leads to. Thanks to that, every calculation comes with a concluding sentence, and the interview still runs at the right pace.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| ước lượng nhanh trên giấy nháp | a quick estimate on scratch paper |
| ra đúng bậc độ lớn để biện luận | produce the right order of magnitude to reason with |
| là hai thế giới khác nhau | are two different worlds |
| những con số thật sự đổi một quyết định thiết kế | the numbers that really change a design decision |
| thì không đổi gì cả | does not change anything at all |
| ước lượng mọi thứ cho đủ bài | estimate everything just to fill the answer |
| đốt mất năm phút quý giá | burn five precious minutes |
| những con số trang trí | decorative numbers |
| bận rộn nhưng không tiến về phía trước | busy but not moving forward |
| nêu giả định thành tiếng | state our assumptions out loud |
| chạy đúng nhịp | runs at the right pace |

**Thuật ngữ cần nhớ**

- ước lượng nhanh → **back-of-envelope estimation**
- người dùng hoạt động hằng ngày → **daily active users (DAU)**
- số truy vấn mỗi giây → **queries per second (QPS)**
- bậc độ lớn → **order of magnitude**
- tỷ lệ đọc trên ghi → **read-to-write ratio**
- giả định → **an assumption**

---

## Phần 4 — Bước ba và bước bốn: thiết kế tổng thể, rồi hợp đồng API

**Tiếng Việt**

Bước thứ ba là vẽ thiết kế tổng thể, tức là các khối lớn nối với nhau theo thứ tự client, load balancer, service, cache, database và queue. Ở bước này chúng ta chưa đi vào bên trong từng khối, mà chỉ đặt tên cho luồng dữ liệu và nói rõ dữ liệu chảy theo hướng nào. Chúng ta nên mô tả ít nhất một luồng đọc và một luồng ghi từ đầu đến cuối, vì hai luồng này làm lộ ra hầu hết vấn đề về sau. Nếu chúng ta vẽ hộp mà không kể được luồng, người phỏng vấn sẽ hiểu là chúng ta đang nhớ lại một sơ đồ chứ không đang thiết kế.

Bước thứ tư là định nghĩa hợp đồng API và mô hình dữ liệu, và bước này nằm sau thiết kế tổng thể nhưng trước phần mở rộng quy mô. Chúng ta chỉ cần vài endpoint chính cùng với vài bảng cốt lõi, không cần đầy đủ. Mục đích là neo thiết kế xuống mặt đất, vì khi đã có tên endpoint và tên trường, mọi câu nói sau đó đều có chỗ bám. Nếu chúng ta bỏ bước này, hậu quả là phần còn lại của buổi phỏng vấn sẽ trôi nổi ở mức chung chung, và người phỏng vấn không kiểm tra được là chúng ta có thật sự nghĩ đến dữ liệu hay không.

**English (bám cấu trúc tiếng Việt)**

The third step is to draw the high-level design, that is, the big blocks connected together in the order client, load balancer, service, cache, database and queue. At this step we do not go inside each block yet, we only name the data flows and say clearly which direction the data moves. We should describe at least one read path and one write path from end to end, because these two paths expose most of the problems that come later. If we draw boxes but cannot tell the flow, the interviewer will understand that we are recalling a diagram rather than designing.

The fourth step is to define the API contract and the data model, and this step sits after the high-level design but before the scaling part. We only need a few main endpoints together with a few core tables, we do not need a complete set. The purpose is to anchor the design to the ground, because once we have endpoint names and field names, every sentence after that has something to hold on to. If we drop this step, the consequence is that the rest of the interview will float at a general level, and the interviewer cannot check whether we have really thought about the data or not.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| các khối lớn nối với nhau | the big blocks connected together |
| chưa đi vào bên trong từng khối | do not go inside each block yet |
| đặt tên cho luồng dữ liệu | name the data flows |
| dữ liệu chảy theo hướng nào | which direction the data moves |
| làm lộ ra hầu hết vấn đề về sau | expose most of the problems that come later |
| đang nhớ lại một sơ đồ chứ không đang thiết kế | recalling a diagram rather than designing |
| nằm sau … nhưng trước … | sits after … but before … |
| neo thiết kế xuống mặt đất | anchor the design to the ground |
| mọi câu nói sau đó đều có chỗ bám | every sentence after that has something to hold on to |
| trôi nổi ở mức chung chung | float at a general level |

**Thuật ngữ cần nhớ**

- thiết kế tổng thể → **high-level design**
- luồng đọc / luồng ghi → **the read path** / **the write path**
- hợp đồng API → **the API contract**
- mô hình dữ liệu → **the data model**
- lược đồ bảng → **the schema**

---

## Phần 5 — Bước năm và bước sáu: đào sâu, rồi tự chỉ ra điểm nghẽn và đánh đổi

**Tiếng Việt**

Bước thứ năm là mở rộng quy mô và đào sâu, và chúng ta chỉ chọn một hoặc hai phần để đào, chứ không đào tất cả. Những phần đáng đào thường là mở rộng cơ sở dữ liệu, tầng cache, hoặc xử lý khoá nóng. Chúng ta nên tự chọn phần đào sâu thay vì chờ người phỏng vấn chỉ định, vì việc tự chọn cho thấy chúng ta biết đâu là chỗ rủi ro nhất. Nếu chúng ta cố đào mọi phần, hậu quả là mọi phần đều nông và không phần nào cho thấy chiều sâu kỹ thuật.

Bước thứ sáu là tự chỉ ra điểm nghẽn và nêu đánh đổi của lựa chọn mình vừa đưa ra. Chúng ta nói thẳng rằng thiết kế này sẽ gãy ở đâu trước, và nói cái giá phải trả cho mỗi quyết định. Ví dụ, chúng ta thêm cache để giảm độ trễ, nhưng đổi lại chúng ta phải sống chung với dữ liệu cũ trong vài giây. Người phỏng vấn chờ đúng câu này, vì một kỹ sư senior là người biết thiết kế của mình sai ở đâu. Nếu chúng ta không nêu đánh đổi, hậu quả là chúng ta bị xếp vào nhóm chỉ biết ghép công nghệ, và điểm ở vòng này sẽ dừng ở mức trung cấp.

**English (bám cấu trúc tiếng Việt)**

The fifth step is to scale and to deep-dive, and we pick only one or two parts to dig into, rather than digging into all of them. The parts worth digging into are usually database scaling, the cache layer, or handling a hot key. We should pick the deep-dive part ourselves instead of waiting for the interviewer to assign it, because picking it ourselves shows that we know where the riskiest place is. If we try to dig into every part, the consequence is that every part stays shallow and no part shows any technical depth.

The sixth step is to point out the bottleneck ourselves and to state the trade-off of the choice we have just made. We say straight out where this design will break first, and we say the price we pay for each decision. For example, we add a cache to cut latency, but in exchange we have to live with stale data for a few seconds. The interviewer is waiting for exactly this sentence, because a senior engineer is someone who knows where their own design is wrong. If we do not state the trade-off, the consequence is that we are put into the group who only bolt technologies together, and our score in this round will stop at the mid-level mark.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| chỉ chọn một hoặc hai phần để đào | pick only one or two parts to dig into |
| những phần đáng đào | the parts worth digging into |
| xử lý khoá nóng | handling a hot key |
| chờ người phỏng vấn chỉ định | waiting for the interviewer to assign it |
| đâu là chỗ rủi ro nhất | where the riskiest place is |
| mọi phần đều nông | every part stays shallow |
| thiết kế này sẽ gãy ở đâu trước | where this design will break first |
| cái giá phải trả cho mỗi quyết định | the price we pay for each decision |
| đổi lại chúng ta phải sống chung với dữ liệu cũ | in exchange we have to live with stale data |
| chỉ biết ghép công nghệ | who only bolt technologies together |

**Thuật ngữ cần nhớ**

- đào sâu → **deep-dive**
- điểm nghẽn → **the bottleneck**
- đánh đổi → **a trade-off**
- dữ liệu cũ → **stale data**
- khoá nóng → **a hot key**

---

## Phần 6 — Phân bổ thời gian trong bốn mươi lăm phút

**Tiếng Việt**

Buổi phỏng vấn thường dài bốn mươi lăm phút, và chúng ta nên chia thời gian theo một tỷ lệ cố định để khỏi phải nghĩ trong lúc căng thẳng. Chúng ta dành khoảng năm phút cho phần làm rõ yêu cầu, khoảng năm phút cho phần ước lượng, khoảng mười phút cho thiết kế tổng thể cùng với API, khoảng mười lăm đến hai mươi phút cho một hoặc hai phần đào sâu, và khoảng năm phút cuối cho điểm nghẽn và đánh đổi. Nguyên tắc quan trọng nhất là chúng ta không được sa lầy vào một phần. Nếu một phần bắt đầu kéo dài quá lâu, chúng ta nói thành tiếng rằng chúng ta sẽ quay lại phần này nếu còn thời gian, rồi đi tiếp.

Việc quản lý thời gian là một kỹ năng được chấm, chứ không phải là chuyện phụ. Một buổi phỏng vấn hết giờ khi chúng ta còn đang vẽ hộp sẽ bị đánh giá là không hoàn thành, dù phần đã vẽ rất tốt. Nếu chúng ta chừa ra năm phút cuối cho đánh đổi, chúng ta luôn kết thúc bằng phần ghi điểm cao nhất. Vì vậy, chúng ta nên liếc đồng hồ ở phút thứ mười lăm và ở phút thứ ba mươi để tự kiểm tra nhịp độ.

**English (bám cấu trúc tiếng Việt)**

An interview usually lasts forty-five minutes, and we should split the time by a fixed ratio so that we do not have to think while we are under pressure. We give about five minutes to clarifying the requirements, about five minutes to the estimation, about ten minutes to the high-level design together with the API, about fifteen to twenty minutes to one or two deep-dive parts, and about five final minutes to the bottleneck and the trade-offs. The most important rule is that we must not get stuck in one part. If one part starts to drag on for too long, we say out loud that we will come back to this part if there is time left, and then we move on.

Managing time is a skill that is scored, not a side matter. An interview that runs out of time while we are still drawing boxes will be judged as unfinished, even though the part we drew was very good. If we save five final minutes for the trade-offs, we always end with the part that scores the highest. Therefore, we should glance at the clock at minute fifteen and at minute thirty to check our own pace.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| chia thời gian theo một tỷ lệ cố định | split the time by a fixed ratio |
| để khỏi phải nghĩ trong lúc căng thẳng | so that we do not have to think while under pressure |
| chúng ta dành khoảng năm phút cho | we give about five minutes to |
| không được sa lầy vào một phần | must not get stuck in one part |
| bắt đầu kéo dài quá lâu | starts to drag on for too long |
| quay lại phần này nếu còn thời gian | come back to this part if there is time left |
| là một kỹ năng được chấm | is a skill that is scored |
| bị đánh giá là không hoàn thành | will be judged as unfinished |
| chừa ra năm phút cuối | save five final minutes |
| liếc đồng hồ | glance at the clock |

**Thuật ngữ cần nhớ**

- phân bổ thời gian → **time allocation**
- ngân sách thời gian → **a time budget**
- sa lầy → **to get stuck** / **to get bogged down**
- nhịp độ → **the pace**
- hết giờ → **to run out of time**

---

## Phần 7 — Dẫn dắt buổi phỏng vấn và nói to suy nghĩ

**Tiếng Việt**

Có một nhóm ứng viên biết rất nhiều nhưng vẫn trượt vòng này, và lý do gần như luôn nằm ở quy trình chứ không nằm ở kiến thức. Họ nhảy vào chi tiết quá sớm, họ không hỏi yêu cầu, họ im lặng suy nghĩ rồi mới nói ra kết quả, họ không nêu đánh đổi, và họ thiết kế thừa để khoe kiến thức. Người phỏng vấn chấm quá trình tư duy, chứ không chỉ chấm kết quả cuối cùng. Vì vậy, khi chúng ta im lặng, người phỏng vấn không có gì để chấm cả. Chúng ta phải nói to suy nghĩ của mình trong suốt buổi, kể cả khi chúng ta đang phân vân giữa hai lựa chọn.

Điều thứ hai, và cũng là điều phân biệt cấp senior, là chúng ta phải dẫn dắt buổi phỏng vấn. Dẫn dắt nghĩa là chúng ta chủ động đề xuất phạm vi, nói giả định thành tiếng, tự chọn phần đào sâu, và tự chỉ ra điểm nghẽn. Nếu chúng ta ngồi chờ người phỏng vấn hỏi rồi mới trả lời, chúng ta bị trừ điểm vì thiếu tinh thần làm chủ. Có một tình huống bẫy hay gặp: người phỏng vấn ném đề ra rồi im lặng chờ. Sự im lặng đó chính là bài kiểm tra xem chúng ta có dẫn dắt được hay không, nên phản ứng đúng là đặt câu hỏi làm rõ và nói giả định, chứ không phải lao vào vẽ.

**English (bám cấu trúc tiếng Việt)**

There is a group of candidates who know a lot but still fail this round, and the reason almost always lies in the process rather than in the knowledge. They jump into details too early, they do not ask about the requirements, they think in silence and only then say the result, they do not state trade-offs, and they over-engineer in order to show off their knowledge. The interviewer scores the thinking process, not only the final result. Therefore, when we stay silent, the interviewer has nothing to score at all. We must think out loud throughout the session, even when we are hesitating between two options.

The second thing, and also the thing that marks the senior level, is that we must drive the interview. Driving means that we actively propose the scope, say our assumptions out loud, pick the deep-dive part ourselves, and point out the bottleneck ourselves. If we sit and wait for the interviewer to ask before we answer, we lose points for a lack of ownership. There is a common trap situation: the interviewer throws out the question and then waits in silence. That silence is exactly the test of whether we can drive or not, so the right reaction is to ask clarifying questions and state assumptions, not to rush into drawing.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| lý do gần như luôn nằm ở | the reason almost always lies in |
| nhảy vào chi tiết quá sớm | jump into details too early |
| im lặng suy nghĩ rồi mới nói ra kết quả | think in silence and only then say the result |
| thiết kế thừa để khoe kiến thức | over-engineer in order to show off their knowledge |
| không có gì để chấm cả | has nothing to score at all |
| đang phân vân giữa hai lựa chọn | are hesitating between two options |
| điều phân biệt cấp senior | the thing that marks the senior level |
| chủ động đề xuất phạm vi | actively propose the scope |
| bị trừ điểm vì thiếu tinh thần làm chủ | lose points for a lack of ownership |
| ném đề ra rồi im lặng chờ | throws out the question and then waits in silence |
| lao vào vẽ | rush into drawing |

**Thuật ngữ cần nhớ**

- nói to suy nghĩ → **to think out loud**
- dẫn dắt buổi phỏng vấn → **to drive the interview**
- tinh thần làm chủ → **ownership**
- phạm vi → **the scope**
- thiết kế thừa → **to over-engineer**

---

## Phần 8 — Khung này dùng thật ở công ty, không chỉ trong phỏng vấn

**Tiếng Việt**

Khung sáu bước không chỉ phục vụ phỏng vấn, mà chính là cách chúng ta viết một tài liệu thiết kế thật ở công ty. Một tài liệu thiết kế chuẩn đi theo thứ tự: bối cảnh và yêu cầu, mục tiêu và những thứ không nằm trong mục tiêu, thiết kế đề xuất, các phương án đã cân nhắc, các đánh đổi, và kế hoạch triển khai. Phần các phương án đã cân nhắc cùng với phần đánh đổi chính là bước sáu ở trên, chỉ khác là được viết ra giấy thay vì nói miệng. Nhờ đó, mỗi lần chúng ta luyện khung này cho phỏng vấn, chúng ta cũng đang luyện kỹ năng viết tài liệu hằng ngày.

Các công ty lớn đều dùng một biến thể của khuôn mẫu này, ví dụ tài liệu thiết kế của Google hoặc cách làm ngược từ thông cáo báo chí của Amazon. Điểm chung của các khuôn mẫu đó là chúng bắt buộc người viết nêu những thứ không nằm trong mục tiêu và nêu các phương án thay thế. Đây đúng là tinh thần làm rõ phạm vi và nêu đánh đổi mà chúng ta vừa học. Nếu trong phỏng vấn chúng ta nói được rằng chúng ta cũng viết tài liệu theo cách này ở công ty, câu trả lời của chúng ta lập tức có sức nặng của kinh nghiệm thật.

**English (bám cấu trúc tiếng Việt)**

The six-step framework does not only serve the interview, it is exactly how we write a real design document at a company. A standard design document follows this order: context and requirements, goals and non-goals, the proposed design, the alternatives considered, the trade-offs, and the rollout plan. The alternatives-considered part together with the trade-offs part is exactly step six above, the only difference is that it is written on paper instead of said out loud. Thanks to that, every time we practise this framework for the interview, we are also practising our everyday document-writing skill.

Large companies all use a variant of this template, for example Google's design docs or Amazon's working-backwards approach from a press release. What those templates have in common is that they force the writer to state the non-goals and to state the alternatives. This is exactly the spirit of clarifying the scope and stating the trade-offs that we have just learned. If in the interview we can say that we also write documents this way at our company, our answer immediately carries the weight of real experience.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| không chỉ phục vụ phỏng vấn | does not only serve the interview |
| những thứ không nằm trong mục tiêu | non-goals |
| các phương án đã cân nhắc | the alternatives considered |
| kế hoạch triển khai | the rollout plan |
| chỉ khác là được viết ra giấy thay vì nói miệng | the only difference is that it is written on paper instead of said out loud |
| một biến thể của khuôn mẫu này | a variant of this template |
| điểm chung của các khuôn mẫu đó | what those templates have in common |
| bắt buộc người viết nêu | force the writer to state |
| có sức nặng của kinh nghiệm thật | carries the weight of real experience |

**Thuật ngữ cần nhớ**

- tài liệu thiết kế → **a design document** / **a design doc**
- mục tiêu và phi mục tiêu → **goals and non-goals**
- phương án thay thế → **an alternative**
- kế hoạch triển khai → **the rollout plan**
- khuôn mẫu → **a template**

---

## Mô hình ghi nhớ

**Tiếng Việt**

Làm rõ, ước lượng, phác thảo, chốt hợp đồng, mở rộng, rồi đánh đổi. Chúng ta hỏi trước khi vẽ, vì chính các yêu cầu phi chức năng mới vẽ ra kiến trúc.

**English (bám cấu trúc tiếng Việt)**

Clarify, estimate, sketch, contract, scale, then trade-off. We ask before we draw, because it is the non-functional requirements that draw the architecture.

---

## Bảng thuật ngữ tổng hợp

| Tiếng Việt | Tiếng Anh | Ghi chú phát âm |
|---|---|---|
| quy trình | process | Anh đọc /ˈprəʊses/ — "PRÂU-sess", không phải "PRA-sess" như Mỹ |
| kiến trúc | architecture | /ˈɑːkɪtektʃə/ — "A-ki-tếch-chờ", trọng âm **đầu**, chữ *ch* đầu đọc là /k/ |
| yêu cầu | requirement | /rɪˈkwaɪəmənt/ — "ri-KOAI-ơ-mần", trọng âm **âm thứ hai** |
| yêu cầu chức năng | functional requirements | |
| yêu cầu phi chức năng | non-functional requirements | |
| quy mô | scale | |
| độ trễ | latency | /ˈleɪtənsi/ — "LÂY-tân-si", âm đầu là "lây" không phải "la" |
| độ sẵn sàng | availability | a-vây-lơ-**BI**-li-ti, trọng âm rơi vào âm thứ **tư** |
| tính nhất quán | consistency | cần-**SIS**-tần-si, trọng âm âm thứ hai |
| độ bền dữ liệu | durability | Anh /ˌdjʊərəˈbɪləti/ — "diu-ơ-rơ-**BI**-li-ti", đầu là "dyoo" không phải "đu" |
| ước lượng nhanh | back-of-envelope estimation | *estimate* động từ /ˈestɪmeɪt/ đọc "-mêit", danh từ /ˈestɪmət/ đọc "-mợt" |
| bậc độ lớn | order of magnitude | |
| tỷ lệ đọc trên ghi | read-to-write ratio | *ratio* /ˈreɪʃiəʊ/ — "RÂY-shi-âu", không đọc "ra-ti-ô" |
| người dùng hoạt động hằng ngày | daily active users (DAU) | |
| số truy vấn mỗi giây | queries per second (QPS) | *query* /ˈkwɪəri/ — "KUY-ơ-ri" |
| hàng đợi | queue | /kjuː/ — đọc đúng như chữ "Q", bốn chữ cái cuối câm |
| bất đồng bộ | asynchronous | ây-**SING**-krơ-nợs, trọng âm âm thứ hai |
| thiết kế tổng thể | high-level design | |
| luồng đọc / luồng ghi | read path / write path | *path* có âm **th** cuối, lưỡi chạm răng, không thành "pát" |
| hợp đồng API | API contract | |
| lược đồ | schema | /ˈskiːmə/ — "SKI-mơ", không đọc "sê-ma" hay "sờ-che-ma" |
| điểm cuối | endpoint | |
| đào sâu | deep-dive | |
| điểm nghẽn | bottleneck | |
| đánh đổi | trade-off | |
| dữ liệu cũ | stale data | *stale* /steɪl/ — "stêi-l", có âm **st** đầu rõ |
| khoá nóng | hot key | |
| thông lượng | throughput | âm **th** đầu /θ/, không đọc thành "trú"; "THRU-put" |
| ngưỡng | threshold | âm **th** đầu, "THRESH-hâuld" |
| tuyến đường / định tuyến | route | Anh đọc /ruːt/ — "rút"; Mỹ đọc /raʊt/ "rao-t". Với khách Anh, dùng /ruːt/ |
| rủi ro | risk | âm cuối **-sk** phải bật ra, không thành "rít" |
| máy chủ | host | âm cuối **-st** phải bật ra, không thành "hâu" |
| nói to suy nghĩ | to think out loud | *think* âm **th** /θ/, không đọc thành "tin" |
| dẫn dắt buổi phỏng vấn | to drive the interview | |
| tinh thần làm chủ | ownership | |
| phạm vi | scope | |
| nằm ngoài phạm vi | out of scope / non-goals | |
| thiết kế thừa | to over-engineer | âu-vơ-en-jị-**NIA**, trọng âm rơi vào âm **cuối** |
| giả định | an assumption | ơ-**SĂMP**-shợn, trọng âm âm thứ hai |
| phương án thay thế | an alternative | Anh /ɔːlˈtɜːnətɪv/ — ôn-**TƠ**-nơ-tiv, trọng âm âm thứ hai |
| tài liệu thiết kế | a design document | |
| kế hoạch triển khai | the rollout plan | |
| tồn kho | inventory | Anh /ˈɪnvəntri/ — "IN-vần-tri", trọng âm đầu, ba âm tiết |
| bộ nhớ đệm | cache | /kæʃ/ — đọc đúng như "cash", không đọc "ca-chê" |

---

## Bài nói

> **Cách luyện:** bật ghi âm, nói **60–90 giây** cho mỗi đề, **không nhìn tài liệu** khi nói. Mỗi bài phải dùng ít nhất **sáu thuật ngữ** trong bảng trên. Nghe lại bản ghi và đánh dấu chỗ nào bạn phải dừng lại tìm từ — chính chỗ đó là cụm cần học lại từ bảng ánh xạ.

1. **Explain to a junior engineer** the six steps you go through in a system design question, and say why the order of those steps matters.

2. **Explain to a junior engineer** the difference between functional and non-functional requirements, and why you settle the non-functional ones before you draw anything.

3. **A colleague says:** *"Clarifying questions are a waste of time — the interviewer gives you the problem, so you should just start designing."* **Explain what is wrong with that**, and describe what actually happens when the interviewer stays silent after giving you the problem.

4. **Someone on your team wants to** estimate every number in the design doc, including the byte size of every field. **Explain why you would push back**, and say which numbers you would keep.

5. **Describe what happens when** a candidate spends thirty minutes deep-diving into database sharding and then runs out of time. Say how you would manage the forty-five minutes instead.

6. **When would you choose** strong consistency over eventual consistency? Give one system where you must have it and one where you would not pay for it, and state the trade-off in each case.

7. **Explain to a non-technical manager** why the "alternatives considered" and "trade-offs" sections of a design doc are the most valuable parts of the document.
