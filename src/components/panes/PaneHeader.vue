<template>
  <div class="pane-header toolbar" :class="{ '-loading': loading }">
    <div class="pane-header__loader"></div>
    <span class="ml4 mrauto" @dblclick="renamePane">{{ name }}</span>

    <slot />

    <button type="button" v-if="showSearch" @click="openSearch">
      <i class="icon-search"></i>
    </button>

    <dropdown
      v-if="showTimeframe"
      :options="timeframes"
      :selected="timeframe"
      placeholder="tf."
      @output="$store.commit(paneId + '/SET_TIMEFRAME', +$event)"
    >
      <template v-slot:selection="{ item }">
        <span>{{ item }}</span>
      </template>
    </dropdown>

    <button type="button" @click="openSettings">
      <i class="icon-cog"></i>
    </button>

    <dropdown :options="menu" class="-text-left" @open="highlightPane(true)" @close="highlightPane(false)">
      <template v-slot:option="{ value }">
        <div>
          <i class="-lower" :class="'icon-' + value.icon"></i>

          <span>{{ value.label }}</span>
        </div>
      </template>
      <template v-slot:selection>
        <i class="icon-more"></i>
      </template>
    </dropdown>
  </div>
</template>

<script lang="ts">
import Vue from 'vue'
import Component from 'vue-class-component'

import dialogService from '@/services/dialogService'

import CountersPaneDialog from '@/components/counters/CountersPaneDialog.vue'
import TradesPaneDialog from '../trades/TradesPaneDialog.vue'
import ChartPaneDialog from '../chart/ChartPaneDialog.vue'
import StatsPaneDialog from '../stats/StatsPaneDialog.vue'
import PricesPaneDialog from '../prices/PricesPaneDialog.vue'

@Component({
  name: 'PaneHeader',
  props: {
    paneId: {
      type: String
    },
    showSearch: {
      type: Boolean,
      default: true
    },
    showTimeframe: {
      type: Boolean,
      default: false
    },
    loading: {
      type: Boolean,
      default: false
    }
  }
})
export default class extends Vue {
  paneId: string
  timeframes = {
    1: '1s',
    3: '3s',
    5: '5s',
    10: '10s',
    30: '30s',
    60: '1m',
    [60 * 3]: '3m',
    [60 * 5]: '5m',
    [60 * 15]: '15m',
    [60 * 21]: '21m',
    [60 * 60]: '1h',
    [60 * 60 * 2]: '2h',
    [60 * 60 * 4]: '4h',
    [60 * 60 * 8]: '8h',
    [60 * 60 * 24]: '1d'
  }
  menu = [
    {
      icon: 'copy-paste',
      label: 'Duplicate pane',
      click: this.duplicatePane
    },
    {
      icon: 'plus',
      label: 'Zoom in',
      click: this.zoomIn
    },
    {
      icon: 'minus',
      label: 'Zoom out',
      click: this.zoomOut
    },
    {
      icon: 'trash',
      label: 'Remove pane',
      click: this.removePane
    }
  ]

  get name() {
    return this.$store.state.panes.panes[this.paneId].name || this.paneId
  }

  get zoom() {
    return this.$store.state.panes.panes[this.paneId].zoom || 1
  }

  get type() {
    return this.$store.state.panes.panes[this.paneId].type
  }

  get timeframe() {
    return this.$store.state[this.paneId].timeframe
  }

  openSettings() {
    switch (this.type) {
      case 'counters':
        dialogService.open(CountersPaneDialog, { paneId: this.paneId })
        break
      case 'stats':
        dialogService.open(StatsPaneDialog, { paneId: this.paneId })
        break
      case 'chart':
        dialogService.open(ChartPaneDialog, { paneId: this.paneId })
        break
      case 'trades':
        dialogService.open(TradesPaneDialog, { paneId: this.paneId })
        break
      case 'prices':
        dialogService.open(PricesPaneDialog, { paneId: this.paneId })
        break
    }
  }

  openSearch() {
    this.$store.dispatch('app/showSearch', { paneId: this.paneId })
  }

  zoomIn(event: Event) {
    this.$store.dispatch('panes/setZoom', { id: this.paneId, zoom: 0.1 })

    event.preventDefault()
    event.stopPropagation()
  }

  zoomOut(event: Event) {
    this.$store.dispatch('panes/setZoom', { id: this.paneId, zoom: -0.1 })

    event.preventDefault()
    event.stopPropagation()
  }

  removePane() {
    this.$store.dispatch('panes/removePane', this.paneId)
  }

  duplicatePane() {
    this.$store.dispatch('panes/duplicatePane', this.paneId)
  }

  copySettings() {
    this.$store.dispatch('panes/copySettings', this.paneId)
  }

  highlightPane(value: boolean) {
    this.$el.parentElement.parentElement.classList[value ? 'add' : 'remove']('-highlight')
  }

  pasteSettings() {
    let settings: any

    try {
      settings = JSON.parse(this.$store.state.app.paneClipboard)
    } catch (error) {
      this.$store.dispatch('app/showNotice', {
        title: 'No pane settings to paste',
        type: 'error'
      })
      return
    }

    this.$store.dispatch('panes/applySettings', {
      id: this.paneId,
      settings: settings
    })
  }

  async renamePane() {
    const name = await dialogService.prompt({
      action: 'Rename',
      input: this.name
    })

    if (name && name !== this.name) {
      this.$store.commit('panes/SET_PANE_NAME', { id: this.paneId, name: name })
    }
  }
}
</script>
