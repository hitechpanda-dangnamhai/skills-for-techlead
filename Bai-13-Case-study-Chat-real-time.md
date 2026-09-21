# Bài 13 — Case study: Chat real-time
### Bản song ngữ: tiếng Anh bám cấu trúc tiếng Việt

> **Cách dùng:** đọc đoạn tiếng Việt trước → tự dịch trong đầu → so với bản tiếng Anh bên dưới → **đọc to bản tiếng Anh hai lần**. Lần một đọc chậm cho đúng từ, lần hai đọc liền mạch như đang nói với người phỏng vấn. Bài này khác hẳn các bài trước ở một điểm: kết nối là trạng thái, nên mọi thứ chúng ta đã học về dịch vụ không giữ trạng thái đều phải xem lại.

---

## Phần 1 — Vì sao chat khác mọi hệ thống trước đó

**Tiếng Việt**

Trong một hệ thống thông thường, máy khách hỏi và máy chủ trả lời, nên máy chủ luôn ở thế bị động. Trong chat, máy chủ phải chủ động đẩy tin nhắn xuống máy khách vào bất cứ lúc nào, kể cả khi máy khách không hỏi gì. Cách làm cũ là máy khách hỏi liên tục xem có tin mới không, và cách đó vừa tốn tài nguyên vừa trễ. Nếu chúng ta hỏi mỗi ba giây, một tin nhắn có thể chờ tới ba giây mới tới nơi, mà phần lớn lượt hỏi lại trả về rỗng. Vì vậy chúng ta dùng WebSocket, tức là một đường ống hai chiều mở sẵn giữa máy khách và máy chủ.

Đường ống mở sẵn giải quyết được vấn đề độ trễ, nhưng nó tạo ra một vấn đề mới về kiến trúc. Kết nối đó dính vào đúng một máy chủ cụ thể, nên máy chủ đó bây giờ giữ trạng thái chứ không còn thay thế được cho nhau. Chúng ta không thể tuỳ tiện gửi một tin nhắn tới bất kỳ máy nào rồi mong nó tới được người nhận. Nói cách khác, chúng ta vừa đánh đổi tính không giữ trạng thái lấy khả năng đẩy tin theo thời gian thực. Đây là câu mà chúng ta nên nói ra sớm trong buổi phỏng vấn, vì nó cho thấy chúng ta hiểu chi phí thật của lựa chọn này.

**English (bám cấu trúc tiếng Việt)**

In an ordinary system, the client asks and the server answers, so the server is always in a passive position. In chat, the server has to actively push messages down to the client at any moment, even when the client is not asking for anything. The old approach is that the client keeps asking whether there is a new message, and that approach both wastes resources and is slow. If we ask every three seconds, one message may wait up to three seconds before it arrives, while most of the asks come back empty. Therefore we use WebSocket, that is, a two-way pipe held open between the client and the server.

The always-open pipe solves the latency problem, but it creates a new architectural problem. That connection is stuck to exactly one specific server, so that server now holds state and is no longer interchangeable with the others. We cannot casually send a message to any machine and then hope it reaches the recipient. In other words, we have just traded statelessness for the ability to push messages in real time. This is a sentence we should say early in the interview, because it shows that we understand the real cost of this choice.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| máy chủ luôn ở thế bị động | the server is always in a passive position |
| chủ động đẩy tin nhắn xuống máy khách | actively push messages down to the client |
| kể cả khi máy khách không hỏi gì | even when the client is not asking for anything |
| hỏi liên tục xem có tin mới không | keeps asking whether there is a new message |
| phần lớn lượt hỏi lại trả về rỗng | most of the asks come back empty |
| một đường ống hai chiều mở sẵn | a two-way pipe held open |
| dính vào đúng một máy chủ cụ thể | is stuck to exactly one specific server |
| không còn thay thế được cho nhau | is no longer interchangeable |
| chúng ta không thể tuỳ tiện gửi | we cannot casually send |
| chúng ta vừa đánh đổi tính không giữ trạng thái lấy | we have just traded statelessness for |

