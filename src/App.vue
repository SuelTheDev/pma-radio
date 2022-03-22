<template>
  <div id="app">
    <img src="@/assets/radio.png" />
    <div class="radio-display">
      <div class="top-container">
        <i class="fa-solid font-size-small" :class="volumeIcons[volume]"></i>
        <span class="notification-text">{{ message }}</span>
        <i
          class="fa-solid font-size-small-2"
          :class="batteryIcons[battery]"
        ></i>
      </div>
      <div class="text-container">
        <span class="placeholder">0000</span>
        <span class="real-text">{{ frequencyDisplay }}</span>
      </div>
    </div>
    <div class="control-container">
      <div class="center-panel control-panel-container">
        <button class="button-painel" @click.stop="aumentarVolume"></button>
        <button class="button-painel" @click.stop="diminuirVolume"></button>
      </div>
      <div class="left-panel side-panel control-panel-container">
        <button class="button-painel" @click.stop="ativarDesativarSom"></button>
        <button class="button-painel" @click.stop="desconectar"></button>
      </div>
      <div class="right-panel side-panel control-panel-container">
        <button class="button-painel" @click.stop="escolherFrequencia"></button>
        <button
          class="button-painel"
          @click.stop="selecionarFrequencia"
        ></button>
      </div>
    </div>
  </div>
</template>

<script>
import gsap from "gsap";

export default {
  name: "App",
  data: () => ({
    message: "Desconectado",
    activeFrequency: true,
    frequencyDisplay: 0,
    battery: 0,
    volume: 2,
    lastVolume: -1,
    is_muted: false,
    tw: null,
    batteryIcons: [
      "fa-battery-empty",
      "fa-battery-quarter",
      "fa-battery-half",
      "fa-battery-full",
    ],
    volumeIcons: [
      "fa-volume-xmark",
      "fa-volume-off",
      "fa-volume-low",
      "fa-volume-high",
    ],
  }),
  watch: {
    frequency: function(val) {
      this.frequencyDisplay = val
    },
  }, 
  methods: {
    sendRequest(data) {
      this.$axios.post("pma-radio", data).then(resp => {
        if (!resp.status){
          this.frequencyDisplay = "Error"
        } else {
          switch (resp.type) {
            case "connected": 
              this.message = "Connectado"
              break;
            case "disconnected":
              this.message = "Desconectado"
              this.frequency = 0
              break;
          }
        }
      }).catch(() => {
        this.frequencyDisplay = "Erro";
      });
    },

    selecionarFrequencia: function () {
      this.sendRequest({ action: "SET_FREQUENCY", payload: this.frequency });
    },
    ativarDesativarSom: function () {
      this.is_muted = !this.is_muted;
      if (this.is_muted) {
        this.lastVolume = this.volume;
        this.volume = 0;
      } else {
        this.volume = this.lastVolume;
      }
      this.sendRequest({ action: "MUTE_VOLUME", payload: this.is_muted });
    },
    desconectar: function () {
      this.frequency = 0;
      this.sendRequest({ action: "DISCONNECT" });
    },
    escolherFrequencia: function () {
      this.activeFrequency = !this.activeFrequency;
    },
    aumentarVolume: function () {
      if (!this.activeFrequency) {
        if (this.volume < this.volumeIcons.length - 1) {
          this.volume++;
          this.sendRequest({ action: "CHANGE_VOLUME", payload: this.volume });
        }
      } else {
        if (this.frequency < 9999) this.frequency++;
      }
    },
    diminuirVolume: function () {
      if (!this.activeFrequency) {
        if (this.volume > 0) {
          this.volume--;
          this.sendRequest({ action: "CHANGE_VOLUME", payload: this.volume });
        }
      } else {
        if (this.frequency > 0) this.frequency--;
      }
    },
    animarBateria: function () {
      setInterval(() => {
        if (this.battery >= 3) this.battery = -1;
        this.battery++;
      }, 1000);
    },
    onMessage(ev) {
      const { data } = ev;
      if (data.action) {
        if (data.action == "show") {
          if (!this.tw || (this.tw && !this.tw.isActive())) {
            this.tw = gsap
              .to("#app", { duration: 1, bottom: -160 })
              .eventCallback("onReverseComplete", null);
          }
        }
      }
    },
    onkeydown(ev) {
      const { key } = ev;
      if (key.toLowerCase() == "escape") {
        if (!this.tw.isActive()) {
          this.tw.eventCallback("onReverseComplete", () => {
            this.sendRequest({ action: "CLOSE_NUI" });
          });
          this.tw.reverse();
        }
      }
    },
  },
  async mounted() {
    await this.$nextTick();
    window.addEventListener("message", this.onMessage);
    window.addEventListener("keydown", this.onkeydown);
    this.animarBateria();
  },
  beforeDestroy() {
    window.removeEventListener("message", this.onMessage);
    window.removeEventListener("keydown", this.onkeydown);
  },
};
</script>

