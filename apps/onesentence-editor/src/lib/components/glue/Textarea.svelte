<script lang="ts">
	import type { HTMLInputAttributes } from 'svelte/elements';
	interface $$Props extends HTMLInputAttributes {
		label?: string;
		name?: string;
	}
	export let label: string = '';
	export let name: string = '';
	export let value: string = '';
</script>

<div class="form-control">
	{#if label}
		<label class="label" for={name}>{label}</label>
	{/if}
	<textarea
		bind:value
		on:blur
		on:change
		on:click
		on:input
		on:contextmenu
		on:focus
		on:keydown
		on:keypress
		on:keyup
		on:mouseenter
		on:mouseleave
		on:mouseover
		on:paste
		{...$$restProps}
		{name}
		class={`${$$props.class} textarea h-24 w-full text-base placeholder:text-sm`}
	/>
</div>
