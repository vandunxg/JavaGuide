# Tiến độ dịch JavaGuide → Tiếng Việt

> **Nguồn sự thật duy nhất** về trạng thái dịch. Agent/người nào tiếp tục công việc: **đọc file này đầu tiên**.
> Luật dịch: [`../CLAUDE.md`](../CLAUDE.md) · Thuật ngữ: [`GLOSSARY.md`](GLOSSARY.md)

**Nội dung**: 444/444 file · **Hạ tầng site**: xong · **Cập nhật**: 2026-09-18

Ký hiệu: `[ ]` chưa dịch · `[x]` xong · `[~]` đang dịch

---

## 🚦 BẮT ĐẦU TỪ ĐÂY

**File tiếp theo cần dịch**: Không còn file chưa dịch.

Xem toàn bộ việc còn lại: `make sync`

Quy trình mỗi file:

1. Đọc file gốc: `cat docs/<path>` (CodeGraph KHÔNG index `.md`, xem CLAUDE.md §6)
2. Dịch → ghi vào `vi/<path>` (đúng đường dẫn mirror)
3. Verify chữ Hán: `make check` → chỉ được còn ngoại lệ ở CLAUDE.md §3.3.1
4. Fix YAML frontmatter (CLAUDE.md §3.1.1 — **bắt buộc**, nếu không sẽ fail build)
5. `make vi-build` → phải "success"
6. Đánh dấu `[x]` vào file này + cập nhật số đếm ở trên

### Lệnh

| Lệnh          | Việc                                          |
| ------------- | --------------------------------------------- |
| `make help`   | Danh sách lệnh                                |
| `make cn-dev` | Site tiếng Trung → localhost:8080             |
| `make vi-dev` | Site tiếng Việt → localhost:8081/vi/          |
| `make build`  | Build cả hai site                             |
| `make sync`   | Sau `git pull`: file nào mới / gốc nào đã đổi |
| `make check`  | Quét chữ Hán còn sót                          |

> ⚠️ **KHÔNG dùng `pnpm vi:dev`** — script đã bị gỡ khỏi `package.json` để tránh
> conflict khi pull upstream. Dùng `make` (xem CLAUDE.md §9).

---

## ✅ Hạ tầng site VuePress (HOÀN TẤT — đã build thành công)

Chi tiết: [`../CLAUDE.md`](../CLAUDE.md) §8.

- [x] `.vuepress/config.ts` — `base:"/vi/"`, `dest:"./dist/vi"`, `lang:"vi-VN"`, loại PROGRESS/GLOSSARY khỏi pages
- [x] `.vuepress/theme.ts` — `docsDir:"vi"`, SEO/JSON-LD tiếng Việt, canonical `/vi`, `trimDescription()` đổi sang dấu câu Latin, `legacyRedirects` làm rỗng, `linksCheck` hạ xuống `warn`
- [x] `.vuepress/navbar.ts` — dịch xong
- [x] `.vuepress/sidebar/` — 11 file, dịch xong toàn bộ (289 chuỗi)
- [x] `.vuepress/components/` — dịch chuỗi UI (LayoutToggle, unlock ×2, LazyMermaid, ClickImagePreview)
- [x] `.vuepress/features/unlock/` — dịch comment
- [x] `.vuepress/styles/` — dịch comment (SCSS giữ nguyên → UI y hệt bản gốc)
- [x] `.vuepress/public/` — copy nguyên (logo, favicon, icon)
- [x] `package.json` — **đã revert về nguyên bản** (zero-conflict, xem CLAUDE.md §9)
- [x] `Makefile` — toàn bộ lệnh, thay cho npm scripts
- [x] `i18n/lang-switch/` — nút chuyển ngôn ngữ CN | VI trên **cả hai** site
- [x] `i18n/cn.config.ts` — wrapper config, chèn nút vào site CN mà không sửa `docs/`
- [x] `i18n/sync-check.mjs` — báo cáo file chưa dịch / gốc đã đổi sau khi pull

**Kiểm chứng đã chạy**:

