<template>
    <div class="row bg-blue-grey-14 max-height-calc full-width" :style="{maxHeight: `calc(100vh - ${overscan * 2}px - 50px)`, height: `calc(100vh - ${overscan * 2}px - 50px)`}">
      <div class="col-8 max-inherit column">
        <q-btn v-if="agenda?.current?.length" text-color="black" color="indigo-2" size="lg" class="col-auto q-ma-xs">Aktualnie trwa:</q-btn>
        <div class="overflow-auto col" ref="scrollable">
          <div class="q-gutter-md q-pa-md">
            <q-card v-for="a in agenda.current" :key="a.agenda_id" class="current-card" ref="currentCard">
              <q-card-section horizontal>
                <q-card-section class="q-pa-none full-width">
                  <q-card-section class="q-py-sm rounded-borders-tr" :class="screenLocation?.id === a.location.id ? 'bg-indigo-10' : 'bg-indigo-2'">
                    <div class="text-h3 q-mt-sm q-mb-xs font-weigh-bold" :class="{'text-white':screenLocation?.id === a.location.id}">{{ a.name }}
                      <q-chip dark size="lg" v-if="a.settings.canceled && a.settings.canceled === 'on'" color="negative" class="q-ml-sm">odwołany!</q-chip>
                      <q-chip dark size="lg" v-if="a.settings.changed && a.settings.changed === 'on'" color="orange-10" class="q-ml-sm">zmiana!</q-chip>
                      <q-chip dark size="lg" v-if="a.settings.extra_points && a.settings.extra_points === 'on'" color="blue-10" class="q-ml-sm">punktowany</q-chip>
                    </div>
                  </q-card-section>
                  <q-card-section class="preview q-py-sm" :class="[a.image.full?'col-10':'col-12',{'bg-indigo-7 text-white':screenLocation?.id === a.location.id}]" >
                    <div class="row">
                      <div class="col-6 self-center flex items-center"><span class="text-weight-bold text-h4">{{ formatTime(a) }}</span><q-icon size="sm" name="schedule" class="q-ml-md" /><span class="text-h5 q-ml-xs">{{ formatDuration(a) }}</span></div>
                      <div class="col-6  text-weight-bold text-right">
                        <q-btn color="primary" size="lg" :icon=" screenLocation?.id === a.location.id ? 'my_location' : 'location_on'" :label="screenLocation?.id === a.location.id ? 'tutaj' : a.location.name" @click="saveLocation(a.location)" />
                      </div>
                    </div>
                    <div class="q-mt-sm truncate-overflow text-h6 text-weight-regular" v-html="a.post_content" />
                  </q-card-section>
                  <q-linear-progress stripe class="rounded-borders-br" size="20px" :value="a.progress/100" :color="screenLocation?.id === a.location.id ? 'indigo-10' : 'indigo-4'" />
                </q-card-section>
                <div class="ima col-2">
                  <q-img
                    :src="a.image.full"
                    fit="cover"
                    class="full-height rounded-borders-l"
                  />
                </div>
              </q-card-section>
            </q-card>
          </div>
        </div>
      </div>
      <div class="col-4 max-inherit column">
        <q-btn v-if="agenda?.upcomming?.length" text-color="black" color="green-2" size="lg" class="col-auto q-ma-xs">Wkrótce będzie:</q-btn>
        <div class="overflow-hidden col">
          <div class="q-gutter-md q-pa-md">
            <q-card v-for="a in agenda.upcomming" :key="a.agenda_id">
              <q-card-section class="q-py-sm" :class="screenLocation?.id === a.location.id ? 'bg-green-10' : 'bg-green-2'">
                <div class="text-h3 q-mt-sm q-mb-xs font-weigh-bold" :class="{'text-white':screenLocation?.id === a.location.id}">{{ a.name }}
                  <q-chip dark size="lg" v-if="a.settings.canceled && a.settings.canceled === 'on'" color="negative" class="q-ml-sm">odwołany!</q-chip>
                  <q-chip dark size="lg" v-if="a.settings.changed && a.settings.changed === 'on'" color="orange-10" class="q-ml-sm">zmiana!</q-chip>
                  <q-chip dark size="lg" v-if="a.settings.extra_points && a.settings.extra_points === 'on'" color="blue-10" class="q-ml-sm">punktowany</q-chip>
                </div>
              </q-card-section>

              <q-card-section class="preview q-pt-sm q-pb-sm" :class="{'bg-green-7 text-white':screenLocation?.id === a.location.id}">
                <div class="row">
                  <div class="col-6 self-center"><span class="text-weight-bold text-h4">{{ formatTime(a) }}</span></div>
                  <div class="col-6  text-weight-bold text-right">
                    <q-btn color="primary" size="lg" :icon=" screenLocation?.id === a.location.id ? 'my_location' : 'location_on'" :label="screenLocation?.id === a.location.id ? 'tutaj' : a.location.name" @click="saveLocation(a.location)" />
                  </div>
                </div>
              </q-card-section>
            </q-card>
          </div>
        </div>
      </div>
    </div>
