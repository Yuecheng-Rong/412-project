<template>
  <div>
    <!-- settle the size of the inside container -->
    <!-- If no members are current, show nothing -->
    <v-container v-if="currentMember.uid" fluid class="insideContainer">
      <v-layout row wrap>
        <v-flex xs12 class="ScrollStyle">
        <!-- 当一般情况下，显示单人的卡片 -->
          <div v-show="!currentShowingLeader">
            <h3 style="text-align:center"> {{ currentMember.name }} </h3>
            <br>
            <div
            v-for="(cards,cardsIndex) in currentCards"
            :key="cardsIndex"
            v-show="cards.finished === currentShowingFinished">
              <cards v-bind:card="cards"/>
              <br>
            </div>
            <div style="text-align:center">
              <v-btn v-on:click="addCard()" v-show="!currentShowingFinished && currentGroup.groupLeader === currentUser || currentMember.uid === currentUser" dark color="blue-grey darken-2" ripple round>
                <v-icon>add</v-icon>
              </v-btn>
            </div>
          </div>
          <!-- 当leader按钮被按下后，显示该组内所有人卡片 -->
          <!-- sort仅对currentCards有效，对leader无效 -->
          <div
          v-show="currentShowingLeader">
          <v-container fluid grid-list-md>
            <v-layout row wrap>
              <v-flex xs12 class="ScrollStyle"
              v-for="(cards,cardsIndex) in currentCards"
              :key="cardsIndex"
              v-show="cards.finished === currentShowingFinished">
                <h3 style="text-align:center"> {{ cards.ownerName }} </h3>
                <br>
                <cards v-bind:card="cards"/>
                <br>
              </v-flex>
            </v-layout>
          </v-container>
          </div>
        </v-flex>
      </v-layout>
    </v-container>

    <!-- add card dialog -->
    <v-dialog
    v-model="currentAddingCards"
    max-width="500px">
      <v-card>
        <v-layout
          column>
          <v-flex xs12>
            <v-card>
              <v-layout column>
                <v-flex xs10>
                  <v-card-title primary-title>
                    <v-layout column>
                      <div class="headline">
                        Task Title:
                        <span v-if="refreshThePage"></span>
                        <v-text-field  v-model="newCard.name" required clearable=""></v-text-field>
                      </div>
                      <br>
                      <div class="headline">
                        Description:
                        <v-text-field  v-model="newCard.description" required clearable=""></v-text-field>
                      </div>
                      <br>
                      <div class="headline">
                        Due Date:
                        <v-text-field  v-model="newCard.dueDate" hint="format: yyyy-mm-dd" required clearable=""></v-text-field>
                      </div>
                      <br>
                      <div class="headline">
                        Done by:
                        <v-text-field  v-model="newCard.dueTime" hint="format: hh:mm" required clearable=""></v-text-field>
                      </div>
                      <br>
                      <div class="headline">
                        Importance:
                        <v-btn
                          icon
                          @click="changeImportance(1)">
                          <v-icon
                            color="red"
                            v-if="newCard.importance > 0">
                          star</v-icon>
                          <v-icon
                            color="red"
                            v-if="newCard.importance < 1">
                          star_border</v-icon>
                        </v-btn>
                        <v-btn
                          icon
                          @click="changeImportance(2)">
                          <v-icon
                            color="red"
                            v-if="newCard.importance > 1">
                          star</v-icon>
                          <v-icon
                            color="red"
                            v-if="newCard.importance < 2">
                          star_border</v-icon>
                        </v-btn>
                        <v-btn
                          icon
                          @click="changeImportance(3)">
                          <v-icon
                            color="red"
                            v-if="newCard.importance > 2">
                          star</v-icon>
                          <v-icon
                            color="red"
                            v-if="newCard.importance < 3">
                          star_border</v-icon>
                        </v-btn>
                        <v-btn
                          icon
                          @click="changeImportance(4)">
                          <v-icon
                            color="red"
                            v-if="newCard.importance > 3">
                          star</v-icon>
                          <v-icon
                            color="red"
                            v-if="newCard.importance < 4">
                          star_border</v-icon>
                        </v-btn>
                        <v-btn
                          icon
                          @click="changeImportance(5)">
                          <v-icon
                            color="red"
                            v-if="newCard.importance > 4">
                          star</v-icon>
                          <v-icon
                            color="red"
                            v-if="newCard.importance < 5">
                          star_border</v-icon>
                        </v-btn>
                      </div>
                    </v-layout>
                  </v-card-title>
                </v-flex>

                <v-card-actions>
                  <v-spacer></v-spacer>
                  <v-flex/>
                    <v-btn color="success" @click="confirmAddingThisCard">SUBMIT</v-btn>
                    <v-btn color="error" @click="currentAddingCards=false">Close</v-btn>
                </v-card-actions>
              </v-layout>

            </v-card>
          </v-flex>
        </v-layout>
      </v-card>
    </v-dialog>

  </div>
</template>