- `make vi-build` → success, 11 trang, asset prefix `/vi/`, `<html lang="vi-VN">`
- `make cn-build` → success, **641 trang**, wrapper config load được config upstream
- Toggle: trang đã dịch → link đúng trang tương ứng; trang chưa dịch → nút VI mờ, trỏ về `/vi/` (không link gãy)
- `git diff` **rỗng hoàn toàn** → pull upstream không conflict

⚠️ **Việc phải làm khi dịch xong toàn bộ**: đổi `linksCheck.build` từ `"warn"` về `"error"` trong `vi/.vuepress/theme.ts`.

---

## Giai đoạn 0 — Snippets (dùng chung) ✅ 7/7

- [x] `snippets/article-header.snippet.md`
- [x] `snippets/article-footer.snippet.md`
- [x] `snippets/small-advertisement.snippet.md`
- [x] `snippets/planet.snippet.md`
- [x] `snippets/planet2.snippet.md`
- [x] `snippets/rag-project.snippet.md`
- [x] `snippets/yuanma.snippet.md`

## Giai đoạn 1 — Trục lộ trình học · 8/8 ✅

- [x] `README.md` — trang chủ
- [x] `home.md` — mục lục tổng
- [x] `roadmap/README.md` — tổng hợp lộ trình
- [x] `roadmap/java-roadmap.md` — ⭐ lộ trình Java backend (77KB)
- [x] `roadmap/java-to-ai-roadmap.md` — lộ trình AI cho dev Java/Go (63KB)
- [x] `roadmap/backend-to-ai-agent-roadmap.md` — đề xuất chuyển từ backend sang AI Agent
- [x] `roadmap/full-stack-roadmap.md` — lộ trình full-stack
- [x] `roadmap/test-development-roadmap.md` — lộ trình phát triển và kiểm thử

## Giai đoạn 2 — Java core · 80/80 ✅

### 2.0 Mục lục

- [x] `java/README.md`

### 2.1 Java Basics (15)

- [x] `java/basis/README.md`
- [x] `java/basis/java-basic-questions-01.md` (70KB)
- [x] `java/basis/java-basic-questions-02.md` (47KB)
- [x] `java/basis/java-basic-questions-03.md`
- [x] `java/basis/java-keyword-summary.md`
- [x] `java/basis/why-there-only-value-passing-in-java.md`
- [x] `java/basis/generics-and-wildcards.md`
- [x] `java/basis/reflection.md`
- [x] `java/basis/proxy.md`
- [x] `java/basis/spi.md`
- [x] `java/basis/serialization.md`
- [x] `java/basis/bigdecimal.md`
- [x] `java/basis/money-long-vs-bigdecimal.md`
- [x] `java/basis/unsafe.md`
- [x] `java/basis/syntactic-sugar.md`

### 2.2 Collection (13) ✅

- [x] `java/collection/README.md`
- [x] `java/collection/java-collection-questions-01.md`
- [x] `java/collection/java-collection-questions-02.md`
- [x] `java/collection/java-collection-precautions-for-use.md`
- [x] `java/collection/arraylist-source-code.md`
- [x] `java/collection/linkedlist-source-code.md`
- [x] `java/collection/hashmap-source-code.md`
- [x] `java/collection/concurrent-hash-map-source-code.md`
- [x] `java/collection/linkedhashmap-source-code.md`
- [x] `java/collection/copyonwritearraylist-source-code.md`
- [x] `java/collection/arrayblockingqueue-source-code.md`
- [x] `java/collection/priorityqueue-source-code.md`
- [x] `java/collection/delayqueue-source-code.md`

### 2.3 Concurrent (17) ✅

- [x] `java/concurrent/README.md`
- [x] `java/concurrent/java-concurrent-questions-01.md`
- [x] `java/concurrent/java-concurrent-questions-02.md` (66KB)
- [x] `java/concurrent/java-concurrent-questions-03.md` (97KB)
- [x] `java/concurrent/jmm.md`
- [x] `java/concurrent/java-lock.md`
- [x] `java/concurrent/optimistic-lock-and-pessimistic-lock.md`
- [x] `java/concurrent/cas.md`
- [x] `java/concurrent/aqs.md` (104KB)
- [x] `java/concurrent/reentrantlock.md` (48KB)
- [x] `java/concurrent/atomic-classes.md`
- [x] `java/concurrent/threadlocal.md`
- [x] `java/concurrent/java-thread-pool-summary.md` (52KB)
- [x] `java/concurrent/java-thread-pool-best-practices.md`
- [x] `java/concurrent/java-concurrent-collections.md`
- [x] `java/concurrent/completablefuture-intro.md`
- [x] `java/concurrent/virtual-thread.md`

