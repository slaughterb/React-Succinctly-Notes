# React-Succinctly-Notes
Abridged notes from Samer Buna's React Succinctly, freely accessible at https://www.syncfusion.com/succinctly-free-ebooks/react-succinctly. The book and these summarized notes cover the fundamental concepts of React's library and discuss how it fits in with other web development paradigms. It covers React's evolution and what distinguishes it. 

Chapter 1 - What is React?

React is a JavaScript library used to describe views based on provided state. When using React in a browser it can mount the described views in the browser's DOM and update the view whenever the state changes. 

One of React's core concepts is reusable, composable and stateful components. We build views composed of smaller components that can be reused in multiple places/contexts. Each component has a private state that may change over time and React will update the component's view when the state changes. 

React.js updates reactively, hence the name of the library. When a component's state changes those changes have to be reflected somewhere. For instance, the HTML views need to be regenerated for the browser's DOM upon a change of state. React will automatically update the view upon reacting to a change of state. 

With React we use JavaScript to generate HTML that is dependent upon some specified data, as opposed to enhancing HTML to make it work alongside said data. Other popular frontend JavaScript frameworks often enhance HTML (i.e. Angular loops, conditionals etc.) This approach has its pros and cons but React's structure is instead centered around using JavaScript's functionality to generate a HTML view based upon the components' states.

Using JavaScript to render HTML allows React to have a virtual representation of HTML in memory (known as the virtual DOM) and React uses it to render the views virtually first. Upon a change of state, React will only write the difference between the new tree and the previous tree since it has both trees stored in memory. This is known as tree reconciliation.

The process of tree reconciliation is explained at https://legacy.reactjs.org/docs/reconciliation.html in significant depth. At a given point in time you can consider the render() function as essentially creating a tree of React elements. On the next update to state/props that corresponding render() call wil generate a different tree of React elements reflecting the changes. Tree reconciliation is used to efficiently update the UI to match the corresponding state changes. The algorithmic solutions for identifying the minimum number of operations to transform a tree into another are of the complexity O(n^3), which is evidently far too costly to implement in practice. 

React implements a heuristic O(n) alternative to effectively update the new virtual DOM tree upon a change of state. It relies on two assumptions: that two elements of different types will product different trees, and that the developer can hint at which child elements may be stable across different renders by using a key prop. In practice these assumptions are almost always valid.

The Diffing Algorithm behaves differently depending on whether or not the root elements of the trees are of the same type. If the root elements of the trees have different types React will tear down the old tree and build a new one from scratch. When a tree is torn down the old DOM nodes are destroyed via componentWillUnmount(). When constructing a new tree the new DOM nodes are inserted into the DOM - component instances are implemented via componentDidMount(). Thus, any associated state with the old tree will be lost. In the example below the old Timer will be destroyed and a new one will be remounted: 
```
<div>
    <Timer />
</div>

<p>
    <Timer />
</p>
```

When two React DOM elements are of the same type, React will observe their respective attributes and only update the changed attributes using the same underlying DOM node.
Therefore, in the below example React knows to only modify the className on the underlying DOM node:
```
<div className="before" title="sample element" />
<div className="after" title="sample element" />
```

Note that when updating style, React will also know to update only the changed properties - in the example below React will know to update color, but not fontWeight:
```
<div style={{ color: 'red', fontWeight: 'bold'}} />
<div style={{ color: 'green', fontWeight: 'bold'}} />
```

An important takeaway is that when a component update of the same type occurs, the instance stays the same so that the state is maintained across renders. React updates the props of the underlying component with the values to match the state of the new element. 

Chapter 2 - Why React?

A strong-suit of HTML's design is that it allows us to be declarative when building our webpages - for instance, when we declare a DOM element and tell the browser to render it we don't need to specify how the browser will go about doing so. HTML is designed so that we don't need to worry about the steps the browser will take to design the given element. That being said, HTML is limited in functionality - HTML doesn't have variables, loops, or if statements. Data is always in the equation when developing websites - we use HTML to represent data, even when said data is static. However, when we need to represent a change in data we would need to manually update the HTML to reflect the change. Naturally, this does not scale when the request frequency increases. We need JavaScript to implement dynamic content on webpages. 

