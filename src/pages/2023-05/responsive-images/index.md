---
setup: |
  import Layout from '/src/layouts/BlogPost.astro'
  import Tangent from "/src/blogComponents/lib/Tangent.astro"
title: "How To Speed Up Page Load With Responsive Images"
date: "2023-05-08"
description: "Image loading is a major factor in page load times which is why it is important to use responsive images to minimize the amount of data that is transferred to users."
tags: ["HTML"]
---

Making sure your images look good on all screen sizes is a difficult task since you need to consider the size of the image, where the image is placed, how much of the image is shown, the speed of the user's connection, and so much more. What ends up happening is most developers will just use the same image for all screen sizes and let the browser resize the image to fit the screen. This is a bad practice because the browser will still download the full size image (which is generally very large) even if it is only being displayed at a fraction of its size. This is a waste of your user's bandwidth and will slow down your page load times significantly (especially on slower connections).

The solution to this problem is to use responsive images. Responsive images are images that are optimized for the user's screen size. This means that the image will be downloaded at the correct size and quality for the user's device. This will significantly reduce the amount of data that is transferred to the user and will speed up your page load times. There are many ways to implement responsive images, from simple to complex, and in this article I will be showing you all the ways you can render responsive images on your website.

## `img` `srcset` Attribute

By far the easiest way to implement responsive images is to use the `srcset` attribute on the `img` tag. This attribute allows you to define multiple different sized images and the browser will then automatically choose the image that best fits the user's screen size.

```html
<img
  src="tree-1200.jpg"
  alt="A tree"
  srcset="tree-400.jpg 400w, tree-800.jpg 800w, tree-1200.jpg 1200w"
/>
```

This code may look confusing so let me explain exactly what is happening. To get started we have the normal `src` and `alt` attributes that you are already used to with all images. In the rare case that the browser your user is using does not support `srcset` the `src` url will be used for the image instead. This is incredibly unlikely, though, as `srcset` has been supported in all major browsers for 5-10 years.

The confusing attribute is `srcset`. This attribute accepts a comma separated list of image urls and their widths. If we look at the first item in the list `tree-400.jpg 400w` you can see that the url is `tree-400.jpg`. The name of this url does not matter, but generally when you have multiple of the same image at different sizes you will want to name them with the size in the name.

The second part of this item is `400w`. This is probably confusing as `w` is a not a CSS unit, instead `w` stands for the intrinsic width of the image in pixels. This is the actual width of the image file. You can easily find this width by inspecting the image in your file browser/explorer. If you are on Windows you can right click the image and select `Properties` and on a Mac there should be an option called `Get Info`. In this case the image is 400 pixels wide so we set the width to `400w`.

The browser will then use this information to automatically determine which image to download. For example, if the user's screen is less than 400px wide it will use the `tree-400.jpg` image as that is the smallest image that can be used without any stretching/blurring of pixels. Once the browser becomes larger than 400px the browser will switch to using the `tree-800.jpg` image. This is because the 400px image is now smaller than the current screen size and would be stretched/blurred if used. This would continue for all screen sizes until the browser reaches the largest image listed.

This is great because now on small screen sizes the browser will download a smaller image while large screen sizes will still get a high resolution image. This will significantly reduce the amount of data that is transferred to the user and will speed up your page load times. Below is an example of what this looks like. Try changing your browser size to a small size and then reloading the page to see that the smaller image is downloaded.

```html
<img
  style="width: 100%; border-radius: 1rem;"
  src="https://placehold.co/3200x800/png"
  srcset="
    https://placehold.co/800x200/png   800w,
    https://placehold.co/1600x400/png 1600w,
    https://placehold.co/3200x800/png 3200w
  "
/>
```

<img
  style="width: 100%; border-radius: 1rem;"
  src="https://placehold.co/3200x800/png?v=1"
  srcset="
    https://placehold.co/800x200/png?v=1   800w,
    https://placehold.co/1600x400/png?v=1 1600w,
    https://placehold.co/3200x800/png?v=1 3200w
  "
