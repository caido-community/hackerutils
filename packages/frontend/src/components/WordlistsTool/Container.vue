<script setup lang="ts">
import Button from "primevue/button";
import Dropdown from "primevue/dropdown";
import Textarea from "primevue/textarea";
import { ref } from "vue";

import { useSDK } from "@/plugins/sdk";

const sdk = useSDK();

type ListId =
  | "url-encoded"
  | "unicode-encoded"
  | "cmd-exec"
  | "cmd-exec-unix"
  | "nosql"
  | "mongo"
  | "ssti";

const mode = ref<ListId>("url-encoded");
const result = ref<string>("");
const loading = ref<boolean>(false);

const options = [
  { label: "All characters URL-encoded (%00-%FF)", value: "url-encoded" },
  {
    label: "All characters Unicode-escaped (\\u0000-\\u00FF)",
    value: "unicode-encoded",
  },
  { label: "Command Injection: command_exec.txt", value: "cmd-exec" },
  {
    label: "Command Injection: command-execution-unix.txt",
    value: "cmd-exec-unix",
  },
  { label: "NoSQL Injection: NoSQL.txt", value: "nosql" },
  { label: "NoSQL Injection: MongoDB.txt", value: "mongo" },
  { label: "SSTI: ssti.fuzz", value: "ssti" },
];

const urls: Record<
  Exclude<ListId, "url-encoded" | "unicode-encoded">,
  string
> = {
  "cmd-exec":
    "https://raw.githubusercontent.com/swisskyrepo/PayloadsAllTheThings/refs/heads/master/Command%20Injection/Intruder/command_exec.txt",
  "cmd-exec-unix":
    "https://raw.githubusercontent.com/swisskyrepo/PayloadsAllTheThings/refs/heads/master/Command%20Injection/Intruder/command-execution-unix.txt",
  nosql:
    "https://raw.githubusercontent.com/swisskyrepo/PayloadsAllTheThings/refs/heads/master/NoSQL%20Injection/Intruder/NoSQL.txt",
  mongo:
    "https://raw.githubusercontent.com/swisskyrepo/PayloadsAllTheThings/refs/heads/master/NoSQL%20Injection/Intruder/MongoDB.txt",
  ssti: "https://raw.githubusercontent.com/swisskyrepo/PayloadsAllTheThings/refs/heads/master/Server%20Side%20Template%20Injection/Intruder/ssti.fuzz",
};

const generate = async () => {
  if (mode.value === "url-encoded") {
    const list = Array.from(
      { length: 256 },
      (_v, i) => `%${i.toString(16).toUpperCase().padStart(2, "0")}`,
    );
    result.value = list.join("\n");
    return;
  }

  if (mode.value === "unicode-encoded") {
    const list = Array.from(
      { length: 256 },
      (_v, i) => `\\u${i.toString(16).toUpperCase().padStart(4, "0")}`,
    );
    result.value = list.join("\n");
    return;
  }

  const url = urls[mode.value];
  if (url === undefined) {
    sdk.window.showToast("Unknown source", { variant: "error" });
    return;
  }

  loading.value = true;
  const res = await fetch(url).catch(() => undefined as Response | undefined);
  if (res === undefined) {
    sdk.window.showToast("Network error while fetching", { variant: "error" });
    loading.value = false;
    return;
  }
  if (!res.ok) {
    sdk.window.showToast(`Failed to fetch (${res.status})`, {
      variant: "error",
    });
    loading.value = false;
    return;
  }
  const text = await res.text();
  result.value = text;
  loading.value = false;
};

const onCopy = async () => {
  await navigator.clipboard.writeText(result.value);
  sdk.window.showToast("Copied", { variant: "success" });
};

const onDownload = () => {
  const content = encodeURIComponent(result.value);
  const href = `data:text/plain;charset=utf-8,${content}`;
  const a = document.createElement("a");
  a.href = href;
  a.download = `${mode.value}.txt`;
  document.body.appendChild(a);
  a.click();
  document.body.removeChild(a);
};

const onAddToFiles = async () => {
  const content = result.value;
  if (content === undefined || content === "") {
    sdk.window.showToast("No content to add", { variant: "error" });
    return;
  }

  const blob = new Blob([content], { type: "text/plain" });
  const file = new File([blob], `${mode.value}.txt`, { type: "text/plain" });

  try {
    await sdk.files.create(file);
    sdk.window.showToast("Added to files", { variant: "success" });
  } catch (_e) {
    sdk.window.showToast("Failed to add to files", { variant: "error" });
  }
};

const onClear = () => {
  result.value = "";
};
</script>

<template>
  <div class="h-full flex flex-col gap-3">
    <div class="flex items-center gap-2">
      <Dropdown
        v-model="mode"
        :options="options"
        option-label="label"
        option-value="value"
        class="w-56"
      />
      <Button
        :label="loading ? 'Loading...' : 'Load'"
        :disabled="loading"
        icon="fas fa-bolt"
        @click="generate"
      />
      <div class="flex-1" />
      <Button label="Copy" icon="fas fa-copy" @click="onCopy" />
      <Button label="Download" icon="fas fa-download" @click="onDownload" />
      <Button label="Add to files" icon="fas fa-file" @click="onAddToFiles" />
      <Button label="Clear" icon="fas fa-eraser" @click="onClear" />
    </div>
    <Textarea v-model="result" class="h-full resize-none" rows="50" />
  </div>
</template>
