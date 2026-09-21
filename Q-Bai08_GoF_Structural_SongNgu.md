# Bài 8 — GoF Structural: Adapter · Facade · Proxy · Decorator · Composite · Bridge
### Bản song ngữ: tiếng Anh bám cấu trúc tiếng Việt

> **Cách dùng bài này:** đọc đoạn tiếng Việt trước, tự dịch trong đầu, rồi mới nhìn sang đoạn tiếng Anh để so. Sau khi so xong, **đọc to đoạn tiếng Anh hai lần** — lần một để nhìn chữ, lần hai để nghe chính mình. Phần bài nói ở cuối là bắt buộc; không nói ra thì bài này chỉ là bài đọc.

---

## Phần 1 — Nhóm Structural và hai cái bẫy nằm trong bài này

**Tiếng Việt**

Nhóm Structural lo việc ghép các object thành một cấu trúc lớn hơn mà cấu trúc đó vẫn giữ được sự linh hoạt. Bài này chứa một bẫy phỏng vấn kinh điển, đó là phân biệt Adapter với Facade và với Proxy, bởi vì cả ba đều bọc một thứ khác nhưng ý đồ của chúng khác nhau. Bài này còn chứa một bẫy nữa rất phổ biến với người làm TypeScript, đó là Decorator của GoF không phải là decorator của TypeScript hay của Nest. Nội dung ở đây nối thẳng vào Bài 6, nơi chúng ta đã học về port và adapter theo tinh thần DIP. Nó cũng chuẩn bị nền cho phần kiến trúc ở Bài 11, bởi vì các kiến trúc hiện đại được dựng lên chính từ những mảnh ghép trong nhóm này.

**English (bám cấu trúc tiếng Việt)**

The Structural group takes care of combining objects into a larger structure while that structure still keeps its flexibility. This lesson contains a classic interview trap, which is telling the Adapter apart from the Facade and from the Proxy, because all three of them wrap something else but their intents are different. This lesson also contains one more trap that is very common for people working with TypeScript, which is that the GoF Decorator is not the decorator of TypeScript or of Nest. The content here connects straight into Lesson 6, where we learned about ports and adapters in the spirit of DIP. It also prepares the ground for the architecture part in Lesson 11, because modern architectures are built exactly from the pieces in this group.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| ghép các object thành một cấu trúc lớn hơn | combining objects into a larger structure |
| vẫn giữ được sự linh hoạt | still keeps its flexibility |
| phân biệt Adapter với Facade | telling the Adapter apart from the Facade |
| cả ba đều bọc một thứ khác | all three of them wrap something else |
| ý đồ của chúng khác nhau | their intents are different |
| một bẫy nữa rất phổ biến với người làm TypeScript | one more trap that is very common for people working with TypeScript |
| chuẩn bị nền cho phần kiến trúc | prepares the ground for the architecture part |
| được dựng lên chính từ những mảnh ghép trong nhóm này | are built exactly from the pieces in this group |

**Thuật ngữ cần nhớ**

- sự linh hoạt → **flexibility**
- bọc → **wrap**
- ý đồ → **intent**
- mảnh ghép → **piece** / **building block**
- kiến trúc → **architecture**

---

## Phần 2 — Adapter: đổi interface cho hai bên nói chuyện được

**Tiếng Việt**

Adapter chuyển interface của một class sang đúng interface mà client mong đợi. Ví dụ backend quen thuộc nhất là chúng ta bọc một SDK của bên thứ ba hoặc bọc một API cũ về đúng interface của domain mình. Nhờ cách này, phần nghiệp vụ chỉ nói chuyện bằng ngôn ngữ của chính nó, chứ phần nghiệp vụ không phải học ngôn ngữ của nhà cung cấp. Khi công ty đổi nhà cung cấp, chúng ta chỉ viết một adapter mới, và toàn bộ phần nghiệp vụ không đổi một dòng nào. Ý này chính là tinh thần của Anti-Corruption Layer trong DDD mà chúng ta sẽ học ở Bài 13 và Bài 14. Nếu chúng ta không có lớp adapter, thì tên trường và quy ước của SDK sẽ dần thấm vào code nghiệp vụ, và lúc đó việc đổi nhà cung cấp trở thành một dự án riêng chứ nó không còn là một thay đổi nhỏ.

