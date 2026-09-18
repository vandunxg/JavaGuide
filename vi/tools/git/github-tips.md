---
title: Tổng hợp mẹo hữu ích về GitHub
description: Tổng hợp các mẹo sử dụng GitHub hiệu quả, bao gồm trang cá nhân, badge dự án, đọc code, GitHub Actions, Explore/Trending và cách nâng cao hiệu quả cộng tác open source.
category: Công cụ phát triển
tag:
  - Git
head:
  - - meta
    - name: keywords
      content: mẹo GitHub,trang cá nhân,README,thông tin thống kê,đóng góp open source,GitHub Actions,đọc code
---

GitHub không chỉ là nền tảng lưu trữ code. Với developer, GitHub đồng thời đảm nhiệm vai trò giới thiệu dự án, đọc code, cộng tác open source, build tự động và trang cá nhân. Bài viết này tổng hợp một số mẹo sử dụng GitHub khá hữu ích.

## Tạo CV GitHub và báo cáo thường niên GitHub bằng một cú nhấp chuột

Thông qua website [https://resume.github.io/](https://resume.github.io/), bạn có thể tạo một CV GitHub trực tuyến chỉ bằng một cú nhấp chuột.

Tuy nhiên, có nên đưa link GitHub vào CV hay không còn phụ thuộc vào chất lượng nội dung của tài khoản. Nếu tài khoản có các dự án hoàn chỉnh, lịch sử duy trì liên tục, README rõ ràng và lịch sử commit tương đối chuẩn mực, link GitHub sẽ là một điểm cộng; nếu chỉ có repository trống hoặc code luyện tập tạm thời thì không cần cố đưa vào. Hiệu quả sau khi tạo như hình dưới đây.

![CV GitHub](https://oss.javaguide.cn/2020-11/image-20201108192205620.png)

Thông qua website <https://www.githubtrends.io/wrapped>, bạn có thể tạo một báo cáo thường niên GitHub cá nhân. Báo cáo này liệt kê đóng góp dự án trong năm, các ngôn ngữ lập trình được sử dụng thường xuyên nhất và thông tin đóng góp chi tiết.

![](https://oss.javaguide.cn/github/dootask/image-20211226144607457.png)

## Trang chủ GitHub cá nhân hóa

Hiện GitHub hỗ trợ tùy chỉnh một số nội dung hiển thị trên trang cá nhân. Hiệu quả hiển thị như hình dưới đây.

![Hiệu quả hiển thị trang chủ cá nhân hóa](https://oss.javaguide.cn/java-guide-blog/image-20210616221212259.png)

Để làm được điều này rất đơn giản: bạn chỉ cần tạo một repository cùng tên với tài khoản GitHub, sau đó tùy chỉnh nội dung của `README.md`.

Nội dung tùy chỉnh hiển thị trên trang cá nhân chính là nội dung của `README.md` (_bạn nào chưa biết cú pháp Markdown thì tự kiểm điểm 5 phút_).

![Tạo một repository cùng tên với tài khoản GitHub](https://oss.javaguide.cn/java-guide-blog/image-20201107110309341.png)

Phần này cũng có thể tùy chỉnh rất đa dạng! Ví dụ: thông qua dự án open source [github-readme-stats](https://hellogithub.com/periodical/statistics/click/?target=https://github.com/anuraghazra/github-readme-stats), bạn có thể hiển thị thông tin thống kê GitHub được tạo động trong README. Hiệu quả hiển thị như hình dưới đây.

![Tạo động thông tin thống kê GitHub bằng github-readme-stats](https://oss.javaguide.cn/java-guide-blog/image-20210616221312426.png)

Về trang chủ cá nhân hóa, bài viết sẽ không nói thêm. Nếu quan tâm, bạn có thể tự tìm hiểu.

## Tùy chỉnh badge dự án

Các badge dự án bạn thấy trên GitHub đều được tạo thông qua website [https://shields.io/](https://shields.io/). Các badge của dự án JavaGuide được hiển thị như hình dưới đây.

![Badge dự án](https://oss.javaguide.cn/2020-11/image-20201107143136559.png)

Ngoài việc tạo badge tĩnh, shield.io còn có thể đọc động trạng thái dự án và tạo badge tương ứng.

![Badge dự án tùy chỉnh](https://oss.javaguide.cn/2020-11/image-20201107143502356.png)

Hiệu quả của badge mô tả trạng thái dự án sau khi tạo như hình dưới đây.

![Badge mô tả trạng thái dự án](https://oss.javaguide.cn/2020-11/image-20201107143752642.png)

## Tự động thêm biểu đồ đóng góp cho dự án

Thông qua công cụ repobeats, bạn có thể thêm biểu đồ tổng quan cơ bản về đóng góp cho dự án GitHub như hình dưới đây.

![](https://oss.javaguide.cn/github/dootask/repobeats.png)

Địa chỉ: <https://repobeats.axiom.co/>.

## Emoji GitHub

![Emoji GitHub](https://oss.javaguide.cn/2020-11/image-20201107162254582.png)

Nếu muốn sử dụng emoji trên GitHub, bạn có thể tìm thử tại đây: [www.webfx.com/tools/emoji-cheat-sheet/](https://www.webfx.com/tools/emoji-cheat-sheet/).

![Emoji GitHub trực tuyến](https://oss.javaguide.cn/2020-11/image-20201107162432941.png)

## Đọc source code dự án GitHub hiệu quả

GitHub Codespaces cung cấp môi trường phát triển trực tuyến tương tự VS Code, phù hợp để tạm thời đọc, debug hoặc nhanh chóng tham gia dự án open source. Với các dự án lớn hoặc dự án cần dependency và service local, bạn vẫn nên clone về local, sử dụng IDE quen thuộc để đọc và debug.

Dưới đây là một số cách đọc source code dự án GitHub thường dùng.

### Chrome extension Octotree

Đây là cách đã quá quen thuộc và cũng là cách tôi yêu thích nhất. Sau khi sử dụng Octotree, sidebar của trang web sẽ hiển thị dự án theo cấu trúc cây, mang đến cảm giác đọc source code giống IDE.

![Chrome extension Octotree](https://oss.javaguide.cn/2020-11/image-20201107144944798.png)

### Sourcegraph

Nếu không muốn clone dự án về local, bạn cũng có thể sử dụng các công cụ tìm kiếm và đọc code như Sourcegraph. Sourcegraph hỗ trợ tìm kiếm code cross-repository, nhảy tới reference và các tính năng khác, khá hữu ích khi đọc các dự án lớn.

Sau khi tải extension này, trang chủ dự án của bạn sẽ xuất hiện một icon nhỏ như hình dưới đây. Nhấp vào icon này để đọc source code dự án trực tuyến.

![](https://oss.javaguide.cn/2020-11/image-20201107145749659.png)

Hiệu quả đọc code bằng Sourcegraph tương tự như hình dưới đây: code cũng được hiển thị theo cấu trúc cây và còn hỗ trợ nhảy giữa các class.

![](https://oss.javaguide.cn/2020-11/image-20201107150307314.png)

### Clone dự án về local

Trước tiên clone dự án về local, sau đó sử dụng IDE yêu thích để đọc. Nếu muốn hiểu sâu một dự án, đây là lựa chọn hàng đầu.

```bash
git clone https://github.com/Snailclimb/JavaGuide.git
```

## Mở rộng tính năng GitHub

**Enhanced GitHub** giúp GitHub dễ sử dụng hơn. Browser extension này có thể hiển thị dung lượng repository, dung lượng file và hỗ trợ tải nhanh từng file.

![](https://oss.javaguide.cn/2020-11/image-20201107160817672.png)

## Tự động tạo mục lục cho file Markdown

Nếu muốn tạo mục lục cho file Markdown, bạn có thể sử dụng các extension như **Markdown Preview Enhanced** trong VS Code.

Hiệu quả mục lục sau khi tạo như hình dưới đây. Bạn chỉ cần nhấp vào link trong mục lục để chuyển tới vị trí tương ứng trong bài viết, giúp cải thiện trải nghiệm đọc.

![](<https://oss.javaguide.cn/2020-11/iShot2020-11-07%2016.14.14%20(1).png>)

Tuy nhiên, hiện GitHub đã tự động tạo mục lục cho file Markdown, chỉ là bạn cần mở mục lục bằng nút trên trang.

![](https://oss.javaguide.cn/github/cosy/image-20211227093215005.png)

## Tận dụng GitHub Explore

Explore tích hợp sẵn của GitHub là một tính năng rất mạnh và hữu ích, phù hợp để khám phá dự án, chủ đề và xu hướng công nghệ.

Nói đơn giản, GitHub Explore cung cấp các dịch vụ sau:

1. Đề xuất dự án dựa trên sở thích cá nhân của bạn;
2. GitHub Topics phân loại và tổng hợp một số dự án theo category/topic. Ví dụ, [Data visualization](https://github.com/topics/data-visualization) tổng hợp một số dự án open source liên quan đến trực quan hóa dữ liệu, còn [Awesome Lists](https://github.com/topics/awesome) tổng hợp các repository thuộc series Awesome;
3. Thông qua GitHub Trending, bạn có thể xem một số dự án open source đang phổ biến gần đây, đồng thời lọc dự án theo ngôn ngữ và khoảng thời gian;
4. GitHub Collections tương tự một bộ sưu tập bookmark. Ví dụ, bộ sưu tập [Teaching materials for computational social science](https://github.com/collections/teaching-computational-social-science) tổng hợp các tài nguyên open source liên quan đến khóa học computer science, còn bộ sưu tập [Learn to Code](https://github.com/collections/learn-to-code) tổng hợp một số repository hữu ích cho việc học lập trình;
5. ……

![](https://oss.javaguide.cn/github/javaguide/github-explore.png)

## GitHub Actions rất mạnh

Bạn có thể hiểu đơn giản GitHub Actions là nền tảng automation được tích hợp sẵn trong GitHub. Thông qua GitHub Actions, bạn có thể trực tiếp hoàn tất các công việc như build, test, deploy, quét dependency và task định kỳ trên GitHub.

Để tìm hiểu chi tiết về GitHub Actions, bạn nên xem [Tutorial nhập môn GitHub Actions](https://www.ruanyifeng.com/blog/2019/09/getting-started-with-github-actions.html) do thầy Ruan Yifeng viết.

GitHub Actions có một marketplace chính thức, trong đó có rất nhiều Actions do người khác đóng góp và có thể tái sử dụng trực tiếp.

![](https://oss.javaguide.cn/github/javaguide/image-20211227100147433.png)

## Lời kết

Bạn không cần ghi nhớ toàn bộ mẹo GitHub ngay một lần. Trước tiên sử dụng trang cá nhân, badge dự án, đọc code, Explore/Trending và GitHub Actions là đã có thể bao phủ phần lớn tình huống sử dụng hằng ngày.

Ngoài ra, bài viết này không đi sâu vào cú pháp tìm kiếm GitHub. Trong thực tế, tìm kiếm bằng keyword, lọc theo ngôn ngữ, sắp xếp theo số Star và lọc theo thời gian cập nhật thường hữu ích hơn việc học thuộc các cú pháp phức tạp.

<!-- @include: @article-footer.snippet.md -->
