## My Shortcode

Hugo Image (Inspired by Gatsby Image)

Creates Placeholder images using the blur-up technique or a "pixellated" technique

[https://github.com/cmgriffing/hugo-image](https://github.com/cmgriffing/hugo-image)

<div class="notes">
I need to figure out a better way to distribute it instead of just copying into place.

It is meant to create placeholders

</div>

## Why

- Faster page loads
- Less initial data over the wire
- Uses Intersection Observer to lazy load images

<div class="notes">
loading="lazy" does not do placeholders

loading="lazy" is also not enabled by default in Safari (mobile and desktop)

</div>

## Usage

An example of a remote image path:

```Go
// An example of a remote image
{{<hugo-image src="https://www.fillmurray.com/460/301">}}

// An example of a Global Resource in the `assets` folder:
{{<hugo-image src="images/460x300-global.jpeg" resourceType="global">}}

// An example of a Page resource in the content's folder:
{{<hugo-image src="460x300.jpg">}}
```

<div class="notes">
Content images live in the same folder as a content file
</div>

## Configuration

- src - required - This is the path to the image to process. It can be remote, global, or a page resource.

- pixelHeight - optional - This is the height in pixels to make the img element. Helps fight page shifting. Uses actual image height by default.

- pixelWidth - optional - This is the width in pixels to make the img element. Helps fight page shifting. Uses actual image width by default.

## Configuration (contd)

- resourceType - optional - "page" | "global" - This determines where the image asset is loaded from. Default will look in Page folder. Defaults to "page".

- preloaderType - optional - "blurred" | "pixellated" - This determines which preloader type is used. Defaults to "blurred".

## More info

[https://github.com/cmgriffing/hugo-image](https://github.com/cmgriffing/hugo-image)

Roadmap:

- figure out how to include script and style snippets at a global level instead of shortcodes
- add file for render hook to process markdown images

## What do you call an electric car that isn't moving?

## Static

![](assets/agent-smith.gif)

---
