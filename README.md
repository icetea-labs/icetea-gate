# icetea-gate

A decentralized gate to the off-chain world.

## Đặt vấn đề

Việc có thể truy cập từ smart contract ra thế giới off-chain là yêu cầu thiết yếu nếu muốn viết các dapp thiết thực cho người dùng. Thông tin cần lấy từ off-chain vô cùng đa dạng, một số ví dụ:
- các app liên quan cá cược thường cho đặc cược về các sự kiện ngoài thế giới thực (kết quả bóng đá, ai thắng cử tổng thống, kết quả xổ số, đua ngựa, v.v.).
- các app tài chính cần thông tin bên ngoài: thông tin về giao dịch, giá cả, v.v. Ví dụ: nếu người dùng chuyển tiền cho tôi qua hệ thống ngoài (VISA, thẻ cào) thì contract được kích hoạt.

Các ví dụ thì còn nhiều.

## Cách tiếp cận hiện tại

Đa phần thông tin offchain được nhập vào smart contract theo 2 cách:
1. Smart contract được code sao cho có thể phân quyền cho 1 số admin được input off-chain data. Sau đó, admin nhập bằng tay (qua UI hoặc script)
2. Tạo ra 1 hệ thống offchain để dùng quyền admin update contract 1 cách tự động

Nhược điểm:
- Phải tin người hoặc tổ chức chịu trách nhiệm nhập liệu. Tưởng tượng: bạn chơi lô trên mạng, mà kết quả lô lại do chủ lô nhập vào thì làm sao mà tin được?
- Phức tạp: vì không có framework sẵn, ai muốn làm tool đều phải tự làm

## Trong bối cảnh của Icetea Blockchain

Hiện tại cũng có 1 số giải pháp, ví dụ:
- Oraclize: không phi tập trung
- Chainlink: chỉ tập trung vào Ethereum, khó hy vọng nó sẽ hỗ trợ icetea trong ngắn và trung hạn

Mục tiêu của Icetea là blockchain hữu ích có thể dùng được thật cho các ứng dụng thực tế. Vì thế cần thiết kế cơ chế truy cập offchain data ngay từ đầu.

## Cách dùng Icetea Gate

Trên Icetea, deploy 1 contract, gọi là `gate` contract.
Các contract khác khi muốn truy cập thông tin off-chain thì sẽ thông qua `gate` contract.

> Icetea sẽ deploy sẵn 1 contract như vậy, gọi là `system.gate`. Tuy nhiên, các 3rd-party có thể cung cấp các contract khác nếu muốn.

Cách dùng.

Khi yêu cầu thông tin.

```js
const gate = require('system.gate') // loadContract system.gate
const queryOpts = {
  path: 'sport/soccer/premier_league',
  data: { // depends on 'path'
    game: 'manu-liv',
    date: '2019/02/01'
  },
  condition: {
    datasources: 2, // at least 2 datasources agree
    reputation: 4, // only accept data source with reputation >= 4 stars
  }
}
gate.query(queryOpts, 'myCallbackFunctionName', 'someValue')
```

Cần chú ý là khi gọi `gate.query` bạn cần truyền thông tin về tên hàm nhận kết quả (ở ví dụ trên là `myCallbackFunctionName`. Tên hàm là `string` chứ không phải 1 `function`. Nếu bạn bỏ qua tham số này thì giá trị default là `onOffchainData`. Bạn thậm chí có thể điền tên 1 hàm ở contract khác sẽ nhận callback (Ví dụ: `someContractAddress.someFunctionName`. Mặc định là chính contract này.

```js
myCallbackFunctionName(offchainData, callId) {
   if (callId === 'someValue') { // 'someValue' is passed when you call gate.query
     // do something with offchainData
   }
}
```

Chú ý là `myCallbackFunctionName` sẽ được gọi trong __block__ tới chứ không phải gọi ngay trong transaction đang xảy ra (một block ở icetea blockchaihn có thể mất vài giây). Khi coding smart contract, bạn cần chú ý điều đó.

## Icetea gate nodes

Icetea gate nodes là các node sẽ liên tục monitor `system.gate`. Khi có event thì sẽ thực hiện query offchain data theo yêu cầu, sau đó gọi callback function để cung cấp offchain data cho contract.

Mục tiêu của Icetea là cung cấp offchain data 1 cách _decentralized_, nên các gate nodes này sẽ là 1 mạng lưới được quản lý 1 cách decentralized và có incentive cho việc cung cấp data. Đối với mỗi node lại có 1 danh sách các datasources. Cùng 1 loại dữ liệu có thể có nhiều nguồn data để so sánh với nhau. Các nodes cũng như các datasources sẽ được dánh giá điểm tín nhiệm (reputation) bằng hệ thống sao (rating). Người đánh giá cần được verify bằng hệ thống digital identity của Icetea.

Các thiết kế cụ thể của hệ thống decentralized gate nói trên sẽ được làm rõ hơn ở tài liệu khác.

## Bản MVP

Ở bản MVP thì chỉ cần 01 gate node và chạy được theo flow ở trên. Chưa cần có tính năng rating các nguồn data.
Demo sẽ là 1 bot về dự đoán xổ số Miền Bắc hoặc dự đoán kết quả bóng đá.

Thời gian demo MVP: 24 April.
Mục tiêu của Icetea là cung cấp offchain data 1 cách _decentralized_, nên các gate nodes này sẽ là 1 mạng lưới được quản lý 1 cách decentralized. Đối với mỗi node lại có 1 danh sách các datasources. Cùng 1 loại dữ liệu có thể có nhiều nguồn data để so sánh với nhau. Các nodes cũng như các datasources sẽ được dánh giá điểm tín nhiệm (reputation) bằng hệ thống sao (rating). Người đánh giá cần được verify bằng hệ thống digital identity của Icetea.
