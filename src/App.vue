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
        <span class="real-text">{{ frequency }}</span>
      </div>
    </div>
    <div class="control-container">
      <div class="center-panel control-panel-container">
        <button class="button-painel" @click.stop="aumentarVolume"></button>
        <button class="button-painel" @click.stop="diminuirVolume"></button>
      </div>
      <div class="left-panel side-panel control-panel-container">
        <button class="button-painel"></button>
        <button class="button-painel" @click.stop="zerarFrequencia"></button>
      </div>
      <div class="right-panel side-panel control-panel-container">
        <button class="button-painel" @click.stop="definirFrequencia"></button>
        <button class="button-painel"></button>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "App",
  data: () => ({
    message: "Desconectado",
    activeFrequency: false,
    frequency: 0,
    battery: 0,
    volume: 2,
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
  methods: {
    zerarFrequencia: function () {
      this.frequency = 0;
    },
    definirFrequencia: function () {
      this.activeFrequency = !this.activeFrequency;
    },
    aumentarVolume: function () {
      if (!this.activeFrequency) {
        if (this.volume < this.volumeIcons.length - 1) this.volume++;
      } else {
        if (this.frequency < 9999) this.frequency++;
      }
    },
    diminuirVolume: function () {
      if (!this.activeFrequency) {
        if (this.volume > 0) this.volume--;
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
  },
  async mounted() {
    await this.$nextTick();
    this.animarBateria();
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
  // background: transparent !important;
  background: black;
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
}

#app img {
  z-index: 20;
  position: relative;
}

#app .radio-display {
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
