<template>
  <div id="app">
    <div class="title">
      <span>wechat侧边栏打包</span>
    </div>
    <div class="content-global">

      <el-select v-model="branch" placeholder="请选择" filterable style="margin-left: 15px;">
        <el-option
          v-for="item in branchData"
          :key="item.value"
          :label="item.label"
          :value="item.value">
        </el-option>
      </el-select>
      <el-select v-model="dir" placeholder="请选择" filterable style="margin-left: 15px;">
        <el-option
          v-for="item in dirData"
          :key="item.value"
          :label="item.label"
          :value="item.value">
        </el-option>
      </el-select>
      <el-button  @click="build" :loading="loading" type="success" style="margin-left: 15px;">打包</el-button>
    </div>
    <!-- <div class="pm2-table">
      <el-table stripe :data="tableData" border style="width: 100%">
        <el-table-column prop="pid" label="pid" width="180">
        </el-table-column>
        <el-table-column prop="name" label="进程名称" width="180">
        </el-table-column>
        <el-table-column prop="cpu" label="cpu"> </el-table-column>
        <el-table-column prop="memory" label="memory"> </el-table-column>
        <el-table-column prop="status" label="status">
          <template slot-scope="{row}">
            <span :class="getStatusColor(row.status)">{{row.status}}</span>
          </template>
        </el-table-column>
        <el-table-column prop="restartTime" label="重启次数"> </el-table-column>
        <el-table-column prop="port" label="服务端口号"> </el-table-column>
        <el-table-column prop="watch" label="watch"> </el-table-column>
        <el-table-column label="操作" width="850">
          <template slot-scope="{row}">

            <el-button v-if="row.name === 'build-wechat'"  @click="build(row.name)" :loading="loading" type="success" round>打包</el-button>
            <el-button  @click="viewLogs(row.pm_out_log_path)" :loading="loading" type="success" round>查看日志</el-button>
            <el-button  @click="viewLogs(row.pm_err_log_path)" :loading="loading" type="success" round>查看错误日志</el-button>
            <el-button  @click="restartProcess(row.name)" :loading="loading" type="primary" round>重启服务</el-button>
            <el-button :disabled="row.disabled" @click="stopProcess(row.name)" :loading="loading" type="danger" round>停止服务</el-button>
            <el-button :disabled="row.disabled || row.status !== 'stopped'" @click="deleteProcess(row.name)" :loading="loading" type="danger" round>删除服务</el-button>
          </template>
        </el-table-column>
      </el-table>
    </div> -->
    <div class="pre-code" v-if="!loading">
      <div>
        <span>正常日志</span>
      </div>
      <pre>
        <code>
          {{codeStr}}
        </code>
      </pre>
    </div>
    <div class="pre-code" v-if="!loading">
      <div>
        <span>错误日志</span>
      </div>
      <pre>
        <code>
          {{codeStrErr}}
        </code>
      </pre>
    </div>
  </div>
</template>

