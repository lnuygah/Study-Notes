# 📚 React 组件类型全览笔记

## 一、📄 页面组件（Page Components）

**用于组成完整页面，通常与路由关联**

* 🔹 命名示例：`LoginPage`, `HomePage`, `UserCenterPage`
* 🔹 位置常见于：`/src/pages/` 或 `/src/routes/`
* 🔹 特点：

  * 拥有完整页面布局
  * 发起 API 请求（调用 `services`）
  * 组合多个 UI 子组件
  * 通过 `react-router` 显示在某个 URL 路径下

```tsx
// src/pages/LoginPage.tsx
const LoginPage = () => {
  return (
    <div>
      <LoginForm />
    </div>
  );
};
```

---

## 二、🧩 展示型组件 / UI 组件（Presentational / UI Components）

**只负责展示，没有业务逻辑**

* 🔹 命名示例：`Button`, `Card`, `Input`, `Avatar`
* 🔹 位置常见于：`/src/components/`
* 🔹 特点：

  * 接收 `props`，只做 UI 渲染
  * 无状态或仅有局部状态
  * 可复用性高、粒度小

```tsx
// src/components/Button.tsx
const Button = ({ children, onClick }: { children: React.ReactNode; onClick: () => void }) => {
  return <button onClick={onClick}>{children}</button>;
};
```

---

## 三、⚙️ 容器型组件 / 智能组件（Container / Smart Components）

**处理数据、状态、逻辑，不负责样式**

* 🔹 命名示例：`UserListContainer`, `TodoManager`, `LoginFormContainer`
* 🔹 位置：通常也在 `components/` 或 `containers/`
* 🔹 特点：

  * 管理状态（`useState` / `useEffect`）
  * 调用服务端接口
  * 包含展示型组件

```tsx
const LoginFormContainer = () => {
  const [username, setUsername] = useState("");
  const [password, setPassword] = useState("");

  const handleSubmit = async () => {
    await PostLogin(username, password);
  };

  return <LoginForm username={username} password={password} onSubmit={handleSubmit} />;
};
```

---

## 四、♻️ 高阶组件（Higher-Order Components, HOC）

**一个函数，接收一个组件，返回一个增强的新组件**

* 🔹 命名习惯：`withXxx`（如 `withAuth`, `withTheme`）
* 🔹 用法：用于添加权限控制、注入 props、日志记录等功能

```tsx
function withAuth<P>(Component: React.ComponentType<P>) {
  return (props: P) => {
    if (!isLoggedIn()) return <Redirect to="/login" />;
    return <Component {...props} />;
  };
}
```

---

## 五、📦 自定义 Hook 组件（Custom Hooks）

**复用逻辑的函数组件，不返回 JSX**

* 🔹 命名习惯：必须以 `use` 开头，例如：`useUserInfo`, `useScroll`, `useForm`
* 🔹 特点：

  * 复用状态逻辑或副作用逻辑
  * 不能直接用于渲染 UI
  * 可组合调用多个 hook

```tsx
function useUserInfo() {
  const [user, setUser] = useState<User | null>(null);
  useEffect(() => {
    fetchUser().then(setUser);
  }, []);
  return user;
}
```

---

## 六、💬 纯函数组件（Functional / Stateless Components）

**最简单的组件写法**

* 🔹 特点：

  * 没有副作用
  * 没有状态
  * 没有 hooks，仅仅根据 props 返回 UI

```tsx
const Greeting = ({ name }: { name: string }) => <div>Hello, {name}</div>;
```

---

## 七、🔀 动态组件（Lazy Loaded / Suspense Components）

**按需加载组件以优化性能**

* 🔹 使用场景：

  * 页面或模块体积大、首屏不需要加载
  * 利用 `React.lazy` 和 `Suspense`

```tsx
const AboutPage = React.lazy(() => import("./AboutPage"));

<Suspense fallback={<Loading />}>
  <AboutPage />
</Suspense>
```

---

# ✅ 总结一张表

| 类型       | 用途             | 是否渲染 UI | 是否含状态 | 是否调用接口 |
| -------- | -------------- | ------- | ----- | ------ |
| 页面组件     | 构建完整页面         | ✅       | ✅     | ✅      |
| 展示型组件    | 渲染 UI          | ✅       | ❌     | ❌      |
| 容器型组件    | 管理状态与逻辑        | ✅       | ✅     | ✅      |
| 高阶组件     | 增强其他组件         | ❌       | 可选    | 可选     |
| 自定义 Hook | 抽离复用逻辑         | ❌       | ✅     | ✅      |
| 纯函数组件    | 接收 props 输出 UI | ✅       | ❌     | ❌      |
| 动态组件     | 延迟加载           | ✅       | 可选    | 可选     |
