<script lang="ts">
  import { X } from 'lucide-svelte';

  type Props = {
    onClose: () => void
    onSubmit: (url: string) => void
    error: string | null
    loading: boolean
  }

  const { onClose, onSubmit, error = null, loading = false }: Props = $props();
  let url = $state('');

  function handleSubmit(event: Event) {
    event.preventDefault();
    onSubmit(url);
  }
</script>

<div 
  id="repo-modal"
  class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50"
  on:click={(e) => onClose()}
>
  <div class="bg-white rounded-lg p-6 w-full max-w-md mx-4">
    <div class="flex justify-between items-center mb-4">
      <h2 class="text-xl font-semibold">Add Repository</h2>
      <button 
        class="text-gray-400 hover:text-gray-600"
        on:click={onClose}
      >
        <X class="w-5 h-5" />
      </button>
    </div>
    
    <p class="text-gray-600 text-sm mb-4">
      Enter the base URL of the plugin repository. The system will automatically append "versioning.json" if needed.
    </p>

    <form class="space-y-4" on:submit={handleSubmit}>
      <input
        type="url"
        bind:value={url}
        placeholder="https://example.com/repository"
        required
        class="w-full h-10 px-3 text-base border border-gray-200 rounded-lg focus:outline-none focus:ring-2 focus:ring-gray-200 focus:border-transparent"
      />

      {#if error}
        <div class="p-3 bg-red-50 border border-red-200 rounded-lg">
          <p class="text-sm text-red-600">{error}</p>
        </div>
      {/if}

      <div class="flex justify-end space-x-2 pt-2">
        <button 
          type="button"
          class="px-4 py-2 text-sm font-medium text-gray-700 bg-white border border-gray-200 rounded-lg hover:bg-gray-50"
          on:click={onClose}
        >
          Cancel
        </button>
        <button 
          type="submit"
          disabled={loading}
          class="px-4 py-2 text-sm font-medium text-white bg-black rounded-lg hover:bg-gray-800 disabled:opacity-50 disabled:cursor-not-allowed"
        >
          {loading ? 'Adding...' : 'Add Repository'}
        </button>
      </div>
    </form>
  </div>
</div>