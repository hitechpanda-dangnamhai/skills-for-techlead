# Bài 3 — DRY / KISS / YAGNI · OOP vs Functional trong TypeScript
### Bản song ngữ: tiếng Anh bám cấu trúc tiếng Việt

> **Cách dùng bài này:** đọc đoạn tiếng Việt trước, tự dịch trong đầu, rồi mới nhìn sang đoạn tiếng Anh để so. Sau khi so xong, **đọc to đoạn tiếng Anh hai lần** — lần một để nhìn chữ, lần hai để nghe chính mình. Phần bài nói ở cuối là bắt buộc; không nói ra thì bài này chỉ là bài đọc.

---

## Phần 1 — Ba chữ này là bộ phanh chống over-engineering

**Tiếng Việt**

Ba chữ DRY, KISS và YAGNI là bộ phanh chống over-engineering của chúng ta. Chúng cực kỳ quan trọng cho phần judgment của một Tech Lead mà chúng ta sẽ học ở Bài 15, bởi vì phần lớn thiệt hại trong một dự án không đến từ code quá đơn giản, mà đến từ code phức tạp hơn mức cần thiết. Bài này cũng phá một định kiến rất phổ biến, đó là định kiến rằng thiết kế tốt nghĩa là OOP thuần. Trong TypeScript và Node, first-class function thường thay được nhiều pattern của GoF, và điều đó không có nghĩa là chúng ta thiếu thiết kế. Nếu chúng ta không có ba cái phanh này, thì mỗi lần chúng ta học được một pattern mới, chúng ta sẽ tìm cách nhét nó vào dự án dù dự án không cần.

**English (bám cấu trúc tiếng Việt)**

The three words DRY, KISS and YAGNI are our brakes against over-engineering. They are extremely important for the judgment part of a Tech Lead that we will study in Lesson 15, because most of the damage in a project does not come from code that is too simple, but from code that is more complex than necessary. This lesson also breaks a very common prejudice, which is the prejudice that good design means pure OOP. In TypeScript and Node, first-class functions can often replace many GoF patterns, and that does not mean that we lack design. If we do not have these three brakes, then every time we learn a new pattern, we will look for a way to squeeze it into the project although the project does not need it.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| bộ phanh chống over-engineering | our brakes against over-engineering |
| phần lớn thiệt hại trong một dự án | most of the damage in a project |
| phức tạp hơn mức cần thiết | more complex than necessary |
| phá một định kiến rất phổ biến | breaks a very common prejudice |
| thiết kế tốt nghĩa là OOP thuần | good design means pure OOP |
| điều đó không có nghĩa là chúng ta thiếu thiết kế | that does not mean that we lack design |
| mỗi lần chúng ta học được một pattern mới | every time we learn a new pattern |
| tìm cách nhét nó vào dự án | look for a way to squeeze it into the project |

**Thuật ngữ cần nhớ**

- làm quá mức cần thiết → **over-engineering**
- khả năng phán đoán, sự cân nhắc → **judgment**
- thiệt hại → **damage**
- định kiến → **prejudice**
- hàm là công dân hạng nhất → **first-class function**

---

## Phần 2 — Phát biểu đúng của ba nguyên tắc

**Tiếng Việt**

Chúng ta hãy phát biểu ba nguyên tắc này thật chính xác, bởi vì người phỏng vấn thường hỏi phát biểu chứ họ không hỏi tên viết tắt. DRY, tức là Don't Repeat Yourself, nói rằng mỗi mẩu kiến thức hoặc mỗi quy tắc nghiệp vụ phải có một nguồn chân lý duy nhất trong hệ thống. KISS, tức là Keep It Simple, nói rằng chúng ta chọn giải pháp đơn giản nhất mà vẫn đủ dùng cho bài toán hiện tại. YAGNI, tức là You Aren't Gonna Need It, nói rằng chúng ta đừng xây trước những thứ chưa cần, dù chúng ta tin rằng biết đâu sau này sẽ cần. Ba nguyên tắc này cảnh báo ba loại lãng phí khác nhau, đó là lặp kiến thức, phức tạp thừa, và làm trước thứ chưa ai yêu cầu. Nếu chúng ta chỉ nhớ tên viết tắt mà phát biểu sai nội dung, chúng ta sẽ áp dụng sai và làm hỏng thiết kế nhân danh nguyên tắc.

