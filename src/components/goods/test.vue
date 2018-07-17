<template>
    <div class="reback">
        <div class="top">
            <div class="topfixed">
                <span class="span1 icon-line-left" @click="goDetail()"></span>
                <span class="span2" >项目回报</span>
                <span class="span3 icon-prompt" @click="isShowModal=true">&nbsp;风险提示</span>
            </div>
        </div>  
        
        <div class="center">    
            <div class="contents">
                <div class="items" v-for="item in goodsLists">
                    <div class="items-money"v-if="item.returnGrade == 0" >￥{{item.price}}</div>
                    <div class="items-num"  v-if="item.returnGrade == 0" >限额&nbsp;<b>{{item.stock}}</b>&nbsp;份&nbsp;|&nbsp;剩余&nbsp;<b>{{item.saleStock}}</b>&nbsp;份</div>
                    <div class="item-yh"    v-if="item.returnGrade == 0" v-html="item.description"></div>
                    <div class="item-icon"  v-if="item.returnGrade == 0" ><img :src="item.goodsList[0].image[0].filePath"></div>
                    <div class="item-yf"    v-if="item.returnGrade == 0" >配送费用：<b>{{item.shipCost?item.shipCost:'免运费'}}</b></div>
                    <div class="item-yf"    v-if="item.returnGrade == 0">预计回报发送时间：<b>{{item.expectReturnDate}}</b></div>
                    <div :class="{'item-btn':item.saleStock>0,'item-disabled':item.saleStock==0}" v-if="item.returnGrade == 0" @click="goShopping(item)"><p>我要参与</p></div>
                    <!-- 共建类型 -->
                    <div class="title-type2" v-if="item.returnGrade == 1">{{item.description}}</div>
                    <div class="code-type2"  v-if="item.returnGrade == 1"><div><img :src="item.goodsList[0].image[0].filePath"></div></div>
                    <div class="phone-type2" v-if="item.returnGrade == 1">客服电话：400-900-1688</div>
                    <div class="phone-type2" v-if="item.returnGrade == 1">工作日 9：00-18：00</div>

                </div>
            </div>
        </div> 
        <div class="bottom">市场有风险投资需谨慎</div>
        <!-- 模态框商品组的展示 -->
        <div class="fix-modal">
            <mt-popup v-model="popupVisible" position="bottom" :closeOnClickModal='true'>
                <div class="shopping" v-if="shopping">
                    <div class="items">
                        <div class="items-money clearfix">
                            <span class="span1">￥{{goodsDetail.price}}</span>
                            <span class="span2 icon-close2" @click="popupVisible=false;shopping=false"></span>
                        </div>
                        <div class="items-num">限额&nbsp;<b>{{goodsDetail.stock}}</b>&nbsp;份&nbsp;|&nbsp;剩余&nbsp;<b>{{goodsDetail.saleStock}}</b>&nbsp;份</div>
                        <div class="item-yh" v-html="goodsDetail.description"></div>
                        <div class="item-icon"><img :src="imgSrc"></div>
                        <div class="item-yf">配送费用：<b>{{goodsDetail.shipCost?goodsDetail.shipCost:'免运费'}}</b></div>
                        <div class="item-yf">预计回报发送时间：<b>{{goodsDetail.expectReturnDate}}</b></div>
                        <div class="item1">{{choseInfo}}</div>

                        <!-- 商品组属性及详情************************************************************** -->
                        <div class="item2" v-for="item in typeArray">
                            <div class="left">{{item.key}}</div>
                            <div class="right">
                                <span  :class="{'active':item2.isActive,'disabled':item2.isDisabled}" v-for="item2 in item.type" @click="typeBtn(item2,item.type)">{{item2.key}}</span>
                                
                            </div>
                        </div>
                        <!-- 购物车加减******************************************************* -->
                        <div class="item3 clearfix">
                            <div class="left">数量<b v-if="!isCanSubmit">（ <b>库存：{{stock}}</b>）</b></div>
                            <div class="right right-add">
                                <span @click="subCount('-')">-</span>
                                <input type="text" @keyup="keyup($event)" v-model="count">
                                <span class="icon-plus" @click="subCount('+')">+</span>
                            </div>
                        </div>
                    </div>
                    <div :class="{'pay-btn':!isCanSubmit,'pay-btn-disabled':isCanSubmit}"><span @click="goCheck()">去结算</span></div>
                </div>
                
            </mt-popup>
        </div>

    </div>
