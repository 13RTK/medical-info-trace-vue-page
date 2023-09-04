<template>
    <div v-if="!isSubmitted" class="container">
        <nav>
            <slot name="admin-login" class="link"></slot> |
            <slot name="issue-query" class="link"></slot> |
            <slot name="history" class="link"></slot>
        </nav>
        <slot name="home-title"></slot>

        <div enctype="multipart/form-data" class="form">
            <br />
            <input
                type="text"
                class="poster"
                name="poster"
                v-model.lazy.trim="poster"
                placeholder="反馈人"
            />
            <br />
            <input
                type="text"
                class="desc"
                name="desc"
                v-model.lazy.trim="desc"
                placeholder="描述"
            />
            <br />

            <img v-if="imageSelected" v-bind:src="imageURL" alt="img review" />
            <label for="file">{{ imageLableText }}</label>
            <input
                type="file"
                name="image"
                id="file"
                accept=".jpg, .jpeg, .png"
                class="img"
                v-on:change.prevent="imageChange($event)"
            />
            <br />
            <br />

            <input type="button" value="上传" v-on:click.prevent="upload" />
        </div>
    </div>
    <submit-display
        v-else
        v-bind:issueObj="issueObj"
        v-on:cancel-submit="cancalSubmit"
        v-bind:componentName="componentName"
        v-on:back-home="switchComponent($event)"
    >
        <template v-slot:admin-login>
            <a href="#" class="admin-login">管理员登陆</a>
        </template>

        <template v-slot:issue-query>
            <a href="#" class="issue-query">工单查询</a>
        </template>

        <template v-slot:history>
            <a href="#" class="history">历史工单</a>
        </template>

        <template v-slot:uploaded-title>
            <h1>提交成功</h1>
        </template>
    </submit-display>
</template>

<script>
import SubmitDisplay from "./SubmitDisplay.vue";

export default {
    name: "Home",
    props: ["componentName"],
    data() {
        return {
            poster: "",
            desc: "",
            file: null,
            responseDataId: 0,
            imageLableText: "点击上传图片",
            imageName: "",
            imageSelected: false,
            imageURL: "",
            isSubmitted: false,
            issueObj: null,
        };
    },
    methods: {
        imageChange(event) {
            console.log("image called");
            this.file = event.target.files[0];

            this.imageURL = URL.createObjectURL(this.file);
            this.imageSelected = true;

            this.imageLableText = "点击更换图片";
        },

        upload() {
            console.log("upload called");

            if (
                this.poster.length === 0 ||
                this.desc.length === 0 ||
                this.file === null
            ) {
                const missingFields = [];
                if (this.poster.length === 0) {
                    missingFields.push("提交人");
                }
                if (this.desc.length === 0) {
                    missingFields.push("描述");
                }
                if (this.file === null) {
                    missingFields.push("图片");
                }

                alert(`请填入相关内容！缺失内容: ${missingFields.join(",")}`);
                return;
            }

            this.isSubmitted = true;
            this.issueObj = {
                desc: this.desc,
                poster: this.poster,
                imageURL: this.imageURL,
                file: this.file,
            };
        },

        cancalSubmit() {
            this.isSubmitted = false;
        },

        switchComponent(event) {
            const target = event.target;

            console.log(event);
            console.log(event.name);
            console.log(target);
        },
    },

    components: {
        SubmitDisplay,
    },
};
</script>

<style scoped>
.container {
    margin-top: 20px;
    font-family: sans-serif;
    margin: 0 auto;
}

.link {
    margin: 10px 100px;
}

input[type="button"] {
    appearance: none;
    border: 0.2em solid #e91e63;
    background: hsl(0 0 0/0);
    padding: 0.85em 1.5em;
    color: #e91e63;
    border-radius: 2em;
    transition: 1s;
    width: 40%;
}

input[type="button"]:hover,
input[type="button"]:focus,
input[type="button"]:active {
    background: #e91e63;
    color: #fff;
}

input[type="file"] {
    display: none;
}

label[for="file"] {
    width: 50%;
    display: grid;
    margin: 0 auto;
    margin-top: 20px;
    grid-auto-flow: column;
    grid-gap: 0.5em;
    justify-items: center;
    align-content: center;
    color: #e91e63;
    border: 0.2em solid #e91e63;
    background: hsl(0 0 0/0);
    padding: 0.5em 1.5em;
    color: #e91e63;
    border-radius: 2em;
    transition: 1s;
}

.hidden {
    display: none;
}

label[for="file"]:hover,
label[for="file"]:focus,
label[for="file"]:active {
    background: #e91e63;
    color: #fff;
}
</style>
