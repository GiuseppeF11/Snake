<script>
import Snake_Body from './Snake_Body.vue';

export default {
    data() {
        return {
            isMuted: false,
            audioSource: "/public/sound/Tetris 99 - Main Theme.mp3", // Sostituisci con il percorso del tuo file audio
            isAudioPlaying: false // Aggiungi la variabile per tracciare lo stato della riproduzione audio
        };
    },
    components: {
        Snake_Body,
    }, 
    methods: {
        // Funzione per mute/unmute
        toggleMute() {
            this.$refs.audioPlayer.muted = !this.$refs.audioPlayer.muted;
            this.isMuted = !this.isMuted;
        },
        // Funzione per avviare/fermare la riproduzione audio
        togglePlayPause() {
            if (this.isAudioPlaying) {
                this.$refs.audioPlayer.pause();
            } else {
                this.$refs.audioPlayer.play();
            }
            this.isAudioPlaying = !this.isAudioPlaying;
        }
    }
};
</script>

<template>
    <main>
        <section class="audio-player-container row ">
            <div class="audio-controls">
                <!-- Bottone per mute/unmute -->
                <button class="text-start px-2" @click="toggleMute">
                    <i :class="isMuted ? 'fa fa-volume-off' : 'fa fa-volume-up'"></i>
                </button>
                <!-- Bottone per avviare/fermare la riproduzione -->
                <button @click="togglePlayPause">
                    <i :class="isAudioPlaying ? 'fa fa-pause' : 'fa fa-play'"></i>
                </button>
            </div>
            <!-- Elemento audio con loop -->
            <audio ref="audioPlayer" :src="audioSource" loop></audio>
        </section>
        <section>
            <Snake_Body/>
        </section>
    </main>
</template>

<style lang="scss" scoped>

.audio-controls button {
    background: none;
    border: none;
    font-size: 24px;
    color: #333;
    cursor: pointer;
    margin: 0 5px;
    width: 40px;
    transition: color 0.3s ease;
    &:hover {
        color: #666;
    }
}

.audio-controls input[type="range"]::-webkit-slider-thumb {
    -webkit-appearance: none;
    width: 15px;
    height: 15px;
    border-radius: 50%;
    background: #333;
    cursor: pointer;
}
</style>
