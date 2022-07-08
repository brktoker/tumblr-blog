<template>
  <div>
    <h1 class="title">Demo</h1>
    <div class="total-post"><span style="margin-bottom: 0.5em;"> Total Post: {{postTotal}} </span></div>
    <a class="RSS" href="https://demo.tumblr.com/rss"><img src="https://static.tumblr.com/vr9xgox/oiLnf8zpv/rss.gif" id="rss" alt="RSS"></a>
    <div id="content">
      <input class="form-control mr-sm-2 mb-5" type="search" placeholder="Search in blog date" v-model="searchQuery" aria-label="Search">
      <div class="post" v-for="(post,index) in filteredList" :key="index">
        <div class="date">
          <div class="date_brick">
            {{post.date.split(" ")[2]}} <br> {{post.date.split(" ")[1] + 'TH'}}
          </div>
          {{post.date.split(",")[0]}}
        </div>
        <div v-for="(orderedPost, index) in post.posts" :key="index" class="post">
          <div v-if="orderedPost.type === 'quote'" class="quote">
            <div class="quote_text">
              <span class="short">
                {{orderedPost['quote-text']}}
              </span>
            </div>
            <div class="source" v-html="orderedPost['quote-source']">
            </div>
          </div>

          <div v-if="orderedPost.type === 'photo'" class="photo">
            <a :href="orderedPost.url">
              <img :src="orderedPost['photo-url-400']"
                :alt="orderedPost.slug"
                target="_blank">
            </a>
            <div class="caption" v-html="orderedPost['photo-caption']">
              
            </div>
          </div>
        
          <div v-if="orderedPost.type === 'link'" class="link">
            <a :href="orderedPost.url" class="link" target="_blank">{{orderedPost['link-text']}}</a>
            <span class="description" v-html="orderedPost['link-description']">
            </span>
          </div>

          <div v-if="orderedPost.type === 'conversation'" class="conversation">
            <ul>           
              <li v-for="c in orderedPost.conversation">
                <span class="label" :class="[c.name === 'Jack' ? 'user_1' : 'user_2']">{{c.label}}</span>
                {{c.phrase}}
              </li>
            </ul>
          </div>

          <div v-if="orderedPost.type === 'audio'" class="audio">
            <span :id="'audio_player_' + orderedPost.id">
              <div class="audio_player" v-html="orderedPost['audio-player']" style="height: 100px !important;">

              </div>
              <div class="caption" v-html="orderedPost['audio-caption']">

              </div>
            </span>
          </div>

          <div v-if="orderedPost.type === 'regular'" class="regular">
            <h2>
              <a :href="orderedPost.url" target="_blank"  variant="outline-success">{{orderedPost['regular-title'] !== null ? orderedPost['regular-title'] : ''}} </a>
              <a v-b-modal="showMdal" v-b-tooltip :title="'show post modal'">
                <img style="margin-left: 0.5em;cursor: pointer;" height="16" v-b-modal="showMdal" src="data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAACAAAAAgCAYAAABzenr0AAAABmJLR0QA/wD/AP+gvaeTAAAB10lEQVRYhe3XP2sbMRzG8efnO2S8pYHMoXWHhKJFZy8lg6HkD8SFFqos3QuFDn0xWZoGSgjJ4jl7h1IP9pkDTxmsF5BA6nDDgUFSh/ogBDeSz3BerFUnfT8nENxRFEU3ADawnHEbAtgIguBFlmXjMsu1Wm1Na61CAMiybDwcDv+UCeCcgzGGSpnRWWMFWAGWDgiLLpRSBqPR6AQA6vX6p06no4vsU+gEpJSBUuoMAAfAlVJnUsqgFMD0zX8A2K5WqwdEtAvgpVLqstVqzX2icwHyOBG9YoztdrvduziO7wHsA9hM0/RiXoQ3YFY8n1sE4QX4X1wI0RZCtBdBOAFPxI+I6JyIzoUQR0URToBS6hTAltb6TR5vNBofiOiEiN5XKpVDIvomhPiYI7TWB9ba52mafl8YAODaGLOXJMk4j1trT4noXb/f/9nr9X5PEcc5IkmSsTFmD8C1a3OKoshOJpN1n++Bx/GHc81m87Ux5spa+2UwGFy49uKcP2OM3XnfgqfiADDrJHyGF8AVXwThBPjGiyKcAGvtVwBtn/hDBIC3RPTZ9azznsZxvOMbfrTuFwDn2qV/D6wAK8DSASHw70eRc15qePpzihDArdZaMcZKBWitAeDmL9RZCU0ERk2yAAAAAElFTkSuQmCC">
              </a>
            </h2>
            <span v-html="orderedPost['regular-body']"></span>
            <div>
              <b-modal id="show-modal" size="xl" :title="orderedPost['regular-title'] !== null ? orderedPost['regular-title'] : ''">
                <p class="my-4" v-html="orderedPost['regular-body']" ></p>
              </b-modal>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>

