<script>
    import Matching from "$lib/components/Matching.svelte";
    export let data;
    $: ({ list: { vocabulary } } = data);

    /** @type {{ en: string, fr: string }[]} */
    let words = [];

    $: (words = vocabulary.map(({ word, translation }) => ({
        en: word,
        fr: translation
    })));
</script>

<div class="max-w-[600px] mx-auto">
    <Matching
        sourceWords={words}
    />
</div>