**English (bám cấu trúc tiếng Việt)**

Let us state these three principles very precisely, because interviewers usually ask for the statement and they do not ask for the acronym. DRY, that is, Don't Repeat Yourself, says that each piece of knowledge or each business rule must have a single source of truth in the system. KISS, that is, Keep It Simple, says that we choose the simplest solution that is still good enough for the current problem. YAGNI, that is, You Aren't Gonna Need It, says that we should not build in advance the things we do not need yet, although we believe that we might need them later. These three principles warn against three different kinds of waste, which are duplicated knowledge, unnecessary complexity, and work done before anyone asked for it. If we only remember the acronym but state the content wrongly, we will apply it wrongly and damage the design in the name of the principle.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| chúng ta hãy phát biểu … thật chính xác | let us state … very precisely |
| họ không hỏi tên viết tắt | they do not ask for the acronym |
| mỗi mẩu kiến thức | each piece of knowledge |
| một nguồn chân lý duy nhất | a single source of truth |
| đơn giản nhất mà vẫn đủ dùng | the simplest … that is still good enough |
| đừng xây trước những thứ chưa cần | should not build in advance the things we do not need yet |
| biết đâu sau này sẽ cần | we might need them later |
| cảnh báo ba loại lãng phí khác nhau | warn against three different kinds of waste |
| phức tạp thừa | unnecessary complexity |
| làm hỏng thiết kế nhân danh nguyên tắc | damage the design in the name of the principle |

**Thuật ngữ cần nhớ**

- tên viết tắt → **acronym**
- nguồn chân lý duy nhất → **single source of truth**
- đủ dùng → **good enough**
- lãng phí → **waste**
- nhân danh → **in the name of**

---

## Phần 3 — DRY nói về kiến thức, không nói về ký tự

**Tiếng Việt**

DRY là nguyên tắc bị hiểu sai nặng nhất trong ba nguyên tắc trên. Rất nhiều người hiểu DRY là "trong code không được có hai đoạn giống nhau", nhưng đó không phải là điều mà nguyên tắc này nói. DRY nói về kiến thức, chứ nó không nói về ký tự. Hai đoạn code trông giống hệt nhau nhưng thay đổi vì hai lý do khác nhau thì chúng ta không nên gộp lại. Khi chúng ta gộp chúng lại, chúng ta tạo ra một coupling giả, và người ta gọi thứ đó là abstraction sai hoặc abstraction quá sớm. Ngày mai, khi một bên cần đổi mà bên kia không đổi, chúng ta buộc phải nhét thêm tham số và cờ rẽ nhánh vào hàm chung, và hàm chung đó dần dần méo mó. Câu thần chú mà chúng ta nên thuộc là câu này: hai thứ giống nhau hôm nay không bảo đảm rằng chúng sẽ cùng đổi vào ngày mai.

**English (bám cấu trúc tiếng Việt)**

