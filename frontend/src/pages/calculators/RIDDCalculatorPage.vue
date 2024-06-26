<script lang="ts">
import { ref, type Ref } from 'vue'
import { useRIDDCalculator } from '@/hooks/calculators/useRIDDCalculator'
import { useCalculator } from '@/hooks/calculators/useCalculator'
import axios from 'axios'

export default {
    setup(props, ctx) {
        let volume: Ref<string> = ref('')
        let time: Ref<string> = ref('')
        let result: Ref<string> = ref('')
        // let resultInDrops: Ref<String> = ref('')
        let isError: Ref<boolean> = ref(false)
        let errorMessage: Ref<string> = ref('')

        const { calculatorInfo, isCalculatorLoading } = useCalculator('rate-intravenous-drip-drug')

        const calculate_result = async () => {
            try {
                const { getResult } = useRIDDCalculator(Number(volume.value), Number(time.value))
                result.value = String((await getResult())?.result)
                clearForm()
                isError.value = false
            } catch (error: any) {
                if (axios.isAxiosError(error)) {
                    isError.value = true
                    if (error.response && error.response.data && error.response.data.violations) {
                        errorMessage.value = error.response.data.violations
                            .map((violation: { message: string }) => violation.message)
                            .join(', ')
                    }
                } else if (error instanceof Error) {
                    errorMessage.value = error.message
                } else {
                    errorMessage.value = 'An unknown error occurred'
                }
            }
        }

        const clearForm = () => {
            volume.value = ''
            time.value = ''
        }

        return {
            volume,
            time,
            result,
            calculate_result,
            clearForm,
            calculatorInfo,
            isCalculatorLoading,
            isError,
            errorMessage
        }
    }
}
</script>
<template>
    <div class="content border border-1 border-secondary rounded-3 p-3 mx-5 my-3">
        <h1 class="text-center">
            Калькулятор для расчет скорости внутривенного капельного введения препарата
        </h1>
        <div class="content row d-flex justify-content-between">
            <div class="content w-75 row">
                <div class="col-6 px-5 py-2 mb-3">
                    <form @submit.prevent>
                        <base-input
                            labelText="Объем раствора, мл"
                            inputType="number"
                            inputPlaceholder="Введите объем раствора:"
                            v-model="volume"
                            class="mt-4 mb-4"
                        />
                        <base-input
                            labelText="Желаемое время введения, часов"
                            inputType="number"
                            inputPlaceholder="Введите желаемое время введения:"
                            v-model="time"
                            class="mt-4 mb-4"
                        />
                        <div class="d-flex justify-content-center">
                            <base-button @click="calculate_result" class="mx-1"
                                >Рассчитать</base-button
                            >
                            <base-button @click="clearForm" class="mx-1">Сбросить</base-button>
                        </div>
                    </form>
                </div>
                <div class="content col-6 row my-auto p-3 w-50 border bg-light bg-gradient">
                    <div v-if="!isError" class="col-12 my-auto border">
                        Результат вычисления: {{ result }} капель в минуту
                    </div>
                    <div v-else class="col-12 my-auto border">
                        <span class="text-danger">Произошла ошибка!</span>
                    </div>
                    <div v-if="!isError" class="col-12 border mt-3">
                        А также:
                        {{
                            isNaN(parseFloat((parseFloat(result) / 60).toFixed(2)))
                                ? ''
                                : (parseFloat(result) / 60).toFixed(2)
                        }}
                        капель в секунду
                    </div>
                    <div v-else class="col-12 border mt-3">
                        Возможное решение: {{ errorMessage }}
                    </div>
                </div>
            </div>
        </div>

        <div class="content px-3">
            <div>
                <h2>Об калькуляторе</h2>
                <p>{{ calculatorInfo?.info.split('.')[0] + '.' }}</p>
            </div>
            <div>
                <h2>Формула</h2>
                <img
                    src="../../static/image/calculators/rate_intravenous_drip_drug/formula.png"
                    width=""
                    alt="Формула"
                    class="d-inline-block align-middle mr-2"
                />
                <p class="mt-2">
                    Расчет производится по следующей формуле: количество капель в минуту = V*20/t,
                    где V - объем раствора в милилитрах, t - время в минутах, 20 - среднее
                    количество капель в милилитре, v - скорость введения в каплях в минуту
                </p>
            </div>
        </div>
    </div>
</template>
