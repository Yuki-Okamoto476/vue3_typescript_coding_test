<template>
  <div class="pref">
    <h1>コーディングテスト</h1>
    <h2>都道府県</h2>
    <div class="pref__list">
      <div
        v-for="(prefItem, index) in prefItems"
        :key="index"
        class="pref__item"
      >
        <input
          :id="`pref-${index}`"
          type="checkbox"
          name="pref"
          @change="createPopulationGraph(prefItem.prefCode, $event)"
        />
        <label :for="`pref-${index}`">{{ prefItem.prefName }}</label>
      </div>
    </div>
    <highcharts :options="chartOptions"></highcharts>
  </div>
</template>

<script lang="ts">
import axios from 'axios';
import { Chart } from 'highcharts-vue';
import { defineComponent, onMounted, ref } from 'vue';

interface PrefItems {
  prefCode: number;
  prefName: string;
}

interface ChartSeries {
  data: number;
  name: string;
}

export default defineComponent({
  name: 'App',
  components: {
    highcharts: Chart,
  },
  setup() {
    const config = {
      headers: { 'X-API-KEY': process.env.VUE_APP_API_KEY },
      timeout: 30000,
      params: {
        cache: new Date().getTime(),
      },
    };

    const prefItems = ref<PrefItems[]>([]);
    const getPrefData = () => {
      const REQUEST_URL =
        'https://opendata.resas-portal.go.jp/api/v1/prefectures';
      axios
        .get(REQUEST_URL, config)
        .then((response) => {
          prefItems.value = response.data.result;
        })
        .catch(() => {
          alert('データの取得に失敗しました');
        });
    };
    onMounted(getPrefData);

    const chartOptions = ref<any>({
      series: [],
      xAxis: {
        categories: [
          1960, 1965, 1970, 1975, 1980, 1985, 1990, 1995, 2000, 2005, 2010,
          2015, 2020, 2025, 2030, 2035, 2040, 2045,
        ],
        title: {
          text: '年',
        },
      },
      yAxis: {
        title: {
          text: '人口数',
        },
      },
      title: {
        text: '都道府県別人口推移',
      },
      credits: {
        enabled: false,
      },
    });
    const createPopulationGraph = (value: number, $event: Event) => {
      const matchedPref = prefItems.value.find(
        (item) => item.prefCode === value
      );
      if (($event.target as HTMLInputElement).checked) {
        const REQUEST_URL = `https://opendata.resas-portal.go.jp/api/v1/population/composition/perYear?prefCode=${value}`;
        axios
          .get(REQUEST_URL, config)
          .then((response) => {
            const result = response.data.result.data[0].data;
            const resultData = [] as Array<number>;
            for (let i = 0; i < result.length; i++) {
              resultData.push(result[i].value);
            }
            chartOptions.value.series.push({
              data: resultData,
              name: matchedPref?.prefName,
            });
          })
          .catch(() => {
            alert('データの取得に失敗しました');
          });
      } else {
        const filteredOption = chartOptions.value.series.filter(
          (el: ChartSeries) => el.name === matchedPref?.prefName
        );
        chartOptions.value.series = chartOptions.value.series.filter(
          (el: ChartSeries) => {
            return !filteredOption.includes(el);
          }
        );
      }
    };

    return {
      prefItems,
      chartOptions,
      createPopulationGraph,
    };
  },
});
</script>

<style lang="scss" scoped>
.pref__list {
  margin-bottom: 40px;
}

.pref__item {
  display: inline-block;
}
</style>
