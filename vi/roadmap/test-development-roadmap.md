---
title: "Lộ trình học Test Development (phiên bản mới nhất 2026): Từ testing đến Quality Engineering trong kỷ nguyên AI"
description: "Lộ trình học phiên bản mới nhất 2026 dành cho Test Development và software testing, bao quát nền tảng computer science, ngôn ngữ lập trình, lý thuyết testing, API automation, UI automation, performance testing, CI/CD, test platform, AI-assisted testing và testing ứng dụng AI."
category: Lộ trình học
head:
  - - meta
    - name: keywords
      content: Lộ trình học Test Development,Lộ trình học Test Development, lộ trình học software testing,lộ trình học Test Development 2026,AI testing,automated testing,API testing,UI automated testing,performance testing,test platform,Quality Engineering
---

Xin chào, tôi là Guide. Đây là phiên bản mới nhất 2026 của lộ trình học dành cho Test Development.

Trong phần bình luận, thường có bạn hỏi:

> Java backend cạnh tranh quá, có thể chuyển sang Test Development không?
>
> Test Development có đơn giản hơn development một chút không?
>
> AI đã có thể viết test case, vậy sau này vị trí testing còn đáng học không?

Nhận định của tôi khá trực tiếp: Test Development có thể là một hướng tìm việc tốt, nhưng đừng hiểu đây là “phương án dự phòng sau khi không học nổi backend”. Test Development engineer (Software Development Engineer in Test, viết tắt là SDET) quả thực thiên về kỹ thuật hơn trong hệ thống testing, nhưng yêu cầu bạn kết nối programming, testing theory, engineering tools, business understanding và quality assurance.

Nếu chỉ biết một chút manual testing, biết một chút Postman, viết được vài Selenium script thì trong môi trường hiện tại, năng lực cạnh tranh vẫn chưa đủ. Hướng tốt hơn là: có thể viết automated framework, kết nối CI/CD, đọc log và monitoring để định vị vấn đề, thực hiện API, UI, performance và stability verification, đồng thời sử dụng AI cho test case generation, script maintenance, log analysis, defect attribution và evaluation ứng dụng AI.

Bài viết này chủ yếu dành cho ba nhóm bạn:

- Sinh viên các chuyên ngành liên quan đến computer science, muốn chuẩn bị theo hướng Test Development, automated testing hoặc Quality Engineering.
- Những bạn đang làm functional testing, muốn bổ sung năng lực programming và automation.
- Những bạn có nền tảng Java / Python / backend, muốn mở rộng hướng tìm việc sang Test Development.

Xin nhắc trước một điều: nếu mục tiêu dài hạn của bạn là thuần business development, kinh nghiệm Test Development chưa chắc luôn có thể chuyển mượt về vị trí development. Thành quả của dự án Test Development thường thể hiện nhiều hơn ở quality system, automation efficiency, problem localization và platform capability, cách trình bày không hoàn toàn giống business feature development. Sau này muốn quay lại development cũng được, nhưng cần thiết kế cách trình bày trong CV và phỏng vấn từ sớm.

## Trước hết hãy hiểu Test Development thực sự làm gì

Testing truyền thống chú trọng hơn vào “tính năng này có vấn đề không”. Test Development còn phải tiến thêm vài bước: Vì sao vấn đề này bị bỏ sót? Sau này có thể tự động phát hiện không? Những vấn đề như vậy có thể được tích lũy thành tool, platform hoặc process không?

Trong team thực tế, công việc của Test Development có thể bao gồm:

- Tham gia requirement review, sớm xác định boundary condition, exception flow và risk point.
- Thiết kế test case, bao phủ function, API, compatibility, security, performance và stability.
- Viết script API automation, UI automation và App automation.
- Xây dựng automated testing framework, quản lý test data, test environment và test report.
- Tích hợp automated test case vào các pipeline như Jenkins, GitHub Actions, GitLab CI.
- Thực hiện performance test, phân tích các chỉ số throughput, response time, error rate, CPU, memory, database và cache.
- Phát triển test platform, chẳng hạn như test case management, automated scheduling, report aggregation, coverage analysis và precise testing.
- Testing ứng dụng AI, chẳng hạn RAG QA, intelligent customer service, Agent tool calling và multimodal application.

Vì vậy, code của Test Development chủ yếu nằm ở testing framework, testing tool, quality platform và script định vị vấn đề. Lượng code có thể không lớn bằng business development, nhưng mức độ hiểu engineering chain phải đầy đủ hơn.

## Trong kỷ nguyên AI, Test Development cần học thêm gì

Ảnh hưởng của AI đến testing đã rất rõ ràng.

Một mặt, AI có thể giúp bạn nâng cao hiệu suất testing. Ví dụ, dựa trên requirement document để tạo test point, bổ sung boundary case, viết lại API automation script, phân tích failure log, tạo bản nháp đầu tiên của performance test report. Trước đây viết 20 test case mất nửa tiếng, hiện tại có thể yêu cầu AI tạo bản đầu tiên trước, sau đó bạn review và bổ sung.

