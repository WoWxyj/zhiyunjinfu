<template>
    <div class="" v-loading="listLoading">
        <div class="indexloanTopTitle">质子贷
            <span v-if="isread == 1" @click="messageGo"><img src="../../assets/changeImg/top-message-icon-no.png" alt=""></span>
            <span v-else @click="messageGo"><img src="../../assets/changeImg/top-message-icon.png" alt=""></span>
        </div>

        <!--大额 小额 精准推荐 黑名单检测-->
        <div class="sliderPlateDiv">
            <div class="sliderDiv" v-for="(item, index) in sliders" :key="index" @click="goLoan(item)">
                <img :src="item.param_url">
                <span>{{ item.module_name }}</span>
            </div>
        </div>

        <!-- 搜索start -->
        <div class="indSearch">
            <span class="sliderClock"><img src="../../assets/img/clock-icon-slider.png" alt=""></span>
            <ul class="marquee_list" :class="{marquee_top:animate}">
                <li v-for="item in items">{{ item }}</li>
            </ul>
        </div>


        <!-- 免注册申请  -->
        <div class="plateOne">
            <ul>
                <li v-for="lable in headLabel" @click="lableUrl(lable)">
                    <div class="leftlable">
                        <h3>{{ lable.module_name }}</h3>
                        <p>{{ lable.param_remark }}</p>
                    </div>
                    <div class="rightlable">
                        <img :src="lable.param_url">
                    </div>
                </li>
            </ul>
        </div>

        <div class="advantageImg">
            <img @click="parambannerUrl" :src="sub_versiontwo_banner.param_url" alt="">
        </div>


        <div v-show="nolineshow" class="allproductcont" >
            <div class="specialTitle">
                <h3>
                    <i class="specialTitleIcon"><img src="../../assets/changeImg/special-title-icon.png" alt=""></i>特别推荐
                    <span class="lookMore" @click="lookMore">更多贷款
                        <i class="specialTitleIconGt"><img src="../../assets/changeImg/special-title-icon-gt.png" alt=""></i>
                    </span>
                </h3>
            </div>

            <ul class="indexContDiv" slot="scrollList">
                <li v-for="loanprot in loanprots" @click="goProduct(loanprot.id)">
                    <!--<div class="loanposiLabel">-->
                    <!--<div v-if=" typeof loanprot.is_review != 'undefined'">-->
                    <!--<div style="position: relative;" v-if="loanprot.is_review.havedate == 1">-->
                    <!--<span class="loanIconSpan">{{loanprot.is_review.datetime}}</span><img-->
                    <!--src="@/assets/img/read-already-big.png" alt="">-->
                    <!--</div>-->
                    <!--<div v-else-if="loanprot.is_review.havedate == 2">-->
                    <!--<span class="loanIconSpan">{{loanprot.is_review.datetime}}</span><img-->
                    <!--src="@/assets/img/apply-already-big.png" alt="">-->
                    <!--</div>-->
                    <!--</div>-->
                    <!--</div>-->
                    <div class="loanContDivTop">
                        <div class="loanContDivTopImg">
                            <img :src="loanprot.loan_logo" alt="">
                        </div>
                        <div class="loanContDivTopTitlt">
                            <h3>{{loanprot.loan_name}}</h3>
                            <p>
                                可贷金额：<span>{{loanprot.loan_small}} - {{loanprot.loan_big}}</span>
                            </p>
                            <p v-if="loanprot.loantime_type == 1">
                                参考月利率：<span>{{loanprot.loan_interest}}%</span>
                            </p>
                            <p v-else>参考日利率：<span>{{loanprot.loan_interest}}%</span></p>
                        </div>
                        <div class="loanContDivTopapply">
                            <span>{{loanprot.applyfornum}}成功申请</span>
                            <div class="loanContApply" @click.stop="gothreeProduct(loanprot)">立即申请</div>
                        </div>
                    </div>
                    <div class="loanContDivBottom">{{loanprot.product_brief}}</div>
                </li>
            </ul>
            <!--<indexproduct></indexproduct>-->
        </div>

        <!--活动弹窗-->
        <div class="shareDiv shareDivwech" v-show="shareDivWechat">
        <div class="popupImg">
        <img class="popimg" @click="popupUrlLink" :src="popupImg">
        <img class="close" @click="closeshare" src="../../assets/img/popup-close.png" alt="">
        </div>
        </div>
    </div>
</template>

