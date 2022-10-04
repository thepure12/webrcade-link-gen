<template>
  <div>
    <!-- System -->
    <b-form-group label="System">
      <b-input-group>
        <b-form-select v-model="system" :options="types" @change="props.type=$event.type"></b-form-select>
        <b-input-group-append>
          <b-btn @click="system=''">&#10005;</b-btn>
        </b-input-group-append>
      </b-input-group>
    </b-form-group>
    <!-- Title -->
    <b-form-group label="Game Title">
      <b-input-group>
        <b-form-input v-model="props.title" debounce="3000"></b-form-input>
        <b-input-group-append>
          <b-btn @click="props.title=''">&#10005;</b-btn>
        </b-input-group-append>
      </b-input-group>
    </b-form-group>
    <!-- Rom Set Url -->
    <b-form-group label="Rom Set URL">
      <b-input-group>
        <b-form-input v-model="romSet" placeholder="https://<website>.org/details/<roms-set>"></b-form-input>
        <b-input-group-append>
          <b-btn @click="romSet='';roms=[]">&#10005;</b-btn>
          <b-btn variant="primary" @click="fetchRoms">
            <b-spinner v-if="fetchingRoms" variant="white" label="Spinning" small></b-spinner>
            <span v-else>Go</span>
          </b-btn>
        </b-input-group-append>
      </b-input-group>
    </b-form-group>
    <!-- Rom URL -->
    <b-form-group label="Rom URL">
      <b-input-group>
        <v-select v-if="roms.length" v-model="props.rom" :options="roms" label="name" :reduce="rom => rom.url">
        </v-select>
        <b-form-input v-else v-model="props.rom"></b-form-input>
        <b-input-group-append>
          <b-btn @click="props.rom=''">&#10005;</b-btn>
        </b-input-group-append>
      </b-input-group>
    </b-form-group>
    <!-- Thumbnail URL -->
    <b-form-group>
      <template #label>
        Icon URL
        <span v-if="props.type">
          (<a :href="`https://github.com/webrcade-assets/webrcade-assets-${system.assets}-images/find/master`"
            target="_blank">Webrcade Assets</a>)
        </span>
      </template>
      <b-input-group>
        <b-form-input v-model="props.icon"></b-form-input>
        <b-input-group-append>
          <b-btn @click="props.icon=''">&#10005;</b-btn>
        </b-input-group-append>
      </b-input-group>
      <img v-if="props.icon" class="m-1 border rounded" :src="props.icon" alt="Not found" width="100" height="100" />
    </b-form-group>
    <!-- Preview -->
    <template v-if="system && props.rom">
      <div class="d-flex justify-content-center">
        <div class="h3">{{props.title ? props.title : "Preview"}}</div>
      </div>
      <div class="d-flex justify-content-center">
        <div class="frame-container" :class="fullscreen?'fullscreen':''">
          <iframe :src="link" frameborder="0" height="300"></iframe>
          <b-btn @click="fullscreen=!fullscreen">&#x26F6;</b-btn>
        </div>
      </div>
    </template>
    <!-- Link -->
    <b-form-group label="Standalone Link">
      <b-input-group>
        <b-form-input v-model="link" disabled></b-form-input>
        <b-input-group-append>
          <b-btn @click="copyLink">Copy</b-btn>
          <b-btn :href="link" variant="primary" target="_blank">Go</b-btn>
        </b-input-group-append>
      </b-input-group>
    </b-form-group>
  </div>
</template>

<script>
export default {
  data() {
    return {
      props: {
        type: "",
        title: "",
        app: "",
        icon: "",
        rom: ""
      },
      system: "",
      romSet: "",
      fetchingRoms: false,
      roms: [],
      fullscreen: false,
      types: [
        {
          label: "Nintendo",
          options: [
            { text: "Game Boy", value: { type: "vba-m-gb", assets: "gb", app: "gba" } },
            { text: "Game Boy Color", value: { type: "vba-m-gbc", assets: "gbc", app: "gba" } },
            { text: "Game Boy Advance", value: { type: "vba-m-gba", assets: "gba", app: "gba" } },
            { text: "NES", value: { type: "fceux", assets: "nes", app: "nes" } },
            { text: "SNES", value: { type: "snes9x", assets: "snes", app: "snes" } },
            { text: "Nintendo 64", value: { type: "parallel-n64", assets: "n64", app: "n64" } },
          ]
        },
        {
          label: "Sega",
          options: [
            { text: "Game Gear", value: { type: "genplusgx-gg", assets: "gg", app: "gg" } },
            { text: "Genesis", value: { type: "genplusgx-md", assets: "genesis", app: "genesis" } },
            { text: "Master System", value: { type: "genplusgx-sms", assets: "sms", app: "sms" } },
            { text: "SG-1000", value: { type: "genplusgx-sg", assets: "sg1000", app: "sg1000" } },
          ]
        }
      ],
    }
  },
  computed: {
    link() {
      let props = Buffer.from(JSON.stringify(this.props)).toString("base64")
      return `https://play.webrcade.com/app/${this.system.app}/?props=${props}&ctx=standalone`
    }
  },
  watch: {
    fullscreen(val) {
      if (val) document.body.classList.add("no-overflow")
      else document.body.classList.remove("no-overflow")
    }
  },
  methods: {
    copyLink() {
      navigator.clipboard.writeText(this.link)
      this.$bvToast.toast("Enjoy!", {
        title: 'Standalone Link Copied',
        autoHideDelay: 5000,
        variant: "success",
        static: true,
        toaster: "b-toaster-top-center"
      })
    },
    async fetchRoms() {
      if (this.romSet) {
        this.fetchingRoms = true
        await this.$axios.get(`https://iaapi.thepure12.repl.co/files?url=${this.romSet}`)
          .then(res => this.roms = res.data.files)
          .catch(err => this.$bvToast.toast("Error", {
            title: 'There was an error fetching rom set. Please check url and try again.',
            autoHideDelay: 5000,
            variant: "danger",
            static: true,
            toaster: "b-toaster-top-center"
          }))
        this.fetchingRoms = false
      }
    }
  }
}
</script>
<style>
.v-select {
  display: flex;
  flex-flow: column;
  flex: 1 1 auto;
}

.vs__dropdown-toggle {
  flex: 1 1 auto;
  padding: 0 !important;
}

.vs__selected {
  margin: 0 !important;
}

.vs__clear {
  display: none;
}

.frame-container {
  position: relative;
}

.frame-container>button {
  position: absolute;
  right: 0;
}

.fullscreen,
.fullscreen>iframe {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: 1000;
}

.fullscreen>button {
  z-index: 1001;
}

.no-overflow {
  overflow: hidden;
}
</style>