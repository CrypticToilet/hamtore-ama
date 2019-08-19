<template>
  <div>
    <v-layout row wrap>
      <v-flex xs12>
        <v-card class="white ma-2">
          <v-flex dark flat class="teal pa-1">
            <v-icon color="white">description</v-icon>
            <span class="white--text px-2">説明</span>
          </v-flex>

          <v-card-text>
            <h1>
              <v-icon large color="blue">info</v-icon>ドテン式(ノーポジが無い)ストラテジのリストを作成します。ポジション計算に必須です。
            </h1>
            <div>※ドテン式の例：Vega, MotuStart, 00Amber, perno, UCSGEARS, viostore 等</div>
            <h3 class="mt-3">
              <v-icon>check</v-icon>ドテンリスト作成手順
            </h3>
            <ol>
              <li><b class="red--text">"テーブルを読み込み"</b>ボタンをクリックしてStatusテーブルを読み込みます。</li>
              <li>
                bF/MEXに登録されたストラテジ一覧が表示されます。
                <b class="red--text">ドテン式のストラテジ全てのチェックボックスをオンにしてください。</b>
              </li>
              <li><b class="red--text">"ファイルに保存"</b>ボタンを押すとチェックされているストラテジ名一覧(doten_list.json)がダウンロードされます。</li>
              <li>ファイルがダウンロードされたら"config"フォルダにファイルをコピー(上書き)して はむとれ ama をリロードしてください。</li>
            </ol>
          </v-card-text>
        </v-card>
      </v-flex>
      <v-progress-linear v-show="loading" :indeterminate="loading"></v-progress-linear>
      <!-- Stats -->
      <v-flex xs12 class="my-4">
        <v-card class="white ma-2">
          <v-flex dark flat class="teal pa-1">
            <v-icon color="white">description</v-icon>
            <span class="white--text px-2">bitFlyer Status</span>
          </v-flex>

          <v-layout row>
            <v-btn @click="getStatus()" href="#" class="teal white--text" :loading="loading">
              <v-icon color="white">play_circle_filled</v-icon>
              <span class="px-2">テーブルを読み込み</span>
            </v-btn>
            <v-btn @click="loadDotenList()" href="#" class="teal white--text" :loading="loading">
              <v-icon color="white">play_circle_filled</v-icon>
              <span class="px-2">doten_listを読み込み</span>
            </v-btn>
            <v-btn
              @click="downloadDotenTable()"
              href="#"
              class="teal white--text"
              :loading="loading"
            >
              <v-icon color="white">save</v-icon>
              <span class="px-2">ファイルに保存</span>
            </v-btn>

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
            :items="strategies"
            class="elevation-1"
            :search="v_table_search"
            :rows-per-page-items="[9999, 10, 20, 30]"
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

        <v-card class="white ma-2 mt-4">
          <v-card-text>
            <h3>次のStatusテーブルが読み込まれます (IDがconfigにて設定済み)</h3>
            <div v-if="passconfig.fusion_tables_id_status_bf != undefined">
              <v-icon color="green">check_circle</v-icon>bitFlyer
            </div>
            <div v-if="passconfig.fusion_tables_id_status_bitmex != undefined">
              <v-icon color="green">check_circle</v-icon>BitMEX
            </div>
            <div
              v-if="passconfig.fusion_tables_id_status_bitmex === undefined && passconfig.fusion_tables_id_status_bitmex === undefined"
            >
              <v-icon color="red" class="ma-2">error</v-icon>エラー：bF/MEXどちらのテーブルIDも読み込まれませんでした。config.jsonが正しく作成されていない可能性があります。
            </div>
          </v-card-text>

          <v-card-text>
            <h3>
              <v-icon>check</v-icon>
              <a href="#" @click="oritatami1 = !oritatami1">なぜドテン型のストラテジリストが必要なのですか？</a>
              <v-icon v-if="!oritatami1">arrow_drop_down</v-icon>
              <v-icon v-if="oritatami1">arrow_drop_up</v-icon>
            </h3>
            <v-card-text v-if="oritatami1">
              はむとれのストラテジはON/OFF型（ロング・ショート・ノーポジ）ドテン型(ロング・ショート) の2種類です。
              <br />全てのストラテジのVolumeが"1"に指定してあった場合はそれぞれ
              <br />ON/OFF型： 1, 0, -1 (Status上のVolume:1)
              <br />ドテン型: 1, -1 (Status上のVolume:2) いずれかのポジションとなります。
              <br />はむとれ ama はStatus上のVolumeとドテン式かどうかで理論上のポジション範囲を計算してエラーを判断しています。
              <br />従って ON/OFF型は Volume*1 , Volume*0, Volume*-1 のいずれかのポジション
              <br />ドテン型は Volume*1/2, Volume*-1/2 のいずれかのポジション
              <br />という比較で分ける必要があります。この兼ね合いでドテンリストを作っていないとhistory上に計上された値がエラーかどうかが判断できません。
              <v-card-text class="grey--text">
                ※はむとれ ama はピラミッディング型(同じ方向に何度もエントリーする)ストラテジの不整合チェックには対応しておりません。
                <br />また、この仕組みでは不整合を100%発見することができません(全て拾うためにはpineをプログラムで解釈 or TVをスクレイピングする必要があり実装が困難です)
              </v-card-text>
            </v-card-text>
          </v-card-text>
        </v-card>
      </v-flex>
    </v-layout>

    <!-- スナックバー -->
    <v-snackbar class="ma-2" :top="false" v-model="snack" :timeout="timeout">
      <v-icon v-if="icon === 1" color="green" class="ma-2">check</v-icon>
      <v-icon v-if="icon === 2" color="red" class="ma-2">error</v-icon>
      {{ snack_text }}
      <v-btn color="blue" text @click="snack = false">Close</v-btn>
    </v-snackbar>
  </div>
