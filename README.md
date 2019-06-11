# Piercing the DOM | Kaleidos PhiDays talk

> «Perforando el Shadow DOM» Estructura y recursos para la charla de los ΦDays 2019

**Warning: This is a work in progress**

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
  
> **WARNING** How to syle a slot?

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

The `<slot>` elements create a flexible template that can then be used to populate the shadow DOM of a web component.

#### :slotted psudo-selector (TODO)
The ::slotted() CSS pseudo-element represents any element that has been placed into a slot inside an HTML template

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
Custom properties (sometimes referred to as CSS variables or cascading variables) are entities defined by CSS authors that contain specific values to be reused throughout a document. They are set using custom property notation (e.g., --main-color: black;) and are accessed using the var() function (e.g., color: var(--main-color);).

#### Why
Users can tweak internal styles if the component's author provides styling hooks using CSS custom properties. Conceptually, the idea is similar to `<slot>`. You create "style placeholders" for users to override.

#### How
Your component can declare custom properties and a fallback, and the CSS custom properties penetrate the Shadow DOM! This means than Custom Properties can be defined outside the shadow Dom but as long as the component expects it and its defined, the web component will receive the value.

```css
<style>
  fancy-tabs {
    --fancy-tabs-bg: black;
  }
</style>
<fancy-tabs >...</fancy-tabs>
```

```css
:host([background]) {
  background: var(--fancy-tabs-bg, #9E9E9E);
}

```

#### When

This is very useful for theming, specially for design systems or similar cases where the font, spacing, weights, colors and so on is usually set up and shared.

### :host
-------------------------------------------
The newly created html element can be styled too. There is a new selector inside the scoped CSS that allow authors to style how the new element will be represented, and not something inside its template.

#### Why
By default all new HTML elements created by the authors will be `display: inline` but we can author how the element will be represented in its position. This is nos strictly crossing the shadow DOM, but affects the convention of the inside and outside

#### How

Applying styles to the container is simple:

```css
host {
  display: block; /* by default, custom elements are display: inline */
  contain: content; /* CSS containment FTW. */
}
```

#### When
Is recommended that every html element should contain the HTML required to display itself regardless of its context.
Is a common mistake to style the element from the parent, and the fact is that parent styles override host styles (makes sense) but will require more lines of CSS and won't be reusable. 

### :host(selector)
-------------------------------------------
The HTML elements can style its host container and it behaviours in its context. By default all new HTML elements created by the authors will be `display: inline` but we can author how the element will be represented in its position. This is nos strictly crossing the shadow DOM, but affects the convention of the inside and outside

#### Why
Its better to set the style of the host container from the inside of the component that from the outside, avoinding code duplicity and manipulation from other context. In case of emergency, CSS from the parent has more priority that host styles.

#### How

Applying styles to the container is simple:

```css
host {
  display: block; /* by default, custom elements are display: inline */
  contain: content; /* CSS containment FTW. */
}
```

#### When
Is recommended that every html element should contain the HTML required to display itself regardless of its context.
Is a common mistake to style the element from the parent, and the fact is that parent styles override host styles (makes sense) but will require more lines of CSS and won't be reusable.

:host-context() selectors
-------------------------------------------
Similar to the host selector, we might want to style a component depending its hierarchy. Similar to the previous selector, but in this case the selector will be targeted when any of the parents of this component has the defined selector

#### Why
It can happen that depending on the position of the component we wnat to style it differently but keep the same behaviour. Instead of providing a uber complicated API, we can choose just to add a `host-context` selector to ensure that the API interface does not change but depending on its context, the UI dos change. 

#### How
Its simple, similar to the host selector

```css
host-context(selector) {
  display: block; /* by default, custom elements are display: inline */
  contain: content; /* CSS containment FTW. */
}
```

#### When
Imagine we have two instances of a component with a button. The first one is a cgild of the `main` element and the second one is child of the `aside` HTML element. We might want to display a slightly more visual button for the first one, to make it stand out. Using host-context this will not require more code in the logic of the component.


### :part() pseudo-selector
-------------------------------------------
This pseudo selector will allow authors to style style inside a shadow tree, from outside of that shadow tree keeping the structure of the components and targeting only some nodes.

#### Why
The previous proposed method for styling inside the shadow tree, the `>>>` combinator, turned out to be too powerful for its own good.
The problem with using just custom properties for styling/theming is that it places the on the element author to basically declare every possible styleable property as a custom property.

#### How
It has a los of opctions but, basically:

```html
<x-foo>
  <!-- #shadow-root -->
    <div part="some-box"><span>...</span></div>
    <input part="some-input">
    <div>...</div> /* not styleable
</x-foo>
```

```css
x-foo::part(some-box) {
  background: CornflowerBlue;
}

x-foo::part(some-input) {
  display: flex;
}
```

#### When
When some part of your component might be exposed to the outside to allow you to add specific styles and it also need to style its pseudo selectors as `Hover` or others. 

### :theme() pseudo-selector
-------------------------------------------
This pseudo selector is similar to the previous one except for an important matter: it can match regardless of whether the originating element is a shadow host or not. This can go arbitrarily deep in the shadow tree. So, no matter how deeply nested they are, you could style all the exposed parts

#### Why
This pseudo-selector will allow that, given an exposed part in the elements, author will be able to change all of them using a single command. Since this go deep the shadow root, is similar to custom properties, but for pieces of code.

#### How
```html
<x-wrapper>
  <!-- #shadow-root -->
  <x-button part="label">
    <!-- This background will be CornflowerBlue color -->
  </x-button>
    <x-bar>
      <!-- #shadow-root -->
      <x-button part="label">
        <!-- This background will be CornflowerBlue color -->
      </x-button>
    </x-bar>
  <x-foo></x-foo>
</x-wrapper>
```

```css
:root::theme(label) {
  background: CornflowerBlue;
}
```

#### When
matching all elements in a page is a powerful tool for creating custom themes. It allows the authors to create themes using a single selector, for instance, for a full library of components.

## Summary
-----------------

- Use HTML properties for component logic
- Use HTML `slots` to allow the authors to add HTML and style it using the `slotted` psudo selector
- Do not use `::shadow` or `/deep/` (or similar) pseudo selectors since they are deprecated and should be removed.
- Use :host() to style the tag of a component from the shadow DOM instead that from its parent.
- Use host-context() to style the component tag when it depends of a parent HTML element.
- Use custom properties to allow an author to style a value from multiple components from outside.
- Use the `:part` pseudo selector to allow an author to style a piece of component from outside.
- Use the `:theme` pseudo selector to allow authors to style a piece of all the components from outside.


## Resources

- [Fundamentals: web components](https://developers.google.com/web/fundamentals/web-components/shadowdom)
- [MDN Shadow DOM](https://developer.mozilla.org/en-US/docs/Web/Web_Components/Using_shadow_DOM)
- [Shadow piercing Combinators in the Wild](https://github.com/w3c/webcomponents/wiki/Shadow-piercing-Combinators-in-the-Wild)
- [What's the substitute for ::shadow and /deep/?](https://stackoverflow.com/questions/35741722/whats-the-substitute-for-shadow-and-deep)
- [::part and ::theme, an ::explainer](https://meowni.ca/posts/part-theme-explainer/)
- [What is the different between :host ,:host() ,:host-context selectors](https://stackoverflow.com/questions/51326676/what-is-the-different-between-host-host-host-context-selectors)
- [:host-context on MDN](https://developer.mozilla.org/en-US/docs/Web/CSS/:host-context())
