<script>
export default {
    data() {
        return {
            isMounted: false,
            playShow: true,
            pauseShow: false,
            restartShow: false,
            currentTime: null,
            duration: null,
            rewindTime: -10,
            forwardTime: 10,
            audio: true,
            volume: 1,
            barWidth: null,
            mouseDown: false
        }
    },
    mounted() {
        this.isMounted = true;
    },
    computed: {
        progress() {
            let percent;
            if(!this.isMounted || this.duration === null || this.currentTime === null) {
                 percent = 0;
            } else {
                percent = this.currentTime / this.duration * 100;
                // console.log(this.currentTime, this.duration);
                // ^ Was used a lot for testing purposes so I'm leaving it here
            }
            return `${percent}%`;
        },
        timeStamp() {
            const time1 = this.timeToString(this.currentTime);
            const time2 = this.timeToString(this.duration);

            return `${time1} / ${time2}`;
        }
    },
    methods: {
        playPauseEvent() {
            if(this.restartShow) {
                this.restartShow = false;
                this.playShow = true;
            }

            if(this.playShow) {
                this.$refs.video.play();
                this.playShow = false;
                this.pauseShow = true;
            } else {
                this.$refs.video.pause();
                this.playShow = true;
                this.pauseShow = false;
            }
        },
        metadataLoadedEvent(event) {
            this.duration = event.target.duration;
        },
        timeUpdateEvent(event) {
            this.currentTime = event.target.currentTime; 
        },
        endedEvent() {
            this.playShow = false;
            this.pauseShow = false;
            this.restartShow = true;
        },
        timeToString(totalTime) {
            totalTime = Math.floor(totalTime);
            
            let [minutes, hours] = [0, 0];

            if(totalTime >= 60) {
                minutes = Math.floor(totalTime / 60);
                
                if(minutes >= 60) {
                    hours = Math.floor(minutes / 60);
                    minutes = minutes - hours * 60;
                }
            }
            
            const convertToString = val => {
                val = String(val);
                if(val.length < 2) {
                    val = '0' + val;
                }
                
                return val;
            }
            
            totalTime = totalTime - minutes * 60 - hours * 3600;
            totalTime = convertToString(totalTime);
            minutes = convertToString(minutes);
            
            let returnVal = minutes + ':' + totalTime;
            if(hours > 0) {
                returnVal = hours + ':' + returnVal;
            }
            
            return returnVal;
        },
        rewind() {
            if(this.restartShow) return;
            
            if(this.$refs.video.currentTime + this.rewindTime < 0) {
                this.$refs.video.currentTime = 0;
            } else {
                this.$refs.video.currentTime += this.rewindTime;
            }
        },  
        forward() {
            if(this.restartShow) return;

            if(this.$refs.video.currentTime + this.forwardTime > this.duration) {
                this.$refs.video.currentTime = this.duration;
            } else {
                this.$refs.video.currentTime += this.forwardTime;
            }
        },
        audioOnOff() {
            this.$refs.video.muted = this.audio;
            this.audio = !this.audio;

            if(this.audio) {
                this.$refs.volumeBar.value = this.volume * 100;
            } else {
                this.$refs.volumeBar.value = 0;
            }
        },
        volumeChange(event) {
            this.volume = Number(event.target.value) / 100;
            this.$refs.video.volume = this.volume;

            if(this.volume === 0) {
                this.audio = false;
            } else {
                this.audio = true;
            }
        },
        progressBarUpdate(event) {
            if(this.barWidth === null) {
                this.barWidth = event.target.offsetWidth;
            }

            if(this.restartShow) {
                this.playPauseEvent();
            }

            const percent = event.offsetX / this.barWidth;  
            // console.log(percent * this.duration);
            // ^ Was used a lot for testing purposes so I'm leaving it here
            this.$refs.video.currentTime = percent * this.duration;
        },
        mouseMoveUpdate(event) {
            if(!this.mouseDown) return;
            else if(this.barWidth === null) {
                this.barWidth = event.target.offsetWidth;
            }

            if(this.restartShow) {
                this.playPauseEvent();
            }

            const percent = event.offsetX / this.barWidth;  
            this.$refs.video.currentTime = percent * this.duration;
            this.currentTime = percent * this.duration;
        }
    }
}
</script>

<template>
    <div class="video-container">
        <video ref="video" class="video" src="./../assets/movie.mp4" 
            @click="playPauseEvent" 
            @loadedmetadata="metadataLoadedEvent" 
            @timeupdate="timeUpdateEvent" 
            @ended="endedEvent"
        ></video>
        <div class="controls">
            <div @click="progressBarUpdate" 
                 @mousedown="() => mouseDown = true" 
                 @mouseup="() => mouseDown = false"
                 @mousemove="mouseMoveUpdate"
                 class="progress-bar">
                <div class="current-bar" :style="{ width: progress }">
                </div>
            </div>
            <div class="buttons">
                <button @click="playPauseEvent">
                    <img v-show="playShow" src="./../assets/icons/play.png" alt="play button">
                    <img v-show="pauseShow" src="./../assets/icons/pause.png" alt="pause button">
                    <img v-show="restartShow" src="./../assets/icons/restart.png" alt="restart button">
                </button>
                <button class="rewind-forward" @click="rewind">
                    <img src="./../assets/icons/rewind.png" alt="rewind button">
                </button>
                <button class="rewind-forward" @click="forward">
                    <img src="./../assets/icons/forward.png" alt="forward button">
                </button>
                <p class="time">{{timeStamp}}</p>
                <button class="audio" @click="audioOnOff">
                    <img v-show="audio" src="./../assets/icons/audio.png" alt="audio icon">
                    <img v-show="!audio" src="./../assets/icons/no-audio.png" alt="no audio icon">
                </button>
                <input ref="volumeBar" @input="volumeChange" type="range" min="0" max="100" step="1" value="100">
            </div>  
        </div>
    </div>
</template>

<style scoped>
    .video-container {
        width: 100%;
        position: relative;
        overflow-y: hidden;
    }

    .video {
        width: 100%;
    }

    .controls {
        width: 100%;
        height: 3.5rem;
        position: absolute;
        bottom: 0;
        background-color: var(--controls-bg);
        transform: translateY(calc(100% - 5px));
        transition: all .3s;
        display: flex;
        flex-direction: column;
    }

    .video-container:hover .controls {
        transform: translateY(0);
    } 

    .progress-bar {
        height: var(--bar-height);
        width: 100%;
        cursor: pointer;
    }

    .current-bar {
        height: var(--bar-height);
        background-color: var(--bar-color);
    }

    .buttons {
        padding-top: .2rem;
        padding-left: .5rem;
        user-select: none;
        display: flex;
        justify-content: flex-start;
        align-items: center;
    }

    .buttons button {
        background: none;
        border: 0;
        outline: 0;
        cursor: pointer;
    }

    .buttons button img {
        width: 2.5rem;
    }

    .time {
        margin-left: .3rem;
        color: white;
    }

    .rewind-forward,
    .audio {
        scale: .7;
    }

    .audio {
        margin-left: .5rem;
    }

    .audio + input[type="range"] {
        accent-color: white;
        margin-left: .5rem;
    }
</style>