<script>
    
    import { createEventDispatcher } from 'svelte';

    import CustomTextField from './CustomTextField.svelte';
    import CustomDropDown from './CustomDropDown.svelte';

    export let currentObject;

    let textField;
    let textFieldData;

	const dispatch = createEventDispatcher();

    let heightVar = 26;

    let eventProperty;

    function resetEvent() {
        currentObject.Event.when = null;
        currentObject.Event.do = null;
        currentObject.Event.detail = null;
        dispatch('message', {
            data: currentObject,
        });
        heightVar = 26;
    }

    

    
</script>



<main>
    
    <div class="blank"></div>
    <div id="event-property" bind:this={eventProperty}>
        <div id="event-block" style="height:{heightVar}px">
            <CustomDropDown title={'when'} items={['Click', 'Mouse Over']} borderStyle={false} on:message={(Event)=>{
                currentObject.Event.when = Event.detail.data;
                dispatch('message', {
                    data: currentObject,
                });
                heightVar = 52;
            }}></CustomDropDown>

            {#if currentObject.Event.when}
                <CustomDropDown title={'do'} items={['Move Page', 'Alert', 'Style']} borderStyle={false} on:message={(Event)=>{
                    currentObject.Event.do = Event.detail.data;
                    dispatch('message', {
                        data: currentObject,
                    });
                    heightVar = 83;
                }}></CustomDropDown>
            {/if}


            {#if currentObject.Event.do == 'Move Page'}
                <CustomDropDown title={'to'} items={['Page1', 'Page2']} borderStyle={false} on:message={(Event)=>{
                    currentObject.Event.detail = Event.detail.data;
                    dispatch('message', {
                        data: currentObject,
                    });
                }}></CustomDropDown>

            {:else if currentObject.Event.do == 'Alert'}
                <CustomTextField type={1} title={[""]} data={[textFieldData]} dataType={'string'} borderStyle={false} on:message={(event)=>{
                    currentObject.Event.detail = event.detail.data[0];
                    dispatch('message', {
                        data: currentObject,
                    });
                }}></CustomTextField>

            {:else if currentObject.Event.do == 'Style'}
                <CustomTextField type={1} title={[""]} data={[textFieldData]} dataType={'string'} borderStyle={false} on:message={(event)=>{
                    currentObject.Event.detail = event.detail.data[0];
                    dispatch('message', {
                        data: currentObject,
                    });
                }}></CustomTextField>
            {/if}


            {#if currentObject.Event.detail}
                <button on:click={resetEvent}></button>
            {/if}
        </div>
    </div>
    
</main>



<style>
    main {
        display: flex;
        flex-direction: column;
        justify-content: top;
    }
    #event-property {
        display: flex;
        flex-direction: row;
        justify-content: center;
    }

    #event-block {
        border: 1px solid white;
        width: 148px;
        border-radius: 2px;
    }

    #event-block:hover {
        border: 1px solid #e2e2e2;
    }

    #event-block:focus-within {
        border: 1px solid #18A0FB;
    }

    .blank {
        width: 200px;
        height: 8px;
    }







    
    
</style>

