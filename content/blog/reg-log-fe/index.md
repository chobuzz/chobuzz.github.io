---
title: 회원가입 / 로그인 만들기 Frontend
date: "2023-10-28"
description: 프론트엔드 회원가입 / 로그인 만들기
---

- 시작 전

### 회원가입, 로그인 기능을 구현하기 Frontend 기본 파일

https://github.com/legobitna/todo-fe-login-student

이 파일 Clone을 시작으로 회원가입 / 로그인을 구현함.

(기본적인 UI세팅만 되어있는 파일)

# 회원가입 / 로그인 만들기 Frontend

---

회원가입 및 로그인 기능 Frontend 구현.

## 회원가입 Frontend 만들기

---

1. name, email, password, re-enter password 에 대한 값을 받아야 한다.
2. password와 re-enter password가 일치하는지 확인해야한다.
3. pasword가 일치하지 않으면 에러를 보내고,
4. password가 일치하면, 회원가입 버튼을 누를 시 위에 해당하는 값을 백엔드에 전송해야한다.
5. 회원가입이 성공적으로 완료가 되면, 로그인 페이지로 넘겨주기.

```jsx
import React, { useState } from "react"
import { useNavigate } from "react-router-dom"
import Button from "react-bootstrap/Button"
import Form from "react-bootstrap/Form"
import api from "../utils/api"

const RegisterPage = () => {
  const [name, setName] = useState("")
  const [email, setEmail] = useState("")
  const [password, setPassword] = useState("")
  const [secPassword, setSecPassword] = useState("")
  const [error, setError] = useState("")
  const navigate = useNavigate()

  const handleSubmit = async event => {
    event.preventDefault()
    try {
      if (password !== secPassword) {
        //에러 : 패스워드가 일치하지 않습니다 다시 입력해주세요
        throw new Error("패스워드가 일치하지 않습니다 다시 입력해주세요")
      }
      // 일치하면 api호출
      const response = await api.post("/user", { name, email, password })
      if (response.status === 200) {
        navigate("/login")
      } else {
        throw new Error(response.data.error)
      }
    } catch (error) {
      setError(error.message)
      console.log("error", error.message)
    }
  }

  return (
    <div className="display-center">
      {error && <div className="red-error">{error}</div>}
      <Form className="login-box" onSubmit={handleSubmit}>
        <h1>회원가입</h1>
        <Form.Group className="mb-3" controlId="formName">
          <Form.Label>Name</Form.Label>
          <Form.Control
            type="string"
            placeholder="Name"
            onChange={event => setName(event.target.value)}
          />
        </Form.Group>

        <Form.Group className="mb-3" controlId="formBasicEmail">
          <Form.Label>Email address</Form.Label>
          <Form.Control
            type="email"
            placeholder="Enter email"
            onChange={event => setEmail(event.target.value)}
          />
        </Form.Group>

        <Form.Group className="mb-3" controlId="formBasicPassword">
          <Form.Label>Password</Form.Label>
          <Form.Control
            type="password"
            placeholder="Password"
            onChange={event => setPassword(event.target.value)}
          />
        </Form.Group>

        <Form.Group className="mb-3" controlId="formBasicPassword">
          <Form.Label>re-enter the password</Form.Label>
          <Form.Control
            type="password"
            placeholder="re-enter the password"
            onChange={event => setSecPassword(event.target.value)}
          />
        </Form.Group>

        <Button className="button-primary" type="submit">
          회원가입
        </Button>
      </Form>
    </div>
  )
}

export default RegisterPage
```

## 로그인 Frontend 만들기 ( ⭐ 도전과정 )

---

### 개발 순서

1. 유저는 로그인을 할 수 있다.
2. 로그인이 실패한 경우 에러메세지를 로그인창 상단에 보여준다.
3. 로그인 성공할 경우 유저정보를 state에 저장한다.
4. 로그인이 성공한 경우 토큰값을 session storage에 저장한다.
5. 로그인이 성공한 경우 api헤더에 토큰값을 디폴트로 설정해준다.
6. 로그인이 성공한 경우 할일페이지 /로 넘어간다.

### 로그인 폼 구현

pages/LoginPage.js

- ‘useState’ 훅을 활용한 상태 관리

```jsx
const [email, setEmail] = useState("");
const [password, setPassword] = useState("");

<Form.Control
	type="email"
	placeholder="Enter email"
	onChange={(event) => setEmail(event.target.value)}
/>

<Form.Control
	type="password"
	placeholder="Password"
	onChange={(event) => setPassword(event.target.value)}
/>
```

- 유저가 로그인 정보를 입력하여 제출하였을 때

```jsx
const handleLogin = async (event) => {
    event.preventDefault();
    try {
      if (email === "" || password === "") {
        setError("이메일과 비밀번호를 모두 입력하세요.");
      } else {
        //여기에서 로그인 로직 구현. 성공하면 다음 페이지로 리디렉션.
        //실패하면 다른 오류 메세지 설정.
        const response = await api.post("/user/login", { email, password });
        if (response.status === 200) {
          navigate("/");
        } else {
          setError(response.data.error);
        }
      }
    } catch (error) {
      setError(error.message);
    }
  };

return (
    <div className="display-center">
      {error && <div className="red-error">{error}</div>}
      <Form className="login-box" onSubmit={handleSubmit}>
```

- 유저정보 저장, 토큰 저장

```jsx
if (response.status === 200) {
  //유저정보 저장
  setUser(response.data.user)
  //sessionStorage에 토큰 저장
  sessionStorage.setItem("toekn", response.data.token)

  navigate("/")
}
```

- handleLogin 최종

```jsx
const handleLogin = async event => {
  event.preventDefault()
  try {
    if (email === "" || password === "") {
      setError("이메일과 비밀번호를 모두 입력하세요.")
    } else {
      //여기에서 로그인 로직 구현. 성공하면 다음 페이지로 리디렉션.
      //실패하면 다른 오류 메세지 설정.
      const response = await api.post("/user/login", { email, password })
      console.log("rrrrr", response)
      if (response.status === 200) {
        //유저정보 저장
        setUser(response.data.user)
        //sessionStorage에 토큰 저장
        sessionStorage.setItem("toekn", response.data.token)
        //토큰값을 api헤더에 저장하기 (api.body는 post에서만 유효하고 get에서는 활용하지 못하기 때문)
        api.defaults.headers["authorization"] = "Bearer " + response.data.token
        setError("")
        navigate("/")
      } else {
        setError(response.message)
      }
    }
  } catch (error) {
    setError(error.message)
  }
}
```

이 내용은 코딩알려주는누나 풀스택 강의를 보면서 공부한 내용입니다!!

관심있으신분들은 [코딩알려주는누나](https://codingnoona.thinkific.com/) 강의를 참고하세요!!

출처: [코딩알려주는누나](https://codingnoona.thinkific.com/) 풀스택 강의
