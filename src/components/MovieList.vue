<template>
  <div class="app">
    <v-app>
      <v-card>
        <v-card-title>
          Movie List
          <v-spacer></v-spacer>
          <v-text-field
            v-model="search"
            append-icon="search"
            label="Search"
            single-line
            hide-details
          ></v-text-field>
        </v-card-title>
        <v-data-table
          v-if="movies.length > 0"
          :headers="headers"
          :items="movies"
          hide-actions
          class="elevation-1"
        >
          <template v-slot:items="props">
            <td class="text-xs-left">{{ props.item.Title }}</td>
            <td class="text-xs-left">{{ props.item.Year }}</td>
            <td class="text-xs-left">{{ props.item.imdbID }}</td>
            <td class="text-xs-right" :key="props.item.star">
              <v-btn
                fab
                dark
                small
                :color="props.item.star ? 'primary' : 'pink'"
                @click="starHandle(props.item)"
              >
                <v-icon dark>favorite</v-icon>
              </v-btn>
            </td>
          </template>
          <template v-slot:no-results>
            <v-alert :value="true" color="primary" icon="warning">
              Your search for "{{ search }}" found no results.
            </v-alert>
          </template>
        </v-data-table>
        <div class="text-xs-center my-2">
          <v-pagination
            v-model="curPage"
            :total-visible="7"
            circle
            :length="totalPage"
            @input="getPage"
          ></v-pagination>
        </div>
      </v-card>
      <v-container fluid>
        <v-layout row wrap>
          <v-flex xs12>
            <h1 class="my-3">Favorite Movies</h1>
          </v-flex>
        </v-layout>

        <div class="starMovieCardWrapper">
          <v-card
            v-for="(movie, key) in starMovies"
            :key="key"
            class="starMovieCard mx-2 my-2"
          >
            <div class="starMovies py-3">
              <span class="title font-weight-bold">{{ movie.Title }}</span>
              <span class="body-1 font-italic py-3">( {{ movie.Year }} )</span>
              <span class="caption">ID: {{ movie.imdbID }}</span>
            </div>
          </v-card>
        </div>
      </v-container>
    </v-app>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  name: 'movieList',
  data() {
    return {
      movies: [],
      totalMovie: 0,
      totalPage: 0,
      search: '',
      curPage: 1,
      headers: [
        {
          text: 'Movie Title',
          align: 'left',
          sortable: false,
          value: 'Title',
        },
        {
          text: 'Movie Year',
          align: 'left',
          sortable: false,
          value: 'Year',
        },
        {
          text: 'Movie imdb ID',
          align: 'left',
          sortable: false,
          value: 'imdbID',
        },
        {
          text: 'Add to My Favorites',
          align: 'right',
          sortable: false,
          value: 'action',
        },
      ],
      starMovies: [],
    };
  },
  methods: {
    getPage(value) {
      this.getMovies(this.search, value);
    },
    async getMovies(title, page) {
      await axios.get(`https://jsonmock.hackerrank.com/api/movies/search?Title=${title}&page=${page}`)
        .then((res) => {
          this.movies = res.data.data.map(item => ({
            Title: item.Title,
            Year: item.Year,
            imdbID: item.imdbID,
            star: false,
          }));
          this.totalPage = res.data.total_pages;
          this.totalMovie = res.data.total;
        })
        .catch((err) => {
          // eslint-disable-next-line
          console.log('error: ', err);
        });
    },
    starHandle(item) {
      if (!item.star) this.starMovies.push(item);
      else this.starMovies = this.starMovies.filter(ele => ele !== item);
      const ind = this.movies.indexOf(item);
      this.movies[ind].star = !item.star;
    },
  },
  created() {
    this.getMovies('', 1);
  },
  watch: {
    search(value) {
      this.getMovies(value, this.curPage);
    },
  },
};
</script>

<style scoped>
.starMovieCardWrapper {
  display: grid;
  grid-template-columns: 1fr 1fr 1fr 1fr 1fr;
}
.starMovies {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}
</style>
