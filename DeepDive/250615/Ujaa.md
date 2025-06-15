# 9.2 깃허브 100% 활용하기

CI: 코드 중앙 저장소에서 코드를 지속적으로 빌드하고 테스트해 코드의 정합성을 확인하는 과정

→ 코드에 변화가 생길 때마다 테스트, 빌드, 정적 분석, 보안 취약점 분석을 해야하는 것

가장 데표적인 솔루션은 **Jenkins**와 **github actions**

- Jenkins: CI에 필요한 다양한 기능을 제공하는 무료 솔루션이지만 설치를 해야하는 단점이 있음
- Github Actions: 깃허브에서 일어날 수 있는 다양한 이벤트를 기반으로 사용자가 원하는 actions를 취할 수 있게 도와주는 서비스

### 깃허브 액선 기본 개념

```yaml
# action: 러너에서 실행되는 하나의 작업 단위

name: chapter7 build # 액션의 이름
run-name: ${{ github.actor }} has been added new commit. # 액션이 실행될 때 나타낼 타이틀명

on: # 언제 이 액션을 실행할지 명시
	push: # 이벤트: 깃허브 액션의 실행을 일으키는 이벤트
		branch-ignore:
			- 'main'

jobs: # 하나의 러너에서 실행되는 여러 작업(병렬로 실행된다)
	build: # 임의로 지정한 job 이름
		# runner: 깃허브 액션이 실행될 서버(명시하지 않으면 공용 깃허브 액션 서버를 이용)
		runs-on: ununtu-latest # 어떤 환경에서 작업이 실행되는지 명시함. 러너명을 지정하면 커스텀 러너 사용 가능
		steps: # job 내부에서 일어나는 여러 작업
			- uses: actions/checkout@v3
			- uses: actions/setup-node@v3
			with:
				node-version: 16
		- name: 'install dependencies' # 스텝 이름
			working-directory: ./chapter7/my-app
			run: npm ci
		- name: 'build'
			working-directory: ./chapter7/my-app
			run: npm run build
```

### 깃허브에서 제공하는 기본 액션

| 이름 | 기능 |
| --- | --- |
| actions/checkout | 깃허브 저장소를 체크아웃하는 액션(ref를 사용해 특정 브렌치나 커밋을 체크아웃할 수 있음) |
| actions/setup-node | Node.js를 설치하는 액션 |
| actions/github-script | Github API가 제공하는 기능을 사용할 수 있도록 도와주는 액션 |
| actions/stale | 오래된 이슈나 PR을 자동으로 닫거나 더 이상 커뮤니케이션하지 못하도록 닫음 |
| actions/dependency-review-action | 의존성 그래프에 대한 변경이 발생했을 때 실행되는 액션 |
| github/codeql-action | 깃허브의 코드 분석 솔루션인 code-ql을 활용해 저장소 내 코드의 취약점을 분석한다. |

### 알아두면 좋은 다른 액션

