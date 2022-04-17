<script>
    
    import { createEventDispatcher } from 'svelte';

    import CustomTextField from './CustomTextField.svelte';
    import CustomColorPicker from './CustomColorPicker.svelte';
    import CustromHr from './CustomHr.svelte'
    import CustomSelectComponent from './CustomSelectComponent.svelte'
    

    export let type;
    export let currentObject = null;
    export let currentComponent = null;
    export let componentArray = null;
    export let currentSelect;

    let CustomSelectComponentChild;


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

    class CurrentComponentClass {
        constructor() {
            this.width = null;
            this.height = null;
        }
    }

    if (!currentObject) {
        currentObject = new CurrentObjectClass();
    }
    else if (!currentComponent) {
        currentComponent = new CurrentComponentClass();
    }

	const dispatch = createEventDispatcher();

    export function refresh(data) {
        CustomSelectComponentChild.refresh(data);
    }

</script>


<CustromHr/>

<div class="blank"></div>


{#if type == 'rect'}
    <CustomTextField type={1} title={["ID"]} data={[currentObject.id]} dataType={'string'} on:message={(event)=>{
        currentObject.id = event.detail.data[0]
        dispatch('editObjectProperty', {
            id: currentObject.id,
        });
    }}></CustomTextField>

    <div class="blank"></div>

    <CustomTextField type={2} title={["X", "Y"]} data={[currentObject.x, currentObject.y]} dataType={'int'} on:message={(event)=>{
        currentObject.x = event.detail.data[0] * 1;
        currentObject.y = event.detail.data[1] * 1;
        dispatch('editObjectProperty', {
            x: parseInt(currentObject.x),
            y: parseInt(currentObject.y),
        });
    }}></CustomTextField>

    <div class="blank"></div>

    <CustomTextField type={2} title={["W", "H"]} data={[currentObject.width, currentObject.height]} dataType={'int'} on:message={(event)=>{
        currentObject.width = event.detail.data[0] * 1;
        currentObject.height = event.detail.data[1] * 1;
        dispatch('editObjectProperty', {
            width: currentObject.width,
            height: currentObject.height,
        });
    }}></CustomTextField>

    <div class="blank"></div>

    <CustomColorPicker title={"Color"} data={currentObject.color} on:message={(event)=>{
        currentObject.color = event.detail.data
        dispatch('editObjectProperty', {
            color: currentObject.color,
        });
    }}></CustomColorPicker>

    <div class="blank"></div>

    <CustomColorPicker title={"Stroke"} data={currentObject.stroke} data2={currentObject.strokeWidth} on:message={(event)=>{
        currentObject.stroke = event.detail.data.toUpperCase();
        currentObject.strokeWidth = event.detail.data2 * 1;
        dispatch('editObjectProperty', {
            stroke: currentObject.stroke,
            strokeWidth: currentObject.strokeWidth,
        });
    }}></CustomColorPicker>
    
{:else if type == 'ellipse'}
    <CustomTextField type={1} title={["ID"]} data={[currentObject.id]} dataType={'string'} on:message={(event)=>{
        currentObject.id = event.detail.data[0]
        dispatch('editObjectProperty', {
            id: currentObject.id,
        });
    }}></CustomTextField>

    <div class="blank"></div>

    <CustomTextField type={2} title={["X", "Y"]} data={[currentObject.x, currentObject.y]} dataType={'int'} on:message={(event)=>{
        currentObject.x = event.detail.data[0] * 1;
        currentObject.y = event.detail.data[1] * 1;
        dispatch('editObjectProperty', {
            x: currentObject.x,
            y: currentObject.y,
        });
    }}></CustomTextField>

    <div class="blank"></div>

    <CustomTextField type={2} title={["W", "H"]} data={[currentObject.width, currentObject.height]} dataType={'int'} on:message={(event)=>{
        currentObject.width = event.detail.data[0] * 1;
        currentObject.height = event.detail.data[1] * 1;
        dispatch('editObjectProperty', {
            width: currentObject.width,
            height: currentObject.height,
        });
    }}></CustomTextField>

    <div class="blank"></div>

    <CustomColorPicker title={"Color"} data={currentObject.color} on:message={(event)=>{
        currentObject.color = event.detail.data
        dispatch('editObjectProperty', {
            color: currentObject.color,
        });
    }}></CustomColorPicker>

    <div class="blank"></div>

    <CustomColorPicker title={"Stroke"} data={currentObject.stroke} data2={currentObject.strokeWidth} on:message={(event)=>{
        currentObject.stroke = event.detail.data.toUpperCase();
        currentObject.strokeWidth = event.detail.data2 * 1;
        dispatch('editObjectProperty', {
            stroke: currentObject.stroke,
            strokeWidth: currentObject.strokeWidth,
        });
    }}></CustomColorPicker>

{:else if type == 'textBox'}
    <CustomTextField type={1} title={["ID"]} data={[currentObject.id]} dataType={'string'} on:message={(event)=>{
        currentObject.id = event.detail.data[0]
        dispatch('editObjectProperty', {
            id: currentObject.id,
        });
    }}></CustomTextField>

    <div class="blank"></div>

    <CustomTextField type={2} title={["X", "Y"]} data={[currentObject.x, currentObject.y]} dataType={'int'} on:message={(event)=>{
        currentObject.x = event.detail.data[0] * 1;
        currentObject.y = event.detail.data[1] * 1;
        dispatch('editObjectProperty', {
            x: currentObject.x,
            y: currentObject.y,
        });
    }}></CustomTextField>

    <div class="blank"></div>

    <CustomTextField type={1} title={["Fs"]} data={[currentObject.fontSize]} dataType={'int'} on:message={(event)=>{
        currentObject.fontSize = event.detail.data[0] * 1;
        dispatch('editObjectProperty', {
            fontSize: currentObject.fontSize,
        });
    }}></CustomTextField>

    <div class="blank"></div>

    <CustomColorPicker title={"Color"} data={currentObject.color} on:message={(event)=>{
        currentObject.color = event.detail.data;
        dispatch('editObjectProperty', {
            color: currentObject.color,
        });
    }}></CustomColorPicker>

{:else if type == 'component'}

    <CustomTextField type={1} title={["Width"]} data={[currentComponent.width]} size={[148, 50]} dataType={'int'} on:message={(event)=>{
        currentComponent.width = event.detail.data[0] * 1;
        dispatch('editComponentProperty', {
            width: currentComponent.width,
        });
    }}></CustomTextField>

    <div class="blank"></div>

    <CustomTextField type={1} title={["Height"]} data={[currentComponent.height]} size={[148, 50]} dataType={'int'} on:message={(event)=>{
        currentComponent.height = event.detail.data[0] * 1;
        dispatch('editComponentProperty', {
            height: currentComponent.height,
        });
    }}></CustomTextField>

{:else if type == 'componentImage'}
    {currentObject.id}
    <CustomTextField type={2} title={["X", "Y"]} data={[currentObject.x, currentObject.y]} dataType={'int'} on:message={(event)=>{
        currentObject.x = event.detail.data[0] * 1;
        currentObject.y = event.detail.data[1] * 1;
        dispatch('editComponentImageProperty', {
            x: currentObject.x,
            y: currentObject.y,
        });
    }}></CustomTextField>

{:else if type == 'page'}

    <CustomSelectComponent bind:this={CustomSelectComponentChild} {componentArray} {currentSelect} on:selectComponent1={(event)=>{
        //console.log(event.detail.data)
        dispatch('selectComponent2', {
            data: event.detail.data,
        });
    }}
    on:deselectComponent1={(event)=>{
        //console.log(event.detail.data)
        dispatch('deselectComponent2', {
            data: event.detail.data,
            componentIndex: event.detail.componentIndex,
            list: event.detail.list,
        });
    }}/>
{/if}

<style>
    .blank {
        width: 200px;
        height: 8px;
    }
</style>

