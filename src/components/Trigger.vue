<template>
    <button
        class="trigger-btn"
        :class="{ triggered: triggerOn }"
        @click="toggleTrigger"
    >{{ triggerInfo.instrument }}: {{ triggerInfo.step }} </button>
</template>

<script>
export default {
    name: 'Trigger',
    props: {
        triggerInfo: Object
    },
    data() {
        return {
            triggerOn: false
        }
    },
    emits: ['triggerState'],
    methods: {
        toggleTrigger() {
            this.triggerOn = !this.triggerOn
            this.$emit("triggerState", this.triggerOn, this.triggerInfo)
            // console.log(this.triggerInfo)
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