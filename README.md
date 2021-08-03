# practice-Babel
Babel 기본 학습을 위한 저장소<br>
'참고'에 있는 영상과 블로그를 따라서 진행하였습니다.

## 참고
프론트엔드 개발환경 이해 5강 - 바벨(Babel) 기본 - 김정환<br>
[영상](https://youtu.be/j8HhZ4RgBxQ)
[블로그](https://jeonghwan-kim.github.io/series/2019/12/22/frontend-dev-env-babel.html)
<br><br>
## 1. 기본 동작
### 변환 전 코드
```
// src/app.js:
const alert = msg => window.alert(msg)
```
### 설치
``` 
npm install -D @babel/core  @babel/cli
```
### 실행
```
npx babel app.js
const alert = msg => window.alert(msg);
```
바벨을 총 3단계로 빌드를 진행한다.<br>
파싱 -> 변환 -> 출력
<br><br>
## 2. 플러그인
babel은 기본적으로 코드를 받아서 코드로 변환한다.
따라서 바벨은 파싱, 출력만 담당하고 변환은 따로 진행이되는데 이것을 <strong>플러그인</strong>이라고 한다.

### 2.1 커스텀 플러그인
#### 예시
const 키워드로 선언된 변수를 var 키워드로 변경해준다.
```
// my-babel-plugin.js:
module.exports = function myplugin() {
  return {
    visitor: {
      // https://github.com/babel/babel/blob/master/packages/babel-plugin-transform-block-scoping/src/index.js#L26
      VariableDeclaration(path) {
        console.log("VariableDeclaration() kind:", path.node.kind) // const

        if (path.node.kind === "const") {
          path.node.kind = "var"
        }
      },
    },
  }
}
```
#### 빌드
```
npx babel app.js --plugins ./myplugin.js
```
#### 실행결과
```
var alert = msg => window.alert(msg);
```

### 2.2 NPM 패키지 플러그인 사용하기
#### 예시
NPM 패키지로 제공되는 플러그인으로 const, let과 같은 블록 유효 범위를 가지는 변수를 함수 유효범위를 가지는 var로 변환해준다.
```
npm install -D @babel/plugin-transform-block-scoping
```
#### 빌드
```
npx babel app.js --plugins @babel/plugin-transform-block-scoping
```
#### 실행결과
```
var alert = msg => window.alert(msg);
```

#### 설정파일 분리하기
플러그인을 여러개 사용할 경우 명령어가 길어지기에 설정파일로 분리를 해준다.
```
// babel.config.js:
module.exports = {
  plugins: [
    "@babel/plugin-transform-block-scoping",
    "@babel/plugin-transform-arrow-functions",
    "@babel/plugin-transform-strict-mode",
  ],
}
```
위의 예시로 사용한 block-scoping과 함께
arrow-function을 일반함수로 변경해주는 arrow-functions, 엄격모드를 위해 js 상단에 "use strict" 텍스트를 자동으로 작성해주는 strict-mode 플러그인을 설치 후 설정 파일에 추가하였다. 
#### 빌드
```
npx babel app.js
```
#### 실행결과
```
"use strict";

var alert = function (msg) {
  return window.alert(msg);
};
```
<br><br>
## 3. 프리셋
목적에 맞게 여러 플러그인을 세트로 모아 놓은 것을 <strong>프리셋</strong>이라고 한다.
### 3.1 커스텀 프리셋
사용한 3개 플러그인을 하나의 프리셋으로 만든다.
```
//my-babel-preset.js:
module.exports = function mypreset() {
  return {
    plugins: [
      "@babel/plugin-transform-arrow-functions",
      "@babel/plugin-transform-block-scoping",
      "@babel/plugin-transform-strict-mode",
    ],
  }
}
```

```
// babel.config.js
module.exports = {
  presets: ["./mypreset.js"],
}
```

### 3.2 프리셋 사용하기
bable은 목적에 따라 몇 가지 프리셋을 제공한다.
* preset-env
* preset-flow
* preset-react
* preset-typescript

#### preset-env
preset-env는 es6문법을 es5문법으로 변경해주는 프리셋입니다.

#### 패키지 다운로드
```
npm install -D @babel/preset-env
``` 
```
/ babel.config.js:
module.exports = {
  presets: ["@babel/preset-env"],
}
```

## 4. env 프리셋 설정과 폴리필
### 4.1 타겟 브라우저
지원해야하는 브라우저 버전을 지정할 수있다.
```
// babel.config.js :
module.exports = {
  presets: [
    [
      "@babel/preset-env",
      {
        targets: {
          chrome: "79",
          ie: "11",
        },
      },
    ],
  ],
}
```

### 4.2 폴리필
ES6의 Promise 객체를 사용하는 코드이다.
```
// app.js:
new Promise()
```
```
npx babel app.js

"use strict";

new Promise();
```
env 프리셋으로 변환을 시도했지만 Promise는 변환 되지 않았다.<br>
Babel은 변환가능 한것만 빌드하고 그렇지 못한 것들은 <strong>폴리필</storng>이라고 부르는 코드조각을 추가해서 해결한다.<br><br>
env 프리셋은 폴리필을 지정할 수 있는 옵션을 제공한다.
```
// babel.config.js:
module.exports = {
  presets: [
    [
      "@babel/preset-env",
      {
        useBuiltIns: "usage", // 폴리필 사용 방식 지정
        corejs: {
          // 폴리필 버전 지정
          version: 2,
        },
      },
    ],
  ],
}
```
useBuiltIns는 어떤 방식으로 폴리필을 사용할지 설정하는 옵션이다. "usage" , "entry", false 세 가지 값을 사용하는데 기본값이 false 이므로 폴리필이 동작하지 않았던 것이다.<br>
반면 usage나 entry를 설정하면 폴리필 패키지 중 core-js를 모듈로 가져온다(이전에 사용하던 babel/polyfile은 바벨 7.4.0부터 사용하지 않는다).
<br><br>
폴리필이 추가된 결과이다.
```
npx babel src/app.js
"use strict";

require("core-js/modules/es6.promise");
require("core-js/modules/es6.object.to-string");

new Promise();
```
core-js 패키지로부터 프라미스 모듈을 가져오는 임포트 구문이 상단에 추가되었다.



