<template>
  <section>
  <el-form :model="tenant" :rules="tenantRule" ref="tenantForm" label-position="left" label-width="80px" class="demo-ruleForm login-container">
    <h3 class="title">租户注册</h3>
    <el-row>
      <el-col :span="11">
        <el-form-item prop="companyName" label="公司名称">
          <el-input type="text" v-model="tenant.companyName" auto-complete="off" placeholder="请输入公司名称"></el-input>
        </el-form-item>
      </el-col>
      <el-col :span="11" :offset="1">
        <el-form-item label="机构类型">
          <el-select clearable style="width: 100%" v-model="tenant.tenantType" placeholder="请选择">
            <el-option
                    v-for="item in tenantTypes"
                    :key="item.id"
                    :label="item.name"
                    :value="item.id">
              <span style="float: left">{{ item.name }}</span>
              <span style="float: right; color: #8492a6; font-size: 13px">{{ item.description }}</span>
            </el-option>
          </el-select>
        </el-form-item>
      </el-col>
    </el-row>
    <el-row>
      <el-col :span="11">
        <el-form-item label="座机电话">
          <el-input type="text" v-model="tenant.companyNum" auto-complete="off" placeholder="请输入座机电话"></el-input>
        </el-form-item>
      </el-col>
      <el-col :span="11" :offset="1">
        <el-form-item prop="username" label="账号">
          <el-input type="text" v-model="tenant.username" auto-complete="off" placeholder="请输入用户名"></el-input>
        </el-form-item>
      </el-col>
    </el-row>
    <el-row>
      <el-col :span="11">
        <el-form-item label="公司地址">
          <el-input @focus="showBaiduDialog" type="text" v-model="tenant.address" auto-complete="off" placeholder=""></el-input>
        </el-form-item>
      </el-col>
      <el-col :span="11" :offset="1">
        <el-form-item prop="password" label="密码">
          <el-input type="password" v-model="tenant.password" auto-complete="off" placeholder="请输入密码"></el-input>
        </el-form-item>
      </el-col>
    </el-row>
    <el-row>
      <el-col :span="11">

        <el-form-item prop="setMealId" label="套餐选择">
          <!--<el-input type="text" v-model="tenant.setmeal_id" auto-complete="off"></el-input>-->
          <el-select style="width:100%" v-model="tenant.setmeal_id" clearable placeholder="请选择套餐">
            <el-option
                    v-for="item in setmeals"
                    :key="item.id"
                    :label="item.mealName"
                    :value="item.id">
              <span style="float: left">{{ item.mealName }}</span>
              <span style="float: right; color: #8492a6; font-size: 13px">￥{{ item.mealPrice }}</span>
            </el-option>
          </el-select>

        </el-form-item>
      </el-col>
      <el-col :span="11" :offset="1">
        <el-form-item prop="repassword" label="重复密码">
          <el-input type="password" v-model="tenant.repassword" auto-complete="off" placeholder="请再次输入密码"></el-input>
        </el-form-item>
      </el-col>
    </el-row>
    <el-row>
      <el-col :span="11">
        <el-form-item label="公司logo">
          <!--<el-input type="text" v-model="tenant.logo" auto-complete="off" placeholder=""></el-input>-->
          <el-upload
                  class="upload-demo"
                  action="http://localhost:9527/services/file/file/upload"
                  :on-success="handleSuccess"
                  :on-remove="handleRemove"
                  limit="10"
                  :file-list="fileList"
                  list-type="picture">
            <el-button size="small" type="primary">点击上传</el-button>
            <div slot="tip" class="el-upload__tip">只能上传jpg/png文件，且不超过500kb</div>
          </el-upload>
        </el-form-item>

      </el-col>
    </el-row>
    <el-row>
      <el-col :span="23" style="text-align: right">
        <el-button size="small" type="primary" @click="subRegister" :loading="registerLoading">保存</el-button>
        <el-button size="small" @click="toLogin">有账号，直接登录</el-button>
      </el-col>
    </el-row>
  </el-form>


    <el-dialog
            title="地址选择"
            :visible.sync="dialogVisible"
            width="60%">
      <baidu-map :center="{lng: 116.403765, lat: 39.914850}" :zoom="15">
        <bm-auto-complete v-model="keyword" :sugStyle="{zIndex: 2100}">
          <div style="margin-bottom:10px">
            <input type="text" placeholder="请输入关键字" class="searchinput"/>
            <el-button type="success" @click="confireAddress">确定</el-button>
          </div>
        </bm-auto-complete>
        <bm-view class="bmap"/>
        <bm-local-search :keyword="keyword" :auto-viewport="true" :panel="false"></bm-local-search>
      </baidu-map>

    </el-dialog>

  </section>
</template>

