# PLAYWRIGHT CORE — INVENTORY ĐẦY ĐỦ METHOD + PARITY STEEL MCP

> Trích xuất trực tiếp playwright.dev (fetch sống, bảng mục lục từng class, đếm thật):
> **Page 106 · Locator 71 · BrowserContext 36 · ElementHandle 33 · Frame ~60 · Keyboard/Mouse/Touchscreen · Dialog 5 · Download 6** + các lớp卫 tinh (Console/Request/Response/Route/Cookie/CDPSession/APIRequestContext/Tracing/Video/Worker/Clock).
> Verdict: ✅ steel làm được · ⚠️ một phần (lý do kèm) · ❌ chưa có (trỏ GAP-n trong BUGS-AND-GAPS.md).

## 1. LOCATOR (71 methods)

### ✅ tương đương (qua steel_snapshot/find + @eN + act)
- **Tạo/định vị:** locator · nth · first · last · and · or · filter · count · all · getByRole · getByText · getByLabel · getByPlaceholder · getByAltText · getByTitle · getByTestId · page · frameLocator · contentFrame · describe · description · toString
- **Đọc:** allInnerTexts · allTextContents · innerText · innerHTML · textContent · getAttribute · inputValue · ariaSnapshot · ariaSnapshotJSON · boundingBox · isChecked · isDisabled · isEditable · isEnabled · isHidden · isVisible · visible
- **Hành động:** click · fill · focus · hover · press · pressSequentially · selectOption · check · uncheck · setChecked · clear · scrollIntoViewIfNeeded · waitFor (find/wait_for) · highlight/hideHighlight (live-view cho người)
### ⚠️ một phần
- **dblclick** → act(click) thiếu clickCount=2 (GAP-9) · **tap** → cần touch device (GAP-9) · **selectText/screenshot** → screenshot có, clip-theo-element chưa (GAP-10) · **type** (typing chậm kbell) → act(type) gần đúng
### ❌ chưa có
- **dragTo · drop** (GAP-1) · **setInputFiles** (GAP-2) · **dispatchEvent** (policy-untrusted) · **evaluate · evaluateAll · evaluateHandle · elementHandle · elementHandles** (policy-untrusted) · **waitForFunction** (policy)

## 2. PAGE (106 methods)

### ✅ tương đương
- **Điều hướng:** goto · reload(=navigate lại URL) · goBack(act go_back) · waitForURL · waitForLoadState(settle) · url · title · content(scrape html) · setFrameViewport≈(session_options viewport)
- **Đọc/a11y:** ariaSnapshot · ariaSnapshotJSON(snapshot tree) · getBy*(thế bằng steel_find) · locator · mainFrame · frames · frame · frameLocator · opener · context · isClosed(session_diagnostics)
- **Bằng chứng:** screenshot(session/URL capture) · pdf(steel_pdf) · bringToFront(live-view cho người)
- **Chờ:** waitForSelector/waitForNavigation(steel_wait_for text/url/selector) · waitForTimeout(settle/batch pacing)
- **Tương tác kế thừa Locator:** click · fill · focus · hover · press · type · check · uncheck · setChecked · selectOption · getAttribute · innerText/HTML · textContent · inputValue · isChecked/isDisabled/isEditable/isEnabled/isHidden/isVisible · waitFor(wait_for)
- **Tổ chức:** pause(≈live-view người lái) · pickLocator(≈live-view + find) · removeAllListeners(—) · close(session_release) · setDefaultTimeout/​setDefaultNavigationTimeout(budget nội bộ)
### ⚠️ một phần
- **dblclick · tap** như Locator (GAP-9) · **screenshot clip/element** (GAP-10) · **setExtraHTTPHeaders** (chỉ cấp session, không per-context — GAP-9) · **emulateMedia** (colorScheme một phần qua GAP-9) · **video/screencast** (≈replay dashboard, không file mp4 — GAP-12)
### ❌ chưa có
- **dragAndDrop** (GAP-1) · **setInputFiles** (GAP-2) · **route · routeFromHAR · routeWebSocket · unroute · unrouteAll · requests · waitForRequest · waitForResponse** (GAP-5) · **consoleMessages · pageErrors · clearConsoleMessages · clearPageErrors** (GAP-6) · **waitForEvent** *đầy đủ* — dialog/popup/download (GAP-4/3/8) · **addInitScript · addScriptTag · addStyleTag · exposeFunction · exposeBinding · evaluate · evaluateHandle · requestGC · waitForFunction · workers** (policy-untrusted) · **clock** (GAP-11) · **coverage** (GAP-12) · **localStorage · sessionStorage** (GAP-7) · **addLocatorHandler · removeLocatorHandler · cancelPickLocator · hideHighlight** (niche — tự động overlay-dismiss đã có dạng dismiss_overlays)

