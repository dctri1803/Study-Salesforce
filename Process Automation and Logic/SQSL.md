### **Ngôn ngữ Tìm kiếm Đối tượng Salesforce (SOSL)**

#### **1. SOSL là gì?**
- SOSL là một ngôn ngữ tìm kiếm trong Salesforce, giúp tìm kiếm văn bản trên nhiều đối tượng (Objects) cùng lúc.
- Khác với SOQL (chỉ truy vấn một đối tượng mỗi lần), SOSL có thể tìm kiếm trên tất cả các đối tượng chuẩn và tùy chỉnh.
- SOSL tìm kiếm theo **từ khóa**, còn SOQL tìm kiếm theo **kết quả khớp chính xác**.

#### **2. Cách sử dụng SOSL trong Apex**
- SOSL có thể được nhúng trực tiếp vào mã Apex, gọi là **inline SOSL**.
- Ví dụ truy vấn tìm kiếm tất cả **Account** và **Contact** có chứa từ “SFDC” trong bất kỳ trường nào:
  
  ```apex
  List<List<SObject>> searchList = [FIND 'SFDC' IN ALL FIELDS 
                                      RETURNING Account(Name), Contact(FirstName,LastName)];
  ```

#### **3. Khác biệt giữa SOSL và SOQL**
| Đặc điểm     | SOSL | SOQL |
|-------------|------|------|
| Đối tượng tìm kiếm | Nhiều đối tượng cùng lúc | Chỉ một đối tượng mỗi lần |
| Cách so khớp dữ liệu | Tìm theo **từ khóa** | Tìm **chính xác** (trừ khi dùng ký tự đại diện) |
| Khi nào dùng? | Khi cần tìm kiếm dữ liệu trong nhiều đối tượng | Khi cần truy vấn có điều kiện cụ thể trên một đối tượng |

Ví dụ:
- SOSL (`'Digital'` trả về **"Digital"**, **"The Digital Company"**)
- SOQL (`'Digital'` chỉ trả về **"Digital"**)

#### **4. Cách chạy truy vấn SOSL trong Developer Console**
- Mở **Query Editor** trong Developer Console.
- Nhập truy vấn và nhấn **Execute**.

Ví dụ:
```sql
FIND {Wingo} IN ALL FIELDS RETURNING Account(Name), Contact(FirstName, LastName, Department)
```
-> Trả về **Account** và **Contact** có chứa từ “Wingo” trong bất kỳ trường nào.

**Lưu ý**:  
- Trong **Query Editor/API**, dùng `{Wingo}`.  
- Trong **Apex**, dùng `'Wingo'`.

#### **5. Cú pháp cơ bản của SOSL**
```apex
FIND 'SearchQuery' [IN SearchGroup] [RETURNING ObjectsAndFields]
```
- `SearchQuery`: Từ khóa tìm kiếm, có thể dùng **AND, OR, dấu ngoặc** để nhóm điều kiện.
- `SearchGroup`: Phạm vi tìm kiếm (**ALL FIELDS**, **NAME FIELDS**, **EMAIL FIELDS**, …).
- `ObjectsAndFields`: Danh sách đối tượng và trường cần lấy dữ liệu.

Ví dụ:
```apex
FIND 'SFDC OR Wingo' IN ALL FIELDS 
RETURNING Account(Name), Contact(FirstName, LastName, Department)
```

#### **6. Ký tự đại diện trong SOSL**
| Ký tự | Ý nghĩa |
|-------|--------|
| `*`   | Khớp **0 hoặc nhiều ký tự** (ví dụ: `wing*` khớp “wing”, “wingo”, “wings”) |
| `?`   | Khớp **chính xác 1 ký tự** (ví dụ: `w?ng` khớp “wing”, “wang”, nhưng không khớp “wng”) |

#### **7. Lọc, sắp xếp và giới hạn kết quả**
- **Lọc kết quả**:
  ```sql
  RETURNING Account(Name, Industry WHERE Industry='Apparel')
  ```
  -> Chỉ trả về các **Account** có ngành là "Apparel".

- **Sắp xếp kết quả**:
  ```sql
  RETURNING Account(Name ORDER BY Name)
  ```
  -> Sắp xếp **Account** theo **tên**.

- **Giới hạn số lượng kết quả**:
  ```sql
  RETURNING Account(Name LIMIT 10)
  ```
  -> Chỉ trả về **10 Account đầu tiên**.
