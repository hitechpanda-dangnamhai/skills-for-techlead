# Bài 2 — Capacity & back-of-envelope estimation
### Bản song ngữ: tiếng Anh bám cấu trúc tiếng Việt

> **Cách dùng:** đọc đoạn tiếng Việt trước → tự dịch trong đầu → so với bản tiếng Anh bên dưới → **đọc to bản tiếng Anh hai lần**. Lần một đọc chậm cho đúng từ, lần hai đọc liền mạch như đang nói với người phỏng vấn. Riêng bài này, hãy đọc to cả các con số bằng tiếng Anh, vì đọc số là chỗ người Việt vấp nhiều nhất khi phỏng vấn.

---

## Phần 1 — Ước lượng để ra quyết định, không phải để ra con số đúng

**Tiếng Việt**

Ước lượng dung lượng không nhằm ra một con số đúng, mà nhằm trả lời một câu hỏi nhị phân. Câu hỏi đó thường là: chúng ta có cần chia mảnh cơ sở dữ liệu không, chúng ta có cần cache không, và một trung tâm dữ liệu là đủ hay chúng ta phải dùng nhiều trung tâm. Vì mục đích là như vậy, sai số hai lần thì không sao cả. Nhưng nếu chúng ta sai một bậc độ lớn, tức là sai mười lần, thì mọi quyết định kiến trúc phía sau đều sai theo. Vì vậy, mục tiêu của chúng ta là bậc độ lớn, chứ không phải độ chính xác. Nếu chúng ta quên điều này và cố tính ra một con số nhiều chữ số, chúng ta đang làm kế toán chứ không đang thiết kế hệ thống.

**English (bám cấu trúc tiếng Việt)**

Capacity estimation does not aim to produce a correct number, it aims to answer a binary question. That question is usually: do we need to shard the database, do we need a cache, and is one data centre enough or do we have to use several data centres. Because the purpose is like that, being off by a factor of two is fine. But if we are off by an order of magnitude, that is, off by a factor of ten, then every architectural decision after that is wrong as well. Therefore, our target is the order of magnitude, not the precision. If we forget this and try to produce a number with many digits, we are doing accounting rather than designing a system.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| không nhằm ra một con số đúng | does not aim to produce a correct number |
| một câu hỏi nhị phân | a binary question |
| chia mảnh cơ sở dữ liệu | shard the database |
| một trung tâm dữ liệu là đủ hay | is one data centre enough or |
| sai số hai lần thì không sao cả | being off by a factor of two is fine |
| sai một bậc độ lớn | off by an order of magnitude |
| đều sai theo | is wrong as well |
| một con số nhiều chữ số | a number with many digits |
| chúng ta đang làm kế toán | we are doing accounting |

**Thuật ngữ cần nhớ**

- ước lượng dung lượng → **capacity estimation**
- bậc độ lớn → **order of magnitude**
- hệ số nhân, số lần → **a factor of** (…)
- chia mảnh → **to shard**
- độ chính xác → **precision**

---

## Phần 2 — Phép tính thứ nhất: số truy vấn mỗi giây

**Tiếng Việt**

Phép tính đầu tiên là số truy vấn mỗi giây, và chúng ta lấy số người dùng hoạt động hằng ngày nhân với số request mỗi người mỗi ngày, rồi chia cho 86.400 giây. Con số 86.400 là số giây trong một ngày, và chúng ta nên thuộc nó để khỏi phải nhẩm lại giữa buổi phỏng vấn. Ví dụ, 10 triệu người dùng nhân với 10 request mỗi ngày ra 100 triệu request mỗi ngày, và chia cho 86.400 thì chúng ta được khoảng 1.150 request mỗi giây ở mức trung bình. Nhưng lưu lượng không rải đều trong ngày, vì người dùng dồn vào giờ cao điểm. Vì vậy chúng ta nhân thêm một hệ số từ hai đến ba lần để ra mức đỉnh, tức là khoảng 2.300 đến 3.500 request mỗi giây.

