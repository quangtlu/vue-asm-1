<template>
  <div class="container">
    <div class="mx-auto w-4/6 mt-10">
      <div class="flex border-all w-fit shadow">
        <div v-for="step in steps" :key="step.value">
          <div
            class="border-r border-black p text-center px-2 cursor-pointer"
            :class="step.value === currentStep ? 'bg-blue-400' : ''"
            @click="jumpToStep(step.value)"
          >
            {{ step.label }}
          </div>
        </div>
      </div>
      <div class="form-content h-[400px] my-3 overflow-y-auto">
        <div v-show="currentStep === 1">
          <div class="flex flex-col gap-3">
            <div class="flex flex-col gap-1">
              <label for="meal">Please Select a meal</label>
              <select v-model="order.meal" id="meal" class="border-all w-[200px]">
                <option value="">---</option>
                <option v-for="meal in availableMeals" :value="meal">{{ meal }}</option>
              </select>
              <div v-if="!isValidMeal && isShowError" class="text-red-500">
                Please select a meal
              </div>
            </div>
            <div class="flex flex-col gap-1">
              <label for="meal">Please Enter number of people</label>
              <input
                v-model="order.totalPeople"
                type="number"
                min="1"
                max="10"
                class="border w-[100px] border-black"
              />
              <div v-if="!isValidTotalPeople && isShowError" class="text-red-500">
                Number of people must be between 1 and 10
              </div>
            </div>
          </div>
        </div>
        <div v-show="currentStep === 2">
          <div class="mx-auto pt-6">
            <div class="flex flex-col gap-1">
              <label for="restaurant">Please select a Restaurant</label>
              <select id="restaurant" class="border-all" v-model="order.restaurant">
                <option value="">---</option>
                <option :value="restaurant" v-for="restaurant in restaurants" :key="restaurant">
                  {{ restaurant }}
                </option>
              </select>
              <div v-if="!isValidRestaurant && isShowError" class="text-red-500">
                Please select a Restaurant
              </div>
            </div>
          </div>
        </div>
        <div v-show="currentStep === 3">
          <div v-for="dish in order.dishes" class="flex mt-2 gap-2 flex-wrap items-center">
            <div class="flex flex-col gap-1">
              <label for="dish">Please select a dish</label>
              <select class="border-all" id="dish" v-model="dish.item">
                <option value="">---</option>
                <option v-for="item in availableDishes" :key="item.id" :value="item">
                  {{ item.name }}
                </option>
              </select>
              <div :class="!dish.item && isShowError ? '' : 'opacity-0'" class="text-red-500">
                please select a meal
              </div>
            </div>
            <div class="flex flex-col gap-1">
              <label for="totalServing">Please enter no of serving</label>
              <input
                class="border-all w-10"
                type="number"
                min="1"
                v-model="dish.totalServing"
                id="totalServing"
              />
              <div
                :class="!dish.totalServing && isShowError ? '' : 'opacity-0'"
                class="text-red-500"
              >
                please select a meal
              </div>
            </div>
          </div>
          <div
            style="border-radius: 50%"
            class="cursor-pointer border border-black mt-10 w-6 h-6 flex items-center justify-center"
            @click="addOrderDish"
          >
            +
          </div>
          <div v-if="!isValidTotalOrder && isShowError" class="text-red-500">
            The total number of dishes (i.e Number of dishes * respective serving) should be greater
            or equal to the number of people selected in the first step and a maximum of 10 is
            allowed.
          </div>
        </div>
        <div v-show="currentStep === 4">
          <div class="grid grid-cols-3">
            <div>Meal:</div>
            <div class="col-span-2">{{ order.meal }}</div>
          </div>
          <div class="grid grid-cols-3">
            <div>No of people:</div>
            <div class="col-span-2">{{ order.totalPeople }}</div>
          </div>
          <div class="grid grid-cols-3">
            <div>Restaurant:</div>
            <div class="col-span-2">{{ order.restaurant }}</div>
          </div>
          <div class="grid grid-cols-3">
            <div>Dishes:</div>
            <div v-if="resultDishes.length" class="col-span-2 border-all p-1">
              <div v-for="(dish, index) in resultDishes" :key="index">
                {{ dish.item?.name }} - {{ dish.totalServing }}
              </div>
            </div>
          </div>
        </div>
      </div>
      <div
        class="flex justify-between items-center mt-5"
        :class="isShowNextBtn && !isShowPrevBtn ? 'float-right' : ''"
      >
        <button v-if="isShowPrevBtn" @click="currentStep--" class="border-all rounded p-1">
          Previous
        </button>
        <button v-if="isShowNextBtn" @click="nextStep" class="border-all rounded p-1">Next</button>
        <button v-else @click="submitOrder" class="border-all rounded p-1">
          {{ isShowNextBtn ? 'Next' : 'Submit' }}
        </button>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { computed, reactive, ref } from 'vue';
