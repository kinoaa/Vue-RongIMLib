<template>
  <div>
    <el-dialog :visible.sync="dialogBan" width="800px" top="50px" title="禁言" :before-close="close" @close="onClose" @open="onOpen">
      <el-row :gutter="15">
        <el-form ref="addFrom" :model="formData" :rules="rules" size="medium" label-width="100px">
          <el-col :span="20">
            <el-form-item label="脸影号" prop="userId">
              <el-input v-model="formData.userId" disabled :style="{width: '100%'}" />
            </el-form-item>
          </el-col>
          <el-col :span="20">
            <el-form-item label="禁言时长" prop="name">
              <el-select v-model="formData.minute" placeholder="请选择禁言时长">
                <el-option
                  v-for="item in minuteOptions"
                  :key="item.value"
                  :label="item.label"
                  :value="item.value"
                />
              </el-select>
            </el-form-item>
          </el-col>
          <el-col :span="20">
            <el-form-item label="理由" prop="name">
              <el-select v-model="formData.reason" placeholder="请选择封禁理由" @change="change">
                <el-option
                  v-for="item in options"
                  :key="item.value"
                  :label="item.label"
                  :value="item.value"
                />
              </el-select>
            </el-form-item>
          </el-col>
          <el-col :span="20">
            <el-form-item label="详情" prop="reason">
              <el-input v-model.trim="formData.description" maxlength="30" show-word-limit placeholder="请输入详情" clearable :style="{width: '100%'}" />
            </el-form-item>
          </el-col>
        </el-form>
      </el-row>
      <div slot="footer">
        <el-button @click="close">取消</el-button>
        <el-button type="primary" :disabled="disabled" @click="handelConfirm">确定</el-button>
      </div>
    </el-dialog>
  </div>
</template>
<script>
import { addGag } from '@/api/user'
const minuteOptions = [{
  'label': '1个小时',
  'value': 60
}, {
  'label': '6个小时',
  'value': 360
}, {
  'label': '12个小时',
  'value': 720
}, {
  'label': '1天',
  'value': 1440
}, {
  'label': '7天',
  'value': 10080
}, {
  'label': '30天',
  'value': 43200
}]
const options = [{
  'label': '言语挑衅, 无事生非',
  'value': '言语挑衅, 无事生非'
}, {
  'label': '违法行为',
  'value': '违法行为'
}, {
  'label': '辱骂攻击',
  'value': '辱骂攻击'
}, {
  'label': '诈骗钱财',
  'value': '诈骗钱财'
}, {
  'label': '垃圾广告',
  'value': '垃圾广告'
}, {
  'label': '血腥暴力',
  'value': '血腥暴力'
}, {
  'label': '政治敏感',
  'value': '政治敏感'
}, {
  'label': '酒托饭托',
  'value': '酒托饭托'
}, {
  'label': '色情骚扰',
  'value': '色情骚扰'
}]
export default {
  components: { },
  inheritAttrs: false,
  props: {
    dialogBan: {
      type: Boolean,
      default: false
    },
    dialogBanData: {
      type: String,
      default: ''
    }
  },
  data() {
    return {
      disabled: false,
      formData: {
        userId: '',
        minute: 60,
        reason: '言语挑衅, 无事生非',
        description: ''
      },
      rules: {

      },
      minuteOptions: minuteOptions,
      options: options
    }
  },
  watch: {},
  methods: {
    onOpen() {
      this.disabled = false
      this.formData.userId = this.dialogBanData
      this.formData.description = ''
    },
    onClose() {
      this.$refs['addFrom'].resetFields()
    },
    close() {
      this.$emit('childMsg', { dialogBan: false })
    },
    handelConfirm() {
      this.$refs['addFrom'].validate(valid => {
        if (!valid) return
        this.disabled = true
        console.log(this.formData)
        addGag(this.formData).then(res => {
          if (res.succeed) {
            this.close()
            this.$message.success('操作成功')
          } else {
            this.disabled = false
          }
        })
      })
    },
    // 快捷选项改变事件
    change(val) {
      console.log(val)
      if (val) {
        this.formData.reason = val
        this.disabledReason = true
      } else {
        this.formData.reason = ''
        this.disabledReason = false
      }
    }
  }
}

</script>
<style lang="scss" scoped>

</style>
