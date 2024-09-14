# Component Based Architecture in Peasy-UI: Part 5 of the Peasy-UI Series

<!-- TABLE OF CONTENTS -->
<details>
  <summary>Table of Contents</summary>
  <ol>
    <li><a href="#introduction">Introduction</a></li>
    <li><a href="#bindings-and-elements">Bindings for events</a></li>
    <li><a href="#callback-params">Callback Params</a>
      <ul>
        <li><a href="#event">Event</a></li>
        <li><a href="#data-model">Data Model</a></li>
        <li><a href="#target-element">Target Element</a></li>
        <li><a href="#event-string">Event String</a></li>
        <li><a href="#parent-data-object">Parent Data Object</a></li>
      </ul>
    </li>
    <li><a href="#more-information">More information</a></li>
    <li><a href="#conclusion">Conclusion</a></li>
  </ol>
</details>

## Introduction

Today we are going to dive the final layer deeper into the Peasy-UI library. We are going to do a deep dive into how to use Peasy-UI to
build re-usable UI components. This will assist with building modular, scalable, re-usable code blocks and simplify one's code
architecture.

If you missed the introduction article for Peasy-UI, you can read it
[here](https://dev.to/jyoung4242/introduction-to-peasy-ui-part-1-of-the-peasy-ui-series-4gi8), and it can provide the overview for the
library.

### Component Based Design

Larger applications need to be architected in a manner that keeps the code base clean and lean, in a manner where duplicate code isn't
being used. One strategy to help with a "don't repeat yourself" approach is to design your front-end in modular manner, and design that
structure to re-use components. Component based architecture provides increased flexibility, improved maintainability, increased
readibility, and also improves organization of a code base.

### Registering Single File Components with Peasy

Peasy UI makes it possible to create both JavaScript and HTML single file components and import them for use in the app, without the
need of a build step. You also can use this with Typescript when you bundle with your preferred bundling tool that transpiles your TS
files into JS. Peasy-UI only needs a template property in order to render a component, but for Peasy-UI to instantiate and render
components based on a template object/class and (optionally) data, the template object/class needs:

- a template property
- a create method (which gets invoked with designated data model)
- to be known to the parent model either as a property or through the use of the UI.register method

#### JavaScript (TypeScript) Single File Component

There are two ways of fulfilling the 3rd requirement above, providing the component to the parent model. You can either use UI.register
method, or include the exported class into the parent data model yourself. For this example, we will use the UI.register, but in the
demo application below, we demonstrate the other method of meeting this requirement.

```ts
// list-item.js

import { UI } from "https://cdn.skypack.dev/@peasy-lib/peasy-ui";

export class ListItem {
  static template = "<span>Hello, ${name}!</span>";

  constructor(name) {
    this.name = name;
  }
  static create(state) {
    return new ListItem(state.name);
  }
}
UI.register("ListItem", ListItem);
```

Then we can import it into our HTML and implement it, the below example demonstrates how to bypass the build step requirement, by way
of CDN import.

```html
<!-- index.html -->
<body>
  <template id="sfc-app">
    <div>
      <div pui="name <=* names"><list-item pui="ListItem === name"></list-item>
    </div>
  </template>

  <script type="module">
    import { UI } from "https://cdn.skypack.dev/@peasy-lib/peasy-ui";
    import { ListItem } from './list-item.js';

    class SFCApp {
      names = [{ name: 'World' }, { name: 'everyone' }];
    }

    UI.create(document.body, '#sfc-app', new SFCApp());
  </script>
```

#### HTML Single File Component

You can also create your single file components using the HTML format as well. The one benefit from this approach is that your template
html doesn't have to be a string literal anymore, and can be use any IDE syntax highlighting or autocomplete as a useful tool.

```html
<!-- list-item.html -->
<style>
  list-item {
    color: gold;
  }
</style>

<template id="list-item">
  <span>Hello, ${name}!</span>
</template>

<script type="module">
  import { UI } from "https://cdn.skypack.dev/@peasy-lib/peasy-ui";

  export class ListItem {
    static template = document.querySelector("#list-item"); // This can also be a string

    constructor(name) {
      this.name = name;
    }

    static create(state) {
      return new ListItem(state.name);
    }
  }
  UI.register("ListItem", ListItem); // Is necessary for HTML single file components
</script>
```

Since HTML imports aren't here (yet), Peasy UI uses the UI.import and UI.ready methods to support HTML single file components. From a
templating perspective, there's no difference between JavaScript and HTML single file components and the two types can co-exist and be
used in the same app. So the implementation details of the HTML component is:

```html
<!-- index.html -->
<head>
  <script type="module">
    import { UI } from "https://cdn.skypack.dev/@peasy-lib/peasy-ui";

    UI.initialize(); // UI.initialize is necessary with HTML components and needs to be in a head in top file
  </script>
</head>

<body>
  <template id="sfc-app">
    <div>
      <div pui="name <=* names"><list-item pui="ListItem === name"></list-item></div>
    </div>
  </template>

  <script type="module">
    import { UI } from "https://cdn.skypack.dev/@peasy-lib/peasy-ui";

    UI.import("./list-item.html");

    class SFCApp {
      names = [{ name: "World" }, { name: "everyone" }];
    }

    await UI.ready(); // Awaiting UI.ready is necessary with HTML imports
    UI.create(document.body, "#sfc-app", new SFCApp());
  </script>
</body>
```

Going forward, we will dig into a small demonstration app, but it will be using Typescript for its components, and is using Vite as a
build tool and bundler.

## Demo application

[Link to application github repo](https://github.com/jyoung4242/peasy-component-article)

[Link to demo application](https://peasy-component-article.vercel.app/)

<img width="100%" style="width:200px" src="https://i.giphy.com/media/v1.Y2lkPTc5MGI3NjExNnB2dTFzMHgycWYzNDU1MXN5cWR3eHlvaG9lb3RvMndzbnE0OGVscyZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/KHcaVxZX0u3tC0yoXU/giphy.gif">

The demo that's shown here is a very simple app to provide some context on how the Peasy-UI library's component features are utilized.

This demo features two component types, a Label and a Button. The button component is used twice, and is passed different parameters to
control not only its behavior but its aesthetic.

### Label Component

```ts
// ./src/components/label.ts

export type LabelState = {
  count: number;
  className: string;
};

export class Label {
  // HTML template
  // this is the HTML that get's rendered for the component - required for Peasy-UI components
  public static template = `
    <style>
      .label_Component {
          font-size: 3em;
          color: whitesmoke;
      }
    </style>
    <div class="label_Component \${className}">\${count}</div>
  `;

  // internal method that manipulates state
  increment = () => {
    this.count++;
  };

  // internal method that manipulates state
  decrement = () => {
    if (this.count > 0) this.count--;
  };

  // constructor that also defines the component state variables by the public keyword
  constructor(public count: number, public className: string) {}

  // static method that creates an instance of the class - required for Peasy-UI components
  static create(state: LabelState) {
    return new Label(state.count, state.className);
  }
}
```

The Label is a simple component that displays and controls the count value within its own DOM element. In the template literal you can
see that it owns its own styling, plus its own DOM element, which has two peasy bindings in it. First, the public `classname` is used
to add a secondary css class to a label if necessary, in this example it isn't used, but added to show that capability. The second
binding is the public `count` value which is bound to the component Div element.

There are two internal methods, increment() and decrement(), that are used to control the count state, and these provide external
access for other components to call to manipulate the count value. This removes the need to modify the count value directly, and let's
the component manage that.

Also note the create method, which is required functionality to be a Peasy-UI component. Also I define the incoming state, since I'm
using Typescript, to improve the devX on the parent side to esure that I'm using the component properly. This isn't required, but a
nice to have feature.

All aspects of the label's appearance and behaviors are captured in this one file.

### Button Component

```ts
/// ./src/components/button.ts
export type ButtonState = {
  buttonText: string;
  className: string;
  buttonClickHandler: (event: Event, model: any, elem: HTMLElement) => void;
};

export class Button {
  // HTML template
  // this is the HTML that get's rendered for the component - required for Peasy-UI components
  public static template = `
    <style>
      .button_Component {
          font-size: 1em;
          border-radius: 10px;
          border: 0px;
          width: 60px;
          height: 25px;
          margin: 5px;
          font-weight: bold;
          color: #111111;
          background-color: whitesmoke;
      }

      .button_Component:active {
        -webkit-box-shadow: inset 0px 0px 5px #c1c1c1;
        -moz-box-shadow: inset 0px 0px 5px #c1c1c1;
        box-shadow: inset 0px 0px 5px #c1c1c1;
        outline: none;
      }

      .up {
        color: whitesmoke;
        background-color: green;
      }

      .down {
        color: whitesmoke;
        background-color: red;
      }
      
    </style>
    <button class="button_Component \${className}"  \${click@=>buttonClickHandler}">\${buttonText}</button>
  `;

  // constructor that also defines the component state variables by the public keyword
  constructor(
    public buttonText: string,
    public className: string,
    public buttonClickHandler: (event: Event, model: any, elem: HTMLElement) => void
  ) {}

  // static method that creates an instance of the class - required for Peasy-UI components
  static create(state: ButtonState) {
    return new Button(state.buttonText, state.className, state.buttonClickHandler);
  }
}
```

Now the button component has a little more going on than the Label component. We have similar aspects of the Label component, such as
the create method, component level styling in the template literal, plus all the bindings that we have for a button. One of those
bindings is unique, in that the button event callback is passed into the component so that we can use this button for different
processes. This is using the event binding on the 'click' event on the DOM element. The other bindings are the button label and adding
the classname to control different styles.

## Demo Component Implementation

### Data Model

```ts
// ./src/ui.ts

/*
  Importing UI components here
*/

import { Button, ButtonState } from "./components/button";
import { Label, LabelState } from "./components/label";

/*
  Defining the data model which serves as Peasy-UI State
*/
export const model = {
  Label, // adding the component Classes into the data model
  Button,

  labelInstance: undefined as Label | undefined, // this is the assigned 'instance' of the Label component

  // This is the state 'props' that are passed into the Label component
  counterState: { count: 0, className: "" } as LabelState,

  // This is the state 'props' that are passed into the Button Components, one for each instance used
  upButtonState: {
    buttonText: "Up",
    className: "up",
    buttonClickHandler: () => {
      increment();
    },
  } as ButtonState,
  downButtonState: {
    buttonText: "Down",
    className: "down",
    buttonClickHandler: () => {
      decrement();
    },
  } as ButtonState,
};

...
```

So with any Peasy-UI project, we have a data model that represent the application state that Peasy uses. Let's go through this section
by section.

The first two things declared in the data model are the actual component classes. This provides Peasy-UI access to the class
components, and when a template binding calls these out, it uses the create method on each class to instantiate each component.

`labelInstance` is the actual instance of the Label class that is created, this gives us access to the two methods we will use to
control the count on the label. We will see later in the template section how the binding to this works. Its important to note, that i
call out `labelInstance` as either a Label | undefined, and this is because it IS undefined until the UIView.create().attached property
in the main.ts call resolves its promise. This is a means to appease Typescript type checking, but it is important to note and
understand that it is undefined until the View is attached. It is possible to not use the attached property, but there is a risk of
attempting to render something from the data model that hasn't been evaluated yet, and this pattern prevents that from occuring.

```ts
// main.ts
// Peasy-UI method for creating a View and attaching to DOM
await UI.create(document.body, model, template).attached;
```

Once the view is attached to the DOM, then all undefined values in the data model will be evaluated to their intended states.

`counterState`, `upButtonState`, and `downButtonState` are all objects that pass the default states into each component. Special note
here is that as a part of the Button states, we are passing the event callback as state into the components, so we can re-use the
components and control their behaviors.

### String Literal Template

```ts
// .src/ui.ts

/*
  HTML template which is string literal representing what gets rendered by Peasy-UI
*/

export const template = `
<div>
    <style>
        .container {
            position: fixed;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            width: 200px;
            height: 100px;
            padding: 10px;
            border: 2px solid whitesmoke;
            border-radius: 10px;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            gap: 10px;
        }

        .button_container {
            gap: 5px;
            display: flex;
            flex-direction: row;
        }
    </style>

    <div class="container">

        <div class="label_container">
          <!-- Using the Peasy-UI Component here for a Label-->
          <\${Label:labelInstance === counterState}>
        </div>
       
        <div class="button_container">
          <!-- Creating two instances of the Peasy-UI Components here for buttons-->
          <\${Button === upButtonState}>
          <\${Button === downButtonState}>
        </div>
    
    </div>
</div>`;
```

This is the parent string template that is provided into the UIView.Create() method for rendering. Here we can point out the bindings
for the Peasy-UI components.

Some of this is general boilerplate, such as all the CSS styling tags that are used for the parent container, we won't focus on that.
Also, that could be all used in an external .css file just fine and can be used to clean this stuff out, this is developer preference.
The only time one needs to include CSS inside the template is if there is a Peasy-UI binding on the actual CSS. I talk a lot more about
this in a previous article if you are interested in learning more about using Peasy with CSS:
[Using Peasy with CSS](https://dev.to/jyoung4242/controlling-css-with-peasy-ui-part-4-of-the-peasy-ui-series-209a)

Let us look at the two different types of Component bindings here. Let's take the Label binding first.

`<\${Label:labelInstance === counterState}>`

The three important ideas passed here are the class of Component: `Label`, the instance assignment `:labelInstance`, and the state
object provided: `counterState`. All three of these are used with the `===` binding tag. This essentially states that we are creating
an instance of the Label class, using the counterState object as a property to pass into the create method, and that the returning
instance of the class should be saved by reference in the labelInstance value. Since we are using the instance value, we can then have
access to all the internals of the class just like any other JS class can.

```
<\${Button === upButtonState}>
<\${Button === downButtonState}>
```

Now there are just two different aspects of the Button binding. First, we are using two instances of the binding, but are passing two
different state objects, this demonstrates the 're-usability' of the component. As a reminder, this even includes us passing the event
callback into the component so its behavior is provided to it.

Secondly, we are not in need of the instances of the component classes, so we are not binding the return value to anything.

## More information

More information can be found in the github repo for Peasy-Lib, you also will find all the other companion packages there too. Also,
Peasy has a Discord server where we hang out and discuss Peasy and help each other out.

The author's twitter: [Here](https://twitter.com/jyoung424242)

The author's itch: [Here](https://mookie4242.itch.io/)

Peasy-UI Github Repo: [Here](https://github.com/peasy-lib/peasy-lib/tree/main/packages/peasy-ui)

Peasy Discord Server: [Here](https://discord.gg/9VsQrVH94Z)

## Conclusion

In Summary, we looked at the different approaches that can be used to build Peasy-UI components, including single-file HTML components,
or JS/TS based compoenents. We looked at how you can use Peasy-UI components without a build step.

We also reviewed a demo application that demonstrates some of the component features and implementation details. This includes using
both the Class and Instances of a component, and using different state objects to re-use a component with different behaviors.

The big take aways from a component based architecture is modular design, cleaner code, and being able to reuse common components to
improve maintainability of your code base!
