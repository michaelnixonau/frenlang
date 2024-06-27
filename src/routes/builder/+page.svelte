<script>
    export let data;
</script>

<h1 class="font-semibold text-3xl">Dashboard</h1>

<div class="grid grid-cols-3 gap-3 mt-4">
    <!-- Topics -->
    <a class="btn block p-5" href="/builder/topics">
        <h2 class="font-semibold text-xl">Topics</h2>
        <span class="text-gray-500 text-2xl">{data.topicSize}</span>
    </a>

    <!-- Vocabulary -->
    <a class="btn block p-5" href="/builder/vocabulary">
        <h2 class="font-semibold text-xl">Vocabulary</h2>
        <span class="text-gray-500 text-2xl">{data.vocabSize}</span>
    </a>
</div>
