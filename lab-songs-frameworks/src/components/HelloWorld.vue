<template>
  <div >
    <main>
      <header>
        <h3> Angie - Play</h3>
      </header>
      <!-- CONTROLADORES -->
      <input placeholder="Busca a tu artista" v-model="search" @keyup.enter="createArtist()">
      <!-- SELECT AN ARTIST -->
      <select v-model="selectedArtist">
        <option v-for="(artist, firstIndex) in artists">{{artist}}</option>
      </select>
      <!-- LIMITAR # DE CANCIONES -->
      <input type="number" step=1 v-model="input_limit">
    </main>
    <h1>{{selectedArtist}}</h1>
    <div v-for="(artist, artistIndex) in localArtists">
      <div v-for="(song, songIndex) in artist" v-show="song.artist['#text'] == selectedArtist">
        <div class="container">
          <p>Album: {{song.album['#text']}}</p>
          <div>
            <img :src="song.image[2]['#text']">
          </div>
          <p><strong>{{song.name}}</strong></p>
          <h3>{{ song.like }}</h3>
          <div>
            <i class="fa fa-hand-o-up" @click="like(song, artistIndex, songIndex)"></i>
            <i class="fa fa-hand-o-down" @click="dislike(artist, artistIndex, songIndex)"></i>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios'
export default {
  name: 'SongsList',
  data () {
    return {
      // original: 'http://ws.audioscrobbler.com/2.0/?method=user.getartisttracks&user=rj&artist=adele&api_key=9477be879cb8ef4d370328ecdb0e4739&format=json&limit=10',
      search: '',
      selectedArtist: 'Sia',
      input_format: 'json',
      input_limit: 10,
      message: '',
      promises: [],
      localArtists: []
    }
  },
  watch: {
    // Watch the next variables
    input_limit () {
      this.limit = 'limit=' + this.input_limit
    },

    input_format () {
      this.format = 'format=' + this.input_format
    }
  },
  created () {
    // Predefined data
    this.endpoint = 'http://ws.audioscrobbler.com/2.0/'
    this.user = 'user=rj'
    this.api_key = 'api_key=9477be879cb8ef4d370328ecdb0e4739'
    this.format = 'format=' + this.input_format
    this.limit = 'limit=' + this.input_limit
    this.artist = 'artist='
    this.artists = ['Adele', 'Michael Jackson', 'Shakira', 'Maroon 5', 'The Beatles', 'Sia']
    this.meth = 'method=user.getartisttracks'
    this.artists.forEach(ar => {
      this.promises.push(axios.get(`${this.endpoint}?${this.meth}&${this.user}&artist=${ar}&${this.api_key}&${this.format}&${this.limit}`))
    })
    // Get the initial artists of in the promises array
    this.executePromises()
  },
  methods: {
    // Get a new artist not exist currently
    createArtist () {
      // API method
      let ar = this.artist + this.search
      // Build the url for call from axios
      let url = `${this.endpoint}?${this.meth}&${this.user}&${ar}&${this.api_key}&${this.format}&${this.limit}`
      Promise.all([axios.get(url)])
        .then(res => {
          // Push the data into localArtists
          this.localArtists.push(res[0].data.artisttracks.track)
          // Add the 'like' property into the songs
          // given the last index in the localArtist
          this.addPropertyLike(this.localArtists.length - 1)
          // Add the artist in the selection list
          this.artists.push(this.search)
        })
        .catch(err => console.log(`Error, artist not found: ${err}`))
      // Change the selected artist artist view
      this.selectedArtist = this.search
    },
    // Execute the promeses array
    executePromises () {
      let count = 0
      Promise.all(this.promises).then(url => {
        url.forEach((el, index) => {
          count++
          this.localArtists.push(el.data.artisttracks.track)
        })
        for (let i = 0; i < count; i++) {
          this.addPropertyLike(i)
        }
      })
    },
    // Add the property like
    // It adds the property 'like', search with the index parameter and iterates over the songs of this index
    addPropertyLike (index) {
      console.log(`Adding 'like' property for the singer songs: ${this.localArtists[index][0].artist['#text']}`)
      for (let j = 0; j < this.localArtists[index].length; j++) {
        this.localArtists[index][j].like = 0
        // console.log(this.localArtists[index][j].name)
      }
    },
    // It makes a count for the given song
    // param: It is the value of the current song, when the user make a click
    // indexArtist: Represent the index artist
    // songIndex: Represent the index song
    like (param, indexArtist, songIndex) {
      this.localArtists[indexArtist][songIndex].like = this.localArtists[indexArtist][songIndex].like + 1
      this.$forceUpdate()
    },
    dislike (param, indexArtist, songIndex) {
      this.localArtists[indexArtist][songIndex].like >= 1 ? this.localArtists[indexArtist][songIndex].like = this.localArtists[indexArtist][songIndex].like - 1 : null
      this.$forceUpdate()
    }
  }
}
</script>
<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.auto{
  float: left
}
h1, h2 {
  font-weight: normal;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
i{
  font-size: 30px
}
</style>
