# Bài nộp LAB5 — T4 Thương mại điện tử · Nhóm top1build

**Lớp:** L3A · **Bàn / zone:** \_\_\_\_ · **Track:** T4 Thương mại điện tử

| Họ và Tên        | Mã Học Viên | Việc đã làm                                                             |
| ---------------- | ----------- | ----------------------------------------------------------------------- |
| Nguyễn Quang Đạo | 2A202603614 | Phụ trách §1, §2 (Kịch bản rủi ro & Biện luận), §6 (Tiêu chí Pass/Fail) |
|                  |             |                                                                         |
|                  |             |                                                                         |

## §1. Lát cắt & phạm vi

- **Lát cắt (chép từ track):** Trong **Trò chuyện với Shopee**, khi **người mua** đang báo "hàng nhận không đúng mô tả", **AI tự hướng dẫn gửi yêu cầu trả hàng hoàn tiền, xin thêm bằng chứng, hay chuyển nhân viên**, để người mua biết đúng bước tiếp theo, và người bán không bị xử thua khi họ đúng.

**3 việc tính năng KHÔNG làm:**
1. **Không tự động duyệt hoàn tiền ngay lập tức:** Vì rủi ro gian lận (buyer fraud) và xâm hại quyền lợi người bán; các giao dịch bồi hoàn tài chính không thể tự đảo ngược khi người mua gửi hàng giả hoặc ảnh mượn.
2. **Không tự đưa ra phán quyết pháp lý hoặc tuyên bố bên nào đúng/sai trong chat:** Vì AI chỉ là công cụ hỗ trợ đối soát quy trình và làm việc theo xác suất, không thể bảo đảm 100% cập nhật đúng luật hay có đủ ngữ cảnh thực tế, phát ngôn khẳng định sai sẽ đẩy Shopee vào rủi ro tranh chấp pháp lý và khiếu kiện.
3. **Không xử lý thanh toán bồi hoàn ngoài sàn hoặc chia sẻ số tài khoản ngân hàng cá nhân:** Vì vi phạm chính sách giao dịch an toàn của sàn, tiếp tay cho hành vi lừa đảo qua kênh ngoài.

## §2. Kịch bản rủi ro (chặng 2)

### Bảng kịch bản rủi ro (8 kịch bản phủ 4 lớp ①②③④)

