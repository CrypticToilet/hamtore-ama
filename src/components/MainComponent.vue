<template>
  <div>
    <v-layout row wrap>
      <!--Functions-->
      <v-flex lg6 class="my-4">
        <v-card class="white ma-2" height="100%">
          <v-flex dark flat class="teal pa-1">
            <v-icon color="white">play_circle_filled</v-icon>
            <span class="title white--text px-2">Functions</span>
            <span class="caption mx-2 white--text">機能</span>
          </v-flex>

          <v-btn
            flat
            @click="getStatus(radio_broker)"
            href="#"
            class="teal white--text"
            :disabled="this.error_config.length != 0"
            :loading="loading"
          >
            <v-icon>play_circle_filled</v-icon>
            <span class="px-2">ポジションと合計値の調査</span>
          </v-btn>

          <v-btn
            flat
            @click="get1970record"
            href="#"
            class="teal white--text"
            :loading="loading"
            :disabled="this.error_config.length != 0"
          >
            <v-icon>play_circle_filled</v-icon>
            <span class="px-2">エラーレコードの調査</span>
          </v-btn>

          <v-progress-linear v-show="loading" :indeterminate="loading"></v-progress-linear>

          <v-radio-group v-model="radio_broker" :mandatory="true" class="mx-3 pa-0 my-1">
            <template v-slot:label>
              <div>
                <strong>取引所</strong>
              </div>
            </template>
            <v-radio
              label="bitFlyer"
              value="bitflyer"
              v-show="this.fusion_tables_id_status_bf != undefined && this.fusion_tables_id_status_bf != '' "
            ></v-radio>
            <v-radio
              label="BitMEX"
              value="bitmex"
              v-show="this.fusion_tables_id_status_bitmex != undefined && this.fusion_tables_id_status_bitmex != '' "
            ></v-radio>
          </v-radio-group>
          <v-flex lg6>
            <v-text-field
              class="mx-2 my-0 pa-0"
              v-model="v_table_search_ls"
              append-icon="search"
              label="L/S Search"
              single-line
              hide-details
            ></v-text-field>
          </v-flex>
        </v-card>
      </v-flex>

      <!--
        <v-btn flat @click="fileLoadTest()" href="#" class="teal white--text">
          <v-icon>play_circle_filled</v-icon>
          <span class="px-2">file load test</span>
        </v-btn>
      -->

      <!--Summary-->
      <v-flex lg6 class="my-4">
        <v-card class="white ma-2" height="100%">
          <v-flex dark flat class="teal pa-1">
            <v-icon color="white">assignment</v-icon>
            <span class="title white--text px-2">Summary</span>
            <span class="caption mx-2 white--text">サマリ</span>
          </v-flex>
          <div class="ma-2">
            <h1
              class="red--text"
              v-show="isErrorLong || isErrorShort || isErrorSquare || error_records.length != 0"
            >問題が発見されました！</h1>
            <h2>現在のポジション合計：{{ total_volume_in_table }} ( 最大値:{{ status_max_volume }} )</h2>
            <span class="caption grey--text">*最大値はすべてのストラテジが同じ方向にエントリーした場合の合計値</span>
            <!--config.json エラーリスト-->
            <h4 v-if="this.error_config.length != 0" class="mt-2 yellow lighten">
              <v-badge color="red">
                <template v-slot:badge>
                  <span>!</span>
                </template>
                システムエラーがありました。config.jsonを確認してください。
              </v-badge>
            </h4>
            <ul>
              <li
                class="caption red--text"
                v-for="(r, index) in this.error_config"
                :key="index"
              >{{ r }}</li>
            </ul>

            <ul>
              <li
                class="caption grey--text"
                v-for="(r, index) in this.info_config"
                :key="index"
              >{{ r }}</li>
              <li
                class="caption grey--text"
                v-if="isErrorLong || isErrorShort || isErrorSquare"
              >L/Sにエラーが出ている場合まずdoten_listに正しくストラテジが追加されているかご確認ください。doten_listが無いと正しくポジション計算ができません。</li>
              <li
                class="caption teal--text"
                v-if="isErrorLong || isErrorShort || isErrorSquare || error_records.length > 0"
              >1970年のエラーレコードが出ている時は不整合となっている可能性がありますが実際のポジションは正しくHistory上が間違っているパターンが多いです。修正などは慎重に行う必要があります。分からない場合はコミュニティで相談しましょう。</li>
            </ul>
          </div>
        </v-card>
      </v-flex>

      <!-- L/S -->
      <!-- ロング -->
      <v-flex lg4 class="my-4">
        <v-card class="white ma-2" height="100%">
          <v-flex dark flat class="blue lighten-1 pa-1">
            <v-badge left overlap color="red" v-model="isErrorLong">
              <template v-slot:badge>
                <span>!</span>
              </template>
            </v-badge>
            <span class="white--text ma-4">Long</span>
          </v-flex>

          <v-data-table
            :headers="v_table_headers_ls"
            :items="ls_table_long"
            :pagination.sync="pagination"
            :search="v_table_search_ls"
            class="elevation-1"
          >
            <template v-slot:items="props">
              <td>{{ props.item['name'] }}</td>
              <td>{{ props.item['volume'] }}</td>
              <td>
                {{ props.item['isValid'] }}
                <v-badge color="red" v-show="props.item['isValid'] === 'ERROR'">
                  <template v-slot:badge>
                    <span>!</span>
                  </template>
                </v-badge>
              </td>
            </template>
          </v-data-table>
        </v-card>
      </v-flex>

      <!-- ショート -->
      <v-flex lg4 class="my-4">
        <v-card class="white ma-2" height="100%">
          <v-flex dark flat class="red lighten-1 pa-1">
            <v-badge left overlap color="red darken-4" v-model="isErrorShort">
              <template v-slot:badge>
                <span>!</span>
              </template>
            </v-badge>
            <span class="white--text ma-4">Short</span>
          </v-flex>

          <v-data-table
            :pagination.sync="pagination"
            :headers="v_table_headers_ls"
            :items="ls_table_short"
            :search="v_table_search_ls"
            class="elevation-1"
          >
            <template v-slot:items="props">
              <td>{{ props.item['name'] }}</td>
              <td>{{ props.item['volume'] }}</td>
              <td>
                {{ props.item['isValid'] }}
                <v-badge color="red" v-show="props.item['isValid'] === 'ERROR'">
                  <template v-slot:badge>
                    <span>!</span>
                  </template>
                </v-badge>
              </td>
            </template>
          </v-data-table>
        </v-card>
      </v-flex>

      <!-- ノーポジ -->
      <v-flex lg4 class="my-4">
        <v-card class="white ma-2" height="100%">
          <v-flex dark flat class="grey pa-1">
            <v-badge left overlap color="red" v-model="isErrorSquare">
              <template v-slot:badge>
                <span>!</span>
              </template>
            </v-badge>
            <span class="white--text ma-4">Square</span>
          </v-flex>

          <v-data-table
            :pagination.sync="pagination"
            :headers="v_table_headers_ls"
            :items="ls_table_square"
            :search="v_table_search_ls"
            class="elevation-1"
          >
            <template v-slot:items="props">
              <td>{{ props.item['name'] }}</td>
              <td>{{ props.item['volume'] }}</td>
              <td>
                {{ props.item['isValid'] }}
                <v-badge color="red" v-show="props.item['isValid'] === 'ERROR'">
                  <template v-slot:badge>
                    <span>!</span>
                  </template>
                </v-badge>
              </td>
            </template>
          </v-data-table>
        </v-card>
      </v-flex>

      <!-- Record Errors -->
      <v-flex xs12 class="my-4">
        <v-card class="white ma-2">
          <v-flex dark flat class="teal pa-1">
            <v-icon color="white">error_outline</v-icon>
            <span class="white--text px-2">Record Errors</span>
          </v-flex>
          <h4 class="pa-2" v-show="error_records.length === 0">{{ error_records_comment}}</h4>
          <!-- レコードが無いとき表示 -->
          <v-list two-line>
            <template v-for="(error_record, index) in this.error_records">
              <v-list-tile :key="error_record.e[0]" avatar ripple>
                <v-icon left large color="red">error</v-icon>
                <v-list-tile-content>
                  <v-list-tile-title>ストラテジ： {{ error_record.e[1] }} にエラー(ROWID:{{error_record.e[0]}})</v-list-tile-title>
                  <v-list-tile-sub-title
                    class="text--primary"
                  >{{ error_record.prev[0] }} ～ {{ error_record.next[0] }} の間にエラーレコード(1970年問題)が発見されました。</v-list-tile-sub-title>
                  <v-list-tile-sub-title>前後のストラテジ名 {{ error_record.prev[1] }} ～ {{ error_record.next[1] }}</v-list-tile-sub-title>
                </v-list-tile-content>
              </v-list-tile>
              <v-divider v-if="index + 1 < error_records.length" :key="index"></v-divider>
            </template>
          </v-list>
        </v-card>
      </v-flex>

      <!-- Stats -->
      <v-flex xs12 class="my-4">
        <v-card class="white ma-2">
          <v-flex dark flat class="teal pa-1">
            <v-icon color="white">description</v-icon>
            <span class="white--text px-2">Status</span>
          </v-flex>

          <v-layout row>
            <!-- 保存機能は一時的に制限：今表示しているbFかMEXのみのDLとなるため上書きするとどちらかのデータが消えてしまう恐れ
            <v-btn @click="downloadDotenTable()" href="#" class="teal white--text">
              <v-icon color="white">save</v-icon>
              <span class="px-2">ドテンテーブルを保存</span>
            </v-btn>
            -->
            <v-spacer></v-spacer>
            <v-flex xs6>
              <v-text-field
                class="pa-2"
                v-model="v_table_search"
                append-icon="search"
                label="Search"
                single-line
                hide-details
              ></v-text-field>
            </v-flex>
          </v-layout>

          <v-data-table
            :headers="v_table_headers"
            :items="this.status['data']['rows']"
            class="elevation-1"
            :search="v_table_search"
          >
            <template v-slot:items="props">
              <v-container>
                <v-checkbox
                  :input-value="props.item[8]"
                  v-model="props.item[8]"
                  primary
                  hide-details
                ></v-checkbox>
              </v-container>
              <td>{{ props.item[0] }}</td>
              <td>{{ props.item[1] }}</td>
              <td>{{ props.item[2] }}</td>
              <td>{{ props.item[3] }}</td>
              <td>{{ props.item[4] }}</td>
              <td>{{ props.item[5] }}</td>
              <td>{{ props.item[6] }}</td>
              <td>{{ props.item[7] }}</td>
            </template>
          </v-data-table>
        </v-card>
      </v-flex>

      <!-- スナックバー -->
      <v-snackbar class="ma-2" :top="true" v-model="snack" :timeout="timeout">
        <v-icon v-if="icon === 1" color="green" class="ma-2">check</v-icon>
        <v-icon v-if="icon === 2" color="red" class="ma-2">error</v-icon>
        {{ snack_text }}
        <v-btn color="blue" text @click="snackbar = false">Close</v-btn>
      </v-snackbar>

      <!-- ダイアログ -->
      <v-layout justify-center>
        <v-dialog v-model="dialog" max-width="600">
          <v-card>
            <v-card-title class="headline">
              <h4>
                <v-layout align-center>
                  <v-icon class="mx-1" color="blue">help</v-icon>ドテンリスト(doten_list.json)を作成しますか？
                </v-layout>
              </h4>
            </v-card-title>

            <v-card-text>
              ドテン式(ノーポジションが存在しない)ストラテジ一覧ファイルです。
              <br />現在のポジションボリュームが正常かどうかの計算に必要です。
              <v-card-text class="grey--text">
                不要な場合は
                <a href="#" @click="downloadDotenTableDummy()">空のファイル</a>をconfigフォルダに配置してください
              </v-card-text>
            </v-card-text>

            <v-card-actions class="pa-4">
              <v-spacer></v-spacer>

              <v-btn color="success" text @click="dialog = false;toDotenListMaker()">作成画面へ</v-btn>
              <v-btn color="error" text @click="dialog = false">キャンセル</v-btn>
            </v-card-actions>
          </v-card>
        </v-dialog>
      </v-layout>

      <!-- DBG 
      <v-flex lg12 class="my-4">
        <v-card class="white ma-2" height="100%">
          <v-flex dark flat class="teal pa-1">
            <v-icon color="white">assignment</v-icon>
            <span class="title white--text px-2">DEBUG</span>

            <span class="caption mx-2 white--text">デバッグ</span>
          </v-flex>

          <v-btn flat @click=" calculateTotal()" href="#" class="teal white--text">
            <span>DBG:合計を再計算</span>
          </v-btn>
          <v-btn flat @click="save_local" href="#" class="teal white--text">
            <span>DBG:ローカルにセーブ</span>
          </v-btn>
          <v-btn flat @click="load_local" href="#" class="teal white--text">
            <span>DBG:ローカルからロード</span>
          </v-btn>
        </v-card>
      </v-flex>
      -->
    </v-layout>
  </div>