<script>
import axios from 'axios'
// 添加响应拦截器
axios.interceptors.response.use(function (response) {
  // 对响应数据做点什么
  const resData = response.data
  if (resData.success) {
    return Promise.resolve(resData)
  } else {
    window.app.$message(resData.message)
    return Promise.reject(resData)
  }
}, function (error) {
  console.log(error)
  return Promise.reject(error)
})
const qs = require('qs')
export default {
  name: 'App',
  data () {
    return {
      tableData: [],
      codeStr: '暂无日志',
      codeStrErr: '暂无日志',
      branch: 0,
      dir: 0,
      branchData: [],
      dirData: [{
        value: 0,
        label: 'dev'
      }, {
        value: 1,
        label: 'test'
      }, {
        value: 2,
        label: 'test6'
      }, {
        value: 3,
        label: 'pro'
      }],
      loading: false
    }
  },
  created () {
    this.getAllBranch()
  },
  computed: {
    dirLabel () {
      let str = ''
      for (let index = 0; index < this.dirData.length; index++) {
        const element = this.dirData[index]
        if (element.value === this.dir) {
          str = element.label
          break
        }
      }
      return str
    },
    branchLabel () {
      let str = ''
      for (let index = 0; index < this.branchData.length; index++) {
        const element = this.branchData[index]
        if (element.value === this.branch) {
          str = element.label
          break
        }
      }
      return str
    }
  },
  mounted () {
    setTimeout(() => {
      this.reDrawHighlight()
    }, 100)
  },
  methods: {
    getStatusColor (status) {
      let color = 'green'
      switch (status) {
        case 'online':
          color = 'green'
          break
        case 'stopped':
          color = 'red'
          break
        default:
          break
      }
      return color
    },
    // getPm2List () {
    //   this.loading = true
    //   axios.get('/process-list').then(({ data }) => {
    //     this.tableData = (data || []).map(item => {
    //       return {
    //         pid: item.pid,
    //         name: item.name,
    //         status: item.pm2_env?.status,
    //         port: item.pm2_env?.args ? item.pm2_env.args[0] : '暂无端口',
    //         restartTime: item.pm2_env?.restart_time,
    //         pm_out_log_path: item.pm2_env?.pm_out_log_path,
    //         pm_err_log_path: item.pm2_env?.pm_err_log_path,
    //         watch: `${item.pm2_env?.watch}`,
    //         cpu: `${item.monit?.cpu}`,
    //         memory: `${+item.monit?.memory / 1024 / 1024}MB`,
    //         disabled: item.name === 'admin'
    //       }
    //     })
    //     this.loading = false
    //   })
    // },
    getAllBranch () {
      this.loading = true
      axios.post('/get-all-branch').then(({ data }) => {
        const originArr = data.split('\n')
        this.branchData = []
        originArr.forEach((item, index) => {
          if (item) {
            this.branchData.push({
              value: index,
              label: item.trim()
            })
          }
        })
        this.loading = false
      })
    },
    // restartProcess (name) {
    //   this.loading = true
    //   axios.post('/restart-process-name', qs.stringify({
    //     name
    //   })).then(res => {
    //     this.$message.success('重启成功')
    //     this.getPm2List()
    //   })
    // },
    // stopProcess (name) {
    //   this.loading = true
    //   axios.post('/stop-process-name', qs.stringify({
    //     name
    //   })).then(res => {
    //     this.$message.success('停止成功')
    //     this.getPm2List()
    //   })
    // },
    // deleteProcess (name) {
    //   this.loading = true
    //   axios.post('/delete-process-name', qs.stringify({
    //     name
    //   })).then(res => {
    //     this.$message.success('删除成功')
    //     this.getPm2List()
    //   })
    // },
    build () {
      this.loading = true
      axios.post('/restart-build', qs.stringify({
        branch: this.branchLabel,
        dir: this.dirLabel
      })).then(res => {
        this.codeStr = res.data.stdout
        this.codeStrErr = res.data.stderr
        this.reDrawHighlight()
        console.log(res.data)
        this.loading = false
      })
    },
    flushLogs () {
      this.codeStr = '暂无日志'
      this.loading = true
      axios.post('/flush-logs').then(res => {
        this.$message.success('清理成功')
        this.loading = false
      })
    },
    viewLogs (path) {
      this.loading = true
      axios.post('/get-logs', qs.stringify({
        path
      })).then(res => {
        this.$message.success('获取成功')
        this.codeStr = res.data || '暂无日志'
        this.loading = false
        this.reDrawHighlight()
      })
    },
    reDrawHighlight () {
      this.$nextTick(() => {
        document.querySelectorAll('pre code').forEach((block) => {
          window.hljs.highlightBlock(block)
        })
      })
    }
  }
}
</script>

<style lang="scss">
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  text-align: center;
  margin-top: 60px;
  .title {
    font-size: 22px;
    margin-bottom: 20px;
    font-weight: 700;
  }
  .content-global {
    margin-bottom: 20px;
    text-align: left;
  }
  .pre-code {
    text-align: left;
  }
  .pm2-table {
    .green {
      color: #45af7a;
    }
    .red {
      color: #d02a4f;
    }
  }
}
</style>
