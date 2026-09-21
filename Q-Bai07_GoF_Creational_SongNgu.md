# Bài 7 — GoF Creational: Factory · Abstract Factory · Builder · Singleton · Prototype
### Bản song ngữ: tiếng Anh bám cấu trúc tiếng Việt

> **Cách dùng bài này:** đọc đoạn tiếng Việt trước, tự dịch trong đầu, rồi mới nhìn sang đoạn tiếng Anh để so. Sau khi so xong, **đọc to đoạn tiếng Anh hai lần** — lần một để nhìn chữ, lần hai để nghe chính mình. Phần bài nói ở cuối là bắt buộc; không nói ra thì bài này chỉ là bài đọc.

---

## Phần 1 — Ba nhóm GoF và vị trí của nhóm Creational

**Tiếng Việt**

GoF, tức là nhóm Gang of Four, đã chia hai mươi ba pattern thành ba nhóm vào năm 1994. Nhóm Creational lo việc tạo object, và đại diện quen thuộc nhất của nhóm này là Factory. Nhóm Structural lo việc ghép và sắp xếp object, và đại diện của nó là Adapter. Nhóm Behavioral lo việc giao tiếp và phân chia trách nhiệm giữa các object, và đại diện của nó là Strategy. Bài này lo nhóm Creational, nghĩa là chúng ta học cách tách quyết định tạo ra cái gì ra khỏi nơi dùng cái đó. Khi học xong, chúng ta sẽ thấy nhiều pattern vốn cồng kềnh trong Java lại co lại rất nhiều trong TypeScript nhờ first-class function và module, đúng như Bài 3 đã nói.

**English (bám cấu trúc tiếng Việt)**

GoF, that is, the Gang of Four, split twenty-three patterns into three groups in 1994. The Creational group takes care of creating objects, and the most familiar representative of this group is the Factory. The Structural group takes care of combining and arranging objects, and its representative is the Adapter. The Behavioral group takes care of communication and of dividing responsibilities between objects, and its representative is the Strategy. This lesson takes care of the Creational group, which means that we learn how to separate the decision about what to create from the place that uses it. When we finish, we will see that many patterns which are heavy in Java shrink a lot in TypeScript thanks to first-class functions and modules, exactly as Lesson 3 said.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| đã chia … thành ba nhóm | split … into three groups |
| lo việc tạo object | takes care of creating objects |
| đại diện quen thuộc nhất của nhóm này | the most familiar representative of this group |
| ghép và sắp xếp object | combining and arranging objects |
| phân chia trách nhiệm giữa các object | dividing responsibilities between objects |
| tách quyết định tạo ra cái gì ra khỏi nơi dùng cái đó | separate the decision about what to create from the place that uses it |
| vốn cồng kềnh trong Java | which are heavy in Java |
| co lại rất nhiều trong TypeScript | shrink a lot in TypeScript |

**Thuật ngữ cần nhớ**

- nhóm tạo object → **Creational patterns**
- nhóm cấu trúc → **Structural patterns**
- nhóm hành vi → **Behavioral patterns**
- đại diện → **representative**
- cồng kềnh → **heavy** / **bulky**

---

## Phần 2 — Factory Method và lý do `new X()` không phải factory

**Tiếng Việt**

Factory Method tách quyết định tạo ra class cụ thể nào ra khỏi nơi dùng object đó. Nó giấu đi phần logic khởi tạo và phần logic lựa chọn, cho nên nơi gọi chỉ cần nói mình muốn gì chứ nơi gọi không cần biết thứ đó được dựng lên như thế nào. Ở đây có một điểm rất hay bị nhầm, đó là việc gọi thẳng `new X()` không phải là factory. Lý do là câu lệnh đó lộ ra kiểu cụ thể ngay tại chỗ gọi, cho nên nó không tách được hai bên ra khỏi nhau. Một factory đúng nghĩa trả về một abstraction, và người gọi không biết class cụ thể nào nằm phía sau. Nếu chúng ta không phân biệt được hai thứ này, chúng ta sẽ trả lời hụt ở một câu hỏi phỏng vấn rất phổ biến.

