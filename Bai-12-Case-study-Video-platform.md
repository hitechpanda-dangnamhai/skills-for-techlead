# Bài 12 — Case study: Video/feed platform quy mô lớn
### Bản song ngữ: tiếng Anh bám cấu trúc tiếng Việt

> **Cách dùng:** đọc đoạn tiếng Việt trước → tự dịch trong đầu → so với bản tiếng Anh bên dưới → **đọc to bản tiếng Anh hai lần**. Lần một đọc chậm cho đúng từ, lần hai đọc liền mạch như đang nói với người phỏng vấn. Bài này nhiều con số lớn, nên hãy đọc to cả phần ước lượng bằng tiếng Anh, đặc biệt là các đơn vị terabyte và petabyte.

---

## Phần 1 — Video là bài toán băng thông, không phải bài toán tính toán

**Tiếng Việt**

Điều đầu tiên chúng ta nên nói khi nhận đề này là phục vụ video cho hàng triệu người là một bài toán về băng thông. Máy chủ không phải nghĩ gì nhiều khi phát một video, nó chỉ phải đẩy ra một lượng byte khổng lồ một cách đều đặn. Vì vậy, cách đặt vấn đề đúng không phải là cần bao nhiêu CPU, mà là dữ liệu chảy ra từ đâu và đi qua đường nào. Nếu chúng ta phát mọi video từ một trung tâm dữ liệu duy nhất, đường ra của trung tâm đó sẽ nghẽn và người dùng ở xa sẽ chờ rất lâu. Ngoài ra, chi phí dữ liệu đi ra sẽ lớn tới mức không một mô hình kinh doanh nào chịu nổi.

Từ nhận định đó, xương sống của thiết kế hiện ra rất tự nhiên. Chúng ta đẩy nội dung ra rìa mạng, tức là ra các điểm biên gần người dùng, và chúng ta phục vụ phần lớn lưu lượng từ đó. Đồng thời, chúng ta chuẩn bị sẵn nhiều phiên bản của cùng một video, để máy khách chọn phiên bản phù hợp với đường truyền của họ. Hai ý này gộp lại thành một câu mà chúng ta nên thuộc: mạng phân phối nội dung đứng trước, và mọi thứ nặng đều được chuẩn bị từ trước. Nếu chúng ta bắt đầu bằng việc bàn về máy chủ ứng dụng, hậu quả là chúng ta dành thời gian cho phần chiếm chưa tới một phần trăm chi phí hạ tầng.

**English (bám cấu trúc tiếng Việt)**

The first thing we should say when we get this question is that serving video to millions of people is a bandwidth problem. The server does not have to think much when it plays a video, it only has to push out a huge amount of bytes steadily. Therefore, the correct way to frame the problem is not how much CPU we need, but where the data flows out from and which path it travels. If we serve every video from one single data centre, the outbound link of that data centre will choke and far-away users will wait a long time. In addition, the egress cost will grow so large that no business model can bear it.

From that observation, the backbone of the design appears very naturally. We push the content out to the edge of the network, that is, to the edge locations close to the users, and we serve most of the traffic from there. At the same time, we prepare several versions of the same video in advance, so that the client picks the version that suits their connection. These two ideas combine into one sentence that we should know by heart: the content delivery network comes first, and everything heavy is prepared in advance. If we start by discussing the application servers, the consequence is that we spend our time on the part that carries less than one percent of the infrastructure cost.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| máy chủ không phải nghĩ gì nhiều | the server does not have to think much |
| đẩy ra một lượng byte khổng lồ một cách đều đặn | push out a huge amount of bytes steadily |
| cách đặt vấn đề đúng | the correct way to frame the problem |
| dữ liệu chảy ra từ đâu và đi qua đường nào | where the data flows out from and which path it travels |
| đường ra của trung tâm đó sẽ nghẽn | the outbound link of that data centre will choke |
| không một mô hình kinh doanh nào chịu nổi | no business model can bear it |
| xương sống của thiết kế hiện ra rất tự nhiên | the backbone of the design appears very naturally |
| đẩy nội dung ra rìa mạng | push the content out to the edge of the network |
| phù hợp với đường truyền của họ | that suits their connection |
| mọi thứ nặng đều được chuẩn bị từ trước | everything heavy is prepared in advance |