Mặt khác, bản thân ứng dụng AI cũng cần được testing. Hệ thống truyền thống thường deterministic: API trả về sai field là sai; ứng dụng AI phức tạp hơn, cùng một câu hỏi có thể có nhiều câu trả lời khác nhau, câu trả lời trông mạch lạc nhưng sự thật chưa chắc đúng, RAG retrieval có thể không retrieve đúng tài liệu, Agent có thể chọn sai tool hoặc tạo sai parameter.

Điều này có nghĩa Test Development cần bổ sung một năng lực mới: **đánh giá quality của hệ thống không deterministic**.

Bạn ít nhất phải biết cách testing những vấn đề sau:

- Prompt có dễ bị injection attack bypass không?
- RAG có retrieve đúng document không, câu trả lời có faithful với tài liệu được retrieve không?
- JSON do LLM output có ổn định không, hệ thống fallback thế nào khi thiếu field?
- Khi Agent gọi tool, tool selection, parameter generation và việc điền kết quả thực thi trở lại có chính xác không?
- Trong multi-turn conversation, context pollution, historical memory và permission boundary có vấn đề không?
- Sau khi model switch, Prompt adjustment hoặc Embedding update, quality tốt lên hay kém đi?

AI không thể thay thế testing judgment tốt. AI có thể giúp bạn tạo candidate test case và script nhanh hơn, nhưng cuối cùng bạn phải đánh giá coverage đã đầy đủ chưa, assertion có hiệu quả không, failure có thực sự chỉ ra vấn đề không.

## Tổng quan lộ trình học Test Development

Nếu bắt đầu từ con số 0, bạn có thể đi theo thứ tự sau:

```text
Computer science fundamentals -> Programming language -> Database/Linux/Git/Docker -> Testing theory
-> API automation -> UI/App automation -> Performance testing -> CI/CD và Quality Engineering
-> AI-assisted testing -> AI application testing -> Project và interview presentation
```

Đừng vừa bắt đầu đã học một đống tool. Tool có thể nhanh chóng làm quen, nhưng thứ thực sự tạo ra khác biệt trong Test Development là ba điều:

- Có thể viết test code dễ maintain, hạn chế viết script dùng một lần.
- Có thể đưa automation vào engineering process, chạy trong CI và daily regression.
- Có thể giải thích nguyên nhân phía sau một failure, đưa ra evidence từ log, data và chain.

## Giai đoạn một: Bổ sung computer science fundamentals

Phỏng vấn Test Development cũng sẽ hỏi computer science fundamentals, đặc biệt là trong tuyển dụng sinh viên mới tốt nghiệp, thực tập và phỏng vấn big tech.

Không cần học toàn bộ với độ sâu của kỳ thi cao học 408, nhưng phải giải thích rõ các nội dung sau:

- Computer network: HTTP/HTTPS, TCP/UDP, three-way handshake và four-way wavehand, DNS, Cookie / Session / Token, các status code thường gặp, điều gì xảy ra sau khi nhập URL vào browser.
- Operating system: process và thread, context switch, deadlock, memory management, I/O, Linux file permission và các command thường dùng.
- Data structure và algorithm: array, linked list, stack, Queue, hash table, tree, heap, sorting, binary search, two pointers, DFS / BFS, nền tảng dynamic programming.
- Basic system design awareness: cache, rate limiting, timeout, retry, log, monitoring, degradation.

Khi hỏi computer network trong Test Development, câu hỏi thường gắn với troubleshooting scenario. Ví dụ, kiểm tra thế nào khi page load chậm, định vị ra sao khi API thỉnh thoảng timeout, các nguyên nhân có thể có khi bắt được 401 / 403 / 500 từ App. Bạn không thể chỉ học thuộc concept, mà phải có thể lần theo request chain để trình bày.

Có thể kết hợp học nội dung trong JavaGuide:

- [Câu hỏi phỏng vấn computer network thường gặp](../cs-basics/network/other-network-questions.md)
- [Câu hỏi phỏng vấn operating system thường gặp](../cs-basics/operating-system/operating-system-basic-questions-01.md)
- [Process và thread](../cs-basics/operating-system/process-and-thread.md)
- [Tổng hợp command Linux thường dùng](../cs-basics/operating-system/linux-intro.md)
- [Data structure và algorithm](../cs-basics/algorithms/)

Không cần luyện algorithm đến mức khắc nghiệt như vị trí backend development, nhưng phải làm được các bài cơ bản. Test Development vẫn là vị trí kỹ thuật, gặp bài algorithm trong bài test viết và vòng phỏng vấn đầu tiên là chuyện bình thường.

## Giai đoạn hai: Chọn một ngôn ngữ chính, bổ sung một ngôn ngữ phụ

Test Development không thể tránh programming.

