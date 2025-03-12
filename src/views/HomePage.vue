<template>
  <div class="contentWrapper">
    <div class="wrapper">
      <div class="form">
        <div id="shoes">
          <div class="name">品类编号: </div>
          <input type="text" placeholder="品类编号" v-model="type">
        </div>
        <div id="shoes">
          <div class="name">尺码: </div>
          <input type="text" placeholder="尺码" v-model="size">
        </div>
        <div id="shoes">
          <div class="name">硬度: </div>
          <input type="text" placeholder="硬度" v-model="toughness">
        </div>
        <div id="shoes">
          <div class="name">颜色编号: </div>
          <input type="text" placeholder="颜色编号" v-model="color">
        </div>
        <div id="shoes">
          <div class="name">生产年月: </div>
          <input type="text" v-model="yearMonth">
        </div>
        <div id="shoes">
          <div class="name">儿童/成人: </div>
          <select v-model="ageGroup">
            <option value="儿童">儿童</option>
            <option value="成人">成人</option>
          </select>
        </div>
        <div id="shoes">
          <div class="name">非成品: </div>
          <input type="checkbox" v-model="nonComplete">
        </div>
        <div id="shoes">
          <div class="name">厂家编号: </div>
          <input type="text" placeholder="厂家编号" v-model:="producer">
        </div>
        <div id="buttonWrapper">
          <button id="submit" @click="submitForm">提交</button>
          <button id="generateExcel" @click="generateExcel">生成Excel表格</button>
        </div>
      </div>
      <div class="displayInfos">
        <div class="column" v-for="(colTitle, cIdx) in data[0]" :key="cIdx">
          <div class="cell" v-for="(row, rIdx) in data" :key="rIdx">{{ row[cIdx] }}</div>
        </div>
      </div>
    </div>
  </div>
</template>
<style>
  .contentWrapper{
    display: flex;
    justify-content: center;
  }
  .wrapper{
    width: 50%;
    background-color: wheat;
  }
  .wrapper>.form{
    padding: 1rem;
    box-sizing: border-box;
    border-radius: 1rem;
    border: 1px solid;
    background: antiquewhite;
    margin: 1rem;
  }
  .wrapper>.form>div{
    display: flex;
    justify-content: space-between;
    width: 70%;
    margin: 1rem 0 1rem 0;
  }
  .displayInfos{
    margin: 1rem 0 1rem 0;
    width: 100%;
    display: flex;
    justify-content: space-around;
    padding: 1rem;
    box-sizing: border-box;
  }
  .displayInfos>.column{
    font-size: 0.7rem;
    border-right: 1px solid black;
    padding: 0.5rem;
    box-sizing: border-box;
  }
  .displayInfos>.column>.cell{
    height: 1rem;
    align-items: center;
  }
  #buttonWrapper{
    width: 100%!important;
    display: flex;
    justify-content: space-evenly;
  }
  #buttonWrapper>button{
    width: 30%;
    background-color: lightblue;
  }
</style>
<script>
export default {
  name: 'HomePage',
  data () {
    return {
      type: 'S1 S2',
      size: '210 220',
      toughness: '100',
      color: 'H001 W001 ZH01',
      yearMonth: '0125',
      ageGroup: '成人',
      nonComplete: false,
      producer: '01',
      XLSX: null,
      wb: null,
      ws: null,
      data: [
        ['编码', '厂家编码', '生产年月', '品类编码', '非成品', '成人/儿童', '硬度', '颜色', '尺寸']
      ]
    }
  },
  mounted () {
    this.initExcel()
  },
  methods: {
    submitForm () {
      // console.log(this.type, this.size, this.toughness, this.color, this.yearMonth, this.ageGroup, this.nonComplete, this.producer)
      this.generateForm()
    },
    generateForm () {
      const res = []
      const arr = [this.producer, this.yearMonth, this.type, this.nonComplete, this.ageGroup, this.toughness, this.color, this.size]
      function dfs (i, str) {
        if (i >= arr.length) {
          res.push(str.substring(1, str.length))
          return
        }
        let cur = (arr[i] + '').split(' ')
        cur = cur.filter(ele => ele !== '')
        for (let j = 0; j < cur.length; j++) {
          dfs(i + 1, str + ' ' + cur[j])
        }
      }
      dfs(0, '')

      // populate the 2D array -data
      for (const str of res) {
        const code = this.encodeStr(str)
        this.data.push([code].concat(str.split(' ')))
      }
    },
    encodeStr (str) {
      const infos = str.split(' ')
      /*
      infos[0] --> producer
      infos[1] --> yearMonth 2025-01 --> 0125
      infos[2] --> type
      infos[3] --> nonComplete
      infos[4] --> ageGroup
      infos[5] --> toughness
      infos[6] --> color
      infos[7] --> size
      */
      const code = (
        infos[0] + infos[1] + infos[2] + ' - ' +
        (infos[3] === 'false' ? 'M' : 'F') + (infos[4] === '成人' ? 'A' : 'K') + infos[5] + ' - ' +
        infos[6] + infos[7]
      )
      // console.log(code)
      return code
      /* discarded code
      const map = new Map()
      map.set('默认', '01')
      map.set('雪鞋', 'S2')
      map.set('黑', 'H')
      map.set('白', 'W')
      map.set('棕', 'Z')
      map.set('成人', 'A')
      map.set('儿童', 'K')

      let code = ''
      code += map.get(infos[0])
      code += infos[1]
      code += map.get(infos[2])
      */
    },
    initExcel () {
      this.XLSX = require('xlsx')
    },
    generateExcel () {
      this.wb = this.XLSX.utils.book_new()
      this.ws = this.XLSX.utils.aoa_to_sheet(this.data)
      this.XLSX.utils.book_append_sheet(this.wb, this.ws, 'sheet1')
      this.XLSX.writeFile(this.wb, 'result.xlsx') // this line will create an excel file and put it in the download folder of the computer. with data wrote in it
      // console.log('excel file created!')
    }
  }
}
</script>
