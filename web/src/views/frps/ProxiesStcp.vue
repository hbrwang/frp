<template>
  <div>
    <el-table :data="proxies" :default-sort="{ prop: 'name', order: 'ascending' }" style="width: 100%">
      <el-table-column type="expand">
        <template slot-scope="props">
          <el-popover ref="popover4" placement="right" width="600" style="margin-left: 0px" trigger="click">
            <my-traffic-chart :proxy-name="props.row.name" />
          </el-popover>

          <el-button v-popover:popover4 type="primary" size="small" icon="view" :name="props.row.name" style="margin-bottom: 10px" @click="fetchData2">
            Traffic Statistics
          </el-button>

          <el-form label-position="left" inline class="demo-table-expand">
            <el-form-item label="Name">
              <span>{{ props.row.name }}</span>
            </el-form-item>
            <el-form-item label="Type">
              <span>{{ props.row.type }}</span>
            </el-form-item>
            <el-form-item label="Encryption">
              <span>{{ props.row.encryption }}</span>
            </el-form-item>
            <el-form-item label="Compression">
              <span>{{ props.row.compression }}</span>
            </el-form-item>
            <el-form-item label="Last Start">
              <span>{{ props.row.last_start_time }}</span>
            </el-form-item>
            <el-form-item label="Last Close">
              <span>{{ props.row.last_close_time }}</span>
            </el-form-item>
          </el-form>
        </template>
      </el-table-column>
      <el-table-column label="Name" prop="name" sortable />
      <el-table-column label="Connections" prop="conns" sortable />
      <el-table-column label="Traffic In" prop="traffic_in" :formatter="formatTrafficIn" sortable />
      <el-table-column label="Traffic Out" prop="traffic_out" :formatter="formatTrafficOut" sortable />
      <el-table-column label="status" prop="status" sortable>
        <template slot-scope="scope">
          <el-tag v-if="scope.row.status === 'online'" type="success">{{ scope.row.status }}</el-tag>
          <el-tag v-else type="danger">{{ scope.row.status }}</el-tag>
        </template>
      </el-table-column>
    </el-table>
  </div>
</template>

<script>
import Humanize from 'humanize-plus'
import Traffic from '@/components/Traffic.vue'
import { StcpProxy } from '@/utils/proxy.js'
export default {
  components: {
    'my-traffic-chart': Traffic
  },
  data() {
    return {
      proxies: []
    }
  },
  mounted() {
    this.initData()
  },
  methods: {
    formatTrafficIn(row, column) {
      return Humanize.fileSize(row.traffic_in)
    },
    formatTrafficOut(row, column) {
      return Humanize.fileSize(row.traffic_out)
    },
    async initData() {
      const res = await this.$fetch('proxy/stcp')
      if (!res.ok) {
        this.$message.warning('Get proxy info from frps failed!')
        return
      }
      const json = await res.json()
      if (!json) return

      this.proxies = []
      for (const proxyStats of json.proxies) {
        this.proxies.push(new StcpProxy(proxyStats))
      }
    }
  }
}
</script>

<style>
</style>
