---
title: Reading Tao of React
date: 2023-02-20
tags: [文章分享]
cover: "https://imgbed.codingkelvin.fun/uPic/c9gF3x.png"
top_img: false
categories: [文章分享]
---

# Tao of React - Software Design, Architecture & Best Practices
---

I’ve been working with React since 2016 and still there isn’t a single best practice when it comes to application structure and design.
> 2016年以来，我一直在使用React，但是在应用程序结构和设计方面，仍然没有一个最佳实践。

While there are best practices on the micro level, most teams build their own “thing” when it comes to architecture.
> 尽管在微观层面上有最佳实践，但是在架构方面，大多数团队都会构建自己的“事物”。

Of course, there isn’t a universal best practice that can be applied to all businesses and applications. But there are some general rules that we can follow to build a productive codebase.
> 当然，没有一个通用的最佳实践可以应用于所有的业务和应用程序。但是我们可以遵循一些一般规则来构建一个高效的代码库。

The purpose of a software’s architecture and design is to keep it productive and flexible. Developers need to work on it effectively and modify it without rewriting its core.
> 软件架构和设计的目的是使其高效和灵活。开发人员需要有效地对其进行工作，并在不重写其核心的情况下对其进行修改。

This article is a collection of principles and rules that have proven to be effective for me and the teams I’ve worked with.
>这篇文章收集了一些原则和规则，这些原则和规则对我和我合作过的团队都很有效。

I outline good practices about components, application structure, testing, styling, state management and data fetching. Some of the examples are oversimplified so we can focus on the principle not on the implementation.
> 我概述了关于组件、应用程序结构、测试、样式、状态管理和数据获取的良好实践。一些例子过于简单，所以我们可以把重点放在原则上，而不是实现上。

**Take everything here as an opinion**, not an absolute. There’s more than one way to build software.

---
## Components
### Favor Functional Components

Favor functional components - they have a simpler syntax. No lifecycle methods, constructors or boilerplate. You can express the same logic with less characters without losing readability.
> 倾向于使用函数组件 - 它们具有更简单的语法。没有生命周期方法、构造函数或样板。你可以用更少的字符表达相同的逻辑，而不会失去可读性。

Unless you need an error boundary they should be your go-to approach. The mental model you need to keep in your head is a lot smaller.
> 除非你需要一个错误边界，否则它们应该是你的首选方法。你需要在脑海中保持的心理模型要小得多。

```jsx
// 👎 Class components are verbose
class Counter extends React.Component {
  state = {
    counter: 0,
  }
  constructor(props) {
    super(props)
    this.handleClick = this.handleClick.bind(this)
  }
  handleClick() {
    this.setState({ counter: this.state.counter + 1 })
  }
  render() {
    return (
      <div>
        <p>counter: {this.state.counter}</p>
        <button onClick={this.handleClick}>Increment</button>
      </div>
    )
  }
}
// 👍 Functional components are easier to read and maintain
function Counter() {
  const [counter, setCounter] = useState(0)
  handleClick = () => setCounter(counter + 1)
  return (
    <div>
      <p>counter: {counter}</p>
      <button onClick={handleClick}>Increment</button>
    </div>
  )
}
```

### Write Consistent Components

Stick to the same style for your components. Put helper functions in the same place, export the same way and follow the same naming patterns.
> 保持组件的风格一致。将辅助函数放在同一位置，以相同的方式导出并遵循相同的命名模式。

There isn’t a real benefit of one approach over the other
> 一种方法比另一种方法没有真正的好处

No matter if you’re exporting at the bottom of the file or directly in the definition of the component, pick one and stick to it.
> 无论你是在文件底部导出还是直接在组件的定义中导出，都要选择一个并坚持使用它。

### Name Components 
Always name your components. It helps when you’re reading an error stack trace and using the React Dev Tools.
> 总是给你的组件命名。当你阅读错误堆栈跟踪和使用React Dev Tools时，它会有所帮助。

It’s also easier to find where you are when developing if the component’s name is inside the file.
> 如果组件的名称在文件中，那么在开发时也更容易找到你所在的位置。

```jsx
// 👎 Avoid this
export default () => <form>...</form>

// 👍 Name your functions
export default function Form() {
  return <form>...</form>
}
```

### Organize Helper Functions

Helper functions that don’t need to hold a closure over the components should be moved outside. The ideal place is before the component definition so the file can be readable from top to bottom.
> 不需要保持组件闭包的辅助函数应该被移出。理想的位置是在组件定义之前，这样文件就可以从上到下进行阅读。

