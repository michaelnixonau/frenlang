<script>
    import { page } from '$app/stores';

    let errorDescription = '';

    if (typeof location !== 'undefined') {
        errorDescription = location.hash.split('error_description=')[1].replaceAll('+', ' ') + '.';
        console.log(location);
    }
</script>

<div class="text-center container max-w-md mx-auto my-10">
    <h1
            class="text-xl font-semibold text-gray-900"
    >
        There was an error authenticating your account.
    </h1>

    <p
            class="mt-2 text-sm text-gray-500"
    >
        {#if errorDescription}
            {errorDescription}
        {:else}
            Please try again.
        {/if}
    </p>
</div>
