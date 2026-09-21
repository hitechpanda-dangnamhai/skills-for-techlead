# Bài 9 — GoF Behavioral: Strategy · Observer · Command · Template Method · State · Chain of Responsibility
### Bản song ngữ: tiếng Anh bám cấu trúc tiếng Việt

> **Cách dùng bài này:** đọc đoạn tiếng Việt trước, tự dịch trong đầu, rồi mới nhìn sang đoạn tiếng Anh để so. Sau khi so xong, **đọc to đoạn tiếng Anh hai lần** — lần một để nhìn chữ, lần hai để nghe chính mình. Phần bài nói ở cuối là bắt buộc; không nói ra thì bài này chỉ là bài đọc.

---

## Phần 1 — Nhóm Behavioral và hai cặp rất dễ nhầm

**Tiếng Việt**

Nhóm Behavioral nói về cách các object giao tiếp với nhau và cách chúng phân chia trách nhiệm lúc chạy. Đây là nhóm mà chúng ta gặp nhiều nhất khi làm backend, bởi vì middleware chính là Chain of Responsibility, hệ thống event chính là Observer, còn luồng duyệt đơn chính là State. Trong nhóm này có hai cặp rất dễ nhầm, đó là State với Strategy và Observer với Pub/Sub. Nội dung ở đây nối thẳng vào Bài 3, nơi chúng ta đã thấy Strategy co lại thành một hàm trong TypeScript. Nếu chúng ta nói được hai cặp dễ nhầm đó một cách rành mạch, chúng ta đã vượt qua phần lớn ứng viên ở câu hỏi về pattern.

**English (bám cấu trúc tiếng Việt)**

The Behavioral group is about how objects communicate with each other and how they divide responsibilities at run time. This is the group that we meet the most when we do backend work, because middleware is exactly the Chain of Responsibility, an event system is exactly the Observer, while an approval flow is exactly the State. In this group there are two pairs that are very easy to confuse, which are State with Strategy and Observer with Pub/Sub. The content here connects straight into Lesson 3, where we saw the Strategy shrink into a function in TypeScript. If we can talk about those two confusing pairs clearly, we have already passed most candidates on the question about patterns.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| cách chúng phân chia trách nhiệm lúc chạy | how they divide responsibilities at run time |
| nhóm mà chúng ta gặp nhiều nhất | the group that we meet the most |
| luồng duyệt đơn | an approval flow |
| hai cặp rất dễ nhầm | two pairs that are very easy to confuse |
| nơi chúng ta đã thấy Strategy co lại thành một hàm | where we saw the Strategy shrink into a function |
| một cách rành mạch | clearly |
| chúng ta đã vượt qua phần lớn ứng viên | we have already passed most candidates |

**Thuật ngữ cần nhớ**

- giao tiếp → **communicate**
- luồng duyệt → **approval flow**
- nhầm lẫn → **confuse**
- rành mạch → **clearly** / **crisply**
- ứng viên → **candidate**

---

## Phần 2 — Strategy: hoán đổi thuật toán lúc chạy

**Tiếng Việt**

Strategy đóng gói các thuật toán có thể hoán đổi cho nhau sau cùng một interface, và chúng ta chọn thuật toán nào lúc chạy. Trong TypeScript, việc truyền một hàm vào đã là Strategy ở dạng gọn nhất, cho nên chúng ta không cần một cây class để gọi tên pattern này. Chúng ta hãy so sánh cách dùng if/else với cách dùng Strategy qua ví dụ tính phí giao hàng cho nhiều hãng vận chuyển. Với Strategy, việc thêm một hãng mới chỉ là thêm một class hoặc thêm một hàm, và chúng ta không phải sửa code cũ, đúng tinh thần OCP. Chúng ta cũng test được từng strategy một cách riêng biệt, thay vì phải đi qua một hàm lớn có nhiều nhánh. Cái mất là nếu chỉ có hai hoặc ba ca đơn giản, chúng ta sẽ tạo ra nhiều class và nhiều tầng gián tiếp thừa. Vì vậy chúng ta cân nhắc theo số biến thể, nghĩa là càng nhiều biến thể và càng hay thêm mới thì Strategy càng đáng dùng.

**English (bám cấu trúc tiếng Việt)**

