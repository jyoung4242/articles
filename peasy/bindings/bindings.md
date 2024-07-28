# Bindings and Template: Part 2 of the Peasy-UI Series

<!-- TABLE OF CONTENTS -->
<details>
  <summary>Table of Contents</summary>
  <ol>
    <li><a href="#introduction">Introduction</a></li>
    <li><a href="#bindings-and-the-template">Bindings and the Template</a></li>
    <li><a href="#text-bindings">Text Bindings</a>
      <ul>
        <li><a href="#basic-binding">Basic Binding</a></li>
        <li><a href="#conditional-boolean-text-binding">Conditional Boolean Text Binding</a></li>
        <li><a href="#one-time-text-bindings">One-time Text bindings</a></li>
      </ul>
    </li>
    <li><a href="#attribute-bindings">Attribute Bindings</a>
      <ul>
        <li><a href="#bindings-for-element-attributes">Bindings for element attributes</a></li>
        <li><a href="#bindings-for-events">Bindings for events</a></li>
        <li><a href="#binding-whole-elements-to-data-model">Binding whole elements to data model</a></li>
        <li><a href="#binding-for-rendering">Binding for Rendering</a></li>
        <li><a href="#bindings-for-arrays">Binding for Arrays</a></li>
        <li><a href="#bindings-for-components">Bindings for Components</a></li>
      </ul>
    </li>
    <li><a href="#getters">Getters</a></li>
    <li><a href="#more-information">More information</a></li>
    <li><a href="#conclusion">Conclusion</a></li>
  </ol>
</details>

## Introduction

Today we are going to dive a layer deeper into the Peasy-UI library. We are going to explain the different types of bindings that are
available in the library, and how you can incoporate them into the string literal template that you provide to the library to be
rendered to the DOM. For this article all examples are provided using TypeScript, but the library works with vanilla JavaScript
perfectly fine.