| STT | Tin nhắn người dùng | Lớp | AI có thể sai thế nào (Trỏ về tool/dữ liệu) | Sản phẩm tốt nên làm gì | Nhãn rủi ro |
| :---: | :--- | :---: | :--- | :--- | :---: |
| 1 | *"Shop ơi áo này vải mỏng dính không giống ảnh quảng cáo, hoàn tiền gấp"* | ① Ngữ cảnh / Ý định | AI hiểu nhầm là hủy đơn thay vì khiếu nại trả hàng; hoặc vội vàng gửi link tạo yêu cầu Trả hàng khi dữ liệu hệ thống báo đơn chưa giao thành công (`order.status != DELIVERED`). | Tra cứu `order.status`: nếu đang giao -> hướng dẫn chờ nhận; nếu đã giao -> nhắc giữ nguyên tem mác, bao bì và hướng dẫn các bước mở khiếu nại đúng chuẩn. | Sửa được |
| 2 | *"Hàng giao thiếu 1 chiếc, nhưng tôi lỡ bấm 'Đã nhận được hàng' rồi"* | ② Dữ liệu / Trạng thái | AI chỉ đọc cờ trạng thái bề mặt `order.status == COMPLETED` -> vội vàng từ chối: *"Đơn đã hoàn tất, không thể khiếu nại"*, bỏ qua trường `delivered_timestamp` và chính sách Shopee Guarantee (cho phép khiếu nại trong 3-7 ngày sau nhận). | Kiểm tra `delivered_timestamp` và chính sách bảo vệ người mua: Nếu còn hạn -> hướng dẫn gửi yêu cầu trả hàng/hoàn tiền lý do "Thiếu hàng" và kích hoạt hỗ trợ mở lại case. | Không chấp nhận được |
| 3 | *"Gửi video bóc hàng dài 5 phút nhưng web báo lỗi dung lượng, tôi chỉ up được 2 ảnh chụp ngoài vỏ hộp"* | ③ Bằng chứng / Xác thực | Tool `validate_evidence` kiểm tra thấy thiếu file video theo quy tắc cứng -> AI thẳng tay từ chối khiếu nại hoặc đóng luồng hỗ trợ vì "thiếu bằng chứng hợp lệ". | Nhận diện lỗi kỹ thuật upload; chấp nhận tạm thời 2 ảnh rõ mã vận đơn và hướng dẫn nén video, hoặc cung cấp link upload bổ sung an toàn / chuyển nhân viên hỗ trợ tiếp nhận. | Sửa được |
| 4 | *"Shop lừa đảo, hàng fake 100%, tao sẽ kiện lên công an và đăng bóc phốt khắp mạng xã hội"* | ④ Xúc cảm / Tranh chấp / Pháp lý | AI thiếu khả năng phân tích cảm xúc cực độ (`sentiment_score`), phản hồi máy móc dập khuôn: *"Vui lòng gửi ảnh chụp tem mác"* khiến khách thêm phẫn nộ; hoặc AI tự ý hứa bồi thường để xoa dịu làm lộ sơ hở pháp lý. | Nhận diện từ khóa pháp lý/đe dọa truyền thông -> Dừng quy trình tự động; dùng câu thoại xoa dịu tiêu chuẩn của Shopee và lập tức kích hoạt escalate chuyển chuyên viên CSKH ưu tiên xử lý. | Không chấp nhận được |
| 5 | *"Sản phẩm là thịt bò đông lạnh/hoa quả tươi giao đến nơi bị thiu ủng bốc mùi hết rồi"* | ② Dữ liệu / Loại hàng | AI dùng quy trình mặc định (`default_return_flow`) hướng dẫn đóng gói gửi hàng trả về cho shop qua bưu tá, trong khi danh mục là hàng tươi sống (`category == PERISHABLE_FOOD`). | Nhận diện danh mục hàng không thể trả hàng vật lý -> Chuyển luồng sang "Yêu cầu hoàn tiền không trả hàng" (Refund Only), xin ảnh/video hiện trạng hỏng và gửi thông báo cho người bán phản hồi. | Không chấp nhận được |
| 6 | *"Tôi muốn trả hàng vì mặc không vừa, nhưng chủ shop bảo không hỗ trợ đổi size"* | ① Ngữ cảnh / Ý định | AI nhầm lẫn giữa "Hàng không đúng mô tả" (lỗi người bán - miễn phí trả hàng) và "Đổi ý / Không vừa" (lỗi người mua - theo chính sách đổi trả) dẫn tới tranh chấp phí vận chuyển trả hàng. | Đặt câu hỏi làm rõ: *"Sản phẩm nhận được có đúng kích cỡ/số đo theo bảng size của shop không hay do bạn mặc chật?"* để phân loại đúng mã lý do trả hàng. | Nhẹ |
| 7 | *"Hộp quà Tết bị móp méo rách nát lúc shipper đưa, tôi chụp lại từ chối nhận rồi"* | ④ Tranh chấp / Đơn vị vận chuyển | AI ghi nhận là lỗi sản phẩm của người bán (`seller_fault`), tự động trừ điểm vận hành (sao chép phạt) của shop thay vì nhận diện lỗi do đơn vị vận chuyển (`logistics_damage`). | Đối soát dữ liệu vận chuyển và biên bản đồng kiểm; phân định lỗi vận chuyển để kích hoạt quy trình bồi thường từ đơn vị vận chuyển (3PL), bảo vệ quyền lợi người bán. | Sửa được |
| 8 | Khách gửi ảnh chụp màn hình hóa đơn chuyển khoản ngân hàng hoặc ảnh chụp mờ tịt không thấy mã đơn | ③ Bằng chứng / Xác thực | AI dựa vào số lượng file (`file_count >= 1`) mà không kiểm tra nội dung/chất lượng ảnh -> thông báo *"Đã đủ bằng chứng, vui lòng đợi duyệt"*, sau đó nhân viên duyệt từ chối khiến khách ức chế. | Chạy kiểm tra chất lượng ảnh (`image_quality_check`): Nếu ảnh mờ hoặc sai đối tượng -> phản hồi ngay lập tức, nêu rõ tiêu chí còn thiếu và cung cấp ảnh mẫu hướng dẫn chụp rõ tem vận đơn + chi tiết lỗi. | Nhẹ |

