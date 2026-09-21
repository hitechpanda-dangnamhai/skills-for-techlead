# Bài 15 — Judgment Tech Lead: chống over-engineering · enforce design · ADR · evolutionary architecture · kiểm soát AI-code
### Bản song ngữ: tiếng Anh bám cấu trúc tiếng Việt

> **Cách dùng bài này:** đọc đoạn tiếng Việt trước, tự dịch trong đầu, rồi mới nhìn sang đoạn tiếng Anh để so. Sau khi so xong, **đọc to đoạn tiếng Anh hai lần** — lần một để nhìn chữ, lần hai để nghe chính mình. Phần bài nói ở cuối là bắt buộc; không nói ra thì bài này chỉ là bài đọc.

---

## Phần 1 — Tầng meta nằm trên toàn bộ mười bốn bài trước

**Tiếng Việt**

Bài này là tầng meta nằm trên toàn bộ mười bốn bài trước. Nội dung của nó gồm ba việc, đó là chúng ta biết khi nào nên dùng và khi nào không nên dùng những thứ đã học, chúng ta biết cách enforce nguyên tắc trong đội, và chúng ta biết cách kiểm soát chất lượng thiết kế của code do AI sinh ra. Đây chính là phần phân biệt một kỹ sư senior với một Tech Lead thật sự. Người phỏng vấn ở vòng cuối thường hỏi đúng những câu trong bài này, bởi vì họ muốn biết chúng ta ra quyết định như thế nào chứ họ không muốn biết chúng ta nhớ được bao nhiêu pattern. Nếu chúng ta nói tốt phần này, chúng ta bù lại được cho vài chỗ vấp ở các bài kỹ thuật phía trước.

**English (bám cấu trúc tiếng Việt)**

This lesson is the meta layer sitting on top of all fourteen previous lessons. Its content covers three things, which are knowing when to use and when not to use the things we have learned, knowing how to enforce principles inside the team, and knowing how to control the design quality of code that AI generates. This is exactly the part that separates a senior engineer from a real Tech Lead. Interviewers in the final round usually ask exactly the questions in this lesson, because they want to know how we make decisions and they do not want to know how many patterns we can remember. If we speak well on this part, we can make up for a few stumbles in the technical lessons earlier.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| tầng meta nằm trên toàn bộ | the meta layer sitting on top of all |
| khi nào nên dùng và khi nào không nên dùng | when to use and when not to use |
| kiểm soát chất lượng thiết kế | control the design quality |
| phân biệt một kỹ sư senior với một Tech Lead thật sự | separates a senior engineer from a real Tech Lead |
| ở vòng cuối | in the final round |
| chúng ta ra quyết định như thế nào | how we make decisions |
| chúng ta bù lại được cho vài chỗ vấp | we can make up for a few stumbles |

**Thuật ngữ cần nhớ**

- tầng bao trùm → **meta layer**
- áp dụng nghiêm, bắt tuân thủ → **enforce**
- vòng phỏng vấn cuối → **the final round**
- ra quyết định → **make decisions**
- vấp, hụt → **stumble**

---

## Phần 2 — Khi nào KHÔNG nên dùng một design pattern

**Tiếng Việt**

Nguyên tắc chung để quyết định khi nào không dùng một design pattern rất gọn. Chúng ta không dùng nó khi nó thêm một tầng gián tiếp hoặc một abstraction mà chưa có biến thiên thật và chưa có cái đau thật. Chúng ta cũng không dùng nó khi cả đội chưa quen với pattern đó, bởi vì lúc đó code trở nên khó đọc hơn chứ nó không dễ đọc hơn. Cách chọn đúng là chúng ta chọn pattern để giải một vấn đề cụ thể, chứ chúng ta không chọn nó cho sang. Mặc định của chúng ta luôn là giải pháp đơn giản nhất, đúng tinh thần KISS. Nếu chúng ta không giữ được mặc định đó, thì mỗi lần đội học được thứ gì mới, codebase lại mọc thêm một tầng.

**English (bám cấu trúc tiếng Việt)**

