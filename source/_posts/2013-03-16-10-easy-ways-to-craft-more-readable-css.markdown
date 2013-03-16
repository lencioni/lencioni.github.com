---
layout: post
title: "10 Easy Ways to Craft More Readable CSS"
date: 2013-03-16 15:45
comments: true
categories:
  - Causes
  - CSS
  - Readability
  - Sass
  - Stylesheets
---

> Always code as if the [person] who ends up maintaining your code will be a
> violent psychopath who knows where you live. Code for readability.
> —<cite>[John Woods](https://groups.google.com/d/msg/comp.lang.c++/rYCO5yn4lXw/oITtSkZOtoUJ)</cite>

Diving into a large, old piece of CSS typically is neither easy nor
pleasurable. I find that **the biggest challenges in working with old CSS often
lie in understanding the purpose and interactions of the styles**.

When styling new elements, we have the entire context of the implementation
immediately available, and it is easy to write styles that make sense to us at
that very moment. However, in a few weeks or to a fresh pair of eyes, what made
a lot of sense at first might end up being a lot more cryptic. Without a clear
understanding of the purpose and interactions of the styles, modifying
stylesheets can be dangerous, tedious, and cumbersome. Therefore, it is
important to communicate enough context so that future developers will be able
to grok the code easily and make informed decisions.

At [Causes](http://www.causes.com/), we have adopted the following practices
which we believe have improved the maintainability of our stylesheets, reduced
bugs, and increased developer velocity. When you have finished reading this, I
hope that you will have a few more tools to help move your codebase toward
greater maintainability.

<!--- more --->

## 1. Show Your Work

My math teachers would always require me to show my work. And, for good
reason—this exposed the steps that led up to my answer, allowing my teacher to
observe my thought processes and understand the point at which things went
horribly wrong (or magnificently right). In other words, showing my work
communicated a greater context with which to understand the reason something
was done.

Similarly, styles that show the work behind mysterious numbers communicate more
context, making them more readable and maintainable. We can easily apply this
when using a relative unit like
[ems](http://en.wikipedia.org/wiki/Em_(typography\)), where we determine the
value by taking the target size in pixels divided by the context size in
pixels:

    ems = target px / context px

Imagine that we want to have an element with a font size of 10px, but defined
in ems, inside a container with a font size of 16px. The quick and easy way
would be to run through the equation above and simply drop in the result:

``` css Quick, easy, and unreadable ems
font-size: .625em;
```

However, if someone were to come along a few weeks later and see this strange
number, they might be confused about its exact meaning. Using a CSS
preprocessor such as [Sass](http://sass-lang.com/), we can use a little math to
show our work and communicate more context:

``` scss Readable ems with Sass math
font-size: (10px / 16px) * 1em;
```

Or, in plain CSS, a little comment can go a long way:

``` css Readable ems in plain CSS
font-size: .625em; /* 10px / 16px */
```

Now, when someone sees this style, it is plain that it was expected to target a
10px font size in a 16px font size context, and it will be easier to make an
informed decision as to how to modify it.

Think of our styles as machines with many parts. If the machine isn’t working
as we’d like, it will be easier to repair it if we can actually see the moving
parts.

## 2. Two or More, Use a For

Spend some time establishing patterns and defining characteristics generally,
to promote reusability. This can be accomplished through setting up
presentational classes that can be mixed and matched to achieve a unified
visual appearance. It might be a good idea to set up a library of these
patterns that you can share with your team or start with a good framework like
[Typeplate](http://typeplate.com/) or
[Bootstrap](http://twitter.github.com/bootstrap/) that kickstart this process
for us.

With a CSS preprocessor like Sass, we have some very useful tools that allow us
to kick this up a notch, more easily take a
[DRY](http://en.wikipedia.org/wiki/Don%27t_repeat_yourself) approach to CSS,
and do more with less code.

**Dispel magic numbers with variables.** Similar to showing our work, instead
of leaving a hex color like `#00c5cd` for somebody to grok, naming it
`$turquoise` helps others immediately recognize what it is. In this case, the
reusability of variables is just the icing on the cake.

**Perform common calculations with functions.** [Sass has a bunch of built-in
functions](http://sass-lang.com/docs/yardoc/Sass/Script/Functions.html) that
can make our lives easier. To fill in the gaps, Sass allows us to define our
own functions. For example, at Causes we added a Sass function that facilitates
calculations of ems:

``` scss Defining functions in Sass http://sass-lang.com/docs/yardoc/file.SASS_REFERENCE.html#function_directives
@function calc-em($target-px, $context-px) {
  @return ($target-px / $context-px) * 1em;
}

h1 {
  font-size: calc-em(36px, 16px);
}
```

**Use mixins and loops to abstract out common patterns.** Packages like
[Compass](http://compass-style.org/) and [Bourbon](http://bourbon.io/) give us
a lot of mixins right out of the box that make cross-browser compatibility a
breeze. So, instead of this mess:

``` css Vendor-prefixed CSS transitions
-moz-transition: all 0.8s ease-in-out;
-ms-transition: all 0.8s ease-in-out;
-o-transition: all 0.8s ease-in-out;
-webkit-transition: all 0.8s ease-in-out;
transition: all 0.8s ease-in-out;
```

With Compass:

``` scss Simple transitions in SCSS with Compass http://compass-style.org/reference/compass/css3/transition/
@include transition(all 0.8s ease-in-out);
```

Similarly, our first pass at a responsive design implementation added a
stylesheet on top of all of our styles to collect media queries.  However,
there was too much distance between the original style and the style being
modified in the media query, which resulted in the styles quickly becoming out
of sync as developers made modifications. So we sprinkled these styles
throughout our code, moving our media queries adjacent to the styles themselves
to increase locality, and used mixins to simplify the implementation, improve
readability, and reduce repetition:

``` scss Using Sass mixins for media queries http://sass-lang.com/docs/yardoc/file.SASS_REFERENCE.html#mixins
@mixin respond-to($media) {
  @if $media == tablet {
    @media only screen and (max-width: 768px) { @content; }
  }
  @else if $media == mobile {
    @media only screen and (max-width: 360px) { @content; }
  }
}

h1 {
  font-size: calc-em(36px, 16px);

  @include respond-to(tablet) {
    font-size: calc-em(24px, 16px);
  }

  @include respond-to(mobile) {
    font-size: calc-em(18px, 16px);
  }
}
```

However, mixins and loops are a double-edged sword that can easily result in
generating too many styles. Always consider how your styles can be generalized
and reused. There may also be occasions where `@extend` makes a lot of sense,
but [make sure to use it with
placeholders](http://8gramgorilla.com/mastering-sass-extends-and-placeholders/).

## 3. Use Percentages for (Most) Widths

Have you ever beautifully styled an element, only to come back a few months
later to discover that the width needs to be adjusted due to an unrelated
change to a container?

I’ve discovered that if I start with percentages for horizontal dimensions
(widths, margins, and paddings) instead of pixels, *even when I’m not working
with a flexible design*, my styles are more resilient and require much less
babysitting. As a bonus, they tend to work better when dropped into new and
unexpected contexts and transition very well to flexible and responsive
layouts.

When using Sass, lean heavily on the percentage function to show your work (or
leave a comment):

``` scss Sass percentage function http://sass-lang.com/docs/yardoc/Sass/Script/Functions.html#percentage-instance_method
width: percentage(target / context);
```

Browsers use a default content-box sizing model to determine the sizes of
elements. This means that the `width` style will apply to the content box, and
then the padding, border, and margin get added on top of that. This is a
frequent gotcha when using percentages for widths, causing the element to take
up more space than you intended. Thankfully, we can tell the browser to use a
different box sizing model that calculates the size of an element including its
padding and border. Using the `box-sizing` property:

``` css Vendor-prefixed CSS box-sizing
textarea {
  border: 1px solid #333;
  -webkit-box-sizing: border-box;
  -moz-box-sizing: border-box;
  -ms-box-sizing: border-box;
  -o-box-sizing: border-box;
  box-sizing: border-box;
  padding: 1em;
  width: 100%;
}
```

Or, using Compass:

``` scss SCSS and Compass box-sizing http://compass-style.org/reference/compass/css3/box_sizing/
textarea {
  @include box-sizing(border-box);
  border: 1px solid #333;
  padding: 1em;
  width: 100%;
}
```

There are circumstances that require some fixed-width styles. For instance,
imagine a search box widget that consists of a text input and a button
side-by-side. If we want the entire widget to occupy the full width of its
container, but the button to be always as narrow as possible, we might want to
define the width of the button using a fixed unit and the input using a
percentage.

{% img fancy /assets/2013-03-16-flexible-search-widget.png %}

``` html HTML http://jsfiddle.net/jdRJB/
<form class="search-widget" method="get" action="action">
  <input type="submit" value="Search">
  <fieldset>
    <input name="q" type="search">
  </fieldset>
</form>
```

``` css CSS http://jsfiddle.net/jdRJB/
.search-widget fieldset {
  border: 0;
  margin-right: 5em;
  padding: 0;
}

.search-widget input[type=submit] {
  float: right;
  width: 5em;
}

.search-widget input[type=search] {
  width: 100%;
}
```

``` scss SCSS
.search-widget {
  fieldset {
    border: 0;
    margin-right: 5em;
    padding: 0;
  }

  input[type=submit] {
    float: right;
    width: 5em;
  }

  input[type=search] {
    width: 100%;
  }
}
```

## 4. Express Intention

At the core of the previous tips is this: write styles that most closely and
*clearly express the intention of the implementation*. This is my single,
overarching principle for writing readable CSS.

For example, let’s say we have a `heading` and a `blockquote` and we want the
`heading` to be larger and the `blockquote` to be smaller than the body copy.
Using pixels, we could write:

``` css Font sizes using fixed units (px)
h1 {
  font-size: 32px;
}

blockquote {
  font-size: 14px;
}
```

While this will work pretty well for our small, local implementation, the
styles themselves live in isolation and they don’t communicate to the reader
how they relate to one another. In this case, using a relative unit like ems
allows us to define how these styles relate to each other and communicate that
what we really want is for the heading to be twice as big as the body copy and
for blockquotes to be slightly smaller than the body copy:

``` css Font sizes using relative units (em)
h1 {
  font-size: 2em; /* 32px / 16px */
}

blockquote {
  font-size: .875em; /* 14px / 16px */
}
```

Similarly, styles that use `inherit` and `rgba` appropriately are often more
intention-expressing and resilient.

``` scss Using inherit in SCSS
.masthead {
  background-color: #000;
  color: #fff;

  a {
    color: inherit;
  }
}
```

Class and ID names that are consistently formed and semantically meaningful go
along with the spirit of this point. Standardize on how to treat spaces. At
Causes, we use hyphens, but underscores and camelCasing are also reasonable
choices if you are consistent. Tend to pick names that describe the purpose or
meaning of the content, and not how it happens to appear. For instance,
`.local-navigation` is a better name than `.left-column`, especially when you
start using media queries to reflow major elements or decide that your local
navigation should be on the right side one day.

## 5. Be Less Specific

Ideally, styles behave consistently across your site. Behavioral consistency
allows developers to quickly and easily understand the fundamentals of your
classes and apply them confidently in new situations.

One way to achieve this consistency is by defaulting to [less-specific
selectors](http://www.stuffandnonsense.co.uk/archives/css_specificity_wars.html)—write
shorter selector chains, or if using Sass use minimal nesting. Less-specific
selectors allow you to define general style patterns that are easier to reuse,
override where necessary, and may be more performant. Qualifying an ID with any
preceding selectors is a big [code
smell](http://en.wikipedia.org/wiki/Code_smell).

This means that we should tend to prefer classes over IDs and avoid
`!important`. Strive for a flatter structure—the worst I’ve seen is a
stylesheet whose structure directly mirrored the markup it was being applied
to. Overly-qualified selectors are more susceptible to the whims of markup
changes; the structure of our stylesheets should be as markup-agnostic as
possible. Just keep things simple.

Additionally, stylesheets and markup are two sides of the same coin—many
styling problems are actually markup problems in disguise, and solving these
types of problems on the CSS side means you now have two problems. Sometimes a
little markup simplification can go a long way toward fixing a styling issue.

## 6. Love the Cascade

Reset stylesheets, such as [Eric Meyer’s Reset
CSS](http://meyerweb.com/eric/tools/css/reset/), can be good because they
cancel out all default styles, putting all elements at a common starting-point.
However, it is important to define good base styles for typographical elements
(e.g. `h1`, `h2`, `h3`, `ul`, `ol`) on top of the defaults so your site can
have some consistency and you avoid repetition. When styling an element, ask
yourself how many of those styles can be applied globally, and when writing a
style that overrides another, ask yourself if your style override can be
implemented more generally or moved up the chain.

At Causes, we were using a reset stylesheet but had failed to set up some
general, global styles. As a result, each time we wanted to use a new heading
on a page, we had to write some styles to make it look like the headings on
other types of pages. Not only did this slow development, but over time these
styles became inconsistent as some were modified but others were forgotten,
leaving us with a subtly fragmented and unmaintainable visual design. So, we
refactored these styles into sane defaults that are applied sitewide discovered
taht we could move faster and break fewer things.

However, be careful when applying styles to generic container elements (e.g.
`div`, `span`, `section`, `article`, `header`, `footer`, `nav`). For example,
since the `header` element can be easily used in many different contexts, just
like `div`s and `span`s, it might not be a good idea to give them all a certain
background color or font size. This messes up a sane style reset, makes it
difficult to use these general elements in new contexts, and ends up requiring
more complicated overrides that lead to styles with confusing purposes and
tangly interdependencies.

## 7. Leave a Comment

Occasionally, it is not possible to write intention-expressing styles or invent
class names that perfectly encapsulate an idea. These situations may easily
result in unconventional practices or styles whose purposes are not immediately
apparent. Although your code may seem incredibly obvious to you in the moment,
that doesn’t mean it will make sense to you or someone else in the future. When
I find myself in this situation, I leave a little comment to help explain my
intention.

For instance, although !important is best avoided because it violates the
[principle of least
astonishment](http://en.wikipedia.org/wiki/Principle_of_least_astonishment) and
is difficult to override, sometimes it is necessary such as when overriding an
inline style that was added via JavaScript that you don’t have control over.
Whenever you use `!important`, leave a comment explaining why it is important.
This helps prevent a future, well-intentioned developer from coming along and
“fixing” your code. Bonus points for leaving findable keywords in comments,
which help people identify and remove unusual styles as they are no longer
needed.

I typically prefer same-line comments for this purpose because they more easily
stay attached to the style they reference. Sass provides one-line comments:

``` scss Using a comment in SCSS to explain reason for using !important
width: auto !important; // override inline style from JS
```

Standard CSS can still use block-style comments:

``` css Using a comment in CSS to explain reason for using !important
width: auto !important; /* override inline style from JS */
```

## 8. Maintain Order

Similar to structuring our <abbr title="Document Object Model">DOM</abbr> with
an eye toward hierarchy, the order of our selectors should represent some order
of hierarchy or use. For instance, when using Sass at Causes, we follow this
order:

1. Properties (styling the element)
1. Chained pseudo-selectors (styling states of the element)
1. Chained selectors (styling variants of the element)
1. Nested selectors (styling children of the element)

This way, the styles that are most closely related to the element are
vertically closer to the selector itself.

``` scss Different types of selectors in a reasonable order
.cool-link {
  color: #009;

  &:focus,
  &:hover {
    color: #00f;
  }

  &:active {
    color: #006;
  }

  &.featured {
    font-size: 2em;
  }

  .metadata {
    font-size: smaller;
  }
}
```

We also alphabetize our properties. Although it may seem like lukewarm <abbr
title="Obsessive-compulsive disorder">OCD</abbr>, alphabetizing helps us
easily spot duplicates and assists developers in finding the properties they
want to modify, thanks to a standard ordering. As a side-benefit, according to
Google, [properties that consistently appear in the same order will compress
better when
gzipped](https://developers.google.com/speed/docs/best-practices/payload#GzipCompression).
Many modern code editors provide a sorting functionality, which makes
alphabetizing styles a snap. (In Vim command mode I type `vi{:sort`.)

## 9. Use Whitespace Effectively

Whitespace allows us to more clearly see where one selector set ends and
another begins. Indentation helps communicate the hierarchy. Standardize on
these things across your codebase for maximum readability.

You should be minifying and gzipping your CSS already, so there is no penalty
for blank lines in your code.

At Causes, we indent with two spaces, leave a blank line between selector sets
of the same indentation level, and give each selector its own line. These
coding styles not only improve readability of the styles on the filesystem,
they also simplify diffs which eases code review and merge conflict resolution.

## 10. Enforce Coding Style

Maintaining a readable codebase can be challenging if not everyone on your team
has fully bought into these ideas. To help these tactics permeate our team and
persist throughout our codebase, we lean heavily on **automated linting** and
**code review**.

A linter serves as a low-cost first pass to catch most of the basic issues. At
Causes, we are developing a [SCSS linter](https://github.com/causes/scss-lint)
(pull requests welcome) that runs as one of our [Git pre-commit
hooks](https://github.com/causes/git_hooks). We have trained this linter to
enforce basic coding styles such as using one line per selector and
alphabetizing properties. Also, a tool like
[EditorConfig](http://editorconfig.org/) might be worth taking for a spin.

For problems that are more nuanced and difficult to detect through static
analysis, such as how well a set of styles expresses the intention of the
implementation, code review serves as a valuable second pass. When reviewing
stylesheet changes, I try to identify styles whose purpose is not immediately
apparent and explain some small steps that can be taken to make it better.

---

I hope that this article has informed your underlying philosophy and will
assist you on your journey of writing beautiful code. There are plenty of other
relevant areas that I did not touch upon, such as the structure and
organization of your files, but I hope that what I did cover can be part of a
strong and sustainable foundation for your stylesheets.

CSS is like storytelling. There are often many ways to achieve a desired
outcome; some are more coherent than others.
