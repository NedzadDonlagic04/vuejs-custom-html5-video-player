<template>
    <div class="video-container">
        <video class="video" src="./../assets/movie.mp4" autoplay></video>
        <div class="controls">
            <div class="progress-bar">
                <div class="current-bar">
                </div>
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
        height: 50px;
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
        width: 100%;
        height: .5rem;
    }

    .current-bar {
        width: 90%;
        height: 100%;
        background-color: var(--bar-color);
    }
</style>