### 2.4 JVM (11) ✅

- [x] `java/jvm/README.md`
- [x] `java/jvm/memory-area.md`
- [x] `java/jvm/jvm-garbage-collection.md`
- [x] `java/jvm/class-file-structure.md`
- [x] `java/jvm/class-loading-process.md`
- [x] `java/jvm/classloader.md`
- [x] `java/jvm/jvm-parameters-intro.md`
- [x] `java/jvm/jvm-intro.md`
- [x] `java/jvm/jdk-monitoring-and-troubleshooting-tools.md`
- [x] `java/jvm/jvm-in-action.md`
- [x] `java/jvm/jvm-interview-questions.md`

### 2.5 IO (5) ✅

- [x] `java/io/README.md`
- [x] `java/io/io-basis.md`
- [x] `java/io/io-design-patterns.md`
- [x] `java/io/io-model.md`
- [x] `java/io/nio-basis.md`

### 2.6 New Features (18) ✅

- [x] `java/new-features/README.md`
- [x] `java/new-features/java8-common-new-features.md`
- [x] `java/new-features/java8-tutorial-translate.md`
- [x] `java/new-features/java9.md`
- [x] `java/new-features/java10.md`
- [x] `java/new-features/java11.md`
- [x] `java/new-features/java12-13.md`
- [x] `java/new-features/java14-15.md`
- [x] `java/new-features/java16.md`
- [x] `java/new-features/java17.md`
- [x] `java/new-features/java18.md`
- [x] `java/new-features/java19.md`
- [x] `java/new-features/java20.md`
- [x] `java/new-features/java21.md`
- [x] `java/new-features/java22-23.md`
- [x] `java/new-features/java24.md`
- [x] `java/new-features/java25.md`
- [x] `java/new-features/java26.md`

---

## Giai đoạn 3 — CS basics · 68 file đã dịch

### Network

- [x] `cs-basics/network/README.md`
- [x] `cs-basics/network/osi-and-tcp-ip-model.md`
- [x] `cs-basics/network/application-layer-protocol.md`
- [x] `cs-basics/network/tcp-connection-and-disconnection.md`
- [x] `cs-basics/network/tcp-byte-stream-udp-datagram.md`
- [x] `cs-basics/network/http1.0-vs-http1.1.md`
- [x] `cs-basics/network/other-network-questions.md`
- [x] `cs-basics/network/other-network-questions2.md`
- [x] `cs-basics/network/the-whole-process-of-accessing-web-pages.md`
- [x] `cs-basics/network/http-vs-https.md`
- [x] `cs-basics/network/https-rsa-vs-ecdhe.md`
- [x] `cs-basics/network/http-status-codes.md`
- [x] `cs-basics/network/tcp-reliability-guarantee.md`
- [x] `cs-basics/network/tcp-time-wait.md`
- [x] `cs-basics/network/tcp-keepalive-vs-http-keepalive.md`
- [x] `cs-basics/network/dns.md`
- [x] `cs-basics/network/http-vs-rpc.md`
- [x] `cs-basics/network/arp.md`
- [x] `cs-basics/network/nat.md`
- [x] `cs-basics/network/network-attack-means.md`
- [x] `cs-basics/network/computer-network-xiexiren-summary.md`
- [x] `cs-basics/network/can-ping-but-tcp-may-not-connect.md`
- [x] `cs-basics/network/can-tcp-and-udp-use-the-same-port.md`
- [x] `cs-basics/network/maximum-number-of-tcp-connections-per-host.md`

### Operating System

