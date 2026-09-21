# Bài 10 — Code Smell · Anti-pattern · Refactoring
### Bản song ngữ: tiếng Anh bám cấu trúc tiếng Việt

> **Cách dùng bài này:** đọc đoạn tiếng Việt trước, tự dịch trong đầu, rồi mới nhìn sang đoạn tiếng Anh để so. Sau khi so xong, **đọc to đoạn tiếng Anh hai lần** — lần một để nhìn chữ, lần hai để nghe chính mình. Phần bài nói ở cuối là bắt buộc; không nói ra thì bài này chỉ là bài đọc.

---

## Phần 1 — Bài này là mặt sau của tất cả các bài trước

**Tiếng Việt**

Bài này là mặt sau của toàn bộ các bài từ Bài 1 đến Bài 9. Khi chúng ta vi phạm coupling, cohesion hoặc SOLID, chúng ta sẽ ngửi thấy mùi, và những cái mùi đó đều có tên gọi riêng. Bài này dạy chúng ta hai việc, việc thứ nhất là đặt tên đúng cho vấn đề để cả đội có chung một bộ từ vựng khi review, và việc thứ hai là refactor một cách an toàn, nghĩa là đổi cấu trúc mà vẫn giữ nguyên hành vi. Đây là cầu nối sang phần kiến trúc ở Bài 11 và Bài 12, và nó cũng là cầu nối sang phần judgment của Tech Lead ở Bài 15. Với một kỹ sư senior, khả năng gọi đúng tên một vấn đề trong code review có giá trị hơn khả năng tự tay viết lại đoạn code đó.

**English (bám cấu trúc tiếng Việt)**

This lesson is the other side of all the lessons from Lesson 1 to Lesson 9. When we violate coupling, cohesion or SOLID, we will smell something, and those smells all have their own names. This lesson teaches us two things, the first thing is naming the problem correctly so that the whole team shares one vocabulary during review, and the second thing is refactoring safely, which means changing the structure while still keeping the behaviour unchanged. This is the bridge to the architecture part in Lesson 11 and Lesson 12, and it is also the bridge to the Tech Lead judgment part in Lesson 15. For a senior engineer, the ability to name a problem correctly in a code review is worth more than the ability to rewrite that piece of code by hand.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| là mặt sau của toàn bộ các bài | is the other side of all the lessons |
| chúng ta sẽ ngửi thấy mùi | we will smell something |
| đều có tên gọi riêng | all have their own names |
| để cả đội có chung một bộ từ vựng | so that the whole team shares one vocabulary |
| đổi cấu trúc mà vẫn giữ nguyên hành vi | changing the structure while still keeping the behaviour unchanged |
| khả năng gọi đúng tên một vấn đề | the ability to name a problem correctly |
| có giá trị hơn khả năng tự tay viết lại | is worth more than the ability to rewrite … by hand |

**Thuật ngữ cần nhớ**

- mùi code → **code smell**
- bộ từ vựng → **vocabulary**
- giữ nguyên hành vi → **keep the behaviour unchanged**
- cầu nối → **bridge**
- viết lại → **rewrite**

---

## Phần 2 — Smell không phải bug, smell là báo nợ

**Tiếng Việt**

Code smell là dấu hiệu bề mặt báo rằng có một vấn đề thiết kế tiềm ẩn nằm bên dưới. Điểm quan trọng nhất là smell không phải bug, bởi vì code có smell vẫn chạy đúng chức năng. Chúng ta vẫn phải quan tâm đến nó vì nó báo trước nợ kỹ thuật, nghĩa là nó báo trước rằng hệ thống sẽ khó bảo trì và khó mở rộng nếu chúng ta để lâu. Cách nghĩ đúng là chúng ta xem smell như một cái mùi lạ trong nhà, chứ chúng ta không xem nó như một đám cháy. Đám cháy thì phải dập ngay, còn cái mùi thì chúng ta ghi nhận, tìm nguồn, rồi xử lý theo thứ tự ưu tiên. Nếu chúng ta lẫn hai loại này với nhau, chúng ta sẽ hoặc hoảng lên một cách vô ích, hoặc bỏ mặc nợ cho tới khi nó quá lớn.

