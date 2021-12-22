<script>
	const tabs = ['Frontend', 'Backend', 'Fullstack']
	let selectedTab = tabs[0]
	let selectedTabClasses = 'bg-gray-100'
	let repos = []
	let filteredRepos = []
	let languages = new Set()
	let selectedLanguage = null
	let selectedLanguageClasses = 'bg-gray-100'

	$: (async () => {
		repos = await fetchData(selectedTab)
	})()

	$: {
		languages = new Set(repos.map(repo => repo.language))
	}

	$: {
		if (selectedLanguage) {
			filteredRepos = repos.filter(repo => repo.language === selectedLanguage)
		} else {
			filteredRepos = repos
		}
	}

	function selectTab(tab) {
		selectedTab = tab
	}

	function selectLanguage(language) {
		if (selectedLanguage == language) {
			selectedLanguage = ''
		} else {
			selectedLanguage = language
		}
	}

	async function fetchData(selectedTab) {
		const res = await fetch(`/data/${selectedTab.toLocaleLowerCase()}.json`)
		const data = await res.json()
		return data.result
	}
</script>

<svelte:head>
	<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/tailwindcss/2.2.19/tailwind.min.css">
</svelte:head>

<ul class="flex">
	{#each tabs as tab}
		<li
			class="p-4 text-xl border cursor-pointer bg-white {tab == selectedTab && selectedTabClasses}"
			on:click={() => selectTab(tab)}
		>
			{tab}
		</li>
	{/each}
</ul>

<div class="flex space-x-8">
	<ul class="shadow">
		{#each Array.from(languages) as language}
			<li
				on:click={selectLanguage(language)}
				class="p-4 cursor-pointer bg-white {language == selectedLanguage && selectedLanguageClasses}"
			>
				{language}
			</li>
		{/each}
	</ul>

	<ul class="space-y-8">
		{#each filteredRepos as repo}
			<li class="flex items-center bg-white shadow p-4 rounded justify-between">
				<div>
					<div class="flex items-center">
						<div class="text-2xl mr-2">{repo.libraries.join(' + ')}</div>
						{#if repo.frontendEnvironment}
							<div class="text-xs border p-1 uppercase border-blue-300 text-blue-400 rounded">{repo.frontendEnvironment}</div>
						{/if}
					</div>
					<a href="{repo.repositoryURL}" class="text-gray-500" target="_blank">{repo.repositoryURL}</a>
				</div>

				<div class="text-lg">{repo.language}</div>
			</li>
		{/each}
	</ul>
</div>
