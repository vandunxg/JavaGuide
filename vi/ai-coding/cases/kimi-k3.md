---
title: "Thực chiến Kimi K3: dự án full-stack, cải tạo dự án Java và Demo game 3A"
description: "Thông qua ba case thực tế gồm hệ thống theo dõi chủ đề nóng, cải tạo dự án Java và Demo game hành động góc nhìn người thứ ba, kiểm chứng thực tế khả năng của Kimi K3 trong lập trình Agent dài hạn, hiểu đa mô thức và các tác vụ engineering phức tạp."
category: AI Coding thực chiến
head:
  - - meta
    - name: keywords
      content: Kimi K3,Kimi Code,AI Coding,Agent Coding,phát triển full-stack,cải tạo dự án Java,multimodal,tác vụ dài,phát triển game
---

Xin chào, mình là G nhỏ. Kimi K3 đã chính thức ra mắt vào thứ Sáu tuần trước!

Hai ngày qua, câu hỏi được hỏi nhiều nhất về cơ bản đều là một: năng lực của K3 thực sự thế nào? Cảm giác viết code ra sao?

Trong và ngoài nước đã có nhiều chuyên gia mang K3 đi so sánh với các model Coding hàng đầu, phản hồi đều rất tốt. Dữ liệu cũng không biết nói dối, lượng đăng ký và sử dụng K3 mấy ngày nay tăng vọt, năng lực tính toán gần như không chịu nổi.

Mình vẫn muốn xem biểu hiện của nó trong dự án thực tế hơn, vì mình cho rằng đó mới là điều thiết thực nhất.

Vì vậy, lần này mình chuẩn bị ba case rất điển hình: một dự án full-stack, một dự án Java hiện có cần cải tạo, và thêm một Demo game được làm từ đầu.

Lần này K3 mang đến: **2.8T tham số, context 1M, multimodal nguyên bản và tối ưu kiến trúc hướng tới lập trình Agent dài hạn.**

Có thể thấy cốt lõi của nó là giúp Agent liên tục đọc code, xem screenshot, chạy tool và sửa vấn đề trong các tác vụ dài hơn.

