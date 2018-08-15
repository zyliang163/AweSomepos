<template>
<div class="pos">
  <div>
      <el-row >
          <el-col :span='7' class='pos-order' id='order-list'>
            <el-tabs>
              <el-tab-pane label='点餐'>
              <el-table :data='tableData' border style='100%'>
                    <el-table-column prop='goodsName' label='商品名称'></el-table-column>
                    <el-table-column prop='count' label='数量'></el-table-column>
                    <el-table-column prop='price' label='金额'></el-table-column>
                    <el-table-column  label='操作' fixed='right'>
                      <template scope='scope'>
                          <el-button type='text' size='small' @click='delSingleGoods(scope.row)'>删除</el-button>
                          <el-button type='text' size='small' @click='addOrderList(scope.row)'>增加</el-button>
                      </template>
                    </el-table-column>
              </el-table>
              <div class='totalDiv'>
                  数量:{{totalCount}}  &nbsp;&nbsp;&nbsp;&nbsp;  金额:{{totalMoney}}
              </div>
              <div  class='div-btn'>
              <el-button type='warning' >挂单</el-button>
              <el-button type='danger' @click='delAll()'>删除</el-button>
              <el-button type='success' @click='checkout()'>结账</el-button>
              </div>
              </el-tab-pane>
              <el-tab-pane label='挂单'>
              挂单
              </el-tab-pane>
              <el-tab-pane label='外卖'>
              外卖
              </el-tab-pane>
            </el-tabs>
          </el-col>
          <!--商品展示-->
          <el-col :span="17">
             <div class='often-goods'>
               <div class='title'>常用商品</div>
              <div class='often-goods-list'>
                <ul>
                    <li v-for='item in oftenGoods' @click='addOrderList(item)'>
                          <span>{{item.goodsName}}</span>
                          <span class='o-price'>Y {{item.price}}元</span>
                    </li>
                </ul>
              </div>
             </div>

             <div class='goods-type'>
              <el-tabs>
                <el-tab-pane label='汉堡'>汉堡
                  <div>
                  <ul class='cookList'>
                     <li v-for='item in type0Goods' @click='addOrderList(item)'>
                         <span class="foodImg"><img :src="item.goodsImg" width="100%"></span>
                         <span class="foodName">{{item.goodsName}}</span>
                         <span class="foodPrice">￥{{item.price}}元</span>
                    </li>
                  </ul>
                  </div>
                </el-tab-pane>
                <el-tab-pane label='小食'>
                  <div>
                      <ul class='cookList'>
                          <li v-for='item in type1Goods' @click='addOrderList(item)'>
                              <span class="foodImg"><img :src="item.goodsImg" width="100%"></span>
                              <span class="foodName">{{item.goodsName}}</span>
                              <span class="foodPrice">￥{{item.price}}元</span>
                          </li>
                      </ul>
                  </div>
                </el-tab-pane>
                <el-tab-pane label='饮料'>
                  <div>
                      <ul class='cookList'>
                          <li v-for='item in type2Goods' @click='addOrderList(item)'>
                              <span class="foodImg"><img :src="item.goodsImg" width="100%"></span>
                              <span class="foodName">{{item.goodsName}}</span>
                              <span class="foodPrice">￥{{item.price}}元</span>
                          </li>
                      </ul>
                  </div>
                </el-tab-pane>
                <el-tab-pane label='套餐'>
                  <div>
                      <ul class='cookList'>
                          <li v-for='item in type3Goods' @click='addOrderList(item)'>
                              <span class="foodImg"><img :src="item.goodsImg" width="100%"></span>
                              <span class="foodName">{{item.goodsName}}</span>
                              <span class="foodPrice">￥{{item.price}}元</span>
                          </li>
                      </ul>
                  </div>
                </el-tab-pane>
              </el-tabs>
             </div>
          </el-col>
      </el-row>
  </div>
</div>
</template>

