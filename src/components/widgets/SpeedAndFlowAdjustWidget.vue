<template>
  <v-row>
    <v-col cols="12" sm="6">
      <!-- Speed and Flow Adjust -->
      <input-slider
        :label="$t('SpeedAndFlowAdjustWidget.Speed')"
        value-suffix="%"
        input-xs
        v-model.number="speed"
        :disabled="!klippyConnected || hasWait(waits.onSetSpeed)"
        :min="1"
        :max="200"
        :rules="rules">
      </input-slider>
    </v-col>
    <v-col cols="12" sm="6">
      <input-slider
        :label="$t('SpeedAndFlowAdjustWidget.Flow')"
        value-suffix="%"
        input-xs
        v-model.number="flow"
        :disabled="!klippyConnected || hasWait(waits.onSetFlow)"
        :min="1"
        :max="200"
        :rules="rules">
      </input-slider>
    </v-col>
  </v-row>
</template>

<script lang="ts">
import { Component, Mixins } from 'vue-property-decorator'
import InputSlider from '@/components/inputs/InputSlider.vue'
import UtilsMixin from '@/mixins/utils'
import { Waits } from '@/globals'

@Component({
  components: {
    InputSlider
  }
})
export default class SpeedAndFlowAdjustWidget extends Mixins(UtilsMixin) {
  waits = Waits

  rules = [
    (v: number) => (v >= 1) || 'min 1',
    (v: number) => (v <= 200) || 'max 200'
  ]

  get flow () {
    return this.$store.state.socket.printer.gcode_move.extrude_factor * 100 || 100
  }

  set flow (val: number) {
    this.sendGcode('M221 S' + val, this.waits.onSetFlow)
  }

  get speed () {
    return this.$store.state.socket.printer.gcode_move.speed_factor * 100 || 100
  }

  set speed (val: number) {
    this.sendGcode('M220 S' + val, this.waits.onSetSpeed)
  }
}
</script>
