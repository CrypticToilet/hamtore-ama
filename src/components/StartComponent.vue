<template>
  <div>
    <v-layout row justify-center>
      <v-flex lg6 class="my-4">
        <v-card class="white ma-2" height="100%">
          <v-flex dark flat class="teal pa-1">
            <v-icon color="white">play_circle_filled</v-icon>
            <span class="title white--text px-2">Config File Maker</span>
            <span class="caption mx-2 white--text">設定ファイル(config.json)を作成します</span>
          </v-flex>
          <v-flex dark flat class="pa-4">
            <h1>起動に必要な設定ファイルを作成します(config.json)</h1>
            <h1>ダウンロードされますので"config"フォルダへコピーします</h1>
            <div>※メイン画面(ダッシュボード)を使用するにはconfig.jsonファイルが必須です</div>
            <div>※ファイルコピー後に再読み込みしてください</div>
          </v-flex>
          <v-form class="mx-4" ref="form">
            <v-text-field
              class="mx-1 mt-4"
              v-model="password"
              label="Password"
              required
              hint="専用パスワードを新規に指定します。設定をこのパスワードで暗号化します。省略可(暗号化なし)"
              persistent-hint
              :append-icon="e1 ? 'visibility' : 'visibility_off'"
              @click:append="() => (e1 = !e1)"
              :type="e1 ? 'password' : 'text'"
            ></v-text-field>

            <v-text-field
              class="mx-1"
              v-model="password_validation"
              label="Password"
              required
              hint="確認のためPasswordをもう一度入力します。"
              persistent-hint
              :append-icon="e2 ? 'visibility' : 'visibility_off'"
              @click:append="() => (e2 = !e2)"
              :type="e2 ? 'password' : 'text'"
              :rules="[v => password === v || 'パスワードが一致しません']"
            ></v-text-field>

            <v-layout align-center>
              <v-text-field
                v-model="history_id"
                :counter="41"
                :rules="[v => v.length === 41 || 'IDは41文字、入力必須です。']"
                label="History Table ID"
                hint="省略不可：History TableのIDを入力します。"
                required
                class="mt-4"
              ></v-text-field>
              <HowToPermit />
            </v-layout>
            <v-layout align-center>
              <v-icon>info</v-icon>テーブルの公開設定が必要です。
              <a
                :href="openTable(history_id)"
                target="_blank"
              >入力されたIDでテーブルを開く (History)</a>
            </v-layout>

            <v-text-field
              v-model="bf_status_id"
              :counter="41"
              :rules="[ v => ( v.length === 41 || bitmex_status_id.length === 41 && v.length === 0 ) || 'IDは41文字である必要があります。省略する場合はbitFlyerかBitMEXのIDどちらか１つが入力されている必要があります。' ]"
              label="bitFlyer Status Table ID"
              hint="省略可：bitFlyerのTable IDを入力します。"
              class="mt-4"
            ></v-text-field>
            <v-layout align-center>
              <v-icon>info</v-icon>テーブルの公開設定が必要です。
              <a
                :href="openTable(bf_status_id)"
                target="_blank"
              >入力されたIDでテーブルを開く (bitFlyer Status)</a>
            </v-layout>

            <v-text-field
              v-model="bitmex_status_id"
              :counter="41"
              :rules="[ v => ( v.length === 41 || bf_status_id.length === 41 && v.length === 0 ) || 'IDは41文字である必要があります。省略する場合はbitFlyerかBitMEXのIDどちらか１つが入力されている必要があります。' ]"
              label="BitMEX Status Table ID "
              hint="省略可：BitMEXのTable IDを入力します。"
              class="mt-4"
            ></v-text-field>
            <v-layout align-center>
              <v-icon>info</v-icon>テーブルの公開設定が必要です。
              <a
                :href="openTable(bitmex_status_id)"
                target="_blank"
              >入力されたIDでテーブルを開く (BitMEX Status)</a>
            </v-layout>
            <v-layout align-center>
              <v-text-field
                v-model="google_api"
                :rules="[v => !!v || '入力必須です。']"
                label="API Key (Google Developers Console)"
                hint="省略不可：キー毎に文字数が可変です。"
                required
                class="mt-4"
              ></v-text-field>
              <HowToMakeApiKey />
            </v-layout>
            <v-layout align-center>
              <v-icon>info</v-icon>API Keyの取得が必要です。
              <a
                href="https://console.developers.google.com/"
                target="_blank"
              >Google Developers ConsoleでKeyを取得</a>
            </v-layout>

            <v-layout align-center class="my-4">
              <v-checkbox v-model="disabled_url" class="shrink mr-2 mt-0"></v-checkbox>
              <v-text-field
                v-model="google_api_url"
                :rules="[v => !!v || 'Item is required']"
                label="Fusion Tables API URL"
                :disabled="!disabled_url"
                value="https://www.googleapis.com/fusiontables/v2/query?sql="
                required
                hint="URLはGoogleの仕様変更があった場合のみチェックボックスをONにして変更してください。わからない場合は変更しないでください。"
                persistent-hint
              ></v-text-field>
            </v-layout>

            <v-layout class="my-2">
              <v-btn color="success" class="mr-4" @click.stop="validate()">確認</v-btn>
              <v-btn color="error" class="mr-4" @click="reset()">リセット</v-btn>
            </v-layout>
          </v-form>
        </v-card>
      </v-flex>
    </v-layout>

    <!-- ダイアログ -->
    <v-layout justify-center>
      <v-dialog v-model="dialog" max-width="600">
        <v-card>
          <v-card-title class="headline">
            <h4>
              <v-icon class="mx-1">done_outline</v-icon>確認
            </h4>
          </v-card-title>

          <v-card-text v-if="password.length <= 4 && password.length > 0">
            <h4 class="red--text">
              <v-icon color="yellow darken-4" class="mx-1">warning</v-icon>パスワードが4文字以下ですがよろしいですか？
            </h4>
          </v-card-text>
          <v-card-text v-if="password.length === 0">
            <h2 class="red--text">
              <v-icon color="red" class="mx-1">warning</v-icon>パスワードが設定されていません！
            </h2>
            <h4 class="yellow">
              パスワードなしでも実行できますが、設定ファイルの暗号化はされません。
              <br />ローカルサーバー専用で稼働させる等リスクが低い環境でのみお使いください。
            </h4>
          </v-card-text>
          <v-card-text>
            <h4>ダウンロードボタンを押すとファイルが「ダウンロード」されます。 "config" フォルダ内へ上書き保存してください。</h4>
          </v-card-text>
          <!-- 接続テスト -->
          <v-card-text>
            <h3>接続テスト</h3>
            <v-progress-linear v-show="loading" :indeterminate="loading"></v-progress-linear>
            <ul>
              <li>
                Fusion Table API URLの確認：
                <v-icon color="green" v-if="test_api_url === 'OK'">check</v-icon>
                <v-icon color="red" v-if="test_api_url != 'OK' &&  test_api_url != '' ">error</v-icon>
                <span>{{ test_api_url }}</span>
              </li>
              <li>
                API KEYの確認：
                <v-icon color="green" v-if="test_api_key === 'OK'">check</v-icon>
                <v-icon color="red" v-if="test_api_key != 'OK' &&  test_api_key != '' ">error</v-icon>
                <span>{{ test_api_key }}</span>
              </li>
              <li>
                HistoryテーブルのID確認:
                <v-icon color="green" v-if="test_history_id === 'OK'">check</v-icon>
                <v-icon color="red" v-if="test_history_id != 'OK' &&  test_history_id != '' ">error</v-icon>
                <span>{{ test_history_id }}</span>
              </li>
              <li>
                bitFlyer StatusテーブルのID確認:
                <v-icon color="green" v-if="test_bf_id === 'OK'">check</v-icon>
                <v-icon
                  color="red"
                  v-if="test_bf_id != 'OK' &&  test_bf_id != '' && test_bf_id != '設定なし'"
                >error</v-icon>
                <span>{{ test_bf_id }}</span>
              </li>
              <li>
                BitMEX StatusテーブルのID確認:
                <v-icon color="green" v-if="test_mex_id === 'OK'">check</v-icon>
                <v-icon
                  color="red"
                  v-if="test_mex_id != 'OK' &&  test_mex_id != '' && test_mex_id != '設定なし'"
                >error</v-icon>
                <span>{{ test_mex_id }}</span>
              </li>
            </ul>
          </v-card-text>
          <!-- 接続テストここまで -->
          <v-card-text>
            <v-layout align-center>
              <v-icon class="mx-1" color="blue darken-4">info</v-icon>
              <a herf="#" @click="oritatami1 = !oritatami1">技術情報とリスク</a>
              <v-icon v-if="!oritatami1">arrow_drop_down</v-icon>
              <v-icon v-if="oritatami1">arrow_drop_up</v-icon>
            </v-layout>
            <div v-if="oritatami1" class="pa-2">
              <p>
                このシステムはすべてクライアントサイドで処理を行う(サーバー側の処理を必要としない)ため設定ファイルもクライアント側にロードされます。
                <br />従って誰にでもファイルの中身が見えてしまうというリスクがあります。
                <br />その対策としてcrypto-jsによりAESで暗号化しており、ここで設定したパスワードで復号化するシステムになっています。パスワードはいかなる場所にも保存されません。
                <br />これによりこのアプリの作者を含む第三者が設定ファイルの中身を取り出すことを難しくしています。
              </p>
              <p>
                このシステムはオンライン上の静的サイト用サーバーにデプロイすることを想定しておりますが
                念の為URLは他人に教えないようにしてください。
                <br />総当たり攻撃やキーロギングには十分ご注意ください。
              </p>
              <p>コードはStartComponent.vueのdownloadFileメソッド内で暗号化処理されております。</p>
            </div>
          </v-card-text>
          <v-card-text v-show="isTestOK">
            <h2 class="red--text">ダウンロードされたファイルは"config" フォルダ内へ上書き保存して下さい</h2>
          </v-card-text>
          <v-card-actions class="pa-4">
            <v-spacer></v-spacer>
            <v-checkbox v-model="isTestOK" class="shrink mr-2" label="接続テスト確認"></v-checkbox>
            <v-btn
              color="success"
              :disabled="!isTestOK"
              text
              @click="dialog = false;downloadFile()"
            >ダウンロード</v-btn>
            <v-btn color="error" text @click="dialog = false">キャンセル</v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>
    </v-layout>
    <!-- ダイアログここまで -->

    <!-- スナックバー -->
    <v-snackbar class="ma-2" :top="false" v-model="snack" :timeout="timeout">
      <v-icon v-if="icon === 1" color="green" class="ma-2">check</v-icon>
      <v-icon v-if="icon === 2" color="red" class="ma-2">error</v-icon>
      {{ snack_text }}
      <v-btn color="blue" text @click="snack = false">Close</v-btn>
    </v-snackbar>

    <!-- おまけ -->
    <v-layout row justify-center>
      <v-flex lg6 class="my-4">
        <v-card class="white ma-2" height="100%">
          <v-flex dark flat class="teal pa-1">
            <v-icon color="white">play_circle_filled</v-icon>
            <span class="title white--text px-2">Tool</span>
            <span class="caption mx-2 white--text">おまけツール</span>
          </v-flex>
          <div class="pa-4">
            <h3>URLからテーブルIDだけ切り抜くツール</h3>

            <v-text-field label="テーブルのURLをまるごと貼り付けてください" v-model="extract_table_id_target"></v-text-field>

            <v-text-field v-model="extracted_ID" counter="41" label="結果(テーブルID)"></v-text-field>
          </div>
        </v-card>
      </v-flex>
    </v-layout>
    <!-- おまけここまで -->
  </div>
