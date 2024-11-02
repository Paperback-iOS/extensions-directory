<script lang="ts">
  import { onMount } from "svelte"
  import { Search } from "lucide-svelte"
  import RepositoryList from "./components/RepositoryList.svelte"
  import AddRepositoryModal from "./components/AddRepositoryModal.svelte"
  import InstallConfirmModal from "./components/InstallConfirmModal.svelte"
  import { type SourceInfo as _SourceInfo } from "@paperback/types"

  type SourceInfo = _SourceInfo & { id: string; repository: string }
  type SelectedPlugin = [string, string]

  let searchQuery = $state("")
  let selectedPlugins: SelectedPlugin[] = $state([])
  let allPlugins: SourceInfo[] = $state([])
  let loading = $state(true)
  let error: string | null = $state(null)

  let showAddRepo = $state(false)
  let showInstallConfirm = $state(false)
  let addingRepo = $state(false)
  let repoError = $state("")
  let repositories: string[] = $state([])

  async function validateRepository(url: string) {
    try {
      const baseUrl = url.endsWith("versioning.json")
        ? url
        : `${url.replace(/\/$/, "")}/versioning.json`
      const response = await fetch(baseUrl)

      if (!response.ok) {
        throw new Error("Invalid repository URL")
      }

      const data = await response.json()
      if (!data.sources || !Array.isArray(data.sources)) {
        throw new Error("Invalid repository format")
      }

      return baseUrl
    } catch (e) {
      throw new Error("Failed to validate repository")
    }
  }

  async function addRepository(url: string) {
    if (!url) return

    addingRepo = true
    repoError = ""

    try {
      const validatedUrl = await validateRepository(url)
      if (!repositories.includes(validatedUrl)) {
        repositories = [...repositories, validatedUrl]
        localStorage.setItem("repositories", JSON.stringify(repositories))
        await fetchPlugins()
        showAddRepo = false
      } else {
        repoError = "Repository already exists"
      }
    } catch (e) {
      repoError = (e as Error).message
    } finally {
      addingRepo = false
    }
  }

  async function removeRepository(url: string) {
    repositories = repositories.filter((repo) => repo !== url)
    localStorage.setItem("repositories", JSON.stringify(repositories))
    await fetchPlugins()
  }

  async function fetchPlugins() {
    try {
      const responses = await Promise.all(repositories.map((url) => fetch(url)))
      const jsonData = await Promise.all(responses.map((res) => res.json()))

      allPlugins = jsonData.flatMap((data, index) => {
        const repository = repositories[index].replace("versioning.json", "")
        return data.sources.map((source: SourceInfo) => ({
          id: source.id,
          name: source.name,
          version: source.version,
          description: source.description,
          contentRating: source.contentRating,
          developers: source.developers,
          badges: source.badges,
          capabilities: source.capabilities,
          repository: repository,
          icon: repository + source.id + "/static/" + source.icon,
        }))
      })
      loading = false
    } catch (e) {
      console.error("Error fetching plugins:", e)
      error = "Failed to load plugins. Please try again later." + e
      loading = false
    }
  }

  const filteredPlugins = $derived(
    allPlugins.filter(
      (plugin) =>
        plugin.name.toLowerCase().includes(searchQuery.toLowerCase()) ||
        plugin.description.toLowerCase().includes(searchQuery.toLowerCase()),
    ),
  )

  const selectedPluginDetails = $derived(
    selectedPlugins.map(
      ([id, url]) =>
        allPlugins.find(
          (plugin) => plugin.id === id && plugin.repository === url,
        )!,
    ),
  )

  function togglePlugin(pluginId: SelectedPlugin) {
    console.log(pluginId, selectedPlugins)
    let index = indexOf(pluginId)

    if (index > -1) {
      // only splice array when item is found
      selectedPlugins.splice(index, 1) // 2nd parameter means remove one item only
    } else {
      selectedPlugins.push(pluginId)
    }
  }

  function indexOf(pluginId: SelectedPlugin) {
    let index = -1
    let i = 0
    for (const [id, url] of selectedPlugins) {
      if (id === pluginId[0] && url === pluginId[1]) {
        index = i
      }
      i++
    }

    return index
  }

  function isSelected(pluginId: SelectedPlugin) {
    return indexOf(pluginId) >= 0
  }

  async function installSelectedPlugins() {
    console.log("Installing plugins:", selectedPluginDetails)
    showInstallConfirm = false
    selectedPlugins = []
  }

  onMount(() => {
    const savedRepositories = localStorage.getItem("repositories")
    if (savedRepositories) {
      repositories = JSON.parse(savedRepositories)
    }
    fetchPlugins()
  })
