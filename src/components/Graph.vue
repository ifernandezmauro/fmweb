<template>
    <div> 
        <div id="container"></div>
        <div id="sidebar">
            <div id="properties"/>
            <div id="relationships"/>
            <div id="relationshipType"/>
            <div style="textAlign:center;color:white;fontSize:20px;fontWeight:bold">Relationships</div>
        </div>
        
    </div>
</template>

<script>
    import mxgraph from 'mxgraph';

    const graphConfig = {
        mxBasePath: '/mx/',
        mxImagePath: '/mx/images',
        mxLanguage: 'en',
        mxDefaultLanguage: 'en',
        mxLoadResources: false,
        mxLoadStylesheets: false
    };

    const {
        mxClient, mxUtils, mxEvent, mxEditor, mxRectangle, mxGraph, mxGeometry, mxCell, 
        mxImage, mxDivResizer, mxConnectionHandler, mxConstants, mxRubberband, mxCodec, mxForm,
        mxCellAttributeChange, mxShape, mxConnectionConstraint, mxPoint, mxEventObject, mxGraphModel
    } = mxgraph(graphConfig);

    window.mxClient = mxClient;
    window.mxUtils = mxUtils;
    window.mxRectangle = mxRectangle;
    window.mxGraph = mxGraph;
    window.mxEvent = mxEvent;
    window.mxGeometry = mxGeometry;
    window.mxCell = mxCell;
    window.mxImage = mxImage;
    window.mxEditor = mxEditor;
    window.mxDivResizer = mxDivResizer;
    window.mxConnectionHandler = mxConnectionHandler;
    window.mxConstants = mxConstants;
    window.mxRubberband = mxRubberband;
    window.mxCodec = mxCodec;
    window.mxForm = mxForm;
    window.mxCellAttributeChange = mxCellAttributeChange;
    window.mxShape = mxShape;
    window.mxConnectionConstraint = mxConnectionConstraint;
    window.mxPoint = mxPoint;
    window.mxEventObject = mxEventObject;
    window.mxGraphModel = mxGraphModel;

    var editor;
    var doc = mxUtils.createXmlDocument();
    var relationType = 'Optional';

    export default {
        name: 'graph-wrap',
        methods: {
            addSideBarIcon: function(graph, sidebar){
                let newFeature = function(graph, evt){
                    graph.stopEditing(false);

                    let pt = graph.getPointForEvent(evt);

                    let parent = graph.getDefaultParent();
                    let model = graph.getModel();

                    let newFeature = doc.createElement('Feature');
                    newFeature.setAttribute('name', 'New Feature');

                    model.beginUpdate();
                    try{
                        var v1 = graph.insertVertex(parent, null, newFeature, pt.x, pt.y, 120, 60);
                    }finally{
                        model.endUpdate();
                    }

                    graph.setSelectionCell(v1);
                };

                //View Model's XML
                // sidebar.appendChild(mxUtils.button('View XML', function(){
                //     let encoder = new mxCodec();
                //     let node = encoder.encode(editor.graph.getModel());
                //     mxUtils.popup(mxUtils.getPrettyXml(node), true);
                // }));

                let wrapper = document.createElement('div');
                wrapper.style.cursor = 'pointer';
                wrapper.style.backgroundColor = '#ffffff';
                wrapper.style.margin = '10px';
                wrapper.style.width = '150px';
                wrapper.style.height = '50px';
                wrapper.style.textAlign = 'center';
                wrapper.style.display = 'flex';
                wrapper.style.flexWrap = 'wrap';
                wrapper.style.alignItems = 'center';
                wrapper.style.justifyContent = 'center';
                wrapper.style.border = '2px solid black';
                wrapper.innerHTML = '<div>New Feature</div>';
                sidebar.appendChild(wrapper);

                let dragImage = wrapper.cloneNode(true);
                mxUtils.makeDraggable(wrapper, graph, newFeature, dragImage);
            },
            createTextField(graph, form, cell, attribute){
                let input = form.addText(attribute.nodeName + ':', attribute.nodeValue);

                let applyHandler = function(){
                    let newValue = input.value || '';
                    let oldValue = cell.getAttribute(attribute.nodeName, '');

                    if(newValue != oldValue){
                        graph.getModel().beginUpdate();
                        try{
                            cell.setAttribute(attribute.nodeName, newValue);
                            graph.updateCellSize(cell);
                        }
                        finally{
                            graph.getModel().endUpdate();
                        }
                    }
                };

                mxEvent.addListener(input, 'keypress', function(evt){
                    if(evt.keyCode == /*enter*/13 && !mxEvent.isShiftDown(evt)){
                        input.blur();
                    }
                });

                if(mxClient.IS_IE){
                    mxEvent.addListener(input, 'focusout', applyHandler);
                }
                else{
                    mxEvent.addListener(input, 'blur', applyHandler);
                }
            },
            selectionChanged(){
                let div = document.getElementById('properties');
                editor.graph.container.focus();
                div.innerHTML = '';

                div.style.backgroundColor = '#ffffff';
                div.style.margin = '10px';
                div.style.padding = '10px';
                div.style.textAlign = 'center';
                div.style.flexWrap = 'wrap';
                div.style.justifyContent = 'center';
                div.style.border = '2px solid black';

                let cell = editor.graph.getSelectionCell();
                
                if(cell == null){
                    mxUtils.writeln(div, 'None selected');
                }
                else{
                    let center = document.createElement('center');
                    mxUtils.writeln(center, cell.value.nodeName);
                    div.appendChild(center);
                    mxUtils.br(div);

                    if(cell.value.nodeName == 'Feature'){
                        let form = new mxForm();
                        let attrs = cell.value.attributes;
                        if(attrs != null){
                            for(let i = 0; i < attrs.length; i++){
                                this.createTextField(editor.graph, form, cell, attrs[i]);
                            }
                        }

                        div.appendChild(form.getTable());
                    }
                    else if(cell.value.nodeName == 'Relationship'){
                        let attrs = cell.value.attributes;
                        if(attrs != null){
                            for(let i = 0; i < attrs.length; i++){
                                if(attrs[i].nodeName == 'source' || attrs[i].nodeName == 'target'){
                                    mxUtils.writeln(div, attrs[i].nodeName + ': ' + attrs[i].nodeValue);
                                }
                                else{
                                    mxUtils.writeln(div, attrs[i].nodeName + ': ' + attrs[i].nodeValue);
                                }
                            }
                        }
                    }
                    
                    mxUtils.br(div);

                    let button = document.createElement('button');
                    mxUtils.write(button, 'Delete');

                    mxEvent.addListener(button, 'click', function(){
                        editor.graph.getModel().beginUpdate();
                        try{
                            editor.graph.removeCells([cell]);
                        }
                        finally{
                            editor.graph.getModel().endUpdate();
                        }
                    });

                    div.appendChild(button);

                    let wrapRelationships = document.getElementById('relationships');
                    wrapRelationships.innerHTML = '';
                    wrapRelationships.style.margin = '10px';
                    wrapRelationships.style.width = '150px';
                    wrapRelationships.style.height = '150px';
                    wrapRelationships.style.textAlign = 'center';
                    wrapRelationships.style.display = 'grid';

                    let self = this;

                    let optBtn = document.createElement('button');
                    mxUtils.writeln(optBtn, 'Optional');
                    mxEvent.addListener(optBtn, 'click', function(){
                        if(cell.value.nodeName != 'Relationship'){
                            relationType = 'Optional';
                            self.currentRelationship();
                        }
                    });
                    wrapRelationships.appendChild(optBtn);
                    
                    let manBtn = document.createElement('button');
                    mxUtils.writeln(manBtn, 'Mandatory');
                    mxEvent.addListener(manBtn, 'click', function(){
                        if(cell.value.nodeName != 'Relationship'){
                            relationType = 'Mandatory';
                            self.currentRelationship();
                        }
                    });
                    wrapRelationships.appendChild(manBtn);

                    let altBtn = document.createElement('button');
                    mxUtils.writeln(altBtn, "Alternative");
                    mxEvent.addListener(altBtn, 'click', function(){
                        if(cell.value.nodeName != 'Relationship'){
                            relationType = 'Alternative';
                            self.currentRelationship();
                        }
                    });
                    wrapRelationships.appendChild(altBtn);

                    let orBtn = document.createElement('button');
                    mxUtils.writeln(orBtn, 'Or');
                    mxEvent.addListener(orBtn, 'click', function(){
                        if(cell.value.nodeName != 'Relationship'){
                            relationType = 'Or';
                            self.currentRelationship();
                        }
                    });
                    wrapRelationships.appendChild(orBtn);

                    let reqBtn = document.createElement('button');
                    mxUtils.writeln(reqBtn, 'Requires');
                    mxEvent.addListener(reqBtn, 'click', function(){
                        if(cell.value.nodeName != 'Relationship'){
                            relationType = 'Requires';
                            self.currentRelationship();
                        }
                    });
                    wrapRelationships.appendChild(reqBtn);

                    let excBtn = document.createElement('button');
                    mxUtils.writeln(excBtn, 'Excludes');
                    mxEvent.addListener(excBtn, 'click', function(){
                        if(cell.value.nodeName != 'Relationship'){
                            relationType = 'Excludes';
                            self.currentRelationship();
                        }
                    });
                    wrapRelationships.appendChild(excBtn);
                }
            },
            currentRelationship(){
                let div = document.getElementById('relationshipType');
                div.innerHTML = '';
                div.style.color = 'white';
                div.style.fontsize = '15px';
                div.style.textAlign = 'center';
                div.style.justifyContent = 'center';
                
                mxUtils.writeln(div, 'Current: ' + relationType);
            },
            createGraph(){
                mxGraph.prototype.getAllConnectionConstraints = function(terminal){
                    if(terminal != null && terminal.shape != null){
                        if(terminal.shape.stencil != null){
                            if(terminal.shape.stencil.constraints != null){
                                return terminal.shape.stencil.constraints;
                            }
                        }
                        else if(terminal.shape.constraints != null){
                            return terminal.shape.constraints;
                        }
                    }
                    return null;
                }
                mxShape.prototype.constraints = [new mxConnectionConstraint(new mxPoint(0.25, 0), true),
                                                 new mxConnectionConstraint(new mxPoint(0.5, 0), true),
                                                 new mxConnectionConstraint(new mxPoint(0.75, 0), true),
                                                 new mxConnectionConstraint(new mxPoint(0, 0.25), true),
                                                 new mxConnectionConstraint(new mxPoint(0, 0.5), true),
                                                 new mxConnectionConstraint(new mxPoint(0, 0.75), true),
                                                 new mxConnectionConstraint(new mxPoint(1, 0.25), true),
                                                 new mxConnectionConstraint(new mxPoint(1, 0.5), true),
                                                 new mxConnectionConstraint(new mxPoint(1, 0.75), true),
                                                 new mxConnectionConstraint(new mxPoint(0.25, 1), true),
                                                 new mxConnectionConstraint(new mxPoint(0.5, 1), true),
                                                 new mxConnectionConstraint(new mxPoint(0.75, 1), true)]
            
                mxConnectionHandler.prototype.connectImage = new mxImage(require('../assets/connector.gif'), 16, 16);

                if(!mxClient.isBrowserSupported){
                    mxUtils.error('Browser not supported', 200, false);
                }
                else{
                    let container = document.getElementById('container');
                    container.style.position = 'absolute';
                    container.style.overflow = 'hidden';
                    container.style.left = '250px';
                    container.style.top = '0px';
                    container.style.right = '0px';
                    container.style.bottom = '0px';
                    container.style.background = `url("${require('../assets/grid.gif')}")`;

                    let sidebar = document.getElementById('sidebar');
                    sidebar.style.position = 'absolute';
                    sidebar.style.overflow = 'hidden';
                    sidebar.style.padding = '0px';
                    sidebar.style.left = '0px';
                    sidebar.style.top = '0px';
                    sidebar.style.width = '250px';
                    sidebar.style.bottom = '0px';
                    sidebar.style.display = 'flex';
                    sidebar.style.flexDirection = 'column-reverse';
                    sidebar.style.alignItems = 'center';
                    sidebar.style.justifyContent = 'flex-end';
                    sidebar.style.backgroundColor = '#5C575E';

                    if(mxClient.IS_QUIRKS){
                        document.body.style.overflow = 'hidden';
                        new mxDivResizer(container);
                        new mxDivResizer(sidebar);
                    }

                    editor = new mxEditor();
                    editor.setGraphContainer(container);

                    editor.graph.connectionHandler.factoryMethod = function(source, target, style){
                        let newRelationship = doc.createElement('Relationship');
                        newRelationship.setAttribute('type', relationType);

                        if(relationType == 'Optional' || relationType == 'Mandatory'){
                            newRelationship.setAttribute('relationship', 'Parenthood');
                            if(relationType == 'Optional') style = 'startArrow=0;endArrow=oval;endFill=0';
                            else if(relationType == 'Mandatory') style = 'startArrow=0;endArrow=oval;endFill=1';
                        }
                        else if(relationType == 'Alternative' || relationType == 'Or'){
                            newRelationship.setAttribute('relationship', 'LogicAssociation');
                            if(relationType == 'Alternative') style = 'startArrow=block;endArrow=block;startFill=0;endFill=0';
                            else if(relationType == 'Or') style = 'startArrow=block;endArrow=block;startFill=1;endFill=1';
                        }
                        else if(relationType == 'Requires' || relationType == 'Excludes'){
                            newRelationship.setAttribute('relationship', 'Dependency');
                            if(relationType == 'Requires') style = 'dashed=1;startArrow=0;endArrow=block;endFill=1';
                            else if(relationType == 'Excludes') style = 'dashed=1;startArrow=block;endArrow=block;startFill=1;endFill=1';
                        }

                        let edge = new mxCell(newRelationship, new mxGeometry());
                        edge.setEdge(true);
                        edge.setStyle(style);
                        edge.geometry.relative = true;
                        return edge;
                    }

                    let self = this;
                    editor.graph.connectionHandler.addListener(mxEvent.CONNECT, function(sender, evt){
                        let edge = evt.getProperty('cell');
                        let target = editor.graph.getModel().getTerminal(edge, false);
                        let source = editor.graph.getModel().getTerminal(edge, true);

                        //DEF01, DEF02, CST01
                        if(relationType == 'Mandatory' || relationType == "Optional"){
                            let minCardinality = mxUtils.prompt("Minimum Cardinality");
                            let maxCardinality = mxUtils.prompt("Maximum Cardinality");
                            edge.setAttribute('minCardinality', minCardinality);
                            edge.setAttribute('maxCardinality', maxCardinality);

                            //DEF01
                            if(relationType == "Mandatory"){
                                if(!(minCardinality > 0)){
                                    editor.graph.removeCells([edge]);
                                    mxUtils.alert("DEF01. Can't create Mandatory. Minimum cardinality must be greater than 0");
                                }
                            }
                            //DEF02
                            else if(relationType == "Optional"){
                                if(!(minCardinality == 0)){
                                    editor.graph.removeCells([edge]);
                                    mxUtils.alert("DEF02. Can't create Optional. Minimum cardinality must be equal to 0");
                                }
                            }
                            //CST01
                            if(!((minCardinality >= 0) && (minCardinality <= maxCardinality))){
                                editor.graph.removeCells([edge]);
                                mxUtils.alert("CST01. Cant't create " + relationType + ". Minimum Cardinality < 0 or Minimum Cardinality > Maximum Cardinality");
                            }
                        }
                        
                        //CST04, CST05
                        if(relationType == "Or" || relationType == "Alternative"){
                            //CST04
                            let rel = self.fuseArrays(self.getRelationships("Mandatory"), self.getRelationships("Optional"));
                            let a = self.filterTarget(target, rel);
                            let b = self.filterTarget(source, rel);
                            a = self.getSources(a);
                            b = self.getSources(b);
                            if(!self.compareVertex(a, b)){
                                editor.graph.removeCells([edge]);
                                mxUtils.alert("CST04. Can't create " + relationType + ". Only between brothers");
                            }
                            //CST05
                            rel = self.getRelationships("Mandatory");
                            a = self.filterTarget(target, rel);
                            b = self.filterTarget(source, rel);
                            a = self.getSources(a);
                            b = self.getSources(b);
                            if(!self.compareVertex(a, b)){
                                editor.graph.removeCells([edge]);
                                mxUtils.alert("CST05. Can't create " + relationType + ". Only between Mandatory Features");
                            }
                        }

                        //CST07, CST08
                        if(relationType == "Requires" || relationType == "Excludes"){
                            //CST07
                            let rel = self.fuseArrays(self.getRelationships("Mandatory"), self.getRelationships("Optional"));
                            let a = self.filterTarget(target, rel);
                            let b = self.filterTarget(source, rel);
                            a = self.getSources(a);
                            b = self.getSources(b);
                            if(self.compareVertex(a, b)){
                                editor.graph.removeCells([edge]);
                                mxUtils.alert("CST07. Can't create " + relationType + " between brothers");
                            }

                            // //CST08
                            // if(relationType == "Excludes"){
                            //     let descendantsArray = new Array();
                            //     descendantsArray = self.getDescendants(source, descendantsArray);
                            //     console.log(descendantsArray);
                            //     if(!self.findInArray(descendantsArray, target)){
                            //         mxUtils.alert("CST08 se cumple");
                            //     }
                            //     else{
                            //         mxUtils.alert("CST08 NO se cumple");
                            //     }
                            // }
                        }
                    });

                    editor.graph.setConnectable(true);
                    editor.graph.setCellsDisconnectable(true);
                    editor.graph.setPanning(true);
                    editor.graph.setAllowDanglingEdges(false);

                    editor.graph.centerZoom = false;
                    editor.graph.swimlaneNesting = false;
                    editor.graph.dropEnabled = true;

                    editor.graph.isHtmlLabel = function(cell){
                        return !this.isSwimlane(cell) && !this.model.isEdge(cell);
                    };

                    editor.graph.isCellEditable = function(){
                        return false;
                    };

                    editor.graph.convertValueToString = function(cell){
                        if(mxUtils.isNode(cell.value)){
                            if(cell.value.nodeName.toLowerCase() == 'feature'){
                                let featureName = cell.getAttribute('name', '');
                                return featureName;
                            }
                        }
                        return '';
                    };

                    this.addSideBarIcon(editor.graph, sidebar);

                    //Add Rubberband selection
                    // new mxRubberband(editor.graph);

                    //Change Vertex style
                    let style = editor.graph.getStylesheet().getDefaultVertexStyle();
                    style[mxConstants.STYLE_STROKECOLOR] = 'black';
                    style[mxConstants.STYLE_FILLCOLOR] = 'white';
                    style[mxConstants.STYLE_FONTCOLOR] = 'black';
                    style[mxConstants.STYLE_FONTSIZE] = '12';
                    style[mxConstants.STYLE_SPACING] = 4;

                    //Change Edges style
                    style = editor.graph.getStylesheet().getDefaultEdgeStyle();
                    style[mxConstants.STYLE_STROKECOLOR] = 'black';
                    style[mxConstants.STYLE_LABEL_BACKGROUNDCOLOR] = 'white';
                    style[mxConstants.STYLE_FONTCOLOR] = 'black';
                    style[mxConstants.STYLE_FONTSIZE] = 10;

                    //Properties pannel
                    editor.graph.getSelectionModel().addListener(mxEvent.CHANGE, () =>{
                        this.selectionChanged();
                        this.currentRelationship();
                    });
                    this.selectionChanged();
                    this.currentRelationship();
                }
            },
            getDescendants(cell, descendantsArray){
                //Gets all descendants of a given cell
                let self = this;
                let outgoing = mxGraphModel.prototype.getOutgoingEdges(cell);
                let descendants = self.getTargets(outgoing);
                if(descendants.length > 0){
                    descendantsArray = self.fuseArrays(descendantsArray, descendants);
                    for(let i = 0; i < descendants.length; i++){
                        self.getDescendants(descendants[i], descendantsArray);
                    }
                }
                return descendantsArray;
            },
            getAncestors(cell, ancestorsArray){
                //Gets all ancestors of a given cell
                let self = this;
                let incoming = mxGraphModel.prototype.getIncomingEdges(cell);
                let ancestors = self.getSources(incoming);
                if(ancestors.length > 0){
                    ancestorsArray = self.fuseArrays(ancestorsArray, ancestors);
                    for(let i = 0; i < ancestors.length; i++){
                        self.getAncestors(ancestors[i], ancestorsArray);
                    }
                }
                return ancestorsArray;
            },
            getRelationships(relationshipType){
                //Gets all relationships of the given type
                let allCells = editor.graph.getDefaultParent().children;
                let edges = new Array();
                for(let i = 0; i < allCells.length; i++){
                    if(!allCells[i].isVertex()){
                        if(allCells[i].value.attributes.type.nodeValue == relationshipType){
                            edges.push(allCells[i]);
                        } 
                    }
                }
                return edges;
            },
            filterSource(sourceVertex, arrayEdges){
                //Gets all edges in arrayEdges where sourceVertex is the source
                let finalArray = new Array();
                for(let i = 0; i < arrayEdges; i++){
                    if(editor.graph.getModel().getTerminal(arrayEdges[i], true) == sourceVertex){
                        finalArray.push(arrayEdges[i]);
                    }
                }
                return finalArray;
            },
            filterTarget(targetVertex, arrayEdges){
                //Gets all edges in arrayEdges where targetVertex is the target
                let finalArray = new Array();
                for(let i = 0; i < arrayEdges.length; i++){
                    if(editor.graph.getModel().getTerminal(arrayEdges[i], false) == targetVertex){
                        finalArray.push(arrayEdges[i]);
                    }
                }
                return finalArray;
            },
            getSources(arrayEdges){
                //Gets all the sources from the edges in arrayEdges
                let arraySources = new Array();
                for(let i = 0; i < arrayEdges.length; i++){
                    arraySources.push(editor.graph.getModel().getTerminal(arrayEdges[i], true));
                }
                return arraySources;
            },
            getTargets(arrayEdges){
                //Gets all the targets from the edges in arrayEdges
                let arrayTargets = new Array();
                for(let i = 0; i < arrayEdges.length; i++){
                    arrayTargets.push(editor.graph.getModel().getTerminal(arrayEdges[i], false));
                }
                return arrayTargets;
            },
            compareVertex(arrayVertexA, arrayVertexB){
                //Compares if there is any coincidence from the vertex in arrayA and arrayB. true = any coincidences found
                for(let i = 0; i < arrayVertexA.length; i++){
                    for(let j = 0; j < arrayVertexB.length; j++){
                        if(arrayVertexA[i] == arrayVertexB[j]){
                            return true;
                        }
                    }
                }
                return false;
            },
            findInArray(arrayVertex, vertex){
                //Looks if vertex is present in arrayVertex. true = vertex found
                for(let i = 0; i < arrayVertex.length; i++){
                    if(arrayVertex[i] == vertex){
                        return true;
                    }
                }
                return false;
            },
            fuseArrays(arrayA, arrayB){
                //Fuses arrayA and arrayB getting rid of all duplicates
                let finalArray = new Array();
                for(let i = 0; i < arrayA.length; i++){
                    if(!finalArray.includes(arrayA[i])){
                        finalArray.push(arrayA[i]);
                    }
                }
                for(let i = 0; i < arrayB.length; i++){
                    if(!finalArray.includes(arrayB[i])){
                        finalArray.push(arrayB[i]);
                    }
                }
                return finalArray;
            }
        },
        mounted(){
            this.createGraph();
        }
    }
</script>