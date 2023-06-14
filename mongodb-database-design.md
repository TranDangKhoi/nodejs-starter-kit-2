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

## Các quy tắc cần nhớ để có thể nhận biết khi nào nên dùng nhúng, khi nào nên dùng tham chiếu

- Quy tắc 1: Ưu tiên nhúng trừ khi chúng ta có một lý do thuyết phục để không làm như vậy

- Quy tắc 2: Khi cần truy cập vào một đối tượng riêng biệt, đây là lúc nên dùng tham chiếu

- Collection Products:

```json
{
  "name": "Some random bicycle",
  "manufacturer": "Acme Corp",
  "catalog_number": "1234",
  "parts": ["ObjectID('AAAA')", "ObjectID('BBBB')", "ObjectID('CCCC')"]
}
```

- Collection Parts:

```json
{
  "_id": "ObjectID('AAAA')",
  "partno": "123-aff-456",
  "name": "#4 grommet",
  "qty": "94",
  "cost": "0.94",
  "price": " 3.99"
}
// ... Tự tưởng tượng thêm 2 parts ObjectID BBBB và CCCC
```

- Quy tắc 3: Tránh joins/lookups nếu có thể, nhưng cũng đừng sợ nếu nó giúp chúng ta có một schema tốt hơn

- Quy tắc 4 (đặc biệt chú ý): Array không nên phát triển không giới hạn. Nếu có hơn vài trăm document ở phía "nhiều" thì đừng nhúng chúng; Nếu có hơn vài ngàn document ở phía "nhiều" thì đừng sử dụng array ObjectID tham chiếu. Mảng với số lượng lớn item là lý do không nên dùng nhúng.

#### Giải thích lí do

Điều gì sẽ xảy ra nếu chúng ta có một schema mà có khả năng có đến hàng triệu, hàng tỷ các document phụ thuộc. À mà ngoài đời liệu có trường hợp nào có đến hàng triệu, hàng tỉ document phụ thuộc không? Câu trả lời là có, và nó rất thực tế.

Hãy cùng tưởng tượng chúng ta tạo một ứng dụng ghi log server. Mỗi máy chủ có thể lưu trữ hàng tỉ message log. Nếu dùng array trong MongoDB, cho dù các bạn đã dùng array chứa các Object ID thì cũng có khả năng các bạn chạm đến giới hạn là 16 MB cho document. Vậy nên chúng ta cần suy nghĩ lại cách thiết kế làm sao cho khi database phình to ra thì vẫn không bị giới hạn.

Bây giờ, thay vì tập trung mối quan hệ giữa host và log message, chúng ta hãy nhìn ngược lại, mỗi log message sẽ lưu trữ một host mà nó thuộc về. Bằng cách này thì chúng ta sẽ không sợ bị giới hạn bởi 16 MB nữa.

Hosts:

```json
{
    "_id": ObjectID("AAAB"),
    "name": "goofy.example.com",
    "ipaddr": "127.66.66.66"
}
```

Log Message:

```json
{
    "time": ISODate("2014-03-28T09:42:41.382Z"),
    "message": "cpu is on fire!",
    "host": ObjectID("AAAB")
}
```

- Quy tắc 5: Với MongoDB, cách bạn mô hình hóa dữ liệu phụ thuộc vào cách bạn sử dụng dữ liệu. Bạn muốn cấu trúc dữ liệu của bạn phù hợp với cách mà ứng dụng của bạn query và update nó.

### Recap

1 - 1: Ưu tiên cặp key-value trong document

1 - ít: Ưu tiên nhúng

1 - nhiều: Ưu tiên tham chiếu

1 - rất nhiều: Ưu tiên tham chiếu

Nhiều - Nhiều: Ưu tiên tham chiếu

# Thiết kế Schema Twitter bằng MongoDB

## Một số ghi chú nhỏ

- Tên collection nên được đặt theo dạng số nhiều, kiểu snake_case, ví dụ `users`, `refresh_tokens`

- Tên field nên được đặt theo dạng snake_case, ví dụ `email_verify_token`, `forgot_password_token`

- `_id` là trường được MongoDB tự động tạo ra, không cần phải thêm trường `_id` vào. Cũng không nên tìm mọi cách để đổi tên `_id` thành `id` hay thay đổi cơ chế của nó. Vì sẽ làm giảm hiệu suất của MongoDB

- Trường `created_at`, `updated_at` nên có kiểu `Date` để dễ dàng sắp xếp, tìm kiếm, lọc theo thời gian

- Trường `created_at` nên luôn luôn được thêm vào khi tạo mới document

- Trường `updated_at` thì optional

- Tất cả trường đại diện id của document thì nên có kiểu `ObjectId`

