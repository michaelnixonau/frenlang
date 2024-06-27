<script>
    import ProgressDots from '$lib/components/ProgressDots.svelte';
    import Flashcard from '$lib/components/Flashcard.svelte';

    export let data;

    /** @type {{ front: string, back: string }[]} */
    let flashcards = [];

    $: (flashcards = data.vocabulary.map(({ word, translation }) => ({
        front: word,
        back: translation
    })));

    const confidence = Array(flashcards.length).fill(0);

    let currentCard = 0;
    let flipped = false;
</script>

<div class="mt-4 mb-10">
    <ProgressDots
            total={flashcards.length}
            current={currentCard + 1}
    />
</div>

<Flashcard
    on:confidence={(e) => {
        confidence[currentCard] = e.detail;
        currentCard = (currentCard + 1) % flashcards.length;
        flipped = false;
    }}
    bind:flipped
>
    <p slot="front">{flashcards[currentCard].front}</p>
    <p slot="back" class="font-bold">{flashcards[currentCard].back}</p>
</Flashcard>

<!--{JSON.stringify(confidence)}-->
