<template>
  <div class="d-flex gap-3 justify-content-between">
    <div class="d-inline form-check">
      <label class="form-check-label d-flex justify-content-center align-items-center gap-2 flex-nowrap">
        <input class="form-check-input m-0" type="checkbox" v-model="isAllSelected" @click="toggleSelectAll">
        Select all
      </label>
    </div>
    <template v-for="groupName in Object.keys(checklistGroups)" :key="groupName">
      <div class="d-inline form-check">
        <label class="form-check-label d-flex justify-content-center align-items-center gap-2 flex-nowrap">
          <input class="form-check-input m-0" type="checkbox" :value="groupName" :id="`checkbox-selectHeader-${groupName}`" v-model="selectedGroups" @change="toggleGroup(groupName)">
          {{ groupName }}
        </label>
      </div>
    </template>
  </div>
  <div class="row mt-3">
    <div v-for="groupName in Object.keys(checklistGroups)" :key="groupName" class="col-6">
      <div v-for="checklist in checklistGroups[groupName]" :key="checklist.id">
        <label>
          <input type="checkbox" :value="checklist.id" v-model="selectedchecklistIds" @click="togglechecklist(checklist)">
          {{ checklist[props.titleKey] }}
        </label>
      </div>
      <br>
    </div>
  </div>
</template>

<script setup>
import { ref, watch, defineProps, defineEmits } from 'vue'

const props = defineProps({
  checklists: {
    type: Array,
    required: true
  },
  groupKey: {
    type: String,
    default: 'group_name'
  },
  titleKey: {
    type: String,
    default: 'translated_name'
  },
  modelValue: Array
});

const emit = defineEmits(['update:modelValue']);

const checklistGroups = ref([]);

watch(() => props.checklists, (newchecklists) => {
  checklistGroups.value = newchecklists.reduce((groups, checklist) => {
    if (!groups[checklist[props.groupKey]]) {
      groups[checklist[props.groupKey]] = []
    }
    groups[checklist[props.groupKey]].push(checklist)
    return groups
  }, [])
})

const selectedGroups = ref([]);
const selectedchecklistIds = ref([...props.modelValue]);

const isAllSelected = ref(false)

function togglechecklist(checklist) {
  const checklistIndex = selectedchecklistIds.value.findIndex((selectedchecklistId) => selectedchecklistId === checklist.id)
  if (checklistIndex === -1) {
    selectedchecklistIds.value.push(checklist.id)
  } else {
    selectedchecklistIds.value.splice(checklistIndex, 1)
  }
  const groupchecklistIds = checklistGroups.value[checklist[props.groupKey]].map(gf=>gf.id);

  if (groupchecklistIds.length === selectedchecklistIds.value.filter(id=> groupchecklistIds.includes(id)).length) {
    selectedGroups.value.push(checklist[props.groupKey]);
  } else {
    selectedGroups.value = selectedGroups.value.filter(g=> g != checklist[props.groupKey]);
  }

  if (Array.from(new Set(selectedGroups.value)).length === Object.keys(checklistGroups.value).length) {
    isAllSelected.value = true;
  } else {
    isAllSelected.value = false;
  }
console.log(selectedchecklistIds.value);
  emit('update:modelValue', selectedchecklistIds.value);
}


function toggleGroup(groupName) {
  const groupchecklists = checklistGroups.value[groupName]
  
const selectedGroupchecklists = groupchecklists.filter((checklist) => {
  return selectedchecklistIds.value.includes(checklist.id)
});

if (selectedGroupchecklists.length === groupchecklists.length) {
  selectedchecklistIds.value = selectedchecklistIds.value.filter((id) => {
    return !groupchecklists.map(f=>f.id).includes(id)
  });
  selectedGroups.value = selectedGroups.value.filter(g=> g != groupName);
} else {
  groupchecklists.forEach((checklist) => {
    if (!selectedchecklistIds.value.includes(checklist.id)) {
      selectedchecklistIds.value.push(checklist.id)
    }
  });
  selectedGroups.value.push(groupName);
}
if (Array.from(new Set(selectedGroups.value)).length === Object.keys(checklistGroups.value).length) {
  isAllSelected.value = true;
} else {
  isAllSelected.value = false;
}
  
  emit('update:modelValue', selectedchecklistIds.value)
}


function toggleSelectAll() {
  if (isAllSelected.value) {
    selectedchecklistIds.value = [];
    selectedGroups.value = [];
  } else {
    selectedchecklistIds.value = props.checklists.map(checklist => checklist.id);
    selectedGroups.value = Object.keys(checklistGroups.value);
  }

  emit('update:modelValue', selectedchecklistIds.value)
}


</script>
