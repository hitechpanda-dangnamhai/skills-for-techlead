# Bài 11 — API Gateway, Service Mesh & Service Discovery
### Bản song ngữ: tiếng Anh bám cấu trúc tiếng Việt

> **Cách dùng bài này:** đọc đoạn tiếng Việt trước → tự dịch trong đầu, không nhìn xuống → so sánh với đoạn tiếng Anh ngay bên dưới → **đọc to bản tiếng Anh hai lần**: lần một đọc chậm cho đúng từng âm, lần hai đọc liền mạch như đang nói với người phỏng vấn.

---

## ① Service discovery: các service tìm nhau bằng cách nào

**Tiếng Việt**

Sau khi chúng ta đã tách hệ thống thành nhiều service, câu hỏi hạ tầng đầu tiên là các service tìm thấy nhau bằng cách nào. Trong môi trường container, số lượng bản chạy thay đổi liên tục và địa chỉ mạng của chúng cũng đổi theo. Vì vậy chúng ta không bao giờ được ghi cứng địa chỉ của service phía sau vào trong mã nguồn hoặc vào một tệp cấu hình tĩnh. Cách làm đúng là mỗi bản chạy tự đăng ký vào một sổ đăng ký khi nó khởi động, và tự rút tên khi nó dừng. Bên gọi thì tra cứu sổ đăng ký đó để lấy danh sách các địa chỉ đang còn sống.

Có hai kiểu tra cứu và chúng ta nên phân biệt được cả hai. Kiểu thứ nhất là tra cứu ở phía client, nghĩa là chính bên gọi hỏi sổ đăng ký rồi tự chọn một bản chạy để gửi request. Cách này cho bên gọi toàn quyền cân bằng tải, nhưng nó nhét logic hạ tầng vào trong mọi service. Kiểu thứ hai là tra cứu ở phía máy chủ, nghĩa là một bộ cân bằng tải hoặc một cổng vào hỏi giúp, còn bên gọi chỉ cần biết đúng một địa chỉ ổn định. Trong môi trường Kubernetes, phần lớn công việc này đã được đối tượng Service và hệ thống tên miền nội bộ lo sẵn, nên chúng ta hiếm khi phải tự dựng sổ đăng ký.

**English (bám cấu trúc tiếng Việt)**

After we have split the system into many services, the first infrastructure question is how the services find each other. In a container environment, the number of running instances changes continuously and their network addresses change along with it. Therefore we must never hard-code the address of a downstream service into the source code or into a static configuration file. The right approach is that each instance registers itself into a registry when it starts up, and removes its name when it stops. The caller then looks up that registry to get the list of addresses that are still alive.

There are two lookup styles and we should be able to tell both apart. The first style is client-side discovery, which means that the caller itself asks the registry and then picks an instance to send the request to. This way gives the caller full control over load balancing, but it pushes infrastructure logic into every service. The second style is server-side discovery, which means that a load balancer or a gateway asks on our behalf, while the caller only needs to know exactly one stable address. In a Kubernetes environment, most of this work is already handled by the Service object and the internal domain name system, so we rarely have to build a registry ourselves.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| câu hỏi hạ tầng đầu tiên | the first infrastructure question |
| số lượng bản chạy thay đổi liên tục | the number of running instances changes continuously |
| ghi cứng địa chỉ | hard-code the address |
| vào một tệp cấu hình tĩnh | into a static configuration file |
| tự đăng ký vào một sổ đăng ký | registers itself into a registry |
| tự rút tên khi nó dừng | removes its name when it stops |
| danh sách các địa chỉ đang còn sống | the list of addresses that are still alive |
| chúng ta nên phân biệt được cả hai | we should be able to tell both apart |
| nó nhét logic hạ tầng vào trong mọi service | it pushes infrastructure logic into every service |
| hỏi giúp | asks on our behalf |
| đã được lo sẵn | is already handled |

**Thuật ngữ cần nhớ**

- tìm kiếm dịch vụ → **service discovery**
- sổ đăng ký dịch vụ → **a service registry**
- bản chạy của một service → **an instance**
- ghi cứng vào mã nguồn → **to hard-code**
- hệ thống tên miền → **the domain name system (DNS)**

---

## ② API Gateway: cửa trước của cả hệ thống

**Tiếng Việt**

