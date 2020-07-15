<template>
  <div>
    <h2 class="mt0 mb3 f4 lh-title">1. Generate a new R-Puzzle</h2>
    <p class="mt0 mb3 f5 lh-copy">
      First thing we must do to create an R-Puzzle is work out our <code class="code bg-light-gray ph2 pv1">K</code>
      and <code class="code bg-light-gray ph2 pv1">R</code> value. Paypresto doesn't do this directly, but the code
      below (adapted from <a href="https://github.com/deanmlittle/rpuzzle" class="link blue hover-hot-pink">Dean Little's rpuzzle</a>
      library) should help.
    </p>
    <p class="mt0 mb3 f5 lh-copy">
      Click the button to generate your <code class="code bg-light-gray ph2 pv1">K</code> and <code class="code bg-light-gray ph2 pv1">R</code> values.
    </p>

    <div class="mb1 f7 lh-copy gray">Create R-Puzzle</div>
    <pre class="mt0 mb3 pa3 f6 lh-copy near-white bg-dark-gray br2" style="white-space: pre-wrap"><code>import { Bn, Point, PrivKey } from 'bsv'

const kBuf = PrivKey.fromRandom().bn.toBuffer(),
      k    = Bn.fromBuffer(kBuf),
      G    = Point.getG(),
      N    = Point.getN(),
      Q    = G.mul(k),
      r    = Buffer.from( Q.x.umod(N).toArray() ),
      rBuf = r[0]>127 ? Buffer.concat([Buffer.alloc(1), r]) : r;
</code></pre>

    <div v-if="store.status === 0">
      <a class="dib pa3 link white bg-blue dim pointer"
        @click="genPuzzle">
        Generate R-Puzzle
      </a>
    </div>
    <div v-else>
      <div class="mb1 f7 lh-copy gray">R value (we'll use this in our locking script)</div>
      <pre class="mt0 mb3 pa2 f6 lh-copy near-white bg-dark-gray br2" style="white-space: pre-wrap">{{ store.rHex }}</pre>
      <div class="mb1 f7 lh-copy gray">K value (we'll use this to solve the puzzle later)</div>
      <pre class="mt0 mb3 pa2 f6 lh-copy near-white bg-dark-gray br2" style="white-space: pre-wrap">{{ store.kHex }}</pre>
    </div>
  </div>
</template>

<script>
import { Bn, Point, PrivKey } from 'bsv'
import store from 'store'

export default {
  data() {
    return {
      store,
      script: null
    }
  },

  methods: {
    genPuzzle() {
      const kBuf = PrivKey.fromRandom().bn.toBuffer(),
            k    = Bn.fromBuffer(kBuf),
            G    = Point.getG(),
            N    = Point.getN(),
            Q    = G.mul(k),
            r    = Buffer.from( Q.x.umod(N).toArray() ),
            rBuf = r[0]>127 ? Buffer.concat([Buffer.alloc(1), r]) : r;
      
      this.store.kHex = kBuf.toString('hex')
      this.store.rHex = rBuf.toString('hex')
      this.store.status = 1
    }
  }
}
</script>