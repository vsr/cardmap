<script setup lang="ts">
import { reactive, computed, ref } from 'vue'

interface DraggingState {
  draggingCard: EventTarget | null;
  activeCard: EventTarget | null;
}
interface Card {
  text: String;
  id: string;
  position?: {
    x: number;
    y: number;
  }
}

const cards: Card[] = reactive([
  {text: "Hello", id: '24324'},
  {text: "There",id: '243212'},
  {text: "People", id: '243434'},
  {text: "Of", id: '2899'},
]);
const draggingState: DraggingState = reactive({draggingCard: null, activeCard: null});
const cardContainerRef = ref(null);

const cardId = (card: HTMLDivElement | null) => card && card.getAttribute('id') || null;
const getCardById = (id: String | null) => cards.find(card => card.id===id);

const updateDraggingCard = function(draggingCard: EventTarget | null){
  draggingState.draggingCard = draggingCard;
  if(draggingCard) {
    draggingState.activeCard = draggingState.draggingCard;
    const card = getCardById(cardId(draggingCard as HTMLDivElement));
    if(card && !card.position) {
      const bound = (draggingCard as HTMLDivElement).getBoundingClientRect();
      card.position = {
        x: bound.x, y: bound.y
      }
    }
  }
}

const mousedown = (ev: Event) => {
  console.log('mousedown', ev.target);
  const card = ev.target;
  if(card && (card as HTMLDivElement).classList.contains('card')) {
    updateDraggingCard(card);
  }
}
const mousemove = (ev: MouseEvent) => {
  if(draggingState.draggingCard) {
    const card = getCardById(cardId(draggingState.draggingCard as HTMLDivElement));
    let translateX = ev.clientX;
    let translateY = ev.clientY;
    if(card?.position) {
      translateX -= card.position.x;
      translateY -= card.position.y;
    }
    (draggingState.draggingCard as HTMLDivElement).style.transform = `translate(${translateX}px, ${translateY}px)`;
  }
}
const mouseup = (ev: Event) => {
  console.log('mouseup', ev);
  updateDraggingCard(null);
}
const mouseleave = (ev:Event) => {
  updateDraggingCard(null);
}

const resetCards = () => {
  console.log('reset', cardContainerRef.value);
  const el = cardContainerRef.value as HTMLDivElement | null;
  if(el) {
    Array.from(el.children).forEach(card => (card as HTMLDivElement).style.transform = '');
  }
}

const isActiveCard = (card: Card) =>
  card && cardId(draggingState.activeCard as HTMLDivElement) === card.id;
const isDraggingCard = (card: Card) =>
  card && cardId(draggingState.draggingCard as HTMLDivElement) === card.id;
</script>

<template>
  <div class="w-screen h-screen relative bg-slate-100 flex flex-wrap"
    ref="cardContainerRef"
    @mousedown="mousedown" @mouseup="mouseup" @mousemove="mousemove" @mouseleave="mouseleave">
    <div v-for="card in cards"
    :class="{'border-red-600 z-10 shadow-sm': isActiveCard(card), 'bg-opacity-95 shadow-lg z-30': isDraggingCard(card)}"
     class="card select-none w-32 h-32 flex place-content-center items-center p-1 m-2 rounded border-2 bg-slate-50 shadow-sm duration-75 ease-linear transition-transform"
     :id="card.id">{{card.text}}</div>

     <div class="fixed bottom-0 w-full bg-white p-2 text-sm">
       <button class="p-2 rounded bg-slate-800 text-slate-100" @click="resetCards">Reset</button>
     </div>
  </div>
</template>

<style scoped>

</style>
