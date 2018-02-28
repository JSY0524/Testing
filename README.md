# 인스테드 연동가이드

## 1. 인스테드 [관리자 페이지](http://insteadadmin.hssa.me)에서 가맹점등록

## 2. instead javascript import
```html
<script src="instead-guide/instead-1.0.0.js"></script>
```

## 3. client_id로 initialize 한 후에 새로운 인증 token을 발급 받습니다.
```javascript
instead.init('client_id');
```
<hr/>
```javascript
this.access_token = JSON.parse(req.responseText).access_token;
```

## 4. 아이템 정보 및 요청자 정보와 token을 server로 보내면 session_id를 발급 받습니다.
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
<hr/>
```javascript
req.setRequestHeader("Authorization", "Bearer" + this.access_token);
this.session_id = JSON.parse(req.responseText).session_id;
```

## 5. 결제 통보 받을 URL을 발급 받습니다.

# REST API
결제 후 정보를 확인하고 정상처리여부를 검증할 수 있도록 REST API를 제공하고 있습니다.

# 결제방식
카드결제, 휴대폰결제, 실시간 계좌이체
위 세가지를 제공하고있습니다.