Trong lúc tính, chúng ta phải nói giả định thành tiếng thay vì bịa một con số từ hư không. Chúng ta nói rõ rằng chúng ta giả sử mỗi người dùng gửi 10 request mỗi ngày, rồi chúng ta mời người phỏng vấn sửa lại nếu con số thật khác. Người phỏng vấn chấm cách chúng ta suy luận, nên một con số sai nhưng có cơ sở vẫn được chấp nhận. Nếu chúng ta chỉ tính mức trung bình và bỏ qua mức đỉnh, hậu quả là chúng ta thiết kế một hệ thống sập đúng vào giờ đông người nhất. Hệ thống không bao giờ chết vì mức trung bình, hệ thống chết vì mức đỉnh.

**English (bám cấu trúc tiếng Việt)**

The first calculation is the number of queries per second, and we take the number of daily active users, multiply it by the number of requests per user per day, and then divide it by 86,400 seconds. The number 86,400 is the number of seconds in a day, and we should know it by heart so that we do not have to work it out again in the middle of the interview. For example, 10 million users multiplied by 10 requests per day gives 100 million requests per day, and dividing that by 86,400 gives us about 1,150 requests per second on average. But the traffic is not spread evenly across the day, because users pile up in the peak hours. Therefore we multiply by a further factor of two to three to get the peak, that is, about 2,300 to 3,500 requests per second.

While we calculate, we must state our assumptions out loud instead of making up a number out of thin air. We say clearly that we assume each user sends 10 requests per day, and then we invite the interviewer to correct it if the real number is different. The interviewer scores the way we reason, so a number that is wrong but well-founded is still accepted. If we only calculate the average and ignore the peak, the consequence is that we design a system which collapses exactly at the busiest hour. A system never dies because of the average, a system dies because of the peak.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| lấy A nhân với B rồi chia cho C | take A, multiply it by B, and then divide it by C |
| chúng ta nên thuộc nó | we should know it by heart |
| để khỏi phải nhẩm lại | so that we do not have to work it out again |
| lưu lượng không rải đều trong ngày | the traffic is not spread evenly across the day |
| người dùng dồn vào giờ cao điểm | users pile up in the peak hours |
| nhân thêm một hệ số từ hai đến ba lần | multiply by a further factor of two to three |
| bịa một con số từ hư không | making up a number out of thin air |
| mời người phỏng vấn sửa lại | invite the interviewer to correct it |
| sai nhưng có cơ sở | wrong but well-founded |
| sập đúng vào giờ đông người nhất | collapses exactly at the busiest hour |

**Thuật ngữ cần nhớ**

- số truy vấn mỗi giây → **queries per second (QPS)**
- mức trung bình / mức đỉnh → **average** / **peak**
- hệ số đỉnh → **the peak factor**
- lưu lượng → **traffic**
- giờ cao điểm → **peak hours**

---

## Phần 3 — Phép tính thứ hai: dung lượng lưu trữ

**Tiếng Việt**

Phép tính thứ hai là dung lượng lưu trữ, và chúng ta lấy số mục dữ liệu mỗi năm nhân với kích thước trung bình của một mục, rồi nhân với một hệ số phụ trội. Hệ số phụ trội gồm hai phần: số bản sao mà chúng ta giữ, thường là hai đến ba bản, và phần siêu dữ liệu đi kèm mỗi mục. Ví dụ, 500 triệu ảnh mỗi năm nhân với 1 megabyte mỗi ảnh nhân với 3 bản sao ra khoảng 1,5 petabyte mỗi năm. Đây là con số cho một năm, nên chúng ta phải nhân tiếp cho số năm mà hệ thống dự kiến chạy. Nếu chúng ta chỉ báo cáo con số của năm đầu, người phỏng vấn sẽ hỏi ngay là ba năm nữa thì sao.

Người Việt hay quên hệ số nhân bản khi tính lưu trữ, và đây là lỗi nguy hiểm nhất trong bài này. Chúng ta quên nhân ba thì con số ra nhỏ hơn ba lần, và ba lần thì gần chạm một bậc độ lớn. Khi con số nhỏ đi như vậy, chúng ta sẽ kết luận sai rằng một cụm máy chủ đơn giản là đủ. Hậu quả là thiết kế của chúng ta thiếu chỗ chứa ngay trong năm đầu tiên, và đội vận hành phải chữa cháy bằng cách mua thêm đĩa gấp. Vì vậy, chúng ta nên đọc to hệ số nhân bản mỗi khi tính lưu trữ, coi như một thói quen bắt buộc.

