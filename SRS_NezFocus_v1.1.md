# SOFTWARE REQUIREMENTS SPECIFICATION (SRS)
# DỰ ÁN: NEZFOCUS — ỨNG DỤNG MOBILE HỌC NHÓM & CHAT/VOICE REAL-TIME CHO SINH VIÊN
### (Kiến trúc lõi theo mô hình Discord, định vị sản phẩm: Study/Focus Server — iOS & Android, React Native)

**Phiên bản:** 1.1
**Trạng thái:** Approved – Baseline v1.1
**Ngày baseline:** 29/09/2026 (bản 1.0 phát hành 15/09/2026)
**Chuẩn tham chiếu:** ISO/IEC/IEEE 29148:2018 (Requirements Engineering)
**Đối tượng sử dụng:** Product Owner, Backend Team, Mobile Team, DevOps Team, QA Team, Giảng viên hướng dẫn

---

## MỤC LỤC

0. Kiểm soát tài liệu — Lịch sử phiên bản, Decision Log, Quy ước, Quản lý thay đổi
1. Giới thiệu — Mục đích, Phạm vi (gồm Out of scope), Thuật ngữ, Tài liệu tham khảo
2. Tổng quan hệ thống — Personas, Định hướng Study/Focus, Giả định & phụ thuộc, Ràng buộc, Release Plan, KPI, Rủi ro, Pháp lý
3. UX/UI & Hành vi chung (Mobile) — Điều hướng, Sitemap, Theme, Trạng thái màn hình, User flows, Platform Matrix
4. Yêu cầu chức năng — 4.1 Auth · 4.2 Guild/Channel · 4.3 RBAC · 4.4 Messaging · 4.5 Voice/Video · 4.6 Search/Notification · 4.7 Friends/DM · 4.8 Group DM/Call · 4.9 Study/Focus · 4.10 Safety
5. Đặc tả giao diện ngoài — REST, Endpoint catalog, Gateway protocol, LiveKit/TURN, Push, Hệ thống khác
6. Yêu cầu phi chức năng — Performance, Scale, Security, Availability, Observability, Mobile, A11y, i18n, Privacy, Backup, Usability
7. Kiến trúc & Tech Stack — Kiến trúc đồ án/production, Stack đã chốt, Kiến trúc Mobile (React Native), Sequence diagrams, ADR
8. Mô hình dữ liệu — PostgreSQL DDL, ScyllaDB (production), State machines, Permissions, Retention, CSDL cục bộ
9. Kiểm thử & Truy vết — Test levels, bộ test, thiết bị, load test, Traceability Matrix, DoD
10. Backlog — Premium/Boost, Gap analysis
11. Phụ lục — Mã lỗi, Gateway close codes, Glossary, Liên kết, Tổng hợp thay đổi so với v1.0

---

## 0. KIỂM SOÁT TÀI LIỆU (DOCUMENT CONTROL)

### 0.1. Lịch sử phiên bản (Revision History)

| Phiên bản | Ngày | Tác giả | Trạng thái | Tóm tắt thay đổi |
|---|---|---|---|---|
| 1.0 | 15/09/2026 | Team NezFocus | Released (nội bộ) | Bản đầu: Auth, Guild/Channel, RBAC, Messaging, Voice/Video, Search/Notification, Friends/DM, Group DM/Call, Study/Focus Mode, Backlog Premium, Gap Analysis. Phạm vi gồm Web, Desktop, Mobile. |
| 1.1 | 29/09/2026 | Team NezFocus | **Approved – Baseline v1.1** | (1) **Thu hẹp phạm vi: chỉ Mobile App (iOS + Android) viết bằng React Native/TypeScript**; Web & Desktop chuyển ra ngoài phạm vi. (2) Thêm Mục 0 Document Control và Decision Log (Mục 0.2). (3) Thêm Release Plan, KPI, Rủi ro, Ràng buộc pháp lý (2.6–2.9). (4) Viết lại Mục 3 UX theo điều hướng mobile. (5) Mỗi REQ có thêm cột **Release** và **Depends-on**, AC tách nhiều case (gồm case âm). (6) Sửa toàn bộ mâu thuẫn nội bộ của v1.0 (Phụ lục 11.5). (7) Bổ sung REQ mới: quên mật khẩu, đổi email, xóa tài khoản, recovery code 2FA, Google/Apple Sign-In, giới hạn tài khoản chưa xác thực, deep link invite, ban chặn rejoin, idempotency/nonce, pinned messages (đưa vào MVP), system messages, TURN fallback, lọc search theo quyền, push đa thiết bị, DM pair unique, cuộc gọi đánh thức app (CallKit/ConnectionService), solo focus timer, quyền điều khiển focus timer, quy tắc tính giờ học, Scheduled Study Events, Member Activity Overview, Report/Safety. (8) Thêm Mục 5 External Interfaces (REST, Gateway protocol, SFU/TURN, Push). (9) NFR tách cột "Mục tiêu đồ án" vs "Production"; bổ sung đầy đủ NFR Mobile, A11y, i18n, Privacy, Backup. (10) Chốt tech stack (một lựa chọn cho mỗi hạng mục) và ghi ADR (Mục 7.6). (11) Schema: PostgreSQL cho toàn bộ dữ liệu ở MVP (kể cả messages), ScyllaDB là mục tiêu production; bổ sung bảng thiếu, sửa lỗi reactions/nonce/timezone/audit log cascade; thêm state machines & thuật toán permission. (12) Thêm Testing Strategy & Traceability Matrix. (13) Bảng mã lỗi đầy đủ + Gateway close codes. |

### 0.2. Nhật ký quyết định (Decision Log)

> Tất cả quyết định dưới đây đã được team **chốt ngày 29/09/2026** và là baseline của v1.1. Thay đổi bất kỳ quyết định nào phải qua quy trình Change Request (Mục 0.4). Lý do kỹ thuật chi tiết của các lựa chọn công nghệ xem ADR (Mục 7.6).

| ID | Chủ đề | Quyết định | Lý do tóm tắt | Trạng thái | Ngày | Mục liên quan |
|---|---|---|---|---|---|---|
| DEC-01 | Phạm vi nền tảng | **Chỉ Mobile App: iOS + Android.** Web client (Next.js/PWA) và Desktop client ra ngoài phạm vi. | Mobile là trọng tâm đồ án; tập trung nguồn lực vào một client chất lượng. | Đã chốt | 29/09/2026 | 1.2, 3, 3.8 |
| DEC-02 | Framework mobile | **React Native + TypeScript**, dùng **Expo (development build + prebuild/CNG)**, không dùng Expo Go. | Dùng chung TypeScript với backend NestJS; Expo prebuild vẫn cho phép native module (LiveKit, CallKeep, PushKit) qua config plugin, cộng EAS Build cho iOS. | Đã chốt | 29/09/2026 | 7.2, 7.3, ADR-01/02 |
| DEC-03 | Định danh user | **Bỏ discriminator**; `username` unique toàn hệ thống (chữ thường, 2–32 ký tự `a-z0-9_.`). | v1.0 mâu thuẫn giữa kiểm tra trùng username và `username#discriminator`. | Đã chốt | 29/09/2026 | REQ-AUTH-01, REQ-FRIEND-01, 8.2 |
| DEC-04 | Ưu tiên sản phẩm | **Study/Focus lên MVP** (REQ-STUDY-01/02/03/05/06/08/10/11/15); hạ Screen Share, Multi-region, Audit Log, 2FA xuống v1.x/Backlog. | Study/Focus là điểm khác biệt; tránh thành "Discord clone kém hơn Discord". | Đã chốt | 29/09/2026 | 2.6, cột Release Mục 4 |
| DEC-05 | Study Leaderboard | **Mặc định TẮT**; Owner chủ động bật; user vẫn opt-out cá nhân. | Privacy-by-default, tôn trọng người không muốn bị so sánh. | Đã chốt | 29/09/2026 | REQ-STUDY-04 |
| DEC-06 | Tính giờ Focus khi app ở background | (a) Đang trong voice channel: giữ audio session (iOS) / foreground service (Android) ⇒ Gateway sống ⇒ heartbeat 60s; (b) Solo Focus hoặc mất heartbeat: server tính theo sự kiện start/stop do server ghi nhận, giới hạn bởi thời gian thực trôi qua trên server, chỉ cộng pha `focus`. | Tránh mất giờ học khi user khóa màn hình để học; server luôn là nguồn sự thật. | Đã chốt | 29/09/2026 | REQ-STUDY-15, 7.3.3, 7.3.6 |
| DEC-07 | Cuộc gọi đánh thức app khi bị kill | **Tách REQ-DMVC-04 (CallKit/ConnectionService + PushKit VoIP), Release v1.x.** MVP dùng push ưu tiên cao + màn hình cuộc gọi trong app. | Cần Apple Developer Program và tuân thủ quy tắc PushKit; rủi ro timeline MVP. | Đã chốt | 29/09/2026 | REQ-DMVC-01/04 |
| DEC-08 | Đăng nhập bên thứ ba | **v1.x: Google Sign-In và Sign in with Apple triển khai cùng lúc.** | Sinh viên có tài khoản Google của trường; chủ động đáp ứng yêu cầu App Store về tùy chọn đăng nhập tương đương khi có đăng nhập bên thứ ba. | Đã chốt | 29/09/2026 | REQ-AUTH-12 |
| DEC-09 | Nhạc lo-fi dùng chung | **Tách khỏi REQ-STUDY-02 thành REQ-STUDY-14, Backlog**; chỉ dùng nhạc royalty-free/CC0 do team tự host. | Rủi ro bản quyền và chi phí kỹ thuật. | Đã chốt | 29/09/2026 | REQ-STUDY-02/14 |
| DEC-10 | Định nghĩa "guild học tập" (REQ-STUDY-09) | Guild là "học tập" nếu `template='study_server'` **hoặc** user tự đánh dấu guild là "Học tập" trong cài đặt cá nhân. | v1.0 chưa định nghĩa. | Đã chốt | 29/09/2026 | REQ-STUDY-09, 8.2 |
| DEC-11 | Screen share | **Android: v1.x** (MediaProjection + foreground service). **iOS: Backlog** (cần Broadcast Upload Extension). Chỉ xem được screen share của người khác trên cả 2 nền tảng từ v1.x. | Chi phí triển khai iOS cao, không phải tính năng cốt lõi của học nhóm. | Đã chốt | 29/09/2026 | REQ-VC-03, 3.8 |
| DEC-12 | Backend | **Modular monolith NestJS (TypeScript)**, ORM **Drizzle**, WebSocket Gateway bằng **`ws`** (`@nestjs/platform-ws`). | Dùng chung ngôn ngữ với app; monolith phù hợp quy mô đồ án, module hóa để tách service sau. | Đã chốt | 29/09/2026 | 7, ADR-08/09 |
| DEC-13 | SFU | **LiveKit** (1 node tự host hoặc LiveKit Cloud khi demo), dùng TURN tích hợp. | Có SDK React Native chính thức, simulcast/adaptive stream, TURN sẵn. | Đã chốt | 29/09/2026 | REQ-VC-*, 5.5, ADR-10 |
| DEC-14 | Message broker | **MVP không dùng message broker**: event nội bộ qua `@nestjs/event-emitter`; job trì hoãn/định kỳ qua **BullMQ** trên Redis sẵn có. Production: **NATS JetStream**. | Giảm hạ tầng cho đồ án; Redis đã có sẵn cho Pub/Sub/rate limit. | Đã chốt | 29/09/2026 | 7.1, ADR-11 |
| DEC-15 | Tìm kiếm | **Meilisearch**. | Nhẹ, dễ vận hành, bỏ dấu tiếng Việt khi tìm, hỗ trợ filter theo `channel_id` để lọc quyền. | Đã chốt | 29/09/2026 | REQ-SEARCH-01/03, ADR-12 |
| DEC-16 | Băm mật khẩu | **Argon2id**. | Khuyến nghị hiện đại, kháng GPU tốt hơn bcrypt. | Đã chốt | 29/09/2026 | NFR-SEC-02 |
| DEC-17 | Lưu tin nhắn | **PostgreSQL cho MVP-Đồ án**; **ScyllaDB là mục tiêu production** (khi vượt ngưỡng tải, xem 8.3). | Một CSDL duy nhất cho đồ án, vẫn giữ repository interface để chuyển đổi. | Đã chốt | 29/09/2026 | 8.2, 8.3, ADR-13 |
| DEC-18 | Độ tuổi người dùng | Điều khoản sử dụng: **người dùng từ 18 tuổi trở lên** (đối tượng sinh viên). | Tránh phát sinh nghĩa vụ xử lý dữ liệu của trẻ em ở giai đoạn đồ án/pilot. | Đã chốt | 29/09/2026 | 2.9, REQ-AUTH-01 |
| **TBD-01** | Tham vấn pháp lý **Nghị định 147/2024/NĐ-CP** | **Cần kiểm tra / tham vấn người có chuyên môn pháp lý** xem NezFocus khi phát hành công khai tại Việt Nam có thuộc diện áp dụng hay không và nghĩa vụ cụ thể (vd: xác thực tài khoản). SRS **không** khẳng định chi tiết nghĩa vụ. | Chưa có ý kiến pháp lý. | **TBD** — phải đóng trước khi phát hành công khai; không chặn MVP-Đồ án nội bộ. | — | 2.9 |

### 0.3. Quy ước đọc tài liệu (Document Conventions)

- **REQ-ID:** `REQ-<NHÓM>-<SỐ>`. ID đã phát hành ở v1.0 **không được tái sử dụng** cho nghĩa khác; REQ bị loại/di chuyển được ghi chú "Đã chuyển sang …".
- **Priority (MoSCoW):** Must / Should / Could. Mục Backlog dùng thêm thang P1/P2/P3 (Mục 10).
- **Release:** `MVP-Đồ án` (bắt buộc có khi nộp/bảo vệ đồ án) · `v1.x` (ngay sau MVP) · `Backlog` (dài hạn, Mục 10).
- **Depends-on:** các REQ phải hoàn thành trước. Quy tắc: một REQ không được có Release sớm hơn REQ nó phụ thuộc, và Priority không được cao hơn REQ nó phụ thuộc.
- **Acceptance Criteria:** dạng Given/When/Then, đánh số `AC1`, `AC2`…; REQ quan trọng có ít nhất 1 case âm (đánh dấu "(âm)").
- **Mã lỗi:** dạng `4xxxx`/`5xxxx`, tra cứu tại Phụ lục 11.1. Gateway close code `4xxx` tại Phụ lục 11.2.
- **Ký hiệu:** 🆕 REQ mới ở v1.1; ✏️ REQ được sửa so với v1.0.

### 0.4. Quản lý thay đổi (Change Control)

- Baseline v1.1 chỉ được thay đổi qua **Change Request (CR)**: người đề xuất ghi CR (mô tả, lý do, REQ bị ảnh hưởng, ước lượng công) → Product Owner + Tech Lead duyệt → cập nhật SRS lên v1.2 kèm dòng Revision History.
- Thay đổi chỉ sửa lỗi chính tả/định dạng không cần CR, nhưng phải ghi vào Revision History.

---

## 1. GIỚI THIỆU (INTRODUCTION)

### 1.1. Mục đích (Purpose)

Tài liệu này đặc tả các yêu cầu chức năng và phi chức năng cho **NezFocus** — ứng dụng mobile học nhóm và giao tiếp cộng đồng theo mô hình server/guild, hỗ trợ nhắn tin văn bản real-time, gọi thoại/video qua WebRTC, hệ thống phân quyền (RBAC) tương tự Discord, cùng bộ tính năng **Study/Focus** (Pomodoro đồng bộ, phòng học im lặng, streak, task board, deadline tổng hợp) dành cho sinh viên. Tài liệu là nguồn tham chiếu thống nhất (single source of truth) cho Product, Backend, Mobile, DevOps và QA trong suốt vòng đời phát triển.

> **Định hướng sản phẩm (DEC-04):** NezFocus phát triển song song hai lớp: **lõi giao tiếp** (Auth, Guild/Channel, RBAC, Messaging, Voice, Notification, Friends/DM) ở mức tối thiểu đủ dùng, và **lớp Study/Focus** (Mục 4.9) là điểm khác biệt chính. Các tính năng lõi tốn công nhưng ít giá trị với sinh viên (multi-region voice, screen share, audit log đầy đủ…) được đẩy sang v1.x/Backlog. Chi tiết tại Mục 2.3 và 2.6.

### 1.2. Phạm vi (Scope)

**Trong phạm vi (DEC-01):**

- **Mobile App iOS + Android** — React Native + TypeScript, Expo development build/prebuild (DEC-02). Đây là **client duy nhất** của sản phẩm.
- **Backend** — modular monolith NestJS (DEC-12) gồm các module Auth, Guild/Channel, RBAC, Messaging, Gateway (WebSocket), Voice (cấp token LiveKit), Study, Notification, Search, Scheduler; PostgreSQL + Redis + Meilisearch + 1 LiveKit node + object storage S3-compatible.
- **Hạ tầng hỗ trợ deep link** — domain `nezfocus.app` phục vụ file `apple-app-site-association`/`assetlinks.json`, **trang chuyển hướng tĩnh** cho link invite/xác thực email/reset mật khẩu khi thiết bị chưa cài app (chỉ hiển thị thông tin tối thiểu + nút tải app; **không phải web client**).
- **Kiến trúc production mục tiêu** (microservices, ScyllaDB, NATS, multi-region) được mô tả ở Mục 7 để định hướng mở rộng, **không** là yêu cầu nghiệm thu của MVP-Đồ án.

**Ngoài phạm vi (Out of scope):**

| Hạng mục | Ghi chú |
|---|---|
| Web client (Next.js/React, PWA) | Đã có ở v1.0, loại khỏi phạm vi theo DEC-01. Mọi yêu cầu chỉ dành cho web (Web Push, `prefers-color-scheme` của trình duyệt, phím tắt Ctrl+K/Enter/Shift+Enter, layout 3–4 cột) đã bị bỏ. |
| Desktop client (Tauri/Electron) | Loại khỏi phạm vi theo DEC-01. |
| Tablet-optimized layout | App chạy được trên tablet ở chế độ phóng to layout điện thoại; không có layout riêng. |
| Gói thuê bao trả phí (NezFocus+), Server Boost | Đặc tả sẵn ở Mục 10 (Backlog). |
| Livestream ra nền tảng thứ ba, Bot framework/Webhook, Threads/Forum/Stage channels, Custom emoji/sticker, Server Discovery | Mục 10 (Backlog). |
| Mã hóa đầu-cuối (E2EE), AR/VR | Không có kế hoạch. |
| Admin console dạng web | Chức năng quản trị nền tảng tối thiểu nằm trong app, chỉ hiện với tài khoản admin (REQ-SAFE-02). |

### 1.3. Định nghĩa, từ viết tắt và thuật ngữ

| Thuật ngữ | Giải thích |
|---|---|
| **SFU** (Selective Forwarding Unit) | Media server chuyển tiếp có chọn lọc các luồng RTP từ người gửi tới nhiều người nhận mà không transcode, giảm tải CPU so với MCU. |
| **MCU** (Multipoint Control Unit) | Media server giải mã, trộn và mã hóa lại luồng audio/video thành một luồng duy nhất; tốn CPU hơn SFU. |
| **TURN/STUN** | Giao thức hỗ trợ WebRTC vượt NAT/firewall. STUN giúp client biết địa chỉ public; TURN relay media khi kết nối trực tiếp thất bại (4G CGNAT, wifi trường chặn UDP). |
| **Gateway** | Module quản lý kết nối WebSocket persistent: heartbeat, session resume, phân phối sự kiện (event dispatch). |
| **RBAC** | Role-Based Access Control — mỗi role mang một permission bitmap, gán cho member theo guild. |
| **Guild** | Thuật ngữ kỹ thuật cho "Server". UI hiển thị "Server" hoặc "Study Server". |
| **Permission Overwrite** | Allow/deny bitmap ở cấp channel cho một role hoặc member cụ thể (thuật toán tại Mục 8.5). |
| **Fanout** | Phân phối một sự kiện tới toàn bộ client đang subscribe kênh/guild liên quan. |
| **VAD** | Voice Activity Detection — phát hiện giọng nói để hiển thị "đang nói". |
| **Presence** | Trạng thái hiện diện: Online, Idle, DND, Invisible/Offline. |
| **Snowflake ID** | ID 64-bit duy nhất toàn cục (timestamp + worker + sequence). **Trong JSON luôn serialize dạng string** vì kiểu `number` của JavaScript không biểu diễn chính xác số nguyên 64-bit. |
| **Cursor-based Pagination** | Phân trang bằng con trỏ (`before`/`after` = message ID) thay vì OFFSET. |
| **Hot Partition** | Một partition CSDL phân tán nhận tải vượt trội so với các partition khác. |
| **JWT** | JSON Web Token — token xác thực stateless. |
| **WSS** | WebSocket Secure (WebSocket qua TLS). |
| **Nonce / Idempotency Key** | Chuỗi do client sinh cho mỗi thao tác tạo mới, giúp server nhận diện request gửi lặp (retry) và không tạo bản ghi trùng. |
| **CallKit / ConnectionService** | Framework hệ điều hành (iOS / Android) hiển thị cuộc gọi VoIP như cuộc gọi gốc của máy (màn hình khóa, full-screen). |
| **PushKit (VoIP push)** | Loại push iOS dành riêng cho cuộc gọi, đánh thức app kể cả khi bị kill. |
| **Expo prebuild / CNG** | Continuous Native Generation: sinh project native iOS/Android từ cấu hình `app.config.ts` + config plugins, cho phép dùng native module ngoài Expo SDK. |
| **Development build** | Bản build app chứa native module của dự án, dùng thay Expo Go trong quá trình phát triển. |
| **Focus Session** | Phiên Pomodoro đồng bộ gắn với 1 voice channel (REQ-STUDY-01). |
| **Solo Focus** | Phiên Pomodoro cá nhân không gắn voice channel (REQ-STUDY-10). |
| **Pha (Phase)** | Một đoạn trong chu kỳ Focus: `focus`, `short_break`, `long_break`. |
| **Focused minutes** | Số phút thuộc pha `focus` hệ thống ghi nhận cho user theo REQ-STUDY-15. |
| **Study Streak** | Số ngày liên tiếp (theo timezone của user) có tổng focused minutes ≥ 15 (REQ-STUDY-03). |
| **Study Server** | Guild tạo từ template `study_server` (REQ-STUDY-05). |
| **Local day** | Ngày lịch tính theo `user_settings.timezone` (IANA, vd `Asia/Ho_Chi_Minh`). |

### 1.4. Tài liệu tham khảo (References)

| # | Nguồn | Nội dung áp dụng | Độ tin cậy |
|---|---|---|---|
| R1 | ISO/IEC/IEEE 29148:2018 — Requirements engineering | Cấu trúc SRS, thuộc tính requirement tốt (verifiable, traceable, unambiguous). | Chuẩn quốc tế |
| R2 | Discord Engineering Blog — "How Discord Stores Trillions of Messages" | ScyllaDB, bucket theo thời gian, request coalescing (mục tiêu production). | Nguồn chính thức |
| R3 | Discord Engineering Blog — các bài về data platform/analytics | ETL tách khỏi hệ transactional. | Nguồn chính thức |
| R4 | Discord Developer Documentation (Permissions, Gateway) | Thuật toán permission overwrite, Gateway opcode/heartbeat/resume. | Nguồn chính thức |
| R5 | LiveKit Documentation (server, React Native SDK) | Token truy cập room, TURN tích hợp, simulcast/adaptive stream. | Nguồn chính thức |
| R6 | Tài liệu React Native, Expo (prebuild, config plugins, EAS Build), React Navigation, Notifee, React Native Firebase, react-native-callkeep | Kiến trúc client (Mục 7.3). | Nguồn chính thức |
| R7 | Nghị định 13/2023/NĐ-CP về bảo vệ dữ liệu cá nhân | NFR-PRIV, Mục 2.9. | Văn bản pháp luật |
| R8 | App Store Review Guidelines, Apple PushKit/CallKit docs, Google Play Policy (bản hiện hành) | Xóa tài khoản trong app, VoIP push, đăng nhập bên thứ ba, foreground service. | Chính sách nền tảng — kiểm tra bản mới nhất trước mỗi lần submit |
| R9 | Bài viết cộng đồng: Medium "The Genius Architecture Behind Discord's Voice Chat", CompileReady, Coditioning System Design Guides | Ý tưởng tổng quan (fanout, rate limiting, voice edge). | Tham khảo thứ cấp — không dùng làm căn cứ duy nhất |

> **Ghi chú áp dụng:** R2 → 8.3 (message store production). R4 → 5.4 (Gateway), 8.5 (permission). R5 → REQ-VC-*, 5.5. R6 → 7.3. R7/R8 → 2.9, 6.9, REQ-AUTH-10, REQ-DMVC-04.

---

## 2. TỔNG QUAN HỆ THỐNG (OVERALL DESCRIPTION)

### 2.1. Bối cảnh sản phẩm (Product Perspective)

NezFocus là hệ thống độc lập gồm **Mobile App (iOS/Android)** và **Backend**. App giao tiếp với backend qua **REST API** (CRUD), **WebSocket Gateway** (sự kiện real-time) và **WebRTC** tới LiveKit SFU (media). Khi app ở nền/bị kill, sự kiện quan trọng tới qua **FCM/APNs** (và PushKit cho cuộc gọi từ v1.x). Ở mức đồ án, backend là modular monolith; kiến trúc mục tiêu production là microservices (Mục 7).

### 2.2. Đối tượng người dùng (User Classes & Personas)

#### 2.2.1. User classes kỹ thuật

| Persona | Mô tả | Mục tiêu chính | Quyền hạn tiêu biểu |
|---|---|---|---|
| **Guest** | Người chưa đăng nhập, mở link invite. | Xem preview guild, đăng ký/đăng nhập. | Chỉ xem preview invite (REQ-SRV-07). |
| **Unverified User** | Đã đăng ký, chưa xác thực email. | Hoàn tất xác thực. | Bị giới hạn theo REQ-AUTH-13. |
| **Member** | Người dùng tham gia guild qua invite. | Nhắn tin, vào voice, học cùng. | `SEND_MESSAGES`, `CONNECT`, `READ_MESSAGE_HISTORY`… (tùy overwrite). |
| **Moderator** | Được gán role có quyền kiểm duyệt. | Duy trì trật tự, xử lý vi phạm, quản lý phòng học. | `KICK_MEMBERS`, `BAN_MEMBERS`, `MANAGE_MESSAGES`, `MUTE_MEMBERS`, `MODERATE_MEMBERS`, `MANAGE_STUDY_ROOMS`. |
| **Owner** | Người tạo guild, có toàn quyền. | Quản trị toàn diện guild. | Bỏ qua mọi kiểm tra permission; chuyển quyền Owner, xóa guild. |
| **Platform Admin / Trust & Safety** | Tài khoản vận hành của team. | Xử lý report, khóa tài khoản lạm dụng. | Màn hình admin trong app (REQ-SAFE-02); chỉ xem nội dung trong phạm vi report. |

#### 2.2.2. Persona sản phẩm (lớp định vị sinh viên)

| Persona | Bối cảnh sử dụng | Nhu cầu chính | Ánh xạ vào role kỹ thuật | REQ đáp ứng |
|---|---|---|---|---|
| **Sinh viên tự học (Self-learner)** | Tham gia Study Server ôn thi, vào phòng học im lặng mỗi tối; đôi khi học một mình. | Không bị xao nhãng, đo được thời gian học, thấy bạn bè cũng đang học. | Member | REQ-STUDY-01/02/03/04/10/15 |
| **Trưởng nhóm học/CLB (Group Lead)** | Quản lý guild nhóm đồ án/CLB, tổ chức phòng học theo lịch, giao việc. | Nhắc deadline, thấy ai đang "mất tích", tạo phòng học có lịch cố định. | Moderator/Owner + `MANAGE_STUDY_ROOMS`, `MANAGE_TASKS`, `MANAGE_EVENTS` | REQ-STUDY-05/06/11/12/13 |
| **Sinh viên bận rộn (Casual/on-the-go)** | Mở app lúc rảnh trên điện thoại. | Nắm nhanh deadline, join nhanh phòng học có bạn bè, chỉ nhận thông báo quan trọng. | Member | REQ-STUDY-08/09, REQ-NOTIF-02/04, REQ-SRV-07 |

### 2.3. Định hướng sản phẩm: "Study/Focus Server" cho Sinh viên

#### 2.3.1. Vì sao chọn hướng này

Discord là nền tảng đa mục đích, không tối ưu cho việc học. NezFocus giữ khung kỹ thuật Guild/Channel/RBAC/Voice/Messaging (bài toán kiến trúc cốt lõi của đồ án) nhưng **định vị use-case chính** là nơi sinh viên tự tổ chức: lớp học phần, nhóm đồ án, CLB học thuật, nhóm ôn thi — và dùng chủ yếu trên điện thoại.

#### 2.3.2. Bảng khác biệt hóa so với Discord

| Khía cạnh | Discord | NezFocus |
|---|---|---|
| Đơn vị tổ chức | "Server" chung chung | "Study Server" — template có sẵn cấu trúc kênh học tập (REQ-STUDY-05) |
| Voice channel | Chỉ để nói chuyện | Bật **Focus Session** (Pomodoro đồng bộ) và chế độ **Silent Co-working** (REQ-STUDY-01/02) |
| Tiến độ học | Không có | Study Streak, thống kê giờ học cá nhân/nhóm (REQ-STUDY-03/04) |
| Deadline/Task | Không có | Task Board theo channel + màn hình "Việc cần làm" tổng hợp (REQ-STUDY-06/08) |
| Lịch học nhóm | Scheduled Events chung | Study Event lặp lại theo tuần, tự bật Focus Session (REQ-STUDY-12) |
| Thông báo | Ngang hàng | Ưu tiên deadline & lời mời vào phòng học, "Chế độ Học tập" theo khung giờ (REQ-STUDY-09) |
| Học một mình | Không có | Solo Focus Timer vẫn tính streak (REQ-STUDY-10) |

### 2.4. Giả định và phụ thuộc (Assumptions & Dependencies)

**Giả định:**
- Internet tối thiểu 1 Mbps cho voice, 3 Mbps cho video 720p.
- Thiết bị: **iOS ≥ 15, Android ≥ 8.0 (API 26)**, RAM ≥ 3GB (NFR-MOB-05).
- Ở mức đồ án: tối đa ~200 user đồng thời khi demo/pilot.
- Team có ít nhất 1 máy macOS hoặc dùng EAS Build (cloud) để build iOS; có Apple Developer Program trước khi làm REQ-DMVC-04 và phát hành TestFlight.

**Phụ thuộc:**

| Phụ thuộc | Dùng cho | Phương án đồ án | Phương án production |
|---|---|---|---|
| LiveKit (SFU) | Voice/video | 1 LiveKit node tự host hoặc LiveKit Cloud (free tier) | LiveKit cluster multi-region |
| TURN/STUN | Vượt NAT (REQ-VC-08) | TURN tích hợp của LiveKit (TURN/TLS cổng 443) | TURN cluster theo region |
| Object storage S3-compatible | File đính kèm, avatar | MinIO hoặc S3 bucket | S3 + CDN |
| Redis | Pub/Sub, presence, rate limit, resume buffer, BullMQ | 1 instance | Redis Cluster |
| Meilisearch | Tìm kiếm tin nhắn (v1.x) | 1 instance | Cluster/managed |
| Email provider | Xác thực email, reset mật khẩu | Dịch vụ gửi email có free tier | Dịch vụ có SLA, SPF/DKIM/DMARC |
| Firebase Cloud Messaging + APNs | Push notification | FCM (Android; iOS qua APNs key) | Như đồ án |
| Apple PushKit | Cuộc gọi đến khi app bị kill (REQ-DMVC-04) | v1.x | v1.x |
| Google Identity, Sign in with Apple | REQ-AUTH-12 | v1.x | v1.x |
| Expo EAS (Build/Submit) | Build iOS/Android, phân phối TestFlight/Internal testing | Gói free | Gói trả phí nếu cần |
| ClamAV | Quét virus file upload (NFR-SEC-04) | 1 container | Cluster hoặc dịch vụ quét |

### 2.5. Ràng buộc thiết kế (Design Constraints)

> Chỉ số định lượng (latency, concurrency, uptime) **chỉ quy định tại Mục 6 (NFR)** để tránh trùng lặp.

| Hạng mục | Ràng buộc |
|---|---|
| Nền tảng client | Chỉ iOS/Android, React Native + TypeScript (strict mode), Expo development build/prebuild (DEC-01/02). |
| Giao thức | REST (JSON) cho CRUD, WebSocket (WSS) cho real-time, WebRTC cho media, qua TLS (NFR-SEC-01). |
| Kiến trúc client | Layered architecture bắt buộc: UI không gọi trực tiếp REST/WebSocket (Mục 7.3.1). |
| Kiến trúc backend | Modular monolith NestJS, module tách theo bounded context để có thể tách service sau. |
| ID | Snowflake ID cho mọi entity chính; serialize string trong JSON. |
| Thời gian | Server lưu UTC (`TIMESTAMPTZ`); tính toán "theo ngày" của Study dùng timezone của user. |
| Ngôn ngữ | UI tiếng Việt (mặc định) và tiếng Anh (NFR-I18N). |
| Chính sách store | Tuân thủ App Store Review Guidelines và Google Play Policy (R8). |

### 2.6. Kế hoạch phát hành (Release Plan)

| Release | Mục tiêu | Nhóm REQ chính |
|---|---|---|
| **MVP-Đồ án** | Demo end-to-end trên iOS + Android: đăng ký → tạo Study Server → mời bạn qua link → chat → vào phòng học im lặng → chạy Focus Session → khóa màn hình vẫn nhận chuyển pha → xem streak & việc cần làm → gọi 1-1. | AUTH-01/02/04/05/06/07/08/10/13 · SRV-01/02/03/04/05/07 · ROLE-01/02/03/04/05/07/08 · MSG-01/02/03/04/05/06/07/08/09/10/11 · VC-01/02/04/05/08 · SEARCH-02 · NOTIF-01/02/04 · FRIEND-01/02/03/04 · DM-01/02/03 · GDM-01/02/03 · DMVC-01/03 · STUDY-01/02/03/05/06/08/10/11/15 · SAFE-01 |
| **v1.x** | Hoàn thiện trải nghiệm & bảo mật nâng cao. | AUTH-03/09/11/12 · SRV-06 · ROLE-06 · VC-03 (Android)/07 · SEARCH-01/03 · NOTIF-03 · DMVC-02/04 · STUDY-04/09/12/13 · SAFE-02 |
| **Backlog** | Theo nhu cầu thực tế (Mục 10). | VC-03 (iOS) · VC-06 · STUDY-07/14 · REQ-PREM-* · REQ-BKLG-* |

### 2.7. Chỉ số thành công sản phẩm (Success Metrics / KPI)

