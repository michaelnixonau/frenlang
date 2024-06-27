<script>
    import EditVocab from '$lib/components/EditVocab.svelte';
    import {goto} from '$app/navigation';
    import {onMount} from 'svelte';

    export let data;
    const {item} = data;

    console.log(item);

    let {
        word_type: wordType,
        word,
        translation,
        gender,
        plural_form: pluralForm,
        topics
    } = item;

    /** @type boolean */
    let loading = false;

    /** @type {() => void} */
    let reset;

    /**
     * Update the vocabulary item.
     */
    async function update() {
        loading = true;
        try {
            console.log(JSON.stringify({
                word,
                translation,
                topic_ids: topics.map(t => t.id),
            }));
            const res = await fetch(`/builder/vocabulary/${data.item.id}`, {
                method: 'PATCH',
                headers: {
                    'Content-Type': 'application/json',
                },
                body: JSON.stringify({
                    word_type: wordType,
                    word,
                    translation,
                    gender,
                    plural_form: wordType === 'noun' ? pluralForm : undefined,
                    topic_ids: topics.map(t => t.id),
                }),
            });
            if (res.ok) {
                goto('/builder/vocabulary');
            } else {
                const json = await res.json();
                console.error(json);
            }
        } catch (e) {
            console.error(e);
        } finally {
            loading = false;
        }
    }
</script>

<svelte:window on:keydown={e => {
    if (e.ctrlKey && e.key === 'Enter') {
        update();
    }
    if (e.key === 'Escape') {
        goto('/builder/vocabulary');
    }
}} />

<div class="flex justify-between items-center">
    <h1 class="font-semibold text-3xl">Edit Vocabulary</h1>
    <div class="flex gap-2">
        <a class="btn btn-secondary flex items-center gap-2" href="/builder/vocabulary">
            Cancel
            <span class="text-xs text-slate-500 font-mono border border-slate-300 rounded p-1">
                Esc
            </span>
        </a>
        <button
                class="btn btn-primary flex items-center gap-2"
                disabled={loading}
                on:click={update}
        >
            Update
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
