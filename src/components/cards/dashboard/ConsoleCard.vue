<template>
  <collapsable-card
    :title="$t('ConsoleCard.Console')"
    icon="$console"
    cardClasses="mb-2 mb-sm-4 d-flex flex-column"
    contentClasses="flex-grow-1 flow-shrink-0"
    menuBreakpoint="none"
    menuIcon="$cog"
    :height="450"
    :draggable="true"
    :inLayout="inLayout"
    :enabled="enabled"
    @enabled="$emit('enabled', $event)"
    @collapsed="handleCollapseChange">

    <template v-slot:menu>
      <v-checkbox
        v-model="hideTempWaits"
        color="primary"
        class="ma-2"
        hide-details
        :label="$t('ConsoleCard.Hide temp waits')"
      >
      </v-checkbox>
    </template>

    <console-widget
      ref="console"
      :items="items"
    ></console-widget>

  </collapsable-card>
</template>

<script lang="ts">
import { Component, Mixins, Prop, Watch } from 'vue-property-decorator'
import PrintStatusWidget from '@/components/widgets/PrintStatusWidget.vue'
import UtilsMixin from '@/mixins/utils'
import ConsoleWidget from '@/components/widgets/ConsoleWidget.vue'
import { ConsoleEntry } from '@/store/socket/types'

@Component({
  components: {
    PrintStatusWidget,
    ConsoleWidget
  }
})
export default class ConsoleCard extends Mixins(UtilsMixin) {
  @Prop({ type: Boolean, default: true })
  enabled!: boolean

  get items (): ConsoleEntry[] {
    return this.$store.getters['socket/getConsoleEntries']
  }

  get inLayout (): boolean {
    return (this.$store.state.config.layoutMode)
  }

  get consoleComponent (): ConsoleWidget {
    return this.$refs.console as ConsoleWidget
  }

  @Watch('inLayout')
  inLayoutChange (inLayout: boolean) {
    if (!inLayout) {
      this.consoleComponent.scrollToEnd()
    }
  }

  handleCollapseChange (collapsed: boolean) {
    if (!collapsed) {
      this.consoleComponent.scrollToEnd()
    }
  }

  get hideTempWaits (): boolean {
    return this.$store.state.config.fileConfig.general.hideTempWaits
  }

  set hideTempWaits (value: boolean) {
    this.$store.dispatch('config/saveGeneric', { key: 'fileConfig.general.hideTempWaits', value })
  }
}
</script>
