<script lang="ts">
    import { useClient, mergeThemes } from './client/index.js';
    import type { RenderResponse } from './contracts.js';
    import Render from './render.svelte';
    import { useDialogs } from './stores/dialogs.js';
    import { installOperations } from './operations/index.js';
    import { createContext } from './context/index.js';
    import { installAdapters } from './adapters/index.js';
    import { onMount, setContext } from 'svelte';
    import { generateColorVariables } from './utils/color/index.js';
    import { Toaster } from '../ui/sonner/index.js';
    import { Colors } from './styles.js';
    import { ModeWatcher } from 'mode-watcher';

    export let data: RenderResponse;

    const dialogs = useDialogs();
    const client = useClient();
    const context = createContext();

    installOperations(client);
    installAdapters(client);

    setContext('context', context);
    setContext('theme', data.theme);

    const colors = mergeThemes('colors', Colors);
    const style = generateColorVariables(colors);

    onMount(() => {
        document.body.style.cssText = style;
    });
</script>

<ModeWatcher />
<Toaster richColors position="top-right" closeButton />

{#each $dialogs as dialog}
    <Render context={dialog.context} props={{ [dialog.component]: dialog }} />
{/each}

<Render props={data.content} />
