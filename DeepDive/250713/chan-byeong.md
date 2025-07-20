# 14장 웹사이트 보안을 위한 리액트와 웹페이지 보안 이슈

이번 장에서는 리액트를 기반으로 한 웹 어플리케이션을 만드는 프론트엔드 개발자가 신경 써야 할 다양한 보안 이슈를 살펴보자

## 14.1 리액트에서 발생하는 크로스 사이트 스크립팅 (XSS)

XSS란 개발자가 아닌 제3자가 웹사이트에 악성 스크립트를 삽입해 실행할 수 있는 취약점을 의미한다.

리액트에서 이 XSS 이슈는 어떻게 발생할 수 있을까?

### 14.1.1 dangerouslySetInnerHTML prop

dangerouslySetInnerHTML은 특정 브라우저 DOM의 innerHTML을 특정한 내용으로 교체할 수 있는 방법이다.
일반적으로 게시판과 같이 사용자나 관리자가 입력한 내용을 브라우저에 표시하는 용도로 사용된다.

```javascript
function App() {
  return <div dangerouslySetInnerHTML={{ __html: "First and Seconde" }} />;
}
```

위으 컴포넌트 결과는 <div> First and Seconde </div> 이다.

dangerouslySetInnerHTML는 오직 `__html`을 키로 가지고 있는 객체만 인수로 받을 수 있으며 이 인수로 넘겨받은 문자열을 DOM에 그대로 표시하는 역할을 한다.

dangerouslySetInnerHTML의 위험성은 위수로 받는 문자열에 제한이 없다는 점이다.

예를 들여 문자열 대신 `<span><svg/onload=alert(origin)></span>`을 넣으면 웹페이지 방문시 `origin`이 알림창으로 나타나게 된다.

따라서 dangerouslySetInnerHTML을 통해 DOM에 삽입하는 요소를 검증하는 과정이 필요하다.

### 14.1.2 useRef를 활용한 직접 삽입

dangerouslySetInnerHTML와 비슷한 방법으로 DOM에 요소를 직접 삽입하는 방법으로 `useRef`가 존재한다.

`useRef`를 사용하면 DOM 요소에 직접 접근이 가능하고 앞서 보았던 방법과 비슷하게 innerHTML에 보안 취약점이 있는 스크립트를 삽입하면 동일한 문제가 발생한다.

```javascript
...
const divRef = useRef(null)

useEffect(() => {
  if(divRef.current) {
    divRef.current.innerHTML = <span><svg/onload=alert(origin)></span>
  }
})

return <div ref={divRef} />
```

### 14.1.3 리액트에서 XSS 문제를 피하는 방법

리액트에서 XSS를 피할 수 있는 가장 확실한 방법은 제3자가 삽입할 수 있는 HTML을 안전한 HTML코드로 한번 치환하는 것이다.

이러한 과정을 `sanitize` 또는 `escape`라고 한다. 이를 하는 가장 확실한 방법은 Npm 라이브러리를 활용하는 것이다.

여러 라이브러리 중 `sanitize-html`을 활용한 예제를 살펴보자

```javascript
import sanitizeHtml, { IOptions as SanitizeOptions } from 'sanitize-html'

//허용하는 태그
const allowedTags = ['div', 'a', ...]

//허용하는 태그에서 허용하는 속성
const defaultAttr = ['style', 'class' ]

//허용할 Iframe 도메인
const allowedIframeDomains = ['www.naver.com']

...

const sanitizedOptions = {
  allowedTags,
  allowedIframeDomains,
  ...
}

...

const sanitizedHtml = sanitizeHtml(html, sanitizedOptions)

```

`sanitize-html`은 허용하는 태그들을 나열하는 허용 목록을 일일이 작성해야하는 불편함이 존재한다. 하지만 이 방법이 훨씬 안전하다 만약 허용 목록에 추가하지 않은 태그가 있다면
해당 요소가 안 보이는 문제가 있을 수 있지만 반대로 차단 목록에 작성해야 할 태그를 놓친다면 이는 심각한 보안 문제가 발생할 수도 있다.

또 한 가지 중요한 점은 단순히 보여줄 때뿐만 아니라 사용자가 컨텐츠를 저장할 때도 한번 이스케이프 과정을 거치는 것이 더 효율적이고 안전하다는 것이다.

