<template>
    <div class="bg-gray cb user-list" v-loading="userLoading">
        <div class="pc w1200">
            <h3 class="title tc">账户管理</h3>
            <div class="w1400">
                <div class="top_ico">
                    <i class="el-icon-plus click" @click="toUrl('newAddress')"></i>
                </div>
                <el-table :data="addressList" stripe border>
                    <el-table-column prop="address" label="地址" align="center" min-width="360">
                    </el-table-column>
                    <el-table-column :label="$t('public.alias')" align="center" width="130">
                        <template slot-scope="scope">
                            <span>{{scope.row.alias }}</span>
                        </template>
                    </el-table-column>
                    <el-table-column label="备注" align="center" width="150">
                        <template slot-scope="scope">
                            <span v-if="scope.row.note" class="click"
                                  @click="setNote(scope.row)">{{scope.row.note}}</span>
                            <i v-else class="el-icon-edit click" @click="setNote(scope.row)"></i>
                        </template>
                    </el-table-column>
                    <el-table-column prop="totalLock" label="锁定" align="center" width="150">
                    </el-table-column>
                    <el-table-column prop="balance" label="可用余额" align="center" width="150">
                    </el-table-column>
                    <el-table-column label="操作" align="center" width="250">
                        <template slot-scope="scope">
                            <!--<label class="click tab_bn" @click="editPassword(scope.row)">修改密码</label>
                            <span class="tab_line">|</span>-->
                            <label class="click tab_bn" @click="backAddress(scope.row)">备份</label>
                            <span class="tab_line">|</span>
                            <label class="click tab_bn" @click="deleteAddress(scope.row)">移除</label>
                            <span class="tab_line">|</span>
                            <label disabled v-if="scope.row.isCurrent">进入</label>
                            <label class="click tab_bn" @click="selectionAddress(scope.row)" v-else>进入</label>
                        </template>
                    </el-table-column>
                </el-table>
                <div class="pages">
                    <div class="page-total">共 {{addressList.length}}</div>
                </div>
            </div>
        </div>

        <div class="mobile bg">
            <div class="bg3">
                <BackBar :isBackIcon="isBackIcon" :rightTitle="rightTitle">
                </BackBar>

                <div class="addrss-list">
                    <SelectAddress :shortAddress="isShort" ref="selectAccount">
                    </SelectAddress>
                </div>

                <ul class="lis">
                    <li @click="toUrl('newAddress','')">{{$t('newAddress.newAddress2')}} <i
                            class="el-icon-arrow-right"></i></li>
                    <li @click="toUrl('newAddress','')">{{$t('address.address14')}} <i class="el-icon-arrow-right"></i>
                    </li>
                    <li @click="toUrl('backupsAddress','')">{{$t('address.address15')}} <i
                            class="el-icon-arrow-right"></i></li>
                    <li @click="toUrl('backupsAddress','')" class="last">{{$t('address.address16')}} <i
                            class="el-icon-arrow-right"></i></li>
                    <li @click="deleteAddress" class="remove">{{$t('address.address17')}} <i
                            class="el-icon-arrow-right"></i></li>
                    <li @click="cleanAll" class="remove">{{$t('address.address18')}} <i class="el-icon-arrow-right"></i>
                    </li>
                </ul>
            </div>

        </div>

        <el-dialog title="设置备注" :visible.sync="noteDialog" :close-on-click-modal="false" :close-on-press-escape="false">
            <el-form :model="noteForm" :rules="rulesForm" ref="noteForm">
                <el-form-item label="地址">
                    <el-input v-model="noteForm.address" autocomplete="off" disabled></el-input>
                </el-form-item>
                <el-form-item label="备注" prop="note">
                    <el-input v-model="noteForm.note" autocomplete="off"></el-input>
                </el-form-item>
            </el-form>
            <div slot="footer" class="dialog-footer tc">
                <el-button @click="resetForm('noteForm')">取 消</el-button>
                <el-button type="primary" @click="submitForm('noteForm')">确 定</el-button>
            </div>
        </el-dialog>

        <Password ref="password" @passwordSubmit="passSubmit">
        </Password>
    </div>
</template>

<script>
    import Password from '@/components/PasswordBar'
    import {divisionDecimals, accountList, Plus} from '@/api/util'
    import {passwordVerification} from '@/api/requestData'

    export default {
        data() {
            return {
                isBackIcon: true,//back 返回icon显示
                rightTitle: this.$t('address.address13'),//back title
                addressList: [],//地址列表
                selectAddressInfo: '', //操作的地址信息
                remarkDialog: false,//备注弹框
                remarkInfo: '',//备注信息
                prefix: 'TNVT',//地址前缀
                isShort: false,
                userLoading: true,//加载动画
                noteDialog: false,//设置备注
                noteForm: {
                    address: '',
                    note: ''
                },
                rulesForm: {
                    note: [
                        {min: 1, max: 20, message: '长度在 1 到 20 个字符', trigger: 'blur'}
                    ],
                }
            }
        },
        components: {
            Password
        },
        created() {

        },
        mounted() {
            this.getAddressList();
        },
        methods: {

            //获取地址组件选择的地址信息
            changeAccountInfo() {
                if (this.$refs.selectAccount.accountInfo) {
                    this.selectAddressInfo = this.$refs.selectAccount.accountInfo;
                }
            },

            /**
             * 获取账户列表
             */
            async getAddressList() {
                let newAccountData = accountList(0);
                this.selectAddressInfo = accountList(1);
                for (let item of newAccountData) {
                    if (!item.note || item.note.toString() === 'undefined') {
                        item.note = '';
                    }
                    let newAddressInfo = await this.getAddressInfoByNode(item);
                    //console.log(newAddressInfo);
                    if (newAddressInfo.success) {
                        item.alias = newAddressInfo.data.alias;
                        item.balance = divisionDecimals(newAddressInfo.data.balance);
                        item.consensusLock = divisionDecimals(newAddressInfo.data.consensusLock);
                        item.timeLock = divisionDecimals(newAddressInfo.data.timeLock);
                        item.totalLock = Plus(item.consensusLock, item.timeLock).toString();
                        item.totalBalance = divisionDecimals(newAddressInfo.data.totalBalance);
                    }
                }
                this.addressList = newAccountData;
                this.userLoading = false;
                //console.log(this.addressList);

                //如果没有账户跳转到创建地址界面
                if (newAccountData.length === 0) {
                    this.$router.push({
                        name: "newAddress",
                        query: {'address': ''}
                    })
                }
            },

            /**
             * 获取地址网络信息
             * @param addressInfo
             **/
            async getAddressInfoByNode(addressInfo) {
                return await this.$post('/', 'getAccount', [addressInfo.address])
                    .then((response) => {
                        //console.log(response);
                        if (response.hasOwnProperty('result')) {
                            return {success: true, data: response.result}
                        } else {
                            return {success: false, data: response}
                        }
                    })
                    .catch((error) => {
                        return {success: false, data: error}
                    });
            },

            /**
             * @disc: 设置备注
             * @params:
             * @date: 2020-12-01 15:52
             * @author: Wave
             */
            setNote(addressInfo) {
                //console.log(addressInfo);
                if (!addressInfo.note) {
                    addressInfo.note = ''
                }
                this.selectAddressInfo = addressInfo;
                this.noteForm.address = this.selectAddressInfo.address;
                this.noteForm.note = this.selectAddressInfo.note;
                this.noteDialog = true;
            },

            /**
             * @disc: 设置备注提交
             * @params:
             * @date: 2020-12-01 15:54
             * @author: Wave
             */
            submitForm(formName) {
                this.$refs[formName].validate((valid) => {
                    if (valid) {
                        console.log(this.noteForm);
                        console.log(this.addressList);
                        for (let item of this.addressList) {
                            if (item.address === this.selectAddressInfo.address) {
                                item.note = this.noteForm.note
                            }
                        }

                        let newAccountData = accountList(0);
                        for (let item of newAccountData) {
                            if (item.address === this.selectAddressInfo.address) {
                                item.note = this.noteForm.note
                            }
                        }
                        localStorage.setItem('addressData', JSON.stringify(newAccountData));

                        let selectAddressInfo = accountList(1);
                        if (selectAddressInfo.address === this.selectAddressInfo.address) {
                            selectAddressInfo.note = this.noteForm.note;
                            localStorage.setItem('accountInfo', JSON.stringify(selectAddressInfo));
                        }
                        this.noteDialog = false;
                    } else {
                        return false;
                    }
                });
            },

            /**
             * @disc: 取消设置备注
             * @params:
             * @date: 2020-12-01 15:54
             * @author: Wave
             */
            resetForm(formName) {
                this.$refs[formName].resetFields();
                this.noteDialog = false;
            },

            /**
             *  修改密码
             * @param rowInfo
             **/
            editPassword(rowInfo) {
                this.toUrl('editPassword', rowInfo.address)
            },

            /**
             *  备份账户
             * @param rowInfo
             **/
            backAddress(rowInfo) {
                this.selectAddressInfo = rowInfo;
                this.$router.push({
                    name: "backupsAddress",
                    query: {address: rowInfo.address}
                })
            },

            /**
             *  移除账户
             **/
            deleteAddress(rowInfo) {
                this.$confirm(this.$t('tab.tab29'), this.$t('tab.tab32'), {
                    confirmButtonText: this.$t('tab.tab30'),
                    cancelButtonText: this.$t('address.address3'),
                    type: 'warning',
                    showClose: false,
                    closeOnClickModal: false,
                    closeOnPressEscape: false,
                    center: true
                }).then(() => {
                    this.backAddress(rowInfo);
                }).catch(() => {
                    this.selectAddressInfo = rowInfo;
                    this.$refs.password.showPassword(true)
                });

            },

            /**
             * 进入账户（使用账户）
             * @param rowInfo
             **/
            selectionAddress(rowInfo) {
                //console.log(rowInfo);
                let newAccountList = accountList(0);
                for (let item of newAccountList) {
                    //添加选中
                    item.isCurrent = false;
                    if (item.address === rowInfo.address) {
                        item.isCurrent = true;
                        for (let k of this.addressList) {
                            if (k.address === rowInfo.address) {
                                k.isCurrent = true;
                            } else {
                                k.isCurrent = false;
                            }
                        }
                        localStorage.setItem('accountInfo', JSON.stringify(item));
                    }
                }
                localStorage.setItem("addressData", JSON.stringify(newAccountList));
            },

            /**
             *  获取密码框的密码
             * @param password
             **/
            async passSubmit(password) {
                let accountInfo = this.selectAddressInfo;

                let passwordInfo = await passwordVerification(accountInfo, password);
                if (!passwordInfo.success) {
                    this.$message({message: this.$t('tips.tips18'), type: 'error', duration: 3000});
                    return;
                }

                if (accountInfo.address === passwordInfo.address) {
                    this.addressList.splice(this.addressList.findIndex(item => item.address === accountInfo.address), 1);
                    if (this.addressList.length === 0) {
                        this.toUrl('newAddress', '');
                        localStorage.removeItem('accountInfo');
                    } else {
                        this.addressList[0].isCurrent = true;
                        localStorage.setItem('accountInfo', JSON.stringify(this.addressList[0]));
                    }
                    this.selectAddressInfo = this.addressList[0];
                    localStorage.setItem('addressData', JSON.stringify(this.addressList));
                } else {
                    this.$message({message: this.$t('address.address13'), type: 'error', duration: 1000});
                }
            },

            /**
             * 连接跳转
             * @param name
             * @param param
             */
            toUrl(name) {
                //console.log(name);
                this.$router.push({
                    name: name,
                    query: {}
                })
            },

            //清除所有
            cleanAll() {
                localStorage.clear();
                sessionStorage.clear();
                this.$router.push({
                    name: 'home',
                });
                window.location.reload();
                //window.open('')
            }
        }
    }
</script>

<style lang="less">
    @import "./../../assets/css/style";

    .user-list {

        .pc {
            height: 800px;
            background-color: #F9FAFD;

            .title {
                font-size: 20px;
                padding: 20px auto 0;
                line-height: 20px;
            }

            .w1400 {
                .top_ico {
                    float: right;
                    font-size: 26px;
                }

                .el-table {
                    .el-table__body-wrapper {
                        .el-table__body {
                            .el-table__row {
                                .cell {
                                    .tab_bn, .tab_line {
                                        padding: 0 5px;
                                    }
                                }
                            }
                        }
                    }
                }
            }

            .el-dialog__body {
                background-color: @Bcolour;
                padding: 0 20px 50px 20px;

                .address-remark {
                    margin: 50px auto 0;

                    .btn-next {
                        margin: 40px auto 0;
                        text-align: center;

                        .el-button {
                            width: 9.5rem;
                        }

                        .el-button--success {
                            span {
                                color: white;
                            }
                        }
                    }
                }
            }
        }

        .mobile {
            height: 100%;

            .addrss-list {
                height: 3.4rem;
                margin: 0.5rem 0.5rem 0;

                .el-select {
                    width: 100%;

                    .el-input {
                        .el-input__inner {
                            @media screen and (max-width: 1000px) {
                                background: #1a1c2e;
                                border: 1px solid #333651;
                            }
                        }
                    }
                }
            }

            .lis {
                margin: 0 0 0 0;
                border-top: 0.2rem solid #0A0B18;
                height: 36rem;
                padding: 0 0;

                li {
                    color: #F5F7FA;
                    border-bottom: 0.1rem solid #333651;
                    padding: 0 0.5rem;
                    line-height: 2.6rem;
                    height: 2.6rem;
                    font-size: 0.8rem;

                    i {
                        display: block;
                        float: right;
                        font-size: 1.2rem;
                        margin: 0.8rem 0 0 0;
                    }
                }

                .last {
                    border-bottom: 0;
                }

                .remove {
                    color: #dd6161;
                    border-top: 0.3rem solid #0A0B18;
                    line-height: 2.9rem;
                    height: 2.9rem;
                    border-bottom: 0;

                    i {
                        color: #F5F7FA;
                    }
                }
            }
        }
    }

    .el-message-box__wrapper {
        @media screen and (max-width: 1000px) {
            margin: 0 0.5rem 0;
            width: 96%;
        }

        .el-message-box {
            @media screen and (max-width: 1000px) {
                width: 100%;
            }

            .el-message-box__message {
                p {
                    color: red;
                }
            }
        }
    }

</style>
