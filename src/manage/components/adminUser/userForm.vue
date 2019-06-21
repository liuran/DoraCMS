<template>
    <div class="dr-adminUserForm">
        <el-dialog width="75%" size="small" title="填写用户信息" :visible.sync="dialogState.show" :close-on-click-modal="false">
            <el-form :model="dialogState.formData" :rules="rules" ref="ruleForm" label-width="120px" class="demo-ruleForm">

                <el-row :gutter="20">
                    <el-col :span="12">
                        <el-form-item label="用户名" prop="userName">
                            <el-input size="small" v-model="dialogState.formData.userName"></el-input>
                        </el-form-item>
                        <el-form-item label="姓名" prop="name">
                            <el-input size="small" v-model="dialogState.formData.name"></el-input>
                        </el-form-item>

                        <el-form-item label="修改密码" v-if="dialogState.formData.password != ''">
                            <el-switch
                                    v-model="bChangePass"
                                    active-color="#13ce66"
                                    inactive-color="#ff4949">
                            </el-switch>
                        </el-form-item>


                        <el-form-item label="新密码" prop="password" v-if="bShowPassEdit">
                            <el-input size="small" type="password" v-model="dialogState.formData.password"></el-input>
                        </el-form-item>
                        <el-form-item label="确认密码" prop="confirmPassword" v-if="bShowPassEdit">
                            <el-input size="small" type="password" v-model="dialogState.formData.confirmPassword"></el-input>
                        </el-form-item>

                        <el-form-item label="用户组" prop="group">
                            <el-select size="small" v-model="dialogState.formData.group" placeholder="请选择用户组">
                                <el-option :key="index" v-for="(group,index) in groups" :label="group.name" :value="group._id"></el-option>
                            </el-select>
                        </el-form-item>
                        <el-form-item label="电话" prop="phoneNum">
                            <el-input size="small" v-model="dialogState.formData.phoneNum"></el-input>
                        </el-form-item>
                        <el-form-item label="邮箱" prop="email">
                            <el-input size="small" v-model="dialogState.formData.email"></el-input>
                        </el-form-item>

                    </el-col>
                    <el-col :span="12">
                        <el-form-item label="头像" prop="logo">
                            <el-upload
                                    class="avatar-uploader"
                                    action="/system/upload?type=images"
                                    :show-file-list="false"
                                    :on-success="handleAvatarSuccess"
                                    :before-upload="beforeAvatarUpload">
                                <img v-if="dialogState.formData.logo" :src="dialogState.formData.logo" class="avatar" style="width: 120px">
                                <i v-else class="el-icon-plus avatar-uploader-icon"></i>
                            </el-upload>
                        </el-form-item>

                        <el-form-item label="有效" prop="enable">
                            <el-switch on-text="是" off-text="否" v-model="dialogState.formData.enable"></el-switch>
                        </el-form-item>
                        <el-form-item label="备注" prop="comments">
                            <el-input size="small" type="textarea" v-model="dialogState.formData.comments"></el-input>
                        </el-form-item>
                    </el-col>
                </el-row>

                <el-row :gutter="20">
                    <el-col :span="2"></el-col>
                    <el-col :span="20" style="text-align: right">
                        <el-form-item>
                        <el-button size="medium" type="primary" @click="submitForm('ruleForm')">{{dialogState.edit ? '更新' : '保存'}}</el-button>
                        <el-button size="medium" @click="resetForm('ruleForm')">重置</el-button>
                    </el-form-item>
                    </el-col>
                    <el-col :span="2"></el-col>
                </el-row>
            </el-form>
        </el-dialog>
    </div>
</template>
<script>
import crypto from "~server/lib/utils/crypto.js";
import services from '../../store/services.js';
const validatorUtil = require('../../../../utils/validatorUtil.js');

