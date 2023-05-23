# yarn-berry-monorepo-template

### yarn berry 셋팅

```bash
yarn set version berry
yarn set version stable
```

### yarn workspace 패키지 만들기

```bash
yarn init -w
```

### package.json에 workspace 추가

root에 apps 폴더 생성 후에 아래와 같이 root에 있는 package.json → worksapces에 경로를 추가 설정해줍니다. 

```bash
{
  "name": "yarn-berry-monorepo-template",
  "packageManager": "yarn@3.5.1",
  "private": true,
  "workspaces": [
    "packages/*",
    "apps/*"
  ]
}
```

### next 프로젝트 설치하기

apps 폴더 밑에 web이라는 이름으로 next 프로젝트를 생성해준다. 

```bash
yarn create next-app web
```

해당 프로젝트를 설치하고 package.json에 아래와 같이 name을 변경시켜준다. 

```bash
{
  "name": "@hp/web",
  "version": "0.1.0",
  "private": true,
  ...
}
```

기존에는 web으로 되어있는데 모노레포에서는 보통 `@회사이름/프로젝트이름` 형식으로 컨벤션처럼 위와 같이 사용합니다. 

설치한 패키지들을 .pnp.cjs에 적용시키기 위해서 root로 돌아가서 yarn 명령어를 실행합니다. 

```bash
yarn
```

적용하고나면 .pnp.cjs에 위에 설치한 프로젝트와 관련된 dependency의 변경점이 적용되게 된다. 

그리고 root에서 아래의 명령어로 프로젝트를 실행합니다. 

```bash
yarn workspace @hp/web run dev
```

### 타입스크립트 적용하기

위에서 만든 web 프로젝트에 들어가보면 아래와 같이 에러가 나는 것을 확인할 수 있는데 

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/69e95d35-40df-493f-b3d4-ac87f0ca0ff3/Untitled.png)

이는 yarn 공식문서에서 확인해보면 IDE에서 Plug n Play 방식을 사용하고 Typescript를 설치하기 위해서는 sdk를 설치해야 한다고 알려주고 있다. 따라서 우리는 아래와 같은 방식을 추가적으로 진행해야 합니다.

```bash
yarn add -D typescript
yarn dlx @yarnpkg/sdks vscode
```

root에서 타입스크립트를 설치하고 sdk를 설치해줘야 합니다.

그리고 나서 다시 접속하게 되면 에러가 사라진 것을 확인할 수 있습니다. 