DRY is the most badly misunderstood principle among the three principles above. Very many people understand DRY as "there must be no two identical blocks in the code", but that is not what this principle says. DRY talks about knowledge, and it does not talk about characters. Two blocks of code that look exactly the same but change for two different reasons should not be merged by us. When we merge them, we create a false coupling, and people call that thing a wrong abstraction or a premature abstraction. Tomorrow, when one side needs to change while the other side does not change, we are forced to push extra parameters and branching flags into the shared function, and that shared function gradually becomes distorted. The mantra that we should learn by heart is this one: two things that are the same today do not guarantee that they will change together tomorrow.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| bị hiểu sai nặng nhất | the most badly misunderstood |
| đó không phải là điều mà nguyên tắc này nói | that is not what this principle says |
| nói về kiến thức, chứ … không nói về ký tự | talks about knowledge, and … does not talk about characters |
| trông giống hệt nhau | look exactly the same |
| chúng ta không nên gộp lại | should not be merged by us |
| một coupling giả | a false coupling |
| abstraction quá sớm | a premature abstraction |
| nhét thêm tham số và cờ rẽ nhánh | push extra parameters and branching flags |
| dần dần méo mó | gradually becomes distorted |
| câu thần chú mà chúng ta nên thuộc | the mantra that we should learn by heart |
| không bảo đảm rằng chúng sẽ cùng đổi | do not guarantee that they will change together |

**Thuật ngữ cần nhớ**

- hiểu sai → **misunderstand**
- gộp lại → **merge**
- coupling giả → **false coupling**
- quá sớm → **premature**
- cờ rẽ nhánh → **branching flag**
- méo mó, biến dạng → **distorted**

---

## Phần 4 — Một ví dụ DRY quá đà và cái giá của nó

**Tiếng Việt**

Chúng ta hãy xem một ví dụ cụ thể để thấy DRY quá đà gây hại như thế nào. Giả sử chúng ta có hai hàm kiểm tra dữ liệu, một hàm kiểm tra tuổi người dùng và một hàm kiểm tra năm sản xuất của sản phẩm, và cả hai tình cờ cùng kiểm tra rằng con số nằm trong khoảng từ không đến một trăm năm mươi. Chúng ta thấy hai hàm giống nhau nên chúng ta gộp chúng thành một hàm chung nhận vào giá trị nhỏ nhất và giá trị lớn nhất. Sáu tháng sau, sản phẩm được phép có năm sản xuất lên tới chín nghìn chín trăm chín mươi chín, còn luật tuổi thì thêm một điều kiện riêng cho người chưa thành niên. Lúc đó chúng ta phải nhét thêm tham số và biến thể vào hàm chung, và hàm chung trở nên khó đọc hơn cả hai hàm ban đầu cộng lại. Đáng lẽ chúng ta nên để hai hàm riêng, bởi vì luật tuổi và luật năm sản xuất thay đổi vì hai lý do khác nhau. KISS và YAGNI khuyên chúng ta cứ để riêng cho tới khi xuất hiện một nhu cầu gộp thật sự.

**English (bám cấu trúc tiếng Việt)**

Let us look at a concrete example to see how too much DRY does harm. Suppose we have two functions that check data, one function that checks the age of a user and one function that checks the production year of a product, and both of them happen to check that the number lies in the range from zero to one hundred and fifty. We see that the two functions look the same, so we merge them into one shared function that takes in a minimum value and a maximum value. Six months later, the product is allowed to have a production year up to nine thousand nine hundred and ninety-nine, while the age rule adds a separate condition for a minor. At that point we have to push extra parameters and variants into the shared function, and the shared function becomes harder to read than the two original functions together. We should have kept the two functions separate, because the age rule and the production year rule change for two different reasons. KISS and YAGNI advise us to keep them separate until a real need to merge appears.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| để thấy DRY quá đà gây hại như thế nào | to see how too much DRY does harm |
| giả sử chúng ta có | suppose we have |
| cả hai tình cờ cùng kiểm tra rằng | both of them happen to check that |
| nằm trong khoảng từ … đến | lies in the range from … to |
| một hàm chung nhận vào | one shared function that takes in |
| được phép có … lên tới | is allowed to have … up to |
| thêm một điều kiện riêng cho người chưa thành niên | adds a separate condition for a minor |
| khó đọc hơn cả hai hàm ban đầu cộng lại | harder to read than the two original functions together |
| đáng lẽ chúng ta nên để hai hàm riêng | we should have kept the two functions separate |
| cho tới khi xuất hiện một nhu cầu gộp thật sự | until a real need to merge appears |

