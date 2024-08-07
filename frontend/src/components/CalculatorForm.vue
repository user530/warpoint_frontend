<script setup lang="ts">
import type { ITariff, IExchangeData } from '@/shared/types';
import { computed, onMounted, ref, watch } from 'vue';
import { CalculatorFieldTariff, CalculatorFieldCurrency, CalculatorFieldPeriod, CalculatorFieldDisplayTotal, SpinnerDefault } from '@/components';
import { tickerFromCurrency, getDiscountForOption, fetchCalculatorData } from '@/shared/utils';
import { Currency } from '@/shared/enums';

interface IFormValues {
    tariffId: number | null;
    currency: keyof typeof Currency | null;
    paymentOptionId: number | null;
}

const tariffs = ref<ITariff[]>([]);
const exchangeRates = ref<IExchangeData[]>([]);

const isLoading = ref<boolean>(true);

const formValues = ref<IFormValues>({
    tariffId: null,
    currency: null,
    paymentOptionId: null,
});

// Props and handler for the tariff selection field
const tariffFieldProps = computed(() => tariffs.value.map(tariff => ({ id: tariff.id, name: tariff.name })));
const tariffSelectionHandler = (newTariffId: number) => {
    formValues.value.tariffId = newTariffId;
};

const selectedTariff = computed(() => {
    return tariffs.value.find((tariff) => tariff.id === formValues.value.tariffId) ?? null;
});

// Props and handler for the currency selection field 
const currencyProps = computed(() => {
    return selectedTariff.value?.baseCurrency ?? Currency.RUB;
});
const currencySelectionHandler = (newCurrency: keyof typeof Currency) => {
    formValues.value.currency = newCurrency;
};

const periodFieldProps = computed(() => {
    return selectedTariff.value && selectedTariff.value.paymentOptions
        ? selectedTariff.value.paymentOptions.map(option => ({ id: option.id, paymentPeriod: option.paymentPeriod }))
        : [];
}
);

const periodSelectionHandler = (newPeriodId: number | null) => {
    formValues.value.paymentOptionId = newPeriodId;
}

const exchangeRate = computed(
    () => {
        if (!selectedTariff.value || !formValues.value.currency || !formValues.value.paymentOptionId) return 0;
        const baseCurrency = tickerFromCurrency(selectedTariff.value.baseCurrency);
        const rates = exchangeRates.value.find((rate) => rate.base_code === baseCurrency);
        if (!baseCurrency || !rates || !rates.conversion_rates) return 0;
        const { currency } = formValues.value;
        const rate = rates.conversion_rates[currency];
        return rate;
    }
);

const totalAmount = computed(() => {
    if (!selectedTariff.value || !formValues.value || !formValues.value.paymentOptionId) return 0;
    const selectedOption = selectedTariff.value.paymentOptions.find((option) => option.id === formValues.value.paymentOptionId);
    return selectedOption ? +(selectedOption.basePrice * exchangeRate.value).toFixed(2) : 0;
});

const discount = computed(
    () => {
        const { paymentOptionId, currency, tariffId } = formValues.value;
        if (!selectedTariff.value || !paymentOptionId || !currency || !tariffId) return 0;
        const disc = getDiscountForOption(selectedTariff.value.paymentOptions, paymentOptionId);
        return disc ? +disc.toFixed(2) : 0;
    }
);

const amountCurrency = computed(
    () => {
        const selectedCurrency = formValues.value.currency;
        return selectedCurrency ? ` ${selectedCurrency}` : '';
    }
)

// Reset form values on the tariff selection
watch(
    () => formValues.value.tariffId,
    (newTariffId: number | null) => {
        const selectedTariff = tariffs.value.find((tariff) => tariff.id == newTariffId);
        if (!selectedTariff) return;
        const { baseCurrency, paymentOptions } = selectedTariff;
        const selectedCurrency = tickerFromCurrency(baseCurrency);
        const firstPeriodId = paymentOptions && Array.isArray(paymentOptions)
            ? paymentOptions[0].id
            : null;
        formValues.value.currency = !selectedCurrency ? null : selectedCurrency;
        formValues.value.paymentOptionId = !firstPeriodId ? null : firstPeriodId;
    }
);

