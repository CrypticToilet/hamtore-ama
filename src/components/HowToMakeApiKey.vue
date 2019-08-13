<template>
  <v-dialog v-model="dialog" max-width="1000">
    <v-btn slot="activator" color="green lighten-4"><v-icon class="mr-2">help</v-icon>API Keyの取得方法</v-btn>
    <v-card class="pa-4">
      <v-card-title>
        <h2>Google Developers Console にて API Keyの取得方法</h2>
        <v-btn color="success" @click="dialog = false" class="mx-4">閉じる</v-btn>
      </v-card-title>
      <p><a href="https://console.developers.google.com/" target="_blank">Google Developers Console</a>を開いて取得します</p>

      <h3>1. "認証情報" を選択</h3>
      <img src="img/6.jpg" class="grey pa-1" />

      <h3 class="mt-4">2. "認証情報を作成" を選択</h3>
      <img src="img/7.jpg" class="grey pa-1" />

      <h3 class="mt-4">3. "API キー" を選択</h3>
      <p>ここをクリックすると発行されます</p>
      <img src="img/8.jpg" class="grey pa-1" />

      <h3 class="mt-4">4. "自分のAPIキー" に発行されたKeyが記入されています</h3>
      <p>キーをクリップボードにコピーしたら閉じてフォームに貼り付けます</p>
      <img src="img/9.jpg" class="grey pa-1" />

      <div class="my-2">
        <v-btn color="success" @click="dialog = false">閉じる</v-btn>
      </div>
    </v-card>
  </v-dialog>
</template>
<script>
export default {
  name: "HowToMakeApiKey",
  data() {
    return {
      dialog: false
    };
  }
};
</script>

