<script>
    import NewListDialog from '$lib/components/NewListDialog.svelte';
    import TimeAgo from 'javascript-time-ago';
    import en from 'javascript-time-ago/locale/en';
    import { onMount, onDestroy } from 'svelte';
    import ListAlt from '~icons/material-symbols/list-alt-sharp';
    import Square from '~icons/material-symbols/square-outline';

    export let data;
    $: ({ session, lists } = data);

    TimeAgo.addDefaultLocale(en);
    const timeAgo = new TimeAgo('en-US');

    // Create a reactive statement for the formatted times
    $: formattedTimes = lists.map(list => ({
        ...list,
        formattedTime: timeAgo.format(new Date(list.created_at))
    }));

    // Set up an interval to update the times
    let interval;
    onMount(() => {
        interval = setInterval(() => {
            formattedTimes = formattedTimes.map(list => ({
                ...list,
                formattedTime: timeAgo.format(new Date(list.created_at))
            }));
        }, 30000); // Update every 30 secs
    });

    // Clean up the interval when the component is destroyed
    onDestroy(() => {
        if (interval) clearInterval(interval);
    });
</script>

<svelte:head>
    <title>Frenlang by EduKits</title>
</svelte:head>

<div class="container max-w-sm mx-auto flex flex-col gap-5 my-20">
    <h1 class="text-6xl font-bold text-center">Frenlang</h1>
    <p class="text-center">Frenlang is a language learning platform that uses spaced repetition to help you learn a new language.</p>
    {#if !session}
        <a href="/sign-in" class="btn btn-primary text-center">Sign in to get started</a>
    {/if}
</div>

{#if session}
<!--    <div class="container grid grid-cols-1 gap-5 md:grid-cols-2">-->
    <div class="container">
        <div class="flex flex-col gap-5">
            <div class="flex items-center justify-between gap-5">
                <h2 class="text-4xl font-semibold">Your lists</h2>
                <NewListDialog invalidateAllOnCreation={true} />
            </div>
            <div class="grid gap-5 sm:grid-cols-1 md:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4">
                {#each formattedTimes as list}
                    <a class="btn block p-5 font-normal" href="/list/{list.id}">
                        <p class="flex items-center gap-1 text-slate-500 font-semibold">
                            {#if list.vocabulary_count > 0}
                                <ListAlt class="p-0 h-6 w-auto" /> {list.vocabulary_count}
                            {:else}
                                <Square class="p-0 h-6 w-auto" /> Empty List
                            {/if}
                        </p>
                        <h3 class="text-xl font-semibold">{list.name}</h3>
                        <p>{list.description}</p>
                        <p class="text-sm text-slate-400">Created {list.formattedTime}</p>
                    </a>
                {/each}
            </div>
        </div>
    </div>
{/if}
