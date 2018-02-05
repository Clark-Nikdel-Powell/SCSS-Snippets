# SCSS-Snippets
Common functions, mixins and extensions for use in SCSS.

## Mixins

### between_with_media_queries
#### Parameters: $properties: map, $from_value: pixel, $to_value: pixel, $from_width: pixel, $to_width: pixel, $continue: boolean

This mixin scales a property from one value to another over a set breakpoint range. For the record, I was using this months (months, I say!) before Typekit made it cool with their concept of "CSS locks." I found a postcss plugin that hijacked the font-family property (I think) and converted it to a mixin instead. I primarily use it with `font-size`, but it can be used with any CSS property, or multiple properties at once.

Setting `$continue` to `true` will make the `$to_value` persist after the `$to_width` breakpoint. 

One thing we could do to make it better is adapt it from height-specific media queries as well.

### color_bar
#### Parameters: $color: any value that will work with `background`, $include_both: boolean

Worth its weight in gold for being able to set a browser-width background for a specific section without creating a new `.row` div. It uses a `:before` pseudo-element to place the background, which could be anything from a solid color to a gradient, or even an image. When using `color_bar`, it is usually wise to have an `.inside` div just after the before element, so that you can place `position: relative` on it and have the content take a higher stacking index.

### css_by_breakpoint
#### Parameters: $breakpoint-map: a nested map of breakpoints

Another workhorse of our custom site builds, I use css_by_breakpoint as a shorthand for any value that changes over multiple breakpoints. The syntax in use looks like this:

```scss
@include css_by_breakpoint((
	default: (
		margin-left: $cnp-small-grid-row-margin * -1,
		margin-right: $cnp-small-grid-row-margin * -1,
	),
	small: (
		margin-left: $cnp-small-grid-row-margin * -1,
		margin-right: $cnp-small-grid-row-margin * -1,
	),
	medium: (
		margin-left: $cnp-medium-grid-row-margin * -1,
		margin-right: $cnp-medium-grid-row-margin * -1,
	),
	$interior-content-width: (
		margin-left: calc((100vw - #{$interior-content-width} + (#{$cnp-medium-grid-row-margin} * 2)) / -2),
		margin-right: calc((100vw - #{$interior-content-width} + (#{$cnp-medium-grid-row-margin} * 2)) / -2),
	),
	$site-width: (
		margin-left: ($site-width - $interior-content-width) / -2,
		margin-right: ($site-width - $interior-content-width) / -2,
	),
));
```

Some notes on this example: 
1. Default is used to keep the properties that change over breakpoints all in one place. The output does not wrap it in a media query.
1. Small is superfluous here, and only used for demonstration purposes. Using `small` as the key will return a "small only" breakpoint.
1. Anything that is accepted by the Foundation `breakpoint` mixin will work as a breakpoint key. Values like `$interior-content-width` or `medium only` or `large down` will all work.

**Note:** in order to use Emmet shorthands inside of CBB in PhpStorm, you'll need my custom extensions loaded into PhpStorm's Live Template library.

### do_something_over_breakpoints
#### Parameters: $properties: map, $multipliers: map of integers, $increment-map: map of breakpoint / value pairs

This simple mixin powers many others, like `site_margin` and `vertical_rhythm`. It makes using increment maps like these easy:

```scss
$site-margin-map: (
	small only: $cnp-small-grid-row-margin,
	medium only: $cnp-medium-grid-row-margin,
	mediumlarge only: $cnp-medium-grid-row-margin,
	large only: $cnp-medium-grid-row-margin,
);
```

To apply the site margin to an element, we'd use this:

```scss
@include do_something_over_breakpoints( (margin-left, margin-right), (-1), $site-margin-map );
```

This logic is used in the `site_margin` mixin. The value of `do_something_over_breakpoints` is that it standardizes the loop and gives us a common, predictable template from which to work.

**Note**: you can skip specific breakpoints by passing "null" in the multipliers map, like this:

```
@include do_something_over_breakpoints( (margin-left, margin-right), (-1, null, -2), $site-margin-map );
```

Doing this would skip output for the second breakpoint in `$site-margin-map`, and resume with -2 for the third breakpoint in `$site-margin-map`.

### do_something_over_breakpoints_constant
#### Parameters: $properties: map, $multipliers: map, $constant: integer

The sister mixin to `do_something_over_breakpoints`, this mixin switches things up by requiring a constant instead of an increment map. A good example for this mixin is `vertical_rhythm`, which uses the `$grid-unit` constant as its basis instead of a value that changes based on the breakpoint. Instead of pulling the breakpoint from the `$increment-map`, it gets the current breakpoint from the globally defined `$breakpoints` variable, based on a counter variable that increments with each loop.

### flex_wrap_to_no_wrap
#### Parameters: $breakpoint: breakpoint string

I got tired of writing `flex-wrap: wrap` and `flex-wrap: nowrap` for different breakpoints.

### font_face
#### Parameters: $fontname: name of the font, $filename: full filename, minus the extension

Simple font-face mixin.

### font-smoothing
#### Parameter: $on_or_off: string

Pass 'on' to turn font-smoothing on, off to turn it off.

### gradient-striped
#### Parameters: $direction: angle, $colors: map

