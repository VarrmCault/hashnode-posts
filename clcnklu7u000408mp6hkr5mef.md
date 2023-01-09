# My go-to SCSS toolbox for responsive design

As a junior developer, I've used Bootstrap's grid system *a lot* to structure my web pages and the breakpoint classes were magic to my beginner self. No need to say that I barely knew the difference between margin and padding; I completely relied on Bootstrap classes and examples to build my templates.

While class-based grid systems like Bootstrap's are still around and used by many developers, I would never go back to building templates using predefined responsive classes.

### Why I dislike quick styling classes

Let's start by clarifying that there is nothing wrong with the quick styling classes. They are useful to add some style to an HTML element without writing your own classes that would do the same thing. It's not what they do, it's **how you use them**.

What I've seen in the many projects using Bootstrap or similar toolkits is what I like to call **the spaghetti syndrome**. Take this code from Bootstrap's documentation for example:

```xml
<div class="container">
  <div class="row">
    <div class="col-6 col-sm-3">.col-6 .col-sm-3</div>
    <div class="col-6 col-sm-3">.col-6 .col-sm-3</div>

    <div class="w-100"></div>

    <div class="col-6 col-sm-3">.col-6 .col-sm-3</div>
    <div class="col-6 col-sm-3">.col-6 .col-sm-3</div>
  </div>
</div>
```

So far so good. We are building a grid design with no homemade CSS, just a few classes from Bootstrap and we're done. But what happens when you need *more* styling and handle *more* screen sizes? Likely, you will just add the required classes to your `div`. Or at least this is what a lot of developers do because it's easier than rewriting something from scratch and/or they lack the CSS skills to do so easily. Here is another example with a more complex responsive design but still quite basic:

```xml
<div class="container">
  <div class="row">
    <div class="col-sm-5 col-md-6">.col-sm-5 .col-md-6</div>
    <div class="col-sm-5 offset-sm-2 col-md-6 offset-md-0">.col-sm-5 .offset-sm-2 .col-md-6 .offset-md-0</div>
  </div>
  <div class="row">
    <div class="col-sm-6 col-md-5 col-lg-6">.col-sm-6 .col-md-5 .col-lg-6</div>
    <div class="col-sm-6 col-md-5 offset-md-2 col-lg-6 offset-lg-0">.col-sm-6 .col-md-5 .offset-md-2 .col-lg-6 .offset-lg-0</div>
  </div>
</div>
```

Now imagine that you add more content with classes for styling (colors, borders), layout (flex, spacing) or any other quick style Bootstrap has to offer and here you go: your template turned into a whole plate of spaghetti and has become harder to understand at first glance, not to mention the headache when you have to maintain a whole codebase like this.

### Media queries to the rescue

To be honest, I didn't (really) work on the responsive design of my applications until a couple of years ago. I did make it nice and friendly with flexbox and CSS grids, but none of the apps I've been working on were planned to be used on smaller devices like tablets and smartphones.

Flexbox and CSS grids are the basis of responsive design but they won't allow you to achieve a nice desktop-to-mobile design on their own. What you need is the little extra provided by CSS media queries: a tool to help you reconfigure your layout based on the device screen size. Combined with flexbox and grids, you can now build a neat responsive design for your application that looks nicer than a simple 12-column layout. As a reminder, a standard media query looks like this:

```css
/* If the screen size is 600px wide or less, hide the element */
@media (max-width: 600px) {
  .my-element {
    display: none;
  }
}
```

Among other things, you can define a max-width, a min-width or a combination of both. However, you'll probably want to work with specific reusable breakpoints instead of hard-coded pixel values.

### Use SCSS mixins to reuse media queries

The first thing to do is to define the target screen sizes you want to use as breakpoints. It is very similar to Bootstrap's breakpoints except that you have full control over how many breakpoints you need; you can also start with Bootstrap's breakpoints which should match a standard set of screen sizes. Here are the ones I like to use based on articles I've read about the different screen sizes available out there:

```css
$breakpoints: (
  desktop-lg: 1920px,
  desktop-sm: 1280px,
  tablet-landscape-lg: 1280px,
  tablet-landscape-sm: 1024px,
  tablet-portrait-lg: 800px,
  tablet-portrait-sm: 768px,
  mobile-lg: 375px,
  mobile-md: 360px,
  mobile-sm: 320px,
);
```

Your breakpoints can be named as you want, I just like to have a quick idea about the screen size I'm targeting without having a look at the breakpoint definition.

Now that the breakpoints are defined we can add the media query mixins. The first one you will need is a media query that applies CSS to devices **above or** **below a certain screen size**. The "above" mixin looks like this:

```css
// Screen >= breakpoint
// Usage: @include media-above(desktop-sm)
@mixin media-above($breakpoint) {
  @if map-has-key($map: $breakpoints, $key: $breakpoint) {
    $screenSize: map-get(
      $map: $breakpoints,
      $key: $breakpoint,
    );

    @media (min-width: $screenSize) {
      @content;
    }
  } @else {
    @warn 'Invalid breakpoint name: #{$breakpoint}';
  }
}
```

We first check if the breakpoint has been defined using `map-has-key` then retrieve the matching screen size. After that, the mixin only creates the appropriate media query and takes advantage of `@content` to say that whatever CSS definition was included in the mixin will belong to the media query. The "below" mixin is the same except that we constrain the media query with `max-width` instead of `min-width`.

You can now include the file containing your mixins wherever you need media queries and use them like this:

```css
@import 'responsive-mixins';

.burger-menu {
    width: 30px;
}

@include media-above(mobile-lg) {
    /* Hide burger-menu for tablet and desktop */
    .burger-menu {
        display: none;
    }
}
```

There are other useful mixins you can define to create quick media queries:

* `media-between`
    
* `mobile` : shortcut for `media-below(tablet-portrait-sm)`
    
* `tablet`: shortcut for `media-between(tablet-portrait-sm, desktop-sm)`
    
* `desktop`: shortcut for `media-above(desktop-sm)`
    

### Final result

**Side note**: if you already use the latest Bootstrap in your project and would only like to get rid of the thousands of template classes, you can check [Bootstrap's documentation about media queries](https://getbootstrap.com/docs/5.3/layout/breakpoints/#media-queries) which is really close to the homemade mixins shown here. If you use any other CSS framework, check their documentation to know if they offer equivalent mixins.

Here is the file containing all of the mixins and breakpoints mentioned above:

```css
$breakpoints: (
  desktop-lg: 1920px,
  desktop-sm: 1280px,
  tablet-landscape-lg: 1280px,
  tablet-landscape-sm: 1024px,
  tablet-portrait-lg: 800px,
  tablet-portrait-sm: 768px,
  mobile-lg: 375px,
  mobile-md: 360px,
  mobile-sm: 320px,
);

// Screen >= breakpoint
// Usage: @include media-above(desktop-sm)
@mixin media-above($breakpoint) {
  @if map-has-key($map: $breakpoints, $key: $breakpoint) {
    $screenSize: map-get(
      $map: $breakpoints,
      $key: $breakpoint,
    );

    @media (min-width: $screenSize) {
      @content;
    }
  } @else {
    @warn 'Invalid breakpoint name: #{$breakpoint}';
  }
}

// Screen < breakpoint
// Usage: @include media-below(desktop-lg)
@mixin media-below($breakpoint) {
  @if map-has-key($map: $breakpoints, $key: $breakpoint) {
    $screenSize: map-get(
        $map: $breakpoints,
        $key: $breakpoint,
      ) -
      1;

    @media (max-width: $screenSize) {
      @content;
    }
  } @else {
    @warn 'Invalid breakpoint name: #{$breakpoint}';
  }
}

// Lower breakpoint <= screen < upper breakpoint
// Usage: @include media-between(desktop-sm,mobile-md)
@mixin media-between($lowerBreakpoint, $upperBreakpoint) {
  $lowerExists: map-has-key(
    $map: $breakpoints,
    $key: $lowerBreakpoint,
  );
  $upperExists: map-has-key(
    $map: $breakpoints,
    $key: $upperBreakpoint,
  );

  @if $lowerExists and $upperExists {
    $lowerScreenSize: map-get(
      $map: $breakpoints,
      $key: $lowerBreakpoint,
    );
    $upperScreenSize: map-get(
        $map: $breakpoints,
        $key: $upperBreakpoint,
      ) -
      1;

    @media (min-width: $lowerScreenSize) and (max-width: $upperScreenSize) {
      @content;
    }
  } @else {
    @if ($lowerExists == false) {
      @warn 'Invalid breakpoint name: #{$lowerBreakpoint}';
    }

    @if ($upperExists == false) {
      @warn 'Invalid breakpoint name: #{$upperBreakpoint}';
    }
  }
}

// Usage: @include mobile { ... }
@mixin mobile() {
  @include media-below(tablet-portrait-sm) {
    @content;
  }
}

// Usage: @include tablet { ... }
@mixin tablet() {
  @include media-between(tablet-portrait-sm, desktop-sm) {
    @content;
  }
}

// Usage: @include desktop { ... }
@mixin desktop() {
  @include media-above(desktop-sm) {
    @content;
  }
}
```

### Before we say goodbye

I want to conclude with another reminder that using pre-made CSS classes is perfectly fine - why would you write a custom class to put Flexbox on a div when someone has already written one for you? Just be careful with *how much* you use them to build your layout as a good responsive design will probably require you to add a lot of them to have a nice user experience.

Don't be afraid to get your hands dirty with SCSS/LESS, this is a good way to improve your skills while also having full control over your CSS!