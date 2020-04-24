<template>
  <q-layout view="lHh Lpr lFf">
    <q-header elevated>
      <q-toolbar>
        <q-btn
          flat
          dense
          round
          icon="menu"
          aria-label="Menu"
          @click="leftDrawerOpen = !leftDrawerOpen"
        />

        <q-toolbar-title>
          Sua agenda
        </q-toolbar-title>
      </q-toolbar>
    </q-header>

    <q-drawer
      v-model="leftDrawerOpen"
      show-if-above
      v-if="!dataClime"
      bordered
      content-class="bg-grey-1"
    >
      <q-list>
        <q-item-label
          header
          class="text-grey-8"
        >
          Essential Links
        </q-item-label>
        <EssentialLink
          v-for="link in essentialLinks"
          :key="link.title"
          v-bind="link"
        />
      </q-list>
    </q-drawer>
    <section class="container-backdrop">
      <div class="content-lead">
        <h6>{{ dataClime.data.date | moment("dddd, MMMM Do YYYY") }}</h6>
        <h4>{{ dataClime.name }}</h4>
        <h2>{{ dataClime.data.temperature }} C&deg;</h2>
        <h6 class="color-black">Sensação Termica: {{ dataClime.data.sensation}}&deg;</h6>
      </div>
       <div class="content-img">
         <h4>{{ dataClime.data.condition }}</h4>
         <img v-bind:src="`../statics/${dataClime.data.icon}.png`" alt="its rain" width="150" height="150">
       </div>
    </section>

    <section>
      <div class="content-list-title">
        <h5><b>Lista de tarefa para hoje</b></h5>
        <button @click="open = true" label="Open">Adicionar tarefa</button>
      </div>
    </section>
    <section class="container-list-todo">
        <ul>
          <li class="content-list-todo" v-for="list in listToDoToday" :key="list.title">
              <div class="content-list-checkbox">
                <input type="checkbox">
                <label for="checkbox">{{list.title}}</label>
              </div>
              <img v-if="list.priority" src="../statics/alert.png" class="list-priodity" title="Essa Tarefa é priodidade" alt="Priodidade" width="30">
              <img src="../statics/delete.png" v-on:click="deleteList()" alt="delete" width="30">
         </li>
        </ul>
    </section>

    <section>
      <div class="content-list-title">
        <h5><b>Noticias</b></h5>
      </div>
    </section>

    <section class="container-news-list">
      <ul>
        <li v-for="news in dataNews.articles" class="content-list-news" :key="news.title">
          <a :href="news.url" target="_blank">
              <img v-bind:src="news.urlToImage" @error="imageLoadError" alt="image" width="300">
          </a>
          <div class="spacing-border">
            <a :href="news.url" target="_blank">
              <h5>{{ news.title }}</h5>
            </a>
            <p>{{news.description}}</p>
          </div>
          <div class="footer-list-content">
            Por {{news.author}} - Às {{news.publishedAt |  moment("HH:MM")}}
          </div>
        </li>
      </ul>
    </section>

     <section>
      <div class="content-list-title">
        <h5><b>Albuns e musicas Lançadas hoje</b></h5>
      </div>
    </section>
    <section class="container-list-albuns">
      <ul v-if="albunsReleased">
        <li class="content-list-albuns" v-for="albums in albunsReleased.items" :key="albums.id">
            <img :src="albums.images[1].url" alt="" width="300" height="300">
            <div class="content-about-album">
                <p>{{albums.album_type}}</p>
                <h5>{{albums.name}}</h5>
                <div class="about-artists">
                  <p class="gray">De</p>
                <p v-for="artist in albums.artists" :key="artist.id">
                  <a :href="artist.external_urls.spotify" target="_blank">
                    {{ artist.name }} &nbsp;
                  </a>
                </p>
                </div>
                <a :href="albums.external_urls.spotify" type="button" class="play-on-spotify" target="_blank">Play on Spotify</a>
            </div>
        </li>
      </ul>
    </section>

    <q-page-container>
      <router-view />
    </q-page-container>
  </q-layout>
</template>

<script>
import Vue from 'vue'
import EssentialLink from 'components/EssentialLink'
import axios from 'axios'