| KPI | Định nghĩa | Mục tiêu đồ án (pilot ≥ 30 user thật trong 2 tuần) |
|---|---|---|
| Activation rate | % user mới hoàn thành ≥ 1 Focus Session (solo hoặc nhóm) trong 24h đầu | ≥ 50% |
| Weekly focused minutes / active user | Trung bình focused minutes mỗi user hoạt động trong tuần | ≥ 120 phút |
| D7 retention | % user quay lại vào ngày thứ 7 sau đăng ký | ≥ 25% |
| Streak ≥ 3 ngày | % user active có streak ≥ 3 | ≥ 20% |
| Push opt-in rate | % user cấp quyền thông báo | ≥ 70% |
| Group study ratio | % focused minutes trong Focus Session nhóm (so với solo) | Theo dõi |
| Task completion | % task có deadline được chuyển `done` trước deadline | Theo dõi |
| Crash-free sessions | % phiên không crash (iOS và Android riêng) | ≥ 99% (NFR-MOB-08) |

**Tracking plan (analytics):** `user_signed_up`, `email_verified`, `guild_created{template}`, `invite_accepted{source: deeplink|code}`, `focus_session_started{mode: group|solo, cycle}`, `focus_phase_completed{phase}`, `focus_session_left{focused_minutes}`, `streak_updated{days}`, `task_created`, `task_status_changed{to}`, `voice_joined{voice_mode}`, `push_permission{granted}`, `push_opened{type}`. Event không chứa nội dung tin nhắn hay PII ngoài `user_id` đã pseudonymize (NFR-PRIV-03); user có thể tắt analytics trong Quyền riêng tư.

### 2.8. Rủi ro (Risks)

| ID | Rủi ro | Khả năng | Ảnh hưởng | Biện pháp giảm thiểu |
|---|---|---|---|---|
| RISK-01 | Phạm vi quá lớn so với thời gian đồ án | Cao | Cao | Release Plan (2.6), backend monolith (7.4), review scope mỗi sprint. |
| RISK-02 | CallKit/PushKit phức tạp; vi phạm quy tắc PushKit khiến iOS ngừng giao VoIP push | Trung bình | Cao | REQ-DMVC-04 ở v1.x (DEC-07); luôn báo CallKit cho mọi VoIP push (7.3.4). |
| RISK-03 | WebRTC thất bại sau NAT (4G CGNAT, wifi trường chặn UDP) | Cao | Cao | TURN/TLS 443 bắt buộc (REQ-VC-08); test mạng thật (9.3). |
| RISK-04 | OS kill/suspend socket khi background làm mất real-time/giờ học | Cao | Trung bình | Push song song WebSocket (7.3.3), DEC-06. |
| RISK-05 | Native module thiếu Expo config plugin chính thức | Trung bình | Trung bình | Viết local config plugin trong repo; ưu tiên thư viện có plugin (ADR-02). |
| RISK-06 | Khác biệt hành vi Android theo hãng (tối ưu pin mạnh trên Xiaomi/Oppo/Samsung) làm chết foreground service/notification | Cao | Trung bình | Hướng dẫn user tắt tối ưu pin cho app khi cần; test trên máy thật nhiều hãng (9.3). |
| RISK-07 | Gian lận streak/leaderboard | Trung bình | Thấp | Server là nguồn sự thật về thời gian; giới hạn đổi timezone (REQ-STUDY-03). |
| RISK-08 | Rò rỉ dữ liệu cá nhân / không tuân thủ pháp lý | Thấp | Cao | NFR-PRIV, NFR-SEC, Mục 2.9, TBD-01. |
| RISK-09 | Chi phí voice/TURN vượt free tier khi demo | Trung bình | Trung bình | Giới hạn user/room khi demo, Silent Room mặc định tắt camera, giám sát băng thông. |
| RISK-10 | Bị từ chối khi review App Store (thiếu xóa tài khoản, thiếu quyền giải thích microphone/camera, background mode không hợp lệ) | Trung bình | Cao | Checklist store (9.6); chỉ khai báo background mode thực sự dùng (7.3.3). |

### 2.9. Ràng buộc pháp lý & chính sách nền tảng

| Hạng mục | Yêu cầu | Trạng thái |
|---|---|---|
| Bảo vệ dữ liệu cá nhân (R7 — NĐ 13/2023/NĐ-CP) | Thông báo & xin đồng ý xử lý dữ liệu khi đăng ký; Privacy Policy truy cập được trong app; user xem/xuất/xóa dữ liệu của mình; quy trình xử lý sự cố lộ lọt. Chi tiết tại NFR-PRIV. | Đã chốt — bắt buộc từ MVP (Privacy Policy + consent + REQ-AUTH-10). |
| Quản lý dịch vụ mạng xã hội / định danh tài khoản (NĐ 147/2024/NĐ-CP) | **Cần kiểm tra / tham vấn người có chuyên môn pháp lý** về việc có thuộc diện áp dụng và nghĩa vụ cụ thể khi phát hành công khai. SRS không khẳng định chi tiết nghĩa vụ. | **TBD-01** |
| Xóa tài khoản trong app (R8) | App cho phép tạo tài khoản phải cho phép xóa tài khoản ngay trong app. | Đã chốt — REQ-AUTH-10. |
| Đăng nhập bên thứ ba trên iOS (R8) | Cung cấp Sign in with Apple cùng lúc với Google Sign-In. | Đã chốt — DEC-08, REQ-AUTH-12. |
| VoIP push & CallKit (R8) | Mỗi VoIP push trên iOS phải được báo tới CallKit như một cuộc gọi. | Đã chốt — REQ-DMVC-04, 7.3.4. |
| Quyền truy cập thiết bị | Chuỗi giải thích (usage description) rõ ràng cho microphone, camera, thông báo, ảnh; chỉ xin quyền tại thời điểm cần (NFR-MOB-10). | Đã chốt. |
| Bản quyền & nhận diện | Không dùng logo/tên/màu nhận diện của Discord; nhạc nền phải có license (DEC-09). | Đã chốt. |
| Độ tuổi | Người dùng từ 18 tuổi trở lên (DEC-18), khai báo trong Điều khoản và rating trên store. | Đã chốt. |

---

## 3. UX/UI & HÀNH VI CHUNG (MOBILE)

> Toàn bộ mục này áp dụng cho app iOS/Android (DEC-01). Layout nhiều cột và phím tắt của Web/Desktop ở v1.0 đã bị bỏ.

### 3.1. Điều hướng tổng thể (Navigation Model)

Điều hướng theo mô hình Discord mobile: **Bottom Tab Bar** + **drawer vuốt trái/phải** trong tab Home.

```
┌──────────────────────────────────────────┐
│  [≡] # hoi-dap              [🔍] [👥]     │  ← Header kênh: mở drawer trái / tìm kiếm / drawer phải
├──────────────────────────────────────────┤
│                                          │
│            Nội dung kênh                 │  ← Chat / Voice room / Focus timer
│      (vuốt phải → drawer trái,           │
│       vuốt trái → drawer phải)           │
│                                          │
├──────────────────────────────────────────┤
│ [ô nhập tin nhắn] [+] [😊] [➤]           │
├──────────────────────────────────────────┤
│  🏠 Home │ ⏱ Học │ ✅ Việc │ 🔔 Thông báo │ 👤 Cá nhân │ ← Bottom tabs
└──────────────────────────────────────────┘

Drawer trái (vuốt phải):                 Drawer phải (vuốt trái):
┌────┬───────────────────────┐          ┌───────────────────────┐
│ 💬 │  Tên Study Server  ⚙  │          │ [Thành viên][Task][📌] │
│ ── │  ▾ THÔNG TIN          │          │  Online — 3            │
│ G1 │    # thong-bao        │          │   👤 An (Trưởng nhóm)  │
│ G2 │    # tai-lieu         │          │   👤 Bình              │
│ G3 │  ▾ PHÒNG HỌC          │          │  Offline — 5           │
│ ＋ │    🔊 Phòng học im lặng│          │   ...                  │
│    │       Focus 12:34 👤👤 │          │                        │
│    │    🔊 Phòng thảo luận  │          │                        │
└────┴───────────────────────┘          └───────────────────────┘
 Guild rail (72dp): 💬 = Tin nhắn trực tiếp (Friends/DM), G1..Gn = guild, ＋ = tạo/tham gia
```

| Thành phần | Hành vi |
|---|---|
| **Tab Home** | Màn hình chat của kênh mở gần nhất; drawer trái chứa guild rail + channel list; drawer phải chứa Member list / Task Board / Pinned (chuyển bằng segmented control). Icon 💬 trên rail mở danh sách DM & Friends. |
| **Tab Học** | Solo Focus Timer (REQ-STUDY-10), danh sách "Phòng học đang có bạn bè" (voice channel có Focus Session đang chạy trong các guild của user), streak & thống kê tuần, leaderboard (nếu guild bật). |
| **Tab Việc cần làm** | Study Digest (REQ-STUDY-08): task được gán cho user từ mọi guild, nhóm theo Quá hạn / Hôm nay / Tuần này / Sau đó. |
| **Tab Thông báo** | Hộp thư thông báo trong app: mention, reply, lời mời kết bạn, nhắc deadline, lời mời vào phòng học, cuộc gọi nhỡ. Badge số chưa đọc. |
| **Tab Cá nhân** | Profile, trạng thái (presence), Cài đặt (Tài khoản, Quyền riêng tư, Thông báo & Chế độ Học tập, Giao diện, Ngôn ngữ, Timezone), Đăng xuất. |
| **Mini-player** | Khi đang trong voice/Focus Session và rời màn hình voice: thanh nổi phía trên bottom tab hiển thị tên kênh, pha & thời gian còn lại, nút mute/rời phòng. |
| **Cử chỉ** | Vuốt phải mở drawer trái; vuốt trái mở drawer phải; nhấn giữ tin nhắn mở action sheet (Reply, React, Sửa, Xóa, Ghim, Sao chép, Báo cáo); nút Back hệ thống (Android) đóng drawer/sheet trước khi rời màn hình. |
| **Deep link** | `https://nezfocus.app/invite/<code>`, `https://nezfocus.app/channels/<guild_id>/<channel_id>`, `https://nezfocus.app/tasks/<task_id>`, `https://nezfocus.app/verify?token=…`, `https://nezfocus.app/reset?token=…` mở đúng màn hình (REQ-SRV-07). |

### 3.2. Sơ đồ màn hình (Mobile Sitemap)

```
Auth stack
 ├─ Welcome
 ├─ Đăng ký → Kiểm tra email → (deep link) Xác thực thành công
 ├─ Đăng nhập → (2FA TOTP / Recovery code) [v1.x]
 ├─ Đăng nhập Google / Apple [v1.x]
 └─ Quên mật khẩu → (deep link) Đặt lại mật khẩu
Onboarding stack (lần đầu)
 ├─ Chọn timezone (mặc định theo máy) & ngôn ngữ
 ├─ Xin quyền thông báo (có màn hình giải thích trước)
 └─ Tạo Study Server / Nhập mã-link mời / Bỏ qua
Main (Bottom tabs)
 ├─ Home
 │   ├─ Channel (Text): Chat · Reply · Upload · Search trong kênh [v1.x]
 │   ├─ Channel (Voice): Voice room · Focus Session controls · Screen share view [v1.x]
 │   ├─ Drawer phải: Members · Task Board · Pinned
 │   ├─ Guild Settings: Tổng quan · Roles · Members · Invites · Bans · Audit Log [v1.x] · Study settings · Events [v1.x]
 │   ├─ Channel Settings: Tổng quan · Quyền (overwrites) · Voice mode · Slow mode [v1.x]
 │   ├─ Create/Join Guild (template General / Study Server)
 │   └─ DM & Friends: Friends (Online/Tất cả/Đang chờ/Đã chặn) · DM · Group DM · Call screen
 ├─ Học: Solo Focus · Phòng học đang hoạt động · Streak & thống kê · Leaderboard [v1.x]
 ├─ Việc cần làm: Danh sách tổng hợp → Task detail (nhảy về kênh gốc)
 ├─ Thông báo: Inbox
 └─ Cá nhân: Profile · Tài khoản (email [v1.x], mật khẩu, 2FA [v1.x], phiên đăng nhập, xóa tài khoản)
             · Quyền riêng tư (DM từ thành viên server, analytics, ẩn khỏi leaderboard, danh sách chặn)
             · Thông báo (mức theo guild [v1.x], Chế độ Học tập [v1.x]) · Giao diện · Ngôn ngữ · Timezone
Modal / Overlay
 ├─ Incoming Call (trong app — MVP; CallKit/ConnectionService — v1.x)
 ├─ Invite preview
 ├─ Report dialog
 └─ Admin (chỉ tài khoản Platform Admin) [v1.x]
```

> **Yêu cầu tài liệu kèm theo:** wireframe/hi-fi cho toàn bộ màn hình trên phải có trong file Figma của dự án, đường dẫn ghi vào Phụ lục 11.4 trước sprint UI đầu tiên.

### 3.3. Giao diện & Nhận diện thương hiệu (Theme & Branding)

- **Dark Mode mặc định** khi user chưa chọn; tùy chọn: Tối / Sáng / **Theo hệ thống** (theo `Appearance` của iOS/Android). Preference lưu theo tài khoản (`user_settings.theme`) để đồng bộ giữa các thiết bị của user.
- **Bảng màu riêng của NezFocus** (không dùng mã màu nhận diện của Discord như v1.0): định nghĩa bằng design tokens (`bg.primary`, `bg.sidebar`, `accent.focus`, `accent.break`, `status.danger`…) dùng chung giữa Figma và code (file theme TypeScript). Tông chủ đạo dịu mắt, gợi cảm giác tập trung.
- Màu pha `focus`/`break` phải phân biệt được với người mù màu (kèm icon/nhãn, không chỉ dựa vào màu — NFR-A11Y-02).
- Hỗ trợ Dynamic Type (iOS) / Font scale (Android) tới 200% không vỡ layout (NFR-A11Y-03).
- Icon app, splash screen, màu status bar/navigation bar theo theme.

### 3.4. Hành vi chung (General Behavior)

- **Optimistic UI:** tin nhắn gửi đi hiển thị ngay ở trạng thái `pending` (mờ), chuyển `sent` khi server ACK, hoặc `failed` kèm nút Thử lại. Retry dùng lại cùng `nonce` (REQ-MSG-09). State machine tại 8.4.5.
- **Typing Indicator:** "X đang nhập…" tự hết hạn sau 10 giây (REQ-MSG-06).
- **Gửi tin:** nút ➤ để gửi; phím Enter trên bàn phím ảo **xuống dòng** (không gửi). Nếu dùng bàn phím vật lý thì không có hành vi đặc biệt.
- **Toast/Snackbar:** lỗi không chặn thao tác (rate limit, gửi thất bại); banner cố định "Đang kết nối lại…" khi Gateway mất kết nối > 3 giây khi app ở foreground.
- **Haptics:** rung nhẹ khi chuyển pha Focus (nếu app foreground), khi gửi reaction; tôn trọng cài đặt tắt rung của hệ thống.
- **Pull-to-refresh** cho danh sách (DM, task, thông báo); cuộn lên đầu kênh để tải tin cũ (cursor pagination).
- **Xác nhận hành động nguy hiểm:** xóa kênh/guild/tài khoản, ban, chuyển quyền Owner dùng dialog xác nhận (nhập lại tên hoặc mật khẩu).
- **Âm thanh:** âm báo chuyển pha nhẹ, tôn trọng chế độ im lặng của máy (iOS silent switch / Android Do Not Disturb) trừ khi user bật "Luôn phát âm báo Focus".

### 3.5. Trạng thái màn hình (Empty / Loading / Error / Offline)

| Trạng thái | Yêu cầu |
|---|---|
| Loading | Skeleton cho danh sách (guild, channel, message, task); không dùng spinner toàn màn hình quá 1 giây. |
| Empty | Mỗi danh sách có empty state kèm hành động gợi ý (chưa có guild → "Tạo Study Server"/"Nhập mã mời"; chưa có task → "Thêm việc đầu tiên"; chưa có streak → "Bắt đầu Focus 25 phút"). |
| Error | Thông điệp tiếng Việt dễ hiểu theo mã lỗi (Phụ lục 11.1), nút thử lại; không hiển thị stack trace/mã nội bộ thô. |
| Offline | Hiển thị dữ liệu cache (7.3.2) + banner "Bạn đang offline"; vẫn cho soạn tin (vào outbox); nút vào voice bị vô hiệu kèm giải thích. |
| Permission denied | Kênh không có quyền gửi: ô nhập bị vô hiệu kèm lý do ("Bạn không có quyền gửi tin nhắn trong kênh này" / "Bạn đang bị timeout đến HH:mm"). |
| Thiếu quyền hệ điều hành | Micro/camera/thông báo bị từ chối: màn hình giải thích + nút mở Cài đặt hệ thống. |

### 3.6. Luồng người dùng chính (Key User Flows)

| Flow | Các bước chính | REQ |
|---|---|---|
| UF-01 Đăng ký & kích hoạt | Đăng ký → nhận email → bấm link (deep link mở app) → đã xác thực → onboarding | AUTH-01/13, SRV-07 |
| UF-02 Tạo Study Server & mời bạn | Tạo guild chọn template Study → sinh cấu trúc kênh → tạo invite → chia sẻ qua share sheet hệ thống (Zalo, Messenger…) | SRV-01/03, STUDY-05 |
| UF-03 Tham gia qua link | Bấm link → app mở màn hình preview (chưa cài: trang chuyển hướng tới store) → Join → vào `#thong-bao` | SRV-04/07 |
| UF-04 Học nhóm | Vào "Phòng học im lặng" (mic tắt sẵn) → Start Focus 25/5 → khóa màn hình → nhận local notification chuyển pha → rời phòng → streak cập nhật | STUDY-01/02/03/11/15 |
| UF-05 Học một mình | Tab Học → Solo Focus 50/10 → khóa màn hình → hoàn thành → streak cập nhật | STUDY-10/03/15 |
| UF-06 Giao việc đồ án | Mở Task Board của `#do-an` → thêm task, gán người, deadline → người được gán nhận push nhắc trước 2h | STUDY-06/08 |
| UF-07 Gọi 1-1 | Mở DM → bấm gọi → người nhận thấy màn hình cuộc gọi đến (push nếu app nền) → Chấp nhận / Từ chối / nhỡ sau 45s | DMVC-01/03 |

### 3.7. Ma trận hỗ trợ tính năng theo nền tảng (Platform Matrix)

| Tính năng | Android | iOS | Ghi chú |
|---|---|---|---|
| Chat, file, reaction, pin, reply | MVP | MVP | |
| Voice channel (audio) | MVP | MVP | |
| Video camera trong voice/call | MVP | MVP | |
| Xem screen share của người khác | v1.x | v1.x | |
| Phát screen share | v1.x | Backlog | DEC-11. Android: MediaProjection + foreground service `mediaProjection`. iOS: cần Broadcast Upload Extension. |
| Focus Session / Solo Focus | MVP | MVP | |
| Local notification chuyển pha khi app nền/khóa màn hình | MVP | MVP | Android 12+ cần quyền exact alarm hoặc chấp nhận sai lệch (7.3.6). |
| Push notification | MVP (FCM) | MVP (APNs qua FCM) | |
| Cuộc gọi đến khi app ở nền (push ưu tiên cao) | MVP | MVP | REQ-DMVC-01. |
| Cuộc gọi đến khi app bị kill, UI gọi gốc | v1.x (ConnectionService/full-screen intent) | v1.x (PushKit + CallKit) | REQ-DMVC-04. |
| Giữ voice khi khóa màn hình | MVP (foreground service `microphone`) | MVP (background mode `audio`/`voip`) | DEC-06. |
| Deep link invite | MVP (App Links) | MVP (Universal Links) | |
| Offline cache & outbox | MVP | MVP | |
| Google Sign-In / Sign in with Apple | v1.x | v1.x | DEC-08; Sign in with Apple trên Android qua luồng web OAuth. |
| Biometric unlock app | Backlog | Backlog | |

---

## 4. YÊU CẦU CHỨC NĂNG CHI TIẾT (FUNCTIONAL REQUIREMENTS)

> Cột **Release** và **Depends-on** theo quy ước Mục 0.3. 🆕 = REQ mới ở v1.1; ✏️ = REQ được sửa so với v1.0.

### 4.1. Auth & Profile (`REQ-AUTH-*`)

| REQ-ID | Tên chức năng | Mô tả chi tiết | Priority | Release | Depends-on | Acceptance Criteria |
|---|---|---|---|---|---|---|
| REQ-AUTH-01 ✏️ | Đăng ký tài khoản | Đăng ký bằng email + username + password + xác nhận ≥ 18 tuổi (DEC-18) + đồng ý Điều khoản & Chính sách quyền riêng tư (checkbox bắt buộc, lưu thời điểm và phiên bản chính sách đã đồng ý). `username` unique toàn hệ thống, chuẩn hóa chữ thường, 2–32 ký tự `a-z0-9_.` (DEC-03). Password 8–128 ký tự, không nằm trong danh sách mật khẩu phổ biến. Gửi email xác thực (link deep link, TTL 24h). | Must | MVP-Đồ án | — | **AC1:** Given email & username hợp lệ chưa tồn tại, When submit, Then tài khoản tạo ở trạng thái `unverified`, email xác thực được gửi trong ≤ 5 giây. **AC2 (âm):** Given email hoặc username đã tồn tại, Then trả `40901` với thông điệp chung "Email hoặc username không khả dụng". **AC3 (âm):** Given username `Nez Focus!`, Then trả `40001` kèm lỗi field `username`. **AC4 (âm):** Given chưa tick đồng ý điều khoản, Then `40001`. |
| REQ-AUTH-02 ✏️ | Đăng nhập & chống brute-force | Đăng nhập bằng email hoặc username + password, trả Access Token (JWT, TTL 15 phút) và Refresh Token (TTL 30 ngày). Token lưu bằng `react-native-keychain` (iOS Keychain / Android Keystore), **không** lưu trong AsyncStorage. **Không khóa cứng tài khoản** (tránh kẻ xấu cố tình khóa tài khoản người khác): sau 5 lần sai trong 10 phút theo cặp (tài khoản, IP) → yêu cầu CAPTCHA; sau 20 lần sai/giờ theo IP → `40029`. Gửi email cảnh báo khi đăng nhập thành công từ thiết bị mới. | Must | MVP-Đồ án | AUTH-01 | **AC1:** Given credential đúng, When login, Then nhận access + refresh token và `session_id`. **AC2 (âm):** Given sai password, Then `40102` (không phân biệt sai email hay sai password). **AC3 (âm):** Given đã sai 5 lần trong 10 phút từ cùng IP cho cùng tài khoản, When login lần 6 không kèm CAPTCHA, Then `40105`; đăng nhập từ IP khác của chủ tài khoản không bị chặn. **AC4:** Given user bật 2FA (v1.x), When password đúng, Then trả `40104` kèm `mfa_ticket` (TTL 5 phút). **AC5:** Given app bị gỡ và cài lại, Then token cũ không còn dùng được trên thiết bị (phải đăng nhập lại). |
| REQ-AUTH-03 ✏️ | Xác thực hai yếu tố (TOTP) | Bật/tắt 2FA qua TOTP (RFC 6238). Khi bật, **mọi lần đăng nhập** yêu cầu mã TOTP, trừ thiết bị user đã tick "Tin cậy thiết bị này trong 30 ngày" (thiết bị tin cậy = refresh-token family có `trusted_until`). Bật 2FA yêu cầu xác nhận 1 mã hợp lệ. `mfa_secret` mã hóa at-rest (NFR-SEC-06). | Should | v1.x | AUTH-02 | **AC1:** Given 2FA bật, thiết bị chưa tin cậy, When nhập mã đúng kèm `mfa_ticket`, Then cấp token. **AC2 (âm):** Given sai mã 3 lần trên cùng `mfa_ticket`, Then ticket bị hủy, trả `40101`; sai tổng 10 lần/giờ → `40029`. **AC3:** Given thiết bị tin cậy còn hạn, Then không yêu cầu TOTP. |
| REQ-AUTH-04 ✏️ | Refresh Token Rotation & phát hiện tái sử dụng | Mỗi lần refresh cấp refresh token mới và vô hiệu token cũ. Token thuộc một family (1 family = 1 phiên đăng nhập = 1 thiết bị). Phát hiện token đã dùng bị dùng lại → **thu hồi toàn bộ family** và ngắt Gateway của phiên đó. Client dùng cơ chế "single-flight" để nhiều request đồng thời chỉ gọi refresh một lần. | Must | MVP-Đồ án | AUTH-02 | **AC1:** Given refresh token hợp lệ chưa dùng, When gọi refresh, Then nhận cặp token mới. **AC2 (âm):** Given refresh token đã dùng, When gọi lại, Then `40103`, toàn bộ family bị thu hồi, Gateway session nhận close code `4004`. **AC3 (âm):** Given refresh token hết hạn, Then `40100`, app chuyển về màn hình đăng nhập. **AC4:** Given 5 request API cùng lúc gặp access token hết hạn, Then client chỉ gửi 1 request refresh. |
| REQ-AUTH-05 | Quản lý Presence | User đặt trạng thái thủ công: Online/Idle/DND/Invisible (`user_settings.preferred_status`); presence thực tế lưu ở Redis. Tự chuyển Online→Idle sau 10 phút không tương tác hoặc khi app ở nền > 5 phút. Presence tổng hợp đa thiết bị: Online nếu ≥ 1 thiết bị active. | Must | MVP-Đồ án | Gateway (5.4) | **AC1:** Given user không tương tác 10 phút, Then presence `idle` được broadcast tới member guild chung và bạn bè. **AC2:** Given user đặt Invisible, Then người khác thấy `offline` nhưng user dùng app bình thường. **AC3:** Given mọi thiết bị mất kết nối Gateway, Then sau ≤ 60 giây presence chuyển `offline`. |
| REQ-AUTH-06 ✏️ | Cập nhật hồ sơ cá nhân | Chỉnh display name, avatar (chọn từ thư viện ảnh/chụp mới, crop vuông trên app), bio, banner, accent color, per-guild nickname. **MVP chỉ nhận ảnh tĩnh** jpg/png/webp/heic ≤ 8MB; app nén ảnh trước khi upload; server chuyển đổi sang webp và resize 128/256/512px. Avatar động (GIF) là tính năng Premium (REQ-PREM-03, Backlog). | Should | MVP-Đồ án | AUTH-01 | **AC1:** Given ảnh png 2MB, When upload, Then response profile chứa URL các kích thước. **AC2 (âm):** Given file GIF, Then `40014`. **AC3 (âm):** Given ảnh 9MB sau nén, Then `40013`. **AC4:** Given ảnh HEIC chụp từ iPhone, Then upload thành công. |
| REQ-AUTH-07 | Đăng xuất & thu hồi phiên | Đăng xuất thiết bị hiện tại (xóa token khỏi Keychain/Keystore, xóa cache cục bộ, gỡ push token). Màn hình Phiên đăng nhập liệt kê thiết bị (tên máy, nền tảng, IP gần đúng, hoạt động cuối), cho phép thu hồi từng phiên hoặc tất cả phiên khác. | Should | MVP-Đồ án | AUTH-04 | **AC1:** Given user chọn "Thu hồi" một phiên, Then family của phiên đó bị vô hiệu ngay, Gateway nhận close code `4004`, push token của thiết bị đó bị xóa. **AC2:** Given đăng xuất, Then CSDL cục bộ bị xóa và không còn thông báo nào gửi tới thiết bị. |
| REQ-AUTH-08 🆕 | Quên / Đặt lại mật khẩu | Nhập email → luôn trả thông điệp chung "Nếu email tồn tại, bạn sẽ nhận được link" (chống dò email). Link reset (deep link) dùng 1 lần, TTL 30 phút. Đặt lại thành công → thu hồi mọi phiên và gửi email thông báo. | Must | MVP-Đồ án | AUTH-01 | **AC1:** Given email tồn tại, Then email gửi trong ≤ 5 giây. **AC2 (âm):** Given email không tồn tại, Then response giống hệt AC1 (HTTP 202, cùng body). **AC3 (âm):** Given link đã dùng hoặc quá 30 phút, Then `40002`. **AC4:** Given reset thành công, Then mọi refresh token cũ bị thu hồi. **AC5 (âm):** Given > 3 yêu cầu/giờ cho cùng email, Then `40029`. |
| REQ-AUTH-09 🆕 | Đổi email | Yêu cầu nhập lại mật khẩu (và TOTP nếu bật), gửi link xác nhận tới email mới; email cũ nhận thông báo kèm link "Không phải tôi" để hoàn tác trong 7 ngày. | Should | v1.x | AUTH-01 | **AC1:** Given user xác nhận link ở email mới, Then email được cập nhật, email cũ nhận thông báo. **AC2 (âm):** Given email mới đã thuộc tài khoản khác, Then `40901`. |
| REQ-AUTH-10 🆕 | Xóa tài khoản trong app | Settings > Tài khoản > Xóa tài khoản, yêu cầu nhập lại mật khẩu. Nếu user là Owner của guild còn member khác → phải chuyển quyền Owner hoặc xóa guild trước. Tài khoản vào `pending_deletion` 14 ngày (đăng nhập lại để hủy), sau đó: xóa PII (email, avatar, bio, push token, dữ liệu học tập cá nhân), tin nhắn giữ lại nhưng tác giả hiển thị "Người dùng đã xóa", xóa quan hệ bạn bè/block. | Must | MVP-Đồ án | AUTH-02, SRV-05 | **AC1:** Given user không sở hữu guild có member khác, When xác nhận xóa, Then mọi phiên bị thu hồi, tài khoản ở `pending_deletion`. **AC2 (âm):** Given user là Owner guild có 5 member, Then `40010` kèm danh sách guild cần xử lý. **AC3:** Given qua 14 ngày, When job purge chạy, Then PII bị xóa/ẩn danh, tin nhắn hiển thị "Người dùng đã xóa". **AC4:** Given đăng nhập lại trong 14 ngày, Then app hỏi và cho phép hủy yêu cầu xóa. |
| REQ-AUTH-11 🆕 | Mã khôi phục 2FA | Khi bật 2FA, sinh 10 recovery code dùng 1 lần (hiển thị 1 lần, cho phép sao chép/lưu, lưu server dạng hash). Có thể tái sinh bộ mới (vô hiệu bộ cũ) sau khi xác thực lại. | Should | v1.x | AUTH-03 | **AC1:** Given mất thiết bị TOTP, When nhập 1 recovery code hợp lệ, Then đăng nhập thành công và mã đó không dùng lại được. **AC2 (âm):** Given recovery code đã dùng, Then `40101`. |
| REQ-AUTH-12 🆕 | Google Sign-In & Sign in with Apple | Đăng nhập/đăng ký bằng Google và Apple (OIDC), triển khai cùng lúc (DEC-08). Tài khoản mới tạo ở trạng thái `verified`, yêu cầu chọn username. Nếu email trùng tài khoản đã có (đăng ký bằng mật khẩu) → yêu cầu đăng nhập bằng mật khẩu một lần để liên kết (không tự động gộp). Hỗ trợ email ẩn danh (private relay) của Apple. | Should | v1.x | AUTH-01 | **AC1:** Given email Google chưa có tài khoản, When đăng nhập Google, Then tạo tài khoản `verified`, chuyển tới màn hình chọn username. **AC2 (âm):** Given email trùng tài khoản hiện có chưa liên kết, Then `40906` và app chuyển tới luồng liên kết. **AC3:** Given user Apple chọn "Ẩn email", Then tài khoản dùng địa chỉ relay và vẫn nhận được email hệ thống. |
| REQ-AUTH-13 🆕 | Giới hạn tài khoản chưa xác thực | User `unverified` được đăng nhập, xem guild đã tham gia, đọc tin nhắn; **không được** gửi tin nhắn, tạo guild, tạo invite, gửi friend request, gọi thoại, vào voice. Gửi lại email xác thực tối đa 1 lần/phút, 10 lần/ngày. Tài khoản chưa xác thực sau 7 ngày bị xóa tự động. | Must | MVP-Đồ án | AUTH-01 | **AC1 (âm):** Given user `unverified`, When gửi tin nhắn, Then `40305`. **AC2:** Given bấm link xác thực hợp lệ, Then trạng thái `verified`, Gateway gửi `USER_UPDATE`, app tự mở khóa chức năng. **AC3 (âm):** Given gửi lại email lần 2 trong 1 phút, Then `40029`. |

### 4.2. Guild & Channels (`REQ-SRV-*`)

