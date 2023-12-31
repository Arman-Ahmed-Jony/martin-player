<template>
  <v-app>
    <v-container fluid>
      <audio
        preload="metadata"
        ref="audio"
        :src="currentTrack"
        @ended="handleEndTrack"
      ></audio>
      <v-row align="center" justify="center">
        <v-col cols="12" sm="8" md="6" lg="4">
          <v-card class="mx-auto" max-width="400">
            <v-card-title class="text-center"
              >{{this.playlists[this.currentPlaylistIndex]?.Name}}: {{currentTrack}}</v-card-title
            >
            <v-card-text>
              <v-row align="center" justify="center">
                <v-col cols="12">
                  <v-slider
                    v-model="value"
                    @update:model-value="handleTrackSlided"
                    :max="max"
                    :min="min"
                    :step="step"
                    thumb-label
                  ></v-slider>
                </v-col>
                <v-col cols="12" class="text-center">
                  <v-btn icon @click="prevTrack"
                    ><v-icon>mdi-skip-previous</v-icon></v-btn
                  >
                  <v-btn icon @click="togglePlayPause"
                    ><v-icon>{{
                      isPlaying ? "mdi-pause" : "mdi-play"
                    }}</v-icon></v-btn
                  >
                  <v-btn icon @click="nextTrack"
                    ><v-icon>mdi-skip-next</v-icon></v-btn
                  >
                  <v-btn
                    icon
                    @click="isLoop = !isLoop"
                    :color="isLoop ? 'red' : 'white'"
                    ><v-icon>mdi-repeat</v-icon></v-btn
                  >
                </v-col>
              </v-row>
            </v-card-text>
          </v-card>
        </v-col>
      </v-row>
    </v-container>
    <v-container fluid>
      <v-row v-for="(playlist, playlistindex) in playlists"
          :key="playlistindex" align="center" justify="center">
        <v-col

          cols="12"
          sm="8"
          md="6"
          lg="4"
        >
          <v-card class="mx-auto" max-width="400">
            <v-card-title class="text-center"
              >Playlist: {{ playlist.Name }}</v-card-title
            >
            <v-card-text
              @click="selectCurrentPlaylistIndex(playlistindex, index)"
              v-for="(music, index) in playlist.list"
              :key="index"
            >
              {{ music }}
            </v-card-text>
          </v-card>
        </v-col>
      </v-row>
    </v-container>
  </v-app>
</template>

<script>
import PLAYLISTS from "@/assets/music.json";
export default {
  data() {
    return {
      value: 0,
      min: 0,
      max: 100,
      step: 1,
      AUDIO: null,
      playlists: [],
      currentTrackIndex: 0,
      currentPlaylistIndex: 0,
      isPlaying: false,
      isLoop: false,
    };
  },
  mounted() {
    this.playlists = PLAYLISTS.playlist;
    this.AUDIO = this.$refs.audio;
    this.AUDIO.addEventListener("playing", () => {
      this.isPlaying = true;
    });
    this.AUDIO.addEventListener("pause", () => {
      this.isPlaying = false;
    });
    this.AUDIO.addEventListener("timeupdate", () => {
      this.value = (this.AUDIO.currentTime / this.AUDIO.duration) * 100;
    });
  },
  computed: {
    currentTrack() {
      try {
        return this.playlists[this.currentPlaylistIndex].list[
          this.currentTrackIndex
        ];
      } catch (error) {
        return "";
      }
    },
    playPauseIcon() {
      return this.isPlaying ? "fa fa-pause" : "fa fa-play";
    },
  },
  methods: {
    async selectCurrentPlaylistIndex(playlistindex, index) {
      this.currentTrackIndex = index;
      this.currentPlaylistIndex = playlistindex;

      await setTimeout(() => {
        this.AUDIO.play();
      }, 1000);
    },
    handleVolumeSlided(val) {
      this.AUDIO.volume = val / 100;
    },
    handleTrackSlided(val) {
      this.AUDIO.currentTime = (this.AUDIO.duration * val) / 100;
    },
    async togglePlayPause() {
      if (this.AUDIO.paused && !this.isPlaying) {
        await this.AUDIO.play();
      } else {
        this.AUDIO.pause();
      }
    },
    async prevTrack() {
      this.currentTrackIndex--;
      if (this.currentTrackIndex < 0) {
        this.currentTrackIndex = this.playlists[this.currentPlaylistIndex].list.length - 1;
      }
      this.AUDIO.src = this.currentTrack;

      await setTimeout(() => {
        this.AUDIO.play();
      }, 1000);
    },
    handleEndTrack() {

      if(!this.isLoop) {
        if(this.playlists[this.currentPlaylistIndex].list.length-1 ===  this.currentTrackIndex){
           this.currentTrackIndex= 0
           this.AUDIO.src = this.currentTrack;
           this.AUDIO.pause();
           return
        }
      }else{

        return this.nextTrack()
      }
      return this.nextTrack()
    },
    async nextTrack() {
      this.currentTrackIndex++;
      if (this.currentTrackIndex >= this.playlists[this.currentPlaylistIndex].list.length) {
        this.currentTrackIndex = 0;
      }
      this.AUDIO.src = this.currentTrack;
      await setTimeout(() => {
        this.AUDIO.play();
      }, 1000);
    },
  },
};
</script>
