<script setup>
import { ref, watch, computed, onMounted, onUnmounted } from 'vue'
import { FontAwesomeIcon } from '@fortawesome/vue-fontawesome';
import { faMagnifyingGlass, faXmark, faTrash, faWheelchair } from '@fortawesome/free-solid-svg-icons';
import DeleteModal from './components/DeleteModal.vue'
import TuraRadioGroup from './components/TuraGroup.vue'

const srp_list_orig = ref([])
const srp_list_filtered = ref(undefined)
const zbory_list = ref([])
const tury_list = ref([])
const deletingItem = ref(undefined)
const search = ref('')
const timer = ref(null)
const filter_tura = ref('ALL')

const srp_list = computed(() => {
    return srp_list_filtered.value != undefined ? srp_list_filtered.value : srp_list_orig.value
})

onMounted(() => {
    fetch('/api/config/all')
    .then(response => response.json())
    .then(d => {
        console.log("cfg:", d)
        tury_list.value = d.tury
    })
    .catch(err => console.error('Load config exception:', err))

    fetch('/api/srp/zbory')
    .then(response => response.json())
    .then(d => {
        console.log("zbory:", d)
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

function filtering(pattern, tid) {
    let list = []
    
    if(pattern.length > 0) {
        pattern = pattern.toLowerCase()

        list = srp_list_orig.value.filter((item) => {

            if(tid != "ALL") {
                const dep_tura = zbor_tura(item.congregation_id)?.tid
                if(dep_tura != tid)
                    return false
            }

            if(item.car1.regnum.toLowerCase().includes(pattern)) return true
            if(item.car2 && item.car2.regnum.toLowerCase().includes(pattern)) return true
            if(item.car3 && item.car3.regnum.toLowerCase().includes(pattern)) return true
            // if(item.pass_nr != pattern) return true
            
            const tmp = zbor_info(item.congregation_id)
            if(tmp.toLowerCase().includes(pattern)) return true
            return false
        })
    }
    else {
        list = srp_list_orig.value.filter((item) => {
            if(tid !== "ALL") {
                const dep_tura = zbor_tura(item.congregation_id)?.tid
                if(dep_tura != tid)
                    return false
            }
            return true
        })
    }

    list.sort((a,b) => {
        const dep_tura_a = zbor_tura(a.congregation_id).shortcut
        const dep_tura_b = zbor_tura(b.congregation_id).shortcut
        const tmp = dep_tura_a.localeCompare(dep_tura_b)
        if(tmp !== 0) return tmp

        const zb_name_a = zbor_name(a.congregation_id)
        const zb_name_b = zbor_name(b.congregation_id)
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

function zbor_info(congregation_id) {
    let zbor = zbory_list.value.find((item) => item.id === congregation_id)
    if(zbor == undefined) return ""
    return zbor.lang + " - " + zbor.number + " - " + zbor.name
}

function zbor_tura(congregation_id) {
    const zbor = zbory_list.value.find((item) => item.id === congregation_id)
    if(zbor == undefined) return undefined
    const tura = tury_list.value.find(item => item.tid === zbor.tura)
    return tura
}

function zbor_name(congregation_id) {
    let zbor = zbory_list.value.find((item) => item.id === congregation_id)
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
            <div class="ms-2">
                <div class="title">Identyfikatory parkingowe - niepełnosprawni</div>
                <div class="text-muted">Tryb moderatora</div>
            </div>
        </div>

        <TuraRadioGroup v-model="filter_tura" class="tura-radio-group" />

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
        <table class="table table-dark table-bordered table-hover table-sm">
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
                    <td colspan="8">
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
                    <td>
                        <span title="Pasażer lub kierowca ze szczególnymi ograniczeniami ruchowymi">
                            {{ zbor_info(srp.congregation_id) }}
                        </span>
                        <span title="Pasażer lub kierowca ze szczególnymi ograniczeniami ruchowymi">
                            <FontAwesomeIcon v-if="srp.smr" class="ms-1 text-warning" :icon="faWheelchair" />
                        </span>
                    </td>
                    <td>{{ zbor_tura(srp.congregation_id).shortcut }}</td>
                    <td>{{ srp.pass_nr }}</td>
                    <template v-if="'car2' in srp">
                        <td>
                            {{ srp.car1.regnum }}
                            <span v-if="srp.car1.lpg" class="ms-1 text-warning">(LPG)</span>
                        </td>
                        <td>
                            {{ srp.car2?.regnum }}
                            <span v-if="srp.car2?.lpg" class="ms-1 text-warning">(LPG)</span>
                        </td>
                        <td>
                            {{ srp.car3?.regnum }}
                            <span v-if="srp.car3?.lpg" class="ms-1 text-warning">(LPG)</span>                           
                        </td>
                    </template>
                    <template v-else>
                        <td colspan="3">
                            {{ srp.car1.regnum }}
                            <span v-if="srp.car1.lpg" class="ms-1 text-warning">(LPG)</span>
                        </td>
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
