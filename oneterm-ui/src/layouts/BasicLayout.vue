<template>
  <a-layout :class="['layout', device]">
    <!-- SideMenu -->
    <a-drawer
      v-if="isMobile()"
      placement="left"
      :wrapClassName="`drawer-sider ${navTheme}`"
      :closable="false"
      :visible="collapsed"
      @close="drawerClose"
    >
      <side-menu
        mode="inline"
        :menus="sideBarMenu"
        :theme="navTheme"
        :collapsed="false"
        :collapsible="true"
        @menuSelect="menuSelect"
      ></side-menu>
    </a-drawer>

    <side-menu
      v-else-if="isSideMenu()"
      mode="inline"
      :menus="sideBarMenu"
      theme="light"
      :collapsed="collapsed"
      :collapsible="true"
    ></side-menu>

    <a-layout
      :class="[layoutMode, `content-width-${contentWidth}`]"
      :style="{ paddingLeft: contentPaddingLeft, minHeight: '100vh' }"
    >
      <!-- layout header -->
      <global-header
        :mode="layoutMode"
        :menus="sideBarMenu"
        :theme="navTheme"
        :collapsed="collapsed"
        :device="device"
        @toggle="toggle"
      />

      <!-- layout content -->
      <a-layout-content :style="{ height: '100%', paddingBottom: '24px', paddingTop: fixedHeader ? '64px' : '0' }">
        <multi-tab v-if="multiTab"></multi-tab>
        <transition name="page-transition">
          <router-view v-if="alive" />
        </transition>
      </a-layout-content>
    </a-layout>
  </a-layout>
</template>

<script>
import { triggerWindowResizeEvent } from '@/utils/util'
import { mapState, mapActions } from 'vuex'
import { mixin, mixinDevice } from '@/utils/mixin'
import config from '@/config/setting'

import RouteView from './RouteView'
import MultiTab from '@/components/MultiTab'
import SideMenu from '@/components/Menu/SideMenu'
import GlobalHeader from '@/components/GlobalHeader'

export default {
  name: 'BasicLayout',
  mixins: [mixin, mixinDevice],
  components: {
    RouteView,
    MultiTab,
    SideMenu,
    GlobalHeader,
  },
  data() {
    return {
      production: config.production,
      collapsed: false,
      alive: true,
    }
  },
  computed: {
    ...mapState({
      // dynamic main route
      mainMenu: (state) => state.routes.appRoutes,
    }),
    contentPaddingLeft() {
      if (!this.fixSidebar || this.isMobile()) {
        return '0'
      }
      if (this.sidebarOpened) {
        return '220px'
      }
      return '80px'
    },
    sideBarMenu() {
      const sideMenus = this.mainMenu.filter((item) => this.$route.path.startsWith(item.path))
      if (sideMenus.length > 1) {
        return sideMenus.find((item) => item.path !== '/').children
      } else {
        return sideMenus[0].children
      }
    },
  },
  provide() {
    return {
      reloadBoard: this.reload,
      collapsed: () => {
        return this.collapsed
      },
    }
  },
  watch: {
    sidebarOpened(val) {
      this.collapsed = !val
    },
  },
  created() {
    this.setMenuData()
    this.collapsed = !this.sidebarOpened
  },
  mounted() {
    const userAgent = navigator.userAgent
    if (userAgent.indexOf('Edge') > -1) {
      this.$nextTick(() => {
        this.collapsed = !this.collapsed
        setTimeout(() => {
          this.collapsed = !this.collapsed
        }, 16)
      })
    }
  },
  methods: {
    ...mapActions(['setSidebar']),
    toggle() {
      this.collapsed = !this.collapsed
      this.setSidebar(!this.collapsed)
      triggerWindowResizeEvent()
    },
    setMenuData() {},
    reload() {
      this.alive = false
      this.$nextTick(() => {
        this.alive = true
      })
    },
    menuSelect() {
      if (!this.isDesktop()) {
        this.collapsed = false
      }
    },
    drawerClose() {
      this.collapsed = false
    },
  },
}
</script>

<style lang="less">
@import url('../style/global.less');

/*
 * The following styles are auto-applied to elements with
 * transition="page-transition" when their visibility is toggled
 * by Vue.js.
 *
 * You can easily play with the page transition by editing
 * these styles.
 */

.page-transition-enter {
  opacity: 0;
}

.page-transition-leave-active {
  opacity: 0;
}

.page-transition-enter .page-transition-container,
.page-transition-leave-active .page-transition-container {
  -webkit-transform: scale(1.1);
  transform: scale(1.1);
}
</style>