**Thuật ngữ cần nhớ**

- kiểm tra dữ liệu → **validate** / **validation**
- tình cờ → **happen to** / **by coincidence**
- giá trị nhỏ nhất, lớn nhất → **minimum / maximum value**
- biến thể → **variant**
- người chưa thành niên → **a minor**

---

## Phần 5 — YAGNI chống lại việc trừu tượng hoá theo phỏng đoán

**Tiếng Việt**

YAGNI chống lại một thói quen nghe rất có trách nhiệm nhưng thật ra rất tốn kém, đó là trừu tượng hoá sẵn cho tương lai. Thói quen này có tên riêng trong danh sách code smell, và người ta gọi nó là speculative generality, tức là tổng quát hoá theo phỏng đoán. Biểu hiện quen thuộc của nó là chúng ta viết một interface chỉ có đúng một hiện thực, hoặc chúng ta thêm một lớp cấu hình cho một giá trị chưa bao giờ đổi. Nguyên tắc thay thế là chúng ta chỉ trừu tượng hoá khi chúng ta đã đau thật, nghĩa là khi đã có ít nhất hai ca dùng thật và chúng ta nhìn rõ trục thay đổi chung của chúng. Hậu quả của việc làm ngược lại rất cụ thể, vì mỗi người mới vào dự án phải đi qua ba tầng gián tiếp mới tìm ra chỗ code thật sự chạy. Chi phí đó chúng ta trả hằng ngày, trong khi lợi ích chỉ là một khả năng mà có thể không bao giờ xảy ra.

**English (bám cấu trúc tiếng Việt)**

YAGNI fights against a habit that sounds very responsible but is in fact very expensive, which is abstracting in advance for the future. This habit has its own name in the list of code smells, and people call it speculative generality, that is, generalising based on a guess. Its familiar symptom is that we write an interface that has exactly one implementation, or we add a configuration layer for a value that has never changed. The replacement principle is that we only abstract when we have already felt real pain, which means when there are at least two real use cases and we clearly see their shared axis of change. The consequence of doing the opposite is very concrete, because every new person on the project has to go through three layers of indirection before they find the place where the code really runs. We pay that cost every day, while the benefit is only a possibility that may never happen.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| nghe rất có trách nhiệm nhưng thật ra rất tốn kém | sounds very responsible but is in fact very expensive |
| trừu tượng hoá sẵn cho tương lai | abstracting in advance for the future |
| có tên riêng trong danh sách code smell | has its own name in the list of code smells |
| tổng quát hoá theo phỏng đoán | generalising based on a guess |
| biểu hiện quen thuộc của nó | its familiar symptom |
| chỉ có đúng một hiện thực | has exactly one implementation |
| khi chúng ta đã đau thật | when we have already felt real pain |
| ba tầng gián tiếp | three layers of indirection |
| chỗ code thật sự chạy | the place where the code really runs |
| chi phí đó chúng ta trả hằng ngày | we pay that cost every day |
| một khả năng mà có thể không bao giờ xảy ra | a possibility that may never happen |

**Thuật ngữ cần nhớ**

- mùi code, dấu hiệu code xấu → **code smell**
- tổng quát hoá theo phỏng đoán → **speculative generality**
- biểu hiện, triệu chứng → **symptom**
- tầng gián tiếp → **layer of indirection**
- ca sử dụng → **use case**

---

## Phần 6 — Trong TypeScript, một hàm thường thay được cả cây class

**Tiếng Việt**

