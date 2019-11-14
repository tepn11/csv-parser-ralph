<template>
  <q-page class="flex flex-center">
    <main-table />
  </q-page>
</template>

<script>
import MainTable from '../components/MainTable.vue';

export default {
  name: 'PageIndex',
  components: { MainTable },
};
</script>
