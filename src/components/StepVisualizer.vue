<template>
    <div :class="{ 'current-step': isCurrentStep }" class="step-vis">
        <p style="margin: 0;">{{ stepNumber }}</p>
    </div>
</template>

<script>
export default {
    name: "StepVisualizer",
    props: {
        isCurrentStep: Boolean,
        stepNumber: Number
    }
}
</script>

<style scoped>
    .step-vis {
        width: 3rem;
        height: 3rem;
        background-color: #ddd;
        margin: 0.25rem;
        border-radius: 2rem;
        display: flex;
        align-items: center;
        justify-content: center;
    }
    .current-step {
        background-color: hotpink;
    }
</style>