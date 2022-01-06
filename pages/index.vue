<template>
  <v-container v-else class="d-flex" style="min-height: 100%">
    <div v-if="loadData"
         class="d-flex justify-center" style="margin-top: 40vh; transform: translateY(-50%);" >
      <v-progress-circular
        indeterminate
        color="#ede3e8"
        :size="200"
        :width="10"
      ></v-progress-circular>
    </div>
    <v-row v-else class="justify-center align-center">
      <v-col cols="4">
        <v-row class="justify-center">
          <v-progress-linear
            color="amber"
            height="150"
            v-model="value"
            style="transform: rotate(-90deg)"
          >
            <div style="transform: rotate(90deg)">{{incValue}} / {{objValue}}</div>
          </v-progress-linear>
        </v-row>
      </v-col>
      <v-col cols="8">
        <v-row class="justify-center">
          <v-btn @click="updateInc">Increment</v-btn>
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
    objValue: 0,
    lastValue: 0,
    value: 0,
    loadData: false,
    connected: false,
    loading: false,
  }),
  methods: {
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
      await this.$fire.firestore.collection("increment").doc("incrementdoc").update(doc).then(() => {
        this.getValue();
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
    this.loadData = true;
    await this.getSnapshotInc().then(() => {
      this.getValue();
    });
    this.loadData = false;
  }

}
</script>