![Thông tin ra mắt chính thức của Kimi K3](https://oss.javaguide.cn/github/javaguide/ai/coding/k3/k3-official-announcement.png)

Bài này vẫn theo cách cũ của mình: trước tiên xem cách tích hợp, sau đó xem biểu hiện thực tế của nó trong một số tác vụ engineering, cuối cùng quay lại trao đổi về những điểm nổi bật trong lần cập nhật này của K3.

## Tích hợp Kimi K3

K3 hỗ trợ gần như mọi Coding Agent phổ biến và các framework Agent tổng quát, chẳng hạn Claude Code, Roo Code, OpenCode, OMP, OpenClaw, Hermes.

Trước tiên, mình lấy Kimi Code CLI chính thức của Kimi làm ví dụ để giới thiệu cách tích hợp ngắn nhất. Nếu bạn đã dùng Claude Code, Roo Code, OpenCode hoặc CC Switch thì vẫn có thể tiếp tục dùng toolchain cũ.

### Tích hợp Kimi Code CLI

Đây là Coding Agent trên terminal chính thức của Kimi, tương tự Claude Code.

Sau khi cài đặt thành công, bạn thực thi lệnh `kimi` trong thư mục dự án, rồi có thể yêu cầu nó đọc code, sửa code, chạy lệnh, giải thích lỗi và tạo commit message.

Lệnh cài đặt cho macOS và Linux như sau:

```bash
curl -fsSL https://code.kimi.com/kimi-code/install.sh | bash
```

Windows PowerShell dùng:

```powershell
irm https://code.kimi.com/kimi-code/install.ps1 | iex
```

Sau khi cài đặt xong, có thể dùng `kimi --version` để xem version:

![Cài đặt Kimi Code CLI và xem version sau khi hoàn tất](https://oss.javaguide.cn/github/javaguide/ai/coding/k3/k3-kimi-cli-install.png)

Sau khi cài xong, chuyển đến thư mục dự án cần test và chạy:

```bash
kimi
```

Lần khởi động đầu tiên sẽ yêu cầu đăng nhập tài khoản Kimi. Sau khi vào Kimi Code CLI, nhập `/model` và chọn `k3`.

![Chọn k3 trong danh sách model sau khi khởi động bằng kimi -y](https://oss.javaguide.cn/github/javaguide/ai/coding/k3/k3-kimi-y-model-list.png)

Với cá nhân mình, trước khi dùng chính thức, mình thường thực hiện một bài test nhỏ chỉ đọc:

```text
Đọc cấu trúc thư mục và code cốt lõi của dự án hiện tại, nêu trách nhiệm của từng module. Trước tiên không sửa file và không thực thi lệnh có side effect.
```

![Kimi đọc trước cấu trúc thư mục và code cốt lõi của dự án Java](https://oss.javaguide.cn/github/javaguide/ai/coding/k3/k3-case2-read-java-project.png)

### Coding Agent hiện có vẫn tiếp tục dùng được

Nếu đã quen dùng Claude Code, Roo Code hoặc OpenCode thì không cần dựng lại workflow vì K3. Sau khi tạo Kimi Code API Key, có thể tích hợp K3 qua interface tương thích.

Các framework Agent tổng quát như OpenClaw, Hermes cũng có thể gọi Kimi Code. K3 phụ trách model inference, còn cách đọc file, gọi tool và quản lý quyền cụ thể vẫn do framework Agent bên ngoài xử lý.

Lấy CC Switch làm ví dụ, khi thêm provider có thể chọn trực tiếp Kimi For Coding:

![Chọn Kimi For Coding khi thêm provider](https://oss.javaguide.cn/github/javaguide/ai/coding/k3/k3-provider-select.png)

Sau đó nhập API Key và request URL, giữ API format mặc định:

![Cấu hình API Key và request URL của Kimi For Coding](https://oss.javaguide.cn/github/javaguide/ai/coding/k3/k3-provider-config.png)

Nhấn lấy danh sách model, ánh xạ model role vào `k3`:

![Chọn k3 khi ánh xạ model Kimi For Coding](https://oss.javaguide.cn/github/javaguide/ai/coding/k3/k3-provider-model-mapping.png)

Ở đây không cần bật route bổ sung:

![Chọn trực tiếp Kimi For Coding trong CC Switch](https://oss.javaguide.cn/github/javaguide/ai/coding/k3/k3-cc-switch-direct-provider.png)

Test thực tế:

![Tích hợp Claude Code với Kimi](https://oss.javaguide.cn/github/javaguide/ai/coding/k3/k3-claude-code-read-project.png)

## Case một: Xây dựng hệ thống theo dõi chủ đề nóng

Trong case đầu tiên, mình yêu cầu K3 xây dựng một **hệ thống theo dõi chủ đề nóng**, tên dự án là **HotPulse**.

Lần này mình không chỉ yêu cầu nó làm một frontend đẹp mắt. Frontend đương nhiên phải xem, nhưng mình muốn biết hơn liệu nó có thể kết nối backend chain, database, queue, test và error recovery hay không.

Mục tiêu nghiệm thu lần này là một MVP có thể chạy: người dùng tạo keyword Monitor, hệ thống định kỳ hoặc thủ công fetch Hacker News / RSS, lưu nội dung gốc thành Observation, sau đó qua giai đoạn analyze để tạo Entry, cuối cùng thực hiện Notify delivery. Frontend phải xem được trạng thái Feed, Monitor Run, Delivery, đồng thời hỗ trợ đọc, favorite và archive.

Trong system design proposal gửi cho K3, mình đặt ra các ràng buộc chính: backend dùng Hono, SQLite, Drizzle; frontend dùng React 19, React Router, TanStack Query, thêm Socket.IO. Test không được phụ thuộc vào public network thực tế, notification dùng fake webhook server, database dùng SQLite tạm thời.

Để bỏ qua bước phê duyệt thủ công khi ghi file và thực thi lệnh, có thể dùng lệnh `kimi -y`.

Việc thúc đẩy task dùng lệnh `/goal`. Để xem năng lực của model trong tác vụ dài, mình không bật bất kỳ Skill nào và cũng không cung cấp plugin chuyên dụng bổ sung cho dự án.

![Dùng goal trong Kimi Code CLI để thúc đẩy task HotPulse](https://oss.javaguide.cn/github/javaguide/ai/coding/k3/k3-hotpulse-goal-active.png)

Có một chi tiết nhỏ: con số hơn 8 giờ xuất hiện trong ảnh không có nghĩa model thực sự làm việc liên tục lâu như vậy. Mình bắt đầu chạy vào khoảng 12 giờ đêm hôm trước, giữa chừng bị kẹt nên để đó, sáng tiếp tục thúc đẩy. Thời gian hoàn thành thực tế có hiệu quả khoảng 1 giờ.

### Trước tiên nó hoàn thiện backend chain

Thứ tự thực hiện của K3 khá ổn: trước tiên làm shared contract, Zod schema và unit test, sau đó bổ sung cấu hình server, database, queue, domain service, cuối cùng kết nối fetch, analyze, notification, scheduling định kỳ, backend Worker và maintenance task.

Giữa chừng cũng không phải lúc nào mọi thứ đều thuận lợi. K3 tự chạy TypeScript check và test, sau khi thấy lỗi thì quay lại sửa helper, handler và test.

![Kimi sửa test server và lỗi TypeScript](https://oss.javaguide.cn/github/javaguide/ai/coding/k3/k3-hotpulse-server-tests.png)

### Làm frontend sau khi backend hoàn toàn xanh

Phía server giữa chừng chạy `94/94` hoàn toàn xanh trước, sau đó K3 mới bắt đầu xử lý phía client.

![Chuyển sang frontend SPA sau khi test server pass](https://oss.javaguide.cn/github/javaguide/ai/coding/k3/k3-hotpulse-client-stage.png)

Phần frontend bổ sung các page Monitor, Feed, Targets, Deliveries và System.

Dưới đây là báo cáo bàn giao cuối cùng của HotPulse MVP:

![Báo cáo bàn giao cuối cùng của HotPulse MVP](https://oss.javaguide.cn/github/javaguide/ai/coding/k3/k3-hotpulse-goal-complete.png)

Phạm vi chức năng bàn giao cuối cùng về cơ bản bao phủ các điều kiện nghiệm thu MVP mình đưa ra.

Tuy nhiên, bản đầu tiên chạy được nhưng thực sự khá thô, chỉ có thể xem là phiên bản MVP.

Trang chỉnh sửa Monitor có thể cấu hình tên, keyword, khoảng thời gian chạy, trạng thái bật và nguồn:

![Trang chỉnh sửa Monitor của HotPulse](https://oss.javaguide.cn/github/javaguide/ai/coding/k3/k3-hotpulse-monitor-edit.png)

Trang Feed có thể hiển thị Entry sau khi fetch và analyze, đồng thời hỗ trợ lọc theo status, Monitor, thứ tự và favorite:

![Trang Feed của HotPulse](https://oss.javaguide.cn/github/javaguide/ai/coding/k3/k3-hotpulse-feed.png)

### Tiếp tục tối ưu

Sau đó, mình lại yêu cầu Kimi tiếp tục tối ưu vài vòng.

![Báo cáo cuối cùng sau khi HotPulse MVP được tối ưu thêm](https://oss.javaguide.cn/github/javaguide/ai/coding/k3/k3-hotpulse-optimization-report.png)

Vòng này chủ yếu đẩy những chỗ «dùng được» của bản đầu tiên thêm một bước theo hướng sản phẩm. Chẳng hạn như trang chủ mặc định, hướng dẫn cho người mới, phản hồi khi chạy thủ công, button pending, chống submit trùng, toast, dialog xác nhận xóa và copy tiếng Trung thống nhất hơn.

Tiếp đó, mình còn yêu cầu nó thực hiện **mở rộng đa nguồn**.

Lần này không chỉ đơn thuần sửa giao diện. HotPulse không còn chỉ xem Hacker News / RSS, mà đưa RSS, Webpage, GitHub và Twitter vào cùng một pipeline scrape → analyze → notify.

![Hoàn tất mở rộng đa nguồn HotPulse V1.1 và vượt qua kiểm chứng](https://oss.javaguide.cn/github/javaguide/ai/coding/k3/k3-hotpulse-v11-multisource-complete.png)

Điểm thú vị nhất ở đây không phải là viết thêm vài fetcher, mà là K3 thống nhất các nguồn này vào intermediate layer `source_items`, dùng `(monitorId, externalId)` làm unique key, sau đó dùng `contentFingerprint` để phát hiện thay đổi nội dung.

Với một hệ thống monitoring thực sự, điểm khó là sau khi một nguồn bị lỗi, cấu trúc page thay đổi hoặc interface chậm đi, toàn bộ chain có thể duy trì ổn định hay không, có thể tự động degrade và tiếp tục chạy hay không, đồng thời có thể chỉ ra vị trí vấn đề hay không.

Đây là hiệu quả hiện tại, đã gần với hình thái sản phẩm.

Địa chỉ video Demo: <https://www.zhihu.com/pin/2062834508802569097>

Điểm giá trị nhất của case này là **K3 có thể liên tục nắm chắc mục tiêu trong một task tương đối dài: đọc yêu cầu, tách module, sửa code, chạy test, sửa lỗi, cuối cùng bàn giao kết quả có thể kiểm chứng.**

## Case hai: Cải tạo dự án hiện có

Trong case thứ hai, mình chuẩn bị hai task nhỏ để sửa vấn đề hoặc thêm chức năng trên dự án hiện có.

Loại task này không giống MVP full-stack đầu tiên. Trọng tâm là trước tiên đọc hiểu code hiện có, sau đó lần theo hiện tượng để tìm call chain, data source và chi tiết implementation. Với Coding Agent, loại task này gần với phát triển hằng ngày hơn: phát hiện vấn đề trên production hoặc local, định vị rõ, sửa trong phạm vi nhỏ nhất rồi chạy kiểm chứng.

Ở đây, mình lấy project thực chiến phân tích cổ phiếu multi-agent tiếp theo của Planet làm ví dụ; trong thời gian này tutorial vẫn tiếp tục được hoàn thiện và bổ sung.

Trước tiên là task đầu tiên: sửa lỗi ký tự bị lỗi khi tìm kiếm cổ phiếu.

### Sửa lỗi ký tự khi tìm kiếm cổ phiếu

Hiện tượng vấn đề nhìn qua là thấy ngay: toàn bộ tên tiếng Trung trong kết quả tìm kiếm cổ phiếu đều biến thành ký tự lỗi, chỉ còn mã cổ phiếu là xem được.

![Kết quả tìm kiếm cổ phiếu xuất hiện ký tự lỗi](https://oss.javaguide.cn/github/javaguide/ai/coding/k3/k3-case2-stock-garbled-before.png)

Sau khi đưa screenshot và hiện tượng cho K3, trước tiên nó lần theo entry tìm kiếm để tìm `marketDataService.searchStock`, rồi tiếp tục truy đến logic fallback của data source.

Cuối cùng vấn đề nằm trên chain `HttpBodyFetcher` / `HttpUtils`: search API `suggest3.sinajs.cn` và market API `hq.sinajs.cn` của Sina trả về encoding GBK, trong khi `SinaDataSource` của project mặc định dùng `HttpUtils.get`, cố định decode theo UTF-8, khiến tên tiếng Trung bị decode sai tại đây.

Cốt lõi của việc sửa là để data source của Sina decode theo GBK, đồng thời bổ sung request method hỗ trợ `Referer` request header.

![Kimi định vị và sửa lỗi ký tự khi tìm kiếm cổ phiếu](https://oss.javaguide.cn/github/javaguide/ai/coding/k3/k3-case2-stock-garbled-fixed.png)

Sau khi sửa, K3 chạy `mvn -pl stock-crawler -am install`, khởi động lại backend và test thực tế `GET /api/market/search?keyword=开` (từ khóa `开` được nhập nguyên văn), xác nhận response trả về các tên tiếng Trung bình thường như “Thần Khai Cổ phần, Khai Lập Y tế, Kinh Vĩ Huy Khai, Khai Khai B share, Khai Phát Công nghệ”.

Task nhỏ này nhìn qua không lớn, nhưng thể hiện thói quen engineering của model rõ hơn việc “hãy viết cho tôi một utility class”: nó lần theo từ entry, data source, HTTP decode utility đến encoding của external API, cuối cùng cũng không mở rộng sang các module không liên quan.

### Phát triển bảng điều khiển lợi nhuận danh mục

Task thứ hai là thêm một **bảng điều khiển lợi nhuận danh mục** vào project cổ phiếu này.

Nó cần dựa trên giá vốn, khối lượng nắm giữ của các cổ phiếu đã có trong danh sách theo dõi và market data realtime để tính tổng market value, floating P&L, P&L trong ngày, tỷ trọng nắm giữ lớn nhất và các thông tin khác. Đồng thời, style phải thống nhất với phần hiện có.

![K3 nhận task phát triển bảng điều khiển lợi nhuận danh mục](https://oss.javaguide.cn/github/javaguide/ai/coding/k3/k3-case2-portfolio-task.png)

Yêu cầu này nhìn như một page, nhưng thực tế phải đồng thời hoàn thiện backend calculation, interface layering, frontend display và test.

K3 trước tiên đọc các field của cổ phiếu theo dõi, market API và layering hiện có của project rồi mới bắt đầu implementation. Tính toán tiền tệ thống nhất dùng `BigDecimal`; khi thiếu market data thì cho từng cổ phiếu degrade, tránh một exception làm hỏng cả page.

![K3 hoàn tất bảng điều khiển lợi nhuận danh mục và đưa ra kết quả kiểm chứng](https://oss.javaguide.cn/github/javaguide/ai/coding/k3/k3-case2-portfolio-complete.png)

Cuối cùng bàn giao một dashboard có thể truy cập trực tiếp: phía trên là các summary card như tổng market value, tổng cost, floating P&L; phía dưới có thể xem cost, current price, position và profit của từng cổ phiếu. Backend API, frontend route và test cũng được bổ sung cùng lúc.

![Trang cuối cùng của bảng điều khiển lợi nhuận danh mục](https://oss.javaguide.cn/github/javaguide/ai/coding/k3/k3-case2-portfolio-dashboard.png)

Đến bước này thì thực ra đã có thể dùng được.

Nhưng sau đó mình nghĩ lại, trong project thực tế, nhiều vấn đề không đơn giản là «có page hay chưa», mà là business definition có nhất quán không, exception có được xử lý không, metric có thể drill down tiếp không.

Vì vậy, mình lại yêu cầu K3 làm một bản V2 trên nền tảng dashboard này.

Phiên bản này tiếp tục bổ sung business definition và risk metric. Chẳng hạn sửa vấn đề business definition tổng hợp không nhất quán khi một phần market data không khả dụng, thêm ranking đóng góp lợi nhuận, tỷ lệ ba position lớn nhất, HHI concentration, annualized volatility và maximum drawdown.

![K3 tiếp tục tối ưu bảng điều khiển lợi nhuận danh mục V2](https://oss.javaguide.cn/github/javaguide/ai/coding/k3/k3-case2-portfolio-v2-task.png)

Điểm mình thấy khá thú vị ở đây là nó không trực tiếp chất các field lên frontend, mà trước tiên quay lại backend để xác nhận field của K-line service, căn chỉnh trading date, cũng như cách degrade khi cổ phiếu tạm dừng giao dịch hoặc dữ liệu không đủ, rồi đặt calculation logic vào `PortfolioRiskCalculator` độc lập.

Đây rất giống nhịp làm việc của một engineer bình thường.

![K3 hoàn tất bảng điều khiển lợi nhuận danh mục V2 và vượt qua kiểm chứng](https://oss.javaguide.cn/github/javaguide/ai/coding/k3/k3-case2-portfolio-v2-complete.png)

Cuối cùng, trong page V2 vẫn có chi tiết position ban đầu, bên dưới bổ sung tổng quan đóng góp lợi nhuận và rủi ro.

Phần đóng góp lợi nhuận cho biết cổ phiếu nào thực sự đóng góp vào P&L của portfolio; phần tổng quan rủi ro cũng hiển thị annualized volatility, maximum drawdown, VaR, HHI concentration và tỷ lệ ba position lớn nhất.

![Trang bảng điều khiển lợi nhuận danh mục V2](https://oss.javaguide.cn/github/javaguide/ai/coding/k3/k3-case2-portfolio-v2-dashboard.png)

Trong case này, nó có thể kết nối hoàn chỉnh chain trong các ràng buộc của project hiện có: trước tiên reuse data, sau đó bổ sung calculation và API, cuối cùng chạy test và kiểm tra page. Quan trọng hơn, khi tiếp tục tối ưu ở vòng hai, nó vẫn có thể lần theo business definition, đưa từ «có thể hiển thị» đến «business definition của data đáng tin cậy hơn».

Ngoài ra, sau khi hoàn thành, mình còn dùng GPT 5.6 Sol để review code. Nhìn chung logic implementation và code quality đều không có vấn đề, có thể commit trực tiếp.

## Case ba: Làm một Demo game có chất lượng 3A từ đầu

Ban đầu mình thực ra đã phân vân về case thứ ba.

Phía trước đã có một dự án full-stack và một dự án Java hiện có được cải tạo. Nếu thêm một bug fix thông thường nữa thì thông tin sẽ hơi trùng lặp.

Sau đó mình quyết định đổi sang một thứ trực quan hơn.

Làm game.

Hướng này có phần khác biệt. Hệ thống business chú trọng layering, API, test và business definition; game lại chú trọng một nhóm năng lực khác: hình ảnh, physics, cảm giác input, camera, combat feedback, sound effect, UI, performance, và quan trọng nhất là có thực sự chơi được hay không.

Hơn nữa, năng lực làm game của K3 thực sự khá khó tin.

Mình cũng thấy nhiều feedback tương tự trong cộng đồng quốc tế. Có người nói thẳng rằng K3 rất mạnh khi làm mini game bằng một câu prompt, thậm chí còn dùng nó để tái tạo một Demo 3D theo phong cách 《Minecraft》.

![Feedback của cộng đồng quốc tế về năng lực làm game của K3](https://oss.javaguide.cn/github/javaguide/ai/coding/k3/k3-game-community-feedback.png)

Vậy mình nghĩ, được, không chỉ xem người khác chơi.

Mình cũng tự thử.

Prompt của mình rất đơn giản: yêu cầu nó trong thư mục trống hiện tại phát triển từ đầu một Demo game hành động khoa học viễn tưởng góc nhìn người thứ ba có chất lượng 3A. Bối cảnh là một căn cứ công nghiệp tương lai bị bỏ hoang. Người chơi phải tiến vào căn cứ, đánh bại kẻ địch trên đường, lấy lõi năng lượng, cuối cùng đánh bại mechanical Boss rồi rút lui.

Về kỹ thuật, mình yêu cầu nó dùng Vite, TypeScript, Three.js và Rapier. Không tự viết physics engine; việc di chuyển nhân vật, collision và gravity đều giao cho Rapier. Game bắt buộc phải có movement, sprint, dodge, aim, shooting, hit reaction, death restart, enemy thường, elite enemy, multi-phase Boss, complete quest loop, PBR material, dynamic lighting, shadow, fog, particle, Bloom, hit feedback và camera shake.

![Yêu cầu K3 phát triển Demo game hành động góc nhìn người thứ ba có chất lượng 3A từ đầu](https://oss.javaguide.cn/github/javaguide/ai/coding/k3/k3-game-steel-haven-prompt.jpg)

Nói thật, khi viết prompt này, kỳ vọng trong lòng mình cũng không quá cao.

Vì game không phải một webpage thông thường.

Một button trên webpage lệch một chút vẫn có thể chấp nhận, nhưng chỉ cần camera, collision, shooting, enemy AI, Boss phase, hiệu ứng hoặc sound effect của game có một khâu không đúng là người chơi cảm nhận được ngay.

K3 trước tiên đưa ra game design và technical proposal. Project có tên 《STEEL HAVEN · Steel Sanctuary》, flow hoàn chỉnh gồm landing platform, đột phá cổng căn cứ, elite guard ở central courtyard, lấy lõi năng lượng, đánh thức WARDEN-9 Boss, Boss battle ba phase, cố thủ tại extraction point, sau khi chết khởi động lại từ checkpoint.

![Game design và cấu trúc thư mục do K3 đưa ra](https://oss.javaguide.cn/github/javaguide/ai/coding/k3/k3-game-steel-haven-design.jpg)

Sau đó nó bắt đầu viết project trực tiếp.

Sau khi bàn giao bước đầu, sản phẩm đã không còn ở mức «ghép vài block rồi nói đây là game» nữa.

Nó thực sự làm ra game loop: third-person character control, di chuyển WASD, sprint bằng Shift, dodge bằng Space, aim bằng chuột phải, shooting bằng chuột trái, reload bằng R, interact bằng E, enemy thường, elite enemy, Boss ba phase, quest objective, Boss health bar, death restart, victory settlement, thậm chí còn có PBR material, environmental reflection, shadow, fog, Bloom, muzzle flash, explosion particle, hit marker và screen shake.

![Báo cáo bàn giao phiên bản đầu tiên của STEEL HAVEN](https://oss.javaguide.cn/github/javaguide/ai/coding/k3/k3-game-steel-haven-report-v1.jpg)

Đương nhiên, bản đầu tiên không phải không có vấn đề.

Trong báo cáo bàn giao, nó cũng tự ghi vài vấn đề đã biết, chẳng hạn chưa có jump, khi Boss đối diện người chơi thì sẽ bị model của chính nó che khuất trong thời gian ngắn, artifact build hơi lớn. Quan trọng hơn, sau khi thực sự chơi, mình nhận thấy rõ shooting feel vẫn có thể tiếp tục trau chuốt.

Đó là bước thứ hai.

Mình yêu cầu nó tập trung tối ưu shooting action và combat experience, không làm lại một game khác mà tiếp tục sửa trên project hiện có. Trước tiên nó xem screenshot và code, liệt kê 7 vấn đề ảnh hưởng đến shooting feel: tay trái không tham gia cầm súng, muzzle và crosshair không thẳng hàng, nhân vật che màn hình khi aim, muzzle flash bị cơ thể che, spread khi bắn liên thanh không tương ứng với spread thực tế, hit feedback chưa đủ tầng, sound effect và action reload không đồng bộ.

Sau đó nó bắt đầu sửa các module Player, PlayerController, CameraRig, Weapon, Game, Physics, AudioManager, HUD và enemy.

![K3 tối ưu shooting action và combat feel ở vòng hai](https://oss.javaguide.cn/github/javaguide/ai/coding/k3/k3-game-steel-haven-report-v2.jpg)

Sau vòng sửa này, sản phẩm đã giống game hơn.

Khi aim, nhân vật chuyển từ giữa màn hình xuống góc phải bên dưới, phía trước crosshair thoáng hơn; khi bắn có độ giật của súng và camera ngẩng lên; bắn liên thanh sẽ tích lũy spread; reload có các action theo từng giai đoạn gồm hạ xuống và nghiêng vào trong, đẩy magazine ra, lắp vào và kéo charging handle; hit, critical hit và kill có marker và sound effect khác nhau; tia lửa trên tường cũng phun theo normal, không còn giống decal hard-code dán trên tường.

Địa chỉ video Demo: <https://www.zhihu.com/pin/2062833196090275836>

Vì sao mình cho rằng case này nhất định phải đưa vào?

Vì nó thể hiện rất rõ năng lực multimodal và Agent dài hạn của K3.

Ở vòng đầu, nó phải tách gameplay, level, enemy, Boss, physics, visual, audio, UI và test từ một câu yêu cầu. Ở vòng hai, nó lại phải dựa trên screenshot và trải nghiệm thực tế để quay lại sửa shooting feel, đưa từ «chơi được» đến «giống một game hơn».

Đây không phải là việc đơn giản tạo ra một frontend page.

Đây là chuyển nhiều trải nghiệm chủ quan thành chi tiết code.

Những thứ như shooting feel rất khó định nghĩa bằng một unit test. Crosshair có bị cơ thể che không, lúc bắn có độ giật không, hit có feedback không, reload có nhịp điệu không, Boss battle có tạo cảm giác áp lực không, tất cả đều phải phán đoán bằng hình ảnh và trải nghiệm.

Điều này cho thấy K3 không chỉ viết được business code, mà còn có thể xử lý các tác vụ engineering sáng tạo và tổng hợp hơn. Chỉ cần mục tiêu đủ rõ, điều kiện nghiệm thu đủ cụ thể, nó có thể đi từ game design đến Demo game browser có thể chơi được, rồi tiếp tục tối ưu vòng hai dựa trên feedback trải nghiệm.

Phần này thực sự vượt quá kỳ vọng của mình một chút.

## Điểm nổi bật trong lần cập nhật này của K3

Trong một số Coding benchmark, K3 cơ bản đều đứng ở nhóm đầu, có project thậm chí đứng hạng nhất. Đặc biệt ở các test gần với phát triển hằng ngày và task dài như Terminal Bench, Program Bench, SWE Marathon, biểu hiện đều khá nổi bật.

![So sánh Coding benchmark của Kimi K3](https://oss.javaguide.cn/github/javaguide/ai/coding/k3/k3-coding-benchmarks.png)

Đương nhiên, benchmark chỉ nên dùng để tham khảo.

Trong project thực tế, các scenario Agent tiếp xúc sẽ phức tạp hơn: task chạy càng lâu, requirement, code, log và test output càng dồn lại với nhau.

Vì vậy context 1M rất quan trọng. Ít nhất nó giúp giữ lại nhiều requirement, code và terminal output hơn trong context, tránh việc sau vài vòng chạy, mọi chuyện xảy ra trước đó đều bị đẩy ra ngoài.

Tuy nhiên, 1M chỉ giải quyết được chứa bao nhiêu, không tự động giải quyết được cách sử dụng. Vì vậy lần này K3 còn giới thiệu Kimi Delta Attention và Attention Residuals. Trong context có quy mô một triệu token, tốc độ decode nhanh hơn tối đa 6.3 lần, đồng thời tăng khoảng 25% hiệu suất training với chi phí bổ sung dưới 2%.

Bạn không cần nhớ các thuật ngữ này. Nói đơn giản là: **K3 đã được nâng cấp về chất trong scenario task dài, ổn định hơn!**

Phần Agent và multimodal cũng khá thú vị. Trong Agent tổng quát, task bảng tính và browser, K3 cơ bản đều ở nhóm đầu, một số project đứng hạng nhất. Task visual cũng không bị bỏ lại, xét tổng thể khá cân bằng.

![So sánh General Agents và Visual Agents benchmark của Kimi K3](https://oss.javaguide.cn/github/javaguide/ai/coding/k3/k3-agent-visual-benchmarks.png)

Tuy nhiên, điều developer thông thường cuối cùng quan tâm vẫn là ba việc: **hiệu quả có đủ không, chi phí có chịu nổi không, tốc độ có nhanh không.**

Cảm nhận của mình về K3 ở điểm này là tốc độ rất nhanh, giá đặt trong các model cùng loại cũng khá thấp, có thể nói gần với giá của Sonnet, đồng thời đạt trải nghiệm cấp Opus trong các task phức tạp, cost-performance thực sự rất cao.

**⭐️Đề xuất đọc thêm:**

- [Học backend + hướng dẫn phỏng vấn](https://javaguide.cn/home.html): bao phủ kiến thức cốt lõi và nội dung phỏng vấn về Java, computer basics, database, framework, system design và phát triển backend.
- [Học phát triển ứng dụng AI + hướng dẫn phỏng vấn](https://javaguide.cn/ai/): bao phủ kiến thức và nội dung phỏng vấn về phát triển ứng dụng AI như LLM, RAG, Agent, MCP, Prompt, evaluation và system design.
- [Hướng dẫn thực chiến AI Coding](https://javaguide.cn/ai-coding/): bao phủ kỹ năng sử dụng và nội dung phỏng vấn về các tool như Claude Code, Cursor, Codex, Trae.

## Tổng kết

Sau khi chạy xong các case này, mình tin rằng bạn cũng như mình đã có nhận thức trực quan hơn về năng lực của K3, thay vì chỉ dựa vào tham số và bảng xếp hạng.

Kimi vốn luôn thuộc nhóm dẫn đầu về năng lực frontend. Lần ra mắt K3 này giúp năng lực trở nên toàn diện hơn.

Trong full-stack MVP, `/goal` có thể thúc đẩy task dài tiến lên; khi sửa vấn đề trong project hiện có, nó có thể lần theo entry, service layer, data source và utility class; đến case game, nó lại có thể kết nối gameplay, physics, camera, shooting feel và feedback system.

Mình sẽ muốn đặt nó vào những task có mục tiêu rõ ràng, điều kiện nghiệm thu cụ thể, cho phép nó liên tục thực thi lệnh và sửa lỗi. Chẳng hạn xây dựng một MVP, sửa một bug cross-module hoặc làm một interactive Demo có thể chơi được. Những task như vậy thể hiện năng lực của Coding Agent rõ hơn việc đơn thuần generate page.

Điểm khiến mình ấn tượng lần này là K3 thực sự có thể duy trì ổn định trong task dài, hiệu quả hoàn thành task cũng rất tốt, hầu như không có bug sau khi hoàn thành các case này.

Nếu giá, tốc độ và availability có thể tiếp tục ổn định, K3 sẽ là một model thuộc nhóm đầu vừa có cost-performance vừa có năng lực rất đáng gờm.

Ở đây mình không viết kiểu «vượt qua ai» hay «đè bẹp ai».

Điều developer cuối cùng quan tâm thực ra rất đơn giản: task có chạy xong không, lỗi có sửa được không, code có pass review không, giá có đủ để dùng hằng ngày không.

K3 gần đây được thảo luận rất sôi nổi trong nước, cũng có không ít người đánh giá thấp. Đề xuất của mình vẫn là: đừng vội chạy theo để khen, cũng đừng vội chạy theo để chê. Tốt nhất hãy tự chạy nó trên project thực tế và đánh giá model bằng thực hành cùng cảm nhận thực tế.
