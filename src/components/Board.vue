<script setup lang="ts">
import { reactive, computed, onMounted } from 'vue';
import Card from "./Card.vue";
import AddIcon from './AddIcon.vue';
import CheckIcon from './CheckIcon.vue';
import MinusIcon from './MinusIcon.vue';
import ClearIcon from './ClearIcon.vue';

export interface CardInterface {
  text: String;
  id: string;
  tags: String[];
}
const LS_CARD = "CARD_DATA";
const colors = [
  "bg-red-600",
  "bg-orange-600",
  "bg-amber-600",
  "bg-yellow-600",
  "bg-lime-600",
  "bg-green-600",
  "bg-teal-600",
  "bg-cyan-600",
  "bg-sky-600",
  "bg-blue-600",
  "bg-indigo-600",
  "bg-violet-600",
  "bg-purple-600",
  "bg-fuchisia-600",
  "bg-pink-600",
  "bg-rose-600"
];


const data: { cards: CardInterface[], tagSelection: Set<String>, tagColorMap: { [key: string]: string }, } = reactive({
  cards: [],
  tagSelection: new Set(),
  tagColorMap: {},
});
const nextColor = () => colors[Math.floor(Math.random() * colors.length)];;

const tags = computed(() => {
  const tagSet: Set<String> = new Set();
  data.cards.forEach(card => card.tags && card.tags.forEach(tag => tagSet.add(tag)));
  const tagsList = Array.from(tagSet) as String[];
  tagsList.forEach((tag) => {
    if (!data.tagColorMap[tag as string]) {
      data.tagColorMap[tag as string] = nextColor();
    }
  })
  return tagsList.sort();
});




const filteredCards = computed(() => {
  if (data.tagSelection.size === 0) return data.cards;
  return data.cards.filter(card => {
    let filter = true;
    if (card.tags) {
      data.tagSelection.forEach(tag => {
        if (!card.tags.includes(tag)) {
          filter = false;
        }
      })
    }
    return filter;
  })
});


onMounted(() => {
  try {
    const lsData = window.localStorage.getItem(LS_CARD);
    if (lsData) {
      data.cards = JSON.parse(lsData).cards;
    }
  } catch (err) {
    console.error("Error loading data", err);
  }
});

const save = () => window.localStorage.setItem(LS_CARD, JSON.stringify(data));

const onCardUpdate = (cardData: CardInterface) => {
  const index = data.cards.findIndex((card) => card.id === cardData.id);
  if (index !== -1) {
    data.cards[index] = { ...data.cards[index], ...cardData };
  }
  save();
};

const onCardDelete = (cardData: CardInterface) => {
  const index = data.cards.findIndex((card) => card.id === cardData.id);
  if (index !== -1) {
    data.cards.splice(index, 1);
  }
  save();
};


const add = () => data.cards.push({ text: "", tags: [], id: Date.now().toString() });

const toggleTagSelection = (tag: String) => {
  if (data.tagSelection.has(tag)) {
    data.tagSelection.delete(tag);
  } else {
    data.tagSelection.add(tag);
  }
};
const isTagSelected = (tag: String) => data.tagSelection.has(tag);

const clearTagSelection = () => data.tagSelection.clear();
</script>

<template>
  <div class="w-screen h-screen relative bg-slate-100 flex flex-col">
    <ul
      role="list"
      class="flex flex-wrap content-start flex-grow overflow-y-auto pb-16"
      aria-label="Cards"
    >
      <Card
        v-for="card in filteredCards"
        :onUpdate="onCardUpdate"
        :onCardDelete="onCardDelete"
        :tagColorMap="data.tagColorMap"
        class="card"
        :id="card.id"
        :card="card"
      ></Card>
    </ul>
    <div class="fixed bottom-0 w-full bg-white max-h-[4em] p-0 text-sm flex shadow-xl">
      <div class="flex-grow flex self-center pr-2 border-r border-slate-300">
        <ul class="p-2 overflow-x-auto flex-grow" role="list" aria-label="Filter by Tags">
          <li class="flex gap-1" role="item">
            <button
              class="p-1 rounded border cursor-pointer select-none bg-slate-600 text-slate-50 flex"
              role="button"
              tabindex="0"
              :class="{ 'ring-2 ring-slate-600': isTagSelected(tag), [`${data.tagColorMap[tag as string]}`]: true }"
              @click="toggleTagSelection(tag)"
              v-for="tag in tags"
            >
              <CheckIcon class="mr-1 self-center" v-if="isTagSelected(tag)" />
              <MinusIcon class="mr-1 self-center" v-if="!isTagSelected(tag)" />
              {{ tag }}
            </button>
          </li>
        </ul>
        <button
          class="p-1 self-center text-sm rounded cursor-pointer select-none bg-transparent text-slate-600 flex"
          @click="clearTagSelection"
          aria-label="Clear Tag filter"
        >
          <ClearIcon />
        </button>
      </div>
      <div class="flex p-2 justify-end gap-1 min-w-[10em] drop-shadow">
        <button
          class="inline-flex items-center px-2 py-1 border border-transparent text-sm font-medium rounded-md shadow-sm text-white bg-green-600 hover:bg-green-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-gray-900"
          @click="add"
        >
          <AddIcon role="presentation" class="mr-1" />Add Card
        </button>
      </div>
    </div>
  </div>
</template>
