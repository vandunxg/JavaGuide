# Theo dõi tối ưu bản dịch

## Quy tắc

- Đối chiếu từng file với `docs/<path>` trước khi sửa `vi/<path>`.
- Bám sát nội dung gốc; không tự thêm, lược bỏ hoặc diễn giải lan man.
- Giữ nguyên code, API, URL, link nội bộ, directive VuePress và cấu trúc Markdown.
- Giữ thuật ngữ kỹ thuật bằng English theo `CLAUDE.md` và `GLOSSARY.md`.
- Không để sót chữ Hán, trừ các ngoại lệ được quy định trong `CLAUDE.md`.
- Mỗi agent chỉ sửa file được giao và không chạy Git.
- Mỗi file dịch được commit riêng ngay sau khi agent hoàn tất và đã kiểm tra diff.

## Trạng thái

- Batch hiện tại: 2
- Mục tiêu batch: 50 file
- Cách chọn: theo thứ tự lộ trình trong `vi/PROGRESS.md`
- Trạng thái: hoàn tất batch 2
- Kết quả: 50 file đã tối ưu; không có file unchanged
- Kiểm tra batch 2: `git diff --check` đạt; `make check` exit 0; `make vi-build` thành công, 721 pages

|   # | File                                                     | Agent       | Trạng thái | Ghi chú                     |
| --: | -------------------------------------------------------- | ----------- | ---------- | --------------------------- |
|   1 | `snippets/article-header.snippet.md`                     | reviewer-01 | unchanged  | Đã đối chiếu, không cần sửa |
|   2 | `snippets/article-footer.snippet.md`                     | reviewer-02 | verified   | Commit riêng                |
|   3 | `snippets/small-advertisement.snippet.md`                | reviewer-03 | unchanged  | Đã đối chiếu, không cần sửa |
|   4 | `snippets/planet.snippet.md`                             | reviewer-04 | verified   | Commit riêng                |
|   5 | `snippets/planet2.snippet.md`                            | reviewer-05 | verified   | Commit riêng                |
|   6 | `snippets/rag-project.snippet.md`                        | reviewer-06 | verified   | Commit riêng                |
|   7 | `snippets/yuanma.snippet.md`                             | reviewer-07 | verified   | Commit riêng                |
|   8 | `README.md`                                              | reviewer-08 | verified   | Commit riêng                |
|   9 | `home.md`                                                | reviewer-09 | verified   | Commit riêng                |
|  10 | `roadmap/README.md`                                      | reviewer-10 | verified   | Commit riêng                |
|  11 | `roadmap/java-roadmap.md`                                | reviewer-11 | verified   | Commit riêng                |
|  12 | `roadmap/java-to-ai-roadmap.md`                          | reviewer-12 | verified   | Commit riêng                |
|  13 | `roadmap/backend-to-ai-agent-roadmap.md`                 | reviewer-13 | verified   | Commit riêng                |
|  14 | `roadmap/full-stack-roadmap.md`                          | reviewer-14 | verified   | Commit riêng                |
|  15 | `roadmap/test-development-roadmap.md`                    | reviewer-15 | verified   | Commit riêng                |
|  16 | `java/README.md`                                         | reviewer-16 | verified   | Commit riêng                |
|  17 | `java/basis/README.md`                                   | reviewer-17 | verified   | Commit riêng                |
|  18 | `java/basis/java-basic-questions-01.md`                  | reviewer-18 | verified   | Commit riêng                |
|  19 | `java/basis/java-basic-questions-02.md`                  | reviewer-19 | verified   | Commit riêng                |
|  20 | `java/basis/java-basic-questions-03.md`                  | reviewer-20 | verified   | Commit riêng                |
|  21 | `java/basis/java-keyword-summary.md`                     | reviewer-21 | verified   | Commit riêng                |
|  22 | `java/basis/why-there-only-value-passing-in-java.md`     | reviewer-22 | verified   | Commit riêng                |
|  23 | `java/basis/generics-and-wildcards.md`                   | reviewer-23 | verified   | Commit riêng                |
|  24 | `java/basis/reflection.md`                               | reviewer-24 | verified   | Commit riêng                |
|  25 | `java/basis/proxy.md`                                    | reviewer-25 | verified   | Commit riêng                |
|  26 | `java/basis/spi.md`                                      | reviewer-26 | verified   | Commit riêng                |
|  27 | `java/basis/serialization.md`                            | reviewer-27 | verified   | Commit riêng                |
|  28 | `java/basis/bigdecimal.md`                               | reviewer-28 | verified   | Commit riêng                |
|  29 | `java/basis/money-long-vs-bigdecimal.md`                 | reviewer-29 | verified   | Commit riêng                |
|  30 | `java/basis/unsafe.md`                                   | reviewer-30 | verified   | Commit riêng                |
|  31 | `java/basis/syntactic-sugar.md`                          | reviewer-31 | verified   | Commit riêng                |
|  32 | `java/collection/README.md`                              | reviewer-32 | verified   | Commit riêng                |
|  33 | `java/collection/java-collection-questions-01.md`        | reviewer-33 | verified   | Commit riêng                |
|  34 | `java/collection/java-collection-questions-02.md`        | reviewer-34 | verified   | Commit riêng                |
|  35 | `java/collection/java-collection-precautions-for-use.md` | reviewer-35 | verified   | Commit riêng                |
|  36 | `java/collection/arraylist-source-code.md`               | reviewer-36 | verified   | Commit riêng                |
|  37 | `java/collection/linkedlist-source-code.md`              | reviewer-37 | verified   | Commit riêng                |
|  38 | `java/collection/hashmap-source-code.md`                 | reviewer-38 | verified   | Commit riêng                |
|  39 | `java/collection/concurrent-hash-map-source-code.md`     | reviewer-39 | verified   | Commit riêng                |
|  40 | `java/collection/linkedhashmap-source-code.md`           | reviewer-40 | verified   | Commit riêng                |
|  41 | `java/collection/copyonwritearraylist-source-code.md`    | reviewer-41 | verified   | Commit riêng                |
|  42 | `java/collection/arrayblockingqueue-source-code.md`      | reviewer-42 | verified   | Commit riêng                |
|  43 | `java/collection/priorityqueue-source-code.md`           | reviewer-43 | verified   | Commit riêng                |
|  44 | `java/collection/delayqueue-source-code.md`              | reviewer-44 | verified   | Commit riêng                |
|  45 | `java/concurrent/README.md`                              | reviewer-45 | verified   | Commit riêng                |
|  46 | `java/concurrent/java-concurrent-questions-01.md`        | reviewer-46 | verified   | Commit riêng                |
|  47 | `java/concurrent/java-concurrent-questions-02.md`        | reviewer-47 | verified   | Commit riêng                |
|  48 | `java/concurrent/java-concurrent-questions-03.md`        | reviewer-48 | verified   | Commit riêng                |
|  49 | `java/concurrent/jmm.md`                                 | reviewer-49 | verified   | Commit riêng                |
|  50 | `java/concurrent/java-lock.md`                           | reviewer-50 | verified   | Commit riêng                |

