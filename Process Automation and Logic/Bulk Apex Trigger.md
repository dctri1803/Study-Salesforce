##**Bulk Apex Triggers**:

### **1. Bulk Trigger Design Patterns**
- **Triggers trong Apex được tối ưu hóa để xử lý dữ liệu theo batch.**  
- Áp dụng **bulk design pattern** giúp tối ưu hiệu suất, giảm tải tài nguyên và tránh vượt quá giới hạn governor limits.  
- Mọi trigger nên luôn xử lý **một tập hợp bản ghi (collection of sObjects)** thay vì từng bản ghi riêng lẻ.

---

### **2. Xử lý Tập Hợp Bản Ghi (Operating on Record Sets)**
- Không nên giả định trigger chỉ chạy cho một bản ghi duy nhất.  
- **Ví dụ sai:** Chỉ xử lý một bản ghi duy nhất từ `Trigger.new[0]`.  
- **Ví dụ đúng:** Dùng `for` loop để duyệt qua tất cả bản ghi trong `Trigger.new`.

```apex
trigger MyTriggerBulk on Account(before insert) {
    for(Account a : Trigger.new) {
        a.Description = 'New description';
    }
}
```

---

### **3. Bulk SOQL (Truy vấn dữ liệu hiệu quả)**
- **Tránh SOQL trong vòng lặp** vì có thể gây lỗi vượt quá giới hạn 100 SOQL/query.  
- **Sai:** Truy vấn từng record riêng lẻ trong vòng lặp.  
- **Đúng:** Dùng `IN :Trigger.new` để lấy dữ liệu một lần duy nhất.

```apex
List<Account> acctsWithOpps = [SELECT Id, (SELECT Id FROM Opportunities) 
                               FROM Account WHERE Id IN :Trigger.new];

for(Account a : acctsWithOpps) {
    List<Opportunity> relatedOpps = a.Opportunities;
}
```

---

### **4. Bulk DML (Thao tác dữ liệu hiệu quả)**
- **Tránh gọi DML (insert/update/delete) trong vòng lặp** vì giới hạn tối đa 150 DML.  
- **Sai:** Gọi `update` từng record một.  
- **Đúng:** Gom tất cả bản ghi cần cập nhật vào một danh sách rồi `update` một lần.

```apex
List<Opportunity> oppsToUpdate = new List<Opportunity>();

for(Opportunity opp : relatedOpps) {
    if ((opp.Probability >= 50) && (opp.Probability < 100)) {
        opp.Description = 'New description';
        oppsToUpdate.add(opp);
    }
}

update oppsToUpdate; // Chỉ gọi update một lần
```

---

### **5. Ứng dụng thực tế – Trigger tạo Opportunity tự động**
**Yêu cầu:**  
- Khi tạo mới hoặc cập nhật **Account**, kiểm tra xem **đã có Opportunity chưa**.  
- Nếu chưa có, tạo một **Opportunity mặc định**.

**Giải pháp:**  
- Dùng `switch on Trigger.operationType` để phân biệt `insert` và `update`.
- Dùng `NOT IN` trong SOQL để lọc các Account **chưa có Opportunity**.
- Chỉ thực hiện một lần `insert` cho tất cả Opportunity cần tạo.

```apex
trigger AddRelatedRecord on Account(after insert, after update) {
    List<Opportunity> oppList = new List<Opportunity>();
    List<Account> toProcess = null;

    switch on Trigger.operationType {
        when AFTER_INSERT {
            toProcess = Trigger.New;
        }
        when AFTER_UPDATE {
            toProcess = [SELECT Id, Name FROM Account
                         WHERE Id IN :Trigger.New 
                         AND Id NOT IN (SELECT AccountId FROM Opportunity 
                                        WHERE AccountId IN :Trigger.New)];
        }
    }

    for (Account a : toProcess) {
        oppList.add(new Opportunity(Name=a.Name + ' Opportunity',
                                    StageName='Prospecting',
                                    CloseDate=System.today().addMonths(1),
                                    AccountId=a.Id));
    }

    if (!oppList.isEmpty()) {
        insert oppList;
    }
}
```

---

### **Tóm tắt chính**
✅ **Luôn xử lý tập hợp bản ghi trong trigger**  
✅ **Dùng truy vấn SOQL một lần thay vì lặp qua từng record**  
✅ **Gom các thao tác DML thành một batch thay vì xử lý từng bản ghi riêng lẻ**  
✅ **Áp dụng Bulk Trigger Design Patterns để đảm bảo trigger hoạt động hiệu quả**  

Bạn có muốn mình tóm gọn hơn nữa không? 🚀