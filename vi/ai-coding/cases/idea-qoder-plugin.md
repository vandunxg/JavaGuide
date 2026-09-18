---
title: "Thực chiến nhiều tình huống với plugin IDEA + Qoder: tối ưu API và refactor code"
description: "Thông qua hai case thực chiến, trình bày hiệu quả thực tế của IDEA kết hợp plugin Qoder trong các tình huống tối ưu deep pagination và refactor legacy code, đồng thời chia sẻ sự chuyển đổi từ cách làm người thực thi sang người chỉ huy."
category: AI Coding thực chiến
head:
  - - meta
    - name: keywords
      content: Qoder,IDEA plugin,AI coding,AI-assisted development,code refactoring,deep pagination optimization,JetBrains,intelligent coding
---

Xin chào mọi người, tôi là Xiao G. Nếu bạn là người dùng thường xuyên JetBrains IDE, khả năng cao bạn từng phân vân: muốn dùng AI hỗ trợ coding, nhưng các công cụ phổ biến như Cursor, Trae, Qoder phần lớn đều dựa trên VS Code. Chuyển sang dùng ư? Không nỡ bỏ trải nghiệm debug và refactor của JetBrains. Không chuyển ư? Lại cảm thấy mình đang bỏ lỡ lợi ích hiệu suất của AI.

Có người sẽ nói: Các công cụ terminal như Claude Code, Gemini CLI chẳng phải rất tiện sao? Đúng là rất tiện, nhưng nói thật, CLI mode cũng có nhược điểm rõ rệt: không có UI tương tác native, việc xem code và review diff đều chưa đủ trực quan. Dù có thể giảm bớt vấn đề này thông qua một số open source project như vibe kanban, 1Code, khi làm project phức tạp vẫn còn một số giới hạn.

Các backend developer hiện nay nhìn chung được chia thành bốn nhóm:

| Nhóm                | Tổ hợp công cụ                                             | Đặc điểm                                                  |
| ------------------- | ---------------------------------------------------------- | --------------------------------------------------------- |
| **Nhóm CLI**        | Claude Code/Gemini CLI/Codex                               | Thao tác terminal, hiệu suất cao nhưng UI tương tác yếu   |
| **Nhóm VS Code**    | VS Code + plugin                                           | Nhẹ và linh hoạt, tính năng bị giới hạn                   |
| **Nhóm hybrid**     | CLI/AI coding IDE (như Cursor) viết → JetBrains nghiệm thu | AI hỗ trợ + IDEA dự phòng                                 |
| **Nhóm all-in-one** | **JetBrains + plugin Qoder**                               | **Tập trung flow state, xử lý mọi việc trong một cửa sổ** |

Hiện tại tôi thuộc “nhóm dùng hybrid”: Claude Code và IDEA + plugin Qoder là tổ hợp chính.

Với nhiều project có logic phức tạp, cảm giác kiểm soát của IDEA giúp bạn yên tâm hơn.

Trong bài viết này, tôi sẽ thông qua hai case thực chiến để xem hiệu quả thực tế của IDEA kết hợp Qoder, đồng thời chia sẻ một số mẹo hữu ích.

## Hướng dẫn làm quen với plugin Qoder JetBrains

### Cài đặt và cấu hình

**Bước 1**: Nhấp **Settings | Plugins**, tìm kiếm **"qoder"**, chọn Qoder - Agentic AI Coding Platform rồi cài đặt.

