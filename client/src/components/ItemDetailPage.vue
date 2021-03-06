<template>
  <div
    v-if="data"
    class="d-flex flex-wrap"
  >
    <v-container>
      <v-row>
        <v-col
          :cols="12"
          :sm="4"
          :lg="3"
          :xl="2"
        >
          <div class="mb-4 cover-container">
            <img class="cover"
              :src="data.imgUrl.length ? data.imgUrl : '/img/no-image.svg'"
            />
          </div>
          <div>
            <WannaRental v-if="data.type === 0" @reload="reload" :propItem="data" @checkRentalable="checkRentalable"/>
            <RentalForm @reload="reload" :propItem="data" @checkRentalable="checkRentalable" :disable="!checkRental"/>
            <ReturnForm @reload="reload" :propItem="data"/>
            <v-btn color="error" block v-if="checkOwnOrAdmin" @click="destroyItem" error>削除</v-btn>
          </div>
          <div>
            <v-btn v-if="isLiked" block @click="removeLike" class="my-1">
              <mdi-icon left color="indigo" name="mdi-thumb-up" />
              いいね {{ likeCount }}
            </v-btn>
            <v-btn v-else block @click="like" class="my-1">
              <mdi-icon left disabled name="mdi-thumb-up" />
              いいね {{ likeCount }}
            </v-btn>
          </div>
          <v-container class="pa-0">
            <v-row row wrap no-gutters>
              <v-col class="ma-0 flex-grow-0 flex-shrink-0" no-gutter v-for="like in data.likes" :key="like.id" >
                <Icon
                  :user="like"
                  :size="25"
                />
              </v-col>
            </v-row>
          </v-container>
        </v-col>
        <v-col
          :cols="12"
          :sm="8"
          :lg="9"
          :xl="10"
        >
          <h1>{{data.name}}</h1>
          <div class="mb-8">
            {{ data.description }}
          </div>
          <div class="mb-8">
            <h2>
              所有者
              <RegisterOwnerForm @reload="reload"/>
            </h2>
            <!-- FIXME: 他のタスクに手をつけたかったので表示が適当です -->
            <div v-for="owner in data.owners" :key="owner.id">
              <Icon
                :user="owner.user"
                :size="25"
              />
              {{ owner.user.name }} {{ checkRentalable(owner) }}
              <EditOwner
                :itemID="data.id"
                :propOwner="owner"
                :propLatestLogs="data.latestLogs"
                @reload="reload"
              />
            </div>
          </div>
          <div class="mb-4">
            <h2>
              コメント
              <CommentDialog :propItem="data"/>
            </h2>
            <v-list v-if="data.comments.length" color="transparent">
              <v-list-item v-for="comment in data.comments" :key="comment.id" class="pl-0">
                <v-list-item-avatar>
                  <Icon :user="comment.user" :size="40" />
                </v-list-item-avatar>
                <v-list-item-content>
                  {{ comment.text }}
                </v-list-item-content>
              </v-list-item>
            </v-list>
            <div v-else>
              コメントがありません
            </div>
          </div>
          <div class="mb-8">
            <div>
              <h2>ログ</h2>
              <div v-if="data.logs.length">
                <div v-for="log in reverseLogs" :key="log.id">
                  <Icon
                    :user="log.user"
                    :size="25"
                  />
                  {{ createLogMessage(log) }}
                </div>
              </div>
              <div v-else>
                ログがありません
              </div>
            </div>
          </div>
        </v-col>
      </v-row>
    </v-container>
  </div>
</template>

<script>
import Icon from './shared/Icon'
import axios from 'axios'
import RegisterOwnerForm from './shared/RegisterOwnerForm'
import RentalForm from './shared/RentalForm'
import CommentDialog from './shared/CommentDialog'
import ReturnForm from './shared/ReturnForm'
import EditOwner from './shared/EditOwner'
import WannaRental from './shared/WannaRental'
import MdiIcon from './shared/MdiIcon.vue'

