<template>
    <div>
        <div style="margin-bottom: 10px">
            <el-input placeholder="任务名称 / 创建者" icon="search" @change="searchTask" style="width: 240px"></el-input>
            <el-button type="primary" style="float: right" @click="showTaskModal">新增任务</el-button>
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
            <el-table-column prop="startTime" label="开始时间"></el-table-column>
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
        <el-dialog title="" :visible.sync="dialogFormVisible">
            <el-tabs v-model="activeName" @tab-click="tabClick">
                <el-tab-pane label="任务信息" name="first"></el-tab-pane>
                <el-tab-pane label="数据源配置" name="second"></el-tab-pane>
                <el-tab-pane label="数据处理策略" name="third"></el-tab-pane>
            </el-tabs>
            <div v-show="activeName == 'first'">
                <el-form :model="taskInfoForm" :rules="infoFormRules" ref="taskInfoForm">
                    <el-form-item label="任务名称" :label-width="formLabelWidth" prop="taskName">
                        <el-input v-model="taskInfoForm.taskName" auto-complete="off"></el-input>
                    </el-form-item>
                    <el-form-item label="任务描述" :label-width="formLabelWidth" prop="taskDesc">
                        <el-input type="textarea" v-model="taskInfoForm.taskDesc"></el-input>
                    </el-form-item>
                    <el-form-item label="开始时间" :label-width="formLabelWidth" prop="startTime">
                        <el-date-picker
                                v-model="taskInfoForm.startTime"
                                type="datetime"
                                placeholder="默认：任务启动时间">
                        </el-date-picker>
                    </el-form-item>
                    <el-form-item label="结束时间" :label-width="formLabelWidth" prop="startTime">
                        <el-date-picker
                                v-model="taskInfoForm.stopTime"
                                type="datetime"
                                placeholder="默认：增量获取">
                        </el-date-picker>
                    </el-form-item>
                    <el-form-item label="启动模式" :label-width="formLabelWidth" prop="startMode">
                        <el-radio v-model="taskInfoForm.startMode" :label="0">标准启动</el-radio>
                        <el-radio v-model="taskInfoForm.startMode" :label="1">cron表达式</el-radio>
                        <el-form-item label="启动模式" :label-width="formLabelWidth" prop="startCron"
                                      v-if="taskInfoForm.startMode==1">
                            <el-input v-model="taskInfoForm.startCron" placeholder="cron"></el-input>
                        </el-form-item>

                    </el-form-item>
                </el-form>
            </div>
            <div v-show="activeName == 'second'">
                <el-form :model="taskConfigForm" :rules="sourceFormRules" ref="taskConfigForm">
                    <el-form-item label="数据源" :label-width="formLabelWidth" prop="sourceDBId">
                        <el-select filterable v-model="taskConfigForm.sourceDBId" placeholder="请选择数据源">
                            <el-option
                                    v-for="item in watchInstanceList"
                                    :key="item.id"
                                    :label="item.host+' - '+item.name"
                                    :value="item.id">
                            </el-option>
                        </el-select>
                    </el-form-item>
                    <el-form-item label="数据源库" :label-width="formLabelWidth" prop="watchDB">
                        <el-select filterable v-model="taskConfigForm.watchDB" placeholder="请选择数据源库">
                            <el-option
                                    v-for="item in watchSchemaList"
                                    :key="item.key"
                                    :label="item.name+' - '+ item.comment"
                                    :value="item.name">
                            </el-option>
                        </el-select>
                    </el-form-item>
                    <el-form-item label="数据源表" :label-width="formLabelWidth" prop="watchTable">
                        <el-select filterable v-model="taskConfigForm.watchTable" placeholder="请选择数据源表">
                            <el-option
                                    v-for="item in watchTableList"
                                    :key="item.key"
                                    :label="item.name+' - ' +item.comment"
                                    :value="item.name">
                            </el-option>
                        </el-select>
                    </el-form-item>
                    <el-form-item label="监听字段" :label-width="formLabelWidth">
                        <el-transfer
                                filterable
                                :filter-method="filterColumn"
                                :render-content="renderDbTable"
                                filter-placeholder="请输入监听字段"
                                v-model="taskConfigForm.watchColumns"
                                :data="watchColumnsList">
                        </el-transfer>
                    </el-form-item>
                    <div>
                        <el-form-item label="收集字段" :label-width="formLabelWidth">
                            <el-transfer
                                    filterable
                                    :filter-method="filterColumn"
                                    :render-content="renderDbTable"
                                    filter-placeholder="请输入收集字段"
                                    v-model="taskConfigForm.catchColumns"
                                    :data="watchColumnsList">
                            </el-transfer>
                        </el-form-item>
                    </div>
                </el-form>
            </div>
            <div v-show="activeName == 'third'">
                <el-form :model="taskConfigForm" :rules="targetFormRules" ref="targetForm0" v-if="taskConfigForm.catchMode==0">
                    <el-form-item label="收集方式" :label-width="formLabelWidth" prop="catchMode">
                        <el-select v-model="taskConfigForm.catchMode" placeholder="请选择">
                            <el-option
                                    v-for="item in catchModeArray"
                                    :key="item.id"
                                    :label="item.name"
                                    :value="item.id">
                            </el-option>
                        </el-select>
                    </el-form-item>
                    <el-form-item label="Redis实例" :label-width="formLabelWidth" prop="targetRedisId">
                        <el-select v-model="taskConfigForm.targetRedisId" placeholder="请选择">
                            <el-option
                                    v-for="item in redisList"
                                    :key="item.id"
                                    :label="item.host+' - '+item.name"
                                    :value="item.id">
                            </el-option>
                        </el-select>
                    </el-form-item>
                    <el-form-item label="实例index" :label-width="formLabelWidth" prop="targetRedisIndex">
                        <el-select v-model="taskConfigForm.targetRedisIndex" placeholder="请选择">
                            <el-option
                                    v-for="item in 16"
                                    :key="item-1"
                                    :label="item-1"
                                    :value="item-1">
                            </el-option>
                        </el-select>
                    </el-form-item>
                    <el-form-item label="Key前缀" :label-width="formLabelWidth" prop="targetRedisQueue">
                        <el-input v-model="taskConfigForm.targetRedisQueue"></el-input>
                    </el-form-item>
                    <el-form-item label="分发数量" :label-width="formLabelWidth" prop="modNumber">
                        <el-input v-model="taskConfigForm.modNumber"></el-input>
                    </el-form-item>
                    <el-form-item label="取模字段" :label-width="formLabelWidth" prop="modColumn">
                        <el-select v-model="taskConfigForm.modColumn" placeholder="请选择">
                            <el-option
                                    v-for="item in catchColumnsList"
                                    :key="item.name"
                                    :label="item.name+' - '+item.comment"
                                    :value="item.name">
                            </el-option>
                        </el-select>
                    </el-form-item>
                </el-form>
                <el-form :model="taskConfigForm" :rules="targetFormRules" ref="targetForm1" v-if="taskConfigForm.catchMode==1">
                    <el-form-item label="收集方式" :label-width="formLabelWidth" prop="catchMode">
                        <el-select v-model="taskConfigForm.catchMode" placeholder="请选择">
                            <el-option
                                    v-for="item in catchModeArray"
                                    :key="item.id"
                                    :label="item.name"
                                    :value="item.id">
                            </el-option>
                        </el-select>
                    </el-form-item>
                    <el-form-item label="目标实例" :label-width="formLabelWidth" prop="targetDBId">
                        <el-select v-model="taskConfigForm.targetDBId" placeholder="请选择目标实例">
                            <el-option
                                    v-for="item in watchInstanceList"
                                    :key="item.id"
                                    :label="item.host+' - '+item.name"
                                    :value="item.id">
                            </el-option>
                        </el-select>
                    </el-form-item>
                    <el-form-item label="目标库" :label-width="formLabelWidth" prop="targetDB">
                        <el-select v-model="taskConfigForm.targetDB" placeholder="请选择目标库">
                            <el-option
                                    v-for="item in targetSchemaList"
                                    :key="item.name"
                                    :label="item.name+' - '+ item.comment"
                                    :value="item.name">
                            </el-option>
                        </el-select>
                    </el-form-item>
                    <el-form-item label="目标表" :label-width="formLabelWidth" prop="targetTable">
                        <el-select v-model="taskConfigForm.targetTable" placeholder="请选择目标表">
                            <el-option
                                    v-for="item in targetTableList"
                                    :key="item.name"
                                    :label="item.name+' - ' +item.comment"
                                    :value="item.name">
                            </el-option>
                        </el-select>
                    </el-form-item>
                    <el-form-item label="字段映射" :label-width="formLabelWidth" prop="targetColumns">
                        <div style="overflow: hidden;">
                            <div class="dragging-wrap">
                                <ul class="list-group">
                                    <li v-if="catchColumnsList.length==0">
                                        暂无数据
                                    </li>
                                    <li class="list-group-item" v-for="element in catchColumnsList">
                                        <el-popover trigger="hover" placement="top">
                                            <p>{{element.name}}</p>
                                            <p>{{element.comment}}</p>
                                            <span slot="reference" class="name-wrapper">
                                                    {{element.name}}
                                                </span>
                                        </el-popover>
                                    </li>
                                </ul>
                            </div>
                            <div class="dragging-wrap">
                                <draggable class="list-group" element="ul" v-model="targetColumnsList"
                                           :options="dragOptions" :move="onMove" @start="isDragging=true"
                                           @end="isDragging=false">
                                    <transition-group type="transition" :name="'flip-list'">
                                        <li class="list-group-item" v-for="element in targetColumnsList"
                                            :key="element.key">
                                            <el-popover trigger="hover" placement="top">
                                                <p>{{element.label}}</p>
                                                <p>{{element.comment}}</p>
                                                <span slot="reference" class="name-wrapper">
                                                         <i class="el-icon-rank"></i>
                                                        {{element.label}}
                                                </span>
                                            </el-popover>
                                        </li>
                                    </transition-group>
                                </draggable>
                            </div>
                        </div>
                    </el-form-item>
                </el-form>
                <el-form :model="taskConfigForm" :rules="targetFormRules" ref="targetForm2" v-if="taskConfigForm.catchMode==2">
                    <el-form-item label="收集方式" :label-width="formLabelWidth" prop="catchMode">
                        <el-select v-model="taskConfigForm.catchMode" placeholder="请选择">
                            <el-option
                                    v-for="item in catchModeArray"
                                    :key="item.id"
                                    :label="item.name"
                                    :value="item.id">
                            </el-option>
                        </el-select>
                    </el-form-item>
                    <el-form-item label="文件名称" :label-width="formLabelWidth" prop="targetFile">
                        <el-input v-model="taskConfigForm.targetFile"></el-input>
                    </el-form-item>
                </el-form>
                <el-form :model="taskConfigForm" :rules="targetFormRules" ref="targetForm3" v-if="taskConfigForm.catchMode==3">
                    <el-form-item label="收集方式" :label-width="formLabelWidth" prop="catchMode">

                        <el-select v-model="taskConfigForm.catchMode" placeholder="请选择">
                            <el-option
                                    v-for="item in catchModeArray"
                                    :key="item.id"
                                    :label="item.name"
                                    :value="item.id">
                            </el-option>
                        </el-select>
                    </el-form-item>
                    <el-form-item label="Topic前缀" :label-width="formLabelWidth" prop="targetCmqTopic">
                        <el-input v-model="taskConfigForm.targetCmqTopic"></el-input>
                    </el-form-item>
                    <el-form-item label="Topic密码" :label-width="formLabelWidth" prop="targetCmqPassword">
                        <el-input type="password" v-model="taskConfigForm.targetCmqPassword "></el-input>
                    </el-form-item>
                    <el-form-item label="分发数量" :label-width="formLabelWidth" prop="modNumber">
                        <el-input v-model="taskConfigForm.modNumber"></el-input>
                    </el-form-item>
                    <el-form-item label="取模字段" :label-width="formLabelWidth" prop="modColumn">
                        <el-select v-model="taskConfigForm.modColumn" placeholder="请选择">
                            <el-option
                                    v-for="item in catchColumnsList"
                                    :key="item.name"
                                    :label="item.name+' - '+item.comment"
                                    :value="item.name">
                            </el-option>
                        </el-select>
                    </el-form-item>
                </el-form>
            </div>
            <div slot="footer" class="dialog-footer">
                <el-button @click="dialogFormVisible = false">取 消</el-button>
                <template v-if="activeName == 'first'">
                    <el-button type="primary" @click="submitForm('taskInfoForm')">
                        <span>下一步</span>
                    </el-button>
                </template>
                <template v-if="activeName == 'second'">
                    <el-button type="primary" @click="submitForm('taskConfigForm')">
                        <span>下一步</span>
                    </el-button>
                </template>
                <template v-if="activeName == 'third'">
                    <el-button type="primary" @click="submitForm('target_setting')">
                        <span>确定</span>
                    </el-button>
                </template>
            </div>
        </el-dialog>
    </div>