### Quyết định: 2 kịch bản đắt nhất & 1 kịch bản suýt chọn

1. **Kịch bản đắt nhất 1: Kịch bản số 4 (Buyer tức giận tột độ, tố cáo lừa đảo, đe dọa kiện tụng & bóc phốt truyền thông)**
   - **Vì sao đắt nhất:** Chi phí sai sót (Cost of Error) ở kịch bản này không dừng lại ở giá trị đơn hàng mà đe dọa trực tiếp đến **uy tín thương hiệu (PR crisis), nguy cơ pháp lý và sự an toàn vận hành của sàn**. Nếu AI trả lời dập khuôn hoặc tự hứa hẹn bồi thường sai luật, câu từ của AI sẽ trở thành bằng chứng chống lại Shopee trước cơ quan chức năng hoặc bị phát tán làm khủng hoảng truyền thông.
2. **Kịch bản đắt nhất 2: Kịch bản số 5 (Hàng thực phẩm tươi sống / Hàng hư hỏng nhanh giao hoàn trả vật lý)**
   - **Vì sao đắt nhất:** Gây **thiệt hại tài chính và hậu cần không thể đảo ngược (irreversible)**. Nếu AI chỉ dẫn đóng gói gửi trả qua bưu tá cho thực phẩm ôi thiu: hàng sẽ thối rữa trong kho trung chuyển gây ô nhiễm môi trường/kho bãi, phát sinh chi phí vận chuyển vô nghĩa, người bán nhận hàng hỏng không thể bán lại, đẩy người mua và người bán vào tranh chấp gay gắt.
3. **Kịch bản suýt chọn: Kịch bản số 2 (Đã bấm 'Đã nhận được hàng' nhưng giao thiếu hàng)**
   - **Vì sao suýt chọn:** Tần suất xảy ra rất cao trong thực tế, dễ gây mất niềm tin của khách vào cơ chế bảo vệ của Shopee ("Shopee Guarantee").
   - **Vì sao không chọn:** Dù gây ức chế, nhưng sai sót này **hoàn toàn có thể sửa chữa/đảo ngược được (reversible)** thông qua quy trình hỗ trợ bổ sung (mở ticket khiếu nại nội bộ hoặc nhân viên can thiệp mở lại cổng hoàn tiền), không để lại hậu quả phá hủy vật chất hay rủi ro pháp lý/truyền thông nghiêm trọng như 2 kịch bản trên.

## §3. Khi AI sai (chặng 3)

Với mỗi kịch bản đắt nhất:

- AI nói gì (nguyên văn) · người dùng thấy gì · làm gì tiếp · ai được báo, sau bao lâu.
- Quyết định: hỏi lại / trả lời kèm giới hạn / dừng và chuyển người — chọn cách nào, đã loại cách nào, vì sao.
- Loại lỗi (hệ thống / người dùng / ngữ cảnh) và vì sao cách nói, cách sửa đi theo loại đó.
- 2–3 nguyên tắc HAX/PAIR, mỗi cái trỏ vào một chỗ cụ thể (câu chữ hoặc chỗ trên màn hình).
- Lỗi nào sửa câu chữ không cứu được (nếu có), và thiết kế cứu nó thế nào.

## §4. Mức tự chủ (chặng 4)

> Tên kịch bản dưới đây dùng nhãn tạm, sẽ đổi sang số kịch bản của §2 khi TV1 chốt bảng:
> **[KB-sửa]** shop sửa mô tả sau khi bán (nồi chiên "6 lít" → "nồi 6L, giỏ 5L") · **[KB-doạ]** tin cụt + doạ bóc phốt + đòi khoá shop · **[KB-thường]** áo đặt L nhận M, shop đã nhắn nhận gửi nhầm.

### Mức cho từng tool

