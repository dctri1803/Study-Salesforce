### **Bắt đầu với Apex Triggers **

#### **Mục tiêu học tập**
Sau khi hoàn thành, bạn sẽ có thể:
- Viết một trigger cho đối tượng Salesforce.
- Sử dụng các biến ngữ cảnh (context variables) trong trigger.
- Gọi phương thức từ một lớp trong trigger.
- Dùng phương thức `addError()` để hạn chế việc lưu dữ liệu.

---

### **Viết Apex Trigger**
Apex Trigger giúp thực thi hành động tùy chỉnh trước hoặc sau khi các sự kiện (insert, update, delete) xảy ra với dữ liệu. Bạn có thể dùng triggers để:
- Cập nhật dữ liệu liên quan.
- Hạn chế một số thao tác nhất định.
- Chạy truy vấn SOQL, DML, hoặc gọi hàm Apex tùy chỉnh.

#### **Cú pháp Trigger**
```apex
trigger TriggerName on ObjectName (trigger_events) {
   code_block
}
```
Ví dụ trigger đơn giản: 
```apex
trigger HelloWorldTrigger on Account (before insert) {
    System.debug('Hello World!');
}
```

#### **Các loại Trigger**
- **Before Trigger**: Chạy trước khi dữ liệu được lưu, dùng để kiểm tra hoặc cập nhật dữ liệu.
- **After Trigger**: Chạy sau khi dữ liệu được lưu, dùng để cập nhật dữ liệu liên quan hoặc truy vấn thông tin hệ thống.

---

### **Sử dụng Biến Ngữ Cảnh (Context Variables)**
- `Trigger.new`: Danh sách bản ghi mới trong **insert** hoặc **update**.
- `Trigger.old`: Danh sách bản ghi cũ trước khi **update** hoặc **delete**.
- `Trigger.isInsert`, `Trigger.isUpdate`, `Trigger.isDelete`: Xác định trigger được gọi bởi hành động nào.
- `Trigger.isBefore`, `Trigger.isAfter`: Xác định trigger chạy trước hay sau khi lưu dữ liệu.

Ví dụ cập nhật `Description` của tài khoản:
```apex
trigger HelloWorldTrigger on Account (before insert) {
    for(Account a : Trigger.new) {
        a.Description = 'New description';
    }
}
```

---

### **Gọi Phương Thức từ Lớp Khác**
Trigger có thể gọi các phương thức từ lớp khác để tái sử dụng code.

Ví dụ gửi email khi tạo Contact mới:
```apex
trigger ExampleTrigger on Contact (after insert) {
    if (Trigger.isInsert) {
        Integer recordCount = Trigger.new.size();
        EmailManager.sendMail('your@email.com', 'New Contact Created', recordCount + ' contact(s) were inserted.');
    }
}
```
Lớp `EmailManager`:
```apex
public class EmailManager {
    public static void sendMail(String address, String subject, String body) {
        Messaging.SingleEmailMessage mail = new Messaging.SingleEmailMessage();
        mail.setToAddresses(new String[] {address});
        mail.setSubject(subject);
        mail.setPlainTextBody(body);
        Messaging.sendEmail(new Messaging.SingleEmailMessage[] { mail });
    }
}
```

---

### **Thêm Bản Ghi Liên Quan**
Trigger có thể tạo các bản ghi liên quan, ví dụ tự động tạo cơ hội bán hàng (Opportunity) khi có tài khoản mới:
```apex
trigger AddRelatedRecord on Account(after insert, after update) {
    List<Opportunity> oppList = new List<Opportunity>();
    Map<Id,Account> acctsWithOpps = new Map<Id,Account>(
        [SELECT Id,(SELECT Id FROM Opportunities) FROM Account WHERE Id IN :Trigger.new]);
    
    for(Account a : Trigger.new) {
        if (acctsWithOpps.get(a.Id).Opportunities.isEmpty()) {
            oppList.add(new Opportunity(
                Name = a.Name + ' Opportunity',
                StageName = 'Prospecting',
                CloseDate = System.today().addMonths(1),
                AccountId = a.Id));
        }
    }
    if (!oppList.isEmpty()) {
        insert oppList;
    }
}
```

---

### **Sử dụng Trigger Exceptions**
Dùng phương thức `addError()` để ngăn chặn một số hành động, ví dụ ngăn xóa tài khoản có cơ hội bán hàng liên quan:
```apex
trigger AccountDeletion on Account (before delete) {
    for (Account a : [SELECT Id FROM Account
                     WHERE Id IN (SELECT AccountId FROM Opportunity) AND
                     Id IN :Trigger.old]) {
        Trigger.oldMap.get(a.Id).addError('Cannot delete account with related opportunities.');
    }
}
```

---

### **Trigger và Callouts**
Trigger có thể gọi API bên ngoài, nhưng phải chạy bất đồng bộ bằng cách sử dụng `@future(callout=true)`. Ví dụ:
```apex
public class CalloutClass {
    @future(callout=true)
    public static void makeCallout() {
        HttpRequest request = new HttpRequest();
        request.setEndpoint('http://yourHost/yourService');
        request.setMethod('GET');
        HttpResponse response = new HTTP().send(request);
    }
}
```
Gọi callout từ trigger:
```apex
trigger CalloutTrigger on Account (before insert, before update) {
    CalloutClass.makeCallout();
}
```

---

### **Tóm tắt**
- **Triggers** giúp tự động hóa xử lý dữ liệu trong Salesforce.
- **Có hai loại triggers**: `before` (chỉnh sửa dữ liệu trước khi lưu) và `after` (chỉnh sửa dữ liệu sau khi lưu).
- **Biến ngữ cảnh** (`Trigger.new`, `Trigger.old`,...) giúp truy cập dữ liệu trong trigger.
- **Trigger có thể gọi phương thức từ lớp khác** để tái sử dụng code.
- **Sử dụng `addError()` để ngăn các hành động không mong muốn**.
- **Trigger có thể gọi API bên ngoài** nhưng phải chạy bất đồng bộ.

Đây là các kiến thức quan trọng giúp bạn làm việc hiệu quả với Apex Trigger trong Salesforce! 🚀