<template>
  <div class="mw7 center pv4">
    <h1 class="mt0 mb4 f2 lh-title">Paypresto R-Puzzle demo</h1>
    <p class="mt0 mb3 f5 lh-copy">
      This page demonstrates how Paypresto can be used to both create and spend
      R-Puzzle transactions, using <strong>any</strong> consumer wallet.
    </p>
    <p class="mt0 mb3 f5 lh-copy">
      Because Paypresto is built on top of TxForge, it can be used to create and
      spend any custom transaction type or smart contract. Check out both
      libraries:
    </p>
    <ul class="mt0 mb3 f5 lh-copy">
      <li><a href="http://www.paypresto.co" class="link blue hover-hot-pink">Paypresto</a></li>
      <li><a href="https://github.com/libitx/txforge" class="link blue hover-hot-pink">TxForge</a></li>
    </ul>
    <p class="mt0 mb3 f5 lh-copy">
      Follow the steps below to create and then redeem an R-Puzzle.
    </p>

    <k-value class="mv4" />
    <locking-script class="mv4" v-show="store.status >= 1" />
    <unlocking-script class="mv4" v-show="store.status >= 2" />
  </div>
</template>

<script>
import KValue from 'components/KValue.vue'
import LockingScript from 'components/LockingScript.vue'
import UnlockingScript from 'components/UnlockingScript.vue'
import keystore from 'keystore'
import store from 'store'

export default {
  data() {
    return {
      store
    }
  },

  created() {
    keystore.init()
  },

  components: {
    KValue,
    LockingScript,
    UnlockingScript
  }
}
</script>