Không cần mất quá nhiều thời gian để cân nhắc chọn ngôn ngữ. Java và Python đều có thể dùng cho Test Development, chỉ là trọng tâm khác nhau:

| Ngôn ngữ      | Scenario phù hợp hơn                                                                                             | Đề xuất                                                                                                              |
| ------------- | ---------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------- |
| Java          | Đã có nền tảng Java, công ty mục tiêu thiên về Java tech stack, muốn làm test platform hoặc backend quality tool | Có thể trực tiếp dùng Java để ứng tuyển Test Development, trọng tâm bổ sung JUnit, TestNG, Rest Assured, Spring Boot |
| Python        | Muốn nhanh chóng viết script, API automation, data processing, log analysis, gọi AI toolchain                    | Rất phù hợp để bắt đầu Test Development, trọng tâm bổ sung pytest, requests, Playwright, Locust                      |
| Java + Python | Muốn bao phủ rộng hơn các vị trí và project                                                                      | Java làm platform và engineering foundation, Python làm automation script và AI-assisted tool                        |

Nếu vốn là Java backend, đừng vứt bỏ Java rồi học lại Python chỉ vì Test Development. Kinh nghiệm về Spring Boot, MySQL, Redis, API design, unit test và log troubleshooting của Java đều có thể chuyển sang Test Development, đồng thời cũng dễ trình bày hơn trong CV.

Nếu chưa có nền tảng ngôn ngữ rõ ràng, khi chuẩn bị Test Development trong ngắn hạn có thể học Python trước. Python nhẹ hơn khi viết API automation, data processing, batch script và gọi AI API.

Ở giai đoạn này ít nhất cần đạt được:

- Có thể viết program cơ bản, hiểu function, class, exception, collection, file read/write và network request.
- Có thể dùng testing framework để viết unit test, chẳng hạn JUnit / Mockito của Java và pytest của Python.
- Có thể đóng gói HTTP request, xử lý Token, Header, Cookie, parameterization và assertion.
- Có thể đọc hiểu project directory structure, biết code, config, test và log lần lượt nằm ở đâu.
- Có thể viết một service đơn giản, chẳng hạn Spring Boot hoặc Flask / FastAPI, để luyện API testing.

Có thể xem nội dung liên quan đến Java:

- [Câu hỏi phỏng vấn Java Basics thường gặp](../java/basis/java-basic-questions-01.md)
- [Câu hỏi phỏng vấn Java Collection thường gặp](../java/collection/java-collection-questions-01.md)
- [Java unit test](../system-design/basis/unit-test.md)

## Giai đoạn ba: Bổ sung database, Linux, Git, Docker và CI/CD

Công việc Test Development không chỉ diễn ra trên browser page. Nhiều vấn đề cuối cùng đều liên quan đến data, environment, log và release process.

Về database, MySQL là trọng tâm. Ít nhất cần biết:

- Viết SQL thường gặp: query, filter, sort, pagination, aggregation và join query.
- Đọc hiểu table structure, biết ảnh hưởng của primary key, unique index, normal index và foreign key.
- Hiểu transaction, isolation level, dirty read, non-repeatable read và phantom read.
- Dùng SQL để tạo test data, kiểm tra API response và database state có nhất quán không.
- Có thể giải thích các vấn đề thường gặp như slow SQL, index failure và connection exhaustion.

Linux chủ yếu được dùng để deploy, xem log và troubleshooting. Cần thành thạo các command thường dùng: `cd`, `ls`, `cat`, `tail`, `grep`, `awk`, `sed`, `ps`, `top`, `df`, `du`, `curl`, `netstat` / `ss`.

Git cần biết branch, commit, merge, resolve conflict, rollback và xem commit history. Docker cần biết pull image, viết Dockerfile đơn giản, start container, mount config và xem log. CI/CD ít nhất phải biết pipeline được trigger thế nào, thực thi test ra sao, tạo report thế nào và định vị thế nào khi failure.

Tài liệu nên đọc kèm:

- [Câu hỏi phỏng vấn MySQL thường gặp](../database/mysql/mysql-questions-01.md)
- [Git tutorial cho người mới bắt đầu](../tools/git/git-intro.md)
- [Docker tutorial cho người mới bắt đầu](../tools/docker/docker-intro.md)

Bài tập ở giai đoạn này có thể rất cụ thể: tự viết một service nhỏ, dùng Docker khởi động MySQL và backend service, sau đó dùng GitHub Actions hoặc Jenkins chạy một nhóm API automation test. Dù chức năng rất nhỏ, chỉ cần chain hoàn chỉnh cũng tốt hơn nhiều so với chỉ học tool.

## Giai đoạn bốn: Testing theory và test case design

Testing theory không nên dừng ở việc học thuộc thuật ngữ. Nó giải quyết một vấn đề cụ thể: trước một function, làm sao đánh giá mình đã test đủ chưa?