That reduces the noise in the component and leaves inside only those that need to be there.
> 这减少了组件中的噪音，并且只在需要的地方留下了那些需要的东西。

```jsx
// 👎 Avoid nesting functions which don't need to hold a closure.
function Component({ date }) {
  function parseDate(rawDate) {
    ...
  }
  return <div>Date is {parseDate(date)}</div>
}
// 👍 Place the helper functions after the component
function Component({ date }) {
  return <div>Date is {parseDate(date)}</div>
}
function parseDate(date) {
  ...
}
```

You want to keep the least amount of helper functions inside the definition. Move as many as possible outside and pass the values from state as arguments.
> 你想保持定义中的辅助函数的最少数量。尽可能多地将其移出，并将状态中的值作为参数传递。

Composing your logic out of pure functions that rely only on input makes it easier to track bugs and extend.
> 将你的逻辑组合成仅依赖于输入的纯函数，可以更容易地跟踪错误并扩展。

```jsx
// 👎 Helper functions shouldn't read from the component's state
export default function Component() {
  const [value, setValue] = useState('')
  function isValid() {
    // ...
  }
  return (
    <>
      <input
        value={value}
        onChange={e => setValue(e.target.value)}
        onBlur={validateInput}
      />
      <button
        onClick={() => {
          if (isValid) {
            // ...
          }
        }}
      >
        Submit
      </button>
    </>
  )
}
// 👍 Extract them and pass only the values they need
export default function Component() {
  const [value, setValue] = useState('')
  return (
    <>
      <input
        value={value}
        onChange={e => setValue(e.target.value)}
        onBlur={validateInput}
      />
      <button
        onClick={() => {
          if (isValid(value)) {
            // ...
          }
        }}
      >
        Submit
      </button>
    </>
  )
}
function isValid(value) {
  // ...
}
```

### Don't Hardcode Repetitive Markup

Don’t hardcode markup for navigation, filters or lists. Use a configuration object and loop through the items instead.
> 不要为导航、过滤器或列表硬编码标记。使用配置对象并循环遍历项目。

This means you only have to change the markup and items in a single place.
> 这意味着你只需要在一个地方更改标记和项目。

```jsx
// 👎 Hardcoded markup is harder to manage.
function Filters({ onFilterClick }) {
  return (
    <>
      <p>Book Genres</p>
      <ul>
        <li>
          <div onClick={() => onFilterClick('fiction')}>Fiction</div>
        </li>
        <li>
          <div onClick={() => onFilterClick('classics')}>
            Classics
          </div>
        </li>
        <li>
          <div onClick={() => onFilterClick('fantasy')}>Fantasy</div>
        </li>
        <li>
          <div onClick={() => onFilterClick('romance')}>Romance</div>
        </li>
      </ul>
    </>
  )
}

// 👍 Use loops and configuration objects
const GENRES = [
  {
    identifier: 'fiction',
    name: Fiction,
  },
  {
    identifier: 'classics',
    name: Classics,
  },
  {
    identifier: 'fantasy',
    name: Fantasy,
  },
  {
    identifier: 'romance',
    name: Romance,
  },
]

function Filters({ onFilterClick }) {
  return (
    <>
      <p>Book Genres</p>
      <ul>
        {GENRES.map((genre) => (
          <li>
            <div onClick={() => onFilterClick(genre.identifier)}>
              {genre.name}
            </div>
          </li>
        ))}
      </ul>
    </>
  )
}
```

### Manage Component Size

A React component is just a function that gets props and returns markup. They adhere to the same software design principles.
> 一个React组件只是一个函数，它获取props并返回标记。它们遵循相同的软件设计原则。

If a function is doing too many things, extract some of the logic and call another function. It’s the same with components - if you have too much functionality, split it in smaller components and call them instead.
> 如果一个函数做了太多的事情，就提取一些逻辑并调用另一个函数。这对于组件来说也是一样的——如果你有太多的功能，就将其拆分为较小的组件并调用它们。

If a part of the markup is complex, requires loops and conditionals - extract it.
> 如果标记的一部分复杂，需要循环和条件——提取它。

Rely on props and callbacks for communication and data. Lines of code are not an objective measure. Think about responsibilities and abstractions instead.
> 依靠props和回调进行通信和数据。代码行数不是一个客观的度量。相反，考虑职责和抽象。

### Write Comments in JSX
When something needs more clarity open a code block and provide the additional information just like you would in a regular function. The markup is a part of the logic so when you feel that something needs more clarity - provide it.
> 当需要更多的清晰度时，打开一个代码块，并像在常规函数中一样提供额外的信息。标记是逻辑的一部分，所以当你觉得有些东西需要更多的清晰度时，就提供它。

