# 좋은 리액트 코드 작성을 위한 환경 구축하기

## ESLint를 활용한 정적 코드 분석

정적 코드 분석이란 코드의 실행과는 별개로 코드 그 자체만으로 코드 스멜(잠재적으로 버그를 야기할 수 있는 코드)을 찾아내어 문제의 소지가 있는 코드를 사전에 수정하는 것을 의미한다.

JS 생태계에서 가장 많이 사용되는 정적 코드 분석 도구는 바로 ESLint다. 

### ESLint는 어떻게 코드를 분석할까?

ESLint는 JS 코드를 정적 분석해 잠재적인 문제를 발견하고 나아가 수정까지 도와주는 도구다. 

ESLint의 작동 순서는 다음과 같다.

1. JS 코드를 문자열로 읽는다.
2. JS 코드를 분석할 수 있는 파서(parser)로 코드를 구조화한다.
3. 2번에서 구조화한 트리를 AST라고 하며, 이 구조화된 트리를 기준으로 각종 규칙과 대조
4. 규칙과 대조했을 때 이를 위반한 코드를 알리거나(report) 수정한다.(fix)

JS를 분석하는 파서에는 여러 가지가 있는데, ESLint는 기본값으로 espree를 사용한다. 

`function hello(str) {}`

위 코드를 `espree`로 분석하면 다음과 같이 JSON 형태로 구조화된 결과를 얻을 수 있다.

```jsx
{
	"type" : "Program",
	"start" : 0,
	"end" : 22,
	"range" : [0, 22],
	"body" : [
		"type" : "FunctionDeclaration",
		"end" : 22,
		"range" : [0, 22],
		"id" : {
			"type" : "Identifier",
			...
			"name" : "hello"
			...
		},
		...
		...
	],
	"sourceType" : "module"
}
```

이처럼 한 줄 밖에 안되는 단순한 코드라도 JSON으로 생성된 트리에 다양한 정보가 담겨 있음을 확인할 수 있다. espree 같은 코드 분석 도구는 단순히 변수인지, 함수인지, 함수명은 무엇인지 등만 파악하는 것이 아니라 코드의 정확한 위ㅣ와 같은 아주 세세한 정보도 분석해 알려준다.

TS의 경우 마찬가지로 `@typescript-eslint/typescript-estree` 라고 하는 espree 기반 파서가 있으며, 이를 통해 타입스크립트 코드를 분석해 구조화하다. 