Trong TypeScript và Node, nhiều pattern của GoF co lại đáng kể nhờ first-class function, closure và module. Strategy trở thành việc chúng ta truyền một hàm vào, và chúng ta không cần một cây class cho việc đó. Command trở thành một closure bắt lấy đúng context mà nó cần. Singleton trở thành một module, bởi vì Node cache module, nên thứ chúng ta export ra đã sẵn là một instance duy nhất. Decorator ở mức hành vi trở thành một hàm bọc một hàm khác, và người ta gọi thứ đó là higher-order function. Điều quan trọng ở đây là judgment, nghĩa là chúng ta chọn paradigm theo bài toán, chứ chúng ta không ép mọi thứ về OOP thuần chỉ để cho giống sách. Khi phỏng vấn, việc nói được rằng một pattern co lại thành một hàm trong ngôn ngữ này cho thấy chúng ta hiểu ý đồ của pattern, chứ chúng ta không chỉ nhớ sơ đồ class của nó.

**English (bám cấu trúc tiếng Việt)**

In TypeScript and Node, many GoF patterns shrink considerably thanks to first-class functions, closures and modules. Strategy becomes the act of passing a function in, and we do not need a class tree for that. Command becomes a closure that captures exactly the context it needs. Singleton becomes a module, because Node caches modules, so the thing we export is already a single instance. Decorator at the behaviour level becomes a function that wraps another function, and people call that thing a higher-order function. The important thing here is judgment, which means that we choose the paradigm according to the problem, and we do not force everything into pure OOP just to look like the book. In an interview, being able to say that a pattern shrinks into a function in this language shows that we understand the intent of the pattern, and we do not only remember its class diagram.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| co lại đáng kể nhờ | shrink considerably thanks to |
| trở thành việc chúng ta truyền một hàm vào | becomes the act of passing a function in |
| một cây class | a class tree |
| bắt lấy đúng context mà nó cần | captures exactly the context it needs |
| thứ chúng ta export ra đã sẵn là | the thing we export is already |
| một hàm bọc một hàm khác | a function that wraps another function |
| chọn paradigm theo bài toán | choose the paradigm according to the problem |
| chỉ để cho giống sách | just to look like the book |
| chúng ta hiểu ý đồ của pattern | we understand the intent of the pattern |
| sơ đồ class của nó | its class diagram |

**Thuật ngữ cần nhớ**

- co lại, thu gọn → **shrink**
- bao đóng (hàm giữ context) → **closure**
- hàm bậc cao → **higher-order function**
- hệ hình, trường phái lập trình → **paradigm**
- ý đồ (của pattern) → **intent**

---

## Phần 7 — Ví dụ hằng ngày: truyền hàm thay vì dựng cây class

**Tiếng Việt**

Chúng ta hãy đưa ý ở trên vào một ví dụ mà ai cũng gặp hằng ngày. Thay vì viết một class `SortStrategy` cùng ba subclass cho ba cách so sánh, chúng ta gọi `sort(items, compareFn)` và chúng ta truyền hàm so sánh vào. Cách này gọn hơn, dễ test hơn, và nó đúng với tinh thần của TypeScript. Một ví dụ khác là chúng ta thay một câu `switch` dài bằng một object map ánh xạ từ tên phương thức sang hàm xử lý, nhờ đó việc thêm một loại mới chỉ là thêm một dòng chứ không phải sửa hàm điều phối. Chính các API chuẩn của JavaScript đã làm đúng như vậy, ví dụ `Array.sort` nhận vào một hàm so sánh và `Array.map` nhận vào một hàm biến đổi. Vì vậy phong cách hàm ở đây là idiom của ngôn ngữ, chứ nó không phải là dấu hiệu của việc thiếu thiết kế.

**English (bám cấu trúc tiếng Việt)**

