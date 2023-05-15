<template>
  <main class="preview-page">
    <section class="cart">
      <table>
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
      </table>
      <div class="summary-column">
        <div class="order-summary">
          <h2>Cart</h2>
          <ul class="order-summary-list">
            <li class="order-summary-list-list-item">
              <img
                src="~/assets/images/sunglasses.png"
                class="order-summary-list-list-item-image"
                alt
              />
              <p class="order-summary-list-list-item-title">Sunglasses</p>
              <p class="order-summary-list-list-item-price">50.00</p>
            </li>


            <li class="order-summary-list-list-item">
              <img
                src="~/assets/images/headphones.png"
                class="order-summary-list-list-item-image"
                alt
              />
              <p class="order-summary-list-list-item-title">Headphones</p>
              <p class="order-summary-list-list-item-price">50.00</p>
            </li>


            <li class="order-summary-list-list-item" v-if="cartItems.length" v-for="prod in this.cartItems">
              <p class="order-summary-list-list-item-title"> ({{prod.quantity}}) {{prod.product.name}}</p>
              <p class="order-summary-list-list-item-title">
                <button v-on:click="addItemToCart(prod.product)">+</button>
                <button v-on:click="">-</button>
              </p>
            </li>


            <li class="order-summary-list-list-item">
              <p class="order-summary-list-list-item-title">CartID</p>
              <p class="order-summary-list-list-item-price">{{cartId}}</p>
            </li>


            <li class="order-summary-list-list-item">
              <p class="order-summary-list-list-item-title">GuestEmail</p>
              <p class="order-summary-list-list-item-price">{{guestEmail}}</p>
            </li>
          </ul>
        </div>
        <div class="cart-footer">
          <span class="cart-footer-label">Total:</span>
          <span class="cart-footer-amount">100.00</span>
          <nuxt-link :to="`/checkout/${type}`">
            <p class="button">Continue to checkout</p>
          </nuxt-link>
        </div>
      </div>
    </section>
  </main>
</template>


<script>
export default {
  asyncData({ route }) {
    return { type: route.query.type };
  },
  head: {
    title: "Cart preview",
  },
  data() {
    return {
      url: "https://8080-adyenexampl-adyenmagent-7j25ev8o4sr.ws-eu97.gitpod.io",
      bearer: "mbvjlftxgpunwaiqi0tfsn2dhkhxpips",
      cartId: '',
      guestEmail: '',
      items: [],
      cartItems: [],
    }
  },


  async mounted() {
    this.storage();


    await this.getCartId();
    await this.addGuestToCart();
    await this.listStoreItems();


    localStorage.setItem('cart', this.cartId);


  },


  methods: {
    storage() {
      localStorage.setItem('url', 'https://8080-adyenexampl-adyenmagent-7j25ev8o4sr.ws-eu97.gitpod.io');
      localStorage.setItem('bearer', "mbvjlftxgpunwaiqi0tfsn2dhkhxpips");
    },


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


  },
};
</script>
