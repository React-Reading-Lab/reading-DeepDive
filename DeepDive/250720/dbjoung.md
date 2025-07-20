# 웹사이트 보안을 위한 리액트와 웹페이지 보안 이슈

보안 이슈는 버그와 같이 개발자의 실수에서 비롯되는 경우도 있지만, 이러한 코드나 프로세스가 보안에 문제가 된다는 것을 인지하지 못하는 경우에도 발생한다. 또한, 보안 이슈는 프레임워크나 라이브러리가 모두 알아서 해결해 주는 것이 아니기 때문에 반드시 개발자 스스로가 주의를 기울여야 한다.

이번 장에서는 리액트를 기반으로 한 웹 앱을 만드는 프론트엔드 개발자가 신경 써야 할 다양한 보안 이슈를 다룬다.

## 리액트에서 발생하는 크로스 사이트 스크립팅(XSS)

크로스 사이트 스크립팅(Cross-Site Scripting, XSS)란 웹 앱에서 가장 많이 보이는 취약점 중 하나로, 웹사이트 개발자가 아닌 제 3자가 웹사이트에 악성 스크립트를 삽입해 실행할 수 있는 취약점을 의미한다.

일반적으로 게시판과 같이 사용자가 입력을 할 수 있고, 이 입력을 다른 사용자에게 보여줄 수 있는 경우에 발생한다.

예를 들어, 어떤 게시판에 사용자가 다음과 같은 글을 올린다고 가정해보자.

```html
<p>사용자가 글을 작성했습니다.</p>
<script>
  alert("XSS");
</script>
```

위 글을 방문했을 때 별도의 조치가 없다면 script도 함께 실행되어 window.alert도 함께 실행될 것이다.

프론트엔드 개발자와 마찬가지로 script가 실행될 수 있다면 웹사이트 개발자가 할 수 있는 모든 작업을 함께 수행할 수 있으며, 쿠키를 획득해 사용자의 로그인 세션 등을 탈취하거나 사용자의 데이터를 변경하는 등 각종 위험성이 있다.

리액트에서 이 XSS 이슈는 다음과 같은 이유로 발생한다.

### dangerouslySetlnnerHTML prop

dangerouslySetlnnerHTML 은 특정 브라우저 DOM의 innerHTML을 특정한 내용으로 교체할 수 있는 방법이다. 일반적으로 게시판과 같이 사용자나 관리자가 입력한 내용을 브라우저에 표시하는 용도로 사용된다.

```jsx
function App() {
  // 다음 결과물은 <div>First · Second</div> 이다.
  return <div dangerouslySetInnerHTML={{ __html: "Fisrt &middot; Second" }} />;
}
```

dangerouslySetInnerHTML 은 오직 \_\_HTML 을 키를 가지고 있는 객체만 인수로 받을 수 있으며, 이 인수로 넘겨받은 문자열을 그대로 DOM에 표시하는 역할을 한다.

이 dangerouslySetInnerHTML의 위험성은 인수로 받는 문자열에는 제한이 없다는 것이다.

```jsx
const html = `<span><svg/onload=alert(origin)></span>`

function App() {
	return <div dangerouslySetInnerHTML={{ __html: html}}>
}

export default App;
```

위 코드를 실행하면 origin 변수값이 alert 창에 표시된다.

### useRef를 활용한 직접 삽입

useRef로도 DOM에 직접 삽입할 수 있다. useRef를 활용하면 직접 DOM에 접근할 수 있으므로 이 DOM에 앞서와비슷한방식으로 innerHTML에 보안 취약점이 있는 스크립트를 삽입하면 동일한 문제가 발생한다.

```jsx
const html = `<span><svg/onload=alert(origin)></span>`;

function App() {
  const divRef = useRef < HTMLDivElement > null;

  useEffect(() => {
    if (divRef.current) {
      divRef.current.innerHTML = html;
    }
  });

  return <div ref={divRef} />;
}
```

### 리액트에서 XSS 문제를 피하는 방법

