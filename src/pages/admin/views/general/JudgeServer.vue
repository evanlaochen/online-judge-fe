<template>
  <div class="view">
    <icon-btn name="Machine Judge Server" icon="android" :disabled="isJudgeServer" @click.native="exchangeIdentity('JudgeServer')"></icon-btn>
    <icon-btn name="SpiderServer" icon="first-order" :disabled="!isJudgeServer" @click.native="exchangeIdentity('SpiderServer')"></icon-btn>
    <Panel :title="isJudgeServer ? 'Machine Judge Server' : 'Spider Judge Server'">
      <el-table
        :data="servers"
        :default-expand-all="true"
        border>
        <el-table-column
          type="expand">
          <template slot-scope="props">
            <p><span class="ex-tip">{{$t('m.Service_URL')}}:</span><code>{{ props.row.url }}</code></p>
            <p><span class="ex-tip">{{$t('m.Last_Heartbeat')}}:</span>{{ props.row.lastHeartBeat }}</p>
            <p><span class="ex-tip">{{$t('m.Create_Time')}}:</span>{{ props.row.crtTs }}</p>
            <p><span class="ex-tip">Visit Token:</span><code>{{ props.row.visitToken || '-' }}</code></p>
            <p v-show="isJudgeServer"><span class="ex-tip">Judge Version:</span>{{ props.row.judgeVersion }}</p>
          </template>
        </el-table-column>
        <el-table-column
          prop="status"
          label="Status">
          <template slot-scope="scope">
            <el-tag
              :type="scope.row.status === 1 ? 'success' : scope.row.status === -1 ? 'danger' : 'info'">
              {{ scope.row.status === 1 ? 'Normal' : scope.row.status === -1 ? 'Abnormal' : 'disabled' }}
            </el-tag>
          </template>
        </el-table-column>
        <el-table-column
          label="Hostname">
          <template slot-scope="scope">
            {{scope.row.hostname || '-'}}
          </template>
        </el-table-column>
        <el-table-column
          prop="name"
          label="Name">
        </el-table-column>
        <el-table-column
          label="Task Number">
          <template slot-scope="scope">
            {{ scope.row.taskNumber || 0 }}
          </template>
        </el-table-column>
        <el-table-column
          prop="cpuCore"
          label="CPU Core">
          <template slot-scope="scope">{{ scope.row.cpuCore || '-' }}</template>
        </el-table-column>
        <el-table-column
          prop="cpuUsage"
          label="CPU Usage">
          <template slot-scope="scope">{{ scope.row.cpuUsage ? scope.row.cpuUsage + '%' : '-' }}</template>
        </el-table-column>
        <el-table-column
          prop="memoryUsage"
          label="Memory Usage">
          <template slot-scope="scope">{{ scope.row.memoryUsage ? scope.row.memoryUsage + '%' : '-' }}</template>
        </el-table-column>
        <el-table-column
          fixed="right"
          label="Options">
          <template slot-scope="scope">
            <icon-btn name="Edit" icon="edit" @click.native="openJudgeServerDialog(scope.row.id)"></icon-btn>
          </template>
        </el-table-column>
      </el-table>
      <div class="panel-options">
        <el-button type="primary" size="small" @click="openJudgeServerDialog(null)" icon="el-icon-plus" disabled>Create</el-button>
      </div>
    </Panel>
    <el-dialog :title="judgeServerDialogTitle" :visible.sync="showEditJudgeServerDialog"
               @open="onOpenJudgeServerDialog" :close-on-click-modal="false">
      <el-form label-position="top">
        <el-form-item :label="'Name'" required>
          <el-input
            v-model="judgeServer.name"
            disabled
            :placeholder="'name'" class="title-input">
          </el-input>
        </el-form-item>
        <el-form-item :label="'Hostname'" required>
          <el-input
            v-model="judgeServer.hostname"
            disabled
            :placeholder="'hostname'" class="title-input">
          </el-input>
        </el-form-item>
        <el-form-item :label="'URL'" required>
          <el-input
            v-model="judgeServer.url"
            disabled
            :placeholder="'URL'" class="title-input">
          </el-input>
        </el-form-item>
        <Radio-group v-model="judgeServer.status">
          <Radio :label="0" :disabled="judgeServer.status === -1">
            <Icon type="ios-circle-filled"></Icon>
            <span>停用</span>
          </Radio>
          <Radio :label="1" :disabled="judgeServer.status === -1">
            <Icon type="ios-checkmark"></Icon>
            <span>正常</span>
          </Radio>
        </Radio-group>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <cancel @click.native="showEditJudgeServerDialog = false"></cancel>
        <save type="primary" @click.native="submitJudgeServer"></save>
      </span>
    </el-dialog>
  </div>
