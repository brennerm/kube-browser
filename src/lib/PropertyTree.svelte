<script lang="ts">
	import type { PropertyInfo } from './schemaResolver';
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
			return `array<${prop.ref}>`;
		} else if (prop.isArray) {
			return 'array';
		} else if (prop.ref) {
			return prop.ref;
		}
		return prop.type;
	}

	function hasNestedContent(prop: PropertyInfo): boolean {
		return !!(prop.properties && prop.properties.length > 0);
	}
</script>

<div class="property-item mb-2" style="margin-left: {level * 1.5}rem">
	<div class="flex items-start gap-2">
		{#if hasNestedContent(property)}
			<button
				class="btn btn-xs btn-circle btn-ghost shrink-0"
				onclick={toggleExpand}
				aria-label={expanded ? 'Collapse' : 'Expand'}
			>
				{expanded ? '▼' : '▶'}
			</button>
		{:else}
			<div class="w-8 shrink-0"></div>
		{/if}

		<div class="flex-1 min-w-0">
			<div class="flex items-center gap-3 flex-wrap mb-1">
				<span class="font-mono font-semibold text-primary text-base {property.required ? 'text-error' : ''}">
					{property.name}
				</span>
				<span class="font-mono text-sm text-base-content/60 italic">
					{getTypeDisplay(property)}
				</span>
				{#if property.required}
					<span class="badge badge-error badge-sm">required</span>
				{/if}
			</div>

			{#if property.description}
				<div class="text-sm leading-relaxed text-base-content/70 whitespace-pre-wrap wrap-break-word mt-1">
					{property.description}
				</div>
			{/if}
		</div>
	</div>

	{#if expanded && hasNestedContent(property)}
		<div class="mt-2 pt-2 border-l-2 border-base-content/10 pl-2">
			{#each property.properties as nestedProp}
				<PropertyTree property={nestedProp} level={level + 1} />
			{/each}
		</div>
	{/if}
</div>
