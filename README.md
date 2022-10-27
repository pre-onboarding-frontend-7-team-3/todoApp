# 원티드 프리온보딩 프론트엔드 3팀 - Assignment #1

[사전 과제](https://github.com/walking-sunset/selection-task)로 구현한 Todo 앱에 대한 **Best Practice**

>[조은지](https://github.com/Joeunji0119/)(팀장)
> [김창희](https://github.com/PiperChang/) [문지원](https://github.com/moonkorea00/) [박정민](https://github.com/ono212/) [이상민](https://github.com/dltkdals224/) [이지원](https://github.com/365support/) [조수진](https://github.com/suzz-in/) [고영훈](https://github.com/YeonghunKO/)

## **배포 주소**

[https://todoapp365.netlify.app](https://todoapp365.netlify.app)

>테스트 계정 :
>
>##### 아이디 : test@test.com
>
>##### 비밀번호 : password!@

## 목차

- [리팩토링](#리팩토링)
- [폴더 구조](#폴더-구조)
- [팀 코드 컨벤션](#팀-코드-컨벤션)

## 리팩토링

#### 라이브 코드리뷰로 각자 구현한 코드에 대한 피드백 및 리팩토링 후 Best Practice를 채택했습니다.

- [ ] React Suspense + dynamic import로 lazy loading

- 변경을 하는 부분에서 suspense를 Route 전체를 감싸주어야 하는지, Todo 컴포넌트만 감싸줘야하는지 고민을 했습니다. 오히려 lazy loading 시 더 느려질 수 있음을 고려하여, Todo 컴포넌트에만 suspense를 적용시켰습니다.

```javascript
const Todo = lazy(() => import("./pages/Todo"));

function App() {
  return (
    <Routes>
      <Route path="/" element={<Auth />} />

      <Route
        path="/todo"
        element={
          <Suspense fallback={<div css={mainContainer}>...로딩중</div>}>
            <Todo />
          </Suspense>
        }
      />
    </Routes>
  );
}
```

참고 파일: [src/api/client.js](https://github.com/365support/wanted-pre-onboarding-frontend/blob/main/src/api/client.js)

<br />

- [ ] useAuth hook으로 사용자 리다이렉트 처리

- 토큰에 다른 값이 들어갔을 때를 방지하기 위해서 토큰이 맞는지 확인하는 과정입니다. 토큰이 맞는지 확인하는 api가 없기 때문에 토큰이 필요한 api에 요청하고 결과에 따라서 처리했습니다. 

```javascript
const useAuth = () => {
  const navigate = useNavigate();

  useEffect(() => {
    const getData = () => {
      getTodoApi()
        .then((res) => {
          navigate("/todo");
        })
        .catch((err) => {
          navigate("/");
        });
    };
    getData();
  }, []);

  return;
};
```

<br />

- [ ] 컴포넌트 style 관련 디렉토리  개선

- 상태값을 받아와 사용하는 css 가 태그 안에 존재해 보기 힘든 부분이 있어 emotion/styled를 통해 추출하고 css 파일을 분리해서 확장 가능성이 좋게 수정했습니다.

- div 내 style 관련 attribute의 길어짐이 Component구성 방식을 바라보는 시각적 사고를 널뛰게한다고 판단했습니다.
- 
- @emtion/react의 css를 통해 style관련 폴더로 모듈화하는 방식으로 수정하였습니다.

```
// TodoItem.jsx

import * as todoItemStyle from "./style";

// ..

<todoItemStyle.CheckBox isCompleted={list.isCompleted} onClick={(e) => onCheckClick(e)}>
		{list.isCompleted && <MdDone />}
</todoItemStyle.CheckBox>
```

```javascript
// style.js

import { css } from "@emotion/react";
import { COLOR } from "../../../shared/style";
import styled from "@emotion/styled";

// ..

export const CheckBox = styled.div`
  ${checkCircle}
  border: ${({ isCompleted }) => (isCompleted ? ` 1px solid ${COLOR.White100}` : "")};
  color: ${({ isCompleted }) => (isCompleted ? `${COLOR.White100}` : "")};
`;
```

> 참고 파일: [src/api/client.js](https://github.com/365support/wanted-pre-onboarding-frontend/blob/main/src/api/client.js)

<br />

- [ ] useValidatedEmail/useValidatedPassword 함수에서 중복되는 로직을 useValidate hook으로 관리
 
- 이메일 조건: @ 포함, 비밀번호 조건: 8자 이상
- 입력된 이메일과 비밀번호가 유효성 검사를 통과할 때만 로그인/회원가입 버튼이 활성화됩니다.
- `useValidate` 커스텀 훅을 사용하여 이메일과 비밀번호의 유효성 검사를 해주었습니다.
- 각각 이메일과 비밀번호의 유효성 검사를 할 때 로직을 재사용할 수 있도록 `type`을 인자로 받아 사용할 수 있도록 훅으로 분리했습니다.
- 인자 : `type`
    - `“email”` | `“password”`
    - 인자로 유효성 검사를 할 type을 받아 type에 따라 유효성 검사와 경고 메세지를 보여줍니다.
- 반환값 : `[validity, warnList, oncheckValidity]`
    - `validity` : 유효성 검사를 통과하는지 여부 (boolean)
    - `warnList` : 유효성 검사를 통과하지 않을 때(validity === true) 보여줄 경고메시지
    - `onCheckValidity` : type에 따른 유효성 검사를 하는 로직이 있는 함수

```javascript
const useValidate = (type) => {
  const [validity, setValidity] = useState(false);
  const [warnList, setWarnList] = useState([]);

  const oncheckValidity = (text) => {
    const warnList = [];
    if (!text) {
      return setValidity(false);
    }
    const regexforValAuth = {
      password: {
        warnText: "비밀번호는 8글자 이상이어야 합니다.",
        fn: new RegExp("(?=.{8,})"),
      },
      email: {
        warnText: "이메일에는 @가 포함되어야 합니다.",
        fn: new RegExp("@"),
      },
    };
    const { warnText, fn } = regexforValAuth[type];
    if (!fn.test(text)) {
      warnList.push(warnText);
    }

    setWarnList(warnList);
    if (warnList.length > 0) {
      setValidity(false);
    } else {
      setValidity(true);
    }
  };
  return [validity, warnList, oncheckValidity];
};
```

> 참고 파일: [src/api/client.js](https://github.com/365support/wanted-pre-onboarding-frontend/blob/main/src/api/client.js)

<br />

- [ ] 낙관적 업데이트 context API로 개선

> #### 리팩토링 전

- 낙관적 업데이트를 하는 함수가 산재되어있었습니다.
    - `todolist.jsx`에 update , delete 함수가 있었구요,  `todoItem.jsx`에  넘겨주고 있었습니다.

```javascript
// todoList.jsx
const TodoList = ({ todoData, setTodoData }) => {
  const handleTodoUpdate = useCallback(
    ...
    [todoData]
  );

  const handleTodoDelete = useCallback(
   ...
    [todoData]
  );

  return (
    <div css={todoWrapper}>
      {todoData?.map((list, id) => (
        <TodoItem
          key={list.id}
          list={list}
          handleTodoUpdate={handleTodoUpdate}
          handleTodoDelete={handleTodoDelete}
        />
      ))}
    </div>
  );
};
```

> #### 리팩토링 후

- `context api` + `useReducer`를 이용해서 업데이트함수를 `todoItem.jsx`로 모았습니다.
    - 한 곳에서 관리하기 때문에 유지보수가 편해졌습니다
    - props drlling을 하지 않아도 됩니다.

```javascript
// todoList.jsx
const TodoList = () => {
  const todoData = useContext(todoContext);
  const dispatch = useContext(dispatchContext);

  return (
    <ul css={todoWrapper}>
      {todoData?.map((list, id) => (
        <TodoItem key={list.id} list={list} />
      ))}
    </ul>
  );
};


// todoItem.jsx
const TodoItem = ({ list }) => {

  const dispatch = useContext(dispatchContext);

  const handleTodoUpdate = useCallback(
    content => {
      // console.log('content', content);
      updateTodoApi(content.id, content.todo, content.isCompleted)
        .then(res => {
          console.log('res', res);
          dispatch({ type: 'EDIT', todo: res.data });
        })
        .catch(err => {
          console.log('주 에러 : ', err);
        });
    },
    [list, content]
  );

  const handleTodoDelete = useCallback(
    id => {
      deleteTodoApi(id)
        .then(res => {
          dispatch({ type: 'DELETE', id });
        })
        .catch(err => {
          console.log('주 에러 : ', err);
        });
    },
    [list]
  );
...
}
```

참고 파일: src/component/Todo

- [ ] 시멘틱한 마크업

> 참고 파일: [src/api/client.js](https://github.com/365support/wanted-pre-onboarding-frontend/blob/main/src/api/client.js)


## **폴더 구조**


```
📦 src
├── 📂 api
├── 📂 component
│   ├── 📂 auth
│   │    ├── 📄 Login
│   │    └── 📄 SignUP
│   ├── 📂 todo
│   │    ├── 📄 TodoHeader
│   │    ├── 📄 TodoCreate
│   │    ├── 📄 TodoList
│   │    └── 📄 TodoItem
├── 📂 hooks
├── 📂 pages
│   ├── 📄 auth
│   └── 📄 todo
└── 📂 utils

```


## 팀 코드 컨벤션

- [ ] git commit message 컨벤션

| 커밋명 | 내용 |
| --- | --- |
| Feat | 파일, 폴더, 새로운 기능 추가 |
| Fix | 버그 수정 |
| Docs | 제품 코드 수정 없음 |
| Style | 코드 형식, 정렬, 주석 등의 변경 |
| Refactor | 코드 리팩토링 |
| Test | 테스트 코드 추가 |
| Chore | 환경설정, 빌드 업무, 패키지 매니저 설정등.. |
| Hotfix | 치명적이거나 급한 버그 수정 |

- [ ] branch 컨벤션

| 브랜치명 | 내용 |
| --- | --- |
| feature | 파일, 폴더, 새로운 기능 추가 |
| fix | 버그 수정 |
| docs | 제품 코드 수정 없음 |
| refactor | 코드 리팩토링 |
| hotfix | 치명적이거나 급한 버그 수정 |
