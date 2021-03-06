<template lang="html">
  <div>
    <div class="about-content page-container">
      <aside class="content-side-bar">
        <a v-for="(nav, navIndex) in navBarTitles"
           :class="{ current: navIndex === currentCategory.index }"
           @click="navSwitch(navIndex, nav)"
           v-if="navBarTitles"
           :key="nav.id"><i class="iconfont icon-triangle-arrow-r"></i>{{ nav.title }}</a>
      </aside>
      <div class="news-container" v-loading="isLoading">
        <div class="figure-container" v-if="pageItems.length">
          <div v-for="news in pageItems"
               class="news-figure"
               :key="news.id">
            <h3 class="news-title"><router-link :to="news.path" tag="a">{{ news.title }}</router-link></h3>
            <p class="news-info clearfix">
              <span class="news-date">{{ news.created_time | formatDate }}</span>
              <span class="news-read"><i class="iconfont icon-eye"></i><em class="views-time">{{ news.ctr }}</em></span>
            </p>
            <div class="news-img" v-if="news.imgUrl">
              <img :src="news.imgUrl" alt="">
            </div>
            <p class="simple-brief">{{ news.preview }}</p>
          </div>
        </div>
        <div v-else
             style="text-align:center;color: #6e6e6e;"
             class="figure-container">
          暂无文章
        </div>
      </div>
      <div style="text-align:center;font-size:18px;margin-top:15px;" v-if="pageItems.length">
        <el-pagination
          background
          layout="prev, pager, next"
          :current-page.sync="currentPage"
          :total="total"
          :page-size="pageSize"
          @current-change="getPageData">
        </el-pagination>
      </div>
    </div>
  </div>
</template>

<script>
import { dateFormatter } from '@utils/index'
import { mapState, mapMutations } from 'vuex'
import { getAcademyData } from '@api/index'
export default {
  data () {
    return {
      isLoading: false,
      title: '',
      navBarTitles: [],
      pageItems: [],
      sectionMenuItem: null,
      currentPage: 1,
      total: 0,
      pageSize: 10
    }
  },
  watch: {
    '$route': 'onRouteChange'
  },
  computed: {
    ...mapState([
      'currentCategory',
      'headerMenuItem'
    ])
  },
  mounted () {
    // 利用section 和category来锁定当前的菜单
    this.section = this.$route.name
    this.category = this.$route.params.category
    let sectionMenuItem = this.headerMenuItem[this.headerMenuItem.findIndex(item => item.name === this.section)]
    this.title = sectionMenuItem.title
    this.navBarTitles = sectionMenuItem.subMenuItem
    this.sectionMenuItem = sectionMenuItem
    this.getPageData()
  },
  methods: {
    getPageData () {
      // 获取页面数据
      let params = {
        category: this.category,
        page: this.currentPage
      }

      return Promise
        .resolve()
        .then(_ => {
          this.isLoading = true
          return getAcademyData(this.section, params)
        })
        .then(res => {
          console.log(res)
          if (res.status === 200) {
            let data = res.data
            this.pageItems = data.results.map(item => this.processData(item))
            console.log(this.pageItems)
            this.total = data.count
            this.pageSize = this.total < 10 ? this.total : 10
          }
          this.isLoading = false
        })
        .catch(error => console.log(JSON.stringify(error)))
    },
    processData (item) {
      return {
        id: item.id || null,
        preview: item.preview || null,
        title: item.title || null,
        imageUrl: item.image_url || null,
        created_time: item.created_time || null,
        updated_time: item.updated_time || null,
        ctr: item.ctr || null, // 请求次数
        path: `/${this.section}/${this.category}/${item.id}` || '/'
      }
    },
    navSwitch (index, nav) {
      // Swtich Data
      let category = {
        index,
        title: nav.title,
        path: nav.path
      }
      this.switchCategory(category)
      this.$router.push({ path: nav.path })
    },
    // 当路由发生变化
    onRouteChange () {
      this.category = this.$route.params.category
      this.currentPage = 1
      this.getPageData()
    },
    ...mapMutations([
      'showImagePage',
      'switchCategory'
    ])
  },
  filters: {
    formatDate: date => { // 时间转变格式
      return dateFormatter(date, 'yyyy-M-d')
    }
  }
}
</script>

<style lang="scss" scoped>

/* External CSS */
</style>
