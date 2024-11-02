<script lang="ts">
  import type { SourceInfo as _SourceInfo } from "@paperback/types";
  import { X } from "lucide-svelte";
  type SourceInfo = _SourceInfo & {id: string, repository: string}
  type Props = {
    plugins: SourceInfo[];
    onClose: () => void;
    onConfirm: () => void;
  };

  const { plugins = [], onClose, onConfirm }: Props = $props();
</script>

<div
  id="install-modal"
  class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50"
  on:click={(e) => onClose()}
>
  <div
    class="bg-white rounded-lg w-full max-w-md mx-4 flex flex-col max-h-[80vh]"
  >
    <div class="p-6 border-b border-gray-200">
      <div class="flex justify-between items-center">
        <h2 class="text-xl font-semibold">Confirm Installation</h2>
        <button class="text-gray-400 hover:text-gray-600" on:click={onClose}>
          <X class="w-5 h-5" />
        </button>
      </div>
    </div>

    <div class="p-6 overflow-y-auto flex-1">
      <p class="text-gray-600 text-sm mb-4">
        The following plugins will be installed:
      </p>

      <div class="space-y-2">
        {#each plugins as plugin}
          <div class="p-2 bg-gray-50 rounded flex items-center space-x-3">
            <img
              src={plugin.icon}
              alt={plugin.name}
              class="w-6 h-6 object-cover rounded"
            />
            <div class="flex-1">
              <p class="text-sm font-medium">{plugin.name}</p>
              <p class="text-xs text-gray-500">{plugin.repository}</p>
              <p class="text-xs text-gray-400">v{plugin.version}</p>
            </div>
          </div>
        {/each}
      </div>
    </div>

    <div class="p-6 border-t border-gray-200">
      <div class="flex justify-end space-x-2">
        <button
          class="px-4 py-2 text-sm font-medium text-gray-700 bg-white border border-gray-200 rounded-lg hover:bg-gray-50"
          on:click={onClose}
        >
          Cancel
        </button>
        <button
          class="px-4 py-2 text-sm font-medium text-white bg-black rounded-lg hover:bg-gray-800"
          on:click={onConfirm}
        >
          Confirm Install
        </button>
      </div>
    </div>
  </div>
</div>
