# react-connect-context-map

> This is a fork of [https://github.com/contiamo/react-connect-context](react-connect-context).

[![Edit Creating a configurable component library with React Context](https://codesandbox.io/static/img/play-codesandbox.svg)](https://codesandbox.io/s/6wpk3lo9lr)

This module exports a function factory that produces an HOC to wrap a component and provide the value of the context as props to the component, much like the [react-redux](https://react-redux.js.org/) API. The benefits of this approach versus the traditional Consumer:

* All values are passed as props and can be read in your component's lifecycle methods
* You can trivially export a "pure" version of the component as well as a connected version
* Unlike a Redux container, the connected component can be rendered without being nested in a Provider component
* The factory produces an HOC which can be applied to multiple components
* You can select slices of the context value and map them to props that fit the component API
* You can reference the props passed into the connected component when mapping the context value to props
* You can customize the merging behavior to translate all props fed to the wrapped component

See this Medium post: [Creating a configurable component library with React Context](https://medium.com/@mmiller42/creating-a-configurable-component-library-with-react-context-a940b5965a9b) to learn how to leverage this module to create a UI component library with shared configuration for the project using it.

## Getting Started

1. `yarn add react-connect-context-map`
1. At the top of your file, `import { connectContext } from "react-connect-context-map"`
1. Wrap your component in the function as so: `connectContext(Context.Consumer)(MyComponent)`

### Examples

#### Introductory example

```tsx
import React from "react"
import { render } from "react-dom"
import { connectContext } from "react-connect-context-map"

// The shape of the context value
interface ContextType {
  color: string
}

// The shape of the props passed to your component
interface MergedPropsType extends ContextType {
  children: React.ReactNode
  myProp: string
}

// Create a new React Context instance
const Context = React.createContext<ContextType>({ color: "red" })

// Create an HOC for connecting your component to context
const createContainer = connectContext<ContextType>(Context.Consumer)

// The connected component will receive the props passed in as well as values
// from the context object by default
const Content: React.SFC<MergedPropsType> = ({ children, color, myProp }) => (
  <div>
    <p>{children}</p>
    <div>
      I have looked into context and I've seen the color is:
      <span style={{ color }}>
        <strong>{color}</strong>
      </span>! I also get my own props like <strong>{myProp}</strong>!
    </div>
  </div>
)

// Call the HOC on the component to connect it to context
const ConnectedContent = createContainer<MergedPropsType>(Content)

const App: React.SFC = () => (
  <div className="demo">
    <ConnectedContent myProp="THIS IS MY PROP, HI!">
      Hello! I've written this component so that Magical Context-based text
      appears after children!
    </ConnectedContent>
  </div>
)

// Render the connected components in the provider
render(
  <Context.Provider value={{ color: "red" }}>
    <App />
  </Context.Provider>,
  document.querySelector("#root")
)
```

#### Mapping context properties to props

This example shows how a context value can be mapped to a different prop name or value.

```tsx
import React from "react"
import { render } from "react-dom"
import { connectContext } from "react-connect-context-map"

interface ContextType {
  color: string
}

// The shape of the props that have been derived from context
interface ContextPropsType {
  isRed: boolean
}

// The shape of the props that are not derived from context and may be
// passed to the connected component
interface OwnPropsType {
  myProp: string
}

// The shape of the props passed to the connected component
interface MergedPropsType extends ContextPropsType, OwnPropsType {}

const Context = React.createContext<ContextType>({ color: "red" })

const Content: React.SFC<MergedPropsType> = ({ isRed, myProp }) => (
  <p>This pen {isRed ? "is" : "is not"} red. My prop is {myProp}.</p>
)

// ContextPropsType can be used when the result of mapContextToProps does
// not match the shape of ContextType
const createContainer = connectContext<ContextType, ContextPropsType>(
  Context.Consumer,
  context => ({
    isRed: context.color === "red"
  })
)

// OwnPropsType can be used to define the props on the wrapped component
// when additional props not inferred from context are provided
const ConnectedContent = createContainer<
  MergedPropsType,
  OwnPropsType
>(Content)

render(
  <Context.Provider value={{ color: "red" }}>
    <App />
  </Context.Provider>,
  document.querySelector("#root")
)
```

#### Customizing prop merging

This example shows how you can customize the merge behavior. Here we even map a non-context prop to another key.

```tsx
import React from "react"
import { render } from "react-dom"
import { connectContext } from "react-connect-context-map"

interface ContextType {
  color: string
}

interface OwnPropsType {
  theProp: string
}

interface MergedPropsType extends ContextType, OwnPropsType {}

const Context = React.createContext<ContextType>({ color: "red" })

const Content: React.SFC<MergedPropsType> = ({ color, theProp }) => (
  <p>This pen {isRed ? "is" : "is not"} red. My prop is {theProp}.</p>
)

const createContainer = connectContext<ContextType>(
  Context.Consumer,
  null,
  // Merge context properties into props and translate myProp to theProp
  (context, props) => ({
    ...context,
    theProp: props.myProp
  })
)

const ConnectedContent = createContainer<
  MergedPropsType,
  OwnPropsType
>(Content)

render(
  <Context.Provider value={{ color: "red" }}>
    <App />
  </Context.Provider>,
  document.querySelector("#root")
)
```

#### Using non-object context values with `mapContextToProps`

This example shows how you can allow for non-object context values if you provide a custom `mapContextToProps` function.

```tsx
import React from "react"
import { render } from "react-dom"
import { connectContext } from "react-connect-context-map"

// Context value is not an object
type ContextType = string

interface ContextPropsType {
  color: string
}

interface OwnPropsType {
  myProp: string
}

interface MergedPropsType extends ContextPropsType, OwnPropsType {}

const Context = React.createContext<ContextType>({ color: "red" })

const Content: React.SFC<MergedPropsType> = ({ color, theProp }) => (
  <p>This pen {isRed ? "is" : "is not"} red. My prop is {theProp}.</p>
)

const createContainer = connectContext<
  ContextType,
  ContextPropsType
>(
  Context.Consumer,
  // Context value is mapped to an object that conforms to
  // ContextPropsType
  context => ({ color: context })
)

const ConnectedContent = createContainer<
  MergedPropsType,
  OwnPropsType
>(Content)

render(
  <Context.Provider value="red">
    <App />
  </Context.Provider>,
  document.querySelector("#root")
)
```

## API

### connectContext

Factory function that creates a container component HOC to consume context `Context` and map values to match `ContextProps`.

```tsx
connectContext<Context, ContextProps extends Object = Context>(
  // The React Consumer component.
  ContextConsumer: React.Consumer<Context>,

  // A function that maps the consumed context value to props to pass to
  // the component. The default function requires the context value to be
  // an object and maps its properties to component props.
  mapContextToProps: MapContextToProps<Context, ContextProps, OwnProps>
    = context => context,

  // A function that merges the props that have been mapped from context
  // values with the props passed to the connected component. The default
  // function merges context props with the passed props, with the latter
  // overwriting the former.
  mergeProps: MergeProps<ContextProps, OwnProps, MergedProps> =
    (contextProps, ownProps) => ({ ...contextProps, ...ownProps })
): createContainer // HOC to connect a component.
```

### createContainer

An HOC that returns a connected component that accepts props `OwnProps` and derives `ContextProps` to merge into `MergedProps`. It returns a component that accepts `OwnProps` as props and renders the given component with `MergedProps`.

```tsx
createContainer<MergedProps extends ContextProps, OwnProps = Partial<MergedProps>>(
  // The component to connect.
  Component: React.SFC<MergedProps>
): React.SFC<OwnProps> // The container component.
```

### MapContextToProps

A function type that maps the consumed context value `Context` and props passed to the connected component `OwnProps` to a subset of the props that can be derived from context `ContextProps`, to pass to the component.

```tsx
type MapContextToProps<Context, ContextProps, OwnProps> = (
  // The consumed context value.
  context: Context,

  // The props passed to the connected component.
  ownProps: OwnProps
) => ContextProps // The props derived from context.
```

### MergeProps

A function type that merges the props that have been mapped from context `ContextProps` with the props passed to the connected component `OwnProps` to return all the props `MergedProps`, to pass to the wrapped component.

```tsx
type MergeProps<ContextProps, OwnProps, MergedProps> = (
  // The result of `mapContextToProps`.
  contextProps: ContextProps,

  // The props passed to the connected component.
  ownProps: OwnProps
) => MergedProps // The props to pass to the given component.
```

## Frequently Asked Questions

### Can I pick state and only re-render when necessary?

Sure. Consider using [`PureComponent`](https://reactjs.org/docs/react-api.html#reactpurecomponent) or [`shouldComponentUpdate`](https://reactjs.org/docs/react-component.html#shouldcomponentupdate) to let your components know when or when _not_ to update.

Additionally, unlike [Redux](https://github.com/reactjs/redux), React 16.3 allows the creation of _multiple_, composable `Context`s, so ideally, you'd be using a `Context` that is small enough to house _just the information_ that you'd like to reuse in order to properly [separate concerns](https://en.wikipedia.org/wiki/Separation_of_concerns) and correctly use the [principle of least privilege](https://en.wikipedia.org/wiki/Principle_of_least_privilege) when passing context around.

---

Made with ❤️ at [Contiamo](https://contiamo.com) in Berlin.
