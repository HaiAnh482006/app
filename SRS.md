# SOFTWARE REQUIREMENTS SPECIFICATION (SRS)
# DỰ ÁN: STUDELY — NỀN TẢNG CHAT/VOICE REAL-TIME CHO SINH VIÊN
### (Kiến trúc lõi theo mô hình Discord Clone, định vị sản phẩm: Study/Focus Server)

**Phiên bản:** 1.0
**Ngày phát hành:** 15/09/2026
**Chuẩn tham chiếu:** IEEE 830-1998 / ISO/IEC/IEEE 29148:2018
**Đối tượng sử dụng:** Backend Team, Frontend Team, DevOps/SRE Team, QA Team, Product Owner

---

## MỤC LỤC

1. Giới thiệu (Introduction)
2. Tổng quan hệ thống (Overall Description) — bao gồm 2.5 Định hướng sản phẩm: Study/Focus Server cho Sinh viên
3. UX/UI & General Behavior
4. Yêu cầu chức năng chi tiết (Functional Requirements) — bao gồm 4.9 Study/Focus Mode
5. Yêu cầu phi chức năng (Non-Functional Requirements)
6. Kiến trúc hệ thống & Tech Stack
7. Cơ sở dữ liệu Schema
9. Backlog — Gói Premium (Nitro-equivalent) & Server Boost
10. Rà soát độ phủ tính năng so với Discord (Gap Analysis)
11. Phụ lục (Appendix)

> *(Số thứ tự 8 đã được gộp vào mục 11 — Phụ lục, để giữ nguyên đánh số REQ-ID gốc trong tài liệu đã chia sẻ trước đó.)*

---

## 1. GIỚI THIỆU (INTRODUCTION)

### 1.1. Mục đích (Purpose)

Tài liệu này đặc tả đầy đủ các yêu cầu chức năng và phi chức năng cho hệ thống **Studely** — một nền tảng giao tiếp cộng đồng theo mô hình server/guild, hỗ trợ nhắn tin văn bản real-time, gọi thoại/video qua WebRTC, và hệ thống phân quyền (RBAC) phức tạp tương tự Discord. Tài liệu đóng vai trò là nguồn tham chiếu thống nhất (single source of truth) cho các đội Backend, Frontend, DevOps và QA trong suốt vòng đời phát triển sản phẩm (SDLC), từ giai đoạn thiết kế, triển khai đến kiểm thử và vận hành.

> **Định hướng sản phẩm:** Bộ khung ở Mục 4.1–4.8 (Auth, Guild/Channel, RBAC, Messaging, Voice/Video, Search/Notification, Friends/DM, Group DM/Call) là **lõi kỹ thuật bắt buộc** — triển khai trước, đúng như 1 Discord clone tiêu chuẩn. Sau khi lõi hoàn thiện, sản phẩm định vị vào nhóm đối tượng cụ thể là **sinh viên/người học** thông qua bộ tính năng bổ sung ở **Mục 4.9 — Study/Focus Mode**, biến "Guild" từ khái niệm chung chung thành "Study Server" (lớp học, nhóm học, CLB học thuật). Chi tiết định vị sản phẩm xem Mục 2.5.

### 1.2. Phạm vi (Scope)

Studely là một hệ thống đa nền tảng (cross-platform) bao gồm:

- **Web Client:** xây dựng trên Next.js/React, hỗ trợ Progressive Web App (PWA).
- **Desktop Client:** đóng gói qua Tauri hoặc Electron cho Windows/macOS/Linux.
- **Mobile Client:** xây dựng bằng Flutter hoặc React Native cho iOS/Android.
- **Backend Services:** kiến trúc microservices xử lý Auth, Gateway (WebSocket), Guild/Channel Management, Messaging Fanout, Voice/Video Signaling (SFU), Search, Notification.

Hệ thống KHÔNG bao gồm trong phạm vi phiên bản **MVP 1.0** (out of scope cho bản đầu, nhưng được đặc tả sẵn ở **Mục 9 — Backlog** để triển khai ở phase sau): livestream tới nền tảng thứ ba (YouTube/Twitch simulcast), gói thuê bao trả phí (Nitro-equivalent) và các tính năng mở rộng khác gắn với nó, AR/VR overlay.

### 1.3. Định nghĩa, từ viết tắt và thuật ngữ (Definitions, Acronyms, Abbreviations)

| Thuật ngữ | Giải thích |
|---|---|
| **SFU** (Selective Forwarding Unit) | Kiến trúc media server chuyển tiếp có chọn lọc các luồng RTP từ người gửi tới nhiều người nhận mà không transcode, giúp giảm tải CPU server so với MCU. |
| **MCU** (Multipoint Control Unit) | Kiến trúc media server giải mã, trộn (mix) và mã hóa lại luồng audio/video thành một luồng duy nhất gửi tới từng client; tốn CPU cao hơn SFU nhưng giảm băng thông upload từ server. |
| **Gateway** | Lớp dịch vụ quản lý kết nối WebSocket persistent giữa client và server, chịu trách nhiệm heartbeat, session resume, và phân phối sự kiện (event dispatch). |
| **RBAC** (Role-Based Access Control) | Mô hình kiểm soát truy cập dựa trên vai trò (role), trong đó mỗi role mang một tập quyền (permission bitmap) và được gán cho user theo guild. |
| **Guild** | Thuật ngữ nội bộ chỉ một "Server" — đơn vị tổ chức cao nhất chứa Category, Channel, Role, Member. |
| **Permission Overwrite** | Cơ chế ghi đè quyền ở cấp độ channel (allow/deny bitmap) cho một role hoặc một member cụ thể, có độ ưu tiên cao hơn quyền guild-level. |
| **Fanout** | Quá trình phân phối một sự kiện (vd: tin nhắn mới) tới toàn bộ client đang subscribe kênh/guild liên quan. |
| **VAD** (Voice Activity Detection) | Thuật toán phát hiện tín hiệu giọng nói trong luồng audio để tối ưu băng thông và hiển thị chỉ báo "đang nói". |
| **Presence** | Trạng thái hiện diện của user: Online, Idle, Do Not Disturb (DND), Invisible/Offline. |
| **Snowflake ID** | Định dạng ID 64-bit duy nhất toàn cục, mã hóa timestamp + worker ID + sequence, dùng làm khóa chính cho hầu hết entity (tương tự Twitter Snowflake/Discord ID). |
| **Cursor-based Pagination** | Kỹ thuật phân trang dùng con trỏ (ID/timestamp) thay vì OFFSET, tối ưu hiệu năng trên tập dữ liệu lớn và tránh trùng lặp khi dữ liệu thay đổi liên tục. |
| **Hot Partition** | Tình trạng một partition trong hệ CSDL phân tán nhận tải ghi/đọc vượt trội so với các partition khác, gây nghẽn cổ chai. |
| **JWT** (JSON Web Token) | Chuẩn token dùng để xác thực stateless giữa client và server. |
| **WSS** | WebSocket Secure — giao thức WebSocket chạy trên TLS. |

### 1.4. Tài liệu tham khảo (References)

| # | Nguồn | Nội dung áp dụng |
|---|---|---|
| R1 | Mẫu cấu trúc SRS chuẩn công nghiệp — qndev.github.io/resources/SRS.pdf | Cấu trúc mục lục, định dạng bảng requirement, acceptance criteria. |
| R2 | Discord Engineering Blog — "How Discord Stores Trillions of Messages" (discord.com/blog) | Chiến lược chuyển từ Cassandra sang ScyllaDB, kỹ thuật giảm thiểu hot partition, request coalescing cho message store. |
| R3 | Discord Engineering Blog — "How Discord Creates Insights from Trillions of Data Points" (discord.com/blog) | Kiến trúc data pipeline với BigQuery, Airflow DAGs, phân tích cohort cho analytics service. |
| R4 | "The Genius Architecture Behind Discord's Voice Chat" — Medium/@sohail_saifi | Mô hình Global Voice Gateway phân tán theo khu vực địa lý, cơ chế thích ứng jitter/packet loss. |
| R5 | Compile Ready — System Design: Discord (compileready.com) | Phân rã thành phần hệ thống ở tầm nhìn cao (high-level), luồng dữ liệu client-server. |
| R6 | Coditioning — Discord SWE System Design Guide (coditioning.com) | Cơ chế chat fanout, sliding-window rate limiting, quy trình moderation. |

> **Ghi chú áp dụng:** Các pattern từ R2 (ScyllaDB, hot partition mitigation) được áp dụng vào mục 6 (kiến trúc) và mục 7 (schema Messages). Pattern từ R4 (Global Voice Gateway) được áp dụng vào REQ-VC-*. Pattern rate limiting từ R6 được áp dụng vào mục 5 (NFR-SEC).

---

## 2. TỔNG QUAN HỆ THỐNG (OVERALL DESCRIPTION)

### 2.1. Bối cảnh sản phẩm (Product Perspective)

Studely là hệ thống độc lập (không phụ thuộc nền tảng thứ ba để vận hành lõi), triển khai theo kiến trúc microservices, giao tiếp nội bộ qua gRPC/message queue, và giao tiếp với client qua REST API (thao tác CRUD) kết hợp WebSocket Gateway (sự kiện real-time).

### 2.2. Đối tượng người dùng (User Personas)

| Persona | Mô tả | Mục tiêu chính | Quyền hạn tiêu biểu |
|---|---|---|---|
| **Member** | Người dùng thông thường tham gia guild qua invite link. | Nhắn tin, tham gia voice channel, tùy chỉnh profile cá nhân. | Send Messages, Connect Voice, Read Message History (tùy channel overwrite). |
| **Moderator** | Được Owner/Admin gán role có quyền kiểm duyệt. | Duy trì trật tự cộng đồng, xử lý vi phạm. | Kick Members, Ban Members, Manage Messages, Mute/Deafen (voice), Timeout. |
| **Owner** | Người tạo guild, có toàn quyền tối thượng. | Quản trị toàn diện guild: cấu trúc, role, tích hợp, xóa guild. | Administrator (bao trùm mọi quyền), Manage Roles, Manage Server, Transfer Ownership. |

### 2.3. Giả định và phụ thuộc (Assumptions & Dependencies)

**Giả định:**
- Người dùng có kết nối Internet ổn định tối thiểu 1 Mbps cho voice chat chất lượng thấp, 3 Mbps cho video 720p.
- Trình duyệt client hỗ trợ WebRTC chuẩn (Chrome/Firefox/Safari phiên bản gần đây) và WebSocket.
- Đội DevOps có sẵn hạ tầng Kubernetes hoặc tương đương để triển khai microservices có khả năng auto-scale.

**Phụ thuộc:**
- Dịch vụ SFU bên thứ ba/mã nguồn mở (LiveKit hoặc Mediasoup) cho xử lý media WebRTC.
- Đối tượng lưu trữ tương thích S3 (MinIO tự host hoặc AWS S3) cho file đính kèm và avatar.
- Redis Cluster cho Pub/Sub liên Gateway-node và caching session/presence.
- CDN cho phân phối static asset (avatar, emoji, sticker).

### 2.4. Ràng buộc thiết kế (Design Constraints)

| Hạng mục | Ràng buộc |
|---|---|
| **Độ trễ (Latency)** | Round-trip time (RTT) cho gửi/nhận tin nhắn text phải < 100ms trong cùng khu vực (region); độ trễ audio voice chat mục tiêu < 150ms end-to-end. |
| **Đồng thời (Concurrency)** | Gateway cluster phải hỗ trợ tối thiểu 50,000 kết nối WebSocket đồng thời/node, có khả năng horizontal scale không giới hạn qua Redis Pub/Sub làm message broker liên node. |
| **Khả năng tương thích ngược** | Giao thức Gateway phải hỗ trợ session resume (không mất sự kiện) khi client mất kết nối tạm thời dưới 60 giây. |
| **Bảo mật truyền tải** | Toàn bộ giao tiếp client-server bắt buộc qua TLS 1.3 (HTTPS/WSS), không cho phép fallback plaintext. |
| **Đa vùng địa lý** | Voice Gateway phải triển khai theo mô hình phân tán đa khu vực (multi-region edge) để tối thiểu hóa RTT theo vị trí địa lý người dùng, tương tự mô hình Global Voice Gateway của Discord. |

### 2.5. Định hướng sản phẩm: "Study/Focus Server" cho Sinh viên

#### 2.5.1. Vì sao chọn hướng này

Discord bản gốc là nền tảng đa mục đích (gaming, cộng đồng, công việc...) — không tối ưu riêng cho việc học. Studely giữ nguyên toàn bộ khung kỹ thuật (Guild/Channel/RBAC/Voice/Messaging) vì đây là bài toán kiến trúc cốt lõi của đồ án, nhưng **định vị use-case chính** là nơi sinh viên/nhóm học tự tổ chức: lớp học phần, nhóm đồ án, CLB học thuật, nhóm ôn thi.

