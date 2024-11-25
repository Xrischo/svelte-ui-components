<script lang="ts">
  interface Props {
    wrapper: any
    isOpen: boolean
    isDialog: boolean
    children?: import('svelte').Snippet
  }

  let { wrapper = $bindable(), isOpen, isDialog, children }: Props = $props()
</script>

<div
  class="wrapper flex flex-col bg-surface-300-600-token rounded p-1"
  bind:this={wrapper}
  class:is-open={isOpen}
  role={isDialog ? 'dialog' : undefined}
  aria-label="color picker"
>
  {@render children?.()}
</div>

<style>
  div {
    width: max-content;
  }
  [role='dialog'] {
    position: absolute;
    top: calc(var(--input-size, 25px) + 12px);
    left: 0;
    z-index: var(--picker-z-index, 2);
  }
</style>