**English (bám cấu trúc tiếng Việt)**

The Factory Method separates the decision about which concrete class to create from the place that uses that object. It hides the creation logic and the selection logic, so the calling site only has to say what it wants and the calling site does not need to know how that thing is built. Here there is a point that is very often confused, which is that calling `new X()` directly is not a factory. The reason is that this statement exposes the concrete type right at the call site, so it cannot separate the two sides from each other. A real factory returns an abstraction, and the caller does not know which concrete class sits behind it. If we cannot tell these two things apart, we will answer short on a very common interview question.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| quyết định tạo ra class cụ thể nào | the decision about which concrete class to create |
| nó giấu đi phần logic khởi tạo | it hides the creation logic |
| nơi gọi | the calling site / the call site |
| được dựng lên như thế nào | how that thing is built |
| một điểm rất hay bị nhầm | a point that is very often confused |
| lộ ra kiểu cụ thể ngay tại chỗ gọi | exposes the concrete type right at the call site |
| một factory đúng nghĩa | a real factory |
| class cụ thể nào nằm phía sau | which concrete class sits behind it |
| nếu chúng ta không phân biệt được hai thứ này | if we cannot tell these two things apart |

**Thuật ngữ cần nhớ**

- logic khởi tạo → **creation logic**
- nơi gọi → **the call site**
- người gọi → **the caller**
- phân biệt → **tell apart** / **distinguish**
- trả về → **return**

---

## Phần 3 — Factory kiểu TypeScript: một map hàm

**Tiếng Việt**

Trong Node và TypeScript, cái mà chúng ta gọi là factory thường chỉ là một object map ánh xạ từ tên loại sang hàm tạo. Ví dụ chúng ta có một map gồm khoá `pdf` trỏ tới hàm tạo bản PDF và khoá `csv` trỏ tới hàm tạo bản CSV, thay vì một cây class kèm một câu switch. Cái được của cách này là code gọn, việc thêm một loại gần như chỉ là thêm một mục vào map, và chúng ta tận dụng được first-class function. Cái mất là ràng buộc kiểu lỏng hơn, và chúng ta có ít khung đỡ hơn khi phải xử lý các biến thể phức tạp hoặc nhiều dòng sản phẩm. Cách chọn là chúng ta nhìn vào độ phức tạp, nghĩa là chúng ta dùng map cho trường hợp đơn giản và dùng phân cấp class khi thật sự cần cấu trúc. Khi trả lời phỏng vấn, việc nêu được cả cái được lẫn cái mất quan trọng hơn việc chọn phe.

**English (bám cấu trúc tiếng Việt)**

In Node and TypeScript, the thing we call a factory is usually only an object map from the type name to a creating function. For example we have a map where the key `pdf` points to the function that creates the PDF version and the key `csv` points to the function that creates the CSV version, instead of a class tree together with a switch statement. The gain of this way is that the code is tidy, adding a type is almost only adding one entry to the map, and we make good use of first-class functions. The loss is that the type constraints are looser, and we have less scaffolding when we have to handle complex variants or several product lines. The way to choose is that we look at the complexity, which means that we use a map for the simple case and we use a class hierarchy when we really need structure. In an interview, being able to state both the gain and the loss matters more than picking a side.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| ánh xạ từ tên loại sang hàm tạo | from the type name to a creating function |
| trỏ tới hàm tạo bản PDF | points to the function that creates the PDF version |
| thay vì một cây class kèm một câu switch | instead of a class tree together with a switch statement |
| cái được của cách này là | the gain of this way is that |
| gần như chỉ là thêm một mục vào map | is almost only adding one entry to the map |
| chúng ta tận dụng được | we make good use of |
| ràng buộc kiểu lỏng hơn | the type constraints are looser |
| chúng ta có ít khung đỡ hơn | we have less scaffolding |
| nhiều dòng sản phẩm | several product lines |
| quan trọng hơn việc chọn phe | matters more than picking a side |

