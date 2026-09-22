# 100 CASE THẬT — Duyệt / Sử dụng / Tự động hóa website bằng Steel MCP

> Sinh ra từ: (a) nghiên cứu nguồn mở steel-mcp-server@3ceb5c3 (đọc trực tiếp mã nguồn),
> (b) websearch + webfetch tài liệu Playwright & các site practice thật (demoqa.com,
> the-internet.herokuapp.com, automationexercise.com, vnexpress.net, vLXP/Moodle gerçek),
> (c) thực chiến đo đạc trên Steel cloud browser trong phiên làm quiz vLXP 30/30.
> Mỗi case: website thật, độ khó ★–★★★★★, tool Steel phủ, và cờ BUG-# / GAP-#when áp lực
> vướng điểm yếu đã biết (xem BUGS-AND-GAPS.md).

**Bộ tool bị khóa phủ (mọi tool phải ≥2 case):** navigate, snapshot, find, act(click, type,
fill_form, select, check, hover, scroll, press, go_back, dismiss_overlays), batch, wait_for,
scrape, screenshot, pdf, session_create, session_options, session_diagnostics,
session_release, handoff, live_view, replay.

---

## A. ĐỌC & THU THẬP NỘI DUNG (12)

| # | Case (website thật) | Khó | Tools Steel | Cờ |
|---|---|---|---|---|
| 1 | VnExpress: vào chuyên mục, đọc bài mới nhất (đã probe sống) | ★ | navigate + snapshot + scrape | — |
| 2 | Dân trí: bấm "Xem thêm" 4 lần, scrape 3 trang | ★★ | navigate + act(click) + scrape | — |
| 3 | Tuổi Trẻ: tra từ khóa, lấy 10 kết quả đầu | ★★ | navigate + fill_form + press(Enter) + scrape | — |
| 4 | VnExpress mục dài: cuộn lấy 20 bài tiếp | ★★ | act(scroll) + scrape + batch | BUG-4 |
| 5 | Moodle/vLXP form trắc nghiệm lazy-render: đếm 27 checkbox ẩn dưới fold | ★★★ | snapshot + act(check) | BUG-1/2/6, GAP-15 |
| 6 | Wikipedia VI: đọc trang thảo luận, cần login mới sửa | ★★★ | navigate + scrape + session_create | — |
| 7 | Reddit: mở thread, bấm "load more comments" ×3 | ★★★ | navigate + act(click) + wait_for | BUG-1 |
| 8 | Báo có paywall overlay: dismiss rồi đọc toàn bài | ★★★ | navigate + act(dismiss_overlays) + scrape | — |
| 9 | Google News: theo dòng chủ đề, truy nguồn gốc từng bài | ★★ | navigate + find + act(click) | — |
| 10 | YouTube: mô tả + expand bình luận (lazy) + chụp bằng chứng | ★★★ | navigate + act(click) + scrape + screenshot | BUG-6 |
| 11 | Unsplash grid ảo hóa: cuộn sâu 10 màn + screenshot full | ★★★ | act(scroll) + screenshot + wait_for | BUG-4/6 |
| 12 | dichvucong.gov.vn: tra cứu thủ tục hành chính theo tỉnh | ★★ | navigate + select + find + scrape | — |

## B. FORM & NHẬP LIỆU (15)

| # | Case | Khó | Tools | Cờ |
|---|---|---|---|---|
| 13 | Google Forms: khảo sát 10 câu trắc nghiệm + submit | ★★ | act(fill_form, check, click) + wait_for | BUG-1/3 |
| 14 | Tiki: search, filter, thêm 3 SP vào giỏ | ★★★ | navigate + fill_form + act(click) + scrape + batch | — |
| 15 | Shopee: chọn biến thể SP (màu/size) rồi thêm giỏ | ★★★ | find + act(click, check) + scrape | — |
| 16 | Đăng ký Gmail mới: form đa bước (bỏ qua CAPTCHA qua handoff) | ★★★★ | act(type) + handoff + wait_for | GAP-13 |
| 17 | Login Zalo/Facebook web + OTP | ★★★★ | fill_form + handoff + session_diagnostics | GAP-13 |
| 18 | Internet banking VCB: login + OTP + tra số dư trang chủ | ★★★★ | act(type) + handoff + screenshot | GAP-13 |
| 19 | Vietjet đặt vé 1 chiều: form hành khách 4 trường ×3 | ★★★★ | fill_form + select + press + batch | — |
| 20 | Vietnam Airlines: chọn ghế trên sơ đồ SVG tương tác | ★★★★★ | find + act(hover, click) + wait_for | GAP-9 |
| 21 | Agoda: filter giá + sort + chọn phòng | ★★★★ | act(select, click) + scrape + screenshot | — |
| 22 | BHXH nộp hồ sơ trực tuyến có đính kèm file | ★★★★ | fill_form + act(click) | GAP-2 |
| 23 | GDT tra cứu mã số thuế doanh nghiệp | ★★ | navigate + fill_form + scrape | — |
| 24 | VietnamPost tra vận đơn + in vận đơn | ★ | navigate + scrape + pdf | — |
| 25 | Google Maps: tìm địa điểm, copy link chia sẻ (canvas) | ★★★★ | navigate + fill_form + act(click) + scrape | GAP-10 |
| 26 | Zalo OA: gửi tin nhắn mini app cho KH | ★★★★ | act(type) + press + wait_for | — |
| 27 | Grab web: đặt xe thí điểm (bản đồ WebGL + auth) | ★★★★★ | navigate + act + handoff + screenshot | GAP-9/13 |

