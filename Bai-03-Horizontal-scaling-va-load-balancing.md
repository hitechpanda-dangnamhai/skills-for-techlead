# Bài 3 — Horizontal scaling & Load balancing
### Bản song ngữ: tiếng Anh bám cấu trúc tiếng Việt

> **Cách dùng:** đọc đoạn tiếng Việt trước → tự dịch trong đầu → so với bản tiếng Anh bên dưới → **đọc to bản tiếng Anh hai lần**. Lần một đọc chậm cho đúng từ, lần hai đọc liền mạch như đang nói với người phỏng vấn.

---

## Phần 1 — Mở rộng ngang thay vì mở rộng dọc

**Tiếng Việt**

Mở rộng dọc nghĩa là chúng ta thuê một đầu bếp giỏi hơn, còn mở rộng ngang nghĩa là chúng ta thuê thêm nhiều đầu bếp. Người đầu bếp giỏi có một cái trần, vì không ai nấu nhanh vô hạn được, và nếu người đó ốm thì cả bếp phải đóng cửa. Nhiều đầu bếp thì mở rộng được gần như vô hạn, và khi một người ốm thì bếp vẫn chạy. Đổi lại, chúng ta cần một người điều phối đơn hàng, và các đầu bếp không được giữ ghi chú riêng trong túi. Hai điều kiện đó chính là bộ cân bằng tải và tính không giữ trạng thái ở phía dịch vụ.

Trong hệ thống thật, mở rộng dọc có ba nhược điểm mà chúng ta phải nói được thành lời. Thứ nhất, phần cứng có trần, nên đến một lúc nào đó chúng ta không mua được máy to hơn nữa. Thứ hai, một máy duy nhất là một điểm chết duy nhất, nên máy đó hỏng là cả dịch vụ ngừng. Thứ ba, mỗi lần nâng cấp máy thì chúng ta phải chấp nhận thời gian dừng. Mở rộng ngang giải quyết cả ba điều trên, nhưng nó đòi hỏi chúng ta phải thiết kế lại dịch vụ cho đúng, chứ nó không tự nhiên mà có được.

**English (bám cấu trúc tiếng Việt)**

Vertical scaling means that we hire a better chef, while horizontal scaling means that we hire more chefs. A better chef has a ceiling, because nobody can cook infinitely fast, and if that person falls ill then the whole kitchen has to close. Many chefs can scale almost without limit, and when one person falls ill the kitchen still runs. In exchange, we need someone to coordinate the orders, and the chefs must not keep their own private notes in their pocket. Those two conditions are exactly the load balancer and the stateless property on the service side.

In a real system, vertical scaling has three drawbacks that we must be able to put into words. First, the hardware has a ceiling, so at some point we cannot buy a bigger machine any more. Second, a single machine is a single point of failure, so when that machine breaks the whole service stops. Third, every time we upgrade the machine we have to accept some downtime. Horizontal scaling solves all three things above, but it requires us to design the service correctly, because it does not come for free.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| có một cái trần | has a ceiling |
| không ai nấu nhanh vô hạn được | nobody can cook infinitely fast |
| cả bếp phải đóng cửa | the whole kitchen has to close |
| mở rộng được gần như vô hạn | can scale almost without limit |
| một người điều phối đơn hàng | someone to coordinate the orders |
| không được giữ ghi chú riêng trong túi | must not keep their own private notes in their pocket |
| ba nhược điểm mà chúng ta phải nói được thành lời | three drawbacks that we must be able to put into words |
| đến một lúc nào đó | at some point |
| chấp nhận thời gian dừng | accept some downtime |
| nó không tự nhiên mà có được | it does not come for free |

**Thuật ngữ cần nhớ**

- mở rộng dọc → **vertical scaling**
- mở rộng ngang → **horizontal scaling**
- trần phần cứng → **the hardware ceiling**
- điểm chết duy nhất → **a single point of failure (SPOF)**
- thời gian dừng → **downtime**
- chịu lỗi → **fault tolerance**

---

## Phần 2 — Không giữ trạng thái là điều kiện của mở rộng ngang

