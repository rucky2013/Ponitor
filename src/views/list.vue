  <template>
    <div id="allcontainer">
     <p class="loading" v-show="goods.length === 0">该分类没有商品</p>
     <div class="pure-g" v-else>
      <div class="pure-u-1-6" v-for="good in goods">
        <div class="good">
          <div class="icon">
            <img v-bind:src="good.image" alt="{{ good.name}}" class="pure-img"  title="{{ good.name}}">
          <div class="name" title="{{ good.name}}">{{ good.name | asLength 12}}<br/><a href="{{good.url}}" target="_blank" style="cursor:pointer">详情</a>&nbsp;<a style="cursor:pointer"  @click="showChart(good._id)">趋势</a></div> 
          <div class="price">{{ good.priceText }}</div> 
          </div>
          </div>
       </div>
     </div>
    </div>
    <nv-chart :show-chart-modal.sync="showChartModal" ></nv-chart>
  </template>

<script>
import request from 'superagent'
import notie from 'notie'
import nprogress from 'nprogress'
export default {
  data () {
    return { 
    	goods:[],
        showChartModal:false,
        adding:false,
        goodUrl:''
     }
  }, 
    route: {
      data(transition){
        const type=transition.to.query.type;
        request
          .post('/api/good/'+type)
          .end((err, res) => {
            nprogress.done()
            if (err) {
              notie.alert(3, err.message, 1.5)
            } else {
              this.goods = res.body.data;
              if(res.body.status===403){
                  localStorage.clear();
              }
            }
          })
      }
    },
  ready () {
    this.handle = setInterval(() => {
      this.count++
    }, 1000)
  },
  methods:{
    showChart:function(gid){
      request
        .post('/api/chart/'+gid)
        .end((err, res) => {
          nprogress.done()
          if (res.body.result_code===-1) {
            notie.alert(3, res.body.error, 1.5)
          } else if(err){
            notie.alert(3, err.message, 1.5)
          }else{
             this.$data.showChartModal=true;
             this.$data.chartOption = res.body;
             Ponitor.createEchartsLine("priceChart",this.$data.chartOption);

          }
        })
    }
  },
    components:{
      'nvChart':require('../components/charts.vue')
    },
  destroyed () {
    clearInterval(this.handle)
  }
}
</script>


<style>

#allcontainer{
   margin:0 240px 240px;
	.loading{
		margin-top: 20px;
	    color: #393;
	    text-align: center;
	    font-size: 0.8em;
    }
   .good{
      text-align: center;
    .icon{
      padding: 30px;
      padding-bottom: 10px;
      img{
        border-radius: 20px;
      }
    }
    .name{
      font-size: 16px;
    }
    .price{
      font-size: 0.8em;
      margin-top: .2em;
      background-color: #DCDCDC;
    }
  }
}
</style>
