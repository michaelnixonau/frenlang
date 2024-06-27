<script>
    import Select from '$lib/components/form/Select.svelte';
    import RadioGroup from '$lib/components/form/RadioGroup.svelte';
    import TopicTags from '$lib/components/form/TopicTags.svelte';
    import {wordTypes, masculineArticles, feminineArticles} from '$lib/french.js';
    import {onMount} from 'svelte';

    /** @type string */
    export let word = '';

    /** @type string */
    export let translation = '';

    /** @type string */
    export let wordType = '';

    /** @type {'m'|'f'|''} */
    export let gender = '';

    /** @type string */
    export let pluralForm = '';

    /** @type {any} */
    export let topics;

    /** @type {HTMLInputElement} */
    let wordInput;

    /**
     * Infer gender from the word based on the article/preposition.
     * For French.
     * @returns {void}
     */
    function inferGender() {
        const words = word.split(' ');
        const firstWord = words[0].toLowerCase();

        if (masculineArticles.includes(firstWord)) {
            gender = 'm';
        } else if (feminineArticles.includes(firstWord)) {
            gender = 'f';
        }
    }

    /**
     * Reset the form fields after adding a new vocab item.
     * @returns {void}
     */
    export const reset = () => {
        word = '';
        translation = '';
        pluralForm = '';
        gender = '';
        wordInput.focus();
    }

    onMount(() => {
        wordInput.focus();
    });
</script>

<div class="my-4 flex flex-col w-3/12">
    <Select
            labelTitle="Word Type"
            options={Object.entries(wordTypes).map(([value, label]) => ({ value, label }))}
            bind:value={wordType}
    />
</div>

<hr class="border-b border-b-slate-50" />

<div class="my-4">
    <form>
        <div class="grid grid-cols-2 gap-4 mb-4">
            <div>
                <label for="word" class="block text-sm mb-1 text-slate-500">Word</label>
                <input type="text" class="w-full" bind:this={wordInput} bind:value={word} on:input={inferGender}>
            </div>
            <div>
                <label for="meaning" class="block text-sm mb-1 text-slate-500">Translation</label>
                <input type="text" class="w-full" bind:value={translation}>
            </div>
        </div>
        {#if wordType === 'noun'}
            <!-- Plural Form -->
            <div class="grid grid-cols-2 gap-4 mb-4">
                <div>
                    <label for="plural-form" class="block text-sm mb-1 text-slate-500">Plural Form</label>
                    <input type="text" class="w-full" bind:value={pluralForm}>
                </div>
            </div>
            <!-- Gender -->
            <div class="flex flex-col gap-1">
                <div class="block text-sm mb-1 text-slate-500">Gender</div>
                <RadioGroup
                        options={[
                        { value: 'm', label: 'Masculine' },
                        { value: 'f', label: 'Feminine' },
                    ]}
                        bind:value={gender}
                />
            </div>
        {/if}
    </form>
</div>

<hr class="border-b border-b-slate-50" />

<TopicTags bind:topics />
