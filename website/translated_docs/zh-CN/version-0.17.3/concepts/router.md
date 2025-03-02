---
title: Router
description: Yew 的官方 Router
id: version-0.17.3-router
original_id: router
---

[在crates.io上查看 router ](https://crates.io/crates/yew-router)

Routers 在单页应用（SPA）中根据 URL 的不同显示不同的页面。当点击一个链接时，Router 在本地设置 URL 以指向应用程序中有效的路由，而不是默认请求一个不同的远程资源。然后 Router 检测到此更改后决定要渲染的内容。

## 核心元素

### `Route`

包含一个字符串，该字符串表示网址中域名之后的所有内容，还可以选择表示存储在 history api 中的状态。

### `RouteService`

与浏览器通信以获取和设置路由。

### `RouteAgent`

拥有一个 RouteService，并用于当路由改变时协调更新，无论更新是来自应用程序自身逻辑还是来自浏览器触发的事件。

### `Switch`

`Switch` trait 用于在该 trait 的实现者之间转换 `Route`。

### `Router`

Router 组件同 `RouterAgent` 进行通信，并将自动把它从 Agent 那里获得的 Routes 解析为 Switches，并通过 `render` 属性暴露该 Switch，该属性允许指定将生成的 Switch 转换为 `HTML` 的方式。

## 如何使用 Router

首先，你要创建一个表征你的应用程序所有状态的类型。请注意，虽然这通常是一个枚举，但也支持结构体，并且你可以在内部嵌套实现了 `Switch` trait 的其他项。

然后你应该为了你创建的类型派生 `Switch`。对于枚举，每一个成员都必须用 `#[at = "/some/route"]` 进行标注，如果你使用结构体，则标注必须出现在结构体声明之外。

```rust
#[derive(Switch)]
enum AppRoute {
  #[at = "/login"]
  Login,
  #[at = "/register"]
  Register,
  #[at = "/delete_account"]
  Delete,
  #[at = "/posts/{id}"]
  ViewPost(i32),
  #[at = "/posts/view"]
  ViewPosts,
  #[at = "/"]
  Home
}
```

:::caution 请注意，通过为派生宏生成实现`Switch`将按照从前到后的顺序为每个标注的路径进行匹配，所以当有路由符合多个`at`注解的路径时，将会永远只会匹配第一个符合的路径。例如，如果您定义了以下`Switch` ，将只会匹配到`AppRoute::Home` 路由。

```rust
#[derive(Switch)]
enum AppRoute {
  #[at = "/"]
  Home,
  #[at = "/login"]
  Login,
  #[at = "/register"]
  Register,
  #[at = "/delete_account"]
  Delete,
  #[at = "/posts/{id}"]
  ViewPost(i32),
  #[at = "/posts/view"]
  ViewPosts,
}
```

:::

你还可以在 `#[at = ""]` 标注中使用 `{}` 的变体来捕获片段。`{}` 表示捕获文本直到下一个分隔符（根据上下文可能是"/"，"?"，"&amp;" 或 "#"）。`{*}` 表示捕获文本直到后续字符匹配为止，如果不存在任何字符，则它将匹配任何内容。`{<number>}` 表示捕获文本直到遇到指定数目的分隔符为止（例如：`{2}` 将一直捕获文本直到遇到两个分隔符为止）。

对于带有命名字段的结构和枚举，必须在捕获组中指定字段的名称，例如： `{user_name}`或`{*:age}` 。

Switch trait 适用于比字符串更结构化的捕获组。你可以指定实现了 `Switch` trait 的任何类型。因此，你可以指定捕获组为 `usize`，并且如果 URL 的捕获部分无法转换为它，则该成员不会被匹配。