**Thuật ngữ cần nhớ**

- hỏi thăm liên tục → **polling**
- hỏi thăm giữ lâu → **long polling**
- kết nối hai chiều → **a two-way connection**
- có giữ trạng thái → **stateful**
- theo thời gian thực → **in real time**

---

## Phần 2 — Mở rộng tầng kết nối

**Tiếng Việt**

Với hàng triệu kết nối đồng thời, chúng ta không thể giữ tất cả trên một máy, nên chúng ta chạy nhiều cổng kết nối song song. Mỗi cổng giữ một phần các kết nối, và bộ cân bằng tải chia người dùng ra khi họ nối vào. Đến đây xuất hiện câu hỏi cốt lõi của bài: khi người dùng A gửi tin cho người dùng B, làm sao chúng ta biết B đang nối ở cổng nào. Câu trả lời là chúng ta cần một sổ địa chỉ, tức là một bảng tra ánh xạ từ mã người dùng tới mã cổng đang giữ kết nối của người đó. Bảng tra này thường nằm trong Redis, vì nó phải đọc rất nhanh và phải được mọi cổng nhìn thấy.

Với sổ địa chỉ, việc gửi tin trở nên rõ ràng theo ba trường hợp. Nếu người nhận đang nối ở chính cổng đang xử lý, chúng ta đẩy thẳng xuống kết nối đó. Nếu người nhận đang nối ở một cổng khác, chúng ta phát một thông điệp qua kênh của cổng đó, và cổng kia nhận rồi đẩy xuống máy khách. Nếu người nhận không có mục nào trong sổ địa chỉ, tức là họ đang ngoại tuyến, chúng ta lưu tin lại và đẩy khi họ nối lại. Chúng ta cũng nên tách tầng giữ kết nối ra khỏi tầng xử lý nghiệp vụ, vì hai tầng đó mở rộng theo hai chiều khác nhau và hỏng theo hai kiểu khác nhau.

**English (bám cấu trúc tiếng Việt)**

With millions of concurrent connections, we cannot hold them all on one machine, so we run many connection gateways in parallel. Each gateway holds a portion of the connections, and the load balancer spreads the users out as they connect. At this point the core question of the exercise appears: when user A sends a message to user B, how do we know which gateway B is connected to. The answer is that we need an address book, that is, a lookup table mapping from a user id to the id of the gateway holding that person's connection. This lookup table usually lives in Redis, because it has to be read very fast and it has to be visible to every gateway.

With the address book, sending a message becomes clear across three cases. If the recipient is connected to the very gateway that is handling this, we push it straight down that connection. If the recipient is connected to a different gateway, we publish a message on that gateway's channel, and the other gateway receives it and pushes it down to the client. If the recipient has no entry in the address book, that is, they are offline, we store the message and deliver it when they reconnect. We should also separate the connection-holding layer from the business logic layer, because those two layers scale along two different dimensions and fail in two different ways.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| hàng triệu kết nối đồng thời | millions of concurrent connections |
| nhiều cổng kết nối song song | many connection gateways in parallel |
| chia người dùng ra khi họ nối vào | spreads the users out as they connect |
| làm sao chúng ta biết B đang nối ở cổng nào | how do we know which gateway B is connected to |
| một sổ địa chỉ | an address book |
| một bảng tra ánh xạ từ mã người dùng tới mã cổng | a lookup table mapping from a user id to a gateway id |
| phải được mọi cổng nhìn thấy | has to be visible to every gateway |
| chúng ta phát một thông điệp qua kênh của cổng đó | we publish a message on that gateway's channel |
| không có mục nào trong sổ địa chỉ | has no entry in the address book |
| mở rộng theo hai chiều khác nhau | scale along two different dimensions |

**Thuật ngữ cần nhớ**

- cổng kết nối → **a connection gateway**
- sổ đăng ký kết nối → **a connection registry**
- xuất bản và đăng ký nhận → **publish and subscribe (pub/sub)**
- kênh → **a channel**
- ngoại tuyến → **offline**

---