| REQ-ID | Tên chức năng | Mô tả chi tiết | Priority | Release | Depends-on | Acceptance Criteria |
|---|---|---|---|---|---|---|
| REQ-SRV-01 ✏️ | Tạo Guild mới | User `verified` tạo guild với tên (2–100 ký tự), icon tùy chọn, chọn template `general` hoặc `study_server` (REQ-STUDY-05). Người tạo là Owner; role `@everyone` (id = guild_id) tạo tự động với quyền mặc định (8.5.3). Template `general` sinh: 1 category "Kênh chung", text channel `general`, voice channel "Phòng thoại chung". Giới hạn 100 guild/user. | Must | MVP-Đồ án | AUTH-13 | **AC1:** Given tên hợp lệ, template general, Then guild có đúng 1 category, 1 text channel `general`, 1 voice channel "Phòng thoại chung", role `@everyone`, Owner = người tạo. **AC2 (âm):** Given tên 1 ký tự, Then `40001`. **AC3 (âm):** Given user đã ở 100 guild, Then `40011`. |
| REQ-SRV-02 ✏️ | Quản lý Category & Channel | CRUD Category/Text/Voice channel, sắp xếp bằng kéo-thả (nhấn giữ rồi kéo). **Quy tắc tên:** text channel tự chuẩn hóa dạng slug: chữ thường, khoảng trắng → `-`, **cho phép chữ cái tiếng Việt có dấu**, 1–100 ký tự; voice channel & category: văn bản tự do 1–100 ký tự. Voice channel có thuộc tính `voice_mode` (`normal`/`silent` — REQ-STUDY-02). Giới hạn 500 channel/guild. | Must | MVP-Đồ án | SRV-01, ROLE-08 | **AC1:** Given user có `MANAGE_CHANNELS`, When kéo thả đổi vị trí, Then thứ tự mới persist và `CHANNEL_UPDATE` tới member online trong ≤ 1 giây. **AC2:** Given tạo text channel "Hỏi Đáp Chung", Then tên lưu là `hỏi-đáp-chung`. **AC3:** Given tạo voice channel "Phòng học im lặng", Then tên giữ nguyên. **AC4 (âm):** Given không có `MANAGE_CHANNELS`, Then `40302`. **AC5:** Given xóa category còn channel con, Then channel con chuyển về không thuộc category nào. |
| REQ-SRV-03 ✏️ | Sinh Invite Link | Tạo invite với thời hạn (30 phút/1h/1 ngày/7 ngày/không giới hạn — mặc định 7 ngày) và giới hạn lượt dùng (1/5/10/25/50/100/không giới hạn). Mã 8–10 ký tự ngẫu nhiên (entropy ≥ 47 bit). Yêu cầu `CREATE_INVITE`. Chia sẻ qua share sheet hệ thống. | Must | MVP-Đồ án | SRV-01 | **AC1 (âm):** Given invite max_uses=5 đã dùng 5 lần, When lượt thứ 6, Then `40041`. **AC2 (âm):** Given invite hết hạn, Then `40041`. **AC3 (âm):** Given 1 IP tra cứu > 30 mã không tồn tại/phút, Then `40029`. |
| REQ-SRV-04 ✏️ | Tham gia Guild qua Invite | Join bằng link hoặc nhập mã; tự gán `@everyone`. Kiểm tra: không bị ban (REQ-ROLE-07), chưa là member, chưa vượt 100 guild. | Must | MVP-Đồ án | SRV-03, ROLE-07 | **AC1:** Given invite hợp lệ, When Join, Then user xuất hiện trong member list trong ≤ 1 giây (`GUILD_MEMBER_ADD`) và system message "X đã tham gia" xuất hiện ở system channel (REQ-MSG-11). **AC2 (âm):** Given user bị ban, Then `40306`, `uses` không tăng. **AC3:** Given đã là member, Then trả 200 kèm guild (idempotent), `uses` không tăng. |
| REQ-SRV-05 ✏️ | Rời/Xóa/Chuyển nhượng Guild | Member rời guild (Owner không rời được khi chưa chuyển quyền). Owner xóa guild (soft-delete 30 ngày, có thể khôi phục; sau đó purge). **Chuyển quyền Owner:** xác nhận bằng **nhập lại mật khẩu** (hoặc TOTP nếu đã bật 2FA). Owner cũ **giữ nguyên các role hiện có**, chỉ mất tư cách Owner. | Must | MVP-Đồ án | SRV-01, AUTH-02 | **AC1:** Given Owner chuyển quyền cho X và nhập đúng mật khẩu, Then `guilds.owner_id = X`, Owner cũ giữ role cũ, audit log ghi `GUILD_OWNER_TRANSFER`. **AC2 (âm):** Given sai mật khẩu, Then `40102`. **AC3 (âm):** Given Owner bấm "Rời server", Then `40012`. **AC4:** Given guild bị soft-delete, Then guild biến mất khỏi rail mọi member (`GUILD_DELETE`) và invite trả `40041`. |
| REQ-SRV-06 ✏️ | Slow Mode theo channel | Khoảng thời gian tối thiểu giữa 2 tin nhắn liên tiếp của cùng user trong 1 channel (0s–6h). User có `MANAGE_MESSAGES` hoặc `MANAGE_CHANNELS` được miễn. App hiển thị đếm ngược trên ô nhập. | Could | v1.x | MSG-01 | **AC1 (âm):** Given slow mode = 10s, When gửi tin thứ 2 trong 10s, Then `40030` kèm `retry_after`. **AC2:** Given moderator, Then không bị giới hạn. |
| REQ-SRV-07 🆕 | Deep Link & preview Invite | `https://nezfocus.app/invite/<code>` là Universal Link (iOS)/App Link (Android, `autoVerify`): đã cài app → mở màn hình preview invite trong app (tên, icon, số member, số đang online). Chưa cài → trang chuyển hướng tĩnh hiển thị tên guild + nút App Store/Google Play; sau khi cài và mở app lần đầu, app đề nghị dán mã mời (lấy từ clipboard khi user cho phép). Link xác thực email và reset mật khẩu dùng cùng cơ chế. | Should | MVP-Đồ án | SRV-03 | **AC1:** Given app đã cài, When bấm link trong Zalo/Messenger, Then app mở thẳng màn hình preview invite. **AC2:** Given chưa cài, Then trang chuyển hướng hiển thị trong ≤ 2 giây kèm nút tải app. **AC3 (âm):** Given invite hết hạn, Then màn hình preview hiển thị "Lời mời không còn hiệu lực". **AC4:** Given chưa đăng nhập bấm Join, Then sau khi đăng nhập/đăng ký, luồng tiếp tục join đúng guild đó. |

### 4.3. RBAC & Permissions (`REQ-ROLE-*`)

| REQ-ID | Tên chức năng | Mô tả chi tiết | Priority | Release | Depends-on | Acceptance Criteria |
|---|---|---|---|---|---|---|
| REQ-ROLE-01 ✏️ | Tạo & Sắp xếp Role | CRUD role (tên, màu, hoist, mentionable); thứ tự role quyết định hierarchy. Người có `MANAGE_ROLES` chỉ sửa/xóa/gán được role **thấp hơn** role cao nhất của mình và chỉ cấp các quyền mình đang có. Owner không bị giới hạn. `ADMINISTRATOR` **không** bỏ qua hierarchy. Giới hạn 250 role/guild. | Must | MVP-Đồ án | SRV-01 | **AC1 (âm):** Given role A cao hơn role B, When member chỉ có B (kể cả có `MANAGE_ROLES` hoặc `ADMINISTRATOR`) sửa A, Then `40301`. **AC2 (âm):** Given member có `MANAGE_ROLES` nhưng không có `BAN_MEMBERS`, When tạo role có `BAN_MEMBERS`, Then `40302`. **AC3:** Given Owner, Then sửa được mọi role. |
| REQ-ROLE-02 ✏️ | Permission Bitmap Guild-level | Mỗi role mang bitmap 64-bit; danh sách bit cố định tại Mục 8.5.1 (gồm các quyền Study: `MANAGE_STUDY_ROOMS`, `CREATE_TASKS`, `MANAGE_TASKS`, `MANAGE_EVENTS`). Quyền guild-level của member = OR bitmap `@everyone` và mọi role của member. | Must | MVP-Đồ án | ROLE-01 | **AC1:** Given role có `ADMINISTRATOR`, Then mọi kiểm tra quyền (trừ hierarchy) trả `true`. **AC2:** Given member có role X (`SEND_MESSAGES`) và Y (`CONNECT`), Then quyền guild-level gồm cả 2. |
| REQ-ROLE-03 ✏️ | Permission Overwrite Channel-level | Allow/deny bitmap theo channel cho role hoặc member. Thứ tự áp dụng **đúng thuật toán Mục 8.5.2**: base → Administrator/Owner → overwrite `@everyone` → overwrite các role (gộp: deny rồi allow) → overwrite member. Không có `VIEW_CHANNEL` ⇒ mất mọi quyền trong kênh. Channel con kế thừa overwrite của category khi "đồng bộ với category" được bật. | Must | MVP-Đồ án | ROLE-02 | **AC1:** Given role X deny `SEND_MESSAGES` ở channel C nhưng overwrite member Y allow `SEND_MESSAGES`, Then Y gửi được. **AC2:** Given role X deny và role Z allow `SEND_MESSAGES` ở C, member có cả X và Z, Then gửi được (allow của role thắng deny của role). **AC3 (âm):** Given `@everyone` deny `VIEW_CHANNEL` ở C và member không có overwrite allow, Then C không xuất hiện trong channel list và API trả `40401` (không lộ sự tồn tại). |
| REQ-ROLE-04 | Gán/Thu hồi Role cho Member | Người có `MANAGE_ROLES` gán/gỡ role, giới hạn bởi hierarchy của chính họ. | Must | MVP-Đồ án | ROLE-01 | **AC1 (âm):** Given moderator có role cao nhất ở vị trí 5, When gán role ở vị trí 6, Then `40301`. **AC2:** Given gán thành công, Then `GUILD_MEMBER_UPDATE` broadcast và quyền mới có hiệu lực ngay trên app của member (không cần khởi động lại). |
| REQ-ROLE-05 ✏️ | Kick / Ban / Timeout | Kick (rời guild, có thể quay lại bằng invite mới), Ban (chặn vĩnh viễn, tùy chọn xóa tin nhắn 0/1/7 ngày gần nhất), Timeout (60 giây – 28 ngày: chỉ còn `VIEW_CHANNEL` và `READ_MESSAGE_HISTORY`, không gửi tin/react/vào voice). Không tác động được lên người có role cao hơn hoặc bằng, hoặc lên Owner. Kèm lý do (tùy chọn). | Must | MVP-Đồ án | ROLE-04 | **AC1:** Given ban với `delete_message_days=7`, Then tin nhắn của user trong 7 ngày gần nhất trên mọi channel của guild bị soft-delete và `MESSAGE_DELETE_BULK` broadcast. **AC2 (âm):** Given user bị timeout gửi tin, Then `40307` kèm `timeout_until`. **AC3 (âm):** Given moderator ban Owner, Then `40301`. **AC4:** Given member đang trong voice bị kick/ban/timeout, Then bị ngắt khỏi voice ngay. |
| REQ-ROLE-06 | Audit Log | Ghi mọi hành động quản trị (ban/kick/timeout/role/channel/overwrite/invite/owner transfer/settings) kèm actor, target, thay đổi (before/after), lý do, thời điểm. Xem được bởi người có `VIEW_AUDIT_LOG`. Dữ liệu được ghi từ MVP; màn hình xem ở v1.x. | Should | v1.x | ROLE-05 | **AC1:** Given ban kèm reason "spam", When xem Audit Log, Then entry có đủ actor/target/action/reason/timestamp, mới nhất trước. **AC2 (âm):** Given user không có `VIEW_AUDIT_LOG`, Then `40302`. **AC3:** Given guild bị purge, Then audit log vẫn được giữ đủ thời hạn NFR-SEC-05. |
| REQ-ROLE-07 🆕 | Danh sách Ban & chặn rejoin | Ban lưu vào `guild_bans` (user_id, lý do, người ban, thời điểm). User bị ban không thể join lại bằng bất kỳ invite nào cho tới khi được unban. Người có `BAN_MEMBERS` xem danh sách và unban. | Must | MVP-Đồ án | ROLE-05 | **AC1 (âm):** Given user bị ban, When dùng invite mới, Then `40306`. **AC2:** Given unban, Then user join lại được bằng invite hợp lệ. |
| REQ-ROLE-08 🆕 | Kiểm tra quyền tập trung | Mọi endpoint và event Gateway kiểm tra quyền qua **một** module Permission dùng thuật toán 8.5.2; kết quả tính quyền được cache (Redis, TTL ngắn, xóa cache khi role/overwrite/member thay đổi). App dùng cùng thuật toán (thư viện TypeScript dùng chung) để ẩn/vô hiệu UI, nhưng server luôn là nơi quyết định. | Must | MVP-Đồ án | ROLE-02, ROLE-03 | **AC1:** Given bộ test ma trận quyền (≥ 50 case, 9.2), Then kết quả server và thư viện client trùng khớp 100%. **AC2 (âm):** Given client bị sửa để hiện nút xóa tin nhắn, When gọi API xóa tin của người khác không có quyền, Then `40302`. **AC3:** Given role bị gỡ quyền, Then quyền mới có hiệu lực trong ≤ 2 giây. |

### 4.4. Messaging & Fanout (`REQ-MSG-*`)

| REQ-ID | Tên chức năng | Mô tả chi tiết | Priority | Release | Depends-on | Acceptance Criteria |
|---|---|---|---|---|---|---|
| REQ-MSG-01 ✏️ | Gửi tin nhắn real-time | Gửi tin text (≤ 2000 ký tự) qua REST POST kèm `nonce`, server kiểm tra quyền → rate limit → persist → fanout `MESSAGE_CREATE` qua Gateway tới client đang subscribe channel, và kích hoạt Notification (push) cho người cần nhận. | Must | MVP-Đồ án | ROLE-08, MSG-09 | **AC1:** Given có `SEND_MESSAGES`, When POST message, Then persist và `MESSAGE_CREATE` tới mọi client online trong channel trong ≤ 300ms (P95, mục tiêu đồ án, NFR-PERF-01). **AC2 (âm):** Given nội dung rỗng và không có file, Then `40001`. **AC3 (âm):** Given 2001 ký tự, Then `40001`. **AC4 (âm):** Given không có quyền, Then `40302`. |
| REQ-MSG-02 ✏️ | Markdown & link preview | Markdown cơ bản (bold/italic/strikethrough/inline code/code block/quote/spoiler). App render bằng bộ parser giới hạn (không render HTML thô, không chạy link `javascript:`). Link preview (Open Graph) lấy **bất đồng bộ bởi worker phía server**, với biện pháp chống SSRF: chỉ `http/https`, chặn IP private/loopback/link-local/metadata (sau khi resolve DNS và sau mỗi redirect), tối đa 3 redirect, timeout 3 giây, tối đa 1MB. Ảnh preview được proxy qua server. | Should | MVP-Đồ án | MSG-01 | **AC1:** Given `**bold**`, Then hiển thị in đậm. **AC2:** Given link hợp lệ, Then embed card xuất hiện trong ≤ 3 giây qua `MESSAGE_UPDATE`, không làm chậm việc gửi tin. **AC3 (âm):** Given link `http://169.254.169.254/` hoặc domain resolve về `10.0.0.5`, Then không sinh preview và worker không gửi request tới địa chỉ đó. **AC4 (âm):** Given `[click](javascript:alert(1))`, Then hiển thị dạng text, không bấm được. |
| REQ-MSG-03 | Mentions | `@user`, `@role` (role `is_mentionable` hoặc người gửi có `MENTION_EVERYONE`), `@everyone`/`@here` (yêu cầu `MENTION_EVERYONE`), `#channel`. Có gợi ý autocomplete khi gõ `@`/`#`. | Must | MVP-Đồ án | MSG-01 | **AC1 (âm):** Given không có `MENTION_EVERYONE` gõ `@everyone`, Then text hiển thị nguyên văn, không kích hoạt thông báo hàng loạt, không trả lỗi. **AC2:** Given `@An` trong tin, Then An nhận mention badge và push (theo REQ-NOTIF-02/04). |
| REQ-MSG-04 ✏️ | Đính kèm File | Chọn ảnh/video từ thư viện, chụp ảnh, hoặc chọn file; tối đa 25MB/file, 10 file/tin. **Luồng upload:** app xin presigned URL (`POST /attachments`) kèm tên, kích thước, MIME → upload thẳng lên object storage (presigned có ràng buộc content-length) → gửi tin kèm `attachment_id`. Server kiểm tra magic bytes + quét virus trước khi file hiển thị cho người khác; sinh thumbnail cho ảnh/video. Hiển thị tiến trình upload, cho phép hủy. | Must | MVP-Đồ án | MSG-01 | **AC1 (âm):** Given file 25.1MB, Then app từ chối trước khi upload và server cũng trả `40013` nếu bị gọi trực tiếp. **AC2 (âm):** Given file đổi đuôi `.jpg` nhưng nội dung là file thực thi, Then `40014`, file bị xóa. **AC3 (âm):** Given file nhiễm virus (mẫu EICAR), Then tin nhắn không hiển thị file, người gửi nhận thông báo lỗi. **AC4:** Given mạng rớt giữa lúc upload, Then app cho phép thử lại mà không tạo tin nhắn trùng. |
| REQ-MSG-05 ✏️ | Edit/Delete/Reply/React | **Sửa** (chỉ tác giả; nhãn "đã chỉnh sửa"); **Xóa** (tác giả hoặc `MANAGE_MESSAGES`; soft-delete); **Reply** (preview tin gốc, nhảy tới tin gốc; nếu tin gốc đã xóa hiển thị "Tin nhắn gốc đã bị xóa"); **React** (tối đa 20 emoji khác nhau/tin; mỗi user 1 lần/emoji; bấm lần 2 để gỡ; yêu cầu `ADD_REACTIONS` để thêm emoji mới). | Must | MVP-Đồ án | MSG-01 | **AC1:** Given tác giả sửa, Then `MESSAGE_UPDATE` kèm `edited_at`. **AC2 (âm):** Given người khác (không phải tác giả) sửa, Then `40302` (kể cả có `MANAGE_MESSAGES`). **AC3:** Given tác giả hoặc người có `MANAGE_MESSAGES` xóa, Then `MESSAGE_DELETE` broadcast; **(âm)** người khác xóa → `40302`. **AC4:** Given tin đã có 20 emoji khác nhau, When thêm emoji thứ 21, Then `40905`. **AC5:** Given user react 👍 hai lần, Then reaction bị gỡ, số đếm đúng. **AC6:** Given reply tin đã bị xóa, Then hiển thị "Tin nhắn gốc đã bị xóa". |
| REQ-MSG-06 | Typing Indicator | Client gửi `TYPING_START` khi gõ (tối đa 1 lần/8 giây); server broadcast; tự hết hạn sau 10s. | Should | MVP-Đồ án | Gateway (5.4) | **AC1:** Given A đang gõ, Then B đang xem channel thấy "A đang nhập…" trong ≤ 500ms, tự ẩn sau 10s không có event mới. |
| REQ-MSG-07 ✏️ | Fanout có chọn lọc | Event channel chỉ gửi tới client đang subscribe guild/DM liên quan và có `VIEW_CHANNEL`. **Đồ án:** 1 process Gateway giữ map subscription trong bộ nhớ + Redis Pub/Sub (để chạy được ≥ 2 instance khi cần). **Production:** chỉ publish tới Gateway node đang giữ kết nối liên quan. | Must | MVP-Đồ án | ROLE-08 | **AC1:** Given guild 100 member, 20 online, trong đó 5 không có `VIEW_CHANNEL` ở C, When gửi tin vào C, Then chỉ 15 client nhận `MESSAGE_CREATE`. **AC2 (production):** Given 10,000 member với 200 online, Then chỉ node giữ kết nối của 200 client nhận event. |
| REQ-MSG-08 | Rate Limiting (sliding window) | Tối đa 5 tin/5 giây/user/channel và 30 tin/phút/user toàn hệ thống (sliding-window log trên Redis). | Must | MVP-Đồ án | MSG-01 | **AC1 (âm):** Given tin thứ 6 trong 5 giây, Then `40029` kèm header `Retry-After`; app giữ tin ở trạng thái `failed` và tự thử lại sau `Retry-After`. |
| REQ-MSG-09 🆕 | Idempotency & Nonce | Mọi request tạo mới (message, reaction, task, friend request, upload intent) mang `nonce` (tin nhắn) hoặc header `Idempotency-Key` (endpoint khác) do app sinh (UUIDv4/ULID). Server lưu khóa 24h theo (user, key); request lặp trả **cùng kết quả** thay vì tạo bản ghi mới. `MESSAGE_CREATE` echo lại `nonce` để app khớp tin `pending` với tin thật. | Must | MVP-Đồ án | — | **AC1:** Given outbox retry cùng `nonce` 3 lần do mất mạng, Then chỉ có 1 tin trong channel, cả 3 response trả cùng `message.id`. **AC2 (âm):** Given cùng `Idempotency-Key` nhưng body khác, Then `40907` (Idempotency key đã dùng cho request khác). **AC3:** Given app nhận `MESSAGE_CREATE` có `nonce` trùng tin `pending`, Then thay tin pending bằng tin thật, không hiện 2 bản. |
| REQ-MSG-10 🆕 | Ghim tin nhắn (Pinned) | Người có `MANAGE_MESSAGES` ghim/bỏ ghim; tối đa 50 tin ghim/channel (cả DM: mọi thành viên DM được ghim). Danh sách ghim xem ở drawer phải. Ghim sinh system message. (Chuyển từ REQ-BKLG-02.) | Should | MVP-Đồ án | MSG-11 | **AC1:** Given ghim tin, Then `CHANNEL_PINS_UPDATE` broadcast và tin xuất hiện trong danh sách ghim. **AC2 (âm):** Given đã có 50 tin ghim, Then `40904`. **AC3 (âm):** Given không có `MANAGE_MESSAGES`, Then `40302`. |
| REQ-MSG-11 🆕 | System Messages | Tin nhắn hệ thống có `type` riêng (Mục 8.2): thành viên tham gia, ghim tin, cuộc gọi nhỡ/kết thúc (DM), đổi tên/ảnh Group DM, thêm/bớt thành viên Group DM, Focus Session bắt đầu/kết thúc (voice channel có kênh chat liên kết), nhắc deadline. Không sửa được; xóa được bởi `MANAGE_MESSAGES`. Guild có `system_channel_id` (mặc định kênh text đầu tiên). | Must | MVP-Đồ án | MSG-01 | **AC1:** Given cuộc gọi nhỡ, Then cả 2 phía thấy system message "Cuộc gọi nhỡ" kèm nút "Gọi lại". **AC2 (âm):** Given user sửa system message, Then `40302`. |


### 4.5. Voice/Video — WebRTC/SFU (`REQ-VC-*`)

| REQ-ID | Tên chức năng | Mô tả chi tiết | Priority | Release | Depends-on | Acceptance Criteria |
|---|---|---|---|---|---|---|
| REQ-VC-01 ✏️ | Kết nối Voice Channel qua LiveKit | Khi user bấm Join: app gửi `VOICE_STATE_UPDATE` qua Gateway → backend kiểm tra `CONNECT` (và `user_limit`) → trả **LiveKit access token** (TTL 10 phút, grant đúng room = `voice:<channel_id>`, `canPublish` theo quyền `SPEAK`/`VIDEO`) + URL LiveKit. App kết nối bằng `@livekit/react-native`; **signaling media do LiveKit đảm nhiệm** (không tự xây Voice Gateway riêng). Voice state (ai đang trong kênh, mute/deaf, camera) broadcast qua Gateway chính (`VOICE_STATE_UPDATE`). Một user chỉ ở 1 voice channel/cuộc gọi tại một thời điểm trên toàn tài khoản. | Must | MVP-Đồ án | ROLE-08, VC-08 | **AC1:** Given user có `CONNECT`, When bấm Join, Then audio 2 chiều hoạt động trong ≤ 3 giây (P95) trên 4G/wifi. **AC2 (âm):** Given không có `CONNECT`, Then `40302`, không cấp token. **AC3:** Given user đang ở voice A join voice B, Then tự rời A trước khi vào B. **AC4:** Given khóa màn hình khi đang trong voice, Then audio vẫn tiếp tục (iOS background audio, Android foreground service kèm notification "Đang trong phòng thoại"). |
| REQ-VC-02 | Speaking indicator (VAD) | Dùng active speaker detection của LiveKit để hiển thị viền sáng quanh avatar người đang nói. | Should | MVP-Đồ án | VC-01 | **AC1:** Given user bắt đầu nói, Then avatar có viền sáng trong ≤ 300ms trên app của người khác. |
| REQ-VC-03 ✏️ | Screen Share | **Android: v1.x** — chia sẻ toàn màn hình qua MediaProjection + foreground service, tối đa 720p30 (tiết kiệm pin/nhiệt), không kèm âm thanh hệ thống. **iOS: Backlog** — cần Broadcast Upload Extension (DEC-11). Cả 2 nền tảng **xem** được screen share từ v1.x. Yêu cầu quyền `VIDEO`. | Could | v1.x | VC-01 | **AC1:** Given user Android bắt đầu share, Then người xem (iOS/Android) thấy luồng trong vùng Stage với độ trễ ≤ 1 giây. **AC2 (âm):** Given user iOS ở v1.x, Then nút "Chia sẻ màn hình" không hiển thị. **AC3 (âm):** Given không có quyền `VIDEO`, Then `40302`. |
| REQ-VC-04 | Mute/Deafen | Tự mute/deafen; Moderator server-mute/deafen (`MUTE_MEMBERS`/`DEAFEN_MEMBERS`) — thực thi ở tầng SFU (thu hồi quyền publish/subscribe audio qua LiveKit server API). | Must | MVP-Đồ án | VC-01 | **AC1:** Given moderator server-mute X, When X nói, Then audio của X không tới người khác dù X bấm unmute trên app; app X hiển thị "Bạn đã bị tắt tiếng bởi quản trị". **AC2 (âm):** Given member không có `MUTE_MEMBERS`, Then `40302`. |
| REQ-VC-05 | Thích ứng mạng | Bật simulcast + adaptive stream + dynacast của LiveKit; app hiển thị chỉ báo chất lượng mạng. Silent Room mặc định chỉ audio thấp bitrate (32 kbps Opus), video bật theo lựa chọn user. | Should | MVP-Đồ án | VC-01 | **AC1:** Given packet loss của 1 client > 5%, Then layer video gửi tới client đó giảm trong ≤ 2 giây mà không ảnh hưởng client khác. **AC2:** Given mạng chuyển wifi → 4G, Then kết nối tự phục hồi trong ≤ 5 giây không cần join lại. |
| REQ-VC-06 | Voice đa vùng (multi-region) | LiveKit triển khai theo khu vực, client được route tới node gần nhất theo RTT. | Could | Backlog | VC-01 | **AC1:** Given user ở APAC, Then kết nối SFU node APAC trừ khi RTT đo được cho thấy node khác tốt hơn. |
| REQ-VC-07 ✏️ | Giới hạn user trong voice channel | Người có `MANAGE_CHANNELS` đặt `user_limit` (0 = không giới hạn, 1–99). Người có `MOVE_MEMBERS` được vào kể cả khi đầy. | Could | v1.x | VC-01 | **AC1 (âm):** Given giới hạn 5 và đã đủ 5, When user thứ 6 join, Then `40051`. **AC2:** Given user có `MOVE_MEMBERS`, Then vào được. |
| REQ-VC-08 🆕 | TURN fallback | ICE cấu hình STUN + TURN/UDP + **TURN/TLS cổng 443** (TURN tích hợp của LiveKit). Kết nối tự thử lần lượt UDP → TCP → TURN/TLS. | Must | MVP-Đồ án | — | **AC1:** Given mạng chặn toàn bộ UDP, Then vẫn join voice thành công (qua TURN/TLS) trong ≤ 5 giây. **AC2:** Given 4G của 3 nhà mạng lớn tại Việt Nam, Then join thành công (test 9.3). |

### 4.6. Search & Notifications (`REQ-SEARCH-*` / `REQ-NOTIF-*`)

| REQ-ID | Tên chức năng | Mô tả chi tiết | Priority | Release | Depends-on | Acceptance Criteria |
|---|---|---|---|---|---|---|
| REQ-SEARCH-01 ✏️ | Tìm kiếm tin nhắn | Full-text search trong channel/guild/DM trên Meilisearch, **không phân biệt dấu tiếng Việt**, filter `from:`, `has:link/image/file`, `before:`/`after:`, `mentions:`. Index cập nhật bất đồng bộ qua job (BullMQ) khi tin được tạo/sửa/xóa. | Should | v1.x | SEARCH-03 | **AC1:** Given query `from:alice has:image`, Then chỉ trả tin của alice có ảnh. **AC2:** Given query "giai tich", Then khớp tin chứa "Giải tích". **AC3:** Given tin vừa bị xóa, Then không xuất hiện trong kết quả sau ≤ 5 giây. |
| REQ-SEARCH-02 ✏️ | Cursor-based Pagination | Tải lịch sử bằng `before`/`after`/`around` = message ID, `limit` 1–100 (mặc định 50). | Must | MVP-Đồ án | MSG-01 | **AC1:** Given `before=<id>&limit=50`, Then trả đúng 50 tin liền trước, không trùng/lệch dù có tin mới gửi song song. **AC2 (âm):** Given `limit=500`, Then `40001`. **AC3 (âm):** Given user không có `READ_MESSAGE_HISTORY`, Then trả danh sách rỗng. |
| REQ-SEARCH-03 🆕 | Lọc kết quả search theo quyền | Server tính danh sách channel user được xem (`VIEW_CHANNEL` + `READ_MESSAGE_HISTORY`) và áp làm filter bắt buộc trong truy vấn Meilisearch; DM chỉ gồm DM user là thành viên. Không bao giờ trả tin từ channel user không có quyền, kể cả số lượng kết quả. | Must | v1.x | ROLE-08 | **AC1 (âm):** Given kênh `#ban-quan-tri` user không xem được chứa từ "deadline", When search "deadline" cấp guild, Then không có kết quả nào từ kênh đó và `total` không tính các tin đó. **AC2 (âm):** Given user đã rời Group DM, Then tin của Group DM đó không còn trong kết quả. |
| REQ-NOTIF-01 ✏️ | Unread & Mention Badge | Unread theo channel/guild, mention badge riêng (đỏ, có số); read state (`last_read_message_id`, `mention_count`) lưu server, đồng bộ đa thiết bị qua `MESSAGE_ACK`. Badge icon app (iOS badge / Android launcher badge nếu hỗ trợ) = tổng mention + DM chưa đọc. | Must | MVP-Đồ án | MSG-01 | **AC1:** Given đọc hết channel trên điện thoại A, Then badge trên điện thoại B (nếu cùng tài khoản) biến mất trong ≤ 2 giây. **AC2:** Given mention mới, Then mention badge tăng 1 và badge icon app cập nhật. |
| REQ-NOTIF-02 ✏️ | Push Notification | FCM (Android) / APNs qua FCM (iOS) khi user không có kết nối Gateway foreground và nhận: mention, reply, DM, lời mời kết bạn, cuộc gọi, nhắc deadline, lời mời vào phòng học, Study Event sắp bắt đầu. Notification channel Android tách riêng: Tin nhắn, Cuộc gọi, Nhắc học tập. Bấm notification mở đúng màn hình. Tùy chọn "Ẩn nội dung xem trước". Xin quyền thông báo sau màn hình giải thích (Android 13+ `POST_NOTIFICATIONS`). | Should | MVP-Đồ án | NOTIF-01 | **AC1:** Given app ở nền và có mention, Then push hiển thị trong ≤ 5 giây. **AC2:** Given bật "Ẩn nội dung", Then push chỉ hiện "Bạn có tin nhắn mới". **AC3:** Given bấm push, Then app mở đúng channel và cuộn tới tin. **AC4 (âm):** Given user bị block người gửi, Then không nhận push. |
| REQ-NOTIF-03 | Mức thông báo theo guild/channel | All / Only Mentions / Nothing; mute tạm thời 15 phút/1h/8h/24h/vĩnh viễn. | Could | v1.x | NOTIF-02 | **AC1:** Given channel "Only Mentions", When có tin thường, Then không push nhưng unread badge thường vẫn cập nhật. |
| REQ-NOTIF-04 🆕 | Push đa thiết bị | Nếu user đang có Gateway foreground ở bất kỳ thiết bị nào (hoạt động trong 2 phút gần nhất) → **không** gửi push tới thiết bị khác cho tin nhắn thường/mention (vẫn gửi cho cuộc gọi). Khi đọc trên 1 thiết bị → gửi silent/data push để xóa notification đã hiển thị trên thiết bị khác. Mỗi thiết bị đăng ký push token riêng (`push_tokens`); token lỗi `NotRegistered` bị xóa tự động. | Should | MVP-Đồ án | NOTIF-02, AUTH-07 | **AC1:** Given user đang mở app trên điện thoại A, When có mention, Then điện thoại B không nhận push. **AC2:** Given đã đọc DM trên A, Then notification DM trên B bị xóa trong ≤ 10 giây (khi B có mạng). **AC3:** Given FCM trả token không hợp lệ, Then token bị xóa khỏi `push_tokens`. |

### 4.7. Kết bạn & Direct Message (`REQ-FRIEND-*` / `REQ-DM-*`)

> Luồng độc lập với Guild — truy cập qua icon 💬 trên guild rail.

| REQ-ID | Tên chức năng | Mô tả chi tiết | Priority | Release | Depends-on | Acceptance Criteria |
|---|---|---|---|---|---|---|
| REQ-FRIEND-01 ✏️ | Gửi lời mời kết bạn | Gửi friend request bằng **username** (DEC-03) hoặc từ profile của member guild chung. Chặn trùng request đang pending; giới hạn 50 request/ngày/user. Nếu B đã gửi request cho A, việc A gửi request cho B = chấp nhận. | Must | MVP-Đồ án | AUTH-13 | **AC1 (âm):** Given A đã gửi request tới B đang pending, When A gửi lại, Then `40902`. **AC2 (âm):** Given đã là bạn, Then `40908`. **AC3 (âm):** Given request thứ 51 trong ngày, Then `40029`. **AC4 (âm):** Given B đã block A, Then trả `40303` (thông điệp chung, không lộ việc bị block). **AC5:** Given B đã gửi request cho A, When A gửi request cho B, Then 2 người thành bạn. |
| REQ-FRIEND-02 | Chấp nhận / Từ chối / Hủy lời mời | B Accept hoặc Decline (xóa request, A không được thông báo); A Cancel khi còn pending. | Must | MVP-Đồ án | FRIEND-01 | **AC1:** Given B Accept, Then cả A và B thấy nhau trong Friend List (`RELATIONSHIP_ADD`), DM 1-1 giữa 2 người sẵn sàng (REQ-DM-03). |
| REQ-FRIEND-03 ✏️ | Chặn người dùng (Block) | A block B: hủy quan hệ bạn bè/request; B không gửi được DM/friend request tới A; không gửi push từ B tới A; trong DM 1-1 cũ, tin của B bị ẩn; trong **guild channel**, tin của B hiển thị thu gọn "1 tin nhắn bị chặn — Hiện" (A tự mở xem được). B **không** được biết mình bị block. | Must | MVP-Đồ án | FRIEND-01 | **AC1 (âm):** Given A block B, When B gửi DM cho A, Then `40303` với thông điệp chung "Không thể gửi tin nhắn cho người dùng này" (giống hệt lỗi do cài đặt quyền riêng tư ở REQ-DM-02). **AC2:** Given A xem channel guild có tin của B, Then tin hiển thị thu gọn. **AC3:** Given A unblock B, Then B gửi DM được (nếu thỏa REQ-DM-02). |
| REQ-FRIEND-04 | Danh sách bạn bè & trạng thái | Friend List tách tab Online / Tất cả / Đang chờ / Đã chặn; online (Online/Idle/DND) trước, tìm nhanh theo tên. | Should | MVP-Đồ án | FRIEND-02 | **AC1:** Given 200 bạn bè, When mở Friends, Then danh sách hiển thị trong ≤ 300ms từ cache cục bộ, cập nhật real-time khi presence đổi. |
| REQ-DM-01 | Nhắn tin 1-1 (DM) | Text/file/reaction/reply/pin trong DM 1-1, dùng lại toàn bộ pipeline của REQ-MSG-*, không dùng RBAC guild. | Must | MVP-Đồ án | MSG-01, DM-03 | **AC1:** Given A và B là bạn, When A gửi DM, Then B nhận `MESSAGE_CREATE` với độ trễ như REQ-MSG-01. |
| REQ-DM-02 ✏️ | DM với người chưa kết bạn chung server | Cho phép DM tới user chưa kết bạn nhưng chung ≥ 1 guild, trừ khi người nhận tắt "Cho phép tin nhắn từ thành viên server" (mặc định bật; có thể tắt theo từng guild). | Should | MVP-Đồ án | DM-01 | **AC1 (âm):** Given B tắt DM từ thành viên server, When A (chung guild, chưa là bạn) gửi DM, Then `40303` (cùng mã & thông điệp với trường hợp bị block). **AC2 (âm):** Given A và B không chung guild, không là bạn, Then `40303`. |
| REQ-DM-03 🆕 | DM 1-1 duy nhất cho mỗi cặp | Mỗi cặp user chỉ có **một** DM 1-1 (unique theo cặp id đã sắp xếp). Mở DM với người đã có DM → trả DM cũ. DM bị "đóng" (ẩn khỏi danh sách) theo từng phía, mở lại khi có tin mới. | Must | MVP-Đồ án | — | **AC1:** Given A và B đã có DM, When A (hoặc B) mở DM với nhau lần nữa, Then trả đúng `dm_channel_id` cũ. **AC2:** Given 2 request tạo DM đồng thời, Then chỉ 1 DM được tạo (ràng buộc unique ở CSDL). |

