
<template style="">
  <div class="page-container_main">
    <audio id="notification" src="/sound.mp3" muted />
    <section class="page-container">
      <LayoutHeader v-if="!pages.includes($nuxt.$route.name)" />
      <section class="page-container__layout">
        <LayoutNav v-if="$nuxt.$route.name!=='index'" /><Nuxt class="page-container__content" />
      </section>
      <LayoutFooter />
    </section>
    <section v-if="modal.type" class="modal" @click.self="Close">
      <UserAvatarChangeModal
        v-if="modal.type === 'updateUserAvatar'"

        :avatar="'https://cdnsocial.katelinlis.xyz/public/clients/1/cc31b175288b869bd96c18fa2070f9cc.jpeg'"
      />
      <UserAvatarOpenAvatar
        v-if="modal.type === 'OpenUserAvatar'"

        :avatar="modal.data"
      />
    </section>
  </div>
</template>

<script>
import { mapState, mapActions } from 'vuex'

export default {
  data () {
    return { pages: ['login', 'register'] }
  },
  head () {
    return {
      htmlAttrs: {
        lang: this.$i18n.localeProperties.code
      },
      bodyAttrs: {
        style: this.modal.type ? ' overflow: hidden;' : ''
      },
      link: [
        {
          rel: 'canonical',
          href: 'https://only-one.su' + this.$nuxt.$route.path
        }
      ],
      meta: [
        {
          name: 'viewport',
          content: 'width=device-width,initial-scale=1.0'
        },
        {
          name: 'og:site_name',
          hid: 'og:site_name',
          content: 'Only one'
        },
        {
          name: 'og:title',
          hid: 'og:title',
          content: 'Only one'
        }
      ]
    }
  },

  computed: {
    ...mapState({
      modal: state => state.modal,
      socket: state => state.socket

    })

  },
  async beforeMount () {
    await this.InitWebSocket()
  },
  mounted  () {
    // this.socket = new WebSocket('wss://only-one.su/ws')
    // this.getMe()
    if (this.$cookies.get('token')) {
      this.socket.addEventListener('open', (event) => {
        console.log('ok')
        this.socket.send(JSON.stringify({
          data: this.$cookies.get('token'),
          type: 'init'
        }))
      })

      this.socket.addEventListener('message', (event) => {
        if (document.getElementById('notification').muted) { document.getElementById('notification').muted = false }
        document.getElementById('notification').play()
        JSON.parse(event.data).forEach((element) => {
          switch (element.type) {
            case 'request_send':
              this.$toast.info(`${element.username} отправил(а) заявку в друзья`, {
                position: 'bottom-right',
                duration: 15000,
                action: [
                  {
                    text: 'открыть',
                    onClick: () => {
                      this.$router.push(`/user/${element.userid}`)
                    }
                  }
                ]
              })
              break
            case 'request_accept':
              this.$toast.info(`${element.username} принял(а) заявку в друзья`, {
                position: 'bottom-right',
                duration: 15000,
                action: [
                  {
                    text: 'открыть',
                    onClick: () => {
                      this.$router.push(`/user/${element.userid}`)
                    }
                  }
                ]
              })
              break
            case 'message_send':
              this.$toast.info(`${element.username} написал(а) в личные сообщения`, {
                position: 'bottom-right',
                duration: 15000,
                action: [
                  {
                    text: 'открыть',
                    onClick: () => {
                      this.$router.push(`/im?im=${element.userid}`)
                    }
                  }
                ]
              })
              break
          }
        })
      })
    }
  },
  methods: {
    Close () {
      console.log(event)
      this.$store.commit('SetModal', { type: '', data: {} })
    },
    async Update () {
      await this.getMe()
    },
    ...mapActions(['getMe', 'InitWebSocket'])
  }
}
</script>

<style scoped>
  .modal{
    position:absolute;
    z-index:1000;
    background-color: #80808085;
    width: 100%;
    height: 100%;
    top: 0;
    left: 0;
  }
  .page-container_main {
    position: relative;
    width: 100%;
    min-height: 100%;

  }
  .page-container {
    position: relative;
    display: flex;
    flex-direction: column;
    margin: 0 auto;
    width: 80vw;
    min-height: 100%;

  }

  .page-container__layout {
    position: relative;
    display: flex;
    flex-grow: 1;
    align-self: stretch;
  }

  .page-container__layout * {
    display: flex;
    flex-grow: 1;
    align-items: stretch;
    overflow: auto;
  }

  .page-container__layout nav {
    flex-grow: 0;
    flex-shrink: 0;
    width: 13vw;
  }

  @media screen and (orientation: portrait) {
    .page-container {
      width: 100%;
    }
    .navsMenuDesktop {
      display: none;
    }

    .container {
      width: 100vw;
      left: 0;
    }

    .content {
      position: absolute;
      margin: 0 auto;
    }
  }
  .content {
    position: relative;
    margin-left: 1%;
    height: max-content;
    min-height: 700px;
    z-index: 1;
    width: 80%;
  }
  @media only screen and (max-width: 768px) {
    .content {
      left: unset;
      width: 100%;
    }
  }

</style>

<style>
@import url('https://fonts.googleapis.com/css2?family=Inter:wght@500&family=Roboto:wght@500;700&display=swap');

  body{
    background: #F5F8FD;
  }
  * {
    margin: 0;
    padding: 0;
    border: 0;
    box-sizing: border-box;
    font-family: sans-serif;
  }
  button {
   border-radius: 4px;
   margin: 4px 0;
   padding: 4px;
  }
  html, body, #__nuxt, #__layout {
    width: 100%;
    height: 100%;
  }
</style>
