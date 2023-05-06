<script setup>
import { data } from "autoprefixer";
import { ref, watchEffect, computed } from "vue";
import { cloneDeepWith, isArray } from "lodash";

const rawData = ref(null);
const dataList = ref([]);
const dataStr = ref("");

watchEffect(() => {
  if (rawData.value) {
    const reader = new FileReader();

    reader.addEventListener(
      "load",
      () => {
        dataStr.value = reader.result;
        dataList.value = JSON.parse(dataStr.value);
      },
      false
    );
    reader.readAsText(rawData.value);
  }
});

const flatten = (data, prefix = "note") => {
  if (!data || Object.keys(data).length <= 0) return;

  prefix = prefix + ".";
  // data = cloneDeepWith(data, cloneCustomizer)

  const keysList = [];

  const flatObj = {};
  const [key, value] = Object.entries(data)[0];
  let queue = [{ key, value }];
  let path = [];
  while (queue.length > 0) {
    let { key, value } = queue.shift();
    const parentKey = key.startsWith(prefix) ? key : `${prefix}${key}`;
    const entries = Object.entries(value).map((entry) => {
      let [key, value] = entry;
      key = `${parentKey}.${key}`;
      return { key, value };
    });
    for (const entry of entries) {
      const { key, value } = entry;
      if (value !== null && typeof value === "object" && !isArray(value)) {
        queue.push(entry);
      } else {
        if (!flatObj[key]) {
          flatObj[key] = value;
          keysList.push(key);
        } else {
          // this should never happen
          console.log(`duplicate key found: ${entry}`);
        }
      }
    }
  }
  return { flatObj, keysList };
};

const flatObj = computed(() => {
  if (!dataList.value[0]) return;
  return flatten(dataList.value[0]);
});
</script>

<template>
  <q-page class="q-pa-lg">
    <q-file
      bg-color="info"
      square
      filled
      v-model="rawData"
      label="Upload Data File"
    />
    <!-- <pre>{{ flatObj?.flatObj }}</pre> -->
    <div>
      <p v-for="key in flatObj?.keysList" :key="key">
        {{ key }}
      </p>
    </div>
  </q-page>
</template>
