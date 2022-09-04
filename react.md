# React

## JSX

- syntax extension
- JSX expressions => JS functions => JS object
- basics:
    ```javascript
    // can embed expressions
    const name = 'Ashley';
    const element = <h1>Hello, {name}</h1>

    // specify attributes with string or JSX
    // attribute names are camelCase
    const element = <a className="link" href={profileLink}>example.com</a>

    // empty tag can be closed with '/>'
    const element = <img src={user.avatarURL} />
    ```
- DOM escapes any embedded expressions before rendering (prevents injection attacks)

## Elements
- building blocks of React
- how to render:
    ```javascript
    const root = ReactDOM.createRoot(
        // there is an element with an id="root"
        document.getElementById('root')
    );
    const element = <h1>Hello, world</h1>;
    root.render(element);
    ```
- elements are immutable (can't change attributes, children)
- elements represent UI at certain point in time
- to update UI: create a new element and pass to `root.render()`
    - React DOM only updates what is different from previous element

## Components
- like a JS function (accepts inputs/**props**) and return React element
- function vs class:
    ```javascript
    // function
    function Welcome(props){
        return <h1>Hello, {props.name}</h1>
    }

    //class
    class Welcome extends React.Component{
        render(){
            return <h1>Hello, {this.props.name}</h1>
        }
    }
    ```
- how to render:
    ```javascript
    // elements can be user-defined components
    // all attributes are passed to component through single object (props)
    const element = <Welcome name="Ashley" />
- always start component name with capital letters
- can compose components:
    ```javascript
    function App(){
        return (
            <div>
                <Welcome name="Ashley" />
            </div>
        );
    }
    ```
- React components **CANNOT** modify props (must be a pure function)

## State & Lifecycle
- state is private and controlled by its component
- how to add state:
    ```javascript
    // class - use a constructor
    class Clock extends React.Component{
        constructor(props){
            super(props); // pass props to base constructor
            this.state = {date: new Date()};
        }
        render(){
            return (
                <h1>It is {this.state.date.toLocaleTimeString()}.</h1>
            );
        }
    }

    // function - useState hook
    import { useState } from 'react';
    function Clock(){
        const {date, setDate} = useState(new Date());
        return (
            <h1>It is {date.toLocaleTimeString()}.</h1>
        );
    }
    ```
- lifecycle methods
    - `componentDidMount()`: when component is rendered to DOM for first time
    - `componentWillUnmount()`: when DOM produced by component is removed
    ```javascript
    tick(){
        // setState tells React that state has been changed and calls render() to update
        this.setState({
            date: new Date()
        });
    }

    componentDidMount(){
        // sponatenously added additional field to class
        this.timerId = setInterval(
            () => this.tick(), 1000
        )
    }

    componentWillUnmount(){
        clearInterval(this.timeId);
    }
    ```
- how to use state correctly
    - use `setState()` to modify
    - state updates may be asynchronous, instead use second form of `setState()`
        ```javascript
        // previous state as first parameter
        // props as second parameter
        this.setState((state, props)) => {
            counter: state.counter + props.increment
        }
        ```
    - updates to individual states will persist with each other
    - data flows down
        - state is passed as prop to child
        - child would not know if props was a state, prop, or manually typed

# Events
- 

# Conditionals
- using `if` and `else` statements in JSX expressions
- inline if with `&&`
    - e.g. `{unreadMessage.length > 0 && <Component />}`
    - note: expression is returne with value even if value is falsy
- ternary operator
- return `null` to hide component (does not affect lifecycle methods)

# Lists
- use `map()``
- **key** (string attribute) to identify items with stable identity
    - don't use index as keys
    - key must be unique among siblings
    - key are not props passed into components
    ```javascript
    const element = { numbers.map((number) => 
        <li key={number.toString()}>
            {number}
        </li>
    )};
    ```
    
# Moving state up
- when components share the same state
- move the state up to common parent (or make one) and pass to components are props

# Other
- **controlled components**: set state when `onChange()` is called
- **composition**: generic > specifc components