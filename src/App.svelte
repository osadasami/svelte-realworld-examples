<script>
  const localDataKey = 'svelte-realworld-examples'
  const tabs = ['Frontend', 'Backend', 'Fullstack']
  const stars = JSON.parse(localStorage.getItem(localDataKey)) || {}

  let selectedTab = tabs[0]
  let selectedTabClasses = 'bg-gray-100'
  let repos = []
  let filteredRepos = []
  let languages = new Set()
  let selectedLanguage = null
  let selectedLanguageClasses = 'bg-gray-100'

  $: {
    localStorage.setItem(localDataKey, JSON.stringify(stars))
  }

  $: (async () => {
    repos = fetchData(selectedTab)
  })()

  $: {
    repos.then((data) => {
      languages = new Set(data.map((repo) => repo.language))
    })
  }

  $: {
    repos.then((data) => {
      if (selectedLanguage) {
        filteredRepos = data.filter(
          (repo) => repo.language === selectedLanguage
        )
      } else {
        filteredRepos = data
      }
    })
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

  function getRepoNameFromUrl(repoUrl) {
    return repoUrl.replace('https://github.com/', '')
  }

  function fetchRepoInfo(repoName) {
    return fetch(`http://api.github.com/repos/${repoName}`)
  }

  async function getStars(repo) {
    const repoName = getRepoNameFromUrl(repo.repositoryURL)

    if (stars[repoName]) return

    const res = await fetchRepoInfo(repoName)
    const data = await res.json()

    stars[repoName] = data.stargazers_count
  }

  function starsCount(repo) {
    const repoName = getRepoNameFromUrl(repo.repositoryURL)
    return stars[repoName]
  }
</script>

<svelte:head>
  <link
    rel="stylesheet"
    href="https://cdnjs.cloudflare.com/ajax/libs/tailwindcss/2.2.19/tailwind.min.css"
  />
</svelte:head>

<ul class="flex">
  {#each tabs as tab}
    <li
      class="p-4 text-xl border cursor-pointer bg-white {tab == selectedTab &&
        selectedTabClasses}"
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
        class="p-4 cursor-pointer bg-white {language == selectedLanguage &&
          selectedLanguageClasses}"
      >
        <div>
          {language}
        </div>
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
              <div
                class="text-xs border p-1 uppercase border-blue-300 text-blue-400 rounded"
              >
                {repo.frontendEnvironment}
              </div>
            {/if}
          </div>
          <a href={repo.repositoryURL} class="text-gray-500" target="_blank"
            >{repo.repositoryURL}</a
          >
        </div>

        <div class="text-lg">{repo.language}</div>

        <div>
          {#if starsCount(repo)}
            <div>{starsCount(repo)}</div>
          {/if}
          <button
            class="p-2 text-xs bg-blue-100 rounded border"
            on:click={getStars(repo)}>Get stars</button
          >
        </div>
      </li>
    {/each}
  </ul>
</div>
