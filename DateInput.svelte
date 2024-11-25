<script lang="ts">
    import dayjs from "dayjs"
    import { calendarSVG } from ""
    import DatePicker from "./DatePicker.svelte"
    import { onMount } from 'svelte'
    import { fly } from "svelte/transition"
    import MultiDatePicker from "./MultiDatePicker.svelte"


  interface Props {
    value?: string | Date | null;
    valueAsDate?: Date | undefined;
    values?: string[] | null;
    label?: string;
    disabled?: boolean;
    name: string;
    type?: 'single' | 'multiple';
    width?: string;
    height?: string;
    background?: string;
    border?: string;
    pickerBorder?: string;
    futureDatesOnly?: boolean;
    pastDatesOnly?: boolean;
    restrictedYears?: boolean;
    closeOnSelection?: boolean;
    placeholder?: string;
    textSize?: string;
    labelSize?: string;
    error?: string;
    onclick?: (event: MouseEvent & { currentTarget: EventTarget & HTMLDivElement; }) => void
    ondateChange?: (date: string) => any
  }

  let {
    value = $bindable(),
    valueAsDate = $bindable(new Date()) as Date | undefined,
    values = $bindable([]),
    label = '',
    disabled = false,
    name,
    type = 'single',
    width = 'w-60',
    height = 'h-12',
    background = 'bg-white text-black dark:bg-surface-800 dark:text-white',
    border = 'border border-gray-300 dark:border-surface-600',
    pickerBorder = 'border border-gray-300 dark:border-surface-600',
    futureDatesOnly = false,
    pastDatesOnly = false,
    restrictedYears = true,
    closeOnSelection = false,
    placeholder = 'Pick a date',
    textSize = 'text-sm',
    labelSize = 'text-sm',
    error = '',
    onclick = () => {},
    ondateChange = () => {},
  }: Props = $props();
    
    
    let pickerOut: boolean = $state(false)

    let selectedField: HTMLDivElement = $state() as HTMLDivElement
    let datePickerParent: HTMLDivElement = $state() as HTMLDivElement
    // function to handle document click
    const handleClick = (event: any) => {
        if (!event.target.closest(`.${name}`)) {
            pickerOut = false
        }
    }

    // add event listener on mount and remove it on destroy
    onMount(() => {
        document.addEventListener('click', handleClick)
        const dialogElement = document.querySelector(`dialog`)

        if (dialogElement) {
            dialogElement.addEventListener('click', handleClick)
        }
        return () => {
            document.removeEventListener('click', handleClick)
            if (dialogElement) {
                dialogElement.removeEventListener('click', handleClick)
            }
        }
    })


    function calculatePosition() {
        if (datePickerParent && datePickerParent.getBoundingClientRect().bottom + 10 > window.innerHeight && datePickerParent.getBoundingClientRect().top - datePickerParent.offsetHeight - selectedField.offsetHeight + 10 > 0) {
            datePickerParent.style.top = `${-datePickerParent.offsetHeight + 20}px`
        }

        if (datePickerParent && datePickerParent.getBoundingClientRect().right + 10 > (wrapper.parentNode as HTMLDivElement).getBoundingClientRect().right) {
            datePickerParent.style.left = `-${datePickerParent.offsetWidth - selectedField.offsetWidth}px`
        }
    }  


    let wrapper: HTMLDivElement = $state() as HTMLDivElement

    // this needs to be $derived but I can't figure out if it's possible to make it work with $bindable on top
    $effect(() => {
        valueAsDate = value ? new Date(value) : undefined
    });
    $effect(() => {
        if (datePickerParent) {
            calculatePosition()
        }
    });
    let labelClasses = $derived(disabled ? 'opacity-50' : '')
    let divClasses =  $derived(`${background} ${border} ${disabled ? 'cursor-not-allowed' : 'hover:cursor-pointer'} `)
</script>

<div class='{width} relative' bind:this={wrapper}>

    {#if label}
        <label for="date" class="block mb-2 {labelSize} {labelClasses}">{label}</label>
    {/if}
    <!-- svelte-ignore a11y_click_events_have_key_events -->
    <!-- svelte-ignore a11y_no_static_element_interactions -->
    <!-- svelte-ignore a11y_no_noninteractive_tabindex -->
    <div tabindex={disabled ? -1 : 0}
        bind:this={selectedField}
        onclick={() => {disabled ? undefined : pickerOut = !pickerOut}} class="{name} flex flex-row rounded-lg {divClasses} p-2 {height} items-center"
        onkeydown={(e) => {
            if (e.key === 'Escape') {
                e.preventDefault()
                e.stopPropagation()
                pickerOut = false
            } else if (e.key === 'Enter' || e.key === ' ') {
                e.preventDefault()
                e.stopPropagation()
                pickerOut = !pickerOut
            } else if (e.key === 'Tab') {
                pickerOut = false
            } else if (e.key === 'ArrowUp' || e.key === 'ArrowLeft') {
                e.preventDefault()
                if (type === 'single') {
                    if (futureDatesOnly) {
                        let previousDay = dayjs(value).subtract(1, 'day')
                        let today = dayjs()
                        value = value && !previousDay.isBefore(today) ? previousDay.format('YYYY-MM-DD') : today.format('YYYY-MM-DD')
                    } else {
                        value = value ? dayjs(value).subtract(1, 'day').format('YYYY-MM-DD') : dayjs().format('YYYY-MM-DD')
                    }
                } else {
                    pickerOut = !pickerOut
                }
            } else if (e.key === 'ArrowDown' || e.key === 'ArrowRight') {
                e.preventDefault()
                if (type === 'single') {
                    value = value ? dayjs(value).add(1, 'day').format('YYYY-MM-DD') : dayjs().format('YYYY-MM-DD')
                } else {
                    pickerOut = !pickerOut
                }
            }
        }}>
        <div onclick={onclick} class="{disabled ? 'opacity-50' : ''} items-center flex">
            {@html calendarSVG}
        </div>
        <span class="ml-2 {textSize} text-black dark:text-white {disabled ? 'opacity-50' : ''}">
            {#if type === 'multiple' && values}
                {values.length === 0 ? 'Pick Dates' : values.map((date) => dayjs(date).format('DD/MM/YYYY')).join(', ')}
            {:else if type === 'single'}
                {value ? dayjs(value).format('DD/MM/YYYY') : placeholder}
            {/if}
        </span>
    </div>

  {#if pickerOut}
    <div bind:this={datePickerParent} class="w-fit rounded-md absolute flex z-50 {background} {pickerBorder} mt-1" 
    in:fly={{ y: -10, duration: 200 }} out:fly={{ y: -10, duration: 200 }}>
        <div class="p-2">
            {#if type === 'single'}
                <DatePicker {futureDatesOnly} {pastDatesOnly} {restrictedYears} size='small' bind:date={value} 
                ondateChange={(e) => {
                    pickerOut = closeOnSelection ? false : pickerOut
                    ondateChange(e)
                }} />
            {:else if type === 'multiple'}
                <MultiDatePicker {futureDatesOnly} size='small' bind:dates={values} />
            {/if}
      </div>
    </div>
  {/if}
  
  {#if error}
    <span class="text-error-500 mb-2">{error}</span>
    {/if}

</div>