### 4.8. Group DM & Gọi thoại/video (`REQ-GDM-*` / `REQ-DMVC-*`)

| REQ-ID | Tên chức năng | Mô tả chi tiết | Priority | Release | Depends-on | Acceptance Criteria |
|---|---|---|---|---|---|---|
| REQ-GDM-01 | Tạo nhóm nhỏ giữa bạn bè | Tạo Group DM bằng cách chọn 2–9 bạn bè (tối đa 10 người gồm người tạo). Không có role, không nhiều channel, không invite công khai. | Must | MVP-Đồ án | FRIEND-02 | **AC1:** Given chọn 3 bạn bè, When "Tạo nhóm", Then tạo `dm_channel` `type=GROUP_DM` với đúng 4 thành viên, không sinh Guild/Role/Category. **AC2 (âm):** Given chọn người không phải bạn bè, Then `40302`. |
| REQ-GDM-02 ✏️ | Quản lý thành viên Group DM | Mọi thành viên thêm được bạn bè của mình vào nhóm, hoặc tự rời. Người tạo (owner nhóm) đổi tên/ảnh và gỡ thành viên; owner rời nhóm → quyền owner chuyển cho thành viên tham gia sớm nhất còn lại. Mọi thay đổi sinh system message. | Should | MVP-Đồ án | GDM-01, MSG-11 | **AC1:** Given nhóm 5 người, When 1 người rời, Then nhóm còn 4 người hoạt động bình thường. **AC2:** Given chỉ còn 1 người, Then nhóm chuyển trạng thái ẩn (giữ lịch sử) tới khi có người được thêm lại. **AC3 (âm):** Given thành viên thường gỡ người khác, Then `40302`. |
| REQ-GDM-03 ✏️ | Giới hạn quy mô | Tối đa 10 thành viên/Group DM. Cuộc gọi Group DM dùng **cùng LiveKit SFU** (room `call:<call_id>`), không dùng mesh. | Must | MVP-Đồ án | GDM-01 | **AC1 (âm):** Given nhóm đủ 10, When thêm người thứ 11, Then `40052`. |
| REQ-DMVC-01 ✏️ | Gọi thoại/video 1-1 (trong app) | Từ DM 1-1, bắt đầu cuộc gọi thoại hoặc video. Người nhận: nếu app foreground → màn hình cuộc gọi đến toàn màn hình + nhạc chuông; nếu app ở nền → **push ưu tiên cao** (FCM high priority / APNs time-sensitive) mở màn hình cuộc gọi khi bấm. Chỉ gọi được khi 2 người là bạn bè hoặc thỏa REQ-DM-02. (Đánh thức khi app bị kill: REQ-DMVC-04.) | Must | MVP-Đồ án | DM-01, VC-01 | **AC1:** Given B mở app, When A gọi, Then B thấy màn hình cuộc gọi đến trong ≤ 2 giây. **AC2:** Given app B ở nền, Then B nhận push cuộc gọi trong ≤ 5 giây. **AC3:** Given B chấp nhận, Then kết nối media trong ≤ 3 giây (như REQ-VC-01). **AC4 (âm):** Given B đang trong cuộc gọi khác, Then A thấy "Người nhận đang bận", sinh system message cuộc gọi nhỡ cho B. **AC5 (âm):** Given B đã block A, Then `40303`. |
| REQ-DMVC-02 | Gọi nhóm trong Group DM | Thành viên bất kỳ bắt đầu cuộc gọi; người khác thấy banner "Đang trong cuộc gọi" và tham gia/rời tùy ý. | Should | v1.x | GDM-03, DMVC-01 | **AC1:** Given cuộc gọi đang có 3/5 thành viên, When thành viên thứ 4 bấm Join, Then vào cùng room trong ≤ 3 giây, không tạo cuộc gọi mới. |
| REQ-DMVC-03 | Từ chối / Nhỡ cuộc gọi | Người nhận Decline, hoặc không phản hồi 45 giây → Missed; ghi system message trong DM. | Must | MVP-Đồ án | DMVC-01, MSG-11 | **AC1:** Given không ai bắt máy trong 45s, Then cuộc gọi tự kết thúc, cả 2 phía thấy "Cuộc gọi nhỡ" + nút "Gọi lại". **AC2:** Given B Decline, Then A thấy "Cuộc gọi bị từ chối" ngay lập tức. |
| REQ-DMVC-04 🆕 | Cuộc gọi đến khi app bị kill (UI gọi gốc) | **iOS:** PushKit VoIP push → `react-native-voip-push-notification` → báo CallKit ngay (qua `react-native-callkeep`) → màn hình gọi gốc cả khi khóa máy. **Android:** FCM high-priority data message → `react-native-callkeep` (ConnectionService self-managed) + full-screen intent. Trả lời/từ chối từ UI gốc đồng bộ về server. Cuộc gọi hiển thị trong lịch sử cuộc gọi của hệ thống (iOS). | Should | v1.x | DMVC-01 | **AC1:** Given app B bị kill, When A gọi, Then B thấy UI cuộc gọi gốc trong ≤ 3 giây. **AC2:** Given A hủy trước khi B trả lời, Then UI gọi trên máy B đóng trong ≤ 3 giây (VoIP push/FCM "cancel"). **AC3 (âm):** Given VoIP push tới khi cuộc gọi đã kết thúc, Then app vẫn báo CallKit rồi kết thúc ngay (tuân thủ quy tắc PushKit). |


### 4.9. Study/Focus Mode — Tính năng định vị cho Sinh viên (`REQ-STUDY-*`)

> Xây trên hạ tầng Guild/Channel/Voice/Messaging/Notification sẵn có, gói trong **module Study** của backend (không tách service riêng ở đồ án). Job định kỳ/trì hoãn (nhắc deadline, đóng session treo, cập nhật streak) chạy trên Scheduler (BullMQ). Đây là phần tạo câu chuyện sản phẩm riêng thay vì "Discord clone thuần kỹ thuật".

| REQ-ID | Tên chức năng | Mô tả chi tiết | Priority | Release | Depends-on | Acceptance Criteria |
|---|---|---|---|---|---|---|
| REQ-STUDY-01 ✏️ | Focus Session đồng bộ cho Voice Channel (Pomodoro) | Mỗi voice channel có tối đa **1** Focus Session đang chạy. Cấu hình: preset 25/5, 50/10 hoặc tùy chỉnh (focus 5–120 phút, nghỉ ngắn 1–30 phút, nghỉ dài 5–60 phút sau mỗi 2–8 chu kỳ — tùy chọn, mặc định tắt). Server phát `FOCUS_SESSION_UPDATE` chứa `server_started_at`, cấu hình chu kỳ, trạng thái pause (`paused_at`, `total_paused_ms`) và `server_time` để app tính độ lệch đồng hồ. App tự suy ra pha hiện tại và thời gian còn lại từ dữ liệu này (không đếm theo đồng hồ máy riêng lẻ). Khi chuyển pha: âm báo nhẹ + rung (foreground) hoặc local notification (nền/khóa màn hình, 7.3.6). Người vào muộn thấy đúng pha hiện tại. Session tự kết thúc khi bị dừng hoặc khi voice channel trống liên tục 5 phút. Quyền điều khiển theo REQ-STUDY-11; cách tính giờ theo REQ-STUDY-15. | Must | MVP-Đồ án | VC-01, STUDY-11, STUDY-15 | **AC1:** Given 4 người trong voice channel, When 1 người Start 25/5, Then đồng hồ hiển thị đồng nhất trên cả 4 app (sai lệch < 1 giây, kể cả khi đồng hồ một máy lệch 2 phút so với giờ chuẩn). **AC2 (âm):** Given channel đã có session đang chạy, When Start lần nữa, Then `40903`. **AC3:** Given người thứ 5 vào ở phút 12 của pha focus, Then thấy "Focus — còn 13:00". **AC4:** Given khóa màn hình, When tới mốc chuyển pha, Then nhận local notification "Hết giờ học — nghỉ 5 phút" với sai lệch ≤ 60 giây. **AC5:** Given Pause lúc còn 10:00 rồi Resume sau 3 phút, Then mọi app hiển thị còn 10:00. **AC6 (âm):** Given cấu hình focus 200 phút, Then `40001`. **AC7:** Given mọi người rời kênh 5 phút, Then session tự kết thúc, `FOCUS_SESSION_END` broadcast. |
| REQ-STUDY-02 ✏️ | Silent Co-working Room | Voice channel `voice_mode='silent'`: khi join, **mic tự tắt ở phía app** (không phải server-mute — user tự bật lại được ngay) và camera mặc định tắt (user tự bật để tạo cảm giác "cùng học"). Hiển thị nhãn "Im lặng" trên channel list. Chỉ người có `MANAGE_CHANNELS` đổi `voice_mode`. (Nhạc nền chung đã tách thành REQ-STUDY-14 — DEC-09.) | Should | MVP-Đồ án | VC-01, SRV-02 | **AC1:** Given user join Silent Room, Then mic ở trạng thái muted, camera tắt. **AC2:** Given user bấm bật mic, Then mic bật ngay, không cần quyền đặc biệt. **AC3:** Given join voice channel `normal`, Then mic bật theo cài đặt mặc định của user (hành vi thường). |
| REQ-STUDY-03 ✏️ | Study Streak cá nhân | Streak = số **local day** liên tiếp có tổng focused minutes ≥ 15 (cộng dồn mọi Focus Session nhóm + Solo trong ngày, theo REQ-STUDY-15). Tính theo `user_settings.timezone`. Streak tăng ngay khi đạt ngưỡng trong ngày. Nếu ngày hôm qua (local) không đạt ngưỡng, streak hiển thị 0 từ 00:00 hôm nay. Lưu `current_streak_days`, `longest_streak_days`. Đổi timezone tối đa 1 lần/24h. Hiển thị trên profile (biểu tượng lửa) và tab Học. | Should | MVP-Đồ án | STUDY-15 | **AC1:** Given hôm qua 20 phút, hôm nay 10 + 8 phút (2 phiên), Then streak = 2 sau phiên thứ hai. **AC2:** Given hôm qua chỉ 10 phút, Then hôm nay streak hiển thị 0 cho tới khi đạt 15 phút. **AC3:** Given phiên focus kéo dài qua 00:00 local, Then phút trước 00:00 tính cho ngày cũ, phút sau tính cho ngày mới. **AC4 (âm):** Given user đổi timezone lần 2 trong 24h, Then `40029`. |
| REQ-STUDY-04 ✏️ | Study Leaderboard theo guild | Bảng xếp hạng tổng focused minutes của member theo tuần/tháng. **Mặc định TẮT** (DEC-05); Owner/`MANAGE_GUILD` bật. User opt-out cá nhân: dữ liệu vẫn ghi nhận nhưng không hiện tên. Chỉ tính phút học của Focus Session trong chính guild đó. | Could | v1.x | STUDY-15 | **AC1:** Given guild vừa tạo, Then leaderboard tắt. **AC2:** Given leaderboard bật và user chọn "Ẩn tôi khỏi bảng xếp hạng", Then user không xuất hiện trên leaderboard nhưng thống kê cá nhân vẫn đủ. **AC3 (âm):** Given leaderboard tắt, When gọi API leaderboard, Then `40302`. |
| REQ-STUDY-05 ✏️ | Template "Study Server" | Khi tạo guild chọn template `study_server`: sinh category **"THÔNG TIN"** gồm `thong-bao` (overwrite `@everyone` deny `SEND_MESSAGES`), `tai-lieu`, `hoi-dap`; category **"PHÒNG HỌC"** gồm voice "Phòng học im lặng" (`silent`) và voice "Phòng thảo luận nhóm" (`normal`); role "Trưởng nhóm" có `MANAGE_STUDY_ROOMS`, `MANAGE_TASKS`, `MANAGE_EVENTS`, `MANAGE_MESSAGES` gán cho Owner. Mọi thứ chỉnh sửa được sau đó như guild thường. | Should | MVP-Đồ án | SRV-01, STUDY-02 | **AC1:** Given chọn template Study Server, Then guild có đúng 2 category, 3 text channel, 2 voice channel (1 silent, 1 normal), role "Trưởng nhóm" như mô tả. **AC2 (âm):** Given member thường gửi tin vào `thong-bao`, Then `40302`. |
| REQ-STUDY-06 ✏️ | Task Board theo Channel | Người có `MANAGE_CHANNELS` bật Task Board cho text channel. Task: tiêu đề (≤ 200), mô tả (≤ 2000), 1 người phụ trách (tùy chọn, phải là member có `VIEW_CHANNEL` ở kênh), deadline (tùy chọn), trạng thái `todo`/`doing`/`done`. **Quyền:** tạo task cần `CREATE_TASKS` (mặc định có ở `@everyone`); đổi trạng thái: người tạo, người phụ trách hoặc `MANAGE_TASKS`; sửa/xóa: người tạo hoặc `MANAGE_TASKS`. **Nhắc hạn:** push trước 24h và 2h cho người phụ trách nếu chưa `done`; task không có người phụ trách chỉ nhắc **người tạo** (không nhắc cả kênh để tránh spam). Người phụ trách rời guild → task về "chưa gán", báo người tạo. Hiển thị ở drawer phải; realtime qua `TASK_CREATE/UPDATE/DELETE`. Tối đa 500 task chưa xong/kênh. | Should | MVP-Đồ án | SRV-02, ROLE-08, NOTIF-02 | **AC1:** Given task gán cho B, deadline 18:00, When 16:00 và task chưa `done`, Then B nhận push nhắc. **AC2:** Given task không gán ai, Then chỉ người tạo nhận nhắc. **AC3 (âm):** Given gán cho người không xem được kênh, Then `40016`. **AC4 (âm):** Given member không phải người tạo/phụ trách, không có `MANAGE_TASKS` đổi trạng thái, Then `40302`. **AC5:** Given B rời guild, Then task của B chuyển "chưa gán" và người tạo nhận thông báo. **AC6 (âm):** Given Task Board chưa bật ở kênh, When tạo task, Then `40302`. |
| REQ-STUDY-07 | Study Buddy Matching | User đăng ký "đang tìm bạn học" kèm chủ đề; hệ thống gợi ý guild công khai (Server Discovery — REQ-BKLG-09) hoặc user cùng chủ đề. | Could | Backlog | BKLG-09 | **AC1:** Given user đăng ký "Giải tích 2", When có guild công khai gắn tag đó, Then guild xuất hiện trong gợi ý, sắp theo số member hoạt động gần đây. |
| REQ-STUDY-08 ✏️ | Việc cần làm tổng hợp (Study Digest) | Tab "Việc cần làm" gom mọi task được gán cho user từ mọi guild (chỉ kênh user còn xem được), nhóm Quá hạn / Hôm nay / Tuần này / Sau đó / Không hạn, sắp deadline tăng dần; lọc theo guild; bấm task nhảy tới kênh gốc. Có cache cục bộ để xem offline. | Should | MVP-Đồ án | STUDY-06 | **AC1:** Given user có task ở 3 guild, Then cả 3 hiện chung 1 danh sách đúng thứ tự, có link tới kênh gốc. **AC2:** Given user mất quyền xem kênh chứa task, Then task biến mất khỏi danh sách. **AC3:** Given offline, Then vẫn thấy danh sách lần đồng bộ gần nhất. |
| REQ-STUDY-09 ✏️ | Chế độ Học tập (lọc thông báo theo khung giờ) | User đặt khung giờ lặp (vd 20:00–22:00 các ngày trong tuần, theo timezone của user). Trong khung giờ: chỉ push **mention trực tiếp, DM, cuộc gọi, nhắc deadline, lời mời vào phòng học**; tin nhắn thường của guild **không phải học tập** (theo DEC-10) không push, chỉ cập nhật badge. | Could | v1.x | NOTIF-03 | **AC1:** Given đang trong khung giờ và guild không phải học tập có tin thường, Then không push, badge vẫn cập nhật. **AC2:** Given có mention trực tiếp trong guild đó, Then vẫn push. **AC3:** Given guild tạo từ template `study_server`, Then tin thường vẫn theo cài đặt thông báo bình thường. |
| REQ-STUDY-10 🆕 | Solo Focus Timer | Tab Học cho phép chạy Pomodoro cá nhân không cần voice channel (cùng tùy chọn chu kỳ như REQ-STUDY-01), có Pause/Resume/Stop, tính vào streak và thống kê cá nhân (không tính vào leaderboard guild). Không chạy đồng thời với việc tham gia Focus Session nhóm. Tùy chọn hiển thị "Đang học một mình" cho bạn bè. | Should | MVP-Đồ án | STUDY-15 | **AC1:** Given chạy Solo 25/5 và khóa màn hình, Then nhận local notification chuyển pha, sau khi hoàn tất 25 phút focus thì thống kê +25. **AC2 (âm):** Given đang ở Focus Session nhóm, When Start Solo, Then `40903`. **AC3:** Given app bị kill giữa chừng rồi mở lại sau 10 phút, Then timer khôi phục đúng vị trí từ dữ liệu server. |
| REQ-STUDY-11 🆕 | Quyền điều khiển Focus Session | Thiết lập theo voice channel `focus_control`: `everyone` (mặc định) hoặc `managers`. **Start:** với `everyone` — bất kỳ ai đang trong kênh; với `managers` — chỉ `MANAGE_STUDY_ROOMS`. **Pause/Resume/Skip pha/Stop/Sửa chu kỳ:** người khởi tạo session hoặc người có `MANAGE_STUDY_ROOMS`. Người khởi tạo rời kênh → quyền điều khiển chuyển cho participant ở trong kênh lâu nhất. Mọi thao tác điều khiển sinh `FOCUS_SESSION_UPDATE` kèm tên người thao tác. | Must | MVP-Đồ án | VC-01, ROLE-08 | **AC1 (âm):** Given member thường (không phải người khởi tạo) bấm Stop, Then `40302`. **AC2:** Given người khởi tạo rời kênh, Then participant ở lâu nhất có nút điều khiển trong ≤ 2 giây. **AC3 (âm):** Given `focus_control='managers'`, member thường bấm Start, Then `40302`. **AC4:** Given moderator có `MANAGE_STUDY_ROOMS`, Then điều khiển được mọi session. |
| REQ-STUDY-12 🆕 | Scheduled Study Events | Người có `MANAGE_EVENTS` tạo sự kiện học: tiêu đề, voice channel, thời điểm bắt đầu, thời lượng, lặp hàng tuần theo các thứ đã chọn (tối đa 12 tuần), preset Focus tùy chọn. Member bấm "Quan tâm"; nhận push 15 phút trước giờ bắt đầu. Đến giờ: nếu bật "Tự bắt đầu Focus", server tự tạo Focus Session khi có ≥ 1 người trong kênh. Hiển thị trong guild và tab Học. | Should | v1.x | STUDY-01, NOTIF-02 | **AC1:** Given sự kiện 20:00 thứ 2-4-6, user "Quan tâm", Then user nhận push lúc 19:45 mỗi buổi. **AC2 (âm):** Given thời điểm bắt đầu ở quá khứ, Then `40001`. **AC3 (âm):** Given không có `MANAGE_EVENTS`, Then `40302`. **AC4:** Given "Tự bắt đầu Focus" và có 2 người trong kênh lúc 20:00, Then session bắt đầu với preset đã chọn. |
| REQ-STUDY-13 🆕 | Tổng quan hoạt động thành viên | Màn hình cho người có `MANAGE_STUDY_ROOMS`: mỗi member có lần hoạt động gần nhất trong guild (tin nhắn/vào voice), focused minutes 7 ngày trong guild, số task đang mở/quá hạn; lọc "không hoạt động ≥ 7 ngày". Member có thể ẩn số phút học (`user_settings.share_study_stats=false`) → hiển thị "Đã ẩn". | Could | v1.x | STUDY-06, STUDY-15 | **AC1:** Given member không hoạt động 8 ngày, When lọc "không hoạt động ≥ 7 ngày", Then member xuất hiện. **AC2:** Given member ẩn thống kê, Then cột phút học hiển thị "Đã ẩn". **AC3 (âm):** Given member thường mở màn hình, Then `40302`. |
| REQ-STUDY-14 | Nhạc nền dùng chung (lo-fi) | Silent Room có thể phát nhạc nền chung, đồng bộ qua SFU (một audio track do server phát), chỉ dùng nhạc royalty-free/CC0 do team host (DEC-09). Mỗi user tự chỉnh/tắt âm lượng nhạc. | Could | Backlog | STUDY-02 | **AC1:** Given nhạc đang phát, Then mọi người nghe cùng đoạn nhạc (lệch ≤ 1 giây). **AC2:** Given user tắt nhạc, Then chỉ phía user đó tắt. |
| REQ-STUDY-15 🆕 | Quy tắc tính giờ học (Focused minutes) | Theo DEC-06: **(1)** Chỉ cộng phút thuộc pha `focus` (không tính nghỉ, không tính lúc pause). **(2) Nhóm:** khoảng tham gia = từ lúc join tới lúc leave do server ghi nhận; trong lúc tham gia app gửi heartbeat Focus qua Gateway mỗi 60s; app giữ kết nối khi khóa màn hình nhờ audio session (iOS) / foreground service (Android). Nếu mất heartbeat > 3 phút, server đóng khoảng tham gia tại heartbeat cuối cùng. **(3) Solo:** server ghi start/pause/resume/stop; nếu app mất mạng lúc kết thúc, app gửi lại sự kiện stop khi có mạng, server chấp nhận nhưng **không vượt quá lịch pha tính theo đồng hồ server**; session không có stop sau 4 giờ tự đóng tại mốc 4 giờ. **(4)** Tổng focused minutes tối đa 16 giờ/local day. **(5)** Server là nguồn sự thật duy nhất; app chỉ hiển thị. | Must | MVP-Đồ án | VC-01 | **AC1:** Given user ở Focus 25/5 nhóm suốt 2 chu kỳ, khóa màn hình trên cả iOS và Android, Then ghi nhận 50 phút. **AC2 (âm):** Given user tắt mạng hoàn toàn 10 phút giữa pha focus, Then chỉ phút trước heartbeat cuối cùng được tính (không cộng 10 phút mất kết nối). **AC3:** Given Solo 25 phút, app mất mạng lúc kết thúc và gửi stop sau 1 giờ, Then ghi nhận 25 phút (không phải 85). **AC4 (âm):** Given app gửi stop với số phút lớn hơn lịch pha theo đồng hồ server, Then server dùng giá trị theo đồng hồ server. |

### 4.10. An toàn & Kiểm duyệt nền tảng (`REQ-SAFE-*`)

| REQ-ID | Tên chức năng | Mô tả chi tiết | Priority | Release | Depends-on | Acceptance Criteria |
|---|---|---|---|---|---|---|
| REQ-SAFE-01 🆕 | Báo cáo vi phạm (Report) | Nhấn giữ tin nhắn / mở profile / guild → "Báo cáo" với lý do (spam, quấy rối, nội dung nhạy cảm, lừa đảo, khác + mô tả). Báo cáo trong guild gửi tới moderator guild (người có `MANAGE_MESSAGES`) **và** hàng đợi của Platform Admin; lưu snapshot bằng chứng (nội dung tin + 5 tin trước/sau + metadata) để không mất khi tin bị xóa. Người bị báo cáo không biết ai báo cáo. Giới hạn 20 report/ngày/user. (Hiện thực hóa một phần REQ-BKLG-12.) | Should | MVP-Đồ án | MSG-01 | **AC1:** Given báo cáo 1 tin, Then report xuất hiện trong hàng đợi admin kèm snapshot, kể cả khi tác giả xóa tin sau đó. **AC2 (âm):** Given report thứ 21 trong ngày, Then `40029`. **AC3 (âm):** Given báo cáo tin trong kênh user không xem được, Then `40401`. |
| REQ-SAFE-02 🆕 | Công cụ Platform Admin trong app | Tài khoản có cờ `is_platform_admin` thấy mục "Quản trị" trong app: danh sách report, xem bằng chứng, hành động (bỏ qua, xóa tin, khóa tài khoản 1/7/30 ngày/vĩnh viễn, gỡ guild vi phạm). Mọi hành động ghi `platform_audit_logs`. Tài khoản bị khóa không đăng nhập được. | Could | v1.x | SAFE-01 | **AC1:** Given admin khóa tài khoản X 7 ngày, Then mọi phiên của X bị thu hồi, X đăng nhập nhận `40308` kèm thời hạn. **AC2 (âm):** Given user thường gọi API admin, Then `40401` (không lộ endpoint). |

---

## 5. ĐẶC TẢ GIAO DIỆN NGOÀI (EXTERNAL INTERFACE REQUIREMENTS)

### 5.1. Giao diện người dùng

Xem Mục 3 (điều hướng, sitemap, trạng thái màn hình) và file Figma (Phụ lục 11.4).

### 5.2. Quy ước REST API

| Hạng mục | Quy ước |
|---|---|
| Base URL | `https://api.nezfocus.app/api/v1` — version trong path; thay đổi phá vỡ tương thích phải lên `v2`, `v1` được giữ ≥ 3 tháng sau khi app bản mới lên store (vì user mobile cập nhật chậm). |
| Định dạng | JSON UTF-8, key `snake_case`. Snowflake ID **luôn là string**. Thời gian ISO 8601 UTC (`2026-09-29T13:00:00.000Z`). |
| Xác thực | Header `Authorization: Bearer <access_token>`. Endpoint công khai: đăng ký, đăng nhập, refresh, quên mật khẩu, xác thực email, preview invite. |
| Header client | `X-Client-Platform: ios|android`, `X-Client-Version: 1.2.0`, `Accept-Language: vi|en`. Server trả `42601` nếu phiên bản app thấp hơn phiên bản tối thiểu (buộc cập nhật). |
| Phân trang | Cursor: `before`, `after`, `around`, `limit` (1–100, mặc định 50). Không dùng offset. |
| Idempotency | Header `Idempotency-Key` cho mọi POST tạo mới (trừ message dùng `nonce` trong body) — REQ-MSG-09. |
| Rate limit | Header `X-RateLimit-Limit`, `X-RateLimit-Remaining`, `X-RateLimit-Reset`; khi vượt: HTTP 429 + `Retry-After` (giây) + mã `40029`. |
| Upload | Presigned URL (REQ-MSG-04): `POST /attachments` → `{ attachment_id, upload_url, expires_at }` → `PUT upload_url` → dùng `attachment_id` khi gửi tin. |

**Định dạng lỗi thống nhất:**

```json
{
  "code": 40001,
  "message": "Dữ liệu không hợp lệ",
  "errors": {
    "username": [{ "code": "INVALID_FORMAT", "message": "Chỉ dùng a-z, 0-9, dấu _ và ." }]
  },
  "request_id": "01J8Z6Q3K1V9W2",
  "retry_after": null
}
```

- `message` đã bản địa hóa theo `Accept-Language`; app vẫn nên ánh xạ `code` sang chuỗi của app.
- `request_id` được ghi trong log/trace (NFR-OBS-02) để truy vết khi user báo lỗi.

### 5.3. Danh mục endpoint chính (MVP-Đồ án + v1.x)

| Nhóm | Method & Path | Mô tả | REQ |
|---|---|---|---|
| Auth | `POST /auth/register` | Đăng ký | AUTH-01 |
| Auth | `POST /auth/verify-email` · `POST /auth/verify-email/resend` | Xác thực email / gửi lại | AUTH-01/13 |
| Auth | `POST /auth/login` · `POST /auth/login/mfa` | Đăng nhập / bước 2FA | AUTH-02/03 |
| Auth | `POST /auth/refresh` · `POST /auth/logout` | Rotation / đăng xuất | AUTH-04/07 |
| Auth | `POST /auth/password/forgot` · `POST /auth/password/reset` | Quên / đặt lại mật khẩu | AUTH-08 |
| Auth | `POST /auth/oauth/google` · `POST /auth/oauth/apple` | Đăng nhập bên thứ ba | AUTH-12 |
| Account | `GET/PATCH /users/@me` · `PATCH /users/@me/settings` | Profile, cài đặt (theme, timezone, privacy) | AUTH-06, STUDY-03 |
| Account | `GET /users/@me/sessions` · `DELETE /users/@me/sessions/{id}` | Quản lý phiên | AUTH-07 |
| Account | `POST /users/@me/email` · `DELETE /users/@me` · `POST /users/@me/data-export` | Đổi email / xóa tài khoản / xuất dữ liệu | AUTH-09/10, NFR-PRIV-04 |
| Account | `POST /users/@me/mfa/totp` · `POST /users/@me/mfa/recovery-codes` | 2FA | AUTH-03/11 |
| Devices | `PUT /users/@me/push-tokens` · `DELETE /users/@me/push-tokens/{token}` | Đăng ký/gỡ push token (FCM/APNs/VoIP) | NOTIF-02/04, DMVC-04 |
| Guild | `POST /guilds` · `GET/PATCH/DELETE /guilds/{id}` · `POST /guilds/{id}/transfer` | Tạo/sửa/xóa/chuyển quyền | SRV-01/05, STUDY-05 |
| Guild | `GET/POST /guilds/{id}/channels` · `PATCH /guilds/{id}/channels/positions` | Channel & sắp xếp | SRV-02 |
| Guild | `PATCH/DELETE /channels/{id}` · `PUT/DELETE /channels/{id}/permissions/{target_id}` | Sửa channel, overwrite | SRV-02, ROLE-03 |
| Invite | `POST /channels/{id}/invites` · `GET /invites/{code}` · `POST /invites/{code}/accept` · `DELETE /invites/{code}` | Invite | SRV-03/04/07 |
| Member | `GET /guilds/{id}/members` · `PATCH /guilds/{id}/members/{user_id}` · `DELETE /guilds/{id}/members/{user_id}` | Danh sách, nickname/role/timeout, kick | ROLE-04/05 |
| Role | `GET/POST /guilds/{id}/roles` · `PATCH/DELETE /guilds/{id}/roles/{role_id}` · `PATCH /guilds/{id}/roles/positions` | Role | ROLE-01/02 |
| Ban | `GET /guilds/{id}/bans` · `PUT/DELETE /guilds/{id}/bans/{user_id}` | Ban/unban | ROLE-05/07 |
| Audit | `GET /guilds/{id}/audit-logs` | Audit log | ROLE-06 |
| Message | `GET/POST /channels/{id}/messages` · `PATCH/DELETE /channels/{id}/messages/{mid}` | Lịch sử/gửi/sửa/xóa | MSG-01/05, SEARCH-02 |
| Message | `PUT/DELETE /channels/{id}/messages/{mid}/reactions/{emoji}/@me` | React | MSG-05 |
| Message | `GET /channels/{id}/pins` · `PUT/DELETE /channels/{id}/pins/{mid}` | Ghim | MSG-10 |
| Message | `POST /channels/{id}/typing` · `POST /channels/{id}/messages/{mid}/ack` | Typing / read state | MSG-06, NOTIF-01 |
| Upload | `POST /attachments` | Presigned upload | MSG-04 |
| Search | `GET /guilds/{id}/messages/search` · `GET /channels/{id}/messages/search` | Tìm kiếm | SEARCH-01/03 |
| Friend | `GET /users/@me/relationships` · `POST /users/@me/relationships` · `PUT/DELETE /users/@me/relationships/{user_id}` | Kết bạn / chấp nhận / hủy / block | FRIEND-01..04 |
| DM | `POST /users/@me/channels` (1-1 hoặc group) · `PUT/DELETE /channels/{id}/recipients/{user_id}` | DM & Group DM | DM-01/03, GDM-01/02 |
| Call | `POST /channels/{id}/calls` · `POST /calls/{id}/accept` · `POST /calls/{id}/decline` · `POST /calls/{id}/end` | Cuộc gọi DM | DMVC-01..04 |
| Voice | `POST /voice/token` (kèm `channel_id` hoặc `call_id`) | Cấp LiveKit token | VC-01, GDM-03 |
| Study | `POST /channels/{id}/focus-session` · `PATCH /focus-sessions/{id}` (pause/resume/skip/stop/config) | Focus Session nhóm | STUDY-01/11 |
| Study | `POST /users/@me/solo-focus` · `PATCH /solo-focus/{id}` | Solo Focus | STUDY-10 |
| Study | `GET /users/@me/study-stats` · `GET /guilds/{id}/leaderboard` · `GET /guilds/{id}/member-activity` | Thống kê, leaderboard, hoạt động | STUDY-03/04/13 |
| Task | `PATCH /channels/{id}/task-board` · `GET/POST /channels/{id}/tasks` · `PATCH/DELETE /tasks/{id}` · `GET /users/@me/tasks` | Task Board & Digest | STUDY-06/08 |
| Event | `GET/POST /guilds/{id}/study-events` · `PATCH/DELETE /study-events/{id}` · `PUT/DELETE /study-events/{id}/interest` | Study Events | STUDY-12 |
| Safety | `POST /reports` · `GET /admin/reports` · `POST /admin/reports/{id}/actions` | Report & admin | SAFE-01/02 |

> Đặc tả chi tiết request/response từng endpoint được sinh tự động bằng OpenAPI 3.1 từ code NestJS (`@nestjs/swagger`) và là một phần của Definition of Done (Mục 9.6).

### 5.4. Giao thức Gateway (WebSocket)

**Kết nối:** `wss://gateway.nezfocus.app/?v=1&encoding=json`. Mỗi frame là JSON:

```json
{ "op": 0, "d": { ... }, "s": 42, "t": "MESSAGE_CREATE" }
```

`op` = opcode · `d` = payload · `s` = sequence number (chỉ với op 0, tăng dần theo session) · `t` = tên event (chỉ với op 0).

**Opcodes:**

