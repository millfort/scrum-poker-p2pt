<script>
import JoinMenu from './components/JoinMenu.vue'
import Room from './components/Room.vue'

import P2PT from "p2pt"

const State = { INITIAL: 0, JOINING: 1, JOINED: 2 }
const RoomState = { VOTING: 1, FINISH: 2 }
const announceURLs = [
  "wss://tracker.openwebtorrent.com",
  "wss://tracker.sloppyta.co:443/announce",
  "wss://tracker.novage.com.ua:443/announce",
  "wss://tracker.btorrent.xyz:443/announce",
]
const appName = 'millfort-scrum-poker-p2pt'
const defaultName = 'Anonymous'

export default {
  components: {
    JoinMenu,
    Room,
  },
  data() {
    return {
      State: State,
      RoomState: RoomState,

      state: State.INITIAL,
      roomState: RoomState.VOTING,

      members: {},
    }
  },
  methods: {
    join(room, username) {
      if (this.state === State.JOINING) {
        return
      }
      console.log('JOIN', room, username)
      this.room = room
      this.username = username
      this.p2pt = new P2PT(announceURLs, appName + room)
      this.peers = {}
      this.members['local'] = { username: username }
      this.state = State.JOINING

      this.p2pt.on('trackerconnect', (tracker, stats) => {
        console.log('TRACKET-CONNECT', tracker, stats)
        this.state = State.JOINED
      })
      this.p2pt.on('trackerwarning', (tracker, stats) => {
        console.log('TRACKET-WARNING', tracker, stats)
      })

      this.p2pt.on('peerconnect', (peer) => {
        console.log('PEER-CONNECT', peer)
        this.peers[peer.id] = peer
        this.members[peer.id] = { username: defaultName }
        this.p2pt.send(peer, JSON.stringify({ method: 'info', username: username, score: this.members['local'].score }))
      })

      this.p2pt.on('peerclose', (peer) => {
        console.log('PEER-CLOSE', peer)
        delete this.peers[peer.id]
        delete this.members[peer.id]
      })

      this.p2pt.on('msg', (peer, msg) => {
        console.log('MSG', peer, msg)
        msg = JSON.parse(msg)
        switch (msg.method) {
          case 'info':
            this.members[peer.id].username = msg.username
            this.members[peer.id].score = msg.score
            break
          case 'score':
            this.members[peer.id].score = msg.score
            break
          case 'next':
            this.roomState = RoomState.VOTING
            for (const id in this.members) {
              this.members[id].score = null
            }
            break
          case 'show':
            this.roomState = RoomState.FINISH
            break
        }
      })

      this.p2pt.start()
    },
    rate(score) {
      this.members['local'].score = score
      for (const id in this.peers) {
        this.p2pt.send(this.peers[id], JSON.stringify({ method: 'score', score: score }))
      }
    },
    next() {
      this.roomState = RoomState.VOTING
      this.members['local'].score = null
      for (const id in this.peers) {
        this.members[id].score = null
        this.p2pt.send(this.peers[id], JSON.stringify({ method: 'next' }))
      }
    },
    show() {
      this.roomState = RoomState.FINISH
      for (const id in this.peers) {
        this.p2pt.send(this.peers[id], JSON.stringify({ method: 'show' }))
      }
    },
  },
}
</script>

<template>
  <main>
    <JoinMenu @join="join" v-if="state === State.INITIAL"></JoinMenu>
    <Room @rate="rate" @next="next" @show="show" :State="RoomState" :state="roomState" :members="members" v-if="state === State.JOINED">
    </Room>
  </main>
</template>
