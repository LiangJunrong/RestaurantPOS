<template>
  <div class="pos">
      <div>
        <el-row>
          <el-col :span='7' class="pos-order" id="order-list">
            <el-tabs class="style1">
              <el-tab-pane label="点餐">
                <el-table :data="tableData">
                  <el-table-column prop="goodsName" label="商品名称"></el-table-column>
                  <el-table-column prop="count" label="数量"></el-table-column>
                  <el-table-column prop="price" label="金额"></el-table-column>
                  <el-table-column label="操作" width="100" fixed="right">
                    <template slot-scope="scope">
                      <el-button type="text" size="small" @click="addOrderList(scope.row)">增加</el-button>
                      <el-button type="text" size="small" @click="delSingleGoods(scope.row)">删除</el-button>
                    </template>
                  </el-table-column>
                </el-table>
                <div class="totalDiv">
                  <small>数量</small>：<span>{{totalCount}}</span>&nbsp;&nbsp;&nbsp;&nbsp;
                  <small>金额</small> <span>{{totalMoney}}元</span>
                </div>
                <div class="div-btn">
                  <el-button type="warning">挂单</el-button>
                  <el-button type="danger" @click="delAllGoods">删除</el-button>
                  <el-button type="success" @click="checkout">结账</el-button>
                </div> 
              </el-tab-pane>
              <el-tab-pane label="挂单">
               挂单
              </el-tab-pane>
              <el-tab-pane label="外卖">
               外卖
              </el-tab-pane>
            </el-tabs>
          </el-col>
          <el-col :span="17">
            <div class="often-goods">
              <div class="title">常用商品</div>
              <div class="often-goods-list">
                <ul>
                  <li v-for="goods in oftenGoods" @click="addOrderList(goods)">
                    <span>{{goods.goodsName}}</span>
                    <span class="o-price">￥{{goods.price}}元</span>
                  </li>
                </ul>
              </div>
            </div>
            <div class="goods-type">
              <el-tabs>
                <el-tab-pane label="汉堡">
                  <div>
                    <ul class="cookList">
                      <li v-for="goods in type0Goods" @click="addOrderList(goods)">
                        <span class="foodImg">
                          <img width="100%" :src="goods.goodsImg" alt="">
                        </span>
                        <span class="foodName">{{goods.goodsName}}</span>
                        <span class="foodPrice">￥{{goods.price}}元</span>
                      </li>
                    </ul>
                  </div>
                </el-tab-pane>
                <el-tab-pane label="小食">
                  <div>
                    <ul class="cookList">
                      <li v-for="goods in type1Goods" @click="addOrderList(goods)">
                        <span class="foodImg">
                          <img width="100%" :src="goods.goodsImg" alt="">
                        </span>
                        <span class="foodName">{{goods.goodsName}}</span>
                        <span class="foodPrice">￥{{goods.price}}元</span>
                      </li>
                    </ul>
                  </div>
                </el-tab-pane>
                <el-tab-pane label="饮料">
                  <div>
                    <ul class="cookList">
                      <li v-for="goods in type2Goods" @click="addOrderList(goods)">
                        <span class="foodImg">
                          <img width="100%" :src="goods.goodsImg" alt="">
                        </span>
                        <span class="foodName">{{goods.goodsName}}</span>
                        <span class="foodPrice">￥{{goods.price}}元</span>
                      </li>
                    </ul>
                  </div>
                </el-tab-pane>
                <el-tab-pane label="套餐">
                  <div>
                    <ul class="cookList">
                      <li v-for="goods in type3Goods" @click="addOrderList(goods)">
                        <span class="foodImg">
                          <img width="100%" :src="goods.goodsImg" alt="">
                        </span>
                        <span class="foodName">{{goods.goodsName}}</span>
                        <span class="foodPrice">￥{{goods.price}}元</span>
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
import axios from "axios";
export default {
  name: "pos",
  data() {
    return {
      tableData: [],
      oftenGoods: [],
      type0Goods: [],
      type1Goods: [],
      type2Goods: [],
      type3Goods: [],
      totalMoney: 0,
      totalCount: 0
    };
  },
  created: function() {
    axios
      .get("http://jspang.com/DemoApi/oftenGoods.php")
      .then(reponse => {
        this.oftenGoods = reponse.data;
      })
      .catch(error => {
        alert("网络错误，不能访问");
      });
    axios
      .get("http://jspang.com/DemoApi/typeGoods.php")
      .then(reponse => {
        this.type0Goods = reponse.data[0];
        this.type1Goods = reponse.data[1];
        this.type2Goods = reponse.data[2];
        this.type3Goods = reponse.data[3];
      })
      .catch(error => {
        alert("网络错误，不能访问");
      });
  },
  mounted: function() {
    var orderHeight = document.body.clientHeight;
    document.getElementById("order-list").style.height = orderHeight + "px";
  },
  methods: {
    addOrderList(goods) {
      this.totalCount = 0; //汇总数量为0
      this.totalMoney = 0;
      //商品是否存在于订单列表中
      let isHave = false;
      for (let i = 0; i < this.tableData.length; i++) {
        if (this.tableData[i].goodsId == goods.goodsId) {
          isHave = true;
        }
      }

      //根据判断编写业务逻辑
      if (isHave) {
        //改变列表中商品的数量
        let arr = this.tableData.filter(o => o.goodsId == goods.goodsId);
        arr[0].count++;
      } else {
        let newGoods = {
          goodsId: goods.goodsId,
          goodsName: goods.goodsName,
          price: goods.price,
          count: 1
        };
        this.tableData.push(newGoods);
      }
      this.getAllMoney();
    },
    //删除单个商品
    delSingleGoods(goods) {
      this.tableData = this.tableData.filter(o => o.goodsId != goods.goodsId);
      this.getAllMoney();
    },
    //删除所有商品
    delAllGoods(){
      this.tableData=[];
      this.totalCount=0;
      this.totalMoney=0;
    },
    //汇总数量金额
    getAllMoney() {
      this.totalCount = 0;
      this.totalMoney = 0;
      if (this.tableData) {
        //计算汇总金额和数量
        this.tableData.forEach(element => {
          this.totalCount += element.count;
          this.totalMoney = this.totalMoney + element.price * element.count;
        });
      }
    },
    //模拟结账
    checkout(){
      if(this.totalCount!=0){
        this.tableData=[];
        this.totalCount=0;
        this.totalMoney=0;
        this.$message({
          message:'结账成功！',
          type:'success'
        });
      }else{
        this.$message.error('不能空结~');
      }
    }
  }
};
</script>

