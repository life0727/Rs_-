<template>
<div class="container" v-loading.fullscreen.lock="publicLoading" element-loading-text="系统拼命加载中" element-loading-spinner="el-icon-loading">
<p style="line-height: 50px;margin: 10px 0 0 0;position: relative">客户列表
  <el-button type="warning" style="padding: 4px 29px;position: absolute;right: -15px;top:15px;background-color: #fcd482;border-color:#fcd482;"  @click="add_btn">添加</el-button>
</p>
  <div class="custom container" style="background:#ffffff;padding:15px 10px;">
       <el-table
        :data="tabledata_user"
        border
        style="width: 100%">
        <el-table-column
          label="用户名"
          show-overflow-tooltip
          prop="username">
        </el-table-column>
        <el-table-column
          label="账号"
          width=100
          show-overflow-tooltip
          prop="account">
        </el-table-column>
        <el-table-column
          label="电话"
          width=100
          show-overflow-tooltip
          prop="tel">
        </el-table-column>
        <el-table-column
          label="QQ号"
          width=80
          show-overflow-tooltip
          prop="qq">
        </el-table-column>
        <el-table-column
          label="邮件"
          show-overflow-tooltip
          prop="email">
        </el-table-column>
        <el-table-column
          label="公司名称"
          show-overflow-tooltip
          prop="companyName">
        </el-table-column>
        <el-table-column
          label="公司地址"
          width=200
          show-overflow-tooltip
          prop="companyAddress">
        </el-table-column>
        <el-table-column
          label="备注"
          show-overflow-tooltip>
          <template scope="scope">{{ scope.row.remark}}</template>
        </el-table-column>
        <el-table-column label="操作">
        <template scope="scope">
        <el-button type="text" @click="fenpei(scope.row)" style="color: #32a880">项目分配</el-button>
          <!-- <el-button
            size="small"
            @click="fenpei(scope.row)">项目分配</el-button>-->
         <el-button type="text" @click="write(scope.row)" style="color: #32a880">编辑</el-button>   
            <!-- <el-button 
            size="small"
            type="info"
            @click="write( scope.row)">编辑</el-button> -->
          <el-button type="text" @click="del(scope.row)" style="color:#f56c6c;">删除</el-button>  
          <!-- <el-button
            size="small"
            type="danger"
            @click="del(scope.$index, scope.row)">删除</el-button> -->
        </template>
        </el-table-column>
        
      </el-table>
    <!-- <el-checkbox style="margin: 10px;" v-model="select_All"   v-show="this.tabledata_user.length!==0" >全选所有</el-checkbox>
    <el-button type="text" @click="del('all')" style="color:#f56c6c;margin-left: 5px;" v-show="select_All&&this.tabledata_user.length!==0" >确认删除</el-button> -->
  </div>
  <!-- 添加用户模态框 -->
    <el-dialog :title="title" :visible.sync="dialog_user" size="tiny" id="dialog_add" custom-class="ev_dialogClass">
      <el-form label-position="right" label-width="120px" >
        <el-form-item label="用户名">
          <el-input v-model="name" placeholder="请输入用户名" ></el-input>
        </el-form-item>
        <el-form-item label="账号">
          <el-input v-model="account" placeholder="请输入账号" :disabled="title=='客户修改'"></el-input>
        </el-form-item>
        <el-form-item label="密码">
          <el-input :type="password_type" v-model="password" placeholder="请输入密码" style="position: relative"></el-input><i @click="look_password" :class="password_type=='text' ? 'fa fa-eye-slash' : 'fa fa-eye' " style="position: absolute;right: 75px;line-height: 36px;"></i>
        </el-form-item>
        <el-form-item label="手机号">
          <el-input  v-model="phone" placeholder="请输入手机号"></el-input>
        </el-form-item>
        <el-form-item label="QQ号">
          <el-input  v-model="qq" placeholder="请输入QQ号"></el-input>
        </el-form-item>
        <el-form-item label="Email">
          <el-input  v-model="email" placeholder="请输入Email"></el-input>
        </el-form-item>
        <el-form-item label="公司名称">
          <el-input  v-model="companyName" placeholder="请输入公司名称"></el-input>
        </el-form-item>
        <el-form-item label="公司地址">
          <el-input  v-model="companyAddress" placeholder="请输入公司地址"></el-input>
        </el-form-item>
        <el-form-item label="备注">
          <el-input type="textarea" v-model="remark"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialog_user = false">取 消</el-button>
        <el-button type="primary" @click="add(write_obj)">确 定</el-button>
      </span>
  </el-dialog>
  <!-- 查看用户项目模态框 -->
    <el-dialog title="项目列表" :visible.sync="dialog_project" width="400px !important" id="dialog_project" custom-class="ev_dialogClass">
    <p style="color:#FF5015;">*请选择分配给当前客户的项目 </p>
       <el-table
        :data="projectList"
        border
        ref="table"
        @selection-change="select_pro"
        style="width: 100%">
        <el-table-column
          type="selection"
          width="55">
        </el-table-column>
        <el-table-column
          label="项目名称"
          show-overflow-tooltip>
          <template scope="scope">{{ scope.row.name }}</template>
        </el-table-column>
      </el-table>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialog_project = false">取 消</el-button>
        <el-button type="primary" @click="add_">确 定</el-button>
      </span>
  </el-dialog>
