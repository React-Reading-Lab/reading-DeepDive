# 애플리케이션에서 확인하기

- **create-react-app**: performanceObserver API를 사용하는 web-vitals 라이브러리를 활용
- **create-next-app**: NextWebVitalsMetric 함수를 제공한다. 또한, Next.js에 특화된 사용자 지표도 제공한다.
    1. Next.js-hydration: 페이지가 서버 사이드에서 렌더링되어 하이드레이션하는 데 걸린 시간
    2. Next.js-route-change-to-render: 페이지가 경로를 변경한 후 페이지를 렌더링 시작하는데 걸리는 시간
    3. Next.js-render: 경로 변경이 완료된 후 페이지를 렌더링하는 데 걸리는 시간

# 구글 라이트하우스

라이트하우스란? 구글에서 제공하는 웹 페이지 성능 측정 도구로 오픈소스로 운영된다.

PWA, SEO 등 웹페이지를 둘러싼 다양한 요소를 측정하고 점검할 수 있다.

- 브라우저 확장 프로그램, 개발자 도구, CLI로 사용 가능
- 확장 프로그램이 측정에 영향을 줄 수 있어서 시크릿 모드로 실행하는 게 권장됨

## 탐색 모드

페이지에 접속했을 때부터 페이지 로딩이 완료될 때까지의 성능을 측정한 모드

### 성능

측정 지표: FCP, LCP, CLS, TTI, Speed Index, TBT

- TTI: 페이지에서 사용자가 완전히 상호작용할 수 있을 때까지 걸리는 시간
    - 최초 콘텐츠풀 페인트로 측정되는 페이지 내 콘텐츠가 표시되는 시점
    - 보여지는 페이지 요소의 대부분에 이벤트 핸들러가 부착되는 시점
    - 페이지가 유저의 상호작용에 50ms 내로 응답하는 시점
    - 3.8 이내면 좋음, 7.3 이내면 보통, 그 이후는 개선이 필요
- Speed Index: 페이지가 로드되는 동안 콘텐츠가 얼마나 빨리 시각적으로 표시되는지 계산
    - 3.4 이내면 좋음, 5.8 이내면 보통, 그 이후는 개선 필요
- Total Blocking Time: 메인 스레드에서 50ms 이상 실행되는 작업을 모아서 메인 스레드가 차단될 가능성이 있는 시간을 계산

### 접근성

- 장애인 및 고령자 등 신체적으로 불편한 사람들이 일반적인 사용자와 동등하게 웹페이지를 볼 수 있도록 하는 것

### 권장사항

보안, 표준 모드, 최신 라이브러리, 소스 맵 등 웹사이트를 개발할 때 고려해야할 요소를 얼마나 지키고 있는지 확인한다.

### 검색엔진 최적화

구글과 같은 검색엔진이 쉽게 웹페이지 정보를 가져가서 공개할 수 있도록 최적화되어 있는지 확인.

## 기간 모드

실제 웹페이지를 탐색하는 동안 지표를 측정하는 것이다.

### 흔적

웹 성능을 추적한 결과를 보여준다.

### 트리맵

페이지를 불러올 때 함께 로딩한 모든 리소스를 함께 모아볼 수 있는 곳이다.

## 스냅샷

현재 페이지 상태를 기준으로 분석.

# WebPageTest

webpagetest는 웹사이트 성능 분석도구로 가장 널리 알려진 도구다.

## 종류

- Site Performance: 성능
- Core Web Vitals: 핵심 지표
- LightHouse: 구글 라이트하우스 도구
- Visual Comparison: 2개 이상의 사이트를 동시에 실행해 시간의 흐름에 따른 로딩 과정을 비교하는 도구
- Traceroute: 네트워크 경로를 확인하는 도구

## Performance Summary

- 얼마나 빠른가?
    - 최초 바이트까지의 시간이 짧은지, 콘텐츠 렌더링이 즉각적으로 일어나는지, 최대 콘텐츠풀 페인트 시간이 합리적인지 확인한다.
- 사용성이 좋은가?
    - 콘텐츠 누적 이동을 최소화하고 있는지, 상호작용을 빠르게 하고 있는지, 접근성 이슈가 있는지 클라이언트 사이드에서 과도하게 HTML을 많이 렌더링하는지 등을 점검한다.
- 보안적으로 괜찮은가?
    - 렌더링을 블로킹하는 제 3자 라이브러리가 있는지 실질적인 위협이 되는 보안 요소가 있는지를 나타낸다.
  
