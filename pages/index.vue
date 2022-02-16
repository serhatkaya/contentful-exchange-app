<template>
  <div>
    <div class="navbar">
      <select id="refreshOffset" v-model="refreshEvery" name="refreshEvery">
        <option value="5">Refresh every 5 seconds</option>
        <option value="10">Refresh every 10 seconds</option>
        <option value="15">Refresh every 15 seconds</option>
      </select>
    </div>
    <div class="wrapper">
      <div
        v-for="currency in entries"
        :id="currency.id"
        :key="currency.id"
        class="card"
      >
        <div class="card-line">
          <div class="card-image" style="--card-color-rgb: 247, 147, 26">
            <img
              :alt="currency.universalShortening"
              :src="currency.icon"
              class="card-image__image"
            />
          </div>

          <div class="card-name">
            <h4 class="card-name__title">{{ currency.name }}</h4>
            <span class="card-name__short">
              {{ currency.universalShortening }}
            </span>
          </div>
        </div>
        <div class="card-line">
          <div class="card-details__cell">
            <span class="card-tag"> TRY </span>
          </div>
          <strong class="card-value"> {{ currency.value }} </strong>
          <div class="card-details">
            <!-- row -->
            <div class="card-details__cell">
              <span class="card-tag"> USD </span>
            </div>
            <div class="card-details__cell">
              <span class="card-microvalue is-green card-change">
                {{ currency.usd }}
              </span>
            </div>
            <!-- row -->
            <div class="card-details__cell">
              <span class="card-tag"> EUR </span>
            </div>
            <div class="card-details__cell">
              <span class="card-microvalue card-volume">
                {{ currency.eur }}
              </span>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import * as gql from 'gql-query-builder';
import { createClient } from '~/plugins/contentful.js';
const client = createClient();

export default {
  name: 'IndexPage',
  asyncData({ env }) {
    return Promise.all([
      // fetch all blog posts sorted by creation date
      client.getEntries(
        gql.query({
          operation: 'moneyUnitCollection',
          fields: [
            {
              items: [
                {
                  sys: ['id'],
                },
                'universalShortening',
              ],
            },
          ],
        }).query
      ),
    ])
      .then(([r]) => {
        return {
          entries: [
            ...r.items.map((entry) => {
              return {
                icon: entry.fields.icon.fields.file.url.replace(
                  '//',
                  'https://'
                ),
                universalShortening: entry.fields.universalShortening,
                name: entry.fields.name,
                id: entry.sys.id,
                value: 0,
                usd: 0,
                eur: 0,
              };
            }),
          ],
        };
      })
      .catch(console.error);
  },
  data() {
    return {
      entries: [],
      refreshEvery: 5,
      lastRefresh: 0,
    };
  },
  created() {
    this.setRefreshInterval();
  },
  methods: {
    setRefreshInterval() {
      const self = this;
      setInterval(async () => {
        if (
          (new Date().getTime() - this.lastRefresh) / 1000 >=
          this.refreshEvery
        ) {
          await self.fetchCurrencies(self);
          this.lastRefresh = new Date().getTime();
        }
      }, 1000);
    },
    fetchCurrencies: async (self) => {
      self.entries = await self.updateCurrencies(self.entries, self);
    },
    updateCurrencies: async (entries, self) => {
      for (let i = 0; i < entries.length; i++) {
        const res = (entries[i].value = await self.getCurrencyValue(
          entries[i].universalShortening
        ));
        entries[i].value = res.value;
        entries[i].usd = res.usd;
        entries[i].eur = res.eur;
      }
      return entries;
    },
    getCurrencyValue(currency) {
      return new Promise((resolve, reject) => {
        try {
          this.$axios
            .$get(`https://open.er-api.com/v6/latest/${currency}`)
            .then((r) => {
              resolve({
                value: r.rates.TRY,
                usd: r.rates.USD,
                eur: r.rates.EUR,
              });
            });
        } catch (error) {
          reject(error);
        }
      });
    },
  },
};
</script>
<style>
body {
  margin: 0;
  font-family: 'Space Grotesk', sans-serif;
  background-color: #111827;
}

* {
  box-sizing: border-box;
}

