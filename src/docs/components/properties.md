---
title: Properties
description: Properties
url: /docs/properties
contributors:
  - jthoms1
  - rwaskiewicz
---

# Prop Decorator

Props are custom attributes/properties exposed publicly on an HTML element. They allow developers to pass data into a
component to use. Props are declared on a component using the `@Prop()` decorator, like so:

```tsx
// First, we import Prop from '@stencil/core'
import { Component, Prop } from '@stencil/core';

@Component({
    tag: 'todo-list',
})
export class TodoList {
    // Second, we decorate a class member with @Prop()
    @Prop() color: string;
    
    render() {
        return <div>The color is {this.color}</div>
    }
}
```

In the example above, `@Prop()` is placed before (decorates) the `color` class member. By adding `@Prop()`, we expose
`color` as an attribute on the element, which can be set wherever the component is used:

```html
<!-- Here we use the component in an HTML file -->
<todo-list color="purple"></todo-list>
```
```tsx
{/* Here we use the component in a TSX file */}
<todo-list color={"purple"}></todo-list>
```

In fact, in this example the `TodoList` component is used exactly the same in HTML and TSX! However, in some cases, the
way props are used differs slightly between HTML and TSX.

## Variable Casing

In the JavaScript ecosystem, it's common to use 'camelCase' when defining variable names. The example component below
includes two class members that are camelCase - `thingToDo` & `isSelected`.

```tsx
import { Component, Prop } from '@stencil/core';

@Component({
    tag: 'todo-list-item',
})
export class ToDoListItem {
  @Prop() thingToDo: string;
  @Prop() isSelected: boolean;
}
```

Both `thingToDo` and `isSelected` are props, because they are decorated with the `@Prop()` decorator. Their usage in
HTML and TSX is nearly identical. However, in HTML, the attribute must use 'dash-case' like so:

```html
<todo-list-item thing-to-do="Learn Stencil" is-selected="true"></todo-list-item>
```

Where in TSX, you set an attribute using camelCase:

```tsx
<todo-list-item thingToDo="Learn Stencil" isSelected={true}></todo-list-item>
```

One thing to note is that in the TSX example above, `isSelected` has a value of `true` that is surrounded by curly
braces, where the HTML version uses double quotes. In the next section, we'll explore this further.

## Types

TODO
TODO
TODO
TODO

## IDK YET
They can also be accessed via JS from the element.

```tsx
const todoListElement = document.querySelector('todo-list');
console.log(todoListElement.myHttpService); // MyHttpService
console.log(todoListElement.color); // blue
```


Within the `TodoList` class, the Props are accessed via the `this` operator.

```tsx
logColor() {
  console.log(this.color)
}
```


Child components should not know about or reference parent components, so Props should be used to pass data down from 
the parent to the child. 

Props can be a `number`, `string`, `boolean`, or even an `Object` or `Array`. By default, when a member 
decorated with a `@Prop()` decorator is set, the component will efficiently rerender.

```tsx
// TodoList.tsx
import { Prop } from '@stencil/core';
import { MyHttpService } from '../some/local/directory/MyHttpService';
...
export class TodoList {
  @Prop() color: string;
  @Prop() favoriteNumber: number;
  @Prop() isSelected: boolean;
  @Prop() myHttpService: MyHttpService;
}
```

When using user-defined types like `MyHttpService`, the type must be exported using the `export` keyword. Above,
`MyHttpService` is imported from `'../some/local/directory/MyHttpService'`, and must be exported from that file.

If `MyHttpService` were defined in `TodoList.tsx`, the `export` keyword would still be required, as Stencil needs to
know what type `myHttpService` is when passing an instance of `MyHttpService` to `TodoList` from a parent component.

```tsx
// TodoList.tsx
import { Prop } from '@stencil/core';
...
// the export keyword is still required here, so that a parent component knows what a `MyHttpService` is
export type MyHttpService = {
  // type definition goes here  
};
...
export class TodoList {
  @Prop() color: string;
  @Prop() favoriteNumber: number;
  @Prop() isSelected: boolean;
  @Prop() myHttpService: MyHttpService;
}
```

## Prop options

The `@Prop(opts?: PropOptions)` decorator accepts an optional argument to specify certain options, such as the 
`mutability`, the name of the DOM attribute or if the value of the property should or shouldn't be reflected into the 
DOM.

```tsx
export interface PropOptions {
  attribute?: string;
  mutable?: boolean;
  reflect?: boolean;
}
```

### Prop mutability

It's important to know, that a Prop is _by default_ immutable from inside the component logic. Once a value is set by a user, the component cannot update it internally.

However, it's possible to explicitly allow a Prop to be mutated from inside the component, by declaring it as **mutable**, as in the example below:

```tsx
import { Prop } from '@stencil/core';

...
export class NameElement {

  @Prop({ mutable: true }) name: string = 'Stencil';

  componentDidLoad() {
    this.name = 'Stencil 0.7.0';
  }
}
```

### Attribute Name

Properties and component attributes are strongly connected but not necessarily the same thing. While attributes are a HTML concept, properties are a JS one inherent from Object-Oriented Programming.

