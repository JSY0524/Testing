# 인스테드 연동가이드

## 1. 인스테드 [관리자 페이지](http://insteadadmin.hssa.me)에서 가맹점등록

## 2. instead javascript import
#### instead.co.kr 서버로 연결하기 위해 아래의 script를 입력합니다.
```html
<script src="instead-guide/instead-1.0.0.js"></script>
```

## 3. client_id로 initialize 한 후에 새로운 인증 token을 발급 받습니다.
```javascript
instead.init('client_id');
```
#### instead.init의 원리
<hr/>
```javascript
window.INSTEAD.prototype.init = function(client_id) {
    var req = new XMLHttpRequest();
    req.open("POST", API + "/token", true);
    req.setRequestHeader("Content-Type", "application/json");
    var self = this;
    req.onreadystatechange = function() {
        if (req.readyState == 4 && req.status == 200) {
            self.access_token = JSON.parse(req.responseText).access_token;
        }
    };
    req.send(JSON.stringify({
        "client_id": client_id
    }));
}
```
## 4. 아이템 정보 및 요청자 정보 등을 포함하여 요청하면 끝납니다.
```javascript
instead.requestPay({ 
    name: "홍길동", 
    tel: "010-0000-0000", 
    merchantUid: "instead_guid_"+new Date().getTime(), 
    post : "10000", 
    addr: "서울특별시 인스테드", 
    email: "instead@instead-corp.com", 
    itemList:[ 
        { 
            itemName: "운동화", 
            imgUrl: "http://instead-corp.com/shoes.jpg", 
            itemAmt: 1000, 
            orderUrl: "http://shop.hssa.me" 
        } 
    ], 
    payEndDttm: 150135478, 
    sandboxYn: '1', 
    request_cb: function(res) { console.log(res); } 
});
```

# REST API
결제 후 정보를 확인하고 정상처리여부를 검증할 수 있도록 REST API를 제공하고 있습니다.

# 결제방식
카드결제, 휴대폰결제, 실시간 계좌이체
위 세가지를 제공하고있습니다.
