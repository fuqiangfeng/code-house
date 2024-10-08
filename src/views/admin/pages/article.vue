<template>
    <div class="container-fluid container-box">
        <div class="row d-none d-lg-flex">
            <div class="col-lg-6 d-flex">
                <el-dropdown v-if="!state.item.tabs.includes('setting')" class="custom mimic me-2" trigger="click">
                    <span class="el-dropdown-link d-flex align-items-center">
                        {{ state.item.sort }}
                        <i-svg name="down"></i-svg>
                    </span>
                    <template #dropdown>
                        <el-dropdown-item v-on:click="method.order('create_time desc', '最新')">最新</el-dropdown-item>
                        <el-dropdown-item v-on:click="method.order('create_time asc', '最早')">最早</el-dropdown-item>
                    </template>
                </el-dropdown>
                <div class="input-group custom-search me-1">
                    <i-svg name="search" color="rgb(var(--icon-color))" size="18px"></i-svg>
                    <input v-model="state.item.search" class="form-control custom search mimic" autocomplete="new-password" type="text" placeholder="标题 | 内容 | 备注">
                </div>
                <button v-on:click="method.refresh()" class="btn btn-auto mx-1 mimic" type="button">刷新</button>
                <button v-on:click="method.add()" v-if="!utils.in.array(state.item.tabs, ['remove','setting'])" class="btn btn-auto ms-1 mimic" type="button">添加</button>
            </div>
            <div class="col-lg-6 d-flex justify-content-end" style="z-index: -1">
                <button class="btn btn-auto h-35px mimic" disabled type="button">
                    {{ state.item.title }}
                </button>
            </div>
        </div>
        <div class="row mt-3">
            <div class="col-12">
                <el-tabs v-model="state.item.tabs" v-on:tab-change="method.change" id="tabs-area" class="circle">

                    <el-tab-pane name="all">
                        <template #label>
                            <span class="fw-bolder font-12">全部</span>
                        </template>
                        <table-article :params="state.params.all" v-model:init="state.tabs.all" v-on:refresh="method.refresh" ref="all"></table-article>
                    </el-tab-pane>

                    <el-tab-pane name="check">
                        <template #label>
                            <span class="fw-bolder font-12">审核通过</span>
                        </template>
                        <table-article :params="state.params.check" v-model:init="state.tabs.check" v-on:refresh="method.refresh" ref="check"></table-article>
                    </el-tab-pane>

                    <el-tab-pane name="audit">
                        <template #label>
                            <span class="fw-bolder font-12">等待审核</span>
                        </template>
                        <table-article :params="state.params.audit" v-model:init="state.tabs.audit" v-on:refresh="method.refresh" ref="audit"></table-article>
                    </el-tab-pane>

                    <el-tab-pane name="remove">
                        <template #label>
                            <span class="fw-bolder font-12">回收站</span>
                        </template>
                        <table-article :params="state.params.remove" v-model:init="state.tabs.remove" v-on:refresh="method.refresh" ref="remove" type="remove"></table-article>
                    </el-tab-pane>

                    <el-tab-pane name="setting">
                        <template #label>
                            <span class="fw-bolder font-12">设置</span>
                        </template>
                        <div class="row">
                            <div class="col-md-4">
                                <atom-article ref="article"></atom-article>
                            </div>
                        </div>
                    </el-tab-pane>

                </el-tabs>
            </div>
        </div>
    </div>

    <mouse-menu ref="mouse" v-bind="state.item.menu"></mouse-menu>
</template>

<script setup>
import utils from '{src}/utils/utils'
import { push } from '{src}/utils/route'
import MouseMenu from '@howdyjs/mouse-menu'
import AtomArticle from '{src}/comps/admin/atom/article.vue'
import TableArticle from '{src}/comps/admin/table/article.vue'
import { list as MenuList, config as MenuConfig } from '{src}/utils/menu'
import cache from "{src}/utils/cache.js";

