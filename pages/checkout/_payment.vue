<template>
  <div>
    <div id="payment-page">
      <div class="forms">
        <div class="form-shopper-data">
          <h2> Your Details </h2>
          <form>
            <label for="fname">First name:</label>
            <label for="lname">Last name:</label><br>
            <input type="text" id="fname" name="fname">
            <input type="text" id="lname" name="lname"><br>
            <label for="femail">Email:</label>
            <label for="fphone">Phone:</label><br>
            <input type="text" id="femail" name="femail">
            <input type="text" id="fphone" name="fphone"><br>
            <button type='button' @click="setFormShopperData()">submit</button>
          </form>
        </div>
        <div class="form-shipping-data collapsed">
          <h2> Shipping Address </h2>
          <form>
            <label for="fstreet">Street:</label>
            <label for="fpostcode">Postcode</label><br>
            <input type="text" id="fstreet" name="fstreet">
            <input type="text" id="fpostcode" name="fpostcode"><br>
            <label for="fcity">City:</label>
            <label for="fregion">Region</label><br>
            <input type="text" id="fcity" name="fcity">
            <input type="text" id="fregion" name="fregion"><br>
            <label for="fcountry">Country</label>
            <label for="samebilling" id="samebilling-label">Same as Billing</label>
            <input type="checkbox" id="samebilling" name="samebilling"><br>
            <input type="text" id="fcountry" name="fcountry"><br>
            <button type='button' @click="setFormShippingAddress()">submit</button>
          </form>
        </div>
        <div class="form-billing-data collapsed">
          <h2> Billing Address </h2>
          <form>
            <label for="fbstreet">Street:</label>
            <label for="fbpostcode">Postcode</label><br>
            <input type="text" id="fbstreet" name="fstreet">
            <input type="text" id="fbpostcode" name="fpostcode"><br>
            <label for="fbcity">City:</label>
            <label for="fbregion">Region</label><br>
            <input type="text" id="fbcity" name="fbcity">
            <input type="text" id="fbregion" name="fbregion"><br>
            <label for="fbcountry">Country</label>
            <input type="text" id="fbcountry" name="fbcountry"><br>
            <button type='button' @click="setFormBillingAddress()">submit</button>
          </form>
        </div>
        <div class="shipping-method-selector">
          <h2> Shipping Method Address </h2>
          <div class="shipping-method-container" v-for="(meth, index) in this.shippingMethods" :key="index">
            <input type="radio" :id="'smethod-' + index" name="smethod" @change="onCheckBoxChange($event)">
            <label for="smethod"> {{meth.carrier_title}} - {{meth.method_title}}: + {{meth.amount.value}} {{meth.amount.currency}} </label><br>
          </div>
        </div>
      </div>
      <div class="container">
        <div class="payment-container">
          <div class="container-wrapper" v-for="(pm, index) in this.paymentMethods":key="index">
            <div class="pm-header" v-if="pm.type!='scheme'">
              <input
                type="radio"
                class="pm-radio"
                :id="`radio-${index}`"
                name="radiopm"
                :value="pm.type"
                @change="onSelectPaymentMethod($event)"
                v-model="selectedpm"
              >
              <img
                v-if="pm.icon"
                :src="pm.icon.url"
              >
              <label
                v-if="paymentMethodsResponse.paymentMethods[index]"
                class="pm-label"
                for="radiopm"
              >
                Pay with  {{paymentMethodsResponse.paymentMethods.filter(m => m.type == pm.type ).length > 0 ? paymentMethodsResponse.paymentMethods.filter(m => m.type == pm.type )[0].name : pm.type}}
              </label>
            </div>
            <div class="component" v-if="pm.type!='scheme'" :id="`${pm.type}-container`" v-show="selectedpm === `${pm.type}`"></div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
let AdyenCheckout;
import '@adyen/adyen-web/dist/adyen.css';

if (process.client) {
  AdyenCheckout = require("@adyen/adyen-web");
}

