<script lang="ts">
    import { base } from '$app/paths';
    import { page } from '$app/stores';
    import { CardContainer, GridItem1, Id } from '$lib/components';
    import { canWriteDatabases } from '$lib/stores/roles';
    import type { PageData } from './$types';
    export let data: PageData;
    export let showCreate = false;
    const project = $page.params.project;
</script>

<CardContainer
    showEmpty={$canWriteDatabases}
    total={data.databases.total}
    on:click={() => (showCreate = true)}
    event="database"
    service="databases">
    {#each data.databases.databases as database}
        <GridItem1 href={`${base}/project-${project}/databases/database-${database.$id}`}>
            <svelte:fragment slot="title">{database.name}</svelte:fragment>
            <Id value={database.$id}>{database.$id}</Id>
        </GridItem1>
    {/each}
    <svelte:fragment slot="empty">
        <p>Create a database</p>
    </svelte:fragment>
</CardContainer>