#### 2.5.2. Persona bổ sung (lớp trên Member/Moderator/Owner ở mục 2.2)

| Persona | Bối cảnh sử dụng | Nhu cầu chính | Ánh xạ vào role kỹ thuật |
|---|---|---|---|
| **Sinh viên tự học (Self-learner)** | Tham gia 1 "Study Server" ôn thi cùng bạn bè, join phòng học im lặng mỗi tối. | Không bị xao nhãng, đo được mình đã học bao lâu, thấy bạn bè cũng đang học (động lực xã hội). | Member trong guild, dùng Study/Focus Mode. |
| **Trưởng nhóm học/CLB (Group Lead)** | Quản lý 1 guild là nhóm đồ án hoặc CLB, cần tổ chức phòng học theo ca, giao việc, theo dõi tiến độ thành viên. | Công cụ nhắc deadline, thấy được ai đang "mất tích" (không hoạt động), tạo phòng học có lịch cố định. | Moderator/Owner + quyền `ManageStudyRooms`. |
| **Sinh viên bận rộn (Casual/on-the-go)** | Chỉ vào app lúc rảnh, chủ yếu qua mobile, cần nắm nhanh deadline sắp tới và join nhanh 1 phòng học đang có bạn bè. | Notification thông minh (chỉ nhắc việc quan trọng), UI mobile tối giản để check nhanh. | Member, chủ yếu dùng mobile client (đúng trọng tâm kiến trúc App ở Mục 6.3). |

#### 2.5.3. Bảng khác biệt hóa so với Discord gốc

| Khía cạnh | Discord gốc | Studely (định hướng sinh viên) |
|---|---|---|
| Đơn vị tổ chức | "Server" chung chung (game, cộng đồng...) | "Study Server" — có thể gắn nhãn theo môn học/nhóm đồ án ngay khi tạo (REQ-STUDY-08) |
| Voice channel | Chỉ để nói chuyện | Có thể bật **Focus Mode** (Pomodoro đồng bộ, im lặng mặc định) — xem 4.9 |
| Không có khái niệm "tiến độ học" | — | Study Streak, thống kê giờ học cá nhân/nhóm (REQ-STUDY-03/04) |
| Không có deadline/task | — | Task board gắn theo channel (REQ-STUDY-06) |
| Thông báo | Tất cả ngang hàng | Ưu tiên nhắc deadline & mời vào phòng học hơn tin nhắn phiếm (REQ-STUDY-09) |

---

## 3. UX/UI & GENERAL BEHAVIOR

### 3.1. Bố cục tổng thể (Layout)

Giao diện chính tuân theo layout **3 cột** cố định trên Web/Desktop, thu gọn thích ứng (responsive collapse) trên Mobile:

```
┌───────────┬──────────────────────┬──────────────────────┬───────────────┐
│  Cột 1     │   Cột 2               │   Cột 3 (chính)       │  Cột 4 (tùy)   │
│  Guild     │   Channel/Category    │   Message Area /      │  Member List   │
│  Rail      │   Sidebar             │   Voice Stage         │  (per channel) │
│ (72px,     │   (240px, danh sách   │   (flexible, chat     │  (240px, ẩn/   │
│  icon list)│   kênh, tìm kiếm DM)  │   log + input box)     │  hiện được)    │
└───────────┴──────────────────────┴──────────────────────┴───────────────┘
```

- **Cột 1 — Guild Rail:** danh sách icon tròn các guild đã tham gia, có badge unread/mention (chấm đỏ số đếm).
- **Cột 2 — Channel Sidebar:** phân nhóm theo Category (collapsible), hiển thị Text Channel (biểu tượng `#`) và Voice Channel (biểu tượng loa, kèm avatar member đang trong kênh).
- **Cột 3 — Nội dung chính:** log tin nhắn cuộn vô hạn (infinite scroll ngược), thanh nhập liệu hỗ trợ Markdown live preview, đính kèm file kéo-thả.
- **Cột 4 — Member List:** phân nhóm theo role (role có màu hiển thị riêng), có thể ẩn/hiện bằng phím tắt.

### 3.2. Chế độ hiển thị mặc định (Default Theme)

- **Dark Mode là mặc định** cho toàn bộ client mới cài đặt, tuân theo bảng màu nền tối (`#313338` nền chính, `#1e1f22` nền sidebar) nhằm giảm mỏi mắt khi sử dụng lâu.
- Light Mode có sẵn như tùy chọn thủ công trong Settings, lưu preference theo tài khoản (đồng bộ đa thiết bị) chứ không theo thiết bị.
- Hệ thống tôn trọng `prefers-color-scheme` của OS khi user chưa từng set preference thủ công (first-time default resolution).

### 3.3. Hành vi chung (General Behavior)

- **Optimistic UI:** tin nhắn gửi đi hiển thị ngay lập tức ở trạng thái "pending" (opacity giảm), cập nhật thành "sent" khi server ACK, hoặc "failed" kèm nút Retry nếu lỗi.
- **Typing Indicator:** hiển thị "X đang nhập..." tự động hết hạn sau 10 giây không có sự kiện typing mới.
- **Toast Notification:** thông báo lỗi hệ thống (mất kết nối Gateway, rate limit) hiển thị dạng toast góc dưới, không chặn thao tác người dùng.
- **Keyboard-first:** hỗ trợ đầy đủ phím tắt điều hướng kênh (Ctrl+K quick switcher), gửi tin nhắn (Enter), xuống dòng (Shift+Enter).

---

## 4. YÊU CẦU CHỨC NĂNG CHI TIẾT (FUNCTIONAL REQUIREMENTS)

### 4.1. Auth & Profile (`REQ-AUTH-*`)

| REQ-ID | Tên chức năng | Mô tả chi tiết | Priority | Acceptance Criteria |
|---|---|---|---|---|
| REQ-AUTH-01 | Đăng ký tài khoản | User đăng ký bằng email + username + password. Hệ thống kiểm tra trùng lặp email/username, gửi email xác thực (verification link, TTL 24h). | Must | Given email hợp lệ chưa tồn tại, When submit form, Then tài khoản tạo ở trạng thái `unverified` và email xác thực được gửi trong vòng 5 giây. |
| REQ-AUTH-02 | Đăng nhập bằng JWT | Xác thực bằng email/username + password, trả về Access Token (JWT, TTL 15 phút) và Refresh Token (TTL 30 ngày, lưu httpOnly cookie). | Must | Given credential đúng, When login, Then nhận access+refresh token; sai password 5 lần liên tiếp trong 10 phút thì tài khoản bị khóa tạm 15 phút. |
| REQ-AUTH-03 | Xác thực hai yếu tố (2FA) | Hỗ trợ bật/tắt 2FA qua TOTP (Google Authenticator/Authy), yêu cầu mã 6 số khi login từ thiết bị lạ. | Should | Given 2FA đã bật, When login đúng password, Then hệ thống yêu cầu mã TOTP trước khi cấp token; mã sai 3 lần thì khóa thử tiếp 5 phút. |
| REQ-AUTH-04 | Refresh Token Rotation | Access Token hết hạn được làm mới qua Refresh Token; mỗi lần refresh cấp refresh token mới, refresh token cũ bị vô hiệu (rotation). | Must | Given refresh token hợp lệ chưa dùng, When gọi refresh endpoint, Then nhận access token mới + refresh token mới; refresh token cũ dùng lại bị từ chối (mã lỗi tái sử dụng). |
| REQ-AUTH-05 | Quản lý Presence | User có thể set trạng thái thủ công: Online/Idle/DND/Invisible; hệ thống tự chuyển Online→Idle sau 10 phút không hoạt động. | Must | Given user không tương tác 10 phút, When idle timeout kích hoạt, Then presence broadcast tới toàn bộ guild chung là `idle`. |
| REQ-AUTH-06 | Cập nhật hồ sơ cá nhân | Chỉnh sửa display name, avatar (upload ảnh, resize tự động), bio, banner, accent color, per-guild nickname riêng. | Should | Given ảnh avatar hợp lệ (≤ 8MB, jpg/png/gif), When upload, Then ảnh được lưu MinIO/S3 và CDN URL trả về trong response profile. |
| REQ-AUTH-07 | Đăng xuất & thu hồi phiên | User đăng xuất một thiết bị hoặc thu hồi toàn bộ phiên đang hoạt động (session management UI). | Should | Given danh sách session hiển thị, When user chọn "Revoke", Then refresh token của session đó bị vô hiệu ngay lập tức và Gateway connection tương ứng bị ngắt. |

### 4.2. Guild & Channels (`REQ-SRV-*`)

| REQ-ID | Tên chức năng | Mô tả chi tiết | Priority | Acceptance Criteria |
|---|---|---|---|---|
| REQ-SRV-01 | Tạo Guild mới | User tạo guild mới với tên, icon tùy chọn; tự động trở thành Owner với role `@everyone` mặc định. | Must | Given tên guild hợp lệ (2-100 ký tự), When submit, Then guild được tạo với 1 category mặc định + 1 text channel `#general` + 1 voice channel `#General Voice`. |
| REQ-SRV-02 | Quản lý Category & Channel | CRUD Category, Text Channel, Voice Channel; sắp xếp thứ tự kéo-thả (drag-drop position); giới hạn tên kênh theo slug-safe format. | Must | Given user có quyền Manage Channels, When kéo thả đổi vị trí kênh, Then thứ tự mới được persist và broadcast realtime tới mọi member online. |
| REQ-SRV-03 | Sinh Invite Link | Tạo link mời tham gia guild với tùy chọn: thời hạn hết hạn (30 phút/1h/1 ngày/7 ngày/không giới hạn), giới hạn số lượt dùng (1/5/10/25/50/100/không giới hạn). | Must | Given tạo invite với max_uses=5, When lượt dùng thứ 6 truy cập, Then hệ thống trả lỗi `40041 Invite Expired/MaxUses`. |
| REQ-SRV-04 | Tham gia Guild qua Invite | User dùng invite code để join guild, tự động gán role `@everyone`, hiển thị trong guild rail. | Must | Given invite code hợp lệ chưa hết hạn/hết lượt, When user accept, Then user xuất hiện trong member list guild trong vòng 1 giây (qua Gateway event `GUILD_MEMBER_ADD`). |
| REQ-SRV-05 | Rời/Xóa/Chuyển nhượng Guild | Member có thể rời guild; Owner có thể xóa guild vĩnh viễn (soft-delete 30 ngày trước khi purge) hoặc chuyển quyền Owner cho member khác. | Should | Given Owner chọn "Transfer Ownership" cho member X, When xác nhận qua 2FA, Then role Owner chuyển sang X và Owner cũ hạ xuống role mặc định do X chỉ định. |
| REQ-SRV-06 | Channel-level Slow Mode | Admin thiết lập giới hạn thời gian tối thiểu giữa 2 tin nhắn liên tiếp của cùng 1 user trong 1 channel (0s-6h). | Could | Given slow mode = 10s, When user gửi tin nhắn thứ 2 trong vòng 10s, Then hệ thống trả lỗi `40029 SlowModeRateLimited` kèm thời gian chờ còn lại. |

### 4.3. RBAC & Permissions (`REQ-ROLE-*`)

| REQ-ID | Tên chức năng | Mô tả chi tiết | Priority | Acceptance Criteria |
|---|---|---|---|---|
| REQ-ROLE-01 | Tạo & Sắp xếp Role | CRUD role với tên, màu, icon; thứ tự role quyết định hierarchy (role cao hơn không thể bị chỉnh sửa bởi role thấp hơn, kể cả nếu có quyền Manage Roles). | Must | Given role A ở vị trí cao hơn role B, When member chỉ có role B cố gắng chỉnh sửa role A, Then hệ thống từ chối với lỗi `40301 InsufficientRoleHierarchy`. |
| REQ-ROLE-02 | Permission Bitmap Guild-level | Mỗi role mang 1 bitmap 64-bit biểu diễn tập quyền (Administrator, ManageServer, ManageRoles, ManageChannels, KickMembers, BanMembers, ManageMessages, SendMessages, Connect, Speak, ...). | Must | Given role có bit `ADMINISTRATOR` bật, When kiểm tra bất kỳ quyền con nào, Then hệ thống luôn trả `true` (Administrator override toàn bộ). |
| REQ-ROLE-03 | Permission Overwrite Channel-level | Cho phép allow/deny bitmap riêng theo channel, áp dụng cho role cụ thể hoặc member cụ thể, độ ưu tiên: `member overwrite > role overwrite > guild role permission`. | Must | Given role X bị deny `SendMessages` ở channel C nhưng member overwrite của user Y (thuộc role X) allow `SendMessages`, When Y gửi tin trong C, Then thao tác thành công (member overwrite thắng). |
| REQ-ROLE-04 | Gán/Thu hồi Role cho Member | Owner/Moderator có quyền `ManageRoles` gán hoặc gỡ role cho member, giới hạn bởi role hierarchy của chính họ. | Must | Given moderator có role cao nhất ở vị trí 5, When cố gán role ở vị trí 3 (cao hơn) cho member khác, Then hệ thống từ chối thao tác. |
| REQ-ROLE-05 | Kick / Ban / Timeout Member | Moderator thực hiện kick (buộc rời, có thể quay lại qua invite mới), ban (chặn vĩnh viễn, có tùy chọn xóa lịch sử tin nhắn N ngày gần nhất), timeout (câm tạm thời từ 60s đến 28 ngày). | Must | Given moderator ban user với `delete_message_days=7`, When thao tác hoàn tất, Then toàn bộ tin nhắn của user đó trong 7 ngày gần nhất bị xóa mềm trên mọi channel của guild. |
| REQ-ROLE-06 | Audit Log | Ghi lại mọi hành động quản trị nhạy cảm (ban/kick/role change/channel delete) kèm actor, target, timestamp, lý do (reason field). | Should | Given moderator thực hiện ban kèm reason "spam", When xem Audit Log, Then entry hiển thị đầy đủ actor/target/action/reason/timestamp, sắp xếp mới nhất trước. |