export default {
  data() {
    return {
      jsonData : {},
      posts: {},
      postTotal: 0,
      searchQuery: '',
      showMdal: 'show-modal'
    }
  },
  mounted () {
    // this.getJsonBlogData()
    this.orderDateType
    var linkList = document.querySelectorAll('a');
    if (linkList.length > 0) {
      for(var i in linkList){
        linkList[i].setAttribute('target', '_blank');
      }
    }
  },
  methods: {
    matches(obj) {
      const term = this.searchQuery.toLowerCase();
      return obj.date.toLowerCase().includes(term) || obj.date.toLowerCase().includes(term);
    }
  },
  computed: {
    orderDateType () {
      let jsonData = require('../assets/data.json')
      this.postTotal = jsonData['posts-total']
      const groups = jsonData.posts.reduce((groups, post) => {
        let date = post.date.split('2006')[0]
        if (!groups[date]) {
          groups[date] = [];
        }
        groups[date].push(post);
        return groups;
      }, {})

      const groupDatesData = Object.keys(groups).map((date) => {
        return {
          date,
          posts: groups[date]
        }
      })
      return groupDatesData
    },
    filteredList () {
      if (!this.searchQuery || this.searchQuery === '' ) {
        return this.orderDateType
      }
      return this.orderDateType
      .map((v) => {
        if (this.matches(v)) {
          return v;
        }
        const matchingSubcats = Object.values(v.posts)
          .filter((v) => this.matches(v));
          
        if (matchingSubcats.length > 0) {
          return Object.assign({}, v, {subcats: matchingSubcats});
        }
      })
      .filter((v) => v);
    }
  }
};
</script>
<style>

body {
  font-family: Helvetica, Arial, sans-serif;
}

p {
  font-family: Helvetica, Arial, sans-serif;
}

h1.title {
  width: 600px;
  padding: 0px 100px 20px 100px;
  margin: 50px auto 4px auto;
  text-align: center;
  font: Bold 55px 'Trebuchet MS', 'Lucida Sans Unicode', 'Lucida Grande', 'Lucida Sans', Arial, sans-serif;
  letter-spacing: -2px;
  line-height: 50px;
  position: relative;
}
#content {
  width: 420px;
  margin: auto;
  position: relative;
}
.post {
    position: relative;
    margin-bottom: 40px;
    padding-right: 20px;
}
.post .date {
    position: absolute;
    left: -260px;
    text-align: right;
    width: 230px;
    white-space: nowrap;
    font: normal 34px Helvetica, sans-serif;
    letter-spacing: -2px;
    color: #ccc;
    text-transform: uppercase;
    line-height: 35px;
}
.post .date .date_brick {
  float: right;
  height: 30px;
  width: 45px;
  background-color: #6498cc;
  color: #bbd5f1;
  font: Bold 12px Verdana, Sans-Serif;
  text-align: center;
  line-height: 12px;
  margin-left: 10px;
  padding-top: 5px;
  letter-spacing: 0px;
  overflow: hidden;
}
.post .quote .quote_text {
  font-family: Helvetica, Arial, sans-serif;
  font-weight: bold;
  color: #888;
  border-left: solid 5px #6498cc;
  padding-left: 10px;

}
.post .quote .quote_text .short {
  font-size: 36px;
  line-height: 40px;
  letter-spacing: -1px;
}
.source {
  font-size: 16px;
  font-weight: Bold;
  color: #555;
  margin-top: 5px;
}
.source a {
  color: #555;
}

.link a {
  font: Bold 20px Helvetica, Arial, sans-serif;
  letter-spacing: -1px;
  color: #c00;
}
.link .description {
  font-size: 13px;
  font-weight: normal;
  letter-spacing: -1px;
  color: #444;
}

.photo .caption {
  font-size: 14px;
  font-weight: bold;
  color: #444;
  margin-top: 10px;
  padding: 0px 20px 0px 20px;
}
.photo .caption a {
  color: #444;
}

.conversation ul {
  list-style-type: none;
  margin: 0px;
  padding: 0px 0px 0px 1px;
  border-left: solid 5px #bbb;
}
.conversation ul li:nth-child(odd) {
  background-color: #f4f4f4;
  font-family: Helvetica, Arial, sans-serif;
}
.conversation ul li:nth-child(even) {
  background-color: #e8e8e8;
  font-family: Helvetica, Arial, sans-serif;
}
.conversation .label {
  font-weight: bold;
}
.user_1 {
  color: #c00;
}
.user_2 {
  color: #00c;
}

.caption {
  font-size: 14px;
  font-weight: bold;
  color: #444;
  margin-top: 10px;
  padding: 0px 20px 0px 20px;
}
.caption a {
  color: #444;
}

.regular {
  font-size: 12px;
  color: #444;
  line-height: 17px;
}
.regular h2 {
  font-size: 18px;
  font-weight: bold;
  color: #6498cc;
  letter-spacing: -1px;
  margin: 0px 0px 5px 0px;
}
.regular blockquote {
  font-style: italic;
  border-left: solid 2px #444;
  padding-left: 10px;
}

.total-post {
  font-family: Helvetica, Arial, sans-serif;
  display: flex;
  justify-content: flex-end;
  position: relative;
  border-bottom: solid 1px #ccc;
  width: 600px;
  text-align: center;
  margin: 5px auto 0.5em auto;
}
.RSS {
  font-family: Helvetica, Arial, sans-serif;
  display: flex;
  justify-content: flex-end;
  position: relative;
  width: 600px;
  text-align: center;
  margin: 5px auto 1em auto;
}
/* .total-post span {
  margin-bottom: 5px;
  background-color: #ccc;
} */
</style>
