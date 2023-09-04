<template>
    <keep-alive>
        <component
            v-bind:is="componentName"
            v-bind:componentName="componentName"
        >
            <template v-slot:admin-login>
                <a href="#" class="admin-login" v-on:click="gotoLogin"
                    >管理员登陆</a
                >
            </template>

            <template v-slot:issue-query>
                <a href="#" class="issue-query" v-on:click="gotoQuery"
                    >工单查询</a
                >
            </template>

            <template v-slot:history>
                <a href="#" class="history" v-on:click="gotoHistory"
                    >历史工单</a
                >
            </template>

            <template v-slot:home-title>
                <h1>问题反馈</h1>
            </template>
        </component>
    </keep-alive>
</template>

<script>
import Home from "@/components/Home.vue";
import History from "@/components/History.vue";
import Query from "@/components/Query.vue";

export default {
    name: "App",
    data() {
        return {
            componentName: "Home",
        };
    },

    components: {
        Home,
        History,
        Query,
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
</style>
