<script setup>
import { ref, watch, computed, onMounted, onUnmounted } from 'vue'
import DeleteModal from './components/DeleteModal.vue'

const srp_list_orig = ref([])
const srp_list_filtered = ref(undefined)
const zbory_list = ref([])
const deletingItem = ref(undefined)
const search = ref('')
const timer = ref(null)

const srp_list = computed(() => {
    return srp_list_filtered.value != undefined ? srp_list_filtered.value : srp_list_orig.value
})

onMounted(() => {
    fetch('/api/srp/zbory')
    .then(response => response.json())
    .then(d => zbory_list.value = d)
    .catch(err => console.error('Load all congregations list exception:', err))

    load_all_srp()
})

onUnmounted(() => {
    if(timer.value != null) {
        clearTimeout(timer.value)
        timer.value = null
    }
})

watch(search, (nv) => {
    if(timer.value != null)
        clearTimeout(timer.value)
    timer.value = setTimeout(() => {
        if(search.value.length) {
            nv = nv.toLowerCase()
            srp_list_filtered.value = srp_list_orig.value.filter((item) => {
                if(item.regnum1.toLowerCase().includes(nv)) return true
                if(item.regnum2?.toLowerCase().includes(nv)) return true
                if(item.regnum3?.toLowerCase().includes(nv)) return true
                if(item.pass_nr == nv) return true
                
                let tmp = zbor_info(item.zbor_id)
                if(tmp?.toLowerCase().includes(nv)) return true
                return false
            })
        }
        else
            srp_list_filtered.value = undefined
        timer.value = null
    }, 500)
})

function load_all_srp() {
    fetch('/api/srp/all')
    .then(response => response.json())
    .then(d => srp_list_orig.value = d)
    .catch(err => console.error('Load all SRP exception:', err))
}

function zbor_info(zbor_id) {
    let zbor = zbory_list.value.find((item) => item.id === zbor_id)
    if(zbor == undefined) return ""
    return zbor.lang + " - " + zbor.number + " - " + zbor.name
}

function onStartDelete(srp) {
    console.log('Start deleting:', srp)
    deletingItem.value = srp
}
function onDelete(srp) {
    console.log('Deleting:', srp)

    fetch(`/api/srp/delete/${deletingItem.value.id}`)
    .then(response => {
        if(response.status === 200) {
            srp_list_filtered.value = []
            srp_list_orig.value = []
            load_all_srp()
        }
    })
    .catch(err => console.error('Delete SRP exception:', err))
}
</script>

<template>
    <header>
        <div class="d-flex flex-row align-items-center">
            <img src="@/assets/Parking_icon.svg" width="40" height="40" />
            <span class="ms-2 title">Identyfikatory parkingowe - tryb moderatora</span>
        </div>
        <div>
            <div class="input-group ">
                <span class="input-group-text">
                    <i class="fa-solid fa-magnifying-glass" />
                </span>
                <input
                    v-model="search"
                    type="text"
                    class="form-control"
                    placeholder="Wpisz ciąg filtrowania" 
                />
                <button class="btn btn-secondary" @click="search = ''">
                    <i class="fa-solid fa-xmark" />
                </button>
            </div>
        </div>
    </header>

    <main class="mt-3">
        <table class="table table-dark table-bordered">
            <thead>
                <tr>
                    <th rowspan="2">#</th>
                    <th rowspan="2">Zbór</th>
                    <th rowspan="2">Nr identyfikatora</th>
                    <th colspan="3">Nr rejestracyjny pojazdu</th>
                    <th rowspan="2">Ops</th>
                </tr>
                <tr>
                    <th>Piątek</th>
                    <th>Sobota</th>
                    <th>Niedziela</th>
                </tr>
            </thead>
            <tbody v-if="srp_list_orig.length === 0">
                <tr>
                    <td colspan="7">
                        <div class="d-inline-flex flex-row align-items-center">
                            <div class="spinner-border" role="status" />
                            <div class="ms-3">Proszę czekać. Trwa ładowanie danych ...</div>
                        </div>
                    </td>
                </tr>
            </tbody>
            <tbody v-else>
                <tr v-for="(srp, index) in srp_list" :key="index">
                    <th>{{ index+1 }}</th>
                    <td>{{ zbor_info(srp.zbor_id) }}</td>
                    <td>{{ srp.pass_nr }}</td>
                    <template v-if="srp.regnum2?.length">
                        <td>{{ srp.regnum1 }}</td>
                        <td>{{ srp.regnum2 }}</td>
                        <td>{{ srp.regnum3 }}</td>
                    </template>
                    <template v-else>
                        <td colspan="3">{{ srp.regnum1 }}</td>
                    </template>
                    <td>
                        <button type="button"
                            class="btn btn-danger btn-sm" 
                            title="Kasowanie"
                            @click="onStartDelete(srp)"
                            data-bs-toggle="modal"
                            data-bs-target="#deleteModal"
                        >
                            <i class="fa-solid fa-trash" />
                        </button>
                    </td>
                </tr>
            </tbody>
        </table>

    </main>

    <DeleteModal
        id="deleteModal"
        :record="deletingItem"
        @ok="onDelete"
    />
</template>

<style scoped>
header {
    line-height: 1.5;
    place-items: center;
    display: flex;
    flex-direction: row;
    justify-content: space-between;
}

.title {
    font-size: 1.5rem;
}

td, th {
    vertical-align: middle;
}
</style>
