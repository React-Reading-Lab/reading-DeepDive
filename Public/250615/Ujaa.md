# React Compiler RC

## 세 줄 요약

1. React Compiler의 안정적인 출시를 준비 중이다.
2. eslint-plugin-react-compiler를 eslint-plugin-react-hooks로 통합한다.
3. swc 지원을 추가했고, babel 없이 빌드할 수 있도록 oxc와 협업 중이다.

React Compiler는 **자동 메모이제이션**을 통해 리액트 앱을 최적화하는 빌드 도구이며, 현재 거의 안정적이고 배포 환경에서 사용 가능하다. React 17 이상과 호환되며 19버전을 사용중이지 않다면 컴파일러 설정에서 최소 타겟 react 버정르 명시하고 react-comiler-runtime 패키지를 프로젝트에 추가하면 된다.

현재 컴파일러는 optional chaining (`user?.name`)이나 배열 인덱스 접근 (`arr[0]`)도 의존성으로 추적할 수 있다. 이를 통해 불필요한 리렌더링을 줄이고 UI를 더 빠르게 만들 것이다.