<script>
  import {
    getrolling,
    usercenter,
    channelStop,
    paramData,
    messageListAllIsread,
    clickMenu,
    popupMessage,
    messageUpdateStatus,
    applylook
  } from '../../api/api'
  import indexproduct from '../../components/swiperSlide.vue'
  import lookno from '../none/lookno.vue'
  import qs from 'qs'

  export default {
    data () {
      return {
        listLoading: false,
        // 搜索框循环滚动
        items:  [],
        // 免注册申请 免下载申请
        headLabel: [],
        // 大额专区  小额专区
        sliders: [],
        loanprots: [],
        sub_versiontwo_banner:'',  //广告位图片
        isread: '',
        onlineShow: false,
        nolineshow: true,
        popupImg: require('../../assets/img/none-opcity.png'),
        shareDivWechat: false,
        popupType: '',
        popupid: '',
        messageId: '',
        linkAddress: '',
        animate: false
      }
    },
    created () {
      this.$store.state.showBottomNav = true
      setInterval(this.showMarquee, 2000)
    },
    components: {
      indexproduct,
      lookno
    },
    mounted () {
      paramData['protocol']['token'] = sessionStorage.getItem('token')
      this.getchannelStop()
    },

    methods: {
      //公告滚动
      showMarquee: function () {
        this.animate = true;
        setTimeout(()=>{
          this.items.push(this.items[0]);
          this.items.shift();
          this.animate = false;
        },500)
      },

      //广告位跳转
      parambannerUrl(){
        if (this.sub_versiontwo_banner.param_module == '0001') {
          this.$router.push({
            path: '/newcutloan',
            query: {
              id: this.sub_versiontwo_banner.id,
              moduleName: this.sub_versiontwo_banner.module_name,
              parambannerUrl: this.sub_versiontwo_banner.parambanner_url
            }
          })
        }
      },

      //渠道停用
      getchannelStop () {
        channelStop(paramData).then(res => {
          if(res.code == 514){
            this.$router.push({path: '/none'})
          }else{
            this.getData(paramData)
            this.getpopupMessage()
            if (sessionStorage.getItem('motion')) {
              this.gettokenIndexMotion()
              let locationurl = location.href
              let parseUrl = qs.parse(locationurl.split('?')[1])
              if (typeof parseUrl.phone != 'undefined') {
                sessionStorage.setItem('userphone', parseUrl.phone)
              }
              if (window.location.href.indexOf('motion') > 0) {
                this.$router.push({path: '/loan'})
              }
            }
          }
        })
      },
      // 弹窗活动
      getpopupMessage () {
        paramData['protocol']['token'] = sessionStorage.getItem('token')
        let _this = this
        popupMessage(paramData).then(res => {
          if (res.code === 200) {
            _this.popupImg = res.data.link_img
            _this.popupType = res.data.type
            _this.popupid = res.data.id
            _this.messageId = res.data.message_id
            _this.linkAddress = res.data.link_address
            if (_this.popupType == 0) {
              this.shareDivWechat = false
              return
            } else {
              if (sessionStorage.getItem('token')) {  //是否登录  登陆情况下一天弹出一次   未登录每次到首页都弹出
                let nowTime = new Date().toDateString()   //获取当前时间
                let oldTime = sessionStorage.getItem('oldTime')  //获取前一次本地session存储的时间
                if (oldTime == nowTime) { //弹窗一天弹出一次  判断两个时间戳是否为同一天  是：弹窗不显示 否：显示
                  _this.shareDivWechat = false
                  sessionStorage.setItem('oldTime', nowTime) //然后将当前的时间存储到session
                } else {
                  _this.shareDivWechat = true
                  sessionStorage.setItem('oldTime', nowTime)
                }
              } else {
                _this.shareDivWechat = true
              }
            }
          }
        })
      },
      //1=额度评测  2=黑名单检测  3=精准推荐  4=活动/产品 0=无
      popupUrlLink () {
        if (this.popupType === 1) {
          this.$router.push({path: '/assessmentData'})
        } else if (this.popupType === 2) {
          this.$router.push({path: '/blackList'})
        } else if (this.popupType === 3) {
          this.$router.push({path: '/accurateRecom'})
        } else if (this.popupType === 4) {
          paramData['protocol']['token'] = sessionStorage.getItem('token')
          let operation_type
          let ua = navigator.userAgent.toLowerCase()
          if (ua.match(/MicroMessenger/i) == 'micromessenger') {
            operation_type = '0404'
          } else {
            operation_type = '0104'
          }
          let param = {
            operation_type: operation_type,
            loan_infor_id: this.popupid,
            product_type: '0001',
            product_operation_type: '0001',
            message_id: this.messageId,
            message_type: '0000'
          }
          paramData['data'] = param
          applylook(paramData).then(res => {
            if (res.code == 200) {
              location.href = this.linkAddress
            } else {
              alert(res.message)
            }
          })
          messageUpdateStatus(paramData).then(res => {
            if (res.code == 200) {

            }
          })
        }
      },

      // //弹窗关闭按钮
      closeshare () {
        this.shareDivWechat = false
      },

      gettokenIndexMotion () {
        let url = location.href
        let parseUrl = qs.parse(url.split('?')[1])
        if (typeof parseUrl.usernum != 'undefined') {
          sessionStorage.setItem('token', decodeURIComponent(parseUrl.usernum))
        }
      },

      reload () {
        location.reload()
      },
      lookMore () {
        this.$router.push({path: '/loan'})
      },
      //选择城市
      chooseCity () {
        this.$router.push({path: '/city'})
      },
      // 点击搜索跳转
      searchGo: function () {
        this.$router.push({path: '/search'})
      },
      // 大额专区
      goLoan: function (item) {
        let ua = navigator.userAgent.toLowerCase()
        let operation_type
        if (ua.match(/MicroMessenger/i) == 'micromessenger') {
          operation_type = '0407'
        } else {
          operation_type = '0107'
        }
        let param = {
          token: sessionStorage.getItem('token'),
          menu_id: item.id,
          operation_type: operation_type
          // protocol:
        }
        paramData['data'] = param
        clickMenu(paramData).then(res => {
          if (res.code == 200) {
            if (item.param_module == '0001') {
              this.$router.push({
                path: '/newcutloan',
                query: {
                  id: item.id,
                  moduleName: item.module_name,
                  parambannerUrl: item.parambanner_url
                }
              })
            } else if (item.param_module == '0002') {
              this.$router.push({path: '/accurateRecom'})
            } else if (item.param_module == '0003') {
              this.$router.push({path: '/card'})
            } else if (item.param_module == '0004') {
              if (sessionStorage.getItem('token')) {
                paramData['protocol']['token'] = sessionStorage.getItem(
                  'token'
                )
                let parmas = {
                  login_type: sessionStorage.getItem('login_type')
                }
                paramData['data'] = parmas
                usercenter(paramData).then(res => {
                  if (res.code == 200) {
                    if (res.data.free == 2) {
                      this.$router.push({path: '/blackList'})
                    } else {
                      this.$router.push({
                        path: '/blackListReport',
                        query: {
                          blackUsername: res.data.customer_name,  //姓名
                          blackIdcard: res.data.u_no,  //身份证
                          blackMobile: res.data.mobile,  //手机号码
                          blackType: 3
                        }
                      })
                    }
                  } else {
                    alert(res.message)
                  }
                })
              } else {
                this.$router.push({path: '/usercenter'})
              }
            }
          }
        })
      }
      ,
      //第一模块 免注册申请  免下载申请
      lableUrl (lable) {
        let ua = navigator.userAgent.toLowerCase()
        let operation_type
        if (ua.match(/MicroMessenger/i) == 'micromessenger') {
          operation_type = '0407'
        } else {
          operation_type = '0107'
        }
        let param = {
          token: sessionStorage.getItem('token'),
          menu_id: lable.id,
          operation_type: operation_type
          // protocol:
        }
        paramData['data'] = param
        clickMenu(paramData).then(res => {
          if (res.code == 200) {
            if (lable.param_module == '0001') {
              this.$router.push({
                path: '/newcutloan',
                query: {
                  id: lable.id,
                  moduleName: lable.module_name,
                  parambannerUrl: lable.parambanner_url
                }
              })
            }
          }
        })
      }
      ,
      // 顶部活动icon
      messageGo () {
        // 一键已读
        let that = this
        that.listLoading = true
        let param = {
          message_type: '0000'
        }
        paramData['data'] = param
        messageListAllIsread(paramData).then(res => {
          this.isread = 2
          this.$router.push({path: '/message'})
        })
        let paramlable = {
          message_type: '0001'
        }
        paramData['data'] = paramlable
        messageListAllIsread(paramData).then(res => {
          this.isread == 2
          this.$router.push({path: '/message'})
        })
        localStorage.setItem('readalread', this.isread)
      },

      getData (dateindex) {
        let that = this
        that.listLoading = true
        getrolling(dateindex)
          .then(res => {
            that.listLoading = false
            if (res.code == 200) {
              that.sub_versiontwo_banner = res.data.sub_versiontwo_banner   //广告位图片
              that.items = res.data.rolling   //滚动公告
              that.headLabel = res.data.sub_versiontwo_sublabel    //第一板块标签
              that.sliders = res.data.sub_versiontwo_mainlabel     //第二板块标签
              that.loanprots = res.data.chunk[0].product   //产品
              that.isread = res.data.isread
            }
            else {
              that.listLoading = false
              that.onlineShow = true
            }
          })
      },

    }
  }
</script>
<style lang="scss">
    @import "../../assets/scss/common.scss";
    @import "../../assets/scss/channeloneclass.scss";
    @import "../../assets/scss/noline.scss";
</style>