## C. TƯƠNG TÁC UI SÂU (15)

| # | Case | Khó | Tools | Cờ |
|---|---|---|---|---|
| 28 | Trello: kéo thẻ giữa 2 cột kanban (drag thật) | ★★★★ | find + act | GAP-1 |
| 29 | Notion: tạo page, gõ markdown, tổ hợp phím | ★★★★ | act(type, press) + batch | — |
| 30 | Google Sheets: nhập A1:C3 + copy công thức xuống | ★★★★ | fill_form + press | BUG-3 |
| 31 | Google Docs: gõ đoạn + Ctrl+B/I (contenteditable sâu) | ★★★★ | act(type, press) + scrape | — |
| 32 | Figma web: zoom + chọn layer (canvas riêng) | ★★★★★ | act(hover, press) + screenshot | GAP-10 |
| 33 | Excalidraw: vẽ hộp + kéo tâm (drag giữa tọa độ) | ★★★★★ | act | GAP-1 |
| 34 | CodeSandbox: gõ code + Run + đọc console output | ★★★★ | act(type, click) + scrape | GAP-6 |
| 35 | Admin demo (datatables.net): sort + paging 5 trang | ★★ | act(click) + scrape + batch | — |
| 36 | Google Calendar: tạo sự kiện + kéo đổi giờ (drag) | ★★★★ | fill_form + act | GAP-1 |
| 37 | Gmail web: search operator, gắn label, snooze | ★★★★ | fill_form + act(click) + wait_for | — |
| 38 | X/Twitter: like + repost + follow 1 thread | ★★★ | act(hover, click) + batch | — |
| 39 | TikTok web: upload video (file chooser) | ★★★★ | navigate + act(click) | GAP-2 |
| 40 | Carousel swipe cảm ứng (trang Landing mobile) | ★★★ | session_options(device) + act | GAP-9 |
| 41 | Modal trong modal (popup 2 tầng): đóng đúng tầng | ★★★ | act(dismiss_overlays, click) + snapshot | — |
| 42 | Upload ảnh đại diện LinkedIn (file chooser + crop) | ★★★★ | act(click) + find | GAP-2 |

## D. AUTH & SESSION (10)

| # | Case | Khó | Tools | Cờ |
|---|---|---|---|---|
| 43 | vLXP SSO Viettel: login 1 lần, cookie sống qua nhiều session (đã thực chiến) | ★★★★ | handoff + session_create + live_view | GAP-7 |
| 44 | HTTP Basic Auth (httpbin.org/basic-auth) | ★★★ | session_options(credentials) + navigate | — |
| 45 | OAuth Google vào app demo, đọc consent screen | ★★★★ | navigate + act + wait_for | GAP-13 |
| 46 | So sánh giá 2 tab song song (Google Shopping) | ★★★ | navigate + scrape + batch | GAP-8 |
| 47 | Logout rồi verify redirect về trang chủ | ★★ | act(click) + wait_for(url) | — |
| 48 | Flow "quên mật khẩu" + nhận email (mailinator) | ★★★★ | fill_form + scrape + wait_for | — |
| 49 | Cookie banner EU (cookiebot demo): dismiss + verify | ★★ | navigate + act(dismiss_overlays) | — |
| 50 | Cloudflare Turnstile demo site | ★★★★★ | navigate + wait_for | GAP-13 |
| 51 | reCAPTCHA v2 demo: thử pass không solver | ★★★★★ | act(click) + handoff | GAP-13 |
| 52 | Chống 429:Prices site giới hạn — backoff + retry 3 lần | ★★★ | batch + wait_for + session_diagnostics | — |

