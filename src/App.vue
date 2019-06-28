<template>
  <div class="container mt-5">
      <h1>Cosmos Reward Tracker</h1>


      <b-form @submit="onSubmit">
        <b-form-group
            id="input-group-1"
            label="Cosmos Address"
            label-for="input-1"
        >
            <b-form-input
            id="input-1"
            v-model="form.address"
            type="text"
            required
            placeholder="Insert your Cosmos Hub Wallet Address"
            ></b-form-input>
        </b-form-group>

      <b-button type="submit" variant="primary">Track</b-button>
    </b-form>


    <div class="mt-5" v-if="items.length > 0">
        <h2>Staking transactions</h2>
        <b-table striped :items="items"></b-table>
    </div>


  </div>
</template>

<script>
export default {
  name: 'app',
  data () {
    return {
        atomPrice: 0,
        form: {
            address: 'cosmos1pdd93h0u3ats75relnqs99x2mpvkc5mvegn2tt' // cosmos1dw23avhehg6xtw9w2qkhcmy33nwv2wu3whezvh
        },
        msg: 'Welcome to Your Vue.js App',
        items: [
       ]
    }
  },

  methods: {
      onSubmit(evt) {
        evt.preventDefault()

        var self = this
        // call stargate api to get staking transactions
        this.$http.get("https://stargate.cosmos.network/staking/delegators/" + this.form.address +"/txs")
        .then(function (res) {

            self.items = res.data.map(function(obj) {

                // TODO: consider transactions with more then one message
                var tx = obj.tx.value.msg[0].value;
    
                var atomAmount = (tx.amount.amount / 1000000).toFixed(2);

                return {
                    height: obj.height,
                    atom: atomAmount, 
                    timestamp: obj.timestamp,
                    action: obj.tags[0].value, // TODO: Consider if the action tag is not the first
                    validator: tx.validator_address,
                    actual_usd_value: atomAmount * self.atomPrice + " $"
                }
            })

        })
      },

      getAtomPrice() {

            var url = "https://min-api.cryptocompare.com/data/price?fsym=ATOM&tsyms=USD";
            var self = this   
            this.$http.get(url).then(function(r) {
                self.atomPrice = r.data.USD;
            })
      }
  },

  mounted() {
      // Get actual Atom CmC Price
      this.getAtomPrice()
  }

}
</script>

<style lang="scss">
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

h1, h2 {
  font-weight: normal;
}

ul {
  list-style-type: none;
  padding: 0;
}

li {
  display: inline-block;
  margin: 0 10px;
}

a {
  color: #42b983;
}
</style>