The general principle for deciding when not to use a design pattern is very short. We do not use it when it adds a layer of indirection or an abstraction while there is no real variation yet and no real pain yet. We also do not use it when the whole team is not familiar with that pattern, because at that point the code becomes harder to read and it does not become easier to read. The right way to choose is that we pick a pattern to solve a specific problem, and we do not pick it to look impressive. Our default is always the simplest solution, exactly in the spirit of KISS. If we cannot hold that default, then every time the team learns something new, the codebase grows one more layer.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| nguyên tắc chung để quyết định | the general principle for deciding |
| chưa có biến thiên thật và chưa có cái đau thật | there is no real variation yet and no real pain yet |
| khi cả đội chưa quen với pattern đó | when the whole team is not familiar with that pattern |
| code trở nên khó đọc hơn | the code becomes harder to read |
| chúng ta không chọn nó cho sang | we do not pick it to look impressive |
| mặc định của chúng ta luôn là | our default is always |
| nếu chúng ta không giữ được mặc định đó | if we cannot hold that default |
| codebase lại mọc thêm một tầng | the codebase grows one more layer |

**Thuật ngữ cần nhớ**

- tầng gián tiếp → **layer of indirection**
- biến thiên → **variation**
- quen thuộc với → **familiar with**
- mặc định → **default**
- cho sang, cho oai → **to look impressive**

---

## Phần 3 — Over-engineering và under-engineering là hai phía của một cái cân

**Tiếng Việt**

Chúng ta phải nhận ra được cả hai phía của cái cân, bởi vì lệch về phía nào cũng tốn kém. Dấu hiệu của over-engineering gồm quá nhiều tầng, những interface một-một, những abstraction không bao giờ có hiện thực thứ hai, và sự linh hoạt được xây sẵn cho một tương lai chưa tới. Dấu hiệu của under-engineering thì ngược lại, gồm việc sao chép dán khắp nơi, coupling rất chặt, và không có chỗ nào để cắm test vào. Hai bên gây ra hai loại chi phí khác nhau, bởi vì bên thừa làm chậm mọi người mỗi ngày, còn bên thiếu làm mọi thay đổi trở nên rủi ro. Điểm cân bằng không cố định, mà nó phụ thuộc vào bối cảnh gồm độ phức tạp, vòng đời của hệ thống, và năng lực của đội. Khi phỏng vấn, việc nói được cả hai phía cho thấy chúng ta đã từng trả giá cho cả hai.

**English (bám cấu trúc tiếng Việt)**

We have to recognise both sides of the scale, because tipping to either side is expensive. The signs of over-engineering include too many layers, one-to-one interfaces, abstractions that never get a second implementation, and flexibility built in advance for a future that has not arrived. The signs of under-engineering are the opposite, and they include copy-paste everywhere, very tight coupling, and no place to plug a test into. The two sides cause two different kinds of cost, because the excessive side slows everybody down every day, while the deficient side makes every change risky. The balance point is not fixed, but it depends on the context, which includes the complexity, the lifetime of the system, and the capability of the team. In an interview, being able to speak about both sides shows that we have paid the price for both.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| cả hai phía của cái cân | both sides of the scale |
| lệch về phía nào cũng tốn kém | tipping to either side is expensive |
| những abstraction không bao giờ có hiện thực thứ hai | abstractions that never get a second implementation |
| sự linh hoạt được xây sẵn cho một tương lai chưa tới | flexibility built in advance for a future that has not arrived |
| việc sao chép dán khắp nơi | copy-paste everywhere |
| không có chỗ nào để cắm test vào | no place to plug a test into |
| bên thừa làm chậm mọi người mỗi ngày | the excessive side slows everybody down every day |
| bên thiếu làm mọi thay đổi trở nên rủi ro | the deficient side makes every change risky |
| chúng ta đã từng trả giá cho cả hai | we have paid the price for both |

**Thuật ngữ cần nhớ**

- làm quá tay → **over-engineering**
- làm thiếu → **under-engineering**
- cái cân → **the scale**
- điểm cân bằng → **the balance point**
- năng lực → **capability**

---

## Phần 4 — Dẫn dắt một cuộc thảo luận thiết kế thay vì chặn cứng