If you missed the introduction article for Peasy-UI, you can read it
[here](https://dev.to/jyoung4242/introduction-to-peasy-ui-part-1-of-the-peasy-ui-series-4gi8), and it can provide the overview for the
library, and even introduces upcoming topics in the series.

<!-- !!!! <pre><code>...</code></pre> -->

## Bindings and the Template

I quick recap of what bindings are, what the template is, and how they work with the library to give you control of what gets rendered
to the DOM.

Peasy-UI is a library that injects content into DOM elements that you specify. The content that gets injected is dictated by a string
literal template that provides HTML markup that represents the content to be injected.

```ts
const template = `
  <div> Render Me </div>
  <button> Click </button>
`;
```

This is an example of the string literal template that you pass to Peasy-UI and it will render it into the targeted DOM element.

Bindings provide a means to control that template dynamically. For example, let's do a Hello World example.

```ts
const model = {
  name: "Bob",
};

const template = `
<div>
    Hello: \${name}!!!!
</div>
`;

await UI.create(document.body, model, template).attached;
```

This code creates the data store (model), that is bound to the html in the string literal template. UI.create takes the model,
template, and the targeted DOM element to inject into as parameters. In your Javascript code, you can change `model.name` to whatever
string you'd like, and it will dynamically re-render the new text into the DOM. Side note: the await in the UI.create method pauses the
execution of subsequent logic until Peasy-UI attaches the content into the DOM. This way you can prevent trying to make changes or
access DOM content that isn't rendered yet.

## Text bindings

There are two types of bindings in Peasy-UI, text bindings and attribute bindings, and we will explore both types. One thing to note is
that since we are providing a string literal to Peasy-UI, we use the backslash escape character to designate it as a binding.

### Basic binding

The most basic binding is simply injecting data into the DOM as text. The Hello World above is a prime example of this. Use cases for
this binding can include manipulating the innertext or innerHTML content inside a DOM element, such as changing a name. I've also used
this for changing CSS properties inside a CSS string.

BINDING: `${propertyName}`

As in the example above, we can use this:

```ts
const model = {
  name: "Bob",
};

const template = `
<div>
    Hello: \${name}!!!!
</div>
`;
```

The binding maps the 'name' property to the tag in the HTML and replaces name with the name content from the model.

### Conditional boolean text binding

In order to control if a binding is active, we can leverage some conditional bindings.

BINDING: `${'value' = propertyName}`

BINDING: `${'value' ! propertyName}`

An example of this would be:

```ts
const model = {
  darkMode: true,
};

const template = `
<div>
    I am using \${'Dark mode' = darkMode} \${'Light mode' ! darkMode} now!
</div>
`;
```

Based on the truthy status of the darkMode property in the model, different text renders. If true, the div will say
`I am using Dark mode now!` and if its set to false, it will render `I am using Light mode now!`.

### One-time text bindings

The previous bindings get constantly evaluated with respect to RequestAnimationFrame(). However, sometimes you may only want to
evaluate it one-time, like on initialization. Peasy-UI provides one-time bindings.

BINDING: `${|'value' = prop}`

BINDING: `${|'value' ! prop}`

The same example above can be reused, the only difference is that the evaluation of the darkMode property will only be done on the
first rendering cycle. Please take note of the pipe character in the binding, as that is what designates it as a one-time binding.

## Attribute Bindings

Attribute bindings use either `\${ }` or the pui attribute within an element tag to affect the behaviour of the element. For the sake
of this article we will continue using the `\${ }` nomenclature. We will explore the different methods of assigning bindings in a
future article.

### Bindings for element attributes

Peasy-UI gives us the power to access element attributes and allows us to monitor and control those values easily.

There are four basic element attribute bindings:

BINDING: `${attribute <== propertyName}` bind in one direction from the model to the binding

BINDING: `${attribute <=| propertyName}` bind in one direction, one-time, from the model to the binding

BINDING: `${attribute ==> propertyName}` bind in one direction, from the element to the model

BINDING: `${attribute <=> propertyName}` bi-directional, or two-way, between the element and the model

Let's look at some examples of this:

```ts
const model = {
  color: "red",
};

const template = `
<div>
    <input type="radio" \${'red' ==> color}>Red</input>
    <input type="radio" \${'green' ==> color}>Green</input>
</div>
`;
```

In this example you are binding a custom attribute on each element 'red' and 'green' to the color property in the model. When the radio
button in this group is selected, it changes the data in the model property 'color'.

Let's look at a bidirectional example:

```ts
const model = {
  userName: "Bob",
};

const template = `
<div>
    <input type="text" \${value <=> userName}>\${userName}</input>
</div>
`;
```

The magic that exists here is that you can set username in the data model programatically, and it will automatically change the
rendered text in the input field. Or, you can type into the input field and it will dynamically change the value in the data model.

### Bindings for events

So what if want to render a drop down select input, or a button? Peasy-UI gives you access to mapping the eventlisters automatically
and tying them into the data model.

BINDING: `${ event @=> callbackName}`

Buttons provide easy examples for this, but all DOM events can be used. event model Element EventTypes peasy data object

```ts
const model = {
  clickHandler: (event: HTMLEvent, model: any, element: HTMLElement, eventType: string, object: any) => {
    window.alert("I got clicked");
  },
};

const template = `
<div>
    <button \${click @=> clickHandler}> Click Me! </button>
</div>
`;
```

Okay, there is a bit to unpack here. First lets focus on the binding, you can bind any HTML DOM event, such as change, input, click,
mouseenter, etc... as an event binding, and then you provide the 'handler' callback which exists IN the data model.

Peasy passes 5 standard parameters into the handler for your convenience. The first parameter is the HTMLEvent that actually is fired.
The second parameter is the 'localized' data model that is bound to the element. This CAN be tricky, because depending on your nesting
of elements, it maybe be a localized data model for just that element, or if it is a top level binding, it will be the standard data
model. We will get into nesting a bit later, but just know that another option is provided to help with this.

The third parameter is the target element that fired the event, so you can access any attributes nested in the element. The fourth
element is the string representation of the event, such as 'click' or 'change' and this helps in case you bind multiple events to the
same callback, you can distinguish which event triggered this callback. The final parameter is the overarching data model. Inside this
object, regardless of how nested your binding is, will be an object that contains the model property, and you can navigate that object
to have access to ANY property in the data model.

### Binding whole elements to data model

Do you know how annoying it is to have to perform a `documement.getElementById('myElement');` in your Javascript? Yeah, Peasy-UI makes
that a bit easier.

BINDING: `${ ==> propertyName}`

Let's look at this timesaver example:

```ts
const model = {
  myInputElement: undefined as HTMLElement | undefined,
};

const template = `
<div>
    <input type="text" \${==> myInputElement} />
</div>
`;
```

What this binding does is maps the DOM elements rendered and binds their reference inside the model. In this example, instead of having
to do the `getElementById()` method, model.myInputElement is that reference after the first rendering cycle. The element has to render
'first' then the reference to it gets captured in the data model. This is why its set to undefined by default, then gets set to the
element reference after the HTML gets injected into the DOM. So now to have access to the element value, i can just access
`model.myInputElement.value` or any attribute in that element that I'd like access to.

### Binding for rendering

What if you want to be selective about what gets rendered and maybe have portions of your template not rendered under certain
conditions. Yup, Peasy-UI allows you to control overall rendering as well.

BINDING: `${=== propertyName}`

BINDING: `${!== propertyName}`

Let's looke at the example of this binding:

```ts
const model = {
  isDog: true,
};

const template = `
<div>
    <div \${=== isDog}> I am a Dog</div>
    <div \${!== isDog}> I am a Cat</div>
</div>
```

In this example you will only see one of the bound Div's being rendered, and by toggling between true and false in the data
model.isDog, you can control which Div is being rendered.

### Binding for arrays

Sometimes you need to list things out in the DOM. Examples may be forum posts, options in a select input, or list items in a nav bar.
Peasy-UI gives you an iterable renderer, so you can translate a list or array in the data model to multiple element renderings.

BINDING: `${ item <=* propertyArray}`

Let's give a simple example:

```ts
const model = {
  myItems: [{text: 'one', value: '1'},{text: 'one', value: '1'},{text: 'one', value: '1'}],
};

const template = `
<div>
    <select>
      <option \${opt <=* myItems} value="\${opt.value}">\${opt.text}</option>
    </select>
</div>
```

So this use case iterates over the 'myItems' property array in the data model, and renders a separate option element in the select tag.

Each rendered option tag has its own model scope as you can see, with opt.text and opt.value.

### Bindings for components

So with larger projects, there is a desire to organize a code base and utilize code splitting to keep aspects of a project manageable.
Peasy-UI allows for component based design. We will 'lightly' touch on this topic in this article as the overall component feature will
get its own article for properly exploring its capabilities.

A component in peasy is any Javascript object which contains a static, public template property and a static create method.

BINDING: `${ComponentName === state}`

Let's look at one basic example, with more to follow in future article:

```ts
class MyComponent{
  public static template = `
  <div>
      <div> Hello \${name}</div>
      <div> My count is: \${count}</div>
  </div>
  `;

  constructor(public name: string,public count:number){}

  static create( state: {name:string, count:number}){
    return new MyComponent(state.name, state.count)
  }
}

const model = {
  MyComponent,
  componentData:{
    name: 'bob',
    count: 10,
  }
};

const template = `
<div>
    <\${MyComponent === componentData}>
</div>
```

So the MyComponent class must have a static create method that accepts incoming state, and a public static template. What peasy will do
is set the local class properties as the internal state for each component, and will inject the class template into the overall project
template.

This can let you break a larger project down into smaller pieces. There is a lot more you can do with components, and it warrants its
own article to explore futher.

## Getters

The final binding idea to be covered is the usage of getters in the data model and what they bring to the table. Sometimes a simple
primative just doesn't cut it in the data model, and getters provide a means of providing more complicated logic and the return value
will be the binding value to the binding. There's no unique binding tag to cover here, just an example:

```ts

const model = {
  val1: 1,
  val2: 3,
  get myGetter(){
      return this.val1 + this.val2;
  },
};

const template = `
<div>
    <span>My getter value is: \${myGetter}</span>
</div>
```

The nice part of using a getter is that it gets re-evaluated every rendering loop. So if any values change within its own logic, it
will get updated.

## More information

More information can be found in the github repo for Peasy-Lib, you also will find all the other companion packages there too. Also,
Peasy has a Discord server where we hang out and discuss Peasy and help each other out.

The author's twitter: [Here](https://twitter.com/jyoung424242)

The author's itch: [Here](https://mookie4242.itch.io/)

Github Repo: [Here](https://github.com/peasy-lib/peasy-lib/tree/main/packages/peasy-ui)

Discord Server: [Here](https://discord.gg/9VsQrVH94Z)

## Conclusion

So this article covered all the basics of the Peasy-UI library's data bindings, which is the magic that powers how the library is used
to render essentially anything you need in the DOM. We discussed Text Bindings and Attribute Bindings. For each we covered the binding
syntax, and how it can be used in examples.