제 3자가 삽입할 수 있는 HTML을 안전한 HTML 코드로 한 번 치환하는 것이다. 이러한 과정을 새니타이즈(sanitize) 또는 이스케이프(escape)라고 하는데, 새니타이즈를 직접 구현해 사용하는 등 다양한 방법이 있지만 가장 확실한 방법은 npm에 있는 라이브러리를 사용하는 것이다.

새니타이즈 관련 유명한 라이브러리는 다음과 같다.

- DOMpurity
- sanitize-html
- js-xss

이 중 sanitize-thml은 허용할 태그와 목록을 일일이 나열하는 이른바 허용 목록 방식을 채택하고 있다.

허용 목록에 추가하는 것을 깜빡한 태그나 속성이 있다면 단순히 HTML이 안 보이는 사소한 이슈로 그치겠지만, 차단 목록으로 해야할 것을 놓친다면 그 즉시 보안 이슈로 연결되기에 비교적 안전하다.

- 이러한 치환 과정은 사버에서 수행하는 것이 좋다. post 요청을 스크립트나 curl 등으로 직접 요청하는 경우에는 스크립트에서 실행하는 이스케이프 과정을 생략하고 바로 저장될 가능성이 있다.

또, 단순히 게시판과 같은 예시가 웹사이트에 없다고 하더라도 XSS 문제는 충분히 발생할 수 있다. 예를 들어, 다음과 같이 쿼리스트링에 있는 내용을 그대로 실행하거나 보여주는 경우에도 보안 취약점이 발생한다.

```jsx
import { useRouter } from "next/router";

function App() {
  const router = useRouter();
  const query = router.query;
  const html = query?.html?.toString() || "";

  return <div dangerouslySetInnerHTML={{ __html: html }} />;
}
```

따라서 개발자는 \*\*자신이 작성한 코드가 아닌 query, GET 파라미터, 서버에 저장된 사용자가 입력한 데이터 등 외부에 존재하는 모든 코드를 위험한 코드로 간주하고 이를 적절히 처리하는 것이 좋다.

## getServerSideProps와 서버 컴포넌트를 주의하자

서버에는 일반 사용자에게 노출되면 안 되는 정보들이 담겨 있기 때문에 클라이언트, 즉 브라우저에 정보를 내려줄 때는 조심해야 한다.

```jsx
export default function App() {
  if (!validateCooke(cookie)) {
    Router.replace(/*...*/);
    return null;
  }

  /* do something... */
}

export const getServerSideProps = async (ctx: GetServerSidePropsContext) => {
  const cookie = ctx.req.headers.cookie || "";
  return {
    props: {
      cookie,
    },
  };
};
```

위 코드는 cookie 정보를 가져온 다음 이를 클라이언트 리액트 컴포넌트에 문자열로 제공해 클라이언트에서 쿠키의 유효성에 따라 이후 작업을 처리한다. 의도한대로는 작동하겠지만, 보안 관점에서는 좋지 못하다.

- getServerSideProps가 반환하는 props 값은 모두 사용자의 HTML에 기록되고, 또한 전역 변수로 등락되어 스크립트로 충분히 접근할 수 있는 보안 위협에 노출되는 값이 된다.
- getServerSideProps에서 처리할 수 있는 리다이랙트가 클라이언트에서 실행되어 성능 측면에서도 손해를 본다.

getServerSideProps가 반환하는 값 또는 서버 컴포넌트가 클라이언트 컴포넌트에 반환하는 props는 반드시 필요한 값으로만 철저하게 제한되어야 한다.

위 코드는 아래와 같이 수정할 수 있다.

```jsx
export default function App({ token }: { token: string }) {
  const user = JSON.parse(window.atob(token.split(".")[1]));
  const user_id = user.id;

  /* do something... */
}

export const getServerSideProps = async (ctx: GetServerSidePropsContext) => {
  const cookie = ctx.req.headers.cookie || "";

  const token = validateCookie(cookie);

  if (!token) {
    return {
      redirect: {
        destination: "/404",
        permanent: false,
      },
    };
  }

  return {
    props: {
      token,
    },
  };
};
```

