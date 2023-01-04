<template>
  <div
    class="flex flex-col h-full justify-center gap-8 max-w-prose px-8 py-8 mx-auto text-slate-900 dark:text-slate-100"
  >
    <header class="flex flex-col">
      <h1 class="text-4xl mb-2">LGTMuse</h1>
      <h2 class="text-xl mb text-slate-500 dark:text-slate-400">
        A muse for what looks good to you
      </h2>
      <h3 class="text-xl italic text-slate-500 dark:text-slate-400">
        ...on a book cover!
      </h3>
    </header>

    <main
      class="rounded-2xl p-4 bg-amber-200 dark:bg-yellow-900 dark:text-slate-200 saturate-50"
    >
      <div class="flex flex-col gap-4">
        <h2 class="capitalize text-xl">
          <span>{{ title }}</span>
          <span
            v-if="subtitle"
            class="capitalize text-base text-slate-500 dark:text-slate-400"
            >...{{ subtitle }}</span
          >
        </h2>
        <LoaderEllipsis v-if="loading" />
        <div class="text-base text-justify flex flex-col gap-2">
          <div v-for="line in summary" :key="line">{{ line }}</div>
        </div>
      </div>
    </main>
    <transition
      enter-active-class="duration-300 ease-out"
      enter-from-class="transform opacity-0 -translate-y-10"
      leave-active-class="duration-300 ease-in"
      leave-to-class="transform opacity-0 -translate-y-10"
    >
      <div
        v-if="!loading"
        class="self-center text-xl flex flex-col items-center"
      >
        <h2>Coming soon to a bookstore near you!</h2>
        <h2>...Or, maybe not.</h2>
        <button
          class="mt-8 px-4 py-1 rounded-full bg-amber-200 dark:bg-yellow-900 saturate-50"
          @click="loadGoodTitleMetadata"
        >
          Try again?
        </button>
      </div>
    </transition>
    <div class="text-xs italic text-slate-500 dark:text-slate-400 text-center">
      Title brought to you by
      <a href="https://datamuse.com/" target="_blank">Datamuse</a>. Synopsis
      brought to you by
      <a href="https://beta.openai.com/" target="_blank">OpenAI</a>. Derived
      content does not reflect this site's values or those of its author in any
      way.
      <br />
      Happy New Year!
    </div>
  </div>
</template>

<script setup lang="ts">
import LoaderEllipsis from "./components/LoaderEllipsis.vue";
import { onMounted, ref } from "vue";

const TITLE_PLACEHOLDERS = [
  "Looking...",
  "Guessing...",
  "Thinking...",
  "Musing...",
];

const PLACEHOLDERS = [
  "Figuring out a killer novel...",
  "Determining the one book to rule them all...",
  "Absorbing the universe, all for you and your book...",
  "Vanquishing writer's block...",
];

function rval<T>(arr: T[]) {
  return arr[Math.floor(Math.random() * arr.length)];
}

const loading = ref(true);
const title = ref("");
const subtitle = ref("");
const summary = ref<string[]>([]);

async function loadGoodTitleMetadata() {
  loading.value = true;
  title.value = rval(TITLE_PLACEHOLDERS);
  subtitle.value = "";
  summary.value = [rval(PLACEHOLDERS)];

  try {
    const deriveReq = await fetch(
      "https://blw6096nkc.execute-api.us-west-2.amazonaws.com/Prod/derive?acronym=LGTM"
    );
    if (!deriveReq.ok) {
      throw new Error("Request failed.");
    }
    title.value = (await deriveReq.json()).derived;

    const summaryReq = await fetch(
      "https://blw6096nkc.execute-api.us-west-2.amazonaws.com/Prod/summarize?" +
        new URLSearchParams({ title: title.value })
    );
    if (!summaryReq.ok) {
      throw new Error("Request failed.");
    }
    const json = await summaryReq.json();
    summary.value = json.summary;
    subtitle.value = json.subtitle ?? subtitle.value;
  } catch (err) {
    console.error("Error occurred calling API:", err);
  }

  loading.value = false;
}

onMounted(loadGoodTitleMetadata);
</script>