Nội dung cơ bản bao gồm:

- Testing process: requirement analysis, test plan, test case design, execution, defect tracking, regression, release verification và retrospective.
- Testing classification: unit test, integration test, system test, acceptance test, regression test và smoke test.
- Testing type: functional, API, performance, security, compatibility, usability và stability.
- Test case design method: equivalence partitioning, boundary value, decision table, cause-effect graph, state transition, orthogonal experiment và error guessing.
- Defect management: defect title, reproduction steps, actual result, expected result, environment information, log và screenshot.

Trong phỏng vấn thường gặp các câu hỏi scenario như test elevator, water cup, login page, shopping cart, WeChat Moments, red packet và file upload. Khi trả lời, đừng chỉ liệt kê theo function point mà có thể triển khai theo các dimension:

- Functional flow: normal path và exception path.
- Data boundary: null value, quá dài, special character, duplicate và invalid format.
- Permission và security: chưa đăng nhập, privilege escalation, sensitive information và rate limiting.
- Compatibility: browser, system version, network state và screen size.
- Performance và stability: high concurrency, weak network, duplicate submission và long-running.
- Observability: log, instrumentation, alert và error code có thể giúp định vị không.

AI có thể giúp tạo bản nháp đầu tiên, nhưng bạn phải biết review. Ví dụ, khi yêu cầu AI tạo test point cho login page, AI có thể bao phủ account, password, verification code và remember login, nhưng thường bỏ sót risk control, rate limiting, account lock, third-party login, Token renewal, concurrent login và audit log.

Test Development phải có thể bổ sung những phần bị bỏ sót này.

## Giai đoạn năm: API testing và API automation

API automation là một trong những phần Test Development nên ưu tiên chinh phục nhất.

Lý do rất đơn giản: API ổn định hơn UI, tốc độ thực thi nhanh hơn và dễ tích hợp vào CI hơn. Automation test chủ lực của nhiều team chính là API regression.

Trước tiên bắt đầu từ tool. Các tool như Postman, Apifox, Reqable và Insomnia, ít nhất phải biết một tool, có thể hoàn thành API debugging, environment variable, pre-request script, post-request assertion và test collection execution.

Sau đó viết code. Python có thể dùng `requests + pytest`, Java có thể dùng `JUnit / TestNG + Rest Assured`. Một API automation framework tử tế ít nhất cần bao gồm:

- Environment config: test environment, staging environment, API domain, account và Token.
- Request encapsulation: xử lý thống nhất Header, Cookie, authentication, timeout và retry.
- Data management: chuẩn bị test data, cleanup, parameterization và database verification.
- Assertion system: status code, response field, business code, database state và message queue side effect.
- Report output: Allure, HTML report, failure log, request và response detail.
- CI integration: thực thi theo mỗi commit hoặc theo lịch hằng ngày, sau failure có thể định vị đến test case cụ thể.

Đừng chỉ assertion `status_code == 200`. Assertion thực sự có giá trị trong API testing phải chứng minh business state là chính xác. Ví dụ, sau khi API create order thực thi, cần verify order table, inventory change, payment state, message event hoặc audit log.

AI rất phù hợp để làm ba việc ở giai đoạn này:

- Dựa trên tài liệu OpenAPI / Swagger để tạo bản đầu tiên của test case.
- Dựa trên API response để tạo data model và assertion template.
- Giúp review xem test case có chỉ test success path hay không.

Nhưng test data, business assertion và environment cleanup phải do bạn tự kiểm soát. AI không biết trong hệ thống của bạn những field nào sẽ ảnh hưởng đến flow tiếp theo.

## Giai đoạn sáu: UI automation và App automation

UI automation có thể làm, nhưng đừng ngay từ đầu đặt toàn bộ regression lên UI.

Chi phí của UI automation cao hơn API automation. Page structure có thể thay đổi, element locator có thể mất hiệu lực, network và rendering tạo ra instability, nếu maintenance không tốt rất dễ biến thành “ngày nào cũng có người sửa script”. Vì vậy UI automation phù hợp hơn để bao phủ các main flow có value cao, ổn định và đi qua nhiều page, chẳng hạn login, place order, payment, approval và publish.

Web UI automation có thể tập trung vào:

- Playwright: modern Web automation tool, có wait mechanism, debugging experience, parallel execution và multi-browser support khá tốt.
- Selenium: xuất hiện lâu hơn, có nhiều existing project trong doanh nghiệp, nguyên lý cũng thường được hỏi trong phỏng vấn.
- Cypress: được frontend team sử dụng nhiều, phù hợp với end-to-end testing cho Web application.

Nếu bắt đầu học mới từ năm 2026, tôi đề xuất ưu tiên Playwright, sau đó tìm hiểu Selenium. Hệ sinh thái và giá trị phỏng vấn của Selenium vẫn còn, nhưng với stability và development experience của project mới, Playwright thường dễ chịu hơn.

