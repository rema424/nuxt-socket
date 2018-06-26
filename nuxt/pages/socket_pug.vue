<template lang="pug">
  section.section
    #wrapper.container
      article.media
        .media-content
          .field.is-grouped
            p.control.is-expanded
              input.input(type="text" v-model="message" @keyup.enter="sendMessage" placeholder="message")
            p.control
              button.button.is-info(@click="sendMessage") Pug
              
      article.media(v-for="(message, index) in reverseMessages" :key="index")
        figure.media-left
          p.image.is-64x64
            img(src="https://bulma.io/images/placeholders/128x128.png")
        .media-content
          .content
            p 
              strong id: {{ message.user }}
              br
              | {{ message.text }}
              br
              small 
                a Like
                |  · 
                a Reply
                |  · {{ message.date }}

      article.media
        figure.media-left
          p.image.is-64x64
            img(src="https://bulma.io/images/placeholders/128x128.png")
        .media-content
          .content
            p
              strong Barbara Middleton
              br
              | Lorem ipsum dolor sit amet, consectetur adipiscing elit. Duis porta eros lacus, nec ultricies elit blandit non. Suspendisse pellentesque mauris sit amet dolor blandit rutrum. Nunc in tempus turpis.
              br
              small
                a Like
                |  · 
                a Reply
                |  · 3 hrs

      article.media
        figure.media-left
          p.image.is-64x64
            img(src="https://bulma.io/images/placeholders/128x128.png")
        .media-content
          .content
            p
              strong Barbara Middleton
              br
              | Lorem ipsum dolor sit amet, consectetur adipiscing elit. Duis porta eros lacus, nec ultricies elit blandit non. Suspendisse pellentesque mauris sit amet dolor blandit rutrum. Nunc in tempus turpis.
              br
              small
                a Like
                |  · 
                a Reply
                |  · 3 hrs

      b-loading(:is-full-page="false" :active.sync="isLoading" :can-cancel="false")
</template>

<script>
import io from 'socket.io-client'

export default {
  data() {
    return {
      message: '',
      messages: [],
      socket: '',
      isLoading: true
    }
  },
  computed: {
    // 配列の後ろ（新しいもの）から順に表示させたいので反転させる
    reverseMessages: function() {
      return this.messages.slice().reverse()
    },
  },
  mounted() {
    // VueインスタンスがDOMにマウントされたらSocketインスタンスを生成する
    this.socket = io()

    // サーバー側で保持しているメッセージを受信する
    this.socket.on('new-message', message => {
      this.messages.push( message || {} )
    })

    // コンポーネントがマウントされてから1秒間はローディングする
    setTimeout(() => {
      this.isLoading = false
    }, 1000)
  },
  methods: {
    sendMessage() {
      // スペースのみの場合は即時リターンをする
      if (!this.message.trim()) {
        return
      }

      let now = new Date()  // 現在時刻（世界標準時）を取得
      now.setTime(now.getTime() + 1000 * 60 * 60 * 9) // 日本時間に変換
      now = now.toJSON().split('T')[1].slice(0, 5)  // 時刻のみを取得

      // メッセージオブジェクトを作る
      let message = {
        user: this.socket.id,
        date: now,
        text: this.message.trim(),
      }

      // 自身（Vueインスタンス）のデータオブジェクトにメッセージを追加する
      this.messages.push(message)
      // サーバー側にメッセージを送信する
      this.socket.emit('send-message', message)
      // input要素を空にする
      this.message = ''
    }
  }
}
</script>

<style scoped>
#wrapper
{
  max-width: 600px;
}
</style>
