### 📌 **Những điều cần nhớ về Queueable Apex**  

#### 🔹 **Queueable Apex là gì?**  
Queueable Apex là một phiên bản nâng cấp của **future methods**, kết hợp giữa sự đơn giản của future với sức mạnh của **Batch Apex**. Nó giúp xử lý các tác vụ bất đồng bộ một cách linh hoạt hơn.  

#### 🔹 **Lợi ích của Queueable Apex**  
✅ **Hỗ trợ kiểu dữ liệu không nguyên thủy**: Có thể truyền **sObject**, danh sách sObject, hoặc các kiểu dữ liệu tùy chỉnh.  
✅ **Dễ dàng theo dõi**: Khi thực thi `System.enqueueJob()`, sẽ nhận được **AsyncApexJob ID** để theo dõi trạng thái công việc.  
✅ **Chaining (xâu chuỗi công việc)**: Có thể gọi một job khác sau khi job hiện tại kết thúc.  

---

### 🔹 **So sánh Queueable Apex vs Future Methods**  
| **Tiêu chí**      | **Queueable Apex** | **Future Methods** |
|------------------|-----------------|----------------|
| Truyền kiểu dữ liệu | Hỗ trợ sObject, custom objects | Chỉ hỗ trợ kiểu nguyên thủy |
| Theo dõi tiến trình | Có thể theo dõi bằng `AsyncApexJob` | Không theo dõi được |
| Chaining Jobs | Có thể xâu chuỗi job | Không hỗ trợ |
| Dễ bảo trì | Dễ dàng mở rộng | Khó bảo trì hơn |

✅ **Lời khuyên**: Nếu không có lý do đặc biệt, **nên dùng Queueable thay vì Future** do tính linh hoạt và khả năng mở rộng tốt hơn.  

---

### 🔹 **Cú pháp Queueable Apex**
```apex
public class SomeClass implements Queueable {
    public void execute(QueueableContext context) {
        // Xử lý logic tại đây
    }
}
```

Ví dụ thực tế: **Cập nhật ParentId cho các Account**
```apex
public class UpdateParentAccount implements Queueable {
    private List<Account> accounts;
    private ID parent;
    
    public UpdateParentAccount(List<Account> records, ID id) {
        this.accounts = records;
        this.parent = id;
    }
    
    public void execute(QueueableContext context) {
        for (Account account : accounts) {
            account.ParentId = parent;
        }
        update accounts;
    }
}
```
**Gọi Queueable Job:**  
```apex
List<Account> accounts = [SELECT Id FROM Account WHERE BillingState = 'NY'];
Id parentId = [SELECT Id FROM Account WHERE Name = 'ACME Corp' LIMIT 1].Id;

UpdateParentAccount updateJob = new UpdateParentAccount(accounts, parentId);
ID jobID = System.enqueueJob(updateJob);
```

---

### 🔹 **Chaining Jobs (Xâu chuỗi công việc)**  
- Có thể gọi một job khác bên trong phương thức `execute()`, giúp thực hiện công việc tuần tự.  
- **Chỉ được phép gọi 1 job từ 1 job cha** (không thể gọi nhiều job con từ 1 job cha).  
- **Không giới hạn độ sâu của chuỗi**, nhưng trong Developer Edition và Trial Orgs, tối đa chỉ được xâu chuỗi 5 job.  

Ví dụ: **Gọi job tiếp theo từ trong execute()**
```apex
public class FirstJob implements Queueable {
    public void execute(QueueableContext context) {
        // Xử lý logic
        System.enqueueJob(new SecondJob());
    }
}
```
❗ **Lưu ý:** Trong **Apex Test**, không thể chain jobs. Cần kiểm tra bằng `Test.isRunningTest()` trước khi gọi `System.enqueueJob()`.  

---

### 🔹 **Testing Queueable Apex**  
- Sử dụng `Test.startTest()` và `Test.stopTest()` để đảm bảo job chạy ngay trong test.  
- Không thể kiểm tra job chaining trong test.  

Ví dụ kiểm thử:
```apex
@isTest
public class UpdateParentAccountTest {
    @testSetup
    static void setup() {
        List<Account> accounts = new List<Account>();
        accounts.add(new Account(Name='Parent'));
        
        for (Integer i = 0; i < 100; i++) {
            accounts.add(new Account(Name='Test Account'+i));
        }
        insert accounts;
    }

    @isTest
    static void testQueueable() {
        Id parentId = [SELECT Id FROM Account WHERE Name = 'Parent' LIMIT 1].Id;
        List<Account> accounts = [SELECT Id FROM Account WHERE Name LIKE 'Test Account%'];

        UpdateParentAccount updater = new UpdateParentAccount(accounts, parentId);

        Test.startTest();
        System.enqueueJob(updater);
        Test.stopTest();

        System.assertEquals(100, [SELECT COUNT() FROM Account WHERE ParentId = :parentId]);
    }
}
```

---

### 🔹 **Những điều cần nhớ về Queueable Apex**  
✅ Mỗi lần thực thi `System.enqueueJob()` sẽ tính vào giới hạn chung của **asynchronous Apex executions**.  
✅ **Tối đa 50 job** có thể được thêm vào queue trong một transaction.  
✅ **Không thể gọi nhiều job con** từ cùng một job cha.  
✅ **Giới hạn xâu chuỗi**: Developer Edition và Trial Orgs chỉ có thể xâu chuỗi tối đa **5 jobs**.  
✅ **Không dùng Queueable để cập nhật dữ liệu từ Trigger** nếu có nguy cơ gây ra lỗi governor limit.  

---

📌 **Tóm lại:**  
- **Queueable Apex** mạnh hơn **Future Methods** do hỗ trợ kiểu dữ liệu phức tạp, theo dõi tiến trình và hỗ trợ chaining.  
- Dùng **System.enqueueJob()** để đưa job vào hàng đợi.  
- **Chaining job** giúp chạy các công việc liên tiếp nhưng chỉ được phép gọi **một job con** từ một job cha.  
- **Kiểm thử bằng Test.startTest() / Test.stopTest()** để đảm bảo chạy đồng bộ trong test.  