직접 구조화를 해보고 싶다면 [AST explorer](https://astexplorer.net/)에서 사용해볼 수 있다. 

### eslint-plugin과 eslint-config

`eslint-plugin-` 과 `eslint-config` 는 모두 ESLint와 관련된 패키지지만 각각의 역할이 다르다.

#### eslint-plugin

`eslint-plugin`이라는 접두사로 시작하는 플러그인은 규칙을 모아놓은 패키지다. 

예를 들어,

- `eslint-plugin-import` : JS에서 다른 모듈을 불러오는 import와 관련된 다양한 규칙을 제공한다.
- `eslint-plugin-react` : JSX 배열에 키를 선언하지 않았다는 경고 메시지를 본 적이 있따면 바로 이 `eslint-plugin-react`가 제공하는 규칙 중 하나인 `react/jsx-key`가 경고를 보여준 것이다.
    - eslint는 정적 분석 도구라서 key가 유니크한 값인지 까지는 확인해줄수 없지만, 존재 여부 만으로도 큰 도움을 받을 수 있다.

#### eslint-config

`eslint-plugin`이 리액트, import와 같은 특정 프레임워크나 도메인 같은 규칙을 묶어서 제공하는 패키지라면, `eslint-config`는 이러한 `eslint-plugin`을 한데 묶어서 완벽하게 한 세트로 제공하는 패키지라 할 수 있다. 

- 원하는 규칙들을 한데 모아서 설치하고 적용하는 것도 좋지만 ESLint를 설정하는 것 또한 만만치 않기 때문에, 대부분의 경우 이미 존재하는 `eslint-config`를 설치해서 빠르게 적용하는 경우가 일반적이다.

#### 네이밍 규칙

`eslint-plugin`과 `eslint-config` 의 네이밍에는 한 가지 규칙이 있다. `eslint-plugin, eslint-config` 라는 접두사를 준수해야하며, 반드시 한 단어로 구성해야 한다. 

- eslint-plugin-naver : 가능
- eslint-plugin-naver-financials : 불가능

하지만 특정 스코프가 앞에 붙는 것 까지는 가능하다.

- `@titicaca/eslint-config-triple`은 가능한 것이다.

#### 라이브러리 종류

- eslint-config-airbnb
    - 에어비앤비에서 만들었으며, 에어비앤비 개발자뿐만 아니라 500여 명의 수많은 개발자가 유지보수하고 있는 단연 가장 유명한 `eslint-config`다.
- @titicaca/triple-config-kit
    - 한국 커뮤니티에서 운영되는 eslint-config 중 하나. 유지보수가 활발한 편에 속한다.
    - 대부분의 eslint-config가 `eslint-config-airbnb`를 기반으로 약간의 룰을 수정해 배포되고 있는 것과 다르게 해당 패키지는 자체적으로 정의한 규칙을 기반으로 운영된다.
    - 외부로 제공하는 규칙에 대한 테스트 코드가 존재한다. eslint-config를 사용하는 개발자가 규칙을 수정하거나 추가할 때, 기대하는 바대로 eslint-config-triple에서 규칙이 추가됐는지 확인할 수 있다.
- eslint-config-next
    - 리액트 기반 next.js 프레임워크를 사용하고 있는 프로젝트에서 사용할 수 있는 eslint-config다.
    - 이 라이브러리에서 눈여겨볼 점은 단순히 JS 코드를 정적으로 분석할 뿐만 아니라 페이지나 컴포넌트에서 반환하는 jsx 구문 및 _app, _document에서 작성돼 있는 HTML 코드 또한 정적 분석 대상으로 분류해 제공한다는 점이다.
    - 핵심 웹 지표(core web vitals)를 분석해 제공하는 기능도 포함돼 있다.
    - Next.js로 작성된 코드라면 반드시 설치하느게 좋다.

## 나만의 ESLint 규칙 만들기

eslint-config나 eslint-plugin에서 제공하고 있지 않은 규칙을 코드를 수정해야하는 경우가 있다. 이 경우 파일 내부에서 코드를 직접 고치거나 풀 리퀘스트의 코드 리뷰에서 확인해서 수정하는 것도 좋지만, 사람이 일일이 확인하는 순간 비효율적이고 실수가 많아지게 된다.

이런 경우 나만의 ESLint 규칙을 생성해서 관리하면 개발자가 일일이 수정하는 것보다 훨씬 더 빠르게 쉽게 구성할 수 있고, 이후에 반복되는 실수 또한 방지할 수 있어 매우 유용하다. 

### 이미 존재하는 규칙을 커스텀해서 적용하기 : import React를 제거하기 위한 ESLint 규칙 만들기

리액트 17 버전부터는 새로운 JSX 런타임 덕분에 `import React` 구문이 필요 없어졌다. 이에 따라 해당 구문을 삭제하면 아주 약간이나마 번들러의 크기를 줄일 수 있다.

- `import React` 를 포함하면 웹팩에서 트랜스파일링 시 react__WEBPACK_IMPORTED_MODULE_0__ 이라는 변수를 선언하는데, 어디에서도 쓰지 않는다. `import React`를 삭제해주면 해당 라인이 사라져 그만큼 번들러 크기가 줄어드는 것.
    - (웹팩의 트리쉐이킹이 사용하지 않는 변수 등을 전부 정리해주긴 하지만, 트리쉐이킹 대상이 줄어든다는 점에서 여전히 이득이다.)
- 이제 기존 규칙을 커스터마이징하여 해당 이슈를 감지해보자. 여기서 사용할 규칙명은 `no-restricted-imports` 다.
    - 이 규칙은 어떤 모듈을 import 하는 것을 금지하기 위해 만들어진 규칙이다.

```jsx
module.exports = {
	rules : {
		'no-restricted-imports' : [
			'error',
			{
				// paths에서 금지시킬 모듈을 추가한다.
				paths : [
					// 모듈명
					name : 'react',
					// 모듈의 이름
					importNames : ['default'],
					// 경고 메시지 
					message : "import React from 'react'는 react 17부터 더 이상 필요하지 않음",
				],
			},
		],
	}
}
```

- 위 규칙을 넣은 후 `import React` 가 있는 코드에서 ESLint를 실행하면 ‘message’에 넣은 경고문이 출력된다.
- 이런 원리를 사용하면 트리쉐이킹이 되지 않는 `lodash` 같은 라이브러리를 import 하는 것도 방지할 수 있다. (lodash는 CommonJS로 작성돼 있어 트리쉐이킹이 되지 않아 번들 사이즈가 큼. lodash/* 형식으로 import하는 것을 추천)

### 완전히 새로운 규칙 만들기 : new Date를 금지시키는 규칙

JS 환경에서는 시간을 알기 위해 `new Date()`를 사용하곤 한다.  하지만 **이 현재 시간**은 기기에 종속된 현재 시간으로, 기기의 현재 시간을 바꿔버리면 new Date()가 반환하는 현재 시간 또한 변경된다.

- 단, `new Date('2022-0101')` 같은 형태는 허용해야 할 것이다. 이 경우는 현재 시간을 가져오는 것이 아니라 JS Date 객체를 활용해 특정 시간을 가져오는 것이 목적이기 때문이다.
- 위 규칙을 만들기 위해서는 JS 코드 내부에서 new Date()가 어떻게 AST로 분석되는 지를 알아야 한다.
    
    ```jsx
    {
    	"type" : "Program",
    	"start" : 0,
    	"end" : 10,
    	"range" : [0, 10],
    	"body" : [
    		{
    			"type" : "ExpressionStatement", // 해당 코드의 표현식 전체를 나타냄 
    			"start" : 0,
    			"end" : 10,
    			"range" : [0, 10],
    			// ExpressionStatement.expression : 
    				// ExpressionStatement에 어떤 표현이 들어가 있는지 확인.
    				// ESLint에서 확인하는 하나의 노드 단위.
    			"expression" : { 
    				// ExpressionStatement.expression.type : 
    				// 해당 표현이 어떤 타입인지 나타내는데, 여기에서는 생성자(new)를 사용한 
    				// newExpression임을 알 수 있음.
    				"type" : "NewExpression",
    				"start" : 0,
    				"end" : 10,
    				"range" : [0, 10],
    				// ExpressionStatement.callee : 
    				// 생성자를 사용한 표현식에서 생성자의 이름을 나타냄. Date임을 알 수 있음.
    				"callee" : {
    					"type" : "Identifier",
    					"start" : 4,
    					"end" : 8,
    					"range" : [4, 8],
    					"name" : "Date"
    				},
    				// ExpressionStatement.expression.arguments : 
    				// 생성자를 표현한 표현식에서 생성자에 전달하는 인수. 
    				// 여기에서는 인수가 없다.
    				"arguments" : [],
    			},
    		},
    	],
    	"sourceType" : "module"
    }
    ```
    
    - 즉, `new Date()` 로 현재 시간을 가져오는 코드를 금지하고 싶으면, 아래의 내용으로 규칙을 만들면 된다.
        - type는 `NewExpression`이며,
        - callee.name이 `Date`이고,
        - ExpressionStatement.expression.arguments가 빈 배열
- ESLint의 `create` 함수로 규칙을 만들어보자.
    
    ```jsx
    /**
    * @type {import('eslint').Rule.RuleModule}
    */
    
    module.exports = {
    	// meta 필드는 해당 규칙과 관련된 정보를 나타내는 필드다. 
    	// 실제 규칙이 작동하는 코드와는 큰 관계 없으며, 여기서 사용 가능한 옵션은 
    	// 공식 홈페이지의 meta 필드를 참고하면 된다.
    	meta : {
    		type : 'suggestion',
    		docs : {
    			description : 'disallow use of the new Date()',
    			recommended : false,
    		},
    		fixable : 'code',
    		schema : [],
    		messages : {
    			message : 'new Date()는 클라이언트에서 실행 시 해당 기기의 시간에 의존이라
    				정확하지 않습니다. 현재 시간이 필요하다면 ServerDate()를 사용해 주세요.',	
    		},
    	},
    	
    	// create 함수는 객체를 반환해야 한다. 
    	// 이 객체에서는 코드 스멜을 감지할 선택자나 이벤트명 등을 선언할 수 있다.
    	// 여기서는 NewExpression이라고 하는 타입의 선택자를 키로 선언해,
    	// new 생성자를 사용할 때 ESLint가 실행되게 한다.
    		create : function (context) {
    		return {
    			// ESLint가 실행되어 해당 NewExpression을 찾았을 때, 해당 node를 기준으로
    			// 찾고자 하는 생성자인지 검증하는 코드를 넣는다.
    			NewExpression : function (node) {
    				// 여기서는 callee.name이 Date이고 인수는 없는 경우를 찾는다.
    				if (node.callee.name === 'Date' && node.arguments.length == 0) {
    					// 이를 찾았다면 context.report를 통해 해당 코드 스멜을 리포트하고, 
    					// 문제가 되는 node와 찾았을 때 노출하고 싶은 message를 가리킨다.
    					// 이 메시지 정보는 meta.messages에 선언한 키의 value를 가져오게 된다.
    					context.report({
    						node : node,
    						messageId : 'message',
    						// fix를 키로 하는 함수를 활용해 자동으로 수정하는 코드를 넣어줄 수 있다.
    						// 아래처럼 설정하면 new Date() 구문은 ServerDate() 라는 구문으로
    						// 수정되어 적용된다.
    						fix : function (fixer) {
    							return fixer.replaceText(node, 'ServerDate()')
    						},
    					})
    				}
    			},
    		}
    	},
    }
    ```
    
    - 위 처럼 규칙을 설정했으면 만들어 배포해야 사용할 수 있다.
    - 규칙은 하나씩 만들어 배포할 수 없고, 무조건 `eslint-plugin` 형태로 묶음 배포 해야한다.
    - `eslint-plugin`을 구성할 환경은 `yo` 와 `generate-eslint` 도구 등을 활용할 수 있으며, 자세한 내용은 P487 참고

## 주의할 점

ESLint를 사용하면서 주의해야할 점이 몇 가지 있다.

### Prettier와의 충돌

Prettier는 코드의 포매팅을 도와주는 도구다. ESLint와 마찬가지로 코드를 정적 분석해서 문제를 해결한다는 점은 동일하지만, 두 패키지가 지향하는 목표는 다르다. 

- Prettier :
    - 포매팅과 관련된 직업, 즉 줄바꿈, 들여쓰기, 작은따옴표와 큰 따옴표 등을 담당한다.
    - JS에서만 작동하는 ESLint와는 다르게, Prettier는 JS 뿐만 아니라 HTML, CSS, 마크다운, JSON 등 다양한 언어에도 적용 가능하다.
- 여기서 문제는 Prettier와 ESLint가 **서로 충돌**을 일으킬 수 있다는 것이다. 즉, ESLint에서도 Prettier에서 처리하는 작업(들여쓰기, 줄바꿈, 따옴표, 최대 글자 수 등)을 처리할 수 있기 때문에, 두 가지 모두를 JS 코드에서 실행한다면 서로 충돌하는 규칙으로 인해 에러가 발생하고, 최악의 경우 ESLint, Prettier 모두 만족하지 못하는 코드가 만들어질 수도 있다.
- 이 문제를 해결하는 방법은 두 가지다.
    - 서로 규칙이 충돌되지 않게끔 규칙을 잘 선언하는 것. Prettier에서 제공하는 규칙을 어기지 않도록, ESLint에서는 해당 규칙을 끄는 방법이다.
        - 이 경우 코드에 ESLint를 적용하는 작업과 코드의 포매팅을 하는 작업이 서로 다른 패키지에서 발생하게 됨.
    - 두 번째 방법은 JS나 TS는 ESLint에, 그 외의 파일은 모두 Prettier에 맡기는 것.
        - 대신 JS에 추가적으로 필요한 Prettier 관련 규칙은 모두 `eslint-plugin-prettier` 를 사용한다.
        - `eslint-plugin-prettier`는 Prettier 에서 제공하는 모든 규칙을 ESLint에서 사용할 수 있는 규칙으로 만들어둔 플러그인이다.

### 규칙에 대한 예외 처리, 그리고 `react-hooks/no-exhaustive-deps`

만약 일부 코드에서 특정 규칙을 임시로 제외시키고 싶다면 `eslint-disable-` 주석을 사용하면 된다.

특정 줄만 제외하거나, 파일 전체를 제외하거나, 특정 범위에 걸쳐 제외하는 것이 가능하다.

```jsx
// 특정 줄만 제외
console.log('hello world'); // eslint-disable-line no-console

// 다음 줄 제외
// eslint-disable-next-line no-console
console.log('hello world');

// 특정 여러 줄 제외
/* eslint-disable no-console */
console.log('JavaScript debug log');
console.log('eslint is disable now');
/* eslint-enable no-console */

// 파일 전체에서 제외
/* eslint-disable no-console */
console.log('hello world');
```

리액트 개발자라면 `// eslint-disable-line no-exhaustive-deps` 을 가장 많이 사용할  것이다. 

- 이 규칙은 `useEffect`나 `useMemo`와 같이 의존 배열이 필요한 훅에 의존성 배열을 제대로 선언했는지 확인하는 역할을 한다.
- 겉보기에는 굉장히 별거 아닌 규칙처럼 보이지만 이 규칙을 위해 작성된 코드는 자그마치 1,800여 줄에 걸쳐 있다.
- 하지만 위 결정은 대부분의 경우 위험을 야기한다.
    - 괜찮다고 임의로 판단 : 실제로 면밀히 검토해서 괜찮은 경우라면 해당 변수는 컴포넌트의 상태와 별개로 동작한다는 것을 의미한다. 이 경우 해당 변수를 어디서 어떻게 선언할 지 다시 고민해봐야 한다.
    - 의존성 배열이 너무 긴 경우 : 의존성 배열이 너무 길다는 것은 `useEffect` 내부 함수가 너무 길다는 말과 동일하다. `useEffect`가 너무 길다면 `useEffect` 를 쪼개서 의존성 배열의 가독성과 안정성을 확보해야 한다.
    - 마운트 시점에 한 번만 실행하고 싶은 경우 :
        - 가장 흔히 볼 수 있는 경우로, 의도적으로 []로 모든 의존성을 제거해 컴포넌트가 마운트되는 시점에만 실행하고 싶은 경우다.
        - 이러한 접근 방법은 과거 클래스 컴포넌트에서 사용되던 생명주기 형태의 접근 방법으로, 함수 컴포넌트의 패러다임과는 맞지 않을 가능성이 있다. 또한, 상태와 관계없이 한 번만 실행돼야 하는 것이 있다면 해당 컴포넌트에 존재할 이유가 없다. 이 경우 적절한 위치로 옮기자.
    - 물론 정말 넣을 것이 없어서 []를 넣는 경우는 당연히 제외다. 여기서 말하는 경우는 **상태에 의존하고 있음에도 고의로 빈 배열을 넣는 경우**다.
    

### ESLint 버전 충돌

- (책 발간 연도 기준) create-react-app을 실행하면 설치되는 `react-scripts` 의 5.0.1 버전에는 `ESLint 8`에 의존성을, `eslint-config-triple`은 `ESLint 7`에 의존성을 가지고 있다.
- ESLint가 실행되는 순간 높은 버전인 8이 실행됐고, 8에는 `eslint-plugin-promise'가 없기 때문에 에러가 발생한다. 이 때문에 공식 문서에서는 ESLint를 `peerDependencies`로 설정해 두라고 권장하고 있다.
- 이러한 문제를 미연에 방지하려면 설치하고자 하는 `eslint-config, eslint-plugin`이 지원하는 ESLint 버전을 확인하고, 또 설치하고자 하는 프로젝트에서 ESLint 버전을 어떻게 지원하고 있는지 살펴봐야 한다.
- 최근 `eslint-config-triple` 의 4.x 버전이 릴리스되면서 이러한 문제가 어느 정도 해결된 것으로 보인다.
    - ESLint 공식 문서에서도 ESLint의 의존성은 `peerDependencies`로 명시하도록 권장하고 있찌만, 몇몇 이를 준수하지 못한 패키지를 설치할 때는 주의하자.

## 리액트 팀이 권장하는 리액트 테스트 라이브러리

테스트란 개발자가 만든 프로그램이 코딩을 한 의도대로 작동하는지 확인하는 일련의 작업을 의미한다. 

백엔드의 테스트는 일반적으로 아래의 항목 중점이다.

- 서버나 데이터베이스에서 원하는 데이터를 올바르게 가져올 수 있는지
- 데이터 수정 간 교차 상태가 경쟁 상태가 발생하지는 않는지
- 데이터 손실은 없는지
- 특정 상황에서 장애가 발생하지 않는지
- 이처럼 일반적으로 화이트박스 테스트로, 작성한 코드가 의도대로 작동하는지 확인해아 한다.
    - 이는 GUI가 아닌 AUI(Application User Interface; 응용 프로그램 사용자 인터페이스)에서 주로 수행해야 하기 때문에 어느 정도 백엔드에 대한 이해가 있는 사람만 가능하다.

반면 프론트엔드는 일반적인 사용자와 동일하거나 유사한 환경에서 수행된다.

- 사용자가 프로그램에서 수행할 주요 비즈니스 로직이나 경우의 수를 고려
    - 즉, 블랙박스 형태로 테스트를 진행.
    - 시나리오가 어느정도 정해져있는 백엔드와 다르게 사용자에게 완전히 노출된 영역이므로 어떻게 작동할지 최대한 예측해서 확인해야 함
- FE 개발은 HTML, CSS 같이 디자인 요소뿐만 아니라 사용자의 인터랙션, 의도치 않은 작동 등 브라우저에서 발생할 수 있는 다양한 시나리오를 고려해야 함.
    - 일반적으로 테스팅이 번거롭고 손이 많이 감.
    - 하지만 그만큼 다양한 테스팅 라이브러리가 있음

## React Testing Library란?

`DOM Testing Library`를 기반으로 만들어진 테스팅 라이브러리로, 리액트를 기반으로 한 테스트를 수행하기 위해 만들어짐.

### DOM Testing Library?

- jsdom을 기반으로 하고 있음
    - jsdom이란 순수하게 JS 스크립트로 작성된 라이브러리로, html이 없는 JS만 존재하는 환경, 예를 들면 Node.js 같은 환경에서 HTML과 DOM을 사용할 수 있도록 해주는 라이브러리다.
    - jsdom을 활용하면 JS 환경에서도 HTML을 사용할 수 있으므로 이를 기반으로 `DOM Testing Library`에서 제공하는 API를 사용해 테스트를 수행할 수 있다.

```jsx
const jsdom = require('jsdom');
const { JSDOM } from jsdom;

const dom = new JSDOM(`<!DOCUMENT html><p>Hello world</p>`);

console.log(dom.window.document.querySelector('p').textContent);
```

- 이처럼 jsdom을 활용해 JS 환경에서 HTML을 사용할 수 있는 DOM Tesiting Library와 비슷한 원리로, 리액트 기반 환경에서 리액트 컴포넌트를 테스팅 할 수 있는 것이 바로 React Testing Library다. 되어 ㅍ
    - 실제로 리액트 컴포넌트를 렌더링하지 않고도 컴포넌트가 의도대로 작동하고 있는 지 확인 가능
    - 굳이 테스트 환경을 구축하지 않아도 편리
    - Provider, hook 등 리액트를 구성하는 다양한 요소들 테스트 가능

### JS 테스트의 기초

테스트 코드란 내가 작성한 코드가 **내가 코드를 작성했던 당시의 의도와 목적에 맞는지 확인하는 코드**를 의미한다. 예를 들면 아래의 sum 함수는 실제로 기대하는 결과가 반환되는 지 확인하는 게 좋을 것이다.

```jsx
// sum 함수
function sum(a, b) {
	return a+b;
}

// 테스트1
// 함수를 실행했을 때의 실제 결과
let actual = sum(1, 2);
// 함수를 실행했을 때 기대하는 결과
let expected = 3;

if (expected !== actual) {
	throw new Error(`${expected} is not equal to ${actual}`);
}

// 테스트2
actual = sum(2, 2);
expected = 4;

if (expected !== actual) {
	throw new Error(`${expected} is not equal to ${actual}`);
}
```

이처럼, 기본적인 테스트 코드를 거치는 방법을 요약하면 다음과 같다.

1. 테스트할 함수나 모듈을 선정한다.
2. 함수나 모듈이 반환하길 기대하는 값을 적는다.
3. 함수나 모듈의 실제 반환 값을 적는다.
4. 3번의 기대에 따라 2번의 결과가 일치하는지 확인한다.
5. 기대하는 결과를 반환한다면 테스트는 성공이며, 만약 기대와 다른 결과를 반환하면 에러를 던진다.

Node.js는 `assert`라는 모듈을 기본적으로 제공한다. 이 모듈을 사용하면 위와 같이 작동하도록 만들 수 있다. (일반적으로 테스트 코드와 실제 코드는 분리해서 작성)

```jsx
// Node.js에서 기본적으로 제공하는 assert를 사용한 함수

const assert = require('assert');

function sum(a, b) {
	return a+b;
}

assert.equal(sum(1, 2), 3);
assert.equal(sum(2, 2), 4);
assert.equal(sum(1, 2), 4); // AssertionError [ERR_ASSERTION] [ERR_ASSERTION]: 3 == 4
```

이처럼, 테스트 결과를 확인할 수 있도록 도와주는 라이브러리를 `어설션(assertion) 라이브러리` 라고 함.

- 위 테스트의 경우 equal 함수를 통해 어느정도 테스트 코드의 목적을 달성했다고 볼 수 있으나, 테스트 코드가 실행되는 것을 지켜보는 입장은 또 다르다.
    - 좋은 테스트 코드는 다양한 테스트 코드가 작성되고 통과하는 것뿐만 아니라, **어떤 테스트가 무엇을 테스트하는지 일목요연하게 보여주는 것도 중요**하다.
    - 이러한 테스트의 기승전결을 완성해 주는 것이 바로 **테스팅 프레임워크**다.

#### 테스팅 프레임워크

- **테스팅 프레임워크**들은 어설션을 기반으로 테스트를 수행하며, 여기에 추가로 테스트 코드 작성자에게 도움이 될 만한 정보를 알려주는 역할도 함께 수행한다.
- JS에서 유명한 테스팅 프레임워크로는 Jest, Mocha, Karma, Jasmine 등이 있다.
- 리액트 진영에서는 리액트와 마찬가지로 메타에서 작성한 오픈소스 라이브러리인 `Jest`가 널리 쓰인다.
- Jest의 경우 자체적으로 제작한 expect 패키지를 사용해 어설션을 수행한다.

```jsx
// math.js
function sum(a, b) {
	return a+b;
}

module.exports = {
	sum,
}
```

```jsx
// math.test.js
const { sum } = require('./math');

test('두 인수가 덧셈이 되어야 한다.', ()=>{
	expect(sum(1, 2)).toBe(3)
});

test('두 인수가 덧셈이 되어야 한다.', ()=>{
	expect(sum(2, 2)).toBe(3) // 에러
});
```

- 위 테스트를 수행하면 앞에서 `assert`를 사용했던 것과는 다르게 테스트 관련 정보를 다양하게 확인할 수 있다.
- Jest를 비롯한 테스팅 프레임워크를 사용하면 `무엇을 테스트했는 지`, `소요된 시간은 어느 정도인지`, `무엇이 성공하고 실패했는지`, `전체 결과는 어떤지`에 대한 자세한 정보를 알 수 있다.
- 그 외 특이한 점은 **test, expect 등의 메소드를 import나 require 같은 모듈을 불러오기 위해 사용하는 구문 없이 바로 사용**했다는 것, 그리고 **node가 아닌 jest로 실행한다는 점**이다.
    - 해당 코드를 jest가 아닌 node로 실행한다면 에러가 발생하는데, 그 이유는 test와 expect 모두 Node.js 환경의 global, 즉 전역 스코프에 존재하지 않기 때문이다.
    - Jest를 비롯한 테스팅 프레임워크에는 이른바 글로벌(global)이라 해서 실행 시에 전역 스코프에 기본적으로 넣어주는 값들이 있다.
        - Jest는 이 값을 실제 테스트 직전에 미리 전역 스코프에 넣어준다.
        - 이렇게 하면 일일이 테스트에 관련한 정보를 임포트하지 않고도 사용할 수 있게 된다. → 이는 간결하고 빠른 테스트 코드 작성에 도움을 준다.
        - 만약 node로 실행하고 싶으면 `import`로 사용한 메소드들을 `@jest/globals`에서 불러오면 되지만, 테스트 코드 작성을 번거롭게 하므로 선호되지 않는다.

## 리액트 컴포넌트 테스트 코드 작성하기

기본적으로 리액트에서 컴포넌트 테스트는 다음과 같은 순서로 진행된다.

1. 컴포넌트를 렌더링한다.
2. 필요하다면 컴포넌트에서 특정 액션을 수행한다.
3. 컴포넌트 렌더링과 2번 액션을 통해 기대하는 결과와 실제 결과를 비교한다.

`npx create-react-app react-test --template typescript` 로 리액트를 설치하면 `App.test.tsx` 파일이 처음부터 만들어져 있다. (create-react-app에는 이미 react-testing-library가 포함돼 있다.)

리액트 컴포넌트에서 테스트하는 일반적인 시나리오는 **특정한 무언가를 지닌 HTML 요소가 있는지 여부**다. 이를 확인하는 방법은 크게 3가지다.

- getBy… :
    - 인수의 조건에 맞는 요소를 반환하며, 해당 요소가 없거나 두 개 이상이면 에러를 발생시킨다. 복수 개를 찾고 싶다면 getAllBy…
- findBy… :
    - getBy… 와 거의 유사하나 한 가지 큰 차이점은 Promise를 반환한다는 것이다. 즉, 비동기로 찾는다는 것을 의미한다.
    - 기본값으로 1000ms의 타임아웃을 가지고 있다. 마찬가지로 두 개 이상이면 에러를 발생시킨다. 복수 개를 찾고 싶다면 findAllBy… 사용.
    - 비동기 액션 이후에 요소를 찾을 때 사용한다.
- queryBy… :
    - 인수의 조건에 맞는 요소를 반환하는 대신, 찾지 못한다면 null을 반환.
    - getBy… 와 findBy… 는 찾지 못하면 에러를 발생시키기 때문에 찾지 못해도 에러를 발생시키고 싶지 않다면 해당 방법을 사용하면 된다.
    - 마찬가지로 복수개를 찾았을 때는 에러를 발생시키며, 복수 개를 찾고 싶다면 queryAllBy… 를 사용하면 된다.

컴포넌트를 테스트하는 파일은 App.tsx, App.test.tsx 의 경우와 마찬가지로 같은 디렉터리상에 위치하는 것이 일반적이다. 

이름 규칙인 `*.ㅅㄷㄴㅅ.{t|s}jsx만 준수한다면 디렉터리 내부에서 명확하게 구별되고, 대부분의 프레임워크가 이러한 이름으로 된 파일은 번들링에서 제외하므로 유용하게 사용할 수 있다.

#### 정적 컴포넌트

정적 컴포넌트, 즉 별도의 상태가 존재하지 않아 항상 같은 결과를 반환하는 컴포넌트를 테스트하는 방법은 크게 어렵지 않다.

테스트를 원하는 컴포넌트를 렌더링한 다음, 테스트를 원하는 요소를 찾아 원하는 테스트를 수행하면 된다. 

예를 들어, `ul`로 만든 `li`에 `<a>` 로 링크를 입힌 정적 컴포넌트에 링크가 제대로 있는지 확인하려면 다음과 같이 테스트 코드를 작성해볼 수 있을 것이다.

```jsx
import { render, screen } from '@testing-library/react'

import StaticComponent from './index'

beforeEach(()=>{
	render(<StaticComponent />)
});

describe('링크 확인', ()=>{
	it('링크가 3개 존재한다.', ()=>{
		const ul = screen.getByTestId('ul');
		expect(ul.children.length).toBe(3);
	});

	it('링크 목록의 스타일이 square다.', ()=>{
		const ul = screen.getByTestId('ul');
		expect(ul).toHaveStyle('list-sytle-type: square;');
	});
});

describe('리액트 링크 테스트', ()=>{
	it('리액트 링크가 존재한다.', ()=>{
		const reactLink = screen.getByText('리액트');
		expect(reactLink).toBeVisible()
	});

	it('리액트 링크가 올바른 주소로 존재한다.', ()=>{
		const reactLink = screen.getByText('리액트');
		expect(reactLink.tagName).toEqual('A');
		expect(reactLink).toHaveAttribute('href', 'https://reactjs.org');
	});
});

describe('블로그 링크 테스트', ()=>{
	it('블로그 링크가 존재한다.', ()=>{
		const blogLink = screen.getByText('블로그');
		expect(blogLink).toBeVisible()
	});

	it('블로그 링크가 올바른 주소로 존재한다.', ()=>{
		const blogLink = screen.getByText('블로그');
		expect(blogLink.tagName).toEqual('A');
		expect(blogLink).toHaveAttribute('href', 'https://yceffort.kr');
	});
	
	it('블로그는 같은 창에서 열려야 한다.', () => {
		const blogLink = screen.getByText('블로그');
		expect(blogLink).not.toHaveAttribute('target');
	})
});
```

- beforeEach : 각 테스트(it)를 수행하기 전에 실행하는 함수다. 여기서는 각 테스트를 실행하기에 앞서 Static Component를 렌더링한다.
- describe : 비슷한 속성을 가진 테스트를 하나의 그룹으로 묶는 역할을 한다.  꼭 필요한 메서드는 아니다. describe 안에 또 describe를 사용할 수 있다.
- it : test와 완전히 동일하며, test의 축약어(alias)이다. `describe ... it` 형태로 자연스럽게 읽히도록 작성된 명칭이다.
- testId : 리액트 테스팅 라이브러리의 예약어다. get 등의 선택자로 선택하기 어렵거나 곤란한 요소를 선택하기 위해 사용할 수 있다. HTML의 DOM 요소에 testId 데이터셋을 선언해 두면, 이후 테스트 시에 `getByTestId`, `findByTestId` 등으로 선택할 수 있다. 웹에서의 `querySelector([data-testid="${yourId}"]` 와 동일한 역할을 한다.

요약하자면 각 테스트를 수행하기 전에 `StaticComponent`를 렌더링하고, `describe`로 연관된 테스트를 묶어서 `it`으로 **it 함수 내부에 정의된 테스트를 수행하는 테스트 파일이라고 정의**할 수 있다.

#### 동적 컴포넌트

상태값이 있는 컴포넌트는 사용자의 액션에 따라 state 값이 변경된다. 이 경우의 테스트 방법에 대해 알아보자.

예시로 사용할 사용자 정의 컴포넌트인 `InputComponent`는 사용자의 키보드 타이핑 입력을 받는 input과 이를 alert로 띄우는 button으로 구성된 간단한 컴포넌트다. input은 최대 20자까지 영문과 숫자만 입력하도록 제한돼 있다. 버튼은 글자가 없으면 disabled 되도록 처리돼 있고, 클릭 시 alert 창을 띄운다.

이 컴포넌트를 테스트위한 코드는 다음과 같을 수 있다.

```jsx
import { fireEvent, render } from '@testing-library/react'
import userEvent from '@testing-library/user-event'

import { InputComponent } from '.'

describe('InputComponent 테스트', ()=>{
	const setup = () => {
		const screen = render(<InputComponent />);
		const input = screen.getByLabelText('input') as HTMLInputElement;
		const button = screen.getByText(/제출하기/i) as HTMLButtonElement;
		return {
			input,
			button,
			...screen,
		}
	}
	
	it('input의 초깃값은 빈 문자열이다.', ()=>{
		const { input } = setup();
		expect(input.value).toEqual('');
	});
	
	it('input의 최대 길이가 20자로 설정돼 있다.', () => {
		const { input } = setup();
		expect(input).toHaveAttribute('maxlength', '20');
	})
	
	it('영문과 숫자만 입력된다.', ()=>{
		const { input } = setup();
		const inputValue = '안녕하세요123';
		userEvent.type(input, inputValue);
		expect(input.value).toEqual('123');
	});
	
	it('아이디를 입력하지 않으면 버튼이 활성화되지 않는다.', ()=>{
		const { button } = setup();
		expect(button).toBeDisabled();
	})
	
	it('아이디를 입력하면 버튼이 활성화된다.', () => {
		const { button, input } = setup();
		
		const inputValue = 'helloworld';
		userEvent.type(input, inputValue);
		
		expect(input.value).toEqual(inputValue);
		expect(button).toBeEnabled();
	})
	
	it('버튼을 클릭하면 alert가 해당 아이디로 표시된다.', ()=>{
		const alertMock = jest.spyOn(window, 'alert')
				.mockImplementation((_: string) => undifined);
				
		const { button, input } = setup();
		const inputValue = 'helloworld'
		userEvent.type(input, inputValue);
		fireEvent.click(button)
		
		expect(alertMock).toHaveBeenCalledTimes(1);
		expect(alertMock).toHaveBeenCalledWith(inputValue);
	});
}); 
```

- setup 함수 :
    - 내부에서 컴포넌트를 렌더링하고, 또 테스트에 필요한 `button`과 `input`을 반환한다.
    - 이 파일에서 수행하는 모든 테스트는 렌더링과 button, input을 필요로 하므로 이를 하나의 함수로 묶어 두었다.
- userEvent.type : 사용자가 타이핑하는 것을 흉내내는 메소드다.
    - 사용자가 키보드로 타이핑하는 것과 동일한 작동을 만들 수 있다.
    - `@testing-library/react`에서 제공하는 fireEvent와 차이가 있다.
    - 기본적으로 userEvent는 fireEvent의 여러 이벤트를 순차적으로 실행해 좀 더 자세하게 사용자의 작동을 흉내 낸다.
    - 대부분의 이벤트를 테스트할 때는 `fireEvent`로 충분하고 훨씬 더 빠르다. 단, 특별히 사용자의 이벤트를 흉내 내야 할 때만 userEvent를 사용하면 된다.
- jest.spyOn(window, ‘alert’).mockImplementation()
    - jest.spyOn : Jest가 제공하는 spyOn은 어떠한 특정 메소드를 오염시키지 않고 실행이 됐는지, 또 어떤 인수로 실행됐는지 등 실행과 관련된 정보를 얻고 싶을 때 사용한다. 여기서는 (window, ‘alert’)라는 인수와 함께 사용됐는데, 이는 window 객체의 메소드 alert를 구현하지 않고 해당 메소드가 실행됐는지만 관찰하겠다는 뜻이다.
    - mockImplementation : 해당 메소드에 대해 모킹(mocking) 구현을 도와준다. 현재 Jest를 실행하는 Node.js 환경에서는 window.alert가 존재하지 않으므로 해당 메소드를 모의 함수(mock)로 구현해야 하는데, 이것이 바로 `mockImplementation` 의 역할이다.
    - 위 코드에서는 `toHaveBeenCalledTimes`와 `toHaveBeenCalledWith`로 `window.alert`가 몇 번 호출됐는 지, 어떤 인수로 호출됐는지를 알고자 하는 것이다. 하지만 `window.alert` 는 Node.js 환경에 존재하지 않으므로, 이를 대신할 함수(`(_:string)⇒undifined` )를 모킹해서 테스트 한 것.

#### 비동기 이벤트가 발생하는 컴포넌트

비동기 이벤트, 특히 `fetch`가 실행되는 컴포넌트는 어떻게 테스트 할 수 있을까?

버튼을 클릭하면 `/todos/:id`로 fetch 요청을 보내 데이터를 불러오는 컴포넌트가 있다고 하자. 데이터를 불러오는 데 성공하면 응답값 중 하나를 노출하지만, 실패하면 에러 문구를 노출하는 컴퍼넌트다. 

앞서 살펴본 모킹을 통해 fetch를 테스트할 수 있지만, 이 방법으로는 모든 시나리오를 테스트할 수 없다. 

- 서버 응답에서 오류가 발생한 경우, ok, status, json 의 모든 값을 바꿔서 다시 모킹해야 하기 때문이다. 이러한 방식은 테스트를 수행할 때마다 모든 경우를 새롭게 모킹해야 하므로 테스트 코드가 길고 복잡해진다.

이러한 문제를 해결하기 위해 `MSV(Mock Service Worker)`가 등장했다. 

- MSW : Node.js나 브라우저에서 모두 사용할 수 있는 모킹 라이브러리다.
    - 브라우저에서는 서비스 워커를 활용해 실제 네트워크 요청을 가로채는 방식으로 모킹을 구현한다.
    - Node.js 환경에서는 https나 XMLHttpRequest의 요청을 가로채는 방식으로 작동한다.
    
    → 즉, Node.js나 브라우저에서는 **fetch 요청을 하는 것과 동일하게 네트워크 요청을 수행**하고, **이 요청을 중간에 MSW가 감지하여 미리 준비한 모킹 데이터를 제공**하는 방식이다. 
    
    → 이러한 방식은 fetch의 모든 기능을 그대로 사용하면서도 응답에 대해서만 모킹할 수 있으므로 fetch를 모킹하는 것이 훨씬 쉬워진다. 
    

```jsx
import { fireEvent, render, screen } from '@testing-library/react'
import { rest } from 'msw'
import { setupServer } from 'msw/node'

import { FetchComponent } from '.'

const Mock_TODO_RESPONSE = {
	userId : 1, 
	id : 1,
	title : 'delectus aut autem',
	completed : false,
}

const server = setupServer(
	rest.get('/todos/:id', ()=>{
		const todoId = req.params.id;
		
		if (Number(todoId)) {
			return res(ctx.json({ ...MOCK_TODO_RESPONSE, id : Number(todoId) }))
		} else {
			return res(ctx.status(404));
		}
	});
);
// MSW를 활용해 fetch 응답 모킹.
// setupServer는 MSW에서 제공하는 메소드로, 서버를 만드는 역할을 한다.
// 이 함수 내부에서 Express나 Koa와 비슷하게 라우트를 선언할 수 있다.
// 라우트 내부에서 서버 코드를 작성하는 것과 동일하게 코드를 작성하고, 
// 대신 응답하는 데이터만 미리 준비해 둔 모킹 데이터를 반환하면 된다.
// 위 코드에서는 라우트 /todos/:id의 요청만 가로채서 숫자일 때는 준비한 목 데이터를,
// 숫자가 아니라면 404를 반환하도록 구성.

beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());
// 테스트 코드를 시작하기 전에 서버를 기동하고, 실행이 종료되면 서버를 종료시킨다.
// server.resetHandlers()는 setupServer의 기본 설정으로 되돌리는 역할

beforeEach(() => {
	redner(<FetchComponent />);
})

// 이 아래로는 fetch가 발생하는 시나리오를 테스트.
describe('FetchComponent 테스트', () => {
	it('데이터를 불러오기 전에는 기본 문구가 뜬다.', async () => {
		const nowLoading = screen.getByText(/불러온 데이터가 없습니다./);
		expect(nowLoading).toBeInTheDocument();
	});
	
	it('버튼을 클릭하면 데이터를 불러온다.', async () => {
		const button = screen.getByRole('button', { name : /1번/ })
		fireEvent.click(button);
		
		const data = await screen.findByText(MOCK_TODO_RESPONSE.title);
		expect(data).toBeInTheDocument();
	});

	// 앞서 setupServer에서는 정상적인 응답만 모킹했기 때문에, 에러가 발생하는 경우를 
	// 테스팅하기 어렵다. 
	// 서버 응답이 실패하는 경우를 테스트하기 위해 server.use를 사용해 기존 setupServer의
	// 내용을 새롭게 덮어 쓴다. 
	// 여기에서는 /todos/:id 라우팅을 모든 경우에 503이 오도록 작성했다. 	
	// server.use를 활용한 서버 기본 동작을 덮어쓰는 작업은 해당 it 내에서만 유효해야 한다.
	// 다른 테스트 시에는 원래대로 서버 작동이 다시 변경되어야 하므로 afterEach에서
	// resetHandlers를 실행한다.
	it('버튼을 클릭하고 서버 요청에서 에러가 발생하면 에러 문구를 노출한다.', async () =>{
		server.use(
			rest.get('/todos/:id', (req, res, ctx) => {
				return res(ctx.status(503));
			});
		);
		
		const button = screen.getByRole('button', { name : /1번/ });
		fireEvent.click(button);
		
		const error = await screen.findByText(/에러가 발생했습니다./);
		expect(error).toBeInTheDocument();
	})
});
```

- 여기서 중요한 것은 `MSW를 사용한 fetch 응답 모킹`과 `findBy를 활용해 비동기 요청이 끝난 뒤에 제대로 된 렌더링이 일어났는지 기다린 후에 확인하는 것`이다.

## 사용자 정의 훅 테스트하기

훅을 테스트하는 과정도 동일하게 진행할 수 있다. 훅이 들어가있는 컴포넌트를 만들거나, 혹은 훅이 들어있는 컴포넌트에 대해 별도로 훅에 대한 테스트를 만들 수도 있을 것이다.

하지만 이 경우 전자는 테스트 코드 작성 외에 작업이 더 추가된다는 점, 후자는 해당 훅이 모든 테스트 케이스를 커버하지 못하 경우 또 다른 테스트 가능한 컴포넌트를 찾아야 한다는 단점이 있다.

이런 불편함을 해결하기 위해 `react-hooks-testing-library` 라이브러리가 있다.

예제로 사용할 `useEffectDebugger` 훅은 컴포넌트명과 props를 인수로 받아 해당 컴포넌트가 어떤 props의 변경으로 인해 리렌더링됐는지 확인해 주는, 일종의 디버거 역할을 한다. 기능은 다음과 같다.

- 최초 컴포넌트 렌더링 시에는 호출X
- 이전 props를 useRef로 저장해 두고, 새로운 props를 넘겨받을 때마다 이전 props와 비교해 무엇이 렌더링을 발생시켰는지 확인
- 이전 props와 신규 props의 비교는 리액트의 원리와 동일하고 Object.is를 활용해 얕은 비교 수행
- 어디까지나 props가 변경되는 것만 확인할 수 있음. (다른 렌더링 시나리오는 감지 X)
- `process.env.NODE_ENV==='production'` 인 경우에는 로깅x. (웹팩 트리쉐이킹통한 최적화)

```jsx
import { renderHook } from '@testing-library/react'

import useEffectDebugger, { CONSOLE_PREFIX } from './useEffectDebugger'

const consoleSpy = jest.spyOn(console, 'log');
// console.log 호출 여부 확인. 
const componentName = 'TestComponent'
// 테스트 대상 컴포넌트의 이름을 componentName에 저장.
```

```jsx
describe('useEffectDebugger', ()=>{
	// 매번 테스트가 끝난 후에는 process.env.NODE_ENV를 다시 develop으로 변경.
	// ts에서는 NODE_ENV를 읽기 전용으로 간주하기 때문에, 강제 작성 해야한다.
	afterAll(()=>{
		// @ts-ignore
		process.env.NODE_ENV == 'development'
	});
	// ... 아래에서 이어짐
});
```

```jsx
it('props가 없으면 호출하지 않는다.', ()=>{
	renderHook(() => useEffectDebugger(componentName));
	
	expect(consoleSpy).not.toHaveBeenCalled();
});

it('최초에는 호출되지 않는다.', () => {
	const props = { hello: 'world' }
	
	renderHook(() => useEffectDebugger(componentName, props));
	// 훅을 렌더링하기 위해서는 renderHook을 래핑해서 사용해야 한다. 
	// renderHook 함수는 내부에서 TestComponent라고 하는 컴포넌트를 생성하고, 이 컴포넌트
	// 내부에서 전달받은 훅을 실행한다.
	
	expect(consoleSpy).not.toHaveBeenCalled();
});
```

```jsx
it('props가 변경되지 않으면 호출되지 않는다.', () => {
	const props = { hello : 'world' };
	const { rerender } = renderHook(() => useEffectDebugger(componentName, props));
	
	// 이번 테스트에서는 컴포넌트를 다시 렌더링해 훅 내부의
	// console.log가 실행되지 않는지를확인.
	expect(consoleSpy).not.toHaveBeenCalled();
	
	// 만약 renderHook을 한번 더 실행하면 컴포넌트가 새로 만들어지므로, 
	// renderHook이 반환하는 rerender 메소드를 사용.
	// renderHook은 rerender 말고도 unmount도 반환. 이 함수는 컴포넌트를 언마운트시킨다.
	rerender();
	
	expect(consoleSpy).not.toHaveBeenCalled();
})
```

```jsx
it('props가 변경되면 다시 호출한다.', () => {
	const props = { hello : 'world' };
	
	// props 비교를 정확히 하고 있는지 확인하기 위해 훅에 서로 다른 props를 인수로 넘긴다.
	// renderHook은 initialProps를 인자로 넘겨 훅의 초깃값을 지정할 수 있다. 
	const { rerender } = renderHook(
		({ componentName, props }) => useEffectDebugger(componentName, props),
		{
			initialProps : {
				componentName,
				props,
			},
		},
	);
	
	// 이후 rerender할 때 초깃값을 변경해 다시 렌더링한다.
	const newProps = { hello : 'world2 };
	rerender({ componentName, props : newProps });
	
	// 리렌더링이 발생했따면 console.log가 호출됐을 것이다.
	expect(consoleSpy).toHaveBeenCalled();
});
```

```jsx
// process.env.NODE_ENV == "production" 을 설정하면 어떠한 경우에도 consoleSpy가 
// 호출되지 않는 지 확인한다.
it('process.env.NODE_ENV가 production이면 호출되지 않는다.', () => {
	// @ts-ignore
	process.env.NODE_ENV = 'production'
	// 위에서 NODE_ENV에 'production' 강제 주입
	
	const props = { hello : 'world' }
	
	const { rerender } = renderHook(
		() => useEffectDebugger(componentName, props),
		{
			initialProps : {
				componentName,
				props
			},
		},
	);
	
	const newProps = { hello : 'world2' }
	
	rerender({ componentName, props : newProps });
	
	expect(consoleSpy).not.toHaveBeenCalled();
})
```

## 테스트를 작성하기에 앞서 고려해야 할 점

테스트 커버리지를 맹신해선 안된다.

- 소프트웨어의 테스트에 대해 논할 때 **테스트 커버리지**라고 해서 **해당 소프트웨어가 얼마나 테스트됐는지를 나타내는 지표**에 대해 들어본 적이 있을 것이다. 하지만 **테스트 커버리지는 단순히 얼마나 많은 코드가 테스트되고 있는 지를 나타내는 지표일 뿐**, 테스트가 잘되고 있는지를 나타내지 않는다.
- 또한 테스트 커버리지를  100%까지 끌어올릴 수 있는 상황은 생각보다 드물다. TDD 개발 방법론을 차용해서 테스트를 우선시하더라도 서버 코드와는 다르게 프론트엔드 코드는 사용자의 입력이 매우 다양해, 이러한 모든 상황을 커버해 테스트를 작성하기란 거의 불가능하다.

따라서 테스트 코드를 작성하기 전에 생각해 봐야 할 최우선 과제는, `애플리케이션에서 가장 취약하거나 중요한 부분을 파악`하는 것이다. 앱에서 **가장 핵심이 되는 부분부터 먼저 테스트 코드를 하나씩 작성해 나가는 것**이 중요하다. 

**테스트 코드는 개발자가 단순 코드 작성만으로는 쉽게 이룰 수 없는 목표인 `소프트웨어 품질에 대한 확신을 얻기 위해 작성하는 것`이라는 점을 반드시 기억해야 한다.**

## 그 밖에 해볼 만한 여러 가지 테스트

프론트엔드 개발에 있어 테스트를 수행할 수 있는 방법은 굉장히 다양하다. 

- 유닛테스트 (Unit Test) : 각각의 코드나 컴포넌트가 독립적으로 분리된 환경에서 의도된 대로 정확히 작동하는지 검증
- 통합테스트 (Integration Test) : 유닛 테스트를 통과한 여러 컴포넌트가 묶여서 하나의 기능으로 정상적으로 작동하는지 확인
- 엔드 투 엔드 (End to End Test) : 흔히 E2E 테스트라 하며, 실제 사용자처럼 작동하는 로봇을 활용해 애플리케이션의 전체적인 기능을 확인하는 테스트.

리액트 테스팅 라이브러리는 유닛 테스트 내지는 통합 테스트를 도와주는 도구로, E2E 테스트를 수행하려면 Cypress 같은 다른 라이브러리의 힘을 빌려야 한다. 

유닛테스트에서 엔드 투 엔드 테스트로 갈수록 테스트 실패 지점도 많아지고, 테스트 코드도 복잡해지며, 테스트해야 할 경우의 수도 많아지고, 테스트 코드 구축도 어려워진다. 하지만 그만큼 개발자에게 있어 코드에 대한 자신감을 심어줄 수 있는 가능성도 커진다.

## 정리

테스트 방법은 여러가지가 있지만, 목적은 **애플리케이션이 비즈니스 요구사항을 충족하는지 확인**하는 것 딱 하나 뿐이다. 

처음부터 너무 많은 준비가 필요한 E2E는 테스트 코드를 작성하기도 전에 지치거나 혹은 다른 급한 일에 테스트 코드 작성의 우선순위가 밀려날지도 모른다. 그러나 조금씩, 핵심적인 부분부터 테스트 코드를 작성하다보면 SW 품질에 대한 확실을 갖게 될 것이다.