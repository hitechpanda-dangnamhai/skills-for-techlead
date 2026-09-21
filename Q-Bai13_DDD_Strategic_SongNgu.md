# Bài 13 — DDD (1): Strategic — Bounded Context · Ubiquitous Language · Entity vs Value Object
### Bản song ngữ: tiếng Anh bám cấu trúc tiếng Việt

> **Cách dùng bài này:** đọc đoạn tiếng Việt trước, tự dịch trong đầu, rồi mới nhìn sang đoạn tiếng Anh để so. Sau khi so xong, **đọc to đoạn tiếng Anh hai lần** — lần một để nhìn chữ, lần hai để nghe chính mình. Phần bài nói ở cuối là bắt buộc; không nói ra thì bài này chỉ là bài đọc.

---

## Phần 1 — Phần strategic của DDD luôn có ích, kể cả khi không làm DDD đầy đủ

**Tiếng Việt**

DDD do Eric Evans đưa ra năm 2003, và nó là cách chúng ta mô hình hoá một nghiệp vụ phức tạp bên trong khung kiến trúc của Bài 11 và Bài 12. Bài này lo phần strategic, tức là Bounded Context và Ubiquitous Language. Phần strategic có một điểm rất đáng giá, đó là nó luôn hữu ích kể cả khi chúng ta không làm DDD một cách đầy đủ. Bài này cũng lo hai viên gạch nền là Entity và Value Object, bởi vì mọi thứ ở Bài 14 đều được dựng trên hai khái niệm đó. Nếu chúng ta chỉ có thời gian học một nửa của DDD, chúng ta nên học đúng nửa nằm trong bài này.

**English (bám cấu trúc tiếng Việt)**

DDD was put forward by Eric Evans in 2003, and it is the way we model a complex business inside the architectural frame of Lesson 11 and Lesson 12. This lesson takes care of the strategic part, that is, the Bounded Context and the Ubiquitous Language. The strategic part has one very valuable point, which is that it is always useful even when we do not do DDD in full. This lesson also takes care of two foundation bricks, which are the Entity and the Value Object, because everything in Lesson 14 is built on those two concepts. If we only have time to learn half of DDD, we should learn exactly the half that sits in this lesson.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| do Eric Evans đưa ra năm 2003 | was put forward by Eric Evans in 2003 |
| bên trong khung kiến trúc của | inside the architectural frame of |
| có một điểm rất đáng giá | has one very valuable point |
| kể cả khi chúng ta không làm DDD một cách đầy đủ | even when we do not do DDD in full |
| hai viên gạch nền | two foundation bricks |
| đều được dựng trên hai khái niệm đó | is built on those two concepts |
| chúng ta nên học đúng nửa nằm trong bài này | we should learn exactly the half that sits in this lesson |

**Thuật ngữ cần nhớ**

- mô hình hoá → **model** (động từ)
- chiến lược → **strategic**
- chiến thuật → **tactical**
- viên gạch nền → **foundation brick** / **building block**
- một cách đầy đủ → **in full**

---

## Phần 2 — Bounded Context là một thế giới có ngôn ngữ riêng

**Tiếng Việt**

Bounded Context là một ranh giới, và bên trong ranh giới đó thì một model cùng một ngôn ngữ được giữ nhất quán. Nói cách khác, mỗi context là một thế giới riêng và nó có từ vựng riêng của nó. Trong một hệ thống thương mại điện tử, chúng ta thường tách các context như danh mục sản phẩm, đặt hàng, thanh toán và giao vận. Mỗi context có model riêng cho những khái niệm nghe rất giống nhau, ví dụ mỗi context có một khái niệm đơn hàng riêng của nó. Các context giao tiếp với nhau qua định danh hoặc qua sự kiện, chứ chúng không dùng chung một bảng dữ liệu. Việc vạch ranh giới này là một quyết định chiến lược, cho nên chúng ta phải làm nó trước khi chúng ta viết code.

