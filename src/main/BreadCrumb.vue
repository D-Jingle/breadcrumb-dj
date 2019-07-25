<template>
  <div class="breadContainer">
    <el-breadcrumb separator="/">
      <el-breadcrumb-item :to="item.router" v-for="item of breadcrumb" :key="item.path">{{item.name}}</el-breadcrumb-item>
    </el-breadcrumb>
  </div>
</template>

<script>
  export default {
    name: 'BreadCrumb',
    data () {
      return {
        allRoutes: [],
        currentRouter: [],
        parameters: {},
        breadcrumb: []
      }
    },
    methods: {
      getBreadcrumb () {
        this.allRoutesFormat()
        this.currentRouteFormat()
        this.matchRouter()
      },
      allRoutesFormat () {
        let routes = this.$router.options.routes
        this.allRoutes = routes.map(route => {
          return {
            name: route.name,
            pathArr: route.path.substring(1).split('/')
          }
        })
      },
      currentRouteFormat () {
        this.currentRouter = this.$route.fullPath.substring(1).split('/')
      },
      matchRouter () {
        // 匹配路由
        let currentRouterArr = this.allRoutes.filter((routeItem) => {
          if (routeItem.pathArr.length <= this.currentRouter.length) {
            let flag = true
            for (let i = 0; i < routeItem.pathArr.length; i++) {
              if (routeItem.pathArr[i].slice(0, 1) === ':') {
                this.parameters[routeItem.pathArr[i].slice(1)] = this.currentRouter[i]
              } else {
                if (routeItem.pathArr[i] !== this.currentRouter[i]) {
                  flag = false
                }
              }
            }
            return flag
          } else {
            return false
          }
        })
        if (currentRouterArr[0].pathArr[0] !== '') currentRouterArr.unshift(this.allRoutes[0])
        this.routerReformat(currentRouterArr)
      },
      routerReformat (currentRouterArr) {
        this.breadcrumb = []
        currentRouterArr.map(item => {
          let str = ''
          item.pathArr.forEach((path, index) => {
            if (path.slice(0, 1) === ':') {
              // 所有 name 只匹配一个参数
              if (index + 1 === item.pathArr.length) {
                item.name += `（${this.parameters[path.slice(1)]}）`
              }
              str += '/' + this.parameters[path.slice(1)]
            } else {
              str += '/' + path
            }
          })
          this.breadcrumb.push({
            name: item.name,
            router: str
          })
        })
        this.breadcrumb.sort(this.baseSort)
      },
      baseSort (router1, router2) {
        return router1.router.length - router2.router.length
      }
    },
    watch: {
      $route () {
        this.getBreadcrumb()
      }
    },
    mounted () {
      this.getBreadcrumb()
    }
  }
</script>

<style scoped>

  .breadContainer{
    margin-left: 1rem;
    margin-top: 1rem;
  }

</style>
