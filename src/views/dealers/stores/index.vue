<template>
    <div class="list-page">
        <el-card shadow="never"
            ><el-form inline :model="filters"
                ><el-form-item label="门店名称"
                    ><el-input v-model="filters.name" clearable /></el-form-item
                ><el-form-item label="所属业务员"
                    ><el-select v-model="filters.business" filterable clearable
                        ><el-option
                            v-for="item in business"
                            :key="item"
                            :label="item"
                            :value="item" /></el-select></el-form-item
                ><el-form-item label="所属经销商"
                    ><el-select v-model="filters.dealer" filterable clearable
                        ><el-option
                            v-for="item in dealers"
                            :key="item"
                            :label="item"
                            :value="item" /></el-select></el-form-item
                ><el-form-item label="创建时间"
                    ><el-date-picker
                        v-model="filters.dateRange"
                        type="daterange"
                        value-format="YYYY-MM-DD"
                        range-separator="至" /></el-form-item
                ><el-form-item
                    ><el-button type="primary" @click="page = 1">查询</el-button
                    ><el-button @click="reset">重置</el-button
                    ><el-button @click="exportData">导出</el-button></el-form-item
                ></el-form
            ></el-card
        ><el-card shadow="never" class="!border-none mt-4"
            ><template #header><span>门店列表</span></template
            ><el-table :data="paged" stripe table-layout="fixed"
                ><el-table-column prop="name" label="门店名称" min-width="160" /><el-table-column label="门店门头" min-width="110"><template #default="{ row }"><el-image :src="row.frontImage" class="store-image" fit="cover" /></template></el-table-column><el-table-column label="门店货架" min-width="110"><template #default="{ row }"><el-image :src="row.shelfImage" class="store-image" fit="cover" /></template></el-table-column><el-table-column
                    prop="contact"
                    label="联系人"
                    min-width="100"
                /><el-table-column prop="phone" label="联系电话" min-width="140" /><el-table-column
                    prop="business"
                    label="所属业务员"
                    min-width="120"
                /><el-table-column
                    prop="dealer"
                    label="所属经销商"
                    min-width="200"
                /><el-table-column prop="region" label="所属区域" min-width="170" /><el-table-column
                    prop="address"
                    label="详细地址"
                    min-width="240"
                    show-overflow-tooltip
                /><el-table-column label="状态" min-width="90"
                    ><template #default="{ row }"
                        ><el-switch v-model="row.enabled" /></template></el-table-column
                ><el-table-column
                    prop="createdAt"
                    label="创建时间"
                    min-width="170"
                /><el-table-column label="操作" width="150" fixed="right"
                    ><template #default="{ row }"
                        ><el-button link type="success" @click="recharge(row)">充值</el-button
                        ><el-button link type="danger" @click="remove(row)"
                            >删除</el-button
                        ></template
                    ></el-table-column
                ></el-table
            >
            <div class="pagination">
                <el-pagination
                    v-model:current-page="page"
                    background
                    layout="total,prev,pager,next"
                    :total="filtered.length"
                /></div></el-card
        ><el-dialog v-model="rechargeVisible" title="门店积分充值" width="480px"
            ><el-form :model="rechargeForm" label-width="90px"
                ><el-form-item label="门店"
                    ><el-input v-model="rechargeForm.name" disabled /></el-form-item
                ><el-form-item label="充值类型"
                    ><el-radio-group v-model="rechargeForm.type"
                        ><el-radio value="增加">增加</el-radio
                        ><el-radio value="扣减">扣减</el-radio></el-radio-group
                    ></el-form-item
                ><el-form-item label="充值数值"
                    ><el-input-number
                        v-model="rechargeForm.amount"
                        :min="1" /></el-form-item></el-form
            ><template #footer
                ><el-button @click="rechargeVisible = false">取消</el-button
                ><el-button type="primary" @click="confirmRecharge">确认</el-button></template
            ></el-dialog
        >
    </div>
