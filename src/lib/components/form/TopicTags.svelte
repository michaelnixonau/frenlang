<script>
    import SearchableCombobox from '$lib/components/form/SearchableCombobox.svelte';

    /** @type {import('./SearchableCombobox.svelte').Item[]} */
    export let topics = [];

    /**
     * Fetch topics from the server.
     * @param {string} searchTerm
     * @returns {Promise<import('./SearchableCombobox.svelte').Item[]>}
     */
    async function fetchTopics(searchTerm) {
        const queryParams = new URLSearchParams({
            search: searchTerm.toLowerCase(),
            limit: String(10),
            exclude: JSON.stringify([
                ...topics.map(topic => topic.id),
            ])
        });
        try {
            const response = await fetch('/builder/topics/search?' + queryParams);
            const data = await response.json();
            return data;
        } catch (error) {
            console.error(error);
            return [];
        }
    }
</script>

<SearchableCombobox
        bind:selectedItems={topics}
        fetchItems={fetchTopics}
        label="Add topic"
        placeholder="Type to search for topics"
/>

{#if topics.length > 0}
    <div class="flex flex-col gap-1 my-5">
        <span class="block text-sm mb-1 text-slate-500">Selected topics</span>
        <div class="flex flex-wrap gap-2">
            {#each topics as topic (topic.id)}
                <button
                        class="bg-sky-100 text-sky-900 px-2 py-1 rounded-md hover:bg-red-100 hover:text-red-900"
                        on:click={() => {
                        topics = topics.filter((t) => t.id !== topic.id);
                    }}
                >
                    {topic.name}
                </button>
            {/each}
        </div>
    </div>
{/if}
