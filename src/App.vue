<template>
    <v-app>
        <v-toolbar app>
            <v-toolbar-title class="headline">
                <span> Vue Sorting Visualizations </span>
                <span class="font-weight-light"> Demo App </span>
            </v-toolbar-title>
            <v-spacer></v-spacer>
            <v-btn flat href="https://github.com" target="_blank">
                <span class="mr-2">Latest Release</span>
            </v-btn>
        </v-toolbar>

        <v-content>
            <Visualization/>
        </v-content>
    </v-app>
</template>

<script>
    import Visualization from './components/Visualization'

    export default {
        name: 'App',
        components: {
            Visualization
        },
        data() {
            return {
                //
            }
        }
    }
</script>
