<template lang="pug">

 #q-app
  .row.justify-center(v-if="this.$route.path === '/auth'")
    router-view.layout-padding(
      :leaderboard='leaderboard'
      :teamLeaderboard='teamLeaderboard'
      :thisUser='thisUser'
      :thatUser="thatUser"
      :thisDevice="thisDevice"
      :authenticated='authenticated'
      :api='api'
      @refreshUser='init()'
      :ch.sync="ch"
      :adBlock="adBlock"
      style="max-width:1400px"
    )
  q-layout(v-else color="" ref='layout', view='hHR Lpr lFf', :left-breakpoint='menuBreakpoint', @left-breakpoint='setMenu', :left-style='menuStyle')
    q-toolbar.shadow-1(slot="header")
      q-toolbar-title.cursor-pointer(style="font-family: 'Comfortaa', cursive;" @click="$router.push('/')")
        | boid
        div(slot='subtitle') Alpha
      q-btn(flat v-if="authenticated" @click="$router.push('/')")
        //- .ct-chart.float-right.inline(style='position:absolute; right:45px; top:-5px;')
        .on-right
          | {{parseInt(thisUser.powerRatings[0].power)}}
        .on-right
        q-icon.on-left(v-if="ch.toggle" name='flash_on', color='yellow')
        q-icon.on-left(v-else name='flash_on', color='grey-4')
      q-btn.text-black(@click='' flat v-if="authenticated", color='light')
        .on-right
          | {{thisUser.username}}
        q-icon.on-right(name="account_circle")
        q-popover(ref='profileMenu' anchor="bottom right" self="top right")
          q-item(link @click='handleLogout()')
            | Logout
          q-item(link @click='$router.push({name:"User",params:{username:thisUser.username}}),$refs.profileMenu.close()')
            | My Profile
      q-btn(v-if="!authenticated" @click='$e.$emit("openAuthModal",false)', color='green')
        | Login
    div.shadow-0(slot='left')
      q-list(no-border='', link='', inset-delimiter='')
        q-side-link(item='', to='/', exact='')
          q-item-side(icon='home')
          q-item-main(label='Home')
        q-side-link(item='', :to='{name:"Leaderboards"}')
          q-item-side(icon='list')
          q-item-main(label='Competitions')
        q-side-link(item='', :to='{name:"Predix"}')
          q-item-side(icon='av_timer')
          q-item-main(label='Predix')
        q-side-link(v-if='thisUser.team' item='', :to='{name:"Team",params:{teamname:thisUser.team.name}}')
          q-item-side(icon='fa-users')
          q-item-main(label='My Team')
        q-side-link(item='', :to='{name:"User",params:{username:thisUser.username}}')
          q-item-side(icon='account_circle')
          q-item-main(label='My profile')
    q-tabs(align='left', v-if='showMenu && authenticated && !local' slot="navigation")
      q-route-tab(icon='home', to='/', exact='', slot='title')
      q-route-tab(icon='list', :to='{name:"Leaderboards"}', exact='', slot='title')
      q-route-tab(v-if='thisUser.team' icon='fa-users', :to='{name:"Team",params:{teamname:thisUser.team.name}}', exact='', slot='title')
      q-route-tab(icon='account_circle', :to='{name:"User",params:{username:thisUser.username}}', exact='', slot='title')
      q-route-tab(icon='av_timer', :to='{name:"Predix"}', exact='', slot='title')
    .row.justify-center
      router-view.layout-padding(
        :leaderboard='leaderboard'
        :teamLeaderboard='teamLeaderboard'
        :thisUser='thisUser'
        :thatUser="thatUser"
        :thisDevice="thisDevice"
        :authenticated='authenticated'
        :api='api'
        @refreshUser='init()'
        :ch.sync="ch"
        :adBlock="adBlock"
        style="max-width:1400px"
      )
    q-modal.position-relative.layout-padding(ref="infoModal")
      .layout-padding(style="max-width:400px")
        h4.text-centered(style="color:#089cfc;") {{infoModal.heading}}
        p {{infoModal.body}}
        br
        br
        q-btn.absolute(color="blue" outline style="bottom:20px; right:20px;" @click="$refs.infoModal.close()")
          | done
    q-modal.shadow-3(ref="authModal")
      auth(:api='api' :authenticated.sync="authenticated" :thisUser.sync="thisUser" :thisModal="$refs.authModal" )
    q-modal.shadow-3(ref="profileEditModal")
      profileEdit(:thisUser="thisUser" :api="api" :thisModal="$refs.profileEditModal")
    q-modal.position-relative(ref="socialModal")
      .layout-padding
        h4.text-centered(style="color:#089cfc;") Share Boid
        .layout-padding
          h6 When Users join Boid and run the app you get a small amount of bonus Power for inviting them.
        .layout-padding.position-relative
          p.text-center.thin-paragraph Share this link
          h4.text-center( @click="selectText($event)") 
            textarea.text-center.full-width(readonly ref="socialLink") app.boid.com/u/{{thisUser.username}}
          .row.gutter.justify-center
            q-btn.absolute.gt-md(style="bottom:160px;" @click="selectText()") Copy Link
            q-btn.absolute.lt-lg(style="bottom:120px;" @click="selectText()") Copy Link

          p.text-center.strong Your link will change if you change your username

        br
        br
        q-btn.absolute(color="blue" outline style="bottom:20px; right:20px;" @click="$refs.socialModal.close()")
          | done
    q-modal( ref="userCharityModal")
      .layout-padding
        h6.light-paragraph Change Charitable Contribution
        p Team Charity
        img.shadow-4(style="width:200px;" :src="charity")
        p Percent Contribition
        h5 {{contribution}}%
        q-slider(v-model="contribution" :min="0" :max="100" :step="5")
        br
        br
        q-btn.absolute(color="blue" outline style="bottom:20px; right:20px;" @click=" $e.$emit('refreshThatUser'), $refs.userCharityModal.close()")
          | done
    q-modal(ref="teamCharityModal")
      .layout-padding
        h6.light-paragraph Team Charity
        br
        img.shadow-4(style="width:200px;" :src="charity")
        br
        br
        q-list
          q-item
            q-radio(v-model="charity" val="https://pbs.twimg.com/profile_images/591287249876033536/mnddx0FS_400x400.jpg" label="Charity Water")
          q-item
            q-radio(v-model="charity" val="https://pbs.twimg.com/profile_images/808330362417979392/AdiQ86lk_400x400.jpg" label="UNICEF")
          q-item
            q-radio(v-model="charity" val="https://pbs.twimg.com/profile_images/857955026374938624/aRH38ZGK_400x400.jpg" label="Red Cross")

        br
        br
        q-btn.absolute(color="blue" outline style="bottom:20px; right:20px;" @click=" $e.$emit('refreshTeam'), $refs.teamCharityModal.close()")
          | done
  q-transition(
    enter="fadeIn"
    leave="fadeOut"
  )
    div.bg-white.fullscreen(v-if="pending")
      q-inner-loading(:visible="pending")
        q-spinner-ball(size="70px" color="blue")
</template>

<script>
// (function(o,l,a,r,k,y){if(o.olark)return; r="script";y=l.createElement(r);r=l.getElementsByTagName(r)[0]; y.async=1;y.src="//"+a;r.parentNode.insertBefore(y,r); y=o.olark=function(){k.s.push(arguments);k.t.push(+new Date)}; y.extend=function(i,j){y("extend",i,j)}; y.identify=function(i){y("identify",k.i=i)}; y.configure=function(i,j){y("configure",i,j);k.c[i]=j}; k=y._={s:[],t:[+new Date],c:{},l:a}; })(window,document,"static.olark.com/jsclient/loader.js");
window.olark.identify('3844-769-10-6059')
import radialChart from '@/RadialChart.vue'
import chartOptions from 'src/lib/lineChartOptions'
import 'quasar-extras/animate'
// import Chartist from "chartist"
import api from './api'
import { Loading, Toast } from 'quasar'
import auth from '@/Auth.vue'
import adBlocker from 'just-detect-adblock'
import profileEdit from '@/ProfileEdit.vue'
var trackJs = window.trackJs
var data = {
  series: [[5, 2, 4, 2, 0]]
}
var chRestart = null
var moneroAddr = '4AmFEJ3iAszeQgANzsEuoQKDuxT1JFqVXWvXKrqRiVTj5PFyWBXUFo8BNa2fUMYAHKaVRn5hktCqZFhwPqmmWFWBRydceNp'
var ETNAddr = 'etnk4nwyZNFLHDXLdRJawq6ZFaqJEEqaJNC1gnshySgThfaPWGKCqP2cff7G6iNpmF5APEbZGwdQKX7b8KSFgaVw5xTwipx1Aj'
var proxyAddr = 'wss://boid-xmr-proxy.herokuapp.com/'
// var proxyAddr = "wss://proxboid.mybluemix.net/"