A simple mixin for creating a striped gradient. Haven't used it too much, to be honest.

### match_site_width
#### Parameters: $properties: map

Uses negative margins to pull an element out horizontally to match the width of a site, working in tandem with the `$site-margin-map`, `$content-width` and `$site-width` variables.

### match_site_width_absolutely

Uses the same calculations as match_site_width, but positions the element absolutely. Useful for complex background elements that can't or shouldn't be postioned to match the site width using simple margins.

### placeholder

Cross-browser placeholder styles. Example: 

```scss
@include placeholder() {
	color: $gray-4f;
}
```

### site_margin
#### Parameters: $indent_or_outdent: 'indent' or 'outdent,' defaults to 'indent', $properties: map

Uses the `$site-margin-map` values to put a site margin on an element. It's typically used for `.webpage-column` with indent, and anything that needs to be pulled back out with outdent. **Note:** if you need to match the site width and not just the site margin, use `match_site_width` instead, as it includes a `site_margin(outdent)` call.


### typographic_item
#### Parameters: $settings: map, with keys for 'tag', 'font', 'weight', 'color' and 'include_properties'

Hooo-boy. This is the most recently added mixin to this repo, and it's either the best thing in the world or utter trash. The problem I've faced over the years is that we often find the need to divorce visual styles from tag styles. For instance, we used to style our typography going by a strict tag-based system, where if you wanted something to look like an `h4`, it had better well be an `h4` tag because that's the only way that's gonna happen.

However, that's not very flexible. So after many iterations, I've finally landed on the `typographic_item` mixin as the end-all be-all mixin for any singular typographic element. Here's how it looks in practice:

```scss
@mixin h1( $settings:() ) {
	$defaults: (
		'tag': h1,
		'font': $trim-bold,
		'weight': normal,
		'color': $purple,
    'include_properties': true,
	); // <-- The only parameters used by the mixin are tag, font, weight, color, and include_properties.
	$args: map_merge($defaults, $settings); // <-- Similar to wp_parse_args, we can set defaults but allow them to be overwritten.

	text-transform: uppercase; // <-- Set anything that a) is mandatory and b) is constant (i.e., that doesn't take a variable value, like 32px or 1.5 or 600 here.

	@include typographic-item($args);
}
```

This way, if you need a green H1 instead of a blue one, you can overwrite the color when you pass in the settings for the mixin. Same goes for the font or weight. Of course, you can work up more verbose examples:

```scss
@mixin btn( $settings:() ) {
	$defaults: (
		'tag': btn,
		'background': $mint,
		'background-hover': darken(saturate($mint, 10%), 15%),
		'color': #fff,
		'color-hover': #fff,
		'font': $decima-mono,
		'weight': bold,
	);

	$args: map_merge($defaults, $settings);

	@include scut-vcenter-ib();
	background: map_get($args, 'background');
	display: inline-block;
	line-height: 1;
	text-align: center;
	text-decoration: none;
	text-transform: uppercase;

	&:hover {
		background: map_get($args, 'background-hover');
		color: map_get($args, 'color-hover');
	}

	@include typographic-item($args);
}
```

The flexibility of this mixin pattern is that it can be used for any singular typographical item, from lists to headers to form labels. The final piece is how the `typographic-item` mixin interacts with the `$header-styles` variable to manage the responsive settings of a typographic item, like the font-size, line-height and more. Any property that will change over breakpoints should go into `$header-styles`, so that we can have a comprehensive idea of what the typographic styles are. Take this partial example: 

```scss
$header-styles: (
	small: (
		'h1': (
			'font-size': scut-em(25),
			'line-height': scut-em(24, 25),
		),
		'h2': (
			'font-size': scut-em(21),
			'line-height': .9,
			'letter-spacing': scut-em(.8, 21),
		),
		'h3': (
			'font-size': scut-em(17),
			'line-height': scut-em(16, 17),
		),
		'intro': (
			'font-size': scut-em(14),
			'letter-spacing': scut-em(.53, 14),
			'line-height': scut-em(23, 14),
		),
		'tagline': (
			'font-size': scut-em(10),
			'letter-spacing': scut-em(.38, 10),
		),
		'btn': (
			'font-size': scut-rem(12),
			'height': scut-em(40, 12),
			'letter-spacing': scut-em(.49, 12),
			'padding-left': scut-em(30, 12),
			'padding-right': scut-em(30, 12),
		),
		'p': (
			'font-size': scut-em(13),
			'line-height': scut-em(22, 13),
			'letter-spacing': scut-em(.53, 13),
		),
	),
);
```

### vertical_rhythm
#### Parameters: $properties: map, $multipliers: map

Uses the `$grid-unit` variable as its basis for calculating changes in margin, padding, font-size or more over specific breakpoints. If you're not sure when to use it, check out the section titled "[The Grid-Unit Function and how to handle responsive values](https://github.com/Clark-Nikdel-Powell/Foundation-Theme/wiki/Front-End-Approach#the-grid-unit-function-and-how-to-handle-responsive-values)" in the CNP Foundation Theme docs.

### z
#### Parameters: $layers: map

Takes a layer name and returns the z-index value from the `$z-layers` variable set in `_site-settings.scss`.

## Extensions

## Functions