export default {
  name: 'MainLayout',
  
  components: {
    EssentialLink
  },
  methods: {
    deleteList() {
      console.log("teste")
    }
  },
  mounted() {
    this.$axios.get('http://apiadvisor.climatempo.com.br/api/v1/weather/locale/3477/current?token=8564057d6b6ce399e09795c0fff9883a')
      .then((response) => {
        this.dataClime = response.data
      })
      .catch(() => {
        this.$q.notify({
          color: 'negative',
          position: 'top',
          message: 'Loading failed',
          icon: 'report_problem'
        })
      })
       this.$axios.get('http://newsapi.org/v2/top-headlines?sources=google-news-br&apiKey=571e4099ff5446c3b2d87b545b9b257e')
      .then((response) => {
        this.dataNews = response.data
      })
      .catch(() => {
        this.$q.notify({
          color: 'negative',
          position: 'top',
          message: 'Loading failed',
          icon: 'report_problem'
        })
      })
      const config = {
        headers: { Authorization: `Bearer BQCLWF46DEq9DnYV0gVAoTlS8qWmFyJqBch7hkGXrBVTHcUhwL_HlV2j_9LJqQHOcJ30M7pdc5fCZO2zZaFA4YqTS_K4sSe6fOM3l03MVcL5c3PxFVJAbD6TrmpNSoJhJcVVq-uaKKZxXQY8GdCRHrY4PphK9YOeA7d1gd-mrnPKCp66aajk59Tv-JT5kkLwIUBOvbF5G-vxFDgNZ3tRy8Ex1vAzzAm-YyBPdCqXkc2LD_RihuiWTBHBMeTln0loxkN_58GMxCilS0Y` }
      };
      this.$axios.get('https://api.spotify.com/v1/browse/new-releases?country=BR', config)
      .then((response) => {
        this.albunsReleased = response.data.albums
      })
      .catch(() => {
        this.$q.notify({
          color: 'negative',
          position: 'top',
          message: 'Loading failed',
          icon: 'report_problem'
        })
      })

},

methods: {
  imageLoadError(e) {
    e.target.src = '../statics/No-Image-Found.png'
  }
},

  data () {
    return {
      leftDrawerOpen: false,
      dataClime: null,
      dataNews: null,
      open: false,
      albunsReleased: null,
      listToDoToday: [
        {
         title: 'Comprar Roupar1',
         description: "Tenho que ir para lavanderina para poder lavar minhas roupas sujas",
         validate: "20/04/2020/ 20hs",
         priority: true
      }, 
      {
         title: 'Comprar Roupar e sair por ai pra comprar',
         description: "Tenho que ir para lavanderina",
         validate: "20/04/2020/ 20hs",
         priority: false
      },
       {
         title: 'Comprar Roupar3',
         description: "Tenho que ir para lavanderina",
         validate: "20/04/2020/ 20hs",
         priority: true
      }
      ],
      essentialLinks: [
        {
          title: 'Docs',
          caption: 'quasar.dev',
          icon: 'school',
          link: 'https://quasar.dev'
        },
        {
          title: 'Github',
          caption: 'github.com/quasarframework',
          icon: 'code',
          link: 'https://github.com/quasarframework'
        },
        {
          title: 'Discord Chat Channel',
          caption: 'chat.quasar.dev',
          icon: 'chat',
          link: 'https://chat.quasar.dev'
        },
        {
          title: 'Forum',
          caption: 'forum.quasar.dev',
          icon: 'record_voice_over',
          link: 'https://forum.quasar.dev'
        },
        {
          title: 'Twitter',
          caption: '@quasarframework',
          icon: 'rss_feed',
          link: 'https://twitter.quasar.dev'
        },
        {
          title: 'Facebook',
          caption: '@QuasarFramework',
          icon: 'public',
          link: 'https://facebook.quasar.dev'
        },
        {
          title: 'Quasar Awesome',
          caption: 'Community Quasar projects',
          icon: 'favorite',
          link: 'https://awesome.quasar.dev'
        }
      ]
    }
  }
}
</script>

<style>
a {
  text-decoration: none;
  color: #236cb5;
}

#q-app {
  background-color: #9e9e9e14;
}

.gray {
  color: #7d7d7d;
  font-weight: bold;
  margin-right: 5px;
}

.spacing-border {
 padding: 15px;
 
}

