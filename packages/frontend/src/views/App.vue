<script setup lang="ts">
import Card from "primevue/card";
import Listbox from "primevue/listbox";
import Splitter from "primevue/splitter";
import SplitterPanel from "primevue/splitterpanel";
import { type Component, computed, ref } from "vue";

import { Base64Tool } from "@/components/Base64Tool";
import { JsonTool } from "@/components/JsonTool";
import { TimestampTool } from "@/components/TimestampTool";
import { UrlTool } from "@/components/UrlTool";
import { UuidTool } from "@/components/UuidTool";
import { WordlistsTool } from "@/components/WordlistsTool";
import { JsonEscapeTool } from "@/components/JsonEscapeTool";

type Page = {
  id: string;
  name: string;
  icon: string;
  comp: Component;
};

const pages = ref<Page[]>([
  { id: "base64", name: "Base64", icon: "fas fa-lock", comp: Base64Tool },
  { id: "url", name: "URL Encoding", icon: "fas fa-link", comp: UrlTool },
  { id: "json", name: "JSON", icon: "fas fa-code", comp: JsonTool },
  {
    id: "json-escape",
    name: "JSON Escape",
    icon: "fas fa-quote-right",
    comp: JsonEscapeTool,
  },
  { id: "uuid", name: "UUID", icon: "fas fa-fingerprint", comp: UuidTool },
  { id: "time", name: "Timestamp", icon: "fas fa-clock", comp: TimestampTool },
  {
    id: "wordlists",
    name: "Wordlists",
    icon: "fas fa-list",
    comp: WordlistsTool,
  },
]);

const activeId = ref<string>(pages.value[0]!.id);

const current = computed(() =>
  pages.value.find((p) => p.id === activeId.value)
);
</script>

<template>
  <div class="h-full w-full">
    <Splitter
      layout="horizontal"
      class="h-full"
      :pt="{ gutter: { class: 'bg-transparent' }, root: { class: 'h-full' } }"
    >
      <SplitterPanel :size="20" :min-size="15" class="h-full">
        <Card
          class="h-full"
          :pt="{ body: { class: 'h-full p-0' }, content: { class: 'h-full' } }"
        >
          <template #content>
            <div class="h-full flex flex-col">
              <div
                class="p-3 font-bold border-b border-surface-700"
              >
                Tools
              </div>
              <div class="p-2 h-full">
                <Listbox
                  v-model="activeId"
                  :options="pages"
                  option-label="name"
                  option-value="id"
                  class="w-full h-fit"
                  listStyle="max-height:500px"
                >
                  <template #option="{ option }">
                    <div class="flex items-center gap-2">
                      <i class="w-4" :class="option.icon" />
                      <span>{{ option.name }}</span>
                    </div>
                  </template>
                </Listbox>
              </div>
            </div>
          </template>
        </Card>
      </SplitterPanel>
      <SplitterPanel :size="80" :min-size="30" class="h-full">
        <Card
          class="h-full"
          :pt="{
            body: { class: 'h-full p-0' },
            content: { class: 'h-full flex flex-col' },
          }"
        >
          <template #content>
            <div class="h-full p-3">
              <component :is="current?.comp" />
            </div>
          </template>
        </Card>
      </SplitterPanel>
    </Splitter>
  </div>
</template>
