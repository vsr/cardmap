<script setup lang="ts">
import { reactive, computed, ref, nextTick } from 'vue';

interface CardInterface {
  text: String;
  id: string;
  tags: String[];
}

const cardEl = ref(null);
const textEl = ref(null);
const tagsEl = ref(null);

const props = defineProps({
  card: { type: Object, required: true },
  onUpdate: {type: Function, required: true},
  onCardDelete: {type: Function, required: true},
});

const state: {
    editable: boolean
    tagsEditing: boolean
} = reactive({editable: false, tagsEditing: false});

const focusCard = () => cardEl.value && (cardEl.value as HTMLDivElement).focus();

const edit = (editable:boolean) => {
    state.editable = editable;
    if(editable) {
        nextTick(() => {
            textEl.value && (textEl.value as HTMLDivElement).focus();
        })
    } else {
        focusCard();
    }
}

const tagsEdit = (editable:boolean) => {
    state.tagsEditing = editable;
    if(editable) {
        nextTick(() => {
            if(tagsEl.value) {
                const el = tagsEl.value as HTMLInputElement;
                el.value = (props.card.tags || []).join(",");
                el.focus();
            }
        })
    } else {
            if(tagsEl.value) {
                const el = tagsEl.value as HTMLInputElement;
                const tags = el.value.split(",").map(tag => tag.trim());
                props.onUpdate({...props.card, tags })
            }
            focusCard();
    }
}
const change = () => {
    if(textEl.value) {
        const text = (textEl.value as HTMLDivElement).textContent
        props.onUpdate({...props.card, text })
    }
}

const tagTap = (function detectDoubleTapClosure() {
  let lastTap = 0;
  let timeout:number;
  return function detectDoubleTap(event:TouchEvent) {
    const curTime = new Date().getTime();
    const tapLen = curTime - lastTap;
    if (tapLen < 500 && tapLen > 0) {
      tagsEdit(true)
      event.preventDefault();
    } else {
      timeout = setTimeout(() => {
        clearTimeout(timeout);
      }, 500);
    }
    lastTap = curTime;
  };
}());

const cardKeypress = (ev: KeyboardEvent) => {
    console.log('ev', ev);
    switch(ev.key) {
        case 'e':
            ev.preventDefault();
            return edit(true);
        case 't':
            ev.preventDefault();
            return tagsEdit(true);
        default:
            return;
    }
}

const noop = (ev:KeyboardEvent) =>{};
const keydown = (ev:KeyboardEvent) =>{
    if(ev.key === 'Escape') {
        edit(false);
        tagsEdit(false);
    }
};
const deleteCard = () => {
    confirm("You sure want to delete this card?") && props.onCardDelete(props.card);
}
</script>
<template>
<div class="relative group overflow-hidden flex flex-col focus:border-indigo-600 select-none min-h-[4em] min-w-[12em] max-w-[24em] max-h-[24em]  place-content-center items-center p-1 m-2   duration-75 ease-linear transition-transform bg-white rounded-md shadow " tabindex="0"
    ref="cardEl" :aria-label="props.card.text" role="list"
    @keypress="cardKeypress">
    <button @click.stop="deleteCard"
         class="absolute right-0 top-0 px-2 py-1 font-bold bg-pink-300 text-slate-200 focus:bg-pink-500  text-xs opacity-0 group-hover:opacity-100 group-focus:opacity-100 hover:opacity-100 focus:opacity-100 outline-none">delete</button>

    <span class="my-3 text-xl w-full min-h-[2em] bg-white flex place-content-center items-center overflow-x-hidden border-b border-slate-200"
        @dblclick="edit(true)"
        @focusout="edit(false)"
        :contenteditable="state.editable"
        @input.stop.prevent="change"
        @keypress.stop="noop"
        @keydown="keydown"
        ref="textEl" >
        {{props.card.text}}
    </span>
    <span class="min-h-[1em] text-xs  flex-grow flex-row w-full  flex-wrap gap-1 "
        @dblclick="tagsEdit(true)"
        @touchend="tagTap"
        >
        <span class="px-2 py-1 bg-slate-800 text-slate-50 inline-flex rounded border shadow-sm" v-if="props.card.tags && !state.tagsEditing" v-for="tag in props.card.tags">{{tag}}</span>
        <input
            class="shadow-sm text-md font-mono p-1 ring-2 ring-indigo-600 outline-none block w-full sm:text-sm border-gray-300 rounded-sm"
            placeholder="Enter comma separated words for tags"
            @focusout="tagsEdit(false)"
            @keypress.stop="noop"
            @keydown="keydown"
            v-if="state.tagsEditing"  ref="tagsEl" name="tags" aria-label="Tags" />
    </span>

</div>
</template>
<style scoped>
[contenteditable="true"] {
    @apply rounded-sm bg-yellow-50 outline-none;
}
</style>