<script lang="ts">
	import { formatDefinitionName, type PropertyInfo } from './schemaResolver';
	import PropertyTree from './PropertyTree.svelte';

	interface Props {
		property: PropertyInfo;
		level?: number;
	}

	let { property, level = 0 }: Props = $props();

	let expanded = $state(level < 2); // Auto-expand first 2 levels

	function toggleExpand() {
		expanded = !expanded;
	}

	function getTypeDisplay(prop: PropertyInfo): string {
		if (prop.isArray && prop.ref) {
			return `${formatDefinitionName(prop.ref)}[]`;
		} else if (prop.isArray) {
			return '[]';
		} else if (prop.ref) {
			return formatDefinitionName(prop.ref);
		}
		return prop.type;
	}

	function hasNestedContent(prop: PropertyInfo): boolean {
		return !!(prop.properties && prop.properties.length > 0);
	}
</script>

<div class="property-item mb-2" style="margin-left: {level * 0.5}rem">
	<div class="flex items-start gap-2">
		{#if hasNestedContent(property)}
			<button
				class="btn btn-circle shrink-0 btn-ghost btn-xs"
				onclick={toggleExpand}
				aria-label={expanded ? 'Collapse' : 'Expand'}
			>
				<span class="transition-transform duration-300" class:-rotate-90={!expanded}>▼</span>
			</button>
		{:else}
			<div class="w-8 shrink-0"></div>
		{/if}

		<div class="min-w-0 flex-1">
			<div class="mb-1 flex flex-wrap items-center gap-3">
				<span class="font-mono text-base font-semibold">
					{property.name}
				</span>
				{#if property.required}
					<span class="badge badge-xs font-bold badge-error">required</span>
				{/if}

				<span
					class="grow justify-self-end text-right font-mono text-xs wrap-anywhere text-base-content/60 italic"
				>
					{getTypeDisplay(property)}
				</span>
			</div>

			{#if property.description}
				<div
					class="mt-1 text-sm leading-relaxed wrap-anywhere whitespace-pre-wrap text-base-content/70"
				>
					{property.description}
				</div>
			{/if}
		</div>
	</div>

	{#if expanded && hasNestedContent(property)}
		<div class="mt-2 border-l-2 border-base-content/10 pt-2 pl-2">
			{#each property.properties as nestedProp}
				<PropertyTree property={nestedProp} level={level + 1} />
			{/each}
		</div>
	{/if}
</div>
