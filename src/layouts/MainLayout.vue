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
      </q-list>
    </q-drawer>
    <section class="container-backdrop" v-bind:style="backgroundClime">
      <div class="content-lead">
        <h6>{{ dataClime.data.date | moment("HH:MM, dddd, MMMM Do YYYY")}}</h6>
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
        <button @click="show()" label="Open">Adicionar tarefa</button>
      </div>
    </section>
    <section v-if="listToDoToday.length > 0" class="container-list-todo">
        <ul>
          <li class="content-list-todo" v-for="list in listToDoToday" :key="list.title">
              <div class="content-list-checkbox verticall">
                <span>Nome da atividade</span>
                <p>{{list.assignmentName}}</p>
              </div>
              <div class="content-list-checkbox d-flex verticall">
                 <span>Hora de inicio</span>
                 <p>{{list.assignmentDate}}</p>
              </div>
              <div class="content-list-checkbox d-flex verticall">
                 <span>Hora de termino</span>
                 <p>{{list.assignmentDateEnd}}</p>
              </div>
              <img v-if="list.assignmentAlert" src="../statics/alert.png" class="list-priodity" title="Essa Tarefa é priodidade" alt="Priodidade" width="30">
              <img src="../statics/delete.png" @click="deleteList(list.id)" alt="delete" width="30">
         </li>
        </ul>
    </section>
    <section v-else>
      <div class="content-list-title">
        <h5 style="color: red;"><b>Você não tem tarefas para hoje, adicione uma se houver.</b></h5>
      </div>
    </section>

    <section>
      <div class="content-list-title">
        <h5><b>Noticias</b></h5>
      </div>
    </section>

    <section v-if="dataNews" class="container-news-list">
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
                <a :href="albums.external_urls.spotify" type="button" class="play-on-spotify" target="_blank">Abrir no Spotify</a>
            </div>
        </li>
      </ul>
      <div class="autorization" v-else>
        <button @click="spotifyLogin()">
          Autorizar Spotify
        </button>
      </div>
    </section>
    <modal name="hello-world" :draggable="true" :adaptive="true" height="auto">
      <div class="modal-container">
        <h5 class="title-modal">Criar uma nova tarefa</h5>
        <form v-on:submit.prevent="createAssignment(assignmentName, assignmentDate)">
          <div class="d-flex">
            <input type="text" placeholder="Nome da tarefa" v-model="assignmentName">
            <div class="d-flex">
              <input type="checkbox" name="urgent" id="urgente" v-model="assignmentAlert">  
              <label for="urgente">Urgente</label>
            </div>
          </div>
          <div class="d-flex">
            <input type="text" v-model="assignmentDate" placeholder="Hora de inicio" v-mask="'##:##'">
            <input type="text" v-model="assignmentDateEnd" placeholder="Hora de Termino" v-mask="'##:##'">
          </div>
          <button type="button" @click="createAssignment(assignmentName, assignmentDate, assignmentDateEnd, assignmentAlert)">Criar tarefa</button>
        </form>
      </div>
    </modal>

    <q-page-container>
      <router-view />
    </q-page-container>
  </q-layout>
</template>

<script>
import Vue from 'vue'
import EssentialLink from 'components/EssentialLink'
import lodash from 'lodash'
import axios from 'axios'
import { Notify } from 'quasar'

