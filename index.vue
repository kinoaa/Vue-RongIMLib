<template>
  <div class="box">
    <div class="app-container">
      <div class="talk-room">
        <p class="title">聊天室详情</p>
        <div class="content">
          <talk-item :talk="talk" />
        </div>
        <div class="announcement" style="">
          <div>
            <el-input
              v-model="notice"
              type="textarea"
              :rows="3"
              class="say"
              autocomplete="off"
              placeholder="请输入公告内容,最多为50个字"
              maxlength="50"
              show-word-limit
            />
            <button class="send-announcement" @click="handlerPublish">发布</button>
          </div>
        </div>
      </div>
      <div class="talk-list">
        <p class="title">聊天室成员列表({{ num }})</p>
        <div class="content contents">
          <div v-for="(item,index) in list" :key="index" class="list">
            <img :src="item.portrait" alt="">
            <div class="nick">{{ item.name }}</div>
            <el-button v-permission="8010102" type="primary" class="button" size="medium" @click="ban(item.id)">禁言</el-button>
          </div>
        </div>
      </div>
    </div>
    <ban :dialog-ban="dialogBan" :dialog-ban-data="dialogBanData" @childMsg="childMsg" />
  </div>
</template>
<script>
import talkItem from './components/talkItem'
import { getCaption } from '@/filters/index'
import { getIMToken, publishBulletin } from '@/api/user'
import ban from './components/noTalk'
export default {
  name: 'Chatroom',
  components: { talkItem, ban },
  data() {
    return {
      num: 0,
      notice: '',
      gobalToken: {},
      list: [],
      talk: [],
      dialogBan: false,
      dialogBanData: '',
      rongyun: null,
      messageList: []
    }
  },
  created() {
    this.isToken()
    this.$nextTick(() => {
      const height = document.getElementsByClassName('app-main')[0].clientHeight
      document.getElementsByClassName('box')[0].style.height = height + 'px'
    })
  },
  destroyed() {
    var chatRoom = this.rongyun.ChatRoom.get({
      id: '62786522'
    })
    chatRoom.quit().then(function() {
      console.log('退出聊天室成功')
    })
  },
  methods: {
    // 获取token
    getIMToken() {
      getIMToken().then(res => {
        var time = new Date().getTime()
        res.time = time
        // 将token保存下来
        this.gobalToken = res
        var tokenStr = JSON.stringify(res)
        localStorage.setItem('token', tokenStr)
        // 初始化融云
        this.linkToRongs(res.data)
      })
    },
    // 判断token是否过期
    isToken() {
      var now = new Date().getTime()
      // 获取上一次存储的token
      var oldToken = JSON.parse(localStorage.getItem('token'))
      // 判断之前获取的token
      if (oldToken) {
        var tokenTime = oldToken.time
        // 判断时间是否过期了
        if (now - tokenTime > 29 * 24 * 60 * 60 * 1000) {
          this.getIMToken()
        } else {
          this.gobalToken = JSON.parse(localStorage.getItem('token'))
          this.linkToRongs(this.gobalToken.data)
          return
        }
      }
      this.getIMToken()
    },
    // 链接融云
    linkToRongs(token) {
      const that = this
      // 初始化融云
      // 测试服 key:
      // 正式服 key:
      let RongClientKey
      if (process.env.VUE_APP_BASE_API2 === '') {//测试服
        RongClientKey = ''
      } else {//正式服
        RongClientKey = ''
      }
      // 应用初始化以获取 RongIMLib 实例对象，请务必保证此过程只被执行一次
      // eslint-disable-next-line no-undef
      that.rongyun = RongIMLib.init({ appkey: RongClientKey })
      const im = that.rongyun
      // 添加事件监听
      im.watch({
        // 监听会话列表变更事件
        conversation(event) {
          // 假定存在 getExistedConversationList 方法，以获取当前已存在的会话列表数据
          const conversationList = that.getExistedConversationList(im)
          // 发生变更的会话列表
          const updatedConversationList = event.updatedConversationList
          // 通过 im.Conversation.merge 计算最新的会话列表
          const latestConversationList = im.Conversation.merge({ conversationList, updatedConversationList })
          console.log(latestConversationList)
        },
        // 监听消息通知
        message(event) {
          that.messageList = []
          // 新接收到的消息内容
          const message = event.message
          if (message.messageType === 'RC:InfoNtf') {
            message.content.messageName = 'InfoNtf'
            that.talk.push(message.content)
          } else {
            message.content.messageName = 'TextMessage'
            message.content.user.portrait = getCaption(message.content.user.portrait)
            that.messageList.push(message.content)
            message.content.sentTime = message.sentTime
            that.talk.push(message.content)
          }
        },
        // 监听 IM 连接状态变化
        status(event) {
          console.log('connection status:', event.status)
        },
        // 监听聊天室 KV 数据变更
        chatroom(event) {
          const updatedEntries = event.updatedEntries
          console.log('聊天室 KV 存储数据更新', updatedEntries)
        }
      })
      // 建立 IM 连接
      const chatRoomId = '62786522'
      var count = 50 // 排序方式
      im.connect({ token: token }).then(user => {
        this.$message.success('加入聊天室成功')
        console.log('链接成功, 链接用户 id 为: ', user.id)
        var chatRoom = im.ChatRoom.get({
          id: chatRoomId
        })
        chatRoom.join({
          count: count // 进入后, 自动拉取 20 条聊天室最新消息
        }).then(function() {
          console.log('加入聊天室成功')
          chatRoom.getInfo().then(function(result) {
            var userCount = result.userCount
            var user = that.uniq(that.messageList)
            // 刷选用户
            user.map(item => {
              that.list.push(item.user)
            })
            that.num = userCount
          })
        })
      }).catch(error => {
        this.$message.success('加入聊天室失败')
        console.log('链接失败: ', error.code, error.msg)
      })
    },
    // 数组去重(元素是对象 按照id进行筛选)
    uniq(oldArr) {
      var newArr = []
      for (var i = 0; i < oldArr.length; i++) {
        var flag = true
        for (var j = 0; j < newArr.length; j++) {
          if (oldArr[i].id === newArr[j].id) {
            // 如果相同id 将传入的name赋值给newArr[j] 以防用户在聊天的时候修改昵称 和头像
            newArr[j].portrait = oldArr[i].portrait
            newArr[j].name = oldArr[i].name
            flag = false
          }
        }
        if (flag) {
          newArr.push(oldArr[i])
        }
      }
      return newArr
    },
    // 发布公告
    handlerPublish() {
      if (this.notice === '') {
        this.$message.error('请输入内容')
        return
      }
      const data = {
        content: this.notice
      }
      publishBulletin(data).then(res => {
        if (res.succeed) {
          this.$message.success('公告发布成功')
          this.talk.push({
            message: this.notice
          })
          this.notice = ''
        }
      })
    },
    // 禁言
    ban(userId) {
      this.dialogBan = true
      this.dialogBanData = userId
    },
    // 子组件通信
    childMsg(msg) {
      if (msg.dialogBan === false) {
        this.dialogBan = false
        this.dialogBanData = ''
      }
    }
  }
}
</script>

