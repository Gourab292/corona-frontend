<template>
  <main class="container">
    <div class="flex flex-wrap -mx-2" style="padding-bottom: 96px;">
      <div v-if="pageState === PAGE_STATES.LOADING" class="px-5 text-center">
        Loading...
      </div>
      <div
        v-else-if="pageState === PAGE_STATES.HAS_ERROR"
        class="px-5 text-center"
      >
        {{ error }}
      </div>
      <div v-else-if="pageState === PAGE_STATES.HAS_FETCHED" class="w-full ">
        <div class="flex flex-wrap -mt-2">
          <div class="w-full p-2">
            <survey
              class="my-2"
              :desktop-image="surveyConfig.desktopImage"
              :mobile-image="surveyConfig.mobileImage"
              :link="surveyConfig.link"
              :expires-on="surveyConfig.expiresOn"
            />
          </div>
          <div class="w-full lg:w-1/2 p-2">
            <Overview :info="overviewInfo" :country="country" />
          </div>
          <div class="w-1/2 lg:w-1/4 p-2">
            <FatalityRate
              :days="fatalityRate.days"
              :series="fatalityRate.data"
            />
          </div>
          <div class="w-1/2 lg:w-1/4 p-2">
            <PositiveRate
              :days="positiveRate.days"
              :series="positiveRate.data"
            />
          </div>
        </div>
        <div class="flex flex-wrap">
          <!--        <div class="w-full md:w-1/2 lg:w-1/4 p-2">-->
          <!--          <growth-rate :confirmed="overviewInfo.confirmed"-->
          <!--                       :ytdConfirmed="overviewInfo.diffConfirmed"/>-->
          <!--        </div>-->
          <div class="w-full md:w-1/2 lg:w-1/3 p-2">
            <line-chart-number
              :height="180"
              :data="[0, 10, 20, 10, 40, 20, 50, 60]"
              :title="$t('critical_cases_icu')"
              :subtitle-red="`${criticalCases.inICUCount}%`"
              :subtitle="$t('of_total_cases').toLowerCase()"
              :number="criticalCases.totalCount"
            />
          </div>
          <div class="w-full md:w-1/2 lg:w-1/3 p-2">
            <line-chart-number
              :height="180"
              :data="[0, 10, 20, 10, 40, 20, 50, 60]"
              :title="$t('daily_cases_receiving_treatment')"
              :subtitle-red="`${activeCases.percentage}%`"
              :subtitle="$t('of_total_cases').toLowerCase()"
              :number="activeCases.totalCount"
            />
          </div>
          <div class="w-full md:w-1/2 lg:w-1/3 p-2">
            <line-chart-number
              :height="180"
              :data="[0, 10, 20, 10, 40, 20, 50, 60]"
              :title="$t('daily_confirmed_cases')"
              :number="perMillionConfirmedCases.totalCount"
              :subtitle="$t('per_million_population')"
            />
          </div>
        </div>
        <div class="flex flex-wrap">
          <div class="w-full md:w-3/4 p-2">
            <PastDaysChart
              :height="360"
              :trend-data="countryTrend.trendData"
              :trend-dates="countryTrend.trendDates"
              :title="$t('past_2_weeks_chart')"
              :country="country"
              style="margin-bottom: 12px;"
            />
            <PastDaysChartNewCases
              :height="360"
              :trend-data="countryNewCasesTrend.newCasesTrendData"
              :trend-dates="countryNewCasesTrend.newCasesTrendDates"
              :title="$t('past_new_cases_chart')"
              :country="country"
              style="margin-bottom: 12px;"
            />
            <TrendingNews :country="country" />
          </div>

          <div class="w-full md:w-1/4 p-2">
            <client-only>
              <TwitterFeed :twitter-handle="handle" />
            </client-only>
          </div>
        </div>
      </div>
    </div>
  </main>
</template>

<script>
import FatalityRate from "~/components/Analytics/FatalityRate"
import LineChartNumber from "~/components/Country/LineChartNumber"
import PastDaysChart from "~/components/Country/PastDaysChart"
import PastDaysChartNewCases from "~/components/Country/PastDaysChartNewCases"
import Overview from "~/components/Country/Overview"
import PositiveRate from "~/components/Analytics/PositiveRate"
import Survey from "~/components/Survey"
import TwitterFeed from "~/components/TwitterFeed"
import TrendingNews from "~/components/TrendingNews"
import { COUNTRIES, twitterHandles } from "~/utils/constants"
import covid19AiImage from "~/assets/image/covid19ai.png"

