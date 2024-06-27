<script>
    import { createRadioGroup, melt, createSync } from '@melt-ui/svelte';

    /** @type {Array<{ value: string, label: string }>} */
    export let options;

    /** @type {string} */
    export let value;

    const {
        states,
        elements: { root, item, hiddenInput },
        helpers: { isChecked },
    } = createRadioGroup({
        defaultValue: 'default',
    });

    const sync = createSync(states);
    $: sync.value(value, (v) => value = v);
</script>

<div
    use:melt={$root}
    class="flex flex-col gap-3 data-[orientation=horizontal]:flex-row"
>
    {#each options as { value, label }}
        <div class="flex items-center gap-3">
            <button
                use:melt={$item(value)}
                class="grid h-6 w-6 cursor-default place-items-center rounded-full border border-slate-300 hover:border-slate-500 focus:border-sky-500 transition-colors"
                id={value}
                aria-labelledby="{value}-label"
            >
                {#if $isChecked(value)}
                    <div class="h-3 w-3 rounded-full bg-sky-500" />
                {/if}
            </button>
            <label
                class="leading-none"
                for={value}
                id="{value}-label"
            >
                {label}
            </label>
        </div>
    {/each}
</div>
