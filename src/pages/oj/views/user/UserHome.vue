<template>
  <div class="container">
    <Card class="profile-card">
      <!-- 프로필 헤더 -->
      <div class="profile-header">
        <div class="avatar-container">
          <img class="avatar" :src="profile.avatar"/>
        </div>
      </div>

      <!-- 프로필 본문 -->
      <div class="profile-content">
        <p class="user-info">
          <span class="emphasis">{{ profile.user.username }}</span>
          <span v-if="profile.school">@{{ profile.school }}</span>
        </p>
        <p v-if="profile.mood">{{ profile.mood }}</p>
        <hr class="split"/>

        <div class="flex-container">
          <div class="left">
            <p>{{$t('m.UserHomeSolved')}}</p>
            <p class="emphasis">{{profile.accepted_number}}</p>
          </div>
          <div class="middle">
            <p>{{$t('m.UserHomeserSubmissions')}}</p>
            <p class="emphasis">{{profile.submission_number}}</p>
          </div>
          <div class="right">
            <p>{{$t('m.UserHomeScore')}}</p>
            <p class="emphasis">{{profile.total_score}}</p>
          </div>
        </div>

        <div id="problems">
          <p v-if="problems.length">{{$t('m.List_Solved_Problems')}}</p>
          <p v-else>{{$t('m.UserHomeIntro')}}</p>
          <div class="btns">
            <Button v-for="problemID in problems" :key="problemID" type="ghost" @click="goProblem(problemID)">
              {{ problemID }}
            </Button>
          </div>
        </div>

        <div id="icons">
          <a :href="profile.github"><Icon type="social-github-outline" size="30"/></a>
          <a :href="'mailto:'+ profile.user.email"><Icon class="icon" type="ios-email-outline" size="30"/></a>
          <a :href="profile.blog"><Icon class="icon" type="ios-world-outline" size="30"/></a>
        </div>
      </div>
    </Card>
  </div>
</template>
<script>
  import { mapActions } from 'vuex'
  import time from '@/utils/time'
  import api from '@oj/api'

  export default {
    data () {
      return {
        username: '',
        profile: {},
        problems: []
      }
    },
    mounted () {
      this.init()
    },
    methods: {
      ...mapActions(['changeDomTitle']),
      init () {
        this.username = this.$route.query.username
        api.getUserInfo(this.username).then(res => {
          this.changeDomTitle({title: res.data.data.user.username})
          this.profile = res.data.data
          this.getSolvedProblems()
          let registerTime = time.utcToLocal(this.profile.user.create_time, 'YYYY-MM-D')
          console.log('The guy registered at ' + registerTime + '.')
        })
      },
      getSolvedProblems () {
        let ACMProblems = this.profile.acm_problems_status.problems || {}
        let OIProblems = this.profile.oi_problems_status.problems || {}
        // todo oi problems
        let ACProblems = []
        for (let problems of [ACMProblems, OIProblems]) {
          Object.keys(problems).forEach(problemID => {
            if (problems[problemID]['status'] === 0) {
              ACProblems.push(problems[problemID]['_id'])
            }
          })
        }
        ACProblems.sort()
        this.problems = ACProblems
      },
      goProblem (problemID) {
        this.$router.push({name: 'problem-details', params: {problemID: problemID}})
      },
      freshProblemDisplayID () {
        api.freshDisplayID().then(res => {
          this.$success('Update successfully')
          this.init()
        })
      }
    },
    computed: {
      refreshVisible () {
        if (!this.username) return true
        if (this.username && this.username === this.$store.getters.user.username) return true
        return false
      }
    },
    watch: {
      '$route' (newVal, oldVal) {
        if (newVal !== oldVal) {
          this.init()
        }
      }
    }
  }
</script>

<style lang="less" scoped>
.container {
  display: flex;
  justify-content: center;
  align-items: center;
  width: 100%;
  margin-top: 50px;
}

/* 🌟 Card 스타일 개선 */
.profile-card {
  position: relative;
  width: 60%;
  padding: 80px 40px 50px; /* 프로필 사진이 자연스럽게 위치하도록 조정 */
  border-radius: 10px;
  background: #fff;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
  text-align: center;
}

/* 📌 프로필 사진이 `Card` 내부에서 상단에 걸쳐 배치되도록 설정 */
.profile-header {
  position: relative;
  display: flex;
  justify-content: center;
  align-items: center;
}

/* 📌 프로필 사진이 카드 안쪽에 적절히 배치되도록 조정 */
.avatar-container {
  position: absolute;
  top: -85px; /* 기존 -70px에서 -85px로 올려서 유저명과 간격 확보 */
  width: 120px;
  height: 120px;
  background: white;
  border-radius: 50%;
  padding: 5px;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.2);
}

/* 프로필 사진 크기 설정 */
.avatar {
  width: 100%;
  height: 100%;
  border-radius: 50%;
}

/* 📌 프로필 본문과의 간격 조정 */
.profile-content {
  margin-top: 50px;
}

/* 📌 유저명이 프로필 사진과 겹치지 않도록 추가 조정 */
.user-info {
  margin-top: 20px; /* 기존보다 더 넉넉한 여백 추가 */
}

/* 📌 구분선 스타일 */
.split {
  margin: 20px auto;
  width: 90%;
}

/* 📌 통계 정보 정렬 */
.flex-container {
  display: flex;
  justify-content: space-around;
  align-items: center;
  text-align: center;
  padding: 20px;
}

#problems {
  margin-top: 40px;
  font-size: 18px;
}

.btns {
  margin-top: 15px;
}

/* 📌 아이콘 배치 */
#icons {
  margin-top: 20px;
  display: flex;
  justify-content: center;
  gap: 15px;
}

.icon {
  padding-left: 10px;
}
</style>
