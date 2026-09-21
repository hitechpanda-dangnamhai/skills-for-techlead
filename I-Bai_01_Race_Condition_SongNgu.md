# Bài 1 — Race condition & 4 chiến lược chống race
### Bản song ngữ: tiếng Anh bám cấu trúc tiếng Việt

> **Cách dùng bài này:** đọc đoạn tiếng Việt trước → tự dịch trong đầu, không nhìn xuống → so với đoạn tiếng Anh bên dưới → **đọc to bản tiếng Anh hai lần**: lần một đọc chậm cho đúng từng âm, lần hai đọc trôi như đang nói với người phỏng vấn.

---

## ① Race condition là gì và khoảng trống đọc–ghi

**Tiếng Việt**

Race condition là tình huống mà kết quả cuối cùng phụ thuộc vào thứ tự và thời điểm của các thao tác chạy đồng thời trên cùng một tài nguyên. Chúng ta hãy hình dung hai nhân viên cùng nhìn vào một tờ giấy ghi "còn một vé". Cả hai đều thấy còn vé, cả hai đều gạch đi và bán, và cuối cùng chúng ta bán hai vé cho một chỗ ngồi. Vấn đề không nằm ở thao tác ghi, mà nằm ở khoảng trống giữa lúc đọc và lúc ghi. Trong khoảng trống đó, dữ liệu mà chúng ta đang cầm trên tay đã trở nên cũ. Nếu chúng ta không bịt khoảng trống này, hệ thống sẽ chạy đúng trong hầu hết thời gian và chỉ sai dưới tải cao. Hậu quả là chúng ta bán quá tồn kho, tiền không khớp sổ, và bug rất khó tái hiện.

**English (bám cấu trúc tiếng Việt)**

A race condition is a situation where the final result depends on the order and the timing of operations that run at the same time on the same resource. Let us picture two staff members looking at one piece of paper that says "one ticket left". Both of them see that a ticket is still there, both of them cross it out and sell it, and in the end we sell two tickets for one seat. The problem does not lie in the write operation, but it lies in the gap between the moment of reading and the moment of writing. Inside that gap, the data that we are holding in our hand has already become stale. If we do not close this gap, the system will run correctly most of the time and will only go wrong under high load. The consequence is that we oversell our stock, the money does not match the books, and the bug is very hard to reproduce.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| kết quả cuối cùng phụ thuộc vào | the final result depends on |
| chạy đồng thời trên cùng một tài nguyên | run at the same time on the same resource |
| chúng ta hãy hình dung | let us picture |
| gạch đi và bán | cross it out and sell it |
| khoảng trống giữa lúc đọc và lúc ghi | the gap between the moment of reading and the moment of writing |
| dữ liệu đang cầm trên tay đã trở nên cũ | the data that we are holding in our hand has already become stale |
| bịt khoảng trống này | close this gap |
| chạy đúng trong hầu hết thời gian | run correctly most of the time |
| tiền không khớp sổ | the money does not match the books |
| rất khó tái hiện | very hard to reproduce |

**Thuật ngữ cần nhớ**

- tài nguyên chia sẻ → **shared resource**
- khoảng trống đọc–ghi → **the read-write gap**
- dữ liệu cũ → **stale data**
- bán quá tồn kho → **to oversell**
- tái hiện lại bug → **to reproduce a bug**

---

## ② Vì sao Node chạy một luồng mà vẫn dính race

**Tiếng Việt**

