# Thiết kế cơ sở dữ liệu như thế nào với database là MongoDB?

> Hay còn gọi là thiết kế MongoDB schema.

Đây là câu hỏi đầu tiên chúng ta đặt ra mà khi chúng ta bắt đầu code một dự án với MongoDB. Và câu trả lời là: **`tùy vào từng trường hợp`**.

Vì có rất nhiều yếu tố ảnh hưởng đến schema của bạn. Ví dụ:

- Ứng dụng có tính chất đọc hay ghi nhiều?
- Dữ liệu nào thường được truy cập cùng nhau?
- Các yếu tố về hiệu suất của bạn như thế nào?
- Dữ liệu của bạn sẽ tăng và mở rộng như thế nào?

Bây giờ, chúng ta sẽ tìm hiểu về cách mô hình hóa database bằng việc sử dụng các ví dụ thực tế:

- Khi chúng ta sử dụng SQL để thiết kế CSDL lưu trữ thông tin người dùng, chúng ta sẽ phải phân mảnh ra thành các bảng khác nhau. Ví dụ, một người dùng có thể làm nhiều nghề nghiệp khác nhau và một người dùng có thể có nhiều xe hơi khác nhau.

![https://duthanhduoc.com/images/2023/thiet-ke-co-so-du-lieu-voi-mongodb/relational_user_model.png](https://duthanhduoc.com/images/2023/thiet-ke-co-so-du-lieu-voi-mongodb/relational_user_model.png)

Lúc này ta sẽ không thể lưu tất cả thông tin liên quan tới nghề nghiệp và xe hơi của người dùng vào trong cùng một bảng được, mà phải tách ra làm 3 bảng khác nhau, một bảng để lưu trữ thông tin độc nhất của người dùng ( những thứ mà tất cả mọi người đều chỉ có một như là tên tuổi, quê quán, ...), các bảng còn lại là lưu trữ những dữ liệu mà "một người dùng có thể có nhiều".

Ở ví dụ trên, Paul là một người dùng của website chúng ta (có id ở bảng Users là 1). Anh ta làm một lúc 3 nghề, vì vậy ta thấy khóa ngoại user_id = 1 là để trỏ tới id = 1 ở bảng Users, điều này có nghĩa là Paul làm cùng một lúc 3 nghề (banking, finance, trader). Tương tự với bảng Cars.

### Nhưng MongoDB nói riêng và NoSQL nói chung thì không thiết kế như vậy

MongoDB schema hoạt động khác với SQL. Với kiểu thiết kế cơ sở dữ liệu theo MongoDB, chúng ta sẽ:

- Không có quy trình chính thức
- Không có những thuật toán
- Không có những quy tắc

Khi bạn thiết kế cơ sở dữ liệu bằng MongoDB, bạn chỉ cần quan tâm là thiết kế đó hoạt động tốt cho ứng dụng của bạn là được. 2 App khác nhau, xử dụng data giống hệt nhau nhưng có thể nó sẽ có những schema khác nhau. Đơn giản vì 2 app đó được sử dụng theo những cách khác nhau.

Khi thiết kế cơ sở dữ liệu chúng ta nên quan tâm:

- Lưu data
- Cung cấp hiệu suất tốt khi query
- Yêu cầu phần cứng hợp lý

Bây giờ chúng ta cùng xem lại ví dụ trên như thế nào khi thiết kế theo MongoDB nhé

```json
{
  "first_name": "Paul",
  "surname": "Miller",
  "cell": "447557505611",
  "city": "London",
  "location": [45.123, 47.232],
  "profession": ["banking", "finance", "trader"],
  "cars": [
    {
      "model": "Bentley",
      "year": 1973
    },
    {
      "model": "Rolls Royce",
      "year": 1965
    }
  ]
}
```

Như các bạn thấy, tất cả nằm trong cùng một collection luôn, không lòng vòng. Nhưng mọi thứ không đơn giản tới vậy, đọc tiếp nhóe

## Nhúng và Tham chiếu

Khi thiết kế schema cho MongoDB, chúng ta sẽ đứng ở giữa 2 lựa chọn là **Embedding** hay **Referencing**, hay còn gọi là **nhúng** và **tham chiếu**.

Nhúng có nghĩa là đưa hết data vào trong một document, còn tham chiếu là lưu trữ data trong một document thuộc collection riêng biệt và tham chiếu đến nó thông qua việc sử dụng khóa ngoại và toán tử **$lookup** (tương tự JOIN trong SQL).

### Nhúng (nên ưu tiên khí có thể dùng)

Chính là cách làm giống như ví dụ trên, nhúng tất cả data vào trong một document

- Ưu điểm:

  - Bạn có thể truy xuất tất cả thông tin liên quan trong một query
  - Tránh việc join hoặc lookup trong ứng dụng
  - Update các thông tin liên quan trong một query duy nhất

- Hạn chế:

  - Khi document lớn lên sẽ gây gánh nặng cho những trường không liên quan. Bạn có thể tăng hiệu suất truy vấn bằng cách hạn chế kích thước của các document mà bạn gửi qua cho mỗi truy vấn.

  - Giới hạn cho document là 16 MB trong MongoDB. Nếu bạn nhúng quá nhiều dữ liệu bên trong một document duy nhất, bạn có thể đụng phải giới hạn này.

### Tham chiếu

Tham chiếu hoạt động tương tự như toán tử JOIN trong một truy vấn SQL. Nó cho phép chúng ta chia dữ liệu để tạo ra các truy vấn hiệu quả và có thể mở rộng hơn, nhưng vẫn duy trì mối quan hệ giữa dữ liệu.

- Ưu điểm:

  - Bằng cách chia dữ liệu, bạn sẽ có các document nhỏ hơn.

  - Ít khả năng đạt giới hạn 16-MB cho mỗi document.

  - Những dữ liệu không cần thiết sẽ không được đính kèm vào các truy vấn.

  - Giảm lượng trùng lặp dữ liệu. Tuy nhiên, điều quan trọng cần lưu ý là đôi khi chúng ta chấp nhận trùng lặp dữ liệu để đem lại một schema tốt hơn.

- Hạn chế:
  - Để truy xuất được hết data, chúng ta cần tối thiểu là 2 query hoặc dùng `$lookup`
