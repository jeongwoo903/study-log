# [React] CRA 템플릿 없이 React 시작하기


yarn init -y
(-y 옵션은 모두 yes를 하겠다는 뜻이다.)
package.json 생성

yarn add styled-components

react 필수 라이브러리 설치
yarn add react react-dom

typescript, react @types 설치
yarn add -D typescript @types/react @types/react-dom
(-D 옵션은 개발, 빌드, 테스트 할 때만 사용한다는 뜻이다.)

tsconfig.json 생성
yarn tsc --init


## vite설치

yarn add -D vite @vitejs/plugin-react

vite.config.ts
```typescript
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  server: {
    open: true, // 서버 시작 시 브라우저 자동 열림
  },
  build: {
    outDir: 'dist', // 빌드 파일이 생성될 디렉토리
  },
  resolve: {
    alias: {
      '@': '/src', // src 폴더에 대한 경로 별칭 설정
    },
  },
});

```

index.html 생성.

src/main.tsx
```typescript jsx
import React from 'react';
import ReactDOM from 'react-dom/client';
import App from './App';

ReactDOM.createRoot(document.getElementById('root') as HTMLElement).render(
  <React>
    <App />
  </React>
);

```

src/App.tsx 
package.json에 스크립트 추가
```json
{
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "serve": "vite preview"
  }
}
```



```typescript jsx
import React from 'react';

function App() {
  return (
    <div>
      <h1>Hello, Vite + React + TypeScript!</h1>
    </div>
  );
}

export default App;
```

```json
{
  "files": [],
  "compilerOptions": {
    "target": "esnext",
    "lib": ["dom", "esnext"],
    "jsx": "preserve",
    "module": "esnext",
    "moduleResolution": "node",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"]
    }
  },
  "include": ["src"],
  "references": [
    {
      "path": "./tsconfig.app.json"
    },
    {
      "path": "./tsconfig.node.json"
    }
  ]
}
```

tsconfig.app.json
```json
{
  "compilerOptions": {
    "composite": true,
    "tsBuildInfoFile": "./node_modules/.tmp/tsconfig.app.tsbuildinfo",
    "target": "ES2020",
    "useDefineForClassFields": true,
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "module": "ESNext",
    "skipLibCheck": true,

    /* Bundler mode */
    "moduleResolution": "bundler",
    "allowImportingTsExtensions": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "moduleDetection": "force",
    "noEmit": true,
    "jsx": "react-jsx",

    /* Linting */
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true
  },
  "include": ["src"]
}

```

tsconfig.node.json
```json
{
  "compilerOptions": {
    "composite": true,
    "tsBuildInfoFile": "./node_modules/.tmp/tsconfig.node.tsbuildinfo",
    "skipLibCheck": true,
    "module": "ESNext",
    "moduleResolution": "bundler",
    "allowSyntheticDefaultImports": true,
    "strict": true,
    "noEmit": true
  },
  "include": ["vite.config.ts"]
}

```