쿠키 전체를 제공하는 것이 아니라 클라이언트에서 필요한 token 값만 제한적으로 반환하고, 이 값이 없을 때 예외 처리할 리다이렉트도 모두 서버에서 처리했다.

## `<a>` 태그의 값에 적절한 제한을 둬야 한다

```jsx
function App() {
  function handleClick() {
    console.log("hello");
  }

  return (
    <>
      <a href="javascript:;" onClick={handleClick}>
        링크
      </a>
    </>
  );
}
```

위는 a태그의 기본 기능인 href로 선언된 URL로 페이지를 이동하는 것을 막고 onClick 이벤트와 같이 별도 이벤트 핸들러만 작동시키기 위한 용도로 주요 사용된다.

이러한 방식은 마크업 관점에서 안티패턴이다. a 태그는 반드시 페이지 이동이 있을 때만 사용하는 게 좋다.

여기서 `javascript:` 는 href 내에 JS 코드가 존재한다면 이를 실행하라는 의미다. 따라서, href 속성에 사용자가 입력한 주소를 넣을 수 있다면 이 또한 보안 이슈로 이어질 수 있다.

따라서 href에 들어갈 수 있는 값을 제한해야 한다. 또한 피싱 사이트로 이어지지 않도록 origin도 확인해 처리하면 더 좋다.

## HTTP 보안 헤더 설정하기

HTTP 보안 헤더란 브라우저가 렌더링하는 내용과 관련된 보안 취약점을 미연에 방지하기 위해 브라우저와 함께 작동하는 헤더를 의미한다.

웹사이트 보안에 가장 기초적인 부분으로, HTTP 보안 헤더만 효율적으로 사용할 수 있어도 많은 보안 취약점을 예방할 수 있다.

대표적인 HTTP 보안 헤더로 무엇이 있는 지 살펴보자.

### Strict-Transport-Security

HTTP의 이 응답 헤더는 모든 사이트가 HTTPS를 통해서만 접근해야 하며, 만약 HTTP로 접근하는 경우 이러한 모든 시도는 HTTPS로 변경되게 한다.

```jsx
Strict-Transport-Security: max-age=<expire-time>; includeSubDomains
```

`<expire-time>`은 이 설정을 브라우저가 기억해야 하는 시간을 의미하며, 초 단위로 기록된다. 이 기간 내에는 HTTP로 사용자가 요청한다 하더라도 브라우저는 이 시간을 기억하고 있다가 자동으로 HTTPS로 요청하게 된다.

### X-XSS-Protection

비표준 기술로, 현재 사파리와 구형 브라우저에서만 제공되는 기능이다.

페이지에서 XSS 취약점이 발견되면 페이지 로딩을 중단하는 헤더다. 하지만 이 헤더를 전적으로 믿어서는 안되며, 반드시 페이지 내에 XSS 관련 처리가 존재하는 게 좋다.

### X-Frame-Options

X-Frame-Options는 페이지를 frame, iframe, embed, object 내부에서 렌더링을 허용할지를 나타낼 수 있다. 즉, 외부에서 자신의 페이지를 위와 같은 방식으로 삽입되는 것을 막아주는 헤더다.

### Permissions-Policy

웹사이트에서 사용할 수 있는 기능과 사용할 수 없는 기능을 명시적으로 선언하는 헤더다. 개발자는 다양한 브라우저의 기능이나 API를 선택적으로 활성화하거나 비활성화 할 수 있다.

여기서 말하는 **기능**이란 카메라나 GPS같이 브라우저가 제공하는 기능을 말한다.

예를 들어, 브라우저에서 사용자의 위치를 확인하는 기능(geolocation)과 같은 코드를 전혀 작성하지 않았다고 가정해보자. 그러나 해당 기능이 별도로 차단돼 있지 않고, 그 와중에 XSS 공격 등으로 이 기능을 취득해서 사용하게 되면 사용자의 위치를 획득할 수 있게 된다.