The Strategy wraps up algorithms that are interchangeable with each other behind one interface, and we choose which algorithm at run time. In TypeScript, passing a function in is already the Strategy in its most compact form, so we do not need a class tree in order to call this pattern by name. Let us compare the if/else way with the Strategy way through the example of computing the shipping fee for several carriers. With the Strategy, adding a new carrier is only adding one class or adding one function, and we do not have to edit the old code, exactly in the spirit of OCP. We can also test each strategy separately, instead of having to go through one large function with many branches. The loss is that if there are only two or three simple cases, we will create many classes and many unnecessary layers of indirection. Therefore we weigh it by the number of variants, which means that the more variants there are and the more often new ones are added, the more the Strategy is worth using.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| các thuật toán có thể hoán đổi cho nhau | algorithms that are interchangeable with each other |
| ở dạng gọn nhất | in its most compact form |
| để gọi tên pattern này | in order to call this pattern by name |
| tính phí giao hàng cho nhiều hãng vận chuyển | computing the shipping fee for several carriers |
| đúng tinh thần OCP | exactly in the spirit of OCP |
| thay vì phải đi qua một hàm lớn có nhiều nhánh | instead of having to go through one large function with many branches |
| nhiều tầng gián tiếp thừa | many unnecessary layers of indirection |
| chúng ta cân nhắc theo số biến thể | we weigh it by the number of variants |
| càng nhiều … thì … càng đáng dùng | the more … there are, the more … is worth using |

**Thuật ngữ cần nhớ**

- thuật toán → **algorithm**
- hoán đổi được → **interchangeable**
- hãng vận chuyển → **carrier**
- nhánh (trong code) → **branch**
- biến thể → **variant**

---

## Phần 3 — Observer, và ranh giới giữa Observer với Pub/Sub

**Tiếng Việt**

Observer mô tả một quan hệ một-nhiều giữa các object. Khi subject đổi trạng thái, nó thông báo cho tất cả observer đã đăng ký nhận tin. Các observer được thêm vào hoặc được bỏ ra ngay lúc chạy, cho nên số lượng người nghe không cố định. Ví dụ quen thuộc là cập nhật giao diện, xoá cache khi dữ liệu đổi, và phát domain event khi một nghiệp vụ hoàn tất. Điểm mạnh của pattern này là subject không phải biết trước có bao nhiêu bên đang quan tâm đến thay đổi của nó.

Chúng ta phải phân biệt Observer với Pub/Sub, bởi vì hai thứ này khác nhau ở mức coupling. Với Observer, subject biết các observer và nó gọi chúng trực tiếp, cho nên mọi thứ chạy trong cùng một tiến trình và mức coupling chỉ ở tầm vừa. Với Pub/Sub, hai bên nói chuyện qua một broker hoặc qua một kênh sự kiện trung gian, cho nên publisher hoàn toàn không biết ai là subscriber. Vì vậy Pub/Sub cho mức tách rời mạnh hơn, và nó thường đi kèm xử lý bất đồng bộ và chạy xuyên nhiều tiến trình. Trong thực tế, `EventEmitter` chạy trong cùng tiến trình nên nó gần với Observer, còn message broker như Kafka hoặc RabbitMQ thì đúng là Pub/Sub. Khi phỏng vấn, chỉ cần một câu hỏi ngắn là có broker ở giữa hay không, và chúng ta trả lời được chính xác.

**English (bám cấu trúc tiếng Việt)**

The Observer describes a one-to-many relationship between objects. When the subject changes state, it notifies all the observers that have subscribed. The observers are added or removed right at run time, so the number of listeners is not fixed. Familiar examples are updating the user interface, invalidating the cache when data changes, and emitting a domain event when a business operation completes. The strength of this pattern is that the subject does not have to know in advance how many parties are interested in its changes.

