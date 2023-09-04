<template>
    <div class="check" v-if="!isComfirmed">
        <h1>确认提交？</h1>
        <label>反馈人：{{ issueObj.poster }} </label>
        <br />

        <label>问题描述：{{ issueObj.desc }} </label>

        <br />
        <label>图片：</label>
        <br />
        <img v-bind:src="issueObj.imageURL" alt="" />

        <br />
        <button type="button" class="btn" v-on:click.prevent="confirm">
            确认
        </button>
        <button type="button" class="btn" v-on:click.prevent="cancel">
            取消
        </button>
    </div>

    <div v-else class="response">
        <nav>
            <a href="#" class="link">回到主页</a>
            |
            <slot name="issue-query" class="link"></slot>
            |
            <slot name="history" class="link"></slot>
        </nav>
        <slot name="uploaded-title"></slot>
        <h2>您的工单号为：{{ responseDataId }}</h2>
    </div>
</template>

<script>
export default {
    name: "SubmitDisplay",
    data() {
        return {
            localPath: "http://127.0.0.1:8080/api/v1/issue",
            remotePath: "http://116.62.152.170:8080/api/v1/issue",
            isComfirmed: false,
            responseDataId: 0,
            responseList: [],
        };
    },
    props: {
        issueObj: {
            type: Object,
        },
        componentName: {
            type: String,
        },
    },
    emits: ["cancel-submit"],

    methods: {
        cancel() {
            console.log("cancel called");
            this.$emit("cancel-submit");
        },

        async confirm() {
            console.log("confirm called");

            // Build formData
            const { file, poster, desc } = this.issueObj;

            const formData = new FormData();
            const extname = file.name.split(".")[1];
            formData.append("poster", poster);
            formData.append("desc", desc);
            formData.append(
                "image",
                file,
                `${this.poster}-${this.desc}.${extname}`
            );

            // Post data
            // try {
            const response = await fetch(this.remotePath, {
                method: "POST",
                body: formData,
            });
            if (!response.ok) {
                alert("上传失败！");
                return false;
            }

            const responseData = await response.json();

            // console.log(responseData);
            // console.log(responseData.data.issue);
            // console.log(responseData.status);
            // Upload failed
            if (responseData.status !== "File uploaded") {
                alert("上传失败！");
                return false;
            }

            // Upload success
            this.responseDataId = responseData.data.issue[0];

            // Save to local storage
            this.saveIssueHistory(responseData);

            // Set isComfirmed
            this.isComfirmed = true;
            // } catch (error) {
            //     console.log(error);
            //     return false;
            // }
        },

        saveIssueHistory(responseData) {
            if (!window.localStorage.getItem("historyIssues")) {
                window.localStorage.setItem(
                    "historyIssues",
                    JSON.stringify([responseData.data])
                );
            } else {
                const oldIssuesArr = JSON.parse(
                    window.localStorage.getItem("historyIssues")
                );

                window.localStorage.setItem(
                    "historyIssues",
                    JSON.stringify([...oldIssuesArr, responseData.data])
                );
            }
        },

        // linkJump() {
        //     this.$emit("link-jump");
        // },
    },
};
</script>

<style scoped>
button {
    margin-left: 5px;
    margin-right: 5px;
}
</style>
