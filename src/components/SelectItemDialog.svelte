<script>
  import { onMount, tick } from "svelte";
  import ItemList from "./ItemList.svelte";
  import ANIMATIONS from "../shared/animation.js";
  import GearObject from "../shared/GearObject.js";

  const KEYS = { TAB: 9, ESC: 27 };

  // List of components refferences
  let components = { ItemList: null };

  export let itemType = "mainhand",
    gearName;

  let // Dialog related variables
    dialogEl,
    overlayEl,
    focusableEls,
    firstFocusableEl,
    lastFocusableEl,
    focusedElBeforeOpen,
    visible = false,
    //
    searchFieldEl,
    searchKeyword = "",
    // The max tier of each item type
    tiersCount = {
      head: 7,
      armor: 7,
      mainhand: 8,
      offhand: 8,
      bag: 8,
      cape: 8,
      potion: 9,
      food: 10,
      shoes: 20,
    },
    tiersIndex = new GearObject(),
    categories = new GearObject(),
    categoriesIndex = new GearObject();

  // Immediately populate data's
  setupCategoriesData();
  setupTiersAndCateriesIndex();

  onMount(() => {
    const _focusableEls = dialogEl.querySelectorAll(
      "a[href], area[href], input:not([disabled]), select:not([disabled]), textarea:not([disabled]), button:not([disabled]), [tabindex='0']"
    );
    focusableEls = Array.from(_focusableEls);
    firstFocusableEl = focusableEls[0];
    lastFocusableEl = focusableEls[focusableEls.length - 1];
  });

  export async function open() {
    focusedElBeforeOpen = document.activeElement;
    document.body.style.overflow = "hidden";
    dialogEl.addEventListener("keydown", handleDialogKeyDown);
    document.addEventListener("keydown", handleDocumentKeyDown, { once: true });
    visible = true;
    const animation = dialogEl.animate(ANIMATIONS.FADE_IN.KEYFRAMES, {
      duration: 150,
      easing: ANIMATIONS.FADE_IN.EASING,
    });
    animation.onfinish = async function () {
      await tick();
      firstFocusableEl.focus();
      components.ItemList.getItems();
    };
  }

  function close() {
    const animation = dialogEl.animate(ANIMATIONS.FADE_IN.KEYFRAMES, {
      duration: 150,
      easing: ANIMATIONS.FADE_IN.EASING,
      direction: "reverse",
    });
    animation.onfinish = async function () {
      visible = false;
      if (focusedElBeforeOpen) {
        await tick();
        document.body.style.overflow = "auto";
        focusedElBeforeOpen.focus();
      }
    };
  }

  function setupTiersAndCateriesIndex() {
    // Populate categoriesIndex for using in checkbox input binding
    for (const _itemType of Object.keys(categories)) {
      for (let index = 0; index < categories[_itemType].length; index++) {
        categoriesIndex[_itemType][index] = false;
      }
    }
    // Populate tiersIndex for using in checkbox input binding
    for (const _itemType of Object.keys(tiersCount)) {
      for (let index = 0; index < tiersCount[_itemType]; index++) {
        tiersIndex[_itemType][index] = false;
      }
    }
  }

  function setupCategoriesData() {
    categories.mainhand = [
      "sword",
      "bow",
      "crossbow",
      "dagger",
      "axe",
      "staff",
      "spear",
    ];
    categories.offhand = ["shield"];
  }

  function clearSearchKeyword() {
    searchKeyword = "";
    components.ItemList.getItems();
    searchFieldEl.focus();
  }

  function handleDocumentKeyDown(e) {
    if (e.keyCode == KEYS.ESC) close();
  }

  function handleDialogKeyDown(e) {
    function handleBackwardTab() {
      if (document.activeElement === firstFocusableEl) {
        e.preventDefault();
        lastFocusableEl.focus();
      }
    }

    function handleForwardTab() {
      if (document.activeElement === lastFocusableEl) {
        e.preventDefault();
        firstFocusableEl.focus();
      }
    }

    switch (e.keyCode) {
      case KEYS.TAB:
        if (focusableEls.length === 1) {
          e.preventDefault();
          break;
        }
        if (e.shiftKey) {
          handleBackwardTab();
        } else {
          handleForwardTab();
        }
        break;
      case KEYS.ESC:
        close();
        break;
      default:
        break;
    }
  }

  function handleOverlayClick(e) {
    if (e.target == overlayEl) {
      close();
    }
  }
