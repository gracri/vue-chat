<template>
    <div id="chat">
        <input type="text" v-model.trim="messageInput" @keyup.enter="send(messageInput)">
    </div>
</template>

<script>
import * as Firebase from 'firebase'
import Geohash from 'latlon-geohash'

export default {
  name: 'chat',
  data () {
    return {
      room: null,
      precision: 6, // default precision
      db: null, // assign Firebase SDK later
      messageInput: ''
    }
  },
  mounted () {
    this.db = Firebase.initializeApp({
      apiKey: '',
      authDomain: 'chat-proj-e47b3.firebaseapp.com',
      databaseURL: 'https://chat-proj-e47b3.firebaseio.com',
      storageBucket: 'chat-proj-e47b3.appspot.com',
      messagingSenderId: '657098277792'
    })

    this.init()
  },
  methods: {
    init() {
      if (navigator.geolocation) {
        navigator.geolocation.getCurrentPosition((position) => {
          var geohash = Geohash.encode(position.coords.latitude, position.coords.longitude, this.precision);

          // initilize the room based on geohash
          this.room = this.db.database().ref().child('rooms/' + geohash)
        }, (err) => {
          // error handling here
        })
      } else {
        console.error('Cannot access geolocation')
      }
    },
    send(messageInput) {
      // A data entry.
      let data = {
        message: messageInput
      };

      // Get a key for a new message.
      let key = this.room.push().key;
      this.room.child('messages/' + key).set(data)

      // clean the message
      this.messageInput = ''
    }
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
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
</style>