Nhiều người tin rằng Node chạy một luồng nên không thể có race condition, và đây là hiểu lầm phổ biến nhất trong chủ đề này. JavaScript đúng là chạy trên một luồng duy nhất, nhưng mỗi lần chúng ta viết `await`, hàm đang chạy sẽ nhả quyền điều khiển về cho event loop. Trong lúc request A đang chờ database trả về giá trị stock, event loop hoàn toàn có thể chạy request B, và request B sẽ đọc đúng cái giá trị cũ mà A vừa đọc. Vì vậy race ở đây không phải là hai luồng tranh nhau một biến trong RAM như ở Java hoặc Go. Race ở đây xảy ra trên dữ liệu chia sẻ nằm ngoài tiến trình, thường là một hàng trong database hoặc một khóa trong Redis. Nếu chúng ta hiểu sai điểm này, chúng ta sẽ đi tìm bug ở tầng ngôn ngữ trong khi bug nằm ở tầng dữ liệu.

**English (bám cấu trúc tiếng Việt)**

Many people believe that Node runs on one thread and therefore cannot have a race condition, and this is the most common misunderstanding in this topic. JavaScript does indeed run on a single thread, but every time we write `await`, the running function gives up control back to the event loop. While request A is waiting for the database to return the stock value, the event loop can perfectly well run request B, and request B will read exactly the same old value that A has just read. Therefore the race here is not two threads fighting over a variable in RAM as in Java or Go. The race here happens on shared data that sits outside the process, usually a row in the database or a key in Redis. If we get this point wrong, we will go looking for the bug at the language level while the bug sits at the data level.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| hiểu lầm phổ biến nhất trong chủ đề này | the most common misunderstanding in this topic |
| đúng là chạy trên một luồng duy nhất | does indeed run on a single thread |
| nhả quyền điều khiển về cho | gives up control back to |
| hoàn toàn có thể chạy | can perfectly well run |
| đúng cái giá trị cũ mà A vừa đọc | exactly the same old value that A has just read |
| hai luồng tranh nhau một biến trong RAM | two threads fighting over a variable in RAM |
| nằm ngoài tiến trình | sits outside the process |
| nếu chúng ta hiểu sai điểm này | if we get this point wrong |
| đi tìm bug ở tầng ngôn ngữ | go looking for the bug at the language level |
| trong khi bug nằm ở tầng dữ liệu | while the bug sits at the data level |

**Thuật ngữ cần nhớ**

- một luồng duy nhất → **a single thread**
- nhả quyền điều khiển → **to give up control** / **to yield control**
- dữ liệu chia sẻ ngoài tiến trình → **out-of-process shared data**
- một hàng trong database → **a row in the database**
- vòng lặp sự kiện → **the event loop**

---

## ③ Check-then-act và bẫy TOCTOU

**Tiếng Việt**

Dạng race hay gặp nhất trong code nghiệp vụ có tên là check-then-act, và tên học thuật của nó là TOCTOU, viết tắt của *Time-Of-Check to Time-Of-Use*. Chúng ta kiểm tra một điều kiện trước, ví dụ như `if stock >= 1`, và sau đó chúng ta hành động, ví dụ như trừ đi một đơn vị stock. Vấn đề là giữa lúc kiểm tra và lúc hành động, state có thể đã bị người khác thay đổi, cho nên điều kiện mà chúng ta vừa kiểm tra đã hết đúng. Hai request cùng đọc `stock` bằng một, cả hai cùng thấy điều kiện thỏa mãn, và cả hai cùng trừ đi một. Kết quả là chúng ta bán hai đơn hàng trong khi kho chỉ còn một sản phẩm, và `stock` trong database tụt xuống số âm. Nếu chúng ta để lỗi này lọt lên production, đội vận hành sẽ phải hủy đơn thủ công và hoàn tiền cho khách, và đó là loại chi phí không bao giờ được ghi vào ticket.

**English (bám cấu trúc tiếng Việt)**