**English (bám cấu trúc tiếng Việt)**

The second calculation is the storage size, and we take the number of items per year, multiply it by the average size of one item, and then multiply it by an overhead factor. The overhead factor has two parts: the number of copies we keep, usually two to three copies, and the metadata that comes with each item. For example, 500 million photos per year multiplied by 1 megabyte per photo multiplied by 3 copies gives about 1.5 petabytes per year. This is the number for one year, so we have to multiply again by the number of years the system is expected to run. If we only report the number for the first year, the interviewer will immediately ask what happens three years from now.

Vietnamese engineers often forget the replication factor when they calculate storage, and this is the most dangerous mistake in this lesson. If we forget to multiply by three, the number comes out three times smaller, and three times is close to a whole order of magnitude. When the number shrinks like that, we will wrongly conclude that a simple server cluster is enough. The consequence is that our design runs out of space in the very first year, and the operations team has to fight the fire by buying more disks in a hurry. Therefore, we should read the replication factor out loud every time we calculate storage, and treat that as a compulsory habit.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| số mục dữ liệu mỗi năm | the number of items per year |
| một hệ số phụ trội | an overhead factor |
| phần siêu dữ liệu đi kèm mỗi mục | the metadata that comes with each item |
| số năm mà hệ thống dự kiến chạy | the number of years the system is expected to run |
| ba năm nữa thì sao | what happens three years from now |
| gần chạm một bậc độ lớn | close to a whole order of magnitude |
| chúng ta sẽ kết luận sai rằng | we will wrongly conclude that |
| thiếu chỗ chứa ngay trong năm đầu tiên | runs out of space in the very first year |
| chữa cháy bằng cách mua thêm đĩa gấp | fight the fire by buying more disks in a hurry |
| coi như một thói quen bắt buộc | treat that as a compulsory habit |

**Thuật ngữ cần nhớ**

- dung lượng lưu trữ → **storage size**
- hệ số nhân bản → **the replication factor**
- bản sao → **a replica**
- siêu dữ liệu → **metadata**
- phụ trội → **overhead**
- tăng trưởng → **growth**

---

## Phần 4 — Phép tính thứ ba: băng thông và tiền

**Tiếng Việt**

Phép tính thứ ba là băng thông, và chúng ta lấy số truy vấn mỗi giây ở mức đỉnh nhân với kích thước trung bình của một gói dữ liệu trả về. Kết quả là số byte mỗi giây, và chúng ta đổi nó ra megabyte mỗi giây cho dễ nói. Con số này trả lời ba câu hỏi cùng lúc: chúng ta có cần mạng phân phối nội dung không, chúng ta phải trả bao nhiêu tiền cho dữ liệu đi ra, và chúng ta nên chọn loại máy chủ nào. Chúng ta luôn phải nối con số với quyết định ngay trong cùng một câu, thay vì đọc con số rồi im lặng. Ví dụ, chúng ta nói rằng băng thông đỉnh khoảng 200 megabyte mỗi giây, vì vậy chúng ta đặt ảnh sau một mạng phân phối nội dung thay vì phục vụ trực tiếp từ máy chủ gốc.

Băng thông là chỗ mà thiết kế chạm thẳng vào hoá đơn hằng tháng. Dữ liệu đi ra khỏi nhà cung cấp đám mây bị tính tiền theo từng gigabyte, và với nền tảng nhiều ảnh hoặc nhiều video thì khoản này lớn hơn cả tiền máy chủ. Đẩy nội dung tĩnh qua mạng phân phối nội dung cắt được phần lớn khoản đó, vì bản sao được phục vụ từ điểm biên gần người dùng. Đây là lý do mọi nền tảng nặng về media đều đặt mạng phân phối nội dung lên trước. Nếu chúng ta bỏ qua phép tính băng thông, hậu quả là chúng ta trình bày một kiến trúc chạy được về mặt kỹ thuật nhưng không ai duyệt được về mặt chi phí.