**Thuật ngữ cần nhớ**

- băng thông → **bandwidth**
- chi phí dữ liệu đi ra → **egress cost**
- rìa mạng → **the edge of the network**
- máy khách → **the client**
- chuẩn bị trước → **to prepare in advance**

---

## Phần 2 — Ước lượng để thấy quy mô thật

**Tiếng Việt**

Chúng ta giả sử nền tảng nhận một triệu video mới mỗi ngày, và mỗi tệp gốc nặng khoảng một trăm megabyte. Riêng tệp gốc đã là một trăm terabyte mỗi ngày, tức là khoảng ba mươi sáu petabyte mỗi năm. Chúng ta còn phải tạo bốn phiên bản chất lượng khác nhau cho mỗi video, nên tổng dung lượng lưu trữ tăng thêm vài lần nữa. Con số này lớn tới mức chúng ta không thể nghĩ tới việc lưu chúng trong cơ sở dữ liệu, và kho đối tượng là lựa chọn duy nhất hợp lý. Đây là kết luận đầu tiên mà chúng ta nên nói ra sau khi tính xong.

Bây giờ chúng ta tính phần băng thông, vì đó mới là phần đắt nhất. Nếu mỗi ngày có mười triệu lượt xem và mỗi lượt tiêu thụ khoảng năm megabyte, chúng ta có năm mươi terabyte dữ liệu đi ra mỗi ngày. Chia cho tám mươi sáu nghìn bốn trăm giây, chúng ta được khoảng sáu trăm megabyte mỗi giây ở mức trung bình, và khoảng gấp ba lần con số đó ở giờ cao điểm. Với mức này, việc phục vụ trực tiếp từ máy chủ gốc là bất khả thi, nên mạng phân phối nội dung không còn là một lựa chọn mà là một điều kiện. Cách nói ghi điểm là chúng ta gắn ngay con số vào kết luận, thay vì đọc con số rồi để nó lơ lửng.

**English (bám cấu trúc tiếng Việt)**

We assume the platform receives one million new videos per day, and each master file weighs about one hundred megabytes. The master files alone are one hundred terabytes per day, that is, about thirty-six petabytes per year. We also have to create four different quality versions for each video, so the total storage grows by several times more. This number is so large that we cannot even think about storing them in a database, and object storage is the only sensible option. This is the first conclusion we should say out loud once we finish the calculation.

Now we calculate the bandwidth part, because that is the most expensive part. If there are ten million views per day and each view consumes about five megabytes, we have fifty terabytes of data going out per day. Dividing by eighty-six thousand four hundred seconds, we get about six hundred megabytes per second on average, and about three times that number at the peak hours. At this level, serving directly from the origin server is impossible, so the content delivery network is no longer an option but a condition. The way of speaking that scores is that we attach the number to a conclusion immediately, rather than reading out the number and leaving it hanging.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| mỗi tệp gốc nặng khoảng một trăm megabyte | each master file weighs about one hundred megabytes |
| riêng tệp gốc đã là | the master files alone are |
| tổng dung lượng lưu trữ tăng thêm vài lần nữa | the total storage grows by several times more |
| lớn tới mức chúng ta không thể nghĩ tới việc | so large that we cannot even think about |
| lựa chọn duy nhất hợp lý | the only sensible option |
| mỗi lượt tiêu thụ khoảng năm megabyte | each view consumes about five megabytes |
| là bất khả thi | is impossible |
| không còn là một lựa chọn mà là một điều kiện | is no longer an option but a condition |
| gắn ngay con số vào kết luận | attach the number to a conclusion immediately |
| để nó lơ lửng | leaving it hanging |

**Thuật ngữ cần nhớ**

- tệp gốc → **the master file**
- phiên bản chất lượng → **a quality variant**
- kho đối tượng → **object storage**
- tiêu thụ → **to consume**
- máy chủ gốc → **the origin server**

---