.container-backdrop {
  background-image: linear-gradient(rgba(0, 0, 0, 0), rgb(247, 247, 247)), url('../statics/background-rain-nigth.jpg');
  background-repeat: no-repeat;
  background-size: cover;
  height: 400px;
  display: flex;
  justify-content: space-between;
}

.container-backdrop .content-lead h4, h6, h2 {
  font-weight: bold;
  position: relative;
  top: 100px;
  left: 300px;
  margin-bottom: -40px;
}

.container-backdrop .content-lead h4, h2, .color-black {
  color: #000000;
}

.container-backdrop .content-img {
  position: relative;
  top: -20px;
  right: 300px;
  height: auto;
  flex-direction: column-reverse;
  text-align: center;
  align-items: center;
  display: flex;
}

.container-backdrop .content-img .color-black {
  font-size: 18px;
}

.container-backdrop .content-img h4 {
  font-weight: bold;
  color: #949494;
  font-size: 25px;
  margin-top: 15px;
}

.container-list-todo {
  margin: auto;
}

.container-list-todo .content-list-todo {
  margin: 20px;
  border-radius: 8px;
  text-align: center;
  width: 90%;
}

section .content-list-title {
  margin-left: 200px;
  display: flex;
  justify-content: space-between;
}

section .content-list-title button {
  margin: auto;
  background-color: #fff;
  font-weight: bold;
  color: #1976d2;
  border: 2px solid #1976d2;
  border-radius: 8px;
  padding: 10px 20px;
  cursor: pointer;
}

.container-list-todo ul {
  list-style: none;
  margin-left: 100px;
}

.container-list-todo ul li {
  display: flex;
  background: #1976d2;
  padding: 14px;
  border-radius: 8px;
  position: relative;
  border: 3px solid #1976d2;
  justify-content: space-between;
  box-shadow: 2px 2px 2px gray;
}

.container-list-todo ul li input {
  width: 20px;
  height: 20px;
  position: relative;
  bottom: -4px;
}

.container-list-todo ul li label {
  color: #fff;
  font-weight: bold;
  margin-left: 30px;
  text-align: left;
}

.container-list-todo ul li .content-list-checkbox {
  width: 100%;
  display: flex;
}

.container-list-todo ul li .list-priodity {
  margin-right: 10px;
}

.container-news-list {
  display: flex;
  justify-content: center;
}

.container-news-list ul {
   display: grid;
   grid-template-columns: 1fr;
   grid-gap: 24px;
   list-style: none;
}

.container-news-list ul .content-list-news {
  margin: 20px;
  display: flex;
  justify-content: space-between;
  width: 900px;
  position: relative;
  border-bottom: 2px solid #80808059;
}

.container-news-list ul li h5 {
  font-size: 20px;
  margin-top: -10px;
  margin-bottom: 10px;
  line-height: 30px;
  font-weight: 500;
}

.container-news-list ul li p {
  color: #333333;
}

.container-news-list ul .content-list-news img {
  width: 320px;
  height: 180px;
  margin-bottom: 20px;
}

.container-news-list ul li .footer-list-content {
  position: absolute;
  bottom: 0;
  padding-bottom: 10px;
  right: 15px;
}

.container-news-list ul li .spacing-border p {
  font: 16px/20px Arial,sans-serif;
  font-weight: 400;
}

.container-list-albuns {
  margin: auto;
}

.container-list-albuns ul {
  display: grid;
  grid-template-columns: 1fr;
  grid-gap: 24px;
  list-style: none;
}

.container-list-albuns ul li{
  border: 2px solid #eaeaea;
  display: flex;
  background-color: #222326;
  color: #fff;
  align-items: center;
  margin: auto;
  width: 1000px;
}

.container-list-albuns ul li .content-about-album {
  margin-left: 20px;
  margin-top: 20px;
}

.container-list-albuns ul li .content-about-album .about-artists {
  display: flex;
  color: #fff;
}

.container-list-albuns ul li h5 {
  font-weight: bold;
  font-size: 30px;
  color: #fff;
}

.container-list-albuns ul li .about-artists a {
  color: #eaeaea;
}

.container-list-albuns ul li .play-on-spotify {
  border-radius: 30px;
  color: #fff;
  border: 2px solid #1db954;
  background-color: #1db954;
  font-weight: bold;
  padding: 5px 24px;
  cursor: pointer;
}
</style>