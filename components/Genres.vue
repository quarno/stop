<template>
  <div>
    <div class="box" v-for="genre in genres" :key="genre.name">
      <p class="subtitle" v-text="genre.name" />
    </div>
  </div>
</template>

<script lang="ts">
import Vue, { PropOptions } from "vue";

interface Genre {
  name: string;
  score: number;
}

export default Vue.extend({
  name: "Genres",

  props: {
    timeRange: {
      type: String,
      required: true,
    } as PropOptions<String>,
  },

  data() {
    return {
      loaded: false,
      genres: [] as Genre[],
    };
  },

  async fetch() {
    let genres: Genre[] = [];

    const artists = await this.$axios.$get(
      `https://api.spotify.com/v1/me/top/artists?time_range=${this.timeRange}`
    );
    for (const artist of artists.items) {
      for (const genre of artist.genres) {
        // increment genre score if it exists, otherwise add it
        let existing = genres.filter((element) => element.name == genre);
        if (existing.length == 1) {
          genres[genres.indexOf(existing[0])].score++;
        } else {
          let newGenre: Genre = {
            name: genre,
            score: 1,
          };
          genres.push(newGenre);
        }
      }
    }

    // sort the genres
    genres.sort((first, second) => second.score - first.score);

    // get only 10 items
    this.genres = genres.slice(0, 10);

    this.loaded = true;
  },
});
</script>
