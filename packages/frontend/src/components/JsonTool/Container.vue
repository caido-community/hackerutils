<script setup lang="ts">
import Button from "primevue/button";
import InputNumber from "primevue/inputnumber";
import Textarea from "primevue/textarea";
import { ref } from "vue";

import { useSDK } from "@/plugins/sdk";

const sdk = useSDK();

const input = ref("");
const output = ref("");
const indent = ref<number>(2);

const parseSmartJson = (raw: string): unknown => {
  const text = raw.trim();
  if (text.length === 0) return "";

  const parseMulti = (t: string): unknown => {
    const first = JSON.parse(t);
    let val: unknown = first;
    let depth = 0;
    while (typeof val === "string" && depth < 3) {
      const inner = val.trim();
      try {
        val = JSON.parse(inner);
        depth += 1;
      } catch (_e) {
        break;
      }
    }
    return val;
  };

  try {
    return parseMulti(text);
  } catch (_e) {
    const sanitized = text
      .replace(/\r/g, "\\r")
      .replace(/\n/g, "\\n")
      .replace(/\t/g, "\\t");

    try {
      const decoded = JSON.parse(`"${sanitized}"`) as string;
      try {
        return parseMulti(decoded);
      } catch (_ee) {
        return decoded;
      }
    } catch (_eee) {
      throw new Error("Invalid JSON");
    }
  }
};

const onBeautify = () => {
  try {
    const obj = parseSmartJson(input.value);
    output.value = JSON.stringify(obj, undefined, indent.value);
  } catch (_e) {
    sdk.window.showToast("Invalid JSON", { variant: "error" });
  }
};

const onMinify = () => {
  try {
    const obj = parseSmartJson(input.value);
    output.value = JSON.stringify(obj);
  } catch (_e) {
    sdk.window.showToast("Invalid JSON", { variant: "error" });
  }
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
    <div class="flex gap-2 items-center">
      <Button
        label="Beautify"
        icon="fas fa-wand-magic-sparkles"
        @click="onBeautify"
      />
      <Button label="Minify" icon="fas fa-compress" @click="onMinify" />
      <div class="flex-1" />
      <span class="text-sm px-2">Indent</span>
      <InputNumber
        v-model="indent"
        :min="0"
        :max="8"
        show-buttons
        class="w-28"
      />
      <Button label="Copy" icon="fas fa-copy" @click="onCopy" />
      <Button label="Clear" icon="fas fa-eraser" @click="onClear" />
    </div>
    <div class="grid grid-cols-2 gap-3 h-full">
      <Textarea v-model="input" class="h-full resize-none" rows="50" />
      <Textarea v-model="output" class="h-full resize-none" rows="50" />
    </div>
  </div>
</template>
