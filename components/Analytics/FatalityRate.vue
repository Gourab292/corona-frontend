<template>
  <div
    class="bg-white rounded border border-gray-400 p-1 w-full sm: w-1/2 h-full"
  >
    <div class="flex flex-wrap">
      <div class="w-full sm:w-1/2">
        <client-only placeholder="Loading...">
          <apexcharts
            ref="chart"
            type="donut"
            width="100%"
            height="150px"
            :options="options"
            :series="series"
          />
        </client-only>
      </div>
      <div
        class="w-full sm:w-1/2 flex flex-col py-2 pl-2 justify-center text-center sm:text-left"
      >
        <strong class="mb-2">{{ $t("fatality_rate") }}</strong>
        <!-- <small class="mb-2">{{ $t('time_from_confirmation_to_discharge') }}</small> -->
        <!-- <strong>{{ $t('number_of_days', { number: days }) }}</strong> -->
      </div>
    </div>
  </div>
</template>

<style scoped></style>

<script>
export default {
  name: "FatalityRate",
  props: {
    days: {
      type: Number,
      default: 0
    },
    series: {
      type: Array,
      required: true
    }
  },
  data: function() {
    return {
      options: {
        chart: {
          type: "donut"
        },
        legend: {
          show: false
        },
        fill: {
          colors: ["#CCCCCC", "#FF9AB2"]
        },
        tooltip: {
          enabled: false
        },
        dataLabels: {
          enabled: false
        },
        states: {
          hover: {
            filter: {
              type: "none"
            }
          },
          active: {
            allowMultipleDataPointsSelection: false,
            filter: {
              type: "none"
            }
          }
        },
        plotOptions: {
          pie: {
            customScale: 0.9,
            expandOnClick: false,
            donut: {
              size: "85%",
              labels: {
                show: true,
                name: {
                  offsetY: 20,
                  color: "#828282",
                  formatter: () => {
                    return this.$t("of_total_cases")
                  }
                },
                value: {
                  offsetY: -15,
                  fontSize: "25px",
                  color: "#000000",
                  formatter: function(a, all) {
                    return all.config.series[1] + "%"
                  }
                },
                total: {
                  show: true,
                  label: this.$t("of_total_cases"),
                  fontSize: "9px",
                  formatter: function(value) {
                    return value.config.series[1] + "%"
                  }
                }
              }
            }
          }
        }
      }
    }
  }
}
</script>