XSS 위험성이 존재하는 컨텐츠를 데이터베이스에 저장하지 않는 것이 예기치 못한 위협을 방지하는 데 훨씬 도움이 될뿐만 아니라, 한번 이스케이프 과정을 거치면 그 뒤로는 일일이 이스케이프 과정을 거치치 않아도 되기 때문에 훨씬 효율적이다.

이러한 치환과정은 서버에서 처리하는 것이 좋다. 일반적인 사용자라면 클라이언트에서 처리하는 것이 큰 문제가 되지 않을 수 있지만 만약 사용자가 직접 POST 요청을 스크립트나 curl을 통해서 요청하는 경우 문제가 발생할 수도 있다. 따라서 모든 데이터를 우선 의심해본다는 자세로 서버에서 이스케이프 처리를 하는 것이 좋다.

_리액트의 JSX 데이터 바인딩_

```text
앞에서 소개한 XSS와 관련해 리액트의 숨겨진 메커니즘이 있다. 왜 dangerouslySetInnerHTML이라는 속성이 별도로 존재하는 걸까? 그 이유는 기본적으로 리액트는 XSS를 방어하기 위해 이스케이프 작업이 존재하기 때문이다.

html = <div><svg/onlaod=alert(origin)></div>

`return <div>{html}</div>`

이러한 코드가 존재할 때 만약 리액트가 이스케이프하지 않는다면 해당 페이지 접속 시 alert 창이 등장할 것이다. 하지만 리액트는 기본적으로 이스케이프 처리를 하기 때문에 alert창은 등장하지 않는다.
```

## 14.2 getServerSideProps와 서버 컴포넌트를 주의하자

서버 사이드 렌더링과 서버 컴포넌트는 성능 이점을 가져다 줌과 동시에 서버라는 개발 환경을 프론트엔드 개발자에게 쥐어준 셈이 됐다. 서버에는 일반 사용자에게 노출되면 안 되는 정보들이 담겨 있기 때문에 클라이언트, 즉 브라우저에 정보를 내려줄 때는 조심해야 한다.

```javascript
export default function App({ cookie }) {
  if(!validateCookie(cookie)) {
    Router.replace(...)
    return null
  }

  ...
}

export const getServerSideProps = async (ctx: GetServerSidePropsContext) => {
  const cookie = ctx.req.headers.cookie || ''
  return {
    props: {
      cookie,
    },
  }
}
```

위의 예제는 getServerSideProps에서 cookie 정보를 가져온 다음, 이를 클라이언트 리액트 컴포넌트에 문자열로 제공해 클라이언트에서 쿠키의 유효성에 따라 이후 작업을 처리한다.

이는 의도한 대로 동작하는 코드일지는 몰라도 보안 관점에서는 썩 좋지 않다. 먼저 앞서 getServerSideProps에 대해 살펴본 것처럼 getServerSideProps가 반환하는 props 값은 모두 사용자의 HTML에 기록되고, 또한 전역 변수로 등록되어 자바스크립트로 접근이 가능한 보안 위협에 노출되는 값이 된다.

또한 충분히 getServerSideProps에서 처리할 수 있는 리다이렉트가 클라이언트에서 실행되어 성능 측면에서도 손해를 본다.getServerSideProps가 반환하는 값 또는 서버 컴포넌트가 클라이언트 컴폰넌트에 반환하는 props는 반드시 필요한 값으로만 철저하게 제한되어야 한다.

앞의 코드는 아래와 같이 수정이 가능하다.

```javascript
export default function App({token}) {
  const user = JSON.parse(window.atob(token.split('.')[1]))
  const user_id = user.id

  ...
}

export const getServerSideProps = async (ctx: GetServerSidePropsContext) {
  const cookie = ctx.req.headers.cookie || ''

  const token = validateCookie(cookie)

  if (!token) {
    return {
      redirect: {
        destination: '/404',
        permanent: false
      }
    }
  }

  return {
    props: {
      token,
    }
  }
}

```

변경된 점은 cookie 전체를 제공하는 것이 아니라 클라이언트에서 필요한 token값만을 제한해서 넘겨주고, 이 값이 없을 경우에 대한 예외도 서버에서 처리하였다.
이로써 불필요하게 cookie에 대한 모든 정보가 클라이언트에 반환되는 경우를 피했고 리다이렉트 또한 좀 더 빨라질 것이다.

