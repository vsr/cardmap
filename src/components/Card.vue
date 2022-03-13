<script setup lang="ts">
import { reactive, computed, ref, nextTick } from 'vue';

interface CardInterface {
  text: String;
  id: string;
  tags: String[];
}

const textEl = ref(null);
const tagsEl = ref(null);

const props = defineProps({
  card: { type: Object, required: true },
  onUpdate: {type: Function, required: true}
});

const state: {
    editable: boolean
    tagsEditing: boolean
} = reactive({editable: false, tagsEditing: false});

const edit = (editable:boolean) => {
    state.editable = editable;
    if(editable) {
        nextTick(() => {
            textEl.value && (textEl.value as HTMLDivElement).focus();
        })
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
                const tags = el.value.split(",");
                props.onUpdate({...props.card, tags })
            }
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
</script>
<template>
<div class="relative overflow-hidden flex flex-col" >

    <span class="text-xl w-full min-h-[2em] bg-white flex place-content-center items-center overflow-x-hidden"
        @dblclick="edit(true)"
        @focusout="edit(false)"
        :contenteditable="state.editable"
        @input="change"
        ref="textEl" >
        {{props.card.text}}
    </span>
    <span class="min-h-[1em] p-1 text-xs  flex-grow flex-row w-full  flex-wrap gap-1 "
        @dblclick="tagsEdit(true)"
        @touchend="tagTap"
        >
        <span class="px-2 py-1 bg-slate-800 text-slate-50 inline-flex rounded border shadow-sm" v-if="props.card.tags && !state.tagsEditing" v-for="tag in props.card.tags">{{tag}}</span>
        <input @focusout="tagsEdit(false)" v-if="state.tagsEditing" class="" ref="tagsEl" name="tags" aria-label="Tags" />
    </span>

</div>
</template>
<style scoped>
[contenteditable="true"] {
    @apply border-2 border-yellow-500 outline-none;
}
</style>