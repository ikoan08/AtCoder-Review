<template>
  <div>
    <v-row>
      <v-col cols="4">
        <v-text-field placeholder="UserID" outlined dense v-model="UserID"></v-text-field>
      </v-col>
      <v-col cols="8">
        <v-btn @click="GetData()">検索</v-btn>
      </v-col>
      <v-col cols="12">
        <v-btn-toggle v-model="ProblemClass" mandatory>
          <v-btn>all</v-btn>
          <v-btn>a</v-btn>
          <v-btn>b</v-btn>
          <v-btn>c</v-btn>
          <v-btn>d</v-btn>
          <v-btn>e</v-btn>
          <v-btn>f</v-btn>
        </v-btn-toggle>
      </v-col>
      <div>
        <v-col v-if="ProblemClass===0">
          <ul>
            <li v-for="(p, index) in Problems" :key="index">
              <p>
                <a :href="p.URL" target="_blank">{{ p.ProblemName }}</a>
                : {{ p.Count }} 回既に解いた
              </p>
            </li>
          </ul>
        </v-col>
        <v-col v-else-if="ProblemClass===1">
          <ul>
            <li v-for="(p, index) in Problems_a" :key="index">
              <p>
                <a :href="p.URL" target="_blank">{{ p.ProblemName }}</a>
                : {{ p.Count }} 回既に解いた
              </p>
            </li>
          </ul>
        </v-col>
        <v-col v-else-if="ProblemClass===2">
          <ul>
            <li v-for="(p, index) in Problems_b" :key="index">
              <p>
                <a :href="p.URL" target="_blank">{{ p.ProblemName }}</a>
                : {{ p.Count }} 回既に解いた
              </p>
            </li>
          </ul>
        </v-col>
        <v-col v-else-if="ProblemClass===3">
          <ul>
            <li v-for="(p, index) in Problems_c" :key="index">
              <p>
                <a :href="p.URL" target="_blank">{{ p.ProblemName }}</a>
                : {{ p.Count }} 回既に解いた
              </p>
            </li>
          </ul>
        </v-col>
        <v-col v-else-if="ProblemClass===4">
          <ul>
            <li v-for="(p, index) in Problems_d" :key="index">
              <p>
                <a :href="p.URL" target="_blank">{{ p.ProblemName }}</a>
                : {{ p.Count }} 回既に解いた
              </p>
            </li>
          </ul>
        </v-col>
        <v-col v-else-if="ProblemClass===5">
          <ul>
            <li v-for="(p, index) in Problems_e" :key="index">
              <p>
                <a :href="p.URL" target="_blank">{{ p.ProblemName }}</a>
                : {{ p.Count }} 回既に解いた
              </p>
            </li>
          </ul>
        </v-col>
        <v-col v-else-if="ProblemClass===6">
          <ul>
            <li v-for="(p, index) in Problems_f" :key="index">
              <p>
                <a :href="p.URL" target="_blank">{{ p.ProblemName }}</a>
                : {{ p.Count }} 回既に解いた
              </p>
            </li>
          </ul>
        </v-col>
      </div>
    </v-row>
    <v-overlay :value="Loading"></v-overlay>
  </div>
</template>

<script lang="ts">
import Vue from 'vue'

//問題を取得する基本の型
export interface Problem {
  ProblemName: string
  Count: number
  URL: string
  Time: number
}

