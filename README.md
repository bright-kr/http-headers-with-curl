# Sending HTTP Headers with cURL

[![Bright Data Promo](https://github.com/bright-kr/LinkedIn-Scraper/raw/main/Proxies%20and%20scrapers%20GitHub%20bonus%20banner.png)](https://brightdata.co.kr/)

이 가이드는 HTTP 헤더를 cURL과 함께 효과적으로 사용하여 데이터 수집 및 서버 통신 역량을 향상시키는 방법을 안내합니다:

- [Understanding HTTP Headers](#understanding-http-headers)
- [Getting Started with cURL Headers](#getting-started-with-curl-headers)
- [Viewing Default cURL Headers](#viewing-default-curl-headers)
- [Modifying Default Headers with -H](#modifying-default-headers-with--h)
- [Creating Custom Headers](#creating-custom-headers)
- [Working with Empty Headers](#working-with-empty-headers)
- [Deleting Headers](#deleting-headers)
- [Sending Multiple Headers at Once](#sending-multiple-headers-at-once)
- [Summary](#summary)

## Understanding HTTP Headers

하이퍼텍스트 전송 프로토콜(HTTP)은 [stateless protocol](https://byjus.com/gate/difference-between-stateless-and-stateful-protocol/)로서 클라이언트가 리クエスト를 발행하고 서버 リスポンス를 기다리는 클라이언트-서버 아키텍처를 따릅니다. 이러한 리クエスト에는 HTTP 메서드, 서버 위치, 경로, 쿼리 파라メータ, 그리고 헤ッダー와 같은 중요한 요소가 포함됩니다.

HTTP 헤ッダー는 본질적으로 클라이언트와 서버 간에 메타데이터와 지침을 전달하는 키-값 쌍입니다. 이는 콘텐츠 유형, 캐싱 규칙, 인증 방법과 같은 파라メータ를 지정하는 데 중요한 역할을 하며, 원활하고 안전한 클라이언트-서버 상호작용을 보장합니다. [web scraping](https://brightdata.co.kr/blog/how-tos/what-is-web-scraping) 작업에서 HTTP 헤ッダー는 다양한 user agent를 시뮬레이션하고, 콘텐츠 네고시에이션을 관리하며, 웹사이트 요구사항 및 프로토콜에 따라 인증을 처리함으로써 리クエスト를 맞춤화할 수 있게 해줍니다.

Web스크레이핑에서 HTTP 헤ッダー의 일반적인 활용 사례로는 user-agent(UA) 변경, リスポンス 형식 지정, 조건부 리クエスト 수행, 애플리케이션 프로그래밍 인터페이스(API)로 인증 수행 등이 있습니다.

## Getting Started with cURL Headers

이 튜토리얼을 진행하기 전에, 터미널에서 다음 명령을 실행하여 시스템에 curl이 설치되어 있는지 확인하십시오:

```sh
curl --version
```

올바르게 설치되어 있다면 다음과 같은 버전 정보를 받게 됩니다:

```
curl 7.55.1 (Windows) libcurl/7.55.1 WinSSL
Release-Date: [unreleased]
Protocols: dict file ftp ftps http https imap imaps pop3 pop3s smtp smtps telnet tftp
Features: AsynchDNS IPv6 Largefile SSPI Kerberos SPNEGO NTLM SSL
```

`curl is not recognized as an internal or external command, operable program or batch file` 또는 `command not found`와 같은 오류가 발생하면, [install curl](https://curl.se/download.html)해야 합니다.

또한 헤ッダー 확인을 위한 서비스가 필요합니다. 예를 들어 [httpbin.org](http://httpbin.org/)는 간단한 HTTP 리クエスト 및 リスポンス 서비스를 제공합니다.

curl에 익숙하신 분이라면, 기본 구문이 다음 패턴을 따른다는 것을 알고 계실 것입니다:

```sh
curl [options] [url]
```

즉, `mywebpage.com`에서 콘텐츠를 가져오려면 다음을 실행합니다:

```sh
curl www.mywebpage.com
```

## Viewing Default cURL Headers

httpbin.org를 사용하여 curl이 기본적으로 보내는 헤ッダー를 확인하려면 다음 명령을 실행하십시오:

```sh
curl http://httpbin.org/headers
```

リスポンス에는 전송된 헤ッダー가 표시됩니다:

```json
{
  "headers": {
    "Accept": "*/*",
    "Host": "httpbin.org",
    "User-Agent": "curl/7.55.1",
    "X-Amzn-Trace-Id": "Root=1-65fd2eb0-0617353714d52f3777c9c267"
  }
```

`Accept`, `Host`, `User-Agent` 헤ッダー는 curl 요청에 기본으로 포함됩니다.

`Accept` 헤ッダー는 클라이언트가 처리할 수 있는 미디어 유형을 서버에 알립니다. 클라이언트가 수용할 콘텐츠 유형을 전달하여 클라이언트와 서버 간 콘텐츠 네고시에이션을 용이하게 합니다.

클라이언트가 JSON을 선호함을 나타내는 `Accept` 헤ッダー는 다음과 같습니다:

```
Accept: application/json
```

`User-Agent` 필드에는 클라이언트 정보가 포함되며, 이 경우 버전 번호가 포함된 curl 애플리케이션(설치된 버전과 일치)입니다.

`Host` 헤ッダー는 HTTP 리クエ스트의 특정 웹 도메인(호스트)과 포트 번호를 식별합니다. 포트가 지정되지 않으면 기본 포트가 가정됩니다(HTTP는 포트 80, HTTPS는 포트 443).

`X-Amzn-Trace-Id`는 기본 curl 헤ッダー가 아니라, 리クエ스트가 AWS 로드 밸런서와 같은 Amazon Web Services(AWS) 서비스를 통해 라우팅되었음을 나타내며 HTTP 리クエ스트 트레이싱에 사용할 수 있습니다.

curl이 기본적으로 보내는 헤ッダー가 무엇인지 확인하려면 `-v` 또는 `--verbose` 플래그로 verbose 모드를 사용할 수 있습니다. 이 모드는 헤ッダー सहित 상세한 리クエ스트 및 リスポンス 정보를 표시합니다.

기본 curl 헤ッダー를 보려면 다음 명령을 실행하십시오:

```sh
curl -v http://httpbin.org/headers
```

출력은 다음과 유사합니다:

```
- Trying 50.16.63.240...
* TCP_NODELAY set
* Connected to httpbin.org (50.16.63.240) port 80 (#0)
> GET /headers HTTP/1.1
> Host: httpbin.org
> User-Agent: curl/7.55.1
> Accept: */*
>
< HTTP/1.1 200 OK
< Date: Fri, 22 Mar 2024 07:18:00 GMT
< Content-Type: application/json
< Content-Length: 173
< Connection: keep-alive
< Server: gunicorn/19.9.0
< Access-Control-Allow-Origin: *
< Access-Control-Allow-Credentials: true
<
{
  "headers": {
    "Accept": "*/*",
    "Host": "httpbin.org",
    "User-Agent": "curl/7.55.1",
    "X-Amzn-Trace-Id": "Root=1-65fd30a8-624365ad52781957578cd5b1"
  }
}
* Connection #0 to host httpbin.org left intact
```

부등호(>)로 시작하는 줄은 클라이언트(curl)가 엔드포인트로 전송한 내용을 보여주며, 다음 헤ッダー가 전송되었음을 확인합니다:

- 엔드포인트/headers로의 `GET`(HTTP 메서드)
- 값이 `httpbin.org`인 `Host`
- 값이 `curl/7.55.1`인 `User-Agent`
- 값이 `*/*`인 `Accept`

출력에서 `< Content-Type: application/json`과 같이 부등호(<)로 시작하는 줄은 リスポンス 헤ッダー를 반영합니다.

## Modifying Default Headers with -H

`-H` 또는 `--header` 플래그는 서버로 커스텀 헤ッダー를 전송할 수 있게 해주며, 테스트에 유용합니다.

예를 들어 `User-Agent`를 `curl/7.55.1`에서 `Your-New-User-Agent`로 변경하려면 다음을 사용하십시오:

```sh
curl -H "User-Agent: Your-New-User-Agent" http://httpbin.org/headers
```

リスポンス에는 다음이 표시됩니다:

```json
{
  "headers": {
    "Accept": "*/*",
    "Host": "httpbin.org",
    "User-Agent": "Your-New-User-Agent",
    "X-Amzn-Trace-Id": "Root=1-65fd5123-3ebe566a4681427c6996c72c"
  }
}
```

`Accept` 헤ッダー를 `*/*`(어떤 콘텐츠 유형이든 수용)에서 `application/json`(JSON 콘텐츠만 수용)으로 수정하려면 다음을 실행하십시오:

```sh
curl --header "Accept: application/json" http://httpbin.org/headers
```

출력은 다음과 같습니다:

```json
{
  "headers": {
    "Accept": "application/json",
    "Host": "httpbin.org",
    "User-Agent": "curl/7.55.1",
    "X-Amzn-Trace-Id": "Root=1-65fd55c3-05c21f81770c1c5e6343b1fc"
  }
}
```

> **Note:**
>
> 이 예에서는 `-H` 대신 `--header`를 사용했습니다. 이 플래그들은 동등하며 동일한 기능을 수행합니다.

curl 7.55.0 버전부터는 헤ッダー가 포함된 파일을 사용할 수도 있습니다. 헤ッダー 파일 이름이 `header_file`인 경우 다음을 사용할 수 있습니다:

```sh
curl -H @header_file
```

## Creating Custom Headers

커스텀 헤ッダー는 표준 HTTP 헤ッダー 외에 추가 정보를 제공하는 개발자 정의 필드입니다.

curl로 커스텀 헤ッダー를 전송하려면 `-H` 플래그를 사용하십시오. 예를 들어 값이 `Value of custom header`인 `My-Custom-Header`라는 커스텀 헤ッダー를 전송하려면 다음을 실행합니다:

```sh
curl -H "My-Custom-Header: Value of custom header" http://httpbin.org/headers
```

リスポンス는 다음과 같습니다:

```json
{
  "headers": {
    "Accept": "*/*",
    "Host": "httpbin.org",
    "My-Custom-Header": "Value of custom header",
    "User-Agent": "curl/7.55.1",
    "X-Amzn-Trace-Id": "Root=1-65fd7d2a-3b683be160ff2965023b3a31"
  }
}
```

## Working with Empty Headers

때로는 빈 헤ッダー 전송이 필요합니다. 예를 들어 특정 API 요구사항을 준수하기 위해 콘텐츠가 없더라도 특정 헤ッダー가 필요할 수 있습니다. 예를 들어 [HTTP Strict Transport Security (HSTS) header](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Strict-Transport-Security)는 웹사이트에서 안전한 HTTPS 연결을 강제합니다. 이 헤ッダー에는 일반적으로 HSTS 지속 기간과 동작에 대한 지시문이 포함되지만, 빈 값으로 전송하면 즉시 HSTS 강제를 보장합니다.

빈 헤ッダー는 이전에 설정된 헤ッダー 값을 지우는 데도 사용할 수 있습니다. 기본으로 설정된 헤ッダー를 리셋 또는 클리어하려면, 빈 헤ッダー 전송으로 해당 값을 효과적으로 제거할 수 있습니다.

curl로 빈 헤ッダー를 전송하려면, 빈 값을 나타내기 위해 헤ッダー 이름 뒤에 세미콜론을 지정하십시오. 다음 명령은 `My-Custom-Header`라는 빈 커스텀 헤ッダー를 전송하는 방법을 보여줍니다:

```sh
curl -H "My-Custom-Header;" http://httpbin.org/headers
```

출력에는 `My-Custom-Header`가 빈 값으로 표시됩니다:

```json
{
  "headers": {
    "Accept": "*/*",
    "Host": "httpbin.org",
    "My-Custom-Header": "",
    "User-Agent": "curl/7.55.1",
    "X-Amzn-Trace-Id": "Root=1-65fd84e2-7a42d9d62a42741e448c426f"
  }
}
```

## Deleting Headers

curl로 헤ッダー를 완전히 제거하려면, 헤ッダー 이름 뒤에 콜론을 붙이고 그 뒤에 값을 지정하지 않으면 됩니다.

예를 들어 기본 `User-Agent` 헤ッダー를 제거하려면 다음을 사용하십시오:

```sh
curl -H "User-Agent:" http://httpbin.org/headers
```

リスポンス에는 `User-Agent` 헤ッダー가 포함되지 않으며, 제거되었음을 확인할 수 있습니다:

```json
{
  "headers": {
    "Accept": "*/*",
    "Host": "httpbin.org",
    "X-Amzn-Trace-Id": "Root=1-65fd862d-13b181583501ae11046374a1"
  }
}
```

## Sending Multiple Headers at Once

지금까지는 단일 헤ッダー 예시를 살펴보았지만, curl은 여러 헤ッダー의 동시 전송을 지원합니다. 명령에 여러 개의 `-H` 플래그를 포함하기만 하면 됩니다.

예를 들어 값이 각각 `one`, `two`인 두 헤ッダー(`Custom-Header-1`, `Custom-Header-2`)를 전송하려면 다음을 실행하십시오:

```sh
curl -H "Custom-Header-1: one" -H "Custom-Header-2: two" http://httpbin.org/headers
```

출력은 다음과 같습니다:

```json
{
  "headers": {
    "Accept": "*/*",
    "Custom-Header-1": "one",
    "Custom-Header-2": "two",
    "Host": "httpbin.org",
    "User-Agent": "curl/7.55.1",
    "X-Amzn-Trace-Id": "Root=1-65fd8781-143be3502c559bc5605fc6f1"
  }
}
```

## Summary

이 문서에서는 HTTP 헤ッダー의 기초를 다루고 curl을 사용하여 이를 효과적으로 관리하는 방법을 시연했습니다.

포괄적인 Web스크레이핑 솔루션을 원하신다면 Bright Data를 고려해 보십시오. Bright Data는 익명성을 강화하고 IP 차단을 방지하는 [proxy services](https://brightdata.co.kr/proxy-types)뿐만 아니라, CAPTCHA 없이 지리적으로 제한된 콘텐츠에 접근할 수 있도록 돕는 [Web Unlocker](https://brightdata.co.kr/products/web-unlocker) 등 전문 도구와 서비스를 제공합니다.

오늘 무료 체험을 시작해 보십시오!