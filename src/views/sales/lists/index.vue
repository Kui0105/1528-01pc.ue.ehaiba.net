<template>
    <div class="sales-page">
        <el-card shadow="never" class="!border-none"
            ><el-form :model="filters" inline class="filter-form">
                <el-form-item label="姓名"
                    ><el-input v-model="filters.name" clearable placeholder="请输入销售姓名"
                /></el-form-item>
                <el-form-item label="联系电话"
                    ><el-input v-model="filters.phone" clearable placeholder="请输入联系电话"
                /></el-form-item>
                <el-form-item label="创建时间"
                    ><el-date-picker
                        v-model="filters.dateRange"
                        type="daterange"
                        value-format="YYYY-MM-DD"
                        range-separator="至"
                        start-placeholder="开始日期"
                        end-placeholder="结束日期"
                /></el-form-item>
                <el-form-item
                    ><el-button type="primary" @click="resetPage">查询</el-button
                    ><el-button @click="resetFilters">重置</el-button
                    ><el-button :icon="Download" @click="exportSales">导出</el-button></el-form-item
                >
            </el-form></el-card
        >
        <el-card shadow="never" class="!border-none mt-4"
            ><template #header
                ><div class="heading">
                    <span>销售列表</span
                    ><el-button type="primary" :icon="Plus" @click="openForm()">新增销售</el-button>
                </div></template
            >
            <el-table :data="pagedRows" stripe table-layout="fixed"
                ><el-table-column
                    prop="id"
                    label="ID"
                    min-width="130"
                    class-name="first-column"
                /><el-table-column prop="name" label="姓名" min-width="160" /><el-table-column
                    prop="phone"
                    label="联系电话"
                    min-width="170"
                /><el-table-column label="销售区域" min-width="280"
                    ><template #default="{ row }"
                        ><el-tooltip :content="row.region" placement="top" effect="dark"
                            ><div class="region-tags">
                                <el-tag
                                    v-for="path in row.regionPaths"
                                    :key="path.join('-')"
                                    size="small"
                                    type="info"
                                    >{{ path.join(' / ') }}</el-tag
                                >
                            </div></el-tooltip
                        ></template
                    ></el-table-column
                ><el-table-column label="状态" width="120"
                    ><template #default="{ row }"
                        ><el-switch
                            v-model="row.enabled"
                            @change="toggleStatus(row)" /></template></el-table-column
                ><el-table-column
                    prop="createdAt"
                    label="创建时间"
                    min-width="180"
                /><el-table-column label="操作" width="170" fixed="right" class-name="last-column"
                    ><template #default="{ row }"
                        ><el-button link type="primary" @click="openForm(row)">编辑</el-button
                        ><el-button link type="danger" @click="removeRow(row)"
                            >删除</el-button
                        ></template
                    ></el-table-column
                ></el-table
            >
            <div class="pagination">
                <el-pagination
                    v-model:current-page="page"
                    background
                    layout="total, prev, pager, next"
                    :total="filteredRows.length"
                />
            </div>
        </el-card>
        <el-dialog v-model="formVisible" :title="editingId ? '编辑销售' : '新增销售'" width="620px"
            ><el-form :model="form" label-width="100px"
                ><el-form-item label="姓名" required><el-input v-model="form.name" /></el-form-item
                ><el-form-item label="联系电话" required
                    ><el-input v-model="form.phone" /></el-form-item
                ><el-form-item label="销售区域" required
                    ><el-cascader
                        v-model="form.regionPaths"
                        :options="regionOptions"
                        multiple
                        collapse-tags
                        collapse-tags-tooltip
                        clearable
                        filterable
                        :props="{ multiple: true, emitPath: true, checkStrictly: false }"
                        :show-all-levels="true"
                        placeholder="可多选省 / 市 / 区县" /></el-form-item
                ><el-form-item label="状态"
                    ><el-radio-group v-model="form.enabled"
                        ><el-radio :label="true">启用</el-radio
                        ><el-radio :label="false">禁用</el-radio></el-radio-group
                    ></el-form-item
                ></el-form
            ><template #footer
                ><el-button @click="formVisible = false">取消</el-button
                ><el-button type="primary" @click="saveRow">确认</el-button></template
            ></el-dialog
        >
    </div>
