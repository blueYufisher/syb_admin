<template>
    <div style="margin-top: 10px" v-loading="fullloading">
        <div style="margin-bottom: 10px;display: flex;justify-content: center;align-items: center">
            <el-input
                    placeholder="默认展示部分用户，可以通过用户名搜索更多用户..."
                    prefix-icon="el-icon-search"
                    size="small"
                    style="width: 400px;margin-right: 10px"
                    v-model="keywords">
            </el-input>
            <el-button size="small" type="primary" icon="el-icon-search" @click="searchClick">搜索</el-button>
        </div>
        <div style="display: flex;justify-content: space-around;flex-wrap: wrap;text-align: left">
            <el-card style="width: 350px;margin-bottom: 20px" v-for="(item,index) in hrs" :key="item.id"
                     v-loading="cardLoading[index]">
                <div slot="header" class="clearfix">
                    <span>{{item.userName}}</span>
                    <el-button type="text"
                               style="color: #f6061b;margin: 0px;float: right; padding: 3px 0;width: 15px;height:15px"
                               icon="el-icon-delete" @click="deleteHr(item.id)"><i class="el-icon-delete"></i></el-button>
                </div>
                <div>
                    <div style="width: 100%;display: flex;justify-content: center">
                        <img :src="item.userface" alt="item.userName"
                             style="width: 70px;height: 70px;border-radius: 70px">
                    </div>
                    <div style="margin-top: 20px">
                        <div><span class="user-info">用户名:{{item.userName}}</span></div>
                        <div><span class="user-info">手机号码:{{item.phone}}</span></div>
                        <div><span class="user-info">QQ:{{item.qq}}</span></div>
                        <div><span class="user-info">邮箱:{{item.mail}}</span></div>
                        <div class="user-info" style="display: flex;align-items: center;margin-bottom: 3px">
                            用户状态:
                            <el-switch
                                    style="display: inline;margin-left: 5px"
                                    v-model="item.state"
                                    active-color="#13ce66"
                                    inactive-color="#aaaaaa"
                                    active-text="启用"
                                    :key="item.id"
                                    @change="switchChange(item.state,item.id,index)"
                                    inactive-text="禁用">
                            </el-switch>
                        </div>
                        <div class="user-info">
                            用户角色:
                            <el-tag
                                    v-for="role in item.roles"
                                    :key="role.rid"
                                    type="success"
                                    size="mini"
                                    style="margin-right: 5px"
                                    :disable-transitions="false">{{role.namezh}}
                            </el-tag>
                            <el-popover
                                    v-loading="eploading[index]"
                                    placement="right"
                                    title="角色列表"
                                    width="200"
                                    @hide="updateHrRoles(item.id,index)"
                                    :key="item.id"
                                    trigger="click">
                                <el-select v-model="selRoles" multiple placeholder="请选择角色">
                                    <el-option
                                            v-for="ar in allRoles"
                                            :key="ar.rid"
                                            :label="ar.namezh"
                                            :value="ar.rid">
                                    </el-option>
                                </el-select>
                                <!--<el-button type="text" icon="el-icon-more" style="color: #09c0f6;padding-top: 0px"-->
                                <!--slot="reference"-->
                                <!--@click="loadSelRoles(item.roles,index)" :disabled="moreBtnState"></el-button>-->
                                <i class="el-icon-more" style="color: #09c0f6;cursor: pointer" slot="reference"
                                   @click="loadSelRoles(item.roles,index)" disabled="true"></i>
                            </el-popover>
                        </div>
                        <div><span class="user-info">备注:{{item.note}}</span></div>
                    </div>
                </div>
            </el-card>
        </div>
    </div>