We have to tell the Observer apart from Pub/Sub, because these two things differ in the level of coupling. With the Observer, the subject knows the observers and it calls them directly, so everything runs inside the same process and the level of coupling is only moderate. With Pub/Sub, the two sides talk through a broker or through an intermediate event channel, so the publisher does not know at all who the subscriber is. Therefore Pub/Sub gives a stronger level of decoupling, and it usually comes with asynchronous handling and with running across several processes. In practice, `EventEmitter` runs inside the same process so it is close to the Observer, while a message broker such as Kafka or RabbitMQ is truly Pub/Sub. In an interview, we only need one short question, which is whether there is a broker in the middle, and we can answer accurately.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| một quan hệ một-nhiều | a one-to-many relationship |
| nó thông báo cho tất cả observer đã đăng ký nhận tin | it notifies all the observers that have subscribed |
| số lượng người nghe không cố định | the number of listeners is not fixed |
| xoá cache khi dữ liệu đổi | invalidating the cache when data changes |
| không phải biết trước có bao nhiêu bên đang quan tâm | does not have to know in advance how many parties are interested |
| chạy trong cùng một tiến trình | runs inside the same process |
| mức coupling chỉ ở tầm vừa | the level of coupling is only moderate |
| một kênh sự kiện trung gian | an intermediate event channel |
| cho mức tách rời mạnh hơn | gives a stronger level of decoupling |
| chạy xuyên nhiều tiến trình | running across several processes |
| có broker ở giữa hay không | whether there is a broker in the middle |

**Thuật ngữ cần nhớ**

- đối tượng phát tin → **subject** / **publisher**
- người nghe, người đăng ký → **listener** / **subscriber**
- đăng ký nhận tin → **subscribe**
- vô hiệu hoá cache → **invalidate the cache**
- bất đồng bộ → **asynchronous**
- tiến trình → **process**

---

## Phần 4 — Lạm dụng `EventEmitter` cho luồng nghiệp vụ

**Tiếng Việt**

`EventEmitter` của Node là hiện thân của Observer, và chính vì nó tiện nên nó rất hay bị lạm dụng. Rủi ro lớn nhất xuất hiện khi chúng ta dùng nó cho luồng nghiệp vụ chính, bởi vì lúc đó luồng điều khiển trở thành ẩn và người đọc code rất khó lần vết. Rủi ro thứ hai là nếu sự kiện `error` không được xử lý, tiến trình Node sẽ dừng hẳn. Rủi ro thứ ba là rò bộ nhớ, bởi vì chúng ta thêm listener vào mà chúng ta quên gỡ chúng ra. Rủi ro thứ tư là thứ tự chạy và tính đồng bộ trở nên rất khó suy luận khi số listener tăng lên. Vì vậy nguyên tắc an toàn là chúng ta dùng event cho các thông báo phụ, còn luồng nghiệp vụ chính thì chúng ta gọi trực tiếp hoặc chúng ta đẩy qua hàng đợi.

**English (bám cấu trúc tiếng Việt)**

The `EventEmitter` in Node is the embodiment of the Observer, and exactly because it is convenient it is very often overused. The biggest risk appears when we use it for the main business flow, because at that point the control flow becomes hidden and the reader of the code finds it very hard to trace. The second risk is that if the `error` event is not handled, the Node process will stop completely. The third risk is a memory leak, because we add listeners and we forget to remove them. The fourth risk is that the running order and the synchronisation become very hard to reason about when the number of listeners grows. Therefore the safe principle is that we use events for secondary notifications, while for the main business flow we call directly or we push it through a queue.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| là hiện thân của Observer | is the embodiment of the Observer |
| chính vì nó tiện nên nó rất hay bị lạm dụng | exactly because it is convenient it is very often overused |
| luồng điều khiển trở thành ẩn | the control flow becomes hidden |
| người đọc code rất khó lần vết | the reader of the code finds it very hard to trace |
| tiến trình Node sẽ dừng hẳn | the Node process will stop completely |
| chúng ta quên gỡ chúng ra | we forget to remove them |
| trở nên rất khó suy luận | become very hard to reason about |
| chúng ta dùng event cho các thông báo phụ | we use events for secondary notifications |
| chúng ta đẩy qua hàng đợi | we push it through a queue |

**Thuật ngữ cần nhớ**

- lạm dụng → **overuse**
- luồng điều khiển → **control flow**
- lần vết → **trace**
- rò bộ nhớ → **memory leak**
- hàng đợi → **queue**

---

## Phần 5 — Command và Template Method

**Tiếng Việt**

Command đóng gói một yêu cầu thành một object, nhờ đó chúng ta truyền yêu cầu đi, chúng ta lưu nó lại, hoặc chúng ta hoãn nó lại. Chính vì yêu cầu đã trở thành dữ liệu, chúng ta mở ra được nhiều khả năng mà một lời gọi hàm thường không cho. Khả năng thứ nhất là hoàn tác và làm lại, bởi vì mỗi command biết cách đảo ngược chính nó. Khả năng thứ hai là xếp hàng đợi và lập lịch, ví dụ mỗi job trong một job queue chính là một command. Khả năng thứ ba là ghi log, phát lại và kiểm toán, bởi vì chúng ta lưu được đúng chuỗi hành động đã xảy ra.

