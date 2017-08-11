---
layout: post
title:  "Using SASS Flexbox"
author: russell_wenban
---

Although at Qudini we use the Bootstrap Framework, I've been using [Flexbox](https://developer.mozilla.org/en-US/docs/Learn/CSS/CSS_layout/Flexbox) to layout items in a number of instances.

Flexbox can be used in all modern browsers and Webviews on mobiles, see [caniuse](http://caniuse.com/#feat=flexbox).

I’ve create a working playground to test samples to use in larger projects. See: Plunker - Using [SASS Flexbox](https://plnkr.co/edit/Q8r9z3LznyQsse9vppPu)

Hopefully this guide will allow the reader to implement a systematic approach to create HTML layouts. It is worth reading the Flexbox theory in the links provided first, or just start experimenting.

## Sass FlexBox

I've found that [SassFlexbox](https://github.com/zessx/sass-flexbox) is a very handy SASS library.

It can be used make the SASS classes more succinct using the [%placeholder](http://thesassway.com/intermediate/understanding-placeholder-selectors) syntax.

Also, using Sass-flexbox removes the need to add the annoying prefix classes by hand.

I created a further SASS file `ui-flexbox-styles.scss` to write reusable placeholders and SASS Classes. This allows the `ui.scss` to define Classes with less boiler plate code.

Once the flex box styles has been written they can be added quickly by applying the single class.

**Example**

```
// Will align multiple items vertically and horizontally
%flex-center-center-wrap-stretch {
  @extend %display-flex;
  @extend %align-items-center;
  @extend %justify-content-center;
  @extend %flex-wrap-wrap;
  @extend %align-content-stretch;
}
.flex-center-center-wrap-stretch {
  @extend %flex-center-center-wrap-stretch;
}
```

The `flex-center-center-wrap-stretch` Class can be used to align multiple items within a container.

## Systematic Approach

CSS can be a rather frustrating business, when creating Flexbox classes I always follow a systematic approach  I follow the order  in the [css-tricks](https://css-tricks.com/snippets/css/a-guide-to-flexbox/) guide.

1. display
2. flex-direction
3. flex-wrap (if required)
4. flex-flow (if required)
5. justify-content
6. align-items
7. align-content
8. add properties for the children

## Simple Examples

## Item in centre of content

The content is centred both horizontally and vertically, using Flexbox this is increadablry easy, just by adding one class `container-center`.


```
// will align the content dead center
%container-center {
  @extend %display-flex;
  @extend %flex-direction-column;
  @extend %justify-content-center;
  @extend %align-items-center;
}
.container-center {
 @extend %container-center;
}
```

**html**

```
<!-- one item dead centre -->
  <div class="container-full container-center">
    <div class="red square"></div>
  </div>
```

See [Plunker](https://plnkr.co/edit/ACY04CTPObd9dDkQ2tOX)


## Set of vertically aligned buttons

Following the the sequential pattern: display (flex of course), direction (row), justify the items in the row from the left, align items in the cross-axis.

```
// will align the content vertically centrally and horizontal from the left (with no margin)
%flex-row-start-center {
  @extend %display-flex;
  @extend %flex-direction-row;
  @extend %justify-content-flex-start;
  @extend %align-items-center;
}
.flex-row-start-center {
  @extend %flex-row-start-center;
}
```

**html**

In this example using standard bootstrap buttons with added Flexbox styles to display them within the given content.

```
<div class="container-full">
  <div class="button-container flex-row-start-center green-outline">
    <button type="button" class="btn btn-default">Home</button>
    <button type="button" class="btn btn-primary">About</button>
    <button type="button" class="btn btn-success right">Contact</button>
  </div>
</div>
```

The right hand side button can be aligned to the right by adding a class to the child:

```
.right {
   margin-left: auto;
}
```

In this case the Bootstrap style `pull-right` won’t work so it’s a good use-case for flexbox.

See [Plunker](https://plnkr.co/edit/IRk6peFHYllMt11ARJlY)


## Conclusion

My [Plunker](https://plnkr.co/edit/Q8r9z3LznyQsse9vppPu?p=preview) includes further examples including the [Holy Grail](https://philipwalton.github.io/solved-by-flexbox/demos/holy-grail/), container with Grow (items fill the available height), container with a sticky footer, a two panel layout and a container with multiple rows and columns. Feel free to try examples from this Plunker, fork your own and keep experimenting.

I hope you have learnt something from this blog post, at the least I believe I have provided a starting of point in how to systematically use Flexbox.

## Useful links

* [A Complete Guide to Flexbox](https://css-tricks.com/snippets/css/a-guide-to-flexbox/)
* [The Ultimate Flexbox Cheat Sheet](http://www.sketchingwithcss.com/samplechapter/cheatsheet.html)
* Some useful tips and examples can be found here: [webflow.com](https://preview.webflow.com/preview/flexbox?preview=78f49011ac3db6ccea265c2ba8a94185&m=1)
* Mozilla gives a very clear description of Flex terminology: [Using CSS Flexible Boxes](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Flexible_Box_Layout/Using_CSS_flexible_boxes)
* On Stack Overflow see: [Methods for Aligning Flex Items along the Main Axis](https://stackoverflow.com/questions/32551291/in-css-flexbox-why-are-there-no-justify-items-and-justify-self-properties/33856609#33856609) this answer provides a use [playground]((https://stackoverflow.com/questions/32551291/in-css-flexbox-why-are-there-no-justify-items-and-justify-self-properties/33856609#33856609)) to explore how to position items outside their default position on the _main axis_
