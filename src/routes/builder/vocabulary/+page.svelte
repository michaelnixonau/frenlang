<script>
    import Select from '$lib/components/form/Select.svelte';
    import MaterialSymbolsSearch from '~icons/material-symbols/search';
    import MaterialSymbolsAdd from '~icons/material-symbols/add';
    import MaterialSymbolsChevronLeft from '~icons/material-symbols/chevron-left';
    import MaterialSymbolsChevronRight from '~icons/material-symbols/chevron-right';
    import {createTable, Render, Subscribe} from 'svelte-headless-table';
    import {addSortBy} from 'svelte-headless-table/plugins';
    import {writable} from "svelte/store";
    import {goto, invalidateAll} from '$app/navigation';
    import {wordTypes} from '$lib/french.js';
    import ArrowDown from '~icons/material-symbols/keyboard-arrow-down';
    import ArrowUp from '~icons/material-symbols/keyboard-arrow-up';

    export let data;
    $: ({ vocabulary, vocabSize, page, pageSize, orderBy, orderDir, wordType } = data);
    let vocabTableData = writable([]);
    $: vocabTableData.set(vocabulary.map(vocab => ({ id: vocab.id, word: vocab.word, translation: vocab.translation, word_type: vocab.word_type })));

    const table = createTable(vocabTableData, {
        sort: addSortBy({
            disableMultiSort: true,
            serverSide: true,
        }),
    });
    const columns = table.createColumns([
        table.column({ header: 'Word', accessor: 'word' }),
        table.column({ header: 'Translation', accessor: 'translation' }),
        table.column({ header: 'Type', accessor: 'word_type' }),
    ]);

    const {
        headerRows,
        rows,
        tableAttrs,
        tableBodyAttrs,
        pluginStates,
    } = table.createViewModel(columns, {
        rowDataId: (item) => item.id,
    });
    const {sortKeys} = pluginStates.sort;

    $: {
        sortKeys.set([{ id: orderBy, order: orderDir }]);
    }

    sortKeys.subscribe(value => {
        if (!orderBy || !orderDir) return;
        // If this matches the current URL, we don't need to navigate
        const id = value[0]?.id ?? 'id';
        const order = value[0]?.order ?? 'asc';
        if (id === orderBy && order === orderDir) return;
        goto(`?${buildQuery({ ...$queryParams, orderBy: id, orderDir: order, page })}`);
    });

    /**
     * Constructs query params for a nav event.
     * @param {{orderBy: string, orderDir: string, page: number, wordType: string}} params
     * @returns {string}
     */
    function buildQuery(params) {
        const query = new URLSearchParams();
        for (const [key, value] of Object.entries(params)) {
            query.set(key, value.toString());
        }
        return query.toString();
    }

    const queryParams = writable({});
    $: { queryParams.set({ page, orderBy, orderDir, wordType })}

    /** @type {string} */
    let prevLink;

    /** @type {string} */
    let nextLink;

    /** @type {string[]} */
    let pageLinks;
    $: {
        prevLink = page > 1 ? `?${buildQuery({ ...$queryParams, page: page - 1 })}` : null;
        nextLink = vocabSize > page * pageSize ? `?${buildQuery({ ...$queryParams, page: page + 1 })}` : null;
        pageLinks = Array.from({ length: Math.ceil(vocabSize / pageSize) }, (_, i) => {
            const pageNum = i + 1;
            return `?${buildQuery({ ...$queryParams, page: pageNum })}`;
        });
    }

    /**
     * Delete a vocabulary item.
     * @param {number} id
     */
    async function deleteVocabulary(id) {
        try {
            const response = await fetch('/builder/vocabulary/' + id, {
                method: 'DELETE',
            });
            const { error } = await response.json();
            if (error) {
                console.error(error);
                return;
            }
            await invalidateAll();
        } catch (e) {
            console.error(e);
        }
    }

    /**
     * Given an ID, returns an array of topic names.
     * @param {number} id
     * @returns {string[]}
     */
    function getTopics(id) {
        const vocab = vocabulary.find(v => v.id === id);
        if (!vocab) return [];
        return vocab.topics.map(topic => topic.name);
    }
</script>

