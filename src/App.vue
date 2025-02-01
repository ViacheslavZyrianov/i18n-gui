<script setup lang="ts">
import {ref, onMounted, type Ref, watch} from 'vue'

interface Locale {
  label: string
  isEnabled: boolean
}

interface Translation {
  text: string
  isEnabled: boolean
}

const i18nData = ref<Record<string, Record<string, Translation>>>({})
const i18nDataFiltered = ref<Record<string, Record<string, Translation>>>({})
const localesData: Ref<Record<string, Locale>> = ref({})
const filters = ref({
  isShowEnabled: true,
  isShowDisabled: true,
  searchByKey: "",
  searchByTranslation: {},
})

const fetchI18nData = async () => {
  try {
    const response = await fetch('/api/i18n.json')
    i18nData.value = await response.json()
    i18nDataFiltered.value = i18nData.value;
  } catch (error) {
    console.error('Error fetching i18n.json:', error)
  }
}

const fetchLocalesData = async () => {
  try {
    const response = await fetch('/api/locales.json')
    localesData.value = await response.json()
  } catch (error) {
    console.error('Error fetching locales.json:', error)
  }
}

const setFiltersSearchByTranslation = () => {
  Object.keys(localesData.value).forEach((localeKey) => {
    filters.value.searchByTranslation[localeKey] = ""
  })
}

const onSearchByTranslationInput = (localeKey: string, translationText: string) => {
  const filteredData = Object.fromEntries(
    Object.entries(i18nData.value).filter(([_, item]) => item.translations[localeKey].toLowerCase().includes(translationText.toLowerCase()))
  )
  i18nDataFiltered.value = translationText ? filteredData : i18nData.value;
}

const onSearchByKey = (searchKey: string) => {
  const filteredData = Object.fromEntries(
    Object.entries(i18nData.value).filter(([key, _]) =>
      key.toLowerCase().includes(searchKey.toLowerCase())
    )
  )
  i18nDataFiltered.value = searchKey ? filteredData : i18nData.value;
}

const filterByEnabledStatus = () => {
  const { isShowEnabled, isShowDisabled } = filters.value;
  const filteredData = Object.fromEntries(
    Object.entries(i18nData.value).filter(([_, item]) => {
      if (isShowEnabled && isShowDisabled) return true;
      if (isShowEnabled) return item.isEnabled;
      if (isShowDisabled) return !item.isEnabled;
      return false;
    })
  );
  i18nDataFiltered.value = filteredData;
};

const onResetFilters = () => {
  filters.value.isShowEnabled = true;
  filters.value.isShowDisabled = true;
  filters.value.searchByKey = ""
  setFiltersSearchByTranslation()
  i18nDataFiltered.value = i18nData.value;
}

watch(() => [
  filters.value.isShowEnabled,
  filters.value.isShowDisabled
], filterByEnabledStatus)

onMounted(async () => {
  await fetchI18nData()
  await fetchLocalesData()
  setFiltersSearchByTranslation()
})
</script>

<template>
  <div class="panel">
    <button>Add translation</button>
    <button>Save</button>
  </div>
  <table>
    <thead>
      <tr>
        <th>Enabled</th>
        <th>Key</th>
        <th v-for="(locale, localeKey) in localesData" :key="localeKey">
          {{ locale.label }}
        </th>
        <th>Action</th>
      </tr>
      <tr>
        <td>
          <label>Show Enabled&nbsp;<input v-model="filters.isShowEnabled" type="checkbox"></label>
          <br>
          <label>Show Disabled&nbsp;<input v-model="filters.isShowDisabled" type="checkbox"></label>
        </td>
        <td>
          <input
            v-model="filters.searchByKey"
            type="text"
            placeholder="Search by key"
            @input="onSearchByKey($event.target.value)"
          >
        </td>
        <td v-for="(_, searchByTranslationItemKey) in filters.searchByTranslation" :key="searchByTranslationItemKey">
          <input
            v-model="filters.searchByTranslation[searchByTranslationItemKey]"
            type="text"
            placeholder="Search by translation"
            @input="onSearchByTranslationInput(searchByTranslationItemKey, $event.target.value)"
          >
        </td>
        <td>
          <button @click="onResetFilters">Reset</button>
        </td>
      </tr>
    </thead>
    <tbody>
      <tr v-for="(i18nItem, i18nKey) in i18nDataFiltered" :key="i18nKey">
        <td>
          <input type="checkbox" :checked="i18nItem.isEnabled">
        </td>
        <td>
          {{i18nKey}}
        </td>
        <td v-for="(_, localeKey) in localesData" :key="localeKey">
          <input class="input" type="text" :value="i18nItem.translations[localeKey]">
        </td>
        <td>
          <button>
            Delete
          </button>
        </td>
      </tr>
    </tbody>
  </table>
</template>

<style scoped>
table {
  width: 100%;
  border-collapse: collapse;
  border: 1px solid #2c2c2c;

  tr, td, th {
    border: 1px solid #2c2c2c;
    padding: 0;
  }

  th, td {
    padding: 0.5rem;
  }
}

label {
  user-select: none;
  cursor: pointer;
}

.panel {
  display: flex;
  align-items: center;
  justify-content: flex-end;
  margin-bottom: 32px;
  column-gap: 16px;
}
</style>
