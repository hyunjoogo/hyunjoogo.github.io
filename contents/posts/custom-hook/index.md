---
title: "정체 모를 함수에서 훅 사용은 자제 부탁드릴께요 🙏(Feat.인스타감성)"
description: "useNavigate를 모듈화 시키고 싶어요."
date: 2023-05-01
update: 2023-05-01
tags:
- React
- React-custom-hook
- react-hooks/rules-of-hooks
series: "리액트 커스텀 훅"
---
부제 : useNavigate를 모듈화 시키고 싶어요.

# 상황 : useNavigate()를 모듈로 변경하고 싶어요.

react-router-dom의 useNavigate는 사용자의 URL를 변경하는 명령형 메서드입니다. (v6 기준)

<Link>와 비슷한 역할을 합니다.  사용방법은 아래와 같습니다.

```jsx
import { useNavigate } from "react-router-dom";

const navigate = useNavigate();

navigate('/', { replace: true });
```

하지만 사용하는 곳마다 매번 이렇게 사용해야하므로 모듈화를 시키고 싶었습니다.

```jsx
// redirect.js
import { useNavigate } from "react-router-dom";

export const redirect = (url) => {
  const navigate = useNavigate();

  navigate(url, { replace: true });
};

// 사용하는 컴포넌트
import { redirect } from "../redirect";

if (사용자정보) {
	redirect("/");
}
```

이렇게 redirect라는 함수를 만들었습니다. 원하는 곳에서 redirect(URL)만 호출하면 됩니다.

저는 서버로부터 받은 데이터에 사용자의 정보가 있으면 사용자의 URL를 변경하려고 합니다.

## 오류: eslint : 정체 모를 함수에서 훅 사용은 자제하실께요🙏

```
Line 4:20: React Hook "useNavigate" is called in function "redirect" that is neither a React function component nor a custom React Hook function. React component names must start with an uppercase letter. React Hook names must start with the word "use" react-hooks/rules-of-hooks

// 위의 에러를 번역했습니다.
리액트 함수 컴포넌트도 아니고 커스텀 훅 함수도 아닌 "redirect" 함수에서 리액트 훅 "useNavigate"가 호출되었습니다. 리액트 컴포넌트 이름은 대문자로 시작해야 합니다. 리액트 훅 이름은 "use"라는 단어로 시작해야 합니다. react-hooks/rules-of-hooks
```

침착하게 한글 오류를 읽고 생각을 해보도록 합시다.

## 원인: 고객님 공식문서 숙지 부탁드려요 🙏

"useNavigate" 리액트 훅이 "redirect" 함수에서 호출되었지만, "redirect" 함수는 리액트 함수 컴포넌트도 아니고 커스텀 리액트 훅도 아닙니다.

오류 메시지가 친절하게 알려주듯이, 리액트 컴포넌트 이름은 반드시 대문자로 시작해야 하고, 리액트 훅 이름은 반드시 "use"이라는 단어로 시작해야 합니다.

그러니까 함수 컴포넌트로 변경하던지 커스텀 훅 명명 규칙을 따라달라는 거죠. 거기다가 너무 친절하게 ~~인스타공지 아니~~  [공식문서 숙지](https://legacy.reactjs.org/docs/hooks-rules.html)를 부탁하는 군요. (참고로 몇달전에 [React 공식문서](https://react.dev/)가 새단장을 했습니다)

공식문서는 두 가지의 룰을 지켜달라고 합니다.

✅ 최상위 레벨에서만 훅 호출

✅ 리액트 함수에서만 훅 호출

### 최상위 레벨에서만 훅 호출

공식문서를 읽어보시면 루프, 조건문 또는 중첩 함수 내부에서 호출하지 말고 초기 리턴이 오기전에 최상위 수준에서 호출하라고 합니다. 컴포넌트가 렌더링될 때마다 훅이 같은 순서로 호출되는 것을 보장할 수 있다고 하네요.

공식문서에 있는 예시를 가지고 왔습니다.

```jsx
// 🔴 조건문 내부에서 useEffect라는 훅을 호출한 나쁜 예시
  if (name !== '') {
    useEffect(function persistForm() {
      localStorage.setItem('formData', name);
    });
  }

// 👍 useEffect라는 훅 내부에서 조건문을 사용하는 좋은 예시
useEffect(function persistForm() {
    if (name !== '') {
      localStorage.setItem('formData', name);
    }
  });
```

### 리액트 함수에서만 훅 호출

훅을 사용할 수 있는 함수에서만 사용하라는 이야기입니다. 아래 두 규칙을 지키는 함수에서만 가능합니다.

✅ 리액트 함수 컴포넌트에서 훅을 호출합니다. (대문자로 시작하는 함수 컴포넌트)

✅ 커스텀 훅에서 훅을 호출할 수 있습니다. (useRedirect ⭕️, redirect ❌)

함수 컴포넌트 또는 use~~~로 시작하는 함수(우리는 커스텀 훅이라고 부르기로 했어요)에서만 사용할 수 있다고 하네요.

## 해결책 : 문제해결은 아니지만 오류에 대한 해결책(?)

모듈화를 하겠다고 만든 코드가 알고보니 커스텀 훅인 상황이 된겁니다.

이제 근본 원인을 파악했으니 오류를 해결해보도록 하겠습니다. 이 오류를 해결하려면 두 가지 옵션이 있습니다:

1. **"redirect"를 리액트 함수 컴포넌트로 변환하기:** 1.
   - "redirect" 함수의 이름을 "Redirect"와 같이 대문자로 시작하도록 바꿉니다.
   - 이건 우리가 원하는 방향이 아닙니다.
2. **사용자 정의 리액트 훅 생성하기:** 2.
   - "redirect" 함수의 이름을 "useRedirect"와 같이 훅의 명명 규칙을 따르도록 바꿉니다.
   - 파일명도 야심차게 바꾸었습니다.

    ```jsx
    // useRedirect.js
    export const useRedirect = (url) => {
      const navigate = useNavigate();
    
      navigate(url, { replace: true });
    };
    
    // 사용하는 컴포넌트
    import { redirect } from "../redirect";
    
    if (사용자정보) {
    	redirect("/");
    }
    ```

   - 아…. 조건문에서 사용하지 말라고 했지…. 원하는 방향이지 사용할 수 없군요


그렇다면 조건문이 아닌 곳에서도 저렇게 사용할 수 있을까요?

정답은 “아니요”입니다.  `const navigate = useNavigate();` 이것 또한 커스텀 훅이기 때문에 룰을 지켜야하기 때문에 안되는거죠.

즉, 이 모듈을 사용할 수가 없습니다. 아래와 같이 사용하는 컴포넌트마다 훅을 만들어줘야합니다.

```jsx
import { redirect } from "../redirect";

const navigate = useNavigate();

if (사용자정보) {
	navigate('/', { replace: true });
}
```

## 결론적으로: 배운 교훈

네. 제목이 곧 내용이었습니다. 오류는 해결했지만 문제해결을 못하게 된 거지요.

`const navigate = useNavigate();` 이것을 없애고 사용하려고 했지만 안타깝네요. 그래도 커스텀 훅을 만드는 규칙과 공식문서를 한번이라도 더 보게 되는 이벤트가 발생했네요. 만….족 못하지만 만족합니다 😂