**Tiếng Việt**

Một dịch vụ không giữ trạng thái là dịch vụ mà mỗi instance không giữ trạng thái phiên của người dùng trong bộ nhớ của chính nó. Nhờ đó, bất kỳ instance nào cũng phục vụ được bất kỳ request nào, và bộ cân bằng tải được tự do gửi request đi đâu tuỳ ý. Trạng thái vẫn tồn tại, nhưng chúng ta đẩy nó ra ngoài, thường là vào Redis hoặc vào cơ sở dữ liệu. Đây là điều kiện tiên quyết của mở rộng ngang, chứ không phải là một tối ưu hoá cho đẹp. Chỉ khi các instance giống hệt nhau về mặt trạng thái, chúng ta mới thêm hay bớt máy một cách tự do.

Nếu chúng ta lưu trạng thái trong bộ nhớ của tiến trình ứng dụng, hậu quả xuất hiện ngay khi hệ thống lớn lên. Một người dùng đăng nhập ở máy thứ nhất, rồi request tiếp theo rơi vào máy thứ hai, và máy thứ hai không biết người đó là ai. Chúng ta sẽ thấy hiện tượng người dùng bị đăng xuất ngẫu nhiên, và hiện tượng đó rất khó tái hiện khi gỡ lỗi. Ngoài ra, chúng ta không thể triển khai kiểu cuốn chiếu một cách an toàn, vì tắt một máy là mất trạng thái đang nằm trên máy đó. Vì vậy, câu hỏi đầu tiên khi nhìn một dịch vụ cần mở rộng luôn là: trạng thái đang nằm ở đâu.

**English (bám cấu trúc tiếng Việt)**

A stateless service is a service in which each instance does not keep the user's session state in its own memory. Thanks to that, any instance can serve any request, and the load balancer is free to send a request wherever it wants. The state still exists, but we push it outside, usually into Redis or into the database. This is a precondition for horizontal scaling, not an optimisation for elegance. Only when the instances are identical in terms of state can we add or remove machines freely.

If we store the state in the memory of the application process, the consequence appears as soon as the system grows. A user logs in on the first machine, then the next request lands on the second machine, and the second machine does not know who that person is. We will see users being logged out at random, and that symptom is very hard to reproduce while debugging. In addition, we cannot do a rolling deployment safely, because shutting down one machine means losing the state sitting on that machine. Therefore, the first question when we look at a service that needs to scale is always: where does the state live.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| trong bộ nhớ của chính nó | in its own memory |
| bất kỳ instance nào cũng phục vụ được bất kỳ request nào | any instance can serve any request |
| được tự do gửi request đi đâu tuỳ ý | is free to send a request wherever it wants |
| chúng ta đẩy nó ra ngoài | we push it outside |
| điều kiện tiên quyết | a precondition |
| một tối ưu hoá cho đẹp | an optimisation for elegance |
| giống hệt nhau về mặt trạng thái | identical in terms of state |
| bị đăng xuất ngẫu nhiên | logged out at random |
| rất khó tái hiện khi gỡ lỗi | very hard to reproduce while debugging |
| triển khai kiểu cuốn chiếu | a rolling deployment |
| trạng thái đang nằm ở đâu | where does the state live |

**Thuật ngữ cần nhớ**

- không giữ trạng thái → **stateless**
- trạng thái phiên → **session state**
- kho phiên bên ngoài → **an external session store**
- điều kiện tiên quyết → **a precondition**
- triển khai cuốn chiếu → **a rolling deployment**

---

## Phần 3 — Vì sao chúng ta tránh sticky session

**Tiếng Việt**

