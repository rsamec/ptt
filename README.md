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
     "name": "My first container",
     "elementName": "Container",
     "style": { "top": 0, "left": 0, "height": 200, "width": 740, "position": "relative" },
     "boxes": [{
        "name": "My first text",
        "elementName": "TextContent",
        "style": { "top": 0, "left": 0 },
        "props":{
             "content": "Hello world"
            }
        }]
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

As an example - data binding suppport for each component and its props can be simple added by extending PTT Node by 

+   **bindings** - component bindings - corresponds to component props, each prop can have its own binding exporession
   
```json
{
 "name": "Hello World Example",
 "elementName": "PTTv1",
 "containers": [
    {
     "name": "My first container",
     "elementName": "Container",
     "style": { "top": 0, "left": 0, "height": 200, "width": 740, "position": "relative" },
     "boxes": [{
        "name": "My first text",
        "elementName": "TextContent",
        "style": { "top": 0, "left": 0 },
        "props":{
            "content": "Hello world"
            }
        },
        bindings":{
            "content": "data.message"
            }
        }]
    }]
}
```
The binding expression path __data.message__ is evaluated before rendering occurs and the value __Hello world__ is replaced with the data binding result.

  
## PTT reference usage

The __content applying__ is not the part of the PPT format specification. This section is intended for PTT rendering implementers to guide them when solving layouting and positioning of components.
It is an reference example how to specify PTT rendering. It describes rules how to render content based on PTT document definition. The PTT implementers will define their own layout components and rendering rules acoordings their specific needs.

PTT document layout is mainly specified by the type of containers used in the definition (Container,BackgroundContainer,Page,Grid,Row,Cell,Table etc.).

Typically we can distinguish containers based on responsivness (adjusting components dimensions and positions)

+  responsive (Grid,Row,Cell)
+  static - (Container, Page, BackgroundContainer) - they retain its dimensions and positions

We can distinguish containers based on whatever visual effects provide

+ logical (Container, Page) - has no visual effect - only to grouping logically related components
+ visual - simple layouting only (Row) - e.g. layout visually as seqeuence
+ visual - other visual effect other than simple layouting as background, border, gutters, etc. (BackgroundContainer, Cell, Grid, Table)

### Containers components

The default layout component is __Container__ and belongs to  __static__ and __logical__ containers.
The default responsive component is __Grid__ and belongs to __responsive__ and __visual__ containers.

#### Container

It supports this positioning schemas (the meaning is the same as CSS positions schemas)

+   __absolute positioning__ - position is assigned with respect to its parent (container)
+   __relative positioning__ - normal flow with support for offset relative to this position - for containers (sections) 

Content applying offers two rendering modes

+   __mixed positioning__ - absolute positioning for boxes (widgets) and relative for containers (sections)
+   __normal flow__ - relative positioning for both - boxes and containers 

The final dimension and position is determined by the position schema and the style properties (width, height, top, left, transform) of the PTT Node.

#### Grid

It is based on css flexbox - you can add child containers (Cell) as flex box items.