</template>

<script>
    const sancronos = require("sancronos-validator");
    import axios from 'axios'
    import draggable from 'vuedraggable'

    const message = ['vue.draggable', 'draggable', 'component', 'for', 'vue.js 2.0', 'based', 'on', 'Sortablejs']
    const apiUrl = 'http://localhost:9998/';
    export default {
        components: {
            draggable,
        },
        data() {
            return {
                self: this,
                infoFormValid: false,
                sourceFormValid: false,
                targetFormName: 'targetForm0',
                editable: true,
                isDragging: false,
                delayedDragging: false,
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
                }],
                activeName: 'first',
                dialogFormVisible: false,
                taskInfoForm: {
                    taskId: null,
                    taskName: '',
                    taskDesc: '',
                    startTime: '',
                    stopTime: '',
                    startMode: 0,
                    startCron: '',
                    taskConfig: {}
                },
                taskConfigForm: {
                    taskId: null,//任务ID
                    sourceDBId: 0, //监听实例ID
                    watchDB: '', //监听库名
                    watchTable: '',// 监听表名
                    watchColumns: [], //监听字段
                    catchColumns: [], // 收集字段
                    catchMode: 0, //收集模式
                    targetCmqTopic: '', //目标topic名称
                    targetCmqPassword: '', //目标topic密码
                    targetFile: '', //目标文件名称,
                    targetRedisId: 0, //目标redis实例ID ,
                    targetRedisIndex: 0, //目标redis dbindex
                    targetRedisQueue: '', //目标Redis key前缀，
                    targetDBId: 0, //目标实例
                    targetDB: '',
                    targetTable: '',//目标表名,
                    targetColumns: [],
                    modColumn: '', //取模字段
                    modNumber: 0 //取模数量
                },
                formLabelWidth: '120px',
                infoFormRules: {
                    taskName: [
                        {required: true, message: '请输入任务名称', trigger: 'blur'}
                    ],
                    taskDesc: [
                        {required: true, message: '请输入任务描述', trigger: 'blur'}
                    ],
                    startMode: [
                        {required: true, message: '请选择启动模式', trigger: 'change'}
                    ],
                    startCron: [
                        {validator: this.validateCron, trigger: 'blur'}
                    ]
                },
                sourceFormRules: {
                    sourceDBId: [
                        {required: true, message: '请选择数据源', trigger: 'change'}
                    ],
                    watchDB: [
                        {required: true, message: '请选择数据源库', trigger: 'change'}
                    ],
                    watchTable: [
                        {required: true, message: '请选择数据源表', trigger: 'change'}
                    ]
                },
                targetFormRules: {
                    catchMode: [
                        {required: true, message: '请选择收集模式', trigger: 'change'}
                    ],
                    targetRedisId: [
                        {required: true, message: '请选择redis实例ID', trigger: 'change'}
                    ],
                    targetRedisIndex: [
                        {required: true, message: '请选择实例index', trigger: 'change'}
                    ],
                    targetRedisQueue: [
                        {required: true, message: '请输入Key前缀', trigger: 'blur'}
                    ],
                    modNumber: [
                        {required: true, message: '请输入分发数量', trigger: 'blur'}
                    ],
                    modColumn: [
                        {required: true, message: '请选择取模字段', trigger: 'change'}
                    ],
                    /*db模式*/
                    targetDBId: [
                        {required: true, message: '请选择目标实例', trigger: 'change'}
                    ],
                    targetDB: [
                        {required: true, message: '请选择目标库', trigger: 'change'}
                    ],
                    targetTable: [
                        {required: true, message: '请选择目标表', trigger: 'change'}
                    ],
                    /*file模式*/
                    targetFile: [
                        {required: true, message: '请输入文件名称', trigger: 'blur'}
                    ],
                    /*cmq模式*/
                    targetCmqTopic: [
                        {required: true, message: '请输入Topic前缀', trigger: 'blur'}
                    ],
                    targetCmqPassword: [
                        {required: true, message: '请输入Topic密码', trigger: 'blur'}
                    ]
                },
                catchModeArray: [
                    {
                        id: 0,
                        name: 'Redis'
                    },
                    {
                        id: 1,
                        name: 'DB'
                    },
                    {
                        id: 2,
                        name: 'File'
                    },
                    {
                        id: 3,
                        name: 'CMQ'
                    }
                ],
                watchInstanceList: [],
                watchSchemaList: [],
                watchTableList: [],
                watchColumnsList: [],
                watchColumnsMap: {},
                targetSchemaList: [],
                targetTableList: [],
                targetColumnsList: [],
                redisList: [],
                catchColumnsList: []
            };
        },
        mounted() {
            this.getDB();
            this.getRedis();
        },
        watch: {
            'sourceForm.sourceDBId'(val, oldVal) {
                if (val) {
                    this.taskConfigForm.watchDB = '';
                    this.taskConfigForm.watchTable = '';
                    this.getDBLib(val)
                }
            },
            'sourceForm.watchDB'(val, oldVal) {
                if (val) {
                    this.taskConfigForm.watchTable = '';
                    this.getDBOri(this.taskConfigForm.sourceDBId, val)
                }
            },
            'sourceForm.watchTable'(val, oldVal) {
                if (val) {
                    this.watchColumnsList = [];
                    this.taskConfigForm.watchColumns = [];
                    this.taskConfigForm.catchColumns = [];
                    this.getDBTable(this.taskConfigForm.sourceDBId, this.taskConfigForm.watchDB, val)
                }
            },
            'taskConfigForm.targetDBId'(val, oldVal) {
                if (val) {
                    this.taskConfigForm.targetDB = '';
                    this.taskConfigForm.targetTable = '';
                    this.getDBLib(val)
                }
            },
            'taskConfigForm.targetDB'(val, oldVal) {
                if (val) {
                    this.taskConfigForm.targetTable = '';
                    this.getDBOri(this.taskConfigForm.targetDBId, val)
                }
            },
            'taskConfigForm.targetTable'(val, oldVal) {
                if (val) {
                    this.targetColumnsList = [];
                    this.taskConfigForm.targetColumns = [];
                    this.getDBTable(this.taskConfigForm.targetDBId, this.taskConfigForm.targetDB, val)
                }
            },
            'activeName'(val, oldVal) {
                if (val == 'third') {
                    this.catchColumnsList = [];
                    this.taskConfigForm.catchColumns.forEach((item) => {
                        var obj = this.watchColumnsMap[item]
                        this.catchColumnsList.push(obj)

                    })
                    console.log(this.catchColumnsList)
                }
            },
            'taskConfigForm.catchMode'(val, oldVal) {
                if ('undefined' !== val) {
                    this.targetFormName = 'targetForm' + val;
                    console.log(this.$refs['targetForm' + oldVal]);
                    this.$refs['targetForm' + oldVal].clearValidate();
                    this.taskConfigForm.catchMode = val;
                }
            },
            isDragging(newValue) {
                if (newValue) {
                    this.delayedDragging = true
                    return
                }
                this.$nextTick(() => {
                    this.delayedDragging = false
                })
            }
        },
        methods: {
            validateCron: function (rule, value, callback) {
                try {
                    sancronos.isValid(badtab, true);
                    callback();
                } catch (err) {
                    callback(new Error('请输入正确的corn表达式'));
                }
            },
            filterColumn: function (query, item) {
                return item.label.indexOf(query) > -1;
            },
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

            },
            showTaskModal: function () {
                this.dialogFormVisible = true;
            },
            addTask: function () {
                this.taskInfoForm.taskConfig = this.taskConfigForm;
                console.log(this.taskInfoForm);
                this.dialogFormVisible = false;
            },
            submitForm(formName) {
                if (this.activeName === 'third' && !this.infoFormValid) {
                    this.$message({
                        message: '请先完成任务信息的填写',
                        type: 'warning'
                    });
                    this.activeName = 'first';
                    return;
                }

                if (this.activeName === 'third' && !this.sourceFormValid) {
                    this.$message({
                        message: '请先完成源信息的配置',
                        type: 'warning'
                    });
                    this.activeName = 'second';
                    return;
                }

                if ('target_setting' === formName) {
                    formName = this.targetFormName;
                }
                this.$refs[formName].validate((valid) => {
                    if (valid) {
                        switch (this.activeName) {
                            case 'first':
                                this.activeName = 'second';
                                this.infoFormValid = true;
                                break;
                            case 'second':
                                if (!this.checkSource()) {
                                    return;
                                }
                                this.activeName = 'third';
                                this.sourceFormValid = true;
                                break;
                            case 'third':
                                this.addTask();
                                break;
                        }
                    } else {
                        console.log('error submit!!');
                        return false;
                    }
                });
            },
            tabClick: function () {

            },
            getDB() {
                axios.get('/config/db')
                    .then((response) => {
                        this.watchInstanceList = response.data.data;
                    })
            },
            getRedis() {
                axios.get('/config/redis')
                    .then((response) => {
                        this.redisList = response.data.data;
                    })
            },
            getDBLib(instance) {
                axios.get('config/db/' + instance)
                    .then((response) => {
                        if (this.activeName == 'second') {
                            this.watchSchemaList = response.data.data;
                        } else {
                            this.targetSchemaList = response.data.data;
                        }
                    })
            },
            getDBOri(instance, schema) {
                axios.get('config/db/' + instance + '/' + schema)
                    .then((response) => {
                        if (this.activeName == 'second') {
                            this.watchTableList = response.data.data;
                        } else {
                            this.targetTableList = response.data.data;
                        }

                    })
            },
            getDBTable(instance, schema, table) {
                axios.get('config/db/' + instance + '/' + schema + '/' + table)
                    .then((response) => {
                        response.data.data.forEach((item, index) => {
                            if (this.activeName === 'second') {
                                this.watchColumnsMap[item.name] = item;
                                this.watchColumnsList.push({
                                    label: item.name,
                                    key: item.name,
                                    comment: item.comment
                                })
                            } else {
                                this.targetColumnsList.push({
                                    label: item.name,
                                    key: item.name,
                                    comment: item.comment
                                })
                            }
                        })
                    })
            },
            renderDbTable(h, option) {
                return <el-popover trigger="hover" placement="top"><div>{option.label}</div><div>{option.comment}</div><span slot="reference" class="name-wrapper">{option.label}</span></el-popover>
            },
            onMove({relatedContext, draggedContext}) {
                const relatedElement = relatedContext.element;
                const draggedElement = draggedContext.element;
                return (!relatedElement || !relatedElement.fixed) && !draggedElement.fixed
            },
            checkSource() {
                if (this.taskConfigForm.watchColumns.length == 0) {
                    this.$message({
                        message: '请至少选择一个监听字段',
                        type: 'warning'
                    });
                    return false
                }
                if (this.taskConfigForm.catchColumns.length == 0) {
                    this.$message({
                        message: '请至少选择一个收集字段',
                        type: 'warning'
                    });
                    return false
                }
                return true
            }
        },
        computed: {
            dragOptions() {
                return {
                    animation: 0,
                    group: 'description',
                    disabled: !this.editable,
                    ghostClass: 'ghost'
                };
            }
        },
    };
</script>

<style>
    .el-dialog {
        min-width: 800px;
    }

    .step-title {
        padding-left: 50px;
    }

    .el-dialog__body {
        padding-top: 0;
    }

    .el-date-editor--datetimerange.el-input, .el-date-editor--datetimerange.el-input__inner {
        width: 455px;
    }

    .el-select {
        width: 100%;
    }

    .el-transfer-panel {
        width: 250px;
    }

    .el-dialog__body {
        overflow: auto;
        max-height: 500px;
    }

    .flip-list-move {
        transition: transform 0.5s;
    }

    .no-move {
        transition: transform 0s;
    }

    .ghost {
        opacity: .5;
        background: #C8EBFB;
    }

    .list-group {
        min-height: 20px;
    }

    .list-group-item {
        cursor: move;
    }

    .list-group-item i {
        cursor: pointer;
    }

    .dragging-wrap {
        width: 50%;
        float: left;
    }

    ul, li {
        list-style: none;
        padding: 0;
        margin: 0;
    }

    .list-group {
        border: 1px solid #ebeef5;
        width: 250px;
    }

    .list-group li {
        border-bottom: 1px solid #ebeef5;
        padding-left: 10px;
    }

    .list-group-item span {
        width: 100%;
        display: block;
    }
</style>
