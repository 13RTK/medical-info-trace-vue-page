<template>
    <div v-if="!isQueried" class="container">
        <nav>
            <slot name="goHome" class="link"></slot> |
            <slot name="history" class="link"></slot>
        </nav>
        <slot name="title"></slot>
        <div class="query-div" v-if="!isQuering">
            <input
                type="text"
                v-model.lazy.trim="queryId"
                placeholder="请输入工单ID"
            />
            <br />
            <button @click.prevent="queryIssue">查询</button>
        </div>
        <div v-else>
            <h2>正在查询。。。</h2>
        </div>
    </div>

    <div v-else>
        <nav @click="resetQuery">
            <slot name="goHome" class="link"></slot>
            |
            <a href="#" class="issue-query">工单查询</a>
            |
            <slot name="history" class="link"></slot>
        </nav>

        <ul v-for="item in resultIssueArr">
            <li>{{ item[0] }} : {{ item[1] }}</li>
        </ul>
    </div>
</template>

<script>
import { issueRemotePath } from "../../config.js";

export default {
    name: "Query",
    props: ["renderIssue", "staffList"],
    data() {
        return {
            queryId: "",
            localPath: "http://127.0.0.1:8080/api/v1/issue",
            issueRemotePath,
            isQueried: false,
            isQuering: false,

            queryIssueObj: null,
            resultIssueArr: [],
        };
    },

    methods: {
        async queryIssue() {
            this.isQuering = true;
            if (this.queryId === "") {
                alert("请输入正确的工单号！!");
                this.isQuering = false;
                return;
            }

            const response = await fetch(
                `${this.issueRemotePath}?issueId=${this.queryId}`
            );
            if (!response.ok) {
                alert("查询失败，请输入正确的工单号！!");
                this.isQuering = false;
                return;
            }
            const responseData = await response.json();

            this.isQueried = true;
            this.queryIssueObj = responseData.data.issue;
            this.resultIssueArr = this.renderIssue(this.queryIssueObj);
        },

        resetQuery() {
            console.log("reset query called");
            this.isQueried = false;
            this.queryId = "";
            this.isQuering = false;
        },
    },
};
</script>

<style scoped>
li {
    font-size: 15px;
    text-align: left;
}
</style>
