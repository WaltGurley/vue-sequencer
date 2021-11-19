<template>
    <button
        class="trigger-btn"
        :class="{ triggered: triggerOn }"
        @mousedown="toggleTrigger"
    ></button>
</template>

<script>
export default {
    name: 'Trigger',
    props: {
        triggerInfo: Object,
        clearAllTriggers: Boolean
    },
    data() {
        return {
            triggerOn: false
        }
    },
    emits: ['triggerState'],
    watch: {
        clearAllTriggers(shouldClear) {
            console.log('in trigger')
            if (shouldClear) {
                this.triggerOn = false
            }
        }
    },
    methods: {
        toggleTrigger() {
            this.triggerOn = !this.triggerOn
            this.$emit("triggerState", this.triggerOn, this.triggerInfo)
        }
    }
}
</script>

<style scoped>
    .trigger-btn {
        width: 3rem;
        height: 3rem;
        border: none;
        background-color: darkgray;
        margin: 0.25rem;
    }

    .triggered {
        background-color: #333;
    }
</style>