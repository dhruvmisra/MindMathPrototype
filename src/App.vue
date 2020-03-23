<template>
  <div id="app">
    <Toolbar :showHelp="showHelp" :status="status" @playClicked="start" @restartClicked="restart" />
    <div class="main-container">
      <section class="selection w-50">
        <!-- <h5 class="mt-5">Numbers: {{ selectedNumbers }}</h5> -->
        <div class="numbers">
          <button
            class="btn-number"
            :class="{ selected: selectedNumbers.indexOf(number) != -1 }"
            @click="status == 'running' && selectedNumbers.indexOf(number) != -1 ? unselectNumber(number) : selectNumber(number)"
            v-for="number in numbers"
            :key="number"
            :disabled="status != 'running'"
          >{{ status == 'not-started' ? '' : number }}</button>
        </div>

        <div class="target-wrap text-center">
          <h3 class="target">{{ status == 'not-started' ? '' : target }}</h3>
        </div>

        <!-- <h5 class="mt-5">Operator: {{ selectedOp }}</h5> -->
        <div class="operators">
          <button
            class="btn-operator"
            :class="{ selected: selectedOp.sign == operator.sign }"
            :style="{ background: operator.color }"
            @click="status == 'running' && selectedOp.sign == operator.sign ? unselectOp(operator) : selectOp(operator)"
            v-for="operator in operators"
            :key="operator.sign"
            :disabled="status != 'running'"
          >{{ status == 'not-started' ? '' : operator.sign }}</button>
        </div>
        <!-- <p class="mt-3">Operators Used: {{ operatorsUsed }}</p> -->

        <!-- <button class="btn btn-info d-block mx-auto my-5" :disabled="status != 'running'" @click="compute">Compute</button> -->
      </section>

      <section class="output border-left w-50 p-5">
        <div class="slots">
          <div class="slot row" v-for="(slot, i) in slots" :key="i">
            <h5>{{ slot.num1 }}</h5>
            <h5
              :style="{ background: slot.op.color }"
            >{{ slot.op.sign }}</h5>
            <h5>{{ slot.num2 }}</h5>
            <h5>=</h5>
            <h5>{{ slot.res }}</h5>
          </div>
        </div>
        <div v-show="showPopup" class="result-bg"></div>
        <div v-if="showPopup" class="result text-center">
          <div class="text-success" v-if="status == 'won'">
            <h3>Won</h3>
            <p>Score (Slots + Operators Used + Time left): {{ currentSlot+1 }} + {{ operatorsUsed.length }} + {{ timer }} = {{ currentSlot+1+operatorsUsed.length+timer }}</p>
          </div>
          <h3 class="text-danger" v-else>Lost</h3>
        </div>

        <button class="btn btn-secondary help-btn" @click="showHelp = !showHelp">Help</button>
        <div class="help p-3" v-if="showHelp">
          50 * 3 = 150 <br>
          150 + 75 + 7 + 1 = 233 <br>
          233 * 4 = 932
        </div>
      </section>
      <section class="timer w-50">
        <div class="timer-inner" :style="{ width: timer/totalTime*100 + '%' }">{{ timer }}</div>
      </section>
    </div>

  </div>
</template>

<script>
import Toolbar from '@/components/Toolbar/Toolbar';

