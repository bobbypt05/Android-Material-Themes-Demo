#Android Material Themes Made Easy

###Styles And Themes
Android was built with the concept that almost every user interface element of an app should be customizable, allowing apps to adjust their look and feel to express their brand all the while keeping the common behaviors that users already understand. To support this, Android has [styles](http://developer.android.com/guide/topics/resources/style-resource.html), which are defined in XML as collections of attributes that can be applied to [`Views`](http://developer.android.com/reference/android/view/View.html). A single style can adjust colors, dimensions, spacing, fonts, and any other customizable attributes that a `View` supports. By grouping these attributes together, a single style can be applied to many `Views`. If that style is updated, all of the `Views` using the style will update as well, without needing to adjust each and every `View`. Styles can also extend from other styles, with the ability to override and add new attributes to create multiple variations of a style. 

Now that we understand the basics of styles, what are themes in Android? Ironically enough, themes are actually just styles. So, what's the difference then? Really, it comes down to how styles and themes are used. Themes are applied to an [`Activity`](http://developer.android.com/reference/android/app/Activity.html) or an entire [`Application`](http://developer.android.com/reference/android/app/Application.html), which applies the style attributes to every `View` in the `Activity` or `Application` layout hierarchy. Since they are applied to every `View`, themes are much more broad, and cannot be as specific as styles, and many times themes will set styles for specific types of `Views`. The other big difference is that themes can assign global attributes that can be used within the layouts of the `Activity` or `Application` they are applied to, which we will cover later. So to sum it up, a style is a collection of attributes applied at a local, individual component level, while themes are applied at a per page or entire application level.

###Understanding Attributes
So we know that styles and themes set attributes, but wait, what really is an [attribute](http://developer.android.com/training/custom-views/create-view.html#customattr)? Basically, you can think of attributes just like variables, that represent properties that can be set, changed, and applied to the user interface to define look and behavior. Attributes are defined in XML as a certain format, such as a boolean, a number, text, an enum, or even a reference to another resource. These attributes can then be referenced when defining user interface elements in code or XML. The Android SDK defines its own attributes under the `android:` namespace, which are used by the default user interface elements and themes. the full list be found in [`R.attr`](http://developer.android.com/reference/android/R.attr.html). These default Android attributes are useful for styling the default user interface, but apps can add their own custom attributes in `attrs.xml` to be used with custom themes and `Views`. 

Now let's start with an example of how an attribute can be used. The simplest use case for an attribute is an explicit property that a `View` will apply. For example, `TextView` supports the attribute `android:textColor`, which sets a specific color for its text. So keeping the "*attributes are like variables*" comparison, in Java terms, we are basically setting the `textColor` variable to a value.

```java
int textColor = 0x00FF00; 
```

In XML, we would create a simple text style that sets `android:textColor` to a color. This style would then be applied to `TextViews` in an app, and if the standard text color needed to change, only the style would need to be updated, rather than every layout.

```xml
<style name="TextStyle" parent="TextAppearance.AppCompat">
	<item name="android:textColor">#00FF00</item>
</style>
```

Just like variables, attributes can reference other attributes. This enables a more complex use for attributes, where they do not correspond to explicit `View` properties, but are instead global theme properties that can be referenced by other attributes that do set explicit `View` properties. For example, the default Android theme sets the attribute `android:textColorPrimary`, which is a color, but not an explicit property of any `View`. However, it can be used by `TextView` and applied to the attribute `android:textColor`. Going back to the "*attributes are like variables*" comparison, in Java terms, we are now setting the `textColorPrimary` variable to a value, which is then being set to `textColor`.

```java
int textColorPrimary = 0x00FF00;
int textColor = textColorPrimary;
```

In XML, we would use a theme that sets `android:textColorPrimary` to a color, and a text style that sets `android:textColor` to `?android:textColorPrimary`. Again, this style could then be applied to `TextViews` in an app, but now if the text color needed to change, the theme can be updated, or another theme that sets `android:textColorPrimary` could be applied in XML or at runtime, without needing to change any style or layouts.

```xml
<style name="AppTheme" parent="Theme.AppCompat.Light.DarkActionBar">
	<item name="android:textColorPrimary">#00FF00</item>
</style>

<style name="TextStyle" parent="TextAppearance.AppCompat">
	<item name="android:textColor">?android:textColorPrimary</item>
</style>
```

Now you should see the delineation between styles and themes starting to surface with how they use attributes. Styles are great for applying explicit `View` attributes, and directly applying them to `Views`. Likewise, themes can set global theme attributes, which can then in turn be used by styles and `Views`. Putting this all together, themes can create a palette of theme attributes, which styles use to apply to explicit `View` attributes. This way, themes set the overall brand for an app, while styles use that brand to create the look and feel in variations that are applied to  `Views`. The intention here is to make user interfaces more module, where the "skinning" of an app is in the themes and styles, and the structure and the content is in the layouts. This makes it possible to adjust each independently from one another, making future updates to the user interface much simpler.

###Raster Graphics And Asset Overload
Themes and styles seem like the simple solution to make a completely dynamic user interface that can be updated and maintained with ease. This all sounds almost too good to be true, so what's the catch? Well, before Android 5.0 (API 21), themes were useful for settings backgrounds, layout spacing and sizes, font sizes and colors, divider sizes and colors, and other basic elements of the user interface. However, the primary user interface elements to apply styles to (buttons, text boxes, switches, images, icons, etc) were much more difficult to stylize. This was because these elements were [raster graphics](https://en.wikipedia.org/wiki/Raster_graphics) (png, jpeg, etc), which have set dimensions and colors, and are not dynamic in any way. This meant a button could not just reference a theme color and colorize itself. Instead, a colored version of the button had to be created and included as a resource in the app. In order to handle every button state (5) and screen density bucket (6) that Android supports, 30 (5x6) assets would need to be generated for that single button. That's just one colored button, if a new color of the button was needed, another set of assets was needed. Even if the color of an existing button needed to be adjusted, the full set of assets would need to be replaced. So you can see how this quickly became a very tedious chore, making it difficult to update and maintain themes and styles for control elements, since their assets were not dynamic in any way.

Wait, could something other than raster images be used for these user interface elements? Android does support [`ShapeDrawables`](http://developer.android.com/reference/android/graphics/drawable/ShapeDrawable.html), which are vector shape assets that can scale without losing fidelity, removing the need for multiple density versions for an asset. They can also reference color and dimension resources, so why not just reference theme attributes from a `ShapeDrawable`? Unfortunately, before Android 5.0, referencing theme attributes in `ShapeDrawables` would compile just fine, but throw an `Exception` at runtime. `ShapeDrawables` could get closer to a more dynamic set of user interface elements, but ultimately had a similar limitation as raster images, where a separate asset for each color was needed. This allowed for slightly easier theming of controls, but still took a lot of work.

###Theming In A Material World
Google was aware of these painpoints when theming and styling apps, and in Summer 2014, they unveiled [Material Design](https://www.google.com/design/spec/material-design/introduction.html) alongside Android 5.0. The material theme was a strong divergence from the Holo theme that had been around since Fall 2011 with Android 4.0 (API 14). Material design added new animations, user interface controls, touch feedback, and a bright new color palette. But more importantly, Android 5.0 added new features critical to theming almost every aspect of Material user interfaces.

1. [Color tinting](http://developer.android.com/training/material/drawables.html#DrawableTint)  support for `Drawables` and `Views`
2. Theme attribute access in [Drawable Resources](http://developer.android.com/guide/topics/resources/drawable-resource.html)
3. Ability to theme individual `Views`

This finally opened the door to dynamically colorize user interface elements without having to make multiple versions for each color. Color tinting allowed icons and images to be colored on the fly, while `ShapeDrawables` could reference color theme attributes to dynamically color user interface elements. Lastly, being able to apply themes at the `View` level allowed `Views` in the same layout to use completely different themes. 

This all sounded great, but what about legacy devices that didn't have the new features of Android 5.0? Google's answer was the [v7 AppCompat Support Library](http://developer.android.com/tools/support-library/features.html#v7-appcompat), which included "tint-aware" Material Design user interface elements that supported all the way back to Android 2.1 (API 7).

###AppCompat 101
The AppCompat Support Library is very powerful, and full of useful features. However, it comes with less than sparse documentation, so figuring out how to effectively use it is another story.  ---

---In short, if you setup your themes and styles using the concepts discussed above, AppCompat will make using Material Design in your apps a breeze.

--- 


###[REMOVE] Raster Graphics And Asset Overload
First, what's a [raster graphic](https://en.wikipedia.org/wiki/Raster_graphics)? Simply put, it's a rectangular matrix of dots, with each dot representing a color. Common examples are [`.jpg`](https://en.wikipedia.org/wiki/JPEG) and [`.png`](https://en.wikipedia.org/wiki/Portable_Network_Graphics). Raster graphics can be scaled down without much image degradation, but when scaling them up they become blurry and pixelated. Generally it is good practice to always create raster graphics at or above the maximum resolution that they will be used at, to ensure that they will only ever be scaled down. Android supports a wide array of [screen densities](https://www.captechconsulting.com/blogs/understanding-density-independence-in-android), and in order to do so, properly scaled versions of every raster graphic is needed for each support screen density. 

Android includes a stock set of user interface elements, including buttons, text boxes, switches, tab bars, drop down menus, and more. These elements are replaceable, and the default style for each element can be set in the theme to use a custom version. So, what do we need to replace something as simple as a [`Button`](http://developer.android.com/reference/android/widget/Button.html)? First, you need stretchable [9-patch drawables](http://developer.android.com/guide/topics/graphics/2d-graphics.html#nine-patch), one for each interactive state that `Button` supports. 

1. Default
2. Focused
3. Pressed
4. Disabled
5. Disabled + Focused

Only 5 assets, not so bad. But wait, there's more! Remember that if we are using raster graphics, Android needs scaled versions of *every* raster graphic to support those different screen densities.

1. Low (`ldpi` / 120dpi) 
2. Medium - (`mdpi` / 160dpi) 
3. High - (`hdpi` / 240dpi) 
4. Extra High - (`xhdpi` / 320dpi) 
5. Extra Extra High - (`xxhdpi` / 480dpi) 
6. Extra Extra Extra High - (`xxxhdpi` / 640dpi) 

These days, `ldpi` devices are pretty much extinct, and `xxxhdpi` devices haven't become prominent enough to support just yet. So, if we create our 5 state images at the 4 common screen densities, we are now up to 20 assets, for a single `Button`. And we still aren't done yet, we need a [`StateListDrawable`](http://developer.android.com/guide/topics/resources/drawable-resource.html#StateList) to combine all of the state images into a single image asset that we can reference. At that point, then we can replace the default `Button` style with our custom image asset.

Now, let's say we need another `Button` that is a slightly different color than the first button, but is otherwise identical. Sorry, you have to make 20 more assets just for that `Button`. What if you want to make the corners of the first `Button` a little smoother? You guessed it, you need to recreate all 20 assets and replace every one of them.
