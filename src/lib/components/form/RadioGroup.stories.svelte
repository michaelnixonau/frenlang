<script context="module">
    import RadioGroup from './RadioGroup.svelte';

    export const meta = {
        title: 'RadioGroup',
        component: RadioGroup,
        args: {
            options: [
                { value: 'option1', label: 'Option one' },
                { value: 'option2', label: 'Option two' },
                { value: 'option3', label: 'Option three' },
            ],
        },
    }
</script>

<script>
    import { Story, Template } from "@storybook/addon-svelte-csf";
</script>

<Template let:args>
    <RadioGroup {...args} />
</Template>

<Story name="Default" args={{}} />