**English (bám cấu trúc tiếng Việt)**

A code smell is a surface sign telling us that there is a hidden design problem lying underneath. The most important point is that a smell is not a bug, because code with a smell still runs correctly. We still have to care about it because it warns us of technical debt, which means that it warns us that the system will be hard to maintain and hard to extend if we leave it for a long time. The right way to think is that we see a smell as a strange odour in the house, and we do not see it as a fire. A fire has to be put out immediately, while an odour is something we note down, trace to its source, and then handle in order of priority. If we mix these two kinds up, we will either panic for no reason, or leave the debt alone until it grows too big.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| dấu hiệu bề mặt báo rằng | a surface sign telling us that |
| một vấn đề thiết kế tiềm ẩn nằm bên dưới | a hidden design problem lying underneath |
| vẫn chạy đúng chức năng | still runs correctly |
| nó báo trước nợ kỹ thuật | it warns us of technical debt |
| nếu chúng ta để lâu | if we leave it for a long time |
| một cái mùi lạ trong nhà | a strange odour in the house |
| đám cháy thì phải dập ngay | a fire has to be put out immediately |
| xử lý theo thứ tự ưu tiên | handle in order of priority |
| hoảng lên một cách vô ích | panic for no reason |
| bỏ mặc nợ cho tới khi nó quá lớn | leave the debt alone until it grows too big |

**Thuật ngữ cần nhớ**

- nợ kỹ thuật → **technical debt**
- tiềm ẩn → **hidden** / **latent**
- dập lửa → **put out a fire**
- thứ tự ưu tiên → **order of priority**
- hoảng loạn → **panic**

---

## Phần 3 — God Object và Feature Envy

**Tiếng Việt**

Smell đầu tiên và cũng nổi tiếng nhất là God Object, tức là một class ôm quá nhiều trách nhiệm và biết quá nhiều thứ. Class kiểu này vi phạm SRP một cách rõ ràng, và nó có cohesion thấp cùng lúc với coupling cao. Vì mọi module đều gọi vào nó, mọi thay đổi đều phải đi qua nó, và rủi ro của mỗi lần sửa tăng dần theo thời gian. Cách chữa là chúng ta tách trách nhiệm theo actor, đúng như chúng ta đã học ở Bài 4. Dấu hiệu nhận ra rất đơn giản, đó là chúng ta mở một file dài hai nghìn dòng mà cái tên của file không nói lên được nó làm gì.

Smell thứ hai là Feature Envy, và nó xảy ra khi một method quan tâm đến dữ liệu của class khác nhiều hơn quan tâm đến dữ liệu của chính class chứa nó. Biểu hiện dễ thấy là method đó gọi liên tục các getter của một object khác rồi tự tính toán trên dữ liệu lấy về. Đây là dấu hiệu cho biết method đang nằm sai chỗ, chứ nó không phải dấu hiệu cho biết method được viết dở. Cách chữa chuẩn là chúng ta chuyển method đó sang chính class sở hữu dữ liệu, và thao tác này có tên riêng là move method. Sau khi chuyển xong, chuỗi getter biến mất và chúng ta cũng khử luôn phần nào vấn đề Law of Demeter ở Bài 2.

**English (bám cấu trúc tiếng Việt)**

The first and also the most famous smell is the God Object, that is, a class that holds too many responsibilities and knows too many things. A class like this violates SRP clearly, and it has low cohesion at the same time as high coupling. Because every module calls into it, every change has to go through it, and the risk of each edit grows over time. The cure is that we split the responsibilities by actor, exactly as we learned in Lesson 4. The sign for recognising it is very simple, which is that we open a file of two thousand lines whose name does not tell us what it does.

