<template>
  <div class="wrapper">
      <home-header @listen="openConfig" @showStatus="showStatus" @showItr="showItrDia"></home-header>
      <div class="content">
        <el-row type="flex" justify="space-between" :gutter="10">
          <el-col :span="7">
            <list-content @listen="getTickList" class="left_content"></list-content>
          </el-col>
          <el-col :span="17">
            <time-tick :list="list" :initialConfig="initialConfig" @clearlist="clearlist" class="right_top_content"></time-tick>
            <count-back class="right_bot_content"></count-back>
          </el-col>
        </el-row>
      </div>
      <config-info :configTrigger="configTrigger" :initialConfig="initialConfig" @closeConfig="closeConfig" @confirmConfig="confirmConfig"></config-info>
      <user-status v-if="showUserStatus" :dialogVisible="showUserStatus" :userStatus="userStatus"  @closeDialog="closeDia"></user-status>
      <promo-intro  :dialogVisible="showPromoIntro" @closeItrDia="closeItrDia"></promo-intro>
  </div>
</template>

<script>
import homeHeader from '@/components/HomeHeader'
import listContent from '@/components/ListContent'
import configInfo from '@/components/ConfigInfo'
import countBack from '@/components/CountBack'
import timeTick from '@/components/TimeTick'
import userStatus from '@/components/UserStatus'
import promoIntro from '@/components/PromoIntro'

export default {
  data () {
    return {
      configTrigger: false,
      initialConfig: {},
      list: {},
      showUserStatus: false,
      userStatus: {},
      showPromoIntro: false
    }
  },
  created () {
    this.getUserStatus()
    this.getInitialConfig()
  },
  methods: {
    openConfig () {
      this.configTrigger = true
    },
    closeConfig () {
      this.configTrigger = false
    },
    closeDia () {
      this.showUserStatus = false
    },
    showStatus () {
      this.showUserStatus = true
    },
    closeItrDia () {
      this.showPromoIntro = false
    },
    showItrDia () {
      this.showPromoIntro = true
    },
    getUserStatus () {
      this.$http({
        method: 'POST',
        url: '/api/user/status'
      }).then((res) => {
        if (res.data.status === true) {
          let firstIn = res.data.is_first_in
          this.userStatus = res.data.data
          if (firstIn) {
            this.showPromoIntro = true
          } else {
            this.showUserStatus = res.data.is_show
          }
        } else {
          this.$message({
            message: '初始化时间失败，请重新刷新页面',
            type: 'warning'
          })
        }
      })
    },
    getInitialConfig () {
      this.$http({
        method: 'POST',
        url: '/api/config'
      }).then((res) => {
        if (res.data.status === true) {
          this.initialConfig = res.data.data
        }
      })
    },
    // 确认配置，发送配置保存请求
    confirmConfig (configInfo) {
      this.$http({
        method: 'POST',
        url: '/api/addconfig',
        data: configInfo
      }).then((res) => {
        if (res.data.status === true) {
          this.$message({
            message: '新增配置成功',
            type: 'success'
          })
          this.configTrigger = false
          this.getInitialConfig()
        } else {
          this.$message({
            message: '新增配置失败，请重试',
            type: 'warning'
          })
        }
      })
    },
    // 获取子组件向上传递的任务信息
    getTickList (list) {
      this.list = JSON.parse(JSON.stringify(list))
    },
    clearlist () {
      this.list = {}
    }
  },
  components: {
    homeHeader, // 顶部
    listContent, // 左侧列表
    configInfo, // 个人设置
    countBack, // chart图表显示
    timeTick, // 计时
    userStatus, // 用户的状态
    promoIntro // 番茄的用法说明
  }
}
</script>

<style lang="scss">
.content{
  padding: 30px 5%;

  .right_bot_content{
    margin-top: 30px;
  }
}
</style>
