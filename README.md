# Notes
Mastering Modern Mobile Front-End: Ionic, Angular, CSS3/SCSS, HTML5.
And preparing for Angular 2.0, ECMA2015









#TOC
## CSS
[Flexbox Examples](#flexbox-examples)

#Flexbox Examples
```css
.row {
  display: flex;

  flex-direction: row(def) | row-reverse | column | column-reverse;
  flex-wrap: nowrap(def) | wrap | wrap-reversea;
  /* shorthand */
  flex-flow: @flex-direction || @flex-wrap;

  /* main axis */
  justify-content: flex-start(def) | flex-end | center | space-between | space-around;

  /* cross-axis useful only when flex-wrap is wrap*/
  align-content: flex-start | flex-end | center | space-between | space-around;

  /* cross-axis per line */
  align-items: flex-start(def) | flex-end | center | baseline | stretch
}
```