**English (bám cấu trúc tiếng Việt)**

The third calculation is the bandwidth, and we take the number of queries per second at peak and multiply it by the average size of one response payload. The result is the number of bytes per second, and we convert it into megabytes per second so that it is easier to say. This number answers three questions at once: do we need a content delivery network, how much do we have to pay for the data going out, and which server type should we choose. We must always connect the number to a decision within the same sentence, instead of reading out the number and then going quiet. For example, we say that the peak bandwidth is about 200 megabytes per second, therefore we put the images behind a content delivery network instead of serving them directly from the origin server.

Bandwidth is the place where the design touches the monthly bill directly. Data going out of a cloud provider is charged per gigabyte, and for a platform with many images or many videos this item is larger than the server cost itself. Pushing static content through a content delivery network cuts most of that item, because the copy is served from an edge location close to the user. This is the reason why every media-heavy platform puts a content delivery network in front. If we skip the bandwidth calculation, the consequence is that we present an architecture which works technically but which nobody can approve on cost.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| kích thước trung bình của một gói dữ liệu trả về | the average size of one response payload |
| đổi nó ra megabyte mỗi giây cho dễ nói | convert it into megabytes per second so that it is easier to say |
| trả lời ba câu hỏi cùng lúc | answers three questions at once |
| dữ liệu đi ra | the data going out |
| nối con số với quyết định ngay trong cùng một câu | connect the number to a decision within the same sentence |
| đọc con số rồi im lặng | reading out the number and then going quiet |
| phục vụ trực tiếp từ máy chủ gốc | serving them directly from the origin server |
| chạm thẳng vào hoá đơn hằng tháng | touches the monthly bill directly |
| bị tính tiền theo từng gigabyte | is charged per gigabyte |
| được phục vụ từ điểm biên gần người dùng | is served from an edge location close to the user |
| không ai duyệt được về mặt chi phí | nobody can approve on cost |

**Thuật ngữ cần nhớ**

- băng thông → **bandwidth**
- gói dữ liệu → **the payload**
- mạng phân phối nội dung → **a content delivery network (CDN)**
- máy chủ gốc → **the origin server**
- điểm biên → **an edge location**
- chi phí dữ liệu đi ra → **egress cost**

---

## Phần 5 — Phép tính thứ tư: tỷ lệ đọc trên ghi

**Tiếng Việt**

Phép tính thứ tư không ra byte, mà ra một tỷ lệ, đó là tỷ lệ giữa lượng đọc và lượng ghi. Tỷ lệ này quan trọng vì nó cho chúng ta biết nên đặt nỗ lực tối ưu ở đâu. Nếu lượng đọc lớn hơn lượng ghi rất nhiều, chúng ta nghiêng về cache, bản sao chỉ đọc và mạng phân phối nội dung. Ngược lại, nếu lượng ghi lớn hơn lượng đọc, chúng ta nghiêng về chia mảnh, hàng đợi và kho lưu trữ tối ưu cho ghi như Cassandra. Hai hướng này dẫn tới hai kiến trúc rất khác nhau, nên chúng ta phải hỏi tỷ lệ trước khi vẽ.

Nhiều người bỏ qua tỷ lệ này vì nó không phải là một con số oai, và đó là một sai lầm tốn kém. Nếu chúng ta đoán nhầm hướng, chúng ta sẽ dồn công sức vào tầng cache trong khi điểm nghẽn thật nằm ở đường ghi. Hậu quả là hệ thống vẫn chậm sau khi chúng ta đã tiêu rất nhiều thời gian tối ưu, và không ai hiểu vì sao. Trong phỏng vấn, chúng ta chỉ cần một câu ngắn để lấy lại quyền chủ động: chúng ta hỏi tỷ lệ đọc trên ghi là bao nhiêu, và chúng ta nói luôn mỗi hướng sẽ dẫn tới thiết kế nào. Câu hỏi đó cho thấy chúng ta hiểu rằng kiến trúc là hệ quả của hình dạng tải, chứ không phải là danh sách công nghệ.

**English (bám cấu trúc tiếng Việt)**