**English (bám cấu trúc tiếng Việt)**

A Bounded Context is a boundary, and inside that boundary one model together with one language is kept consistent. In other words, each context is its own world and it has its own vocabulary. In an e-commerce system, we usually split contexts such as the product catalogue, ordering, billing and shipping. Each context has its own model for concepts that sound very similar, for example each context has its own concept of an order. The contexts communicate with each other through identifiers or through events, and they do not share one data table. Drawing these boundaries is a strategic decision, so we have to do it before we write the code.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| bên trong ranh giới đó | inside that boundary |
| được giữ nhất quán | is kept consistent |
| mỗi context là một thế giới riêng | each context is its own world |
| nó có từ vựng riêng của nó | it has its own vocabulary |
| những khái niệm nghe rất giống nhau | concepts that sound very similar |
| giao tiếp với nhau qua định danh hoặc qua sự kiện | communicate with each other through identifiers or through events |
| chúng không dùng chung một bảng dữ liệu | they do not share one data table |
| việc vạch ranh giới này là một quyết định chiến lược | drawing these boundaries is a strategic decision |

**Thuật ngữ cần nhớ**

- ranh giới ngữ cảnh → **bounded context**
- nhất quán → **consistent**
- danh mục sản phẩm → **product catalogue**
- giao vận → **shipping**
- định danh → **identifier**

---

## Phần 3 — Vì sao "một model duy nhất cho cả hệ thống" là kỳ vọng sai

**Tiếng Việt**

Có một kỳ vọng rất phổ biến nhưng cũng rất sai, đó là kỳ vọng rằng cả hệ thống dùng chung đúng một model. Ví dụ rõ nhất là từ "Customer", bởi vì từ này mang nghĩa khác nhau ở mỗi context. Trong context bán hàng, một khách hàng là một đầu mối và là người đứng tên đơn hàng. Trong context thanh toán, cùng từ đó lại là một tài khoản có hạn mức và có lịch sử hoá đơn. Trong context hỗ trợ, khách hàng lại là người sở hữu một ticket và có lịch sử trao đổi. Nếu chúng ta ép cả ba nghĩa vào một model toàn cục, model đó sẽ phình ra và sẽ chứa những trường mâu thuẫn nhau, còn nếu chúng ta chia context thì mỗi model gọn lại và độ phức tạp giảm hẳn.

**English (bám cấu trúc tiếng Việt)**

There is an expectation that is very common but also very wrong, which is the expectation that the whole system shares exactly one model. The clearest example is the word "Customer", because this word carries a different meaning in each context. In the sales context, a customer is a lead and is the person whose name is on the order. In the billing context, the same word is instead an account that has a credit limit and an invoice history. In the support context, the customer is instead the owner of a ticket and has a conversation history. If we force all three meanings into one global model, that model will swell and will hold fields that contradict each other, while if we split the contexts then each model becomes smaller and the complexity drops sharply.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| một kỳ vọng rất phổ biến nhưng cũng rất sai | an expectation that is very common but also very wrong |
| cả hệ thống dùng chung đúng một model | the whole system shares exactly one model |
| mang nghĩa khác nhau ở mỗi context | carries a different meaning in each context |
| là người đứng tên đơn hàng | is the person whose name is on the order |
| một tài khoản có hạn mức | an account that has a credit limit |
| là người sở hữu một ticket | is the owner of a ticket |
| nếu chúng ta ép cả ba nghĩa vào một model toàn cục | if we force all three meanings into one global model |
| sẽ chứa những trường mâu thuẫn nhau | will hold fields that contradict each other |
| độ phức tạp giảm hẳn | the complexity drops sharply |

**Thuật ngữ cần nhớ**

- kỳ vọng → **expectation**
- đầu mối bán hàng → **lead**
- hạn mức tín dụng → **credit limit**
- hoá đơn → **invoice**
- mâu thuẫn → **contradict**

---