## Phần 3 — Tốc độ bit thích ứng

**Tiếng Việt**

Người dùng không xem video trên cùng một chất lượng đường truyền, nên chúng ta không thể phục vụ cho tất cả một tệp duy nhất. Giải pháp chuẩn của ngành là tốc độ bit thích ứng, và nó gồm hai ý đơn giản. Ý thứ nhất là chúng ta mã hoá trước cùng một video ở nhiều độ phân giải và nhiều tốc độ bit, tạo thành một thang chất lượng. Ý thứ hai là chúng ta cắt mỗi phiên bản thành những đoạn nhỏ, thường dài vài giây, rồi phát một tệp kê khai liệt kê tất cả các đoạn đó. Máy khách đọc tệp kê khai, đo tốc độ mạng hiện tại, rồi tự chọn đoạn tiếp theo ở mức chất lượng phù hợp.

Điểm hay của cách này là chất lượng có thể thay đổi giữa chừng mà không cần tải lại. Khi mạng của người dùng yếu đi, máy khách hạ chất lượng xuống thay vì để video dừng lại và quay vòng. Với người xem, một hình ảnh hơi mờ vẫn tốt hơn nhiều so với một vòng quay chờ, và đó chính là đánh đổi mà thiết kế này chọn. Chúng ta cũng nên nói rằng việc mã hoá nhiều phiên bản làm tăng chi phí lưu trữ và chi phí xử lý lúc tải lên. Nếu chúng ta chỉ giữ một phiên bản duy nhất để tiết kiệm, hậu quả là người dùng ở vùng mạng yếu sẽ bỏ đi sau vài giây chờ, và đó là nhóm người dùng đông nhất ở nhiều thị trường.

**English (bám cấu trúc tiếng Việt)**

Users do not watch video on the same connection quality, so we cannot serve one single file to everybody. The industry-standard solution is adaptive bitrate, and it consists of two simple ideas. The first idea is that we pre-encode the same video at several resolutions and several bitrates, forming a quality ladder. The second idea is that we cut each version into small segments, usually a few seconds long, and then publish a manifest file listing all of those segments. The client reads the manifest, measures the current network speed, and then picks the next segment at the appropriate quality level itself.

The nice thing about this approach is that the quality can change midway without needing a reload. When the user's network gets weaker, the client drops the quality down instead of letting the video stop and spin. For a viewer, a slightly blurry picture is still much better than a loading spinner, and that is exactly the trade-off this design chooses. We should also say that encoding several versions increases the storage cost and the processing cost at upload time. If we keep only one single version in order to save money, the consequence is that users on weak networks will leave after a few seconds of waiting, and that is the largest group of users in many markets.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| trên cùng một chất lượng đường truyền | on the same connection quality |
| giải pháp chuẩn của ngành | the industry-standard solution |
| mã hoá trước | to pre-encode |
| tạo thành một thang chất lượng | forming a quality ladder |
| cắt mỗi phiên bản thành những đoạn nhỏ | cut each version into small segments |
| một tệp kê khai liệt kê tất cả các đoạn | a manifest file listing all of the segments |
| thay đổi giữa chừng mà không cần tải lại | change midway without needing a reload |
| để video dừng lại và quay vòng | letting the video stop and spin |
| một hình ảnh hơi mờ | a slightly blurry picture |
| một vòng quay chờ | a loading spinner |

**Thuật ngữ cần nhớ**

- tốc độ bit thích ứng → **adaptive bitrate (ABR)**
- độ phân giải → **resolution**
- đoạn video → **a segment**
- tệp kê khai → **a manifest**
- thang chất lượng → **the bitrate ladder**

---

## Phần 4 — Đường ống từ lúc tải lên tới lúc sẵn sàng

**Tiếng Việt**