Template Method thì đi theo hướng khác, bởi vì lớp cha định ra khung của thuật toán còn lớp con chỉ override vài bước. Những bước cho phép override đó thường được gọi là hook. Cách này dựa trên inheritance, cho nên nó tạo ra coupling chặt với lớp cha, đúng như chúng ta đã cảnh báo ở Bài 2. Khi chúng ta cần linh hoạt hơn, chúng ta nên cân nhắc chuyển sang Strategy, nghĩa là chúng ta dùng composition thay cho kế thừa. Một câu trả lời phỏng vấn tốt là chúng ta nói rằng Template Method cố định cái khung và chỉ mở vài lỗ, còn Strategy thì cho phép thay cả thuật toán.

**English (bám cấu trúc tiếng Việt)**

The Command wraps a request into an object, thanks to that we pass the request around, we store it, or we postpone it. Exactly because the request has become data, we open up many possibilities that a normal function call does not give. The first possibility is undo and redo, because each command knows how to reverse itself. The second possibility is queueing and scheduling, for example each job in a job queue is exactly a command. The third possibility is logging, replaying and auditing, because we can store the exact sequence of actions that happened.

The Template Method goes in a different direction, because the parent class defines the skeleton of the algorithm while the child class only overrides a few steps. Those steps that are allowed to be overridden are usually called hooks. This way is based on inheritance, so it creates tight coupling with the parent class, exactly as we warned in Lesson 2. When we need more flexibility, we should consider moving to the Strategy, which means that we use composition instead of inheritance. A good interview answer is that we say the Template Method fixes the skeleton and only opens a few holes, while the Strategy allows the whole algorithm to be replaced.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| chúng ta hoãn nó lại | we postpone it |
| chính vì yêu cầu đã trở thành dữ liệu | exactly because the request has become data |
| mỗi command biết cách đảo ngược chính nó | each command knows how to reverse itself |
| xếp hàng đợi và lập lịch | queueing and scheduling |
| chuỗi hành động đã xảy ra | the sequence of actions that happened |
| lớp cha định ra khung của thuật toán | the parent class defines the skeleton of the algorithm |
| những bước cho phép override đó | those steps that are allowed to be overridden |
| cố định cái khung và chỉ mở vài lỗ | fixes the skeleton and only opens a few holes |
| cho phép thay cả thuật toán | allows the whole algorithm to be replaced |

**Thuật ngữ cần nhớ**

- hoãn lại → **postpone**
- hoàn tác / làm lại → **undo** / **redo**
- lập lịch → **schedule**
- kiểm toán, đối soát → **audit**
- khung thuật toán → **skeleton**
- điểm móc để mở rộng → **hook**

---

## Phần 6 — State khác Strategy ở chỗ ai điều khiển việc chuyển

**Tiếng Việt**

State và Strategy có cấu trúc class gần như giống hệt nhau, cho nên chúng ta phải phân biệt chúng bằng ý đồ. Với State, object tự chuyển trạng thái của chính nó, và hành vi của nó đổi theo trạng thái hiện tại. Các state trong pattern này biết chúng chuyển được sang những state nào, cho nên tri thức về đường đi nằm ngay bên trong các state. Với Strategy, bên ngoài chọn thuật toán rồi truyền nó vào, còn các strategy thì độc lập với nhau. Một strategy không tự chuyển sang một strategy khác, bởi vì đó không phải việc của nó. Vì vậy câu trả lời gọn nhất là chúng ta hỏi ai điều khiển việc chuyển, và chúng ta hỏi ý đồ ở đây là quản trạng thái hay là hoán thuật toán.

**English (bám cấu trúc tiếng Việt)**