The most common form of race in business code is called check-then-act, and its academic name is TOCTOU, short for *Time-Of-Check to Time-Of-Use*. We check a condition first, for example `if stock >= 1`, and after that we act, for example subtract one unit of stock. The problem is that between the moment of checking and the moment of acting, the state may have been changed by somebody else, therefore the condition that we have just checked is no longer true. Two requests both read `stock` equal to one, both of them see that the condition is satisfied, and both of them subtract one. The result is that we sell two orders while the warehouse has only one product left, and the `stock` in the database drops below zero. If we let this bug slip through to production, the operations team will have to cancel the orders by hand and refund the customers, and that is the kind of cost that never gets written into a ticket.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| dạng race hay gặp nhất trong code nghiệp vụ | the most common form of race in business code |
| viết tắt của | short for |
| và sau đó chúng ta hành động | and after that we act |
| có thể đã bị người khác thay đổi | may have been changed by somebody else |
| điều kiện vừa kiểm tra đã hết đúng | the condition that we have just checked is no longer true |
| cùng thấy điều kiện thỏa mãn | both see that the condition is satisfied |
| tụt xuống số âm | drops below zero |
| để lỗi này lọt lên production | let this bug slip through to production |
| hủy đơn thủ công | cancel the orders by hand |
| không bao giờ được ghi vào ticket | never gets written into a ticket |

**Thuật ngữ cần nhớ**

- kiểm-tra-rồi-hành-động → **check-then-act**
- điều kiện không còn đúng nữa → **the condition no longer holds**
- đội vận hành → **the operations team**
- hoàn tiền cho khách → **to refund the customers**

---

## ④ Atomic conditional update so với đọc-sửa-ghi

**Tiếng Việt**

Cách chữa gọn nhất cho check-then-act là đẩy điều kiện xuống database, thay vì giữ nó ở tầng ứng dụng. Chúng ta viết một câu lệnh duy nhất dạng `UPDATE products SET stock = stock - 1 WHERE id = $1 AND stock > 0`. Câu lệnh này an toàn vì database tự khóa hàng đó trong đúng một thao tác, cho nên không còn khoảng trống để ai chen vào giữa. Sau khi chạy, chúng ta đọc số hàng bị ảnh hưởng: nếu bằng một thì trừ thành công, còn nếu bằng không thì hàng đã hết và chúng ta trả lỗi cho người dùng. Cách này không cần lock toàn cục, vì vậy thông lượng vẫn cao ngay cả khi mười nghìn người cùng bấm mua. Đây chính là lý do các hệ thống bán vé thường đẩy quyết định vào database thay vì tự quyết ở tầng ứng dụng.

Ngược lại, kiểu đọc-sửa-ghi ở tầng ứng dụng mở toang cửa cho lost update. Chúng ta đọc giá trị ra RAM, chúng ta tính toán trên bản sao đó, và sau đó chúng ta ghi đè kết quả trở lại database. Nếu hai request cùng đọc một giá trị, request nào ghi sau sẽ xóa sạch kết quả của request ghi trước. Người dùng không nhận được thông báo lỗi nào cả, vì cả hai câu lệnh ghi đều thành công về mặt kỹ thuật. Vì vậy lost update là loại bug im lặng, và chúng ta chỉ phát hiện ra khi đối soát số liệu cuối tháng.

**English (bám cấu trúc tiếng Việt)**

The neatest fix for check-then-act is to push the condition down into the database, instead of keeping it at the application layer. We write a single statement in the form `UPDATE products SET stock = stock - 1 WHERE id = $1 AND stock > 0`. This statement is safe because the database locks that row by itself inside exactly one operation, therefore there is no gap left for anybody to slip into the middle. After it runs, we read the number of affected rows: if it equals one then the subtraction has succeeded, and if it equals zero then the item is sold out and we return an error to the user. This approach does not need a global lock, therefore the throughput stays high even when ten thousand people click buy at the same time. This is exactly the reason why ticketing systems usually push the decision into the database instead of deciding by themselves at the application layer.

