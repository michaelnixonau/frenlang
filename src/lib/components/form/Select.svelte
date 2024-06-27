<script>
    import { createSelect, melt } from '@melt-ui/svelte';
    import { fade } from 'svelte/transition';
    import { createEventDispatcher } from 'svelte';
    import ArrowDropDown from '~icons/material-symbols/arrow-drop-down';
    import ArrowDropUp from '~icons/material-symbols/arrow-drop-up';
    import MaterialSymbolsCheck from '~icons/material-symbols/check';

    const dispatch = createEventDispatcher();

    /** @type {string|undefined} */
    export let labelTitle;

    /** @type {string} */
    export let placeholder = 'Select an option';

    /** @type {Array<{ value: string, label: string }>} */
    export let options = [];

    /** @type {string|undefined} */
    export let value;

    /**
     * Given a value, return the corresponding option object.
     * If no option is found, return undefined.
     * @param {string} value
     * @returns {{ value: string, label: string } | undefined}
     */
    function getOption(value) {
        return options.find((o) => o.value === value);
    }

    const {
        elements: { trigger, menu, option, label },
        states: { selected, selectedLabel, open },
        helpers: { isSelected },
    } = createSelect({
        forceVisible: true,
        positioning: {
            placement: 'bottom',
            fitViewport: true,
            sameWidth: true,
        },
    });

    // Sync state at the beginning.
    selected.set(getOption(value || ''));

    // Sync state when value changes.
    $: {
        const currentSelectedValue = $selected?.value;
        if (currentSelectedValue !== value) {
            selected.set(getOption(value || ''));
        }
    }

    // Sync value when state changes.
    $: {
        const currentSelectedValue = $selected?.value;
        if (currentSelectedValue !== value) {
            value = currentSelectedValue;
            dispatch('change', value);
        }
    }
</script>

<div class="flex flex-col">
    <!-- svelte-ignore a11y-label-has-associated-control - $label contains the 'for' attribute -->
    {#if labelTitle}
        <label class="block text-sm mb-1 text-slate-500" use:melt={$label}>{labelTitle}</label>
    {/if}
    <button
            class="flex justify-between items-center text-left border border-gray-300 rounded-md py-2 px-3 focus:outline-none focus:border-sky-500 focus:ring focus:ring-sky-500 focus:ring-opacity-20 {$selectedLabel ? 'text-black' : 'text-slate-500'}"
            use:melt={$trigger}
            aria-label="Food"
    >
        {$selectedLabel || placeholder}
        {#if $open}
            <!-- Add your ChevronUp icon here -->
            <ArrowDropUp />
        {:else}
            <!-- Add your ChevronDown icon here -->
            <ArrowDropDown />
        {/if}
    </button>
    {#if $open}
        <div
                class="z-10 bg-white p-1 flex max-h-[300px] flex-col overflow-y-auto rounded-lg shadow-md shadow-slate-100 border border-slate-200"
                use:melt={$menu}
                transition:fade={{ duration: 150 }}
        >
            {#each options as {label, value} (value)}
                <div
                        class="relative cursor-pointer scroll-my-2 rounded-md py-2 px-4 pl-8 hover:bg-sky-100 data-[highlighted]:bg-sky-200 data-[highlighted]:text-sky-900 data-[disabled]:opacity-50"
                        use:melt={$option({ value, label })}
                >
                    <div class="check {$isSelected(value) ? 'block' : 'hidden'}">
                        <MaterialSymbolsCheck />
                    </div>

                    {label}
                </div>
            {/each}
        </div>
    {/if}
</div>

<style lnag="postcss">
    .check {
        position: absolute;
        left: theme(spacing.2);
        top: 50%;
        z-index: theme(zIndex.20);
        translate: 0 calc(-50% + 1px);
        color: theme(colors.sky.500);
    }
</style>
