<template>
  <div id="app">
    <div class="panel scores">
      <div class="score">
        <h1>Jogador</h1>
        <div class="life-bar">
          <div
            class="life"
            :class="{ danger: playerLife < 20 }"
            :style="{ width: playerLife + '%' }"
          ></div>
        </div>
        <div>{{ playerLife }}%</div>
      </div>
      <div class="score">
        <h1>Monstro</h1>
        <div class="life-bar">
          <div
            class="life"
            :class="{ danger: monsterLife < 20 }"
            :style="{ width: monsterLife + '%' }"
          ></div>
        </div>
        <div>{{ monsterLife }}%</div>
      </div>
    </div>
    <div v-if="hasResult" class="panel result">
      <div v-if="monsterLife == 0" class="win">Você ganhou!!! :)</div>
      <div v-else class="lose">Você perdeu! :(</div>
    </div>
    <div class="panel buttons">
      <template v-if="running">
        <button @click="attack(false)" class="btn attack">Ataque</button>
        <button @click="attack(true)" class="btn especial-attack">
          Ataque Especial
        </button>
        <button @click="healAndHurt" class="btn heal">Curar</button>
        <button @click="running = false" class="btn give-up">Desistir</button>
      </template>
      <button v-else @click="startGame" class="btn new-game">
        Iniciar Jogo
      </button>
    </div>
    <div v-if="logs.length" class="panel logs">
      <ul>
        <li
          v-for="(log, index) in logs"
          :class="log.cls"
          class="log"
          :key="index"
        >
          {{ log.text }}
        </li>
      </ul>
    </div>
  </div>
</template>

<script lang="ts">
import { Component, Vue, Watch } from "vue-property-decorator";
import LogsInterface from "./interfaces/LogsInterface";

@Component
export default class App extends Vue {
  running = false;
  playerLife = 100;
  monsterLife = 100;
  logs: Array<LogsInterface> = [];

  get hasResult(): boolean {
    return this.playerLife == 0 || this.monsterLife == 0;
  }

  startGame(): void {
    this.running = true;
    this.playerLife = 100;
    this.monsterLife = 100;
    this.logs = [];
  }

  attack(especial: boolean): void {
    this.hurt("monsterLife", 5, 10, especial, "Jogador", "Monstro", "player");
    if (this.monsterLife > 0) {
      this.hurt("playerLife", 7, 12, false, "Monstro", "Jogador", "monster");
    }
  }

  hurt(
    // eslint-disable-next-line @typescript-eslint/no-explicit-any
    prop: this[any],
    min: number,
    max: number,
    especial: boolean,
    source: string,
    target: string,
    cls: string
  ): void {
    const plus = especial ? 5 : 0;
    const hurt = this.getRandom(min + plus, max + plus);
    this[prop] = Math.max(this[prop] - hurt, 0);
    this.registerLog(`${source} atingiu ${target} com ${hurt}.`, cls);
  }

  healAndHurt(): void {
    this.heal(10, 15);
    this.hurt("playerLife", 7, 12, false, "Monstro", "Jogador", "monster");
  }

  heal(min: number, max: number): void {
    const heal = this.getRandom(min, max);
    this.playerLife = Math.min(this.playerLife + heal, 100);
    this.registerLog(`Jogador ganhou força de ${heal}.`, "player");
  }

  getRandom(min: number, max: number): number {
    const value = Math.random() * (max - min) + min;
    return Math.round(value);
  }

  registerLog(text: string, cls: string): void {
    this.logs.unshift({ text, cls });
  }

  @Watch("hasResult")
  onHasResultChanged(value: string) {
    if (value) this.running = false;
  }
}
</script>

<style>
/* Geral */

html {
  font-family: "Montserrat", sans-serif;
}

#app {
  display: flex;
  flex-direction: column;
}

.panel {
  margin: 10px;
  padding: 20px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.15);
}

/* Área dos Pontos */

.scores {
  display: flex;
}

.score {
  flex: 1;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}

.score h1 {
  font-weight: 300;
  font-size: 1.6rem;
}

.life-bar {
  width: 80%;
  height: 20px;
  border: 1px solid #aaa;
}

.life-bar .life {
  display: flex;
  justify-content: center;
  height: 100%;
  background-color: green;
}

.life-bar .life.danger {
  background-color: red;
}

/* Área do Resultado */

.result {
  display: flex;
  justify-content: center;
  font-size: 1.3rem;
  font-weight: 600;
}

.result .win {
  color: green;
}

.result .lose {
  color: red;
}

/* Área dos Botões (Controle do Jogo) */

.buttons {
  display: flex;
  justify-content: center;
}

.btn {
  padding: 5px 10px;
  margin: 0px 10px;
  border-radius: 5px;
  text-transform: uppercase;
  font-size: 1.1rem;
}

.new-game {
  background-color: #4253af;
  color: #fff;
}

.attack {
  background-color: #e51c23;
  color: #fff;
}

.especial-attack {
  background-color: #ff9800;
  color: #000;
}

.heal {
  background-color: #259b24;
  color: #fff;
}

.give-up {
  background-color: #bbb;
  color: #000;
}

/* Área dos Logs */

.logs ul {
  display: flex;
  flex-direction: column;
  list-style: none;
  padding: 0;
  margin: 0;
}

.logs ul li {
  display: flex;
  justify-content: center;
  margin: 4px 0px;
  padding: 3px 0px;
  font-weight: 600;
  font-size: 1.1rem;
  text-transform: uppercase;
  border-radius: 3px;
}

.player {
  background-color: #4253afaa;
  color: #fff;
}

.monster {
  background-color: #e51c23aa;
  color: #fff;
}
</style>