[Editor SDKs](https://yarnpkg.com/getting-started/editor-sdks)

### 유틸 함수 공유하기

packages → lib 폴더를 만들고 lib폴더로 이동 후 → 아래의 스크립트를 실행합니다. 

```bash
yarn init

yarn add -D typescript
```

해당 스크립트를 실행하고나면 package.json에는 아래와 같이 적용됩니다. 

```bash
{
  "name": "lib",
  "packageManager": "yarn@3.5.1",
  "devDependencies": {
    "typescript": "^5.0.4"
  }
}
```

위에서 했던 방법과 동일하게 name을 모노레포의 형식에 맞게 아래와 같이 변경하고 추가적으로 main이라는 key에 default 파일을 설정해줍니다. 

```bash
{
  "name": "@hp/lib",
  "packageManager": "yarn@3.5.1",
  "main": "./src/index.ts",
  "devDependencies": {
    "typescript": "^5.0.4"
  }
}
```

타입스크립트를 적용했으니 tsconfig.json 파일을 생성 후에 아래와 같이 기본 설정을 진행합니다. 

```bash
{
  "$schema": "https://json.schemastore.org/tsconfig",
  "compilerOptions": {
    "strict": true,
    "useUnknownInCatchVariables": true,
    "allowJs": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "isolatedModules": true,
    "newLine": "lf",
    "module": "ESNext",
    "moduleResolution": "node",
    "target": "ESNext",
    "lib": ["ESNext", "dom"],
    "esModuleInterop": true,
    "allowSyntheticDefaultImports": true,
    "baseUrl": "./src",
    "noEmit": false,
    "incremental": true,
    "resolveJsonModule": true,
    "paths": {}
  },
  "exclude": ["**/node_modules", "**/.*/", "./dist", "./coverage"],
  "include": ["**/*.ts", "**/*.js", "**/.cjs", "**/*.mjs", "**/*.json"]
}
```

위와 같이 기본 설정을 진행 한 후 src → index.ts 파일을 만든 후 아래의 코드를 실행합니다. 

```bash
export const sayHello = () => {
  return "sayHello 함수 호출";
};
```

이후에 사용하고자 하는 apps → web에 위에서 설정한 lib 폴더의 의존성을 주입해줘야 합니다. 

해당 의존성을 주입하기 위해서 root로 돌아와서 아래와 같은 명령어를 실행합니다. 

```bash
yarn workspace @hp/web add @hp/lib
```

그렇게 적용하고 나면 apps 폴더에 package.json에는 lib의 의존성이 설치됩니다. 

```bash
{
  "name": "@hp/web",
  "version": "0.1.0",
  "private": true,
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "next lint"
  },
  "dependencies": {
    "@hp/lib": "workspace:^",
    "@types/node": "20.1.7",
    "@types/react": "18.2.6",
    "@types/react-dom": "18.2.4",
    "eslint": "8.40.0",
    "eslint-config-next": "13.4.2",
    "next": "13.4.2",
    "react": "18.2.0",
    "react-dom": "18.2.0",
    "typescript": "5.0.4"
  }
}
```

위와 같이 적용됐으면 apps 폴더 밑에 web에 들어가서 sayHello를 호출해주면 아래와 같은 결과를 얻을 수 있습니다. 

![스크린샷 2023-05-18 오전 9.49.36.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a9457d0d-5bff-4541-94f2-48f2e9bcdc01/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2023-05-18_%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB_9.49.36.png)

### 타입스크립트 tsconfig.json 공유하기

모노레포를 사용하는 이유중에 하나는 모든 프로젝트의 통일성을 제공하기 위함이다. 예를 들어 ESLint , Prettier , tsconfig.json … 공통으로 사용하는 rule들을 모든 프로젝트에 공유함으로써 통일성을 주기 때문이다. 

먼저 tsconfig.json을 공유하는 방법부터 알아보겠습니다. 

먼저 기본이 되는 tsconfig 파일을 root에 tsconfig.base.json이라는 이름으로 파일을 만들고 아래와 같이 적용해줍니다. 

```bash
{
  "compilerOptions": {
    "strict": true,
    "useUnknownInCatchVariables": true,
    "allowJs": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "noEmit": true,
    "esModuleInterop": true,
    "moduleResolution": "node",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "incremental": true,
    "newLine": "lf"
  },
  "exclude": ["**/node_modules", "**/.*/"]
}
```

이 파일을 다른 프로젝트 파일에 적용시키기 위해 각각의 tsconfig.json 파일에 들어가서 root에서 만들어 준 tsconfig.base.json 파일을 extends 해줍니다.

`lib` 폴더에는 아래와 같은 설정을 진행합니다. 

```bash
{
  "$schema": "https://json.schemastore.org/tsconfig",
  "extends": "../../tsconfig.base.json",
  "compilerOptions": {
    "module": "ESNext",
    "moduleResolution": "node",
    "target": "ESNext",
    "lib": ["ESNext", "dom"],
    "esModuleInterop": true,
    "allowSyntheticDefaultImports": true,
    "baseUrl": "./src",
    "noEmit": false,
    "incremental": true,
    "resolveJsonModule": true
  },
  "exclude": ["**/node_modules", "**/.*/", "./dist", "./coverage"],
  "include": ["**/*.ts", "**/*.js", "**/.cjs", "**/*.mjs", "**/*.json"]
}
```

`apps` 폴더에는 아래와 같은 설정을 진행합니다. 

```bash
{
  "$schema": "https://json.schemastore.org/tsconfig",
  "extends": "../../tsconfig.base.json",
  "compilerOptions": {
    "baseUrl": "./src",
    "target": "esnext",
    "lib": ["dom", "dom.iterable", "esnext"],
    "module": "esnext",
    "jsx": "preserve",
    "incremental": true
  },
  "exclude": ["**/node_modules", "**/.*/"],
  "include": [
    "next-env.d.ts",
    "**/*.ts",
    "**/*.tsx",
    "**/*.mts",
    "**/*.js",
    "**/*.cjs",
    "**/*.mjs",
    "**/*.jsx",
    "**/*.json"
  ]
}
```

### Prettier와 ESLint 공유하기

prettier와 ESLint를 공유하기 위해 먼저 prettier , eslint 설정에 필요한 모듈들을 root에서 설치해줍니다. 

그리고 타입스크립트와 마찬가지로 sdk를 설치해줍니다. 

```bash
yarn add prettier eslint eslint-config-prettier eslint-plugin-import eslint-plugin-react eslint-plugin-react-hooks eslint-import-resolver-typescript @typescript-eslint/eslint-plugin @typescript-eslint/parser -D

yarn dlx @yarnpkg/sdks
```

그리고 settings.json에 아래와 같은 코드를 추가해줍니다. 

```bash
"editor.defaultFormatter": "esbenp.prettier-vscode",
"editor.formatOnSave": true,
"editor.rulers": [
  120
],
```

이후 apps 폴더에서 저장하기 버튼을 클릭하게 되면 prettier가 잘 동작하는것을 확인할 수 있습니다. 

eslint또한 root에 eslintrc.js 파일을 생성후에 아래와 같이 설정 파일을 적용하고 다른 eslint 파일들은 모두 삭제합니다.

```bash
module.exports = {
  root: true,

  env: {
    es6: true,
    node: true,
    browser: true,
  },

  parser: "@typescript-eslint/parser",
  parserOptions: {
    ecmaFeatures: { jsx: true },
  },

  extends: [
    "eslint:recommended",
    "plugin:@typescript-eslint/recommended",
    "plugin:react/recommended",
    "plugin:react-hooks/recommended",
    "prettier",
  ],
  plugins: ["@typescript-eslint", "import", "react", "react-hooks"],
  settings: {
    "import/resolver": { typescript: {} },
    react: { version: "detect" },
  },
  rules: {
    "no-implicit-coercion": "error",
    "no-warning-comments": [
      "warn",
      {
        terms: ["TODO", "FIXME", "XXX", "BUG"],
        location: "anywhere",
      },
    ],
    curly: ["error", "all"],
    eqeqeq: ["error", "always", { null: "ignore" }],

    // Hoisting을 전략적으로 사용한 경우가 많아서
    "@typescript-eslint/no-use-before-define": "off",
    // 모델 정의 부분에서 class와 interface를 합치기 위해 사용하는 용법도 잡고 있어서
    "@typescript-eslint/no-empty-interface": "off",
    // 모델 정의 부분에서 파라미터 프로퍼티를 잘 쓰고 있어서
    "@typescript-eslint/explicit-function-return-type": "off",
    "@typescript-eslint/no-parameter-properties": "off",
    "@typescript-eslint/no-var-requires": "warn",
    "@typescript-eslint/no-non-null-asserted-optional-chain": "warn",
    "@typescript-eslint/no-inferrable-types": "warn",
    "@typescript-eslint/no-empty-function": "off",
    "@typescript-eslint/naming-convention": [
      "error",
      {
        format: ["camelCase", "UPPER_CASE", "PascalCase"],
        selector: "variable",
        leadingUnderscore: "allow",
      },
      { format: ["camelCase", "PascalCase"], selector: "function" },
      { format: ["PascalCase"], selector: "interface" },
      { format: ["PascalCase"], selector: "typeAlias" },
    ],
    "@typescript-eslint/explicit-module-boundary-types": "off",
    "@typescript-eslint/array-type": ["error", { default: "array-simple" }],
    "@typescript-eslint/no-unused-vars": [
      "error",
      { ignoreRestSiblings: true },
    ],
    "@typescript-eslint/member-ordering": [
      "error",
      {
        default: [
          "public-static-field",
          "private-static-field",
          "public-instance-field",
          "private-instance-field",
          "public-constructor",
          "private-constructor",
          "public-instance-method",
          "private-instance-method",
        ],
      },
    ],

    "import/order": [
      "error",
      {
        groups: [
          "builtin",
          "external",
          "internal",
          "parent",
          "sibling",
          "index",
          "object",
        ],
        alphabetize: { order: "asc", caseInsensitive: true },
      },
    ],

    "react/prop-types": "off",
    // React.memo, React.forwardRef에서 사용하는 경우도 막고 있어서
    "react/display-name": "off",
    "react-hooks/exhaustive-deps": "error",
    "react/react-in-jsx-scope": "off",
    "react/no-unknown-property": ["error", { ignore: ["css"] }],
  },
};
```

```bash
"eslint.packageManager": "yarn",
"eslint.validate": ["javascript", "javascriptreact", "typescript", "typescriptreact"],
```

📌 참고 
https://github.com/sungkwangkim/yarn-berry-test