Business logic is always coupled to the markup, at least a little. So we should provide any context about the domain that may not be obvious.
> 业务逻辑总是与标记相结合，至少有一点。所以我们应该提供任何关于可能不明显的域的上下文。

```jsx
function Component(props) {
  return (
    <>
      {/* Subscribers should not see any ads. */}
      {user.subscribed ? null : <Advert />}
    </>
  )
}
```

### Use Error Boundaries

An error in one component shouldn’t bring down the entire UI. There are rare cases in which we want to take down the whole page or redirect if a critical error happens. Most of the times we’d be fine if we just hide a specific element from the screen.
> 组件中的一个错误不应该使整个UI崩溃。如果发生关键错误，我们希望将整个页面关闭或重定向的情况很少。大多数情况下，如果我们只是从屏幕上隐藏一个特定的元素，我们就会很好。

In a function that deals with data you may have multiple try/catch statements. Put error boundaries to use not just on the top level. Wrap elements on the screen that can exist separately to avoid cascading failures.
> 在处理数据的函数中，你可能有多个try / catch语句。不仅在顶级使用错误边界。包装屏幕上可以单独存在的元素，以避免级联故障。

```jsx
function Component() {
  return (
    <Layout>
      <ErrorBoundary>
        <CardWidget />
      </ErrorBoundary>

      <ErrorBoundary>
        <FiltersWidget />
      </ErrorBoundary>

      <div>
        <ErrorBoundary>
          <ProductList />
        </ErrorBoundary>
      </div>
    </Layout>
  )
}
```

### Destructure Props
Most React components are just functions. They get props and return markup. In a normal function you use the arguments it is passed directly, so it makes sense to apply the same principle here. No need to repeat everywhere.props
> 大多数React组件只是函数。它们获取props并返回标记。在一个正常的函数中，你直接使用它传递的参数，所以在这里应用相同的原则是有意义的。没有必要在每个地方重复props。

A reason not to destructure the props would be to distinguish what is external and what is internal state. But in a regular function there is no distinction between arguments and variables. Don’t create unnecessary patterns.
> 不要解构props的原因是区分外部和内部状态。但在常规函数中，参数和变量之间没有区别。不要创建不必要的模式。

```jsx
// 👎 Don't repeat props everywhere in your component
function Input(props) {
  return <input value={props.value} onChange={props.onChange} />
}
// 👍 Destructure and use the values directly
function Component({ value, onChange }) {
  const [state, setState] = useState('')
  return <div>...</div>
}

```

### Manage the Number of Props
The question of how many props should a component receive is a subjective one. The number of props that a component has is correlated to how much it’s doing. The more props you pass to it the more responsibilities it has.
> 组件应该接收多少props是一个主观的问题。组件具有的props数量与它正在做的事情有关。你传递给它的props越多，它就有越多的责任。

A high number of props is a signal that a component is doing too much.
> props的数量高是组件做太多事情的信号。

If I go above 5 props I consider whether this component should be split. In some cases, it may just need a lot of data. An input field, for example, may have a lot of props. In others it’s a sign that something needs to be extracted.
> 如果我超过5个props，我会考虑是否应该拆分这个组件。在某些情况下，它可能只需要很多数据。例如，输入字段可能有很多props。在其他情况下，这是需要提取某些东西的信号。

Note: The more props a component takes, the more reasons to rerender.
> 注意：组件接收的props越多，就越有理由重新渲染。

### Pass Objects Instead of Primitives

One way to limit the amount of props is to pass an object instead of primitive values. Rather than passing down the user name, email and settings one by one you can group them together. This also reduces the changes that need to be done if the user gets an extra field for example.
> 限制props数量的一种方法是传递一个对象而不是原始值。你可以将用户名，电子邮件和设置分别传递下来，而不是一次传递一个。这也减少了如果用户获得额外的字段，需要进行的更改。

Using TypeScript makes this even easier.
> 使用TypeScript使这更容易。

```jsx
// 👎 Don't pass values on by one if they're related
<UserProfile
  bio={user.bio}
  name={user.name}
  email={user.email}
  subscription={user.subscription}
/>

// 👍 Use an object that holds all of them instead
<UserProfile user={user} />
```