</div>  
</template>

<script>

import { Sort,Map,date_change,SetSessionStorage,publicSearch,successBack,tipsMessage,similar,jsonToStrMap }  from '../../assets/js/map.js'
export default {
  mounted :function () {
    this.search();
  },
  data () {
  	return {
      tabledata_user:[],//用户总数据
      projectList:[],//用户项目列表数据
      dialog_user:false, //模态框显示隐藏控制
      dialog_project:false, //模态框显示隐藏控制
      write_obj:'', //确定添加用户时的数据
      password_type:'password', //输入框的类型
      name:'', //用户信息
      account:'', //用户信息
      password:'', //用户信息
      phone:'', //用户信息
      qq:'', //用户信息
      email:'', //用户信息
      companyName:'', //用户信息
      companyAddress:'', //用户信息
      remark:'', //用户信息
      selection:[], //单选框
      userbId:'', //分配用户的id
      title:'', //客户修改和添加用户的flag
      publicLoading:false, //公共loading
      select_All:false//全选状态绑定
  	}
  },
  methods: {
    search(){ //查询用户的数据总方法
      let useraID = "123456";//userAId 这个是当前A系统登录的的id,前台取不到的话,可以随意给一个.后台会处理
      let dta = {
        'method':'get'
      };
      let params = {"customerDto":JSON.stringify(dta)};
      this.publicLoading = true;
      publicSearch('rsa/userA/'+useraID+'/customer',"GET",params).then((data) =>{//ajax
        this.publicLoading = false;
        if(data.code == 1003){//超时用户重新认证
           this.$prompt('请输此账户的密码：', '提示', {
                        confirmButtonText: '确定',
                        cancelButtonText: '取消',
                        inputType:'password',
                        customClass:'ev_confirmClass',
                        closeOnClickModal:false,
                      }).then(({ value }) => {
                        publicSearch('rsa/userA/author',"POST",{"password":value}).then((data) =>{//ajax
                          if(successBack(data,this)){
                            this.search();
                          };
                        });  
                      }).catch(() => {      
                      });
        }else{//不需要验证
          if(successBack(data,this)){
            console.log(data)
            this.tabledata_user = data.data;
          };
        } 
      })
    },
    write(a){ //客户修改
      this.write_obj=a;
      this.title='客户修改';
      this.dialog_user = true;
      this.password_type='password';
      this.name=a.username;
      this.account=a.account;
      this.password=a.password;
      this.phone=a.tel;
      this.qq=a.qq;
      this.email=a.email;
      this.companyName=a.companyName;
      this.companyAddress=a.companyAddress;
      this.remark=a.remark;
      console.log(a)
    },
    add_btn(){ //添加客户前操作
      this.dialog_user = true;
      this.title='添加客户';
      this.password_type='password';
      this.name='';
      this.account='';
      this.password='';
      this.phone='';
      this.qq='';
      this.email='';
      this.companyName='';
      this.companyAddress='';
      this.remark='';
    },
    add(a){//添加客户方法
      console.log(a)
      let useraId = "123456";
      if(this.name===''||this.account===''||this.password===''){
        tipsMessage('用户名密码不能为空','warning',this)
      }else{
      let userData = {
                      username: this.name,
                      account: this.account,
                      password: this.password,
                      tel: this.phone,
                      qq: this.qq,
                      email: this.email,
                      companyName: this.companyName,
                      companyAddress: this.companyAddress,
                      remark: this.remark,
                      id:a.id
                    }
      this.publicLoading = true;             
      if(this.title==='添加客户'){//添加客户的确定
        publicSearch('rsa/userA/'+useraId+'/customer',"post",{"customerDto":JSON.stringify({method : "post",userB : userData})}).then((data) =>{//ajax 
          this.publicLoading = false;
          if(data.code == 1003){//超时用户重新认证
             this.$prompt('请输此账户的密码：', '提示', {
                          confirmButtonText: '确定',
                          cancelButtonText: '取消',
                          inputType:'password',
                          customClass:'ev_confirmClass',
                          closeOnClickModal:false,
                        }).then(({ value }) => {
                          publicSearch('rsa/userA/author',"POST",{"password":value}).then((data) =>{//ajax
                            if(successBack(data,this)){
                              this.dialog_user = false;
                              this.search();
                            };
                          });  
                        }).catch(() => {      
                        });
          }else{//不需要验证
            if(successBack(data,this)){
              this.dialog_user = false;
              this.search();
            } 
          } 
        })  
      }else{//修改客户的确定
        console.log(this.name)
        if(a.username!=this.name||a.password!=this.password||a.tel!=this.phone||a.qq!=this.qq||a.email!=this.email||a.companyName!=this.companyName||a.companyAddress!=this.companyAddress||a.remark!=this.remark){//判断是否修改
          publicSearch('rsa/userA/'+useraId+'/customer',"POST",{"customerDto":JSON.stringify({method : "put",userB : userData})}).then((data) =>{//ajax 
            this.publicLoading = false;
            if(data.code == 1003){//超时用户重新认证
               this.$prompt('请输此账户的密码：', '提示', {
                            confirmButtonText: '确定',
                            cancelButtonText: '取消',
                            inputType:'password',
                            customClass:'ev_confirmClass',
                            closeOnClickModal:false,
                          }).then(({ value }) => {
                            publicSearch('rsa/userA/author',"POST",{"password":value}).then((data) =>{//ajax
                              if(successBack(data,this)){
                                this.dialog_user = false;
                                this.search();
                              };
                            });  
                          }).catch(() => {      
                          });
            }else{//不需要验证
              if(successBack(data,this)){
                this.dialog_user = false;
                this.search();
              } 
            } 
          })  
          }else{//没修改
            this.dialog_user = false;
            tipsMessage('没修改','warning',this);
          }
        }
      }
    },
    fenpei(row){ //项目分配的相关数据（toggleRowSelection方法是手动修改已经分给当前客户的项目的多选框的dom）
      console.log(row)
      this.userbId = row.id;
      let idarr = [];
      let dta = {
        'method':'get'
      };
      let params = {"customerProDto":JSON.stringify(dta)};
      this.dialog_project = true;
      this.publicLoading = true;
      publicSearch('rsa/customer/'+row.id+'/project',"get",params).then((data) =>{//ajax 
        this.publicLoading = false;
         if(data.code == 1003){//超时用户重新认证
               this.$prompt('请输此账户的密码：', '提示', {
                            confirmButtonText: '确定',
                            cancecustomerDtoButtonText: '取消',
                            inputType:'password',
                            customClass:'ev_confirmClass',
                            closeOnClickModal:false,
                          }).then(({ value }) => {
                            publicSearch('rsa/userA/author',"POST",{"password":value}).then((data) =>{//ajax
                              if(successBack(data,this)){
                                this.fenpei(row);
                              };
                            });  
                          }).catch(() => {      
                          });
            }else{//不需要验证
              if(successBack(data,this)){
                let arr = [];
                if(data.data.optionalProjectList!=null && data.data.optionalProjectList.length > 0){
                    arr=arr.concat(data.data.optionalProjectList);
                };
                if(data.data.selectedProjectList!=null && data.data.selectedProjectList.length > 0){
                    arr=arr.concat(data.data.selectedProjectList);
                };
                for(let i=0;i<arr.length;i++){
                  if(arr[i]===undefined||arr[i]===null){
                    arr.splice(i, 1);
                  }
                };
                 this.$nextTick(function(){
                  for(let i=0;i<arr.length;i++){
                    idarr.push(arr[i].id);
                  };
                  if(data.data.selectedProjectList){
                    for(let t=0;t<data.data.selectedProjectList.length;t++){
                      if(idarr.indexOf(data.data.selectedProjectList[t].id)!=-1){
                        this.$refs.table.toggleRowSelection(data.data.selectedProjectList[t],true);
                      }
                    }
                  }
                });
                this.projectList=arr;
              }  
            } 
      })  
    },
    select_pro(val){
      this.selection=val;
    },
    add_(){ //确定分配后的操作
      //console.log(this.selection)
      let idarr = [];
      for(let i of this.selection){
        idarr.push(i.id);
      };
      let dta = {
          method : "post",
          proIdList:idarr
      };
      this.publicLoading = true;
      publicSearch('rsa/customer/'+this.userbId+'/project',"get",{"customerProDto":JSON.stringify(dta)}).then((data) =>{//ajax 
        this.publicLoading = false;
        console.log(data);
        if(data.code == 1003){//超时用户重新认证
           this.$prompt('请输此账户的密码：', '提示', {
                        confirmButtonText: '确定',
                        cancecustomerDtoButtonText: '取消',
                        inputType:'password',
                        customClass:'ev_confirmClass',
                        closeOnClickModal:false,
                      }).then(({ value }) => {
                        publicSearch('rsa/userA/author',"POST",{"password":value}).then((data) =>{//ajax
                          if(successBack(data,this)){
                            this.add_();
                          };
                        });  
                      }).catch(() => {      
                      });
        }else{
          if(successBack(data,this)){
            tipsMessage(data.message,'success',this);
            this.dialog_project = false;
          };
        }
      })  
    },
    del(Dta){// 删除项目
      let userBIds = [],useraId = "123456";
      if(Dta == 'all'){
        for(let i of this.tabledata_user){
          userBIds.push(i.id);
        };
      }else{
        userBIds.push(Dta.id);
      };         
      let dta = {
                method : "delete",
                userBIds
            };
      console.log(Dta);
      this.$confirm(Dta == 'all' ? '是否删除全部' : '是否删除', '提示', {
                  confirmButtonText: '确定',
                  cancelButtonText: '取消',
                  customClass:'ev_confirmClass',
                  type: 'warning'
                }).then(() => {
                  publicSearch('rsa/userA/'+useraId+'/customer',"POST",{"customerDto":JSON.stringify(dta)}).then((data) =>{//ajax 
                      if(data.code == 1003){//超时用户重新认证
                         this.$prompt('请输此账户的密码：', '提示', {
                                      confirmButtonText: '确定',
                                      cancecustomerDtoButtonText: '取消',
                                      inputType:'password',
                                      customClass:'ev_confirmClass',
                                      closeOnClickModal:false,
                                    }).then(({ value }) => {
                                      publicSearch('rsa/userA/author',"POST",{"password":value}).then((data) =>{//ajax
                                        if(successBack(data,this)){
                                          this.search();
                                        };
                                      });  
                                    }).catch(() => {      
                                    });
                      }else{//不需要验证
                        if(successBack(data,this)){
                          this.search();
                        } 
                      } 
                  }); 
                }).catch(() => { tipsMessage('取消删除','warning',this) });
    },
    look_password(){ //切换密码框的可见不可见
      if(this.password_type==='text'){
        this.password_type='password'
      }else{
        this.password_type='text'
      }    
    }
  }/*,
  watch:{
      $route (to,from){
        console.log(from)
        if(from.path==='/main/event'){
          console.log(4555)
        }
      }
    }*/
}
</script>
<style lang="scss" >
 #dialog_add{
      .el-dialog__body{
        padding-left: 5px;
        padding-bottom: 10px;
        .el-form-item__content{
          margin: 0 !important;
          .el-input,.el-textarea{
            width: 400px !important;
          }
        }
      }
    }
