<template>
  <v-container class="d-flex container">
    <v-row class="justify-center align-center">
      <v-col cols="10">
        <v-row class="justify-center">
          <v-btn class="button" id="moving-button" @click="updateInc"><h1>+</h1></v-btn>
        </v-row>
        <v-row class="justify-center">
          <h1 class="title">{{ incValue }} clicks</h1>
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
    randomMoves: null,
    currentTranslateX: 0,
    currentTranslateY: 0,
  }),
  methods: {

    random(maxIntervalValue) {
      let num = Math.floor(Math.random() * maxIntervalValue);
      num *= Math.round(Math.random()) ? 1 : -1;
      return num;
    },

    randomNoFloor(maxIntervalValue) {
      let num = Math.random() * maxIntervalValue;
      num *= Math.round(Math.random()) ? 1 : -1;
      return num;
    },

    randomBool() {
      return Math.round(Math.random());
    },

    randomInterval(min, max) {
      return Math.floor(Math.random() * (max - min) + min)
    },

    checkBoundaries() {
      this.currentTranslateX > 100 && (this.currentTranslateX = 95);
      this.currentTranslateY > 100 && (this.currentTranslateY = 95);
    },

    initializeButton() {
      this.randomMoves = setInterval(() => {
        if (this.randomBool()) {
          let doc = document.getElementById("moving-button");
          this.currentTranslateX += this.randomNoFloor(2);
          this.currentTranslateY += this.randomNoFloor(2);
          this.checkBoundaries();
          doc.style.transition = "ease-in transform 0.1s";
          doc.style.transform = `translateX(${this.currentTranslateX}vw) translateY(${this.currentTranslateY}vh)`;
        }
      }, 300);
    },

    moveButton() {
      let bool = this.randomBool();
      let doc = document.getElementById("moving-button");
      if (bool) {
        doc.style.transition = "ease-in opacity";
        doc.style.opacity = "0"
      }
      setTimeout(() => {
        doc.style.transition = "ease-in transform";
        this.currentTranslateX = this.random(40);
        this.currentTranslateY = this.random(35);
        doc.style.transform = `translateX(${this.currentTranslateX}vw) translateY(${this.currentTranslateY}vh)`;
        if (bool) {
          doc.style.transition = "ease-in opacity";
          doc.style.opacity = "1";
        }
      }, 1)
    },

    async getSnapshotInc(){
      const ref = await this.$fire.firestore.collection("increment").doc("incrementdoc")
      await ref.onSnapshot((querySnap) => querySnap.data() && (this.incValue = querySnap.data().inc));
    },

    async updateInc() {
      this.incValue += 1;
      await this.$fire.firestore.collection("increment").doc("incrementdoc").set({"inc": this.incValue})
      await this.getSnapshotInc();
      this.moveButton();
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
    this.$fire.auth.onAuthStateChanged(async () => {
      this.$fire.auth.currentUser && (this.connected = true);
      await this.getSnapshotInc();
      this.initializeButton();
    });
  },

  destroyed() {
    clearInterval(this.randomMoves);
  }

}
</script>

<style scoped>
.title {
  font-size: 6em;
}
.button {
  height: 5em;
  width: 5em;
}
.container {
  min-height: 100%;
}
</style>