**Thuật ngữ cần nhớ**

- mục trong map → **entry**
- ràng buộc kiểu → **type constraint**
- khung đỡ → **scaffolding**
- độ phức tạp → **complexity**
- chọn phe → **pick a side**

---

## Phần 4 — Abstract Factory tạo cả một họ sản phẩm

**Tiếng Việt**

Abstract Factory khác Factory Method ở quy mô của thứ được tạo ra. Factory Method tạo ra một sản phẩm, còn Abstract Factory tạo ra cả một họ sản phẩm liên quan và bảo đảm rằng chúng cùng thuộc một gia đình. Ví dụ dễ hình dung nhất là một bộ giao diện theo chủ đề, trong đó `Button` và `Checkbox` phải cùng là bản tối hoặc cùng là bản sáng. Một ví dụ khác là một bộ driver phải cùng một nhà cung cấp, bởi vì trộn driver của hai nhà cung cấp sẽ gây ra những lỗi rất khó tìm. Chúng ta cần pattern này khi các object bắt buộc phải đi cùng nhau đúng bộ. Nếu chúng ta dùng nhiều factory rời rạc trong tình huống đó, thì sớm muộn cũng có người ghép nhầm một sản phẩm của bộ này với một sản phẩm của bộ kia.

**English (bám cấu trúc tiếng Việt)**

The Abstract Factory differs from the Factory Method in the scale of the thing that is created. The Factory Method creates one product, while the Abstract Factory creates a whole family of related products and guarantees that they belong to the same family. The example that is easiest to picture is a themed user-interface set, in which `Button` and `Checkbox` must both be the dark version or both be the light version. Another example is a driver set that must come from the same vendor, because mixing drivers from two vendors will cause bugs that are very hard to find. We need this pattern when the objects are required to go together as a matching set. If we use several separate factories in that situation, then sooner or later somebody will wrongly combine a product from one set with a product from the other set.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| khác … ở quy mô của thứ được tạo ra | differs from … in the scale of the thing that is created |
| cả một họ sản phẩm liên quan | a whole family of related products |
| bảo đảm rằng chúng cùng thuộc một gia đình | guarantees that they belong to the same family |
| ví dụ dễ hình dung nhất | the example that is easiest to picture |
| một bộ giao diện theo chủ đề | a themed user-interface set |
| trộn driver của hai nhà cung cấp | mixing drivers from two vendors |
| bắt buộc phải đi cùng nhau đúng bộ | are required to go together as a matching set |
| nhiều factory rời rạc | several separate factories |
| sớm muộn cũng có người ghép nhầm | sooner or later somebody will wrongly combine |

**Thuật ngữ cần nhớ**

- họ sản phẩm → **a family of products**
- bảo đảm → **guarantee**
- chủ đề giao diện → **theme**
- nhà cung cấp → **vendor**
- đúng bộ → **a matching set**

---

## Phần 5 — Builder và câu hỏi "options object đã đủ chưa"

**Tiếng Việt**

Builder dùng để tạo một object phức tạp theo từng bước. Nó giúp chúng ta tránh cái gọi là telescoping constructor, tức là kiểu constructor nhận sáu bảy tham số liền nhau mà không ai nhớ nổi thứ tự của chúng. Mỗi bước trong builder có một cái tên rõ ràng, và object trở thành bất biến sau khi chúng ta gọi phương thức `build`. Tuy nhiên trong TypeScript, một options object dạng `new X({a, b, c})` thường đã đủ tốt và đủ rõ ràng. Builder chỉ thật sự đáng dùng khi giữa các bước có thứ tự bắt buộc, hoặc khi có kiểm tra hợp lệ phức tạp, hoặc khi chúng ta cần nhiều biến thể dựng khác nhau. Nếu chúng ta dựng một builder cho object chỉ có ba trường, chúng ta đang viết thêm rất nhiều code mà chúng ta không đổi lấy được lợi ích nào.

**English (bám cấu trúc tiếng Việt)**