</template>

<script>
//import doten_list from "../assets/doten_list.json"; //public/conifg/内から読み込むように変更
//import user_config from "../assets/config.json"; //テーブルのidやapikeyなどは親から受け取ります

export default {
  name: "MainComponent",
  props: ["passconfig"], //user_configの親から子へ受け渡し用。受け渡しの制約で"_"が使用できないため別の名前を指定している
  data() {
    return {
      //statsuテーブルを格納
      status: {
        data: {
          rows: []
        }
      },
      snack: false, //スナックバー用
      snack_text: "", //スナックバー用
      timeout: 3000, //スナックバー用
      icon: 1, //スナックバー用
      user_config: {},
      radio_broker: "", //取引所選択(デフォルトとしても使用できるのでプログラム内で記述)
      loading: false, //ローディング用
      status_length: 0, //statusの要素数
      status_max_volume: 0, //statusでONになっているストラテジのトータルvolume(MAX)
      doten_list: [],
      dialog: false, //ドテンリスト作成ダイアログ用
      //エラーのレコード 現在は1970エラーのみ
      error_records: [],
      error_records_comment: "No Data", //Record Errorsに表示されるコメント
      error_config: [], //サマリに表示するエラー 設定データ不足
      info_config: [], //エラーではないがコンフィグに不足がある情報
      isErrorLong: false, //LSテーブルの中にエラーコードがあった場合バッジを表示する
      isErrorShort: false,
      isErrorSquare: false,
      //LSを管理
      ls_table_long: [],
      ls_table_short: [],
      ls_table_square: [],
      total_volume_in_table: 0, //historyテーブル上のvolumeを合計した値
      pagination: {
        sortBy: "isValid",
        descending: false
      },
      //-- ユーザーconfig  用
      fusion_tables_id_history: "", //history
      fusion_tables_id_status_bf: "", //stats bF
      fusion_tables_id_status_bitmex: "", //stats mex
      api_key: "",
      api_url: "",
      //-- ユーザーconfig終わり
      config: {
        //google側にアクセスするときのヘッダ用コンフィグ
        headers: {
          Accept: "application/json"
        }
      },
      v_table_search: "", //status検索用
      v_table_search_ls: "", //ポジションテーブル検索用
      v_table_headers: [
        { text: "Doten", sortable: false },
        { text: "Strategy Name", sortable: true, value: "0" },
        { text: "Active", value: "1" },
        { text: "Product Code", value: "2" },
        { text: "Volume", value: "3", sortable: true },
        { text: "Time", value: "4", sortable: true },
        { text: "ToDoubles", value: "5" },
        { text: "Exchange", value: "6" },
        { text: "Order Type", value: "7" }
      ],
      v_table_headers_ls: [
        { text: "Strategy Name", sortable: true, value: "name" },
        { text: "Volume", sortable: true, value: "volume" },
        { text: "Status", sortable: true, value: "isValid" }
      ]
    };
  },
  async created() {
    //config.jsonエラーチェック
    console.log("- Checking config.json -");

    if (this.passconfig.encrypted_config != "") {
      //まだ暗号化解除されていないのでテーブルを読み込める状態ではないためログインへ遷移する
      this.$emit("needLogin");
    }

    this.user_config = this.passconfig; //親から渡ってきたデータの名前を変更(アンダーバーは認識しないのでpassconfigとして受け渡し)
    if (!this.user_config.fusion_tables_id_history) {
      this.error_config.push("Error: fusion_tables_id_history is not defined");
    } else {
      this.fusion_tables_id_history = this.user_config.fusion_tables_id_history;
    }

    if (!this.user_config.fusion_tables_id_status_bf) {
      this.info_config.push(
        "info: bitFlyerは設定が空欄のため利用できません( fusion_tables_id_status_bf is not defined. )"
      );
    } else {
      this.fusion_tables_id_status_bf = this.user_config.fusion_tables_id_status_bf;
    }

    if (!this.user_config.fusion_tables_id_status_bitmex) {
      this.info_config.push(
        "info: BitMEXは設定が空欄のため利用できません( fusion_tables_id_status_bitmex is not defined. )"
      );
    } else {
      this.fusion_tables_id_status_bitmex = this.user_config.fusion_tables_id_status_bitmex;
    }

    if (!this.user_config.api_key) {
      this.error_config.push("Error: api_key is not defined");
    } else {
      this.api_key = this.user_config.api_key;
    }

    if (!this.user_config.api_url) {
      this.error_config.push("Error: api_url is not defined");
    } else {
      this.api_url = this.user_config.api_url;
    }

    if (this.error_config.length === 0) {
      console.log("Load OK: config.json");
    } else {
      this.error_config.push("Something went wrong!: config.json");
      console.log("Something went wrong!: config.json");
    }

    //取引所選択のラジオボタン初期設定。bfかbitmexどちらかconfigに設定なしの場合があるのでここの処理にて初期値を設定しておく
    this.radio_broker = this.user_config.fusion_tables_id_status_bf
      ? "bitflyer"
      : this.user_config.fusion_tables_id_status_bitmex
      ? "bitmex"
      : null;
    if (this.radio_broker === null) {
      console.log(
        "取引所ラジオボタン初期化中にエラー： 設定ファイルが存在しないか、取引所が設定ファイルにない"
      );
    }

    //doten_list.jsonの読み込み
    await this.$axios
      .get("config/doten_list.json")
      .then(response => {
        this.doten_list = response.data;
      })
      .catch(e => {
        this.info_config.push(
          "info: ドテンリスト(doten_list.json)がありませんでした。エラー判定が正常にできない可能性があります。ドテン式のストラテジを稼働している場合リストを作成してください。"
        );
        this.dialog = true; //ドテンリストを作成しますか？のダイアログ
      });
  },
  methods: {
    //ファイルロードテスト
    //vue.jsの場合ローカルからファイルをaxiosにて呼び出す場合はpublic配下に配置する
    async fileLoadTest() {
      const file_url = "/test.json";
      this.$axios
        .get(file_url, this.config)
        .then(response => {
          console.log(response.data);
        })
        .catch(reason => {
          console.log("fileLoadTest error :" + reason);
        });
    },
    //statusテーブルを取得
    async getStatus(broker) {
      this.loading = true;

      //リセット

      this.ls_table_long = [];
      this.ls_table_short = [];
      this.ls_table_square = [];
      this.total_volume_in_table = 0;
      this.status_length = 0;
      this.isErrorLong = false;
      this.isErrorShort = false;
      this.isErrorSquare = false;
      this.status_max_volume = 0;

      const fusion_tables_id_status =
        broker === "bitflyer"
          ? this.fusion_tables_id_status_bf
          : broker === "bitmex"
          ? this.fusion_tables_id_status_bitmex
          : undefined;
      const sql =
        "SELECT * FROM " +
        fusion_tables_id_status +
        " WHERE Exchange='" +
        broker +
        "'";
      const sql_act = sql + "&key=" + this.api_key;
      const url = this.api_url + encodeURI(sql_act);

      console.log("statusを取得します：" + url);

      await this.$axios
        .get(url, this.config)
        .then(response => {
          this.status = response;

          const strategies = this.status["data"]["rows"];
          for (let strategy of strategies) {
            const isDoten = this.dotenCheck(strategy[0]);
            strategy[8] = isDoten;

            //statusがONの数をカウントして保存する
            if (strategy[1].toUpperCase() === "ON") {
              this.status_length++;
              //全部のストラテジが同じ方向にエントリーした場合のトータル計算（ドテン式の場合はロット数半分)
              let v = 0;
              v = isDoten
                ? this.bfKirisute(strategy[3] / 2)
                : this.bfKirisute(strategy[3]);
              console.log(v + " : " + strategy[0]);
              this.status_max_volume = this.bfKirisute(
                this.status_max_volume + v
              );
            }
          }
        })
        .catch(reason => {
          console.log("status取得に失敗しました:" + reason);
          if (reason.response.status === 400) {
            this.error_config.push(
              "status取得に失敗：config.jsonに指定したテーブルID(" +
                broker +
                ")が間違っている可能性があります。"
            );
          } else {
            this.error_config.push("status取得に失敗しました：" + reason);
          }
        });
      console.log("status取得：正常終了 - ONの数:" + this.status_length);
      this.getLS();
    },

    //1970エラーコードのサーチ
    async get1970record() {
      this.loading = true;
      this.error_records.length = 0; //error_recordsをリセット
      const sql =
        "SELECT Rowid,Strategy FROM " +
        this.fusion_tables_id_history +
        " WHERE Time LIKE '%1970%';";
      const sql_act = sql + "&key=" + this.api_key;
      const url = this.api_url + encodeURI(sql_act);
      console.log("1970レコードをサーチします:url = " + url);
      this.$axios
        .get(url, this.config)
        .then(response => {
          const strategies = response["data"]["rows"];
          if (strategies === undefined) {
            console.log("1970エラーのデータはありません");
            this.error_records_comment = "エラーコードは見つかりませんでした";
            this.loading = false;
          } else {
            for (let strategy of strategies) {
              //エラーコードが存在した場合
              console.log(
                "1970エラーコードRowid:" +
                  strategy[0] +
                  " Strategy:" +
                  strategy[1]
              );

              //対象の1個前のレコードと次のレコード番号
              const prev = Number(strategy[0]) - 1;
              const next = prev + 2;

              //注意！Fusion TableはWHEREにORが使えない
              const sql2 =
                "SELECT Time,Strategy FROM " +
                this.fusion_tables_id_history +
                " WHERE Rowid = '" +
                prev +
                "'";
              const sql_act2 = sql2 + "&key=" + this.api_key;
              const url2 = this.api_url + encodeURI(sql_act2);

              const sql3 =
                "SELECT Time,Strategy FROM " +
                this.fusion_tables_id_history +
                " WHERE Rowid = '" +
                next +
                "'";
              const sql_act3 = sql3 + "&key=" + this.api_key;
              const url3 = this.api_url + encodeURI(sql_act3);

              var prev_record;
              var next_record;

              this.$axios //前のレコードを調査
                .get(url2, this.config)
                .then(response => {
                  let time = response["data"]["rows"][0][0];
                  let name = response["data"]["rows"][0][1];
                  let prev_record = [time, name];

                  this.$axios //次のレコードを調査
                    .get(url3, this.config)
                    .then(response => {
                      let next_record;
                      if (response["data"]["rows"] != undefined) {
                        let time = response["data"]["rows"][0][0];
                        let name = response["data"]["rows"][0][1];
                        next_record = [time, name];
                      } else {
                        //1970エラーが最後のレコードだった場合responseは空になるため処理を分ける
                        next_record = ["レコードなし", "レコードなし"];
                      }
                      console.log(
                        "エラーは" +
                          prev_record[0] +
                          " から " +
                          next_record[0] +
                          " のあいだで発生しました"
                      );
                      console.log(
                        "ストラテジ名:" +
                          prev_record[1] +
                          " から " +
                          next_record[1] +
                          "のあいだで発生しました"
                      );

                      //エラーコードをpush
                      const push_rec = {
                        prev: prev_record,
                        e: [strategy[0], strategy[1]],
                        next: next_record
                      };
                      this.error_records.push(push_rec);

                      this.loading = false;
                    })
                    .catch(reason => {
                      this.error_config.push(
                        "1970レコード(次のレコード)検索中にエラー：" + reason
                      );
                      console.log(
                        "1970レコード(次のレコード)検索中にエラー:" + reason
                      );
                      this.loading = false;
                    });
                })
                .catch(reason => {
                  this.error_config.push(
                    "1970レコード(1つ前のレコード)検索中にエラー：" + reason
                  );
                  console.log(
                    "1970レコード(1つ前のレコード)検索中にエラー:" + reason
                  );
                  this.loading = false;
                });
            }
          }
        })
        .catch(reason => {
          this.error_config.push("1970レコード検索中にエラー：" + reason);
          console.log("1970レコード検索中にエラー:" + reason);
          this.loading = false;
        });
    }, //1970エラーコードのサーチここまで

    //ロング・ショートテーブルに仕分けする
    async getLS() {
      const strategies = this.status["data"]["rows"];
      for (let strategy of strategies) {
        if (strategy[1].toUpperCase() === "ON") {
          const sql =
            "SELECT Strategy,TotalVolume,Time FROM " +
            this.fusion_tables_id_history +
            " WHERE Strategy='" +
            strategy[0] +
            "' ORDER BY Time DESC LIMIT 1 ";
          const sql_act = sql + "&key=" + this.api_key;
          const url = this.api_url + encodeURI(sql_act);

          this.$axios
            .get(url, this.config)
            .then(response => {
              console.log(strategy[0] + "の現在ポジションを取得します");
              if (response["data"]["rows"] === undefined) {
                console.log(strategy[0] + "のデータはありませんでした");
              } else {
                console.log(
                  "strategy:" +
                    response["data"]["rows"][0][0] +
                    "|Volume:" +
                    response["data"]["rows"][0][1] +
                    "|Time:" +
                    response["data"]["rows"][0][2]
                );

                var push_data = {
                  name: response["data"]["rows"][0][0],
                  volume: response["data"]["rows"][0][1]
                  //time: response["data"]["rows"][0][2] timeは表示するほどの情報ではないため省略
                };

                //Volumeが正常かどうか調べる
                const isDoten = this.dotenCheck(push_data.name); //ドテン方式かどうか
                const status_volume = strategy[3]; //status上のvolume
                const volume_range = isDoten
                  ? //ドテン式の場合の正常レンジ
                    [status_volume / 2, (status_volume / 2) * -1]
                  : //非ドテン式の場合の正常レンジ
                    [0, status_volume, status_volume * -1];

                //エラーメモ：volume_rangeがnumber push_data.volumeがStringとして扱われる場合が原因不明で発生する事がある
                //push_data.volumeをfloatへ変換して対応。逆に両方共toStringでやるとすべてOKになってしまうバグがあった
                push_data.isValid =
                  volume_range.some(
                    item => parseFloat(item) === parseFloat(push_data.volume)
                  ) === true
                    ? "OK"
                    : "ERROR";
                console.log(
                  push_data.name +
                    "の正常レンジ:" +
                    push_data.volume +
                    " = " +
                    JSON.stringify(volume_range) +
                    "正常？:" +
                    push_data.isValid
                );

                //トータルを計算
                this.total_volume_in_table =
                  this.total_volume_in_table +
                  parseFloat(response["data"]["rows"][0][1]);
                this.total_volume_in_table = this.bfKirisute(
                  this.total_volume_in_table
                );

                //volumeを加算:切り捨て処理とL/Sノーポジの仕分け
                if (parseFloat(response["data"]["rows"][0][1]) > 0) {
                  this.ls_table_long.push(push_data);
                  this.isErrorLong =
                    push_data.isValid === "ERROR" ? true : this.isErrorLong;
                } else if (parseFloat(response["data"]["rows"][0][1]) < 0) {
                  this.ls_table_short.push(push_data);
                  this.isErrorShort =
                    push_data.isValid === "ERROR" ? true : this.isErrorShort;
                } else {
                  this.ls_table_square.push(push_data);
                  this.isErrorSquare =
                    push_data.isValid === "ERROR" ? true : this.isErrorSquare;
                }
              }

              //loading完了判定用
              this.status_length--;
              if (this.status_length === 0) {
                this.loading = false;
              }
              console.log("残り:" + this.status_length);
            })
            .catch(reason => {
              this.error_config.push(
                "L/S仕分け中にエラー:historyテーブルのIDが間違っている可能性があります。"
              );
              console.log("L/S仕分け中にエラー:" + strategy[0] + ":" + reason);
            });
        }
      }
    },

    // bFの理論上トータルを計算 切り捨て箇所に注意
    calculateTotal() {
      var total = 0;
      for (let item of this.ls_table_long) {
        total = total + item["volume"];
      }
      for (let item of this.ls_table_short) {
        total = total + item["volume"];
      }
      total = this.bfKirisute(total);
      console.log(total);
      this.total_volume_in_table = total;
    },
    // functions -----------------------------------------------------------------------------
    bfKirisute(num) {
      return Math.round(num * 100) / 100;
    },
    dotenCheck(name) {
      //ドテンチェック:boolean
      if (this.doten_list instanceof Array) {
        //配列かどうか確認する ファイルの内容が[]でなかった場合はelseに行く
        return this.doten_list.some(item => item === name);
      } else {
        console.log("エラー： doten_listの内容が不正です！");
        this.openSnack(
          "エラー：doten_listが不正に書き換えられています!内容を確認してください",
          2
        );
        return false;
      }
    },
    // ドテンテーブルをダウンロード ------------------------------------------------------------
    downloadDotenTable() {
      var d = [];
      for (let item of this.status["data"]["rows"]) {
        if (item[8] === true) {
          d.push(item[0]);
        }
      }

      var blob = new Blob([JSON.stringify(d)], { type: "application/json" });
      let link = document.createElement("a");
      link.href = window.URL.createObjectURL(blob);
      link.download = "doten_list.json";
      link.click();
    },
    // ドテンテーブルをダウンロード(空ファイル) --------------------------------------------------
    downloadDotenTableDummy() {
      var d = [];

      var blob = new Blob([JSON.stringify(d)], { type: "application/json" });
      let link = document.createElement("a");
      link.href = window.URL.createObjectURL(blob);
      link.download = "doten_list.json";
      link.click();
    },
    toDotenListMaker() {
      //ドテンリスト作成画面へスイッチ
      this.$emit("toDotenListMaker");
    },
    openSnack(text, icon) {
      this.icon = icon; //v-iconの指定 1:check 2:error
      //メモ icon は{{}}で直接テキストで指定できるが、olorをv-bindで指定できない仕様なのでこの方法にする
      this.snack = false;
      this.snack_text = text;
      this.snack = true;
    },
    // DBG -------------------------------------------------------------------------------------
    save_local() {
      try {
        console.log("DBG:クライアントサイドストレージに保存します");

        localStorage.setItem("status", JSON.stringify(this.status));
        localStorage.setItem(
          "ls_table_long",
          JSON.stringify(this.ls_table_long)
        );
        localStorage.setItem(
          "ls_table_short",
          JSON.stringify(this.ls_table_short)
        );
        localStorage.setItem(
          "ls_table_square",
          JSON.stringify(this.ls_table_square)
        );
        //エラーコード
        localStorage.setItem(
          "error_records",
          JSON.stringify(this.error_records)
        );

        console.log("DBG:セーブ完了");
      } catch (e) {
        console.log("setItemでエラー:" + e);
      }
    },
    load_local() {
      try {
        console.log("DBG:クライアントサイドストレージからロードします");
        this.status = JSON.parse(localStorage.getItem("status"));
        for (let item of this.status["data"]["rows"]) {
          item[8] = false; //ドテンかどうかのフラグを初期化
        }
        this.ls_table_long = JSON.parse(localStorage.getItem("ls_table_long"));
        this.ls_table_short = JSON.parse(
          localStorage.getItem("ls_table_short")
        );
        this.ls_table_square = JSON.parse(
          localStorage.getItem("ls_table_square")
        );
        //エラーコード
        this.error_records = JSON.parse(localStorage.getItem("error_records"));

        console.log("DBG:ロード完了");
      } catch (e) {
        console.log("getItemでエラー");
        localStorage.removeItem("status");
      }
    }
  } //methods
};
</script>