**Tiếng Việt**

Bây giờ chúng ta xử lý một tình huống rất thật. Một bạn dev giỏi muốn áp DDD đầy đủ, hexagonal và CQRS cho một service CRUD nội bộ rất nhỏ. Cách xử lý sai thứ nhất là chúng ta chặn cứng, bởi vì làm vậy thì bạn ấy mất động lực và đội mất một người chịu học. Cách xử lý sai thứ hai là chúng ta chiều theo, bởi vì cả đội sẽ phải nuôi mức phức tạp đó trong nhiều năm. Cách đúng là chúng ta dẫn dắt bằng những câu hỏi về giá trị và chi phí, gắn với độ phức tạp thực tế, vòng đời dự kiến và năng lực của đội. Chúng ta cũng có thể đề xuất một hướng ở giữa, đó là bắt đầu đơn giản nhưng để sẵn seam cho hệ thống tiến hoá về sau, và quan trọng nhất là chúng ta ra quyết định kèm theo lý do rõ ràng.

**English (bám cấu trúc tiếng Việt)**

Now let us handle a very real situation. A strong developer wants to apply full DDD, hexagonal and CQRS to a very small internal CRUD service. The first wrong way to handle it is that we block it flatly, because doing so makes them lose their motivation and the team loses somebody willing to learn. The second wrong way to handle it is that we simply give in, because the whole team will have to feed that level of complexity for years. The right way is that we guide with questions about value and cost, tied to the real complexity, the expected lifetime and the capability of the team. We can also propose a middle path, which is to start simple but to leave seams ready for the system to evolve later, and the most important thing is that we make the decision together with a clear reason.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| chúng ta xử lý một tình huống rất thật | let us handle a very real situation |
| cách xử lý sai thứ nhất là chúng ta chặn cứng | the first wrong way to handle it is that we block it flatly |
| bạn ấy mất động lực | they lose their motivation |
| đội mất một người chịu học | the team loses somebody willing to learn |
| chúng ta chiều theo | we simply give in |
| sẽ phải nuôi mức phức tạp đó trong nhiều năm | will have to feed that level of complexity for years |
| chúng ta dẫn dắt bằng những câu hỏi về giá trị và chi phí | we guide with questions about value and cost |
| chúng ta có thể đề xuất một hướng ở giữa | we can also propose a middle path |
| để sẵn seam cho hệ thống tiến hoá về sau | leave seams ready for the system to evolve later |

**Thuật ngữ cần nhớ**

- chặn cứng → **block flatly**
- chiều theo, nhượng bộ → **give in**
- động lực → **motivation**
- hướng ở giữa → **a middle path**
- tiến hoá → **evolve**

---

## Phần 5 — Enforce bằng công cụ để không tự biến mình thành nút thắt

**Tiếng Việt**

Một câu hỏi rất hay ở vòng Tech Lead là làm sao enforce nguyên tắc thiết kế trong một đội đông mà chúng ta không tự biến mình thành nút thắt cổ chai. Câu trả lời là chúng ta công cụ hoá thay vì phụ thuộc vào con người. Cụ thể là chúng ta thêm luật kiểm tra ranh giới kiến trúc vào CI, ví dụ chúng ta dùng `dependency-cruiser` hoặc một plugin về boundary của ESLint để chặn tự động việc thư mục domain import thư mục hạ tầng. Bên cạnh đó chúng ta dùng ADR kèm một mẫu chuẩn, chúng ta giữ những buổi design review nhẹ nhàng, và chúng ta kèm cặp trực tiếp cho người mới. Ý tưởng cốt lõi là chúng ta biến nguyên tắc thành lan can bảo vệ tự động, chứ chúng ta không đứng làm người gác cổng. Khi một pull request cố tình vi phạm thì CI báo lỗi ngay, và không ai phải tranh cãi trong phần bình luận nữa.

**English (bám cấu trúc tiếng Việt)**

