<script>
  import * as axios from "axios";
  import Item from "./Item.svelte";

  const httpx = axios.default;

  class ItemData {
    constructor() {
      this.items = [];
      this.lastModifiedDate = "";
      this.filteredItems = [];
    }
  }

  export let itemType,
    searchKeyword,
    //
    tiers = [],
    categories = [],
    categoriesIndex = [];

  let gear = {
      bag: new ItemData(),
      cape: new ItemData(),
      mainhand: new ItemData(),
      offhand: new ItemData(),
      head: new ItemData(),
      armor: new ItemData(),
      shoes: new ItemData(),
      food: new ItemData(),
    },
    currentGear = { items: [], filteredItems: [] },
    loading = false;

  export async function getItems() {
    currentGear = gear[itemType];
    const requestUrl = `/data/${itemType}.json`;
    loading = true;
    httpx.get(requestUrl).then(({ data, headers }) => {
      const _lastModifiedDate = headers["last-modified"];
      if (currentGear.lastModifiedDate !== _lastModifiedDate) {
        currentGear.items = currentGear.filteredItems = data;
        currentGear.lastModifiedDate = _lastModifiedDate;
      }
      filter();
      loading = false;
    });
  }

  function filter() {
    if (!!searchKeyword.length) {
      currentGear.filteredItems = currentGear.items.filter((item) =>
        item.name.toLowerCase().includes(searchKeyword)
      );
    } else {
      if (currentGear.filteredItems.length != currentGear.items.length) {
        currentGear.filteredItems = currentGear.items;
      }
    }

    let selectedTiers = [];
    tiers.forEach((checked, index) => {
      if (checked) selectedTiers.push(index + 1);
    });

    if (!!selectedTiers.length) {
      currentGear.filteredItems = currentGear.filteredItems.filter((item) =>
        selectedTiers.includes(item.tier)
      );
    }

    let selectedCategories = [];
    categoriesIndex.forEach((checked, index) => {
      if (checked) selectedCategories.push(categories[itemType][index]);
    });

    if (!!selectedCategories.length) {
      currentGear.filteredItems = currentGear.filteredItems.filter((item) =>
        selectedCategories.includes(item.type)
      );
    }
  }
</script>

<div class="item-lists flex flex-row">
  {#if !loading}
    {#each currentGear.filteredItems as item}
      <Item {item} />
    {/each}
  {/if}
</div>
