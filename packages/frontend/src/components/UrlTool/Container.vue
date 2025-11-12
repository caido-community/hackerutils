<script setup lang="ts">
import Button from "primevue/button";
import Textarea from "primevue/textarea";
import { ref } from "vue";

import { useSDK } from "@/plugins/sdk";

const sdk = useSDK();

const input = ref("");
const output = ref("");

const onEncode = () => {
  try {
    output.value = encodeURIComponent(input.value);
  } catch (_e) {
    sdk.window.showToast("Failed to encode", { variant: "error" });
  }
};

const onDecode = () => {
  try {
    output.value = decodeURIComponent(input.value);
  } catch (_e) {
    sdk.window.showToast("Failed to decode", { variant: "error" });
  }
};

const onSwap = () => {
  const a = input.value;
  input.value = output.value;
  output.value = a;
};

const onCopy = async () => {
  const val = output.value;
  if (val === undefined) return;
  await navigator.clipboard.writeText(val);
  sdk.window.showToast("Copied", { variant: "success" });
};

const onClear = () => {
  input.value = "";
  output.value = "";
};
</script>

<template>
  <div class="h-full flex flex-col gap-3">
    <div class="flex gap-2">
      <Button label="Encode" icon="fas fa-arrow-down" @click="onEncode" />
      <Button label="Decode" icon="fas fa-arrow-up" @click="onDecode" />
      <Button label="Swap" icon="fas fa-right-left" @click="onSwap" />
      <div class="flex-1" />
      <Button label="Copy" icon="fas fa-copy" @click="onCopy" />
      <Button label="Clear" icon="fas fa-eraser" @click="onClear" />
    </div>
    <div class="grid grid-cols-2 gap-3 h-full">
      <Textarea v-model="input" class="h-full resize-none" rows="50" />
      <Textarea v-model="output" class="h-full resize-none" rows="50" />
    </div>
  </div>
</template>