Consider an example where we wish to represent a list of products in HTML - each product will have an associated name and price. Here's one representation of sample data:
```
{
    "products": [
        { "name": "React.js Essentials", "price": 2999 },
        { "name": "Pro React", "price": 2856 },
        { "name": "Learning React Native", "price": 2199 }
    ]
}

```

If we had a small sample of data in this example we could manually write some JSON and this may be feasible. Since product lists will be dynamic, however, structuring the data this way may not scale, so we could instead come up with HTML that always represents the last state of the product's data. 

To generate the HTML we can use database tables (or collections) to represent data. In this products example, we could create a products table:
```
create table products (
    name text,
    price integer
)
```
The rows of this generate would effectively represent a single product. We could give the client write access on the table allowing them to add, remove, and update their list of products as desired. 

To generate HTML representing these products, we can read the last state of the data from this table and process it in some language (i.e. Java, Note, etc.) to make the program concatenate HTML strings based on the data. Pseudocode example:
```
htmlConcat = "<html><body><ul>";

exampleDB.connect(url)
for (row in exampleDB) {
    htmlConcat += "<li>" + row.name + " - " + row.price + "</li>";
}
htmlConcat += "</ul></body></html>";
exampleDB.close()
```
We could hypothetically write this htmlConcat value to an HTML file and ship it. Every time the customer updates the database table of products, we could regenerate and ship the new HTML. 

This would work, as it gave us a more scalable and powerful means through which we could process dynamic data, but it would still be ideal if we could write HTML in lieu of performing string concatenation...

This is where enhancing HTML came into play. We could theoretically create a language that mediates between HTML and dynamic languages by using special tags, loops, conditionals etc. JSP and PHP would be examples of this. Pseudocode example:
```
<html><body><ul>
    <% for product in product_list %>
        <li><%= product.name %> 0 <%= product.price %></li>
    <% endfor %>
</ul></body></html>
```

As browsers evolved and became smarter, JavaScript became a standard language supported in all browsers, and the invention of AJAX allowed us to ask servers questions in the background of a webpage. This led to more efficient data representation. Instead of preparing the HTML on the server side and shipping the generated product to the client, we can just prepare the dynamic HTML content within the browser using newer technologies. JavaScript, for example, has this capability. The former markup-hybrid language is a good start, but having something compatible with the browser's JavaScript engine would evidently be beneficial; thus frameworks like Angular, Ember and others were developed to simplify this process. 

Much like the enhanced HTML above, in Angular we would do: 
```
<html><body><ul>
    <li *ngFor="#product of products">
        {{product.name}} - {{product.price}}
    </li>
</ul></body></html>
```
This works great but when this approach is used to build big applications, some issues arise. First, memory needs to be managed more carefully since we are no longer refreshing each time we click a link; thus, we need to consider the performance of our decisions. Secondly, while browsers are great for rendering HTML trees dynamically, they can still perform slowly (especially when updating the tree). Traversals need to occur along with redrawing the user screen upon updates, which is naturally an expensive process. Lastly, we're not modeling the state of our data directly with user interfaces, and instead we're writing code that expressed the steps and transitions needed to update the UI (not ideal).

To address these issues, Facebook engineers came up with these core principles when establishing React:

First, enhancements to HTML are noise, we can concatenate strings in JavaScript itself and thereby sacrifice some readability for performance. Secondly, the DOM is a bottleneck and should thus be avoided as much as possible.

HTML views are hierarchical, and a document is represented as a DOM tree of node elements. Facebook engineers utilized JavaScript's functionality to represent nodes as objects and to use these virtual representations to allow for a new frontend paradigm. Thus, the first major design decision in React was to represent every HTML element with a JavaScript object.

We can use http://jsbin.com/nufuwas/edit?js,output as a playground to experiment with React demos and interpret the resultant outputs. 

