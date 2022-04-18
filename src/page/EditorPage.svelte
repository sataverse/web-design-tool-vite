<script>

    import CustomTool1 from './component/CustomTool1.svelte'
    import CustomTool2 from './component/CustomTool2.svelte'
    import CustomLayer from './component/CustomLayer.svelte'
    import CustomProperty from './component/CustomProperty.svelte'

    import { windowHeight } from './store.js';
    import { toolSectionHeight } from './store.js';
    import { radioSection } from './store.js';

    import { onMount } from "svelte";
    import { fabric } from "fabric";
    import JSZip from "jszip"



    let innerWidth = window.innerWidth;
    let innerHeight = window.innerHeight;

    let CustomPropertyChild;

    

    class ThemeClass {
        constructor() {
            this.mainColor = '#18A0FB';         // 메인색상
            this.backgroundColor = '#FFFFFF';   // 배경색상
            this.fontColor1 = '#404040';        // 폰트색상1 - 배경이 배경색상일때
            this.fontColor2 = '#FFFFFF';        // 폰트색상2 - 배경이 메인색상일때
            this.hoverColor = '#E6E6E6';        // 마우스 오버시 색상
        }
    }

    class CurrentObjectClass {
        constructor() {
            this.object = null;
            this.index = null;
            this.type = null;
            this.id = null;
            this.x = null;
            this.y = null;
            this.width = null;
            this.height = null;
            this.color = null;
            this.src = null;
            this.rotate = null;
            this.fontSize = null;
            this.stroke = null;
            this.strokeWidth = null;
            this.tagType = null;
            this.Event = new class Event {
                constructor() {
                    this.when = null;
                    this.do = null;
                    this.detail = null;
                }
            };
        }
    }

    class PageClass {
        constructor(index, path, canvas, canvasId, canvasWrapperId) {
            this.index = index;
            this.path = path;
            this.canvas = canvas;
            this.zoom = 0.3;
            this.canvasId = canvasId;
            this.canvasWrapperId = canvasWrapperId;
            this.object = new Array(0);
            this.defaultObject = new class DefaultObjectClass {
                constructor() {
                    this.box = null;
                    this.title = null;
                }
            }
            this.selectComponent = new Array(0);
        }
    }

    class SelectComponentClass {
        constructor() {
            this.componentId = null;
            this.componentImage = null;
            this.x = null;
            this.y = null;
        }
    }

    class ComponentClass {
        constructor(index, path, canvas, canvasId, canvasWrapperId) {
            this.index = index;
            this.path = path;
            this.canvas = canvas;
            this.zoom = 0.3;
            this.canvasId = canvasId;
            this.canvasWrapperId = canvasWrapperId;
            this.object = new Array(0);
            this.defaultObject = new class DefaultObjectClass {
                constructor() {
                    this.box = null;
                    this.title = null;
                }
            }
            this.componentProperty = new class ComponentPropertyClass {
                constructor() {
                    this.imageSrc = null;
                    this.width = null;
                    this.height = null;
                }
            };
        }
    }

    class ObjectClass {
        constructor(object, index, componentIndex, tagType) {
            this.object = object;
            this.index = index;
            this.componentIndex = componentIndex;
            this.tagType = tagType;
        }
    }

    class CurrentComponentClass {
        constructor() {
            this.width = null;
            this.height = null;
        }
    }


    class UIClass {
        constructor() {
            this.mouseType = 1;
            //this.clickedObjectType = 'rect';
            this.propertyOrEvent = 'property';
            this.pageOrComponent = 'page';
            this.currentZoom = null;
            this.currentPageMode = 'page';
            this.componentIndex = 0;
            this.currentPagePath = '/';
            this.pageIndex = 0;
            this.currentComponentObjectArray = new Array(0);
            this.objectType = 'page';
            this.moveViewBool = false;
            this.mouseMiddleDown = false;
            this.isDown = false;
            this.origX = null;
            this.origY = null;
            this.isMove = false;
            this.isScale = false;
            this.tempArray = new Array(0);
            this.currentSelect = new Array(0);
        }
    }



    let ui = new UIClass();
    let theme = new ThemeClass();
    let currentObject = new CurrentObjectClass();
    let currentComponent = new CurrentComponentClass();
    let pageArray = new Array(0);
    //pageArray[0] = new PageClass(0,'/Page0',null,'','');
    //pageArray[pageArray.length] = new PageClass(pageArray.length, '/Page' + pageArray.length ,null,'','');

    let componentArray = new Array(0);
    //ui.currentComponentObjectArray[0] = new ObjectClass(null,null,null,null);
    //componentArray[0] = new ComponentClass(0, '/Component0',null,'','');
    //componentArray[componentArray.length] = new ComponentClass(componentArray.length, '/Component' + componentArray.length ,null,'','');
    

    onMount(async () => {

        fabric.Object.prototype.set({
            borderColor: "#18A0FB",
            cornerColor: "#18A0FB",
            cornerSize: 8,
            transparentCorners: false,
        });

        // 기본 페이지 설정
        function setDefaultPage() {
            pageArray[0] = new PageClass(0, '/', new fabric.Canvas('canvas-page-' + 0, {
                    objectCaching:false,
                    imageSmoothingEnabled: false,
                    fireMiddleClick: true,
                }),
                'canvas-page-0',
                'canvas-page-0-wrapper',
            );

            pageArray[0].canvas.setWidth(innerWidth - 400);
            pageArray[0].canvas.setHeight(innerHeight - 50);
            pageArray[0].defaultObject.box = new fabric.Rect({
                id: '///defaultBox///',
                left: 0,
                top: 0,
                strokeWidth: 0,
                fill: "#ffffff",
                stroke: "#ffffff",
                width: 1920,
                height: 1080,
            })
            pageArray[0].defaultObject.title = new fabric.Textbox('/', {
                id: '///defaultText///',
                fontFamily: 'Noto Sans KR',
                width: 200,
                top: -40,
                fontSize: 20,
                fontWeight: '30',
                fill: '#404040',
            })
            pageArray[0].defaultObject.box.selectable = true;
            pageArray[0].defaultObject.box.hasControls = true;
            pageArray[0].defaultObject.box.lockMovementX = true;
            pageArray[0].defaultObject.box.lockMovementY = true;
            pageArray[0].defaultObject.box.setControlsVisibility({
                mt: false,
                mb: true,
                ml: false,
                mr: false,
                bl: false,
                br: false,
                tr: false,
                tl: false,
                mtr: false,
            });
            pageArray[0].defaultObject.title.selectable = false;
            pageArray[0].defaultObject.title.hasControls = false;

            pageArray[0].canvas.add(pageArray[0].defaultObject.box);
            pageArray[0].canvas.add(pageArray[0].defaultObject.title);

            pageArray[0].canvas.defaultCursor = `url("../icon/Cursor1.png"), auto`;
            pageArray[0].canvas.hoverCursor = `url("../icon/Cursor1.png"), auto`;

            componentArray[0] = new ComponentClass(0, 'Component0', new fabric.Canvas('canvas-component-' + 0, {
                    imageSmoothingEnabled: false,
                    fireMiddleClick: true,
                }),
                'canvas-component-0',
                'canvas-component-0-wrapper',
            );

            componentArray[0].canvas.setWidth(innerWidth - 400);
            componentArray[0].canvas.setHeight(innerHeight - 50);
            componentArray[0].defaultObject.box = new fabric.Rect({
                id: '///defaultBox///',
                left: 0,
                top: 0,
                strokeWidth: 0,
                fill: "#ffffff",
                stroke: "#ffffff",
                width: 1920,
                height: 1080,
            })
            componentArray[0].defaultObject.title = new fabric.Textbox('Component0', {
                id: '///defaultText///',
                fontFamily: 'Noto Sans KR',
                width: 200,
                top: -40,
                fontSize: 20,
                fontWeight: '400',
                fill: '#404040',
            })
            componentArray[0].defaultObject.box.selectable = false;
            componentArray[0].defaultObject.box.hasControls = false;
            componentArray[0].defaultObject.title.selectable = false;
            componentArray[0].defaultObject.title.hasControls = false;

            componentArray[0].canvas.add(componentArray[0].defaultObject.box);
            componentArray[0].canvas.add(componentArray[0].defaultObject.title);

            document.getElementById('canvas-component-0-wrapper').style.display = 'none';

            componentArray[0].canvas.defaultCursor = `url("../icon/Cursor1.png"), auto`;
            componentArray[0].canvas.hoverCursor = `url("../icon/Cursor1.png"), auto`;


            pageArray[0].canvas.on('mouse:down', function(event) {
                if (event.button == 1) {
                    if (ui.mouseType == 1) {
                        if (event.target==null) {
                            ui.objectType = 'page';
                        }
                        else if (event.target.id != '///defaultBox///' && event.target.id != '///defaultText///') {
                            ui.objectType = 'componentImage'
                            let activeObject = pageArray[0].canvas.getActiveObject();
                            currentObject.object = activeObject;
                            currentObject.x = activeObject.left;
                            currentObject.y = activeObject.top;
                            currentObject.id = activeObject.id;
                        }
                        else {
                            ui.objectType = 'page';
                        }

                        pageArray[ui.pageIndex].canvas.remove(this)
                    }
                }

            }); 

            // 키보드 이동 이벤트
            fabric.util.addListener(document.body, 'keydown', function(event) {
                
                var key = event.which || event.keyCode;

                if(event.shiftKey && key === 37)
                    moveSelected(1, 10);
                else if(event.shiftKey && key === 38)
                    moveSelected(2, 10);
                else if(event.shiftKey && key === 39)
                    moveSelected(3, 10);
                else if(event.shiftKey && key === 40)
                    moveSelected(4, 10);
                else if(key === 37)
                    moveSelected(1, 1);
                else if(key === 38)
                    moveSelected(2, 1);
                else if(key === 39)
                    moveSelected(3, 1);
                else if(key === 40)
                    moveSelected(4, 1);
                else if(key === 46)
                    deleteObject();
                
            });

            function moveSelected(direction, value) {

                let activeObject = componentArray[0].canvas.getActiveObject();

                if (activeObject) {
                    switch (direction) {
                        case 1:
                            activeObject.set('left', activeObject.left - value);
                            break;
                        case 2:
                            activeObject.set('top', activeObject.top - value);
                            break;
                        case 3:
                            activeObject.set('left', activeObject.left + value);
                            break;
                        case 4:
                            activeObject.set('top', activeObject.top + value);
                            break;
                    }
                    activeObject.setCoords();
                    componentArray[0].canvas.renderAll();
                    currentObject.x = activeObject.left;
                    currentObject.y = activeObject.top;
                }  else {
                    
                }

            }

            function deleteObject() {
                let activeObject = componentArray[0].canvas.getActiveObject();

                componentArray[0].object.forEach(function(element) {
                    if(element.object.id === activeObject.id) {
                        componentArray[0].object[element.index].object.id = '//**//';
                    }
                })

                componentArray[0].canvas.remove(activeObject);
                ui.currentComponentObjectArray = componentArray[0].object;
            }
            // 키보드 이동 이벤트



            componentArray[0].canvas.on('mouse:down', function(event) {
                
                if (event.button == 1) {
                
                    if (ui.mouseType == 1) {
                        if (event.target==null) {
                            ui.objectType = 'component';
                            currentComponent.width = componentArray[0].defaultObject.box.width;
                            currentComponent.height = componentArray[0].defaultObject.box.height;
                        }
                        else if (event.target.id != '///defaultBox///' && event.target.id != '///defaultText///') {
                            
                            let activeObject = componentArray[0].canvas.getActiveObject();

                            currentObject.id = activeObject.id;
                            currentObject.x = activeObject.left;
                            currentObject.y = activeObject.top;
                            currentObject.width = activeObject.width;
                            currentObject.height = activeObject.height;
                            currentObject.color = activeObject.fill.substr(1);  
                            currentObject.stroke = activeObject.stroke.substr(1);
                            currentObject.strokeWidth = activeObject.strokeWidth;

                            if (activeObject.type == 'rect') {
                                ui.objectType = 'rect';
                            }

                            else if (activeObject.type == 'ellipse') {
                                ui.objectType = 'ellipse';
                            }

                            else if (activeObject.type == 'textbox') {
                                ui.objectType = 'textBox';
                                currentObject.fontSize = activeObject.fontSize;
                            }

                            
                        }
                        else {
                            ui.objectType = 'component';
                            currentComponent.width = componentArray[0].defaultObject.box.width;
                            currentComponent.height = componentArray[0].defaultObject.box.height;
                        }
                    }
                    else if (ui.mouseType == 2) {

                        ui.isDown = true;
                        let pointer = componentArray[0].canvas.getPointer(event.e);
                        let objectIndex = componentArray[0].object.length;
                        ui.origX = parseInt(pointer.x);
                        ui.origY = parseInt(pointer.y);
                        
                        componentArray[0].object[objectIndex] = new ObjectClass(
                            new fabric.Rect({
                                id: 'Object' + objectIndex,
                                left: ui.origX,
                                top: ui.origY,
                                originX: 'left',
                                originY: 'top',
                                width: pointer.x-ui.origX,
                                height: pointer.y-ui.origY,
                                angle: 0,
                                fill: '#C4C4C4',
                                stroke: '#ffffff',
                                strokeWidth: 0,
                                transparentCorners: false,
                            }),
                            objectIndex,
                            0,
                            'rect',
                        )
                        componentArray[0].canvas.add(componentArray[0].object[objectIndex].object);
                        currentObject.object = componentArray[0].object[objectIndex].object;
                        currentObject.id = currentObject.object.id;
                        currentObject.rotate = currentObject.object.angle;
                        currentObject.color = currentObject.object.fill.substr(1);  
                        currentObject.stroke = currentObject.object.stroke.substr(1);
                        currentObject.strokeWidth = currentObject.object.strokeWidth;
                        currentObject.tagType = 'rect';
                        fabric.Object.prototype.selectable = true;
                        
                        ui.currentComponentObjectArray = componentArray[0].object;
                        ui.objectType = 'rect';
                    }

                    else if (ui.mouseType == 3) {

                        ui.isDown = true;
                        let pointer = componentArray[0].canvas.getPointer(event.e);
                        let objectIndex = componentArray[0].object.length;
                        ui.origX = parseInt(pointer.x);
                        ui.origY = parseInt(pointer.y);

                        componentArray[0].object[objectIndex] = new ObjectClass(
                            new fabric.Ellipse({
                                id: 'Object' + objectIndex,
                                left: ui.origX,
                                top: ui.origY,
                                originX: 'left',
                                originY: 'top',
                                rx: pointer.x-ui.origX,
                                ry: pointer.y-ui.origY,
                                angle: 0,
                                fill: '#C4C4C4',
                                stroke: '#ffffff',
                                strokeWidth: 0,
                                transparentCorners: false,
                            }),
                            objectIndex,
                            0,
                            'ellipse',
                        )
                        componentArray[0].canvas.add(componentArray[0].object[objectIndex].object);
                        currentObject.object = componentArray[0].object[objectIndex].object;
                        currentObject.id = currentObject.object.id;
                        currentObject.rotate = currentObject.object.angle;
                        currentObject.color = currentObject.object.fill.substr(1);  
                        currentObject.stroke = currentObject.object.stroke.substr(1);
                        currentObject.strokeWidth = currentObject.object.strokeWidth;
                        currentObject.tagType = 'ellipse';
                        fabric.Object.prototype.selectable = true;

                        ui.currentComponentObjectArray = componentArray[0].object;
                        ui.objectType = 'ellipse';
                    }

                    else if (ui.mouseType == 4) {
                        
                        var pointer = componentArray[0].canvas.getPointer(event.e);
                        var objectIndex = componentArray[0].object.length;
                        ui.origX = parseInt(pointer.x);
                        ui.origY = parseInt(pointer.y);
                        
                        componentArray[0].object[objectIndex] = new ObjectClass(
                            new fabric.Textbox('', {
                                id: 'Object' + objectIndex,
                                left: ui.origX,
                                top: ui.origY,
                                originX: 'left',
                                originY: 'top',
                                width: 20,
                                editable: true,
                                hasControls: false,
                                fontSize: 20,
                                fill: '#000000',
                                fontFamily: 'Noto Sans KR',
                            }),
                            objectIndex,
                            0,
                            'textBox',
                        )
                        componentArray[0].canvas.add(componentArray[0].object[objectIndex].object);
                        currentObject.object = componentArray[0].object[objectIndex].object;
                        currentObject.id = currentObject.object.id;
                        fabric.Object.prototype.selectable = true;

                        componentArray[0].canvas.setActiveObject(componentArray[0].object[objectIndex].object);
                        componentArray[0].object[objectIndex].object.enterEditing();
                        componentArray[0].canvas.renderAll();
                        ui.currentComponentObjectArray = componentArray[0].object;
                        ui.mouseType = 1;
                        ui.objectType = 'textBox';
                        currentObject.tagType = 'textBox';
                        currentObject.x = componentArray[0].object[objectIndex].object.left;
                        currentObject.y = componentArray[0].object[objectIndex].object.top;
                        currentObject.fontSize = componentArray[0].object[objectIndex].object.fontSize;
                        currentObject.color = currentObject.object.fill.substr(1);

                        componentArray.forEach((element) => {
                            element.canvas.defaultCursor = `url("../icon/Cursor1.png"), auto`;
                            element.canvas.hoverCursor = `url("../icon/Cursor1.png"), auto`;
                        });

                    }
                    else if (ui.mouseType == 5) {
                        componentArray[0].canvas.selection = false;
                        ui.moveViewBool = true;
                    }

                }
                if (event.button == 2) {
                    ui.mouseMiddleDown = true;
                }

            }); 

            componentArray[0].canvas.on('mouse:move', function(event){

                if (event.button == 1) {
                    if (ui.mouseType == 2) {

                        let objectIndex = componentArray[0].object.length - 1;
                        
                        if (!ui.isDown) return;
                        let pointer = componentArray[0].canvas.getPointer(event.e);
                        
                        if(ui.origX>pointer.x){
                            // @ts-ignore
                            componentArray[0].object[objectIndex].object.set({ left: parseInt(Math.abs(pointer.x)) });
                        }
                        if(ui.origY>pointer.y){
                            // @ts-ignore
                            componentArray[0].object[objectIndex].object.set({ top: parseInt(Math.abs(pointer.y)) });
                        }
                        
                        // @ts-ignore
                        componentArray[0].object[objectIndex].object.set({ width: parseInt(Math.abs(ui.origX - pointer.x)) });
                        
                        // @ts-ignore
                        componentArray[0].object[objectIndex].object.set({ height: parseInt(Math.abs(ui.origY - pointer.y)) });
                        componentArray[0].canvas.renderAll();
                        currentObject.x = currentObject.object.left;
                        currentObject.y = currentObject.object.top;
                        currentObject.width = currentObject.object.width;
                        currentObject.height = currentObject.object.height;
                        ui.isMove = true;
                        componentArray[0].canvas.selection = false;
                    }

                    else if (ui.mouseType == 3) {

                        var objectIndex = componentArray[0].object.length - 1;
                        
                        if (!ui.isDown) return;
                        var pointer = componentArray[0].canvas.getPointer(event.e);
                        
                        if(ui.origX>pointer.x){
                            // @ts-ignore
                            componentArray[0].object[objectIndex].object.set({ left: parseInt(Math.abs(pointer.x)) });
                        }
                        if(ui.origY>pointer.y){
                            // @ts-ignore
                            componentArray[0].object[objectIndex].object.set({ top: parseInt(Math.abs(pointer.y)) });
                        }
                        
                        // @ts-ignore
                        componentArray[0].object[objectIndex].object.set({ rx: parseInt(Math.abs(ui.origX - pointer.x)/2) });
                        // @ts-ignore
                        componentArray[0].object[objectIndex].object.set({ ry: parseInt(Math.abs(ui.origY - pointer.y)/2) });
                        componentArray[0].canvas.renderAll();
                        currentObject.x = currentObject.object.left;
                        currentObject.y = currentObject.object.top;
                        currentObject.width = currentObject.object.width;
                        currentObject.height = currentObject.object.height;
                        ui.isMove = true;
                        componentArray[0].canvas.selection = false;
                    }

                    else if (ui.mouseType == 5 && ui.moveViewBool) {
                        let delta = new fabric.Point(event.e.movementX, event.e.movementY);
                        componentArray[0].canvas.relativePan(delta);
                    }
                }

                if (ui.mouseMiddleDown) {
                    let delta = new fabric.Point(event.e.movementX, event.e.movementY);
                    componentArray[0].canvas.relativePan(delta);
                }
            });

            componentArray[0].canvas.on('mouse:up', function(event){
                
                if (event.button == 1) {
                    if (ui.mouseType == 1 && ui.isScale) {
                        ui.isScale = false;
                        let activeObject = componentArray[0].canvas.getActiveObject();
                        activeObject.scaleX = 1;
                        activeObject.scaleY = 1;
                        activeObject.width = currentObject.width;
                        activeObject.height = currentObject.height;
                        componentArray[0].canvas.renderAll();
                    }
                    else if (ui.mouseType == 2) {
                        
                        if(ui.isMove == false) {
                            currentObject.object.set('width', 100);
                            currentObject.object.set('height', 100);
                            currentObject.x = currentObject.object.left;
                            currentObject.y = currentObject.object.top;
                            currentObject.width = currentObject.object.width;
                            currentObject.height = currentObject.object.height;
                        }
                        componentArray[0].canvas.selection = true;
                        ui.isDown = false;
                        ui.isMove = false;
                        componentArray[0].canvas.renderAll();
                        ui.mouseType = 1; 
                        componentArray.forEach((element) => {
                            element.canvas.defaultCursor = `url("../icon/Cursor1.png"), auto`;
                            element.canvas.hoverCursor = `url("../icon/Cursor1.png"), auto`;
                        });
                        
                        ui.currentComponentObjectArray = componentArray[0].object;
                        currentObject.tagType = 'rect';

                        componentArray[0].canvas.getObjects().forEach(function(element) {
                            if(element.id === currentObject.id) {
                                componentArray[0].canvas.setActiveObject(element);
                                componentArray[0].canvas.renderAll();
                                currentObject.id = element.id;
                                currentObject.x = element.left;
                                currentObject.y = element.top;
                                currentObject.width = element.width;
                                currentObject.height = element.height;
                            }
                        });

                    }

                    else if (ui.mouseType == 3) {
                        
                        if(ui.isMove == false) {
                            currentObject.object.set('width', 100);
                            currentObject.object.set('height', 100);
                            currentObject.x = currentObject.object.left;
                            currentObject.y = currentObject.object.top;
                            currentObject.width = currentObject.object.width;
                            currentObject.height = currentObject.object.height;
                        }
                        componentArray[0].canvas.selection = true;
                        ui.isDown = false;
                        ui.isMove = false;
                        componentArray[0].canvas.renderAll();
                        ui.mouseType = 1; 
                        componentArray.forEach((element) => {
                            element.canvas.defaultCursor = `url("../icon/Cursor1.png"), auto`;
                            element.canvas.hoverCursor = `url("../icon/Cursor1.png"), auto`;
                        });

                        ui.currentComponentObjectArray = componentArray[0].object;
                        currentObject.tagType = 'ellipse';



                        componentArray[0].canvas.getObjects().forEach(function(element) {
                            if(element.id === currentObject.id) {
                                componentArray[0].canvas.setActiveObject(element);
                                componentArray[0].canvas.renderAll();
                                currentObject.id = element.id;
                                currentObject.x = element.left;
                                currentObject.y = element.top;
                                currentObject.width = element.width;
                                currentObject.height = element.height;
                            }
                        });
                        
                    }

                    else if (ui.mouseType == 5) {
                        componentArray[0].canvas.selection = true;
                        ui.moveViewBool = false;
                    }
                        
                }

                if (event.button == 2) {
                    ui.mouseMiddleDown = false;
                }
            });



            pageArray[0].canvas.on('object:moving', function(event){
                if (event.target.id != '////defaultBox///' && event.target.id != '///defaultText///') {
                    let activeObject = pageArray[0].canvas.getActiveObject();
                    ui.objectType = 'componentImage';


                    currentObject.object = activeObject;
                    currentObject.x = parseInt(currentObject.object.left);
                    currentObject.y = parseInt(currentObject.object.top);
                    activeObject.set({
                        top: currentObject.y,
                        left: currentObject.x,
                    });

                    let index = returnIndex2(pageArray[0].selectComponent, activeObject.id);
                    // @ts-ignore
                    pageArray[0].selectComponent[index].x = currentObject.x;
                    // @ts-ignore
                    pageArray[0].selectComponent[index].y = currentObject.y;

                }
            });


            componentArray[0].canvas.on('object:moving', function(event){
                if (event.target.id != '////defaultBox///' && event.target.id != '///defaultText///') {
                    let activeObject = componentArray[0].canvas.getActiveObject();
                    if (activeObject == 'rect')
                        ui.objectType = 'rect';
                    if (activeObject == 'ellipse')
                        ui.objectType = 'ellipse';
                    else if (activeObject == 'textbox')
                        ui.objectType = 'textBox';
                    
                    currentObject.object = activeObject;
                    currentObject.x = parseInt(currentObject.object.left);
                    currentObject.y = parseInt(currentObject.object.top);
                    activeObject.set({
                        top: currentObject.y,
                        left: currentObject.x,
                    });

                    let index = returnIndex(componentArray[0].object, activeObject.id);
                    // @ts-ignore
                    componentArray[0].object[index].object = activeObject;
                }
            });


            componentArray[0].canvas.on('object:scaling', function(event){
                if (event.target.id != '///defaultBox///' && event.target.id != '///defaultText///') {
                    ui.isScale = true;
                    let activeObject = componentArray[0].canvas.getActiveObject();
                    if (activeObject == 'rect')
                        ui.objectType = 'rect';
                    if (activeObject == 'ellipse')
                        ui.objectType = 'ellipse';
                    else if (activeObject == 'textbox')
                        ui.objectType = 'textBox';
                    currentObject.width = Math.floor(activeObject.getScaledWidth());
                    currentObject.height = Math.floor(activeObject.getScaledHeight());
                    componentArray[0].canvas.renderAll()
                }
            });
            
            function resetCurrentObject() {
                currentObject = new CurrentObjectClass();
            }
        }


        function setCanvasZoom() {
            
            let zoomLeft1 = (pageArray[0].canvas.width - pageArray[0].defaultObject.box.width * pageArray[0].zoom) / 2;
            let zoomTop1 = (pageArray[0].canvas.height - pageArray[0].defaultObject.box.height * pageArray[0].zoom) / 2;

            pageArray[0].canvas.setViewportTransform([
                pageArray[0].zoom,
                0,
                0,
                pageArray[0].zoom,
                zoomLeft1,
                zoomTop1,
            ]);

            //pageArray[0].zoomLeft = zoomLeft1;
            //pageArray[0].zoomTop = zoomTop1;

            let zoomLeft2 = (componentArray[0].canvas.width - componentArray[0].defaultObject.box.width * componentArray[0].zoom) / 2;
            let zoomTop2 = (componentArray[0].canvas.height - componentArray[0].defaultObject.box.height * componentArray[0].zoom) / 2;

            componentArray[0].canvas.setViewportTransform([
                componentArray[0].zoom,
                0,
                0,
                componentArray[0].zoom,
                zoomLeft2,
                zoomTop2,
            ]);

            //componentArray[0].zoomLeft = zoomLeft2;
            //componentArray[0].zoomTop = zoomTop2;


            pageArray[0].canvas.on('mouse:wheel', function(opt) {
                if (opt.e.ctrlKey) {
                    var delta = opt.e.deltaY;
                    var zoom = pageArray[0].canvas.getZoom();
                    zoom *= 0.999 ** delta;
                    if (zoom > 20) zoom = 20;
                    if (zoom < 0.01) zoom = 0.01;
                    pageArray[0].canvas.zoomToPoint({ x: opt.e.offsetX, y: opt.e.offsetY }, zoom);
                    opt.e.preventDefault();
                    opt.e.stopPropagation();
                    ui.currentZoom = roundTwo(pageArray[0].canvas.getZoom());
                    pageArray[0].canvas.renderAll();

                    pageArray[0].defaultObject.title.top = -2000 * 1/roundTwo(pageArray[0].canvas.getZoom());
                    pageArray[0].defaultObject.title.fontSize = 1000 * 1/roundTwo(pageArray[0].canvas.getZoom());
                }
            });

            componentArray[0].canvas.on('mouse:wheel', function(opt) {
                if (opt.e.ctrlKey) {
                    var delta = opt.e.deltaY;
                    var zoom = componentArray[0].canvas.getZoom();
                    zoom *= 0.999 ** delta;
                    if (zoom > 20) zoom = 20;
                    if (zoom < 0.01) zoom = 0.01;
                    componentArray[0].canvas.zoomToPoint({ x: opt.e.offsetX, y: opt.e.offsetY }, zoom);
                    opt.e.preventDefault();
                    opt.e.stopPropagation();
                    ui.currentZoom = roundTwo(componentArray[0].canvas.getZoom());

                    componentArray[0].defaultObject.title.top = -2000 * 1/roundTwo(componentArray[0].canvas.getZoom());
                    componentArray[0].defaultObject.title.fontSize = 1000 * 1/roundTwo(componentArray[0].canvas.getZoom());
                }
            });

            ui.currentZoom = roundTwo(pageArray[0].canvas.getZoom());;

            pageArray[0].canvas.defaultCursor = `url("../icon/Cursor1.png"), auto`;
            componentArray[0].canvas.defaultCursor = `url("../icon/Cursor1.png"), auto`;

        }

        setDefaultPage();
        setCanvasZoom();

        document.getElementById('layer-section-box').addEventListener("wheel", (event) => {
                if (event.ctrlKey) {
                    event.preventDefault(); 
                }
            }, true
        );

        document.getElementById('property-section-box').addEventListener("wheel", (event) => {
                if (event.ctrlKey) {
                    event.preventDefault(); 
                }
            }, true
        );

        document.getElementById('tool-section-box').addEventListener("wheel", (event) => {
                if (event.ctrlKey) {
                    event.preventDefault(); 
                }
            }, true
        );
    });


































    function setCanvasZoom() {
            
        let zoomLeft1 = (pageArray[ui.pageIndex].canvas.width - pageArray[ui.pageIndex].defaultObject.box.width * pageArray[ui.pageIndex].zoom) / 2;
        let zoomTop1 = (pageArray[ui.pageIndex].canvas.height - pageArray[ui.pageIndex].defaultObject.box.height * pageArray[ui.pageIndex].zoom) / 2;

        pageArray[ui.pageIndex].canvas.setViewportTransform([
            pageArray[ui.pageIndex].zoom,
            0,
            0,
            pageArray[ui.pageIndex].zoom,
            zoomLeft1,
            zoomTop1,
        ]);

        //pageArray[0].zoomLeft = zoomLeft1;
        //pageArray[0].zoomTop = zoomTop1;

        let zoomLeft2 = (componentArray[ui.componentIndex].canvas.width - componentArray[ui.componentIndex].defaultObject.box.width * componentArray[ui.componentIndex].zoom) / 2;
        let zoomTop2 = (componentArray[ui.componentIndex].canvas.height - componentArray[ui.componentIndex].defaultObject.box.height * componentArray[ui.componentIndex].zoom) / 2;

        componentArray[ui.componentIndex].canvas.setViewportTransform([
            componentArray[ui.componentIndex].zoom,
            0,
            0,
            componentArray[ui.componentIndex].zoom,
            zoomLeft2,
            zoomTop2,
        ]);

        //componentArray[0].zoomLeft = zoomLeft2;
        //componentArray[0].zoomTop = zoomTop2;


        pageArray[ui.pageIndex].canvas.on('mouse:wheel', function(opt) {
            if (opt.e.ctrlKey) {
                var delta = opt.e.deltaY;
                var zoom = pageArray[ui.pageIndex].canvas.getZoom();
                zoom *= 0.999 ** delta;
                if (zoom > 20) zoom = 20;
                if (zoom < 0.01) zoom = 0.01;
                pageArray[ui.pageIndex].canvas.zoomToPoint({ x: opt.e.offsetX, y: opt.e.offsetY }, zoom);
                opt.e.preventDefault();
                opt.e.stopPropagation();
                ui.currentZoom = roundTwo(pageArray[ui.pageIndex].canvas.getZoom());
                pageArray[ui.pageIndex].canvas.renderAll();
                pageArray[ui.pageIndex].defaultObject.title.top = -2000 * 1/roundTwo(pageArray[ui.pageIndex].canvas.getZoom());
                pageArray[ui.pageIndex].defaultObject.title.fontSize = 1000 * 1/roundTwo(pageArray[ui.pageIndex].canvas.getZoom());
            }
        });

        componentArray[ui.componentIndex].canvas.on('mouse:wheel', function(opt) {
            if (opt.e.ctrlKey) {
                var delta = opt.e.deltaY;
                var zoom = componentArray[ui.componentIndex].canvas.getZoom();
                zoom *= 0.999 ** delta;
                if (zoom > 20) zoom = 20;
                if (zoom < 0.01) zoom = 0.01;
                componentArray[ui.componentIndex].canvas.zoomToPoint({ x: opt.e.offsetX, y: opt.e.offsetY }, zoom);
                opt.e.preventDefault();
                opt.e.stopPropagation();
                ui.currentZoom = roundTwo(componentArray[ui.componentIndex].canvas.getZoom());
                componentArray[ui.componentIndex].defaultObject.title.top = -2000 * 1/roundTwo(componentArray[ui.componentIndex].canvas.getZoom());
                componentArray[ui.componentIndex].defaultObject.title.fontSize = 1000 * 1/roundTwo(componentArray[ui.componentIndex].canvas.getZoom());
            }
        });

        ui.currentZoom = 60;

        pageArray[ui.pageIndex].canvas.defaultCursor = `url("../icon/Cursor1.png"), auto`;
        componentArray[ui.componentIndex].canvas.defaultCursor = `url("../icon/Cursor1.png"), auto`;

    }



    // 페이지, 컴포넌트 추가
    function addPage(type) {

        if (type == 'page') {

            let length = pageArray.length;
            
            let canvasWrapper = document.createElement('div');
            canvasWrapper.id = 'canvas-page-' + length + '-wrapper';
            document.getElementById('canvas-section-box').appendChild(canvasWrapper);

            let canvas = document.createElement('canvas');
            canvas.id = 'canvas-page-' + length;
            document.getElementById('canvas-page-' + length + '-wrapper').appendChild(canvas);
            
            pageArray[length] = new PageClass(length, '/Page' + length, new fabric.Canvas('canvas-page-' + length, {
                    imageSmoothingEnabled: false,
                    fireMiddleClick: true,
                }),
                'canvas-page-' + length,
                'canvas-page-' + length + '-wrapper',
            );

            pageArray[length].canvas.setWidth(innerWidth - 400);
            pageArray[length].canvas.setHeight(innerHeight - 50);
            pageArray[length].defaultObject.box = new fabric.Rect({
                id: '/Page' + length,
                left: 0,
                top: 0,
                strokeWidth: 0,
                fill: "#ffffff",
                stroke: "#ffffff",
                width: 1920,
                height: 1080,
            })
            pageArray[length].defaultObject.title = new fabric.Textbox('/Page' + length, {
                fontFamily: 'Noto Sans KR',
                width: 200,
                top: -40,
                fontSize: 20,
                fontWeight: '400',
                fill: '#404040',
            })
            pageArray[length].defaultObject.box.selectable = true;
            pageArray[length].defaultObject.box.hasControls = true;
            pageArray[length].defaultObject.box.lockMovementX = true;
            pageArray[length].defaultObject.box.lockMovementY = true;
            pageArray[length].defaultObject.box.setControlsVisibility({
                mt: false,
                mb: true,
                ml: false,
                mr: false,
                bl: false,
                br: false,
                tr: false,
                tl: false,
                mtr: false,
            });
            
            pageArray[length].defaultObject.title.selectable = false;
            pageArray[length].defaultObject.title.hasControls = false;

            pageArray[length].canvas.add(pageArray[length].defaultObject.box);
            pageArray[length].canvas.add(pageArray[length].defaultObject.title);
        
        
            pageArray.forEach(element => {
                document.getElementById(element.canvasWrapperId).style.display = 'none';
            });
            componentArray.forEach(element => {
                document.getElementById(element.canvasWrapperId).style.display = 'none';
            });

            document.getElementById(pageArray[length].canvasWrapperId).style.display = 'block';
    
            let zoomLeft = (pageArray[length].canvas.width - pageArray[length].defaultObject.box.width * pageArray[length].zoom) / 2;
            let zoomTop = (pageArray[length].canvas.height - pageArray[length].defaultObject.box.height * pageArray[length].zoom) / 2;

            pageArray[length].canvas.setViewportTransform([
                pageArray[length].zoom,
                0,
                0,
                pageArray[length].zoom,
                zoomLeft,
                zoomTop,
            ]);

            pageArray[length].zoomLeft = zoomLeft;
            pageArray[length].zoomTop = zoomTop;

            pageArray[length].canvas.on('mouse:wheel', function(opt) {
                if (opt.e.ctrlKey) {
                    var delta = opt.e.deltaY;
                    var zoom = pageArray[length].canvas.getZoom();
                    zoom *= 0.999 ** delta;
                    if (zoom > 20) zoom = 20;
                    if (zoom < 0.01) zoom = 0.01;
                    pageArray[length].canvas.zoomToPoint({ x: opt.e.offsetX, y: opt.e.offsetY }, zoom);
                    opt.e.preventDefault();
                    opt.e.stopPropagation();
                    ui.currentZoom = roundTwo(pageArray[length].canvas.getZoom());

                    pageArray[length].object.title.top = -2000 * 1/roundTwo(pageArray[length].canvas.getZoom());
                    pageArray[length].object.title.fontSize = 1000 * 1/roundTwo(pageArray[length].canvas.getZoom());
                }
            });

            
            pageArray[length].canvas.on('object:moving', function(event){
                if (event.target.id != '////defaultBox///' && event.target.id != '///defaultText///') {
                    let activeObject = pageArray[length].canvas.getActiveObject();
                    ui.objectType = 'componentImage';


                    currentObject.object = activeObject;
                    currentObject.x = parseInt(currentObject.object.left);
                    currentObject.y = parseInt(currentObject.object.top);
                    activeObject.set({
                        top: currentObject.y,
                        left: currentObject.x,
                    });

                    let index = returnIndex2(pageArray[length].selectComponent, activeObject.id);
                    // @ts-ignore
                    pageArray[0].selectComponent[index].x = currentObject.x;
                    // @ts-ignore
                    pageArray[0].selectComponent[index].y = currentObject.y;

                    //let index = returnIndex(pageArray[0].object, activeObject.id);
                    //// @ts-ignore
                    //pageArray[0].selectComponent[index].object = activeObject;
                }
            });

            ui.currentZoom = roundTwo(pageArray[length].canvas.getZoom());

            pageArray[length].canvas.defaultCursor = `url("../icon/Cursor1.png"), auto`;
            pageArray[length].canvas.hoverCursor = `url("../icon/Cursor1.png"), auto`;

            ui.pageIndex = length;

            pageArray[length].canvas.on('mouse:down', function(event) {
                if (event.button == 1) {
                    if (ui.mouseType == 1) {
                        if (event.target==null) {
                            ui.objectType = 'page';
                        }
                        else if (event.target.id != '///defaultBox///' && event.target.id != '///defaultText///') {
                            ui.objectType = 'componentImage'
                            let activeObject = pageArray[length].canvas.getActiveObject();
                            currentObject.object = activeObject;
                            currentObject.x = activeObject.left;
                            currentObject.y = activeObject.top;
                        }
                        else {
                            ui.objectType = 'page';
                        }
                    }
                }

            });

            
        
            ui.currentSelect = null;
            ui.currentSelect = Array(0);

            pageArray[ui.pageIndex].selectComponent.forEach((element, index) => {
                ui.currentSelect[index] = element.componentId;
            });

            ui.currentPageMode = 'page';
            ui.objectType = 'page';

            CustomPropertyChild.refresh(ui.currentSelect)
            
        }

        else if (type == 'component') {

            componentArray.forEach(element => {
                element.canvas.discardActiveObject().renderAll();
            });

            let length = componentArray.length;
            
            let canvasWrapper = document.createElement('div');
            canvasWrapper.id = 'canvas-component-' + length + '-wrapper';
            document.getElementById('canvas-section-box').appendChild(canvasWrapper);

            let canvas = document.createElement('canvas');
            canvas.id = 'canvas-component-' + length;
            document.getElementById('canvas-component-' + length + '-wrapper').appendChild(canvas);
            
            componentArray[length] = new ComponentClass(length, 'Component' + length, new fabric.Canvas('canvas-component-' + length, {
                    imageSmoothingEnabled: false,
                    fireMiddleClick: true,
                }),
                'canvas-component-' + length,
                'canvas-component-' + length + '-wrapper',
            );

            componentArray[length].canvas.setWidth(innerWidth - 400);
            componentArray[length].canvas.setHeight(innerHeight - 50);
            componentArray[length].defaultObject.box = new fabric.Rect({
                id: '///defaultBox///',
                left: 0,
                top: 0,
                strokeWidth: 0,
                fill: "#ffffff",
                stroke: "#ffffff",
                width: 1920,
                height: 1080,
            })
            componentArray[length].defaultObject.title = new fabric.Textbox('Component' + length, {
                id: '///defaultText///',
                fontFamily: 'Noto Sans KR',
                width: 200,
                top: -40,
                fontSize: 20,
                fontWeight: '400',
                fill: '#404040',
            })
            componentArray[length].defaultObject.box.selectable = false;
            componentArray[length].defaultObject.box.hasControls = false;
            componentArray[length].defaultObject.title.selectable = false;
            componentArray[length].defaultObject.title.hasControls = false;

            componentArray[length].canvas.add(componentArray[length].defaultObject.box);
            componentArray[length].canvas.add(componentArray[length].defaultObject.title);
        
            pageArray.forEach(element => {
                document.getElementById(element.canvasWrapperId).style.display = 'none';
            });
            componentArray.forEach(element => {
                document.getElementById(element.canvasWrapperId).style.display = 'none';
            });

            document.getElementById(componentArray[length].canvasWrapperId).style.display = 'block';
    
            let zoomLeft = (componentArray[length].canvas.width - componentArray[length].defaultObject.box.width * componentArray[length].zoom) / 2;
            let zoomTop = (componentArray[length].canvas.height - componentArray[length].defaultObject.box.height * componentArray[length].zoom) / 2;

            componentArray[length].canvas.setViewportTransform([
                componentArray[length].zoom,
                0,
                0,
                componentArray[length].zoom,
                zoomLeft,
                zoomTop,
            ]);

            componentArray[length].zoomLeft = zoomLeft;
            componentArray[length].zoomTop = zoomTop;

            componentArray[length].canvas.on('mouse:wheel', function(opt) {
                if (opt.e.ctrlKey) {
                    var delta = opt.e.deltaY;
                    var zoom = componentArray[length].canvas.getZoom();
                    zoom *= 0.999 ** delta;
                    if (zoom > 20) zoom = 20;
                    if (zoom < 0.01) zoom = 0.01;
                    componentArray[length].canvas.zoomToPoint({ x: opt.e.offsetX, y: opt.e.offsetY }, zoom);
                    opt.e.preventDefault();
                    opt.e.stopPropagation();
                    ui.currentZoom = roundTwo(componentArray[length].canvas.getZoom());
                    componentArray[length].defaultObject.title.top = -2000 * 1/roundTwo(componentArray[length].canvas.getZoom());
                    componentArray[length].defaultObject.title.fontSize = 1000 * 1/roundTwo(componentArray[length].canvas.getZoom());
                }
            });

            ui.currentZoom = roundTwo(componentArray[length].canvas.getZoom());

            componentArray[length].canvas.defaultCursor = `url("../icon/Cursor1.png"), auto`;
            componentArray[length].canvas.hoverCursor = `url("../icon/Cursor1.png"), auto`;

            ui.componentIndex = length;
            ui.currentPageMode = 'component';


            // 키보드 이벤트
            fabric.util.addListener(document.body, 'keydown', function(event) {
                
                var key = event.which || event.keyCode;

                if(event.shiftKey && key === 37)
                    moveSelected(1, 10);
                else if(event.shiftKey && key === 38)
                    moveSelected(2, 10);
                else if(event.shiftKey && key === 39)
                    moveSelected(3, 10);
                else if(event.shiftKey && key === 40)
                    moveSelected(4, 10);
                else if(key === 37)
                    moveSelected(1, 1);
                else if(key === 38)
                    moveSelected(2, 1);
                else if(key === 39)
                    moveSelected(3, 1);
                else if(key === 40)
                    moveSelected(4, 1);
                else if(key === 65)
                    deleteObject();
            });

            function moveSelected(direction, value) {

                let activeObject = componentArray[length].canvas.getActiveObject();

                if (activeObject) {
                    switch (direction) {
                        case 1:
                            activeObject.set('left', activeObject.left - value);
                            break;
                        case 2:
                            activeObject.set('top', activeObject.top - value);
                            break;
                        case 3:
                            activeObject.set('left', activeObject.left + value);
                            break;
                        case 4:
                            activeObject.set('top', activeObject.top + value);
                            break;
                    }
                    activeObject.setCoords();
                    componentArray[length].canvas.renderAll();
                    currentObject.x = activeObject.left;
                    currentObject.y = activeObject.top;
                }  else {
                    
                }

            }

            function deleteObject() {
                let activeObject = componentArray[length].canvas.getActiveObject();

                componentArray[length].object.forEach(function(element) {
                    if(element.object.id === activeObject.id) {
                        componentArray[length].object[element.index].object.id = '//**//';
                    }
                })

                componentArray[length].canvas.remove(activeObject);
                ui.currentComponentObjectArray = componentArray[length].object;
            }
            
            componentArray[length].canvas.on('mouse:down', function(event) {
                
                if (event.button == 1) {
                
                    if (ui.mouseType == 1) {
                        if (event.target==null) {
                            ui.objectType = 'component';
                            currentComponent.width = componentArray[0].defaultObject.box.width;
                            currentComponent.height = componentArray[0].defaultObject.box.height;
                        }
                        else if (event.target.id != '///defaultBox///' && event.target.id != '///defaultText///') {

                            let activeObject = componentArray[length].canvas.getActiveObject();

                            currentObject.id = activeObject.id;
                            currentObject.x = activeObject.left;
                            currentObject.y = activeObject.top;
                            currentObject.width = activeObject.width;
                            currentObject.height = activeObject.height;
                            currentObject.color = activeObject.fill.substr(1);  
                            currentObject.stroke = activeObject.stroke.substr(1);
                            currentObject.strokeWidth = activeObject.strokeWidth;

                            if (activeObject.type == 'rect') {
                                ui.objectType = 'rect';
                            }
                            else if (activeObject.type == 'ellipse') {
                                ui.objectType = 'ellipse';
                            }  
                            else if (activeObject.type == 'textbox') {
                                ui.objectType = 'textBox';
                                currentObject.fontSize = activeObject.fontSize;
                            }
                        }
                        else {
                            ui.objectType = 'component';
                            currentComponent.width = componentArray[length].defaultObject.box.width;
                            currentComponent.height = componentArray[length].defaultObject.box.height;
                        }
                    }
                    else if (ui.mouseType == 2) {

                        ui.isDown = true;
                        var pointer = componentArray[length].canvas.getPointer(event.e);
                        ui.origX = parseInt(pointer.x);
                        ui.origY = parseInt(pointer.y);
                        var objectIndex = componentArray[length].object.length;

                        componentArray[length].object[objectIndex] = new ObjectClass(
                            new fabric.Rect({
                                id: 'Object' + objectIndex,
                                left: ui.origX,
                                top: ui.origY,
                                originX: 'left',
                                originY: 'top',
                                width: pointer.x-ui.origX,
                                height: pointer.y-ui.origY,
                                angle: 0,
                                fill: '#C4C4C4',
                                stroke: '#ffffff',
                                strokeWidth: 0,
                                transparentCorners: false,
                            }),
                            objectIndex,
                            0,
                            'rect',
                        )
                        componentArray[length].canvas.add(componentArray[length].object[objectIndex].object);
                        currentObject.object = componentArray[length].object[objectIndex].object;
                        currentObject.id = currentObject.object.id;
                        currentObject.rotate = currentObject.object.angle;
                        //currentObjectRound = currentObject.rx;
                        currentObject.color = currentObject.object.fill.substr(1);  
                        currentObject.stroke = currentObject.object.stroke.substr(1);
                        currentObject.strokeWidth = currentObject.object.strokeWidth;
                        fabric.Object.prototype.selectable = true;

                        ui.currentComponentObjectArray = componentArray[length].object;
                        ui.objectType = 'rect';
                    }

                    else if (ui.mouseType == 3) {

                        ui.isDown = true;
                        var pointer = componentArray[length].canvas.getPointer(event.e);
                        var objectIndex = componentArray[length].object.length;
                        ui.origX = parseInt(pointer.x);
                        ui.origY = parseInt(pointer.y);

                        componentArray[length].object[objectIndex] = new ObjectClass(
                            new fabric.Ellipse({
                                id: 'Object' + objectIndex,
                                left: ui.origX,
                                top: ui.origY,
                                originX: 'left',
                                originY: 'top',
                                rx: pointer.x-ui.origX,
                                ry: pointer.y-ui.origY,
                                angle: 0,
                                fill: '#C4C4C4',
                                stroke: '#ffffff',
                                strokeWidth: 0,
                                transparentCorners: false,

                            }),
                            objectIndex,
                            0,
                            'ellipse',
                        )
                        componentArray[length].canvas.add(componentArray[length].object[objectIndex].object);
                        currentObject.object = componentArray[length].object[objectIndex].object;
                        currentObject.id = currentObject.object.id;
                        currentObject.rotate = currentObject.object.angle;
                        //currentObjectRound = currentObject.rx;
                        currentObject.color = currentObject.object.fill.substr(1);  
                        currentObject.stroke = currentObject.object.stroke.substr(1);
                        currentObject.strokeWidth = currentObject.object.strokeWidth;
                        fabric.Object.prototype.selectable = true;

                        ui.currentComponentObjectArray = componentArray[length].object;
                        ui.objectType = 'ellipse';
                    }

                    else if (ui.mouseType == 4) {
                        
                        var pointer = componentArray[length].canvas.getPointer(event.e);
                        var objectIndex = componentArray[length].object.length;
                        ui.origX = parseInt(pointer.x);
                        ui.origY = parseInt(pointer.y);
                        
                        componentArray[length].object[objectIndex] = new ObjectClass(
                            new fabric.Textbox('', {
                                id: 'Object' + objectIndex,
                                left: ui.origX,
                                top: ui.origY,
                                originX: 'left',
                                originY: 'top',
                                width: 20,
                                editable: true,
                                hasControls: false,
                                fontSize: 20,
                                fontFamily: 'Noto Sans KR',
                            }),
                            objectIndex,
                            0,
                            'textBox',
                        )
                        componentArray[length].canvas.add(componentArray[length].object[objectIndex].object);
                        //currentObject = componentArray[length].canvasPageObject[objectIndex].object;
                        //currentObjectID = currentObject.id;
                        fabric.Object.prototype.selectable = true;
                        componentArray[length].canvas.setActiveObject(componentArray[length].object[objectIndex].object);
                        componentArray[length].object[objectIndex].object.enterEditing();
                        componentArray[length].canvas.renderAll();
                        ui.currentComponentObjectArray = componentArray[length].object;
                        ui.mouseType = 1;
                        ui.objectType = 'textBox';
                        currentObject.x = componentArray[length].object[objectIndex].object.left;
                        currentObject.y = componentArray[length].object[objectIndex].object.top;
                        currentObject.fontSize = componentArray[length].object[objectIndex].object.fontSize;
                        currentObject.color = currentObject.object.fill.substr(1);

                        componentArray.forEach((element) => {
                            element.canvas.defaultCursor = `url("../icon/Cursor1.png"), auto`;
                            element.canvas.hoverCursor = `url("../icon/Cursor1.png"), auto`;
                        });

                    }
                    else if (ui.mouseType == 5) {
                        componentArray[length].canvas.selection = false;
                        ui.moveViewBool = true;
                    }

                }
                if (event.button == 2) {
                    ui.mouseMiddleDown = true;
                }

            }); 

            componentArray[length].canvas.on('mouse:move', function(event){

                if (event.button == 1) {
                    if (ui.mouseType == 2) {

                        var objectIndex = componentArray[length].object.length - 1;
                        
                        if (!ui.isDown) return;
                        var pointer = componentArray[length].canvas.getPointer(event.e);
                        
                        if(ui.origX>pointer.x){
                            // @ts-ignore
                            componentArray[length].object[objectIndex].object.set({ left: parseInt(Math.abs(pointer.x)) });
                        }
                        if(ui.origY>pointer.y){
                            // @ts-ignore
                            componentArray[length].object[objectIndex].object.set({ top: parseInt(Math.abs(pointer.y)) });
                        }
                        // @ts-ignore
                        componentArray[length].object[objectIndex].object.set({ width: parseInt(Math.abs(Math.floor(ui.origX - pointer.x))) });
                        // @ts-ignore
                        componentArray[length].object[objectIndex].object.set({ height: parseInt(Math.abs(Math.floor(ui.origY - pointer.y))) });
                        componentArray[length].canvas.renderAll();
                        currentObject.x = currentObject.object.left;
                        currentObject.y = currentObject.object.top;
                        currentObject.width = currentObject.object.width;
                        currentObject.height = currentObject.object.height;
                        ui.isMove = true;
                        componentArray[length].canvas.selection = false;
                    }

                    else if (ui.mouseType == 3) {

                        var objectIndex = componentArray[length].object.length - 1;
                        
                        if (!ui.isDown) return;
                        var pointer = componentArray[length].canvas.getPointer(event.e);
                        
                        if(ui.origX>pointer.x){
                            // @ts-ignore
                            componentArray[length].object[objectIndex].object.set({ left: parseInt(Math.abs(pointer.x)) });
                        }
                        if(ui.origY>pointer.y){
                            // @ts-ignore
                            componentArray[length].object[objectIndex].object.set({ top: parseInt(Math.abs(pointer.y)) });
                        }
                        
                        // @ts-ignore
                        componentArray[length].object[objectIndex].object.set({ rx: parseInt(Math.abs(ui.origX - pointer.x)/2) });
                        // @ts-ignore
                        componentArray[length].object[objectIndex].object.set({ ry: parseInt(Math.abs(ui.origY - pointer.y)/2) });
                        componentArray[length].canvas.renderAll();
                        currentObject.x = currentObject.object.left;
                        currentObject.y = currentObject.object.top;
                        currentObject.width = currentObject.object.width;
                        currentObject.height = currentObject.object.height;
                        ui.isMove = true;
                        componentArray[length].canvas.selection = false;
                    }

                    else if (ui.mouseType == 5 && ui.moveViewBool) {
                        let delta = new fabric.Point(event.e.movementX, event.e.movementY);
                        componentArray[length].canvas.relativePan(delta);
                    }
                }

                if (ui.mouseMiddleDown) {
                    let delta = new fabric.Point(event.e.movementX, event.e.movementY);
                    componentArray[length].canvas.relativePan(delta);
                }
            });

            componentArray[length].canvas.on('mouse:up', function(event){

                if (event.button == 1) {

                    if (ui.mouseType == 1 && ui.isScale) {
                        ui.isScale = false;
                        let activeObject = componentArray[length].canvas.getActiveObject();
                        activeObject.scaleX = 1;
                        activeObject.scaleY = 1;
                        activeObject.width = currentObject.width;
                        activeObject.height = currentObject.height;
                        componentArray[length].canvas.renderAll();
                    }
                
                    if (ui.mouseType == 2) {
                        
                        if(ui.isMove == false) {
                            currentObject.object.set('width', 100);
                            currentObject.object.set('height', 100);
                            currentObject.x = currentObject.object.left;
                            currentObject.y = currentObject.object.top;
                            currentObject.width = currentObject.object.width;
                            currentObject.height = currentObject.object.height;
                        }
                        componentArray[length].canvas.selection = true;
                        ui.isDown = false;
                        ui.isMove = false;
                        componentArray[length].canvas.renderAll();
                        ui.mouseType = 1; 
                        componentArray.forEach((element) => {
                            element.canvas.defaultCursor = `url("../icon/Cursor1.png"), auto`;
                            element.canvas.hoverCursor = `url("../icon/Cursor1.png"), auto`;
                        });

                        ui.currentComponentObjectArray = componentArray[length].object;
                        currentObject.tagType = 'rect';

                        componentArray[length].canvas.getObjects().forEach(function(element) {
                            if(element.id === currentObject.id) {
                                componentArray[length].canvas.setActiveObject(element);
                                componentArray[length].canvas.renderAll();
                                currentObject.id = element.id;
                                currentObject.x = element.left;
                                currentObject.y = element.top;
                                currentObject.width = element.width;
                                currentObject.height = element.height;
                            }
                        });
                    }

                    if (ui.mouseType == 3) {
                        
                        if(ui.isMove == false) {
                            currentObject.object.set('width', 100);
                            currentObject.object.set('height', 100);
                            currentObject.x = currentObject.object.left;
                            currentObject.y = currentObject.object.top;
                            currentObject.width = currentObject.object.width;
                            currentObject.height = currentObject.object.height;
                        }
                        componentArray[length].canvas.selection = true;
                        ui.isDown = false;
                        ui.isMove = false;
                        componentArray[length].canvas.renderAll();
                        ui.mouseType = 1; 
                        componentArray.forEach((element) => {
                            element.canvas.defaultCursor = `url("../icon/Cursor1.png"), auto`;
                            element.canvas.hoverCursor = `url("../icon/Cursor1.png"), auto`;
                        });

                        ui.currentComponentObjectArray = componentArray[length].object;
                        currentObject.tagType = 'ellipse';

                        componentArray[length].canvas.getObjects().forEach(function(element) {
                            if(element.id === currentObject.id) {
                                componentArray[length].canvas.setActiveObject(element);
                                componentArray[length].canvas.renderAll();
                                currentObject.id = element.id;
                                currentObject.x = element.left;
                                currentObject.y = element.top;
                                currentObject.width = element.width;
                                currentObject.height = element.height;
                            }
                        });
                        
                    }

                    else if (ui.mouseType == 5) {
                        componentArray[length].canvas.selection = true;
                        ui.moveViewBool = false;
                    }
                        
                }

                if (event.button == 2) {
                    ui.mouseMiddleDown = false;
                }
            });




            componentArray[length].canvas.on('object:moving', function(event){
                if (event.target.id != '///defaultBox///' && event.target.id != '///defaultTitle///') {
                    let activeObject = componentArray[length].canvas.getActiveObject();
                    if (activeObject == 'rect')
                        ui.objectType = 'rect';
                    if (activeObject == 'ellipse')
                        ui.objectType = 'ellipse';
                    else if (activeObject == 'textbox')
                        ui.objectType = 'textBox';
                    
                    currentObject.object = activeObject;
                    currentObject.x = parseInt(currentObject.object.left);
                    currentObject.y = parseInt(currentObject.object.top);
                    currentObject.object.set({
                        top: currentObject.y,
                        left: currentObject.x,
                    });                    
                }
            });

            componentArray[length].canvas.on('object:scaling', function(event){
                if (event.target.id != '///defaultBox///' && event.target.id != '///defaultTitle///') {
                    ui.isScale = true;
                    let activeObject = componentArray[length].canvas.getActiveObject();
                    if (activeObject == 'rect')
                        ui.objectType = 'rect';
                    if (activeObject == 'ellipse')
                        ui.objectType = 'ellipse';
                    else if (activeObject == 'textbox')
                        ui.objectType = 'textBox';
                    currentObject.width = Math.floor(activeObject.getScaledWidth());
                    currentObject.height = Math.floor(activeObject.getScaledHeight());
                    componentArray[length].canvas.renderAll()
                }
            });
        }
    }

    function resetCurrentObject() {
        currentObject = new CurrentObjectClass();
    }

    function roundTwo(num) {
        let m = Number((Math.abs(num) * 100).toPrecision(15));
        let returnNum = Math.round(m) / 100 * Math.sign(num) * 100;
        // @ts-ignore
        returnNum = returnNum.toFixed(0);
        return returnNum;
    }

    function returnIndex(array, id) {
        let index;
        array.forEach((element, i) => {
            if(element.object.id == id)
                index = element.index;
        });
        return index;
    }

    function returnIndex2(array, id) {
        let index;
        array.forEach((element, i) => { 
            
            if(element.componentId == id)
                index = i;
        });
        return index;
    }

    function loadImage(width, height, index) {
        let tempCanvas = componentArray[index].canvas;
        tempCanvas.discardActiveObject();
        var sel = new fabric.ActiveSelection(tempCanvas.getObjects(), {
            canvas: tempCanvas,
        });
        tempCanvas.setActiveObject(sel);
        tempCanvas.viewportTransform = [1, 0, 0, 1, 0, 0];
        tempCanvas.requestRenderAll();



        
        try {

            let activeGroup = tempCanvas.getActiveObject()

            let url = activeGroup.canvas.toDataURL({
                format: 'png', 
                width: width,
                height: height,
            })

            componentArray.forEach(element => {
                let zoomLeft2 = (element.canvas.width - element.defaultObject.box.width * element.zoom) / 2;
                let zoomTop2 = (element.canvas.height - element.defaultObject.box.height * element.zoom) / 2;
                element.canvas.setViewportTransform([
                    element.zoom,
                    0,
                    0,
                    element.zoom,
                    zoomLeft2,
                    zoomTop2,
                ]);
            });

            componentArray[index].componentProperty.imageSrc = url;
            componentArray[index].componentProperty.width = width;
            componentArray[index].componentProperty.height = height;

        } catch (error) {
        }
    }


    function deleteComponentImage(deleteComponentId, componentIndex, list) {

        pageArray[ui.pageIndex].selectComponent.forEach(element => {
            if (element.componentId == deleteComponentId) {
                returnObj(deleteComponentId);
                let activeObject = pageArray[ui.pageIndex].canvas.getActiveObject();
                pageArray[ui.pageIndex].canvas.remove(activeObject);
                pageArray[ui.pageIndex].canvas.renderAll();

                element.componentId = '//deleted//';
                //pageArray[ui.pageIndex].selectComponent = null;
                //pageArray[ui.pageIndex].selectComponent = Array(0);
                ui.currentSelect = null;
                ui.currentSelect = Array(0);
            }
                
        });

        function returnObj(name) {
            pageArray[ui.pageIndex].canvas.getObjects().forEach(function(element) {
                if(element.id == name) {
                    pageArray[ui.pageIndex].canvas.setActiveObject(element);          
                    pageArray[ui.pageIndex].canvas.renderAll();    
                }
            })
        }


    }

    function createComponentImage(data) {

        

        var pugImg = new Image();
        pugImg.onload = function (img) {    
            // @ts-ignore
            var img = new fabric.Image(pugImg, {
                id: data.componentId,
                width: data.width,
                height: data.height,
            });
            // @ts-ignore
            img.setControlsVisibility({
                mt: false,
                mb: false,
                ml: false,
                mr: false,
                bl: false,
                br: false,
                tr: false,
                tl: false,
                mtr: false,
            });
            pageArray[ui.pageIndex].canvas.add(img);
            
        };






        pugImg.src = data.componentImage;

        ui.currentSelect = null;
        ui.currentSelect = Array(0);


        let i = pageArray[ui.pageIndex].selectComponent.length;
    
        pageArray[ui.pageIndex].selectComponent[i] = data;
                
        pageArray[ui.pageIndex].selectComponent[i].object = componentArray[data.componentIndex]

        pageArray[ui.pageIndex].selectComponent.forEach((element, index) => {
            ui.currentSelect[index] = element.componentId;
        });

    }




    function setZoomDefault() {
        
        if (ui.currentPageMode == 'page') {

            let zoomLeft = (pageArray[ui.pageIndex].canvas.width - pageArray[ui.pageIndex].defaultObject.box.width * 0.6) / 2;
            let zoomTop = (pageArray[ui.pageIndex].canvas.height - pageArray[ui.pageIndex].defaultObject.box.height * 0.6) / 2;

            pageArray[ui.pageIndex].canvas.setViewportTransform([
                0.6,
                0,
                0,
                0.6,
                zoomLeft,
                zoomTop,
            ]);

            pageArray[ui.pageIndex].defaultObject.box.top = -2000 * 1/roundTwo(pageArray[ui.pageIndex].canvas.getZoom());
            pageArray[ui.pageIndex].defaultObject.box.fontSize = 1000 * 1/roundTwo(pageArray[ui.pageIndex].canvas.getZoom());

            ui.currentZoom = 60;
        }

        else if (ui.currentPageMode == 'component') {

            let zoomLeft = (componentArray[ui.componentIndex].canvas.width - componentArray[ui.componentIndex].defaultObject.box.width * 0.6) / 2;
            let zoomTop = (componentArray[ui.componentIndex].canvas.height - componentArray[ui.componentIndex].defaultObject.box.height * 0.6) / 2;

            
            componentArray[ui.componentIndex].canvas.setViewportTransform([
                0.6,
                0,
                0,
                0.6,
                zoomLeft,
                zoomTop,
            ]);

            componentArray[ui.componentIndex].defaultObject.box.top = -2000 * 1/roundTwo(componentArray[ui.componentIndex].canvas.getZoom());
            componentArray[ui.componentIndex].defaultObject.box.fontSize = 1000 * 1/roundTwo(componentArray[ui.componentIndex].canvas.getZoom());

            ui.currentZoom = 60;
        }
    }









    //코드 생성 부분

    const zip = new JSZip();
    var componentFileContents = [];
    var pageFileContents = [];
    var addedFileContents = [];


    function createComponentFile() {
        var i = 0;

        componentArray.forEach(comp => {
            componentFileContents[i] = '';

            if(comp.path != '//deleted//') {
                
                comp.object.forEach(obj => {

                    if(obj.object.id != '//**//'){
                        componentFileContents[i] += '<' + 'div ';
                        componentFileContents[i] += 'id="'+obj.object.id+'">';
                        if(obj.tagType == 'textBox') componentFileContents[i] += obj.object.text;
                        componentFileContents[i] += '<' + '/div' + '>\n';
                    }

                });

                componentFileContents[i] += '\n<' + 'style>\n';

                comp.object.forEach(obj => {

                    if(obj.object.id != '//**//'){
                        componentFileContents[i] += "\t#" + obj.object.id + " {\n";

                        //넓이, 높이, 위치 좌표
                        componentFileContents[i] += '\t\tposition: absolute;\n';
                        if(obj.tagType == 'text'){
                            componentFileContents[i] += '\t\twidth: '+Math.ceil(obj.object.width+10)+'px;\n';
                        } else {
                            componentFileContents[i] += '\t\twidth: '+Math.ceil(obj.object.width)+'px;\n';
                        }
                        componentFileContents[i] += '\t\theight: '+Math.ceil(obj.object.height)+'px;\n';
                        componentFileContents[i] += '\t\tleft: '+Math.ceil(comp.defaultObject.box.width/2 + obj.object.left)+'px;\n';
                        componentFileContents[i] += '\t\ttop: '+Math.ceil(comp.defaultObject.box.height/2 - 20 + obj.object.top)+'px;\n';

                        //배경색
                        if(obj.tagType == 'rect'){
                            componentFileContents[i] += '\t\tbackground-color: '+obj.object.fill+';\n';
                        }

                        //테두리
                        if(obj.object.stroke != null && obj.object.strokeWidth != null){
                            componentFileContents[i] += '\t\tborder: '+obj.object.strokeWidth+'px solid '+obj.object.stroke.toUpperCase()+';\n';
                        }
                        
                        //폰트
                        if(obj.tagType == 'textBox'){
                            componentFileContents[i] += '\t\tfont-size: '+obj.object.fontSize+'px;\n';
                        }

                        componentFileContents[i] += '\t}\n';
                    }

                });

                componentFileContents[i] += '<'+'/style>\n';
                i+=1;
            }
        });
    }


    function createPageFile() {
        var i=0;

        pageArray.forEach(page =>{
            pageFileContents[i] = '';

            if(page.path != '//deleted//'){

                pageFileContents[i] += '<'+'script>\n';
                page.selectComponent.forEach(sel => {
                    if(sel.componentId != '//deleted//'){
                        pageFileContents[i] += "\timport "+sel.componentId+" from './component/"+sel.componentId+".svelte'\n";
                    }
                });
                pageFileContents[i] += '</'+'script>\n';

                pageFileContents[i] += '<'+'main>\n';
                page.selectComponent.forEach(sel => {
                    if(sel.componentId != '//deleted//'){
                        pageFileContents[i] += '\t<'+'div id="'+sel.componentId+'">\n';
                        pageFileContents[i] += '\t\t<'+sel.componentId+'></'+sel.componentId+'>\n';
                        pageFileContents[i] += '\t</'+'div>\n';
                    }
                });
                pageFileContents[i] += '</'+'main>\n';

                pageFileContents[i] += '<'+'style>\n';
                page.selectComponent.forEach(sel => {
                    if(sel.componentId != '//deleted//'){
                        pageFileContents[i] += "\t#" + sel.componentId + " {\n";
                        pageFileContents[i] += '\t\tposition: relative;\n';
                        pageFileContents[i] += '\t\ttop: '+sel.y+'px;\n';
                        pageFileContents[i] += '\t\tleft: '+sel.x+'px;\n';
                        pageFileContents[i] += '\t}\n';
                    }
                });
                pageFileContents[i] += '</'+'style>\n';

                i+=1;
            }
        });
    }

    function createAddedFile() {
        var i = 1;


        //main.js
        addedFileContents[0] = 'import App from "./App.svelte";\n';
        addedFileContents[0] += 'var app = new App({target: document.body});\n';
        addedFileContents[0] += 'export default app;';


        //App.svelte
        addedFileContents[1] = '<'+'script>\n';
        addedFileContents[1] += "\timport Root from './Root.svelte'\n";
        addedFileContents[1] += '</'+'script>\n';
        addedFileContents[1] += '<'+'main>\n';
        addedFileContents[1] += '\t<'+'div id="root">\n';
        addedFileContents[1] += '\t\t<'+'Root>'+'</'+'Root'+'>\n';
        addedFileContents[1] += '\t</'+'div>\n';
        addedFileContents[1] += '</'+'main>\n';


        //global.css
        addedFileContents[2] = 'html, body {\n\tposition: relative;\n\twidth: 100%;\n\theight: 100%;\n}\n';
        addedFileContents[2] += 'body {\n\tcolor: #333;\n\tmargin: 0;\n\tpadding: 8px;\n\tbox-sizing: border-box;\n\t';
        addedFileContents[2] += 'font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Oxygen-Sans, Ubuntu, Cantarell, "Helvetica Neue", sans-serif;\n}\n';
        addedFileContents[2] += 'a {\n\tcolor: rgb(0,100,200);\n\ttext-decoration: none;\n}\n';
        addedFileContents[2] += 'a:hover {\n\ttext-decoration: underline;\n}\n';
        addedFileContents[2] += 'a:visited {\n\tcolor: rgb(0,80,160);\n}\n';
        addedFileContents[2] += 'label {\n\tdisplay: block;\n}\n';
        addedFileContents[2] += 'input, button, select, textarea {\n\tfont-family: inherit;\n\tfont-size: inherit;\n\t-webkit-padding: 0.4em 0;\n\tpadding: 0.4em;\n\t';
        addedFileContents[2] += 'margin: 0 0 0.5em 0;\n\tbox-sizing: border-box;\n\tborder: 1px solid #ccc;\n\tborder-radius: 2px;\n}\n';
        addedFileContents[2] += 'input:disabled {\n\tcolor: #ccc;\n}\n';
        addedFileContents[2] += 'button {\n\tcolor: #333;\n\tbackground-color: #f4f4f4;\n\toutline: none;\n}\n';
        addedFileContents[2] += 'button:disabled {\n\tcolor: #999;\n}\n';
        addedFileContents[2] += 'button:not(:disabled):active {\n\tbackground-color: #ddd;\n}\n';
        addedFileContents[2] += 'button:focus {\n\tborder-color: #666;\n}\n';


        //index.html
        addedFileContents[3] = '<!DOCTYPE html>\n';
        addedFileContents[3] += '<html lang="en">\n';
        addedFileContents[3] += '<'+'head>\n';
        addedFileContents[3] += "\t<meta charset='utf-8'>\n";
        addedFileContents[3] += "\t<meta name='viewport' content='width=device-width,initial-scale=1'>\n";
        addedFileContents[3] += "\t<title>Svelte app</title>\n";
        addedFileContents[3] += "\t<link rel='stylesheet' href='/global.css'>\n";
        addedFileContents[3] += "\t<link rel='stylesheet' href='/build/bundle.css'>\n";
        addedFileContents[3] += "\t<"+"script defer src='/build/bundle.js'><"+"/script>\n";
        addedFileContents[3] += '</head>\n<body>\n</body>\n</html>\n';


        //.gitignore
        addedFileContents[4] = '/node_modules/\n/public/build/\n\n.DS_Store\n';


        //package.json
        addedFileContents[5] = '{\n  "name": "svelte-app",\n  "version": "1.0.0",\n  "private": true,\n  "scripts": {\n';
        addedFileContents[5] += '    "build": "rollup -c",\n    "dev": "rollup -c -w",\n    "start": "sirv public --no-clear"    \n  },';
        addedFileContents[5] += '\n  "devDependencies": {\n    "@rollup/plugin-commonjs": "^17.0.0",\n    "@rollup/plugin-node-resolve": "^11.0.0",';
        addedFileContents[5] += '\n    "rollup": "^2.3.4",\n    "rollup-plugin-css-only": "^3.1.0",\n    "rollup-plugin-livereload": "^2.0.0",';
        addedFileContents[5] += '\n    "rollup-plugin-svelte": "^7.0.0",\n    "rollup-plugin-terser": "^7.0.0",\n    "svelte": "^3.0.0"\n  },';
        addedFileContents[5] += '\n  "dependencies": {\n    "sirv-cli": "^2.0.0"\n  }\n}\n';


        //rollup.config.js
        addedFileContents[6] = "import svelte from 'rollup-plugin-svelte';\n";
        addedFileContents[6] += "import commonjs from '@rollup/plugin-commonjs';\n";
        addedFileContents[6] += "import resolve from '@rollup/plugin-node-resolve';\n";
        addedFileContents[6] += "import livereload from 'rollup-plugin-livereload';\n";
        addedFileContents[6] += "import { terser } from 'rollup-plugin-terser';\n";
        addedFileContents[6] += "import css from 'rollup-plugin-css-only';\n";
        addedFileContents[6] += "\nconst production = !process.env.ROLLUP_WATCH;\n\n";
        addedFileContents[6] += "function serve() {\n"
        addedFileContents[6] += "\tlet server;\n\tfunction toExit() {\n\t\tif (server) server.kill(0);\n\t}\n";
        addedFileContents[6] += "\treturn {\n\t\twriteBundle() {\n\t\t\tif (server) return;\n\t\t\t";
        addedFileContents[6] += "server = require('child_process').spawn('npm', ['run', 'start', '--', '--dev'], {\n\t\t\t\t";
        addedFileContents[6] += "stdio: ['ignore', 'inherit', 'inherit'],\n\t\t\t\tshell: true\n\t\t\t});\n\t\t\t";
        addedFileContents[6] += "process.on('SIGTERM', toExit);\n\t\t\tprocess.on('exit', toExit);\n\t\t}\n\t};\n}\n\n";
        addedFileContents[6] += "export default {\n\tinput: 'src/main.js',\n\toutput: {\n\t\t"
        addedFileContents[6] += "sourcemap: true,\n\t\tformat: 'iife',\n\t\tname: 'app',\n\t\tfile: 'public/build/bundle.js'\n\t},";
        addedFileContents[6] += "\n\tplugins: [\n\t\tsvelte({\n\t\t\tcompilerOptions: {\n\t\t\t\tdev: !production\n\t\t\t}\n\t\t}),";
        addedFileContents[6] += "\n\t\tcss({ output: 'bundle.css' }),\n\t\tresolve({\n\t\t\t";
        addedFileContents[6] += "browser: true,\n\t\t\tdedupe: ['svelte']\n\t\t}),\n\t\tcommonjs(),";
        addedFileContents[6] += "\n\t\t!production && serve(),\n\t\t!production && livereload('public'),\n\t\tproduction && terser()";
        addedFileContents[6] += "\n\t],\n\twatch: {\n\t\tclearScreen: false\n\t}\n};\n";
    }

    function makeZip() {
        var i = 0;

        var mainJavascriptFile = new Blob([addedFileContents[0]], {type:'text/plain'});
        zip.folder("src").file("main.js", mainJavascriptFile);

        var appSvelteFile = new Blob([addedFileContents[1]], {type:'text/plain'});
        zip.folder("src").file("App.svelte", appSvelteFile);

        var globalCssFile = new Blob([addedFileContents[2]], {type:'text/plain'});
        zip.folder("public").file("global.css", globalCssFile);

        var indexHtmlFile = new Blob([addedFileContents[3]], {type:'text/plain'});
        zip.folder("public").file("index.html", indexHtmlFile);

        var gitignoreFile = new Blob([addedFileContents[4]], {type:'text/plain'});
        zip.file(".gitignore", gitignoreFile);

        var packageJsonFile = new Blob([addedFileContents[5]], {type:'text/plain'});
        zip.file("package.json", packageJsonFile);

        var rollupFile = new Blob([addedFileContents[6]], {type:'text/plain'});
        zip.file("rollup.config.js", rollupFile);

        componentArray.forEach(comp => {
            if(comp.path != "//deleted//") {
                var componentSvelteFile = new Blob([componentFileContents[i]], {type:'text/plain'});
                zip.folder("src").folder("component").file(comp.path+".svelte", componentSvelteFile);
                i+=1;
            }
        });

        i = 0;
        pageArray.forEach(page => {
            if(page.path != "//deleted//") {
                var pageSvelteFile = new Blob([pageFileContents[i]], {type:'text/plain'});
                if(page.path == '/') {
                    zip.folder("src").file("Root.svelte", pageSvelteFile);
                } else {
                    zip.folder("src").file(page.path.substr(1)+".svelte", pageSvelteFile);
                }
                i+=1;
            }
        });

        zip.generateAsync({
            type: "blob",
            compression: "DEFLATE"
        })
        .then((myZip) => {
            const url = URL.createObjectURL(myZip);
            const a =  document.createElement('a');
            a.download = 'test.zip';
            a.href = url;
            a.click();
        });
    }

