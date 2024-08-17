<script>
    import EditVocab from '$lib/components/EditVocab.svelte';
    import {goto} from "$app/navigation";
    import {removeArticleFr} from '$lib/french.js';
    import {removeArticleEn} from '$lib/english.js';
    import {addToast} from '$lib/components/Toaster.svelte';

    /** @type boolean */
    let loading = false;

    /** @type string */
    let word = '';

    /** @type string */
    let translation = '';

    /** @type string */
    let wordType = 'noun';

    /** @type {'m'|'f'|''} */
    let gender = '';

    /** @type string */
    let pluralForm = '';

    /** @type {any[]} */
    let topics;

    /** @type {() => void} */
    let reset;

    async function add() {
        if (loading) {
            return;
        }
        loading = true;
        try {
            const cleanWord = removeArticleFr(word).trim();
            const cleanTranslation = removeArticleEn(translation).trim();
            const body = JSON.stringify({
                word_type: wordType,
                word: cleanWord,
                translation: cleanTranslation,
                plural_form: wordType === 'noun' ? removeArticleFr(pluralForm).trim() : undefined,
                gender: wordType === 'noun' ? gender : undefined,
                topic_ids: topics.map(t => t.id),
            });
            console.log(body);
            const response = await fetch('/builder/vocabulary', {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: body,
            });
            const {error} = await response.json();
            if (error) {
                throw new Error(error);
            }
            reset();
            addToast({ data: {
                title: 'Success',
                description: `Added "${cleanWord}" to vocabulary.`,
                type: 'success',
            }});
        } catch(e) {
            console.log(e.message);
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

<svelte:window on:keydown={e => {
    if (e.ctrlKey && e.key === 'Enter') {
        add();
        console.log('Ctrl + Enter');
    }
    if (e.key === 'Escape') {
        goto('/builder/vocabulary');
    }
}} />

<div class="flex justify-between items-center">
    <h1 class="font-semibold text-3xl">Add Vocabulary</h1>
    <div class="flex gap-2">
        <a class="btn btn-secondary flex items-center gap-2" href="/builder/vocabulary">
            Cancel
            <span class="text-xs text-slate-500 font-mono border border-slate-300 rounded p-1">
                Esc
            </span>
        </a>
        <button class="btn btn-primary flex items-center gap-2" disabled={loading} on:click={add}>
            {loading ? 'Adding' : 'Add'}
            <span class="text-xs text-sky-100 font-mono border border-sky-400 rounded p-1">
                Ctrl + Enter
            </span>
        </button>
    </div>
</div>

<EditVocab
    bind:word
    bind:translation
    bind:wordType
    bind:gender
    bind:pluralForm
    bind:topics
    bind:reset
/>
