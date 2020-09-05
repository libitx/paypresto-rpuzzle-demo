<template>
  <div>
    <h2 class="mt0 mb3 f4 lh-title">2. Broadcast the locking script</h2>
    <p class="mt0 mb3 f5 lh-copy">
      Now we get to use Paypresto. Using the <code class="code bg-light-gray ph2 pv1">P2RPH</code>
      cast, we can create the R-Puzzle by passing the <code class="code bg-light-gray ph2 pv1">R</code>
      value to the locking script.
    </p>
    <p class="mt0 mb3 f5 lh-copy">
      Click the button to create the R-Puzzle.
    </p>

    <div class="mb1 f7 lh-copy gray">Broadcast locking transaction</div>
    <pre class="mt0 mb3 pa3 f6 lh-copy near-white bg-dark-gray br2" style="white-space: pre-wrap"><code>import { Presto } from 'paypresto.js'
import { Cast } from 'txforge'
import { P2RPH } from 'txforge/casts'

const payment = Presto.create({
  key: privKey,
  description: 'Paypresto R-Puzzle Locking TX',
  outputs: [
    Cast.lockingScript(P2RPH, { satoshis: 1000, rBuf })
  ]
})

payment
  .on('funded', _ => payment.pushTx())</code></pre>

    <div v-if="store.status === 1">
      <a class="dib pa3 link white bg-blue dim pointer"
        @click="lockPuzzle">
        Broadcast R-Puzzle
      </a>
    </div>
    <div v-else>
      <div class="mb1 f7 lh-copy gray">Locking UTXO</div>
      <pre class="mt0 mb3 pa2 f6 lh-copy near-white bg-dark-gray br2" style="white-space: pre-wrap">{{ store.lockingUTXO }}</pre>
    </div>

    <transition name="fade">
      <SlideOver
        ref="slide"
        v-show="showSlide"
        @close="showSlide = false" />
    </transition>

  </div>
</template>

<script>
import { Presto } from 'paypresto.js'
import { Cast } from 'txforge'
import { P2RPH } from 'txforge/casts'
import SlideOver from 'components/SlideOver.vue'
import keystore from 'keystore'
import store from 'store'

export default {
  data() {
    return {
      store,
      invoiceId: null,
      showSlide: false
    }
  },

  computed: {
    rBuf() {
      return Buffer.from(this.store.rHex, 'hex')
    }
  },

  methods: {
    lockPuzzle() {
      if (!this.invoiceId) {
        const cast = Cast.lockingScript(P2RPH, { satoshis: 1000, rBuf: this.rBuf })
        const payment = Presto.create({
          key: keystore.privKey,
          description: 'Paypresto R-Puzzle Locking TX',
          outputs: [cast]
        })

        setTimeout(_ => {
          this.$refs.slide.mount(payment)
        }, 10)

        payment
          .on('invoice', invoice => {
            console.log('locking script invoiced')
            this.invoiceId = invoice.id
          })
          .on('funded', payment => {
            console.log('locking script funded')
            payment.pushTx()
          })
          .on('success', ({ txid }) => {
            console.log('locking script pushed')
            this.store.lockingTXID = txid
            this.store.lockingUTXO = {
              txid,
              vout: 0,
              satoshis: 1000,
              script: cast.getScript().toHex()
            }
            this.store.status = 2
          })
          .on('error', err => {
            console.error(err)
            alert('Something went wrong. Check browser console.')
          })
      }
      this.showSlide = true
    }
  },

  components: {
    SlideOver
  }
}
</script>