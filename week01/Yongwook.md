# 1장 웹 기술과 HTTP

> **1장에서 다루는 내용**
>
> - 웹 페이지가 브라우저에 로드되는 방식
> - HTTP가 무엇이고 HTTP/1.1까지 어떻게 진화했는지
> - HTTPS의 기초
> - 기본 HTTP 도구

## 1.1 웹의 동작 방식

대다수의 사람이 인터넷을 서핑하고자 웹 브라우저를 사용하는 방법을 알고 있기는 하지만, 소수만이 해당 기술이 어떻게 동작하는지, 왜 HTTP가 웹의 핵심인지, 또는 왜 다음 버전(HTTP/2)이 웹 커뮤니티에서 그 정도의 흥분을 불러일으키는지 이해한다.

### 1.1.1 인터넷과 월드 와이드 웹

많은 사람에게 인터넷과 월드 와이드 웹은 같은 뜻이지만, 두 용어를 구분 짓는 것은 중요하다.

인터넷은 인터넷 프로토콜 (IP, Internet Protocol) 을 공유해 메세지를 라우팅하는 식으로 연결된 공용 컴퓨터의 모음이다. 인터넷은 월드 와이드 웹, 이메일, 파일 공유, 인터넷 전화 등의 여러 서비스로 구성된다.

따라서 월드 와이드 웹이 인터넷에서 가장 잘 보이는 부분이긴 하지만, 그저 일부분일 뿐이다.

