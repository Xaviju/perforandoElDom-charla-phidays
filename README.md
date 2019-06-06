# ShadowDOM piercing talk

«Perforando el Shadow DOM» Estructura y recursos para la charla de los ΦDays 2019

### Custom Elements
-------------------

The browser gives us an excellent tool for structuring web applications. It's called HTML. Great as HTML may be, its vocabulary and extensibility are limited.

With Custom Elements, web developers can create new HTML tags, beef-up existing HTML tags, or extend the components other developers have authored. The API is the foundation of web components. It brings a web standards-based way to create reusable components using nothing more than vanilla JS/HTML/CSS. The result is less code, modular code, and more reuse in our apps.

#### Extending standard elements

```javascript
class FancyButton extends HTMLButtonElement {
  constructor() {
    super();
    this.addEventListener('click', e => this.drawRipple(e.offsetX, e.offsetY));
  }

  drawRipple(x, y) {
    let div = document.createElement('div');
    div.classList.add('ripple');
    div.classList.add('run');
  }
}

customElements.define('fancy-button', FancyButton, {extends: 'button'});
```

#### Custom elements image

[Custom elements visualization](https://rangleio.ghost.io/content/images/2019/04/webcomponents2_desktop_16_9.png)

#### Best practices

- The name of a custom element must contain a dash `(-)`
- You can't register the same tag more than once.
- Custom elements cannot be self-closing.

### Shadow DOM
------------------

Over the years we've invented an exorbitant number of tools to circumvent the issues of global styles and HTML elements. From IDs to BEM.

An important aspect of web components is encapsulation — being able to keep the markup structure, style, and behavior hidden and separate from other code on the page so that different parts do not clash, and the code can be kept nice and clean. 

It introduces scoped styles to the web platform. Without tools or naming conventions, you can bundle CSS with markup, hide implementation details, and author self-contained components in vanilla JavaScript.


#### Shadow DOM image

[Shadow DOM visualization](https://dab1nmslvvntp.cloudfront.net/wp-content/uploads/2012/07/shadow-dom-render1.png)

#### Best practices

- Set a `:host` display style
- Set a `:host` display style for `hidden` attrs
- Use the lifecycle

## Shadow DOM Piercing methods

### HTML Properties
-------------------------
HTML attributes are the main method for passing data from a parent component to its children.

It's common for HTML properties to reflect their value back to the DOM as an HTML attribute. For example, when the values of hidden or id are changed in JS they are changed in HTML.

#### Why
Reflecting a property is useful anywhere you want to keep the element's DOM representation in sync with its JavaScript state. 

#### How

```html
<!-- Custom element  -->
<custom-element></custom-element>
```

```javascript
get disabled() {
  return this.hasAttribute('disabled');
}

set disabled(val) {
  // Reflect the value of `disabled` as an attribute.
  if (val) {
    this.setAttribute('disabled', '');
  } else {
    this.removeAttribute('disabled');
  }
  this.toggleDrawer();
}
```

Props:

    - Vue: https://vuejs.org/v2/guide/components.html#Passing-Data-to-Child-Components-with-Props
    - Angular: https://angular.io/guide/component-interaction#pass-data-from-parent-to-child-with-input-binding


#### When

Using this when altering the internal state of the component (only regarding logic)

### Slots
----------------------------
Slots are placeholders inside your component that users can fill with their own markup. By defining one or more slots, you invite outside markup to render in your component's shadow DOM. Essentially, you're saying "Render the user's markup over here".

#### Why

Elements are allowed to "cross" the shadow DOM boundary when a <slot> invites them in.

#### How

Inside the element shadow DOM

```html
#shadow-root
  <div id="tabs">
    <slot id="tabsSlot" name="title"></slot> <!-- named slot -->
  </div>
  <div id="panels">
    <slot id="panelsSlot"></slot>
  </div>
```

Outside the element shadow DOM

```html
<fancy-tabs>
  <button slot="title">Title</button>
  <button slot="title" selected>Title 2</button>
  <button slot="title">Title 3</button>
  <!-- Elements that have no slotted name attr will fit into the unnamed slot -->
  <section>content panel 1</section>
  <section>content panel 2</section>
  <section>content panel 3</section>
</fancy-tabs>
```

#### When

Alter HTML content from outside the component's shadow DOM

### ::shadow and /deep/
-----------------------

The original intent of `/deep/` was to provide a way to handle exceptional cases of when the scoping boundary is in the way of an occasional practical need to tweak styling from outside of the scope or globally.

It might make sense to use `/deep/` to define global rules that apply into the scoped styles or to tweak some style for some requirement.

#### How does it looks?

Piercing an elements DOM from outside to overwrite its styles

```css
custom-element /deep/ h3 {
  font-style: italic;
}
```


#### Why deprectaded?

Instead, this selector was used to alter most of the styles set by the component developer duplicating styles for ever UI widget. If the number of required rules to grow too large because we need to overwrite too many parts, what's the benefit of scoping?

### Custom properties
-------------------------

#### Why
#### How
#### When

### :host() and :host-context() selectors
-------------------------------------------

#### Why
#### How
#### When

### :part() and :theme() psudo-selectors
-------------------------------------------

#### Why
#### How
#### When

## Summary
-----------------

### Best practices
### Conclusion

## Resources

- [Fundamentals: web components](https://developers.google.com/web/fundamentals/web-components/shadowdom)
- [MDN Shadow DOM](https://developer.mozilla.org/en-US/docs/Web/Web_Components/Using_shadow_DOM)
- [Shadow piercing Combinators in the Wild](https://github.com/w3c/webcomponents/wiki/Shadow-piercing-Combinators-in-the-Wild)
- [What's the substitute for ::shadow and /deep/?](https://stackoverflow.com/questions/35741722/whats-the-substitute-for-shadow-and-deep)
- [::part and ::theme, an ::explainer](https://meowni.ca/posts/part-theme-explainer/)
- [What is the different between :host ,:host() ,:host-context selectors](https://stackoverflow.com/questions/51326676/what-is-the-different-between-host-host-host-context-selectors)
- [:host-context on MDN](https://developer.mozilla.org/en-US/docs/Web/CSS/:host-context())