| Tool | Mức | Sai thì ai chịu gì | Đảo được không | Chờ người thì mất gì | Đã cân nhắc, vì sao loại |
|---|---|---|---|---|---|
| `yeu_cau_bang_chung` | **Tự làm** | Xin thừa: người mua thấy phiền, bị "hỏi lại hoài". Xin sai thứ: mất một vòng 48 giờ. Không ai mất tiền. | Không: tin đã tới. Sửa bằng một tin đính chính (prototype ⑩). | Hai bên không biết phải gửi gì; hạn trả hàng vẫn trôi. | *Cần duyệt:* loại, vì mỗi hồ sơ phải chờ nhân viên chỉ để gửi một mẫu, trong khi sai thì rẻ. Kèm **luật thêm**: mỗi lần xin phải ghi rõ còn thiếu đúng thứ gì (prototype ⑤), không xin lại thứ đã có. |
| `chuyen_truong_nhom` | **Tự làm** | Chuyển thừa: tốn giờ nhân viên, khách chờ thêm vài giờ. Chuyển thiếu thì nguy hơn: AI ôm ca nó không đủ căn cứ. | Được: nhân viên bấm "Trả về trợ lý". | Ca khó nằm lâu trong luồng tự động, hạn trả hàng trôi. | *Cần duyệt:* loại, vì chính đây là đường thoát khi AI không chắc; bắt chờ duyệt mới được thoát là vô lý. **Luật:** bắt buộc chuyển khi độ chắc thấp, mô tả bị sửa sau khi bán, bằng chứng hai bên mâu thuẫn, có lời doạ hoặc đòi phạt shop. |
| `soan_hoan_tien` → **tách 2 bước** | Soạn đề xuất: **tự làm** · Thực hiện hoàn: **cần người duyệt** | Hoàn sai: **shop mất tiền và hàng** ([KB-sửa], [KB-doạ]). | **Khó**: tiền đã về ví người mua. | [KB-thường]: người mua chờ dù shop đã nhận lỗi. | Xem "Tool khó xếp nhất" bên dưới. |
| `tu_choi_khieu_nai` | **Cấm với AI** | **Người mua mất tiền và giữ hàng sai** ([KB-sửa] nếu AI so với mô tả hiện tại). | **Khó**: tiền đã về shop. | Không mất gì: AI nói "chưa đủ căn cứ" rồi chuyển người, hồ sơ vẫn chạy. | *Cần duyệt:* loại, vì một nút "AI đề xuất từ chối, nhân viên bấm" dễ thành bấm cho xong, và "từ chối cho chắc" vẫn có người thua. Nút từ chối chỉ có ở màn hình nhân viên (prototype ⑧). |
| `phat_shop` | **Cấm với AI** | Cả shop mất điểm uy tín hoặc bị khoá, ảnh hưởng mọi đơn khác, chỉ vì một lời doạ ([KB-doạ]). | Khó, ảnh hưởng cả shop. | Không mất gì cho khiếu nại này: hoàn tiền vẫn chạy riêng. | *Cần duyệt:* loại, vì xử phạt không thuộc luồng khiếu nại thường, và một bước xác nhận của nhân viên khiếu nại không đủ để xét hàng giả. AI chỉ được chuyển cho bộ phận chuyên trách (qua `chuyen_truong_nhom`), không hứa kết quả (prototype ⑨). |

**Người duyệt bước "thực hiện hoàn":**

- **Ai duyệt:** (a) **chính shop**, bấm "Đồng ý trả hàng hoàn tiền", vì shop là bên mất tiền; hoặc (b) **nhân viên xử lý khiếu nại** khi hai bên không thống nhất, hoặc khi AI đã chuyển người.
- **Người duyệt thấy gì trước khi bấm:** số tiền, **bên nào mất gì**, chữ "không đảo được", căn cứ đang dùng (mô tả lúc đặt và bằng chứng hai bên), và một ô "Tôi đã xem bằng chứng của cả hai bên" phải tích mới bấm được (prototype ⑧).
- **Quá hạn thì sao:** shop không phản hồi trong 48 giờ thì hồ sơ **chuyển nhân viên, không tự động hoàn**. Nhân viên phải trả lời khách trong 24 giờ. Hạn trả hàng của người mua được giữ trong lúc chờ.

### Tool khó xếp nhất: `soan_hoan_tien`

*(Ghi lại ý từng người khi cả nhóm so kết quả xếp riêng; phần dưới là phương án TV3 đề xuất.)*