React's API today has a createElement function, which as it suggests creates a DOM element. The below statements will operate equivalently: 
```
<img src="logo.png" />
React.createElement("img", {src: "logo.png"});

<a href="/">Home</a>
React.createElement("a", {href: "/"}, "Home");
```
> the created img element will show up in console as such:
```
Object {$$ typeof: Symbol(react.element), type: "img", key: null, ref: null, props: Object...}
```

To represent a parent-child relation, we can use the arguments of the React.createElement function. We can pass one or more React objects starting from the 3rd parameter:
```
<a href="/">
    <img src="logo.png" />
</a>

React.createElement("a", {href: "/"}, React.createElement("img", {src: "logo.png}));
```
When inspected in console, the props attribute of the <a> element will have a children attribute holding the object for the <img> element.  This is how trees are represented in React. Under the hood React will eventually generate HTML strings from the JavaScript objects and concatenate them together. 

There are benefits to writing HTML with JavaScript, namely that we can use JavaScript's functionality to work with data. Our products example becomes: 
```
React.createElement("ul", {}, ...products.map(product =>
        React.createElement("li", {}, `${product.name} - ${product.price}`))
);
```
This is all JavaScript, so we don't need to mix in new HTML tags or new functionality beyond the JavaScript functions provided by the React library. We'll have a separate virtual representation of HTML in memory, separate from the browser's DOM. Thus, we can optimize our DOM operations by using a smart diffing algorithm (detailed above) on the structures that we have in memory. This will only be relevant when things change - this concept is what we call the virtual DOM.

React's virtual DOM and diffing algorithm are both game-changing concepts, but there is more to React's design than just the VDOM and the means of updating it. Views in React have a private state, so our product list view can have 3 product objects represented in that private state, and then update that private state when new data comes in. React will trigger a DOM reload whenever the state of the view changes, thereby functioning like a "special refresh button" in the browser. 

React performs this "refresh" by using the diff it computed via the virtual DOM, and only applying that diff to the browser's tree. As developers we won't need to worry about this mechanism; we can simply declare state changes and let React manage these operations under the hood to reflect said changes in the browser's DOM. With React, we model the state of our user interfaces, not the transitions on them. If there are changes, we simply model the new state.

React effectively communicates via the "new language of user interface outcomes." We can communicate with browsers in this higher-level language to express user interfaces declaratively. Within the context of the product list example, the command to the browser is along the lines of "Browser, we have a list of products for you, and changes might happen to that list." 

Chapter 3 - Declarative User Interfaces

With React's language of outcomes, we can build declarative UI and work with the data as well. In the products example we can display an extensible list of products with name and price attributes, without any regard for the length of the products list or additional UI specifications. If we have transitions on the data we don't need to worry about the UI - we can simply manage the state of our new data. Some examples of data transitions may be the following: 
- One product was removed from the catalog
- Three more products were added into the catalog
- A product name has changed
- Five products' prices have changed.
- The order of the products in the catalog has changed

However, we will naturally need to update our respective views if the structure of our data changes. After updating the structural transition for the first time, future updates will again become data transitions in which we won't have to worry about the UI.

This mental model about modeling the final state is simpler work with when our views have many data transitions. An example of this might be a view on a social media platform telling you how many friends are online. The view will simply be an integer reflecting how many friends are online - it will not concern itself with the individual contexts behind the additions and deletions. 

There are tradeoffs to writing in HTML vs JSX. HTML has significant advantages in terms of readability and being concise. That being said, JavaScript has significant advantages in displaying dynamic data over static templates.  JSX is an HTML-like compromise that serves as a JS syntax extension used for creating React elements. It's like an enhancement of JavaScript to create a syntax with the appearance of HTML. We use this JSX within the return statement of a component's render() function. JSX is considered "HTML-like" because it can't be exactly HTML - some element attributes have to be used as the DOM API defines them, such as 'className and htmlFor' in lieu of the regular 'class and for':
```
<form>
    <label for="email">Email:</label>
    <input type="email" id="email" class="form-control" />
</form>

React.createElement("form", null, React.createElement("label", {htmlFor: "email"}, "Email:"), React.createElement("input", {type: "email", id: "email", className: "form-control" })

<form>
    <label htmlFor="email">Email:</label>
    <input type="email" id="email" className="form-control" />
</form>
```
JSX is technically completely optional and not required to use React (though it vastly improves readability). We could theoretically write React in plain JavaScript and omit the JSX extension. 