Let us put the idea above into an example that everybody meets every day. Instead of writing a `SortStrategy` class together with three subclasses for three ways of comparing, we call `sort(items, compareFn)` and we pass the comparison function in. This way is tidier, easier to test, and it fits the spirit of TypeScript. Another example is that we replace a long `switch` statement with an object map from the method name to the handler function, thanks to that adding a new type is only adding one line and not editing the dispatch function. The standard JavaScript APIs themselves do exactly this, for example `Array.sort` takes in a comparison function and `Array.map` takes in a transform function. Therefore the functional style here is an idiom of the language, and it is not a sign of missing design.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| một ví dụ mà ai cũng gặp hằng ngày | an example that everybody meets every day |
| cùng ba subclass cho ba cách so sánh | together with three subclasses for three ways of comparing |
| nó đúng với tinh thần của | it fits the spirit of |
| ánh xạ từ tên phương thức sang hàm xử lý | from the method name to the handler function |
| chỉ là thêm một dòng chứ không phải sửa | is only adding one line and not editing |
| hàm điều phối | the dispatch function |
| chính các API chuẩn … đã làm đúng như vậy | the standard APIs themselves do exactly this |
| một hàm biến đổi | a transform function |
| là idiom của ngôn ngữ | is an idiom of the language |
| dấu hiệu của việc thiếu thiết kế | a sign of missing design |

**Thuật ngữ cần nhớ**

- hàm so sánh → **comparison function**
- bảng ánh xạ dạng object → **object map**
- hàm xử lý → **handler**
- hàm điều phối → **dispatch function**
- lối nói quen thuộc của ngôn ngữ → **idiom**

---

## Phần 8 — Soát code AI và lúc mà lặp code lại rẻ hơn

**Tiếng Việt**

Bây giờ chúng ta hãy dùng ba nguyên tắc này làm bộ lọc để soát code do AI sinh ra. AI rất hay "DRY hoá" hai đoạn giống nhau thành một hàm được tham số hoá, và như chúng ta đã thấy ở trên, việc đó tạo ra coupling giả. AI cũng rất hay over-engineer, ví dụ nó sinh ra một class và một interface cho thứ mà chúng ta chỉ cần một hàm. Khi review, chúng ta hỏi một câu duy nhất theo tinh thần YAGNI: nhu cầu này đã có thật chưa, hay chúng ta chỉ đang phỏng đoán? Nếu nhu cầu chưa có thật, chúng ta xoá lớp trừu tượng đó đi và chúng ta giữ giải pháp đơn giản. Việc xoá code thừa hôm nay rẻ hơn nhiều so với việc gỡ một abstraction sai sau hai năm.

Chúng ta cũng nên nhớ một kết luận nghe có vẻ ngược đời nhưng rất thực tế, đó là đôi khi lặp code còn rẻ hơn một abstraction sai. Lý do là code lặp thì dễ nhìn thấy và dễ sửa, trong khi một abstraction sai thì giấu vấn đề đi và bắt mọi người phải đi đường vòng. Khi chúng ta gặp một hàm chung đầy cờ rẽ nhánh cho hai ca khác nhau, cách chữa đúng là tách nó trở lại thành hai hàm độc lập. Sau khi tách xong, tiêu chí để biết chúng ta đã làm đúng là mỗi nhánh sống riêng và không còn cờ chung nào nữa. Đây cũng là câu trả lời mạnh nhất cho câu hỏi bẫy "thấy hai đoạn code y hệt thì gộp ngay chứ", bởi vì câu trả lời đúng là chúng ta phải hỏi hai đoạn đó có cùng lý do thay đổi hay không.

**English (bám cấu trúc tiếng Việt)**

Now let us use these three principles as a filter for reviewing code that AI generates. AI very often "DRYs up" two similar blocks into one parameterised function, and as we saw above, that creates a false coupling. AI also very often over-engineers, for example it produces a class and an interface for something where we only need a function. When we review, we ask a single question in the spirit of YAGNI: is this need already real, or are we only guessing? If the need is not real yet, we delete that abstraction layer and we keep the simple solution. Deleting extra code today is much cheaper than unwinding a wrong abstraction after two years.

