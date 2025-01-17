<template>
    <v-col class="col-kanban pb-0">
        <v-sheet class="fill-height" outlined rounded>
            <v-card
                class="d-flex flex-column fill-height grey lighten-4"
                flat
                @dblclick="showCreateTaskForm"
            >
                <v-card-title
                    v-if="!isUpdateFormVisible"
                    class="flex-grow-0 task-list-title py-1 px-2 text-body-2 font-weight-light text-uppercase"
                    :class="{ 'cursor-grab': isAdmin }"
                >
                    <h2 class="mr-8 text-body-2 font-weight-light">
                        {{ list.name }}
                        <span class="ml-4 grey--text text--darken-1">
                            {{ filteredList.length }}
                        </span>
                    </h2>

                    <v-spacer />

                    <v-btn icon small @click="showCreateTaskForm">
                        <v-icon size="18">
                            mdi-plus
                        </v-icon>
                    </v-btn>

                    <v-menu v-if="isAdmin" offset-y>
                        <template v-slot:activator="{ on, attrs }">
                            <v-btn icon small v-bind="attrs" v-on="on">
                                <v-icon>
                                    mdi-dots-horizontal
                                </v-icon>
                            </v-btn>
                        </template>

                        <v-list>
                            <v-list-item @click="showUpdateForm">
                                <v-list-item-title>
                                    Modifier
                                </v-list-item-title>
                            </v-list-item>
                            <v-list-item @click="showRemoveDialog">
                                <v-list-item-title>
                                    Supprimer
                                </v-list-item-title>
                            </v-list-item>
                            <v-list-item
                                v-if="!isArchiveVisible"
                                @click="showArchive"
                            >
                                <v-list-item-title>
                                    Afficher les tâches archivées
                                </v-list-item-title>
                            </v-list-item>
                            <v-list-item v-else @click="hideArchive">
                                <v-list-item-title>
                                    Masquer les tâches archivées
                                </v-list-item-title>
                            </v-list-item>
                        </v-list>
                    </v-menu>
                </v-card-title>

                <tasks-list-form-update
                    v-else
                    class="flex-grow-0"
                    :list-data="list"
                    @update="hideUpdateForm"
                    @cancel="hideUpdateForm"
                />

                <v-card-text class="flex-grow-1 px-2 pb-2">
                    <draggable
                        v-if="dataFetched"
                        class="fill-height"
                        :list="draggableTasksIds"
                        group="tasks"
                        @change="updateTasksOrder"
                    >
                        <tasks-list-card
                            v-for="taskId in filteredList"
                            :key="taskId"
                            :task-id="taskId"
                            class="mb-1"
                            @show="showTaskDetail"
                            @update="updateTask"
                            @dblclick.capture="console.log('plop')"
                        >
                            {{ taskId }}
                        </tasks-list-card>
                    </draggable>

                    <template v-else>
                        <v-sheet
                            class="mb-1"
                            outlined
                            rounded
                            v-for="taskId in filteredList"
                            :key="taskId"
                        >
                            <v-skeleton-loader type="list-item-two-line" />
                        </v-sheet>
                    </template>
                </v-card-text>
            </v-card>
        </v-sheet>

        <v-dialog v-model="isRemoveDialogVisible" max-width="400px">
            <v-card>
                <v-card-title class="text-h5 font-weight-light">
                    Supprimer une colonne
                </v-card-title>

                <v-card-text>
                    Vous allez supprimer la colonne
                    <strong>{{ list.name }}</strong> et toutes les tâches qui
                    s'y trouvent.
                </v-card-text>

                <v-card-actions>
                    <v-spacer></v-spacer>

                    <v-btn color="error" depressed @click="remove">
                        Supprimer
                    </v-btn>

                    <v-btn depressed @click="hideRemoveDialog">
                        Annuler
                    </v-btn>
                </v-card-actions>
            </v-card>
        </v-dialog>
    </v-col>
</template>

<script>
import { mapActions, mapState, mapGetters } from 'vuex';
import draggable from 'vuedraggable';
import TasksListCard from './TasksListCard.vue';
import TasksListFormUpdate from './TasksListFormUpdate.vue';

export default {
    name: 'tasks-list',
    components: {
        draggable,
        TasksListCard,
        TasksListFormUpdate
    },
    props: {
        listId: Number
    },
    data() {
        return {
            isUpdateFormVisible: false,
            isRemoveDialogVisible: false,
            isArchiveVisible: false
        };
    },
    computed: {
        list() {
            return this.lists[this.listId];
        },
        displayedList() {
            return this.list.tasksList.filter(taskId => {
                return (
                    this.tasks[taskId].isArchived === this.isArchiveVisible ||
                    this.isArchiveVisible
                );
            });
        },
        filteredList() {
            if (!this.isFilterSet()) {
                return this.displayedList;
            }

            const filteredList = this.displayedList.filter(taskId => {
                const matchingTags = this.filters.tags.every(tagId => {
                    return this.tasks[taskId].tags.includes(tagId);
                });
                if (!matchingTags) return false;

                const matchingUsers = this.filters.assignees.every(userId => {
                    return this.tasks[taskId].assignees.includes(userId);
                });
                if (!matchingUsers) return false;

                const matchingWords = this.filters.words.every(word => {
                    const concatTaskStrings =
                        this.tasks[taskId].title +
                        this.tasks[taskId].description;
                    return concatTaskStrings.toLowerCase().includes(word);
                });
                if (!matchingWords) return false;

                return true;
            });

            return filteredList;
        },
        draggableTasksIds() {
            return this.list.tasksList.map(taskId => taskId);
        },
        ...mapState('boards', ['lists', 'tasks', 'filters', 'dataFetched']),
        ...mapGetters('boards', ['isAdmin'])
    },
    methods: {
        showArchive() {
            this.isArchiveVisible = true;
        },
        hideArchive() {
            this.isArchiveVisible = false;
        },
        showCreateTaskForm() {
            this.$emit('showCreateTaskForm', { listId: this.listId });
        },
        showUpdateForm() {
            this.isUpdateFormVisible = true;
        },
        hideUpdateForm() {
            this.isUpdateFormVisible = false;
        },
        showRemoveDialog() {
            this.isRemoveDialogVisible = true;
        },
        hideRemoveDialog() {
            this.isRemoveDialogVisible = false;
        },
        showTaskDetail(taskId) {
            this.$emit('showTask', taskId);
        },
        remove() {
            this.removeList(this.listId).then(() => {
                this.hideRemoveDialog();
            });
        },
        updateTask(updatedTask) {
            this.$emit('updateTask', updatedTask);
        },
        updateTasksOrder(change) {
            const type = this.setChangeType(change);
            const changeData = {
                listId: this.listId,
                taskId: change[type].element,
                type,
                oldIndex: change[type].oldIndex,
                newIndex: change[type].newIndex
            };

            this.$emit('updateTasksOrder', changeData);
        },
        setChangeType(change) {
            // Déplacement dans la même liste
            if (change.hasOwnProperty('moved')) {
                return 'moved';
            }
            // Arrivée dans une nouvelle liste
            else if (change.hasOwnProperty('added')) {
                return 'added';
            }
            // Départ de l'ancienne liste
            else {
                return 'removed';
            }
        },
        isFilterSet() {
            return (
                !!this.filters.tags.length ||
                !!this.filters.assignees.length ||
                !!this.filters.words.length
            );
        },
        ...mapActions('boards', ['removeList'])
    }
};
</script>

<style scoped>
.col-kanban {
    min-width: 355px !important;
    max-width: 355px !important;
}
</style>
