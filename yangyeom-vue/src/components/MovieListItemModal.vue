<template>
<div class="modal fade bd-example-modal-lg" tabindex="-1" role="dialog" :id="'movie-' + movie.code">
  <div class="modal-dialog modal-xl" role="document">
    <div class="modal-content">
      <div class="modal-header">
        <h5 class="modal-title">{{ movie.title }} | 평균 평점: {{ movie.score_avg }}점/5.0점</h5>
        <button type="button" class="close" data-dismiss="modal" aria-label="Close">
          <span aria-hidden="true">&times;</span>
        </button>
      </div>
      <div class="modal-body">
        <img :src="poster_url" :alt="movie.title" width="200px">
      </div>
      <div class="modal-body">
        <hr>
        <p>{{ movie.description }}</p>
      </div>
      <div class="modal-body">
        <div v-if="ifIRated">
          <hr>
          <h3>나의 평가</h3>
          <h4>{{myReview[0].score}}점/5.0점</h4>
          <h6>{{myReview[0].username}} | {{myReview[0].content}}</h6>
          <button v-if="writerCheck(myReview[0].user)" @click="reviewDelete(myReview[0].id)">삭제</button>
        </div>
        <hr>
        <h3>다른 사람들의 평가</h3>
        <hr>
        <div v-for="review in movie.review_set" :key="review.id">
          <!-- <div v-if="!writerCheck(review.user)"> -->
            <h4>{{review.score}}점/5.0점</h4>
            <h6>{{review.username}} | {{review.content}}</h6>
            <!-- <button v-if="writerCheck(review.user)" @click="reviewDelete(review.id)">삭제</button> -->
            <hr>
          <!-- </div> -->
        </div>
        <div v-if="isLogined">
            <star-rating v-model="score_rating" :glow="10" :show-rating="false" :increment="0.5" :border-width="1" border-color="#d8d8d8" :rounded-corners="true" :star-points="[23,2, 14,17, 0,19, 10,34, 7,50, 23,43, 38,50, 36,34, 46,19, 31,17]"/>
          <form @submit.prevent="">
            <input type="text" v-model="content_rating" :placeholder="reviewPlaceholder" @change="save">
            <button type="submit">확인</button>
          </form>
        </div>
      </div>
      <div class="modal-footer">
        <button type="button" class="btn btn-secondary" data-dismiss="modal">Close</button>
      </div>
    </div>
  </div>
</div>
</template>

<script>
import axios from 'axios'
import { mapGetters } from 'vuex'
import StarRating from 'vue-star-rating'

export default {
  name: 'movie-list-item-modal',
  props: {
    movie_: {
      type: Object,
      required: true,
    }
  },
  components: {
    StarRating
  },
  data() {
      return {
        reviews: [],
        content_rating: '',
        score_rating: 0,
        movie: this.movie_,
        poster_url: `http://image.tmdb.org/t/p/w500/${this.movie_.poster_url}`
      }
  },
  methods: {
    save() {
      if (!this.ifIRated) {
        console.log('!this.ifIRated', !this.ifIRated)
        const data = {
          content: this.content_rating,
          score: this.score_rating,
          user: this.user,
          movie: this.movie.code
        }
        axios.post(`http://127.0.0.1:8000/api/v1/movie/${this.movie.code}/reviews/`, data, this.options)
          .then(response => {
            console.log(response)
            // const review = {
            //   content: response.data.content,
            //   score: response.data.score,
            //   username: response.data.username,
            //   user: response.data.user,
            //   id: response.data.id
            // }
            // this.movie.review_set.push(review)
            // this.reviews = response.data.review_set
            this.movie = response.data
            this.content_rating = ''
            console.log('리뷰들', this.movie.review_set)
          })
          .catch(error => {
            console.log(error)
          })
      }
    },
    reviewDelete(review_id) {
      let conf = this.options
      conf.user = this.user
      console.log(conf)
      axios.delete(`http://127.0.0.1:8000/api/v1/movie/${this.movie.code}/reviews/${review_id}/`, conf)
        .then(response => {
            console.log(response)
            // this.movie.review_set = this.movie.review_set.filter(review => review.id !== review_id)
            this.movie = response.data
        })
        .catch(error => {
            console.log(error)
        })
      this.content_rating = ''
    },
    writerCheck(writer) {
      // console.log('리뷰주인', writer)
      // console.log('로그인한 유저',this.user)
      return writer === this.user
    },
  },
  mounted(){
    let conf = this.options
    conf.user = this.user
    // console.log('유저', this.user)
    // console.log('리뷰 가져올 때 conf', conf)
      // axios.get(`http://127.0.0.1:8000/api/v1/movie/${this.movie.code}/reviews/`)
      //   .then(response => {
      //       this.movie.review_set = response.data
      //       // this.myReview = this.movie.review_set.filter(review => review.user === this.user)
      //       // if (this.myReview.length !== 0) {
      //       //   this.ifIRated = true
      //       //   console.log(this.movie.title,'에서내가 쓴 리뷰', this.myReview)
      //       //   }
      //   })
      //   .catch(error => {
      //       console.log(error)
      //   })
  },
  computed: {
    ...mapGetters([
      'options',
      'user'
    ]),
    ifIRated() {
      // console.log(this.movie.title, '에서 ifIRated', a)
      return this.movie.review_set.filter(review => review.user === this.user).length !== 0
    },
    myReview() {
      return this.movie.review_set.filter(review => review.user === this.user)
    },
    reviewPlaceholder() {
      return this.isLogined? (this.ifIRated? '재평가하려면 리뷰를 삭제하세요': '평가해주세요') : '로그인해주세요'
    },
    isLogined(){
      this.$session.start()
      return this.$session.has('jwt')
    },
    // score_aavg() {
    //   axios.get(`http://127.0.0.1:8000/api/v1/movie/${this.movie.code}/`)
    //     .then(response => {
    //         console.log('??????????????', response)
    //         this.movie = response.data
    //         return 1
    //     })
    //     .catch(error => {
    //         console.log(error)
    //     })
    // }
  },
  // watch: {
  //   movie: {
  //     handler: function() {
  //     axios.get(`http://127.0.0.1:8000/api/v1/movie/${this.movie.code}/`)
  //       .then(response => {
  //           console.log('??????????????', response)
  //           this.movie = response.data
  //       })
  //       .catch(error => {
  //           console.log(error)
  //       })
  //     }
  //   }
  // }
}
</script>

<style>
.modal-backdrop {
  z-index: -1 !important;
}
</style>
