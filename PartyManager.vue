<template>
  <div id="wrap">
    <div id="tab-container" class="flex-row">
      <ul class="flex-row">
        <li
          class="flex-row"
          v-for="(item, index) in 12"
          :key="index"
          :style="{
            flexGrow: isSelected(item) ? '2' : '1',
          }"
        >
          <input
            type="radio"
            name="tab"
            :id="`tab-set-${item}`"
            v-model="selectedTab"
            :value="item"
          />
          <label
            class="flex-row"
            :for="`tab-set-${item}`"
            :style="{
              color: isSelected(item)
                ? [2].includes(Math.floor(index / 2))
                  ? '#444444'
                  : 'white'
                : elementMap[String(Math.floor(index / 2))].hex.replace('#f5ffbb', '#B9C19D'),
              backgroundColor: isSelected(item)
                ? elementMap[String(Math.floor(index / 2))].hex.replace('#f5ffbb', '#B9C19D')
                : 'transparent',
              fontSize: isSelected(item) ? '24px' : '24px',
            }"
          >
            {{ isSelected(item) ? `SET${item}` : '●' }}
          </label>
        </li>
      </ul>
    </div>
    <div id="lists-container" class="flex-row">
      <div>
        <button class="list-button" @click="exportData">save</button>
        <VueDraggable
          class="list-area flex-col"
          v-model="partyLists[selectedTab]"
          animation="150"
          ghostClass="ghost"
          group="party"
        >
          <div v-for="(item, index) in partyLists[selectedTab]" :key="item.listIndex">
            <PartyBoard
              :color="`${elementMap[String(Math.floor((selectedTab - 1) / 2))].hex.replace('#f5ffbb', '#B9C19D')}`"
              :index="index + 1"
              v-model:partyName="item.text"
              :partyData="item.partyData"
              @update:partyName="updatePartyName(index, $event)"
            />
          </div>
        </VueDraggable>
      </div>
      <div>
        <button class="list-button">add</button>
        <VueDraggable
          class="list-area flex-col"
          v-model="partyLists[0]"
          animation="150"
          ghostClass="ghost"
          group="party"
        >
          <div v-for="(item, index) in partyLists[0]" :key="item.listIndex">
            <PartyBoard
              :color="'gray'"
              :index="index + 1"
              v-model:partyName="item.text"
              :partyData="item.partyData"
              @update:partyName="updatePartyName(index, $event)"
            />
          </div>
        </VueDraggable>
      </div>
    </div>
  </div>
  <p>{{ selectedTab }}</p>
</template>

<script setup>
import { computed, onMounted, reactive, ref, watch } from 'vue';
import { useRoute, useRouter } from 'vue-router';
import { VueDraggable } from 'vue-draggable-plus';
// import { axios } from 'axios';
import PartyBoard from '@components/PartyBoard.vue';

import characterJson from '@assets/data/jp/character.json';
import elementMap from '@assets/data/map/elementMap.json';
import rawPartyLists from '@assets/data/party_lists.json';
const partyLists = ref(rawPartyLists);
const selectedTab = ref(1);
const isSelected = computed(() => (item) => {
  return selectedTab.value === item;
});

// 用于创建空白列表, 别删
const blankPartyData = {
  party: {
    union1: [0, 0, 0, 0],
    union2: [0, 0, 0, 0],
    union3: [0, 0, 0, 0],
  },
  input: {
    union1: ['', '', '', ''],
    union2: ['', '', '', ''],
    union3: ['', '', '', ''],
  },
  replacements: {
    union1: [[], [], [], []],
    union2: [[], [], [], []],
    union3: [[], [], [], []],
  },
  skillGauge: { union1: [0, 100], union2: [0, 100], union3: [0, 100] },
  mb2: {
    union1: [
      [-1, -1, -1],
      [-1, -1, -1],
    ],
    union2: [
      [-1, -1, -1],
      [-1, -1, -1],
    ],
    union3: [
      [-1, -1, -1],
      [-1, -1, -1],
    ],
  },
  ex: { union1: [0, 0], union2: [0, 0], union3: [0, 0] },
  awakeEnhanced: {
    union1: [false, false, false],
    union2: [false, false, false],
    union3: [false, false, false],
  },
  params: {
    wfid: '',
    mhid: '',
    abid: '',
    autoState: '',
    questCode: '',
    bindingParty: [],
    author: '',
    videoLink: '',
    articleLink: '',
    note: '',
  },
};
function createPartyList() {
  return reactive(
    Array(10)
      .fill()
      .map((_, index) => ({
        listIndex: index + 1,
        text: '',
        partyData: blankPartyData,
      })),
  );
}
const newPartyLists = reactive(Array.from({ length: 13 }, () => createPartyList()));

