### **Tổng Quan Về Xử Lý Bất Đồng Bộ (Asynchronous Processing Basics)**

#### **Mục Tiêu Học Tập**
Sau khi hoàn thành bài học này, bạn sẽ có thể:  
✅ Giải thích sự khác biệt giữa xử lý đồng bộ và bất đồng bộ.  
✅ Chọn loại Apex bất đồng bộ phù hợp với từng tình huống cụ thể.  

---

## **1. Xử Lý Bất Đồng Bộ Là Gì?**
- **Xử lý bất đồng bộ** giúp thực thi các tác vụ "ở chế độ nền" mà không bắt người dùng phải chờ đợi.  
- Ví dụ thực tế: Khi bạn có nhiều việc cần làm, bạn có thể:  
  - Đợi sửa xe xong rồi mới làm việc khác (**đồng bộ** – mất thời gian chờ).  
  - Để xe lại garage và đi làm các việc khác trong khi xe đang sửa (**bất đồng bộ** – tối ưu thời gian).  

### **Lợi Ích Của Xử Lý Bất Đồng Bộ**
✅ **Tăng hiệu suất người dùng**: Không cần chờ các tác vụ dài chạy xong.  
✅ **Khả năng mở rộng (Scalability)**: Hệ thống có thể xử lý nhiều công việc song song.  
✅ **Giới hạn cao hơn**: Apex bất đồng bộ có các giới hạn cao hơn so với đồng bộ, giúp xử lý khối lượng công việc lớn hơn.  

---

## **2. Các Loại Apex Bất Đồng Bộ**
| **Loại**           | **Mô tả** | **Ứng Dụng** |
|--------------------|----------|--------------|
| **Future Methods** | Chạy trên một luồng riêng, bắt đầu khi có tài nguyên. | Callout đến hệ thống bên ngoài. |
| **Batch Apex** | Xử lý lượng lớn dữ liệu, vượt qua giới hạn thông thường. | Làm sạch hoặc lưu trữ dữ liệu. |
| **Queueable Apex** | Giống Future Methods nhưng hỗ trợ liên kết nhiều job và dùng kiểu dữ liệu phức tạp hơn. | Xử lý nhiều bước tuần tự với dịch vụ Web. |
| **Scheduled Apex** | Chạy theo lịch trình định sẵn. | Tác vụ hàng ngày, hàng tuần. |

🔥 **Lưu ý**: Các loại này có thể kết hợp với nhau. Ví dụ: Một **Scheduled Apex** có thể kích hoạt một **Batch Apex** để xử lý dữ liệu định kỳ.  

---

## **3. Cách Hoạt Động Của Xử Lý Bất Đồng Bộ**
### **Hệ Thống Hàng Đợi (Queue-Based System)**
1️⃣ **Enqueue (Xếp hàng đợi)**: Yêu cầu được đưa vào hàng đợi.  
2️⃣ **Persistence (Lưu trữ tạm thời)**: Lưu yêu cầu vào hệ thống để tránh mất dữ liệu.  
3️⃣ **Dequeue (Xử lý yêu cầu)**: Lấy yêu cầu từ hàng đợi và thực thi. Nếu lỗi, hệ thống sẽ đảm bảo không mất dữ liệu.  

📌 **Quản lý tài nguyên**:  
- Hệ thống Salesforce đảm bảo **công bằng** giữa các tổ chức.  
- Nếu tài nguyên bị quá tải, xử lý bất đồng bộ sẽ **tạm hoãn** cho đến khi có đủ tài nguyên.  

---

### **Tóm Lại**
- Xử lý bất đồng bộ giúp tăng hiệu suất, mở rộng hệ thống và tận dụng giới hạn cao hơn.  
- Có 4 loại Apex bất đồng bộ: **Future Methods, Batch Apex, Queueable Apex, Scheduled Apex**.  
- Salesforce sử dụng hàng đợi để quản lý và đảm bảo tính công bằng khi xử lý.  

🔥 **Khi nào nên dùng Apex bất đồng bộ?**  
✅ Khi gọi hệ thống bên ngoài (callout).  
✅ Khi cần xử lý dữ liệu lớn (Batch Apex).  
✅ Khi cần lập lịch tự động (Scheduled Apex).  
✅ Khi cần liên kết nhiều bước xử lý (Queueable Apex).  