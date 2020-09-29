<template>
    <el-container>
        <el-header style="padding-top: 10px; height: 50px;">
            <div style="overflow: hidden">
                <el-row :gutter="50">
                    <el-col :span="8"  v-if="testData.count >= 0">
                        <el-input placeholder="请输入用例名称" clearable v-model="search" @keyup.enter.native="getTestList"
                                  class="input-with-select"
                                  style="width: 500px">
                            <el-button slot="append" icon="el-icon-search" @click="getTestList"></el-button>


                                <el-select v-model="searchType" slot="prepend" placeholder="用例" @change="searchTypeChangeHandle">
                                     <el-option label="用例" value="1"></el-option>
                                     <el-option label="API" value="2"></el-option>

                                </el-select>

                        </el-input>
                    </el-col>

                    <el-col  :span="2" style="margin-left: 80px">
                        <el-dropdown @command="caseTypeChangeHandle">
                        <el-button type="primary">
                            类型
                            <i class="el-icon-arrow-down el-icon--right"></i>
                        </el-button>
                        <el-dropdown-menu slot="dropdown">
                            <el-dropdown-item command="1">冒烟用例</el-dropdown-item>
                            <el-dropdown-item command="2">集成用例</el-dropdown-item>
                            <el-dropdown-item command="3">监控脚本</el-dropdown-item>
                            <el-dropdown-item command="">所有</el-dropdown-item>
                        </el-dropdown-menu>
                    </el-dropdown>
                    </el-col>

                    <el-col :span="2">
                        <el-button
                            type="primary"
                            @click="resetSearch"
                        >重置
                        </el-button>
                    </el-col>

                    <el-col :span="7">
                        <el-pagination
                            @current-change="handleCurrentChange"
                            :current-page.sync="currentPage"
                            :page-size="11"
                            v-show="testData.count !== 0 "
                            background
                            layout="total, prev, pager, next, jumper"
                            :total="testData.count"
                        >
                        </el-pagination>
                    </el-col>
                </el-row>
            </div>
        </el-header>

        <el-container>
            <el-main style="padding: 0; margin-left: 10px;">
                <div style="position: fixed; bottom: 0; right:0; left: 500px; top: 160px">
                    <el-dialog
                        v-if="dialogTableVisible"
                        :visible.sync="dialogTableVisible"
                        width="70%"
                        :modal-append-to-body="false"
                    >
                        <report :summary="summary"></report>
                    </el-dialog>

                    <el-dialog
                        title="Run Case"
                        :visible.sync="dialogTreeVisible"
                        width="45%"
                        :modal-append-to-body="false"
                    >
                        <div>
                            <div>
                                <el-row :gutter="2">
                                    <el-col :span="8">
                                        <el-switch
                                            style="margin-top: 10px"
                                            v-model="asyncs"
                                            active-color="#13ce66"
                                            inactive-color="#ff4949"
                                            active-text="异步执行"
                                            inactive-text="同步执行">
                                        </el-switch>
                                    </el-col>
                                    <el-col :span="10">
                                        <el-input
                                            v-show="asyncs"
                                            clearable
                                            placeholder="请输入报告名称"
                                            v-model="reportName"
                                            :disabled="false">
                                        </el-input>

                                    </el-col>
                                </el-row>
                            </div>
                            <div style="margin-top: 20px">
                                <el-input
                                    placeholder="输入关键字进行过滤"
                                    v-model="filterText"
                                    size="medium"
                                    clearable
                                    prefix-icon="el-icon-search"
                                >
                                </el-input>

                                <el-tree
                                    :filter-node-method="filterNode"
                                    :data="dataTree"
                                    show-checkbox
                                    node-key="id"
                                    :expand-on-click-node="false"
                                    check-on-click-node
                                    :check-strictly="true"
                                    :highlight-current="true"
                                    ref="tree"
                                >
                            <span class="custom-tree-node"
                                  slot-scope="{ node, data }"
                            >
                                <span><i class="iconfont" v-html="expand"></i>&nbsp;&nbsp;{{ node.label }}</span>
                            </span>
                                </el-tree>
                            </div>

                        </div>
                        <span slot="footer" class="dialog-footer">
                    <el-button @click="dialogTreeVisible = false">取 消</el-button>
                    <el-button type="primary" @click="runTree">确 定</el-button>
                  </span>
                    </el-dialog>
                    <el-table
                        highlight-current-row
                        v-loading="loading"
                        ref="multipleTable"
                        :data="testData.results"
                        :show-header="testData.count !== 0 "
                        stripe
                        height="calc(100%)"
                        @cell-mouse-enter="cellMouseEnter"
                        @cell-mouse-leave="cellMouseLeave"
                        @selection-change="handleSelectionChange"
                    >
                        <el-table-column
                            type="selection"
                            width="55"
                        >
                        </el-table-column>

                        <el-table-column
                            label="用例名称"
                            width="400"
                        >
                            <template slot-scope="scope">
                                <div>{{scope.row.name}}</div>
                            </template>
                        </el-table-column>

                        <el-table-column
                            label="API个数"
                            width="100"
                        >
                            <template slot-scope="scope">
                                <div>{{scope.row.length}} 个</div>
                            </template>
                        </el-table-column>

                        <el-table-column
                            label="用例类型"
                            width="100"
                        >
                            <template slot-scope="scope">
                                <el-tag v-if="scope.row.tag==='冒烟用例'">{{scope.row.tag}}</el-tag>
                                <el-tag v-if="scope.row.tag==='集成用例'" type="success">{{scope.row.tag}}</el-tag>
                                <el-tag v-if="scope.row.tag==='监控脚本'" type="danger">{{scope.row.tag}}</el-tag>
                            </template>
                        </el-table-column>

                        <el-table-column
                            label="更新时间"
                            width="200"

                        >
                            <template slot-scope="scope">
                                <div>{{scope.row.update_time|datetimeFormat}}</div>

                            </template>
                        </el-table-column>

                        <el-table-column
                            label="创建人"
                            width="100"

                        >
                            <template slot-scope="scope">
                                <div>{{scope.row.creator}}</div>

                            </template>
                        </el-table-column>

                        <el-table-column
                            label="更新人"
                            width="100"

                        >
                            <template slot-scope="scope">
                                <div>{{scope.row.updater}}</div>

                            </template>
                        </el-table-column>

                        <el-table-column
                            label="用例操作"
                        >
                            <template slot-scope="scope">
                                <el-row v-show="currentRow === scope.row">
                                    <el-button
                                        type="info"
                                        icon="el-icon-edit"
                                        circle size="mini"
                                        :title="userName === scope.row.creator || isSuperuser ? '编辑' : '只有用例创建者才能编辑'"
                                        :disabled="userName != scope.row.creator && !isSuperuser"
                                        @click="handleEditTest(scope.row.id)"
                                    ></el-button>

                                    <el-button
                                        type="primary"
                                        icon="el-icon-caret-right"
                                        circle size="mini"
                                        title="运行用例"
                                        @click="handleRunTest(scope.row.id, scope.row.name)"
                                    ></el-button>

                                    <el-button
                                        type="success"
                                        icon="el-icon-document-copy"
                                        circle size="mini"
                                        title="复制用例"
                                        @click="handleCopyTest(scope.row.id, scope.row.name)"
                                    >
                                    </el-button>

                                    <el-button
                                        type="danger"
                                        icon="el-icon-delete"
                                       :title="userName === scope.row.creator || isSuperuser ? '编辑' : '只有用例创建者才能删除'"
                                        :disabled="userName != scope.row.creator && !isSuperuser"
                                        circle size="mini"
                                        @click="handleDelTest(scope.row.id)"
                                    >
                                    </el-button>

                                    <el-button
                                        type="warning"
                                        icon="el-icon-refresh"
                                        :title="userName === scope.row.creator || isSuperuser ? '编辑' : '只有用例创建者才能同步'"
                                        :disabled="userName != scope.row.creator && !isSuperuser"
                                        circle size="mini"
                                        @click="handleSyncCaseStep(scope.row.id)"
                                    >
                                    </el-button>
                                </el-row>
                            </template>
                        </el-table-column>
                    </el-table>
                </div>
            </el-main>

        </el-container>
    </el-container>

