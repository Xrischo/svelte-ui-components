<script lang="ts">
  import { onMount } from 'svelte'
  import { quintOut } from 'svelte/easing'
  import { fly } from 'svelte/transition'

  interface Props {
    label: string
    labelClasses?: string
    options: { icon: string; name: string; action: () => void }[]
    dropdownWidht?: string
    dropdownHeight?: string
  }

  let {
    label,
    labelClasses = '',
    options,
    dropdownWidht = 'min-w-10',
    dropdownHeight = 'min-h-32'
  }: Props = $props()
  let menudown: boolean = $state(false)

  let spanElem: HTMLSpanElement = $state() as HTMLSpanElement
  let dropdownMenu: HTMLDivElement = $state() as HTMLDivElement

  function calculateMenu() {
    if (dropdownMenu && dropdownMenu.getBoundingClientRect().bottom + 10 > window.innerHeight) {
      dropdownMenu.style.top = `-${dropdownMenu.offsetHeight}px`
    }

    if (
      dropdownMenu &&
      wrapper &&
      spanElem &&
      dropdownMenu.getBoundingClientRect().right + 10 >
        (wrapper.parentNode as HTMLDivElement).getBoundingClientRect().right
    ) {
      dropdownMenu.style.left = `-${dropdownMenu.offsetWidth - spanElem.offsetWidth}px`
    }

    if (
      dropdownMenu &&
      wrapper &&
      spanElem &&
      dropdownMenu.getBoundingClientRect().left - dropdownMenu.offsetWidth - 10 <
        (wrapper.parentNode as HTMLDivElement).getBoundingClientRect().left
    ) {
      dropdownMenu.style.left = `${dropdownMenu.offsetWidth - spanElem.offsetWidth}px`
    }
  }

  const handleClick = (event: any) => {
    if (!event.target.closest(`.dropdown`)) {
      menudown = false
    }
  }

  onMount(() => {
    document.addEventListener('click', handleClick)
    return () => document.removeEventListener('click', handleClick)
  })

  let wrapper: HTMLDivElement = $state() as HTMLDivElement
  $effect(() => {
    dropdownMenu && calculateMenu()
  })
</script>

<div bind:this={wrapper} class="relative">
  <!-- svelte-ignore a11y_click_events_have_key_events -->
  <!-- svelte-ignore a11y_no_static_element_interactions -->
  <span
    bind:this={spanElem}
    class="dropdown flex items-start text-black dark:text-white cursor-pointer hover:scale-110 duration-200 rounded-md {labelClasses}"
    onclick={() => (menudown = !menudown)}
  >
    {label}
  </span>

  {#if menudown}
    <div
      bind:this={dropdownMenu}
      class="dropdown rounded-md absolute inline-block z-50 bg-surface-200 dark:bg-surface-700 mt-1 {dropdownWidht} {dropdownHeight}"
      transition:fly={{ y: -10, duration: 150, easing: quintOut }}
    >
      <div class="flex flex-col gap-2">
        {#each options as option}
          <!-- svelte-ignore a11y_click_events_have_key_events -->
          <!-- svelte-ignore a11y_no_static_element_interactions -->
          <span
            class="w-full flex flex-row cursor-pointer hover:bg-surface-300 dark:hover:bg-surface-600 duration-150 p-2 bg-surface-200 dark:bg-surface-700 rounded-md"
            onclick={() => {
              menudown = false
              option.action()
            }}
          >
            {#if option.icon}
              <div class="pr-2 mr-2 border-r border-gray-500 dark:border-gray-600">
                {@html option.icon}
              </div>
            {/if}
            {option.name}
          </span>
        {/each}
      </div>
    </div>
  {/if}
</div>