</script>

<div
  bind:this={overlayEl}
  on:click={handleOverlayClick}
  class:hidden={!visible}
  class="dialog-overlay p-4 xs:p-0"
  aria-hidden={!visible}>
  <!-- DIALOG  -->
  <div
    bind:this={dialogEl}
    class="dialog bg-white rounded-sm h-full flex flex-col z-10"
    role="dialog"
    aria-hidden={!visible}
    aria-labelledby="select-item-dialog--title">
    <!-- DIALOG TITLE -->
    <h3 class="p-3 text-xl font-medium" id="select-item-dialog--title">
      Select
      {itemType}
      item for
      {gearName}
    </h3>

    <hr />

    <!-- FIELD SEARCH ITEM -->
    <div class="relative w-full">
      <input
        bind:this={searchFieldEl}
        class="bg-gray-100 w-full outline-none p-2"
        type="text"
        on:input={components.ItemList.getItems}
        bind:value={searchKeyword}
        placeholder="Search item" />
      <button
        on:click={clearSearchKeyword}
        aria-hidden={!searchKeyword.length}
        class:hidden={!searchKeyword.length}
        aria-label="Clear keyword"
        class="absolute h-full right-0 px-3 bg-gray-200 text-sm">clear</button>
    </div>

    <hr />

    <!-- FILTER OPTIONS -->
    <div class="item-filter flex flex-col xs:flex-row w-full">
      <!-- TIERS -->
      <div class="flex flex-row m-2 items-center xs:flex-col xs:w-3/12 w-full">
        {#each tiersIndex[itemType] as tier, index}
          <div class="flex items-center xs:w-full">
            <input
              class="cursor-pointer block"
              type="checkbox"
              id="check--tier-{index + 1}--item"
              title="Tier {index + 1}"
              on:input={components.ItemList.getItems}
              bind:checked={tier} />
            <label
              class="ml-1 my-1 mr-3 select-none cursor-pointer text-sm"
              for="check--tier-{index + 1}--item">Tier
              {index + 1}</label>
          </div>
        {/each}
      </div>
      <!-- CATEGORIES -->
      <div class="flex flex-row m-2 xs:flex-col">
        {#each categoriesIndex[itemType] as selectedCategory, index}
          <div class="flex items-center xs:w-full">
            <input
              class="cursor-pointer block"
              type="checkbox"
              id="check--category-{categories[itemType][index]}--item"
              title={categories[itemType][index]}
              on:input={components.ItemList.getItems}
              bind:checked={selectedCategory} />
            <label
              class="ml-1 my-1 mr-3 select-none cursor-pointer text-sm"
              for="check--category-{categories[itemType][index]}--item">{categories[itemType][index]}</label>
          </div>
        {/each}
      </div>
    </div>

    <hr />

    <!-- DIALOG BODY -->
    <div
      class="dialog-body flex h-full overflow-y-auto max-h-full flex-col p-3">
      <ItemList
        bind:this={components.ItemList}
        {itemType}
        {searchKeyword}
        tiers={tiersIndex[itemType]}
        {categories}
        categoriesIndex={categoriesIndex[itemType]} />
    </div>

    <hr />

    <!-- DIALOG FOOTER -->
    <div class="dialog-actions flex p-2">
      <button
        type="button"
        class="ml-auto px-5 py-2 bg-blue-700 text-white"
        aria-label="Close select item dialog"
        on:click={close}>close</button>
    </div>
  </div>
  <!--  -->
</div>