The fourth calculation does not produce bytes, it produces a ratio, which is the ratio between the read volume and the write volume. This ratio matters because it tells us where to put our optimisation effort. If the read volume is much larger than the write volume, we lean towards a cache, read replicas and a content delivery network. On the other hand, if the write volume is larger than the read volume, we lean towards sharding, queues and a write-optimised store such as Cassandra. These two directions lead to two very different architectures, so we must ask for the ratio before we draw.

Many people skip this ratio because it is not an impressive number, and that is an expensive mistake. If we guess the wrong direction, we will pour our effort into the cache layer while the real bottleneck sits on the write path. The consequence is that the system is still slow after we have spent a lot of time optimising, and nobody understands why. In an interview, we only need one short sentence to take back the initiative: we ask what the read-to-write ratio is, and we say straight away which design each direction leads to. That question shows that we understand that the architecture is a consequence of the shape of the load, not a list of technologies.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| nên đặt nỗ lực tối ưu ở đâu | where to put our optimisation effort |
| chúng ta nghiêng về | we lean towards |
| kho lưu trữ tối ưu cho ghi | a write-optimised store |
| không phải là một con số oai | it is not an impressive number |
| một sai lầm tốn kém | an expensive mistake |
| nếu chúng ta đoán nhầm hướng | if we guess the wrong direction |
| dồn công sức vào tầng cache | pour our effort into the cache layer |
| lấy lại quyền chủ động | take back the initiative |
| là hệ quả của hình dạng tải | is a consequence of the shape of the load |

**Thuật ngữ cần nhớ**

- tỷ lệ đọc trên ghi → **the read-to-write ratio**
- nặng về đọc / nặng về ghi → **read-heavy** / **write-heavy**
- bản sao chỉ đọc → **a read replica**
- tầng cache → **the cache layer**
- hình dạng tải → **the shape of the load**

---

## Phần 6 — Thang bậc độ trễ mà chúng ta nên thuộc

**Tiếng Việt**

Có một bảng con số độ trễ kinh điển mà mọi kỹ sư nên thuộc ở mức bậc độ lớn. Bộ nhớ chính mất khoảng 100 nano giây cho một lần truy cập, còn đọc một megabyte tuần tự từ bộ nhớ chính mất khoảng 250 micro giây. Ổ đĩa thể rắn đọc ngẫu nhiên mất khoảng 150 micro giây, và đọc một megabyte từ ổ đĩa thể rắn mất khoảng một mili giây. Một vòng đi về giữa hai máy trong cùng một trung tâm dữ liệu mất khoảng nửa mili giây, trong khi một lần tìm kiếm trên ổ cứng cơ mất khoảng 10 mili giây. Một vòng đi về xuyên lục địa, ví dụ từ California sang châu Âu rồi quay lại, mất khoảng 100 đến 150 mili giây. Quy tắc để nhớ rất gọn: bộ nhớ tính bằng nano giây, ổ đĩa thể rắn tính bằng micro giây, đĩa và mạng nội bộ tính bằng mili giây, còn xuyên vùng thì tính bằng hàng trăm mili giây.

Chúng ta thuộc các con số này không phải để khoe, mà để biện luận cho một quyết định cụ thể. Vì bộ nhớ nhanh hơn đĩa khoảng một nghìn lần, chúng ta có lý do rõ ràng để đặt một tầng cache trước cơ sở dữ liệu. Vì một lần gọi xuyên vùng tốn hơn một trăm mili giây, chúng ta biết rằng gọi xuyên vùng trong luồng đồng bộ sẽ phá ngân sách độ trễ. Vì vậy chúng ta đặt bản sao gần người dùng, và chúng ta đẩy các việc xuyên vùng sang luồng bất đồng bộ. Nếu chúng ta không nắm thang bậc này, hậu quả là chúng ta tranh luận bằng cảm tính, và mọi lựa chọn của chúng ta đều nghe như ý kiến cá nhân chứ không phải lập luận kỹ thuật.

**English (bám cấu trúc tiếng Việt)**