API Gateway là điểm vào duy nhất mà các client bên ngoài đi qua để chạm tới hệ thống của chúng ta. Nó nhận request từ trình duyệt hoặc từ ứng dụng di động, rồi nó định tuyến request đó tới đúng service bên trong. Nó cũng gộp nhiều lời gọi nội bộ thành một phản hồi duy nhất, để client không phải gọi năm lần cho một màn hình. Ngoài ra, nó là nơi tập trung các việc chung như xác thực, giới hạn tốc độ gọi, và kết thúc lớp mã hoá TLS. Nhờ có nó, cấu trúc nội bộ của chúng ta được che đi, nên chúng ta đổi hoặc gộp service bên trong mà client không hề biết.

Chúng ta nên nói thêm một cảnh báo, vì đây là chỗ rất dễ đi sai. Cổng vào phải giữ vai trò định tuyến và các mối quan tâm xuyên suốt, chứ nó không được trở thành nơi chứa logic nghiệp vụ. Nếu chúng ta nhồi dần các quy tắc nghiệp vụ vào đó, cổng vào sẽ phình lên thành một thành phần mà mọi đội đều phải sửa. Khi đó chúng ta lại có một điểm nghẽn tổ chức, và mọi lần phát hành đều phải xếp hàng chờ nhau. Vì vậy nguyên tắc là cổng vào biết đường đi, nhưng nó không biết ý nghĩa nghiệp vụ của dữ liệu đi qua nó.

**English (bám cấu trúc tiếng Việt)**

The API gateway is the single entry point that external clients pass through to reach our system. It receives requests from the browser or from the mobile application, and then it routes those requests to the right service inside. It also aggregates several internal calls into one single response, so that the client does not have to make five calls for one screen. Besides, it is the place where we centralise the common jobs such as authentication, rate limiting, and terminating the TLS encryption layer. Thanks to it, our internal structure is hidden, so we change or merge services inside without the client knowing at all.

We should add one warning, because this is a place where things very easily go wrong. The gateway must keep the role of routing and cross-cutting concerns, and it must not become the place that holds business logic. If we gradually stuff business rules into it, the gateway will swell up into a component that every team has to modify. At that point we have an organisational bottleneck again, and every release has to queue and wait for the others. Therefore the principle is that the gateway knows the route, but it does not know the business meaning of the data passing through it.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| điểm vào duy nhất | the single entry point |
| đi qua để chạm tới hệ thống của chúng ta | pass through to reach our system |
| gộp nhiều lời gọi nội bộ thành một phản hồi duy nhất | aggregates several internal calls into one single response |
| giới hạn tốc độ gọi | rate limiting |
| kết thúc lớp mã hoá TLS | terminating the TLS encryption layer |
| cấu trúc nội bộ được che đi | our internal structure is hidden |
| mà client không hề biết | without the client knowing at all |
| các mối quan tâm xuyên suốt | cross-cutting concerns |
| nếu chúng ta nhồi dần các quy tắc nghiệp vụ vào đó | if we gradually stuff business rules into it |
| một điểm nghẽn tổ chức | an organisational bottleneck |
| mọi lần phát hành đều phải xếp hàng chờ nhau | every release has to queue and wait for the others |
| cổng vào biết đường đi | the gateway knows the route |

**Thuật ngữ cần nhớ**

- cổng vào hệ thống → **an API gateway**
- điểm vào duy nhất → **a single entry point**
- gộp nhiều lời gọi → **request aggregation**
- giới hạn tốc độ gọi → **rate limiting**
- mối quan tâm xuyên suốt → **a cross-cutting concern**

---

## ③ Bắc–nam và đông–tây: hai loại lưu lượng khác nhau

**Tiếng Việt**

Trong sơ đồ hệ thống, chúng ta chia lưu lượng thành hai loại và mỗi loại có công cụ riêng của nó. Loại thứ nhất gọi là bắc–nam, tức là lưu lượng đi giữa client bên ngoài và hệ thống của chúng ta. Loại thứ hai gọi là đông–tây, tức là lưu lượng đi giữa các service bên trong với nhau. Cổng vào lo phần bắc–nam, vì nó nằm đúng ở biên ngoài nơi mọi client phải đi qua. Lưới dịch vụ lo phần đông–tây, vì nó nằm trên mọi đường đi nội bộ giữa các service.

Việc phân vai này quan trọng hơn nhiều người nghĩ, và người phỏng vấn hay kiểm tra đúng chỗ này. Nếu chúng ta bắt lưu lượng nội bộ vòng ra cổng vào rồi quay lại, chúng ta cộng thêm độ trễ và tạo ra một điểm chết duy nhất. Nếu chúng ta để mỗi service tự lo phần xác thực người dùng cuối, chúng ta lặp lại cùng một đoạn mã ở hai mươi nơi. Vì vậy quy tắc gọn nhất là biên ngoài dùng cổng vào, còn đường nội bộ dùng lưới dịch vụ hoặc gọi trực tiếp. Khi trả lời phỏng vấn, chúng ta nên vẽ ra hai loại lưu lượng đó trước, rồi mới nói tới công cụ.

