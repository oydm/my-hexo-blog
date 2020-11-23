---
title: Vue实现后台管理权限系统及顶栏三级菜单显示功能
p: vue/admin/one
date: 2020-11-23 15:50:01
categories:
- Vue
tags:
- Vue
- HTML
- CSS
- JavaScript
- NodeJs
- admin
---

> 后台基于[vue-element-admin](https://panjiachen.gitee.io/vue-element-admin-site/zh/)简单修改，详细教程请参考 [官方文档](https://panjiachen.gitee.io/vue-element-admin-site/zh/) 加入顶栏菜单，实现点击顶栏控制侧边菜单实现三级互动，以实现多模块功能后台管理。

### 目录结构

 ``` shell
├── build                      // 构建相关  
├── config                     // 配置相关
├── src                        // 源代码
│   ├── api                    // 所有请求
│   ├── assets                 // 主题 字体等静态资源
│   ├── components             // 全局公用组件
│   ├── directive              // 全局指令
│   ├── filtres                // 全局 filter
│   ├── icons                  // 项目所有 svg icons
│   ├── lang                   // 国际化 language
│   ├── mock                   // 项目mock 模拟数据
│   ├── router                 // 路由
│   ├── store                  // 全局 store管理
│   ├── styles                 // 全局样式
│   ├── utils                  // 全局公用方法
│   ├── vendor                 // 公用vendor
│   ├── views                  // view
│   ├── App.vue                // 入口页面
│   ├── main.js                // 入口 加载组件 初始化等
│   └── permission.js          // 权限管理
├── .babelrc                   // babel-loader 配置
├── eslintrc.js                // eslint 配置项
├── .gitignore                 // git 忽略项
├── favicon.ico                // favicon图标
├── index.html                 // html模板
└── package.json               // package.json

 ```

### 系统界面实现效果

{% asset_img one-1.gif %}

### 路由Router
路由配置未做结构更改
``` javascript
export const constantRoutes = [
    {
        path: '/redirect',
        component: Layout,
        hidden: true,
        children: [
            {
                path: '/redirect/:path(.*)',
                component: () => import('@/views/redirect/index')
            }
        ]
    },
    {
        path: '/login',
        component: () => import('@/views/login/index'),
        hidden: true
    },

    {
        path: '/404',
        component: () => import('@/views/404'),
        hidden: true
    },

    {
        path: '/',
        component: Layout,
        redirect: '/dashboard',
        children: [{
            path: 'dashboard',
            name: 'Dashboard',
            component: () => import('@/views/dashboard/index'),
            meta: { title: 'Dashboard', icon: 'dashboard' }
        }]
    },

    {
        path: '/example',
        component: Layout,
        redirect: '/example/table',
        name: 'Example',
        meta: { title: 'Example', icon: 'example' },
        children: [
            {
                path: 'table',
                name: 'Table',
                component: () => import('@/views/table/index'),
                meta: { title: 'Table', icon: 'table' }
            },
            {
                path: 'tree',
                name: 'Tree',
                component: () => import('@/views/tree/index'),
                meta: { title: 'Tree', icon: 'tree' }
            }
        ]
    },

    {
        path: '/form',
        component: Layout,
        children: [
            {
                path: 'index',
                name: 'Form',
                component: () => import('@/views/form/index'),
                meta: { title: 'Form', icon: 'form' }
            }
        ]
    },

    {
        path: '/nested',
        component: Layout,
        redirect: '/nested/menu1',
        name: 'Nested',
        meta: {
            title: 'Nested',
            icon: 'nested'
        },
        children: [
            {
                path: 'menu1',
                component: () => import('@/views/nested/menu1/index'), // Parent router-view
                name: 'Menu1',
                meta: { title: 'Menu1' },
                children: [
                    {
                        path: 'menu1-1',
                        component: () => import('@/views/nested/menu1/menu1-1'),
                        name: 'Menu1-1',
                        meta: { title: 'Menu1-1' }
                    },
                    {
                        path: 'menu1-2',
                        component: () => import('@/views/nested/menu1/menu1-2'),
                        name: 'Menu1-2',
                        meta: { title: 'Menu1-2' },
                        children: [
                            {
                                path: 'menu1-2-1',
                                component: () => import('@/views/nested/menu1/menu1-2/menu1-2-1'),
                                name: 'Menu1-2-1',
                                meta: { title: 'Menu1-2-1' }
                            },
                            {
                                path: 'menu1-2-2',
                                component: () => import('@/views/nested/menu1/menu1-2/menu1-2-2'),
                                name: 'Menu1-2-2',
                                meta: { title: 'Menu1-2-2' }
                            }
                        ]
                    },
                    {
                        path: 'menu1-3',
                        component: () => import('@/views/nested/menu1/menu1-3'),
                        name: 'Menu1-3',
                        meta: { title: 'Menu1-3' }
                    }
                ]
            },
            {
                path: 'menu2',
                component: () => import('@/views/nested/menu2/index'),
                meta: { title: 'menu2' }
            }
        ]
    },
    {
        path: 'external-link',
        component: Layout,
        children: [
            {
                path: 'https://github.com/PanJiaChen/vue-element-admin',
                meta: { title: 'External Link', icon: 'link' }
            }
        ]
    },
    // 404 page must be placed at the end !!!
    { path: '*', redirect: '/404', hidden: true }
]
```
### Layout [Layout官方配置教程](https://panjiachen.gitee.io/vue-element-admin-site/zh/guide/essentials/layout.html#layout)

#### 新增Topbar组件 
layout/components/Topbar.vue 
在Navbar中引入使用
``` html
<template>
    <div class="top-nav">
        <div class="log">
            // 控制侧边栏伸缩
            <hamburger v-if="!sidebar.hide" :is-active="sidebar.opened" class="hamburger-container"
                       @toggleClick="toggleSideBar"/>
        </div>
        //顶级菜单
        <el-menu
            :active-text-color="variables.menuActiveText"
            :default-active="activeMenu"
            mode="horizontal"
            @select="handleSelect"
        >
            <div v-for="item in routes" :key="item.path" class="nav-item">
                <app-link :to="resolvePath(item)">
                    <el-menu-item
                        v-if="!item.hidden"
                        :index="item.path"
                    >{{ item.meta ? item.meta.title : item.children[0].meta.title }}
                    </el-menu-item>
                </app-link>
            </div>
        </el-menu>
        //增加刷新页面功能及右侧功能菜单
        <div class="right-menu">
            <template v-if="device!=='mobile'">
                <el-tooltip content="刷新页面" effect="dark" placement="bottom">
                    <Refresh />
                </el-tooltip>
                <search id="header-search" class="right-menu-item"/>
                <screenfull id="screenfull" class="right-menu-item hover-effect"/>
                <el-tooltip content="布局大小" effect="dark" placement="bottom">
                    <size-select id="size-select" class="right-menu-item hover-effect"/>
                </el-tooltip>
            </template>
            <el-dropdown class="avatar-container" trigger="click">
                <div class="avatar-wrapper">
                    <el-avatar :size="40" :src="avatar+'?imageView2/1/w/80/h/80'">
                    </el-avatar>
                    <i class="el-icon-caret-bottom"/>
                </div>
                <el-dropdown-menu slot="dropdown" class="user-dropdown">
                    <router-link to="/">
                        <el-dropdown-item>主页</el-dropdown-item>
                    </router-link>
                    <el-dropdown-item divided @click.native="logout">
                        <span style="display:block;">退出</span>
                    </el-dropdown-item>
                </el-dropdown-menu>
            </el-dropdown>
        </div>
    </div>
</template>
```

点击顶级菜单时 触发handleSelect事件，把选中顶级路由的子路由保存store
``` javascript
<script>
    import { mapGetters } from 'vuex'
    import AppLink from './Sidebar/Link'
    import { constantRoutes } from '@/router'
    import variables from '@/styles/variables.scss'
    import { isExternal } from '@/utils/validate'
    import Hamburger from '@/components/Hamburger'
    import Screenfull from '@/components/Screenfull'
    import SizeSelect from '@/components/SizeSelect'
    import Search from '@/components/HeaderSearch'
    import Refresh from '@/components/Refresh'

    export default {
        name: 'Topbar',
        components: {
            AppLink,
            Hamburger,
            Screenfull,
            SizeSelect,
            Search,
            Refresh
        },
        data() {
            return {
                routes: constantRoutes
            }
        },
        computed: {
            activeMenu() {
                const route = this.$route
                const { meta, path } = route
                // if set path, the sidebar will highlight the path you set
                if (meta.activeMenu) {
                    return meta.activeMenu
                }
                // 如果是首页，首页高亮
                if (path === '/dashboard') {
                    return '/'
                }
                // 如果不是首页，高亮一级菜单
                const activeMenu = '/' + path.split('/')[1]
                return activeMenu
            },
            variables() {
                return variables
            },
            sidebar() {
                return this.$store.state.app.sidebar
            },
            ...mapGetters(['sidebar',
                           'avatar',
                           'device'])
        },
        mounted() {
            this.initCurrentRoutes()
        },
        methods: {
            toggleSideBar() {
                this.$store.dispatch('app/toggleSideBar')
            },
            // 通过当前路径找到二级菜单对应项，存到store，用来渲染左侧菜单
            initCurrentRoutes() {
                const { path } = this.$route
                let route = this.routes.find(
                    item => item.path === '/' + path.split('/')[1]
                )
                // 如果找不到这个路由，说明是首页
                if (!route) {
                    route = this.routes.find(item => item.path === '/')
                }
                this.$store.commit('permission/SET_CURRENT_ROUTES', route)
                this.setSidebarHide(route)
            },
            // 判断该路由是否只有一个子项或者没有子项，如果是，则在一级菜单添加跳转路由
            isOnlyOneChild(item) {
                if (item.children && item.children.length === 1) {
                    return true
                }
                return false
            },
            resolvePath(item) {
                // 如果是个完成的url直接返回
                if (isExternal(item.path)) {
                    return item.path
                }
                // 如果是首页，就返回重定向路由
                if (item.path === '/') {
                    const path = item.redirect
                    return path
                }

                // 如果有子项，默认跳转第一个子项路由
                let path = ''
                /**
                 * item 路由子项
                 * parent 路由父项
                 */
                const getDefaultPath = (item, parent) => {
                    // 如果path是个外部链接（不建议），直接返回链接，存在个问题：如果是外部链接点击跳转后当前页内容还是上一个路由内容
                    if (isExternal(item.path)) {
                        path = item.path
                        return
                    }
                    // 第一次需要父项路由拼接，所以只是第一个传parent
                    if (parent) {
                        path += (parent.path + '/' + item.path)
                    } else {
                        path += ('/' + item.path)
                    }
                    // 如果还有子项，继续递归
                    if (item.children) {
                        getDefaultPath(item.children[0])
                    }
                }

                if (item.children) {
                    getDefaultPath(item.children[0], item)
                    return path
                }

                return item.path
            },
            handleSelect(key, keyPath) {
                // 把选中路由的子路由保存store
                const route = this.routes.find(item => item.path === key)
                this.$store.commit('permission/SET_CURRENT_ROUTES', route)
                this.setSidebarHide(route)
            },
            // 设置侧边栏的显示和隐藏
            setSidebarHide(route) {
                /*if (!route.children || route.children.length === 1) {
                    this.$store.dispatch('app/toggleSideBarHide', true)
                } else {
                    this.$store.dispatch('app/toggleSideBarHide', false)
                }*/
            },
            async logout() {
                await this.$store.dispatch('user/logout')
                this.$router.push(`/login?redirect=${this.$route.fullPath}`)
            }
        }
    }
</script>
```

### 侧边菜单修改 
侧边菜单获取当前currentRoutes 展示菜单

sidebar/index.vue
``` 
<script>
    import { mapGetters } from 'vuex'
    import Logo from './Logo'
    import SidebarItem from './SidebarItem'
    import variables from '@/styles/variables.scss'

    export default {
        components: { SidebarItem, Logo },
        computed: {
            ...mapGetters([
                'sidebar'
            ]),
            routes() {
                // 获取当前顶级菜单下子菜单，此数据通过顶级菜单事件赋值
                return this.$store.state.permission.currentRoutes.children
            },
            activeMenu() {
                const route = this.$route
                const { meta, path } = route
                // if set path, the sidebar will highlight the path you set
                if (meta.activeMenu) {
                    return meta.activeMenu
                }
                return path
            },
            showLogo() {
                return this.$store.state.settings.sidebarLogo
            },
            variables() {
                return variables
            },
            isCollapse() {
                return !this.sidebar.opened
            }
        }
    }
</script>
```

其他组件及代码未做更改
详细代码可见[github](https://github.com/oydm/vue-admin)