On the other hand, the read-modify-write style at the application layer opens the door wide for lost update. We read the value into RAM, we compute on that copy, and after that we write the result back over the database. If two requests read the same value, whichever request writes later will wipe out the result of the request that wrote earlier. The user does not receive any error message at all, because both write statements succeed from a technical point of view. Therefore lost update is a silent kind of bug, and we only discover it when we reconcile the numbers at the end of the month.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| cách chữa gọn nhất | the neatest fix |
| đẩy điều kiện xuống database | push the condition down into the database |
| tự khóa hàng đó trong đúng một thao tác | locks that row by itself inside exactly one operation |
| không còn khoảng trống để ai chen vào giữa | there is no gap left for anybody to slip into the middle |
| số hàng bị ảnh hưởng | the number of affected rows |
| hàng đã hết | the item is sold out |
| mở toang cửa cho | opens the door wide for |
| request nào ghi sau | whichever request writes later |
| xóa sạch kết quả của | wipe out the result of |
| thành công về mặt kỹ thuật | succeed from a technical point of view |
| đối soát số liệu cuối tháng | reconcile the numbers at the end of the month |

**Thuật ngữ cần nhớ**

- cập nhật có điều kiện, atomic → **atomic conditional update**
- đọc-sửa-ghi → **read-modify-write**
- mất bản cập nhật → **lost update**
- số hàng bị ảnh hưởng → **affected rows**
- lock toàn cục → **a global lock**
- thông lượng → **throughput**

---

## ⑤ Counter ra thiếu — triệu chứng kinh điển của lost update

**Tiếng Việt**

Một triệu chứng kinh điển của lost update là bài toán counter. Chúng ta bắn một nghìn request, mỗi request cộng thêm một, nhưng con số cuối cùng lại nhỏ hơn một nghìn. Thủ phạm gần như luôn luôn là counter được đặt ở tầng ứng dụng hoặc ở cache, và được cập nhật theo kiểu đọc-sửa-ghi. Cách sửa không phải là thêm một lock toàn cục, mà là dùng một thao tác tăng atomic: trong database chúng ta viết `col = col + 1`, còn trong Redis chúng ta gọi lệnh `INCR`. Khi phép cộng nằm gọn trong một thao tác, không request nào có thể đọc được giá trị nửa vời của request khác. Nếu chúng ta chọn lock toàn cục thay vì atomic increment, chúng ta sẽ trả giá bằng thông lượng mà không nhận thêm được chút tính đúng đắn nào.

**English (bám cấu trúc tiếng Việt)**

A classic symptom of lost update is the counter problem. We fire one thousand requests, each request adds one, but the final number comes out smaller than one thousand. The culprit is almost always a counter that is placed at the application layer or in the cache, and that is updated in the read-modify-write style. The fix is not to add a global lock, but to use an atomic increment operation: in the database we write `col = col + 1`, and in Redis we call the `INCR` command. When the addition sits neatly inside one operation, no request can read a half-finished value of another request. If we choose a global lock instead of an atomic increment, we will pay the price in throughput without gaining any extra correctness.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| một triệu chứng kinh điển của | a classic symptom of |
| chúng ta bắn một nghìn request | we fire one thousand requests |
| con số cuối cùng lại nhỏ hơn | the final number comes out smaller than |
| thủ phạm gần như luôn luôn là | the culprit is almost always |
| cách sửa không phải là … mà là … | the fix is not to … but to … |
| nằm gọn trong một thao tác | sits neatly inside one operation |
| giá trị nửa vời | a half-finished value |
| trả giá bằng thông lượng | pay the price in throughput |
| mà không nhận thêm được chút tính đúng đắn nào | without gaining any extra correctness |

**Thuật ngữ cần nhớ**

- tăng atomic → **atomic increment**
- thủ phạm → **the culprit**
- tính đúng đắn → **correctness**
- mức tranh chấp → **contention**

---

## ⑥ Ba sai lầm thường gặp khi chống race

**Tiếng Việt**

