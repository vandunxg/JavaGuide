---
title: "Tổng hợp câu hỏi phỏng vấn RAG"
description: "Hệ thống hóa các câu hỏi phỏng vấn RAG thường gặp, bao quát các trọng tâm về nền tảng RAG, Embedding, vector database, chiến lược Chunk, xử lý tài liệu, Hybrid Search, Query Rewrite, Rerank, GraphRAG, cập nhật knowledge base và đánh giá RAG, kèm các bài viết tham khảo tương ứng."
category: AI
tag:
  - "Phỏng vấn RAG"
  - "vector database"
  - "Phỏng vấn AI"
head:
  - - meta
    - name: keywords
      content: "câu hỏi phỏng vấn RAG,câu hỏi phỏng vấn RAG,câu hỏi phỏng vấn Retrieval-Augmented Generation,câu hỏi phỏng vấn Embedding,câu hỏi phỏng vấn vector database,câu hỏi phỏng vấn GraphRAG,câu hỏi phỏng vấn tối ưu RAG,câu hỏi phỏng vấn Chunk,câu hỏi phỏng vấn Hybrid Search,câu hỏi phỏng vấn Rerank"
---

Một chain RAG phải xử lý việc phân tích tài liệu, Chunk, Embedding, index, recall, ranking, tổ chức context và generation. Sau khi hệ thống chạy một thời gian, còn phát sinh các vấn đề về phiên bản tài liệu, thay đổi quyền hạn, xây dựng lại index và đánh giá hiệu quả. Các câu hỏi được nhóm theo chương của chuyên đề RAG trên JavaGuide, mỗi nhóm đều kèm bài viết chi tiết.

## Nền tảng RAG

Nội dung liên quan: [《Khái niệm cơ bản về RAG: truy xuất, sinh và đánh đổi trong engineering》](../rag/rag-basis.md)

Câu hỏi nền tảng xoay quanh quy trình hoạt động và trường hợp sử dụng của RAG, đồng thời so sánh với search truyền thống, fine-tuning và long context. Hallucination, citation và từ chối trả lời thường được hỏi đào sâu ở phần sau.

Câu hỏi phỏng vấn thường gặp:

- RAG là gì? Vì sao cần RAG?
- RAG khác gì với search engine truyền thống?
- Nên chọn RAG hay fine-tuning? Khi nào dùng RAG, khi nào fine-tuning, khi nào kết hợp cả hai?
- Chọn model Embedding trong hệ thống RAG thế nào? Vì sao?
- Cosine similarity, inner product và khoảng cách Euclidean khác nhau thế nào?
- Giải quyết vấn đề hallucination của RAG thế nào? RAG có nhất thiết không tạo ra hallucination không?
- Vấn đề Lost in the Middle là gì? Ứng phó thế nào?
- Long context window có thay thế RAG không?
- Hệ thống RAG có những chỉ số đánh giá nào?
- Ưu điểm và hạn chế của RAG là gì?
- Trường hợp nào phù hợp để dùng RAG? Trường hợp nào không phù hợp?

## Vector database và index

Nội dung liên quan: [《Thuật toán vector index và vector database của RAG》](../rag/rag-vector-store.md)

Câu hỏi về vector retrieval thường đi từ Embedding và phép đo khoảng cách đến index ANN, rồi chuyển sang quy mô dữ liệu, query latency, điều kiện filter và chi phí vận hành. Chỉ ghi nhớ tên sản phẩm sẽ rất khó ứng phó với các câu hỏi đào sâu về lựa chọn công nghệ.

Câu hỏi phỏng vấn thường gặp:

