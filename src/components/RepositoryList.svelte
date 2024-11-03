<script lang="ts">
  import { onMount } from "svelte"
  import {
    ChevronDown,
    ChevronRight,
    Plus,
    Trash2,
    Share,
    Save,
  } from "lucide-svelte"

  type Props = {
    repositories: string[]
    readonly: boolean
    onShareClick: () => void
    onSaveClick: () => void
    onAddClick: () => void
    onRemoveRepository: (url: string) => void
  }

  const {
    repositories = [],
    readonly,
    onSaveClick,
    onShareClick,
    onAddClick,
    onRemoveRepository,
  }: Props = $props()
  let isExpanded = $state(true)

  onMount(() => {
    const savedState = localStorage.getItem("repositoryListExpanded")
    isExpanded = savedState === null ? true : savedState === "true"
  })

  function toggleExpanded() {
    isExpanded = !isExpanded
    localStorage.setItem("repositoryListExpanded", isExpanded.toString())
  }
</script>

<div class="mb-6 border border-gray-200 rounded-lg overflow-hidden">
  <div
    class="flex items-center justify-between p-4 bg-gray-50 cursor-pointer"
    on:click={toggleExpanded}
  >
    <div class="flex items-center space-x-2">
      {#if isExpanded}
        <ChevronDown class="w-5 h-5 text-gray-500" />
      {:else}
        <ChevronRight class="w-5 h-5 text-gray-500" />
      {/if}
      <h2 class="text-lg font-medium">Repositories</h2>
      {#if readonly}
        <span
          class="px-2 py-1 text-xs font-semibold text-red-600 bg-red-100 rounded-full"
        >
          READONLY
        </span>
      {/if}
    </div>
    {#if !readonly}
      <div class="flex items-center space-x-2">
        <button
          class="px-3 py-1.5 text-sm font-medium border border-gray-200 rounded-lg hover:bg-white transition-colors flex items-center"
          on:click|stopPropagation={onShareClick}
        >
          <Share class="w-4 h-4 mr-1.5" />
          Share
        </button>
        <button
          class="px-3 py-1.5 text-sm font-medium border border-gray-200 rounded-lg hover:bg-white transition-colors flex items-center"
          on:click|stopPropagation={onAddClick}
        >
          <Plus class="w-4 h-4 mr-1.5" />
          Add
        </button>
      </div>
    {:else}
      <button
        class="px-3 py-1.5 text-sm font-medium border border-gray-200 rounded-lg hover:bg-white transition-colors flex items-center"
        on:click|stopPropagation={onSaveClick}
      >
        <Save class="w-4 h-4 mr-1.5" />
        Save
      </button>
    {/if}
  </div>

  {#if isExpanded}
    <div class="p-4 space-y-2">
      {#each repositories as repo}
        <div class="flex items-center justify-between p-2 bg-gray-50 rounded">
          <span class="text-sm text-gray-600 truncate flex-1 mr-4">{repo}</span>
          <button
            class="p-1.5 text-gray-400 hover:text-red-500 rounded-lg hover:bg-white transition-colors"
            on:click={() => onRemoveRepository(repo)}
          >
            <Trash2 class="w-4 h-4" />
          </button>
        </div>
      {/each}
    </div>
  {/if}
</div>
