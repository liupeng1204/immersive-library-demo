<template>
  <div class="app">
    <div id="immersive-frame"></div>
    <div class="controls_container">
      <div class="controls">
        <form @submit.prevent.stop="initLibrary">
          <input v-model="userName" required type="text" name="userName" placeholder="user name"/>
          <input v-model="userId" required type="text" name="userId" placeholder="user id"/>
          <button type="submit">initialize</button>
        </form>
        <form @submit.prevent.stop="enterRoom">
          <input v-model="roomId" required type="text" name="roomId" placeholder="room id"/>
          <button :disabled="!library" >enter room</button>
        </form>
        <form @submit.prevent.stop="changeContent">
          <input v-model="contentData" required type="text" name="contentData" placeholder="content data"/>
          <select v-model="contentType" name="contentType">
            <option value="forge">forge</option>
            <option value="matterport">matterport</option>
          </select>
          <button :disabled="!library || !this.contentType || !this.contentData" >change content</button>
        </form>

        <button @click="destroy" :disabled="!library" class="red">destroy</button>

        <a :href="newSessionLink" target="_blank">Open other session</a>

        <hr />
        <p>Room status:</p>
        <pre class="room_status">{{ roomStatus }}</pre>
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
import ImmersiveLibrary from '@superviz/immersive-library'

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
    amountOfParticipants: ''
  }),
  computed: {
    newSessionLink() {
      return `/?roomId=${this.roomId}`
    },
    roomStatus() {
      return `current state: ${this.currentState}
current state reason: ${this.currentStateReason}
host user id: ${this.currentHostUserId}
amount of participants: ${this.amountOfParticipants}`
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
        'lnt7va99nvbp2wqp5ruev9k0gj9hjw',
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
      this.library.subscribeToRealtimeState(this.onRealtimeState);
      this.library.subscribeToRoomInfoUpdated(this.onRoomInfoUpdated);
    },
    enterRoom() {
      this.library.enterRoom(this.roomId)
    },
    changeContent() {
      this.library.changeContent(this.contentType, this.contentData)
    },
    destroy() {
      this.library.destroy()
    },
    addMessageToLog(message) {
      this.eventLog += `${message}\n`
      this.$nextTick(() => {
        this.$refs.eventLog.scrollTop = this.$refs.eventLog.scrollHeight
      })
    },
    onUserJoined(user) {
      this.addMessageToLog(`User joined: ${user.name} ${user.userId}`)
    },
    onUserLeave(user) {
      this.addMessageToLog(`User left: ${user.name} ${user.userId}`)
    },
    onAmountOfParticipants(amountOfParticipants) {
      this.amountOfParticipants = amountOfParticipants
    },
    onHostChange({ newHostUserId, oldHostUserId }) {
      this.currentHostUserId = newHostUserId
    },
    onRealtimeState(state, reason) {
      this.currentState = state
      this.currentStateReason = reason ?? 'none'
      this.addMessageToLog(`State changed to: ${state} - ${reason}`)
    },
    onRoomInfoUpdated(roomInfo) {
      this.addMessageToLog(`Room info updated: ${JSON.stringify(roomInfo, null, '  ')}`)
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

  .controls
    display: flex
    flex-direction: column
    grid-area: controls

  .events
    display: flex
    flex-direction: column
    grid-area: logs

    pre
      overflow-x: auto
      overflow-y: scroll
      width: $controls-width
      height: 500px
</style>