export default {
  components: {
    FatalityRate,
    LineChartNumber,
    PastDaysChart,
    PastDaysChartNewCases,
    Overview,
    PositiveRate,
    Survey,
    TwitterFeed,
    TrendingNews
  },

  data() {
    const PAGE_STATES = {
      LOADING: "LOADING",
      HAS_FETCHED: "HAS_FETCHED",
      HAS_ERROR: "HAS_ERROR"
    }

    return {
      PAGE_STATES,
      pageState: PAGE_STATES.LOADING,
      overviewInfo: {
        confirmed: 0,
        diffConfirmed: 0,
        recovered: 0,
        diffRecovered: 0,
        deaths: 0,
        diffDeaths: 0
      },
      fatalityRate: {
        days: 0,
        data: []
      },
      positiveRate: {
        days: 0,
        data: []
      },
      criticalCases: {
        totalCount: 0,
        inICUCount: 0
      },
      activeCases: {
        totalCount: 0,
        percentage: 0
      },
      perMillionConfirmedCases: {
        totalCount: 0
      },
      countryTrend: {
        trendData: [],
        trendDates: []
      },
      countryNewCasesTrend: {
        newCasesTrendData: [],
        newCasesTrendDates: []
      },
      surveyConfig: {
        desktopImage: covid19AiImage,
        mobileImage: covid19AiImage,
        link: "https://www.facebook.com/events/641115143102661/",
        expiresOn: "2020-04-29"
      }
    }
  },

  computed: {
    country() {
      const countryToFind = this.$route.params.country
      const countryEntry = COUNTRIES.find(country =>
        country.urlAliases.includes(countryToFind)
      )
      return countryEntry || {}
    },
    countryCode() {
      return this.country?.code
    },
    handle() {
      const countryEntry = twitterHandles.find(
        country => this.countryCode === country.code
      )
      return countryEntry?.account || "WHO"
    }
  },

  mounted() {
    this.loadInformation(this.countryCode)
    this.loadCountryTrendData(this.countryCode)
    this.loadCountryNewCasesTrendData(this.countryCode)
  },

  methods: {
    async loadInformation(countryCode) {
      let totalCases

      try {
        totalCases = (
          await this.$api.stats.getCountrySpecificStats(countryCode)
        )?.[0]
      } catch (err) {
        this.pageState = this.PAGE_STATES.HAS_ERROR
        this.error = err.data?.message ?? "Something went wrong."
        return
      }

      this.pageState = this.PAGE_STATES.HAS_FETCHED

      // Total cases information
      this.overviewInfo.confirmed = totalCases.totalConfirmed
      this.overviewInfo.recovered = totalCases.totalRecovered
      this.overviewInfo.deaths = totalCases.totalDeaths

      // Daily change information
      this.overviewInfo.diffConfirmed = totalCases.dailyConfirmed
      this.overviewInfo.diffDeaths = totalCases.dailyDeaths

      // Fatality Rate & Positive Rate
      // Data prep for FR and PR components
      const FRU = Number(totalCases.FR).toFixed(1)
      const PRU = Number(totalCases.PR).toFixed(1)
      const FRL = 100 - FRU
      const PRL = 100 - PRU

      // Fatality Rate
      this.fatalityRate.days = 10
      this.fatalityRate.data = [Number(FRL), Number(FRU)]

      // Positive Rate
      this.positiveRate.days = 10
      this.positiveRate.data = [Number(PRL), Number(PRU)]

      // Critical Cases
      this.criticalCases.totalCount = totalCases.totalCritical
      this.criticalCases.inICUCount = (
        (totalCases.totalCritical / totalCases.totalConfirmed) *
        100
      )?.toFixed(1)

      // Active Cases
      this.activeCases.totalCount = totalCases.activeCases
      this.activeCases.percentage = (
        (totalCases.activeCases / totalCases.totalConfirmed) *
        100
      )?.toFixed(1)

      this.perMillionConfirmedCases.totalCount =
        totalCases.totalConfirmedPerMillionPopulation
    },

    async loadCountryTrendData(countryCode) {
      let countryTrendRaw

      try {
        countryTrendRaw = await this.$api.stats.getTrendByCountryDate(
          countryCode,
          new Date(Date.now() - 13 * 24 * 60 * 60 * 1000)
            .toISOString()
            .slice(0, 10),
          new Date(Date.now() + 1 * 24 * 60 * 60 * 1000)
            .toISOString()
            .slice(0, 10)
        )
      } catch (err) {
        this.pageState = this.PAGE_STATES.HAS_ERROR
        this.error = err.data?.message ?? "Something went wrong."
        return
      }

      // Country Trend
      let countryTrendConfirmed = []
      let countryTrendRecovered = []
      let countryTrendDeath = []
      let confirmedLastMax = 0
      let recoveredLastMax = 0
      let deadLastMax = 0

      countryTrendRaw.forEach(country => {
        confirmedLastMax = Math.max(
          country["total_confirmed"],
          confirmedLastMax
        )
        recoveredLastMax = Math.max(
          country["total_recovered"],
          recoveredLastMax
        )
        deadLastMax = Math.max(country["total_deaths"], deadLastMax)

        countryTrendConfirmed.push(confirmedLastMax)
        countryTrendRecovered.push(recoveredLastMax)
        countryTrendDeath.push(deadLastMax)

        this.countryTrend.trendDates.push(country["last_updated"].slice(0, 10))
      })
      this.countryTrend.trendData = [
        {
          name: this.$t("confirmed"),
          data: countryTrendConfirmed
        },
        {
          name: this.$t("recovered"),
          data: countryTrendRecovered
        },
        {
          name: this.$t("death"),
          data: countryTrendDeath
        }
      ]
    },

    async loadCountryNewCasesTrendData(countryCode) {
      let countryNewCasesTrendRaw

      try {
        countryNewCasesTrendRaw = await this.$api.stats.getNewCasesByCountryDate(
          countryCode,
          // start date
          new Date(Date.now() - 60 * 24 * 60 * 60 * 1000)
            .toISOString()
            .slice(0, 10),
          // end date
          new Date(Date.now() + 1 * 24 * 60 * 60 * 1000)
            .toISOString()
            .slice(0, 10)
        )
      } catch (err) {
        this.pageState = this.PAGE_STATES.HAS_ERROR
        this.error = err.data?.message ?? "Something went wrong."
        return
      }

      // Country Trend
      let countryNewCasesTrendConfirmed = []
      let countryNewCasesTrendRecovered = []
      let countryNewCasesTrendDeath = []

      countryNewCasesTrendRaw.forEach(country => {
        countryNewCasesTrendConfirmed.push(
          country["new_infections"] > 0 ? country["new_infections"] : 0
        )
        countryNewCasesTrendRecovered.push(
          country["new_recovered"] > 0 ? country["new_recovered"] : 0
        )
        countryNewCasesTrendDeath.push(
          country["new_deaths"] > 0 ? country["new_deaths"] : 0
        )
        this.countryNewCasesTrend.newCasesTrendDates.push(
          country["last_updated"].slice(0, 10)
        )
      })
      this.countryNewCasesTrend.newCasesTrendData = [
        {
          name: this.$t("confirmed"),
          data: countryNewCasesTrendConfirmed
        },
        {
          name: this.$t("recovered"),
          data: countryNewCasesTrendRecovered
        },
        {
          name: this.$t("death"),
          data: countryNewCasesTrendDeath
        }
      ]
    }
  },
  head() {
    const country = this.country && this.country.name
    const countryCode = this.countryCode && this.countryCode.toLowerCase()

    const title = this.$t("covid_corona_tracker_country", { country })
    const description = this.$t("covid_corona_tracker_country_description", {
      country
    })

    const baseUrl = process.env.BASE_URL || "https://www.coronatracker.com"
    const countryStatsType =
      this.$route.query.referrer === "recent"
        ? "countryStatsRecent"
        : "countryStatsToday"
    const imageUrl = `${
      process.env.BASE_URL
    }/.netlify/functions/take-screenshot?type=${countryStatsType}&countryCode=${countryCode}&t=${Date.now()}`
    const pageUrl = process.browser
      ? window.location.href
      : `${baseUrl}/country/${this.$route.params.country}/`

    return {
      title,
      titleTemplate: "%s",
      meta: [
        { hid: "title", name: "title", content: title },
        { hid: "description", name: "description", content: description },
        { hid: "og-title", property: "og:title", content: title },
        {
          hid: "og-description",
          property: "og:description",
          content: description
        },
        { hid: "og-url", property: "og:url", content: pageUrl },
        { hid: "og-image", property: "og:image", content: imageUrl },
        {
          hid: "og-image-width",
          property: "og:image:width",
          content: countryStatsType === "countryStatsRecent" ? "984" : "720"
        },
        {
          hid: "og-image-height",
          property: "og:image:height",
          content: countryStatsType === "countryStatsRecent" ? "515" : "375"
        },
        {
          hid: "twitter-card",
          property: "twitter:card",
          content: "summary_large_card"
        },
        { hid: "twitter-url", property: "twitter:url", content: pageUrl },
        { hid: "twitter-title", property: "twitter:title", content: title },
        {
          hid: "twitter-description",
          property: "twitter:description",
          content: description
        },
        { hid: "twitter-image", property: "twitter:image", content: imageUrl }
      ]
    }
  },

  metaInfo: {
    title: "Country Specific Analytics"
  }
}
</script>
