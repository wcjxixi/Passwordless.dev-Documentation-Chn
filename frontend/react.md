# React

{% hint style="success" %}
对应的[官方页面地址](https://docs.passwordless.dev/guide/frontend/react.html)
{% endhint %}

## 如何 <a href="#how-to" id="how-to"></a>

首先，按照[此处](https://create-react-app.dev/)所述创建 React 应用程序。

生成 React 应用程序后，我们要安装我们的客户端库：

```sh
npm install @passwordlessdev/passwordless-client
```

我们的登录页面可能包含类似以下内容：

```javascript
import {useContext, useRef, useState} from "react";
import authContext from "../context/AuthProvider";
import * as Passwordless from "@passwordlessdev/passwordless-client";
import YourBackendClient from "../services/YourBackendClient";
import {PASSWORDLESS_API_KEY, PASSWORDLESS_API_URL} from "../configuration/PasswordlessOptions";

export default function LoginPage() {
    const errRef = useRef();
    const [errMsg, setErrMsg] = useState("");
    const [success, setSuccess] = useState(false);
    const { setAuth }  = useContext(authContext);

    const handleSubmit = async (e) => {
        e.preventDefault();

        // 如果是自托管，PASSWORDLESS_API_URL 将与 https://v4.passwordless.dev 不同
        const passwordless = new Passwordless.Client({
            apiUrl: PASSWORDLESS_API_URL,
            apiKey: PASSWORDLESS_API_KEY
        });
        const yourBackendClient = new YourBackendClient()
        
        // 首先我们获取我们的令牌
        const token = await passwordless.signinWithDiscoverable();
        if (!token) {
            return;
        }

        // 然后您在后端验证令牌的有效性。
        const verifiedToken = await yourBackendClient.signIn(token.token);

        // 例如，如果您的令牌被视为有效，您的后端可能会返回 JWT 令牌。
        localStorage.setItem('jwt', verifiedToken.jwt);

        // 我们很高兴能继续。
        setAuth({ verifiedToken });
        setSuccess(true);
    }

    return (
        <>
            {success ? (
                <section>
                    <h1>You are logged in!</h1>
                    <br />
                    <p>{/* <a href="#">Go to Home</a> */}</p>
                </section>
            ) : (
                <section>
                    <p
                        ref={errRef}
                        className={errMsg ? "errmsg" : "offscreen"}
                        aria-live="assertive"
                    >
                        {errMsg}
                    </p>
                    <h1>Sign In</h1>
                    <button onClick={handleSubmit}>Sign In</button>
                    <p>
                        Need an Account?
                        <br />
                        <span className="line">
              <a href="#">Sign Up</a>
            </span>
                    </p>
                </section>
            )}
        </>
    );
}
```

我们的注册页面可能看起来像这样：

```javascript
import {useEffect, useRef, useState} from "react";
import * as Passwordless from "@passwordlessdev/passwordless-client";
import {PASSWORDLESS_API_KEY, PASSWORDLESS_API_URL} from "../configuration/PasswordlessOptions";
import { ToastContainer, toast } from 'react-toastify';
import YourBackendClient from "../services/YourBackendClient";

export default function RegisterPage() {
    const userRef = useRef();
    const firstNameRef = useRef();
    const lastNameRef = useRef();
    const aliasRef = useRef();
    const errRef = useRef();
    const [user, setUser] = useState("");
    const [firstName, setFirstName] = useState("");
    const [lastName, setLastName] = useState("");
    const [alias, setAlias] = useState("");
    const [errMsg, setErrMsg] = useState("");

    useEffect(() => {
        userRef.current.focus();
    }, []);


    useEffect(() => {
        setErrMsg("");
    }, [user]);

    const handleSubmit = async (e) => {
        let registerToken = null;
        try {
            const yourBackendClient = new YourBackendClient();

            // 例如，在我们的应用程序中，除了用户名和令牌别名之外，我们还可以存储额外的字段，例如名字和姓氏。
            registerToken = await yourBackendClient.register(user, firstName, lastName, alias);
        }
        catch (error)
        {
            toast(error.message, {
                className: 'toast-error'
            });
        }

        // 如果之前发生错误，'registerToken' 将为空，因此您不想向身份验证器注册令牌。
        if (registerToken) {
            const p = new Passwordless.Client({
                apiKey: PASSWORDLESS_API_KEY,
                apiUrl: PASSWORDLESS_API_URL
            });
            const finalResponse = await p.register(registerToken.token, alias);

            if (finalResponse) {
                toast(`Registered '${alias}'!`);
            }
        }
    };

    return (
        <>
            <section>
                <p
                    ref={errRef}
                    className={errMsg ? "errmsg" : "offscreen"}
                    aria-live="assertive"
                >
                    {errMsg}
                </p>
                <h1>Register</h1>
                <label htmlFor="username">Username:</label>
                <input
                    type="text"
                    id="username"
                    ref={userRef}
                    autoComplete="off"
                    onChange={(e) => setUser(e.target.value)}
                    value={user}
                    required
                    aria-describedby="uidnote"
                />
                <label htmlFor="firstname">FirstName:</label>
                <input
                    type="text"
                    id="firstName"
                    ref={firstNameRef}
                    autoComplete="off"
                    onChange={(e) => setFirstName(e.target.value)}
                    value={firstName}
                    required
                    aria-describedby="uidnote"
                />
                <label htmlFor="lastname">LastName:</label>
                <input
                    type="text"
                    id="lastname"
                    ref={lastNameRef}
                    autoComplete="off"
                    onChange={(e) => setLastName(e.target.value)}
                    value={lastName}
                    required
                    aria-describedby="uidnote"
                />
                <label htmlFor="alias">Alias:</label>
                <input
                    type="text"
                    id="alias"
                    ref={aliasRef}
                    autoComplete="off"
                    onChange={(e) => setAlias(e.target.value)}
                    value={alias}
                    required
                    aria-describedby="uidnote"
                />
                <button onClick={handleSubmit}>Register</button>
                <p>Already registered?</p>
                <ToastContainer />
            </section>
        </>
    );
}
```

假设您有多个组件需要访问与身份验证相关的数据，例如用户是否已登录。您可以使用此 `AuthContext` 向任何需要它的组件提供身份验证状态和更新功能，而不是通过组件层次结构以道具的形式传递这些数据。这减少了对道具钻探的需要，并使代码更清晰、更易于维护。

```javascript
import { createContext, useState } from "react";

const AuthContext = createContext({});

export const AuthProvider = ({ children }) => {
    const [auth, setAuth] = useState({});
    return (
        <AuthContext.Provider value={{ auth, setAuth }}>
            {children}
        </AuthContext.Provider>
    );
};

export default AuthContext;
```

例如，您可以在子组件中使用 `useContext` 钩子来访问 AuthContext.Provider 提供的 auth 状态和 setAuth 函数。

```javascript
import { useContext } from "react";
import AuthContext from "../context/AuthProvider";

const useAuth = () => {
    return useContext(AuthContext);
}

export default useAuth;
```

我们的 `index.jsx: AuthProvider` 将封装您的整个应用程序，使所有使用它的组件都能获得身份验证状态。

```javascript
import React from 'react';
import ReactDOM from 'react-dom/client';
import { BrowserRouter } from 'react-router-dom';
import './index.css';
import App from './App';
import {AuthProvider} from "./context/AuthProvider";

const root = ReactDOM.createRoot(
    document.getElementById('root')
);
root.render(
  <React.StrictMode>
      <AuthProvider>
          <BrowserRouter>
              <App />
          </BrowserRouter>
      </AuthProvider>
  </React.StrictMode>
);
```

我们需要能够解码 JWT 令牌，并读取用户分配的角色。

```javascript
import { useLocation, Navigate, Outlet } from "react-router-dom";
import useAuth from "../hooks/useAuth";
import jwtDecode from "jwt-decode";

function hasMatchingRole(allowedRoles, userRoles) {
    if (!allowedRoles || allowedRoles.length === 0) {
        return true;
    }

    for (let i = 0; i < allowedRoles.length; i++) {
        if (userRoles.indexOf(allowedRoles[i]) !== -1) {
            return true;
        }
    }

    return false;
}


const RequireAuth = ({ allowedRoles }) => {
    const { auth } = useAuth();
    const location = useLocation();

    let isAllowed = true;

    if (allowedRoles) {
        if (auth?.verifiedToken?.jwt) {
            const decodedToken = jwtDecode(auth.verifiedToken.jwt);
            isAllowed = hasMatchingRole(allowedRoles, decodedToken.role);
        } else {
            isAllowed = false;
        }
    }

    // 您想重定向到特定页面或显示错误消息吗？
    return (
        isAllowed
            ? <Outlet />
            : auth?.verifiedToken
                ? <Navigate to="/unauthorized" state={{ from: location }} replace />
                : <Navigate to="/login" state={{ from: location }} replace />
    );
}

export default RequireAuth;
```

在我们的 `app.jsx` 中或无论您路由到哪里，您都可以按如下方式定义您的路由，无论您是否需要特定角色来访问某个路由。

```javascript
import React, {Component} from 'react';
import {Route, Routes} from "react-router-dom";
import UserPage from "./pages/UserPage";
import AdminPage from "./pages/AdminPage";
import PublicPage from "./pages/PublicPage";
import Layout from "./components/Layout";
import LoginPage from "./pages/LoginPage";
import RegisterPage from "./pages/RegisterPage";
import RequireAuth from "./components/RequireAuth";
import {ROLE_ADMIN, ROLE_USER} from "./constants/Roles";
import UnauthorizedPage from "./pages/UnauthorizedPage";
import 'react-toastify/dist/ReactToastify.css';


class App extends Component {
    constructor(props) {
        super(props);
    }

    render() {
        return (
        <Layout>
            <Routes>
                <Route exact path="/" element={ <PublicPage/> } />
                <Route path="/register" element={ <RegisterPage/> } />
                <Route path="/login" element={ <LoginPage/> } />
                <Route path="unauthorized" element={<UnauthorizedPage />} />

                <Route element={<RequireAuth allowedRoles={[ROLE_USER]} />}>
                    <Route path="/user" element={ <UserPage/> } />
                </Route>

                <Route element={<RequireAuth allowedRoles={[ROLE_ADMIN]} />}>
                    <Route path="/admin" element={ <AdminPage/> } />
                </Route>
            </Routes>
        </Layout>
        );
    }
}

export default App;
```

## 参考 <a href="#references" id="references"></a>

* [示例应用程序](https://github.com/passwordless/passwordless-react-example)
