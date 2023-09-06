<template>
    <div>
        <nav>
            <slot name="goHome" class="link"></slot>
            |
            <slot name="issue-query" class="link"></slot>
            |
            <slot name="history" class="link"></slot>
        </nav>
        <slot name="title" v-if="!isLogin"></slot>

        <div class="login-form" v-if="!isLogin && !isPosted && !assign">
            <input
                type="text"
                v-model.lazy.trim="username"
                placeholder="请输入用户名"
            />
            <br />

            <input
                type="passowrd"
                v-model.lazy.trim="password"
                placeholder="请输入密码"
            />
            <br />
            <button @click.prevent="login">登录</button>
        </div>
        <div v-else-if="isPosted && !isLogin && !assign">
            <h2>正在登录。。。</h2>
        </div>
        <div v-else-if="isPosted && isLogin && !assign">
            <div v-if="isAdmin">
                <h1>欢迎管理员: {{ username }}</h1>
                <button @click.prevent="logout">退出登录</button>
                <br />

                工单状态：
                <select
                    name=""
                    id=""
                    v-model="filterState"
                    @change="modifyList"
                >
                    <option value="all">所有工单</option>
                    <option value="wait">未处理</option>
                    <option value="fixing">处理中</option>
                    <option value="complete">已完成</option>
                </select>
                <br />

                <div v-if="!triedIssueList && !assign">
                    <h2>加载中。。。</h2>
                </div>
                <div v-else-if="triedIssueList && allIssueList.length === 0">
                    <h2>无法获取工单</h2>
                </div>
                <div v-else-if="triedIssueList && allIssueList.length !== 0">
                    <div v-for="item in filteredIssueList" class="single-issue">
                        <ul v-for="prop in item">
                            <li>{{ prop[0] }} : {{ prop[1] }}</li>
                        </ul>
                        <button @click.prevent="assignIssue(item)">
                            分配负责人
                        </button>
                    </div>
                </div>
            </div>

            <div v-else>
                <h1>欢迎: {{ username }}</h1>
                <button @click.prevent="logout">退出登录</button>
                <br />

                <ul v-for="issue in staffIssueList" class="staffIssueList">
                    <div v-for="item in renderIssue(issue)">
                        <li>{{ item[0] }} : {{ item[1] }}</li>
                        <button
                            v-if="
                                item[0] === '工单状态' && item[1] !== '已完成'
                            "
                            @click.prevent="completeIssue(issue.id)"
                        >
                            完成
                        </button>
                    </div>
                    <br />
                </ul>
            </div>
        </div>
        <div v-else-if="assign">
            <h1>所有员工</h1>
            <button @click.prevent="cancelAssign">取消</button>
            <div
                v-for="staff in convertStaffList(staffList)"
                class="assign-page"
            >
                <div v-for="item in staff">
                    <li>{{ item[0] }} : {{ item[1] }}</li>
                </div>
                <button @click.prevent="assignStaff(staff[0][1])">分配</button>
            </div>
        </div>
    </div>
