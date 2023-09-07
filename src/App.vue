<template>
    <keep-alive>
        <component
            :is="componentName"
            v-if="componentName === 'Home'"
            :componentName="componentName"
            @upload-issue="saveIssue"
        >
            <template v-slot:admin-login>
                <a href="#" class="admin-login" @click="gotoLogin"
                    >管理员登陆</a
                >
            </template>

            <template v-slot:issue-query>
                <a href="#" class="issue-query" @click.prevent="gotoQuery"
                    >工单查询</a
                >
            </template>

            <template v-slot:history>
                <a href="#" class="history" @click.prevent="gotoHistory"
                    >历史工单</a
                >
            </template>

            <template v-slot:title>
                <h1>问题反馈</h1>
            </template>
        </component>

        <component
            :is="componentName"
            v-else-if="componentName === 'SubmitDisplay'"
            :issueObj="issueObj"
            @cancel-submit="cancelSubmit"
        >
            <template v-slot:goHome>
                <a href="#" class="admin-login" @click="gotoHome">回到主页</a>
            </template>

            <template v-slot:issue-query>
                <a href="#" class="issue-query" @click="gotoQuery">工单查询</a>
            </template>

            <template v-slot:history>
                <a href="#" class="history" @click.prevent="gotoHistory"
                    >历史工单</a
                >
            </template>

            <template v-slot:title>
                <h1>提交成功！</h1>
            </template>
        </component>

        <component
            :is="componentName"
            :renderIssue="renderIssue"
            :staffList="staffList"
            v-else-if="componentName === 'Query'"
        >
            <template v-slot:goHome>
                <a href="#" class="go-home" @click="gotoHome">回到主页</a>
            </template>

            <template v-slot:issue-query>
                <a href="#" class="issue-query" @click="gotoQuery">工单查询</a>
            </template>

            <template v-slot:history>
                <a href="#" class="history" @click="gotoHistory">历史工单</a>
            </template>

            <template v-slot:title>
                <h1>查询工单</h1>
            </template>
        </component>

        <component
            :is="componentName"
            :renderIssue="renderIssue"
            v-else-if="componentName === 'History'"
        >
            <template v-slot:goHome>
                <a href="#" class="admin-login" @click="gotoHome">回到主页</a>
            </template>

            <template v-slot:issue-query>
                <a href="#" class="issue-query" @click="gotoQuery">工单查询</a>
            </template>

            <template v-slot:title>
                <h1>历史工单</h1>
            </template>
        </component>

        <component
            :is="componentName"
            :renderIssue="renderIssue"
            v-else-if="componentName === 'Admin'"
        >
            <template v-slot:goHome>
                <a href="#" class="go-home" @click="gotoHome">回到主页</a>
            </template>

            <template v-slot:issue-query>
                <a href="#" class="issue-query" @click.prevent="gotoQuery"
                    >工单查询</a
                >
            </template>

            <template v-slot:history>
                <a href="#" class="history" @click.prevent="gotoHistory"
                    >历史工单</a
                >
            </template>

            <template v-slot:title>
                <h1>请登录</h1>
            </template>
        </component>
    </keep-alive>
</template>

<script>
import Home from "@/components/Home.vue";
import History from "@/components/History.vue";
import Query from "@/components/Query.vue";
import SubmitDisplay from "@/components/SubmitDisplay.vue";
import Admin from "@/components/Admin.vue";
import { staffRemotePath, staffRemoteGetAllPath } from "../config.js";

export default {
    name: "App",
    data() {
        return {
            componentName: "Home",
            issueObj: null,
            staffRemotePath,
            staffRemoteGetAllPath,
            staffList: [],
        };
    },

    components: {
        Home,
        History,
        Query,
        SubmitDisplay,
        Admin,
    },

    methods: {
        async getAllStaff() {
            const response = await fetch(this.staffRemoteGetAllPath);
            if (!response.ok) {
                alert("获取员工列表失败！");
            }

            const responseData = await response.json();
            this.staffList = responseData.data;
        },

        gotoLogin() {
            this.componentName = "Admin";
        },

        gotoQuery() {
            this.componentName = "Query";
            this.staffList = this.getAllStaff();
        },

        gotoHistory() {
            this.componentName = "History";
            this.staffList = this.getAllStaff();
        },

        saveIssue(issueObj) {
            console.log("Listened upload issue event");
            this.issueObj = issueObj;
            this.componentName = "SubmitDisplay";
        },

        gotoHome() {
            this.componentName = "Home";
        },

        cancelSubmit() {
            this.gotoHome();
        },

        async getStaffNameById(staffId) {
            if (staffId === null || !staffId) {
                console.log("missing staffId!");
                return;
            }

            const response = await fetch(
                `${this.staffRemotePath}?staffId=${staffId}`
            );
            if (!response.ok) {
                console.log("failed to fetch staff by id!");
                return;
            }

            const responseData = await response.json();
            const staff = await responseData.data.staff[0];
            console.log(staff.staff_name);

            return staff.staff_name;
        },

        renderIssue(issueObj) {
            const stateMap = new Map([
                ["wait", "等待处理"],
                ["fixing", "处理中"],
                ["complete", "已完成"],
            ]);

            const resArr = [...Object.entries(issueObj)]
                .map((item) => {
                    if (item[0] === "id") {
                        return ["工单号", item[1]];
                    } else if (item[0] === "poster") {
                        return ["提交人", item[1]];
                    } else if (item[0] === "description") {
                        return ["工单描述", item[1]];
                    } else if (item[0] === "state") {
                        return ["工单状态", stateMap.get(item[1])];
                    } else if (item[0] === "createDate") {
                        return [
                            "创建时间",
                            item[1].slice(0, 10) + " " + item[1].slice(11, 19),
                        ];
                    } else if (item[0] === "staffId") {
                        const staffName =
                            item[1] === null
                                ? "无"
                                : this.staffList.filter(
                                      (staff) => staff.id === item[1]
                                  )[0].staffName;

                        return ["负责人", staffName];
                    } else if (item[0] === "fixedDate") {
                        return [
                            "完成时间",
                            item[1] === null
                                ? "无"
                                : item[1].slice(0, 10) +
                                  " " +
                                  item[1].slice(11, 19),
                        ];
                    } else {
                        return [];
                    }
                })
                .filter((item) => item.length === 2);

            return resArr;
        },
    },
};
</script>

<style>
* {
    text-align: center;
}

input[type="text"] {
    appearance: none;
    margin-bottom: 20px;
    border: none;
    outline: none;
    border-bottom: 0.2em solid #e91e63;
    background: rgba(#e91e63, 0.2);
    border-radius: 0.2em 0.2em 0 0;
    padding: 0.4em;
    color: #e91e63;
}

img {
    width: 50%;
    height: auto;
}

button {
    appearance: none;
    border: 0.2em solid #e91e63;
    background: hsl(0 0 0/0);
    padding: 0.85em 1.5em;
    color: #e91e63;
    border-radius: 2em;
    transition: 1s;
    width: 40%;
}

button:hover,
button:focus,
button:active {
    background: #e91e63;
    color: #fff;
}

.hidden {
    display: none;
}
</style>
