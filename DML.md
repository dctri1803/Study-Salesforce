Dưới đây là những ý chính cần nhớ về DML trong Apex:

### 1. **Các thao tác DML trong Apex**
   - **insert**: Thêm bản ghi mới.
   - **update**: Cập nhật bản ghi hiện có.
   - **upsert**: Chèn hoặc cập nhật bản ghi tùy theo ID hoặc một trường chỉ định.
   - **delete**: Xóa bản ghi (chuyển vào Recycle Bin).
   - **undelete**: Khôi phục bản ghi đã bị xóa từ Recycle Bin.
   - **merge**: Hợp nhất tối đa 3 bản ghi cùng loại, xóa bản ghi dư và di chuyển dữ liệu liên quan.

### 2. **ID được gán tự động khi chèn bản ghi**
   - Sau khi dùng `insert`, ID của bản ghi sẽ được tự động gán và có thể sử dụng tiếp.

### 3. **DML Bulk (Xử lý nhiều bản ghi)**
   - DML có thể thực hiện trên một danh sách bản ghi, giúp tối ưu hiệu suất và tránh vi phạm giới hạn governor.
   - Ví dụ:  
     ```apex
     List<Contact> contacts = new List<Contact>{
         new Contact(FirstName='Joe', LastName='Smith'),
         new Contact(FirstName='Kathy', LastName='Brown')
     };
     insert contacts; // Chỉ tính là 1 câu lệnh DML
     ```

### 4. **Upsert - Chèn hoặc cập nhật bản ghi**
   - **Không chỉ định trường**: Upsert dựa vào `Id` của bản ghi.
   - **Chỉ định trường**: Dựa vào một trường `External ID` hoặc trường có `idLookup=true`.
   - Nếu tìm thấy bản ghi → cập nhật, không tìm thấy → chèn mới.

### 5. **Database Methods (Thay thế cho DML)**
   - Các phương thức:  
     - `Database.insert()`
     - `Database.update()`
     - `Database.upsert()`
     - `Database.delete()`
     - `Database.undelete()`
     - `Database.merge()`
   - **Khác biệt với DML**:  
     - Hỗ trợ tham số `allOrNone` (mặc định `true`).
     - Khi `allOrNone=false`, nếu có lỗi, chỉ những bản ghi hợp lệ được chèn/cập nhật.

### 6. **Xử lý ngoại lệ DML**
   - DML thất bại sẽ ném `DmlException`.
   - Có thể bắt lỗi và xử lý như sau:
     ```apex
     try {
         Account acct = new Account();
         insert acct;
     } catch (DmlException e) {
         System.debug('Lỗi DML: ' + e.getMessage());
     }
     ```

### 7. **Xử lý quan hệ giữa các bản ghi**
   - **Chèn bản ghi có liên kết**:
     ```apex
     Account acct = new Account(Name='SFDC Account');
     insert acct;
     Contact mario = new Contact(FirstName='Mario', LastName='Ruiz', AccountId=acct.Id);
     insert mario;
     ```
   - **Cập nhật bản ghi liên quan** (Cần 2 câu lệnh `update` riêng biệt):
     ```apex
     Contact queriedContact = [SELECT Account.Name FROM Contact WHERE FirstName='Mario' LIMIT 1];
     queriedContact.Phone = '123456789';
     queriedContact.Account.Industry = 'Technology';
     update queriedContact;
     update queriedContact.Account;
     ```

Những ý trên giúp bạn nắm vững cách sử dụng DML trong Apex! 🚀
