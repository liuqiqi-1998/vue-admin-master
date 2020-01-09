<template>
    <section>
        <!--工具条-->
        <el-col :span="24" class="toolbar" style="padding-bottom: 0px;">
            <el-form :inline="true" :model="filters">
                <el-form-item>
                    <el-input v-model="filters.keyword" placeholder="关键字"></el-input>
                </el-form-item>
                <el-form-item>
                    <el-select v-model="filters.status" clearable placeholder="课程状态">
                        <el-option
                                v-for="item in options"
                                :key="item.value"
                                :label="item.label"
                                :value="item.value">
                        </el-option>
                    </el-select>
                </el-form-item>
                <el-form-item>
                    <el-button type="primary" v-on:click="getCourses">查询</el-button>
                </el-form-item>
                <el-form-item>
                    <el-button type="primary" @click="handleAdd">新增课程</el-button>
                    <el-button type="primary" @click="handleMarket" :disabled="this.sels.length==0">市场信息</el-button>
                    <el-button type="primary" @click="handlePic" :disabled="this.sels.length==0">图片信息</el-button>
                    <el-button type="primary" @click="handleOnline" :disabled="this.sels.length==0">上架</el-button>
                    <el-button type="primary" @click="handleOffline" :disabled="this.sels.length==0">下架</el-button>
                </el-form-item>
            </el-form>
        </el-col>

        <!--列表-->
        <el-table :data="courses" highlight-current-row  @selection-change="selectChange"  style="width: 100%;">

            <el-table-column type="selection" width="55">
            </el-table-column>
            <el-table-column type="index" width="60">
            </el-table-column>
            <el-table-column prop="name" label="名称" width="120" sortable>
            </el-table-column>
            <el-table-column prop="courseType.name" label="课程类型" width="100" sortable>
            </el-table-column>
            <el-table-column prop="tenantName" label="机构" width="100" sortable>
            </el-table-column>
            <el-table-column prop="userName" label="创建用户" width="120" sortable>
            </el-table-column>
            <el-table-column prop="startTime" label="上架时间" min-width="180" sortable :formatter="formatStartTime">
            </el-table-column>
            <el-table-column prop="endTime" label="下架时间" min-width="180" sortable :formatter="formatEndTime">
            </el-table-column>
            <el-table-column prop="status" label="课程状态" min-width="180" sortable>
                <template scope="scope">
                    <span v-if="scope.row.status == 1" style="color: green;">
                        上架
                    </span>
                    <span v-else style="color: red;">
                        未上架
                    </span>
                </template>
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


        <!--新增界面-->
        <el-dialog title="新增课程" :visible.sync="addCourseVisible" :close-on-click-modal="false"  >
            <el-form :model="addCourse" label-width="80px" :rules="addCourseRules" ref="addCourse">
                <el-form-item label="课程名称" prop="name" >
                    <el-input v-model="addCourse.name" auto-complete="off" style="width: 60%"></el-input>
                </el-form-item>
                <el-form-item label="课程类型" prop="courseType.name" >
                    <el-select v-model="addCourse.courseTypeId" clearable placeholder="请选择课程类型" style="width: 60%">
                        <el-option
                                v-for="item in courseTypes"
                                :key="item.id"
                                :label="item.name"
                                :value="item.id">
                        </el-option>
                    </el-select>
                </el-form-item>
                <el-form-item label="适用人群" prop="users">
                    <el-input v-model="addCourse.users" auto-complete="off" style="width: 60%"></el-input>
                </el-form-item>
                <el-form-item label="课程等级" prop="grade">
                    <el-radio-group v-model="addCourse.grade" >
                        <el-radio v-for="courseLevel in courseLevels"
                                  :label="courseLevel.id">{{courseLevel.name}}</el-radio>
                    </el-radio-group>
                </el-form-item>
                <el-form-item label="简介" prop="intro">
                    <!--<el-input v-model="addCourse.detail.intro" auto-complete="off"></el-input>-->
                    <quill-editor
                            v-model="addCourse.detail.intro"
                            :content="content"
                            :options="editorOption"
                            @blur="onEditorBlur($event)"
                            @focus="onEditorFocus($event)"
                            @ready="onEditorReady($event)">
                    </quill-editor>
                </el-form-item>
                <el-form-item label="详情" prop="description">
                    <el-input v-model="addCourse.detail.description" auto-complete="off"></el-input>
                </el-form-item>

            </el-form>
            <div slot="footer" class="dialog-footer">
                <el-button @click.native="addCourseVisible = false">取消</el-button>
                <el-button type="primary" @click.native="addSubmit" :loading="addLoading">提交</el-button>
            </div>
        </el-dialog>

        <!--市场信息-->
        <!---------------------------------------->
        <!--图片信息-->
        <el-dialog title="添加图片信息" :visible.sync="addPicVisible" :close-on-click-modal="false"  style="width: 80%">
            <el-form :model="addPic" label-width="80px" ref="addPic">

                <el-form-item label="课程图片">
                    <el-upload
                            class="upload-demo"
                            action="http://localhost:9527/services/file/file/upload"
                            :on-success="handleSuccess"
                            :on-remove="handleRemove"
                            limit="10"
                            :file-list="fileList"
                            list-type="picture">
                        <el-button size="small" type="primary">点击上传</el-button>
                        <div slot="tip" class="el-upload__tip">只能上传jpg/png文件，且不超过500kb,不超过10张</div>
                    </el-upload>
                </el-form-item>

            </el-form>
            <div slot="footer" class="dialog-footer">
                <el-button @click.native="addPicVisible = false">取消</el-button>
                <el-button type="primary" @click.native="addPicSubmit" >提交</el-button>
            </div>
        </el-dialog>




    </section>