/>

<Tangent>
  When doing your testing you may have noticed that the image downloaded was actually larger than you expected. For example, if your screen width was 700px your browser may still have downloaded the 1600px wide image instead of the 800px wide image. This is because the browser also takes into account the pixel density of your screen. If you are on a high resolution device or your have a high zoom level on your browser the browser will download a larger image to make sure it looks good on your screen since each CSS pixel actually corresponds to multiple pixels on your screen. To check your devices pixel density you can use <code>window.devicePixelRatio</code> in the console.
</Tangent>

### How To Handle Different Pixel Densities

Sometimes you have an image that will always be the same size on the screen, but you want to make sure it looks good on high resolution devices. For example, if you have a logo that is always 100px wide it will look blurry on high resolution devices if you only provide a 100px wide image. To handle this you can use the `srcset` attribute to provide multiple images of sizes using the `x` unit to denote pixel density.

```html
<img
  src="logo-200.jpg"
  alt="Our Logo"
  srcset="logo-100.jpg 1x, logo-150.jpg 1.5x, logo-200.jpg 2x"
/>
```

The above code is very similar to our previous `srcset` example, but the major difference is we are using units like `1.5x` and `2x` instead of hard coded pixel values. This unit refers to the pixel density of the screen. For example if someone's screen has a pixel density of 1.25 device pixels per CSS pixel the `logo-150.jpg` image will be used since that is the smallest image that can be used without stretching/blurring pixels.

<Tangent>
  You don't need to include the <code>1x</code> unit as that is the default. If you only have 2 images you can use <code>logo-100.jpg, logo-200.jpg 2x</code> instead of <code>logo-100.jpg 1x, logo-200.jpg 2x</code>.
</Tangent>

## `img` `sizes` Attribute

What we have covered so far is the most basic way to implement responsive images, but in many scenarios your image size is not actually the same as the width of your screen. This blog is a good example of that. On small screen sizes the content in my blog (including images) takes up the full width of the screen, but on larger screen sizes I have the content centered on the page with a limited maximum width. If we only used `srcset`, like above, our images would scale based off the full size of the browser window which would result in the images being larger than needed on large screen sizes. This is where the `sizes` attribute comes in.

The `sizes` attribute lets you define either a single size for your image, such as 50vw, or a list of media queries that will be used to determine which size your image should be. By default when you do not add the `sizes` attribute to your `img` it assumes a size of `100vw` which is why the images above scaled off the full width of the browser window. Let's take a look at how we would use the `sizes` attribute to take into account a blog like this that has a maximum size.

```html {5}
<img
  src="tree-1200.jpg"
  alt="A tree"
  srcset="tree-400.jpg 400w, tree-800.jpg 800w, tree-1200.jpg 1200w"
  sizes="(max-width: 800px) 100vw, 800px"
/>
```

The code above is identical to the code from before, but we have added the `sizes` attribute. The `sizes` attribute accepts a comma separated list of media queries and sizes. To understand what is happening let's breakdown each item in the list.

Our first item `(max-width: 800px) 100vw` has two parts. The first part is the media query we want to check against. In this case we are checking if the screen width is less than 800px. The second part is the size we want to use for our image if the media query is true. In this case we are using `100vw` which means we want the browser to choose the image size based off the full width of the browser window.

The second item `800px` does not have a media query and instead just has a size. This is considered our fallback size. If all the media queries defined before this size are false it will use this fallback size instead. Essentially you can think of it as having a media query that is always true. What we are saying with this item is that our image should be selected assuming our image takes up 800px on the screen. The browser will then use this size to determine which image to download. It may download an image larger than 800px if your browser has a high resolution or you are zoomed in on the page, but in general this is a good way of ensuring your images are not larger than needed.