## 3. BROWSERCONTEXT (36)

### ✅
- pages · newPage(session mới) · browser(session_diagnostics) · close(release) · isClosed · waitForEvent*.innerText-family qua snapshot · setDefaultTimeout · setDefaultNavigationTimeout
### ⚠️
- newCDPSession — tồn tại nội bộ (toàn pipeline là CDP) nhưng **không expose cho agent** (chủ đích) · setHTTPCredentials(session_options credentials) · cookies/addCookies/clearCookies — quản trị phía profile, agent không API (GAP-7) · setGeolocation/locale/timezone/colorScheme — một phần (GAP-9) · storageState — persist nhưng không xuất JSON (GAP-7)
### ❌
- grantPermissions/clearPermissions (GAP-7) · route/routeFromHAR/routeWebSocket/unroute* (GAP-5) · exposeBinding/exposeFunction/addInitScript (policy) · setOffline (GAP-11) · serviceWorkers/backgroundPages (GAP-5) · debugger · clock · request · tracing · credentials (GAP-5/7/11/12)

## 4. ELEMENTHANDLE (33)
Kế thừa Locator-family qua @eN-ref: ✅ click/fill/hover/press/check/uncheck/contentFrame/ownerFrame/boundingBox/getAttribute/innerText/innerHTML/textContent/inputValue/is* · ⚠️ waitForElementState (wait_for gần đúng) · ❌ dragTo? same GAP-1 · setInputFiles (GAP-2) · screenshot element (GAP-10) · dispatchEvent/evaluate* (policy).

## 5. KEYBOARD / MOUSE / TOUCHSCREEN
- **Keyboard:** ✅ down/up/insertText(≈act type) · press ✅ (whitelist) · ⚠️ tổ hợp phím tùy ý bị chặn an toàn
- **Mouse:** ✅ click(a Leadership act click 5-điểm) · ✅ move/​hover · ⚠️ dblclick (GAP-9) · ❌ down/drag/wheel riêng lẻ (GAP-1; wheel có qua scroll)
- **Touchscreen:** ❌ tap (GAP-9)

## 6. DIALOG (5) · DOWNLOAD (6) · FRAME (~60)
- **Dialog:** ❌ toàn bộ accept/dismiss/beforeunload — không có handler khai báo trước (GAP-4) — hiện chỉ dismiss_overlays mù
- **Download:** ❌ toàn bộ — path/saveAs/suggestedFilename/cancel/failure/delete (GAP-3)
- **Frame:** ✅ kế thừa Page-family qua cross-frame snapshot/ref · ❌ đơn vị riêng (agent không thao tác raw frame — được proxy đủ qua tree)

## 7. LỚP VỆ TINH
- **ConsoleMessage ✅ đọc qua GAP-6(tương lai)** · **Request/Response/Route ❌ GAP-5** · **Cookie một phần GAP-7** · **CDPSession ❌ policy** · **APIRequestContext ❌ (ngoài phạm vi browser)** · **Tracing/Video/coverage ❌ GAP-12** · **Worker ❌ GAP-5** · **Clock ❌ GAP-11** · **Selectors/Playwright(browser factory) — không áp dụng (cloud session thay thế) · Touchscreen GAP-9** · **BrowserType.launch/connect — đấy chính là Steel cloud session**

## 8. TỔNG KẾT PARITY (method-level, đếm trên bộ đã trích)

| Lớp | Methods | ✅ | ⚠️ | ❌ |
|---|---|---|---|---|
| Locator | 71 | 54 | 6 | 11 |
| Page | 106 | 63 | 9 | 34 |
| BrowserContext | 36 | 10 | 7 | 19 |
| ElementHandle | 33 | 24 | 2 | 7 |
| Keyboard/Mouse/Touch | ~18 | 8 | 4 | 6 |
| Dialog/Download | 11 | 0 | 1 | 10 |
| Frame | ~40 dùng lại | ~30 | 4 | 6 |
| **TỔNG** | **~315** | **189 (60%)** | **33 (10%)** | **93 (30%)** |

**Đọc số:** 60% ✅ hôm nay + 10% ⚠️ = **70%面 tương tác người-thật đã parity**. Trong 93 ❌: **~70% gom vào đúng 15 GAP đã liệt kê** (network/file/drag/emulation/events/storage) — tức khả thi gom về **~90% ✅** sau khi vá P1–P5 + GAP 1–15; phần còn lại là **policy-untrusted** (evaluate thô, exposeFunction, CDP trực tiếp) và **ngoài phạm vi browser** (APIRequestContext) — chủ đích bảo mật của Steel, không phải thiếu năng lực.
