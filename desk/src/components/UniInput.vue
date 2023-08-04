<template>
  <div class="space-y-2">
    <div class="text-xs text-gray-600">
      {{ field.label }}
      <span v-if="field.required" class="text-red-500">*</span>
    </div>
    <component
      :is="component"
      :placeholder="placeholder"
      :value="value"
      @change="emitUpdate(field.fieldname, $event.value || $event)"
    />
  </div>
</template>

<script setup lang="ts">
import { computed, h } from "vue";
import { createResource, Autocomplete, Input } from "frappe-ui";
import { Field } from "@/types";
import SearchComplete from "./SearchComplete.vue";

type Value = string | number | boolean;

interface P {
  field: Field;
  value: Value;
}

interface R {
  fieldname: Field["fieldname"];
  value: Value;
}

interface E {
  (event: "change", value: R);
}

const props = defineProps<P>();
const emit = defineEmits<E>();

const component = computed(() => {
  if (props.field.url_method) {
    return h(Autocomplete, {
      options: apiOptions.data,
    });
  } else if (props.field.fieldtype === "Link" && props.field.options) {
    return h(SearchComplete, {
      doctype: props.field.options,
    });
  } else if (props.field.fieldtype === "Select") {
    return h(Autocomplete, {
      options: props.field.options
        .split("\n")
        .map((o) => ({ label: o, value: o })),
    });
  } else {
    return Input;
  }
});

const apiOptions = createResource({
  url: props.field.url_method,
  auto: !!props.field.url_method,
  transform: (data) =>
    data.map((o) => ({
      label: o,
      value: o,
    })),
});

const placeholder = computed(() => {
  if (props.field.fieldtype === "Data") {
    return "Type something";
  }
  return "Select an option";
});

function emitUpdate(fieldname: Field["fieldname"], value: Value) {
  emit("change", { fieldname, value });
}
</script>