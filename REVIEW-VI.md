# REVIEW — Kiến trúc steel-mcp-server & Khả năng nâng "discover" lên tin cậy gần tuyệt đối

> Ngữ cảnh đánh giá: một tác vụ thực chiến (quiz vLXP/Moodle 30 câu, 27 checkbox lazy-render)
> đã được đo đạc bằng chính repo này (MCP route) và bằng Playwright gắn CDP song song.
> Mọi kết luận dưới đây đều có chứng chỉ `file:line` từ mã nguồn hoặc số liệu đo thực.

## 0. Kết luận trước (BLUF)

| Câu hỏi | Trả lời |
|---|---|
| Có thể cải tiến để discover/react **tin cậy hơn mạnh** không? | **Có** — 5 patch cụ thể ở §4, mỗi patch khép ngay 1 lớp thất bại đã đo được |
| Có đạt "tin cậy **tuyệt đối**" không? | **Không tuyệt đối — và không nên hứa**. Trần trên là: mọi tương tác **DOM-reachable** đều làm được (~những gì Playwright/CDP làm được). Sàn dưới còn lại (đã thu hẹp rất sát): handler kiểm tra `isTrusted`, CAPTCHA/anti-bot chủ đích, closed shadow root, rào same-origin cross-frame |
| Lớp thất bại nào rẻ nhất để phá? | **Click-path không có fallback DOM-dispatch** — bằng chứng 0/27 (route hiện tại) vs 27/27 (DOM dispatch) trên cùng một DOM |

## 1. Kiến trúc thực tế (đọc từ mã nguồn)

```
MCP client (stdio/HTTP)
  └─ src/core/server.ts + tools/ (session | browse | batch | handoff | replay)
       └─ core/page.ts :: BrowserPage — lái MỘT page qua CDP (steel/cdp.ts)
            act(click|check|hover|type|fill_form|select|scroll|press)
            snapshot() / find()  ← core/snapshot.ts (đọc DOMSnapshot + Accessibility domain)
```

### 1.1 Click pipeline hiện tại (page.ts)
```
resolveTarget(@eN → backendNodeId)
  → candidatePoints()           # DOM.scrollIntoViewIfNeeded + DOM.getBoxModel
                                # + lấy 5 điểm mẫu trên content-quad (page.ts:430-443)
  → hitTestPoint() × điểm       # DOM.getNodeForLocation; gọi tên blocker;
                                # khoan dung nếu blocker là con cháu của target (page.ts:458-512)
  → Input.dispatchMouseEvent    # press + release (page.ts:595-599)
  → settleNow()                 # đối chiếu baseline: navigated/domMutated/focusChanged
```
Điều tốt đã có (không cần đề xuất lại): `scrollIntoViewIfNeeded` **đã gọi trước click**;
hit-test gọi **tên phần tử che** (cookie banner) thay vì báo mù; đếm failure-episode để
chặn agent lặp бесконечно; refs `(loaderId, backendNodeId)` sống qua đổi label.

### 1.2 Snapshot pipeline (snapshot.ts)
- DOM.getSnapshot layout/styles/attributes + Accessibility; chỉ node **có box + nhận pointer
  events** mới được phát ref `(loaderId, backendNodeId)` (snapshot.ts:489-504).
- Ngân sách node (DEFAULT_MAX_NODES = 1500), redact nhạy cảm, defang link.
- Accessible name hiện chỉ lấy `aria-label`/label-Qualified (snapshot.ts:510) —

### 1.3 Route đã có sẵn DOM-dispatch — nhưng chỉ cho `select`
`act(select)` chạy `Runtime.callFunctionOn` gán `.value` + bắn `input`/`change`
(page.ts:757-767). Tức là **cơ chế fallback DOM đã tồn tại trong codebase** — chỉ chưa
được nối vào nhánh `click`/`check`.

## 2. Chứng nghiệm thực chiến (đo trên cùng một trang)

| Thử | Route | Kết quả |
|---|---|---|
| Tick 27 checkbox lazy-render | MCP `act(check)` (coordinate path) | **0/27** —すべて dính `clickHitTestUnstableError` ("Chrome found no page node", errors.ts:554) |
| Cùng 27 checkbox đó | Playwright evaluate `el.click()` qua CDP cùng browser | **27/27** ngay lần đầu |
| Radio (nút có paint) | fill_form + phím mũi tên | 55/55 — route keyboard không cần hit-test |
| Hiểu nhãn option | snapshot @eN | Thất bại — text nằm trong `div#id` mà input tham chiếu qua `aria-labelledby`; tree hiện text "mồ côi" + input 无 tên |
| Thấy câu chưa render | snapshot | Không thấy — thiết kế chỉ phát ref cho node có box (đúng Chủ ý an toàn, nhưng agent không biết còn bao nhiêu câu dưới fold) |

## 3. Chẩn đoán: vì sao route coordinate chết trên lớp này

Lazy-render dạng `content-visibility`/IntersectionObserver: **box model tồn tại** (nên qua
được `candidatePoints`) nhưng node **chưa được paint** → `DOM.getNodeForLocation` trả
"No node found" ở mọi điểm mẫu → 2 lần refresh layout (page.ts:562-589) → lỗi cứng.
Không phải bug — đây là chủ ý "trusted-only" của pipeline: mọi click đều là input thật.
Chỉ là chủ ý đó **không có tầng thoát hiểm**.

