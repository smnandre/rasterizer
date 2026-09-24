---
description: Rasterize a document to PNG, choose an adapter, set scale and background, and write the bitmap to a file or a string.
---

# Usage

## Rasterize a document

```php
use Atelier\Rasterizer\Rasterizer;

$rasterizer = Rasterizer::create();
$bitmap = $rasterizer->rasterize($svg);

$bitmap->save('icon.png');
```

`Rasterizer::create()` selects the first available adapter in package order:
`resvg`, then `rsvg-convert`.

## Input forms

`rasterize()` accepts a markup string, any `\Stringable`, or an `SvgInput`. A
bare string or `\Stringable` is always treated as markup, so an
`Atelier\Svg\Svg` document (which is `\Stringable`) passes directly:

```php
$rasterizer->rasterize('<svg xmlns="http://www.w3.org/2000/svg" ...>');
$rasterizer->rasterize($svgDocument); // any \Stringable
```

For a file or stream, use the named constructors on `SvgInput`. This keeps a
plain string unambiguous (it is markup, never a path):

```php
use Atelier\Rasterizer\Svg\SvgInput;

$rasterizer->rasterize(SvgInput::fromFile('logo.svg'));
$rasterizer->rasterize(SvgInput::fromFile(new SplFileInfo('logo.svg')));
$rasterizer->rasterize(SvgInput::fromStream($handle));
```

Input is validated. Empty input, markup without an `<svg>` element, or an
unreadable file or stream throws `InvalidSvgInputException`.

## Set size and background

Pass a `BitmapOptions` instance. See [Options](options.md) for every field.

```php
use Atelier\Rasterizer\Bitmap\BitmapOptions;

$bitmap = $rasterizer->rasterize($svg, new BitmapOptions(
    width: 1200,
    height: 630,
    keepAspectRatio: true,
    background: '#ffffff',
));
```

With both `width` and `height` set, `keepAspectRatio: true` requests a fit
inside the requested box without distortion. Use `keepAspectRatio: false` for
the selected adapter's exact-size behavior. `rsvg-convert` supports both modes;
`resvg` preserves aspect ratio and does not expose a separate stretch flag.

## Use the result

`rasterize()` returns a `BitmapResult` holding the encoded bytes in memory:

```php
$bitmap->contents;   // raw PNG bytes
$bitmap->format;     // BitmapFormat::Png
$bitmap->mimeType;   // 'image/png'
$bitmap->width;      // produced width, or null
$bitmap->height;     // produced height, or null

$bitmap->save('out/card.png');
echo 'data:'.$bitmap->mimeType.';base64,'.base64_encode($bitmap->contents);
```

`save()` throws `RasterizationFailedException` when the target directory is
missing or not writable.

## Choose where temporary files are written

Each rasterizer writes the SVG to a temporary file before invoking its binary.
By default it uses the system temp directory. Override it per instance:

```php
use Atelier\Rasterizer\Adapter\ResvgRasterizer;

new ResvgRasterizer(temporaryDirectory: '/var/run/atelier');
```

## Log process execution

Pass any PSR-3 logger to the facade, factory, or adapter. Logging records process
execution events; it does not replace `ProcessRunnerInterface`, which remains
the injectable process boundary.

```php
use Atelier\Rasterizer\Rasterizer;

$rasterizer = Rasterizer::create(logger: $logger);
```

Use a custom runner only when you need to decorate or replace process execution:

```php
use Atelier\Rasterizer\Adapter\ResvgRasterizer;

$rasterizer = new ResvgRasterizer(processRunner: $runner);
```

## Select an adapter explicitly

Use the facade convenience methods when you want a fixed adapter with default
configuration:

```php
use Atelier\Rasterizer\Rasterizer;

$rasterizer = Rasterizer::resvg();
$rasterizer = Rasterizer::rsvgConvert();
```

Use adapter classes directly when you need constructor options such as a custom
binary path or temporary directory.

For advanced selection rules, pass adapter factories to `RasterizerFactory`:

```php
use Atelier\Rasterizer\Adapter\ResvgRasterizerFactory;
use Atelier\Rasterizer\Adapter\RsvgConvertRasterizerFactory;
use Atelier\Rasterizer\RasterizerFactory;

$rasterizer = (new RasterizerFactory([
    new RsvgConvertRasterizerFactory(),
    new ResvgRasterizerFactory(),
]))->create();
```

## Errors

All failures extend `Atelier\Rasterizer\Exception\ExceptionInterface`:

| Exception                      | Cause                                             |
| ------------------------------ | ------------------------------------------------- |
| `BinaryNotFoundException`      | The rasterizer binary was not found.              |
| `InvalidArgumentException`     | A public API received an invalid argument.        |
| `InvalidSvgInputException`     | Input is empty or is not SVG.                     |
| `NoRasterizerAvailableException` | No configured adapter factory is supported.     |
| `UnsupportedFormatException`   | The adapter cannot produce the requested format.  |
| `RasterizationFailedException` | The process failed, timed out, or produced no output, or the file could not be written. |