![internet vs www](https://user-images.githubusercontent.com/46441280/159440794-f4a76676-2553-41d1-b402-e8fc7a7d0a32.png)

HTTP는 웹 브라우저가 웹 페이지를 요청하는 방식이다.

HTTP를 살펴볼 때 주로 월드 와이드 웹을 다루게 된다. 그러나 전통적인 웹 프론트엔드 없이도 서비스들이 HTTP 기반으로 구축되기 때문에 경계선은 더욱 흐려지고 있고, 이는 웹 자체를 정의하는 것이 더욱 까다로워지고 있음을 의미한다. (IoT같은 예가 있다.)

이 모든 것은 월드 와이드 웹이라는 용어가 올바르지 않게 인터넷과 자주 혼용돼 사용되더라도 크게 어긋나지 않는다는 것을 나타낸다.

### 1.1.2 웹을 돌아다닐 때 일어나는 일

이 책을 최대한 활용하려면 웹 탐색이 동작하는 방식을 정확히 이해해야 한다.

브라우저를 작동시켜 [www.google.com](https://www.google.com) 으로 이동한다고 하자. 몇 초만에 다음 그림과 같은 일이 일어날 것이다.

![web-interactions](https://user-images.githubusercontent.com/46441280/159444134-b2c6a44b-ff98-43f8-a352-f165c990a067.png)

1. 브라우저는 www.google.com의 실제 주소를 도메인 이름 시스템 (DNS, Domain Name System) 서버로 요청한다. DNS 서버는 이를 IP 주소로 바꿔준다.
IP 주소를  DNS에 요청할 때 인터넷 탐색을 빠르게 만들기 위해 DNS는 가장 가까운 서버의 IP 주소를 제공한다.

2. 웹 브라우저가 웹 서버에 이 주소로 80포트 또는 443포트에 IP를 통한 (TCP, Transmission Control Protocol) 연결을 요청한다.
IP는 인터넷을 통해 트래픽을 보내는 데 사용되지만, TCP는 연결을 안정적으로 만드는 안정성과 재전송을 추가해준다.

3. 브라우저가 웹 서버와 연결을 맺고 있으면 웹 사이트를 요청하기 시작할 수 있다. 이 단계에서 HTTP가 관여한다.

4. 구글 서버가 요청받은 URL에 응답한다. 일반적으로 첫 페이지에서 돌려받는 것은 HTML 형식의 텍스트지만, 구글은 https로만 운영되기 때문에 <https://www.google.com> 으로 리디렉션하는 특수한 HTTP 명령을 반환한다.
뭔가 잘못된 경우에는 HTTP 응답 코드를 돌려받는데, 대표적인 코드가 `404 Not Found`다

5. 웹 브라우저가 반환된 응답을 처리한다. HTML을 받았다면 코드 해석을 시작하고 메모리에 해당 페이지의 내부 표현인 DOM (Document Object Model)을 구축한다.

6. 웹 브라우저가 추가로 필요한 리소스를 요청한다. 구글은 상당히 간결하게 웹 페이지를 유지한다. 구글의 경우 이 단계에서 16개의 리소스만 요청하는데, 이러한 리소스 각각이 1 ~ 6 단계를 따라 유사한 방식으로 요청된다.

7. 브라우저가 중요한 리소스를 충분히 얻으면 화면에 페이지를 그리기 시작한다.

8. 페이지를 처음 표시한 후 웹 브라우저는 백그라운드에서 페이지에 필요한 다른 리소스들을 계속 다운로드하고 처리하는 대로 페이지를 업데이트한다.

9. 완전히 로드되면 브라우저는 로딩아이콘을 멈추고, 자바스크립트 코드에서 페이지가 어떤 동작을 수행할 준비가 됐다는 표시로 사용할 수 있는 `OnLoad` 이벤트를 발생시킨다.

10. 이 후로도 계속 필요한 리소스를 요청하고 받는다. 페이스북 피드가 새로고침 버튼 없이도 업데이트 되는 걸 생각하면 이해하기 쉽다.

## 1.2 HTTP란 무엇인가?

앞에서 언급한 것 처럼 HTTP는 하이퍼텍스트 전송 프로토콜을 의미한다.

이름처럼 처음에는 하이퍼텍스트 문서 (다른 문서로의 링크를 담은 문서)를 전송할 의도로 만들어 졌다.

하지만 개발자들이 이 프로토콜을 다른 문서 유형을 전송하는 데 사용될 수 있음을 곧 깨달았고, HTTP라는 약자가 적절하지는 않지만 바꾸기에는 너무 늦게 된 것이다.

HTTP는 TCP/IP가 제공하는 안정적인 네트워크 연결에 의존하지만, 네트워크 연결이 설정되는 방법의 하부 수준 세부 사항을 다루지는 않는다.

HTTP 애플리케이션이 네트워크 오류와 연결 종료를 어떻게 다룰지 염두에 둬야 하지만 프로토콜 자체는 이러한 작업까지 감안하지는 않는다.

다음 그림은 이 모델이 웹 트래픽에 연결된 방식과 HTTP가 이 모델의 어디에 들어맞는지를 대략적으로 보여준다.

![osi-http](https://user-images.githubusercontent.com/46441280/159450545-1d721e6a-52a7-4113-8159-90809850aefc.png)

HTTP는 본질적으로 요청 및 응답 프로토콜이다. 웹 브라우저가 HTTP 문법을 사용해 요청을 만들고 웹 서버로 보내면 웹 서버는 요청받은 리소스를 포함한 메시지로 응답한다.

연결을 맺은 다음 HTTP 요청의 기본 문법은 다음과 같다.

```http
GET /page.html↵
```

↵ 기호는 줄 바꿈 문자를 나타낸다.

기본 형식에서 HTTP는 이정도로 단순하다. HTTP 메서드 중 하나를 주고, 그 다음 원하는 리소스를 넣는다.

이 지점에서 기억할 것은 이미 TCP/IP와 같은 기술을 사용해 적절한 서버에 연결됐고, 단순히 그 서버에서 원하는 리소스를 요청하는 것이기 때문에 연결이 어떻게 일어나거나 유지되는지는 관계할 필요가 없다는 것이다.

맥이라면 다음과 같은 방식으로 HTTP 요청을 날려볼 수 있다.

```bash
nc www.google.com 80
# Connection to www.google.com port 80 [tcp/http] succeeded!
```

앞서 언급한 것처럼 HTTP의 성공 비결은 서비스 수준에서 비교적 HTTP를 구현하기 쉽게 만들어준 간결함이다.

또한 HTTP가 단순하기 때문에 애플리케이션이 수많은 독립적인 웹 서비스로 쪼개지는 마이크로 서비스 아키텍처 스타일의 대유행이 이어졌다.

## 1.3 HTTP의 문법과 역사

### 1.3.1 HTTP/0.9

처음으로 공개된 HTTP의 사양은 1991년에 공표된 버전 0.9다.

이 사양의 길이는 700 단어 미만으로 짧다.

- GET, 문서 주소(공백 문자 없이), 캐리지 리턴`CR`과 라인 피드`LF`(캐리지 리턴은 선택 사항)로 구성된 ASCII 텍스트 한줄이 전송돼야 한다.
- 서버는 HTML 형식의 메시지로 응답해야 하는데, 사양에는 `ASCII 문자로 된 바이트 스트림`으로 정의한다.
- 메시지는 서버의 연결종료로 끝나야 한다.
- 그리고 오류 응답은 HTML 문법으로 된 사람이 읽을 수 있는 텍스트로 제공돼야 한다. 텍스트 내용 외에는 오류 응답을 성공적인 응답과 구분할 방법은 없다.
- 요청은 idempotent이다. (여러 번 적용하더라도 결과가 달라지지 않는다.) 서버는 연결 종료 후 요청에 대한 어떤 정보도 저장할 필요가 없다.

다음은 HTTP/0.9에서 가능한 유일한 명령이다.

```http
GET /section/page.html↵
```

### 1.3.2 HTTP/1.0

월드와이드웹은 거의 즉시 성공해버렸다.

대다수의 웹 서버는 벌써 0.9 사양을 훨씬 넘어선 확장을 구현했다.

1996년 5월 HTTP/1.0이 RFC 1945로 발표됐으나, 공식사양은 아니다.

이 문서의 최상단에는 다음과 같이 서술하고 있다.

> 이 메모는 인터넷 공동체를 위해 정보를 제공한다. 이 메모는 어떤 종류의 인터넷 표준도 명시하지 않는다.

RFC의 공식 상태와 관계 없이 HTTP/1.0은 다음과 같은 몇 가지 주요 기능을 추가했다.

- **더 많은 요청 메서드**: `HEAD`와 `POST`가 추가됐다.
- **선택적인 HTTP 버전 번호가 모든 메시지에 추가됨**: 이전 버전과의 호환성을 쉽게 제공하고자 Default는 HTTP/0.9로 설정됐다.
- **요청 및 응답 모두에 같이 보내질 수 있는 HTTP 헤더**: 요청 받은 리소스와 전송하는 응답에 대한 더 많은 정보를 제공한다.
- **응답이 성공적인지 등의 정보를 표시하는 3자리 응답코드**: 이 코드는 리디렉션 요청, 조건부 요청, 오류 상태를 가능하게 했다.

사실 HTTP/1.0은 새로운 기능을 정의했다기보다는 이미 일어난 일을 문서화하기 위한 것이었다.

#### HTTP/1.0 메서드

`GET` 메서드는 HTTP/0.9와 거의 동일하게 남기는 했지만 헤더의 추가는 조건적인 GET을 가능하게 했다. 앞서 언급한 것처럼 사용자는 하이퍼텍스트 문서보다 더 많은 것을 GET 메서드로 얻을 수 있고, 그 외의 어떤 종류의 미디어를 다운로드하는 데에도 HTTP를 사용할 수 있다.

`HEAD` 메서드는 리소스 자체를 다운로드받지 않고도 클라이언트가 리소스에 대한 모든 메타정보를 얻을 수 있도록 허용한다.

`POST` 메서드는 클라이언트가 웹 서버에 데이터를 전송할 수 있게 해준다. POST 메서드는 콘텐츠가 HTTP 요청의 요소로서 클라이언트에서 서버로 보내지게 하며, 처음으로 HTTP 요청이 HTTP 응답과 같이 본문을 가질 수 있음을 나타낸다.

#### HTTP 요청 헤더

HTTP/1.0은 HTTP 헤더를 도입했다. HTTP GET 요청은 다음과 같은 형식에서

```http
GET /page.html↵
```

다음과 같이 변경됐다.

```http
GET /page.html HTTP/1.0↵
Header1: Value1↵
Header2: Value2↵
↵
```

또는 헤더 없이 다음과 같이 사용할 수도 있다.

```http
GET /page.html HTTP/1.0↵
↵
```

HTTP/1.0이 표준 헤더를 몇 가지 정의했지만, 사용자 정의 헤더를 허용한다는 것도 보여준다.

사양에는

> 수신자가 이러한 필드를 인지할 수 있다고 가정할 수 없으며, 무시할 수 있는 반면, 표준 헤더는 HTTP/1.0 호환 서버에서 반드시 처리돼야 한다.

고 명시적으로 서술했다.

전형적인 HTTP/1.0 GET 요청은 다음과 같다.

```http
GET /page.html HTTP/1.0↵
Accept: text/html,application/xhtml+xml,image/jxr/,*/*↵
Accept-Encoding: gzip, deflate, br↵
Accept-Language: en-GB,en-US;q=0.8,en;q=0.6↵
Connection: keep-alive↵
Host: www.example.com↵
User-agent: MyAwesomeWebBrowser 1.1↵
↵
```

#### HTTP 응답 코드

HTTP/1.0 서버의 전형적인 응답은 다음과 같다.

```http
A typical response from an HTTP/1.0 sever is
HTTP/1.0 200 OK
Date: Sun, 25 Jun 2017 13:30:24 GMT
Content-Type: text/html
Server: Apache

<!doctype html>
<html>
<head>
...etc.
```

다음 표는 HTTP/1.0 사양에 정의된 HTTP 응답 코드를 보여준다.
[여기](https://www.w3.org/Protocols/HTTP/1.0/spec.html#Status-Codes)서 자세히 볼 수 있다.

| 카테고리 | 값 | 코드 기술 | 상세 설명 |
| -- | -- | --| -- |
|1xx|N/A|N/A|HTTP/1.0은 어떤 1xx 상태코드도 정의하지 않지만 카테고리는 정의한다 |
|2xx(Successful)|200|OK|이 코드는 성공적인 요청에 대한 표준 응답 코드다.|
| |201|Created|이 코드는 POST 요청에 대해 반환해야 한다. |
| |202|Accepted|이 요청은 처리되고 있지만 아직 처리가 완료되지 않았다.|
| |204|No Content|이 요청은 수락되고 처리됐지만 반환할 어떤 BODY 응답도 없다.|
|3xx(Redirection)|300|multiple choices|이 코드는 직접적으로 사용되지 않는다.|
| |301|Moved permanently|`Location` HTTP 응답 헤더는 리소스의 새로운 URL을 제공해야 한다.|
| |302|Moved temporarily| |
| |304|Not modified| 이 코드는 BODY가 다시 전송될 필요가 없다는 조건적 응답으로 사용된다. |
|4xx(Client Error)|400|Bad request|요청이 해석될 수 없었으며 재전송되기 전에 변경돼야만 한다|
| |401|Unauthorized|이 코드는 보통 인증되지 않았음을 의미한다|
| |403|Forbidden|이 코드는 보통 인증이 됐지만 접근 권한이 없음을 의미한다|
| |404|Not found| |
|5xx(Server Error)|500|Internal server error|요청이 서버 측 오류로 완료될 수 없다|
| |501|Not Implemented|서버가 요청을 인지하지 못했다.|
| |502|Bad gateway|서버가 게이트웨이나 프록시의 역할을 하며 다운스트림 서버에서 오류를 수신했다|
| |503|Service unavailable|서버의 부하가 지나쳤거나 유지 보수를 위해 내려가 있어서 요청을 이행할 수 없었다|

#### HTTP 응답 헤더

첫 응답 줄 다음에는 HTTP/1 헤더 응답 줄이 0줄 이상 있다. 요청 헤더와 응답 헤더는 동일한 형식을 따른다.

헤더 다음에는 두 개의 리턴 문자가 있고, 그 다음에는 본문 내용이 있다.

```http
GET /
HTTP/1.0 302 Found
Location: http://www.google.ie/?gws_rd=cr&dcr=0&ei=BWe1WYrf123456qpIbwDg
Cache-Control: private
Content-Type: text/html; charset=UTF-8
Date: Sun, 25 Jun 2017 16:23:33 GMT
Server: gws
Content-Length: 268
X-XSS-Protection: 1; mode=block
X-Frame-Options: SAMEORIGIN

<HTML><HEAD>
...etc.
```

### 1.3.3 HTTP/1.1

HTTP/1.1은 HTTP 프로토콜을 최적화해 사용할 수 있도록 몇 가지 사항(지속적 연결, 필수 서버 헤더, 개선된 캐싱 옵션, 청크 인코딩 등)을 더 개선했다.

HTTP/1.1을 설명하려면 자체로 책 하나가 필요하겠지만, 여기에서는 이 책의 뒷 부분에 나올 HTTP/2의 논의에 대한 배경과 맥락을 제공하는 데 필요한 요점을 다루려고 한다.

#### 필수적인 호스트 헤더

HTTP 요청 행에서 제공된 URL은 절대 URL이 아니라 상대 URL이다. HTTP가 생성됐을 때는 하나의 웹 서버가 하나의 웹 사이트만 호스팅한다고 가정했다. 그러나 오늘날 많은 웹 서버는 여러 사이트를 동일한 서버에서 호스팅한다.

따라서 이 기능은 요청에 Host 헤더를 추가하는 방식으로 구현됐다.

```http
GET / HTTP/1.1
Host: www.google.com
```

이 헤더는 HTTP/1.0에서는 선택적이었지만 HTTP/1.1에서는 필수적이다.

#### 지속적인 연결

처음에 HTTP은 단일 요청 및 응답 프로토콜이었다. 클라이언트가 연결을 맺고, 리소스를 요청하고, 응답을 받으면 연결이 종료됐다.

웹에 미디어가 더 풍부해짐에 따라 이 연결 종료는 낭비로 밝혀졌다.

이 기능은 새로운 Connection 헤더로 해결됐다.

이 헤더에 Keep-Alive를 지정함으로써 클라이언트는 서버에 추가적인 요청 전송을 허용하고자 연결을 맺은 채로 그대로 두라고 요청한다.

```http
GET /page.html HTTP/1.0
Connection: Keep-Alive
```

서버는 평상시처럼 응답하지만 지속적인 연결을 지원하는 경우 `Connection: Keep-Alive` 헤더를 응답에 포함한다.

```http
HTTP/1.0 200 OK
Date: Sun, 25 Jun 2019 13:30:24 GMT
Connection: Keep-Alive
Content-Type: text/html
Content-Length: 12345
Server: Apache

<!doctype html>
<html>
<head>
...etc.
```

연결 종료가 없어지면서 언제 응답이 완료됐는지 알기가 어려워졌으며, 응답 본문의 길이를 정의하고자 Content-Length HTTP 헤더를 사용해야 한다.

HTTP/1.1은 이 절차를 표준에 추가한 데 그치지 않고 더 나아가 이 절차를 기본값으로 변경했다.

어떤 HTTP/1.1 연결이든 `Connection: Keep-Alive` 헤더가 응답에 없더라도 지속적인 연결을 사용 중이라고 가정할 수 있다.

어떤 이유로든 연결을 종료하고 싶다면 `Connection: Close` 헤더를 응답에 포함시켜야 한다.

#### 기타 새로운 기능

HTTP/1.1은 다음을 포함하는 다른 많은 기능을 도입했다.

- HTTP/1.0에서 정의된 GET, POST, HEAD에 새로운 메서드 PUT, OPTIONS와 잘 사용되지는 않는 CONNECT, TRACE, DELETE가 추가됐다.
- 개선된 캐싱 방법을 도입해 서버가 리소스를 브라우저의 캐시에 저장해서 필요하면 나중에 재사용할 수 있도록 클라이언트에 지시할 수 있다. Cache-Control HTTP 헤더는 HTTP/1.1에 HTTP/1.0의 Expires 헤더보다 많은 옵션을 도입했다.
- HTTP가 상태 없는 프로토콜에서 상태를 가질 수 있게 하는 HTTP 쿠키를 도입했다.
- HTTP 응답에서의 문자 집합과 언어를 도입했다.
- 프록시 지원 기능을 도입했다.
- 인증 기능을 도입했다.
- 새로운 상태 코드를 도입했다.
- 후행 헤더를 도입했다.

## 1.4 HTTPS 개론

HTTP는 원래 일반 텍스트 프로토콜이었다. HTTP 메시지는 인터넷을 통해 암호화되지 않은 상태로 전송됐으며, 따라서 메시지가 목적지로 라우팅될 때 메시지를 보는 모든 참여자가 읽을 수 있다.

HTTPS는 전송중의 메시지를 전송 계층 보안 (TLS, Transport Layer Security) 프로토콜을 사용해 암호화하는 HTTP의 보안 버전이다.

HTTPS는 HTTP 메시지에 다음과 같은 세 가지 중요한 개념을 추가했다.

- **암호화**: 메시지는 전송 중에 제3자에게 읽힐 수 없다.
- **무결성**: 암호화된 메시지가 디지털 서명되고 서명이 복호화되기 전에 암호학적으로 검증되기 때문에 메시지는 전송 중에 변경되지 않는다.
- **인증**: 서버는 클라이언트가 메시지를 주고받으려던 바로 그 서버다.

HTTPS는 공개키 암호화를 사용해 동작하므로 사용자가 처음 연결할 때 서버가 디지털 인증서 형태의 공개키를 제공할 수 있다.

브라우저는 이 공개키를 사용해 메시지를 암호화한다.

짝이 되는 개인키는 서버만이 가졌으므로 서버에서만 메시지를 복호화할 수 있다.

HTTPS는 HTTP를 기반으로 하며 HTTP 프로토콜 자체와 거의 원활하게 이어진다.

기본값으로 다른 포트에서 호스팅되며 다른 URL 체계를 갖지만 암호화와 복호화를 제외하고는 HTTP가 사용되는 방식과 같다.

클라이언트가 HTTPS 서버에 접속하면 협상 단계(TLS 핸드셰이크)를 거치게 된다.

이 과정에서 서버는 공개키를 제공하고, 클라이언트와 서버는 사용할 암호화 방식에 합의하고, 그 다음 클라이언트와 서버가 미래에 사용할 공유 암호화 키를 협상한다.

## 1.5 HTTP 메시지를 보고 보내고 받는 용도로 쓰이는 도구

커맨드라인 도구는 한계가 있으며, 그 중 무엇도 대다수의 대규모 웹 페이지를 다루지 못한다.

텔넷보다 나은 방식으로 HTTP 요청을 보고 보낼 수 있는 여러 도구가 있다.

### 1.5.1 웹 브라우저의 개발자 도구 사용

개발자 도구를 열고 페이지를 로드하면 네트워크 탭은 모든 HTTP 요청을 보여주고, 그 중 하나를 클릭하면 요청 및 응답 헤더를 포함한 상세한 정보를 생성한다.

HTTPS를 브라우저가 다루기 때문에 개발자 도구는 암호화되기 전의 HTTP 요청 메시지와 복호화된 후의 응답 메시지만 보여준다.

대부분의 경우 암호화와 복호화를 처리할 수 있는 적절한 도구가 있다면 HTTPS를 설정한 다음에 무시할 수 있다.

### 1.5.2 HTTP 요청 전송

Advanced REST client 애플리케이션은 가공하지 않은 HTTP 메시지를 보내고 응답을 보는 방법을 제공한다.

Postman, Rested, RESTClient, RESTMan 등의 유사한 애플리케이션들은 모두 비슷한 기능을 갖고 있다.

### 1.5.3 HTTP 요청을 보고 전송하기 위한 기타 도구

브라우저 밖에서도 많은 도구를 사용할 수 있다.

일부는 커맨드라인(curl, wget, httpie)에서 동작하고 일부는 데스크톱 클라이언트(SOAP-UI)와 동작한다.

## 요약

- HTTP는 웹의 핵심 기술 중 하나다.
- 브라우저는 웹 페이지 하나를 로드하고자 여러 개의 HTTP 요청을 생성한다.
- HTTP 프로토콜은 단순한 텍스트 기반 프로토콜로 출발했다.
- HTTP는 더욱 복잡해졌지만, 기본적인 텍스트 기반 형식은 지난 20년동안 변경되지 않았다.
- HTTPS는 표준 HTTP 메시지를 암호화한다.
- HTTP 메시지를 보고 전송하는 데 다양한 도구를 사용할 수 있다.