Sai lầm đầu tiên là thêm `sleep` hoặc retry mù để né race, và đây là một anti-pattern rõ ràng. Một bản vá dựa trên timing rất giòn, vì nó chỉ làm cho bug khó tái hiện hơn chứ không làm cho bug biến mất. Khi tải thay đổi hoặc khi mạng chậm hơn một chút, bug sẽ quay lại đúng như cũ. Sai lầm thứ hai là tin rằng code chạy tốt trên máy dev thì cũng sẽ chạy tốt trên production. Trên máy dev chỉ có một tiến trình và tải rất thấp, cho nên các request gần như đi tuần tự và chúng ta không bao giờ nhìn thấy race. Trên production chúng ta chạy nhiều instance song song, vì vậy concurrency mới thật sự xuất hiện.

Sai lầm thứ ba là cái bẫy nguy hiểm nhất của bài này: dùng một `Map` hoặc một biến đếm trong RAM để làm khóa. Cách này không chống được race giữa nhiều instance, bởi vì mỗi instance giữ state riêng của nó và các instance không nhìn thấy nhau. Ví dụ chúng ta đếm request trong RAM để giới hạn tốc độ, nhưng khi chạy trên bốn instance thì người dùng được phép gửi gấp bốn lần hạn mức. Nếu chúng ta muốn chống race across-instance, chúng ta phải có một shared store đóng vai trò điểm đồng bộ duy nhất, thường là Redis hoặc database. Điểm mấu chốt cần nhớ là race condition là thuộc tính của phạm vi tài nguyên, chứ không phải là thuộc tính của ngôn ngữ. Một biến local không bao giờ bảo vệ được một tài nguyên chia sẻ.

**English (bám cấu trúc tiếng Việt)**

The first mistake is to add a `sleep` or a blind retry in order to dodge the race, and this is a clear anti-pattern. A timing-based patch is very brittle, because it only makes the bug harder to reproduce and does not make the bug go away. When the load changes or when the network gets a little slower, the bug will come back exactly as before. The second mistake is to believe that code which runs well on the dev machine will also run well on production. On the dev machine there is only one process and the load is very low, therefore the requests go almost sequentially and we never see the race. On production we run many instances in parallel, therefore the concurrency truly shows up.

The third mistake is the most dangerous trap of this lesson: using a `Map` or a counter variable in RAM as a lock. This approach cannot stop a race between many instances, because each instance keeps its own state and the instances do not see each other. For example we count requests in RAM in order to limit the rate, but when we run on four instances the user is allowed to send four times the limit. If we want to stop a race across instances, we must have a shared store that plays the role of the single point of synchronisation, usually Redis or the database. The key point to remember is that a race condition is a property of the scope of the resource, and not a property of the language. A local variable can never protect a shared resource.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| retry mù để né race | a blind retry in order to dodge the race |
| một bản vá dựa trên timing rất giòn | a timing-based patch is very brittle |
| chỉ làm cho bug khó tái hiện hơn | only makes the bug harder to reproduce |
| bug sẽ quay lại đúng như cũ | the bug will come back exactly as before |
| các request gần như đi tuần tự | the requests go almost sequentially |
| concurrency mới thật sự xuất hiện | the concurrency truly shows up |
| cái bẫy nguy hiểm nhất của bài này | the most dangerous trap of this lesson |
| các instance không nhìn thấy nhau | the instances do not see each other |
| gấp bốn lần hạn mức | four times the limit |
| đóng vai trò điểm đồng bộ duy nhất | plays the role of the single point of synchronisation |
| là thuộc tính của phạm vi tài nguyên | is a property of the scope of the resource |

**Thuật ngữ cần nhớ**

- cách làm phản mẫu → **an anti-pattern**
- bản vá dựa trên thời điểm → **a timing-based patch**
- giòn, dễ vỡ → **brittle**
- kho dùng chung → **a shared store**
- điểm đồng bộ → **a synchronisation point**
- xuyên nhiều instance → **across instances**

---

## ⑦ Duplicate do double-submit và bốn chiến lược chống race

**Tiếng Việt**

