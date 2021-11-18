<template>
    <div :class="{ 'current-step': isCurrentStep }" class="step-vis"></div>
</template>

<script>
export default {
    name: "StepVisualizer",
    props: {
        isCurrentStep: Boolean
    }
}
</script>

<style scoped>
    .step-vis {
        width: 3rem;
        height: 3rem;
        border: none;
        background-color: lightgray;
        margin: 0.25rem;
    }
    .current-step {
        background-color: hotpink;
    }
</style>