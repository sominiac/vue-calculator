<script setup>
import {onMounted, onBeforeUnmount, ref, useTemplateRef} from "vue";

const DEFAULT_VALUE = 0;
const MATH_OPERATION_KEYS = {
  plus: "PLUS",
  minus: "MINUS",
  multiply: "MULTIPLY",
  divide: "DIVIDE",
  percent: "PERCENT",
  fraction: "FRACTION",
  square: "SQUARE",
  squareRoot: "SQUARE_ROOT",
  negative: "NEGATIVE",
}
const MATH_OPERATION_VALUE = {
  [MATH_OPERATION_KEYS.plus]: "+",
  [MATH_OPERATION_KEYS.minus]: "-",
  [MATH_OPERATION_KEYS.multiply]: "x",
  [MATH_OPERATION_KEYS.divide]: "/",
}

const amount = ref(DEFAULT_VALUE);
const firstValue = ref(DEFAULT_VALUE);
const secondValue = ref(DEFAULT_VALUE);
const mathOperation = ref(null);
const sum = ref(DEFAULT_VALUE);
const isAmountCalculated = ref(false);
const history = ref([]);

const equalRef = useTemplateRef('equalRef');

onMounted(() => {
  window.addEventListener("keydown", keyUpHandle)
})

onBeforeUnmount(() => {
  window.removeEventListener("keydown", keyUpHandle)
})

function getPercent() {
  if (firstValue.value === DEFAULT_VALUE) {
    _reset()
    return;
  }

  if (mathOperation.value === MATH_OPERATION_KEYS.plus || mathOperation.value === MATH_OPERATION_KEYS.minus) amount.value = firstValue.value * amount.value / 100;
  if (mathOperation.value === MATH_OPERATION_KEYS.multiply || mathOperation.value === MATH_OPERATION_KEYS.divide) amount.value = amount.value / 100;
}

function getFraction() {
  amount.value = 1 / amount.value;
  firstValue.value = amount.value;
}

function getSquare() {
  amount.value = amount.value * amount.value;
  firstValue.value = amount.value;
}

function getSquareRoot() {
  amount.value = Math.sqrt(amount.value);
  firstValue.value = amount.value;
}

function keyUpHandle({code, key}) {
  if (code.includes("Digit") && (+key || key === "0")) {
    setAmount(key);
  }

  if (code.includes("Enter")) {
    equalRef.value.focus();
  }

  if (code.includes("Backspace")) {
    _delete();
  }

  if (code.includes("Delete")) {
    _reset();
  }

  if (code.includes("Escape")) {
    _reset(false);
  }

  if (code.includes("Comma") || code.includes("Period")) {
    setAmount(",")
  }

  if (code.includes("Minus")) {
    _math(MATH_OPERATION_KEYS.minus)
  }

  if (code.includes("Equal")) {
    _math(MATH_OPERATION_KEYS.plus)
  }

  if (code.includes("Slash")) {
    _math(MATH_OPERATION_KEYS.divide)
  }

  if (key === "*") {
    _math(MATH_OPERATION_KEYS.multiply)
  }

  if (key === "%") {
    getPercent();
  }
}

function onTransformation(mathOperationKey) {
  if (mathOperationKey === MATH_OPERATION_KEYS.percent) getPercent();
  if (mathOperationKey === MATH_OPERATION_KEYS.fraction) getFraction();
  if (mathOperationKey === MATH_OPERATION_KEYS.square) getSquare();
  if (mathOperationKey === MATH_OPERATION_KEYS.squareRoot) getSquareRoot();
  if (mathOperationKey === MATH_OPERATION_KEYS.negative) amount.value *= -1;
}

function onClick(event) {
  const value = event.target.textContent;

  setAmount(value);
}

function onReset(isFullReset = true) {
  _reset(isFullReset);
}

function onDelete() {
  _delete();
}

function onMath(mathOperationKey) {
  _math(mathOperationKey);
}

function onCalculate() {
  _calculate(mathOperation.value);
}