![image](https://github.com/user-attachments/assets/eb1ac541-aaf1-4230-830a-cad55a419119)

## Opportunities & Experiments

### **Is it Quick?**

- 렌더링을 블로킹하는 CSS가 있는지 확인한다.
    
    ![image](https://github.com/user-attachments/assets/93ef7957-88cd-4394-b09e-519708277039)

- 문자의 노출을 지연시키는 커스텀 폰트가 있는지 확인한다. font-display=block와 같은 형식으로 폰트를 불러온다면 해당 폰트가 로딩될 때까지 문자가 보이지 않을 것이다.

    ![image](https://github.com/user-attachments/assets/efa4a4b4-8529-4f83-a4a2-1815250d50cd)
    
- 최초로 다운로드받은 HTML과 최종 결과물 HTML 사이에 크기 차이가 작아야 한다.
    
    ![image](https://github.com/user-attachments/assets/fb7d9688-e6f9-4bfa-9ffc-6b868bd26409)


### Is it Usable

- 이미지 비율 부채로 인한 레이아웃 이동 가능성. 비율이 없으면 브라우저는 이미지가 로딩되기 전까지 해당 이미지의 크기를 알 수 없어 결과적으로 레이아웃 이동이 발생한다.

    ![image](https://github.com/user-attachments/assets/91b8266f-49a7-4ca0-9915-09cb1094d530)

- 접근성 이슈.
    
    ![image](https://github.com/user-attachments/assets/eebb1e22-5359-40f9-bcea-838290b8c451)

- 스크린 리더기가 해당 콘텐츠를 읽는데 걸림돌이 될 것이다.
    
    ![image](https://github.com/user-attachments/assets/fe23c71c-0b48-4310-bec8-4cb6b212e65a)

### **Is it Resilient?**

- 자바스크립트 의존이 높으면 자바스크립트 에러와 제 3자 네트워크 요청 실패 등으로 페이지 렌더링 실패 가능성이 높아진다.
    
    ![image](https://github.com/user-attachments/assets/0146ca89-c14e-4377-959d-1141a9bf20af)

## Observed Metrics

![image](https://github.com/user-attachments/assets/75f71530-cd50-461d-843f-b30e1ae5c9db)

최초 바이트까지의 시간, 렌더링 시작에 소요되는 시간, 최초 콘텐츠풀 펭니트 등 측정할 수 있는 다양한 시간 지표에 대해 나타낸다. 

## Filmstrip

웹사이트를 필름을 보는 것처럼 시간의 흐름에 따라 어떻게 웹사이트가 그려졌는지, 어떤 리소스가 불러와졌는지 확인할 수 있는 메뉴다. 렌더링을 가로막는 리소스나 예상보다 일찍 실행되는 스크립트 등을 확인할 수 있다.

![image](https://github.com/user-attachments/assets/e2684877-4dbc-4709-b958-efc5ff1ba9ab)

## Details

더 자세한 정보

![image](https://github.com/user-attachments/assets/ec5d2ea6-3a2f-4a14-a8b4-5355ec0eb9ca)

## Web Vitals

웹 핵심 지표에 대한 설명

## Optimizations

리소스가 얼마나 최적화되어 있는지 나타내는 지표.

![image](https://github.com/user-attachments/assets/e45f9ba1-5ffa-4556-9eaa-75a423c475c0)

- Keep Alive: 서버와의 연결을 유지하고 있는지
- GZip으로 리소스를 압축하고 있는지
- 이미지를 적절히 압축했는지
- Progressive JPEG로 JPEG를 렌더링하고 있는지
- 리소스 캐시 정책이 올바르게 수립돼 있는지
- 리소스가 CDN을 거치고 있는지

## Content

제공하는 콘텐츠, 에셋별로 묶어서 통계를 보여준다.

![image](https://github.com/user-attachments/assets/a164ab7f-9be9-46d5-b314-74b82ffdf000)

## Domains

에셋들이 어느 도메인에서 왔는지 도메인별로 묶어서 확인할 수 있다.

## Console log

사용자가 웹페이지를 접속했을 때 console.log로 무엇이 기록됐는지 확인할 수 있다.

## Detected Technologies

사용한 기술을 알 수 있다.

![image](https://github.com/user-attachments/assets/715e4a5d-feeb-4c57-8a32-e688d65603e8)

## Main-thread Processing

메인 스레드가 어떤 작업을 처리했는지 확인할 수 있다.

![image](https://github.com/user-attachments/assets/7b6caea4-95eb-4b9f-9cb2-600153a5506a)

## Lighthouse Report

라이트하우스 리포트를 확인할 수 있다.

## 추가 ) Image Analysis

![image](https://github.com/user-attachments/assets/522874c6-6bf1-4e17-add9-d1d8dcf3d48a)

# 크롬 개발자 도구

- 시크릿에서 열기를 권유

## 성능 통계

- performance insight는 웹사이트의 성능을 자세히 확인할 수 있다.
- 웹사이트 로딩 시작부터 끝까지를 확인하거나 원하는 액션을 수행할 때 성능을 측정할 수 있다.
- Throttling으로 네트워크와 CPU 속도를 지연시켜서 테스트할 수도 있다.
- 뷰포트가 잘린만큼 측정하기 때문에 원하는 사이즈로 설정하고 해야한다.
- performance summary에서 제공하는 내용과 비슷하다.

## 성능

- 성능 통계와 마찬가지로 웹사이트 측정 후 성능을 분석한다.
- 요약 탭에서는 특정 기간의 CPU, 네트워크 요청, 스크린샷, 메모리 점유율을 요약해서 볼 수 있다.
- 네트워크에서는 리소스별로 각기 다른 색상으로 확인할 수 있다. 참고로 위에 있는 요청이 우선순위가 높은 요청이다.

    ![image](https://github.com/user-attachments/assets/886e9876-8afc-44cf-8a46-466dfde696ce)

- 하단 그래프로는 힙, 노드, 리스너의 변화를 살펴볼 수 있다. 그리고 해당 영역에서 발생한 이벤트가 무엇인지 추적할 수 있다.
