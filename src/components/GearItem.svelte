<script>
  import { createEventDispatcher } from "svelte";

  const dispatch = createEventDispatcher();

  export let itemType;

  function onClick() {
    dispatch("click", { itemType });
  }
</script>

<button
  type="button"
  title="Select item for {itemType} gear"
  aria-label="Select item for {itemType} gear"
  class="item item--{itemType} cursor"
  on:click={onClick} />