function setAmount(value) {
  if (value === ",") {
    amount.value = `${amount.value}.`;
    return;
  }

  if (amount.value && !isAmountCalculated.value) {
    amount.value = Number(`${amount.value}${value}`);
  } else {
    amount.value = value;
    isAmountCalculated.value = false;
  }
}

function setExpression(item) {
  firstValue.value = item.firstValue;
  secondValue.value = item.secondValue;
  mathOperation.value = item.mathOperation;
  amount.value = item.sum;
}

function _reset(isFullReset = true) {
  amount.value = DEFAULT_VALUE;
  secondValue.value = DEFAULT_VALUE;

  if (isFullReset) {
    firstValue.value = DEFAULT_VALUE;
    mathOperation.value = null;
    sum.value = DEFAULT_VALUE;
  }
}

function _math(mathOperationKey) {
  firstValue.value = amount.value;
  mathOperation.value = mathOperationKey;
  _reset(false);
}

function _delete() {
  if (isAmountCalculated.value) {
    _reset();
    return;
  }

  amount.value = +String(amount.value).slice(0, -1);
}

function _calculate(value) {
  secondValue.value = amount.value;

  if (value === MATH_OPERATION_KEYS.plus) sum.value = +firstValue.value + +secondValue.value;
  if (value === MATH_OPERATION_KEYS.minus) sum.value = firstValue.value - secondValue.value;
  if (value === MATH_OPERATION_KEYS.multiply) sum.value = firstValue.value * secondValue.value;
  if (value === MATH_OPERATION_KEYS.divide) sum.value = firstValue.value / secondValue.value;

  amount.value = sum.value;
  isAmountCalculated.value = true;

  history.value.push({firstValue: firstValue.value, secondValue: secondValue.value, mathOperation: mathOperation.value, sum: sum.value});
}

</script>