### 4.4. Messaging & Fanout (`REQ-MSG-*`)

| REQ-ID | Tên chức năng | Mô tả chi tiết | Priority | Acceptance Criteria |
|---|---|---|---|---|
| REQ-MSG-01 | Gửi tin nhắn Real-time | Gửi tin nhắn text qua REST POST, server persist và fanout qua Gateway WebSocket tới toàn bộ client đang subscribe channel đó. | Must | Given user có quyền `SendMessages`, When POST message, Then tin nhắn persist trong < 50ms và event `MESSAGE_CREATE` broadcast tới mọi client trong channel trong < 100ms (P95). |
| REQ-MSG-02 | Markdown & Rich Text | Hỗ trợ cú pháp Markdown cơ bản (bold/italic/strikethrough/code block/quote), auto-embed link preview (Open Graph fetch). | Should | Given nội dung chứa `**bold**`, When client render, Then hiển thị chữ in đậm; link hợp lệ tự động sinh embed card trong vòng 2 giây (async, không block gửi tin nhắn). |
| REQ-MSG-03 | Mentions | Hỗ trợ `@user` (mention cá nhân), `@role` (mention theo role), `@everyone`/`@here` (yêu cầu quyền `MentionEveryone`), `#channel` (link kênh clickable). | Must | Given user không có quyền `MentionEveryone` gõ `@everyone`, When gửi, Then text hiển thị nguyên văn không kích hoạt notification hàng loạt (chỉ escape, không throw lỗi). |
| REQ-MSG-04 | Đính kèm File | Upload file/ảnh/video tối đa 25MB/file (mặc định), lưu trữ MinIO/S3, sinh thumbnail cho ảnh/video, hiển thị progress bar khi upload. | Must | Given file 25.1MB, When upload, Then hệ thống từ chối trước khi bắt đầu upload với lỗi `40013 FileTooLarge`. |
| REQ-MSG-05 | Edit/Delete/Reply/React | Sửa tin nhắn (hiển thị nhãn "đã chỉnh sửa"), xóa tin nhắn (soft-delete), reply có preview tin gốc, thả reaction emoji (giới hạn 20 emoji khác nhau/tin nhắn). | Must | Given user là tác giả tin nhắn hoặc có quyền `ManageMessages`, When xóa, Then event `MESSAGE_DELETE` broadcast realtime; user khác không có quyền cố xóa nhận lỗi `40302 Forbidden`. |
| REQ-MSG-06 | Typing Indicator | Client gửi event `TYPING_START` khi user gõ phím trong ô input; server broadcast tới channel, tự hết hạn sau 10s không lặp lại. | Should | Given user A đang gõ, When user B đang xem channel đó, Then B thấy "A đang nhập..." trong vòng 500ms sau event, tự ẩn nếu không nhận event mới sau 10s. |
| REQ-MSG-07 | Message Fanout Pattern | Với channel/guild có lượng subscriber lớn, hệ thống áp dụng chiến lược fanout kết hợp: push trực tiếp qua Redis Pub/Sub cho các Gateway node đang có client online trong channel đó (tránh fanout tới toàn bộ node không liên quan). | Must | Given guild có 10,000 member nhưng chỉ 200 đang online trong channel, When broadcast message, Then chỉ các Gateway node đang giữ kết nối của 200 client đó nhận event (không broadcast toàn cluster). |
| REQ-MSG-08 | Rate Limiting theo Sliding Window | Giới hạn số tin nhắn/user/channel theo thuật toán sliding-window log (ví dụ tối đa 5 tin nhắn/5 giây) để chống spam. | Must | Given user gửi tin nhắn thứ 6 trong cửa sổ 5 giây, When request tới, Then server trả `40029 RateLimited` kèm header `Retry-After`. |

### 4.5. Voice/Video WebRTC/SFU (`REQ-VC-*`)

| REQ-ID | Tên chức năng | Mô tả chi tiết | Priority | Acceptance Criteria |
|---|---|---|---|---|
| REQ-VC-01 | Kết nối Voice Channel qua SFU | Client join voice channel, thiết lập kết nối WebRTC tới SFU node (LiveKit/Mediasoup) gần nhất theo khu vực địa lý; signaling qua WebSocket riêng (Voice Gateway). | Must | Given user join voice channel, When kết nối thiết lập, Then audio 2 chiều hoạt động trong vòng 3 giây kể từ lúc click Join (P95). |
| REQ-VC-02 | Voice Activity Detection (VAD) | Client-side VAD phát hiện khi user đang nói, gửi event tới server để hiển thị vòng sáng (speaking indicator) quanh avatar cho toàn bộ participant. | Should | Given user bắt đầu nói, When VAD kích hoạt, Then avatar user hiển thị viền xanh sáng trong vòng 200ms trên client của toàn bộ participant khác. |
| REQ-VC-03 | Screen Share | Chia sẻ màn hình/cửa sổ ứng dụng cụ thể, hỗ trợ độ phân giải tối đa 1080p60, kèm tùy chọn chia sẻ âm thanh hệ thống. | Should | Given user bắt đầu share màn hình, When participant khác xem, Then video stream hiển thị trong "Stage" riêng với độ trễ < 500ms. |
| REQ-VC-04 | Mute/Deafen | User tự mute mic hoặc deafen (tắt cả nghe); Moderator có thể server-mute/deafen member khác (yêu cầu quyền `MuteMembers`/`DeafenMembers`). | Must | Given moderator server-mute user X, When X cố nói, Then audio track của X không được forward tới participant khác dù client X vẫn hiển thị đang unmute cục bộ (mute ở tầng SFU). |
| REQ-VC-05 | Thích ứng Jitter/Packet Loss | SFU áp dụng cơ chế điều chỉnh bitrate động (adaptive bitrate) và simulcast layer selection dựa trên điều kiện mạng đo được (RTT, jitter, packet loss) của từng client riêng lẻ. | Should | Given packet loss của 1 client tăng trên 5%, When SFU phát hiện, Then chất lượng audio/video gửi tới client đó tự động giảm layer trong vòng 2 giây mà không ảnh hưởng client khác. |
| REQ-VC-06 | Global Voice Gateway đa vùng | Hệ thống Voice Gateway triển khai phân tán theo khu vực địa lý (multi-region: ví dụ APAC, US, EU); client được route tới edge node gần nhất dựa trên đo lường RTT thực tế. | Should | Given user ở khu vực APAC, When join voice channel, Then client kết nối tới SFU node khu vực APAC (không phải node mặc định ở region khác) trừ khi RTT đo được cho thấy node khác tốt hơn. |
| REQ-VC-07 | Voice Channel User Limit | Owner cấu hình giới hạn số lượng user tối đa trong 1 voice channel (0 = không giới hạn, hoặc 1-99). | Could | Given voice channel giới hạn 5 user và đã đủ 5, When user thứ 6 cố join, Then hệ thống từ chối kết nối SFU với lỗi `40051 VoiceChannelFull`. |

### 4.6. Search & Notifications (`REQ-SEARCH/NOTIF-*`)

| REQ-ID | Tên chức năng | Mô tả chi tiết | Priority | Acceptance Criteria |
|---|---|---|---|---|
| REQ-SEARCH-01 | Tìm kiếm tin nhắn nâng cao | Tìm kiếm full-text trong phạm vi channel/guild/DM, hỗ trợ filter theo `from:user`, `has:link/image/file`, `before:/after:date`, `mentions:`. | Should | Given query `from:alice has:image`, When search trong channel, Then kết quả chỉ trả tin nhắn của alice có đính kèm ảnh, sắp xếp theo độ liên quan/thời gian. |
| REQ-SEARCH-02 | Cursor-based Pagination cho Message History | Load lịch sử tin nhắn cũ hơn dùng cursor (message ID Snowflake) thay vì offset, đảm bảo không lệch trang khi có tin nhắn mới liên tục được thêm vào. | Must | Given client scroll lên xem tin cũ, When gọi API với `before=<message_id>`, Then trả đúng 50 tin nhắn liền trước đó, không trùng lặp dù có tin nhắn mới được gửi song song. |
| REQ-NOTIF-01 | Unread & Mention Badge | Hiển thị số đếm unread theo channel/guild, badge riêng biệt (màu đỏ) khi có mention trực tiếp, đồng bộ trạng thái đã đọc (read state) đa thiết bị. | Must | Given user đọc hết channel trên thiết bị A, When mở thiết bị B, Then badge unread của channel đó biến mất trong vòng 2 giây (đồng bộ qua Gateway event `MESSAGE_ACK`). |
| REQ-NOTIF-02 | Push Notification | Gửi push notification (qua FCM/APNs cho mobile, Web Push cho browser) khi user offline hoặc app ở background và nhận mention/DM. | Should | Given user app ở background và nhận mention, When event xảy ra, Then push notification hiển thị trong vòng 5 giây kèm preview nội dung (tôn trọng cài đặt privacy "ẩn nội dung"). |
| REQ-NOTIF-03 | Tùy chỉnh mức thông báo | User cấu hình mức thông báo theo guild/channel: All Messages / Only Mentions / Nothing; hỗ trợ mute tạm thời (15 phút/1h/8h/24h/vĩnh viễn). | Could | Given channel được set "Only Mentions", When có tin nhắn thường (không mention user), Then không sinh push notification nhưng vẫn cập nhật unread badge thường (không phải mention badge). |

### 4.7. Kết bạn & Direct Message (`REQ-FRIEND-*` / `REQ-DM-*`)

> Luồng này độc lập với Guild — tương ứng tab "Home" trong Discord thật, nơi user tương tác 1-1 với bạn bè mà không cần chung server.

| REQ-ID | Tên chức năng | Mô tả chi tiết | Priority | Acceptance Criteria |
|---|---|---|---|---|
| REQ-FRIEND-01 | Gửi lời mời kết bạn | User gửi friend request qua username#discriminator hoặc từ member list chung guild; hệ thống chặn gửi trùng lặp yêu cầu đang pending. | Must | Given A gửi request tới B lần đầu, When B chưa phản hồi, Then A gửi lại request tới B nhận lỗi `40902 FriendRequestAlreadyPending` thay vì tạo bản ghi mới. |
| REQ-FRIEND-02 | Chấp nhận / Từ chối / Hủy lời mời | B có thể Accept (tạo quan hệ bạn bè 2 chiều) hoặc Decline (xóa request); A có thể Cancel request đang pending trước khi B phản hồi. | Must | Given B click Accept, When thao tác hoàn tất, Then cả A và B đều thấy nhau trong Friend List, đồng thời một `dm_channel` 1-1 được tự động tạo/kích hoạt giữa 2 người. |
| REQ-FRIEND-03 | Chặn người dùng (Block) | User block người dùng khác: ẩn toàn bộ tin nhắn/DM cũ và mới từ người bị block, tự động hủy friend relationship nếu đang là bạn bè, chặn gửi friend request/DM mới. | Must | Given A block B, When B cố gửi tin nhắn DM cho A, Then request bị từ chối ở tầng server với lỗi `40303 BlockedByRecipient`, B không nhận được thông báo cụ thể lý do (tránh lộ thông tin block). |
| REQ-FRIEND-04 | Danh sách bạn bè & trạng thái | Hiển thị Friend List phân nhóm theo trạng thái (Online/Idle/DND lên trước, Offline gộp cuối), tìm kiếm nhanh theo tên. | Should | Given danh sách 200 bạn bè, When mở tab Home > Friends, Then danh sách render với thời gian < 300ms nhờ cache local, cập nhật realtime khi presence thay đổi. |
| REQ-DM-01 | Nhắn tin 1-1 (DM) | Gửi tin nhắn text/file/reaction/reply trong `dm_channel` 1-1, tái sử dụng toàn bộ pipeline persist + fanout của REQ-MSG-01/04/05, không phụ thuộc guild/RBAC. | Must | Given A và B là bạn bè, When A gửi DM, Then B nhận event `MESSAGE_CREATE` trên `dm_channel` tương ứng với độ trễ tương đương REQ-MSG-01. |
| REQ-DM-02 | Nhắn tin cho người lạ chung server | Cho phép nhắn DM tới user chưa kết bạn nhưng đang chung ít nhất 1 guild, trừ khi người nhận đã tắt "Cho phép DM từ thành viên server" trong Privacy Settings. | Should | Given B đã tắt DM từ thành viên server, When A (chung guild, chưa là bạn) gửi DM, Then hệ thống từ chối với lỗi `40304 DMNotAllowed`. |

