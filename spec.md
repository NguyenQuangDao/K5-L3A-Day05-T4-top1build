# Bài nộp LAB5 — T4 Thương mại điện tử · Nhóm top1build

**Lớp:** L3A · **Bàn / zone:** \_\_\_\_ · **Track:** T4 Thương mại điện tử

| Họ và Tên            | Mã Học Viên | Việc đã làm                                                                                                          |
| -------------------- | ----------- | -------------------------------------------------------------------------------------------------------------------- |
| Nguyễn Quang Đạo     | 2A202603614 | Phụ trách §1, §2 (Kịch bản rủi ro & Biện luận), §6 (Tiêu chí Pass/Fail)                                              |
| Nguyễn Vũ Thành Long | 2A202603635 | Phụ trách §3 (Khi AI sai: câu thoại, loại lỗi, HAX/PAIR, lỗi không cứu được bằng câu chữ) và ghi chú HAX/PAIR cho §5 |
|                      |             | Phụ trách §4 (Mức tự chủ), §5 (Prototype `prototype/index.html` và ảnh đánh số), thêm KB9 vào §2                     |

## §1. Lát cắt & phạm vi

- **Lát cắt (chép từ track):** Trong **Trò chuyện với Shopee**, khi **người mua** đang báo "hàng nhận không đúng mô tả", **AI tự hướng dẫn gửi yêu cầu trả hàng hoàn tiền, xin thêm bằng chứng, hay chuyển nhân viên**, để người mua biết đúng bước tiếp theo, và người bán không bị xử thua khi họ đúng.

**3 việc tính năng KHÔNG làm:**

1. **Không tự động duyệt hoàn tiền ngay lập tức:** Vì rủi ro gian lận (buyer fraud) và xâm hại quyền lợi người bán; các giao dịch bồi hoàn tài chính không thể tự đảo ngược khi người mua gửi hàng giả hoặc ảnh mượn.
2. **Không tự đưa ra phán quyết pháp lý hoặc tuyên bố bên nào đúng/sai trong chat:** Vì AI chỉ là công cụ hỗ trợ đối soát quy trình và làm việc theo xác suất, không thể bảo đảm 100% cập nhật đúng luật hay có đủ ngữ cảnh thực tế, phát ngôn khẳng định sai sẽ đẩy Shopee vào rủi ro tranh chấp pháp lý và khiếu kiện.
3. **Không xử lý thanh toán bồi hoàn ngoài sàn hoặc chia sẻ số tài khoản ngân hàng cá nhân:** Vì vi phạm chính sách giao dịch an toàn của sàn, tiếp tay cho hành vi lừa đảo qua kênh ngoài.

## §2. Kịch bản rủi ro (chặng 2)

### Bảng kịch bản rủi ro (8 kịch bản phủ 4 lớp ①②③④)