## E. BẰNG CHỨNG: ẢNH / PDF / SO SÁNH (8)

| # | Case | Khó | Tools | Cờ |
|---|---|---|---|---|
| 53 | Chụp chính sách lãi suất VCB fullPage làm lưu chứng | ★★ | screenshot + pdf | — |
| 54 | Xuất PDF hóa đơn điện tử + đối chiếu text trong PDF | ★★★ | pdf + scrape | BUG-6 |
| 55 | TradingView: chụp chart 2 thời điểm, so pixel-diff | ★★★★ | screenshot + wait_for | GAP-10 |
| 56 | Trang lazy dài: cuộn qua hết rồi mới chụp full (không sót trắng) | ★★★ | act(scroll) + screenshot | BUG-6 |
| 57 | Chụp DPR 2 so với DPR 1 (độ nét ảnh) | ★★★ | session_options + screenshot | GAP-9 |
| 58 | Quay lại hành vi chạy fail cho QA xem | ★★★ | replay + session_diagnostics | GAP-11 |
| 59 | Chụp dark mode vs light mode | ★★★ | session_options + screenshot | GAP-9 |
| 60 | Diff DOM snapshot trước/sau deploy (A/B) | ★★★ | snapshot ×2 + so text | — |

## F. HUMAN-IN-THE-LOOP (7)

| # | Case | Khó | Tools | Cờ |
|---|---|---|---|---|
| 61 | Handoff cho người gõ CAPTCHA rồi nhận lại quyền | ★★★★ | handoff + wait_for | GAP-13 |
| 62 | Live-view: người theo dõi bot chạy demo sale | ★★★ | live_view + act + screenshot | — |
| 63 | Replay phiên fail cho团队 QA chẩn đoán | ★★★ | replay + session_diagnostics | — |
| 64 | Dùng profile có sẵn login (khỏi SSO lại) | ★★★ | session_options(profileId) | — |
| 65 | Session treo: dùng diagnostics để phân tích sống/chết | ★★★ | session_diagnostics | BUG-7/9 |
| 66 | Kịch bản dừng an toàn khi biến cố, chờ người phán | ★★★★ | batch + wait_for + handoff | — |
| 67 | Bot setup giỏ hàng — người bấm thanh toán (mốc handoff) | ★★★★★ | act + handoff + live_view | — |

## G. BATCH & KỊCH BẢN DÀI (10)

| # | Case | Khó | Tools | Cờ |
|---|---|---|---|---|
| 68 | Batch 15 bước Tiki end-to-end tới giỏ (đã thực chiến dạng 15-step) | ★★★ | batch + find | BUG-5 |
| 69 | So giá 1 SP trên 5 site trong 1 kịch bản | ★★★★ | batch + scrape | — |
| 70 | Đo dead-link 50 URL chính sách | ★★ | batch(navigate) | BUG-7 |
| 71 | Watchdog giá: lặp check 10 chu kỳ 30s | ★★★ | batch + wait_for | GAP-11 |
| 72 | Điền form 30 trường / 58 input (đã thực chiến 58/58) | ★★★ | fill_form + act(check) | BUG-3 |
| 73 | Áp lực: 200 action trong 5 phút — quan sát rate/budget | ★★★ | batch + session_diagnostics | — |
| 74 | Verify từng bước batch trước khi qua bước sau (settle) | ★★★ | batch + snapshot | — |
| 75 | 2 session chạy song song (so 2 giỏ hàng) | ★★★★ | session_create ×2 + batch | GAP-8 |
| 76 | Chạy batch trên trang lazy-render chuẩn bị DOM trước | ★★★★ | scroll + wait_for + batch | BUG-6 |
| 77 | Recovery: DOM đổi giữa batch → reacquire rồi tiếp | ★★★★ | batch + find | BUG-5 |

## H. ANTI-BOT & KHÓ NHẤT (8)

| # | Case | Khó | Tools | Cờ |
|---|---|---|---|---|
| 78 | Amazon: search + add-to-list (bot fence) | ★★★★★ | navigate + act + wait_for | GAP-13 |
| 79 | LinkedIn: login + scrape profile công khai | ★★★★★ | handoff + scrape | GAP-13 |
| 80 | Ticketmaster virtual waiting room: đợi tới lượt rồi act | ★★★★★ | wait_for + act(click) | — |
| 81 | Raffle giày: submit form trong ~1s khi mở | ★★★★★ | fill_form + press + batch | — |
| 82 | Steam Market đặt buy order (auth + economics) | ★★★★★ | fill_form + act + handoff | — |
| 83 | săn vé giá sai: fill nhanh trong TTL 3 phút | ★★★★★ | batch + fill_form | BUG-7 |
| 84 | Nộp phi dịch vụ công lúc cao điểm (queue + timeout) | ★★★★★ | wait_for + act + handoff | BUG-7 |
| 85 | Trang ngâm hàng SPA gọi API ẩn: lấy dữ liệu từ response | ★★★★ | navigate + scrape | GAP-5 |

