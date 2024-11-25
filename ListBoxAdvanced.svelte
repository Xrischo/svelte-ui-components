<script lang="ts">
  import { onMount } from "svelte";

  interface Props {
    type?: "single" | "multiple";
    label?: string;
    selectedItem?: number | string | number[] | string[];
    selectedItemsMultiple?: (string | number | number[] | string[])[];
    getListItem: (item: (typeof itemList)[number]) => string | number;
    getSelectedItem: (
      item: (typeof itemList)[number],
    ) => string | number | number[] | string[];
    itemList: any[];
    size?: string;
    itemStyle?: string;
    background?: string;
    selectedClass?: string;
    currentClass?: string;
    onchange: () => void;
  }

  let {
    type = "single",
    label = "",
    selectedItem = $bindable(""),
    selectedItemsMultiple = $bindable([]),
    getListItem,
    getSelectedItem,
    itemList,
    size = "h-60 w-40",
    itemStyle = "",
    background = "bg-surface-200 text-black dark:bg-surface-700 dark:text-white",
    selectedClass = "bg-secondary-500 text-white",
    currentClass = "bg-surface-300 text-black dark:bg-surface-600 dark:text-white",
    onchange = () => {},
  }: Props = $props();

  let initialisedItem: string | number | number[] | string[];
  let initialisedItemsMultiple: (string | number | number[] | string[])[];
  let initialItems: any[];
  let mounted = $state(false);

  onMount(() => {
    initialisedItem = selectedItem;
    initialisedItemsMultiple = selectedItemsMultiple;
    initialItems = itemList;
    mounted = true;
  });

  function checkSelectItemIsInitialised() {
    if (JSON.stringify(initialItems) !== JSON.stringify(itemList)) {
      selectedItemsMultiple = [];
    } else {
      selectedItemsMultiple = initialisedItemsMultiple;
    }
  }

  function getSelectedClasses(item: typeof selectedItem, reactivity: any) {
    let pureItem = getSelectedItem(item);

    return Array.isArray(pureItem)
      ? arraysEqual(pureItem, selectedItem)
        ? selectedClass
        : arraysEqual(pureItem, initialisedItemsMultiple)
          ? currentClass
          : "hover:bg-surface-300 dark:hover:bg-surface-600"
      : pureItem === selectedItem
        ? selectedClass
        : pureItem === initialisedItem
          ? currentClass
          : "hover:bg-surface-300 dark:hover:bg-surface-600";
  }

  function arraysEqual(a: any, b: any) {
    if (a === b) return true;
    if (a == null || b == null) return false;
    if (a.length !== b.length) return false;

    for (var i = 0; i < a.length; ++i) {
      if (a[i] !== b[i]) return false;
    }
    return true;
  }

  $effect(() => {
    if (mounted && itemList) checkSelectItemIsInitialised();
  });
</script>

<div class="flex flex-col">
  {#if label}
    <div class="text-sm ml-1 mb-2">
      {label}
    </div>
  {/if}
  {#if type === "single"}
    <div
      class="
                {size} 
                {background} 
                flex
                flex-col
                overflow-y-auto
                rounded
                border-solid
                border
                border-surface-300
                dark:border-surface-600
                p-2
            "
    >
      {#each itemList as item}
        <!-- svelte-ignore a11y_click_events_have_key_events -->
        <!-- svelte-ignore a11y_no_static_element_interactions -->
        <div
          onclick={() => {
            if (selectedItem != getSelectedItem(item)) {
              selectedItem = getSelectedItem(item);
              onchange();
            }
          }}
          class="
                    flex
                    items-center
                    hover:cursor-pointer
                    duration-150
                    {itemStyle}
                    {getSelectedClasses(item, selectedItem)}
                "
        >
          {getListItem(item)}
        </div>
      {/each}
    </div>
  {:else if type === "multiple"}
    <div
      class="
                {size} 
                {background} 
                flex
                flex-col
                overflow-y-auto
                rounded
                border-solid
                border
                border-surface-300
                dark:border-surface-600
                p-2

            "
    >
      {#each itemList as item}
        <!-- svelte-ignore a11y_click_events_have_key_events -->
        <!-- svelte-ignore a11y_no_static_element_interactions -->
        <div
          onclick={() => {
            if (selectedItemsMultiple.includes(getSelectedItem(item)))
              selectedItemsMultiple = selectedItemsMultiple.filter(
                (i) => i !== getSelectedItem(item),
              );
            else
              selectedItemsMultiple = [
                ...selectedItemsMultiple,
                getSelectedItem(item),
              ];
          }}
          class="
                    flex
                    items-center
                    hover:cursor-pointer
                    duration-150
                    {itemStyle}
                    {selectedItemsMultiple.includes(getSelectedItem(item))
            ? selectedClass
            : 'hover:bg-surface-300 dark:hover:bg-surface-600'}
                "
        >
          {getListItem(item)}
        </div>
      {/each}
    </div>
  {/if}
</div>