The Builder is used to create a complex object step by step. It helps us avoid what is called the telescoping constructor, that is, the kind of constructor that takes six or seven parameters in a row where nobody can remember their order. Each step in the builder has a clear name, and the object becomes immutable after we call the `build` method. However, in TypeScript, an options object in the form of `new X({a, b, c})` is usually already good enough and clear enough. The Builder is only really worth using when the steps have a required order, or when there is complex validation, or when we need several different build variants. If we build a builder for an object that only has three fields, we are writing a lot of extra code and we are not getting any benefit in exchange.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| tạo một object phức tạp theo từng bước | create a complex object step by step |
| cái gọi là telescoping constructor | what is called the telescoping constructor |
| nhận sáu bảy tham số liền nhau | takes six or seven parameters in a row |
| không ai nhớ nổi thứ tự của chúng | nobody can remember their order |
| object trở thành bất biến | the object becomes immutable |
| thường đã đủ tốt và đủ rõ ràng | is usually already good enough and clear enough |
| chỉ thật sự đáng dùng khi | is only really worth using when |
| có thứ tự bắt buộc | have a required order |
| nhiều biến thể dựng khác nhau | several different build variants |
| chúng ta không đổi lấy được lợi ích nào | we are not getting any benefit in exchange |

**Thuật ngữ cần nhớ**

- constructor nhiều tham số chồng chất → **telescoping constructor**
- bất biến → **immutable**
- kiểm tra hợp lệ → **validation**
- trường dữ liệu → **field**
- đối tượng tuỳ chọn → **options object**

---

## Phần 6 — Vì sao Singleton bị coi là anti-pattern

**Tiếng Việt**

Singleton bị nhiều người coi là anti-pattern vì một lý do chính, đó là nó tạo ra global state ẩn. Trạng thái toàn cục làm cho việc test trở nên khó, bởi vì state rò từ bài test này sang bài test khác và kết quả phụ thuộc vào thứ tự chạy. Nó cũng tạo ra coupling ngầm, bởi vì bất kỳ ai cũng gọi được `getInstance` mà họ không phải khai báo phụ thuộc đó ở đâu cả. Ngoài ra chúng ta còn gặp các vấn đề về concurrency, về vòng đời của object, và về thứ tự khởi tạo giữa các thành phần. Có một điểm rất quan trọng với người làm Node, đó là module trong Node vốn đã là singleton sẵn. Lý do là module được cache sau lần import đầu tiên, cho nên một dòng `export const db = new Pool()` đã cho chúng ta một instance dùng chung cho toàn ứng dụng. Vì vậy chúng ta không cần tự cài đặt Singleton bằng tay, và việc tự cài đặt chỉ thêm code mà nó không thêm giá trị.

**English (bám cấu trúc tiếng Việt)**

The Singleton is seen by many people as an anti-pattern for one main reason, which is that it creates hidden global state. Global state makes testing hard, because state leaks from one test into another test and the result depends on the running order. It also creates implicit coupling, because anybody can call `getInstance` and they do not have to declare that dependency anywhere. In addition we also meet problems about concurrency, about the lifecycle of the object, and about the initialisation order between components. There is a very important point for people working with Node, which is that a module in Node is already a singleton by itself. The reason is that the module is cached after the first import, so one line of `export const db = new Pool()` already gives us one shared instance for the whole application. Therefore we do not need to implement a Singleton by hand, and implementing it by hand only adds code and it does not add value.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| bị nhiều người coi là | is seen by many people as |
| nó tạo ra global state ẩn | it creates hidden global state |
| state rò từ bài test này sang bài test khác | state leaks from one test into another test |
| kết quả phụ thuộc vào thứ tự chạy | the result depends on the running order |
| nó tạo ra coupling ngầm | it creates implicit coupling |
| họ không phải khai báo phụ thuộc đó ở đâu cả | they do not have to declare that dependency anywhere |
| vòng đời của object | the lifecycle of the object |
| thứ tự khởi tạo giữa các thành phần | the initialisation order between components |
| vốn đã là singleton sẵn | is already a singleton by itself |
| chúng ta không cần tự cài đặt … bằng tay | we do not need to implement … by hand |

