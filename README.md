# PTT - Page Transform Tree

This documents is a full specification of the PTT document format. PTT is a simple JSON describing the logical component tree used to render document pages in various formats (HTML, PDF,...) and sizes (A4,A3, Letter,...).

## <a name="PTT">Page Transform Tree</a>

The PTT is essentially a set of semantic assumptions laid on top of the JSON syntax. The PPT document is a plain text JSON describing visual content on the page.
The typical description of the visual content consists of two parts:

+   logical component tree - it enables composition of elements - it describes the logical hierarchical layout of components
+   visual component tree - it describes the visual appearance - it describes the visual hierarchical layout of components

The PPT format is __framework agnostic__ and the is why the PTT document prescribes __only the logical component__ tree. The visual component description is not the part of the PPT format

### The structure of PTT document

It is a simple component tree that consists of these two nodes

+   **containers** - nodes that are invisible components - usable for logical grouping of reactive parts of document (sections)
+   **boxes** - terminal nodes (leaf) that are visible components - (components, boxes, widgets) - it maps to props of component

There is an minimal 'Hello world' example. The logical tree consists of one container and one box with TextBox element.

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
        "name": "TextBox",
        "elementName": "TextBox",
        "style": { "top": 0, "left": 0 },
        "props":{
             "content": "Hello world"
            }
        }],
    }]
}
```

### PPT Node

+   **containers** node - collection of children
+   **boxes** node - collection of widgets

The component schema tree is composed using __containers__ property as collection of children.
The boxes on the other hand is a leaf collection that can not have other children.

### PTT Node Properties

Each node can have these object properties

+   **name** - optional element identifier (has no impact on page tree rendering)
+   **elementName** - required component name - type of element to render 
+   **style** - required element positions and dimensions
    +   top, left - element position - if not specified the default value is 0
    +   width, height - element dimensions - if not specified the default value is the same as its parent
    +   position - support for various position schemas -> absolute or relative position of elements (normal flow, flex or grid position schemas is not yet implemented)
    +   zIndex - optional - it defines stacking context, if not defined - stacking context is based on the order in document
    +   transform - optional - it enables to translate, rotate, scale, move transform origin component
+   **props** - component props as component's options - specific properties of element to render

### PTT positioning schemas and units

PPT format distinguishes this positioning schemas (the meaning is the same as CSS positions schemas)  
 
+   __absolute positioning__ - position is assigned with respect to its parent (container)
+   __relative positioning__ - normal flow with support for offset relative to this position - for containers (sections) 

There are two rendering modes

+   __mixed positioning__ - absolute positioning for boxes (widgets) and relative for containers (sections)
+   __normal flow__ - relative positioning for both - boxes and containers 

The units are pixes and be careful is DPI dependend.
 

### PTT pages rendering

This section specifies how the page rendering occurs.

#### Page options

+   page height and width
+   page orientation ('portrait', 'landscape') is optional and defaults to 'portrait'.
+   margin can provided as an object containing one or more of the following properties: 'top', 'left', 'bottom', 'right'

#### Page algorithm

Example of algorithm how to render pages from PTT document definition. 

+   apply data binding for containers (sections) (visibility, ranges for repeatable containers)
+   remove invisible containers
+   expand repeatable containers - cloning row templates
+   transform relative positions to absolute positions
+   reduce to boxes (terminal nodes) - using containers absolute positions (top,height) and its dimensions (with, height)
+   group to pages - create pages and add boxes to them
+   apply data binding for boxes (widgets)


The proprietary implementation of the algorithm can be found [here](https://github.com/rsamec/react-page-renderer/blob/master/src/utilities/transformToPages.js).
