<template>
  <div class="titleWording">
    <img
      class="statusIcon"
      :src="iconLink"
    /> {{tipTitle}}
    <div
      class="messageWording"
      @click="updateAccount"
    >
    <slot name="content"></slot>
    </div>
  </div>
</template>

<script>
export default {
  name: 'TipContainer',
  props: {
    tipTitle: String,
    tipContent: String,
    tipType: String,
    iconLink: String
  },
  methods: {
    updateAccount (e) {
      if (e.target.nodeName === 'STRONG') {
        this.$emit('triggerUpdate', this.tipType)
      }
    }
  }
}
</script>

<style>
.titleWording {
  width: 100%;
  font-size: 25px;
  color: rgb(0, 109, 204);
  padding-bottom: 20px;
  padding: 20px;
}

.statusIcon {
  height: 35px;
  margin-right: 20px;
  margin-top: -3px;
  max-width: 100%;
  vertical-align: middle;
  box-sizing: border-box;
}

.messageWording {
  font-size: 16px;
  color: rgb(0, 109, 204);
  margin-left: 65px;
  margin-top: 5px;
  box-sizing: border-box;
}

.messageWording a {
  color: rgb(0, 109, 204);
}
</style>
