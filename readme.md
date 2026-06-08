# SVG Controls

A lightweight, vanilla JavaScript library that adds **zoom**, **pan**, **element selection**, and **reset** capabilities to any SVG embedded via `<object>` tag. Perfect for diagrams, maps, floor plans, or any interactive SVG.

**Repository**: [https://github.com/arce/svg-controls](https://github.com/arce/svg-controls)

## Features

- **Zoom** – scroll wheel zooms in/out centered on the mouse cursor.
- **Pan** – click and drag to move the viewport.
- **Element selection** – click on any SVG element to highlight it with a customizable stroke color.
- **Double‑click reset** – restores the original viewBox.
- **Full API** – enable/disable features, trigger zoom/pan programmatically, get selection callbacks.
- **Zero dependencies** – pure JavaScript.
- **CORS aware** – handles cross‑origin restrictions gracefully (logs errors, fails silently).

## How it works

The script automatically detects an `<object>` element with MIME type `image/svg+xml` (or `image/xml+svg`), waits for the SVG to load, and attaches event handlers directly to the embedded SVG document. It manipulates the `viewBox` attribute to implement zoom and pan, and highlights selected elements by temporarily changing their `stroke` and `stroke-width`.

## Installation

Include the script in your HTML **after** the `<object>` element (or anywhere, as it auto‑initializes on `load`):

```html
<script src="svg-controls.js"></script>
```

### Basic HTML structure

```html
<object id="mySvg" data="diagram.svg" type="image/svg+xml"></object>
```

No additional configuration is required – the library will attach itself to the first `<object>` of the correct type on the page.

## Usage

Once the library is loaded, you can access the control API through the `<object>` element’s `__SVGControl` property:

```html
<object id="mySvg" type="image/svg+xml" data="map.svg"></object>
<script>
  const obj = document.getElementById('mySvg');
  obj.addEventListener('load', () => {
    const api = obj.__SVGControl;
    if (api) {
      api.toggleFeature('zoom', false);      // disable zoom
      api.setSelectionColor('blue');         // change highlight color
      api.zoom(1.2);                         // zoom in 20% at center
      api.pan(50, 20);                       // pan by 50px horizontally, 20px vertically
      api.select('someElementId');            // select an element by its ID
      api.reset();                            // restore original viewBox
    }
  });
</script>
```

### Waiting for the SVG to be ready

The `__SVGControl` object becomes available as soon as the script finishes attaching to the SVG. Use the `load` event on the `<object>` element to be safe.

## API Reference

### `obj.__SVGControl`

The controller object attached to the `<object>` element. All methods are optional – call them only when needed.

#### Methods

| Method | Description |
|--------|-------------|
| `setCallback(type, fn)` | Registers a callback function. `type` can be `'zoom'`, `'pan'`, or `'select'`. The callback receives a single argument: for `zoom` the zoom factor, for `pan` an object with `dx` and `dy`, for `select` the selected SVG element (or `null` if deselected). |
| `toggleFeature(feature, enabled)` | Enable/disable a feature. `feature` can be `'zoom'`, `'pan'`, or `'select'`. `enabled` is a boolean. |
| `setSelectionColor(color)` | Changes the stroke color used for highlighting selected elements. Default is `'red'`. Accepts any valid CSS color string. |
| `zoom(factor, clientX, clientY)` | Zooms by `factor` (e.g., `1.2` to zoom in, `0.8` to zoom out) relative to the point `(clientX, clientY)` in viewport coordinates. If coordinates are omitted, zoom is centered on the `<object>` element. |
| `pan(dx, dy)` | Pans the view by `dx` and `dy` pixels (in screen space, automatically converted to viewBox units). |
| `select(id)` | Highlights the SVG element with the given `id`. Pass `null` or an empty string to clear selection. |
| `reset()` | Restores the original `viewBox` (the state at the moment the script attached). |
| `destroy()` | Removes all event listeners and cleans up internal references. The SVG returns to its original state (except any `viewBox` modifications persist). |

### Global state (read‑only)

The script maintains internal state that can be inspected for debugging, but you should **not** modify it directly. Use the API methods instead.

## Events & Callbacks

The library fires callbacks when user interactions occur (or when you call API methods). Register them with `setCallback`:

```javascript
api.setCallback('zoom', (factor) => console.log(`Zoom factor: ${factor}`));
api.setCallback('pan', (dx, dy) => console.log(`Panned by ${dx}, ${dy}`));
api.setCallback('select', (element) => {
  if (element) console.log(`Selected element with id: ${element.id}`);
  else console.log('Selection cleared');
});
```

## Important Notes

- **CORS restrictions** – If the SVG is served from a different origin, the script may not be able to access `contentDocument`. Ensure the SVG is served with proper CORS headers or host it on the same domain.
- **Only one SVG** – The library attaches to the **first** `<object>` element of type `image/svg+xml` found in the DOM. For multiple SVGs, you need to modify the script or instantiate it manually (see advanced section below).
- **SVG must have IDs** – For element selection to work, the SVG elements you want to select must have an `id` attribute.
- **Initial viewBox** – If the SVG does not have a `viewBox`, the script will create one based on its bounding box or client dimensions.

## Advanced: manual instantiation

If you need to control a specific SVG or want to avoid auto‑initialization, you can extract the core logic and call it manually. The auto‑initialization only runs once on `window.load`. For more control, consider integrating the internal class into your own module (not provided by default).

## Browser Support

Works in all modern browsers (Chrome, Firefox, Edge, Safari) **with CORS‑compliant SVGs**. Partial support in older browsers that lack `PointerEvent` (fallback behavior may be limited).

## License

MIT