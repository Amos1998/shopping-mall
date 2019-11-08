<template>
    <div class="order_main">
      <div class="order_top">
        <span class="order_item" :class="{order_curr:'allOrders' == curr}" mark="allOrders" @click="fn('allOrders')">全部订单</span>
        <span class="order_item" :class="{order_curr:'pendingPayment' == curr}" @click="fn('pendingPayment')">待付款</span>
        <span class="order_item" :class="{order_curr:'pendingReceipt' == curr}" @click="fn('pendingReceipt')">待收货</span>
      </div>

      <el-row class="order_list">
        <el-col :md="11" class="car_information">订单详情</el-col>
        <el-col :md="4" class="car_information">收货人</el-col>
        <el-col :md="5" class="car_canshu">金额</el-col>
        <el-col :md="4" class="car_operation">操作</el-col>
      </el-row>
      <template v-if="orderManageInfo.length ==  0">
        <div class="order_empty">
          没有任何订单哦，快去挑选您喜欢的东西吧.
        </div>
      </template>
      <template v-else>
        <div v-if="orderManageInfo.length != 0">
          <div v-for="(product,product_index) in orderManageInfo" class="product" :key="product_index"
          >
            <div class="order_time">
              <span class="order_time_l">{{product.AddTime|dateFormatter}}</span>
              <span class="order_time_r" v-if="is_pay == 0"><i class="el-icon-delete"></i></span>
            </div>
            <el-row class="product_details" >
              <!--商品图片-->
              <el-col :md="4" class="product_img" >
                <img :src="product.thumb_url" style="width: 100px;height: 100px;cursor: pointer;" @click.stop="switchUrl(product.pro_type,product.pro_stype,product.goods_id)"/>
              </el-col>

              <!--商品详情-->
              <el-col :md="4" class="product_order_info">
                <div class="product_order_name">
                  <a @click.stop="switchUrl(product.pro_type,product.pro_stype,product.goods_id)">{{product.goods_name}}</a>
                </div>
                <div>

                </div>
                <div style="width: 100%;text-align: left;">
                  颜色: {{product.color_type}}
                </div>
                <br />
                <div v-if="product.length_type">
                  长度: {{product.length_type}}
                </div>
              </el-col>

              <!--商品数量-->
              <el-col :md="3" class="product_number">
                X{{product.buy_count}}
              </el-col>
              <!--收货人-->
              <el-col :md="4" class="product_receiver">
                <div class="receiver">Amos
                  <el-tooltip class="item" effect="light" placement="left-start" offset="0">
                    <div slot="content">用户名<br/>地址<br />手机号</div>
                  <i class="el-icon-user"></i>
                </el-tooltip>

          </div>
              </el-col>

              <!--金额统计-->
              <el-col :md="5" class="product_amount">
                <div class="product_total_price">
                  总额 ￥{{ priceAmount(product) | priceFilter}}
                  <br>
                  <span>在线支付</span>
                </div>
              </el-col>

              <!--操作选项-->
              <el-col :md="4" class="product_operation">
                <div v-if="is_pay == 2">
                  <el-button type="danger">退货</el-button>
                </div>
                <div v-else-if="is_pay == 0">
                  <el-button type="primary">购买</el-button>
                </div>
                <div v-else-if="is_pay == 1">
                  <el-button type="primary">确定收货</el-button>
                </div>
              </el-col>
            </el-row>
          </div>
        </div>
      </template>
    </div>
</template>

<script>
    import {mapState,mapActions} from 'vuex'

    export default {
        name: "Order_management",
        data(){
            return {
                //当前选中项 (与mark对应)
                curr:"allOrders",
                is_pay: 2
            }
        },
        computed: {
            ...mapState(['orderManageInfo']),
        },
        mounted() {
            this.$store.dispatch('getOrderManageInfo',{is_pay:this.is_pay});
        },
        methods:{
            switchUrl(pro_type,pro_stype,goods_id){
                this.$router.push('/Shopping_info?pro_type='+pro_type+'&pro_stype='+pro_stype+'&goods_id='+ goods_id)
            },
            //选择展示内容
            fn(mark){
                this.curr = mark;
                if( mark == 'pendingPayment'){
                    this.is_pay = 0;//未付款
                } else if(mark == 'pendingReceipt'){
                    this.is_pay = 1;//已付款，发货中
                } else if(mark == 'allOrders'){
                    this.is_pay = 2;//已收货
                }
                this.$store.dispatch('getOrderManageInfo',{is_pay:this.is_pay});
            },
            //商品价格
            priceAmount(goods){
                return parseInt(goods.price) * parseInt(goods.buy_count)
            },
        },
        filters: {
            // 金额显示过滤 两位小数点，不足补0
            priceFilter(money) {
                return money.toFixed(2)
            },
            //参数是后台返回的Object对象，不是字符串
            dateFormatter(time){
                let zoneDate = new Date(time).toJSON();
                let date = new Date(+new Date(zoneDate)+8*3600*1000).toISOString().replace(/T/g,' ').replace(/\.[\d]{3}Z/,'');
                return date;
          }
        },
    }
