<template>
  <v-row
    v-if="false"
    style="height: 50vh;"
    class="fill-height ma-0"
    align="center"
    justify="center"
  >
    <v-progress-circular
      indeterminate
      color="primary"
    ></v-progress-circular>
  </v-row>
  <v-row
    v-else
    justify="center"
  >
    <v-col
      class="scroll-container"
      lg="4"
      md="4"
      sm="6"
    >
      <div
        class="pa-3"
      >
        <p class="text-h6 mb-1">이번주 ...에서는</p>
        <p class="text-caption mb-2">현재 위치를 기반으로 인기가 많은 게시글을 추천해드립니다.</p>
      </div>
      <v-card
        class="mb-3"
      >
        <v-card-actions
          class="pa-0"
        >
          <div
            id="map"
            class="map"
            width="100"
            
          >
          </div>
        </v-card-actions>
      </v-card>
      
      <v-divider></v-divider>

      <div
        class="pa-3"
      >
        <p class="text-h6 mb-1">...님이 관심있을 법한 장소</p>
        <p class="text-caption mb-2">...님께서 좋아하신 장소들을 기반해서 추천해드립니다.</p>
        <v-card>
          <v-card-actions
            v-if="recommandsByLike.length>0"
            class="pa-0"
          >
            <!-- 위치기반 인기 게시글 캐러셀 시작 -->
            <v-carousel
              :show-arrows="false"
              hide-delimiter-background
              :mouse-drag="true"
              height="80%"
            >
              <v-carousel-item
                v-for="(recommand, index) in recommandsByLike"
                :key="index"
              >
                <v-img
                  :src="`https://picsum.photos/300/300?image=${index * 5 + 10}`"
                  :lazy-src="`https://picsum.photos/300/300?image=${index * 5 + 10}`"
                  aspect-ratio="1"
                  class="grey lighten-2 image-hover"
                >
                  <!-- 이미지 요청이 길어지면 뜨는 로딩 창 시작 -->
                  <template v-slot:placeholder>
                    <v-row
                      class="fill-height ma-0"
                      align="center"
                      justify="center"
                    >
                      <v-progress-circular
                        indeterminate
                        color="grey lighten-5"
                      ></v-progress-circular>
                    </v-row>
                  </template>
                  <!-- 이미지 요청이 길어지면 뜨는 로딩 창 끝 -->

                  <!-- 이미지에 위치 정보를 나타내기 위한 트랜지션 시작 -->
                  <!-- <v-expand-transition>
                    <div
                      v-if="hover"
                      class="d-flex black lighten-2 reveal white--text text-center"
                      style="height: 100%; font-size: 1.2rem;"
                    >
                      <span style="white-space: pre-line">{{ article.pin.addressName | truncate(20, '...') }}</span>
                    </div>
                  </v-expand-transition> -->
                  <!-- 이미지에 위치 정보를 나타내기 위한 트랜지션 끝 -->
                </v-img>
              </v-carousel-item>
            </v-carousel>
            <!-- 위치기반 인기 게시글 캐러셀 끝 -->

          </v-card-actions>
          <v-card-text class="text-subtitle-1" v-else>관심있는 게시글에 좋아요를 눌러주세요!<br>회원님이 관심있을 법한 장소를 추천해드립니다!</v-card-text>
        </v-card>
      </div>

    </v-col>
  </v-row>
</template>

<script>
import axios from 'axios'

const SERVER_URL = process.env.VUE_APP_SERVER_URL

export default {
  name: 'Curation',
  data() {
    return {
      loading: true,
      recommandsByLocation: [
        {test: 'content1'}, {test: 'content2'}, {test: 'content3'}, {test: 'content4'}, {test: 'content5'},
      ],
      recommandsByLike: [],
      map: null,
      lat: 0,
      lon: 0,
    }
  },
  computed: {
    getToken(){
      const token = sessionStorage.getItem('jwt')
      const config = {
        headers: {
          'X-Authorization-Firebase': token
        }
      }
      return config
    }
  },
  created() {
    this.dataFetch()
  },
  mounted() {
    // window.kakao && window.kakao.maps? this.initMap() : this.addKakaoMapScript();
        window.kakao && window.kakao.maps
      ? this.initMap()
      : this.addKakaoMapScript();
  },
  methods: {
    dataFetch() {
      axios.get(`${SERVER_URL}/recommendations/articles`, this.getToken)
      .then(res => {
        this.recommandsByLike = res.data
        console.log(this.recommandsByLike)
      })
      .then(() => {
        this.loading = false
      })
      .catch(err => {
        alert(err)
      })
    },
    addKakaoMapScript() {
      const script = document.createElement("script");
      /* global kakao */
      script.onload = () => kakao.maps.load(this.initMap)
      script.src =
        "http://dapi.kakao.com/v2/maps/sdk.js?autoload=false&appkey=41dd8e1c2fab039d8dbbff2e13e8d5a5";
      document.head.appendChild(script);
    },
    initMap() {
      const self = this
      
      // 사용자의 현재 위치를 기반으로 맵 보여준다.
      if (navigator.geolocation) {
        // GeoLocation을 이용해서 접속 위치를 얻어옵니다
        const success = function (pos) {
          console.log(pos.coords.latitude, pos.coords.longitude)
          const requestOptions = { 
            center: new kakao.maps.LatLng(pos.coords.latitude, pos.coords.longitude),
            level: 4
          }
          const container = document.getElementById("map")
          self.map = new kakao.maps.Map(container, requestOptions)
        }

        const error = function (err) {
          console.warn('ERROR(' + err.code + '): ' + err.message);
        }

        navigator.geolocation.getCurrentPosition(success, error, {
          enableHighAccuracy: true,
          timeout: 5000,
          maximumAge: 0
        })
      } else { // HTML5의 GeoLocation을 사용할 수 없을때 마커 표시 위치와 인포윈도우 내용을 설정합니다
        const options = { 
          center: new kakao.maps.LatLng(36.3551966493045, 127.298384348217),
          level: 4
        }
        const container = document.getElementById("map")
        self.map = new kakao.maps.Map(container, options)
      }
      
      this.createArticleMarker()
    },
    createArticleMarker() {
      // 다음 코드를 보고 참조하여 마커를 찍는 코드를 작성하자!
      // var imageSrc = "https://t1.daumcdn.net/localimg/localimages/07/mapapidoc/markerStar.png"; 
    
      // for (var i = 0; i < positions.length; i ++) {
          
      //   // 마커 이미지의 이미지 크기 입니다
      //   var imageSize = new kakao.maps.Size(24, 35); 
        
      //   // 마커 이미지를 생성합니다    
      //   var markerImage = new kakao.maps.MarkerImage(imageSrc, imageSize); 
        
      //   // 마커를 생성합니다
      //   var marker = new kakao.maps.Marker({
      //       map: map, // 마커를 표시할 지도
      //       position: positions[i].latlng, // 마커를 표시할 위치
      //       title : positions[i].title, // 마커의 타이틀, 마커에 마우스를 올리면 타이틀이 표시됩니다
      //       image : markerImage // 마커 이미지 
      //   });

      //   kakao.maps.event.addListener(marker, 'click', makeOverListener(map, marker, onArticleMarker(item.articleId))); 
      // } // for loop 끝
      // 
    },
    onArticleMarker(articleId) {
    },
  },
}
</script>

<style scoped>
/* 스크롤 컨테이너 안의 아이템이 넘쳐도 스크롤 컨테이너의 크기는 고정 */
  .scroll-container {
    width: 100%;
    overflow: hidden;
    margin-bottom: 50px;
    padding: 0;
  }
</style>