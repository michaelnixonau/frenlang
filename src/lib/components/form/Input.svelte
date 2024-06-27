<script>
    export let type;
    export let name;
    export let value = null;
    export let placeholder = null;
    export let required = false;
</script>

<input
    type={type}
    name={name}
    required={required}
    value={value}
    placeholder={placeholder}
    class="w-full border border-gray-300 rounded-md py-2 px-3 focus:outline-none focus:border-sky-500 focus:ring focus:ring-sky-500 focus:ring-opacity-20"
>
