<template>
    <div class="container">
        <div class="wrapper-panels">
            <div
                @mouseup="penalUp(1)"
                @mousedown="penalDown(1)"
                :class="{active: isActive1}"
                class="panel top-left-panel"
            ></div>
            <div
                @mouseup="penalUp(2)"
                @mousedown="penalDown(2)"
                :class="{active: isActive2}"
                class="panel top-right-panel"
            ></div>
            <div
                @mouseup="penalUp(3)"
                @mousedown="penalDown(3)"
                :class="{active: isActive3}"
                class="panel bottom-left-panel"
            ></div>
            <div
                @mouseup="penalUp(4)"
                @mousedown="penalDown(4)"
                :class="{active: isActive4}"
                class="panel bottom-right-panel"
            ></div>
        </div>
        <div class="setting">
            <div class="round-counter">{{ counterLevel }}</div>
            <div class="btn">
                <button :disabled="isDisabledBtn" @click="startGame">Начать</button>
            </div>
            <div class="total-info" v-if="isEnd">{{ totalInfo }}</div>
            <div class="options">
                <div class="title"><b>Сложность</b></div>
                <label for="isi">Легкий<input v-model="options" :value="1500" name="options" id="isi"
                                              type="radio"></label>
                <label for="middle">Нормальный<input v-model="options" :value="1000" name="options" id="middle"
                                                     type="radio"></label>
                <label for="hard">Сложный<input v-model="options" :value="400" name="options" id="hard"
                                                type="radio"></label>
            </div>
        </div>
    </div>
</template>

<script>
import audio1 from '../assets/1.ogg'
import audio2 from '../assets/2.ogg'
import audio3 from '../assets/3.ogg'
import audio4 from '../assets/4.mp3'
export default {
    name: 'Game',
    data() {
        return {
            panels: [ 1, 2, 3, 4 ],
            sequence: [],
            sequenceToGuess: [],
            sounds: {
                audio1: new Audio(audio1),
                audio2: new Audio(audio2),
                audio3: new Audio(audio3),
                audio4: new Audio(audio4)
            },
            round: 0,
            totalRound: 0,
            options: 400,
            isActive1: false,
            isActive2: false,
            isActive3: false,
            isActive4: false,
            canClick: false,
            isEnd: false,
            isDisabledBtn: false
        }
    },
    computed: {
        totalInfo() {
            return `Проигрыш. Вы прошли ${ this.totalRound } уровня`
        },
        counterLevel() {
            return `Уровень ${ this.round }`
        }
    },
    methods: {
        penalUp( panel ) {
            this[`isActive${ panel }`] = false
        },
        penalDown( panel ) {

            if ( !this.canClick ) return

            const expectedPanel = this.sequenceToGuess.shift()

            if ( expectedPanel === panel ) {
                this.sounds[`audio${ panel }`].load()
                this.sounds[`audio${ panel }`].play()
                this[`isActive${ panel }`] = true

                if ( this.sequenceToGuess.length === 0 ) {
                    this.sequence.push(this.getRandomPanel())
                    this.sequenceToGuess = [ ...this.sequence ]
                    // time for the next level
                    new Promise(resolve => setTimeout(() => {
                        this.startFlashing()
                        resolve()
                    }, 1000))
                }
            } else {
                this.isEnd = true
                this.isDisabledBtn = false
                this.sequence = []
                this.sequenceToGuess = []
                this.totalRound = this.round
                this.round = 0
            }
        },
        startGame() {
            this.isDisabledBtn = true
            this.startFlashing()
        },
        async startFlashing() {
            this.isEnd = false
            this.round++
            if ( this.sequence.length === 0 ) {
                this.sequence.push(this.getRandomPanel())
                this.sequenceToGuess = [ ...this.sequence ]
            }
            this.canClick = false
            for ( const panel of this.sequence ) {
                await this.flash(panel)
            }
            this.canClick = true
        },
        flash( panel ) {
            this.sounds[`audio${ panel }`].play()

            return new Promise(resolve => {
                this[`isActive${ panel }`] = true

                //time of flash
                setTimeout(() => {
                    this[`isActive${ panel }`] = false

                    // time between flashes
                    setTimeout(() => resolve(), 250)
                }, this.options)
            })
        },
        getRandomPanel() {
            return this.panels[Math.floor(Math.random() * this.panels.length)]
        }
    }
}
</script>

<style scoped>
.container {
    padding-top: 50px;
    display: flex;
    justify-content: center;
    align-items: center;
}

.wrapper-panels {
    display: flex;
    flex-wrap: wrap;
    flex-basis: 400px;
}

.panel {
    width: 200px;
    cursor: pointer;
    height: 200px;
    opacity: 0.4;
}

.panel:hover {
    border: 2px solid black;
}

.bottom-right-panel {
    border-bottom-right-radius: 100%;
    background-color: red;
}

.bottom-left-panel {
    border-bottom-left-radius: 100%;
    background-color: blue;
}

.top-right-panel {
    border-top-right-radius: 100%;
    background-color: green;
}

.top-left-panel {
    border-top-left-radius: 100%;
    background-color: yellow;
}

.setting {
    padding-left: 50px;
}

.btn {
    padding-top: 15px;
    padding-bottom: 5px;
}

.title {
    padding-top: 30px;
    font-size: 20px;
    padding-bottom: 5px;
}

.total-info {
    position: absolute;
    font-size: 18px;

}

.options {
    display: flex;
    flex-direction: column;
}

.active {
    opacity: 1;
}
</style>
