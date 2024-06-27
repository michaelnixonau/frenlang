<script>
    import Matching from "$lib/components/Matching.svelte";
</script>

<div class="max-w-[600px] mx-auto">
    <Matching
            sourceWords={[
        { en: "one", fr: "un" },
        { en: "two", fr: "deux" },
        { en: "three", fr: "trois" },
        { en: "four", fr: "quatre" },
        { en: "five", fr: "cinq" },
        { en: "six", fr: "six" },
        { en: "seven", fr: "sept" },
        { en: "eight", fr: "huit" },
        { en: "nine", fr: "neuf" },
        { en: "ten", fr: "dix" },
    ]}
    />
</div>