</template>

<script>
    import Report from '../../../reports/DebugReport'

    export default {

        name: "TestList",
        components: {
            Report
        },

        props: {
            run: Boolean,
            back: Boolean,
            project: {
                require: true
            },
            host: {
                require: true
            },
            node: {
                require: false
            },
            del: Boolean
        },

        watch: {
            filterText(val) {
                this.$refs.tree.filter(val);
            },

            run() {
                this.asyncs = false;
                this.reportName = "";
                this.getTree();
            },
            node() {
                this.search = '';
                this.searchType = '1';
                this.getTestList();
            },

            back() {
                this.getTestList();
            },

            del() {
                if (this.selectTest.length !== 0) {
                    this.$confirm('此操作将永久删除测试用例集，是否继续?', '提示', {
                        confirmButtonText: '确定',
                        cancelButtonText: '取消',
                        type: 'warning',
                    }).then(() => {
                        this.$api.delAllTest({data: this.selectTest}).then(resp => {
                            this.getTestList();
                        })
                    })
                } else {
                    this.$notify.warning({
                        title: '提示',
                        message: '请至少选择一个用例集',
                        duration: 1000
                    })
                }
            },

            search(){
                this.getTestList()
            },

        },
        data() {
            return {
                isSuperuser: this.$store.state.is_superuser,
                userName: this.$store.state.user,
                search: '',
                reportName: '',
                asyncs: false,
                filterText: '',
                expand: '&#xe65f;',
                dialogTreeVisible: false,
                dataTree: {},
                loading: false,
                dialogTableVisible: false,
                selectTest: [],
                summary: {},
                currentRow: '',
                testData: {
                    count: 0,
                    results: []
                },
                currentPage: 1,
                caseType: '',
                searchType: '1' // 1：用例名称搜索 2：api名称或者api url
            }
        },

        methods: {
            getTree() {
                this.$api.getTree(this.$route.params.id, {params: {type: 2}}).then(resp => {
                    this.dataTree = resp.tree;
                    this.dialogTreeVisible = true;
                })
            },

            filterNode(value, data) {
                if (!value) return true;
                return data.label.indexOf(value) !== -1;
            },

            runTree() {
                this.dialogTreeVisible = false;
                const relation = this.$refs.tree.getCheckedKeys();
                if (relation.length === 0) {
                    this.$notify.error({
                        title: '提示',
                        message: '请至少选择一个节点',
                        duration: 1500
                    });
                } else {
                    this.$api.runSuiteTree({
                        "host": this.host,
                        "project": this.project,
                        "relation": relation,
                        "async": this.asyncs,
                        "name": this.reportName
                    }).then(resp => {
                        if (resp.hasOwnProperty("status")) {
                            this.$message.info({
                                message: resp.msg,
                                duration: 1500
                            });
                        } else {
                            this.summary = resp;
                            this.dialogTableVisible = true;
                        }
                    })
                }
            },

            handleRunTest(id, name) {
                this.loading = true;
                this.$api.runTestByPk(id, {params: {project: this.project, name: name, host: this.host}}).then(resp => {
                    this.summary = resp;
                    this.dialogTableVisible = true;
                    this.loading = false;
                }).catch(resp => {
                    this.loading = false;
                })
            },
            handleCurrentChange(val) {
                this.$api.getTestPaginationBypage({
                    params: {
                        page: this.currentPage,
                        project: this.project,
                        node: this.node,
                        search: this.search,
                        searchType: this.searchType,
                    }
                }).then(resp => {
                    this.testData = resp;
                })
            },

            handleEditTest(id) {
                this.$api.editTest(id).then(resp => {
                    this.$emit('testStep', resp);
                })
            },

            handleCopyTest(id, name) {
                this.$prompt('请输入用例集名称', '提示', {
                    confirmButtonText: '确定',
                    inputPattern: /^[\s\S]*.*[^\s][\s\S]*$/,
                    inputErrorMessage: '用例集不能为空',
                    inputValue: name,
                }).then(({value}) => {
                    this.$api.coptTest(id, {
                        'name': value,
                        'relation': this.node,
                        'project': this.project
                    }).then(resp => {
                        if (resp.success) {
                            this.getTestList();
                        } else {
                            this.$message.error(resp.msg);
                        }
                    })
                })
            },

            handleSelectionChange(val) {
                this.selectTest = val;
            },

            handleDelTest(id) {
                this.$confirm('此操作将永久删除该测试用例集，是否继续?', '提示', {
                    confirmButtonText: '确定',
                    cancelButtonText: '取消',
                    type: 'warning',
                }).then(() => {
                    this.$api.deleteTest(id).then(resp => {
                        if (resp.success) {
                            this.getTestList();
                        } else {
                            this.$message.error(resp.msg)
                        }
                    })
                })
            },
            handleSyncCaseStep(id) {
                this.$confirm('同步测试用例中的用例步骤，是否继续?', '提示', {
                    confirmButtonText: '确定',
                    cancelButtonText: '取消',
                    type: 'warning',
                }).then(() => {
                    this.$api.syncTest(id).then(resp => {
                        if (resp.success) {
                            this.$notify.success("同步用例步骤成功")
                            this.getTestList();
                        } else {
                            this.$message.error(resp.msg)
                        }
                    })
                })
            },
            resetSearch() {
                this.searchType = "1",
                this.search = "",
                this.node = "",
                this.caseType = "",
                this.getTestList()
            },

            caseTypeChangeHandle(command){
                this.caseType = command
                debugger
                this.getTestList()
            },

            searchTypeChangeHandle(value){
                this.searchType = value
                this.getTestList()
            },


            getTestList() {
                this.$api.testList({
                    params: {
                        project: this.project,
                        node: this.node,
                        search: this.search,
                        searchType: this.searchType,
                        caseType: this.caseType
                    }
                }).then(resp => {
                    this.testData = resp;
                })
            },
            cellMouseEnter(row) {
                this.currentRow = row;
            },

            cellMouseLeave(row) {
                this.currentRow = '';
            }
        },
        mounted() {
            this.getTestList()
        }
    }
</script>

<style scoped>
    .el-select {
        width: 80px;
    }

</style>
