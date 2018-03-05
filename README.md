# 인스테드 연동가이드

### 결제대행 서비스 인스테드를 연동하는 방법입니다.

## 연동하기에 앞서서
##### “http://”, “ftp://”, “market://”과 같은 문자열을 url scheme이라 부릅니다. url scheme을 통해 앱이 실행되는 방식은 다음과 같습니다.

##### - 웹페이지에서 하이퍼링크 클릭 시 url scheme이 system에 전달됨
##### - system에서 전달된 url scheme을 보고 실행 가능한 앱이 있는지 확인
##### - 해당 url scheme을 받을 수 있는 앱이 있다면 앱을 실행시키며 이 url을 함께 전달
##### - 앱이 실행되면서 url에 포함된 내용을 참조해서 특정 기능을 수행함

## 1. 인스테드 [관리자 페이지](https://admin.instead.co.kr/)에서 가맹점등록

## 2. 인스테드 라이브러리를 HTML에 추가합니다.

```html
<script src="instead-guide/instead-1.0.0.js"></script>
```

## 3. init(client_id)을 호출합니다.
* 매개변수 __client_id__는 거래할 고객의 고유 번호를 string 형식으로 전달하며, 해당 고유 번호를 가진 고객의 거래를 위한 초기화 작업을 진행합니다.
* 고객 id는 {"client_id": client_id}의 JSON 형식으로 POST 메소드를 통해 _/token_ 으로 전달됩니다.
###### &nbsp; example
```javascript
INSTEAD.init('gqvQvhVU0M')
```


## 4. 새로운 인증 token을 발급 받습니다.

```javascript
var req = new XMLHttpRequest();
this.access_token = null;
this.access_token = JSON.parse(req.responseText).access_token;
```

## 5. requestPay(input)를 호출하여 결제를 요청합니다.
* input은 JSON 형식으로 전달하며, 아래 정보들을 포함해야 합니다.

| 키 이름     | 타입          | 설명                                                  |
|-------------|---------------|----------------------------------------------------|
| __name__        | _string_        | 결제 요청자의 이름                                     |
| __tel__         | _string_        | 결제 요청자의 전화번호(예: "010-1234-5678")             |
| __merchantUid__ | _string_        | 각 주문 건의 고유 번호("itsm_" + (요청 시간)으로 이루어지며, 데이터베이스에서 키로 사용)           |
| __post__        | _string_        | 배송지의 우편번호 5자리                                 |
| __addr__        | _string_        | 배송할 주소                                          |
| __email__       | _string_        | 결제 요청자의 이메일 주소                               |
| __itemList__    | _JSON[]_        | 주문할 각 상품을 JSON 형태로 나타낸 객체의 배열             |
| __payEndDttm__  | _string_        | 결제 요청 만료 시간                                    |
| __sandboxYn__   | _string_        | 가상 결제 여부(가상 결제: "1", 실제 결제: "0")            |
| __totalAmt__    | _number_        | 전체 주문 수량                                        |
| __request_cb__  | _function(res)_ | 요청에 대한 콜백 함수                                   |

* 키 itemList가 가질 JSON 요소들의 형태는 다음과 같습니다.

| 키 이름  | 타입   | 설명                |
|----------|--------|---------------------|
| itemName | string | 상품명              |
| imgUrl   | string | 상품 이미지의 URL   |
| itemAmt  | string | 상품의 수량         |
| orderUrl | string | 각 상품의 웹 페이지 |


###### &nbsp; example
```javascript
INSTEAD.requestPay({
    name: "김스테드",
    tel: "010-1234-5678",
    merchantUid: "itsm_" + new Date().getTime(),
    post : "10154",
    addr: "경북 포항시 남구 대이로 63번길 6",
    email: "instead@instead.com",
    itemList:[
        {
            itemName: "슬레진저 2017 신상 남여공용 운동화/스니커즈, 겨울 신상 특가 상품",
            imgUrl: "http://img.hiphoper.com/images/lweb/item/AK/61/AK612621/htriple_bitten_muffler1.jpg",
            itemAmt: 1,
            orderUrl: "http://www.hiphoper.com/shop/item.php?it_id=AK612621&ca_id=BB70"
        },
        {
            itemName: "Shoes & Handbags",
            imgUrl: "http://img.hiphoper.com/images/lweb/item/AK/61/AK612621/htriple_bitten_muffler1.jpg",
            itemAmt: 1,
            orderUrl: "http://www.hiphoper.com/shop/item.php?it_id=AK612621&ca_id=BB70"
        },
        {
            itemName: "TRIPLE BITTEN MUFFLER",
            imgUrl: "http://img.hiphoper.com/images/lweb/item/AK/61/AK612621/htriple_bitten_muffler1.jpg",
            itemAmt: 1
            orderUrl: "http://www.hiphoper.com/shop/item.php?it_id=AK612621&ca_id=BB70"
        }
        ],
    payEndDttm: "2018-12-31 23:59:59",
    sandboxYn: "1",
    totalAmt:3,
    request_cb: function(res){
        console.log(res);
        document.getElementById("response").innerHTML = res.result+','+res.method;
    }
}); 
```


## 6. 결제 통보 받을 URL을 발급 받습니다.

```javascript
API = "https://instead.co.kr/api";
window.open(API + '/pay?session_id='+this.session_id,...);
```
##### URL은 session_id를 기반으로 구성되어 있으며 해당 session_id를 기반으로 구매정보를 가져옵니다.

## 7. 결제가 완료되면 쇼핑몰 측 서버에 결과를 전송합니다.

```javascript
function sendResult(requestNo, instead_uid, status){
    let get_info = `
        select  a.PAYMENT_URI 'payment_uri', b.MERCHANT_UID 'merchant_uid'  
        from    STORE a, REQUEST b 
        where   a.STORE_NO = b.STORE_NO 
                and b.REQUEST_NO = ?`;
    db.pool.query(get_info,[requestNo])
    .then(data=>{
        data = data[0];
        let options = {
            method: 'GET',
            uri: data.payment_uri+`?instead_uid=${instead_uid}&merchant_uid=${data.merchant_uid}&pay_status=${status}`,
            json: true
        }
        return rp(options);
    })
}
```


## 8. 결제 취소는 [관리자페이지](https://admin.instead.co.kr/)에서 할 수 있습니다.

# REST API
결제 후 정보를 확인하고 정상처리여부를 검증할 수 있도록 REST API를 제공하고 있습니다.

# 결제방식
##### 카드결제, 휴대폰결제, 실시간 계좌이체 <br>
위 세가지를 제공하고있습니다.
