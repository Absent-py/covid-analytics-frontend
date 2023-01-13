<template>
  <section class="section">
    <v-container theme="dark">
      <v-card width="100%" align="center" theme="dark">
        <v-container>
          <h1>Covid Analytics</h1>
          <v-container align="start">
            <h3>Preview data</h3>
          </v-container>
          <v-row>
            <v-col>
              <v-select
                  v-model="selectedTable"
                  :items="tables"
                  label="TABLE"
              ></v-select>
              <v-text-field
                  v-model="limit"
                  label="LIMIT"
              ></v-text-field>
            </v-col>
            <v-col>
              <v-textarea
                  filled
                  :v-model="setQuery()"
                  :placeholder="query"
              ></v-textarea>
            </v-col>
          </v-row>
          <v-btn @click="fetchPreview">
            <h3>Preview</h3>
          </v-btn>
        </v-container>
      </v-card>
    </v-container>
  </section>
  <section class="section">
    <v-container v-if="response !== ''">
      <v-table theme="dark" density="compact">
        <thead>
        <tr>
          <th
              v-for="header in headers()"
          >{{ header.text }}</th>
        </tr>
        </thead>
        <tbody>
        <tr
            v-for="row in response"
        >
          <td
              v-for="col in row"
          >{{ col }}</td>
        </tr>
        </tbody>
      </v-table>
    </v-container>
  </section>
  <section class="section">
    <v-btn @click="getChartData">Death statistics</v-btn>
    <BarChart :chartData="chartDataDeath"></BarChart>
    <BarChart :chartData="chartDataDeathDiff"></BarChart>
  </section>
</template>

<script>
import axios from "axios";
import {BarChart} from 'vue-chart-3'
import {Chart, registerables} from "chart.js"

Chart.register(...registerables)

