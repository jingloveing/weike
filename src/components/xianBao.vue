<template>
  <div>
    <scroller :on-infinite="infinite" :on-refresh="refresh" ref="myscroller">
    <div class="main" style="font-size: 0;">
      <img :src="banner" alt="" style="height: 1.8rem;width: 100%">
      <ul class="timeTab" id="timeTab">
        <li :class="[item.active? activeClass:'']" v-for="(item,index) in time"
            v-on:click="navClickEvent(time,index)">{{item.start_time}}
          <span v-show="item.status==3">已开抢</span>
          <span v-show="item.status==2">抢购中</span>
          <span v-show="item.status==1">待开抢</span>
        </li>
      </ul>
      <!--<router-view></router-view>-->
      <div class="goods_list">
        <ul class="goods">
          <li v-for="(list,index) in goodsList" :key="index" :onerror="defaultImg">
            <!--<router-link :to="{name:'goodsDetail',query:{id:list.id,type:1}}">-->
              <img :src="list.pict_url" alt="" class="pic">
            <!--</router-link>-->
            <div class="content">
              <p class="title" v-text="list.title">产品标题产品标题</p>
              <p class="des" v-text="list.item_description">产品补充介绍</p>
              <div style="margin-top: .35rem;position: relative;">
                <div>
                  <!--<div class="juan">-->
                    <!--<span class="j_num" v-text="list.coupon_number">200</span>-->
                  <!--</div>-->
                  <!--<span style="font-size: .2rem;color: #ff526d;margin-left: .1rem;">折上折</span>-->
                  <span class="juan_style">
                      <span class="juan_style_left">券</span>
                      <span class="juan_style_right">{{list.coupon_number}}元</span>
                    </span>
                </div>
                <div>
                  <span class="new_num"><span style="font-size: .28rem;">￥</span>{{list.zk_final_price.rmb}}<span style="font-size: .20rem;" v-show="list.zk_final_price.corner!=='00'">.{{list.zk_final_price.corner}}</span></span>
                  <del class="old_num">￥{{list.reserve_price.rmb}}<span v-show="list.reserve_price.corner!=='00'">.{{list.reserve_price.corner}}</span></del>
                </div>
                <div style="position: absolute;right: 0;bottom: .05rem;text-align: center;">
                  <p style="font-size: .24rem;color: #ff526d;">已抢{{list.volume}}件</p>
                  <!--<router-link :to="{name:'goodsDetail',query:{id:list.id,type:1}}">-->
                     <span class="togo" :class="list.not_start=='1'?'not_start':''" @click="toBuy(list.id,list.not_start)">立刻抢 <img
                       src="../assets/lt_white.png" alt="" style="width: .10rem;height: .16rem;"></span>
                  <!--</router-link>-->

                </div>
              </div>
            </div>
          </li>
        </ul>
      </div>
    </div>
    </scroller>
    <loading v-model="showLoading" :text="loadText"></loading>
  </div>