## Phần 4 — Ubiquitous Language và hệ quả lên cách đặt tên

**Tiếng Việt**

Ubiquitous Language là ngôn ngữ chung được dùng nhất quán cả trong lúc trò chuyện lẫn ở bên trong code. Đây là ngôn ngữ chung giữa lập trình viên và chuyên gia nghiệp vụ. Vấn đề mà nó giải quyết là việc dịch sai giữa phía nghiệp vụ và phía kỹ thuật, bởi vì mỗi lần dịch là một lần thông tin bị méo đi. Rất nhiều lỗi nghiệp vụ đắt tiền bắt nguồn từ chỗ hai bên dùng cùng một từ nhưng họ hiểu theo hai nghĩa. Ngôn ngữ này gắn với từng bounded context, cho nên cùng một từ vẫn mang nghĩa riêng trong mỗi thế giới.

Hệ quả trực tiếp của nguyên tắc này là tên class và tên method phải phản ánh đúng thuật ngữ của domain. Ví dụ chúng ta viết `Order.place`, chứ chúng ta không viết `OrderManager.processData`. Những cái tên kỹ thuật mơ hồ như `DataManager` hoặc `Helper` là dấu hiệu cho thấy chúng ta đã đánh mất ngôn ngữ chung. Khi tên trong code khớp với tên mà chuyên gia nghiệp vụ dùng, chúng ta đọc code cho họ nghe được và họ chỉ ra được chỗ sai. Đó là lợi ích thực tế nhất của Ubiquitous Language, và nó không đòi hỏi chúng ta phải áp dụng toàn bộ DDD.

**English (bám cấu trúc tiếng Việt)**

The Ubiquitous Language is the shared language used consistently both during conversation and inside the code. This is the shared language between developers and domain experts. The problem it solves is the mistranslation between the business side and the technical side, because every translation is one more chance for the information to be distorted. Very many expensive business bugs start from the point where the two sides use the same word but they understand it in two meanings. This language is tied to each bounded context, so the same word still carries its own meaning inside each world.

The direct consequence of this principle is that class names and method names must reflect the terminology of the domain exactly. For example we write `Order.place`, and we do not write `OrderManager.processData`. Vague technical names such as `DataManager` or `Helper` are a sign showing that we have lost the shared language. When the names in the code match the names the domain expert uses, we can read the code out to them and they can point out what is wrong. That is the most practical benefit of the Ubiquitous Language, and it does not require us to adopt the whole of DDD.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| được dùng nhất quán cả trong lúc trò chuyện lẫn ở bên trong code | used consistently both during conversation and inside the code |
| việc dịch sai giữa phía nghiệp vụ và phía kỹ thuật | the mistranslation between the business side and the technical side |
| mỗi lần dịch là một lần thông tin bị méo đi | every translation is one more chance for the information to be distorted |
| bắt nguồn từ chỗ | start from the point where |
| họ hiểu theo hai nghĩa | they understand it in two meanings |
| phải phản ánh đúng thuật ngữ của domain | must reflect the terminology of the domain exactly |
| những cái tên kỹ thuật mơ hồ | vague technical names |
| chúng ta đã đánh mất ngôn ngữ chung | we have lost the shared language |
| chúng ta đọc code cho họ nghe được | we can read the code out to them |
| nó không đòi hỏi chúng ta phải áp dụng toàn bộ DDD | it does not require us to adopt the whole of DDD |

**Thuật ngữ cần nhớ**

- ngôn ngữ chung khắp nơi → **ubiquitous language**
- chuyên gia nghiệp vụ → **domain expert**
- dịch sai → **mistranslation**
- méo mó, sai lệch → **distorted**
- mơ hồ → **vague**

---

## Phần 5 — Entity trả lời câu hỏi "ai"

**Tiếng Việt**

