# PTT - Page Transform Tree

This documents is a full specification of the PTT document format. PTT is a simple JSON describing the component tree used to render document pages in various formats (HTML, PDF,...), sizes (A4,A3, Letter,...) and even various visual media (screen, papers).

## <a name="PTT">Page Transform Tree</a>

The PTT is essentially a set of semantic assumptions laid on top of the JSON syntax. The PPT document is a plain JSON describing either logical nor visual content on the document.
The typical description of the content consists of two parts:

+   component tree - it describes the logical and/or visual composition of components
+   visual appearence of component - it describes the visual properties of components

The typical applying of PTT definition (content rendering) consists of two steps

+ logical component tree - it compose tree of elements - it forms the logical hierarchical layout of components
+ visual component tree - it renders components according the logical component tree to the visual component tree - it forms the visual hierarchical layout of components

The PPT format is __framework agnostic__ and the is why the PTT document prescribes __only the content description__. The __content applying__ is not the part of the PPT format specification.

### The structure of PTT document

It is a simple component tree that consists of these two nodes

+   **containers** - nodes that are containers for other components - visual and logical grouping of parts of document (sections, containers,grids, rows, cells, panels, etc. )
+   **boxes** - terminal nodes (leaf) that are visible components - (components, boxes, widgets) - renders to document (typically by simple mapings to props of component)

There is an minimal 'Hello world' example. The PTT consists of one container and one box with TextBox element.

```json
{
 "name": "Hello World Example",
 "elementName": "PTTv1",
 "containers": [
    {
     "name": "container",
     "elementName": "Container",
     "style": { "top": 0, "left": 0, "height": 200, "width": 740, "position": "relative" }
     "boxes": [{
        "name": "My first text",
        "elementName": "TextContent",
        "style": { "top": 0, "left": 0 },
        "props":{
             "content": "Hello world"
            }
        }],
    }]
}
```
### PPT Node

+   **containers** node - collection of other containers (children)
+   **boxes** node - collection of widgets

The component schema tree is composed using __containers__ property as collection of children.
The boxes on the other hand is a leaf collection that can not have other children.

### PTT Node Properties

Each node can have these object properties

+   **name** - optional element identifier (has no impact on document rendering)
+   **elementName** - required component name - type of element to render 
+   **style** - element positions and dimensions, stacking context and transformation properties
    +   top, left - element position - if not specified the default value is 0
    +   width, height - element dimensions - if not specified the default value is the same as its parent
    +   position - support for various position schemas -> specify the layouting of elements (normal flow, absolute, relative positioning)
    +   zIndex - optional - it defines stacking context, if not defined - stacking context is based on the order in document
    +   transform - optional - it enables to translate, rotate, scale, move transform origin
+   **props** - component props as component's options - specific properties of element to render

### PTT Document Node Properties

This is the root node of the PTT document.

+   **name** - required document name (has no impact on document rendering)
+   **elementName** - required ptt document schema version 


### PTT extensibility

PTT is simple JSON. You can extend PTT document with any properties other than specified above.

## PTT reference usage

The __content applying__ is not the part of the PPT format specification. This section is intended for PTT rendering implementers to guide them when solving layouting and positioning of components.

PTT document layout is mainly specified by the type of containers used in the definition (Container,BackgroundContainer, Page,Grid,Row,Cell, etc.). 

We can distinguish containers based on reponsivness (adjusting components dimensions and positions)

+  responsive (Grid,Row,Cell)
+  static - (Container, Page, BackgroundContainer) - they retain its dimensions and positions

We can distinguish containers based on whetever providing visual effects

+ logical (Container, Page, Row) - has no visual effect (only simple layouting) - only to grouping logically related components
+ visual - has visual effect other than simple layouting as background, border, gutters, etc. (BackgroundContainer, Cell, Grid)


The default container element is __Container__ and belongs to  __static__ and __logical__ containers.

It supports this positioning schemas (the meaning is the same as CSS positions schemas)  

+   __absolute positioning__ - position is assigned with respect to its parent (container)
+   __relative positioning__ - normal flow with support for offset relative to this position - for containers (sections) 

Content applying offers two rendering modes

+   __mixed positioning__ - absolute positioning for boxes (widgets) and relative for containers (sections)
+   __normal flow__ - relative positioning for both - boxes and containers 

The units are pixes and be careful is DPI dependend.
 
The default responsive element is __Grid__ and belongs to __responsive__ and __visual__ containers.

It is based on css flexbox - you can add child containers (Cell) as flex box items. 

### PTT pages rendering

This section specifies how the page rendering can occur.

Seperate based on different visual media (screen and paper).

The both share container data binding and widgets data binding.

Container data binding steps
+  apply data binding for containers (sections) (visibility, ranges for repeatable containers)
+  remove invisible containers
+  expand repeatable containers - cloning row templates
  

### Screen rendering

Screen rendering prefers responsivnes.

### PTT Screen algorithm

+  apply container data binding
+  traverse PTT and apply container and boxes rendering with applying widgets data binding 

### Paper rendering

Paper rendering requires static elements (absolutely positioning).

#### PTT Paper options

+   page height and width
+   page orientation ('portrait', 'landscape') is optional and defaults to 'portrait'.
+   margin can provided as an object containing one or more of the following properties: 'top', 'left', 'bottom', 'right'

#### PTT Paper algorithm

Example of algorithm how to render pages from PTT document definition. 

+   apply container data binding
+   transform relative positions to absolute positions
+   reduce to boxes (terminal nodes) - using containers absolute positions (top,height) and its dimensions (with, height)
+   group to pages - create pages and add boxes to them
+   for all pages - create page, apply boxes rendering and apply widgets data binding


The proprietary implementation of the algorithm can be found [here](https://github.com/rsamec/react-page-renderer/blob/master/src/utilities/transformToPages.js).
