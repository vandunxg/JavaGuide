---
title: "Chuyên đề Docker: container, image, repository, data volume, network và triển khai container hóa"
description: "Lộ trình học Docker về phỏng vấn và container hóa, bao quát container, image, repository, Docker Engine, data volume, network, các command thường dùng và thực hành triển khai container hóa."
category: Công cụ phát triển
tag:
  - Docker
  - container
  - triển khai
sitemap:
  changefreq: weekly
  priority: 0.85
head:
  - - meta
    - name: keywords
      content: Docker,container,image,repository,Docker Engine,data volume,network,containerization,tính nhất quán môi trường,backend development
---

Docker là công cụ container hóa rất phổ biến trong backend development, thường được dùng để nhanh chóng khởi động các service phụ thuộc như MySQL, Redis, Kafka ở local, đồng thời cũng thường được dùng trong môi trường test và triển khai. Khi học Docker, cần kết hợp các khái niệm cốt lõi với thực hành command.

## Dành cho ai

- Backend developer muốn nhanh chóng hiểu nền tảng container hóa của Docker.
- Người cần dùng Docker để xây dựng môi trường development local, môi trường test hoặc các service phụ thuộc.
- Người đọc chuẩn bị phỏng vấn, cần giải thích rõ container, image, data volume, network và sự khác biệt giữa container với virtual machine.
- Engineer đã biết copy command Docker nhưng chưa rõ về image build, vòng đời container và data persistence.

## Trọng tâm học

- Container giải quyết vấn đề isolation và consistency của môi trường chạy ứng dụng.
- Image là template tĩnh, container là instance sau khi image chạy, repository dùng để phân phối và tái sử dụng image.
- Các command Docker thường dùng cần được nắm theo image management, vòng đời container, xem log, port mapping và truy cập container.
- Data volume dùng cho data persistence và mount directory của host machine, network dùng cho việc giao tiếp giữa các container và giữa container với external service.
- Docker Compose dùng để định nghĩa và chạy ứng dụng nhiều container, phù hợp với môi trường development local và service orchestration đơn giản.
- Docker không phải giải pháp triển khai vạn năng; production còn cần cân nhắc image security, resource limit, log, monitoring và khả năng orchestration.

## Thứ tự đọc đề xuất

1. [Tổng hợp core concept Docker](./docker-intro.md): trước tiên hiểu container, image, repository, Docker Engine và sự khác biệt giữa container với virtual machine.
2. [Thực chiến Docker](./docker-in-action.md): luyện tập image pull, khởi động container, port mapping, data volume, xem log và triển khai các service thường dùng qua command thực tế.
3. Kết hợp một dự án Java để luyện tập: dùng Docker khởi động các service như MySQL, Redis mà ứng dụng phụ thuộc, sau đó quan sát log, data directory và port mapping.

## Bài viết cốt lõi

- [Tổng hợp core concept Docker](./docker-intro.md): giải thích container, image, repository, Docker Engine, Docker architecture, Docker Compose và sự khác biệt giữa Docker với virtual machine.
- [Thực chiến Docker](./docker-in-action.md): tìm hiểu image management, container management, service deployment, xây dựng môi trường local và các cách troubleshooting thường gặp thông qua command và tình huống thực tế.

## Câu hỏi thường gặp

- Docker chủ yếu giải quyết vấn đề gì?
- Container và virtual machine khác nhau thế nào?
- Image và container có quan hệ gì?
- Dockerfile, image, container, repository lần lượt là gì?
- Vì sao data có thể mất sau khi xóa container? Data volume giải quyết vấn đề gì?
- Port mapping và container network lần lượt giải quyết vấn đề gì?
- Làm thế nào để xem log container, truy cập container, dừng và xóa container?
- `docker compose` phù hợp để giải quyết vấn đề gì? Nó khác gì so với thực thi riêng `docker run`?
- Docker có giá trị gì tương ứng trong môi trường development, test và deployment?

## Chuyên đề liên quan

- [Hệ thống kiến thức về công cụ phát triển](../)
- [Chuyên đề Git](../git/)
- [Chuyên đề Maven](../maven/)
- [Thiết kế hệ thống high availability](../../high-availability/)

<!-- @include: @article-footer.snippet.md -->