| STT | Tin nhắn người dùng                                                                                     |               Lớp                | AI có thể sai thế nào (Trỏ về tool/dữ liệu)                                                                                                                                                                                                                      | Sản phẩm tốt nên làm gì                                                                                                                                                                                   |     Nhãn rủi ro      |
| :-: | :------------------------------------------------------------------------------------------------------ | :------------------------------: | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :------------------: |
|  1  | _"Shop ơi áo này vải mỏng dính không giống ảnh quảng cáo, hoàn tiền gấp"_                               |       ① Ngữ cảnh / Ý định        | AI hiểu nhầm là hủy đơn thay vì khiếu nại trả hàng; hoặc vội vàng gửi link tạo yêu cầu Trả hàng khi dữ liệu hệ thống báo đơn chưa giao thành công (`order.status != DELIVERED`).                                                                                 | Tra cứu `order.status`: nếu đang giao -> hướng dẫn chờ nhận; nếu đã giao -> nhắc giữ nguyên tem mác, bao bì và hướng dẫn các bước mở khiếu nại đúng chuẩn.                                                |       Sửa được       |
|  2  | _"Hàng giao thiếu 1 chiếc, nhưng tôi lỡ bấm 'Đã nhận được hàng' rồi"_                                   |      ② Dữ liệu / Trạng thái      | AI chỉ đọc cờ trạng thái bề mặt `order.status == COMPLETED` -> vội vàng từ chối: _"Đơn đã hoàn tất, không thể khiếu nại"_, bỏ qua trường `delivered_timestamp` và chính sách Shopee Guarantee (cho phép thời gian khiếu nại tới 15 ngày tùy loại hàng sau nhận). | Kiểm tra `delivered_timestamp` và chính sách bảo vệ người mua: Nếu còn hạn -> hướng dẫn gửi yêu cầu trả hàng/hoàn tiền lý do "Thiếu hàng" và kích hoạt hỗ trợ mở lại case.                                | Không chấp nhận được |
|  3  | _"Gửi video bóc hàng dài 5 phút nhưng web báo lỗi dung lượng, tôi chỉ up được 2 ảnh chụp ngoài vỏ hộp"_ |     ③ Bằng chứng / Xác thực      | Tool `validate_evidence` kiểm tra thấy thiếu file video theo quy tắc cứng -> AI thẳng tay từ chối khiếu nại hoặc đóng luồng hỗ trợ vì "thiếu bằng chứng hợp lệ".                                                                                                 | Nhận diện lỗi kỹ thuật upload; chấp nhận tạm thời 2 ảnh rõ mã vận đơn và hướng dẫn nén video, hoặc cung cấp link upload bổ sung an toàn / chuyển nhân viên hỗ trợ tiếp nhận.                              |       Sửa được       |
|  4  | _"Shop lừa đảo, hàng fake 100%, tao sẽ kiện lên công an và đăng bóc phốt khắp mạng xã hội"_             | ④ Xúc cảm / Tranh chấp / Pháp lý | AI thiếu khả năng phân tích cảm xúc cực độ (`sentiment_score`), phản hồi máy móc dập khuôn: _"Vui lòng gửi ảnh chụp tem mác"_ khiến khách thêm phẫn nộ; hoặc AI tự ý hứa bồi thường để xoa dịu làm lộ sơ hở pháp lý.                                             | Nhận diện từ khóa pháp lý/đe dọa truyền thông -> Dừng quy trình tự động; dùng câu thoại xoa dịu tiêu chuẩn của Shopee và lập tức kích hoạt escalate chuyển chuyên viên CSKH ưu tiên xử lý.                | Không chấp nhận được |
|  5  | _"Sản phẩm là thịt bò đông lạnh/hoa quả tươi giao đến nơi bị thiu ủng bốc mùi hết rồi"_                 |      ② Dữ liệu / Loại hàng       | AI dùng quy trình mặc định (`default_return_flow`) hướng dẫn đóng gói gửi hàng trả về cho shop qua bưu tá, trong khi danh mục là hàng tươi sống (`category == PERISHABLE_FOOD`).                                                                                 | Nhận diện danh mục hàng không thể trả hàng vật lý -> Chuyển luồng sang "Yêu cầu hoàn tiền không trả hàng" (Refund Only), xin ảnh/video hiện trạng hỏng và gửi thông báo cho người bán phản hồi.           | Không chấp nhận được |
|  6  | _"Tôi muốn trả hàng vì mặc không vừa, nhưng chủ shop bảo không hỗ trợ đổi size"_                        |       ① Ngữ cảnh / Ý định        | AI nhầm lẫn giữa "Hàng không đúng mô tả" (lỗi người bán - miễn phí trả hàng) và "Đổi ý / Không vừa" (lỗi người mua - theo chính sách đổi trả) dẫn tới tranh chấp phí vận chuyển trả hàng.                                                                        | Đặt câu hỏi làm rõ: _"Sản phẩm nhận được có đúng kích cỡ/số đo theo bảng size của shop không hay do bạn mặc chật?"_ để phân loại đúng mã lý do trả hàng.                                                  |         Nhẹ          |
|  7  | _"Hộp quà Tết bị móp méo rách nát lúc shipper đưa, tôi chụp lại từ chối nhận rồi"_                      | ④ Tranh chấp / Đơn vị vận chuyển | AI ghi nhận là lỗi sản phẩm của người bán (`seller_fault`), tự động trừ điểm vận hành (sao chép phạt) của shop thay vì nhận diện lỗi do đơn vị vận chuyển (`logistics_damage`).                                                                                  | Đối soát dữ liệu vận chuyển và biên bản đồng kiểm; phân định lỗi vận chuyển để kích hoạt quy trình bồi thường từ đơn vị vận chuyển (3PL), bảo vệ quyền lợi người bán.                                     |       Sửa được       |
|  8  | Khách gửi ảnh chụp màn hình hóa đơn chuyển khoản ngân hàng hoặc ảnh chụp mờ tịt không thấy mã đơn       |     ③ Bằng chứng / Xác thực      | AI dựa vào số lượng file (`file_count >= 1`) mà không kiểm tra nội dung/chất lượng ảnh -> thông báo _"Đã đủ bằng chứng, vui lòng đợi duyệt"_, sau đó nhân viên duyệt từ chối khiến khách ức chế.                                                                 | Chạy kiểm tra chất lượng ảnh (`image_quality_check`): Nếu ảnh mờ hoặc sai đối tượng -> phản hồi ngay lập tức, nêu rõ tiêu chí còn thiếu và cung cấp ảnh mẫu hướng dẫn chụp rõ tem vận đơn + chi tiết lỗi. |         Nhẹ          |

### Quyết định: 2 kịch bản đắt nhất & 1 kịch bản suýt chọn

1. **Kịch bản đắt nhất 1: Kịch bản số 4 (Buyer tức giận tột độ, tố cáo lừa đảo, đe dọa kiện tụng & bóc phốt truyền thông)**
   - **Vì sao đắt nhất:** Chi phí sai sót (Cost of Error) ở kịch bản này không dừng lại ở giá trị đơn hàng mà đe dọa trực tiếp đến **uy tín thương hiệu (PR crisis), nguy cơ pháp lý và sự an toàn vận hành của sàn**. Nếu AI trả lời dập khuôn hoặc tự hứa hẹn bồi thường sai luật, câu từ của AI sẽ trở thành bằng chứng chống lại Shopee trước cơ quan chức năng hoặc bị phát tán làm khủng hoảng truyền thông.
2. **Kịch bản đắt nhất 2: Kịch bản số 5 (Hàng thực phẩm tươi sống / Hàng hư hỏng nhanh giao hoàn trả vật lý)**
   - **Vì sao đắt nhất:** Gây **thiệt hại tài chính và hậu cần không thể đảo ngược (irreversible)**. Nếu AI chỉ dẫn đóng gói gửi trả qua bưu tá cho thực phẩm ôi thiu: hàng sẽ thối rữa trong kho trung chuyển gây ô nhiễm môi trường/kho bãi, phát sinh chi phí vận chuyển vô nghĩa, người bán nhận hàng hỏng không thể bán lại, đẩy người mua và người bán vào tranh chấp gay gắt.
3. **Kịch bản suýt chọn: Kịch bản số 2 (Đã bấm 'Đã nhận được hàng' nhưng giao thiếu hàng)**
   - **Vì sao suýt chọn:** Tần suất xảy ra rất cao trong thực tế, dễ gây mất niềm tin của khách vào cơ chế bảo vệ của Shopee ("Shopee Guarantee").
   - **Vì sao không chọn:** Dù gây ức chế, nhưng sai sót này **hoàn toàn có thể sửa chữa/đảo ngược được (reversible)** thông qua quy trình hỗ trợ bổ sung (mở ticket khiếu nại nội bộ hoặc nhân viên can thiệp mở lại cổng hoàn tiền), không để lại hậu quả phá hủy vật chất hay rủi ro pháp lý/truyền thông nghiêm trọng như 2 kịch bản trên.

## §3. Khi AI sai (chặng 3)

