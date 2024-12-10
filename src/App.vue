<template>
  <div>
    <h1>Sélectionnez les librairies à utiliser</h1>

    <label>
      <input type="checkbox" v-model="isSelectAll" @change="toggleSelectAll" />
      Tout cocher
    </label>

    <div class="checkbox-container">
      <div v-for="(lib, key) in libraries" :key="key">
        <label>
          <input type="checkbox" v-model="selectedLibraries" :value="key" />
          {{ key }}
        </label>
      </div>
    </div>

    <div class="download-container">
      <button class="button" @click="copyToClipboard(router)">Copier router/index.js</button>
      <button class="button" @click="copyToClipboard(confModules)">Copier conf.modules.js</button>
      <button class="button" @click="copyToClipboard(packageJson)">Copier package.json</button>
    </div>

    <h2>Router</h2>
    <pre>{{ router }}</pre>

    <h2>conf.modules.js</h2>
    <pre>{{ confModules }}</pre>

    <h2>package.json (dependencies)</h2>
    <pre>{{ packageJson }}</pre>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'

const libraries = ref({})
const isSelectAll = ref(false)
const selectedLibraries = ref([])

const confModules = computed(() => {
  let content = `module.exports = {\n  modules: {\n`
  selectedLibraries.value.forEach((lib) => {
    const isCustom = libraries.value[lib].package.includes('inside-customs')

    content += `    ${lib}: require('..${isCustom ? '' : '/node_modules/@maecia'}/${libraries.value[lib].package}${libraries.value[lib].path ? `/${libraries.value[lib].path}` : ''}').default,\n`
  })
  content += `  },\n  vuePlugins: []\n}`
  return content
})

const packageJson = computed(() => {
  let dependencies = {}

  selectedLibraries.value.forEach((lib) => {
    const packageName = libraries.value[lib].package

    // Skip inside-customs
    if (packageName !== 'inside-customs') {
      dependencies[packageName] = `git+ssh://git@bitbucket.org/maecia/${packageName}.git#develop`
    }
  })

  // Convert the dependencies object to JSON format
  let content = `"dependencies": {\n`
  for (let dep in dependencies) {
    content += `    "@maecia/${dep}": "${dependencies[dep]}",\n`
  }

  content += `    "inside-collection": "git+ssh://git@bitbucket.org/maecia/inside-collection.git#develop"\n`
  content += `  }`
  return content
})

const router = computed(() => {
  let content = `export default[\n`

  selectedLibraries.value.forEach((lib) => {
    const currentLib = libraries.value[lib]

    if (currentLib.routes) {
      currentLib.routes.forEach((route) => {
        content += `  {\n    name: '${route}',\n    meta: { disabled: false }\n  },\n`
      })
    }
  })

  content += `  {\n    name: 'CatalogPage',\n    meta: { disabled: false }\n  },\n`
  content += `  {\n    name: 'SimplePage',\n    meta: { disabled: false }\n  },\n`
  content += `]`

  return content
})

// Fetch libraries data from JSON file on mount
const loadLibraries = async () => {
  try {
    const response = await fetch('../public/librairies.json')
    libraries.value = await response.json()
  } catch (error) {
    console.error('Erreur lors du chargement des librairies', error)
  }
}

// Copy to clipboard function
const copyToClipboard = (text) => {
  navigator.clipboard.writeText(text)
}

const toggleSelectAll = () => {
  if (isSelectAll.value) {
    selectedLibraries.value = Object.keys(libraries.value)
  } else {
    selectedLibraries.value = []
  }
}

onMounted(() => {
  loadLibraries()
})
</script>

<style scoped>
pre {
  background: #f8f8f8;
  padding: 10px;
  border: 1px solid #ccc;
}
</style>
