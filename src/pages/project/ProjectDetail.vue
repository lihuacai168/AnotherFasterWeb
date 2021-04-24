<template>

    <div>
        <ul class="title-project">
            <li class="title-li" title="Test API Project">
                <b>{{ projectInfo.name }}</b>
                <b class="desc-li">{{ projectInfo.desc }}</b>

            </li>
        </ul>

        <ul class="project_detail">
            <li class="pull-left">
                <p class="title-p"><i class="iconfont">&#xe74a;</i> &nbsp;{{ projectInfo.api_count }} 个接口</p>
                <p class="desc-p">接口总数</p>
            </li>
            <li class="pull-left">
                <p class="title-p"><i class="iconfont">&#xe61f;</i> &nbsp;{{ projectInfo.case_step_count }} 个监控接口</p>
                <p class="desc-p">监控接口个数</p>
            </li>
            <li class="pull-left">
                <p class="title-p"><i class="iconfont">&#xe61e;</i> &nbsp;{{ projectInfo.task_count }} 项任务</p>
                <p class="desc-p">定时任务个数</p>
            </li>

            <li class="pull-left">
                <p class="title-p"><i class="iconfont">&#xe6da;</i> &nbsp;{{ projectInfo.case_count }} 个用例</p>
                <p class="desc-p">用例集总数</p>
            </li>

        </ul>
        <ul class="project_detail">
<!--            <li class="pull-left">-->
<!--                <p class="title-p"><i class="iconfont">&#xe609;</i> &nbsp;{{ projectInfo.host_count }} 套环境</p>-->
<!--                <p class="desc-p">环境总数</p>-->
<!--            </li>            -->
            <li class="pull-left">
                <p class="title-p"><i class="iconfont">&#xe609;</i> &nbsp;{{ projectInfo.api_cover_rate }}% 接口覆盖率</p>
                <p class="desc-p">用例步骤和接口总数的比例</p>
            </li>
            <li class="pull-left">
                <p class="title-p"><i class="iconfont">&#xe66e;</i> 共 {{ projectInfo.report_count }} 个报告
                    {{ projectInfo.report_success }} 成功,{{ projectInfo.report_fail }} 失败 </p>
                <p class="desc-p">测试报告总数</p>
            </li>
            <li class="pull-left">
                <p class="title-p"><i class="iconfont">&#xee32;</i> &nbsp;{{ projectInfo.config_count }} 套配置</p>
                <p class="desc-p">配置总数</p>
            </li>
            <li class="pull-left">
                <p class="title-p"><i class="iconfont">&#xe692;</i> &nbsp;{{ projectInfo.variables_count }} 对变量</p>
                <p class="desc-p">全局变量对数</p>
            </li>
        </ul>

        <div id="myChart"
             :style="{width: '100%', height: '300px', 'overflow-x': 'hidden', 'overflow-y': 'hidden'}"></div>


    </div>
</template>

<script>
export default {
    name: "ProjectDetail",
    data() {
        return {
            visitInfo: {},
            projectInfo: {}
        }
    },
    methods: {
        drawLine() {
            // 基于准备好的dom，初始化echarts实例
            let myChart = this.$echarts.init(document.getElementById('myChart'))

            console.log("drawLine: " + this.visitInfo["count"])
            debugger
            // 绘制图表
            myChart.setOption({
                title: {text: '近七天用户访问量'},
                tooltip: {},
                xAxis: {
                    data: this.visitInfo["create_time"]

                },
                yAxis: {type: 'value'},
                series: [{
                    name: '访问量',
                    type: 'bar',
                    data: this.visitInfo["count"]
                }]
            });
        },

        getVisitData() {
            const project = this.$route.params.id;
            debugger
            this.$api.getVisit({
                params: {
                    project: project
                }
            }).then(res => {
                this.visitInfo = res
                this.drawLine();
            })
        },
        success(resp) {
            this.$notify({
                message: resp["msg"],
                type: 'success',
                duration: this.$store.state.duration
            });
        },
        failure(resp) {
            this.$notify.error({
                message: resp.msg,
                duration: this.$store.state.duration
            });
        },

        getProjectDetail() {
            const pk = this.$route.params.id;
            this.$api.getProjectDetail(pk).then(res => {
                this.projectInfo = res
            })
        }
    },
    mounted() {
        this.getVisitData()
        this.getProjectDetail();
    },

    beforeMount() {
    }

}
</script>

<style scoped>


.desc-p {
    padding-top: 10px;
    font-size: 12px;
    color: #b6b6b6;
}

.title-p {
    font-size: 18px;
    margin-top: 10px;
}

.title-project li a {
    font-size: 12px;
    text-decoration: none;
    color: #a3a3a3;
    margin-left: 20px;

}

.pull-left {
    float: left;
    margin-left: 10px;
}

.project_detail li {
    margin-top: 10px;
    text-indent: 20px;
    display: inline-block;
    height: 90px;
    width: calc(20% - 1.5px);
    border: 1px solid #ddd;
}

.project_detail {
    height: 100px;
    margin-top: 20px;
}

.title-project {
    margin-top: 40px;
    margin-left: 10px;
}

ul li {
    list-style: none;
}

.title-li {
    font-size: 24px;
    color: #607d8b;
}

.desc-li {
    margin-top: 10px;
    color: #b6b6b6;
    font-size: 14px;
}
</style>
