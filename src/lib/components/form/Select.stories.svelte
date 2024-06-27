<script context="module">
    import Select from './Select.svelte';

    export const meta = {
        title: 'Select',
        component: Select,
        args: {
            options: [
                { value: 'noun', label: 'Noun' },
                { value: 'verb', label: 'Verb' },
                { value: 'adjective', label: 'Adjective' },
                { value: 'adverb', label: 'Adverb' },
                { value: 'pronoun', label: 'Pronoun' },
                { value: 'determiner', label: 'Determiner' },
                { value: 'preposition', label: 'Preposition' },
                { value: 'conjunction', label: 'Conjunction' },
                { value: 'interjection', label: 'Interjection' },
            ],
        },
    }
</script>

<script>
    import { Story, Template } from "@storybook/addon-svelte-csf";
</script>

<Template let:args>
    <Select {...args} />
</Template>

<Story name="Default" args={{}} />
