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
                    <div class="item-icon"  v-if="item.returnGrade == 0" ><img :src="staticUrl+item.goodsList[0].image[0].filePath"></div>
                    <div class="item-yf"    v-if="item.returnGrade == 0" >配送费用：<b>{{item.shipCost?item.shipCost:'免运费'}}</b></div>
                    <div class="item-yf"    v-if="item.returnGrade == 0">预计回报发送时间：<b>{{item.expectReturnDate}}</b></div>
                    <div :class="{'item-btn':item.saleStock>0,'item-disabled':item.saleStock==0}" v-if="item.returnGrade == 0" @click="goShopping(item)"><p>我要参与</p></div>
                    <!-- 共建类型 -->
                    <div class="title-type2" v-if="item.returnGrade == 1">{{item.description}}</div>
                    <div class="code-type2"  v-if="item.returnGrade == 1"><div><img :src="staticUrl+item.goodsList[0].image[0].filePath"></div></div>
                    <div class="phone-type2" v-if="item.returnGrade == 1">客服电话：400-900-1688</div>
                    <div class="phone-type2" v-if="item.returnGrade == 1">工作日 9：00-18：00</div>

                </div>
            </div>
        </div> 
        <div class="bottom">市场有风险投资需谨慎</div>
        <!-- 模态框商品组的展示 -->
        <div class="fix-modal">
            <mt-popup v-model="popupVisible" position="bottom" :closeOnClickModal='false'>
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
                <!-- 预约投资 -->
                <div class="invest" v-show="isInvest">
                    <div class="close icon-close2" @click="popupVisible=false;isInvest=false"></div>
                    <div class="title">感谢您的参与！您将预约成为我们的构建人，成为构建人后您将获得某某权益</div>
                    <div class="code"><img src="./img/bg_product.jpg"></div>
                    <div class="phone">客服电话：400-900-1688</div>
                    <div class="phone">工作日 9：00-18：00</div>
                </div>
            </mt-popup>
        </div>
        
        <v-modal title="模态框" :isShow="isShowModal" :width="270" @close="isShowModal=false">
            <div class="div-modal">
                <p class="title">风险提示</p>
                <p class="text-title">&nbsp;&nbsp;&nbsp;&nbsp;1、众筹不是商品交易。支持者根据自己的判断选择、支持众筹项目，与发起人共同实现梦想并获得发起人承诺的回报，众筹存在一定风险。</p>
                <p class="text-title">&nbsp;&nbsp;&nbsp;&nbsp;2、铼米平台只提供平台网络空间、技术服务和支持等中介服务。铼米作为居间方，并不是发起人或支持者中的任何一方，众筹仅存在于发起人和支持者之间，使用铼米平台产生的法律后果由发起人与支持者自行承担。</p>
                <p class="text-title">&nbsp;&nbsp;&nbsp;&nbsp;3、众筹项目的回报发放、发票开具及其他后续服务事项均由发起人负责。如果发生发起人无法发放回报、延迟发放回报、不提供回报后续服务等情形，您需要直接和发起人协商解决，铼米对此不承担任何责任。</p>
                <p class="text-title">&nbsp;&nbsp;&nbsp;&nbsp;4、由于发起人能力和经验不足、市场风险、法律风险等各种因素，众筹可能失败。众筹期限届满前失败或募集失败的，您支持的款项会全部原路退还给您；其他情况下，您需要直接和发起人协商解决，铼米对此不承担任何责任。</p>
                <p class="tips"><span @click="isShowModal=false">我知道了</span></p>

            </div>
        </v-modal>

    </div>
    
</template>
<script>
import result from'./goods.js'
import "./reback.scss";
import { Popup } from "mint-ui";
import Vue from "vue";
import modal from '@/components/base/modal/modal'
Vue.component(Popup.name, Popup,modal);
import { Toast } from 'mint-ui';
export default {
    components: {
    "v-modal": modal,
    
  },
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
      goodsLists:[],
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
        this.imgSrc = this.staticUrl+this.goodsDetail.goodsList[0].image[0].filePath;
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

        // console.log(typeArr)
        
        let obj = {};
        typeArr.forEach((value,index,array)=>{
            obj[value.goodsAttributeName] =  []; 
        })
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
            for(var key2 in typeObj){ //将具体一个规格的商品id简化出来，设置激活状态，将新的数据结构放在数组里面
                if(key2!='v'){
                    let goodsArr = typeObj[key2];
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
        // console.log(this.typeArray);
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
            this.imgSrc = this.staticUrl+this.goodsDetail.goodsList[0].image[0].filePath;
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
            this.imgSrc = this.staticUrl+this.checkedObj.goods.image[0].filePath;
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
     
  }
};
</script>

<style lang="scss" scoped>
</style>