Sticky session nghĩa là bộ cân bằng tải ghim một người dùng vào một instance cố định, để các request sau của người đó luôn đi về đúng máy cũ. Cách này nhìn qua thì tiện, vì chúng ta không cần đẩy session ra ngoài. Nhưng nó gây ra ba vấn đề. Thứ nhất, nó cản trở việc cân bằng lại tải, vì bộ cân bằng tải không được tự do chuyển người dùng sang máy khác. Thứ hai, khi một node chết, toàn bộ session đang nằm trên node đó sẽ mất. Thứ ba, tải sẽ bị lệch, vì có máy nhận nhiều người dùng nặng còn có máy thì nhàn rỗi. Vì ba lý do trên, cách làm được ưa chuộng hiện nay là đẩy session ra một kho bên ngoài, thường là Redis, hoặc dùng token tự chứa như JWT.

Có một trường hợp mà sticky session vẫn còn hợp lý, và chúng ta nên biết để trả lời cho cân bằng. Khi hệ thống là một ứng dụng cũ mà chúng ta chưa thể sửa, ghim phiên là cách vá tạm rẻ nhất. Tuy nhiên, chúng ta phải nói rõ rằng đây là nợ kỹ thuật chứ không phải là thiết kế đúng. Nếu chúng ta để nguyên cách này khi hệ thống lớn lên, hậu quả là mỗi lần triển khai đều làm rơi phiên của một phần người dùng. Vì vậy, chúng ta nên coi việc đẩy phiên ra ngoài là mặc định, và coi việc ghim phiên là ngoại lệ có thời hạn.

**English (bám cấu trúc tiếng Việt)**

A sticky session means that the load balancer pins one user to a fixed instance, so that the later requests of that person always go back to the same machine. At first glance this approach looks convenient, because we do not need to push the session out. But it causes three problems. First, it gets in the way of rebalancing the load, because the load balancer is not free to move a user to another machine. Second, when a node dies, all the sessions sitting on that node will be lost. Third, the load will become uneven, because some machines take many heavy users while other machines sit idle. For the three reasons above, the preferred approach nowadays is to push the session out to an external store, usually Redis, or to use a self-contained token such as JWT.

There is one case where a sticky session is still reasonable, and we should know it so that our answer stays balanced. When the system is a legacy application that we cannot fix yet, pinning the session is the cheapest temporary patch. However, we must say clearly that this is technical debt rather than a correct design. If we leave this approach in place as the system grows, the consequence is that every deployment drops the sessions of a portion of the users. Therefore, we should treat pushing the session out as the default, and treat pinning the session as an exception with an expiry date.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| ghim một người dùng vào một instance cố định | pins one user to a fixed instance |
| nhìn qua thì tiện | at first glance this looks convenient |
| cản trở việc cân bằng lại tải | gets in the way of rebalancing the load |
| không được tự do chuyển | is not free to move |
| tải sẽ bị lệch | the load will become uneven |
| máy thì nhàn rỗi | machines sit idle |
| vì ba lý do trên | for the three reasons above |
| token tự chứa | a self-contained token |
| cách vá tạm rẻ nhất | the cheapest temporary patch |
| làm rơi phiên của một phần người dùng | drops the sessions of a portion of the users |
| ngoại lệ có thời hạn | an exception with an expiry date |

**Thuật ngữ cần nhớ**

- ghim phiên → **a sticky session**
- cân bằng lại → **to rebalance**
- tải bị lệch → **uneven load** / **load skew**
- nhàn rỗi → **idle**
- ứng dụng cũ → **a legacy application**
- nợ kỹ thuật → **technical debt**

---

## Phần 4 — Bộ cân bằng tải tầng 4 và tầng 7

**Tiếng Việt**

Bộ cân bằng tải tầng bốn định tuyến dựa trên địa chỉ IP và cổng, nên nó rất nhanh nhưng nó không hiểu nội dung của request. Bộ cân bằng tải tầng bảy hiểu giao thức HTTP, nên nó định tuyến được theo đường dẫn hoặc theo header. Ngoài ra, tầng bảy còn kết thúc TLS giúp chúng ta, thử lại request hỏng, và ghim phiên theo cookie nếu chúng ta thật sự cần. Đổi lại, tầng bảy tốn nhiều tài nguyên hơn và thêm một chút độ trễ vào mỗi request. Vì vậy chúng ta chọn tầng bảy khi cần định tuyến thông minh hoặc cần giảm tải TLS, và chọn tầng bốn khi chúng ta chỉ cần thông lượng thuần.

