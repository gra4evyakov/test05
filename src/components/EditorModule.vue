<script setup>
import {shallowRef, ref } from 'vue'
import SvgUndo from "@/assets/SvgUndo";
import SvgHeading from "@/assets/SvgHeading";
import SvgParagraph from "@/assets/SvgParagraph";
import SvgImportImg from "@/assets/SvgImportImg";

const stack = ref('');
const undoStack = ref([]);
const redoStack = ref([]);
const editor = ref(null);

const buttons = shallowRef([
  {id: 1, name: SvgUndo, class: 'undo', click: () => makeUndo()},
  {id: 2, name: SvgUndo, class: 'redo', click: () =>  makeRedo()},
  {id: 3, name: SvgHeading, class: 'heading', click: () =>  makeHeading()},
  {id: 4, name: SvgParagraph, class: 'paragraph', click: () =>  makeParagraph()},
  {id: 5, name: SvgImportImg, class: 'import-img', click:  () => importImage()},
  {id: 6, name: 'copy-html', class: 'copy-html', click:  () => copyAsHtml(), text: 'Скопировать HTML'},
])


function updateValue() {
  stack.value = editor.value.innerHTML;
  undoStack.value.push(stack.value);
}

function makeUndo() {
  if (undoStack.value.length > 0) {
    redoStack.value.push(stack.value);
    stack.value = undoStack.value.pop();
    editor.value.innerHTML = stack.value;
  }
}

function makeRedo() {
  if (redoStack.value.length > 0) {
    undoStack.value.push(stack.value);
    stack.value = redoStack.value.pop();
    editor.value.innerHTML = stack.value;
  }
}

function makeHeading() {
  document.execCommand('formatBlock', false, 'h1');
}

function makeParagraph() {
  document.execCommand('formatBlock', false, 'p');
}

function importImage() {
  const url = prompt('Enter image URL:');
  document.execCommand('insertImage', false, url);
}

function copyAsHtml() {
  const selectedHTML = document.createElement('textarea');
  selectedHTML.value = editor.value.innerHTML;
  document.body.appendChild(selectedHTML);
  selectedHTML.select();
  document.execCommand('copy');
  selectedHTML.remove();
  alert('Copied HTML to clipboard');
}

</script>

<template>
  <div class="wysiwyg-editor">
    <div class="toolbar-buttons">
        <button v-for="button in buttons" @click="button.click" :class="`toolbar-button--${button.class}`" :key="button.id">
          <template v-if="button.name === 'copy-html'">
            {{ button.text }}
          </template>
          <component v-else :is="button.name" />
        </button>
    </div>
    <div contenteditable="true" ref="editor" @input="updateValue">
      {{ stack ? null : 'Click and type something!' }}
    </div>
  </div>
</template>

<style lang="scss">

.wysiwyg-editor {
  .toolbar-buttons {
    display: flex;
    padding: 10px;
    gap: 12px;
  }

  button {
    background-color: #282828;
    border: 1px solid #282828;
    border-radius: 4px;
    cursor: pointer;
    font-size: 16px;
    display: flex;
    align-items: center;
    justify-content: center;
    width: 42px;
    height: 38px;
    transition: .4s;

    &.toolbar-button--copy-html {
      background-color: transparent;
      border: unset;
      color: #639EFF;
      width: unset;
      height: unset;
    }

    &.toolbar-button--redo {
      transform: scale(-1, 1);
    }

    &:hover {
      color: #ccc;

      svg path {
        transition: .4s;
        fill: #ccc;
      }
    }
  }

  [contenteditable="true"] {
    padding: 10px;
    min-height: 200px;
    font-size: 18px;
    display: flex;
    flex-direction: column;
    text-align: left;

    p {
      max-width: 620px;
      margin-block: 30px;
    }

    h1 {
      font-size: 31px;
      line-height: 74%;
      color: #FFFFFF;
      margin-block: 46px 33px;
    }

    img {
      max-width: 100%;
    }
  }
}
</style>