| op | Tên | Hướng | Mô tả |
|---|---|---|---|
| 0 | DISPATCH | Server → Client | Sự kiện nghiệp vụ, có `s` và `t`. |
| 1 | HEARTBEAT | 2 chiều | Client gửi `d = last_sequence` theo chu kỳ; server có thể yêu cầu heartbeat ngay. |
| 2 | IDENTIFY | Client → Server | `{ token, device: { platform, app_version, device_id }, presence }` — mở session mới. |
| 3 | PRESENCE_UPDATE | Client → Server | Đổi presence thủ công (REQ-AUTH-05). |
| 4 | VOICE_STATE_UPDATE | Client → Server | Join/leave/đổi voice channel, self mute/deaf/camera. |
| 5 | FOCUS_HEARTBEAT | Client → Server | Xác nhận đang tham gia Focus Session mỗi 60s (REQ-STUDY-15). |
| 6 | RESUME | Client → Server | `{ token, session_id, seq }` — nối lại session, nhận lại event bị lỡ. |
| 7 | RECONNECT | Server → Client | Yêu cầu client kết nối lại (vd server deploy) rồi RESUME. |
| 8 | SUBSCRIBE | Client → Server | Đăng ký nhận event chi tiết của guild đang mở (typing, presence member list) — tránh gửi thừa cho guild không mở. |
| 9 | INVALID_SESSION | Server → Client | `d = resumable (bool)`; nếu `false` client phải IDENTIFY lại và tải lại trạng thái qua REST. |
| 10 | HELLO | Server → Client | Gửi ngay sau khi kết nối: `{ heartbeat_interval: 41250 }`. |
| 11 | HEARTBEAT_ACK | Server → Client | Xác nhận heartbeat. |

**Vòng đời:**
1. Kết nối → nhận `HELLO` → gửi heartbeat đầu tiên sau `heartbeat_interval × random(0..1)` (tránh dồn tải khi server khởi động lại), sau đó theo chu kỳ.
2. Gửi `IDENTIFY` (hoặc `RESUME` nếu có `session_id`) → nhận `READY` (user, guild tóm tắt, DM, read states, relationships, session_id) hoặc `RESUMED`.
3. Không nhận `HEARTBEAT_ACK` sau 2 chu kỳ → coi kết nối chết, đóng và kết nối lại với backoff lũy thừa (1s, 2s, 4s … tối đa 30s, có jitter).
4. **Resume buffer:** server giữ tối đa **1000 event hoặc 60 giây** gần nhất mỗi session (Redis). RESUME trong giới hạn → phát lại event có `s` > `seq`; quá giới hạn → `INVALID_SESSION(false)`.
5. App chuyển nền: xem 7.3.3.

**Danh sách event DISPATCH (MVP-Đồ án):**

| Nhóm | Event |
|---|---|
| Phiên | `READY`, `RESUMED`, `USER_UPDATE`, `USER_SETTINGS_UPDATE`, `SESSIONS_REPLACE` |
| Guild | `GUILD_CREATE`, `GUILD_UPDATE`, `GUILD_DELETE`, `GUILD_MEMBER_ADD`, `GUILD_MEMBER_UPDATE`, `GUILD_MEMBER_REMOVE`, `GUILD_ROLE_CREATE/UPDATE/DELETE`, `GUILD_BAN_ADD/REMOVE` |
| Channel | `CHANNEL_CREATE`, `CHANNEL_UPDATE`, `CHANNEL_DELETE`, `CHANNEL_PINS_UPDATE` |
| Message | `MESSAGE_CREATE` (echo `nonce`), `MESSAGE_UPDATE`, `MESSAGE_DELETE`, `MESSAGE_DELETE_BULK`, `MESSAGE_REACTION_ADD/REMOVE`, `MESSAGE_ACK`, `TYPING_START` |
| Presence/Social | `PRESENCE_UPDATE`, `RELATIONSHIP_ADD`, `RELATIONSHIP_REMOVE` |
| Voice/Call | `VOICE_STATE_UPDATE`, `CALL_CREATE`, `CALL_UPDATE`, `CALL_DELETE` |
| Study | `FOCUS_SESSION_START`, `FOCUS_SESSION_UPDATE`, `FOCUS_SESSION_END`, `SOLO_FOCUS_UPDATE`, `STREAK_UPDATE`, `TASK_CREATE`, `TASK_UPDATE`, `TASK_DELETE`, `STUDY_EVENT_CREATE/UPDATE/DELETE` (v1.x) |
| Thông báo | `NOTIFICATION_CREATE` (inbox tab Thông báo) |

**Close codes:** Phụ lục 11.2.

### 5.5. Giao diện Voice/Video (LiveKit, TURN)

| Hạng mục | Đặc tả |
|---|---|
| Cấp token | Backend ký LiveKit access token (JWT) bằng API secret; grant: `roomJoin`, `room = voice:<channel_id>` hoặc `call:<call_id>`, `canPublish` (theo `SPEAK`/`VIDEO`), `canSubscribe = true`, `canPublishData = false`; TTL 10 phút (chỉ cần hợp lệ lúc kết nối). `identity = user_id`. |
| Thực thi quyền | Server-mute/kick khỏi voice: backend gọi LiveKit Server API (`MutePublishedTrack`, `UpdateParticipant`, `RemoveParticipant`). |
| Webhook | LiveKit webhook (`participant_joined/left`, `room_finished`) gửi về backend để đối chiếu voice state và đóng khoảng tham gia Focus (REQ-STUDY-15). Webhook được xác thực chữ ký. |
| ICE/TURN | STUN + TURN/UDP 3478 + TURN/TLS 443 (REQ-VC-08). |
| Codec | Audio Opus (32 kbps Silent Room, 64 kbps mặc định); video VP8 simulcast 3 layer (180p/360p/720p). |
| Bảo mật media | DTLS-SRTP (mặc định WebRTC). |

### 5.6. Giao diện Push Notification

| Kênh | Dùng cho | Đặc tả |
|---|---|---|
| FCM (Android) | Tin nhắn, mention, nhắc học, cuộc gọi | Tin nhắn: notification message (hệ thống tự hiển thị khi app bị kill) với `android_channel_id`; cuộc gọi & đồng bộ xóa notification: data message priority `high`. |
| APNs qua FCM (iOS) | Tin nhắn, mention, nhắc học | `apns-push-type: alert`; nhắc học & cuộc gọi MVP dùng `interruption-level: time-sensitive`; `thread-id` = channel_id để nhóm notification; `mutable-content: 1` cho Notification Service Extension (giải mã/ẩn nội dung, v1.x). |
| APNs VoIP (PushKit) | Cuộc gọi đến (REQ-DMVC-04, v1.x) | Gửi trực tiếp từ backend tới APNs bằng VoIP token riêng (`push_tokens.kind='apns_voip'`), topic `<bundle_id>.voip`, payload tối thiểu `{ call_id, caller_name, has_video }`. |
| Payload chung | — | `{ type, id, guild_id?, channel_id?, message_id?, title, body }`; không chứa nội dung khi user bật "Ẩn nội dung xem trước". Tối đa 4KB. |

### 5.7. Giao diện hệ thống khác

| Hệ thống | Giao tiếp | Ghi chú |
|---|---|---|
| Email provider | API/SMTP qua job BullMQ, retry 3 lần | Template song ngữ vi/en; link dùng domain deep link. |
| Object storage | S3 API, presigned URL | Bucket private; ảnh đọc qua URL ký hoặc CDN có token. |
| Meilisearch | HTTP API nội bộ | Chỉ backend truy cập; không lộ ra app. |
| ClamAV | TCP `clamd` nội bộ | Quét trước khi đánh dấu attachment `clean`. |
| Google / Apple OIDC | Xác minh `id_token` phía server | REQ-AUTH-12. |
| Deep link | `apple-app-site-association`, `.well-known/assetlinks.json` trên `nezfocus.app` | REQ-SRV-07. |

---

## 6. YÊU CẦU PHI CHỨC NĂNG (NON-FUNCTIONAL REQUIREMENTS)

> Cột **Mục tiêu đồ án** là tiêu chí nghiệm thu của MVP-Đồ án (đo trên môi trường demo/pilot). Cột **Production** là mục tiêu định hướng khi phát hành rộng — **không** dùng để nghiệm thu đồ án. Điều kiện đo mặc định: server đặt tại region Singapore hoặc Việt Nam, client ở Việt Nam trên wifi ≥ 20 Mbps hoặc 4G, thiết bị tham chiếu tầm trung (Mục 9.3).

### 6.1. Hiệu năng (NFR-PERF)

| ID | Yêu cầu | Mục tiêu đồ án | Production |
|---|---|---|---|
| NFR-PERF-01 | Độ trễ từ lúc gửi tin tới lúc client khác trong channel nhận `MESSAGE_CREATE` | P95 < 300ms | P95 < 100ms cùng region |
| NFR-PERF-02 | Độ trễ audio end-to-end trong voice | P95 < 250ms | P95 < 150ms, P99 < 250ms |
| NFR-PERF-03 | Mở channel: hiển thị tin từ cache cục bộ / tải xong 50 tin mới nhất từ server | < 100ms / P95 < 500ms | < 100ms / P95 < 300ms |
| NFR-PERF-04 | REST API (không tính upload) | P95 < 300ms ở 50 req/s tổng | P95 < 200ms ở 1,000 req/s/instance |
| NFR-PERF-05 | Join voice tới lúc có audio 2 chiều | P95 < 3s | P95 < 2s |
| NFR-PERF-06 | Sai lệch đồng hồ Focus giữa các client trong cùng session | < 1s | < 500ms |

### 6.2. Khả năng mở rộng (NFR-SCALE)

| ID | Yêu cầu | Mục tiêu đồ án | Production |
|---|---|---|---|
| NFR-SCALE-01 | Kết nối WebSocket đồng thời | 500 trên 1 instance (load test) | ≥ 100,000 toàn cluster, 50,000/node, scale ngang qua Redis/NATS |
| NFR-SCALE-02 | Message store | PostgreSQL, index `(channel_id, id DESC)`, đủ cho ≥ 1 triệu tin với NFR-PERF-03 | ScyllaDB partition `(channel_id, bucket)`. **Lưu ý:** bucket theo thời gian giới hạn *kích thước* partition, **không** giải quyết hot-write của một channel cực nóng; hot-write xử lý bằng rate limit + request coalescing + cache tin mới nhất (8.3). |
| NFR-SCALE-03 | Voice | 1 LiveKit node, ≥ 10 room đồng thời × 10 người audio | LiveKit cluster auto-scale theo số room, chọn node còn nhiều capacity nhất |
| NFR-SCALE-04 | Kiến trúc | Module không truy cập bảng của module khác trực tiếp (qua service interface) để tách được service | Microservices tách theo module |

### 6.3. Bảo mật (NFR-SEC)

| ID | Yêu cầu | Mục tiêu đồ án | Production |
|---|---|---|---|
| NFR-SEC-01 | TLS cho mọi traffic (HTTPS/WSS/TURN-TLS); TLS 1.2+ (ưu tiên 1.3); không cho phép `http`/`ws` (tắt cleartext trên Android, giữ ATS mặc định trên iOS) | Bắt buộc | Bắt buộc + HSTS |
| NFR-SEC-02 | Mật khẩu băm **Argon2id** (DEC-16; tham số tối thiểu m=19 MiB, t=2, p=1); không log mật khẩu, token, mã OTP | Bắt buộc | Bắt buộc |
| NFR-SEC-03 | Rate limiting sliding-window theo loại hành động (auth, message, invite lookup, friend request, report, voice join) | Bắt buộc | Bắt buộc + WAF |
| NFR-SEC-04 | File upload: kiểm tra magic bytes, chặn loại thực thi, quét virus (ClamAV) trước khi hiển thị; bỏ EXIF GPS khỏi ảnh | Bắt buộc | Bắt buộc |
| NFR-SEC-05 | Audit log append-only (không có API sửa/xóa; không bị xóa theo guild), giữ ≥ 90 ngày | Bắt buộc | ≥ 1 năm |
| NFR-SEC-06 | Bí mật nhạy cảm (`mfa_secret`, token OAuth) mã hóa at-rest bằng khóa ứng dụng (AES-256-GCM) lưu ngoài CSDL; secret server qua biến môi trường/secret manager, không commit vào repo | Bắt buộc | Secret manager + xoay khóa |
| NFR-SEC-07 | JWT ký bằng **EdDSA hoặc RS256** (không dùng HS256 với secret dùng chung), có `kid` để xoay khóa; access token không chứa PII ngoài `user_id` | Bắt buộc | Bắt buộc |
| NFR-SEC-08 | Mobile: token trong Keychain/Keystore (`react-native-keychain`); không log dữ liệu nhạy cảm ra console ở build release; tắt debug menu/Flipper ở release; bật Hermes bytecode | Bắt buộc | + phát hiện root/jailbreak (cảnh báo, không chặn) |
| NFR-SEC-09 | Markdown render an toàn (không HTML thô, không `javascript:`), link preview chống SSRF (REQ-MSG-02) | Bắt buộc | Bắt buộc |
| NFR-SEC-10 | Kiểm tra phụ thuộc: `npm audit`/Dependabot trên CI, không có lỗ hổng mức High/Critical chưa xử lý khi release | Bắt buộc | Bắt buộc + SAST |
| NFR-SEC-11 | Phân quyền phía server cho mọi request (REQ-ROLE-08); ID tài nguyên không đoán được không thay thế cho kiểm tra quyền | Bắt buộc | Bắt buộc |

### 6.4. Độ sẵn sàng & Tin cậy (NFR-AVAIL)

| ID | Yêu cầu | Mục tiêu đồ án | Production |
|---|---|---|---|
| NFR-AVAIL-01 | Uptime | ≥ 99% trong 2 tuần pilot | ≥ 99.9%/tháng (~43.8 phút downtime) |
| NFR-AVAIL-02 | Không mất tin nhắn đã ACK | PostgreSQL WAL + backup hằng ngày | Replication factor ≥ 3 |
| NFR-AVAIL-03 | App tự reconnect + RESUME khi mất Gateway < 60 giây, không cần user thao tác; không mất event (5.4) | Bắt buộc | Bắt buộc |
| NFR-AVAIL-04 | Voice tự kết nối lại khi mạng đổi (wifi↔4G) hoặc SFU node lỗi | ≤ 10 giây | ≤ 5 giây |
| NFR-AVAIL-05 | Degraded mode: khi Meilisearch/LiveKit/Email lỗi, chat và Focus (phần đồng hồ) vẫn hoạt động; tính năng lỗi hiển thị thông báo rõ ràng (`50301`) | Bắt buộc | Bắt buộc |

### 6.5. Khả năng bảo trì & Quan sát (NFR-OBS / NFR-MAINT)

| ID | Yêu cầu | Mục tiêu đồ án | Production |
|---|---|---|---|
| NFR-OBS-01 | Metrics Prometheus (latency, error rate, throughput, số kết nối WS, số room voice); log JSON có `request_id`, `user_id` | Prometheus + Grafana 1 dashboard | + alerting |
| NFR-OBS-02 | Tracing OpenTelemetry cho luồng gửi tin → fanout → push | Bật cho REST + job | Toàn hệ thống |
| NFR-OBS-03 | Crash & error reporting mobile (Sentry `@sentry/react-native`) kèm source map, phân biệt iOS/Android và phiên bản app | Bắt buộc | Bắt buộc |
| NFR-OBS-04 | Analytics tách khỏi CSDL giao dịch (tracking plan 2.7) | Event lưu bảng riêng/dịch vụ analytics | ETL sang data warehouse (R3) |
| NFR-MAINT-01 | TypeScript `strict` cả app và backend; ESLint + Prettier; CI chạy lint + typecheck + test cho mọi PR | Bắt buộc | Bắt buộc |
| NFR-MAINT-02 | Code coverage unit test: module Permission & Study ≥ 80%, toàn backend ≥ 60%, logic app (store/use case) ≥ 50% | Bắt buộc | ≥ 70% toàn bộ |
| NFR-MAINT-03 | Migration CSDL có version (Drizzle Kit) cho backend và CSDL cục bộ của app; không sửa migration đã chạy | Bắt buộc | Bắt buộc |

### 6.6. Mobile (NFR-MOB)

| ID | Yêu cầu | Mục tiêu đồ án | Production |
|---|---|---|---|
| NFR-MOB-01 | Cold start tới màn hình chat có dữ liệu cache (thiết bị tầm trung) | < 3s | < 2s |
| NFR-MOB-02 | Kích thước tải về từ store | Android < 60MB (AAB split theo ABI), iOS < 80MB | Android < 40MB, iOS < 60MB |
| NFR-MOB-03 | Mượt giao diện: cuộn danh sách tin nhắn 1,000 tin, drawer, animation | ≥ 55 FPS trên thiết bị tầm trung, không khung hình đứng > 700ms | ≥ 58 FPS |
| NFR-MOB-04 | Bộ nhớ | RAM app < 300MB khi đang trong voice có 4 video; không rò rỉ bộ nhớ sau 30 phút chuyển kênh liên tục | < 250MB |
| NFR-MOB-05 | Phiên bản OS & thiết bị tối thiểu | iOS ≥ 15, Android ≥ 8.0 (API 26), RAM ≥ 3GB | Như đồ án |
| NFR-MOB-06 | Pin | Voice audio-only 60 phút khóa màn hình: ≤ 10% pin; Focus Solo 60 phút khóa màn hình (không voice): ≤ 2% pin; không có wake lock khi không ở voice/call | ≤ 8% / ≤ 1% |
| NFR-MOB-07 | Dữ liệu di động | Silent Room audio-only ≤ 20MB/giờ; ảnh trong chat tải bản thumbnail trước, ảnh gốc khi bấm; tùy chọn "Tiết kiệm dữ liệu" tắt tự tải video | Như đồ án |
| NFR-MOB-08 | Ổn định | Crash-free sessions ≥ 99% (mỗi nền tảng), ANR rate Android < 0.47% | Crash-free ≥ 99.5% |
| NFR-MOB-09 | Offline | Xem được guild, channel, 50 tin gần nhất của các kênh đã mở, bạn bè, task được gán khi không có mạng; soạn tin vào outbox, tự gửi khi có mạng (7.3.2) | Như đồ án |
| NFR-MOB-10 | Quyền hệ điều hành | Chỉ xin quyền tại thời điểm cần (mic khi vào voice lần đầu, camera khi bật camera, thông báo sau onboarding, ảnh khi chọn ảnh — dùng photo picker không cần quyền toàn thư viện nếu hệ điều hành hỗ trợ); có màn hình giải thích trước khi hộp thoại hệ thống hiện; app vẫn dùng được khi bị từ chối quyền | Bắt buộc | Bắt buộc |
| NFR-MOB-11 | Background | Chỉ khai báo background mode/foreground service thật sự dùng: iOS `audio`, `voip` (v1.x), `remote-notification`; Android foreground service type `microphone`/`camera` (voice/call), `mediaProjection` (screen share v1.x); tuân thủ giới hạn khởi chạy foreground service của Android 12+ | Bắt buộc | Bắt buộc |
| NFR-MOB-12 | Cập nhật app | Kiểm tra phiên bản tối thiểu khi mở app (`42601`) → màn hình bắt buộc cập nhật; cập nhật JS qua OTA (EAS Update) chỉ cho bản vá không đổi native, tuân thủ chính sách store | Bắt buộc | Bắt buộc + staged rollout |
| NFR-MOB-13 | Tương thích hãng Android | Test trên ≥ 3 hãng (Samsung, Xiaomi, Oppo/Vivo); có hướng dẫn trong app tắt tối ưu pin khi phát hiện notification/foreground service bị hệ thống chặn | Bắt buộc | Bắt buộc |
| NFR-MOB-14 | Hỗ trợ dark/light theo hệ thống, notch/Dynamic Island, gesture navigation Android, bàn phím không che ô nhập | Bắt buộc | Bắt buộc |

### 6.7. Khả năng truy cập (NFR-A11Y)

| ID | Yêu cầu | Mục tiêu đồ án | Production |
|---|---|---|---|
| NFR-A11Y-01 | Mọi phần tử tương tác có `accessibilityLabel`/`accessibilityRole`; đọc được bằng VoiceOver và TalkBack cho các luồng UF-01…UF-07 | Bắt buộc cho UF-01, UF-04, UF-05 | Toàn bộ, theo WCAG 2.1 AA |
| NFR-A11Y-02 | Tương phản chữ ≥ 4.5:1; thông tin không chỉ truyền đạt bằng màu (pha Focus có nhãn/icon) | Bắt buộc | Bắt buộc |
| NFR-A11Y-03 | Hỗ trợ cỡ chữ hệ thống tới 200% không cắt chữ ở màn hình chính | Bắt buộc | Bắt buộc |
| NFR-A11Y-04 | Vùng chạm tối thiểu 44×44pt (iOS) / 48×48dp (Android) | Bắt buộc | Bắt buộc |
| NFR-A11Y-05 | Tôn trọng "Giảm chuyển động" của hệ thống (tắt animation không cần thiết) | Should | Bắt buộc |

### 6.8. Quốc tế hóa (NFR-I18N)

| ID | Yêu cầu | Mục tiêu đồ án | Production |
|---|---|---|---|
| NFR-I18N-01 | Ngôn ngữ tiếng Việt (mặc định) & tiếng Anh; không hard-code chuỗi UI (dùng file ngôn ngữ) | Bắt buộc | + ngôn ngữ khác |
| NFR-I18N-02 | Định dạng ngày giờ, số theo locale; thời gian hiển thị theo timezone của thiết bị, tính toán Study theo `user_settings.timezone` | Bắt buộc | Bắt buộc |
| NFR-I18N-03 | Tìm kiếm & autocomplete (mention, username, tên kênh) không phân biệt dấu tiếng Việt; chuẩn hóa Unicode NFC khi lưu | Bắt buộc | Bắt buộc |

### 6.9. Quyền riêng tư & Dữ liệu cá nhân (NFR-PRIV)

| ID | Yêu cầu | Mục tiêu đồ án | Production |
|---|---|---|---|
| NFR-PRIV-01 | Privacy Policy & Điều khoản truy cập được trong app và trang store; consent khi đăng ký có lưu phiên bản & thời điểm (R7) | Bắt buộc | Bắt buộc |
| NFR-PRIV-02 | Thu thập tối thiểu: không thu thập danh bạ, vị trí; IP chỉ lưu dạng rút gọn cho danh sách phiên, IP đầy đủ trong log tối đa 30 ngày | Bắt buộc | Bắt buộc |
| NFR-PRIV-03 | Analytics: pseudonymized `user_id`, không chứa nội dung tin nhắn; user tắt được | Bắt buộc | Bắt buộc |
| NFR-PRIV-04 | Quyền của chủ thể dữ liệu: xem/sửa profile, **xuất dữ liệu** (JSON: profile, tin nhắn của mình, dữ liệu học tập) trong ≤ 7 ngày, **xóa tài khoản** (REQ-AUTH-10) | Xóa: bắt buộc; Xuất: Should | Bắt buộc |
| NFR-PRIV-05 | Thống kê học tập là dữ liệu cá nhân: mặc định chỉ bản thân thấy chi tiết; chia sẻ với guild theo cài đặt (REQ-STUDY-04/13) | Bắt buộc | Bắt buộc |
| NFR-PRIV-06 | Quy trình xử lý sự cố lộ lọt dữ liệu: phát hiện → đánh giá → thông báo theo quy định → khắc phục (runbook) | Runbook đơn giản | Runbook đầy đủ + diễn tập |
| NFR-PRIV-07 | Thời hạn lưu trữ: xem Mục 8.6 | Bắt buộc | Bắt buộc |

### 6.10. Sao lưu & Khôi phục (NFR-BACKUP)

| ID | Yêu cầu | Mục tiêu đồ án | Production |
|---|---|---|---|
| NFR-BACKUP-01 | Sao lưu PostgreSQL | Hằng ngày, giữ 7 bản; object storage bật versioning | PITR (WAL archiving), giữ 30 ngày, lưu khác region |
| NFR-BACKUP-02 | RPO / RTO | RPO ≤ 24h / RTO ≤ 8h | RPO ≤ 5 phút / RTO ≤ 1h |
| NFR-BACKUP-03 | Diễn tập khôi phục | 1 lần trước buổi bảo vệ đồ án | Hằng quý |

### 6.11. Khả dụng (NFR-USAB)

| ID | Yêu cầu | Mục tiêu đồ án | Production |
|---|---|---|---|
| NFR-USAB-01 | User mới hoàn thành UF-02 (tạo Study Server & mời bạn) không cần hướng dẫn | ≥ 80% trong buổi test 5 người | ≥ 90% |
| NFR-USAB-02 | Bắt đầu Focus Session từ màn hình chính | ≤ 3 lần chạm | ≤ 2 lần chạm |
| NFR-USAB-03 | Thông điệp lỗi bằng ngôn ngữ người dùng, nêu cách khắc phục | Bắt buộc | Bắt buộc |

---

## 7. KIẾN TRÚC HỆ THỐNG & TECH STACK

### 7.1. Kiến trúc tổng thể

#### 7.1.1. Kiến trúc MVP-Đồ án (được nghiệm thu)

```
 ┌──────────────────────────────┐
 │  NezFocus Mobile App          │
 │  (React Native, iOS/Android)  │
 └──┬──────────┬──────────┬──────┘
    │ HTTPS    │ WSS      │ WebRTC (UDP / TURN-TLS 443)
    │ REST     │ Gateway  │
 ┌──▼──────────▼───────┐  │        ┌──────────────────────┐
 │ Reverse proxy (TLS)  │  └───────▶│  LiveKit SFU (+TURN)  │
 │ Caddy / Nginx        │           └─────────┬────────────┘
 └──┬───────────────────┘                     │ webhook
 ┌──▼──────────────────────────────────────────▼───────────────────┐
 │                 NestJS Modular Monolith (TypeScript)              │
 │  Auth · Users · Guilds/Channels · Permission · Messaging ·         │
 │  Gateway (ws) · Voice (LiveKit token) · Calls · Study (Focus,     │
 │  Streak, Tasks, Events) · Notification · Search · Safety · Admin  │
 │  ── event nội bộ: @nestjs/event-emitter                           │
 │  ── Scheduler/Worker: BullMQ (nhắc deadline, purge, email, index, │
 │     link preview, virus scan, đóng Focus treo)                    │
 └──┬─────────────┬──────────────┬─────────────┬─────────────┬──────┘
    │             │              │             │             │
 ┌──▼───────┐ ┌───▼─────┐ ┌──────▼─────┐ ┌─────▼─────┐ ┌─────▼──────────┐
 │PostgreSQL│ │  Redis   │ │Meilisearch │ │MinIO / S3 │ │ ClamAV         │
 │(toàn bộ  │ │(pub/sub, │ │ (v1.x)     │ │(file,     │ │                │
 │ dữ liệu, │ │ presence,│ └────────────┘ │ avatar)   │ └────────────────┘
 │ kể cả    │ │ rate lim,│                └───────────┘
 │ messages)│ │ resume,  │     Bên ngoài: FCM/APNs · Email provider ·
 └──────────┘ │ BullMQ)  │               Google/Apple OIDC · Sentry
              └──────────┘
 Triển khai: Docker Compose trên 1 VM (hoặc PaaS), 1 domain nezfocus.app cho deep link.
```

#### 7.1.2. Kiến trúc production mục tiêu (định hướng, không nghiệm thu)

```
 Mobile App ──HTTPS──▶ API Gateway/LB (Envoy) ──▶ Auth Svc · Guild Svc · Messaging Svc · Study Svc
            ──WSS────▶ Gateway Cluster (N node) ◀──▶ Redis Cluster (presence, resume)
            ──WebRTC─▶ LiveKit cluster multi-region (+TURN theo region)
 Messaging Svc ──▶ ScyllaDB (messages, partition (channel_id, bucket))
 Mọi service ──▶ PostgreSQL (metadata, primary + read replica)
 NATS JetStream: MESSAGE_CREATED → Fanout (Gateway nodes) · Notification Svc (FCM/APNs/VoIP)
                 · Search Indexer (Meilisearch) · Study Svc (thống kê) · Analytics ETL
 Scheduler/Worker cluster · Call Signaling Svc · Object Storage + CDN · Observability stack
```

### 7.2. Tech Stack (đã chốt)

| Lớp | Công nghệ chốt | Ghi chú / ADR |
|---|---|---|
| Mobile framework | **React Native + TypeScript, Expo (development build + prebuild/CNG)**, Hermes | ADR-01, ADR-02 |
| Điều hướng | **React Navigation** (native stack, bottom tabs, drawer lồng nhau cho drawer trái/phải) | ADR-03 |
| State | **Zustand** (client/UI state, session, voice, focus timer) + **TanStack Query** (server state, cache, retry) | ADR-04 |
| CSDL cục bộ | **op-sqlite + Drizzle ORM** (schema & migration có type) | ADR-05 |
| WebRTC | **@livekit/react-native** (+ `@livekit/react-native-webrtc`) | ADR-10 |
| Cuộc gọi hệ thống | **react-native-callkeep** (CallKit/ConnectionService) + **react-native-voip-push-notification** (PushKit) — v1.x | ADR-06 |
| Push | **@react-native-firebase/messaging** | ADR-07 |
| Local notification | **Notifee** (`@notifee/react-native`) | ADR-07 |
| Cache ảnh | **expo-image** | ADR-05 |
| Lưu token bảo mật | **react-native-keychain** | NFR-SEC-08 |
| Tác vụ nền định kỳ | **react-native-background-fetch** | 7.3.2 |
| Khác (mobile) | `@react-native-community/netinfo` (trạng thái mạng), `i18next` + `react-i18next`, `zod` (validate, dùng chung với backend), `@sentry/react-native`, `openapi-typescript` + `openapi-fetch` (API client có type sinh từ OpenAPI) | |
| Build & phân phối | **EAS Build / EAS Submit**, TestFlight, Google Play Internal testing; EAS Update cho bản vá JS | NFR-MOB-12 |
| Backend | **NestJS modular monolith (TypeScript)**, **Drizzle ORM**, WebSocket bằng **`ws`** (`@nestjs/platform-ws`), `ioredis`, **BullMQ**, `livekit-server-sdk`, `firebase-admin`, APNs VoIP qua module `http2` của Node | ADR-08, ADR-09 |
| CSDL | **PostgreSQL 16** (toàn bộ dữ liệu MVP); **ScyllaDB** là mục tiêu production cho messages | ADR-13 |
| Cache / Pub-Sub | **Redis 7** | |
| Message broker | **Không dùng ở MVP** (event-emitter + BullMQ); production: **NATS JetStream** | ADR-11 |
| SFU | **LiveKit** (TURN tích hợp) | ADR-10 |
| Tìm kiếm | **Meilisearch** | ADR-12 |
| Object storage | **MinIO** (tự host khi demo) — tương thích S3 để chuyển sang S3 | |
| Băm mật khẩu | **Argon2id** (`argon2` npm) | DEC-16 |
| Observability | Prometheus + Grafana, OpenTelemetry, Loki; Sentry cho app & backend | NFR-OBS |
| Triển khai | Docker Compose trên 1 VM (đồ án); Kubernetes (production) | |

### 7.3. Kiến trúc Mobile App (React Native) — TRỌNG TÂM ĐỒ ÁN

#### 7.3.1. Layered Architecture

```
┌──────────────────────────────────────────────────────────────────┐
│ PRESENTATION  (src/features/*/screens, components)                │
│  - Screens: GuildDrawer, ChannelScreen, VoiceRoomScreen,          │
│    FocusTimerPanel, SoloFocusScreen, TaskBoardPanel, TodoScreen,  │
│    FriendsScreen, DMScreen, CallScreen, IncomingCallOverlay       │
│  - Hooks trình bày (useChannelMessages, useFocusSession…) chỉ     │
│    đọc Zustand/TanStack Query và gọi use case                     │
└──────────────────────────────┬───────────────────────────────────┘
                               │ gọi use case (hàm TS thuần)
┌──────────────────────────────▼───────────────────────────────────┐
│ DOMAIN  (src/domain) — TypeScript thuần, không import React/RN     │
│  - Entities & types (sinh từ OpenAPI + zod)                        │
│  - Use cases: sendMessage, joinVoice, startFocusSession,           │
│    computeFocusPhase, acceptFriendRequest, startCall…              │
│  - Permission calculator (package dùng chung với backend, 8.5)    │
│  - Repository interfaces                                           │
└──────────────────────────────┬───────────────────────────────────┘
                               │ implement bởi
┌──────────────────────────────▼───────────────────────────────────┐
│ DATA  (src/data)                                                    │
│  - Repositories: kết hợp Remote + Local, cache-first/network-first  │
│  - Remote: ApiClient (openapi-fetch), GatewayClient (WebSocket tự  │
│    viết theo 5.4), VoiceClient (LiveKit), PushService (RNFirebase), │
│    CallService (CallKeep/VoIP push)                                │
│  - Local: op-sqlite + Drizzle (8.7), Keychain; cài đặt cục bộ      │
│    lưu trong bảng kv của SQLite                                     │
│  - Sync engine: áp Gateway event vào SQLite → cập nhật TanStack    │
│    Query cache (setQueryData) → UI render                          │
└──────────────────────────────────────────────────────────────────┘
```

**Nguyên tắc bắt buộc:**
- Screen/hook trình bày **không** gọi trực tiếp `fetch`/WebSocket/LiveKit — luôn qua use case → repository, để thay backend thật bằng mock server chỉ cần đổi implementation repository.
- **Zustand** giữ state phía client: phiên đăng nhập, trạng thái kết nối Gateway, voice state hiện tại, Focus timer, UI (drawer, draft tin nhắn). **TanStack Query** giữ server state (guild, channel, message page, task, profile) với `queryFn` đọc local trước rồi đồng bộ mạng. Không lưu cùng một dữ liệu ở cả hai nơi.
- Domain layer có unit test không cần môi trường RN (Jest thuần).

**Cấu trúc thư mục đề xuất:** `app.config.ts`, `plugins/` (local config plugins), `src/{app (navigation, providers), features/{auth,guild,chat,voice,study,tasks,friends,calls,settings}, domain, data, shared (ui kit, theme tokens, i18n)}`.

#### 7.3.2. Offline-first & Cache cục bộ

| Loại dữ liệu | Chiến lược | Công nghệ |
|---|---|---|
| Profile, guild, channel, role (để tính quyền offline), bạn bè, DM list, settings | Cache-first; cập nhật từ `READY`/event Gateway và REST khi mở màn hình | op-sqlite + Drizzle |
| Lịch sử tin nhắn (≥ 50 tin gần nhất mỗi kênh đã mở, tối đa 200 kênh, dọn LRU) | Ghi xuống SQLite khi nhận từ Gateway/REST; hiển thị local trước, gọi mạng để lấy delta bằng `after=<last_message_id>` | Bảng `cached_messages` (8.7) |
| Outbox | Tin nhắn/reaction/task tạo khi offline lưu với `status='pending'` + `nonce`; flush theo thứ tự khi NetInfo báo có mạng và khi app vào foreground; `react-native-background-fetch` thử flush định kỳ (best-effort, iOS không đảm bảo thời điểm). Retry lũy thừa, tối đa 24h rồi đánh dấu `failed`. | Bảng `outbox` |
| Task được gán, Focus stats, streak | Cache để xem offline (REQ-STUDY-08) | SQLite |
| Ảnh (avatar, đính kèm) | Disk cache LRU giới hạn 200MB | expo-image (`cachePolicy: 'disk'`) |
| Token | Không bao giờ lưu trong SQLite/AsyncStorage | react-native-keychain |

