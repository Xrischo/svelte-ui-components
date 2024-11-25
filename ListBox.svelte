<script lang="ts">
    import { onMount } from "svelte";

    interface Props {
        type?: "single" | "multiple";
        // export let selectedItem: string | number = ''
        value:
            | (string | number | boolean)[]
            | string
            | number
            | boolean
            | undefined
            | null;
        options?: { value: string | number | boolean; name: string }[];
        width?: string;
        height?: string;
        padding?: string;
        border?: string;
        background?: string;
        selectedClass?: string;
        stopPropagation?: boolean;
        focusInput?: () => void;
        unselect?: boolean;
        textSize?: "text-sm" | "text-md" | "text-lg";
        onchange?: (
            value:
                | (string | number | boolean)[]
                | string
                | number
                | boolean
                | undefined
                | null,
        ) => void;
    }

    let {
        type = "single",
        value = $bindable(),
        options = [],
        width = "w-40",
        height = "h-60",
        padding = "p-2",
        border = "border border-gray-300 dark:border-surface-600",
        background = "bg-white text-black dark:bg-surface-800 dark:text-white",
        selectedClass = "bg-gray-200 text-black dark:bg-surface-600 dark:text-white",
        stopPropagation = false,
        focusInput = () => {},
        unselect = false,
        textSize = "text-sm",
        onchange = () => {},
    }: Props = $props();

    let initialisedItem:
        | string
        | number
        | boolean
        | (string | number | boolean)[]
        | undefined
        | null;

    onMount(() => {
        initialisedItem = value;
    });

    function handleItem(item: {
        value: string | number | boolean;
        name: string;
    }) {
        if (type === "multiple") {
            if (!Array.isArray(value)) value = [item.value];
            else {
                if (value.includes(item.value))
                    value = value.filter((i) => i !== item.value);
                else value = [...value, item.value];
            }
        } else {
            if (unselect && value === item.value) value = undefined;
            else value = item.value;
        }
        onchange(value);
    }

    let outerDiv: HTMLDivElement = $state() as HTMLDivElement;
</script>

<!-- svelte-ignore a11y_no_static_element_interactions -->
<!-- svelte-ignore a11y_click_events_have_key_events -->
<div
    bind:this={outerDiv}
    onclick={(e) => {
        e.preventDefault();
        e.target === outerDiv && e.stopPropagation();
        focusInput();
    }}
    class="
            {width}
            {height}
            {background}
            p-2
            flex
            flex-col
            overflow-y-auto
            rounded-lg
            {border}
        "
>
    {#if options.length === 0}
        <div>No options</div>
    {:else}
        {#each options as item}
            <!-- svelte-ignore a11y_click_events_have_key_events -->
            <!-- svelte-ignore a11y_no_static_element_interactions -->
            <div
                onclick={(e) => {
                    e.preventDefault();
                    handleItem(item);
                    stopPropagation && e.stopPropagation();
                    focusInput();
                }}
                class="
                    {padding}
                    {textSize}
                    rounded-md
                    mt-1
                    flex
                    items-center
                    hover:cursor-pointer
                    py-2 px-2
                    {(Array.isArray(value) && value.includes(item.value)) ||
                item.value === value
                    ? selectedClass
                    : 'hover:bg-gray-100 dark:hover:bg-surface-700'}
                "
            >
                {item.name}
            </div>
        {/each}
    {/if}
</div>