### 4.8. Group DM & Gọi thoại/video 1-1 (`REQ-GDM-*` / `REQ-DMVC-*`)

| REQ-ID | Tên chức năng | Mô tả chi tiết | Priority | Acceptance Criteria |
|---|---|---|---|---|
| REQ-GDM-01 | Tạo nhóm nhỏ giữa bạn bè | User tạo Group DM bằng cách thêm 2-9 bạn bè (tối đa 10 người/nhóm bao gồm người tạo) — hoàn toàn khác Guild: không có role, không có nhiều channel, không cần invite link công khai. | Must | Given user chọn 3 bạn bè và bấm "Create Group DM", When submit, Then hệ thống tạo 1 `dm_channel` với `type=GROUP_DM` chứa đúng 4 thành viên, không sinh Guild/Role/Category nào. |
| REQ-GDM-02 | Quản lý thành viên Group DM | Bất kỳ thành viên nào cũng có thể thêm bạn bè mới vào nhóm (nếu người đó là bạn của mình) hoặc tự rời nhóm; người tạo nhóm có thể đổi tên/ảnh nhóm và gỡ thành viên khác. | Should | Given nhóm có 5 người, When 1 thành viên rời, When nhóm còn ≥ 2 người thì tồn tại bình thường; nếu chỉ còn 1 người, Then nhóm tự chuyển về trạng thái ẩn (không xóa lịch sử) cho tới khi có người join lại. |
| REQ-GDM-03 | Giới hạn quy mô | Hệ thống giới hạn cứng 10 thành viên/Group DM (khác Guild vốn không giới hạn số lượng), do Group DM không dùng SFU cluster đa vùng như voice channel guild mà dùng mesh/SFU nhẹ tối ưu cho nhóm nhỏ. | Must | Given nhóm đã đủ 10 thành viên, When thêm thành viên thứ 11, Then hệ thống từ chối với lỗi `40052 GroupDMFull`. |
| REQ-DMVC-01 | Gọi thoại/video 1-1 | Từ màn hình DM 1-1, user có thể khởi tạo cuộc gọi thoại hoặc video trực tiếp (không cần "join channel" như voice channel của guild); người nhận thấy incoming-call UI toàn màn hình kèm rung/chuông kể cả khi app ở background. | Must | Given A gọi video cho B, When B đang mở app ở nền, Then B nhận push notification dạng VoIP/CallKit (iOS) hoặc high-priority FCM (Android) hiển thị incoming-call trong vòng 2 giây, độ trễ thiết lập kết nối tương tự REQ-VC-01. |
| REQ-DMVC-02 | Gọi nhóm trong Group DM | Bất kỳ thành viên nào cũng có thể bắt đầu cuộc gọi thoại/video trong Group DM; các thành viên khác thấy banner "đang trong cuộc gọi" và có thể tham gia/rời bất kỳ lúc nào mà không làm gián đoạn người còn lại. | Should | Given cuộc gọi nhóm đang diễn ra với 3/5 thành viên, When thành viên thứ 4 bấm Join, Then người đó kết nối vào cùng phiên SFU hiện có trong < 3 giây, không tạo phiên gọi mới. |
| REQ-DMVC-03 | Từ chối/Nhỡ cuộc gọi | Người nhận có thể từ chối cuộc gọi (Decline) hoặc không phản hồi trong 45 giây thì tự động chuyển thành Missed Call, ghi lại dưới dạng system message trong lịch sử DM. | Must | Given A gọi B và không ai bắt máy trong 45s, When timeout, Then cuộc gọi tự kết thúc, cả 2 phía thấy system message "Cuộc gọi nhỡ" trong khung chat, kèm nút "Gọi lại". |

### 4.9. Study/Focus Mode — Tính năng định vị cho Sinh viên (`REQ-STUDY-*`)

> Toàn bộ mục này xây **trên nền** hạ tầng đã có (Guild/Channel/Voice/Messaging/Notification) — không tạo service riêng, chỉ mở rộng metadata và hành vi client. Đây là phần giúp đồ án có câu chuyện sản phẩm rõ ràng thay vì chỉ là "Discord clone thuần kỹ thuật".

| REQ-ID | Tên chức năng | Mô tả chi tiết | Priority | Acceptance Criteria |
|---|---|---|---|---|
| REQ-STUDY-01 | Focus Mode cho Voice Channel (Pomodoro đồng bộ) | Bất kỳ voice channel nào cũng bật được "Focus Session": chọn chu kỳ (25/5, 50/10, tùy chỉnh), bộ đếm thời gian hiển thị đồng bộ cho mọi participant, tự chuyển "Học"→"Nghỉ" và phát âm báo nhẹ khi chuyển pha. | Must | Given 4 người trong voice channel bật Focus Session 25/5, When 1 người bấm Start, Then đồng hồ đếm ngược hiển thị đồng nhất (sai lệch < 1s) trên toàn bộ client tham gia, dựa trên 1 `server_timestamp` gốc phát qua Gateway (không tính theo client clock riêng lẻ). |
| REQ-STUDY-02 | Silent Co-working Room | Chế độ voice channel mặc định mic tắt cho tất cả (trừ khi user chủ động bật để hỏi/thảo luận), video khuyến khích bật để tạo cảm giác "cùng học"; có tùy chọn phát nhạc nền lo-fi dùng chung (đồng bộ qua server, không phải mỗi client tự phát lệch nhịp). | Should | Given user join Silent Co-working Room, When kết nối thành công, Then mic mặc định ở trạng thái muted (client-side mute, không phải server-mute — user tự bật lại được ngay), khác hành vi voice channel thường (REQ-VC-01) vốn mic mặc định bật. |
| REQ-STUDY-03 | Study Streak cá nhân | Hệ thống tự tính "streak" (số ngày liên tiếp có ít nhất 1 Focus Session ≥ 15 phút), hiển thị trên profile cá nhân kèm biểu tượng lửa; streak reset nếu bỏ lỡ 1 ngày. | Should | Given user hoàn thành Focus Session hôm nay và hôm qua, When mở profile, Then streak hiển thị "2 ngày"; nếu user bỏ lỡ 1 ngày liên tiếp, Then streak reset về 0 vào 00:00 giờ địa phương của user. |
| REQ-STUDY-04 | Thống kê giờ học nhóm (Study Leaderboard) | Mỗi guild có bảng xếp hạng tổng giờ Focus Session của thành viên theo tuần/tháng, Owner có thể bật/tắt tính năng này (tôn trọng người không muốn bị so sánh — opt-out cá nhân vẫn được). | Could | Given guild bật Leaderboard, When user cá nhân chọn "Ẩn tôi khỏi bảng xếp hạng" trong Settings, Then giờ học của user vẫn được ghi nhận nội bộ nhưng không hiển thị tên trên leaderboard công khai. |
| REQ-STUDY-05 | Gắn nhãn Guild là "Study Server" | Khi tạo guild, Owner chọn template "Study Server" (khác template "General"): tự sinh sẵn cấu trúc category gợi ý (`#thong-bao`, `#tai-lieu`, `#hoi-dap`, `Phòng học im lặng`, `Phòng thảo luận nhóm`) thay vì chỉ 1 channel mặc định như REQ-SRV-01. | Should | Given user chọn template "Study Server" khi tạo guild, When guild khởi tạo xong, Then guild có sẵn 2 category, 3 text channel, 2 voice channel (1 Silent Co-working, 1 Discussion) theo đúng preset, user vẫn chỉnh sửa được sau đó như guild thường. |
| REQ-STUDY-06 | Task Board gắn theo Channel | Mỗi text channel có thể bật 1 Task Board đơn giản (dạng checklist: tiêu đề, người phụ trách tùy chọn, deadline tùy chọn, trạng thái Chưa làm/Đang làm/Xong) hiển thị dạng panel bên cạnh khung chat — không phải Forum Channel đầy đủ, chỉ đủ dùng cho nhóm nhỏ phân việc đồ án. | Could | Given member thêm 1 task với deadline ngày mai, When deadline còn 2 giờ và task vẫn "Chưa làm"/"Đang làm", Then hệ thống gửi reminder notification riêng cho người phụ trách (nếu có gán) hoặc toàn kênh (nếu không gán ai). |
| REQ-STUDY-07 | Gợi ý ghép nhóm học (Study Buddy Matching) | User có thể đăng ký "đang tìm bạn học" kèm môn/chủ đề quan tâm; hệ thống gợi ý các guild công khai (đã bật Server Discovery — REQ-BKLG-09) hoặc user khác cùng quan tâm chủ đề đó để kết bạn/tham gia. | Could | Given user đăng ký chủ đề "Giải tích 2", When có guild công khai gắn tag "Giải tích 2", Then guild đó xuất hiện trong danh sách gợi ý ở tab Discovery, sắp xếp theo số thành viên đang hoạt động gần đây. |
| REQ-STUDY-08 | Deadline tổng hợp cá nhân (Study Digest) | Tổng hợp toàn bộ task được gán cho user (từ mọi guild đang tham gia) vào 1 màn hình duy nhất "Việc cần làm", sắp xếp theo deadline gần nhất — tránh phải mở từng guild để kiểm tra. | Should | Given user có task ở 3 guild khác nhau, When mở tab "Việc cần làm", Then cả 3 task hiển thị chung 1 danh sách, sắp xếp deadline tăng dần, mỗi task có link nhảy thẳng tới channel gốc. |
| REQ-STUDY-09 | Ưu tiên hóa Notification cho ngữ cảnh học tập | Trong Notification Settings, user có thể set "Chế độ Học tập" theo khung giờ cố định (vd: 20h-22h hằng ngày): tự động chỉ cho phép notification loại Mention/Deadline/Cuộc gọi lọt qua, tạm ẩn tin nhắn thường của guild không liên quan học tập. | Could | Given "Chế độ Học tập" đang active (20h-22h) và guild không gắn tag học tập gửi tin nhắn thường, When tin nhắn đến, Then không sinh push notification, chỉ cập nhật unread badge âm thầm; tin nhắn có mention trực tiếp tới user vẫn thông báo bình thường. |

---

## 5. YÊU CẦU PHI CHỨC NĂNG (NON-FUNCTIONAL REQUIREMENTS)

### 5.1. Hiệu năng (Performance — NFR-PERF)

| ID | Yêu cầu |
|---|---|
| NFR-PERF-01 | RTT gửi/nhận tin nhắn text trong cùng region phải < 100ms ở P95. |
| NFR-PERF-02 | Độ trễ audio end-to-end trong voice chat phải < 150ms ở P95, < 250ms ở P99. |
| NFR-PERF-03 | Thời gian tải initial message history (50 tin gần nhất) khi mở channel phải < 300ms ở P95. |
| NFR-PERF-04 | API REST (không tính upload file) phải phản hồi < 200ms ở P95 dưới tải 1,000 request/giây/service instance. |

### 5.2. Khả năng mở rộng (Scalability — NFR-SCALE)

| ID | Yêu cầu |
|---|---|
| NFR-SCALE-01 | Gateway service phải horizontal scale qua nhiều node, đồng bộ trạng thái session/presence qua Redis Pub/Sub, không có single point of bottleneck khi thêm node. |
| NFR-SCALE-02 | Message store phải áp dụng chiến lược phân vùng (partitioning) theo channel_id để tránh hot partition khi 1 channel có traffic cao đột biến, tham khảo mô hình ScyllaDB của Discord (R2). |
| NFR-SCALE-03 | Hệ thống phải hỗ trợ tối thiểu 100,000 kết nối WebSocket đồng thời trên toàn cluster ở giai đoạn MVP, có khả năng scale tuyến tính khi thêm node Gateway. |
| NFR-SCALE-04 | SFU cluster phải hỗ trợ auto-scale theo số phòng voice active, với cơ chế load balancing chọn node có capacity thấp nhất khi tạo phòng mới. |

### 5.3. Bảo mật (Security — NFR-SEC)