Hai kịch bản đắt nhất chọn ở §2: **KB4** (người mua đe dọa kiện / bóc phốt) và **KB5** (hàng tươi sống bị thiu, AI chỉ dẫn gửi trả vật lý).

### 3.1. KB4 — Người mua đe dọa pháp lý / truyền thông

> Tin nhắn: _"Shop lừa đảo, hàng fake 100%, tao sẽ kiện lên công an và đăng bóc phốt khắp mạng xã hội"_

**AI sai thế nào:** trả lời dập khuôn ("Vui lòng gửi ảnh tem mác") làm khách tức hơn, hoặc tự hứa bồi thường / nhận "shop sai" để xoa dịu → câu của AI thành bằng chứng chống lại Shopee và người bán.

**Cách xử lý đã chọn (dừng và chuyển người, trong đúng 1 lượt).** Trong 5 tool của track, ca này chỉ dùng `chuyen_truong_nhom` và `yeu_cau_bang_chung`; AI **không** gọi `soan_hoan_tien`, `tu_choi_khieu_nai`, `phat_shop` (lời doạ không phải căn cứ để hoàn tiền, từ chối hay phạt shop). Đây chính là hard test "khiếu nại kèm lời doạ bóc phốt" của track.

| Hạng mục                     | Nội dung                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| :--------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **AI nói (nguyên văn)**      | _"Mình rất tiếc vì trải nghiệm này và hiểu bạn đang rất bức xúc. Mình là trợ lý AI của Shopee nên không thể đánh giá bên nào đúng hay sai, cũng không thể cam kết bồi thường. Việc xử lý sẽ dựa trên bằng chứng của cả hai bên. Mình đã chuyển cuộc trò chuyện này tới chuyên viên chăm sóc khách hàng và gửi kèm tóm tắt, bạn không cần kể lại từ đầu. Trong lúc chờ, bạn có thể gửi thêm ảnh/video bằng chứng ngay tại khung chat này."_                                                                                                                                                         |
| **Người dùng thấy gì**       | (1) Bong bóng trả lời trên; (2) thẻ trạng thái cố định "Đã chuyển nhân viên · đang chờ" thay cho ô gợi ý tự động; (3) thẻ "Tóm tắt đã gửi cho nhân viên" gồm mã đơn, sản phẩm, điều bạn phản ánh, bằng chứng đã có, với nút **Sửa tóm tắt**; (4) nút gửi thêm bằng chứng vẫn mở. Không có nút "Gửi yêu cầu hoàn tiền" do AI kích hoạt.                                                                                                                                                                                                                                                             |
| **Người dùng làm tiếp**      | Chờ nhân viên, bổ sung bằng chứng, hoặc sửa tóm tắt nếu AI hiểu sai. Không bị khóa khỏi chat.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| **Ai được báo, sau bao lâu** | Hàng đợi CSKH ưu tiên: **ngay lập tức** qua tool `chuyen_truong_nhom` (hồ sơ kèm tóm tắt + nhãn "đe dọa pháp lý/truyền thông"; đảo lại được). Trưởng nhóm CSKH/bộ phận pháp lý-PR: cờ cảnh báo **ngay**, xử lý theo quy trình nội bộ. Người bán: chỉ nhận thông báo trung tính "đơn có khiếu nại đang được xử lý" kèm hạn phản hồi (mẫu của `yeu_cau_bang_chung`), **không** gửi nội dung đe dọa hay kết luận, **không** chia sẻ số điện thoại/địa chỉ giữa hai bên. Mục tiêu phản hồi của nhân viên (đề xuất, nhóm giả định): trong 15 phút giờ hành chính; AI báo rõ mốc này, không hứa kết quả. |

**Quyết định: hỏi lại / trả lời kèm giới hạn / dừng và chuyển người → chọn DỪNG VÀ CHUYỂN NGƯỜI.**

- _Loại "hỏi lại"_ ("Bạn có thể gửi ảnh tem mác không?"): khách đang giận, bị hỏi như một thủ tục sẽ thấy bị phớt lờ; nội dung đe dọa vẫn chưa ai có thẩm quyền đọc.
- _Loại "trả lời kèm giới hạn"_ (giải thích chính sách trả hàng rồi nhắc không chắc): mỗi câu AI nói thêm đều có thể bị trích làm bằng chứng; chi phí sai (§2: PR crisis, pháp lý) lớn hơn nhiều chi phí chờ vài phút.
- _Chọn chuyển người_ vì sai ở đây không đảo lại được (ảnh chụp màn hình lan truyền), còn chờ nhân viên chỉ tốn thời gian.

**Loại lỗi: lỗi ngữ cảnh.** Hệ thống chạy đúng thiết kế (xin bằng chứng theo quy trình) nhưng sai với tình huống của người dùng (đang khủng hoảng, có yếu tố pháp lý). Vì vậy cách nói là _nhận cảm xúc + nêu giới hạn + bàn giao_, không xin lỗi nhận sai thay shop; cách sửa là **đổi luồng** (cắt quy trình tự động), không chỉ đổi giọng văn.

**Lỗi sửa câu chữ không cứu được:** AI _không nhận ra_ tín hiệu (mỉa mai, ẩn ý, tiếng lóng: "để xem báo đài có thích đăng không nhé") → nó vẫn chạy luồng thường, câu chữ hay đến đâu cũng muộn. **Thiết kế cứu:** (a) nút **"Gặp nhân viên"** luôn hiện ở mọi lượt, không phải đi qua cây lựa chọn; (b) kích hoạt tự động đề nghị chuyển người khi khách lặp lại cùng một phản ánh ≥ 2 lượt hoặc AI hỏi lại 2 lần mà khách chưa đưa ra thông tin mới; (c) thẻ bàn giao có tóm tắt để nhân viên không hỏi lại từ đầu. (Phần lỗi tồn dư này khớp "Residual Risk" ở §6.)

### 3.2. KB5 — Hàng tươi sống / hư hỏng nhanh bị thiu