App automation chủ yếu tìm hiểu Appium. Cần hiểu device connection, element locator, wait, swipe, permission popup, log capture, weak network và compatibility trên nhiều device model.

UI / App automation cần nắm trọng tâm:

- Page Object Model, đừng viết toàn bộ element locator và business step trong một file.
- Stable wait, hạn chế dùng `sleep` cố định.
- Lưu giữ screenshot, video, Trace và failure log.
- Test data isolation, tránh để các test case ảnh hưởng lẫn nhau.
- Parallel execution và failure retry, nhưng retry không được che giấu vấn đề thật.

AI có thể giúp tạo bản đầu tiên của script từ page structure, cũng có thể đoán vấn đề locator dựa trên failure screenshot. Nhưng stability của UI automation đến từ engineering design, tốc độ tạo script chỉ là điểm bắt đầu.

## Giai đoạn bảy: Performance testing và stability testing

Performance testing không thể chỉ dừng ở việc mở JMeter lên chạy một lần, sau đó dán một biểu đồ QPS.

Trước hết cần xác định rõ mục tiêu testing: verify capacity của một API, capacity của core chain, peak traffic, stability, rate limiting/degradation hay tìm bottleneck. Mục tiêu khác nhau thì performance test model cũng khác nhau.

Tool thường gặp:

- JMeter: rất phổ biến trong doanh nghiệp, phù hợp để load test API và Web service.
- k6: script experience hiện đại hơn, phù hợp với developer collaboration.
- Locust: viết scenario bằng Python, phù hợp để model hóa user behavior phức tạp.
- Gatling: performance tốt, phổ biến hơn trong Scala tech stack.

Các metric cần chú ý:

- Throughput: QPS, TPS.
- Response time: average, P95, P99.
- Error rate: HTTP error, business error và timeout.
- Resource metric: CPU, memory, disk I/O, network I/O, thread count và connection count.
- Dependency metric: database slow SQL, connection pool, Redis hit rate và message queue backlog.

Performance test report phải trả lời được một số câu hỏi:

- Business scenario của lần load test này là gì?
- Concurrent user, request ratio, data scale và duration là bao nhiêu?
- Bottleneck nằm ở đâu, evidence là gì?
- Metric thay đổi thế nào trước và sau optimization?
- Boundary của conclusion hiện tại là gì?

Load test không có monitoring sẽ có value rất thấp. Ít nhất phải xem được service log, system resource, database metric và error stack. Khi response chậm, trước tiên cần phán đoán đó là do application thread exhausted, database chậm, external API chậm, GC, network hay chính performance test script có vấn đề.

## Giai đoạn tám: CI/CD, quality platform và precise testing

Ranh giới để Test Development tiến lên thường nằm ngoài script: có thể tích lũy quality capability vào team process hay không.

Bước đầu tiên là CI/CD. Đưa API automation, unit test, static check, UI smoke và test report vào pipeline. Sau mỗi lần merge code, tự động chạy một nhóm test case quan trọng; khi failure có thể xem log, screenshot, request response và module phụ trách.

Bước thứ hai là test platform. Một platform đơn giản cũng có thể rất có giá trị, chẳng hạn:

- Test case management: duy trì API, scenario, priority, tag và execution status.
- Automated scheduling: trigger test theo project, branch, environment và tag.
- Report aggregation: hiển thị pass rate, failure reason và historical trend.
- Test data management: chuẩn bị data như account, order, inventory và approval flow.
- Defect linkage: tự động liên kết failure test case với defect hoặc thông báo cho người phụ trách.

Bước thứ ba là coverage và precise testing. Java có thể dùng JaCoCo, Python có thể dùng Coverage.py. Chúng cho biết automation test case đã cover những code line, branch và method nào. Tiến thêm một bước, có thể kết hợp change scope, call chain và historical failure record để ưu tiên thực thi các test case có khả năng phát hiện vấn đề cao hơn.

Precise testing không nhất thiết là yêu cầu bắt buộc với sinh viên mới tốt nghiệp, nhưng rất phù hợp làm project nâng cao. So với “tôi đã viết một API automation framework”, nếu có thể nói rõ “tôi lọc regression test case dựa trên code coverage và changed file” thì hàm lượng kỹ thuật sẽ cao hơn đáng kể.

## Giai đoạn chín: Sử dụng AI-assisted testing thế nào

AI-assisted testing không nên dừng ở “hãy viết test case giúp tôi”.

Cách dùng thực tế hơn là chia nhỏ task, để AI thực hiện candidate generation và hỗ trợ review:

```text
Hãy dựa trên mô tả requirement bên dưới để đưa ra test point.
Yêu cầu:
1. Phân loại theo function, API, permission, security, compatibility, performance và exception scenario;
2. Với mỗi test point, viết precondition, operation step và expected result;
3. Liệt kê riêng những điểm bạn chưa chắc chắn, cần product xác nhận;
4. Không bịa ra business rule không xuất hiện trong requirement.
```

