<script lang="ts">
import { api, SessionSnapshot, Recording } from 'lib/api'
import { timeAgo } from 'lib/time'
import moment from 'moment'
import RelativeDate from 'RelativeDate.svelte';
import { onDestroy } from 'svelte';
import { link } from 'svelte-spa-router'
import { Alert, Button, FormGroup, Spinner } from 'sveltestrap'

export let params = { id: '' }

let error: Error|null = null
let session: SessionSnapshot|null = null
let recordings: Recording[]|null = null

async function load () {
    session = await api.getSession(params)
    recordings = await api.getSessionRecordings(params)
}

async function close () {
    api.closeSession(session!)
}

function getTargetDescription () {
    if (session?.target) {
        return `${session.target.name} (${session.target.ssh?.host}:${session.target.ssh?.port})`
    } else {
        return 'Not selected yet'
    }
}

load().catch(e => {
    error = e
})

const interval = setInterval(load, 1000)
onDestroy(() => clearInterval(interval))

</script>

{#if !session && !error}
    <Spinner />
{/if}

{#if error}
    <Alert color="danger">{error}</Alert>
{/if}

{#if session}
    <div class="page-summary-bar">
        <div>
            <h1>Session</h1>
            <div class="text-muted">
                {#if session.ended}
                    {moment.duration(moment(session.ended).diff(session.started)).humanize()} long, <RelativeDate date={session.started} />
                {:else}
                    {moment.duration(moment().diff(session.started)).humanize()}
                {/if}
            </div>
        </div>
        {#if !session.ended}
            <Button class="ms-auto" outline on:click={close}>
                Close now
            </Button>
        {/if}
    </div>

    <div class="row mb-4">
        <div class="col-12 col-md-6">
            <FormGroup floating label="User">
                {#if session.username}
                    <input type="text" class="form-control" readonly value={session.username} />
                {:else}
                    <input type="text" class="form-control" readonly value="Not logged in" />
                {/if}
            </FormGroup>
        </div>
        <div class="col-12 col-md-6">
            <FormGroup floating label="Target">
                <input type="text" class="form-control" readonly value={getTargetDescription()} />
            </FormGroup>
        </div>
    </div>

    {#if recordings?.length }
        <h3>Recordings</h3>
        <div class="list-group list-group-flush">
            {#each recordings as recording}
                <a
                    class="list-group-item list-group-item-action"
                    href="/recordings/{recording.id}"
                    use:link>
                    <div class="main">
                        <strong>
                            {recording.name}
                        </strong>
                        <small class="meta ms-auto">
                            {timeAgo(recording.started)}
                        </small>
                    </div>
                </a>
            {/each}
        </div>
    {/if}
{/if}

<style lang="scss">
.list-group-item {
    .main {
        display: flex;
        align-items: center;

        > * {
            margin-right: 20px;
        }
    }

    .meta {
        opacity: .75;
    }
}
</style>