</template>
<script>
export default {
    props: ["renderIssue"],
    name: "Admin",
    data() {
        return {
            isLogin: false,
            username: "",
            isAdmin: false,
            password: "",
            isPosted: false,

            assignIssueId: 0,
            assign: false,

            allIssueList: [],
            filteredIssueList: [],
            triedIssueList: false,

            staffList: [],
            staffIssueList: [],
            staffId: 0,

            filterState: "all",

            issueLocalPath: "http://127.0.0.1:8080/api/v1/issue",
            issueRemotePath: "http://110.40.154.138:8080/api/v1/issue",
            issueRemoteGetAllPath:
                "http://110.40.154.138:8080/api/v1/issue-all",
            issueRemoteCompletePath:
                "http://110.40.154.138:8080/api/v1/issue-complete",

            staffLocalPath: "http://127.0.0.1:8080/api/v1/staff",
            staffRemotePath: "http://110.40.154.138:8080/api/v1/staff",
            staffRemoteGetAllPath:
                "http://110.40.154.138:8080/api/v1/staff-all",

            staffRemoteIssuePath:
                "http://110.40.154.138:8080/api/v1/staff-issue-all",

            stateMap: new Map([
                ["wait", "等待处理"],
                ["fixing", "处理中"],
                ["complete", "已完成"],
            ]),
        };
    },

    methods: {
        async completeIssue(issueId) {
            const response = await fetch(
                `${this.issueRemoteCompletePath}?issueId=${issueId}`
            );
            if (!response.ok) {
                alert("请勿重复完成工单!");
            }

            const responseData = await response.json();
            if (responseData.status !== "success") {
                alert("请勿重复完成工单!");
            }

            alert("完成工单！");
            this.getStaffIssueList(this.staffId);
        },

        cancelAssign() {
            this.assign = false;
        },

        convertStaffList(staffList) {
            const mapedList = staffList.map((staff) => {
                return [...Object.entries(staff)]
                    .map((item) => {
                        if (item[0] === "id") {
                            return ["员工号", item[1]];
                        } else if (item[0] === "staffName") {
                            return ["员工姓名", item[1]];
                        } else {
                            return [];
                        }
                    })
                    .filter((item) => item.length > 0);
            });
            console.log(mapedList);

            return mapedList;
        },

        assignIssue(item) {
            this.assignIssueId = item[0][1];
            this.assign = true;
        },

        async assignStaff(staffId) {
            const response = await fetch(this.issueRemotePath, {
                method: "PATCH",
                headers: {
                    "Content-Type": "application/json",
                },
                body: JSON.stringify({
                    id: this.assignIssueId,
                    staffId,
                }),
            });

            const responseData = await response.json();
            if (responseData.status !== "success") {
                if (responseData.message === "missing fields") {
                    alert("缺少参数!");
                    this.assign = false;
                    return;
                } else {
                    alert("请勿重复分配!");
                    this.assign = false;
                    return;
                }
            } else {
                alert("分配成功!");
                this.assign = false;
                this.renderAdminIssueList();
                return;
            }
        },

        modifyList() {
            if (this.filterState === "all") {
                this.filteredIssueList = this.allIssueList;
                return;
            }

            // Filter the state equal the filter state in the allIssueList
            this.filteredIssueList = this.allIssueList.filter((issue) => {
                const stateProp = issue[3];
                return stateProp[1] === this.stateMap.get(this.filterState);
            });
        },

        async login() {
            this.isPosted = true;
            const response = await fetch(this.staffRemotePath, {
                method: "POST",
                headers: {
                    "Content-Type": "application/json",
                },
                body: JSON.stringify({
                    username: this.username,
                    password: this.password,
                }),
            }).catch((error) =>
                console.log(`error from user fetch : ${error}`)
            );

            if (!response.ok) {
                this.isPosted = false;
                alert("登录失败! 请检查输入！");
                return;
            }

            const responseData = await response.json();
            this.isLogin = true;
            this.isAdmin = responseData.data.user.staffRole === "admin";
            this.staffId = responseData.data.user.id;
            this.getAllStaff();

            if (this.isAdmin) {
                this.renderAdminIssueList();
            } else {
                this.renderStaffIssueList();
            }
        },

        logout() {
            this.isLogin = false;
            this.isAdmin = false;
            this.username = "";
            this.password = "";
            this.isPosted = false;
        },

        /**
         * Renders the admin issue list asynchronously.
         *
         * @return {Promise<void>} - A promise that resolves when the rendering is complete.
         */
        async renderAdminIssueList() {
            let issueRawList = await this.getAllList();

            issueRawList = issueRawList.map((issue) => {
                return this.renderAdminIssue(issue);
            });

            this.allIssueList = issueRawList;
            this.filteredIssueList = this.allIssueList;
            this.filterState = "all";
            console.log(`All issue list: ${this.allIssueList}`);
        },

        async getAllList() {
            const response = await fetch(this.issueRemoteGetAllPath);
            this.triedIssueList = true;
            if (!response.ok) {
                alert("获取工单列表失败!");
                return;
            }

            const responseData = await response.json();
            return responseData.data.issueArr;
        },

        getStaffNameById(id) {
            const staff = this.staffList.filter((staff) => staff.id === id)[0];

            return staff.staffName;
        },

        renderAdminIssue(issueObj) {
            // const stateMap = new Map([
            //     ["wait", "等待处理"],
            //     ["fixing", "处理中"],
            //     ["complete", "已完成"],
            // ]);

            const resAdminIssue = [...Object.entries(issueObj)].map((item) => {
                if (item[0] === "id") {
                    return ["工单号", item[1]];
                } else if (item[0] === "poster") {
                    return ["提交人", item[1]];
                } else if (item[0] === "description") {
                    return ["工单描述", item[1]];
                } else if (item[0] === "state") {
                    return ["工单状态", this.stateMap.get(item[1])];
                } else if (item[0] === "createDate") {
                    return [
                        "创建时间",
                        item[1].slice(0, 10) + " " + item[1].slice(11, 19),
                    ];
                } else if (item[0] === "staffId") {
                    return [
                        "负责人",
                        item[1] === null
                            ? "无"
                            : this.getStaffNameById(item[1]),
                    ];
                } else if (item[0] === "fixedDate") {
                    return ["完成时间", item[1] === null ? "无" : item[1]];
                } else {
                    return [];
                }
            });

            return resAdminIssue.filter((item) => item.length !== 0);
        },

        renderStaffIssueList() {
            this.getStaffIssueList(this.staffId);
        },

        async getAllStaff() {
            const response = await fetch(this.staffRemoteGetAllPath);
            if (!response.ok) {
                alert("获取员工列表失败！");
            }

            const responseData = await response.json();
            this.staffList = responseData.data;
        },

        async getStaffIssueList(staffId) {
            console.log("Enter getStaffIssueList");

            const response = await fetch(
                `${this.staffRemoteIssuePath}?staffId=${staffId}`
            );
            if (!response.ok) {
                alert("获取工单列表失败!");
                return;
            }

            const responseData = await response.json();
            this.staffIssueList = responseData.data.issueList.filter(
                (issue) => issue.id !== null
            );
        },
    },
};
</script>
<style scoped>
input {
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

select {
    width: 50%;
    height: 30px;
    margin-bottom: 5px;
}

button {
    margin-bottom: 10px;
    text-align: center;
}

li {
    text-align: left;
    margin-left: 10px;
}

.single-issue {
    border-style: solid;
}

.assign-page {
    border-style: solid;
}

.staffIssueList {
    border-style: solid;
}
</style>