## Phần 3 — Lưu tin trước, báo nhận sau

**Tiếng Việt**

Thứ tự giữa việc ghi và việc báo nhận là chi tiết nhỏ nhưng nó quyết định người dùng có mất tin hay không. Nguyên tắc là chúng ta ghi tin nhắn xuống kho bền vững trước, rồi mới báo cho người gửi rằng tin đã gửi thành công. Nếu chúng ta báo trước rồi mới ghi, và máy chủ chết đúng khoảnh khắc giữa hai việc đó, người gửi tin rằng tin đã đi trong khi thực tế nó biến mất. Đây là loại lỗi phá vỡ lòng tin của người dùng nhanh hơn bất kỳ lỗi hiệu năng nào. Vì vậy chúng ta luôn nói rõ trình tự này khi trình bày, dù nó chỉ là một câu ngắn.

Về việc giao tin, chúng ta chọn bảo đảm giao ít nhất một lần thay vì đúng một lần. Lý do là bảo đảm đúng một lần trong hệ phân tán rất đắt và thường không cần thiết. Đổi lại, chúng ta phải chấp nhận rằng cùng một tin có thể được giao hai lần khi có thử lại. Chúng ta khử trùng lặp bằng cách gán cho mỗi tin một mã do máy khách sinh ra, rồi bỏ qua tin nào có mã đã thấy. Cách này chuyển bài toán từ chỗ khó là mạng sang chỗ dễ là bộ nhớ, và nó là mẫu chuẩn cho mọi hệ thống nhắn tin.

**English (bám cấu trúc tiếng Việt)**

The order between writing and acknowledging is a small detail but it decides whether users lose messages or not. The principle is that we write the message down to durable storage first, and only then tell the sender that the message was sent successfully. If we acknowledge first and write afterwards, and the server dies in exactly the moment between those two actions, the sender believes the message has gone while in reality it has vanished. This is the kind of bug that breaks user trust faster than any performance bug. Therefore we always state this ordering clearly when we present, even though it is only one short sentence.

On delivery, we choose an at-least-once guarantee rather than an exactly-once one. The reason is that an exactly-once guarantee in a distributed system is very expensive and usually unnecessary. In exchange, we have to accept that the same message may be delivered twice when there is a retry. We deduplicate by giving each message an id generated by the client, and then dropping any message whose id we have already seen. This approach moves the problem from the hard place, which is the network, to the easy place, which is memory, and it is the standard pattern for every messaging system.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| thứ tự giữa việc ghi và việc báo nhận | the order between writing and acknowledging |
| ghi tin nhắn xuống kho bền vững trước | write the message down to durable storage first |
| chết đúng khoảnh khắc giữa hai việc đó | dies in exactly the moment between those two actions |
| trong khi thực tế nó biến mất | while in reality it has vanished |
| phá vỡ lòng tin của người dùng | breaks user trust |
| bảo đảm giao ít nhất một lần | an at-least-once guarantee |
| khi có thử lại | when there is a retry |
| một mã do máy khách sinh ra | an id generated by the client |
| bỏ qua tin nào có mã đã thấy | dropping any message whose id we have already seen |
| chuyển bài toán từ chỗ khó sang chỗ dễ | moves the problem from the hard place to the easy place |

**Thuật ngữ cần nhớ**

- báo nhận → **to acknowledge** / **an ack**
- lưu bền → **to persist**
- ít nhất một lần → **at-least-once**
- đúng một lần → **exactly-once**
- khử trùng lặp → **deduplication**

---

## Phần 4 — Thứ tự tin nhắn trong một phòng

**Tiếng Việt**

Người dùng chấp nhận tin đến chậm, nhưng họ không chấp nhận tin đến sai thứ tự, vì một cuộc trò chuyện đảo lộn thì vô nghĩa. Điều may mắn là chúng ta chỉ cần bảo đảm thứ tự bên trong một phòng, chứ không cần thứ tự toàn cục giữa mọi phòng. Cách làm phổ biến là mỗi phòng có một bộ đếm riêng, và mỗi tin nhắn nhận một số thứ tự tăng dần trong phòng đó. Máy khách sắp xếp theo số thứ tự này khi hiển thị, nên dù các gói tin tới không đúng trình tự thì màn hình vẫn đúng. Chúng ta nên tránh dùng đồng hồ của máy gửi làm thứ tự, vì đồng hồ giữa các máy luôn lệch nhau chút ít.