이러한 방식의 접근법은 비단 getServerSideProps와 서버 컴포넌트뿐만 아니라 리덕스에서 서버 사이드에서 가져온 상태로 가져오는 `window.__PRELOAD_STATE__`와 같은 값을 초기화할 떄도 적용된다. `window.__PRELOAD_STATE__`의 값은 XSS에 취약할 수 있기 때문에 반드시 sanitize를 거치고 꼭 필요한 값만을 제공해야 한다.

## 14.3 <a> 태그의 값에 적잘한 제한을 둬야 한다.

웹 개발 시에 <a> 태그의 href에 `javascript:`로 시작하는 자바스크립트 코드를 넣어둔 경우를 본 적이 있을 것이다.
이는 <a> 태그의 기본 기능, 즉 href로 선언된 URL로 페이지를 이동하는 것을 막고 onClick이벤트와 같이 별도 이벤트 핸들러만 작동시키기 위한 용도로 자주 사용된다.

```javascript
return (
  <>
    <a href='javascript:;' onClick={handleClick}>
      link
    </a>
  </>
);
```

위 코드에서 `link`를 클릭하면 이동하지 않고 `handleClick`이 실행된다.

이러한 방식은 마크업 관점에서 안티패턴으로 볼 수 있다. 해당 코드는 href가 작동하지 않는 것이 아니라 javasript:;만이 실행되기 때문에 아무것도 일어나지 않는 것처럼 보인다

## 14.4 HTTP 보안 헤더 설정하기

HTTP 보안 헤더란 브라우저가 렌더링하는 내용과 관련된 보안 취약점을 미연에 방지하기 위해 브라우저와 함께 작동하는 헤더를 의미한다.

이는 웹사이트 보안에 가장 기초적인 부분으로 HTTP 보안 헤더만 효율적으로 사용할 수 있어도 많은 보안 취약점을 방지할 수 있다.

대표적으로 HTTP 보안 헤더로 무엇이 있는지 먼저 살펴보고, 이를 NextJs 등에 어떻게 적용하여 사용할 수 있을지 살펴보자

### 14.4.1 Strict-Transport-Security

HTTP의 Strict-Transport-Security 응답 헤더는 모든 사이트가 HTTPS를 통해 접근해야 하며, 만약 HTTP로 접근하는 경우 이러한 모든 시도는 HTTPS로 변경되게 한다.

사용법은 다음과 같다.

`Strict-Transport-Security: max-age=<expire-time>; includeSubDomains`

<expire-time>은 해당 설정을 브라우저가 기억해야 하는 시간을 의미하며, 초 단위로 기록된다. 이 기간 내에 HTTP로 사용자가 요청한다 하더라도 브라우저는 이 시간을 기억하고 있다가 자동으로 HTTPS로 요청하게 된다. 만약 헤더의 해당 시간이 경과되면 HTTP로 로드를 시도한 다음에 응답에 따라 HTTPS로 이동하는 등의 작업을 수행할 것이다.

만약 시간이 0으로 설정되어 있다면 헤더가 즉시 만료되고 HTTP로 요청한다.

일반적으로 1년 단위로 허용하지만 권장값은 2년이다.

`includeSubDomains`가 있을 경우 이러한 규칙이 모든 하위 도메인에 적용된다.

### 14.4.2 X-XSS-Protection

`X-XSS-Protection`은 비표준 기술로, 현재 사파리와 구형 브라우저에서만 제공되는 기능이다.

이 헤더는 XSS 취약점이 발견되면 페이지 로딩을 중단하는 헤더이다. 이 헤더는 `Content-Security-Policy`가 있다면 그다지 필요 없지만 `Content-Security-Policy`를 지원하지 않는 구형 브라우저에서는 사용이 가능하다.

그러나 이 헤더를 전적으로 믿어서는 안되고 페이지 내부에 XSS에 대한 처리가 되는 것이 좋다.

```text
X-XSS-Protection: 0
X-XSS-Protection: 1
X-XSS-Protection: 1; mode=block
X-XSS-Protection: 1; report=<reporting-uri>
```

- 0은 XSS 필터링을 끈다.
- 1은 기본값으로 XSS 필터링을 킨다.만약 XSS 공격이 감지되면 해당 코드를 제거한 안전한 페이지를 보여준다.
- 1; mode=block은 1과 유사하지만 코드를 제거하지는 것이 아니라 아예 접근를 막아버린다.
- 1; report=<reporting-uri>은 크로미움 기반 브라우저에서만 동작하면 XSS 공격이 감지되면 보고서를 해당 uri로 보낸다.

