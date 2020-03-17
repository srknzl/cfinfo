<template>
  <view class="container">
    <TouchableOpacity class="toggleButton" :onPress="() => {showRating = !showRating}">
      <Text class="whiteText">Toggle View</Text>
    </TouchableOpacity>
    <view v-if="showRating">
      <view class="container" v-if="ratingHistory.length != 0">
        <text class="blackText">Rating History for {{ratingHistory[0].handle}}</text>
        <scroll-view class="ratingScroll">
          <view class="card" v-for="c in ratingHistory" :key="c.contestId">
            <text class="blackText">Rank: {{c.rank}}</text>
            <text class="blackText">Old Rating: {{c.oldRating}}</text>
            <text class="blackText">New Rating: {{c.newRating}}</text>
            <text
              :style="c.newRating - c.oldRating >= 0 ? {color: 'green'} : {color: 'red'}"
            >Change: {{c.newRating - c.oldRating}}</text>
            <text class="blackText">Contest Name: {{c.contestName}}</text>
            <text
              class="blackText"
            >Rating Change Date: {{getTimeFromUnixTime(c.ratingUpdateTimeSeconds)}}</text>
          </view>
        </scroll-view>
      </view>
      <text
        v-if="ratingHistory.length == 0"
      >Enter your handle in the previous page to fetch your rating history.</text>
    </view>
    <view class="container" v-if="!showRating">
      <view class="paginationButtons">
        <TouchableOpacity
          class="customButton"
          :style="contestPageNumber==0 ? {backgroundColor: 'grey'} : {}"
          :disabled="contestPageNumber==0"
          :onPress="prevContestPage"
        >
          <Text class="whiteText">Previous</Text>
        </TouchableOpacity>
        <TouchableOpacity
          class="customButton"
          :disabled="contestPageNumber >= constestPageCount -1"
          :onPress="nextContestPage"
          :style="contestPageNumber >= constestPageCount -1 ? {backgroundColor: 'grey'} : {}"
        >
          <Text class="whiteText">Next</Text>
        </TouchableOpacity>
      </view>
      <view class="contests">
        <text class="blackText">Upcoming contests:</text>
        <scroll-view>
          <view
            class="card"
            v-for="c in paginatedContests"
            :key="c.id"
            :style="c.phase == 'BEFORE' ? {}: {backgroundColor: 'pink'}"
          >
            <text class="blackText">{{c.name}}</text>
            <text class="blackText">{{getTimeFromUnixTime(c.startTimeSeconds)}}</text>
            <text class="blackText">{{getTimeFromUnixTime(c.startTimeSeconds+c.durationSeconds)}}</text>
          </view>
        </scroll-view>
      </view>
      <view class="handleRating">
        <TextInput
          class="ratingHandleInput"
          v-model="ratingHandle"
          placeholder="Handle"
          placeholderTextColor="grey"
        />
        <TouchableOpacity
          class="customButton"
          :disabled="ratingHandle == ''"
          :onPress="getRatingOfUser"
          :style="ratingHandle == '' ? {backgroundColor: 'grey'} : {}"
        >
          <Text class="whiteText">Submit</Text>
        </TouchableOpacity>
      </view>
    </view>
  </view>
</template>

<style>
.container {
  background-color: white;
  align-items: center;
  flex: 1;
}
.card {
  background-color: greenyellow;
}
.paginationButtons {
  flex-direction: row;
  margin-top: 20;
}
.toggleButton {
  margin-top: 10;
  max-width: 200;
  background-color: blue;
  border-radius: 20;
  align-items: center;
  padding: 10;
  justify-content: center;
  max-height: 35;
}
.customButton {
  background-color: blue;
  border-radius: 20;
  align-items: center;
  padding: 10;
  justify-content: center;
  max-height: 35;
  margin: 5;
}
.whiteText {
  color: white;
}
.blackText {
  color: black;
  font-weight: bold;
}
.card {
  border-style: solid;
  border-width: 2;
  padding: 4;
  margin: 5;
  border-radius: 5;
  align-items: center;
}
.ratingHandleInput {
  height: 40;
  width: 100;
  border-color: black;
  border-width: 1;
  border-style: solid;
  margin: 5;
  border-radius: 5;
  background-color: white;
}
.ratingScroll{
  padding: 4;
  margin-bottom: 50;
}
.contests {
  align-items: center;
  overflow: hidden;
  flex: 8;
}
.handleRating {
  flex-direction: row;
  justify-content: space-between;
  align-items: center;
  width: 50%;
  flex: 2;
}
</style>
<script>
import axios from "axios";
export default {
  data: {
    contests: [],
    contestPageNumber: 0,
    constestSize: 15,
    ratingHandle: "",
    ratingHistory: [],
    showRating: false
  },
  computed: {
    constestPageCount() {
      let l = this.contests.length;
      let s = this.constestSize;
      return Math.ceil(l / s);
    },
    paginatedContests() {
      const start = this.contestPageNumber * this.constestSize;
      const end = start + this.constestSize;
      return this.contests.slice(start, end);
    }
  },
  created() {
    axios
      .get("https://codeforces.com/api/contest.list")
      .then(res => {
        this.contests = res.data.result;
      })
      .catch(err => {
        console.log(err);
      });
  },
  methods: {
    nextContestPage() {
      this.contestPageNumber++;
    },
    prevContestPage() {
      this.contestPageNumber--;
    },
    getTimeFromUnixTime(unix_timestamp) {
      const months_arr = [
        "January",
        "February",
        "March",
        "April",
        "May",
        "June",
        "July",
        "August",
        "September",
        "October",
        "November",
        "December"
      ];
      // Convert timestamp to milliseconds
      const date = new Date(unix_timestamp * 1000);

      // Year
      const year = date.getFullYear();

      // Month
      const month = months_arr[date.getMonth()];

      // Day
      const day = date.getDate();

      // Hours
      const hours = date.getHours();

      // Minutes
      const minutes = "0" + date.getMinutes();

      // Display date time in MM-dd-yyyy h:m:s format
      const convdataTime =
        day + "-" + month + "-" + year + " " + hours + ":" + minutes.substr(-2);
      return convdataTime;
    },
    getRatingOfUser() {
      axios
        .get("https://codeforces.com/api/user.rating", {
          params: {
            handle: this.ratingHandle
          }
        })
        .then(res => {
          this.ratingHistory = res.data.result.reverse();
          this.showRating = true;
        })
        .catch(err => {
          console.log(err);
        });
    }
  }
};
</script>