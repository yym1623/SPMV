<script>
import axios from 'axios';
import Skeleton from '../components/Skeleton.vue';

export default {
  components: {
    Skeleton
  },
  data() {
    return {
      SEARCH_URL : 'https://api.themoviedb.org/3/search/multi',
      // 개봉예정 URL
      SEARCH_MOVIE_URL : 'https://api.themoviedb.org/3/movie/popular', 
      SEARCH_TV_URL : 'https://api.themoviedb.org/3/tv/popular',

      // 관련 영화 및 티비
      SEARCH_INFO_URL : 'https://api.themoviedb.org/3/movie',

      // KEY
      API_KEY : 'e95aab0b32ea685f4064a7364dec77f4',
      
      // IMG_URL
      SEARCHR_IMG : 'https://image.tmdb.org/t/p/original',

      // DATA
      SEARCH_TITLE : this.$route.params.title,

      SEARCH_FILE : [],

      CHECK_DATA : "",

      skeleton : true,

      total_page  : Number(this.$route.params.total_page),
      search_page : 2,

    }
  },
  methods: {
    // detail page
    detail(id) {
      this.$router.push({
        name: "DetailInfo",
        params: {
          id,
          CHECK_DATA : this.CHECK_DATA
        }
      })
    },
  },
  watch: {
    // router data감시할땐 문자열로 감싸면서 함수로 만들어야 작동한다 -> 해당 라우터 데이터가 들어있는 변수로 함수선언할땐 왜 작동안하나?
    async "$route.params.title"(value) {
      await axios.get(`${this.SEARCH_URL}?api_key=${this.API_KEY}&language=ko&query=${value}`)
      .then((res) => {
        this.SEARCH_TITLE = value
        this.SEARCH_FILE = res.data.results;
        this.CHECK_DATA = res.data.results[0].media_type;
      }).catch((error) => {
        console.log(error)
        this.SEARCH_TITLE = "";
      })
    }
  },
  updated() {
    // console.log(this.total_page)
    const io = new IntersectionObserver((entries, observer) => {
      entries.forEach(entry => {
        if (entry.isIntersecting) { // 감지대상이 교차영역에 진입 할 경우
          // axios
          if(this.total_page >= this.search_page) {
            axios.get(`${this.SEARCH_URL}?api_key=${this.API_KEY}&language=ko&query=${this.SEARCH_TITLE}&page=${this.search_page}`)
            .then((res) => {
              // console.log(res);
              this.SEARCH_FILE = this.SEARCH_FILE.concat(res.data.results)
            }).catch((error) => {
              console.log(error)
            })
            this.search_page++;
            observer.unobserve(entry.target); // 이미지 로딩 이후론 관찰할 필요 x
          }
        }
      })
    })
    const search = document.querySelector('.search__item__plus');
    io.observe(search)
  },
  async mounted() {
    if(this.SEARCH_TITLE === '') {
      this.skeleton = false;
      return
    } else {
      await axios.get(`${this.SEARCH_URL}?api_key=${this.API_KEY}&language=ko&query=${this.SEARCH_TITLE}`)
        .then((res) => {
          console.log(res)
          this.SEARCH_FILE = res.data.results;
          if(res.data.results.length === 0) {
            return;
          } else {
            this.CHECK_DATA = res.data.results[0].media_type;
          }
          this.skeleton = false;
        }).catch((error) => {
          console.log(error)
        })
    }
  }
}
</script>


<template>
  <div class="search">
    <div class="search__list">
      <Skeleton v-if="skeleton" />
      <div class="search__item" v-else-if="this.CHECK_DATA !== ''">
        <div class="__item" v-for="item in SEARCH_FILE" :key="item" @click="detail(item.id)">
          <img :src="`${SEARCHR_IMG}/${item.poster_path}`" onerror="this.src='https://previews.123rf.com/images/julynx/julynx1408/julynx140800023/30746516-%EC%82%AC%EC%9A%A9%ED%95%A0-%EC%88%98-%EC%97%86%EA%B1%B0%EB%82%98-%EC%9D%B4%EB%AF%B8%EC%A7%80-%EC%82%AC%EC%A7%84-%EC%97%86%EC%9D%8C.jpg'"  />
        </div>
      </div>
      <div class="no__item" v-else>검색된 결과가 없습니다</div>
    </div>
    <div class="search__item__plus"></div>
  </div>
