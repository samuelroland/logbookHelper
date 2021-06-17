

<template>
  <Popup :version="version" :versionDate="versionDate" />
</template>

<script>
import Popup from "@/components/Popup.vue";

export default {
  name: "App",
  data() {
    return {
      version: "v1.0",
      versionDate: "02.04.2021"
    };
  },
  components: { Popup }
};
</script>
