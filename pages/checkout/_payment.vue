<template>
  <div>
    <div id="payment-page">
      <div class="btnarea"> 
        <button class="graphbtn" @click="getCartId()">Get CartID</button>
        <button class="graphbtn" @click="addGuestToCart()">Set Guest email</button>
      </div>
      <div class="btnarea"> 
        <button class="graphbtn" @click="setShippingAdress()">+ Shipping Ad</button>
        <button class="graphbtn" @click="setBillingAddress()">+ Billing Ad</button>
        <button class="graphbtn" @click="setShippingMethod()">+ Shipping Met</button>
      </div> 
      <div class="btnarea">
        <button class="graphbtn" @click="listStoreItems()">List Items</button>
        <button class="graphbtn" @click="addItemToCart(defaultItem)">Add to Cart</button>
        <button class="graphbtn" @click="getPaymentMethods()">Get PMs</button>
      </div>
      <div class="btnarea">
        <button class="graphbtn" @click="placeOrder()">Place Order</button>
        <button class="graphbtn" @click="">Status req</button>
        <button class="graphbtn" @click="adyenDetails()">Details Req</button>
      </div>
      <table>
        <tr>
          <td class="statitle">Magento Host</td> 
          <td class="statval">{{url}}</td>
        </tr>
        <tr>
          <td class="statitle">Magento Bearer</td> 
          <td class="statval">{{bearer}}</td>
        </tr>
        <tr>
          <td class="statitle">CartID</td> 
          <td class="statval">{{cartId}}</td>
        </tr>
        <tr>
          <td class="statitle">Shipping Address</td> 
          <td class="statval">{{shippingAddress}}</td>
        </tr>
        <tr>
          <td class="statitle">Billing Address</td> 
          <td class="statval">{{billingAddress}}</td>
        </tr>
        <tr>
          <td class="statitle">Shipping Method</td> 
          <td class="statval">{{shippingMethod}}</td>
        </tr>
        <tr>
          <td class="statitle">GuestEmail</td> 
          <td class="statval">{{guestEmail}}</td>
        </tr>
        <tr v-if="cartItems.length">
          <td class="statitle">Cart Items</td> 
          <td class="statval">
            <table class="nestedtable">
              <tr v-for="prod in this.cartItems">
                <td class="statitle"> ({{prod.quantity}}) {{prod.product.name}}</td> 
                <td class="statval"> 
                  <button v-on:click="addItemToCart(prod.product)">+</button>
                  <button v-on:click="">-</button>
                </td>
              </tr>    
            </table>
          </td>
        </tr>
        <tr v-if="items.length">
          <td> Browse Store </td>
        </tr>
        <tr v-for="item in this.items">
          <td class="statitle">{{item.name}}</td> 
          <td class="statval">{{item.price_range.minimum_price.regular_price.value}} 
          {{item.price_range.minimum_price.regular_price.currency}} 
          <button v-on:click="addItemToCart(item)">+</button>
          </td>
        </tr>
        <tr v-if="paymentMethods.length">
          <td class="statitle">Payment Methods</td> 
          <td class="statval">{{paymentMethods}}</td>
        </tr>

        <tr v-if="placeOrderResponse != ''">
          <td class="statitle">Place Order Resp</td> 
          <td class="statval">{{orderId}} {{placeOrderResponse.adyen_payment_status.resultCode}}</td>
        </tr>
        <tr v-if="adyenDetailsResponse != ''">
          <td class="statitle">Details Response</td> 
          <td class="statval">{{adyenDetailsResponse}}</td>
        </tr>
        <tr v-if="adyenStatusResponse != ''">
          <td class="statitle">Status Response</td> 
          <td class="statval">{{adyenStatusResponse}}</td>
        </tr>
      </table>
      
      <div class="container">
        <div class="payment-container">
          <div class="payment" :ref="`${type}`"></div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
let AdyenCheckout;
if (process.client) {
  AdyenCheckout = require("@adyen/adyen-web");
}