In Stencil, the `@Prop()` decorator applied to a **property** will instruct the Stencil compiler to also listen for changes in a DOM attribute.

Usually the name of a property is the same as the attribute, but this is not always the case. Take the following component as example:

```tsx
import { Component, Prop } from '@stencil/core';

@Component({ tag: 'my-cmp' })
class Component {
  @Prop() value: string;
  @Prop() isValid: boolean;
  @Prop() controller: MyController;
}
```

This component has **3 properties**, but the compiler will create **only 2** attributes: `value` and `is-valid`.

```markup
<my-cmp value="Hello" is-valid></my-cmp>
```

Notice that the `controller` type is not a primitive, since DOM attributes can ONLY be strings, it does not make sense to have an associated DOM attribute called "controller".

At the same time, the `isValid` property follows a _camelCase_ naming, but attributes are case-insensitive, so the attribute name will be `is-valid` by default.

Fortunately, this "default" behaviour can be changed using the `attribute` option of the `@Prop()` decorator:


```tsx
import { Component, Prop } from '@stencil/core';

@Component({ tag: 'my-cmp' })
class Component {
  @Prop() value: string;
  @Prop({ attribute: 'valid' }) isValid: boolean;
  @Prop({ attribute: 'controller' }) controller: MyController;
}
```

By using this option, we are being explicit about which properties have an associated DOM attribute and the name of it.


### Reflect Properties Values to Attributes

In some cases it may be useful to keep a Prop in sync with an attribute. In this case you can set the `reflect` option in the `@Prop()` decorator to `true`, since it defaults to `false`:

```tsx
@Prop({
  reflect: true
})
```

When a "prop" is set to "reflect", it means that their value will be rendered in the DOM as an HTML attribute:

Take the following component as example:

```tsx
@Component({ tag: 'my-cmp' })
class Cmp {
  @Prop({ reflect: true }) message = 'Hello';
  @Prop({ reflect: false }) value = 'The meaning of life...';
  @Prop({ reflect: true }) number = 42;
}
```

When rendered in the DOM, it will look like:

```markup
<my-cmp message="Hello" number="42"></my-cmp>
```
Notice that properties set to "reflect" (true) render as attributes, and properties not set to "reflect" do not.

While the properties not set to "reflect", such as 'value', are not rendered as attributes, it does not mean it's not there - the `value` property still contains the `The meaning of life...` value as assigned:

```tsx
const cmp = document.querySelector('my-cmp');
console.log(cmp.value); // it prints 'The meaning of life...'
```

## Prop default values and validation

Setting a default value on a Prop:

```tsx
import { Prop } from '@stencil/core';

...
export class NameElement {
  @Prop() name: string = 'Stencil';
}
```

To do validation of a Prop, you can use the [@Watch()](reactive-data/#watch-decorator) decorator:

```tsx
import { Prop, Watch } from '@stencil/core';

...
export class TodoList {
  @Prop() name: string = 'Stencil';

  @Watch('name')
  validateName(newValue: string, oldValue: string) {
    const isBlank = typeof newValue !== 'string' || newValue === '';
    const has2chars = typeof newValue === 'string' && newValue.length >= 2;
    if (isBlank) { throw new Error('name: required') };
    if (!has2chars) { throw new Error('name: has2chars') };
  }
}
```

## Property Values

### Boolean Properties

A property on a Stencil component that has a type of `boolean` may be declared as:
```tsx
@Component({
    tag: 'todo-list-item',
})
export class TodoListItem {
    @Prop() isComplete: boolean;
    ...
}
```
```html
<todo-list-item is-complete></todo-list-item>
```
```html
<todo-list-item is-complete="true"></todo-list-item>
```

```html
<todo-list-item is-complete="false"></todo-list-item>
```
TODO: Cover the cases like '"0"', '"""', '"False"'
```html
<todo-list-item></todo-list-item>
```
HTML:
```html
<todo-list-item is-complete="true"></todo-list-item>
<todo-list-item is-complete="false"></todo-list-item>
<todo-list-item is-complete="0"></todo-list-item>
```
TS:
```tsx
<todo-list-item is-complete=true></todo-list-item>
<todo-list-item is-complete="false"></todo-list-item>
<todo-list-item is-complete=false></todo-list-item>
<todo-list-item is-complete="0"></todo-list-item>
<todo-list-item is-complete={0}></todo-list-item>
<todo-list-item is-complete=undefined></todo-list-item>
<todo-list-item is-complete=null></todo-list-item>
<todo-list-item is-complete=""></todo-list-item>
```


While `propValue` can reasonably be expected to be of `any` type as far as TypeScript is concerned, it is _not_
safe to assume that the type of `propValue` (I.E. `typeof propValue`) matches the type derived from `propType`. This
function provides the capability to parse/coerce a property's value to potentially any other JavaScript type.

Property values represented in TSX preserve their type information. Below, the BigInt 0 is passed to a component.
This `propValue` will preserve its type information (`typeof propValue === bigint`). Note that is based on the type
of the value being passed in, not the type declared of the class member decorated with `@prop`.
```tsx
<my-cmp prop-val={0n}></my-cmp>
```

html is always a string via `typeof`, case sensitive (`False`)