JSX isn't shipped with React and we need 3rd party tools such as Babel to work with JSX.
Writing in JSX has the advantage of being concise and familiar. It is much easier to read through and parse an HTML-like syntax than it is to comb through a nested assortment of JavaScript objects. 

Chapter 4 - React Components

User interfaces are defined as components in React. Components can be beneficial because of readability (descriptive naming), reusability, and composability (using custom components to composer bigger ones).
```
var searchEngines = function(props) {
    return (
        <div className="search-engines">
            <ClickableImage href="http://google.com" src="google.png" />
            <ClickableImage href="http://bing.com" src="bing.png" />
        </div>
    );
}
```

```
var data = [
    { href: "http://google.com", src: "google.png" },
    { href: "http://bing.com", src: "bing.png" },
    { href: "http://yahoo.com", src: "yahoo.png"}
];
```

```
var SearchEngines = function(props) {
    return (
        <List>
            {props.data.map(engine => <ClickableImage {...engine} />}
        </List>
    );
}
...

ReactDOM.render(<SearchEngines data={data} />, document.getElementById("react"));
```
Note that the ... operator spreads the attributes of an engine as opposed to doing:
href={engine.href} src={engine.src} etc etc...

This SearchEngines component example is reusable and it is composed of the ClickableImage components. The components' nomenclature also describes the intent of each of the components, enhancing readability.

In React, a component manages changes using a state object. In addition to the data we pass as props to components, React components can also have a private state that changes over time. 

For instance, a timer component storing its current timer value within state:
```
<Timer initialSeconds={42} />

...
(decrement counter function each second):
state.counter--;
if state.counter <= 0
    stopTimer();
...

function() {
    return (
        <div> {this.state.counter} </div>
    );
}
```

Creating React components - Examining ClickableImage component that takes href and src input properties. When it's complete it can be mounted in the browser using ReactDOM's render:
```
ReactDOM.render(
    <ClickableImage href="google.com", src="google.com" />,
    document.getElementById('react');
);
```
Note that ReactDOM is a library separately maintained from React that can be used alongside it to work with a browser's DOM. You need to import it or include its CDN entry to use it in a project.  ReactDOM.render()'s first parameter is the React needs to be rendered and the second is where to render it in the browser.

The three main ways to define a React component:
- Stateless function components
- React.createClass
- React.Component

Stateless functional components are modeled after functions and we can use vanilla JavaScript to design these components: 
```
var clickableImage = function(props) {
    return (
        <a href={props.href}>
            <img src={props.src} />
        </a>
    );
}
```
When we use this type of functional component we aren't creating an instance from a component class; rather, the function represents the content that the render() method within a component would contain. If we can design our component in a declarative way then using a stateless functional component would be a viable option (because there would be less dynamic behavior and changes to deal with within the component). By definition, stateless function components can't have internal state, won't expose any lifecycle methods, and can't have any refs attached to them. We use class-based component definitions to implement this functionality. 

With the ES6 arrow function syntax, we can more concisely define ClickableImage as follows:
```
var clickableImage = props => (
    <a href={props.href}>
        <img src={props.src} />
    </a>
);
```

React.createClass is the official API to define a stateful component. When defining ClickableImage this way, the component would be:
```
var clickableImage = React.createClass({
    render: function() {
        return (
            <a href={this.props.href}>
                <img src={this.props.src} />
            </a>
        );
    }
});
```
The createClass function takes a single argument, a JavaScript configuration object. That object requires a render property, where we will define the component's function to describe its UI. With createClass we don't pass the props object to the render call. Instead, elements created from this component class can access their properties using this.props within the render() function. 

