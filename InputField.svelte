<script lang="ts">
  import dayjs from 'dayjs'
  import SelectSimple from './SelectSimple.svelte'

  interface Props {
    label?: string
    outerLabel?: string
    background?: string
    width?: string
    value?: number | string | null
    type?: 'money' | 'number' | 'text' | 'password' | 'dob'
    includeNegativeNumbers?: boolean
    disabled?: boolean
    applyDisabledClasses?: boolean
    layout?: 'bordered' | 'open'
    trail?: string
    height?: string
    placeholder?: string
    minvalue?: string
    maxvalue?: string
    trailOptions?: { name: string; value: string | number }[]
    trailValue?: string | number | undefined | null
    maxCharacters?: number | undefined
    setDelimiter?: boolean
    joinWithDelimiter?: boolean
    delimiter?: string | undefined
    numberOfDelimiters?: number | undefined
    delimiterWidth?: string | undefined
    delimitInputPadding?: string
    delimitersDivPadding?: string
    focusBeginning?: boolean
    leadingZeros?: boolean
    maxDelimiterValues?: number[]
    textSize?: 'text-sm' | 'text-md' | 'text-lg'
    labelSize?: string
    compulsory?: boolean
    error?: string
    oninput?: (v: typeof value) => any
  }

  let {
    label = '',
    outerLabel = '',
    background = 'bg-white text-black dark:bg-surface-700 dark:text-white',
    width = '',
    value = $bindable(),
    type = 'text',
    includeNegativeNumbers = true,
    disabled = false,
    applyDisabledClasses = true,
    layout = 'bordered',
    trail = '',
    height = '',
    placeholder = '',
    minvalue = '',
    maxvalue = '',
    trailOptions = [],
    trailValue = $bindable(undefined),
    maxCharacters = undefined,
    setDelimiter = false,
    joinWithDelimiter = false,
    delimiter = undefined,
    numberOfDelimiters = undefined,
    delimiterWidth = undefined,
    delimitInputPadding = '',
    delimitersDivPadding = '',
    focusBeginning = false,
    leadingZeros = false,
    maxDelimiterValues = [],
    textSize = 'text-sm',
    labelSize = 'text-sm',
    compulsory = false,
    error = '',
    oninput = () => {}
  }: Props = $props()

  let baseInputClasses = `w-full ${textSize} ${background} focus:border-black dark:focus:border-gray-400 focus:outline-none focus:ring-0 peer`
  let baseLabelClasses =
    'absolute duration-200 transform scale-75 origin-[0] peer-focus:black peer-focus:dark:text-black peer-placeholder-shown:scale-100 peer-focus:scale-75'

  let delimiterValueOne: string | number | undefined | null = $state(undefined)
  let delimiterValueTwo: string | number | undefined | null = $state(undefined)
  let delimiterValueThree: string | number | undefined | null = $state(undefined)
  let delimiterValueFour: string | number | undefined | null = $state(undefined)
  let delimiterValueFive: string | number | undefined | null = $state(undefined)

  const delimiters: (string | number | undefined | null)[] = [
    delimiterValueOne,
    delimiterValueTwo,
    delimiterValueThree,
    delimiterValueFour,
    delimiterValueFive
  ]

  let firstInitialised = false
  const getFirstInitialised = () => firstInitialised

  function updateValueWithDelimiters(...reactivityValues: (string | number | undefined | null)[]) {
    value = reactivityValues.join(joinWithDelimiter ? delimiter : '')
  }

  function updateDelimitersWithValue() {
    if (!maxCharacters) return

    delimiterValueOne = value?.toString().slice(0, maxCharacters)
    delimiterValueTwo = value?.toString().slice(maxCharacters, maxCharacters * 2)
    delimiterValueThree = value?.toString().slice(maxCharacters * 2, maxCharacters * 3)
    delimiterValueFour = value?.toString().slice(maxCharacters * 3, maxCharacters * 4)
    delimiterValueFive = value?.toString().slice(maxCharacters * 4, maxCharacters * 5)
    firstInitialised = true
  }

  function onDelimiterInput(
    event: Event & { currentTarget: EventTarget & HTMLInputElement },
    i: number,
    customMaxChars?: number
  ) {
    let valueString = event.currentTarget.value
    let valueNumber = parseInt(valueString)

    if (isNaN(valueNumber)) {
      valueString = valueString.slice(0, -1)
      valueNumber = parseInt(valueString)
      if (isNaN(valueNumber)) {
        // @ts-ignore
        valueNumber = ''
      }
    } else {
      const lastChar = valueString.slice(-1)
      if (!lastChar.match(numberRegex)) {
        valueString = valueString.slice(0, -1)
        valueNumber = parseInt(valueString)
      }
    }

    const maxChars = maxCharacters || customMaxChars || 0

    if (maxChars && valueString.length === maxChars) {
      // focus next element
      let nextElement
      nextElement = event.currentTarget.nextElementSibling?.nextElementSibling
      // @ts-ignore
      if (nextElement && nextElement.focus) {
        // @ts-ignore
        nextElement.focus()
      }
    } else if (maxChars && valueString.length > maxChars) {
      valueString = valueString.slice(0, maxChars)
      valueNumber = parseInt(valueString)
      // focus next element
      let nextElement
      nextElement = event.currentTarget.nextElementSibling?.nextElementSibling
      // @ts-ignore
      if (nextElement && nextElement.focus) {
        // @ts-ignore
        nextElement.focus()
      }
    }

    if (maxDelimiterValues && maxDelimiterValues.length > i) {
      if (valueNumber && valueNumber > maxDelimiterValues[i]) {
        valueString = maxDelimiterValues[i].toString()
        valueNumber = maxDelimiterValues[i]
      }
    }

    if (type === 'dob') {
      if (i === 0) {
        delimiterValueOne = valueString
      } else if (i === 1) {
        delimiterValueTwo = valueString
      } else if (i === 2) {
        delimiterValueThree = valueString
      }
      let updatedDob = checkDobValidity(
        delimiterValueOne?.toString(),
        delimiterValueTwo?.toString(),
        delimiterValueThree?.toString()
      )
      if (updatedDob.year !== delimiterValueThree) {
        delimiterValueThree = updatedDob.year
        if (delimiterValueThree?.length === 4) {
          focusNextElement(event)
        }
      }

      if (updatedDob.month !== delimiterValueTwo) {
        delimiterValueTwo = updatedDob.month
      }
      if (delimiterValueTwo?.length === 2) {
        focusNextElement(event)
        if (delimiterValueThree?.length === 4) {
          focusNextElement(event)
        }
      }

      if (updatedDob.day !== delimiterValueOne) {
        delimiterValueOne = updatedDob.day
        if (delimiterValueOne?.length === 2) {
          focusNextElement(event)
          if (delimiterValueTwo?.length === 2) {
            focusNextElement(event)
            if (delimiterValueThree?.length === 4) {
              focusNextElement(event)
            }
          }
        }
      }
    } else {
      if (i === 0) {
        delimiterValueOne =
          valueNumber && maxvalue.length > 0 && valueNumber > parseInt(maxvalue)
            ? maxvalue
            : valueNumber && minvalue.length > 0 && valueNumber < parseInt(minvalue)
              ? minvalue
              : leadingZeros
                ? valueString
                : valueNumber.toString()
      } else if (i === 1) {
        delimiterValueTwo =
          valueNumber && maxvalue.length > 0 && valueNumber > parseInt(maxvalue)
            ? maxvalue
            : valueNumber && minvalue.length > 0 && valueNumber < parseInt(minvalue)
              ? minvalue
              : leadingZeros
                ? valueString
                : valueNumber.toString()
      } else if (i === 2) {
        delimiterValueThree =
          valueNumber && maxvalue.length > 0 && valueNumber > parseInt(maxvalue)
            ? maxvalue
            : valueNumber && minvalue.length > 0 && valueNumber < parseInt(minvalue)
              ? minvalue
              : leadingZeros
                ? valueString
                : valueNumber.toString()
      } else if (i === 3) {
        delimiterValueFour =
          valueNumber && maxvalue.length > 0 && valueNumber > parseInt(maxvalue)
            ? maxvalue
            : valueNumber && minvalue.length > 0 && valueNumber < parseInt(minvalue)
              ? minvalue
              : leadingZeros
                ? valueString
                : valueNumber.toString()
      } else if (i === 4) {
        delimiterValueFive =
          valueNumber && maxvalue.length > 0 && valueNumber > parseInt(maxvalue)
            ? maxvalue
            : valueNumber && minvalue.length > 0 && valueNumber < parseInt(minvalue)
              ? minvalue
              : leadingZeros
                ? valueString
                : valueNumber.toString()
      }
    }
  }

  function onDelimiterFocus(
    event: Event & { currentTarget: EventTarget & HTMLInputElement },
    i: number
  ) {
    if (i === 0) {
      return
    }

    if (!delimiters[i]) {
      const prevElement = event.currentTarget.previousElementSibling?.previousElementSibling
      // @ts-ignore
      const prevElementValue = prevElement?.value
      if (prevElementValue && prevElementValue.length === maxCharacters) {
        return
      } else if (prevElement) {
        // @ts-ignore
        prevElement.focus()
      }
    }
  }

  function onDelimiterKeyDown(
    event: KeyboardEvent & { currentTarget: EventTarget & HTMLInputElement },
    i: number
  ) {
    const cursorPosition = event.currentTarget.selectionStart

    // Left arrow key
    if (event.key === 'ArrowLeft') {
      if (cursorPosition === 0 && i > 0) {
        focusPreviousElement(event)
      }
    }

    // Right arrow key
    if (event.key === 'ArrowRight') {
      if (cursorPosition === event.currentTarget.value.length && i < numberOfDelimiters!) {
        focusNextElement(event)
      }
    }

    // Backspace key
    if (
      event.key === 'Backspace' &&
      (event.currentTarget.value.length === 0 || cursorPosition === 0) &&
      i > 0
    ) {
      focusPreviousElement(event)
    }
  }

  function focusNextElement(event: Event & { currentTarget: EventTarget & HTMLInputElement }) {
    const nextElement = event.currentTarget.nextElementSibling?.nextElementSibling
    // @ts-ignore
    if (nextElement && nextElement.focus) {
      // @ts-ignore
      nextElement.focus()
    }
  }

  function focusPreviousElement(event: Event & { currentTarget: EventTarget & HTMLInputElement }) {
    const prevElement = event.currentTarget.previousElementSibling?.previousElementSibling
    // @ts-ignore
    if (prevElement && prevElement.focus) {
      // @ts-ignore
      prevElement.focus()
    }
  }

  function checkDobValidity(day?: string, month?: string, year?: string) {
    if (day && day.length === 1 && Number.parseInt(day) >= 4 && Number.parseInt(day) < 10) {
      day = '0' + day
    }
    if (month && month.length === 1 && Number.parseInt(month) >= 2 && Number.parseInt(month) < 10) {
      month = '0' + month
    }
    if (
      year &&
      year.length &&
      Number.parseInt(year) > dayjs().get('year') % 100 &&
      Number.parseInt(year) <= 100
    ) {
      year = '19' + year
    }

    if (day && month && day.length && month.length === 2 && Number.parseInt(day) > 27) {
      if (
        (Number.parseInt(day) > 30 && Number.parseInt(month) === 4) ||
        Number.parseInt(month) === 6 ||
        Number.parseInt(month) === 9 ||
        Number.parseInt(month) === 11
      ) {
        day = '30'
      } else if (Number.parseInt(month) === 2) {
        if (year && year.length === 4 && Number.parseInt(year) % 4 === 0) {
          day = '29'
        } else if (year && year.length === 4 && Number.parseInt(year) % 4 !== 0) {
          day = '28'
        }
      }
    }

    if (day && day.length && Number.parseInt(day) > 31) {
      day = '31'
    }
    if (month && month.length && Number.parseInt(month) > 12) {
      month = '12'
    }
    if (
      year &&
      ((year.length === 3 && Number.parseInt(year) > dayjs().get('year') / 10) ||
        Number.parseInt(year) > dayjs().get('year'))
    ) {
      year = String(dayjs().get('year'))
    }

    value = (year || '') + '-' + (month || '') + '-' + (day || '')

    return { year, month, day }
  }
  let disabledClasses = $derived('text-black border-gray-400 dark:text-white dark:border-gray-600')
  let inputSpecificClasses = $derived(
    layout === 'bordered'
      ? ` 
                ${type === 'money' ? 'px-8' : 'px-3'}
                ${label ? 'pt-4 pb-3' : 'py-3.5'}
                ${trailOptions.length > 0 ? 'border-0 border-l border-t border-b rounded-l-lg' : 'border rounded-lg'}
                relative
                z-10
                `
      : ` 
                block 
                py-2.5 
                px-0 
                border-0 
                border-b-2 
                appearance-none 
                `
  )
  let labelSpecificClasses = $derived(
    layout === 'bordered'
      ? ` 
                ${background}
                ${type === 'money' ? 'peer-placeholder-shown:left-5' : ''}
                -translate-y-4 
                top-1
                origin-[0]
                px-2.5
                z-20
                peer-focus:px-2 
                peer-placeholder-shown:-translate-y-1/2
                peer-placeholder-shown:top-1/2
                peer-placeholder-shown:z-0
                peer-focus:top-2
                peer-focus:-translate-y-4
                peer-focus:start-1
                peer-focus:z-20
                start-1
                `
      : `
                text-sm
                -translate-y-6
                top-3
                -z-10
                peer-focus:start-0
                peer-placeholder-shown:scale-100
                peer-placeholder-shown:translate-y-0
                peer-focus:-translate-y-6
            `
  )
  let inputClasses = $derived(
    `${baseInputClasses} ${disabledClasses} ${inputSpecificClasses} ${height} ${disabled && applyDisabledClasses ? 'opacity-50 cursor-not-allowed' : ''}`
  )
  let labelClasses = $derived(`${baseLabelClasses} ${disabledClasses} ${labelSpecificClasses}`)
  let outerLabelClasses = $derived(
    disabled && applyDisabledClasses ? 'text-gray-400' : 'text-black dark:text-white'
  )
  let delimiterInputClasses = $derived(
    `text-sm bg-transparent border-0 focus:outline-none focus:ring-0 peer py-2.5 ${disabledClasses} ${delimiterWidth} ${delimitInputPadding}`
  )
  let numberRegex = $derived(includeNegativeNumbers ? /[-]?[0-9]+/ : /[0-9]+/)
  $effect(() => {
    if (setDelimiter) {
      if (!getFirstInitialised()) {
        updateDelimitersWithValue()
      } else {
        updateValueWithDelimiters(
          delimiterValueOne,
          delimiterValueTwo,
          delimiterValueThree,
          delimiterValueFour,
          delimiterValueFive
        )
      }
    }
  })

  $effect(() => {
    oninput(value)
  })
