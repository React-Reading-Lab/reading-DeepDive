# 리액트에서 발생하는 크로스 사이트 스크립팅(XSS)

웹사이트 개발자가 아닌 제 3자가 악성 스크립트를 삽입해 실행할 수 있는 취약점이다. 쿠키를 회득해 사용자 로그인 세션을 탈취하거나 사용자 데이터를 변경하는 스크립트를 실행할 수 있다.

이제 어떤 요소가 XSS를 발생하게 할 수 있는지 알아보자. 

## dangerouslySetInnerHTML props

이 속성은 브라우저 DOM의 innerHTML을 특정한 내용으로 교체할 수 있는 방법이다. 

```jsx
<div dangerouslySetInnerHTML={{ _html: "<span>안녕</span>" }} /> // <div><span>안녕</span></div>
```

<aside>
👉

기본적으로 리액트는 이스케이프 작업을 해주고 이 속성은 이스케이프 작업이 일어나지 않기 때문에 dangerously가 붙는 것이다.

</aside>

## useRef를 활용한 직접 삽입

위 속성처럼 useRef를 이용해 DOM에 직접 접근해 innerHTML로 DOM 내용을 직접 삽입할 수 있는 방법도 있다.

```jsx
divRef.current.innerHTML = html
```

## 리액트에서 XSS 문제 피하기

제 3자가 삽입할 수 있는 HTML을 안전한 코드로 바꾸는 과정을 거치면 된다. 이를 sanitize라고 한다. 라이브러리를 사용해 진행하면 된다.

```jsx
import sanitizeHTML, { IOptions as SanitizeOptions } from "sanitize-html"

const allowedTags = [
	"div",
	"p",
	"span",
	...
];

const sanitizedOptions: SanitizeOptions = {
	allowedTags
};

...

<div dangerouslySetInnerHTML={{ _html: sanitizeHTML(html, sanitizedOptions) }} />
```

# getServerSideProps와 서버 컴포넌트 주의하자

getServerSideProps가 반환하는 props는 모든 사용자의 HTML에 기록되기 때문에 getServerSideProps가 반환하는 값 또는 서버 컴포넌트가 클라이언트 컴포넌트에 반환하는 props는 반드시 필요한 값만 철저히 제한되어야 한다.

```jsx
const getServerSideProps = async (ctx) => {
	const cookie = ctx.req.headers.cookie || "";
	
	// bad
	return {
		props: {
			cookie,
		}
	}
	
	// good
	
	const token = validateCookie(cookie);
	
	if (!token) {
		return {
			redirect: { ... }
		}
	}
	
	return {
		props: {
			token,
		}
	}
}
```

# <a> 태그 값에 적절한 제한 두기

a 태그는 href 안에 자바스크립트가 들어가면 실행된다. 그런데 이는 react에서도 미래에 막기로 했고 안티패턴으로 해석된다. a 태그의 href안에 사용자가 값을 넣을 수 있다면 보안 문제가 발생할 수 있기 때문이다.

그래서 href로 들어갈 수 있는 값은 제한하는 게 좋고, 사이트로 이동하는 것을 막기 위해 orgin을 확인하는 것도 좋다.

```jsx
function isSafeHref(href: string) {
	let isSafe = false
	try {
		const url = new URL(href)
		if(["http", "https"].includes(url.protocol)){
			isSafe = true
		}
	} catch {
		isSafe = false
	}
	
	return isSafe
}
```

# HTTP 보안 헤더 설정하기

## Strict-Transport-Security

모든 사이트가 HTTPS를 통해 접근해야 하며, HTTP로 접근하면 HTTPS로 변경하는 헤더다.

expire-time으로 브라우저가 이 설정을 기억해야하는 시간을 기록할 수 있다. (권장값은 2년)

includeSubDomains은 하위 도메인에도 적용됨

```python
Strict-Transport-Security: max-age=<expire-time>; includeSubDomains
```

## X-XSS-Protection

비표준 기술로 현재 사파리와 구형 브라우저에서만 제공됨

XSS 취약점이 있으면 페이지 로딩을 중단하는 헤더. 그렇지만 이 헤더를 믿지말고 직접 XSS를 막을 수 있는 방법을 생각해야 함.

