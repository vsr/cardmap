<script setup lang="ts">
import { reactive, computed, onMounted } from 'vue';
import Card from "./Card.vue";

export interface CardInterface {
  text: String;
  id: string;
  tags: String[];
}
const LS_CARD = "CARD_DATA";
const data: {cards: CardInterface[], tagSelection: Set<String>} = reactive({cards: [], tagSelection: new Set()});
const tags = computed(() => {
  const tagSet: Set<String> = new Set();
  data.cards.forEach(card => card.tags && card.tags.forEach(tag => tagSet.add(tag)));
  return Array.from(tagSet) as String[];
});
const filteredCards = computed(() => {
  if(data.tagSelection.size === 0) return data.cards;
  return data.cards.filter(card => {
    let filter = false;
    if(card.tags) {
      card.tags.forEach(tag => {
        if(data.tagSelection.has(tag)) filter = true;
      });
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
    data.cards[index] = {...data.cards[index], ...cardData };
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


const add = () => data.cards.push({text: "", tags: [], id: Date.now().toString() });

const toggleTagSelection = (tag:String) => {
  if(data.tagSelection.has(tag)) {
    data.tagSelection.delete(tag);
  } else {
    data.tagSelection.add(tag);
  }
};
const isTagSelected = (tag:String) => data.tagSelection.has(tag);

const clearTagSelection = () => data.tagSelection.clear();
</script>

<template>
  <div class="w-screen h-screen relative bg-slate-100">
    <ul role="list" class="flex flex-wrap content-start" aria-label="Cards">
      <Card v-for="card in filteredCards"
      :onUpdate="onCardUpdate"
      :onCardDelete="onCardDelete"
      class="card "
      :id="card.id" :card="card"></Card>
     </ul>
     <div class="fixed bottom-0 w-full bg-white p-2 text-sm flex ">
       <div class="flex-grow overflow-x-auto">
         <div class="flex gap-1">
           <span class="p-1 rounded border cursor-pointer select-none"
           :class="{'bg-indigo-700 text-indigo-50': isTagSelected(tag)}"
            @click="toggleTagSelection(tag)" v-for="tag in tags">{{tag}}</span>
         </div>
       </div>
       <div class="flex gap-1">
         <button class="p-2 rounded bg-indigo-700 text-slate-100" @click="clearTagSelection">View All</button>
         <button class="p-2 rounded bg-indigo-700 text-slate-100" @click="add">Add</button>
       </div>
     </div>
  </div>
</template>