export default {
  name: "App",
  components: {
    Toolbar
  },
  data() {
    return {
      totalTime: 300,
      timer: 300,
      timerInterval: null,
      target: 932,
      numbers: [3, 4, 1, 7, 50, 75],
      operators: [
        {
          sign: "+",
          color: "#37AEC0"
        }, 
        {
          sign: "-",
          color: "#AEA621"
        }, 
        {
          sign: "*",
          color: "#DA5074"
        }, 
        {
          sign: "/",
          color: "#C98E00"
        }, 
      ],
      selectedNumbers: [],
      selectedOp: {},
      operatorsUsed: [],
      currentSlot: 0,
      slots: [
        {
          num1: null,
          num2: null,
          op: '',
          res: null
        },
      ],
      status: 'not-started',
      showHelp: false,
      showPopup: false
    };
  },
  watch: {
    selectedNumbers: function(newVal, oldVal) {
      this.slots[this.currentSlot].num1 = this.selectedNumbers[0];
      this.slots[this.currentSlot].num2 = this.selectedNumbers[1];
    },
    selectedOp: function(newVal, oldVal) {
      this.slots[this.currentSlot].op = this.selectedOp;
    },
    status: function(newVal, oldVal) {
      if(newVal != 'running') {
        this.showPopup = true;
      }
    }
  },
  mounted() {
    let vm = this;

    document.getElementsByClassName('result-bg')[0].addEventListener('click', () => {
      vm.showPopup = false;
    });
  },
  methods: {
    selectNumber(n) {
      if (this.selectedNumbers.length < 2) {
        this.selectedNumbers.push(n);
        
        if(this.selectedNumbers.length == 2 && typeof this.selectedOp.sign != 'undefined') {
          this.slots[this.currentSlot].num1 = this.selectedNumbers[0];
          this.slots[this.currentSlot].num2 = this.selectedNumbers[1];
          this.compute();
        }
      }
    },
    unselectNumber(n) {
      let i = this.selectedNumbers.indexOf(n);
      if (i != -1) {
        this.selectedNumbers.splice(i, 1);
      }
    },
    selectOp(op) {
      this.selectedOp = op;

      if(this.selectedNumbers.length == 2) {
        this.slots[this.currentSlot].op = this.selectedOp;
        this.compute();
      }
    },
    unselectOp(op) {
      this.selectedOp = {};
    },
    start() {
      let vm = this;
      this.timer = this.totalTime;
      this.timerInterval = setInterval(function() {
        vm.timer -= 1;
        if(vm.timer == 0) {
          clearInterval(vm.timerInterval);
        }
      }, 1000);
      this.status = 'running';
    },
    restart() {
      let vm = this;
      clearInterval(vm.timerInterval);
      this.timer = this.totalTime;
      this.timerInterval = setInterval(function() {
        vm.timer -= 1;
        if(vm.timer == 0) {
          clearInterval(vm.timerInterval);
        }
      }, 1000);
      this.selectedNumbers = [];
      this.selectedOp = {};
      this.numbers = [3, 4, 1, 7, 50, 75];
      this.currentSlot = 0;
      this.slots = [{
        num1: null,
        num2: null,
        op: '',
        res: null
      }];
      
    },
    compute() {
      // Checking the required conditions
      if(this.selectedNumbers.length < 2 || this.selectedOp.sign == '') {
        return;
      }

      let num1 = this.selectedNumbers[0],
          num2 = this.selectedNumbers[1]

      // Computing the result
      switch(this.slots[this.currentSlot].op.sign) {
        case '+':
          this.slots[this.currentSlot].res = num1 + num2;
          break;
        case '-':
          this.slots[this.currentSlot].res = num1 - num2;
          break;
        case '*':
          this.slots[this.currentSlot].res = num1 * num2;
          break;
        case '/':
          this.slots[this.currentSlot].res = Math.round((num1 / num2)*100)/100;
          break;
      }

      // Update numbers array
      this.numbers.splice(this.numbers.indexOf(num1), 1);
      this.numbers.splice(this.numbers.indexOf(num2), 1);
      this.numbers.push(this.slots[this.currentSlot].res);

      // Update operatorsUsed
      if(this.operatorsUsed.indexOf(this.slots[this.currentSlot].op.sign) == -1) {
        this.operatorsUsed.push(this.slots[this.currentSlot].op.sign);
      }

      // Checking answer and end of game
      if(this.target == this.slots[this.currentSlot].res) {
        this.status = 'won';
        clearInterval(this.timerInterval);
        return;
      } else if(this.numbers.length == 1) {
        this.status = 'lost';
        clearInterval(this.timerInterval);
        return;
      }

      // Reinitializing state variables
      this.selectedNumbers = [];
      this.selectedOp = {};
      this.currentSlot++;
      this.slots.push({
        num1: null,
        num2: null,
        op: '',
        res: null
      });
    },
  }
};
</script>

<style>
#app {
  min-height: 100vh;
  background: #3c295b;
  color: white;
}
.main-container {
  display: flex;
  height: calc(100vh - 55px);
}
.selection {
  padding-top: 50px;
}
.target-wrap {
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 40px 0;
  filter: drop-shadow(0 5px 15px rgba(255, 174, 0, 0.3));
}
.target {
  width: 200px;
  height: 200px;
  border-radius: 50%;
  padding: 80px;
  margin: 0px auto;
  background: orange;
  clip-path: polygon(50% 0%, 61% 35%, 98% 35%, 68% 57%, 79% 91%, 50% 70%, 21% 91%, 32% 57%, 2% 35%, 39% 35%);
}
.numbers {
  display: flex;
  width: fit-content;
  margin: 0 auto;
  /* opacity: 0.5; */
}
.focus, .selected {
  opacity: 1;
}
.focused::before {
  content: '';
  position: fixed;
  left: 0;
  top: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.856);
  z-index: -1;
}
.operators {
  display: flex;
  width: fit-content;
  margin: 0 auto;
  /* opacity: 0.5; */
}
.btn-number {
  min-width: 80px;
  font-size: 2em;
  font-weight: bold;
  padding: 10px;
  margin: 5px;
  border: solid 1px white;
  border-radius: 50%;
  background: transparent;
  transition: all 100ms ease;
}
.btn-number:hover {
  font-size: 1.6em;
  background: rgba(0, 0, 0, 0.2);
}
/* .btn-number:nth-child(1), .btn-number:nth-child(6) {
  transform: translateY(140px);
}
.btn-number:nth-child(2), .btn-number:nth-child(5) {
  transform: translateY(70px);
} */
.btn-operator {
  width: 100px;
  font-size: 2em;
  font-weight: bold;
  padding: 20px;
}
.btn-operator:hover {
  font-size: 1.6em;
  filter: brightness(0.8) !important;
}
button:disabled {
  opacity: 0.5 !important;
}
button:disabled:hover {
  font-size: 2em;
}

.selected {
  transform: scale(1.05);
  background: rgba(0, 89, 255, 0.514);
  box-shadow: 0 10px 20px rgba(255, 255, 255, 0.2) !important;
}
.timer {
  position: absolute;
  bottom: 5px;
  left: 0;
  background: white;
  height: 15px;
}
.timer-inner {
  width: 80%;
  height: 100%;
  background: rgb(85, 204, 81);
  text-align: center;
  font-size: 10px;
}
.slot h5 {
  min-width: 60px;
  margin: 10px;
  padding: 15px;
  text-align: center;
  border-radius: 10px;
  border: solid 1px rgba(128, 128, 128, 0.233);
}
.result {
  position: absolute;
  top: 0;
  left: 0;
  bottom: 0;
  right: 0;
  margin: auto;
  width: 50%;
  height: fit-content;
  padding: 40px;
  background: rgb(255, 255, 255);
  z-index: 2;
}
.result-bg {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.493);
  z-index: 1;
}
.help-btn {
  position: fixed;
  bottom: 20px;
  right: 20px;
}
.help {
  position: fixed;
  bottom: 70px;
  right: 20px;
  background: black;
  border-radius: 10px;
}
</style>
