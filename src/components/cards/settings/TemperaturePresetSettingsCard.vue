<template>
  <collapsable-card
    :title="$t('TemperaturePresetSettingsCard.Temperature Presets')"
    cardKey="TemperaturePresetSettings"
    icon="$fire"
    subTitle="$t('TemperaturePresetSettingsCard.Apply predefined temperatures on the dashboard')"
  >
    <v-card-text>

      <div v-for="(preset, i) in presets" :key="i">
        <v-layout align-center >
          <div class="grey--text focus--text mr-4">{{ preset.name }}</div>
          <v-chip
            label
            class="mx-1 my-2"
            v-for="(value, k) in preset.values"
            :key="k"
            :disabled="!value.active">
            {{ k }}: {{ value.value }}<small>°C</small>
          </v-chip>
          <div class="ml-auto">
            <v-btn
              @click="openEditDialog(preset, i)"
              icon
              color="primary">
              <v-icon small>$pencil</v-icon>
            </v-btn>
            <v-btn
              @click="remove(i)"
              icon
              color="error">
              <v-icon small>$delete</v-icon>
            </v-btn>
          </div>
        </v-layout>
        <v-divider></v-divider>
      </div>
      <v-btn
        @click="openAddDialog"
        class="mt-2"
        color="secondary">
        <v-icon small left>$plus</v-icon>
        {{$t('TemperaturePresetSettingsCard.Add preset')}}
      </v-btn>

      <v-dialog
        v-model="dialog.active"
        max-width="250px"
      >
        <v-form
          ref="tempPresetsform"
          @submit.prevent="save(preset)"
          v-model="dialog.valid"
        >
          <v-card color="secondary darken-1">
            <v-card-title>
              <span class="headline">{{ $t('TemperaturePresetSettingsCard.'+ (dialog.index >= 0) ? 'Edit preset' : 'Add preset') }}</span>
            </v-card-title>
            <v-card-text>
              <v-text-field
                :label="$t('TemperaturePresetSettingsCard.Preset Name')"
                v-model="preset.name"
                :height="36"
                :rules="[rules.required]"
                hide-details="auto"
                class="mb-2"
                filled dense>
              </v-text-field>
              <template v-if="dialog.active">
                <v-text-field
                  v-for="item in heaters" :key="item.name"
                  v-model="preset.values[item.name].value"
                  :label="item.name"
                  :rules="[rules.numRequired, rules.numMin]"
                  :append-outer-icon="preset.values[item.name].active ? '$checkboxMarked' : '$checkboxBlank'"
                  @click:append-outer="preset.values[item.name].active = !preset.values[item.name].active"
                  hide-details="auto"
                  type="number"
                  suffix="°C"
                  class="mb-2"
                  filled dense>
                </v-text-field>
                <v-text-field
                  v-for="item in fans" :key="item.name"
                  v-model="preset.values[item.name].value"
                  :label="item.name"
                  :rules="[rules.numRequired, rules.numMin]"
                  :append-outer-icon="preset.values[item.name].active ? '$checkboxMarked' : '$checkboxBlank'"
                  @click:append-outer="preset.values[item.name].active = !preset.values[item.name].active"
                  hide-details="auto"
                  type="number"
                  suffix="°C"
                  class="mb-2"
                  filled dense>
                </v-text-field>
              </template>
            </v-card-text>
            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn color="warning" text @click="dialog.active = false" type="button">{{$t('TemperaturePresetSettingsCard.Cancel')}}</v-btn>
              <v-btn color="primary" type="submit">{{ $t('TemperaturePresetSettingsCard.'+ (dialog.index >= 0) ? 'Save' : 'Add') }}</v-btn>
            </v-card-actions>
          </v-card>
        </v-form>
      </v-dialog>
    </v-card-text>
  </collapsable-card>
</template>

<script lang="ts">
import { Component, Mixins } from 'vue-property-decorator'
import { cloneDeep } from 'lodash-es'
import UtilsMixin from '@/mixins/utils'
import { TemperaturePreset } from '@/store/config/types'
import { Fan, Heater } from '@/store/socket/types'
import { VForm } from '@/types/vuetify'

@Component({})
export default class TemperaturePresetSettingsCard extends Mixins(UtilsMixin) {
  get heaters (): Heater[] {
    return this.$store.getters['socket/getHeaters']
  }

  get fans (): Fan[] {
    return this.$store.getters['socket/getOutputs'](['temperature_fan'])
  }

  get presets () {
    return this.$store.getters['config/getTempPresets']
  }

  get form (): VForm {
    return this.$refs.tempPresetsform as VForm
  }

  dialog = {
    index: -1, // add or edit
    active: false,
    valid: false
  }

  preset: TemperaturePreset = {
    name: '',
    values: {}
  }

  rules = {
    required: (v: string) => !!v || 'Required',
    numRequired: (v: number | string) => v !== '' || 'Required',
    numMin: (v: number) => v >= 0 || 'Min 0'
  }

  openEditDialog (preset: TemperaturePreset, index: number) {
    this.dialog = {
      index,
      active: true,
      valid: false
    }
    this.preset = cloneDeep(preset)
  }

  openAddDialog () {
    this.preset = {
      name: '',
      values: {}
    }
    for (const item of this.heaters) {
      this.preset.values[item.name] = { value: 0, type: 'heater', active: true }
    }
    for (const item of this.fans) {
      this.preset.values[item.name] = { value: 0, type: 'fan', active: true }
    }
    this.dialog = {
      index: -1,
      active: true,
      valid: false
    }
  }

  save (preset: TemperaturePreset) {
    const valid = this.form.validate()
    if (valid) {
      this.$store.dispatch('config/updatePreset', { preset, index: this.dialog.index })
      this.dialog.active = false
    }
  }

  remove (index: number) {
    this.$store.dispatch('config/removePreset', index)
    this.dialog.active = false
  }
}
</script>

<style lang="scss" scoped>
  .presets-table table > tbody > tr > td {
    padding-top: 8px;
    padding-bottom: 8px;
    vertical-align: top;
  }
</style>
