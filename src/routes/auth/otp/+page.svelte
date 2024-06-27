<script>
    import { page } from '$app/stores';
    import { goto } from '$app/navigation';
    import { createPinInput, melt } from '@melt-ui/svelte';
    import {onMount} from 'svelte';

    const otpLength = 6;
    let loading = false;
    let errorMessage = '';

    async function verify() {
        if (loading) {
            return;
        }

        const email = $page.data.email;
        loading = true;

        try {
            // Make API request to this page, but with POST method
            const response = await fetch(location.href, {
                method: 'POST',
                body: JSON.stringify({ email, otp, bonjour: 'hello' }),
                headers: {
                    'content-type': 'application/json'
                },
            });
            const result = await response.json();
            const { data, error } = result;
            console.log(result);
            if (error) {
                throw new Error('Unable to verify the One-Time Passcode.');
            }
            errorMessage = '';
            if (data.session) {
                goto('/', {
                    invalidateAll: true,
                });
            }
        } catch (error) {
            if (error instanceof Error) {
                errorMessage = error.message;
            }
        } finally {
            loading = false;
        }
    }

    onMount(() => {
        if (!$page.data.email) {
            goto('/');
        }
    });

    const {
        elements: { root, input },
        states: { value },
    } = createPinInput();

    $: otp = $value.join('');

    $: if (otp.length === otpLength) {
        verify();
    }
</script>

<div class="container mx-auto max-w-sm my-10">
    <h1 class="text-3xl font-semibold text-center">Enter your one-time passcode</h1>
    <p class="text-center py-2 text-slate-500">{$page.data.email || ''}</p>

    {#if errorMessage}
        <div class="bg-red-100 border border-red-300 text-red-700 px-4 py-3 my-4 rounded relative text-sm"
             role="alert">
            <strong class="font-semibold">Error:</strong>
            <span class="block sm:inline">{errorMessage}</span>
        </div>
    {/if}

    <form class="mt-8" on:submit|preventDefault={verify}>
        <div class="mb-4">
            <label for="otp" class="block text-sm mb-1 text-slate-500">One-Time Passcode</label>
            <!-- One-Time Passcode -->
            <div use:melt={$root} class="flex items-center justify-center gap-3">
                {#each Array.from({ length: otpLength }) as _}
                    <input
                        class="w-1/6 text-center"
                        use:melt={$input()}
                    />
                {/each}
            </div>
        </div>
        <div class="mb-4">
            <button type="submit"
                    disabled={loading}
                    class="w-full bg-sky-500 text-white font-semibold rounded-md p-2 border border-sky-600 hover:bg-sky-600 active:bg-sky-700 focus:outline-none focus:ring-sky-300 disabled:opacity-50 disabled:cursor-not-allowed">
                {#if loading}
                    Loading...
                {:else}
                    Sign In
                {/if}
            </button>
        </div>
    </form>
</div>