Nếu chúng ta phân mảnh kho tin nhắn, chúng ta nên chọn mã phòng làm khoá phân mảnh. Nhờ đó, mọi tin của cùng một phòng nằm trên cùng một mảnh, và việc giữ thứ tự trở nên đơn giản trong phạm vi mảnh đó. Cách chọn khoá này cũng làm cho việc đọc lịch sử trò chuyện rất hiệu quả, vì tất cả dữ liệu cần đọc nằm cạnh nhau. Đổi lại, một phòng cực đông có thể trở thành một phân vùng nóng, và chúng ta phải nói ra rủi ro đó. Nếu chúng ta chọn mã người dùng làm khoá phân mảnh, hậu quả là mỗi lần mở một cuộc trò chuyện chúng ta phải gom dữ liệu từ nhiều mảnh và sắp xếp lại.

**English (bám cấu trúc tiếng Việt)**

Users accept messages arriving late, but they do not accept messages arriving in the wrong order, because a scrambled conversation is meaningless. Fortunately, we only need to guarantee ordering inside one room, we do not need a global ordering across all rooms. The common approach is that each room has its own counter, and each message receives an increasing sequence number within that room. The client sorts by this sequence number when displaying, so even if the packets arrive out of order the screen is still correct. We should avoid using the sender's clock as the ordering, because clocks across machines are always slightly out of step.

If we shard the message store, we should choose the room id as the shard key. Thanks to that, all the messages of one room sit on the same shard, and keeping the order becomes simple within that shard. This choice of key also makes reading conversation history very efficient, because all the data we need to read sits next to each other. In exchange, an extremely busy room can become a hot partition, and we have to say that risk out loud. If we choose the user id as the shard key, the consequence is that every time we open a conversation we have to gather data from several shards and sort it again.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| họ không chấp nhận tin đến sai thứ tự | they do not accept messages arriving in the wrong order |
| một cuộc trò chuyện đảo lộn thì vô nghĩa | a scrambled conversation is meaningless |
| điều may mắn là | fortunately |
| thứ tự toàn cục giữa mọi phòng | a global ordering across all rooms |
| một số thứ tự tăng dần trong phòng đó | an increasing sequence number within that room |
| dù các gói tin tới không đúng trình tự | even if the packets arrive out of order |
| đồng hồ giữa các máy luôn lệch nhau chút ít | clocks across machines are always slightly out of step |
| tất cả dữ liệu cần đọc nằm cạnh nhau | all the data we need to read sits next to each other |
| một phòng cực đông | an extremely busy room |
| gom dữ liệu từ nhiều mảnh và sắp xếp lại | gather data from several shards and sort it again |

**Thuật ngữ cần nhớ**

- thứ tự → **ordering**
- số thứ tự → **a sequence number**
- sai trình tự → **out of order**
- khoá phân mảnh → **a shard key**
- lịch sử trò chuyện → **conversation history**

---

## Phần 5 — Trạng thái hiện diện và trạng thái đang gõ

**Tiếng Việt**

Trạng thái hiện diện nhìn có vẻ là tính năng nhỏ nhưng nó lại tốn kém nhất trong cả hệ thống. Lý do là nó thay đổi liên tục, và mỗi lần thay đổi thì phải phát tới tất cả những người đang nhìn thấy người đó. Với một người có năm trăm liên hệ, một lần chuyển từ ngoại tuyến sang trực tuyến sinh ra năm trăm lượt đẩy. Nếu chúng ta phát mọi thay đổi ngay lập tức, số lượt đẩy của riêng phần hiện diện có thể vượt xa số lượt đẩy của chính tin nhắn. Đây là kiểu chi phí ẩn mà chỉ người đã vận hành hệ thống thật mới nhắc tới.