Cách trả lời tốt trong phỏng vấn là chúng ta gắn lựa chọn này với một nhu cầu cụ thể, chứ không đọc thuộc lòng định nghĩa. Ví dụ, chúng ta nói rằng chúng ta cần tách đường dẫn của API và đường dẫn của ảnh sang hai cụm khác nhau, vì vậy chúng ta dùng tầng bảy. Hoặc chúng ta nói rằng đây là dịch vụ chơi game dùng giao thức riêng và cần độ trễ thấp nhất, vì vậy chúng ta dùng tầng bốn. Nếu chúng ta chọn sai tầng, hậu quả thường không phải là hệ thống chết, mà là chúng ta phải nhét logic định tuyến vào trong mã ứng dụng. Đó là chỗ mà một quyết định hạ tầng sai biến thành nợ kỹ thuật nằm rải khắp mã nguồn.

**English (bám cấu trúc tiếng Việt)**

A layer four load balancer routes based on the IP address and the port, so it is very fast but it does not understand the content of the request. A layer seven load balancer understands the HTTP protocol, so it can route by path or by header. In addition, layer seven can also terminate TLS for us, retry failed requests, and pin sessions by cookie if we really need that. In exchange, layer seven consumes more resources and adds a little latency to each request. Therefore we choose layer seven when we need smart routing or when we need to offload TLS, and we choose layer four when we only need raw throughput.

The good way to answer in an interview is that we tie this choice to a concrete need, rather than reciting the definition from memory. For example, we say that we need to split the API path and the image path into two different clusters, therefore we use layer seven. Or we say that this is a game service using its own protocol and needing the lowest possible latency, therefore we use layer four. If we pick the wrong layer, the consequence is usually not that the system dies, but that we have to stuff the routing logic into the application code. That is the place where a wrong infrastructure decision turns into technical debt scattered all over the codebase.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| định tuyến dựa trên địa chỉ IP và cổng | routes based on the IP address and the port |
| không hiểu nội dung của request | does not understand the content of the request |
| định tuyến được theo đường dẫn hoặc theo header | can route by path or by header |
| kết thúc TLS giúp chúng ta | terminate TLS for us |
| thử lại request hỏng | retry failed requests |
| tốn nhiều tài nguyên hơn | consumes more resources |
| cần giảm tải TLS | need to offload TLS |
| chỉ cần thông lượng thuần | only need raw throughput |
| đọc thuộc lòng định nghĩa | reciting the definition from memory |
| nhét logic định tuyến vào trong mã ứng dụng | stuff the routing logic into the application code |
| nợ kỹ thuật nằm rải khắp mã nguồn | technical debt scattered all over the codebase |

**Thuật ngữ cần nhớ**

- định tuyến → **to route** / **routing**
- kết thúc TLS → **TLS termination**
- giảm tải TLS → **to offload TLS**
- thông lượng → **throughput**
- mã nguồn → **the codebase**

---

## Phần 5 — Ba thuật toán phân phối và cái bẫy của round-robin

**Tiếng Việt**

Bộ cân bằng tải cần một luật để quyết định request tiếp theo đi về máy nào, và chúng ta nên thuộc ba luật phổ biến. Luật thứ nhất là xoay vòng, tức là mỗi máy nhận một request theo thứ tự, và luật này đơn giản nhất. Luật thứ hai là ít kết nối nhất, tức là request mới đi về máy đang giữ ít kết nối nhất. Luật thứ ba là băm theo khoá, tức là chúng ta lấy một khoá như mã người dùng rồi băm ra để chọn máy, nhờ đó cùng một người luôn về cùng một máy. Luật thứ ba giữ được tính cục bộ của cache, nên nó hữu ích khi mỗi máy có cache riêng theo người dùng.

