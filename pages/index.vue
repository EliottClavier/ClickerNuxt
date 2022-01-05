<template>
  <v-container class="d-flex" style="min-height: 100%">
    <v-row class="justify-center align-center">
      <v-col cols="10">
        <v-row class="justify-center">
          <v-btn @click="updateInc">Increment</v-btn>
        </v-row>
        <v-row class="justify-center">
          <h1 style="font-size: 6em">{{ incValue }} clicks</h1>
        </v-row>
      </v-col>
    </v-row>
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