### 14.4.3 X-Frame-Options

`X-Frame-Options`은 페이지를 frame, iframe, embed, object 내부에서 렌더링을 허용할지를 나타낸다.

예를 들어, 네이버와 비슷한 주소를 가진 페이지가 있고, 이 페이지에서 네이버를 iframe으로 렌더링한다고 해보자. 사용자는 이 페이지를 진짜 네이버로 오해할 수 있고 공격자는 이를 활용하여 사용자의 개인정보를 탈취할 수 있다.

`X-Frame-Options`는 외부에서 자신의 페이지를 위와 같은 방식으로 삽입되는 것을 막아주는 헤더이다.

```text
X-Frame-Options: DENY
X-Frame-Options: SAMEORIGIN
```

- DENY:프레임 관련 코드가 있다면 무조건 막는다.
- SAMEORIGIN: 같은 origin에 대해서만 프레임을 허용한다.

### 14.4.4 Permissions-Policy

Permissions-Policy는 웹사이트에서 사용할 수 있는 기능과 사용할 수 없는 기능을 명시적으로 선언하는 헤더이다. 개발자는 다양한 브라우저의 기능이나 API를 선택적으로 활성화하거나 필요에 따라 비활성화할 수 있다. 여기서 말하는 기능은 카메라나 GPS와 같이 브라우저가 제공하는 기능을 말한다.

```text
# 모든 geolocation 사용을 막는다.
Permissions-Policy: geolocation=()

#geolocation을 페이지 자신과 몇 가지 페이지에 대해서만 허용한다.
Permissions-Policy: geolocation=(self "https://a.yceffort.kr")

#카메라는 모든 곳에서 허용한다.
Permissions-Policy: camera=*;

#pip 기능을 막고, geolocation은 자신과 특정 페이지만 허용하며, 카메라는 모든 곳에서 허용한다.
Permissions-Policy: picture-in-picture=(), geolocation=(self https://yceffort.kr), camera=*;
```

_*self*는 누굴 말하는 거지?_

### 14.4.5 X-Content-Type-Options

이 헤더를 이해하려면 먼저 MIME이 무엇인지 알아야 한다. MIME란 Multipurpose Internet Mail Extensions의 약자로 Content-type의 값으로 사용된다.
원래는 메일 전송 시 사용하던 인코딩 방식이지만, 현재는 Content-type의 대표적인 값으로 사용된다.

`X-Content-Type-Options`란 Content-type 헤더에서 제공하는 MIME 유형이 브라우저에 의해 임의로 변경되지 않게 하는 헤더이다.

즉, Content-type: text/css 헤더가 없는 파일은 브라우저가 임의로 CSS를 사용할 수 없다. 한마디로 웹서버가 브라우저에게 강제로 이 파일을 읽는 방식을 지정하는 것이 해당 헤더의 역할이다.

예를 들어 어떤 사용자가 .jpg 파일을 웹서버에 업로드했는데 실제 해당 파일이 스크립트 정보를 담고 있다고 해보자. 브라우저는 .jpg로 파일을 요청했지만 실제 파일 내용은 스크립트인 것을 보고 해당 코드를 실행할 수도 있다. 여기서 만약 악의적인 스크립트가 담겨져 있다면 보안에 위협이 될 수 있다.

이 경우 다음과 같은 헤더를 설정해 두면 파일의 타입이 CSS나 MIME이 text/css가 아닌 경우, 혹은 파일 내용이 script나 MIME 타입이 자바스크릡트 타입이 아니면 차단하게 된다.

`X-Content-Type-Options: nosniff`

### 14.4.6 Referrer-Policy

HTTP 요청에는 Referer라는 헤더가 존재하는데, 이 헤더에는 현재 요청을 보낸 페이지의 주소가 나타난다. 만약 링크를 통해 들어왔다면 해당 링크를 포함하고 있는 페이지 주소가, 다른 도메인에 요청을 보낸다면 해당 리소스를 사용하는 페이지의 주소가 포함된다.

이 헤더는 사용자가 어디서 와서 방문 중인지 인식할 수 있는 헤더지만 반대로 사용자 입장에서는 원치 않는 정보가 노출될 위험도 존재한다. Referrer-Policy 헤더는 이 Referer 헤더에서 사용할 수 있는 데이터를 나타낸다.

