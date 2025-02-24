<script lang="ts">
  import { tweened } from "svelte/motion";
  import { cubicOut } from "svelte/easing";
  export let display = "inline-flex";
  export let value: number;
  export let accuracy: "none" | "value" | "ticks" = "value";
  export let disabled = false;

  let rangeElem: HTMLInputElement, range: number, percent: number;
  $: {
    const max = Number(rangeElem?.max || 100);
    const min = Number(rangeElem?.min || 0);
    range = max - min;
    percent = ($valueDisplayed - min) / range;
  }
  const valueDisplayed = tweened(value || 0, { duration: 150, easing: cubicOut });
  const updateValue = (e: any) => {
    value = Number(e.target.value);
    e.target.value = $valueDisplayed;
    $valueDisplayed = value;
  };
  $: $valueDisplayed = value;
  const generateTicks = (size: number, step: number) =>
    Array.from({ length: size / step + 1 }, (_, i) => i * (step / size) * 100);
</script>

<div
  class="m3-container"
  class:show-value={accuracy != "none"}
  style="display: {display}; --percent: {percent * 100}%;"
>
  <input
    type="range"
    bind:this={rangeElem}
    value={$valueDisplayed}
    {disabled}
    on:input={updateValue}
    {...$$props}
  />
  <div class="track" />
  {#if accuracy == "ticks"}
    {#each generateTicks(range, $$props.step || 1) as tick}
      <div
        class="tick"
        class:active={tick / 100 < value / range}
        style="left: calc({tick}% + {tick * (-4 / 100) + 1}px);"
      />
    {/each}
  {/if}
  <div class="thumb" />
  <div class="layer" />
  <div class="value"><span class="label-medium">{value}</span></div>
</div>

<style>
  .m3-container {
    position: relative;
    align-items: center;
    height: 2.5rem;
    min-width: 10rem;
  }
  input {
    position: absolute;
    height: 100%;
    width: calc(100% + 1rem);
    left: -0.5rem;
    opacity: 0;
    margin: 0;
    appearance: none;
    cursor: pointer;
  }
  .track {
    height: 0.25rem;
    width: 100%;
    border-radius: 0.25rem;
    background-image: linear-gradient(
      to right,
      rgb(var(--md-sys-color-primary)) 0%,
      rgb(var(--md-sys-color-primary)) var(--percent),
      rgb(var(--md-sys-color-surface-variant)) var(--percent),
      rgb(var(--md-sys-color-surface-variant)) 100%
    );
    pointer-events: none;
  }
  .thumb {
    position: absolute;
    left: calc(var(--percent) - 0.625rem);
    width: 1.25rem;
    height: 1.25rem;
    border-radius: 1.25rem;
    background-color: rgb(var(--md-sys-color-primary));
    box-shadow: var(--md-sys-elevation-1);
    pointer-events: none;
  }
  .layer {
    position: absolute;
    left: calc(var(--percent) - 1.25rem);
    width: 2.5rem;
    height: 2.5rem;
    border-radius: 1.25rem;
    pointer-events: none;
    transition: background-color 150ms;
  }
  .value {
    position: absolute;
    left: calc(var(--percent) - 0.875rem);
    top: -1.75rem;
    width: 1.75rem;
    height: 1.75rem;
    display: inline-flex;
    justify-content: center;
    align-items: center;
    border-radius: 1.75rem 1.75rem 0 1.75rem;
    transform: rotate(45deg);
    background-color: rgb(var(--md-sys-color-primary));
    color: rgb(var(--md-sys-color-on-primary));
    opacity: 0;
    transition: opacity 150ms;
    pointer-events: none;
  }
  .value > span {
    display: inline-block;
    transform: rotate(-45deg);
  }
  .tick {
    position: absolute;
    width: 2px;
    height: 2px;
    border-radius: 2px;
    top: 50%;
    transform: translate(0, -50%);
    background-color: rgb(var(--md-sys-color-on-surface-variant) / 0.38);
  }
  .tick.active {
    background-color: rgb(var(--md-sys-color-on-primary) / 0.38);
  }
  input:hover ~ .layer {
    background-color: rgb(var(--md-sys-color-primary) / 0.08);
  }
  input:focus-visible ~ .layer,
  input:active ~ .layer {
    background-color: rgb(var(--md-sys-color-primary) / 0.12);
  }
  .show-value > input:hover ~ .value,
  .show-value > input:focus-visible ~ .value,
  .show-value > input:active ~ .value {
    opacity: 1;
  }

  input:disabled {
    cursor: auto;
  }
  input:disabled + .track {
    background-image: linear-gradient(
      to right,
      rgb(var(--md-sys-color-on-surface) / 0.38) 0%,
      rgb(var(--md-sys-color-on-surface) / 0.38) var(--percent),
      rgb(var(--md-sys-color-on-surface) / 0.12) var(--percent),
      rgb(var(--md-sys-color-on-surface) / 0.12) 100%
    );
  }
  input:disabled ~ .thumb {
    background-color: rgb(var(--md-sys-color-on-surface) / 0.38);
    box-shadow: none;
  }
  input:disabled ~ .layer {
    display: none;
  }
  input:disabled ~ .value {
    display: none;
  }
</style>
