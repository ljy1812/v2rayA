<script lang="ts" setup>
import { ref } from 'vue'
const { t } = useI18n()

const columns = [
  { key: 'id', label: 'ID', width: 70 },
  { key: 'host', label: t('subscription.host'), width: 220 },
  { key: 'remarks', label: t('subscription.remarks'), width: 120 },
  { key: 'status', label: t('subscription.timeLastUpdate'), width: 180 }
]

let selectRows = $ref<any[]>([])
const handleSelectionChange = (val: any) => { selectRows = val }

let isUpdating = $ref(false)
const updateSubscription = async(row: any) => {
  isUpdating = true
  const { data } = await useV2Fetch('subscription').put({ id: row.id, _type: row._type }).json()
  isUpdating = false

  if (data.value.code === 'SUCCESS')
    ElMessage.success({ message: t('common.success'), duration: 5000 })
}

const removeSubscription = async(row: any) => {
  const { data } = await useV2Fetch('touch').delete({
    touches: selectRows.map((x) => { return { id: x.id, _type: x._type } })
  }).json()

  if (data.value.code === 'SUCCESS')
    proxies.value.subs = data.value.data.touch.subscriptions
}

// 新增的数据和方法
const startRow = ref(1)
const endRow = ref(1)

const selectRange = () => {
  const table = document.querySelector('.el-table__body-wrapper') as HTMLElement
  const rows = table.querySelectorAll('.el-table__row')

  // 清空所有选中状态
  selectRows.length = 0

  for (let i = 0; i < rows.length; i++) {
    const row = rows[i]
    const index = i + 1
    if (index >= startRow.value && index <= endRow.value) {
      (row.querySelector('.el-table__cell .el-checkbox__input') as HTMLInputElement).click()
    }
  }
}

</script>

<template>
  <OperateImport />

  <!-- 新增的输入框和按钮 -->
  <div class="mb-4">
    <ElInputNumber v-model="startRow" :min="1" placeholder="开始行数" />
    <ElInputNumber v-model="endRow" :min="1" placeholder="结束行数" class="ml-2" />
    <ElButton @click="selectRange" class="ml-2">选中</ElButton>
  </div>

  <ElButton
    v-if="!(Array.isArray(selectRows) && selectRows.length === 0)"
    class="ml-2"
    @click="removeSubscription(selectRows)"
  >
    {{ t('operations.delete') }}
  </ElButton>

  <style scoped>
.mb-4 {
  margin-bottom: 1rem;
}

.ml-2 {
  margin-left: 0.5rem;
}
</style>

  <ElTable :data="proxies.subs" @selection-change="handleSelectionChange">
    <ElTableColumn type="selection" width="55" />
    <ElTableColumn v-for="c in columns" :key="c.key" :prop="c.key" :label="c.label" :min-width="c.width" />
    <ElTableColumn property="servers.length" :label="t('subscription.numberServers')" min-width="70" />
    <ElTableColumn :label="t('operations.name')" min-width="240">
      <template #default="scope">
        <ElButton size="small" :loading="isUpdating" @click="updateSubscription(scope.row)">
          <UnoIcon v-if="!isUpdating" class="ri:refresh-line mr-1" /> {{ t('operations.update') }}
        </ElButton>
        <OperateRemark :data="scope.row" />
        <OperateShare :data="scope" />
      </template>
    </ElTableColumn>
  </ElTable>
</template>
