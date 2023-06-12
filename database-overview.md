# 1. Cơ sở dữ liệu cơ bản

## Cơ sở dữ liệu là gì?

CSDL là nơi lưu trữ dữ liệu ví dụ: Google Sheet, MySQL, MongoDB, File có đuôi .json,...

## Tại sao phải cần nó?

Tại sao không lưu vào JSON cho dễ?

Đó là bởi vì 1 khi cơ sở dữ liệu lớn lên, nếu dùng 1 file JSON thì rất khó để quản lý. Cơ sở dữ liệu có thể lên đến hàng chục GB, mở 1 file JSON nặng chục GB sao mà nổi?

Vậy nên cần một hệ quản trị cơ sở dữ liệu (DBMS - Database Management System) để giúp chúng ta quản lý việc đọc và ghi thuận tiện hơn.

## Một số hệ quản trị CSDL

- SQL: MySQL, PostgeSQL, Oracle, MariaDB,...
- NoSQL: MongoDB, DynamoDB, Cassandra,...
- No-Code & Other: Google Sheets, Notion,...

Tham khảo thêm: [https://2022.stateofdb.com/sql](https://2022.stateofdb.com/sql)

# 2. SQL vs NoSQL

Đọc bài gốc tại đây: [SQL vs NoSQL: How to Choose](https://www.sitepoint.com/sql-vs-nosql-choose/)

## SQL

- Học 1 ngôn ngữ nhưng dùng được ở nhiều cơ sở dữ liệu khác nhau
- Schema chặt chẽ
- Khuyến khích các tiêu chuẩn chuẩn hóa để giảm thiểu sự dư thừa dữ liệu
- Có thể mở rộng nhưng sẽ hơi tốn công

## NoSQL

- Dữ liệu được lưu dưới dạng JSON với các cặp key-value
- Không cần Schema, lưu được hầu như bất cứ thứ gì
- Không cần sử dụng JOINS, để có thể tạo relationship giữa các bảng
- Hiệu năng tuyệt vời, cùng với khả năng scale dễ dàng

## Ví dụ về sự ưu việt của NoSQL

### Dùng SQL

Chúng ta cần xây dựng hệ thống dữ liệu cho danh bạ. Vậy nên sao khi khảo sát chúng ta có được bảng `contacts` như sau

- id
- title
- firstname
- lastname
- gender
- telephone
- email
- address1
- address2
- address3
- city
- region
- zipcode
- country

**Vấn đề 1**: Sau khi code được 1 thời gian chúng ta nhận ra rất ít người có 1 số điện thoại, thường họ sẽ có SDT nơi làm việc, ở nhà, cá nhân,... Vậy nên chúng ta không thể thêm `telephone1`, `telephone2`,... vào bảng `contacts` được. Chúng ta sẽ tạo thêm 1 bảng riêng là `telephones`

- id
- contact_id
- number
- telphone_type (kiểu enum lưu kiểu số điện thoại ở nhà, cty,...)

**Vấn đề 2**: Chúng ta lại gặp vấn đề tương tự với email, họ có thể có nhiều email. Vậy nên chúng ta tạo thêm bảng `emails`

- id
- contact_id
- address
- email_type ( có thể lưu dạng text hoặc để enum cho email nhà, email công ty v.v...)

**Vấn đề 3**: Người dùng có thể có nhiều địa chỉ nhà, nên chúng ta tạo thêm bảng `addresses`

- id
- contact_id
- address_type
- address
- city
- region
- zipcode
- country

Và cuối cùng bảng `contacts` chúng ta còn

- id
- title
- firstname
- lastname
- gender

Cuối cùng chúng ta đã chuẩn hóa được dữ liệu, phòng ngừa được rủi ro khi dữ liệu to lên.

Nhưng **không**

Chúng ta chưa xem xét hết trong tương lai 1 vài tuần nữa, yêu cầu được cho vào là thêm các trường như `date_of_birth`, `company` vào bảng `contacts`. Và cho do chúng ta thêm bao nhiêu trường đi chăng nữa thì cũng không thể cảm thấy đủ, vì yêu cầu sẽ được thêm vào làm chúng ta thêm `social_network_account`, `hobby`,...

Vậy mỗi lần thêm 1 trường vào bảng là chúng ta đều phải suy nghĩ tạo thêm 1 bảng là `otherdata` hay sao?

> Điều này làm cho CSDL chúng ta bị chia nhỏ ra hàng chục, hàng trăm bảng => Chúng bị phân mảnh

Việc phân mảnh database sẽ làm cho dev khó có thể quan sát được logic của database, vì càng phân mảnh sẽ càng làm tăng độ phức tạp => **thời gian phát triển ứng dụng bị chậm đi**

Khi chúng ta muốn query để lấy data đầy đủ của một contact, chúng ta phải dùng JOIN kết hợp với hàng chục bảng liên quan

Áp dụng full-text search cũng khó hơn. Ví dụ ai đó nhập một chuỗi bất kỳ để tìm kiếm trong danh bạ, chúng ta phải kiểm tra tất cả trong các bảng `contacts`, `telephones`, `addresses`, `emails`... và sắp xếp thứ tự các kết quả tương ứng.

> SQL quá cứng nhắc

### Dùng NoSQL (MongoDB) thay thế

Dữ liệu của bảng `contacts` liên quan đến con người. Chúng ta không thể đoán trước được, và yêu cầu từng thời điểm sẽ khác nhau. Vậy nên lúc này NoSQL sẽ phát huy ưu điểm của mình, chúng ta có thể lưu tất cả data liên quan `contacts` trong một document của một collection duy nhất:

```js
{
  name: [
    "Billy", "Bob", "Jones"
  ],
  company: "Fake Goods Corp",
  jobtitle: "Vice President of Data Management",
  telephone: {
    home: "0123456789",
    mobile: "9876543210",
    work: "2244668800"
  },
  email: {
    personal: "bob@myhomeemail.net",
    work: "bob@myworkemail.com"
  },
  address: {
    home: {
      line1: "10 Non-Existent Street",
      city: "Nowhere",
      country: "Australia"
    }
  },
  birthdate: ISODate("1980-01-01T00:00:00.000Z"),
  twitter: '@bobsfakeaccount',
  note: "Don't trust this guy",
  weight: "200lb",
  photo: "52e86ad749e0b817d25c8892.jpg"
}
```

Ở data trên, chúng ta chưa lưu `date_of_birth` hay `social_network_account`, nhưng không sao, vì NoSQL rất linh hoạt trong trường hợp này, có thể thêm sửa xóa thoải mái.

Vì chúng ta lưu hết trong 1 document, chúng ta có thể dễ dàng biết được cấu trúc tổng quát của data, cũng như dễ dàng thực hiện full text search tất cả các trường trong `contacts`

Đầu tiên chúng ta định nghĩa index cho tất cả các trường trong collection `contacts`

```js
db.contacts.createIndex({ "$**": "text" });
```

và thực hiện full text search

```js
db.contact.find({
  $text: { $search: "something" },
});
```

## Khi nào nên dùng MongoDB

MongoDB là CSDL đa năng được dùng theo nhiều cách khác nhau, có thể dùng được trong nhiều ngành nghề (Viễn thông, game, tài chính, sức khỏe, bán lẻ)

- Khi muốn tích hợp lượng data lớn
- Data có cấu trúc phức tạp
- Khi cần một CSDL có khả năng mở rộng nhanh, rẻ
- Khi cần một CSDL giúp tốc độ phát triển phần mềm nhanh

## Khi nào nên dùng SQL

- Cần một CSDL chặt chẽ về cấu trúc
- Bạn quen thuộc với ngôn ngữ SQL

## Đọc thêm

- [The Top 4 Reasons Why You Should Use MongoDB](https://www.mongodb.com/developer/products/mongodb/top-4-reasons-to-use-mongodb)
- [Why Use MongoDB and When to Use It?](https://www.mongodb.com/why-use-mongodb)

# MongoDB cơ bản

## MongoDB là gì

- Một hệ quản trị cơ sở dữ liệu NoSQL
- Ra đời 2007, được dùng rộng rãi trên toàn thế giới, có thể coi là một trong những CSDL NoSQL được dùng nhiều nhất hiện nay
- Thay vì lưu trong từng hàng hay cột như SQL, MongoDB dùng document theo dạng BJSON. Ứng dụng có thể lấy data ra theo dạng JSON

```json
{
  "_id": 1,
  "name": {
    "first": "Ada",
    "last": "Lovelace"
  },
  "title": "The First Programmer",
  "interests": ["mathematics", "programming"]
}
```

- Tính linh hoạt cao, cho phép lưu nhiều loại cấu trúc dữ liệu
- Khả năng scale dễ dàng

## Cài đặt MongoDB

- Tải về máy và cài trên local
- **Dùng Cloud**: Mongo Atlas

Khi đang học thì nên dùng Mongo Atlas cho tiện, để khi cần support thì mọi người chỉ cần gửi code, mọi người có thể truy cập vào db của bạn mà không cần tải db bạn về.

## Kết nối MongoDB

- Dùng Mongo Compass
- Dùng MongoSH (terminal)
- Dùng mongo driver (SDK tích hợp vào code)
- Dùng Extension MongoDB cho VS Code

## Một số thuật ngữ trong MongoDB

Mình sẽ lấy Mongo Atlas làm ví dụ

- Cấp độ cao nhất là Organization
- 1 Organization có thể có nhiều projects
- 1 project có thể có nhiều clusters
- 1 cluster có thể có nhiều databases
- Trong mỗi database chúng ta lại có các collections
- Mỗi collection lại có nhiều documents

Cluster có thể hiểu như là một máy chủ VPS, dùng để cài đặt MongoDB. Từ đó chúng ta có thể tạo thêm nhiều database trên máy chủ đó

Collection tương đương với bảng bên SQL
Document tương đương hàng bên SQL
