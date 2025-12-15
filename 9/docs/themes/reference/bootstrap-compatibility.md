---
title: Bootstrap compatibility
---

# Bootstrap compatibility

Starting from PrestaShop 9.0 you can choose between two themes `classic` or `hummingbird`, the Hummingbird theme is based on more modern design and dependencies and gives better performance and SEO.

However, Hummingbird uses [Bootstrap 5.2](https://getbootstrap.com/docs/5.2/getting-started/introduction/) as a base when Classic was based on [Bootstrap 4](https://getbootstrap.com/docs/4.0/getting-started/introduction/). Since many modules developed these years are based on Bootstrap 4, switching to Hummingbird may cause some unexpected side effects.

{{% notice note %}}
If you want more details about the changes in Bootstrap 5 we invite you to read their [migration guide](https://getbootstrap.com/docs/5.2/migration/).
{{% /notice %}}

The ideal thing to do is for module developers to adapt their module to be compatible with Bootstrap 5, as ultimately the Classic theme will be deprecated and won't evolve anymore, the only remaining theme provided by PrestaShop will be based on Bootstrap 5.

In the meantime, if you need a quick solution we provide a [compatibility layer library](https://github.com/PrestaShop/bootstrap-compatibility-layer) that you can include in your module or theme so that your code remains compatible even with Bootstrap 5 as a base.

## Installation

You can install the Bootstrap compatibility layer from your preferred package.

### Package

```yml
npm install bootstrap-compatibility-layer
# OR
yarn add bootstrap-compatibility-layer
# OR
pnpm add bootstrap-compatibility-layer
```

### CDN

Or use it directly through our CDN:

```html
[...]
<link href="https://unpkg.com/bootstrap-compatibility-layer@1/dist/bootstrap-compatibility-layer.min.css" rel="stylesheet">
[...]
<script src="https://unpkg.com/bootstrap-compatibility-layer@1"></script>
[...]
```

### Use a hook

To integrate the compatibility layer from your module you can use the `actionFrontControllerSetMedia` hook for example, it is **very important** that you include the assets relying on the `addCss` and `addJs` methods, this way if other modules also include the same asset it prevents it from being loaded multiple times.

```php
class my_module extends Module {
    ...

    public function hookActionFrontControllerSetMedia()
    {
        $this->context->controller->addCss('https://unpkg.com/bootstrap-compatibility-layer@1/dist/bootstrap-compatibility-layer.min.css');
        $this->context->controller->addJs('https://unpkg.com/bootstrap-compatibility-layer@1');
    }
}
```

## Usage

### Package

To use the library as a package you can load it in your script:

```ts
import BSCompatibilityLayer from 'bootstrap-compatibility-layer';
```

The compatibility layer initializes automatically; however, you can call the methods independently if you need to.

```ts
BSCompatibilityLayer.updateAllDataAttributes();
```

### CDN

If you only want to use attribute modifiers, you can place the tag wherever you want in your code. However, if you want to use jQuery commands relating to Bootstrap, it is advisable to put it first in the list of your scripts and to wait for the DOM to be loaded before using them like the following code:

```html
[...]
<link href="https://unpkg.com/bootstrap-compatibility-layer@1/dist/bootstrap-compatibility-layer.min.css" rel="stylesheet">
[...]
<script src="https://unpkg.com/bootstrap-compatibility-layer@1"></script>
<script>
  document.addEventListener('DOMContentLoaded', function() {
    $('[data-toggle="popover"]').popover()

    $('[data-toggle="tooltip"]').tooltip()
  });
</script>
```
