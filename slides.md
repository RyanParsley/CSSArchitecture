name: CSS Architecture 
class: middle, center, title

# CSS Architecture
## Talkin' SMACSS

### Ryan Parsley
#### January 01, 2023

???

Well, here we go again

The presentation in which I attempt to articulate a strategy for proper care and
feeding of CSS. First off, if you haven't read [Jonathan Snook's take](http://smacss.com/book/)
on the subject, I recommend you still check out the source material. I have it
linked up at the end.

---

## What I like about SMACSS

> SMACSS is a way to examine your design process and as a way to fit those rigid
> frameworks into a flexible thought process

???

First off, I like the cut of Snook's jib. CSS is a complicated profession and
Jonathan leans into thought process over tools/libs/implementation details.


I like how this system focuses on separating CSS into categories and applies
patterns in a more targeted fashion. CSS isn't really one thing and Snook does a
good job explaining this.

---

## Categories

* What do I call this?
* Where should I put this?
* Should I even be writing this?
* Where should I look for prior art before I likely introduce duplicate CSS?

???

The ability to separate CSS into easy-to-communicate channels of concern will 
help drive our CSS architecture decisions. 


---

## Categories

* Base
* Layout
* Module
* State
* Theme

---
background-image: url(./assets/base.jpg)
background-size: 700px auto

## Base (examples from SMACSS)


???

The base rules are your defaults.

This should probably come directly from the Design System, and if it doesn't,
should be applied globally in the root stylesheet instead of in every component
that wants sensible defaults.

Wording that a different way may make it more self evident:
  "Does it make sense to define a _default_ inside view encapsulation?"

- Establishing defaults for your app
- Heavily rely on the Design system here
- Never use a !important here

---

## Base (example from SMACSS)

```scss
// styles.scss
// styles/_base.scss

body, form {
  margin: 0;
  padding: 0;
}

a {
  color: #039;
}

a:hover {
  color: #03F;    
}

```

???

This is the demo that Snook shares in his book.

---

## Base

```scss
// styles.scss
// styles/_base.scss

html,
body {
  padding: 0;
  margin: 0;
  height: 100%;
  background-color: $whitish;
  color: $grey;
}

body * {
  box-sizing: border-box;
  font-family: $font-family-sans;
}

```

??? 

This is probably more how yours should look though

---

## Layout

- Scoped like page layout
- Structure to compose components
- Traditionally ID selectors
  - we probably want component selectors. 
- _Could_ live in a more globally accessible place
  - `styles/_layout.scss` is common enough
- _Probably should_ live in a set of components
  - `shared/layout.component` feels sensible to me 

???

Snook calls out that you can use the word layout at many different scopes and
be correct, but what he means is essentially a "page layout" or a higher level
of laying out how various components (modules) come together.

---

## Layout (example from SMACSS)

```scss
// styles/_layout.scss

#header, #article, #footer {
    width: 960px;
    margin: auto;
}

#article {
    border: solid #CCC;
    border-width: 1px 0 0;
}
```

???

You can see what he's going for here, the app/ page has one header and footer
and this is the level at which we think about establishing how they render.

---

## Layout

```scss
// shared/layout/layout.component.ts

@Component({
  selector: 'app-layout',
  template: `
    <div class="aside" *ngIf="showAside">
      <ng-content select="[aside]"></ng-content>
    </div>
    <div class="primary">
      <ng-content></ng-content>
    </div>
    <div class="secondary" *ngIf="showDetails">
      <ng-content select="[secondary]"></ng-content>
    </div>
  `,
  styles: [`
    .primary {
      flex: 1;
    }
    .aside,
    .secondary {
      flex: 0 0 33.3%;
    }
  `]
})
```

???

The bulk of style of this category should probably live within a layout
component, in our shared styles at the root of the project or a combination of
the 2 like the full height panel layout.

---

## Module

- Read: components
- View encapsulation simplifies this
- Still worth thinking about

???

This class of style probably makes sense to live in the Style block of a given component, but still should be questioned before simply adding more to the app. Why do you feel you _need_ this CSS. Are you using the right classes and DOM structure? Are you striving for pixel perfection implementation of a mock that is off brand? Did you blindly copy from zeplin? The answer to these questions should guide your hand here.

---

## State

> 1. State styles can apply to layout and/or module styles; and
> 2. State styles indicate a JavaScript dependency.

```scss
.is-hidden
.is-collapsed
.is-error
```

???

This is the category of CSS that feels most compatible with utility classes.
Reusable classes like `.is-hidden` should probably be seen as bit of a smell
considering we're not writing jquery though. Why put something in the DOM and
force the browser to parse it just to render it invisible?

If you can keep this abstract, you should and create a utility class or
placeholder at the root of the project. When you're working with a state
modifier that only makes sense applied to a given module, define it scoped as
such. Do so either through naming convention `is-tab-active` or through a
component's view encapsulation. 

---

## Theme (example from SMACSS)

```scss
// in module-name.css

.mod {
    border: 1px solid;
}

// in theme.css

.mod {
    border-color: blue;
}
```

???

You probably don't need this, but I want to go over it quickly for completeness. 

Note: there's not an override, this example defines the theme specific attribute
in a theme file and not in the module's file.

The DS has a light and dark theme, which is one way to interpret this category.
Should we lean into responsive design, you can make an argument for that sort of
shift being a "mobile theme". For now, we'll mostly assume this category as not
our problem. 

You may use this to define different color schemes or maybe more densely packed
typography for. 

---

## What maybe doesn't fit for us

* But we use scss
* `src/styles.scss` has no view encapsulation
* View encapsulation lower the stakes of name-spacing
* You shouldn't be writing a lot of base or theme styles as a rule

???

SMACSS was written as a good strategy for plane ol vanilla CSS with no consideration or recommendation for getting tied to frameworks. As such, I think the problem solved with naming may better be solved in our context by location of css. SMACSS based thinking applied to our Angular + DS architecture probably shakes out like this. 

*Base* code should come directly from the DS and be applied globally (`src/styles/`). Keep all this out of component files. Layout components (`modules/layout-*`) and global styles (`src/styles/`) will share the responsibility of *layout* styles. 

*Module* is synonymous with how we see components, but we need to lean into granular components for this to work smoothly. Also, be more aware of how your base styles want to work and be more conservative about overriding that all willie nillie. 

*State* CSS may live in the globally accessible styles (`src/styles`) if it makes sense outside of the context of your component, but is probably largely fine living in your component so long as you've confirmed you're not writing redundant code (this really is the enemy here). 

Let's hold off on *theme* for the time being. 

---

## Mobile First

* **Not about** _optimizing for mobile_
* **Is about** starting with clarity

???

---

## Mobile First

```scss
// styles/_nav.scss

%nav-vertical {
  @extend %nav;
}

%nav-horizontal {
  @extend %nav;

  @media (min-width: $small) {
    li {
      display: inline-block;
    }

    a {
      margin: 0 0.25em;
    }
  }
}
```

???

In this example, I define navigation to be vertically oriented by default, then
alter that to fit horizontal. However, I enable the horizontal variant in a
media query because it's only applicable for bigger screens.
---

# Resources

* [SMACSS](http://smacss.com/book/)
* [Notes on layouts identified in Match](https://dev.azure.com/jbhunt/EngAndTech/_wiki/wikis/Applications.wiki/17561/Layouts)
* [Custom utility class defined in Match](https://dev.azure.com/jbhunt/EngAndTech/_git/app_operationsexecution_load_workflow_ui?path=/src/styles/panel-full-height.scss) (can be improved, but in the right direction)
* [Example of a layout component](https://dev.azure.com/jbhunt/EngAndTech/_git/app_operationsexecution_load_workflow_ui?path=/src/app/modules/layout/layout-panel/layout-panel.component.ts) (can be improved, but in the right direction)

