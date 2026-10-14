# Báo cáo phân tích và lựa chọn toán tử chuyển hướng Output trong PowerShell

## Phần 1: So sánh toán tử ghi đè (`>`) và ghi nối tiếp (`>>`)

| Tiêu chí so sánh | Toán tử Ghi đè (`>`) | Toán tử Ghi nối tiếp (`>>`) |
| :--- | :--- | :--- |
| **Cơ chế hoạt động** | Tương đương lệnh `Out-File`. Nếu tệp chưa tồn tại, tệp mới sẽ được tạo. Nếu tệp đã tồn tại, **toàn bộ nội dung cũ sẽ bị xóa sạch** và thay thế hoàn toàn bằng dữ liệu mới. | Tương đương lệnh `Out-File -Append`. Dữ liệu mới sẽ được **tự động nối tiếp vào cuối tệp** hiện có mà không làm ảnh hưởng hay mất mát dữ liệu cũ. |
| **Rủi ro mất dữ liệu** | **Rất cao.** Nếu vô tình chạy lại kịch bản hoặc gõ nhầm vào cuối tháng (khi báo cáo đã tích lũy dữ liệu của 30 ngày trước), toàn bộ lịch sử sẽ **bốc hơi vĩnh viễn**. | Thấp. Dữ liệu cũ được bảo toàn an toàn, chỉ có thêm dòng log mới được ghi bổ sung bên dưới. |
| **Ngữ cảnh sử dụng thực tế** | Phù hợp cho việc **tạo mới/ghi đè báo cáo ca sáng** (khi cần làm sạch báo cáo cũ và chỉ muốn lấy dữ liệu mới nhất của ngày hôm đó). | Phù hợp cho hệ thống **theo dõi lỗi 24/7**, ghi nhật ký liên tục hoặc tích lũy dữ liệu lịch sử theo thời gian. |
| **Encoding file đầu ra** | Mặc định sử dụng bảng mã **UTF-16** (do cơ chế gốc của `Out-File`), có thể gây lỗi hiển thị ký tự (ký tự lạ, lỗi tiếng Việt) nếu mở bằng các công cụ đọc log chỉ hỗ trợ UTF-8/ANSI. | Giống với `>`, mặc định ghi theo **UTF-16**, đòi hỏi kỹ sư cần lưu ý khi cấu hình hoặc kiểm tra hiển thị. |

---

## Phần 2: Lựa chọn giải pháp tối ưu cho hệ thống log 24/7

### 1. Giải pháp đề xuất
Đối với hệ thống ghi nhận sự cố lỗi phát sinh liên tục trong ngày, phương án tối ưu nhất là sử dụng **ghi nối tiếp (`>>`)** kết hợp kiểm soát bảng mã, hoặc chuyên nghiệp hơn là dùng lệnh **`Add-Content`**.

### 2. Lập luận lý do
- **Đảm bảo tính bền vững của dữ liệu:** Hệ thống chạy liên tục cần tích lũy dữ liệu theo thời gian thực mà không bị ghi đè làm mất mát các lỗi đã ghi nhận từ trước.
- **Kiểm soát Encoding chuẩn UTF-8:** Mặc dù `>>` (tương đương `Out-File -Append`) rất nhanh gọn, nhưng nó dùng mặc định UTF-16 có thể gây phiền toái khi tích hợp với các công cụ đọc log bên ngoài. Để chuyên nghiệp tuyệt đối, ta sử dụng **`Add-Content`** với tham số ép kiểu mã hóa rõ ràng:

```powershell
Get-Content server.log | Select-String "ERROR" | Add-Content -Path daily_report.txt -Encoding utf8