A very good question in the Tech Lead round is how to enforce design principles in a large team without turning ourselves into a bottleneck. The answer is that we turn it into tooling instead of depending on people. Concretely, we add architecture boundary rules into CI, for example we use `dependency-cruiser` or a boundary plugin for ESLint in order to block automatically the case where the domain folder imports the infrastructure folder. Alongside that we use ADRs together with a standard template, we keep design reviews light, and we mentor newcomers directly. The core idea is that we turn principles into automatic guardrails, and we do not stand there as the gatekeeper. When a pull request deliberately violates the rule then CI fails immediately, and nobody has to argue in the comments any more.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| trong một đội đông | in a large team |
| mà chúng ta không tự biến mình thành nút thắt cổ chai | without turning ourselves into a bottleneck |
| chúng ta công cụ hoá thay vì phụ thuộc vào con người | we turn it into tooling instead of depending on people |
| luật kiểm tra ranh giới kiến trúc | architecture boundary rules |
| để chặn tự động việc | in order to block automatically the case where |
| chúng ta giữ những buổi design review nhẹ nhàng | we keep design reviews light |
| chúng ta kèm cặp trực tiếp cho người mới | we mentor newcomers directly |
| lan can bảo vệ tự động | automatic guardrails |
| chúng ta không đứng làm người gác cổng | we do not stand there as the gatekeeper |
| không ai phải tranh cãi trong phần bình luận nữa | nobody has to argue in the comments any more |

**Thuật ngữ cần nhớ**

- nút thắt cổ chai → **bottleneck**
- công cụ hoá → **turn into tooling**
- lan can bảo vệ → **guardrail**
- người gác cổng → **gatekeeper**
- kèm cặp → **mentor**

---

## Phần 6 — ADR giữ lại phần "vì sao"

**Tiếng Việt**

ADR là viết tắt của Architecture Decision Record, và nó là một tài liệu rất ngắn ghi lại một quyết định kiến trúc. Một ADR tối thiểu gồm bốn phần, và chúng ta nên thuộc đúng bốn phần này. Phần thứ nhất là bối cảnh, tức là tình huống và các ràng buộc tại thời điểm ra quyết định. Phần thứ hai là quyết định, phần thứ ba là các lựa chọn khác đã được cân nhắc, và phần thứ tư là hệ quả cùng với trạng thái hiện tại của quyết định. Giá trị lớn nhất của ADR là nó giữ lại phần "vì sao", nhờ đó người mới vào dự án hiểu nhanh và cả đội không tranh luận lại cùng một chuyện sau mỗi sáu tháng. Trong thực tế, chúng ta chỉ cần một thư mục trong repo chứa các file được đánh số, và mỗi file dài chưa tới một trang.

**English (bám cấu trúc tiếng Việt)**

ADR stands for Architecture Decision Record, and it is a very short document recording one architectural decision. A minimal ADR has four parts, and we should know these four parts by heart. The first part is the context, that is, the situation and the constraints at the moment of the decision. The second part is the decision, the third part is the alternatives that were considered, and the fourth part is the consequences together with the current status of the decision. The greatest value of an ADR is that it keeps the "why", thanks to which new people on the project understand quickly and the whole team does not argue the same thing again every six months. In practice, we only need one folder in the repository holding numbered files, and each file is less than one page long.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| là viết tắt của | stands for |
| một tài liệu rất ngắn ghi lại | a very short document recording |
| chúng ta nên thuộc đúng bốn phần này | we should know these four parts by heart |
| các ràng buộc tại thời điểm ra quyết định | the constraints at the moment of the decision |
| các lựa chọn khác đã được cân nhắc | the alternatives that were considered |
| hệ quả cùng với trạng thái hiện tại | the consequences together with the current status |
| nó giữ lại phần "vì sao" | it keeps the "why" |
| không tranh luận lại cùng một chuyện sau mỗi sáu tháng | does not argue the same thing again every six months |
| các file được đánh số | numbered files |

**Thuật ngữ cần nhớ**

- bản ghi quyết định kiến trúc → **Architecture Decision Record**
- bối cảnh → **context**
- lựa chọn thay thế → **alternative**
- hệ quả → **consequence**
- trạng thái → **status**

---

## Phần 7 — Thiết kế để dễ đổi, chứ không phải để đoán mò tương lai

**Tiếng Việt**

