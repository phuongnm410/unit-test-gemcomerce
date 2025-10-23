<template>
    <div>
        <div class="wrapper font-inter flex flex-col">
            <div class="wrapper_unit grid grid-cols-2 p-2">
                <span class='text-[#AAA] font-normal'>Unit</span>
                <div class='bg-[#212121] rounded-[8px] grid grid-cols-2 text-center text-[#F9F9F9] cursor-pointer h-10'>
                    <button @click="switchUnit('%')"
                        :class="unit === '%' ? 'bg-[#424242] rounded-[6px] m-1 cursor-pointer transition' : 'cursor-pointer text-[#AAA]'">%</button>
                    <button @click="switchUnit('px')"
                        :class="unit === 'px' ? 'bg-[#424242] rounded-[6px] m-1 cursor-pointer transition' : 'cursor-pointer text-[#AAA]'">px</button>
                </div>
            </div>
            <div class="wrapper_value grid grid-cols-2 p-2">
                <span class="text-[#AAA] font-normal">Value</span>
                <div class="bg-[#212121] grid grid-cols-[1fr_1.5fr_1fr] rounded-[8px] text-center items-center h-10 overflow-visible"
                    :class="{
                        'bg-[#3B3B3B]': hoverInput && !onFocus,
                        'outline outline-[#3C67FF]': onFocus
                    }">
                    <div class="relative">
                        <button class="flex items-center justify-center w-12 h-10  transition cursor-pointer rounded-l-[8px]"
                            @click="checkMinus" :disabled="isMinusDisabled" @mouseenter="handleMinusHover(true)"
                            @mouseleave="handleMinusHover(false)"
                            :class="{ 'bg-[#3B3B3B]': hoverMinus && !isMinusDisabled }"><svg
                                xmlns="http://www.w3.org/2000/svg" width="12" height="2" viewBox="0 0 12 2" fill="none">
                                <path fill-rule="evenodd" clip-rule="evenodd"
                                    d="M0 0.75C0 0.335786 0.335786 0 0.75 0L11.25 0C11.6642 0 12 0.335786 12 0.75C12 1.16421 11.6642 1.5 11.25 1.5H0.75C0.335786 1.5 0 1.16421 0 0.75Z"
                                    :fill="isMinusDisabled ? '#AAA' : '#F9F9F9'" />
                            </svg>

                        </button>
                        <div v-if="MinusTooltipVisible"
                            class="pointer-events-none absolute left-1/2 top-0 -translate-x-1/2 -translate-y-[calc(100%+16px)]
              bg-[#212121] text-[#F9F9F9] font-normal py-[3px] px-[8px] rounded-[8px] whitespace-nowrap z-10 text-center">
                            Value must be greater than 0
                            <div
                                class="absolute left-1/2 -translate-x-1/2 bottom-[-6px]
                w-0 h-0 border-l-[6px] border-l-transparent border-r-[6px] border-r-transparent border-t-[6px] border-t-[#212121]">
                            </div>
                        </div>
                    </div>

                    <input class="bg-transparent outline-none border-none w-full text-center h-10 text-[#F9F9F9] overflow-visible"
                        type="text" name="" inputmode="decimal" v-model="valueInput" @mouseenter="hoverInput = true"
                        @mouseleave="hoverInput = false" @focus="onFocus = true" @blur="handleValidate()">
                    <div class="relative">

                        <button class="flex items-center justify-center w-12 h-10 transition cursor-pointer rounded-r-[8px]"
                            @click="checkPlus" :disabled="isPlusDisabled" @mouseenter="handlePlusHover(true)"
                            @mouseleave="handlePlusHover(false)"
                            :class="{ 'bg-[#3B3B3B]': hoverPlus && !isPlusDisabled }"><svg
                                xmlns="http://www.w3.org/2000/svg" width="12" height="12" viewBox="0 0 12 12"
                                fill="none">
                                <path
                                    d="M6.75 0.75C6.75 0.335786 6.41421 0 6 0C5.58579 0 5.25 0.335786 5.25 0.75V5.25H0.75C0.335786 5.25 0 5.58579 0 6C0 6.41421 0.335786 6.75 0.75 6.75H5.25L5.25 11.25C5.25 11.6642 5.58579 12 6 12C6.41421 12 6.75 11.6642 6.75 11.25V6.75H11.25C11.6642 6.75 12 6.41421 12 6C12 5.58579 11.6642 5.25 11.25 5.25H6.75V0.75Z"
                                    :fill="isPlusDisabled ? '#AAA' : '#F9F9F9'" />
                            </svg>

                            <div v-if="PlusTooltipVisible" class="pointer-events-none absolute left-1/2 top-0 -translate-x-1/2 -translate-y-[calc(100%+8px)]
             bg-[#212121] text-[#F9F9F9] font-normal py-[3px] px-[8px]
             rounded-[8px] whitespace-nowrap z-10 text-center">
                                Value must be smaller than 100
                                <div class="absolute left-1/2 -translate-x-1/2 bottom-[-6px]
                 w-0 h-0 border-l-[6px] border-l-transparent
                 border-r-[6px] border-r-transparent
                 border-t-[6px] border-t-[#212121]"></div>
                            </div>
                        </button>
                    </div>


                </div>
            </div>
        </div>
    </div>
</template>
<script lang="ts" setup>
import { ref, computed } from 'vue'

type Unit = '%' | 'px'

const unit = ref<Unit>('%');

const valueInput = ref<string | number>('0');
const preValue = ref<string | number>('0');

const hoverInput = ref(false);
const hoverMinus = ref(false);
const hoverPlus = ref(false);
const onFocus = ref(false);


const MIN_VALUE = 0
const MAX_VALUE = computed(() => (unit.value === '%' ? 100 : Infinity));

const isMinusDisabled = computed(() => Number(valueInput.value) <= MIN_VALUE);
const isPlusDisabled = computed(() => unit.value === '%' ? Number(valueInput.value) >= 100 : false)

const MinusTooltipVisible = ref(false);
const PlusTooltipVisible = ref(false);

function validateInput(input: string | number): number {

    if (input === null || input === undefined || input === '') return 0;

    try {
        let cleaned = String(input).replace(',', '.');
        cleaned = cleaned.replace(/[^0-9.]/g, '');

        const parts = cleaned.split('.');
        if (parts.length > 2) {
            cleaned = parts[0] + '.' + parts[1];
        }

        let num = parseFloat(cleaned);
        if (isNaN(num)) num = 0;
        if (num < 0) num = 0;

        if (unit.value === '%' && num > 100) return Number(preValue.value);

        return num;

    } catch (error) {
        console.error('Validate error:', error);
        return 0;
    }

}

//out focus call handleValidate
function handleValidate() {
    const validated = validateInput(valueInput.value)
    valueInput.value = validated.toString()
    preValue.value = valueInput.value
    onFocus.value = false
}

function checkMinus() {
    let number = validateInput(valueInput.value);
    number = number - 1;
    if (number < MIN_VALUE) number = MIN_VALUE;
    valueInput.value = number.toString();
    preValue.value = valueInput.value;
}

function checkPlus() {
    let number = validateInput(valueInput.value);
    number = number + 1;
    if (number > MAX_VALUE.value) number = MAX_VALUE.value;
    valueInput.value = number.toString();
    preValue.value = valueInput.value;
}

function switchUnit(u: Unit) {
    unit.value = u;
    let number = validateInput(u);
    if (unit.value === '%' && number > 100) {
        number = 100;
    }
    valueInput.value = number.toString();
    preValue.value = valueInput.value;
}

// hover control with tooltip 
function handleMinusHover(state: boolean) {
    if (isMinusDisabled.value) {
        hoverMinus.value = false;
        MinusTooltipVisible.value = state;
    } else {
        hoverMinus.value = state;
        MinusTooltipVisible.value = false;
    }
}

function handlePlusHover(state: boolean) {
    if (isPlusDisabled.value) {
        hoverPlus.value = false;
        PlusTooltipVisible.value = state;
    } else {
        hoverPlus.value = state;
        PlusTooltipVisible.value = false;
    }
}
</script>