State and Strategy have almost exactly the same class structure, so we have to tell them apart by intent. With State, the object changes its own state by itself, and its behaviour changes according to the current state. The states in this pattern know which states they can move to, so the knowledge about the path sits right inside the states. With Strategy, the outside chooses the algorithm and then passes it in, while the strategies are independent of each other. One strategy does not move to another strategy by itself, because that is not its job. Therefore the shortest answer is that we ask who controls the transition, and we ask whether the intent here is managing state or swapping algorithms.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| có cấu trúc class gần như giống hệt nhau | have almost exactly the same class structure |
| object tự chuyển trạng thái của chính nó | the object changes its own state by itself |
| đổi theo trạng thái hiện tại | changes according to the current state |
| biết chúng chuyển được sang những state nào | know which states they can move to |
| tri thức về đường đi nằm ngay bên trong các state | the knowledge about the path sits right inside the states |
| bên ngoài chọn thuật toán rồi truyền nó vào | the outside chooses the algorithm and then passes it in |
| đó không phải việc của nó | that is not its job |
| ai điều khiển việc chuyển | who controls the transition |
| quản trạng thái hay là hoán thuật toán | managing state or swapping algorithms |

**Thuật ngữ cần nhớ**

- trạng thái hiện tại → **the current state**
- chuyển trạng thái → **transition**
- độc lập với nhau → **independent of each other**
- điều khiển → **control**
- hoán đổi → **swap**

---

## Phần 7 — Chain of Responsibility chính là middleware

**Tiếng Việt**

Chain of Responsibility dựng một chuỗi handler, và mỗi handler hoặc tự xử lý yêu cầu hoặc chuyển tiếp yêu cầu cho handler kế tiếp. Điều rất đáng nhớ là middleware trong Express và trong Nest chính là pattern này, bởi vì request đi lần lượt qua từng khâu rồi mới tới handler cuối cùng. Nhờ cấu trúc đó, chúng ta thêm khâu, bớt khâu, hoặc đổi thứ tự các khâu một cách rất linh hoạt. Một pipeline điển hình gồm xác thực, giới hạn tần suất, kiểm tra dữ liệu, rồi mới đến phần xử lý nghiệp vụ. Tiêu chí để biết chúng ta làm đúng là việc thêm một khâu mới không buộc chúng ta sửa các khâu đã có. Nếu một handler phải biết về một handler khác, thì chuỗi đã bị coupling và lợi ích của pattern mất đi.

**English (bám cấu trúc tiếng Việt)**

The Chain of Responsibility builds a chain of handlers, and each handler either handles the request itself or forwards the request to the next handler. The thing that is very worth remembering is that middleware in Express and in Nest is exactly this pattern, because the request goes through each stage in turn before it reaches the final handler. Thanks to that structure, we add a stage, remove a stage, or reorder the stages very flexibly. A typical pipeline consists of authentication, rate limiting, data validation, and only then the business handling part. The criterion for knowing that we did it right is that adding a new stage does not force us to edit the existing stages. If one handler has to know about another handler, then the chain has become coupled and the benefit of the pattern is lost.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| dựng một chuỗi handler | builds a chain of handlers |
| hoặc tự xử lý … hoặc chuyển tiếp | either handles … itself or forwards |
| điều rất đáng nhớ là | the thing that is very worth remembering is that |
| đi lần lượt qua từng khâu | goes through each stage in turn |
| đổi thứ tự các khâu một cách rất linh hoạt | reorder the stages very flexibly |
| giới hạn tần suất | rate limiting |
| rồi mới đến phần xử lý nghiệp vụ | and only then the business handling part |
| không buộc chúng ta sửa các khâu đã có | does not force us to edit the existing stages |
| chuỗi đã bị coupling | the chain has become coupled |

**Thuật ngữ cần nhớ**

- chuyển tiếp → **forward**
- khâu, chặng → **stage**
- dây chuyền xử lý → **pipeline**
- giới hạn tần suất → **rate limiting**
- đổi thứ tự → **reorder**

---

## Phần 8 — State machine cho workflow, và soát code AI

**Tiếng Việt**

Một luồng duyệt đơn nhiều bước là ví dụ điển hình để dùng State. Đơn đi từ trạng thái tạo mới sang chờ duyệt, rồi sang được duyệt hoặc bị từ chối, và cuối cùng là đóng lại. Hành vi ở mỗi bước một khác, cho nên nếu chúng ta rải những câu kiểm tra trạng thái khắp nơi thì code sẽ rất khó kiểm soát. Cách làm tốt là chúng ta mô hình nó bằng một state machine, nghĩa là chúng ta gom hành vi và danh sách chuyển tiếp hợp lệ theo từng trạng thái. Nhờ đó chúng ta chặn được những chuyển tiếp sai ngay tại chỗ, và việc thêm một trạng thái mới chỉ là sửa bảng chuyển tiếp. Tuy nhiên khi luồng chỉ có hai trạng thái tuyến tính, một enum kèm vài câu kiểm tra đã đủ, và chúng ta không nên over-engineer.