Có một câu nói sai mà người phỏng vấn rất hay dùng để thử chúng ta: dùng xoay vòng thì tải đã chia đều rồi. Câu đó sai vì xoay vòng chia đều số lượng request, chứ không chia đều công sức xử lý. Trong thực tế, các request dài ngắn khác nhau rất nhiều, ví dụ một truy vấn báo cáo nặng hơn một lần đọc hồ sơ hàng trăm lần. Vì vậy, máy nào nhận phải toàn request nặng vẫn quá tải trong khi các máy khác vẫn rảnh. Với loại tải không đều như vậy, chúng ta nên dùng luật ít kết nối nhất, hoặc dùng một biến thể có trọng số theo sức máy.

**English (bám cấu trúc tiếng Việt)**

A load balancer needs a rule to decide which machine the next request goes to, and we should know three common rules by heart. The first rule is round robin, that is, each machine takes one request in turn, and this rule is the simplest one. The second rule is least connections, that is, a new request goes to the machine that currently holds the fewest connections. The third rule is hashing by key, that is, we take a key such as the user id and hash it to pick a machine, so that the same person always goes to the same machine. The third rule preserves cache locality, so it is useful when each machine has its own per-user cache.

There is a wrong statement that interviewers very often use to test us: if we use round robin then the load is already spread evenly. That statement is wrong because round robin spreads the number of requests evenly, but it does not spread the processing effort evenly. In practice, requests differ a lot in length, for example one reporting query is hundreds of times heavier than one profile read. Therefore, whichever machine happens to take all the heavy requests is still overloaded while the other machines stay free. For this kind of uneven load, we should use the least-connections rule, or use a weighted variant based on machine capacity.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| một luật để quyết định request tiếp theo đi về máy nào | a rule to decide which machine the next request goes to |
| mỗi máy nhận một request theo thứ tự | each machine takes one request in turn |
| máy đang giữ ít kết nối nhất | the machine that currently holds the fewest connections |
| băm theo khoá | hashing by key |
| giữ được tính cục bộ của cache | preserves cache locality |
| người phỏng vấn rất hay dùng để thử chúng ta | interviewers very often use to test us |
| chia đều số lượng request, chứ không chia đều công sức xử lý | spreads the number of requests evenly, but does not spread the processing effort evenly |
| nặng hơn … hàng trăm lần | hundreds of times heavier than … |
| máy nào nhận phải toàn request nặng | whichever machine happens to take all the heavy requests |
| một biến thể có trọng số theo sức máy | a weighted variant based on machine capacity |

**Thuật ngữ cần nhớ**

- xoay vòng → **round robin**
- ít kết nối nhất → **least connections**
- băm theo khoá → **hash by key**
- tính cục bộ của cache → **cache locality**
- quá tải → **overloaded**

---

## Phần 6 — Kiểm tra sức khoẻ và rút cạn kết nối trước khi tắt máy

**Tiếng Việt**

Bộ cân bằng tải phải liên tục kiểm tra sức khoẻ của từng máy phía sau, thường bằng cách gọi một đường dẫn nhẹ như healthz. Khi một máy trả lời sai hoặc không trả lời, bộ cân bằng tải ngừng gửi request tới máy đó cho đến khi nó khoẻ trở lại. Nhờ cơ chế này, một máy hỏng không kéo theo lỗi cho người dùng, vì lưu lượng tự động dồn sang các máy còn lại. Chúng ta nên để đường kiểm tra sức khoẻ kiểm tra đúng thứ cần thiết, ví dụ kết nối tới cơ sở dữ liệu, chứ không chỉ trả về mã 200 vô điều kiện. Nếu đường kiểm tra quá hời hợt, hậu quả là một máy đã hỏng bên trong vẫn được coi là khoẻ và vẫn tiếp tục nhận request.

Chiều ngược lại cũng quan trọng không kém, đó là lúc chúng ta chủ động tắt một máy. Khi chúng ta thu nhỏ cụm hoặc triển khai cuốn chiếu, máy sắp tắt vẫn đang phục vụ nhiều kết nối dở dang. Vì vậy chúng ta phải rút cạn kết nối trước, nghĩa là ngừng nhận request mới nhưng vẫn xử lý nốt request đang chạy. Chỉ khi các request cũ đã xong hoặc đã hết thời gian chờ, chúng ta mới thật sự tắt tiến trình. Nếu chúng ta tắt máy ngay lập tức, hậu quả là người dùng nhận về hàng loạt lỗi năm trăm đúng vào lúc chúng ta triển khai, và đội vận hành sẽ tin rằng bản phát hành mới có lỗi.