export default {
  name: 'MainLayout',
  
   data () {
    return {
      leftDrawerOpen: false,
      dataClime: null,
      dataNews: null,
      open: false,
      tokenAuthSpotify: localStorage.getItem('token'),
      accessTokeSpotify: localStorage.getItem('access_token'),
      backgroundClime: null,
      assignmentName: null,
      assignmentDate: null,
      assignmentPriority: null,
      assignmentAlert: null,
      assignmentDateEnd: null,
      assignmentID: null,
      albunsReleased: null,
      listToDoToday: JSON.parse(localStorage.getItem('assignment')) ? JSON.parse(localStorage.getItem('assignment')).list : []
    }
  },
  mounted() {
    const token = window.location.search.split('code=')[1];
    if(token){
      localStorage.setItem('token', token);
       const config = {
        headers: { 
          'token': token,
        }
    };
    this.$axios.post('http://localhost:3333/spotify', config)
      .then((response) => {
        localStorage.setItem('access_token', response.data.access_token)
    })
    .catch(() => {
      this.$q.notify({
      color: 'negative',
      position: 'top-right',
      message: 'Não é possivel listar dados de albuns no momento. Tente novamente mais tarde!',
      icon: 'report_problem'
    })
  })
    }
    this.$axios.get('http://apiadvisor.climatempo.com.br/api/v1/weather/locale/3477/current?token=8564057d6b6ce399e09795c0fff9883a')
      .then((response) => {
        this.dataClime = response.data;
        this.backgroundClime = { 
        backgroundImage: 'linear-gradient(rgba(0, 0, 0, 0), rgb(247, 247, 247)), url(../statics/background-' +  this.dataClime.data.icon  + '.jpg)'
      }
      })
      .catch(() => {
        this.$q.notify({
          color: 'negative',
          position: 'top-right',
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
          position: 'top-right',
          message: 'Loading failed',
          icon: 'report_problem'
        })
      })
      const config = {
        headers: { Authorization: `Bearer ${ this.accessTokeSpotify }` }
      };
      this.$axios.get('https://api.spotify.com/v1/browse/new-releases?country=BR', config)
      .then((response) => {
        this.albunsReleased = response.data.albums
      })
      .catch(() => {
        this.$q.notify({
          color: 'negative',
          position: 'top-right',
          message: 'Não é possivel listar dados de albuns no momento. Tente novamente mais tarde!',
          icon: 'report_problem'
        })
      })
},

methods: {
  imageLoadError(e) {
    e.target.src = '../statics/No-Image-Found.png'
  },
  spotifyLogin(){
    let redirect = 'http://localhost:8080/#'
    window.location.href = `https://accounts.spotify.com/authorize?client_id=3f00284ce6504784ac44547e33d589d9&response_type=code&redirect_uri=${encodeURIComponent(redirect)}&scope=user-read-private%20user-read-email&state=34fFs29kd09`
  },
  async deleteList(id) {
    const storage = JSON.parse(localStorage.getItem('assignment'));
    const idIsnotEqual = item => item.id !== id;

    const data = {
      list: storage.list.filter(idIsnotEqual)
    }

    localStorage.setItem('assignment', JSON.stringify(data))
    this.listToDoToday = JSON.parse(localStorage.getItem('assignment')).list;
  },
  async createAssignment(assignmentName, assignmentDate, assignmentDateEnd, assignmentAlert){
    if(assignmentName === null || assignmentDate === null) return this.$q.notify({
          color: 'negative',
          progress: true,
          position: 'top-right',
          message: 'Por favor preencha todos os campos!.',
          icon: 'report_problem'
    })
    const assignmentStorage = JSON.parse(localStorage.getItem('assignment')) || { 'list': []};
    const id = Math.floor(1000 + Math.random() * 9000);
    const assigment = {
      assignmentName,
      assignmentDate,
      assignmentDateEnd,
      assignmentAlert,
      id
    }

    assignmentStorage.list.push(assigment)
    localStorage.setItem('assignment', JSON.stringify(assignmentStorage))
    this.hide();
    this.listToDoToday = JSON.parse(localStorage.getItem('assignment')).list;
  },
  show() {
      this.$modal.show('hello-world');
  },
  hide() {
      this.$modal.hide('hello-world');
  }
},
}
</script>

<style>
button {
  cursor: pointer;
}
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
  text-shadow: 1px 1px 7px #000000;
  color: #fff;
}

.container-backdrop .content-lead h4, h2, .color-black {
  color: #fff;
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
  color: #fff;
  font-size: 25px;
  margin-top: 15px;
  text-shadow: 1px 1px 7px #000000;
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

.title-modal {
  font-weight: bold;
  color: #1976d2;
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
  width: 54px;
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
  font-weight: bold;
  color: #333;
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

.container-list-albuns .autorization {
  margin-top: 100px;
  display: flex;
  justify-content: center;
}

.container-list-albuns .autorization button{
  border-radius: 30px;
  color: #fff;
  border: 2px solid #1db954;
  background-color: #1db954;
  font-weight: bold;
  padding: 5px 24px;
  cursor: pointer;
}

.modal-container {
  padding: 15px;
  width: 100%;
  text-align: center;
}

.modal-container form{
  display: flex;
  flex-direction: column;
  width: 90%;
  padding: 15px;
  margin: auto;
}

.modal-container form label {
  font-weight: 500;
  font-size: 15px;
  color: #222326;
}

.modal-container form input, select {
  padding: 10px;
  border-radius: 8px;
  border: 2px solid #eeeeee;
  background-color: #eaeaea;
  width: 100%;
}

.modal-container form input[type="checkbox"] {
  width: 20px;
  position: relative;
  bottom: -5px;
}

.modal-container form * {
  margin: 2px;
}

.modal-container form button {
  background-color: #236cb5;
  color: #eaeaea;
  border-radius: 8px;
  padding: 10px;
  border: none;
  margin-top: 15px;
}

.d-flex {
  display: flex;
}

.verticall {
  flex-direction: column;
}

.verticall p, span {
  color: #ffff;
}

.verticall span {
  font-size: 10px;
}
.verticall p {
  font-weight: bold;
}
</style>