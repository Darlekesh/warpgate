<script lang="ts">
import { api, Recording, RecordingKind } from 'lib/api'
import { Alert, Spinner } from 'sveltestrap'
import TerminalRecordingPlayer from 'player/TerminalRecordingPlayer.svelte'
import { onDestroy } from 'svelte';

export let params = { id: '' }

let error: Error|null = null
let recording: Recording|null = null

async function load () {
    recording = await api.getRecording(params)

}

function getTCPDumpURL () {
    return `/api/recordings/${recording?.id}/tcpdump`
}

load().catch(e => {
    error = e
})

</script>


<div class="page-summary-bar">
    <h1>Session recording</h1>
</div>

{#if !recording && !error}
<Spinner />
{/if}

{#if error}
<Alert color="danger">{error}</Alert>
{/if}

{#if recording?.kind === 'Traffic'}
    <a href={getTCPDumpURL()}>Download tcpdump file</a>
{/if}
{#if recording?.kind === 'Terminal'}
    <TerminalRecordingPlayer recording={recording} />
{/if}
