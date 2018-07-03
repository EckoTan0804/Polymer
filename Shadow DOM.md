# Shadow DOM 





## Shadow DOM concepts

Shadow DOM is a new DOM feature that helps you build components. You can think of shadow DOM as a **scoped subtree** inside your element.



Consider a header component that includes a page title and a menu button: The DOM tree for this element might look like this:

```html
<my-header>
  <header>
    <h1>
    <button>
```

Shadow DOM lets you place the children in a scoped subtree, so **document-level CSS can't restyle the button by accident**, for example. This subtree is called a shadow tree.

```html
<my-header>
  #shadow-root
    <header>
      <h1>
      <button>
```

The *shadow root* is the top of the shadow tree. The element that the tree is attached to (`<my-header>`) is called the ***shadow host***. The host has a property called `shadowRoot` that refers to the shadow root. The shadow root has a `host` property that identifies its host element.



The shadow tree is **separate** from the element's `children`. You can think of this shadow tree as part of the component's **implementation**, which **outside elements don't need to know about.** The element's children are part of its public interface.

You can add a shadow tree to an element imperatively by calling `attachShadow`:

```javascript
var div = document.createElement('div');
var shadowRoot = div.attachShadow({mode: 'open'});
shadowRoot.innerHTML = '<h1>Hello Shadow DOM</h1>';
```



Polymer provides a declarative mechanism for adding a shadow tree using a [DOM template](https://www.polymer-project.org/2.0/docs/devguide/dom-template). When you provide a DOM template for an element, Polymer attaches a shadow root for each instance of the element and copies the template contents into the shadow tree.

```html
<dom-module id="my-header">
  <template>
    <style>...</style>
    <header>
      <h1>I'm a header</h1>
      <button>Menu</button>
    </header>
  </template>
</dom-module>
```

Note that the template includes a `<style>` element. **CSS placed in the shadow tree is scoped to the shadow tree, and won't leak out to other parts of your DOM.**

> ```
> The <style> element inside the <template> lets you define styles that are scoped to the shadow DOM,
> so they don't affect the rest of the document.
> ```



### Shadow DOM and composition

By default, if an element has shadow DOM, **the shadow tree is rendered instead of the element's children.** To allow children to render, you can add a `<slot>` element to your shadow tree. Think of the `<slot>` as a placeholder showing where child nodes will render. Consider the following shadow tree for `<my-header>`: