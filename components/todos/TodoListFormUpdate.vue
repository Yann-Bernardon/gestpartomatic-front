<template>
    <v-container class="pa-0">
        <div class="position-relative">
            <todo-list-form
                v-on="$listeners"
                icon="mdi-pencil-outline"
                ref="todoListForm"
                removable
                :todo-data="todo"
                @remove="startCountdown"
                @submit="submit"
            />

            <v-overlay absolute class="mb-1" :value="isOverlayVisible">
                <v-btn
                    color="error"
                    ref="abortRemoveButton"
                    @click="abortRemove"
                >
                    <v-icon left>
                        mdi-close-circle-outline
                    </v-icon>
                    Annuler la suppression
                </v-btn>
            </v-overlay>
        </div>

        <v-progress-linear
            background-color="white"
            bottom
            class="progress--no-transition"
            :color="progress.color || 'primary'"
            :value="Math.floor(progress.value)"
        ></v-progress-linear>
    </v-container>
</template>

<script>
import { mapGetters } from 'vuex';
import TodoListForm from './TodoListForm.vue';
import TodoCountdown from './mixins/TodoCountdown.mixin';

export default {
    name: 'todo-list-form-update',
    components: { TodoListForm },
    mixins: [TodoCountdown],
    props: {
        todoId: Number
    },
    data() {
        return {
            isOverlayVisible: false
        };
    },
    computed: {
        todo() {
            return this.todoById(this.todoId);
        },
        ...mapGetters('todos', ['todoById'])
    },
    methods: {
        async submit(updatedTodo) {
            // Découpage de la description en 2 données pour l'API
            const splittedDescription = updatedTodo.description.split('\n');
            const sentTitle = splittedDescription.shift();
            const sentDescription = splittedDescription.join('\n');

            const updatePayload = {
                title: sentTitle,
                description: sentDescription,
                deadline: updatedTodo.deadline
            };

            this.$store
                .dispatch('todos/update', {
                    updatePayload,
                    todoId: this.todoId
                })
                .then(() => {
                    this.$emit('close');
                });
        },
        startCountdown() {
            this.isOverlayVisible = true;

            this.$nextTick().then(() => {
                this.$refs.abortRemoveButton.$el.focus();
            });

            this.setCountDown('error', () => {
                this.isOverlayVisible = false;
                this.remove();
            });
        },
        async remove() {
            this.$store.dispatch('todos/remove', this.todoId);
        },
        abortRemove() {
            this.stopCountdown();
            this.isOverlayVisible = false;
            this.$refs.todoListForm.$refs.inputDescription.focus();
        }
    }
};
</script>

<style scoped>
.position-relative {
    position: relative;
}
</style>