Chúng ta làm nhẹ phần này bằng ba biện pháp. Thứ nhất, máy khách gửi một nhịp tim theo chu kỳ, và chúng ta lưu trạng thái trong Redis với thời gian sống ngắn hơn hai chu kỳ. Nếu không nhận được nhịp tim nào nữa, mục đó tự hết hạn và chúng ta coi người đó là ngoại tuyến, nên chúng ta không cần một sự kiện đăng xuất tường minh. Thứ hai, chúng ta bóp nhịp cập nhật, ví dụ chỉ phát thay đổi tối đa một lần mỗi mười giây cho mỗi người. Thứ ba, chúng ta chấp nhận rằng trạng thái hiện diện là xấp xỉ và trễ vài giây, vì hiển thị sai vài giây không gây thiệt hại gì. Với trạng thái đang gõ thì chúng ta còn mạnh tay hơn: chỉ gửi trong phạm vi phòng đang mở, và để nó tự tắt sau vài giây mà không cần một sự kiện dừng gõ.

**English (bám cấu trúc tiếng Việt)**

Presence looks like a small feature but it is in fact the most expensive one in the whole system. The reason is that it changes constantly, and each change has to be broadcast to everyone who can see that person. For a person with five hundred contacts, one switch from offline to online generates five hundred pushes. If we broadcast every change immediately, the number of pushes from presence alone can far exceed the number of pushes from the messages themselves. This is the kind of hidden cost that only someone who has operated a real system will bring up.

We lighten this part with three measures. First, the client sends a heartbeat on a cycle, and we store the status in Redis with a time to live shorter than two cycles. If no more heartbeats arrive, that entry expires by itself and we treat the person as offline, so we do not need an explicit logout event. Second, we throttle the update rate, for example broadcasting a change at most once every ten seconds per person. Third, we accept that presence is approximate and a few seconds behind, because showing it wrongly for a few seconds causes no damage at all. With the typing indicator we go even further: we only send it within the room that is currently open, and we let it switch itself off after a few seconds without needing a stop-typing event.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| nhìn có vẻ là tính năng nhỏ | looks like a small feature |
| phát tới tất cả những người đang nhìn thấy người đó | broadcast to everyone who can see that person |
| sinh ra năm trăm lượt đẩy | generates five hundred pushes |
| có thể vượt xa | can far exceed |
| kiểu chi phí ẩn | the kind of hidden cost |
| máy khách gửi một nhịp tim theo chu kỳ | the client sends a heartbeat on a cycle |
| mục đó tự hết hạn | that entry expires by itself |
| một sự kiện đăng xuất tường minh | an explicit logout event |
| chúng ta bóp nhịp cập nhật | we throttle the update rate |
| chúng ta còn mạnh tay hơn | we go even further |
| để nó tự tắt sau vài giây | let it switch itself off after a few seconds |

**Thuật ngữ cần nhớ**

- trạng thái hiện diện → **presence**
- nhịp tim → **a heartbeat**
- bóp nhịp → **to throttle**
- thời gian sống → **time to live (TTL)**
- trạng thái đang gõ → **a typing indicator**

---

## Phần 6 — Trò chuyện nhóm khác trò chuyện một đối một

**Tiếng Việt**

Trong trò chuyện một đối một, một tin nhắn chỉ cần tới một người, nên việc định tuyến rất đơn giản. Trong trò chuyện nhóm, một tin nhắn phải tới tất cả thành viên, nên chúng ta gặp lại đúng bài toán phát tán đã học ở bài bảng tin. Với nhóm nhỏ vài chục người, chúng ta cứ đẩy thẳng cho từng người và không cần nghĩ nhiều. Với nhóm rất lớn, ví dụ một kênh có năm mươi nghìn thành viên, việc đẩy cho từng người trở nên rất tốn kém, nhất là khi phần lớn họ không mở kênh đó. Lúc này chúng ta chuyển sang cách kéo cho những thành viên ít hoạt động, và chỉ đẩy cho những người đang mở kênh.