**Thuật ngữ cần nhớ**

- trạng thái toàn cục ẩn → **hidden global state**
- rò rỉ → **leak**
- ngầm → **implicit**
- tính đồng thời → **concurrency**
- vòng đời → **lifecycle**
- lưu đệm → **cache**

---

## Phần 7 — "Chỉ một instance" không bắt buộc phải là Singleton

**Tiếng Việt**

Có một câu hỏi thách đố hay gặp, đó là câu "ứng dụng chỉ được có đúng một connection pool, vậy thì phải dùng Singleton chứ còn gì nữa". Câu trả lời của chúng ta là chuyện chỉ có một instance không bắt buộc phải đi kèm Singleton pattern. Cách làm được ưa chuộng hiện nay là chúng ta cung cấp instance đó qua DI, nghĩa là chúng ta để container giữ đúng một bản dùng chung. Nhờ cách này, chúng ta vẫn thay được nó trong test, chúng ta vẫn ghi đè được nó theo môi trường, và chúng ta vẫn kiểm soát được thứ tự khởi tạo. Singleton tự cài bằng tay thì ngược lại, bởi vì nó tạo ra một phụ thuộc toàn cục ẩn mà không ai nhìn thấy trong chữ ký của constructor. Chính các SDK lớn cũng đi theo hướng này, ví dụ client của AWS được tạo một lần rồi được inject đi khắp nơi, chứ nó không được lấy ra bằng một hàm tĩnh.

**English (bám cấu trúc tiếng Việt)**

There is a challenge question that we meet often, which is the question "the application is only allowed to have exactly one connection pool, so it must be a Singleton, what else could it be". Our answer is that having only one instance does not have to come together with the Singleton pattern. The preferred way nowadays is that we supply that instance through DI, which means that we let the container hold exactly one shared copy. Thanks to this way, we can still replace it in tests, we can still override it per environment, and we can still control the initialisation order. A hand-written Singleton is the opposite, because it creates a hidden global dependency that nobody can see in the constructor signature. The big SDKs themselves also follow this direction, for example the AWS client is created once and then injected everywhere, and it is not fetched through a static function.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| một câu hỏi thách đố hay gặp | a challenge question that we meet often |
| vậy thì phải dùng Singleton chứ còn gì nữa | so it must be a Singleton, what else could it be |
| không bắt buộc phải đi kèm | does not have to come together with |
| cách làm được ưa chuộng hiện nay | the preferred way nowadays |
| giữ đúng một bản dùng chung | hold exactly one shared copy |
| ghi đè được nó theo môi trường | override it per environment |
| kiểm soát được thứ tự khởi tạo | control the initialisation order |
| trong chữ ký của constructor | in the constructor signature |
| nó không được lấy ra bằng một hàm tĩnh | it is not fetched through a static function |

**Thuật ngữ cần nhớ**

- bể kết nối → **connection pool**
- được ưa chuộng → **preferred**
- ghi đè → **override**
- môi trường → **environment**
- chữ ký hàm → **signature**
- hàm tĩnh → **static function**

---

## Phần 8 — Prototype, clone nông và clone sâu, và soát code AI

**Tiếng Việt**

Pattern cuối trong nhóm này là Prototype, và ý tưởng của nó là tạo object mới bằng cách clone một mẫu có sẵn. Chúng ta dùng nó khi việc khởi tạo tốn kém, hoặc khi chúng ta không biết kiểu cụ thể của thứ cần tạo. JavaScript vốn là một ngôn ngữ dựa trên prototype, cho nên chúng ta có sẵn `Object.create` và `structuredClone` để làm việc này. Điều phải cảnh giác là khác biệt giữa clone nông và clone sâu, bởi vì clone nông chỉ sao chép tầng ngoài còn các object lồng bên trong vẫn dùng chung tham chiếu. Nếu chúng ta nhầm hai loại clone này, hai object tưởng là độc lập lại sửa lẫn dữ liệu của nhau, và loại lỗi đó rất khó tìm ra.

