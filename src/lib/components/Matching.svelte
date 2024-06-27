<script>
    // import ProgressNav from '$lib/ProgressNav.svelte';
    // import Summary from '$lib/Summary.svelte';
    import { createEventDispatcher } from 'svelte';
    const dispatch = createEventDispatcher();

    /*
     * Word matching game in Svelte.
     * 5 pairs of words are shown at a time, in a mixed order.
     * English on the left, French on the right.
     * The matching words may not be next to each other.
     * The user must click on the English word, then on the matching French word.
     * When both have been selected, a new pair of words is put into those places.
     */

    export let sourceWords = [];

    let shuffledWords = [...sourceWords].sort(() => Math.random() - 0.5);

    const WordStatus = {
        Default: 0,
        CorrectMatch: 1,
        IncorrectMatch: 2,
    };

    // Construct initial words list by taking the first 5 pairs from the shuffled list.
    let words = [];
    if (shuffledWords.length < 5) {
        words = shuffledWords;
        shuffledWords = [];
    } else {
        while (words.length < 5) {
            words.push(shuffledWords.pop());
        }
    }
    let lhsWords = words.map((word) => {
        return {
            value: word.en,
            match: word.fr,
            status: WordStatus.Default,
            fading: false,
        }
    });

    // Shuffle French words
    let rhsWords = words.map((word) => {
        return {
            value: word.fr,
            match: word.en,
            status: WordStatus.Default,
            fading: false,
        }
    }).sort(() => Math.random() - 0.5);

    // Track which words are currently selected
    let lhsSelectedId = null;
    let rhsSelectedId = null;

    // Track index of correct matches
    let matches = 0;
    let incorrectMatches = 0;

    // Queue upcoming words
    let lhsQueue = [];
    let rhsQueue = [];

    // Track whether the game is over
    let gameOver = false;

    // Track game times
    let startTime = Date.now();
    let timeToComplete = null;

    // Add next word to queue
    function unqueue(lhsId, rhsId) {
        if (lhsQueue.length > 0) {
            const lhsWord = lhsQueue.pop();
            const rhsWord = rhsQueue.splice(Math.floor(Math.random() * rhsQueue.length), 1)[0];

            // Replace the selected words with new ones
            lhsWords[lhsId] = { ...lhsWord };
            rhsWords[rhsId] = { ...rhsWord };
        } else {
            if (matches === sourceWords.length) {
                // Game is over
                timeToComplete = Date.now() - startTime;
                gameOver = true;
            }
        }
    }

    function selectEnWord(id) {
        rhsSelectedId = null;
        lhsSelectedId = id;
    }

    function selectFrWord(id) {
        if (lhsSelectedId === null) {
            return;
        }
        rhsSelectedId = id;

        // Check if the selected words match
        if (lhsWords[lhsSelectedId].match === rhsWords[rhsSelectedId].value) {
            matches++;

            let lhsId = lhsSelectedId;
            let rhsId = rhsSelectedId;

            let newWord = shuffledWords.pop();
            if (newWord) {
                lhsQueue.push({value: newWord.en, match: newWord.fr, status: WordStatus.Default, fading: false});
                rhsQueue.push({value: newWord.fr, match: newWord.en, status: WordStatus.Default, fading: false});
            }

            lhsWords[lhsId].status = WordStatus.CorrectMatch;
            rhsWords[rhsId].status = WordStatus.CorrectMatch;

            // After a short delay, fade out the matched words
            setTimeout(() => {
                lhsWords[lhsId].fading = true;
                rhsWords[rhsId].fading = true;
            }, 500);

            // After a longer delay, replace the matched words with new ones
            setTimeout(() => {
                unqueue(lhsId, rhsId);
            }, 2000);
        } else {
            incorrectMatches++;

            const lhsId = lhsSelectedId;
            const rhsId = rhsSelectedId;

            lhsWords[lhsId].status = WordStatus.IncorrectMatch;
            rhsWords[rhsId].status = WordStatus.IncorrectMatch;

            // After a short delay, reset the selected words
            setTimeout(() => {
                lhsWords[lhsId].status = WordStatus.Default;
                rhsWords[rhsId].status = WordStatus.Default;
            }, 1000);
        }

        // Reset the selected words
        lhsSelectedId = null;
        rhsSelectedId = null;
    }

    let pairs = [];

    $: {
        pairs = [];
        for (let i = 0; i < lhsWords.length; i++) {
            pairs.push({
                lhs: lhsWords[i].value,
                rhs: rhsWords[i].value
            });
        }
    }
</script>

{#if gameOver}
<!--    <Summary-->
<!--            xp={sourceWords.length}-->
<!--            time={timeToComplete}-->
<!--            accuracy={matches / (matches + incorrectMatches)}-->
<!--            button={{-->
<!--                text: 'Continue',-->
<!--                click: () => dispatch('exit')-->
<!--            }}-->
<!--    />-->
{:else}
<!--    <ProgressNav-->
<!--            current={matches + 1}-->
<!--            total={sourceWords.length}-->
<!--            on:exit={() => dispatch('exit')}-->
<!--    />-->

    <div class="game">
        {#each pairs as {lhs, rhs}, index (index)}
            <div class="pair">
                <button
                        class="btn word"
                        class:selected={index === lhsSelectedId}
                        class:correctMatch={lhsWords[index].status === WordStatus.CorrectMatch}
                        class:incorrectMatch={lhsWords[index].status === WordStatus.IncorrectMatch}
                        class:fading={lhsWords[index].fading}
                        on:click={() => selectEnWord(index)}
                >
                    {lhs}
                </button>
                <button
                        class="btn word"
                        class:selected={index === rhsSelectedId}
                        class:correctMatch={rhsWords[index].status === WordStatus.CorrectMatch}
                        class:incorrectMatch={rhsWords[index].status === WordStatus.IncorrectMatch}
                        class:fading={rhsWords[index].fading}
                        on:click={() => selectFrWord(index)}
                >
                    {rhs}
                </button>
            </div>
        {/each}
    </div>
{/if}

<style lang="postcss">
    .game {
        padding: 1rem 0;
    }

    .pair {
        display: grid;
        grid-template-columns: 1fr 1fr;
        grid-gap: 0.5rem;
        margin-bottom: 0.5rem;
    }

    .word {
      display: block;
      width: 100%;
      padding: 0.75rem;
      border-radius: 0.5rem;
      background-color: #fff;
      color: #4a5568;
      font-size: 1rem;
      opacity: 1;
      transition: opacity 0.5s ease-in-out,
      background-color 0.15s ease-in-out,
      color 0.15s ease-in-out;
    }

    .word:hover {
        background-color: #f7fafc;
        transition: none;
    }

    .word.selected {
        background-color: theme(colors.sky.100);
        color: theme(colors.sky.800);
        border-color: theme(colors.sky.300);
    }

    .word.correctMatch {
        background-color: theme(colors.lime.100);
        color: theme(colors.lime.800);
        border-color: theme(colors.lime.300);
        cursor: default;
    }

    .word.incorrectMatch {
        background-color: theme(colors.red.100);
        color: theme(colors.red.800);
        border-color: theme(colors.red.300);
        cursor: default;
    }

    .word.fading {
      opacity: 0;
    }
</style>