<script>
import { mapGetters } from 'vuex'
import Cards from '@/components/Cards'
import firebase from 'firebase'
export default {
  components: { Cards },
  /**
   * 在本component中，所有的数据均来自store里的current家族
   * 存储修改也是直接call setCurrent
   */
  name: 'MainPage',
  data() {
    return {
      // group
      newgroup: {
        name: '',
        id: '',
        groupLeader: '',
        members: []
      },
      // members
      newMember: {
        name: '',
        id: '',
        uid: '',
        cards: []
      },
      // cards
      newCard: {
        id: '',
        name: '',
        description: '',
        dueDate: '',
        dueTime: '',
        importance: '',
        addTime: '',
        ownerUid: '',
        ownerName: '',
        ownerIDInGroup: '',
        finished: false
      },
      // open/close the adding card dialog
      currentAddingCards: false,
      // refresh the page
      refreshThePage: false
    }
  },
  // 凡是和store有关的东西都在这里
  // 理论上他应该是自动更新的
  computed: {
    currentGroup() {
      if (this.$store.getters.getCurrentGroup){
        return this.$store.getters.getCurrentGroup
      } else {
        return { groupLeader: ''}
      }
    },
    currentMember() {
      if (this.$store.getters.getCurrentMember){
        return this.$store.getters.getCurrentMember
      } else {
        return { name: '' }
      }
    },
    currentCards() {
      if (this.$store.getters.getCurrentCards){
        return this.sortTheCards(this.$store.getters.getCurrentCards)
      } else {
        return null
      }
    },
    // 这个开关决定了是否按下了leader按钮
    currentShowingLeader() {
      return this.$store.getters.getLeaderButtonPushed
    },
    // 这个开关决定了是展示完成的tasks还是未完成的tasks
    currentShowingFinished() {
      return this.$store.getters.getFinish
    },
    currentUser() {
      return firebase.auth().currentUser.uid
    }
  },
  watch: {
    /**
    '$route' (to, from) {

    }
    currentGroup (newer, older) {

    },
    currentMember (newer, older) {

    },
    currentCards (newer, older) {

    },
    */
  },
  methods: {
    addCard() {
      this.clearNewCard()
      this.currentAddingCards = true
    },
    confirmAddingThisCard() {
      var toStore = []
      toStore[0] = this.newCard
      toStore[1] = this.currentMember
      this.$store.dispatch('addcard', toStore)
      this.addCard()
      this.currentAddingCards = false
    },
    // Sort the cards depending on which sort user wants
    sortTheCards(cardsToBeSort) {
      const sortType = this.$store.getters.getSortType
      const antisort = this.$store.getters.getAntiSort
      // 由于不能直接对currentCards进行sort，
      // 要将其先置于一个可以sort的地方
      var tempCurrentCards = []
      for (var cards in cardsToBeSort){
        // 读取每张currentCards并将他塞入一个新建的array
        tempCurrentCards[tempCurrentCards.length] = cardsToBeSort[cards]
      }
      // compare based on the sort type
      // pay attention to anti-sort or not!
      function compare (a, b) {
        if (sortType === 1) {
          // sort by importance
          if (a.importance > b.importance) {
            if (antisort) { return 1 } else { return -1 }
          }
          if (a.importance < b.importance) {
            if (antisort) { return -1 } else { return 1 }
          }
        } else if (sortType === 2) {
          // sort by due date
          if (new Date(a.dueDate + ' ' + a.dueTimee) < new Date(b.dueDate + ' ' + b.dueTime)) {
            if (antisort) { return 1 } else { return -1 }
          }
          if (new Date(a.dueDate + ' ' + a.dueTime) > new Date(b.dueDate + ' ' + b.dueTime)) {
            if (antisort) { return -1 } else { return 1 }
          }
        } else if (sortType === 0) {
          // sort by create time
          if (a.addTime < b.addTime) {
            if (antisort) { return 1 } else { return -1 }
          }
          if (a.addTime > b.addTime) {
            if (antisort) { return -1 } else { return 1 }
          }
        } else if (sortType === 3) {
          // Leader form
          // do nothing
        }
        return 0
      }
      tempCurrentCards.sort(compare)
      return tempCurrentCards
    },
    // Define the importance of the new card
    changeImportance: function(payload) {
      this.newCard.importance = payload
      this.refreshThePage = !this.refreshThePage
    },
    /**
     * 清除
     */
    clearNewGroup: function() {
      this.newgroup = {
        name: '',
        id: '',
        groupLeader: '',
        members: []
      }
    },
    clearNewMember: function() {
      this.newMember = {
        name: '',
        id: '',
        uid: '',
        cards: []
      }
    },
    clearNewCard: function() {
      this.newCard =  {
        id: '',
        name: '',
        description: '',
        dueDate: '',
        dueTime: '',
        importance: '',
        addTime: '',
        ownerUid: '',
        ownerName: '',
        ownerIDInGroup: '',
        finished: false
      }
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
  .insideContainer {
    position: absolute;
    width: 80%;
    left: 80px;
    top: 125px;
    max-height: 600px;
    background-color:lightgrey
  }
  .ScrollStyle  {
    max-height: 550px;
    overflow-y: scroll;
  }
</style>
