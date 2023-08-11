<template>
  <div class="bg-blue-grey-14 overscan full-height">
    <q-toolbar class="bg-blue-grey-10 text-white">
      <img src="./../../public/icons/apple-touch-icon.png" height="48">
      <q-toolbar-title class="text-h4">
        Agenda {{ screenLocation ? ' - ' + screenLocation.name : '' }}
      </q-toolbar-title>
      <q-btn round flat @click="setOverscan(true)">+</q-btn>
      <q-btn round flat class="q-mr-md" @click="setOverscan(false)">-</q-btn>
      <div class="text-h4">
        <span class="text-h6 q-mr-lg">{{ date }}</span>
        <span class="text-h4">{{ time }}</span>
      </div>
    </q-toolbar>
    <router-view :overscan="overscan" @location="saveLocation"/>
  </div>
</template>

<script>
import { defineComponent, ref, computed } from 'vue'

export default defineComponent({
  name: 'MainLayout',
  setup() {
    const now = new Date()
    const options = { weekday: 'long', month: 'long', day: 'numeric' }
    const date = ref(now.toLocaleDateString('pl-PL', options))
    const time = ref(now.toLocaleTimeString('pl-PL'))
    const overscan = ref(Number(localStorage.getItem("overscan")))

    const setOverscan = (more) => {
      if (more) {
        overscan.value = overscan.value + 5
      } else {
        overscan.value = overscan.value - 5
      }
      localStorage.setItem("overscan", overscan.value)
    }

    const overscanInPx = computed(() => overscan.value + 'px')

    const screenLocation = ref(null)

    const saveLocation = (loc) => {
      screenLocation.value = loc
    }

    setInterval(()=>{
      const now = new Date()
      date.value = now.toLocaleDateString('pl-PL', options)
      time.value = now.toLocaleTimeString('pl-PL')
    }, 1000)
    return {
      date,
      time,
      saveLocation,
      setOverscan,
      screenLocation,
      overscan,
      overscanInPx
    }
  }
})
</script>
<style scoped>
.overscan {
  padding: v-bind('overscanInPx');
}
</style>