Evolutionary architecture là cách thiết kế sao cho hệ thống dễ đổi mà chúng ta không phải đoán trước mọi tương lai. Nguyên tắc là chúng ta tạo seam và abstraction tại đúng điểm thực sự biến động, còn ở những chỗ chưa rõ thì chúng ta để hở thay vì khoá sớm. Chúng ta phải phân biệt thật rạch ròi giữa việc làm cho dễ đổi và việc đoán mò tương lai. Làm cho dễ đổi nghĩa là chúng ta đặt seam tại chỗ đã đau thật, còn đoán mò nghĩa là chúng ta dựng abstraction cho một nhu cầu chưa tồn tại. Cách vận hành là chúng ta giữ YAGNI làm mặc định, rồi chúng ta refactor liên tục ngay khi biến thiên xuất hiện. Ở quy mô lớn hơn, nhiều đội còn viết những bài test tự động cho chính các thuộc tính kiến trúc, và người ta gọi chúng là fitness function.

**English (bám cấu trúc tiếng Việt)**

Evolutionary architecture is the way of designing so that the system is easy to change without us having to predict every future. The principle is that we create seams and abstractions at exactly the points that really move, while at the points that are still unclear we leave them open instead of locking them early. We have to draw a very sharp line between making things easy to change and guessing at the future. Making things easy to change means that we put a seam where the pain has already been felt, while guessing means that we build an abstraction for a need that does not exist yet. The way to operate is that we keep YAGNI as the default, and then we refactor continuously as soon as the variation appears. At a larger scale, many teams even write automated tests for the architectural properties themselves, and people call them fitness functions.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| mà chúng ta không phải đoán trước mọi tương lai | without us having to predict every future |
| tại đúng điểm thực sự biến động | at exactly the points that really move |
| chúng ta để hở thay vì khoá sớm | we leave them open instead of locking them early |
| phân biệt thật rạch ròi giữa | draw a very sharp line between |
| việc đoán mò tương lai | guessing at the future |
| tại chỗ đã đau thật | where the pain has already been felt |
| cho một nhu cầu chưa tồn tại | for a need that does not exist yet |
| ngay khi biến thiên xuất hiện | as soon as the variation appears |
| test tự động cho chính các thuộc tính kiến trúc | automated tests for the architectural properties themselves |

**Thuật ngữ cần nhớ**

- kiến trúc tiến hoá → **evolutionary architecture**
- dự đoán → **predict**
- khoá sớm → **lock early**
- đoán mò → **guess**
- hàm đo sức khoẻ kiến trúc → **fitness function**

---

## Phần 8 — Kiểm soát code AI, và lý luận ở tầng nguyên tắc

**Tiếng Việt**

Phần cuối cùng và cũng thời sự nhất là kiểm soát chất lượng thiết kế của code do AI sinh ra. Các công cụ như Copilot hay Cursor viết code chạy được rất nhanh, nhưng chúng thường vi phạm thiết kế, ví dụ chúng nhét logic vào controller hoặc chúng import database vào domain. Cách phân vai đúng là AI lo phần cú pháp, còn con người chỉ huy và kiểm tra phần thiết kế. Về công cụ, chúng ta dựa vào lint kiểm tra ranh giới, một checklist gắn vào pull request, và những buổi review tập trung vào ranh giới cùng coupling chứ không sa vào chuyện style vặt. Chúng ta cũng nên viết quy ước kiến trúc ngay trong repo để chính AI đọc được và tuân theo. Câu thần chú của cả mục này là chúng ta hiểu để chỉ huy, chứ chúng ta không học thuộc để gõ.

Điều cuối cùng là lý do vì sao một Tech Lead phải lý luận ở tầng nguyên tắc chứ không phải ở tầng công cụ. Pattern và framework chỉ là phương tiện, và chúng thay đổi theo thời gian, bởi vì thứ đang thịnh hành hôm nay có thể biến mất sau năm năm. Nguyên tắc thì ổn định hơn nhiều, ví dụ coupling, cohesion và SOLID vẫn đúng qua nhiều thế hệ công nghệ. Vì vậy chúng ta biện minh cho quyết định bằng những đánh đổi cụ thể, chứ chúng ta không nói rằng framework bảo thế. Chính điểm này giúp người phỏng vấn phân biệt một ứng viên thuộc tên pattern với một ứng viên hiểu nguyên tắc, bởi vì người hiểu nguyên tắc luôn nói được vì sao và luôn nói được khi nào không nên dùng.