export default {
  components: {},
  props: {

  },
  data() {
    return {
      url: "",
      bearer: "",
      clientKey: "test_Y6ET72GBOBFGXFVJCPAHJTQU4MVGZDSR",
      cartId: '',
      redirectResult: '',
      checkout: '',
      selectedpm: '',
      paymentMethods: [],
      shippingMethods: [],
      adyenStatusResponse: '',
      paymentMethodsResponse: '',
      orderId:'',
      stateData:'',
      shopperBillingAddress: {
        firstName: '',
        lastName: '',
        street: '',
        city: '',
        country: '',
        region: '',
        postcode: '',
        country_code: '',
        phone: '',
      },
      shopperShippingAddress: {
        firstName: '',
        lastName: '',
        street: '',
        city: '',
        country: '',
        region: '',
        postcode: '',
        country_code: '',
        phone: '',
      },
    }
  },
  head() {
    return {
      title: "Payment page",
    };
  },

  asyncData({ route }) {
    return { type: route.params.payment };
  },
  created() {

  },
  async mounted() {
    const urlParams = new URLSearchParams(window.location.search);
    this.redirectResult = urlParams.get('redirectResult');

    this.storage()
  },

  methods: {
    storage() {
      this.url = localStorage.getItem('url');
      this.bearer = localStorage.getItem('bearer');
      this.cartId = localStorage.getItem('cart');
    },

    //// HANDLERS
    // Changing ShippingMethod needs a refresh on PM list using new amount and cart info
    async onCheckBoxChange(event) {
      let method = this.shippingMethods[event.target.id.substring(event.target.id.indexOf('-') + 1)];
      let response = await this.setShippingMethod(method);

      await this.getPaymentMethods();
    },

    // Refreshing config when selecting new payment method from list
    onSelectPaymentMethod(event) {
      let method = event.target;
      this.createConfig();
    },

    //// FORMS
    // Save ShopperData form locally and set guest email
    async setFormShopperData() {
      let firstName = document.getElementById('fname').value;
      let lastName = document.getElementById('lname').value;
      let email = document.getElementById('femail').value;
      let phone = document.getElementById('fphone').value;

      let response = await this.addGuestToCart(email);

      this.shopperShippingAddress.firstName = firstName;
      this.shopperShippingAddress.lastName = lastName;
      this.shopperShippingAddress.phone = phone;

      this.shopperBillingAddress.firstName = firstName;
      this.shopperBillingAddress.lastName = lastName;
      this.shopperBillingAddress.phone = phone;

      if(response.data.setGuestEmailOnCart.cart){
        document.getElementsByClassName("form-shopper-data")[0].classList.add("collapsed");
        document.getElementsByClassName("form-shipping-data")[0].classList.remove("collapsed");
      }

    },

    // Save ShippingAddress form locally and set it on cart
    async setFormShippingAddress() {
      this.shopperShippingAddress.street =  document.getElementById('fstreet').value;
      this.shopperShippingAddress.postcode = document.getElementById('fpostcode').value;
      this.shopperShippingAddress.city = document.getElementById('fcity').value;
      this.shopperShippingAddress.region = document.getElementById('fregion').value;
      this.shopperShippingAddress.country_code = document.getElementById('fcountry').value;

      let response = await this.setShippingAdress();

      if(response.data.setShippingAddressesOnCart.cart){
        document.getElementsByClassName("form-shipping-data")[0].classList.add("collapsed");
        if(document.getElementById('samebilling').checked) {
          this.shopperBillingAddress.street =  document.getElementById('fstreet').value;
          this.shopperBillingAddress.postcode = document.getElementById('fpostcode').value;
          this.shopperBillingAddress.city = document.getElementById('fcity').value;
          this.shopperBillingAddress.region = document.getElementById('fregion').value;
          this.shopperBillingAddress.country_code = document.getElementById('fcountry').value;

          let response = await this.setBillingAddress();

          if(response.data.setBillingAddressOnCart.cart) {
            document.getElementsByClassName("form-billing-data")[0].classList.add("collapsed");
          }
        }
        else {
          document.getElementsByClassName("form-billing-data")[0].classList.remove("collapsed");
        }
      }
    },

    // Save ShippingAddress form locally and set it on cart
    async setFormBillingAddress() {
      this.shopperBillingAddress.street =  document.getElementById('fbstreet').value;
      this.shopperBillingAddress.postcode = document.getElementById('fbpostcode').value;
      this.shopperBillingAddress.city = document.getElementById('fbcity').value;
      this.shopperBillingAddress.region = document.getElementById('fbregion').value;
      this.shopperBillingAddress.country_code = document.getElementById('fbcountry').value;

      let response = await this.setBillingAddress();

      if(response.data.setBillingAddressOnCart.cart){
        document.getElementsByClassName("form-billing-data")[0].classList.add("collapsed");
      }
    },

    // Create checkout config and mount checkout components
    async createConfig() {

      let configuration = {
        environment: 'test',
        clientKey: this.clientKey,
        countryCode: this.shopperBillingAddress.country_code,
        paymentMethodsResponse: this.paymentMethodsResponse,
        onPaymentCompleted: (result, component) => {
          console.info(result, component);
        },
        onChange: this.handleOnChange,
        onAdditionalDetails: await this.adyenDetails,
        onError: (error, component) => {
          console.error(error.name, error.message, error.stack, component);
        },
        onSubmit: this.placeOrder,
        paymentMethodsConfiguration: {

        }
      };

      const pmExclude = ['alipay', 'unionpay', 'applepay', 'c_cash', 'wechatpayQR', 'genericgiftcard', 'givex', 'bankTransfer_NL', 'ratepay', 'paypal', 'giftcard', 'eps', 'sepadirectdebit', 'multibanco'];
      this.paymentMethods = this.paymentMethods.filter((pm, index) => !pmExclude.includes(pm.type));
      this.paymentMethodsResponse.paymentMethods = this.paymentMethodsResponse.paymentMethods.filter((pm, index) => !pmExclude.includes(pm.type));
      console.log(this.paymentMethods);
      console.log(this.paymentMethodsResponse.paymentMethods);
      let configs = this.paymentMethods.map(pm => {
        if (pm.configuration) {
          configuration['paymentMethodsConfiguration'][pm.type] = pm.configuration;
          configuration['paymentMethodsConfiguration'][pm.type]['showPayButton'] = true;
        }
      })

      const checkout = await AdyenCheckout(configuration);
      this.checkout = checkout;

      // Mount config into each container
      this.paymentMethods.map((pm, index) => pm.type != 'scheme' ? checkout.create(pm.type, configuration).mount('#' + pm.type + '-container') : null);

    },

    // Function for components onchange listener (not used atm but can be used to show changes in state data
    handleOnChange(state, component) {
      this.stateData = state.data;
    },

    handleDetails(state, component){
      let response = this.adyenDetails();
    },

    processResult(paymentStatus){
      localStorage.setItem('orderNumber', this.orderId);
      localStorage.setItem('resultCode', paymentStatus.resultCode);
      switch (paymentStatus.resultCode) {
        case "Authorised":
          window.location.href = window.location.origin + '/result/success';
          break;
        case "Pending":
        case "Received":
          window.location.href = window.location.origin + '/result/pending';
          break;
        case "Refused":
          window.location.href = window.location.origin + '/result/failed';
          break;
        default:
          window.location.href = window.location.origin + '/result/error';
          break;
      }
    },

    async handlePaymentError(error){
      console.error(error);
      alert("Payment was REFUSED");
      await this.getPaymentMethods();

    },

    // Query logic to place an order for an adyen payment
    async placeOrder(state, component) {
      try {
        const cartId = this.cartId;
        const stateData = JSON.stringify(state.data);
        let data = "";

        if (state.data.paymentMethod.type === "scheme") {
          data = JSON.stringify({
            query: `mutation setPaymentMethod($cartId: String! $stateData: String!) { setPaymentMethodOnCart( input: { cart_id: $cartId payment_method: { code:`
              + '"' + "adyen_cc" + '"'
              + `, adyen_additional_data_cc: { cc_type:`
              + '"' + state.data.paymentMethod.brand + '"'
              + `, stateData: $stateData}}}) {cart { selected_payment_method { code title } }} placeOrder( input: { cart_id: $cartId }) { order { order_id adyen_payment_status { isFinal resultCode additionalData action}}}}`,
            variables: {cartId: cartId, stateData: stateData },
          });
        } else {
          let brand = state.data.paymentMethod.type;
          data = JSON.stringify({
            query: `mutation setPaymentMethod($cartId: String! $stateData: String!) { setPaymentMethodOnCart( input: { cart_id: $cartId payment_method: { code:`
              + '"' + "adyen_hpp" + '"'
              + `, adyen_additional_data_hpp: { brand_code:`
              + '"' + brand + '"'
              + `, stateData: $stateData}}}) {cart { selected_payment_method { code title } }} placeOrder( input: { cart_id: $cartId }) { order { order_id adyen_payment_status { isFinal resultCode additionalData action}}}}`,
            variables: {cartId: cartId, stateData: stateData },
          });
        }

        const response = await this.sendGraphQLReq(data);
        this.orderId = response.data.placeOrder.order.order_id;
        let paymentStatus = response.data.placeOrder.order.adyen_payment_status;

        if (!paymentStatus.isFinal) {
          console.log(paymentStatus.action);
          let pmtype = state.data.paymentMethod.type === "scheme" ? "card" : state.data.paymentMethod.type;
          this.checkout.createFromAction(JSON.parse(paymentStatus.action)).mount('#' + pmtype  + '-container');
        } else {
          alert(paymentStatus.resultCode);
          this.processResult(paymentStatus);
        }
        return response;

      } catch (error) {
        this.handlePaymentError(error);
      }
    },

    // Query logic to get the adyen details of the transaction
    async adyenDetails(state, component) {
      try {
        const cartId = this.cartId;
        let orderId = this.orderId;

        let payload = state.data;
        payload.orderId = orderId;
        payload = JSON.stringify(payload);

        const data = JSON.stringify({
          query: `mutation getAdyenPaymentDetails($payload: String!, $cartId: String!) {adyenPaymentDetails(payload: $payload, cart_id: $cartId) {isFinal resultCode additionalData action}}`,
          variables: {cartId: cartId, payload: payload },
        });

        const response = await this.sendGraphQLReq(data);

        alert(response.data.adyenPaymentDetails.resultCode);
        this.processResult(response.data.adyenPaymentDetails);
        // { "adyenPaymentDetails": { "isFinal": true, "resultCode": "Authorised", "additionalData": null, "action": null } }

        return response;

      } catch (error) {
        this.handlePaymentError(error);
      }
    },

    // Query logic to get the current payment status
    async adyenStatus() {
      try {
        const cartId = this.cartId;
        const orderId = this.orderId;

        const data = JSON.stringify({
          query: `query getAdyenPaymentStatus($orderNumber: String!, $cartId: String!) { adyenPaymentStatus(orderNumber: $orderNumber, cartId: $cartId) { isFinal resultCode additionalData action}}`,
          variables: {cartId: cartId, orderNumber: orderId },
        });

        const response = await this.sendGraphQLReq(data);
        this.adyenStatusResponse = response.data.adyenPaymentStatus.resultCode;
        return response;

      } catch (error) {
        this.handlePaymentError(error);
      }
    },

    // Query logic to set guest email on Cart
    async addGuestToCart(shopperEmail) {
      try {
        const cartId = this.cartId;

        const data = JSON.stringify({
          query:  'mutation{setGuestEmailOnCart( input: { cart_id: '
            + '"' + cartId + '"'
            + ' email: '+ '"' + shopperEmail
            + '"' + ' }) {cart { email }}}',
        });

        const response = await this.sendGraphQLReq(data);

        return response;

      } catch (error) {
        console.error(error);
        alert("Error occurred. Look at console for details");
      }
    },

    // Query logic to set shipping address on Cart
    async setShippingAdress() {
      try {
        const cartId = this.cartId;

        const data = JSON.stringify({
          query: `mutation{setShippingAddressesOnCart(input: {cart_id:` + '"'
            + cartId + '"'
            + `, shipping_addresses: [{address: {firstname:`
            + '"' + this.shopperShippingAddress.firstName + '"'
            + `, lastname:` + '"'
            + this.shopperShippingAddress.lastName + '"'
            + `, company:`
            + '"' + "Magento"
            + '"'
            + `, street:[`
            + '"' + this.shopperShippingAddress.street + '"'
            + `], city:`
            + '"' + this.shopperShippingAddress.city + '"'
            + `, region:`
            + '"' + this.shopperShippingAddress.region + '"'
            + `, postcode:`
            + '"' + this.shopperShippingAddress.postcode + '"'
            + `, country_code:`
            + '"' + this.shopperShippingAddress.country_code + '"'
            + `, telephone:`
            + '"' + this.shopperShippingAddress.phone + '"'
            + `, save_in_address_book: false}}]}) {cart {shipping_addresses {firstname lastname company street city region {code label} postcode telephone available_shipping_methods { available amount {value currency } carrier_code carrier_title error_message method_code method_title } country { code label }}}}}`,
        });

        const response = await this.sendGraphQLReq(data);
        this.shippingMethods = response.data.setShippingAddressesOnCart.cart.shipping_addresses[0].available_shipping_methods;

        return response;

      } catch (error) {
        console.error(error);
        alert("Error occurred. Look at console for details");
      }
    },

    // Query logic to set billing address on Cart
    async setBillingAddress() {
      try {
        const cartId = this.cartId;

        const data = JSON.stringify({
          query: 'mutation{setBillingAddressOnCart(input: {cart_id:'
            + '"' + cartId + '"' +
            ', billing_address: {address: {firstname: '
            + '"' + this.shopperBillingAddress.firstName + '"'
            + ', lastname: '
            + '"' + this.shopperBillingAddress.lastName + '"'
            + ', company: "Magento" , street: ['
            + '"' + this.shopperBillingAddress.street + '"'
            + '], city:'
            + '"' + this.shopperBillingAddress.city + '"'
            + ', region:'
            + '"' + this.shopperBillingAddress.region + '"'
            + ', postcode:'
            + '"' + this.shopperBillingAddress.postcode + '"'
            + ', country_code:'
            + '"' + this.shopperBillingAddress.country_code + '"'
            + ', telephone:'
            + '"' + this.shopperBillingAddress.phone + '"'
            + ', save_in_address_book: false }, same_as_shipping: true }}) {cart {billing_address {firstname lastname company street city region { code label} postcode telephone country { code label }}} }}',
        });

        const response = await this.sendGraphQLReq(data);
        return response;

      } catch (error) {
        console.error(error);
        alert("Error occurred. Look at console for details");
      }
    },

    // Query logic to set shipping method on Cart
    async setShippingMethod(method) {
      try {
        const cartId = this.cartId;

        //set shippingmethod
        const data = JSON.stringify({
          query: `mutation {setShippingMethodsOnCart( input: { cart_id: `
            + `"` + cartId + `"`
            + `, shipping_methods: [{carrier_code: `
            + `"` + method.carrier_code + `"`
            + `, method_code:  `
            + `"` + method.method_code + `"`
            + `}]}) {cart { shipping_addresses { selected_shipping_method { carrier_code carrier_title method_code method_title amount { value currency }}}}}}`,
        });

        const response = await this.sendGraphQLReq(data);
        return response;

      } catch (error) {
        console.error(error);
        alert("Error occurred. Look at console for details");
      }
    },

    // Query logic to get available payment methods based on cart
    async getPaymentMethods() {
      try {
        const cartId = this.cartId;

        const data = JSON.stringify({
          query: `query getAdyenPaymentMethods($cartId: String!) {adyenPaymentMethods(cart_id: $cartId) {paymentMethodsExtraDetails {type icon { url width height} isOpenInvoice configuration {amount {value currency} currency}} paymentMethodsResponse { paymentMethods { name type brand brands issuers {id name} configuration { merchantId merchantName} details { key type items { id name } optional }}}}}`,
          variables: {cartId: cartId},
        });

        const response = await this.sendGraphQLReq(data);
        this.paymentMethods = response.data.adyenPaymentMethods.paymentMethodsExtraDetails;
        this.paymentMethodsResponse = response.data.adyenPaymentMethods.paymentMethodsResponse;
        console.log(this.paymentMethodsResponse.paymentMethods[0].name);

        await this.createConfig();
        return response;

      } catch (error) {
        console.error(error);
        alert("Error occurred. Look at console for details");
      }
    },

    // Function to send any query to graphql endpoint of the host
    async sendGraphQLReq(data) {
      try {
        const host = this.url;
        const bearer = this.bearer;
        var response;
        response = await fetch(host +'/graphql', {
          method: 'POST',
          mode: 'cors',
          headers: {
            "Content-Type": "application/json",
            'Content-Length': data.length,
            Authorization: 'Bearer '+ bearer,
            'Origin': 'https://8080-adyenexampl-adyenvueonl-wumtblawveo.ws-eu96.gitpod.io',
          },
          body: data,
        })
          .then((res) => res.json())
          //.then((result) => console.log( result))
          .then(result => response = result)
        return response;

      } catch (error) {
        console.error(error);
        alert("Error occurred. Look at console for details");
      }
    },

  },
};
</script>
