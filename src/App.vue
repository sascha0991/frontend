<template>
  <div id="app">
    <!--    <form class="upload-group">

    <button @click="launchFilePicker">
      Browse
    </button>
    <input ref="file"
           type="file"
           accept=".csv"
           @change="onFileChange( $event.target.files)"/>
    <p v-if="file">{{file[0].name}}</p>
    <button
        :disabled="!file"
        @click="saveData"> OK
    </button>
  </form>
    -->
    <div class="tabs">
      <div @click="showImage=false" :class="{'selected': !showImage}" class="tab">Show subgraph</div>
      <div @click="loadImage" :class="{'selected': showImage}" class="tab">Show image</div>
    </div>
    <div v-if="!showImage">
      <svg width="100%" height="100%">
        <defs>
          <marker id="m-end" markerWidth="10" markerHeight="10" refX="9" refY="3" orient="auto"
                  markerUnits="strokeWidth">
            <path d="M0,0 L0,6 L9,3 z"></path>
          </marker>
        </defs>
      </svg>

      <d3-network ref='net' :net-nodes="nodes" :net-links="links" :options="options" :link-cb="lcb"/>
    </div>
    <div v-else>
      <div class="imageWrapper">
        <img id="image" alt="network">
      </div>
    </div>
  </div>
</template>

<script>
import D3Network from 'vue-d3-network';
//import backend from '@/backend';
import axios from 'axios'

export default {
  name: 'App',
  components: {
    D3Network
  },
  data() {
    return {
      showImage: false,
      image: null,
      receivedData: "",
      nodes: [
        {id: 1},
        {id: 2},
        {id: 3}],
      links: [
        {sid: 1, tid: 2},
        {sid: 2, tid: 3},
        {sid: 3, tid: 1},],
      nodeSize: 14,
      canvas: false,
      file: null,
    }
  },
  computed: {
    options() {
      return {
        force: 3000,
        size: {w: 1000, h: 1000},
        nodeSize: this.nodeSize,
        nodeLabels: true,
        canvas: this.canvas,
        linkWidth: 2
      }
    },
    receivedNodes() {
      if (this.receivedData.data !== undefined) {
        return this.receivedData.data.nodes;
      } else {
        return [];
      }
    },
    receivedLinks() {
      if (this.receivedData.data !== undefined) {
        return this.receivedData.data.links;
      } else {
        return [];
      }
    }
  },
  created() {
    this.get_nodes();
    this.get_links();
  },
  methods: {
    lcb(link) {
      link._svgAttrs = {
        'marker-end': 'url(#m-end)'
      }
      link._color = 'gray';
      return link
    },
    get_json() {
      axios.get('http://localhost:5000/json', {
        headers: {
          'Content-Type': 'application/json'
        }
      }).then(resp => {
        this.receivedData = resp.data;
      }).catch(error => {
        console.log(error);
      });
    },
    get_nodes() {
      axios.get('http://localhost:5000/nodes', {
        headers: {
          'Content-Type': 'application/json'
        }
      }).then(resp => {
        console.log(resp)
        this.nodes = resp.data.nodes;
      }).catch(error => {
        console.log(error);
      });
    },
    get_links() {
      //const axios = require('axios');

      axios.get('http://localhost:5000/links', {
        headers: {
          'Content-Type': 'application/json'
        }
      }).then(resp => {
        this.links = resp.data.links;
      }).catch(error => {
        console.log(error);
      });
    }, launchFilePicker() {
      this.$refs.file.click();
    },
    async getImage() {
      axios.get('http://localhost:5000/image', {responseType: 'blob'})
          .then(response => {
            let imageNode = document.getElementById('image');
            let imgUrl = URL.createObjectURL(response.data)
            imageNode.src = imgUrl
          })
    },
    async onFileChange(file) {
      this.file = file;
    },
    saveData() {
      let formData = new FormData();
      formData.append("file", this.file[0]);
      axios.post("http://localhost:5000/upload", formData, {
        headers: {
          'Content-Type': 'multipart/form-data'
        }
      }).then(function () {
        console.log("success")
      }).catch(function (error) {
        console.log("error")
        console.log(error)
      })

    },
    loadImage() {
      this.showImage = true;
      this.image = this.getImage();
    }
  }
}
</script>

<style>
@import url('https://fonts.googleapis.com/css?family=PT+Sans');

#app {
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
  font-family: 'PT Sans', sans-serif;
  background-color: #eee;
}

h1, a {
  color: #1aad8d;
  text-decoration: none;
}

ul.menu {
  list-style: none;
  position: absolute;
  z-index: 100;
  min-width: 20em;
  text-align: left;
}

ul.menu li {
  margin-top: 1em;
  position: relative;
}

#m-end path {
  fill: gray;
}

.tabs {
  display: flex;
  cursor: pointer;
  background-color: greenyellow;
  font-size: 2rem;
  width: 100%;
}

.tab {
  width: 50%;
}

.selected {
  font-weight: bolder;
  background-color: green;
}

.imageWrapper {
  width: 100%;
  height: 100%;
}

img {
  width: 1000px;
}

</style>