API automation có thể dùng như sau:

```text
Hãy dựa trên tài liệu OpenAPI này để tạo bản nháp API test case bằng pytest.
Yêu cầu:
1. Phân biệt normal path và exception path;
2. Mỗi test case đều phải có assertion rõ ràng;
3. Không chỉ assertion HTTP 200;
4. Đánh dấu những nơi cần bổ sung test data thủ công.
```

Phân tích failure log có thể dùng như sau:

```text
Hãy phân tích failure log của CI lần này.
Yêu cầu:
1. Trước tiên phán đoán failure xảy ra ở environment, test data, assertion, API hay business code;
2. Đưa ra 3 nguyên nhân có khả năng nhất;
3. Đưa ra command troubleshooting tiếp theo hoặc log cần xem;
4. Không kết luận trực tiếp, đánh dấu “chưa chắc chắn” ở những chỗ thiếu evidence.
```

Giá trị của loại Prompt này là giúp AI liệt kê candidate space. Phán đoán thực sự vẫn đến từ log, data, code và business rule.

Tiến thêm một bước, có thể dùng AI để quality review. Ví dụ yêu cầu AI kiểm tra automated framework có hard-code environment không, test case có phụ thuộc lẫn nhau không, assertion có quá yếu không, failure retry có che giấu vấn đề không. Hướng này tương đồng với tư duy Spec Coding, code review và local validation trong [Hướng dẫn thực hành AI programming](../ai-coding/).

## Giai đoạn mười: AI application testing và LLM evaluation

Nếu muốn lộ trình Test Development phù hợp hơn với yêu cầu của kỷ nguyên AI, nhất định phải bổ sung phần này.

AI application testing trước tiên có thể chia thành bốn loại:

| Hướng          | Nội dung testing trọng tâm                                                                   | Ví dụ                                                    |
| -------------- | -------------------------------------------------------------------------------------------- | -------------------------------------------------------- |
| LLM API app    | Structured output, timeout retry, rate limiting, degradation, cost, audit                    | Resume parsing, customer service QA, text classification |
| RAG app        | Document parsing, chunking, retrieval, Rerank, answer faithfulness, citation traceability    | Enterprise knowledge base, policy QA                     |
| Agent app      | Tool selection, parameter generation, execution trace, permission boundary, failure recovery | Tự động tra order, tự động tạo report                    |
| Multimodal app | Image/audio input, recognition accuracy, abnormal file, security boundary                    | Image moderation, receipt recognition                    |

Ở đây không thể chỉ dùng tư duy truyền thống “input bằng output”. Output của AI application thường có nhiều câu trả lời chấp nhận được, evaluation cần giống một quality regression system hơn.

Có thể bắt đầu từ các concept sau:

- Golden Set: chuẩn bị một nhóm test set chất lượng cao, bao phủ normal problem, boundary problem, adversarial problem và high-risk business problem.
- Retrieval evaluation: xem đúng document có được retrieve không, vị trí trong TopK có đủ cao không.
- Generation evaluation: xem câu trả lời có đúng không, có faithful với tài liệu không, có trích dẫn evidence không.
- Tool calling evaluation: xem Agent có chọn đúng tool không, parameter có chính xác không, sau failure có recovery không.
- Security evaluation: prompt injection, jailbreak, sensitive information leakage và privilege escalation.

Về tool, có thể tìm hiểu các evaluation framework như DeepEval, RAGAS và promptfoo, cũng có thể tự viết một script nhẹ: đọc test set, gọi API của application theo batch, lưu question, answer, citation, duration, Token cost và human score.

Nội dung liên quan trong JavaGuide có thể xem:

- [Thực hành engineering khi gọi LLM API](../ai/llm-basis/llm-api-engineering.md)
- [Giải thích chi tiết structured output của LLM](../ai/llm-basis/structured-output-function-calling.md)
- [Hệ thống evaluation ứng dụng AI](../ai/llm-basis/llm-evaluation.md)
- [Concept cơ bản về RAG](../ai/rag/rag-basis.md)
- [Hướng dẫn thực hành prompt engineering cho LLM](../ai/agent/prompt-engineering.md)

Mục tiêu của giai đoạn này không nằm ở model algorithm. Test Development nên chú trọng hơn vào engineering quality: verify model output thế nào, phát hiện quality degradation ra sao, replay vấn đề production thế nào, chứng minh thế nào trước release rằng thay đổi lần này không làm hiệu quả kém đi.

## Nên thực hiện project thế nào

Điều tối kỵ trong CV Test Development là project quá hời hợt. Chỉ viết “thành thạo automated testing”, “biết sử dụng JMeter”, “hiểu AI testing” sẽ khiến interviewer khó đánh giá năng lực thực tế của bạn.

Cách tốt hơn là làm 2 đến 3 project có thể chạy được và giải thích rõ ràng.

