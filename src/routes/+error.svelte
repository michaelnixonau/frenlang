<script>
    import { page } from '$app/stores';
</script>

<div class="text-center py-40">
    <h1 class="text-5xl font-semibold my-2">
        Error {$page.status}
    </h1>

    {#if $page.error}
        <p class="text-xl text-slate-500 my-2">{$page.error.message}</p>
    {/if}

    <a href="/" class="mt-4 text-sky-600 hover:underline">Go back home</a>
</div>
