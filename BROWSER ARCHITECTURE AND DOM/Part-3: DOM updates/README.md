
# Part 3: Why DOM Updates Are Expensive and How Browser Rendering Actually Works

In the previous section, we learned that JavaScript can access and modify the DOM. We also learned that modern web applications depend heavily on DOM manipulation because user interfaces are constantly changing. Buttons change state, notifications appear, messages arrive, shopping carts update, and dashboards refresh continuously. At first glance, this may not seem like a significant problem. After all, if JavaScript can update the DOM whenever necessary, why would developers need anything more?

The answer lies in understanding what actually happens after a DOM update occurs.

Many beginners imagine that changing the DOM is a simple operation. They picture JavaScript modifying an element and the browser instantly displaying the new result. In reality, a DOM update is often the beginning of a much larger chain of events occurring inside the browser. The browser may need to perform several expensive calculations before the user can see the updated content.

To understand why React was eventually created, we must first understand this hidden work.

Imagine that a webpage contains a simple heading displaying the text:

```text
Welcome
```

Suppose JavaScript changes the heading to:

```text
Welcome to Our International Corporate Management Platform
```

From a developer's perspective, this appears to be a small change. Only the text has been modified. However, from the browser's perspective, this change may have consequences throughout the page.

The browser must now determine whether the new text occupies more space than the old text. If it occupies more space, neighboring elements may need to move. Containers may need to expand. Scrollbars may need to appear. Portions of the layout may need to be recalculated.

What seemed like a simple text change can trigger a surprisingly large amount of work.

To understand this work, we must examine how browsers transform code into visual content.

When a browser receives HTML and CSS, it does not immediately display them on the screen. Instead, the browser constructs several internal structures that help it understand both the content of the page and the appearance of the page.

Earlier, we studied the DOM. The DOM tells the browser what elements exist within the document. It describes the structure of the webpage. The DOM knows that there is a heading, a button, a form, a paragraph, or an image. However, the DOM does not fully describe how these elements should appear visually.

Consider the following HTML:

```html
<h1>Hello World</h1>
```

The DOM can tell the browser that an `h1` element exists. However, the DOM alone cannot answer questions such as:

* What color should the heading be?
* What font should it use?
* How large should it appear?
* How much space should surround it?

The browser requires another source of information.

This information comes from CSS.

CSS describes the visual appearance of elements. It specifies colors, fonts, spacing, positioning, borders, backgrounds, shadows, animations, and many other visual properties. Just as the browser converts HTML into the DOM, it also converts CSS into another internal structure.

This structure is called the CSSOM.

The CSSOM, which stands for Cascading Style Sheets Object Model, can be thought of as the browser's internal representation of all CSS rules associated with the page. If the DOM answers the question:

> What elements exist?

then the CSSOM answers the question:

> How should those elements appear?

Neither structure is sufficient on its own.

Imagine a construction company building a house.

One document describes the structure of the building. It shows where the rooms, walls, doors, and windows are located. Another document describes the appearance of the building. It specifies paint colors, flooring materials, lighting fixtures, and decorative details.

Neither document alone is enough to construct the final building.

Similarly, the browser needs both the DOM and the CSSOM.

The browser combines these two structures to create what is known as the Render Tree.

The Render Tree is one of the most important concepts in browser rendering.

You can think of the Render Tree as the browser's blueprint for what should actually appear on the screen.

The DOM contains every element in the document.

The CSSOM contains every styling rule.

The Render Tree combines both pieces of information and represents only the elements that should be visually rendered.

This distinction is important.

Not every element in the DOM necessarily appears on the screen.

Consider the following example:

```html
<div style="display:none">
    Hidden Content
</div>
```

This element exists in the DOM. JavaScript can access it. The browser knows it exists.

However, because it is hidden using `display: none`, the user cannot see it.

Since the user cannot see it, it is excluded from the Render Tree.

The Render Tree therefore represents only the visual portion of the document.

Once the Render Tree has been created, the browser still cannot display anything.

The browser knows which elements should appear and how they should look, but it does not yet know exactly where those elements should be positioned.

Imagine receiving a list of furniture for a house.

You know that the house contains:

* A sofa
* A dining table
* A bed
* A television

However, you do not yet know where any of these items should be placed.

A similar situation exists inside the browser.

The browser knows which elements exist, but it must still determine their precise positions and dimensions.

This process is called layout calculation.

During layout calculation, the browser determines the exact width, height, and position of every visible element.

For example, the browser may calculate that a button should be:

```text
Width: 120 pixels
Height: 40 pixels
Position: x = 200, y = 300
```

These calculations must be performed for every visible element on the page.

For small webpages, this process is relatively inexpensive.

However, modern applications are rarely small.

Consider Facebook.

A single Facebook page may contain thousands of DOM nodes.

The browser must calculate positions and dimensions for a vast number of elements.

As applications become larger, layout calculations become increasingly expensive.

Once layout calculations are complete, the browser finally knows:

* What elements exist.
* How those elements should appear.
* Where those elements should be positioned.

Only then can the browser begin drawing them.

This drawing process is known as painting.

Painting is the stage during which the browser converts information into actual visual pixels.

Text is drawn.

Backgrounds are drawn.

Borders are drawn.

Images are drawn.

Shadows are drawn.

Everything the user sees ultimately results from the painting process.

At this point, the webpage becomes visible.

However, this is where the performance problem begins to emerge.

Whenever JavaScript modifies the DOM, the browser may need to repeat parts of this entire process.

A seemingly simple update can force the browser to:

* Recalculate styles.
* Rebuild portions of the Render Tree.
* Recalculate layouts.
* Repaint visual elements.

The larger the application becomes, the more expensive these operations become.

This is the hidden cost of DOM manipulation.

And this is the exact problem React's creators began investigating.

They asked a crucial question:

> If DOM updates are expensive, can we somehow reduce the number of unnecessary DOM updates before they ever reach the browser?

That question ultimately led to one of React's most famous innovations:

**The Virtual DOM**, which we will study next.
