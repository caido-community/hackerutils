<script setup lang="ts">
import Button from "primevue/button";
import Dropdown from "primevue/dropdown";
import InputText from "primevue/inputtext";
import { ref } from "vue";

import { useSDK } from "@/plugins/sdk";

const sdk = useSDK();

type Unit = "seconds" | "milliseconds";

const unit = ref<Unit>("seconds");
const now = ref<string>("");
const input = ref<string>("");
const parsed = ref<string>("");

const options = [
  { label: "Seconds", value: "seconds" },
  { label: "Milliseconds", value: "milliseconds" },
];

const refresh = () => {
  const ms = Date.now();
  now.value =
    unit.value === "seconds" ? Math.floor(ms / 1000).toString() : ms.toString();
};

const parseInput = () => {
  const v = input.value.trim();
  if (v === "") {
    parsed.value = "";
    return;
  }
  const num = Number(v);
  if (Number.isNaN(num)) {
    sdk.window.showToast("Invalid timestamp", { variant: "error" });
    return;
  }
  const ms = v.length > 10 ? num : num * 1000;
  parsed.value = new Date(ms).toISOString();
};

const onCopyNow = async () => {
  await navigator.clipboard.writeText(now.value);
  sdk.window.showToast("Copied", { variant: "success" });
};
</script>

<template>
  <div class="h-full flex flex-col gap-4">
    <div class="flex items-center gap-2">
      <Dropdown
        v-model="unit"
        :options="options"
        option-label="label"
        option-value="value"
        class="w-40"
      />
      <Button label="Now" icon="fas fa-clock" @click="refresh" />
      <InputText v-model="now" readonly class="flex-1" />
      <Button label="Copy" icon="fas fa-copy" @click="onCopyNow" />
    </div>

    <div class="flex items-center gap-2">
      <InputText
        v-model="input"
        placeholder="Timestamp to ISO"
        class="flex-1"
      />
      <Button label="Parse" icon="fas fa-arrow-right" @click="parseInput" />
    </div>
    <InputText v-model="parsed" readonly />
  </div>
</template>