### Conditional Rendering
In some situations using short circuit operators for conditional rendering may backfire and you may end up with an unwanted in your UI. To avoid this default to using ternary operators. The only caveat is that they’re more verbose.0
> 在某些情况下，使用短路运算符进行条件渲染可能会反作用，并且你可能会在UI中得到不需要的结果。为了避免这种情况，请默认使用三元运算符。唯一的警告是它们更冗长。

The short-circuit operator reduces the amount of code which is always nice. Ternaries are more verbose but there is no chance to get it wrong. Plus, adding the alternative condition is less of a change.
> 短路运算符减少了代码量，这总是很好的。三元运算符更冗长，但没有机会错过。此外，添加替代条件是更少的变化。

```jsx
// 👎 Try to avoid short-circuit operators
function Component() {
  const count = 0
  return <div>{count && <h1>Messages: {count}</h1>}</div>
}
// 👍 Use a ternary instead
function Component() {
  const count = 0
  return <div>{count ? <h1>Messages: {count}</h1> : null}</div>
}
```

### Avoid Nested Ternary Operators

Ternary operators become hard to read after the first level. Even if they seem to save space at the time, it’s better to be explicit and obvious in your intentions.
> 三元运算符在第一级后变得难以阅读。即使它们似乎在那时节省了空间，最好在你的意图中明确和明显。

```jsx
// 👎 Nested ternaries are hard to read in JSX
isSubscribed ? (
  <ArticleRecommendations />
) : isRegistered ? (
  <SubscribeCallToAction />
) : (
  <RegisterCallToAction />
)
// 👍 Place them inside a component on their own
function CallToActionWidget({ subscribed, registered }) {
  if (subscribed) {
    return <ArticleRecommendations />
  }
  if (registered) {
    return <SubscribeCallToAction />
  }
  return <RegisterCallToAction />
}
function Component() {
  return (
    <CallToActionWidget
      subscribed={subscribed}
      registered={registered}
    />
  )
}
```

### Move Lists in a Separate Component
Looping through a list of items is a common occurrence, usually done with the function. However, in a component that has a lot of markup, the extra indentation and the syntax of don’t help with readability.mapmap
> 遍历项目列表是常见的情况，通常使用函数完成。但是，在具有大量标记的组件中，额外的缩进和语法不会有助于可读性。

When you need to map over elements, extract them in their own listing component, even if the markup isn’t much. The parent component doesn’t need to know about the details, only that it’s displaying a list.
> 当你需要映射元素时，将它们提取到自己的列表组件中，即使标记不多。父组件不需要知道细节，只需要知道它正在显示一个列表。

Only keep a loop in the markup if the component’s main responsibility is to display it. Try to keep a single mapping per component but if the markup is long or complicated, extract the list either way.
> 只要组件的主要职责是显示它，就只在标记中保留循环。尝试在每个组件中保留一个映射，但如果标记很长或复杂，无论如何都要提取列表。

```jsx
// 👎 Don't write loops together with the rest of the markup
function Component({ topic, page, articles, onNextPage }) {
  return (
    <div>
      <h1>{topic}</h1>
      {articles.map((article) => (
        <div>
          <h3>{article.title}</h3>
          <p>{article.teaser}</p>
          <img src={article.image} />
        </div>
      ))}
      <div>You are on page {page}</div>
      <button onClick={onNextPage}>Next</button>
    </div>
  )
}

// 👍 Extract the list in its own component
function Component({ topic, page, articles, onNextPage }) {
  return (
    <div>
      <h1>{topic}</h1>
      <ArticlesList articles={articles} />
      <div>You are on page {page}</div>
      <button onClick={onNextPage}>Next</button>
    </div>
  )
}
```

### Assign Default Props When Destructuring

One way to specify default prop values is to attach a property to the component. This means that the component function and the values for its arguments are not going to sit together.defaultProps
> 指定默认prop值的一种方法是将属性附加到组件。这意味着组件函数和其参数的值不会一起坐下来。

Prefer assigning default values directly when you’re destructuring the props. It makes it easier to read the code from top to bottom without jumping and keeps the definitions and values together.
> 在解构props时，首选直接分配默认值。这使得从上到下阅读代码更容易，而不必跳转，并且可以将定义和值放在一起。

```jsx
// 👎 Don't define the default props outside of the function
function Component({ title, tags, subscribed }) {
  return <div>...</div>
}

Component.defaultProps = {
  title: '',
  tags: [],
  subscribed: false,
}

// 👍 Place them in the arguments list
function Component({ title = '', tags = [], subscribed = false }) {
  return <div>...</div>
}
```
# Next
https://alexkondov.com/tao-of-react/#avoid-nested-render-functions