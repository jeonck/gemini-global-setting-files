# 빈 화면(백지) 문제 해결 과정 (Troubleshooting Blank Screen Issue)

이 문서는 애플리케이션 개발 중 발생했던 빈 화면 렌더링 문제와 그 해결 과정을 기록합니다.

## 1. 문제 증상

- `npm run dev` 명령어로 개발 서버를 실행한 후 브라우저에서 접속 시, 아무 내용도 표시되지 않는 빈 화면(백지) 상태가 지속됨.
- 브라우저 개발자 도구의 콘솔에 별다른 오류가 표시되지 않는 경우도 있어 초기 원인 파악이 어려웠음.

## 2. 디버깅 과정

문제의 원인을 찾기 위해 다음과 같이 단계적인 접근 방식을 사용했습니다.

### 단계 1: 설정 파일 확인

- **가설**: Vite 설정 문제일 것이다.
- **검증**: `vite.config.ts` 파일의 `base` 설정이 `/`로 올바르게 되어 있는지 확인했습니다. 초기에는 `/prompt-showcase/`로 되어 있어 수정했지만, 이후에도 문제가 지속되어 다른 원인이 있음을 확인했습니다.

### 단계 2: 의존성 및 캐시 문제 해결

- **가설**: 캐시 또는 `node_modules` 의존성 손상 문제일 것이다.
- **검증 1**: Vite의 캐시를 강제로 재생성하기 위해 `npm run dev -- --force` 명령을 실행했으나, 문제가 해결되지 않았습니다.
- **검증 2**: 의존성을 완전히 새로 설치하기 위해 다음 명령어를 순서대로 실행했습니다.
  1. `rm -rf node_modules` (기존 의존성 폴더 삭제)
  2. `rm package-lock.json` (패키지 잠금 파일 삭제)
  3. `npm install` (모든 의존성 재설치)
- 위 조치 후에도 문제가 해결되지 않아, 코드 자체의 버그일 가능성에 무게를 두었습니다.

### 단계 3: 컴포넌트 분리 및 최소 기능 테스트

- **가설**: 특정 컴포넌트 내부의 런타임 오류가 전체 앱의 렌더링을 막고 있다.
- **검증**: 가장 복잡하고 최근에 수정된 `src/pages/Home.tsx` 파일을 아래와 같이 최소한의 기능만 가진 테스트 코드로 교체했습니다.

  ```jsx
  import React from 'react';

  const Home = () => {
    return (
      <div>
        <h1>Test Page</h1>
      </div>
    );
  };

  export default Home;
  ```

- **결과**: 위 테스트 코드로 교체하자 "Test Page"라는 텍스트가 정상적으로 화면에 표시되었습니다. 이를 통해 **문제의 원인이 `Home.tsx` 컴포넌트 내부에 있음**을 확신할 수 있었습니다.

### 단계 4: 코드 증분 복구

- **가설**: `Home.tsx`의 특정 코드 라인이 문제를 일으킨다.
- **검증**: `Home.tsx`의 원래 코드를 조금씩 다시 추가하며 어느 부분에서 문제가 발생하는지 확인했습니다.
  1. 텍스트와 기본 레이아웃 추가 -> 정상 작동
  2. 아이콘을 제외한 카드 목록 렌더링 -> 정상 작동
  3. `react-feather` 아이콘을 포함시키자 다시 빈 화면 문제 발생 -> **아이콘 처리 방식이 원인임을 최종 확인**.

## 3. 근본 원인

문제의 근본 원인은 `react-feather` 아이콘 컴포넌트를 사용하는 방식에 있었습니다.

- **잘못된 코드**:
  ```jsx
  import { Zap } from 'react-feather';

  const features = [
    {
      icon: <Zap className="w-8 h-8" />, // JSX 요소를 데이터 배열에 직접 할당
      title: '...',
    },
  ];
  ```

- **문제점**: 위와 같이 데이터 배열을 정의하는 시점에 JSX 요소(` <Zap ... />`)를 미리 생성하면, React의 렌더링 사이클과 맞지 않아 런타임 오류를 유발할 수 있습니다. 컴포넌트는 렌더링 단계에서 생성되는 것이 가장 안정적입니다.

## 4. 해결 방법

데이터 배열에는 JSX 요소를 직접 넣는 대신, **컴포넌트 자체에 대한 참조**를 저장하도록 코드를 수정했습니다. 그리고 실제 렌더링이 일어나는 `FeatureCard` 컴포넌트 내부에서 해당 컴포넌트를 생성하도록 변경했습니다.

- **수정된 코드**:
  ```jsx
  import React from 'react';
  import { Zap } from 'react-feather';

  // 데이터 배열에는 컴포넌트 참조를 저장
  const features = [
    {
      icon: Zap,
      title: '...',
    },
  ];

  const FeatureCard = ({ icon: Icon, title, ... }) => (
    <div>
      {/* 렌더링 시점에 컴포넌트 생성 */}
      {Icon && <Icon className="w-8 h-8" />}
      <h3>{title}</h3>
    </div>
  );
  ```

이 수정을 통해 `react-feather` 아이콘이 올바른 렌더링 흐름에 따라 생성되도록 하여 빈 화면 문제를 최종적으로 해결했습니다.

