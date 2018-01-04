<template>
    <div>
        <div style="margin-bottom: 10px">
            <el-input placeholder="任务名称 / 创建者" icon="search" @change="searchTask" style="width: 240px"></el-input>
            <el-button type="primary" style="float: right">新增任务</el-button>
        </div>
        <el-table :data="taskList"
                  :filter-method="searchTask"
                  stripe border
                  style="width: 100%"
                  max-height="680">
            <el-table-column label="任务名称">
                <template scope="scope">
                    <el-popover trigger="hover" placement="top">
                        <p>{{ scope.row.taskDesc }}</p>
                        <p slot="reference" class="name-wrapper">
                            {{ scope.row.taskName }}
                        </p>
                    </el-popover>
                </template>
            </el-table-column>
            <el-table-column prop="startTime" abel="开始时间"></el-table-column>
            <el-table-column prop="stopTime" label="结束时间"></el-table-column>
            <el-table-column prop="startMode" label="启动模式" :formatter="formatStartMode"></el-table-column>
            <el-table-column prop="taskStatus" label="任务状态" :formatter="formatTaskStatus"></el-table-column>
            <el-table-column prop="creator" label="创建者"></el-table-column>
            <el-table-column prop="createdTime" label="创建时间"></el-table-column>
            <el-table-column prop="updatedTime" label="最后操作时间"></el-table-column>
            <el-table-column label="操作">
                <template scope="scope">
                    <el-button v-if="scope.row.taskStatus == 0" @click="startTask(scope.row.taskId)" type="text"
                               size="small">启动
                    </el-button>
                    <el-button v-if="scope.row.taskStatus == 1" @click="stopTask(scope.row.taskId)" type="text"
                               size="small">停止
                    </el-button>
                    <el-button type="text" size="small">编辑</el-button>
                </template>
            </el-table-column>
        </el-table>
    </div>
</template>

<script>
    const apiUrl = 'http://localhost:9998/';
    export default {
        data() {
            return {
                taskSearchCondition: '',
                taskList: [{
                    taskId: 1,
                    taskName: '散标数据同步',
                    startTime: '2017-10-25 10:00:00',
                    stopTime: '',
                    startMode: 0,
                    startCron: '',
                    taskStatus: 0,
                    creator: 'jinjun.zhang',
                    createdTime: '2017-10-30 12:01:13',
                    updatedTime: '2017-10-30 12:01:13',
                    taskDesc: 't_loan表迁移到t_sku_info表，只同步网贷标的'
                }, {
                    taskId: 2,
                    taskName: '资管数据监控',
                    startTime: '',
                    stopTime: '',
                    startMode: 1,
                    startCron: '0/30 * * * * ?',
                    taskStatus: 1,
                    creator: 'xianfeng.wu',
                    createdTime: '2017-12-22 12:01:13',
                    updatedTime: '2017-12-22 12:01:13',
                    taskDesc: '监听t_user_invest表变换，上报监管数据'
                }]
            };
        },
        created() {
        },
        methods: {
            startTask: function (taskId) {
                console.log("start task : " + taskId);
            },
            stopTask: function (taskId) {
                console.log("stop task : " + taskId);
            },
            formatStartMode: function (row, column) {
                return row.startMode === 0 ? '正常' : row.startCron;
            },
            formatTaskStatus: function (row, column) {
                return row.startMode === 0 ? '停止' : '启动';
            },
            searchTask: function (val) {

            }
        }
    };
</script>

<style>
</style>
