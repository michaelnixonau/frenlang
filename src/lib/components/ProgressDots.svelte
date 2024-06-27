<script>
    export let total = 0;
    export let current = 0;
</script>

<div class="flex justify-between gap-2 w-min mx-auto">
    {#each Array.from({ length: total }) as _, i}
        {#if i + 1 === current}
            <div class="w-3 h-3 border-2 border-sky-400 rounded-full"></div>
        {:else if i < current}
            <div class="w-3 h-3 bg-sky-400 rounded-full"></div>
        {:else}
            <div class="w-3 h-3 bg-slate-300 rounded-full"></div>
        {/if}
    {/each}
</div>
