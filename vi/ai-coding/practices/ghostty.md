---
title: Terminal phù hợp với Claude Code/Codex hơn iTerm2, tôi đã chuyển sang Ghostty
description: Giới thiệu cách cài đặt, vị trí file cấu hình, font, theme, Starship, phím tắt chia màn hình, Quick Terminal, Shell Integration, SSH và các vấn đề thường gặp của terminal Ghostty, phù hợp để người dùng Claude Code và Codex CLI xây dựng một terminal làm việc thuận tay.
category: Kỹ thuật AI Coding
tag:
  - Ghostty
  - Claude Code
  - Codex
  - Công cụ terminal
head:
  - - meta
    - name: keywords
      content: Ghostty,Cài đặt Ghostty,Cấu hình Ghostty,Hướng dẫn Ghostty,Terminal Claude Code,Codex CLI,Terminal AI Coding,Công cụ terminal,Starship,Shell Integration
---

Xin chào, tôi là Tiểu G. Tôi đã chuyển terminal từ iTerm2 sang Ghostty được ba tháng.

Nhìn chung trải nghiệm khá ổn, nên tôi chia sẻ trong bài viết này.

Ghostty không phải terminal chính thức được Claude Code chỉ định, nhưng quả thực đã trở nên phổ biến nhờ Claude Code. Boris Cherny, người sáng lập Claude Code, từng nhắc đến việc các lập trình viên trong development team của họ rất thích Ghostty khi nói về thói quen sử dụng của team.

