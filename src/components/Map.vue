<template>
    <div class="map" ref="googleMap" />
</template>

<script>
import GoogleMapsApiLoader from 'google-maps-api-loader'
import axios from 'axios'

export default {
    data () {
        return {
            datas: [],
            google: null,
        }
    },
    async mounted () {
        this.google = await GoogleMapsApiLoader({
            // google-maps-api-loaderのAPIキー
            apiKey: '*****************'
        })

        // 公衆無線LANアクセスポイント一覧取得APIのURL 取得件数はlimitパラメータで指定
        const url = 'https://api.data.metro.tokyo.lg.jp/v1/WifiAccessPoint?limit=100'
        // 公衆無線LANアクセスポイント一覧を取得
        await axios.get(url).then(response => {
            this.datas = response.data[0]
        })

        this.initializeMap()
    },
    methods: {
        initializeMap () {
          // 現在位置の取得に成功した場合の処理
          const success = (pos) => {
              const mapContainer = this.$refs.googleMap
              //現在位置の緯度と経度を取得
              const latlng = new this.google.maps.LatLng(pos.coords.latitude, pos.coords.longitude)
              // 現在位置を基準に初期表示位置の設定
              const mapConfig = {center: latlng, zoom: 12}

              // GoogleMapの描画
              this.map = new this.google.maps.Map(
                  mapContainer,
                  mapConfig
              )
              // 現在位置にマーカーを表示
              const marker = new this.google.maps.Marker( {
                  map: this.map ,
                  position: latlng ,
                  animation: this.google.maps.Animation.DROP,
                  icon: {
                      fillColor: "#FF0000", //塗り潰し色
                      fillOpacity: 0.8, //塗り潰し透過率
                      path: this.google.maps.SymbolPath.CIRCLE,//円を指定
                      scale: 50,  //円のサイズ
                      strokeColor: "#FF0000", //枠の色
                      strokeWeight: 1.0 //枠の透過率
                  },
                  label: {
                      text: 'イマココ', //ラベル文字
                      color: '#FFFFFF', //文字の色
                      fontSize: '26px'  //文字のサイズ
                  }
                })
                console.log(marker)

                // 取得し公衆無線LANアクセスポイント一覧をループし各地にマーカーを表示
                const marker2 = []
                this.datas.forEach((item, i) => {
                    marker2[i + 1] = new this.google.maps.Marker( {
                        map: this.map ,
                        position: new this.google.maps.LatLng(item.設置地点.地理座標.緯度, item.設置地点.地理座標.経度) ,
                        animation: this.google.maps.Animation.DROP,
                    })
                })
              console.log(marker2)
          }
          //現在位置の取得に失敗した場合の処理
          const fail = () => {
              alert('位置情報の取得に失敗しました。')
          }

          // 現在位置を取得する処理の動作オプション
          const option = {
             enableHighAccuracy: true,  // より精度の高い位置情報を取得するか（true／false）
             timeout : 5000,  // 取得タイムアウトまでの時間（ミリ秒）
             maximumAge: 0, // 位置情報の有効期限（ミリ秒）0に指定すると常に新しい情報に更新しようとする。
           }

          // 現在位置を取得
          navigator.geolocation.getCurrentPosition(success,fail,option)
        },
    }
}
</script>
<style>
.map {
  padding-bottom: 75%;
}
</style>