### Project một: API automation testing framework

Chọn một hệ thống thực tế hoặc bán thực tế, chẳng hạn e-commerce, blog, online education hoặc admin management. Ít nhất bao phủ các API như login, user, product, order và payment callback.

Project cần bao gồm:

- Phân tích API document và thiết kế test case.
- Request encapsulation, authentication, parameterization, data preparation và cleanup.
- Database verification, không chỉ xem API response.
- Allure / HTML test report.
- GitHub Actions / Jenkins tự động thực thi.
- Lưu giữ failure log, request response và environment information.

Khi phỏng vấn có thể trình bày: thiết kế directory structure thế nào, xử lý Token ra sao, quản lý test data thế nào, tránh test case phụ thuộc lẫn nhau ra sao, tích hợp vào pipeline thế nào.

### Project hai: Web UI automation hoặc App automation

Với hướng Web, đề xuất dùng Playwright hoặc Selenium; hướng App có thể dùng Appium.

Đừng làm quá nhiều page, trước tiên hãy làm vững main flow. Ví dụ login, user management, role permission, article publishing và order processing của một admin management system.

Project cần bao gồm:

- Page Object Model.
- Multi-environment config.
- Screenshot, Trace, failure video hoặc log.
- Test case tag, chẳng hạn smoke, regression và core flow.
- Parallel execution và failure retry strategy.
- CI scheduled execution và report archival.

Khi phỏng vấn cần tập trung trình bày stability: chọn element locator thế nào, xử lý wait ra sao, maintenance thế nào sau khi page thay đổi, phán đoán thế nào để biết failure là do script hay business issue.

### Project ba: Performance testing và problem localization

Chọn một API chain, chẳng hạn product query, place order, login hoặc search.

Project cần bao gồm:

- Performance test scenario design: single API, mixed scenario, peak scenario và stability scenario.
- JMeter, k6 hoặc Locust script.
- Thu thập monitoring metric: CPU, memory, database, Redis và application log.
- Performance report: QPS, P95, P99, error rate và bottleneck analysis.
- Ít nhất một lần so sánh trước và sau optimization, chẳng hạn thêm index, điều chỉnh connection pool hoặc giảm slow SQL.

Project này rất phù hợp để chứng minh bạn biết định vị vấn đề, còn tool execution chỉ là một mắt xích trong đó.

### Project bốn: AI application quality evaluation platform

Nếu muốn làm nổi bật năng lực Test Development trong kỷ nguyên AI, có thể làm một evaluation platform nhẹ.

Ví dụ làm một tool evaluation cho RAG QA system:

- Hỗ trợ import Golden Set: question, standard answer và expected citation document.
- Gọi RAG API theo batch, ghi lại answer, citation fragment, duration và Token cost.
- Tính retrieval hit, xem answer có bao gồm key fact không, có trích dẫn đúng tài liệu không.
- Hỗ trợ human scoring và đánh dấu failure sample.
- Xuất comparison report: khác biệt hiệu quả giữa model A và model B, Prompt v1 và Prompt v2, các TopK config khác nhau.

Project này không cần làm quá lớn. Điều quan trọng là thể hiện bạn hiểu phương pháp quality evaluation của AI application, chứ không chỉ biết yêu cầu model trả lời câu hỏi.

## Trình bày năng lực Test Development khi phỏng vấn thế nào

Khi phỏng vấn Test Development, đừng mô tả bản thân là “người biết rất nhiều tool”. Tool chỉ là entry point, điều interviewer thực sự muốn biết là bạn có thể bảo đảm quality hay không.

Có thể tổ chức cách trình bày trong CV và phỏng vấn theo mạch sau:

- Tôi đã phụ trách quality assurance cho system hoặc module nào.
- Tôi phân tích requirement và thiết kế test point thế nào.
- Tôi đã xây dựng những automation capability nào, bao phủ những API hoặc flow nào.
- Automation được tích hợp vào CI/CD thế nào, sau failure thông báo và định vị ra sao.
- Tôi đã phát hiện vấn đề nào, định vị thế nào, cuối cùng sửa hoặc thúc đẩy sửa ra sao.
- Tôi đã nâng cao efficiency ở những điểm nào, chẳng hạn report aggregation, test data construction, coverage statistics và test case selection.
- Tôi sử dụng AI-assisted testing thế nào, đồng thời bảo đảm AI output được human và automation verification ra sao.

Nếu chuyển từ backend sang Test Development, có thể nhấn mạnh các ưu thế sau:

- Hiểu API design và backend implementation hơn, có thể định vị vấn đề từ code và log.
- Hiểu database, cache, message queue và distributed chain hơn, có thể test các risk bên ngoài bề mặt function.
- Có thể phát triển testing tool hoặc platform, công việc không chỉ là thực thi test case.

Nếu chuyển từ functional testing sang Test Development, có thể nhấn mạnh các ưu thế sau:

