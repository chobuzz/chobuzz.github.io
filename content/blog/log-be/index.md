---
title: 로그인 심화 Backend
date: "2023-10-30"
description: Backend 로그인 심화
---

## 만들어야 하는 2가지 기능

---

### 1. 이미 로그인을 한 유저는 로그인페이지로 돌아갈 수 없다.

1. 로그인을 했다면, 토큰을 저장한다.
2. 토큰값을 읽어온다.
3. 토큰이 사용 가능한 토큰인지 체크한다 (토큰이 만료되지 않고, 토큰을 해독했을 때 유저 ID가 있다 = 백엔드에서 체크)
4. 토큰이 사용가능하면, 토큰을 바탕으로 유저객체를 보내준다.
5. 유저값을 저장을 한다.
6. 유저가 있다면 투두 페이지를 보여준다.

### 2. 로그인을 안한 유저는 절대 투두페이지로 들어갈 수 없다.

### 1. 이미 로그인을 한 유저는 로그인페이지로 돌아갈 수 없다.(Frontend)

---

- Private Route : 페이지별 권한관리의 시작

App.js

```jsx
<Routes>
      <Route
        path="/"
        element={
          <PrivateRoute>
            <TodoPage />
          </PrivateRoute>
        }
      />
```

PrivateRoute로 TodoPage 감싸주기

- PrivateRoute 생성

src/route/PrivateRoute.js 생성하기

```jsx
import React from "react"

const PrivateRoute = ({ user }) => {
  return <div>PrivateRoute</div>
}

//user값이 있으면, Todopage : redirect to /login

export default PrivateRoute
```

- user생성

App.js

```jsx
const [user, setUser] = useState(null);

...

<PrivateRoute user={user}>
```

user생성 후, PrivateRoute에 user Props 전달.

- PrivateRoute.js 구현

```jsx
import React from "react"
import { Navigate } from "react-router-dom"

const PrivateRoute = ({ user, children }) => {
  return (
    //children은 react안에서 쓰는 Props (따로 user처럼 app.js에서 안줘도 됨)
    //user값이 있으면, Todopage : redirect to /login
    user ? children : <Navigate to="/login" />
  )
}

export default PrivateRoute
```

- 토큰값 읽어오기

```jsx
function App() {
  const [user, setUser] = useState(null);
  const getUser = async () => {
    try {
      const token = sessionStorage.getItem("token");
      // const response = api.get("/user/????")
    } catch (error) {}
  };
```

### 1. 이미 로그인을 한 유저는 로그인페이지로 돌아갈 수 없다.(Backend)

---

토큰을 통해 유저 id빼내고 → 그 아이디로 유저 객체 찾아서 보내주기

user.api.js

```jsx
//토큰을 통해 유저 id빼내고 → 그 아이디로 유저 객체 찾아서 보내주기
router.get("/me", authController.authenticate)
```

라우터 생성 (authController.authenticate 함수 만들어 주기)

controllers/auth.controller.js 생성 (authController.authenticate 함수 만들어 주기)

```jsx
const authController = {}
const jwt = require("jsonwebtoken")
require("dotenv").config()
const JWT_SECRET_KEY = process.env.JWT_SECRET_KEY

authController.authenticate = (req, res, next) => {
  try {
    const tokenString = req.headers.authorization // Bearer {TokenValue}
    if (!tokenString) {
      throw new Error("invalid token")
    }
    const token = tokenString.replace("Bearer ", "")
    jwt.verify(token, JWT_SECRET_KEY, (error, payload) => {
      if (error) {
        throw new Error("invalid token")
      }
      console.log("payload???", payload)
    })
  } catch (error) {
    res.status(400).json({ status: "fail", message: error.message })
  }
}

module.exports = authController
```

- postman을 이용해서 에러와 API 통신 확인하기.

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/599f6e82-7928-4b8a-a253-7d3f844feede/089283a7-47ab-47ca-9db4-0c28015e8548/Untitled.png)

말도 안되는 값을 보냈을 때, 오류가 잘 뜸

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/599f6e82-7928-4b8a-a253-7d3f844feede/3c6cbeea-5980-45fb-ae74-04dfa291c702/Untitled.png)

올바른 값을 보내면 payload??? 가 잘 뜨는것을 확인할 수 있음.

controllers/auth.controller.js

```jsx
res.status(200).json({ status: "success", userId: payload._id })
```