Double-submit và double-click cũng là duplicate sinh ra từ concurrency, chứ không phải chỉ là lỗi của người dùng. Một request có thể bị gửi lại vì mạng chậm rồi client tự retry, hoặc vì người dùng mở hai tab và bấm ở cả hai nơi. Nhiều đội chặn lỗi này bằng cách disable nút submit trên giao diện, nhưng cách đó không cứu được gì. Giao diện không kiểm soát được retry của tầng mạng, không kiểm soát được nhiều tab, và cũng không kiểm soát được nhiều thiết bị. Chúng ta phải chặn ở phía server, bằng một idempotency key cộng với một unique constraint trong database. Khi request thứ hai đến, unique constraint sẽ từ chối nó, và chúng ta trả về đúng kết quả của lần đầu thay vì tạo thêm một đơn hàng.

Tổng kết lại, chúng ta có bốn chiến lược chống race, và việc của một tech lead là chọn đúng một cái cho từng tình huống. Chiến lược thứ nhất là atomic conditional write, phù hợp khi chúng ta chỉ sửa một trường đơn giản như stock hoặc counter. Chiến lược thứ hai là pessimistic lock, phù hợp khi conflict xảy ra thường xuyên và việc làm lại rất đắt. Chiến lược thứ ba là optimistic lock cộng với retry, phù hợp khi conflict hiếm và việc làm lại rẻ. Chiến lược thứ tư là unique constraint, phù hợp khi chúng ta cần bảo đảm một thứ chỉ được tạo đúng một lần. Chúng ta chọn dựa trên ba yếu tố: mức tranh chấp, chi phí của việc retry, và phạm vi của tài nguyên cần bảo vệ.

**English (bám cấu trúc tiếng Việt)**

A double submit and a double click are also duplicates that come out of concurrency, and not merely a mistake of the user. A request can be sent again because the network is slow and the client retries by itself, or because the user opens two tabs and clicks in both places. Many teams block this bug by disabling the submit button on the interface, but that way does not save anything. The interface does not control the retry of the network layer, does not control many tabs, and also does not control many devices. We have to block it on the server side, with an idempotency key plus a unique constraint in the database. When the second request arrives, the unique constraint will reject it, and we return exactly the result of the first time instead of creating one more order.

To sum up, we have four strategies against race, and the job of a tech lead is to choose exactly one of them for each situation. The first strategy is an atomic conditional write, which fits when we only change one simple field such as stock or a counter. The second strategy is a pessimistic lock, which fits when conflicts happen often and redoing the work is very expensive. The third strategy is an optimistic lock plus a retry, which fits when conflicts are rare and redoing the work is cheap. The fourth strategy is a unique constraint, which fits when we need to guarantee that a thing is created exactly once. We choose based on three factors: the level of contention, the cost of the retry, and the scope of the resource that we need to protect.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| chứ không phải chỉ là lỗi của người dùng | and not merely a mistake of the user |
| client tự retry | the client retries by itself |
| cách đó không cứu được gì | that way does not save anything |
| chặn ở phía server | block it on the server side |
| trả về đúng kết quả của lần đầu | return exactly the result of the first time |
| thay vì tạo thêm một đơn hàng | instead of creating one more order |
| việc của một tech lead là chọn đúng một cái | the job of a tech lead is to choose exactly one of them |
| phù hợp khi | which fits when |
| việc làm lại rất đắt | redoing the work is very expensive |
| bảo đảm một thứ chỉ được tạo đúng một lần | guarantee that a thing is created exactly once |
| phạm vi của tài nguyên cần bảo vệ | the scope of the resource that we need to protect |

**Thuật ngữ cần nhớ**

- gửi trùng hai lần → **a double submit**
- khóa idempotency → **an idempotency key**
- ràng buộc duy nhất → **a unique constraint**
- khóa bi quan → **a pessimistic lock**
- khóa lạc quan → **an optimistic lock**
- chi phí của việc thử lại → **the cost of the retry**

