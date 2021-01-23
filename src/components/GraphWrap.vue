<template>
    <div>
        <div id="container"></div>
        <div id="sidebar">
            <edit-feature ref="form" @change="changeObjectValues" :cell-data="currentCell"/>
        </div>
    </div>
</template>

<script>
    import mxgraph from 'mxgraph';
    import EditFeature from "./EditFeature";

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
        mxImage, mxDivResizer, mxObjectCodec, mxCodecRegistry, mxConnectionHandler, mxConstants
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
    window.mxObjectCodec = mxObjectCodec;
    window.mxCodecRegistry = mxCodecRegistry;
    window.mxConnectionHandler = mxConnectionHandler;
    window.mxConstants = mxConstants;

    var editor;
    var doc = mxUtils.createXmlDocument();

    window.CustomUserObject = function(name) {
        this.name = name || "New Feature";
        this.clone = function(){
            return mxUtils.clone(this);
        };
    };

    export default {
        name: 'graph-wrap',
        components: {EditFeature},
        data(){
            return{
                currentCell: null
            }
        },
        methods: {
            selectionChanged(){
                let cell = editor.graph.getSelectionCell();
                this.currentCell = cell;
            },
            changeObjectValues(newCellValue){
                editor.graph.model.setValue(this.currentCell, newCellValue);
            },
            addSideBarIcon: function(graph, sidebar){
                let funct = function(graph, evt){
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

                let wrapper = document.createElement('div');
                wrapper.style.cursor = 'pointer';
                wrapper.style.backgroundColor = '#ffffff';
                wrapper.style.margin = '10px';
                wrapper.style.width = '200px';
                wrapper.style.height = '50px';
                wrapper.style.textAlign = 'center';
                wrapper.style.display = 'flex';
                wrapper.style.flexWrap = 'wrap';
                wrapper.style.alignItems = 'center';
                wrapper.style.justifyContent = 'center';
                wrapper.style.border = '2px dashed black';
                wrapper.innerHTML = '<div>New Feature</div>';
                sidebar.appendChild(wrapper);

                let dragImage = wrapper.cloneNode(true);
                mxUtils.makeDraggable(wrapper, graph, funct, dragImage);
            },
            createGraph(){
                mxConnectionHandler.prototype.connectImage = new mxImage('src/assets/connector.gif', 16, 16);
                if(!mxClient.isBrowserSupported){
                    mxUtils.error('Browser not supported', 200, false);
                }else{
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
                    sidebar.style.backgroundColor = '#2A3C24';

                    if(mxClient.IS_QUIRKS){
                        document.body.style.overflow = 'hidden';
                        new mxDivResizer(container);
                        new mxDivResizer(sidebar);
                    }

                    editor = new mxEditor();
                    editor.setGraphContainer(container);

                    editor.graph.setConnectable(true);
                    editor.graph.setCellsDisconnectable(true);
                    editor.graph.setPanning(true);
                    editor.graph.setAllowDanglingEdges(false);

                    editor.graph.getSelectionModel().addListener(mxEvent.CHANGE, () => {
                        this.selectionChanged();
                    });
                    this.selectionChanged();

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
                        if(cell.value != null && cell.value.name != null){
                            return cell.value.name;
                        }
                        return mxGraph.prototype.convertValueToString.apply(this, arguments);
                    };

                    editor.graph.getLabel = function(cell){
                        if(cell && this.isHtmlLabel(cell) && cell.value){
                            let label = '';
                            label += '<div style="width: 100%;display: flex; justify-content: space-between;align-items: center">';
                            label += '<strong>' + mxUtils.htmlEntities(cell.value.name, false) + '</strong>' 
                            label += '</div>';

                            return label
                        }

                        return mxGraph.prototype.getLabel.apply(this, arguments);
                    };

                    let customObject = new window.CustomUserObject();

                    let object = new mxCell(customObject, new mxGeometry(0, 0, 200, 50), '');
                    object.setVertex(true);
                    object.setConnectable(true);

                    this.addSideBarIcon(editor.graph, sidebar, object);
                }
            },
            init(){
                // let codecCustomUserObject = new mxObjectCodec(new window.CustomUserObject());
                // codecCustomUserObject.encode = function(enc, obj){
                //     let node = enc.document.createElement('CustomUserObject');
                //     mxUtils.setTextContent(node, JSON.stringify(obj));

                //     return node;
                // };
                // codecCustomUserObject.decode = function(dec, node){
                //     let obj = JSON.parse(mxUtils.getTextContent(node));
                //     let beatyObj = new window.CustomUserObject();
                //     obj = Object.assign(beatyObj, obj);
                //     return obj;
                // };
                // mxCodecRegistry.register(codecCustomUserObject);

                this.createGraph();
            },
        },
        mounted(){
            this.init();
        }
    }
</script>