export default {
  components: {},
  props: {
    
  },
  data() {
    return {
      sessionId: '',
      redirectResult: '',
      clientKey: '',
      url: "https://8080-adyenexampl-adyenmagent-wdx2n73tvz2.ws-eu96.gitpod.io",
      bearer: "3es3aafuow8r2jj8zb57cf43wa1ixk2t",
      cartId: '',
      guestEmail: '',
      items: [],
      cartItems: [],
      defaultItem : { quantity: 3, sku: "24-MB04"},
      shippingAddress: '',
      billingAddress: '',
      shippingMethod: '',
      paymentMethods: [],
      placeOrderResponse: '',
      adyenDetailsResponse: '',
      adyenStatusResponse: '',
      orderId:'',
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

    //const checkout = await this.createAdyenCheckout();
    // Create an instance of Drop-in and mount it
    //checkout.create(this.type).mount(this.$refs[this.type]);
  },
  methods: {

    async getCartId() {
      try {
        const host = this.url;
        const bearer = this.bearer;

        // Create cart data
        const data = JSON.stringify({
          query: `mutation{createEmptyCart}`,
        });

        const response = await this.sendGraphQLReq(host, bearer, data);
        
        this.cartId = response.data.createEmptyCart;
        return response;
        
      } catch (error) {
        console.error(error);
        alert("Error occurred. Look at console for details");
      }
    },

    async addGuestToCart() {
      try {
        const host = this.url;
        const bearer = this.bearer;
        const cartId = this.cartId;

        const data = JSON.stringify({
        query:  'mutation{setGuestEmailOnCart( input: { cart_id: ' + '"' + cartId + '"' + ' email: "developer@admin.com"}) {cart { email}}}',
        });

        const response = await this.sendGraphQLReq(host, bearer, data);
        this.guestEmail = response.data.setGuestEmailOnCart.cart.email;

        return response;

      } catch (error) {
        console.error(error);
        alert("Error occurred. Look at console for details");
      }
        
    },

    async listStoreItems() {
      try{
        const host = this.url;
        const bearer = this.bearer;


        const data = JSON.stringify({
        query:  `{products( search: "Messenger" filter: { price: { to: "50" } } pageSize: 25 sort: { price: DESC }) { items { name sku price_range { minimum_price { regular_price { value currency } } }} total_count page_info { page_size }}}`,
        });

        const response = await this.sendGraphQLReq(host, bearer, data);
        this.items = response.data.products.items;
        // this.logStatus();

        return response;

      } catch (error) {
        console.error(error);
        alert("Error occurred. Look at console for details");
      }
    },

    async addItemToCart(item) {
      try {

        const host = this.url;
        const bearer = this.bearer;
        const cartId = this.cartId;
        const sku = item.sku;
        const quantity = 1;
        const products = '{ quantity:' + quantity + ' sku:' + '"' + sku + '"' +'}';

        // Add items to cart
        const data = JSON.stringify({
        query: `mutation{ addProductsToCart( cartId: ` + '"' + cartId + '"' + ` cartItems: [` + products + `] ) {  cart {  items { product { name  sku } quantity }  } } }`,
        });

        const response = await this.sendGraphQLReq(host, bearer, data);
        this.cartItems = response.data.addProductsToCart.cart.items;
        return response;

      } catch (error) {
        console.error(error);
        // alert("Error occurred. Look at console for details");
      }
    },

    async removeItemFromCart(item) {
      try {
        const host = this.url;
        const bearer = this.bearer;
        const cartId = this.cartId;
        const sku = item.sku;
        const quantity = 1;
        const products = '{ quantity:' + quantity + ' sku:' + '"' + sku + '"' +'}';

        // Add items to cart
        const data = JSON.stringify({
        query: `mutation{ addProductsToCart( cartId: ` + cartId + ` cartItems: [` + products + `] ) {  cart {  items { product { name  sku } quantity }  } } }`,
        });

        const response = await this.sendGraphQLReq(host, bearer, data);          
        this.cartItems = response.data.addProductsToCart.cart.items;
        return response;

      } catch (error) {
        console.error(error);
        // alert("Error occurred. Look at console for details");
      }
    
    },

    async sendGraphQLReq(host, bearer, data) {
      try {
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

    logStatus() {
      console.log(this.url);
      console.log(this.bearer);
      console.log(this.cartId);
      console.log(this.guestEmail);
      console.log(this.items);
    },

    async setShippingAdress() {
      try {
        const host = this.url;
        const bearer = this.bearer;
        const cartId = this.cartId;

        //shippingaddr
        const data = JSON.stringify({
          query: `mutation{setShippingAddressesOnCart(input: {cart_id:` + '"' + cartId + '"' + `, shipping_addresses: [{address: {firstname:` + '"' + "Bob" + '"' + `, lastname:` + '"' + "Roll" + '"' + `, company:` + '"' + "Magento" + '"' + `, street:[` + '"' + "Magento Pkwy" + '"' + ',' + '"' + "Main Street" + '"' + `], city:` + '"' + "New York" + '"' + `, region:` + '"' + "TX" + '"' + `, postcode:` + '"' + "78753" + '"' + `, country_code:` + '"' + "US" + '"' + `, telephone:` + '"' + "8675309" + '"' + `, save_in_address_book: false}}]}) {cart {shipping_addresses {firstname lastname company street city region {code label} postcode telephone country { code label }}}}}`,
        });

        const response = await this.sendGraphQLReq(host, bearer, data);
        this.shippingAddress = response.data.setShippingAddressesOnCart.cart.shipping_addresses[0];
        return response;

      } catch (error) {
        console.error(error);
        alert("Error occurred. Look at console for details");
      }
    },

    async setBillingAddress() {
      try {
        const host = this.url;
        const bearer = this.bearer;
        const cartId = this.cartId;

        //billingaddre
        const data = JSON.stringify({
          query: `mutation{setBillingAddressOnCart(input: {cart_id:` + `"` + cartId + `"` + `, billing_address: {address: {firstname: "Bob" lastname: "Roll" company: "Magento" street: ["Magento Pkwy", "Main Street"] city: "New York" region: "TX" postcode: "78753" country_code: "US" telephone: "8675309" save_in_address_book: false} same_as_shipping:true,}}) {cart {billing_address {firstname lastname company street city region { code label} postcode telephone country { code label }}} }}`,
        });

        const response = await this.sendGraphQLReq(host, bearer, data);
        this.billingAddress = response.data.setBillingAddressOnCart.cart.billing_address;
        return response;

      } catch (error) {
        console.error(error);
        alert("Error occurred. Look at console for details");
      }
    },

    async setShippingMethod() {
      try {
        const host = this.url;
        const bearer = this.bearer;
        const cartId = this.cartId;

        //set shippingmethod
        const data = JSON.stringify({
          query: `mutation {setShippingMethodsOnCart( input: { cart_id: ` + `"` + cartId + `"` + `, shipping_methods: [{carrier_code: "tablerate" method_code: "bestway"}]}) {cart { shipping_addresses { selected_shipping_method { carrier_code carrier_title method_code method_title amount { value currency}}}}}}`,
        });

        const response = await this.sendGraphQLReq(host, bearer, data);
        this.shippingMethod = response.data.setShippingMethodsOnCart.cart.shipping_addresses[0].selected_shipping_method;
        return response;

      } catch (error) {
        console.error(error);
        alert("Error occurred. Look at console for details");
      }
    },

    async getPaymentMethods() {
      try {
        const host = this.url;
        const bearer = this.bearer;
        const cartId = this.cartId;

        //get PMs
        const data = JSON.stringify({
          query: `query getAdyenPaymentMethods($cartId: String!) {adyenPaymentMethods(cart_id: $cartId) {paymentMethodsExtraDetails {type icon { url width height}isOpenInvoice configuration {amount {value currency} currency}} paymentMethodsResponse { paymentMethods { name type brand brands configuration { merchantId merchantName} details { key type items { id name } optional }}}}}`,
          variables: {cartId: cartId},
        });

        const response = await this.sendGraphQLReq(host, bearer, data);
        this.paymentMethods = response.data.adyenPaymentMethods.paymentMethodsExtraDetails;
        return response;

      } catch (error) {
        console.error(error);
        alert("Error occurred. Look at console for details");
      }
    },

    async placeOrder() {
      try {
        const host = this.url;
        const bearer = this.bearer;
        const cartId = this.cartId;

        const stateData = "{\"riskData\":{\"clientData\":\"eyJ2ZXJzaW9uIjoiMS4wLjAiLCJkZXZpY2VGaW5nZXJwcmludCI6IkRwcXdVNHpFZE4wMDUwMDAwMDAwMDAwMDAwS1piSVFqNmt6czAwNTU4MjEzMzZjVkI5NGlLekJHeWRJdFhHVURNbUJpeDdSWDNhejgwMDJEU0tnWWczclJMMDAwMDBUSXZqWDFCWHc0blhrQVpBdzljY0RVdzlTYkQ4NGg6NDAiLCJwZXJzaXN0ZW50Q29va2llIjpbIl9ycF91aWQ9MWNhMjYzNjItNWQxYy0yNGZhLTAzMDEtYjQ5YmVkYjI0YWEyIl19\"},\"paymentMethod\":{\"type\":\"scheme\",\"holderName\":\"\",\"encryptedCardNumber\":\"eyJhbGciOiJSU0EtT0FFUCIsImVuYyI6IkEyNTZDQkMtSFM1MTIiLCJ2ZXJzaW9uIjoiMSJ9.VXiDmSur6eB5kyqKetJt0lnDA-dOvBBgnrRIKdrIJYJ4guYaEA0ldZZbgMyj5Cf-HjbJ-0xqkAkyIEvb-MeQSK1QHUC7ZbbEHCVdr2ZwUbNVTk44Un8YFixzodfNzCsdo-MtBG_D_vFosAwpQCu-jpivUIC2tglV14Z0L7QPat5LkFuk1b8jKmBzEApC5OCIwjf-M0UaJegbD_sNk1XAbINBeMA-LTIcbQ-4UuJhgbFjeS7R3Vi_H1W2Y8ZJ9szTgj1whwDkQf3lKzAox3AaZRDiuWRCtW-9OMh3y_OuVxSLfhGE7FWfGMVxtPVqzDQJbXDGEpQ-c80c_3uo_lXgww.ahyUReX7xwcJHY26CDfiBA.nZ9O4ArkrLZCmzfJw71_-zb8-ujE8B_NDW8GHRdM60KJr5_x9vYk8ul2cLAlxLVgJTjl-KK4N0asjMZIuP0Fot-vgnUfoHrCaOzR44ftIDbCsFVgcj00lhIO5kY1blVy5QyqeOdssZZsBAABosz4dz8EXqPrn0uwTP1wJR4eN-_6xZNwH5OZwn6lxObCp4d1xrBdQbm_j1kVIO6QW4Cpx3DpnFsQdTcwrk2H3U9iF7oVm8FHtsiBUYtE5fa-FfXwrryT--KHBZozth8VQac_8Y639AF60MEZ0fNHuFEWeAW_CGvoX5Jj1o9ZXgZWzzaPDUORjq0mmPp-unAV1-DtLI2a0QOWgW0JxSP0fhb--jCd0dNqrT3sgovzqpn8RdPZK6Zx79jE-2_v68BYI7C_qP7J1Ypun42mrztP5I8O9POKdoqrGhYqIntq_GxQmJhKR9WQLUFeWbDzRD1SoUyDihZKiQWxlIjBvTnYgWWxNGYumlBT0Dgx2XNS_jQ7XiPKeNU9QcBvgdoV_DnxV3jecttccsIixa6lKEU8Iyhx_L5iZEJ8GzxSac3eSoYYnYzt_2pOLYydMvs-Clx2sTB54_9vzLRVhdybtY8BJzMGUbtxulEh2ifDH2gRrbchX3_Ny2RcgQAS_f8UUTWLXfyusuo2FBzaAiD09ZE_DxjiBY.dMPVR3WXCBw4I5VEjG1hCJXzEWP14WEDZF4yHSY-hAg\",\"encryptedExpiryMonth\":\"eyJhbGciOiJSU0EtT0FFUCIsImVuYyI6IkEyNTZDQkMtSFM1MTIiLCJ2ZXJzaW9uIjoiMSJ9.JL5g9ZVQWgsk7UzvhdbAuIdAH8xZlZWKc3PEoN-cVD4RUdn0ksQfv6SFuarWfqISrceVGgDRR4AwW9315_LBsc0-Sda2Wx6PV8AA6SMOwXaGmIUw38rEAP5IDAFf3NrE1OidbppCBwq0UtWeKFJg-crz6pkz-aXVW2Z986WzOyupnwJG3cfJRWalTzkiD4RG4mzi1VbU0jGSoBmreM-_YAEzqfhQLNJHSyuifcrI9pJanCHlQduyh879M_l7hxxanJVaZluBmfjNvYwRUaeAYwEvEH8Bil1lRnfWi9ZAKq9v_C6llpTnDpcyucNNK9c0VgsvzuBN7m_S3ifjwve60g.0Imu0g3QVX_y2POScJf0cQ.iH2e4E-3zkPqok5P4wGYGkuLFvRmq0EBmcXzmkN4XuSpi7FzOZT3qbCzYtGAr2Q-zX8wgdVBm3Vro118KVbAMg.nNJJEmucL_Zte8bzX-fQrCzuRbtXcY_hCs0Yd-5k-qk\",\"encryptedExpiryYear\":\"eyJhbGciOiJSU0EtT0FFUCIsImVuYyI6IkEyNTZDQkMtSFM1MTIiLCJ2ZXJzaW9uIjoiMSJ9.Rgx9dZsq9KjAntOWnMQSNRtA6A5A_koVCyWBdugCZBxQ8PuOov37irI-UxWX3reas8HGfYVO2FNKvsy_L3-gRg0LBseiRWvwHAzSYu3ditjOexGfdm3PkPW2DO4rge5Y2QrRzGJsXfEEDDO8t3q8D_4QzU2PTNRfrdWyguTm2djFbCDjFJJDhY160kvgkJ94RycY5jPD6n9g1-5HinlKPdeO3X7W7CtByHTVpMYiiYL2ps1JCQmZMUaa_VBKFWLj_mNDKp-dh34N6AsWcUcyEIjEIZCCosAjmpo6DXm2fZV_f8NEZ_vZqqPQKmvtG1wewuLKW5fbDIPMV2UHBt7a7Q.P2PNtKcWAULqbDaly6zI_A.dmjTl8gc5t4OPk3SEtXvKlYOhAbzdTIsXcmJGeR1aA1sSh2vkfh2ps5mOoW0qi_A00LAePIJ3eCYIpk5yONbFg.wE7UTFLapynPDZsuCDONU83gUpk4rFK_xXO_xpAogY\",\"encryptedSecurityCode\":\"eyJhbGciOiJSU0EtT0FFUCIsImVuYyI6IkEyNTZDQkMtSFM1MTIiLCJ2ZXJzaW9uIjoiMSJ9.XDB4g6DNnvvQjC6Ob5AQUnVPV6jaeQYy4U64Frqwv9ddbVphp8ZNdEIZnbv8cAOot9iBlaHW-dMyVmjWpEmg1DdP60MlGMZegKmADfO1FfPoeld1oSbnGj9dba0PWMGE8zwfhyvk4qqiogvMH4CDVfaY9Pss4VwsWNPU7PsnICbvejY0crFCoNFdCYn_4lTd6lEZ7MosXJVqoP811kp29rh4CEk1fGj09OKOQirsDttptdGBFnEiCtS-TctsmPxjVy6hYPevsnKI8tVQ7Qa8FYboLHMB-fC_AWWg8bHgibgCBriGZwNr76MpograJxBunRWA0Qnzg6z17U08PK7O4A.2dbZcKQoNZp5skk5i_ypgA.nj0K7atw65BRF0kzv49WMujbzMUCuoV-plLuex230gyNKRLCRE7OGMV2GWzGsa3HJLbbesxU7B2LAueIKu61AKya7-ejeIxp3DwT42QMl0NM58y6OFlMMeghg_yJ17pGRJAU84fgRFHcf6LKp2fM8F0RvjP4KIl22AeF3T2kBIekPjVn0TfEj_nHTPyPRxTw7XwJULw_tWwTR_UW4iONPnCZlOldAtVo28pbPjcB8nToO1KvxNoMwwjLAy_SL0Je5pWBi39Zn1Hrrv9NS3tssDOF0Xz_kt82do293mAGBwTnLc_hfxOqy58lLYf4Auj32q5QvYSnlf-lNV7YuNTYoJQvdKb48qQQYLhUXEwnMcMaZauSdoL37F9wVxowWXgb3_A8_L4aRMe2vjC2mkvsG4GeYTZR92a0UR9TVJt2fiL7dr6-jdx4PlG0yBjdFtVBzayHfBL_cgyHWII4oXqjMSW8W1phVDkNuCeXrqrTPIXpUdrV9X7OG_KUj04EhOy-8odPGwJcxsZEXzLteimYgtrnHxlHWAGOtcHUXjW9My_PLU4ax06PMrD_ULZKgEIG.3DPmw7hgeZtDsh7zIfN9T4UNvbhaf5kAQpX0FJIm9CI\",\"brand\":\"visa\",\"checkoutAttemptId\":\"9bb9e15a-b08d-4fa2-ae03-b40db2a535411682783460062C86E169D488EA857F05DD9175AD1505133F842BC2C899879C062EFBA04AC1727\"},\"browserInfo\":{\"acceptHeader\":\"\/\",\"colorDepth\":24,\"language\":\"en-US\",\"javaEnabled\":false,\"screenHeight\":1200,\"screenWidth\":1920,\"userAgent\":\"Mozilla\/5.0 (X11; Linux x86_64; rv:109.0) Gecko\/20100101 Firefox\/112.0\",\"timeZoneOffset\":-120},\"origin\":\"https:\/\/8080-adyenexampl-adyenmagent-wdx2n73tvz2.ws-eu96.gitpod.io\",\"clientStateDataIndicator\":true}";

        //place order
        const data = JSON.stringify({
          query: `mutation setPaymentMethod($cartId: String! $stateData: String!) { setPaymentMethodOnCart( input: { cart_id: $cartId payment_method: { code:` + '"' + "adyen_cc" + '"' + `, adyen_additional_data_cc: { cc_type:` + '"' + "VI" + '"' + `, stateData: $stateData}}}) {cart { selected_payment_method { code title } }} placeOrder( input: { cart_id: $cartId }) { order { order_id adyen_payment_status { isFinal resultCode additionalData action}}}}`,
          variables: {cartId: cartId, stateData: stateData },
        });

        const response = await this.sendGraphQLReq(host, bearer, data);
        this.placeOrderResponse = response.data.placeOrder.order;
        this.orderId = response.data.placeOrder.order.order_id;
        return response;

      } catch (error) {
        console.error(error);
        alert("Error occurred. Look at console for details");
      }
    },

    async adyenDetails() {
      try {
        const host = this.url;
        const bearer = this.bearer;
        const cartId = this.cartId;
        const orderId = this.orderId;

        const payload = "{\"details\":{\"threeDSResult\":\"eyJ0cmFuc1N0YXR1cyI6IlkiLCJhdXRob3Jpc2F0aW9uVG9rZW4iOiJBYjAyYjRjMCFCUUFCQWdCdnZEWlg0M0xIbkliZWxYZk9vTmdIQkgrdDV2YS9yWmRmUjljVC85RXN3blNHUkh0d1k5OERjYmZKc0FpYTBsdUVMOFdPMy81Y2tZSGpudUVkYlpFTGZKNFN1MVJFUkhWL0FCd2hwMVo0eWdRandGMnpNQXUwNXpmRmRwcklLOWVPWG1lWnQrYXVkZW5MbmViVU4rMlVTaXltMXh0aFFrUGY2Z0gzcmdCTm1DMVJPQTdldXE1T1BRNzQ4alVrWXVSSVpzL0lGRkptcU5nek8yaVpGMG50dFNRNUZWQm4yanR2TEJYNWo2cW4rc0xJS2RFY0oxcW5WRmE2Rm11bGZZV3BkM3VwOUNjNGtVQkcwYlQ5SHVZL3dGUGp4QTMwUXV5R0REMVNhNXI4V2tpazk5bkpGMjF0VWdpMXI4YlkvUFl2NllxSkxyWUQyYVplMFlYTE9VUElJUzhEREZzVk9veGRsSE5IdkZRaHlqb0tPV1ZLcWJKak5GL2pScnd5NzFCS1M1NFdpVjZBNmdNT0szVktFNEZEYjZ2R2pYVFBpUnZqSVRQUGhHU013S2NvWE1janlEanBnSXpWOWQwdmp2MElUVGV4VGlnUUFQM21pUkJYQ2JzQU1oYlQ5RUNScE11MmptUTZ6dU1qS0c3djlhZXhXbTB1ZnBpN0V3RWtLUUhVSHRaSkQwU2svdUhuVGtQT1lVcTBDN3F4T21uZXBOYlJXbkh2VUZIcGlBdENpd0h1L09oaGp1cEJuekdmVDgrbHRqMDE1NkEwZDlkLzRqMDU5TnBOeWcvK3g0OVBjb0lsanFsZENPSUFFaDNGams0MTZta1dXSmx6QXArVTlVRWNVMVcyM095bkgrenVpWC9ZcDYwS1EzaUhVc3BZT25NSGJqeE5waU9TaXd6UHROUlFqcGFZaEp4dmtVa0FTbnNpYTJWNUlqb2lRVVl3UVVGQk1UQXpRMEUxTXpkRlFVVkVPRGRETWpSRVJEVXpPVEE1UWpnd1FUYzRRVGt5TTBVek9ESXpSRFk0UkVGRFF6azBRamxHUmpnek1EVkVReUo5aVNlUzUxVVJ5Z1J1MlNJKzlNelJRdVJEOTFybXY5SnorVk82d3lKMDB0YXZYdzY0RW9UZEZ3cEl0cVpkczdWOEttbi9Bcisya0VQdHlFOUhwbzZmdUM1dGNmTTg3L1R1dmlJSTEwWlpjV1d1a1FYQjRHSWZUZ2tCL29ybmZFem1iWmNxTmZUQVJlL2RPQWtyTjAwYkg2Z2NKcFMrdzdEakUwVklUOFRWOWJIL2o0ZSsrbjcxUVQ0UzlxdmR0Z2FIM2ExcjYveVMwZXVub2oyL1NmWmRnZXJhRUFKbEFJWXdpNzhReHFZQWE5dTQxcko2akxqZllhZHpmT2QrYXpZMGxvUHFOQ0hZc3Mydk5lamRFYmNwVy95YlVvTEFtMCsvb0tnZ2JDR1VSV2krb3dXL29JdUFnamE0TEszc2NZWnRmZkNqTzFybWZFZlpVVllSUllZV0NvSHo4WTdvbFErRVQzdWJDN0RDa1dkL21CZDIzam12NmV0cWwrVzJvSlR6bGRDdmVRVWNOeldsMTVvQ2tSV1QxclhXa1lpRUVxWVZIUGNXekhxQVd0VHBvK3BLbWFXQmdNcGlKMnlmMERzVHZ0RHQ4YU5ralI0NGRuaVZ2dlJLRmE1NzVmRlRlai9DeTdhZ3BhYmV4bXBTOTF3ZEErQ1MyOUpwejFzWGxkdDl2NWMxcEE1S3U1Wk5xcHRhblB1VllueENqa0ZEOFBOUzBLVjBSS3A1RG8xYmJ2b1I4RVBCZUVuVUd4QjBQWXdKMU0vb2hWN29kbjBDZFVSWVdNZGIzQ1hJbzZvWVNwR2xxNW5scFFGQVRmRzJyMjE1N3J1QlZML1JZUUdrQXZuQ0pkZ3NIY1JReFZmck04aXJ1ajdqQW5mTkVpQnVEemRobXVkQkZYTVhzOHRhTFRaTEtRcjllVHlkaERrRUFGWUUzaXVHelF3MjJoWFNTMU5VcWZEVjlIM1NGRlhrandYbSswNTZTUGFPb1V5SUo4TkxvTkdNNmxoMi9JMHZsTENkWjlLTGdKRUJ4T1ZZQlB4N0FvbE82d3BDZFRRRXV4dWhHR0Y0UFZEcXBpOFJ0bzllVTVjMFlQamxGQWVVQnJ4VDVWTERrcUoxNE5sK1pXaHN6YlM5TUQzTk1zVWhsNVF4R3BhZEZkSi9XSE0vb3JTS1NzM0RMeGFsc1hvcU5aNklxV1FlUmI5WnJvbktnMmtlU05oT0cxdlVMMTBuWHJDRnlvWkJkS0w1MUYyNFF6VDRvSDE0VjRPYklkc0dMTkcxYzF3WmZqWHBjQVZ1TFBPWFlSdG55cCtLRXFSTUhFMnc1S255Y2hqTE11ZWlyaGUxMWU5KzV0KzBVLzBLclJXMjF5a2JKdjZyY0FiL2ZNcTdRaktET1pEY0F5VVFNbHp3YnFFaDFXeGJPYjJVUmNHRHhDR0VjbXFybG5DRW9GYUMxSXNoQUQrRUJGd2dEMDJQdGZFdGowS2hvc05vdWpkcVRNblNKSCtNNWcyZ0wzSEdjVHlKWjYrYnZGY1ZvL240eG9SL1U5MTZwUTlNZ2o4bGdDKytmVEVjSlpEdDdRdmtYY0ppcUd1V0ZJUWllb2k0c3VTY2lDWEJJTU9INEJCRXNzSjNjZ1ArNGxucmozTVhEVzg3Ti9MMkdERDFLZG4yVEFLS1RwZno0c21iLytkaUZ4TUhEMjhkVzl0dHJ4SnY4WFJNUXdmQWNiV2N0QlJQVVJncXJZR1EzaERmampkd0NNcmU1Qjk3QVYvOE15eVA1Wi9ZYnhMTHRBVEhCMzNLa3Y1WlJKcXZzVHdkWHNSTk80Z3A1QnJDSmVHVlFMRXNYWDc0TzdYWjlsRnFMd2NxTmFJcUxPbHRmSTFuaStON21iZG1KNHRMNWlJNVY4MXFlYVhJdk83VTZHMGVPZktnZVNlQnRpRDJjUVZyOEZGU2FPcVJCUU1DUElzb0VubnpnZ0VLVXdqbHNuRnlyNi9QOUJ2d0FoTGRseVVkbkYybFpBaUZyNDhKUjhXTEtrTkg3YUx0U3VQOTIycDBYS2Mwa2FCS2U3TTZrc215eDd3ZGxPTnNjaXpGUGdXSjkwMHI2YzhyYmJ5SWw4aHBYajBrV3ZWWmtZWUVxaTZVYVNJdXdZR2Z6U3BWT25sU0ViKzd0cXhtQVhNMFFUUUh1VVlyZTkwZjZRRWVWWXpLZFBPQWJUTWk0eFBmYmpJQkZPSDkyTXFrVTlYc0k0TXNSNTl3dmJZUnZtcTc1cS8zeEdaUWZCeDUwWDM1U09JcEtIUlovU3NDVGV0NFkzRm1qMGdCa1l4emxOTTBMaVNPenFKS3VIK1VZWjc5ME5lWFk3LzNMdi9BcUtqWCtXb1R5QnNVcHNuVE9CZ2VubjU4ZlNFRkp6d0hMOFVWaWF4RUIrcGxPMFZjWHphaFgrcXZJZDNOMnpnMkFxRFBCWWZOazArdjZsNW5LN08vRzZTTTU1SEVqNHh3UVp3SWorNFNRZWp4NUhaZ1JTU0w0S2Z4RFpjdHJmWHMvR25FK3ZDaW92VkJHWHhBdHlSb0tTVnhNMVQ1Skd1b3J3YmVzbEk4TFVHMGlaVnZzbWlpUktSdHA2Q1F1bkErTWFTVkM2WEZ2WVRlVHpkak9Hc1V5MnVud2R0ZS9KWEJGRWFDK2M0VnRhV2F6bEgwazZPQ0hvRHcrd21qaER3TWVLdGpVRTBsTnRPeFZhZXN2S0ZxVkhzUFkxRkVUcW9waVdpdmFXaWNDMyt4UldBcVdDelRCbXU4bVc2d1V3NHF5UFhHdG45M29URGF1WlFpVmV4dFVneDg5NVFwU25PSWpnU1cyalcxcXRBRHYxbHZvaHYyc0hONGZ1bVkvZzZ0TE04bnFlcEpsTUtwaUJldWI4WXlMNVVmUzJOWThCbnNsS0VCSHZ2T2M1cEo2S1lHZGNDaElCenErWXNKUldvNXhBaDlVdXZBLzNtUDUydjBkVFI5T3NmRnZoc2cwN1hlbTZSZ3Nsd1FzQ01sSWdsM3o1bFlVNzBCRzIyK055MUZqcmt3MlF1emx5VGZlZ2ZHVHJlNEpDeEdvODgwNG4zeVd2bk5lZk1DMkxiT3hCdzVlM1hGSk9oRnNVby8yY1Vaa1NlVkFvdGFWRXRXNGRYNjhsS2hFN1RRQmpPWlBteVhTTUNLeERscTBvRCtLc2E4Q21zTDB3WTVWUXFpdWVlcnQxTnR3bjQwanhJTUNKaTRmais0QmJrd3hsOUhqdzkvQmJQWmk0d0RBN1pIeEJWZWNaaWljL25SdTh4cDFEaTZIbnd2M21VdGdRcGJNTWJhU2hFOEMrbDJBTzVBRzNlTGowVTA0ejIzNWVtVTNIOGRpWnhISyt3RmFzVXZOU3duSWpvSENpcW5OOUJUTUtFalEvTmZlZ29ScFR6VDQzenpzTGZnU3Y1MmlWaHpxTHd5MTlwc0NtY2ZmM3JDVm45RzlBdldnSTlma3d6KzJqSE1pZUl1eHFhNEVKemN4WHdra1VnWkdrNkVaZHlMMFloZkg5VXdrTDdUbzdVeGRmclo1R3FLblYySDZWWlRNd0xlK1R4ZTMyYndYWVZjdFBTRHpZRjdIS2NkaERKT2FGNldwVE84cHEraVpKbW4ydXNiekM0a1ZpZHdQMThzNFNFdXJ6WE9oeVp5RUczNktGeVJ5bXpRRFRvYU9OQ0tVeG1xR01DMWxFeGcwMlVCWWRpc1M0Z2ZLNFU4c0NkTzEvK08vaXdEZDcrVy9sVGdLenV4V25rNnpIQ09IT2FxMHFBdTBHRFU5NU5xdEV5TVlmNkRqcFNIeTdYa05MZStibktVUkl6NUZLRGtLY005TlJ6TlJLOHgrSEFrOGs2NVFidnZVck5ERU1qYzNNK08vM0xxNWFIaHlhNjdyMExWMFBHMDJMU0RKdjNWNHdpV0FZUnFCL0sxTjRIODExR3VrOGprWmx3UEFwOTRmWTloMEQybDBBSTJqSHBYc2pLdkgrSk1qUjRxU0JKVkdiMnRkYkpUcjVkVDY4TUMvWGFzaWwrL1VheElWaHdWc3VwbnRMcXhXdUNyQm1WTTFDU21CZ0hobjkvYzNtRmJHT1FUOWtSeHdyMklpQXQ3QzM4SjMxNElSdTVqQzlkSmRKRk8rK1k5VHV3UDlsSGpPaWNXaHRhN0YrSmFEc1lJWTF3N0dwejhSRXo5T0VSb0RXSm5hS09hcEhGcXd3RklrVmxxQ2pkY3hTalNDS0RXVHdSdU93ZnNUend3T1dobys4dHRuejJWUDYvVjQzUEFLK0I3SWZROHZIaWxVK202WWZYaTMzdlVhanZBYmdxVXdSQ0xSaGNYdnRZamF0djJFdTlvd3g1d1ZYUEJEREdmV3FLRDdObEI5RDNhYlBSL05LbWxCcmRZNEZmUEN3QmVWaDN2SHBVOXJFTDBZRVBHUklmSGNjVzR0VmdpNWxDeWczVnFaYSthRlFQYjJqdFBFUnhvdmVYZk9XWGJ0MkhiTWxZeXNFNkFsSndNN21yK2ZXTmtkdjVzWG1jMWx2WVo1M3QrQ3ZkaCswQVExbGIxK0pza0ErUDNXVEphWFBseGN4N3kzeEJxVm5NRWhIbzRKWk1zNGNhenJlWGZoeVg5aUpjN1prR0RSWnR3dmFDQVF1OHB0SkV4Rk8xYlZNT210T0RQMzVtMmx1bVN5RWw4R1R0KzZsc3RtWGhDaTRzcjk1OU93ejZLOTRUd0daWWtWWHdDb01WdTNlaEtsSFRSZUM2aHIvSWh5RUdyTkxQeEVxcjNKd2ZJQk1hTkpKNHh5QVdWUU9QMTlPZ2FaZTRVZE5pTExUeEM1Q0c1S1BIeFNabzZXOXFHZnJsc1lJTUswY3hGY2U5VXA1cUFncHR2L2ZubVdSMnhaRjhDQXRNaDVRVEFvSlcvTldjejU4cWlDYXV1akdYSzRraEgzTnVKMlh0Y09pUzd4YVdNKzZxS0p6MmNLaVFxcDFhcFF4WWNLK285T3NZZkxCTVFTUHdOVlZQZlZ1L0luMDN3R0RQUlRTQVF5QytncVVLZ3FsM3YySzNic0NmRHZLaTh5Sk1CZnhWVkdOY2xJbXZQa3BGb21ESGM4bXFlSEwrYi9PNXdPN2VOK21pa25rYmNPSXdGZ1lQa3FTY0Yvcld3QlVjb0lMeGpmMHU0cHR2QTBmVHRVdnRsK0UyQjdZTVljQ0hBN2VtUWFCMlFpNmxGTWV0amRVZDgvUDlBSllsMTY2TWFXQS9teGcyaURZRHljMVlPYzdtbFdXYm43bGdXZjB6QjFSdjMwSlJLbHFsaUppUm1JOW8yMUFaZXhwRkFZd29taFRsSTd2ZUs1L3FmRXd4c1dVVUVBczFQcjdLVWlzVldKenpVdHc0Yi9rRkhSekFVVm83MmJ6WXQ3ZHM0c21LTWE0b3VRMmR3cUxnRjRsejRoWWcydVpIdDFhc0wxNW90U1AyaEQ0ZHJUcHJnYk53aWNvK3NLNlIzd3BqL1dqOWdVSzJKaVBDeitqWHZDVVpQR1llQjN6cEZJNlZxbEpabVMyUjE2eGk3WU1uVDBkdGhDeDlndnlkNTh2ZVNYZjl5SnJwWEpKcDhaTFNhZ3MrL0JYdFBoQVU1bkVRNlZBc2VqbCt5Nm5hS0VqV0JCVVNoNWFGMjlWSDcrWEJPOFhDa21MYnlQQ0V1eU5lWGZ5NmJzcXBCcWpnNlhYakVZQlI4cVNLaWhUR21nTUNrOGx5R1pCOWI1TkxsT0QzUmZPZDVGSktqekU0am80WDZXU1VweWhmVHBBRGZ3a3I2akNYd242bXpQM1JueXNYK0dXM0ZhWmRJN2pORWxPQk4zdUo5TDkwbW9WTVRWRElqSXI1OUhaQ1N1M1ZRWXBYMXpPVExSV2hoRnFIdk40Z3MyT2xlMG9RZDJuQzQ5dGFWTmtQVzNSaWdvZ3BCRHozdVBQQTdsTFpHalkyT29JaC9OakdHWTMxak0vaFNwemR4bTlORytIbTB2TWVIZEVqK2pycUI0UFRpcnZSQTRlZStRZUMrRllrbkYvQnRyQzYzTm55OVV2K2p2aUFTQzZmd3FvYjQ2dk5JTXZOOFlINlBjTXNHUXJMejJwcnFWZitUQjJOZlNtbEM2NmxNOEJjaG5DaVNQZHZkM0R1REZTcjl0dTVuK2tjeFliTUdIUGxMVCtpdXh2N2drZTRDQ3p1Sk9xSUlOSUFueHZINy9nbnM4d2lXYjZwOFprUDc2QTdhRks0YzJxNThtLzdPcEppRXpVSTZXdzkrSWpsc3lMNHBhMmdyTTJXa1l6dTlRL0F4SlRWVDBBdDdRamxVUHBLd25EaGpDOXBWZnVjVUhwYlkwdGd0WlE9PSJ9\"},\"orderId\":\""+orderId+" \"}";

        //place order
        const data = JSON.stringify({
          query: `mutation getAdyenPaymentDetails($payload: String!, $cartId: String!) {adyenPaymentDetails(payload: $payload, cart_id: $cartId) {isFinal resultCode additionalData action}}`,
          variables: {cartId: cartId, payload: payload },
        });

        const response = await this.sendGraphQLReq(host, bearer, data);
        this.adyenDetailsResponse = response.data;
        return response;

      } catch (error) {
        console.error(error);
        alert("Error occurred. Look at console for details");
      }
    },

    // Original methods that can still be reused
    async createAdyenCheckout() {
      const configuration = {
        clientKey: localStorage.getItem('clientKey') ,
        locale: "en_US",
        environment: "test", // change to live for production
        showPayButton: true,
        //session: session,
        paymentMethodsConfiguration: {
          ideal: {
            showImage: true
          },
          card: {
            hasHolderName: true,
            holderNameRequired: true,
            name: "Credit or debit card",
            amount: {
              value: 1000,
              currency: "EUR"
            }
          },
          paypal: {
            amount: {
              currency: "USD",
              value: 1000
            },
            environment: "test",
            countryCode: "US"   // Only needed for test. This will be automatically retrieved when you are in production.
          }
        },
        onPaymentCompleted: (result, component) => {
          console.log("result: " + result);
          this.handleServerResponse(result, component);
        },
        onError: (error, component) => {
          console.error(error.name, error.message, error.stack, component);
        }
      };
      return new AdyenCheckout(configuration);
    },

    async finalizeCheckout() {
      try {
        // Create AdyenCheckout re-using existing Session
        const checkout = await this.createAdyenCheckout({id: localStorage.getItem('sessionID')});

        // Submit the extracted redirectResult (to trigger onPaymentCompleted() handler)
        checkout.submitDetails({details: this.redirectResult});

      } catch (error) {
        console.error(error);
        alert("Error occurred. Look at console for details");
      }
    },

    async createAdyenCheckout(session) {
      const configuration = {
        clientKey: localStorage.getItem('clientKey') ,
        locale: "en_US",
        environment: "test", // change to live for production
        showPayButton: true,
        session: session,
        paymentMethodsConfiguration: {
          ideal: {
            showImage: true
          },
          card: {
            hasHolderName: true,
            holderNameRequired: true,
            name: "Credit or debit card",
            amount: {
              value: 1000,
              currency: "EUR"
            }
          },
          paypal: {
            amount: {
              currency: "USD",
              value: 1000
            },
            environment: "test",
            countryCode: "US"   // Only needed for test. This will be automatically retrieved when you are in production.
          }
        },
        onPaymentCompleted: (result, component) => {
          console.log("result: " + result);
          this.handleServerResponse(result, component);
        },
        onError: (error, component) => {
          console.error(error.name, error.message, error.stack, component);
        }
      };

      return new AdyenCheckout(configuration);
    },

    async callServer(url, data) {
      const res = await fetch(url, {
        method: "POST",
        body: data ? JSON.stringify(data) : "",
        headers: {
          "Content-Type": "application/json",
        },
      });
      return await res.json();
    },

    async handleServerResponse(res, component) {
      if (res.action) {
        component.handleAction(res.action);
      } else {
        switch (res.resultCode) {
          case "Authorised":
            window.location.href = "/result/success";
            break;
          case "Pending":
          case "Received":
            window.location.href = "/result/pending";
            break;
          case "Refused":
            window.location.href = "/result/failed";
            break;
          default:
            window.location.href = "/result/error";
            break;
        }
      }
    }

  },
};
</script>