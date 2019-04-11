# Đăng kí

1. Gate node đăng kí với gate contract
2. Gate sinh ra số ngẫu nhiên (tương tự API_KEY)
3. Key này lưu vào storage của contract và ko public ra
4. Key được trả về cho gate node

Khi gate node set data thì gửi kèm hash của (key & nonce). Nonce được tăng 1 sau mỗi lần và được lưu lại. Như vậy hash sẽ luôn thay đổi và không thể reuse bởi kẻ gian.

Như vậy gate contract có thể verify là 1 lời gọi có đúng là từ 1 gate node đã đăng ký hay ko.

Có 2 cách để contract X nhận offchain data:
- gate node set trực tiếp vào X. X gọi sang gate contract để verify
- gate node gọi vào gate contract, gate contract gọi vào X. X check chỉ nhận sender là gate contract

Ban đầu MVP thì cách nào cũng ok, nhưng cách 2 là phù hợp vì sau này gate contract đứng ra làm trung gian để lựa chọn giữa các gate node (đầu giá).
