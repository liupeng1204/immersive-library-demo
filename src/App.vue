<template>
  <div class="app">
    <div id="immersive-frame"></div>
    <div class="controls_container">
      <div class="controls">
        <form @submit.prevent.stop="initLibrary">
          <input v-model="userName" required type="text" name="userName" placeholder="user name"/>
          <input v-model="userId" required type="text" name="userId" placeholder="user id"/>
          <button :disabled="library" type="submit">initialize</button>
        </form>
        <form @submit.prevent.stop="enterRoom">
          <input v-model="roomId" required type="text" name="roomId" placeholder="room id"/>
          <button :disabled="!library || currentState !== 2" >enter room</button>
        </form>
        <form @submit.prevent.stop="changeContent">
          <input v-model="contentData" required type="text" name="contentData" placeholder="content data"/>
          <select v-model="contentType" name="contentType">
            <option value="forge">forge</option>
            <option value="matterport">matterport</option>
            <option value="sketchfab">sketchfab</option>
            <option value="pureweb">pureweb</option>  
          </select>
          <button :disabled="!library || !this.contentType || !this.contentData" >change content</button>
        </form>

        <form @submit.prevent.stop="changeHost">
          <select v-model="newHostId" :disabled="!participants.length">
            <option v-for="participant in participants" :value="participant.externalUserId" :key="participant.externalUserId">
              {{ participant.name }}
            </option>
          </select>
          <button :disabled="!participants.length"> change host </button>
        </form>

        <form @submit.prevent.stop="setFollow">
          <select v-model="followerId" :disabled="!participants.length">
            <option :value="null"> - </option>
            <option v-for="participant in participants" :value="participant.externalUserId" :key="participant.externalUserId">
              {{ participant.name }}
            </option>
          </select>
          <button :disabled="!participants.length"> set follow </button>
        </form>

        <form @submit.prevent.stop="goToUser">
           <select v-model="goToUserId" :disabled="!participants.length">
            <option :value="null"> - </option>
            <option v-for="participant in participants" :value="participant.externalUserId" :key="participant.externalUserId">
              {{ participant.name }}
            </option>
          </select>
          <button :disabled="!participants.length"> go to </button>
        </form>

        <button @click="gather" :disabled="!isHost">gather</button>
        <button @click="destroy" :disabled="!library" class="red">destroy</button>

        <a :href="newSessionLink" target="_blank">Open other session</a>

        <hr />
        <p>Library status:</p>
        <pre>{{ libraryStatus }}</pre>
        <hr />
      </div>
      <div class="events">
        <p>Event log:</p>
        <pre ref="eventLog">{{ eventLog }}</pre>
      </div>
    </div>
  </div>
</template>

<script>
import ImmersiveLibrary, { LibraryStateTypes, LibraryStateReasonTypes } from '@superviz/immersive-library'

const DEVELOPER_KEY = import.meta.env.VITE_SUPERVIZ_DEVELOPER_KEY
export default {
  data: () => ({
    library: null,
    roomId: '',
    userId: '',
    userName: '',
    contentType: '',
    contentData: '',
    eventLog: '',
    currentState: '',
    currentStateReason: '',
    currentHostUserId: '',
    amountOfParticipants: '',

    participants: [],

    newHostId: '',
    followerId: '',
    goToUserId: '',
  }),
  computed: {
    newSessionLink() {
      return `/?roomId=${this.roomId}`
    },
    libraryStatus() {
      return `state: ${LibraryStateTypes[this.currentState] ?? 'unknown'}
state reason: ${LibraryStateReasonTypes[this.currentStateReason] ?? 'none'}
host user id: ${this.currentHostUserId}
participants: ${this.amountOfParticipants}`
    },
    isHost() {
      return this.currentHostUserId === this.userId;
    }
  },
  mounted() {
    const currentUrl = new URL(window.location.href)

    this.roomId = currentUrl.searchParams.get('roomId') ?? ''
    this.userName = currentUrl.searchParams.get('userName') ?? ''
    this.userId = currentUrl.searchParams.get('userId') ?? ''
  },
  methods: {
    async initLibrary() {
      this.library = await ImmersiveLibrary(
        DEVELOPER_KEY,
        {
          wrapperId: 'immersive-frame',
          language: 'en',
          debug: true,
          userInfo: {
            externalUserId: this.userId,
            name: this.userName
          },
          region: 'us'
        }
      );

      this.library.subscribeToUserJoined(this.onUserJoined);
      this.library.subscribeToUserLeave(this.onUserLeave);
      this.library.subscribeToAmountOfParticipants(this.onAmountOfParticipants);
      this.library.subscribeToHostChange(this.onHostChange);
      this.library.subscribeToStateUpdate(this.onStateChange);
      this.library.subscribeToRoomInfoUpdated(this.onRoomInfoUpdated);
    },
    enterRoom() {
      this.library.enterRoom(this.roomId)
    },
    changeContent() {
      this.library.changeContent(this.contentType, this.contentData)
    },
    changeHost() {
      this.library.changeHost(this.newHostId)
    },
    setFollow() {
      this.library.followUser(this.followerId)
    },
    gather() {
      this.library.gatherAll()
    },
    goToUser() {
      this.library.goToUser(this.goToUserId)
    },
    destroy() {
      this.library.destroy()

      this.library = null
      this.contentType = ''
      this.contentData = ''
      this.eventLog = ''
      this.currentState = ''
      this.currentStateReason = ''
      this.currentHostUserId = ''
      this.amountOfParticipants = ''
    },
    addMessageToLog(message) {
      this.eventLog += `${message}\n`
      this.$nextTick(() => {
        this.$refs.eventLog.scrollTop = this.$refs.eventLog.scrollHeight
      })
    },
    onUserJoined(user) {
      this.addMessageToLog(`User joined: ${user.name} ${user.externalUserId}`)

      
      this.participants = this.library.users
    },
    onUserLeave(user) {
      this.addMessageToLog(`User left: ${user.name} ${user.externalUserId}`)
      this.participants = this.library.users
    },
    onAmountOfParticipants(amountOfParticipants) {
      this.amountOfParticipants = amountOfParticipants
    },
    onHostChange({ newHostUserId, oldHostUserId }) {
      this.currentHostUserId = newHostUserId
    },
    onStateChange(state, reason) {
      this.currentState = state
      this.currentStateReason = reason
      this.addMessageToLog(`State changed to: ${state}, ${reason}`)
    },
    onRoomInfoUpdated(roomInfo) {
      this.addMessageToLog(`Room info updated: ${JSON.stringify(roomInfo, null, ' ')}`)
    },
  }
}
</script>

<style lang="sass">

$controls-width: 250px

.app
  height: 100%
  display: grid
  grid-template-areas: "frame info"
  grid-template-columns: auto $controls-width
  column-gap: 5px

#immersive-frame
  grid-area: frame

  iframe
    width: 100%
    height: 100%

.controls_container
  grid-area: info
  display: grid
  grid-template-areas: "controls" "logs"
  grid-template-rows: auto 1fr

  pre
    overflow: auto
    width: $controls-width

  .controls
    display: flex
    flex-direction: column
    grid-area: controls

  .events
    display: flex
    flex-direction: column
    grid-area: logs

    pre
      height: 300px
</style>