<style  scoped lang='scss'>
.box{
  background-color: #f5f5f5;
}
.app-container{
  display: flex;
  justify-content: space-around;
}
.talk-room{
  width: 44%;
  height: 800px;
  min-width: 400px;
  background-color: #fff;
  padding: 0 10px;
}
.title{
    margin: 0;
    font-size: 20px;
    font-weight: 400;
    text-align: center;
    padding-top: 10px;
    padding-bottom: 10px;
    border-bottom: 1px solid #ccc;
  }
.talk-list{
  width: 29%;
  background-color: #fff;
}
.announcement{
 div{
  display: flex;
  justify-items: center;
  height: 83px;
  margin-top: 10px;
  .say{
    margin: 0;
  }
 }
}
.send-announcement {
  width: 80px;
  background-color: #67c23a;
  border: none;
  user-select: none;
  border-radius: 4px;
}
.content{
  height: 655px;
  position: relative;
  overflow-x: hidden;
  overflow-y: auto;
}
.contents{
  height: 755px;
}
.list{
  display: flex;
  align-items: center;
  padding: 10px;
  border-bottom: 1px solid #ccc;
  img{
    width: 40px;
    height: 40px;
    border-radius: 10px;
  }
  .nick{
    margin-left: 10px;
  }
  .button{
    position: absolute;
    right: 10px;
  }
}
.content::-webkit-scrollbar {
  width: 5px;

  height: 13px;

  -webkit-border-radius: 5px;

  -moz-border-radius: 5px;

  border-radius: 5px;
}

.content::-webkit-scrollbar-thumb {
  background-color: #fff;

  background-clip: padding-box;

  -webkit-border-radius: 5px;

  -moz-border-radius: 5px;

  border-radius: 5px;

  min-height: 28px;
}
</style>
