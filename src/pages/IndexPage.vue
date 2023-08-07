<template>
  <q-page class="flex bg-blue-grey-14" :style-fn="styleTweak">
    <div class="row max-inherit full-width">
      <div class="col-8 max-inherit column">
        <q-btn v-if="agenda?.current?.length" text-color="black"  color="indigo-2" size="lg" class="col-auto q-ma-xs">Aktualnie trwa:</q-btn>
        <div class="overflow-auto col" ref="scrollable">
          <div class="q-gutter-md q-pa-md">
            <q-card v-for="a in agenda.current" :key="a.agenda_id" class="current-card" ref="currentCard">
              <q-card-section class="bg-indigo-2 q-py-sm">
                <div class="text-h3 q-mt-sm q-mb-xs font-weigh-bold">{{ a.name }}
                  <q-chip dark size="lg" v-if="a.settings.canceled && a.settings.canceled === 'on'" color="negative" class="q-ml-sm">odwołany!</q-chip>
                  <q-chip dark size="lg" v-if="a.settings.changed && a.settings.changed === 'on'" color="orange-10" class="q-ml-sm">zmiana!</q-chip>
                  <q-chip dark size="lg" v-if="a.settings.extra_points && a.settings.extra_points === 'on'" color="blue-10" class="q-ml-sm">punktowany</q-chip>
                </div>
              </q-card-section>
              <q-card-section horizontal>
                <q-card-section class="preview q-pt-sm q-pb-none" :class="a.image.full?'col-10':'col-12'">
                  <div class="row">
                    <div class="col-6 self-center flex items-center"><span class="text-weight-bold text-h4">{{ formatTime(a) }}</span><q-icon size="sm" name="schedule" class="q-ml-md" /><span class="text-h5 q-ml-xs">{{ formatDuration(a) }}</span></div>
                    <div class="col-6  text-weight-bold text-right">
                      <q-btn color="primary" size="lg" icon="location_on" :label="a.location.name" />
                    </div>
                  </div>
                  <div class="q-mt-md truncate-overflow text-h6 text-weight-regular" v-html="a.post_content" />
                </q-card-section>
                <div v-if="a.image.full" class="col-2 items-end">
                  <q-img
                    class="preview"
                    :src="a.image.full"
                    fit="contain"
                  />
                </div>
              </q-card-section>
              <q-linear-progress stripe rounded size="20px" :value="a.progress/100" color="indigo-4" />
            </q-card>
          </div>
        </div>
      </div>
      <div class="col-4 max-inherit column">
        <q-btn v-if="agenda?.upcomming?.length" text-color="black" color="orange-2" size="lg" class="col-auto q-ma-xs">Wkrótce będzie:</q-btn>
        <div class="overflow-hidden col">
          <div class="q-gutter-md q-pa-md">
            <q-card v-for="a in agenda.upcomming" :key="a.agenda_id">
              <q-card-section class="bg-orange-2 q-py-sm">
                <div class="text-h3 q-mt-sm q-mb-xs font-weigh-bold">{{ a.name }}
                  <q-chip dark size="lg" v-if="a.settings.canceled && a.settings.canceled === 'on'" color="negative" class="q-ml-sm">odwołany!</q-chip>
                  <q-chip dark size="lg" v-if="a.settings.changed && a.settings.changed === 'on'" color="orange-10" class="q-ml-sm">zmiana!</q-chip>
                  <q-chip dark size="lg" v-if="a.settings.extra_points && a.settings.extra_points === 'on'" color="blue-10" class="q-ml-sm">punktowany</q-chip>
                </div>
              </q-card-section>

              <q-card-section class="preview q-pt-sm q-pb-sm">
                <div class="row">
                  <div class="col-6 self-center"><span class="text-weight-bold text-h4">{{ formatTime(a) }}</span></div>
                  <div class="col-6  text-weight-bold text-right">
                    <q-btn color="primary" size="lg" icon="location_on" :label="a.location.name" />
                  </div>
                </div>
              </q-card-section>
            </q-card>
          </div>
        </div>
      </div>
    </div>
  </q-page>
</template>

<script>
import { defineComponent, ref } from 'vue'
import { useQuasar } from 'quasar'

export default defineComponent({
  name: 'IndexPage',
  setup () {
    const agenda = ref({});
    const currentCard = ref(null)
    const dismiss = ref(null)
    const nowDisplaying = ref(0)
    const $q = useQuasar()

    const styleTweak = (offset) =>
    {
      return {maxHeight: offset ? `calc(100vh - ${offset}px)` : '100vh', height: offset ? `calc(100vh - ${offset}px)` : '100vh'}
    }
    const timer = function () {
      fetch('https://2023.futerkon.pl/wp-json/wpvue/agenda_screen' + window.location.search)
        .then(response => { console.log('tick!'); return response.json();})
        .then(data => agenda.value = data)
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
        dismiss.value()
      }
    );

    const scroll = function() {
      if (nowDisplaying.value === 0) {
        nowDisplaying.value = currentCard.value.length - 1
      } else {
        nowDisplaying.value = 0;
      }
      console.log(nowDisplaying.value);
      currentCard.value?.[nowDisplaying.value]?.$el.scrollIntoView({ behavior: "smooth", block: "center", inline: "nearest" })

    }

    setInterval(timer(),  10 * 1000) //co minute

    setInterval(scroll,   10 * 1000) //co 10 sek

    const formatTime = (a) => {
      const start = new Date(a.start_date * 1000)
      const end = new Date((Number(a.start_date) + (a.duration * 60))*1000)
      return start.toLocaleTimeString('pl-PL', { hour: "2-digit", minute: "2-digit" }) + ' - ' + end.toLocaleTimeString('pl-PL', { hour: "2-digit", minute: "2-digit" })
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
      styleTweak,
      formatTime,
      formatDuration,
      currentCard
    }
  }
})
</script>
<style lang="scss">
.max-inherit {
  max-height: inherit;
}
.current-card {
  .preview{
    max-height: 9rem;
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