| ID | Yêu cầu |
|---|---|
| NFR-SEC-01 | Toàn bộ traffic client-server bắt buộc TLS 1.3; WebSocket phải dùng WSS, không cho phép fallback ws:// plaintext ở môi trường production. |
| NFR-SEC-02 | Password lưu trữ bắt buộc hash bằng bcrypt (cost factor ≥ 12) hoặc Argon2id, không bao giờ lưu plaintext hoặc log ra hệ thống logging. |
| NFR-SEC-03 | Áp dụng rate limiting theo sliding-window ở tầng API Gateway cho mọi endpoint public, ngưỡng riêng theo loại action (auth, message, voice-join) để chống brute-force và spam. |
| NFR-SEC-04 | File upload phải qua kiểm tra MIME type thực tế (magic bytes, không chỉ dựa vào extension) và virus scan trước khi lưu trữ vĩnh viễn. |
| NFR-SEC-05 | Audit Log cho hành động quản trị phải immutable (append-only), lưu trữ tối thiểu 90 ngày. |

### 5.4. Độ sẵn sàng & Độ tin cậy (Availability & Reliability — NFR-AVAIL)

| ID | Yêu cầu |
|---|---|
| NFR-AVAIL-01 | SLA uptime hệ thống tổng thể tối thiểu 99.9% (tương đương downtime tối đa ~43.8 phút/tháng). |
| NFR-AVAIL-02 | Message store phải có replication factor ≥ 3 để đảm bảo không mất dữ liệu khi 1 node fail. |
| NFR-AVAIL-03 | Client phải hỗ trợ tự động reconnect + session resume khi mất kết nối Gateway tạm thời (< 60 giây), không yêu cầu user thao tác thủ công. |
| NFR-AVAIL-04 | Voice channel phải failover sang SFU node dự phòng trong vòng 5 giây nếu node hiện tại gặp sự cố, tối thiểu hóa gián đoạn cuộc gọi. |

### 5.5. Khả năng bảo trì & Quan sát (Maintainability & Observability — NFR-OBS)

| ID | Yêu cầu |
|---|---|
| NFR-OBS-01 | Mọi microservice phải expose metrics chuẩn Prometheus (latency, error rate, throughput) và structured logging (JSON) tập trung qua ELK/Loki. |
| NFR-OBS-02 | Distributed tracing (OpenTelemetry) bắt buộc cho luồng request xuyên nhiều service (vd: gửi tin nhắn → fanout → push notification). |
| NFR-OBS-03 | Pipeline phân tích dữ liệu hành vi người dùng (cohort, retention) phải chạy tách biệt khỏi hệ thống transactional, theo mô hình ETL định kỳ tương tự BigQuery + Airflow DAGs của Discord (R3), không ảnh hưởng hiệu năng CSDL chính. |

---

## 6. KIẾN TRÚC HỆ THỐNG & TECH STACK

### 6.1. Kiến trúc tổng thể (High-Level Architecture)

```
                                   ┌─────────────────────┐
                                   │        CDN            │
                                   │ (static/avatar/emoji) │
                                   └──────────┬───────────┘
                                              │
   ┌───────────┐   HTTPS/REST    ┌───────────▼────────────┐
   │  Web/      │────────────────▶│      API Gateway /      │
   │  Desktop/  │                 │  Load Balancer (Nginx/  │
   │  Mobile    │◀────────────────│      Envoy)             │
   │  Client    │   WSS (Gateway) └───────────┬────────────┘
   └─────┬──────┘                             │
         │                    ┌────────────────┼────────────────────────┐
         │                    │                │                        │
         │           ┌────────▼──────┐ ┌───────▼────────┐   ┌──────────▼─────────┐
         │           │  Auth Service   │ │ Guild/Channel   │   │   Messaging          │
         │           │ (JWT/2FA)       │ │ Service (RBAC)  │   │   Service             │
         │           └────────┬──────┘ └───────┬────────┘   └──────────┬─────────┘
         │                    │                │                        │
         │           ┌────────▼─────────────────▼────────────────────────▼─────────┐
         │           │                     Postgres (primary metadata)                │
         │           │      Users / Guilds / Channels / Roles / Permissions           │
         │           └───────────────────────────────────────────────────────────────┘
         │
         │           ┌───────────────────────────────────────────────────────────────┐
         │           │           ScyllaDB / Cassandra (Message Store, wide-column)     │
         │           │   Partition Key: channel_id | Clustering Key: message_id (desc) │
         │           └───────────────────────────────────────────────────────────────┘
         │
   ┌─────▼───────────────────┐        ┌─────────────────────────┐
   │   Gateway Cluster         │◀──────▶│   Redis Cluster           │
   │  (WebSocket, N nodes,     │ Pub/Sub│  (Pub/Sub liên-node,      │
   │   session/heartbeat)      │        │   Presence cache, Rate    │
   └─────────────┬────────────┘        │   Limit counters)         │
                 │                     └─────────────────────────┘
   ┌─────────────▼────────────┐        ┌─────────────────────────┐
   │   Voice Gateway            │◀──────▶│  SFU Cluster              │
   │ (signaling, multi-region)  │  WebRTC│  (LiveKit / Mediasoup,    │
   │                             │        │   multi-region edge)      │
   └────────────────────────────┘        └─────────────────────────┘

   ┌────────────────────────┐   ┌────────────────────────┐
   │  Object Storage           │   │  Analytics Pipeline       │
   │  (MinIO/S3 — files,       │   │  (Airflow DAG → BigQuery/ │
   │   avatars, attachments)   │   │   ClickHouse, batch ETL)  │
   └────────────────────────┘   └────────────────────────┘
```

### 6.2. Tech Stack đề xuất

| Lớp | Công nghệ | Lý do lựa chọn |
|---|---|---|
| Frontend Web | Next.js/React + TypeScript, TailwindCSS | SSR cho SEO landing page, CSR cho app shell, hệ sinh thái component lớn. |
| Desktop | Tauri (ưu tiên) hoặc Electron | Tauri cho binary nhẹ hơn, bảo mật tốt hơn qua Rust backend. |
| Mobile | Flutter hoặc React Native | Codebase chung iOS/Android, hiệu năng animation tốt (Flutter) hoặc tái dùng code React (RN). |
| API/Gateway Service | Go (Golang) hoặc Node.js (NestJS) | Go cho service cần throughput cao (Gateway, Messaging); Node.js cho service CRUD nghiệp vụ (Guild/Auth). |
| Metadata DB | PostgreSQL (với read replica) | ACID cho dữ liệu quan hệ: Users, Guilds, Roles, Permissions. |
| Message Store | ScyllaDB (ưu tiên) hoặc Cassandra | Wide-column store tối ưu ghi cao, mô hình theo kinh nghiệm thực tế của Discord (R2). |
| Cache/Pub-Sub | Redis Cluster | Session cache, Presence, Rate limiting (sliding window), Pub/Sub liên Gateway node. |
| Media/Voice | LiveKit hoặc Mediasoup (SFU) | Mã nguồn mở, hỗ trợ simulcast, đã kiểm chứng ở quy mô production. |
| Object Storage | MinIO (self-host) / AWS S3 | Lưu trữ file đính kèm, avatar, emoji tùy chỉnh. |
| Message Queue | Kafka hoặc NATS | Event streaming cho fanout ở quy mô lớn, decouple giữa Messaging Service và Notification Service. |
| Search | Elasticsearch/OpenSearch hoặc Meilisearch | Full-text search cho REQ-SEARCH-01. |
| Analytics | Airflow (orchestration) + BigQuery/ClickHouse | ETL batch, cohort analysis, theo mô hình R3. |
| Observability | Prometheus + Grafana, OpenTelemetry, Loki/ELK | Metrics, tracing, log tập trung. |
| Container/Orchestration | Docker + Kubernetes | Auto-scale, self-healing cho microservices. |

### 6.3. Kiến trúc Mobile App (Client-side Architecture) — TRỌNG TÂM ĐỒ ÁN

> Vì đây là đồ án môn App, phần backend ở mục 6.1/6.2 chỉ đóng vai trò **bối cảnh hệ thống** (để sinh viên hiểu API/Gateway giao tiếp thế nào); khi triển khai thực tế cho đồ án, backend có thể rút gọn thành 1 service monolith + Postgres + Redis + 1 SFU node đơn (bỏ ScyllaDB/Kafka/multi-region nếu không đủ thời gian — xem ghi chú "Scope rút gọn" ở 6.3.5). Phần **client/App** dưới đây mới là nơi cần đầu tư chi tiết nhất.

#### 6.3.1. Layered Architecture (Clean Architecture / MVVM)

App tổ chức theo 3 tầng tách biệt, giao tiếp một chiều từ trên xuống, giúp test được từng lớp độc lập:

```
┌──────────────────────────────────────────────────────────────┐
│  PRESENTATION LAYER                                            │
│  - Screens/Widgets (UI): GuildRail, ChannelList, ChatScreen,   │
│    FriendList, CallScreen, IncomingCallOverlay                 │
│  - ViewModel / Controller (Riverpod-StateNotifier / Bloc /     │
│    MVVM ViewModel) — giữ UI State, gọi UseCase, KHÔNG chứa     │
│    logic nghiệp vụ                                             │
└───────────────────────────┬──────────────────────────────────┘
                             │  gọi UseCase (interface, không phụ thuộc Flutter/RN widget)
┌───────────────────────────▼──────────────────────────────────┐
│  DOMAIN LAYER (thuần Dart/TypeScript, không phụ thuộc SDK UI)  │
│  - Entities: User, Guild, Channel, Message, Friendship, Call   │
│  - UseCases: SendMessageUseCase, JoinVoiceUseCase,              │
│    AcceptFriendRequestUseCase, StartCallUseCase                │
│  - Repository Interfaces (abstract)                            │
└───────────────────────────┬──────────────────────────────────┘
                             │  implement bởi
┌───────────────────────────▼──────────────────────────────────┐
│  DATA LAYER                                                    │
│  - RepositoryImpl: kết hợp Remote + Local, quyết định          │
│    cache-first hay network-first theo loại dữ liệu             │
│  - RemoteDataSource: REST client (Dio/Retrofit) + Gateway       │
│    WebSocket client + WebRTC client wrapper                    │
│  - LocalDataSource: DB cục bộ (Drift/Room/Realm) — xem 6.3.2    │
└──────────────────────────────────────────────────────────────┘
```

**Nguyên tắc bắt buộc cho đồ án:** ViewModel không bao giờ gọi thẳng REST/WebSocket API — luôn qua UseCase → Repository, để khi giảng viên yêu cầu đổi backend (vd: mock server sang server thật) chỉ cần đổi `RepositoryImpl`, không sửa UI.

#### 6.3.2. Offline-first & Local Cache (điểm cộng lớn cho đồ án)

App phải hoạt động được ở mức tối thiểu khi mất mạng (xem tin nhắn cũ, danh sách bạn bè, danh sách guild), không trắng màn hình:

| Loại dữ liệu | Chiến lược cache local | Công nghệ đề xuất |
|---|---|---|
| Profile, Guild list, Channel list, Friend list, Role | Cache-first, đồng bộ lại khi có mạng (background sync) | SQLite qua Drift (Flutter) / Room (Android-Kotlin) / Realm (React Native) |
| Message history (50 tin gần nhất/channel đã mở) | Ghi xuống local ngay khi nhận qua Gateway hoặc REST, hiển thị local trước — gọi mạng chỉ để "làm mới" (delta sync) | Bảng local `cached_messages` mirror schema mục 7.2, đánh index theo `channel_id` |
| Outbox tin nhắn khi offline | Queue tin nhắn chưa gửi được (status=`pending`), tự động retry khi có mạng trở lại (nền tảng cho REQ-MSG-01 "Optimistic UI") | Local outbox table + WorkManager (Android) / BGTaskScheduler (iOS) / background_fetch |
| Media (avatar, ảnh đính kèm) | LRU disk cache, giới hạn dung lượng (vd: 200MB), tự xóa cache cũ nhất | `cached_network_image` (Flutter) / Coil-Glide (Android) / react-native-fast-image |

#### 6.3.3. Quản lý vòng đời kết nối Gateway (WebSocket) trên Mobile

Đây là phần **đặc thù mobile** khác hẳn web — app bị OS kill/suspend nền nên không thể giữ WebSocket sống vĩnh viễn:

```
App Foreground        →  WebSocket Gateway kết nối liên tục, heartbeat mỗi 30-45s
App Background (< 5p) →  Giữ kết nối (OS cho phép trong background window ngắn),
                          giảm heartbeat để tiết kiệm pin
App Background (> 5p) →  OS có thể suspend socket → App chuyển hoàn toàn sang
                          dựa vào Push Notification (FCM/APNs) để nhận biết sự kiện mới
App Foreground trở lại →  Reconnect WebSocket + gửi "Resume" kèm last_sequence_number
                          đã nhận → Server replay các event đã miss (đúng NFR-AVAIL-03)
```

