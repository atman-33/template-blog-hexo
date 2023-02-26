---
title: 【JavaScript】CreateJSとクラスを用いたシューティングゲームのサンプル
date: 2023-02-25 20:49:17
tags:
- JavaScript
- シューティングゲーム
categories: JavaScript
thumbnail: /img/thumbnails/thumbnail-9.png
---

CreateJSとクラス（ オブジェクト指向 ）を用いた『シューティングゲーム』を作成してみました。

2020/03/06 追記:
JavaScriptのコンストラクタ外にフィールドを宣言すると、スマホ用ブラウザでは動作しない問題が分かったため修正を実施。

## 概要
マウス、もしくは画面タッチで自機を動かしながら弾を発射することで、敵機を倒していくゲームとなります。

[サンプルゲームを再生](https://atman-33.github.io/shooting-game-js/)

[【ソースコードはこちら】](https://github.com/atman-33/shooting-game-js)

{% asset_img shooting_game.png %}

## パッケージ構成
構成は下記となります。

今回は js/class フォルダ内にPlayer（自機）、Enemy（敵機）、Bullet（弾 ）のクラスを３つ準備しました。

```
js-shooting-game
├ index.html
├ css
│  └base.css
└ js
   └class
　    ├Player.js
　    ├Enemy.js
　    └Bullet.js
```

## ソースコード解説
JavaScript のクラスは１クラス１ファイルにしました。
（Java方式です）

メインプログラムであるhtmlファイル（今回ではindex.html）から、各クラスのファイル読み込ませることで、クラスを使用することができます。

### ①メインプログラム序盤
index.htmlはシューティングゲームが動作するページとなります。

ここで、各CSSやJavascriptを読み込んでプログラムが動作します。

▼index.html
```
<html>
<head>
    <meta charset="utf-8" />
    <link rel="stylesheet" href="css/base.css" />

    <script src="//code.createjs.com/1.0.0/createjs.min.js"></script>
    <script src="js/class/Player.js"></script>
    <script src="js/class/Bullet.js"></script>
    <script src="js/class/Enemy.js"></script>

    <script type="text/javascript">

　以降、メインプログラムを記載
　　・
　　・
　　・
```

最初に CreageJS のライブラリを読み込んでいます。

```
<script src="//code.createjs.com/1.0.0/createjs.min.js"></script>
```

こうすることで、後で説明する各クラスも含めてCreateJSのオブジェクトが使用可能となります。

【補足】
//code.createjs.com/1.0.0/createjs.min.js の冒頭「//」は「http://」と「https://」のどちらにも対応して動作させることができます。

### ②プレイヤークラス
プレイヤー（自機）には、下記機能を持たせました。

マウス位置への移動
敵機を倒した際の経験値取得
【補足】
プレイヤーに準備させるべき機能として『弾を発射する』というのもあげられますが、プログラムのシンプル化も考慮してメインプログラム（index.html）側に持たせております。

▼Player.js
```
// ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ----
// breif : プレイヤークラス
// note  :
// ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ----
class Player extends createjs.Shape{

    // getter
    getX() { return this.x; }   // X位置を返す。
    getY() { return this.y; }   // Y位置を返す。
    getLevel() { return this.level }
    getExp() { return this.exp }

    // ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ----
    // breif : コンストラクタ
    // note  :
    // ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ----
    constructor() {

        super();    // 親クラスのコンストラクタ呼び出し

        this.x = 0;
        this.y = 0;

        this.exp = 0;
        this.level = 1;

        // プレイヤーの形を定義
        this.graphics.beginFill("white").moveTo(0, -10).lineTo(-5, 0).lineTo(5, 0).closePath();
    }

    // ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ----
    // breif : プレイヤーを移動する。
    // note  :
    // ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ----
    move(stage) {

        // 自機をマウス座標まで移動させる(減速で移動)
        this.x += (stage.mouseX - this.x) * 0.1;
        this.y += (stage.mouseY - this.y) * 0.1;
    }

    // ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ----
    // breif : 経験値を取得
    // note  :
    // ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ----
    addExp(exp) {
        this.exp = this.exp + exp;
        this.level = Math.ceil(this.exp / 500)
    }
}
```

プレイヤークラスはcreatejs.Shapeを継承したサブクラスにしております。
（つまり、createjs.Shapre はプレイヤークラスからするとスーパークラスとなります。）

こうすることで、 createjs.Shape の持つフィールドとメソッドが流用可能となります。

### ③敵クラス
敵クラスは先に述べたプレイヤークラスとほぼ同じです。

一つ異なる点は、『弾との衝突判定を行う』機能を持たせている点です。この機能を用いて、プレイヤーが放った弾と衝突した際に消滅させる処理をメインプログラム側に実装させています。

▼Enemy.Class
```
// ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ----
// breif : エネミークラス
// note  :
// ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ----
class Enemy extends createjs.Shape{

    // getter
    getX() { return this.x; }   // X位置を返す。
    getY() { return this.y; }   // Y位置を返す。

    // ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ----
    // breif : コンストラクタ
    // note  :
    // ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ----
    constructor(stageW, stageH) {

        super();    // 親クラスのコンストラクタ呼び出し

        this.x = 0;
        this.y = 0;

        // 敵の形を定義
        this.graphics.beginFill("red").moveTo(10,-5).lineTo(10,5).lineTo(5,5).lineTo(5,10)
            .lineTo(-5,10).lineTo(-5,5).lineTo(-10,5).lineTo(-10,-5).closePath();


        // 画面上側からランダムに生成
        this.x = stageW * Math.random();
        this.y = stageH;
    }

    // ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ----
    // breif : 敵を移動する。
    // note  :
    // ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ----
    move() {

        // 敵を移動させる。
        this.y += 1;
    }

    // ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ----
    // breif : 敵とプレイヤーの弾との衝突判定をする。
    // note  :
    // ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ----
    collideWith(bullet) {

        // 敵から見た発射弾のローカル座標を取得
        var pt = bullet.localToLocal(0, 0, this);

        // 当たり判定を行う
        return this.hitTest(pt.x, pt.y);

    }
}
```

ここでは、衝突判定に扱っている localToLocal や hitTest の詳細説明は割愛します。

簡単に言うとすれば、敵（自分）を軸に弾の位置を再計算させた上で、図形が重なっているかどうかを確認させています。

### ④弾クラス
弾クラスもプレイヤークラス、エネミークラスとほぼ同じです。

▼Bullet.js
```
// ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ----
// breif : 弾クラス
// note  :
// ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ----
class Bullet extends createjs.Shape{

    // getter
    getX() { return this.x; }   // X位置を返す。
    getY() { return this.y; }   // Y位置を返す。

    // ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ----
    // breif : コンストラクタ
    // note  :
    // ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ----
    constructor(x, y, level) {

        super();    // 親クラスのコンストラクタ呼び出し

        this.x = x;
        this.y = y;

        this.level = level;

        // 弾の形を定義
        this.graphics.beginFill("white").drawCircle(0, 0, 3);
    }

    // ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ----
    // breif : 弾を移動する。
    // note  :
    // ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ----
    move() {

        // 弾を移動させる。
        this.y -= 10 + this.level;
    }
}
```
【補足】
ここまで説明したプレイヤークラス、エネミークラス、弾クラスの３つは、類似したフィールドやメソッドを使用しています。

このような場合、オブジェクト指向では継承を用いるべきなのです。

参考までにより良いと思う継承の流れを記載しておきます。
createjs.Shape　→　（仮）PanelShape
　　　　　　　　　　　　　├　→　Plyaer.js
　　　　　　　　　　　　　├　→　Enemy.js
　　　　　　　　　　　　　└　→　Bullet.js

### ⑤メインプログラム
最後に今まで説明した各クラスをインスタンス化して活用するメインプログラムについてです。

▼index.html
```
<html>
<head>
    <meta charset="utf-8" />
    <link rel="stylesheet" href="css/base.css" />

    <script src="//code.createjs.com/1.0.0/createjs.min.js"></script>
    <script src="js/class/Player.js"></script>
    <script src="js/class/Bullet.js"></script>
    <script src="js/class/Enemy.js"></script>

    <script>

// 読み込みが終わってから初期化
window.addEventListener("load", init);
function init() {
    var stage = new createjs.Stage("myCanvas");

    var enemyList = [];   // 敵の配列
    var bulletList = [];  // 発射弾の配列
    var count = 0;        // フレーム番号
    var scoreNum = 0;     // スコア
    var STAGE_W = 540;    // 画面サイズ
    var STAGE_H = 960;

    // 背景を作成
    var bg = new createjs.Shape();
    bg.graphics.beginFill("black").drawRect(0, 0, STAGE_W, STAGE_H);
    stage.addChild(bg);

    // 自機を作成
    var player = new Player();
    stage.addChild(player); // ステージに追加

    // スコア欄を作成
    var scoreBoard = new createjs.Text("", "20px sans-serif", "white");
    scoreBoard.text = "Score:" + String(0);
    stage.addChild(scoreBoard);

    // レベル欄を作成
    var levelBoard = new createjs.Text("", "20px sans-serif", "white");
    levelBoard.text = "level:" + String(1);
    levelBoard.x = 150;
    levelBoard.y = 0;
    stage.addChild(levelBoard);

    // マウス座標欄を作成
    var mouseBoard = new createjs.Text("", "10px sans-serif", "white");
    mouseBoard.x = 0;
    mouseBoard.y = 30;
    stage.addChild(mouseBoard);

    // タッチ操作も可能にする(iOS,Android向け)
    if (createjs.Touch.isSupported()) {
        createjs.Touch.enable(stage);
    }

    // マウスイベントの登録
    stage.addEventListener("click", handleClick);

    // tick イベントの登録
    createjs.Ticker.framerate = 60; // setFPSは非推奨
    createjs.Ticker.addEventListener("tick", handleTick);

    // クリックした時の処理
    function handleClick(event) {

      // レベル増加に合わせて弾数増加
      for (var i = 0; i < player.level; i++) {
        var bullet = new Bullet(player.x, player.y, player.level);
        stage.addChild(bullet);   // ステージに追加
        bulletList.push(bullet);  // 配列に保存

        if (player.level > 1) {
          var bullet = new Bullet(player.x + i * 10, player.y, player.level);
          stage.addChild(bullet);   // ステージに追加
          bulletList.push(bullet);  // 配列に保存
        }

        if (player.level > 1) {
          var bullet = new Bullet(player.x - i * 10, player.y, player.level);
          stage.addChild(bullet);   // ステージに追加
          bulletList.push(bullet);  // 配列に保存
        }
      }
    }

    // tick イベントの処理
    function handleTick() {

        // 自機をマウス座標まで移動させる(減速で移動)
        player.move(stage);

        // フレーム番号を更新(インクリメント)
        count = count + 1;

        // 60フレームに1回、敵を生成
        if (count % 60 == 0) {
          // 敵を生成
          var enemy = new Enemy(STAGE_W, 50);
          stage.addChild(enemy);   // ステージに追加
          enemyList.push(enemy);  // 配列に保存
        }

        // 発射弾の移動処理
        for (var i = 0; i < bulletList.length; i++) {

          bulletList[i].move();

          // 画面端まで移動したら
          if (bulletList[i].x > STAGE_W || bulletList[i].y > STAGE_H) {
            stage.removeChild(bulletList[i]); // 画面から削除
            bulletList.splice(i, 1); // 配列から削除
          }
        }

        // 敵の移動処理
        for (var i = 0; i < enemyList.length; i++) {

          // 敵機の移動
          enemyList[i].move();

          // 画面端まで移動したら
          if (enemyList[i].x < 0 || enemyList[i].y > STAGE_H) {
            showGameOver(); // ゲームオーバー処理へ
          }
        }

        // 発射弾と敵の当たり判定
        for (var j = 0; j < enemyList.length; j++) {
          for (var i = 0; i < bulletList.length; i++) {
            var bullet = bulletList[i];
            var enemy = enemyList[j];

            // 当たり判定を行う
            if (enemy.collideWith(bullet) == true) {
              // 発射弾の削除
              stage.removeChild(bullet);
              bulletList.splice(i, 1);

              // 敵の削除
              stage.removeChild(enemyList[j]);
              enemyList.splice(j, 1);

              // スコアの更新
              scoreNum += 100;
              player.addExp(100);

              // ステージクリア判定
              if (scoreNum >= 3000) {
                showGameClear();
              }

              break;
            }
          }
        }

        // 各テキストの更新
        scoreBoard.text = "Score:" + String(scoreNum);    
        levelBoard.text = "Level:" + String(player.getLevel()) + "  Exp:" + String(player.getExp());
        mouseBoard.text = "Mouse X:" + String(stage.mouseX) + " Y:" + String(stage.mouseY);


        // ステージの更新
        stage.update();
  }

  // ゲームオーバー
  function showGameOver() {
    alert("ゲームオーバー！ あなたのスコアは " + scoreNum + " でした。");

    // 各種イベントをまとめて解除
    createjs.Ticker.removeAllEventListeners();
    stage.removeAllEventListeners();
  }

  // ステージクリア
  function showGameClear() {
    alert(scoreNum + "点達成！　ゲームクリア！");

    // 各種イベントをまとめて解除
    createjs.Ticker.removeAllEventListeners();
    stage.removeAllEventListeners();
  }
}


    </script>

</head>
<body>
    <canvas id="myCanvas" width="540" height="960"></canvas>
</body>
</html>
```

大きな流れとしては、初期化　→　ループ処理　となります。

この方法は、ゲームも含め様々なアプリケーションの基本的な流れとなりますので重要な考え方です。

初期化　→　function init
ループ処理　→　function handleTick

更に規模の大きいゲームやアプリケーションを作る際、メインプログラムと役割ごとのクラスを作成していく考え方は役に立つと思います。

以上です。
