<template>
  <div>
    <h2 class="mt0 mb3 f4 lh-title">3. Broadcast the unlocking script</h2>
    <p class="mt0 mb3 f5 lh-copy">
      Great, now we have an R-Puzzle UTXO we can unlock it and spend it.
    </p>
    <p class="mt0 mb3 f5 lh-copy">
      At this point it's fair to say you don't really <em>need</em> Paypresto.
      You can easily create the unlocking transaction with TxForge and send it
      to your favourite miner directly. However, the whole purpose of this demo
      is to show how custom outputs AND inputs can be used with Paypresto, so
      lets crack on.
    </p>
    <p class="mt0 mb3 f5 lh-copy">
      The main difference in this example is the signing of the inputs. We must
      manually build the tx first, and then sign both the R-Puzzle and the
      standard <code class="code bg-light-gray ph2 pv1">P2PKH</code> input that
      Paypresto always adds.
    </p>
    <p class="mt0 mb3 f5 lh-copy">
      Enter your receiving address below and click the button to unlock the R-Puzzle.
    </p>

    <div class="mb1 f7 lh-copy gray">Broadcast unlocking transaction</div>
    <pre class="mt0 mb3 pa3 f6 lh-copy near-white bg-dark-gray br2" style="white-space: pre-wrap"><code>import { Presto } from 'paypresto.js'
import { Cast } from 'txforge'
import { P2RPH } from 'txforge/casts'

const payment = Presto.create({
  key: privKey,
  description: 'Paypresto R-Puzzle Unlocking TX',
  inputs: [
    Cast.unlockingScript(P2RPH, lockingUTXO)
  ],
  changeAddress: changeAddr
})

payment
  .on('funded', _ => {
    payment.forge.build()
    payment
      .signTxIn(0, { kBuf })
      .signTxIn(1, { keyPair: payment.keyPair })
      .pushTx()
  })</code></pre>

    <div v-if="store.status === 2">
      <div class="mb1 f7 lh-copy gray">Enter address to send to</div>
      <input type="text"
        class="input-rest db w-100 mw7 mb3 ph3 pv2 f4 code lh-block ba bw1 b--hot-pink"
        v-model="changeAddr" />

      <a class="dib pa3 link white"
        :class="btnClass"
        @click="unlockPuzzle">
        Redeem R-Puzzle
      </a>
    </div>
    <div v-else>
      <div class="mb1 f7 lh-copy gray">Unlocking TXID</div>
      <pre class="mt0 mb3 pa2 f6 lh-copy near-white bg-dark-gray br2" style="white-space: pre-wrap">{{ store.unlockingTXID }}</pre>

      <div class="ph4 pv3 bg-lightest-blue ba b--light-blue">
        <h2 class="mt0 mb3 f5 lh-title">All done!</h2>
        <p class="mt0 mb3 f5 lh-copy">
          Well done, you have created and spent an R-Puzzle using your favourite
          consumer wallet. You can view the transactions here:
        </p>
        <ul class="mt0 mb3 f5 lh-copy">
          <li><a :href="lockingTxWocLink" target="_blank" class="link blue hover-hot-pink">Locking TX</a></li>
          <li><a :href="unlockingTxWocLink" target="_blank" class="link blue hover-hot-pink">Unlocking TX</a></li>
        </ul>
      </div>
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
      changeAddr: null,
      invoiceId: null,
      showSlide: false
    }
  },

  computed: {
    isReady() {
      if (this.changeAddr && this.changeAddr.match(/^[13][a-km-zA-HJ-NP-Z0-9]{26,33}$/igm)) {
        return true
      } else {
        return false
      }
    },

    btnClass() {
      if (this.isReady) {
        return 'bg-blue dim pointer'
      } else {
        return 'bg-gray'
      }
    },

    kBuf() {
      return Buffer.from(this.store.kHex, 'hex')
    },

    lockingTxWocLink() {
      return `https://whatsonchain.com/tx/${ this.store.lockingTXID }`
    },

    unlockingTxWocLink() {
      return `https://whatsonchain.com/tx/${ this.store.unlockingTXID }`
    }
  },

  methods: {
    unlockPuzzle() {
      if (!this.invoiceId) {
        const cast = Cast.unlockingScript(P2RPH, { ...this.store.lockingUTXO })
        const payment = Presto.create({
          key: keystore.privKey,
          description: 'Paypresto R-Puzzle Unlocking TX',
          inputs: [cast],
          changeAddress: this.changeAddr
        })

        setTimeout(_ => {
          this.$refs.slide.mount(payment)
        }, 10)

        payment
          .on('invoice', invoice => {
            console.log('unlocking script invoiced')
            this.invoiceId = invoice.id
          })
          .on('funded', payment => {
            console.log('unlocking script funded')
            payment.forge.build()
            payment
              .signTxIn(0, { kBuf: this.kBuf })
              .signTxIn(1, { keyPair: payment.keyPair })
              .pushTx()
          })
          .on('success', ({ txid }) => {
            console.log('unlocking script pushed')
            this.store.unlockingTXID = txid
            this.store.status = 3
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