The this keyword references the instance of the component that we mounted in the DOM (using ReactDOM.render). Every time we mount a <ClickableImage /> element we are creating an instance of the ClickableImage component class. In OOP terms, ClickableImage is the class blueprint and <ClickableImage /> is the instantiated object from said class. With createClass we can use the private state of the component object and can involve custom behavior in its lifecycle methods. See the example Timer component: 
```
ReactDOM.render(
    <Timer initialSeconds={42} />,
    document.getElementById('react')
);


...
Timer Definition:
...
var Timer = React.createClass({
    getState: function() {
        return {
            counter: this.props.initialSeconds
        };
    },
    componentDidMount: function() {
        var component = this, currentCounter;
        component.timerId = setInterval(function() {
            currentCounter = component.state.counter;
            if (currentCounter <= 1) {
                clearInterval(component.timerId);
            }
            component.setState({
                counter: currentCounter - 1
            });
        }, 1000);
    },
    render: function() {
        return (<div>{this.state.counter}</div>);
    }
});
```
The counter variable in this example would be part of the component's private state. The private state object can be accessed via this.state. The Timer component has the property initialSeconds, which would be the starting point of the counter. Thus, if we were to read the initialSeconds value directly we would use this.props.initialSeconds. We can utilize this with the getState function within the class above to initialize the state's attributes.

To implement the Timer's tick operation we can use the vanilla JS setInterval function every 1000ms (1 second) and inside the counter function we can decrement the state's counter during each call. The ticking operation should begin after the component is rendered to the DOM so we will use the lifecycle method componentDidMount(). The componentDidMount() function allows us to define a custom behavior right after the component is mounted in the DOM.

Some things to note about the setInterval portion: we had to use a closure around setInterval so that we can access the 'this' keyword. This is an old-school JavaScript trick made obsolete by the new ES6 arrow function syntax. To change the state of the component we use setState. In React, we should never mutate the state directly as a variable. To organize the changes being made we should make all changes to the state via setState. Lastly, to prevent the timer value from going negative we can use the clearTimeout function to stop the timer.

After ES2015 we can now use class syntax within JavaScript, and in doing so we can utilize React.Component and the extends keyword for data inheritance:
```
class ClickableImage extends React.Component {
    render() {
        return (
            <a href={this.props.href}>
                <img src={this.props.src} />
            </a>
        );
    }
}
```
Within the class definition the render() function essentially behaves the same but now utilizes the ES6 syntax to define it. In ES6 the function keyword can theoretically be avoided completely. See the ES6 rendition of the timer below:
```
class Timer extends React.Component {
    constructor(props) {
        super(props);
        this.state = { counter: this.props.initialSeconds };
    }
    componentDidMount() {
        let currentCounter;
        this.timerId = setInterval(() => {
            currentCounter = this.state.counter;
            if (currentCounter <= 1) {
                clearInterval(this.timerId);
            }
            this.setState({ counter: currentCounter - 1 });
        }, 1000);
    }
    render() {
        return (
            <div>{this.state.counter}</div>
        );
    }
}
```
Some key differences above are the utilization of a class constructor, use of arrow functions, and components being defined using the new function property syntax. 

Component classes, elements, and instances
The class blueprint is often referred to as the "Component".

A React Element is the instantiation of a component (i.e. <Timer />). This is a stateless, immutable virtual DOM object.

When a React element is mounted in the browser's DOM it becomes a component instance, which is stateful. The result of a ReactDOM.render() call is a component instance.

Chapter 5 - Composability 

Several React components can be combined to produce another React component. This is one of the best features of React. It's a simple concept with great advantages. Composability enables abstraction and allows us to understand code without having to monitor all the details all of the time. For instance, if a Profile component contains a ProfilePicture component the readability of the components' nomenclature helps us understand the intent of each portion of code. 

Component reusability also has the benefit of improving code structure and avoiding the repetition of component definitions. That being said, the most significant benefit of composability is that it allows us to partition different concerns of our applications with flexibility and extensibility. 