- [x] `cs-basics/operating-system/README.md`
- [x] `cs-basics/operating-system/operating-system-basic-questions-01.md`
- [x] `cs-basics/operating-system/operating-system-basic-questions-02.md`
- [x] `cs-basics/operating-system/process-and-thread.md`
- [x] `cs-basics/operating-system/interrupt-exception-syscall.md`
- [x] `cs-basics/operating-system/cpu-scheduling-and-load.md`
- [x] `cs-basics/operating-system/ipc.md`
- [x] `cs-basics/operating-system/os-lock-and-sync.md`
- [x] `cs-basics/operating-system/dead-lock.md`
- [x] `cs-basics/operating-system/memory-management.md`
- [x] `cs-basics/operating-system/virtual-memory.md`
- [x] `cs-basics/operating-system/file-system.md`
- [x] `cs-basics/operating-system/io-multiplexing.md`
- [x] `cs-basics/operating-system/zero-copy.md`
- [x] `cs-basics/operating-system/linux-intro.md`
- [x] `cs-basics/operating-system/shell-intro.md`

### Algorithms

- [x] `cs-basics/algorithms/README.md`
- [x] `cs-basics/algorithms/complexity-analysis.md`
- [x] `cs-basics/algorithms/binary-search.md`
- [x] `cs-basics/algorithms/two-pointers-and-sliding-window.md`
- [x] `cs-basics/algorithms/dfs-bfs.md`
- [x] `cs-basics/algorithms/backtracking.md`
- [x] `cs-basics/algorithms/dynamic-programming.md`
- [x] `cs-basics/algorithms/greedy.md`
- [x] `cs-basics/algorithms/top-k.md`
- [x] `cs-basics/algorithms/string-algorithm-problems.md`
- [x] `cs-basics/algorithms/linkedlist-algorithm-problems.md`
- [x] `cs-basics/algorithms/10-classical-sorting-algorithms.md`
- [x] `cs-basics/algorithms/classical-algorithm-problems-recommendations.md`
- [x] `cs-basics/algorithms/common-data-structures-leetcode-recommendations.md`
- [x] `cs-basics/algorithms/the-sword-refers-to-offer.md`

### Data Structure

- [x] `cs-basics/data-structure/README.md`
- [x] `cs-basics/data-structure/linear-data-structure.md`
- [x] `cs-basics/data-structure/hash-table.md`
- [x] `cs-basics/data-structure/tree.md`
- [x] `cs-basics/data-structure/heap.md`
- [x] `cs-basics/data-structure/red-black-tree.md`
- [x] `cs-basics/data-structure/skip-list.md`
- [x] `cs-basics/data-structure/trie.md`
- [x] `cs-basics/data-structure/union-find.md`
- [x] `cs-basics/data-structure/bloom-filter.md`
- [x] `cs-basics/data-structure/graph.md`
- [x] `cs-basics/data-structure/lru-cache.md`

---

## Giai đoạn 4 — Database · 44 file đã dịch

- [x] `database/README.md`, `database/basis.md`, `database/nosql.md`, `database/character-set.md`
- [x] `database/mysql/` — 16 file đã dịch
- [x] `database/redis/` — 13 file đã dịch
- [x] `database/sql/` — 7 file đã dịch
- [x] `database/mongodb/` — 3 file đã dịch
- [x] `database/elasticsearch/elasticsearch-questions-01.md`
- [x] `database/mysql/mysql-questions-01.md`
- [x] `database/redis/3-commonly-used-cache-read-and-write-strategies.md`

## Giai đoạn 5 — System design · 36 file đã dịch

- [x] `system-design/` — 36 file đã dịch

## Giai đoạn 6 — Distributed system · 28 file đã dịch

- [x] `distributed-system/` — các file nền tảng và protocol đã dịch trước đó
- [x] `distributed-system/microservices-interview-questions.md`
- [x] `distributed-system/rpc/README.md`
- [x] `distributed-system/rpc/rpc-intro.md`
- [x] `distributed-system/rpc/dubbo.md`
- [x] `distributed-system/api-gateway.md`
- [x] `distributed-system/spring-cloud-gateway-questions.md`
- [x] `distributed-system/distributed-id.md`
- [x] `distributed-system/distributed-id-design.md`
- [x] `distributed-system/distributed-lock.md`
- [x] `distributed-system/distributed-lock-implementations.md`
- [x] `distributed-system/distributed-transaction.md`
- [x] `distributed-system/distributed-configuration-center.md`
- [x] `distributed-system/distributed-process-coordination/zookeeper/README.md`
- [x] `distributed-system/distributed-process-coordination/zookeeper/zookeeper-intro.md`
- [x] `distributed-system/distributed-process-coordination/zookeeper/zookeeper-plus.md`
- [x] `distributed-system/distributed-process-coordination/zookeeper/zookeeper-in-action.md`