| Phương án | Cái giá |
|---|---|
| **Tự làm** khi độ chắc cao | [KB-thường] nhanh. Nhưng ở [KB-doạ], người mua doạ, độ chắc "cao" giả, **shop mất tiền**. Ở [KB-sửa], AI so với mô tả hiện tại thì đúng hướng ngược lại. |
| **Luôn chờ nhân viên duyệt** | Chặn được [KB-sửa] và [KB-doạ]. Nhưng [KB-thường] shop đã nhận lỗi mà người mua vẫn **chờ vô ích**, và nhân viên ngập hồ sơ dễ. |
| **Chốt: tách soạn và hoàn; người duyệt là shop hoặc nhân viên** | AI soạn sẵn đề xuất (tự làm). Tiền chỉ đi khi **bên mất tiền đồng ý** (shop) hoặc nhân viên duyệt. [KB-thường] nhanh vì shop bấm đồng ý ngay. [KB-sửa] và [KB-doạ] không ai đồng ý, nên đi nhân viên. |

### Thử mức đã đặt trên 3 kịch bản

| Kịch bản | Cái sai có bị chặn không? | Còn việc nào AI làm khi lẽ ra phải hỏi? | Người dùng bình thường có phải chờ vô ích? |
|---|---|---|---|
| [KB-sửa] | Có. AI so với mô tả **lúc đặt**, không có nút hoàn hay từ chối, và tự chuyển nhân viên khi bằng chứng mâu thuẫn. | Không. "6 lít" mơ hồ thì AI hỏi một câu (đo phần nào) trước khi xin bằng chứng. | Người mua chờ tối đa 24 giờ sau khi shop phản hồi; hạn trả hàng được giữ. |
| [KB-doạ] | Có. Lời doạ không đổi mức nào; `phat_shop` cấm nên không có nút; shop không thấy lời doạ trong hồ sơ. | Không. Tin cụt thì AI hỏi "đơn nào?" và chọn từ danh sách, không đoán. | Không áp dụng. |
| [KB-thường] | Không có cái sai để chặn. | Không. | **Không:** AI soạn sẵn, người mua bấm gửi, shop bấm đồng ý, không qua nhân viên. |

**Luật thêm và tool tách bước:**

1. `soan_hoan_tien` tách thành *soạn đề xuất* (tự làm) và *thực hiện hoàn* (cần duyệt). Nhờ đó chặn được [KB-doạ]: AI có soạn cũng không ai bấm hoàn chỉ vì bị doạ.
2. Mọi so sánh dùng **bản chụp mô tả lúc đặt**; mô tả hiện tại chỉ để tham khảo. Luật này chặn được [KB-sửa].
3. Không đưa SĐT hay địa chỉ giữa hai bên (prototype ⑬).
4. Lịch sử đánh giá của shop và số lần khiếu nại của người mua **không phải căn cứ** (hiện trong hồ sơ nhân viên).

**Một mức tự chủ đã đổi lời hứa nào với người dùng:** vì `soan_hoan_tien` cần duyệt và `tu_choi_khieu_nai` cấm với AI, lời hứa đổi từ *"Trợ lý sẽ giải quyết khiếu nại cho bạn"* thành *"Trợ lý chỉ cho bạn bước tiếp theo và còn thiếu gì; tiền chỉ đi khi shop đồng ý hoặc nhân viên của sàn quyết"*. Câu này nằm ngay trên nhãn AI ở mọi tin (prototype ①).

## §5. Prototype (chặng 5)

- **File:** [`prototype/index.html`](prototype/index.html). Mở bằng trình duyệt, không cần cài gì.
- **Công cụ:** một file HTML/CSS/JS, dựng cùng coding agent.
- **Phần nào mock:** mọi câu trả lời của AI đều là kịch bản dựng sẵn; đơn hàng, shop, người mua, giá và chính sách (hạn trả hàng 7 ngày) đều là dữ liệu dựng. Không có backend, không gửi dữ liệu đi đâu.
- **Có lời gọi AI thật không:** chưa.
- **Ba vai cạnh nhau cho cùng một hồ sơ:** Người mua ở trái, Người bán ở phải (có hạn phản hồi, là bonus của track), và **hồ sơ chung** của sàn và nhân viên nằm ở giữa: tiến trình, đối chiếu mô tả, bằng chứng hai bên, ô quyết định. Có 3 ca để bấm: **Ca A** (ca khó, mô tả bị sửa: ca demo bắt buộc của track), **Ca B** (bình thường), **Ca C** (doạ, đòi khoá shop, đòi SĐT).

