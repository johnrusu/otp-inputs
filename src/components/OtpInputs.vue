<script setup>
import { defineProps, ref, onMounted, computed } from "vue";
import { useClipboard } from "../composables/useClipboard";

// Clipboard composable usage
const { isCopied, error, copy, read } = useClipboard();

const inputRefs = ref([]);

const props = defineProps({
  inputs: { default: 4, type: Number },
  acceptanceRexExp: { default: /[^\d]+/g, type: RegExp },
});

const otpArray = ref(Array(props.inputs).fill(""));

const handlePaste = (ev, index) => {
  ev.preventDefault();
  const text = ev.clipboardData
    .getData("text/plain")
    .replace(props.acceptanceRexExp, "");
  if (text.length === props.inputs) {
    for (let i = 0; i < props.inputs; i++) {
      inputRefs.value[i].value = text[i];
      otpArray.value[i] = text[i];
    }
    inputRefs.value[props.inputs - 1].focus();
  }
};

const handleInput = (ev, index) => {
  const value = ev.target.value.replace(props.acceptanceRexExp, "");
  ev.target.value = value;
  otpArray.value[index] = value;
  if (value.length === 1 && index < props.inputs - 1) {
    inputRefs.value[index + 1].focus();
  } else if (value.length === 0 && index > 0) {
    inputRefs.value[index - 1].focus();
  }
};

const handleKeyDown = (ev, index) => {
  if (ev.key === "Backspace" && index > 0 && !ev.target.value) {
    inputRefs.value[index - 1].focus();
  }
};

const resetOtp = () => {
  otpArray.value = Array(props.inputs).fill("");
  inputRefs.value.forEach((input) => (input.value = ""));
  inputRefs.value[0].focus();
};

// Compute OTP value from otpArray
const otpValue = computed(() => otpArray.value.join(""));
</script>
<template>
  <div class="otp-container flex gap-5">
    <input
      :id="`opt-input-${index}`"
      v-for="(input, index) in props.inputs"
      :key="`opt-input-${index}`"
      class="w-12 h-12 text-center text-xl border border-gray-300 mx-1 rounded flex-inline"
      :ref="(el) => (inputRefs[index] = el)"
      @input="(e) => handleInput(e, index)"
      @keydown="(e) => handleKeyDown(e, index)"
      @paste="(e) => handlePaste(e, index)"
      maxlength="1"
    />
    <div v-if="otpValue && otpValue.length === props.inputs">
      <button
        class="ml-4 px-3 py-2 bg-blue-500 text-white rounded h-12 cursor-pointer"
        @click="copy(otpValue)"
      >
        Copy OTP
      </button>
      <button
        @click="resetOtp"
        class="ml-2 px-3 py-2 bg-gray-300 text-black rounded h-12 cursor-pointer"
      >
        Reset OTP
      </button>
      <span v-if="isCopied" class="ml-2 text-green-600">Copied!</span>
      <span v-if="error" class="ml-2 text-red-600">Copy failed</span>
    </div>
  </div>
</template>
