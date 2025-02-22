// Licensed to the Apache Software Foundation (ASF) under one
// or more contributor license agreements.  See the NOTICE file
// distributed with this work for additional information
// regarding copyright ownership.  The ASF licenses this file
// to you under the Apache License, Version 2.0 (the
// "License"); you may not use this file except in compliance
// with the License.  You may obtain a copy of the License at
//
//   http://www.apache.org/licenses/LICENSE-2.0
//
// Unless required by applicable law or agreed to in writing,
// software distributed under the License is distributed on an
// "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
// KIND, either express or implied.  See the License for the
// specific language governing permissions and limitations
// under the License.

<template>
  <a-modal
    :visible="showGroupActionModal"
    :closable="true"
    :maskClosable="false"
    :cancelText="$t('label.cancel')"
    @cancel="handleCancel"
    width="50vw"
    style="top: 20px;overflow-y: auto"
    centered
  >
    <span slot="title"> {{ $t(message.title) }}
      <a
        v-if="message.docHelp || $route.meta.docHelp"
        style="margin-left: 5px"
        :href="$config.docBase + '/' + (message.docHelp || $route.meta.docHelp)"
        target="_blank">
        <a-icon type="question-circle-o"></a-icon>
      </a>
    </span>
    <template slot="footer">
      <a-button key="back" @click="handleCancel"> {{ $t('label.close') }} </a-button>
    </template>
    <a-card :bordered="false" style="background:#f1f1f1">
      <div><a-icon type="check-circle-o" style="color: #52c41a; margin-right: 8px"/> {{ $t('label.success') + ': ' + succeededCount }}</div>
      <div><a-icon type="close-circle-o" style="color: #f5222d; margin-right: 8px"/> {{ $t('state.failed') + ': ' + failedCount }}</div>
      <div><a-icon type="sync-o" style="color: #1890ff; margin-right: 8px"/> {{ $t('state.inprogress') + ': ' + selectedItems.filter(item => item.status === 'InProgress').length || 0 }}</div>
    </a-card>
    <a-divider />
    <div v-if="showGroupActionModal">
      <a-table
        v-if="selectedItems.length > 0"
        size="middle"
        :columns="selectedColumns"
        :dataSource="tableChanged ? filteredItems : selectedItems"
        :rowKey="(record, idx) => (this.$route.path.includes('/template') || this.$route.path.includes('/iso')) ? record.zoneid: record.id"
        :pagination="true"
        @change="handleTableChange"
        style="overflow-y: auto">
        <div slot="status" slot-scope="text">
          <status :text=" text ? text : $t('state.inprogress') " displayText></status>
        </div>
        <template slot="algorithm" slot-scope="record">
          {{ returnAlgorithmName(record.algorithm) }}
        </template>
        <template slot="privateport" slot-scope="record">
          {{ record.privateport }} - {{ record.privateendport }}
        </template>
        <template slot="publicport" slot-scope="record">
          {{ record.publicport }} - {{ record.publicendport }}
        </template>
        <template slot="protocol" slot-scope="record">
          {{ record.protocol | capitalise }}
        </template>
        <template slot="startport" slot-scope="record">
          {{ record.icmptype || record.startport >= 0 ? record.icmptype || record.startport : $t('label.all') }}
        </template>
        <template slot="endport" slot-scope="record">
          {{ record.icmpcode || record.endport >= 0 ? record.icmpcode || record.endport : $t('label.all') }}
        </template>
        <template slot="vm" slot-scope="record">
          <div><a-icon type="desktop"/> {{ record.virtualmachinename }} ({{ record.vmguestip }})</div>
        </template>
      </a-table>
      <br/>
    </div>
  </a-modal>
</template>
<script>
import Status from '@/components/widgets/Status'

export default {
  name: 'BulkActionProgress',
  components: {
    Status
  },
  filters: {
    capitalise: val => {
      if (val === 'all') return 'All'
      return val.toUpperCase()
    }
  },
  props: {
    showGroupActionModal: {
      type: Boolean,
      default: false
    },
    selectedItems: {
      type: Array,
      default: () => []
    },
    selectedColumns: {
      type: Array,
      default: () => []
    },
    message: {
      type: Object,
      default: () => {}
    }
  },
  created () {
    this.filteredItems = this.selectedItems
  },
  data () {
    return {
      appliedFilterStatus: {},
      filteredItems: [],
      filterItemsTimer: null,
      tableChanged: false
    }
  },
  inject: ['parentFetchData'],
  watch: {
    succeededCount (count) {
      if (count > 0) {
        this.filterItemsDelayed()
      }
    },
    failedCount (count) {
      if (count > 0) {
        this.filterItemsDelayed()
      }
    }
  },
  computed: {
    succeededCount () {
      return this.selectedItems.filter(item => item.status === 'success').length || 0
    },
    failedCount () {
      return this.selectedItems.filter(item => item.status === 'failed').length || 0
    }
  },
  methods: {
    handleTableChange (pagination, filters, sorter) {
      this.filteredItems = this.selectedItems
      this.appliedFilterStatus = filters.status
      this.filterItems()
      this.tableChanged = true
    },
    filterItems () {
      if (this.appliedFilterStatus?.length > 0) {
        this.filteredItems = this.selectedItems.filter(item => {
          if (this.appliedFilterStatus.includes(item.status)) {
            return item
          }
        })
      }
    },
    filterItemsDelayed () {
      clearTimeout(this.filterItemsTimer)
      this.filterItemsTimer = setTimeout(() => {
        this.filterItems()
      }, 50)
    },
    handleCancel () {
      this.filteredItems = []
      this.tableChanged = false
      this.$emit('handle-cancel')
    },
    returnAlgorithmName (name) {
      switch (name) {
        case 'leastconn':
          return 'Least connections'
        case 'roundrobin' :
          return 'Round-robin'
        case 'source':
          return 'Source'
        default :
          return ''
      }
    }
  }
}
</script>