</template>

<script>
  import api from '../../api.js'

  export default {
    name: 'JudgeServer',
    data () {
      return {
        showEditJudgeServerDialog: false,
        executedImmediate: true,
        servers: [],
        token: '',
        intervalId: -1,
        // 对话框标题
        judgeServerDialogTitle: 'Edit JudgeServer',
        mode: 'create',
        // 评测机 (new | edit) model
        judgeServer: {
          hostname: '',
          id: null,
          name: '',
          status: 0,
          url: ''
        },
        judgeServerList: [],
        // 当前显示的信息列表: 评测机 | 爬虫
        isJudgeServer: true
      }
    },
    mounted () {
      this.init()
    },
    methods: {
      exchangeIdentity (identity) {
        this.executedImmediate = true
        if (identity === 'JudgeServer') {
          this.isJudgeServer = true
        } else {
          this.isJudgeServer = false
        }
        this.init()
      },
      performTask () {
        if (this.isJudgeServer) {
          this.refreshJudgeServerList()
        } else {
          this.refreshSpiderServerList()
        }
      },
      init () {
        // 定时器模拟防止请求爆炸
        if (this.intervalId) {
          clearTimeout(this.intervalId)
        }
        if (this.executedImmediate) {
          this.performTask()
          this.executedImmediate = false
        } else {
          this.intervalId = setTimeout(() => {
            this.performTask()
          }, 5000)
        }
      },
      submitJudgeServer () {
        let params = {
          judgeTypeId: this.judgeServer.id,
          status: this.judgeServer.status
        } 
        api.updateJudgeOrSpiderServer(params).then(res => {
          this.showEditJudgeServerDialog = false
          this.executedImmediate = true
          this.init()
        }).catch()
      },
      // 打开编辑对话框的回调
      onOpenJudgeServerDialog () {
        // todo 优化
        // 暂时解决 文本编辑器显示异常bug
        setTimeout(() => {
          if (document.createEvent) {
            let event = document.createEvent('HTMLEvents')
            event.initEvent('resize', true, true)
            window.dispatchEvent(event)
          } else if (document.createEventObject) {
            window.fireEvent('onresize')
          }
        }, 0)
      },
      openJudgeServerDialog (id) {
        this.showEditJudgeServerDialog = true
        if (id !== null) {
          this.currentJudgeServerId = id
          this.judgeServerDialogTitle = this.isJudgeServer ? 'Edit JudgeServer' : 'Edit SpiderServer'
          this.servers.find(item => {
            if (item.id === this.currentJudgeServerId) {
              this.judgeServer.hostname = item.hostname
              this.judgeServer.status = item.status
              this.judgeServer.name = item.name
              this.judgeServer.url = item.url
              this.judgeServer.id = item.id
              this.mode = 'edit'
            }
          })
        } else {
          this.judgeServerDialogTitle = this.isJudgeServer ? 'Create JudgeServer' : 'Create SpiderServer'
          this.judgeServer.hostname = ''
          this.judgeServer.status = 0
          this.judgeServer.name = ''
          this.judgeServer.url = ''
          this.judgeServer.id = null
          this.mode = 'create'
        }
      },
      refreshSpiderServerList () {
        api.getSpiderServer().then(res => {
          this.servers = res.data.data
          this.init()
        })
      },
      refreshJudgeServerList () {
        api.getJudgeServer().then(res => {
          this.servers = res.data.data
          this.init()
        })
      }
    },
    beforeRouteLeave (to, from, next) {
      // 清除定时器
      if (this.intervalId) {
        clearInterval(this.intervalId)
      }
      next()
    },
    beforeDestroy () {
      // 清除定时器
      if (this.intervalId) {
        clearTimeout(this.intervalId)
      }
    }
  }
</script>
<style scoped>
  .ex-tip {
    color: #464c5b; 
		font-size: 15px; 
		font-weight: 700; 
		font-family: "Helvetica Neue",Helvetica,"PingFang SC","Hiragino Sans GB","Microsoft YaHei","微软雅黑",Arial,sans-serif; 
		margin: 0 9px;
  }
</style>