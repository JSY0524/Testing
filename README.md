# 인스테드 연동가이드

### 결제대행 서비스 인스테드를 연동하는 방법입니다.

## 연동하기에 앞서서
##### “http://”, “ftp://”, “market://”과 같은 문자열을 url scheme이라 부릅니다. url scheme을 통해 앱이 실행되는 방식은 다음과 같습니다.

##### - 웹페이지에서 하이퍼링크 클릭 시 url scheme이 system에 전달됨
##### - system에서 전달된 url scheme을 보고 실행 가능한 앱이 있는지 확인
##### - 해당 url scheme을 받을 수 있는 앱이 있다면 앱을 실행시키며 이 url을 함께 전달
##### - 앱이 실행되면서 url에 포함된 내용을 참조해서 특정 기능을 수행함

## 1. 인스테드 [관리자 페이지](http://insteadadmin.hssa.me)에서 가맹점등록

## 2. instead javascript import 하기

### 2-1. 아래의 script를 import 해야합니다.
```html
<script src="instead-guide/instead-1.0.0.js"></script>
```

### 2-2. client_id로 initialize 한 후에 새로운 인증 token을 발급 받습니다.
```javascript
instead.init('client_id');
```
<hr/>

##### <instead-1.0.0.js>

```javascript
var req = new XMLHttpRequest();
this.access_token = null;
this.access_token = JSON.parse(req.responseText).access_token;
```

## 3. 아이템 정보 및 요청자 정보와 token을 server로 보내면 session_id를 발급 받습니다.

##### Ex) '홍길동' 구매자가 'http://shop.hssa.me' 에서 아래의 정보에 따라 운동화를 주문 했을 때의 저장되는 data 내용입니다.

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

## 4. 결제 통보 받을 URL을 발급 받습니다.

##### <instead-1.0.0.js>

```javascript
API = "https://instead.co.kr/api";
window.open(API + '/pay?session_id='+this.session_id,...);
```
##### URL은 session_id를 기반으로 구성되어 있으며 해당 session_id를 기반으로 구매정보를 가져옵니다.

# REST API
결제 후 정보를 확인하고 정상처리여부를 검증할 수 있도록 REST API를 제공하고 있습니다.

# 결제방식
카드결제, 휴대폰결제, 실시간 계좌이체
위 세가지를 제공하고있습니다.