<!-- Header -->
<div class="flex justify-between">
    <div class="flex flex-row gap-2 items-center">
        <h1 class="font-semibold text-3xl">Vocabulary</h1>
        <!-- Count -->
        <span class="bg-slate-200 text-slate-800 px-2 py-1 rounded-full text-xs">{vocabSize}</span>
    </div>
    <div class="flex gap-2">
        <input type="text" placeholder="Enter search term">
        <button class="btn flex items-center gap-1"><MaterialSymbolsSearch /> Search</button>
        <a href="/builder/vocabulary/new" class="btn btn-primary flex items-center gap-1"><MaterialSymbolsAdd /> Add New</a>
    </div>
</div>

<!-- Filter by word type -->
<div class="my-4 flex flex-col w-3/12">
    <Select
            labelTitle="Filter by Word Type"
            options={[
                { value: '', label: 'All Types' },
                ...Object.entries(wordTypes).map(([value, label]) => ({ value, label })),
            ]}
            bind:value={wordType}
            on:change={() => goto(`?${buildQuery({ ...$queryParams, wordType, page: 1 })}`)}
    />
</div>

<div class="relative overflow-x-auto my-5">
    <table {...$tableAttrs} class="w-full text-sm text-left rtl:text-right text-gray-500">
        <thead class="text-gray-700 border-b">
        {#each $headerRows as headerRow (headerRow.id)}
            <Subscribe rowAttrs={headerRow.attrs()} let:rowAttrs>
                <tr {...rowAttrs}>
                    {#each headerRow.cells as cell (cell.id)}
                        <Subscribe attrs={cell.attrs()} let:attrs props={cell.props()} let:props>
                            <th
                                    {...attrs}
                                    class="px-6 py-3 {props.sort.disabled ? 'cursor-default' : 'cursor-pointer hover:bg-slate-100'}"
                                    on:click={props.sort.toggle}
                            >
                                <Render of={cell.render()} />
                                {#if props.sort.order === 'asc'}
                                    <ArrowDown class="inline" />
                                {:else if props.sort.order === 'desc'}
                                    <ArrowUp class="inline" />
                                {/if}
                            </th>
                        </Subscribe>
                    {/each}
                    <th class="px-6 py-3">Topics</th>
                    <th class="px-6 py-3 text-right">Actions</th>
                </tr>
            </Subscribe>
        {/each}
        </thead>
        <tbody {...$tableBodyAttrs}>
        {#each $rows as row (row.id)}
            <Subscribe rowAttrs={row.attrs()} let:rowAttrs>
                <tr {...rowAttrs} class="bg-white border-b">
                    {#each row.cells as cell (cell.id)}
                        <Subscribe attrs={cell.attrs()} let:attrs>
                            <td {...attrs} class="px-6 py-2 font-medium text-gray-900 whitespace-nowrap">
                                <Render of={cell.render()} />
                            </td>
                        </Subscribe>
                    {/each}
                    <td>
                        {#each getTopics(row.dataId) as topic}
                            <span class="bg-sky-100 text-sky-600 px-2 py-1 rounded-full text-xs mr-1">{topic}</span>
                        {/each}
                    </td>
                    <td class="px-6 py-2 whitespace-nowrap text-right text-sm font-medium">
                        <a href="/builder/vocabulary/{row.dataId}/edit" class="text-sky-600 mr-2 hover:underline">Edit</a>
                        <button on:click={() => deleteVocabulary(row.dataId)} class="text-red-600 hover:underline">Delete</button>
                    </td>
                </tr>
            </Subscribe>
        {/each}
        </tbody>
    </table>
</div>

<!-- Pagination -->
{#if vocabSize > pageSize}
    <div class="mx-auto w-min flex gap-1 text-sm">
        <a href={prevLink} class="btn flex items-center" aria-disabled={page <= 1}><MaterialSymbolsChevronLeft /></a>
        {#each Array.from({ length: Math.ceil(vocabSize / pageSize) }, (_, i) => i + 1) as pageNum}
            <a href={pageLinks[pageNum - 1]} class="btn {pageNum === page ? 'bg-slate-100' : ''}">{pageNum}</a>
        {/each}
        <!--{#if vocabSize > page * pageSize}-->
            <a href={nextLink} class="btn flex items-center" aria-disabled={vocabSize <= page * pageSize}><MaterialSymbolsChevronRight /></a>
        <!--{/if}-->
    </div>
{/if}
