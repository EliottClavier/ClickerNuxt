
<template>
  <div :style="{left:`${x}%`,top:`${y}%`,width:`${w}rem`,height:`${x}rem`,position:'absolute'}" >
    <iframe src="https://streamelements.com/overlay/61d6bbe307ccdea20b5dbc08/OAGnBhFKPj-wpfHBcvs2112URuyyIsR0Wx9Mq9qbzYMuA4dd" width="100%" height="100%" style="position:absolute" frameBorder="0" allowFullScreen></iframe>
  </div>
</template>
<script>
  export default{
    data: () => ({
      x: 0,
      y: 0,
      w: 0,
      h: 0,
    }),
    methods: {
      randomInterval(min, max) {
        return Math.floor(Math.random() * (max - min) + min)
      },
    },
    mounted() {
      this.x = this.randomInterval(0, 90);
      this.y = this.randomInterval(0, 90);
      this.w = this.randomInterval(20, 100);
      this.h = this.w;
    }
  }
</script>
