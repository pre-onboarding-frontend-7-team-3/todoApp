# 원티드 프리온보딩 프론트엔드 3팀 - Assignment #1

[사전 과제](https://github.com/walking-sunset/selection-task)로 구현한 Todo 앱에 대한 **Best Practice**

>[조은지](https://github.com/Joeunji0119/)(팀장)
> [김창희](https://github.com/PiperChang/) [문지원](https://github.com/moonkorea00/) [박정민](https://github.com/ono212/) [이상민](https://github.com/dltkdals224/) [이지원](https://github.com/365support/) [조수진](https://github.com/suzz-in/) [고영훈](https://github.com/YeonghunKO/)

## **배포 주소**

📌  [https://todoapp365.netlify.app](https://todoapp365.netlify.app)

>테스트 계정 <br/> 
> 아이디 : test00@test.com 비밀번호 : password!@

## 기능 시연 GIF
### ⭐️ 로그인 , 회원가입
<img src="https://user-images.githubusercontent.com/86206374/196597041-76df2fad-5b60-4d06-b9d7-d161e55f964c.gif" width="500" height="450"/>

### ⭐️ Todo List
<img src="https://user-images.githubusercontent.com/86206374/196597578-733c4e83-6490-4539-b98b-66ce709d7b53.gif" width="500" height="450"/>

## 목차

- [리팩토링](#리팩토링)
- [폴더 구조](#폴더-구조)
- [팀 코드 컨벤션](#팀-코드-컨벤션)



## 리팩토링

#### 라이브 코드리뷰로 각자 구현한 코드에 대한 피드백 및 리팩토링 후 Best Practice를 채택했습니다.
채택 기준

1. 기능작동
2. 최적화를 위한 메모리제이션 메소드 사용
3. 컴포넌트 나누기
4. 재사용가능한 모듈

- [ ] React Suspense + dynamic import로 lazy loading

- 변경을 하는 부분에서 suspense를 Route 전체를 감싸주어야 하는지, Todo 컴포넌트만 감싸줘야하는지 고민을 했습니다. <br/> 
오히려 lazy loading 시 더 느려질 수 있음을 고려하여, Todo 컴포넌트에만 suspense를 적용시켰습니다.

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

참고 파일: [App.jsx](https://github.com/365support/wanted-pre-onboarding-frontend/blob/main/App.jsx)

<br />

- [ ] useAuth hook으로 사용자 리다이렉트 처리

- 토큰에 다른 값이 들어갔을 때를 방지하기 위해서 토큰이 맞는지 확인하는 과정입니다. <br/> 
토큰이 맞는지 확인하는 api가 없기 때문에 토큰이 필요한 api에 요청하고 결과에 따라서 처리했습니다. 

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

- 상태값을 받아와 사용하는 css 가 태그 안에 존재해 보기 힘든 부분이 있어 <br/>  emotion/styled를 통해 추출하고 css 파일을 분리해서 확장 가능성이 좋게 수정했습니다.


```js
// TodoItem.jsx
<todoItemStyle.CheckBox isCompleted={list.isCompleted}>
		{list.isCompleted && <MdDone />}
</todoItemStyle.CheckBox>
```

```javascript
export const CheckBox = styled.div`
  border: ${({ isCompleted }) => (isCompleted ? ` 1px solid ${COLOR.White100}` : "")};
`;
```

> 참고 파일: [src/api/client.js](https://github.com/365support/wanted-pre-onboarding-frontend/blob/main/src/api/client.js)

<br />

- [ ] useValidatedEmail/useValidatedPassword 함수에서 중복되는 로직을 useValidate hook으로 병합
 
- `useValidate` 커스텀 훅을 사용하여 이메일과 비밀번호의 유효성 검사를 해주었습니다.<br/>  각각 이메일과 비밀번호의 유효성 검사를 할 때 로직을 재사용할 수 있도록 `type`을 인자로 받아 사용할 수 있도록 훅으로 분리했습니다.


> 참고 파일: [src/api/client.js](https://github.com/365support/wanted-pre-onboarding-frontend/blob/main/src/api/client.js)

<br />

- [ ] 낙관적 업데이트 context API로 개선

> #### 리팩토링 전

- TodoList 업데이트를 하는 함수가 여러 폴더에 있었습니다.
- `context api` + `useReducer`를 이용해서 업데이트함수를 `todoItem.jsx`로 모았습니다.
    - 한 곳에서 관리하기 때문에 유지보수가 편해졌습니다.
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
      updateTodoApi(content.id, content.todo, content.isCompleted)
        .then(res => {
          dispatch({ type: 'EDIT', todo: res.data });
        })
        .catch(err => {
          console.log('주 에러 : ', err);
        });
    },
    [list, content]
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
