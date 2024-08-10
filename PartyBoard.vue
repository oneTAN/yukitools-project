<template>
  <div>
    <div class="party-board-container">
      <div class="title-area">
        <div>
          <div
            :style="{
              background: `linear-gradient(240deg, transparent 16px, ${color} 16px)`,
            }"
          >
            {{ index }}
          </div>
          <input
            v-model="localPartyName"
            ref="inputRef"
            :disabled="!isEditing"
            @blur="handleBlur"
          />
        </div>
        <div>
          <div>fgr5sf5g</div>
          <div></div>
        </div>
      </div>
      <div class="content-area flex-row">
        <div class="party-area">
          <div>
            <div v-for="(item, index) in 6" class="img-container">
              <img :src="characterRarityArray(index)" />
              <img :src="characterSrcArray(index)" />
            </div>
          </div>
          <div>
            <div v-for="(item, index) in 6" class="img-container">
              <img :src="equipmentRarityArray(index)" />
              <img class="image-rendering-pixelated" :src="equipmentSrcArray(index)" />
              />
            </div>
          </div>
        </div>
        <div @click="toggleEdit" class="edit-button-area">
          <button>展开</button>
          <button class="edit-button"><i class="fas fa-pencil-alt"></i></button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, watch, nextTick, defineProps, defineEmits } from 'vue';

import characterJson from '@assets/data/jp/character.json';
import equipmentJson from '@assets/data/cn/equipment.json';
import itemJson from '@assets/data/cn/item.json';
import rarityMap from '@assets/data/map/rarityMap.json';

// 接收从父组件传入的 `props`
const props = defineProps({
  color: {
    type: String,
    required: true,
  },
  index: {
    type: Number,
    required: true,
  },
  partyName: {
    type: [String, Number],
    default: '队伍A',
    required: false,
  },
  partyData: {
    type: Object,
    required: true,
  },
});

const emit = defineEmits(['update:partyName']);
const isEditing = ref(false);
const inputRef = ref(null);
const localPartyName = ref(props.partyName);

function toggleEdit() {
  // 切换编辑模式, 进入编辑模式时自动聚焦input
  isEditing.value = !isEditing.value;
  nextTick(() => {
    if (isEditing.value && inputRef.value) {
      inputRef.value.focus();
    }
  });
};

function handleBlur() {
  // 失焦时退出编辑模式, 并更新partyName
  isEditing.value = false;
  emit('update:partyName', localPartyName.value);
};


// 提取图片的一些有的没的
const characterIdArray = computed(() => [
  props.partyData.party.union1[0] || 0,
  props.partyData.party.union2[0] || 0,
  props.partyData.party.union3[0] || 0,
  props.partyData.party.union1[1] || 0,
  props.partyData.party.union2[1] || 0,
  props.partyData.party.union3[1] || 0,
]);
const characterSrcArray = (index) => {
  const characterId = characterIdArray.value[index];
  return `/src/assets/image/character/square_0/${characterJson[characterId]?.[0] || 'blank'}.png`;
};
const characterRarityArray = (index) => {
  const characterId = characterIdArray.value[index];
  return `/src/assets/image/scene/general/rarity/item_${rarityMap[characterJson[characterId]?.[2]]?.label || 'empty'}.png`;
};
const equipmentIdArray = computed(() => [
  props.partyData.party.union1[2] || 0,
  props.partyData.party.union2[2] || 0,
  props.partyData.party.union3[2] || 0,
  props.partyData.party.union1[3] || 0,
  props.partyData.party.union2[3] || 0,
  props.partyData.party.union3[3] || 0,
]);
const equipmentSrcArray = (index) => {
  const equipmentId = equipmentIdArray.value[index] || 0; // Use 0 as a fallback for null/undefined
  const srcArray = [];

  if (index < 3) {
    srcArray.push(`/src/assets/image/${equipmentJson[equipmentId]?.[6] || 'item/blank'}.png`);
  } else {
    srcArray.push(`/src/assets/image/${itemJson[equipmentId]?.[2] || 'item/blank'}.png`);
  }

  return srcArray;
};
const equipmentRarityArray = (index) => {
  const equipmentId = equipmentIdArray.value[index];
  return `/src/assets/image/scene/general/rarity/item_${rarityMap[equipmentJson[equipmentId]?.[11]]?.label || 'empty'}.png`;
};
</script>