export default {
  name: 'Home',
  components: {
    BarChart,
  },
  data: () => ({
    renderComponent: true,
    chartDataDeath: {
      labels: Array,
      datasets: [{
        data: Array
      }]
    },
    chartDataDeathDiff: {
      labels: Array,
      datasets: [{
        data: Array
      }]
    },
    selectedTable: 'Activity',
    tables: [
      'Cases',
      'Activity',
      'Country',
      'Resist',
    ],
    limit: 5,
    query: '',
    response: '',
    headersResist: [
      {text: 'Region', value: 'region'},
      {text: 'Country', value: 'country'},
      {text: 'ISO', value: 'iso'},
      {text: 'ISO 1366', value: 'iso_3166'},
      {text: 'prov_category', value: 'prov_category'},
      {text: 'prov_measure', value: 'prov_measure'},
      {text: 'who_category', value: 'who_category'},
      {text: 'who_measure', value: 'who_measure'},
      {text: 'Targeted', value: 'targeted'},
      {text: 'Non compliance', value: 'non_compliance'},
      {text: 'Source type', value: 'source_type'},
      {text: 'Date', value: 'date'}
    ],
    headersCases: [
      {text: 'Case type', value: 'case_type'},
      {text: 'Total tested', value: 'total_tested'},
      {text: 'Cases', value: 'cases'},
      {text: 'Difference', value: 'difference'},
      {text: 'Date', value: 'date'},
      {text: 'Key', value: 'combined_key'},
      {text: 'Region', value: 'country_region'},
      {text: 'State', value: 'province_state'},
      {text: 'Admin', value: 'admin'},
      {text: 'ISO 2', value: 'iso2'},
      {text: 'ISO 3', value: 'iso3'},
      {text: 'Fips', value: 'fips'},
      {text: 'Lat', value: 'lat'},
      {text: 'Lon', value: 'lon'},
      {text: 'Population', value: 'population'},
      {text: 'Hospitalized', value: 'hospitalized'},
      {text: 'Source', value: 'source'},
      {text: 'Runtime', value: 'runtime'}
    ],
    headersCountry: [
      {text: 'Date', value: 'date'},
      {text: 'Location', value: 'location'},
      {text: 'New cases', value: 'cases_new'},
      {text: 'New deaths', value: 'deaths_new'},
      {text: 'Total cases', value: 'cases_total'},
      {text: 'Total deaths', value: 'deaths_total'},
      {text: 'Weekly cases', value: 'cases_weekly'},
      {text: 'Weekly deaths', value: 'deaths_weekly'},
      {text: 'Biweekly cases', value: 'cases_biweekly'},
      {text: 'Biweekly deaths', value: 'deaths_biweekly'}
    ],
    headersActivity: [
      {text: 'Positive', value: 'positive_cases'},
      {text: 'Country', value: 'country'},
      {text: 'State', value: 'province_state'},
      {text: 'Date', value: 'date'},
      {text: 'Continent', value: 'continent'},
      {text: 'Source', value: 'source'},
      {text: 'Death', value: 'death'},
      {text: 'FIPS', value: 'fips'},
      {text: 'ISO 3', value: 'iso3'},
      {text: 'Country short', value: 'country_short'},
      {text: 'ISO 2', value: 'iso2'},
      {text: 'New positive', value: 'positive_cases_new'},
      {text: 'Death count', value: 'death_count'}
    ],
  }),
  methods: {
    setQuery() {
      let query = 'SELECT * FROM "' + this.selectedTable + '" LIMIT ' + this.limit
      this.query = query
      return query
    },
    headers() {
      if (this.query.includes('Cases')) {
        return this.headersCases
      }
      if (this.query.includes('Resist')) {
        return this.headersResist
      }
      if (this.query.includes('Activity')) {
        return this.headersActivity
      }
      if (this.query.includes('Country')) {
        return this.headersCountry
      }
    },
    parseResponse(response, headers) {
      let object = []
      let line = {}
      for (let item of response) {
        let field = 0
        if (headers.length === 18) {
          line = {
            [headers[field].value]: item[field],
            [headers[field + 1].value]: item[field + 1],
            [headers[field + 2].value]: item[field + 2],
            [headers[field + 3].value]: item[field + 3],
            [headers[field + 4].value]: item[field + 4],
            [headers[field + 5].value]: item[field + 5],
            [headers[field + 6].value]: item[field + 6],
            [headers[field + 7].value]: item[field + 7],
            [headers[field + 8].value]: item[field + 8],
            [headers[field + 9].value]: item[field + 9],
            [headers[field + 10].value]: item[field + 10],
            [headers[field + 11].value]: item[field + 11],
            [headers[field + 12].value]: item[field + 12],
            [headers[field + 13].value]: item[field + 13],
            [headers[field + 14].value]: item[field + 14],
            [headers[field + 15].value]: item[field + 15],
            [headers[field + 16].value]: item[field + 16],
            [headers[field + 17].value]: item[field + 17]
          }
        }
        if (headers.length === 13) {
          line = {
            [headers[field].value]: item[field],
            [headers[field + 1].value]: item[field + 1],
            [headers[field + 2].value]: item[field + 2],
            [headers[field + 3].value]: item[field + 3],
            [headers[field + 4].value]: item[field + 4],
            [headers[field + 5].value]: item[field + 5],
            [headers[field + 6].value]: item[field + 6],
            [headers[field + 7].value]: item[field + 7],
            [headers[field + 8].value]: item[field + 8],
            [headers[field + 9].value]: item[field + 9],
            [headers[field + 10].value]: item[field + 10],
            [headers[field + 11].value]: item[field + 11],
            [headers[field + 12].value]: item[field + 12]
          }
        }
        if (headers.length === 12) {
          line = {
            [headers[field].value]: item[field],
            [headers[field + 1].value]: item[field + 1],
            [headers[field + 2].value]: item[field + 2],
            [headers[field + 3].value]: item[field + 3],
            [headers[field + 4].value]: item[field + 4],
            [headers[field + 5].value]: item[field + 5],
            [headers[field + 6].value]: item[field + 6],
            [headers[field + 7].value]: item[field + 7],
            [headers[field + 8].value]: item[field + 8],
            [headers[field + 9].value]: item[field + 9],
            [headers[field + 10].value]: item[field + 10],
            [headers[field + 11].value]: item[field + 11]
          }
        }
        if (headers.length === 10) {
          line = {
            [headers[field].value]: item[field],
            [headers[field + 1].value]: item[field + 1],
            [headers[field + 2].value]: item[field + 2],
            [headers[field + 3].value]: item[field + 3],
            [headers[field + 4].value]: item[field + 4],
            [headers[field + 5].value]: item[field + 5],
            [headers[field + 6].value]: item[field + 6],
            [headers[field + 7].value]: item[field + 7],
            [headers[field + 8].value]: item[field + 8],
            [headers[field + 9].value]: item[field + 9]
          }
        }
        object.push(line)
      }
      return object
    },
    async fetchPreview() {
      const response = await axios.post('http://127.0.0.1:5000/execute',
          {'query': this.query},
          {
            headers: {
              'content-type': 'application/json'
            }
          })
      let parsed
      if (this.query.includes('Resist')) {
        parsed = this.parseResponse(response.data.data, this.headersResist)
      }
      if (this.query.includes('Cases')) {
        parsed = this.parseResponse(response.data.data, this.headersCases)
      }
      if (this.query.includes('Country')) {
        parsed = this.parseResponse(response.data.data, this.headersCountry)
      }
      if (this.query.includes('Activity')) {
        parsed = this.parseResponse(response.data.data, this.headersActivity)
      }
      this.response = parsed
    },
    async fetchDeath() {
      const response = await axios.get('http://127.0.0.1:5000/death')
      return response.data
    },
    async fetchDeathDiff() {
      const response = await axios.get('http://127.0.0.1:5000/death/diff')
      return response.data
    },
    async getChartData() {
      console.log('working')
      let response = await this.fetchDeath()
      let chartData = {
        labels: response,
        datasets: [{
          data: response
        }]
      }
      this.chartDataDeath = chartData

      response = await this.fetchDeathDiff()
      chartData = {
        labels: response,
        datasets: [{
          data: response
        }]
      }
      this.chartDataDeathDiff = chartData
    },
    async forceRerender() {
      this.renderComponent = false;
      await this.$nextTick();
      this.renderComponent = true;
    }
  }
}
</script>

<style>
.section{
  margin-bottom: 50px;
}
</style>