| 이름 | 기능 |
| --- | --- |
| calibreapp/image-actions | PR로 올라온 이미지를 sharp 패키지를 이용해 무손실로 압축 후 다시 커밋해준다.(이미지를 가져다가 다시 커밋을 해야하기 때문에 secrets.GITHUB_TOKEN을 추가해야 한다. |
| lirantal/is-website-vulnerable | 특정 웹사이트에 라이브러리 취약점이 존재하는지 확인하는 액션이다. Snyk 솔루션을 기반으로 작동한다. |
| lighthouse CI | 구글에서 제공하는 액션으로 웹 성능 지표인 라이트하우스를 기반으로 CI를 실행할 수 있는 도구 |

## 깃허브 Dependabot으로 보안 취약점 해결하기

> **semantic versioning**
> 
> 
> major.minor.patch형태로 버전을 표기하는 방식.
> 
> ^: major 범위 이하에서 업데이트
> 
> ~: patch 버전 내에서 업데이트
> 

### 의존성

- dependencies: json에서 npm install하면 설치되는 의존성이다.
- devDependencies: 프로젝트를 개발할 때는 필요하지만 실행하는데 필요하지 않은 패키지를 여기 의존성에 추가한다.
- peerDependencies: 라이브러리와 패키지에서 보통 많이 사용되는 개념. 호환성 때문에 필요한 경우 명시할 때 사용한다.
    
    ```json
    // 만약 재사용 가능한 훅을 제공하는 패키지를 만든다면 react 16.8버전 이상 사용해야 함
    {
    	"peerDependencies": {
    		"react": ">=16.8
    	}
    }
    ```
    

Dependabot을 사용하면 See Dependabot alert를 통해 어떤 의존성 문제가 있는지 확인할 수 있고, 취약점 단계별로 Critical, High, Moderate, Low로 분류한다.

취약점을 해결하는 방법은 1. Dependabot이 열어주는 PR로 해결하기, 2. overrides 기능을 사용해서 패키지 내부 버전을 강제로 올리는 것이다.

# 9.3 리액트 애플리케이션 배포하기

코드를 사용자에게 제공하려면 실제 인터넷 망에 배포해야 한다. 규모가 작아 배포하기 어려울 경우 AWS, GCP, Azure 등 클라우드 서비스를 활용할 수도 있고, SaaS 서비스를 사용할 수도 있다.

## Netlify

웹 서비스를 배포할 수 있도록 도와주는 클라우드 컴퓨팅 서비스다.

### 제공 기능

- 배포와 관련된 알림 설정 가능
- 외부에서 구매한 도메인 연결 가능
- Sentry, Aloglia, 레디스 등 다양한 통합 서비스를 이용할 수 있음
- 서버리스 함수 제공
- 인증 처리를 돕는 API 제공
- 사요자 초대

### 가격

- 월 대역폭 최대 100GB
- 빌드 시간 최대 300분
- 여러 사이트 운영 시 한 번에 한 곳만 빌드 가능

## Vercel

Next.js를 비롯한, Turborepo, SWC를 만든 회사에서 만든 클라우드 플랫폼 서비스다.

### 제공 기능

- Netlify에 비해 알림 기능이 한정적이지만 기본적으로 다양한 알림을 제공
- 외부에서 구매한 도메인 연결 가능
- 서버리스 함수 제공, Next.js에서 제공하는 /pages/api 내용도 함수로 구분되어 접근 가능
- 다양한 템플릿 제공

### 가격

- 월 대역폭 최대 100GB
- 이미지 최적화 1000개 제한
- 서버리스 함수 100GB 제한, 함수 실행 시간 10초 이내로 제한
- 동시에 한 가지만 빌드
- 하루에 배포 100개 제한

## DigitalOcean

미국의 클라우드 컴퓨팅, 호스트 플랫폼 업체다. 다른 업체와 다르게 다양한 리소스에 대한 문서화가 상세하게 되어있다. Vercel과 Netlify가 정적 웹사이트 배포에 초점을 맞추고 있다면 DigitalOcean은 좀 더 다양한 클라우드 컴퓨팅을 제공한다.

### 제공 기능

- 알림 기능이 있으나 github으로 알림을 보내는 방법은 제공하지 않음
- 실제 서비스가 실행되고 있는 컨테이너에 직접 접근 가능
- 앱에 추가로 설치할 수 있는 다양한 앱을 마켓 형태로 제공
- 도메인 연결

# 9.4 리액트 애플리케이션 도커라이즈하기

도커는 서비스 운영에 필요한 애플리케이션을 격리해 컨테이너로 만드는데 이용하는 소프트웨어다. 앞선 특정 배포 서비스에 종속적이지도 않고 비용에도 자유로울 수 있다.

## 도커란?

개발자가 모던 애플리케이션을 구축, 공유, 실행하는 것을 도와줄 수 있도록 설계된 플랫폼이다. 도커는 지루한 설정을 대신해 주므로 코드를 작성하는 일에만 집중할 수 있다.

- 이미지: 도커에서 컨테이너를 만드는데 사용되는 템플릿. Dokerfile로 만들 수 있으면 파일을 빌드하면 이미지가 된다.
- 컨테이너: 도커의 이미지를 실행한 상태
- DockerFile: 어떤 이미지 파일을 만들지 정의하는 파일
- 태그: 이미지를 식별할 수 있는 레이블 값(ex. ubuntu:latest, latest 태그)
- 리포지토리: 이미지를 모아두는 저장소
- 레지스트리: 리포지토리에 접근할 수 있게 해주는 서비스(ex. Docker Hub)

### 관련 명령어

```bash
docker build -t foo:bar ./ # ./에 있는 DockerFile을 기준으로 foo:bar라는 이미지 생성
docker push # 이미지나 리포지토리를 도커 레지스트리에 업로드
docker tag # 이미지에 태그 생성
docker inspect # 이미지나 컨테이너 세부 정보 출력
docker run # 이미지를 기반으로 새로운 컨테이너 생성
docker ps # 현재 가동 중인 컨테이너 목록 확인
docker rm # 컨테이너 삭제(실행 중이면 stop으로 멈추고 삭제)
```

### 도커 이미지 만들기

```bash
# 도커 공식 홈페이지에서 설치 후 버전 확인
docker --version
```

```powershell
# 이미지가 어떤 베이스 이미지(또다른 이미지)에서 실행될지 정한다.(이미지 베이스는 도커허브에서 가져온다.)
FROM node:18.12.0-alpine3.16 as build 

WORKDIR /app # 작업 수행 기본 디렉터리

COPY package.json ./package.json # 기본 작업 디렉토리에 파일 복사
COPY package-lock.json ./package-lock.json

RUN npm ci # RUN으로 명령어 실행

COPY . ./ 

RUN npm run build # 빌드에 필요한 리소스 복사 후 빌드 명령어 실행
```

```powershell
docker build . -t cra:test
```

```powershell
FROM nginx:1.23.2-alpine as start

COPY ./nginx/nginx.conf /etc/nginx/nginx.conf
# 앞서 as build로 선언한 단계를 복사해 오는 것. 거기에서 /app/build만 가져와 /usr/share/nginx/html에 복사하는 것
COPY --from=build /app/build /usr/share/nginx/html

EXPOSE 3000

ENTRYPOINT ["nignx", "-g", "daemon off;"] # 컨테이너 실행 후 실행할 명령어
```

> deps와 build 단계 모두 한 파일에 만드는건가?
> 

> 저장소의 역할이 정확히 뭐지?
> 

## 도커로 만든 이미지 배포하기

1. 도커 허브 로그인
2. Create Repository
3. 이미지 푸쉬(Push to Hub)
    
    ```powershell
    docker tag cra:test yceffort/react:cra-test # yceffort 계정의 react 저장소에 이미지 생성
    ```
    

### 도커 이미지 실행하기

1. GCP 프로젝트 생성
2. gcloud cli 설치
3. gcloud auth login
4. Artifact Registry 찾아서 접속 후 빈 저장소 생성
5. 이미지 푸쉬
    
    ```powershell
    gcloud auth confingure-docker northamerica-northeast1-docker.pkg.dev
    docker tag cra:test corthamerica-northeast1-docker.pkg.dev/yceffort/my-react/react/cra:test
    ```
    
6. Cloud Run에서 이미지 실행
