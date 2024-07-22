<script setup>
import { ref, watch, computed, onMounted, onUnmounted } from 'vue'
import { FontAwesomeIcon } from '@fortawesome/vue-fontawesome';
import { faMagnifyingGlass, faXmark, faTrash } from '@fortawesome/free-solid-svg-icons';
import DeleteModal from './components/DeleteModal.vue'

const srp_list_orig = ref([])
const srp_list_filtered = ref(undefined)
const zbory_list = ref([])
const deletingItem = ref(undefined)
const search = ref('')
const timer = ref(null)
const filter_tura = ref('ALL')

const srp_list = computed(() => {
    return srp_list_filtered.value != undefined ? srp_list_filtered.value : srp_list_orig.value
})

onMounted(() => {
    fetch('/api/srp/zbory')
    .then(response => response.json())
    .then(d => {
        zbory_list.value = d
        load_all_srp()
    })
    .catch(err => console.error('Load all congregations list exception:', err))
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
        filtering(nv, filter_tura.value)
        timer.value = null
    }, 500)
})

watch(filter_tura, (nv) => {
    if(timer.value != null)
        clearTimeout(timer.value)
    timer.value = setTimeout(() => {
        filtering(search.value, nv)
        timer.value = null
    }, 500)
})

function filtering(pattern, tura) {
    let list = []

    if(pattern.length) {
        pattern = pattern.toLowerCase()
        list = srp_list_orig.value.filter((item) => {
            if(tura !== "ALL") {
                let dep_tura = zbor_tura(item.zbor_id)
                if(dep_tura !== tura)
                    return false
            }

            if(item.regnum1.toLowerCase().includes(pattern)) return true
            if(item.regnum2?.toLowerCase().includes(pattern)) return true
            if(item.regnum3?.toLowerCase().includes(pattern)) return true
            if(item.pass_nr == pattern) return true
            
            let tmp = zbor_info(item.zbor_id)
            if(tmp?.toLowerCase().includes(pattern)) return true
            return false
        })
    }
    else {
        list = srp_list_orig.value.filter((item) => {
            if(tura !== "ALL") {
                let dep_tura = zbor_tura(item.zbor_id)
                if(dep_tura !== tura)
                    return false
            }
            return true
        })
    }

    list.sort((a,b) => {
        let dep_tura_a = zbor_tura(a.zbor_id)
        let dep_tura_b = zbor_tura(b.zbor_id)
        let tmp = dep_tura_a.localeCompare(dep_tura_b)
        if(tmp !== 0) return tmp

        let zb_name_a = zbor_name(a.zbor_id)
        let zb_name_b = zbor_name(b.zbor_id)
        return zb_name_a.localeCompare(zb_name_b)
    })

    srp_list_filtered.value = list
}

function load_all_srp() {
    fetch('/api/srp/all')
    .then(response => response.json())
    .then(d => {
        srp_list_orig.value = d
        filtering(search.value, filter_tura.value)
    })
    .catch(err => console.error('Load all SRP exception:', err))
}

function zbor_info(zbor_id) {
    let zbor = zbory_list.value.find((item) => item.id === zbor_id)
    if(zbor == undefined) return ""
    return zbor.lang + " - " + zbor.number + " - " + zbor.name
}

function zbor_tura(zbor_id) {
    let zbor = zbory_list.value.find((item) => item.id === zbor_id)
    if(zbor == undefined) return ""
    return `W${zbor.tura}`
}

function zbor_name(zbor_id) {
    let zbor = zbory_list.value.find((item) => item.id === zbor_id)
    if(zbor == undefined) return ""
    return zbor.name
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
            <span class="ms-2 title">Identyfikatory parkingowe Lodowisko - tryb moderatora</span>
        </div>

        <div class="tura-radio-group">
            <div class="form-check">
                <input v-model="filter_tura" value="ALL" class="form-check-input" type="radio" name="tura" id="tura_all">
                <label class="form-check-label" for="tura_all">Wszystko</label>
            </div>
            <div class="form-check">
                <input v-model="filter_tura" value="W2" class="form-check-input" type="radio" name="tura" id="tura_w2">
                <label class="form-check-label" for="tura_w2">W2</label>
            </div>
            <div class="form-check">
                <input v-model="filter_tura" value="W3" class="form-check-input" type="radio" name="tura" id="tura_w3">
                <label class="form-check-label" for="tura_w3">W3</label>
            </div>
        </div>

        <div>
            <div class="input-group ">
                <span class="input-group-text">
                    <FontAwesomeIcon :icon="faMagnifyingGlass" />
                </span>
                <input
                    v-model="search"
                    type="text"
                    class="form-control"
                    placeholder="Wpisz ciąg filtrowania" 
                />
                <button class="btn btn-secondary" @click="search = ''">
                    <FontAwesomeIcon :icon="faXmark" />
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
                    <th rowspan="2">Tura</th>
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
                    <td>{{ zbor_tura(srp.zbor_id) }}</td>
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
                            <FontAwesomeIcon :icon="faTrash" />
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

.tura-radio-group {
    display: flex;
    flex-direction: row;
    gap: 20pt;
}
</style>