There is a classic table of latency numbers that every engineer should know at the order-of-magnitude level. Main memory takes about 100 nanoseconds for one access, while reading one megabyte sequentially from main memory takes about 250 microseconds. A solid-state drive takes about 150 microseconds for a random read, and reading one megabyte from a solid-state drive takes about one millisecond. A round trip between two machines inside the same data centre takes about half a millisecond, while one seek on a spinning hard disk takes about 10 milliseconds. A round trip across continents, for example from California to Europe and back, takes about 100 to 150 milliseconds. The rule to remember is very compact: memory is measured in nanoseconds, solid-state drives in microseconds, disks and the local network in milliseconds, and cross-region in hundreds of milliseconds.

We learn these numbers by heart not in order to show off, but in order to argue for a specific decision. Because memory is about a thousand times faster than disk, we have a clear reason to put a cache layer in front of the database. Because one cross-region call costs more than a hundred milliseconds, we know that a cross-region call on the synchronous path will blow the latency budget. Therefore we place replicas close to the users, and we push the cross-region work onto an asynchronous path. If we do not have this hierarchy in our head, the consequence is that we argue by gut feeling, and every choice of ours sounds like a personal opinion rather than a technical argument.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| mà mọi kỹ sư nên thuộc | that every engineer should know |
| đọc một megabyte tuần tự | reading one megabyte sequentially |
| một vòng đi về | a round trip |
| một lần tìm kiếm trên ổ cứng cơ | one seek on a spinning hard disk |
| quy tắc để nhớ rất gọn | the rule to remember is very compact |
| không phải để khoe, mà để biện luận | not in order to show off, but in order to argue |
| nhanh hơn đĩa khoảng một nghìn lần | about a thousand times faster than disk |
| sẽ phá ngân sách độ trễ | will blow the latency budget |
| chúng ta tranh luận bằng cảm tính | we argue by gut feeling |
| nghe như ý kiến cá nhân | sounds like a personal opinion |

**Thuật ngữ cần nhớ**

- thang bậc độ trễ → **the latency hierarchy**
- bộ nhớ chính → **main memory**
- ổ đĩa thể rắn → **a solid-state drive (SSD)**
- vòng đi về → **a round trip**
- xuyên vùng → **cross-region**
- ngân sách độ trễ → **the latency budget**

---

## Phần 7 — Chỉ tính con số đổi được quyết định, và tránh bẫy nhảy thẳng sang số máy chủ

**Tiếng Việt**

Nguyên tắc cuối của bài này là chúng ta chỉ ước lượng những con số đổi được một quyết định. Trước mỗi phép tính, chúng ta tự hỏi một câu duy nhất: con số này có khiến chúng ta chọn khác đi không. Nếu câu trả lời là không, chúng ta bỏ phép tính đó và đi tiếp. Thời gian trong phỏng vấn rất hẹp, nên mỗi phút tiêu vào một con số trang trí là một phút bị lấy khỏi phần đào sâu. Ước lượng đúng cách nghe giống một chuỗi ngắn gồm giả định, con số, rồi quyết định, và chuỗi đó lặp lại vài lần là đủ.

Có một cái bẫy kinh điển mà nhiều ứng viên rơi vào ở bài này. Chúng ta tính ra khoảng 1.150 truy vấn mỗi giây rồi kết luận ngay rằng hệ thống cần 50 máy chủ. Kết luận đó sai ở hai chỗ. Thứ nhất, chúng ta chưa hề ước lượng một máy chủ gánh được bao nhiêu truy vấn mỗi giây, mà con số đó phụ thuộc vào kích thước gói dữ liệu, vào CPU và vào lượng vào ra đĩa. Thứ hai, chúng ta đã dùng mức trung bình thay vì mức đỉnh, trong khi hệ thống phải sống sót ở mức đỉnh. Cách nói đúng là chúng ta ước lượng sức chứa của một máy trước, rồi chia mức đỉnh cho sức chứa đó, rồi cộng thêm phần dự phòng cho lúc một máy chết.

**English (bám cấu trúc tiếng Việt)**

The final rule of this lesson is that we only estimate the numbers that can change a decision. Before each calculation, we ask ourselves one single question: does this number make us choose differently. If the answer is no, we drop that calculation and move on. Time in an interview is very tight, so every minute spent on a decorative number is a minute taken away from the deep-dive. Estimation done properly sounds like a short chain of an assumption, a number, and then a decision, and repeating that chain a few times is enough.