<script>
import axios from 'axios'
export default {
  name: 'HelloWorld',
  data(){
    return {
    tableData: [],
     oftenGoods:[],
     type0Goods:[],
     type1Goods:[],
     type2Goods:[],
     type3Goods:[],
     totalMoney:0,
     totalCount:0,
    }

  },
  created:function(){
    axios.get('http://jspang.com/DemoApi/oftenGoods.php')
      .then(res=>{
      this.oftenGoods=res.data;
      })
      .catch(error=>{
      console.log(error);
      });
      axios.get('http://jspang.com/DemoApi/typeGoods.php')
        .then(res=>{
        this.type0Goods=res.data[0];
        this.type1Goods=res.data[1];
        this.type2Goods=res.data[2];
        this.type3Goods=res.data[3];
        })
        .catch(error=>{
        console.log(error);
        });
  }
  ,
  mounted:function(){
  var height=document.body.clientHeight;
  console.log(height);
  document.getElementById('order-list').style.height=height+'px';
  },
  methods:{
      addOrderList(goods){
            this.totalMoney=0;
            this.totalCount=0;
          //判断是否存在商品列表
            let isHave=false;
            for(let i=0;i<this.tableData.length;i++){
                if(this.tableData[i].goodsId==goods.goodsId){
                         isHave=true;
                }
            }
          //编写业务逻辑
          if(isHave){
            //改变列表中商品数量
            let arr=this.tableData.filter(o=>o.goodsId==goods.goodsId);
            console.log(arr);
            arr[0].count++;
          }else{
          let newGoods={goodsId:goods.goodsId,goodsName:goods.goodsName,price:goods.price,count:1};
          this.tableData.push(newGoods);
          }
          this.getAllMoney();

      },
      //删除单个商品
      delSingleGoods(goods){
       this.tableData=this.tableData.filter(o=>o.goodsId!=goods.goodsId);
       this.getAllMoney();
      },
      delAll(){
        this.tableData=[];
        this.totalCount=0;
        this.totalMoney=0;
      }
      ,
      //信息汇总
      getAllMoney(){
          this.totalCount=0;
          this.totalMoney=0;
          if(this.tableData){
            //计算汇总金额和数量
             this.tableData.forEach((element)=>{
                this.totalCount+=element.count;
                this.totalMoney+=element.count*element.price;
             });
          }
      },
      //c
      checkout(){
          if(this.totalCount!=0){
              this.tableData=[];
              this.totalCount=0;
              this.totalMoney=0;
              this.$message({
                message:'结账成功,感谢为店里做出了贡献!',
                type:'success',
              });
          }else{
          this.$message.error('不能空结,老板了解你急切的性情!');
          }
      }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.pos-order{
  background-color:#f9fafc;
  border-right:1px solid #c0ccda;
}
.div-btn{
margin-top:20px;
}
.title{
height:20px;
border-bottom:1px solid #D3dce6;
background-color:#f9fafc;
padding:10px;
text-align:left;
}
.often-goods-list ul li{
list-style:none;
float:left;
border:1px solid #f9fafc;
padding:10px;
margin:10px;
background-color:#fff;
}
.o-price{
color:#58B7ff;
}
.goods-type{
clear:both;
}
.cookList li{
       list-style: none;
       width:23%;
       border:1px solid #E5E9F2;
       height: auot;
       overflow: hidden;
       background-color:#fff;
       padding: 2px;
       float:left;
       margin: 2px;
       cursor:pointer;
   }
   .cookList li span{

        display: block;
        float:left;
   }
   .foodImg{
       width: 40%;
   }
   .foodName{
       font-size: 18px;
       padding-left: 10px;
       color:brown;

   }
   .foodPrice{
       font-size: 16px;
       padding-left: 10px;
       padding-top:10px;
   }
   .totalDiv{
    background-color:#fff;
    padding:10px;
    border-bottom:#E5E9F2;
   }
</style>