## I. "USER THẬT CẦM CHUỘT" END-TO-END (10)

| # | Case | Khó | Tools | Cờ |
|---|---|---|---|---|
| 86 | Mua hàng Tiki tới bước COD rồi dừng | ★★★★★ | batch + act + handoff | BUG-5/7 |
| 87 | Agoda đặt phòng tới "Xác nhận" rồi hủy | ★★★★★ | batch + act + wait_for | — |
| 88 | Upwork: employer đăng job đầy đủ | ★★★★★ | fill_form + act + handoff | — |
| 89 | vLXP quiz 30 câu → submit → đọc điểm (THỰC CHIẾN 30/30) | ★★★★★ | session + act(check) + press + scrape | BUG-1/3 |
| 90 | WordPress: đăng bài kèm ảnh đại diện | ★★★★ | fill_form + act | GAP-2 |
| 91 | Substack: subscribe + xác nhận email | ★★★ | fill_form + scrape + wait_for | — |
| 92 | Google Sheets nhập bảng điểm 30 dòng × 5 cột | ★★★★ | fill_form + press + batch | — |
| 93 | Zalo web: mở hội thoại, ghim tin nhắn | ★★★★ | find + act(click) + press | BUG-5 |
| 94 | Thư viện số: tìm + mượn sách online | ★★★ | navigate + fill_form + scrape | — |
| 95 | Đăng ký thi IELTS British Council (form dài + thanh toán dừng) | ★★★★★ | fill_form + select + handoff | — |

## J. ĐO LƯỜNG & KIỂM ĐỊNH (5)

| # | Case | Khó | Tools | Cờ |
|---|---|---|---|---|
| 96 | Đo thời gian load 10 trang (TTFB qua navigate logs) | ★★★ | navigate + session_diagnostics | GAP-5 |
| 97 | Audit a11y nhanh: đọc tree snapshot, đếm control vô tên | ★★★ | snapshot | BUG-2 |
| 98 | SEO batch: title/meta/desc 100 URL | ★★ | scrape + batch | — |
| 99 | Kiểm ảnh gãy hàng loạt (trang báo dài) | ★★ | scrape + screenshot | — |
| 100 | soi chính tả tiếng Việt 20 trang chính sách | ★★★ | scrape + batch | — |

---

## MA TRẬN PHỦ TOOL (tool → case tiêu biểu)

- **navigate**: 1, 12, 23, 44, 50, 70, 96 · **snapshot**: 1, 5, 41, 60, 74, 97 · **find**: 9, 14, 28, 38, 42, 93
- **act.click**: 2, 7, 14, 35, 80 · **act.type**: 16, 26, 29, 31, 34 · **fill_form**: 3, 13, 19, 23, 72, 92
- **act.select**: 12, 21, 95 · **act.check**: 5, 13, 15, 89 · **act.hover**: 20, 32, 38
- **act.scroll**: 4, 11, 56, 76 · **act.press**: 3, 19, 29, 31, 81 · **go_back**: 47, 87 (+mọi luồng multi-page: 2, 9, 14)
- **dismiss_overlays**: 8, 41, 49 · **batch**: 4, 14, 35, 52, 68, 73, 98 · **wait_for**: 7, 10, 13, 45, 47, 50, 80, 91
- **scrape**: 1, 12, 23, 69, 85, 98 · **screenshot**: 10, 11, 53, 55, 56, 57, 59, 99 · **pdf**: 24, 53, 54
- **session_create/options/release**: 6, 44, 57, 59, 64, 75 · **session_diagnostics**: 17, 52, 58, 63, 65, 73, 96
- **handoff**: 16, 17, 61, 66, 67, 79 · **live_view**: 43, 62, 67 · **replay**: 58, 63

## PHÂN BỐ ĐỘ KHÓ
★×16 · ★★×18 · ★★★×36 · ★★★★×21 · ★★★★★×9 — tăng dần từ "đọc trang tĩnh" tới
"săn vé giá sai trong TTL" và "form 58 input lazy-render" (đã có số đo thực chiến).