**English (bám cấu trúc tiếng Việt)**

In a system diagram, we divide the traffic into two kinds and each kind has its own tool. The first kind is called north-south, that is, the traffic going between external clients and our system. The second kind is called east-west, that is, the traffic going between the internal services themselves. The gateway takes care of the north-south part, because it sits exactly at the outer edge where every client has to pass through. The service mesh takes care of the east-west part, because it sits on every internal path between the services.

This division of roles matters more than many people think, and interviewers often test exactly this spot. If we force the internal traffic to go out to the gateway and come back, we add extra latency and we create a single point of failure. If we let each service handle end-user authentication itself, we repeat the same piece of code in twenty places. Therefore the shortest rule is that the outer edge uses the gateway, while the internal paths use the service mesh or a direct call. When we answer in an interview, we should draw those two kinds of traffic first, and only then talk about the tools.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| mỗi loại có công cụ riêng của nó | each kind has its own tool |
| lưu lượng đi giữa | the traffic going between |
| các service bên trong với nhau | the internal services themselves |
| nó nằm đúng ở biên ngoài | it sits exactly at the outer edge |
| nó nằm trên mọi đường đi nội bộ | it sits on every internal path |
| việc phân vai này | this division of roles |
| bắt lưu lượng nội bộ vòng ra rồi quay lại | force the internal traffic to go out and come back |
| xác thực người dùng cuối | end-user authentication |
| chúng ta lặp lại cùng một đoạn mã ở hai mươi nơi | we repeat the same piece of code in twenty places |
| quy tắc gọn nhất là | the shortest rule is |
| rồi mới nói tới công cụ | and only then talk about the tools |

**Thuật ngữ cần nhớ**

- lưu lượng vào ra hệ thống → **north-south traffic**
- lưu lượng giữa các service → **east-west traffic**
- biên ngoài → **the outer edge**
- đường đi nội bộ → **an internal path**
- phân vai → **a division of roles**

---

## ④ Lưới dịch vụ và mô hình xe phụ

**Tiếng Việt**

Lưới dịch vụ là một lớp hạ tầng chuyên lo phần giao tiếp giữa service với service. Trong mô hình truyền thống, chúng ta chạy một proxy nhỏ ngay cạnh mỗi bản chạy của service, và người ta gọi nó là xe phụ. Mọi lưu lượng vào và ra service đều đi qua cái proxy đó, nên proxy có thể áp đặt rất nhiều chính sách. Cụ thể, nó lo mã hoá hai chiều, thử lại, thời gian chờ, ngắt mạch, chia lưu lượng theo tỷ lệ, và thu thập số liệu đo. Điểm hấp dẫn nhất là chúng ta có tất cả những thứ đó mà không phải sửa một dòng nào trong mã nguồn của service.

Bên cạnh các proxy còn có một mặt phẳng điều khiển, tức là phần não bộ cấu hình cho toàn bộ các proxy. Chúng ta khai báo chính sách một lần ở đó, rồi mặt phẳng điều khiển đẩy cấu hình xuống mọi xe phụ trong cụm. Nhờ vậy một tổ chức lớn áp được cùng một chuẩn cho hàng trăm service do hàng chục đội viết. Đổi lại, chúng ta phải vận hành thêm một hệ thống nữa và chấp nhận độ trễ mà proxy thêm vào mỗi chặng. Đây chính là lý do chúng ta phải cân nhắc kỹ trước khi đưa lưới dịch vụ vào một hệ thống nhỏ.

**English (bám cấu trúc tiếng Việt)**

A service mesh is an infrastructure layer that specialises in the communication between service and service. In the traditional model, we run a small proxy right next to each running instance of a service, and people call it a sidecar. All the traffic going into and out of the service passes through that proxy, so the proxy can enforce a great many policies. Specifically, it takes care of mutual encryption, retries, timeouts, circuit breaking, splitting traffic by percentage, and collecting telemetry. The most attractive point is that we get all of those things without modifying a single line in the source code of the service.

