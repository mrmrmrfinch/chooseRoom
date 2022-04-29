<template>
  <v-main style="height: 100vh">
    <v-container style="height: 100%">
            <div class='ripple-background' style="position:fixed;top:50%; left:50%">
        <div class='circle xxlarge shade1'></div>
        <div class='circle xlarge shade2'></div>
        <div class='circle large shade3'></div>
        <div class='circle mediun shade4'></div>
        <div class='circle small shade5'></div>
      </div>
      <div class="center">
        <h1
          class="introTitle"
          style="
            font-weight: 600;
            color: black;
            text-shadow: 2px 2px 28px rgba(255, 255, 255, 0.5);
          "
        >
          An Acoustically-Accurate<br />Room Selector Tool.
        </h1>
        <p style="color: black; padding-bottom: 20px;padding-top:20px;line-height:20px">
          A Project from Acoustics Portfolio (AME 292) at University of Rochester.
          <br />
          Visit our <a href="http://acousticsportfolio.digitalscholar.rochester.edu/ch/">wordpress website</a> for an in-depth look at our process.
        </p>
        <v-btn elevation="2" class="btnMedia" x-large onclick="window.location='/questions'">
          Get Started</v-btn
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

/* on small screens, make the text smaller */
@media only screen and (max-width: 600px) {
  .center {
    text-align: left;
    margin: 0;
    padding:30px;
    position: absolute;
    width: 100%;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
  }
  .introTitle {
    font-size: 2em;
  }

  .btnMedia {
    width:200px;
    height:30px;
    margin-bottom: 20px;
  }
}

/* on large screens, text normal */
@media only screen and (min-width: 600px) {
  .center {
    margin: 0;
    position: absolute;
    width: 100%;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    text-align: center;
  }
    .introTitle {
    font-size: 5em;
  }
}


.circle {
  position: absolute;
  border-radius: 50%;
  background: white;
  animation: ripple 15s infinite;
  box-shadow: 0px 0px 1px 0px #508fb9;
}

.small {
  width: 400px;
  height: 400px;
  left: -200px;
  bottom: -200px;
}

.medium {
  width: 600px;
  height: 600px;
  left: -300px;
  bottom: -300px;
}

.large {
  width: 800px;
  height: 800px;
  left: -400px;
  bottom: -400px;
}

.xlarge {
  width: 1000px;
  height: 1000px;
  left: -500px;
  bottom: -500px;
}

.xxlarge {
  width: 1200px;
  height: 1200px;
  left: -600px;
  bottom: -600px;
}

.shade1 {
  opacity: 0.05;
  filter: blur(2px);
}
.shade2 {
  opacity: 0.1;
  filter: blur(4px);
}

.shade3 {
  opacity: 0.1;
  filter: blur(5px);
}

.shade4 {
  opacity: 0.1;
  filter: blur(10px);
}

.shade5 {
  opacity: 0.1;
  filter: blur(20px);
}

@keyframes ripple {
  0% {
    transform: scale(0.8);
  }

  50% {
    transform: scale(1.2);
  }

  100% {
    transform: scale(0.8);
  }
}
</style>
