/*
 * Copyright 2014-2019 the original author or authors.
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *      http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */
<template>
    <div>
        <el-select v-model="userName" placeholder="请选择用户" @change="changeId">
            <el-option :label="item.userName" :value="item.userName" :key="item.userId" v-for='(item,index) in userList'></el-option>
        </el-select>
        <div v-if='inputs.length' v-for='(item,index) in inputs' :key='item.name' style="padding: 10px 0;">
            <span style="display:inline-block;width:80px;">{{item.type}}</span>
            <el-input v-model="parameter[index]" style="width: 250px;"></el-input>
        </div>
        <div style="text-align:right">
            <el-button type="primary" @click="submit">确定</el-button>
            <el-button @click="close">取消</el-button>
        </div>
    </div>
</template>
<script>
import { sendTransation, getUserList } from "@/util/api";
import errcode from "@/util/errcode";
export default {
    name: "changeUser",
    props: ["abi"],
    data: function() {
        return {
            userName: "",
            userList: [],
            userId: null,
            inputs: [],
            parameter: [],
            abifile: JSON.parse(this.abi)
        };
    },
    mounted: function() {
        this.getUserData();
        this.changeConstructor();
    },
    methods: {
        changeConstructor: function() {
            if (this.abifile.length) {
                this.abifile.forEach(value => {
                    if (value.type === "constructor") {
                        this.inputs = value.inputs;
                    }
                });
            }
        },
        changeId: function() {
            if (this.userName) {
                this.userList.forEach(value => {
                    if (this.userName === value.userName) {
                        this.userId = value.userId;
                    }
                });
            }
        },
        close: function() {
            this.$emit("close");
        },
        submit: function() {
            let data = {
                userId: this.userId,
                params: this.parameter
            };

            this.$emit("change", data);
            this.$emit("close");
        },
        getUserData: function() {
            let reqData = {
                networkId: localStorage.getItem("networkId"),
                pageNumber: 1,
                pageSize: 1000
            };
            getUserList(reqData, {})
                .then(res => {
                    if (res.data.code === 0) {
                        res.data.data.forEach(value => {
                            if (value.hasPk === 1) {
                                this.userList.push(value);
                            }
                        });
                        if (this.userList.length) {
                            this.userName = this.userList[0].userName;
                            this.userId = this.userList[0].userId;
                        }
                    } else {
                        this.$message({
                            type: "error",
                            message: errcode.errCode[res.data.code].cn
                        });
                    }
                })
                .catch(err => {
                    this.$message({
                        type: "error",
                        message: "系统错误！"
                    });
                });
        }
    }
};
</script>

