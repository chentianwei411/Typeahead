<template>
  <div>
    <!-- burl event: 当失去焦点时触发事件！ -->
    <input v-model="query" @burl="reset" type="text" class="SearchInput" :placeholder="placeholder">
    <!-- 使用过渡 -->
    <transition-group name='fade' tag="ul" class="Results">
      <li v-for="item in filtered" :key="item.id">
        <span>
          <strong>{{ item.title }}</strong> - <small>{{ item.id }}</small><br>
          <small>{{ item.body }}</small>
        </span>
      </li>
    </transition-group>
    <p v-show="isEmpty">Sorry, but we can't find any match for given term :( </p>
  </div>
</template>

<script>
  export default {
    name: 'Typeahead',

    props: {
      //用户可传API Endpoint(URL)这是string格式，或者他从父组件传已经建立的数组对象。
      // 这个source prop是required,防止空传入，Vuejs会抛出错误到控制台。
      source: {
        type: [String, Array],
        required: true
      },
      // 通过指定的property来filter对象。
      // 比如一个数组对象，每个对象都有tilte property，当用户在inputbox输入什么，它会检索符合的title
      filterKey: {
        type: String,
        required: true
      },
      // 决定当用户输入3个字符后开始filter。
      startAt: {
        type: Number,
        default: 3
      },
      placeholder: {
        type: String,
        default: ""
      }
    },

    data() {
      return {
        // items储存通过source prop传下来的数据
        items: [],
        // 绑定到v-model, 实时更新用户的input.
        query: ''
      }
    },

    methods: {
      fetchItems() {
        var that = this
        if (typeof this.source === 'string') {
          fetch(this.source)
          .then(stream => stream.json())
          .then(function(data) {
            that.items = data
            // console.log(that.items)
            return that.items
          })
          // .catch(error => console.error(error))
        } else {
          this.items = this.source
          console.log("3", this.items)
        }
      },

      reset() {
        this.query = ""
      }
    },

    // 当挂载这个组件实例时:
    mounted() {
      this.fetchItems()
    },

    computed: {
      filtered() {
        // 当用户输入的字符数量达到startAt的要求时
        if (this.query.length >= this.startAt) {
          // 使用Array.prototype.filter()：逐个检索item,返回符合条件的一个数组。
          // 类似Ruby的Array#select

          return this.items.filter(item => {
            // 保证item有filterKey的特性：
            if (item.hasOwnProperty(this.filterKey)) {
              //这个item的filterKey特性的值（一个string）包含this.query则返回值大于-1。
              // string.indexof("xxx")返回“xxx”所在的索引位置，可以可以用javascript的正则表达式方法。
              // 方法另看，如何给正则表达式插入一个变量？以及javaScript的正则表达式的方法使用。
              // https://www.cnblogs.com/season-huang/p/3544873.html
              return item[this.filterKey].toLowerCase().indexOf(this.query.toLowerCase()) > -1
            } else {
              console.error(`Seems like property you passed down ${this.filterKey}`)
            }
          })
        }
      },

      isEmpty() {
        if (typeof this.filtered === 'undefined') {
          return false
        } else {
          return this.filtered.length < 1
        }
      }
    }

  }

</script>

<style>
  .SearchInput {
    width: 100%;
    padding: 1.5em 1em;
    font-size: 1em;
    outline: 0;
    border: 5px solid #41B883;
  }
  .Results {
    margin: 0;
    padding: 0;
    text-align: left;
    position: relative;
  }
  .Results li {
    background: rgba(53, 73, 94, 0.3);
    margin: 0;
    padding: 1em;
    list-style: none;
    width: 100%;
    border-bottom: 1px solid #394E62;
    transition: ease-in-out 0.5s;
  }
  .fade-enter-active, .fade-leave-active {
    transition: opacity 0.3s;
  }
  .fade-enter, .fade-leave-to {
    opacity: 0;
  }
</style>