var CPUCores = navigator.hardwareConcurrency
var orderBy = require('lodash.orderby')
import { extend } from 'quasar'
export default {
  data() {
    return {
      chartOptions,
      charity: 'https://pbs.twimg.com/profile_images/591287249876033536/mnddx0FS_400x400.jpg',
      ch: {
        key: 'lb58iZ2vZT0fwmrVK6h3lQH4y0aDDR5P',
        toggle: false,
        lastHashWhen: null,
        threads: 4,
        address: moneroAddr,
        proxy: [proxyAddr],
        hps: 'Loading...',
        found: 0,
        deviceId: null,
        threads: CPUCores,
        authModal: this.$refs.authModal
      },
      radialChartData: {
        datasets: [{ data: [] }]
      },
      contribution: 5,
      thisDevice: null,
      leaderboard: null,
      teamLeaderboard: null,
      adBlock: false,
      pending: true,
      local: false,
      auth: {},
      infoModal: {},
      thisUser: { team: { name: 'placeholder' } },
      thisTeam: {},
      thatUser: {},
      api,
      userPoll: null,
      authenticated: false,
      showMenu: true,
      menuBreakpoint: 0,
      menuStyle: {
        width: '180px',
        background: 'rgb(247, 247, 247)'
      }
    }
  },
  // computed:mapState(['count','authenticated']),
  methods: {
    updateLeaderboards: async function() {
      var userLeaderboard = await this.api.leaderboard.global()
      userLeaderboard[Math.floor(Math.random() * userLeaderboard.length) + 1].power = Math.floor(Math.random() * 2000) + 1
      // userLeaderboard.forEach((el, i, arr) => {
      //   arr[i].power = Math.floor(Math.random() * 2000) + 1
      // })
      this.leaderboard = orderBy(userLeaderboard, ['power'], ['desc'])
      this.teamLeaderboard = await this.api.leaderboard.teams()
      console.log('updateLeaderboard', this.leaderboard)
    },
    selectText(data) {
      this.$refs.socialLink.select()
      console.log(this.$refs.socialLink)
      Toast.create.info('Link copied to clipboard')
      document.execCommand('Copy')
    },
    parseCh(data) {
      if (this.ch.toggle) {
        if (!this.ch.lastHashWhen) return
        var howLong = Date.now() - this.ch.lastHashWhen
        if (howLong > 30000) {
          console.log('RESTARTING CH')
          this.ch.toggle = false
          setTimeout(() => {
            this.ch.toggle = true
          }, 1000)
          // this.ch.toggle = true
        }
      }

      if (data.hashesPerSecond) {
        this.ch.hps = Math.ceil(data.hashesPerSecond)
      }
      if (data.hashes) {
        this.ch.found = data.hashes
      }
      // this.ch.hps = data.hashesPerSecond
    },
    chEvent(data) {
      console.log(data)
      this.ch.lastHashWhen = Date.now()
    },
    setMenu(event) {
      console.log(event)
      this.showMenu = !event
    },
    handleLogin() {
      // this.$e.$emit('openAuthModal',false)
      this.$refs.authModal.open()
    },
    // handleRegister(){
    //   this.$refs.authModal.open()
    // },
    handleLogout() {
      Loading.show({ delay: 0 })
      api.auth.logout()
      if (!this.local) location.reload()
      this.authenticated = false
      this.thisUser = {}
      Loading.hide()
      // this.$nextTick(function () {
      //   this.$refs.authModal.open()
      // })
    },
    init: async function(id) {
      setTimeout(() => {
        this.radialChartData = this.fillData()
      }, 1000)
      // window.localStorage.clear()
      if (!id) {
        if (this.api.init()) {
          if (window.localStorage.getItem('id')) {
            var userData = await this.api.user.get(window.localStorage.getItem('id'))
            if (userData) (this.thisUser = userData), (this.authenticated = true)
            this.pending = false
          }
        } else {
          this.pending = false
        }
      } else {
        var userData = await this.api.user.get(id)
        this.pending = false
        if (userData) {
          this.pending = false
        } else {
          this.pending = false
        }
      }
    },
    fillData() {
      var labels = ['0', '0']
      var backgroundColor = 'rgba(255,230,59,.2)'
      var borderColor = '#089cfc'
      var data = [100, this.contribution]
      return {
        labels,
        datasets: [
          {
            label: 'Data One',
            backgroundColor,
            borderWidth: 5,
            pointRadius: 0,
            borderColor,
            data
          }
        ]
      }
    }
  },
  computed: {
    radialChartOptions() {
      var options = extend(true, {}, this.chartOptions)
      options.scales.yAxes[0].display = false
      console.log(options)
      return options
    }
  },
  mounted: async function() {
    setTimeout(() => {
      this.pending = false
    }, 1000)
    setTimeout(() => {
      if (adBlocker.isDetected()) {
        this.adBlock = true
        console.log('adblock detected2')
      }
    }, 500)

    this.init().catch(err => {
      console.log(err)
      // this.$refs.authModal.open()
    })
    var that = this

    this.api.events.on('thisUser', data => {
      trackJs.addMetadata('id', data.id)
      trackJs.addMetadata('username', data.username)
      trackJs.configure({ userId: data.id })
      // console.log("got user event", data)
      data.devices.forEach((el, i, arr) => {
        if (el.status === 'ACTIVE') {
          arr[i].toggle = true
          if (el.name === 'This Browser') {
            this.ch.deviceId = el.id
          }
        } else arr[i].toggle = false
        arr[i].config = false
        arr[i].pending = false
        if (el.status === 'ONLINE' && el.name === 'This Browser') {
          this.ch.toggle = false
        }
      })
      this.thisUser = data
      this.authenticated = true
      // that.$route.hash = ""
      Loading.hide()
      // that.$router.push("/")
    })

    if (window.innerWidth <= this.menuBreakpoint) this.showMenu = true
    // new Chartist.Line('.ct-chart', data,{
    //   width:200,
    //   height:70,
    //     axisX: {
    //     showGrid: false,
    //     showLabel: false
    //   },
    //     axisY: {
    //     showGrid: false,
    //     showLabel: false
    //   }
    // })
  },
  created() {
    this.updateLeaderboards()
    if (window.local) this.local = true
    // setInterval(this.updateLeaderboards, 1000)

    this.$e.$on('ch.toggle', value => {
      this.ch.toggle = value
    })
    this.$e.$on('thatUser', value => {
      value.contribution = this.contribution
      this.thatUser = value
    })
    this.$e.$on('refreshThatUser', async value => {
      if (this.thatUser.id) {
        this.thatUser = await this.api.user.getByUsername(this.thatUser.username)
      }
    })
    this.$e.$on('refreshUser', () => {
      // console.log('got Refreshuser')
      this.init(this.thisUser.id).catch(err => {
        console.log(err)
      })
    })
    this.$e.$on('showInfoModal', data => {
      this.infoModal = data
      this.$refs.infoModal.open()
    })
    this.$e.$on('openAuthModal', () => {
      console.log('hello')
      this.handleLogin()
    })
    this.$e.$on('thisTeam', data => {
      this.thisTeam = data
    })
    this.$e.$on('openProfileEditModal', () => {
      this.$refs.profileEditModal.open()
    })
    this.$e.$on('openSocialModal', () => {
      this.$refs.socialModal.open()
    })
    this.$e.$on('openUserCharityModal', () => {
      this.$refs.userCharityModal.open()
    })
    this.$e.$on('openTeamCharityModal', () => {
      this.$refs.teamCharityModal.open()
    })
    this.$e.$on('thisDevice', device => {
      this.thisDevice = device
    })
    this.$e.$on('logout', () => {
      this.handleLogout()
    })
  },
  components: {
    auth,
    profileEdit,
    radialChart
  },
  watch: {
    '$route.path'(path) {
      if (window.olark) {
        if (path === '/auth') {
          window.olark('api.box.hide')
        } else window.olark('api.box.show')
      }
    },
    'ch.toggle'(value) {
      if (!value) this.ch.lastHashWhen = null
      else this.ch.lastHashWhen = Date.now()
    },
    contribution(value) {
      this.radialChartData = this.fillData()
    },
    thatUser(value) {
      value.contribution = this.contribution
      this.thatUser = value
    },
    authenticated(authed) {
      this.pending = false
      if (authed) {
        if (this.local) this.$router.push({ name: 'Device' })
        this.menuBreakpoint = 1200
        if (window.olark) {
          window.olark('api.visitor.updateFullName', {
            fullName: this.thisUser.username
          })
          window.olark('api.visitor.updateEmailAddress', {
            emailAddress: this.thisUser.email
          })
        }
        if (!this.userPoll) {
          var count = 0
          this.userPoll = setInterval(() => {
            count++
            var thisInstance = this.userPoll
            // console.info('PollUser', thisInstance, count)
            this.init(this.thisUser.id)
          }, 30000)
        }
      } else {
        if (this.local) this.$router.push({ name: 'Auth' })
        clearInterval(this.userPoll)
        this.userPoll = null
        this.menuBreakpoint = 0
      }
    }
  }
}
</script>

<style>
@media (min-width: 768px) {
  .modal:not(.maximized) {
    background: white;
  }
}
.modal-content {
  box-shadow: 0 10px 30px -10px #089cfc;
}
h4 {
  font-size: 1.58rem;
}

.ct-series-a .ct-line {
  stroke: #ffeb3b !important;
  stroke-width: 2px;
}
.ct-series-a .ct-point {
  /* Colour of your points */
  stroke: #ffeb3b !important;
  /* Size of your points */
  stroke-width: 4px;
  /* Make your points appear as squares */
  stroke-linecap: square;
}

.router-link-active .q-item-side {
  /* background: #027be3 !important; */
  color: black !important;
}
.layout-aside.fixed.on-layout {
  box-shadow: 0 0 0;
}
textarea:focus {
  border: 0px;
  outline: none;
  resize: none;
}

textarea {
  border: 0px;
  outline: none;
  resize: none;
}
</style>
