<template>
    <div class="detail-page">
        <el-card shadow="never"
            ><el-form :model="filters" inline
                ><el-form-item label="经销商名称"
                    ><el-select
                        v-model="filters.dealer"
                        filterable
                        clearable
                        placeholder="请选择经销商"
                        ><el-option
                            v-for="d in dealers"
                            :key="d"
                            :label="d"
                            :value="d" /></el-select></el-form-item
                ><el-form-item label="变动时间"
                    ><el-date-picker
                        v-model="filters.dateRange"
                        type="daterange"
                        value-format="YYYY-MM-DD"
                        range-separator="至" /></el-form-item
                ><el-form-item
                    ><el-button type="primary" @click="resetPage">查询</el-button
                    ><el-button @click="openRecharge">充值</el-button
                    ><el-button :icon="Download" @click="exportData">导出</el-button></el-form-item
                ></el-form
            ></el-card
        ><el-card shadow="never" class="mt-4"
            ><template #header><span>经销积分明细</span></template
            ><el-table :data="pagedRows" stripe
                ><el-table-column prop="recordId" label="记录ID" /><el-table-column
                    prop="dealer"
                    label="经销商名称" /><el-table-column
                    prop="reason"
                    label="变动原因" /><el-table-column
                    prop="amount"
                    label="变动数额" /><el-table-column
                    prop="after"
                    label="变动后余额" /><el-table-column prop="createdAt" label="变动时间"
            /></el-table>
            <div class="pagination">
                <el-pagination
                    v-model:current-page="page"
                    background
                    layout="total, prev, pager, next"
                    :total="filteredRows.length"
                /></div></el-card
        ><el-dialog v-model="rechargeVisible" title="经销商积分充值" width="520px"
            ><el-form :model="rechargeForm" label-width="100px"
                ><el-form-item label="经销商"
                    ><el-select v-model="rechargeForm.dealer" filterable
                        ><el-option
                            v-for="d in dealers"
                            :key="d"
                            :label="d"
                            :value="d" /></el-select></el-form-item
                ><el-form-item label="充值类型"
                    ><el-radio-group v-model="rechargeForm.type"
                        ><el-radio value="增加">增加</el-radio
                        ><el-radio value="扣减">扣减</el-radio></el-radio-group
                    ></el-form-item
                ><el-form-item label="充值数值"
                    ><el-input-number v-model="rechargeForm.amount" :min="1" /></el-form-item
                ><el-form-item label="说明"
                    ><el-input
                        v-model="rechargeForm.remark"
                        type="textarea" /></el-form-item></el-form
            ><template #footer
                ><el-button @click="rechargeVisible = false">取消</el-button
                ><el-button type="primary" @click="confirmRecharge">确认</el-button></template
            ></el-dialog
        >
    </div>
</template>
<script setup lang="ts" name="dealerPointDetails">
import { Download } from '@element-plus/icons-vue'
const dealers = [
    '华东食品商贸有限公司',
    '华南优选供应链有限公司',
    '华北盛达商贸有限公司',
    '西南佳品经销有限公司'
]
const reasons = ['兑换商品扣减积分', '平台充值增加积分', '平台充值扣减积分']
const rows = reactive<any[]>(
    Array.from({ length: 26 }, (_, i) => ({
        recordId: `DJF202609${10001 + i}`,
        dealer: dealers[i % 4],
        reason: reasons[i % 3],
        amount: i % 3 === 1 ? 500 : -120,
        after: 6800 + i * 120,
        createdAt: `2026-09-${String(2 - (i % 2)).padStart(2, '0')} 10:20:00`
    }))
)
const filters = reactive({ dealer: '', dateRange: [] as string[] })
const page = ref(1),
    pageSize = 10
const filteredRows = computed(() =>
    rows.filter(
        (r) =>
            (!filters.dealer || r.dealer === filters.dealer) &&
            (filters.dateRange.length !== 2 ||
                (r.createdAt.slice(0, 10) >= filters.dateRange[0] &&
                    r.createdAt.slice(0, 10) <= filters.dateRange[1]))
    )
)
const pagedRows = computed(() =>
    filteredRows.value.slice((page.value - 1) * pageSize, page.value * pageSize)
)
const resetPage = () => {
    page.value = 1
}
const rechargeVisible = ref(false)
const rechargeForm = reactive({ dealer: '', type: '增加', amount: 100, remark: '' })
const openRecharge = () => {
    rechargeVisible.value = true
}
const confirmRecharge = () => {
    if (!rechargeForm.dealer || !rechargeForm.amount || !rechargeForm.remark.trim())
        return ElMessage.warning('请完整填写充值信息')
    const amount = rechargeForm.type === '增加' ? rechargeForm.amount : -rechargeForm.amount
    rows.unshift({
        recordId: `DJF${Date.now()}`,
        dealer: rechargeForm.dealer,
        reason: rechargeForm.type === '增加' ? '平台充值增加积分' : '平台充值扣减积分',
        amount,
        after: 6800 + amount,
        createdAt: '2026-09-02 15:30:00'
    })
    rechargeVisible.value = false
    ElMessage.success('充值记录已保存')
}
const exportData = () => {
    const data = [
        ['记录ID', '经销商名称', '变动原因', '变动数额', '变动后余额', '变动时间'],
        ...filteredRows.value.map((r) => [
            r.recordId,
            r.dealer,
            r.reason,
            r.amount,
            r.after,
            r.createdAt
        ])
    ]
    const url = URL.createObjectURL(
        new Blob([`\ufeff${data.map((r) => r.join(',')).join('\n')}`], {
            type: 'text/csv;charset=utf-8;'
        })
    )
    const a = document.createElement('a')
    a.href = url
    a.download = '经销积分明细.csv'
    a.click()
    URL.revokeObjectURL(url)
}
</script>
<style scoped>
.pagination {
    display: flex;
    justify-content: flex-end;
    margin-top: 16px;
}
.detail-page :deep(.el-select) {
    width: 240px;
}
</style>
