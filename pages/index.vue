<template>
  <v-container>
    <v-btn @click="updateInc">Increment</v-btn>
  </v-container>
</template>

<script>
export default {
  name: 'index',
  data: () => ({
    incValue: 0
  }),
  methods: {
    async getSnapshotInc(){
      const ref = await this.$fire.firestore.collection("increment").doc("incrementdoc")
      await ref.onSnapshot((querySnap) => querySnap.data() && (this.incValue = querySnap.data().inc));
    },

    async updateInc() {
      this.incValue += 1;
      await this.$fire.firestore.collection("increment").doc("incrementdoc").set({"inc": this.incValue})
    },
  },

  async mounted() {
    await this.getSnapshotInc();
  }

}
</script>