<template>
  <section class="flex-grow-1 d-flex justify-content-between align-items-start overflow-hidden gap-3 mt-3">
    <div class="h-100 d-flex flex-column flex-grow-1">
      <header>
        <h2 class="fs-2 fw-semibold">Calculator</h2>
      </header>
      <div class="flex-grow-1 d-grid gap-2 mt-3">
        <div class="row g-0 d-flex flex-column justify-content-end align-items-end">
          <span class="fs-6 text-end" v-if="firstValue">{{firstValue}}&nbsp;{{MATH_OPERATION_VALUE[mathOperation]}}<span v-if="secondValue">&nbsp;{{secondValue}}</span><span v-if="sum">&nbsp;=</span></span>
          <span class="fs-1 text-end">{{ amount }}</span>
        </div>
        <div class="row g-0 text-center column-gap-2">
          <button class="col-sm btn rounded math-operation fs-4" @click="onTransformation(MATH_OPERATION_KEYS.percent)">
            <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="currentColor" class="bi bi-percent" viewBox="0 0 16 16">
              <path d="M13.442 2.558a.625.625 0 0 1 0 .884l-10 10a.625.625 0 1 1-.884-.884l10-10a.625.625 0 0 1 .884 0M4.5 6a1.5 1.5 0 1 1 0-3 1.5 1.5 0 0 1 0 3m0 1a2.5 2.5 0 1 0 0-5 2.5 2.5 0 0 0 0 5m7 6a1.5 1.5 0 1 1 0-3 1.5 1.5 0 0 1 0 3m0 1a2.5 2.5 0 1 0 0-5 2.5 2.5 0 0 0 0 5"/>
            </svg>
          </button>
          <button class="col-sm btn rounded math-operation fs-4" @click="onReset(false)">CE</button>
          <button class="col-sm btn rounded math-operation fs-4" @click="onReset">C</button>
          <button class="col-sm btn rounded math-operation fs-4" @click="onDelete">
            <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="currentColor" class="bi bi-backspace" viewBox="0 0 16 16">
              <path d="M5.83 5.146a.5.5 0 0 0 0 .708L7.975 8l-2.147 2.146a.5.5 0 0 0 .707.708l2.147-2.147 2.146 2.147a.5.5 0 0 0 .707-.708L9.39 8l2.146-2.146a.5.5 0 0 0-.707-.708L8.683 7.293 6.536 5.146a.5.5 0 0 0-.707 0z"/>
              <path d="M13.683 1a2 2 0 0 1 2 2v10a2 2 0 0 1-2 2h-7.08a2 2 0 0 1-1.519-.698L.241 8.65a1 1 0 0 1 0-1.302L5.084 1.7A2 2 0 0 1 6.603 1zm-7.08 1a1 1 0 0 0-.76.35L1 8l4.844 5.65a1 1 0 0 0 .759.35h7.08a1 1 0 0 0 1-1V3a1 1 0 0 0-1-1z"/>
            </svg>
          </button>
        </div>
        <div class="row g-0 text-center column-gap-2">
          <button class="col-sm btn rounded math-operation fs-4" @click="onTransformation(MATH_OPERATION_KEYS.fraction)">1/x</button>
          <button class="col-sm btn rounded math-operation fs-4" @click="onTransformation(MATH_OPERATION_KEYS.square)">
            <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="currentColor" class="bi bi-superscript" viewBox="0 0 16 16">
              <path d="m4.266 12.496.96-2.853H8.76l.96 2.853H11L7.62 3H6.38L3 12.496zm2.748-8.063 1.419 4.23h-2.88l1.426-4.23zm5.132-1.797v-.075c0-.332.234-.618.619-.618.354 0 .618.256.618.58 0 .362-.271.649-.52.898l-1.788 1.832V6h3.59v-.958h-1.923v-.045l.973-1.04c.415-.438.867-.845.867-1.547 0-.8-.701-1.41-1.787-1.41C11.565 1 11 1.8 11 2.576v.06z"/>
            </svg>
          </button>
          <button class="col-sm btn rounded math-operation fs-4" @click="onTransformation(MATH_OPERATION_KEYS.squareRoot)">SQRT <span title="Square root">
            <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" fill="currentColor" class="bi bi-info-circle" viewBox="0 0 16 16">
            <path d="M8 15A7 7 0 1 1 8 1a7 7 0 0 1 0 14m0 1A8 8 0 1 0 8 0a8 8 0 0 0 0 16"/>
            <path d="m8.93 6.588-2.29.287-.082.38.45.083c.294.07.352.176.288.469l-.738 3.468c-.194.897.105 1.319.808 1.319.545 0 1.178-.252 1.465-.598l.088-.416c-.2.176-.492.246-.686.246-.275 0-.375-.193-.304-.533zM9 4.5a1 1 0 1 1-2 0 1 1 0 0 1 2 0"/>
          </svg>
          </span></button>
          <button class="col-sm btn rounded math-operation fs-4" @click="onMath(MATH_OPERATION_KEYS.divide)">
            <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" fill="currentColor" class="bi bi-slash-lg" viewBox="0 0 16 16">
              <path fill-rule="evenodd" d="M13.854 2.146a.5.5 0 0 1 0 .708l-11 11a.5.5 0 0 1-.708-.708l11-11a.5.5 0 0 1 .708 0"/>
            </svg>
          </button>
        </div>
        <div class="row g-0 text-center column-gap-2">
          <button class="numbers col-sm btn rounded fs-4" @click="onClick">7</button>
          <button class="numbers col-sm btn rounded fs-4" @click="onClick">8</button>
          <button class="numbers col-sm btn rounded fs-4" @click="onClick">9</button>
          <button class="col-sm btn rounded math-operation fs-4" @click="onMath(MATH_OPERATION_KEYS.multiply)">
            <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" fill="currentColor" class="bi bi-x-lg" viewBox="0 0 16 16">
              <path d="M2.146 2.854a.5.5 0 1 1 .708-.708L8 7.293l5.146-5.147a.5.5 0 0 1 .708.708L8.707 8l5.147 5.146a.5.5 0 0 1-.708.708L8 8.707l-5.146 5.147a.5.5 0 0 1-.708-.708L7.293 8z"/>
            </svg>
          </button>
        </div>
        <div class="row g-0 text-center column-gap-2">
          <button class="numbers col-sm btn rounded fs-4" @click="onClick">4</button>
          <button class="numbers col-sm btn rounded fs-4" @click="onClick">5</button>
          <button class="numbers col-sm btn rounded fs-4" @click="onClick">6</button>
          <button class="col-sm btn rounded math-operation fs-4" @click="onMath(MATH_OPERATION_KEYS.minus)">
            <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" fill="currentColor" class="bi bi-dash-lg" viewBox="0 0 16 16">
              <path fill-rule="evenodd" d="M2 8a.5.5 0 0 1 .5-.5h11a.5.5 0 0 1 0 1h-11A.5.5 0 0 1 2 8"/>
            </svg>
          </button>
        </div>
        <div class="row g-0 text-center column-gap-2">
          <button class="numbers col-sm btn rounded fs-4" @click="onClick">1</button>
          <button class="numbers col-sm btn rounded fs-4" @click="onClick">2</button>
          <button class="numbers col-sm btn rounded fs-4" @click="onClick">3</button>
          <button class="col-sm btn rounded math-operation fs-4" @click="onMath(MATH_OPERATION_KEYS.plus)">
            <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" fill="currentColor" class="bi bi-plus-lg" viewBox="0 0 16 16">
              <path fill-rule="evenodd" d="M8 2a.5.5 0 0 1 .5.5v5h5a.5.5 0 0 1 0 1h-5v5a.5.5 0 0 1-1 0v-5h-5a.5.5 0 0 1 0-1h5v-5A.5.5 0 0 1 8 2"/>
            </svg>
          </button>
        </div>
        <div class="row g-0 text-center column-gap-2">
          <button class="numbers col-sm btn rounded fs-4" @click="onTransformation(MATH_OPERATION_KEYS.negative)">
            <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" fill="currentColor" class="bi bi-plus-slash-minus" viewBox="0 0 16 16">
              <path d="m1.854 14.854 13-13a.5.5 0 0 0-.708-.708l-13 13a.5.5 0 0 0 .708.708M4 1a.5.5 0 0 1 .5.5v2h2a.5.5 0 0 1 0 1h-2v2a.5.5 0 0 1-1 0v-2h-2a.5.5 0 0 1 0-1h2v-2A.5.5 0 0 1 4 1m5 11a.5.5 0 0 1 .5-.5h5a.5.5 0 0 1 0 1h-5A.5.5 0 0 1 9 12"/>
            </svg>
          </button>
          <button class="numbers col-sm btn rounded fs-4" @click="onClick">0</button>
          <button class="numbers col-sm btn rounded fs-4" @click="onClick">,</button>
          <button class="col-sm btn rounded bg-primary fs-4" ref="equalRef" @click="onCalculate">=</button>
        </div>
      </div>
    </div>
    <aside class="h-100 col-3 overflow-y-scroll">
      <div class="d-flex align-items-center gap-2">
        <span>History</span>
        <span>Memory</span>
      </div>
      <ul class="d-flex flex-column overflow-y-scroll mt-2 px-0">
        <li class="history-item w-100 d-inline-flex flex-column align-items-end row-gap-2 rounded p-1" v-for="item in history" @click="setExpression(item)">
          <span class="fs-6">{{ item.firstValue }}&nbsp;{{ MATH_OPERATION_VALUE[item.mathOperation] }}&nbsp;{{ item.secondValue }}&nbsp;=</span>
          <span class="fs-4">{{ item.sum }}</span>
        </li>
      </ul>
    </aside>
  </section>
</template>

<style scoped>
  .numbers {
    background-color: #dee2e6;
  }

  .math-operation {
    background-color: #ced4da;
  }

  .history-item {
    cursor: pointer;
  }

  .history-item:hover {
    background-color: #dee2e6;
  }
</style>