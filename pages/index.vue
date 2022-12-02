<template>
  <v-container style="width: 500px;">
    <v-row no-gutters>
      <v-col>
        <h1>Veto de Mapas 4º Torneio Horus</h1>
      </v-col>
    </v-row>
    <v-row no-gutters>
      <v-col>
        <v-select
          v-model="format"
          :readonly="vetoInProgress"
          class="mt-3"
          outlined
          dense
          label="Formato"
          :items="formats"
        />
      </v-col>
    </v-row>
    <v-row no-gutters>
      <v-col>
        <v-text-field
          v-model="higher"
          :readonly="vetoInProgress"
          outlined
          dense
          label="Time 1 (Maior Tier)"
        />
      </v-col>
    </v-row>
    <v-row no-gutters>
      <v-col>
        <v-text-field
          v-model="lower"
          :readonly="vetoInProgress"
          outlined
          dense
          label="Time 2 (Menor Tier)"
        />
      </v-col>
    </v-row>
    <v-row no-gutters>
      <v-col>
        <v-btn
          :disabled="vetoInProgress"
          style="float: right;"
          color="primary"
          outlined
          @click="startVeto"
        >
          Iniciar
        </v-btn>
        <v-btn
          v-if="vetoInProgress"
          style="float: right; margin-right: 5px;"
          color="error"
          outlined
          @click="restartVeto"
        >
          Reiniciar
        </v-btn>
        <v-btn
          v-if="vetoInProgress && choosingSide"
          style="float: right; margin-right: 5px;"
          color="primary"
          outlined
          fab
          @click="chooseSide('CT')"
        >
          CT
        </v-btn>
        <v-btn
          v-if="vetoInProgress && choosingSide"
          style="float: right; margin-right: 5px;"
          color="error"
          outlined
          fab
          @click="chooseSide('TR')"
        >
          TR
        </v-btn>
      </v-col>
    </v-row>
    <v-divider style="margin-top: 10px;" />
    <h1>{{ message }}</h1>
    <v-divider style="margin-top: 10px;" />
    <v-row v-if="vetoInProgress" style="margin-top: 10px;">
      <v-container>
        <v-btn
          v-for="(mapa, i) in mapas"
          :key="i"
          class="button-click"
          :disabled="!mapa.available || waiting"
          outlined
          color="primary"
          style="margin: 10px"
          @click="clickMap(mapa, i)"
        >
          {{ mapa.name }}
        </v-btn>
      </v-container>
      <v-divider style="margin-top: 10px;" />
      <v-list>
        <v-list-item v-for="(log, i) in logs" :key="i" style="margin-top: 2px;">
          {{ log }}
        </v-list-item>
      </v-list>
    </v-row>
  </v-container>
</template>

<script>
export default {
  name: 'IndexPage',

  data () {
    return {
      choosingSide: false,
      lastMap: null,
      higher: null,
      lower: null,
      turn: null,
      step: -1,
      waiting: false,
      vetoInProgress: false,
      message: '',
      format: null,
      formats: [
        { text: 'MD3', value: 'md3' },
        { text: 'MD1', value: 'md1' }
      ],
      logs: [],
      mapas: [
        { name: 'de_dust2', available: true },
        { name: 'de_mirage', available: true },
        { name: 'de_inferno', available: true },
        { name: 'de_overpass', available: true },
        { name: 'de_vertigo', available: true },
        { name: 'de_nuke', available: true },
        { name: 'de_ancient', available: true }
      ],
      flows: {
        md1: [
          { action: 'mapBan', team: 'lower' },
          { action: 'mapBan', team: 'lower' },
          { action: 'mapBan', team: 'higher' },
          { action: 'mapBan', team: 'higher' },
          { action: 'mapBan', team: 'higher' },
          { action: 'mapBan', team: 'lower' }
        ],
        md3: [
          { action: 'mapBan', team: 'lower' },
          { action: 'mapBan', team: 'higher' },
          { action: 'mapPick', team: 'lower' },
          { action: 'chooseSide', team: 'higher' },
          { action: 'mapPick', team: 'higher' },
          { action: 'chooseSide', team: 'lower' },
          { action: 'mapBan', team: 'higher' },
          { action: 'mapBan', team: 'lower' }
        ]
      }
    }
  },

  methods: {
    startVeto () {
      if (!this.higher || !this.lower || !this.format) {
        return alert('Informe nome do time 1, time 2 e formato.')
      }
      this.vetoInProgress = true
      this.runStep()
    },
    runStep () {
      if (this.step >= this.flows[this.format].length + 1) {
        return
      }
      this.step++
      this.waiting = true
      const steps = this.flows[this.format]
      if (!steps[this.step]) {
        if (this.format === 'md3') {
          const last = this.mapas.filter(m => m.available)
          this.logs.push(`${last[0].name} was left over`)
          this.choosingSide = false
          this.step++
          return
        } else {
          this.choosingSide = true
          const last = this.mapas.filter(m => m.available)
          this.logs.push(`${last[0].name} was left over`)
          const message = `${this.higher} chooses the start side`
          this.message = message
          this.step++
          return
        }
      }
      this.turn = steps[this.step].team
      const team = steps[this.step].team === 'lower' ? this.lower : this.higher
      const acao = steps[this.step].action === 'mapBan' ? 'bans one map' : steps[this.step].action === 'mapPick' ? 'picks one map' : 'chooses the start side'
      const message = `${team} ${acao}`
      this.message = message
      if (steps[this.step].action === 'chooseSide') {
        this.choosingSide = true
        this.waiting = true
        return
      }
      this.waiting = false
    },
    clickMap (mapa, i) {
      this.mapas[i].available = false
      const steps = this.flows[this.format]
      const team = steps[this.step].team === 'lower' ? this.lower : this.higher
      const acao = steps[this.step].action === 'mapBan' ? `banned ${mapa.name}` : steps[this.step].action === 'mapPick' ? `picked ${mapa.name}` : null
      const message = acao ? `${team} ${acao}` : null
      if (message) {
        this.logs.push(message)
      }
      this.lastMap = mapa.name
      this.runStep()
    },
    chooseSide (side) {
      const steps = this.flows[this.format]
      const team = steps[this.step] ? (steps[this.step].team === 'lower' ? this.lower : this.higher) : this.higher
      const lastMap = steps[this.step] ? this.lastMap : this.mapas.filter(m => m.available)[0].name
      const message = `${team} choosed ${side} side to start in ${lastMap}`
      this.logs.push(message)
      this.waiting = false
      this.choosingSide = false
      this.runStep()
    },
    restartVeto () {
      window.location.reload()
    }
  }
}
</script>