해준 후, 포스트맨 확인하면,

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/599f6e82-7928-4b8a-a253-7d3f844feede/f7ac2bcb-03ea-488a-a3d7-133417614320/Untitled.png)

아주 잘뜸

> 여기까지 이 토큰이 사용가능한 토큰인지 확인하는 작업을 완료했다.
> 지금부터는 이 토큰과 우리가 가지고 있는 아이디를 가지고, User 객체를 보내주는 작업을 할 예정

위의 예제에서

controllers/auth.controller.js

```jsx
authController.authenticate = (req, res, next) => {
```

next라는 미들웨어를 썼는데, 이 미들웨어는 이 함수를 호출하여 다음 함수로 요청을 전달한다.

routes/user.api.js

```jsx
router.get("/me", authController.authenticate, userController.getUser)
```

이런식으로 다음 함수를 지정해주고,

controllers/auth.controller.js

```jsx
authController.authenticate = (req, res, next) => {
  try {
    const tokenString = req.headers.authorization // Bearer {TokenValue}
    if (!tokenString) {
      throw new Error("invalid token")
    }
    const token = tokenString.replace("Bearer ", "")
    jwt.verify(token, JWT_SECRET_KEY, (error, payload) => {
      if (error) {
        throw new Error("invalid token")
      }
      req.userId = payload._id
      next()
    })
  } catch (error) {
    res.status(400).json({ status: "fail", message: error.message })
  }
}

module.exports = authController
```

여기에서

```jsx
req.userId = payload._id
next()
```

이런식으로 다음 함수에 전달을 한다.

그러면, 다음함수인

controllers/user.controller.js에서

```jsx
userController.getUser = async (req, res) => {
  try {
    const { userId } = req //또는 req.userId
    // console.log("userid", userId);
    const user = await User.findById(userId)

    if (!user) {
      throw new Error("can not find user")
    }
    res.status(200).json({ status: "success", user })
  } catch (error) {
    res.status(400).json({ status: "fail", message: error.message })
  }
}
```

이렇게 받아서 쓸 수 있다.

이렇게 전달받은 유저정보를 postman을 통해 받아보자

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/599f6e82-7928-4b8a-a253-7d3f844feede/a1e4e41c-3efb-4a4e-ae58-fe4027573c28/Untitled.png)

유저 객체가 잘 반환된다.

models/User.js에서

```jsx
userSchema.methods.toJSON = function () {
  const obj = this._doc
  delete obj.password
  delete obj.updatedAt
  delete obj.createdAt
  delete obj.__v
  return obj
}
```

필요한 정보만 받아볼 수 있게 세팅한다.

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/599f6e82-7928-4b8a-a253-7d3f844feede/5b6d7c56-ac72-4083-9e88-f5dfd280a869/Untitled.png)

getUser 백엔드 처리를 해줬으니, 다시 Frontend로 간다.

### getUser 설정하기 (Frontend)

---

로그인 한 유저는 바로 TodoPage를 보여주고, 로그인을 안한 유저는 login페이지로 넘겨주는 작업을 하기 위한 getUser 설정하기.

App.js

```jsx
function App() {
  const [user, setUser] = useState(null);

  const getUser = async () => {
    //토큰을 통해 유저정보를 가져온다.
    try {
      const storedToken = sessionStorage.getItem("token");
      if (storedToken) {
        console.log("storedToekn = true");
        const response = await api.get("/user/me");
        console.log("!!!!!!!!!", response);
        setUser(response.data.user);
      }
    } catch (error) {
      setUser(null);
    }
  };

  useEffect(() => {
    getUser();
  }, []);
```

이렇게 유저 정보를 불러오는 코드까지 짜준다.

- LoginPage에 user와 App.js에 user가 겹치므로, LoginPage의 user를 App.js에서 Props로 받아온다.

App.js

```jsx
<Route path="/login" element={<LoginPage setUser={setUser} />} />
```

LoginPage.js

```jsx
element={<LoginPage user={user} setUser={setUser} />}
```

const [user, setUser] = useState(””) 를 지워준다.

- 유저가 있으면 투두페이지로 이동하기 (Frontend)

pages/LoginPage.js

```jsx
if (user) {
    return <Navigate to="/" />;
  }

  return (
```

## 추가) 작성자 기능까지 추가로 만들어 보자.

---

현재 테이블과 들어있는 정보.

Task

- \_id
- task
- isComplete

User

- \_id
- name
- email
- password