Entity là loại object có một định danh riêng, và nó tồn tại xuyên suốt thời gian dù các thuộc tính của nó thay đổi. Ví dụ một người dùng mang số bốn mươi hai vẫn là chính người đó sau khi anh ta đổi tên và đổi email. Vì vậy chúng ta so sánh hai entity bằng định danh, chứ chúng ta không so sánh chúng bằng nội dung của các trường. Cách nghĩ ngắn gọn là entity trả lời câu hỏi "ai", bởi vì nó là một cá thể có lịch sử riêng. Trong code, điều này có nghĩa là method so sánh của entity chỉ nhìn vào trường định danh.

**English (bám cấu trúc tiếng Việt)**

An Entity is the kind of object that has its own identity, and it exists through time even though its attributes change. For example a user carrying the number forty-two is still that very person after he changes his name and changes his email. Therefore we compare two entities by identity, and we do not compare them by the content of the fields. The short way to think is that an entity answers the question "who", because it is an individual with its own history. In code, this means that the comparison method of an entity only looks at the identity field.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| có một định danh riêng | has its own identity |
| tồn tại xuyên suốt thời gian | exists through time |
| dù các thuộc tính của nó thay đổi | even though its attributes change |
| vẫn là chính người đó | is still that very person |
| chúng ta so sánh hai entity bằng định danh | we compare two entities by identity |
| bằng nội dung của các trường | by the content of the fields |
| nó là một cá thể có lịch sử riêng | it is an individual with its own history |
| chỉ nhìn vào trường định danh | only looks at the identity field |

**Thuật ngữ cần nhớ**

- thực thể → **entity**
- danh tính, định danh → **identity**
- thuộc tính → **attribute**
- cá thể → **individual**
- so sánh → **compare**

---

## Phần 6 — Value Object trả lời câu hỏi "cái gì" và "bao nhiêu"

**Tiếng Việt**

Value Object thì được định danh bằng chính giá trị của nó, và nó bất biến. Khi chúng ta muốn thay đổi một Value Object, chúng ta không sửa nó mà chúng ta tạo ra một cái mới. Ví dụ quen thuộc là một số tiền gồm một trăm và đơn vị đô la, một địa chỉ, hoặc một khoảng thời gian. Hai Value Object bằng nhau khi mọi giá trị bên trong chúng bằng nhau, cho nên hai số tiền cùng là một trăm đô la thì bằng nhau. Tiêu chí phân biệt rất gọn, đó là chúng ta hỏi thứ này có cần định danh riêng hay không, và nếu cần thì nó là Entity còn nếu không cần thì nó là Value Object. Cách nghĩ ngắn gọn là Value Object trả lời câu hỏi "cái gì" hoặc "bao nhiêu", chứ nó không trả lời câu hỏi "ai".

**English (bám cấu trúc tiếng Việt)**

A Value Object is identified by its own value, and it is immutable. When we want to change a Value Object, we do not edit it but we create a new one. Familiar examples are an amount of money made of one hundred and the dollar currency, an address, or a date range. Two Value Objects are equal when every value inside them is equal, so two amounts that are both one hundred dollars are equal. The criterion for telling them apart is very short, which is that we ask whether this thing needs its own identity or not, and if it needs one then it is an Entity while if it does not need one then it is a Value Object. The short way to think is that a Value Object answers the question "what" or "how much", and it does not answer the question "who".

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| được định danh bằng chính giá trị của nó | is identified by its own value |
| chúng ta không sửa nó mà chúng ta tạo ra một cái mới | we do not edit it but we create a new one |
| một số tiền gồm một trăm và đơn vị đô la | an amount of money made of one hundred and the dollar currency |
| một khoảng thời gian | a date range |
| khi mọi giá trị bên trong chúng bằng nhau | when every value inside them is equal |
| tiêu chí phân biệt rất gọn | the criterion for telling them apart is very short |
| có cần định danh riêng hay không | whether it needs its own identity or not |
| nó không trả lời câu hỏi "ai" | it does not answer the question "who" |

**Thuật ngữ cần nhớ**

