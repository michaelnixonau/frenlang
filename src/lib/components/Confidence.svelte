<script>
    /** @type {number|undefined} */
    let confidence = undefined;

    const bgColors = [
        'bg-red-600',
        'bg-orange-500',
        'bg-amber-500',
        'bg-lime-500',
        'bg-sky-500',
    ];
</script>

How well did you know this?

<!-- Confidence scale from 1 to 5 -->
<div class="w-full flex justify-center items-center">
    {#each [1, 2, 3, 4, 5] as value}
        <button
            class={
                'px-4 py-2 m-1 rounded-lg text-white '
                + bgColors[value - 1]
            }
            on:click={() => confidence = value}
        >
            {value}
        </button>
    {/each}
</div>
