## Phần 1: Báo cáo phân tích luồng dữ liệu (I/O Flow)

Từ lúc bạn nhấn `Ctrl + S` trên máy tính cục bộ cho đến khi tệp tin xuất hiện trạng thái đồng bộ thành công (tích xanh trên Cloud), luồng xử lý I/O diễn ra qua các bước kỹ thuật sau:
[Nhấn Ctrl+S] 
      │
      
[Ghi đè tệp tin trên SSD Local (I/O trực tiếp tốc độ cao)]
      │
      
[Phần mềm Client Sync quét sự kiện File System (File Change Event)]
      │
      ├─ [Kiểm tra quy tắc loại trừ (Ignore Rules / .gitignore hoặc cấu hình Sync)]
      │     └─Nằm trong node_modules? ── [HỦY BỎ (Bỏ qua, không đồng bộ)]
      │
      └─ [Nằm trong source code / docs] ── [TIẾP TỤC TIẾN TRÌNH]
            │
            
      [Đóng gói và mã hóa delta (chỉ truyền phần thay đổi)]
            │
            
      [Đẩy dữ liệu lên Cloud Storage qua HTTPS (Background Upload)]
            │
          
      [Nhận phản hồi ACK từ Cloud ── Hiển thị tích xanh (Synced)]


-   **Điểm cốt lõi:** Quá trình lưu file cục bộ diễn ra tức thì nhờ tốc độ của ổ cứng SSD (không bị nghẽn mạng). Tiến trình đồng bộ Cloud được đẩy ngầm (background process) và tuân thủ tuyệt đối bộ lọc loại trừ (Ignore Filters).
    

## Phần 2: Thiết kế tổ chức thư mục Cloud chuẩn cho Web Developer

Để quản lý dự án hiệu quả, tối ưu hóa tính năng **Files On-Demand** (chỉ tải file khi cần) và loại bỏ hoàn toàn bẫy node_modules, cấu trúc thư mục được phân định rõ ràng như sau:
 Cloud-Root (Thư mục gốc đồng bộ Cloud)
 └──  Projects
      └──  SmartFood-App (Thư mục dự án JavaScript)
           ├──  src/                  <-- (ĐỒNG BỘ) Source code chính (JS, HTML, CSS)
           ├──  public/               <-- (ĐỒNG BỘ) Tài nguyên tĩnh (Favicon, index.html)
           ├──  docs/                 <-- (ĐỒNG BỘ) Tài liệu kỹ thuật, sơ đồ, API specs
           ├──  package.json          <-- (ĐỒNG BỘ) Khai báo thư viện phụ thuộc
           ├──  package-lock.json     <-- (ĐỒNG BỘ) Khóa phiên bản thư viện
           ├──  .gitignore            <-- (ĐỒNG BỘ) Cấu hình loại trừ cho Git
           └──  node_modules/         <-- [LOẠI TRỪ TUYỆT ĐỐI KHỎI SYNC]



### Giải pháp kỹ thuật xử lý bẫy `node_modules`:

1.  **Tại sao phải loại trừ?** Thư mục `node_modules` chứa hàng chục ngàn file nhỏ. Nếu đồng bộ, phần mềm Cloud (Google Drive/OneDrive) sẽ bị treo (freeze) do giới hạn số lượng file xử lý đồng thời, đồng thời chiếm hàng GB dung lượng vô ích trên mây.
    
2.  **Cách khôi phục khi làm việc trên máy mới:**
    
    -   Khi tải dự án về máy mới qua cơ chế _Files On-Demand_ (chỉ hiện cấu trúc file nhẹ trên ổ cứng), bạn không tải node_modules.
        
