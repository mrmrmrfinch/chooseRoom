<template>
  <v-main style="height: 100vh">
    <v-container style="height: 100%">
      <div class="center">
        <h1
          style="
            font-size: 5rem;
            font-weight: 600;
            color: white;
            text-shadow: 2px 2px 14px rgba(0, 0, 0, 0.2);
          "
        >
          An Acoustically-Accurate<br />Room Selector Tool.
        </h1>
        <p style="color: white; padding-bottom: 20px">
          A Project from University of Rochester AME292, 2022 Spring.
        </p>
        <v-btn elevation="2" x-large onclick="window.location='/questions'">
          Questions Demo</v-btn
        >
        <v-btn elevation="2" x-large onclick="window.location='/reverb'">
          Reverb Demo</v-btn
        >
      </div>
    </v-container>
  </v-main>
</template>

<script>
export default {
  name: "indexPage",
};
</script>

<style scope>
@font-face {
  font-family: "Manrope";
  src: url("/manrope.woff2") format("woff2");
}

body {
  background: linear-gradient(
    -55deg,
    #45858c,
    #a3d9d9,
    #d9c589,
    #bf9765,
    #d92b04
  );
  background-size: 400% 400%;
  animation: gradient 15s ease infinite;
  height: 100vh;
  font-family: "Manrope";
}

@keyframes gradient {
  0% {
    background-position: 0% 50%;
  }
  50% {
    background-position: 100% 50%;
  }
  100% {
    background-position: 0% 50%;
  }
}

.center {
  margin: 0;
  position: absolute;
  width: 100%;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  text-align: center;
}
</style>