### Ảnh có đánh số

![Ca A: bấm hết ca khó](prototype/screen-1-ca-kho.png)

![Ca A: nhân viên duyệt hoàn tiền](prototype/screen-2-nhan-vien-duyet.png)

![Ca C: doạ bóc phốt, đòi khoá shop](prototype/screen-3-ca-doa-khoa-shop.png)

| Số | Thể hiện gì | Nguyên tắc | Mức tự chủ |
|---|---|---|---|
| ① | Nhãn "Trợ lý AI · không tự quyết hoàn tiền hay từ chối" | G1 · Luật AI 134/2025 Điều 11 | (lời hứa ở §4) |
| ② | Hộp căn cứ: mô tả **lúc đặt** cạnh mô tả **hiện tại** (đã sửa 09/11), kèm lời người mua báo | G11 · PAIR Explainability + Trust | — |
| ③ | Thanh độ chắc ba mức (cao / thấp / chưa đủ căn cứ) kèm lý do bằng chữ ngay dưới | G2 | — |
| ④ | Hỏi đúng một câu có sẵn lựa chọn ("đo phần nào?", "đơn nào?") | G10 | — |
| ⑤ | Danh sách "còn thiếu đúng 1 thứ" | G1 (người mua: "không ai nói tôi còn thiếu cái gì") | — |
| ⑥ | Nhãn xanh "Đã tự làm: gửi mẫu xin bằng chứng cho shop, hạn 48 giờ"; bên người bán có đồng hồ đếm ngược | Hai bên cùng được nghe | `yeu_cau_bang_chung`: tự làm |
| ⑦ | "Đã chuyển nhân viên, kết quả trước 13/11 15:00, hạn trả hàng được giữ"; nhân viên có nút "Trả về trợ lý" | PAIR Errors + Graceful failure | `chuyen_truong_nhom`: tự làm, có hoàn tác |
| ⑧ | Hộp xác nhận: ai mất gì, "không đảo được", phải tích "đã xem bằng chứng của cả hai bên" | — | `soan_hoan_tien` (bước hoàn): cần duyệt |
| ⑨ | AI không có nút từ chối hay phạt; đòi khoá shop thì chuyển bộ phận chuyên trách | — | `tu_choi_khieu_nai`, `phat_shop`: cấm |
| ⑩ | "AI hiểu sai? Sai chỗ nào?" để người mua sửa hồ sơ | G9 · G15 | — |
| ⑪ | Nút "Gặp nhân viên" luôn có ở mọi bước | G8 | `chuyen_truong_nhom` |
| ⑫ | Hạn trả hàng và số ngày còn lại | Rubric track #8 | — |
| ⑬ | "Không hiện SĐT/địa chỉ"; đòi SĐT shop thì từ chối và chỉ sang Chat trong đơn | An toàn của track | — |

*(Khi TV2 chốt 2–3 nguyên tắc HAX/PAIR ở §3, giữ các số tương ứng làm số chính khi demo.)*

### Quyết định cách hiện

- **Độ chắc:** chọn **thanh ba mức, có chữ và lý do** (③). Đã nghĩ tới phần trăm ("72%"), nhưng loại vì người mua không biết 72% là nhiều hay ít, và con số tạo cảm giác chính xác giả. Lý do bằng chữ ("6 lít không ghi là nồi hay giỏ") giúp người dùng tự kiểm.
- **Bằng chứng:** chọn **đặt mô tả lúc đặt cạnh mô tả hiện tại**, tô nền bản lúc đặt (②). Đã nghĩ tới chỉ hiện mô tả lúc đặt, nhưng loại vì giấu lần sửa của shop thì người bán không hiểu vì sao bị xét theo bản cũ, và người mua không thấy vì sao AI không chắc.
- **Điều khiển mức tự chủ:** chọn **ba vai trên một màn, hồ sơ chung đặt ở giữa** người mua và người bán, mỗi hành động gắn dải màu theo mức (xanh tự làm, vàng cần duyệt, đỏ cấm). Bố cục này cho thấy sự trung lập: hai bên cùng nhìn một hồ sơ, và nút quyết định tiền chỉ nằm ở ô của nhân viên hoặc ở phía shop (bên mất tiền). Đã nghĩ tới một nút gạt đổi mức như mẫu của giảng viên, nhưng loại vì ca T4 khó ở chỗ **ai** bấm (shop hay nhân viên), không phải mức nào.

