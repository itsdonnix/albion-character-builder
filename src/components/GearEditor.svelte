<script>
  import { createEventDispatcher } from "svelte";
  import GearItem from "./GearItem.svelte";

  const dispatch = createEventDispatcher();

  export let gearName = "";

  const itemTypes = [
    "head",
    "armor",
    "mainhand",
    "offhand",
    "bag",
    "cape",
    "potion",
    "food",
    "shoes",
    // "mount",
  ];

  function onClickGearType({ detail }) {
    dispatch("select-item", { ...detail, gearName });
  }
</script>

<div class="gear-editor w-full">
  <!-- GEAR NAME FIELD -->
  <div class="flex w-full sticky top-0" style="z-index:2">
    <input
      aria-label="Gear name"
      class="w-full outline-none bg-gray-100 mx-1 p-2"
      type="text"
      bind:value={gearName}
      placeholder="Type a name for this gear" />
  </div>
  <div class="item-list__body item-list__body--gear">
    {#each itemTypes as itemType}
      <GearItem {itemType} on:click={onClickGearType} />
    {/each}
  </div>
  <div class="gear-stats py-3 ml-1">
    <ul class="stats-lists">
      <li><span>Damage:</span> <span>10</span></li>
      <li><span>Physical Damage:</span> <span>10</span></li>
      <li><span>Attack Speed:</span> <span>10/s</span></li>
    </ul>
  </div>
</div>
