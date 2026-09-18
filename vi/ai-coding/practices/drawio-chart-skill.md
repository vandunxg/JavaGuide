---
title: "Skill vẽ draw.io dành riêng cho JavaGuide được open source: tự động tạo sơ đồ kỹ thuật draw.io có thể chỉnh sửa bằng Agent"
description: Chia sẻ tư duy thiết kế, cách cài đặt và quy trình sử dụng Skill drawio-chart, giải thích vì sao bài viết kỹ thuật phù hợp hơn khi giữ lại source file draw.io, cũng như cách để Agent tạo flowchart, architecture diagram và sơ đồ quan hệ module có thể bảo trì.
category: Thực chiến AI Coding
tag:
  - AI Coding
  - Skills
  - draw.io
  - Codex
  - Technical Writing
head:
  - - meta
    - name: keywords
      content: draw.io,drawio-chart,Agent Skills,AI coding,technical article illustrations,Codex,diagrams.net
---

Xin chào, tôi là Tiểu G. Nhiều khi tôi cảm nhận được tác động mà thời đại AI mang lại cho mình từ những việc rất nhỏ.

Trước đây, viết một bài kỹ thuật ít nhất cũng mất một tuần, bài dài hơn thậm chí mất cả tháng. Trong đó, 1/3 thời gian tiêu tốn vào việc minh họa khô khan.