<script>
  import SelectTree from '@/components/SelectTree.vue'
  export default {
      components:{
          SelectTree
      },
    data() {
      var validaterePassword = (rule, value, callback) => {
          if (value !== this.tenant.password) {
              callback(new Error('两次输入密码不一致!'))
          } else {
              callback()
          }
      }
      return {
          keyword:"",
          dialogVisible:false,
        //上传图片
        fileList: [{name: 'food.jpeg', url: 'https://fuss10.elemecdn.com/3/63/4e7f3a15429bfda99bce42a18cdd1jpeg.jpeg?imageMogr2/thumbnail/360x360/format/webp/quality/100'}, {name: 'food2.jpeg', url: 'https://fuss10.elemecdn.com/3/63/4e7f3a15429bfda99bce42a18cdd1jpeg.jpeg?imageMogr2/thumbnail/360x360/format/webp/quality/100'}],

          props:{
              // 配置项（必选）
              value: "id",
              label: "name",
              children: "children"
          },
          setmeals:[],
          tenantTypes:[],
          registerLoading:false,
          tenant:{
              "address": "",
              "companyName": "",
              "companyNum": "",
              "tenantType": "",
              "coordinate": "",
              "email": "",
              "logo": "",
              "password": "",
              "setmeal_id": null,
              "username": "",
              "repassword":""
          },
          tenantRule:{
              companyName:[
                  {required:true,message:"请输入公司名称",trigger:'blur'},
                  {max:10,min:5,message:"名称长度在5~10个字符之间",trigger:'blur'}
              ],
              companyTel:[
                  {required:true,message:"请输入手机号码",trigger:'blur'},
                  {pattern:/^1[3-9]{1}[0-9]{9}$/,required:true,message:"手机号码格式不正确",trigger:'blur'}
              ],
              email:[
                  {required:true,message:"请输入邮箱",trigger:'blur'},
                  {type:"email",message:"邮箱格式不正确",trigger:'blur'}
              ],
              username:[
                  {required:true,message:"请输入用户名",trigger:'blur'},
                  {pattern:/^[A-Za-z]{1}[A-Za-z0-9_]*$/,message:"必须是以字母开头，包含字母数字下划线",trigger:'blur'},
                  {max:16,min:4,message:"用户名长度在4到16位之间",trigger:'blur'}
              ],
              /*password:[
                  {required:true,message:"请输入密码",trigger:'blur'},
                  {pattern:/(?=^.{6,16}$)(?=.*\d)(?=.*[A-Z])(?=.*[a-z])(?=.*[!@#$%^&*,\\.]).*$/,message:"6-16(包含数字大写字母小写字母和特殊字符)",trigger:'blur'}
              ],
              repassword:[
                  {required:true,message:"请再次输入密码",trigger:'blur'},
                  {validator: validaterePassword,trigger:'blur'}
              ],*/
              setmeal_id:[
                  {required:true,message:"请选择套餐",trigger:'blur'}
              ]
          }
      }
    },
    methods: {
        confireAddress(){
          this.tenant.address = this.keyword;
          this.keyword = "";
          this.dialogVisible = false;
        },
        showBaiduDialog(){
            this.dialogVisible = true;
        },
        //加载所有套餐
        getSetmeals(){
          this.$http.get("/system/meal/list")
              .then(res=>{
                  this.setmeals = res.data;
          })
        },
        //加载所有的机构类型
        getTenantTypes(){
            this.$http.get("/system/tenantType/list")
                .then(res=>{
                    this.tenantTypes = res.data;
                })
        },
        toLogin(){
            this.$router.push({ path: '/login' });
        },
        subRegister(){
            this.$refs.tenantForm.validate((valid) => {
                if (valid) {

                    //封装请求参数
                    let param = {};

                    let tenant = {};
                    tenant.tenantType = this.tenant.tenantType;
                    tenant.companyName = this.tenant.companyName;
                    tenant.companyNum = this.tenant.companyNum;
                    tenant.address = this.tenant.address;
                    tenant.logo = this.tenant.logo;
                    param.tenant = tenant;

                    param.username = this.tenant.username;
                    param.password = this.tenant.password;
                    param.meal = this.tenant.setmeal_id;

                    this.$confirm('确认提交吗？', '提示', {}).then(() => {
                        this.registerLoading = true;
                        this.$http.post("/system/tenant/register",param)
                            .then(res=>{
                                this.registerLoading = false;
                                let {success,message} = res.data;
                                if (success){
                                    this.$message({
                                        message:message ,
                                        type: 'success'
                                    });
                                    //清空form
                                    this.tenant = {
                                        "address": "",
                                        "companyName": "",
                                        "companyNum": "",
                                        "tenantType": "",
                                        "coordinate": "",
                                        "email": "",
                                        "logo": "",
                                        "password": "",
                                        "setmeal_id": null,
                                        "username": "",
                                        "repassword":""
                                    }
                                }else{
                                    this.$message({
                                        message: message,
                                        type: 'error'
                                    });
                                }
                            })
                    });
                }
            });
        },
      //上传成功的回调
      handleSuccess(response, file, fileList) {
          console.log(response);
          let{success,message,resultObj} = response;
          //如果成功
          if(success){
            this.tenant.logo = resultObj;
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
      }
    },
      mounted(){
        this.getSetmeals();
        this.getTenantTypes();
      }
  }

</script>

<style lang="scss" scoped>
  .login-container {
    /*box-shadow: 0 0px 8px 0 rgba(0, 0, 0, 0.06), 0 1px 0px 0 rgba(0, 0, 0, 0.02);*/
    -webkit-border-radius: 5px;
    border-radius: 5px;
    -moz-border-radius: 5px;
    background-clip: padding-box;
    margin: 80px auto;
    width: 700px;
    padding: 35px 35px 15px 35px;
    background: #fff;
    border: 1px solid #eaeaea;
    box-shadow: 0 0 25px #cac6c6;
    .title {
      margin: 0px auto 40px auto;
      text-align: center;
      color: #505458;
    }
    .remember {
      margin: 0px 0px 35px 0px;
    }
  }



  .searchinput{
    width: 300px;
    box-sizing: border-box;
    padding: 9px;
    border: 1px solid #dddee1;
    line-height: 20px;
    font-size: 16px;
    height: 38px;
    color: #333;
    position: relative;
    border-radius: 4px;
  }

  .bmap{
    width: 100%;
    height: 300px;
  }
</style>