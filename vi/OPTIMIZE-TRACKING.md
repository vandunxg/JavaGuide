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
- Trạng thái: đang chờ review

|   # | File                                                     | Agent   | Trạng thái | Ghi chú |
| --: | -------------------------------------------------------- | ------- | ---------- | ------- |
|   1 | `snippets/article-header.snippet.md`                     | pending | pending    |         |
|   2 | `snippets/article-footer.snippet.md`                     | pending | pending    |         |
|   3 | `snippets/small-advertisement.snippet.md`                | pending | pending    |         |
|   4 | `snippets/planet.snippet.md`                             | pending | pending    |         |
|   5 | `snippets/planet2.snippet.md`                            | pending | pending    |         |
|   6 | `snippets/rag-project.snippet.md`                        | pending | pending    |         |
|   7 | `snippets/yuanma.snippet.md`                             | pending | pending    |         |
|   8 | `README.md`                                              | pending | pending    |         |
|   9 | `home.md`                                                | pending | pending    |         |
|  10 | `roadmap/README.md`                                      | pending | pending    |         |
|  11 | `roadmap/java-roadmap.md`                                | pending | pending    |         |
|  12 | `roadmap/java-to-ai-roadmap.md`                          | pending | pending    |         |
|  13 | `roadmap/backend-to-ai-agent-roadmap.md`                 | pending | pending    |         |
|  14 | `roadmap/full-stack-roadmap.md`                          | pending | pending    |         |
|  15 | `roadmap/test-development-roadmap.md`                    | pending | pending    |         |
|  16 | `java/README.md`                                         | pending | pending    |         |
|  17 | `java/basis/README.md`                                   | pending | pending    |         |
|  18 | `java/basis/java-basic-questions-01.md`                  | pending | pending    |         |
|  19 | `java/basis/java-basic-questions-02.md`                  | pending | pending    |         |
|  20 | `java/basis/java-basic-questions-03.md`                  | pending | pending    |         |
|  21 | `java/basis/java-keyword-summary.md`                     | pending | pending    |         |
|  22 | `java/basis/why-there-only-value-passing-in-java.md`     | pending | pending    |         |
|  23 | `java/basis/generics-and-wildcards.md`                   | pending | pending    |         |
|  24 | `java/basis/reflection.md`                               | pending | pending    |         |
|  25 | `java/basis/proxy.md`                                    | pending | pending    |         |
|  26 | `java/basis/spi.md`                                      | pending | pending    |         |
|  27 | `java/basis/serialization.md`                            | pending | pending    |         |
|  28 | `java/basis/bigdecimal.md`                               | pending | pending    |         |
|  29 | `java/basis/money-long-vs-bigdecimal.md`                 | pending | pending    |         |
|  30 | `java/basis/unsafe.md`                                   | pending | pending    |         |
|  31 | `java/basis/syntactic-sugar.md`                          | pending | pending    |         |
|  32 | `java/collection/README.md`                              | pending | pending    |         |
|  33 | `java/collection/java-collection-questions-01.md`        | pending | pending    |         |
|  34 | `java/collection/java-collection-questions-02.md`        | pending | pending    |         |
|  35 | `java/collection/java-collection-precautions-for-use.md` | pending | pending    |         |
|  36 | `java/collection/arraylist-source-code.md`               | pending | pending    |         |
|  37 | `java/collection/linkedlist-source-code.md`              | pending | pending    |         |
|  38 | `java/collection/hashmap-source-code.md`                 | pending | pending    |         |
|  39 | `java/collection/concurrent-hash-map-source-code.md`     | pending | pending    |         |
|  40 | `java/collection/linkedhashmap-source-code.md`           | pending | pending    |         |
|  41 | `java/collection/copyonwritearraylist-source-code.md`    | pending | pending    |         |
|  42 | `java/collection/arrayblockingqueue-source-code.md`      | pending | pending    |         |
|  43 | `java/collection/priorityqueue-source-code.md`           | pending | pending    |         |
|  44 | `java/collection/delayqueue-source-code.md`              | pending | pending    |         |
|  45 | `java/concurrent/README.md`                              | pending | pending    |         |
|  46 | `java/concurrent/java-concurrent-questions-01.md`        | pending | pending    |         |
|  47 | `java/concurrent/java-concurrent-questions-02.md`        | pending | pending    |         |
|  48 | `java/concurrent/java-concurrent-questions-03.md`        | pending | pending    |         |
|  49 | `java/concurrent/jmm.md`                                 | pending | pending    |         |
|  50 | `java/concurrent/java-lock.md`                           | pending | pending    |         |
