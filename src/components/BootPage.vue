<!-- 表格分页组件 -->
<template>
    <nav class="boot-nav">
        <ul class="pagination boot-page">
            <li>
                <a href="javascript:void(0)" aria-label="Previous" @click="onFirstClick()">
                    <span aria-hidden="true">&laquo;</span>
                </a>
            </li>
            <li>
                <a href="javascript:void(0)" aria-label="Next" @click="onPrevClick()">
                    <span aria-hidden="true">‹</span>
                </a>
            </li>
            <li v-for="page in pages" :class="activeNum === $index ? 'active' : ''">
                <a href="javascript:void(0)" v-text="page" @click="onPageClick($index)"></a>
            </li>
            <li>
                <a href="javascript:void(0)" aria-label="Next" @click="onNextClick()">
                    <span aria-hidden="true">›</span>
                </a>
            </li>
            <li>
                <a href="javascript:void(0)" aria-label="Next" @click="onLastClick()">
                    <span aria-hidden="true">&raquo;</span>
                </a>
            </li>
        </ul>
        <div class="page-total">
            共 <span v-text="pageTotal"></span> 页
        </div>
    </nav>
    <select class="form-control boot-select" v-model="len">
        <option v-for="arr in lens" :value="arr" v-text="arr" :selected="$index === 0 ? true : false"></option>
    </select>
</template>

<script>
export default {
    props: {

        // 页码
        pages: {
            type: Array,
            default: function () {
                return [1]
            }
        },

        // 是否请求服务器端数据
        async: {
            type: Boolean,
            default: false
        },

        // 每页显示个数
        len: {
            type: Number,
            default: 10
        },

        // 显示个数数组
        lens: {
            type: Array,
            default: function () {
                return [10, 50, 100]
            }
        },

        // 表格数据（数组）
        data: {
            type: Array,
            default: function () {
                return []
            }
        },

        // AJAX地址
        url: {
            type: String,
            default: ''
        },

        // 显示页数
        pageLen: {
            type: Number,
            default: 5
        },

        // 总页数 
        pageTotal: {
            type: Number,
            default: 1
        },

        // 参数内容
        param: {
            type: Object,
            default: function () {
                return {}
            }
        }
    },
    data () {
        return {
            activeNum: 0
        }
    },
    methods: {

        // 点击页码刷新数据
        onPageClick (index) {
            this.activeNum = index
        },

        // 上一页
        onPrevClick () {

            // 当前页是否为当前最小页码
            if (this.activeNum > 0) {
                this.activeNum = this.activeNum - 1
            } else {
                if (this.pages[0] !== 1) {
                    let newPages = []

                    for (let i = 0; i < this.pages.length; i++) {
                        newPages[i] = this.pages[i] - 1
                    }

                    this.pages = newPages
                    this.getData()
                }
            }
        },

        // 下一页
        onNextClick () {

            // 当前页是否为当前最大页码
            if (this.activeNum < this.pages.length - 1) {
                this.activeNum = this.activeNum + 1
            } else {
                if (this.pages[this.pages.length - 1] < this.pageTotal) {
                    let newPages = []

                    for (let i = 0; i < this.pages.length; i++) {
                        newPages[i] = this.pages[i] + 1
                    }

                    this.pages = newPages

                    this.getData()
                }
            }
        },

        // 第一页
        onFirstClick () {
            if (this.pages[0] === 1) {
                this.activeNum = 0
            } else {
                let originPage = []

                for (let i = 1; i <= this.pageLen; i++) {
                    originPage.push(i)
                }

                this.pages = originPage
                this.activeNum = 0
            }
        },

        // 最后一页
        onLastClick () {
            if (this.pageTotal <= this.pageLen) {
                this.activeNum = this.pages.length - 1
            } else {
                let lastPage = []

                for (let i = this.pageLen - 1; i >= 0; i--) {
                    lastPage.push(this.pageTotal - i)
                }

                this.pages = lastPage
                this.activeNum = this.pages.length - 1
            }
        },

        // 获取页码
        getPages () {
            this.pages = []

            if (!this.async) {
                this.pageTotal = Math.ceil(this.data.length / this.len)
            }

            // 比较总页码和显示页数
            if (this.pageTotal <= this.pageLen) {
                for (let i = 1; i <= this.pageTotal; i++) {
                    this.pages.push(i)
                }
            } else {
                for (let i = 1; i <= this.pageLen; i++) {
                    this.pages.push(i)
                }
            }
        },

        // 页码变化获取数据
        getData () {
            if (!this.async) {
                let len = this.len,
                    pageNum = this.pages[this.activeNum] - 1,
                    newData = [];

                for (let i = pageNum * len; i < (pageNum * len + len); i++) {
                    this.data[i] !== undefined ? newData.push(this.data[i]) : ''
                }
                
                this.$dispatch('data', newData)
            } else {
                this.param.active = this.pages[this.activeNum]
                this.param.len = this.len

                this.$http({
                    url: this.url, 
                    method: 'POST',
                    data: this.param
                })
                .then(function (response) {
                    this.pageTotal = response.data.page_num

                    if (this.pages.length !== this.pageLen || this.pageTotal < this.pageLen) {
                        this.getPages()
                    }

                    if (!response.data.data.length) {
                        this.activeNum = this.pageTotal - 1
                    }

                    this.$dispatch('data', response.data)
                })
            }
        },

        // 刷新表格
        refresh () {
            this.getData()
        },

        // 重置并刷新表格
        refresh2 () {
            this.pages = [1]
                        
            if (this.activeNum === 0) {
                this.getData()
            } else {
                this.activeNum = 0
            }
        }
    },
    ready () {
        if (!this.async) {
            this.getPages()
        } 

        this.getData()
    },
    watch: {

        // 监听显示数量
        'len' (newVal, oldVal) {
            if (!this.async) {
                this.getPages()

                if (this.activeNum + 1 > this.pages.length) {
                    this.activeNum = this.pages.length - 1
                }

                this.getData()
            } else {
                this.refresh2()
            }
        },

        // 监测当前页变化
        'activeNum' (newVal, oldVal) {
            this.getData()
        }
    },
    events: {
        'refresh::page' () {
            this.refresh()
        }
    }
}
</script>

<style scoped>
.boot-select {
    float: right;
    width: 80px;
}

.boot-nav {
    float: right;
}

.boot-page {
    display: inline-block;
    margin: 2px 10px 0 20px;
    vertical-align: middle;
}

.page-total {
    display: inline-block;
    vertical-align: middle;
}
</style>