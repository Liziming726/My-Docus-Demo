---
title: About Next.js Rendering / 关于Next.js渲染
date: 2022-09-21
tags: [文章分享]
cover: "https://imgbed.codingkelvin.fun/uPic/c9gF3x.png"
top_img: false
categories: [文章分享]
---

## Thought Why Next.js Rendering is so fast?

> 1. First, let's take a look at the rendering process of Next.js.

```markdown
About Link

该Link组件支持在同一个 Next.js 应用程序中的两个页面之间进行客户端导航。

客户端导航意味着使用 JavaScript进行页面转换，这比浏览器完成的默认导航要快。
```

这是您可以验证的简单方法：

- 使用浏览器的开发者工具将backgroundCSS 属性更改<html>为yellow。
- 单击链接可在两个页面之间来回切换。
- 您会看到黄色背景在页面转换之间持续存在。

这表明浏览器没有加载整个页面并且客户端导航正在工作。

![111](https://www.nextjs.cn/static/images/learn/navigate-between-pages/client-side.gif)

如果您使用<a href="…">而不是<Link href="…">这样做，则在链接点击时背景颜色将被清除，因为浏览器会进行完全刷新。

### 代码拆分和预取

Next.js 会自动进行代码拆分，因此每个页面只加载该页面所需的内容。这意味着在呈现主页时，最初不会提供其他页面的代码。

即使您添加数百个页面，这也可以确保主页快速加载。

仅加载您请求的页面的代码也意味着页面变得孤立。如果某个页面抛出错误，应用程序的其余部分仍然可以工作。

此外，在 Next.js 的生产版本中，每当`Link`组件出现在浏览器的视口中时，Next.js 都会自动在后台**预取**链接页面的代码。当您单击链接时，目标页面的代码已经在后台加载，页面转换几乎是即时的！

### 概括

> Next.js 通过代码拆分、客户端导航和预取（在生产中）自动优化您的应用程序以获得最佳性能。

> 2. Before we talk about data fetching, let’s talk about one of the most important concepts in Next.js: Pre-rendering.

### 预渲染

默认情况下，Next.js 预渲染每个页面。这意味着 Next.js会提前为每个页面生成 HTML，而不是全部由客户端 JavaScript 完成。预渲染可以带来更好的性能和SEO。

每个生成的 HTML 都与该页面所需的最少 JavaScript 代码相关联。当浏览器加载页面时，其 JavaScript 代码将运行并使页面完全交互。（这个过程称为水合作用。）

### 摘要：预渲染与无预渲染

这是一个快速的图形摘要：

![222](https://www.nextjs.cn/static/images/learn/data-fetching/pre-rendering.png)

![333](https://www.nextjs.cn/static/images/learn/data-fetching/no-pre-rendering.png)

### 接下来说说Next.js中预渲染的两种形式。

Next.js 有两种预渲染形式：静态生成 **(ssr)** 和服务器端渲染 **(ssg)**。不同之处在于它何时为页面生成 HTML。

- 静态生成是在**构建时**生成 HTML 的预渲染方法。然后在每个请求上重用预呈现的 HTML
- 服务器端渲染是在**每个请求上生成** HTML 的预渲染方法。

>**重要的是，Next.js 允许您选择用于每个页面的预渲染表单。您可以通过对大多数页面使用静态生成并为其他页面使用服务器端渲染来创建“混合”Next.js 应用程序。**

![3333](https://www.nextjs.cn/static/images/learn/data-fetching/per-page-basis.png)

### 有数据和无数据的静态生成

静态生成可以在有数据和没有数据的情况下完成。

到目前为止，我们创建的所有页面都不需要获取外部数据。当应用程序为生产而构建时，这些页面将自动静态生成。

![2222](https://www.nextjs.cn/static/images/learn/data-fetching/static-generation-without-data.png)

但是，对于某些页面，如果不先获取一些外部数据，您可能无法呈现 HTML。也许您需要在构建时访问文件系统、获取外部 API 或查询数据库。Next.js 开箱即用地支持这种情况——`带数据的静态生成。`

![44444](https://www.nextjs.cn/static/images/learn/data-fetching/static-generation-with-data.png)

### 使用数据进行静态生成getStaticProps

它是如何工作的？那么，在 Next.js 中，当你导出一个页面组件时，你也可以导出一个async名为getStaticProps. 如果你这样做，那么：

- getStaticProps在生产中的构建时运行，并且…
- 在函数内部，您可以获取外部数据并将其作为道具发送到页面。

```jsx
export default function Home(props) { ... }

export async function getStaticProps() {
  // Get external data from the file system, API, DB, etc.
  const data = ...

  // The value of the `props` key will be
  //  passed to the `Home` component
  return {
    props: ...
  }
}
```

> 本质上，getStaticProps它允许你告诉 Next.js：“嘿，这个页面有一些数据依赖——所以当你在构建时预渲染这个页面时，一定要先解决它们！”

---

> ## About the use of getStaticProps / 关于如何使用getStaticProps预渲染页面
>
> ![5555](https://www.nextjs.cn/static/images/learn/data-fetching/index-page.png)
>
>### 在请求时获取数据getServerSideProps(服务端渲染)
>
>要使用服务器端渲染，您需要导出getServerSideProps而不是getStaticProps从页面导出。
>
>  **因为getServerSideProps是在请求时调用的，所以它的参数（context）包含了请求特定的参数。** getServerSideProps仅当您需要预渲染必须在**请求时获取其数据的页面时才应使用。** 第一个字节的时间（TTFB）会比getStaticProps因为服务器必须计算每个请求的结果要慢，并且如果没有额外的配置， CDN无法缓存结果。

# 🍻 Finish ! Good night! 
---


## What is Rendering?
There is an unavoidable unit of work to convert the code you write in React into the HTML representation of your UI. This process is called **rendering.**

Rendering can take place on the server or on the client. It can happen either ahead of time at build time, or on every request at runtime.

With Next.js, three types of rendering methods are available: **Server-Side Rendering, Static Site Generation, and Client-Side Rendering.**

## Pre-Rendering
Server-Side Rendering and Static Site Generation are also referred to as **Pre-Rendering** because the fetching of external data and transformation of React components into HTML happens before the result is sent to the client.

## Client-Side Rendering vs. Pre-Rendering
In a standard React application, the browser receives an empty HTML shell from the server along with the JavaScript instructions to construct the UI. This is called **client-side rendering** because the initial rendering work happens on the user's device.

![tupian](https://nextjs.org/static/images/learn/foundations/client-side-rendering.png)

```markdown
Note:  You can opt to use client-side rendering for specific components in your Next.js application by choosing to fetch data with React’s useEffect() or a data fetching hook such as useSWR.
```

In contrast, Next.js **pre-renders** every page by default. Pre-rendering means the HTML is generated in advance, on a server, instead of having it all done by JavaScript on the user's device.

In practice, this means that for a fully client-side rendered app, the user will see a blank page while the rendering work is being done. Compared to a pre-rendered app, where the user will see the constructed HTML:

![tupain1](https://nextjs.org/static/images/learn/foundations/pre-rendering.png)

Let’s discuss the two types of pre-rendering:

## Server-Side Rendering

With server-side rendering, the HTML of the page is generated on a server for **each** request. The generated HTML, JSON data, and JavaScript instructions to make the page interactive are then sent to the client.

On the client, the HTML is used to show a fast non-interactive page, while React uses the JSON data and JavaScript instructions to make components interactive (for example, attaching event handlers to a button). This process is called **hydration.**

In Next.js, you can opt to server-side render pages by using `getServerSideProps`.

```markdown
Note: React 18 and Next 12 introduce an alpha version of React server components. Server 
components are completely rendered on the server and do not require client-side JavaScript to render. In addition, server components allow developers to keep some logic on the server and only send the result of that logic to the client. This reduces the bundle size sent to the client and improves client-side rendering performance. Learn more about React server components here.
```

## Static Site Generation

With Static Site Generation, the HTML is generated on the server, but unlike server-side rendering, there is no server at runtime. Instead, content is generated once, at build time, when the application is deployed, and the HTML is stored in a `CDN` and re-used for each request.

In Next.js, you can opt to statically generate pages by using `getStaticProps.`

The beauty of Next.js is that you can choose the most appropriate rendering method for your use case on a page-by-page basis, whether that's Static Site Generation, Server-side Rendering, or Client-Side Rendering. To learn more about which rendering method is right for your specific use case, see the data fetching docs.

=======
---
title: About Next.js Rendering / 关于Next.js渲染
date: 2022-09-21
tags: [文章分享]
cover: "https://imgbed.codingkelvin.fun/uPic/c9gF3x.png"
top_img: false
categories: [文章分享]
---

## Thought Why Next.js Rendering is so fast?

> 1. First, let's take a look at the rendering process of Next.js.

```markdown
About Link

该Link组件支持在同一个 Next.js 应用程序中的两个页面之间进行客户端导航。

客户端导航意味着使用 JavaScript进行页面转换，这比浏览器完成的默认导航要快。
```

这是您可以验证的简单方法：

- 使用浏览器的开发者工具将backgroundCSS 属性更改<html>为yellow。
- 单击链接可在两个页面之间来回切换。
- 您会看到黄色背景在页面转换之间持续存在。

这表明浏览器没有加载整个页面并且客户端导航正在工作。

![111](https://www.nextjs.cn/static/images/learn/navigate-between-pages/client-side.gif)

如果您使用<a href="…">而不是<Link href="…">这样做，则在链接点击时背景颜色将被清除，因为浏览器会进行完全刷新。

### 代码拆分和预取

Next.js 会自动进行代码拆分，因此每个页面只加载该页面所需的内容。这意味着在呈现主页时，最初不会提供其他页面的代码。

即使您添加数百个页面，这也可以确保主页快速加载。

仅加载您请求的页面的代码也意味着页面变得孤立。如果某个页面抛出错误，应用程序的其余部分仍然可以工作。

此外，在 Next.js 的生产版本中，每当`Link`组件出现在浏览器的视口中时，Next.js 都会自动在后台**预取**链接页面的代码。当您单击链接时，目标页面的代码已经在后台加载，页面转换几乎是即时的！

### 概括

> Next.js 通过代码拆分、客户端导航和预取（在生产中）自动优化您的应用程序以获得最佳性能。

> 2. Before we talk about data fetching, let’s talk about one of the most important concepts in Next.js: Pre-rendering.

### 预渲染

默认情况下，Next.js 预渲染每个页面。这意味着 Next.js会提前为每个页面生成 HTML，而不是全部由客户端 JavaScript 完成。预渲染可以带来更好的性能和SEO。

每个生成的 HTML 都与该页面所需的最少 JavaScript 代码相关联。当浏览器加载页面时，其 JavaScript 代码将运行并使页面完全交互。（这个过程称为水合作用。）

### 摘要：预渲染与无预渲染

这是一个快速的图形摘要：

![222](https://www.nextjs.cn/static/images/learn/data-fetching/pre-rendering.png)

![333](https://www.nextjs.cn/static/images/learn/data-fetching/no-pre-rendering.png)

### 接下来说说Next.js中预渲染的两种形式。

Next.js 有两种预渲染形式：静态生成 **(ssr)** 和服务器端渲染 **(ssg)**。不同之处在于它何时为页面生成 HTML。

- 静态生成是在**构建时**生成 HTML 的预渲染方法。然后在每个请求上重用预呈现的 HTML
- 服务器端渲染是在**每个请求上生成** HTML 的预渲染方法。

>**重要的是，Next.js 允许您选择用于每个页面的预渲染表单。您可以通过对大多数页面使用静态生成并为其他页面使用服务器端渲染来创建“混合”Next.js 应用程序。**

![3333](https://www.nextjs.cn/static/images/learn/data-fetching/per-page-basis.png)

### 有数据和无数据的静态生成

静态生成可以在有数据和没有数据的情况下完成。

到目前为止，我们创建的所有页面都不需要获取外部数据。当应用程序为生产而构建时，这些页面将自动静态生成。

![2222](https://www.nextjs.cn/static/images/learn/data-fetching/static-generation-without-data.png)

但是，对于某些页面，如果不先获取一些外部数据，您可能无法呈现 HTML。也许您需要在构建时访问文件系统、获取外部 API 或查询数据库。Next.js 开箱即用地支持这种情况——`带数据的静态生成。`

![44444](https://www.nextjs.cn/static/images/learn/data-fetching/static-generation-with-data.png)

### 使用数据进行静态生成getStaticProps

它是如何工作的？那么，在 Next.js 中，当你导出一个页面组件时，你也可以导出一个async名为getStaticProps. 如果你这样做，那么：

- getStaticProps在生产中的构建时运行，并且…
- 在函数内部，您可以获取外部数据并将其作为道具发送到页面。

```jsx
export default function Home(props) { ... }

export async function getStaticProps() {
  // Get external data from the file system, API, DB, etc.
  const data = ...

  // The value of the `props` key will be
  //  passed to the `Home` component
  return {
    props: ...
  }
}
```

> 本质上，getStaticProps它允许你告诉 Next.js：“嘿，这个页面有一些数据依赖——所以当你在构建时预渲染这个页面时，一定要先解决它们！”

---

> ## About the use of getStaticProps / 关于如何使用getStaticProps预渲染页面
>
> ![5555](https://www.nextjs.cn/static/images/learn/data-fetching/index-page.png)
>
>### 在请求时获取数据getServerSideProps(服务端渲染)
>
>要使用服务器端渲染，您需要导出getServerSideProps而不是getStaticProps从页面导出。
>
>  **因为getServerSideProps是在请求时调用的，所以它的参数（context）包含了请求特定的参数。** getServerSideProps仅当您需要预渲染必须在**请求时获取其数据的页面时才应使用。** 第一个字节的时间（TTFB）会比getStaticProps因为服务器必须计算每个请求的结果要慢，并且如果没有额外的配置， CDN无法缓存结果。

# 🍻 Finish ! Good night! 
---


## What is Rendering?
There is an unavoidable unit of work to convert the code you write in React into the HTML representation of your UI. This process is called **rendering.**

Rendering can take place on the server or on the client. It can happen either ahead of time at build time, or on every request at runtime.

With Next.js, three types of rendering methods are available: **Server-Side Rendering, Static Site Generation, and Client-Side Rendering.**

## Pre-Rendering
Server-Side Rendering and Static Site Generation are also referred to as **Pre-Rendering** because the fetching of external data and transformation of React components into HTML happens before the result is sent to the client.

## Client-Side Rendering vs. Pre-Rendering
In a standard React application, the browser receives an empty HTML shell from the server along with the JavaScript instructions to construct the UI. This is called **client-side rendering** because the initial rendering work happens on the user's device.

![tupian](https://nextjs.org/static/images/learn/foundations/client-side-rendering.png)

```markdown
Note:  You can opt to use client-side rendering for specific components in your Next.js application by choosing to fetch data with React’s useEffect() or a data fetching hook such as useSWR.
```

In contrast, Next.js **pre-renders** every page by default. Pre-rendering means the HTML is generated in advance, on a server, instead of having it all done by JavaScript on the user's device.

In practice, this means that for a fully client-side rendered app, the user will see a blank page while the rendering work is being done. Compared to a pre-rendered app, where the user will see the constructed HTML:

![tupain1](https://nextjs.org/static/images/learn/foundations/pre-rendering.png)

Let’s discuss the two types of pre-rendering:

## Server-Side Rendering

With server-side rendering, the HTML of the page is generated on a server for **each** request. The generated HTML, JSON data, and JavaScript instructions to make the page interactive are then sent to the client.

On the client, the HTML is used to show a fast non-interactive page, while React uses the JSON data and JavaScript instructions to make components interactive (for example, attaching event handlers to a button). This process is called **hydration.**

In Next.js, you can opt to server-side render pages by using `getServerSideProps`.

```markdown
Note: React 18 and Next 12 introduce an alpha version of React server components. Server 
components are completely rendered on the server and do not require client-side JavaScript to render. In addition, server components allow developers to keep some logic on the server and only send the result of that logic to the client. This reduces the bundle size sent to the client and improves client-side rendering performance. Learn more about React server components here.
```

## Static Site Generation

With Static Site Generation, the HTML is generated on the server, but unlike server-side rendering, there is no server at runtime. Instead, content is generated once, at build time, when the application is deployed, and the HTML is stored in a `CDN` and re-used for each request.

In Next.js, you can opt to statically generate pages by using `getStaticProps.`

The beauty of Next.js is that you can choose the most appropriate rendering method for your use case on a page-by-page basis, whether that's Static Site Generation, Server-side Rendering, or Client-Side Rendering. To learn more about which rendering method is right for your specific use case, see the data fetching docs.


In the next section, we’ll discuss where your code can be stored or run after it’s deployed.