</template>

<script>
import CryptoJS from "crypto-js";
import HowToPermit from "./HowToPermit";
import HowToMakeApiKey from "./HowToMakeApiKey";

export default {
  name: "StartComponent",
  components: { HowToPermit, HowToMakeApiKey },
  data() {
    return {
      snack: false, //スナックバー用
      snack_text: "", //スナックバー用
      timeout: 0, //スナックバー用
      icon: 1, //スナックバー用
      oritatami1: false, //折りたたみ：技術情報とリスク
      loading: false,
      isTestOK: false, //最後の接続テストが全部完了したか。チェックボックスをオンにすれば強制的にファイル出力もいける
      test_password: "", //後で削除
      dialog: false,
      disabled_url: false,
      password: "",
      password_validation: "",
      history_id: "",
      bitmex_status_id: "",
      bf_status_id: "",
      google_api: "",
      google_api_url: "https://www.googleapis.com/fusiontables/v2/query?sql=",
      e1: true, //passwordのvisible
      e2: true,
      config: {
        //google側にアクセスするときのヘッダ用コンフィグ
        headers: {
          Accept: "application/json"
        }
      },
      //接続テスト用
      test_api_url: "",
      test_api_key: "",
      test_history_id: "",
      test_bf_id: "",
      test_mex_id: "",
      //接続テスト用ここまで
      //おまけ機能用
      extract_table_id_target: ""
    };
  },
  methods: {
    validate() {
      if (this.$refs.form.validate()) {
        this.dialog = true;
        this.connection_test(); //接続テスト本体
      }
    },
    openTable(id) {
      return (
        "https://fusiontables.google.com/DataSource?docid=" + id + "#rows:id=1"
      );
    },
    //接続テスト
    async connection_test() {
      this.loading = true;
      //テスト結果のリセット
      this.test_api_url = "";
      this.test_api_key = "";
      this.test_history_id = "";
      this.test_bf_id = "";
      this.test_mex_id = "";
      this.isTestOK = false;

      //api urlのテスト
      //sql=の後に変な文字列が入っていてもテストでは正常と判断されてしまうのではじめにこの値をチェック
      if (this.google_api_url.slice(-4) != "sql=") {
        console.log(
          "接続テスト：api_urlを見直してください。sqlに既に変な値が設定されてしまっているようです。"
        );
        this.test_api_url =
          "api_urlを見直してください。sqlに既に変な値が設定されてしまっているようです。";
        return;
      }
      //そもそも存在しないサーバを指定した場合のためにtryが必要
      try {
        await this.$axios
          .get(this.google_api_url, this.config)
          .then(response => {
            console.log(
              "接続テスト：API取得中謎のエラー：エラーコードが帰ってこないということは別の正常なページが表示されてしまっている"
            );
            this.test_api_url =
              "API取得中謎のエラー：別の正常なページが表示されてしまっている？";
            return;
          })
          .catch(reason => {
            switch (reason.response.status) {
              case 403:
                //403が帰ってくれば正常(IDもKEYも指定せずAPIにアクセスした場合はURLさえ正しければ403が返る)
                console.log("接続テスト：api_urlは正常です");
                this.test_api_url = "OK";
                break;
              case 404:
                console.log(
                  "接続テスト：api_urlを正しく設定してください(404 not found)"
                );
                this.test_api_url =
                  "api_urlを正しく設定してください(404 not found)";
                return;
            }
          });
      } catch {
        console.log("接続テスト：api_urlのサーバーが存在しません。");
        this.test_api_url = "api_urlのサーバーが存在しません";
        return;
      }

      //historyのテスト
      this.test_history_id = await this.tableTest(this.history_id);
      //bfのテスト
      if (this.bf_status_id.length != 0) {
        this.test_bf_id = await this.tableTest(this.bf_status_id);
      } else {
        this.test_bf_id = "設定なし";
      }
      //mexのテスト
      if (this.bitmex_status_id.length != 0) {
        this.test_mex_id = await this.tableTest(this.bitmex_status_id);
      } else {
        this.test_mex_id = "設定なし";
      }

      console.log("test_history_id = " + this.test_history_id);
      console.log("test_bf_id = " + this.test_bf_id);
      console.log("test_mex_id = " + this.test_mex_id);

      if (
        this.test_api_url === "OK" &&
        this.test_api_key === "OK" &&
        this.test_history_id === "OK"
      ) {
        if (
          (this.test_bf_id === "OK" && this.test_mex_id === "OK") ||
          (this.test_bf_id === "OK" && this.test_mex_id === "設定なし") ||
          (this.test_bf_id === "設定なし" && this.test_mex_id === "OK")
        ) {
          this.isTestOK = true;
        }
      }

      this.loading = false;
    },
    async tableTest(id) {
      //func テーブルIDが存在するかテストする
      var r; //return用
      const sql = "SELECT * FROM " + id + " WHERE ROWID=0";
      const sql_act = sql + "&key=" + this.google_api;
      const url = this.google_api_url + encodeURI(sql_act);

      await this.$axios
        .get(url, this.config)
        .then(response => {
          this.test_api_key = "OK";
          r = "OK";
        })
        .catch(reason => {
          switch (reason.response.status) {
            case 400:
              switch (reason.response.data.error.errors[0].reason) {
                case "keyInvalid":
                  console.log("接続テスト：API KEYが不正です");
                  this.test_api_key = "API KEYが不正です";
                  r = "正しいAPI KEYを入力してから再テストしてください";
                  break;
                case "badQueryCouldNotParse":
                  console.log("接続テスト：IDが不正です");
                  r = "IDが不正です";
                  break;
                default:
                  console.log(
                    "接続テスト：400想定外のエラー" +
                      JSON.stringify(
                        reason.response.data.error.errors[0].reason
                      )
                  );
                  this.test_api_key =
                    "不明なエラー(reasonの値を確認してください) => " +
                    JSON.stringify(reason.response.data.error.errors[0].reason);
                  r =
                    "不明なエラー(reasonの値を確認してください) => " +
                    JSON.stringify(reason.response.data.error.errors[0].reason);
              }
              break;
            case 401:
              console.log(
                "接続テスト：401エラー(テーブルの公開設定がされていない)"
              );
              this.test_api_key = "OK";
              r = "テーブルの公開設定をしてください";
              break;
          }
        });

      return r;
    },
    reset() {
      this.password = "";
      this.password_validation = "";
      this.history_id = "";
      this.bitmex_status_id = "";
      this.bf_status_id = "";
      this.google_api = "";
      this.disabled_url = false;
      this.google_api_url =
        "https://www.googleapis.com/fusiontables/v2/query?sql=";
      this.$refs.form.resetValidation();

      //テスト結果のリセット
      this.test_api_url = "";
      this.test_api_key = "";
      this.test_history_id = "";
      this.test_bf_id = "";
      this.test_mex_id = "";
    },
    downloadFile() {
      var output = {};
      output.fusion_tables_id_history = this.history_id;
      output.fusion_tables_id_status_bf = this.bf_status_id;
      output.fusion_tables_id_status_bitmex = this.bitmex_status_id;
      output.api_key = this.google_api;
      output.api_url = this.google_api_url;
      output.encrypted_config = "";

      try {
        var cyp = {};
        if (this.password.length > 0) {
          //パスワードが指定されている場合暗号化
          cyp = {
            encrypted_config: CryptoJS.AES.encrypt(
              JSON.stringify(output),
              this.password
            ).toString()
          };
        } else {
          //パスワードなしは暗号化しない
          cyp = output;
        }

        //ファイルをダウンロードする(第三引数で改行による整形)
        var blob = new Blob([JSON.stringify(cyp, null, "\t")], {
          type: "application/json"
        });
        let link = document.createElement("a");
        link.href = window.URL.createObjectURL(blob);
        link.download = "config.json";
        link.click();

        
      this.openSnack("config.jsonがダウンロードされます。configフォルダへ保存後、ページを再読み込みしてください",1);

      } catch {
        console.log("error:暗号化処理中になんらかのエラー");
      }

    },
    openSnack(text, icon) {
      this.icon = icon; //v-iconの指定 1:check 2:error
      //メモ icon は{{}}で直接テキストで指定できるが、olorをv-bindで指定できない仕様なのでこの方法にする
      this.snack = false;
      this.snack_text = text;
      this.snack = true;
    }
  }, //methods
  computed: {
    extracted_ID: function() {
      const search1 = this.extract_table_id_target.indexOf("docid=");
      if (search1 > 0) {
        const result1 = this.extract_table_id_target.substr(search1 + 6);
        const search2 = result1.indexOf("#");
        const result2 = search2 > 0 ? result1.slice(0, search2) : result1;
        return result2;
      } else {
        return "";
      }
    }
  }
};
</script>

<style>
</style>