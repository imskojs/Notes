# Notes
Mastering Modern Mobile Front-End: Ionic, Angular, CSS3/SCSS, HTML5, ..., Angular 2.0, postCSS, ECMA2015, and maybe TypeScript.









#TOC
## CSS
[Flexbox Examples](#flexbox-examples)

#Flexbox
## Properties 
```css
.row {
  display: flex;

  flex-direction: row | row-reverse | column | column-reverse;
  flex-wrap: nowrap | wrap | wrap-reversea;
  /* shorthand */
  flex-flow: @flex-direction || @flex-wrap;

  /* main axis */
  justify-content: flex-start | flex-end | center | space-between | space-around;

  /* cross-axis useful only when flex-wrap is wrap*/
  align-content: flex-start | flex-end | center | space-between | space-around;

  /* cross-axis per line */
  align-items: flex-start | flex-end | center | baseline | stretch
}

.col {

  order: <integer>;

  flex-grow: 0;
  flex-shrink: 1;
  flex-basis: auto | <length>
  /* shorthand (use short)*/
  flex: @flex-grow @flex-shrink @flex-basis;  /* default none */ 

  align-self: flex-start | flex-end | center | space-between | space-around;

  /* float, clear, vertical-align does not work on flex item, but it does work on flex container. */
}

```
## Centering single content
### Method 1
```css
.row {
  display: flex;
  height: 50vw; /* if specified, content will be centered vertically  */
}

.col {
  flex: 0 0 20vw;
  height: 20vw;
  line-hieght: 20vw;
  text-align: center;
  margin: auto;
}
```

### Method 2
```css
.row {
  display: flex;
  height: 50vw;
  justify-content: center;
  align-items: center;
}
.col {
  flex: 0 0 20vw
  height: 20vw;
  line-hieght: 20vw;
  text-align: center;
  /* no margin auto */
}
```


Q: would .rowOuter>.rowInner>.col work?
i.e. .rowInner both display: flex, and flex: 0 0 90vw;










