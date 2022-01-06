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
          <v-btn id="moving-button" @click="updateInc" rounded><h1>{{ objValue - incValue }}</h1></v-btn>
        </v-row>
        <v-row class="justify-center">
          <h1 class="custom-title">{{ incValue }} clicks</h1>
        </v-row>
      </v-col>
      <div id="leaderboard" class="d-flex align-center custom-text">
        <ul class="m-0 pl-5 pr-5">
          <li class="mb-3">
            <span v-if="connected">Votre score : {{ score }}</span>
            <span v-else> Connexion requise pour scorer !</span>
          </li>
          <li class="d-flex align-center" v-for="(item, idx) in leaderboard" :key="item.pseudo + item.score" :style="{color: `${ id === item.id && ('#ffc107') }`}">
            <img v-if="idx === 0" class="mr-2" style="height: 1em" src="https://img.icons8.com/color-glass/48/000000/crown.png"/>
            {{ item.pseudo }} - {{ item.score }}
          </li>
        </ul>
      </div>
      <div id="connexion-section" class="d-flex align-center">
        <v-text-field v-if="!connected" outlined placeholder="Pseudonyme" v-model="pseudo" class="mr-2"
            :append-icon="pseudo ? 'mdi-play' : undefined" v-on:keyup.enter="connexion" @click:append="connexion" maxlength="12">
        </v-text-field>
        <v-btn v-else :disabled="loading" @click="connexion" outlined>Deconnexion</v-btn>
      </div>
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
    id: "",
    pseudo: "",
    score: 0,
    leaderboard: [],
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
      this.currentTranslateX < -100 && (this.currentTranslateX = 5);
      this.currentTranslateX > 100 && (this.currentTranslateX = 95);
      this.currentTranslateY < -100 && (this.currentTranslateY = 5);
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
        await this.getValue();
      });
    },

    async getUserScore() {
      const ref = await this.$fire.firestore.collection("users").doc(localStorage.getItem("userID"));
      if (ref) {
        const snapshot = await ref.get();
        const doc = snapshot.data();
        this.score = doc.score;
      }
    },

    async getLeaderboard() {
      const ref = await this.$fire.firestore.collection("users").orderBy("score", "desc").limit(5);
      await ref.onSnapshot(async (querySnap) => {
        let tempArray = [];
        await querySnap.forEach((doc) => {
          let tempDoc = doc.data();
          tempDoc.id = doc.id;
          tempArray.push(tempDoc);
        });
        this.leaderboard = tempArray;
        await this.getValue();
      });
    },

    async updateInc() {
      // Pas de controle avec this.connected pour laisser les rules Firebase fonctionner
      this.incValue += 1;
      if (this.connected) {
        this.score += 1;
      }
      let doc = {
        inc: this.incValue,
      };
      if(this.incValue >= this.objValue){
        let rand = Math.round(Math.random()*(200-10)+10);
        doc.obj = this.objValue+rand;
        doc.last = this.objValue;
      }
      this.goldeningButton();
      console.log("call")
      await this.$fire.firestore.collection("increment").doc("incrementdoc").update(doc).then(async () => {
        this.connected && (await this.$fire.firestore.collection("users").doc(localStorage.getItem("userID")).update({"score": this.score}));
        await this.getValue();
        await this.getSnapshotInc();
        this.moveButton();
      })
    },

    async getValue(){
      this.value = 100-Math.round(((this.objValue-this.incValue)/(this.objValue-this.lastValue))*100);
    },

    goldeningButton() {
      let doc = document.getElementById("moving-button");
      doc.style.background = this.objValue - 1 === this.incValue ? '#ffc107' : '';
    },

    async connexion(){
      if(this.connected){
        this.loading = true;
        this.$fireModule.auth().signOut().then(() => {
          localStorage.setItem("userID", "");
          localStorage.setItem("userPseudo", "");
          this.id = "";
          this.score = 0;
          this.connected = false;
          this.loading = false;
        })
      } else if (this.pseudo) {
        this.loading = true;
        this.$fireModule.auth().signInAnonymously().then(async () => {
          let ref = this.$fire.firestore.collection("users").doc();
          localStorage.setItem('userID', ref.id);
          localStorage.setItem('userPseudo', this.pseudo);
          await ref.set({
            pseudo: this.pseudo,
            score: 0
          });
          this.id = ref.id;
          this.connected = true;
          this.loading = false;
        });
      }
    }
  },

  async mounted() {
    await this.getLeaderboard();
    this.initializeButton();
    this.$fire.auth.onAuthStateChanged(async () => {
      if (this.$fire.auth.currentUser) {
        this.id = localStorage.getItem("userID");
        this.pseudo = localStorage.getItem("userPseudo")
        console.log(localStorage)
        await this.getUserScore().then(() => this.connected = true);
      }
      this.loadData = true;
      await this.getSnapshotInc().then(() => {
        this.getValue();
        this.loadData = false;
      });
    });
  },

  destroyed() {
    this.randomMoves && (clearInterval(this.randomMoves));
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
.custom-text {
  font-size: 1.2em;
  font-weight: bold;
}
#moving-button {
  height: 5em;
  width: 5em;
  z-index: 10;
}
#leaderboard {
  position: absolute;
  top: 1em;
  left: 1.5em;
  flex-direction: column;
}
#connexion-section {
  position: absolute;
  top: 1em;
  right: 1.5em;
}
.custom-container {
  min-height: 100%;
}
ul {
  list-style-type: none;
}
</style>