- Để biết kiểu dữ liệu mà mongo hỗ trợ thì xem tại [đây](https://docs.mongodb.com/manual/reference/bson-types/)

## Phân tích chức năng

## users

- Người dùng đăng ký nhập `name`, `email`, `day_of_birth`, `password` là được. Vậy `name`, `email`, `day_of_birth`, `password` là những trường bắt buộc phải có bên cạnh`_id` là trường tự động tạo ra bởi MongoDB

- Sau khi đăng ký xong thì sẽ có email đính kèm `email_verify_token` để xác thực email (`duthanhduoc.com/verify-email?email_verify_token=123321123`). Mỗi user chỉ có 1 `email_verify_token` duy nhất, vì nếu user nhấn re-send email thì sẽ tạo ra `email_verify_token` mới thay thế cái cũ. Vậy nên ta lưu thêm trường `email_verify_token` vào schema `users`. Trường này có kiểu `string`, nếu user xác thực email thì ta sẽ set `''`.

- Tương tự ta có chức năng quên mật khẩu thì sẽ gửi mail về để reset mật khẩu, ta cũng dùng `forgot_password_token` để xác thực (`duthanhduoc.com/forgot-password?forgot_password_token=123321123`). Vậy ta cũng lưu thêm trường `forgot_password_token` vào schema `users`. Trường này có kiểu `string`, nếu user reset mật khẩu thì ta sẽ set `''`.

- Nên có một trường là `verify` để biết trạng thái tài khoản của user. Ví dụ chưa xác thực email, đã xác thực, bị khóa, lên tích xanh ✅. Vậy giá trị của nó nên là enum

- Người dùng có thể update các thông tin sau vào profile: `bio`, `location`, `website`, `username`, `avatar`, `cover_photo`. Vậy ta cũng lưu các trường này vào schema `users` với kiểu là `string`. `avatar`, `cover_photo` đơn giản chi là string url thôi. Đây là những giá trị optional, tức người dùng không nhập vào thì vẫn dùng bình thường. Nhưng cũng nên lưu set `''` khi người dùng không nhập gì để tiện quản lý.

- Cuối cùng là trường `created_at`, `updated_at` để biết thời gian tạo và cập nhật user. Vậy ta lưu thêm 2 trường này vào schema User với kiểu `Date`. 2 trường này luôn luôn có giá trị.

```ts
enum UserVerifyStatus {
  Unverified, // chưa xác thực email, mặc định = 0
  Verified, // đã xác thực email
  Banned, // bị khóa
}
interface User {
  _id: ObjectId;
  name: string;
  email: string;
  date_of_birth: Date;
  password: string;
  created_at: Date;
  updated_at: Date;
  email_verify_token: string; // jwt hoặc '' nếu đã xác thực email
  forgot_password_token: string; // jwt hoặc '' nếu đã xác thực email
  verify: UserVerifyStatus;

  bio: string; // optional
  location: string; // optional
  website: string; // optional
  username: string; // optional
  avatar: string; // optional
  cover_photo: string; // optional
}
```

## refresh_tokens

Hệ thống sẽ dùng JWT để xác thực người dùng. Vậy mỗi lần người dùng đăng nhập thành công thì sẽ tạo ra 1 JWT access token và 1 refresh token.

- JWT access token thì không cần lưu vào database, vì chúng ta sẽ cho nó stateless
- Còn refresh token thì cần lưu vào database để tăng tính bảo mật.

Một user thì có thể có nhiều refresh token (không giới hạn), nên không thể lưu hết vào trong collection `users` được => Quan hệ 1 - rất nhiều

Và đôi lúc chúng ta chỉ quan tâm đến refresh token mà không cần biết user là ai. Vậy nên ta tạo ra một collection riêng để lưu refresh token.

```ts
interface RefreshToken {
  _id: ObjectId;
  token: string;
  created_at: Date;
  user_id: ObjectId;
}
```

## followers

Một người dùng có thể follow rất nhiều user khác, nếu dùng 1 mảng `followings` chứa ObjectId trong collection `users` thì sẽ không tối ưu. Vì dễ chạm đến giới hạn 16MB của MongoDB.

Chưa hết, nếu dùng mảng `followings` thì khi muốn tìm kiếm user A đang follow ai rất dễ nhưng ngược lại, tìm kiếm ai đang follow user A thì lại rất khó.

Vậy nên ta tạo ra một collection riêng để lưu các mối quan hệ follow giữa các user là hợp lý hơn cả.

1 user có rất nhiều follower, và 1 follower cũng có rất nhiều user khác follow lại => Quan hệ rất nhiều - rất nhiều

```ts
interface Follower {
  _id: ObjectId;
  user_id: ObjectId;
  followed_user_id: ObjectId;
  created_at: Date;
}
```

## tweets

Chúng ta sẽ chọn ra những tính năng chính của tweet để clone

1. Tweet có thể chứa text, hashtags, metions, ảnh, video
2. Tweet có thể hiển thị cho everyone hoặc Twitter Circle
3. Tweet có thể quy định người reply (everyone, người mà chúng ta follow , người chúng ta metion)

- Tweet sẽ có nested tweet, nghĩa là tweet có thể chứa tweet con bên trong. Nếu dùng theo kiểu nested object sẽ không phù hợp, vì sớm thôi, nó sẽ chạm đến giới hạn. Chưa kể query thông tin 1 tweet con rất khó.

Vậy nên ta sẽ lưu trường `parent_id` để biết tweet này là con của ai. Nếu `parent_id` là `null` thì đó là tweet gốc.

- Nếu là tweet bình thường thì sẽ có `content` là string. Còn nếu là retweet thì sẽ không có `content` mà chỉ có `parent_id` thôi, lúc này có thể cho content là `''` hoặc `null`, như mình phân tích ở những bài trước thì mình thích để `''` hơn, đỡ phải phân tích trường hợp `null`. Vậy nên `content` có thể là `string`.

> Nếu là '' thì sẽ chiếm bộ nhớ hơn là null, nhưng điều này là không đáng kể so với lợi ích nó đem lại

- `audience` đại diện cho tính riêng tư của tweet. Ví dụ tweet có thể là public cho mọi người xem hoặc chỉ cho nhóm người nhất định. Vậy nên `visibility` có thể là `TweetAudience` enum.

- `type` đại diện cho loại tweet. Ví dụ tweet, retweet, quote tweet.

- `hashtag` là mảng chứa ObjectId của các hashtag. Vì mỗi tweet có thể có nhiều hashtag. Vậy nên `hashtag` có thể là `ObjectId[]`.

- `mentions` là mảng chứa ObjectId của các user được mention. Vì mỗi tweet có thể có nhiều user được mention. Vậy nên `mentions` có thể là `ObjectId[]`.

- `medias` là mảng chứa ObjectId của các media. Vì mỗi tweet chỉ có thể có 1 vài media. Nếu upload ảnh thì sẽ không upload được video và ngược lại. Vậy nên `medias` có thể là `Media[]`.

- Bên twitter sẽ có rất là nhiều chỉ số để phân tích lượt tiếp cận của 1 tweet. Trong giới hạn của khóa học thì chúng ta chỉ phân tích lượt view thôi.

  Lượt view thì chúng ta chia ra làm 2 loại là `guest_views` là số lượng lượt xem của tweet từ người dùng không đăng nhập và `user_views` là dành cho đã đăng nhập. 2 trường này mình sẽ cho kiểu dữ liệu là `number`.

```ts
interface Tweet {
  _id: ObjectId;
  user_id: ObjectId;
  type: TweetType;
  audience: TweetAudience;
  content: string;
  parent_id: null | ObjectId; //  chỉ null khi tweet gốc
  hashtags: ObjectId[];
  mentions: ObjectId[];
  medias: Media[];
  guest_views: number;
  user_views: number;
  created_at: Date;
  updated_at: Date;
}
```

```ts
interface Media {
  url: string;
  type: MediaType; // video, image
}
enum MediaType {
  Image,
  Video,
}
enum TweetAudience {
  Everyone, // 0
  TwitterCircle, // 1
}
enum TweetType {
  Tweet,
  Retweet,
  Comment,
  QuoteTweet,
}
```

## bookmarks

Bookmark các tweet lại, mỗi user không giới hạn số lượng bookmark. Sở dĩ không cần `updated_at` là vì trong trường hợp người dùng unbookmark thì chúng ta sẽ xóa document này đi.

```ts
interface Bookmark {
  _id: ObjectId;
  user_id: ObjectId;
  tweet_id: ObjectId;
  created_at: Date;
}
```

## likes

Tương tự `bookmarks` thì chúng ta có collection `likes`

```ts
interface Like {
  _id: ObjectId;
  user_id: ObjectId;
  tweet_id: ObjectId;
  created_at: Date;
}
```

## hashtags

- Hỗ trợ tìm kiếm theo hashtag.
- Mỗi tweet có thể có ít hashtag.
- Mỗi hashtag có rất nhiều tweet.

❌Không nên làm như dưới đây

```ts
interface Tweet {
  _id: ObjectId
  user_id: ObjectId
  type: TweetType
  audience: TweetAudience
  content: string
  parent_id: null | ObjectId //  chỉ null khi tweet gốc
  ❌hashtags:string[] // Không nên nhúng như thế này, vì sẽ gây khó khăn trong việc tìm kiếm những tweet nào có hashtag này, cũng như là gây lặp lại dữ liệu về tên hastag
  mentions: ObjectId[]
  medias: Media[]
  guest_views: number
  user_views: number
  created_at: Date
  updated_at: Date

}
```

=> Quan hệ ít - rất nhiều

- Lưu một array ObjectId `hashtags` trong collection `tweets`

- Tạo ra một collection riêng để lưu `hashtags` và không lưu mảng `tweet_id` vào trong collection `hashtags`. Vì nếu lưu `tweet_id` vào trong collection `hashtags` thì sẽ dễ chạm đến giới hạn 16MB của MongoDB. Và cũng không cần thiết để lưu, vì khi search các tweet liên quan đến hashtag thì chúng ta sẽ dùng id hashtag để tìm kiếm trong collection `tweets`.

```ts
interface Hashtag {
  _id: ObjectId;
  name: string;
  created_at: Date;
}
```