**English (bám cấu trúc tiếng Việt)**

The last part and also the most current one is controlling the design quality of code that AI generates. Tools such as Copilot or Cursor write working code very fast, but they often violate the design, for example they push logic into the controller or they import the database into the domain. The correct division of roles is that AI takes care of the syntax, while the human directs and checks the design. As for tooling, we rely on boundary lint, a checklist attached to the pull request, and reviews that focus on boundaries and coupling rather than sinking into small style matters. We should also write the architectural conventions right inside the repository so that AI itself can read them and follow them. The mantra of this whole subject is that we understand in order to direct, and we do not memorise in order to type.

The last point is the reason why a Tech Lead must reason at the level of principles and not at the level of tools. Patterns and frameworks are only means, and they change over time, because the thing that is popular today may disappear in five years. Principles are far more stable, for example coupling, cohesion and SOLID still hold across several generations of technology. Therefore we justify our decisions with concrete trade-offs, and we do not say that the framework told us so. This very point helps interviewers separate a candidate who has memorised pattern names from a candidate who understands the principles, because the person who understands the principles can always say why and can always say when not to use something.

**Ánh xạ cụm khó**

| Tiếng Việt | Tiếng Anh |
|---|---|
| phần cuối cùng và cũng thời sự nhất | the last part and also the most current one |
| viết code chạy được rất nhanh | write working code very fast |
| chúng nhét logic vào controller | they push logic into the controller |
| cách phân vai đúng là | the correct division of roles is that |
| con người chỉ huy và kiểm tra phần thiết kế | the human directs and checks the design |
| chứ không sa vào chuyện style vặt | rather than sinking into small style matters |
| để chính AI đọc được và tuân theo | so that AI itself can read them and follow them |
| chúng ta hiểu để chỉ huy, chứ chúng ta không học thuộc để gõ | we understand in order to direct, and we do not memorise in order to type |
| thứ đang thịnh hành hôm nay có thể biến mất sau năm năm | the thing that is popular today may disappear in five years |
| chúng ta không nói rằng framework bảo thế | we do not say that the framework told us so |
| một ứng viên thuộc tên pattern | a candidate who has memorised pattern names |

**Thuật ngữ cần nhớ**

- thời sự, đang nóng → **current**
- phân vai → **division of roles**
- danh sách kiểm → **checklist**
- biện minh → **justify**
- thịnh hành → **popular** / **in fashion**

---

## Mô hình ghi nhớ

**Tiếng Việt**

Nguyên tắc là la bàn và nó ổn định, còn pattern cùng framework chỉ là phương tiện và chúng thay được. Một Tech Lead biện minh bằng đánh đổi, enforce bằng lan can tự động chứ không bằng người gác cổng, ghi lại phần "vì sao" bằng ADR, và với AI thì hiểu để chỉ huy chứ không học thuộc để gõ.

**English (bám cấu trúc tiếng Việt)**

Principles are the compass and they are stable, while patterns and frameworks are only means and they are replaceable. A Tech Lead justifies with trade-offs, enforces with automatic guardrails and not with a gatekeeper, records the "why" with ADRs, and with AI understands in order to direct rather than memorising in order to type.

---

## Bảng thuật ngữ tổng hợp

