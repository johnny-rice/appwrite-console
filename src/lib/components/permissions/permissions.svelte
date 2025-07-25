<script context="module" lang="ts">
    export type Permission = {
        create: boolean;
        read: boolean;
        update: boolean;
        delete: boolean;
    };

    export type PermissionsTypes = 'create' | 'read' | 'update' | 'delete';
</script>

<script lang="ts">
    import { Button } from '$lib/elements/forms';
    import { symmetricDifference } from '$lib/helpers/array';
    import { onMount } from 'svelte';
    import { writable } from 'svelte/store';
    import Actions from './actions.svelte';
    import Row from './row.svelte';
    import { Icon, Layout, Selector, Table, Typography } from '@appwrite.io/pink-svelte';
    import { IconPlus, IconX } from '@appwrite.io/pink-icons-svelte';
    import type { PinkColumn } from '$lib/helpers/types';
    import { Card } from '$lib/components';

    export let withCreate = false;
    export let hideOnClick = false;
    export let permissions: string[] = [];

    let showUser = false;
    let showTeam = false;
    let showLabel = false;
    let showCustom = false;

    const groups = writable<Map<string, Permission>>(new Map());

    onMount(() => {
        permissions.forEach(fromPermissionString);
        return groups.subscribe(() => {
            const current = exportRoles();
            if (symmetricDifference(current, permissions).length) {
                permissions = current;
            }
        });
    });

    function create(event: CustomEvent<string[]>) {
        for (const role of event.detail) {
            addRole(role);
        }

        showTeam = showUser = false;
    }

    function addRole(role: string) {
        if ($groups.has(role)) {
            return;
        }

        groups.update((n) => {
            n.set(role, {
                create: false,
                read: false,
                update: false,
                delete: false
            });

            return n;
        });
    }

    function fromPermissionString(permission: string): void {
        const type = permission.slice(0, permission.indexOf('('));
        const role = permission.slice(permission.indexOf('("') + 2, permission.indexOf('")'));

        groups.update((n) => {
            if (!n.has(role)) {
                n.set(role, {
                    create: false,
                    read: false,
                    update: false,
                    delete: false
                });
            }
            n.get(role)[type] = true;

            return n;
        });
    }

    function togglePermission(role: string, permission: PermissionsTypes): void {
        groups.update((n) => {
            n.get(role)[permission] = !n.get(role)[permission];

            return n;
        });
    }

    function deleteRole(role: string): void {
        groups.update((n) => {
            n.delete(role);

            return n;
        });
    }

    function exportRoles() {
        return [...$groups].reduce((prev, [role, permission]) => {
            ['create', 'read', 'update', 'delete'].forEach((type) => {
                if (permission[type] === true) {
                    prev.push(`${type}("${role}")`);
                }
            });

            return prev;
        }, []);
    }

    function sortRoles([a]: [string, Permission], [b]: [string, Permission]) {
        if ((a === 'any') !== (b === 'any')) {
            return a === 'any' ? -1 : 1;
        }
        if ((a === 'users') !== (b === 'users')) {
            return a === 'users' ? -1 : 1;
        }
        if ((a === 'guests') !== (b === 'guests')) {
            return a === 'guests' ? -1 : 1;
        }

        return a.localeCompare(b);
    }

    const columns: PinkColumn[] = [
        { id: 'role', width: { min: 80 } },
        { id: 'create', width: { min: 80 }, hide: !withCreate },
        { id: 'read', width: { min: 80 } },
        { id: 'update', width: { min: 80 } },
        { id: 'delete', width: { min: 80 } },
        { id: 'action', width: 40 }
    ];
</script>

{#if [...$groups]?.length}
    <div class="table-wrapper">
        <Table.Root {columns} let:root>
            <svelte:fragment slot="header" let:root>
                <Table.Header.Cell column="role" {root}>Role</Table.Header.Cell>
                {#if withCreate}
                    <Table.Header.Cell column="create" {root}>Create</Table.Header.Cell>
                {/if}
                <Table.Header.Cell column="read" {root}>Read</Table.Header.Cell>
                <Table.Header.Cell column="update" {root}>Update</Table.Header.Cell>
                <Table.Header.Cell column="delete" {root}>Delete</Table.Header.Cell>
                <Table.Header.Cell column="action" {root} />
            </svelte:fragment>
            {#each [...$groups].sort(sortRoles) as [role, permission] (role)}
                <Table.Row.Base {root}>
                    <Table.Cell column="role" {root}>
                        <Row {role} />
                    </Table.Cell>
                    <Table.Cell column="create" {root}>
                        <Selector.Checkbox
                            checked={permission.create}
                            on:change={() => togglePermission(role, 'create')} />
                    </Table.Cell>
                    <Table.Cell column="read" {root}>
                        <Selector.Checkbox
                            size="s"
                            checked={permission.read}
                            on:change={() => togglePermission(role, 'read')} />
                    </Table.Cell>
                    <Table.Cell column="update" {root}>
                        <Selector.Checkbox
                            size="s"
                            checked={permission.update}
                            on:change={() => togglePermission(role, 'update')} />
                    </Table.Cell>
                    <Table.Cell column="delete" {root}>
                        <Selector.Checkbox
                            size="s"
                            checked={permission.delete}
                            on:change={() => togglePermission(role, 'delete')} />
                    </Table.Cell>

                    <Table.Cell column="action" {root}>
                        <Button compact icon ariaLabel="delete" on:click={() => deleteRole(role)}>
                            <Icon icon={IconX} size="s" />
                        </Button>
                    </Table.Cell>
                </Table.Row.Base>
            {/each}
        </Table.Root>
    </div>

    <div>
        <Actions
            bind:showLabel
            bind:showCustom
            bind:showTeam
            bind:showUser
            {groups}
            {hideOnClick}
            on:create={create}
            let:toggle>
            <Button secondary on:click={toggle}>
                <Icon icon={IconPlus} slot="start" size="s" />
                Add role
            </Button>
        </Actions>
    </div>
{:else}
    <Card variant="secondary">
        <Layout.Stack direction="column" alignItems="center" gap="xl">
            <Actions
                bind:showLabel
                bind:showCustom
                bind:showTeam
                bind:showUser
                {groups}
                {hideOnClick}
                on:create={create}
                let:toggle>
                <Button secondary icon on:click={toggle}>
                    <Icon icon={IconPlus} size="s" />
                </Button>
            </Actions>
            <Typography.Text>Add a role to get started</Typography.Text>
        </Layout.Stack>
    </Card>
{/if}

<style lang="scss">
    .table-wrapper {
        scrollbar-width: none;
        -ms-overflow-style: none;

        &::-webkit-scrollbar {
            display: none;
        }
    }
</style>
