<template>
    <div class="container">
        <nav>
            <slot name="goHome" class="link"></slot> |
            <slot name="issue-query" class="link"></slot>
        </nav>

        <slot name="title"></slot>

        <div v-if="historyArr === null || !isQueried">
            <h3>查询中</h3>
        </div>
        <div v-else-if="historyArr.length === 0 && isQueried">
            <h3>没有查询到工单</h3>
        </div>
        <div>
            <ul v-for="issue in historyArr">
                <ul v-for="item in issue">
                    <li>{{ item[0] }} : {{ item[1] }}</li>
                </ul>
                <br />
            </ul>
        </div>
    </div>
</template>

<script>
import { issueRemotePath } from "../../config.js";

export default {
    name: "History",
    props: ["renderIssue"],
    data() {
        return {
            historyArr: [],
            localPath: "http://127.0.0.1:8080/api/v1/issue",
            issueRemotePath,
            isQueried: false,
        };
    },

    methods: {
        async fillIssueList() {
            if (!window.localStorage.getItem("historyIssues")) {
                this.historyArr = [];
                this.isQueried = true;
                return;
            }

            const issueIdArr = JSON.parse(
                window.localStorage.getItem("historyIssues")
            ).map((item) => item.issue[0]);

            for (const issueId of issueIdArr) {
                const issue = await this.queryIssueById(issueId);
                if (issue !== null) {
                    this.historyArr.push(issue);
                }
            }

            this.historyArr = this.historyArr.map((issue) => {
                const renderedIssue = this.renderIssue(issue);
                console.log(`renderedIssue: ${JSON.stringify(renderedIssue)}`);

                return renderedIssue;
            });

            this.isQueried = true;
        },

        async queryIssueById(id) {
            const response = await fetch(
                `${this.issueRemotePath}?issueId=${id}`
            );
            if (!response.ok) {
                return null;
            }

            const responseData = await response.json();
            return responseData.data.issue;
        },
    },

    mounted: function () {
        console.log("init execute");
        this.fillIssueList();
    },
};
</script>

<style scoped>
li {
    text-align: left;
}
</style>