이를 대비해 해당 헤더를 사용하면, 혹히 XSS 가 발생하더라도 사용자에게 미칠 수 있는 악영향을 제한할 수 있다.

### X-Content-Type-Options

MIME(Multipurpose Internet Mail Extension)은 Content-type 값으로 사용되는데, 원래는 메일을 전송할 때 사용하던 인코딩 방식이었지만 지금은 Content-type에서 대표적으로 사용되고 있다.

MIME은 JPG, CSS, JSON, text/html 등 다양하다.

X-Content-Type-Options는 이 Content-type, 헤더에서 제공하는 MIME 타입 유형이 브라우저에 의해 임의로 변경되지 않게 해주는 헤더다. 웹서버가 브라우저의 파일 읽는 방식을 강제로 지정하도록 하는 것이다.

- 예를 들어, 공격자가 jpg 파일을 서버에 업로드했는데, 이 파일은 사실 스크립트 정보를 담고 있다고 가정해보자. 이 경우 브라우저는 jpg 파일을 요청했지만 실제 파일 내용은 스크립트인 것을 보고 해당 스크립트를 실행시켜버릴 수도 있다. 이러한 보안 위협을 방지할 수 있는 것이다.

### Referrer-Policy

HTTP 요청에는 Refferer라는 헤더가 존재한다. 이 헤더에는 현재 요청을 보낸 페이지의 주소가 나타난다.

만약 링크를 통해 들어왔다면 해당 링크를 포함하고 있는 페이지의 주소가, 다른 도메인에 요청을 보낸다면 해당 리소스를 사용하는 페이지의 주소가 포함된다.

이 헤더는 사용자가 어디서 와서 방문 중인지 인식할 수 있는 헤더지만, 반대로 사용자 입장에서는 원치 않는 정보가 노출될 위험도 존재한다.

Referrer-Policy 헤더는 이 Referrer-Policy 헤더는 이 Referer 헤더에서 사용할 수 있는 데이터를 나타낸다.

Referer를 얘기할 때는 출처(origni)을 빼놓을 수 없다. 출처와 이를 구성하는 용어에 대해 먼저 알아보자.

`https://yceffort.kr` 이라는 주소의 출처는 다음과 같이 구성돼 있다.

