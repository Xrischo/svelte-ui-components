<script lang="ts">
  import dayjs from 'dayjs'
  import { calendarSVG } from ''
  import { onMount } from 'svelte'
  import { fly } from 'svelte/transition'
  import DateRangePicker from './DateRangePicker.svelte'

  interface Props {
    date1?: string | Date
    date2?: string | Date
    label?: string
    disabled?: boolean
    name: string
    width?: string
    height?: string
    background?: string
    border?: string
    futureDatesOnly?: boolean
    pastDatesOnly?: boolean
    selectFirstDate?: boolean
    error?: string
    onclick?: (e: MouseEvent & { currentTarget: EventTarget & HTMLElement }) => void
    ondateChange?: (date1: string | Date | undefined, date2: string | Date | undefined) => void
  }

  let {
    date1 = $bindable(dayjs().format('YYYY-MM-DD')),
    date2 = $bindable(dayjs().format('YYYY-MM-DD')),
    label = '',
    disabled = false,
    name,
    width = 'w-56',
    height = 'h-12',
    background = 'bg-white text-black dark:bg-surface-800 dark:text-white',
    border = 'border border-gray-300 dark:border-surface-600',
    futureDatesOnly = false,
    pastDatesOnly = false,
    selectFirstDate = true,
    error = '',
    onclick = (e: MouseEvent & { currentTarget: EventTarget & HTMLElement }) => {},
    ondateChange = (date1: string | Date | undefined, date2: string | Date | undefined) => {}
  }: Props = $props()

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
    return () => document.removeEventListener('click', handleClick)
  })

  function calculatePosition() {
    if (
      datePickerParent &&
      datePickerParent.getBoundingClientRect().bottom + 10 > window.innerHeight &&
      datePickerParent.getBoundingClientRect().top -
        datePickerParent.offsetHeight -
        selectedField.offsetHeight +
        10 >
        0
    ) {
      datePickerParent.style.top = `${-datePickerParent.offsetHeight + 20}px`
    }

    if (datePickerParent && datePickerParent.getBoundingClientRect().right > window.innerWidth) {
      datePickerParent.style.left = `-${datePickerParent.offsetWidth - selectedField.offsetWidth}px`
    }
  }

  let wrapper: HTMLDivElement = $state() as HTMLDivElement
  $effect(() => {
    datePickerParent && calculatePosition()
  })
  let labelClasses = $derived(disabled ? 'text-gray-400' : 'text-black dark:text-white')
  let divClasses = $derived(
    disabled
      ? 'bg-surface-400 dark:bg-surface-700 hover:cursor-not-allowed'
      : `${background} ${border} hover:cursor-pointer`
  )
</script>

<div class="{width} relative" bind:this={wrapper}>
  {#if label}
    <label for="date" class="block mb-2 text-sm {labelClasses}">{label}</label>
  {/if}
  <!-- svelte-ignore a11y_click_events_have_key_events -->
  <!-- svelte-ignore a11y_no_static_element_interactions -->
  <!-- svelte-ignore a11y_no_noninteractive_tabindex -->
  <div
    tabindex={disabled ? -1 : 0}
    bind:this={selectedField}
    onclick={() => {
      pickerOut = !pickerOut
    }}
    class="{name} flex flex-row rounded-lg {divClasses} p-2 {height} items-center"
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
      }
    }}
  >
    <div {onclick} class="{disabled ? 'opacity-50' : ''} items-center flex">
      {@html calendarSVG}
    </div>
    <span class="ml-2 text-sm text-black dark:text-white {disabled ? 'opacity-50' : ''}">
      {date1 && date2
        ? dayjs(date1).format('DD/MM/YYYY') + ' - ' + dayjs(date2).format('DD/MM/YYYY')
        : date1
          ? dayjs(date1).format('DD/MM/YYYY') + ' - Date'
          : date2
            ? 'Date - ' + dayjs(date2).format('DD/MM/YYYY')
            : 'Pick a range'}
    </span>
  </div>

  {#if pickerOut}
    <div
      bind:this={datePickerParent}
      class="w-fit rounded-md absolute flex z-50 {background} {border} mt-1"
      in:fly={{ y: -10, duration: 200 }}
      out:fly={{ y: -10, duration: 200 }}
    >
      <!-- svelte-ignore a11y_click_events_have_key_events -->
      <!-- svelte-ignore a11y_no_static_element_interactions -->
      <div class="p-2" onclick={(e) => {
        e.stopPropagation();
        onclick(e);
      }}>
        <DateRangePicker
          bind:date1
          bind:date2
          {futureDatesOnly}
          {pastDatesOnly}
          {selectFirstDate}
          {background}
          {ondateChange}
        />
      </div>
    </div>
  {/if}

  {#if error}
    <span class="text-error-500 mb-2">{error}</span>
  {/if}
</div>