There is a classic trap that many candidates fall into in this lesson. We calculate about 1,150 queries per second and then immediately conclude that the system needs 50 servers. That conclusion is wrong in two places. First, we have not estimated at all how many queries per second one server can carry, and that number depends on the payload size, on the CPU and on the amount of disk input and output. Second, we have used the average instead of the peak, whereas the system has to survive at the peak. The correct way to put it is that we estimate the capacity of one machine first, then we divide the peak by that capacity, and then we add extra headroom for the moment when one machine dies.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| những con số đổi được một quyết định | the numbers that can change a decision |
| có khiến chúng ta chọn khác đi không | does it make us choose differently |
| chúng ta bỏ phép tính đó và đi tiếp | we drop that calculation and move on |
| thời gian trong phỏng vấn rất hẹp | time in an interview is very tight |
| bị lấy khỏi phần đào sâu | taken away from the deep-dive |
| một cái bẫy kinh điển | a classic trap |
| kết luận đó sai ở hai chỗ | that conclusion is wrong in two places |
| chúng ta chưa hề ước lượng | we have not estimated at all |
| lượng vào ra đĩa | the amount of disk input and output |
| trong khi hệ thống phải sống sót ở mức đỉnh | whereas the system has to survive at the peak |
| cộng thêm phần dự phòng | add extra headroom |

**Thuật ngữ cần nhớ**

- sức chứa của một máy → **per-server capacity**
- phần dự phòng → **headroom**
- con số trang trí → **a decorative number**
- lập kế hoạch dung lượng → **capacity planning**
- ngưỡng tự mở rộng → **the autoscaling threshold**

---

## Mô hình ghi nhớ

**Tiếng Việt**

Từ người dùng ra truy vấn, từ truy vấn ra byte, và từ byte ra quyết định. Một con số chỉ đáng tính nếu nó đổi được thiết kế, và bộ nhớ rẻ hơn đĩa khoảng một nghìn lần về độ trễ.

**English (bám cấu trúc tiếng Việt)**

From users to queries, from queries to bytes, and from bytes to decisions. A number is only worth calculating if it can change the design, and memory is about a thousand times cheaper than disk in terms of latency.

---

## Bảng thuật ngữ tổng hợp

