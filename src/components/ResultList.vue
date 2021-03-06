<template>
    <div class="result-list">
        <template v-if="resultList.length === 0">
            <div class="result-empty">
                <h3>抱歉，当前没有搜到结果哦</h3>
            </div>
        </template>
        <template v-else>
            <template v-for="(item, index) in resultList">
                <div :key="index" class="result-item">
                    <div class="title"><a :href="item.link_url">{{ item.title }}</a></div>
                    <div class="description">{{ item.description }}</div>
                    <div class="foot">
                        <div>{{ showAuthor?item.user_name:"" }} - {{ showCreatedTime ? timeFormat(item.create_time):''}}
                        </div>
                        <div v-if="showTags">
                            <div v-for="(tag,index) in item.tags" :key="index">{{ tag }}</div>
                        </div>
                    </div>
                </div>
            </template>
        </template>
    </div>
</template>

<script>
    import axios from "axios";

    export default {
        name: "ResultList",
        props: {
            searchText: {
                type: String
            },
            currentPage: {
                type: Number
            },
            numOfPage: {
                type: Number
            },
            unlimitedScroll: {
                type: Boolean
            },
            showCreatedTime: {
                type: Boolean
            },
            showTags: {
                type: Boolean
            },
            showAuthor: {
                type: Boolean
            }
        },
        data: function () {
            return {
                resultList: []
            }
        },
        computed: {},
        watch: {
            searchText: function () {
                this.search();
            },
            currentPage: function () {
                this.search();
            },
            unlimitedScroll:function () {
                if(this.unlimitedScroll){
                    this.setOnscrollEvent();
                }
            }
        },
        mounted: function () {
            this.search();
            this.setOnscrollEvent();
        },
        methods: {
            /**
             * @description 进行查询
             */
            search: function () {
                let _this = this;
                axios
                    .get("https://i.snssdk.com/search/api/study/", {
                        params: {
                            keyword: _this.searchText,
                            offset: (_this.currentPage - 1) * 10
                        }
                    })
                    .then(function (response) {
                        if (response.data.code === 0) {
                            _this.resultList = response.data.data;
                        }
                    })
                    .catch(function (error) {
                        console.log(error);
                    })
            },

            /**
             * 在现有内容上追加内容
             */
            continueSearch: function () {
                let _this = this;
                axios
                    .get("https://i.snssdk.com/search/api/study/", {
                        params: {
                            keyword: _this.searchText,
                            offset: _this.resultList.length
                        }
                    })
                    .then(function (response) {
                        console.log("continue search");
                        if (response.data.code === 0) {
                            _this.resultList = _this.resultList.concat(response.data.data);
                            _this.resultList = _this.resultList.slice(0);
                        }
                    })
                    .catch(function (error) {
                        console.log(error);
                    })
            },

            /**
             * 对时间进行格式化
             * @param stringNumber
             * @returns {string}
             */
            timeFormat: function (stringNumber) {
                let date = new Date();
                date.setTime(Number(stringNumber))
                return date.toLocaleString();
            },

            /**
             * Document Height
             *
             * @returns {number}
             */
            getDocumentTop: function () {
                var scrollTop = 0, bodyScrollTop = 0, documentScrollTop = 0;
                if (document.body) {
                    bodyScrollTop = document.body.scrollTop;
                }
                if (document.documentElement) {
                    documentScrollTop = document.documentElement.scrollTop;
                }
                scrollTop = (bodyScrollTop - documentScrollTop > 0) ? bodyScrollTop : documentScrollTop;
                return scrollTop;
            },

            /**
             * window height
             */
            getWindowHeight: function () {
                var windowHeight = 0;
                if (document.compatMode === "CSS1Compat") {
                    windowHeight = document.documentElement.clientHeight;
                } else {
                    windowHeight = document.body.clientHeight;
                }
                return windowHeight;
            },

            /**
             * Scroll Height
             */
            getScrollTop: function () {
                var scrollHeight = 0, bodyScrollHeight = 0, documentScrollHeight = 0;
                if (document.body) {
                    bodyScrollHeight = document.body.scrollHeight;
                }
                if (document.documentElement) {
                    documentScrollHeight = document.documentElement.scrollHeight;
                }
                scrollHeight = (bodyScrollHeight - documentScrollHeight > 0) ? bodyScrollHeight : documentScrollHeight;
                return scrollHeight;
            },

            /**
             * @description set the onscroll event
             */
            setOnscrollEvent: function () {
                if (this.unlimitedScroll === true) {
                    var _this = this;

                    window.onscroll = function () {
                        if(_this.unlimitedScroll === false){
                            return;
                        }

                        //htmlHeight 是网页的总高度
                        // let htmlHeight = document.body.scrollHeight || document.documentElement.scrollHeight;
                        //clientHeight是网页在浏览器中的可视高度，
                        let clientHeight = document.body.clientHeight || document.documentElement.clientHeight;
                        //scrollTop是浏览器滚动条的top位置，
                        let scrollTop = document.body.scrollTop || document.documentElement.scrollTop;

                        if (scrollTop > clientHeight / 2) {
                            _this.continueSearch();
                        }
                    }
                }
            },

        }
    }
</script>

<style lang="less" scoped>
    .result-list {
        background-color: rgb(230, 230, 230);
        padding-bottom: 7px;

        .result-item {
            margin-top: 10px;
            margin-bottom: 10px;
            padding: 10px 20px;
            background-color: white;

            .title {
                font-weight: bold;
                text-align: left;
                height: 1.5rem;
                line-height: 1.5rem;
                overflow: hidden;
                text-overflow: ellipsis;

                a {
                    color: #000000;
                    text-decoration: none;
                }
            }

            .description {
                text-align: justify;
                text-indent: 2em;
                height: 4rem;
                overflow: hidden;
                text-overflow: ellipsis;
                color: black;
                font-size: 1rem;
            }

            .foot {
                overflow: auto;

                & > div {
                    &:nth-of-type(1) {
                        float: left;
                        font-size: small;
                        color: gray;
                        line-height: 26px;
                    }

                    &:nth-of-type(2) {
                        float: right;

                        div {
                            float: right;
                            font-size: small;
                            background-color: #40bad5;
                            border-radius: 10px;
                            color: white;
                            padding: 2px 4px 4px;
                            /*border: 1px solid gray;*/
                            margin: 1px;
                        }
                    }
                }
            }
        }
    }

    .result-empty {
        width: 100vw;
        height: 100vh;
        text-align: center;
        line-height: 100vw;
        color: gray;
    }
</style>