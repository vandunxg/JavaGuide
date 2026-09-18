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

- Batch hiện tại: 1
- Mục tiêu batch: 50 file
- Cách chọn: theo thứ tự lộ trình trong `vi/PROGRESS.md`
- Trạng thái: hoàn tất batch, đã dừng theo yêu cầu
- Kết quả: 48 file đã tối ưu và commit riêng; 2 file unchanged
- Kiểm tra: `git diff --check` đạt; `make vi-build` thành công, 721 pages

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
