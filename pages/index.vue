<template>
  <v-container class="d-flex" style="min-height: 100%">
    <v-row class="justify-center align-center">
      <v-col cols="10">
        <v-row class="justify-center">
          <v-btn id="moving-button" @click="updateInc">Increment</v-btn>
        </v-row>
        <v-row class="justify-center">
          <h1 style="font-size: 6em">{{ incValue }} clicks</h1>
        </v-row>
      </v-col>
    </v-row>
    <v-btn :disabled="loading" @click="connexion">{{connected ? "Deconnexion" : "Connexion"}}</v-btn>
  </v-container>
</template>

<script>
export default {
  name: 'index',
  data: () => ({
    incValue: 0,
    connected: false,
    loading: false,
  }),
  methods: {

    random() {
      let num = Math.floor(Math.random() * 40);
      num *= Math.round(Math.random()) ? 1 : -1;
      return num;
    },

    async getSnapshotInc(){
      const ref = await this.$fire.firestore.collection("increment").doc("incrementdoc")
      await ref.onSnapshot((querySnap) => querySnap.data() && (this.incValue = querySnap.data().inc));
    },

    async updateInc() {
      this.incValue += 1;
      await this.$fire.firestore.collection("increment").doc("incrementdoc").set({"inc": this.incValue})

      let doc = document.getElementById("moving-button");
      doc.style.transform = `translateX(${this.random()}vw) translateY(${this.random()}vh)`;
    },

    async connexion(){
      if(this.connected){
        this.loading = true;
        this.$fireModule.auth().signOut().then((result) => {
          this.connected = false;
          this.loading = false;
        })
      }else{
        this.loading = true;
        this.$fireModule.auth().signInAnonymously().then((result) => {
          this.connected = true;
          this.loading = false;
        });
      }
    }
  },

  async mounted() {
    await this.getSnapshotInc();
  }

}
</script>
