<template>
    <v-app>
        <v-main>
            <v-container fill-height fluid class="align-start pa-3">
                <main-menu></main-menu>
                <nuxt />
            </v-container>
        </v-main>
        <v-footer v-if="shortkeys.length" app class="pt-0" inset>
            <ul
                class="shortkeys-list blue-grey--text text--lighten-1 text-body-2 pl-0"
            >
                <li
                    v-for="shortkey of shortkeys"
                    :key="shortkey.keys"
                    class="mr-6"
                >
                    <kbd class="mr-1">{{ shortkey.keys }}</kbd>
                    {{ shortkey.label }}
                </li>
            </ul>
        </v-footer>
        <global-snackbar />
    </v-app>
</template>

<script>
import { mapActions, mapGetters, mapState } from 'vuex';
import MainMenu from '../components/menus/MainMenu.vue';
import GlobalSnackbar from '../components/commons/GlobalSnackbar';

export default {
    name: 'default',
    components: { MainMenu, GlobalSnackbar },
    computed: {
        darkMode() {
            return this.auth.settingDarkMode;
        },
        ...mapState('auth', ['auth']),
        ...mapGetters('help', ['shortkeys'])
    },
    mounted() {
        this.$vuetify.theme.dark = this.auth.settingDarkMode;
    },
    async fetch() {
        this.getUser();
        this.getUsers();
    },
    methods: {
        ...mapActions('auth', ['getUser']),
        ...mapActions('users', ['getUsers'])
    },
    watch: {
        darkMode(newValue) {
            this.$vuetify.theme.dark = newValue;
        }
    }
};
</script>

<style scoped>
.shortkeys-list {
    list-style-type: none;
}
.shortkeys-list li {
    display: inline-block;
}
</style>
