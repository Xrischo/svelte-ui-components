<script lang="ts">
  import dayjs from 'dayjs'
  import { onMount } from 'svelte'

  interface Props {
    label?: string
    background?: string
    time?: string
    minValue?: string
    maxValue?: string
    minDistance?: number
    distanceUnder?: string
    distanceOver?: string
    width?: string
    height?: string
    border?: string
    disabled?: boolean
    error?: string
    ontimeChange?: (time: string) => void
  }

  let {
    label = '',
    background = 'bg-white text-black dark:bg-surface-700 dark:text-white',
    time = $bindable('00:00'),
    minValue = $bindable('00:00'),
    maxValue = $bindable('23:59'),
    minDistance = 0,
    distanceUnder = $bindable(),
    distanceOver = $bindable(),
    width = 'w-34',
    height = 'h-12',
    border = 'border border-gray-400 dark:border-gray-500',
    disabled = false,
    error = '',
    ontimeChange = () => {}
  }: Props = $props()

  let defaultInputClass: string =
    'w-14 bg-transparent text-center text-xl focus:outline-none focus:ring-0 rounded-md border-0'

  function checkDistance() {
    if (distanceUnder && distanceOver) return
    if (
      distanceUnder &&
      dayjs(`01-01-2000 ${distanceUnder}`).diff(
        dayjs(`01-01-2000 ${hourInput.value}:${minuteInput.value}`),
        'minute'
      ) != minDistance
    )
      distanceUnder = dayjs(`01-01-2000 ${hourInput.value}:${minuteInput.value}`)
        .subtract(minDistance, 'minute')
        .format('HH:mm')
    else if (
      distanceOver &&
      dayjs(`01-01-2000 ${distanceOver}`).diff(
        dayjs(`01-01-2000 ${hourInput.value}:${minuteInput.value}`),
        'minute'
      ) != minDistance
    )
      distanceOver = dayjs(`01-01-2000 ${hourInput.value}:${minuteInput.value}`)
        .add(minDistance, 'minute')
        .format('HH:mm')
  }
  function shiftInputFocus() {
    let focused = document.activeElement
    let inputs = document.getElementsByTagName('input')

    let index = Array.prototype.indexOf.call(inputs, focused)

    if (index > -1 && index < inputs.length - 1) {
      inputs[index + 1].focus()
    }
  }

  function unFocus() {
    minuteInput.blur()
  }

  let minHour: string = $derived(minValue.split(':')[0].slice(0, 2))
  let minMinute: string = $derived(minValue.split(':')[1].slice(0, 2))
  let maxHour: string = $derived(maxValue.split(':')[0].slice(0, 2))
  let maxMinute: string = $derived(maxValue.split(':')[1].slice(0, 2))

  let hourInput: HTMLInputElement = $state() as HTMLInputElement
  let minuteInput: HTMLInputElement = $state() as HTMLInputElement

  let hourFocused: boolean
  let minuteFocused: boolean

  function handleHoursInput() {
    hourInput.value = hourInput.value.slice(0, 2)

    time = hourInput.value + ':' + minuteInput.value

    if (hourInput.value.length === 2 || Number.parseInt(hourInput.value) > 2) {
      minuteInput.focus()
    }
    ontimeChange(time)
  }

  function handleMinutesInput() {
    minuteInput.value = minuteInput.value.slice(0, 2)
    time = hourInput.value + ':' + minuteInput.value

    if (minuteInput.value.length === 2 || Number.parseInt(minuteInput.value) > 5) {
      shiftInputFocus()
      unFocus()
    }

    ontimeChange(time)
  }

  function handleHoursFocusOut() {
    if (hourInput.value.length === 0) {
      hourInput.value = '00'
      time = hourInput.value + ':' + minuteInput.value
    } else if (hourInput.value.length === 1) {
      hourInput.value = '0' + hourInput.value
      time = hourInput.value + ':' + minuteInput.value
    } else if (hourInput.value.length === 2 && parseInt(hourInput.value) > 23) {
      hourInput.value = '23'
      time = hourInput.value + ':' + minuteInput.value
    }

    if (hourInput.value > maxHour) {
      maxValue = hourInput.value + ':' + minuteInput.value
    }
    if (hourInput.value < minHour) {
      minValue = hourInput.value + ':' + minuteInput.value
    }
    if (minuteInput.value > maxMinute && hourInput.value === maxHour) {
      maxValue = hourInput.value + ':' + minuteInput.value
    }
    if (minuteInput.value < minMinute && hourInput.value === minHour) {
      minValue = hourInput.value + ':' + minuteInput.value
    }
    if (minDistance && (distanceOver || distanceUnder)) {
      checkDistance()
    }

    changeInputValues()
    ontimeChange(time)
    hourFocused = false
  }

  function handleMinutesFocusOut() {
    if (minuteInput.value.length === 0) {
      minuteInput.value = '00'
      time = hourInput.value + ':' + minuteInput.value
    } else if (minuteInput.value.length === 1) {
      minuteInput.value = '0' + minuteInput.value
      time = hourInput.value + ':' + minuteInput.value
    } else if (minuteInput.value.length === 2 && parseInt(minuteInput.value) > 59) {
      minuteInput.value = '59'
      time = hourInput.value + ':' + minuteInput.value
    }

    if (minuteInput.value > maxMinute && hourInput.value === maxHour) {
      maxValue = hourInput.value + ':' + minuteInput.value
    }
    if (minuteInput.value < minMinute && hourInput.value === minHour) {
      minValue = hourInput.value + ':' + minuteInput.value
    }

    if (minDistance && (distanceOver || distanceUnder)) {
      checkDistance()
    }

    minuteFocused = false

    changeInputValues()
    ontimeChange(time)
  }

  function changeInputValues() {
    const timeSplit = time.split(':')
    hourInput.value = timeSplit[0]
    minuteInput.value = timeSplit[1]
  }

  function handleHoursFocusIn() {
    hourFocused = true
    hourInput.select()
  }

  function handleMinutesFocusIn() {
    minuteFocused = true
    minuteInput.select()
  }

  onMount(() => {
    changeInputValues()

    // when the variable changes
    $effect(() => {
        time && changeInputValues()
    })
  })

