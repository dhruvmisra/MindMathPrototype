<template>
  <div id="app">
    <h1 class="text-center">Mind Math Prototype</h1>

    <div class="main-container">
      <section class="timer w-50">
        <div class="timer-inner" :style="{ width: timer/totalTime*100 + '%' }">{{ timer }}</div>
      </section>

      <section class="selection w-50 p-5">
        <p class="text-center">Select 2 numbers and an operator and press Compute</p>
        <h3 class="target text-center p-4 my-5">Target: {{ target }}</h3>
        <h5 class="mt-5">Numbers: {{ selectedNumbers }}</h5>
        <div class="numbers row justify-content-center mx-0">
          <button
            class="btn btn-warning m-3"
            :class="{ selected: selectedNumbers.indexOf(number) != -1 }"
            @click="status == 'running' && selectedNumbers.indexOf(number) != -1 ? unselectNumber(number) : selectNumber(number)"
            v-for="number in numbers"
            :key="number"
          >{{ number }}</button>
        </div>
        <h5 class="mt-5">Operator: {{ selectedOp }}</h5>
        <div class="operators row justify-content-center mx-0">
          <button
            class="btn btn-danger m-3"
            :class="{ selected: selectedOp == operator }"
            @click="status == 'running' && selectedOp == operator ? unselectOp(operator) : selectOp(operator)"
            v-for="operator in operators"
            :key="operator"
          >{{ operator }}</button>
        </div>
        <p class="mt-3">Operators Used: {{ operationsUsed }}</p>

        <button class="btn btn-info d-block mx-auto my-5 w-100" :disabled="status != 'running'" @click="compute">Compute</button>
      </section>

      <section class="output border-left w-50 p-5">
        <h5 class="text-center">Output Slots</h5>
        <div class="slots">
          <div class="slot row" v-for="(slot, i) in slots" :key="i">
            <h5>{{ slot.num1 }}</h5>
            <h5>{{ slot.op }}</h5>
            <h5>{{ slot.num2 }}</h5>
            <h5>=</h5>
            <h5>{{ slot.res }}</h5>
          </div>
        </div>
        <div v-if="status != 'running'" class="result text-center my-3 p-3">
          <div class="text-success" v-if="status == 'won'">
            <h3>Won</h3>
            <p>Score (Slots + Operators Used + Time left): {{ currentSlot+1 }} + {{ operationsUsed.length }} + {{ timer }} = {{ currentSlot+1+operationsUsed.length+timer }}</p>
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
    </div>

  </div>
</template>

<script>
export default {
  name: "App",
  components: {},
  data() {
    return {
      totalTime: 300,
      timer: 120,
      timerInterval: null,
      target: 932,
      numbers: [3, 4, 1, 7, 50, 75],
      operators: ["+", "-", "*", "/"],
      selectedNumbers: [],
      selectedOp: '',
      operationsUsed: [],
      currentSlot: 0,
      slots: [
        {
          num1: null,
          num2: null,
          op: '',
          res: null
        },
      ],
      status: 'running',
      showHelp: false,
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
  },
  mounted() {
    let vm = this;
    this.timer = this.totalTime;
    this.timerInterval = setInterval(function() {
      vm.timer -= 1;
      if(vm.timer == 0) {
        vm.status = 'lost';
        clearInterval(vm.timerInterval);
      }
    }, 1000)
  },
  methods: {
    selectNumber(n) {
      if (this.selectedNumbers.length < 2) {
        this.selectedNumbers.push(n);
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
    },
    unselectOp(op) {
      this.selectedOp = '';
    },
    compute() {
      if(this.selectedNumbers.length < 2 || this.selectedOp == '') {
        return;
      }

      let num1 = this.selectedNumbers[0],
          num2 = this.selectedNumbers[1]

      switch(this.slots[this.currentSlot].op) {
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

      this.numbers.splice(this.numbers.indexOf(num1), 1);
      this.numbers.splice(this.numbers.indexOf(num2), 1);
      this.numbers.push(this.slots[this.currentSlot].res);

      if(this.operationsUsed.indexOf(this.slots[this.currentSlot].op) == -1) {
        this.operationsUsed.push(this.slots[this.currentSlot].op);
      }

      if(this.target == this.slots[this.currentSlot].res) {
        this.status = 'won';
        clearInterval(this.timerInterval);
        return;
      } else if(this.numbers.length == 1) {
        this.status = 'lost';
        clearInterval(this.timerInterval);
        return;
      }

      this.selectedNumbers = [];
      this.selectedOp = '';
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
  height: 90vh;
}
.selected {
  transform: translateY(-10px) !important;
  box-shadow: 0 5px 10px rgba(255, 255, 255, 0.5) !important;
  transition: all ease 150ms !important;
}
.timer {
  position: absolute;
  top: 55px;
  left: 50%;
  transform: translateX(-50%);
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
  margin: 10px;
  padding: 20px;
  border-radius: 10px;
  border: solid 1px rgba(128, 128, 128, 0.233);
}
.result, .target, .numbers, .operators {
  background: rgb(62, 42, 92);
  box-shadow: 0 5px 15px rgba(40, 27, 59, 0.6);
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
