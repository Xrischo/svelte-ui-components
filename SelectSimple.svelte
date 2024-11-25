<script lang="ts">
  import ListBox from './ListBox.svelte'
  import { onMount } from 'svelte'
  import { fly } from 'svelte/transition'
  import { quintOut } from 'svelte/easing'

  // these are internal / closed source
  import { getPopupStore } from ''
  import { arraysEqual } from ''
  import { selectArrowSVG } from ''

  const popup = getPopupStore()

  interface Props {
    options?: { value: any; name: string; filterValue?: any }[]
    value: any
    label?: string
    placeholder?: string
    disabled?: boolean
    listHeight?: string
    listWidth?: string
    padding?: string
    textSize?: 'text-sm' | 'text-md' | 'text-lg'
    labelSize?: string
    background?: string
    border?: string
    chipColour?: string
    name: string
    type?: 'single' | 'multiple'
    outputLayout?: 'chip' | 'text' | 'hidden'
    width?: string
    unselect?: boolean
    compulsory?: boolean
    dynamicFetch?: {
      getDynamically: boolean
      dynamicAPI: string
      prefetch: boolean
      extractDynamicOptionsFunction: (data: any) => typeof options
    }
    filtering?: {
      enabled: boolean
      filters: {
        name: string
        value: string | number | boolean
        checked: boolean
      }[]
    }
    hasSearchbar?: boolean
    minListHeight?: string
    maxListHeight?: string
    error?: string
    onchange?: (value: any) => any
  }

  let {
    options = $bindable([]),
    value = $bindable(),
    label = '',
    placeholder = 'Choose',
    disabled = false,
    listHeight = 'max-h-60',
    listWidth = 'w-full',
    padding = 'p-1',
    textSize = 'text-sm',
    labelSize = 'text-sm',
    background = 'bg-white text-black dark:bg-surface-800 dark:text-white',
    border = 'border border-gray-400 dark:border-surface-500',
    chipColour = 'bg-tertiary-500 text-white',
    name,
    type = 'multiple',
    outputLayout = 'chip',
    width = 'w-full',
    unselect = false,
    compulsory = false,
    dynamicFetch = {
      getDynamically: false,
      dynamicAPI: '',
      prefetch: false,
      extractDynamicOptionsFunction: () => []
    },
    filtering = $bindable({
      enabled: false,
      filters: []
    }),
    hasSearchbar = false,
    minListHeight = '',
    maxListHeight = '',
    error = '',
    onchange = () => {}
  }: Props = $props()

  let stopProp = $derived(type === 'multiple')

  let filteredOptions: typeof options = []
  let searchAndFilterApplied: typeof options = $state([])
  let dynamicFetched: boolean = $state(false)

  let searchValue: string = $state('')

  let listOut: boolean = $state(false)
  let selectedField: HTMLDivElement = $state() as HTMLDivElement
  let listBoxParent: HTMLDivElement = $state() as HTMLDivElement
  // this is for reactivity, when we have an await function for dynamic fetching
  // the listbox is initialised with 0 height
  let listBoxChild: HTMLDivElement = $state() as HTMLDivElement

  let labelClasses = $derived(disabled ? 'opacity-50' : '')
  let divClasses = $derived(
    `${background} ${disabled ? 'opacity-50 cursor-not-allowed' : 'cursor-pointer'}`
  )
  
  function customTransition(node: HTMLSpanElement) {
    return {
      duration: 300,
      easing: quintOut,
      css: (t: number) => `
                transform: scale(${t});
                opacity: ${t}
            `
    }
  }

  function keyNavigation(e: KeyboardEvent & { currentTarget: EventTarget & HTMLDivElement }) {
    if (e.key === 'Escape') {
      listOut = false
    } else if (e.key === 'Enter' || e.key === ' ') {
      listOut = !listOut
    } else if (e.key === 'ArrowUp') {
      e.preventDefault()
      if (type === 'single') {
        if ((value === undefined || Array.isArray(value)) && options.length > 0) {
          value = options[options.length - 1].value
        } else if (value !== undefined && options.length > 0) {
          let index = options.findIndex((o) => o.value === value)
          if (index === 0) value = options[options.length - 1].value
          else value = options[index - 1].value
        }
        onchange(value)
      }
    } else if (e.key === 'ArrowDown') {
      e.preventDefault()
      if (type === 'single') {
        if (value === undefined && options.length > 0) value = options[0].value
        else if (value !== undefined && options.length > 0) {
          let index = options.findIndex((o) => o.value === value)
          if (index === options.length - 1) value = options[0].value
          else value = options[index + 1].value
        }
        onchange(value)
      }
    } else if (e.key === 'Tab') {
      listOut = false
    }
  }

  const handleClick = (event: any) => {
    if (!event.target.closest(`.${name}`)) {
      listOut = false
    }
  }

  onMount(() => {
    if (filtering.enabled) {
      filtering.filters.push({ name: 'ALL', value: '', checked: true })
    }
    if (!dynamicFetch.getDynamically) {
      filteredOptions = options
    }
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

  function calculateListBox() {
    if (
      listBoxParent &&
      listBoxParent.getBoundingClientRect().bottom + 10 > window.innerHeight &&
      listBoxParent.getBoundingClientRect().top -
        listBoxParent.offsetHeight -
        selectedField.offsetHeight +
        10 >
        0
    ) {
      listBoxParent.style.top =
        label && label.length > 0
          ? `${-listBoxParent.offsetHeight + 20}px`
          : `${-listBoxParent.offsetHeight - 10}px`
    }
  }

  function checkValuesAreOfOptions() {
    if (Array.isArray(value)) {
      if (type === 'multiple' && value.some((v) => !options.some((o) => o.value === v))) {
        value = undefined
      } else if (type === 'single' && !options.some((o) => arraysEqual(o.value, value))) {
        value = undefined
      }
    } else if (
      !options.some((o) => o.value === value || (o.value !== null && typeof o.value === 'object'))
    ) {
      value = undefined
    }

    filteredOptions = options
    searchFilter()
  }

  async function getDynamicOptions() {
    if (dynamicFetched) return options

    dynamicFetched = true
    return fetch(dynamicFetch.dynamicAPI)
      .then((res) => res.json())
      .then((body) => {
        if (!body.success) {
          $popup(body.error, 'error')
        } else {
          options = dynamicFetch.extractDynamicOptionsFunction(body.data)
          // filteredOptions = options
          // searchAndFilterApplied = options
        }
      })
      .catch((err) => $popup(err.message, 'error'))
  }

  function applyFilter(latestFilter: {
    name: string
    value: string | number | boolean
    checked: boolean
  }) {
    let allFilter = filtering.filters[filtering.filters.length - 1]
    if (latestFilter.name === 'ALL' && latestFilter.checked) {
      filtering.filters = filtering.filters.map((f) => {
        f.checked = false
        return f
      })
      allFilter.checked = true
    }
    let checkedAmount = filtering.filters.filter((f) => f.checked && f.name !== 'ALL').length
    allFilter.checked = checkedAmount == 0
    filtering = filtering

    if (allFilter.checked) {
      filteredOptions = options
    } else if (!latestFilter.checked) {
      filteredOptions = filteredOptions.filter((fo) => fo.filterValue !== latestFilter.value)
    } else {
      filteredOptions = options.filter((o) =>
        filtering.filters.filter((f) => f.checked).some((f) => f.value === o.filterValue)
      )
    }

    searchFilter()
  }

  function searchFilter() {
    if (hasSearchbar)
      searchAndFilterApplied = filteredOptions.filter((o) =>
        o.name.toLowerCase().includes(searchValue.toLowerCase())
      )
    else searchAndFilterApplied = options
  }

  $effect(() => {
    listBoxParent && calculateListBox()
  })
  $effect(() => {
    listBoxChild && calculateListBox()
  })
  $effect(() => {
    options && checkValuesAreOfOptions()
  })

  let spanInputElement: HTMLSpanElement = $state() as HTMLSpanElement

  function giveInputFocus() {
    if (hasSearchbar) {
      if (type === 'single' && value !== undefined) {
        spanInputElement?.blur()
        spanInputElement.textContent = ''
      } else {
        spanInputElement?.focus()
        var range = document.createRange()
        range.selectNodeContents(spanInputElement) // Select the entire contents of the div
        range.collapse(false) // collapse to end
        var sel = window.getSelection()
        sel?.removeAllRanges()
        sel?.addRange(range)
      }
    }
  }

  // Here is the reactivity for change on search value
  $effect(() => {
    if (!listOut) {
      spanInputElement?.blur()
      searchValue = ''
    } else {
      searchFilter()
    }
  })
</script>

<div class="relative {width}">
  {#if label}
    <label for="date" class="block mb-2 {labelClasses} {labelSize}">
      {label}
      {#if compulsory}
        <span class="text-red-500">*</span>
      {/if}
    </label>
  {/if}
  <!-- svelte-ignore a11y_click_events_have_key_events -->
  <!-- svelte-ignore a11y_no_static_element_interactions -->
  <!-- svelte-ignore a11y_no_noninteractive_tabindex -->
  <div
    tabindex={disabled ? -1 : 0}
    bind:this={selectedField}
    onclick={() => {
      giveInputFocus()
      disabled ? '' : (listOut = !listOut)
    }}
    onkeydown={keyNavigation}
    onmouseenter={() => {
      if (dynamicFetch.getDynamically && dynamicFetch.prefetch && !dynamicFetched)
        getDynamicOptions()
    }}
    class="{name} min-h-[50px] {width} {border} flex flex-row rounded-lg {divClasses} p-2 py-3.5 items-center"
  >
    <span
      class="ml-2 {textSize} text-black dark:text-white {disabled
        ? 'opacity-50'
        : ''} flex flex-row flex-wrap gap-1 items-center"
    >
      {#if value === undefined || (Array.isArray(value) && value.length === 0)}
        {#if !hasSearchbar}
          <span class="text-gray-500 dark:text-gray-400 {textSize}">
            {placeholder}
          </span>
        {/if}
      {:else if type === 'multiple'}
        {#if outputLayout === 'chip' && Array.isArray(value)}
          {#each value.toSorted((a, b) => searchAndFilterApplied.findIndex((o) => o.value === a) - searchAndFilterApplied.findIndex((o) => o.value === b)) as v}
            <span
              transition:customTransition
              class="mr-1 px-2 py-0.5 rounded-lg {textSize} {chipColour}"
              >{searchAndFilterApplied.find((o) => o.value === v)?.name || v.toString()}</span
            >
          {/each}
        {:else if outputLayout === 'text' && Array.isArray(value)}
          {#each value.map((v) => options.find((o) => o.value === v)?.name || 'Unknown') as v, i}
            <span>{i === value.length - 1 ? v : v.concat(',')}</span>
          {/each}
          <!-- {value.map((v) => options.find(o => o.value === v)?.name || 'Unknown').join(', ')} -->

          {#if typeof document !== 'undefined' && (document.activeElement === spanInputElement || searchValue.length > 0)}
            <span class="-translate-x-1">,</span>
          {/if}
        {:else if outputLayout === 'hidden' && Array.isArray(value)}
          <span>Selected Items...</span>
        {/if}
      {:else if type === 'single'}
        {#if Array.isArray(value) && value.length > 0}
          {options.find((o) => arraysEqual(o.value, value))?.name || value?.toString()}
        {:else}
          {options.find((o) => o.value === value)?.name || value?.toString()}
        {/if}
      {/if}

      {#if hasSearchbar}
        <span
          contenteditable="true"
          bind:this={spanInputElement}
          bind:textContent={searchValue}
          onkeydown={(e) => {
            e.stopPropagation()
            if (e.key === 'Backspace' && searchValue === '') {
              if (Array.isArray(value) && value.length > 0) {
                value = value.slice(0, value.length - 1)
              }
            }
          }}
          class="{(Array.isArray(value) && value.length === 0) || value === undefined
            ? 'spaninput'
            : ''} border-0 focus:outline-none focus:ring-0"
          style="word-wrap: break-word;"
          {placeholder}
        ></span>
      {/if}
    </span>
    <div class="absolute right-3 w-6 h-6 {listOut ? 'rotate-180' : ''} duration-200">
      {@html selectArrowSVG}
    </div>
  </div>

  {#if listOut}
    <!-- svelte-ignore a11y_click_events_have_key_events -->
    <!-- svelte-ignore a11y_no_static_element_interactions -->
    <div
      bind:this={listBoxParent}
      class="{minListHeight} {maxListHeight} rounded-lg absolute inline-block w-full z-50 bg-surface-200 dark:bg-surface-700 mt-1"
      in:fly={{ y: -10, duration: 200 }}
      out:fly={{ y: -10, duration: 200 }}
      onclick={(e) => {
        stopProp && e.stopPropagation()
      }}
    >
      {#if dynamicFetch.getDynamically && dynamicFetch.dynamicAPI.length > 0}
        {#await getDynamicOptions() then _}
          {#if filtering.enabled && filtering.filters.length > 0}
            <div
              bind:this={listBoxChild}
              onclick={(e) => {
                e.stopPropagation()
                giveInputFocus()
              }}
              class="flex flex-row flex-wrap px-3 pt-2 gap-2"
            >
              {#each filtering.filters as filter}
                <div class="flex items-center mb-4">
                  <input
                    id={filter.name}
                    bind:checked={filter.checked}
                    onchange={() => applyFilter(filter)}
                    type="checkbox"
                    class="w-4 h-4 {filter.checked
                      ? ''
                      : 'dark:bg-gray-700'} text-blue-600 bg-gray-100 border-gray-300 rounded focus:ring-blue-500 dark:focus:ring-blue-600 dark:ring-offset-gray-800 focus:ring-2 dark:text-blue-600 dark:border-gray-600"
                  />
                  <label
                    for={filter.name}
                    class="ms-1 text-md font-medium text-gray-900 dark:text-gray-300"
                  >
                    {filter.name}
                  </label>
                </div>
              {/each}
              <div class="w-full border-b border-gray-500"></div>
            </div>
          {/if}
          <ListBox
            {background}
            focusInput={giveInputFocus}
            {unselect}
            {type}
            options={searchAndFilterApplied}
            bind:value
            height={listHeight}
            width={listWidth}
            {padding}
            {textSize}
            {onchange}
          />
        {/await}
      {:else}
        {#if filtering.enabled && filtering.filters.length > 0}
          <div
            bind:this={listBoxChild}
            onclick={(e) => {
              e.stopPropagation()
              giveInputFocus()
            }}
            class="flex flex-row flex-wrap px-3 pt-2 gap-2"
          >
            {#each filtering.filters as filter}
              <div class="flex items-center mb-4">
                <input
                  id={filter.name}
                  bind:checked={filter.checked}
                  onchange={() => applyFilter(filter)}
                  type="checkbox"
                  class="w-4 h-4 {filter.checked
                    ? ''
                    : 'dark:bg-gray-700'} text-blue-600 bg-gray-100 border-gray-300 rounded-lg focus:ring-blue-500 dark:focus:ring-blue-600 dark:ring-offset-gray-800 focus:ring-2 dark:text-blue-600 dark:border-gray-600"
                />
                <label
                  for={filter.name}
                  class="ms-1 text-md font-medium text-gray-900 dark:text-gray-300"
                >
                  {filter.name}
                </label>
              </div>
            {/each}
            <div class="w-full border-b border-gray-500"></div>
          </div>
        {/if}
        <ListBox
          stopPropagation={stopProp}
          {unselect}
          {type}
          options={searchAndFilterApplied}
          bind:value
          height={listHeight}
          width={listWidth}
          {padding}
          {textSize}
          {onchange}
        />
      {/if}
    </div>
  {/if}

  {#if error && error.length > 0}
    <span class="text-error-500 mb-2">{error}</span>
  {/if}
</div>

<style>
  .spaninput:empty:before {
    content: attr(placeholder);
    color: #a0a0a0;
    pointer-events: none;
    display: block; /* For Firefox */
  }
</style>