| Tiếng Việt | Tiếng Anh | Ghi chú phát âm |
|---|---|---|
| tầng bao trùm | meta layer | *meta* Anh đọc /ˈmetə/ — "ME-tơ" |
| áp dụng nghiêm, bắt tuân thủ | enforce | trọng âm cuối: en-FORCE |
| vòng phỏng vấn cuối | the final round | |
| ra quyết định | make decisions | *decision* trọng âm âm hai: de-CI-sion, âm giữa là "zh" |
| vấp, hụt | stumble | /ˈstʌmbl/ — "STAM-bồ" |
| tầng gián tiếp | layer of indirection | *indirection* trọng âm áp chót: in-di-REC-tion |
| biến thiên | variation | trọng âm áp chót: va-ri-A-tion |
| quen thuộc với | familiar with | trọng âm âm hai: fa-MI-liar |
| mặc định | default | trọng âm cuối: de-FAULT |
| cho sang, cho oai | to look impressive | *impressive* trọng âm âm hai: im-PRE-ssive |
| làm quá tay | over-engineering | *engineering* trọng âm áp chót: en-gi-NEE-ring |
| làm thiếu | under-engineering | |
| cái cân | the scale | |
| điểm cân bằng | the balance point | *balance* trọng âm đầu: BA-lance |
| năng lực | capability | trọng âm âm ba: ca-pa-BI-li-ty |
| chặn cứng | block flatly | |
| chiều theo, nhượng bộ | give in | |
| động lực | motivation | trọng âm áp chót: mo-ti-VA-tion |
| hướng ở giữa | a middle path | |
| tiến hoá | evolve | trọng âm cuối: e-VOLVE, đuôi **-lv** phải bật |
| nút thắt cổ chai | bottleneck | trọng âm đầu: BO-ttle-neck |
| công cụ hoá | turn into tooling | |
| lan can bảo vệ | guardrail | *guard* — chữ **u** câm, đọc "gaad" |
| người gác cổng | gatekeeper | |
| kèm cặp | mentor | trọng âm đầu: MEN-tor |
| bản ghi quyết định kiến trúc | Architecture Decision Record | *record* danh từ trọng âm đầu: RE-cord; động từ re-CORD |
| bối cảnh | context | trọng âm đầu: CON-text |
| lựa chọn thay thế | alternative | trọng âm âm hai: al-TER-na-tive |
| hệ quả | consequence | trọng âm đầu: CON-se-quence |
| trạng thái | status | Anh đọc /ˈsteɪtəs/ — "STAY-tợs" |
| kiến trúc tiến hoá | evolutionary architecture | *evolutionary* — e-vo-LU-tio-na-ry, đọc chậm từng âm |
| dự đoán | predict | trọng âm cuối: pre-DICT |
| khoá sớm | lock early | |
| đoán mò | guess | /ges/ — chữ **u** câm, đọc là "ghét" |
| hàm đo sức khoẻ kiến trúc | fitness function | |
| thời sự, đang nóng | current | trọng âm đầu: CU-rrent |
| phân vai | division of roles | *division* trọng âm âm hai: di-VI-sion |
| danh sách kiểm | checklist | |
| biện minh | justify | trọng âm đầu: JUS-ti-fy |
| thịnh hành | popular / in fashion | *popular* trọng âm đầu: PO-pu-lar |
| quy ước | convention | trọng âm âm hai: con-VEN-tion |
| cú pháp | syntax | trọng âm đầu: SYN-tax |

---

## Bài nói

> **Cách làm:** ghi âm lại giọng của mình, mỗi câu nói liên tục **60–90 giây**, **không nhìn tài liệu** khi nói. Mỗi bài nói phải dùng ít nhất **6 thuật ngữ** trong bảng ở trên. Nghe lại bản ghi âm một lần, ghi ra hai chỗ mình bị vấp, rồi nói lại lần hai.

1. Explain the general rule you use to decide when not to apply a design pattern, and give one example from your own work where you deliberately kept things simple.
2. Describe the concrete signs of over-engineering and of under-engineering, and say how you find the balance point for a given project.
3. A strong developer on your team wants full DDD, hexagonal and CQRS for a small internal CRUD service. Describe out loud how you would run that conversation without blocking them flatly and without simply giving in.
4. Explain how you would enforce the dependency rule across a team of thirty engineers without becoming the bottleneck yourself.
5. Explain what an ADR is, name its four parts, and say what a new joiner gains from reading one six months later.
6. A colleague says: "The AI-generated code passes all the tests, so let us merge it." Explain why you would push back and exactly what you would look at in the review.
7. Explain why you reason about principles rather than frameworks when you defend an architectural decision, and give one decision you would justify purely in terms of coupling and cohesion.