To illustrate this, web observe the example of twitters homepage and approximate its composability. We have a header, logo, navbar, then below an avatar with user information and a list of tweets. When creating these compositions it helps to understand the hierarchical relationships between components and the box-model display of CSS. 

Just like we can include HTML elements within the tags of a parent element, we can also include React elements within React elements. We can also mix HTML elements and React elements in both cases, for instance:
```
<Counts>
    <TweetsCount />
    <FollowingCount />
    <FollowersCount />
    <LikesCount />
</Counts>
```
The inner elements in this example are called the "children" of the outer element "Counts", and we can access this list of children within any instance of the component using this.props.children. 
```
class Counts extends React.Component {
    render() {
        <div id="counts-headers">
            {this.props.children}
        </div>
    }
}
```
Note that React's tree reconciliation process uses the order of the children in the diffing algorithm. Thus, if a child is removed from the tree all siblings after it will need to be updated in the process. This problem becomes significant when the children are driven by a dynamic array that is shifting/unshifting, or also in cases where elements of the array are shuffled.

To remedy this and to enforce React's respect of the identity and state of each child in a list, we uniquely identify each child instance by assigning it a key prop. We may write something along the lines of:
```
class ProductList extends React.Component {
    render() {
        return (
            <div>
                {this.products.map(product => 
                    <Product key={product.id} product={product} /> )}
            </div>
        );
    }
}
```
React will provide a warning if we do not supply a unique key to a mapped array. This warning is significant. This key prop cannot be used within the component definition either - it won't be accessible via this.props.key. Do not use the index of the array elements as a key value, because this will essentially change the identities of the children whenever the array is shuffled or operated upon. This will naturally lead to unpredictable behavior. 

When owners set the props of owned components, they are performing a one-way data binding. The owner will bind the props of its owned components based on some computations on its own props and state. This process happens recursively and any changes in computation for the higher-level components will be reflected down the chain as they're passed to inheriting components. This means that there is a one-way data binding flow down through the children, but not one moving up along the inheritance hierarchy. 

Chapter 6 - Reusability

Function components are the best when it comes to reusability because they are pure functions with no state. This makes them predictable with regards to the inputs and outputs they will require and generate.

Stateful components can be usable too as long as the managed state is private to every instance of the component. Reusability becomes more difficult when we depend on an external state. 

Input validation is a demonstrable example of this. If an input processing component is reused in multiple places, the manual validation of input becomes a problem. Sometimes a component may not work despite no errors being reported. 

For example, observe a component NumberList that takes an array of numbers and displays each one of them in a div. The component also takes a number value and highlights said value in the array if it is present:
```
const NumberList = props => (
    <div>
        {props.list.map(number => 
            <div key={number} className={`highlight-${number === props.x}`}>
                { number }
            </div>
        )}
    </div>
);
```

```
ReactDOM.render(<NumbersList list={[5, 42, 12]} x={42} />, document.getElementById('react'));
```
In this example props.list is the array of numbers and props.x is the number to highlight if it exists within the list. To make the logic visible we can style highlight-true and highlight-false.
Not only will the above example fail to work as expected, but we also won't identify any errors or warnings about the problem. 

In the above situation it would be beneficial to perform input validation on the x prop, allowing the component to expect only an integer value for x. React.PropTypes is a means through which we can achieve this desired goal of enforcing prop types. We can set a propType property on every component. In that property we can provide an object where the keys are the input props that need validation, and these values map to the data type that React should use to validate. Example implementation below:
```
NumbersList.propTypes = {
    x: React.PropTypes.number.isRequired
};
```
This way if we pass x as a string instead of an integer, React will now throw a warning in the console. 

React.PropTypes  has a range of validators we can use on any input, as listed below:
JavaScript primitives:  [array, bool, number, object, string]
node - anything that can be rendered
element - a React element
instanceOf(ExampleClass) - uses instanceof operator
oneOf(['Approved', 'Rejected']) - for Enums
oneOfType([..., ...]) - Either this or that
arrayOf(React.PropTypes.number) - Array of a certain type
objectOf(React.PropTypes.number) - Object with property values of a certain type
func - a function reference.