## §6. Tốt là gì (chặng 6 — nhẹ, bỏ được nếu thiếu giờ)

### 3 tiêu chí Pass/Fail chấm được
1. **Tiêu chí 1 - Tự động dừng và Escalate khi có khủng hoảng (Pass/Fail):**
   - *Định nghĩa:* Khi tin nhắn người dùng chứa từ khóa pháp lý (công an, kiện tụng, luật sư) hoặc đe dọa bóc phốt truyền thông, AI **100% phải dừng quy trình tự động trong đúng 1 lượt hội thoại**, hiển thị thông báo xoa dịu tiêu chuẩn và chuyển tiếp phiên chat sang nhân viên CSKH (kèm phiếu tóm tắt ngữ cảnh).
   - *Fail nếu:* AI tiếp tục trả lời máy móc, tự suy đoán đúng/sai hoặc hứa bồi hoàn tài chính bằng văn bản.
2. **Tiêu chí 2 - Tự động chuyển luồng theo danh mục hàng hóa (Pass/Fail):**
   - *Định nghĩa:* Đối với các đơn hàng thuộc danh mục hàng tươi sống / hư hỏng nhanh (`category == PERISHABLE_FOOD`), AI **100% không được tạo mã vận đơn hoặc chỉ dẫn người mua đóng gói gửi trả vật lý**, mà bắt buộc phải chuyển sang luồng *"Yêu cầu hoàn tiền không trả hàng (Refund Only)"* và gửi thông báo cho người bán phản hồi.
   - *Fail nếu:* AI gửi hướng dẫn đóng gói gửi hàng trả về cho shop qua đơn vị vận chuyển.
3. **Tiêu chí 3 - Kiểm tra & hướng dẫn bằng chứng cụ thể tại chỗ (Pass/Fail):**
   - *Định nghĩa:* Khi ảnh/video người dùng gửi lên không đạt tiêu chuẩn (mờ, thiếu tem mã vận đơn, video lỗi), AI phải phản hồi chỉ rõ đích danh tiêu chí thiếu (ví dụ: *"Ảnh chưa thấy rõ mã vận đơn"* hoặc *"Ảnh sản phẩm bị mờ góc rách"*) trong vòng **≤ 3 giây** kèm ảnh mẫu minh họa, trước khi mở cổng nộp khiếu nại chính thức.
   - *Fail nếu:* AI thông báo chung chung *"Bằng chứng không hợp lệ"* mà không giải thích, hoặc AI duyệt bừa khi chưa có bằng chứng hợp lệ.

### Đánh giá thiết kế trên 2 kịch bản đắt nhất
- **Trên kịch bản đắt nhất 1 (Bức xúc / Pháp lý / Truyền thông):** Thiết kế đạt **Tiêu chí 1**. Hệ thống chặn đứng nguy cơ phát ngôn sai lệch và cắt đứt khủng hoảng bằng cách bàn giao người thật xử lý kịp thời.
- **Trên kịch bản đắt nhất 2 (Hàng thực phẩm tươi sống hỏng):** Thiết kế đạt **Tiêu chí 2**. Chặn đứng thiệt hại tài chính phát sinh từ việc vận chuyển hàng thối rữa và tránh tranh chấp không đáng có giữa người mua và người bán.
- **Lỗi còn lại (Residual Risk):** Nếu người dùng cố tình lách từ khóa (ví dụ: dùng tiếng lóng mỉa mai, ẩn ý đe dọa mà bộ phân tích cảm xúc chưa nhận diện được), AI có thể xử lý như một ca khiếu nại thông thường thêm 1-2 lượt trước khi người dùng thực sự bấm nút "Gặp nhân viên hỗ trợ".

## §7. Demo

- Ca lạ nhận được ở demo: prototype đã làm gì, và nhóm có đổi ý gì sau ca đó không.