</template>

<script>
    export default {

        name: "course",
        data(){
            return {
                content: '',
                editorOption: {
                    modules:{
                        toolbar:[
                            ['bold', 'italic', 'underline', 'strike'],        // toggled buttons
                            ['blockquote', 'code-block']
                        ]
                    }
                },
                options:[
                    {
                        value:0,
                        label:"未上架"
                    },
                    {
                        value:1,
                        label:"上架"
                    }
                ],
                filters:{
                    keyword:"",
                    status:null
                },
                courses:[],
                sels:[],//多选框选中的行
                pageNum:1,
                pageSize:10,
                total:0,
                addCourseVisible:false,
                addPicVisible:false,
                courseTypes:[],
                addLoading: false,
                addCourseRules:{},
                courseLevels:[],
                addCourse:{
                    id:null,
                    name:'',
                    users:'',
                    courseTypeId:null,
                    grade:null,
                    detail:{
                        intro:'',
                        description:''
                    }
                },
                addPic:{
                    resources:""
                },
                fileList:[],

            }
        },
        methods:{
            onEditorBlur(editor) {
                //console.log('editor blur!', editor)
            },
            onEditorFocus(editor) {
                //console.log('editor focus!', editor)
                console.log(editor.scrollingContainer.firstElementChild.innerHTML);
            },
            onEditorReady(editor) {
                //console.log('editor ready!', editor)
            },
            formatStartTime(row, column){
                return this.formatTime(row.startTime);
            },
            formatEndTime(row, column){
                return this.formatTime(row.endTime);
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
            getCourses(){
                let param = {};
                param.page = this.pageNum;
                param.rows = this.pageSize;
                param.keyword = this.filters.keyword;
                param.status = this.filters.status;
                this.$http.post("/course/course/page",param)
                    .then(res=>{
                        let {total,rows} = res.data;
                        this.total = total;
                        this.courses = rows;
                    })
            },
            selectChange(selection){
                this.sels = selection;
            },
            handleAdd(){
                this.addCourseVisible = true;


            },
            getcourseTypes(){
                this.$http.get("/course/courseType/list").then(res=>{
                    this.courseTypes=res.data;
                })
            },
            getCourseLevels(){
                //发送请求到后台获取数据
                this.$http.get("/system/systemdictionaryitem/listSn?sn=dj")
                    .then(result=>{
                        this.courseLevels = result.data;
                    });

            },
            handleMarket(){},
            handlePic(){
                this.addPicVisible = true;
                this.addPic = [];
            },
            handleOnline(){
                //上架
                let param = this.sels.map(e => e.id);//es6语法
                this.$http.post("/course/course/online",param) //$.Post(.....)
                    .then(result=>{
                        let {success,message} = result.data;
                        if (success){
                            this.$message({
                                message: '操作成功!',
                                type: 'success'
                            });
                            //刷新数据
                            this.getCourses();
                        }else{
                            this.$message({
                                message: '操作失败!',
                                type: 'error'
                            });
                        }

                    });
            },
            handleOffline(){
                //下架
                let param = this.sels.map(e => e.id);//es6语法
                this.$http.post("/course/course/offline",param) //$.Post(.....)
                    .then(result=>{
                        let {success,message} = result.data;
                        if (success){
                            this.$message({
                                message: '操作成功!',
                                type: 'success'
                            });
                            //刷新数据
                            this.getCourses();
                        }else{
                            this.$message({
                                message: '操作失败!',
                                type: 'error'
                            });
                        }

                    });
            },
            handleSuccess(response, file, fileList) {
                console.log(response);
                let{success,message,resultObj} = response;
                //如果成功
                if(success){
                    this.addPic.resources = resultObj;
                }else {
                    this.$message({
                        message: message,
                        type: 'error'
                    });
                }
            },
            handleRemove(file, fileList) {
                console.log(file);
                //获取到file_id
                let fileId = file.response.resultObj;
                this.$http.get("/file/file/delete?file_id="+fileId)
                    .then(res=>{
                        let{success,message} = res.data;
                        if(success){
                            this.$message({
                                message: message,
                                type: 'success'
                            });
                        }else {
                            this.$message({
                                message: message,
                                type: 'error'
                            });
                            return false;
                        }
                    });
            },
            addPicSubmit(){
                let param = this.sels.map(e => e.id);//es6语法
                console.log(param);
                param.courseId = this.addPic.courseId;
                param.resources = this.addPic.resources;
                this.$confirm('确认提交吗？', '提示', {}).then(() => {
                    this.$http.post("/course/courseResources/save",param)
                        .then(res=>{
                            this.registerLoading = false;
                            let {success,message} = res.data;
                            if (success){
                                this.$message({
                                    message:message ,
                                    type: 'success'
                                });
                                this.addPicVisible = true;
                            }else{
                                this.$message({
                                    message: message,
                                    type: 'error'
                                });
                            }
                        })
                });

            },

            handleEdit(index,row){
                
            },
            handleDel(index,row){
                
            },
            handleCurrentChange(val){
                this.pageNum = val;
                this.getCourses();
            },
            addSubmit(){
                let param={};
                param.name = this.addCourse.name;
                param.courseTypeId = this.addCourse.courseTypeId;
                param.grade = this.addCourse.grade;
                this.$confirm('确认提交吗？', '提示', {}).then(() => {
                    this.$http.post("/course/course/save",param)
                        .then(res=>{
                            let {success,message} = res.data;
                            if (success){
                                this.$message({
                                    message:message ,
                                    type: 'success'
                                });
                                //清空form
                                this.addCourse={
                                    name:"",
                                    courseTypeId:"",
                                    grade:""
                                }
                            }else{
                                this.$message({
                                    message: message,
                                    type: 'error'
                                });
                            }
                        })
                });
                this.addCourseVisible = false;
                this.getCourses();


            }
        },
        computed: {
            editor() {
                return this.$refs.myTextEditor.quillEditor
            }
        },
        mounted(){
            this.getCourses();
            this.getcourseTypes();
            this.getCourseLevels();
        }
    }
</script>

<style scoped>

</style>