Khi đăng xuất hoặc xóa tài khoản: xóa file SQLite, cache ảnh, Keychain (REQ-AUTH-07).

#### 7.3.3. Vòng đời kết nối Gateway trên Mobile

```
App Foreground               → WebSocket kết nối, heartbeat theo heartbeat_interval (~41s)
App Background, KHÔNG trong  → Giữ kết nối tối đa ~30s (thời gian OS cho phép), sau đó app
  voice/call/Focus nhóm         chủ động đóng socket; sự kiện quan trọng tới qua push (FCM/APNs)
App Background, ĐANG trong   → Giữ socket + LiveKit sống nhờ iOS background mode `audio`
  voice/Focus nhóm              (audio session active) / Android foreground service type
                                `microphone` (notification cố định "Đang trong phòng học");
                                heartbeat Focus 60s tiếp tục (REQ-STUDY-15, DEC-06)
App Foreground trở lại       → Kết nối lại + RESUME {session_id, seq} → server phát lại event bị lỡ
                                (≤ 60s / 1000 event); nếu INVALID_SESSION → IDENTIFY + tải lại
                                delta qua REST (NFR-AVAIL-03)
```

- **Không polling liên tục:** tốn pin/data và vi phạm giới hạn chạy nền của iOS/Android.
- **Push song song WebSocket:** push bảo đảm nhận mention/DM/cuộc gọi khi app bị kill; WebSocket chỉ tối ưu trải nghiệm real-time khi app mở.
- Server biết thiết bị nào đang "foreground" nhờ Gateway session active → dùng cho REQ-NOTIF-04.

#### 7.3.4. Kiến trúc cuộc gọi DM trên Mobile

**MVP-Đồ án (REQ-DMVC-01):**
```
Caller: POST /channels/{dm}/calls → server tạo call_sessions(status=ringing)
  ├─ Callee có Gateway foreground → event CALL_CREATE → IncomingCallOverlay (nhạc chuông, rung)
  └─ Callee không foreground → FCM high-priority / APNs time-sensitive → bấm notification mở app
Callee Accept → POST /calls/{id}/accept → cả 2 lấy LiveKit token (room call:<id>) → kết nối media
Hết 45s không trả lời → server đặt missed, system message (REQ-DMVC-03)
```

**v1.x (REQ-DMVC-04):**
```
                    ┌───────────────────────┐
                    │  Calls module (server)  │
                    └──────────┬────────────┘
         ┌─────────────────────┴─────────────────────┐
  iOS: APNs VoIP push (PushKit)              Android: FCM data message priority high
  → react-native-voip-push-notification      → headless handler → react-native-callkeep
  → NGAY LẬP TỨC gọi RNCallKeep.displayIncomingCall   (ConnectionService self-managed)
    (bắt buộc với mọi VoIP push)               + full-screen intent notification
         └─────────────────────┬─────────────────────┘
                               │ user trả lời trên UI gốc
                    ┌──────────▼────────────┐
                    │ @livekit/react-native  │  kết nối room call:<id>
                    └────────────────────────┘
```

- Mọi VoIP push **phải** được báo cho CallKit, kể cả khi cuộc gọi đã kết thúc (khi đó báo rồi kết thúc ngay) — nếu không iOS sẽ ngừng giao VoIP push cho app.
- Group DM: trước khi tạo cuộc gọi, kiểm tra `call_sessions` đang `active` của kênh → join room hiện có thay vì tạo mới (REQ-DMVC-02).
- Audio session của cuộc gọi do CallKeep/CallKit quản lý; LiveKit cấu hình không tự kích hoạt audio session trên iOS khi dùng CallKit.

#### 7.3.5. Thư viện mobile & cấu hình native

| Hạng mục | Thư viện | Cấu hình native (qua Expo config plugin) |
|---|---|---|
| WebRTC/LiveKit | `@livekit/react-native`, `@livekit/react-native-webrtc` | Quyền mic/camera, background mode `audio`; Android foreground service `microphone`/`camera`. Dùng config plugin do LiveKit/cộng đồng cung cấp. |
| Push | `@react-native-firebase/app`, `@react-native-firebase/messaging` | `google-services.json`, `GoogleService-Info.plist`, capability Push Notifications. |
| Local notification | `@notifee/react-native` | Android: notification channels (Tin nhắn, Cuộc gọi, Nhắc học tập, Phòng học đang chạy); quyền exact alarm nếu dùng (7.3.6). |
| Cuộc gọi | `react-native-callkeep`, `react-native-voip-push-notification` (v1.x) | Background mode `voip`, capability PushKit; Android `MANAGE_OWN_CALLS`, `USE_FULL_SCREEN_INTENT`. Nếu thư viện chưa có config plugin phù hợp → viết **local config plugin** trong `plugins/` (RISK-05). |
| CSDL | `@op-engineering/op-sqlite`, `drizzle-orm` | Migration Drizzle đóng gói trong bundle, chạy khi app khởi động. |
| Token | `react-native-keychain` | iOS Keychain accessibility `AFTER_FIRST_UNLOCK_THIS_DEVICE_ONLY`. |
| Nền | `react-native-background-fetch` | iOS background mode `fetch` + `BGTaskScheduler` identifiers. |
| Ảnh | `expo-image`, `expo-image-picker`, `expo-image-manipulator` | Chuỗi mô tả quyền ảnh/camera. |
| Deep link | `expo-linking` + React Navigation linking | Associated Domains `applinks:nezfocus.app`; Android intent filter `autoVerify`. |

#### 7.3.6. Focus Timer & Local Notification

Đồng hồ Focus (REQ-STUDY-01/10) **tính được trước toàn bộ mốc chuyển pha** từ `server_started_at`, cấu hình chu kỳ và thông tin pause — nên không cần server push cho từng mốc:

1. Khi join Focus Session (hoặc bắt đầu Solo), app nhận `server_started_at`, cấu hình và `server_time` (qua Gateway hoặc REST). App tính `clock_offset = server_time − thời gian máy lúc nhận` và dùng thời gian đã hiệu chỉnh cho mọi phép tính.
2. App dùng **Notifee trigger notification** (`TimestampTrigger`) lên lịch local notification cho các mốc chuyển pha trong **2 giờ tới** (tối đa 16 thông báo, dưới giới hạn 64 thông báo chờ của iOS), bổ sung dần khi app chạy lại. Android: dùng `alarmManager: { allowWhileIdle: true }`; nếu user không cấp quyền exact alarm (Android 12+), chấp nhận sai lệch ≤ vài phút và ghi chú trong UI.
3. Khi có `FOCUS_SESSION_UPDATE` (pause/resume/skip/stop/sửa cấu hình) hoặc user rời phòng: **hủy toàn bộ** notification đã lên lịch và lên lịch lại.
4. Khi app trở lại foreground: tính lại `elapsed = now_hiệu_chỉnh − server_started_at − total_paused` để hiển thị đúng pha, không cần hỏi server (trừ khi đã quá resume window → lấy lại trạng thái session qua REST).
5. Số phút học do **server** tính theo REQ-STUDY-15 (heartbeat 60s qua op `FOCUS_HEARTBEAT` cho nhóm; sự kiện start/stop cho Solo). App không tự gửi "số phút đã học".

### 7.4. Phạm vi triển khai đồ án so với production

| Thành phần production (7.1.2) | Triển khai MVP-Đồ án (7.1.1) |
|---|---|
| Microservices | NestJS modular monolith, module ranh giới rõ (NFR-SCALE-04) |
| ScyllaDB (messages) | Bảng `messages` trong PostgreSQL, index `(channel_id, id DESC)` (8.2) — repository interface giữ nguyên để chuyển đổi |
| Gateway cluster N node | 1–2 instance NestJS `ws`, subscription trong bộ nhớ + Redis Pub/Sub |
| NATS JetStream | `@nestjs/event-emitter` (đồng bộ trong process) + BullMQ cho việc bất đồng bộ |
| LiveKit multi-region + TURN cluster | 1 LiveKit node (hoặc LiveKit Cloud) với TURN tích hợp |
| Kubernetes auto-scale | Docker Compose trên 1 VM, không auto-scale |
| Data warehouse + ETL | Bảng `analytics_events` hoặc dịch vụ analytics có free tier |

### 7.5. Sequence Diagrams

#### 7.5.1. Gửi tin nhắn (MVP-Đồ án)

```
App A          NestJS (REST)           PostgreSQL     Redis            Gateway(ws)      App B     Notification/BullMQ
  │ POST /channels/{c}/messages {content, nonce}                                                          │
  │───────────────▶│                       │             │                 │               │               │
  │                │ verify JWT            │             │                 │               │               │
  │                │ Permission.check(SEND_MESSAGES)     │                 │               │               │
  │                │ nonce đã tồn tại? ───────────────▶  │ (idempotency)   │               │               │
  │                │ rate limit sliding window ────────▶ │                 │               │               │
  │                │ INSERT message (id Snowflake) ─▶│   │                 │               │               │
  │◀── 201 {message, nonce} ──────────────│             │                 │               │               │
  │                │ emit MESSAGE_CREATED (event-emitter)│                 │               │               │
  │                │ publish channel:{c} ───────────────▶│──────────────▶  │ lọc VIEW_CHANNEL               │
  │                │                       │             │                 │── MESSAGE_CREATE ──▶│          │
  │                │ enqueue push-job (mention/DM, user không foreground) ───────────────────────────────▶ │
  │                │ enqueue link-preview-job, search-index-job (v1.x) ─────────────────────────────────▶ │
```

#### 7.5.2. Bắt đầu Focus Session & đồng bộ pha

```
App A (trong voice)       NestJS Study module        PostgreSQL        Gateway        Apps trong kênh
  │ POST /channels/{v}/focus-session {focus:25, break:5}                                   │
  │──────────────────────────▶│ kiểm tra: A đang ở voice {v}, quyền (REQ-STUDY-11),        │
  │                           │ chưa có session active (partial unique index)              │
  │                           │ INSERT focus_sessions(server_started_at=now()) ─▶│          │
  │                           │ INSERT focus_session_intervals cho participant hiện có     │
  │◀── 201 {session, server_time}                                                          │
  │                           │── FOCUS_SESSION_START {server_started_at, cfg, server_time} ─▶ mọi app
  │  (mỗi app) tính clock_offset, render đồng hồ, lên lịch Notifee cho các mốc pha          │
  │  (mỗi app, mỗi 60s) op 5 FOCUS_HEARTBEAT ──────────────────────────────▶│               │
  │                           │ cập nhật last_heartbeat_at của interval                    │
  │  Rời kênh / LiveKit webhook participant_left → đóng interval, tính focused_minutes      │
```

#### 7.5.3. Cuộc gọi 1-1 (MVP)

```
App A            Calls module            Gateway / Push              App B
  │ POST /channels/{dm}/calls ─▶│ call_sessions(ringing)                │
  │                            │── CALL_CREATE (nếu B foreground) ─────▶│ IncomingCallOverlay
  │                            │── FCM/APNs ưu tiên cao (nếu không) ───▶│ notification → mở app
  │                            │◀──────────── POST /calls/{id}/accept ──│
  │◀── CALL_UPDATE(active) ────│── CALL_UPDATE(active) ────────────────▶│
  │ POST /voice/token {call_id} ─▶ token room call:<id>   ◀── POST /voice/token ──│
  │════════════════ media qua LiveKit (TURN nếu cần) ═══════════════════│
  │  (45s không accept) → status=missed, system message "Cuộc gọi nhỡ" cho cả hai │
```

### 7.6. Architecture Decision Records (ADR)

| ADR | Quyết định | Phương án đã cân nhắc | Lý do chọn | Hệ quả |
|---|---|---|---|---|
| ADR-01 | React Native + TypeScript cho mobile | Flutter (thiết kế v1.0), Kotlin/Swift native | Một ngôn ngữ (TypeScript) cho app và backend NestJS, dùng chung type/zod/permission calculator; hệ sinh thái có SDK chính thức cho LiveKit, Firebase. | Phải quản lý native module cẩn thận; hiệu năng danh sách dài cần tối ưu (FlashList/`FlatList` virtualization). |
| ADR-02 | Expo development build + prebuild (CNG), không Expo Go, không bare RN | Bare React Native; Expo Go | Giữ tiện ích Expo (EAS Build iOS không cần máy Mac cho mọi thành viên, config plugins, EAS Update) mà vẫn dùng được native module (LiveKit, CallKeep, VoIP push). Thư mục `ios/`, `android/` được sinh lại, không commit. | Native module không có config plugin → team viết local plugin (RISK-05). Chỉnh native thủ công bị cấm; mọi thay đổi native qua plugin. |
| ADR-03 | React Navigation | Expo Router | Cấu trúc drawer trái/phải lồng nhau + bottom tabs kiểu Discord dễ kiểm soát bằng cấu hình navigator tường minh; tài liệu phong phú. | Deep link cấu hình thủ công qua `linking` config. |
| ADR-04 | Zustand + TanStack Query | Redux Toolkit + RTK Query; MobX | Ít boilerplate; TanStack Query xử lý cache/retry/stale server state; Zustand đủ cho state client đơn giản và dùng được ngoài React (GatewayClient cập nhật trực tiếp). | Cần quy ước rõ dữ liệu nào ở đâu (7.3.1) để tránh trùng lặp. |
| ADR-05 | op-sqlite + Drizzle cho CSDL cục bộ; expo-image cho cache ảnh | WatermelonDB; FastImage | op-sqlite (JSI) nhanh, Drizzle cho schema/migration có type và dùng cùng ORM với backend; cơ chế sync của NezFocus dựa trên Gateway event + cursor delta nên không cần giao thức sync riêng của WatermelonDB. expo-image được Expo duy trì, có disk cache, hỗ trợ tốt với prebuild; FastImage ít được duy trì. | Tự viết sync engine (áp event vào SQLite); reactive UI qua TanStack Query thay vì observable của DB. |
| ADR-06 | react-native-callkeep + react-native-voip-push-notification (v1.x) | Tự viết native module CallKit/ConnectionService | Thư viện phổ biến, bao đủ CallKit và ConnectionService; tiết kiệm thời gian. | Cần kiểm tra tương thích phiên bản RN mới trước khi nâng cấp; phụ thuộc Apple Developer Program. |
| ADR-07 | @react-native-firebase/messaging + Notifee | expo-notifications | RNFirebase cho FCM đầy đủ (data message, headless handler cho cuộc gọi Android); Notifee cho notification channel, trigger notification chính xác (AlarmManager), foreground service notification. | Hai thư viện notification phải phối hợp: FCM nhận, Notifee hiển thị. |
| ADR-08 | NestJS modular monolith | Microservices Go/Node như v1.0; Express thuần | Cùng TypeScript với app; module/DI rõ ràng để tách service sau; một deployable đơn giản cho đồ án. | Gateway và REST chạy chung process ở đồ án; khi tách, Gateway là module đầu tiên tách ra. |
| ADR-09 | Drizzle ORM (backend) + `ws` cho Gateway | Prisma/TypeORM; Socket.IO | Drizzle: SQL rõ ràng, migration có version, dùng chung với app. `ws`: triển khai đúng giao thức opcode/resume tự đặc tả (5.4), không phụ thuộc framing riêng của Socket.IO. | Tự viết heartbeat/resume/backoff ở cả server và client. |
| ADR-10 | LiveKit (SFU + TURN) | Mediasoup; mesh P2P | SDK React Native chính thức, simulcast/adaptive stream, TURN tích hợp, có bản Cloud để demo; mediasoup cần tự viết nhiều signaling. | Phụ thuộc LiveKit; chi phí băng thông nếu dùng Cloud vượt free tier. |
| ADR-11 | MVP không dùng message broker; production dùng NATS JetStream | Kafka; NATS ngay từ MVP | Đồ án 1 process: event-emitter đủ; BullMQ tận dụng Redis sẵn có cho job trì hoãn/retry. NATS nhẹ hơn Kafka khi cần tách service. | Event-emitter không bền vững (mất nếu process chết) → việc cần bền vững (push, email, index) phải đi qua BullMQ. |
| ADR-12 | Meilisearch | Elasticsearch/OpenSearch; PostgreSQL full-text | Nhẹ, dễ vận hành, không phân biệt dấu tiếng Việt, filter theo danh sách `channel_id` để lọc quyền (REQ-SEARCH-03). | Index là bản sao — cần job đồng bộ và xử lý xóa/sửa. |
| ADR-13 | PostgreSQL cho messages ở MVP; ScyllaDB là mục tiêu production | ScyllaDB ngay từ đầu | Một CSDL, transaction đơn giản, đủ cho quy mô đồ án; ScyllaDB chỉ đáng giá khi vượt hàng trăm triệu tin/ghi rất cao. Ngưỡng cân nhắc chuyển: bảng messages > 500 triệu dòng hoặc > 5,000 ghi/giây kéo dài. | Repository `MessageStore` phải trừu tượng hóa để thay implementation. |

---

## 8. MÔ HÌNH DỮ LIỆU (DATA MODEL)

### 8.1. Quy ước

- PostgreSQL 16 lưu **toàn bộ** dữ liệu ở MVP-Đồ án, kể cả `messages` (DEC-17). Schema ScyllaDB cho production ở 8.3.
- Khóa chính: Snowflake `BIGINT` (sinh ở backend). JSON trả về dạng string.
- Thời gian: `TIMESTAMPTZ` (UTC). Ngày theo timezone user: `DATE` kèm cột timezone đã dùng.
- Soft-delete bằng `deleted_at`. Dữ liệu cần tồn tại độc lập với vòng đời guild/user (audit log, report) **không** dùng `ON DELETE CASCADE`.
- Presence, voice state hiện tại, typing, resume buffer, rate-limit counter, idempotency key (ngoài message) lưu **Redis** (dữ liệu tạm, có TTL), không lưu PostgreSQL.
- Migration quản lý bằng Drizzle Kit (NFR-MAINT-03). DDL dưới đây là đặc tả; code migration là nguồn chính thức khi triển khai.
- Thuật ngữ "`dm_channel`" trong các REQ tương ứng một dòng trong bảng `channels` với `type` = 3 (DM) hoặc 4 (GROUP_DM): **guild channel và DM dùng chung bảng `channels`** để `messages.channel_id` luôn tham chiếu một bảng duy nhất.

### 8.2. PostgreSQL DDL

