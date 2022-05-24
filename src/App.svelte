<script>
	import Icon from '@iconify/svelte'
	import VirtualList from '@sveltejs/svelte-virtual-list'
	import { onMount } from 'svelte'
	import './main.css'

	// States
	let ref
	let iconSets = ['ic', 'fa-solid']
	let selectedIconSetPrefix = ''
	let q = ''
	let selectedIcon = ''
	let disabled = false;
  let iconJsons = [];

  (async () => {
    const jsons = await Promise.all(iconSets.map((key) => import(`../node_modules/@iconify/json/json/${key}.json`)))
    iconJsons = jsons
  })()  


	$: allIcons = iconJsons?.flatMap((iconSet) =>
		Object.keys(iconSet.icons).map((key) => ({
			name: `${iconSet.prefix}:${key}`,
			iconset: iconSet.prefix,
		})),
	) || []

  $: console.log(allIcons)

	$: filteredIconsByIconKey = allIcons
    .filter((icon) => !selectedIconSetPrefix || icon?.iconset.includes(selectedIconSetPrefix))
		.filter((icon) => !q || icon?.name.includes(q))

	onMount(() => {
		function initCustomElement() {
			try {
				window.CustomElement.init((element, context) => {
					// Setup with initial value and disabled state
					console.log(element, context)
					selectedIcon = element.value

					// Icon sets array
					if (element?.config?.iconsets) {
						iconSets = element.config.iconsets
					}
				})

				// React on disabled changed (e.g. when publishing the item)
				window.CustomElement.onDisabledChanged((_disabled) => {
					disabled = _disabled
				})

				window.CustomElement.setHeight(ref.clientHeight)
			} catch (err) {
				// Initialization with Kentico Custom element API failed (page displayed outside of the Kentico UI)
				console.error(err)
			}
		}
		initCustomElement()
	})
</script>

<main class="container mx-auto py-3" bind:this={ref}>
	<div class="mx-auto mb-6">
		<label class="mb-3 block">
			<span class="font-bold">Icon Set:</span>
			<select bind:value={selectedIconSetPrefix}>
				<option value="">All</option>
				{#each iconJsons as iconSet (iconSet.prefix)}
					<option value={iconSet.prefix}>{iconSet.default.info.name}</option>
				{/each}
			</select>
		</label>
		<input type="text" class="border rounded-sm px-2 py-3 w-full" placeholder="Search icon" bind:value={q} />
	</div>

	{#if selectedIcon}
		<div class="flex items-center gap-4 font-bold">
      <div>Selected icon: </div>
      <div class="flex items-center gap-2"><Icon icon={selectedIcon} width={36} height={36}></Icon>{selectedIcon}</div>
    </div>
	{/if}

	<hr class="my-6" />

	<VirtualList height="480px" items={filteredIconsByIconKey} let:item>
		<button
			on:click={() => {
				selectedIcon = item.name
				window.CustomElement.setValue(selectedIcon)
			}}
			aria-label={item.name}
			class={`h-12 flex gap-2 items-center ${item.name === selectedIcon ? 'bg-green-400/50' : ''}`}
			{disabled}
		>
			<Icon width={48} height={48} icon={item.name} />
			<span>{item.name}</span>
		</button>
	</VirtualList>
</main>
