<template>
    <div class="modal fade" tabindex="-1">
        <div class="modal-dialog">
            <div class="modal-content">
                <div class="modal-header">
                    <h5>Kasowanie</h5>
                </div>

                <div class="modal-body">
                    <h4>
                        <strong>Czy na pewno skasować ten wpis?</strong>
                    </h4>
                    <div class="mt-4">
                        <label class="form-label">
                            Potwierdź wpisując numer rejestracyjny pojazdu: <strong>{{ props.record?.regnum1 }}</strong>
                        </label>
                        <input class="form-control" v-model="confirmationData" type="text" />
                    </div>
                </div>

                <div class="modal-footer">
                    <button 
                        type="button"
                        class="btn btn-secondary"
                        data-bs-dismiss="modal"
                        aria-label="Zamknij"
                    >
                        Zamknij
                    </button>
                    <button
                        type="button"
                        class="btn btn-danger"
                        data-bs-dismiss="modal"
                        :disabled="!isOkEnabled"
                        @click="$emit('ok', props.record)"
                    >
                        <FontAwesomeIcon :icon="faTrash" /> Usuń
                    </button>
                </div>
            </div>
        </div>
    </div>
</template>


<script setup>
import { ref, computed } from 'vue'
import { FontAwesomeIcon } from '@fortawesome/vue-fontawesome';
import { faTrash } from '@fortawesome/free-solid-svg-icons';


defineEmits(['ok'])
const props = defineProps(['record'])
const confirmationData = ref('')

const isOkEnabled = computed(() => {
    return confirmationData.value === props.record?.regnum1
})
</script>
