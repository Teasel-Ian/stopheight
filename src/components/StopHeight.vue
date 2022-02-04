<template>
  <div class="stopheight">
    <h1>Calculate stop height for fetchd networks</h1>
    <h2>Chain = {{ chain }}</h2>
    <form @submit.prevent="submitPressed">
      <label for="stoptime">Stop Time</label>
      <input
        type="datetime-local"
        id="stoptime"
        name="stoptime"
        v-model="stoptime"
      />
      <label for="stopheight">Stop Height</label>
      <input
        type="number"
        id="stopheight"
        name="stopheight"
        v-model="stopheight"
      />
    </form>
    <p>RecentHeight: {{ recentheight }}</p>
    <p>RecentTime: {{ recenttime }}</p>
    <p>BlockTime: {{ blocktime }}</p>
    <p>StopHeight: {{ stopheight }}</p>
    <p>StopTime: {{ stoptime }}</p>
  </div>
</template>

<script>
import axios from 'axios'

export default {
  name: 'StopHeight',
  props: {
    chain: String,
  },
  data() {
    return {
      recenttime_unix: 0,
      recentheight: 0,
      stoptime_unix: 0,
      stop_height: 0,
      blocktime: 6.08,
    }
  },
  computed: {
    stoptime: {
      get: function () {
        return new Date(this.stoptime_unix * 1000)
          .toISOString()
          .substring(0, 19)
      },
      set: function (strStopTime) {
        this.stoptime_unix = new Date(strStopTime).getTime() / 1000
        this.stop_height = Math.round(
          this.recentheight +
            (this.stoptime_unix - this.recenttime_unix) / this.blocktime,
        )
      },
    },
    stopheight: {
      get: function () {
        return this.stop_height
      },
      set: function (numStopHeight) {
        var stopheight = parseInt(numStopHeight)
        if (stopheight < this.recentheight) stopheight = this.recentheight
        this.stop_height = stopheight
        this.stoptime_unix =
          this.recenttime_unix +
          (this.stopheight - this.recentheight) * this.blocktime
      },
    },
    recenttime: function () {
      return new Date(this.recenttime_unix * 1000)
        .toISOString()
        .substring(0, 19)
    },
  },
  created: function () {
    // Set sensible defaults
    this.recenttime_unix = 1643875715
    this.recentheight = 4430000
    this.blocktime = 6.08
    this.stopheight = this.recentheight + 1000
  },
  mounted: function () {
    this.fetchrecentchaindata()
  },
  methods: {
    async fetchrecentchaindata() {
      const interval_blocks = 100
      try {
        const response = await axios.get('https://rpc-fetchhub.fetch.ai/status')
        const recentheight = parseInt(
          response.data.result.sync_info.latest_block_height,
        )
        const recenttime_unix =
          new Date(response.data.result.sync_info.latest_block_time).getTime() /
          1000
        this.recentheight = recentheight
        this.recenttime_unix = recenttime_unix
        if (this.stopheight < recentheight)
          this.stopheight = recentheight + 1000

        const previousheight = recentheight - interval_blocks
        const response2 = await axios.get(
          `https://rpc-fetchhub.fetch.ai/block?height=${previousheight}`,
        )
        console.log(response2)
        const previoustime_unix =
          new Date(response2.data.result.block.header.time).getTime() / 1000
        const blocktime =
          (recenttime_unix - previoustime_unix) / interval_blocks
        this.blocktime = blocktime
      } catch (err) {
        console.log(err)
      }
    },
  },
}
</script>
<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
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