Đường ống xử lý gồm bốn bước, và chúng ta nên đọc chúng theo đúng thứ tự. Bước một, người dùng tải tệp gốc thẳng lên kho đối tượng bằng một đường dẫn có chữ ký, nên tệp không đi vòng qua máy chủ ứng dụng. Bước hai, chúng ta ghi một bản ghi siêu dữ liệu với trạng thái đang xử lý, rồi đẩy một thông điệp vào hàng đợi cho từng mức chất lượng cần tạo. Bước ba, một đàn máy xử lý phía sau lấy thông điệp ra, chuyển mã video sang mức chất lượng đó, rồi đẩy kết quả lên mạng phân phối nội dung. Bước bốn, khi tất cả các mức đã xong, chúng ta đổi trạng thái thành sẵn sàng và gửi thông báo cho người tải lên.

Toàn bộ đường ống này phải chạy bất đồng bộ, và đây là điểm mà nhiều ứng viên trả lời sai. Việc chuyển mã một video dài có thể mất nhiều phút hoặc thậm chí nhiều giờ, nên chúng ta không thể giữ một request HTTP mở suốt thời gian đó. Nếu chúng ta cố chờ chuyển mã xong rồi mới trả lời, request sẽ hết thời gian chờ và người dùng nhận về một lỗi dù video của họ hoàn toàn ổn. Cách đúng là chúng ta trả lời ngay với trạng thái đang xử lý, rồi báo cho người dùng khi mọi thứ sẵn sàng. Chúng ta cũng nên làm mỗi tác vụ chuyển mã bất biến khi lặp lại theo cặp mã video và mức chất lượng, vì hàng đợi có thể giao lại cùng một thông điệp.

**English (bám cấu trúc tiếng Việt)**

The processing pipeline has four steps, and we should read them out in the right order. Step one, the user uploads the master file straight into object storage using a signed URL, so the file does not go around through the application server. Step two, we write a metadata record with the status "processing", and then push one message into the queue for each quality level we need to create. Step three, a farm of workers at the back takes the messages out, transcodes the video into that quality level, and then pushes the result up to the content delivery network. Step four, when all the levels are done, we change the status to "ready" and send a notification to the uploader.

This whole pipeline has to run asynchronously, and this is the point where many candidates answer wrongly. Transcoding a long video may take many minutes or even many hours, so we cannot hold one HTTP request open for that whole time. If we try to wait for the transcoding to finish before replying, the request will time out and the user receives an error even though their video is perfectly fine. The correct way is that we reply immediately with the "processing" status, and then tell the user when everything is ready. We should also make each transcoding task idempotent on the pair of video id and quality level, because the queue may deliver the same message again.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| đọc chúng theo đúng thứ tự | read them out in the right order |
| tải tệp gốc thẳng lên kho đối tượng | upload the master file straight into object storage |
| không đi vòng qua máy chủ ứng dụng | does not go around through the application server |
| một đàn máy xử lý phía sau | a farm of workers at the back |
| chuyển mã video sang mức chất lượng đó | transcodes the video into that quality level |
| gửi thông báo cho người tải lên | send a notification to the uploader |
| giữ một request HTTP mở suốt thời gian đó | hold one HTTP request open for that whole time |
| request sẽ hết thời gian chờ | the request will time out |
| dù video của họ hoàn toàn ổn | even though their video is perfectly fine |
| có thể giao lại cùng một thông điệp | may deliver the same message again |

**Thuật ngữ cần nhớ**

- đường ống xử lý → **a processing pipeline**
- chuyển mã → **to transcode** / **transcoding**
- đàn máy xử lý → **a worker farm**
- đường dẫn có chữ ký → **a signed URL**
- bất biến khi lặp lại → **idempotent**

---

## Phần 5 — Ảnh đại diện và bản xem trước

**Tiếng Việt**

Ảnh đại diện của video phải được sinh trước ngay lúc tải lên, chứ không sinh ra tại từng request. Trong lúc chuyển mã, chúng ta trích vài khung hình ở những mốc thời gian định sẵn rồi lưu chúng ở nhiều kích thước khác nhau. Các tệp đó nằm trong kho đối tượng và được phục vụ qua mạng phân phối nội dung y hệt như các đoạn video. Nhờ vậy, một trang danh sách hiển thị năm mươi ảnh đại diện chỉ tốn năm mươi lần đọc từ cache biên, chứ không tốn một chút tính toán nào của chúng ta. Đây là một ví dụ rất rõ của nguyên tắc chung: việc gì làm được một lần thì đừng làm lại hàng triệu lần.

