<template>
  <v-container class="d-flex custom-container">
    <v-row v-if="loadData" class="d-flex justify-center align-center">
      <v-progress-circular class="justify-center align-center"
        indeterminate
        color="#ede3e8"
        :size="200"
        :width="10"
      ></v-progress-circular>
    </v-row>
    <v-row v-else class="justify-center align-center">
      <v-progress-linear
        absolute
        bottom
        left
        color="black"
        height="70"
        :value="value"
      >
        <div class="custom-subtitle">{{incValue}} / {{objValue}}</div>
      </v-progress-linear>
      <v-col cols="12">
        <v-row class="justify-center">
          <v-btn id="moving-button" @click="updateInc"><h1>+</h1></v-btn>
        </v-row>
        <v-row class="justify-center">
          <h1 class="custom-title">{{ incValue }} clicks</h1>
        </v-row>
      </v-col>
      <v-btn :disabled="loading" @click="connexion" style="position: absolute; top: 1em; right: 1.5em">{{connected ? "Deconnexion" : "Connexion"}}</v-btn>
    </v-row>
  </v-container>
</template>

<script>
export default {
  name: 'index',
  data: () => ({
    incValue: 0,
    objValue: 0,
    lastValue: 0,
    value: 0,
    loadData: false,
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
        doc.style.transition = "ease-in transform 0.1s";
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
      const ref = await this.$fire.firestore.collection("increment");
      await ref.onSnapshot(async (querySnap) => {
        await querySnap.forEach((doc) => {
          let data = doc.data();
          this.incValue = data.inc;
          this.objValue = data.obj;
          this.lastValue = data.last;
        });
        this.getValue();
      });
    },

    async updateInc() {
      this.incValue += 1;
      let doc = {
        inc: this.incValue,
      };
      if(this.incValue >= this.objValue){
        let rand = Math.round(Math.random()*(200-10)+10);
        doc.obj = this.objValue+rand;
        doc.last = this.objValue;
      }
      await this.$fire.firestore.collection("increment").doc("incrementdoc").update(doc).then(async () => {
        await this.getValue();
        await this.getSnapshotInc();
        this.moveButton();
      })
    },

    async getValue(){
      let val = 100-Math.round(((this.objValue-this.incValue)/(this.objValue-this.lastValue))*100);
      this.value = val;
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
      this.loadData = true;
      await this.getSnapshotInc().then(() => {
        this.getValue();
        this.initializeButton();
        this.loadData = false;
      });
    });
  },

  destroyed() {
    clearInterval(this.randomMoves);
  }

}
</script>

<style type="text/css" scoped>
.custom-title {
  font-size: 6em;
}
.custom-subtitle {
  font-size: 2.5em;
  font-weight: bold;
}
#moving-button {
  height: 5em;
  width: 5em;
  z-index: 10;
}
.custom-container {
  min-height: 100%;
}
</style>
