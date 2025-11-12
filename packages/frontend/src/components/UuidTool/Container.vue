<script setup lang="ts">
import Button from "primevue/button";
import Dropdown from "primevue/dropdown";
import InputSwitch from "primevue/inputswitch";
import InputText from "primevue/inputtext";
import { ref } from "vue";

import { useSDK } from "@/plugins/sdk";

const sdk = useSDK();

type Variant = "v4" | "zero";

const variant = ref<Variant>("v4");
const hyphens = ref(true);
const uppercase = ref(false);
const result = ref("");

const options = [
  { label: "UUID v4", value: "v4" },
  { label: "All zeros", value: "zero" },
];

const generate = () => {
  if (variant.value === "zero") {
    result.value = formatUuid("00000000000000000000000000000000");
    return;
  }
  const id = crypto.randomUUID();
  result.value = uppercase.value
    ? (hyphens.value ? id : id.replaceAll("-", "")).toUpperCase()
    : hyphens.value
      ? id
      : id.replaceAll("-", "");
};

const formatUuid = (hex: string) => {
  const withHyphens = `${hex.slice(0, 8)}-${hex.slice(8, 12)}-${hex.slice(12, 16)}-${hex.slice(16, 20)}-${hex.slice(20)}`;
  const str = hyphens.value ? withHyphens : hex;
  return uppercase.value ? str.toUpperCase() : str;
};

const onCopy = async () => {
  await navigator.clipboard.writeText(result.value);
  sdk.window.showToast("Copied", { variant: "success" });
};

const onClear = () => {
  result.value = "";
};
</script>

<template>
  <div class="h-full flex flex-col gap-3">
    <div class="flex gap-2 items-center">
      <Dropdown
        v-model="variant"
        :options="options"
        option-label="label"
        option-value="value"
        class="w-44"
      />
      <div class="flex items-center gap-2">
        <InputSwitch v-model="hyphens" />
        <span class="text-sm">Hyphens</span>
      </div>
      <div class="flex items-center gap-2">
        <InputSwitch v-model="uppercase" />
        <span class="text-sm">Uppercase</span>
      </div>
      <Button label="Generate" icon="fas fa-bolt" @click="generate" />
      <div class="flex-1" />
      <Button label="Copy" icon="fas fa-copy" @click="onCopy" />
      <Button label="Clear" icon="fas fa-eraser" @click="onClear" />
    </div>
    <InputText v-model="result" readonly />
  </div>
</template>