</template>

<style lang="scss" scoped>
  .search {
    padding-top: 4.1rem;
    // height: 100vh;
    background: #060d17;
    min-height: 100vh;
    .search__list {
      padding-top: 10vh;
      width: 70vw;
      margin: auto;
      .__header {
        height: 10vh;
        display: flex;
        align-items: center;
        color: #fff;
        font-size: 22px;
        .__title {
          display: flex;
          .__movie {
            margin-left: 1rem;
            transition: .2s;
            cursor: pointer;
            &:hover {
              transform: scale(1.1);
              transition: .2s;
            }
          }
          .__tv {
            margin-left: 5rem;
            transition: .2s;
            cursor: pointer;
            &:hover {
              transform: scale(1.1);
              transition: .2s;
            }
          }
        }
      }
      .search__item {
        display: grid;
        grid-gap: 10px;
        grid-template-columns: repeat(6, 1fr);
        justify-items: center;
        .__item {
          .__skeleton {
            width: 11vw;
            height: 14vw;
            border-radius: 10px;
            background-color: #ddd;
            animation: pulse-bg 1s infinite;
            @keyframes pulse-bg {
              0% {
                background-color: #ddd;
              }
              50% {
                background-color: #d0d0d0;
              }
              100% {
                background-color: #ddd;
              }
            }
          }
          img {
            width: 11vw;
            height: 14vw;
            border-radius: 10px;
            transition: .2s;
            &:hover {
              cursor: pointer;
              transform: scale(1.1);
              transition: .2s;
            }
          }
        }
      }
      .no__item {
        display: flex;
        justify-content: center;
        align-items: center;
        font-size: 35px;
        color: #fff;
        // text-align: center;
        border: 1px solid #fff;
        border-radius: 10px;
        height: 20vh;
      }
    }
  }

  @media screen and (max-width: 1024px) {
    .search {
      padding-top: 4.1rem;
      // height: 100vh;
      background: #060d17;
      min-height: 100vh;
      .search__list {
        padding-top: 10vh;
        width: 70vw;
        margin: auto;
        .__header {
          height: 10vh;
          display: flex;
          align-items: center;
          color: #fff;
          font-size: 22px;
          .__title {
            display: flex;
            .__movie {
              font-size: 20px;
              margin-left: 1rem;
              transition: .2s;
              cursor: pointer;
              &:hover {
                transform: scale(1.1);
                transition: .2s;
              }
            }
            .__tv {
              font-size: 20px;
              margin-left: 5rem;
              transition: .2s;
              cursor: pointer;
              &:hover {
                transform: scale(1.1);
                transition: .2s;
              }
            }
          }
        }
        .search__item {
          display: grid;
          grid-gap: 10px;
          grid-template-columns: repeat(4, 1fr);
          justify-items: center;
          .__item {
            .__skeleton {
              width: 17vw;
              height: 20vw;
              border-radius: 10px;
              background-color: #ddd;
              animation: pulse-bg 1s infinite;
              @keyframes pulse-bg {
                0% {
                  background-color: #ddd;
                }
                50% {
                  background-color: #d0d0d0;
                }
                100% {
                  background-color: #ddd;
                }
              }
            }
            img {
              width: 17vw;
              height: 20vw;
              border-radius: 10px;
              transition: .2s;
              &:hover {
                cursor: pointer;
                transform: scale(1.1);
                transition: .2s;
              }
            }
          }
        }
      }
    }
  }
  
  @media screen and (max-width: 768px) {
    .search {
      padding-top: 2.9rem !important;
      .search__list {
        width: 90vw;
        margin: auto;
        .search__item {
          display: grid;
          grid-gap: 10px;
          grid-template-columns: repeat(3, 1fr);
          justify-items: center;
          .__item {
            .__skeleton {
              width: 28vw !important;
              height: 25vh !important;
            }
            img {
              width: 28vw !important;
              height: 25vh !important;
              border-radius: 10px;
              transition: .2s;
              &:hover {
                cursor: pointer;
                transform: scale(1.1);
                transition: .2s;
              }
            }
          }
        }
      }
    }
  }
</style>