Chúng ta cũng nên nói về cách theo dõi trạng thái đã đọc, vì đó là chỗ chi phí âm thầm tăng lên. Trong nhóm lớn, nếu chúng ta lưu chi tiết ai đã đọc tin nào, số bản ghi bằng số tin nhân với số thành viên. Cách nhẹ hơn là mỗi thành viên chỉ lưu một con trỏ đọc, tức là số thứ tự của tin cuối cùng mà họ đã xem. Từ con trỏ đó, chúng ta suy ra số tin chưa đọc mà không cần lưu từng cặp. Nếu chúng ta lưu chi tiết cho mọi nhóm, hậu quả là bảng trạng thái đọc lớn hơn cả bảng tin nhắn, và nó tăng nhanh hơn theo cấp số nhân khi nhóm to dần.

**English (bám cấu trúc tiếng Việt)**

In one-to-one chat, a message only needs to reach one person, so the routing is very simple. In group chat, a message has to reach all the members, so we meet again exactly the fan-out problem we learned in the news feed lesson. For a small group of a few dozen people, we simply push to each person and we do not need to think much. For a very large group, for example a channel with fifty thousand members, pushing to each person becomes very expensive, especially since most of them do not have that channel open. At this point we switch to the pull approach for the less active members, and we only push to the people who currently have the channel open.

We should also talk about how we track read status, because that is where the cost quietly grows. In a large group, if we store in detail who has read which message, the number of records equals the number of messages multiplied by the number of members. The lighter approach is that each member only stores one read cursor, that is, the sequence number of the last message they have seen. From that cursor, we derive the unread count without having to store every pair. If we store the detailed version for every group, the consequence is that the read-status table becomes larger than the message table itself, and it grows exponentially faster as the groups get bigger.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| việc định tuyến rất đơn giản | the routing is very simple |
| chúng ta gặp lại đúng bài toán phát tán | we meet again exactly the fan-out problem |
| chúng ta cứ đẩy thẳng cho từng người | we simply push to each person |
| nhất là khi phần lớn họ không mở kênh đó | especially since most of them do not have that channel open |
| những thành viên ít hoạt động | the less active members |
| chỗ chi phí âm thầm tăng lên | where the cost quietly grows |
| số bản ghi bằng số tin nhân với số thành viên | the number of records equals the messages multiplied by the members |
| một con trỏ đọc | a read cursor |
| chúng ta suy ra số tin chưa đọc | we derive the unread count |
| tăng nhanh hơn theo cấp số nhân khi nhóm to dần | grows exponentially faster as the groups get bigger |

**Thuật ngữ cần nhớ**

- trò chuyện nhóm → **group chat**
- thành viên → **a member**
- con trỏ đọc → **a read cursor**
- số tin chưa đọc → **the unread count**
- biên nhận đã đọc → **a read receipt**

---

## Phần 7 — Đi theo một tin nhắn từ đầu tới cuối

**Tiếng Việt**

Cách trình bày mạnh nhất cho bài này là chúng ta đi theo một tin nhắn duy nhất qua toàn bộ hệ thống. Người gửi gõ tin và máy khách gán cho nó một mã cục bộ, rồi gửi qua đường ống đang mở tới cổng kết nối mà mình đang nối. Cổng đó chuyển tin sang tầng nghiệp vụ, tầng nghiệp vụ ghi tin xuống kho bền vững kèm số thứ tự của phòng, rồi mới báo nhận về cho người gửi. Sau khi ghi xong, hệ thống tra sổ địa chỉ cho từng người nhận và định tuyến theo ba trường hợp đã nói ở phần hai. Người nhận đang trực tuyến thì thấy tin trong vài chục mili giây, còn người ngoại tuyến thì nhận thông báo đẩy và sẽ thấy tin khi họ mở ứng dụng.

