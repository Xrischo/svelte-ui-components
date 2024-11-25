<script lang="ts">
    import ColorPicker, { ChromeVariant } from "svelte-awesome-color-picker"
    import CustomWrapper from "./ColourPicker/CustomWrapper.svelte"
    import CustomColourTextInput from "./ColourPicker/CustomColourTextInput.svelte"
    import { fly } from "svelte/transition"
    import { onMount } from "svelte"


    interface Props {
        hex?: string;
        label?: string;
        placeholder?: string;
        height?: string;
        widthpx?: string;
        background?: string;
        border?: string;
        error?: string;
        onclick?: (e: Event) => void
    }

    let {
        hex = $bindable(''),
        label = '',
        placeholder = 'Choose a colour',
        height = 'h-12',
        widthpx = '',
        background = 'bg-surface-100 text-black dark:bg-surface-700 dark:text-white',
        border = 'border border-gray-400 dark:border-gray-500',
        error = '',
        onclick = () => {},
    }: Props = $props();

    let colorPicker: boolean = $state(false)
    let divElem: HTMLDivElement = $state() as HTMLDivElement
    let inputElem: HTMLDivElement = $state() as HTMLDivElement

    // Check if overflowing below and put on top. If it also overflows on the top, keep it below
    $effect(() => {
        if (divElem && divElem.getBoundingClientRect().bottom + 10 > window.innerHeight && divElem.getBoundingClientRect().top - divElem.offsetHeight - inputElem.offsetHeight > 0) {
            divElem.style.top = `${-divElem.offsetHeight + 20}px`
        }
    });

    const handleClick = (event: any) => {
        if (!event.target.closest(`.colourfield`)) {
            colorPicker = false
        }
    }

    // add event listener on mount and remove it on destroy
    onMount(() => {
        document.addEventListener('click', handleClick)
        return () => document.removeEventListener('click', handleClick)
    })
</script>

<div class="relative">
    {#if label }
        <label for="colourfield" class="block mb-2 text-sm text-black dark:text-white">{label}</label>
    {/if}

    <!-- svelte-ignore a11y_click_events_have_key_events -->
    <!-- svelte-ignore a11y_no_static_element_interactions -->
    <div bind:this={inputElem} class="colourfield flex flex-row w-full {background} {border} rounded-lg {height} items-center hover:cursor-pointer"
        onclick={() => colorPicker = !colorPicker}
    >
        <div class='px-2'>
            <div class='rounded-full w-8 h-8' style='background-color: {hex || '#ffffff'}'></div>
        </div>
        <div class="w-full">
            <span class="text-sm text-gray-500 dark:text-gray-400">
                {hex ? hex : placeholder}
            </span>
        </div>
    </div>

    {#if error}
        <span class='absolute text-error-500 mb-2'>{error}</span>
    {/if}

    <!-- svelte-ignore a11y_click_events_have_key_events -->
    <!-- svelte-ignore a11y_no_static_element_interactions -->
    {#if colorPicker}
        <div bind:this={divElem} onclick={(e) => {
            e.stopPropagation();
            onclick(e)
        }} 
        class="inline-block absolute mt-1" style='z-index: 100;'
        in:fly={{ y: -10, duration: 200 }} out:fly={{ y: -10, duration: 200 }}>

            <ColorPicker 
                isDialog={false} 
                bind:hex 
                on:close={() => colorPicker = false} 
                components={{...ChromeVariant, wrapper: CustomWrapper, textInput: CustomColourTextInput }} 
                sliderDirection='horizontal' 
                --picker-background='black'
                --picker-width={widthpx || (inputElem?.clientWidth ? inputElem?.clientWidth - 8 + 'px' : '')}
            />
        </div>
    {/if}
</div>