<template>
    <div ref="dropdown" class="dropdown">
        <div @click="toggle"><slot name="toggle"></slot></div>
        <div v-if="show" class="open" @click="hide"><slot></slot></div>
    </div>
</template>

<script>
export default {
    data() {
        return {
            show: false
        };
    },
    methods: {
        toggle() {
            this.show = !this.show;
        },
        hide() {
            this.show = false;
        },
        backdrop(event) {
            if (this.show && this.$refs.dropdown && !this.$refs.dropdown.contains(event.target)) {
                this.show = false;
            }
        }
    },
    mounted() {
        document.addEventListener('click', this.backdrop);
    },
    beforeUnmount() {
        document.removeEventListener('click', this.backdrop);
    }
};
</script>