## Anatomy (Shortcode)

File Location:

`layouts/shortcodes/hugo-image.html`

<div class="notes">
The file name determines the shortcode itself.
</div>

## Constants

```Go
{{/* CONSTANTS */}}
{{$PRELOADER_TYPE_BLURRED := "blurred"}}
{{$PRELOADER_TYPE_PIXELLATED := "pixellated"}}
{{$RESOURCE_TYPE_PAGE := "page"}}
{{$RESOURCE_TYPE_GLOBAL := "global"}}
```

<div class="notes">
Useful to prevent too many "magic strings"
</div>

## Props/Parameters

```Go
{{/* PROPS */}}
{{$rawImageSource := .Get "src"}}
{{$pixelHeight := .Get "pixelHeight"}}
{{$pixelWidth := .Get "pixelWidth"}}
{{$fluidMaxHeight := .Get "fluidMaxHeight"}}
{{$fluidMaxWidth := .Get "fluidMaxWidth"}}
{{$preloaderType := .Get "preloaderType"}}
{{$resourceType := .Get "resourceType"}}
```

<div class="notes">
I call them props because of React familiarity

Actually called Parameters

</div>

## Defaults

```Go
{{/* COERCING DEFAULTS */}}
{{$actualPreloaderType := $PRELOADER_TYPE_BLURRED}}
{{with $preloaderType}}
  {{$actualPreloaderType = $preloaderType}}
{{end}}

{{$actualResourceType := $RESOURCE_TYPE_PAGE}}
{{if eq $resourceType $RESOURCE_TYPE_GLOBAL}}
  {{$actualResourceType = $RESOURCE_TYPE_GLOBAL}}
{{end}}
```

<div class="notes">
One thing to note: Go is not picky about type safety here

You could default a value to a string and re-assign to an Image just fine

</div>

## Sourcing Resources

```Go
{{/* Processing Resource */}}
{{$fullSizeImagePath := $rawImageSource}}
{{if strings.HasPrefix $rawImageSource "http"}}
  {{$image  = $rawImageSource | resources.GetRemote}}
{{else}}
  {{if eq $actualResourceType $RESOURCE_TYPE_GLOBAL}}
    {{$image  = resources.Get $rawImageSource }}
  {{else}}
    {{$image  = .Page.Resources.GetMatch $rawImageSource }}
  {{end}}

  {{$fullSizeImagePath = $image.Permalink}}
{{end}}
```

<div class="notes">
This is done to decide whether the resource is:
- remote or local
- locally global or content relative

</div>

## Sourcing Resources

```Go
{{/* Processing Image */}}
{{$preloaderImage := $image.Resize "40x"}}
{{if (eq $actualPreloaderType $PRELOADER_TYPE_PIXELLATED)}}
  {{$preloaderImage = $image | images.Filter (images.Pixelate 60)}}
{{end}}
```

<div class="notes">
Defaults to processing a mini version for blur up

Additionally runs pixellated filter if param is set for it

I would like to not process the image twice. Not sure how yet

</div>

## Sourcing Resources

```Go
{{/* Processing Image */}}
{{$preloaderImage := $image.Resize "40x"}}
{{if (eq $actualPreloaderType $PRELOADER_TYPE_PIXELLATED)}}
  {{$preloaderImage = $image | images.Filter (images.Pixelate 60)}}
{{end}}
```

<div class="notes">
Defaults to processing a mini version for blur up

Additionally runs pixellated filter if param is set for it

I would like to not process the image twice. Not sure how yet

</div>

## Sizing the image

```Go
{{/* Sizing */}}
{{$width := $image.Width}}
{{with $pixelWidth}}
  {{$width = $pixelWidth}}
{{end}}

{{$height := $image.Height}}
{{with $pixelHeight}}
  {{$height = $pixelHeight}}
{{end}}
```

<div class="notes">
This is used to scale the placeholder and prevent layout shift

It defaults to using the original image size

</div>

## The HTML

```html
<div
  class="hugo-image-wrapper"
  style="width: {{$width}}px; height: {{$height}}px"
>
  <img
    class="hugo-image-placeholder {{if (eq $actualPreloaderType $PRELOADER_TYPE_BLURRED)}}hugo-image-blurred{{end}}"
    src="{{$preloaderImage.RelPermalink}}"
    height="{{$height}}"
    width="{{$width}}"
  />
  <img
    src=""
    data-full-size-src="{{$fullSizeImagePath}}"
    class="hugo-image-full-size"
    height="{{$height}}"
    width="{{$width}}"
  />
</div>
```

<div class="notes">
This is the resulting HTML template where we use the values from before

</div>

## The Styling

```css
.hugo-image-wrapper {
  position: relative;
  overflow: hidden;
}

.hugo-image-placeholder {
  position: absolute;
  top: 0;
  right: 0;
  bottom: 0;
  left: 0;
}

.hugo-image-full-size {
  position: absolute;
  top: 0;
  right: 0;
  bottom: 0;
  left: 0;
  opacity: 0;
  transition: opacity 1s;
}

.hugo-image-blurred {
  filter: blur(20px);
}

.hugo-image-loaded {
  opacity: 1;
}
```

<div class="notes">
Just some positioning magic and filtering so that the full image loads over the placeholder

</div>

## The Scripting

```javascript
document.addEventListener("DOMContentLoaded", function () {
  let observer = new IntersectionObserver(
    (entries, observer) => {
      entries.forEach((entry) => {
        if (entry.isIntersecting) {
          const fullSizeImageElement = entry.target.querySelector(
            ".hugo-image-full-size"
          );

          fullSizeImageElement.addEventListener("load", function (event) {
            event.target.classList.add("hugo-image-loaded");
          });

          fullSizeImageElement.addEventListener("error", function (event) {
            // console.log("error", { event });
          });
          fullSizeImageElement.src =
            fullSizeImageElement.getAttribute("data-full-size-src");
        }
      });
    },
    {
      root: document.querySelector("window"),
      rootMargin: "0px",
      threshold: 1.0,
    }
  );
  document.querySelectorAll(".hugo-image-wrapper").forEach((imageWrapper) => {
    observer.observe(imageWrapper);
  });
});
```

<div class="notes">
Intersection observer magic

preloading the full image behind the scenes and fading it in when loaded

</div>

---
