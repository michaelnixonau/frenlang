<script>
    import {writable} from "svelte/store";
    import {createTable, Render, Subscribe} from "svelte-headless-table";
    import {addSortBy} from "svelte-headless-table/plugins";
    import {goto, invalidateAll} from "$app/navigation";

    export let data;
    $: ({ topics, topicSize, page, pageSize, orderBy, orderDir } = data);
    let topicsTableData = writable([]);
    $: updateTopicsTable(topics);

    /**
     * Constructs query params for a nav event.
     * @param {{orderBy: string, orderDir: string, page: number}} params
     * @returns {string}
     */
    function buildQuery(params) {
        const query = new URLSearchParams();
        for (const [key, value] of Object.entries(params)) {
            query.set(key, value.toString());
        }
        return query.toString();
    }

    // Keep track of changes to the entries
    let changes = {};

    function updateTopicsTable(topics) {
        topicsTableData.set(topics.map(topic => ({ id: topic.id, name: topic.name, description: topic.description })));
        changes = {};
    }

    const table = createTable(topicsTableData, {
        sort: addSortBy({
            disableMultiSort: true,
            serverSide: true,
        }),
    });

    const columns = table.createColumns([
        table.column({ header: 'Name', accessor: 'name' }),
        table.column({ header: 'Description', accessor: 'description' }),
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
    const { sortKeys } = pluginStates.sort;

    const queryParams = writable({});
    $: { queryParams.set({ page, orderBy, orderDir })}

    /** @type {string} */
    let prevLink;

    /** @type {string} */
    let nextLink;

    /** @type {string[]} */
    let pageLinks;
    $: {
        prevLink = `?${buildQuery({ ...$queryParams, page: page - 1 })}`;
        nextLink = `?${buildQuery({ ...$queryParams, page: page + 1 })}`;
        pageLinks = Array.from({ length: Math.ceil(topicSize / pageSize) }, (_, i) => {
            const pageNum = i + 1;
            return `?${buildQuery({ ...$queryParams, page: pageNum })}`;
        });
    }

    $: sortKeys.set([{ id: orderBy, order: orderDir }]);

    sortKeys.subscribe(value => {
        if (!orderBy || !orderDir) return;
        const id = value[0]?.id ?? 'id';
        const order = value[0]?.order ?? 'asc';
        if (id === orderBy && order === orderDir) return;
        goto(`?${buildQuery({ orderBy: id, orderDir: order, page })}`);
    });

    /**
     * Delete a topic.
     * @param {number} id
     */
    async function deleteTopic(id) {
        try {
            const response = await fetch('/builder/topics/' + id, {
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
</script>

<div class="flex justify-between">
    <h1 class="font-semibold text-3xl">Topics</h1>
    <div class="flex gap-2">
        <a href="/builder/vocabulary/new" class="btn btn-primary">Add New</a>
    </div>
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
                                    ⬇️
                                {:else if props.sort.order === 'desc'}
                                    ⬆️
                                {/if}
                            </th>
                        </Subscribe>
                    {/each}
                    <th class="px-6 py-3 text-right">Actions</th>
                </tr>
            </Subscribe>
        {/each}
        </thead>
        <tbody {...$tableBodyAttrs}>
        {#each $rows as row (row.id)}
            <Subscribe rowAttrs={row.attrs()} let:rowAttrs>
                <tr {...rowAttrs} class="bg-white border-b {changes[row.dataId] ? 'bg-sky-50' : ''}">
                    {#each row.cells as cell (cell.id)}
                        <Subscribe attrs={cell.attrs()} let:attrs>
                            <td {...attrs} class="px-6 py-4 font-medium text-gray-900 whitespace-nowrap">
                                <p
                                        contenteditable="true"
                                        on:input={e => {
                                            const id = row.dataId;
                                            const key = cell.id;
                                            const value = e.target.innerText;
                                            changes[id] = { ...changes[id], [key]: value };

                                            // If the change for this cell is the same as the original value, remove it
                                            const original = topics.find(topic => topic.id === id);
                                            let isDifferent = false;
                                            for (const [key, value] of Object.entries(changes[id])) {
                                                if (original[key] !== value) {
                                                    isDifferent = true;
                                                    break;
                                                }
                                            }
                                            if (!isDifferent) {
                                                delete changes[id];
                                            }
                                        }}
                                >
                                    <Render of={cell.render()} />
                                </p>
                            </td>
                        </Subscribe>
                    {/each}
                    <td class="px-6 py-4 whitespace-nowrap text-right text-sm font-medium">
                        {#if changes[row.dataId]}
                            <button
                                    class="text-sky-600 mr-2 hover:underline"
                                    on:click={async () => {
                                        try {
                                            const response = await fetch('/builder/topics/' + row.dataId, {
                                                method: 'PATCH',
                                                headers: {
                                                    'Content-Type': 'application/json',
                                                },
                                                body: JSON.stringify(changes[row.dataId]),
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
                                    }}
                            >
                                Save
                            </button>
                        {/if}
                        <button on:click={() => deleteTopic(row.dataId)} class="text-red-600 hover:underline">Delete</button>
                    </td>
                </tr>
            </Subscribe>
        {/each}
        </tbody>
    </table>
</div>

<!-- Pagination -->
{#if topicSize > pageSize}
    <div class="mx-auto w-min flex gap-1 text-sm">
        <a href={prevLink} class="btn" aria-disabled={page <= 1}>Previous</a>
        {#each Array.from({ length: Math.ceil(topicSize / pageSize) }, (_, i) => i + 1) as pageNum}
            <a href={pageLinks[pageNum - 1]} class="btn {pageNum === page ? 'bg-slate-100' : ''}">{pageNum}</a>
        {/each}
        <!--{#if vocabSize > page * pageSize}-->
        <a href={nextLink} class="btn" aria-disabled={topicSize <= page * pageSize}>Next</a>
        <!--{/if}-->
    </div>
{/if}