The second smell is Feature Envy, and it happens when a method cares about the data of another class more than it cares about the data of the class that holds it. The obvious symptom is that this method calls the getters of another object over and over and then computes on the data it fetched. This is a sign telling us that the method sits in the wrong place, and it is not a sign telling us that the method was written badly. The standard cure is that we move that method into the very class that owns the data, and this operation has its own name, which is move method. After the move is done, the getter chain disappears and we also partly remove the Law of Demeter problem from Lesson 2.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| ôm quá nhiều trách nhiệm | holds too many responsibilities |
| có cohesion thấp cùng lúc với coupling cao | has low cohesion at the same time as high coupling |
| rủi ro của mỗi lần sửa tăng dần theo thời gian | the risk of each edit grows over time |
| cái tên của file không nói lên được nó làm gì | the file name does not tell us what it does |
| quan tâm đến dữ liệu của class khác nhiều hơn | cares about the data of another class more |
| gọi liên tục các getter của một object khác | calls the getters of another object over and over |
| method đang nằm sai chỗ | the method sits in the wrong place |
| chính class sở hữu dữ liệu | the very class that owns the data |
| chúng ta cũng khử luôn phần nào | we also partly remove |

**Thuật ngữ cần nhớ**

- ghen tị tính năng (method sai chỗ) → **Feature Envy**
- biểu hiện → **symptom**
- chuyển method sang chỗ khác → **move method**
- sở hữu → **own**
- theo thời gian → **over time**

---

## Phần 4 — Divergent Change và Shotgun Surgery là một cặp ngược chiều

**Tiếng Việt**

Hai smell tiếp theo là một cặp ngược chiều nhau, và người phỏng vấn rất hay hỏi cặp này để kiểm tra xem chúng ta có nhầm hay không. Divergent Change là khi một class phải đổi vì nhiều lý do khác nhau, ví dụ nó đổi vì luật thuế và nó cũng đổi vì định dạng xuất file. Đó là dấu hiệu thiếu SRP, cho nên hướng xử lý là chúng ta tách class đó ra. Shotgun Surgery thì ngược lại, bởi vì đó là khi một thay đổi duy nhất buộc chúng ta phải sửa rải rác ở nhiều class. Đó là dấu hiệu coupling cao và thiếu gom nhóm, cho nên hướng xử lý là chúng ta gộp những phần liên quan lại một chỗ. Cách nhớ ngắn gọn là Divergent thì tách ra, còn Shotgun thì gộp lại.

**English (bám cấu trúc tiếng Việt)**

The next two smells are a pair that point in opposite directions, and interviewers very often ask about this pair to check whether we mix them up or not. Divergent Change is when one class has to change for many different reasons, for example it changes because of the tax rules and it also changes because of the export file format. That is a sign of missing SRP, so the way to handle it is that we split that class apart. Shotgun Surgery is the opposite, because it is when a single change forces us to edit scattered places in many classes. That is a sign of high coupling and of missing grouping, so the way to handle it is that we gather the related parts into one place. The short way to remember is that Divergent means split apart, while Shotgun means gather together.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| một cặp ngược chiều nhau | a pair that point in opposite directions |
| để kiểm tra xem chúng ta có nhầm hay không | to check whether we mix them up or not |
| nó đổi vì luật thuế | it changes because of the tax rules |
| định dạng xuất file | the export file format |
| dấu hiệu thiếu SRP | a sign of missing SRP |
| một thay đổi duy nhất buộc chúng ta phải sửa rải rác | a single change forces us to edit scattered places |
| thiếu gom nhóm | missing grouping |
| gộp những phần liên quan lại một chỗ | gather the related parts into one place |
| cách nhớ ngắn gọn là | the short way to remember is that |

**Thuật ngữ cần nhớ**

- thay đổi phân kỳ → **Divergent Change**
- sửa vá rải rác → **Shotgun Surgery**
- thuế → **tax**
- định dạng → **format**
- rải rác → **scattered**

