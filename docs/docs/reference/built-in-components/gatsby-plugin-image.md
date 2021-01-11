---
title: Gatsby Image plugin
---

This guide will show you how to configure your images, including choosing layouts, placeholders and image processing options. If you would like to learn how to set up the image plugins and use images on your site, see [the how-to guide](/docs/how-to/images-and-media/using-gatsby-plugin-image).

## Components

The Gatsby Image plugin includes two components to display responsive images on your site, used for static and dynamic images.

- **[`StaticImage`](#staticimage):** Use this if the image is the same every time the component is used. _Examples: site logo, index page hero image_
- **[`GatsbyImage`](#gatsbyimage):** Use this if the image is passed into the component as a prop, or otherwise changes. _Examples: Blog post hero image, author avatar_

The following are props that can be passed to the components.

### Shared props

The following props can be passed to both `GatsbyImage` and `StaticImage`. You may also use any valid [`<img>` tag props](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/img#attributes), which are forwarded to the underlying `<img>` element.

| Prop              | Type                                                                        | Default       | Description                                                                                                                                     |
| ----------------- | --------------------------------------------------------------------------- | ------------- | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| `alt` (Required)  | `string`                                                                    |               | Alternative text, passed to the `<img>` tag. Required for accessibility.                                                                        |
| `as`              | `ElementType`                                                               | `"div"`       | The HTML element used for the outer wrapper.                                                                                                    |
| `loading`         | `"eager" \| "lazy"`                                                         | `"lazy"`      | Loading behavior for the image. You should set this to `"eager"` for above-the-fold images to ensure they start loading before React hydration. |
| `className`       | `string`                                                                    |               | CSS class applied to the outer wrapper.                                                                                                         |
| `imgClassName`    | `string`                                                                    |               | CSS class applied to the `<img>` element.                                                                                                       |
| `style`           | `CSSProperties`                                                             |               | Inline styles applied to the outer wrapper.                                                                                                     |
| `imgStyle`        | `CSSProperties`                                                             |               | Inline styles applied to the `<img>` element.                                                                                                   |
| `backgroundColor` | `string`                                                                    | `transparent` | Background color applied to the wrapper.                                                                                                        |
| `objectFit`       | [See doc](https://developer.mozilla.org/en-US/docs/Web/CSS/object-fit)      | `cover`       | Resizing behavior for the image within its container.                                                                                           |
| `objectPosition`  | [See doc](https://developer.mozilla.org/en-US/docs/Web/CSS/object-position) | `50% 50%`     | Position of the image within its container.                                                                                                     |

### `StaticImage`

The `StaticImage` component can take all [image options](#image-options) as props, as well as all [shared props](#shared-props). Additionally, it takes this prop:

| Prop             | Type     | Description                                                                                           |
| ---------------- | -------- | ----------------------------------------------------------------------------------------------------- |
| `src` (Required) | `string` | Source image, processed at build time. Can be a path relative to the source file, or an absolute URL. |

### `GatsbyImage`

This component accepts all [shared props](#shared-props), as well as the one below. These props are passed directly to the component, and are not to be confused with [image options](#all-options), which are passed to the GraphQL resolver when using dynamic images.

| Prop               | Type              | Description                                                          |
| ------------------ | ----------------- | -------------------------------------------------------------------- |
| `image` (Required) | `GatsbyImageData` | The image data object, returned from the `gatsbyImageData` resolver. |

## Image options

There are a few differences between how you specify options for `StaticImage` and `GatsbyImage`:

1. **How to pass options:** When using `StaticImage`, options are passed as props to the component, whereas for the `GatsbyImage` component they are passed to the `gatsbyImageData` GraphQL resolver.
1. **Option values:** In the `StaticImage` component, props such as `layout` and `placeholder` take a _string_, while the resolver takes a _[a GraphQL enum](https://graphql.org/learn/schema/#enumeration-types)_, which is in upper case by convention and is not quoted like a string. Both syntaxes are shown in the reference below.

> It is a very good idea to use [the GraphiQL IDE](/docs/how-to/querying-data/running-queries-with-graphiql) when writing your `gatsbyImageData` queries. It includes auto-complete and inline documentation for all of the options and lets you see the generated image data right inside the IDE.

Both static and dynamic images have the following options available:

| Option                                  | Default string/enum value                                                     | Description                                                                 |
| --------------------------------------- | ----------------------------------------------------------------------------- | --------------------------------------------------------------------------- |
| [`layout`](#layout)                     | `"constrained"` / `CONSTRAINED`                                               | Determines the size of the image and its resizing behavior.                 |
| [`aspectRatio`](#aspectratio)           | Source image aspect ratio                                                     | Force a specific ratio between the image's width and height.                |
| [`width`/`height`](#widthheight)        | Source image size                                                             | Change the size of the image.                                               |
| [`placeholder`](#placeholder)           | `"dominantColor"`/`DOMINANT_COLOR`                                            | Choose the style of temporary image shown while the full image loads.       |
| [`formats`](#formats)                   | `["auto","webp"]`/`[AUTO,WEBP]`                                               | File formats of the images generated.                                       |
| [`transformOptions`](#transformoptions) | `{fit: "cover", cropFocus: "attention"}`/`{fit: COVER, cropFocus: ATTENTION}` | Options to pass to sharp to control cropping and other image manipulations. |

See [all options](#all-options).

### `layout`

The image components support three types of layout, which determine the image sizes that are generated, as well as the resizing behavior of the image itself in the browser.

| Layout      | Component prop value | Resolver prop value | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| ----------- | -------------------- | ------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Constrained | `"constrained"`      | `CONSTRAINED`       | This is the default layout. It displays the image at the size of the source image, or you can set a maximum size by passing in `width` or `height`). If the screen or container size is less than the width of the image, it scales down to fit, maintaining its aspect ratio. It generates smaller versions of the image so that a mobile browser doesn't need to load the full-size image.                                                                                                                                                                                                                    |
| Fixed       | `"fixed"`            | `FIXED`             | This is a fixed-size image. It will always display at the same size, and will not shrink to fit its container. This is either the size of the source image, or the size set by the `width` and `height` props. Only use this if you are certain that the container will never need to be narrower than the image.                                                                                                                                                                                                                                                                                               |
| Full width  | `"fullWidth"`        | `FULL_WIDTH`        | Use this for images that are always displayed at the full width of the screen, such as banners or hero images. Like the constrained layout, this resizes to fit the container. However it is not restricted to a maximum size, so will grow to fill the container however large it is, maintaining its aspect ratio. It generates several smaller image sizes for different screen breakpoints, so that the browser only needs to load one large enough to fit the screen. You can pass a `breakpoints` prop if you want to specify the sizes to use, though in most cases you can allow it to use the default. |

You can compare the layouts in the video below:

<video controls autoplay loop>
  <source type="video/mp4" src="./layouts.mp4" />
  <p>Your browser does not support the video element.</p>
</video>

To set the layout, pass in the type to the layout prop. e.g.

```jsx
<StaticImage
  src="./dino.png"
  alt="A dinosaur"
  // highlight-next-line
  layout="fixed"
/>
```

For a dynamic image, pass it to the resolver:

```graphql
dino {
  childImageSharp {
    # highlight-next-line
    gatsbyImageData(layout: FIXED)
  }
}
```

### `width`/`height`

_Layouts: fixed and constrained_

Size props are optional in `GatsbyImage` and `StaticImage`. Because the images are processed at build time, the plugin knows the size of the source image and can add the correct width and height to the `<img>` tag, so it displays correctly with no layout jumping. However, if you want to change the display size you can use the size options to do this.

For a `fixed` layout, these define the size of the image displayed on screen. For a `constrained` image these define the maximum size, as the image will scale down to fit smaller containers if needed.

If you set just one of these, the source image is resized to that width or height while maintaining aspect ratio. If you include both then it is also cropped if needed to ensure it is that exact size.

### `aspectRatio`

_Layouts: fixed, constrained, and fullWidth_

The `aspectRatio` prop forces an image to the specified aspect ratio, cropping if needed. The value is a number, but can be clearer to express as a fraction, e.g. `aspectRatio={16/9}`

For `fixed` and `constrained` images, you can also optionally pass either a `width` or `height`, and it will use that to calculate the other dimension. For example, if you pass `width={800} aspectRatio={4/3}` then `height` will be set to the width divided by the aspect ratio: so `600`. Passing `1` as the aspectRatio will crop the image to a square. If you don't pass a width or height then it will use the source image's width.

For `fullWidth` images you don't specify width or height as it resizes to fit the screen width. Passing `aspectRatio` will crop the image if needed, and the height will scale according to the width of the screen. For example, if you set the aspectRatio to `16/9` then when the image is displayed full width on a screen that is 1024px wide, the image will be 576 pixels high.

> There are several advanced options that you can pass to control the cropping and resizing behavior. For more details, see the advanced options reference.

### `placeholder`

Gatsby image components are lazy-loaded by default, which means that if they are offscreen they are not loaded by the browser until the come into view. To ensure that the layout does not jump around, a placeholder is displayed before the image loads. There are three types of placeholder that you can choose

| Placeholder    | Component prop value | Resolver prop value | Description                                                                                                                                                                        |
| -------------- | -------------------- | ------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Dominant color | `"dominantColor"`    | `DOMINANT_COLOR`    | The default placeholder. This calculates the dominant color of the source image and uses it as a solid background color.                                                           |
| Blurred        | `"blurred"`          | `BLURRED`           | This generates a very low-resolution version of the source image and displays it as a blurred background.                                                                          |
| Traced SVG     | `"tracedSVG"`        | `TRACED_SVG`        | This generates a simplified, flat SVG version of the source image, which it displays as a placeholder. This works well for images with simple shapes or that include transparency. |
| None           | `"none"`             | `NONE`              | No placeholder. You can use the background color option to set a static background if you wish.                                                                                    |

### `formats`

_Default component prop value: `["auto", "webp"]`. Default resolver prop value: `[AUTO, WEBP]`_

The Gatsby Image plugin supports four output formats: JPEG, PNG, WebP and AVIF. By default, the plugin generates images in the same format as the source image, as well as WebP. For example, if your source image is a PNG, it will generate PNG and WebP images. In most cases, you should not change this. However, in some cases you may need to manually set the formats. One reason for doing so is if you want to enable support for AVIF images. AVIF is a new image format that results in significantly smaller file sizes than alternative formats. It currently has [limited browser support](https://caniuse.com/avif), but this is likely to increase. It is safe to include as long as you also generate fallbacks for other browsers, which the image plugin does automatically by default.

### `transformOptions`

These values are passed in as an object to `transformOptions`, either as a prop to `StaticImage`, or to the resolver for dynamic images. They are advanced settings that most people will not need to change. Any provided object is merged with the defaults below.

| Option name | Default                   | Description                                                                                                                                                   |
| ----------- | ------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `grayscale` | `false`                   | Convert image to grayscale                                                                                                                                    |
| `duotone`   | `false`                   | Add duotone effect. Pass `false`, or options object containing `{highlight: string, shadow: string, opacity: number}`                                         |
| `rotate`    | `0`                       | Rotate the image. Value in degrees.                                                                                                                           |
| `trim`      | `false`                   | Trim "boring" pixels. See [the sharp documentation](https://sharp.pixelplumbing.com/api-resize#trim).                                                         |
| `cropFocus` | `"attention"`/`ATTENTION` | Controls crop behavior. See [the sharp documentation](https://sharp.pixelplumbing.com/api-resize#resize) for strategy, position and gravity.                  |
| `fit`       | `"cover"`/`COVER`         | Controls behavior when resizing an image and proving both width and height. See [the sharp documentation.](https://sharp.pixelplumbing.com/api-resize#resize) |

## All options

The Gatsby Image plugin uses [sharp](https://sharp.pixelplumbing.org) for image processing, and supports passing through many advanced options, such as those affecting cropping behavior or image effects including grayscale or duotone, as well as options specific to each format.

| Option                                  | Default                                                              | Description                                                                                                                                                                                                                                                                                                                                                                                                   |
| --------------------------------------- | -------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [`layout`](#layout)                     | `"constrained"` / `CONSTRAINED`                                      | Determines the size of the image and its resizing behavior.                                                                                                                                                                                                                                                                                                                                                   |
| [`width`/`height`](#widthheight)        | Source image size                                                    | Change the size of the image.                                                                                                                                                                                                                                                                                                                                                                                 |
| [`aspectRatio`](#aspectratio)           | Source image aspect ratio                                            | Force a specific ratio between the image's width and height.                                                                                                                                                                                                                                                                                                                                                  |
| [`placeholder`](#placeholder)           | `"dominantColor"`/`DOMINANT_COLOR`                                   | Choose the style of temporary image shown while the full image loads.                                                                                                                                                                                                                                                                                                                                         |
| [`formats`](#formats)                   | `["auto","webp"]`/`[AUTO,WEBP]`                                      | File formats of the images generated.                                                                                                                                                                                                                                                                                                                                                                         |
| [`transformOptions`](#transformoptions) | `{fit: "cover", cropFocus: "attention"}`                             | Options to pass to sharp to control cropping and other image manipulations.                                                                                                                                                                                                                                                                                                                                   |
| `sizes`                                 | Generated automatically                                              | [The `<img>` `sizes` attribute](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/img#attributes), passed to the img tag. This describes the display size of the image, and does not affect generated images. You are only likely to need to change this if your are using full width images that do not span the full width of the screen.                                                           |
| `quality`                               | `50`                                                                 | The default image quality generated. This is overridden by any format-specific options                                                                                                                                                                                                                                                                                                                        |
| `outputPixelDensities`                  | For fixed images: `[1, 2]`<br />For constrained: `[0.25, 0.5, 1, 2]` | A list of image pixel densities to generate. It will never generate images larger than the source, and will always include a 1x image. The value is multiplied by the image width, to give the generated sizes. For example, a `400` px wide constrained image would generate `100`, `200`, `400` and `800` px wide images by default. Ignored for full width layout images, which use `breakpoints` instead. |
| `breakpoints`                           | `[750, 1080, 1366, 1920]`                                            | Output widths to generate for full width images. Default is to generate widths for common device resolutions. It will never generate an image larger than the source image. The browser will automatically choose the most appropriate.                                                                                                                                                                       |
| `blurredOptions`                        | None                                                                 | Options for the low-resolution placeholder image. Ignored unless [`placeholder`](#placeholder) is blurred.                                                                                                                                                                                                                                                                                                    |
| `tracedSVGOptions`                      | None                                                                 | Options for traced placeholder SVGs. See [potrace options](https://www.npmjs.com/package/potrace#parameters). Ignored unless [`placeholder`](#placeholder) is traced SVG.                                                                                                                                                                                                                                     |
| `jpgOptions`                            | None                                                                 | Options to pass to sharp when generating JPG images.                                                                                                                                                                                                                                                                                                                                                          |
| `pngOptions`                            | None                                                                 | Options to pass to sharp when generating PNG images.                                                                                                                                                                                                                                                                                                                                                          |
| `webpOptions`                           | None                                                                 | Options to pass to sharp when generating WebP images.                                                                                                                                                                                                                                                                                                                                                         |
| `avifOptions`                           | None                                                                 | Options to pass to sharp when generating AVIF images.                                                                                                                                                                                                                                                                                                                                                         |