Nhóm Creational cũng là chỗ mà chúng ta phải soát code AI kỹ nhất. AI rất thích đẻ ra một Singleton kèm `getInstance`, và nó cũng thích dựng một Builder cồng kềnh cho thứ chỉ cần một options object. Khi review, chúng ta hỏi hai câu: đoạn này có biến thành global state ẩn hay không, và cái builder này có thật sự cần thiết hay không. Nếu có global state ẩn, chúng ta chuyển sang module export hoặc chúng ta chuyển sang DI. Nếu builder không cần thiết, chúng ta thay nó bằng một options object và chúng ta xoá bớt vài chục dòng code.

**English (bám cấu trúc tiếng Việt)**

The last pattern in this group is the Prototype, and its idea is to create a new object by cloning an existing template. We use it when the initialisation is expensive, or when we do not know the concrete type of the thing we need to create. JavaScript is by nature a prototype-based language, so we already have `Object.create` and `structuredClone` to do this work. The thing we have to watch out for is the difference between a shallow clone and a deep clone, because a shallow clone only copies the outer layer while the nested objects inside still share the same reference. If we confuse these two kinds of clone, two objects that we think are independent will edit each other's data, and that kind of bug is very hard to track down.

The Creational group is also the place where we have to review AI code the most carefully. AI really likes to produce a Singleton with `getInstance`, and it also likes to build a heavy Builder for something that only needs an options object. When we review, we ask two questions: does this piece turn into hidden global state or not, and is this builder really necessary or not. If there is hidden global state, we move to a module export or we move to DI. If the builder is not necessary, we replace it with an options object and we delete a few dozen lines of code.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| bằng cách clone một mẫu có sẵn | by cloning an existing template |
| khi việc khởi tạo tốn kém | when the initialisation is expensive |
| vốn là một ngôn ngữ dựa trên prototype | is by nature a prototype-based language |
| điều phải cảnh giác là | the thing we have to watch out for is |
| chỉ sao chép tầng ngoài | only copies the outer layer |
| các object lồng bên trong vẫn dùng chung tham chiếu | the nested objects inside still share the same reference |
| hai object tưởng là độc lập | two objects that we think are independent |
| sửa lẫn dữ liệu của nhau | edit each other's data |
| rất khó tìm ra | is very hard to track down |
| chúng ta xoá bớt vài chục dòng code | we delete a few dozen lines of code |

**Thuật ngữ cần nhớ**

- bản mẫu → **template**
- sao chép nông / sao chép sâu → **shallow clone** / **deep clone**
- lồng nhau → **nested**
- tham chiếu → **reference**
- lần ra, truy ra → **track down**

---

## Mô hình ghi nhớ

**Tiếng Việt**

Nhóm Creational là nhóm giấu đi chuyện object được sinh ra như thế nào. Trong TypeScript, factory thường là một map hàm, Singleton thường chỉ là một module, và Builder chỉ đáng dùng khi options object không còn đủ.

**English (bám cấu trúc tiếng Việt)**

The Creational group is the group that hides how an object is born. In TypeScript, a factory is usually a map of functions, a Singleton is usually only a module, and a Builder is only worth using when an options object is no longer enough.

---

## Bảng thuật ngữ tổng hợp