Nếu chúng ta sinh ảnh đại diện tại từng request, hậu quả xuất hiện rất nhanh khi lưu lượng tăng. Mỗi lần mở trang chủ, hệ thống phải giải mã video và cắt khung hình cho hàng chục mục, và việc đó tốn CPU gấp nhiều lần việc phục vụ một tệp tĩnh. Trang chủ là trang được mở nhiều nhất, nên chúng ta vô tình đặt phần tốn kém nhất vào đúng đường nóng nhất. Cách trình bày ghi điểm là chúng ta nói rằng chúng ta chuyển chi phí từ lúc đọc sang lúc ghi, vì mỗi video chỉ được tải lên một lần nhưng được xem hàng nghìn lần. Câu đó cũng chính là câu chúng ta đã dùng ở bài về rút gọn đường dẫn, chỉ khác đối tượng áp dụng.

**English (bám cấu trúc tiếng Việt)**

The video thumbnails have to be generated in advance right at upload time, rather than generated at each request. During the transcoding, we extract a few frames at predefined time marks and then store them at several different sizes. Those files sit in object storage and are served through the content delivery network exactly like the video segments. Thanks to that, a listing page showing fifty thumbnails only costs fifty reads from the edge cache, and it costs us no computation at all. This is a very clear example of the general principle: whatever can be done once should not be done a million times.

If we generate thumbnails at each request, the consequence appears very quickly as the traffic grows. Every time the home page opens, the system has to decode the video and cut frames for dozens of items, and that costs many times more CPU than serving a static file. The home page is the most frequently opened page, so we have accidentally put the most expensive part right on the hottest path. The way of presenting that scores is that we say we move the cost from read time to write time, because each video is uploaded once but watched thousands of times. That sentence is also exactly the sentence we used in the URL shortener lesson, only the subject it applies to is different.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| ảnh đại diện của video | the video thumbnails |
| trích vài khung hình ở những mốc thời gian định sẵn | extract a few frames at predefined time marks |
| y hệt như các đoạn video | exactly like the video segments |
| chỉ tốn năm mươi lần đọc từ cache biên | only costs fifty reads from the edge cache |
| việc gì làm được một lần thì đừng làm lại hàng triệu lần | whatever can be done once should not be done a million times |
| giải mã video và cắt khung hình | decode the video and cut frames |
| chúng ta vô tình đặt | we have accidentally put |
| vào đúng đường nóng nhất | right on the hottest path |
| chuyển chi phí từ lúc đọc sang lúc ghi | move the cost from read time to write time |
| chỉ khác đối tượng áp dụng | only the subject it applies to is different |

**Thuật ngữ cần nhớ**

- ảnh đại diện → **a thumbnail**
- khung hình → **a frame**
- sinh trước, ngoại tuyến → **offline generation**
- cache ở biên → **the edge cache**
- tệp tĩnh → **a static file**

---

## Phần 6 — Đếm lượt xem ở quy mô tỷ lượt

**Tiếng Việt**

Chúng ta không được tăng một cột trong cơ sở dữ liệu cho mỗi lượt xem, vì tất cả lượt xem của một video đều ghi vào đúng một dòng. Dòng đó trở thành một dòng nóng, mọi lượt ghi phải xếp hàng chờ khoá, và một video lan truyền sẽ tự tay làm chậm cả cơ sở dữ liệu. Cách làm đúng là chúng ta tăng một bộ đếm trong bộ nhớ nhanh, ví dụ Redis, và mỗi lượt xem chỉ tốn một thao tác nguyên tử rất rẻ. Sau đó, một tác vụ định kỳ chạy mỗi vài giây sẽ lấy giá trị tích luỹ và ghi gộp một lần xuống cơ sở dữ liệu. Nhờ vậy, một triệu lượt xem biến thành vài trăm lượt ghi thay vì một triệu lượt ghi.

