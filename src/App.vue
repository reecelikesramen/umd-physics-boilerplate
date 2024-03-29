<template>
  <div id="container">

    <Content
      v-model:object="object"
      v-model:objX="objX"
      v-model:objY="objY"
      :stageWidth="stageWidth"
      @reset="reset" />

    <div id="code-section">

      <Control
        v-model:runTime="runTime"
        :runTimeLeft="runTimeLeft"
        @runCode="runCode"
        @clearCode="clearCode"
        @reset="reset"
        @focus="focusEditor" />

      <Code
        v-model:code="code"
        @focus="focusEditor" />

    </div>
  </div>
</template>

<script>
import Content from './components/Content.vue'
import Control from './components/Control.vue'
import Code from './components/Code.vue'

export default {
  name: 'App',
  data() {
    return {
      code: '',
      object: {},
      startX: 3,
      startY: 1,
      objX: 0,
      objY: 0,
      stageWidth: 50, // in meters
      runTime: 0,
      runTimeLeft: 0,
      deltaTime: 1/30,
      interval: null,
      timeout: null,
    }
  },
  components: {
    Content,
    Control,
    Code
  },
  methods: {
    runCode() {

      clearTimeout(this.timeout);
      clearInterval(this.interval);

      const userCode = new Function(this.code +
      `
      const functions = {};
      try {
        functions.integrate = integrate;
      } catch(e) {
        functions.integrate = () => {};
      }

      try {
        functions.setup = setup;
      } catch(e) {
        functions.setup = () => {};
      }

      return functions;
      `);

      const functions = userCode();

      functions.setup(this.object);
      this.interval = setInterval((start) => {
        functions.integrate(this.object, this.deltaTime);
        this.runTimeLeft = Math.abs(this.runTime - (Date.now() - start) / 1000).toFixed(1);
      }, 1000 * this.deltaTime, Date.now());

      this.timeout = setTimeout(() => {
        clearInterval(this.interval);
      }, this.runTime * 1000);
    },

    clearCode() {
      clearInterval(this.interval);
      this.code = '';
      // console.log('clear code!');
    },

    reset() {
      this.object.reset();
      this.objX = this.startX;
      this.objY = this.startY;
      // console.log('resetting!')
    },

    focusEditor() {
      document.getElementById('code-editor').children[0].children[0].focus();
      // console.log('focusing!')
    }
  },
  mounted() {
    this.runTime = 3.5;
    this.code =
    `/** This function is used to set the initial conditions of the object. While I was brainstorming the project I imagined that you might make projects where the coder can't control the position directly, only through change in position. That's how I programmed this. All you have access to are the starting velocities and accelerations. */
function setup(object) {
  object.velX = 10;
  object.velY = 15;
  object.accX = 0;
  object.accY = -9.8;
}

/** Everything in here is run (effectively) continuously. Use some of the examples I provided below to fiddle around with how the object integrates. This is standard semi-implicit Euler integration, commonly used for low-impact physics engines. */
function integrate(object, deltaTime) {
  object.addVelX(object.accX * deltaTime);
  object.addVelY(object.accY * deltaTime);
  
  object.addX(object.velX * deltaTime);
  object.addY(object.velY * deltaTime);
}`
  }
}
</script>

<style>
html, body {
  padding: 0;
  margin: 0;
}

#app {
  background-color: #3B353B;
  height: 100vh;
}

#container {
  padding: 0;
  padding-top: 3vh;
  padding-bottom: 2vh;
  height: 95vh;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: space-between;
}

#code-section {
  display: flex;
  flex-direction: column;
  align-items: center;
}
</style>
