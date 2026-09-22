# BUG & GAP — Danh sách phát hiện từ 100 case + đấu mã nguồn

> Nguồn chứng cứ: (a) mã nguồn steel-mcp-server@3ceb5c3 (dẫn file:dòng), (b) số đo thực chiến
> session vLXP 30/30, (c) probe sống trên the-internet.herokuapp.com / vnexpress.net.

## PHẦN 1 — BUG (hành xử sai thiết kế, đã có bằng chứng)

### BUG-1 · click/check không có đường thoát DOM-dispatch (nghiêm trọng nhất)
- **Chứng cứ mã:** page.ts:595-599 chỉ Input.dispatchMouseEvent; select lại có
  Runtime.callFunctionOn (page.ts:757-767) → cơ chế tồn tại nhưng không nối vào click/check.
- **Chứng cứ đo:** 27 checkbox lazy-render → act(check) **0/27** (toàn clickHitTestUnstableError,
  errors.ts:554); cùng DOM đó qua DOM-dispatch → **27/27** ngay lần đầu.
- **Case chạm:** 5, 7, 10, 13, 89. **Vá:** P1 trong REVIEW-VI.md.

### BUG-2 · accessible name bỏ qua aria-labelledby
- **Chứng cứ mã:** snapshot.ts:510 chỉ đọc aria-label; label text ở node khác thì input "mồ côi tên".
- **Chứng cứ đo:** quiz vLXP toàn bộ 91 radio + 56 checkbox input không tên trong tree; mổ DOM
  thấy nhãn nằm ở div id mà input trỏ qua aria-labelledby.
- **Case chạm:** 5, 97. **Vá:** P3.

### BUG-3 · check không đối chiếu .checked sau click
- **Chứng cứ mã:** act(check) chỉ settle theo navigated/domMutated/focusChanged.
- **Chứng cứ đo:** 36 click "thành công" → đếm lại **0 ticked** (framework re-render nuốt state
  giữa loạt click) — tool vẫn không kêu.
- **Case chạm:** 13, 30, 72, 89. **Vá:** P2.

### BUG-4 · scroll cuộn cửa sổ chính, không cuộn scroll-container nội bộ
- **Chứng cứ đo:** quiz 30 câu có container cuộn riêng; act(scroll) + phím PageDown/End chỉ cuộn
  window — câu 20-30 không bao giờ vào viewport; canvas lazy không paint.
- **Case chạm:** 4, 11, 30, 56, 76. **Vá:** scroll cần nhận diện ancestor scrollable gần nhất.

### BUG-5 · ref @eN chết giữa batch không tự reacquire
- **Chứng cứ mã:** refs keyed (loaderId, backendNodeId) (snapshot.ts:613) — đúng đắn, nhưng
  mutation giữa batch làm ref cũ trỏ void → cả step còn lại đổ.
- **Chứng cứ đo:** "DOM.querySelector failed" xuất hiện sau mọi action đổi DOM; phải re-snapshot.
- **Case chạm:** 68, 77, 86, 93. **Vá:** P5 (fingerprint tag#id+text để tự re-resolve).

### BUG-6 · getNodeForLocation mù trên content chưa paint (lazy/virtual)
- **Chứng cứ mã:** hitTestPoint → DOM.getNodeForLocation (page.ts:465-471) trả "No node found"
  với node có box nhưng chưa paint.
- **Chứng cứ đo:** đúng cơ chế khiến 0/27 nêu trên; PDF/screenshot full trên trang lazy dễ sót.
- **Case chạm:** 10, 54, 55, 56, 76. **Vá:** P4 (ghost row + deep sweep) + P1.

### BUG-7 · session 15' hard timeout giết kịch bản giữa chừng
- **Chứng cứ đo:** session chết đúng lúc bấm nộp bài (autosave cứu); probe hôm nay (KRydfOAL)
  chết <1 phút sau navigate. Không có cảnh báo trước ngưỡng.
- **Case chạm:** 83, 84, 86, 89. **Vá:** expose AgeMs+TTLLeft trong mọi response tool;
  auto-checkpoint trọng state trước khi hết giờ.

### BUG-8 · cross-tab navigation làm treo Page domain
- **Chứng cứ đo:** "Page.navigate did not answer" / "Page.getFrameTree did not answer" → phải
  release + recreate session mới ăn.
- **Vá:** watchdog Page domain sau navigate; recover tự động.

### BUG-9 · lỗi session chết không kèm ngữ cảnh tuổi/limit
- **Chứng cứ đo:** "No live browser session..." (probe B hôm nay) — không nói session几 tuổi,
  chết vì timeout hay release tay; agent phải đoán.
- **Vá:** kèm expiresAt/createdAt/lastError vào lỗi.

## PHẦN 2 — GAP: case chưa đáp ứng → tính năng bổ sung (15)

| GAP | Case | Đề xuất |
|---|---|---|
| 1 · drag & drop | 28, 33, 36 | act.drag(source→target): pointer sequence mousedown→move→mouseup như Playwright dragAndDrop |
| 2 · file upload | 22, 39, 42, 90 | intercept file chooser + setInputFiles (đường blob từ workspace) |
| 3 · download capture | 24, 86 | chờ download, lưu xuống workspace, trả đường dẫn |
| 4 · dialog (alert/confirm/prompt/beforeunload) | 80, 87 | handler khai báo trước kịch bản (accept/dismiss + text) |
| 5 · network inspect | 85, 96 | stream request/response tối thiểu (method/url/status/size) + filter |
| 6 · console/log streaming | 34 | subscribe console/pageerror, buffer vòng, truy vấn theo session |
| 7 · storage-state export/import | 43 | xuất/nạp storageState JSON — kéo dài phiên đăng nhập không cần handoff |
| 8 · đa tab/đa cửa sổ orchestration | 46, 75 | list tabs, activate, mỗi tab một ref-space |
| 9 · emulation per-session (device/touch/_timezone/locale/geolocation/colorScheme/DPR) | 20, 25, 40, 57, 59 | mở lại đủ tham số context như Playwright newContext |
| 10 · element-screenshot clip + ảnh diff | 55 | chụp theo ref, so 2 ảnh (pixel + layout), trả khác biệt |
| 11 · clock/throttle | 71 | clock.fastForward + network throttle profiles (3G/4G/offline) |
| 12 · tracing/video export | 58 | trace zip chuẩn Playwright + video mp4 của phiên |
| 13 · thường trực anti-bot/CAPTCHA escalation | 16, 50, 61, 78, 79 | dò tín hiệu captcha → tự động mở handoff людям + wait_for |
| 14 · PDF lazy chuẩn bị | 54 | trước render PDF: sweep scroll toàn trang để paint đủ |
| 15 · deep sweep ghost snapshot | 5, 76 | chế độ deep: liệt kê node chưa paint dạng "ghost ×N" để agent còn biết dưới fold có gì |

## PHÁN QUYẾT

- Sau khi vá **BUG 1-6** + bổ **GAP 1-15**: mọi case ★–★★★★ (95/100) nằm trong tầm;
  5 case ★★★★★ còn lại (50, 51, 78, 79, 81) chạm chủ đích chống-bot của web — giới hạn
  chính sách, không phải giới hạn kỹ thuật của server.
- Đó chính là định nghĩa "lý tưởng như user thật cầm chuột": **mọi thao tác DOM-reachable
  làm được; mọi thao tác bị web từ chối người máy thì chuyển người (handoff) có kiểm soát.**