**English (bám cấu trúc tiếng Việt)**

The load balancer must continuously check the health of each machine behind it, usually by calling a light path such as healthz. When a machine answers incorrectly or does not answer, the load balancer stops sending requests to that machine until it becomes healthy again. Thanks to this mechanism, one broken machine does not turn into errors for the users, because the traffic automatically shifts to the remaining machines. We should let the health check path check the things that really matter, for example the connection to the database, rather than returning a 200 code unconditionally. If the health check is too shallow, the consequence is that a machine which is already broken inside is still considered healthy and still keeps taking requests.

The opposite direction matters just as much, and that is the moment when we shut a machine down on purpose. When we scale the cluster in or do a rolling deployment, the machine that is about to stop is still serving many in-flight connections. Therefore we must drain the connections first, which means stopping new requests but still finishing the requests that are already running. Only when the old requests have finished or have timed out do we actually kill the process. If we shut the machine down immediately, the consequence is that users get a burst of five hundred errors exactly while we are deploying, and the operations team will believe that the new release is broken.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| kiểm tra sức khoẻ của từng máy phía sau | check the health of each machine behind it |
| cho đến khi nó khoẻ trở lại | until it becomes healthy again |
| lưu lượng tự động dồn sang các máy còn lại | the traffic automatically shifts to the remaining machines |
| trả về mã 200 vô điều kiện | returning a 200 code unconditionally |
| quá hời hợt | too shallow |
| chiều ngược lại cũng quan trọng không kém | the opposite direction matters just as much |
| khi chúng ta thu nhỏ cụm | when we scale the cluster in |
| nhiều kết nối dở dang | many in-flight connections |
| rút cạn kết nối | drain the connections |
| đã hết thời gian chờ | have timed out |
| nhận về hàng loạt lỗi năm trăm | get a burst of five hundred errors |

**Thuật ngữ cần nhớ**

- kiểm tra sức khoẻ → **a health check**
- rút cạn kết nối → **connection draining**
- tắt êm → **a graceful shutdown**
- kết nối dở dang → **in-flight connections**
- thu nhỏ cụm → **to scale in**
- hết thời gian chờ → **to time out**

---

## Phần 7 — Chính bộ cân bằng tải cũng là một điểm chết duy nhất

**Tiếng Việt**

Sau khi đặt bộ cân bằng tải trước một cụm máy, chúng ta đã bỏ được điểm chết duy nhất ở tầng ứng dụng nhưng lại tạo ra một điểm chết mới ở tầng vào. Nếu chỉ có một bộ cân bằng tải, thì máy đó hỏng là toàn bộ hệ thống biến mất khỏi Internet, dù hàng chục máy phía sau vẫn chạy tốt. Vì vậy chúng ta phải làm cho chính tầng cân bằng tải có tính sẵn sàng cao. Có vài cách phổ biến: chạy nhiều bộ cân bằng tải rồi phân giải tên miền theo kiểu xoay vòng, dùng địa chỉ anycast để mạng tự chọn điểm gần nhất, dùng một địa chỉ IP nổi chuyển được giữa hai máy, hoặc chạy cặp chủ động và dự phòng. Nguyên tắc chung là không một thành phần nào được phép kéo sập cả hệ thống khi nó chết.

Trên môi trường đám mây, các dịch vụ cân bằng tải được quản lý đã lo phần này ở tầng dưới, nên chúng ta không phải tự dựng cặp máy. Tuy nhiên, chúng ta vẫn phải hiểu nguyên lý để chỉ huy và để kiểm tra thiết kế của người khác. Câu hỏi kiểm tra rất đơn giản: nếu thành phần này chết lúc hai giờ sáng thì điều gì xảy ra. Chúng ta nên đặt câu hỏi đó cho từng hộp trong bản vẽ, và mỗi hộp phải có một câu trả lời rõ ràng. Nếu có một hộp mà câu trả lời là toàn hệ thống ngừng, thì thiết kế đó chưa xong, dù nó trông rất đẹp trên bảng.

