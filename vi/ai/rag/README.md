---
title: "Chuyên đề RAG: xử lý tài liệu, vector database, GraphRAG, tối ưu retrieval và cập nhật knowledge base"
description: "Lộ trình học về RAG và retrieval-augmented generation để phỏng vấn, bao quát xử lý tài liệu, vector database, GraphRAG, tối ưu retrieval, cập nhật knowledge base và đánh giá RAG."
category: AI
tag:
  - RAG
  - vector database
  - Phát triển ứng dụng AI
sidebar: false
---

<!-- @include: @small-advertisement.snippet.md -->

RAG không chỉ là “chia nhỏ tài liệu + vector retrieval”. Parsing, phân quyền, ranking, generation, updating và evaluation đều ảnh hưởng đến câu trả lời cuối cùng.

**Chuyên đề RAG** này dành cho các scenario như hỏi đáp knowledge base doanh nghiệp, intelligent customer service, document assistant và internal search. Nội dung triển khai theo pipeline thực tế của tài liệu sau khi vào hệ thống: parsing, cleaning, chunking, vectorization, indexing, retrieval, reranking, generation, updating và evaluation.

## Dành cho ai

- Developer đang học hoặc triển khai hỏi đáp knowledge base bằng RAG.
- Engineer đã làm Demo “chia nhỏ tài liệu + vector retrieval” nhưng chưa quen với chất lượng retrieval, cập nhật tài liệu, tính nhất quán và evaluation.
- Người chuẩn bị câu hỏi phỏng vấn liên quan đến RAG, vector database, GraphRAG và knowledge base doanh nghiệp.

## Trọng tâm học

- Vấn đề hiệu quả của RAG cần được kiểm tra theo từng giai đoạn: document processing, Chunk, Embedding, retrieval, Rerank, context compression và generation.
- Việc chọn vector database cần dựa trên quy mô dữ liệu, điều kiện filtering, tần suất cập nhật, yêu cầu latency và chi phí vận hành.
- GraphRAG phù hợp hơn với scenario có quan hệ giữa các entity chặt chẽ, nhiều câu hỏi global và cần reasoning xuyên tài liệu.
- Cập nhật knowledge base không chỉ là ghi đè file, mà còn phải tính đến version, deduplication, incremental indexing, rollback và gray release.
- RAG evaluation cần đồng thời xem metrics của retrieval và generation, không thể chỉ dựa vào việc câu trả lời cuối cùng có “nghe có vẻ đúng” hay không.

## Thứ tự đọc đề xuất

1. [Khái niệm cơ bản về RAG: retrieval, generation và trade-off kỹ thuật](./rag-basis.md): trước tiên nắm quy trình cốt lõi, ưu điểm và giới hạn của RAG.
2. [Xử lý tài liệu và chiến lược chunking của RAG](./rag-document-processing.md): hiểu pipeline xử lý trước khi tài liệu được đưa vào index.
3. [Thuật toán vector indexing và vector database của RAG](./rag-vector-store.md): bổ sung kiến thức nền về vector indexing và lựa chọn database.
4. [Tối ưu RAG: từ retrieval, reranking đến context engineering](./rag-optimization.md): nắm retrieval, reranking, rewrite và context compression.
5. [GraphRAG: bổ sung vector retrieval bằng graph structure](./graphrag.md), [Chiến lược cập nhật tài liệu knowledge base RAG](./rag-knowledge-update.md): tìm hiểu sâu hơn về tổ chức knowledge phức tạp và cập nhật liên tục.

## Bài viết cốt lõi

- [Khái niệm cơ bản về RAG: retrieval, generation và trade-off kỹ thuật](./rag-basis.md): hiểu workflow, scenario phù hợp và giới hạn của RAG.
- [Xử lý tài liệu và chiến lược chunking của RAG](./rag-document-processing.md): bao quát parsing file, cleaning, structuring, chiến lược Chunking và xử lý nội dung multimodal.
- [Thuật toán vector indexing và vector database của RAG](./rag-vector-store.md): nắm nguyên lý của các thuật toán indexing như HNSW, IVFFLAT và học cách chọn vector database phù hợp.
- [Tối ưu RAG: từ retrieval, reranking đến context engineering](./rag-optimization.md): kiểm tra vấn đề retrieval xoay quanh chiến lược Chunk, Hybrid Search, Query Rewrite, Rerank và context compression.
- [GraphRAG: bổ sung vector retrieval bằng graph structure](./graphrag.md): hiểu RAG được dẫn dắt bởi knowledge graph, nắm entity, relation, community detection, global retrieval và local retrieval.
- [Chiến lược cập nhật tài liệu knowledge base RAG](./rag-knowledge-update.md): bao quát incremental update, version rollback, deduplication và gray release.

## Câu hỏi thường gặp

- Vì sao RAG vẫn hallucinate? Nên kiểm tra từ những khâu nào?
- Nên chia Chunk lớn hay nhỏ? Xử lý heading, table, code block và nội dung multimodal như thế nào?
- Vector retrieval, keyword retrieval và hybrid retrieval phù hợp với những scenario nào?
- Rerank có tác dụng gì? Khi nào đáng để đưa vào?
- GraphRAG khác RAG thông thường ở điểm nào?
- Làm thế nào để cập nhật knowledge base mà vẫn bảo đảm consistency, rollback và không downtime?
- Đánh giá chất lượng retrieval và chất lượng câu trả lời cuối cùng của ứng dụng RAG như thế nào?

## Chuyên đề liên quan

- [Hệ thống kiến thức về phát triển ứng dụng AI](../)
- [Chuyên đề nền tảng LLM](../llm-basis/)
- [Chuyên đề AI Agent](../agent/)
- [Chuyên đề câu hỏi phỏng vấn về phát triển ứng dụng AI](../interview-questions/)

<!-- @include: @article-footer.snippet.md -->
