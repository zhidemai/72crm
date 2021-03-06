<template>
  <div class="f-container">
    <div v-loading="sendLoading">
      <mix-add ref="mixadd"
               :crmType="crmType"
               :id="id"
               @mixadd-info="submitInfo"></mix-add>
      <flexbox class="se-section">
        <div class="se-name">记录类型</div>
        <el-dropdown style="margin-right: 20px;"
                     trigger="click"
                     @command="handleTypeDrop">
          <flexbox class="se-select">
            <div class="se-select-name">{{followType ? followType : '请选择'}}</div>
            <i class="el-icon-arrow-down el-icon--right"
               style="color:#ccc;"></i>
          </flexbox>
          <el-dropdown-menu slot="dropdown">
            <el-dropdown-item v-for="(item, index) in followTypes"
                              :key="index"
                              :command="item.type">{{item.name}}</el-dropdown-item>
          </el-dropdown-menu>
        </el-dropdown>
        <div class="se-name">下次联系时间</div>
        <el-date-picker class="se-datepicker"
                        v-model="next_time"
                        type="datetime"
                        placeholder="选择日期"
                        value-format="yyyy-MM-dd HH:mm:ss"
                        :editable="false">
        </el-date-picker>
        <el-checkbox v-model="is_event">添加到日程提醒</el-checkbox>
        <el-button @click.native="sendInfo"
                   class="se-send"
                   type="primary">发布</el-button>
      </flexbox>
    </div>
    <div class="log-cont">
      <flexbox v-if="logTypes.length > 0"
               style="border-bottom: 1px solid #E6E6E6;">
        <flexbox v-for="(item, index) in logTypes"
                 :key="index"
                 style="width: auto;"
                 @click.native="logTabsClick(item,index)">
          <div class="log-tabs-item"
               :style="{ color: logType==item.type ? '#F18C70' : '#777'}">{{item.name}}</div>
          <div class="log-tabs-line"
               v-if="logTypes.length -1 != index"></div>
        </flexbox>
      </flexbox>
      <component :is="componentsName"
                 :id="id"
                 :crmType="crmType"></component>
    </div>
  </div>
</template>

<script>
import MixAdd from '../../components/MixAdd'
import RecordLog from '../../components/followLog/RecordLog' // 跟进记录
import { crmRecordSave } from '@/api/customermanagement/common'
import { formatTimeToTimestamp } from '@/utils'

export default {
  /** 线索管理 的 线索详情 的 跟进记录*/
  name: 'clue-follow',
  components: {
    MixAdd,
    RecordLog
  },
  props: {
    /** 模块ID */
    id: [String, Number],
    /** 没有值就是全部类型 有值就是当个类型 */
    crmType: {
      type: String,
      default: ''
    }
  },
  watch: {},
  data() {
    return {
      sendLoading: false,
      /** 记录类型 */
      followTypes: [
        { type: '打电话', name: '打电话' },
        { type: '发邮件', name: '发邮件' },
        { type: '发短信', name: '发短信' },
        { type: '见面拜访', name: '见面拜访' },
        { type: '活动', name: '活动' }
      ],
      followType: '打电话',
      /** 下次联系时间 */
      next_time: '',
      /** 是否添加日程提醒 */
      is_event: false,
      logType: 'record',
      logTypes: []
    }
  },
  computed: {
    componentsName() {
      if (this.logType == 'record') {
        return 'RecordLog'
      }
      return ''
    }
  },
  mounted() {},
  activated: function() {
  },
  deactivated: function() {
  },
  methods: {
    /** 发布 时候的类型选择 */
    handleTypeDrop(command) {
      this.followType = command
    },
    logTabsClick(item, index) {
      this.logType = item.type
    },
    /** 告诉mixad 返回数据 */
    sendInfo() {
      this.$refs.mixadd.$emit('submit-info')
    },
    /** 快捷添加框内 返回的数据用于上传 */
    submitInfo(data) {
      if (this.is_event && !this.next_time) {
        this.$message.error('请选择下次联系时间')
        return
      }
      var params = {}
      params.types = 'crm_' + this.crmType
      params.types_id = this.id
      params.content = data.content
      params.category = this.followType
      var image_ids = data.images.map(function(element, index, array) {
        return element.file_id
      })
      var files_ids = data.files.map(function(element, index, array) {
        return element.file_id
      })
      params.file_id = image_ids.concat(files_ids)
      if (this.next_time) {
        params.is_event = this.is_event ? 1 : 0
        params.next_time = formatTimeToTimestamp(this.next_time)
      }

      this.sendLoading = true
      crmRecordSave(params)
        .then(res => {
          this.sendLoading = false
          this.$message.success(res.data)
          // 重置页面
          this.$refs.mixadd.resetInfo()
          this.is_event = false
          this.next_time = ''
          // 刷新数据
          this.$bus.emit('follow-log-refresh', { type: 'record-log' })
        })
        .catch(() => {
          this.sendLoading = false
        })
    }
  }
}
</script>

<style lang="scss" scoped>
@import '../../styles/followlog.scss';
</style>