**English (bám cấu trúc tiếng Việt)**

After we put a load balancer in front of a cluster of machines, we have removed the single point of failure at the application layer but we have created a new one at the entry layer. If there is only one load balancer, then when that machine breaks the whole system disappears from the Internet, even though dozens of machines behind it are still running fine. Therefore we must make the load-balancing layer itself highly available. There are a few common ways: run several load balancers and resolve the domain name in a round-robin way, use an anycast address so that the network picks the nearest point itself, use a floating IP address that can move between two machines, or run an active-passive pair. The general principle is that no single component is allowed to bring down the whole system when it dies.

In a cloud environment, managed load-balancing services already handle this at a lower layer, so we do not have to build the pair of machines ourselves. However, we still have to understand the principle in order to lead and in order to review someone else's design. The review question is very simple: if this component dies at two in the morning, what happens. We should ask that question for every box in the drawing, and every box must have a clear answer. If there is one box whose answer is that the whole system stops, then that design is not finished, even though it looks very nice on the whiteboard.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| lại tạo ra một điểm chết mới ở tầng vào | we have created a new one at the entry layer |
| biến mất khỏi Internet | disappears from the Internet |
| dù hàng chục máy phía sau vẫn chạy tốt | even though dozens of machines behind it are still running fine |
| có tính sẵn sàng cao | highly available |
| phân giải tên miền theo kiểu xoay vòng | resolve the domain name in a round-robin way |
| để mạng tự chọn điểm gần nhất | so that the network picks the nearest point itself |
| một địa chỉ IP nổi chuyển được giữa hai máy | a floating IP address that can move between two machines |
| cặp chủ động và dự phòng | an active-passive pair |
| không được phép kéo sập cả hệ thống | is not allowed to bring down the whole system |
| để chỉ huy và để kiểm tra thiết kế của người khác | in order to lead and to review someone else's design |
| thiết kế đó chưa xong | that design is not finished |

**Thuật ngữ cần nhớ**

- tính sẵn sàng cao → **high availability (HA)**
- xoay vòng theo DNS → **DNS round robin**
- địa chỉ anycast → **an anycast address**
- IP nổi → **a floating IP**
- chuyển đổi dự phòng → **failover**
- dịch vụ được quản lý → **a managed service**

---

## Mô hình ghi nhớ

**Tiếng Việt**

Không giữ trạng thái cộng với bộ cân bằng tải thì bằng mở rộng ngang. Chúng ta đẩy trạng thái ra ngoài, và chúng ta không để một node nào, kể cả bộ cân bằng tải, trở thành điểm chết duy nhất.

**English (bám cấu trúc tiếng Việt)**

Stateless plus a load balancer equals horizontal scaling. We push the state outside, and we do not let any node, including the load balancer, become a single point of failure.

---

## Bảng thuật ngữ tổng hợp