import { IDishItem, dishes } from './data/dishes';
const MAX_TOTAL = 10;
interface Step {
  value: number;
  label: string;
}

interface IDish {
  item: IDishItem;
  totalServing: number;
}

interface Order {
  meal: string;
  totalPeople: number;
  restaurant: string;
  dishes: IDish[];
}

const steps: Step[] = [
  {
    value: 1,
    label: 'Step 1',
  },
  {
    value: 2,
    label: 'Step 2',
  },
  {
    value: 3,
    label: 'Step 3',
  },
  {
    value: 4,
    label: 'Review',
  },
];
const isShowError = ref(false);
const availableMeals = ['lunch', 'dinner', 'breakfast'];
const currentStep = ref(1);
const order = reactive<Order>({
  dishes: [
    {
      totalServing: 1,
    },
  ],
} as Order);
const isShowPrevBtn = computed(() => currentStep.value > 1);
const isShowNextBtn = computed(() => currentStep.value < steps.length);
const isValidMeal = computed(() => !!order.meal);
const isValidTotalPeople = computed(() => !!order.totalPeople && order.totalPeople <= MAX_TOTAL);
const isValidRestaurant = computed(() => !!order.restaurant);
const totalDishes = computed(() =>
  order.dishes.reduce((prev, current) => prev + current.totalServing, 0)
);
const isValidTotalOrder = computed(
  () => totalDishes.value >= order.totalPeople && totalDishes.value <= MAX_TOTAL
);

const isValidDishes = computed(() =>
  order.dishes.every((item) => item.item && item.totalServing && item.totalServing)
);

const restaurants = computed(() => {
  const availableRestaurant = dishes
    .filter((dish) => dish.availableMeals.includes(order.meal))
    .map((item) => item.restaurant);
  return [...new Set(availableRestaurant)];
});
const availableDishes = computed(() =>
  dishes.filter((item) => item.restaurant === order.restaurant)
);

const resultDishes = computed(() => {
  let result = order.dishes.reduce((acc: IDish[], obj) => {
    let existingObj = acc.find((o) => o.item?.id === obj.item?.id);
    if (existingObj) {
      existingObj.totalServing += obj.totalServing;
    } else {
      acc.push({ ...obj });
    }
    return acc;
  }, []);
  return result;
});

const validateDataByStep = () => {
  let isValidData = true;
  switch (currentStep.value) {
    case 1:
      isValidData = isValidMeal.value && isValidTotalPeople.value;
      break;
    case 2:
      isValidData = isValidRestaurant.value;
      break;
    case 3:
      isValidData = isValidDishes.value && isValidTotalOrder.value;
      break;
    default:
      isValidData = true;
      break;
  }
  if (!isValidData) throw new Error('invalid data');
};

const jumpToStep = (step: number) => {
  try {
    if (step >= currentStep.value) {
      validateDataByStep();
    }
    currentStep.value = step;
    isShowError.value = false;
  } catch (error) {
    isShowError.value = true;
  }
};

const nextStep = () => {
  try {
    validateDataByStep();
    currentStep.value++;
    isShowError.value = false;
  } catch (error) {
    isShowError.value = true;
  }
};

const addOrderDish = () => {
  if (totalDishes.value === MAX_TOTAL) return;
  order.dishes.push({
    totalServing: 1,
  } as IDish);
};

const submitOrder = () => {
  try {
    const canSubmit =
      isValidMeal.value &&
      isValidTotalPeople.value &&
      isValidRestaurant.value &&
      isValidDishes.value &&
      isValidTotalOrder;
    if (!canSubmit) throw new Error('invalid data');
    const { dishes, ...rest } = order;
    console.log({
      ...rest,
      dishes: resultDishes.value,
    });
  } catch (error) {
    console.error('error:', error);
  }
};
</script>
<style scoped>
.border-all {
  border: 1px solid #000;
  border-radius: 5px;
}
</style>
