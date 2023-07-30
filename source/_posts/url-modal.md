---
title: url-modal
date: 2023-07-29 23:58:18
tags: Vue
---

Today I stumbled upon a cool tweet by Cory House (@housecor). He suggested using a URL search parameter, instead of a state to track whether a modal is open or closed.

As outlined by Cory House, these are the advantages of URL modals:
> 1. Users can bookmark or share a link to the open dialog.
> 2. If a dialog represents a unique location, it probably warrants a unique URL.
> 3. You can use a plain anchor to open the dialog. Semantically, an `<a>` is the proper semantic tag for navigation. No button or onClick is required.

Inspired by this idea, I implemented a simple url modal component using Vue 3.

### Overall idea
The idea is simple: use the URL hash to control the modal. Also, to keep track of the hash, we will rely on Vue 3's reactive data.

#### URL hash
The URL hash, the part of the URL after the '#' symbol, serves as a unique identifier for our modal. By altering the URL hash, we can decide when our modal should appear or disappear:
1. If the URL hash matches a certain value, we show the modal.
2. If it is changed or removed, we hide the modal.

#### Reactive hash value
To reactively track changes to the URL hash, we leverage Vue 3's reactivity system. We create a reactive reference to the window location hash, and update it whenever the URL hash changes. This allows our modal to respond dynamically to changes in the URL hash.

### Implementation
#### Listening url hash changes
To start, we need to listen for changes in the URL. For this, we create a reactive reference hash to the window location hash, and update it whenever the URL hash changes:
```ts
const hash = ref(window.location.hash)

const updateHash = () => {
  hash.value = window.location.hash;
}

onMounted(() => {
  window.addEventListener('hashchange', updateHash)
})

onUnmounted(() => {
  window.removeEventListener('hashchange', updateHash)
})
```
#### Modal control events
##### showModal
With the hash reactive reference in place, we can control the visibility of our modal. It is displayed when the hash matches the modalId prop of our modal:
```ts
const showModal = computed(() => {
  return hash.value === `#${props.modalId}`
})
```

##### closeModal
To close the modal, we remove the modal ID from the URL and reset our reactive hash:
```ts
const closeModal = () => {
  resumeBrowserUrl();
  resetReactiveHash();
}

const resumeBrowserUrl = () => {
  const urlWithoutModalId = window.location.origin + '/' + window.location.search;
  window.history.pushState({ path: urlWithoutModalId }, '', urlWithoutModalId);
}

function resetReactiveHash() {
  hash.value = "";
}
```

#### Adding Template
##### 1. Modal trigger, a link. 
Clicking on it will change the URL hash to the modal ID, which opens the modal. `textLink` is a prop given by users of the component.
```vue
<a :href="formatHref(props.modalId)">{{ textLink }}</a>
```
##### 2. Dimmed background behind the modal. 
It's only visible when showModal is true. Clicking on the backdrop will close the modal.
```
<div class="modal-backdrop" @click="closeModal" v-if="showModal">...</div>
```
##### 3. The modal itself. 
`.stop` prevents clicks inside the modal from reaching the backdrop and closing the modal.

```vue
<div class="modal-content" @click.stop>...</div>
````
##### 4. Content Slot
This slot allows users of the component to insert their own content into the modal.
```vue
<slot name="content" :closeModal="closeModal" />
```
#### Full Code
```vue
<template>
  <a :href="formatHref(props.modalId)">{{ linkText }}</a>

  <transition name="modal-fade">
    <div class="modal-backdrop" @click="closeModal" v-if="showModal" >
      <div class="modal-content" @click.stop>
        <slot name="content" :closeModal="closeModal" />
      </div>
    </div>
  </transition>
</template>

<script setup>
import { computed, onMounted, onUnmounted, ref } from 'vue';

const props = defineProps({
  modalId: {
    type: String,
    required: true
  },
  linkText: {
    type: String,
    required: true
  }
})

const hash = ref(window.location.hash)

const formatHref = (modalId) => `#${modalId}`

const updateHash = () => {
  hash.value = window.location.hash;
}

onMounted(() => {
  window.addEventListener('hashchange', updateHash)
})

onUnmounted(() => {
  window.removeEventListener('hashchange', updateHash)
})

const showModal = computed(() => {
  return hash.value === formatHref(props.modalId)
})

const closeModal = () => {
  resumeBrowserUrl();
  resetReactiveHash();
}

const resumeBrowserUrl = () => {
  const urlWithoutModalId = window.location.origin + '/' + window.location.search;
  window.history.pushState({ path: urlWithoutModalId }, '', urlWithoutModalId);
}

function resetReactiveHash() {
  hash.value = "";
}

</script>

<style scoped>
@import './urlModal.css';
</style>
```

#### Usage Example
```vue
<script setup>
import UrlModal from "@/components/UrlModal";
const URL_MODAL_BENEFITS = [
  "✅ Users can bookmark or share a link to the open dialog.",
  "✅ If a dialog represents a unique location, it probably warrants a unique URL.",
  "✅ You can use a plain anchor to open the dialog. Semantically, an <a> is the proper semantic tag for navigation. No button or onClick is required."
]
</script>

<template>
  <UrlModal modal-id="my-modal" linkText="Why we use url modal?" >
    <template #content="{ closeModal }">
      <h2>Modal Benefits (@housecor)</h2>
      <p v-for="text in URL_MODAL_BENEFITS">{{ text }}</p>
      <button @click="closeModal">Close Modal</button>
    </template>
  </UrlModal>
</template>
```

<img src="https://github.com/flaming-cl/flaming-cl.github.io/assets/51183663/d9d60b1c-11d2-49e2-941e-7f1bac73e3c2" width="600" height="600">

And there you have it! A neat, little URL modal made with Vue 3, inspired by a tweet from Cory House. It's an interesting twist on the usual way of doing things, and maybe it could give you some ideas for your next project. 🚀