Chúng ta nên khép lại bằng cách nói ra chỗ hệ thống này sẽ vỡ trước tiên, vì đó là câu người phỏng vấn chờ. Điểm yếu rõ nhất là chính sổ địa chỉ, vì mọi tin nhắn đều phải tra nó và nó là trạng thái tập trung. Điểm yếu thứ hai là các nhóm cực lớn, vì một tin trong nhóm đó biến thành hàng chục nghìn lượt đẩy. Điểm yếu thứ ba là lúc khôi phục sau sự cố, khi hàng triệu máy khách cùng nối lại một lúc và tạo ra một cơn bão kết nối. Chúng ta giảm rủi ro cuối này bằng cách cho máy khách nối lại theo thời gian ngẫu nhiên và lùi dần, thay vì để tất cả cùng thử lại ngay lập tức.

**English (bám cấu trúc tiếng Việt)**

The strongest way to present this exercise is that we follow one single message through the whole system. The sender types the message and the client gives it a local id, then sends it through the open pipe to the connection gateway it is attached to. That gateway passes the message to the business layer, the business layer writes the message down to durable storage together with the room's sequence number, and only then acknowledges back to the sender. After the write finishes, the system looks up the address book for each recipient and routes it according to the three cases described in part two. A recipient who is online sees the message within a few tens of milliseconds, while an offline one receives a push notification and will see the message when they open the app.

We should close by saying where this system will break first, because that is the sentence the interviewer is waiting for. The clearest weak point is the address book itself, because every message has to look it up and it is centralised state. The second weak point is the extremely large groups, because one message in such a group turns into tens of thousands of pushes. The third weak point is the recovery after an incident, when millions of clients reconnect at the same time and create a connection storm. We reduce this last risk by making the clients reconnect at random times with a growing delay, instead of letting them all retry immediately.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| đi theo một tin nhắn duy nhất | follow one single message |
| máy khách gán cho nó một mã cục bộ | the client gives it a local id |
| cổng kết nối mà mình đang nối | the connection gateway it is attached to |
| rồi mới báo nhận về cho người gửi | and only then acknowledges back to the sender |
| tra sổ địa chỉ cho từng người nhận | looks up the address book for each recipient |
| khép lại bằng cách nói ra chỗ hệ thống này sẽ vỡ trước tiên | close by saying where this system will break first |
| điểm yếu rõ nhất | the clearest weak point |
| biến thành hàng chục nghìn lượt đẩy | turns into tens of thousands of pushes |
| tạo ra một cơn bão kết nối | create a connection storm |
| nối lại theo thời gian ngẫu nhiên và lùi dần | reconnect at random times with a growing delay |

**Thuật ngữ cần nhớ**

- thông báo đẩy → **a push notification**
- cơn bão kết nối → **a connection storm**
- lùi dần theo cấp số nhân → **exponential backoff**
- ngẫu nhiên hoá độ trễ → **jitter**
- khôi phục sau sự cố → **recovery after an incident**

---

## Mô hình ghi nhớ

**Tiếng Việt**

Đường ống mở sẵn dính vào một máy chủ cụ thể, nên chúng ta cần một sổ địa chỉ để gửi tin đúng chỗ. Chúng ta ghi trước rồi mới báo nhận, chúng ta giữ thứ tự theo từng phòng, và chúng ta làm nhẹ trạng thái hiện diện bằng nhịp tim cùng thời gian sống.

**English (bám cấu trúc tiếng Việt)**

The always-open pipe is stuck to one specific server, so we need an address book to send messages to the right place. We write first and acknowledge afterwards, we keep the ordering per room, and we lighten presence with heartbeats and a time to live.

---

## Bảng thuật ngữ tổng hợp

