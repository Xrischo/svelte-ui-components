<script lang="ts">
  interface Props {
    error: {
      code: string
    }
  }
  let { error }: Props = $props();
</script>

<div class="w-full h-full flex flex-col items-center mt-20">
    <div class="w-1/2">
      <h1 class="text-7xl font-bold text-red-500">Oops!</h1>
      <p class="text-3xl font-bold mt-10">Something went wrong</p>
      <p class="text-lg mt-10">Please let us know about this error code:</p>
      <p class="text-2xl font-bold text-red-500">{error.code}</p>
    </div>
  </div>