<style lang="scss">
@font-face {
  font-family: "lcd";
  src: url("@/assets/lcd_font.ttf");
}

* {
  user-select: none !important;
}

html {
  margin: 0;
  padding: 0;
}

body {
  background: transparent !important;
  color: white;
  overflow: hidden;
  display: flex;
  flex-direction: row;
  justify-content: flex-end;
  align-items: flex-end;
  height: 100vh;
}

#app {
  position: relative;
  color: rgba(0, 0, 0, 0.85);
  bottom: -850px;

  img {
    z-index: 20;
    position: relative;
  }

  .radio-display {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    height: 75px;
    width: 95px;
    padding: 2px;
    background: #7fa304;
    z-index: 19;

    .top-container {
      display: flex;
      justify-content: space-between;
      padding-right: 4px;

      .notification-text {
        margin: 0.1em;
        font-family: "Times New Roman", Times, serif;
        font-size: 0.5em;
        flex: 1 1 0;
      }
    }

    .text-container {
      font-family: "LCD";
      font-size: 2.8em;
      position: absolute;
      left: 50%;
      transform: translateX(-50%);

      .placeholder {
        color: rgba(0, 0, 0, 0.377);
      }

      .real-text {
        position: absolute;
        right: 0;
        width: 100%;
        direction: rtl;
      }
    }
  }
}

.control-panel-container {
  display: flex;
  flex-direction: column;
  width: 40px;
  align-content: center;
  position: absolute;
}

.control-container {
  position: absolute;
  z-index: 100;
  left: 50%;
  top: 50%;
  height: 83px;
  width: 115px;
  transform: translate(-50%, 50%);

  .center-panel {
    left: 50%;
    top: 50%;
    transform: translate(-55%, -50%);
    button {
      margin-top: 2px;
      height: 34px;
      &:first-child {
        border-top-left-radius: 10px;
        border-top-right-radius: 10px;
      }
      &:last-child {
        border-bottom-left-radius: 10px;
        border-bottom-right-radius: 10px;
        margin-top: 2px;
      }
    }
  }

  .left-panel {
    left: -1px;
    top: 2px;
    button {
      height: 42px;
      width: 32px;
      &:first-child {
        border-top-left-radius: 5px;
        border-top-right-radius: 0px;
      }
      &:last-child {
        border-bottom-left-radius: 20px;
        border-bottom-right-radius: -10px;
        height: 40px;
      }
    }
  }

  .right-panel {
    right: 4px;
    top: 2px;
    button {
      height: 42px;
      width: 32px;
      &:first-child {
        border-top-right-radius: 5px;
        border-top-left-radius: 0px;
      }
      &:last-child {
        border-bottom-right-radius: 20px;
        border-bottom-left-radius: -10px;
        height: 40px;
      }
    }
  }
}

button.button-painel {
  outline: none;
  background: transparent;
}

.font-size-small {
  font-size: 0.7em;
}

.font-size-small-2 {
  font-size: 0.8em;
}

.side-panel {
  width: 31px;
  height: 100%;
}
</style>