**English (bám cấu trúc tiếng Việt)**

The Adapter converts the interface of one class into exactly the interface that the client expects. The most familiar backend example is that we wrap a third-party SDK or wrap a legacy API into exactly our own domain interface. Thanks to this way, the business part only speaks in its own language, and the business part does not have to learn the language of the vendor. When the company changes vendor, we only write a new adapter, and the whole business part does not change a single line. This idea is exactly the spirit of the Anti-Corruption Layer in DDD that we will study in Lesson 13 and Lesson 14. If we do not have an adapter layer, then the field names and conventions of the SDK will gradually seep into the business code, and at that point changing vendor becomes a project of its own and it is no longer a small change.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| sang đúng interface mà client mong đợi | into exactly the interface that the client expects |
| bọc một SDK của bên thứ ba | wrap a third-party SDK |
| bọc một API cũ | wrap a legacy API |
| chỉ nói chuyện bằng ngôn ngữ của chính nó | only speaks in its own language |
| không phải học ngôn ngữ của nhà cung cấp | does not have to learn the language of the vendor |
| chính là tinh thần của | is exactly the spirit of |
| tên trường và quy ước của SDK | the field names and conventions of the SDK |
| sẽ dần thấm vào code nghiệp vụ | will gradually seep into the business code |
| trở thành một dự án riêng | becomes a project of its own |

**Thuật ngữ cần nhớ**

- bên thứ ba → **third-party**
- hệ thống cũ để lại → **legacy**
- quy ước → **convention**
- thấm vào, ngấm vào → **seep into**
- lớp chống nhiễm bẩn → **Anti-Corruption Layer**

---

## Phần 3 — Adapter, Facade và Proxy khác nhau ở ý đồ

**Tiếng Việt**

Cả Adapter, Facade và Proxy đều bọc một thứ khác, cho nên chúng ta phải phân biệt chúng bằng ý đồ chứ chúng ta không phân biệt bằng hình dạng. Adapter đổi interface để hai bên tương thích với nhau, và ẩn dụ dễ nhớ là cái phích cắm chuyển đổi khi chúng ta đi nước ngoài. Facade đơn giản hoá một subsystem phức tạp thành một mặt gọn, nghĩa là nó gom nhiều thứ lại phía sau một cửa duy nhất, và ẩn dụ của nó là người lễ tân khách sạn. Proxy giữ nguyên interface của đối tượng thật, nhưng nó kiểm soát truy cập hoặc nó thêm hành vi, và ẩn dụ của nó là một người đại diện. Khi phỏng vấn, chúng ta nên trả lời theo đúng thứ tự tương thích, đơn giản hoá và kiểm soát, bởi vì ba từ đó phân biệt ba pattern gọn hơn mọi sơ đồ. Nếu chúng ta chỉ nói rằng cả ba đều bọc, người phỏng vấn sẽ kết luận rằng chúng ta chưa dùng chúng trong thực tế.

Riêng với Facade, chúng ta phải nói thêm về một rủi ro rất hay xảy ra. Facade dễ phình to thành một God object khi nó bắt đầu ôm luôn cả business logic vào bên trong. Ranh giới đúng là Facade chỉ điều phối, nghĩa là nó gọi các thành phần bên dưới theo đúng thứ tự, chứ nó không chứa rule nghiệp vụ lõi. Có một câu hỏi bẫy quen thuộc là "đã làm Facade cho gọn rồi thì nhét luôn logic vào cho tiện, đúng không", và câu trả lời của chúng ta là không. Khi logic nghiệp vụ chảy vào Facade, cohesion của cả module sụp xuống và chúng ta quay lại đúng vấn đề God Object ở Bài 10.