.wrapper {
  padding: 24px;
  display: flex;
  flex-direction: row;
  flex-wrap: wrap;
  justify-content: center;
  align-items: center;
}

.card {
  display: flex;
  flex-direction: column;
  gap: 24px;
  width: 380px;
  position: relative;
  border-radius: 20px;
  padding: 26px 28px;
  color: #f9fafb;
  background-color: #000;
  box-shadow: inset 0 0 0 12px #18212f;
  transition: background-color 0.2s;
  cursor: pointer;
}
.card:hover {
  background-color: #110e19;
}
.card:hover::before,
.card:hover::after {
  transform: scaleX(1);
}
.card:hover .card-line:first-of-type {
  transform: translateY(5px);
}
.card:hover .card-line:last-of-type {
  transform: translateY(-5px);
}
.card::before,
.card::after {
  content: '';
  position: absolute;
  left: 28px;
  right: 28px;
  z-index: 3;
  display: flex;
  border-radius: 1px;
  height: 2px;
  background: #a78bfa;
  box-shadow: 0px -18px 80px rgba(167, 139, 250, 0.77),
    0px -9.00879px 40.0391px rgba(167, 139, 250, 0.585456),
    0px -5.42647px 24.1177px rgba(167, 139, 250, 0.501723),
    0px -3.47757px 15.4559px rgba(167, 139, 250, 0.439589),
    0px -2.25388px 10.0172px rgba(167, 139, 250, 0.385),
    0px -1.41879px 6.30574px rgba(167, 139, 250, 0.330411),
    0px -0.815184px 3.62304px rgba(167, 139, 250, 0.268277),
    0px -0.358784px 1.59459px rgba(167, 139, 250, 0.184544);
  transition: transform 0.2s ease-in-out;
  transform: scaleX(0);
  transform-origin: center;
}
.card::before {
  top: 12px;
}
.card::after {
  bottom: 12px;
}

.card-line {
  display: flex;
  align-items: center;
  transition: transform 0.3s;
}

.card-image {
  width: 38px;
  height: 38px;
  border-radius: 100%;
  margin-right: 16px;
  overflow: hidden;
  box-shadow: 0px 54px 80px rgba(var(--card-color-rgb, 255, 255, 255), 0.14),
    0px 27px 40px rgba(var(--card-color-rgb, 255, 255, 255), 0.1),
    0px 16px 24px rgba(var(--card-color-rgb, 255, 255, 255), 0.09),
    0px 10px 15px rgba(var(--card-color-rgb, 255, 255, 255), 0.08),
    0px 7px 10px rgba(var(--card-color-rgb, 255, 255, 255), 0.07),
    0px 4px 6px rgba(var(--card-color-rgb, 255, 255, 255), 0.06),
    0px 2px 3px rgba(var(--card-color-rgb, 255, 255, 255), 0.05),
    0px 1px 2px rgba(var(--card-color-rgb, 255, 255, 255), 0.03);
}
.card-image__image {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.card-name {
  display: flex;
  flex-direction: column;
}
.card-name__title {
  font-size: 18px;
  font-weight: normal;
  line-height: 23px;
  margin: 0;
}
.card-name__short {
  font-size: 12px;
  line-height: 15px;
  color: #9ca3af;
}

.card-value {
  font-weight: 600;
  font-size: 26px;
  line-height: 33px;
  margin-right: auto;
}

.card-details {
  display: grid;
  grid-template-columns: auto 1fr;
  gap: 1px;
  align-items: center;
}
.card-details__cell:nth-child(2n) {
  min-width: 70px;
  text-align: right;
}

.card-tag {
  border-radius: 8px;
  padding: 0 4px;
  font-size: 12px;
  line-height: 15px;
  text-transform: uppercase;
  color: #9ca3af;
  background-color: #374151;
}

.card-microvalue {
  font-size: 14px;
  line-height: 18px;
}
.card-microvalue.is-green {
  color: #10b981;
}
.card-microvalue.is-red {
  color: #b71c1c;
}

.navbar {
  display: flex;
  justify-content: flex-end;
  padding: 1rem 1rem 0 0;
}
</style>