The final dimension and position is determined by [css flexbox](https://github.com/facebook/css-layout) rules and the style properties (width, height, top, left, transform) of the PTT Node.

### Boxes components (widgets)

All boxes components follow the same rendering rules for positioning.

It supports this positioning schemas (the meaning is the same as CSS positions schemas)

 +   __absolute positioning__ - position is assigned with respect to its parent (container)
 +   __relative positioning__ - normal flow with support for offset relative to this position

For widgets with __static__ container parent - the absolute position is used.
For widgets with __responsive__ container parent - the relative position is used.

The final dimension and position is determined by the position schema and the style properties (width, height, top, left, transform) of the PTT Node.

### PTT pages rendering

This section specifies how the page rendering can occur.

There are two different rendering targets corresponding to two visual media (screen and paper).

#### PTT Node rendering

It renders PTT Node according the elementType as specific component based on target format (HTML, PDF, XML, etc.).

Before rendering some specific rules can occur

+   data binding to props can be used.
+   style binding to props can be used - typically using some resolution strategy
+   some specific properties can be used - visibility, pageBreaks, unbreakable, startOnNewPage

Example container rendering steps
+  apply data binding for containers (sections) (visibility, ranges for repeatable containers)
+  remove invisible containers
+  expand repeatable containers (RepeaterContainer) - cloning row templates

### Screen rendering

Screen rendering prefers responsiveness.

### PTT Screen algorithm

+  apply containers node rendering
+  traverse PTT and apply containers and boxes rendering

```js
let ContainerRenderer =  (props) => {

	var containers = props.containers || [];
	var boxes = props.boxes || [];

	let {node, widgets} = props;
	var elementName = node.elementName;

	var styles = {
		left: props.left,
		top: props.top,
		height: props.height,
		width: props.width,
		position: props.position || 'relative'
	};

	var containerComponent = widgets[elementName] || 'div';

	return (<div style={styles}>
		{containers.length !== 0 ? React.createElement(containerComponent, nodeProps, containers.map(function (container, index) {

			var key = container.name + index;

			var left = container.style.left === undefined ? 0 : parseInt(container.style.left, 10);
			var top = container.style.top === undefined ? 0 : parseInt(container.style.top, 10);

			var childComponent = widgets[container.elementName] || 'div';

			return (React.createElement(childComponent, _.extend({child: true, key: key}, childProps),
				<ContainerRenderer key={key}
								   index={index}
								   left={left}
								   top={top}
								   height={container.style.height}
								   width={container.style.width}
								   position={container.style.position || 'relative'}
								   boxes={container.boxes}
								   containers={container.containers}
								   node={container}
								   parent={props.parent}
								   widgets={props.widgets}
								   widgetRenderer={props.widgetRenderer}/>
			));
		}, this)) : null}

		{boxes.map(function (box, index) {

			var key = box.name + index;

			var elName = box.elementName;
			var widget = React.createElement(props.widgetRenderer, {
				widget: props.widgets[elName],
				node: box
			}, null);

			return (
				<div key={key} style={box.style}>
					<div id={box.name}>{widget}</div>
				</div>
			);


		}, this)
		}

	</div>)
}
```

### Paper rendering

Paper rendering requires static elements (absolutely positioning).

#### PTT Paper options

+   page height and width
+   page orientation ('portrait', 'landscape') is optional and defaults to 'portrait'.
+   margin can provided as an object containing one or more of the following properties: 'top', 'left', 'bottom', 'right'

#### PTT Paper algorithm

Example of algorithm how to render pages from PTT document definition. 

+   apply containers rendering
+   transform relative positions to absolute positions
+   reduce to boxes (terminal nodes) - using containers absolute positions (top,height) and its dimensions (with, height)
+   group to pages - create pages and add boxes to them
+   for all pages - create page, apply boxes rendering rendering

```js
function transformToPages(clonedSchema,pageHeight){

    const BOXES_COLLECTION_NAME = "boxes";
    const CONTAINERS_COLLECTION_NAME = "containers";
    const DEFAULT_PAGE_HEIGHT = 1065;
	

    //step -> transform relative positions to absolute positions
    if (pageHeight === undefined) pageHeight = DEFAULT_PAGE_HEIGHT;
    var globalTop = 0;
    var trav = function(node){

        if (node === undefined) return 0;

        var props = node.props || {};

        //grap height and top properties
        var nodeHeight = (node.style === undefined)?0:parseInt(node.style.height, 10);
        if (isNaN(nodeHeight)) nodeHeight = 0;
        var nodeTop = (node.style === undefined)?0:parseInt(node.style.top, 10);
        if (isNaN(nodeTop)) nodeTop = 0;


        var children = node.containers || [];
        var computedHeight = 0;
        if (children === undefined) return computedHeight;
        var childrenHeight = 0;

		//unbreakable -> if section is too height to have enough place to fit the the page - move it to the next page
		var startOnNewPage =  false;
		if (!!props.unbreakable){
			var nodeBottom = globalTop + nodeHeight;
			var nextPageTop = Math.ceil(globalTop/pageHeight) * pageHeight;
			startOnNewPage = nodeBottom > nextPageTop;
		}

		//startOnNewPage - move globalTop to the next page
		if (!!props.startOnNewPage || startOnNewPage) globalTop = Math.ceil(globalTop/pageHeight) * pageHeight;


        //set absolute top property - use last global top + node top (container can have top != 0)
        if (node.style !== undefined) node.style.top = globalTop + nodeTop;

        //recurse by all its children containers
        for (var i in children)
        {
            childrenHeight += trav(children[i]);
        }

        //expand container height if childrenHeight is greater than node height - typically for repeated containers
        computedHeight = _.max([nodeHeight,childrenHeight]) +  nodeTop;

        //compute next top
        globalTop += (computedHeight-childrenHeight);
        
		//return computed height of container
        return computedHeight;
    };
    trav(clonedSchema);


	traverse(clonedSchema).reduce(function (occ,x) {

		if (this.key === CONTAINERS_COLLECTION_NAME) {
			for (var i in x) {
				var el = x[i];
			}
		}
	});
	

	//step -> reduce to boxes - using containers absolute positions (top,height) and its dimensions (with, height)
    //step -> create pages and add boxes to them
    var pages = [];
    var currentPage;
    traverse(clonedSchema).reduce(function (occ,x) {

		
        if (this.key === BOXES_COLLECTION_NAME){
            var parent = this.parent.node;
            for (var i in x){
                var el = x[i];
			
				var elTop = el.style.top && parseInt(el.style.top,10) || 0;
				var elLeft = el.style.left && parseInt(el.style.left,10) || 0;

				var parentStyle = parent.style || {};
                //grab parent positions
                var top = (parentStyle.top && parseInt(parentStyle.top,10) || 0) + elTop;
                var left = (parentStyle.left && parseInt(parentStyle.left,10) || 0) + elLeft;

                //grab parent dimensions
                //TODO: !!!! temporarily - container width simulates boxes width
                var height = (parentStyle.height && parseInt(parentStyle.height, 10) || 0) - elTop;
                var width = (parentStyle.width && parseInt(parentStyle.width, 10) || 0) - elLeft;
                //var height = parseInt(el.style.height,10);
                //var width = parseInt(el.style.width,10);
                if (isNaN(height)) height = 0;
                if (isNaN(width)) width = 0;


                //create newPage
                if (currentPage === undefined || (top + height) > pageHeight * pages.length){
                    var newPage ={pageNumber:pages.length + 1,boxes:[]};
                    pages.push(newPage);
                    currentPage = newPage;
                }

                //decrease top according the pages
                if (pages.length > 1){ top -= (pages.length -1) * pageHeight };

                var style = {'left':left,'top':top,'position':'absolute'};
                if (el.style.width!== undefined) style.width = el.style.width;
                if (el.style.height!== undefined) style.height = el.style.height;
                if (el.style.zIndex!== undefined) style.zIndex = el.style.zIndex;

                //propagate width and height to widget props
                if (!el.props.width && !!el.style.width) el.props.width = el.style.width;
                if (!el.props.height&& !!el.style.height) el.props.height = el.style.height;

                if (el.style.transform !== undefined) {
                    style.WebkitTransform = generateCssTransform(el.style.transform);
                    style.transform = generateCssTransform(el.style.transform);
                }
                // set another box
                currentPage.boxes.push({element:x[i],style:style});
            }
        }
        return occ;
    }, pages);

    return pages;
};


let createBoxedPage = function (page, i) {
			var back = normalizeBackgrounds[i];
			return (<HtmlPage key={'page' + i} position={i} pageNumber={page.pageNumber} widgets={widgets}
							  background={back} pageOptions={pageOptions}>
				{page.boxes.map(function (node, j) {
					var elName = node.element.elementName;
					var widget = <WidgetRenderer key={'page' + i + '_' + j} widget={widgets[elName]}
												 node={node.element}
												 customStyle={customStyles[elName]} dataBinder={dataContext}/>;
					return (
						<div key={'item' + j} style={ node.style}>
							<div id={node.element.name}>{widget}</div>
						</div>
					);
				}, this)}
			</HtmlPage>)
		};
```

The example html renderer implementation can be found [react-html-pages-renderer](https://github.com/rsamec/react-html-pages-renderer).