</script>

<div class="relative">
  {#if label}
    <label for="timepicker" class="block mb-2 text-sm text-black dark:text-white">{label}</label>
  {/if}
  <div class="{background} {height} {width} {border} rounded-md items-center flex">
    <div class="timepicker flex flex-row w-full">
      <div class="items-center flex w-full justify-center">
        <input
          {disabled}
          bind:this={hourInput}
          type="number"
          min="0"
          max="23"
          onfocusin={handleHoursFocusIn}
          onfocusout={handleHoursFocusOut}
          oninput={handleHoursInput}
          class={defaultInputClass}
        />
      </div>
      <span class="flex text-2xl items-center mb-2"> : </span>
      <div class="items-center flex w-full justify-center">
        <input
          {disabled}
          bind:this={minuteInput}
          type="number"
          min="0"
          max="59"
          onfocusin={handleMinutesFocusIn}
          onfocusout={handleMinutesFocusOut}
          oninput={handleMinutesInput}
          class={defaultInputClass}
        />
      </div>
    </div>
  </div>

  {#if error}
    <span class="text-error-500 mb-2">{error}</span>
  {/if}
</div>

<style lang="scss">
  input[type='number']::-webkit-inner-spin-button,
  input[type='number']::-webkit-outer-spin-button {
    -webkit-appearance: none;
    margin: 0;
  }

  input[type='number'] {
    -moz-appearance: textfield;
  }

  ::selection {
    background-color: rgba(var(--color-primary-400) / 0.4);
  }
</style>