</script>

<div class="relative hover:cursor-text {width}">
  {#if outerLabel}
    <label for={label} class="block mb-2 {outerLabelClasses} {labelSize}">
      {outerLabel}
      {#if compulsory}
        <span class="text-red-500">*</span>
      {/if}
    </label>
  {/if}

  <div class="relative flex {height}">
    {#if type === 'text'}
      <input
        autocomplete="off"
        {disabled}
        type="text"
        id={outerLabel + label}
        class={inputClasses}
        {placeholder}
        bind:value
      />
    {:else if type === 'password'}
      <input
        autocomplete="off"
        {disabled}
        type="password"
        id={outerLabel + label}
        class={inputClasses}
        {placeholder}
        bind:value
      />
    {:else if type === 'number'}
      {#if setDelimiter && numberOfDelimiters}
        <div
          class="flex flex-row border border-gray-400 dark:border-gray-600 rounded-lg {width} h-full {delimitersDivPadding} {background}"
        >
          <input
            autocomplete="off"
            {disabled}
            type="text"
            id={outerLabel + label}
            class={delimiterInputClasses}
            {placeholder}
            bind:value={delimiterValueOne}
            oninput={(e) => onDelimiterInput(e, 0)}
            onfocus={(e) => onDelimiterFocus(e, 0)}
            onkeydown={(e) => onDelimiterKeyDown(e, 0)}
          />

          <span class="text-gray-400 text-lg items-center flex">{delimiter}</span>

          <input
            autocomplete="off"
            {disabled}
            type="text"
            id={outerLabel + label}
            class={delimiterInputClasses}
            {placeholder}
            bind:value={delimiterValueTwo}
            oninput={(e) => onDelimiterInput(e, 1)}
            onfocus={(e) => onDelimiterFocus(e, 1)}
            onkeydown={(e) => onDelimiterKeyDown(e, 1)}
          />

          {#if numberOfDelimiters > 1}
            <span class="text-gray-400 text-lg items-center flex">{delimiter}</span>

            <input
              autocomplete="off"
              {disabled}
              type="text"
              id={outerLabel + label}
              class={delimiterInputClasses}
              {placeholder}
              bind:value={delimiterValueThree}
              oninput={(e) => onDelimiterInput(e, 2)}
              onfocus={(e) => onDelimiterFocus(e, 2)}
              onkeydown={(e) => onDelimiterKeyDown(e, 2)}
            />
          {/if}
          {#if numberOfDelimiters > 2}
            <span class="text-gray-400 text-lg items-center flex">{delimiter}</span>

            <input
              autocomplete="off"
              {disabled}
              type="text"
              id={outerLabel + label}
              class={delimiterInputClasses}
              {placeholder}
              bind:value={delimiterValueFour}
              oninput={(e) => onDelimiterInput(e, 3)}
              onfocus={(e) => onDelimiterFocus(e, 3)}
              onkeydown={(e) => onDelimiterKeyDown(e, 3)}
            />
          {/if}
          {#if numberOfDelimiters > 3}
            <span class="text-gray-400 text-lg items-center flex">{delimiter}</span>

            <input
              autocomplete="off"
              {disabled}
              type="text"
              id={outerLabel + label}
              class={delimiterInputClasses}
              {placeholder}
              bind:value={delimiterValueFive}
              oninput={(e) => onDelimiterInput(e, 4)}
              onfocus={(e) => onDelimiterFocus(e, 4)}
              onkeydown={(e) => onDelimiterKeyDown(e, 4)}
            />
          {/if}

          <!-- svelte-ignore a11y_click_events_have_key_events -->
          <!-- svelte-ignore a11y_no_static_element_interactions -->
          <div
            class="w-full"
            onclick={(event) => {
              // @ts-ignore
              event.currentTarget.previousElementSibling.focus()
            }}
          ></div>
        </div>
      {:else}
        <input
          autocomplete="off"
          {disabled}
          type="number"
          id={outerLabel + label}
          class={inputClasses}
          {placeholder}
          bind:value
          oninput={(event) => {
            const valueString = event.currentTarget.value
            let valueNumber = parseInt(valueString)

            if (maxCharacters && valueString.length > maxCharacters) {
              valueNumber = parseInt(valueString.slice(0, maxCharacters))
            }

            if (isNaN(valueNumber)) {
              valueNumber = parseInt(valueString.slice(0, -1))
            } else {
              const lastChar = valueString.slice(-1)
              if (!lastChar.match(numberRegex)) {
                valueNumber = parseInt(valueString.slice(0, -1))
              }
            }

            value =
              valueNumber && maxvalue.length > 0 && valueNumber > parseInt(maxvalue)
                ? maxvalue
                : valueNumber && minvalue.length > 0 && valueNumber < parseInt(minvalue)
                  ? minvalue
                  : valueNumber.toString()
          }}
        />
      {/if}
    {:else if type === 'money'}
      <input
        autocomplete="off"
        {disabled}
        type="number"
        id={outerLabel + label}
        class={inputClasses}
        {placeholder}
        bind:value
        onkeydown={(event) => {
          // @ts-ignore
          if (event.target.value.length > 0 && event?.key === '-') {
            event.preventDefault()
          }
        }}
        oninput={() => {
          value =
            value && maxvalue.length > 0 && parseInt(value.toString()) > parseInt(maxvalue)
              ? maxvalue
              : value && minvalue.length > 0 && parseInt(value.toString()) < parseInt(minvalue)
                ? minvalue
                : value
        }}
      />

      <p
        class="
                absolute
                left-3
                {label ? 'top-4 peer-focus:top-4 peer-placeholder-shown:top-4' : 'top-3.5'}
                z-20
                duration-200
                {disabled && applyDisabledClasses ? 'opacity-50' : ''}
                "
      >
        £
      </p>
    {:else if type === 'dob'}
      <div
        class="flex flex-row border focus:border-black border-gray-400 dark:border-gray-600 rounded-lg {width} h-full {background}"
      >
        <input
          autocomplete="off"
          {disabled}
          type="text"
          id={outerLabel + label}
          class="text-sm bg-transparent border-0 focus:outline-none focus:ring-0 peer py-2.5 w-12"
          placeholder={'DD'}
          bind:value={delimiterValueOne}
          oninput={(e) => onDelimiterInput(e, 0, 2)}
          onkeydown={(e) => onDelimiterKeyDown(e, 0)}
          onfocusin={(e) => {
            // @ts-ignore
            e.target.parentElement.classList.remove('border-gray-400', 'dark:border-gray-600')
            // @ts-ignore
            e.target.parentElement.classList.add('border-black', 'dark:border-gray-400')
          }}
          onfocusout={(e) => {
            // @ts-ignore
            e.target.parentElement.classList.remove('border-black', 'dark:border-gray-400')
            // @ts-ignore
            e.target.parentElement.classList.add('border-gray-400', 'dark:border-gray-600')
          }}
        />

        <span class="text-gray-400 text-lg items-center flex">/</span>

        <input
          autocomplete="off"
          {disabled}
          type="text"
          id={outerLabel + label}
          class="text-sm bg-transparent border-0 focus:outline-none focus:ring-0 peer py-2.5 w-12"
          placeholder={'MM'}
          bind:value={delimiterValueTwo}
          oninput={(e) => onDelimiterInput(e, 1, 2)}
          onkeydown={(e) => onDelimiterKeyDown(e, 1)}
          onfocusin={(e) => {
            // @ts-ignore
            e.target.parentElement.classList.remove('border-gray-400', 'dark:border-gray-600')
            // @ts-ignore
            e.target.parentElement.classList.add('border-black', 'dark:border-gray-400')
          }}
          onfocusout={(e) => {
            // @ts-ignore
            e.target.parentElement.classList.remove('border-black', 'dark:border-gray-400')
            // @ts-ignore
            e.target.parentElement.classList.add('border-gray-400', 'dark:border-gray-600')
          }}
        />

        <span class="text-gray-400 text-lg items-center flex">/</span>

        <input
          autocomplete="off"
          {disabled}
          type="text"
          id={outerLabel + label}
          class="text-sm bg-transparent border-0 focus:outline-none focus:ring-0 peer py-2.5 w-16"
          placeholder={'YYYY'}
          bind:value={delimiterValueThree}
          oninput={(e) => onDelimiterInput(e, 2, 4)}
          onkeydown={(e) => onDelimiterKeyDown(e, 2)}
          onfocusin={(e) => {
            // @ts-ignore
            e.target.parentElement.classList.remove('border-gray-400', 'dark:border-gray-600')
            // @ts-ignore
            e.target.parentElement.classList.add('border-black', 'dark:border-gray-400')
          }}
          onfocusout={(e) => {
            // @ts-ignore
            e.target.parentElement.classList.remove('border-black', 'dark:border-gray-400')
            // @ts-ignore
            e.target.parentElement.classList.add('border-gray-400', 'dark:border-gray-600')
          }}
        />
      </div>
    {/if}

    {#if trail}
      <p
        class="
                absolute
                right-3
                {label ? 'top-4 peer-focus:top-4 peer-placeholder-shown:top-4' : 'top-3.5'}
                z-20
                duration-200
                {disabledClasses}
                "
      >
        {trail}
      </p>
    {/if}
    {#if label}
      <label for="floating_outlined" class={labelClasses}>{label}</label>
    {/if}

    {#if trailOptions.length > 0}
      <div class="w-44">
        <SelectSimple
          {disabled}
          border="rounded-none rounded-r-xl border border-gray-400 dark:border-gray-500"
          name="trailoptions"
          label=""
          type="single"
          bind:value={trailValue}
          options={trailOptions}
        />
      </div>
    {/if}
  </div>

  {#if error && error.length > 0}
    <span class="text-error-500">{error}</span>
  {/if}
</div>

<style>
  input[type='number']::-webkit-inner-spin-button,
  input[type='number']::-webkit-outer-spin-button {
    -webkit-appearance: none;
    margin: 0;
  }

  input[type='number'] {
    -moz-appearance: textfield;
  }
</style>
