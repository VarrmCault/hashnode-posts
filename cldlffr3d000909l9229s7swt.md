# Dealing with view encapsulation in Angular

View encapsulation is a mechanism that encapsulates all of your component's styles within the component's host, meaning that any style you wrote for this component won't affect the rest of your application. You don't have to worry anymore about wrapping your styles with a unique class name to avoid side effects that would break your design.

However, view encapsulation comes with drawbacks when you are working with nested components. Because every component has encapsulated styles, you are not able to change any style that isn't directly written in your template and this is something every developer has to deal with at some point. This article will cover the workarounds to style nested components.

### View encapsulation and nested component styling

Let's take an example:

```xml
<!-- ParentComponent template -->
<div>
    <h1>Some title</h1>

    <div class="thumbnails">
        <app-card
            *ngFor="let card of cards"
            [thumbnail]="card.thumbnail"
            [description]="card.description"
        ></app-card>
    </div>
</div>

<!-- CardComponent template -->
<div class="card-wrapper">
    <img [src]="thumbnail.src" />
    <div class="description">{{ description }}</div>
</div>
```

Let's say that you would like to change the `card-wrapper` style and increase the `description` font size when the card component is used in ParentComponent. Here is what you could write if you had direct access to the template:

```scss
/* ParentComponent stylesheet */
.thumbnails {
    padding: 1rem;
}

.card-wrapper {
    min-width: 500px;

    .description {
        font-size: large;
    }
}
```

ParentComponent and CardComponent are different components, which means that their styles are encapsulated and the CSS code above would not change the style of the card subcomponents.

### The soon-to-disappear ::ng-deep

One of the most common ways to solve the encapsulation issue is to use the `::ng-deep` selector. This selector lets you disable encapsulation for specific rules. Here is what our CSS would look like with ng-deep:

```scss
/* ParentComponent stylesheet */
.thumbnails {
    padding: 1rem;
}

:host ::ng-deep .card-wrapper {
    min-width: 500px;

    .description {
        font-size: large;
    }
}
```

The `.thumbnails` rule is still encapsulated within ParentComponent but the `.card-wrapper` rule is now acting like a global style. Note that **you must use the** `:host` **selector before** `::ng-deep` otherwise the change will be applied to every single CardComponent, including the ones that aren't descendants of ParentComponent.

There is one major issue with this solution though: **the ::ng-deep pseudo-class is deprecated and Angular will eventually drop its support**. To be fair, it has been deprecated for a few years now and to my knowledge, there isn't an official replacement yet. It is still working with Angular 15 but I would not recommend using a deprecated solution.

### Change the encapsulation status of your component

Another way to solve this is to change the way Angular encapsulates the styles for this component. If you didn't already notice, the `@Component` decorator lets you fill a property called `encapsulation` which is where you'll be able to deactivate encapsulation.

This property accepts 3 possible values :

* `ViewEncapsulation.Emulated` : Angular will modify your CSS selectors to ensure a style will only be applied to this particular component. It emulates your browser's [Shadow DOM API](https://developer.mozilla.org/en-US/docs/Web/Web_Components/Using_shadow_DOM); it's the default setting for component encapsulation. If you inspect the generated DOM, you will see that Angular added custom attributes to your HTML tags and those attributes are used to create unique CSS selectors like this one:
    

```scss
[_nghost-hhg-c317] mat-drawer-content[_ngcontent-hhg-c317] .main-content[_ngcontent-hhg-c317] {
    flex: 1;
}
```

* `ViewEncapsulation.ShadowDom` : Angular will use the browser's Shadow DOM API instead of emulating its behavior. Because [not all browsers support Shadow DOM](https://caniuse.com/shadowdomv1), the emulated solution is a better choice unless you target specific browsers.
    
* `ViewEncapsulation.None` : Angular will not apply any encapsulation to the SCSS files declared in the component. The styles will be available to the whole application and you must be careful with the selectors you use to avoid unexpected side effects.
    

You can take advantage of `ViewEncapsulation.None` to change the nested component's style:

```typescript
@Component({
  selector: 'app-parent',
  templateUrl: './parent.component.html',
  styleUrls: ['./parent.component.scss'],
  encapsulation: ViewEncapsulation.None,
})
```

```scss
/* ParentComponent stylesheet */

/* .thumbnails is now applied everywhere in the application */
.thumbnails {
    padding: 1rem;

    .card-wrapper {
        min-width: 500px;

        .description {
            font-size: large;
        }
    }
}
```

The main advantage of this solution is that it is easy to set up and it keeps your global stylesheet clean. However, you'll have to remember to check for unencapsulated components and find unique selectors to avoid unwanted styles elsewhere in your app.

I am not very fond of disabling encapsulation and would rather have a single place to look for global styles, but it's a decent solution for simple and isolated scenarios.

### Change the global styles of your application

Since encapsulation is the issue here, changing the global stylesheets is a logical option. Be sure to specialize your selectors to target the correct elements with your styles:

```scss
/* Global styles.scss stylesheet */

app-parent-component .thumbnails {
    padding: 1rem;

    .card-wrapper {
        min-width: 500px;

        .description {
            font-size: large;
        }
    }
}
```

This isn't the prettiest solution for such a specific and simple case, but this would be the way to go for more framework-related generic styling issues like fonts, margins, etc.

Also, remember that `styles.scss` is just the *default* global stylesheet created by Angular CLI. If you'd like to keep things clean and separate, you can always add more global stylesheets to your `angular.json` configuration.

### Take advantage of CSS variables

CSS variables are a must when you design Angular components. If defined properly, they allow you to change the style without breaking style encapsulation. They will only require you to think about the properties that should be allowed to be reconfigured when you design child components.

Here is what CardComponent should look like:

```scss
/* CardComponent stylesheet */

.card-wrapper {
    /* Basic CSS variable, does not set the property if the
       variable is missing */
    min-width: var(--card-min-width);

    .description {
        /* CSS variable with a default value */
        font-size: var(--card-description-font-size, 1em);
    }
}
```

The style has been adapted to ParentComponent's requirements but you get the idea: if you define styles in a component that should be reused, use CSS variables with default values on meaningful properties.

We can now rewrite ParentComponent's style to declare the variables:

```scss
/* ParentComponent stylesheet */

:host {
  --card-min-width: 500px;
  --card-description-font-size: large;
}

.thumbnails {
    padding: 1rem;
}
```

Note that with CSS variables, the parent doesn't even need to know the structure of the child component.

The major downside of CSS variables is that the whole thing has to be planned beforehand. You can of course update your custom components whenever there is a need for variables, but there is nothing you can do if the library or framework you are using does not use CSS variables to style their components. You will have to resort to the alternative solutions mentioned above.

### Few last words

There is no perfect solution to solve the encapsulation issues. All the solutions above have their pros and cons and you'll have to find the one that works best for you in your specific situation.

There might be better solutions that I don't know about (yet) but I thought I would share what I've learned so far to work around nested component styling.