If we take these two items and put them together it is essentially saying that our image should be chosen based on the width of the browser up until 800px. At that point the image will never take up more than 800px on our screen so we should size our image based on that 800px size. This would be how I would write code for adding responsive images to this blog as my blog is limited to a maximum width at larger screen sizes. Let's look at an example of this in action.

```html
<img
  style="width: 100%; border-radius: 1rem;"
  src="https://placehold.co/3200x800/png"
  srcset="
    https://placehold.co/400x100/png   400w,
    https://placehold.co/800x200/png   800w,
    https://placehold.co/1200x300/png 1200w,
    https://placehold.co/1600x400/png 1600w,
    https://placehold.co/3200x800/png 3200w
  "
  sizes="(max-width: 800px) 100vw, 800px"
/>
```

<img
  style="width: 100%; border-radius: 1rem;"
  src="https://placehold.co/3200x800/png?v=2"
  srcset="
    https://placehold.co/400x100/png?v=2   400w,
    https://placehold.co/800x200/png?v=2   800w,
    https://placehold.co/1200x300/png?v=2 1200w,
    https://placehold.co/1600x400/png?v=2 1600w,
    https://placehold.co/3200x800/png?v=2 3200w
  "
  sizes="(max-width: 800px) 100vw, 800px"
/>

I added a bunch of different image sizes so you can see how they work with various pixel densities. If you are on a high resolution device you may notice that the browser downloaded a larger image than 800px. You can also emulate this by zooming your device in as the more zoomed in your device is the higher the pixel density and if you zoom in enough the browser will be required to download a higher resolution image to make sure it looks good on your screen.

### Potential Pitfalls

The `sizes` attributes is incredibly powerful, but there are a few things to be aware of when using it.

#### Order Matters

If you have a `sizes` attribute with multiple media queries the image that is chosen will be the first media query that is true. This means that the order of your media queries matters.

```html {7}
<img
  src="https://placehold.co/3200x800/png"
  srcset="
    https://placehold.co/400x100/png 400w,
    https://placehold.co/800x200/png 800w
  "
  sizes="(max-width: 800px) 100vw, (max-width: 500px) 50vw, 1200px"
/>
```

If you wrote out your `sizes` like above your code would not work as expected. The reason for this is because the first media query `(max-width: 800px) 100vw` will be true for all screen sizes less than 800px. This means that the second media query `(max-width: 500px) 50vw` will never be used since it will only be true when the screen is less than 500px and the first media query will always be true in those size ranges which means it will always be picked first. To fix this you need to reorder your media queries so that the most specific media queries are first and the least specific media queries are last.

```html {7}
<img
  src="https://placehold.co/3200x800/png"
  srcset="
    https://placehold.co/400x100/png 400w,
    https://placehold.co/800x200/png 800w
  "
  sizes="(max-width: 500px) 50vw, (max-width: 800px) 100vw, 1200px"
/>
```

It is also important to make sure that your default size, the one with no media query, is always last since it always evaluates to true so if it is first it will always be picked over any other media query.

#### Using Percentages

So far I have showed you how you can use exact measurements, like `px`, and how to use measurements that scale off the browser window, like `vw`, but what about percentage sizes, like `50%`. Unfortunately percentage sizes are not supported in the `sizes` attribute. The reason for this is that the browser does not know how wide something defined in percentages will be until it knows the width of the parent element. This means that the browser would have to wait until the entire page is loaded before it could determine which image to download. This would be a bad user experience as the user would have to wait for the entire page to load before they could see any images.

## `picture` Element

Up to this point we have specifically talked about how to render the same image at different sizes to help with load times, but this does not cover the scenario where you want to display a different image at different screen sizes. For example, if you had a large header on your page that spanned the entire width of your page you may want to display a different image on mobile than you do on desktop since you can use an image with more detail on desktop. This is where the `picture` element comes in.

The `picture` element lets you define multiple `source` elements that are used to define different images to use at various screen sizes. The browser will then pick the first `source` element that matches the current screen size and use that image. If none of the `source` elements match the current screen size it will use the `img` defined in the `picture` element as a fallback.

