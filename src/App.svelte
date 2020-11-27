<script>
  import GearEditor from './components/GearEditor.svelte';
  import SelectItemDialog from './components/SelectItemDialog.svelte';

  let components = { selectItemDialog: null };

  let selectedGearName, selectedItemType;

  function openSelectItemDialog({ detail }) {
    selectedGearName = detail.gearName;
    selectedItemType = detail.itemType;
    components.selectItemDialog.open();
  }
</script>

<div>
  <div class="flex justify-center flex-row">
    <GearEditor gearName="Character 4" on:select-item={openSelectItemDialog} />
    <GearEditor gearName="Character 2" on:select-item={openSelectItemDialog} />
    <SelectItemDialog
      bind:this={components.selectItemDialog}
      gearName={selectedGearName}
      itemType={selectedItemType} />
  </div>
</div>