</template>
<script>
    import api from '../../common/js/interface'
    import userAvatar from '../../assets/user-256.png'

    export default {
        data() {
            return {
                serverUrl: process.env.API_ROOT,  //打包部署上线时
                userface:"",
                keywords: '',
                fullloading: false,
                hrs: [],
                cardLoading: [],
                eploading: [],
                allRoles: [],
                moreBtnState: false,
                selRoles: [],
                selRolesBak: []
            }
        },
        mounted: function () {
            this.initCards();
            this.loadAllRoles();
        },
        methods: {
            searchClick() {
                this.initCards();
                this.loadAllRoles();
            },
            updateHrRoles(hrId, index) {
                this.moreBtnState = false;
                var _this = this;
                if (this.selRolesBak.length == this.selRoles.length) {
                    for (var i = 0; i < this.selRoles.length; i++) {
                        for (var j = 0; j < this.selRolesBak.length; j++) {
                            if (this.selRoles[i] == this.selRolesBak[j]) {
                                this.selRolesBak.splice(j, 1);
                                break;
                            }
                        }
                    }
                    if (this.selRolesBak.length == 0) {
                        return;
                    }
                }
                this.eploading.splice(index, 1, true)
                let postData = {
                    rids: this.selRoles,
                    paramObj: {
                        id: hrId
                    }
                }
                api.User.updateUserRoles(JSON.stringify(postData), resp => {
                    console.log("updateUserRoles:", resp);
                    _this.eploading.splice(index, 1, false);
                    if (resp && resp.status == 200) {
                        var data = resp.body;
                        _this.$message({type: 'success', message: data.message});
                        if (data.status) {
                            _this.refreshHr(hrId, index);
                        }
                    } else {
                        _this.$message({type: 'error', message: data.message});
                    }
                })
                // this.putRequest("/system/hr/roles", {
                //     hrId: hrId,
                //     rids: this.selRoles
                // }).then(resp => {
                //     _this.eploading.splice(index, 1, false);
                //     if (resp && resp.status == 200) {
                //         var data = resp.data;
                //         _this.$message({type: data.status, message: data.msg});
                //         if (data.status == 'success') {
                //             _this.refreshHr(hrId, index);
                //         }
                //     }
                // });
            },
            refreshHr(hrId, index) {
                var _this = this;
                _this.cardLoading.splice(index, 1, true);
                api.User.getUserById(hrId, resp => {
                    let user = resp.body.data;
                    user.userface = user.userface !== null ?
                        _this.serverUrl + "\\avatar\\" + user.userface
                        : userAvatar;
                    _this.cardLoading.splice(index, 1, false);
                    _this.hrs.splice(index, 1, user);
                })
                // this.putRequest("/system/hr/id/" + hrId).then(resp => {
                //     _this.cardLoading.splice(index, 1, false)
                //     _this.hrs.splice(index, 1, resp.data);
                // })
            },
            loadSelRoles(hrRoles, index) {
                console.log("loadSelRoles:", hrRoles);
                this.moreBtnState = true;
                this.selRoles = [];
                this.selRolesBak = [];
                hrRoles.forEach(role => {
                    this.selRoles.push(role.rid)
                    this.selRolesBak.push(role.rid)
                })
            },
            loadAllRoles() {
                var _this = this;
                api.Role.roles(resp => {
                    _this.fullloading = false;
                    if (resp && resp.status == 200) {
                        _this.allRoles = resp.body;
                    }
                })
                // this.getRequest("/system/basic/roles").then(resp => {
                //     _this.fullloading = false;
                //     if (resp && resp.status == 200) {
                //         _this.allRoles = resp.data;
                //     }
                // })
            },
            switchChange(newValue, hrId, index) {
                var _this = this;
                _this.cardLoading.splice(index, 1, true)
                if (newValue) {
                    newValue = 1;
                } else {
                    newValue = 0;
                }
                let postData = {
                    state: newValue,
                    id: hrId
                }
                api.User.updateUser(JSON.stringify(postData), resp => {
                    console.log("updateUser:", resp);
                    _this.cardLoading.splice(index, 1, false)
                    if (resp && resp.status == 200) {
                        var data = resp.body;
                        _this.$message({type: 'success', message: data.message});
                        if (data.status == 'error') {
                            _this.$message({type: 'error', message: data.message});
                            _this.refreshHr(hrId, index);
                        }
                    } else {
                        _this.refreshHr(hrId, index);
                    }
                })
                // this.putRequest("/system/hr/", {
                //     state: newValue,
                //     id: hrId
                // }).then(resp => {
                //     _this.cardLoading.splice(index, 1, false)
                //     if (resp && resp.status == 200) {
                //         var data = resp.data;
                //         _this.$message({type: data.status, message: data.msg});
                //         if (data.status == 'error') {
                //             _this.refreshHr(hrId, index);
                //         }
                //     } else {
                //         _this.refreshHr(hrId, index);
                //     }
                // })
            },
            initCards() {
                this.fullloading = true;
                var _this = this;
                var searchWords;
                if (this.keywords === '') {
                    searchWords = 'all';
                } else {
                    searchWords = this.keywords;
                }
                api.User.getUsersByKeywords(searchWords, resp => {
                    // console.log("initCards:", resp);
                    if (resp && resp.status == 200) {
                        _this.hrs = resp.body.data;
                        _this.hrs.forEach(item => {
                            item.userface = item.userface !== null ?
                                _this.serverUrl + "\\avatar\\" + item.userface
                                : userAvatar;

                            item.state = item.state === 1 ? true : false;
                        });
                        var length = resp.body.data.length;
                        _this.cardLoading = Array.apply(null, Array(length)).map(function (item, i) {
                            return false;
                        });
                        _this.eploading = Array.apply(null, Array(length)).map(function (item, i) {
                            return false;
                        });
                    }
                })
                // this.getRequest("/system/hr/" + searchWords).then(resp => {
                //     if (resp && resp.status == 200) {
                //         _this.hrs = resp.data;
                //         var length = resp.data.length;
                //         _this.cardLoading = Array.apply(null, Array(length)).map(function (item, i) {
                //             return false;
                //         });
                //         _this.eploading = Array.apply(null, Array(length)).map(function (item, i) {
                //             return false;
                //         });
                //     }
                // })
            },
            deleteHr(hrId) {
                var _this = this;
                this.fullloading = true;
                api.User.deleteUser(hrId, resp => {
                    _this.fullloading = false;
                    if (resp && resp.status == 200) {
                        var data = resp.body;
                        _this.$message({type: 'success', message: data.message});
                        if (data.status) {
                            _this.initCards();
                            _this.loadAllRoles();
                        }
                    } else {
                        _this.$message({type: 'error', message: data.message});
                        // _this.refreshHr(hrId, index);
                    }
                })
                // this.deleteRequest("/system/hr/" + hrId).then(resp => {
                //     _this.fullloading = false;
                //     if (resp && resp.status == 200) {
                //         var data = resp.data;
                //         _this.$message({type: data.status, message: data.msg});
                //         if (data.status == 'success') {
                //             _this.initCards();
                //             _this.loadAllRoles();
                //         }
                //     }
                // })
            }
        }
    }
</script>
<style>
    .user-info {
        font-size: 12px;
        color: #09c0f6;
    }
</style>
