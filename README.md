# Mobile API Guideline in WePlanet


## REST API

리소스 단위로 주소체계를 사용하는 REST API를 사용을 기본으로 한다.
모든 리소스는 복수명사로 표현하되 특정 리스소를 요청할 경우 하위 주소로 식별자를 사용할 수 있다.

HTTP 메쏘드명을 사용하여 리소스의 처리 방법을 지정할 수 있다.
[9개 메쏘드](https://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol#Request_methods) 중 
CRUD(Crate, Read, Update, Delete)에 해당하는 POST, GET, PUT, DELETE 메소드만 사용한다.
메쏘드는 리소스명과 결합하여 최소 5개의 API로 제공된다.

* POST /users: 유저 생성
* GET /users: 유저 목록 조회
* GET /users/{id}: 유저 조회
* PUT /users/{id}: 유저 수정
* DELETE /users/{id}: 유저 삭제 


## Request

### QueryString

GET, DELETE 메쏘드는 쿼리스트링을 사용할 수 있다.
예를 들어 유저 목록을 조회하는 GET /users API의 경우 검색, 페이지네이션 등의 요청을 할 수 있는데
이 경우 쿼리스트링으로 파라매터를 지정할 수 있다. 
만약 쿼리스트링에 유니코드가 포함될 경우 주소 인코딩후 API 요청해야 한다.

유저 목록 중 특정 키워드로 검색한 결과를 요청할 경우 `query` 파라메터를 사용한다.

```
GET /users?query=keyword
```

유저 목록은 성능을 위해 페이지네이션이 필요한데 `limit`, `offset` 파라메터를 사용한다.

```
GET /users?limit=10&offset=20
```

데이터 전송량을 줄이기 위해 데이터베이스의 특정 컬럼만 요청할 경우 `filter` 파라메터를 사용한다.

```
GET /users?filter=name,email
```

위의 모든 파라매터는 `&`를 사용하여 AND 조합할 수 있다.

```
GET /users?query=keyword&limit=10&offset=20&filter=name,email
```


### Body

POST, PUT 메쏘드는 요청 바디를 사용한다. 
기본적으로 application/x-www-urlencoded를 사용하고 파일의 경우 multipart/form-data를 사용한다.
바디는 JSON 형식을 사용하고 객체나 배열을 포함할 수 있다.


## Response

### StatusCode

API 응답은 상태코드와 바디 두 부분으로 구성된다.
상태코드는 응답 성공과 실패에 관한 정보를 포함하고 바디에는 실제 데이터가 JSON 형식으로 전달된다.
 
사용하는 상태코드는 크게 세 분류

* 2XX: 성공
* 4XX: 요청 에러 
* 5XX: 서버 에러

2XX는 API 성공시 응답하는 코드다. 

GET, PUT 요청을 성공하면 200(Success) 코드를 응답한다. 
 
POST 성공시 리소스 생성을 의미하는 201(Created) 코드를 응답한다.

DELETE는 성공시 리소스가 없음을 의미하는 204(No Content)를 응답한다.

| 상태코드 | 의미        | 메쏘드 | 
| ------ | -------    | ---- | 
| 200    | Success    | GET, PUT |
| 201    | Created    | POST |
| 204    | No Content | DELETE |

4XX는 API 요청하는 클라이언트 측의 에러를 의미한다.

400은 파라메터 에러(Bad Request)를 의미하는데 필수 파라메터 누락, 파라메터 형식 오류 등의 경우 발생한다.

401은 인증이 필요한 API에 대해 인증되지 않은 접근일 경우 (Unauthorize) 발생한다.
엑세스 토큰이 잘못되었거나 엑세스 토큰의 유효기간이 지났을 경우 발생한다.

403(Forbidden)은 로그인 시도시 로그인 정보가 잘못되었을 경우이다. 

404는 리소스가 없는 경우(Not Found) 발생한다
 
409는 추가할 리소스 서버에 있는 리소스와 충돌(Conflict)할 경우 발생한다.
예를 들어 POST /users API로 신규 유저를 추가할 경우 이미 있는 유저(이메일이 유니크할 경우 중복 이메일 유저)일 
경우 유저 생성을 중단하고 409 에러코드를 응답한다.

| 상태코드 | 의미            | 메쏘드 | 
| ------ | -------        | ---- | 
| 400    | Bad Request    | GET, POST, PUT, DELETE |
| 401    | Unauthorized   | GET, POST, PUT, DELETE |
| 403    | Forbidden      | POST (로그인) |
| 404    | Not Found      | GET, POST, PUT, DELETE |
| 409    | Conflict       | POST |


5XX는 서버 에러를 의미한다.

대부분의 서버 에러는 500 코드를 응답한다.

Nginx + Node.js 구성시 노드 서비스가 종료되었을 경우 Nginx 측에서 503 코드를 응답한다.

| 상태코드 | 의미                   | 메쏘드 | 
| ------ | -------               | ---- | 
| 500    | Internal Server Error | GET, POST, PUT, DELETE |
| 503    | Service Unavailable   | GET, POST, PUT, DELETE |


### Body

2XX 상태코드는 응답 바디에 실제 전송할 데이터를 JSON 형식으로 전송한다.

목록을 조회할 경우 리소스명 키에 배열을 담아 응답한다. 

GET /users: 

```json
{
  "users": [
    { "id": 1 },
    { "id": 2 }
  ]
}
```

하나의 리소스를 요청할 경우 리소스명 키에 객체를 담아 응답한다.

GET /users/1: 

```json
{
  "user": { "id": 1 }
}
```

모바일 화면 구성에 따라 리소스를 추가하여 응답할 수 있다.

GET /users/1:

```json
{
  "user": { "id": 1 },
  "friends": [
    { "id": 3 },
    { "id": 5 },
  ]
}

```

### Error Handling

4XX, 5XX 상태코드 응답시 바디에 `error`키를 추가하여 JSON 객체로 응답한다.

```json
{
  "error": {
    "errorCode": "string",
    "message": "string"
  }
}
```

`errorCode`는 사전 정의된 유일한 에러코드이다.
모바일에서는 이 코드를 참고하여 에러메세지 팝업 등의 에러 처리 로직을 구현할 수 있다.

`message`는 디버깅용 메세지를 문자열로 담는다.
경우에 따라 서버측 에러 스택 정보를 추가할 수 있다.