export default Vue.extend({
  name: 'index',
  data() {
    return {
      UserID: '' as string,
      ProblemClass: 0 as number,
      Loading: false,
      Problems: [] as Problem[],
      Problems_a: [] as Problem[],
      Problems_b: [] as Problem[],
      Problems_c: [] as Problem[],
      Problems_d: [] as Problem[],
      Problems_e: [] as Problem[],
      Problems_f: [] as Problem[],
    }
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
          if (response.length === 0) {
            throw new Error('提出した問題が無いヨ！！')
          }
          this.Loading = true
          let map = new Map<string, number>() //問題ごとにカウントを行う連想配列
          let latest_submit = new Map<string, number>() //最後に提出した時間を記録する連想配列

          //格納配列のリセット
          this.Problems.length = 0
          this.Problems_a.length = 0
          this.Problems_b.length = 0
          this.Problems_c.length = 0
          this.Problems_d.length = 0
          this.Problems_e.length = 0
          this.Problems_f.length = 0

          // ACした問題だけ取り出す
          const AC_Results = response.filter((res: any) => {
            return res.result === 'AC'
          })
          if (AC_Results.length === 0) {
            this.Loading = false
            throw new Error('ACした問題が無いヨ！！')
          }
          // 連想配列でカウント
          AC_Results.forEach((res: any) => {
            const key: string = res.contest_id + '/' + res.problem_id
            if (map.has(key)) {
              let x = map.get(key) as number
              map.set(key, x + 1)
              latest_submit.set(
                key,
                Math.max(latest_submit.get(key) as number, res.epoch_second)
              )
            } else {
              map.set(key, 1)
              latest_submit.set(key, res.epoch_second)
            }
          })

          //配列に格納
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
              Time: latest_submit.get(key) as number,
            })
            const klass = ContestInfo[1].split('_').pop()

            //以下は問題ごとに分類して格納、処理は上とほぼ同じ
            if (klass === 'a' || klass === '1') {
              this.Problems_a.push({
                ProblemName: ContestInfo[1],
                Count: value,
                URL:
                  'https://atcoder.jp/contests/' +
                  ContestInfo[0] +
                  '/tasks/' +
                  ContestInfo[1],
                Time: latest_submit.get(key) as number,
              })
            } else if (klass === 'b' || klass === '2') {
              this.Problems_b.push({
                ProblemName: ContestInfo[1],
                Count: value,
                URL:
                  'https://atcoder.jp/contests/' +
                  ContestInfo[0] +
                  '/tasks/' +
                  ContestInfo[1],
                Time: latest_submit.get(key) as number,
              })
            } else if (klass === 'c' || klass === '3') {
              this.Problems_c.push({
                ProblemName: ContestInfo[1],
                Count: value,
                URL:
                  'https://atcoder.jp/contests/' +
                  ContestInfo[0] +
                  '/tasks/' +
                  ContestInfo[1],
                Time: latest_submit.get(key) as number,
              })
            } else if (klass === 'd' || klass === '4') {
              this.Problems_d.push({
                ProblemName: ContestInfo[1],
                Count: value,
                URL:
                  'https://atcoder.jp/contests/' +
                  ContestInfo[0] +
                  '/tasks/' +
                  ContestInfo[1],
                Time: latest_submit.get(key) as number,
              })
            } else if (klass === 'e' || klass === '5') {
              this.Problems_e.push({
                ProblemName: ContestInfo[1],
                Count: value,
                URL:
                  'https://atcoder.jp/contests/' +
                  ContestInfo[0] +
                  '/tasks/' +
                  ContestInfo[1],
                Time: latest_submit.get(key) as number,
              })
            } else if (klass === 'f' || klass === '6') {
              this.Problems_f.push({
                ProblemName: ContestInfo[1],
                Count: value,
                URL:
                  'https://atcoder.jp/contests/' +
                  ContestInfo[0] +
                  '/tasks/' +
                  ContestInfo[1],
                Time: latest_submit.get(key) as number,
              })
            }
          }

          //ソート
          this.Problems_a.sort((x: Problem, y: Problem): number => {
            if (x.Count !== y.Count) return x.Count - y.Count
            else return x.Time - y.Time
          })
          this.Problems_b.sort((x: Problem, y: Problem): number => {
            if (x.Count !== y.Count) return x.Count - y.Count
            else return x.Time - y.Time
          })
          this.Problems_c.sort((x: Problem, y: Problem): number => {
            if (x.Count !== y.Count) return x.Count - y.Count
            else return x.Time - y.Time
          })
          this.Problems_d.sort((x: Problem, y: Problem): number => {
            if (x.Count !== y.Count) return x.Count - y.Count
            else return x.Time - y.Time
          })
          this.Problems_e.sort((x: Problem, y: Problem): number => {
            if (x.Count !== y.Count) return x.Count - y.Count
            else return x.Time - y.Time
          })
          this.Problems_f.sort((x: Problem, y: Problem): number => {
            if (x.Count !== y.Count) return x.Count - y.Count
            else return x.Time - y.Time
          })
          this.Loading = false
        })
        .catch((error) => {
          alert(error)
        })
    },
  },
})
</script>
