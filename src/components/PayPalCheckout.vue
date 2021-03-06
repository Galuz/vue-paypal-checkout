<template>
  <div :id="id" class="paypal-button"></div>
</template>

<script>
import paypal from 'paypal-checkout';
import defaultProps from '../util/defaultProps';
import additionalProps from '../util/additionalProps';
import { propTypes, assignToPropertyObject } from '../util/paypalProp';

const assignTo = assignToPropertyObject(additionalProps);

export default {
  props: Object.assign(
    defaultProps(),
    additionalProps.vmProps(),
  ),
  methods: {
    payment(resolve, reject) {
      const vue = this;

      const transaction = Object.assign({
        amount: {
          total: this.amount,
          currency: this.currency,
        },
      }, assignTo(vue, propTypes.TRANSACTION));

      const payment = Object.assign({
        transactions: [transaction],
      }, assignTo(vue, propTypes.PAYMENT));

      // Using server to confirm
      if(vue.paymentUrl){
        const CREATE_PAYMENT_URL = vue.paymentUrl;
        paypal.request.post(CREATE_PAYMENT_URL)
        .then(res => {
          resolve(res.id)
        }).catch(err => {
          reject(err)
        })
      }

      // Rest type
      else {
        console.log("Using rest")
        return paypal.rest.payment.create(this.env, this.client, payment);
      }
    },
    onAuthorize(data, actions) {
      const vue = this;
      vue.$emit('paypal-paymentAuthorized', data);
      
      if(!vue.paymentUrl){ // Not using server to confirm
        return actions.payment.execute().then((response) => {
          vue.$emit('paypal-paymentCompleted', response);
        });
      }
    },
    onCancel(data) {
      const vue = this;
      vue.$emit('paypal-paymentCancelled', data);
    },
  },
  mounted() {
    const vue = this;
    const button = Object.assign({
      // Pass in env
      env: vue.env,

      // Pass in the client ids to use to create your transaction
      // on sandbox and production environments
      client: vue.client,

      // Pass the payment details for your transaction
      // See https://developer.paypal.com/docs/api/payments/#payment_create for the expected json parameters
      payment: vue.payment,

      // Display a "Pay Now" button rather than a "Continue" button
      commit: vue.commit,

      // Pass a function to be called when the customer completes the payment
      onAuthorize: vue.onAuthorize,

      // Pass a function to be called when the customer cancels the payment
      onCancel: vue.onCancel,
    }, assignTo(vue, propTypes.BUTTON));

    paypal.Button.render(button, vue.$el);
  },
};
</script>
