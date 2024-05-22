<template>
  <div>
    <b-modal
      ref="modal-appointments-grid"
      size="lg"
      @hidden="$emit('hidden')"
      no-close-on-backdrop
    >
      <template #modal-title>
        <span class="text-uppercase font-weight-bolder"> Title </span>
      </template>
      <template #default> </template>
      <template #modal-footer>
        <b-row class="d-flex justify-content-end">
          <b-button variant="primary"> SAVE </b-button>
        </b-row>
      </template>
    </b-modal>
  </div>
</template>
<script>
export default {
  name: "ModalAppointmentsGrid",
  data() {
    return {};
  },
  computed: {},
  created() {},
  mounted() {
    this.toggleModal("modal-appointments-grid");
  },
  methods: {},
};
</script>
<style lang="scss">
</style>