</template>

<script>
import { defineComponent, ref } from 'vue'
import { useQuasar } from 'quasar'

export default defineComponent({
  name: 'IndexPage',
  emits: ['location'],
  props: ['overscan'],
  setup (props, { emit }) {
    const agenda = ref({});
    const currentCard = ref(null)
    const dismiss = ref(null)
    const nowDisplaying = ref(0)
    const $q = useQuasar()
    const screenLocation = ref(null)

    try {
      screenLocation.value = JSON.parse(localStorage.getItem("location"))
      emit('location', screenLocation.value)
    } catch (e) {}

    const timer = function () {
      console.log('fetch');
      fetch('https://2023.futerkon.pl/wp-json/wpvue/agenda_screen' + window.location.search)
        .then(response => { console.log('tick!'); return response.json();})
        .then(data => agenda.value = {current: data.current.sort((a) => a.location.id === screenLocation.value?.id ? -1 : 0), upcomming:data.upcomming})
    }

    window.addEventListener("offline",
      ()=> {
        console.log('offline!')
        dismiss.value = $q.notify({
          spinner: true,
          type: "negative",
          message: "Brak połączenia z internetem, dane mogą być nieaktualne",
          timeout: 60 * 1000
        })
      }
    );

    window.addEventListener("online",
      ()=> {
      console.log('online')
        dismiss.value()
      }
    );

    const scroll = function() {
      if (nowDisplaying.value === 0) {
        nowDisplaying.value = currentCard?.value?.length - 1
      } else {
        nowDisplaying.value = 0;
      }
      currentCard.value?.[nowDisplaying.value]?.$el.scrollIntoView({ behavior: "smooth", block: "center", inline: "nearest" })

    }
    timer()
    setInterval(timer,   60 * 1000) //co minute

    setInterval(scroll,   10 * 1000) //co 10 sek

    const formatTime = (a) => {
      const start = new Date(a.start_date * 1000)
      const end = new Date((Number(a.start_date) + (a.duration * 60))*1000)
      return start.toLocaleTimeString('pl-PL', { hour: "2-digit", minute: "2-digit" }) + ' - ' + end.toLocaleTimeString('pl-PL', { hour: "2-digit", minute: "2-digit" })
    }

    const saveLocation = (location) => {
      if (screenLocation.value?.id === location.id) {
        screenLocation.value = null
      } else {
        screenLocation.value = location
      }
      emit('location', screenLocation.value)
      localStorage.setItem("location", JSON.stringify(screenLocation.value))
    }

    const formatDuration = (a) =>{
      switch (a.duration) {
        case '30': return '30 minut'
        case '60' : return '1 godzina'
        case '90' : return '1,5 godziny'
        case '120' : return '2 godziny'
        case '150' : return '2,5 godziny'
        case '180' : return '3 godziny'
        case '240' : return '4 godziny'
        case '300' : return '5 godzin'
        case '360' : return '6 godzin'
        case '420' : return '7 godzin'
        case '480' : return '8 godzin'
        case '840' : return 'cały dzień'
      }
    }
    return {
      agenda,
      formatTime,
      formatDuration,
      currentCard,
      saveLocation,
      screenLocation
    }
  }
})
</script>
<style lang="scss">
.max-inherit {
  max-height: inherit;
  height: inherit;
}

.current-card {
  .rounded-borders-l {
    border-radius: 0 4px 4px 0;
  }

  .rounded-borders-br {
    border-radius: 0 0 0 4px;
  }

  .rounded-borders-tr {
    border-radius: 4px 0 0 0;
  }

  .truncate-overflow {
    height: 4rem;
    overflow: hidden;
    display: -webkit-box;
    -webkit-line-clamp: 2;
    -webkit-box-orient: vertical;
  }
}
</style>
