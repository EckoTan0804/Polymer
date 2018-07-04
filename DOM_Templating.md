# 	DOM Templating

Many elements use a subtree of DOM elements to implement their features. DOM templating provides an easy way to create a DOM subtree for your element.

By default, adding a DOM template to an element causes Polymer to create a shadow root for the element and clone the template into the shadow tree.

The DOM template is also processed to enable data binding and declarative event handlers.



## Specify a DOM template

Polymer provides three basic ways to specify a DOM template:

- [Use the `` element](https://www.polymer-project.org/2.0/docs/devguide/dom-template#dommodule). This allows you to specify the template entirely in markup, which is most efficient for an element defined in an HTML import.
- [Define a template property on the constructor](https://www.polymer-project.org/2.0/docs/devguide/dom-template#templateobject).
- [Inherit a template from another Polymer element](https://www.polymer-project.org/2.0/docs/devguide/dom-template#inherit).

### Use the dom-module element

To specify an element's DOM template using a `<dom-module>`:

1. Create a `<dom-module>`element with an `id` attribute that matches the element's name.
2. Create a `<template>` element inside the `<dom-module>`.
3. Give the element a static `is` getter that matches the element's name. Polymer uses this to retrieve the `<dom-module>` for the element.

Polymer clones this template's contents into the element's shadow DOM.

Specify an element's DOM template using a dom-module:

```html
<dom-module id="x-foo">

  <template>I am x-foo!</template>

  <script>
    class XFoo extends Polymer.Element {
      static get is() { return  'x-foo' }
    }
    customElements.define(XFoo.is, XFoo);
  </script>

</dom-module>
```

### Define a template property on the constructor

As an alternative to specifying the element's template in markup, you can define a `template` property on the element's constructor. To create a static template getter, override Polymer's default `template` getter. This getter is called *once*, when the first instance of the element is upgraded.

The template getter can return either a string or an instance of `HTMLTemplateElement`. Support for the string return value is deprecated in Polymer 2.4, and will be removed in 3.0.

Polymer 2.4+ provides a helper function (`Polymer.html`) to generate an `HTMLTemplateElement` instance from a JavaScript template literal.

Use Polymer.html to convert a JavaScript template literal to an HTMLTemplateElement:

```javascript
class MyElement extends Polymer.Element {

  static get template() {
    return Polymer.html`<style>:host { color: blue; }</style>
       <h2>String template</h2>
       <div>I've got a string template!</div>`;
  }
}
customElements.define('my-element', MyElement);
```



An concrete example: 

>  In this example I have written 2 methods for templating the DOM:
>
> 1. using the `<template>` element in `<dom-module>` (Line 7 ~ 28)
> 2. Define a template property on the constructor (Line 34 ~ 57)
>
> Just choose one of this two methods you preferred.

~~~javascript
<link rel="import" href="../polymer/polymer-element.html">
<link rel="import" href="../iron-icon/iron-icon.html">

<dom-module id = "hello-world">
  
  
  <!--1. Use the dom-module element to specify a DOM template-->
  <template>
    
    <!--define the shadow DOM-->
    <style>
      
      :host {
        display: inline-block;
      }
      
      iron-icon {
        fill: rgba(0, 0, 0, 0);
        stroke: currentColor;
      }
      
      :host([pressed]) iron-icon {
        fill: red;
      }
    </style>
    
    <iron-icon icon="[[toggleIcon]]" on-click="toggle"></iron-icon>
  </template>
  
  <script>
    
    class HelloWorld extends Polymer.Element {

        // 2. Define a template property on the constructor
        static get template() {
            return Polymer.html
            `
                  <!--define the shadow DOM-->
              <style>
                
                :host {
                  display: inline-block;
                }
                
                iron-icon {
                  fill: rgba(0, 0, 0, 0);
                  stroke: currentColor;
                }
                
                :host([pressed]) iron-icon {
                  fill: red;
                }
              </style>
    
               <iron-icon icon="[[toggleIcon]]" on-click="toggle"></iron-icon>
            `;
        }
      
        static get is() {
            return "hello-world";
        }
        
        static get properties() {
            return {
                pressed: {
                    type: Boolean,
                    notify: true,
                    reflectToAttribute: true,
                    value: false
                },
            
                toggleIcon: {
                    type: String
                }
            }
        }
        
        constructor() {
            super();
        }
        
        toggle() {
            this.pressed = !this.pressed;
        }
    }
    
    customElements.define(HelloWorld.is, HelloWorld);
  </script>
  
</dom-module>

~~~

> **When using a static template getter, the element doesn't need to provide an is getter.** However, the tag name still needs to be passed as the first argument to `customElements.define`, as in the example above.



### Inherit a template from another Polymer element

An element that extends another Polymer element can inherit its template. If the element doesn't provide its own DOM template (using either a `<dom-module>` or a static template object), Polymer uses the same template as the superclass, if any.

#### Polymer 2.x: Inherit a base class template without modifying it

To inherit a base class template without modifying it, **do not** supply a template definition in the child class declaration.

Base class definition:

```html
<dom-module id="base-class">

  <template>This content has been inherited from BaseClass.</template>

  <script>
    class BaseClass extends Polymer.Element {
      static get is() { return  'base-class' }
    }
    customElements.define(BaseClass.is, BaseClass);
  </script>

</dom-module>
```

Child class definition:

```html
<dom-module id="child-class">

  <script>
    class ChildClass extends BaseClass {
      static get is() { return  'child-class' }
    }
    customElements.define(ChildClass.is, ChildClass);
  </script>

</dom-module>
```



#### Polymer 2.x: Override a base class template in a child class

To **override** a base class's template definition, **supply your own template for your child class.**

Base class definition:

```html
<dom-module id="base-class">

  <template>This is BaseClass's template.</template>

  <script>
    class BaseClass extends Polymer.Element {
      static get is() { return  'base-class' }
    }
    customElements.define(BaseClass.is, BaseClass);
  </script>

</dom-module>
```

Child class definition:

```html
<dom-module id="child-class">

  <template>Base class template has been overridden. Hello from ChildClass!</template>

  <script>
    class ChildClass extends BaseClass {
      static get is() { return 'child-class' }
    }
    customElements.define(ChildClass.is, ChildClass);
  </script>
</dom-module>
```



#### Polymer 2.x: Modify a copy of a superclass template

You can modify a superclass template by defining a `template` getter that returns a modified template element. If you're going to modify the superclass template, there are a couple of important rules:

- **Don't modify the superclass template in place; make a copy before modifying.**
- **If you're doing anything expensive, like copying or modifying an existing template, you should memoize the modified template so you don't have to regenerate it when the getter is called.**

The following example shows a simple modification based on a parent template:

```javascript
(function() {
  let memoizedTemplate;

  class MyExtension extends MySuperClass {
    static get template() {
      if (!memoizedTemplate) {
        // create a clone of superclass template (`true` = "deep" clone)
        memoizedTemplate = MySuperClass.template.cloneNode(true);
        // add a node to the template.
        let div = document.createElement('div');
        div.textContent = 'Hello from an extended template.'
        memoizedTemplate.content.appendChild(div);
      }
      return memoizedTemplate;
    }
  }

})();
```

The following example shows how an element could wrap a superclass template with its own template:

```javascript
<dom-module id="my-ext">
    <template>
      <h2>Extended template</h2>
      <!-- superclass template will go here -->
      <div id="footer">
        Extended template footer.
      </div>
    </template>
  <script>
    (function() {
      let memoizedTemplate;

      class MyExtension extends MySuperClass {

        static get is() { return 'my-ext'}
        static get template() {
          if (!memoizedTemplate) {
            // Retrieve this element's dom-module template
            memoizedTemplate = Polymer.DomModule.import(this.is, 'template');

            // Clone the contents of the superclass template
            let superTemplateContents = document.importNode(MySuperClass.template.content, true);

            // Insert the superclass contents
            let footer = memoizedTemplate.content.querySelector('#footer');
            memoizedTemplate.content.insertBefore(superTemplateContents, footer);
          }
          return memoizedTemplate;
        }
      }
      customElements.define(MyExtension.is, MyExtension);
    })();
  </script>
</dom-module>
```

#### Polymer 2.4+: Extend a base class template in a child class

Polymer 2.4+ uses the functionality of embedded expressions in JavaScript template literals to provide convenient ways to [extend inherited templates](https://www.polymer-project.org/2.0/docs/devguide/dom-template#inherited-templates).

[Read more about template literals on MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals).

To extend a base class template with Polymer 2.4+, include the base class template in your child class template literal with the expression `${super.template}`. You will also need to tag the template literal with the `Polymer.html` function:

```javascript
static get template() {
  return Polymer.html`
    <p>${super.template}</p>
  `;
}
```

- The expression `${super.template}` will be interpolated and included in the template of the child class.
- Because the `template` getter is static, `${super.template}` accesses the `template` property on the superclass constructor.
- The `Polymer.html` function checks each expression value. If the expression is an`HTMLTemplateElement`, the `innerHTML` of the template is interpolated.
- To protect against XSS vulnerabilities, `Polymer.html` only interpolates `HTMLTemplateElement` values. Non-template values will throw an error.

It's very simple to combine existing templates:

Base class definition:

```javascript
const html = Polymer.html;

class BaseElement extends Polymer.Element {
  static get template() {
    return html`
      <p>This content has been inherited from BaseElement.</p>`
  }
}

customElements.define('base-element', BaseElement);
```

Child class definition:

```javascript
const html = Polymer.html;

class ChildElement extends BaseElement {
  static get template() {
    return html`
      <p>This content is in ChildElement.</p>
      <p>${super.template}</p>
      <p>Hello again from ChildElement.</p>
      `;
  }
}

customElements.define('child-element', ChildElement);
```



#### Polymer 2.4+: Provide template extension points

Polymer 2.4 makes it easy to provide template extension points in a base class, which a child class can then optionally override. You can provide template extension points by composing your base class template literal with expressions-for example, `${this.headerTemplate}`. You will also need to tag the template literals with the `Polymer.html` function:

```javascript
...
//create a base class template with extension points:
static get template() {
  return Polymer.html`
    <div>${this.headerTemplate}</div>
    <p>Hello this is some content</p>
    <div>${this.footerTemplate}</div>
  `;
}
//supply the default content for the expressions:
static get headerTemplate() { return Polymer.html`...` }
static get footerTemplate() { return Polymer.html`...` }
...
```

Your child class may or may not need to override this content. Here's an example in which the child class overrides the base class content:

Base class definition:

```javascript
const html = Polymer.html;

class BaseClass extends Polymer.Element {
  static get template() {
    return html`
      <div>${this.headerTemplate}</div>
      <p>Hello this is some content</p>
      <div>${this.footerTemplate}</div>
    `;
  }
  static get headerTemplate() { return html`<h1>BaseClass: Header</h1>` }
  static get footerTemplate() { return html`<h1>BaseClass: Footer</h1>` }
}
```

Child class definition:

```javascript
const html = Polymer.html;

class ChildClass extends BaseClass {
    static get template() {
      return html`
        <div class="frame">${super.template}</div>
      `;
    }
    static get headerTemplate() { return html`<h2>ChildClass: Header</h2>` }
    static get footerTemplate() { return html`<h2>ChildClass: Footer</h2>` }
  }
```



 

### Elements with no shadow DOM

To create an element with no shadow DOM, don't specify a DOM template (either using a `<dom-module>` or by overriding the `template` getter), then no shadow root is created for the element.

If the element is extending another element that has a DOM template and you don't want a DOM template, define a `template` getter that returns a falsy value.



### URLs in templates

A relative URL in a template may need to be relative to an application or to a specific component. For example, if a component includes images alongside an HTML import that defines an element, the image URL needs to be resolved relative to the import. However, an application-specific element may need to include links to URLs relative to the main document.

By default, Polymer **does not modify URLs in templates**, so all relative URLs are treated as relative to the main document URL. This is because when the template content is cloned and added to the main document, the browser evaluates the URLs relative to the document (not to the original location of the template).

To ensure URLs resolve properly, Polymer provides two properties that can be used in data bindings:

| Property     | Description                                                  |
| ------------ | ------------------------------------------------------------ |
| `importPath` | A static getter on the element class that defaults to the element HTML import document URL and is overridable. It may be useful to override `importPath` when an element's template is not retrieved from a `<dom-module>` or the element is not defined using an HTML import. |
| `rootPath`   | An instance property set to the value of `Polymer.rootPath` which is globally settable and defaults to the main document URL. It may be useful to set `Polymer.rootPath` to provide a stable application mount path when using client side routing. |

Relative URLs in styles are automatically re-written to be relative to the `importPath` property. Any URLs outside of a `<style>` element should be bound using `importPath` or `rootPath` where appropriate. For example:

```
<img src$="[[importPath]]checked.jpg">
<a href$="[[rootPath]]users/profile">View profile</a>
```

The `importPath` and `rootPath` properties are also supported in Polymer 1.9+, so they can be used by hybrid elements.



## Static node map

Polymer builds a static map of node IDs when the element initializes its DOM template, to provide convenient access to frequently used nodes without the need to query for them manually. **Any node specified in the element's template with an `id` is stored on the `this.$` hash by `id`.**

**The `this.$` hash is created when the shadow DOM is initialized. In the `ready` callback, you must call `super.ready()` before accessing `this.$`.**

> **Note:** Nodes created dynamically using data binding (including those in `dom-repeat` and `dom-if`templates) are *not* added to the `this.$` hash. The hash includes only *statically* created local DOM nodes (that is, the nodes defined in the element's outermost template).

Example:

```html
<dom-module id="x-custom">

  <template>
    Hello World from <span id="name"></span>!
  </template>

  <script>
    class MyElement extends Polymer.Element {
      static get is() { return  'x-custom' }
      ready() {
        super.ready();
        this.$.name.textContent = this.tagName;
      }
    }
  </script>

</dom-module>
```

For locating dynamically-created nodes in your element's shadow DOM, use the standard DOM `querySelector` method.

`this.shadowRoot.querySelector(selector)`



## Remove empty text nodes

Add the `strip-whitespace` boolean attribute to a template to remove any **empty** text nodes from the template's contents. This can result in a minor performance improvement.

**What's an empty node?** `strip-whitespace` removes only text nodes that occur between elements in the template and are ***empty* (that is, they only contain whitespace characters)**. These nodes are created when two elements in the template are separated by whitespace (such as spaces or line breaks). It doesn't remove any whitespace from inside elements.

**With** empty text nodes:

```javascript
<dom-module id="has-whitespace">
  <template>
    <div>Some Text</div>
    <div>More Text</div>
  </template>
  <script>
    class HasWhitespace extends Polymer.Element {
      static get is() { return  'has-whitespace' }
      ready() {
        super.ready();
        console.log(this.shadowRoot.childNodes.length); // 5
      }
    }
    customElements.define(HasWhitespace.is, HasWhitespace);
  </script>
</dom-module>
```

There are five nodes in this element's shadow tree because of the whitespace surrounding the `<div>` elements. The five child nodes are:

text node `<div>Some Text</div>` text node `<div>More Text</div>` text node

**Without** empty text nodes:

```javascript
<dom-module id="no-whitespace">
  <template strip-whitespace>
    <div>Some Text</div>
    <div>More Text</div>
  </template>
  <script>
    class NoWhitespace extends Polymer.Element {
      static get is() { return  'no-whitespace' }
      ready() {
        super.ready();
        console.log(this.shadowRoot.childNodes.length); // 2
      }
    }
    customElements.define(NoWhitespace.is, NoWhitespace);
  </script>
</dom-module>
```

Here, the shadow tree contains only the two `<div>` nodes:

`<div>Some Text</div><div>More Text</div>`

Note that the whitespace *inside* the `<div>` elements isn't affected.