| Tiếng Việt | Tiếng Anh | Ghi chú phát âm |
|---|---|---|
| hỏi thăm liên tục | polling | |
| hỏi thăm giữ lâu | long polling | |
| kết nối hai chiều | a two-way connection | |
| có giữ trạng thái | stateful | |
| theo thời gian thực | in real time | |
| cổng kết nối | a connection gateway | |
| sổ đăng ký kết nối | a connection registry | *registry* — "**RE**-jịs-tri", trọng âm đầu |
| xuất bản và đăng ký nhận | publish and subscribe (pub/sub) | *subscribe* — sợb-**SCRAIB**, trọng âm âm thứ hai |
| kênh | a channel | "**CHA**-nợl", trọng âm đầu |
| ngoại tuyến | offline | |
| báo nhận | to acknowledge / an ack | ợc-**NO**-lịj — chữ **k** đầu **câm** |
| lưu bền | to persist | pơ-**SIST**, trọng âm âm thứ hai; âm cuối **-st** phải bật |
| kho bền vững | durable storage | *durable* Anh — "**DIU**-ơ-rơ-bợl" |
| ít nhất một lần | at-least-once | |
| đúng một lần | exactly-once | |
| khử trùng lặp | deduplication | |
| thứ tự | ordering | |
| số thứ tự | a sequence number | *sequence* — "**SI**-kwợns", trọng âm đầu |
| sai trình tự | out of order | |
| khoá phân mảnh | a shard key | *shard* /ʃɑːd/ — âm đầu là "sh" |
| phân vùng nóng | a hot partition | *partition* — pa-**TI**-shợn, trọng âm âm thứ hai |
| lịch sử trò chuyện | conversation history | |
| trạng thái hiện diện | presence | "**PRE**-zợns", âm giữa là /z/ |
| nhịp tim | a heartbeat | *heart* — chữ **ea** đọc là /ɑː/, "haat" |
| bóp nhịp | to throttle | "**THRO**-tợl", âm **th** /θ/ đầu |
| thời gian sống | time to live (TTL) | |
| trạng thái đang gõ | a typing indicator | |
| trò chuyện nhóm | group chat | |
| thành viên | a member | |
| con trỏ đọc | a read cursor | *cursor* — "**CƠ**-sơ", trọng âm đầu |
| số tin chưa đọc | the unread count | *unread* — an-**RED**, trọng âm âm thứ hai |
| biên nhận đã đọc | a read receipt | *receipt* /rɪˈsiːt/ — chữ **p câm**, đọc "ri-SIIT" |
| thông báo đẩy | a push notification | |
| cơn bão kết nối | a connection storm | |
| lùi dần theo cấp số nhân | exponential backoff | *exponential* — ek-spơ-**NEN**-shợl |
| ngẫu nhiên hoá độ trễ | jitter | "**JI**-tơ", âm đầu là "j" |
| khôi phục sau sự cố | recovery after an incident | *incident* — "**IN**-sị-đợnt", trọng âm đầu |
| đồng thời | concurrent | cần-**CA**-rợnt, trọng âm âm thứ hai |
| phát tán | fan-out | |
| hàng đợi | a queue | /kjuː/ — đọc như chữ "Q" |

---

## Bài nói

> **Cách luyện:** bật ghi âm, nói **60–90 giây** cho mỗi đề, **không nhìn tài liệu** khi nói. Mỗi bài phải dùng ít nhất **sáu thuật ngữ** trong bảng trên. Đề số 7 là đề tổng hợp — hãy dùng nó như một buổi diễn tập trình bày trọn vẹn.

1. **Explain to a junior engineer** why chat cannot be built the same way as a normal request-response API, and what you give up when you choose WebSocket.

2. **Describe what happens when** server A receives a message for user X, but X is connected to server B. Walk through every step until the message reaches X's screen.

3. **A colleague says:** *"We acknowledge the message as soon as we receive it, then write it to the database — it makes the app feel faster."* **Explain what is wrong with that**, and describe the failure the user would experience.

4. **A colleague says:** *"Let's broadcast presence to all contacts every time someone's status changes, so the dots are always accurate."* **Explain why you would push back**, and describe the design you would propose instead.

5. **Explain to a junior engineer** how you guarantee message ordering inside a room, and why you would not use the sender's clock for that.

6. **When would you choose** to push messages to every member of a group, and when would you switch to a pull model? Use a fifty-thousand-member channel as your example.

7. **Describe what happens when** a whole region of gateways restarts and two million clients try to reconnect at the same second, and explain what you would build so that the system survives it.