## 4. Năm patch đề xuất (xếp theo ROI)

### P1 — DOM-dispatch fallback cho click/check (ROI cao nhất)
Sau khi coordinate path kiệt sức (đã 2 layout-attempt + episode), thay vì fail cứng:
```ts
// act('click'|'check'): sau reachablePoint() ném clickHitTestUnstableError
// → chuyển sang dispatch el.click() và VẪN đối chiếu settleNow()
const resolved = await this.session.send('DOM.resolveNode', { backendNodeId: handle.backendNodeId });
await this.session.send('Runtime.callFunctionOn', {
  objectId: resolved.object?.objectId,
  functionDeclaration: 'function() { this.click(); }',
});
// Outcome.summary đánh dấu: "Clicked X (synthetic-DOM fallback)"
```
Điều kiện: giữ mặc định trusted-only; fallback (a) tự động sau khi episode chassis hết,
hoặc (b) opt-in flag `force_dom`. **Bằng chứng giá trị: 0/27 → 27/27 trong đo thực.**

### P2 — Verify trạng thái `.checked` sau `check`
Hiện `check` chỉ đối chiếu domMutated/focusChanged — checkbox bị framework re-render
reset ngay sau click sẽ thành "im lặng" → `clickNoObservedChangeError`. Đọc sự thật:
```ts
const state = await this.session.send('Runtime.callFunctionOn', {
  objectId, functionDeclaration: 'function() { return this.checked; }', returnByValue: true,
});
// Include vào outcome: { checked: state.result?.value === true }
```
Lớp thất bại nó phá: "click rơi nhưng state không đổi" (đã gặp: 36 click → 0 ticked do re-render giữa chừng).

### P3 — Nối `aria-labelledby` vào accessible name (snapshot.ts)
Facts đã giữ attributes (snapshot.ts:256-262): chỉ cần resolve id-list rồi ghép text:
```ts
const labelledBy = attributes['aria-labelledby'];
if (labelledBy) name = labelledBy.split(/\s+/).map(id => docFacts.byId(id)?.text).filter(Boolean).join(' ');
```
Lớp phá: form/widget tách nhãn khỏi input (chính cái khiến ta phải "mổ DOM" tay).

### P4 — Node "ghost" cho vùng chưa render (`deep: true`)
Với node KHÔNG có box nhưng ở trong DOM (`display` không none, đơn giản là chưa layout do
virtualization): không phát ref (giữ bất biến an toàn) nhưng **phát 1 dòng ghost**:
`unrendered-control × 27 (dưới fold, cuộn/đợi rồi snapshot lại)` — để agent hết "mù cấu trúc"
và biết phải chuẩn bị gì. Chế độ `deep` có thể sweep `scrollIntoViewIfNeeded` các container
chưa render trước khi capture (đúng thủ tục Playwright fullPage vẫn dùng).

### P5 — Tự re-acquire ref trước khi báo stale
Refs key `(loaderId, backendNodeId)` (snapshot.ts:613) chết sau navigation/re-render mạnh.
Trước khi ném `stale_ref`: thử re-resolve bằng fingerprint `tag#id + chữ-của-text` đã ghi
khi phát ref; thành công → ref mới + cảnh báo `ref_reacquired`; thất bại mới báo agent
dùng `steel_find`.

## 5. Đánh giá "tin cậy tuyệt đối" — ranh giới thật

| Lớp | Hiện tại | Sau 5 patch | Vostro comment |
|---|---|---|---|
| Nút có paint, không bị che | ✅ | ✅ | đã tốt |
| Nút bị overlay che | ✅ báo tên blocker | ✅ | điểm sáng của repo |
| Lazy-render/unrendered | ❌ cứng | ✅ (P1+P4) | 0/27 → 27/27 đã chứng minh |
| Framework re-render nuốt state | ⚠️ báo sai nguyên nhân | ✅ (P2) | từ "im lặng" → "đọc lại sự thật" |
| Nhãn tách `aria-labelledby` | ❌ mù | ✅ (P3) | |
| Ref chết sau mutation | ⚠️ agent tự find | ✅ tự phục hồi (P5) | |
| Handler kiểm `isTrusted` | ❌ | ❌ (chủ đích) | chỉ người thật/live-view giải được |
| CAPTCHA / closed shadow root / same-origin frame | ❌ | ❌ | giới hạn chính sách — không phải bug |

**Phán quyết:** repos này có thể nâng từ "trusted-only, gặp DOM lạ là tường" lên
"**mọi DOM-reachable đều chạm được, mọi thất bại còn lại đều được gọi tên đúng bản chất**".
Đó là mặt trần thật của automation — ngang mức Playwright thuần nhưng giữ nguyên 2 thứ Playwright
không có: báo lỗi hướng dẫn agent tự sửa + ngân sách token cho model.

## 6. Những quyết định thiết kế KHÔNG nên đụng khi vá

1. Trusted-first + chỉ fallback khi cạn đường — an toàn mặc định (events thật > synthetic trừ khi buộc).
2. Hit-test gọi tên blocker — продолж giữ cho route trusted.
3. Refs sống qua đổi label (`loaderId+backendNodeId`) — P5 chỉ bổ sung, không thay.
4. Ngân sách node + redact — đúng theo RESEARCH.md §5; `deep` phải là opt-in.