#dialog_project{
  .el-dialog__body{
    padding-top: 10px;
    .el-table td, .el-table th{
      padding: 4px 0;
      text-align: center;
    }
  }
}    
  .custom{
  /*    表格头部 */
     thead{
        th{
          height:25px;
        }
        th>.cell{
          padding: 0;
          text-align: center;
          .el-checkbox__inner{
            width:14px;
            height:14px;     
          }
          label{
            margin:0;
          }
          .caret-wrapper{
          margin: 0;
          }
        }
     }
   /*    表格内部 */ 
   tbody{
    /* .current-row{
      color: #ccc !important;
      background-color: #ccc !important;
    } */
    td{
      padding: 7px 0;
      .cell{
        .el-button{
          padding: 3px 0px;
        }
        text-align: center;
        padding: 0;
        .el-button--small {
          padding: 4px 3px;
          margin-left: 3px;
        }
      }
    }
    
    tr{
      height:25px;
    }
    .el-table-column--selection>.cell{
      padding: 0;
      text-align: center;
      .el-checkbox{
        margin:0;
        .el-checkbox__inner{
          width:14px;
          height:14px;
        }
      }
     }
    }
    .el-checkbox__input.is-checked+.el-checkbox__label{
      color: #e6a23c;
    }
    .el-checkbox__input.is-checked .el-checkbox__inner, .el-checkbox__input.is-indeterminate .el-checkbox__inner{
      background-color: #e6a23c;
      border-color: #e6a23c;
    }
  }

</style>