## Giai đoạn 6 — High-performance · 15/15 ✅

- [x] `high-performance/high-performance-system-interview-questions.md`
- [x] `high-performance/cdn.md`
- [x] `high-performance/load-balancing.md`
- [x] `high-performance/read-and-write-separation-and-library-subtable.md`
- [x] `high-performance/sql-optimization.md`
- [x] `high-performance/deep-pagination-optimization.md`
- [x] `high-performance/message-queue/message-queue-interview-questions.md`
- [x] `high-performance/message-queue/message-queue.md`
- [x] `high-performance/message-queue/kafka-questions-01.md`
- [x] `high-performance/message-queue/rocketmq-questions.md`
- [x] `high-performance/message-queue/rabbitmq-questions.md`
- [x] `high-performance/message-queue/disruptor-questions.md`
- [x] `high-performance/data-cold-hot-separation.md`
- [x] `high-performance/README.md`
- [x] `high-performance/message-queue/README.md`

### High-availability · 9/9 ✅

- [x] `high-availability/README.md`
- [x] `high-availability/limit-request.md`
- [x] `high-availability/idempotency.md`
- [x] `high-availability/redundancy.md`
- [x] `high-availability/high-availability-system-interview-questions.md`
- [x] `high-availability/fallback-and-circuit-breaker.md`
- [x] `high-availability/performance-test.md`
- [x] `high-availability/high-availability-system-design.md`
- [x] `high-availability/timeout-and-retry.md`

## Giai đoạn 7 — AI · 38/38 ✅

- [x] `ai/README.md`
- [x] `ai/ai-core-concepts.md`
- [x] `ai/interview-questions/README.md`
- [x] `ai/interview-questions/ai-interview-guide.md`
- [x] `ai/llm-basis/README.md`
- [x] `ai/llm-basis/llm-operation-mechanism.md`
- [x] `ai/llm-basis/llm-api-engineering.md`
- [x] `ai/agent/README.md`
- [x] `ai/agent/agent-basis.md`
- [x] `ai/agent/prompt-engineering.md`
- [x] `ai/agent/context-engineering.md`
- [x] `ai/agent/multi-agent.md`
- [x] `ai/rag/README.md`
- [x] `ai/rag/rag-basis.md`
- [x] `ai/rag/rag-document-processing.md`
- [x] `ai/rag/rag-optimization.md`
- [x] `ai/system-design/README.md`
- [x] `ai/system-design/ai-application-architecture.md`
- [x] `ai/system-design/llm-security.md`
- [x] `ai/system-design/llm-gateway.md`
- [x] `ai/llm-basis/structured-output-function-calling.md`
- [x] `ai/llm-basis/llm-evaluation.md`
- [x] `ai/agent/agent-memory.md`
- [x] `ai/agent/skills.md`
- [x] `ai/agent/mcp.md`
- [x] `ai/agent/harness-engineering.md`
- [x] `ai/agent/workflow-graph-loop.md`
- [x] `ai/agent/loop-engineering.md`
- [x] `ai/rag/rag-vector-store.md`
- [x] `ai/rag/graphrag.md`
- [x] `ai/rag/rag-knowledge-update.md`
- [x] `ai/system-design/ai-observability.md`
- [x] `ai/system-design/ai-voice.md`
- [x] `ai/interview-questions/llm-interview-questions.md`
- [x] `ai/interview-questions/agent-interview-questions.md`
- [x] `ai/interview-questions/agent-project-interview-guide.md`
- [x] `ai/interview-questions/rag-interview-questions.md`
- [x] `ai/interview-questions/ai-system-design-interview-questions.md`

## AI coding · 28/28 ✅

