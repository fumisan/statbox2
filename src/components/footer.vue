<template>
  <div id="footer">
    <div class="resizers">
      <div class="resizer top">
        <span style="padding: 0 20px 0 20px">出典とテーブル</span>
      </div>
      <div style="padding-top: 40px">
        <!--メタ情報とテーブル作成-->
        <div
          v-for="el in footerInner"
          :id="el.id"
          :key="el.id"
        >
          <div v-show="m_divShow (statType, el.side)">
            <div class="bottom-info-div">
              <table class="source-table">
                <tr><th>種類</th><td>{{ m_metaData (statType, el.side).syurui }}</td></tr>
                <tr><th>データ名</th><td>{{ m_metaData (statType, el.side).dataName }}</td></tr>
                <tr><th>statsDataId</th><td>{{ m_metaData (statType, el.side).statsDataId }}</td></tr>
                <tr><th>※項目符号（指標コード）</th><td>{{ m_metaData (statType, el.side).cdCat01 }}</td></tr>
                <tr><th>出典</th><td v-html="m_metaData (statType, el.side).source1" /></tr>
                <tr><th>所在源又は<br>参考資料等</th><td v-html="m_metaData (statType, el.side).source2" /></tr>
              </table>
              ※基礎データの時は「項目符号」。社会生活統計指標の時は「指標コード」
            </div>
            <div class="bottom-table-div">
              <el-table
                :data="m_tableData (statType, el.side)"
                v-show="['pref', 'scatterPref', 'city', 'scatterCity'].includes(statType)"
              >
                <el-table-column
                  prop="citycode"
                  label="citycode"
                  sortable
                />
                <el-table-column
                  prop="cityname"
                  label="cityname"
                />
                <el-table-column
                  prop="data"
                  label="data"
                  sortable
                />
              </el-table>
            </div>
          </div>
        </div>
        <!--メタ情報とテーブル作成ここまで-->
      </div>
    </div>
    <resize-observer @notify="mix_detectResize" />
  </div>
</template>

<script>
  import mixinDetectResize from '../components/mixin/detectResize'
  export default {
    name: "Bottom",
    props: {
      side: {type: String, default: ''},
      statType: {type: String, default: ''}
    },
    mixins: [mixinDetectResize],
    data () {
        return {
          footerInner: [
            {id: 'footer-inner-left', side: 'leftSide'},
            {id: 'footer-inner-right', side: 'rightSide'}
          ]
        }
    },
    methods: {
      m_divShow (statType, side) {
        if (side === 'leftSide') {
          return ['pref', 'scatterPref', 'city', 'scatterCity', 'timePref', 'timeCity'].includes(statType)
        }
          return ['scatterPref', 'scatterCity'] .includes(statType)
      },
      m_metaData (statType, side) {
        let stat, syurui, dataName, statsDataId, cdCat01, source1, source2;
        const sourceLink = '<a href="https://www.stat.go.jp/data/ssds/2.html" target="_blank">整備している項目</a>';
        switch (statType) {
          case 'pref':
          case 'scatterPref':
            stat = side === 'leftSide' ? this.$store.state.statList.leftStatEstatPref : this.$store.state.statList.rightStatEstatPref;
            break;
          case 'city':
          case 'scatterCity':
            stat = side === 'leftSide' ? this.$store.state.statList.leftStatEstatCity : this.$store.state.statList.rightStatEstatCity;
            break;
          case 'timePref':
            stat = this.$store.state.statList.leftStatTimePref;
            break;
          case 'timeCity':
            stat = this.$store.state.statList.leftStatTimeCity;
            break;
        }
        if (stat) {
          if (stat.cdCat01.substr(0, 1) !== '#') {
            syurui = '基礎データ'
          } else {
            syurui = '社会生活統計指標'
          }
          dataName = stat.statName;
          statsDataId = stat.statsDataId;
          cdCat01 = stat.cdCat01;
          source1 = '<a href="https://www.stat.go.jp/data/ssds/index.html" target="_blank">都道府県・市区町村のすがた（社会・人口統計体系）</a>';
          if (stat.cdCat01.substr(0, 1) !== '#') {
            source2 = stat.source + '<br>' + sourceLink
          } else {
            source2 = sourceLink
          }
        }
        return {syurui, dataName, statsDataId, cdCat01, source1, source2};
      },
      m_tableData (statType, side) {
        let data = [];
        let target;
        switch (statType) {
          case 'pref':
            target = side === 'leftSide' ? this.$store.state.statList.leftStatEstatPref : this.$store.state.statList.rightStatEstatPref;
            target = target.statData[this.$store.state.statList.yearRangePref];
            break;
          case 'scatterPref':
            target = side === 'leftSide' ? this.$store.state.statList.leftStatEstatPref : this.$store.state.statList.rightStatEstatPref;
            target = target.statData[this.$store.state.statList.yearRangeScatterPref];
            break;
          case 'city':
            target = side === 'leftSide' ? this.$store.state.statList.leftStatEstatCity : this.$store.state.statList.rightStatEstatCity;
            target = target.statData[this.$store.state.statList.yearRangeCity];
            break;
          case 'scatterCity':
            target = side === 'leftSide' ? this.$store.state.statList.leftStatEstatCity : this.$store.state.statList.rightStatEstatCity;
            target = target.statData[this.$store.state.statList.yearRangeScatterCity];
            break;
          case 'timePref':
            target = this.$store.state.statList.leftStatTimePref.statData;
            target = target[target.length - 1];
            break;
          case 'timeCity':

            break;
        }
        // console.log(target)
        if (target) data = target.data2;
        return data
      }
    }
  }
</script>