```sql
-- =====================================================================
-- USERS & ACCOUNT
-- =====================================================================
CREATE TABLE users (
    id                  BIGINT PRIMARY KEY,                 -- Snowflake
    username            VARCHAR(32)  NOT NULL UNIQUE,        -- DEC-03: unique toàn hệ thống, lowercase
    email               VARCHAR(255) UNIQUE,                 -- NULL sau khi purge tài khoản đã xóa
    password_hash       VARCHAR(255),                        -- Argon2id; NULL nếu chỉ dùng OAuth
    display_name        VARCHAR(32),
    avatar_key          TEXT,                                -- object storage key (sinh URL khi trả về)
    banner_key          TEXT,
    accent_color        INT,
    bio                 VARCHAR(190),
    account_status      VARCHAR(20) NOT NULL DEFAULT 'unverified',
                        -- unverified | active | pending_deletion | deleted | suspended (8.4.6)
    email_verified_at   TIMESTAMPTZ,
    mfa_enabled         BOOLEAN NOT NULL DEFAULT FALSE,
    mfa_secret_enc      BYTEA,                               -- mã hóa AES-256-GCM (NFR-SEC-06)
    is_platform_admin   BOOLEAN NOT NULL DEFAULT FALSE,      -- REQ-SAFE-02
    terms_version       VARCHAR(20) NOT NULL,                -- phiên bản Điều khoản/Privacy đã đồng ý
    terms_accepted_at   TIMESTAMPTZ NOT NULL,
    deletion_requested_at TIMESTAMPTZ,                       -- REQ-AUTH-10
    suspended_until     TIMESTAMPTZ,                         -- REQ-SAFE-02
    created_at          TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at          TIMESTAMPTZ NOT NULL DEFAULT now(),
    CHECK (username ~ '^[a-z0-9_.]{2,32}$')
);

CREATE TABLE user_oauth_identities (                          -- REQ-AUTH-12
    provider            VARCHAR(16) NOT NULL,                 -- google | apple
    provider_subject    VARCHAR(255) NOT NULL,
    user_id             BIGINT NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    email               VARCHAR(255),
    created_at          TIMESTAMPTZ NOT NULL DEFAULT now(),
    PRIMARY KEY (provider, provider_subject)
);

CREATE TABLE user_settings (
    user_id             BIGINT PRIMARY KEY REFERENCES users(id) ON DELETE CASCADE,
    timezone            VARCHAR(64) NOT NULL DEFAULT 'Asia/Ho_Chi_Minh', -- IANA (REQ-STUDY-03)
    timezone_changed_at TIMESTAMPTZ,                          -- giới hạn đổi 1 lần/24h
    locale              VARCHAR(8)  NOT NULL DEFAULT 'vi',
    theme               VARCHAR(8)  NOT NULL DEFAULT 'dark',   -- dark | light | system
    preferred_status    VARCHAR(12) NOT NULL DEFAULT 'online', -- online | idle | dnd | invisible
    allow_dm_from_guild_members BOOLEAN NOT NULL DEFAULT TRUE, -- REQ-DM-02
    hide_push_content   BOOLEAN NOT NULL DEFAULT FALSE,        -- REQ-NOTIF-02
    show_on_leaderboard BOOLEAN NOT NULL DEFAULT TRUE,         -- REQ-STUDY-04 (opt-out cá nhân)
    share_study_stats   BOOLEAN NOT NULL DEFAULT TRUE,         -- REQ-STUDY-13
    show_solo_focus_status BOOLEAN NOT NULL DEFAULT FALSE,     -- REQ-STUDY-10
    analytics_opt_out   BOOLEAN NOT NULL DEFAULT FALSE,        -- NFR-PRIV-03
    study_mode_schedule JSONB,                                 -- REQ-STUDY-09: [{days:[1,3,5], start:"20:00", end:"22:00"}]
    focus_sound_always  BOOLEAN NOT NULL DEFAULT FALSE,
    updated_at          TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE TABLE user_guild_settings (                            -- theo từng guild
    user_id             BIGINT NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    guild_id            BIGINT NOT NULL,                      -- FK tới guilds, khai báo sau
    notify_level        SMALLINT NOT NULL DEFAULT 1,          -- 0=all,1=mentions,2=nothing (REQ-NOTIF-03)
    muted_until         TIMESTAMPTZ,
    is_study            BOOLEAN NOT NULL DEFAULT FALSE,       -- DEC-10: user tự đánh dấu guild học tập
    allow_dm            BOOLEAN,                              -- ghi đè allow_dm_from_guild_members theo guild
    PRIMARY KEY (user_id, guild_id)
);

-- Phiên đăng nhập & refresh token rotation (REQ-AUTH-04/07)
CREATE TABLE auth_sessions (                                  -- 1 dòng = 1 family = 1 thiết bị
    id                  BIGINT PRIMARY KEY,
    user_id             BIGINT NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    device_name         VARCHAR(100),
    platform            VARCHAR(10) NOT NULL,                 -- ios | android
    app_version         VARCHAR(20),
    ip_prefix           VARCHAR(45),                          -- IP rút gọn (NFR-PRIV-02)
    trusted_until       TIMESTAMPTZ,                          -- thiết bị tin cậy cho 2FA (REQ-AUTH-03)
    created_at          TIMESTAMPTZ NOT NULL DEFAULT now(),
    last_active_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
    revoked_at          TIMESTAMPTZ,
    revoke_reason       VARCHAR(32)                           -- logout | revoked | reuse_detected | password_reset | ...
);
CREATE INDEX idx_auth_sessions_user ON auth_sessions(user_id) WHERE revoked_at IS NULL;

CREATE TABLE refresh_tokens (
    token_hash          BYTEA PRIMARY KEY,                    -- SHA-256 của token, không lưu token thô
    session_id          BIGINT NOT NULL REFERENCES auth_sessions(id) ON DELETE CASCADE,
    expires_at          TIMESTAMPTZ NOT NULL,
    used_at             TIMESTAMPTZ,                          -- đã dùng ⇒ dùng lại = reuse (thu hồi family)
    created_at          TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE TABLE auth_tokens (                                    -- xác thực email, reset mật khẩu, đổi email
    token_hash          BYTEA PRIMARY KEY,
    user_id             BIGINT NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    purpose             VARCHAR(20) NOT NULL,                 -- verify_email | reset_password | change_email
    payload             JSONB,                                -- vd email mới
    expires_at          TIMESTAMPTZ NOT NULL,
    used_at             TIMESTAMPTZ,
    created_at          TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE TABLE mfa_recovery_codes (                             -- REQ-AUTH-11
    user_id             BIGINT NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    code_hash           BYTEA NOT NULL,
    used_at             TIMESTAMPTZ,
    PRIMARY KEY (user_id, code_hash)
);

CREATE TABLE push_tokens (                                    -- REQ-NOTIF-02/04, REQ-DMVC-04
    token               TEXT PRIMARY KEY,
    user_id             BIGINT NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    session_id          BIGINT REFERENCES auth_sessions(id) ON DELETE CASCADE,
    kind                VARCHAR(12) NOT NULL,                 -- fcm | apns_voip
    platform            VARCHAR(10) NOT NULL,                 -- ios | android
    locale              VARCHAR(8),
    created_at          TIMESTAMPTZ NOT NULL DEFAULT now(),
    last_seen_at        TIMESTAMPTZ NOT NULL DEFAULT now()
);
CREATE INDEX idx_push_tokens_user ON push_tokens(user_id);

-- =====================================================================
-- GUILDS, CHANNELS (gồm DM), ROLES, MEMBERS
-- =====================================================================
CREATE TABLE guilds (
    id                  BIGINT PRIMARY KEY,
    name                VARCHAR(100) NOT NULL,
    icon_key            TEXT,
    owner_id            BIGINT NOT NULL REFERENCES users(id),
    description         VARCHAR(300),
    template            VARCHAR(20) NOT NULL DEFAULT 'general',  -- general | study_server (REQ-STUDY-05)
    topic_tags          TEXT[] NOT NULL DEFAULT '{}',            -- REQ-STUDY-07
    leaderboard_enabled BOOLEAN NOT NULL DEFAULT FALSE,          -- DEC-05: mặc định TẮT
    system_channel_id   BIGINT,                                  -- REQ-MSG-11
    created_at          TIMESTAMPTZ NOT NULL DEFAULT now(),
    deleted_at          TIMESTAMPTZ                              -- soft-delete 30 ngày (REQ-SRV-05)
);
ALTER TABLE user_guild_settings ADD FOREIGN KEY (guild_id) REFERENCES guilds(id) ON DELETE CASCADE;

CREATE TABLE channels (
    id                  BIGINT PRIMARY KEY,
    guild_id            BIGINT REFERENCES guilds(id) ON DELETE CASCADE, -- NULL với DM/GROUP_DM
    parent_id           BIGINT REFERENCES channels(id) ON DELETE SET NULL, -- category (REQ-SRV-02 AC5)
    type                SMALLINT NOT NULL,       -- 0=category,1=text,2=voice,3=DM,4=GROUP_DM
    name                VARCHAR(100),            -- NULL với DM 1-1
    position            INT NOT NULL DEFAULT 0,
    topic               VARCHAR(1024),
    icon_key            TEXT,                    -- GROUP_DM
    owner_id            BIGINT REFERENCES users(id), -- GROUP_DM owner (REQ-GDM-02)
    slow_mode_secs      INT NOT NULL DEFAULT 0 CHECK (slow_mode_secs BETWEEN 0 AND 21600),
    user_limit          SMALLINT NOT NULL DEFAULT 0 CHECK (user_limit BETWEEN 0 AND 99),
    voice_mode          VARCHAR(8) NOT NULL DEFAULT 'normal',   -- normal | silent (REQ-STUDY-02)
    focus_control       VARCHAR(10) NOT NULL DEFAULT 'everyone',-- everyone | managers (REQ-STUDY-11)
    task_board_enabled  BOOLEAN NOT NULL DEFAULT FALSE,         -- REQ-STUDY-06
    permissions_synced  BOOLEAN NOT NULL DEFAULT TRUE,          -- đồng bộ overwrite với category
    last_message_id     BIGINT,
    created_at          TIMESTAMPTZ NOT NULL DEFAULT now(),
    CHECK ((type IN (3,4)) = (guild_id IS NULL))
);
CREATE INDEX idx_channels_guild ON channels(guild_id) WHERE guild_id IS NOT NULL;
ALTER TABLE guilds ADD FOREIGN KEY (system_channel_id) REFERENCES channels(id) ON DELETE SET NULL;

CREATE TABLE channel_recipients (                             -- thành viên DM / GROUP_DM
    channel_id          BIGINT NOT NULL REFERENCES channels(id) ON DELETE CASCADE,
    user_id             BIGINT NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    joined_at           TIMESTAMPTZ NOT NULL DEFAULT now(),
    closed              BOOLEAN NOT NULL DEFAULT FALSE,       -- ẩn DM khỏi danh sách phía user (REQ-DM-03)
    PRIMARY KEY (channel_id, user_id)
);
CREATE INDEX idx_channel_recipients_user ON channel_recipients(user_id);
-- Ràng buộc tầng ứng dụng: type=3 đúng 2 người; type=4 từ 1 (trạng thái ẩn) tới 10 người (REQ-GDM-02/03)

CREATE TABLE dm_pairs (                                       -- REQ-DM-03: 1 DM 1-1 duy nhất cho mỗi cặp
    user_id_low         BIGINT NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    user_id_high        BIGINT NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    channel_id          BIGINT NOT NULL UNIQUE REFERENCES channels(id) ON DELETE CASCADE,
    PRIMARY KEY (user_id_low, user_id_high),
    CHECK (user_id_low < user_id_high)
);

CREATE TABLE roles (
    id                  BIGINT PRIMARY KEY,                   -- role @everyone có id = guild_id
    guild_id            BIGINT NOT NULL REFERENCES guilds(id) ON DELETE CASCADE,
    name                VARCHAR(100) NOT NULL,
    color               INT NOT NULL DEFAULT 0,
    permissions         BIGINT NOT NULL DEFAULT 0,            -- bitmap 8.5.1
    position            INT NOT NULL DEFAULT 0,               -- lớn hơn = cao hơn; @everyone = 0
    is_hoisted          BOOLEAN NOT NULL DEFAULT FALSE,
    is_mentionable      BOOLEAN NOT NULL DEFAULT FALSE,
    created_at          TIMESTAMPTZ NOT NULL DEFAULT now()
);
CREATE INDEX idx_roles_guild ON roles(guild_id);

CREATE TABLE channel_permission_overwrites (
    channel_id          BIGINT NOT NULL REFERENCES channels(id) ON DELETE CASCADE,
    target_id           BIGINT NOT NULL,                      -- role_id hoặc user_id
    target_type         SMALLINT NOT NULL,                    -- 0=role, 1=member
    allow               BIGINT NOT NULL DEFAULT 0,
    deny                BIGINT NOT NULL DEFAULT 0,
    PRIMARY KEY (channel_id, target_id, target_type),
    CHECK ((allow & deny) = 0)
);

CREATE TABLE guild_members (
    guild_id            BIGINT NOT NULL REFERENCES guilds(id) ON DELETE CASCADE,
    user_id             BIGINT NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    nickname            VARCHAR(32),
    joined_at           TIMESTAMPTZ NOT NULL DEFAULT now(),
    server_muted        BOOLEAN NOT NULL DEFAULT FALSE,
    server_deafened     BOOLEAN NOT NULL DEFAULT FALSE,
    timeout_until       TIMESTAMPTZ,
    last_active_at      TIMESTAMPTZ,                          -- REQ-STUDY-13
    PRIMARY KEY (guild_id, user_id)
);
CREATE INDEX idx_guild_members_user ON guild_members(user_id);

CREATE TABLE guild_member_roles (
    guild_id            BIGINT NOT NULL,
    user_id             BIGINT NOT NULL,
    role_id             BIGINT NOT NULL REFERENCES roles(id) ON DELETE CASCADE,
    PRIMARY KEY (guild_id, user_id, role_id),
    FOREIGN KEY (guild_id, user_id) REFERENCES guild_members(guild_id, user_id) ON DELETE CASCADE
);

CREATE TABLE guild_bans (                                     -- REQ-ROLE-07
    guild_id            BIGINT NOT NULL REFERENCES guilds(id) ON DELETE CASCADE,
    user_id             BIGINT NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    banned_by           BIGINT REFERENCES users(id) ON DELETE SET NULL,
    reason              VARCHAR(512),
    created_at          TIMESTAMPTZ NOT NULL DEFAULT now(),
    PRIMARY KEY (guild_id, user_id)
);

CREATE TABLE invites (
    code                VARCHAR(16) PRIMARY KEY,
    guild_id            BIGINT NOT NULL REFERENCES guilds(id) ON DELETE CASCADE,
    channel_id          BIGINT NOT NULL REFERENCES channels(id) ON DELETE CASCADE,
    creator_id          BIGINT REFERENCES users(id) ON DELETE SET NULL,
    max_uses            INT NOT NULL DEFAULT 0,               -- 0 = không giới hạn
    uses                INT NOT NULL DEFAULT 0,
    expires_at          TIMESTAMPTZ,                          -- NULL = không hết hạn
    revoked_at          TIMESTAMPTZ,
    created_at          TIMESTAMPTZ NOT NULL DEFAULT now()
);
CREATE INDEX idx_invites_guild ON invites(guild_id);

-- =====================================================================
-- MESSAGES (MVP: PostgreSQL — DEC-17)
-- =====================================================================
CREATE TABLE messages (
    id                  BIGINT PRIMARY KEY,                   -- Snowflake (chứa timestamp)
    channel_id          BIGINT NOT NULL REFERENCES channels(id) ON DELETE CASCADE,
    author_id           BIGINT NOT NULL REFERENCES users(id), -- user bị xóa: giữ id, hiển thị "Người dùng đã xóa"
    type                SMALLINT NOT NULL DEFAULT 0,          -- 8.2.1
    content             VARCHAR(2000) NOT NULL DEFAULT '',
    nonce               VARCHAR(64),                          -- REQ-MSG-09
    reply_to_id         BIGINT,                               -- không FK: tin gốc có thể bị purge
    mention_user_ids    BIGINT[] NOT NULL DEFAULT '{}',
    mention_role_ids    BIGINT[] NOT NULL DEFAULT '{}',
    mention_everyone    BOOLEAN NOT NULL DEFAULT FALSE,
    embeds              JSONB,                                -- link preview (REQ-MSG-02)
    system_data         JSONB,                                -- dữ liệu system message (call duration, focus cfg...)
    edited_at           TIMESTAMPTZ,
    pinned_at           TIMESTAMPTZ,                          -- REQ-MSG-10
    pinned_by           BIGINT,
    deleted_at          TIMESTAMPTZ,
    created_at          TIMESTAMPTZ NOT NULL DEFAULT now()
);
CREATE INDEX idx_messages_channel ON messages(channel_id, id DESC);
CREATE UNIQUE INDEX uq_messages_author_nonce ON messages(author_id, nonce) WHERE nonce IS NOT NULL;
CREATE INDEX idx_messages_pins ON messages(channel_id, pinned_at DESC) WHERE pinned_at IS NOT NULL;
CREATE INDEX idx_messages_author_time ON messages(author_id, created_at);  -- ban xóa N ngày, xuất dữ liệu

CREATE TABLE message_reactions (                              -- sửa lỗi MAP<emoji,count> của v1.0
    message_id          BIGINT NOT NULL REFERENCES messages(id) ON DELETE CASCADE,
    emoji               VARCHAR(64) NOT NULL,                 -- Unicode emoji (custom emoji: Backlog)
    user_id             BIGINT NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    created_at          TIMESTAMPTZ NOT NULL DEFAULT now(),
    PRIMARY KEY (message_id, emoji, user_id)
);
-- Giới hạn 20 emoji khác nhau/tin: kiểm tra trong transaction (SELECT COUNT(DISTINCT emoji) ... FOR UPDATE trên messages)

CREATE TABLE attachments (                                    -- REQ-MSG-04
    id                  BIGINT PRIMARY KEY,
    uploader_id         BIGINT NOT NULL REFERENCES users(id),
    channel_id          BIGINT NOT NULL REFERENCES channels(id) ON DELETE CASCADE,
    message_id          BIGINT REFERENCES messages(id) ON DELETE CASCADE,  -- NULL khi chưa gắn vào tin
    filename            VARCHAR(255) NOT NULL,
    content_type        VARCHAR(100) NOT NULL,                -- theo magic bytes, không theo khai báo client
    size_bytes          BIGINT NOT NULL CHECK (size_bytes <= 26214400),
    width               INT,
    height              INT,
    duration_ms         INT,
    storage_key         TEXT NOT NULL,
    thumbnail_key       TEXT,
    scan_status         VARCHAR(10) NOT NULL DEFAULT 'pending', -- pending | clean | infected | failed
    created_at          TIMESTAMPTZ NOT NULL DEFAULT now()
);
CREATE INDEX idx_attachments_message ON attachments(message_id);
-- Job dọn attachment chưa gắn vào tin sau 24h.

CREATE TABLE read_states (                                    -- REQ-NOTIF-01
    user_id             BIGINT NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    channel_id          BIGINT NOT NULL REFERENCES channels(id) ON DELETE CASCADE,
    last_read_message_id BIGINT NOT NULL DEFAULT 0,
    mention_count       INT NOT NULL DEFAULT 0,
    updated_at          TIMESTAMPTZ NOT NULL DEFAULT now(),
    PRIMARY KEY (user_id, channel_id)
);

-- =====================================================================
-- FRIENDS, BLOCKS
-- =====================================================================
CREATE TABLE friendships (                                    -- 1 dòng/cặp (low < high)
    user_id_low         BIGINT NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    user_id_high        BIGINT NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    status              SMALLINT NOT NULL,                    -- 0=pending, 1=accepted (8.4.2)
    requested_by        BIGINT NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    created_at          TIMESTAMPTZ NOT NULL DEFAULT now(),
    responded_at        TIMESTAMPTZ,
    PRIMARY KEY (user_id_low, user_id_high),
    CHECK (user_id_low < user_id_high)
);

CREATE TABLE user_blocks (
    blocker_id          BIGINT NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    blocked_id          BIGINT NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    created_at          TIMESTAMPTZ NOT NULL DEFAULT now(),
    PRIMARY KEY (blocker_id, blocked_id),
    CHECK (blocker_id <> blocked_id)
);

-- =====================================================================
-- CALLS (DM & Group DM)
-- =====================================================================
CREATE TABLE call_sessions (
    id                  BIGINT PRIMARY KEY,
    channel_id          BIGINT NOT NULL REFERENCES channels(id) ON DELETE CASCADE, -- type 3 hoặc 4
    initiator_id        BIGINT NOT NULL REFERENCES users(id),
    has_video           BOOLEAN NOT NULL DEFAULT FALSE,
    status              VARCHAR(10) NOT NULL DEFAULT 'ringing', -- ringing|active|missed|declined|ended (8.4.1)
    started_at          TIMESTAMPTZ NOT NULL DEFAULT now(),
    answered_at         TIMESTAMPTZ,
    ended_at            TIMESTAMPTZ,
    livekit_room        VARCHAR(64) NOT NULL                   -- call:<id>
);
CREATE UNIQUE INDEX uq_call_active_per_channel ON call_sessions(channel_id)
    WHERE status IN ('ringing','active');                      -- 1 cuộc gọi đang diễn ra/kênh

CREATE TABLE call_participants (
    call_id             BIGINT NOT NULL REFERENCES call_sessions(id) ON DELETE CASCADE,
    user_id             BIGINT NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    joined_at           TIMESTAMPTZ,
    left_at             TIMESTAMPTZ,
    response            VARCHAR(10),                           -- accepted | declined | missed
    PRIMARY KEY (call_id, user_id)
);

-- =====================================================================
-- STUDY / FOCUS (REQ-STUDY-*)
-- =====================================================================
CREATE TABLE focus_sessions (                                 -- nhóm, gắn voice channel
    id                  BIGINT PRIMARY KEY,
    channel_id          BIGINT NOT NULL REFERENCES channels(id) ON DELETE CASCADE,
    guild_id            BIGINT NOT NULL REFERENCES guilds(id) ON DELETE CASCADE,
    started_by          BIGINT NOT NULL REFERENCES users(id),
    controller_id       BIGINT NOT NULL REFERENCES users(id), -- REQ-STUDY-11
    focus_min           SMALLINT NOT NULL DEFAULT 25 CHECK (focus_min BETWEEN 5 AND 120),
    short_break_min     SMALLINT NOT NULL DEFAULT 5  CHECK (short_break_min BETWEEN 1 AND 30),
    long_break_min      SMALLINT CHECK (long_break_min BETWEEN 5 AND 60),        -- NULL = tắt nghỉ dài
    long_break_every    SMALLINT CHECK (long_break_every BETWEEN 2 AND 8),
    server_started_at   TIMESTAMPTZ NOT NULL DEFAULT now(),  -- mốc gốc đồng bộ
    paused_at           TIMESTAMPTZ,
    total_paused_ms     BIGINT NOT NULL DEFAULT 0,
    study_event_id      BIGINT,                              -- nếu tự bắt đầu từ Study Event (REQ-STUDY-12)
    ended_at            TIMESTAMPTZ,
    end_reason          VARCHAR(16)                           -- stopped | empty_channel | channel_deleted
);
CREATE UNIQUE INDEX uq_focus_active_per_channel ON focus_sessions(channel_id) WHERE ended_at IS NULL;

CREATE TABLE focus_session_intervals (                        -- 1 dòng = 1 lần join→leave (REQ-STUDY-15)
    id                  BIGINT PRIMARY KEY,
    focus_session_id    BIGINT NOT NULL REFERENCES focus_sessions(id) ON DELETE CASCADE,
    user_id             BIGINT NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    joined_at           TIMESTAMPTZ NOT NULL DEFAULT now(),
    last_heartbeat_at   TIMESTAMPTZ NOT NULL DEFAULT now(),
    left_at             TIMESTAMPTZ,
    focused_minutes     INT NOT NULL DEFAULT 0              -- tính khi đóng interval, chỉ pha focus
);
CREATE INDEX idx_focus_intervals_user ON focus_session_intervals(user_id, joined_at);

CREATE TABLE solo_focus_sessions (                            -- REQ-STUDY-10
    id                  BIGINT PRIMARY KEY,
    user_id             BIGINT NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    focus_min           SMALLINT NOT NULL CHECK (focus_min BETWEEN 5 AND 120),
    short_break_min     SMALLINT NOT NULL CHECK (short_break_min BETWEEN 1 AND 30),
    long_break_min      SMALLINT,
    long_break_every    SMALLINT,
    server_started_at   TIMESTAMPTZ NOT NULL DEFAULT now(),
    paused_at           TIMESTAMPTZ,
    total_paused_ms     BIGINT NOT NULL DEFAULT 0,
    ended_at            TIMESTAMPTZ,                          -- tự đóng sau 4 giờ nếu không có stop
    focused_minutes     INT NOT NULL DEFAULT 0
);
CREATE UNIQUE INDEX uq_solo_active_per_user ON solo_focus_sessions(user_id) WHERE ended_at IS NULL;

CREATE TABLE daily_study_stats (                              -- tổng theo local day (REQ-STUDY-03/04/13)
    user_id             BIGINT NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    local_date          DATE NOT NULL,
    timezone            VARCHAR(64) NOT NULL,
    focused_minutes     INT NOT NULL DEFAULT 0 CHECK (focused_minutes <= 960), -- tối đa 16h/ngày
    solo_minutes        INT NOT NULL DEFAULT 0,
    PRIMARY KEY (user_id, local_date)
);

CREATE TABLE guild_study_stats (                              -- phục vụ leaderboard & member activity
    guild_id            BIGINT NOT NULL REFERENCES guilds(id) ON DELETE CASCADE,
    user_id             BIGINT NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    local_date          DATE NOT NULL,
    focused_minutes     INT NOT NULL DEFAULT 0,
    PRIMARY KEY (guild_id, user_id, local_date)
);

CREATE TABLE user_study_streaks (
    user_id             BIGINT PRIMARY KEY REFERENCES users(id) ON DELETE CASCADE,
    current_streak_days INT NOT NULL DEFAULT 0,
    longest_streak_days INT NOT NULL DEFAULT 0,
    last_qualified_date DATE                                  -- local day gần nhất đạt ≥ 15 phút
);

CREATE TABLE tasks (                                          -- REQ-STUDY-06
    id                  BIGINT PRIMARY KEY,
    channel_id          BIGINT NOT NULL REFERENCES channels(id) ON DELETE CASCADE,
    guild_id            BIGINT NOT NULL REFERENCES guilds(id) ON DELETE CASCADE,
    title               VARCHAR(200) NOT NULL,
    description         VARCHAR(2000),
    assignee_id         BIGINT REFERENCES users(id) ON DELETE SET NULL,
    status              SMALLINT NOT NULL DEFAULT 0,          -- 0=todo,1=doing,2=done (8.4.4)
    due_at              TIMESTAMPTZ,
    reminded_24h_at     TIMESTAMPTZ,
    reminded_2h_at      TIMESTAMPTZ,
    created_by          BIGINT NOT NULL REFERENCES users(id),
    created_at          TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at          TIMESTAMPTZ NOT NULL DEFAULT now(),
    completed_at        TIMESTAMPTZ,
    deleted_at          TIMESTAMPTZ
);
CREATE INDEX idx_tasks_channel ON tasks(channel_id) WHERE deleted_at IS NULL;
CREATE INDEX idx_tasks_assignee_due ON tasks(assignee_id, due_at) WHERE deleted_at IS NULL AND status <> 2; -- REQ-STUDY-08
CREATE INDEX idx_tasks_reminder ON tasks(due_at) WHERE deleted_at IS NULL AND status <> 2;

CREATE TABLE study_events (                                   -- REQ-STUDY-12
    id                  BIGINT PRIMARY KEY,
    guild_id            BIGINT NOT NULL REFERENCES guilds(id) ON DELETE CASCADE,
    channel_id          BIGINT NOT NULL REFERENCES channels(id) ON DELETE CASCADE, -- voice channel
    title               VARCHAR(100) NOT NULL,
    description         VARCHAR(1000),
    starts_at           TIMESTAMPTZ NOT NULL,
    duration_min        SMALLINT NOT NULL CHECK (duration_min BETWEEN 15 AND 480),
    recurrence_days     SMALLINT[],                           -- 1..7 (thứ 2..CN); NULL = 1 lần
    recurrence_until    DATE,                                 -- tối đa 12 tuần
    timezone            VARCHAR(64) NOT NULL,                 -- timezone của người tạo, để lặp đúng giờ địa phương
    auto_start_focus    BOOLEAN NOT NULL DEFAULT FALSE,
    focus_preset        JSONB,
    created_by          BIGINT NOT NULL REFERENCES users(id),
    created_at          TIMESTAMPTZ NOT NULL DEFAULT now(),
    cancelled_at        TIMESTAMPTZ
);

CREATE TABLE study_event_interests (
    event_id            BIGINT NOT NULL REFERENCES study_events(id) ON DELETE CASCADE,
    user_id             BIGINT NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    PRIMARY KEY (event_id, user_id)
);

-- =====================================================================
-- NOTIFICATION INBOX, SAFETY, AUDIT
-- =====================================================================
CREATE TABLE notifications (                                  -- tab Thông báo
    id                  BIGINT PRIMARY KEY,
    user_id             BIGINT NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    type                VARCHAR(24) NOT NULL,   -- mention | reply | friend_request | task_reminder | study_invite | missed_call | event_reminder
    data                JSONB NOT NULL,
    read_at             TIMESTAMPTZ,
    created_at          TIMESTAMPTZ NOT NULL DEFAULT now()
);
CREATE INDEX idx_notifications_user ON notifications(user_id, id DESC);

CREATE TABLE reports (                                        -- REQ-SAFE-01 (không cascade: giữ bằng chứng)
    id                  BIGINT PRIMARY KEY,
    reporter_id         BIGINT,
    target_type         VARCHAR(10) NOT NULL,                 -- message | user | guild
    target_id           BIGINT NOT NULL,
    guild_id            BIGINT,
    reason              VARCHAR(20) NOT NULL,                 -- spam | harassment | sensitive | scam | other
    description         VARCHAR(1000),
    evidence            JSONB NOT NULL,                       -- snapshot tin + ngữ cảnh
    status              VARCHAR(10) NOT NULL DEFAULT 'open',  -- open | actioned | dismissed
    handled_by          BIGINT,
    handled_at          TIMESTAMPTZ,
    created_at          TIMESTAMPTZ NOT NULL DEFAULT now()
);
CREATE INDEX idx_reports_status ON reports(status, created_at);

CREATE TABLE audit_logs (                                     -- NFR-SEC-05: append-only, KHÔNG cascade
    id                  BIGINT PRIMARY KEY,
    guild_id            BIGINT NOT NULL,                      -- không FK: tồn tại sau khi guild bị purge
    actor_id            BIGINT NOT NULL,
    target_id           BIGINT,
    action_type         VARCHAR(50) NOT NULL,   -- MEMBER_BAN, MEMBER_KICK, MEMBER_TIMEOUT, ROLE_UPDATE, CHANNEL_DELETE, OVERWRITE_UPDATE, GUILD_OWNER_TRANSFER, ...
    reason              VARCHAR(512),
    changes             JSONB,                                -- {before, after}
    created_at          TIMESTAMPTZ NOT NULL DEFAULT now()
);
CREATE INDEX idx_audit_guild_time ON audit_logs(guild_id, created_at DESC);
-- Tài khoản CSDL của ứng dụng chỉ có quyền INSERT/SELECT trên audit_logs (REVOKE UPDATE, DELETE);
-- job retention (8.6) chạy bằng tài khoản riêng.

CREATE TABLE platform_audit_logs (                            -- REQ-SAFE-02
    id                  BIGINT PRIMARY KEY,
    admin_id            BIGINT NOT NULL,
    action_type         VARCHAR(50) NOT NULL,
    target_type         VARCHAR(10) NOT NULL,
    target_id           BIGINT NOT NULL,
    report_id           BIGINT,
    details             JSONB,
    created_at          TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

#### 8.2.1. Giá trị `messages.type`

| Giá trị | Tên | Sinh bởi |
|---|---|---|
| 0 | DEFAULT | Tin nhắn thường (có thể có `reply_to_id`) |
| 1 | MEMBER_JOIN | REQ-SRV-04 |
| 2 | CHANNEL_PINNED | REQ-MSG-10 |
| 3 | CALL | REQ-DMVC-01/03 (`system_data`: `{call_id, result: missed|declined|ended, duration_s}`) |
| 4 | RECIPIENT_ADD | REQ-GDM-02 |
| 5 | RECIPIENT_REMOVE | REQ-GDM-02 |
| 6 | CHANNEL_NAME_CHANGE | REQ-GDM-02 |
| 7 | CHANNEL_ICON_CHANGE | REQ-GDM-02 |
| 8 | FOCUS_SESSION_START | REQ-STUDY-01 |
| 9 | FOCUS_SESSION_END | REQ-STUDY-01 (`system_data`: tổng phút, số người) |
| 10 | TASK_REMINDER | REQ-STUDY-06 (hiển thị trong kênh nếu người tạo bật tùy chọn) |

### 8.3. Message Store production (ScyllaDB — mục tiêu, không nghiệm thu)

```sql
CREATE TABLE messages (
    channel_id      BIGINT,
    bucket          INT,           -- floor(snowflake_timestamp_ms / (10 ngày)) — tính từ message_id
    message_id      BIGINT,
    author_id       BIGINT,
    type            SMALLINT,
    content         TEXT,
    nonce           TEXT,
    attachments     TEXT,          -- JSON [{id, filename, content_type, size, width, height, key}]
    embeds          TEXT,
    mention_user_ids SET<BIGINT>,
    mention_role_ids SET<BIGINT>,
    mention_everyone BOOLEAN,
    reply_to_id     BIGINT,
    system_data     TEXT,
    edited_at       TIMESTAMP,
    pinned_at       TIMESTAMP,
    deleted         BOOLEAN,
    PRIMARY KEY ((channel_id, bucket), message_id)
) WITH CLUSTERING ORDER BY (message_id DESC);

CREATE TABLE message_reactions (
    channel_id      BIGINT,
    message_id      BIGINT,
    emoji           TEXT,
    user_id         BIGINT,
    PRIMARY KEY ((channel_id, message_id), emoji, user_id)
);
```

- **Bucket** 10 ngày giới hạn kích thước partition. Phân trang `before=<id>`: bắt đầu từ bucket của `id`, đọc lùi từng bucket cho tới khi đủ `limit`; lưu `channels.first_message_id` để dừng khi đã tới bucket đầu tiên (tránh duyệt vô hạn bucket rỗng).
- **Hot partition:** bucket **không** chống được hot-write của một channel cực nóng (mọi ghi rơi vào bucket hiện tại). Biện pháp: rate limit (REQ-MSG-08, SRV-06), **request coalescing** cho truy vấn "tin mới nhất" trùng lặp, cache N tin mới nhất của kênh nóng trong Redis.
- Idempotency theo `nonce` chuyển sang Redis (`SET NX` 24h) vì ScyllaDB không có unique index phụ.

### 8.4. State Machines

#### 8.4.1. Cuộc gọi (`call_sessions.status`)

```
          create
            │
            ▼
       ┌─────────┐  accept (≥1 người nhận)  ┌────────┐  mọi người rời / end  ┌───────┐
       │ ringing │────────────────────────▶│ active │──────────────────────▶│ ended │
       └────┬────┘                          └────────┘                       └───────┘
            │ decline (DM 1-1)            │ timeout 45s không ai accept     │ caller hủy
            ▼                             ▼                                 ▼
       ┌──────────┐                  ┌────────┐                        ┌────────┐
       │ declined │                  │ missed │                        │ missed │
       └──────────┘                  └────────┘                        └────────┘
Trạng thái kết thúc: ended, missed, declined → sinh system message type=3.
```

#### 8.4.2. Quan hệ bạn bè

| Từ | Sự kiện | Tới | Ghi chú |
|---|---|---|---|
| (không có) | A gửi request | pending (requested_by=A) | REQ-FRIEND-01 |
| pending (by A) | B gửi request cho A | accepted | Tự chấp nhận |
| pending | B Accept | accepted | REQ-FRIEND-02 |
| pending | B Decline / A Cancel | (xóa dòng) | |
| accepted | A hoặc B hủy kết bạn | (xóa dòng) | |
| bất kỳ | A block B | (xóa dòng) + `user_blocks(A,B)` | REQ-FRIEND-03 |

#### 8.4.3. Focus Session (nhóm & solo)

```
 start ──▶ [running: phase = f(now − server_started_at − total_paused)] ──pause──▶ [paused]
                │  ▲                                                               │
                │  └──────────────────────────────── resume ◀─────────────────────┘
                │ stop / kênh trống 5 phút (nhóm) / quá 4 giờ không stop (solo)
                ▼
             [ended]
 Pha trong running: focus → short_break → focus → … (→ long_break sau mỗi N chu kỳ nếu bật)
 skip: nhảy tới đầu pha kế tiếp (server điều chỉnh server_started_at tương ứng, phát FOCUS_SESSION_UPDATE)
```

#### 8.4.4. Task

| Từ | Sự kiện | Tới | Ai được làm |
|---|---|---|---|
| (mới) | create | todo | `CREATE_TASKS` |
| todo | start | doing | người tạo, người phụ trách, `MANAGE_TASKS` |
| todo/doing | complete | done (`completed_at`) | như trên |
| done | reopen | todo | như trên |
| bất kỳ | delete | (soft-delete) | người tạo, `MANAGE_TASKS` |

#### 8.4.5. Tin nhắn phía app (outbox)

```
 soạn & bấm gửi ─▶ [pending] ── 201 / MESSAGE_CREATE khớp nonce ─▶ [sent]
                      │  ▲
       lỗi mạng/5xx/429│  │ retry (cùng nonce, backoff; 429 chờ Retry-After)
                      ▼  │
                   [retrying] ── quá 24h hoặc lỗi 4xx không retry được ─▶ [failed] ── user bấm Thử lại ─▶ [pending]
                                                                              └── user Xóa ─▶ (xóa khỏi outbox)
```

#### 8.4.6. Tài khoản (`users.account_status`)

```
 register ─▶ unverified ── verify email ─▶ active ── yêu cầu xóa ─▶ pending_deletion ── 14 ngày ─▶ deleted
      │ (7 ngày không verify → xóa)          ▲   │                     │ đăng nhập lại & hủy
      ▼                                      │   │ admin khóa           ▼
   (xóa)                                     │   ▼                   active
                                             └ suspended (hết hạn suspended_until)
```

### 8.5. Phân quyền (Permissions)

#### 8.5.1. Bảng bit

| Bit | Tên | Ý nghĩa | Mặc định `@everyone` |
|---|---|---|---|
| 0 | CREATE_INVITE | Tạo invite | ✔ |
| 1 | KICK_MEMBERS | Kick | |
| 2 | BAN_MEMBERS | Ban/unban | |
| 3 | ADMINISTRATOR | Mọi quyền, bỏ qua overwrite (không bỏ qua hierarchy) | |
| 4 | MANAGE_CHANNELS | Tạo/sửa/xóa channel, overwrite, bật Task Board, đổi `voice_mode`, `user_limit`, slow mode | |
| 5 | MANAGE_GUILD | Sửa thông tin guild, bật leaderboard | |
| 6 | ADD_REACTIONS | Thêm emoji mới vào reaction | ✔ |
| 7 | VIEW_AUDIT_LOG | Xem audit log | |
| 8 | VIEW_CHANNEL | Thấy channel, đọc tin mới | ✔ |
| 9 | SEND_MESSAGES | Gửi tin | ✔ |
| 10 | MANAGE_MESSAGES | Xóa tin người khác, ghim | |
| 11 | EMBED_LINKS | Tin có link preview | ✔ |
| 12 | ATTACH_FILES | Đính kèm file | ✔ |
| 13 | READ_MESSAGE_HISTORY | Đọc lịch sử | ✔ |
| 14 | MENTION_EVERYONE | Dùng `@everyone`/`@here`, mention mọi role | |
| 15 | CONNECT | Vào voice channel | ✔ |
| 16 | SPEAK | Nói trong voice | ✔ |
| 17 | VIDEO | Bật camera / screen share | ✔ |
| 18 | MUTE_MEMBERS | Server-mute | |
| 19 | DEAFEN_MEMBERS | Server-deafen | |
| 20 | MOVE_MEMBERS | Di chuyển/ngắt member khỏi voice, vào kênh đã đầy | |
| 21 | CHANGE_NICKNAME | Đổi nickname của mình | ✔ |
| 22 | MANAGE_NICKNAMES | Đổi nickname người khác | |
| 23 | MANAGE_ROLES | Quản lý role (giới hạn hierarchy) | |
| 24 | MODERATE_MEMBERS | Timeout | |
| 25 | MANAGE_STUDY_ROOMS | Điều khiển mọi Focus Session, xem Member Activity | |
| 26 | CREATE_TASKS | Tạo task trên Task Board | ✔ |
| 27 | MANAGE_TASKS | Sửa/xóa/đổi trạng thái mọi task | |
| 28 | MANAGE_EVENTS | Tạo/sửa Study Event | |
| 29–63 | (dự trữ) | | |

Hằng số trong code được định nghĩa **một lần** trong package TypeScript dùng chung (`@nezfocus/permissions`) cho cả backend và app.

#### 8.5.2. Thuật toán tính quyền

```ts
function computeBasePermissions(member, guild): bigint {
  if (guild.ownerId === member.userId) return ALL;
  let perms = roleById(guild.id /* @everyone */).permissions;
  for (const r of member.roles) perms |= r.permissions;
  if (perms & ADMINISTRATOR) return ALL;
  return perms;
}

function computeChannelPermissions(member, guild, channel): bigint {
  let perms = computeBasePermissions(member, guild);
  if (perms === ALL) return ALL;                      // Owner / Administrator

  // 1) overwrite của @everyone
  const ev = overwrite(channel, guild.id, ROLE);
  if (ev) { perms &= ~ev.deny; perms |= ev.allow; }

  // 2) overwrite của các role member đang có: gộp deny và allow rồi áp deny trước, allow sau
  let deny = 0n, allow = 0n;
  for (const r of member.roles) {
    const o = overwrite(channel, r.id, ROLE);
    if (o) { deny |= o.deny; allow |= o.allow; }
  }
  perms &= ~deny; perms |= allow;                     // allow của role thắng deny của role

  // 3) overwrite của riêng member (ưu tiên cao nhất)
  const m = overwrite(channel, member.userId, MEMBER);
  if (m) { perms &= ~m.deny; perms |= m.allow; }

  // 4) quyền ngầm định
  if (!(perms & VIEW_CHANNEL)) return 0n;             // không thấy kênh ⇒ không có quyền gì
  if (channel.type === TEXT && !(perms & SEND_MESSAGES)) {
    perms &= ~(ATTACH_FILES | EMBED_LINKS | MENTION_EVERYONE);
  }
  if (channel.type === VOICE && !(perms & CONNECT)) {
    perms &= ~(SPEAK | VIDEO | MUTE_MEMBERS | DEAFEN_MEMBERS | MOVE_MEMBERS);
  }

  // 5) timeout (REQ-ROLE-05)
  if (member.timeoutUntil && member.timeoutUntil > now()) {
    perms &= (VIEW_CHANNEL | READ_MESSAGE_HISTORY);
  }
  return perms;
}
// Channel có permissions_synced = true dùng overwrite của category cha (parent_id).
// DM/GROUP_DM không dùng thuật toán này: thành viên có quyền gửi/đọc/gọi; owner GROUP_DM có quyền quản lý nhóm.
```

**Hierarchy (REQ-ROLE-01/04/05):** `highest(member)` = `position` lớn nhất trong role của member. Actor (không phải Owner) chỉ được tác động lên role có `position < highest(actor)` và lên member có `highest(target) < highest(actor)`; chỉ được cấp các bit mà chính actor đang có.

#### 8.5.3. Quyền mặc định

- `@everyone`: các bit đánh ✔ ở 8.5.1.
- Template Study Server: thêm role "Trưởng nhóm" = `MANAGE_STUDY_ROOMS | MANAGE_TASKS | MANAGE_EVENTS | MANAGE_MESSAGES | MODERATE_MEMBERS | KICK_MEMBERS`, gán cho Owner; `thong-bao` có overwrite `@everyone` deny `SEND_MESSAGES`.

### 8.6. Chính sách lưu trữ dữ liệu (Data Retention)

| Dữ liệu | Thời hạn | Xử lý khi hết hạn |
|---|---|---|
| Tin nhắn, file | Tới khi bị xóa bởi user/moderator hoặc guild bị purge | Soft-delete → purge nội dung sau 30 ngày |
| Guild soft-deleted | 30 ngày | Purge toàn bộ (trừ audit log, report) |
| Tài khoản `pending_deletion` | 14 ngày | Xóa/ẩn danh PII (REQ-AUTH-10) |
| Tài khoản chưa xác thực | 7 ngày | Xóa |
| Audit log guild | ≥ 90 ngày (đồ án) / ≥ 1 năm (production) | Xóa bằng job retention |
| Report & bằng chứng | 1 năm sau khi đóng | Xóa |
| Refresh token hết hạn/thu hồi, auth token | 30 ngày | Xóa |
| Log ứng dụng có IP đầy đủ | 30 ngày | Xóa |
| Thống kê học tập | Tới khi xóa tài khoản | Xóa cùng tài khoản |
| Analytics event | 13 tháng | Xóa/tổng hợp ẩn danh |
| Attachment chưa gắn vào tin | 24 giờ | Xóa object |

### 8.7. CSDL cục bộ trên app (op-sqlite + Drizzle)

```sql
CREATE TABLE kv (key TEXT PRIMARY KEY, value TEXT NOT NULL);            -- settings cục bộ, cursor đồng bộ
CREATE TABLE users (id TEXT PRIMARY KEY, username TEXT, display_name TEXT, avatar_url TEXT, updated_at INTEGER);
CREATE TABLE guilds (id TEXT PRIMARY KEY, name TEXT, icon_url TEXT, owner_id TEXT, template TEXT, position INTEGER, json TEXT);
CREATE TABLE channels (id TEXT PRIMARY KEY, guild_id TEXT, parent_id TEXT, type INTEGER, name TEXT,
                       position INTEGER, voice_mode TEXT, task_board_enabled INTEGER, last_message_id TEXT, json TEXT);
CREATE TABLE roles (id TEXT PRIMARY KEY, guild_id TEXT, permissions TEXT, position INTEGER, json TEXT);
CREATE TABLE my_memberships (guild_id TEXT PRIMARY KEY, role_ids TEXT, nickname TEXT, timeout_until INTEGER);
CREATE TABLE overwrites (channel_id TEXT, target_id TEXT, target_type INTEGER, allow TEXT, deny TEXT,
                         PRIMARY KEY (channel_id, target_id, target_type));
CREATE TABLE cached_messages (id TEXT PRIMARY KEY, channel_id TEXT NOT NULL, author_id TEXT, type INTEGER,
                              content TEXT, json TEXT, created_at INTEGER);
CREATE INDEX idx_cached_messages_channel ON cached_messages(channel_id, id DESC);
CREATE TABLE outbox (nonce TEXT PRIMARY KEY, kind TEXT NOT NULL, channel_id TEXT, payload TEXT NOT NULL,
                     status TEXT NOT NULL, attempts INTEGER NOT NULL DEFAULT 0, next_attempt_at INTEGER, created_at INTEGER);