</template>
<script>
//  import Vue from 'vue'
  import {Loading} from 'vux'
  export default {
    name: 'xianBao',
    components: {
      Loading
    },
    data: function () {
      return {
        showLoading:false,
        loadText:'加载中...',
        banner:'',
        activeClass: 'active',
        time:[],
        panic_id:'',
        goodsList:[],
        defaultImg: 'this.src="' + require('../../static/images/default_img.png') + '"',
        pageIndex:1,
        limit:10,
        totalPage:0,
        noData: false,
      }
    },
    methods: {
      navClickEvent: function (time, index) {
        /*默认切换类的动作*/
        time.forEach(function (el) {
          el.active = false;
        });
        time[index].active = true;
        this.panic_id=time[index].panic_id
         this.goodsList=[]
        this.pageIndex=1
        this.showLoading=true
        this.getGoodsList()
      },
      //      获取超值线报banner
      getBanner:function(){
//        this.showLoading=true
        this.$http({
          method:'POST',
          url:'/api/newspaper_banner'
        }).then((res)=>{
          if(res.data.code==200){
            this.showLoading=false
            const banner = res.data.data.banner[0].banner_image;
            this.banner = banner;
          }else{

          }
        },(err)=>{
          console.log(err)
        })
      },
      //      获取超值线报抢购时间
      getTime:function(){
//        this.showLoading=true
        this.$http({
          method:'POST',
          url:'/api/newspaper_time'
        }).then((res)=>{
          if(res.data.code==200){
            this.showLoading=false
            const time = res.data.data.time;
            this.time = time;
          }else{

          }
        },(err)=>{
          console.log(err)
        })
      },
      //      获取超值线报抢购商品
      getGoodsList:function(){
//        this.showLoading=true
        this.$http({
          method:'get',
          url:'/api/overflow',
          params:{panic_id:this.panic_id,page:this.pageIndex,limit:this.limit}
        }).then((res)=>{
          this.showLoading=false
          if(res.data.code==200){
            if(res.data.data.goods_list.length==0){
              this.noData=true
              this.$refs.myscroller.finishInfinite(2);
            }else{
              this.goodsList=this.goodsList.concat(res.data.data.goods_list)
              this.$refs.myscroller.finishPullToRefresh()
            }

          }
        },(err)=>{
          console.log(err)
        })
      },
      infinite(done) {
        if (this.noData) {
          setTimeout(() => {
            this.$refs.myscroller.finishInfinite(2);
          })
          return;
        }
        else{
          let self = this;//this指向问题
          setTimeout(()=>{
            self.pageIndex += 1
            self.getGoodsList()
            done()
          },1500)
        }

      },
      refresh(done) {
        var self = this
        this.pageIndex=1
        this.goodsList=[]
        this.getBanner()
        this.getTime()
        this.getGoodsList()
        setTimeout(function () {
          self.top = self.top - 10;
          done()
        }, 1500)
      },
      toBuy(id,type){
        if(type=='1'){
          return
        }else{
          this.$router.push({name:'goodsDetail',query:{id:id,type:1}})
        }
      }
    },
    mounted(){

    },
    created:function(){
      this.showLoading=true
      this.getBanner()
      this.getTime()
      this.getGoodsList()
    }
  }

</script>
<style scoped>
  ul{
    font-size: 0;
    overflow: hidden;
  }

  .timeTab li {
    width: 25%;
    font-size: .28rem;
    text-align: center;
    padding: .15rem 0;
    list-style: none;
    float: left;
    color: white;
    background-color: #414141;
  }

  .active {
    background-color: #ff526d !important;
  }

  .goods {
    background-color: white;
  }

  .goods li {
    overflow: hidden;
    padding: .22rem .2rem;
    border-bottom: 1px solid #f4f4f4;
  }

  .pic {
    width: 2.4rem;
    height: 2.24rem;
    float: left;
  }

  .content {
    width: calc(100% - 2.4rem - .44rem);
    padding: 0 .22rem;
    display: inline-block;
  }

  .title {
    font-size: .32rem;
    color: #333333;
    height: .5rem;
    overflow: hidden;
  }

  .des {
    font-size: .24rem;
    color: #ff526d;
    height: .38rem;
    overflow: hidden;
  }

  .juan {
    display: inline-block;
    width: 47px;
    height: 14px;
    background-image: url('../assets/juan.png');
    background-size: 100% 100%;
    background-repeat: no-repeat;
  }

  .j_num {
    font-size: 10px;
    color: #ff526d;
    line-height: 14px;
    margin-left: 22.5px;
  }

  .new_num {
    font-size: .4rem;
    color: #ff526d;
  }

  .old_num {
    font-size: .2rem;
    color: #999999;
    margin-left: .1rem;
  }
  .togo{
    border-radius: .5rem; margin-top:.05rem;width: 1.16rem;font-size: .24rem;color: white;background-color: #ff526d;line-height: .46rem;display: inline-block;
  }
  .not_start{
    background-color: #999;
  }
</style>