**English (bám cấu trúc tiếng Việt)**

The Adapter, the Facade and the Proxy all wrap something else, so we have to tell them apart by intent and we do not tell them apart by shape. The Adapter changes the interface so that the two sides become compatible, and the metaphor that is easy to remember is the plug adapter we use when we travel abroad. The Facade simplifies a complex subsystem into one tidy surface, which means that it gathers many things behind a single door, and its metaphor is the receptionist of a hotel. The Proxy keeps the interface of the real object unchanged, but it controls access or it adds behaviour, and its metaphor is a representative. In an interview, we should answer in exactly the order of compatibility, simplification and control, because those three words separate the three patterns more neatly than any diagram. If we only say that all three of them wrap, the interviewer will conclude that we have not used them in practice.

For the Facade in particular, we have to say more about a risk that happens very often. The Facade easily swells into a God object when it starts to hold the business logic inside as well. The correct boundary is that the Facade only coordinates, which means that it calls the components underneath in the right order, and it does not hold the core business rules. There is a familiar trap question, which is "since we already made a Facade to keep things tidy, we may as well put the logic in there too, right", and our answer is no. When business logic flows into the Facade, the cohesion of the whole module collapses and we come back to exactly the God Object problem in Lesson 10.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| phân biệt chúng bằng ý đồ | tell them apart by intent |
| chúng ta không phân biệt bằng hình dạng | we do not tell them apart by shape |
| cái phích cắm chuyển đổi khi chúng ta đi nước ngoài | the plug adapter we use when we travel abroad |
| thành một mặt gọn | into one tidy surface |
| gom nhiều thứ lại phía sau một cửa duy nhất | gathers many things behind a single door |
| người lễ tân khách sạn | the receptionist of a hotel |
| giữ nguyên interface của đối tượng thật | keeps the interface of the real object unchanged |
| gọn hơn mọi sơ đồ | more neatly than any diagram |
| dễ phình to thành một God object | easily swells into a God object |
| nhét luôn logic vào cho tiện, đúng không | we may as well put the logic in there too, right |
| cohesion của cả module sụp xuống | the cohesion of the whole module collapses |

**Thuật ngữ cần nhớ**

- tương thích → **compatible** / **compatibility**
- hệ thống con → **subsystem**
- lễ tân → **receptionist**
- người đại diện → **representative**
- phình to → **swell**
- ranh giới → **boundary**

---

## Phần 4 — Ba biến thể của Proxy

**Tiếng Việt**

Proxy có ba biến thể mà chúng ta nên thuộc tên. Biến thể thứ nhất là virtual proxy, và nó dùng để khởi tạo lười, nghĩa là chúng ta chỉ tạo object đắt tiền khi thật sự có người cần đến nó. Biến thể thứ hai là protection proxy, và nó kiểm tra quyền trước khi nó cho phép gọi vào đối tượng thật. Biến thể thứ ba là remote proxy, và nó đại diện cho một object nằm ở xa, ví dụ một stub gọi qua mạng. Bản thân JavaScript còn có sẵn một đối tượng tên là `Proxy` cho phép chúng ta chặn các thao tác truy cập thuộc tính. Điểm chung của cả ba biến thể là interface không đổi, cho nên client không biết mình đang nói chuyện với người đại diện hay đang nói chuyện với chính chủ.

**English (bám cấu trúc tiếng Việt)**

The Proxy has three variants whose names we should know by heart. The first variant is the virtual proxy, and it is used for lazy initialisation, which means that we only create an expensive object when somebody really needs it. The second variant is the protection proxy, and it checks permissions before it allows a call into the real object. The third variant is the remote proxy, and it stands for an object that sits far away, for example a stub that calls over the network. JavaScript itself also has a built-in object named `Proxy` that lets us intercept property access operations. What all three variants have in common is that the interface does not change, so the client does not know whether it is talking to the representative or talking to the real owner.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| ba biến thể mà chúng ta nên thuộc tên | three variants whose names we should know by heart |
| nó dùng để khởi tạo lười | it is used for lazy initialisation |
| khi thật sự có người cần đến nó | when somebody really needs it |
| kiểm tra quyền trước khi nó cho phép gọi vào | checks permissions before it allows a call into |
| nó đại diện cho một object nằm ở xa | it stands for an object that sits far away |
| có sẵn một đối tượng tên là `Proxy` | has a built-in object named `Proxy` |
| chặn các thao tác truy cập thuộc tính | intercept property access operations |
| điểm chung của cả ba biến thể là | what all three variants have in common is that |
| đang nói chuyện với chính chủ | talking to the real owner |

