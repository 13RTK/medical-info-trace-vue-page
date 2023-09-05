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
    </keep-alive>
</template>

<script>
import Home from "@/components/Home.vue";
import History from "@/components/History.vue";
import Query from "@/components/Query.vue";
import SubmitDisplay from "@/components/SubmitDisplay.vue";
import Admin from "@/components/Admin.vue";

export default {
    name: "App",
    data() {
        return {
            componentName: "Home",
            issueObj: null,
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
        gotoLogin() {
            this.componentName = "Admin";
        },

        gotoQuery() {
            this.componentName = "Query";
        },

        gotoHistory() {
            this.componentName = "History";
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

        renderIssue(issueObj) {
            const stateMap = new Map([
                ["wait", "等待处理"],
                ["fixing", "处理中"],
                ["complete", "已完成"],
            ]);

            return [...Object.entries(issueObj)]
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
                    } else {
                        return [];
                    }
                })
                .filter((item) => item.length === 2);
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
