---
title: "Virtual DOM Under The Hood - ANK
"
seoTitle: "Beyond the Framework"
seoDescription: "this is about how virtual DOM works under the hood and how the things actually working in the real life 
a brief description on Virtual DOM in react"
datePublished: 2026-05-04T14:06:08.153Z
cuid: cmor9voye00e92ad3ccl9dvw1
slug: virtual-dom-under-the-hood-ank
cover: https://cdn.hashnode.com/uploads/covers/69f702bc0ab374db9990e9f5/d761315c-d12e-4b98-8867-8a816634bbba.png
ogImage: https://cdn.hashnode.com/uploads/og-images/69f702bc0ab374db9990e9f5/3e92978d-f7bc-45f2-8e2d-7b0124160f91.png
tags: reactjs, mobile-development, chaicode, ank, ank09yadav

---

just imagine you have a list of 1000 items displayed on you screen, Now a user clicks a button that updates just one item in the list.

Sound simple right ?

But under the hood, if you're directly manipulating the DOM using traditional JavaScript, the browser doesn't think like you do. It doesn't know that one item changed. Instead, it may end up recalculating layouts, repainting elements, and touching far more parts of the UI than necessary.

> Here's the catch: The DOM isn't just a simple object—it's a complex, tree-like structure managed by the browser, and every update to it is costly.

### What actually happens in Naive DOM Update?

```javascript
document.getElementById("title").innerText = "Hello World";
```

Looks harmless. But internally, the browser may:

*   *Recalculate styles (Reflow)*
    
*   *Update layout positions*
    
*   *Repaint pixels on the screen*
    

Now imagine doing this **hundreds or thousands of times** in a dynamic application.

That’s where performance starts to break.

## Virtual DOM Concept (solution)

Think of the Virtual DOM as a **lightweight copy** of the real DOM. It is purely a plain JavaScript object.

Because it is just a JavaScript object, it has no connection to the browser’s layout engine. Creating and updating this object is incredibly fast. React treats this Virtual DOM as a "blueprint." When state changes, React builds a new blueprint, compares it with the old one, and *only then* tells the browser exactly what to change in the Real DOM.

## How the Virtual DOM Works  

![](https://cdn.hashnode.com/uploads/covers/69f702bc0ab374db9990e9f5/932e1472-f18a-442e-b98c-dfa97c58e7de.png align="center")

### 1\. Initial Render

This is the baseline boot-up sequence of your application. React needs to know *what* to display before it can manage *how* to change it.

When you call `ReactDOM.render()`, React takes your component hierarchy and, for the very first time, builds a complete **Virtual DOM tree**. This is just a massive, nested JavaScript object that describes the UI structure. *Simultaneously*, it "hydrates" or translates this entire tree into the **Real DOM**, allowing the user to see the initial application.

**Analogy:** You create the initial blueprint for a new house (VDOM) and immediately begin the construction based *only* on that blueprint (Real DOM).

```javascript
// Step 1: The input (what you write in JSX)
const myComponent = (
  <div id="container">
    <button>Click Me</button>
  </div>
);

// Step 2: The Virtual DOM Representation (The invisible JS tree)
// This is an internal React object structure, simplified here.
const vDOMNode = {
  type: 'div',
  props: { id: 'container' },
  children: [
    { type: 'button', props: { children: 'Click Me' }, children: [] }
  ]
};

// Step 3: The Result (What React tells the browser to create)
// <div id="container">
//   <button>Click Me</button>
// </div>
```

### 2\. STATE CHANGE TRIGGER

In this React will *only* re-render a component if its `state` or `props` change. This change can be triggered by user interaction (like clicking the button or any user interaction ), a timer finishing, or a response from a network request. The key method that starts this entire engine is typically `this.setState()` in class components, or a state setter (e.g., `setCount()`) from the `useState` hook.

```javascript

import React, { useState } from 'react';

function Counter() {
  // A. Initial state is set to 'false' (Panel 4.1 state)
  const [isToggled, setIsToggled] = useState(false);

  return (
    <div>
      {/* B. The button seen in Panel 4.2 */}
      <button onClick={() => setIsToggled(!isToggled)}>
        Toggle Me
      </button>
      
      {/* This element will change base on isToggled */}
      <p>{isToggled ? "ON" : "OFF"}</p>
    </div>
  );
}
```

### 3\. DIFFING (RECONCILIATION)

Immediately after the `setState` trigger, React doesn't touch the DOM. Instead, it re-runs your component and generates a **New Virtual DOM tree**, capturing the application’s state *after* the update.

React now possesses two blueprints: the **Previous VDOM** (from Panel 4.1) and the **New VDOM** (just created in Panel 4.2). The `diff` engine walks through both trees concurrently. As the flowchart shows, it’s looking for *nodes that are different*. In the diagram, it highlights `D*` in the new tree because its data (represented by the orange shade) has changed. The algorithm flags this precise node and its modification.

**The How of Diffing:**

*   **Tree Walking:** Start at the root and compare children by index.
    
*   **Optimal Algorithm:** $O(n)$ time complexity (walking the tree once).
    

### 4\. PATCHING

The Diffing engine produces a set of minimal operations (the "patches") required to transform the Previous VDOM into the New VDOM. These patches are bundled together. React then iterates over this list of specific operations and applies them, *one by one*, to the **Real DOM**.

```javascript
// React knows the text node within the <p> element changed from 'OFF' to 'ON'

// The actual (internal) browser update (optimized)
const pElement = document.getElementById('status_paragraph'); 
pElement.textContent = "ON"; // FAST: Minimal DOM manipulation
```

### 5\. The Browser Rendering Pipeline

Once React completes the **Patching** phase, the DOM nodes have been updated in memory. Now, the browser engine (like Chromium or WebKit) takes over to show those changes on your screen. This is what you see when you inspect the "Performance" tab in Chrome DevTools.

## Conclusion : Virtual DOM Is Not Magic — It’s Smart Engineering

At first glance, the Virtual DOM can feel like magic.

You update the state, and somehow the UI updates efficiently without you touching the DOM directly. But now you know the truth:

> It’s not magic—it’s a **carefully designed diffing and reconciliation strategy**.

Instead of blindly updating the entire UI, the system:

*   *Creates a lightweight representation of the UI (Virtual DOM)*
    
*   *Compares it with the previous version*
    
*   *Calculates the minimal set of changes*
    
*   *Applies only those changes to the real DOM*
    

This is what makes modern UI libraries feel fast and responsive.

* * *

## The Real Takeaway (Think Like the Engine)

The most important shift is this:

> Stop thinking in terms of *“how to update the DOM”*  
> Start thinking in terms of *“what should the UI look like for a given state”*

Once you adopt this mindset, everything—components, state, rendering—starts to make sense.

* * *

## But It’s Not Always Perfect

The Virtual DOM is an optimization, not a silver bullet.

*   *It adds an extra layer of abstraction*
    
*   *It consumes memory for maintaining trees*
    
*   *In some cases, direct updates or fine-grained reactivity can be faster*
    

That’s why newer approaches (like signals-based systems) are evolving beyond it.

* * *

## Final Thought

If you truly understand the Virtual DOM, you’re no longer just using a framework—you’re understanding how UI systems are designed.

> “The fastest DOM update… is the one you never make.”