- Embedding là gì? Vì sao cần chuyển text thành vector?
- Vì sao trường hợp sử dụng RAG cần vector database?
- Vì sao thuật toán ANN có thể chấp nhận kết quả không chính xác 100%?
- Có những thuật toán vector index nào? Ưu, nhược điểm của từng thuật toán là gì?
- Flat, HNSW, IVFFLAT, IVF-PQ lần lượt phù hợp với trường hợp nào?
- HNSW khác IVFFLAT thế nào?
- Điều chỉnh parameter `ef_search` của HNSW thế nào? Tăng và giảm lần lượt sẽ gây ra tác động gì?
- Khác biệt cốt lõi nhất giữa vector database và database truyền thống là gì?
- Nếu dữ liệu vector tăng từ 1 triệu lên 100 triệu, cần điều chỉnh gì về mặt kiến trúc?
- Vì sao chọn PostgreSQL + pgvector? Khi nào nên chuyển sang vector database chuyên dụng?

## Xử lý tài liệu và chiến lược Chunk

Nội dung liên quan: [《Xử lý tài liệu và chiến lược phân đoạn của RAG: từ phân tích, làm sạch, Chunking đến xử lý nội dung đa phương thức》](../rag/rag-document-processing.md)

Trước khi đi vào index, tài liệu phải trải qua các bước phân tích, làm sạch, cấu trúc hóa, phân đoạn và bổ sung metadata. Kích thước Chunk chỉ là một parameter trong số đó; cấp độ tiêu đề, bảng, code, số trang, phiên bản và quyền hạn cũng ảnh hưởng đến recall về sau.

Câu hỏi phỏng vấn thường gặp:

- Pipeline xử lý tài liệu RAG thường gồm những bước nào?
- Phân tích, làm sạch và cấu trúc hóa tài liệu lần lượt giải quyết vấn đề gì?
- Vì sao không thể chỉ phân đoạn Chunk theo độ dài cố định?
- Nên cân nhắc thế nào giữa kích thước Chunk, Overlap và ranh giới ngữ nghĩa?
- Xử lý bảng, code block, hình ảnh và nội dung đa phương thức trước khi đưa vào RAG thế nào?
- Làm thế nào để giữ lại cấp độ tiêu đề, số trang, nguồn và metadata quyền hạn trong giai đoạn xử lý tài liệu?
- Chất lượng Chunk kém sẽ gây ra những vấn đề gì cho recall và generation?
- Xây dựng một pipeline xử lý tài liệu cấp enterprise từ đầu thế nào?

## Tối ưu retrieval RAG

Nội dung liên quan: [《Tối ưu RAG: từ recall, ranking đến context engineering》](../rag/rag-optimization.md)

Câu hỏi về tối ưu retrieval trước hết phải phân biệt vấn đề về recall, ranking, context và generation. Hybrid Search, Query Rewrite, Rerank và context compression gặp lỗi ở các vị trí khác nhau, không thể dùng cùng một biện pháp để giải quyết mọi mẫu thất bại.

Câu hỏi phỏng vấn thường gặp:

- Recall của RAG thấp thì nên kiểm tra thế nào?
- Chiến lược Chunk, Metadata, Hybrid Search, Query Rewrite và Rerank lần lượt giải quyết vấn đề gì?
- Hybrid Search là gì? Kết hợp BM25 và vector retrieval thế nào?
- Query Rewrite, HyDE và Self-Query lần lượt phù hợp với trường hợp nào?
- Rerank giải quyết vấn đề gì? Vì sao không thể chỉ dựa vào thứ hạng similarity của vector?
- Context compression có giá trị gì? Khi nào nó làm giảm chất lượng câu trả lời?
- Vì sao tối ưu RAG nhất thiết phải xây dựng tập mẫu thất bại trước?
- Khi RAG production xuất hiện tình trạng “hỏi một đằng, trả lời một nẻo”, nên định vị theo quy trình nào?

## GraphRAG

Nội dung liên quan: [《GraphRAG: bổ sung vector retrieval bằng cấu trúc graph》](../rag/graphrag.md)

