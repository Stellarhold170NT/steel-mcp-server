# PLAYWRIGHT vs STEEL MCP — Bảng so sánh năng lực (parity matrix)

> Nguồn Playwright: tài liệu chính thức playwright.dev/docs/api/* (đã fetch) + kho nguồn mở
> microsoft/playwright. Nguồn Steel: mã nguồn steel-mcp-server@3ceb5c3 (10 action + 17 tool
> đã đọc trực tiếp). Thang verdict:
> ✅ steel làm được tương đương (khác cú pháp) · ⚠️ làm được một phần / qua đường vòng ·
> ❌ chưa có → trỏ GAP-# trong BUGS-AND-GAPS.md.

## 1. Thông dyện năng Playwright

| Nhóm | API tiêu biểu (class Page/Locator/BrowserContext) |
|---|---|
| Điều hướng | goto, goBack, goForward, reload, waitForURL, waitForLoadState |
| Đọc trang | content, title, innerText, getAttribute, ariaSnapshot, accessibility |
| Locators | getByRole, getByText, getByLabel, getByPlaceholder, getByAltText, getByTitle, getByTestId, css, xpath, filter, nth, and/or, locator.strict |
| Chuột/ bàn phím | click, dblclick, fill, type/pressSequentially, check, uncheck, selectOption, hover, focus, blur, press, dragAndDrop, tap, setInputFiles |
| Chờ đợi | waitForSelector, waitFor, auto-waiting actionability (visible/stable/enabled/editable) |
| Khung phẳng | frameLocator, frame转基因 tree, shadow DOM piercing (mặc định) |
| Đa ngữ cảnh | newContext, storageState xuất/nạp, cookies, permissions, geolocation, locale, timezone, colorScheme, reducedMotion, device/DPR, httpCredentials, offline, extraHTTPHeaders |
| Network | route/abort/continue/fallback, request/response/waitForResponse, HAR record/replay, websocket events |
| Bằng chứng | screenshot (page/element/clip/fullPage), pdf, video, tracing (trace.zip), coverage |
| Sự kiện trang | console, pageerror, dialog, download, popup, filechooser, requestfailed |
| JS runtime | evaluate, evaluateHandle, exposeFunction/exposeBinding, addInitScript, workers |
| Thời gian | clock (install/pause/fastForward), setDefaultTimeout |
| CDP | newCDPSession (truy cập thô mọi domain) |

## 2. Bảng parity — "Steel có làm y hệt không?"

### 2.1 Điều hướng & chờ

| Playwright | Steel MCP | Verdict |
|---|---|---|
| page.goto(url) | steel_navigate + settle | ✅ |
| goBack/goForward | act(go_back) / navigate | ✅ (go_back) · ⚠️ goForward qua navigate |
| reload | navigate cùng URL | ⚠️ |
| waitForURL/loadState | steel_wait_for(url/text/selector) | ✅ |
| auto-wait actionability | settle + scrollIntoViewIfNeeded trước click | ⚠️ encode trong pipeline, agent không tắt được |

### 2.2 Tương tác (user thật cầm chuột)

| Playwright | Steel MCP | Verdict |
|---|---|---|
| click / dblclick | act(click) — 5 điểm mẫu trên quad + hit-test gọi tên blocker | ⚠️ →✅ sau P1 (BUG-1) |
| fill / pressSequentially | act(type) / fill_form | ✅ |
| check / uncheck | act(check) | ⚠️ →✅ sau P1+P2 (BUG-1/3) |
| selectOption | act(select) — dispatch input+change | ✅ |
| hover | act(hover) | ✅ |
| press / modifiers | act(press) | ⚠️ tổ hợp phím bảo mật hạn chế theo whitelist |
| scrollIntoViewIfNeeded | tự gọi trước click | ✅ |
| scroll | act(scroll) | ⚠️ window-only (BUG-4) |
| dragAndDrop / tap | — | ❌ GAP-1 / GAP-9 |
| setInputFiles | — | ❌ GAP-2 |

### 2.3 Đọc & tìm phần tử

| Playwright | Steel MCP | Verdict |
|---|---|---|
| getByRole/Text/Label/Placeholder/TestId + filter/nth | steel_find (query text/role) + @eN | ⚠️ vai trò cấp Locators语言 nghèo hơn (P3 giúp) |
| css / xpath | đánh vào snapshot tree | ⚠️ |
| ariaSnapshot | steel_snapshot (tree + @eN + checked) | ✅ (thậm chí budget 1500 node, redact) |
| framepiercing + shadow | cross-frame tự động trong snapshot/act | ✅ |
| innerText/content/getAttribute | steel_scrape (markdown/html) | ✅ |
| strict mode báo lỗi "resolved to N" | find báo 0/nhiều | ⚠️ |

### 2.4 Ngữ cảnh & thiết bị

| Playwright | Steel MCP | Verdict |
|---|---|---|
| newContext() song song | nhiều session_create | ⚠️ mỗi session 1 browser (GAP-8 cho tab) |
| storageState xuất/nạp | profile persist phía server | ⚠️ agent không xuất/nạp được → GAP-7 |
| cookies/permissions API | — | ❌ GAP-7 |
| geolocation/locale/timezone/colorScheme/DPR/device | session_options(device) + credentials | ⚠️ chỉ một phần → GAP-9 |
| httpCredentials | session_options(credentials) | ✅ |
| offline/throttle | — | ❌ GAP-11 |

### 2.5 Bằng chứng & quan sát

| Playwright | Steel MCP | Verdict |
|---|---|---|
| screenshot fullPage/clip | steel_screenshot URL hoắc session | ⚠️ element-clip ❌ → GAP-10 |
| pdf | steel_pdf | ✅ (lazy-render cần GAP-14) |
| video / tracing | replay (dashboard riêng) | ⚠️ export trace.zip/mp4 → GAP-12 |
| console/pageerror stream | — | ❌ GAP-6 |
| request/response/HAR | — | ❌ GAP-5 |

### 2.6 Sự kiện & nâng cao

| Playwright | Steel MCP | Verdict |
|---|---|---|
| dialog accept/dismiss | — | ❌ GAP-4 |
| download capture | — | ❌ GAP-3 |
| popup handling | — | ❌ GAP-8 |
| evaluate/exposeBinding | — (chủ đích không ship — untrusted) | ❌ theo policy → xem xét GAP-5/6 thay thế an toàn |
| clock API | — | ❌ GAP-11 |
| newCDPSession | không expose cho agent | ❌ (chủ đích) |

### 2.7 Con người & vòng đời — Steel có, Playwright không có sẵn

| Năng lực | Steel MCP |
|---|---|
| live-view người xem song song bot | steel live-view URL |
| handoff người gánh giữa dòng (CAPTCHA/OTP/thanh toán) | steel_session_handoff |
| replay phiên đã tắt để chẩn đoán | steel_session_replay |
| ngân sách token / redact nhạy cảm / defang link | encode trong snapshot |
| lỗi tự mô tả ("bị che bởi X — đổi chiến thuật") | errors.ts triết lý thất bại tự sửa |

## 3. KẾT LUẬN — "giống 1 user thật đang cầm chuột?"

- **Hiện tại (v3.0.0 tầm 3ceb5c3):** Steel đã ✅/⚠️ **41 năng lực cốt lõi**, ❌ **19** (lệch
  chủ yếu ở: drag, upload/download, network/console, emulation sâu, dialogs, đa tab).
- **Sau P1–P5 (REVIEW-VI.md) + 15 GAP** (BUGS-AND-GAPS.md): mọi hàng ⚠️ của bảng 2.2 trở ✅ —
  tức **toàn bộ dải "user thật cầm chuột" trên DOM-reachable** đạt parity Playwright, đồng
  thời giữ 5 năng lực chắc 2.7 mà Playwright không có (human-in-loop là tính năng).
- **Không bao giờ parity tuyệt đối 100%** — hai dải còn lại là chủ đích: (a) evaluate thô bị
  từ chối vì mô hình đe dọa untrusted; (b) isTrusted/CAPTCHA là ranh giới web chống bot —
  ranh giới này Steel chuyển người xử lý qua handoff thay vì "vượt bằng mọi giá".