- đối tượng giá trị → **value object**
- bất biến → **immutable**
- khoảng thời gian → **date range**
- bằng nhau → **equal**
- tiêu chí → **criterion**

---

## Phần 7 — "Cứ để tiền là một con số cho gọn" là một cái bẫy

**Tiếng Việt**

Có một câu hỏi thách đố rất hay gặp, đó là câu "cứ để tiền là một con số cho gọn, đúng không". Câu trả lời của chúng ta là không, và lý do nằm ngay ở Bài 10. Khi tiền chỉ là một con số, chúng ta rơi vào primitive obsession, và chúng ta cộng nhầm hai số tiền khác đơn vị mà không có gì cản chúng ta lại. Phần kiểm tra hợp lệ cũng bị rải khắp nơi, bởi vì mỗi chỗ dùng lại phải tự kiểm tra rằng số tiền không âm. Khi chúng ta gói nó thành một Value Object tên là `Money`, mọi invariant nằm gọn tại một chỗ, ví dụ phép cộng sẽ ném lỗi nếu hai đơn vị tiền tệ khác nhau. Một ví dụ khác cùng loại là địa chỉ email, bởi vì gói nó thành Value Object cho phép chúng ta kiểm tra định dạng đúng một lần duy nhất.

**English (bám cấu trúc tiếng Việt)**

There is a challenge question that we meet very often, which is "let us just keep money as a plain number for simplicity, right". Our answer is no, and the reason sits right in Lesson 10. When money is only a number, we fall into primitive obsession, and we wrongly add two amounts in different currencies with nothing stopping us. The validation also gets scattered everywhere, because every place that uses it has to check by itself that the amount is not negative. When we wrap it into a Value Object named `Money`, every invariant sits neatly in one place, for example the addition will throw an error if the two currencies are different. Another example of the same kind is the email address, because wrapping it into a Value Object lets us check the format exactly once.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| cứ để tiền là một con số cho gọn, đúng không | let us just keep money as a plain number for simplicity, right |
| lý do nằm ngay ở Bài 10 | the reason sits right in Lesson 10 |
| chúng ta cộng nhầm hai số tiền khác đơn vị | we wrongly add two amounts in different currencies |
| mà không có gì cản chúng ta lại | with nothing stopping us |
| mỗi chỗ dùng lại phải tự kiểm tra | every place that uses it has to check by itself |
| mọi invariant nằm gọn tại một chỗ | every invariant sits neatly in one place |
| phép cộng sẽ ném lỗi | the addition will throw an error |
| kiểm tra định dạng đúng một lần duy nhất | check the format exactly once |

**Thuật ngữ cần nhớ**

- ám ảnh kiểu nguyên thuỷ → **primitive obsession**
- đơn vị tiền tệ → **currency**
- lệch, không khớp → **mismatch**
- số âm → **negative**
- định dạng → **format**

---

## Phần 8 — Context map, ranh giới service, và soát code AI

**Tiếng Việt**

Ở cấp hệ thống, ranh giới của context thường được ánh xạ sang ranh giới của service. Nhiều đội đi theo quy tắc rằng một microservice tương ứng với một bounded context, và đó là một quy tắc rất hữu ích. Công cụ để vạch ranh giới trước khi chia service là context map, tức là một sơ đồ mô tả các context cùng quan hệ giữa chúng. Chúng ta nên nhấn mạnh rằng phần strategic này hữu ích kể cả khi chúng ta không làm phần tactical của DDD. Nói cách khác, chúng ta vẫn hưởng lợi từ ranh giới rõ ràng và từ ngôn ngữ chung dù chúng ta không dựng aggregate hay domain event.