Câu hỏi về GraphRAG tập trung vào entity relationship, multi-hop reasoning và câu hỏi toàn cục; cũng thường hỏi sâu về cách trích xuất entity relationship, tạo community summary, filter quyền hạn và chi phí cần thiết cho các lần cập nhật sau này. Khi lựa chọn công nghệ, còn phải đối chiếu với vấn đề nghiệp vụ và retrieval chain hiện có.

Câu hỏi phỏng vấn thường gặp:

- GraphRAG giải quyết vấn đề gì? Khác gì với vector RAG tiêu chuẩn?
- Vì sao nói Chunk là những hòn đảo thông tin?
- Vì sao vector similarity không giỏi multi-hop reasoning?
- Entity, relationship và community discovery trong GraphRAG lần lượt là gì?
- Global retrieval và local retrieval khác nhau thế nào?
- Community summary của GraphRAG có giá trị gì? Chi phí nằm ở đâu?
- GraphRAG filter quyền hạn thế nào?
- Trường hợp nào phù hợp với GraphRAG? Trường hợp nào không phù hợp?
- Vì sao các hệ thống trưởng thành lại kết hợp keyword retrieval, vector retrieval, multi-vector retrieval và graph retrieval?

## Cập nhật knowledge base và đánh giá

Nội dung liên quan: [《Cập nhật tài liệu knowledge base của RAG thế nào: cập nhật incremental, version control, loại trùng lặp và xây dựng lại toàn bộ》](../rag/rag-knowledge-update.md), [《Hệ thống đánh giá ứng dụng AI: từ xây dựng Golden Set đến vòng lặp gray release online》](../llm-basis/llm-evaluation.md)

Sau khi knowledge base được đưa lên production, tài liệu, quyền hạn, model Embedding và chiến lược Chunk đều có thể thay đổi. Câu hỏi về cập nhật quan tâm đến cách duy trì tính nhất quán giữa dữ liệu và index; câu hỏi về đánh giá yêu cầu quan sát riêng chất lượng retrieval và chất lượng generation.

Câu hỏi phỏng vấn thường gặp:

- Vì sao knowledge base RAG không thể chỉ thêm mới mà không xóa?
- Nên chọn incremental update hay full rebuild thế nào?
- Sau khi nâng cấp model Embedding, vì sao thường cần xây dựng lại index?
- Thay đổi chiến lược Chunk sẽ ảnh hưởng đến dữ liệu lịch sử nào?
- Làm thế nào để tránh việc nhiều phiên bản của cùng một tài liệu được recall đồng thời?
- Thực hiện gray release, rollback và audit cho việc cập nhật knowledge base thế nào?
- Vì sao đánh giá RAG phải tách chất lượng retrieval và chất lượng generation?
- MRR, NDCG, Recall@K, Context Precision và Faithfulness lần lượt đo lường điều gì?

## Câu hỏi tổng hợp về troubleshooting

- Tài liệu gốc đã được đưa vào database nhưng các câu hỏi liên quan vẫn không recall được Chunk chính xác. Bạn sẽ bắt đầu kiểm tra từ những khâu nào?
- Tài liệu chính xác đã vào candidate pool nhưng luôn nằm ngoài TopK. Nên điều chỉnh recall hay đưa thêm Rerank?
- Kết quả retrieval chính xác nhưng model vẫn trích dẫn sai đoạn. Kiểm tra thứ tự context, việc cắt ngắn và ràng buộc instruction thế nào?
- Phiên bản mới và cũ của cùng một tài liệu được recall đồng thời. Có thể đã xảy ra vấn đề gì trong chain cập nhật dữ liệu và index?
- Khi chỉ có điểm đánh giá chất lượng của câu trả lời cuối cùng, làm thế nào xác định vấn đề nằm ở retrieval hay generation?
- Một loại câu hỏi cần tìm mối quan hệ xuyên qua nhiều tài liệu. Làm thế nào xác định nên tối ưu vector RAG hay đưa GraphRAG vào?
