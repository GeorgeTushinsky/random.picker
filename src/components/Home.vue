<template>
  <div>
    <div class="inputField">
      <input type="text" placeholder="Enter item name"
             @keydown.enter="addItem(currentItem)"
             v-model="currentItem">
          <b-button variant="success" size="sm" @click="addItem(currentItem)">Add to list</b-button>
          <br>
      <p>You can choose text file with options</p><input class="file_input" type='file' @change="addFromFile($event)">
    </div>

    <b-container>
      <b-row>
        <b-col></b-col>
        <b-col cols="3" class="itemsList">
          <h3>Items list</h3>
          <ul>
            <li v-for="(item, index ) in itemList" :key="index" @click="deleteItem(item)">{{item}}</li>
          </ul>
          <b-button variant="danger" size="sm" @click="clearItemsList()">Clear list</b-button>
        </b-col>
        <b-col cols="2"></b-col>
        <b-col cals="3" class="randomField">
          <input type="text" placeholder="Enter count of random items" style="width: 50px"
                        v-model="rItemsCount"
                        @keydown.enter="selectRandomItem()">
          <b-button variant="primary" size="sm" @click="selectRandomItem()">Select random item</b-button>
          <ul style="margin-top: 5px;">
            <li v-for="(item, index) in randomItems" :key="index">{{item}}</li>
          </ul>
        </b-col>
        <b-col></b-col>
      </b-row>
    </b-container>
  </div>
</template>

<script>
import axios from "axios"

export default {
  name: 'Home',
  data () {
    return {
      itemList: [],
      currentItem: '',
      randomItems: [],
      rItemsCount: 1
    }
  },
  created () {
    this.itemList = JSON.parse(localStorage.getItem('rand_listOfItems')) || []
  },
  methods: {
    addItem (item) {
      if(item === '' || item === ' ')
        return
      this.itemList.push(item)
      this.currentItem = ''
      localStorage.setItem('rand_listOfItems', JSON.stringify(this.itemList))
    },
    addFromFile(event) {
      let reader = new FileReader()
      reader.onload = (e) => {
        let arr = e.target.result.split("\n")
        arr.forEach(e => this.addItem(e))
      }
      reader.readAsText(event.target.files[0])
    },
    clearItemsList() {
      this.itemList = []
      localStorage.setItem('rand_listOfItems', JSON.stringify(this.itemList))
    },
    deleteItem (item) {
      this.itemList.splice(this.itemList.indexOf(item), 1)
      localStorage.setItem('rand_listOfItems', JSON.stringify(this.itemList))
    },
    selectRandomItem () {
      let params = {
        min: 0,
        max: this.itemList.length,
        num: this.rItemsCount
      }
      this.randomItems = []
      if(this.itemList.length === 1)
        this.randomItems.push(this.itemList[0])
      else if(this.itemList.length === 0)
        return
      else {
        this.randomItems = []
        this.getRandomInt('post', params).then(res => {
          for (let i = 0; i < res.length; i++) {
            this.randomItems.push(this.itemList[res[i] - 1])
          }
        })
      }
    },
    getRandomInt (m, paramsObj) {
      // let params = {
      //   num: paramsObj.num || 1,
      //   min: paramsObj.min || 1,
      //   max: paramsObj.max || 10,
      //   base: paramsObj.base || 10,
      //   col: paramsObj.col || 1,
      //   format: paramsObj["format"] || "plain",
      //   rnd: paramsObj.rnd || "new"
      // }
      // let url = "https://www.random.org/sequences/?num=" + params.num +
      //   "&min=" + params.min + "&max=" + params.max + "&base=" + params.base +
      //   "&col=" + params.col + "&format=" + params.format + "&rnd=" + params.rnd
      let url = 'https://api.random.org/json-rpc/2/invoke'
      let method = m.toLowerCase()

      return new Promise((resolve, reject) => {
        axios[method](url, {
          jsonrpc: '2.0',
          method: 'generateIntegers',
          params: {
            apiKey: "2a2d5c0b-ff8e-4b8c-b4b7-27f1e5a12132",
            n: paramsObj.num > this.itemList.length ? this.itemList.length : paramsObj.num,
            min: paramsObj.min || 1,
            max: paramsObj.max || 10,
            replacement: false
          },
          id: 228322
        }, {
          headers: {
            'Content-type': 'application/json'
          }
        }).then(res => {
          let randoms = res.data.result.random.data
          return resolve(randoms)
        }).catch(e => {
          console.log(e)
        })
      })
    }
  }
}
</script>

<style scoped>
  .inputField {
    text-align: center;
  }
  .file_input {
    margin-left: 7.5%;
  }

  li {
    border-radius: 3px;
    list-style: none;
    -webkit-user-select: none;
    margin-left: -40px;
  }
  .itemsList > ul > li:hover {
    background-color: lightskyblue;
    cursor: pointer;
  }
  .itemsList {
    text-align: center;
    max-height: 300px;
    max-width: 300px;
    margin-top: 100px;
  }
  .itemsList > ul {
    overflow: auto;
    max-height: 300px;
    min-height: 100px;
  }
  .randomField {
    vertical-align: top;
    margin-top: 100px;
  }
  .randomField > ul {
    overflow: auto;
    min-height: 100px;
    max-height: 300px;
    min-width: 200px;
    max-width: 300px;
  }
</style>