- [x] `ai-coding/README.md`
- [x] `ai-coding/practices/ai-ide.md`
- [x] `ai-coding/practices/cli-vs-ide.md`
- [x] `ai-coding/practices/ghostty.md`
- [x] `ai-coding/practices/claudecode-tips.md`
- [x] `ai-coding/practices/claudecode-commands.md`
- [x] `ai-coding/practices/oh-my-pi.md`
- [x] `ai-coding/practices/claudecode-agentview.md`
- [x] `ai-coding/practices/claude-md-best-practices.md`
- [x] `ai-coding/principles/claude-code-context-management.md`
- [x] `ai-coding/principles/claude-code-memory.md`
- [x] `ai-coding/principles/claude-code-skills.md`
- [x] `ai-coding/principles/claude-code-hooks.md`
- [x] `ai-coding/principles/claude-code-multi-agent.md`
- [x] `ai-coding/practices/programmer-essential-skills.md`
- [x] `ai-coding/practices/skill-selection-and-pruning.md`
- [x] `ai-coding/practices/mattpocock-skills.md`
- [x] `ai-coding/practices/drawio-chart-skill.md`
- [x] `ai-coding/practices/codex-best-practices.md`
- [x] `ai-coding/practices/spec-coding.md`
- [x] `ai-coding/practices/the-cool-tricks-for-vibe-coding.md`
- [x] `ai-coding/cases/idea-qoder-plugin.md`
- [x] `ai-coding/cases/trae-m2.7.md`
- [x] `ai-coding/cases/cc-glm5.1.md`
- [x] `ai-coding/cases/deepseek-v4-claude-code.md`
- [x] `ai-coding/cases/cc-m3.md`
- [x] `ai-coding/cases/kimi-k3.md`
- [x] `ai-coding/project/cc-guide.md`

## Tools · 11/11 ✅

- [x] `tools/README.md`
- [x] `tools/maven/README.md`
- [x] `tools/maven/maven-core-concepts.md`
- [x] `tools/maven/maven-best-practices.md`
- [x] `tools/gradle/gradle-core-concepts.md`
- [x] `tools/git/README.md`
- [x] `tools/git/git-intro.md`
- [x] `tools/git/github-tips.md`
- [x] `tools/docker/README.md`
- [x] `tools/docker/docker-intro.md`
- [x] `tools/docker/docker-in-action.md`

## Books · 7/7 ✅

- [x] `books/README.md`
- [x] `books/cs-basics.md`
- [x] `books/database.md`
- [x] `books/distributed-system.md`
- [x] `books/java.md`
- [x] `books/search-engine.md`
- [x] `books/software-quality.md`

## Interview preparation · 13/13 ✅

- [x] `interview-preparation/README.md`
- [x] `interview-preparation/backend-interview-plan.md`
- [x] `interview-preparation/backend-project-interview-guide.md`
- [x] `interview-preparation/how-to-handle-interview-nerves.md`
- [x] `interview-preparation/internship-experience.md`
- [x] `interview-preparation/interview-experience.md`
- [x] `interview-preparation/java-roadmap.md`
- [x] `interview-preparation/key-points-of-interview.md`
- [x] `interview-preparation/pdf-interview-javaguide.md`
- [x] `interview-preparation/project-experience-guide.md`
- [x] `interview-preparation/resume-guide.md`
- [x] `interview-preparation/self-test-of-common-interview-questions.md`
- [x] `interview-preparation/teach-you-how-to-prepare-for-the-interview-hand-in-hand.md`

## High-quality technical articles · 27/27 ✅

