<script context="module" lang="ts">
    import type { Writable } from 'svelte/store';

    export type FormContext = {
        isSubmitting: Writable<boolean>;
    };
</script>

<script lang="ts">
    import { setContext } from 'svelte';
    import { writable } from 'svelte/store';

    export let noMargin = false;
    export let noStyle = false;
    export let isModal = false;
    export let onSubmit: (e: SubmitEvent) => Promise<void> | void;
    let classes = '';
    export { classes as class };

    let form: HTMLFormElement;

    export let { isSubmitting } = setContext<FormContext>('form', {
        isSubmitting: writable(false)
    });

    export function checkValidity() {
        return form.checkValidity();
    }
    export function triggerSubmit() {
        form.requestSubmit();
    }

    async function submit(e: SubmitEvent) {
        isSubmitting.set(true);
        await onSubmit(e);
        isSubmitting.set(false);
    }
</script>

<form
    bind:this={form}
    class={classes}
    class:form={!noStyle}
    class:common-section={!noMargin}
    class:modal-form={isModal}
    on:submit|preventDefault={submit}>
    <slot />
</form>
