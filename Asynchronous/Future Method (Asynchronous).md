### **Tóm tắt bài học & Ghi chú quan trọng về Future Methods trong Apex**  

---

## **1. Khi nào sử dụng Future Methods?**  
Future Methods được sử dụng để chạy các tiến trình trong một luồng riêng biệt, giúp:  
✅ **Thực hiện callout đến dịch vụ bên ngoài** (nhất là trong trigger hoặc sau DML).  
✅ **Xử lý các tác vụ nặng** mà không làm gián đoạn luồng chính của hệ thống.  
✅ **Tránh lỗi Mixed DML** khi thao tác trên các loại sObject không thể dùng chung trong DML.  

---

## **2. Cú pháp Future Method**  
- **Phải là phương thức `static` và chỉ có thể trả về kiểu `void`.**  
- **Chỉ nhận tham số là kiểu dữ liệu nguyên thủy (primitive) hoặc danh sách của chúng.**  
- **Không thể truyền trực tiếp các đối tượng (sObject) vào phương thức future.**  

Ví dụ:  
```apex
public class SomeClass {
  @future
  public static void someFutureMethod(List<Id> recordIds) {
    List<Account> accounts = [SELECT Id, Name FROM Account WHERE Id IN :recordIds];
    // Xử lý các bản ghi
  }
}
```

📌 **Lý do không thể truyền đối tượng (sObject) vào Future Method:**  
Dữ liệu của đối tượng có thể thay đổi giữa thời điểm gọi phương thức và thời điểm nó thực sự thực thi. Điều này có thể gây ra lỗi dữ liệu không nhất quán.  

---

## **3. Callout trong Future Methods**  
Khi thực hiện callout từ trigger, ta phải sử dụng Future Method có `callout=true`:  
```apex
public class SMSUtils {
    @future(callout=true)
    public static void sendSMSAsync(String fromNbr, String toNbr, String m) {
        String results = sendSMS(fromNbr, toNbr, m);
        System.debug(results);
    }
    
    public static String sendSMS(String fromNbr, String toNbr, String m) {
        String results = SmsMessage.send(fromNbr, toNbr, m);
        insert new SMS_Log__c(to__c=toNbr, from__c=fromNbr, msg__c=results);
        return results;
    }
}
```

---

## **4. Testing Future Methods**  
Do Future Method chạy không đồng bộ, ta phải sử dụng `Test.startTest()` và `Test.stopTest()` để kiểm tra:  
```apex
@IsTest
private class Test_SMSUtils {
  @IsTest
  private static void testSendSms() {
    Test.setMock(HttpCalloutMock.class, new SMSCalloutMock());
    Test.startTest();
      SMSUtils.sendSMSAsync('111', '222', 'Greetings!');
    Test.stopTest();
    
    List<SMS_Log__c> logs = [SELECT msg__c FROM SMS_Log__c];
    System.assertEquals(1, logs.size());
    System.assertEquals('success', logs[0].msg__c);
  }
}
```

---

## **5. Best Practices khi sử dụng Future Methods**  
✅ **Giữ Future Method đơn giản và tối ưu tốc độ thực thi.**  
✅ **Gom các callout vào một phương thức duy nhất** thay vì gọi nhiều Future Methods.  
✅ **Kiểm thử trên tập dữ liệu lớn (trigger với 200 bản ghi) để đảm bảo hiệu suất.**  
✅ **Sử dụng Batch Apex thay vì Future Methods khi xử lý lượng lớn dữ liệu.**  

---

## **6. Những điều cần nhớ**  
⚠️ **Hạn chế của Future Methods:**  
❌ Không thể truyền trực tiếp đối tượng (sObject).  
❌ Không thể đảm bảo thứ tự thực thi.  
❌ Không thể gọi Future Method từ một Future Method khác.  
❌ Không thể sử dụng trong Visualforce Controllers trong `getMethodName()` hoặc constructor.  
❌ Giới hạn **50 future calls mỗi lần thực thi Apex**, và có giới hạn theo ngày.  

🚀 **Lời khuyên:** Nếu cần quản lý tốt hơn việc xử lý không đồng bộ, hãy sử dụng **Queueable Apex** hoặc **Batch Apex** thay vì Future Methods.