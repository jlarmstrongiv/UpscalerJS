# UpscalerJS

<a href="https://travis-ci.org/github/thekevinscott/UpscalerJS"><img alt="Travis (.org)" src="https://img.shields.io/travis/thekevinscott/upscalerjs"></a>
<a href="https://codecov.io/gh/thekevinscott/upscalerjs"><img alt="Codecov" src="https://img.shields.io/codecov/c/github/thekevinscott/upscalerjs"></a>
<a href="https://www.npmjs.com/package/upscaler"><img alt="npm" src="https://img.shields.io/npm/dw/upscalerjs"></a>
<a href="https://github.com/thekevinscott/UpscalerJS/issues"><img alt="Github issues" src="https://img.shields.io/github/issues/thekevinscott/upscalerjs"></a>
<a href="https://github.com/thekevinscott/UpscalerJS/blob/master/LICENSE"><img alt="NPM" src="https://img.shields.io/npm/l/upscaler"></a>

UpscalerJS is a tool for increasing image resolution in Javascript via a Neural Network up to 4x.

![Demo](assets/demo.gif)

[A live demo is here](https://upscaler.ai).

**Features**

* 🚀 Browser Support
* 📦 ️Simple modern ES6 interface
* 🤖 Choose from a variety of pre-trained models, or provide your own
* 📷 Scale images at 2x, 3x, and 4x resolutions.
* ⚛️ Integration with React
* 🛡️ Rigorously tested with close to 100% code coverage

## Motivation

### Why?

Increasing an image's size results in a pixelated image:

![Pixelated 2x](./assets/image-2x.png)

Most browsers by default use an algorithm called bicubic interpolation to get a more pleasing version, but this loses image quality and increases blurriness:

![Bicubic 2x](./assets/image-bicubic-2x.png)

Neural Networks [can allow us to "paint in" the expanded sections of the image](https://paperswithcode.com/task/image-super-resolution), enhancing quality.

![Upscaled 2x](./assets/image-upscaled-2x.png)

### Benefits of the Browser

Why do this in the browser?

Most cutting edge Neural Networks demand heavy computation and big GPUs, but UpscalerJS leverages Tensorflow.js to run directly in your browser. Users' data can stay on their machines, and you don't need to set up a server.

## Getting Started

### Quick Start

```javascript
import Upscaler from 'upscaler';
const upscaler = new Upscaler();
upscaler.upscale('/path/to/image').then(upscaledImage => {
  console.log(upscaledImage); // base64 representation of image src
});
```


### Install

Yarn:

```
yarn add upscaler
```

NPM:

```
npm install upscaler
```

### Examples 

You can [view runnable code examples](https://github.com/thekevinscott/UpscalerJS/tree/master/examples) on CodeSandbox.

## Usage

### Instantiation

When instantiating UpscalerJS, you must provide a model, which determines the image scaling factor.

UpscalerJS provides a number of pretrained models out of the box. UpscalerJS will automatically choose one, or you can pick a pretrained one:

```javascript
const upscaler = new Upscaler({
  model: 'div2k-2x',
});
```

Alternatively, you can provide a path to a pre-trained model of your own:

```javascript
const upscaler = new Upscaler({
  model: '/path/to/model',
  scale: 2,
});
```

When providing your own model, **you must provide an explicit scale**. Conversely, if you are requesting a pretrained model, **do not** provide an explicit scale.

[A full list of pretrained models is available in the models repo](https://github.com/thekevinscott/UpscalerJS-models).

### Upscaling

You can upscale an image with the following code:

```javascript
upscaler.upscale('/path/to/image').then(img => {
  console.log(img);
});
```

You can provide the image in any of the following formats:

* `string` - A URL to an image. Ensure the image can be loaded (for example, make sure the site's CORS policy allows for loading).
* `Image` - an HTML Image element.
* `tf.Tensor3D` - You can also pass a tensor directly.

By default, a base64-encoded `src` attribute is returned. You can change the output type like so:

```javascript
upscaler.upscale('/path/to/image', {
  output: 'tensor',
}).then(img => {
  console.log(img);
});
```

The available types for output are:

* `src` - A src URL of the upscaled image.
* `tf.Tensor3D` - The raw tensor.

#### Performance

For larger images, attempting to run inference can impact UI performance. To address this, you can provide a `patchSize` parameter to infer the image in "patches" and avoid blocking the UI. You will likely also want to provide a `padding` parameter:

```
```javascript
  patchSize: 64,
  padding: 5,
})
```

Without padding, images will usually end up with unsightly artifacting at the seams between patches. You should use as small a padding value as you can get away with (usually anything above 3 will avoid artifacting).

Smaller patch sizes will block the UI less, but also increase overall inference time for a given image.

### Pretrained Models

There are a number of pretrained models provided with the package:

| Key | Dataset | Scale | Example |
| --- | --- | --- | --- |
| `2x` | div2k | 2 | [View](https://github.com/thekevinscott/UpscalerJS-models/tree/master/examples/div2k#2x) |
| `3x` | div2k | 3 | [View](https://github.com/thekevinscott/UpscalerJS-models/tree/master/examples/div2k#3x) |
| `4x` | div2k | 4 | [View](https://github.com/thekevinscott/UpscalerJS-models/tree/master/examples/div2k#4x) |

You can read more about the pretrained models at the dedicated repo [UpscalerJS-models](https://github.com/thekevinscott/UpscalerJS-models).

## API

### `constructor`

Instantiates an instance of UpscalerJS.

#### Example

```javascript
const upscaler = new Upscaler({
  model: 'div2k-2x',
  scale: 2,
  warmupSizes: [[256, 256]]
});
```

#### Options

* `model` (`string`) - A string of a pretrained model, or a URL to load a custom pretrained model.
* `scale` (`number`) - The scale of the custom pretrained model. Required if providing a custom model. If a pretrained model is specified, providing `scale` will throw an error.
* `warmupSizes` (Optional, `Array<[number, number] | { patchSize: number, padding?: number }`>) - An array of sizes to "warm up" the model. By default, the first inference run will be slower than the rest. This passes a dummy tensor through the model to warm it up. It must match your image size exactly. Sizes are specified as `[width, height]`.

Possible model values are:

* `div2k-2x` - Scale of 2x
* `div2k-3x` - Scale of 3x
* `div2k-4x` - Scale of 4x
* `psnr` - Scale of 2x

### `upscale`

Accepts an image and returns a promise resolving to the upscaled version of the image.

#### Example

```javascript
upscaler.upscale('/path/to/image', {
  output: 'tensor',
  patchSize: 64,
  padding: 5,
  progress: (amount) => {
    console.log(`Progress: ${amount}%`);
  }
}).then(upscaledImage => {
  ...
});
```

#### Options

* `src` (`str|HTMLImage|tf.Tensor3D`) - Path to the image, or an `HTMLImage` representation of the image, or a 3-dimensional tensor representation of the image.
* `options`
  * `output` (`src|tensor`) - The desired output of the function. Defaults to a base 64 `src` representation.
  * `patchSize` (`number`) - The desired patch size to use for inference.
  * `padding` (`number`) - Extra padding to be applied to the patch size during inference.
  * `progress` (`(amount: number) => void`) - A progress callback denoting the number of patches processed.

### `warmup`

If desired, the model can be "warmed up" after instantiation by calling `warmup` directly.

#### Example

```javascript
upscaler.warmup([[256, 256]]).then(() => {
  // all done.
});
```

Or you can provide a patch size and padding:

```javascript
upscaler.warmup([{
  patchSize: 64,
  padding: 5,
}]).then(() => {
  // all done.
});
```

#### Options

* `warmupSizes` (Optional, `Array<[number, number]`>) - An array of sizes to "warm up" the model.

### `getModel`

Gets the underlying model.

#### Example

```javascript
upscaler.getModel().then(model => {
})
```

## Troubleshooting

### You must provide an explicit scale

When initializing UpscalerJS with a custom model, you need to also denote the numeric scale of the model (2x, 3x, 4x). UpscalerJS needs to know the scale in order to patch correctly.

Scale can be provided at initialization with:

```javascript
const upscaler = new Upscaler({
  model: '/your/custom/model',
  scale: 2,
})
```

### You are requesting the pretrained model but are providing an explicit scale

A pretrained model is trained to upscale to a specific scale. If you initialize UpscalerJS with a pretrained model and also provide an explicit scale, UpscalerJS will throw an error.

You can fix this by not providing a `scale` property if using a pretrained model:

```javascript
const upscaler = new Upscaler({
  model: 'div2k-2x',
})
```

### Padding is undefined

If specifying a patch size but not padding, you will likely encounter artifacting in the upscaled image.

![Image with artifacting](./assets/image-with-artifacting.png)

Most of the time, this artifacting is undesired. To resolve the artifacting, add an explicit padding:

```
upscaler.upscale('/path/to/img', {
  patchSize: 64,
  padding: 4,
})
```

![Image with artifacting](./assets/image-without-artifacting.png)

If you would like to keep artifacting but hide the warning message, pass an explicit padding value of 0:

```
upscaler.upscale('/path/to/img', {
  patchSize: 64,
  padding: 0,
})
```

## Contributions 

Contributions are welcome! Please follow the existing conventions, use the linter, add relevant tests, and add relevant documentation.

To contribute pretrained models, head over to [UpscalerJS-models](https://github.com/thekevinscott/UpscalerJS-models).

## Support

* Create a [Github issue](https://github.com/thekevinscott/UpscalerJS/issues) for bug reports, feature requests, or questions
* Follow [@thekevinscott](https://twitter.com/thekevinscott) for announcements
* Add a ⭐️ [star on GitHub](https://github.com/thekevinscott/UpscalerJS) or ❤️ [tweet](https://twitter.com/intent/tweet?url=https%3A%2F%2Fgithub.com%2Fthekevinscott%2Fupscaler&via=thekevinscott&hashtags=javascript,image-enhancement,tensorflow.js,super-resolution) to support the project!

## License

This project is licensed under the MIT License. See the [LICENSE](https://github.com/thekevinscott/UpscalerJS/blob/master/LICENSE) for details.

Copyright (c) Kevin Scott ([@thekevinscott](https://thekevinscott.com))
