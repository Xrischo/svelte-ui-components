<script lang="ts">
  interface Props {
    isAlpha: boolean
    rgb: { r: number; g: number; b: number; a: number }
    hsv: { h: number; s: number; v: number; a: number }
    hex: string
    textInputModes: any
    texts: any
    oninput: (colour: any) => void
  }

  let {
    isAlpha,
    rgb = $bindable(),
    hsv = $bindable(),
    hex = $bindable(),
    textInputModes,
    texts,
    oninput
  }: Props = $props()
  const HEX_COLOR_REGEX = /^#?([A-F0-9]{6}|[A-F0-9]{8})$/i
  const modes = ['HEX', 'RGB', 'HSV']
  let mode = 0
  let h = $derived(Math.round(hsv.h))
  let s = $derived(Math.round(hsv.s))
  let v = $derived(Math.round(hsv.v))
  let a = $derived(hsv.a === undefined ? 1 : Math.round(hsv.a * 100) / 100)
  function updateHex(e: { target: any }) {
    const target = e.target
    if (HEX_COLOR_REGEX.test(target.value)) {
      hex = target.value
      oninput(hex)
    }
  }
  function updateRgb(property: string) {
    return function (e: Event & { target: any }) {
      rgb = { ...rgb, [property]: parseFloat(e.target.value) }
      oninput(rgb)
    }
  }
  function updateHsv(property: string) {
    return function (e: Event & { target: any }) {
      hsv = { ...hsv, [property]: parseFloat(e.target.value) }
      oninput(hsv)
    }
  }
</script>

<div class="text-input">
  {#if mode === 0}
    <div class="input-container">
      <input
        class="bg-surface-300 dark:bg-surface-400"
        value={hex}
        oninput={updateHex}
        style="flex: 3"
      />
      {#if isAlpha}
        <input
          class="bg-surface-300 dark:bg-surface-400"
          aria-label="hexadecimal color"
          value={a}
          type="number"
          min="0"
          max="1"
          step="0.01"
          oninput={updateRgb('a')}
        />
      {/if}
    </div>
  {:else if mode === 1}
    <div class="input-container">
      <input
        aria-label="red chanel color"
        value={rgb.r}
        type="number"
        min="0"
        max="255"
        oninput={updateRgb('r')}
      />
      <input
        aria-label="green chanel color"
        value={rgb.g}
        type="number"
        min="0"
        max="255"
        oninput={updateRgb('g')}
      />
      <input
        aria-label="blue chanel color"
        value={rgb.b}
        type="number"
        min="0"
        max="255"
        oninput={updateRgb('b')}
      />
      {#if isAlpha}
        <input
          aria-label="transparency chanel color"
          value={a}
          type="number"
          min="0"
          max="1"
          step="0.01"
          oninput={updateRgb('a')}
        />
      {/if}
    </div>
  {:else}
    <div class="input-container">
      <input
        aria-label="hue chanel color"
        value={h}
        type="number"
        min="0"
        max="360"
        oninput={updateHsv('h')}
      />
      <input
        aria-label="saturation chanel color"
        value={s}
        type="number"
        min="0"
        max="100"
        oninput={updateHsv('s')}
      />
      <input
        aria-label="brightness chanel color"
        value={v}
        type="number"
        min="0"
        max="100"
        oninput={updateHsv('v')}
      />
      {#if isAlpha}
        <input
          aria-label="transparency chanel color"
          value={a}
          type="number"
          min="0"
          max="1"
          step="0.01"
          oninput={updateHsv('a')}
        />
      {/if}
    </div>
  {/if}
</div>

<style>
  .text-input {
    display: flex;
    flex-direction: column;
    gap: 10px;
    margin: 10px 5px 5px;
  }
  .input-container {
    display: flex;
    flex: 1;
    gap: 10px;
  }
  input,
  button {
    flex: 1;
    border: none;
    padding: 0;
    border-radius: 5px;
    height: 30px;
    line-height: 30px;
    text-align: center;
  }
  input {
    width: 5px;
  }

  button {
    cursor: pointer;
    flex: 1;
    margin: 0;
    transition: background-color 0.2s;
  }

  input:focus,
  button:focus {
    outline: none;
  }

  input:focus-visible,
  button:focus-visible {
    outline: 2px solid var(--focus-color, rgb(205, 205, 205));
    outline-offset: 2px;
  }

  .dark input:focus-visible,
  .dark button:focus-visible {
    outline: 2px solid var(--focus-color, rgb(100, 100, 100));
    outline-offset: 2px;
  }
</style>