</template>
<script>
import result from'./goods.js'
import { Popup,Toast } from "mint-ui";
import Vue from "vue";
Vue.component(Popup.name, Popup,);
export default {
    data() {
        return {
        goodsLists:result.data,
        popupVisible: false,
        isShowModal:false,
        shopping:false,
        isInvest:false,
        count:1,
        stock:1,
        choseInfo:'',
        goodsDetail:{},
        typeArray:[],
        imgSrc:'',
        checkedObj:{},
        isCanSubmit:false,
        };
    },
    methods: {
    keyup(event){
      this.count=this.count.replace(/[^\d]/g,'') 
    },
    showModal() {
      this.popupVisible = true;
    },
    onValuesChange(picker, values) {
      if (values[0] > values[1]) {
        picker.setSlotValue(1, values[0]);
      }
    },
    goDetail(){
        // window.location.href=`${hostUrl}/detailzc/${this.$route.params.id}`
        this.$router.push({name:'detailzc',params:{id:this.$route.params.id}})
    },
   
    /*
    * 点击去参与项目
    * */ 
    goShopping(item){
        console.log(item);
        if(item.saleStock === 0)return;
        
        this.popupVisible=true;
        this.shopping=true;
        this.goodsDetail = item;
        this.imgSrc = this.goodsDetail.goodsList[0].image[0].filePath;
        this.currentGoodsGroupObj = item;
        this.verifyData(item);
        
        
    },
    // 商品组分类处理
    verifyData(item){
        
        // 第一步反转 规格名
        let typeArr= [];
        let arr = item.goodsList[0].goodsAttribute;
        item.goodsList.forEach((value,index,array)=>{
            value.goodsAttribute.forEach((value2,index2,array2)=>{
                typeArr.push({
                    goods:value,
                    goodsAttributeId:value2.goodsAttributeId,
                    goodsAttributeName:value2.goodsAttributeName,
                    goodsAttributeSpecId:value2.goodsAttributeSpecId,
                    goodsAttributeSpecName:value2.goodsAttributeSpecName
                });
            })
        })

        console.log(typeArr)
        
        let obj = {};
        typeArr.forEach((value,index,array)=>{
            obj[value.goodsAttributeName] =  []; 
        })
        console.log(obj);
        typeArr.forEach((value,index,array)=>{
            obj[value.goodsAttributeName].push(value)
        })
        

        // console.log(obj);
        // // 第二步反转规格值 
        for(var key in obj){
            let temp = obj[key];
            obj[key]={v:temp};
        }
        // console.log(obj);
        for(var key in obj){ //v临时变量
           obj[key]['v'].forEach((value,index,array)=>{
                obj[key][value.goodsAttributeSpecName] = [];
            })
            obj[key]['v'].forEach((value,index,array)=>{
                obj[key][value.goodsAttributeSpecName].push(value)
            })
        }
        
        // console.log(obj);
        let arrNew = [];
        
        for(var key in obj){
            let typeArr = [];
            let typeObj = obj[key];
            console.log(typeObj);
            for(var key2 in typeObj){ //将具体一个规格的商品id简化出来，设置激活状态，将新的数据结构放在数组里面
                if(key2!='v'){
                    let goodsArr = typeObj[key2];
                    console.log(goodsArr);
                    let idArrs = [];
                    goodsArr.forEach((value,index,array)=>{
                        idArrs.push(value.goods.id);
                    })
                    typeArr.push({type:key,key:key2,arr:typeObj[key2],isActive:false,isDisabled:false,ids:idArrs})
                }
                
            }
            
            // console.log('-----');
            // console.log(typeArr);
            arrNew.push({key:key,type:typeArr})
        }
        this.typeArray = arrNew;
        this.initChecked();
        console.log(this.typeArray);
    },
    subCount(type){
        
        if(this.isCanSubmit) return;
        console.log(this.checkedObj);
        // Toast('提示信息');
        if(!this.stock){
            return;
        }
        console.log('库存加减');
        if(type === '-'){
            if(this.count === 1){
                return;
            }else{
                this.count--;  
            }
        }
        if(type === '+'){
            if(this.count === this.stock){
                Toast(`商品数量超出库存${this.stock}`);
                return;
            }else if(this.count >this.checkedObj.goods.singleBuyNum || this.count === this.checkedObj.goods.singleBuyNum){
                Toast(`商品数量超出最大购买数量${this.checkedObj.goods.singleBuyNum}`);
                return;
            }else{
                this.count++;  
            }
        }
    },
    // 去结算
    goCheck(){
        if(this.isCanSubmit) return;
        console.log(this.checkedObj);
        this.$router.push({'name':'check',params:{id:this.$route.params.id},query:{goodsId:this.checkedObj.goods.id,count:this.count}})
    },
    // 初始化选中
    initChecked(){
        this.choseInfo = "";
        let specAllArr = this.getAllSpec();
        specAllArr.forEach((value,index,arr)=>{
            value.isDisabled = !this.haveStock(value.arr); 
        })
        // 选中第一个可以被激活的商品
        for(var i=0;i<this.typeArray.length;i++){
            for(var j=0;j<this.typeArray[i].type.length;j++){
                console.log(this.typeArray[i].type[j]);
                this.typeBtn(this.typeArray[i].type[j],this.typeArray[i].type);
                if(this.typeArray[i].type[j].isActive){
                    break;
                }
            }
        }
        
    },
    // 点击商品规格
    typeBtn(item,arr){
        if(item.isDisabled){
            return;
        }
        arr.forEach((value,index,arr)=>{
            if(item.key!=value.key){
                value.isActive = false;
            }
            
        })
        item.isActive = !item.isActive;
        let activeArr = this.getActiveSpecArr();
        let specAllArr = this.getAllSpec();
        if(activeArr.length<1){
            specAllArr.forEach((value,index,arr)=>{
                value.isDisabled = !this.haveStock(value.arr); 
            })
        }
        // 判断除了当前之外的能否选中
        this.verifyIsCanSelectExceptCurrent(specAllArr);
        this.selectedTips();
        
    },
    //得到所有规格激活状态的数组
    getActiveSpecArr(){ 
        return this.getAllSpec(true);
    },
    getActiveSpecArrExceptCurrentType(specType){ 
        let arrArr = [];
        this.typeArray.forEach((value,index,arr)=>{
            if(specType!=value.key){
                    value.type.forEach((value2,index2,arr2)=>{
                    if(value2.isActive){  
                        arrArr.push(value2);
                    }
                })
            }
        })
        return arrArr;
        
    },
    // 得到所有的规格的数组
    getAllSpec(isActive=false){ //isActive:true 拿到激活状态下的按钮 否则所有的按钮
        let arrArr = [];
        this.typeArray.forEach((value,index,arr)=>{
            value.type.forEach((value2,index2,arr2)=>{
                if(isActive && value2.isActive){  
                    arrArr.push(value2);
                }
                if(!isActive){
                    arrArr.push(value2);
                } 
            })

        })
        return arrArr;
    },
    
    //求交集
    intersect(goodsArr1,goodsArr2 = null){ //goodsArr2为null返回goodsArr1
        if(goodsArr2 == null){ 
            return goodsArr1;
        }
        let returnArr = [];
        goodsArr1.forEach((value,index,arr)=>{
            goodsArr2.forEach((value2,index2,arr2)=>{
                if(value.goods.id === value2.goods.id){
                    returnArr.push(value);
                }
            })
        })
        return returnArr;
        
    },
    // 判断是否存在库存
    haveStock(goodsArr){
        if(goodsArr.length<1){
            return false;
        }
        return goodsArr.some((item)=>{
            return item.goods.saleStock>0 
        })
    },
    //todo 选中提示
    selectedTips(){
        let tipsArr =  this.typeArray.filter((value,index,arr)=>{
            let isEveryNotActive = value.type.every((item)=>{
                return !item.isActive;
            })
            let isSomeNotDisabled = value.type.some((item)=>{
                return !item.isDisabled;
            })
            return isEveryNotActive && isSomeNotDisabled;
        })
        console.log(tipsArr);
        if(tipsArr.length>0){
            this.choseInfo = `请选择：${tipsArr[0].key}`;
            this.isCanSubmit = true;
            this.checkedObj = {};
            this.count = 1;
            this.imgSrc = this.goodsDetail.goodsList[0].image[0].filePath;
        }
        if(tipsArr.length<1){
            let activeArr = this.getActiveSpecArr();
            console.log(activeArr);
            this.choseInfo = `已选择：`;
            this.isCanSubmit =false;
            activeArr.forEach((value,index,arr)=>{
                this.choseInfo +=`'${value.key}' `;
            })
            let checked = activeArr.reduce((total,current)=>{
                return this.intersect(current.arr,total);
            },null)
            this.checkedObj = checked[0];
            console.log(this.checkedObj);
            this.stock = this.checkedObj.goods.saleStock;
            this.imgSrc = this.checkedObj.goods.image[0].filePath;
        }
        
    },
    // 判断除了当前之外的能否选中
    verifyIsCanSelectExceptCurrent(specAllArr){
        specAllArr.forEach((value,index,arr)=>{ 
            let notCurrenArr = this.getActiveSpecArrExceptCurrentType(value.type);
            // console.log(notCurrenArr);
            if(notCurrenArr.length>0){
                // console.log(notCurrenArr);
                let intersetArrNotCurrent = notCurrenArr.reduce((total,current)=>{
                    return this.intersect(current.arr,total);
                },null)
                let checkArr = this.intersect(intersetArrNotCurrent,value.arr);
                // console.log(checkArr);
                value.isDisabled = !this.haveStock(checkArr); 
                // console.log(checkArr);
            }else{
                value.isDisabled = !this.haveStock(value.arr); 
            }
            
        })
    }
    
  },
  mounted(){
     console.log(this.goodsLists);
  }
}
</script>
<style lang="scss" scoped>
.reback{
    background: #F8F8F8;
    .top{
        background: #ffffff;
        width: 100%;
    }
    .topfixed{

        width: 345px;
        height: 44px;
        margin:0 auto;
        text-align: center;
        color:#ffffff;
        background: #ffffff;
        color: #393A3F;
        .icon-line-left:before{
            font-size: 20px;
            line-height: 44px;
            color: #61687C;
        }
        .icon-forwarding:before{
            font-size: 20px;
            line-height: 44px;
            color: #61687C;
        }
        .span1{
            float: left;

        }
        .span2{
            font-size: 18px;
            line-height: 44px;
        }
        .span3{
            float: right;
            font-size: 14px;
            color: #393A3F;
            font-weight: 300;
        }
        .icon-prompt{
            font-size: 16px;
            line-height: 44px;
        }
    }

    .center{
        background: #ffffff;
        width: 100%;
        box-shadow: 0 -3px 10px 0 #F8F8F8;
    }
    .contents{
        width: 345px;
        margin:0 auto;
        margin-bottom: 10px;
        .items{
            padding-bottom: 30px;
            .items-money{

                font-size: 18px;
                color: #FB6B38;
                line-height: 22px;
                padding-top: 15px;
            }
            .items-num{
                font-size: 14px;
                color: #61687C;
                line-height: 20px;
                margin:15px 0 10px 0;
            }
            .item-yh{
                font-size: 14px;
                color: #61687C;
                line-height: 20px;
            }
            .item-icon{
                margin:15px 0;
                img{
                    width: 60px;
                    height: 60px;
                }
            }
            .item-yf{
                font-size: 12px;
                color: #61687C;
                margin-bottom: 6px;
                b{
                    font-size: 12px;
                    color: #002C00;
                    line-height: 17px;
                }
            }
            .item-btn{
                text-align: center;
                margin-top: 20px;
                p{
                    display: inline-block;
                    width: 180px;
                    height: 40px;
                    line-height: 40px;
                    text-align: center;
                    background-image: linear-gradient(-269deg, #25BEF2 0%, #218CF7 100%);
                    border-radius: 22px;
                    color: #ffffff;

                }
            }
            .item-disabled{
                text-align: center;
                margin-top: 20px;
                p{
                    display: inline-block;
                    width: 180px;
                    height: 40px;
                    line-height: 40px;
                    text-align: center;
                    background: #E3E7ED;
                    border-radius: 22px;
                    color: #9FA3B0;

                }
            }
            .item01{
                font-size: 14px;
                color: #61687C;
                line-height: 20px;
                padding-top: 20px;
            }
            .title-type2{
                padding:20px 0 0 0;
                font-size: 14px;
                color: #61687C;
                line-height: 20px;
            }
            .code-type2{
                text-align: center;
                width: 100%;
                div{
                    display: inline-block;
                    margin: 20px 0;
                    padding:10px;
                    border: 1px solid #E3E7ED;
                    img{
                        width: 200px;
                        height: 200px;
                    }
                }
            }
            .phone-type2{
                text-align: center;
                line-height: 18px;
                margin-bottom: 6px;
            }
        }
    }
    .margin0{
        margin-bottom: 0;
    }
    .bottom{
        background:  #F8F8F8;
        font-size: 11px;
        color: #9FA3B0;
        height: 48px;
        line-height: 48px;
        text-align: center;
    }
    .fix-modal{
        width: 100%;
        height: 200px;
        background: #ffffff;
        position:fixed;
        .mint-popup{
            width: 100%;
            background: #FFFFFF;
            border-radius: 10px 10px 0 0;
        }
        .shopping{
            width: 100%;
            margin:0 auto;
            .items{
                width: 345px;
                margin:0 auto;
                .items-money{

                    font-size: 18px;
                    color: #FB6B38;
                    line-height: 22px;
                    padding-top: 15px;
                    .span1{
                        float: left;
                    }
                    .span2{
                        float: right;
                    }
                    .icon-close2:before{
                        font-size: 20px;
                    }
                }
                .items-num{
                    font-size: 12px;
                    color: #61687C;
                    line-height: 20px;
                    margin:15px 0 10px 0;
                }
                .item-yh{
                    font-size: 12px;
                    color: #61687C;
                    line-height: 17px;
                    font-weight: 200;
                }
                .item-icon{
                    margin:15px 0;
                    img{
                        width: 60px;
                        height: 60px;
                    }
                }
                .item-yf{
                    font-size: 12px;
                    color: #61687C;
                    margin-bottom: 6px;
                    b{
                        font-size: 12px;
                        color: #002C00;
                        line-height: 17px;
                    }
                }
                .item-btn{
                    text-align: center;
                    margin-top: 20px;
                    p{
                        display: inline-block;
                        width: 180px;
                        height: 40px;
                        line-height: 40px;
                        text-align: center;
                        background-image: linear-gradient(-269deg, #25BEF2 0%, #218CF7 100%);
                        border-radius: 22px;
                        color: #ffffff;

                    }
                }
                .item1{
                    font-size: 12px;
                    color: #FB6B38;
                    line-height: 20px;
                    padding-top: 20px;
                    margin-bottom: 20px;
                }
                .item2{
                    .left{
                        float: left;
                        font-size: 12px;
                        color: #61687C;
                        font-weight: 200;
                        line-height: 30px;
                        width: 58px;
                        // margin-right: 10px;
                    }
                    .right{
                        float: left;
                        width: 287px;
                        span.active{
                            display: inline-block;
                            background-image: linear-gradient(-269deg, #25BEF2 0%, #218CF7 100%);
                            border-radius: 15px;
                            height: 30px;
                            text-align: center;
                            line-height: 30px;
                            color: #ffffff;
                            margin-left:10px;
                            padding:0 20px;
                        }
                        span{
                            display: inline-block;
                            background: #F8F8F8;
                            border-radius: 15px;
                            padding:0 20px;
                            height: 30px;
                            text-align: center;
                            line-height: 30px;
                            font-size: 12px;
                            color: #393A3F;
                            margin-left:10px;
                            margin-bottom: 20px;
                        }
                        span.disabled{
                            display: inline-block;
                            background: #ededed;
                            border-radius: 15px;
                            padding:0 20px;
                            height: 30px;
                            text-align: center;
                            line-height: 30px;
                            font-size: 12px;
                            color:#ccc;
                            margin-left:10px;
                            margin-bottom: 20px;
                        }
                    }

                }
                .item3{
                    .left{
                        float: left;
                        font-size: 12px;
                        color: #61687C;
                        font-weight: 200;
                        line-height: 35px;
                        margin-right:20px;
                    }
                    .right{
                        float: left;
                        padding-bottom: 47px;
                        span{
                            float: left;
                            width: 35px;
                            height: 35px;
                            text-align: center;
                            line-height: 35px;
                            background: #E3E7ED;
                            color:#9FA3B0;
                            font-size: 20px;

                        }
                        input{
                            float: left;
                            width: 82px;
                            background: #F8F8F8;
                            height: 35px;
                            text-align: center;
                            line-height: 35px;
                            border: 0;
                            padding:0;
                            font-size: 12px;
                            color: #393A3F;
                        }
                    }
                }

            }
            .pay-btn{
                padding:7px 0;
                text-align: center;
                // box-shadow: 0 -3px 10px 0 #F8F8F8;
                span{
                    display: inline-block;
                    background-image: linear-gradient(-269deg, #25BEF2 0%, #218CF7 100%);
                    border-radius: 22px;
                    width: 345px;
                    height: 35px;
                    color: #ffffff;
                    text-align: center;
                    line-height: 35px;
                }
                width: 375px;
            }
            .pay-btn-disabled{
                padding:7px 0;
                text-align: center;
                box-shadow: 0 -3px 10px 0 #F8F8F8;
                span{
                    display: inline-block;
                    background: #E3E7ED;
                    border-radius: 22px;
                    width: 345px;
                    height: 35px;
                    color: #9FA3B0;
                    text-align: center;
                    line-height: 35px;
                }
                width: 375px;
                
            }
        }
        .invest{
            width: 345px;
            margin:0 auto;
            padding-bottom: 40px;
            .close{
                text-align: right;
                padding: 15px 0 40px 0;
            }
            .icon-close2:before{
                font-size:20px;
            }
            .title{
                font-size: 12px;
                color: #61687C;
                line-height: 17px;
            }
            .code{
                text-align: center;
                padding-top: 20px;
                padding-bottom:15px;
                img{
                    width: 254px;
                    height: 254px;
                }
            }
            .phone{
                text-align: center;
            }
        }

    }
    .div-modal{
        padding-bottom: 20px;

        p.title{
            font-size: 16px;
            color: #393A3F;
            line-height: 22p;
            text-align: center;
            padding:20px 15px;
        }
        p.text-title{
            font-size: 14px;
            color: #393A3F;
            line-height: 20px;
            padding:0 15px;
            font-weight: 200;
        }
        p.tips{
            text-align: center;
            span{
                background-image: linear-gradient(-269deg, #25BEF2 0%, #218CF7 100%);
                border-radius: 22px;
                width: 144px;
                height: 40px;
                text-align: center;
                line-height: 40px;
                font-size: 16px;
                color:#ffffff;
                margin-top: 40px;
                display: inline-block;
            }

        }
    }
    .fix-two{
        width: 100%;
        height: 200px;
        background: #ffffff;
        position:fixed;
        padding-bottom: 40px;
        .mint-popup{
            width: 100%;
            background: #FFFFFF;
            border-radius: 10px 10px 0 0;
        }
        .close{
            text-align: right;
        }
    }
}

</style>


