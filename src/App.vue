<template>
    <v-app>
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
