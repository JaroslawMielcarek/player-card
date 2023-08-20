<template>
  <div class="card-list">
    <PlayerCard class='card' v-for="player in players" :player="player"
      :active="options.modal.componentKey == player.id.toString()" :key="player.id" @selectedPlayer="selectedPlayer" />
  </div>
  <fieldset id="options">
    <legend>Options</legend>
    <label>Gender:
      <select v-model="options.gender">
        <option value="female">Female</option>
        <option value="male">Male</option>
        <option value="mix">Mix</option>
      </select>
    </label>
    <label>Theme:
      <select v-model="options.theme">
        <option value="light">light</option>
        <option value="dark">dark</option>
      </select>
    </label>
  </fieldset>
  <UniversalModal :modalActive="modalVisible" @close="closeModal">
    <component v-if="options.modal.componentKey != ''" :is="options.modal.component" />
  </UniversalModal>
</template>

<script setup lang="ts">
import { computed, ref, provide } from 'vue'
import type { Component } from 'vue'
import { Iplayer, IvoleyStats } from './components/interfaces';
import PlayerCard from './components/PlayerCard.vue';
import UniversalModal from './components/UniversalModal.vue'

const options = ref({
  theme: 'light',
  gender: 'mix',
  modal: {
    show: false,
    component: {},
    componentKey: ''
  }
})
const theme = computed(() => options.value.theme)
provide('theme', theme)
const modalVisible = computed(() => options.value.modal.show)
const closeModal = () => options.value.modal = { show: false, component: {}, componentKey: '' }
const rand = (range: number): number => Math.floor(Math.random() * range)
const players = computed<Iplayer[]>(() => [{
  id: 1,
  name: 'Valter',
  position: 'Central',
  gender: randomiseGender(),
  handPreference: 'Right'
},
{
  id: 2,
  name: 'Manolo',
  position: 'Receptor',
  gender: randomiseGender(),
  height: 181,
  number: 17,
  seasonStats: generatePlayerStats()
},
{
  id: 3,
  name: 'Omar',
  position: 'Setter',
  gender: randomiseGender(),
  height: 166,
  handPreference: 'Left'
},
{
  id: 4,
  name: 'Tapa',
  position: 'Oposite',
  gender: randomiseGender(),
  height: 189,
  seasonStats: generatePlayerStats(),
  handPreference: 'Right'
}])

function randomiseGender() {
  const GENDER = ['female', 'male']
  return (options.value.gender !== 'mix') ? options.value.gender : GENDER[rand(GENDER.length)]
}
function generatePlayerStats(): IvoleyStats {
  const ABILITIES = ['attack', 'recieve', 'set', 'block', 'serve']
  return ABILITIES.reduce((accum, curr) => {
    let good = rand(10)
    let bad = rand(10)
    const p = { good, bad, total: good + bad }

    good += rand(10)
    bad += rand(10)
    const c = { good: good, bad: bad, total: good + bad }

    accum[curr as keyof IvoleyStats] = { current: c, prev: p }
    return accum
  }, {} as IvoleyStats)
}
function selectedPlayer(comp: Component, compKey: string) {
  options.value.modal = { show: true, component: comp, componentKey: compKey }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  margin-top: 60px;
}

#options {
  display: flex;
  flex-direction: column;
  width: fit-content;
  margin-top: 20px;
}

.card-list {
  display: flex;
}

:root {
  --text-accent: #bb60e6;
}

:root .theme-light {
  --card-bg-color: #fafafa;
  --position-bg-color: #ffffff;
  --position-bg-color-rgb: 210, 210, 210;
  --text-color: #000000;
}

:root .theme-dark {
  --card-bg-color: #494949;
  --position-bg-color: #353535;
  --position-bg-color-rgb: 40, 40, 40;
  --text-color: #e9e9e9;
}
</style>
