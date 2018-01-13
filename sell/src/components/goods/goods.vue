<template>
  <div class="good-wapper">
    <img src="./img.jpg" alt="goods.title" style="width: 100%;height: 22vh">
    <div class="content">
      <h3>{{goods.title}}</h3>
      <span>{{goods.price}}</span>
      <p>货号：{{goods.goodsno}} | 品牌：{{goods.brand}}</p>
    </div>
    <div class="detail-wrapper">
      <div class="title">商品信息</div>
      <div class="item">
        <label>颜色</label>
        <span>{{colorsString}}</span>
      </div>
      <div class="item">
        <label>尺码</label>
        <span>{{sizesString}}</span>
      </div>
    </div>
    <div class="buy">
      <button class="buyBtn" @click="goBuy">立即购买</button>
    </div>
    <div class="order" v-show="orderShow">
      <strong>{{price}}</strong>
      <table class="gridtable">
        <tr>
          <td>尺码</td>
          <th v-for="size in sizes">{{size}}</th>
          <td>小计</td>
        </tr>
        <tr v-for="(color, index1) in colors">
          <td>{{color}}</td>
          <td v-for="(size,index2) in sizes">
            <input v-model="count[index1][index2]" :id="index1 + '_' + index2"
                   @blur="check(index1,index2,goods.goods[index1].detail[index2].stock)" type="text"
                   @keyup="count[index1][index2]=count[index1][index2].replace(/[^\d]/g,'')"
                   :placeholder="goods.goods[index1].detail[index2].stock">
          </td>
          <td>{{totalCount[index1]}}</td>
        </tr>
      </table>
      <footer class="footer">
        <div class="footer-wrapper total">合计：{{total}}</div>
        <div class="footer-wrapper amount">总价：¥{{amount}}</div>
        <button :disabled="disabledBtn" @click="makeOrder" class="footer-wrapper makeOrder">立即下单</button>
      </footer>
    </div>
  </div>
</template>

<script type="text/ecmascript-6">
  const ERR_OK = 0;
  export default {
    data() {
      return {
        colors: [],
        colorsString: "",
        sizesString: "",
        sizes: [],
        goods: {},
        orderShow: false,
        topSize: -40,
        count: [],
        price: '单价：10元／件',
        totalCount: [],
        amount: 0,
        total: 0,
        disabledBtn: true,
        focusState: false
      }
    },
    created() {
      this.$http.get('/api/data').then((response) => {
        response = response.data;
        if (response.errno === ERR_OK) {
          this.goods = response.data;
          let goods = this.goods.goods;
          let detail = response.data.goods[0].detail;
          goods.forEach(good => {
            this.colors.push(good.color);
          });
          this.colorsString = this.colors.join(',');

          for (let i = 0; i < detail.length; i++) {
            this.sizes.push(detail[i].size);
          }
          this.sizesString = this.sizes.join(",");
//          for (let i in goods) {
//            let temp = {};
//
//            for (let j in goods[i].detail) {
//              temp[goods[i].detail[j].size] = goods[i].detail[j].stock;
//            }
//            this.detailMap[goods[i].color] = temp;
//            console.log(this.detailMap)
//          }
        }
        for (let i = 0; i < this.colors.length; i++) {
          this.count[i] = new Array();
          this.totalCount[i] = 0;
          for (let j = 0; j < this.sizes.length; j++) {
            this.count[i][j] = null;
          }
        }
      });
    },
    methods: {
      goBuy() {
        this.orderShow = true
      },
      getamount(index1) {
        let tempSum = 0;
        let num = 0;
        for (let i = 0; i < this.count[index1].length; i++) {
//          小计
          if (this.count[index1][i]) {
            tempSum += Number(this.count[index1][i]) | 0;
          }
          ;
          this.totalCount[index1] = tempSum;
//          总计
          this.totalCount.forEach(count => {
            num += Number(count);
          });
          this.amount = num * 10;
//          总额
          this.total = num;
        }
        ;
      },
      check(index1, index2, stock) {
//        如果库存数量小于需购买数量，提示，👎按钮没法点击,
        if (Number(this.count[index1][index2]) >= stock) {
          this.disabledBtn = true;
          alert('大于库存数量，请从新输入');
          return;
        }
        this.getamount(index1, index2, stock);
        if (this.total) {
          this.disabledBtn = false;
        } else {
          this.disabledBtn = true;
        }
      },
      makeOrder() {
        alert('下单成功');
        this.orderShow = false;
      }

    }
  }
</script>

<style lang="scss" rel="stylesheet/scss">
  .content {
    margin-top: 10px;
    font-size: 0;
    text-align: center;

  h3 {
    font-size: 18px;
  }

  span {
    color: red;
    font-size: 16px;
    display: inline-block;
    line-height: 26px;
  }

  p {
    color: #ccc;
    font-size: 14px;
  }

  }
  .detail-wrapper {
    margin-top: 30px;

  .title {
    line-height: 26px;
    height: 26px;
    padding: 0 20px;
    width: 50%;
    margin: 0 auto;
    background: #d5d7dc;
    text-align: center;
    font-size: 16px;
  }

  .item {
    margin: 20px;

  label {
    color: #ccc;
    margin-right: 20px;
    font-size: 16px;
  }

  span {
    color: #222;
    font-size: 16px;
  }

  }
  }
  .buy {
    text-align: center;
    margin-top: 30px;

  .buyBtn {
    border: none;
    background: orangered;
    font-size: 16px;
    padding: 10px 20px;
    color: #fff;
  }

  }
  .order {
    position: fixed;
    width: 100%;
    height: 400px;
    border-top: #d5d7dc 1px solid;
    background: #fff;
    z-index: 100;
    bottom: 0;
    right: 0;
    box-shadow: 5px 5px -2px 0 0 #222222;

  table.gridtable {
    margin: 20px auto;
    width: 90%;
    font-family: verdana, arial, sans-serif;
    font-size: 11px;
    color: #333333;
    border-width: 1px;
    border-color: #666666;
    border-collapse: collapse;

  th {
    border-width: 1px;
    padding: 8px;
    border-style: solid;
    border-color: #666666;
    background-color: #dedede;
  }

  td {
    border-width: 1px;
    padding: 8px;
    border-style: solid;
    border-color: #666666;
    background-color: #ffffff;
  }

  }
  .footer {
    display: flex;

  .footer-wrapper {
    flex: 1;
  }

  .makeOrder {
    padding: 10px 0;
    width: 20px;
  }

  }
  }
</style>