<style scoped>
.pos-order {
  background-color: #f9fafc;
  border-right: 1px solid #c0ccda;
}
.style1 {
  padding-left: 10px;
}
.div-btn {
  text-align: center;
  margin-top: 10px;
}
.title {
  height: 20px;
  border-bottom: 1px solid #d3dce6;
  background-color: #f9fafc;
  padding: 10px;
}
.often-goods-list ul li {
  list-style: none;
  float: left;
  border: 1px solid #e5e9f2;
  padding: 10px;
  margin: 10px;
  background-color: #fff;
  cursor: pointer;
}
.o-price {
  color: #58b7ff;
}
.goods-type {
  clear: both;
  padding-left: 10px;
  cursor: pointer;
}
.cookList li {
  list-style: none;
  width: 23%;
  border: 1px solid #e5e9f2;
  height: auto;
  overflow: hidden;
  background-color: #fff;
  padding: 2px;
  float: left;
  margin: 2px;
}
.cookList li span {
  display: block;
  float: left;
}
.foodImg {
  width: 40%;
}
.foodName {
  font-size: 18px;
  padding-left: 10px;
  color: brown;
}
.foodPrice {
  font-size: 16px;
  padding-left: 10px;
  padding-top: 10px;
}
.totalDiv {
  text-align: center;
  background-color: #fff;
  padding: 10px;
  border-bottom: 1px solid #d3dce6;
}
</style>
