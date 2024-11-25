<script lang="ts">
  interface Props {
    value: string
    placeholder: string
    background?: string
    width?: string
    initialHeight?: string
    disabled?: boolean
    applyDisabledClasses?: boolean
    label?: string
    error?: string
    rounded?: string
    resize?: boolean
    textSize?: 'text-sm' | 'text-md' | 'text-lg'
    labelSize?: string
    compulsory?: boolean
  }

  let {
    value = $bindable(),
    placeholder,
    background = 'bg-white text-black dark:bg-surface-700 dark:text-white border border-surface-300 dark:border-gray-600 focus:border-black dark:focus:border-gray-400',
    width = 'w-full',
    initialHeight = 'h-full',
    disabled = false,
    applyDisabledClasses = true,
    label = '',
    error = '',
    rounded = 'rounded-xl',
    resize = true,
    textSize = 'text-sm',
    labelSize = 'text-sm',
    compulsory = false
  }: Props = $props()

  let labelClasses = $derived(
    disabled && applyDisabledClasses ? 'text-gray-400' : 'text-black dark:text-white'
  )
</script>

<div class="relative hover:cursor-text {width} flex flex-col">
  {#if label}
    <label for={label} class="block mb-2 {labelClasses} {labelSize}">
      {label}
      {#if compulsory}
        <span class="text-red-500">*</span>
      {/if}
    </label>
  {/if}

  <textarea
    bind:value
    {placeholder}
    class="{background} {rounded} w-full focus:outline-none focus:ring-0 {textSize} {initialHeight} {disabled &&
    applyDisabledClasses
      ? 'opacity-50'
      : 'text-black border-gray-400'}"
    style={resize ? '' : 'resize: none'}
  ></textarea>

  {#if error && error.length > 0}
    <span class="text-error-500">{error}</span>
  {/if}
</div>