**Thuật ngữ cần nhớ**

- khởi tạo lười → **lazy initialisation**
- quyền truy cập → **permission**
- chặn, đón đường → **intercept**
- có sẵn trong ngôn ngữ → **built-in**
- thuộc tính → **property**

---

## Phần 5 — Decorator pattern: chồng thêm lớp mà không sửa lõi

**Tiếng Việt**

Decorator bọc một object bằng một lớp khác có cùng interface, và nó thêm trách nhiệm mới ngay lúc runtime. Vì interface không đổi, chúng ta chồng được nhiều lớp lên nhau, ví dụ một lớp ghi log bọc ngoài một lớp cache, còn lớp cache thì bọc ngoài service thật. Cách làm này rất hợp với OCP, bởi vì chúng ta mở rộng hành vi mà chúng ta không sửa class gốc. Ví dụ thực tế là chúng ta thêm caching cho một service chỉ bằng cách bọc nó lại, và khi cần tắt caching thì chúng ta chỉ việc không bọc nữa. Trong hệ sinh thái backend, middleware và interceptor chính là ý tưởng này ở quy mô request, bởi vì chúng bọc quanh handler để thêm log, thêm xác thực và thêm cache. Nếu chúng ta không có Decorator, chúng ta sẽ nhét những mối quan tâm chung đó vào từng service, và code nghiệp vụ sẽ ngập trong thứ không phải nghiệp vụ.

**English (bám cấu trúc tiếng Việt)**

The Decorator wraps an object with another layer that has the same interface, and it adds a new responsibility right at runtime. Because the interface does not change, we can stack many layers on top of each other, for example a logging layer wraps around a caching layer, while the caching layer wraps around the real service. This way fits OCP very well, because we extend the behaviour and we do not edit the original class. A practical example is that we add caching to a service only by wrapping it, and when we need to turn caching off then we simply do not wrap it any more. In the backend ecosystem, middleware and interceptors are exactly this idea at the request level, because they wrap around the handler to add logging, to add authentication and to add caching. If we do not have the Decorator, we will stuff those shared concerns into every service, and the business code will drown in things that are not business.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| bọc một object bằng một lớp khác có cùng interface | wraps an object with another layer that has the same interface |
| thêm trách nhiệm mới ngay lúc runtime | adds a new responsibility right at runtime |
| chồng được nhiều lớp lên nhau | can stack many layers on top of each other |
| bọc ngoài service thật | wraps around the real service |
| chúng ta không sửa class gốc | we do not edit the original class |
| chỉ bằng cách bọc nó lại | only by wrapping it |
| chúng ta chỉ việc không bọc nữa | we simply do not wrap it any more |
| ở quy mô request | at the request level |
| những mối quan tâm chung đó | those shared concerns |
| ngập trong thứ không phải nghiệp vụ | drown in things that are not business |

**Thuật ngữ cần nhớ**

- chồng lớp → **stack layers**
- lớp bọc → **wrapper**
- xác thực → **authentication**
- mối quan tâm chung → **cross-cutting concern**
- bộ chặn request → **interceptor**

---

## Phần 6 — Decorator của GoF không phải decorator của TypeScript

**Tiếng Việt**