참고로 Referer와 Referrer-Policy의 철자가 다른 이유는 Referer라는 오타가 이미 표준으로 등록되 이후에 뒤늦게 오타임을 발견했기 때문이다.

Referer에 대해 이야기할 때는 출처(origin)를 빼놓을 수 없다. 출처와 이를 구성하는 용어에 대해 먼저 알아보자
먼저 https://yceffort.kr이라는 주소의 출처는 다음과 같이 구성돼 있다.

- scheme: https:// HTTPS 프로토콜을 의미한다.
- hostname: yceffort.kr이라는 호스트명을 의미한다.
- port: 443 포트를 의미한다. (https는 기본적으로 443포트를 사용한다.)

sheme, hostname, port의 조합을 출처라고 한다.

그리고 2개의 주소를 비교할 때 same-origin인지, cross-origin인지는 다음과 같이 구분할 수 있다.

|---|---|---|
|출처|비교 결과|이유|
|---|---|---|
|https://www.yceffort.kr:443| cross-origin | 서브도메인이 다르다. |
|http://yceffort.kr:443| cross-origin | scheme이 다르다. |
|https://yceffort.kr:80| cross-origin| port가 다르다. |
|https://yceffort.kr| same-origin | 명시적인 포트는 없지만 HTTPS의 기본 포트인 443으로 간주한다. |

이러한 출처에 대한 정보를 바탕으로 Referrer-Policy의 각 값별로 다음과 같이 작동한다.

주요 Referrer-Policy 값과 동작:

- no-referrer: Referer 헤더를 아예 보내지 않습니다. 요청을 보내는 페이지의 주소 정보를 포함하지 않습니다.

- no-referrer-when-downgrade: HTTPS에서 HTTP로 요청을 보낼 때 Referer 정보를 보내지 않고, 나머지 경우에는 원본 URL을 Referer로 보냅니다.

- origin: 요청이 동일 출처(same-origin) 간에 이루어질 때에는 전체 URL을, 다른 출처(cross-origin) 간에는 출처(origin)만 Referer로 보냅니다.

- same-origin: 동일 출처 요청일 경우에만 전체 URL을 Referer로 보내고, 다른 출처 요청에는 Referer를 보내지 않습니다.

- strict-origin: 항상 출처(origin)만 Referer로 보내지만, HTTPS에서 HTTP로 요청을 보낼 때는 Referer를 보내지 않습니다.

- origin-when-cross-origin: 동일 출처 요청일 때에는 전체 URL을, 다른 출처 요청일 때에는 출처(origin)만 Referer로 보냅니다.

- strict-origin-when-cross-origin: strict-origin과 유사하지만, HTTPS에서 HTTP로 요청을 보낼 때 Referer를 보내지 않는다는 점에서 차이가 있습니다.

- unsafe-url: 항상 전체 URL을 Referer로 보냅니다. 보안에 취약할 수 있으므로 권장되지 않습니다.

Referrer-Policy는 응답 헤더뿐만 아니라 페이지의 <meta /> 태그로도 다음과 같이 설정할 수 있다.

`<meta name='referer' content='origin'/>`

그리고 페이지 이동 시나 이미지 요청, link 태그 등에도 다음과 같이 사용할 수 있다.

`<a href="https://yceffort.kr" referrerpolicy="origin">...</a>`

구글에서는 이용자의 개인정보 보호를 위해 strict-origin-when-cross-origin 혹은 그 이상을 명시적으로 선언해 둘 것을 권고한다. 만약 이 값이 설정돼 있지 않다면 브라우저의 기본값으로 작동하게 되어 웹사이트에 접근하는 환경별로 다른 결과를 만들어 혼란을 야기할 수 있다. 이러한 기본값이 없는 브라우저에서는 Referer 정보가 유츌될 수도 있다.

### 14.4.7 Content-Security-Policy

콘텐츠 보안 정책(CSP)은 XSS 공격이나 데이터 삽입 공격과 같은 다양한 보안 위협을 막기 위해 설계됐다. 대표적으로 이용되는 몇가지만 살펴보자

#### \*-src

font-src, img-src, sript-src 등 다양한 src를 제어할 수 있는 지시문이다.

```text
Content-Security-Policy: font-src <source>;
Content-Security-Policy: font-src <source> <source>;
```

