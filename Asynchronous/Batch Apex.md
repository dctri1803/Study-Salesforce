Tóm tắt bài học về **Batch Apex** và các điểm quan trọng cần ghi nhớ:  

---

### 🔹 **Batch Apex là gì?**  
Batch Apex được sử dụng để xử lý các công việc lớn (hàng nghìn hoặc hàng triệu bản ghi) mà các phương thức đồng bộ bình thường không thể thực hiện do giới hạn nền tảng. Batch Apex giúp xử lý dữ liệu bất đồng bộ theo từng lô (batch), giúp tối ưu hiệu suất và tránh vi phạm giới hạn của Salesforce.

### 🔹 **Ưu điểm của Batch Apex**  
✅ **Tận dụng giới hạn mới**: Mỗi lô xử lý đều có một tập giới hạn mới, giúp tránh lỗi quá giới hạn.  
✅ **Không rollback toàn bộ khi lỗi**: Nếu một lô bị lỗi, các lô khác vẫn tiếp tục chạy bình thường.  
✅ **Xử lý được dữ liệu lớn**: Có thể truy vấn và xử lý đến **50 triệu bản ghi**.

---

### 🔹 **Cấu trúc Batch Apex**  
Batch Apex phải triển khai **`Database.Batchable<T>`** và có 3 phương thức chính:

1. **`start(Database.BatchableContext bc)`**  
   - Xác định tập dữ liệu cần xử lý.  
   - Trả về **QueryLocator** (nếu dùng SOQL) hoặc **Iterable** (nếu duyệt qua danh sách tùy chỉnh).  
   - Dùng **QueryLocator** có thể truy vấn tối đa **50 triệu bản ghi**.

2. **`execute(Database.BatchableContext bc, List<T> scope)`**  
   - Xử lý từng lô dữ liệu, mặc định mỗi lô có **200 bản ghi**.  
   - Không đảm bảo thứ tự xử lý của các lô.  
   - Có thể gọi DML (`insert`, `update`, `delete`...).

3. **`finish(Database.BatchableContext bc)`**  
   - Thực hiện các tác vụ hậu xử lý như gửi email thông báo, ghi log.  

📌 **Ví dụ Batch Apex** (Cập nhật địa chỉ liên hệ theo địa chỉ công ty):  

```apex
public class UpdateContactAddresses implements Database.Batchable<sObject>, Database.Stateful {
    public Integer recordsProcessed = 0;
    
    public Database.QueryLocator start(Database.BatchableContext bc) {
        return Database.getQueryLocator('SELECT Id, BillingStreet, BillingCity, BillingState, BillingPostalCode, ' +
                                        '(SELECT Id, MailingStreet, MailingCity, MailingState, MailingPostalCode FROM Contacts) FROM Account ' +
                                        'WHERE BillingCountry = \'USA\'');
    }

    public void execute(Database.BatchableContext bc, List<Account> scope) {
        List<Contact> contacts = new List<Contact>();
        for (Account acc : scope) {
            for (Contact con : acc.Contacts) {
                con.MailingStreet = acc.BillingStreet;
                con.MailingCity = acc.BillingCity;
                con.MailingState = acc.BillingState;
                con.MailingPostalCode = acc.BillingPostalCode;
                contacts.add(con);
                recordsProcessed++;
            }
        }
        update contacts;
    }

    public void finish(Database.BatchableContext bc) {
        System.debug(recordsProcessed + ' records processed.');
    }
}
```

---

### 🔹 **Gọi Batch Apex**  
```apex
UpdateContactAddresses batchJob = new UpdateContactAddresses();
Id batchId = Database.executeBatch(batchJob, 100); // 100 là batch size
```
📌 **Lưu ý**: Batch Apex luôn tạo một bản ghi `AsyncApexJob` để theo dõi tiến trình.

---

### 🔹 **Giữ trạng thái giữa các batch**  
- Mặc định, Batch Apex không giữ trạng thái giữa các lần thực thi.  
- Nếu cần lưu trữ giá trị giữa các batch, sử dụng **`Database.Stateful`**.  
- Chỉ **biến instance (biến thành viên)** được bảo toàn, biến cục bộ sẽ không giữ giá trị.

---

### 🔹 **Kiểm thử Batch Apex**  
Do Batch Apex chạy bất đồng bộ, cần dùng **Test.startTest()** và **Test.stopTest()** để kiểm thử:

```apex
@isTest
private class UpdateContactAddressesTest {
    @testSetup
    static void setup() {
        List<Account> accounts = new List<Account>();
        List<Contact> contacts = new List<Contact>();
        
        for (Integer i = 0; i < 10; i++) {
            accounts.add(new Account(Name='Account ' + i, BillingCity='New York', BillingCountry='USA'));
        }
        insert accounts;
        
        for (Account acc : [SELECT Id FROM Account]) {
            contacts.add(new Contact(FirstName='Test', LastName='User', AccountId=acc.Id));
        }
        insert contacts;
    }

    @isTest
    static void testBatch() {
        Test.startTest();
        UpdateContactAddresses batchJob = new UpdateContactAddresses();
        Database.executeBatch(batchJob, 10);
        Test.stopTest();

        System.assertEquals(10, [SELECT COUNT() FROM Contact WHERE MailingCity = 'New York']);
    }
}
```

📌 **Lưu ý**:
- Test chỉ chạy với **1 batch** (tức là số bản ghi test không nên vượt quá batch size).
- **`Test.stopTest()`** đảm bảo batch chạy ngay lập tức trong test.

---

### 🔹 **Best Practices (Lưu ý quan trọng)**  
✅ Chỉ dùng Batch Apex khi có **hơn 1 batch** (nếu ít bản ghi, nên dùng **Queueable Apex**).  
✅ Tối ưu **SOQL query** để thu thập bản ghi nhanh nhất.  
✅ Tránh tạo quá nhiều yêu cầu bất đồng bộ để tránh làm chậm hệ thống.  
✅ **Hạn chế gọi Batch Apex từ Trigger**, vì dễ bị vượt giới hạn số batch chạy cùng lúc.  

---

📌 **Tóm lại:**  
- **Batch Apex** giúp xử lý dữ liệu lớn hiệu quả trong Salesforce.  
- Cần **start()**, **execute()**, **finish()** để định nghĩa batch.  
- Dùng **Database.Stateful** nếu cần giữ trạng thái giữa các batch.  
- Kiểm thử batch bằng **Test.startTest() / Test.stopTest()**.  
- Tránh gọi batch từ trigger để không vượt giới hạn Salesforce.  

Bạn có cần mình giải thích thêm hoặc code mẫu nào khác không? 🚀