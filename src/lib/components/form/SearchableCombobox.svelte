<script>
    import ArrowDropDown from '~icons/material-symbols/arrow-drop-down';
    import ArrowDropUp from '~icons/material-symbols/arrow-drop-up';
    import { createCombobox, melt } from '@melt-ui/svelte';
    import { fly } from 'svelte/transition';
    import { onMount } from "svelte";

    /**
     * @typedef {Object} Item
     * @property {string|number} id - Unique identifier for the item
     * @property {string} name - Display name of the item
     * @property {string} description - Description of the item
     * @property {boolean} [disabled] - Whether the item is disabled
     */

    /** @type {Item[]} */
    export let selectedItems = [];

    /** @type {(searchTerm: string) => Promise<Item[]>} */
    export let fetchItems;

    /** @type {string} */
    export let label = "Select items";

    /** @type {string} */
    export let placeholder = "Type to search";

    /** @type {number} */
    export let debounceTime = 300;

    /** @type {Item[]} */
    let availableItems = [];

    /** @type {boolean} */
    let searching = false;

    /** @type {number|null} */
    let timer = null;

    /** @type {string} */
    let lastQuery = '';

    const toOption = (item) => ({
        value: item,
        label: item.name,
        disabled: item.disabled,
    });

    const {
        elements: { menu, input, option, label: comboboxLabel },
        states: { open, inputValue, touchedInput, selected },
        helpers: { isSelected },
    } = createCombobox({
        forceVisible: true,
    });

    $: if (!$open) {
        $inputValue = $selected?.label ?? '';
    }

    /** @type {Item[]} */
    let filteredItems = [];
    $: {
        let userFilteredItems = availableItems;
        if ($touchedInput) {
            userFilteredItems = availableItems.filter(({ name, description }) => {
                const normalizedInput = $inputValue.toLowerCase();
                return (
                    name.toLowerCase().includes(normalizedInput) ||
                    description.toLowerCase().includes(normalizedInput)
                );
            });
        }
        filteredItems = userFilteredItems.filter((item) => {
            return !selectedItems.some((selectedItem) => selectedItem.id === item.id);
        });
    }

    $: handleInput($inputValue);

    /**
     * @param {string} value
     */
    const handleInput = (value) => {
        searching = true;
        if (timer) clearTimeout(timer);
        timer = setTimeout(async () => {
            await fetchItemsFromServer(value);
            searching = false;
        }, debounceTime);
    }

    $: if ($selected) {
        selectedItems = [...selectedItems, $selected.value];
        $selected = undefined;

        if (lastQuery === '') {
            searching = true;
            fetchItemsFromServer('').then(() => {
                searching = false;
            });
        }
    }

    /**
     * Fetch items from the server.
     * @param {string} searchTerm
     */
    async function fetchItemsFromServer(searchTerm) {
        try {
            lastQuery = searchTerm;
            availableItems = await fetchItems(searchTerm);
        } catch (error) {
            console.error(error);
        }
    }

    onMount(() => {
        fetchItemsFromServer('');
    });
</script>

<div class="flex flex-col gap-1 my-5">
    <!-- svelte-ignore a11y-label-has-associated-control - $comboboxLabel contains the 'for' attribute -->
    <label use:melt={$comboboxLabel}>
        <span class="block text-sm mb-1 text-slate-500">{label}</span>
    </label>

    <div class="relative">
        <input
                type="text"
                use:melt={$input}
                class="flex h-10 items-center justify-between rounded-lg bg-white px-3 pr-12 text-black w-full"
                {placeholder}
        />
        <div class="absolute right-2 top-1/2 z-10 -translate-y-1/2 text-magnum-900">
            {#if $open}
                <!-- Add your ChevronUp icon here -->
                <ArrowDropUp />
            {:else}
                <!-- Add your ChevronDown icon here -->
                <ArrowDropDown />
            {/if}
        </div>
    </div>
</div>

{#if $open}
    <ul
            class="z-10 flex max-h-[300px] flex-col overflow-hidden rounded-lg shadow-md shadow-slate-100 border border-slate-200"
            use:melt={$menu}
            transition:fly={{ duration: 150, y: -5 }}
    >
        <!-- svelte-ignore a11y-no-noninteractive-tabindex -->
        <div
                class="flex max-h-full flex-col gap-0 overflow-y-auto bg-white px-2 py-2 text-black"
                tabindex="0"
        >
            {#each filteredItems as item (item.id)}
                <li
                        use:melt={$option(toOption(item))}
                        class="relative cursor-pointer scroll-my-2 rounded-md py-2 px-4 hover:bg-sky-100 data-[highlighted]:bg-sky-200 data-[highlighted]:text-sky-900 data-[disabled]:opacity-50"
                >
                    <div>
                        <span class="font-medium">{item.name}</span>
                        {#if item.description}
                            <span class="block text-sm opacity-75">{item.description}</span>
                        {/if}
                    </div>
                </li>
            {:else}
                <li class="relative cursor-pointer rounded-md py-1 pl-8 pr-4">
                    {#if searching}
                        Searching...
                    {:else}
                        No results found
                    {/if}
                </li>
            {/each}
        </div>
    </ul>
{/if}