위와 같이 선언해두면 font의 src로 가져올 수 있는 소스를 제한할 수 있다.
여기서 선언된 font 소스만 가져올 수 있으며 이 외의 소스는 모두 차단된다.

`Content-Security-Policy: font-src https://yceffort.kr/`

위와 같은 응답 헤더를 반환했다면 다음 폰트는 사용할 수 없게 된다.

```html
<style>
  @font-face {
    ...
    src: url(https://fonts.gstatic.com/...)
  }
</style>
```

이와 비슷한 유형의 지시문에는 다음과 같은 것이 있다.

- script-src: <script>의 src
- style-src: <style>의 src
- connect-src: 스크립트로 접근할 수 있는 URL을 제한한다.
- worker-src: Worker의 리소스
  ...

만약 해당 -src가 선언돼 있지 않다면 `default-src`로 한 번에 처리할 수도 있다.

```text
Content-Security-Policy: default-src <source>;
Content-Security-Policy: default-src <source> <source>;
```

default-src는 다른 여차 \*-src에 대한 폴백 역할을 한다.

#### form-action

form-action은 폼 양식으로 제출할 수 있는 URL을 제한할 수 있다. 다음과 같이 form-action 자체를 모두 막아버리는 것도 가능하다.

`<meta http-equiv="Content-Security-Policy" content="form-action 'none'" />`

### 14.4.8 보안 헤더 설정하기

#### Next.js

Next에서는 HTTP 경로 별로 보안 헤더를 적용할 수 있다. 이 설정은 `next.config.js`에서 다음과 같이 추가할 수 있다.

```javascript
const securityHeaders = [
  {
    key: "default-src",
    value: "'self'",
  },
];

module.exports = {
  async headers() {
    return [
      {
        source: "/:path*",
        headers: securityHeaders,
      },
    ];
  },
};
```

여기서 설정할 수 있는 값은 다음과 같다.

X-DNS-Prefetch-Control , Strict-Transport-Security , X-XSS-Protection , ...

#### NGINX

정적인 파일을 제공하는 NGINX의 경우 다음과 같이 경로별로 add_header 지시자를 사용해 원하는 응답헤더를 추가할 수 있다.

```yaml
location / {
 # ...
 add_header X-XSS-Protection "1; mode=block";
 ...
}
```

### 14.4.9 보안 헤더 확인하기

`https://securityheaders.com`에 방문하여 보안 헤더의 현황을 파악할 수 있다.

## 14.5 취약점이 있는 패키지의 사용을 피하자

## 14.6 OWASP Top 10

OWASP는 Open Worldwide Application Security Project라는 오픈소스 웹 어플리케이션 보안 프로젝트를 의미한다. 주로 웹에서 발생할 수 있는 정보 노출, 악성 스크립트, 보안 취약점 등을 연구하며 주기적으로 10대 웹 어플리케이션 취약점을 공개한다.

https://owasp.org/www-project-top-ten/

# 15장 마치며

## 15.1 리액트 프로젝트를 시작할 때 고려해야 할 사항

### 15.1.1 유지보수 중인 서비스라면 리액트 버전을 최소 16.8.6에서 최대 17.0.2로 올려두자

기존의 클래스형 컴포넌트를 함수형으로 변경해야할 이유가 있을까? 그에 대한 대답은 굳이 그럴필요 없다이다.

리액트 팀에서는 클래스형 컴포넌트를 사라지게 할 계획은 없다고 밝혔다.

### 15.1.2 인터넷 익스플로러 11 지원을 목표한다면 각별히 주의해야 한다.

익스플로러 11이 공식적으로 지원을 중단했다. 다음은 익스플로러에서 지원하지 않는 대표적인 라이브러리이다.

- 리액트: 리액트는 18버전부터 익스플로러 11을 지원하지 않기로 했다.
- Next.js: Next는 13버전 부터 공식적으로 익스플로러 11을 지원하지 않기로 했다.
- query-string: 주소의 쿼리 문자열을 다루는 대표적인 라이브러리인 query-string도 6.x 버전부터 지원하지 않는다.

익스플로러 11을 지원하기로 했다면 각별히 라이브러리 설치에 주의를 기하자

### 15.1.3 서버 사이드 렌더링 어플리케이션을 우선적으로 고려해야 한다.

