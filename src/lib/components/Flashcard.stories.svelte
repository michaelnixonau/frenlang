<script context="module">
    import Flashcard from './Flashcard.svelte';

    export const meta = {
        title: 'Flashcard',
        component: Flashcard,
    }
</script>

<script>
    import { Story, Template } from "@storybook/addon-svelte-csf";
</script>

<Template let:args>
    <Flashcard {...args}>
        <div slot="front">I'm tired.</div>
        <div slot="back">Je suis fatigué.</div>
    </Flashcard>
</Template>

<Story name="Default" args={{}} />
