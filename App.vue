<script>
  import {
    version,
    name
  } from './package.json'
  import {
    versionName
  } from '@/manifest.json'
  import consoleImgs from '@/common/consoleImgs.js'
  // #ifdef APP-PLUS
  import appUpgrade from '@/common/appUpgrade.js'
  const TUICalling = uni.requireNativePlugin(
    'TUICallingUniPlugin-TUICallingModule'
  )
  // #endif

  export default {
    onLaunch: function() {
      // #ifdef H5
      console.log(consoleImgs.fz)
      // todo 下列两行
      uni.setStorageSync('device', 'H5')
      uni.setStorageSync('version', versionName)
      this.$http.request({
        url: '/common/getVersion',
        success: (res) => {
          if (res.data.data.upgrade == 'Y') {
            console.log(
              `%c 有新版本 ` + res.data.data.version,
              'background:#007aff ;padding: 1px; border-radius: 0 3px 3px 0;  color: #fff; font-weight: bold;'
            )
          }
        }
      })

      // #endif
      console.log('App Launch')
      let token = uni.getStorageSync('Authorization')
      if (!token) {
        //不存在则跳转至登录页
        // #ifdef APP-PLUS
        plus.navigator.closeSplashscreen()
        // #endif
      } else {
        // #ifdef H5
        this.$socketTask.connectSocket()
        // #endif
        this.$store.dispatch('get_UserInfo').then((res) => {
          // #ifdef APP-PLUS
          var nickName = res.nickName
          var portrait = res.portrait

          this.$http.request({
            url: '/trtc/getSign',
            success: (res) => {
              var sdkAppID = res.data.data.appId
              var userID = res.data.data.userId
              var userSig = res.data.data.sign
              TUICalling.login({
                  //登录音视频
                  sdkAppID: sdkAppID,
                  userID: userID,
                  userSig: userSig
                },
                (res) => {
                  console.log('音视频登录成功')
                  TUICalling.setUserNickname({
                    nickName: nickName
                  })
                  TUICalling.setUserAvatar({
                    avatar: portrait
                  })
                  plus.io.requestFileSystem(plus.io.PRIVATE_WWW, function(
                    fs) {
                    fs.root.getFile(
                      '/static/longcall.mp3', {
                        create: false
                      },
                      function(fileEntry) {
                        fileEntry.file(function(file) {
                          TUICalling
                            .setCallingBell({
                                ringtone: file
                                  .fullPath
                              },
                              (res) => {}
                            )
                        })
                      }
                    )
                  })
                }
              )
            }
          })
          // 绑定客户端ID
          var nowCid = plus.push.getClientInfo().clientid
          this.$http.request({
            url: '/my/bindCid/' + nowCid,
            success: (res) => {
              console.log('新cid' + nowCid)
              uni.setStorageSync('cid', nowCid)
            }
          })
          // #endif
        })
        uni
          .reLaunch({
            url: 'pages/tabbar1/index'
          })
          .then((res) => {
            // #ifdef APP-PLUS
            plus.navigator.closeSplashscreen()
            // #endif
          })
      }
      // #ifdef APP-PLUS
      //升级检测
      uni.getSystemInfo({
        success: (res) => {
          uni.setStorageSync('device', res.platform)
          plus.runtime.getProperty(plus.runtime.appid, (widgetInfo) => {
            uni.setStorageSync('version', widgetInfo.version)
            this.$http.request({
              url: '/common/getVersion',
              success: (res) => {
                if (res.data.data.upgrade == 'Y') {
                  appUpgrade.init({
                    titleText: '版本更新' + res.data.data
                      .version,
                    packageUrl: res.data.data.url,
                    content: res.data.data.content,
                    forceUpgrade: res.data.data
                      .forceUpgrade == 'Y' ? true : false
                  })
                  appUpgrade.show()
                }
              }
            })
          })
        }
      })
      uni.onNetworkStatusChange((res) => {
        if (res.isConnected) {
          this.$store.dispatch('get_UserInfo')
        }
      })
      // #endif
    },
    onShow: function() {
      console.log('App Show')
      uni.getStorage({
        key: 'call',
        success: (res) => {
          var callx = res.data
          if (callx) {
            var call = JSON.parse(callx)

            function getInervalHour(startDate) {
              //获取两个时间之间的小时
              if (!startDate) {
                return '0秒'
              }
              var ms = new Date().getTime() - startDate
              if (ms < 0) return '0秒'
              if (ms / 1000 < 60) {
                return Math.floor(ms / 1000) + '秒'
              } else {
                return Math.floor(ms / 1000 / 60) + '分'
              }
            }
            var msgType = ''
            if (call.type == 'audio') {
              msgType = 'TRTC_VOICE_END'
            }
            if (call.type == 'video') {
              msgType = 'TRTC_VIDEO_END'
            }
            this.$fc.pushOutMsg({
              msgContent: getInervalHour(call.startTime),
              msgType: msgType,
              windowType: 'SINGLE',
              userId: call.userId
            })
            uni.removeStorageSync('call')
          }
        }
      })
    },
    onHide: function() {
      console.log('App Hide')
    }
  }
</script>

<style lang="scss">
  @import '@/uni_modules/uni-scss/index.scss';
  /*每个页面公共css */
  @import '@/static/styles/animation.css';
  /* #ifndef APP-NVUE */
  @import '@/static/customicons.css';

  .mb10 {
    margin-bottom: 10px;
  }

  // 设置整个项目的背景色
  page {
    box-sizing: border-box;
  }

  /* #endif */

  /************************************************************
** 请将全局样式拷贝到项目的全局 CSS 文件或者当前页面的顶部 **
** 否则页面将无法正常显示                                  **
************************************************************/

  html {
    font-size: 16px;
  }

  body {
    margin: 0;
    font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', 'Roboto', 'Oxygen', 'Ubuntu', 'Cantarell', 'Fira Sans',
      'Droid Sans', 'Helvetica Neue', 'Microsoft Yahei', sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
  }

  view,
  image,
  text {
    box-sizing: border-box;
    flex-shrink: 0;
  }

  #app {
    width: 100vw;
    height: 100vh;
  }

  .flex-row {
    display: flex;
    flex-direction: row;
  }

  .flex-col {
    display: flex;
    flex-direction: column;
  }

  .justify-start {
    justify-content: flex-start;
  }

  .justify-end {
    justify-content: flex-end;
  }

  .justify-center {
    justify-content: center;
  }

  .justify-between {
    justify-content: space-between;
  }

  .justify-around {
    justify-content: space-around;
  }

  .justify-evenly {
    justify-content: space-evenly;
  }

  .items-start {
    align-items: flex-start;
  }

  .items-end {
    align-items: flex-end;
  }

  .items-center {
    align-items: center;
  }

  .items-baseline {
    align-items: baseline;
  }

  .items-stretch {
    align-items: stretch;
  }

  .self-start {
    align-self: flex-start;
  }

  .self-end {
    align-self: flex-end;
  }

  .self-center {
    align-self: center;
  }

  .self-baseline {
    align-self: baseline;
  }

  .self-stretch {
    align-self: stretch;
  }

  .flex-1 {
    flex: 1 1 0%;
  }

  .flex-auto {
    flex: 1 1 auto;
  }

  .grow {
    flex-grow: 1;
  }

  .grow-0 {
    flex-grow: 0;
  }

  .shrink {
    flex-shrink: 1;
  }

  .shrink-0 {
    flex-shrink: 0;
  }

  .relative {
    position: relative;
  }
</style>
