<script>
  import { onMount } from "svelte"
  import ripple from "../Common/Ripple.js"
  import ClassBuilder from "../ClassBuilder.js"

  const cb = new ClassBuilder("icon-button")

  let on = false

  export let _bb
  export let context = ""
  export let onClick = () => {}
  export let disabled = false
  export let href = ""
  export let icon = ""
  export let onIcon = "" //on state icon for toggle button
  export let size = "medium"

  onMount(() => {
    let ctx = !!_bb ? _bb.getContext("BBMD:icon-button:context") : ""
    //It isn't possible to use context within nested components as they do not have their own _bb instance (has to be passed down from parent component). This allows context to be passed as props
    if (!context && !!ctx) {
      context = ctx
    }
  })

  function onButtonClick() {
    open = !open
    _bb.call(onClick)
  }

  $: isToggleButton = !!icon && !!onIcon
  $: useLinkButton = !!href

  $: customs = { size }
  $: props = { customs, extras: ["material-icons", context] }
  $: iconBtnClass = cb.build({ props })
</script>

{#if useLinkButton}
  <a
    on:click={onButtonClick}
    class={iconBtnClass}
    {href}
    {disabled}
    role="button"
    tabindex="0">
    {#if isToggleButton}
      <i
        use:ripple
        class="material-icons mdc-icon-button__icon mdc-icon-button__icon--on">
        {onIcon}
      </i>
      <i use:ripple class="material-icons mdc-icon-button__icon">{icon}</i>
    {:else}{icon}{/if}
  </a>
{:else}
  <button
    on:click={onButtonClick}
    class={iconBtnClass}
    {disabled}
    role="button"
    aria-label="Add to favorites"
    aria-pressed="false"
    tabindex="0">
    {#if isToggleButton}
      <i use:ripple class="material-icons mdc-icon-button__icon">{icon}</i>
      <i
        use:ripple
        class="material-icons mdc-icon-button__icon mdc-icon-button__icon--on">
        {onIcon}
      </i>
    {:else}{icon}{/if}
  </button>
{/if}
