<script>
	import '../app.pcss';
    import { goto, invalidate } from '$app/navigation';
    import { onMount } from 'svelte';
    import { createAvatar } from '@dicebear/core';
    import { shapes } from '@dicebear/collection';
    import Logo from '../img/Logomark.png';
    import Toaster from '$lib/components/Toaster.svelte';
    import { createDropdownMenu, melt } from '@melt-ui/svelte';
    import { fly } from 'svelte/transition';

    export let data;
    $: ({ session, supabase } = data);

    $: svg = createAvatar(shapes, {
        seed: session?.user?.email,
    }).toDataUriSync();

    onMount(() => {
        const { data } = supabase.auth.onAuthStateChange((_, newSession) => {
            // console.log('session', session, newSession);
            if (!newSession) {
                /**
                 * Queue this as a task so the navigation won't prevent the
                 * triggering function from completing
                 */
                // setTimeout(() => {
                //     goto('/', { invalidateAll: true });
                // });
            }
            if (newSession?.expires_at !== session?.expires_at) {
                invalidate('supabase:auth');
            }
        });

        return () => data.subscription.unsubscribe();
    });

    const {
        elements: { menu, item, trigger, arrow },
        states: { open },
    } = createDropdownMenu();
</script>

<svelte:document class="h-full bg-white" />

<div class="flex flex-col min-h-screen">
    <header class="p-4 border-b">
        <div class="container mx-auto flex justify-between items-center">
            <a href="/" class="font-semibold text-xl flex items-center gap-1">
                <img src={Logo} alt="EduKits French" class="h-8 w-8 inline-block" />
                Frenlang <span class="text-sm text-gray-500 mt-1">By EduKits</span>
            </a>
            <nav class="flex items-center gap-3">
                {#if session}
                    <a href="/builder" class="mr-4">Builder</a>
                    <a href="/learn" class="mr-4">Learn</a>
                    <button use:melt={$trigger} class="mr-4 flex items-center gap-2">
                        <img src={svg} alt="Avatar" class="h-7 w-7 rounded-full inline-block" />
                        My Account
                    </button>
                    {#if $open}
                        <div
                                use:melt={$menu}
                                transition:fly={{ duration: 150, y: -10 }}
                                class="bg-white shadow-lg rounded-md border border-gray-200 z-10 min-w-40 flex flex-col p-1"
                        >
                            <div use:melt={$item}>
                                <a href="/profile" class="block py-1 px-2 rounded hover:bg-slate-200">Profile</a>
                            </div>
                            <div use:melt={$item}>
                                <a href="/sign-out" class="block py-1 px-2 rounded hover:bg-slate-200">Sign Out</a>
                            </div>
                        </div>
                    {/if}
                {:else}
                    <a href="/sign-in">Sign In</a>
                {/if}
            </nav>
        </div>
    </header>
    <div class="p-4">
        <main class="flex-auto container mx-auto">
            <slot></slot>
        </main>
    </div>
    <footer class="py-4 border-t mt-auto">
        <div class="container mx-auto text-center text-slate-500 text-xs">
            <p>&copy; Copyright {new Date().getFullYear()} <a href="https://edukits.co/" class="text-sky-600 hover:underline">EduKits International</a>. All rights reserved.</p>
        </div>
    </footer>
</div>

<Toaster />
