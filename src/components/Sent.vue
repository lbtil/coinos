<template>
  <div>
    <v-card class="mb-2">
      <v-alert class="headline black--text text-center" color="yellow"
        >Sent!</v-alert
      >
      <div class="pa-4">
        <div class="mb-2">
          <div class="d-flex justify-center">
            <div class="mr-2">
              <span class="display-1">{{ $format(total, precision) }}</span>
              {{ ticker }}
            </div>
            <div>
              <span
                v-if="payment.account.ticker === 'BTC'"
                class="yellow--text"
              >
                <span class="display-1">{{ fiat(total) }}</span>
                {{ payment.currency }}
              </span>
            </div>
          </div>
        </div>

        <div class="mb-4 text-center">
          <div class="d-flex justify-center">
            <div class="mr-2">
              <span class="headline grey--text text--lighten-2">+ Fee: </span>
              <span class="display-1">{{ fee }}</span>
              {{ user.unit }}
            </div>
            <div>
              <span
                v-if="payment.account.ticker === 'BTC'"
                class="yellow--text"
              >
                <span class="display-1">{{ fiat(payment.fee) }}</span>
                {{ payment.currency }}
              </span>
            </div>
          </div>
        </div>

        <v-btn @click="clearPayment" class="mr-2">
          <v-icon left>$arrow_back</v-icon><span>Send Another</span>
        </v-btn>
        <v-btn
          v-if="['BTC', 'LBTC'].includes(payment.network)"
          @click.native="explore"
        >
          <v-icon left>$open</v-icon><span>Explore</span>
        </v-btn>
      </div>
    </v-card>
    <v-card v-if="payment.redeemcode" class="mb-2">
      <v-card-text class="title white--text text-center">
        <div class="mb-2">
        Redeem code <span class="yellow--text" @click="redeem" style="cursor: pointer">{{ payment.redeemcode }}</span>
        </div>
        <qr :text="redeemUrl" />
        <v-btn @click="redeem" class="mr-1">
          <v-icon left>$open</v-icon>
          Open
        </v-btn>
        <v-btn @click="copy(redeemUrl)" class="mr-1">
          <v-icon left>$copy</v-icon>
          Copy
        </v-btn>
      </v-card-text>
    </v-card>
  </div>
</template>

<script>
import { get, call } from 'vuex-pathify';
import Copy from '../mixins/Copy';
import Qr from './Qr';

export default {
  components: { Qr },
  mixins: [Copy],
  computed: {
    redeemUrl() {
      return `${window.location.protocol}//${window.location.host}/redeem/${this.payment.redeemcode}`;
    },
    fee() {
      return this.$format(this.payment.fee || 0);
    },
    payment: get('payment'),
    precision() {
      let { precision } = this.payment.account;
      if (this.payment.account.ticker === 'BTC' && this.user.unit === 'SAT')
        precision = 0;
      return precision;
    },
    ticker() {
      let { ticker } = this.payment.account;
      if (ticker === 'BTC') return this.user.unit;
      return ticker;
    },
    total() {
      return Math.abs(this.payment.amount);
    },
    user: get('user'),
  },

  methods: {
    redeem() {
      window.location.href = this.redeemUrl;
    },
    clearPayment: call('clearPayment'),

    fiat(n) {
      if (!n || isNaN(n)) return '0.00';
      return ((n * this.payment.rate) / 100000000).toFixed(2);
    },

    explore() {
      let url = 'https://blockstream.info';
      let { hash, network } = this.payment;
      if (network === 'BTC') url += `/tx/${hash}`;
      if (network === 'LBTC') url += `${bs}/liquid/tx/${hash}`;
      window.open(url, '_blank');
    },

    snack: call('snack'),
  },
};
</script>