![Giao diện cài đặt plugin](https://oss.javaguide.cn/github/javaguide/ai/coding/qoder/idea-plugin/plugin-install-interface.png)

**Bước 2**: Sau khi cài đặt xong, nhấp Sign In để đăng nhập hoặc đăng ký.

![Giao diện đăng nhập](https://oss.javaguide.cn/github/javaguide/ai/coding/qoder/idea-plugin/login-interface.png)

**Bước 3 (tùy chọn)**: Giao diện mặc định là tiếng Anh. Nếu quen dùng tiếng Trung, bạn có thể nhấp Plugin Settings ở góc trên bên phải và đặt Display Language thành tiếng Trung giản thể.

![Giao diện cài đặt ngôn ngữ](https://oss.javaguide.cn/github/javaguide/ai/coding/qoder/idea-plugin/language-settings-interface.png)

**Bước 4 (tùy chọn)**: Cấu hình database connection. Qoder hỗ trợ context `@database`, có thể tham chiếu trực tiếp đến table structure của database. Bạn nên cấu hình database liên quan đến project từ trước.

Lấy MySQL làm ví dụ, mở cửa sổ công cụ Database ở bên phải, nhấp dấu **+**, chọn **Data Source | MySQL**:

![Thêm data source](https://oss.javaguide.cn/github/javaguide/ai/coding/qoder/idea-plugin/add-data-source.png)

Điền thông tin connection, sau khi kiểm tra thành công thì nhấp OK.

![Hoàn tất cấu hình database](https://oss.javaguide.cn/github/javaguide/ai/coding/qoder/idea-plugin/database-config-complete.png)

Đến đây, công tác chuẩn bị ban đầu đã hoàn tất.

### Nhiệm vụ 1: Truy vấn order thường xuyên báo lỗi? Dùng Qoder hỗ trợ kiểm tra deep pagination

#### Bối cảnh

Đây là một hệ thống quản trị backend thương mại điện tử, bộ phận vận hành tạo báo cáo phân tích kinh doanh mỗi tháng. Do lượng dữ liệu lớn (order table hơn 10 triệu bản ghi), thời gian development gấp, code tồn tại nhiều rủi ro về performance.

Bộ phận vận hành phản ánh truy vấn order thường xuyên báo lỗi, sau khi xác định thì vấn đề nằm ở API:

```bash
curl -X POST http://localhost:8080/api/report/orders \
  -H "Content-Type: application/json" \
  -d '{"page": 1000000, "size": 10}'
```

Đây là một deep pagination request điển hình. Logic code của API như sau:

```java
@Transactional(readOnly = true)
public OrderListResponse getOrderList(OrderListRequest request) {
    int pageNum = request.getPage() == null ? 1 : request.getPage();
    int pageSize = request.getSize() == null ? 10 : request.getSize();

    // Vấn đề cốt lõi: deep pagination query
    Page<Order> pageParam = new Page<>(pageNum, pageSize);

    LambdaQueryWrapper<Order> wrapper = new LambdaQueryWrapper<>();
    if (request.getStatus() != null && !request.getStatus().isEmpty()) {
        wrapper.eq(Order::getStatus, request.getStatus());
    }
    if (request.getShopId() != null) {
        wrapper.eq(Order::getShopId, request.getShopId());
    }

    // Trường sort có thể chưa có index, dẫn đến full table scan
    wrapper.orderByDesc(Order::getCreatedAt);

    // Deep pagination: LIMIT 9999990, 10
    IPage<Order> orderPage = orderMapper.selectPage(pageParam, wrapper);

    // Truy vấn liên kết thông tin user, shop...
}
```

Khi `page=1000000`, MySQL thực thi `LIMIT 9999990, 10`, cần scan 10 triệu dòng đầu rồi loại bỏ chúng, khiến performance giảm mạnh.

#### Khó khăn của cách làm truyền thống

Theo quy trình truyền thống, việc tuning API cần:

1. Đọc và hệ thống hóa logic code
2. Phân tích không gian tối ưu code
3. Kết hợp log để phân tích execution plan của SQL
4. Đưa ra solution và triển khai
5. Regression test và deploy lên production

Nếu kiểm tra từ code, execution plan của SQL cho đến regression và release, loại vấn đề này thường không thể kết thúc chỉ bằng việc sửa một dòng SQL. Thời gian cụ thể phụ thuộc vào mức độ quen thuộc với project, quy mô dữ liệu và yêu cầu verification.

#### Cách Qoder giải quyết: từ người thực thi sang người chỉ huy

Sau khi có Qoder, tôi dành nhiều thời gian hơn cho việc phân rã nhiệm vụ, review solution và nghiệm thu kết quả.

Chỉ cần hệ thống hóa suy nghĩ và đưa ra mục tiêu rõ ràng:

```bash
API truy vấn order list đang gặp lỗi timeout "java.net.SocketTimeoutException: Read timed out". Cần phân tích từ logic code của API và tầng database rồi đưa ra solution.

Thông tin API: POST http://localhost:8080/api/report/orders
Request parameter: {"page": 1000000, "size": 10}

Hãy đưa ra solution từ các khía cạnh sau:
1. Phân tích các yếu tố có thể gây timeout trong logic code của API
2. Kiểm tra các vấn đề ở tầng database (index, query performance, data volume)
3. Đề xuất các biện pháp tối ưu cụ thể
```

Để Qoder hoàn thành nhiệm vụ tốt hơn, hãy thêm database context:

1. Nhấp nút **+Add Context**
2. Chọn **@database**, chọn database Schema tương ứng

![Thêm database context](https://oss.javaguide.cn/github/javaguide/ai/coding/qoder/idea-plugin/add-database-context-1.png)

#### Phân tích vấn đề và xuất solution

**Xác định code entry point và các nguyên nhân tiềm năng**

Qoder nhanh chóng xác định code entry point và liệt kê các nguyên nhân tiềm năng như deep pagination, sorting index. Kết luận này vẫn cần được kiểm chứng bằng slow query log và `EXPLAIN ANALYZE`:

![Kết quả phân tích code](https://oss.javaguide.cn/github/javaguide/ai/coding/qoder/idea-plugin/code-analysis-result.png)

**Điểm nổi bật: chẩn đoán kết hợp code và database**

Kết hợp database Schema, Qoder đưa ra một báo cáo phân tích tổng hợp. `@database` là khả năng database context hiện được Qoder cung cấp, phù hợp để bổ sung table structure và index information, nhưng không thể thay thế execution plan trên production và data distribution thực tế:

![Báo cáo phân tích tổng hợp](https://oss.javaguide.cn/github/javaguide/ai/coding/qoder/idea-plugin/comprehensive-analysis-report.png)

**Tối ưu ở tầng code**

Qoder đưa ra ba solution, trong đó có delayed association query (subquery chỉ trả về ID, sử dụng covering index để nhanh chóng xác định vị trí):

![Solution tối ưu code](https://oss.javaguide.cn/github/javaguide/ai/coding/qoder/idea-plugin/code-optimization-solution.png)

**Solution đáng chú ý**

Về việc tính tổng số record trong pagination query, Qoder còn đưa ra một solution ước tính: ước tính tổng số lượng thông qua số lượng index page của primary key và số dòng trung bình trên mỗi page. Cách này chỉ phù hợp với tình huống chấp nhận kết quả gần đúng; sai số chịu ảnh hưởng của page gap, page fill rate và data distribution. Các nghiệp vụ như accounting, settlement yêu cầu đếm chính xác thì không thể áp dụng trực tiếp:

![Đề xuất tối ưu database](https://oss.javaguide.cn/github/javaguide/ai/coding/qoder/idea-plugin/database-optimization-suggestion.png)

#### Triển khai solution và nghiệm thu

Sau khi review và đánh giá, chọn solution delayed association + index optimization:

```bash
Dựa trên kết quả review và đánh giá, thực hiện các tối ưu sau:
1. Triển khai chiến lược delayed association query, refactor logic deep pagination query
2. Tạo index structure đã được tối ưu theo đề xuất về index
3. Viết unit test, bao phủ các điểm chức năng cốt lõi và thiết lập performance baseline
```

Sau khi Qoder hoàn tất triển khai, method `getOrderList` được cải tiến:

- Hoàn thành cấu hình và giới hạn logic về page number tối đa dựa trên production incident
- Hoàn thành pagination statistics và list query theo các strategy khác nhau

Qua screenshot có thể thấy naming và method split sau khi refactor quy củ hơn. Việc có hoàn toàn phù hợp với coding convention của team hay không vẫn cần được xác nhận thông qua static check và Code Review của chính project:

![Code sau khi refactor](https://oss.javaguide.cn/github/javaguide/ai/coding/qoder/idea-plugin/refactored-code.png)

Index script có thể được thực thi trực tiếp trong IDE, toàn bộ workflow không cần chuyển cửa sổ:

![Thực thi index](https://oss.javaguide.cn/github/javaguide/ai/coding/qoder/idea-plugin/index-execution.png)

**Regression test**: Qoder hoàn tất việc hệ thống hóa các code branch và tạo unit test cho các tình huống khác nhau:

![Unit test](https://oss.javaguide.cn/github/javaguide/ai/coding/qoder/idea-plugin/unit-test-1.png)

**Stress test**: Qoder tạo stress test code và thêm JIT warm-up. Warm-up chỉ có thể giảm ảnh hưởng của cold start và JIT compilation tức thời lên kết quả, không có nghĩa là test đã gần với production. Để đánh giá optimization có hiệu quả hay không, vẫn cần ghi lại hardware, JDK và GC, data distribution, cache state, concurrency model, sample size cùng p95/p99 latency:

![Stress test](https://oss.javaguide.cn/github/javaguide/ai/coding/qoder/idea-plugin/stress-test.png)

Cuối cùng, Qoder xuất ra work summary hoàn chỉnh, bao gồm technical solution và đề xuất giao tiếp, báo cáo:

![Work summary](https://oss.javaguide.cn/github/javaguide/ai/coding/qoder/idea-plugin/work-summary.png)

Nhấp Qoder trong cửa sổ commit code để tạo commit message dựa trên Diff hiện tại. Trong phần demo này, từ lúc nhập context đến khi tạo candidate change mất khoảng 10 phút; việc review execution plan, stress test, Code Review và verification khi release không nằm trong khoảng thời gian này.

![Commit message](https://oss.javaguide.cn/github/javaguide/ai/coding/qoder/idea-plugin/commit-message.png)

### Nhiệm vụ 2: Hệ thống hóa và refactor một đoạn legacy refund code

#### Bối cảnh: một đoạn “legacy code” không ai dám động vào

Method `applyRefund` của refund module có **hơn 150 dòng code, không có comment, đầy magic value và logic trùng lặp dư thừa**. Yêu cầu mới xuất hiện: thêm risk control rule — **user có order chưa hoàn tất trong vòng 72 giờ không được phép gửi yêu cầu refund**.

**Khó khăn của cách làm truyền thống**:

- Logic code phức tạp, không dám tùy tiện sửa đổi
- Thêm rule mới cần regression test toàn bộ
- Nếu bổ sung feature test, refactor và regression, effort thường cần được đánh giá theo tình hình thực tế của project

#### Hệ thống hóa logic: để Agent đọc legacy code thay bạn

Nhờ khả năng suy luận context của model phía sau Qoder cùng khả năng lập kế hoạch và thực thi nhiệm vụ của Agent, có thể để nó đọc và refactor business function:

```bash
Hãy kết hợp với một data flow đơn giản để giới thiệu chi tiết business flow hoàn chỉnh của việc gửi yêu cầu refund, đồng thời bổ sung các comment tương ứng trong code.
```

Để giảm việc Agent phỏng đoán table structure, hãy gửi Schema hiện có làm context cho Qoder. Schema chỉ có thể bổ sung database structure, không đảm bảo nó hiểu chính xác business rule:

![Thêm database context](https://oss.javaguide.cn/github/javaguide/ai/coding/qoder/idea-plugin/add-database-context-2.png)

Sau khi nhận nhiệm vụ, Qoder bắt đầu từ phần tổng quan, rồi thực hiện nhiệm vụ bằng cách lần lượt hệ thống hóa và chú thích từng branch:

![Quá trình hệ thống hóa logic](https://oss.javaguide.cn/github/javaguide/ai/coding/qoder/idea-plugin/logic-analysis-process.png)

Code có comment tương ứng dễ đọc hơn, nhưng vẫn cần đối chiếu từng comment và data flow với implementation ban đầu, test và product rule:

![Ví dụ code có comment](https://oss.javaguide.cn/github/javaguide/ai/coding/qoder/idea-plugin/commented-code-example.png)

Sau khi kết thúc nhiệm vụ, Qoder hệ thống hóa rõ ràng logic của API và các điểm rule đặc biệt:

![Tóm tắt](https://oss.javaguide.cn/github/javaguide/ai/coding/qoder/idea-plugin/summary-conclusion.png)

#### Refactor code: thiết lập regression baseline trước

Sau khi hoàn tất việc hệ thống hóa logic, đưa ra instruction thứ hai để hoàn thành function refactoring và regression:

```bash
Hãy refactor có hệ thống module chức năng gửi yêu cầu refund theo coding convention của team và tham khảo phương pháp refactor trong cuốn "Refactoring: Improving the Design of Existing Code". Trước khi refactor, hãy bổ sung feature test cho behavior hiện tại; sau khi refactor, thực hiện unit test, integration test và feature test quan trọng, bao phủ các business branch và boundary condition đã xác định. Xuất các branch chưa được bao phủ và behavior cần con người xác nhận, không được dùng coverage để thay thế verification về tính tương đương giữa implementation mới và cũ.
```

Trong quá trình này, Qoder lần lượt hoàn thành:

1. Xem target file: xác định đoạn code cần refactor
2. Phân tích vấn đề code: chỉ ra magic value, code trùng lặp, method quá dài và các vấn đề khác
3. Refactor có hệ thống: lần lượt hoàn thành việc tạo constant, trích xuất code trùng lặp, thiết kế domain model và phân tách responsibility
4. Viết test code để hoàn thành logic regression

Code được tạo cuối cùng như sau. Qoder không trực tiếp sửa `RefundService` ban đầu mà tạo mới `RefundServiceRefactored`. Cách này tạo không gian để đối chiếu implementation mới và cũ, nhưng việc tạo một class mới không đồng nghĩa với “refactor an toàn”: vẫn cần xác định rõ call route, gradual rollout hoặc feature flag, dùng cùng một nhóm input để so sánh kết quả mới và cũ, đồng thời thống nhất thời điểm ngừng implementation cũ. Nếu không, hai phần logic rất dễ tiếp tục drift trong thời gian dài.

```java
/**
 * Refund request (sau khi refactor)
 */
@Transactional(rollbackFor = Exception.class)
public RefundResponse applyRefund(RefundApplyRequest request) {
    log.info("【Refund request】Bắt đầu xử lý: orderId={}, userId={}, amount={}",
            request.getOrderId(), request.getUserId(), request.getRefundAmount());

    // 1. Query và validate order
    Order order = getAndValidateOrder(request.getOrderId(), request.getUserId());

    // 2. Xác định loại refund và xử lý
    if (request.getOrderItemId() != null) {
        return processPartialRefund(request, order);   // Partial refund
    } else {
        return processFullRefund(request, order);      // Full refund
    }
}

/**
 * Xử lý partial refund
 */
private RefundResponse processPartialRefund(RefundApplyRequest request, Order order) {
    log.info("【Refund request】Xử lý partial refund: orderItemId={}", request.getOrderItemId());

    // Query và validate order detail
    OrderItem orderItem = orderItemMapper.selectById(request.getOrderItemId());
    refundValidator.validateOrderItemBelongsToOrder(orderItem, order.getId());

    // Validate refund quantity và amount
    Integer refundQuantity = getRefundQuantity(request.getQuantity());
    refundValidator.validateRefundQuantity(refundQuantity, orderItem.getRefundableQuantity());
    BigDecimal itemRefundableAmount = refundCalculator.calculateItemRefundableAmount(orderItem, refundQuantity);
    refundValidator.validateRefundAmount(request.getRefundAmount(), itemRefundableAmount);

    // Thực hiện risk check + tạo refund record
    performRiskCheck(order, request.getRefundAmount(), request.getUserId());
    Refund refund = createRefundRecord(request, order, refundQuantity);

    log.info("【Refund request】Partial refund thành công: refundId={}", refund.getId());
    return RefundResponse.success(refund.getId());
}
```

**Điểm nổi bật của refactor**:

| Điểm nổi bật            | Mô tả                                                                         |
| ----------------------- | ----------------------------------------------------------------------------- |
| **Tách method**         | Main method chỉ có 15 dòng, tách riêng logic partial refund/full refund       |
| **Tách responsibility** | `refundValidator`, `refundCalculator` xử lý validation và calculation độc lập |
| **Comment rõ ràng**     | Mỗi bước đều có mô tả tương ứng                                               |
| **Log theo convention** | Dùng 【】 đánh dấu các node quan trọng để thuận tiện trace                    |
| **Exception handling**  | Cấu hình transaction rollback policy cho checked exception                    |

Báo cáo unit test ở đây cho thấy branch coverage khoảng 80%. Điều này cho biết một số path đã được thực thi, nhưng không chứng minh behavior trước và sau refactor hoàn toàn giống nhau. Các path quan trọng như amount, state transition, concurrent request và lỗi external dependency vẫn cần feature test, integration test hoặc đối chiếu kết quả mới và cũ:

![Nghiệm thu unit test](https://oss.javaguide.cn/github/javaguide/ai/coding/qoder/idea-plugin/unit-test-verification.png)

#### Iteration chức năng: một instruction, đưa rule lên production

Sau khi hoàn thành việc điều chỉnh structure và regression baseline, có thể xác định risk control logic `validateRiskMaxAmount`, rồi đưa ra instruction cuối cùng cho Qoder:

```bash
Hãy thêm một refund restriction rule trong hệ thống risk control: khi user có bất kỳ order nào ở trạng thái chưa hoàn tất trong 72 giờ gần nhất (3 ngày), hệ thống phải tự động từ chối refund request của user đó.
```

Implementation code tương ứng như sau. Có thể thấy, sau khi hệ thống hóa logic hiện có, validation framework có responsibility đơn nhất và các unit test đi kèm đã sẵn sàng, các incremental iteration tiếp theo cũng dễ xử lý và regression hơn:

![Implementation của feature iteration](https://oss.javaguide.cn/github/javaguide/ai/coding/qoder/idea-plugin/feature-iteration-implementation.png)

#### Tích lũy memory: càng dùng càng hiểu thói quen coding của bạn

Sau khi hoàn thành nhiệm vụ, Qoder hình thành memory cho project này. Memory là capability chính thức hiện tại, nhưng việc có phát huy tác dụng trong các nhiệm vụ tiếp theo hay không vẫn phụ thuộc vào việc memory có được lưu, recall đúng hay không và có conflict với trạng thái hiện tại hay không:

- **Memory về đặc điểm project**: delayed association query tốt hơn cursor pagination, tối ưu API cần đi kèm performance test
- **Memory về coding convention**: tuân thủ _Alibaba Java Development Manual_, dùng `compareTo` để so sánh BigDecimal
- **Memory về business rule**: refund risk control rule (chặn order chưa hoàn tất trong 72 giờ, giới hạn amount của một giao dịch, v.v.)

Trong phần demo này, refund rule và coding convention được ghi vào memory list. Khi sử dụng về sau, vẫn cần review nội dung được recall và kịp thời xóa các rule đã hết hiệu lực:

![Tích lũy memory](https://oss.javaguide.cn/github/javaguide/ai/coding/qoder/idea-plugin/memory-accumulation.png)

## Phân tích capability: Qoder đã làm gì trong ví dụ này

Thông qua hai case thực chiến ở trên, hãy phân tích vai trò của Qoder trong workflow development thực tế.

### 1. Nhận biết engineering và hiểu context

Khả năng hiểu các project engineering lớn của Qoder:

- **Nhận biết database Schema**: Trong nhiệm vụ 1, Qoder kết hợp context `@database` để phân tích table structure và index hiện có của order table, rồi đưa ra candidate index solution. Việc có áp dụng hay không vẫn cần xem xét đầy đủ SQL, selectivity và execution plan.

- **Truy nguyên logic code**: Trong nhiệm vụ 2, trước đoạn refund code dài không có comment, Qoder thông qua static analysis đã hệ thống hóa business flow: order validation → amount calculation → risk control check → data persistence, đồng thời đánh dấu code smell như code trùng lặp, magic value.

- **Liên kết cross-file**: Qoder có thể tự động nhận biết các file liên quan cần cho nhiệm vụ, chẳng hạn tự động trace từ `RefundService` đến các dependency component như `OrderMapper`, `RefundValidator`, không cần tự thêm context.

### 2. Khả năng thực thi nhiệm vụ end-to-end

Qoder không chỉ làm code completion; trong ví dụ này, nó còn tham gia phân tích, thay đổi và tạo test:

| Khía cạnh capability      | Biểu hiện quan sát được trong bài viết                   | Phần vẫn cần con người verify                                          |
| ------------------------- | -------------------------------------------------------- | ---------------------------------------------------------------------- |
| **Nhận biết engineering** | Phân tích database Schema và quan hệ dependency của code | SQL execution plan, data distribution và business rule                 |
| **Thực thi nhiệm vụ**     | Tham gia phân tích, thiết kế, coding và tạo test         | Test effectiveness, code review và release verification                |
| **Hỗ trợ refactor**       | Giữ lại implementation ban đầu và tạo implementation mới | Chuyển route, đối chiếu mới-cũ và ngừng code cũ                        |
| **Memory của project**    | Lưu convention của project và một phần business rule     | Độ chính xác của memory recall và việc nội dung còn hiệu lực hay không |

### 3. Refactor từng bước và incremental iteration

Nhiệm vụ 2 giữ lại implementation ban đầu và tạo implementation mới, có thể dùng nó làm điểm bắt đầu cho migration từng bước.

- **Refactor incremental**: Qoder không trực tiếp sửa `RefundService` hiện có mà tạo class `RefundServiceRefactored` mới. Chỉ sau khi bổ sung migration solution, cách làm này mới có các giá trị sau:

  - Giữ implementation ban đầu để đối chiếu behavior
  - Dần chuyển đổi thông qua feature flag hoặc gradual rollout route
  - Xóa implementation cũ sau khi verification thành công, tránh hai phần logic cùng tồn tại lâu dài

- **Tách responsibility**: Qoder tuân theo Single Responsibility Principle (SRP), tách validation logic, amount calculation và number generation vốn bị trộn lẫn thành các component độc lập:

  - `RefundValidator`: business validation thống nhất
  - `RefundCalculator`: amount calculation logic
  - `RefundNoGenerator`: tạo refund number

- **Transaction boundary**: `rollbackFor = Exception.class` chỉ có hiệu lực khi method được gọi thông qua Spring proxy, exception tiếp tục được throw và resource tham gia cùng một transaction. Self-invocation, việc catch rồi nuốt exception hoặc thao tác cross-service cần được xử lý riêng.

### 4. Nhận biết memory và continuous learning

Các memory này có thể được recall trong những interaction tiếp theo. Khi liên quan đến business rule, nên đặt rule chính thức trong project document có thể review và version hóa; Memory chỉ nên được dùng làm context hỗ trợ.

## Tổng kết

Plugin Qoder JetBrains cung cấp cho backend developer một phương thức làm việc mới: **duy trì thói quen sử dụng JetBrains IDE, đồng thời tận dụng khả năng suy luận, phân tích và triển khai coding của AI Agent**.

Nhìn lại hai case này:

| Khía cạnh       | Qoder có thể hỗ trợ                        | Khâu engineering không thể bỏ qua                                        |
| --------------- | ------------------------------------------ | ------------------------------------------------------------------------ |
| **Phân tích**   | Tìm kiếm code, liên kết Schema             | Đối chiếu log, execution plan và business rule                           |
| **Thay đổi**    | Tạo candidate implementation và test       | Code Review, đối chiếu behavior mới-cũ                                   |
| **Trải nghiệm** | Hoàn thành phần lớn interaction trong IDEA | Stress test trên môi trường thực tế, gradual rollout và theo dõi release |
| **Tích lũy**    | Lưu một phần memory của project            | Quản lý version của rule và xóa nội dung hết hiệu lực                    |

## Lời cuối

Môi trường công nghệ hiện nay giống như đang xây một tòa nhà. AI và framework mới giúp bạn dựng scaffolding rất nhanh, các plugin như Qoder cho phép bạn hoàn thành mọi việc ngay trong IDE quen thuộc mà không cần chuyển cửa sổ làm gián đoạn suy nghĩ. Nhưng nếu thiếu kiến thức về nguyên lý bên trong và tư duy thiết kế software architecture, dù AI có thể giúp bạn triển khai function, bạn vẫn không thể kiểm soát chất lượng delivery của hệ thống.

Nhìn lại hai case trong bài viết:

- **Delayed association query trong nhiệm vụ 1**, phải dựa trên hiểu biết về nguyên lý database index mới có thể đánh giá solution Qoder đưa ra có hợp lý hay không.

- **Code refactor trong nhiệm vụ 2**, phải quen thuộc với _Refactoring: Improving the Design of Existing Code_ và các principle như SRP, DRY trong _Alibaba Java Development Manual_ mới có thể đánh giá chính xác chất lượng refactor của Qoder.

- **JIT warm-up trong performance benchmark test**, chỉ giải quyết được một phần yếu tố gây nhiễu của benchmark test; nếu không hiểu JVM, data và load model, kết quả vẫn có thể sai lệch.

- **Lựa chọn và cân nhắc solution**, cần nắm được business scenario và ranh giới kỹ thuật. Ví dụ, chọn delayed association query thay vì cursor pagination vì cách sau sẽ ảnh hưởng đến user experience. Đây là phán đoán mà AI không thể thay bạn thực hiện.

Khi dùng Qoder để xử lý loại nhiệm vụ này, có ba đề xuất:

1. **Duy trì học các nguyên lý bên trong**: database index, JVM memory model, concurrency programming principle, các kiến thức “nền móng” này sẽ không mất giá vì AI.

2. **Đọc sách kinh điển**: _Refactoring_, _Design Patterns_, _High Performance MySQL_, _Understanding the JVM in Depth_; những tác phẩm kinh điển này giúp bạn xây dựng “thước đo” để đánh giá chất lượng output của AI.

3. **Rèn luyện tư duy architecture**: dùng thời gian tiết kiệm được để suy nghĩ về system architecture và bản chất business.

Nếu bạn chủ yếu sử dụng JetBrains IDE, có thể xem plugin Qoder là một phương án thay thế. Plugin này giảm việc chuyển đổi editor, nhưng chất lượng delivery cuối cùng vẫn phụ thuộc vào execution plan, test, review và release verification.