- **Vì sao không polling liên tục:** tốn pin/data, App Store/Play Store background execution limit không cho phép giữ socket vô thời hạn.
- **Vì sao cần Push song song WebSocket:** Push đảm bảo user vẫn nhận mention/DM/cuộc gọi dù app bị kill hẳn — WebSocket chỉ tối ưu trải nghiệm real-time khi app đang mở.

#### 6.3.4. Kiến trúc gọi thoại/video (VoIP) trên Mobile — REQ-DMVC-*

Khác hẳn voice channel của guild (join thủ công qua UI), cuộc gọi DM cần "đánh thức" app ngay cả khi bị kill — bắt buộc dùng cơ chế VoIP push riêng của từng OS:

```
                    ┌───────────────────────┐
                    │   Call Signaling Svc    │  (REST: POST /calls, WS event CALL_INVITE)
                    └──────────┬────────────┘
                               │
        ┌──────────────────────┼──────────────────────┐
        │                                              │
  iOS: PushKit VoIP push          Android: FCM high-priority
  → hệ điều hành đánh thức app    data message → app tự khởi
  dù bị kill, hiển thị native     tạo Foreground Service +
  CallKit incoming-call UI        ConnectionService/full-screen
        │                         intent notification
        └──────────────────────┬──────────────────────┘
                               │  user Accept
                    ┌──────────▼────────────┐
                    │  WebRTC Client SDK      │  (flutter_webrtc / react-native-webrtc)
                    │  kết nối tới SFU room   │
                    └────────────────────────┘
```

- **CallKit (iOS)** và **ConnectionService (Android)** bắt buộc dùng để cuộc gọi hiển thị đúng chuẩn OS (full-screen, tích hợp nút Home Screen/Lock Screen), không dùng notification thường vì không đủ ưu tiên đánh thức app.
- Với Group DM (REQ-GDM/REQ-DMVC-02), client kiểm tra `call_sessions.status` trước khi tạo phiên WebRTC mới — nếu đã `active` thì join vào room hiện có thay vì tạo room mới.

#### 6.3.5. Scope rút gọn cho đồ án (Simplified Scope Note)

Vì mục 6.1/6.2 mô tả kiến trúc chuẩn production (tham khảo Discord thật), nếu thời gian đồ án hạn chế, nhóm có thể rút gọn backend nhưng **vẫn giữ nguyên kiến trúc client ở 6.3.1–6.3.4** (đây mới là phần được chấm điểm về "App"):

| Thành phần production (6.1/6.2) | Có thể thay bằng (scope đồ án) |
|---|---|
| ScyllaDB/Cassandra phân vùng theo bucket | 1 bảng `messages` trong PostgreSQL, đánh index `(channel_id, created_at)` |
| Gateway cluster nhiều node + Redis Pub/Sub liên node | 1 Node.js/Go process dùng Socket.IO hoặc `ws`, giữ toàn bộ connection trong memory (đủ cho demo lớp học) |
| SFU multi-region (LiveKit self-host cluster) | 1 LiveKit Cloud project (free tier) hoặc 1 LiveKit node đơn |
| Kafka/NATS event streaming | Gọi trực tiếp hàm nội bộ trong cùng service (bỏ qua message queue) |
| Kubernetes auto-scale | Deploy 1 VM/Render/Railway, không cần auto-scale |

#### 6.3.6. Focus Mode Timer & Local Notification (đặc thù Study/Focus)

Khác với chuông cuộc gọi (REQ-DMVC-01, bắt buộc VoIP push vì không đoán trước được thời điểm), đồng hồ đếm ngược Focus Session (REQ-STUDY-01) **có thể tính toán được thời điểm chuyển pha ngay từ lúc bắt đầu** (`server_started_at` + `cycle_focus_min`/`cycle_break_min` là dữ liệu tất định), nên không cần giữ WebSocket sống liên tục hay dùng push server-side:

1. Khi user join Focus Session, client nhận 1 lần `server_started_at` + cấu hình chu kỳ qua Gateway (hoặc REST nếu app vừa mở).
2. Client tự lên lịch **local notification** (không qua server) cho từng mốc chuyển pha Học→Nghỉ→Học kế tiếp, dùng `flutter_local_notifications`/`WorkManager` (Android) hoặc `UNUserNotificationCenter` (iOS) — hoạt động được kể cả khi app ở background, không tốn pin giữ socket.
3. Khi app mở lại foreground, client tính lại `elapsed = now - server_started_at` để hiển thị đúng vị trí trong chu kỳ hiện tại (tự phục hồi UI, không cần hỏi lại server trừ khi lệch giờ thiết bị bất thường).
4. Số phút thực học (`focused_minutes` trong `focus_session_participants`) chỉ được server ghi nhận khi client gửi heartbeat định kỳ (mỗi 60s) xác nhận vẫn đang trong voice channel — tránh user gian lận streak bằng cách join rồi tắt app ngay.

### 6.4. Sequence Diagram — Luồng gửi tin nhắn (Message Send Flow)

```
Client A                 API Gateway         Messaging Svc        ScyllaDB          Redis PubSub        Gateway Node(s)      Client B (subscriber)
   │                          │                     │                  │                   │                     │                      │
   │  POST /messages          │                     │                  │                   │                     │                      │
   │─────────────────────────▶│                     │                  │                   │                     │                      │
   │                          │  validate JWT +      │                  │                   │                     │                      │
   │                          │  permission check     │                  │                   │                     │                      │
   │                          │────────────────────▶│                  │                   │                     │                      │
   │                          │                     │  rate-limit check │                   │                     │                      │
   │                          │                     │  (Redis sliding-  │                   │                     │                      │
   │                          │                     │   window)         │                   │                     │                      │
   │                          │                     │──────────────────────────────────────▶│                     │                      │
   │                          │                     │◀──────────────────────────────────────│  OK                 │                      │
   │                          │                     │  persist message  │                   │                     │                      │
   │                          │                     │─────────────────▶│                    │                     │                      │
   │                          │                     │◀─────────────────│  ACK (Snowflake ID)│                     │                      │
   │                          │◀────────────────────│  201 Created       │                   │                     │                      │
   │◀─────────────────────────│  response (message) │                  │                   │                     │                      │
   │                          │                     │  publish event    │                   │                     │                      │
   │                          │                     │  MESSAGE_CREATE   │                   │                     │                      │
   │                          │                     │───────────────────────────────────────▶│  fanout tới node   │                      │
   │                          │                     │                  │                   │  đang giữ conn của  │                      │
   │                          │                     │                  │                   │  subscriber channel │                      │
   │                          │                     │                  │                   │────────────────────▶│                      │
   │                          │                     │                  │                   │                     │  push qua WSS        │
   │                          │                     │                  │                   │                     │─────────────────────▶│
```

---

## 7. CƠ SỞ DỮ LIỆU SCHEMA

> **Ghi chú:** `users`, `guilds`, `channels`, `roles`, `guild_members` dùng PostgreSQL (quan hệ, ACID). `messages` dùng mô hình wide-column (ScyllaDB/Cassandra CQL) để tối ưu ghi và tránh hot partition, tham khảo pattern R2. Snowflake ID (BIGINT 64-bit) dùng làm khóa chính cho hầu hết entity.

### 7.1. PostgreSQL DDL