Đây là cái bẫy cực kỳ phổ biến với người làm TypeScript và Nest. Decorator của GoF là một pattern, và nó bọc một object để thêm hành vi lúc chạy bằng composition. Decorator trong TypeScript và Nest, ví dụ `@Injectable` hay `@Get`, là một cú pháp chú thích, và nó gắn metadata hoặc biến đổi phần khai báo ngay lúc class được định nghĩa. Hai thứ này trùng tên nhưng khác bản chất, cho nên chúng ta phải nói rõ mình đang bàn về cái nào. Trong phỏng vấn, một câu ngắn gọn kiểu "decorator của GoF bọc object lúc runtime, còn decorator của TypeScript gắn metadata lúc định nghĩa class" đủ để cho thấy chúng ta phân biệt được. Nếu chúng ta trả lời gộp hai thứ làm một, người phỏng vấn sẽ nghĩ rằng chúng ta chỉ quen dùng framework chứ chúng ta chưa hiểu pattern.

**English (bám cấu trúc tiếng Việt)**

This is the extremely common trap for people working with TypeScript and Nest. The GoF Decorator is a pattern, and it wraps an object to add behaviour at run time through composition. The decorator in TypeScript and Nest, for example `@Injectable` or `@Get`, is an annotation syntax, and it attaches metadata or transforms the declaration right at the moment the class is defined. These two things share a name but differ in nature, so we have to say clearly which one we are talking about. In an interview, a short sentence such as "the GoF decorator wraps an object at runtime, while the TypeScript decorator attaches metadata at class definition time" is enough to show that we can tell them apart. If we answer by merging the two things into one, the interviewer will think that we are only used to the framework and we have not understood the pattern.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| cái bẫy cực kỳ phổ biến với | the extremely common trap for |
| thêm hành vi lúc chạy bằng composition | add behaviour at run time through composition |
| một cú pháp chú thích | an annotation syntax |
| nó gắn metadata | it attaches metadata |
| biến đổi phần khai báo | transforms the declaration |
| ngay lúc class được định nghĩa | right at the moment the class is defined |
| trùng tên nhưng khác bản chất | share a name but differ in nature |
| nói rõ mình đang bàn về cái nào | say clearly which one we are talking about |
| chúng ta chỉ quen dùng framework | we are only used to the framework |

**Thuật ngữ cần nhớ**

- chú thích, ký hiệu gắn kèm → **annotation**
- dữ liệu mô tả → **metadata**
- khai báo → **declaration**
- cú pháp → **syntax**
- khác bản chất → **differ in nature**

---

## Phần 7 — Composite: xử lý cây mà không cần biết đang cầm lá hay nhánh

**Tiếng Việt**

Composite dùng để xử lý các cấu trúc dạng cây, tức là những object có thể chứa các object con cùng kiểu. Ý tưởng cốt lõi là chúng ta cho lá và nút gộp cùng chung một interface, nhờ đó client xử lý chúng một cách đồng nhất. Khi client muốn tính tổng, nó chỉ gọi cùng một method, còn phần đệ quy thì nằm bên trong nút gộp chứ phần đệ quy không nằm ở phía client. Ví dụ ngoài giao diện người dùng có rất nhiều, ví dụ hệ thống file gồm thư mục và tệp, sơ đồ tổ chức của công ty, bảng vật tư của một sản phẩm, và cây phân quyền. Lợi ích lớn nhất là client không phải viết những câu điều kiện để hỏi xem đây là lá hay đây là nhánh. Nếu chúng ta không dùng Composite, mỗi chỗ duyệt cây lại phải tự viết đệ quy kèm kiểm tra kiểu, và những đoạn đó sẽ lệch nhau sau vài tháng.

**English (bám cấu trúc tiếng Việt)**

