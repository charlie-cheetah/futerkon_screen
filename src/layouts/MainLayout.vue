<template>
  <q-layout view="lHh Lpr lFf">
    <q-header elevated>
      <q-toolbar class="bg-blue-grey-10">
        <img src="./../../public/icons/apple-touch-icon.png" height="48">
        <q-toolbar-title class="text-h4">
          Agenda
        </q-toolbar-title>

        <div class="text-h4">
          <span class="text-h6 q-mr-lg">{{ date }}</span>
          <span class="text-h4">{{ time }}</span>
        </div>
      </q-toolbar>
    </q-header>

    <q-page-container>
      <router-view />
    </q-page-container>
  </q-layout>
</template>

<script>
import { defineComponent, ref } from 'vue'

export default defineComponent({
  name: 'MainLayout',
  setup() {
    const now = new Date()
    const options = { weekday: 'long', month: 'long', day: 'numeric' }
    const date = ref(now.toLocaleDateString('pl-PL', options))
    const time = ref(now.toLocaleTimeString('pl-PL'))
    setInterval(()=>{
      const now = new Date()
      date.value = now.toLocaleDateString('pl-PL', options)
      time.value = now.toLocaleTimeString('pl-PL')
    }, 1000)
    return {
      date,
      time
    }
  }
})
</script>