- Hiểu business flow và testing mindset hơn.
- Biết rõ hơn những scenario nào dễ bị bỏ sót.
- Automation cần tiếp nối kinh nghiệm manual hiện có, tích lũy phần có tần suất cao, ổn định và lặp lại được thành script và platform.

Các câu hỏi phỏng vấn thường gặp có thể chuẩn bị theo những hướng sau:

- Thiết kế test case cho login, shopping cart, elevator, water cup và file upload thế nào?
- Thiết kế API automation framework thế nào?
- pytest và unittest khác nhau thế nào? JUnit và Mockito phối hợp ra sao?
- Selenium, Playwright và Cypress khác nhau thế nào?
- UI automation không ổn định thì xử lý thế nào?
- Đọc JMeter performance test report thế nào? P95 và P99 lần lượt đại diện cho điều gì?
- Troubleshoot API thỉnh thoảng timeout thế nào?
- Bỏ sót bug production thì retrospective ra sao?
- Verify quality của test case do AI tạo thế nào?
- Testing RAG QA system hoặc Agent tool calling system thế nào?

## Nhịp học trong 3 đến 6 tháng

Nếu có thể duy trì học ổn định 2 đến 4 giờ mỗi ngày, có thể tiến hành theo nhịp sau.

Tháng 1, bổ sung fundamentals. Hoàn thành việc nhập môn một ngôn ngữ, có thể viết script và unit test; đồng thời bổ sung HTTP, Linux, MySQL, Git. Mục tiêu tháng này là viết được code, đọc hiểu log và debug API.

Tháng 2, chinh phục testing theory và API automation. Luyện có hệ thống test case design, hoàn thành một API automation framework, tích hợp test report và CI. Cuối tháng này, CV nên có một API automation project có thể giải thích rõ.

Tháng 3, thực hiện UI automation và performance testing. Với hướng Web ưu tiên Playwright, với performance chọn JMeter hoặc Locust. Đừng tham quá nhiều, hãy làm ổn định main flow như login, place order và query, sau đó thực hiện một performance test report hoàn chỉnh.

Tháng 4, bổ sung Quality Engineering. Học Jenkins / GitHub Actions, Docker, coverage và tư duy về test platform. Đưa hai project trước đó vào pipeline, tự động tạo report, khi failure có thể định vị.

Tháng 5, bổ sung AI-assisted testing và AI application testing. Dùng AI để tạo test case, review test code và phân tích log; sau đó làm một RAG hoặc Agent evaluation project nhỏ, hiểu Golden Set, retrieval, answer faithfulness và tool calling evaluation.

Tháng 6, tập trung chuẩn bị CV và phỏng vấn. Chỉnh project thành phiên bản có thể trình bày khi phỏng vấn: background, solution, technology selection, difficulty, result, risk và retrospective. Luyện các câu hỏi scenario testing phổ biến, câu hỏi algorithm cơ bản và câu hỏi đào sâu về project.

Nếu không đủ thời gian, ưu tiên là: programming language, API automation, testing theory, Linux / MySQL / Git và một project hoàn chỉnh. UI automation, performance testing, AI evaluation và test platform có thể bổ sung theo yêu cầu của vị trí mục tiêu.

## Một vài nhận định cuối cùng

Test Development không phù hợp với người chỉ muốn tìm được việc một cách nhẹ nhàng. Nó technical hơn functional testing truyền thống, đồng thời nhấn mạnh quality perspective hơn pure business development. Bạn vừa phải biết viết code, vừa phải sẵn sàng nghiền ngẫm nhiều lần về boundary, exception, failure và risk.

AI sẽ làm giảm repetitive work chất lượng thấp, nhưng sẽ không khiến quality assurance biến mất. Requirement understanding, test case design, assertion selection, risk judgment, problem localization và evaluation system vẫn cần con người phụ trách.

Nếu chuẩn bị đi theo lộ trình Test Development, nên sớm biến thành quả học tập thành project. Đừng chỉ bookmark roadmap, cũng đừng chỉ học thuộc tool tutorial. Một API automation framework, một UI automation main flow, một performance test report và một AI application evaluation project nhỏ có sức thuyết phục hơn nhiều so với “thành thạo một đống tool”.

Trước tiên hãy chạy thông một chain: từ requirement đến test case, từ API đến assertion, từ script đến CI, từ failure đến localization. Năng lực cạnh tranh của Test Development lớn dần từng chút một trong chính chain này.

## Tài liệu tham khảo

- [Tổng hợp cá nhân về lộ trình học và tài nguyên học tập dành cho Test Development engineer](https://www.nowcoder.com/discuss/585159)
- [Roadmap học AI testing của Black Horse Programmer (bản đầy đủ chính thức 2026)](https://yun.itheima.com/subject/testmap/index.html)
- [Lộ trình học testing và Test Development hoàn chỉnh (toàn nội dung thực tế)](https://www.nowcoder.com/discuss/787069493817229312)