The Composite is used to handle tree-shaped structures, that is, objects that can hold child objects of the same type. The core idea is that we give the leaf and the composite node the same interface, thanks to that the client handles them uniformly. When the client wants to compute a total, it only calls the same method, while the recursion sits inside the composite node and the recursion does not sit on the client side. There are many examples outside the user interface, for example a file system made of folders and files, the organisation chart of a company, the bill of materials of a product, and a permission tree. The biggest benefit is that the client does not have to write conditions asking whether this is a leaf or whether this is a branch. If we do not use the Composite, every place that walks the tree has to write its own recursion together with type checks, and those pieces will drift apart after a few months.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| các cấu trúc dạng cây | tree-shaped structures |
| chứa các object con cùng kiểu | hold child objects of the same type |
| cho lá và nút gộp cùng chung một interface | give the leaf and the composite node the same interface |
| client xử lý chúng một cách đồng nhất | the client handles them uniformly |
| phần đệ quy nằm bên trong nút gộp | the recursion sits inside the composite node |
| sơ đồ tổ chức của công ty | the organisation chart of a company |
| bảng vật tư của một sản phẩm | the bill of materials of a product |
| để hỏi xem đây là lá hay đây là nhánh | asking whether this is a leaf or whether this is a branch |
| mỗi chỗ duyệt cây | every place that walks the tree |
| những đoạn đó sẽ lệch nhau sau vài tháng | those pieces will drift apart after a few months |

**Thuật ngữ cần nhớ**

- lá / nhánh → **leaf** / **branch**
- nút gộp → **composite node**
- đệ quy → **recursion**
- đồng nhất → **uniformly**
- duyệt cây → **walk the tree** / **traverse the tree**

---

## Phần 8 — Bridge: tách hai trục để không nổ số class, và soát code AI

**Tiếng Việt**

Bridge tách abstraction ra khỏi implementation để hai bên biến thiên độc lập với nhau. Dấu hiệu nhận ra Bridge là bài toán có hai trục biến thiên độc lập, ví dụ một trục là loại báo cáo gồm PDF và HTML, còn trục kia là nơi kết xuất gồm màn hình và máy in. Nếu chúng ta dùng kế thừa thẳng cho hai trục đó, số class sẽ bằng tích của hai trục, và người ta gọi hiện tượng này là bùng nổ tổ hợp. Bridge nối hai phân cấp lại bằng composition, cho nên số class chỉ còn bằng tổng của hai trục. Điểm khác biệt so với Adapter nằm ở thời điểm, bởi vì Bridge được thiết kế từ trước để chủ động tách hai trục, còn Adapter là thứ chúng ta thêm vào sau để làm hai thứ có sẵn tương thích với nhau. Khi phỏng vấn, nói được đúng cặp "Bridge thiết kế trước, Adapter chữa sau" là cách nhanh nhất để cho thấy chúng ta không nhầm hai pattern này.

Nhóm Structural cũng có hai lỗi rất hay gặp khi chúng ta soát code do AI sinh ra. Lỗi thứ nhất là AI lẫn hai loại decorator, nghĩa là nó sinh ra một chú thích `@` trong khi thứ chúng ta cần là một lớp bọc, hoặc nó làm ngược lại. Lỗi thứ hai là AI biến Facade thành một God object, bởi vì gom hết mọi thứ vào một class trông rất tiện khi sinh code. Khi review, chúng ta hỏi hai câu: đoạn này đang phục vụ ý đồ nào trong ba ý đồ tương thích, đơn giản hoá và kiểm soát, và class Facade này đang điều phối hay đang chứa rule nghiệp vụ. Hai câu hỏi đó đủ để lôi ra phần lớn lỗi thiết kế trong nhóm này.

**English (bám cấu trúc tiếng Việt)**

The Bridge separates the abstraction from the implementation so that the two sides can vary independently of each other. The sign for recognising the Bridge is that the problem has two independent axes of change, for example one axis is the report type made of PDF and HTML, while the other axis is the rendering target made of screen and printer. If we use direct inheritance for those two axes, the number of classes will be the product of the two axes, and people call this phenomenon a combinatorial explosion. The Bridge connects the two hierarchies through composition, so the number of classes is only the sum of the two axes. The difference from the Adapter lies in the timing, because the Bridge is designed up front to split the two axes deliberately, while the Adapter is something we add later to make two existing things compatible with each other. In an interview, being able to say exactly the pair "the Bridge is designed up front, the Adapter fixes things afterwards" is the fastest way to show that we do not confuse these two patterns.

