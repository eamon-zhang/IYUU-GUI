<template>
    <el-dialog
            :before-close="handleDialogBeforeClose"
            :visible.sync="visible"
            :title="`编辑下载服务器 - ${form.type}`"
            :close-on-click-modal="false"
            top="8vh"
            width="60%"
            @close="handleDialogClose">
        <div>
            <el-form ref="form" :model="form" label-position="top">
                <el-form-item label="客户端ID" prop="uuid">
                    <el-input v-model="form.uuid" :disabled="true" />
                </el-form-item>
                <el-form-item label="服务器名称" prop="name">
                    <el-input v-model="form.name" />
                </el-form-item>
                <el-form-item label="服务器地址" prop="address">
                    <el-input v-model="form.address" />
                </el-form-item>
                <el-form-item v-if="form.hasOwnProperty('username')" label="登录用户名" prop="username">
                    <el-input v-model="form.username" />
                </el-form-item>
                <el-form-item label="登录密码" prop="password">
                    <el-input v-model="form.password" show-password />
                </el-form-item>
                <el-form-item label="连接超时时间" prop="timeout">
                    <el-slider v-model="form.timeout"
                               :format-tooltip="formatTimeoutTooltip"
                               :marks="marks" :max="800e3" :min="5e3"
                               :step="1000" />
                </el-form-item>
            </el-form>
        </div>
        <span slot="footer" class="dialog-footer">
            <el-button @click="handleDialogClose">取 消</el-button>
            <el-button type="primary" :disabled="disable_save_btn" @click="handleClientEditSave">确 定</el-button>
        </span>
    </el-dialog>
</template>

<script>
  import factory from "../../../plugins/btclient/factory";
  import _ from 'lodash'

  export default {
    name: "ClientEdit",
    props: {
      isVisible: {
        type: Boolean,
        default: false
      },

      info: {
        type: Object,
        required: true
      }
    },

    data() {
      return {
        visible: false,
        clients: {},
        disable_save_btn:false,
        form: {},
        marks: {
          5e3: '5 秒',
          60e3: {
            style: {
              color: '#1989FA'
            },
            label: this.$createElement('strong', '1 分钟')
          },
          300e3: '5 分钟',
          600e3: '10 分钟',
        }
      }
    },

    watch: {
      isVisible: function (newValue) {
        this.visible = newValue
        if (this.visible) {
          this.form = _.clone(this.info)
        } else {
          this.cleanFrom()
        }
      },
    },

    methods: {
      cleanFrom() {
        this.form = {}
      },

      handleDialogClose() {
        this.$emit('close-client-edit-dialog')
      },

      async handleClientEditSave() {
        this.disable_save_btn = true
        this.$notify.info('正在进行下载服务器连接测试，请耐心等待')
        const client = factory(this.form)
        try {
          const pong = await client.ping()
          if (pong) {
            this.$store.commit('IYUU/editEnableClient', this.form)
            this.handleDialogClose()
          }
        } catch (e) {
          this.$notify.error('不能正常连接到下载服务器，请检查你的配置或增加超时时间')
        } finally {
          this.disable_save_btn = false
        }
      },

      handleDialogBeforeClose(done) {
        this.cleanFrom()
        done()
      },

      // FIXME 使用时间库来转换，方法复用
      formatTimeoutTooltip(timeout) {
        const totalSecond = timeout / 1000
        const minute = parseInt(totalSecond / 60)
        const second = minute > 0 ? totalSecond % (60 * minute) : totalSecond

        let convert = []
        if (minute > 0) {
          convert.push(`${minute} 分钟`)
        }

        if (second > 0) {
          convert.push(`${second} 秒`)
        }

        return convert.join(' ')
      }
    }
  }
</script>

<style scoped>

</style>