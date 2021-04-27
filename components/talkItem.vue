<template>
  <div>
    <div v-for="(item,index) in talk" :key="index">
      <div v-if="item.messageName==='TextMessage'" class="user">
        <div class="img">
          <img class="imgs" :src="item.user.portrait|getCaption" alt="">
        </div>
        <div class="message">
          <p class="nick-name">{{ item.user.name }}
            <span class="time">{{ item.sentTime | timeFormat }}</span>
          </p>
          <div class="text">{{ item.content }}</div>
          <div class="triangle" />
        </div>
      </div>
      <div v-else class="say">
        <p>{{ item.message }}</p>
      </div>
    </div>
  </div>
</template>

<script>

export default {
  props: {
    talk: {
      type: Array,
      default: () => {
        return []
      }
    }
  }
}
</script>

<style lang="scss" scoped>
.user{
  display: flex;
  margin-top: 20px;
  .imgs{
    width: 40px;
    height: 40px;
    border-radius: 10px;
  }
  .message{
    margin-left: 10px;
    position: relative;
    .nick-name{
      font-size: 16px;
      margin: 0;
    }
    .text{
      width: fit-content;
      background: red;
      max-width: 300px;
      border-radius: 4px;
      padding: 10px;
      background: #409eff;
      color: #fff;
    }
    .triangle{
      position: absolute;
      top: 28px;
      left: -16px;
      width: 0;
      height: 0;
      border: 8px solid #409eff;
      border-left: 8px solid transparent;
      border-top: 8px solid transparent;
      border-bottom: 8px solid transparent;
    }
  }
}
.say{
  display: flex;
  justify-content: center;
  padding: 5px;
  margin-top: 10px;
  p{
    margin: 0;
    background-color: #e6e6e6;
    padding: 6px;
    border-radius: 10px;
    font-size: 12px;
    max-width: 300px;
    word-wrap: break-word;
  }
}
.time{
  font-size: 12px;
  color: #bbb8b8;
}
</style>