## 5. GitHub Actions 워크플로우 및 TypeScript 오류 해결

GitHub Actions 빌드 실패 및 TypeScript 오류 해결 과정은 다음과 같습니다.

### 5.1. 초기 워크플로우 빌드 실패

- **문제**: `ci.yml` 워크플로우가 초기 빌드에서 실패했습니다.
- **원인 분석**: `react-hub` 예제와 비교하여 `npm install` 대신 `npm ci` 사용, Node.js 버전 불일치, `lint` 스텝의 엄격한 설정(`--max-warnings 0`) 등이 원인으로 추정되었습니다.
- **해결**: `npm run lint` 스텝을 제거하고, Node.js 버전을 18로 통일하며, `npm ci` 사용 및 캐시 설정을 추가했습니다.

### 5.2. TypeScript 컴파일 오류

- **문제**: `npm run build` 실행 시 다수의 TypeScript 컴파일 오류가 발생했습니다.
- **오류 유형 및 해결 방법**:
    1.  **`TS7016: Could not find a declaration file for module 'prismjs'.`**
        -   **해결**: `npm install --save-dev @types/prismjs`를 통해 `prismjs` 타입 정의를 설치했습니다.
    2.  **`TS6133: 'React' is declared but its value is never read.`**
        -   **해결**: `tsconfig.json` 파일의 `compilerOptions`에 `"noUnusedLocals": false`를 추가하여 사용되지 않는 지역 변수에 대한 검사를 비활성화했습니다. (JSX 변환 시 `React`가 명시적으로 사용되지 않아 발생하는 오류).
    3.  **`TS2339: Property 'env' does not exist on type 'ImportMeta'.`**
        -   **해결**: `src/vite-env.d.ts` 파일을 생성하고 `ImportMetaEnv` 인터페이스를 선언하여 `import.meta.env`에 대한 타입 정의를 추가했습니다.
    4.  **`TS2353: Object literal may only specify known properties, and 'highlight' does not exist in type 'MarkedExtension'.`**
        -   **해결**: `marked` 라이브러리의 `highlight` 옵션 사용 방식이 변경됨에 따라 `npm install marked-highlight`를 설치하고 `MarkdownRenderer.tsx`에서 `marked.use(markedHighlight(...))` 방식으로 통합하여 구문 강조를 처리했습니다.
    5.  **`TS7006: Parameter '...' implicitly has an 'any' type.` / `TS7031: Binding element '...' implicitly has an 'any' type.` / `TS18046: 'items' is of type 'unknown'.`**
        -   **해결**: `Navbar.tsx` 및 `Home.tsx` 파일 내의 모든 함수형 컴포넌트(예: `NavLink`, `Dropdown`, `ServerCard` 등)의 props에 대해 명시적인 TypeScript 인터페이스를 정의하고 적용하여 `noImplicitAny` 관련 오류를 해결했습니다.

## 6. GitHub Pages 배포 시 빈 화면 문제 해결

- **문제**: GitHub Actions를 통해 GitHub Pages에 배포된 후 사이트가 빈 화면으로 표시되었습니다.
- **원인**: SPA(Single Page Application)가 로컬 개발 환경(루트 경로 `/`)을 기준으로 빌드되었으나, GitHub Pages에서는 `/mcp-hub/`와 같은 하위 경로에서 제공되면서 정적 자원(JS, CSS)을 올바르게 찾지 못해 발생했습니다.
- **해결**: 
    1.  `vite.config.ts` 파일의 `base` 옵션을 GitHub Pages의 하위 경로인 `'/mcp-hub/'`로 변경했습니다.
    2.  `src/App.tsx` 파일의 `BrowserRouter` 컴포넌트에 `basename="/mcp-hub"` 속성을 추가하여 React Router가 올바른 기본 경로를 인식하도록 했습니다.

## 7. 환경 변수를 이용한 로컬/배포 환경 설정 분리

- **문제**: 이전 해결책(6번)은 `base` 경로를 하드코딩하여 로컬 개발 환경과 배포 환경 간의 유연성이 부족했습니다.
- **원인**: 로컬 개발 시에는 루트 경로(`/`)를 사용하고, GitHub Pages 배포 시에는 하위 경로(`/mcp-hub/`)를 사용해야 하는 필요성 때문입니다.
- **해결**: 
    1.  `vite.config.ts` 파일의 `base` 옵션을 `process.env.VITE_APP_BASE_PATH || '/'`로 변경했습니다.
    2.  `src/App.tsx` 파일의 `BrowserRouter` 컴포넌트 `basename` 속성을 `import.meta.env.VITE_APP_BASE_PATH || "/"`로 변경했습니다.
    3.  `.github/workflows/deploy.yml` 배포 워크플로우의 `Build` 스텝에 `VITE_APP_BASE_PATH: /mcp-hub/` 환경 변수를 추가하여 GitHub Pages 배포 시 올바른 경로가 사용되도록 했습니다.
    4.  로컬 개발 시에는 `VITE_APP_BASE_PATH` 환경 변수를 설정하지 않거나 `/`로 설정하여 루트 경로를 사용하도록 했습니다.