```sql
-- ============================================
-- USERS
-- ============================================
CREATE TABLE users (
    id              BIGINT PRIMARY KEY,               -- Snowflake ID
    username        VARCHAR(32) NOT NULL,
    discriminator   VARCHAR(4)  NOT NULL,              -- vd: "0001" cho username trùng
    email           VARCHAR(255) NOT NULL UNIQUE,
    password_hash   VARCHAR(255) NOT NULL,             -- bcrypt/argon2id
    display_name    VARCHAR(32),
    avatar_url      TEXT,
    banner_url      TEXT,
    accent_color    INT,
    bio             VARCHAR(190),
    is_verified     BOOLEAN NOT NULL DEFAULT FALSE,
    mfa_enabled     BOOLEAN NOT NULL DEFAULT FALSE,
    mfa_secret      VARCHAR(64),
    status          VARCHAR(16) NOT NULL DEFAULT 'offline', -- online/idle/dnd/invisible/offline
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
    UNIQUE (username, discriminator)
);

-- ============================================
-- GUILDS (SERVERS)
-- ============================================
CREATE TABLE guilds (
    id              BIGINT PRIMARY KEY,
    name            VARCHAR(100) NOT NULL,
    icon_url        TEXT,
    owner_id        BIGINT NOT NULL REFERENCES users(id),
    description     VARCHAR(300),
    verification_level SMALLINT NOT NULL DEFAULT 0,
    boost_count     INT NOT NULL DEFAULT 0,            -- REQ-PREM-10: số lượt boost tích lũy
    boost_tier      SMALLINT NOT NULL DEFAULT 0,        -- REQ-PREM-11: 0/1/2/3
    vanity_url_slug VARCHAR(32) UNIQUE,                 -- REQ-PREM-11: chỉ mở khi boost_tier >= 2
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
    deleted_at      TIMESTAMPTZ                        -- soft-delete, 30-day purge window
);

-- ============================================
-- CATEGORIES & CHANNELS
-- ============================================
CREATE TABLE channels (
    id              BIGINT PRIMARY KEY,
    guild_id        BIGINT NOT NULL REFERENCES guilds(id) ON DELETE CASCADE,
    parent_id       BIGINT REFERENCES channels(id),     -- category cha, NULL nếu là category gốc
    name            VARCHAR(100) NOT NULL,
    type            SMALLINT NOT NULL,                  -- 0=category,1=text,2=voice
    position        INT NOT NULL DEFAULT 0,
    topic           VARCHAR(1024),
    slow_mode_secs  INT NOT NULL DEFAULT 0,
    user_limit      SMALLINT NOT NULL DEFAULT 0,        -- voice channel only, 0=unlimited
    nsfw            BOOLEAN NOT NULL DEFAULT FALSE,
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now()
);
CREATE INDEX idx_channels_guild ON channels(guild_id);

-- ============================================
-- ROLES
-- ============================================
CREATE TABLE roles (
    id              BIGINT PRIMARY KEY,
    guild_id        BIGINT NOT NULL REFERENCES guilds(id) ON DELETE CASCADE,
    name            VARCHAR(100) NOT NULL,
    color           INT NOT NULL DEFAULT 0,
    permissions     BIGINT NOT NULL DEFAULT 0,          -- 64-bit permission bitmap
    position        INT NOT NULL DEFAULT 0,             -- hierarchy: số lớn hơn = ưu tiên cao hơn
    is_hoisted      BOOLEAN NOT NULL DEFAULT FALSE,      -- hiển thị tách nhóm trong member list
    is_mentionable  BOOLEAN NOT NULL DEFAULT TRUE,
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now()
);
CREATE INDEX idx_roles_guild ON roles(guild_id);

-- ============================================
-- CHANNEL PERMISSION OVERWRITES
-- ============================================
CREATE TABLE channel_permission_overwrites (
    channel_id      BIGINT NOT NULL REFERENCES channels(id) ON DELETE CASCADE,
    target_id       BIGINT NOT NULL,                    -- role_id hoặc user_id
    target_type     SMALLINT NOT NULL,                  -- 0=role, 1=member
    allow           BIGINT NOT NULL DEFAULT 0,
    deny            BIGINT NOT NULL DEFAULT 0,
    PRIMARY KEY (channel_id, target_id, target_type)
);

-- ============================================
-- GUILD MEMBERS
-- ============================================
CREATE TABLE guild_members (
    guild_id        BIGINT NOT NULL REFERENCES guilds(id) ON DELETE CASCADE,
    user_id         BIGINT NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    nickname        VARCHAR(32),
    joined_at       TIMESTAMPTZ NOT NULL DEFAULT now(),
    is_deafened     BOOLEAN NOT NULL DEFAULT FALSE,
    is_muted        BOOLEAN NOT NULL DEFAULT FALSE,
    timeout_until   TIMESTAMPTZ,
    PRIMARY KEY (guild_id, user_id)
);

CREATE TABLE guild_member_roles (
    guild_id        BIGINT NOT NULL,
    user_id         BIGINT NOT NULL,
    role_id         BIGINT NOT NULL REFERENCES roles(id) ON DELETE CASCADE,
    PRIMARY KEY (guild_id, user_id, role_id),
    FOREIGN KEY (guild_id, user_id) REFERENCES guild_members(guild_id, user_id) ON DELETE CASCADE
);

-- ============================================
-- FRIENDSHIPS (Kết bạn) & BLOCK LIST
-- ============================================
-- Lưu 1 hàng/cặp, chuẩn hóa (user_id_low < user_id_high) để tránh trùng lặp 2 chiều
CREATE TABLE friendships (
    user_id_low     BIGINT NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    user_id_high    BIGINT NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    status          SMALLINT NOT NULL,       -- 0=pending, 1=accepted
    requested_by    BIGINT NOT NULL REFERENCES users(id),
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
    responded_at    TIMESTAMPTZ,
    PRIMARY KEY (user_id_low, user_id_high),
    CHECK (user_id_low < user_id_high)
);

CREATE TABLE user_blocks (
    blocker_id      BIGINT NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    blocked_id      BIGINT NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
    PRIMARY KEY (blocker_id, blocked_id)
);

-- ============================================
-- DM CHANNELS (1-1 & Group DM — độc lập với guilds)
-- ============================================
CREATE TABLE dm_channels (
    id              BIGINT PRIMARY KEY,
    type            SMALLINT NOT NULL,       -- 1=DM 1-1, 3=GROUP_DM
    name            VARCHAR(100),            -- chỉ dùng cho GROUP_DM (đặt tên nhóm)
    icon_url        TEXT,                    -- chỉ dùng cho GROUP_DM
    owner_id        BIGINT REFERENCES users(id),  -- chỉ dùng cho GROUP_DM (người tạo)
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE TABLE dm_channel_members (
    dm_channel_id   BIGINT NOT NULL REFERENCES dm_channels(id) ON DELETE CASCADE,
    user_id         BIGINT NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    joined_at       TIMESTAMPTZ NOT NULL DEFAULT now(),
    PRIMARY KEY (dm_channel_id, user_id)
);
-- Ràng buộc ứng dụng-tầng (application-level, không thể diễn đạt thuần SQL):
--   type=1 (DM 1-1)     -> đúng 2 thành viên, không đổi.
--   type=3 (GROUP_DM)   -> tối thiểu 2, tối đa 10 thành viên (REQ-GDM-01/03).

-- ============================================
-- CALL SESSIONS (Voice/Video 1-1 & Group DM)
-- ============================================
CREATE TABLE call_sessions (
    id              BIGINT PRIMARY KEY,
    dm_channel_id   BIGINT NOT NULL REFERENCES dm_channels(id) ON DELETE CASCADE,
    initiator_id    BIGINT NOT NULL REFERENCES users(id),
    started_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
    ended_at        TIMESTAMPTZ,
    status          VARCHAR(16) NOT NULL DEFAULT 'ringing', -- ringing/active/missed/declined/ended
    sfu_room_id     VARCHAR(64)              -- room reference tại SFU nhẹ cho DM (khác voice channel guild)
);

-- ============================================
-- STUDY / FOCUS MODE (REQ-STUDY-*)
-- ============================================
ALTER TABLE guilds ADD COLUMN template VARCHAR(20) DEFAULT 'general';  -- 'general' | 'study_server' (REQ-STUDY-05)
ALTER TABLE guilds ADD COLUMN topic_tags TEXT[];                       -- vd: {'giai-tich-2','ky-thuat-phan-mem'} (REQ-STUDY-07)
ALTER TABLE guilds ADD COLUMN leaderboard_enabled BOOLEAN DEFAULT TRUE; -- Owner bật/tắt (REQ-STUDY-04)

CREATE TABLE focus_sessions (
    id              BIGINT PRIMARY KEY,
    channel_id      BIGINT NOT NULL REFERENCES channels(id) ON DELETE CASCADE,
    started_by      BIGINT NOT NULL REFERENCES users(id),
    cycle_focus_min SMALLINT NOT NULL DEFAULT 25,   -- REQ-STUDY-01: 25/50/tùy chỉnh
    cycle_break_min SMALLINT NOT NULL DEFAULT 5,
    server_started_at TIMESTAMPTZ NOT NULL DEFAULT now(), -- mốc gốc để đồng bộ đếm ngược mọi client
    ended_at        TIMESTAMPTZ
);

-- Ghi nhận thời lượng thực tế từng user tham gia 1 focus session (phục vụ REQ-STUDY-03/04)
CREATE TABLE focus_session_participants (
    focus_session_id BIGINT NOT NULL REFERENCES focus_sessions(id) ON DELETE CASCADE,
    user_id           BIGINT NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    joined_at         TIMESTAMPTZ NOT NULL DEFAULT now(),
    left_at           TIMESTAMPTZ,
    focused_minutes   INT NOT NULL DEFAULT 0,        -- tổng phút thực học (không tính break)
    PRIMARY KEY (focus_session_id, user_id)
);

CREATE TABLE user_study_streaks (
    user_id             BIGINT PRIMARY KEY REFERENCES users(id) ON DELETE CASCADE,
    current_streak_days INT NOT NULL DEFAULT 0,
    longest_streak_days INT NOT NULL DEFAULT 0,
    last_focus_date     DATE,                        -- theo giờ địa phương user, dùng để tính reset streak
    show_on_leaderboard BOOLEAN NOT NULL DEFAULT TRUE -- opt-out cá nhân (REQ-STUDY-04)
);

CREATE TABLE task_boards (
    channel_id      BIGINT PRIMARY KEY REFERENCES channels(id) ON DELETE CASCADE,
    enabled         BOOLEAN NOT NULL DEFAULT FALSE   -- REQ-STUDY-06
);

CREATE TABLE tasks (
    id              BIGINT PRIMARY KEY,
    channel_id      BIGINT NOT NULL REFERENCES channels(id) ON DELETE CASCADE,
    title           VARCHAR(200) NOT NULL,
    assignee_id     BIGINT REFERENCES users(id),      -- NULL = chưa gán, nhắc toàn kênh
    status          SMALLINT NOT NULL DEFAULT 0,       -- 0=todo,1=doing,2=done
    due_at          TIMESTAMPTZ,
    created_by      BIGINT NOT NULL REFERENCES users(id),
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now()
);
CREATE INDEX idx_tasks_assignee_due ON tasks(assignee_id, due_at);  -- phục vụ REQ-STUDY-08 (Study Digest tổng hợp)

-- ============================================
-- INVITES
-- ============================================
CREATE TABLE invites (
    code            VARCHAR(16) PRIMARY KEY,
    guild_id        BIGINT NOT NULL REFERENCES guilds(id) ON DELETE CASCADE,
    channel_id      BIGINT NOT NULL REFERENCES channels(id) ON DELETE CASCADE,
    creator_id      BIGINT NOT NULL REFERENCES users(id),
    max_uses        INT NOT NULL DEFAULT 0,             -- 0 = unlimited
    uses            INT NOT NULL DEFAULT 0,
    expires_at      TIMESTAMPTZ,                        -- NULL = never expires
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now()
);

-- ============================================
-- AUDIT LOG (append-only)
-- ============================================
CREATE TABLE audit_logs (
    id              BIGINT PRIMARY KEY,
    guild_id        BIGINT NOT NULL REFERENCES guilds(id) ON DELETE CASCADE,
    actor_id        BIGINT NOT NULL REFERENCES users(id),
    target_id       BIGINT,
    action_type     VARCHAR(50) NOT NULL,               -- MEMBER_BAN, ROLE_UPDATE, CHANNEL_DELETE, ...
    reason          VARCHAR(512),
    changes_json    JSONB,
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now()
);
CREATE INDEX idx_audit_guild_time ON audit_logs(guild_id, created_at DESC);
```

### 7.2. Message Store Schema (ScyllaDB / Cassandra CQL)

> Thiết kế partition theo `channel_id` để đảm bảo truy vấn "N tin nhắn gần nhất của 1 channel" hiệu quả (single-partition read). Để giảm thiểu hot partition với channel cực lớn (tham khảo R2), áp dụng **bucket theo thời gian** (vd: theo tháng) làm một phần partition key.

```sql
CREATE TABLE messages (
    channel_id      BIGINT,
    bucket          TEXT,          -- vd: "2026-09" — time bucket để tránh hot partition
    message_id      BIGINT,        -- Snowflake ID, chứa timestamp -> dùng làm clustering key DESC
    author_id       BIGINT,
    content         TEXT,
    attachments     LIST<TEXT>,    -- danh sách URL trên object storage
    embeds          TEXT,          -- JSON serialized
    mentions        LIST<BIGINT>,
    reply_to_id     BIGINT,
    edited_at       TIMESTAMP,
    deleted         BOOLEAN,
    reactions       MAP<TEXT, INT>,  -- emoji -> count
    PRIMARY KEY ((channel_id, bucket), message_id)
) WITH CLUSTERING ORDER BY (message_id DESC);
```

- **Request coalescing:** với truy vấn "lấy N tin nhắn mới nhất" trùng lặp trong khoảng thời gian ngắn (nhiều client cùng mở 1 channel hot), tầng Messaging Service áp dụng cơ chế coalesce request trùng nhau thành 1 lần đọc DB thực tế, phân phối kết quả cho tất cả request đang chờ — theo đúng pattern mô tả trong R2.
- **Dùng chung cho DM:** bảng `messages` ở trên dùng chung schema cho cả guild channel lẫn `dm_channels` — cột `channel_id` chấp nhận cả `channels.id` (guild) và `dm_channels.id` (DM/Group DM), vì về bản chất cả hai đều là "một tập tin nhắn thuộc 1 ID kênh duy nhất". Điều này giúp REQ-DM-01 tái sử dụng 100% pipeline persist/fanout của REQ-MSG-01 mà không cần service riêng.

---

## 9. BACKLOG (POST-MVP) — GÓI PREMIUM & TÍNH NĂNG MỞ RỘNG

> Các mục trong chương này **không nằm trong MVP 1.0**, được đặc tả trước để đội Product/Backend không cần thiết kế lại từ đầu khi triển khai phase sau. Priority ở đây dùng thang riêng: **P1 (làm ngay sau MVP)** / **P2 (trung hạn)** / **P3 (dài hạn, cân nhắc theo nhu cầu thực tế)**.

### 9.1. Gói thuê bao Premium — "Studely+" (tương đương Nitro)

#### 9.1.1. Hạ tầng thanh toán & Entitlement (nền tảng bắt buộc trước khi có bất kỳ tính năng trả phí nào)

| REQ-ID | Tên chức năng | Mô tả chi tiết | Priority | Acceptance Criteria |
|---|---|---|---|---|
| REQ-PREM-00 | Payment Gateway Integration | Tích hợp cổng thanh toán (Stripe/VNPay/Momo tùy thị trường), hỗ trợ thanh toán định kỳ (subscription) theo tháng/năm, xử lý webhook renewal/failed-payment/cancellation. | P1 | Given user đăng ký gói tháng, When thanh toán thành công, Then webhook cập nhật `subscription.status=active` trong vòng 5 giây và entitlement được cấp ngay. |
| REQ-PREM-01 | Entitlement Service | Service riêng lưu trạng thái quyền lợi premium của user (active/expired/grace-period), được mọi service khác (Guild, Messaging, Voice) truy vấn qua cache Redis để bật/tắt tính năng theo thời gian thực, không phải deploy lại. | P1 | Given subscription hết hạn, When entitlement service cập nhật, Then toàn bộ tính năng premium của user bị vô hiệu trong vòng 1 phút (không cần user logout/login lại). |
| REQ-PREM-02 | Grace Period & Downgrade an toàn | Khi thanh toán thất bại, user có 3 ngày grace period trước khi hạ cấp; nội dung đã tạo bằng quyền premium (vd: avatar động, emoji đã dùng) không bị xóa nhưng mất hiệu lực hiển thị đặc biệt (vd: avatar động trả về khung tĩnh). | P2 | Given hết grace period, When downgrade thực thi, Then avatar động của user hiển thị lại frame tĩnh đầu tiên, dữ liệu gốc vẫn được giữ nếu user gia hạn lại trong 30 ngày. |

#### 9.1.2. Tính năng cá nhân hóa (Personal Perks)