Với code do AI sinh ra, bài này có ba thứ cần soát. Thứ nhất là AI hay gộp mọi thứ vào một model toàn cục, bởi vì nó không biết ranh giới nghiệp vụ của công ty chúng ta. Thứ hai là AI hay dùng kiểu nguyên thuỷ ở chỗ lẽ ra nên có Value Object, ví dụ nó để email và tiền là chuỗi và số. Thứ ba là AI hay đặt những cái tên kỹ thuật mơ hồ, và những cái tên đó phá vỡ ngôn ngữ chung. Khi review, chúng ta hỏi hai câu, đó là tên trong code có khớp với từ mà chuyên gia nghiệp vụ dùng hay không, và có cụm dữ liệu nào nên được gói thành Value Object hay không.

**English (bám cấu trúc tiếng Việt)**

At the system level, the boundaries of contexts are usually mapped onto the boundaries of services. Many teams follow the rule that one microservice corresponds to one bounded context, and that is a very useful rule. The tool for drawing the boundaries before splitting services is the context map, that is, a diagram describing the contexts together with the relationships between them. We should stress that this strategic part is useful even when we do not do the tactical part of DDD. In other words, we still benefit from clear boundaries and from a shared language although we do not build aggregates or domain events.

For code that AI generates, this lesson has three things that need review. The first is that AI often merges everything into one global model, because it does not know the business boundaries of our company. The second is that AI often uses primitive types where there should have been a Value Object, for example it leaves the email and the money as a string and a number. The third is that AI often gives vague technical names, and those names break the shared language. When we review, we ask two questions, which are whether the names in the code match the words the domain expert uses, and whether there is any group of data that should be wrapped into a Value Object.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| thường được ánh xạ sang ranh giới của service | are usually mapped onto the boundaries of services |
| một microservice tương ứng với một bounded context | one microservice corresponds to one bounded context |
| một sơ đồ mô tả các context cùng quan hệ giữa chúng | a diagram describing the contexts together with the relationships between them |
| chúng ta nên nhấn mạnh rằng | we should stress that |
| chúng ta vẫn hưởng lợi từ ranh giới rõ ràng | we still benefit from clear boundaries |
| nó không biết ranh giới nghiệp vụ của công ty chúng ta | it does not know the business boundaries of our company |
| ở chỗ lẽ ra nên có Value Object | where there should have been a Value Object |
| những cái tên đó phá vỡ ngôn ngữ chung | those names break the shared language |
| có cụm dữ liệu nào nên được gói thành Value Object hay không | whether there is any group of data that should be wrapped into a Value Object |

**Thuật ngữ cần nhớ**

- bản đồ ngữ cảnh → **context map**
- tương ứng với → **correspond to**
- nhấn mạnh → **stress** / **emphasise**
- hưởng lợi từ → **benefit from**
- gói lại → **wrap**

---

## Mô hình ghi nhớ

**Tiếng Việt**

Mỗi Bounded Context là một thế giới có ngôn ngữ riêng của nó, cho nên cùng một từ vẫn mang hai nghĩa khác nhau ở hai thế giới. Entity trả lời câu hỏi "ai" vì nó có định danh và sống qua thời gian, còn Value Object trả lời câu hỏi "cái gì" vì nó bất biến và bằng nhau theo giá trị.

**English (bám cấu trúc tiếng Việt)**

Each Bounded Context is a world with its own language, therefore the same word still carries two different meanings in two worlds. The Entity answers the question "who" because it has an identity and lives through time, while the Value Object answers the question "what" because it is immutable and is equal by value.

---

## Bảng thuật ngữ tổng hợp