</script>

<div class="min-h-screen bg-white text-gray-900">
  <main class="container mx-auto px-4">
    <div class="max-w-3xl mx-auto pt-[20vh]">
      <h1 class="text-[32px] text-center font-semibold mb-6">
        Plugin Directory
      </h1>

      <RepositoryList
        {repositories}
        onAddClick={() => (showAddRepo = true)}
        onRemoveRepository={(url: string) => {
          removeRepository(url)
        }}
      />

      <!-- Search and Install Bar -->
      <div class="grid grid-cols-1 md:grid-cols-[1fr,auto] gap-4 mb-6">
        <div class="relative">
          <div
            class="absolute left-3 top-1/2 -translate-y-1/2 flex items-center space-x-2 text-gray-400"
          >
            <Search class="w-5 h-5" />
          </div>
          <input
            type="text"
            bind:value={searchQuery}
            placeholder="Search plugins..."
            class="w-full h-12 pl-12 pr-4 text-base border border-gray-200 rounded-lg focus:outline-none focus:ring-2 focus:ring-gray-200 focus:border-transparent"
          />
        </div>

        <button
          on:click={() => (showInstallConfirm = selectedPlugins.length > 0)}
          disabled={selectedPlugins.length === 0}
          class="h-12 px-6 text-sm font-medium text-white bg-black rounded-lg hover:bg-gray-800 disabled:opacity-50 disabled:cursor-not-allowed"
        >
          Install ({selectedPlugins.length})
        </button>
      </div>

      <!-- Plugin List -->
      {#if loading}
        <div class="text-center text-gray-600">Loading plugins...</div>
      {:else if error}
        <div class="text-center text-red-600">{error}</div>
      {:else if filteredPlugins.length > 0}
        <div class="space-y-4">
          {#each filteredPlugins as plugin}
            <div
              class="p-4 border border-gray-200 rounded-lg hover:border-gray-300 transition-colors cursor-pointer"
              class:bg-gray-100={isSelected([plugin.id, plugin.repository])}
              on:click={() => togglePlugin([plugin.id, plugin.repository])}
            >
              <div class="flex items-start">
                <img
                  src={plugin.icon}
                  alt={plugin.name}
                  class="w-12 h-12 object-cover rounded mr-4"
                />
                <div class="flex-grow">
                  <div class="flex items-center justify-between mb-1">
                    <h3 class="text-lg font-medium">{plugin.name}</h3>
                    <span class="text-sm text-gray-500">v{plugin.version}</span>
                  </div>
                  <p class="text-sm text-gray-600 mb-2">{plugin.description}</p>
                  <div class="flex flex-wrap items-center gap-2">
                    {#each plugin.badges as badge}
                      <span
                        class="px-2 py-1 text-xs font-semibold rounded-full"
                        style="background-color: {badge.backgroundColor}; color: {badge.textColor};"
                      >
                        {badge.label}
                      </span>
                    {/each}
                    {#if plugin.contentRating === "MATURE"}
                      <span
                        class="px-2 py-1 text-xs font-semibold text-red-600 bg-red-100 rounded-full"
                      >
                        MATURE
                      </span>
                    {/if}
                  </div>
                </div>
                <input
                  type="checkbox"
                  checked={isSelected([plugin.id, plugin.repository])}
                  class="w-5 h-5 border-gray-300 rounded text-black focus:ring-gray-200"
                  hidden
                />
              </div>
            </div>
          {/each}
        </div>
      {:else}
        <div class="text-center text-gray-600">
          No plugins found matching your search.
        </div>
      {/if}

      {#if showAddRepo}
        <AddRepositoryModal
          onClose={() => (showAddRepo = false)}
          onSubmit={addRepository}
          {error}
          loading={addingRepo}
        />
      {/if}

      {#if showInstallConfirm}
        <InstallConfirmModal
          plugins={selectedPluginDetails}
          onClose={() => (showInstallConfirm = false)}
          onConfirm={installSelectedPlugins}
        />
      {/if}
    </div>
  </main>
</div>

<style>
  :global(body) {
    margin: 0;
    font-family:
      system-ui,
      -apple-system,
      BlinkMacSystemFont,
      "Segoe UI",
      Roboto,
      sans-serif;
  }
</style>