---

## Mô hình ghi nhớ

**Tiếng Việt**

Khoảng trống giữa lúc đọc và lúc ghi chính là nơi race sinh ra, cho nên chúng ta bịt nó bằng một thao tác atomic chứ không vá bằng timing. Và race condition là thuộc tính của phạm vi tài nguyên, cho nên một biến local không bao giờ bảo vệ được một tài nguyên chia sẻ.

**English (bám cấu trúc tiếng Việt)**

The gap between the moment of reading and the moment of writing is exactly where the race is born, therefore we close it with an atomic operation and we do not patch it with timing. And a race condition is a property of the scope of the resource, therefore a local variable can never protect a shared resource.

---

## Bảng thuật ngữ tổng hợp

| Tiếng Việt | Tiếng Anh | Ghi chú phát âm (Anh–Anh) |
|---|---|---|
| race condition | **race condition** | *kơn-DI-shợn* — trọng âm giữa, `-tion` = /ʃən/ |
| tài nguyên chia sẻ | **shared resource** | Anh: *ri-ZORS* /rɪˈzɔːs/ — trọng âm rơi vào âm thứ hai, khác Mỹ |
| khoảng trống đọc–ghi | **the read-write gap** | |
| dữ liệu cũ | **stale data** | *stale* = /steɪl/ "xtêi-l", không đọc "sta-le" |
| bán quá tồn kho | **to oversell** | |
| tái hiện bug | **to reproduce** | Anh: *rep-rơ-DYUUS* /riːprəˈdjuːs/ — có âm /dj/, không phải "du" |
| một luồng duy nhất | **a single thread** | *thread* /θred/ — âm **th** đầu lưỡi, không đọc thành "tred" |
| nhả quyền điều khiển | **to yield control** | *yield* /jiːld/ "yiil-d", cụm cuối **-ld** phải bật ra |
| vòng lặp sự kiện | **the event loop** | *event* = *i-VENT*, trọng âm âm thứ hai |
| tiến trình | **process** | Anh: */ˈprəʊses/* "PROU-ses", khác Mỹ "PRA-ses" |
| một hàng trong bảng | **a row** | /rəʊ/ "rou", không đọc thành "rao" |
| kiểm-tra-rồi-hành-động | **check-then-act** | có âm **th** ở *then* |
| điều kiện không còn đúng | **the condition no longer holds** | |
| kho hàng | **the warehouse** | *WAIR-haus* — trọng âm đầu, âm /w/ tròn môi |
| hoàn tiền | **to refund** | động từ = *ri-FAND*; danh từ = *RII-fand* — trọng âm đổi |
| cập nhật có điều kiện, atomic | **atomic conditional update** | *atomic* = *ơ-TOM-ik* /əˈtɒmɪk/ — trọng âm giữa, không phải "A-tô-mic" |
| đọc-sửa-ghi | **read-modify-write** | |
| mất bản cập nhật | **lost update** | |
| số hàng bị ảnh hưởng | **affected rows** | *rows* kết thúc /z/, phải bật ra |
| lock toàn cục | **a global lock** | |
| thông lượng | **throughput** | *THRUU-put* /ˈθruːpʊt/ — âm **th** + cụm **thr**, không đọc "tru-put" |
| bộ nhớ đệm | **cache** | /kæʃ/ đọc y hệt "cash", không đọc "ca-chê" |
| hàng đợi | **queue** | /kjuː/ đọc đúng như tên chữ cái **Q** |
| tăng atomic | **atomic increment** | *IN-krơ-mợnt* — trọng âm đầu |
| thủ phạm | **the culprit** | *KAL-prit* — trọng âm đầu |
| tính đúng đắn | **correctness** | cụm cuối **-ctness** khó, tập nói chậm: *kơ-REKT-nợs* |
| mức tranh chấp | **contention** | *kơn-TEN-shợn* — trọng âm giữa |
| cách làm phản mẫu | **an anti-pattern** | Anh: *AN-ti* /ˈænti/, **không** đọc "AN-tai" kiểu Mỹ |
| giòn, dễ vỡ | **brittle** | *BRI-tợl*, cụm **br** đầu phải rõ |
| kho dùng chung | **a shared store** | |
| điểm đồng bộ | **a synchronisation point** | *sing-krơ-nai-ZÊI-shợn* — trọng âm áp chót; Anh viết **-sation** |
| xuyên nhiều instance | **across instances** | *IN-stợn-siz*, trọng âm đầu |
| chạy song song | **in parallel** | *PA-rơ-lel* /ˈpærəlel/ — trọng âm đầu, ba âm tiết |
| tuần tự | **sequentially** | *si-KWEN-shợ-li* — trọng âm thứ hai, có âm /kw/ |
| biến cục bộ | **a local variable** | *VAIR-ri-ơ-bợl* — bốn âm tiết, trọng âm đầu |
| phạm vi | **the scope** | /skəʊp/ — cụm đầu **sk** và cụm cuối **-p** đều phải rõ |
| gửi trùng hai lần | **a double submit** | *submit* = *sợb-MIT*, trọng âm sau |
| khóa idempotency | **an idempotency key** | *ai-đem-PO-tơn-si* — trọng âm âm thứ ba, đây là từ hay đọc sai nhất |
| ràng buộc duy nhất | **a unique constraint** | *constraint* = *kơn-STRÊINT*, cụm **str** + cụm cuối **-nt** |
| khóa bi quan | **a pessimistic lock** | *pe-si-MIS-tik* — trọng âm áp chót |
| khóa lạc quan | **an optimistic lock** | *op-ti-MIS-tik* — trọng âm áp chót |
| chi phí thử lại | **the cost of the retry** | *retry* danh từ = *RII-trai* hoặc *ri-TRAI*, cả hai đều nghe được |
| đối soát số liệu | **to reconcile the numbers** | *RE-kợn-sail* /ˈrekənsaɪl/ — trọng âm đầu, đuôi đọc "sai-l" |
| đúng một lần | **exactly once** | *once* /wʌns/ — cụm cuối **-ns** phải bật ra |

---

## Bài nói

> **Cách luyện:** bật ghi âm, nói **60–90 giây** cho mỗi đề, **không nhìn tài liệu**. Mỗi bài nói phải dùng ít nhất **sáu thuật ngữ** trong bảng trên. Nói xong nghe lại một lần, đánh dấu chỗ nào bị vấp hoặc phát âm sai, rồi nói lại đề đó một lần nữa.

1. **Explain to a junior developer** why a Node.js service can still hit a race condition even though JavaScript runs on a single thread. Use the read-write gap and the event loop in your explanation.

2. **A colleague says:** *"I added `await sleep(50)` before the update and the duplicate orders stopped, so the bug is fixed."* Explain what is wrong with that reasoning and what you would ask them to do instead.

3. **Describe what happens**, step by step, when two requests both read `stock = 1` and then both write `stock = 0`. Name the failure mode, and explain why the user never sees an error message.

4. **Someone on your team wants to add a distributed lock in Redis** so that a global counter comes out right under load. Explain why you would push back, and what you would use instead.

5. **When would you choose an atomic conditional update over a pessimistic lock**, and when would you choose the other way round? Ground your answer in contention, retry cost and the scope of the resource.

6. **A team lead argues:** *"We already disable the submit button on the front end, so we do not need an idempotency key on the server."* Explain why you disagree, and describe the cases the front-end fix cannot cover.

7. **Explain to a product manager**, without SQL, why the checkout bug only appears on production and never on the staging environment, and why that does not mean the code is fine.

---

> 🔚 **Hết Bài 1.** Gõ `Làm Bài 2` để sang *Optimistic vs Pessimistic Locking*.
