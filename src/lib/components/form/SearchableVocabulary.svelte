<script>
    import SearchableCombobox from '$lib/components/form/SearchableCombobox.svelte';

    /**
     * @typedef {Object} Word
     * @property {string} id Vocabulary ID
     * @property {string} name Word.
     * @property {string} description Translation.
     */

    /** @type {Word[]} */
    export let selectedWords = [];

    /** @type {(string|number)[]} */
    export let excludeIds = [];

    /**
     * Fetch vocabulary entries from the DB.
     * @param {string} searchTerm
     * @returns {Promise<Word[]>}
     */
    async function fetchVocabulary(searchTerm) {
        const queryParams = new URLSearchParams({
            search: searchTerm.toLowerCase(),
            limit: String(10),
            exclude: JSON.stringify([
                ...selectedWords.map(w => w.id),
                ...excludeIds
            ]),
        });
        try {
            const response = await fetch('/builder/vocabulary/search?' + queryParams);
            const data = await response.json();
            return data.map((word) => ({
                id: word.id,
                name: word.word,
                description: word.translation,
            }));
        } catch (error) {
            console.error('Error fetching words:', error);
            return [];
        }
    }
</script>

<SearchableCombobox
    bind:selectedItems={selectedWords}
    fetchItems={fetchVocabulary}
    label="Add vocabulary"
    placeholder="Type to search for words"
/>
