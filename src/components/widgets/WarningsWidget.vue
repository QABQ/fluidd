<template>
  <v-alert text dense icon="$alert" type="warning">
    <div class="text-body-1 mb-2">{{ appName }} {{$t('WarningsWidget.warnings found')}}.</div>
    <ul class="mb-4" v-if="printerWarnings.length > 0">
      <li v-for="(warning, index) in printerWarnings" :key="index" v-html="warning.message"></li>
    </ul>

    <div v-if="moonrakerWarnings.length > 0">
      <div class="mb-2">{{$t('WarningsWidget.Moonraker has failed plugins, please check your logs, update your configuration and restart moonraker')}}.</div>
      <ul class="mb-4">
        <li v-for="(warning, index) in moonrakerWarnings" :key="index" v-html="warning"></li>
      </ul>
    </div>

    <div v-if="printerWarnings.length > 0">{{$t('WarningsWidget.Fluidd setup requirements can be found')}} <a target="_blank" :href="docsUrl">{{$t('WarningsWidget.here')}}.</a></div>
    <div v-if="moonrakerWarnings.length > 0">{{$t('WarningsWidget.Moonraker plugin configuration can be found')}} <a target="_blank" :href="moonrakerDocsUrl">{{$t('WarningsWidget.here')}}.</a></div>
  </v-alert>
</template>

<script lang="ts">
import { Component, Mixins } from 'vue-property-decorator'
import UtilsMixin from '@/mixins/utils'
import { Globals } from '@/globals'

@Component({
  components: {}
})
export default class KlippyCard extends Mixins(UtilsMixin) {
  get docsUrl () {
    return Globals.DOCS_REQUIRED_CONFIGURATION
  }

  get moonrakerDocsUrl () {
    return Globals.DOCS_MOONRAKER_PLUGINS
  }

  get appName () {
    return Globals.APP_NAME
  }

  get printerWarnings () {
    return this.$store.getters['socket/getPrinterWarnings']
  }

  get moonrakerWarnings () {
    return this.$store.getters['socket/getMoonrakerWarnings']
  }
}
</script>
