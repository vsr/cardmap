<script setup lang="ts">
import { reactive, computed, ref, nextTick } from 'vue';

interface Card {
  text: String;
  id: string;
  position?: {
    x: number;
    y: number;
  }
  tags: String[];
}

const textEl = ref(null);

const props = defineProps({
  card: { type: Object, required: true },
});
console.log('props', props);

const state: {
    editable: boolean
} = reactive({editable: false});

const edit = (editable:boolean) => {
    state.editable = editable;
    if(editable) {
        nextTick(() => {
            textEl.value && (textEl.value as HTMLDivElement).focus();
        })
    }
}

</script>
<template>
<div class="relative overflow-hidden" @dblclick="edit(true)"
        @focusout="edit(false)">

    <span class="text-xl min-h-[1em] min-w-[8em]" :contenteditable="state.editable"
        ref="textEl" >
        {{props.card.text}}
    </span>
    <span class="absolute bottom-0 flex flex-wrap gap-1">
        <span class="" v-if="props.card.tags" v-for="tag in props.card.tags">{{tag}}</span>
    </span>
</div>
</template>
<style scoped>
[contenteditable="true"] {
    @apply border-2 border-yellow-500 outline-none;
}
</style>