| Tiếng Việt | Tiếng Anh | Ghi chú phát âm |
|---|---|---|
| ước lượng dung lượng | capacity estimation | *capacity* /kəˈpæsəti/ — cơ-**PA**-sơ-ti, trọng âm âm thứ hai |
| ước lượng nhanh trên giấy | back-of-envelope estimation | *estimate* động từ /ˈestɪmeɪt/ đọc "-mêit", danh từ /ˈestɪmət/ đọc "-mợt" |
| bậc độ lớn | order of magnitude | *magnitude* /ˈmæɡnɪtjuːd/ — "MÁG-ni-tiu-d", trọng âm đầu |
| độ chính xác | precision | pri-**SI**-zhợn, âm giữa là /ʒ/ như trong *vision* |
| số truy vấn mỗi giây | queries per second (QPS) | *query* /ˈkwɪəri/ — "KUY-ơ-ri", không đọc "quơ-ri" |
| người dùng hoạt động hằng ngày | daily active users (DAU) | |
| mức trung bình | average | |
| mức đỉnh | peak | /piːk/ — âm "i" dài, phân biệt với *pick* /pɪk/ |
| hệ số đỉnh | the peak factor | |
| lưu lượng | traffic | |
| giả định | an assumption | ơ-**SĂMP**-shợn, trọng âm âm thứ hai |
| dung lượng lưu trữ | storage size | |
| hệ số nhân bản | the replication factor | |
| bản sao | a replica | Anh /ˈreplɪkə/ — "**RE**-pli-cơ", trọng âm **đầu**, không đọc "rep-LAI-ka" |
| siêu dữ liệu | metadata | Anh thường đọc "**ME**-tơ-đây-tơ", trọng âm đầu |
| phụ trội | overhead | |
| chia mảnh | to shard / sharding | /ʃɑːd/ — "sáad", âm đầu là "sh" không phải "s" |
| băng thông | bandwidth | âm cuối **-dth** khó: đọc "BAND-with", đừng bỏ mất âm cuối |
| thông lượng | throughput | âm **th** /θ/ đầu, "THRU-put", không đọc thành "trú" |
| gói dữ liệu | the payload | |
| mạng phân phối nội dung | a content delivery network (CDN) | |
| máy chủ gốc | the origin server | *origin* /ˈɒrɪdʒɪn/ — "**O**-ri-jin", trọng âm đầu |
| điểm biên | an edge location | |
| chi phí dữ liệu đi ra | egress cost | /ˈiːɡres/ — "**II**-gres", trọng âm đầu, "e" đầu đọc dài |
| tỷ lệ đọc trên ghi | the read-to-write ratio | *ratio* /ˈreɪʃiəʊ/ — "RÂY-shi-âu", không đọc "ra-ti-ô" |
| nặng về đọc / nặng về ghi | read-heavy / write-heavy | |
| bản sao chỉ đọc | a read replica | |
| tầng cache | the cache layer | *cache* /kæʃ/ — đọc đúng như "cash", không đọc "ca-chê" |
| hàng đợi | a queue | /kjuː/ — đọc như chữ "Q", bốn chữ cái cuối câm |
| thang bậc độ trễ | the latency hierarchy | *latency* /ˈleɪtənsi/ "LÂY-tân-si"; *hierarchy* "**HAI**-ơ-ra-ki", trọng âm đầu |
| bộ nhớ chính | main memory | |
| tuần tự | sequential | si-**KWEN**-shợl, trọng âm âm thứ hai |
| ngẫu nhiên | random | |
| ổ đĩa thể rắn | a solid-state drive (SSD) | |
| ổ cứng cơ | a spinning hard disk | *disk* âm cuối **-sk** phải bật ra, không thành "đít" |
| vòng đi về | a round trip | |
| nano giây / micro giây / mili giây | nanosecond / microsecond / millisecond | |
| xuyên vùng | cross-region | |
| ngân sách độ trễ | the latency budget | |
| sức chứa của một máy | per-server capacity | |
| phần dự phòng | headroom | |
| máy chủ | host | âm cuối **-st** phải bật ra, không thành "hâu" |
| rủi ro | risk | âm cuối **-sk** phải bật ra, không thành "rít" |
| lập kế hoạch dung lượng | capacity planning | |
| ngưỡng tự mở rộng | the autoscaling threshold | *threshold* có âm **th** /θ/ đầu, "THRESH-hâuld" |

---

## Bài nói

> **Cách luyện:** bật ghi âm, nói **60–90 giây** cho mỗi đề, **không nhìn tài liệu** khi nói. Mỗi bài phải dùng ít nhất **sáu thuật ngữ** trong bảng trên, và phải đọc to mọi con số bằng tiếng Anh. Nghe lại bản ghi và đánh dấu chỗ nào bạn phải dừng lại tìm từ — chính chỗ đó là cụm cần học lại từ bảng ánh xạ.

1. **Explain to a junior engineer** how you get from 10 million daily active users to a peak QPS number, and say every assumption you make out loud as you go.

2. **A colleague says:** *"We calculated 1,150 queries per second, so we need about 50 servers."* **Explain what is wrong with that**, and describe the two steps they skipped before they could name a server count.

3. **Someone on your team wants to** estimate the byte size of every field in the schema before the design review. **Explain why you would push back**, and say which four numbers you would keep instead.

4. **Describe what happens when** a team calculates storage for 500 million photos a year but forgets the replication factor. Say how big the error is and where it shows up in production.

5. **Explain to a product manager** why serving images through a CDN instead of the origin server changes the monthly bill, using the bandwidth number to support your point.

6. **When would you choose** to spend your optimisation effort on the cache layer, and when would you choose to spend it on sharding and queues instead? Use the read-to-write ratio in your answer.

7. **Explain to a junior engineer**, using the latency hierarchy, why a cross-region call on the synchronous path is dangerous, and what you would do instead.
