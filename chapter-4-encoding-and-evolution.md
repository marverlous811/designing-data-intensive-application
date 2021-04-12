## Chapter 4: encoding and evolution

- Các ứng dụng tất yếu sẽ phải thay đổi theo thời gian. Các tính năng mới được thêm vào hoặc thay đổi để tạo ra 1 sản phẩm mới, với yêu cầu dễ hiểu hơn hoặc bussiness bị thay đổi. Vì vậy chúng ta sẽ phải hướng tới tạo ra 1 hệ thống dễ dàng đáp ứng sự thay đổi đó. Phần lớn trường hợp, sự thay đổi đó cũng kéo theo sự thay đổi các lưu trữ dữ liệu: field mới, kiểu dữ liệu, hoặc dữ liệu cũ sẽ phải biểu thị thoe cách khác. Bằng việc ràng buộc với schema và DB ràng buộc với schema đó => DB có thể mix giữa format dữ liệu mới và cũ ở mỗi thời điểm khác nhau.

- Khi format hoặc schema được thay đổi, code cũng sẽ phải thay đổi thoe. Tuy nhiên các ứng dụng lơn, việc thay đổi code thường xuyên không xảy ra ngay lập tức được:
  
  - Backend: Bạn có thể sử dụng rolling upgrade để deploy phiên bản mới ở vài node. Sau thời gian sử dụng thử, phiên bản mới được xác nhận là chạy mượt mà thì mới update lên toàn bộ các node còn lại. Điều này cho phép việc deploy sẽ không bị downtime và giúp tăng tần suất release với khả năng phát triển tốt hơn.

  - Client: Bạn sẽ phải quan tâm đến nhưng user mà không update phiên bản mới thường xuyên

- Điều này có nghĩa là phiên bản cũ và phiên bản mới, format cũ và format mới cần có khả năng tồn tại song song trong hệ thông tại cùng 1 thời điểm. Để cho hệ thống mượt mà hơn, chúng ta có thể duy trì khả năng tương thích bằng 2 hướng

  - Tương thích ngược (<i>Backward compatibility</i>): phiên bản mới có khả năng đọc dữ liệu được tạo ra bởi phiên bản cũ. Các tiếp cận này thì không quá khó khăn khi người viết code mới sẽ biết được phiên bản cũ sẽ tạo ra định dạng dữ liệu như thế nào và có thể handle được nó

  - Tương thích xuôi (<i>Forward compatibility</i>): phiên bản cũ có thể đọc dữ liệu được tạo ra bởi phiên bản mới. Việc này có thể khó hơn, bởi vì nó yêu cầu code cũ phải bỏ qua phần được thêm vào từ phiên bản code mới.

Trong chapter này chúng ta sẽ xem qua 1 vào format để encoding dữ liệu (JSON, XML, protocol buffers, Thirft???, Avro???), chúng ta sẽ tìm hiểu cách chúng xử lý các thay đổi của schema và các chúng hỗ trỡ việc tương thích giữa format cũ và mới. Chúng ta sẽ xem xét chúng trong việc lưu trữ và giao tiếp (web service, REST, RPC (message passing sytems such as actor and msg queue)) 

### Formats for encoding data

Các chương trình thường sẽ sử dụng 2 kiểu biển diễn dữ liệu

- In memory: dữ liệu được luwu vào object, struct, list, mảng, hash tables, cây.... Những cấu trúc dữ liệu này được tối ưu để truy cập và xử lý bowri CPU (thông thường sẽ dưới dạng pointer)

- Khi bạn muốn lưu dữ liệu ra file hoặc gửi chúng ra ngoài mạng, bạn phải chuyển hóa chúng thành 1 chuỗi các bytes

Do đó, ta sẽ cần 1 kiểu biến đổi giữa 2 dạng biển diễn. Biến đổi từ memory thành sequence of byte được gọi là encode, serialization, marshalling và ngược lại thì được gọi là decode, parsing deserialization, unmarshalling

### Language Specific Format 

Các thư viện encoding rất là tiện lợi, vì chúng cho phép lưu lại và khôi phục các in-memory object với số lượng code ít. Tuy nhiên chúng sẽ có một vào vấn đề:

- encoding thường bị ràng buộc bởi ngôn ngữ lập trình cụ thể và đọc dữ liệu từ 1 ngôn ngữ khác sẽ khá khó.

- để khôi phục lại đúng kiểu dữ liệu, decoding cần phải có khăng năng khởi tạo. Điều này có thể gây ra một số vấn đề security: Kẻ tấn công có thể yêu cầu ứng dụng của bạn giải mã 1 chuỗi byte tùy ý nó có thể khởi tạo các lớp tùy ý cho phép chúng thực hiện những thứ kinh khủng từ xa tùy ý.

- Phiên bản dữ liệu thường là một vấn đề cần cân nhắc trong số các thư viện này. vì chúng tập trung vào việc encoding nhanh chóng và dễ dàng, nên sẽ thường xuyên bỏ qua các vấn đề tương thích.

- Hiệu năng cũng là vấn đề cần được cân nhắc