We should also remember a conclusion that sounds backwards but is very practical, which is that duplication is sometimes cheaper than a wrong abstraction. The reason is that duplicated code is easy to see and easy to fix, while a wrong abstraction hides the problem and forces everybody to take a detour. When we meet a shared function full of branching flags for two different cases, the correct cure is to split it back into two independent functions. After the split is done, the criterion for knowing that we did it right is that each branch lives on its own and there is no shared flag left. This is also the strongest answer to the trap question "if you see two identical blocks, you merge them straight away, right?", because the correct answer is that we have to ask whether those two blocks change for the same reason or not.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| làm bộ lọc để soát code do AI sinh ra | as a filter for reviewing code that AI generates |
| một hàm được tham số hoá | one parameterised function |
| hỏi một câu duy nhất theo tinh thần YAGNI | ask a single question in the spirit of YAGNI |
| hay chúng ta chỉ đang phỏng đoán? | or are we only guessing? |
| gỡ một abstraction sai sau hai năm | unwinding a wrong abstraction after two years |
| nghe có vẻ ngược đời nhưng rất thực tế | sounds backwards but is very practical |
| giấu vấn đề đi | hides the problem |
| bắt mọi người phải đi đường vòng | forces everybody to take a detour |
| đầy cờ rẽ nhánh cho hai ca khác nhau | full of branching flags for two different cases |
| tiêu chí để biết chúng ta đã làm đúng | the criterion for knowing that we did it right |
| mỗi nhánh sống riêng | each branch lives on its own |
| gộp ngay chứ? | you merge them straight away, right? |

**Thuật ngữ cần nhớ**

- tham số hoá → **parameterise**
- lặp code → **duplication**
- gỡ ra, tháo ra → **unwind**
- đường vòng → **a detour**
- tiêu chí → **criterion** (số nhiều: **criteria**)

---

## Mô hình ghi nhớ

**Tiếng Việt**

DRY nói về kiến thức, chứ DRY không nói về ký tự, vì vậy chúng ta chỉ gộp hai thứ khi chúng thay đổi vì cùng một lý do. Chúng ta làm đơn giản trước theo KISS, chúng ta chỉ làm khi thật sự cần theo YAGNI, và trong TypeScript thì một hàm thường thay được cả một cây class.

**English (bám cấu trúc tiếng Việt)**

DRY talks about knowledge, and DRY does not talk about characters, therefore we only merge two things when they change for the same reason. We keep it simple first following KISS, we only build when it is really needed following YAGNI, and in TypeScript one function can often replace a whole class tree.

---

## Bảng thuật ngữ tổng hợp