```python
X-XSS-Protection: 0 // xss 필터를 끈다
X-XSS-Protection: 1 // 필터를 키고 xss관련 코드를 제거한 안전한 페이지를 보여준다
X-XSS-Protection: 1; mode=block // 접근을 막는다
X-XSS-Protection: 1; report=<reporting-uri> // 크로미움 기반 브라우저에서만 작동. xss 감지하면 report함
```

## X-Frame-Options

페이지를 frame, iframe, embed, object 내부에서 렌더링을 허용할지 나타내는 것

```python
X-Frame-Options: DENY // 무조건 막기
X-Frame-Options: SAMEORIGIN // 같은 origin에 대해서만 프레임 허용
```

## Permissions-Policy

웹사이트에서 사용할 수 있는 기능과 없는 기능을 명시적으로 선언하는 헤더

사용하지 않는 기능을 명시해 XSS가 발생했을 때 사용하지 않는 기능의 권한은 뺏어갈 수 없게 할 수 있음

## X-Content-Type-Options

> Mine: Multipurpose Internet Mail Extensions → 메일 전송에 쓰던 인코딩 방식
> 

Content-type 헤더에서 제공하는 Mine 유형이 브라우저에 의해 임의로 변경되지 않게 하는 헤더

## Referrer-Policy

referer는 헤더에서 현재 요청을 보낸 페이지의 주소가 나타난다.

이 헤더는 사용자가 어디서 와서 방문 중인지 인식할 수 있는 헤더다.

Referrer-policy는 이 헤더에서 사용할 수 있는 데이터를 나타낸다.

## Content-Security-Policy

XSS 공격이나 데이터 삽입 공격과 같은 다양한 보안 위혐을 막기 위해 설계되었다.

```python
Content-Security-Policy: font-src <source> // source에 있는 것만 사용 가능
```

# 취약점이 있는 패키지 사용을 피하자

dependabot에서 확인하자

# OWASP TOP 10

1. 사용자가 권한 밖의 행동을 할 수 있는 취약점
2. 암호화 실패
3. 사용자가 제공하는 데이터를 조작한 공격
4. 기획 설계 단계에서 발생하는 보안 취약점
5. 보안 설정 오류로 취약점 발생
6. 취약점이 있거나 지원이 종료된 소프트웨어 사용
7. 사용자 신원 확인에 실패하거나 암호 생성 정책이 없는 경우 등 인증 관련 보안 취약점을 말한다.
8. 소프트웨어와 데이터 무결성 오류
9. 보안 로깅 및 모니터링 안되어 있음
10. 서버 측 요청 변조

---

1. 리액트 버전을 최소 16.8.6에서 최대 17.0.2로 올려두자
    
    그렇다고 클래스 컴포넌트를 함수 컴포넌트로 리팩토링할 필요는 없다
    
2. 인터넷 익스플로러 11 지원을 목표로 한다면 각별히 주의한다
    
    리액트 18, Next.js 13, query-string 6.x는 모두 인터넷 익스플로러를 지원하지 않는다.
    
3. 서버 사이드 렌더링 애플리케이션을 우선적으로 고려한다
    
    많은 사용자를 감당해야 하고, 혹은 그럴 계획이 있다면 서버 사이드 렌더링부터 고려하는 게 좋다.
    
4. 상태 관리 라이브러리는 꼭 필요할 때만 사용한다
5. 리액트 의존성 라이브러리 설치를 조심한다
    
    peerDependencies가 설치하고자 하는 프로젝트의 리액트 버전과 맞는지 확인한다.
    

리액트는 가장 널리 쓰이는 프론트엔드 라이브러리는 맞지만 그렇다고 가장 완벽한 라이브러리라고 단정 짓기는 어렵다.

1. svelte만봐도 리액트보다 코드가 쉬움. 함수형으로 넘어오면서 혼란을 야기함
2. 너무 자유가 많이 때문일 수도.(상태 라이브러리, 스타일 라이브러리)

리액트 중심적인 사고는 위험하다!

리액트는 BSD+Patents 라이선스를 사용하는데 특정한 사건이 발생하면 라이선스가 통지 없이 종료될 수 있다. 사람들의 반발이 심해서 나중에 MIT 라이선스로 넘어갔다.

프론트엔드 개발자들은 오픈소스 덕분에 쉽게 개발하고 있음을 알아야 하고 반대로 오픈소스가 무슨 일을 하고 있는지를 알 필요도 있다.

또한 프레임워크는 언제 사라질지도 모르는 일이기 때문에 우리는 리액트가 아닌 웹을 잘 알아야 하는 것.
