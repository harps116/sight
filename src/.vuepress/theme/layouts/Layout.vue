<template>
  <div class="flex flex-col font-sans h-screen" :class="theme">
    <div
      class="theme-container pt-20 bg-main-background text-primary"
      :class="pageClasses"
      @touchstart="onTouchStart"
      @touchend="onTouchEnd"
    >
      <Navbar
        v-if="shouldShowNavbar"
        :theme="theme"
        @themeChanged="updateTheme"
        @toggle-sidebar="toggleSidebar"
      />

      <div class="sidebar-mask" @click="toggleSidebar(false)"></div>

      <Sidebar :items="sidebarItems" @toggle-sidebar="toggleSidebar">
        <slot slot="top" name="sidebar-top" />
        <slot slot="bottom" name="sidebar-bottom" />
      </Sidebar>

      <main class="main">
        <Home v-if="$page.frontmatter.home" />
        <BlogPage v-else-if="$page.frontmatter.blogpage" />
        <TagsPage v-else-if="$page.frontmatter.tagspage" />
        <Page v-else :sidebar-items="sidebarItems">
          <slot slot="top" name="page-top" />
          <slot slot="bottom" name="page-bottom" />
        </Page>
      </main>
    </div>
  </div>
</template>

<script>
import Home from '@theme/components/Home.vue';
import BlogPage from '@theme/pages/blog/BlogPage.vue';
import TagsPage from '@theme/pages/tags/TagsPage.vue';
import Navbar from '@theme/components/Navbar.vue';
import Page from '@theme/components/Page.vue';
import Sidebar from '@theme/components/Sidebar.vue';
import { resolveSidebarItems } from '../util';

export default {
  components: { Home, BlogPage, Page, Sidebar, TagsPage, Navbar },

  data() {
    return {
      isSidebarOpen: false,
      theme: '',
      footer: 'footer text',
    };
  },

  computed: {
    shouldShowNavbar() {
      const { themeConfig } = this.$site;
      const { frontmatter } = this.$page;
      if (frontmatter.navbar === false || themeConfig.navbar === false) {
        return false;
      }
      return (
        this.$title ||
        themeConfig.logo ||
        themeConfig.repo ||
        themeConfig.nav ||
        this.$themeLocaleConfig.nav
      );
    },

    shouldShowSidebar() {
      const { frontmatter } = this.$page;
      return (
        !frontmatter.home &&
        frontmatter.sidebar !== false &&
        this.sidebarItems.length
      );
    },

    sidebarItems() {
      return resolveSidebarItems(
        this.$page,
        this.$page.regularPath,
        this.$site,
        this.$localePath
      );
    },

    pageClasses() {
      const userPageClass = this.$page.frontmatter.pageClass;
      return [
        {
          'no-navbar': !this.shouldShowNavbar,
          'sidebar-open': this.isSidebarOpen,
          'no-sidebar': !this.shouldShowSidebar,
        },
        userPageClass,
      ];
    },
  },

  mounted() {
    this.theme = localStorage.getItem('theme') || 'theme-light';
    this.$router.afterEach(() => {
      this.isSidebarOpen = false;
    });
  },

  methods: {
    toggleSidebar(to) {
      this.isSidebarOpen = typeof to === 'boolean' ? to : !this.isSidebarOpen;
    },
    updateTheme(theme) {
      this.theme = theme;
    },
    // side swipe
    onTouchStart(e) {
      this.touchStart = {
        x: e.changedTouches[0].clientX,
        y: e.changedTouches[0].clientY,
      };
    },

    onTouchEnd(e) {
      const dx = e.changedTouches[0].clientX - this.touchStart.x;
      const dy = e.changedTouches[0].clientY - this.touchStart.y;
      if (Math.abs(dx) > Math.abs(dy) && Math.abs(dx) > 40) {
        if (dx > 0 && this.touchStart.x <= 80) {
          this.toggleSidebar(true);
        } else {
          this.toggleSidebar(false);
        }
      }
    },
  },
};
</script>

<style lang="stylus">
html
 background-color: var(--bg-color) !important
</style>