| Tiếng Việt | Tiếng Anh | Ghi chú phát âm |
|---|---|---|
| làm quá mức cần thiết | over-engineering | |
| khả năng phán đoán | judgment | /ˈdʒʌdʒmənt/ — "JUJ-mợnt", chữ **d** giữa không đọc rời |
| thiệt hại | damage | trọng âm đầu: DA-mage, đuôi đọc "-mịj" |
| định kiến | prejudice | /ˈpredʒudɪs/ — PRE-ju-dice, đuôi đọc "-đis" chứ không phải "-đais" |
| hàm là công dân hạng nhất | first-class function | |
| tên viết tắt | acronym | /ˈækrənɪm/ — A-cro-nym, trọng âm đầu |
| nguồn chân lý duy nhất | single source of truth | *truth* có âm **th** cuối, đặt lưỡi giữa hai hàm răng |
| đủ dùng | good enough | |
| lãng phí | waste | /weɪst/ — đọc giống "waist", đuôi **-st** phải bật |
| nhân danh | in the name of | |
| hiểu sai | misunderstand | trọng âm cuối: mis-un-der-STAND |
| gộp lại | merge | /mɜːdʒ/ — "mớjơ", không đọc thành "mơ-gờ" |
| coupling giả | false coupling | |
| quá sớm | premature | Anh đọc /ˌpreməˈtjʊə/ — pre-ma-TURE, trọng âm cuối |
| cờ rẽ nhánh | branching flag | |
| méo mó, biến dạng | distorted | trọng âm âm hai: dis-TOR-ted |
| kiểm tra dữ liệu | validate / validation | động từ *validate* trọng âm đầu: VA-li-date |
| tình cờ | happen to / by coincidence | *coincidence* trọng âm âm hai: co-IN-ci-dence |
| biến thể | variant | /ˈveəriənt/ — VA-ri-ant, trọng âm đầu |
| người chưa thành niên | a minor | /ˈmaɪnə/ — "MAI-nơ" |
| mùi code, dấu hiệu code xấu | code smell | |
| tổng quát hoá theo phỏng đoán | speculative generality | *speculative* /ˈspekjələtɪv/ — SPEC-u-la-tive, trọng âm đầu |
| biểu hiện, triệu chứng | symptom | /ˈsɪmptəm/ — SIMP-tom, chữ **p** có đọc |
| tầng gián tiếp | layer of indirection | *indirection* trọng âm áp chót: in-di-REC-tion |
| ca sử dụng | use case | *use* ở đây là danh từ, đọc /juːs/ với đuôi **-s**, không đọc /juːz/ |
| co lại, thu gọn | shrink | cụm **shr-** khó, tập đọc chậm: "shrink" |
| bao đóng | closure | /ˈkləʊʒə/ — CLO-zhơ, âm giữa là "zh" như trong *measure* |
| hàm bậc cao | higher-order function | |
| hệ hình lập trình | paradigm | /ˈpærədaɪm/ — PA-ra-dime, chữ **g** câm |
| ý đồ | intent | trọng âm cuối: in-TENT |
| sơ đồ class | class diagram | *diagram* trọng âm đầu: DI-a-gram |
| thể hiện duy nhất | instance | trọng âm đầu: IN-stance |
| mô-đun | module | Anh đọc /ˈmɒdjuːl/ — "MOD-yule" |
| hàm so sánh | comparison function | *comparison* trọng âm âm hai: com-PA-ri-son |
| bảng ánh xạ dạng object | object map | |
| hàm xử lý | handler | chữ **h** phải bật hơi: "HAND-lơ" |
| hàm điều phối | dispatch function | *dispatch* trọng âm cuối: dis-PATCH |
| lối nói quen thuộc của ngôn ngữ | idiom | /ˈɪdiəm/ — I-di-om, trọng âm đầu |
| tham số hoá | parameterise | *parameter* trọng âm âm hai: pa-RA-me-ter |
| lặp code | duplication | trọng âm áp chót: du-pli-CA-tion |
| gỡ ra, tháo ra | unwind | /ʌnˈwaɪnd/ — un-WIND, vần với "find" |
| đường vòng | a detour | Anh đọc /ˈdiːtʊə/ — DEE-tour, trọng âm đầu |
| tiêu chí | criterion / criteria | số ít cri-TE-ri-on, số nhiều cri-TE-ri-a — đừng dùng lẫn |

---

## Bài nói

> **Cách làm:** ghi âm lại giọng của mình, mỗi câu nói liên tục **60–90 giây**, **không nhìn tài liệu** khi nói. Mỗi bài nói phải dùng ít nhất **6 thuật ngữ** trong bảng ở trên. Nghe lại bản ghi âm một lần, ghi ra hai chỗ mình bị vấp, rồi nói lại lần hai.

1. Explain to a junior developer what DRY really means, and why "no two identical blocks in the code" is the wrong definition.
2. A colleague sees two functions that look identical and wants to merge them into one shared function right away. Explain why you would push back, and what question you would ask first.
3. Someone on your team argues that duplication is always a defect and must always be removed. Explain when you would accept duplication on purpose, and what makes a wrong abstraction more expensive than the duplication it replaced.
4. Describe what happens over six months to a shared `validateRange` function after two different business rules start to diverge.
5. Explain how Strategy, Command and Singleton shrink in TypeScript, and say what this tells an interviewer about how you understand patterns.
6. When would you choose a class hierarchy over passing a function, and when would you choose the opposite? Give the trade-off in both directions.
7. You are reviewing a pull request written by an AI tool that introduces an interface with exactly one implementation. Describe out loud the YAGNI question you would ask and what you would do with the answer.