Alongside the proxies there is also a control plane, that is, the brain that configures all of the proxies. We declare a policy once there, and then the control plane pushes the configuration down to every sidecar in the cluster. Thanks to that, a large organisation can apply the same standard to hundreds of services written by dozens of teams. In exchange, we have to operate one more system and accept the latency that the proxy adds at each hop. This is exactly why we have to think carefully before bringing a service mesh into a small system.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| một lớp hạ tầng chuyên lo | an infrastructure layer that specialises in |
| ngay cạnh mỗi bản chạy | right next to each running instance |
| người ta gọi nó là xe phụ | people call it a sidecar |
| proxy có thể áp đặt rất nhiều chính sách | the proxy can enforce a great many policies |
| chia lưu lượng theo tỷ lệ | splitting traffic by percentage |
| thu thập số liệu đo | collecting telemetry |
| mà không phải sửa một dòng nào | without modifying a single line |
| phần não bộ cấu hình cho toàn bộ các proxy | the brain that configures all of the proxies |
| đẩy cấu hình xuống mọi xe phụ | pushes the configuration down to every sidecar |
| do hàng chục đội viết | written by dozens of teams |
| độ trễ mà proxy thêm vào mỗi chặng | the latency that the proxy adds at each hop |

**Thuật ngữ cần nhớ**

- lưới dịch vụ → **a service mesh**
- proxy chạy kèm mỗi pod → **a sidecar**
- mặt phẳng điều khiển → **the control plane**
- áp đặt chính sách → **to enforce a policy**
- số liệu đo từ hệ thống → **telemetry**

---

## ⑤ Kỷ nguyên xe phụ đang nhường chỗ

**Tiếng Việt**

Đây là phần cập nhật quan trọng nhất của bài, và nó phân biệt người theo dõi ngành với người đọc tài liệu cũ. Mô hình xe phụ hoạt động tốt nhưng nó tốn kém, vì mỗi pod phải nuôi thêm một proxy với bộ nhớ và độ trễ riêng. Khi một cụm có hàng nghìn pod, chi phí cộng dồn đó trở nên rất đáng kể. Vì vậy Istio đã đưa ra chế độ không xe phụ, trong đó một tiến trình chạy trên mỗi node lo phần mã hoá ở tầng bốn, còn một thành phần riêng lo phần tầng bảy khi chúng ta thật sự cần. Chế độ này đã chính thức ổn định từ năm hai nghìn không trăm hai mươi lăm, nên chúng ta nên biết tên và biết ý tưởng của nó.

Bên cạnh đó còn có hai hướng khác đáng nhắc tới trong một cuộc phỏng vấn. Hướng thứ nhất là đưa phần lưới xuống thẳng nhân hệ điều hành bằng eBPF, và Cilium là đại diện tiêu biểu cho hướng này. Hướng thứ hai là giữ mô hình xe phụ nhưng làm cho nó thật nhẹ, và Linkerd đi theo hướng đó với proxy viết bằng Rust. Nếu chúng ta đọc một bài so sánh viết trước giữa năm hai nghìn không trăm hai mươi lăm, bài đó sẽ bỏ qua toàn bộ chế độ không xe phụ. Vì các công cụ này đổi rất nhanh, chúng ta nên kiểm chứng lại tài liệu chính thức ngay trước khi ra quyết định.

**English (bám cấu trúc tiếng Việt)**

This is the most important update in this lesson, and it separates the person who follows the industry from the person who reads old documentation. The sidecar model works well but it is expensive, because each pod has to feed one more proxy with its own memory and its own latency. When a cluster has thousands of pods, that accumulated cost becomes very significant. Therefore Istio introduced a sidecar-free mode, in which one process running on each node handles the encryption at layer four, while a separate component handles the layer seven part when we genuinely need it. This mode became officially stable in the year two thousand and twenty-five, so we should know its name and know its idea.

Alongside that there are also two other directions worth mentioning in an interview. The first direction is to push the mesh down into the operating system kernel using eBPF, and Cilium is the typical representative of this direction. The second direction is to keep the sidecar model but make it really light, and Linkerd follows that direction with a proxy written in Rust. If we read a comparison article written before the middle of two thousand and twenty-five, that article will miss the sidecar-free mode entirely. Because these tools change very fast, we should verify the official documentation again right before we make a decision.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| phân biệt người theo dõi ngành với | separates the person who follows the industry from |
| mỗi pod phải nuôi thêm một proxy | each pod has to feed one more proxy |
| chi phí cộng dồn đó | that accumulated cost |
| chế độ không xe phụ | a sidecar-free mode |
| khi chúng ta thật sự cần | when we genuinely need it |
| đã chính thức ổn định từ | became officially stable in |
| hai hướng khác đáng nhắc tới | two other directions worth mentioning |
| đưa phần lưới xuống thẳng nhân hệ điều hành | push the mesh down into the operating system kernel |
| làm cho nó thật nhẹ | make it really light |
| bài đó sẽ bỏ qua toàn bộ | that article will miss ... entirely |
| ngay trước khi ra quyết định | right before we make a decision |

