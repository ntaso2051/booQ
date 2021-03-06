<template>
  <v-navigation-drawer
    id="app-drawer"
    v-model="drawer"
    app
    dark
    floating
    persistent
    mobile-break-point="991"
    width="260"
  >
    <v-list-item two-line>
      <v-list-item-avatar>
        <img
          :src="logo"
          style="width: 30px;"
        />
      </v-list-item-avatar>
      <v-list-item-title>
        <img
          :src="logoTitle"
          class="logo-title"
        />
      </v-list-item-title>
    </v-list-item>
    <v-divider/>
    <v-list nav>
      <v-list-item>
        <v-text-field
          class="search-input"
          label="Search..."
          color="success"
          v-model="searchString"
          v-on:keyup.enter="searchSideBar()"
        />
      </v-list-item>
      <v-list-item
        v-for="(link, i) in links"
        :key="i"
        :to="link.to"
        :active-class="color"
        exact
      >
        <v-list-item-action>
          <mdi-icon :name="link.icon" />
        </v-list-item-action>
        <v-list-item-title
          v-text="link.text"
        />
      </v-list-item>
    </v-list>
    <template v-slot:append>
      <v-flex layout>
        <v-list nav>
          <v-list-item @click="$store.commit('toggleAboutDialog')">
            <v-list-item-title class="font-weight-light">
              booQ Project {{ version }}
            </v-list-item-title>
          </v-list-item>
        </v-list>
        <v-list
          nav
          width="55"
        >
          <v-list-item
            to="/about"
            :active-class="color"
          >
            <v-list-item-action>
              <mdi-icon name="mdi-help" />
            </v-list-item-action>
            <v-list-item-title>
              about
            </v-list-item-title>
          </v-list-item>
        </v-list>
      </v-flex>
    </template>
    <v-dialog
      v-model="aboutDialogDrawer"
      max-width="400"
    >
      <v-card>
        <v-card-title
          class="headline"
        >
          booQ Project
        </v-card-title>
        <v-card-text>
          バージョン: {{ version }}<br>
          バグ報告・フィードバックは<a href="https://q.trap.jp/channels/team/SysAd/booq/feedback">#team/SysAd/booq/feedback</a>までお願いします。<br>
          GitHubリポジトリ: <a href="https://github.com/traPtitech/booQ">traPtitech/booQ</a><br>
        </v-card-text>
      </v-card>
    </v-dialog>
  </v-navigation-drawer>
</template>

<script>
// Utilities
import { mapMutations, mapState } from 'vuex'
import version from '@/version'
import MdiIcon from '../shared/MdiIcon.vue'

export default {
  name: 'SideBar',
  components: {
    MdiIcon
  },
  data () {
    return {
      logo: '/img/logo-main.svg',
      logoTitle: '/img/logo-type-white.svg',
      searchString: '',
      links: [
        {
          to: '/',
          icon: 'mdi-view-dashboard',
          text: 'Dashboard'
        },
        {
          to: '/items/equipment',
          icon: 'mdi-home-city',
          text: 'Equipment List'
        },
        {
          to: '/items/property',
          icon: 'mdi-book-open-page-variant',
          text: 'Personal Property List'
        },
        // {
        //   to: `/users`,
        //   icon: 'mdi-account',
        //   text: 'User List'
        // },
        {
          to: '/items/new',
          icon: 'mdi-plus-box',
          text: 'Register Item'
        }
      ],
      responsive: false
    }
  },
  computed: {
    ...mapState(['color']),
    drawer: {
      get () {
        return this.$store.state.drawer
      },
      set (val) {
        this.setDrawer(val)
      }
    },
    aboutDialogDrawer: {
      get () {
        return this.$store.state.aboutDialog
      },
      set (val) {
        this.$store.commit('toggleAboutDialog')
      }
    },
    items () {
      return this.$t('Layout.View.items')
    },
    sidebarOverlayGradiant () {
      return `${this.$store.state.sidebarBackgroundColor}, ${this.$store.state.sidebarBackgroundColor}`
    },
    version () {
      return version
    }
  },
  mounted () {
    this.onResponsiveInverted()
    window.addEventListener('resize', this.onResponsiveInverted)
  },
  beforeDestroy () {
    window.removeEventListener('resize', this.onResponsiveInverted)
  },
  methods: {
    ...mapMutations(['setDrawer', 'toggleDrawer']),
    onResponsiveInverted () {
      if (window.innerWidth < 991) {
        this.responsive = true
      } else {
        this.responsive = false
      }
    },
    searchSideBar () {
      this.$router.push({ path: '/items', query: { search: this.searchString } })
    }
  }
}
</script>

<style scoped>
.logo-title {
  position: relative;
  padding-top: 10px;
  padding-left: 5px;
  width: 60px;
}
</style>
