<template>
  <section>
    <h1 class="header">{{ mortgageCalcTitle }} ${{ payment.toFixed(2) }}</h1>
    <p class="sanity-check">** Sanity check: 200k at 8% = $1468</p>
  </section>
  <section>
    <h1>{{ inputHeader }}</h1>
    <div class="input-content" v-for="item in userInputs">
      <label>{{ item.label }}</label>
      <input :type="(item.type)" v-model="item.amount">
    </div>
  </section>
  <section>
    <h1>{{ resultsHeader }}</h1>
    <div class="input-content" v-for="item in results">
      <label>{{ item.label }}</label>
      <div>{{ USDollar.format(item.amount) }}</div>
    </div>
  </section>
  <p>{{ notes }}</p>
</template>

<script setup>
import { computed, reactive, watchEffect } from 'vue'

let USDollar = new Intl.NumberFormat('en-US', {
  style: 'currency',
  currency: 'USD',
  maximumSignificantDigits: 3,
});

// ----- BRRRR Inputs Data Object ----- //
const inputHeader = 'BRRRR Inputs'
const userInputs = reactive({
  initialCash: { type: 'input', label: 'Initial Investment', amount: '200000' },
  annualPercentageRate: { type: 'input', label: 'APR (decimal)', amount: '0.08' },
  yearsOfMortgageDuration: { type: 'input', label: 'Mortgage Term (years)', amount: '30' },
  yearsFromInitialPurchase: { type: 'input', label: 'Hold Duration (years)', amount: '7' },
  acquisitionFrequencyInMonths: { type: 'input', label: 'Aquisition Frequency (months)', amount: '6' },
  reInvest: { type: 'checkbox', label: 'Re-Invest?', amount: true },
  percentToReinvest: { type: 'input', label: 'Percent To Reinvest', amount: '0.9' },
  cashCushion: { type: 'checkbox', label: 'Cash Cushion?', amount: false },
})

// ----- Mortgage Calculator ----- //
const mortgageCalcTitle = 'Mortgage Payment:'
const startingBalance = computed(() => parseFloat(userInputs.initialCash.amount))
const rate = computed(() => parseFloat(userInputs.annualPercentageRate.amount)/12)
const totalPeriods = computed(() => parseFloat(userInputs.yearsOfMortgageDuration.amount)*12)
const payment = computed(() => (startingBalance.value * rate.value) / (1 - Math.pow((1 + rate.value), (-1 * totalPeriods.value)))) // Math.pos uses logarithm so can be inaccurate when squaring

// ----- BRRRR Results Data Object ----- //
const resultsHeader = 'BRRRR Results'
let results = reactive({
  offerPrice: { type: 'input', label: 'Target Offer:', amount: 0 },
  refurbBudget: { type: 'input', label: 'Refurb Budget:', amount: 0 },
  closingCosts: { type: 'input', label: 'Closing Costs:', amount: 0 },
  outTheDoorTotal: { type: 'input', label: 'Purchase Price incl Closing Costs:', amount: 0 },
  afterRefurbValue: { type: 'input', label: 'After Refurb Value:', amount: 0 },
  valueAdd: { type: 'input', label: 'Value Add:', amount: 0 },
  cashCushionAmount: { type: 'input', label: 'Total Cash Cushion:', amount: 0 },
  mortgagePmt: { type: 'input', label: 'Mortgage Payment:', amount: 0 },
  acceptableRentRange: { type: 'input', label: 'Rent Range Needed:', amount: 0 },
  currentRentRange: { type: 'input', label: 'Nearby Rent Range:', amount: 0 },
  totalMonthlyCashFlow: { type: 'input', label: 'Total Monthly Cash Flow:', amount: 0 },
  remainingMonthlyCashFlow: { type: 'input', label: 'Fun Money Cash Flow:', amount: 0 },
  totalEquityWithAppreciation: { type: 'input', label: 'Total Equity with 1% Appreciation:', amount: 0 },
  totalLoanBalance: { type: 'input', label: 'Total Loan Balance:', amount: 0 },
})

// ----- BRRRR Calculator ----- //
const arv = computed(() => (startingBalance.value*0.3)+startingBalance.value)
const refurb = computed(() => (arv.value-startingBalance.value) / 0.7)
const targetOffer = computed(() => startingBalance.value-(refurb.value+refurb.value*0.3))
const closing = computed(() => (targetOffer.value) * 0.035)
const offerTotal = computed(() => (targetOffer.value + closing.value))
const moneyLeftInDeal = computed(() => startingBalance.value-offerTotal.value-refurb.value) // extra ARV added to net worth if Assessment hits target. "Value Add" on page

// ----- Set BRRRR Results ----- //
watchEffect(() => {
  results.afterRefurbValue.amount = arv.value
  results.refurbBudget.amount = refurb.value
  results.offerPrice.amount = targetOffer.value
  results.closingCosts.amount = closing.value
  results.outTheDoorTotal.amount = offerTotal.value
  results.valueAdd.amount = moneyLeftInDeal.value
})

// ----- Notes ----- //
const notes = `
Notes:
This estimator is currently toggled towards producing conservative, or reasonable estimates.
It will be enhanced to show moderate and aggressive estimates.

Since we are working backward from an available lump sum, we need to figure out how much:
1. House we can afford (including closing costs)
2. Renovation we can afford.

First we'll determine the target ARV we'd need in order to pull our initial investment back out of the property.
Since conservative banks finance 70% of a home's ARV, we'll need to consider our initial investment 70% of ARV, which means adding 30% to our
initial cash sum to determine ARV. When calculating ARV, there's a 30% loss in the refurbishment process which will be factored into the offer price.
ARV = initial value + (refurb cost * 0.7). So then, Refurb cost = (ARV - initial value) / 0.7.
Closing Costs are typically 2-5% of sale price. Conservatively using the median value, which is 3.5%

Results are rounded to 3 maximum significant digits, so consider them ballpark.
`


/* TODO




*/

</script>

<style lang="scss">@import "./index.scss"</style>