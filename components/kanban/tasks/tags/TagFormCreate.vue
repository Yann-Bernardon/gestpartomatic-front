<template>
    <v-row>
        <v-col>
            <tag-form
                v-model="formData"
                :disabled="disabled"
                @submit="create"
            />
        </v-col>
        <v-col class="flex-grow-0 d-flex align-center px-0">
            <v-btn
                class="ml-1 pr-0"
                color="primary"
                :disabled="!formData.label"
                icon
                small
                @click="create"
            >
                <v-icon>
                    mdi-plus-circle-outline
                </v-icon>
            </v-btn>
        </v-col>
    </v-row>
</template>

<script>
import TagForm from './TagForm.vue';
export default {
    components: { TagForm },
    name: 'tag-form-create',
    props: {
        disabled: Boolean
    },
    data() {
        return {
            formData: {
                label: '',
                color: '#2196F3',
                isDark: true
            }
        };
    },
    methods: {
        create() {
            this.$emit('create', this.formData);
        }
    }
};
</script>