</template>
<script setup lang="ts" name="salesLists">
import { Download, Plus } from '@element-plus/icons-vue'
const regionOptions = [
    {
        value: '广东省',
        label: '广东省',
        children: [
            {
                value: '广州市',
                label: '广州市',
                children: [
                    { value: '天河区', label: '天河区' },
                    { value: '越秀区', label: '越秀区' }
                ]
            },
            { value: '深圳市', label: '深圳市', children: [{ value: '南山区', label: '南山区' }] }
        ]
    },
    {
        value: '上海市',
        label: '上海市',
        children: [
            { value: '浦东新区', label: '浦东新区' },
            { value: '徐汇区', label: '徐汇区' }
        ]
    },
    {
        value: '湖南省',
        label: '湖南省',
        children: [
            {
                value: '长沙市',
                label: '长沙市',
                children: [
                    { value: '开福区', label: '开福区' },
                    { value: '岳麓区', label: '岳麓区' },
                    { value: '芙蓉区', label: '芙蓉区' }
                ]
            }
        ]
    },
    {
        value: '四川省',
        label: '四川省',
        children: [
            { value: '成都市', label: '成都市', children: [{ value: '武侯区', label: '武侯区' }] }
        ]
    }
]
const regionSamples = [
    ['广东省', '广州市', '天河区'],
    ['上海市', '浦东新区'],
    ['湖南省', '长沙市', '开福区'],
    ['四川省', '成都市', '武侯区']
]
const rows = reactive<any[]>(
    Array.from({ length: 18 }, (_, i) => {
        const paths =
            i === 2
                ? [
                      ['湖南省', '长沙市', '开福区'],
                      ['湖南省', '长沙市', '岳麓区']
                  ]
                : [regionSamples[i % regionSamples.length]]
        return {
            id: `XS${10001 + i}`,
            name: ['张晨', '李伟', '王芳', '赵磊'][i % 4],
            phone: `138${String(10000000 + i * 137).slice(0, 8)}`,
            region: paths.map((p) => p.join(' / ')).join('；'),
            regionPaths: paths,
            enabled: i % 5 !== 0,
            createdAt: `2026-08-${String(28 - (i % 20)).padStart(2, '0')} 10:20:00`
        }
    })
)
const filters = reactive({ name: '', phone: '', dateRange: [] as string[] })
const page = ref(1)
const pageSize = 10
const filteredRows = computed(() =>
    rows.filter(
        (r) =>
            (!filters.name || r.name.includes(filters.name.trim())) &&
            (!filters.phone || r.phone.includes(filters.phone.trim())) &&
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
const resetFilters = () => {
    Object.assign(filters, { name: '', phone: '', dateRange: [] })
    resetPage()
}
const formVisible = ref(false)
const editingId = ref('')
const form = reactive({ name: '', phone: '', regionPaths: [] as string[][], enabled: true })
const openForm = (row?: any) => {
    editingId.value = row?.id || ''
    Object.assign(
        form,
        row
            ? {
                  name: row.name,
                  phone: row.phone,
                  regionPaths: row.regionPaths.map((p: string[]) => [...p]),
                  enabled: row.enabled
              }
            : { name: '', phone: '', regionPaths: [], enabled: true }
    )
    formVisible.value = true
}
const saveRow = () => {
    const selectedPaths = form.regionPaths.filter(
        (path): path is string[] => Array.isArray(path) && path.length > 0
    )
    if (!form.name.trim() || !form.phone.trim() || !selectedPaths.length)
        return ElMessage.warning('请完整填写销售资料')
    const region = selectedPaths.map((p) => p.join(' / ')).join('；')
    if (editingId.value) {
        const row = rows.find((r) => r.id === editingId.value)
        if (row)
            Object.assign(row, {
                ...form,
                region,
                regionPaths: selectedPaths.map((p) => [...p])
            })
    } else
        rows.unshift({
            ...form,
            region,
            regionPaths: selectedPaths.map((p) => [...p]),
            id: `XS${Date.now()}`,
            createdAt: '2026-09-02 10:00:00'
        })
    formVisible.value = false
    ElMessage.success(editingId.value ? '销售已更新' : '销售已新增')
}
const removeRow = async (row: any) => {
    try {
        await ElMessageBox.confirm(`确定删除销售“${row.name}”吗？`, '删除确认', { type: 'warning' })
        rows.splice(rows.indexOf(row), 1)
        ElMessage.success('删除成功')
    } catch {}
}
const toggleStatus = (row: any) => ElMessage.success(`已${row.enabled ? '启用' : '禁用'}销售`)
const exportSales = () => {
    const data = [
        ['姓名', '联系电话', '销售区域', '状态', '创建时间'],
        ...filteredRows.value.map((r) => [
            r.name,
            r.phone,
            r.region,
            r.enabled ? '启用' : '禁用',
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
    a.download = '销售列表.csv'
    a.click()
    URL.revokeObjectURL(url)
}
</script>
<style scoped>
.filter-form {
    margin-bottom: -18px;
}
.heading {
    display: flex;
    align-items: center;
    justify-content: space-between;
}
.pagination {
    display: flex;
    justify-content: flex-end;
    margin-top: 16px;
}
.sales-page :deep(.first-column .cell) {
    padding-left: 16px;
}
.sales-page :deep(.last-column .cell) {
    padding-right: 16px;
}
.sales-page :deep(.el-cascader),
.sales-page :deep(.el-input),
.sales-page :deep(.el-date-editor) {
    width: 220px;
}
.sales-page :deep(.el-dialog .el-cascader) {
    width: 100%;
}
.region-tags {
    display: flex;
    gap: 6px;
    max-width: 260px;
    overflow: hidden;
    white-space: nowrap;
}
.region-tags :deep(.el-tag) {
    flex: 0 0 auto;
    max-width: 220px;
    overflow: hidden;
    text-overflow: ellipsis;
}
.sales-page :deep(.el-dialog .el-form-item__content) {
    min-width: 0;
}
.sales-page :deep(.el-dialog .el-cascader) {
    width: 100%;
}
.sales-page :deep(.el-dialog .el-cascader__tags) {
    max-width: calc(100% - 36px);
    overflow: hidden;
    white-space: nowrap;
}
.sales-page :deep(.el-dialog .el-cascader__tags .el-tag) {
    max-width: 185px;
    overflow: hidden;
    text-overflow: ellipsis;
}
</style>