| Tiếng Việt | Tiếng Anh | Ghi chú phát âm |
|---|---|---|
| mở rộng dọc | vertical scaling | *vertical* — "**VƠ**-ti-cợl", trọng âm đầu |
| mở rộng ngang | horizontal scaling | ho-ri-**ZON**-tơl, trọng âm âm thứ **ba**, không nhấn đầu |
| trần phần cứng | the hardware ceiling | *ceiling* /ˈsiːlɪŋ/ — "SI-ling", âm đầu là "s" không phải "k" |
| điểm chết duy nhất | a single point of failure (SPOF) | |
| thời gian dừng | downtime | |
| chịu lỗi | fault tolerance | *tolerance* — "**TO**-lơ-rợns", trọng âm đầu |
| không giữ trạng thái | stateless | |
| trạng thái phiên | session state | |
| kho phiên bên ngoài | an external session store | |
| điều kiện tiên quyết | a precondition | |
| triển khai cuốn chiếu | a rolling deployment | |
| ghim phiên | a sticky session | |
| cân bằng lại | to rebalance | |
| tải bị lệch | uneven load / load skew | *uneven* — "an-**II**-vợn", trọng âm âm thứ hai |
| nhàn rỗi | idle | /ˈaɪdl/ — "AI-dợl", không đọc "i-đồ" |
| ứng dụng cũ | a legacy application | *legacy* — "**LE**-gơ-si", trọng âm đầu |
| nợ kỹ thuật | technical debt | *debt* /det/ — chữ **b câm**, đọc là "đet" |
| bộ cân bằng tải | a load balancer | |
| định tuyến | to route / routing | Anh đọc /ruːt/ — "rút"; Mỹ đọc /raʊt/. Với khách Anh, dùng /ruːt/ |
| kết thúc TLS | TLS termination | |
| giảm tải TLS | to offload TLS | |
| thông lượng | throughput | âm **th** /θ/ đầu, "THRU-put", không đọc thành "trú" |
| mã nguồn | the codebase | |
| xoay vòng | round robin | |
| ít kết nối nhất | least connections | *least* âm cuối **-st** phải bật ra |
| băm theo khoá | hash by key | |
| tính cục bộ của cache | cache locality | *cache* /kæʃ/ — đọc đúng như "cash", không đọc "ca-chê" |
| quá tải | overloaded | |
| có trọng số | weighted | /ˈweɪtɪd/ — "UÂY-tịd", chữ *gh* câm |
| kiểm tra sức khoẻ | a health check | *health* có âm **th** cuối /θ/, không thành "heo" |
| rút cạn kết nối | connection draining | |
| tắt êm | a graceful shutdown | *graceful* — "GRÂY-sfợl", có cụm **-sf** ở giữa |
| kết nối dở dang | in-flight connections | |
| thu nhỏ cụm | to scale in | |
| mở rộng cụm | to scale out | |
| hết thời gian chờ | to time out | |
| tính sẵn sàng cao | high availability (HA) | *availability* — a-vây-lơ-**BI**-li-ti, trọng âm âm thứ tư |
| xoay vòng theo DNS | DNS round robin | |
| địa chỉ anycast | an anycast address | "**E**-ni-cast", âm cuối **-st** phải bật ra |
| IP nổi | a floating IP | |
| chuyển đổi dự phòng | failover | |
| dịch vụ được quản lý | a managed service | *managed* /ˈmænɪdʒd/ — hai âm tiết, không đọc "ma-na-gịt" |
| máy chủ | host | âm cuối **-st** phải bật ra, không thành "hâu" |
| cụm máy | a cluster | |
| tự mở rộng | autoscaling | |

---

## Bài nói

> **Cách luyện:** bật ghi âm, nói **60–90 giây** cho mỗi đề, **không nhìn tài liệu** khi nói. Mỗi bài phải dùng ít nhất **sáu thuật ngữ** trong bảng trên. Nghe lại bản ghi và đánh dấu chỗ nào bạn phải dừng lại tìm từ — chính chỗ đó là cụm cần học lại từ bảng ánh xạ.

1. **Explain to a junior engineer** why we prefer horizontal scaling over vertical scaling, and what the service has to give up in exchange.

2. **A colleague says:** *"We use round robin, so the load is already spread evenly."* **Explain what is wrong with that**, and say which algorithm you would use instead and why.

3. **Someone on your team wants to** turn on sticky sessions so that they can keep the login state in the instance memory. **Explain why you would push back**, and describe the two options you would offer them instead.

4. **Describe what happens when** one instance dies in the middle of serving traffic, first in a system with an external session store and then in a system with sticky sessions.

5. **Describe what happens when** a rolling deployment kills a machine without draining its connections, and explain what the operations team will see in the error dashboard.

6. **When would you choose** a layer four load balancer over a layer seven one? Give one concrete service for each choice and state the trade-off.

7. **Explain to a manager** why putting a load balancer in front of the cluster does not remove every single point of failure by itself, and what you would add to fix that.
