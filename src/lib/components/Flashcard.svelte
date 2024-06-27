<script>
    import { createEventDispatcher } from 'svelte';

    const dispatch = createEventDispatcher();

    export let flipped = false;
</script>

<div class="container mx-auto text-center">
    <!-- Flashcard -->
    <div class="w-full max-w-sm mx-auto">
        <div class="bg-white border border-slate-300 rounded-xl px-8 py-8 mb-4 card-container" class:content-hidden={!flipped}>
            <div>
                <slot name="front"></slot>
            </div>
            {#if flipped}
                <div class="mt-5 pt-5 border-t border-t-slate-300">
                    <slot name="back"></slot>
                </div>
            {/if}
        </div>
    </div>

    {#if flipped}
        <!-- Confidence Buttons -->
        <div class="flex flex-col w-min mx-auto">
            <p class="text-slate-700 mb-4">
                How well did you know this?
            </p>
            <div class="flex justify-center space-x-4">
                {#each [1, 2, 3, 4, 5] as confidence}
                    <button class="btn" on:click={() => dispatch('confidence', confidence)}>
                        {confidence}
                    </button>
                {/each}
            </div>
            <div class="flex justify-between text-sm text-slate-500 mt-2">
                <span>Not at all</span>
                <span>Perfectly</span>
            </div>
        </div>
    {:else}
        <!-- Show Answer Button -->
        <button on:click={() => flipped = true} class="btn">
            <slot name="show">Show Answer</slot>
        </button>
    {/if}
</div>