Cách khác, phổ biến ở quy mô rất lớn, là đẩy mỗi lượt xem thành một sự kiện vào một luồng dữ liệu rồi tổng hợp ở phía sau. Cách này cho chúng ta thêm khả năng phân tích, ví dụ đếm theo quốc gia hoặc theo thiết bị, mà không phải sửa đường phục vụ. Dù chọn cách nào, chúng ta cũng phải chấp nhận rằng con số hiển thị là xấp xỉ và trễ vài giây. Đây là một đánh đổi rất dễ bảo vệ, vì không người xem nào phân biệt được giữa một triệu hai trăm nghìn và một triệu hai trăm lẻ ba nghìn lượt xem. Nếu chúng ta khăng khăng đòi con số chính xác tuyệt đối theo thời gian thực, hậu quả là chúng ta trả một cái giá hạ tầng rất lớn cho một thứ mà không ai kiểm chứng được.

**English (bám cấu trúc tiếng Việt)**

We must not increment a column in the database for each view, because all the views of one video write into exactly one row. That row becomes a hot row, every write has to queue up waiting for the lock, and one viral video will slow down the entire database with its own hands. The correct approach is that we increment a counter in fast memory, for example Redis, and each view costs only one very cheap atomic operation. Then, a periodic job running every few seconds takes the accumulated value and writes it down to the database in one combined write. Thanks to that, one million views turn into a few hundred writes instead of one million writes.

Another approach, common at very large scale, is to push each view as an event into a data stream and then aggregate it at the back. This approach gives us extra analytical power, for example counting by country or by device, without having to change the serving path. Whichever approach we choose, we also have to accept that the displayed number is approximate and a few seconds behind. This is a very easy trade-off to defend, because no viewer can tell the difference between one million two hundred thousand and one million two hundred and three thousand views. If we insist on an absolutely exact number in real time, the consequence is that we pay a very large infrastructure price for something nobody can verify.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| tăng một cột trong cơ sở dữ liệu | increment a column in the database |
| ghi vào đúng một dòng | write into exactly one row |
| một video lan truyền | one viral video |
| sẽ tự tay làm chậm cả cơ sở dữ liệu | will slow down the entire database with its own hands |
| lấy giá trị tích luỹ | takes the accumulated value |
| ghi gộp một lần | one combined write |
| đẩy mỗi lượt xem thành một sự kiện vào một luồng dữ liệu | push each view as an event into a data stream |
| mà không phải sửa đường phục vụ | without having to change the serving path |
| con số hiển thị là xấp xỉ | the displayed number is approximate |
| một đánh đổi rất dễ bảo vệ | a very easy trade-off to defend |
| một thứ mà không ai kiểm chứng được | something nobody can verify |

**Thuật ngữ cần nhớ**

- dòng nóng → **a hot row**
- bộ đếm xấp xỉ → **an approximate counter**
- ghi gộp định kỳ → **a periodic flush**
- luồng dữ liệu → **a data stream**
- tổng hợp → **to aggregate**

---

## Phần 7 — Bảng tin gợi ý khác bảng tin theo dõi ở chỗ nào

**Tiếng Việt**

Một bảng tin theo dõi lấy nội dung từ những tài khoản mà người dùng đã chọn, nên tập ứng viên nhỏ và đã được xác định sẵn. Một bảng tin gợi ý thì lấy nội dung từ toàn bộ kho video, nên tập ứng viên lớn hơn hàng triệu lần. Vì vậy, kiến trúc phải thêm một tầng chọn ứng viên đứng trước tầng xếp hạng. Tầng chọn ứng viên dùng những phép lọc rẻ để rút từ hàng triệu video xuống còn vài trăm, ví dụ theo chủ đề, theo độ mới, hoặc theo những video mà người dùng tương tự đã xem. Sau đó, tầng xếp hạng dùng một mô hình nặng hơn để chấm điểm vài trăm ứng viên đó và chọn ra vài chục video hiển thị.

