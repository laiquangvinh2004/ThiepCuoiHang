# Kết nối form xác nhận với Google Sheets

Form trong `index.html` cần URL Web App của Google Apps Script để lưu lời chúc và xác nhận tham dự. Tạo một Google Sheet, mở **Extensions → Apps Script**, dán mã dưới đây và thay `YOUR_SPREADSHEET_ID` bằng ID trong URL của trang tính.

```javascript
const SPREADSHEET_ID = 'YOUR_SPREADSHEET_ID';
const SHEET_NAME = 'Xác nhận tham dự';

function doPost(e) {
  const values = (e && e.parameter) || {};
  if (values.website) return response_('OK'); // honeypot chống gửi tự động

  const name = clean_(values.name, 120);
  if (!name) return response_('Missing name');

  const sheet = SpreadsheetApp.openById(SPREADSHEET_ID)
    .getSheetByName(SHEET_NAME) || SpreadsheetApp.openById(SPREADSHEET_ID).insertSheet(SHEET_NAME);
  if (sheet.getLastRow() === 0) {
    sheet.appendRow(['Thời gian nhận', 'Tên khách', 'Lời chúc', 'Xác nhận tham dự', 'Đi cùng']);
  }
  sheet.appendRow([
    new Date(),
    name,
    clean_(values.message, 1000),
    clean_(values.attendance, 80),
    clean_(values.companions, 120)
  ]);
  return response_('OK');
}

function clean_(value, maxLength) {
  let text = String(value || '').trim().slice(0, maxLength);
  // Tránh nội dung do khách nhập bị Google Sheets hiểu là công thức.
  if (/^[=+\-@]/.test(text)) text = "'" + text;
  return text;
}

function response_(text) {
  return ContentService.createTextOutput(text)
    .setMimeType(ContentService.MimeType.TEXT);
}
```

## Triển khai

1. Trong Apps Script, chọn **Deploy → New deployment → Web app**.
2. Chọn **Execute as: Me** và quyền truy cập **Anyone** để khách được mời có thể gửi biểu mẫu.
3. Nhấn **Deploy**, chấp nhận quyền truy cập, rồi sao chép URL kết thúc bằng `/exec`.
4. Mở `index.html`, tìm `var sheetsEndpoint=''` trong script và dán URL vào giữa dấu nháy đơn.
5. Đưa `index.html`, `images.jpg` và `abc.mp3` lên cùng một hosting để khách sử dụng thiệp.

Form gửi các trường `name`, `message`, `attendance` và `companions`. Trình duyệt không cho đọc phản hồi từ Apps Script trong chế độ gửi chéo miền, vì vậy trang hiển thị xác nhận sau khi yêu cầu được gửi; hãy kiểm tra một dòng mới xuất hiện trong Sheet sau lần gửi đầu tiên.

## Ảnh và thông tin mừng cưới

Khi có QR thật, đặt hai ảnh QR cạnh `index.html` (ví dụ `qr-co-dau.png` và `qr-chu-re.png`). Trong `index.html`, điền `bank`, `account`, và `qr` tại `giftInfo.bride` / `giftInfo.groom`. `qr` là tên file ảnh tương ứng; hộp mừng cưới sẽ hiển thị ảnh đó và nút sao chép số tài khoản.
