<template>
  <div>
    <div class="ml20 mt30">
      <d-button-group>
        <slot name="extendBtn"></slot>
        <d-button v-if="showBrokerChart" @click="goBrokerChart" class="button">Broker监控
          <icon name="cpu" style="margin-left: 3px;"/>
        </d-button>
        <d-button v-if="showHostChart" @click="goHostChart" class="button">主机监控
          <icon name="monitor" style="margin-left: 3px;"/>
        </d-button>
        <d-button type="primary" @click="getList" class="button">刷新
          <icon name="refresh-cw" style="margin-left: 3px;"></icon>
        </d-button>
      </d-button-group>
    </div>
    <my-table :data="tableData" :showPin="showTablePin" style="height: 400px;overflow-y:auto" :showPagination="false"/>
  </div>
</template>

<script>
import MyTable from '../../components/common/myTable'
import crud from '../../mixins/crud.js'
import apiRequest from '../../utils/apiRequest.js'
import {timeStampToString} from '../../utils/dateTimeUtils'

export default {
  name: 'broker',
  components: {MyTable},
  mixins: [crud],
  props: {
    topicId: {
      type: String,
      default: ''
    },
    showBrokerChart: {
      type: Boolean,
      default: true
    },
    showHostChart: {
      type: Boolean,
      default: true
    }
  },
  data () {
    return {
      urls: {
        search: `/broker/findByTopic`,
        startInfo: '/monitor/start',
        getUrl: `/grafana/getRedirectUrl`
      },
      tableData: {
        rowData: [],
        colData: [
          {
            title: 'ID',
            key: 'id'
          },
          {
            title: 'IP',
            key: 'ip'
          },
          {
            title: '端口',
            key: 'port'
          },
          {
            title: '启动时间',
            key: 'startupTime'
          },
          {
            title: 'revision',
            key: 'revision'
          }
        ]
      },
      monitorUId: {
        broker: this.$store.getters.uIds.broker,
        host: this.$store.getters.uIds.host
      }
    }
  },
  methods: {
    getList () {
      this.showTablePin = true
      apiRequest.postBase(this.urls.search, {}, this.topicId, false).then((data) => {
        data.data = data.data || []
        this.tableData.rowData = data.data
        for (let i = 0; i < this.tableData.rowData.length; i++) {
          this.getBrokerStatus(this.tableData.rowData, i)
        }
        this.showTablePin = false
      })
    },
    getBrokerStatus (rowData, i) {
      apiRequest.get(this.urls.startInfo + '/' + rowData[i].id).then((data) => {
        if (data.code === 200) {
          this.tableData.rowData[i].startupTime = timeStampToString(data.data.startupTime)
          this.tableData.rowData[i].revision = data.data.revision
        } else {
          this.tableData.rowData[i].startupTime = '不存活'
        }
        this.$set(this.tableData.rowData, i, this.tableData.rowData[i])
      })
    },
    goBrokerChart () {
      apiRequest.get(this.urls.getUrl + '/' + this.monitorUId.broker, {}, {}).then((data) => {
        let url = data.data || ''
        if (url.indexOf('?') < 0) {
          url += '?'
        } else if (!url.endsWith('?')) {
          url += '&'
        }
        url = url + 'var-topic=' + this.topicId
        window.open(url)
      })
    },
    goHostChart () {
      apiRequest.get(this.urls.getUrl + '/' + this.monitorUId.host, {}, {}).then((data) => {
        let url = data.data || ''
        if (url.indexOf('?') < 0) {
          url += '?'
        } else if (!url.endsWith('?')) {
          url += '&'
        }
        url = url + 'var-topic=' + this.topicId
        window.open(url)
      })
    }
  }
}
</script>

<style scoped>

</style>
