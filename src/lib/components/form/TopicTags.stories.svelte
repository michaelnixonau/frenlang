<script context="module">
    import TopicTags from './TopicTags.svelte';

    export const meta = {
        title: 'TopicTags',
        component: TopicTags,
        args: {
            total: 5,
            current: 3,
            topics: [
                { id: 1, name: 'Topic 1' },
                { id: 2, name: 'Topic 2' },
                { id: 3, name: 'Topic 3' },
                { id: 4, name: 'Topic 4' },
                { id: 5, name: 'Topic 5' },
            ],
            availableTopics: [
                { id: 6, name: 'Topic 6' },
                { id: 7, name: 'Topic 7' },
                { id: 8, name: 'Topic 8' },
                { id: 9, name: 'Topic 9' },
                { id: 10, name: 'Topic 10' },
            ],
        },
    }
</script>

<script>
    import { Story, Template } from "@storybook/addon-svelte-csf";
</script>

<Template let:args>
    <TopicTags {...args} />
</Template>

<Story name="Default" args={{}} />