// Data fetching function
const fetchData = async () => {
    try {
        const {
            tariffs: fetchedTariffs,
            exchangeRates: fetchedRates
        } = await fetchCalculatorData();

        tariffs.value = fetchedTariffs;
        exchangeRates.value = fetchedRates;
    } catch (error) {
        console.error(error);
    } finally {
        isLoading.value = false;
    }
}

// Fetch data on load
onMounted(fetchData)

const canSubmit = computed(() => formValues.value.tariffId && formValues.value.currency && formValues.value.paymentOptionId)
const onSubmit = (e: Event) => {
    e.preventDefault();
    alert('Данные формы отправлены (консоль)!');
    console.log(JSON.parse(JSON.stringify(formValues.value)));
};
</script>

<template>
    <SpinnerDefault v-if="isLoading" />

    <form v-else class="calculator-form" @submit="onSubmit">
        <CalculatorFieldTariff :tariffs="tariffFieldProps" v-model="formValues.tariffId" label="Тариф:"
            @tariff-selected="tariffSelectionHandler" />
        <CalculatorFieldCurrency :base-currency="currencyProps" v-model="formValues.currency" label="Валюта:"
            @currency-selected="currencySelectionHandler" />
        <CalculatorFieldPeriod :periods="periodFieldProps" v-model="formValues.paymentOptionId" label="Период оплаты:"
            @period-selected="periodSelectionHandler" />

        <CalculatorFieldDisplayTotal :total="totalAmount" :discount="discount * exchangeRate"
            :currency="amountCurrency" />

        <button type="submit" :disabled="!canSubmit">Перейти к оплате</button>
    </form>
</template>

<style>
@import '../assets/variables.css';

form.calculator-form {
    width: min(600px, 95%);
    display: flex;
    flex-direction: column;
    background-color: var(--color-white);
    padding: 60px;
    border-radius: 16px;
    box-shadow: 0 0 15px rgba(0, 0, 0, 0.1);

    @media (max-width: 1199px) {
        padding: 45px;
    }

    @media (max-width: 767px) {
        padding: 15px;
        border-radius: 8px;
    }

    input,
    select {
        display: block;
        width: 100%;
        height: 44px;
        padding-right: 16px;
        font-size: 20px;
        background-color: var(--color-white);
        border: none;
        border-bottom: 1px solid var(--color-background);
        outline: none;

        &:focus {
            outline: none;
            border-bottom-color: var(--color-primary-dark);
        }

        &:disabled {
            color: var(--color-primary-dark);
            font-weight: 700;
        }

        option {
            background-color: inherit;
        }

        @media (max-width: 1199px) {
            font-size: 18px;
        }

        @media (max-width: 767px) {
            font-size: 16px;
            height: 36px;
        }

    }

    label {
        display: block;
        margin-bottom: 8px;
        font-size: 18px;
        color: var(--color-on-primary);

        @media (max-width: 1199px) {
            font-size: 16px;
            margin-bottom: 4px;
        }

        @media (max-width: 767px) {
            font-size: 14px;
        }
    }

    div+div {
        margin-top: 30px;

        @media (max-width: 1199px) {
            margin-top: 16px;
        }
    }

    button {
        display: inline-block;
        padding-inline: 70px;
        margin-top: 30px;
        margin-inline: auto;
        height: 60px;
        border: 2px solid var(--color-primary);
        border-radius: 8px;
        background-color: var(--color-primary);
        color: var(--color-white);
        font-size: 20px;

        transition: 0.3s;

        &:hover {
            background-color: var(--color-primary-dark);
        }

        &:active {
            background-color: var(--color-primary);
        }

        @media (max-width: 767px) {
            height: 40px;
            font-size: 16px;
            padding-inline: 50px;
        }
    }
}
</style>