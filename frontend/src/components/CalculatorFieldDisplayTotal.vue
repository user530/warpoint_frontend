<script setup lang="ts">

interface IDisplayTotalProps {
    total: number;
    discount: number;
    currency: string;
    labelTotal?: string;
    labelDiscount?: string;
    fieldIdTotal?: string;
    fieldIdDiscount?: string;
}

const { total, discount, labelTotal, labelDiscount, fieldIdTotal, fieldIdDiscount } = defineProps<IDisplayTotalProps>();

</script>

<template>
    <div class="display-total">
        <div>
            <label :for="fieldIdTotal ?? 'total-field'">
                {{ labelTotal ?? 'Сумма к оплате:' }}
            </label>
            <input :id="fieldIdTotal ?? 'discount-field'" type="text" :value="total + currency" disabled>
        </div>

        <div v-if="discount > 0">
            <label :for="fieldIdDiscount ?? 'discount-field'">
                {{ labelDiscount ?? 'Сумма скидки:' }}
            </label>
            <input :id="fieldIdDiscount ?? 'discount-field'" type="text" :value="discount + currency" disabled>
        </div>
    </div>
</template>

<style scoped>
.display-total {
    min-height: 170px;

    @media (max-width: 1199px) {
        min-height: 144px;
    }

    @media (max-width: 767px) {
        min-height: 124px;
    }
}

input {
    text-align: right;
}
</style>