**Thuật ngữ cần nhớ**

- chế độ không xe phụ → **ambient mode**
- nhân hệ điều hành → **the kernel**
- tầng bốn và tầng bảy → **layer four and layer seven**
- chi phí cộng dồn → **accumulated cost**
- kiểm chứng lại tài liệu chính thức → **to verify the official documentation**

---

## ⑥ BFF: một cổng riêng cho mỗi loại client

**Tiếng Việt**

Một cổng vào dùng chung cho mọi loại client nghe có vẻ gọn, nhưng nó tạo ra vấn đề khi các client có nhu cầu khác nhau. Ứng dụng di động thường cần gói dữ liệu nhỏ và ít lời gọi, vì mạng di động chậm và pin có hạn. Trang web trên máy tính thì có thể nhận gói dữ liệu lớn hơn và hiển thị nhiều thứ hơn trong một màn hình. Nếu chúng ta phục vụ cả hai bằng đúng một tập điểm cuối, một bên sẽ luôn nhận thừa dữ liệu còn bên kia phải gọi thêm nhiều lần. Vì vậy chúng ta dựng một cổng riêng cho từng loại client, và mẫu này gọi là backend phục vụ frontend.

Lợi ích lớn nhất không nằm ở kỹ thuật mà nằm ở quyền sở hữu. Mỗi đội frontend sở hữu luôn cổng riêng của mình, nên họ đổi hình dạng phản hồi mà không phải chờ một đội nền tảng ở giữa. Nhờ đó chúng ta cũng tránh được cái cổng vào khổng lồ mà ai cũng phải sửa và không ai dám đụng vào. Cái giá phải trả là một phần logic ghép dữ liệu bị lặp lại ở nhiều cổng khác nhau. Chúng ta chấp nhận sự trùng lặp đó, vì nó rẻ hơn nhiều so với việc bốn đội cùng tranh nhau sửa một tệp cấu hình chung.

**English (bám cấu trúc tiếng Việt)**

One gateway shared by every kind of client sounds tidy, but it creates a problem when the clients have different needs. A mobile application usually needs a small data payload and few calls, because the mobile network is slow and the battery is limited. A web page on a desktop can receive a larger data payload and display more things in one screen. If we serve both with exactly one set of endpoints, one side will always receive excess data while the other side has to make several extra calls. Therefore we build a separate gateway for each kind of client, and this pattern is called backend for frontend.

The biggest benefit does not lie in the technology but lies in the ownership. Each frontend team owns its own gateway as well, so they change the shape of the response without waiting for a platform team in the middle. Thanks to that, we also avoid the giant gateway that everybody has to modify and nobody dares to touch. The price we pay is that part of the data-composition logic is duplicated across several gateways. We accept that duplication, because it is far cheaper than four teams competing to edit one shared configuration file.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| nghe có vẻ gọn | sounds tidy |
| gói dữ liệu nhỏ và ít lời gọi | a small data payload and few calls |
| pin có hạn | the battery is limited |
| đúng một tập điểm cuối | exactly one set of endpoints |
| một bên sẽ luôn nhận thừa dữ liệu | one side will always receive excess data |
| không nằm ở kỹ thuật mà nằm ở quyền sở hữu | does not lie in the technology but lies in the ownership |
| đổi hình dạng phản hồi | change the shape of the response |
| ai cũng phải sửa và không ai dám đụng vào | everybody has to modify and nobody dares to touch |
| logic ghép dữ liệu bị lặp lại | the data-composition logic is duplicated |
| chúng ta chấp nhận sự trùng lặp đó | we accept that duplication |
| tranh nhau sửa một tệp cấu hình chung | competing to edit one shared configuration file |

**Thuật ngữ cần nhớ**

- cổng riêng cho từng loại client → **backend for frontend (BFF)**
- gói dữ liệu trả về → **the payload**
- điểm cuối API → **an endpoint**
- quyền sở hữu → **ownership**
- sự trùng lặp → **duplication**

---

## ⑦ mTLS và nguyên tắc không tin mạng nội bộ

**Tiếng Việt**

