<template>
  <div class="home cb w1200">
    <div class="cb">
      <h6 class="title font18 fW600">NULS </h6>
      <ul class="dapp-list">
        <li @click="toUrl('https://wallet.nuls.io/','',1)">
          <p>{{$t('home.home0')}}</p>
          <b class="font12">{{$t('home.home1')}}</b>
        </li>
        <li @click="toUrl('https://nulscan.io/','',1)">
          <p>{{$t('home.home2')}}</p>
          <b class="font12">{{$t('home.home3')}}</b>
        <li @click="toUrl('http://nabox.io/','',1)">
          <p>NABOX</p>
          <b class="font12">NULS APP</b>
        </li>
        <li @click="toUrl('https://github.com/nuls-io','',1)">
          <p>Githup</p>
          <b class="font12">{{$t('home.home4')}}</b>
        </li>
      </ul>
    </div>
    <div class="cb">
      <h6 class="title font18 fW600">{{$t('public.zoology')}}</h6>
      <ul class="dapp-list">
        <li @click="toUrl('https://pocm.nuls.io/','',1)">
          <p>POCM</p>
          <b class="font12">{{$t('home.home5')}}</b>
        </li>
        <li @click="toUrl('https://factory.nuls.io/','',1)">
          <p>{{$t('home.home7')}}</p>
          <b class="font12">{{$t('home.home8')}}</b>
        </li>
        <!-- <li @click="toUrl('tokenSwap','',0)">-->
        <li>
          <p>Token闪兑(开发中....)</p>
          <b class="font12">NULS生态内的NRC20 Tokon闪兑NULS</b>
        </li>
      </ul>
    </div>
    <div class="cb">
      <h6 class="title font18 fW600">{{$t('public.recreation')}}</h6>
      <ul class="dapp-list">
        <li @click="toUrl('goblin','',0)">
          <img src="@/assets/img/goblin-logo.png" width="50">
          <b class="font12">{{$t('goblin.goblin0')}}<br/>{{$t('goblin.goblin1')}}</b>
        </li>
        <li @click="toUrl('guessNumber','',0)">
          <p>{{$t('home.home9')}}</p>
          <b class="font12">{{$t('home.home10')}}</b>
        </li>
        <li>
          <p>{{$t('home.home28')}}<span class="font12 fW400">({{$t('public.design')}}...)</span></p>
          <b class="font12">{{$t('home.home29')}}</b>
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
  import {openConnection} from '@/api/util'

  export default {
    created() {
    },
    mounted() {
    },
    destroyed() {
    },
    components: {},
    computed: {},
    watch: {},
    methods: {

      /**
       * @disc: 连接跳转
       * @param urlName
       * @param parameter
       * @param type  0:路由跳转 1：外部链接
       */
      toUrl(urlName, parameter, type) {
        if (type === 0) {
          this.$router.push({
            name: urlName
          })
        } else {
          openConnection(urlName);
        }
      },

    }
  }
</script>
<style lang="less">
  @import "./../assets/css/style";

  .home {
    min-height: 800px;
  }
</style>