---

## Phần 5 — Primitive Obsession, Data Clumps và Value Object

**Tiếng Việt**

Primitive Obsession là việc chúng ta lạm dụng kiểu nguyên thuỷ thay cho kiểu của domain, ví dụ chúng ta dùng một chuỗi cho email và dùng một số cho tiền. Data Clumps là việc một nhóm tham số luôn đi cùng nhau ở mọi nơi, ví dụ vĩ độ, kinh độ và độ cao. Hai smell này thường xuất hiện cùng nhau, và hậu quả là phần kiểm tra hợp lệ bị rải khắp nơi thay vì nằm gọn một chỗ. Cách chữa là chúng ta tạo ra một Value Object, nghĩa là chúng ta gói nhóm dữ liệu đó vào một kiểu riêng và đặt phần kiểm tra hợp lệ ngay bên trong kiểu đó. Ví dụ chúng ta tạo một kiểu `Money` giữ cả số tiền lẫn đơn vị tiền tệ, và kiểu đó không cho phép tạo ra một số tiền âm. Ý này nối thẳng sang DDD ở Bài 13, nơi Value Object là một khái niệm trung tâm.

**English (bám cấu trúc tiếng Việt)**

Primitive Obsession is when we overuse primitive types in place of domain types, for example we use a string for an email and we use a number for money. Data Clumps is when a group of parameters always travels together everywhere, for example latitude, longitude and altitude. These two smells usually appear together, and the consequence is that the validation gets scattered everywhere instead of sitting neatly in one place. The cure is that we create a Value Object, which means that we wrap that group of data into its own type and we put the validation right inside that type. For example we create a `Money` type that holds both the amount and the currency, and that type does not allow a negative amount to be created. This idea connects straight to DDD in Lesson 13, where the Value Object is a central concept.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| lạm dụng kiểu nguyên thuỷ thay cho kiểu của domain | overuse primitive types in place of domain types |
| một nhóm tham số luôn đi cùng nhau ở mọi nơi | a group of parameters always travels together everywhere |
| thường xuất hiện cùng nhau | usually appear together |
| bị rải khắp nơi thay vì nằm gọn một chỗ | gets scattered everywhere instead of sitting neatly in one place |
| gói nhóm dữ liệu đó vào một kiểu riêng | wrap that group of data into its own type |
| ngay bên trong kiểu đó | right inside that type |
| giữ cả số tiền lẫn đơn vị tiền tệ | holds both the amount and the currency |
| không cho phép tạo ra một số tiền âm | does not allow a negative amount to be created |
| là một khái niệm trung tâm | is a central concept |

**Thuật ngữ cần nhớ**

- kiểu nguyên thuỷ → **primitive type**
- nhóm dữ liệu luôn đi cùng → **data clump**
- vĩ độ / kinh độ → **latitude** / **longitude**
- đơn vị tiền tệ → **currency**
- đối tượng giá trị → **Value Object**

---

## Phần 6 — Long Method, guard clause và extract method

**Tiếng Việt**

Long Method là smell dễ nhận nhất, bởi vì chúng ta chỉ cần nhìn độ dài của method là thấy ngay. Dấu hiệu đi kèm thường là nhiều cấp lồng nhau và những comment dùng để chia method thành từng đoạn. Khi chúng ta thấy một comment đặt tên cho một đoạn, đó thường là chỗ đáng tách ra thành một method riêng, và thao tác này gọi là extract method. Kỹ thuật thứ hai là guard clause, nghĩa là chúng ta kiểm tra các điều kiện sai ngay ở đầu hàm rồi thoát sớm, nhờ đó phần thân chính không còn bị lồng sâu nữa. Điều bắt buộc trước khi làm hai việc trên là chúng ta phải có test, và nếu code cũ chưa có test thì chúng ta viết characterization test để chụp lại hành vi hiện tại. Nếu chúng ta bỏ qua bước test, chúng ta sẽ không chứng minh được rằng hành vi vẫn giữ nguyên, và lúc đó việc chúng ta làm không còn là refactor nữa.

