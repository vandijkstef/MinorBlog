# CSS Variables and SASS; In search of harmony
Variables in CSS are a thing. Personally, it has been the main reason I started to use preprocessors like Less or Sass. To this very day, I've barely been using mixins or other funky stuff included in the preprocessors, just variables and some color alteration functions.

I'm drifting off-topic, [CSS variables are a thing](https://developer.mozilla.org/en-US/docs/Web/CSS/Using_CSS_variables), and [actually usable in modern browsers](https://caniuse.com/#feat=css-variables). Fully native variables used in CSS, making the language a lot less [DRY](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself).

This article is written as an exploration to use CSS variables in conjunction with preprocessors. The **SCSS** syntax is used for the examples.

## CSS and SASS variables in two minutes
> CSS variables are entities defined by CSS authors that contain specific values to be reused throughout a document.
> 
> -- [MDN](https://developer.mozilla.org/en-US/docs/Web/CSS/Using_CSS_variables)

This is a quick overview of using CSS variables. If you are unfamiliar with CSS variables, I recommend reading through the [MDN documentation](https://developer.mozilla.org/en-US/docs/Web/CSS/Using_CSS_variables) and this [short article from Per Harald Borgen](https://medium.freecodecamp.org/learn-css-variables-in-5-minutes-80cf63b4025d)

### Creating CSS variables
CSS variables **are scoped** to the element they are set on, commonly you should define them on the `:root` selector to be able to access them. To seperate these new properties from exisiting ones, every variable name should be prefixed with `--`. 

``` CSS
:root {
	--main-color: red;
}
```

### Using CSS variables
Using the CSS variables is used with the CSS function `var()`. While the `var()` function is supporting fallbacks, it isn't at all supported in older browsers, thus the following example is recommended usage.
``` CSS
div {
	color: red; /* Fallback for legacy browsers */
	color: var(--main-color, red); /* Red as fallback if --main-color isn't set */
}
```
And for usage in JS
``` JS
const root = document.querySelector(':root');
root.style.setProperty('--main-color', newValue);
```
Note that setting or changing variables through JS allows for scoping. 
This codepen will [demonstrate this scoping](https://codepen.io/vandijkstef/pen/VdxqEg).

### SASS variables
SASS variables are processed into the CSS file, and are not usable on the front-end like the CSS variables are. However, since [CSS Filters](https://caniuse.com/#feat=css-filters) are less supported and don't automatically provide a fallback, SASS can provide several [usefull functions](http://sass-lang.com/documentation/Sass/Script/Functions.html). The basic usage of SASS variables is shown below.
``` SCSS
$main-color: red;

div {
	color: $main-color;
}
```

## The requirements
Based on the above, I'd need the following requirements for a usable solution:
- Able to 'register' variables in SASS and as CSS variable
- Able to use the variable
- Ideally provide a fallback

On combining the two types of variables, I've refered to two examples, which are closing in on the solution, but aren't quite there yet.
- [The Issue with Preprocessing CSS Custom Properties | CSS-Tricks](https://css-tricks.com/issue-preprocessing-css-custom-properties/)
	- Shows some basic usage which and still requires code duplication
- [CSS custom properties and SCSS via mixin](https://codepen.io/malyw/pen/aNwKKv)
	- Advanced example, a lot of snippets have been used in the solution

## Solving the problems
### Register variables
Required CSS syntax
``` CSS
:root {
	/* List of variables and values */
}
```
To create the list we can make use of a [SASS map](https://webdesign.tutsplus.com/tutorials/an-introduction-to-sass-maps-usage-and-examples--cms-22184).
``` SCSS
// Define the rootVars variable
$rootVars: ();

// The mixin to easily fill the map
@mixin setVar($varName, $value){
  $rootVars: map-merge($rootVars, (
    #{$varName}: $value
  )) !global;
}

// Set some variables
@include setVar("color", red);
@include setVar("main", green);
@include setVar("off", orange);

// And make sure they are available in CSS after you have set all your variables
:root {
	@each $key, $value in $rootVars {
		--#{$key}: #{$value};
	}
}
```
#### Output
``` CSS
:root {
  --color: red;
  --main: green;
  --off: orange;
}
```

### Using the variables
Required CSS syntax
``` CSS
div {
	property: value;
	property: var(--name, value);
}
```
To simply retrieve a variable from the map we can use the following function
``` SCSS
// Get the variable from the map
@function getVar($varName) {
	@return map-get($rootVars, #{$varName});
}

// Set the variable on a property
div {
	color: getVar("color");
}
```
#### Output
``` CSS
div {
  color: red;
}
```

### Provide a fallback
To provide a fallback we would need a mixin.
``` SCSS
// Get the property and varName to build the correct code.
@mixin var($property, $varName, $fallback:false) {
	$var: getVar($varName);
	@if $var { // If we got a variable from the map, use it
		#{$property}: #{$var}; // Output legacy CSS
		@if $fallback { // If we have a fallback, include it
			#{$property}: var(--#{$varName}, $var, $fallback);
		} @else { // else, dont
			#{$property}: var(--#{$varName}, $var);
		}
	} @else { // No variable set, use the fallback
		#{$property}: #{$fallback}; // Output legacy CSS
		#{$property}: var(--#{$varName}, $fallback); // Still provide the link to a CSS variable, should it become available
	}
}

// Use the mixin
div {
	@include var('color', 'main');
}
```
#### Output
``` CSS
div {
	color: green;
	color: var(--main, green);
}
```

## Conclusion
To this stage, CSS variables and SASS variables are usable together. The amount of SCSS is small, and far more important, we can get a clean CSS output. They are not in harmony yet. Though `getVar()` allows to be used in SASS functions, the connection to the CSS variable is lost. Additionally, SASS isn't currently generating CSS filters based on their functions. It could be usefull during development, since you can change the `:root` CSS in the devtools, and immediatly see the results. Last, but not least, remember to only use CSS variables if you plan to change variables with JS, otherwise they will just needlessly bloat your CSS.

## Notes
Unless specified otherwise, source material is referenced in June 2018

# Coffee Pasta
Just because I'm as lazy as you are

``` SCSS
// Define the rootVars variable and function
$rootVars: ();

// Mixins and Functions
@mixin setVar($varName, $value){
  $rootVars: map-merge($rootVars, (
    #{$varName}: $value
  )) !global;
}

@function getVar($varName) {
	@return map-get($rootVars, #{$varName});
}

@mixin var($property, $varName, $fallback:false) {
	$var: getVar($varName);
	@if $var { // If we got a variable from the map, use it
		#{$property}: #{$var}; // Output legacy CSS
		@if $fallback { // If we have a fallback, include it
			#{$property}: var(--#{$varName}, $var, $fallback);
		} @else { // else, dont
			#{$property}: var(--#{$varName}, $var);
		}
	} @else { // No variable set, use the fallback
		#{$property}: #{$fallback}; // Output legacy CSS
		#{$property}: var(--#{$varName}, $fallback); // Still provide the link to a CSS variable, should it become available
	}
}

// Set variables
@include setVar("main", green);
@include setVar("contrast", red);

// And make sure this is after you set your variables
:root {
	@each $key, $value in $rootVars {
		--#{$key}: #{$value};
	}
}

// Usage
div {
	color: getVar("contrast");
	@include var("background-color", "main", tomato); // The fallback 'tomato' is optional
}

```