</template>
<script setup lang="ts" name="storeLists">
const business = ['张明', '李婷', '王强', '赵颖']
const dealers = ['华东食品商贸有限公司', '华南优选供应链有限公司', '华北盛达商贸有限公司']
const rows = reactive<any[]>(
    Array.from({ length: 22 }, (_, i) => ({
        id: `MD${10001 + i}`,
        name: ['阳光便利店', '优选超市', '惠民商行', '佳佳便利'][i % 4],
        contact: ['陈伟', '刘芳', '周杰'][i % 3],
        phone: `139${String(10000000 + i * 131).slice(0, 8)}`,
        business: business[i % 4],
        dealer: dealers[i % 3],
        frontImage: `https://images.unsplash.com/photo-1556742049-0cfed4f6a45d?auto=format&fit=crop&w=120&h=80&q=80&sig=${i}`,
        shelfImage: `https://images.unsplash.com/photo-1601598851547-4302969d5e9d?auto=format&fit=crop&w=120&h=80&q=80&sig=${i}`,
        region: ['湖南省长沙市开福区', '广东省广州市天河区', '上海市浦东新区'][i % 3],
        address: [
            '湖南省长沙市开福区福元西路88号',
            '广东省广州市天河区体育西路168号',
            '上海市浦东新区张江路99号'
        ][i % 3],
        enabled: i % 6 !== 0,
        createdAt: `2026-09-${String(3 - (i % 3)).padStart(2, '0')} 11:20:00`
    }))
)
const filters = reactive({ name: '', business: '', dealer: '', dateRange: [] as string[] })
const page = ref(1)
const filtered = computed(() =>
    rows.filter(
        (r) =>
            (!filters.name || r.name.includes(filters.name)) &&
            (!filters.business || r.business === filters.business) &&
            (!filters.dealer || r.dealer === filters.dealer) &&
            (filters.dateRange.length !== 2 ||
                (r.createdAt.slice(0, 10) >= filters.dateRange[0] &&
                    r.createdAt.slice(0, 10) <= filters.dateRange[1]))
    )
)
const paged = computed(() => filtered.value.slice((page.value - 1) * 10, page.value * 10))
const reset = () => {
    Object.assign(filters, { name: '', business: '', dealer: '', dateRange: [] })
    page.value = 1
}
const rechargeVisible = ref(false)
const rechargeForm = reactive({ name: '', type: '增加', amount: 100 })
const recharge = (r: any) => {
    rechargeForm.name = r.name
    rechargeVisible.value = true
}
const confirmRecharge = () => {
    rechargeVisible.value = false
    ElMessage.success('充值记录已保存')
}
const remove = async (r: any) => {
    try {
        await ElMessageBox.confirm(`确认删除门店“${r.name}”吗？`, '删除确认', { type: 'warning' })
        rows.splice(rows.indexOf(r), 1)
    } catch {}
}
const exportData = () => {
    const data = [
        [
            '门店名称',
            '门店门头',
            '门店货架',
            '联系人',
            '联系电话',
            '所属业务员',
            '所属经销商',
            '所属区域',
            '详细地址',
            '状态',
            '创建时间'
        ],
        ...filtered.value.map((r) => [
            r.name,
            r.frontImage,
            r.shelfImage,
            r.contact,
            r.phone,
            r.business,
            r.dealer,
            r.region,
            r.address,
            r.enabled ? '启用' : '禁用',
            r.createdAt
        ])
    ]
    const url = URL.createObjectURL(
        new Blob([data.map((r) => r.join(',')).join('\n')], { type: 'text/csv' })
    )
    const a = document.createElement('a')
    a.href = url
    a.download = '门店列表.csv'
    a.click()
    URL.revokeObjectURL(url)
}
</script>
<style scoped>
.store-image { width: 64px; height: 48px; border-radius: 4px; }
.list-page :deep(.el-input),
.list-page :deep(.el-select) {
    width: 190px;
}
.pagination {
    display: flex;
    justify-content: flex-end;
    margin-top: 16px;
}
</style>