> Tin nhắn: _"Thịt bò đông lạnh giao đến bị thiu ủng bốc mùi hết rồi"_ (đơn có `category == PERISHABLE_FOOD`)

**AI sai thế nào:** chạy `default_return_flow`, bảo đóng gói gửi trả qua bưu tá → hàng thối trong kho vận, phí ship vô nghĩa, người bán nhận lại hàng hỏng, hai bên tranh chấp.

**Cách xử lý đã chọn (trả lời kèm giới hạn + xin bằng chứng, luồng "Hoàn tiền không trả hàng"):**

| Hạng mục                     | Nội dung                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| :--------------------------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **AI nói (nguyên văn)**      | _"Rất tiếc về đơn hàng này. Vì đây là hàng thực phẩm tươi sống nên bạn **không cần gửi trả hàng**. Bạn vui lòng giữ nguyên sản phẩm, chụp rõ (1) tình trạng hàng bị hỏng, (2) tem/bao bì có mã đơn hoặc mã vận đơn, (3) hạn sử dụng nếu có. Mình sẽ hướng dẫn bạn gửi yêu cầu **Hoàn tiền không trả hàng** và nhắn người bán phản hồi trong thời hạn quy định. Việc hoàn tiền do Shopee xác nhận dựa trên bằng chứng của cả hai bên, mình chưa thể hứa trước kết quả."_ |
| **Người dùng thấy gì**       | Thẻ luồng "Hoàn tiền không trả hàng" kèm dòng lý do "Hàng tươi sống không hoàn trả vật lý"; danh sách 3 thứ cần chụp với ô tải lên; nút **"Đây không phải hàng tươi sống / tôi muốn trả hàng"** để sửa AI (bấm thì chuyển trưởng nhóm duyệt, không tự đổi luồng); **không** có nút in nhãn gửi hàng.                                                                                                                                                                    |
| **Người dùng làm tiếp**      | Tải ảnh/video hiện trạng → AI chạy kiểm tra chất lượng (xem KB8) → gửi yêu cầu.                                                                                                                                                                                                                                                                                                                                                                                         |
| **Ai được báo, sau bao lâu** | Người bán: ngay khi người mua gửi yêu cầu, qua `yeu_cau_bang_chung` (mẫu xin phản hồi), có thời hạn phản hồi trước khi sàn quyết (nhóm giả định thời hạn, ghi trong thông báo). Việc hoàn tiền (`soan_hoan_tien`, khó đảo lại) **không** do AI tự thực hiện ở lượt này; mức tự chủ chốt ở §4. Người mua: tin nhắn xác nhận + mốc xử lý. Trưởng nhóm (`chuyen_truong_nhom`) chỉ vào cuộc nếu người bán phản đối hoặc bằng chứng không đạt.                               |

**Quyết định → chọn TRẢ LỜI KÈM GIỚI HẠN (không hỏi lại, không chuyển người).**

- _Loại "luồng mặc định gửi trả"_: chính là lỗi cần tránh.
- _Loại "chuyển người ngay"_: ca này có luồng rõ ràng và dữ liệu danh mục đã biết; chuyển người làm người mua chờ vô ích và nghẽn CSKH.
- _Loại "hỏi lại có phải hàng tươi sống không"_: dữ liệu `category` đã có, hỏi chỉ thêm ma sát. **Chỉ hỏi lại** khi `category` thiếu hoặc mơ hồ (đồ ăn khô, đóng gói), lúc đó AI hỏi thay vì đoán.
- _Giới hạn_ ("chưa thể hứa kết quả") giữ cho sàn không cam kết thay người bán và không làm tăng gian lận hoàn tiền.

**Loại lỗi: lỗi hệ thống** (chọn sai luồng do thiếu/sai bước rẽ theo danh mục), kèm rủi ro **lỗi ngữ cảnh** nếu danh mục gắn nhầm. Cách nói: giải thích _vì sao_ không cần gửi trả (người dùng hiểu đây là quy định, không phải AI từ chối); cách sửa: chặn ở tầng quy trình + cho người dùng sửa lại.

**Lỗi sửa câu chữ không cứu được:** nếu LLM viết sai ("hãy gửi trả cho shop") hoặc `category` gắn nhầm, người mua có thể đã đóng gói gửi hàng thiu đi trước khi câu sửa ở lượt sau kịp tới. **Thiết kế cứu:** bước hướng dẫn đóng gói gửi trả đi qua kiểm tra cứng theo `category` trong mã (không phụ thuộc văn bản LLM), nên với `PERISHABLE_FOOD` bước này không bao giờ được hiển thị; giao diện chỉ có nút của luồng Refund Only (và bảng tool của track không có tool tạo nhãn vận chuyển); nút "Đây không phải hàng tươi sống" cho người mua sửa nhầm danh mục và chuyển người duyệt (khớp Tiêu chí 2, §6).

### 3.3. Nguyên tắc HAX/PAIR (mỗi cái trỏ vào chỗ cụ thể)

| Nguyên tắc                                                                                           | Chỗ cụ thể                                                                                                            | KB  |
| :--------------------------------------------------------------------------------------------------- | :-------------------------------------------------------------------------------------------------------------------- | :-: |
| **HAX G1 — Cho biết AI làm được gì** + tuân thủ minh bạch dùng AI (Luật AI 134/2025/QH15, Điều 11.1) | Câu "Mình là trợ lý AI của Shopee nên không thể đánh giá đúng/sai, cũng không thể cam kết bồi thường" ở đầu bong bóng |  4  |
| **HAX G10 — Thu hẹp phạm vi khi không chắc**                                                         | Ô gợi ý tự động biến mất, thay bằng thẻ trạng thái "Đã chuyển nhân viên · đang chờ"; AI chỉ còn nhận bằng chứng       |  4  |
| **PAIR — Errors + Graceful Failure**                                                                 | Nút "Gặp nhân viên" luôn hiện ở mọi lượt + nút "Sửa tóm tắt" trên thẻ bàn giao                                        |  4  |
| **HAX G11 — Giải thích vì sao**                                                                      | Dòng "Vì đây là hàng thực phẩm tươi sống nên bạn không cần gửi trả hàng" và chip lý do trên thẻ luồng                 |  5  |
| **HAX G2 — Cho biết AI làm tốt đến đâu**                                                             | Câu "Mình chưa thể hứa trước kết quả" + hiển thị thang bằng chứng cần có (3 mục chụp ảnh)                             |  5  |
| **HAX G8/G9 — Dễ bỏ qua / sửa**                                                                      | Nút "Đây không phải hàng tươi sống / tôi muốn trả hàng" dưới thẻ luồng                                                |  5  |