Trong cách nghĩ cũ, chúng ta coi mạng nội bộ là vùng an toàn và chỉ dựng tường lửa ở biên ngoài. Cách nghĩ đó không còn đứng vững, vì chỉ cần một service bị chiếm là kẻ tấn công đã đứng bên trong vùng an toàn. Từ vị trí đó, họ có thể lần sang các service khác, và người ta gọi kiểu di chuyển này là dịch chuyển ngang. Vì vậy nguyên tắc hiện nay là không tin tưởng mặc định, kể cả với lưu lượng nằm hoàn toàn bên trong cụm. Cách thực thi nguyên tắc đó là bắt buộc xác thực hai chiều giữa mọi cặp service, tức là cả bên gọi lẫn bên bị gọi đều phải chứng minh danh tính.

Điểm hay là lưới dịch vụ làm phần này gần như miễn phí về mặt công sức lập trình. Lưới tự cấp chứng chỉ cho từng service, tự xoay vòng chứng chỉ theo định kỳ, và tự mã hoá lưu lượng giữa các proxy. Chúng ta chỉ cần bật một chính sách ở mức không gian tên, và mọi giao tiếp trong đó buộc phải dùng mã hoá hai chiều. Nếu không có lưới, chúng ta phải tự quản lý chứng chỉ trong từng ứng dụng, và việc đó rất dễ sai và rất dễ hết hạn. Vì lý do này, nhu cầu về xác thực hai chiều thường là lý do mạnh nhất để một tổ chức quyết định dựng lưới dịch vụ.

**English (bám cấu trúc tiếng Việt)**

In the old way of thinking, we treated the internal network as a safe zone and we only built a firewall at the outer edge. That way of thinking no longer holds up, because it only takes one service being taken over for the attacker to be standing inside the safe zone. From that position, they can work their way across to other services, and people call this kind of movement lateral movement. Therefore the principle nowadays is not to trust by default, even with traffic that sits entirely inside the cluster. The way to enforce that principle is to require mutual authentication between every pair of services, that is, both the caller and the callee have to prove their identity.

The nice point is that a service mesh does this part almost for free in terms of programming effort. The mesh issues a certificate for each service itself, rotates the certificates periodically itself, and encrypts the traffic between the proxies itself. We only need to turn on one policy at the namespace level, and every communication inside it is forced to use mutual encryption. Without a mesh, we have to manage certificates inside each application ourselves, and that is very easy to get wrong and very easy to let expire. For this reason, the need for mutual authentication is often the strongest reason for an organisation to decide to build a service mesh.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| coi mạng nội bộ là vùng an toàn | treated the internal network as a safe zone |
| cách nghĩ đó không còn đứng vững | that way of thinking no longer holds up |
| chỉ cần một service bị chiếm là | it only takes one service being taken over for |
| họ có thể lần sang các service khác | they can work their way across to other services |
| không tin tưởng mặc định | not to trust by default |
| bắt buộc xác thực hai chiều | to require mutual authentication |
| đều phải chứng minh danh tính | have to prove their identity |
| gần như miễn phí về mặt công sức lập trình | almost for free in terms of programming effort |
| tự xoay vòng chứng chỉ theo định kỳ | rotates the certificates periodically itself |
| ở mức không gian tên | at the namespace level |
| rất dễ sai và rất dễ hết hạn | very easy to get wrong and very easy to let expire |

**Thuật ngữ cần nhớ**

- xác thực hai chiều → **mutual TLS (mTLS)**
- không tin tưởng mặc định → **zero trust**
- dịch chuyển ngang trong mạng → **lateral movement**
- chứng chỉ số → **a certificate**
- xoay vòng chứng chỉ → **certificate rotation**

---

## ⑧ Thử lại chồng nhau và cái giá thật của lưới dịch vụ

**Tiếng Việt**

Khi chúng ta đẩy phần chống chịu lỗi xuống hạ tầng, có một hậu quả bất ngờ mà rất ít người nói ra. Nếu lưới dịch vụ được cấu hình thử lại hai lần và mã nguồn ứng dụng cũng thử lại hai lần, tổng số lời gọi tới service phía sau là bốn chứ không phải hai. Người ta gọi hiện tượng này là khuếch đại thử lại, và nó dìm chết đúng cái service đang ốm. Một biến thể khác cũng khó chịu không kém là cầu dao nằm ở lưới nhảy mà ứng dụng hoàn toàn không biết. Khi đó đội phát triển nhìn thấy lỗi lạ trong log mà không tìm ra nguyên nhân trong mã nguồn của mình. Vì vậy nguyên tắc là chúng ta chỉ đặt chính sách thử lại ở đúng một tầng, và chúng ta ghi rõ tầng đó ở đâu.

