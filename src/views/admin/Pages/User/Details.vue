<template>
  <div class="container-fluid">
    <div class="ad_nav_tabs">
      <ul>
        <li :class="{ is_active: comName === 'Details' }" @click="setCom('Details')">Profile</li>
        <li :class="{ is_active: comName === 'Purchased' }" @click="setCom('Purchased')">Purchased</li>
        <li :class="{ is_active: comName === 'Liked' }" @click="setCom('Liked')">Liked</li>
        <li :class="{ is_active: comName === 'Transaction' }" @click="setCom('Transaction')">Transaction</li>
      </ul>
    </div>
    <div>
      <component :is="comNameComputed" :elmntId="comDataComputed" />
    </div>
  </div>
</template>

<script>
import Details from '../../../../components/admin/User/Details'
import Purchased from '../../../../components/admin/User/Purchased'
import Liked from '../../../../components/admin/User/Liked'
import Transaction from '../../../../components/admin/User/Transaction'
export default {
  components: {
    Details,
    Purchased,
    Liked,
    Transaction
  },
  data() {
    return {
      comName: 'Details',
      elmntId: this.$route.params.id
    }
  },
  computed: {
    comNameComputed() {
      return this.comName
    },
    comDataComputed() {
      if (this.elmntId) return parseInt(this.elmntId)
      else return 0
    }
  },
  methods: {
    setCom(comName) {
      this.comName = comName
    }
  }
}
</script>

<style lang="scss" scoped>
.ad_nav_tabs {
  ul {
    display: flex;
    list-style-type: none;
    padding-inline-start: 0;
    li {
      margin: 0.5em 0.5em 0 1em;
      cursor: pointer;
      padding: 0 0.5em;
      background-color: #FFFFFF;
      min-width: 110px;
      text-align: center;
      box-shadow: 3px 3px 0 rgb(0 0 0 / 10%);
      font-size: 20px;
      font-weight: bold;
    }

    li:active {
      transform: translateY(2px);
    }

    .is_active {
      background-color: wheat;
    }

    .is_active::after {
      content: ' 	●';
      color: pink;
    }

  }
  .ad_nav_tabs_items {
    margin: 1em;
  }
}
</style>