**English (bám cấu trúc tiếng Việt)**

The Long Method is the easiest smell to spot, because we only need to look at the length of the method and we see it right away. The symptom that comes with it is usually many levels of nesting and comments used to divide the method into sections. When we see a comment that names a section, that is usually the place worth extracting into its own method, and this operation is called extract method. The second technique is the guard clause, which means that we check the invalid conditions right at the top of the function and then exit early, thanks to that the main body is no longer deeply nested. The thing that is required before we do those two things is that we must have tests, and if the old code does not have tests yet then we write characterization tests to capture the current behaviour. If we skip the testing step, we will not be able to prove that the behaviour stayed the same, and at that point what we are doing is no longer a refactor.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| chúng ta chỉ cần nhìn … là thấy ngay | we only need to look at … and we see it right away |
| nhiều cấp lồng nhau | many levels of nesting |
| dùng để chia method thành từng đoạn | used to divide the method into sections |
| chỗ đáng tách ra thành một method riêng | the place worth extracting into its own method |
| kiểm tra các điều kiện sai ngay ở đầu hàm rồi thoát sớm | check the invalid conditions right at the top of the function and then exit early |
| phần thân chính không còn bị lồng sâu nữa | the main body is no longer deeply nested |
| điều bắt buộc trước khi làm hai việc trên | the thing that is required before we do those two things |
| để chụp lại hành vi hiện tại | to capture the current behaviour |
| việc chúng ta làm không còn là refactor nữa | what we are doing is no longer a refactor |

**Thuật ngữ cần nhớ**

- lồng nhau → **nesting**
- tách hàm → **extract method**
- câu chặn đầu hàm → **guard clause**
- thoát sớm → **exit early**
- test chụp hành vi hiện tại → **characterization test**

---

## Phần 7 — Định nghĩa chặt của refactoring, và anti-pattern là gì

**Tiếng Việt**

Theo định nghĩa chặt của Martin Fowler, refactoring là việc thay đổi cấu trúc bên trong của code mà không đổi hành vi quan sát được từ bên ngoài. Test chính là tấm lưới an toàn chứng minh rằng hành vi không đổi, cho nên refactor mà không có test chỉ là viết lại code rồi hy vọng. Chúng ta phải phân biệt rõ giữa refactor và việc sửa kèm thêm tính năng, bởi vì hai việc đó có mục tiêu khác nhau. Khi chúng ta trộn hai việc vào cùng một pull request, người review không biết đâu là thay đổi cấu trúc và đâu là thay đổi hành vi. Hậu quả rất thực tế là khi có sự cố, chúng ta không rollback được phần tính năng mà vẫn giữ được phần dọn dẹp.

Anti-pattern khác code smell ở chỗ nó là một giải pháp chứ nó không phải một dấu hiệu. Nói cụ thể hơn, anti-pattern là một cách làm phổ biến nhưng phản tác dụng, và nó phổ biến chính vì nó giải quyết được vấn đề trước mắt. Vài ví dụ quen thuộc trong backend là Golden Hammer, tức là việc gì cũng dùng đúng một công cụ quen tay, và Lava Flow, tức là code chết nằm đó mà không ai dám xoá. Danh sách còn có code rối như mì spaghetti, những con số và chuỗi ma thuật nằm rải rác, và việc tối ưu quá sớm. Điểm chung của chúng là chúng trông có vẻ đúng ở thời điểm viết, nhưng chúng tạo ra một khoản nợ phải trả về sau.

**English (bám cấu trúc tiếng Việt)**

