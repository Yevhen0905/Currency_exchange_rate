<template>
  <main class="main">
    <h2 class="title">Currency exchange rate</h2>
    <div class="container">
      <div class="container_one">
        <select
          class="select"
          name="first_currency"
          id="first_currency"
          v-model="currencyOne"
        >
          <option v-for="currency in listCurrency" :value="currency" :key="currency">
            {{ currency }}
          </option>
        </select>
        <input
          type="number"
          name="input_one"
          id="input_one"
          v-model="amountOne"
          @change="fetchData"
        />
      </div>
      <div class="switch_currency">
        <button class="btn" @click="switchValue">switch</button>
        <div v-if="currencyData.conversion_rates" class="calculation_rate">
          1 {{ currencyOne }} = {{ rate }} {{ currencyTwo }}
        </div>
      </div>
      <div class="container_two">
        <select
          class="select"
          name="second_currency"
          id="second_currency"
          v-model="currencyTwo"
        >
          <option v-for="currency in listCurrency" :value="currency" :key="currency">
            {{ currency }}
          </option>
        </select>
        <input
          type="number"
          name="input_two"
          id="input_two"
          placeholder="0"
          v-model="amountTwo"
          disabled
        />
      </div>
      <div class="date" v-if="currencyData.time_last_update_utc">
        Lastly updated: {{ convertDate }}
      </div>
    </div>
  </main>
</template>

<script setup>
  import {ref, computed, onMounted, watch} from 'vue';

  const currencyData = ref([]);
  const currencyOne = ref('USD');
  const currencyTwo = ref('EUR');
  const rate = ref(null);
  const amountOne = ref(1);
  const amountTwo = ref(0);
  const listCurrency = ref(['USD', 'EUR', 'UAH', 'PLN', 'GBR', 'CZK']);

  const convertDate = computed(() => {
    const dateStr = currencyData.value.time_last_update_utc;
    const date = new Date(dateStr);
    const day = date.getUTCDate().toString().padStart(2, '0');
    const month = (date.getUTCMonth() + 1).toString().padStart(2, '0');
    const year = date.getUTCFullYear().toString();

    return `${day} ${month} ${year}`;
  });

  const fetchData = async () => {
    try {
      const res = await fetch(
        `https://v6.exchangerate-api.com/v6/7b64677fae4fd9ab618c23ee/latest/${currencyOne.value}`
      );
      const data = await res.json();
      currencyData.value = data;
      rate.value = data?.conversion_rates[currencyTwo.value].toFixed(2);
      amountTwo.value = amountOne.value * rate.value;
    } catch (e) {
      console.log(e);
    }
  };

  const switchValue = () => {
    const temporary = currencyOne.value;
    currencyOne.value = currencyTwo.value;
    currencyTwo.value = temporary;
    fetchData();
  };

  watch(currencyTwo, () => {
    fetchData();
  });

  onMounted(fetchData);
</script>

<style lang="scss">
  .main {
    display: flex;
    flex-direction: column;
    align-items: center;
  }

  .title {
    font-size: 36px;
    margin-bottom: 15px;
  }

  .container {
    display: flex;
    flex-direction: column;
    align-items: start;
  }

  .container_one,
  .container_two {
    display: flex;
    gap: 10px;
  }

  .select {
    width: 120px;
    height: 30px;
    border-radius: 5px;
    text-align: center;
  }

  input {
    width: 120px;
    border-radius: 5px;
    outline: none;

    &[disabled] {
      background-color: antiquewhite;
    }
  }

  .btn {
    margin: 10px 0;
    outline: none;
    border: none;
    padding: 5px 44px;
    border-radius: 5px;
    background: #2b5de2;
    color: white;
    font-size: 18px;
  }

  h4 {
    margin-bottom: 10px;
  }

  .switch_currency {
    display: flex;
    align-items: center;
    gap: 15px;
    margin: 15px 0;
  }

  .date {
    margin-top: 15px;
  }
</style>
