<template>
    <div id="app">
        <CollectionRepeat :list="list"/>
    </div>
</template>

<script>
    import CollectionRepeat from './components/collectionRepeat.vue'

    export default {
        name: 'App',
        components: {
            CollectionRepeat
        },
        data() {
            return {
                list: []
            }
        },
        mounted() {
            for (let i = 1; i < 10000; i++) {
                this.list.push(i);
            }
        }
    }
</script>

<style>
    #app {
        font-family: 'Avenir', Helvetica, Arial, sans-serif;
        -webkit-font-smoothing: antialiased;
        -moz-osx-font-smoothing: grayscale;
        text-align: center;
        color: #2c3e50;
    }

    body {
        margin: 0;
    }
</style>
