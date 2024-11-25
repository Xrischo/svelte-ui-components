<script lang="ts">
  import { ProgressRadial } from '@skeletonlabs/skeleton'
  import { quintOut } from 'svelte/easing'
  import { scale } from 'svelte/transition'

  interface Props {
    visible?: boolean
  }

  let { visible = $bindable(true) }: Props = $props()
</script>

<div
  transition:scale={{ duration: 500, opacity: 0.5, start: 0.5, easing: quintOut }}
  onoutrostart={() => (visible = false)}
  onoutroend={() => (visible = true)}
>
  <ProgressRadial width="w-8" stroke={100} strokeLinecap="round" />
</div>
