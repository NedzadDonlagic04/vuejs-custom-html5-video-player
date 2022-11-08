<script>
export default {
    data() {
        return {
            playPause: 'play',
            isMounted: false,
            playShow: true,
            pauseShow: false,
            currentTime: null,
            duration: null
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
                console.log(this.currentTime, this.duration);
            }
            return `${percent}%`;
        }
    },
    methods: {
        playPauseEvent() {
            if(this.playPause === 'play') {
                this.playPause = 'pause';
                this.$refs.video.play();

                this.playShow = false;
                this.pauseShow = true;
            } else {
                this.playPause = 'play';
                this.$refs.video.pause();

                this.playShow = true;
                this.pauseShow = false;
            }
        },
        metadataLoaded(event) {
            this.duration = event.target.duration;
        },
        timeUpdateEvent(event) {
            this.currentTime = event.target.currentTime; 
        }
    }
}
</script>

<template>
    <div class="video-container">
        <video ref="video" class="video" src="./../assets/movie.mp4" @click="playPauseEvent" @loadedmetadata="metadataLoaded" @timeupdate="timeUpdateEvent"></video>
        <div class="controls">
            <div class="progress-bar">
                <div class="current-bar" :style="{ width: progress }">
                </div>
            </div>
            <div class="buttons">
                <button @click="playPauseEvent">
                    <img v-show="playShow" src="./../assets/icons/play.png" alt="play button">
                    <img v-show="pauseShow" src="./../assets/icons/pause.png" alt="pause button">
                </button>
            </div>  
        </div>
    </div>
</template>

<style scoped>
    .video-container {
        width: 100%;
        max-width: 900px;
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
    }

    .current-bar {
        height: var(--bar-height);
        background-color: var(--bar-color);
    }

    .buttons {
        padding-top: .2rem;
        padding-left: .5rem;
        user-select: none;
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
</style>