const { ctx, proxy } = getCurrentInstance()
const state  = reactive({
    user: cache.get('user-info'),
    item: {
        timer : null,
        title : '文章列表',
        search: null,
        sort  : '排序',
        tabs  : 'all',
        menu: {
            ...MenuConfig,
            menuList: [{
                label: '刷新',
                icon: `<svg class="icon" viewBox="0 0 1024 1024" xmlns="http://www.w3.org/2000/svg" width="14" height="14">
                    <path fill="rgb(var(--menu-icon-color))" d="M608 928c-229.76 0-416-186.24-416-416h-0.128c0-0.416 0.128-0.768 0.128-1.184a95.904 95.904 0 1 0-191.872-1.184c0 0.384-0.128 0.768-0.128 1.184l0.032 0.384c0 0.288 0.096 0.544 0.096 0.8H0c0 282.784 229.216 512 512 512 282.016 0 510.592-227.968 511.872-509.632C1022.592 743.072 836.928 928 608 928z"></path>
                    <path fill="rgb(var(--menu-icon-color))" d="M1023.872 512H1024c0-282.784-229.216-512-512-512C230.016 0 1.408 227.968 0.128 509.632 1.408 280.96 187.072 96 416 96c229.76 0 416 186.24 416 416h0.128c0 0.416-0.128 0.768-0.128 1.184a96 96 0 0 0 96 96 95.872 95.872 0 0 0 95.872-94.816c0-0.416 0.128-0.768 0.128-1.184l-0.032-0.384c0-0.288-0.096-0.544-0.096-0.8z"></path>
                </svg>`,
                fn: () => method.refresh()
            },{
                label: '添加',
                icon: `<svg class="icon" viewBox="0 0 1024 1024" xmlns="http://www.w3.org/2000/svg" width="14" height="14">
                    <path fill="rgb(var(--menu-icon-color))" d="M512 1024C229.229714 1024 0 794.770286 0 512S229.229714 0 512 0s512 229.229714 512 512-229.229714 512-512 512z m0-928C282.258286 96 96 282.258286 96 512S282.258286 928 512 928 928 741.741714 928 512 741.741714 96 512 96z m208.018286 463.981714h-160v160.036572a48.018286 48.018286 0 0 1-96.036572 0v-160.036572H303.981714a47.981714 47.981714 0 0 1 0-95.963428h160V304.018286a48.018286 48.018286 0 0 1 96.036572 0v160h160a47.981714 47.981714 0 0 1 0 95.963428z"></path>
                </svg>`,
                fn: () => method.add(),
                hidden: () => utils.in.array(state.item.tabs, ['remove','setting'])
            }],
        },
    },
    params: {
        all: {
            order: 'top desc, id desc'
        },
        check: {
            order: 'top desc, id desc',
            where: [['audit','=',1]]
        },
        audit: {
            order: 'top desc, id desc',
            where: [['audit','=',0]]
        },
        remove: {
            order: 'top desc, id desc',
            onlyTrashed: true
        },
    },
    tabs: {
        all: false,
        remove: false,
    }
})

// 方法
const method = {
    // 设置排序方式
    order(order = 'create_time asc', sort = '排序') {
        state.item.sort = sort
        for (let item in state.params) state.params[item].order = order
        // 指定刷新
        method.refresh('all', 'check', 'audit', 'remove')
    },
    // 添加
    add: () => push({ name: 'admin-article-write' }),
    // 刷新
    refresh(...args) {
        // 允许刷新的参数
        let allow = ['all', 'check', 'audit' , 'remove', 'article']
        // 如果没有传参则刷新所有
        if (args.length === 0) args = allow
        // 如果传参则过滤不允许的参数
        else args = args.filter(item => allow.includes(item))
        // 批量刷新
        for (let item of args) proxy.$refs[item]['init']()
    },
    // 切换 tab
    change: (name) => state.tabs[name] = true
}

onMounted(async () => {

    const allow = ['all', 'check', 'audit', 'remove']

    let root = state.user?.result?.auth?.all ?? false
    if (!root) {
        for (let item of allow) state.params[item].where.push(['uid', '=', state.user?.id])
    }

    state.tabs.all = true

    // 追加鼠标右键菜单
    state.item.menu.menuList.push(...[{line: true}, ...await MenuList()])
})

watch(() => state.item.search, (val) => {

    const allow = ['all', 'check', 'audit', 'remove']

    for (let item of allow) {
        if (!utils.is.empty(val)) state.params[item].like = [
            ['title'  , `%${val}%`],
            ['remark' , `%${val}%`],
            ['content', `%${val}%`],
        ]
        else delete state.params[item].like
    }

    // 防抖 - 没变化的 500ms 后再刷新
    clearTimeout(state.item.timer)
    state.item.timer = setTimeout(() => method.refresh(...allow), globalThis.inis?.lazy_time ?? 500)
})

// 监听 html 下的鼠标右键事件
document.addEventListener('contextmenu', (event) => {
    // 阻止默认事件
    event.preventDefault()
    // 判断点击在不在 #tabs-area 区域内，在不显示右键菜单
    if (!event?.target?.closest('#tabs-area')) proxy.$refs['mouse']?.show(event.x, event.y)
})
</script>