CREATE TABLE read_states (channel_id TEXT PRIMARY KEY, last_read_message_id TEXT, mention_count INTEGER);
CREATE TABLE relationships (user_id TEXT PRIMARY KEY, type INTEGER, json TEXT);
CREATE TABLE dm_channels (id TEXT PRIMARY KEY, type INTEGER, recipient_ids TEXT, last_message_id TEXT, json TEXT);
CREATE TABLE my_tasks (id TEXT PRIMARY KEY, guild_id TEXT, channel_id TEXT, title TEXT, status INTEGER, due_at INTEGER, json TEXT);
CREATE TABLE study_summary (local_date TEXT PRIMARY KEY, focused_minutes INTEGER, streak INTEGER);
```

- ID lưu dạng TEXT (Snowflake string). Bitmap quyền lưu TEXT, chuyển sang `BigInt` khi tính.
- Khi đăng xuất: xóa file CSDL (REQ-AUTH-07).

---

## 9. CHIẾN LƯỢC KIỂM THỬ & TRUY VẾT (TESTING STRATEGY & TRACEABILITY)

### 9.1. Các mức kiểm thử

| Mức | Phạm vi | Công cụ | Chạy khi |
|---|---|---|---|
| Unit (backend) | Service, Permission calculator, Study (tính pha, focused minutes, streak theo timezone), rate limiter | Jest | Mỗi PR |
| Unit (app) | Domain use case, sync engine, reducer Zustand, `computeFocusPhase` | Jest + `@testing-library/react-native` cho component | Mỗi PR |
| Integration (backend) | REST + PostgreSQL + Redis thật (container) | Jest + Testcontainers, Supertest | Mỗi PR |
| Contract | Response khớp OpenAPI; app dùng type sinh từ OpenAPI | `openapi-typescript` + kiểm tra schema trong test integration | Mỗi PR |
| Gateway | IDENTIFY/HEARTBEAT/RESUME/INVALID_SESSION, replay đúng thứ tự `s` | Test client `ws` tự viết | Mỗi PR |
| E2E mobile | Luồng UF-01…UF-07 trên simulator/emulator | **Maestro** | Hằng đêm + trước release |
| Thủ công trên thiết bị thật | Voice/call, notification nền, khóa màn hình, deep link, mạng yếu | Checklist 9.3 | Mỗi release |
| Load | Gateway & REST | **k6** | Trước buổi bảo vệ & trước pilot |
| Bảo mật | OWASP MASVS (mức L1) cho app, OWASP API Top 10 cho backend, `npm audit` | Checklist + công cụ quét | Trước release |
| Usability | 5 sinh viên thực hiện UF-02, UF-04, UF-06 | Quan sát + SUS | 1 lần trước pilot |

### 9.2. Bộ test trọng điểm

| Mã bộ test | Nội dung | REQ |
|---|---|---|
| TS-PERM | Ma trận ≥ 50 case: Owner, Administrator, @everyone deny/allow, nhiều role overwrite xung đột, member overwrite, thiếu VIEW_CHANNEL, timeout, category sync, hierarchy. **Cùng bộ dữ liệu chạy cho backend và thư viện app, kết quả phải trùng 100%.** | ROLE-01..08 |
| TS-AUTH | Đăng ký, xác thực, login throttle/CAPTCHA, rotation & reuse detection, reset mật khẩu, xóa tài khoản, giới hạn unverified | AUTH-* |
| TS-MSG | Gửi/sửa/xóa/reply/react/pin, nonce trùng, rate limit, markdown an toàn, SSRF link preview, upload (quá cỡ, sai MIME, EICAR) | MSG-* |
| TS-GW | Heartbeat timeout, RESUME trong/ngoài cửa sổ 60s, fanout chỉ tới người có quyền | MSG-07, NFR-AVAIL-03 |
| TS-FOCUS | Đồng bộ pha với đồng hồ máy lệch ±5 phút, pause/resume/skip, late join, quyền điều khiển, 1 session/kênh, qua nửa đêm, đổi timezone, mất heartbeat, solo stop trễ, trần 16h/ngày | STUDY-01/03/10/11/15 |
| TS-TASK | Quyền task, nhắc 24h/2h, task không gán ai chỉ nhắc người tạo, người phụ trách rời guild, Digest lọc theo quyền | STUDY-06/08 |
| TS-VOICE | Join/leave, server-mute ở SFU, TURN/TLS khi chặn UDP, đổi mạng wifi↔4G, khóa màn hình giữ audio | VC-* |
| TS-CALL | Gọi/nhận/từ chối/nhỡ 45s, người nhận bận, app foreground/nền, (v1.x) app bị kill | DMVC-* |
| TS-NOTIF | Push khi nền/kill, không push khi đang foreground thiết bị khác, xóa notification khi đọc ở thiết bị khác, ẩn nội dung, deep link từ notification | NOTIF-* |
| TS-OFFLINE | Mở app offline, xem cache, gửi tin vào outbox, có mạng lại tự gửi đúng thứ tự không trùng | MSG-09, NFR-MOB-09 |

### 9.3. Ma trận thiết bị & mạng

| Hạng mục | Tối thiểu cho MVP-Đồ án |
|---|---|
| iOS | 1 iPhone đời cũ chạy iOS 15/16 (vd iPhone 8/SE 2) + 1 iPhone đời mới (iOS mới nhất) |
| Android | Thiết bị tầm trung (RAM 3–4GB, Android 10–11) + 1 máy Android mới nhất; tổng ≥ 3 hãng: Samsung, Xiaomi, Oppo/Vivo (NFR-MOB-13) |
| Thiết bị tham chiếu hiệu năng | Android tầm trung RAM 4GB (đo NFR-MOB-01/03/04/06) |
| Mạng | Wifi gia đình, wifi trường (có thể chặn UDP), 4G của 3 nhà mạng lớn tại Việt Nam, mạng yếu giả lập (Network Link Conditioner / Android emulator: 3G, 1% và 5% packet loss) |
| Trạng thái app | Foreground, nền < 30s, nền > 5 phút, khóa màn hình, bị vuốt tắt (kill), máy khởi động lại |

### 9.4. Mục tiêu load test (k6)

| Kịch bản | Tải | Tiêu chí đạt |
|---|---|---|
| Gateway | 500 kết nối WebSocket đồng thời, heartbeat + 10 tin/giây vào 20 kênh | NFR-PERF-01 đạt, không rớt kết nối, CPU < 70% |
| REST | 50 req/s trộn (đọc lịch sử 60%, gửi tin 20%, task 10%, khác 10%) trong 10 phút | NFR-PERF-04 đạt, lỗi 5xx < 0.1% |
| Focus | 20 phòng × 10 người, heartbeat 60s, 5 lệnh pause/resume/phút | NFR-PERF-06 đạt |
| Voice | 10 phòng × 10 người audio trên 1 LiveKit node | Không suy giảm chất lượng, CPU node < 70% |

### 9.5. Ma trận truy vết (Traceability Matrix)

| Nhu cầu (Persona / Mục tiêu) | REQ | Bộ test | NFR liên quan |
|---|---|---|---|
| Self-learner: học không xao nhãng, cùng bạn bè | STUDY-01, STUDY-02, STUDY-11, VC-01, VC-08 | TS-FOCUS, TS-VOICE, UF-04 | PERF-05, PERF-06, MOB-06, MOB-11 |
| Self-learner: đo thời gian học, duy trì thói quen | STUDY-03, STUDY-10, STUDY-15, STUDY-04 | TS-FOCUS, UF-05 | PRIV-05, I18N-02 |
| Group Lead: tổ chức nhóm học | SRV-01, SRV-03, SRV-07, STUDY-05, ROLE-01..08 | TS-PERM, UF-02, UF-03 | USAB-01 |
| Group Lead: giao việc & nhắc deadline | STUDY-06, STUDY-08, NOTIF-02 | TS-TASK, UF-06 | MOB-09 |
| Group Lead: thấy ai "mất tích", lịch học cố định | STUDY-13, STUDY-12 | TS-FOCUS, TS-TASK (v1.x) | PRIV-05 |
| Casual: nắm nhanh deadline, ít thông báo phiền | STUDY-08, STUDY-09, NOTIF-03, NOTIF-04 | TS-NOTIF, TS-TASK | MOB-01 |
| Giao tiếp nhóm cơ bản | MSG-01..11, DM-01..03, GDM-01..03, FRIEND-01..04 | TS-MSG, TS-GW, TS-OFFLINE | PERF-01, PERF-03, AVAIL-03 |
| Gọi 1-1 | DMVC-01, DMVC-03, DMVC-04 | TS-CALL, UF-07 | PERF-05 |
| An toàn tài khoản | AUTH-01..13 | TS-AUTH | SEC-01..08 |
| An toàn cộng đồng | ROLE-05, ROLE-07, FRIEND-03, SAFE-01, SAFE-02 | TS-PERM, TS-MSG | SEC-05, PRIV-01 |
| Tuân thủ pháp lý & store | AUTH-10, AUTH-12, NFR-PRIV-* | TS-AUTH, checklist 9.6 | PRIV-01..07, MOB-10, MOB-11 |

> Quy ước mã test case: `TC-<REQ-ID>-<n>` (vd `TC-STUDY-01-3` ứng với AC3 của REQ-STUDY-01). Mỗi AC trong Mục 4 phải có ít nhất 1 test case; báo cáo độ phủ AC là một phần của báo cáo đồ án.

### 9.6. Definition of Done & Checklist phát hành

**Definition of Done cho một REQ:**
1. Mọi AC có test case tương ứng và đạt (tự động nếu khả thi).
2. Endpoint có OpenAPI đầy đủ; event Gateway mới được thêm vào danh sách 5.4.
3. Mã lỗi mới có trong Phụ lục 11.1 và chuỗi thông báo vi/en trong app.
4. Chạy được trên cả iOS và Android (trừ khi Platform Matrix 3.7 ghi khác).
5. Có accessibility label cho phần tử tương tác mới (NFR-A11Y-01).
6. Lint, typecheck, test CI xanh; được review bởi ≥ 1 thành viên khác.

**Checklist trước khi submit store / bảo vệ đồ án:**
- [ ] Chức năng xóa tài khoản trong app (REQ-AUTH-10); link Privacy Policy trong app và trên store.
- [ ] Chuỗi mô tả quyền (mic, camera, ảnh, thông báo) rõ ràng, tiếng Việt & tiếng Anh.
- [ ] Chỉ khai báo background mode/foreground service thực sự dùng (NFR-MOB-11); khai báo foreground service type trên Google Play Console.
- [ ] Tài khoản demo cho reviewer của store.
- [ ] Data safety (Google Play) và App Privacy (App Store) khai báo đúng dữ liệu thu thập.
- [ ] Rating độ tuổi phù hợp DEC-18.
- [ ] Sentry nhận được crash từ bản release (đã upload source map).
- [ ] Load test 9.4 đạt; diễn tập khôi phục backup (NFR-BACKUP-03).

---

## 10. BACKLOG (POST-MVP) — GÓI PREMIUM & TÍNH NĂNG MỞ RỘNG

> Các mục trong chương này **không thuộc MVP-Đồ án**, được đặc tả trước để không phải thiết kế lại từ đầu. Priority dùng thang: **P1** (làm ngay sau MVP) / **P2** (trung hạn) / **P3** (dài hạn, theo nhu cầu thực tế). Cột Release của mọi REQ trong chương này là `Backlog`.

### 10.1. Gói thuê bao Premium — "NezFocus+"

> Thanh toán trong app mobile cho quyền lợi số phải tuân thủ chính sách In-App Purchase của App Store/Google Play (R8) — **không** tích hợp cổng thanh toán ngoài cho các quyền lợi này trong app.

#### 10.1.1. Hạ tầng thanh toán & Entitlement

| REQ-ID | Tên chức năng | Mô tả chi tiết | Priority | Depends-on | Acceptance Criteria |
|---|---|---|---|---|---|
| REQ-PREM-00 ✏️ | Thanh toán qua In-App Purchase | Gói tháng/năm qua StoreKit (iOS) và Google Play Billing (Android); backend xác minh giao dịch và nhận thông báo gia hạn/hủy/hoàn tiền từ App Store Server Notifications và Google Real-time Developer Notifications. | P1 | AUTH-02 | **AC1:** Given user mua gói tháng thành công, Then trong ≤ 10 giây `subscriptions.status=active` và entitlement được cấp. **AC2 (âm):** Given biên nhận giả mạo, Then từ chối và không cấp entitlement. |
| REQ-PREM-01 | Entitlement Service | Lưu trạng thái premium (active/expired/grace_period), mọi module truy vấn qua cache Redis để bật/tắt tính năng theo thời gian thực. | P1 | PREM-00 | **AC1:** Given subscription hết hạn, Then mọi tính năng premium bị vô hiệu trong ≤ 1 phút, không cần đăng nhập lại. |
| REQ-PREM-02 | Grace Period & Downgrade an toàn | Thanh toán gia hạn thất bại → grace period theo cơ chế của store (tối thiểu 3 ngày); hết hạn thì nội dung premium mất hiệu ứng nhưng dữ liệu gốc giữ 30 ngày. | P2 | PREM-01 | **AC1:** Given hết grace period, Then avatar động hiển thị frame tĩnh đầu tiên; gia hạn lại trong 30 ngày thì khôi phục. |

#### 10.1.2. Quyền lợi cá nhân

| REQ-ID | Tên chức năng | Mô tả chi tiết | Priority | Depends-on | Acceptance Criteria |
|---|---|---|---|---|---|
| REQ-PREM-03 ✏️ | Avatar/Banner động (GIF) | Chỉ user Premium upload GIF (thống nhất với REQ-AUTH-06: bản thường chỉ ảnh tĩnh). | P2 | PREM-01, AUTH-06 | **AC1 (âm):** Given user thường upload GIF, Then `40015` kèm gợi ý nâng cấp. |
| REQ-PREM-04 ✏️ | Custom emoji dùng xuyên server | User Premium dùng custom emoji của **mọi guild mình đang là thành viên** ở bất kỳ kênh/DM nào. | P2 | BKLG-03 | **AC1:** Given user Premium là thành viên guild A có `:pepehappy:`, When chat ở guild B, Then gửi được emoji dạng ảnh. |
| REQ-PREM-05 | Giới hạn upload cao hơn | 25MB → 500MB/file cho user Premium. | P2 | PREM-01, MSG-04 | **AC1:** Given user Premium upload 300MB, Then được chấp nhận thay vì `40013`. |
| REQ-PREM-06 ✏️ | Screen share chất lượng cao | Screen share 1080p30 (thay vì 720p30 của REQ-VC-03) trên nền tảng hỗ trợ phát screen share. | P3 | VC-03 | **AC1:** Given user Premium Android share, Then negotiate tối đa 1080p nếu băng thông đủ, fallback 720p. |
| REQ-PREM-07 | Huy hiệu & trang trí hồ sơ | Badge Premium, khung avatar, theme profile. | P3 | PREM-01 | **AC1:** Given user chọn khung avatar, Then hiển thị đồng nhất trên iOS và Android. |
| REQ-PREM-08 ✏️ | Nâng giới hạn số Group DM | **MVP chưa giới hạn số Group DM mỗi user.** Nếu sau này áp giới hạn cho tài khoản thường (đề xuất 100 nhóm/user), user Premium được miễn. (v1.0 tham chiếu nhầm tới giới hạn 10 *thành viên* của REQ-GDM-03.) | P3 | PREM-01, GDM-01 | **AC1 (âm):** Given user thường đã ở 100 Group DM, When được thêm vào nhóm mới, Then `40053`. |
| REQ-PREM-09 | Lịch sử tìm kiếm không giới hạn | User thường tìm kiếm trong 12 tháng gần nhất; Premium không giới hạn. | P3 | SEARCH-01 | **AC1:** Given user Premium tìm tin > 12 tháng, Then có kết quả; user thường nhận gợi ý nâng cấp. |
| REQ-PREM-13 🆕 | Thống kê học tập nâng cao | Biểu đồ giờ học theo môn/guild/khung giờ, so sánh tuần, xuất CSV. | P2 | STUDY-15 | **AC1:** Given user Premium mở Thống kê nâng cao, Then thấy phân bổ phút học theo guild trong 12 tuần. |
| REQ-PREM-14 🆕 | Tóm tắt kênh bằng AI | Tóm tắt các tin chưa đọc của một kênh học tập (dùng mô hình ngôn ngữ; nội dung chỉ gửi đi khi user chủ động yêu cầu, có thông báo về xử lý dữ liệu). | P3 | PREM-01 | **AC1:** Given 200 tin chưa đọc ở `hoi-dap`, When bấm "Tóm tắt", Then nhận bản tóm tắt ≤ 10 gạch đầu dòng trong ≤ 15 giây. |
| REQ-PREM-15 🆕 | Solo Focus nâng cao | Chế độ chặn thông báo trong app khi Solo Focus, âm thanh nền (white noise, royalty-free), mục tiêu giờ học tuần. | P3 | STUDY-10 | **AC1:** Given đặt mục tiêu 10 giờ/tuần, Then tab Học hiển thị tiến độ theo tuần. |

#### 10.1.3. Server Boost

| REQ-ID | Tên chức năng | Mô tả chi tiết | Priority | Depends-on | Acceptance Criteria |
|---|---|---|---|---|---|
| REQ-PREM-10 | Boost Guild | User Premium tặng 1–2 lượt boost/tháng cho guild; đạt mốc 3/7/14 boost mở tier 1/2/3. Cột `guilds.boost_count`, `guilds.boost_tier`, `guilds.vanity_url_slug` được thêm bằng migration khi triển khai (không có trong schema MVP). | P2 | PREM-01 | **AC1:** Given guild đạt 3 boost, Then `boost_tier` = 1 và perk tier 1 có hiệu lực ngay cho mọi member. |
| REQ-PREM-11 | Perks theo Boost Tier | Tier 1: +50 emoji slot, voice 128 kbps. Tier 2: +100 emoji slot, voice 256 kbps, banner động, vanity URL `nezfocus.app/<slug>`. Tier 3: +250 emoji slot, voice 384 kbps, upload guild-wide 100MB. | P2 | PREM-10 | **AC1:** Given guild Tier 2 cấu hình vanity URL, Then `nezfocus.app/<slug>` mở preview invite như REQ-SRV-07. |
| REQ-PREM-12 | Huy hiệu "Booster" | Member đã boost có badge riêng và role tự động "Server Booster". | P3 | PREM-10 | **AC1:** Given user boost guild, Then có badge cạnh tên trong member list. |

### 10.2. Rà soát độ phủ so với Discord & đề xuất Backlog

> v1.0 tự đánh giá "✅ Đầy đủ" cho mọi nhóm lõi — nhận định này đã được bỏ vì còn thiếu nhiều chức năng (quên mật khẩu, xóa tài khoản, ghim tin…), nay đã bổ sung ở Mục 4. Bảng dưới liệt kê những gì **vẫn** chưa có.

| REQ-ID | Tên chức năng | Mô tả ngắn | Priority | Trạng thái v1.1 |
|---|---|---|---|---|
| REQ-BKLG-01 | Threads | Nhánh hội thoại con từ 1 tin nhắn, tự archive sau X ngày không hoạt động. | P1 | Backlog |
| REQ-BKLG-02 | Ghim tin nhắn | — | — | **Đã chuyển sang REQ-MSG-10 (MVP-Đồ án)** |
| REQ-BKLG-03 | Custom Emoji & Sticker theo Guild | Owner/Admin upload emoji riêng (slot theo boost tier). | P1 | Backlog |
| REQ-BKLG-04 | Webhooks | Hệ thống ngoài gửi tin tự động vào channel (vd thông báo từ GitHub của nhóm đồ án). | P2 | Backlog |
| REQ-BKLG-05 | Bot & Slash Commands | Ứng dụng bot OAuth2, slash command (`/pomodoro`, `/poll`). | P2 | Backlog |
| REQ-BKLG-06 | AutoMod | Lọc từ khóa/spam/link tự động theo guild. | P2 | Backlog |
| REQ-BKLG-07 | Stage Channels | Kênh hội trường Speaker/Audience (buổi chữa đề, AMA). | P3 | Backlog |
| REQ-BKLG-08 | Forum Channels | Mỗi post là 1 thread có tiêu đề & tag (phù hợp Q&A môn học). | P3 | Backlog |
| REQ-BKLG-09 | Server Discovery | Danh mục guild công khai, preview trước khi join. Tiền đề của REQ-STUDY-07. | P3 | Backlog |
| REQ-BKLG-10 | Announcement Channel + Follow | Crosspost thông báo sang guild khác. | P3 | Backlog |
| REQ-BKLG-11 | Onboarding & Membership Screening | Đồng ý nội quy trước khi chat, gợi ý kênh/role. | P2 | Backlog |
| REQ-BKLG-12 | Trust & Safety nền tảng đầy đủ | Hàng đợi kiểm duyệt nền tảng có SLA, phân loại tự động, kháng nghị. | P1 | **Một phần đã có** ở REQ-SAFE-01 (MVP) và REQ-SAFE-02 (v1.x); phần còn lại Backlog |
| REQ-BKLG-13 ✏️ | Custom Status | Trạng thái tùy chỉnh (text + emoji), vd "Đang ôn Giải tích 2 📚". Bỏ "Rich Presence" game/Spotify vì không phù hợp định vị. | P3 | Backlog |
| REQ-BKLG-14 | Read Receipts trong DM | "Đã xem" trong DM 1-1 (Discord không có; cân nhắc làm khác Discord, cần tùy chọn tắt vì quyền riêng tư). | P3 | Backlog (tùy chọn) |
| REQ-BKLG-15 🆕 | Đồng bộ lịch | Xuất deadline task & Study Event sang Google Calendar / lịch hệ thống (iCal feed hoặc thêm vào lịch máy). | P2 | Backlog |
| REQ-BKLG-16 🆕 | Thư viện tài liệu | Kênh `tai-lieu` có chế độ thư viện: duyệt file theo thư mục/tag, tìm theo tên. | P2 | Backlog |
| REQ-BKLG-17 🆕 | Bình chọn (Poll) | Tạo poll trong kênh, chọn 1/nhiều, hạn chót. | P2 | Backlog |
| REQ-BKLG-18 🆕 | Tích hợp LMS | Nhập danh sách lớp/bài tập từ Google Classroom/Moodle. | P3 | Backlog |
| REQ-BKLG-19 🆕 | Screen share trên iOS | Broadcast Upload Extension (phần iOS của REQ-VC-03, DEC-11). | P3 | Backlog |
| REQ-BKLG-20 🆕 | Web client / Desktop client | Đã loại khỏi phạm vi (DEC-01); xem xét lại khi có nhu cầu học trên máy tính. | P3 | Out of scope hiện tại |

---

## 11. PHỤ LỤC (APPENDIX)

### 11.1. Bảng mã lỗi (Error Codes)

> Mã giữ nguyên từ v1.0 để tương thích (không đổi nghĩa mã đã phát hành). Thông điệp hiển thị cho user do app ánh xạ từ `code`, song ngữ vi/en.

| Mã lỗi | HTTP | Tên | Ý nghĩa | Dùng ở |
|---|---|---|---|---|
| 40001 | 400 | ValidationError | Dữ liệu request không hợp lệ (kèm `errors` theo field). | Nhiều REQ |
| 40002 | 400 | TokenLinkInvalid | Link xác thực email/đặt lại mật khẩu/đổi email không hợp lệ, đã dùng hoặc hết hạn. | AUTH-08, AUTH-09, AUTH-01 |
| 40010 | 409 | OwnershipTransferRequired | Không thể xóa tài khoản khi còn sở hữu guild có member khác. | AUTH-10 |
| 40011 | 403 | MaxGuildsReached | Đã đạt giới hạn 100 guild. | SRV-01, SRV-04 |
| 40012 | 403 | OwnerCannotLeave | Owner phải chuyển quyền trước khi rời guild. | SRV-05 |
| 40013 | 413 | FileTooLarge | File vượt kích thước cho phép. | MSG-04, AUTH-06, PREM-05 |
| 40014 | 415 | UnsupportedFileType | Loại file không được hỗ trợ / nội dung không khớp định dạng. | MSG-04, AUTH-06 |
| 40015 | 403 | PremiumRequired | Tính năng yêu cầu NezFocus+ (Backlog). | PREM-03 |
| 40016 | 400 | InvalidTaskAssignee | Người phụ trách không phải member có quyền xem kênh. | STUDY-06 |
| 40029 | 429 | RateLimited | Vượt giới hạn tần suất; kèm `Retry-After`. | MSG-08, AUTH-02/08/13, SRV-03, FRIEND-01, STUDY-03, SAFE-01 |
| 40030 | 429 | SlowModeRateLimited | Đang trong thời gian chờ slow mode; kèm `retry_after`. (Tách khỏi 40029 ở v1.1.) | SRV-06 |
| 40041 | 410 | InviteInvalid | Invite hết hạn, hết lượt, bị thu hồi hoặc guild đã xóa. | SRV-03, SRV-05 |
| 40051 | 403 | VoiceChannelFull | Voice channel đã đạt `user_limit`. | VC-07 |
| 40052 | 403 | GroupDMFull | Group DM đã đủ 10 thành viên. | GDM-03 |
| 40053 | 403 | MaxGroupDMsReached | Đạt giới hạn số Group DM của tài khoản thường (Backlog). | PREM-08 |
| 40100 | 401 | TokenInvalid | Access/refresh token không hợp lệ hoặc hết hạn. | AUTH-04 |
| 40101 | 401 | InvalidMFACode | Sai mã TOTP / recovery code, hoặc `mfa_ticket` đã bị hủy. | AUTH-03, AUTH-11 |
| 40102 | 401 | InvalidCredentials | Sai thông tin đăng nhập hoặc mật khẩu xác nhận. | AUTH-02, SRV-05 |
| 40103 | 401 | RefreshTokenReused | Refresh token đã dùng bị dùng lại; toàn bộ phiên bị thu hồi. (v1.0 dùng 40901 — đã sửa.) | AUTH-04 |
| 40104 | 401 | MFARequired | Cần bước 2FA; kèm `mfa_ticket`. | AUTH-02, AUTH-03 |
| 40105 | 401 | CaptchaRequired | Cần giải CAPTCHA trước khi thử đăng nhập tiếp. | AUTH-02 |
| 40301 | 403 | InsufficientRoleHierarchy | Vi phạm role hierarchy. | ROLE-01, ROLE-04, ROLE-05 |
| 40302 | 403 | Forbidden | Không đủ quyền thực hiện hành động. | Nhiều REQ |
| 40303 | 403 | CannotMessageUser | Không thể gửi tin/lời mời/cuộc gọi tới user này. **Dùng chung** cho: bị block, người nhận tắt DM từ thành viên server, không chung guild — để không lộ lý do (thay thế `40303 BlockedByRecipient` và `40304 DMNotAllowed` của v1.0). | FRIEND-01, FRIEND-03, DM-02, DMVC-01 |
| 40304 | — | (Ngừng sử dụng) | Gộp vào 40303 ở v1.1. Không tái sử dụng mã này. | — |
| 40305 | 403 | EmailNotVerified | Tài khoản chưa xác thực email. | AUTH-13 |
| 40306 | 403 | BannedFromGuild | User bị ban khỏi guild. | SRV-04, ROLE-07 |
| 40307 | 403 | MemberTimedOut | User đang bị timeout; kèm `timeout_until`. | ROLE-05 |
| 40308 | 403 | AccountSuspended | Tài khoản bị khóa bởi Platform Admin; kèm thời hạn. | SAFE-02 |
| 40401 | 404 | NotFound | Không tìm thấy tài nguyên, hoặc tài nguyên tồn tại nhưng user không được phép biết. | ROLE-03, SAFE-01, SAFE-02 |
| 40901 | 409 | Conflict | Xung đột dữ liệu (email/username đã tồn tại…). | AUTH-01, AUTH-09 |
| 40902 | 409 | FriendRequestAlreadyPending | Đã có lời mời kết bạn đang chờ. | FRIEND-01 |
| 40903 | 409 | FocusSessionAlreadyActive | Kênh đã có Focus Session đang chạy, hoặc user đang ở phiên Focus khác. | STUDY-01, STUDY-10 |
| 40904 | 409 | MaxPinsReached | Kênh đã có 50 tin ghim. | MSG-10 |
| 40905 | 409 | MaxReactionsReached | Tin nhắn đã có 20 emoji reaction khác nhau. | MSG-05 |
| 40906 | 409 | AccountLinkRequired | Email OAuth trùng tài khoản hiện có; cần liên kết. | AUTH-12 |
| 40907 | 409 | IdempotencyKeyReused | `Idempotency-Key` đã dùng cho request có nội dung khác. | MSG-09 |
| 40908 | 409 | AlreadyFriends | Hai người đã là bạn bè. | FRIEND-01 |
| 42601 | 426 | UpgradeRequired | Phiên bản app thấp hơn phiên bản tối thiểu, cần cập nhật. | 5.2, NFR-MOB-12 |
| 50000 | 500 | InternalError | Lỗi hệ thống không xác định (kèm `request_id`). | — |
| 50301 | 503 | ServiceUnavailable | Dịch vụ phụ thuộc (LiveKit, Meilisearch, email) tạm thời không khả dụng. | NFR-AVAIL-05 |

### 11.2. Gateway Close Codes

| Code | Tên | Ý nghĩa | Client nên |
|---|---|---|---|
| 4000 | UnknownError | Lỗi không xác định | Reconnect + RESUME |
| 4001 | UnknownOpcode | Gửi opcode không hợp lệ | Sửa client; reconnect |
| 4002 | DecodeError | Payload không phải JSON hợp lệ | Sửa client; reconnect |
| 4003 | NotAuthenticated | Gửi op trước khi IDENTIFY | Reconnect + IDENTIFY |
| 4004 | AuthenticationFailed | Token không hợp lệ hoặc phiên bị thu hồi (REQ-AUTH-04/07) | **Không** reconnect; refresh token hoặc về màn hình đăng nhập |
| 4005 | AlreadyAuthenticated | IDENTIFY hai lần | Sửa client |
| 4007 | InvalidSeq | `seq` trong RESUME không hợp lệ | IDENTIFY mới |
| 4008 | RateLimited | Gửi quá nhiều frame (> 120 frame/60s) | Reconnect sau backoff |
| 4009 | SessionTimedOut | Session hết hạn resume | IDENTIFY mới |
| 4010 | UnsupportedVersion | Phiên bản giao thức/app không còn hỗ trợ | Hiển thị màn hình cập nhật app |
| 4011 | AccountUnavailable | Tài khoản bị khóa/xóa | Về màn hình đăng nhập, hiển thị lý do |

### 11.3. Bảng thuật ngữ mở rộng (Glossary)

| Thuật ngữ | Giải thích |
|---|---|
| Simulcast | Gửi đồng thời nhiều chất lượng của cùng một luồng video để SFU chọn layer phù hợp cho từng người nhận. |
| Adaptive stream / Dynacast | Tính năng LiveKit: chỉ nhận/gửi layer video cần thiết theo kích thước hiển thị và việc có ai đang xem. |
| Soft-delete | Đánh dấu đã xóa (`deleted_at`) thay vì xóa vật lý. |
| Sliding-window Rate Limit | Giới hạn tần suất theo cửa sổ thời gian trượt, chính xác hơn fixed-window ở biên. |
| Request Coalescing | Gộp nhiều request giống nhau gần như đồng thời thành 1 lần xử lý thực tế. |
| Config plugin (Expo) | Hàm chỉnh sửa project native (Info.plist, AndroidManifest, Gradle…) lúc prebuild. |
| Foreground service | Dịch vụ Android chạy khi app ở nền, bắt buộc hiển thị notification cố định. |
| Full-screen intent | Notification Android có thể mở toàn màn hình (dùng cho cuộc gọi đến). |
| Outbox | Hàng đợi cục bộ các thao tác tạo mới chưa gửi được lên server. |
| Clock offset | Độ lệch giữa đồng hồ thiết bị và đồng hồ server, dùng để đồng bộ Focus timer. |
| Cohort Analysis | Phân tích hành vi theo nhóm user có cùng thời điểm tham gia, dùng đo retention. |

### 11.4. Tài liệu & liên kết dự án

| Tài liệu | Vị trí | Trạng thái |
|---|---|---|
| Thiết kế UI (Figma) | *(điền link trước sprint UI đầu tiên)* | Cần bổ sung |
| Repository mobile & backend | *(điền link)* | Cần bổ sung |
| OpenAPI spec | Sinh tự động tại `/api/docs` của môi trường dev | Theo DoD 9.6 |
| Privacy Policy & Điều khoản | `https://nezfocus.app/privacy`, `https://nezfocus.app/terms` | Cần soạn trước pilot |
| Runbook sự cố & khôi phục | Thư mục `docs/runbooks` trong repo | Cần soạn trước pilot |

### 11.5. Tổng hợp thay đổi so với v1.0 (sửa mâu thuẫn & thiếu sót)

| # | Vị trí v1.0 | Vấn đề ở v1.0 | Xử lý ở v1.1 |
|---|---|---|---|
| 1 | Mục lục | Không có Mục 8 (nhảy 7 → 9) | Đánh số liên tục 0–11. |
| 2 | 2.5.3 | Tham chiếu REQ-STUDY-08 cho "Study Server" | Sửa thành REQ-STUDY-05 (Mục 2.3.2). |
| 3 | 3.1 | Văn bản "3 cột" nhưng sơ đồ 4 cột | Bỏ layout web/desktop; Mục 3 viết lại cho mobile (DEC-01). |
| 4 | 1.2, 6.2, 6.3 | Mobile ghi React Native, Flutter lẫn Kotlin/Realm | Chốt React Native + Expo (DEC-02), Mục 7.3 viết lại. |
| 5 | REQ-AUTH-06 vs REQ-PREM-03 | Mọi user được upload GIF vs GIF là Premium | MVP chỉ ảnh tĩnh; GIF thuộc Premium. |
| 6 | REQ-FRIEND-03 | Trả `BlockedByRecipient` làm lộ việc bị block | Gộp thành `40303 CannotMessageUser` dùng chung với lỗi quyền riêng tư. |
| 7 | REQ-AUTH-01 vs REQ-FRIEND-01 | Username unique và discriminator cùng tồn tại | Bỏ discriminator (DEC-03). |
| 8 | REQ-SRV-05 | Chuyển Owner yêu cầu 2FA trong khi 2FA không bắt buộc; Owner cũ bị hạ role do người khác chỉ định | Xác nhận bằng mật khẩu (hoặc TOTP nếu bật); Owner cũ giữ role. |
| 9 | REQ-SRV-02 vs REQ-STUDY-05 | Tên kênh slug-safe mâu thuẫn tên voice có dấu | Slug chỉ áp cho text channel và cho phép chữ tiếng Việt có dấu. |
| 10 | REQ-STUDY-07 | Phụ thuộc REQ-BKLG-09 (Backlog) nhưng không nói rõ release | REQ-STUDY-07 chuyển Backlog cùng BKLG-09. |
| 11 | REQ-STUDY-08 vs 06 | Digest (Should) phụ thuộc Task Board (Could) | Task Board nâng lên Should, cả hai ở MVP-Đồ án. |
| 12 | REQ-PREM-08 | Nhắc giới hạn "10 group DM" không tồn tại | Viết lại: MVP không giới hạn số Group DM; giới hạn tương lai được mô tả rõ. |
| 13 | NFR-SEC-05 vs `audit_logs` | Audit log immutable nhưng bị xóa cascade theo guild | Bỏ FK/cascade; thu hồi quyền UPDATE/DELETE. |
| 14 | Mục 11.1 | Thiếu 40902, 40303, 40304, 40052 và mã khóa đăng nhập/2FA; refresh reuse dùng 409 | Bảng mã lỗi đầy đủ; refresh reuse = `40103` (401). |
| 15 | 2.5.2 | `ManageStudyRooms` không được định nghĩa; nhu cầu "mất tích", "lịch cố định" không có REQ | Thêm bit `MANAGE_STUDY_ROOMS` (8.5.1), REQ-STUDY-12, REQ-STUDY-13. |
| 16 | 10.1 | Tự chấm "✅ Đầy đủ" | Bỏ; bổ sung REQ còn thiếu vào Mục 4. |
| 17 | REQ-AUTH-02 | Khóa cứng tài khoản sau 5 lần sai ⇒ có thể bị lợi dụng để khóa tài khoản người khác | Throttle theo (tài khoản, IP) + CAPTCHA. |
| 18 | REQ-ROLE-03 | Thứ tự ưu tiên overwrite quá giản lược | Thuật toán đầy đủ ở 8.5.2 + bảng bit 8.5.1. |
| 19 | 6.3.6 vs 6.3.3 (v1.0) | Giờ học chỉ tính khi có heartbeat, nhưng socket bị suspend khi nền ⇒ mất giờ học | DEC-06, REQ-STUDY-15. |
| 20 | Schema `messages` | `reactions MAP<emoji,count>` không biết ai react; thiếu `type`, `nonce`; attachment chỉ có URL | Bảng `message_reactions`, cột `type`/`nonce`/`system_data`, bảng `attachments`. |
| 21 | Schema | Thiếu `read_states`, `guild_bans`, refresh token, push token, settings (timezone, privacy), DM pair unique, 1 Focus Session/kênh | Bổ sung đầy đủ (8.2). |
| 22 | Mục 7.2 v1.0 | Cho rằng bucket theo thời gian chống hot partition | Ghi rõ bucket chỉ giới hạn kích thước partition; biện pháp hot-write ở 8.3. |
| 23 | NFR v1.0 | Mục tiêu production (99.9%, 100k kết nối) dùng làm tiêu chí đồ án | Tách cột Mục tiêu đồ án / Production (Mục 6). |
| 24 | Toàn tài liệu | Thiếu API/Gateway spec, state machine, test strategy, KPI, rủi ro, pháp lý | Thêm Mục 2.6–2.9, 5, 8.4, 9. |
| 25 | 2.4 / 5.3 v1.0 | Trùng lặp chỉ số latency/TLS | Chỉ số định lượng chỉ còn ở Mục 6. |
| 26 | 1.4 | IEEE 830 (đã bị thay thế), nguồn blog dùng như căn cứ chính | Chỉ dẫn ISO/IEC/IEEE 29148; phân loại độ tin cậy nguồn. |
| 27 | REQ-GDM-03 | "Mesh/SFU nhẹ" không rõ ràng | Dùng chung LiveKit SFU. |
| 28 | REQ-DMVC-01 | CallKit/PushKit bắt buộc ở MVP, rủi ro cao | Tách REQ-DMVC-04 sang v1.x (DEC-07). |
| 29 | REQ-STUDY-02 | Nhạc lo-fi chung chưa rõ nguồn và bản quyền | Tách REQ-STUDY-14 sang Backlog (DEC-09). |
| 30 | Mục 3.2 v1.0 | Dùng mã màu của Discord | Bộ design token riêng (3.3). |

---

*Hết tài liệu SRS — NezFocus v1.1 (Approved – Baseline v1.1, 29/09/2026)*
