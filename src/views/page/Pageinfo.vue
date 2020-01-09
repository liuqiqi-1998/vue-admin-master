<template>
    <section>
        <!--工具条-->
        <el-col :span="24" class="toolbar" style="padding-bottom: 0px;">
            <el-form :inline="true" :model="filters">
                <el-form-item>
                    <el-button type="primary" size="small" @click="handleAdd">新增</el-button>
                </el-form-item>
                <el-form-item>
                    <el-button type="primary" size="small" @click="handleAdd">页面静态化</el-button>
                </el-form-item>
            </el-form>
        </el-col>

        <!--列表-->
        <el-table :data="pageinfos" highlight-current-row  @selection-change="selectChange"  style="width: 100%;">

            <el-table-column type="selection" width="55">
            </el-table-column>
            <el-table-column type="index" width="60">
            </el-table-column>
            <el-table-column prop="name" label="网页名称" width="100" sortable>
            </el-table-column>
            <el-table-column prop="alias" label="别名" width="100" sortable>
            </el-table-column>
            <el-table-column prop="type" label="类型" width="80" sortable>
                <template scope="scope">
                    <span v-if="scope.row.type==1">静态页面</span>
                    <span v-else-if="scope.row.type==0">动态页面</span>
                    <span v-else>其他页面</span>

                </template>
            </el-table-column>
            <el-table-column prop="physicalPath" label="物理路径" width="120" sortable>
            </el-table-column>
            <el-table-column prop="templateUrl" label="模板路径" width="120" sortable>
            </el-table-column>
            <el-table-column prop="pageConfig.dataKey" label="数据点" width="100" sortable>
            </el-table-column>
            <el-table-column prop="pageConfig.pageUrl" label="页面路径" width="120" sortable>
            </el-table-column>
            <el-table-column prop="createTime" label="创建时间" min-width="150" sortable :formatter="formatStartTime">
            </el-table-column>
            <el-table-column prop="site.name" label="站点" width="90" sortable>
            </el-table-column>

            <el-table-column label="操作" width="150">
                <template scope="scope">
                    <el-button size="small" @click="handleEdit(scope.$index, scope.row)">编辑</el-button>
                    <el-button type="danger" size="small" @click="handleDel(scope.$index, scope.row)">删除</el-button>
                </template>
            </el-table-column>
        </el-table>

        <!--工具条-->
        <el-col :span="24" class="toolbar">
            <el-pagination layout="prev, pager, next" @current-change="handleCurrentChange" :page-size="pageSize" :total="total" style="float:right;">
            </el-pagination>
        </el-col>

<!----------------------------------------->
        <!--新增-->
        <el-dialog title="添加" :visible.sync="formVisible" :close-on-click-modal="false">
            <el-form :model="pageInfo" label-width="80px" :rules="formRules" ref="pageInfo">
                <el-form-item label="名称">
                    <el-input v-model="pageInfo.name" auto-complete="off"></el-input>
                </el-form-item>
                <el-form-item label="别名">
                    <el-input v-model="pageInfo.alias" auto-complete="off"></el-input>
                </el-form-item>
                <el-form-item label="物理路径">
                    <el-input v-model="pageInfo.physicalPath" auto-complete="off"></el-input>
                </el-form-item>
                <el-form-item label="站点选择">
                    <el-input v-model="pageInfo.siteId" auto-complete="off"></el-input>
                </el-form-item>
                <el-form-item label="上传模板">
                    <el-input v-model="pageInfo.templateUrl" auto-complete="off"></el-input>
                    <el-upload
                            class="upload-demo"
                            action="http://localhost:9527/services/file/file/upload"
                            :on-success="handleUploadSuccess"
                            :file-list="fileList">
                        <el-button size="small" type="primary">点击上传</el-button>
                        <div slot="tip" class="el-upload__tip">只能上传jpg/png文件，且不超过500kb</div>
                    </el-upload>
                </el-form-item>
                <el-form-item label="模板文件">
                    <el-input v-model="pageInfo.templateFile" auto-complete="off"></el-input>
                </el-form-item>

            </el-form>
            <div slot="footer" class="dialog-footer">
                <el-button @click.native="formVisible = false">取消</el-button>
                <el-button type="primary" @click="handleSave" >提交</el-button>
            </div>
        </el-dialog>

    </section>
</template>

<script>
    export default {
        name: "course",
        data(){
            return {

                pageinfos:[],
                sels:[],//多选框选中的行
                pageNum:1,
                pageSize:10,
                total:0,
                fileList:[],
                formVisible:false,
                pageInfo:{
                    name:"",
                    alias:"",
                    type:null,
                    physicalPath:"",
                    siteId:null,
                    templateUrl:"",
                    templateFile:""
                },
                formRules:{

                }



            }
        },
        methods:{

            formatStartTime(row, column){
                return this.formatTime(row.createTime);
            },
            formatTime(time){
                if (!time){
                    return "";
                }
                let date = new Date(time);
                let year = date.getFullYear();
                let month = date.getMonth()+1;
                let day = date.getDay();
                let hours = date.getHours();
                let minutes = date.getMinutes();
                let seconds = date.getSeconds();
                let timeStr = year+"-"+this.formatNum(month)+"-"+this.formatNum(day)+" "
                            + this.formatNum(hours)+":"+this.formatNum(minutes)+":"+this.formatNum(seconds)
                return timeStr
            },
            formatNum(num){
                if (num<10){
                    return "0"+num;
                }
                return num;
            },
            //获取课程
            getPageinfos(){
                let param = {};
                param.page = this.pageNum;
                param.rows = this.pageSize;
                this.$http.post("/page/pageInfo/page",param)
                    .then(res=>{
                        let {total,rows} = res.data;
                        this.total = total;
                        this.pageinfos = rows;
                    })
            },
            selectChange(selection){
                this.sels = selection;
            },
            handleAdd(){
                this.formVisible = true;
            },
            //保存
            handleSave(){
                this.$refs.pageInfo.validate((valid) => {
                    //校验表单成功后才做一下操作
                    if (valid) {
                        this.$confirm('确认提交吗？', '提示', {}).then(() => {
                            //拷贝后面对象的值到新对象,防止后面代码改动引起模型变化
                            let para = Object.assign({}, this.pageInfo);
                            this.$http.post("/page/pageInfo/save",para).then((res) => {
                                this.$message({
                                    message: '操作成功!',
                                    type: 'success'
                                });
                                //重置表单
                                this.$refs['pageInfo'].resetFields();
                                //关闭对话框
                                this.formVisible = false;
                                //刷新数据
                                this.getPageinfos();
                            });
                        });
                    }
                })
            },
            //模板上传成功回调
            handleUploadSuccess(response, file, fileList){
                let {success,message,resultObj} = response;
                if (success){
                    this.pageInfo.templateUrl = resultObj;
                }else{
                    this.$message({
                        message:message ,
                        type: 'error'
                    });
                }
            },
            handleCurrentChange(val){
                this.pageNum = val;
                this.getPageinfos();
            },

        },
        mounted(){
            this.getPageinfos();
            this.getcourseTypes();
        }
    }
</script>

<style scoped>

</style>