Khi một đồng nghiệp đề xuất dựng đầy đủ một lưới dịch vụ cho hệ thống ba service, chúng ta nên phản biện theo ba bước. Bước một, chúng ta hỏi lại vấn đề thật sự cần giải là gì, vì nếu chỉ cần thời gian chờ và cầu dao thì một thư viện trong mã nguồn đã đủ. Bước hai, chúng ta nêu ra chi phí thật, gồm bộ nhớ và độ trễ thêm vào mỗi pod, một mặt phẳng điều khiển phải vận hành, và đường học rất dốc cho cả đội. Bước ba, chúng ta đề nghị chỉ dùng lưới khi số service đủ lớn và khi nhu cầu về mã hoá hai chiều cùng khả năng quan sát là thống nhất toàn tổ chức. Câu chốt nên là lưới dịch vụ giải bài toán quy mô, chứ nó không giải bài toán ba service gọi nhau.

**English (bám cấu trúc tiếng Việt)**

When we push the resilience part down into the infrastructure, there is a surprising consequence that very few people say out loud. If the service mesh is configured to retry twice and the application source code also retries twice, the total number of calls to the downstream service is four rather than two. People call this phenomenon retry amplification, and it drowns exactly the service that is already sick. Another variant that is equally annoying is that the circuit breaker sitting in the mesh trips while the application knows nothing about it. At that point the development team sees strange errors in the log without finding the cause in their own source code. Therefore the principle is that we only put the retry policy at exactly one layer, and we write down clearly where that layer is.

When a colleague proposes building a full service mesh for a three-service system, we should push back in three steps. Step one, we ask back what the real problem to solve is, because if we only need timeouts and circuit breakers then a library in the source code is already enough. Step two, we lay out the real cost, which includes the memory and the latency added to each pod, a control plane that has to be operated, and a very steep learning curve for the whole team. Step three, we propose using a mesh only when the number of services is large enough and when the need for mutual encryption together with observability is consistent across the organisation. The closing sentence should be that a service mesh solves the problem of scale, but it does not solve the problem of three services calling each other.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| đẩy phần chống chịu lỗi xuống hạ tầng | push the resilience part down into the infrastructure |
| một hậu quả bất ngờ mà rất ít người nói ra | a surprising consequence that very few people say out loud |
| là bốn chứ không phải hai | is four rather than two |
| nó dìm chết đúng cái service đang ốm | it drowns exactly the service that is already sick |
| cũng khó chịu không kém | equally annoying |
| mà ứng dụng hoàn toàn không biết | while the application knows nothing about it |
| nhìn thấy lỗi lạ trong log | sees strange errors in the log |
| chúng ta ghi rõ tầng đó ở đâu | we write down clearly where that layer is |
| vấn đề thật sự cần giải là gì | what the real problem to solve is |
| đường học rất dốc cho cả đội | a very steep learning curve for the whole team |
| thống nhất toàn tổ chức | consistent across the organisation |
| giải bài toán quy mô | solves the problem of scale |

**Thuật ngữ cần nhớ**

- khuếch đại thử lại → **retry amplification**
- chồng chính sách lên nhau → **overlapping policies**
- đường cong học tập dốc → **a steep learning curve**
- chi phí phụ thêm → **overhead**
- bài toán quy mô → **the problem of scale**

---

## ⑨ Mô hình ghi nhớ

**Tiếng Việt**

Cổng vào là cửa trước lo lưu lượng bắc–nam, còn lưới dịch vụ là đường nội bộ lo lưu lượng đông–tây với mã hoá hai chiều, thử lại và số liệu đo mà không phải sửa mã nguồn. Tìm kiếm dịch vụ thay cho việc ghi cứng địa chỉ, lưới rất mạnh nhưng đắt và mô hình xe phụ đang nhường chỗ cho chế độ không xe phụ, còn với hệ nhỏ thì một thư viện cộng một cổng vào đã là đủ.

**English (bám cấu trúc tiếng Việt)**

The gateway is the front door handling north-south traffic, while the service mesh is the internal road handling east-west traffic with mutual encryption, retries and telemetry without modifying the source code. Service discovery replaces hard-coding addresses, the mesh is very powerful but expensive and the sidecar model is giving way to ambient mode, while for a small system one library plus one gateway is already enough.

---

## ⑩ Bảng thuật ngữ tổng hợp

