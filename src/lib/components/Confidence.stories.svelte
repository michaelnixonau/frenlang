<script context="module">
    import Confidence from "./Confidence.svelte";

    export const meta = {
        title: 'Confidence',
        component: Confidence,
    }
</script>

<script>
    import { Story, Template } from "@storybook/addon-svelte-csf";
</script>

<Template let:args>
    <Confidence {...args} />
</Template>

<Story name="Default" args={{}} />