여기에서 Task에 작성자 정보(author)도 같이 저장하고싶다.

그래서 Task테이블에서 User테이블)을 참고 할 것이다.

User테이블의 \_id를 가지고 (주키) Task테이블에 author라는 이름으로 저장할 예정(외래키)

### 개발 순서

1. 테이블(컬렉션)의 컬럼을 추가한다. author (현재 로그인한 유저가 누군지 로그인 유저정보를 알아야 한다.)
2. 할일 생성 시 author값을 추가한다.
3. 프론트엔드는 작성자 이름도 함께 보여준다.

models/Task.js (backend)

```jsx
const taskSchema = Schema(
  {
    task: {
      type: String,
      required: true,
    },
    isComplete: {
      type: Boolean,
      required: true,
    },
    author: { type: Schema.Types.ObjectId, required: true, ref: "User" },
  },
  { timestamps: true }
)
```

author를 추가해 준다.

routes/task.api.js

```jsx
router.post("/", authController.authenticate, taskController.createTask)
```

미들웨어 next()로 authCohntroller.authenticate에서 userId를 받아오기.

controllers/task.controller.js

```jsx
taskController.createTask = async (req, res) => {
  try {
    const { task, isComplete } = req.body;
    const { userId } = req;
    const newTask = new Task({ task, isComplete, author: userId })
```

받아온userId를 req에 넣어주기

- author추가 되었는지 확인하기.

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/599f6e82-7928-4b8a-a253-7d3f844feede/e6a483eb-aaff-4342-b008-fcb02ace5cdc/Untitled.png)

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/599f6e82-7928-4b8a-a253-7d3f844feede/6cc97fd9-c646-469f-b091-3cffc69dd206/Untitled.png)

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/599f6e82-7928-4b8a-a253-7d3f844feede/d265c7b0-270b-445d-b5e1-15d64ad1d4bc/Untitled.png)

author정보가 잘 들어가는것을 확인할 수 있다.

### 프론트엔드에서 작성자 이름도 함께 보여주기

pages/TodoPage

```jsx
const getTasks = async () => {
    const response = await api.get("/tasks");
    console.log("taskList", response.data.data);
```

author정보를 잘 받는지 체크

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/599f6e82-7928-4b8a-a253-7d3f844feede/c5c19afe-403e-4929-b26b-ec10e5dcabac/Untitled.png)

마지막 밥먹기에 author정보가 잘 나오는걸 확인.

### author정보를 \_id말고 다른 테이블로 populate하기

controllers/task.controller.js

```jsx
taskController.getTask = async (req, res) => {
  try {
    const taskList = await Task.find({}).populate("author")
    //populate 참고문서 https://mongoosejs.com/docs/populate.html;
    res.status(200).json({ status: "ok", data: taskList })
  } catch (err) {
    res.status(400).json({ status: "fail", error: err })
  }
}
```

populate 는 데이터베이스 join의 역할을 하는데,

여기서는 외래키 ‘author’ 를 주키 ‘\_id’의 테이블로 replace했다고 생각할 수 있다.

populate 참고문서 https://mongoosejs.com/docs/populate.html

조인

데이터베이스에서 조인이란 여러군데 흩어져 있는 정보를 외래키를 기준으로 모아주는 것이다.

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/599f6e82-7928-4b8a-a253-7d3f844feede/ba182165-2dce-4fad-b71e-5d72c7e506b1/Untitled.png)

이와같이 author의 정보가 주키 ‘\_id’의 테이블로 대체된 것을 확인 할 수 있다.

### author이름 보여주기 (프론트엔드)

---

components/TodoItem.js

```jsx
{
  item.author && <div>by {item.author.name}</div>
}
```

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/599f6e82-7928-4b8a-a253-7d3f844feede/3d08e171-362a-40e8-a511-9521425e9b78/Untitled.png)

완료!!

## 로그아웃 만들기

---

- 로그아웃 과정

1. session storage에 있는 token값 지워주기
2. user state 값 null로 바꿔주기

pages/TodoPage

```jsx
<Button variant="danger" style={{ marginTop: "10px" }} onClick={handleLogout}>
  logout
</Button>
```

버튼을 만들어준다.

```jsx
const handleLogout = () => {
  try {
    sessionStorage.removeItem("token")
    setUser(null)
  } catch (error) {
    throw new Error("로그아웃에 실패하셨습니다.")
  }
}
```

sessionStorage의 토큰값을 제거, user정보를 초기화해준다.