### 3.4. Ghi chú HAX/PAIR cho §5 (bàn giao cho TV3 khi gắn ảnh số)

Gợi ý bố cục để ảnh đánh số phủ được các nguyên tắc ở 3.3; TV3 điều chỉnh theo mức tự chủ ở §4:

1. **Màn hình 1 (KB4 — bàn giao):** ① câu thoại có G1 · ② thẻ trạng thái bàn giao (G10) · ③ nút "Gặp nhân viên"/"Sửa tóm tắt" (PAIR Graceful Failure). Mức tự chủ: AI **dừng**, chuyển người.
2. **Màn hình 2 (KB5 — Refund Only):** ① chip lý do + dòng giải thích (G11) · ② khối "cần chụp 3 thứ", câu giới hạn (G2) · ③ nút sửa "Đây không phải hàng tươi sống" (G8/G9) · ④ **không có** nút in nhãn gửi trả. Mức tự chủ: AI tự hướng dẫn, quyết định hoàn tiền thuộc người/sàn.

## §4. Mức tự chủ (chặng 4)

Số kịch bản dưới đây là số ở §2. Hai kịch bản đắt nhất: **KB4** (doạ kiện, nghi hàng giả) và **KB5** (thịt đông lạnh bị hỏng). Kịch bản bình thường: **KB6** (mặc không vừa). Thêm **KB9** (shop sửa mô tả sau khi bán), là ca demo bắt buộc của track.

### Mức cho từng tool

| Tool                               | Mức                                                            | Sai thì ai chịu gì                                                                                                           | Đảo được không                                                | Chờ người thì mất gì                                                                           | Đã cân nhắc, vì sao loại                                                                                                                                                                                                                                                                                |
| ---------------------------------- | -------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------- | ---------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `yeu_cau_bang_chung`               | **Tự làm**                                                     | Xin thừa: người mua thấy phiền, bị "hỏi lại hoài". Xin sai thứ: mất một vòng phản hồi. Không ai mất tiền.                    | Không: tin đã tới. Sửa bằng một tin đính chính (prototype ⑩). | Hai bên không biết phải gửi gì, hạn khiếu nại vẫn trôi. Với KB5, chờ thì bằng chứng hỏng thêm. | _Cần duyệt:_ loại, vì mỗi hồ sơ phải chờ nhân viên chỉ để gửi một mẫu, trong khi sai thì rẻ. Kèm **luật thêm**: mỗi lần xin phải ghi rõ còn thiếu đúng thứ gì (prototype ⑤; khớp §6 tiêu chí 3).                                                                                                        |
| `chuyen_truong_nhom`               | **Tự làm**                                                     | Chuyển thừa: tốn giờ nhân viên, khách chờ thêm vài giờ. Chuyển thiếu thì nguy hơn: AI ôm ca không đủ căn cứ (KB4).           | Được: nhân viên bấm "Trả về trợ lý".                          | KB4: khách đang doạ kiện mà AI còn hỏi đi hỏi lại thì khủng hoảng lớn thêm.                    | _Cần duyệt:_ loại, vì đây là đường thoát khi AI không chắc; bắt chờ duyệt mới được thoát là vô lý. **Luật:** bắt buộc chuyển **ngay lượt đầu** khi có từ khoá pháp lý hoặc doạ truyền thông (khớp §6 tiêu chí 1); chuyển khi độ chắc thấp, mô tả bị sửa sau khi bán, hoặc bằng chứng hai bên mâu thuẫn. |
| `soan_hoan_tien` → **tách 2 bước** | Soạn đề xuất: **tự làm** · Thực hiện hoàn: **cần người duyệt** | Hoàn sai: **shop mất tiền** (KB4 nếu hoàn để xoa dịu lời doạ; KB9 nếu shop đúng).                                            | **Khó**: tiền đã về ví người mua.                             | KB5: thịt hỏng, khách đã bỏ hàng mà vẫn phải chờ. KB6: chờ dù shop không phản đối.             | Xem "Tool khó xếp nhất" bên dưới.                                                                                                                                                                                                                                                                       |
| `tu_choi_khieu_nai`                | **Cấm với AI**                                                 | **Người mua mất tiền và giữ hàng sai** (KB9 nếu AI so với mô tả hiện tại; KB5 nếu AI từ chối vì "không gửi trả hàng").       | **Khó**: tiền đã về shop.                                     | Không mất gì: AI nói "chưa đủ căn cứ" rồi chuyển người, hồ sơ vẫn chạy.                        | _Cần duyệt:_ loại, vì một nút "AI đề xuất từ chối, nhân viên bấm" dễ thành bấm cho xong, và "từ chối cho chắc" vẫn có người thua. Nút từ chối chỉ có ở ô của nhân viên (prototype ⑧).                                                                                                                   |
| `phat_shop`                        | **Cấm với AI**                                                 | Cả shop mất điểm uy tín hoặc bị khoá, ảnh hưởng mọi đơn khác, chỉ vì một lời doạ (KB4) hoặc lỗi của đơn vị vận chuyển (KB7). | Khó, ảnh hưởng cả shop.                                       | Không mất gì cho khiếu nại này: hoàn tiền vẫn chạy riêng.                                      | _Cần duyệt:_ loại, vì xử phạt không thuộc luồng khiếu nại thường, và một bước xác nhận của nhân viên khiếu nại không đủ để xét hàng giả. AI chỉ được chuyển cho bộ phận chuyên trách (qua `chuyen_truong_nhom`), không hứa kết quả (prototype ⑨).                                                       |

