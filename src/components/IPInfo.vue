<template>
  <div
    class="
      absolute
      -bottom-64
      lg:-bottom-20
      flex flex-wrap
      bg-white
      gap-y-4 gap-x-12
      px-8
      pt-8
      mx-20px
      pb-12
      rounded-md
    "
  >
    <div
      class="flex flex-col"
      v-for="[key, value] in Object.entries(Info ? Info : ipData)"
      :key="key"
    >
      <h3 class="text-xs mb-2 uppercase">{{ key }}</h3>
      <span class="text-2xl">{{ value }}</span>
    </div>
    <div class="close absolute top-1 right-1">
      <font-awesome-icon
        @click="$emit('close')"
        class="
          text-2xl text-red-500
          bg-black
          px-4
          py-2
          rounded-md
          hover:bg-gray-500 hover:transition-all
          duration-700
          ease-in-out
          hover:cursor-pointer
        "
        :icon="['fa-solid', 'fa-times']"
        width="20px"
      />
    </div>
  </div>
</template>

<script>
import { ref } from "vue";
export default {
  props: {
    Info: {
      type: Object,
      required: true,
    },
  },
  setup() {
    const ipData = ref({
      "IP Address": "8.8.8.8",
      Location: "United States",
      Timezone: "UTC -07:00",
      ISP: "Google Inc.",
    });
    
    return { ipData };
  },
};
</script>

<style>
</style>