Nhóm Behavioral cũng có hai lỗi rất dễ nhận khi chúng ta soát code do AI sinh ra. Lỗi thứ nhất là AI hay rải những câu kiểm tra trạng thái khắp service thay vì dựng một bảng chuyển tiếp. Lỗi thứ hai là AI lạm dụng `EventEmitter` cho luồng nghiệp vụ, và điều đó tạo ra luồng điều khiển ẩn. Khi review, chúng ta hỏi hai câu: các chuyển tiếp trạng thái có được kiểm soát tại một chỗ hay không, và các event có nuốt lỗi hoặc có rò listener hay không. Hai câu hỏi đó bắt được gần hết các vấn đề vận hành mà nhóm pattern này gây ra.

**English (bám cấu trúc tiếng Việt)**

An approval flow with many steps is the typical example for using State. The request moves from the created state to the pending state, then to approved or rejected, and finally it is closed. The behaviour at each step is different, so if we scatter status checks everywhere then the code becomes very hard to control. The good way is that we model it with a state machine, which means that we gather the behaviour and the list of valid transitions for each state. Thanks to that we block the wrong transitions right on the spot, and adding a new state is only editing the transition table. However, when the flow only has two linear states, an enum together with a few checks is already enough, and we should not over-engineer.

The Behavioral group also has two mistakes that are very easy to spot when we review code that AI generates. The first mistake is that AI often scatters status checks across the service instead of building a transition table. The second mistake is that AI overuses the `EventEmitter` for the business flow, and that creates a hidden control flow. When we review, we ask two questions: are the state transitions controlled in one place or not, and do the events swallow errors or leak listeners or not. Those two questions catch almost all of the operational problems that this group of patterns causes.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| là ví dụ điển hình để dùng State | is the typical example for using State |
| sang được duyệt hoặc bị từ chối | to approved or rejected |
| hành vi ở mỗi bước một khác | the behaviour at each step is different |
| rải những câu kiểm tra trạng thái khắp nơi | scatter status checks everywhere |
| chúng ta mô hình nó bằng một state machine | we model it with a state machine |
| danh sách chuyển tiếp hợp lệ | the list of valid transitions |
| chặn được những chuyển tiếp sai ngay tại chỗ | block the wrong transitions right on the spot |
| chỉ là sửa bảng chuyển tiếp | is only editing the transition table |
| hai trạng thái tuyến tính | two linear states |
| các event có nuốt lỗi hoặc có rò listener hay không | do the events swallow errors or leak listeners or not |
| các vấn đề vận hành | the operational problems |

**Thuật ngữ cần nhớ**

- máy trạng thái → **state machine**
- bảng chuyển tiếp → **transition table**
- rải khắp nơi → **scatter everywhere**
- nuốt lỗi → **swallow errors**
- vận hành → **operational**

---

## Mô hình ghi nhớ

**Tiếng Việt**

Strategy là hoán thuật toán do bên ngoài chọn, còn State là máy trạng thái tự chuyển lấy. Observer thông báo trực tiếp còn Pub/Sub thông báo qua broker, Command đóng gói một hành động thành dữ liệu, và Chain of Responsibility chính là dây chuyền middleware.

**English (bám cấu trúc tiếng Việt)**

The Strategy is swapping algorithms chosen by the outside, while the State is a state machine that moves by itself. The Observer notifies directly while Pub/Sub notifies through a broker, the Command wraps an action into data, and the Chain of Responsibility is exactly the middleware pipeline.

---

## Bảng thuật ngữ tổng hợp

