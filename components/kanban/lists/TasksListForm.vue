<template>
    <v-form
        id="board-task-list-update-form"
        class="pa-2"
        @submit.prevent="submit"
    >
        <v-row>
            <v-col class="pb-0">
                <v-text-field
                    autofocus
                    class="white"
                    dense
                    hide-details
                    label="Titre"
                    v-model="formData.name"
                    outlined
                    ref="inputName"
                    required
                    @keyup.ctrl.enter="submit"
                />
            </v-col>
        </v-row>

        <v-row>
            <v-col>
                <v-select
                    class="white"
                    dense
                    hide-details
                    :items="progressionItems"
                    label="Niveau d'avancement"
                    v-model="formData.progression"
                    outlined
                    @keyup.ctrl.enter="submit"
                />
            </v-col>
        </v-row>

        <v-row>
            <v-col cols="6">
                <v-btn
                    block
                    color="primary"
                    depressed
                    :disabled="!formData.name"
                    type="submit"
                >
                    {{ submitLabel }}
                </v-btn>
            </v-col>
            <v-col cols="6">
                <v-btn block color="error" depressed @click="cancel">
                    Annuler
                </v-btn>
            </v-col>
        </v-row>
    </v-form>
</template>

<script>
export default {
    name: 'tasks-list-form',
    components: {},
    props: {
        listData: {
            type: Object,
            default: () => {
                return {
                    name: '',
                    progression: 'TODO'
                };
            }
        },
        submitLabel: {
            type: String,
            default: 'Valider'
        }
    },
    data() {
        return {
            formData: {
                progression: this.listData.progression,
                name: this.listData.name
            },
            progressionItems: [
                { text: 'À faire', value: 'TODO' },
                { text: 'En cours', value: 'IN_PROGRESS' },
                { text: 'Terminé', value: 'DONE' }
            ]
        };
    },
    methods: {
        submit() {
            this.$emit('submit', this.formData);
        },
        cancel() {
            this.$emit('cancel');
        }
    }
};
</script>