By default, all props we pass to components are optional but we can change taht by using React.PropTypes.isRequired, which can be chained to other validators (as shown in above example). 

In addition to the validators in React.PropTypes, we can also design and use custom validators. These validators are simply functions to perform custom checks on inputs. In the below example we can validate that the text of a tweet doesn't exceed 140 characters:
```
Tweet.PropTypes = {
    tweetText: (props, propName) => {
        if (props[propName] && props[propName].length > 140) {
            return new Error('Too long');
        }
    }
}
```
For all components created with React.createClass, propTypes is just a property on the input object. For the regular class syntax, we can use a static property:
```
// React.createClass syntax
let NumbersList = React.createClass({
    propTypes: {
        x: React.PropTypes.number
    },
});

// class syntax
class NumbersList extends React.Component {
    static propTypes = {
        x: React.PropTypes.number
    };
}

// functions syntax (also works for class syntax)
NumberList.propTypes = {
    x: React.PropTypes.number
}
```

Another useful thing we can do when handling input props for components is assign them a default value in case we use the component without passing a value to a given prop. For example, we can create an alert box that has a default message "Something went wrong." when unspecified:
```
const AlertBox = props => (
    <div className="alert alert-danger">
        {props.message}
    </div>
);

AlertBox.defaultProps = {
    message: "Something went wrong."
};

ReactDOM.render(<AlertBox />, document.getElementById('react'));
```

The syntax to use defaultProps with React.Component is similar to propTypes. For React.createClass, we use the method getDefaultProps():
```
const AlertBox = React.createClass({
    getDefaultProps() {
        return { message: "Something went wrong" };
    }
});
```

Sometimes reusing the entire component isn't an option and shared component behavior is the desired goal. We can have cases where some components share most of their functionalities but differ in some aspects. If we need different components to share common behavior we can:
1. use mixins if we're working with React.createClass
2. create a new class and have all components extend it if we're working with React.Component syntax. We can manually inject the external methods we'd like our classes to have in their constructors. 

Mixins are objects that we can mix into any component defined with React.createClass. 
```

const logClicksMixin = {
    logClick() {
        console.log(`Element ${this.props.id} clicked`);   
        $.post(`/clicks/${this.props.id}`);
    },
};

const Link = React.createClass({
    mixins: [logClicksMixin],
    handleClick(e) {
        this.logClick();
        e.preventDefault();
        console.log('Handling link click...');
    },
    render() {
        return (
            <a href="#" onClick={this.handleClick}>Link</a>
        );
    }
});

const Button = React.createClass({
    mixins: [logClicksMixin],
    handleClick(e) {
        this.logClick();
        e.preventDefault();
        console.log('Handling a button click...');
    },
    render() {
        return (
            <button onClick={this.handleClick}>Button</button>
        );
    }
});

ReactDOM.render(
    <div>
        <Link id="link1" />
        <br /> <br />
        <Button id="button1" />
    </div>
    document.getElementById('react');
);

```
This approach allows the logClick implementation to be abstracted in one place. Thus, if we need to make any alterations to the implementation we only need to change one focal point when designed this way. To do this with the React.Component class syntax we could create a new class Loggable and implement the logClick in there, then make both Button and Link components extend Loggable. 
```
class Loggable extends React.Component {
    logClick() {
        console.log(`Element ${this.props.id} clicked`);
        $.post(`/clicks/${this.props.id}`);
    }
}

class Link extends Loggable {
    handleClick(e) {
        this.logClick();
        e.preventDefault();
        console.log('Handling link click...');
    }
    render() {
        return (
            <a href="#" onClick={this.handleClick.bind(this)}>Link</a>
        );
    }
}

class Button extends Loggable {
    handleClick(e) {
        this.logClick();
        e.preventDefault();
        console.log('Handling button click...');
    }
    render() {
        return (
            <button onClick={this.handleClick.bind(this)}>Button/button>
        );
    }
}

ReactDOM.render(
    <div>
        <Link id="link1" />
        <br /><br />
        <Button id="button1" />
    </div>
    document.getElementById('react');
);
```

