<!-- For very specific use case when the options are not just true and false -->
<script lang="ts">
  interface Props {
    option1: { name: string; value: string | boolean }
    option2: { name: string; value: string | boolean }
    error?: string
    value: string | boolean | unknown
    optionWidth?: string
    optionHeight?: string
    addSkew?: boolean
    label?: string
  }

  let {
    option1,
    option2,
    error = '',
    value = $bindable(),
    optionWidth = 'w-32',
    optionHeight = 'h-8',
    addSkew = false,
    label = ''
  }: Props = $props()

  let selectedClass = 'bg-primary-600 text-white hover:bg-primary-500'
  let unselectedClass =
    'bg-surface-200 text-black dark:bg-surface-700 dark:text-white hover:bg-surface-300 dark:hover:bg-surface-600'
</script>

<div>
  {#if label}
    <div class="mb-2">
      <span class="text-sm">{label}</span>
    </div>
  {/if}

  <div class="flex flex-row items-center relative rounded-md">
    <!-- svelte-ignore a11y_click_events_have_key_events -->
    <!-- svelte-ignore a11y_no_static_element_interactions -->
    <div
      class="{addSkew
        ? 'opt1'
        : ''} border-l border-t border-b border-surface-300 dark:border-surface-600 flex items-center justify-center relative cursor-pointer {optionWidth} {optionHeight} px-2 rounded-s-md {value ===
      option1.value
        ? selectedClass
        : unselectedClass} duration-100"
      onclick={() => (value = option1.value)}
    >
      {option1.name}
    </div>
    <!-- svelte-ignore a11y_click_events_have_key_events -->
    <!-- svelte-ignore a11y_no_static_element_interactions -->
    <div
      class="flex items-center justify-center cursor-pointer border-r border-t border-b border-surface-300 dark:border-surface-600 {optionWidth} {optionHeight} pl-3 pr-2 rounded-e-md {value ===
      option2.value
        ? selectedClass
        : unselectedClass} duration-100"
      onclick={() => (value = option2.value)}
    >
      {option2.name}
    </div>
  </div>

  {#if error && error.length > 0}
    <span class="text-error-500">{error}</span>
  {/if}
</div>

<style>
  .opt1::after {
    position: absolute;
    content: '';
    right: -3px;
    height: 2rem;
    width: 10px;
    transform: skewX(-10deg);
    background: inherit;
  }
</style>