```html
<picture>
  <source media="(max-width: 500px)" srcset="hiking-narrow.jpg" />
  <img src="hiking-wide.jpg" alt="Someone jumping on a hike" />
</picture>
```

<picture>
  <source media="(max-width: 500px)" srcset="/articleAssets/2023-05/responsive-images/hiking-narrow.jpg" />
  <img style="border-radius: 1rem; width: 100%;" src="/articleAssets/2023-05/responsive-images/hiking-wide.jpg" alt="Someone jumping on a hike" />
</picture>

If you grow/shrink the size of your browser you should see the image change between the two different versions. If you are on a mobile device you may need to zoom in/out to see the image change. We have a more cropped version of the image for smaller screen sizes since the focal point of the image, the person, becomes too small on smaller screen sizes.

Now let's look at the actual code to see how it works. In order for a `picture` element to work you need at the very least to put a normal `img` tag inside the `picture` element at the very end.

```html
<picture>
  <img src="hiking-wide.jpg" alt="Someone jumping on a hike" />
</picture>
```

Doing this will just render the image like a normal `img` tag. Where this gets more advanced is with the `source` element. Each variation of your image, other than the default version, should have its own `source` element. In our case we had just one `source`, but you could have as many as you need.

```html
<picture>
  <source media="(max-width: 500px)" srcset="hiking-narrow.jpg" />
  <source media="(max-width: 1000px)" srcset="hiking-medium.jpg" />
  <img src="hiking-wide.jpg" alt="Someone jumping on a hike" />
</picture>
```

Inside of each of those `source` elements you have two main attributes. The `srcset` attribute works just like the `srcset` attribute of the `img` tag. This means if we have multiple resolutions of our `hiking-narrow.jpg` image we could include them in the `srcset`. Most likely, though, when you are using the `picture` element you will only have one resolution in each `source` element so you can just put that as the only url in the `srcset` attribute.

The other attribute is the `media` attribute. This works similar to how media queries work in the `sizes` attribute, but with the `source` element `media` attribute you can only define one media query. These queries are then checked from top to bottom just like the `sizes` attribute and the first one that is matched is used. If none of the media queries match then the `img` tag is used as a fallback which is why we don't have a `source` element specifically for larger screen sizes.

### Why Use The `picture` Element Over Alternatives

One big confusion around the `picture` element is why you would want to use it over the `sizes` attribute of the `img` element or over CSS.

#### Why `sizes` Is Not Right For The Job

The main reason to use the `picture` element is that it will always swap to whatever image is defined in the `source` element that matches the current screen size. This means if you change your screen size by zooming or resizing the window it will swap to the correct image.

The `sizes` attribute works similarly, but only when increasing your screen size. If your screen size shrinks the browser will not switch to or download the smaller image as it already has the larger image so it will continue to render that image. This is great since it will save on bandwidth as there is no point in downloading a smaller image when you already have a larger image, but when you want to display different images on different screen sizes this can be a problem which is why the `picture` element is ideal.

#### Why CSS Is Not Right For The Job

If you are familiar with CSS you may realize that we can accomplish a very similar result by just using some simple CSS properties.

```css
img {
  object-fit: cover;
  object-position: center;
}
```

This will make the image fill the entire width of the parent element and then crop the image so that the center of the image is always visible. This will give us very similar results, but the downside is that we still need to download the full resolution version of the image even on small screen sizes where we only show a fraction of it. This goes against everything we are trying to accomplish by using responsive images.

## Conclusion

Responsive images may seem like a complex topic, but it doesn't have to be. Implementing basic responsive images is as simple as adding a `srcset` attribute to your `img` tag and then letting the browser do the rest. If you want to get more advanced you can use the `sizes` attribute to help the browser pick the correct image, or if you want to display different images at different screen sizes you can use the `picture` element.
