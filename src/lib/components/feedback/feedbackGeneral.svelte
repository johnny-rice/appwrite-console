<script lang="ts">
    import { InputTextarea } from '$lib/elements/forms';
    import { feedbackData } from '$lib/stores/feedback';
</script>

<InputTextarea
    required
    id="feedback"
    autofocus
    bind:value={$feedbackData.message}
    label="Tell us more about your experience"
    placeholder="Share your suggestions and feature requests..." />