According to Martin Fowler's strict definition, refactoring is changing the internal structure of the code without changing the behaviour observable from the outside. Tests are exactly the safety net that proves the behaviour did not change, so refactoring without tests is only rewriting code and then hoping. We have to draw a clear line between refactoring and fixing while adding a feature, because those two jobs have different goals. When we mix the two jobs into the same pull request, the reviewer does not know which part is a structural change and which part is a behavioural change. The very practical consequence is that when an incident happens, we cannot roll back the feature part while still keeping the clean-up part.

An anti-pattern differs from a code smell in that it is a solution and it is not a sign. To say it more concretely, an anti-pattern is a common but counterproductive way of doing things, and it is common exactly because it solves the problem in front of us. A few familiar examples in the backend are the Golden Hammer, that is, using the same familiar tool for every job, and the Lava Flow, that is, dead code sitting there that nobody dares to delete. The list also includes code as tangled as spaghetti, magic numbers and magic strings scattered around, and premature optimisation. What they have in common is that they look correct at the moment they are written, but they create a debt that has to be paid later.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| theo định nghĩa chặt của | according to … strict definition |
| hành vi quan sát được từ bên ngoài | the behaviour observable from the outside |
| tấm lưới an toàn | the safety net |
| chỉ là viết lại code rồi hy vọng | is only rewriting code and then hoping |
| chúng ta phải phân biệt rõ giữa | we have to draw a clear line between |
| khi có sự cố | when an incident happens |
| chúng ta không rollback được phần tính năng | we cannot roll back the feature part |
| một cách làm phổ biến nhưng phản tác dụng | a common but counterproductive way of doing things |
| việc gì cũng dùng đúng một công cụ quen tay | using the same familiar tool for every job |
| code chết nằm đó mà không ai dám xoá | dead code sitting there that nobody dares to delete |
| những con số và chuỗi ma thuật nằm rải rác | magic numbers and magic strings scattered around |
| một khoản nợ phải trả về sau | a debt that has to be paid later |

**Thuật ngữ cần nhớ**

- quan sát được → **observable**
- lưới an toàn → **safety net**
- quay lui bản triển khai → **roll back**
- phản tác dụng → **counterproductive**
- code chết → **dead code**
- tối ưu quá sớm → **premature optimisation**

---

## Phần 8 — Lạm dụng pattern, và quyết định khi deadline gấp

**Tiếng Việt**

Chúng ta cũng phải nói rõ rằng lạm dụng design pattern chính là một smell. Trường hợp điển hình là pattern đúng nhưng được dùng sai chỗ, ví dụ Decorator chồng quá sâu khiến chuỗi object rất khó debug và tốn thêm chi phí gọi. Ví dụ khác là Singleton tạo ra global state, hoặc Factory được dựng lên cho thứ mà chúng ta chỉ khởi tạo đúng một lần. Câu trả lời cho câu hỏi bẫy rằng dự án nên dùng thật nhiều pattern để chứng tỏ chất lượng là không. Pattern là công cụ có chi phí, chứ pattern không phải huy chương để trưng bày.

Cuối cùng là quyết định mà một Tech Lead phải đưa ra khi thấy smell nhưng deadline đang gấp. Chúng ta phân loại theo ba tiêu chí, đó là rủi ro lan rộng, tần suất đội chạm vào chỗ đó, và mức rủi ro của chính thay đổi. Từ ba tiêu chí đó, chúng ta chọn một trong ba hướng, hoặc sửa ngay, hoặc ghi nợ lại, hoặc bỏ qua một cách có chủ đích. Nếu chúng ta ghi nợ, chúng ta phải ghi tường minh bằng một ticket có người chịu trách nhiệm và có lý do, chứ chúng ta không để lại một dòng TODO vô chủ. Chúng ta cũng tránh kiểu dọn dẹp lan man ra ngoài phạm vi của pull request, bởi vì điều đó làm người review mất phương hướng. Riêng với code do AI sinh ra, chúng ta soát ba thứ quen thuộc là method dài, service ôm quá nhiều việc, và những con số ma thuật, và chúng ta luôn hỏi thêm rằng bản refactor mà AI đề xuất có lén đổi hành vi hay không.