기본 HTML에 온전히 자바스크립트로 렌더링과 라우팅을 수행하는 SPA는 좋은 성능 결과를 얻기 어렵다.

사용자의 기기 성능은 제각각이기 때문에 자바스크립트 코드의 실행 속도에 의존적일수록 평균적인 사용자 경험을 제공하기란 어렵다.

### 15.1.4 상태 관리 라이브러리는 꼭 필요할 때만 사용한다.

현재는 Context API와 훅의 등장으로 prop drilling 문제를 겪지 않고도 하위 컴포넌트에 원하는 상태값을 전달할 수 있게 되었다.
물론 이것은 언제까지나 상태를 주입하는 용도로 상태 관리 라이브러리만큼 다양한 기능을 제공하지는 않는다.

### 15.1.5 리액트 의존성 라이브러리 설치를 조심한다.

어떤 라이브러리는 리액트에 의존적이다. 대게 `react-*`같은 이름 형태를 가지고 있다.

```json
  "peerDependencies": {
    "react": "^16.0.8" || "^17.0.1",
    ...
  }
```

이때 반드시 peerDependencies가 설치하고자 하는 프로젝트의 리액트 버전과 맞는지 확인해야 한다.

## 15.2 언젠가 사라질 수도 있는 리액트

이번 절에서는 리액트 개발자가 아닌 웹 개발자로서 바라보는 리액트의 불편함과 더 오랜 기간 웹 개발자로서 자리매김하는 데 도움이 될 수 있는 내용을 살펴본다.

### 15.2.1 리액트는 그래서 정말 완벽한 라이브러리인가?

리액트는 많이 사용하는 라이브러리이지만 완벽한 라이브러리라 하기에는 어렵다.

리액트의 단점을 살펴보자

#### 클래스 컴포넌트에서 함수 컴포넌트로 넘어오면서 느껴지는 혼란

#### 너무 방대한 자유가 주는 혼란

리액트 기반 프로젝트에서 스타일을 입힌다고 하면 다양한 선택지가 존재한다.

- 외부 스타일시트 임포트
- 인라인 스타일
- CSS Module 기법
- styled-component

비단 CSS만이 아니더라도 데이터를 fetch하는 방법, 상태 관리까지 다양한 것들이 파편화 돼있다.
자칫 파편화된 리액트 기술 스택은 새로운 리액트 개발자에게 장애물이 될 수 있다.

### 15.2.2 오픈소스 생태계의 명과 암

#### 페이스북 라이센스 이슈

오픈소스 라이센스 가운데 가장 널리 쓰이는 라이센스를 꼽는다면 바로 MIT 라이센스다. 페이스북은 자사의 오픈소스인 React, Immutable, Jest 등에 이 MIT 라이센스 대신에 BSD+Patents 라이센스를 사용하고 있었다. 이 라이센스는 다른 라이센스와 다르게 '이 라이센스를 적용한 소프트웨어에 대해서 특정한 사건이 발생한다면 라이센스가 통지 없이 종료될 수 있다.'라는 한 가지 조항이 있다.

#### 오픈소스는 무료로 계속 제공될 수 있는가? colors.js faker.js 그리고 바벨

바벨은 자바스크립트 컴파일러로 자바스크립트 최신 문법을 지원하지 않는 브라우저에서도 최신 문법을 사용할 수 있도록 자바스크립트를 컴파일해 주는 도구다.

# 추가 학습

### preload

브라우저에게 필요한 리소스를 미리 알려주는 역할을 수행

브라우저는 해당 리소스를 미리 다운로드함 (단, 바로 실행하지는 않는다.)

```html
<!-- 중요한 폰트 파일 미리 로드 -->
<link
  rel="preload"
  href="/fonts/main.woff2"
  as="font"
  type="font/woff2"
  crossorigin
/>

<!-- 중요한 CSS 파일 미리 로드 -->
<link rel="preload" href="/css/critical.css" as="style" />
```

### preconnect

외부 도메인과 미리 연결을 설정하도록 브라우저에게 지시

DNS 조회, TCP 핸드셰이크, TLS 협상을 미리 수행하고 실제 리소스 다운로드는 하지 않고 연결만 준비

```html
<!-- Google Fonts 연결 미리 설정 -->
<link rel="preconnect" href="https://fonts.googleapis.com" />
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin />

<!-- CDN 연결 미리 설정 -->
<link rel="preconnect" href="https://cdn.example.com" />
```