- sheme: HTTPS 프로토콜을 의미한다.
- hostname: [yceffort.kr](http://yceffort.kr) 이라는 호스트명을 의미한다.
- port : 443 포트를 의미한다.

이 scheme, hostname, port의 조합을 출처(origin)이라 부른다.

두 주소를 비교할 때 **same-origin인지, cross-origin인지 구분**할 수 있다.

[`https://yceffort.kr:443`](https://yceffort.kr:443) 과 비교해보자.

- [`https://fake.kr:443`](https://fake.kr:443) : cross-origin / 도메인이 다르다.
- `https://www.yceffort.kr:443` : cross-origin / 서브 도메인이 다르다.
- [`https://blog.yceffort.kr443`](https://blog.yceffort.kr443) : cross-origin / 서브 도메인이 다르다.
- `http://yceffort.kr:443` : cross-origin / scheme가 다르다.
- [`https://yceffort.kr:80`](https://yceffort.kr:80) : cross-origin / port가 다르다.
- [`https://yceffort.kr:443`](https://yceffort.kr:443) : same-origin / 완전히 같다.
- [`https://yceffort.kr`](https://yceffort.kr) : same-origin / 명시적인 포트가 없지만 HTTPS의 기본 포트인 443으로 간주.

응답 헤더 뿐만 아니라 페이지의 `<meta/>` 태그로도 다음과 같이 설정할 수 있다.

```jsx
<meta name="referrer" content="origin" /> //응답 해더
<a href="http://yceffort.kr" referrerpolicy="origin">...</a> // 메타 태그
```

구글에서는 이용자의 개인정보 보호를 위해 `strict-origin-when-cross-origin` 혹은 그 이상을 명시적으로 선언해 둘 거슬 권고한다.

만약 이 값이 설정돼 있지 않다면 브라우저의 기본값으로 작동하게 되어 웹사이트에 접근하는 환경별로 다른 결과를 만들어 내 혼란을 야기할 수 있으며, 이러한 기본값이 없는 구형 브라우저에서는 Referer 정보가 유출될 수도 있다.

### Content-Security-Policy

콘텐츠 보안 정책(Content-Security-Policy, 이하 CSP)은 XSS 공격이나 데이터 삽입 공격과 같은 다양한 보안 위협을 막기 위해 설계됐다.

- \*-src

  - font-src, img-src, script-src 등 다양한 src를 제어할 수 있는 지시문이다. 얘를 들어, 다음과 같이 응답 헤더를 설정해보자.
    ```jsx
    Content-Security-Policy: font-src https://yceffort.kr/
    ```
    그럼 다음 폰트를 사용할 수 없게 된다.
    ```jsx
    <style>
    	@font-face {
    		font-family: 'Noto Sans KR';
    		font-style: normal;
    		font-weight: 400;
    		font-display: swap;
    			src: url(https://fonts.gstatic.com/s/notosanskr/v13/PbykF...woff2)
    						format('woff2')
    	}
    </style>
    ```
  - script-src, style-src 등 다양한 지시문들이 있다.
  - 만약 -src가 선언돼 있지 않다면 default-src로 한번에 처리할 수도 있다.
    - 이는 다른 여타 _-src에 대한 폴백 역할을 한다. 만약 _-src에 대한 내용이 없다면 이 지시문을 사용하게 된다.

- form-action
  폼 양식으로 제출할 수 있는 URL을 제한할 수 있다. 다음과 같이 form-action 자체를 모두 막아버리는 것도 가능하다.
  ```jsx
  <meta http-equiv="Content-Security-Police" content="form-action 'none'" />;
  export default function App() {
    function handleFromAciton() {
      alert("form action");
    }

    return (
      <div>
        <form action={handleFormAction} method="post">
          <input type="text" name="name" value="foo" />
          <input type="submit" id="submit" value="submit" />
        </form>
      </div>
    );
  }
  ```
  해당 컴포넌트에서 submit을 눌러 form을 제출하면 콘솔에 다음과 같은 에러 메시지가 출력되면서 작동하지 않는다.
  ```jsx
  Refused to send form data to '****' because it violates the following Content
  Security Policy directive: "form-action 'none'".
  ```

### 보안 헤더 설정하기

- Next.js
  Next.js에서는 앱 보안을 위해 HTTP 경로별로 보안 헤더를 적용할 수 있다. 이 설정은 next.config.js에서 추가할 수 있다.
- NGINX
  정적인 파일을 제공하는 NGINX의 경우 경로 별로 add_header 지시자를 사용해 원하는 응답 헤더를 추가할 수 있다.

### 보안 헤더 확인하기

현재 서비스 중인 웹사이트의 보안 헤더를 확인할 수 있는 가장 빠른 방법은 보안 헤더의 현황을 알려주는 [`https://securityheaders.com/`을](https://securityheaders.com/을) 방문하는 것이다.

헤더를 확인하고 싶은 웹사이트의 주소를 입력하면 곧바로 현재 보안 헤더 상황을 알 수 있다.

## 취약점이 있는 패키지 사용을 피하자

npm 프로젝트가 의존하는 패키지 목록은 기본적으로 package.json의 dependencies와 devDependencies에 나열돼 있고, package-lock.json에는 package.json이 의존하는 또 다른 패키지들이 명시돼있다.

이 두 파일만 보고 프로젝트가 의존하는 모든 패키지를 파악하고 관리하는 것은 사실상 불가능하다. 따라서 깃허브 Dependabot이 발견한 취약점은 필요하다면 빠르게 업데이트해 조치해야하낟.

그리고 리액트, Next.js 또는 사용 중인 상태 관리 라이브러리와 같이 프로젝트를 구성하는 핵심적인 패키지는 버저닝과 패치 수정 등을 항상 주의해야 한다.

## OWASP Top 10

OWASP 은 Open Worldwide Application Security Project 라는 오픈소스 웹 앱 보안 프로젝트를 의미한다. 주로 웹에서 발생할 수 있는 정보 노출, 악성 스크립트, 보안 취약점 등을 연구하며, 주기적으로 10대 웹 앱 취약점을 공개하는데 이를 `OWASP Top 10`이라고 부른다.

해당 웹사이트를 살펴보고 어떠한 보안 취약점이 존재하는지, 현재 문제가 되는 부분은 무엇인지 등을 한번 돌이켜보고 점검해 보자.

# 마치며

## 리액트 프로젝트를 시작할 때 고려해야 할 사항

_아래 사항은 책이 저술된 2022-2023년을 기준으로 하는 고려 사항이므로, 간략하게만 정리하고 넘어간다._

- 유지보수 중인 서비스라면 리액트 버전을 최소 16.8.6에서 17.0.2로 올려두자. 이유는 앞선 장에서 살펴봤듯이 17버전에는 리액트 앱 유지보수를 위한 기능이 많이 추가됐기 때문이다.
- 클래스 컴포넌트를 사라지게 할 계획이 없다고 공식적으로 발표했으므로, 클래스 컴포넌트를 급하게 함수 컴포넌트로 전환할 필요는 없다.
- 인터넷 익스플로어 11 지원이 공식적으로 종료됐으니, 혹 해당 브라우저를 사용할 계획이라면 유의하자.
- 서버 사이드 렌더링 앱을 우선적으로 고려한다.
  - 싱글 페이지 앱은 대부분의 경우 라이트하우스와 WebPageTest, 구글 개발자 도구에서 좋은 결과를 얻기 어렵다. 많은 사용자를 감당해야 하고, 혹은 그럴 계획이 있다면, 그리고 서버를 준비할 수 있는 충분한 여유가 있다면 시작부터 서버 사이드 렌더링을 고려하는것이 좋다.
  - Next.js, Remix, Shopify에서 만드는 Hydrogen 또한 좋은 대안이 될 수 있다.
- 상태 관리 라이브러리를 꼭 필요할 때만 쓴다.
  - 상태 관리 라이브러리가 필요한지 여부란, 앱에서 관리해야 할 상태가 많은지 여부를 말한다. 문서 편집기처럼 관리해야 할 생티가 많고, 여러 상태를 합성해서 또 새로운 상태를 파생하는 등 상태에 대한 여러 가지 필요성이 많은 앱이라면 상태 관리를 사용하는 게 좋다.
- 리액트 의존성 라이브러리 설치를 조심한다.
  - 리액트에 의존적인 라이브러리를 설치하려고 하는 경우, 이 라이브러리들은 대부분 `react-**` 같은 이름을 가진다.
  - 이 때, 반드시 peerDependencies가 설치하고자 하는 프로젝트의 리액트 버전과 맞는지 확인해야 한다.

## 언젠가 사라질 수도 있는 리액트

- 리액트는 현재 가장 널리 쓰이는 프론트엔드 라이브러리지만 그렇다고 가장 완벽한 라이브러리라고 단정할 수는 없다. 리액트를 반대하는 사람들의 의견은 주로 다음과 같다.
  - 클래스 컴포넌트에서 함수 컴포넌트로 넘어오면서 느껴지는 혼란
  - 너무 방대한 자유가 주는 혼란
    - 리액트는 스타일을 입힐 수 있는 방법도, 상태 관리 방법도 굉장히 다양한 옵션이 있다. css, fetch 방법, 상태 관리 등 다양한 것들이 파편화돼있다. Anglular나 Vue처럼 하나를 배워두면 서로 다른 곳에서도 빠르게 쓸 수 있는 구조가 아니라 리액트 외에 다른 것들은 새롭게 해워야 한다.

## 오픈소스 생태계의 명과 암

### 페이스북 라이언스 이슈

오픈소스 라이선스 중 가장 많이 쓰이는 라이선스는 MIT다. 이 라이선스는 소프트웨어를 상업적으로 이용하거나, 배포하거나, 개인적으로 이용하는 등에 대한 어떠한 제약 없이 소프트웨어를 취급할 수 있는 매우 자유로운 라이선스다.

하지만 페이스북은 자사의 오픈소스들을 MIT 대신 BSD+Patents 라이선스를 사용하고 있었다.

- 이 라이선스는 `이 라이선스를 적용한 소프트웨어에 대해서 특정한 사건이 발생한다면 라이선스가 통지 없이 종료될 수 있다` 라는 한 가지 눈에 띄느니 조항이 있다.

2017년 7월 아파치 재단은 BSD+Patents 라이선스를 사용하는 것을 금지한다고 밝혔고, 페이스 북은 이 조항을 철회할 생각이 없다고 밝혔다가 많은 격론 끝에 결국 MIT 라이선스로 변경하였다.

### 오픈소스는 무료로 계속 제공될 수 있는가? colors.js, faker.js, 바벨

바벨은 js 컴파일러로, 자바스크립트 최신 문법을 지원하지 않는 브라우저에서도 최신 문법을 사용할 수 있도록 js를 컴파일해주는 도구다.

바벨은 오픈소스 프로젝트이지만, 풀타임 개발자들을 고용해 급여를 주고 있기 때문에 2021년 5월 재정난을 겪고 있다는 글을 올렸었다. 이를 위한 모금도 계속되고 있다.

반면 colors.js는 1.4.0 버전에서 1.4.1 버전으로 패치 업데이트를 하면서 고의로 무한 루프를 삽입한 코드를 커밋해버린 사건이 있었고, faker.js는 5.5.3 버전에서 갑자기 6.6.6으로 넘어갔는데, 이 6.6.6 버전에는 아무런 코드가 남아있지 않았던 사건이 있었다.

프론트엔드 개발은 거의 제로에 가까운 비용으로 앱을 만들고 배포에 사용할 수 있는 환경이다. create-react-app 라이브러리만 해도 dependencies로 물고 있는 라이브러리는 10개 남짓에 불과하지만 정작 node_modules에 설치되는 라이브러리는 700여개에 이른다.

우리는 이 700여개의 라이브러리를 npm 오픈소스 생태계 덕분에 돈 한푼 안들이고 사용하고 있다.

프론트엔드 개발자들은 오픈소스 덕분에 손쉽게 개발하고 있음을 알고 있어야 하고, 반대로 오픈소스가 무슨 일을 하고 있는지를 알 필요도 있다.

### 웹 개발자로서 가져야 할 유연한 자세

무슨 프레임워크나 라이브러리를 사용하든 한 가지 변하지 않는 사실은 HTML과 CSS, JS가 웹 페이지를 구성하는 기초 기술이라는 사실이다. 물론 웹어셈블리와 같이 웹을 구성하는 완전히 새로운 개념이 도입될 수도 있지만, 브라우저가 과거 웹까지 안정적으로 보여주기 위해 많은 노력을 기울이고 있다는 점, JS 또한 마찬가지로 호환성을 유지하기 위해 많은 노력을 기울인다는 점을 보면 당분간 이 사실은 변함이 없을 것이다.

최근 느린 자바스크립트를 대신할 하나의 방안으로 웹어셈블리가 떠오르고 있다.

- 웹 어셈블리는 C, C++, 러스트 같은 시스템 프로그래밍 언어로 작성돼 있기 때문에 일반적으로 웹에서 JS 기반으로 처리하기 어려운 작업을 이를 활용해 처리할 수 있다.
- 때문에 개발자들 사이에서 러스트를 배우는 게 유행처럼 떠올랐지만, 웹 어셈블리는 JS를 대체하는 게 아닌, JS와 함께 상호보완적으로 실행되는 도구라보 고는 게 적절하다.
  - 성능이 필요한 작업은 웹어셈블리로, 그 밖의 일반적인 작업은 JS로

라이브러리와 프레임워크는 그저 도구일 뿐, JS가 토대라는 사실에는 변함이 없다.
