# claude-spring-boot-rules

> Một file `CLAUDE.md` duy nhất giúp Claude Code (và các trợ lý lập trình LLM khác) viết code **Spring Boot / Java backend chuẩn production** — thay vì code spaghetti dài dòng, hardcode lung tung, mù tịt về ngôn ngữ end-user.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Claude Code](https://img.shields.io/badge/Claude-Code-orange)](https://docs.claude.com/en/docs/claude-code)
[![Cursor](https://img.shields.io/badge/Cursor-supported-blue)](https://cursor.sh)
[![Spring Boot](https://img.shields.io/badge/Spring%20Boot-3.x-brightgreen)](https://spring.io/projects/spring-boot)

🇬🇧 **[Read in English →](./README.md)**

> **Kế thừa từ prior art.** Rules #1–#4 phỏng theo [`forrestchang/andrej-karpathy-skills`](https://github.com/forrestchang/andrej-karpathy-skills) (MIT, © Forrest Chang), bản thân nó dựa trên observations của Andrej Karpathy về LLM coding pitfalls. Rules #5–#19 là bổ sung gốc cho Spring Boot / Java backend conventions. Xem [`NOTICE`](./NOTICE) để biết attribution đầy đủ.

---

## Tại sao có repo này?

Repo gốc [`andrej-karpathy-skills`](https://github.com/forrestchang/andrej-karpathy-skills) cho bạn 4 nguyên tắc tổng quát (không phụ thuộc ngôn ngữ) giúp Claude Code viết code tốt hơn. Tốt rồi.

Nhưng khi LLM viết **Spring Boot / Java backend cụ thể**, nó vẫn có thói quen:

- ❌ Hardcode magic string và status value khắp nơi
- ❌ Nhồi business logic vào controller
- ❌ Tự tay copy `entity.getX()` → `dto.setX(x)` thay vì dùng MapStruct
- ❌ Viết tay getter/setter/constructor thay vì dùng Lombok
- ❌ Dùng UUID v4 ngẫu nhiên — hiệu năng index B-tree rất tệ
- ❌ Trả thẳng `Page<T>` nội bộ của Spring ra API
- ❌ Trả error message sai ngôn ngữ so với end-user thực tế
- ❌ Dùng cùng một chiến lược thread pool cho cả I/O chờ đợi và tác vụ nặng CPU
- ❌ Lặp lại HTTP status và error message ở nhiều service

Repo này **mở rộng** Karpathy guidelines bằng 15 rule bổ sung để fix các thói quen xấu đặc thù của Spring Boot. Tối ưu cho dự án Java/Spring Boot 3.x.

> **Ghi chú về localization:** Rule #13 ("Localized Error Messages") ship sẵn ví dụ tiếng Việt vì đó là thị trường của mình. **Rule này được thiết kế để customize hoặc xóa** nếu end-user của bạn nói ngôn ngữ khác. 18 rule còn lại không phụ thuộc ngôn ngữ.

## Trong repo có gì

| File | Mục đích |
|------|----------|
| [`CLAUDE.md`](./CLAUDE.md) | 19 nguyên tắc. Bỏ vào root project, Claude Code tự đọc. |
| [`CURSOR.md`](./CURSOR.md) | Cùng nội dung, định dạng cho Cursor IDE. |
| [`EXAMPLES.md`](./EXAMPLES.md) | Ví dụ code before/after cụ thể cho từng nguyên tắc. |
| [`.claude-plugin/plugin.json`](./.claude-plugin/plugin.json) | Cài như Claude Code plugin. |
| [`.cursor/rules/spring-boot.mdc`](./.cursor/rules/spring-boot.mdc) | Cursor rule tự attach theo pattern. |
| [`skills/spring-boot-guidelines/`](./skills/spring-boot-guidelines/) | Định dạng skill tương thích `skills.sh`. |
| [`NOTICE`](./NOTICE) | Ghi nhận credit và attribution. |

## Tóm tắt 19 nguyên tắc

**Từ upstream (`andrej-karpathy-skills`, MIT © Forrest Chang):**

1. **Suy nghĩ trước khi code** — Nói rõ giả định, không tự ý chọn cách hiểu
2. **Đơn giản trước đã** — Code tối thiểu, không suy diễn
3. **Sửa đúng chỗ cần sửa** — Không "tiện tay" refactor chỗ khác
4. **Định hướng theo mục tiêu** — Có tiêu chí thành công kiểm chứng được

**Bổ sung gốc (Spring Boot / Java backend, MIT © Tobi2904):**

5. **Không hardcode** — Dùng Constants, Enum, config. Không có magic literal
6. **Controller mỏng, Service dày** — Controller không chứa business logic
7. **SOLID & Clean Code** — Đặc biệt là SRP và DIP
8. **Mapping bằng MapStruct** — Không tay không convert DTO ↔ Entity
9. **Dùng Lombok triệt để** — Không tự viết getter/setter/constructor
10. **UUID v7 cho ID** — Sắp xếp theo thời gian, index hiệu quả
11. **Audit bảo mật & hiệu năng chủ động** — Phát hiện SQLi, N+1... là dừng báo ngay
12. **Custom `PageResponse<T>`** — Không trả thẳng `Page<T>` của Spring ra API
13. **Localized Error Messages** — Message cho end-user phải đúng ngôn ngữ user *(default ví dụ tiếng Việt — customizable hoặc xóa được)*
14. **Roleplay user & edge case** — Liệt kê 2+ tình huống fail trước khi báo "xong"
15. **Validation hai tầng** — Stateless đặt annotation `jakarta.validation` trên DTO; rule nghiệp vụ tách thành các `Validator` strategy inject vào Service, không nhồi vào Service
16. **Split Queries thay cho `JOIN FETCH`** — Không bao giờ `JOIN FETCH` một collection; fetch con riêng rồi ghép trong bộ nhớ bằng `HashMap`
17. **`Set<>` cho collection của Entity** — Dùng `Set<>` thay `List<>` cho association của entity — không trùng lặp, đúng quan hệ nhiều-nhiều, tránh `MultipleBagFetchException`
18. **Chọn Executor theo loại workload** — Virtual thread cho I/O; fixed thread pool theo ngân sách CPU nền cho tác vụ nặng CPU
19. **Exception Factory riêng cho từng module** — Tập trung HTTP status và message của module vào các factory method dùng lại được

Chi tiết đầy đủ trong [`CLAUDE.md`](./CLAUDE.md). Ví dụ thực tế trong [`EXAMPLES.md`](./EXAMPLES.md).

---

## Bắt đầu nhanh

### Cách 1 — Claude Code (khuyến nghị)

Bỏ file vào root project Spring Boot:

```bash
cd your-spring-boot-project
curl -O https://raw.githubusercontent.com/Tobi2904/claude-spring-boot-rules/main/CLAUDE.md
```

Claude Code tự đọc `CLAUDE.md` mỗi lần mở session trong thư mục đó. Xong.

### Cách 2 — Cursor IDE

```bash
mkdir -p .cursor/rules
curl -o .cursor/rules/spring-boot.mdc \
  https://raw.githubusercontent.com/Tobi2904/claude-spring-boot-rules/main/.cursor/rules/spring-boot.mdc
```

Cursor tự attach rule theo pattern file.

### Cách 3 — Global cho mọi project

Bỏ `CLAUDE.md` vào home directory tại `~/.claude/CLAUDE.md`, nó sẽ áp dụng cho mọi project.

---

## Triết lý

Mấy rule này **mang tính quan điểm**, không phải chân lý phổ quát. Đây là những bài học từ việc ship Spring Boot API cho thị trường không nói tiếng Anh:

- **Constants > config file > hardcode.** Luôn luôn.
- **Controller là ống dẫn ngu ngốc.** Nhận HTTP vào, đẩy cho service, trả HTTP ra.
- **Mapper + Lombok loại bỏ khoảng 30% boilerplate Java điển hình.** Dùng đi.
- **Error message cho end-user là một mặt UX**, không phải công cụ debug. Viết bằng ngôn ngữ user thực sự dùng.
- **"Chỉ test happy path" là cách bug production ra đời.** Đóng vai user phá hoại.

Nếu không đồng ý rule nào, fork về và xóa. Đó là lý do file để dạng đọc được — không bị nhúng sâu trong tooling. **Rule #13 đặc biệt được thiết kế để customize** — thay tiếng Việt bằng ngôn ngữ thị trường của bạn, hoặc xóa luôn nếu end-user đọc tiếng Anh được.

---

## Đóng góp

Welcome PR, đặc biệt là:

- Ví dụ mới trong `EXAMPLES.md` chỉ ra rule này từng cứu bạn khỏi bug nào
- Bản dịch sang ngôn ngữ khác
- Bổ sung / tinh chỉnh dựa trên pattern Spring Boot 3.x

Mở issue trước nếu định thay đổi lớn.

---

## Credits & Attribution

Repo này đứng trên vai người khác. Credit đầy đủ:

- **[Andrej Karpathy](https://x.com/karpathy)** — vì [observations gốc ngày 26/01/2026](https://x.com/karpathy) về LLM coding pitfalls đã truyền cảm hứng cho mọi thứ downstream. Ông ấy không endorse bất kỳ work nào.
- **[Forrest Chang (@forrestchang)](https://github.com/forrestchang)** — vì đã đúc kết observations đó thành file `CLAUDE.md` 4 nguyên tắc gốc ([`forrestchang/andrej-karpathy-skills`](https://github.com/forrestchang/andrej-karpathy-skills), MIT). Rules #1–#4 trong repo này phỏng theo work của anh ấy.
- **[multica-ai](https://github.com/multica-ai)** — vì [organization mirror](https://github.com/multica-ai/andrej-karpathy-skills) có cấu trúc repo (`.claude-plugin/`, `.cursor/rules/`, `skills/`, `CURSOR.md`, README đa ngôn ngữ) truyền cảm hứng cho layout của repo này.

Xem [`NOTICE`](./NOTICE) để biết chính xác file nào là upstream, file nào là gốc.

Duy trì bởi [@Tobi2904](https://github.com/Tobi2904).

## License

MIT — xem [`LICENSE`](./LICENSE).
