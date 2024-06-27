<script>
    import Add from '~icons/material-symbols/add';
    import { createDialog, melt } from '@melt-ui/svelte';
    import Close from '~icons/material-symbols/close';
    import ProgressActivity from '~icons/material-symbols/progress-activity';
    import { fade, fly } from 'svelte/transition';
    import { cubicOut } from 'svelte/easing';
    import { addToast } from '$lib/components/Toaster.svelte';
    import { invalidateAll } from '$app/navigation';

    /** @type {string} */
    let name = '';

    /** @type {string} */
    let description = '';

    /** @type {boolean} */
    let loading = false;

    /** @type {boolean} */
    export let invalidateAllOnCreation = false;

    const {
        elements: {
            trigger,
            overlay,
            content,
            title: dialogTitle,
            description: dialogDescription,
            close,
            portalled,
        },
        states: { open },
    } = createDialog({
        forceVisible: true,
    });

    /**
     * Slide and fade transition.
     * @param node The node to transition.
     * @param params The transition parameters.
     */
    function slidefade(node, params) {
        const existingTransform = getComputedStyle(node).transform.replace('none', '');
        const initialScale = params.initialScale || 0;

        return {
            delay: params.delay || 0,
            duration: params.duration || 400,
            easing: params.easing || cubicOut,
            css: (t, u) => `transform-origin: center center; transform: ${existingTransform} scale(${initialScale + (1 - initialScale) * t}); opacity: ${t};`
        };
    }

    /**
     * Reset the form.
     * @returns {void}
     */
    function reset() {
        name = '';
        description = '';
    }

    /**
     * Create a new list.
     * @returns {Promise<void>}
     */
    async function create() {
        if (loading) {
            return;
        }
        loading = true;
        try {
            const response = await fetch('/list', {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json',
                },
                body: JSON.stringify({ name, description }),
            });
            const { error } = await response.json();
            if (error) {
                throw new Error(error);
            }
            addToast({ data: {
                title: 'Success',
                description: 'The list has been created.',
                type: 'success',
            }});
            if (invalidateAllOnCreation) {
                await invalidateAll();
            }
            reset();
            $open = false;
        } catch(e) {
            console.error(e);
            addToast({ data: {
                title: 'Error',
                description: e.message,
                type: 'error',
            }});
        } finally {
            loading = false;
        }
    }
</script>

<button
    class="btn btn-primary flex gap-1 items-center"
    use:melt={$trigger}
>
    <Add class="p-0 h-6 w-auto" /> New List
</button>

{#if $open}
    <div class="" use:melt={$portalled}>
        <div
            use:melt={$overlay}
            class="fixed inset-0 z-49 bg-black/50"
            transition:fade={{ duration: 150 }}
        />
        <div
            class="fixed left-1/2 top-1/2 z-50 max-h-[85vh] w-[90vw]
            max-w-[450px] -translate-x-1/2 -translate-y-1/2 rounded-xl bg-white
            p-6 shadow-lg"
            use:melt={$content}
            transition:slidefade={{ duration: 150, initialScale: 0.75 }}
        >
            <h2 use:melt={$dialogTitle} class="m-0 text-2xl font-semibold">New List</h2>
            <p use:melt={$dialogDescription} class="mb-5 mt-2 leading-normal text-slate-500">Create a new list to add vocabulary to.</p>

            <fieldset class="mb-4 flex items-center gap-5">
                <label for="name" class="w-[90px] text-right">Name</label>
                <input
                    type="text"
                    id="name"
                    class="inline-flex w-full flex-1 items-center justify-center"
                    bind:value={name}
                />
            </fieldset>

            <fieldset class="mb-4 flex items-center gap-5">
                <label for="description" class="w-[90px] text-right">Description</label>
                <input
                    type="text"
                    id="description"
                    class="inline-flex w-full flex-1 items-center justify-center"
                    bind:value={description}
                />
            </fieldset>

            <div class="mt-6 flex justify-end gap-2">
                <button use:melt={$close} class="btn">Cancel</button>
                <button
                    class="btn btn-primary flex items-center gap-1"
                    disabled={loading}
                    on:click={create}
                >
                    {#if loading}
                        <span class="animate-spin"><ProgressActivity /></span>
                    {/if}
                    Create
                </button>
            </div>
        </div>
    </div>
{/if}