export default {
  name: 'ItemDetailPage',
  components: {
    Icon,
    RegisterOwnerForm,
    RentalForm,
    CommentDialog,
    ReturnForm,
    EditOwner,
    WannaRental,
    MdiIcon
  },
  data () {
    return {
      data: null,
      contentWidth: 600,
      imageWidth: 250
    }
  },
  created () {
    axios
      .get('/api/items/' + this.$route.params.id)
      .then(res => {
        this.data = res.data
        this.$store.commit('setNavBarTitle', res.data.name)
      })
      .catch(e => { alert(e) })
  },
  mounted () {
    this.conputeWidth()
    window.addEventListener('resize', this.conputeWidth)
  },
  beforeDestroy () {
    this.$store.commit('resetNavBarTitle')
    window.removeEventListener('resize', this.conputeWidth)
  },
  computed: {
    reverseLogs () {
      return this.data.logs.slice().reverse()
    },
    likeCount () {
      return this.data.likes.length
    },
    isLiked () {
      if (!this.$store.state.me) {
        return false
      }
      return this.data.likes.find(user => user.name === this.$store.state.me.name)
    },
    checkOwnOrAdmin () {
      if (this.data.owners) {
        const owner = this.data.owners.find(element => {
          return element.ownerId === this.$store.state.me.id
        })
        if (owner || this.$store.state.me.admin) {
          return true
        }
      }
      return false
    },
    checkRental () {
      return this.data.owners.some(owner => {
        if (!owner.rentalable) {
          return false
        }
        const latestLog = this.data.latestLogs.find(log => {
          return log.owner.id === owner.ownerId
        })
        const rentalableCount = latestLog ? latestLog.count : owner.count
        return rentalableCount !== 0
      })
    }
  },
  methods: {
    conputeWidth () {
      if (window.innerWidth > 991) {
        this.contentWidth = window.innerWidth - 600 // sideBar((window.innerWidth > 991で表示される)と物品のimgがともに260px
      } else if (window.innerWidth > 601) {
        this.contentWidth = window.innerWidth - 300
      } else {
        this.contentWidth = window.innerWidth - 30
      }
    },
    checkRentalable (owner) {
      // FIXME: ロジックがやばい
      if (!owner.rentalable) {
        return '貸し出しできません'
      }
      const latestLog = this.data.latestLogs.find(log => {
        return log.owner.id === owner.ownerId
      })
      let rentalableCount = 0
      if (latestLog) {
        rentalableCount = latestLog.count
      } else {
        rentalableCount = owner.count
      }
      if (rentalableCount === 0) {
        return '現在すべて貸しだし中'
      } else if (rentalableCount === 1) {
        return '貸し出し可能'
      }
      return '貸し出し可能' + '×' + rentalableCount + ' /' + owner.count
    },
    createLogMessage (log) {
      const userName = log.user.name
      const ownerName = log.owner.name
      let ownerWord = ownerName === 'traP' ? '' : `${ownerName}さんの`
      let logComment = log.type === 0 ? '借りました' : '返しました'
      if (log.type === 2) {
        ownerWord = ''
        logComment = '追加しました'
      }
      if (log.type === 3) {
        ownerWord = ''
        logComment = '減らしました'
      }
      const logTime = log.createdAt.replace('T', ' ').replace('+09:00', '')
      return `${userName}さんが${ownerWord}物品を${logComment} - ${logTime}`
    },
    async like () {
      let postLikeError = null
      await axios.post('/api/items/' + this.$route.params.id + '/likes', null)
        .catch(e => {
          alert(e)
          postLikeError = e
        })
      if (!postLikeError) {
        this.data.likes.push(this.$store.state.me)
      }
    },
    async removeLike () {
      let removeLikeError = null
      await axios.delete('/api/items/' + this.$route.params.id + '/likes', null)
        .catch(e => {
          alert(e)
          removeLikeError = e
        })
      if (!removeLikeError) {
        this.data.likes = this.data.likes.filter(user => user.name !== this.$store.state.me.name)
      }
    },
    async reload () {
      const res = await axios.get('/api/items/' + this.$route.params.id).catch(e => { alert(e) })
      this.data = res.data
    },
    async destroyItem () {
      const result = window.confirm('本当に削除しますか？')
      if (result === true) {
        await axios.delete('/api/items/' + this.$route.params.id).catch(e => { alert(e) })
        this.$router.push({ path: '/items' })
      }
    }
  }
}
</script>

<style scoped>
.cover {
  width: 100%;
}
.cover-container {
  display: flex;
  justify-content: center;
}
</style>