| Tiếng Việt | Tiếng Anh | Ghi chú phát âm |
|---|---|---|
| giao tiếp | communicate | trọng âm âm hai: com-MU-ni-cate |
| luồng duyệt | approval flow | *approval* trọng âm âm hai: ap-PRO-val |
| nhầm lẫn | confuse | trọng âm cuối: con-FUSE, đuôi đọc "-fiuz" |
| rành mạch | clearly / crisply | |
| ứng viên | candidate | /ˈkændɪdət/ — CAN-di-date, đuôi đọc nhẹ |
| thuật toán | algorithm | /ˈælgərɪðəm/ — AL-go-ri-thm, trọng âm đầu, đuôi có âm **th** rung |
| hoán đổi được | interchangeable | trọng âm âm ba: in-ter-CHANGE-a-ble |
| hãng vận chuyển | carrier | /ˈkæriə/ — CA-rri-ơ |
| nhánh (trong code) | branch | đuôi **-nch** phải bật rõ |
| biến thể | variant | /ˈveəriənt/ — VA-ri-ant |
| đối tượng phát tin | subject / publisher | *subject* danh từ trọng âm đầu: SUB-ject |
| người nghe, người đăng ký | listener / subscriber | *listener* — chữ **t** câm: "LI-sơ-nơ" |
| đăng ký nhận tin | subscribe | trọng âm cuối: sub-SCRIBE |
| vô hiệu hoá cache | invalidate the cache | *cache* /kæʃ/ — đọc giống "cash" |
| bất đồng bộ | asynchronous | /eɪˈsɪŋkrənəs/ — a-SYN-chro-nous, trọng âm âm hai, **ch** đọc như "k" |
| tiến trình | process | Anh đọc /ˈprəʊses/ — PRO-cess, trọng âm đầu |
| trung gian | intermediate | trọng âm âm ba: in-ter-ME-di-ate |
| lạm dụng | overuse | |
| luồng điều khiển | control flow | |
| lần vết | trace | /treɪs/ — đuôi **-s**, không đọc /treɪz/ |
| rò bộ nhớ | memory leak | |
| hàng đợi | queue | /kjuː/ — đọc là "kiu", bốn chữ cái sau **q** đều câm |
| hoãn lại | postpone | trọng âm cuối: post-PONE |
| hoàn tác / làm lại | undo / redo | |
| lập lịch | schedule | người Anh đọc /ˈʃedjuːl/ — "SHED-yule"; người Mỹ đọc "SKED-jule" |
| kiểm toán, đối soát | audit | /ˈɔːdɪt/ — AW-dit |
| khung thuật toán | skeleton | trọng âm đầu: SKE-le-ton |
| điểm móc để mở rộng | hook | |
| trạng thái hiện tại | the current state | *current* /ˈkʌrənt/ — CU-rrent |
| chuyển trạng thái | transition | trọng âm âm hai: tran-SI-tion |
| độc lập với nhau | independent of each other | *independent* trọng âm âm ba: in-de-PEN-dent |
| hoán đổi | swap | /swɒp/ — cụm **sw-** đầu từ, đọc "swop" |
| chuyển tiếp | forward | trọng âm đầu: FOR-ward |
| khâu, chặng | stage | |
| dây chuyền xử lý | pipeline | |
| giới hạn tần suất | rate limiting | |
| đổi thứ tự | reorder | |
| máy trạng thái | state machine | *machine* trọng âm cuối: ma-CHINE, **ch** đọc như "sh" |
| bảng chuyển tiếp | transition table | |
| rải khắp nơi | scatter everywhere | *scatter* — cụm **sc-** phải bật |
| nuốt lỗi | swallow errors | *swallow* /ˈswɒləʊ/ — SWO-llow |
| vận hành | operational | trọng âm áp chót: o-pe-RA-tion-al |

---

## Bài nói

> **Cách làm:** ghi âm lại giọng của mình, mỗi câu nói liên tục **60–90 giây**, **không nhìn tài liệu** khi nói. Mỗi bài nói phải dùng ít nhất **6 thuật ngữ** trong bảng ở trên. Nghe lại bản ghi âm một lần, ghi ra hai chỗ mình bị vấp, rồi nói lại lần hai.

1. Explain to a junior developer what the Strategy pattern solves, and say why passing a function in TypeScript already counts as a Strategy.
2. Explain the difference between the Observer and Pub/Sub in terms of coupling, and give one example of each from a system you have worked on.
3. A colleague wants to drive the whole order-processing flow through `EventEmitter` because "it decouples everything nicely". Explain why you would push back and what you would use instead.
4. Someone on your team says State and Strategy are the same pattern because the class diagrams look identical. Explain what actually separates them.
5. Describe how a request travels through a middleware pipeline, and explain why that pipeline is the Chain of Responsibility.
6. When would you model a workflow with a full state machine, and when would an enum with a few guards be enough? Give the trade-off in both directions.
7. You are reviewing AI-generated code for a subscription lifecycle, and it checks the status with conditions in six different services. Describe out loud what you would ask for and what the code should look like afterwards.