**English (bám cấu trúc tiếng Việt)**

We also have to say clearly that overusing design patterns is itself a smell. The typical case is a pattern that is correct but is used in the wrong place, for example Decorators stacked too deeply which makes the object chain very hard to debug and costs extra call overhead. Another example is a Singleton that creates global state, or a Factory that is built for something we only instantiate once. The answer to the trap question that a project should use plenty of patterns in order to prove its quality is no. A pattern is a tool that has a cost, and a pattern is not a medal to be displayed.

Finally there is the decision that a Tech Lead has to make when they see a smell but the deadline is tight. We classify it by three criteria, which are the risk of spreading, how often the team touches that place, and the risk level of the change itself. From those three criteria, we choose one of three directions, either fix it now, or log the debt, or skip it deliberately. If we log the debt, we have to log it explicitly with a ticket that has an owner and a reason, and we do not leave behind an ownerless TODO line. We also avoid the kind of clean-up that wanders outside the scope of the pull request, because that makes the reviewer lose their bearings. For code that AI generates in particular, we check three familiar things, which are long methods, services that hold too many jobs, and magic numbers, and we always ask in addition whether the refactor that AI proposes quietly changes the behaviour or not.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| lạm dụng design pattern chính là một smell | overusing design patterns is itself a smell |
| khiến chuỗi object rất khó debug | which makes the object chain very hard to debug |
| tốn thêm chi phí gọi | costs extra call overhead |
| pattern không phải huy chương để trưng bày | a pattern is not a medal to be displayed |
| khi deadline đang gấp | when the deadline is tight |
| tần suất đội chạm vào chỗ đó | how often the team touches that place |
| bỏ qua một cách có chủ đích | skip it deliberately |
| một dòng TODO vô chủ | an ownerless TODO line |
| dọn dẹp lan man ra ngoài phạm vi | clean-up that wanders outside the scope |
| làm người review mất phương hướng | makes the reviewer lose their bearings |
| có lén đổi hành vi hay không | quietly changes the behaviour or not |

**Thuật ngữ cần nhớ**

- chi phí phụ trội → **overhead**
- huy chương → **medal**
- tiêu chí → **criteria**
- ghi nợ lại → **log the debt**
- phạm vi công việc → **scope**

---

## Mô hình ghi nhớ

**Tiếng Việt**

Smell là mùi báo nợ, chứ smell không phải là đám cháy đang bùng lên. Refactor là đổi bên trong mà giữ nguyên bên ngoài, với test làm lưới an toàn, và pattern là công cụ có giá phải trả chứ không phải huy chương.

**English (bám cấu trúc tiếng Việt)**

A smell is an odour warning of debt, and a smell is not a fire that is breaking out. Refactoring is changing the inside while keeping the outside unchanged, with tests as the safety net, and a pattern is a tool with a price to pay and not a medal.

---

## Bảng thuật ngữ tổng hợp

