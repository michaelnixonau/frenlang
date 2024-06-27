<script>
    import {page} from '$app/stores';
    import { goto } from '$app/navigation';
    import Logo from '../../img/Logomark.png';

    export let data;
    $: ({ supabase } = data);

    let email = '';
    let errorMessage = '';
    let loading = false;

    async function signInWithEmail() {
        if (loading) {
            return;
        }

        loading = true;

        try {
            const {data, error} = await supabase.auth.signInWithOtp({
                email: email,
                options: {
                    emailRedirectTo: $page.url.origin + '/auth/callback',
                }
            });

            if (error) {
                throw error;
            }

            goto('/auth/otp?email=' + encodeURIComponent(email));
        } catch (e) {
            errorMessage = e.message;
        } finally {
            loading = false;
        }
    }
</script>

<div class="container mx-auto max-w-sm my-10">
    <img src={Logo} alt="Logo" class="mx-auto w-14 mb-4"/>
    <h1 class="text-3xl font-semibold text-center">Sign in to your account</h1>
    <p class="text-center py-2 text-slate-500">Get started with your email below.</p>

    <form class="mt-8" on:submit|preventDefault={signInWithEmail}>
        {#if errorMessage}
            <div class="bg-red-100 border border-red-300 text-red-700 px-4 py-3 my-4 rounded relative text-sm"
                 role="alert">
                <strong class="font-semibold">Error:</strong>
                <span class="block sm:inline">{errorMessage}</span>
            </div>
        {/if}
        <div class="mb-4">
            <label for="email" class="block text-sm mb-1 text-slate-500">Email</label>
            <input
                    type="email"
                    name="email"
                    required={true} bind:value={email}
                    placeholder="you@example.com"
                    class="w-full"
            />
        </div>
        <div class="mb-4">
            <button
                    type="submit"
                    disabled={loading}
                    class="btn btn-primary w-full"
            >
                {#if loading}
                    Loading...
                {:else}
                    Continue with email
                {/if}
            </button>
        </div>
    </form>
</div>
