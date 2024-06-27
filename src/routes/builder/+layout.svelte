<script>
    import {page} from "$app/stores";

    const navPages = [{
        name: 'Dashboard',
        url: '/builder',
    }, {
        name: 'Vocabulary',
        url: '/builder/vocabulary',
    }, {
        name: 'Phrases',
        url: '/builder/phrases',
    }, {
        name: 'Topics',
        url: '/builder/topics',
    }, {
        name: 'Tools',
        url: '/builder/tools',
    }];

    console.log($page.url.pathname);
</script>

<div class="grid grid-cols-[min-content_1fr] gap-10">
    <!-- Sidebar -->
    <div class="sidebar">
        <nav class="flex flex-col min-w-40">
            {#each navPages as navPage}
                <a href={navPage.url} class={
                "py-2 px-3 mb-1 rounded-md hover:bg-slate-100"
                + ($page.url.pathname.endsWith(navPage.url) ? ' bg-slate-200' : '')
                }>
                    {navPage.name}
                </a>
            {/each}
        </nav>
    </div>

    <!-- Main -->
    <div class="main">
        <slot></slot>
    </div>
</div>
