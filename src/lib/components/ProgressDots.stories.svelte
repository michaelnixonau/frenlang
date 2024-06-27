<script context="module">
    import ProgressDots from './ProgressDots.svelte';

    export const meta = {
        title: 'ProgressDots',
        component: ProgressDots,
        args: {
            total: 5,
            current: 3,
        },
    }
</script>

<script>
    import { Story, Template } from "@storybook/addon-svelte-csf";
</script>

<Template let:args>
    <ProgressDots {...args} />
</Template>

<Story name="Default" args={{}} />