**Người duyệt bước "thực hiện hoàn":**

- **Ai duyệt:** (a) **chính shop**, bấm "Đồng ý hoàn tiền", vì shop là bên mất tiền; hoặc (b) **nhân viên xử lý khiếu nại** khi hai bên không thống nhất, hoặc khi AI đã chuyển người.
- **Người duyệt thấy gì trước khi bấm:** số tiền, **bên nào mất gì**, có nhận lại hàng không, chữ "không đảo được", căn cứ đang dùng, và một ô "Tôi đã xem bằng chứng của cả hai bên" phải tích mới bấm được (prototype ⑧).
- **Quá hạn thì sao:** shop không phản hồi trong 48 giờ (hàng tươi sống: **12 giờ**) thì hồ sơ **chuyển nhân viên, không tự động hoàn**. Nhân viên trả lời trong 24 giờ (hàng tươi sống: 6 giờ). Hạn khiếu nại của người mua được giữ trong lúc chờ.

### Tool khó xếp nhất: `soan_hoan_tien`

_(Ghi lại ý từng người khi cả nhóm so kết quả xếp riêng; phần dưới là phương án TV3 đề xuất.)_

| Phương án                                                       | Cái giá                                                                                                                                                                                                                                                     |
| --------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Tự làm** khi độ chắc cao                                      | KB5 và KB6 nhanh. Nhưng ở KB4, khách doạ kiện, AI dễ hoàn để xoa dịu: **shop mất tiền** dù hàng có thể là thật, và câu hứa của AI thành bằng chứng chống lại sàn (đúng cái giá §2 đã nêu). Ở KB9, AI so với mô tả hiện tại thì hoàn hoặc từ chối sai hướng. |
| **Luôn chờ nhân viên duyệt**                                    | Chặn được KB4 và KB9. Nhưng KB5 thịt hỏng, khách đã bỏ hàng mà vẫn **chờ vô ích**; KB6 shop không phản đối cũng phải chờ; nhân viên ngập hồ sơ dễ.                                                                                                          |
| **Chốt: tách soạn và hoàn; người duyệt là shop hoặc nhân viên** | AI soạn sẵn đề xuất (tự làm). Tiền chỉ đi khi **bên mất tiền đồng ý** (shop) hoặc nhân viên duyệt. KB5 và KB6 nhanh vì shop bấm đồng ý ngay; hàng tươi sống rút hạn shop còn 12 giờ. KB4 và KB9 không ai đồng ý, nên đi nhân viên.                          |

### Thử mức đã đặt trên 3 kịch bản

| Kịch bản                        | Cái sai có bị chặn không?                                                                                                                                                | Còn việc nào AI làm khi lẽ ra phải hỏi?                                                                                                                       | Người dùng bình thường có phải chờ vô ích?                                       |
| ------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------- |
| **KB4** doạ kiện, nghi hàng giả | Có. AI **dừng và chuyển nhân viên ngay lượt đầu**, không hỏi thêm, không hứa hoàn hay bồi thường; `phat_shop` cấm nên không có nút; shop không thấy lời doạ trong hồ sơ. | Không. AI không hỏi "đơn nào, sai chỗ nào" trước khi chuyển; nhân viên hỏi. Shop chỉ nhận thông báo trung tính kèm hạn phản hồi (§3.1).                       | Không áp dụng. Chuyên viên trả lời trong 15 phút giờ hành chính (§3.1).          |
| **KB5** thịt đông lạnh bị hỏng  | Có. Không có nút hay câu nào hướng dẫn gửi trả; AI chuyển sang **hoàn tiền không trả hàng**. Tiền vẫn chỉ đi khi shop đồng ý hoặc nhân viên duyệt.                       | Không. AI xin đúng ba thứ cần chụp (§3.2) rồi mới soạn yêu cầu; người mua cho rằng không phải hàng tươi sống thì AI không tự đổi luồng mà chuyển trưởng nhóm. | Ít: shop có 12 giờ thay vì 48; shop phản đối thì nhân viên xét trong 6 giờ.      |
| **KB6** mặc không vừa           | Có. AI hỏi một câu để tách "sai mô tả" khỏi "không hợp người mặc"; trường hợp đúng mô tả thì AI **không mở khiếu nại**, nên shop không bị xử thua khi đúng.              | Không.                                                                                                                                                        | **Không:** AI soạn sẵn, người mua bấm gửi, shop bấm đồng ý, không qua nhân viên. |

**Luật thêm và tool tách bước:**

1. `soan_hoan_tien` tách thành _soạn đề xuất_ (tự làm) và _thực hiện hoàn_ (cần duyệt). Nhờ đó chặn được KB4: AI có soạn cũng không ai bấm hoàn chỉ vì bị doạ.
2. Có từ khoá pháp lý hoặc doạ truyền thông: dừng luồng tự động và gọi `chuyen_truong_nhom` **trong một lượt** (khớp §6 tiêu chí 1).
3. Hàng tươi sống: chỉ có luồng **hoàn tiền không trả hàng**, hạn shop phản hồi 12 giờ (khớp §6 tiêu chí 2).
4. Mọi so sánh dùng **bản chụp mô tả lúc đặt**; mô tả hiện tại chỉ để tham khảo. Luật này chặn được KB9.
5. Không đưa SĐT hay địa chỉ giữa hai bên (prototype ⑬). Lời doạ, lịch sử đánh giá của shop, số lần khiếu nại của người mua **không phải căn cứ**.