| Tiếng Việt | Tiếng Anh | Ghi chú phát âm |
|---|---|---|
| nhóm tạo object | Creational patterns | *creational* trọng âm âm hai: cre-A-tion-al |
| nhóm cấu trúc | Structural patterns | *structural* — cụm **-str-** phải bật: STRUC-tur-al |
| nhóm hành vi | Behavioral patterns | trọng âm âm hai: be-HA-vio-ral |
| đại diện | representative | trọng âm âm ba: re-pre-SEN-ta-tive |
| cồng kềnh | heavy / bulky | *bulky* — đuôi **-lk-** phải bật |
| logic khởi tạo | creation logic | |
| nơi gọi | the call site | |
| người gọi | the caller | |
| phân biệt | tell apart / distinguish | *distinguish* trọng âm âm hai: dis-TIN-guish, chữ **u** sau **g** đọc "gw" |
| mục trong map | entry | |
| ràng buộc kiểu | type constraint | *constraint* — cụm **-str-** và đuôi **-nt** |
| khung đỡ | scaffolding | /ˈskæfəldɪŋ/ — SCA-ffol-ding, trọng âm đầu |
| độ phức tạp | complexity | trọng âm âm hai: com-PLEX-i-ty |
| họ sản phẩm | a family of products | |
| bảo đảm | guarantee | trọng âm cuối: gua-ran-TEE |
| chủ đề giao diện | theme | /θiːm/ — có âm **th**, đặt lưỡi giữa hai hàm răng |
| nhà cung cấp | vendor | |
| đúng bộ | a matching set | |
| constructor nhiều tham số chồng chất | telescoping constructor | *telescoping* trọng âm đầu: TE-le-sco-ping |
| bất biến | immutable | trọng âm âm hai: im-MU-ta-ble |
| kiểm tra hợp lệ | validation | trọng âm áp chót: va-li-DA-tion |
| trường dữ liệu | field | |
| đối tượng tuỳ chọn | options object | |
| trạng thái toàn cục ẩn | hidden global state | |
| rò rỉ | leak | |
| ngầm | implicit | trọng âm âm hai: im-PLI-cit |
| tính đồng thời | concurrency | trọng âm âm hai: con-CU-rren-cy |
| vòng đời | lifecycle | |
| lưu đệm | cache | /kæʃ/ — đọc giống "cash", KHÔNG đọc theo mặt chữ thành "ca-chê" |
| khởi tạo (bước đầu) | initialisation | i-ni-tia-li-SA-tion, đọc chậm từng âm |
| bể kết nối | connection pool | |
| được ưa chuộng | preferred | trọng âm cuối: pre-FERRED |
| ghi đè | override | trọng âm cuối: o-ver-RIDE |
| môi trường | environment | trọng âm âm hai: en-VI-ron-ment; người Việt hay nuốt âm **-ron-** |
| chữ ký hàm | signature | /ˈsɪgnətʃə/ — SIG-na-ture, đuôi đọc "-chơ" |
| hàm tĩnh | static function | |
| bản mẫu | template | Anh đọc /ˈtempleɪt/ — TEM-plate |
| sao chép nông / sâu | shallow clone / deep clone | *shallow* /ˈʃæləʊ/ — SHA-llow, không đọc "sa-lâu" |
| lồng nhau | nested | trọng âm đầu: NES-ted |
| tham chiếu | reference | /ˈrefrəns/ — nói nhanh còn hai âm: "REF-rợns" |
| lần ra, truy ra | track down | đuôi **-ck** phải bật rõ |

---

## Bài nói

> **Cách làm:** ghi âm lại giọng của mình, mỗi câu nói liên tục **60–90 giây**, **không nhìn tài liệu** khi nói. Mỗi bài nói phải dùng ít nhất **6 thuật ngữ** trong bảng ở trên. Nghe lại bản ghi âm một lần, ghi ra hai chỗ mình bị vấp, rồi nói lại lần hai.

1. Name the three GoF groups, say what problem each group solves, and give one representative pattern for each.
2. Explain to a junior developer why calling `new X()` inside a controller is not a factory, and what changes when you introduce one.
3. A colleague says: "We need exactly one connection pool for the whole app, so I added a `getInstance()` Singleton." Explain why you would push back and what you would propose instead.
4. Describe the difference between a Factory Method and an Abstract Factory, using an example where the objects must come from the same family.
5. Someone on your team wants a full Builder for a config object with four fields. Explain when a Builder earns its cost and why an options object is usually enough in TypeScript.
6. Describe what happens when a developer uses a shallow clone where a deep clone was needed, and how you would notice the bug.
7. You are reviewing AI-generated code for a report exporter, and it contains a `getInstance()` Singleton plus a Builder. Describe out loud the two questions you would ask and the changes you would request.
