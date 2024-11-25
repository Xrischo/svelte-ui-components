<script lang="ts">
  import { onMount } from 'svelte'

  interface Props {
    page?: number
    numberOfPages?: number
    unselectedPageBg?: string
    selectedPageBg?: string
    navButtonClasses?: string
    pageButtonWidth?: number
    pageButtonHeight?: number
    pageButtonGap?: number
    rounded?: string
    textSize?: string
    onpageChange?: (page: number) => void
  }

  let {
    page = $bindable(1),
    numberOfPages = 1,
    unselectedPageBg = 'bg-white hover:bg-gray-100 dark:hover:bg-gray-700 dark:bg-gray-800 text-black dark:text-white border border-gray-300 dark:border-gray-600',
    selectedPageBg = 'bg-black dark:bg-white text-white dark:text-black',
    navButtonClasses = 'bg-white dark:bg-gray-800 text-black dark:text-white border border-gray-300 dark:border-gray-600 px-3 text-sm min-w-fit rounded-md',
    pageButtonWidth = 32,
    pageButtonHeight = 32,
    pageButtonGap = 8,
    rounded = 'rounded-md',
    textSize = 'text-sm',
    onpageChange = (page: number) => {}
  }: Props = $props()

  const overflowDotsWidth = 48

  let buttonsDiv: HTMLDivElement = $state() as HTMLDivElement

  let buttonsOverflow: boolean = $state() as boolean
  let numberOfButtons: number = $state() as number

  function calculateButtonLayout() {
    // if the available width is less than the number of buttons with the gap between them
    buttonsOverflow =
      buttonsDiv.offsetWidth < numberOfPages * pageButtonWidth + (numberOfPages - 1) * pageButtonGap

    if (buttonsOverflow) {
      numberOfButtons = Math.max(
        0,
        Math.floor(
          (buttonsDiv.offsetWidth - overflowDotsWidth) / (pageButtonWidth + pageButtonGap) / 2
        )
      )
    } else {
      numberOfButtons = numberOfPages
    }
  }

  onMount(() => {
    calculateButtonLayout()
  })

  let selectedPageClasses = $derived(`${selectedPageBg} ${rounded} ${textSize}`)
  let unselectedPageClasses = $derived(`${unselectedPageBg} ${rounded} ${textSize}`)
</script>

<div class="flex flex-row justify-between w-full">
  <button
    onclick={() => {
      page > 1 ? page-- : undefined
      onpageChange(page)
    }}
    disabled={page === 1}
    class="{navButtonClasses} {page === 1
      ? 'opacity-50'
      : 'hover:bg-gray-100 dark:hover:bg-gray-700'}"
  >
    {'<'} Previous
  </button>

  <div class="w-full mx-2" bind:this={buttonsDiv}>
    <div class="flex flex-row gap-2 justify-center">
      {#if buttonsOverflow && numberOfButtons}
        {#each Array(numberOfButtons) as _, i}
          <button
            class={page === i + 1 ? selectedPageClasses : unselectedPageClasses}
            style="width: {pageButtonWidth}px; height: {pageButtonHeight}px;"
            onclick={() => {
              page = i + 1
              onpageChange(page)
            }}
          >
            {i + 1}
          </button>
        {/each}
        <div>...</div>
        {#each Array(numberOfButtons) as _, i}
          <button
            class={page === numberOfPages - numberOfButtons + i + 1
              ? selectedPageClasses
              : unselectedPageClasses}
            style="width: {pageButtonWidth}px; height: {pageButtonHeight}px;"
            onclick={() => {
              page = numberOfPages - numberOfButtons + i + 1
              onpageChange(page)
            }}
          >
            {numberOfPages - numberOfButtons + i + 1}
          </button>
        {/each}
      {:else}
        {#each Array(numberOfButtons) as _, i}
          <button
            class={page === i + 1 ? selectedPageClasses : unselectedPageClasses}
            style="width: {pageButtonWidth}px; height: {pageButtonHeight}px;"
            onclick={() => {
              page = i + 1
              onpageChange(page)
            }}
          >
            {i + 1}
          </button>
        {/each}
      {/if}
    </div>
  </div>

  <button
    onclick={() => {
      page < numberOfPages ? page++ : undefined
      onpageChange(page)
    }}
    disabled={page === numberOfPages}
    class="{navButtonClasses} {page === numberOfPages
      ? 'opacity-50'
      : 'hover:bg-gray-100 dark:hover:bg-gray-800'}"
  >
    Next {'>'}
  </button>
</div>