- [x] `high-quality-technical-articles/README.md`
- [x] `high-quality-technical-articles/advanced-programmer/20-bad-habits-of-bad-programmers.md`
- [x] `high-quality-technical-articles/advanced-programmer/meituan-three-year-summary-lesson-10.md`
- [x] `high-quality-technical-articles/advanced-programmer/programmer-quickly-learn-new-technology.md`
- [x] `high-quality-technical-articles/advanced-programmer/seven-tips-for-becoming-an-advanced-programmer.md`
- [x] `high-quality-technical-articles/advanced-programmer/ten-years-of-dachang-growth-road.md`
- [x] `high-quality-technical-articles/advanced-programmer/the-growth-strategy-of-the-technological-giant.md`
- [x] `high-quality-technical-articles/advanced-programmer/thinking-about-technology-and-business-after-five-years-of-work.md`
- [x] `high-quality-technical-articles/interview/how-to-examine-the-technical-ability-of-programmers-in-the-first-test-of-technology.md`
- [x] `high-quality-technical-articles/interview/my-personal-experience-in-2021.md`
- [x] `high-quality-technical-articles/interview/screen-candidates-for-packaging.md`
- [x] `high-quality-technical-articles/interview/some-secrets-about-alibaba-interview.md`
- [x] `high-quality-technical-articles/interview/summary-of-spring-recruitment.md`
- [x] `high-quality-technical-articles/interview/technical-preliminary-preparation.md`
- [x] `high-quality-technical-articles/interview/the-experience-and-thinking-of-an-interview-experienced-by-an-older-programmer.md`
- [x] `high-quality-technical-articles/interview/the-experience-of-get-offer-from-over-20-big-companies.md`
- [x] `high-quality-technical-articles/personal-experience/8-years-programmer-work-summary.md`
- [x] `high-quality-technical-articles/personal-experience/four-year-work-in-tencent-summary.md`
- [x] `high-quality-technical-articles/personal-experience/huawei-od-275-days.md`
- [x] `high-quality-technical-articles/personal-experience/two-years-of-back-end-develop--experience-in-didi-and-toutiao.md`
- [x] `high-quality-technical-articles/programmer/efficient-book-publishing-and-practice-guide.md`
- [x] `high-quality-technical-articles/programmer/high-value-certifications-for-programmers.md`
- [x] `high-quality-technical-articles/programmer/how-do-programmers-publish-a-technical-book.md`
- [x] `high-quality-technical-articles/programmer/programmer-career-directions.md`
- [x] `high-quality-technical-articles/work/32-tips-improving-career.md`
- [x] `high-quality-technical-articles/work/employee-performance.md`
- [x] `high-quality-technical-articles/work/get-into-work-mode-quickly-when-you-join-a-company.md`

## About the author · 11/11 ✅

- [x] `about-the-author/README.md`
- [x] `about-the-author/deprecated-java-technologies.md`
- [x] `about-the-author/dog-that-copies-other-people-essay.md`
- [x] `about-the-author/feelings-after-one-month-of-induction-training.md`
- [x] `about-the-author/feelings-of-half-a-year-from-graduation-to-entry.md`
- [x] `about-the-author/internet-addiction-teenager.md`
- [x] `about-the-author/javaguide-100k-star.md`
- [x] `about-the-author/my-article-was-stolen-and-made-into-video-and-it-became-popular.md`
- [x] `about-the-author/my-college-life.md`
- [x] `about-the-author/writing-technology-blog-six-years.md`
- [x] `about-the-author/zhishixingqiu-two-years.md`

## JavaGuide · 5/5 ✅

- [x] `javaguide/contribution-guideline.md`
- [x] `javaguide/faq.md`
- [x] `javaguide/history.md`
- [x] `javaguide/intro.md`
- [x] `javaguide/use-suggestion.md`

## Open-source project · 8/8 ✅

- [x] `open-source-project/README.md`
- [x] `open-source-project/big-data.md`
- [x] `open-source-project/machine-learning.md`
- [x] `open-source-project/practical-project.md`
- [x] `open-source-project/system-design.md`
- [x] `open-source-project/tool-library.md`
- [x] `open-source-project/tools.md`
- [x] `open-source-project/tutorial.md`

## Zhuanlan · 6/6 ✅

- [x] `zhuanlan/README.md`
- [x] `zhuanlan/back-end-interview-high-frequency-system-design-and-scenario-questions.md`
- [x] `zhuanlan/handwritten-rpc-framework.md`
- [x] `zhuanlan/interview-guide.md`
- [x] `zhuanlan/java-mian-shi-zhi-bei.md`
- [x] `zhuanlan/source-code-reading.md`

### Kiểm định ngữ nghĩa

- [x] Đối chiếu toàn bộ các file đã dịch trong roadmap, Java core, CS basics, Database, System design và Distributed system với file gốc.
- [x] Sửa các lỗi sai nghĩa, thiếu ý, thêm ý, sai công thức, sai thuật ngữ, sai URL và sai ngữ cảnh đã phát hiện.
- [x] `make vi-build` — thành công sau kiểm định.

Đã hoàn tất toàn bộ 444 file tài liệu.