![Boris Cherny nhắc đến việc team Claude Code thích Ghostty](https://oss.javaguide.cn/github/javaguide/ai/coding/boris-ghostty-x.png)

Bản thân tôi cũng chuyển sang dùng nó sau khi xem chia sẻ này rồi phát ngán với iTerm2.

Dùng Claude Code hoặc Codex CLI lâu, terminal sẽ trở thành một workbench nhỏ: vừa xem output của Agent, vừa chạy test, xem log và xử lý Git. iTerm2 dĩ nhiên cũng làm được, nhưng để dùng thuận tay thường phải tốn khá nhiều thời gian cấu hình font, theme, phím tắt và chia màn hình. Ưu điểm của Ghostty là tải xuống xong đã khá dễ dùng, về sau chỉ cần tinh chỉnh theo thói quen.

Ghostty chỉ tập trung làm tốt việc mô phỏng terminal, không có gì màu mè. Nó không tích hợp AI và cũng không phải trình quản lý server.

Dĩ nhiên, iTerm2, Warp, Kitty và các công cụ khác đều rất tốt. Tôi hy vọng bạn đọc bài viết này đừng tranh luận vì những chuyện đó; quan trọng nhất vẫn là công cụ bạn dùng thuận tay!

![Trang chủ chính thức của Ghostty](https://oss.javaguide.cn/github/javaguide/ai/coding/ghostty-homepage.png)

## Cài đặt

Trên macOS, dùng Homebrew:

```bash
brew install --cask ghostty
```

Bạn cũng có thể tải `.dmg` từ trang chính thức rồi kéo vào Applications. Gói macOS chính thức do dự án Ghostty ký và notarize; Homebrew cask cũng dùng `.dmg` chính thức.

Cài xong kiểm tra version:

```bash
/Applications/Ghostty.app/Contents/MacOS/ghostty +version
```

Nếu CLI đã có trong PATH:

```bash
ghostty +version
```

![Output kiểm tra version Ghostty](https://oss.javaguide.cn/github/javaguide/ai/coding/ghostty-version.png)

> Ghi chú version: Cấu hình trong bài viết này được đối chiếu theo Ghostty 1.3.1 (dòng 1.3.x) trên máy của tôi. Ghostty cập nhật khá nhanh, nên lấy các tùy chọn cấu hình trong `ghostty +show-config --default --docs` trên máy của bạn làm chuẩn. Ghostty 1.4.0 dự kiến cung cấp `ghostty +ssh`; phần dưới vẫn giữ cách xử lý SSH của 1.3.x, người dùng 1.4 hãy xem [tài liệu SSH của Ghostty](https://ghostty.org/docs/features/ssh) trước, đừng sao chép nguyên cấu hình cũ.

Cách cài đặt trên Linux tùy distribution. Trên Arch Linux có thể dùng trực tiếp:

```bash
sudo pacman -S ghostty
```

Với các distribution khác, ưu tiên xem trang cài đặt chính thức. Ghostty chỉ phân phối trực tiếp package macOS đã build sẵn; package Linux phần lớn do maintainer của distribution hoặc cộng đồng duy trì. Đừng tùy tiện chạy script cài đặt không rõ nguồn gốc trên máy làm việc hoặc máy công ty.

## Dùng giá trị mặc định trong một ngày trước

Thực ra bạn không cần cấu hình gì vẫn có thể dùng, các giá trị mặc định đã đáp ứng được nhu cầu của phần lớn người dùng.

Ghostty tích hợp sẵn JetBrains Mono theo mặc định và cũng hỗ trợ Nerd Fonts. Phần lớn mọi người không cần cấu hình font vẫn có thể dùng ngay.

Khi mới bắt đầu, đừng vội sao chép vài trăm dòng cấu hình. Hãy mở lên dùng một ngày trước, rồi mới chỉnh font, theme, padding trong cửa sổ, độ trong suốt, clipboard, Shell Integration và phím tắt chia màn hình. Cấu hình terminal càng dài thì càng khó tìm lỗi; một điểm đáng dùng của Ghostty là bạn có thể cấu hình ít hơn.

## File cấu hình ở đâu

Cấu hình Ghostty có dạng `key = value`. Tên file được khuyến nghị hiện nay là `config.ghostty`, còn tên file cũ `config` vẫn được đọc. Các path thường gặp:

```text
~/.config/ghostty/config.ghostty
~/.config/ghostty/config
```

Trên macOS còn đọc:

```text
~/Library/Application Support/com.mitchellh.ghostty/config.ghostty
~/Library/Application Support/com.mitchellh.ghostty/config
```

Nếu cả hai nơi đều có cấu hình, path Application Support của macOS được load sau, các mục xung đột sẽ ghi đè giá trị trước đó. Nếu cấu hình không có hiệu lực, hãy kiểm tra điều này trước.

Các lệnh kiểm tra thường dùng:

```bash
ghostty +list-fonts
ghostty +list-themes
ghostty +list-keybinds --default
ghostty +validate-config
```

Sau khi sửa cấu hình, nhấn `Cmd + Shift + ,` trên macOS để reload, `Ctrl + Shift + ,` trên Linux. Các mục của cửa sổ như độ trong suốt có thể không hot reload; nếu không thay đổi, hãy khởi động lại Ghostty.

## Cấu hình tối thiểu của tôi

Tạo directory trước:

```bash
mkdir -p ~/.config/ghostty
```

Chỉnh sửa cấu hình:

```bash
nano ~/.config/ghostty/config.ghostty
```

Có thể dùng trực tiếp cấu hình này:

```ini
# Font
font-family = "JetBrainsMono Nerd Font Mono"
font-size = 14
font-thicken = true
font-thicken-strength = 80
font-codepoint-map = U+2E80-U+9FFF,U+F900-U+FAFF,U+FF00-U+FFEF=PingFang SC

# Theme
theme = Catppuccin Mocha

# Window
window-padding-x = 12
window-padding-y = 10
window-save-state = always
background-opacity = 0.95
background-blur = 20

# Cursor and scrolling
cursor-style = bar
cursor-style-blink = true
scrollback-limit = 10000000
scrollbar = never

# Shell Integration
shell-integration = detect
shell-integration-features = cursor,sudo,title

# macOS
macos-option-as-alt = left
macos-titlebar-style = transparent
macos-titlebar-proxy-icon = hidden

# Splits
split-divider-color = #45475a
unfocused-split-opacity = 0.92

# Clipboard
copy-on-select = false
clipboard-paste-protection = true
clipboard-paste-bracketed-safe = true
```

Font ở đây là JetBrainsMono Nerd Font Mono, chủ yếu để ký hiệu branch Git, prompt Starship và icon Powerline không biến thành ô vuông. Nếu chưa cài:

```bash
brew install --cask font-jetbrains-mono-nerd-font
```

Đừng đưa `PingFang SC` trực tiếp vào làm `font-family` thứ hai một cách tùy tiện. Khi font chính không khớp, tiếng Anh có thể cũng rơi vào font Trung Quốc, khiến khoảng cách chữ rất kỳ. `font-codepoint-map` chỉ giao các codepoint tiếng Trung cho `PingFang SC`, ổn định hơn.

`copy-on-select = false` là thói quen của tôi. Ghostty mặc định sẽ copy text được chọn, người dùng Linux có thể thích điều này; trên macOS, tôi thích tự nhấn `Cmd + C` hơn để tránh clipboard bị ghi đè nhầm.

Nên giữ `clipboard-paste-protection = true`. Khi copy command nhiều dòng từ web vào terminal, vốn nên có thêm một bước nhắc nhở.

Đơn vị của `scrollback-limit` là byte, không phải số dòng; `10000000` tương đương khoảng 10 MB, và mỗi split, tab sẽ được tính riêng.

![Ghostty kết hợp với Catppuccin Mocha, JetBrainsMono Nerd Font và Starship](https://oss.javaguide.cn/github/javaguide/ai/coding/ghostty-terminal-demo.png)

## Theme

Liệt kê các theme tích hợp sẵn:

```bash
ghostty +list-themes
```

Đổi theme chỉ cần một dòng:

```ini
theme = TokyoNight
```

Tôi thường dùng:

```ini
theme = Catppuccin Mocha
```

Muốn theo chế độ sáng tối của hệ thống:

```ini
theme = dark:Catppuccin Mocha,light:Catppuccin Latte
```

Các theme tích hợp sẵn của Ghostty đã đủ nhiều. Theme tùy chỉnh về bản chất cũng là một đoạn cấu hình được Ghostty load, phần lớn chỉ thay đổi màu sắc; khi tải từ nguồn lạ, hãy mở ra xem để xác nhận nó không tiện tay sửa font, độ trong suốt hoặc keybind.

## Starship tùy chọn

Ghostty quản lý cửa sổ terminal, font, theme và protocol; Starship quản lý shell prompt.

Muốn prompt đồng nhất với phong cách Catppuccin, có thể cài:

```bash
brew install starship
```

Thêm vào cuối `~/.zshrc`:

```bash
command -v starship >/dev/null && eval "$(starship init zsh)"
```

Muốn xác nhận chính xác Starship đang hiển thị những module nào, có thể chạy trong Git repository:

```bash
starship explain
```

![Starship explain hiển thị path, branch và trạng thái Git trong prompt](https://oss.javaguide.cn/github/javaguide/ai/coding/starship-explain.png)

Tôi không khuyến nghị bật toàn bộ module Starship ngay từ đầu. Directory, branch Git, trạng thái Git và thời gian chạy là đủ dùng; Kubernetes, cloud account và container thì dùng đến đâu thêm đến đó. Prompt phải tính toán mỗi lần nhấn Enter, thông tin quá đầy sẽ khiến nó chậm hơn.

## Chia màn hình và phím tắt thường dùng

Trên macOS, trước tiên hãy nhớ các phím này:

| Phím tắt                      | Tác dụng                          |
| ----------------------------- | --------------------------------- |
| `Cmd + T`                     | Tab mới                           |
| `Cmd + W`                     | Đóng terminal hoặc split hiện tại |
| `Cmd + D`                     | Chia màn hình sang phải           |
| `Cmd + Shift + D`             | Chia màn hình xuống dưới          |
| `Cmd + [` / `Cmd + ]`         | Chuyển split trước/sau            |
| `Cmd + Option + phím mũi tên` | Chuyển split theo hướng           |
| `Cmd + Shift + Enter`         | Phóng to/khôi phục split hiện tại |
| `Cmd + F`                     | Tìm trong output lịch sử          |
| `Cmd + Shift + ,`             | Reload cấu hình                   |
| `Cmd + Shift + P`             | Command palette                   |

Khi chạy Claude Code, bố cục ba khu vực là thuận tay nhất:

1. Nhấn `Cmd + D` để chia màn hình trái phải.
2. Đặt con trỏ ở bên phải, nhấn `Cmd + Shift + D` để chia tiếp trên dưới.
3. Chạy Claude Code bên trái, chạy test ở góc trên bên phải, xem log hoặc Git ở góc dưới bên phải.
4. Nếu output của Claude quá dài, nhấn `Cmd + Shift + Enter` để phóng to tạm thời.

Bố cục này không cần tmux và cũng không cần liên tục sắp xếp nhiều cửa sổ.

![Ghostty chạy Claude Code, development service và log trong các split](https://oss.javaguide.cn/github/javaguide/ai/coding/ghostty-split-claude-code.png)

Nếu muốn tự bind phím tắt, dùng format này:

```ini
keybind = trigger=action
```

Ví dụ:

```ini
keybind = cmd+shift+e=equalize_splits
keybind = cmd+shift+f=toggle_split_zoom
```

## Quick Terminal

Quick Terminal là terminal tạm thời trượt xuống từ phía trên màn hình. Nó phù hợp để chạy command tạm thời, không phù hợp để làm workflow chính cả ngày.

Cấu hình:

```ini
quick-terminal-position = top
quick-terminal-screen = main
quick-terminal-autohide = true
quick-terminal-animation-duration = 0.15
keybind = global:ctrl+grave_accent=toggle_quick_terminal
```

Quick Terminal không có phím tắt mặc định, bắt buộc phải tự bind `toggle_quick_terminal`. `global:` không dùng được trên mọi platform: macOS cần cấp quyền Accessibility cho Ghostty; Quick Terminal trên Linux chỉ hỗ trợ Wayland và yêu cầu compositor cung cấp `wlr-layer-shell-v1`, không hỗ trợ X11. Animation trượt vào trên Linux hiện chỉ hỗ trợ KDE, đồng thời phải bật plugin “Sliding Popups” của KWin và khởi động lại hoàn toàn Ghostty; ngay cả khi đã cấu hình `quick-terminal-animation-duration`, các môi trường như GNOME cũng sẽ không có animation này. Khi cấu hình không có vấn đề nhưng phím tắt không phản hồi, hãy kiểm tra protocol hiển thị, khả năng của desktop environment, quyền hệ thống và xung đột phím tắt trước.

Ngoài ra, trên macOS sau khi thay đổi `quick-terminal-position` cần khởi động lại hoàn toàn Ghostty.

## Shell Integration

Mục này tôi sẽ giữ lại:

```ini
shell-integration = detect
```

Ghostty sẽ thêm một đoạn integration script cho zsh, fish, bash, nushell và elvish. Sau khi bật, split mới sẽ đi theo directory hiện tại; ví dụ khi mở split bên phải trong root directory của project, bên phải sẽ không quay lại directory home. Việc xuống dòng và resize của prompt phức tạp cũng ít bị lệch hơn, đồng thời output lịch sử có thể nhảy theo prompt.

Có hai điểm nhỏ cần lưu ý.

`/bin/bash` tích hợp sẵn trên macOS quá cũ, tài liệu chính thức cho biết nó không hỗ trợ tự động inject; người dùng zsh mặc định thường không cần quan tâm. Một điểm khác là khi tự chuyển shell trong Ghostty, chẳng hạn vào `nix-shell`, khả năng integration có thể mất và cần tự load script tương ứng.

## Chưa cần cấu hình SSH vội

Ghostty 1.3.x có terminfo và khả năng protocol riêng. Khi remote host không nhận diện được, các TUI như Neovim và htop có thể hiển thị bất thường.

Nếu bạn chỉ SSH thỉnh thoảng, trước mắt đừng thay đổi. Khi thực sự gặp vấn đề hiển thị trên remote, hãy cân nhắc:

```ini
shell-integration-features = cursor,title,ssh-env,ssh-terminfo
```

Môi trường SSH vốn đã phức tạp, khi không có vấn đề thì nên bớt thêm một lớp bọc.

Sau khi Ghostty 1.4.0 phát hành, ưu tiên đánh giá cách integration do `ghostty +ssh` cung cấp rồi quyết định có giữ cấu hình 1.3.x bên trên hay không.

## Vấn đề thường gặp

Nếu cấu hình không có hiệu lực, trước tiên kiểm tra hai directory rồi chạy validation:

```bash
ls -la ~/.config/ghostty
ls -la "$HOME/Library/Application Support/com.mitchellh.ghostty"
ghostty +validate-config
```

Một số cấu hình trên mạng viết các dấu phân cách như `=== Font ===`, Ghostty không nhận dạng được. Comment phải viết thành `# Font`.

Khoảng cách chữ tiếng Anh rất kỳ, trước tiên kiểm tra tên font có được nhận diện hay không:

```bash
ghostty +list-fonts | rg -i "JetBrains|Mono|Nerd"
```

Nếu bạn viết `font-family = JetBrains Mono` nhưng máy không có font này, Ghostty sẽ fallback. Khi fallback sang font Trung Quốc, tiếng Anh dễ trở nên xấu. Hãy cài font hoặc đổi thành tên family mà Ghostty thực sự nhận diện được.

Tên theme lấy theo output của `ghostty +list-themes`. Nếu thấy `Catppuccin Mocha`, hãy ghi nguyên dạng này trong cấu hình:

```ini
theme = Catppuccin Mocha
```

Độ trong suốt không thay đổi, trước tiên hãy khởi động lại hoàn toàn Ghostty. Một trường hợp khác là Neovim hoặc tmux tự vẽ màu nền; Ghostty mặc định chỉ làm nền cửa sổ trong suốt, không đảm bảo mọi cell có màu nền tường minh đều trong suốt. Nếu thực sự muốn các cell này cũng trong suốt, hãy xem thêm `background-opacity-cells`.

Nếu việc chọn text ghi đè clipboard, hãy tắt:

```ini
copy-on-select = false
```

Phím tắt global của Quick Terminal không phản hồi, hãy kiểm tra ba việc: trong cấu hình có `global:` hay không, quyền hệ thống hoặc desktop environment có hỗ trợ hay không, và phím tắt có bị phần mềm khác chiếm hay không.

## Tổng kết

Nếu chỉ muốn đổi sang một terminal đẹp hơn, iTerm2 cũng có thể điều chỉnh theme và độ trong suốt. Với tôi, Ghostty cho cảm giác nhẹ hơn khi dùng cửa sổ native, chia màn hình mặc định, cấu hình dễ đọc và output dài; đây là cảm nhận trên máy và cách sử dụng cá nhân, không phải kết luận về performance nói chung.

Nên dùng giá trị mặc định trong một ngày trước, sau đó điều chỉnh font, theme và phím tắt theo vấn đề thực tế; dùng quen split rồi mới quyết định có bật Quick Terminal hay không. Bạn cũng có thể để Coding Agent tạo cấu hình ứng viên dựa trên bài viết này, nhưng trước khi ghi vào file cần xác nhận version Ghostty, platform và cấu hình hiện có trên máy để tránh ghi đè phím tắt cá nhân.
