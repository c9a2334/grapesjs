---
title: Getting Started
meta:
  - name: keywords
    content: grapesjs getting started
---

# Getting Started

In this guide we'll see how to create a completely customized page builder from scratch.
Here you can find the final result: [demo](##).

At first, let's import the latest version of the library

```html
<link rel="stylesheet" href="//unpkg.com/grapesjs/dist/css/grapes.min.css">
<script src="//unpkg.com/grapesjs"></script>
```

## Start from the canvas

The first step is to define the interface of our editor and for this purpose we gonna start from basic HTML layouts.
Finding a common structure for the UI of any project is not an easy task that's why GrapesJS prefers to keep this process as simple as possible, by providing just few helpers but letting the user define the whole interface, this guarantees maximum flexibility.
The main part of the GrapesJS editor is the canvas, this is where you gonna create the whole structure of your templates and you definitely can't miss it. Let's try to initiate the editor with just the canvas and no panels.

<<< @/docs/.vuepress/components/demos/DemoCanvasOnly.html
<<< @/docs/.vuepress/components/demos/DemoCanvasOnly.js
<<< @/docs/.vuepress/components/demos/DemoCanvasOnly.css
<Demo>
 <DemoCanvasOnly/>
</Demo>

With just the canvas you're already able to move, copy and delete components from the structure (when you select components in the canvas, the toolbar is shown). For now we just see the example template taken from the container. Let's see now how can we create and drag custom blocks into our canvas.

## Add Blocks
The block in GrapesJS is just a reusable piece of HTML that you can drop in the canvas. A block can be an image, a button, or an entire section with videos, forms and iframes. Let's start from creating another container and append inside it few basic blocks which we can later use to build more complex structures.

```html{4}
<div id="gjs">
  ...
</div>
<div id="blocks"></div>
```
```js
const editor = grapesjs.init({
  // ...
  blockManager: {
    appendTo: '#blocks',
    blocks: [
      {
        id: 'section', // id is mandatory
        label: '<b>Section</b>', // You can use HTML/SVG inside labels
        attributes: { class:'gjs-block-section' },
        content: `<section>
          <h1>This is a simple title</h1>
          <div>This is just a Lorem text: Lorem ipsum dolor sit amet</div>
        </section>`,
      }, {
        id: 'text',
        label: 'Text',
        content: '<div data-gjs-type="text">Insert your text here</div>',
      }, {
        id: 'image',
        label: 'Image',
        // Select the component once it's dropped
        select: true,
        // You can pass components as a JSON instead of a simple HTML string,
        // in this case we also use a defined component type `image`
        content: { type: 'image' },
        // This triggers `active` event on dropped components and the `image`
        // reacts by opening the AssetManager
        activate: true,
      }
    ]
  },
});
```
```css
.gjs-block {
  width: auto;
  height: auto;
  min-height: auto;
}
```
<Demo>
 <DemoBasicBlocks/>
</Demo>

As you see we add our blocks via the initial configuration, which is ok, but obviously there might be the case you would like to add them dynamically, in this case you have to use the [Block Manager API](api/block_manager.html)

```js
editor.BlockManager.add('my-block-id', {
  label: '...',
  category: '...',
  // ...
})
```
::: tip
If you want to get more about blocks we suggest to read its dedicated page: [Block Manager Module](modules/Blocks.html)
:::

## Define Components
Technically, once you drop your HTML block inside the canvas each element of the content is transformed in GrapesJS Component, which is an object containing informations about how the element is rendered in the canvas (managed in the View) and how it might look its final code (created by the properties in the Model). Generally, all Model properties are reflected in the View, so, for example, if you add a new attribute to the model, not only it will be available in the export code (will see later how to get it) but also the element you see in the canvas is updated with new attributes.
While this is a common behavior what it's cool about Components that you can create a totally decoupled view and show to the user whatever you desire (so not necessary reflecting the model). For example, by dragging a placeholder text you can fetch and show instead a dynamic content. If want to get more about Custom Components and how to create and extend them, we recommend to check out [Component Manager Module](modules/Components.html).

GrapesJS comes along with few [built-in Components](modules/Components.html#built-in-components) which enable different core features once rendered in canvas. Just to mention few of them, by double clicking on the image component you will see show up the default [Asset Manager](modules/Assets.html), which you can customize or integrate you own, by double clicking on the text component you're able to edit it via the built-in Rich Text Editor, which is also customizable and [replaceable](guides/Replace-Rich-Text-Editor.html).

As we have seen before you can create Blocks directly as Components
```js
editor.BlockManager.add('my-block-id', {
  // ...
  content: {
    tagName: 'div',
    draggable: false,
    attributes: { 'some-attribute': 'some-value' },
    components: [
      {
        tagName: 'span',
        content: '<b>Some static content</b>',
      }, {
        tagName: 'div',
        // use `content` for static strings, `components` string will be parsed
        // and transformed in Components
        components: '<span>HTML at some point</span>',
      }      
    ]
  }
})
```
::: tip
Check the [Components API](api/components.html) and see how to interact with components dynamically
:::

An example on how to select some inner component and replace its children with new contents

```js
// The wrapper is the root Component
const wrapper = editor.DomComponents.getWrapper();
const myComponent = wrapper.find('div.my-component')[0];
myComponent.components().forEach(component => /* ... do something ... */);
myComponent.components('<div>New content</div>');
```

## Panels & Buttons
Now that we have a canvas and custom blocks let's see how to create a new custom panel with some buttons inside (using [Panels API](api/panels.html)) which trigger commands (the core one or custom).

```html{1,2,3}
<div class="panel__top">
    <div class="panel__basic-actions"></div>
</div>
<div id="gjs">
  ...
</div>
<div id="blocks"></div>
```

```css
.panel__top {
  padding: 0;
  width: 100%;
  display: flex;
  position: initial;
  justify-content: center;
  justify-content: space-between;
}
.panel__basic-actions {
  position: initial;
}
```

```js
editor.Panels.addPanel({
  id: 'panel-top',
  el: '.panel__top',
});
editor.Panels.addPanel({
  id: 'basic-actions',
  el: '.panel__basic-actions',
  buttons: [
    {
      id: 'visibility',
      active: true, // active by default
      className: 'btn-toggle-borders',
      label: '<u>B</u>',
      command: 'sw-visibility', // Built-in command
    }, {
      id: 'export',
      className: 'btn-open-export',
      label: 'Exp',
      command: 'export-template',
      context: 'export-template', // For grouping context of buttons from the same panel
    }, {
      id: 'show-json',
      className: 'btn-show-json',
      label: 'JSON',
      command(editor) {
        editor.Modal.setTitle('Components JSON')
          .setContent(`<textarea style="width:100%; height: 250px;">
            ${JSON.stringify(editor.getComponents())}
          </textarea>`)
          .open();
      },
    }
  ],
});
```

<Demo>
 <DemoCustomPanels/>
</Demo>

So, we have defined where to render the panel with `el: '#basic-panel'` and then for each button we added a `command` property. The command could be the id, an object with `run` and `stop` functions or simply a single function.
Try to use [Commands](api/commands.html) when possible, they allow you to track actions globally and execute also callbacks before and after their execution (you can even interrupt them).

```js
editor.on('run:export-template:before', opts => {
  console.log('Before the command run');
  if (0 /* some condition */) {
    opts.abort = 1;
  }
});
editor.on('run:export-template', () => console.log('After the command run'));
editor.on('abort:export-template', () => console.log('Command aborted'));
```

## Layers
Another utility tool you might find useful when working with web elements is a layer manger. It's just a tree overview of the structure nodes and enables you to manage it easier. To enable it you just have to specify where you want to render it

```html{4,5,6,7,8,9,10,11}
<div class="panel__top">
    <div class="panel__basic-actions"></div>
</div>
<div class="editor-row">
  <div class="editor-canvas">
    <div id="gjs">...</div>
  </div>
  <div class="panel__right">
    <div class="layers-container"></div>
  </div>
</div>
<div id="blocks"></div>
```
<<< @/docs/.vuepress/components/demos/DemoLayers.css

```js
const editor = grapesjs.init({
  // ...
  layerManager: {
    appendTo: '.layers-container'
  },
  // We define a default panel as a sidebar to contain layers
  panels: {
    defaults: [{
      id: 'layers',
      el: '.panel__right',
      // Make the panel resizable
      resizable: {
        maxDim: 350,
        minDim: 200,
        tc: 0, // Top handler
        cl: 1, // Left handler
        cr: 0, // Right handler
        bc: 0, // Bottom handler
        // Being a flex child we need to change `flex-basis` property
        // instead of the `width` (default)
        keyWidth: 'flex-basis',
      },
    }]
  }
});
```
<Demo>
 <DemoLayers/>
</Demo>

## Style Manager
Once you have defined the structure of the template probably the next step is the ability to style it. To meet this need GrapesJS includes the Style Manager module which is composed by CSS style properties and sectors. To make it more clear, let's see how to define a basic set.

Let's start from adding one more panel inside the `panel__right` and another one in `panel__top` which will contain a Layer/Style Manager switcher

```html{3,11}
<div class="panel__top">
    <div class="panel__basic-actions"></div>
    <div class="panel__switcher"></div>
</div>
<div class="editor-row">
  <div class="editor-canvas">
    <div id="gjs">...</div>
  </div>
  <div class="panel__right">
    <div class="layers-container"></div>
    <div class="styles-container"></div>
  </div>
</div>
<div id="blocks"></div>
```
```css
.panel__switcher {
  position: initial;
}
```
```js
const editor = grapesjs.init({
  // ...
  panels: {
    defaults: [
      // ...
      {
        id: 'panel-switcher',
        el: '.panel__switcher',
        buttons: [{
            id: 'show-layers',
            active: true,
            label: 'Layers',
            command: 'commandShowLayers',
          }, {
            id: 'show-style',
            active: true,
            label: 'Styles',
            command: 'commandShowStyle',
        }],
      }
    ]
  },
  styleManager: {
    appendTo: '.styles-container',
    sectors: [{
        name: 'Dimension',
        open: false,
        // Use built-in properties
        buildProps: ['width', 'min-height', 'padding'],
        // Use `properties` to define/override single property
        properties: [
          {
            // Type of the input,
            // options: integer | radio | select | color | slider | file | composite | stack
            type: 'integer',
            name: 'The width', // Label for the property
            property: 'width', // CSS property (if buildProps contains it will be extended)
            units: ['px', '%'], // Units, available only for 'integer' types
            defaults: 'auto', // Default value
            min: 0, // Min value, available only for 'integer' types
          }
        ]
      },{
        name: 'Extra',
        open: false,
        buildProps: ['background-color', 'box-shadow', 'custom-prop'],
        properties: [
          {
            id: 'custom-prop',
            name: 'Custom Label',
            property: 'font-size',
            type: 'select',
            defaults: '32px',
            // List of options, available only for 'select' and 'radio'  types
            options: [
              { value: '12px', name: 'Tiny' },
              { value: '18px', name: 'Medium' },
              { value: '32px', name: 'Big' },
            ],
         }
        ]
      }]
  },
});

// Define commands
editor.Commands.add('commandShowLayers', {
  getRowEl(editor) { return editor.getContainer().closest('.editor-row'); },
  getLayersEl(row) { return row.querySelector('.layers-container') },

  run(editor, sender) {
    const lmEl = this.getLayersEl(this.getRowEl(editor));
    lmEl.style.display = '';
  },
  stop(editor, sender) {
    const lmEl = this.getLayersEl(this.getRowEl(editor));
    lmEl.style.display = 'none';
    sender && sender.set('active', false);
  },
});
editor.Commands.add('commandShowStyle', {
  getRowEl(editor) { return editor.getContainer().closest('.editor-row'); },
  getStyleEl(row) { return row.querySelector('.styles-container') },

  run(editor, sender) {
    const smEl = this.getStyleEl(this.getRowEl(editor));
    smEl.style.display = '';
  },
  stop(editor, sender) {
    const smEl = this.getStyleEl(this.getRowEl(editor));
    smEl.style.display = 'none';
    sender && sender.set('active', false);
  },
});
```

<Demo>
  <DemoStyle/>
</Demo>

Now any component could be defined with its own style, you can add any other CSS property to your sectors and configure it by your needs.

<!-- To get more about style manager extension check out this guide.
Each component can also indicate what to style and what not.

-- Example component with limit styles
-->

## Traits
Most of them time you would style your components and you would place them somewhere in the structure, but sometimes your components might need custom attributes or even behaviours

## Devices
Grapesjs implements also a built-in module witch allows you to work with responsive templates easily. Let's see how to define different devices

-- config devices, desktop, tablet and mobile

On the UI side you will not see differences, but you can already use Devices API to toggle them.

--  show devices api and events

To help the user resize the canvas easily let's add a set of buttons

-- add responsive buttons

If you want to enable a mobile-first approch just change your configurations in this way

-- show mobile first config

## Storage
Once you get all the tools you need for styling and managing your components the last part would to setup the storing and loading process.
Grapesjs implements 2 simple type of storages, the local (by using localStorage, active by default) and the remote one.
Those are enough to cover most of the cases, but it's also possible to add new storage implementations (eg. think about IndexedDB), if you want you can read more about [Storages here].
Let's see how can we setup a simple remote storage.

-- Add remote storage, see old getting started


## Theming
One last step that might actually improve a lot the personality of you editor is how it's look visually. To achive an easy to use theming we have adapted an atomic design for this purpose. To customize the main palette of colors all you have to do is to change some CSS, or variables if you work in SCSS

-- show import in SCSS and CSS