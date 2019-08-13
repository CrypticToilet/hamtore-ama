<template>
  <v-app class="teal lighten-5">
    <v-toolbar app class="teal lighten-1">
      <v-toolbar-side-icon class="white--text" @click="drawer = !drawer"></v-toolbar-side-icon>
      <v-toolbar-title class="headline white--text">
        <span>はむとれ ama</span>
      </v-toolbar-title>
      <v-spacer></v-spacer>
      <span class="mr-2">Ver. 1.0.0</span>
    </v-toolbar>

    <v-content class="mx-4 my-4">
      <h4 v-if="error_msg.length != 0">{{ error_msg }}</h4>
      <MainComponent
        :passconfig="passconfig"
        v-if="this.component_switch === 2"
        @toDotenListMaker="toDotenListMaker()"
        @needLogin="needLogin"
      />
      <LoginComponent
        :passconfig="passconfig"
        v-if="this.component_switch === 1"
        @decrypted="decrypted"
      />
      <StartComponent v-if="this.component_switch === 0" />
      <MakeDotenListComponent
        :passconfig="passconfig"
        v-if="this.component_switch === 3"
        @needLogin="needLogin"
      />
    </v-content>

    <!-- フッター -->
    <v-footer dark padless class="mt-4">
      <v-layout>
        <v-flex xs12 class="teal lighten-1 white--text pa-1">
          <v-btn
            round
            depressed
            color="blue darken-2"
            class="mx-3"
            href="https://twitter.com/cryptictoilet"
            target="_blank"
          >
            <img src="img/Twitter_Social_Icon_Circle_Color.png" height="21" class="mr-2" />
            Twitter
          </v-btn>
          <v-icon small>copyright</v-icon>
          <strong>2019 仮想トイレ (Cryptic Toilet)</strong>
        </v-flex>
      </v-layout>
    </v-footer>

    <!-- ドロワー -->
    <v-navigation-drawer
      app
      absolute
      temporary
      v-model="drawer"
      style="position:fixed; top:0; left:0;"
    >
      <v-toolbar class="teal lighten-1">
        <v-list>
          <v-list-tile>
            <v-list-tile-title class="white--text">MENU</v-list-tile-title>
          </v-list-tile>
        </v-list>
      </v-toolbar>

      <v-divider></v-divider>

      <v-list class="pt-0" dence rounded>
        <v-list-tile
          v-for="link in links"
          :key="link.text"
          @click="component_switch = link.switch;drawer = !drawer"
        >
          <v-list-tile-action>
            <v-icon>{{link.icon}}</v-icon>
          </v-list-tile-action>

          <v-list-tile-content>
            <v-list-tile-title>{{link.text}}</v-list-tile-title>
          </v-list-tile-content>
        </v-list-tile>
      </v-list>
    </v-navigation-drawer>

    <!-- スナックバー -->
    <v-snackbar class="ma-2" :top="true" v-model="snack" :timeout="timeout">
      <v-icon v-if="icon === 1" color="green" class="ma-2">check</v-icon>
      <v-icon v-if="icon === 2" color="red" class="ma-2">error</v-icon>
      {{ snack_text }}
      <v-btn color="blue" text @click="snackbar = false">Close</v-btn>
    </v-snackbar>
  </v-app>
</template>

<script>
import MainComponent from "./components/MainComponent";
import StartComponent from "./components/StartComponent";
import LoginComponent from "./components/LoginComponent";
import MakeDotenListComponent from "./components/MakeDotenListComponent";

//設定データ importはassetから読む方式
//テーブルのidやapikeyなどはpublicに配置しxiosで読み込んでください
//import user_config from "./assets/config.json";

export default {
  name: "App",
  components: {
    MainComponent,
    StartComponent,
    LoginComponent,
    MakeDotenListComponent
  },
  data() {
    return {
      drawer: false,
      links: [
        { icon: "dashboard", text: "ダッシュボード", switch: 2 },
        { icon: "create", text: "ドテンリスト作成", switch: 3 },
        { icon: "create", text: "コンフィグファイル作成", switch: 0 }
      ],
      component_switch: 0, //0=start , 1=login, 2=main , 3=エラー
      error_msg: "",
      user_config: {},
      passconfig: {},
      snack: false, //スナックバー用
      timeout: 2000, //スナックバー用
      snack_text: "", //スナックバー用
      icon: 1
    };
  },
  async created() {
    //メモ：App.vueは内部のコンポーネントをロードし直して擬似的に画面遷移を作っているので
    //config.jsonは起動時に１度しかロードされない
    //従ってパスワード画面から復号化がかかった場合、随時上書きされていく仕組み(暗号化されたconfigが復号化されたconfigで上書きされる)
    await this.$axios
      .get("config/config.json")
      .then(response => {
        this.user_config = response.data;
      })
      .catch(e => {
        console.log(
          "configファイルが存在しませんでした(ファイル作成画面へ移動します):" +
            JSON.stringify(e)
        );
      });

    //コンフィグファイルはがObjectかどうか判断している
    //例：ああああ等 や 空のデータの場合は stringとして読み込まれるのでfalse ←故意にデータを書き換えた場合に起こる可能性
    //[]や{}はどちらもObject扱いになる。この場合↓の条件式はパスするがその次で更に内容確認しているので問題なし
    if (
      Object.prototype.toString.call(this.user_config) === "[object Object]"
    ) {
      if ("encrypted_config" in this.user_config === false) {
        //configにキーが存在するかチェックする
        this.component_switch = 0; //config作成画面へ
        console.log(
          "設定ファイルが空：Config File Makerへ移動" + this.user_config
        );
      } else if (this.user_config.encrypted_config.length > 0) {
        console.log("ログイン画面へ移動：設定ファイルは暗号化されています");
        this.passconfig = this.user_config;
        this.component_switch = 1; //login画面へ
      } else if (this.user_config.encrypted_config.length === 0) {
        this.component_switch = 2; //main画面へ
        console.log("メイン画面へ移動：設定ファイルは暗号化されていません");
        this.decrypted(this.user_config);
      } else {
        this.component_switch = 4; //エラー
        this.error_msg = "エラー： configファイルのencrypted_configデータが空";
      }
    } else {
      this.openSnack(
        "エラー：コンフィグファイルに何もデータがありませんでした。故意に空ファイルが置かれている可能性があります",
        2
      );
    }
  },
  methods: {
    decrypted(d) {
      this.user_config = d;
      this.component_switch = 2;
      this.passconfig = d;

      this.openSnack("ログインOK", 1);
    },
    //メイン画面からドテンリスト作成画面へスイッチ
    toDotenListMaker() {
      this.component_switch = 3;
    },
    openSnack(text, icon) {
      this.icon = icon; //v-iconの指定 1:check 2:error
      //メモ icon は{{}}で直接テキストで指定できるが、olorをv-bindで指定できない仕様なのでこの方法にする
      this.snack = false;
      this.snack_text = text;
      this.snack = true;
    },
    needLogin() {
      //ドテンリストメーカーにログインなしでアクセスすると、これが起動する
      //はじめにそもそも設定ファイルがロードされているかチェックする
      if (
        Object.prototype.toString.call(this.user_config) === "[object Object]"
        && "encrypted_config" in this.user_config
      ) {
        //ロードされている
        this.component_switch = 1;
        this.openSnack("ログインが必要です", 2);
      } else {
        this.component_switch = 0;
        this.openSnack("configファイルが必要です", 2);
      }
    }
  }
};
</script>
