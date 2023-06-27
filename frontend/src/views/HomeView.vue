<template>
    <main class="content">
        <form action="#" method="post">
            <div class="content__wrapper">
                <h1 class="title title--big">Конструктор пиццы</h1>
                <!-- тесто -->
                <dough-selection v-model="pizza.dough" :dough-types="doughTypes"/>
                <!-- диаметр -->
                <diameter-selection v-model="pizza.size" :size-types="sizeTypes"/>
                <div class="content__ingredients">
                    <div class="sheet">
                        <h2 class="title title--small sheet__title">Выберите ингредиенты</h2>
                        <div class="sheet__content ingredients">
                            <!-- соус -->
                            <sauce-selection v-model="pizza.sauce" :sauce-types="sauceTypes"/>
                            <!-- ингредиенты -->
                            <ingredients-selection
                                :values="pizza.ingredients"
                                :ingredient-types="ingredientTypes"
                                @update="updateIngredientAmount"
                            />
                        </div>
                    </div>
                </div>
                <div class="content__pizza">
                    <label class="input">
                        <span class="visually-hidden">Название пиццы</span>
                        <input
                            v-model="pizza.name"
                            type="text"
                            name="pizza_name"
                            placeholder="Введите название пиццы"
                        />
                    </label>
                    <!-- пицца -->
                    <pizza-constructor
                        :dough="pizza.dough"
                        :sauce="pizza.sauce"
                        :ingredients="pizza.ingredients"
                        @drop="addIngredient"
                    />
                    <div class="content__result">
                        <p>Итого: {{ price }} ₽</p>
                        <button type="button" class="button" :disabled="disableSubmit">
                            Готовьте!
                        </button>
                    </div>
                </div>
            </div>
        </form>
    </main>
</template>

<script setup>
import {computed, reactive} from "vue";
import {normalizeDough, normalizeIngredients, normalizeSauces, normalizeSize,} from "@/common/helpers/normalize";

import doughJSON from "@/mocks/dough.json";
import ingredientsJSON from "@/mocks/ingredients.json";
import saucesJSON from "@/mocks/sauces.json";
import sizesJSON from "@/mocks/sizes.json";

import DoughSelection from "@/modules/constructor/DoughSelection.vue";
import DiameterSelection from "@/modules/constructor/DiameterSelection.vue";
import SauceSelection from "@/modules/constructor/SauceSelection.vue";
import IngredientsSelection from "@/modules/constructor/IngredientsSelection.vue";
import PizzaConstructor from "@/modules/constructor/PizzaConstructor.vue";

const doughTypes = doughJSON.map(normalizeDough);
const sizeTypes = sizesJSON.map(normalizeSize);
const sauceTypes = saucesJSON.map(normalizeSauces);
const ingredientTypes = ingredientsJSON.map(normalizeIngredients);

const pizza = reactive({
    name: "",
    dough: doughTypes[0].value,
    size: sizeTypes[0].value,
    sauce: sauceTypes[0].value,
    ingredients: ingredientTypes.reduce((acc, item) => {
        acc[item.value] = 0;

        return acc;
    }, {}),
});

const price = computed(() => {
    const { dough, size, sauce, ingredients } = pizza;

    const sizeMultiplier =
        sizeTypes.find((item) => item.value === size)?.multiplier ?? 1;

    const doughPrice =
        doughTypes.find((item) => item.value === dough)?.price ?? 0;

    const saucePrice =
        sauceTypes.find((item) => item.value === sauce)?.price ?? 0;

    /*
    * Здесь мы при помощи метода map превращаем массив ингредиентов
    * в массив значений, соответствующих итоговой стоимости каждого из них — просто умножаем цену на количество.
    * После чего методом reduce вычисляем сумму всех элементов массива. Это даёт нам общую стоимость ингредиентов.
    */
    const ingredientsPrice = ingredientTypes
        .map((item) => ingredients[item.value] * item.price)
        .reduce((acc, item) => acc + item, 0);

    return (doughPrice + saucePrice + ingredientsPrice) * sizeMultiplier;
});

const disableSubmit = computed(() => {
    return pizza.name.length === 0 || price.value === 0;
});
const addIngredient = (ingredient) => {
    pizza.ingredients[ingredient]++;
};
const updateIngredientAmount = (ingredient, count) => {
    pizza.ingredients[ingredient] = count;
};
</script>

<style lang="scss">
@import "@/assets/scss/ds-system/ds.scss";
@import "@/assets/scss/mixins/mixins.scss";

.content__ingredients {
    width: 527px;
    margin-top: 15px;
    margin-right: auto;
    margin-bottom: 15px;
}

.content__pizza {
    width: 373px;
    margin-top: 15px;
    margin-bottom: 15px;
}

.content__result {
    display: flex;
    align-items: center;
    justify-content: center;

    margin-top: 25px;

    p {
        @include b-s24-h28;

        margin: 0;
    }

    button {
        margin-left: 12px;
        padding: 16px 45px;
    }
}
</style>
