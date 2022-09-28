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
        <b-form-input v-model="props.title"></b-form-input>
        <b-input-group-append>
          <b-btn @click="props.title=''">&#10005;</b-btn>
        </b-input-group-append>
      </b-input-group>
    </b-form-group>
    <!-- Rom URL -->
    <b-form-group label="Rom URL">
      <b-input-group>
        <b-form-input v-model="props.rom"></b-form-input>
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
    <div class="d-flex justify-content-center">
      <div class="h3">{{props.title ? props.title : "Preview"}}</div>
    </div>
    <div class="d-flex justify-content-center">
      <iframe :src="link" frameborder="0" height="300"></iframe>
    </div>
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
      types: [
        {
          label: "Nintendo",
          options: [
            { text: "Game Boy", value: { type: "vba-m-gb", assets: "gb" } },
            { text: "Game Boy Color", value: { type: "vba-m-gbc", assets: "gbc" } },
            { text: "Game Boy Advance", value: { type: "vba-m-gba", assets: "gba" } },
            { text: "NES", value: { type: "fceux", assets: "nes" } },
            { text: "SNES", value: { type: "snes9x", assets: "snes" } },
            { text: "Nintendo 64", value: { type: "parallel-n64", assets: "n64" } },
          ]
        },
        {
          label: "Sega",
          options: [
            { text: "Game Gear", value: { type: "genplusgx-gg", assets: "gg" } },
            { text: "Genesis", value: { type: "genplusgx-md", assets: "genesis" } },
            { text: "Master System", value: { type: "genplusgx-sms", assets: "sms" } },
            { text: "SG-1000", value: { type: "genplusgx-sg", assets: "sg1000" } },
          ]
        }
      ],
    }
  },
  computed: {
    link() {
      let props = Buffer.from(JSON.stringify(this.props)).toString("base64")
      return `https://play.webrcade.com/app/standalone/?app=app%2F${'gba'}%2F&props=${props}&ctx=standalone`
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
    }
  }
}
</script>