| Tiếng Việt | Tiếng Anh | Ghi chú phát âm |
|---|---|---|
| mùi code | code smell | |
| bộ từ vựng | vocabulary | trọng âm âm hai: vo-CA-bu-la-ry |
| cầu nối | bridge | đuôi **-dge** đọc "j" |
| viết lại | rewrite | |
| nợ kỹ thuật | technical debt | *debt* /det/ — chữ **b** câm, đọc là "đet" |
| tiềm ẩn | hidden / latent | *latent* /ˈleɪtənt/ — LAY-tent |
| dập lửa | put out a fire | |
| thứ tự ưu tiên | order of priority | *priority* trọng âm âm hai: pri-O-ri-ty |
| hoảng loạn | panic | trọng âm đầu: PA-nic |
| ghen tị tính năng | Feature Envy | *envy* /ˈenvi/ — EN-vy |
| biểu hiện | symptom | /ˈsɪmptəm/ — SIMP-tom, chữ **p** có đọc |
| chuyển method sang chỗ khác | move method | |
| theo thời gian | over time | |
| thay đổi phân kỳ | Divergent Change | *divergent* trọng âm âm hai: di-VER-gent |
| sửa vá rải rác | Shotgun Surgery | *surgery* /ˈsɜːdʒəri/ — SUR-ge-ry, trọng âm đầu |
| thuế | tax | |
| định dạng | format | danh từ trọng âm đầu: FOR-mat |
| rải rác | scattered | trọng âm đầu: SCA-ttered |
| kiểu nguyên thuỷ | primitive type | *primitive* trọng âm đầu: PRI-mi-tive |
| ám ảnh, lạm dụng | obsession | trọng âm âm hai: ob-SE-ssion |
| nhóm dữ liệu luôn đi cùng | data clump | *clump* — cụm **-mp** cuối phải bật |
| vĩ độ / kinh độ | latitude / longitude | LA-ti-tude / LON-gi-tude, cả hai trọng âm đầu |
| đơn vị tiền tệ | currency | trọng âm đầu: CU-rren-cy |
| đối tượng giá trị | Value Object | |
| lồng nhau | nesting | |
| tách hàm | extract method | động từ *extract* trọng âm cuối: ex-TRACT |
| câu chặn đầu hàm | guard clause | *clause* /klɔːz/ — đuôi đọc **-z** |
| thoát sớm | exit early | *exit* Anh đọc /ˈeksɪt/ — EK-sit |
| test chụp hành vi hiện tại | characterization test | cha-rac-te-ri-ZA-tion, đọc chậm từng âm |
| quan sát được | observable | trọng âm âm hai: ob-SER-va-ble |
| lưới an toàn | safety net | |
| quay lui bản triển khai | roll back | |
| phản tác dụng | counterproductive | trọng âm áp chót: coun-ter-pro-DUC-tive |
| code chết | dead code | |
| dòng dung nham | Lava Flow | *lava* /ˈlɑːvə/ — LAA-va |
| mì rối | spaghetti | trọng âm âm hai: spa-GHE-tti, **gh** đọc như "g" |
| số ma thuật | magic number | |
| tối ưu quá sớm | premature optimisation | *premature* Anh đọc pre-ma-TURE |
| chi phí phụ trội | overhead | trọng âm đầu khi là danh từ: O-ver-head |
| huy chương | medal | /ˈmedl/ — ME-dl, không đọc giống *metal* |
| tiêu chí | criteria | số nhiều cri-TE-ri-a; số ít là *criterion* |
| ghi nợ lại | log the debt | |
| phạm vi công việc | scope | cụm **sc-** đầu và đuôi **-p** phải rõ |
| sự cố | incident | trọng âm đầu: IN-ci-dent |

---

## Bài nói

> **Cách làm:** ghi âm lại giọng của mình, mỗi câu nói liên tục **60–90 giây**, **không nhìn tài liệu** khi nói. Mỗi bài nói phải dùng ít nhất **6 thuật ngữ** trong bảng ở trên. Nghe lại bản ghi âm một lần, ghi ra hai chỗ mình bị vấp, rồi nói lại lần hai.

1. Explain to a junior developer what a code smell is, why it is not a bug, and why the team should still care about it.
2. Explain the difference between Divergent Change and Shotgun Surgery, and say which direction the fix goes in each case.
3. A colleague says: "This project uses fifteen design patterns, so the design must be high quality." Explain what is wrong with that reasoning.
4. Someone on your team wants to refactor a long method and add a new discount rule in the same pull request. Explain why you would push back and how you would split the work.
5. Describe how you would safely refactor a four-hundred-line `OrderProcessor` that currently has no tests, step by step.
6. When would you fix a smell immediately, and when would you log it as debt and move on? Give the criteria you use and the trade-off in both directions.
7. You are reviewing an AI-generated "refactor" of a pricing service. Describe out loud how you would check that the behaviour did not change, and which smells you would look for first.