</script>
<style>
  .el-tooltip__popper.is-light{
    border: 1px solid #ddd;
    background: #fff;
    box-shadow: 0 0 2px 2px #eee;
    border-radius: 1px;
  }
</style>
<style scoped>
  .order_main {
    position: relative;
    width: 980px;
    margin: 0px auto;
    padding: 0px 0px;
    min-height: 250px;
  }
  .order_top{
    margin: 20px 0;
    padding-left: 20px;
  }
  .order_top .order_item{
    padding: 0 5px;
    cursor: pointer;
    margin-right: 20px;
  }
  .order_top .order_item.order_curr{
    padding-bottom: 0;
    color: #e4393c;
    border-bottom: 2px solid #e4393c;
    font-weight: 700;
    text-decoration: none;
  }
  .order_list{
    height: 32px;
    line-height: 32px;
    text-align: center;
    background: #f5f5f5;
    color: #666;
    font-weight: 400;
  }
  .order_empty {
    position: absolute;
    width: 100%;
    top: 50%;
    text-align: center;
  }

  /*订单列表*/
  .product{
    border: 1px solid #e5e5e5;
    margin-top: 20px;
  }
  .order_time{
    background: #f5f5f5;
    height: 36px;
    line-height: 36px;
    color: #aaa;
    overflow: hidden;
    padding: 0 20px;
  }
  .order_time .order_time_l{
    margin-left: 20px;
  }
  .order_time_r{
    float: right;
    width: 50px;
    text-align: center;
    cursor: pointer;
    display: none;
  }
  .product:hover .order_time_r{
    display: block;
  }
  .order_time_r .el-icon-delete{
    color: #151414;
  }
  .product_details {
    /*padding: 15px 0;*/
  }
  .product_details::before
  ,.product_details::after{
    display: table;
    content: " ";
  }

  .product_img {
    /*padding-top: 20px;*/
  }
  .product_order_info,
  .product_total_price,
  .product_receiver,
  .product_number,
  .product_operation{
    box-sizing: border-box;
    /*text-indent: 2rem;*/
    line-height: 25px;
    padding: 10px;
  }

  .product_order_name {
    font-size: 12px;
    text-align: left;
  }

  .product_order_name a {
    cursor: pointer;
    text-decoration: none;
    /*letter-spacing: 2px;*/
    color: #3c3c3c;
    font-size: 18px;
    font-weight: 500;
  }

  .product_order_name a:hover {
    color: red;
    text-decoration: underline;
  }
  .product_number{
    color: #aaa;
  }
  .product_total_price {
    width: 160px;
    font-size: 16px;
    margin: 0 auto;
    color: #AAA;
    font-weight: bold;
    word-wrap: break-word;
  }
  .product_total_price>span{
    display: block;
    margin: 5px 8px 0;
    padding-top: 2px;
    border-top: solid 1px #E5E5E5;
    line-height: 19px;
    height: 19px;
    width: 80%;
  }
  .el-icon-user{
    color: #aaa;
  }
  .product_receiver .receiver{
    font-size: 20px;
  }

  .product_img,
  .product_order_info,
  .product_total_price,
  .product_receiver,
  .product_number,
  .product_operation{
    text-align: center;
    padding: 15px 0;
    height: 150px;
    display: flex;
    align-items: center;
    flex-wrap: wrap;
    align-content:center;
  }
  .product_total_price,
  .product_receiver,
  .product_operation {
    justify-content: center;
  }
  .product_receiver,
  .product_amount,
  .product_operation{
    border-left: 1px solid #e5e5e5;
  }

</style>
