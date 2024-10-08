<script lang="ts">
    import {
        Button,
        InputNumber,
        InputSelect,
        InputText,
        InputTags,
        FormList,
        InputSelectCheckbox,
        InputDateTime
    } from '$lib/elements/forms';
    import { createEventDispatcher, onMount } from 'svelte';
    import { tags, operators, addFilter, queries } from './store';
    import type { Column } from '$lib/helpers/types';
    import type { Writable } from 'svelte/store';
    import { TagList } from '.';

    // We cast to any to not cause type errors in the input components
    /* eslint  @typescript-eslint/no-explicit-any: 'off' */
    export let value: any = null;
    export let columns: Writable<Column[]>;
    export let columnId: string | null = null;
    export let arrayValues: string[] = [];
    export let operatorKey: string | null = null;
    export let singleCondition = false;

    $: column = $columns.find((c) => c.id === columnId) as Column;

    $: operatorsForColumn = Object.entries(operators)
        .filter(([, v]) => v.types.includes(column?.type))
        .map(([k]) => ({
            label: k,
            value: k
        }));

    $: operator = operatorKey ? operators[operatorKey] : null;
    $: {
        columnId;
        operatorKey = null;
    }

    $: isDisabled = !operator;

    onMount(() => {
        value = column?.array ? [] : null;
        if (column?.type === 'datetime') {
            const today = new Date();
            value = today.toISOString();
        }
    });

    function addFilterAndReset() {
        addFilter($columns, columnId, operatorKey, value, arrayValues);
        columnId = null;
        operatorKey = null;
        value = null;
        arrayValues = [];
        if (singleCondition) {
            queries.apply();
        }
    }

    const dispatch = createEventDispatcher<{
        clear: void;
        apply: { applied: number };
    }>();
    dispatch('apply', { applied: $tags.length });
</script>

<div>
    <form on:submit|preventDefault={addFilterAndReset}>
        <ul class="selects u-flex u-gap-8 u-margin-block-start-16 u-flex-vertical-mobile">
            <InputSelect
                id="column"
                options={$columns
                    .filter((c) => c.filter !== false)
                    .map((c) => ({
                        label: c.title,
                        value: c.id
                    }))}
                placeholder="Select column"
                bind:value={columnId} />
            <InputSelect
                id="operator"
                disabled={!column}
                options={operatorsForColumn}
                placeholder="Select operator"
                bind:value={operatorKey} />
        </ul>
        {#if column && operator && !operator?.hideInput}
            {#if column?.array}
                <FormList class="u-margin-block-start-8">
                    {#if column.format === 'enum'}
                        <InputSelectCheckbox
                            name="value"
                            bind:tags={arrayValues}
                            placeholder="Select value"
                            options={column?.elements?.map((e) => ({
                                label: e?.label ?? e,
                                value: e?.value ?? e,
                                checked: arrayValues.includes(e?.value ?? e)
                            }))}>
                        </InputSelectCheckbox>
                    {:else}
                        <InputTags
                            label="values"
                            showLabel={false}
                            id="value"
                            bind:tags={arrayValues}
                            placeholder="Enter values" />
                    {/if}
                </FormList>
            {:else}
                <ul class="u-margin-block-start-8">
                    {#if column.format === 'enum'}
                        <InputSelect
                            id="value"
                            bind:value
                            placeholder="Select value"
                            options={column?.elements?.map((e) => ({
                                label: e?.label ?? e,
                                value: e?.value ?? e
                            }))}
                            label="Value"
                            showLabel={false} />
                    {:else if column.type === 'integer' || column.type === 'double'}
                        <InputNumber id="value" bind:value placeholder="Enter value" />
                    {:else if column.type === 'boolean'}
                        <InputSelect
                            id="value"
                            placeholder="Select a value"
                            required={true}
                            options={[
                                { label: 'True', value: true },
                                { label: 'False', value: false }
                            ].filter(Boolean)}
                            bind:value />
                    {:else if column.type === 'datetime'}
                        {#key value}
                            <InputDateTime
                                id="value"
                                bind:value
                                label="value"
                                showLabel={false}
                                step={60} />
                        {/key}
                    {:else}
                        <InputText id="value" bind:value placeholder="Enter value" />
                    {/if}
                </ul>
            {/if}
        {/if}
        {#if !singleCondition}
            <Button text disabled={isDisabled} class="u-margin-block-start-4" noMargin submit>
                <i class="icon-plus" />
                Add condition
            </Button>
        {/if}
    </form>

    {#if !singleCondition}
        <ul class="u-flex u-flex-wrap u-cross-center u-gap-8 u-margin-block-start-16 tags">
            <TagList />
        </ul>
    {/if}
</div>

<style lang="scss">
    .selects {
        :global(> *) {
            flex: 1;
        }
    }

    .tags {
        :global(b) {
            font-weight: bold;
        }
    }
</style>
