<template>
  <div class="vote" >
    <a name="top"></a>
    <div class="info-wrap" v-show="showInfo">
      <div class="info-list">
        <mt-field label="地区" placeholder="投票人地区" v-model="name"></mt-field>
        <mt-field label="团贷注册手机号" placeholder="请输入手机号" type="tel" @change="change($event)" v-model="mobile"></mt-field>
        <mt-field label="待回款总额" placeholder="请输入金额" type="number"  v-model="amount"></mt-field>
        <div class="padding-left">
          <span @click="agree1 = !agree1" :class="{'mint-checkbox-core-checked':agree1}" class="mint-checkbox-core"></span>本人承诺提交资料真实无误
        </div>
        <div class="padding-left">
          <span  @click="agree2 = !agree2" :class="{'mint-checkbox-core-checked':agree2}" class="mint-checkbox-core"></span>
          本人已阅读<span class="voterule" @click="showWeiTuo">《出借人委员会授权委托书》</span>,并同意授权出借人委员会代表处理有关团贷网维权事宜。  
        </div>
        <div class="form-btn">
          <mt-button @click="saveInfo()" type="primary" size="normal">确定</mt-button>
        </div>
      </div>
    </div>
    <CommitteeConstitution v-if="isShowConstitution" @closeConstitution="closeConstitution($event)"/>
    <WeiTuo v-if="isShowWeiTuo" @closeWeiTuo="closeWeiTuo($event)"/>
    <h3 class="ui-title ui-center">团贷网出借人委员会代表选举投票</h3>
    <div class="ui-center">投票时间：<span class="ui-gray">待定</span></div>
    <div class="ui-link ui-center">
     <a href="#top" @click="showConstitution()"> 出借人委员会章程</a>
      </div>

    <div class="vote-list-t ui-center">出借人委员会代表候选人</div>
    <ul class="cb graybg">
      <li class="fl ui-gray">请选择您认同的候选人，并投票！</li>
      <li class="fr">投票人数：<span class="allNum">{{allVNum}}</span></li>
    </ul>
    <div class="list-warp">
      <div class="candidateWarp"  v-for="(item,i) in list" :key="i">
        <div class="candidateImg" :style="{'background-image':'url('+item.img+')'}"><span class="vote-num">票数：<span class="big-font">{{item.voteNumber}}</span></span></div>
        <div class="padding-left" @click="choose(item,i)"><span :class="{'mint-checkbox-core-checked':item.checked}" class="mint-checkbox-core"></span>{{item.name}}</div>
        <div class="padding-left money-font">待收金额:<span>{{item.amount}}</span></div>
        <div class="padding-left ui-gray">简介：{{item.description}}</div>
      </div>
    </div>
    <div class="form-btn">
      <mt-button @click="saveVoteInfo()" type="primary" size="normal">提交投票信息</mt-button>
    </div>
  </div>
</template>

