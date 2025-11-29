---
title: "Từ Đánh Giá Offline Đến Kết Quả Thực Tế Của Mô Hình Dự Đoán pCTR"
date: 2025-11-28 11:00:00 +0700
categories: [Work]
tags: [modelling, ctr-prediction]
---

Vấn đề cốt lõi và nan giải nhất khi xây dựng mô hình dự đoán Click-Through-Rate (pCTR) chính là làm sao để kết quả đánh giá nội bộ (offline evaluation) có thể tương đồng và sát sườn nhất với kết quả thực tế (business metrics) như eCPM, CTR, CVR. Nếu tồn tại một sự khác biệt lớn giữa hai kết quả này, việc đánh giá và định hướng phát triển mô hình sẽ trở nên vô cùng khó khăn, giống như ôn thi mà đề thi thử không phản ánh đúng cấu trúc đề thi thật.

Qua quá trình triển khai thực tế, tôi đã đúc kết được 3 kinh nghiệm quan trọng dưới đây:

### 1. Định nghĩa lại "Thế nào là một Mô hình Tốt"

Trong bối cảnh của một hệ thống quảng cáo, một mô hình có khả năng triển khai được (serve-able) không chỉ cần chỉ số AUC (khả năng xếp hạng) cao, mà còn phải có độ chính xác xác suất (Calibration) tốt.

**Thử thách Bias - Variance:** Thường thấy, một số mô hình đạt chỉ số AUC rất cao, cho thấy khả năng ranking quảng cáo rất tốt. Tuy nhiên, đổi lại, độ chệch (Bias) và Calibration (sự sai số trong việc dự đoán xác suất) lại kém. Khi tham gia vào quy trình đấu thầu (bidding), Calibration đôi khi còn quan trọng hơn cả AUC.

_Ví dụ: Một quảng cáo A có CTR thực tế là 0.3%. Nó sẽ tự điều chỉnh bid để cạnh tranh với các quảng cáo khác. Nếu mô hình dự đoán quảng cáo A có pCTR quá thấp (chẳng hạn 0.1%), hoặc dự đoán các quảng cáo kém chất lượng khác có pCTR quá cao, thì phần bid "cao cấp" mà quảng cáo A đặt thêm sẽ trở nên vô nghĩa. Hệ thống sẽ vô tình phân phối những quảng cáo giá rẻ nhưng có pCTR được dự đoán cao hơn thực tế, dẫn đến việc eCPM (Doanh thu trên 1000 lượt hiển thị) tổng thể của hệ thống bị suy giảm nghiêm trọng._

Vì vậy, việc xác định rõ ràng "thế nào là một mô hình serve-able và có giá trị kinh doanh" là điều tối quan trọng. Nó giúp chúng ta nhận ra những đánh đổi cần thiết (trade-offs) và có những kỳ vọng rõ ràng hơn khi triển khai online, **mà thường là AUC và calibration**.

### 2. Rào cản Kỹ thuật trong Triển khai Mô hình

Một mô hình đạt kết quả offline tốt không đảm bảo 100% nó sẽ hoạt động hiệu quả khi serve thực tế. Quá trình từ phát triển đến triển khai đòi hỏi rất nhiều nỗ lực và sự tuân thủ các giới hạn nghiêm ngặt từ đội ngũ kỹ thuật (backend).

**Giới hạn về Tài nguyên:**

**_Độ trễ (Latency):_** Một mô hình dù tốt đến mấy nhưng nếu có độ trễ quá cao sẽ không thể sử dụng được trong hệ thống real-time.

**_Bộ nhớ (Memory):_** Model không thể quá lớn, và việc thực hiện dự đoán (inference) thường chỉ giới hạn cho những người dùng hoạt động (active users) trong khung thời gian 24-48 giờ. Điều này kéo theo vô số logic và luật kinh doanh phức tạp, có khả năng làm giảm chất lượng tổng thể mà ta kỳ vọng.

### 3. Mô hình Tốt Vẫn Cần Excellent Execution

Trong quá trình tích hợp mô hình vào hệ thống, vô số lỗi kỹ thuật hoặc bugs có thể âm thầm xuất hiện mà ta không hề hay biết, dẫn đến chất lượng mô hình bị giảm sút dù kết quả offline và benchmark latency rất hoàn hảo.

Để giải quyết, ta cần kiểm tra kỹ lưỡng:

**Xử lý Dữ liệu:** Liệu dữ liệu đầu vào (input data) có được xử lý (transform) chính xác như lúc huấn luyện (train) hay không? Dữ liệu giữa lúc train và lúc inference có bị lệch pha (data mismatch) không?

**Liệu model ta đang đánh giá là model đó hay model khác?:** Hiệu suất kém mà ta quan sát được là của mô hình mới triển khai hay đến từ một mô hình dự phòng (fallback model) nào đó?

Đây là phần mang lại nhiều thử thách nhất và chỉ có thể được giải quyết thông qua:

- Thiết lập các chỉ số theo dõi (monitoring metrics) chất lượng và toàn diện.

- Tinh thần đồng đội, sự minh bạch và thẳng thắn giữa các bên để cùng nhau gỡ lỗi (debug) và giải thích vấn đề.

**Tóm lại,** sự khác biệt giữa chất lượng, behavior mode trong offline evaluation và trong môi trường serving thực thế không phải là thất bại mà là những cơ hội để ta có thể phát triển thêm. Nó buộc chúng ta phải đặt ra nhiều giả thuyết, giải quyết những sai lầm trong đánh giá offline và tìm ra những điểm không khớp (mismatch) trong quy trình triển khai mô hình.