export default {
    props: {
        dialogState: Object,
        groups: Array
    },
    computed: {
        bShowPassEdit () {
            return this.$data.bChangePass || this.$props.dialogState.formData.password == '';
        }
    },

    data() {
        return {
            bChangePass: false,
            rules: {
                userName: [{
                    required: true,
                    message: '请输入用户名',
                    trigger: 'blur'
                }, {
                    validator: (rule, value, callback) => {
                        if (!validatorUtil.checkUserName(value)) {
                            callback(new Error('5-12个英文字符!'));
                        } else {
                            callback();
                        }
                    },
                    trigger: 'blur'
                }],
                name: [{
                    required: true,
                    message: '请输入用户姓名',
                    trigger: 'blur'
                }, {
                    validator: (rule, value, callback) => {
                        if (!validatorUtil.checkName(value)) {
                            callback(new Error('2-6个中文字符!'));
                        } else {
                            callback();
                        }
                    },
                    trigger: 'blur'
                }
                ],
                password: [{
                    required: true,
                    message: '请输入密码',
                    trigger: 'blur'
                }, {
                    validator: (rule, value, callback) => {
                        if (!validatorUtil.checkPwd(value)) {
                            callback(new Error('6-12位，只能包含字母、数字和下划线!'));
                        } else {
                            callback();
                        }
                    },
                    trigger: 'blur'
                }],
                confirmPassword: [{
                    required: true,
                    message: '请确认密码',
                    trigger: 'blur'
                }, {
                    validator: (rule, value, callback) => {
                        if (value !== this.dialogState.formData.password) {
                            callback(new Error('两次输入密码不一致!'));
                        } else {
                            callback();
                        }
                    },
                    trigger: 'blur'
                }],
                group: [{
                    required: true,
                    message: '请选择用户组',
                    trigger: 'change'
                }],
                phoneNum: [{
                    required: false,
                    message: '请输入正确的手机号',
                    trigger: 'blur'
                }, {
                    validator: (rule, value, callback) => {
                        if (!validatorUtil.checkPhoneNum(value)) {
                            callback(new Error('请填写正确的手机号码!'));
                        } else {
                            callback();
                        }
                    },
                    trigger: 'blur'
                }],
                email: [{
                    required: false,
                    message: '请填写邮箱',
                    trigger: 'blur'
                }, {
                    validator: (rule, value, callback) => {
                        if (!validatorUtil.checkEmail(value)) {
                            callback(new Error('请填写正确的邮箱!'));
                        } else {
                            callback();
                        }
                    },
                    trigger: 'blur'
                }],
                comments: [{
                    message: '请填写备注',
                    trigger: 'blur'
                }, {
                    min: 5,
                    max: 30,
                    message: '请输入5-30个字符',
                    trigger: 'blur'
                }]
            }
        };q
    },
    methods: {

        handleAvatarSuccess(res, file) {
            let imageUrl = res;
            this.$set(this.$props.dialogState.formData, 'logo', imageUrl);
//            this.$props.dialogState.formData.logo = imageUrl;
        },

        beforeAvatarUpload(file) {
            const isJPG = file.type === 'image/jpeg';
            const isPNG = file.type === 'image/png';
            const isGIF = file.type === "image/gif";
            const isLt2M = file.size / 1024 / 1024 < 2;

            if (!isJPG && !isPNG && !isGIF) {
                this.$message.error('上传头像图片只能是 JPG 格式!');
            }
            if (!isLt2M) {
                this.$message.error('上传头像图片大小不能超过 2MB!');
            }
            return (isJPG || isPNG || isGIF) && isLt2M;
        },

        confirm() {
            this.$store.dispatch('hideAdminUserForm')
        },

        submitForm(formName) {
            this.$refs[formName].validate((valid) => {
                if (valid) {
                    let params = Object.assign({},this.dialogState.formData);
                    params.password = crypto.MD5(params.password);
                    params.confirmPassword = crypto.MD5(params.confirmPassword);
                    // 更新
                    if (this.dialogState.edit) {
                        services.updateAdminUser(params).then((result) => {
                            if (result.data.state === 'success') {
                                this.$store.dispatch('hideAdminUserForm');
                                this.$store.dispatch('getAdminUserList');
                                this.$message({
                                    message: '更新成功',
                                    type: 'success'
                                });
                            } else {
                                this.$message.error(result.data.message);
                            }
                        });
                    } else {
                        // 新增
                        services.addAdminUser(params).then((result) => {
                            if (result.data.state === 'success') {
                                this.$store.dispatch('hideAdminUserForm');
                                this.$store.dispatch('getAdminUserList');
                                this.$message({
                                    message: '添加成功',
                                    type: 'success'
                                });
                            } else {
                                this.$message.error(result.data.message);
                            }
                        })
                    }

                } else {
                    console.log('error submit!!');
                    return false;
                }
            });
        },
        resetForm(formName) {
            this.$refs[formName].resetFields();
        }

    }
}
</script>