Kiến trúc này kéo theo một hệ thống phục vụ mô hình học máy riêng, và nó có nhịp vận hành khác hẳn phần còn lại. Các đặc trưng của người dùng và của video phải được tính sẵn và lưu trong một kho đặc trưng để tầng xếp hạng đọc rất nhanh. Mô hình được huấn luyện lại theo chu kỳ, được đánh giá bằng thử nghiệm A và B, và được triển khai dần chứ không đổi một lượt. Chúng ta cũng cần một vòng phản hồi ghi lại hành vi xem để làm dữ liệu huấn luyện cho lần sau. Nếu chúng ta trình bày bảng tin gợi ý như thể nó chỉ là một truy vấn sắp xếp phức tạp hơn, hậu quả là chúng ta bỏ qua đúng phần tốn kém nhất và khó vận hành nhất của cả nền tảng.

**English (bám cấu trúc tiếng Việt)**

A follow feed takes content from the accounts the user has chosen, so the candidate set is small and already defined. A recommendation feed takes content from the entire video store, so the candidate set is millions of times larger. Therefore, the architecture has to add a candidate selection stage standing in front of the ranking stage. The candidate selection stage uses cheap filters to narrow millions of videos down to a few hundred, for example by topic, by recency, or by the videos that similar users have watched. Then, the ranking stage uses a heavier model to score those few hundred candidates and pick out the few dozen videos to display.

This architecture brings with it a separate machine learning serving system, and it has an operational rhythm quite different from the rest. The features of the users and of the videos have to be computed in advance and stored in a feature store so that the ranking stage can read them very fast. The model is retrained on a cycle, is evaluated with A and B tests, and is rolled out gradually rather than switched over all at once. We also need a feedback loop that records watching behaviour to serve as training data for the next round. If we present a recommendation feed as if it were merely a more complicated sorting query, the consequence is that we skip exactly the most expensive and hardest-to-operate part of the whole platform.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| tập ứng viên nhỏ và đã được xác định sẵn | the candidate set is small and already defined |
| lớn hơn hàng triệu lần | millions of times larger |
| đứng trước tầng xếp hạng | standing in front of the ranking stage |
| rút từ hàng triệu video xuống còn vài trăm | narrow millions of videos down to a few hundred |
| những video mà người dùng tương tự đã xem | the videos that similar users have watched |
| nó có nhịp vận hành khác hẳn phần còn lại | it has an operational rhythm quite different from the rest |
| được huấn luyện lại theo chu kỳ | is retrained on a cycle |
| được triển khai dần chứ không đổi một lượt | is rolled out gradually rather than switched over all at once |
| một vòng phản hồi ghi lại hành vi xem | a feedback loop that records watching behaviour |
| như thể nó chỉ là một truy vấn sắp xếp phức tạp hơn | as if it were merely a more complicated sorting query |

**Thuật ngữ cần nhớ**

- bảng tin gợi ý → **a recommendation feed**
- chọn ứng viên → **candidate selection**
- kho đặc trưng → **a feature store**
- huấn luyện lại → **to retrain**
- vòng phản hồi → **a feedback loop**

---

## Mô hình ghi nhớ

**Tiếng Việt**

Tệp gốc nằm ở kho đối tượng, các biến thể nằm ở mạng phân phối nội dung, việc nặng thì chạy bất đồng bộ, và việc đếm thì chấp nhận xấp xỉ. Bài toán video là bài toán băng thông cộng đường ống, chứ không phải bài toán tính toán đồng bộ.

**English (bám cấu trúc tiếng Việt)**

The master file sits in object storage, the variants sit on the content delivery network, the heavy work runs asynchronously, and the counting accepts being approximate. A video platform is a bandwidth-plus-pipeline problem, not a synchronous compute problem.

---

## Bảng thuật ngữ tổng hợp