Những độc giả quen thuộc với tôi hẳn biết rằng nhiều hình minh họa trên [JavaGuide](https://mp.weixin.qq.com/s/MP8_Td9h72jAhTntVV4DxQ) được vẽ thủ công bằng draw.io. Mỗi bài viết đều có rất nhiều hình giải thích để hỗ trợ việc hiểu nội dung.

![Các câu hỏi phỏng vấn Java Basics thường gặp](https://oss.javaguide.cn/github/javaguide/intro/java-basic-questions-01-overview.png)

![Tổng hợp các câu hỏi phỏng vấn MySQL thường gặp](https://oss.javaguide.cn/github/javaguide/intro/mysql-questions-01.png)

Nhưng khi bước vào thời đại AI, mọi thứ đã thay đổi hoàn toàn, đặc biệt là việc tạo hình minh họa bằng draw.io.

Năm ngoái, khi Skill còn chưa phổ biến, tôi dùng AI tạo trực tiếp XML tương ứng rồi import vào draw.io.

Ngày nay, nhờ sự ra đời của Skill, việc tạo hình minh họa draw.io đã tự động hóa hơn.

Bài viết này sẽ chia sẻ custom draw.io Skill tôi thường dùng: [`drawio-chart`](https://github.com/Snailclimb/AIGuide/tree/main/skills/drawio-chart), đồng thời trao đổi vì sao tôi chưa hoàn toàn chuyển sang các model tạo ảnh mà vẫn giữ lại source file có thể chỉnh sửa dạng `.drawio`.

## Vì sao chọn draw.io?

Một hình không chỉ cần đẹp khi được tạo ra, mà khả năng chỉnh sửa về sau cũng rất quan trọng.

Sau khi bài kỹ thuật được đăng, tiêu đề, node, mũi tên và thuật ngữ thường xuyên được điều chỉnh. Nếu trong tay chỉ còn một ảnh PNG, hoặc phải tạo lại, hoặc phải sửa cứng trên ảnh, cuối cùng style cũng chưa chắc khớp.

Vì vậy, hiện tại tôi thích giữ lại một source file có thể chỉnh sửa hơn.

Ví dụ, tôi thường lưu riêng các source file `.drawio` trong thư mục tài nguyên ở local. Sau này muốn sửa hình nào thì chỉ cần mở file tương ứng.

![Các source file hình draw.io được lưu lại ở local](https://oss.javaguide.cn/github/javaguide/ai/coding/local-drawio-source-files.png)

Ưu điểm của `draw.io` nằm ở đây: `.drawio` vẫn có thể chỉnh sửa trong diagrams.net hoặc bản desktop của draw.io, đồng thời cũng thuận tiện để export sang PNG, SVG, PDF. Với các hình kỹ thuật như flowchart, architecture diagram và state diagram, source file thường không lớn, đưa vào repository hoặc thư mục tài nguyên cũng không gây áp lực.

Khi export cũng không cần xử lý thêm, trong menu có thể chọn trực tiếp các format phổ biến như PNG, JPEG, WebP, SVG và PDF.

![Lựa chọn format export của draw.io](https://oss.javaguide.cn/github/javaguide/ai/coding/drawio-export-format-options.png)

Khi Skill vừa ra đời, tôi đã sắp xếp quy trình này thành một Skill: [`drawio-chart`](https://github.com/Snailclimb/AIGuide/tree/main/skills/drawio-chart).

Hiện nay, không ít hình minh họa cho các bài viết về AI Coding, Spec Coding và Claude Code mà bạn thấy trên [javaguide.cn](https://javaguide.cn/) về cơ bản đều được thực hiện theo tư duy này:

- Trước tiên để Agent trích xuất cấu trúc, sắp xếp node, tạo `.drawio`, sau đó export hình theo nhu cầu của bài viết;
- Khi cần thể hiện thị giác mạnh hơn, kết hợp thêm GPT-IMG2 để xử lý tiếp.

## Vì sao không chỉ dùng model tạo ảnh?

Trước đây tôi cũng đã thử vài hướng.

[`tt-a1i/archify`](https://github.com/tt-a1i/archify) giống một công cụ dùng ngôn ngữ tự nhiên để tạo technical diagram HTML tự chứa, hỗ trợ chuyển theme và export nhiều format, đồng thời có quy trình validation và rendering.

[`coleam00/excalidraw-diagram-skill`](https://github.com/coleam00/excalidraw-diagram-skill) đi theo style Excalidraw, phù hợp với cảm giác whiteboard và việc trình bày ý tưởng; nó còn dùng Playwright để kiểm tra các vấn đề như chữ bị chồng, mũi tên lệch và khoảng cách mất cân đối.

[`DayuanJiang/next-ai-draw-io`](https://github.com/DayuanJiang/next-ai-draw-io) giống một sản phẩm AI vẽ hình, đưa AI vào draw.io, hỗ trợ Demo online, desktop app, Docker, cài đặt local, MCP Server và cấu hình nhiều model. Khi đó cảm nhận của tôi là nó hơi lag, hiệu suất vẽ cũng không cao. Tuy nhiên hiện dự án vẫn đang được cập nhật, tôi chưa benchmark lại nên không đánh giá performance hiện tại.

Ba dự án này đều có trọng tâm riêng. Archify có cảm giác như thành phẩm mạnh hơn, Excalidraw phù hợp hơn với cách trình bày kiểu whiteboard, còn next-ai-draw-io giống một sản phẩm độc lập hơn.

Nhưng khi áp dụng vào các bài viết kỹ thuật của JavaGuide, những nhu cầu tôi thường gặp hơn là:

- Trong hình có nhiều thuật ngữ tiếng Trung, về sau thường xuyên cần tinh chỉnh.
- Màu sắc, font và style của node trong cùng một nhóm bài viết cần thống nhất hết mức có thể.
- Hình cần chèn được vào Markdown, tài khoản công chúng, web và tài liệu trong repository.
- Sau khi cập nhật bài viết, tốt nhất chỉ cần sửa vài node thay vì làm lại toàn bộ hình.

Trong trường hợp này, source file `.drawio` thuận tiện hơn.

Mở riêng một hệ thống vẽ hình thì hơi nặng đối với việc minh họa bài viết. Coding Agent vốn đã đọc bài, sửa file, chạy command và sắp xếp tài nguyên; để nó tiện thể gọi Skill tạo `.drawio`, sau đó export hàng loạt, sẽ rút ngắn chain đi rất nhiều.

Model tạo ảnh phù hợp để tăng cường thể hiện thị giác, nhưng không thật sự phù hợp cho việc bảo trì lâu dài. Hôm nay sửa một từ, ngày mai thêm một nhánh, rất khó yêu cầu nó chỉ chỉnh chính xác phần nhỏ đó mà vẫn giữ nguyên toàn bộ hình không bị biến dạng.

draw.io không đạt mức “tạo xong là thành phẩm lớn”, nhưng node, connection, container và text đều có thể tiếp tục chỉnh sửa. Với technical diagram, điều này rất có giá trị.

## Quy trình vẽ hình hiện tại của tôi

Quy trình tôi thường dùng hiện nay đại khái như sau:

1. Viết bài trước, hoặc ít nhất sắp xếp lại tuyến chính, flow và quan hệ giữa các concept trong bài.
2. Để Agent xác định những phần nào trong bài đáng vẽ thành hình.
3. Dùng `$drawio-chart` để tạo source file `.drawio`.
4. Khi cần publish thì export PNG / SVG / PDF.
5. Nếu một hình cần thể hiện thị giác mạnh hơn, kết hợp GPT-IMG2 để xử lý tiếp.

Điều tôi quan tâm hơn là tách cấu trúc khỏi phần thể hiện.

`drawio-chart` phụ trách chart có cấu trúc: flow đi như thế nào, module kết nối ra sao, state chuyển đổi thế nào, node nào thuộc cùng một nhóm. GPT-IMG2 phù hợp hơn để xử lý cách thể hiện ở cấp bitmap, ví dụ làm cho một hình giống hình minh họa bài viết hơn và có độ hoàn chỉnh về thị giác hơn.

Cấu trúc vẫn nằm trong `.drawio`, về sau muốn sửa thì có thể sửa.

Bên dưới là quá trình tạo `.drawio` thực tế. Sau khi đọc xong yêu cầu, Agent viết trực tiếp source file, cuối cùng trả về file path và mô tả cấu trúc.

![Codex dùng drawio-chart tạo source file draw.io](https://oss.javaguide.cn/github/javaguide/ai/coding/codex-generate-drawio-source.png)

Sau khi mở, nó vẫn là file draw.io chuẩn. Node, connection và text đều có thể tiếp tục tinh chỉnh thủ công, không bị khóa trong một hình ảnh.

![Cấu trúc SKILL.md được tạo và mở trong draw.io](https://oss.javaguide.cn/github/javaguide/ai/coding/drawio-open-generated-source.png)

Đây là hình gốc chưa được chỉnh sửa sau khi vẽ; có thể thấy đường nối vẫn còn một số chi tiết nhỏ cần điều chỉnh và tối ưu thủ công. Đây cũng là điều khá bình thường.

Tiếp theo hãy xem một số hình được vẽ bằng Skill này.

Flowchart quyết định bảo trì `CLAUDE.md` này là một hình minh họa điển hình thuộc nhóm flow và decision:

![Flowchart quyết định bảo trì CLAUDE.md](https://oss.javaguide.cn/github/javaguide/ai/coding/claudecode/claude-md-best-practices-maintenance-flow.png)

Nội dung cộng tác Multi-Agent nếu chỉ viết bằng text, người đọc rất dễ xem thành một nhóm tên role. Khi vẽ thành pipeline, trách nhiệm của từng Agent và cách thông tin lưu chuyển sẽ trực quan hơn nhiều.

![Pipeline cộng tác ba Agent Multi-Agent](https://oss.javaguide.cn/github/javaguide/ai/coding/spec-coding-multi-agent-pipeline.png)

Các bài viết về Spec Coding cũng tương tự. Nội dung này nói về một workflow chứ không phải một concept độc lập. Khi nối requirement, Spec, implementation và verification trong hình, người đọc có thể nắm tổng thể trước rồi quay lại nội dung bài viết để xem chi tiết.

![Pipeline lập trình theo Spec Coding](https://oss.javaguide.cn/github/javaguide/ai/coding/spec-coding-pipeline-flow.png)

Ngoài ra còn có hình về strategy quản lý Spec, phần giải thích bằng text khá vòng vèo. Những từ như filtering theo tầng, recall chính xác và kiểm soát context đặt trong một hình lại dễ hiểu hơn.

![Strategy quản lý Spec: filtering theo tầng + recall chính xác](https://oss.javaguide.cn/github/javaguide/ai/coding/spec-coding-spec-management-strategy.png)

Không nhất thiết hình nào cũng phải nhờ AI hoàn thành một lần. Phần tiết kiệm thời gian chủ yếu nằm ở nửa đầu: Agent dựng cấu trúc trước, sau đó con người chỉnh theo ngữ cảnh bài viết.

Khi hình minh họa bài kỹ thuật tách rời khỏi bài viết thì sẽ rất phiền. Nếu node, tiêu đề và mũi tên trong hình không khớp với nội dung bài viết, dù đẹp đến đâu cũng vô ích.

## Skill `drawio-chart` làm gì?

Trước đây tôi từng nói trong [《Agent Skills là gì? Khác Prompt và MCP chính xác ở đâu?》](https://javaguide.cn/ai/agent/skills.html) rằng Skill giống một hướng dẫn nhiệm vụ được load on demand hơn.

Nó không phụ trách phát minh một tool mới, cũng không tương đương với Function Calling hay MCP. Nó giải quyết các vấn đề: một loại task nên thực hiện thế nào, khi nào thực hiện, bước nào không được bỏ qua và cần những tài liệu tham khảo nào.

`drawio-chart` chính là việc kết tinh “vẽ hình draw.io cho bài viết kỹ thuật” thành một Skill.

File chính của nó là [`SKILL.md`](https://github.com/Snailclimb/AIGuide/blob/main/skills/drawio-chart/SKILL.md), bên trong chỉ chứa một số nhóm thông tin:

- Khi nào nên dùng, ví dụ draw.io, diagrams.net, flowchart, architecture diagram, sequence diagram, ER diagram, state machine diagram và mind map.
- Khi nào không nên dùng, ví dụ người dùng chỉ muốn code Mermaid, hoặc muốn bitmap illustration, poster hay hình minh họa style whiteboard.
- Trước khi vẽ cần thu thập thông tin gì, ví dụ theme, loại chart, node chính, quan hệ giữa node và có export hay không.
- Khi tạo `.drawio` cần đi theo thứ tự nào, ví dụ tiêu đề, container, node cốt lõi, connection và label.
- Trước khi bàn giao cần kiểm tra gì, ví dụ node đã đầy đủ chưa, quan hệ đã vẽ đúng chưa, label của connection có quá dài không và file name có đúng quy ước không.

Những nội dung chi tiết hơn không được nhồi hết vào `SKILL.md`.

Nó được tách thành một số file trong `references/`:

| File                  | Chứa gì                                                                       |
| --------------------- | ----------------------------------------------------------------------------- |
| `style-spec.md`       | Màu sắc, font, ngữ nghĩa node và style connection                             |
| `xml-and-layout.md`   | Cấu trúc XML draw.io, template node, template connection và gợi ý layout      |
| `export-and-files.md` | Command export PNG / SVG / PDF, cách đặt tên file và quy tắc bàn giao         |
| `use-cases.md`        | Prompt thường gặp, pattern minh họa bài viết nhiều hình và gợi ý đặt tên page |

Cách tách này nhất quán với tư duy thiết kế của Skills.

Không nên viết file chính thành một README quá dài. Agent trước tiên biết Skill này có thể làm gì; sau khi khớp task, nó mới đọc file tham khảo tương ứng theo nhu cầu hiện tại.

Ví dụ, nếu chỉ tạo một flowchart thì không nhất thiết phải đọc đầy đủ quy tắc export. Nếu người dùng yêu cầu rõ export PNG, chỉ cần xem `export-and-files.md`. Khi cần kiểm soát style thì đọc thêm `style-spec.md`.

Nhờ vậy context không bị nhồi đầy bởi các chi tiết không liên quan, việc thực thi cũng ổn định hơn.

## Tôi đã thêm những ràng buộc nào?

Việc vẽ hình rất dễ mất kiểm soát.

Khi Agent tạo hình, có một số vấn đề thường gặp: text của node quá dài, label của connection đè lên mũi tên; màu sắc trong một hình được dùng lộn xộn, trông như tô màu ngẫu nhiên; mỗi connection trong flowchart đều kèm phần giải thích dài; XML bị trộn các HTML tag, về sau dễ gây vấn đề khi render hoặc chỉnh sửa.

Vì vậy, `drawio-chart` chứa khá nhiều ràng buộc cụ thể.

Ví dụ về style, Agent không cần tùy tiện chọn màu tại chỗ. Quy tắc sẽ phân màu theo ngữ nghĩa: entry, business service, infrastructure, client, external dependency, database, cache, message queue và exception state đều có color value tương ứng. Nhờ vậy, khi đặt cùng một nhóm hình cạnh nhau, người đọc không phải hiểu lại màu sắc từ đầu ở mỗi hình.

Về text, nó yêu cầu `mxCell.value` mặc định sử dụng plain text; khi cần xuống dòng thì dùng XML line break entity, không chèn các HTML tag như `<br>`, `<b>` vào node value.

Label của connection cũng phải ngắn. Với connection ngắn, không nên đặt phần giải thích dài; nội dung có thể viết vào node thì viết vào node, có thể đặt ở chú thích bên cạnh thì đặt ở chú thích bên cạnh. Phần lớn cảm giác lộn xộn trong technical diagram bắt đầu từ việc “muốn giải thích một câu trên mọi connection”.

Về export, mặc định trước tiên giữ lại `.drawio`, sau đó export khi cần. Kể cả export thất bại cũng không được xóa source file.

## Cài đặt và sử dụng thế nào?

Tôi đã đưa Skill này vào repository [AIGuide: hướng dẫn phát triển ứng dụng AI, thực chiến AI Coding và phỏng vấn](https://github.com/Snailclimb/AIGuide):

- Entry point của repository: **<https://github.com/Snailclimb/AIGuide/tree/main/skills>**
- Thư mục Skill: **<https://github.com/Snailclimb/AIGuide/tree/main/skills/drawio-chart>**

Nếu bạn đang dùng hệ sinh thái `npx skills`, có thể cài đặt như sau:

```bash
npx skills add Snailclimb/AIGuide/skills/drawio-chart
```

Nếu chỉ muốn cài cho Codex, cũng có thể chỉ định trực tiếp agent:

```bash
npx -y skills add Snailclimb/AIGuide/skills/drawio-chart --agent codex --yes
```

Nếu chủ yếu sử dụng trong Codex, cũng có thể cài đặt từ GitHub:

```bash
python3 ~/.codex/skills/.system/skill-installer/scripts/install-skill-from-github.py \
  --repo Snailclimb/AIGuide \
  --path skills/drawio-chart
```

Cần lưu ý rằng trong các version như `skills@1.5.15`, `npx skills add Snailclimb/AIGuide --path skills/drawio-chart` có thể vẫn quét trước toàn bộ Skill trong repository. Cách viết ổn định hơn là ghi trực tiếp `skills/drawio-chart` vào source, tức là `Snailclimb/AIGuide/skills/drawio-chart` như ở trên.

Khi cài đặt, bạn sẽ thấy nó nhận diện nguồn repository, tìm thấy `drawio-chart`, sau đó cho bạn chọn agent và phạm vi cài đặt.

![Dùng Skills CLI để cài đặt drawio-chart](https://oss.javaguide.cn/github/javaguide/ai/coding/skills-cli-install-drawio-chart.png)

Đôi khi Codex không lập tức rescan Skill mới cài, thường tôi sẽ restart một lần rồi sử dụng.

Khi mới bắt đầu, đừng để Agent tự đoán. Hãy gọi đích danh `$drawio-chart`, nó sẽ dễ đi vào workflow đúng hơn. Ví dụ khi vẽ flowchart đăng nhập, tôi sẽ viết như sau:

```text
Sử dụng $drawio-chart để vẽ flowchart đăng nhập của user, bao gồm:
Nhập username và password -> Xác thực username và password -> Đăng nhập thành công thì chuyển đến trang chủ -> Thất bại thì hiển thị lỗi và cho phép retry.
Yêu cầu export PNG.
```

Khi minh họa cho cả bài viết, tôi thường không quy định trước “phải vẽ bao nhiêu hình”. Cách thuận tiện hơn là giao path của bài viết cho nó, để nó chọn ra các cấu trúc thực sự đáng vẽ từ bài viết trước, sau đó bổ sung yêu cầu về format:

```text
Sử dụng $drawio-chart để đọc bài viết này và tạo một số hình minh họa kỹ thuật phù hợp cho bài viết.
Đặt tất cả hình minh họa vào cùng một file draw.io, mỗi hình con là một diagram page.
Giữ main file name giống với article file name, page name sử dụng chữ thường và dấu gạch ngang.
Style hình minh họa tuân theo quy ước thống nhất của drawio-chart.
```

Ở đây đừng chỉ ném cho nó một câu “hãy vẽ cho tôi một architecture diagram”. Architecture diagram của microservice ít nhất phải nói rõ có những service, storage và external dependency nào, request đi vào từ đâu, call nào là synchronous và chain nào đi qua asynchronous message.

Thứ bạn cung cấp cho nó là một yêu cầu chart rõ ràng thì output tạo ra mới gần với bản dùng được hơn. Chỉ đưa một tiêu đề quá mơ hồ thì phần lớn sau đó vẫn phải sửa thủ công.

## Những hình nào phù hợp với draw.io?

Hiện tại tôi chủ yếu giao ba loại hình cho `drawio-chart`.

Một loại là flowchart trong bài viết. Ví dụ Spec Coding, strategy bảo trì CLAUDE.md và pipeline cộng tác Agent đều có step và branch rõ ràng, draw.io rất phù hợp để tinh chỉnh về sau.

Một loại là architecture diagram và sơ đồ quan hệ module. Điều phiền nhất ở đây là style không thống nhất: trong cùng một bài viết, service node một màu, storage node một màu, external dependency lại được phân biệt riêng, người đọc sẽ dễ theo dõi hơn nhiều.

Một loại khác là những hình thường xuyên phải sửa. Sau khi bài viết được publish, tiêu đề, tên node, hướng mũi tên và kích thước export đều có thể điều chỉnh. Chỉ cần `.drawio` còn đó thì việc sửa không còn phiền.

Tôi không thường dùng nó để làm cover, poster hay product mood illustration; những việc này phù hợp hơn với GPT-IMG2. Nếu muốn style whiteboard vẽ tay, bộ Excalidraw sẽ phù hợp hơn; nếu muốn theme switching, HTML tự chứa và web interaction, Archify cũng đáng xem.