## Batch 2 — 50 file tiếp theo

|   # | File                                                      | Agent        | Trạng thái | Ghi chú                |
| --: | --------------------------------------------------------- | ------------ | ---------- | ---------------------- |
|  51 | `java/concurrent/optimistic-lock-and-pessimistic-lock.md` | reviewer-51  | verified   | Đã đối chiếu và tối ưu |
|  52 | `java/concurrent/cas.md`                                  | reviewer-52  | verified   | Đã đối chiếu và tối ưu |
|  53 | `java/concurrent/aqs.md`                                  | reviewer-53  | verified   | Đã đối chiếu và tối ưu |
|  54 | `java/concurrent/reentrantlock.md`                        | reviewer-54  | verified   | Đã đối chiếu và tối ưu |
|  55 | `java/concurrent/atomic-classes.md`                       | reviewer-55  | verified   | Đã đối chiếu và tối ưu |
|  56 | `java/concurrent/threadlocal.md`                          | reviewer-56  | verified   | Đã đối chiếu và tối ưu |
|  57 | `java/concurrent/java-thread-pool-summary.md`             | reviewer-57  | verified   | Đã đối chiếu và tối ưu |
|  58 | `java/concurrent/java-thread-pool-best-practices.md`      | reviewer-58  | verified   | Đã đối chiếu và tối ưu |
|  59 | `java/concurrent/java-concurrent-collections.md`          | reviewer-59  | verified   | Đã đối chiếu và tối ưu |
|  60 | `java/concurrent/completablefuture-intro.md`              | reviewer-60  | verified   | Đã đối chiếu và tối ưu |
|  61 | `java/concurrent/virtual-thread.md`                       | reviewer-61  | verified   | Đã đối chiếu và tối ưu |
|  62 | `java/jvm/README.md`                                      | reviewer-62  | verified   | Đã đối chiếu và tối ưu |
|  63 | `java/jvm/memory-area.md`                                 | reviewer-63  | verified   | Đã đối chiếu và tối ưu |
|  64 | `java/jvm/jvm-garbage-collection.md`                      | reviewer-64  | verified   | Đã đối chiếu và tối ưu |
|  65 | `java/jvm/class-file-structure.md`                        | reviewer-65  | verified   | Đã đối chiếu và tối ưu |
|  66 | `java/jvm/class-loading-process.md`                       | reviewer-66  | verified   | Đã đối chiếu và tối ưu |
|  67 | `java/jvm/classloader.md`                                 | reviewer-67  | verified   | Đã đối chiếu và tối ưu |
|  68 | `java/jvm/jvm-parameters-intro.md`                        | reviewer-68  | verified   | Đã đối chiếu và tối ưu |
|  69 | `java/jvm/jvm-intro.md`                                   | reviewer-69  | verified   | Đã đối chiếu và tối ưu |
|  70 | `java/jvm/jdk-monitoring-and-troubleshooting-tools.md`    | reviewer-70  | verified   | Đã đối chiếu và tối ưu |
|  71 | `java/jvm/jvm-in-action.md`                               | reviewer-71  | verified   | Đã đối chiếu và tối ưu |
|  72 | `java/jvm/jvm-interview-questions.md`                     | reviewer-72  | verified   | Đã đối chiếu và tối ưu |
|  73 | `java/io/README.md`                                       | reviewer-73  | verified   | Đã đối chiếu và tối ưu |
|  74 | `java/io/io-basis.md`                                     | reviewer-74  | verified   | Đã đối chiếu và tối ưu |
|  75 | `java/io/io-design-patterns.md`                           | reviewer-75  | verified   | Đã đối chiếu và tối ưu |
|  76 | `java/io/io-model.md`                                     | reviewer-76  | verified   | Đã đối chiếu và tối ưu |
|  77 | `java/io/nio-basis.md`                                    | reviewer-77  | verified   | Đã đối chiếu và tối ưu |
|  78 | `java/new-features/README.md`                             | reviewer-78  | verified   | Đã đối chiếu và tối ưu |
|  79 | `java/new-features/java8-common-new-features.md`          | reviewer-79  | verified   | Đã đối chiếu và tối ưu |
|  80 | `java/new-features/java8-tutorial-translate.md`           | reviewer-80  | verified   | Đã đối chiếu và tối ưu |
|  81 | `java/new-features/java9.md`                              | reviewer-81  | verified   | Đã đối chiếu và tối ưu |
|  82 | `java/new-features/java10.md`                             | reviewer-82  | verified   | Đã đối chiếu và tối ưu |
|  83 | `java/new-features/java11.md`                             | reviewer-83  | verified   | Đã đối chiếu và tối ưu |
|  84 | `java/new-features/java12-13.md`                          | reviewer-84  | verified   | Đã đối chiếu và tối ưu |
|  85 | `java/new-features/java14-15.md`                          | reviewer-85  | verified   | Đã đối chiếu và tối ưu |
|  86 | `java/new-features/java16.md`                             | reviewer-86  | verified   | Đã đối chiếu và tối ưu |
|  87 | `java/new-features/java17.md`                             | reviewer-87  | verified   | Đã đối chiếu và tối ưu |
|  88 | `java/new-features/java18.md`                             | reviewer-88  | verified   | Đã đối chiếu và tối ưu |
|  89 | `java/new-features/java19.md`                             | reviewer-89  | verified   | Đã đối chiếu và tối ưu |
|  90 | `java/new-features/java20.md`                             | reviewer-90  | verified   | Đã đối chiếu và tối ưu |
|  91 | `java/new-features/java21.md`                             | reviewer-91  | verified   | Đã đối chiếu và tối ưu |
|  92 | `java/new-features/java22-23.md`                          | reviewer-92  | verified   | Đã đối chiếu và tối ưu |
|  93 | `java/new-features/java24.md`                             | reviewer-93  | verified   | Đã đối chiếu và tối ưu |
|  94 | `java/new-features/java25.md`                             | reviewer-94  | verified   | Đã đối chiếu và tối ưu |
|  95 | `java/new-features/java26.md`                             | reviewer-95  | verified   | Đã đối chiếu và tối ưu |
|  96 | `cs-basics/network/README.md`                             | reviewer-96  | verified   | Đã đối chiếu và tối ưu |
|  97 | `cs-basics/network/osi-and-tcp-ip-model.md`               | reviewer-97  | verified   | Đã đối chiếu và tối ưu |
|  98 | `cs-basics/network/application-layer-protocol.md`         | reviewer-98  | verified   | Đã đối chiếu và tối ưu |
|  99 | `cs-basics/network/tcp-connection-and-disconnection.md`   | reviewer-99  | verified   | Đã đối chiếu và tối ưu |
| 100 | `cs-basics/network/tcp-byte-stream-udp-datagram.md`       | reviewer-100 | verified   | Đã đối chiếu và tối ưu |
