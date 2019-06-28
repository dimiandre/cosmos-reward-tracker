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

    <div class="mt-5" v-if="stats">
        <h2>Stats</h2>


        <b-card-group deck>
            <b-card title="Delegations" v-if="shares > 0">
                <b-card-text>
                    You currently have <br />
                    <big>{{ shares }} ATOMs delegated</big>
                </b-card-text>
                <div slot="footer"><small class="text-muted">Last updated 1 mins ago</small></div>
            </b-card>

            <b-card title="Estimated Reward" v-if="reward > 0">
                <b-card-text>
                    If you keep all your ATOMs delegated in one year you will get around <br /> 
                    <big>{{ reward }} ATOMs</big>
                </b-card-text>
                <div slot="footer"><small class="text-muted">Last updated 1 mins ago</small></div>
            </b-card>

            <b-card title="Global Stats" v-if="bonded_atoms > 0">
                <b-card-text>
                    Total bonded ATOMs: {{ bonded_atoms }} <br />
                    Total not bonded ATOMs: {{ not_bonded_atoms }}<br />
                    Inflation: {{ inflation }}
                </b-card-text>
                <div slot="footer"><small class="text-muted">Last updated 1 mins ago</small></div>
            </b-card>
        </b-card-group>
        
    </div>

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
        bonded_atoms: 0,
        not_bonded_atoms : 0,
        shares: 0,
        inflation: 0,
        reward: 0,
        stats: false,
        form: {
            address: 'cosmos1dw23avhehg6xtw9w2qkhcmy33nwv2wu3whezvh' // cosmos1pdd93h0u3ats75relnqs99x2mpvkc5mvegn2tt
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
    
                var atomAmount = self.convertMeasure(tx.amount.amount);

                return {
                    height: obj.height,
                    atom: atomAmount, 
                    timestamp: obj.timestamp,
                    action: obj.tags[0].value, // TODO: Consider if the action tag is not the first
                    validator: tx.validator_address,
                    actual_usd_value: atomAmount * self.atomPrice + " $"
                }
            })

            // Calculate statistics

            // Get actual delegator delegations
            self.getDelegations()

            // Get atom stats
            self.getAtomStats()

            self.stats = true

        })
      },

      convertMeasure(atoms) {
          return (atoms / 1000000).toFixed(2)
      },

      getAtomPrice() {

            var url = "https://min-api.cryptocompare.com/data/price?fsym=ATOM&tsyms=USD";
            var self = this   
            this.$http.get(url).then(function(r) {
                self.atomPrice = r.data.USD;
            })
      },

      getDelegations() {
          var self = this

            // call stargate api to get staking delegations
            this.$http.get("https://stargate.cosmos.network/staking/delegators/" + this.form.address +"/delegations")
            .then(function (res) {
                
                // Calculate total delegations
                var shares = 0;
                res.data.forEach(function(el) {
                    shares += parseFloat(el.shares);
                });

                self.shares = self.convertMeasure(shares);
                self.reward = (self.shares * 0.12).toFixed(2);
            })
      },

      getAtomStats() {

          // First get stake pool status
          
           var self = this

            // call stargate api to get staking delegations
            this.$http.get("https://stargate.cosmos.network/staking/pool")
            .then(function (res) {
                self.bonded_atoms = self.convertMeasure(res.data.bonded_tokens)
                self.not_bonded_atoms = self.convertMeasure(res.data.not_bonded_tokens)
            })

            // Get inflation
             this.$http.get("https://stargate.cosmos.network/minting/inflation")
            .then(function (res) {
                self.inflation = (res.data*100).toFixed(2) + " %"
            })
      }
  },

  mounted() {
      // Get actual Atom Price
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