The Structural group also has two mistakes that we meet very often when we review code that AI generates. The first mistake is that AI confuses the two kinds of decorator, which means that it produces an `@` annotation while the thing we need is a wrapper layer, or it does the opposite. The second mistake is that AI turns the Facade into a God object, because gathering everything into one class looks very convenient while generating code. When we review, we ask two questions: which intent among the three intents of compatibility, simplification and control does this piece serve, and is this Facade class coordinating or is it holding business rules. Those two questions are enough to pull out most of the design mistakes in this group.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| để hai bên biến thiên độc lập với nhau | so that the two sides can vary independently of each other |
| dấu hiệu nhận ra Bridge là | the sign for recognising the Bridge is that |
| nơi kết xuất gồm màn hình và máy in | the rendering target made of screen and printer |
| số class sẽ bằng tích của hai trục | the number of classes will be the product of the two axes |
| người ta gọi hiện tượng này là bùng nổ tổ hợp | people call this phenomenon a combinatorial explosion |
| chỉ còn bằng tổng của hai trục | is only the sum of the two axes |
| điểm khác biệt … nằm ở thời điểm | the difference … lies in the timing |
| được thiết kế từ trước để chủ động tách | is designed up front to split … deliberately |
| Adapter chữa sau | the Adapter fixes things afterwards |
| trong khi thứ chúng ta cần là một lớp bọc | while the thing we need is a wrapper layer |
| đủ để lôi ra phần lớn lỗi thiết kế | are enough to pull out most of the design mistakes |

**Thuật ngữ cần nhớ**

- biến thiên độc lập → **vary independently**
- tích / tổng → **product** / **sum**
- bùng nổ tổ hợp → **combinatorial explosion**
- kết xuất, dựng hình → **rendering**
- từ trước, ngay từ đầu → **up front**

---

## Mô hình ghi nhớ

**Tiếng Việt**

Adapter đổi phích cắm, Facade làm lễ tân, Proxy làm người gác cổng, còn Decorator dán thêm từng lớp lên object. Composite gộp cả cây lại thành một thứ để client cầm, còn Bridge tách hai trục ra để số class không nổ theo tích.

**English (bám cấu trúc tiếng Việt)**

The Adapter changes the plug, the Facade works as the receptionist, the Proxy works as the gatekeeper, while the Decorator sticks one more layer onto the object. The Composite folds a whole tree into one thing for the client to hold, while the Bridge splits two axes apart so that the number of classes does not explode as a product.

---

## Bảng thuật ngữ tổng hợp