| Tiếng Việt | Tiếng Anh | Ghi chú phát âm |
|---|---|---|
| băng thông | bandwidth | âm cuối **-dth** khó: "BAND-with", đừng bỏ âm cuối |
| chi phí dữ liệu đi ra | egress cost | /ˈiːɡres/ — "**II**-gres", trọng âm đầu |
| rìa mạng | the edge of the network | |
| máy khách | the client | |
| tệp gốc | the master file | |
| phiên bản chất lượng | a quality variant | *variant* — "**VE**-ri-ợnt", trọng âm đầu |
| kho đối tượng | object storage | |
| máy chủ gốc | the origin server | *origin* — "**O**-ri-jin", trọng âm đầu |
| tốc độ bit thích ứng | adaptive bitrate (ABR) | *adaptive* — ơ-**DAP**-tịv, trọng âm âm thứ hai |
| độ phân giải | resolution | re-zơ-**LU**-shợn, trọng âm âm thứ ba |
| đoạn video | a segment | danh từ nhấn đầu: "**SEG**-mợnt" |
| tệp kê khai | a manifest | "**MA**-nị-fest", trọng âm đầu |
| thang chất lượng | the bitrate ladder | |
| vòng quay chờ | a loading spinner | |
| đường ống xử lý | a processing pipeline | |
| chuyển mã | to transcode / transcoding | |
| đàn máy xử lý | a worker farm | |
| đường dẫn có chữ ký | a signed URL | *signed* — chữ **g câm**, đọc "sain-d" |
| bất biến khi lặp lại | idempotent | ai-**DEM**-pơ-tần, trọng âm âm thứ hai |
| hết thời gian chờ | to time out | |
| ảnh đại diện | a thumbnail | *thumb* — chữ **b câm**; âm **th** /θ/ đầu |
| khung hình | a frame | |
| sinh trước, ngoại tuyến | offline generation | |
| cache ở biên | the edge cache | *cache* /kæʃ/ — đọc đúng như "cash" |
| tệp tĩnh | a static file | |
| dòng nóng | a hot row | *row* /rəʊ/ — "râu", không đọc "rao" |
| bộ đếm xấp xỉ | an approximate counter | *approximate* — ơ-**PROK**-si-mợt (tính từ) |
| ghi gộp định kỳ | a periodic flush | |
| luồng dữ liệu | a data stream | |
| tổng hợp | to aggregate | động từ "**A**-grơ-gêit"; danh từ đuôi đọc "-gợt" |
| lan truyền | viral | "**VAI**-rợl", âm đầu là "vai" |
| bảng tin gợi ý | a recommendation feed | *recommendation* — re-cơ-mần-**ĐÂY**-shợn |
| chọn ứng viên | candidate selection | *candidate* — "**CAN**-đi-đợt", trọng âm đầu |
| kho đặc trưng | a feature store | |
| huấn luyện lại | to retrain | |
| vòng phản hồi | a feedback loop | |
| triển khai dần | a gradual rollout | *gradual* — "**GRA**-ju-ợl", trọng âm đầu |
| thông báo | a notification | |
| hàng đợi | a queue | /kjuː/ — đọc như chữ "Q" |
| thông lượng | throughput | âm **th** /θ/ đầu, "THRU-put" |

---

## Bài nói

> **Cách luyện:** bật ghi âm, nói **60–90 giây** cho mỗi đề, **không nhìn tài liệu** khi nói. Mỗi bài phải dùng ít nhất **sáu thuật ngữ** trong bảng trên. Với đề số 1, hãy đọc to cả các con số ước lượng bằng tiếng Anh.

1. **Explain to a junior engineer** why a video platform is a bandwidth problem rather than a compute problem, using rough numbers to make the point.

2. **A colleague says:** *"We should return the upload response only after transcoding finishes, so we can promise the video is playable."* **Explain what is wrong with that**, and describe the flow you would build instead.

3. **A colleague says:** *"We'll run `UPDATE videos SET views = views + 1` on every play — it keeps the count exact."* **Explain what is wrong with that**, and say what accuracy you are willing to give up.

4. **Explain to a junior engineer** what adaptive bitrate is and what the viewer experiences differently when their network gets weaker.

5. **Someone on your team wants to** generate thumbnails on the fly at request time so that the upload pipeline stays simple. **Explain why you would push back**, using the home page as your example.

6. **Describe what happens when** a user uploads a two-hour video, following it from the upload button to the moment a viewer in another country presses play.

7. **When would you choose** to build a recommendation feed rather than a simple follow feed, and what does that decision add to the system that was not there before?
