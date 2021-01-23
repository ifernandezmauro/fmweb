<template>
    <div id="container"></div>
</template>

<script>
    import mxgraph from 'mxgraph';

    const graphConfig = {
        mxBasePath: '/mx/',
        mxImagePath: '/mx/images',
        mxLanguage: 'en',
        mxDefaultLanguage: 'en'
    };

    const {
        mxConnectionHandler, mxImage, mxClient, mxUtils, mxToolbar, mxDivResizer, 
        mxGraphModel, mxGraph, mxDragSource, mxCell, mxGeometry
    } = mxgraph(graphConfig);

    export default {
        name: 'graph',
        data(){
            return{
                currentCell: null
            }
        },
        methods: {
            createGraph(){
                mxConnectionHandler.prototype.connectImage = new mxImage('../assets/connector.gif', 16, 16);
                if(!mxClient.isBrowserSupported()){
                    mxUtils.error('Browser is not supported!', 200, false);
                }
                else{
                    var tbContainer = document.createElement('div');
                    tbContainer.style.position = 'absolute';
                    tbContainer.style.overflow = 'hidden';
                    tbContainer.style.padding = '2px';
                    tbContainer.style.left = '0px';
                    tbContainer.style.top = '26px';
                    tbContainer.style.width = '24px';
                    tbContainer.style.bottom = '0px';

                    document.body.appendChild(tbContainer);
                    
                    var toolbar = new mxToolbar(tbContainer);
                    toolbar.enabled = false;

                    var container = document.createElement('div');
                    container.style.position = 'absolute';
                    container.style.overflow = 'hidden';
                    container.style.left = '24px';
                    container.style.top = '26px';
                    container.style.right = '0px';
                    container.style.bottom = '0px';
                    container.style.background = 'url("../assets/grid.gif")';

                    document.body.appendChild(container);

                    //Workaround for Internet Explorer ignoring certain styles
                    if(mxClient.IS_QUIRKS){
                        document.body.style.overflow = 'hidden';
                        new mxDivResizer(tbContainer);
                        new mxDivResizer(container);
                    }

                    var model = new mxGraphModel();
                    var graph = new mxGraph(container, model);
                    graph.dropEnabled = true;

                    mxDragSource.prototype.getDropTarget = function(graph, x, y){
                        var cell = graph.getCellAt(x, y);
                        if(!graph.isValidDropTarget(cell)){
                            cell = null;
                        }
                        return cell;
                    };

                    graph.setConnectable(true);
                    graph.setMultigraph(false);

                    // var keyHandler = new mxKeyHandler(graph);
                    // var rubberband = new mxRubberband(graph);

                    var addVertex = function(icon, w, h, style){
                        var vertex = new mxCell(null, new mxGeometry(0, 0, w, h), style);
                        vertex.setVertex(true);

                        this.addToolbarItem(graph, toolbar, vertex, icon);
                    };

                    addVertex('../assets/rectangle.gif', 100, 40, '');
                    toolbar.addLine();
                }
            },
            addToolbarItem(graph, toolbar, prototype, image){
                var funct = function(graph, evt, cell){
                    graph.stopEditing(false);

                    var pt = graph.getPointForEvent(evt);
                    var vertex = graph.getModel().cloneCell(prototype);
                    vertex.geometry.x = pt.x;
                    vertex.geometry.y = pt.y;

                    graph.setSelectionCells(graph.importCells([vertex], 0, 0, cell));
                }

                var img = toolbar.addMode(null, image, funct);
                mxUtils.makeDraggable(img, graph, funct);
            }
        },
        mounted(){
            this.createGraph();
        }
    }
</script>
