<template>
    <div>
        <div v-if="tury.length > 1" class="form-check">
            <input v-model="filter_tura" value="ALL" class="form-check-input" type="radio" name="tura" id="tura_all">
            <label class="form-check-label" for="tura_all">Wszystko</label>
        </div>
        
        <div v-for="(tura, index) in tury" :key="index" class="form-check form-check-inline">
            <input class="form-check-input" type="radio" name="tura" :id="`tura_${tura.shortcut}`" :value="tura.tid" v-model="modelValue">
            <label class="form-check-label" :for="`tura_${tura.shortcut}`">{{ tura.name }}</label>
        </div>
    </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'

const modelValue = defineModel({
    type: [String, Number],
    default: undefined
})

const tury = ref([])
onMounted(() => {
    fetch('/api/config/all')
        .then(res => res.json())
        .then(data => {
            tury.value = data.tury
            if(tury.value.length === 1)
                modelValue.value = tury.value[0].tid
        })
})
</script>