<style>
:root {
  --zoom: 2;

  --avatar-gap: 4px;
  --avatar-padding: 1.4px;
  --avatar-stroke: 0.6px;
  --avatar-width: 24px;
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
.image-rendering-pixelated {
  image-rendering: pixelated;
}

.party-board-container {
  display: flex;
  flex-direction: column;
  justify-content: center;

  width: 616px;
  padding: 16px;

  border-radius: 16px;
  box-shadow: 0 0 8px hsla(0, 0%, 0%, 0.2);
  gap: 12px;
  outline: 1px solid #aaaaaa;

  background-color: #fafafa;
  background-image: url('/src/assets/image/scene/general/sprite_sheet/decoration-assets/bottom_right_decoration.svg');
  background-position: right bottom;
  background-repeat: no-repeat;
  background-size: 160px auto;
}
.party-board-container > div:nth-child(1) {
  /* 上面那一行 */
  display: flex;
  flex-direction: row;
  justify-content: space-between;
  align-items: flex-end;

  height: 24px;
  line-height: 24px;
}
.party-board-container > div:nth-child(1) div {
  display: flex;
  flex-direction: row;
  justify-content: center;

  text-align: center;
}
.party-board-container > div:nth-child(1) > div:nth-child(1) > div:nth-child(1) {
  /* 序号标签 */
  width: 64px;
  margin-left: -16px;
  padding-right: 16px;

  font-family: 'SY';
  font-size: 16px;

  color: white;
}
.party-board-container > div:nth-child(1) > div:nth-child(1) > div:nth-child(2),
.party-board-container input {
  /* 队伍名称 */
  font-family: 'SY';
  font-size: 22px;
  font-weight: bold;
  background-color: transparent;
  height: 24px;

  appearance: none;
  border: none;
  color: #444444;
  width: 320px;
}
.party-board-container input:read-write {
  background-color: #aaaaaa;
}
.party-board-container > div:nth-child(1) > div:nth-child(2) {
  /* 两个队伍码div的容器 */
  display: flex;
  flex-direction: row;
  gap: 8px;
  text-align: center;
}
.party-board-container > div:nth-child(1) > div:nth-child(2) > div {
  /* abid和wfid */
  background-color: #5c5a7488;
  color: white;
  line-height: 28px;
  width: 96px;
  height: 28px;
  border-radius: 8px;
  cursor: pointer;
}
.content-area {
  justify-content: space-between;
}

.party-area {
  display: flex;
  flex-direction: row;

  gap: calc(var(--avatar-gap) * var(--zoom) * 1.5);
}
.party-area > div {
  display: grid;

  grid-gap: calc(var(--avatar-gap) * var(--zoom));
  grid-template-columns: auto auto auto;
}

.img-container {
  position: relative;

  display: flex;
  align-items: center;
  justify-content: center;

  width: calc((var(--avatar-width) + var(--avatar-padding) * 2) * var(--zoom));
  height: calc((var(--avatar-width) + var(--avatar-padding) * 2) * var(--zoom));
  padding: calc(var(--avatar-padding) * var(--zoom));

  background-color: #fafafa;
  border-radius: calc(var(--avatar-padding) * var(--zoom));
  outline: calc(var(--avatar-stroke) * var(--zoom)) solid #888888;
}
.party-area > div > div > img {
  /* 所有图片, 包括角色武器和稀有度 */
  position: absolute;
}
.party-area > div:nth-child(1) img,
.party-area > div img:nth-child(1) {
  /* 角色图片和稀有度图片 */
  width: calc(var(--avatar-width) * var(--zoom));

  object-fit: contain;
}
.party-area > div:nth-child(2) img:nth-child(2) {
  /* 武器图片 */
  transform: scale(var(--zoom));
}

.edit-button-area > button.edit-button {
  width: 32px;
  height: 32px;
  border-radius: 50%;
  border: none;
  color: #fafafa;
  background-color: orange;
  overflow: hidden;

  font-size: 22px;
  padding: 0 0 8px 8px;
  cursor: pointer;
}
</style>