</template>

<script>
export default {
  name: "MakeDotenListComponent",
  data() {
    return {
      snack: false,
      icon: 1, //スナックバーのアイコン用(直接テキストで指定する)
      icon_color: "red",
      timeout: 0, //スナックバー用
      snack_text: "",
      oritatami1: false,
      v_table_search: "",
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
      pagination: {
        rowsPerPage: 9999
      },
      doten_list: {},
      loading: false,
      strategies: [],
      //statsuテーブルを格納
      status: {
        data: {
          rows: []
        }
      }
    };
  },
  props: ["passconfig"], //user_configの親から子へ受け渡し用。受け渡しの制約で"_"が使用できないため別の名前を指定している
  created() {
    if (this.passconfig.encrypted_config != "") {
      //まだ暗号化解除されていないのでテーブルを読み込める状態ではないためログインへ遷移する
      this.$emit("needLogin");
    }
  },
  methods: {
    async getStatus() {
      this.loading = true;

      //リセット
      this.strategies = [];

      //bFのstatus
      if (this.passconfig.fusion_tables_id_status_bf != undefined) {
        await this.getStatusSql(this.passconfig.fusion_tables_id_status_bf);
      }

      //mexのstatus
      if (this.passconfig.fusion_tables_id_status_bf != undefined) {
        await this.getStatusSql(this.passconfig.fusion_tables_id_status_bitmex);
      }

      console.log("status取得：正常終了");
      this.openSnack("status読み込み完了 : ドテン式のストラテジをチェック後ファイルを保存します", 1);
      this.loading = false;
    },
    async getStatusSql(table) {
      const sql = "SELECT * FROM " + table;
      const sql_act = sql + "&key=" + this.passconfig.api_key;
      const url = this.passconfig.api_url + encodeURI(sql_act);
      console.log("statusを取得します：" + url);
      await this.$axios
        .get(url, this.config)
        .then(response => {
          this.strategies = this.strategies.concat(response["data"]["rows"]); //配列をマージ(concact)
          for (let strategy of this.strategies) {
            //ドテンリストと照らし合わせる
            const isDoten = this.dotenCheck(strategy[0]);
            strategy[8] = isDoten;
          }
        })
        .catch(reason => {
          console.log("status取得に失敗しました:" + reason);
          this.openSnack("status取得に失敗しました", 2);
        });
    },
    dotenCheck(name) {
      //ドテンチェック:boolean
      if (this.doten_list.length > 0) {
        return this.doten_list.some(item => item === name);
      } else {
        return false;
      }
    },
    async loadDotenList() {
      this.loading = true;
      //doten_list.jsonの読み込み
      await this.$axios
        .get("config/doten_list.json")
        .then(response => {
          this.doten_list = response.data;
          for (let strategy of this.strategies) {
            const isDoten = this.dotenCheck(strategy[0]);
            strategy[8] = isDoten;
          }
          this.openSnack("読み込み完了 : doten_list.json", 1);
        })
        .catch(e => {
          console.log("doten_list.jsonは見つかりませんでした。");
          this.openSnack("読み込み失敗 : doten_list.json (この機能は既にdoten_list.jsonが配置してある場合にのみ有効です)", 2);
        });
      this.loading = false;
    },
    openSnack(text, icon) {
      this.icon = icon; //v-iconの指定 1:check 2:error
      //メモ icon は{{}}で直接テキストで指定できるが、olorをv-bindで指定できない仕様なのでこの方法にする
      this.snack = false;
      this.snack_text = text;
      this.snack = true;
    },
    // ドテンテーブルをダウンロード ------------------------------------------------------------
    downloadDotenTable() {
      try {
        var d = [];
        for (let item of this.strategies) {
          if (item[8] === true) {
            d.push(item[0]);
          }
        }
        //ファイルをダウンロードする(第三引数で改行による整形)
        var blob = new Blob([JSON.stringify(d, null, "\t")], {
          type: "application/json"
        });
        //var blob = new Blob([JSON.stringify(d)], { type: "application/json" }); 改行しない場合
        let link = document.createElement("a");
        link.href = window.URL.createObjectURL(blob);
        link.download = "doten_list.json";
        link.click();

        this.openSnack(
          "doten_list.jsonはconfigフォルダへコピーして下さい。その後はむとれ amaをリロードします。",
          1
        );
      } catch {
        this.openSnack("doten_list.json保存中にエラー", 2);
      }
    }
  } //methods
};
</script>