| REQ-ID | Tên chức năng | Mô tả chi tiết | Priority | Acceptance Criteria |
|---|---|---|---|---|
| REQ-PREM-03 | Avatar/Banner động (GIF) | User Premium upload avatar/banner dạng GIF động, user thường chỉ upload ảnh tĩnh. | P1 | Given user không có Premium, When upload avatar dạng GIF, Then hệ thống từ chối và gợi ý nâng cấp gói. |
| REQ-PREM-04 | Emoji tùy chỉnh dùng xuyên server | User Premium có thể dùng bất kỳ custom emoji nào từ mọi guild mình đã tham gia trong bất kỳ channel/DM nào (không giới hạn emoji của riêng guild đang chat). | P1 | Given user Premium từng thấy emoji `:pepehappy:` từ guild A, When chat trong guild B (không có emoji đó), Then vẫn gõ và gửi được emoji đó dưới dạng ảnh thực (không phải Unicode fallback). |
| REQ-PREM-05 | Giới hạn upload file cao hơn | Nâng giới hạn file đính kèm từ 25MB (REQ-MSG-04 mặc định) lên 500MB cho user Premium. | P1 | Given user Premium upload file 300MB, When gửi, Then hệ thống chấp nhận thay vì trả `40013 FileTooLarge`. |
| REQ-PREM-06 | Video/Screen-share chất lượng cao | Cho phép stream màn hình độ phân giải 4K@60fps thay vì giới hạn 1080p60 mặc định (REQ-VC-03), ưu tiên băng thông ở tầng SFU cho user Premium. | P2 | Given user Premium share màn hình, When kiểm tra stream, Then negotiate resolution tối đa 4K nếu băng thông đủ, fallback 1080p nếu mạng yếu (không lỗi cứng). |
| REQ-PREM-07 | Huy hiệu & Hiệu ứng hồ sơ (Profile Badge/Decoration) | Hiển thị badge Premium cạnh username, cho phép chọn avatar decoration (khung viền động quanh avatar) và theme màu profile riêng. | P2 | Given user Premium chọn 1 avatar decoration, When người khác xem profile, Then decoration hiển thị đồng bộ trên mọi client (web/desktop/mobile). |
| REQ-PREM-08 | Tăng số lượng Group DM tham gia | User thường giới hạn tham gia N group DM đồng thời (vd: 10 nhóm) theo REQ-GDM, user Premium không bị giới hạn này. | P3 | Given user thường đã ở 10 group DM, When được mời thêm 1 group, Then hệ thống từ chối kèm gợi ý nâng cấp; user Premium không bị chặn. |
| REQ-PREM-09 | Message History không giới hạn cho tài khoản cá nhân | User Premium có thể export/tìm kiếm toàn bộ lịch sử tin nhắn cá nhân không giới hạn thời gian (user thường chỉ search trong phạm vi X tháng gần nhất tùy chính sách retention). | P3 | Given user Premium search tin nhắn cũ hơn 12 tháng, When submit query, Then kết quả vẫn trả về đầy đủ; user thường nhận thông báo "Nâng cấp để tìm kiếm lịch sử đầy đủ". |

#### 9.1.3. Tính năng "Server Boost" (Premium áp cho cả Guild)

| REQ-ID | Tên chức năng | Mô tả chi tiết | Priority | Acceptance Criteria |
|---|---|---|---|---|
| REQ-PREM-10 | Boost Guild bằng Premium cá nhân | User Premium có thể "tặng" 1-2 lượt boost/tháng cho 1 guild bất kỳ mình tham gia; guild tích lũy đủ mốc boost (vd: 3/7/14 boost) sẽ mở khóa tier perks tương ứng. | P2 | Given guild đạt 3 boost, When kiểm tra guild tier, Then `guilds.boost_tier` chuyển từ 0 lên 1 và các perk tier 1 (vd: thêm 1 emoji slot, chất lượng voice 128kbps) được kích hoạt ngay cho toàn bộ member. |
| REQ-PREM-11 | Perks theo Boost Tier | Tier 1: +50 emoji slot, voice bitrate 128kbps, 1 role icon tùy chỉnh. Tier 2: +100 emoji slot, voice bitrate 256kbps, banner guild động, vanity URL invite (`studely.app/tenguild`). Tier 3: +250 emoji slot, voice bitrate 384kbps, upload limit guild-wide 100MB. | P2 | Given guild đạt Tier 2, When Owner cấu hình vanity URL, Then link `studely.app/<slug>` hoạt động thay cho invite code ngẫu nhiên. |
| REQ-PREM-12 | Huy hiệu "Booster" trong Member List | Member đã boost guild hiển thị badge riêng trong member list và role tự động "Server Booster" (màu tùy Owner cấu hình). | P3 | Given user boost guild, When xem member list, Then user đó có badge boost cạnh tên, phân nhóm cùng role Server Booster nếu Owner đã bật hoist cho role đó. |

---

## 10. RÀ SOÁT ĐỘ PHỦ SO VỚI DISCORD THẬT (FEATURE GAP ANALYSIS)

### 10.1. Đã đặc tả đầy đủ trong SRS

| Nhóm | Trạng thái |
|---|---|
| Auth/2FA/Presence | ✅ Đầy đủ (4.1) |
| Guild/Channel CRUD, Invite | ✅ Đầy đủ (4.2) |
| RBAC, Permission Overwrite, Audit Log | ✅ Đầy đủ (4.3) |
| Messaging text (send/edit/delete/reply/react/mention/attachment/slowmode) | ✅ Đầy đủ (4.4) |
| Voice/Video guild (SFU, VAD, screen share, mute/deafen, multi-region) | ✅ Đầy đủ (4.5) |
| Search & Notification | ✅ Đầy đủ (4.6) |
| Friends/Block/DM 1-1 | ✅ Đầy đủ (4.7) |
| Group DM & Call 1-1/nhóm nhỏ | ✅ Đầy đủ (4.8) |
| Premium/Nitro/Server Boost | ✅ Đã đưa vào Backlog (9.1) |

### 10.2. Còn thiếu so với Discord thật — đề xuất bổ sung Backlog

Sau khi rà soát lại toàn bộ tương tác cốt lõi của Discord, các nhóm sau **chưa được đặc tả** trong SRS hiện tại. Đề xuất đưa vào Backlog theo mức ưu tiên:

| REQ-ID | Tên chức năng | Mô tả ngắn | Vì sao quan trọng | Priority đề xuất |
|---|---|---|---|---|
| REQ-BKLG-01 | Threads (chủ đề phụ) | Tạo nhánh hội thoại con từ 1 tin nhắn gốc trong channel, không làm loãng luồng chat chính, có thể archive tự động sau X ngày không hoạt động. | Tính năng lõi để quản lý hội thoại đông người — thiếu sẽ khiến channel lớn rất hỗn loạn. | P1 |
| REQ-BKLG-02 | Ghim tin nhắn (Pinned Messages) | Ghim tối đa 50 tin nhắn/channel, hiển thị trong panel riêng. | Tương tác rất cơ bản, người dùng kỳ vọng có ngay từ đầu. | P1 |
| REQ-BKLG-03 | Custom Emoji & Sticker theo Guild | Owner/Admin upload emoji/sticker riêng cho guild (giới hạn slot theo boost tier — liên kết REQ-PREM-11), dùng được trong reaction/message. | Gắn liền văn hóa cộng đồng Discord, ảnh hưởng trực tiếp tới REQ-PREM-04. | P1 |
| REQ-BKLG-04 | Webhooks | Cho phép guild tạo webhook URL để hệ thống bên ngoài (CI/CD, GitHub, cron job) gửi tin nhắn tự động vào 1 channel cụ thể. | Nhu cầu tích hợp rất phổ biến (thông báo build, alert). | P2 |
| REQ-BKLG-05 | Bot Framework & Slash Commands | API cho phép đăng ký bot (OAuth2 application), bot nhận event qua Gateway riêng hoặc Interaction endpoint, hỗ trợ slash command (`/ping`, `/poll`). | Hệ sinh thái bot là một phần cốt lõi khiến Discord khác biệt với ứng dụng chat thường. | P2 |
| REQ-BKLG-06 | AutoMod (kiểm duyệt tự động) | Bộ lọc từ khóa/spam/link tự động chặn hoặc cảnh báo trước khi tin nhắn hiển thị, cấu hình theo guild. | Giảm tải cho Moderator, đặc biệt quan trọng với guild đông người. | P2 |
| REQ-BKLG-07 | Stage Channels | Kênh voice dạng "hội trường" — phân biệt rõ Speaker/Audience, Audience phải "giơ tay" mới được cấp quyền nói. | Dùng cho sự kiện/AMA/town-hall — khác hẳn voice channel thông thường. | P3 |
| REQ-BKLG-08 | Forum Channels | Channel dạng diễn đàn: mỗi "post" là 1 thread riêng có tiêu đề, tag phân loại, thay vì luồng chat tuyến tính. | Phù hợp guild dạng Q&A/hỗ trợ kỹ thuật. | P3 |
| REQ-BKLG-09 | Server Discovery / Public Guild Directory | Cho phép Owner công khai guild lên danh mục tìm kiếm public, có preview trước khi join. | Kênh tăng trưởng người dùng mới quan trọng nếu định hướng cộng đồng mở. | P3 |
| REQ-BKLG-10 | Announcement Channel + Follow | Channel dạng thông báo, guild khác có thể "Follow" để tin nhắn tự động crosspost sang channel của họ. | Phổ biến cho guild dạng tin tức/dự án lớn có nhiều cộng đồng vệ tinh. | P3 |
| REQ-BKLG-11 | Onboarding & Membership Screening | Luồng hỏi đáp/đồng ý quy tắc bắt buộc trước khi member mới được chat, gợi ý channel/role theo sở thích. | Cải thiện trải nghiệm nhập môn cho guild lớn, giảm spam tài khoản rác. | P2 |
| REQ-BKLG-12 | Báo cáo & An toàn (Report/Trust & Safety) | User report tin nhắn/user vi phạm gửi thẳng tới hàng đợi kiểm duyệt nền tảng (không chỉ Moderator guild), kèm ảnh chụp ngữ cảnh tự động. | Bắt buộc về mặt pháp lý/an toàn nền tảng khi mở rộng người dùng thật, không thể bỏ qua lâu dài. | P1 |
| REQ-BKLG-13 | Custom Status & Hoạt động đang chơi (Rich Presence) | Hiển thị "Đang chơi [Game]"/"Đang nghe [Spotify]" cạnh tên user, tùy chỉnh custom status text + emoji. | Tương tác xã hội nhẹ nhưng rất đặc trưng, ảnh hưởng cảm giác "sống" của app. | P3 |
| REQ-BKLG-14 | Read Receipts trong DM | Hiển thị "Đã xem" cho tin nhắn DM 1-1 (khác với unread badge guild-level đã có ở REQ-NOTIF-01). | Discord thật KHÔNG có read receipt hiển thị công khai (chỉ có unread state) — liệt kê ở đây để nhóm **cân nhắc có nên làm khác Discord** (xem gợi ý ở phần định hướng khác biệt bên dưới). | P3 (tùy chọn) |

> **Lưu ý:** REQ-BKLG-01/02/03/12 nên cân nhắc đưa thẳng vào MVP nếu còn dư thời gian, vì đây là các tương tác người dùng kỳ vọng có ngay ("bare minimum" so với 1 app chat hiện đại) chứ không thuần là tính năng nâng cao.



## 11. PHỤ LỤC (APPENDIX)

### 11.1. Bảng mã lỗi (Error Codes)

| Mã lỗi | HTTP Status | Ý nghĩa |
|---|---|---|
| 40001 | 400 | Dữ liệu request không hợp lệ (validation error). |
| 40013 | 400 | File vượt quá kích thước cho phép (`FileTooLarge`). |
| 40029 | 429 | Vượt giới hạn tần suất (`RateLimited` / `SlowModeRateLimited`), kèm header `Retry-After`. |
| 40041 | 404/410 | Invite đã hết hạn hoặc hết lượt sử dụng. |
| 40051 | 400 | Voice channel đã đạt giới hạn user tối đa. |
| 40100 | 401 | Token không hợp lệ hoặc đã hết hạn. |
| 40101 | 401 | Sai mã 2FA/TOTP. |
| 40301 | 403 | Vi phạm role hierarchy (`InsufficientRoleHierarchy`). |
| 40302 | 403 | Không đủ quyền thực hiện hành động (`Forbidden`). |
| 40401 | 404 | Không tìm thấy tài nguyên (user/guild/channel/message). |
| 40901 | 409 | Xung đột dữ liệu (vd: username đã tồn tại, refresh token tái sử dụng). |
| 50000 | 500 | Lỗi hệ thống nội bộ không xác định. |
| 50301 | 503 | Dịch vụ SFU/Voice Gateway tạm thời không khả dụng. |

### 11.2. Bảng thuật ngữ mở rộng (Glossary)

| Thuật ngữ | Giải thích ngắn gọn |
|---|---|
| Simulcast | Kỹ thuật gửi đồng thời nhiều chất lượng (layer) của cùng 1 video stream để SFU chọn layer phù hợp cho từng receiver. |
| Snowflake ID | ID 64-bit duy nhất toàn cục, tự chứa timestamp tạo ra, giúp sort theo thời gian mà không cần truy vấn thêm. |
| Soft-delete | Đánh dấu bản ghi là đã xóa (cờ `deleted`/`deleted_at`) thay vì xóa vật lý, phục vụ khôi phục hoặc audit. |
| Sliding-window Rate Limit | Thuật toán giới hạn tần suất dựa trên cửa sổ thời gian trượt liên tục, chính xác hơn fixed-window ở biên thời gian. |
| Request Coalescing | Kỹ thuật gộp nhiều request giống nhau xảy ra gần như đồng thời thành 1 lần xử lý thực tế duy nhất. |
| Cohort Analysis | Phân tích hành vi người dùng theo nhóm (cohort) có cùng đặc điểm thời gian tham gia, phục vụ đo retention. |

---

*Hết tài liệu SRS — Studely v1.0*