| Tiếng Việt | Tiếng Anh | Ghi chú phát âm |
|---|---|---|
| tìm kiếm dịch vụ | service discovery | dis-**CO**-və-ri, trọng âm âm thứ hai |
| sổ đăng ký dịch vụ | a service registry | **RE**-gis-try, trọng âm âm đầu |
| bản chạy của một service | an instance | **IN**-stəns |
| ghi cứng vào mã nguồn | to hard-code | |
| hệ thống tên miền | DNS | đọc từng chữ cái: D-N-S |
| cổng vào hệ thống | an API gateway | *API* đọc từng chữ cái: A-P-I |
| điểm vào duy nhất | a single entry point | |
| gộp nhiều lời gọi | request aggregation | a-gri-**GAY**-shn |
| giới hạn tốc độ gọi | rate limiting | |
| kết thúc lớp mã hoá | TLS termination | ter-mi-**NAY**-shn |
| mối quan tâm xuyên suốt | a cross-cutting concern | |
| điểm nghẽn | a bottleneck | |
| lưu lượng vào ra hệ thống | north-south traffic | |
| lưu lượng giữa các service | east-west traffic | |
| biên ngoài | the outer edge | |
| lưới dịch vụ | a service mesh | *mesh* /meʃ/ — bật rõ /ʃ/ ở cuối |
| proxy chạy kèm mỗi pod | a sidecar | |
| mặt phẳng điều khiển | the control plane | |
| áp đặt chính sách | to enforce a policy | *enforce* = in-**FORS** |
| số liệu đo từ hệ thống | telemetry | tə-**LE**-mə-tri, trọng âm âm thứ hai |
| chia lưu lượng theo tỷ lệ | traffic splitting | |
| chế độ không xe phụ | ambient mode | **AM**-bi-ənt, trọng âm âm đầu |
| nhân hệ điều hành | the kernel | **KER**-nl, chữ *e* đọc /ɜː/ |
| chi phí cộng dồn | accumulated cost | ə-**KYU**-myu-lay-tid |
| cụm máy | a cluster | |
| cổng riêng cho từng loại client | backend for frontend (BFF) | đọc từng chữ cái: B-F-F |
| gói dữ liệu trả về | the payload | **PAY**-load |
| điểm cuối API | an endpoint | |
| quyền sở hữu | ownership | |
| sự trùng lặp | duplication | du-pli-**KAY**-shn |
| xác thực hai chiều | mutual TLS (mTLS) | *mutual* = **MYU**-chu-al |
| không tin tưởng mặc định | zero trust | *trust* — âm cuối /st/ phải bật rõ |
| dịch chuyển ngang trong mạng | lateral movement | **LA**-tə-rəl, trọng âm âm đầu |
| chứng chỉ số | a certificate | sə-**TI**-fi-kət, trọng âm âm thứ hai |
| xoay vòng chứng chỉ | certificate rotation | rəʊ-**TAY**-shn |
| không gian tên | a namespace | |
| khuếch đại thử lại | retry amplification | am-pli-fi-**KAY**-shn, năm âm tiết |
| chồng chính sách lên nhau | overlapping policies | |
| đường cong học tập dốc | a steep learning curve | *curve* = /kɜːv/, "KƠV" |
| chi phí phụ thêm | overhead | **O**-və-hed |
| bài toán quy mô | the problem of scale | |
| tường lửa | a firewall | |

---

## ⑪ Bài nói

> **Cách luyện:** bật ghi âm, nói **60–90 giây** cho mỗi đề, **không nhìn tài liệu**. Mỗi bài nói phải dùng ít nhất **sáu thuật ngữ** trong bảng trên. Nói xong thì nghe lại một lượt, đánh dấu chỗ mình ngập ngừng, rồi nói lại chính đề đó thêm một lần nữa.

1. Explain to a junior developer why we never hard-code a downstream address, and describe the difference between client-side and server-side discovery.

2. Describe the division of roles between an API gateway and a service mesh, using north-south and east-west traffic to frame it.

3. A colleague wants to add order validation rules into the gateway because it already sees every request. Explain why you would push back.

4. Your team proposes installing a full service mesh for a system with three services. Explain the real costs and what you would recommend instead.

5. After a mesh rollout, p99 latency has risen and the downstream service is being flooded even though traffic is flat. Describe your two main suspects and how you would confirm them.

6. Explain what mutual TLS gives you inside the cluster, and explain what lateral movement means to somebody who is not a security specialist.

7. When would you introduce a backend for frontend rather than serving mobile and web from the same gateway, and what duplication would you accept in return?