// 导出数据, 需要导出空白数据的时候把partyLists.value改成newPartyLists
function exportData() {
  const dataToExport = partyLists.value.map((bigList, bigIndex) =>
    bigList.map((item, smallIndex) => ({
      listIndex: `${bigIndex}-${smallIndex + 1}`, // bigIndex从0开始，smallIndex+1从1开始
      text: item.text,
      partyData: item.partyData,
    })),
  );

  const jsonData = JSON.stringify(dataToExport, null, 2);

  const blob = new Blob([jsonData], { type: 'application/json' });

  const url = URL.createObjectURL(blob);

  const link = document.createElement('a');
  link.href = url;
  link.setAttribute('download', 'party_lists.json');
  document.body.appendChild(link);
  link.click();

  document.body.removeChild(link);
  URL.revokeObjectURL(url);
}

// 编辑队伍名称
function updatePartyName(index, newName) {
  partyLists.value[selectedTab][index].text = newName;
}
</script>

<style>
@font-face {
  font-family: 'SY';
  font-weight: normal;
  font-style: normal;

  font-display: swap;
  src: url('@assets/fonts/SY-Simple.ttf') format('truetype');
}

:root {
  --tab-checked-width: 176px;
  --tab-gap: 1px;
  --tab-height: 72px;
  --tab-width: 80px;
}

.flex-row {
  display: flex;
  flex-direction: row;
}

.flex-col {
  display: flex;
  flex-direction: column;
}
</style>

<style scoped>
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

*::selection {
  background-color: hsla(200, 100%, 25%, 0.6);
  color: white;
  paint-order: stroke fill;
  -webkit-text-stroke: 0px transparent;
}

#wrap {
  width: 1280px;
  height: 100%;
  margin: 0 auto;
  padding: 32px 0;

  background-color: #eeeeee;
}

#tab-container {
  align-items: center;
  justify-content: center;

  width: 100%;
  overflow: hidden;

  font-family: 'SY';

  filter: drop-shadow(0 4px 4px rgba(0, 0, 0, 0.2));
  transition: all 0.8s ease-in-out;
}

#tab-container input {
  display: none;
}

#tab-container ul {
  position: relative;

  overflow: hidden;
  justify-content: center;

  width: 100%;
  height: var(--tab-height);

  background-color: #cccccc;
  border-radius: 16px 16px 0 0;
  gap: 2px;
}

#tab-container ul::after {
  position: absolute;
  z-index: 2;

  display: inline-block;

  box-sizing: border-box;
  width: 100%;
  height: 100%;

  border: 16px solid #fafafa;
  content: '';
}

#tab-container li {
  z-index: 10;

  align-items: center;
  justify-content: center;

  height: var(--tab-height);
  padding: 8px;

  background-color: #fafafa;
}

#tab-container label {
  display: inline-block;
  align-items: center;
  justify-content: center;

  width: 100%;
  height: 100%;

  line-height: 56px;

  border-radius: 8px;
  cursor: pointer;
  text-align: center;
}

#tab-container label:hover {
  box-shadow: 0 0 4px rgba(0, 0, 0, 0.4);
  outline: 1px solid white;
}

#lists-container {
  justify-content: space-evenly;

  width: 100%;
  padding: 16px;

  gap: 16px;
}

#lists-container > div {
  flex-grow: 1;
}

.list-button {
  width: 48px;
  height: 48px;

  border: none;
  border-radius: 25%;
  cursor: pointer;
}

.list-button:hover {
  box-shadow: 0 0 4px rgba(0, 0, 0, 0.4);
  outline: 1px solid white;
}

.list-area {
  align-items: center;
  justify-content: center;

  width: 100%;
  min-height: 200px;

  gap: 8px;
}
</style>