<script>
import Vue from 'vue'
import api from '@/utils/api'
import VueResource from 'vue-resource'
import { Button,Toast,Radio,MessageBox  } from 'mint-ui'
import CommitteeConstitution from './CommitteeConstitution.vue';
import WeiTuo from './WeiTuoShu.vue';
Vue.component(Button.name, Button)
Vue.use(VueResource)
export default {
  name: 'VoteIndex',
  data () {
    return {
    	name: '',
    	mobile:'',
    	amount:'',
    	voteInfo:{},
      allNum:0,
      showInfo:true,
      agree1: false,
      agree2: false,
      formInfo:{},
      canSubmit: false,
      list:[],
      isShowConstitution:false,
      isShowWeiTuo:false,
      allVNum: 0
    }
  },
  components: {
    Button: Button,
    Radio: Radio,
    CommitteeConstitution,
    WeiTuo
  },
  created() {
    this.getData()
  },
  methods: {
  	initData(){
  		Object.assign(this, this.$options.data())
    },
    showConstitution(){
      this.isShowConstitution = true;
    },
    closeConstitution(event){
      this.isShowConstitution = event.flag;
    },
    showWeiTuo(){
    	this.isShowWeiTuo = true
    },
    closeWeiTuo(event){
    	this.isShowWeiTuo = event.flag;
    },
  	change(e) {
  		console.log(e)
	    if(!(/^1[34578]\d{9}$/.test(e))){ 
	        Toast("手机号码有误，请重填")
	        this.mobile = ''
	    } else {
	    	this.canSubmit = true
	    }
  	},
    choose(item,i) {
    	if (this.allNum <3) {
    		item.checked == undefined ? this.$set(this.list[i],'checked',true) : this.$set(this.list[i],'checked', !item.checked)
     	} else{
     		if (item.checked) {
     			this.$set(this.list[i],'checked', false)
     		} else {
     			Toast('最多选3人')
     		}
    	}
    	this.voteInfo = this.list.filter((item,i) => { return item.checked })
    	this.allNum = this.voteInfo.length
    },
    saveInfo(){
      this.showInfo = false
    },
    saveVoteInfo(){
    	if(this.allNum < 1){
    		Toast('请至少选择1位候选人')
    		return
    	}
    	if (!this.name || !this.mobile || !this.amount || !this.agree1 || !this.agree2 ) {
    		Toast('请完善个人信息')
    		this.showInfo = true
    	} else{
    		this.submitForm()
    	}
    },
    submitForm(){
    	let voteInfoArr = []
    	this.voteInfo.forEach(item => {
    		voteInfoArr.push(item.id)
    	})
    	let formData = new FormData()
					formData.append('name', this.name)
					formData.append('mobile', this.mobile)
					formData.append('amount', this.amount)    
					formData.append('voteInfo', JSON.stringify(voteInfoArr))
			this.$http.post(api.vsubmit, formData, {'Content-Type':'Multipart/form-data'}).then((res) => {
			   if(res.status == 200){
    			MessageBox.confirm('提交成功！').then(action => {
    				Object.assign(this, this.$options.data())
    				this.getData()
    				this.showInfo = false
    				window.opener=null
						window.open('','_self')
						window.close()
    			})
	  		} else {
	  			Toast(res.msg)
	  		}
			}, function (res) {
				
			})
    },
    getData(){
    	this.$http.get(api.vnumber).then(res => {
    		if(res.body.status == 200){
    			this.allVNum = res.body.data
    		} else {
    			Toast(res.body.msg)
    		}
    	})
    	this.$http.get(api.members).then(res => {
    		if(res.body.status == 200){
    			this.list = res.body.data
    		} else {
    			Toast(res.body.msg)
    		}
    	})
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.padding-left{
  padding-left:10px 
}
.graybg{
  background: #f8f8f8;
}
.voterule{
  color: #26a2ff;
}
.ui-center{
  text-align: center;
}
.f12{
  font-size: 12px;
}
ul,li{
  list-style:none
}
.vote{
  font-size: 12px;
  line-height: 16px;
}
.form-btn{
  text-align: center;
  margin-top:30px; 
  margin-bottom:30px; 
}
.mint-button--normal{
  margin: auto;
}
.cb{
 padding: 10px;
 overflow: hidden;
 clear: both;
}
.fl{
  float: left;
}
.fr{
  float: right;
}
.allNum{
  font-size: 18px;
  color: #26a2ff
}
.vote-list-t{
  padding-bottom: 6px;
  color: #333;
}
.ui-title{
  padding: 10px 0;
}
.ui-gray{
  color: #999;
  font-size: 14rpx;
}
.ui-link{
  color: #26a2ff;
  font-weight: 500;
  margin: 10px;
}
.list-warp{
  display: flex;
  display: -webkit-box;
  display: -webkit-flex;
  flex-direction: row;
  -webkit-flex-direction: row;
  flex-wrap:wrap;
  -webkit-flex-wrap:warp;
}
.candidateWarp{
  width: 33.3%;
  height: 240px;
  box-sizing: border-box;
  text-align: left;
}
.candidateImg{
  background-repeat: no-repeat;
  background-position: center;
  background-size: cover;  
  height: 120px;
  margin:5px 5px ;
  margin-bottom:6px; 
  position: relative;
}
.candidateImg .vote-num{
  display: block;
  position: absolute;
  width: 94%;
  background: rgba(0,0,0,0.6);
  color: #fff;
  font-weight: 500;
  text-align: right;
  bottom: 0;
  right: 0;
  padding: 4px;
}
.big-font{
  font-size: 20px;
}
.mint-checkbox-core-checked {
  background-color: rgb(38, 162, 255);
  border-color: rgb(38, 162, 255);
}
.mint-checkbox-core-checked::after {
  transform: rotate(45deg) scale(1);
  border-color: rgb(255, 255, 255);
}
.money-font{
  line-height: 20px;
}
.mint-checkbox-core {
  margin-right:8px; 
  margin-bottom:8px;
  height: 18px;
  width: 18px;
  vertical-align: middle;
}
.info-wrap{
  position: fixed;
  height: 100%;
  width: 100%;
  z-index: 10;
  background: rgba(0,0,0,0.6);
  padding-top: 80px;
  box-sizing: border-box;
}
.info-list{
  border-radius: 6px;
  background:#fff;
  height: 400px;
  width: 90%;
  margin: 0 auto;
  padding: 20px 10px 10px 10px;
  box-sizing: border-box;
}
.mint-cell-wrapper{
  text-align: left;
}
</style>