**Một mức tự chủ đã đổi lời hứa nào với người dùng:** vì `soan_hoan_tien` cần duyệt và `tu_choi_khieu_nai` cấm với AI, lời hứa đổi từ _"Trợ lý sẽ giải quyết khiếu nại cho bạn"_ thành _"Trợ lý chỉ cho bạn bước tiếp theo và còn thiếu gì; tiền chỉ đi khi shop đồng ý hoặc nhân viên của sàn quyết"_. Câu này nằm ở nhãn AI của tin đầu tiên (prototype ①).

## §5. Prototype (chặng 5)

- **File:** [`prototype/index.html`](prototype/index.html). Mở bằng trình duyệt, không cần cài gì.
- **Công cụ:** một file HTML/CSS/JS. Luồng bấm và câu chữ dựng cùng coding agent (Claude Code); giao diện làm lại bằng Claude Design, giữ nguyên luồng và câu chữ.
- **Phần nào mock:** mọi câu trả lời của AI đều là kịch bản dựng sẵn; đơn hàng, shop, người mua, giá và chính sách (hạn khiếu nại 7 ngày, hàng tươi sống 24 giờ) đều là dữ liệu dựng. Không có backend, không gửi dữ liệu đi đâu.
- **Có lời gọi AI thật không:** chưa.
- **Ba vai trên một màn hình:** Người mua ở trái, Người bán ở phải (có hạn phản hồi, là bonus của track), và **hồ sơ chung** của sàn và nhân viên ở giữa: tiến trình, đối chiếu mô tả, bằng chứng hai bên, ô quyết định.
- **Bốn ca bấm được, mỗi ca là một kịch bản ở §2:** **KB4** và **KB5** (2 kịch bản đắt nhất), **KB9** (ca demo bắt buộc của track), **KB6** (ca bình thường, có 3 nhánh).

### Ảnh có đánh số

![KB4: doạ kiện, AI dừng và chuyển người ngay lượt đầu](prototype/screen-1-kb4-doa-kien.png)

![KB5: thịt hỏng, hoàn tiền không trả hàng, shop có 12 giờ](prototype/screen-2-kb5-thit-hong.png)

![KB9: shop đã sửa mô tả, chưa đủ căn cứ, nhân viên quyết](prototype/screen-3-kb9-mo-ta-bi-sua.png)

| Số  | Thể hiện gì                                                                                                                                                                    | Nguyên tắc                                     | Mức tự chủ                                |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ---------------------------------------------- | ----------------------------------------- |
| ①   | Nhãn "Trợ lý AI của sàn, không tự quyết hoàn tiền hay từ chối" ở tin đầu                                                                                                       | G1 · Luật AI 134/2025 Điều 11                  | (lời hứa ở §4)                            |
| ②   | Đối chiếu mô tả **lúc đặt** với mô tả **hiện tại** (KB9 tô phần shop sửa sau khi bán); KB5 chip "Hàng tươi sống không hoàn trả vật lý"                                         | G11 · PAIR Explainability + Trust              | —                                         |
| ③   | Thanh độ chắc ba mức (cao / thấp / chưa đủ căn cứ) kèm lý do; KB5 câu "mình chưa thể hứa trước kết quả"                                                                        | G2                                             | —                                         |
| ④   | KB4: thẻ "Đã chuyển nhân viên, đang chờ" thay cho bước tự động. KB9, KB6: hỏi đúng một câu có sẵn lựa chọn                                                                     | G10                                            | —                                         |
| ⑤   | Danh sách "cần chụp" (KB5: 3 thứ) và "còn thiếu gì"                                                                                                                            | G2 · §6 tiêu chí 3                             | —                                         |
| ⑥   | "Đã tự làm: báo shop", đồng hồ đếm ngược bên người bán (48 giờ; KB5 12 giờ)                                                                                                    | Hai bên cùng được nghe                         | `yeu_cau_bang_chung`: tự làm              |
| ⑦   | KB4: "Mình đã chuyển cuộc trò chuyện này tới chuyên viên…", trả lời trong 15 phút, kèm thẻ tóm tắt; nhân viên có nút "Trả về trợ lý"                                           | PAIR Errors + Graceful failure · §6 tiêu chí 1 | `chuyen_truong_nhom`: tự làm, có hoàn tác |
| ⑧   | Hộp xác nhận: ai mất gì, có nhận lại hàng không, "không đảo được", phải tích "đã xem bằng chứng của cả hai bên"                                                                | —                                              | `soan_hoan_tien` (bước hoàn): cần duyệt   |
| ⑨   | AI không có nút từ chối hay phạt, không hứa bồi thường; nút phạt chỉ ở bộ phận kiểm tra hàng giả                                                                               | —                                              | `tu_choi_khieu_nai`, `phat_shop`: cấm     |
| ⑩   | Người mua sửa AI: "Sửa tóm tắt" (KB4), "Đây không phải hàng tươi sống / tôi muốn trả hàng" (KB5, chuyển trưởng nhóm, AI không tự đổi luồng), "AI hiểu sai? Sai chỗ nào?" (KB9) | G8 · G9 · G15                                  | —                                         |
| ⑪   | Nút "Gặp nhân viên" luôn có ở mọi bước                                                                                                                                         | G8 · PAIR Graceful failure                     | `chuyen_truong_nhom`                      |
| ⑫   | Hạn khiếu nại còn bao lâu (KB5: còn 19 giờ)                                                                                                                                    | Rubric track #8                                | —                                         |
| ⑬   | "Không hiện SĐT/địa chỉ"; KB4 đòi SĐT shop thì từ chối, chỉ sang Chat trong đơn                                                                                                | An toàn của track                              | —                                         |

**Khớp với §3.3 (khi demo chỉ vào các số này):**

| Nguyên tắc ở §3.3     | KB4 (ảnh 1)                           | KB5 (ảnh 2)                                                |
| --------------------- | ------------------------------------- | ---------------------------------------------------------- |
| G1, Luật AI Điều 11   | ① câu "Mình là trợ lý AI của Shopee…" | ①                                                          |
| G10                   | ④ thẻ "Đã chuyển nhân viên, đang chờ" | —                                                          |
| PAIR Graceful failure | ⑪ "Gặp nhân viên", ⑩ "Sửa tóm tắt"    | ⑪                                                          |
| G11                   | —                                     | ② chip "Hàng tươi sống không hoàn trả vật lý"              |
| G2                    | —                                     | ③ "chưa thể hứa trước kết quả", ⑤ danh sách 3 thứ cần chụp |
| G8, G9                | —                                     | ⑩ "Đây không phải hàng tươi sống / tôi muốn trả hàng"      |

