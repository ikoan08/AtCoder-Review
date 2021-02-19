<template>
  <v-row>
    <v-col cols="4">
      <v-text-field placeholder="UserID" outlined dense v-model="UserID"></v-text-field>
    </v-col>
    <v-col cols="8">
      <v-btn @click="GetData()">検索</v-btn>
    </v-col>
    <ul>
      <li v-for="(p, index) in Problems" :key="index">
        <p>
          <a :href="p.URL" target="_blank">{{ p.ProblemName }}</a>
          : {{ p.Count }}
        </p>
      </li>
    </ul>
  </v-row>
</template>

<script lang="ts">
import Vue from 'vue'

export default Vue.extend({
  name: 'index',
  data() {
    return {
      UserID: '' as string,
      Problems: [
        {
          ProblemName: '',
          Count: 0,
          URL: '',
        },
      ],
    }
  },
  created() {
    this.Problems.length = 0
  },
  methods: {
    // AtCoder Problems API を axiosで叩きユーザー名から提出情報を入手して加工する
    GetData() {
      //id, epoch_second, problem_id, contest_id, user_id, language, point, length, result, execution_time
      const url = 'https://kenkoooo.com/atcoder/atcoder-api/results'
      this.$axios
        .$get(url, {
          params: {
            user: this.UserID,
          },
        })
        .then((response) => {
          let map = new Map<string, number>() //問題ごとにカウントを行う連想配列
          this.Problems.length = 0
          // ACした問題だけ取り出す
          const AC_Results = response.filter((res: any) => {
            return res.result === 'AC'
          })

          // 連想配列でカウント
          AC_Results.forEach((res: any) => {
            const key: string = res.contest_id + '/' + res.problem_id
            if (map.has(key)) {
              let x = map.get(key) as number
              map.set(key, x + 1)
            } else {
              map.set(key, 1)
            }
          })

          for (let [key, value] of map) {
            const ContestInfo = key.split('/')
            this.Problems.push({
              ProblemName: ContestInfo[1],
              Count: value,
              URL:
                'https://atcoder.jp/contests/' +
                ContestInfo[0] +
                '/tasks/' +
                ContestInfo[1],
            })
          }
        })
        .catch((error) => {
          alert(error)
        })
    },
  },
})
</script>