</script>



<svelte:window bind:innerWidth bind:innerHeight />


<main>
    <!-- 툴 -->
    <div id="tool-section-box"  class="flex-row flex-space-between" style:width={innerWidth+'px'} style:height={$toolSectionHeight+'px'}>

        <CustomTool1 mouseType={ui.mouseType} currentPageMode={ui.currentPageMode} theme={theme} on:message={(event)=>{
            ui.mouseType = event.detail.data;
        }}/>


        <CustomTool2 zoom={ui.currentZoom} 
        on:setZoomDefault={()=>{
            setZoomDefault();
        }}
        on:showPreview={()=>{
            createComponentFile();
            createPageFile();
            createAddedFile();
            makeZip();
            console.log(pageArray)
        }}
        ></CustomTool2>

    </div>
    <!-- 툴 -->

    <div class="flex-row flex-space-between" style:height={innerHeight-$windowHeight-$toolSectionHeight+'px'}>
        <!-- 레이어 -->
        <div id="layer-section-box">
            <CustomLayer pageOrComponent={ui.pageOrComponent} {pageArray} {componentArray} pageIndex={ui.pageIndex} 
            componentIndex={ui.componentIndex} currentPageMode={ui.currentPageMode} 
            currentComponentObjectArray={ui.currentComponentObjectArray} 
            on:selectPageOrComponent={(event)=>{
                ui.pageOrComponent = event.detail.pageOrComponent;
            }}            
            on:createPage={(event)=>{
                if (event.detail.pageOrComponent == 'page') {
                    ui.pageOrComponent = event.detail.pageOrComponent;
                    
                    addPage('page');
                    

                }
                else if (event.detail.pageOrComponent == 'component') {
                    ui.pageOrComponent = event.detail.pageOrComponent;
                    ui.currentComponentObjectArray = new Array(0);
                    ui.componentIndex = event.detail.componentIndex;
                    ui.objectType = 'component';
                    addPage('component');
                    currentComponent.width = componentArray[ui.componentIndex].defaultObject.box.width;
                    currentComponent.height = componentArray[ui.componentIndex].defaultObject.box.height;
                }
                
            }}
            on:changePageComponent={(event)=>{
                if (event.detail.pageOrComponent == 'page') {
                    
                    ui.pageOrComponent = event.detail.pageOrComponent;
                    ui.pageIndex = event.detail.pageIndex;
                    ui.currentPageMode = 'page';


                    pageArray.forEach(element => {
                        document.getElementById(element.canvasWrapperId).style.display = 'none';
                    });
                    componentArray.forEach(element => {
                        document.getElementById(element.canvasWrapperId).style.display = 'none';
                    });


                    componentArray.forEach((element, index) => {
                        loadImage(element.defaultObject.box.width, element.defaultObject.box.height, index);
                    });
                    
                    
                    ui.currentZoom = roundTwo(pageArray[ui.pageIndex].canvas.getZoom());

                    currentObject.id = pageArray[ui.pageIndex].path;

                    document.getElementById('canvas-page-' + ui.pageIndex + '-wrapper').style.display = 'block';
                    
                    ui.objectType = 'page';

                    
                    ui.currentSelect = null;
                    ui.currentSelect = Array(0);

                    pageArray[ui.pageIndex].selectComponent.forEach((element, index) => {
                        ui.currentSelect[index] = element.componentId;
                    });

                    CustomPropertyChild.refresh(ui.currentSelect)

                }
                else if (event.detail.pageOrComponent == 'component') {
                    ui.pageOrComponent = event.detail.pageOrComponent;
                    ui.componentIndex = event.detail.componentIndex;
                    ui.mouseType = event.detail.mouseType;
                    ui.currentPageMode = 'component';

                    componentArray.forEach(element => {
                        element.canvas.discardActiveObject();
                        element.canvas.renderAll();
                    });

                    pageArray.forEach(element => {
                        document.getElementById(element.canvasWrapperId).style.display = 'none';
                    });
                    componentArray.forEach(element => {
                        document.getElementById(element.canvasWrapperId).style.display = 'none';
                    });
                
                    ui.currentZoom = roundTwo(componentArray[ui.componentIndex].canvas.getZoom());
                    
                    currentObject.id = componentArray[ui.componentIndex].path;

                    document.getElementById('canvas-component-' + ui.componentIndex + '-wrapper').style.display = 'block';

                    //currentComponentId = 'canvas-component-' + ui.componentIndex;
                
                    ui.objectType = 'component';
                    currentComponent.width = componentArray[ui.componentIndex].defaultObject.box.width;
                    currentComponent.height = componentArray[ui.componentIndex].defaultObject.box.height;

                    ui.currentComponentObjectArray = componentArray[ui.componentIndex].object;
                
                    pageArray.forEach((element) => {
                        element.canvas.defaultCursor = `url("../icon/Cursor1.png"), auto`;
                        element.canvas.hoverCursor = `url("../icon/Cursor1.png"), auto`;
                    });
                    componentArray.forEach((element) => {
                        element.canvas.defaultCursor = `url("../icon/Cursor1.png"), auto`;
                        element.canvas.hoverCursor = `url("../icon/Cursor1.png"), auto`;
                    });
                }
            }}
            on:pageButtonBlur={(event)=>{
                if (event.detail.pageOrComponent == 'page') {
                    pageArray = event.detail.pageArray;
                }
                else if (event.detail.pageOrComponent == 'component') {
                    componentArray = event.detail.componentArray;
                }
            }}
            on:pageButtonEnter={(event)=>{
                if (event.detail.pageOrComponent == 'page') {
                    pageArray = event.detail.pageArray;
                }
                else if (event.detail.pageOrComponent == 'component') {
                    componentArray = event.detail.componentArray;
                }
            }}
            on:deletePage={(event)=>{
                if (event.detail.pageOrComponent == 'page') {
                    pageArray = event.detail.pageArray;
                    ui.pageIndex = event.detail.pageIndex;
                }
                else if (event.detail.pageOrComponent == 'component') {
                    componentArray = event.detail.componentArray;
                    ui.componentIndex = event.detail.componentIndex;
                }
            }}
            on:clickActiveObject={(event)=>{

                
                componentArray[ui.componentIndex].canvas.getObjects().forEach(function(element) {
                    
                    if(element.id === event.detail.objId) {
                        
                        componentArray[ui.componentIndex].canvas.setActiveObject(element);
                        componentArray[ui.componentIndex].canvas.renderAll();
                        

                        if (element.type == 'rect') {
                            ui.objectType = 'object';
                        }

                        if (element.type == 'ellipse') {
                            ui.objectType = 'object';
                        }
                        
                        else if (element.type == 'textbox'){
                            ui.objectType = 'textBox';
                            currentObject.fontSize = element.fontSize;
                        }
                        currentObject.id = element.id;
                        currentObject.x = element.left;
                        currentObject.y = element.top;
                        currentObject.width = element.width;
                        currentObject.height = element.height;
                    }
                })
                

            }}
            />
        </div>
        <!-- 레이어 -->

        <!-- 캔버스 -->
        <div id="canvas-section-box" style:width={innerWidth-400+'px'} style:height={innerHeight-$windowHeight-$toolSectionHeight+'px'}>
            <div id="canvas-page-0-wrapper">
                <canvas id="canvas-page-0"></canvas>
            </div>
            <div id="canvas-component-0-wrapper">
                <canvas id="canvas-component-0"></canvas>
            </div>
        </div>
        <!-- 캔버스 -->

        <!-- 속성 -->
        <div id="property-section-box">
            <CustomProperty bind:this={CustomPropertyChild} type={ui.objectType} currentObject={currentObject} currentComponent={currentComponent}
            {componentArray}  currentSelect={ui.currentSelect}

            on:selectComponent2={(event)=>{
                createComponentImage(event.detail.data);
            }}

            on:deselectComponent2={(event)=>{
                // @ts-ignore
                deleteComponentImage(event.detail.data, event.detail.componentIndex, event.detail.list)
            }}

            on:editComponentImageProperty={(event)=>{
                let activeObject = pageArray[ui.pageIndex].canvas.getActiveObject();
                if (event.detail.x != undefined && event.detail.y != undefined) {
                    currentObject.x = event.detail.x;
                    currentObject.x = event.detail.x;
                    activeObject.set('left', currentObject.x)
                    activeObject.set('top', currentObject.y)
                    pageArray[ui.pageIndex].canvas.renderAll()
                }
            }}

            on:editObjectProperty={(event)=>{

                let activeObject = componentArray[ui.componentIndex].canvas.getActiveObject();

                if (event.detail.id != undefined) {
                    currentObject.id = event.detail.id;
                    activeObject.set('id', currentObject.id)
                }
                if (event.detail.x != undefined && event.detail.y != undefined) {
                    currentObject.x = event.detail.x;
                    currentObject.x = event.detail.x;
                    activeObject.set('left', currentObject.x)
                    activeObject.set('top', currentObject.y)
                }
                if (event.detail.width != undefined && event.detail.height != undefined) {
                    currentObject.width = event.detail.width;
                    currentObject.height = event.detail.height;
                    activeObject.set('width', currentObject.width)
                    activeObject.set('height', currentObject.height)
                }
                if (event.detail.color != undefined) {
                    currentObject.color = event.detail.color;
                    activeObject.set('fill', '#' + currentObject.color)
                }
                if (event.detail.stroke != undefined && event.detail.strokeWidth != undefined) {
                    currentObject.stroke = event.detail.stroke;
                    currentObject.strokeWidth = event.detail.strokeWidth;
                    activeObject.set('stroke', '#' +  currentObject.stroke)
                    activeObject.set('strokeWidth', currentObject.strokeWidth)
                }
                if (event.detail.fontSize != undefined) {
                    currentObject.fontSize = event.detail.fontSize;
                    activeObject.set('fontSize', currentObject.fontSize)
                }

                componentArray[ui.componentIndex].canvas.renderAll()
                
            }}

            on:editComponentProperty={(event)=>{
                if (event.detail.width != undefined) {
                    currentComponent.width = event.detail.width*1;
                    componentArray[ui.componentIndex].defaultObject.box.set('width', currentComponent.width)
                }

                if (event.detail.height != undefined) {
                    currentComponent.height = event.detail.height*1;
                    componentArray[ui.componentIndex].defaultObject.box.set('height', currentComponent.height)
                }

                componentArray[ui.componentIndex].canvas.renderAll()
                
            }}
            
            ></CustomProperty>
        </div>
        <!-- 속성 -->
    </div>
    
    
</main>



<style>

    .flex-row {
        display: flex;
        flex-direction: row;
    }

    .flex-column {
        display: flex;
        flex-direction: column;
    }

    .flex-center {
        justify-content: center;
    }

    .flex-space-between {
        justify-content: space-between;
    }

    #canvas-section-box {
        background-color: #c4c4c4;
    }

    #layer-section-box {
        width: 200px;
    }

    #property-section-box {
        width: 200px;
    }



</style>