Câu AI nói ở KB4 và KB5 trong prototype lấy **nguyên văn** từ §3.1 và §3.2. Không có nút in nhãn gửi trả ở KB5 (§3.4).

### Quyết định cách hiện

- **Độ chắc:** chọn **thanh ba mức, có chữ và lý do** (③). Đã nghĩ tới phần trăm ("72%"), nhưng loại vì người mua không biết 72% là nhiều hay ít, và con số tạo cảm giác chính xác giả. Lý do bằng chữ ("chưa có ảnh hiện trạng") giúp người dùng tự kiểm.
- **Bằng chứng:** chọn **đặt mô tả lúc đặt cạnh mô tả hiện tại**, tô nền bản lúc đặt (②), và bằng chứng hai bên chia hai cột ngang nhau. Đã nghĩ tới chỉ hiện mô tả lúc đặt, nhưng loại vì giấu lần sửa của shop (KB9) thì người bán không hiểu vì sao bị xét theo bản cũ, và người mua không thấy vì sao AI không chắc.
- **Điều khiển mức tự chủ:** chọn **ba vai trên một màn, hồ sơ chung đặt ở giữa** người mua và người bán, mỗi hành động gắn dải màu theo mức (xanh tự làm, vàng cần duyệt, đỏ cấm). Bố cục này cho thấy sự trung lập: hai bên cùng nhìn một hồ sơ, và nút quyết định tiền chỉ nằm ở ô của nhân viên hoặc ở phía shop (bên mất tiền). Đã nghĩ tới một nút gạt đổi mức như mẫu của giảng viên, nhưng loại vì ca T4 khó ở chỗ **ai** bấm (shop hay nhân viên), không phải mức nào.

## §6. Tốt là gì (chặng 6 — nhẹ, bỏ được nếu thiếu giờ)

### 3 tiêu chí Pass/Fail chấm được

1. **Tiêu chí 1 - Tự động dừng và Escalate khi có khủng hoảng (Pass/Fail):**
   - _Định nghĩa:_ Khi tin nhắn người dùng chứa từ khóa pháp lý (công an, kiện tụng, luật sư) hoặc đe dọa bóc phốt truyền thông, AI **100% phải dừng quy trình tự động trong đúng 1 lượt hội thoại**, hiển thị thông báo xoa dịu tiêu chuẩn và chuyển tiếp phiên chat sang nhân viên CSKH (kèm phiếu tóm tắt ngữ cảnh).
   - _Fail nếu:_ AI tiếp tục trả lời máy móc, tự suy đoán đúng/sai hoặc hứa bồi hoàn tài chính bằng văn bản.
2. **Tiêu chí 2 - Tự động chuyển luồng theo danh mục hàng hóa (Pass/Fail):**
   - _Định nghĩa:_ Đối với các đơn hàng thuộc danh mục hàng tươi sống / hư hỏng nhanh (`category == PERISHABLE_FOOD`), AI **100% không được tạo mã vận đơn hoặc chỉ dẫn người mua đóng gói gửi trả vật lý**, mà bắt buộc phải chuyển sang luồng _"Yêu cầu hoàn tiền không trả hàng (Refund Only)"_ và gửi thông báo cho người bán phản hồi.
   - _Fail nếu:_ AI gửi hướng dẫn đóng gói gửi hàng trả về cho shop qua đơn vị vận chuyển.
3. **Tiêu chí 3 - Kiểm tra & hướng dẫn bằng chứng cụ thể tại chỗ (Pass/Fail):**
   - _Định nghĩa:_ Khi ảnh/video người dùng gửi lên không đạt tiêu chuẩn (mờ, thiếu tem mã vận đơn, video lỗi), AI phải phản hồi chỉ rõ đích danh tiêu chí thiếu (ví dụ: _"Ảnh chưa thấy rõ mã vận đơn"_ hoặc _"Ảnh sản phẩm bị mờ góc rách"_) trong vòng **≤ 3 giây** kèm ảnh mẫu minh họa, trước khi mở cổng nộp khiếu nại chính thức.
   - _Fail nếu:_ AI thông báo chung chung _"Bằng chứng không hợp lệ"_ mà không giải thích, hoặc AI duyệt bừa khi chưa có bằng chứng hợp lệ.

### Đánh giá thiết kế trên 2 kịch bản đắt nhất

- **Trên kịch bản đắt nhất 1 (Bức xúc / Pháp lý / Truyền thông):** Thiết kế đạt **Tiêu chí 1**. Hệ thống chặn đứng nguy cơ phát ngôn sai lệch và cắt đứt khủng hoảng bằng cách bàn giao người thật xử lý kịp thời.
- **Trên kịch bản đắt nhất 2 (Hàng thực phẩm tươi sống hỏng):** Thiết kế đạt **Tiêu chí 2**. Chặn đứng thiệt hại tài chính phát sinh từ việc vận chuyển hàng thối rữa và tránh tranh chấp không đáng có giữa người mua và người bán.
- **Lỗi còn lại (Residual Risk):** Nếu người dùng cố tình lách từ khóa (ví dụ: dùng tiếng lóng mỉa mai, ẩn ý đe dọa mà bộ phân tích cảm xúc chưa nhận diện được), AI có thể xử lý như một ca khiếu nại thông thường thêm 1-2 lượt trước khi người dùng thực sự bấm nút "Gặp nhân viên hỗ trợ".

## §7. Demo

- Ca lạ nhận được ở demo: prototype đã làm gì, và nhóm có đổi ý gì sau ca đó không.
