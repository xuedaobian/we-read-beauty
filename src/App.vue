<template>
  <div>
    <h1>Beauty</h1>
    <div v-html="content" class="cur-active"></div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import MarkdownIt from 'markdown-it';
import { fetchEventSource } from '@microsoft/fetch-event-source';

const md = new MarkdownIt();
const content = ref('')

onMounted(() => {
  eventSourceHandle('http://localhost:8000/send-file');
})
const eventSourceHandle = async (url) => {
  const controller = new AbortController()
  const signal = controller.signal

  fetchEventSource(`${url}`, {
    method: 'POST',
    signal: signal,
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify(''),
    onmessage(msg) {
      if (msg.event === '') {
        content.value += md.render(JSON.parse(msg.data));
      }
      else if (msg.event === 'close') {
        controller.abort();
      }
    },
    onerror(err) {
      throw err;    //必须throw才能停止 
    }
  });
}


</script>
