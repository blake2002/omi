## Components

[Omi](https://github.com/AlloyTeam/omi) is based entirely on component architecture, which allows developers to build web applications like building blocks. Everything is components, components can be nested to create new components.

![Omi Components System](http://images2015.cnblogs.com/blog/105416/201702/105416-20170210093427338-1536910080.png)

### Simple Components

Let's explore a simple Todo example to learn the components system in Omi.

```js
class Todo extends Omi.Component {
    constructor(data) {
        super(data);
    }
    
    add (evt) {
        evt.preventDefault();
        this.data.items.push(this.data.text);
        this.data.text = '';
        this.update();
    }

    style () {
        return `
        h3 { color:red; }
        button{ color:green;}
        `;
    }

    handleChange(target){
        this.data.text = target.value;
    }

    render () {
        return `<div>
                    <h3>TODO</h3>
                    <ul> {{#items}} <li>{{.}}</li> {{/items}}</ul>
                    <form onsubmit="add(event)" >
                        <input type="text" onchange="handleChange(this)"  value="{{text}}"  />
                        <button>Add #{{items.length}}</button>
                    </form>
                </div>`;
    }
}

Omi.render(new Todo({ items: [] ,text : '' }),"body");
```

The HTML generated by the component will eventually be inserted into the body. The above example shows some of the features of Omi:

- Data flow: `data` in `new Todo(data,..)` can be used directly in the template in render method.
- Partial CSS: `h3` in `style()` only effect inside of render. It'll never pollute `h3`  outside of this component. The same rule applies to `button`.
- Declarative event binding: `onchange` will call `handleChange` that inside of the component. `this` refers to the current DOM element, `event` refers to the current DOM Event Object.
- You need to manually call the `this.update()` method to update the component

It is important to note that, for more freedom and flexibility, Omi does not automatically update DOM while data changes. Developers need to call the `update` method manually.

You can also use [oba] (https://github.com/dntzhang/oba) or mobx to implement automatic updates.

<a href="http://alloyteam.github.io/omi/website/redirect.html?type=todo" target="_blank">Click me for the live demo</a>

### Component Nesting

It's ok to not use nesting component if your page is super simple. However, for most of webpages and web applications, it is a necessary to define the nesting Components to implement complex features.

For instance, we can extract a `List` component form the Todo example. This brings maintainable, scalable and reuseable to our project:

```js
class List extends Omi.Component {
    constructor(data) {
        super(data);
    }

    render () {
        return `<ul> {{#items}} <li>{{.}}</li> {{/items}}</ul>`;
    }
}
```

Then how to use this `List`? We need to use `Omi.makeHTML` to make the `List` to a tag which can be used in render method:

```js
import List from './list.js';

Omi.makeHTML('List', List);

class Todo extends Omi.Component {
    constructor(data) {
        super(data);
        this.data.length = this.data.items.length;
        this.listData = { items : this.data.items };
    }

    add (evt) {
        evt.preventDefault();
        this.list.data.items.push(this.data.text);
        this.data.length = this.list.data.items.length;
        this.data.text = '';
        this.update();
    }

    style () {
        return `
        h3 { color:red; }
        button{ color:green;}
        `;
    }

    handleChange(target){
        this.data.text = target.value;
    }

    render () {
        return `<div>
                    <h3>TODO</h3>
                    <List name="list" data="listData" />
                    <form onsubmit="add(event)" >
                        <input type="text" onchange="handleChange(this)"  value="{{text}}"  />
                        <button>Add #{{length}}</button>
                    </form>
                </div>`;
    }
}
```

- In line 3, we use `makeHTML` to make the component to a tag which can be used in render method. Of course, `Omi.makeHTML('List', List);` can also be written in the end of List component.
- In line 9, the parent component defines the 'listData' property
- In line 34, we use List component in the render method. `name` attribute allows us easily find the instance of the component by using `this`.`data="listData"` attribute allows us easily pass `this.listData`  to the sub component from parent component.

It should be noted that the `data` passed from `data="listData"` is cloned to the subcomponents by Object.assign(shallow  copy) , which means if we want to change the DOM, we recommend  that first update the `data` of the instance of subcomponent(not the parent component's `listData` ) and secondly call the `update` method.

In fact there are 4 way to communicate between components, it'll be explained later.

<a href="http://alloyteam.github.io/omi/website/redirect.html?type=todo_nest" target="_blank">Click me for the live demo</a>