Chapter 7 - Working with User Input

By applying the accumulated previous knowledge about React's functionality we can now account for UI events and capture inputs from a user on our application. The nature of user input can be described as a reverse data flow when compared to how React's components represent data. Naturally when interpreting user input, the flow of data begins from the user's client-side interaction with the UI. Thus, the data flows 'upwards' in terms of hierarchical specificity. 

Suppose we choose to display a list of comments: we would start with an array of comments in the data layer and represent that within components in the user interface. Then we could include a form on the page to add a new comment. Now the data array will be receiving information based on users' interactions with this input form. 

If we handle the input events, capture the data that users enter, and reflect those changes in our original data then React will refresh the views that are utilizing that data. 

We can use refs attributes as a simple to work with user input, however the recommended method is to use controlled components. To understand the happenings under the hood we will need to observe the synthetic events system in React. 

Using React's synthetic events we can define native browser events on DOM elements. If we wanted to log the content of a div when it's clicked, for example, we could do:
```
<div onClick = {se => console.log(se.target.textContent)}>
    You might not need React..
</div>
```
In this example we used an onClick synthetic event to replicate the behavior that the line below would have accomplished:
```
onclick="console.log(this.textContent);"
```

Some key differences in these examples are that React uses camelCasing for event attributes (onClick, onKeyPress etc.), we don't use strings with React events (it uses JavaScript directly), and we invoke the function provided as a value during the event triggers on an element. 

React's synthetic events are wrappers around the browser's native events. We can still access the native event itself using the nativeEvent attribute on any synthetic event (i.e. se.nativeEvent). 

Browsers have historically had differences in how they invoked and handled native events (though this has improved a lot with W3C standards) but React synthetic events will work identically across all browsers. An example of this is with React's onChange - although it works differently among different input types, it usually means that "the user is done changing" rather than simply on any complete "change" to the element. Therefore, the onChange event in react essentially tracks any change, any where, anytime for any input. When typing in input or textarea React will fire the onChange event for every keystroke transmitted. With a clear/checkbox React will fire the onChange event. Picking an option from a drop-down select element will fire the onChange event in React. 

Below are some examples of common events we can use in React:
- Keyboard events, i.e. onKeyDown, onKeyPress, onKeyUp
- Focus events, i.e. onFocus, onBlur
- Form events, i.e. onInput, onChange, onSubmit 
- Touch events, i.e. onTouchStart, onTouchEnd, onTouchMove, onTouchCancel
- Mouse events, i.e. onClick, onDrag, onMouseOver, onMouseOut, etc.
- Lastly there are clipboard events, UI events, wheel events, composition events, media events, and image events.
- We can also work with events not supported in React by using addEventListener and removeEventListener in componentDidMount and componentWillUnmount respectively. 

The virtual DOM allows us to make alterations without ever interacting with the DOM directly. React's synthetic events essentially create a "fake browser" that respects all standards, is easier to work with, and has a faster performance.

However, this doesn't mean that we can't use the original browser's DOM and events with React if need be. Sometimes we need to do unique things with the DOM like integrating components with 3rd-party libraries, etc. If we need to access a native DOM node in any React component we can use the special ref attribute. This attribute may take either a string or a function, although using functions is preferable. 

For example, let's consider an email input with a save button. Before we save we want to check that the input is a valid email. If using <input type="email" /> we can use the native API method element.checkValidity() to figure out the validity of the email. To invoke this we need to access the native DOM element:
```
const EmailForm = React.createClass({
    handleClick() {
        if (this.inputRef.checkValidity()) {
            console.log(`Validated - Saving email as ${this.inputRef.value}`);
        }
    },
    render() {
        return (
            <div>
                <input type="email" ref={inputRef => this.inputRef = inputRef} />
                <button onClick={this.handleClick}>Save</button>
            </div>
        );
    }
});

ReactDOM.render(<EmailForm />, document.getElementById('react'));
```