| Tiếng Việt | Tiếng Anh | Ghi chú phát âm |
|---|---|---|
| mô hình hoá | model (động từ) | |
| chiến lược | strategic | trọng âm âm hai: stra-TE-gic |
| chiến thuật | tactical | trọng âm đầu: TAC-ti-cal |
| viên gạch nền | foundation brick / building block | *foundation* trọng âm áp chót: foun-DA-tion |
| một cách đầy đủ | in full | |
| ranh giới ngữ cảnh | bounded context | *bounded* hai âm tiết, đuôi **-ded** đọc rõ |
| nhất quán | consistent | trọng âm âm hai: con-SIS-tent |
| danh mục sản phẩm | product catalogue | *catalogue* trọng âm đầu: CA-ta-logue; chính tả Anh có đuôi **-ue** |
| giao vận | shipping | |
| định danh | identifier | trọng âm âm hai: i-DEN-ti-fi-er |
| kỳ vọng | expectation | trọng âm áp chót: ex-pec-TA-tion |
| đầu mối bán hàng | lead | ở nghĩa này đọc /liːd/ — "liid", không đọc /led/ |
| hạn mức tín dụng | credit limit | *credit* trọng âm đầu: CRE-dit |
| hoá đơn | invoice | trọng âm đầu: IN-voice |
| mâu thuẫn | contradict | trọng âm cuối: con-tra-DICT |
| ngôn ngữ chung khắp nơi | ubiquitous language | /juːˈbɪkwɪtəs/ — "yu-BI-kwi-tợs", trọng âm âm hai |
| chuyên gia nghiệp vụ | domain expert | *expert* trọng âm đầu: EX-pert |
| dịch sai | mistranslation | |
| méo mó, sai lệch | distorted | trọng âm âm hai: dis-TOR-ted |
| mơ hồ | vague | /veɪg/ — đọc là "veig", chữ **u** câm |
| thực thể | entity | trọng âm đầu: EN-ti-ty |
| danh tính, định danh | identity | trọng âm âm hai: i-DEN-ti-ty |
| thuộc tính | attribute | danh từ trọng âm đầu: A-ttri-bute |
| cá thể | individual | trọng âm âm ba: in-di-VI-du-al |
| so sánh | compare | trọng âm cuối: com-PARE |
| đối tượng giá trị | value object | |
| bất biến | immutable | trọng âm âm hai: im-MU-ta-ble |
| khoảng thời gian | date range | |
| bằng nhau | equal | trọng âm đầu: E-qual |
| tiêu chí | criterion | số ít cri-TE-ri-on; số nhiều là *criteria* |
| ám ảnh kiểu nguyên thuỷ | primitive obsession | *primitive* trọng âm đầu: PRI-mi-tive |
| đơn vị tiền tệ | currency | trọng âm đầu: CU-rren-cy |
| lệch, không khớp | mismatch | |
| số âm | negative | trọng âm đầu: NE-ga-tive |
| định dạng | format | |
| bản đồ ngữ cảnh | context map | |
| tương ứng với | correspond to | trọng âm cuối: co-rres-POND |
| nhấn mạnh | stress / emphasise | *emphasise* trọng âm đầu: EM-pha-sise |
| hưởng lợi từ | benefit from | |
| gói lại | wrap | /ræp/ — chữ **w** câm |

---

## Bài nói

> **Cách làm:** ghi âm lại giọng của mình, mỗi câu nói liên tục **60–90 giây**, **không nhìn tài liệu** khi nói. Mỗi bài nói phải dùng ít nhất **6 thuật ngữ** trong bảng ở trên. Nghe lại bản ghi âm một lần, ghi ra hai chỗ mình bị vấp, rồi nói lại lần hai.

1. Explain what a Bounded Context is, and use the word "Customer" to show why one global model across the whole system does not work.
2. Explain to a junior developer what the Ubiquitous Language is and how it changes the names they choose for classes and methods.
3. Explain the difference between an Entity and a Value Object, and give the single question you ask to decide which one you are looking at.
4. A colleague says: "Money is just a number with a currency string next to it, so a Value Object is overkill." Explain why you would push back and what bugs the Value Object prevents.
5. Describe how you would draw two or three bounded contexts for an e-commerce system, and name one word that means different things across them.
6. When is strategic DDD worth doing even if the team never adopts aggregates or domain events? Give the trade-off in both directions.
7. You are reviewing AI-generated code where every service shares one large `Customer` model and every email is a plain string. Describe out loud what you would ask about and what you would change first.