| Tiếng Việt | Tiếng Anh | Ghi chú phát âm |
|---|---|---|
| sự linh hoạt | flexibility | trọng âm âm ba: flex-i-BI-li-ty |
| bọc | wrap | /ræp/ — chữ **w** câm, đọc giống "rap"; *wrapper* cũng vậy: "RA-pơ" |
| ý đồ | intent | trọng âm cuối: in-TENT |
| mảnh ghép | piece / building block | *piece* /piːs/ — đuôi **-s**, không đọc /piːz/ |
| kiến trúc | architecture | /ˈɑːkɪtektʃə/ — AR-chi-tec-ture, âm **ch** đầu đọc như "k" |
| bên thứ ba | third-party | *third* có âm **th** đầu và đuôi **-rd** |
| hệ thống cũ để lại | legacy | /ˈlegəsi/ — LE-ga-cy, trọng âm đầu |
| quy ước | convention | trọng âm âm hai: con-VEN-tion |
| thấm vào | seep into | *seep* /siːp/ — đọc dài "siip" |
| lớp chống nhiễm bẩn | Anti-Corruption Layer | *corruption* trọng âm âm hai: co-RUP-tion |
| tương thích | compatible / compatibility | *compatible* trọng âm âm hai: com-PA-ti-ble |
| hệ thống con | subsystem | trọng âm đầu: SUB-sys-tem |
| mặt tiền, mặt gọn | facade | /fəˈsɑːd/ — "fơ-SAAD", chữ **c** đọc như "s", KHÔNG đọc "fa-kêt" |
| lễ tân | receptionist | trọng âm âm hai: re-CEP-tion-ist |
| người đại diện | representative | trọng âm âm ba: re-pre-SEN-ta-tive |
| phình to | swell | đuôi **-ll** đọc rõ |
| ranh giới | boundary | /ˈbaʊndri/ — BOUN-dry, nói nhanh còn hai âm |
| khởi tạo lười | lazy initialisation | |
| quyền truy cập | permission | trọng âm âm hai: per-MI-ssion |
| chặn, đón đường | intercept | trọng âm cuối: in-ter-CEPT |
| có sẵn trong ngôn ngữ | built-in | |
| thuộc tính | property | trọng âm đầu: PRO-per-ty |
| chồng lớp | stack layers | |
| lớp bọc | wrapper | /ˈræpə/ — chữ **w** câm: "RA-pơ" |
| xác thực | authentication | trọng âm áp chót: au-then-ti-CA-tion, có âm **th** ở giữa |
| mối quan tâm chung | cross-cutting concern | |
| bộ chặn request | interceptor | trọng âm âm ba: in-ter-CEP-tor |
| chú thích gắn kèm | annotation | trọng âm áp chót: an-no-TA-tion |
| dữ liệu mô tả | metadata | Anh đọc /ˈmetədeɪtə/ — ME-ta-day-ta |
| khai báo | declaration | trọng âm áp chót: dec-la-RA-tion |
| cú pháp | syntax | trọng âm đầu: SYN-tax |
| khác bản chất | differ in nature | *nature* /ˈneɪtʃə/ — NAY-chơ |
| lá / nhánh | leaf / branch | *leaf* số nhiều là *leaves* /liːvz/ |
| nút gộp | composite node | *composite* Anh đọc /ˈkɒmpəzɪt/ — COM-po-zit |
| đệ quy | recursion | trọng âm âm hai: re-CUR-sion |
| đồng nhất | uniformly | trọng âm đầu: U-ni-form-ly |
| duyệt cây | walk the tree / traverse the tree | *traverse* trọng âm cuối: tra-VERSE |
| biến thiên độc lập | vary independently | *vary* /ˈveəri/ — "VE-ri", không đọc giống *very* dù rất gần |
| tích / tổng | product / sum | |
| bùng nổ tổ hợp | combinatorial explosion | *combinatorial* — com-bi-na-TO-rial, đọc chậm từng âm |
| kết xuất, dựng hình | rendering | |
| từ trước, ngay từ đầu | up front | |

---

## Bài nói

> **Cách làm:** ghi âm lại giọng của mình, mỗi câu nói liên tục **60–90 giây**, **không nhìn tài liệu** khi nói. Mỗi bài nói phải dùng ít nhất **6 thuật ngữ** trong bảng ở trên. Nghe lại bản ghi âm một lần, ghi ra hai chỗ mình bị vấp, rồi nói lại lần hai.

1. Explain the difference in intent between the Adapter, the Facade and the Proxy, and give one backend example for each.
2. A colleague says: "The Facade is already the single entry point for this module, so let us put the pricing rules in there as well." Explain why you would push back and where those rules belong.
3. Explain to a junior developer why the GoF Decorator and the `@Injectable` decorator in Nest are not the same thing, even though they share a name.
4. Describe how you would add caching to an existing service without editing that service, and what the object graph looks like when logging is stacked on top.
5. Describe what happens to the number of classes when you model report formats and rendering targets with plain inheritance, and how the Bridge changes that number.
6. When would you choose an Adapter and when would you choose a Bridge? Give the trade-off in both directions and mention the timing of the decision.
7. You are reviewing AI-generated code that wraps a third-party SMS SDK. Describe out loud the questions you would ask about the intent of the wrapper and about where the business rules live.
