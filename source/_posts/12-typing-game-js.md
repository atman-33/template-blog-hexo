---
title: 【Javascript】タイピングゲームのサンプル
date: 2023-02-25 23:38:29
tags:
- JavaScript
- タイピングゲーム
categories: JavaScript
thumbnail: /img/thumbnails/thumbnail-12.png
---

JavaScript を用いた『タイピングゲーム』を作成してみました。

___
目次
<!-- toc -->

___

## 概要
制限時間内に、画面に表示された英単語をキーボードから入力していくゲームとなります。

[サンプルゲームを再生](https://atman-33.github.io/typing-game-js/)

[【ソースコードはこちら】](https://github.com/atman-33/typing-game-js)

{% asset_img typing.png %}

## パッケージ構成
構成は下記となります。

```
js_typing_game
├ index.html
├ css
│  └base.css
└ js
   ├Common.js
   └class
　    ├FlashingText.js
　    └Typing.js
```

## ソースコード解説
ゲームを動かすメインのファイルは index.html となります。

各ファイルに関する機能の概要は下記となりますので、大枠を掴んで頂ければ幸いです。

```
index.html	ゲームを動かすメイン処理
base.css	ゲーム画面やキーボードの見栄えを加工
Common.js	問題文のテキスト読込
FlashingText.js	テキストの点滅処理
Typing.js	タイピング文字の判定やキーボード点灯処理
```

ポイントとなるコード部分を主に解説していきます。

___
### ①index.html
今回、ゲームの状態（gameState）を定義し、プレイヤーの操作に応じて、『準備中』→『タイピングゲーム中』→『ゲーム終了』と遷移するように工夫しました。

```
<!DOCTYPE html>
<html>
    <head>
        <meta http-equiv="content-type" charset="utf-8">
        <link rel="stylesheet" href="css/base.css" />
        <script src="//code.createjs.com/1.0.0/createjs.min.js"></script>
        <script src="js/Common.js"></script>
        <script src="js/class/Typing.js"></script>
        <script src="js/class/FlashingText.js"></script>
        <script>
// ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ----
// JavaScript start

// タイピングゲームの状態コード
const READY = 0;    // 準備中
const PLAYING = 1;  // タイピングゲーム中
const END = 2;      // ゲーム終了

var score = 0;      // スコア
var timer = 30;     // タイマー（秒）

var countdown = 3;              // 準備中のカウントダウン
var wordsFile = "words.txt";    // 問題が格納されたテキストファイル

var gameState = READY;      // タイピングゲームの状態
var typing = new Typing();  // タイピングクラス

var flashingText = new FlashingText("input", "スペースを押してください！！", 1000, 500, 0);

var scoreText;
var timerText;

var subject;        // 上段テキスト表示スペース：題名
var input;          // 下段テキスト表示スペース：入力表示内容

var statementList;  // 問題リスト
var statement;      // 問題文
var numOfStatement; // 問題文の何文字目かを表す番号

window.addEventListener("keydown", handleKeydown);
window.addEventListener("keyup", handleKeyup);

// ページ読み込み時に実行する処理
window.onload = function(){

    statementList = readTextFileToArray(wordsFile);
    typing.insertKeyboard("board");

    // 0:準備中
    if(gameState == READY) {

        flashingText.flash();   // 開始前のテキスト点滅表示
    }
}

// ---- ---- ---- ---- ----
// Key 処理 start

// Keydown 処理
function handleKeydown(event) {

    // 入力文字の処理
    var chara = typing.checkWord(event);
    // alert("押されたキーのコード : " + event.keyCode);

    // 0:準備中
    if(gameState == READY) {
        if(event.keyCode == 32){    // スペースを押すと
            startReady();           // 問題開始前の準備へ
        }
    }

    // 1:タイピングゲーム中
    if(gameState == PLAYING) {
        judgeTyping(chara);         // 打ち込まれた文字の正誤判定へ
    }
}

// Keyup 処理
function handleKeyup(event) {

    // CapsLock確認
    var capslock;

    if(typing.checkCapsLock(event) == 1) {
        capslock = document.getElementById('capslock');
        capslock.innerHTML = "CapsLock ON";
    } else {
        capslock = document.getElementById('capslock');
        capslock.innerHTML = "CapsLock OFF";
    }
}

// Key 処理 end
// ---- ---- ---- ---- ----

// ゲーム開始前の準備
function startReady() {

    flashingText.setMsg("まもなく開始します...");
    flashingText.setIsFlashing(0);  // 点滅を停止

    // subject IDタグにカウントダウンを表示
    subject = document.getElementById('subject');
    subject.innerHTML = countdown;

    // 1秒おきにカウントダウン
    var id = setInterval(function(){
        countdown--;
        console.log(countdown);

        subject.innerHTML = countdown;

        if(countdown <= 0){
            clearInterval(id);
            console.log("Finish!");
            gameState = PLAYING;

            startTimer();   // 残り時間のカウントダウンスタート
            loadSubject();  // 問題文の読み込みへ
        }
    }, 1000);
}

// 問題文を読み込み
function loadSubject() {

    // statementList = ["Test", "Apple", "Banana"];

    // 問題リストからランダムに問題を取り出す
    statement = statementList[Math.floor(Math.random() * statementList.length)];

    // 問題文の表示
    subject = document.getElementById('subject');
    subject.innerHTML = statement;

    // 入力内容の表示
    subject = document.getElementById('input');
    subject.innerHTML = "";

    numOfStatement = 0;

    // alert("タイピング文字：" + statement.substr(numOfStatement, 1));

    // キーボードの正解文字を色変え
    typing.active(statement, numOfStatement);
}

// 残り時間をカウントダウン
function startTimer() {

    // timer IDタグにカウントダウンを表示
    timerText = document.getElementById('timer');
    timerText.innerHTML = "残り時間：" + timer;


    // 1秒おきにカウントダウン
    var id = setInterval(function(){
        timer--;
        console.log(timer);

        timerText.innerHTML = "残り時間：" + timer;

        if(timer <= 0){
            if(gameState == PLAYING) {
                // 終了処理
                subject = document.getElementById('subject');
                subject.innerHTML = "---- 終了 ----";
                // alert("終了！！");
                gameState = END;
                clearInterval(id);
            }
        }
    }, 1000);
}

// 打ち込まれた文字の正誤判定
function judgeTyping(chara) {
    var seikai = statement.substr(numOfStatement, 1);

    if(chara == seikai) {
        // 正解の場合
        score = score + 10;

        // 正解文字を追加（input IDタグに追記）
        var input = document.getElementById('input');
        input.innerHTML = input.innerHTML + chara;

        // 次の文字へ
        numOfStatement = numOfStatement + 1;

        if(statement.length != numOfStatement) {
            // 問題文の文字が続く場合
            typing.active(statement, numOfStatement);

        } else {
            score = score + 50;
            // 次の問題へ
            loadSubject();
        }

    } else {
        // 間違いの場合
        // 何もしない
    }

    // スコアを更新
    scoreText = document.getElementById('score');
    scoreText.innerHTML = "スコア：" + score;
}

// JavaScript end
// ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ----
        </script>
    </head>
    <body>
        <!-- パネル表示 -->
        <div id="panel">
            <div id="score">スコア：0</div>
            <div id="timer">残り時間：--</div>
            <div id="capslock">CapsLock --</div>
            <div id="subject">タイピングゲーム</div>
            <div id="input">input</div>
        </div>
        <!-- キーボード表示 -->
        <div id="board">
        </div>
    </body>
    <foot>

    </foot>
</html>
```

実表示させる html 部分が少ないと感じるかもしれませんが、キーボード部分の表示内容（html）は、Typing.jsから呼び出すようにしました。

これは、キーボードの色変更をTyping.jsから操作するため、1ファイル内に情報を格納させておく方が管理の面からしてよいと思ったからです。

___
### ②Common.js
ここでは、指定したテキストファイルを読み込んで配列を返す関数を格納しています。

配列に格納する際の区切りは『改行』としました。

```
// ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ----
// breif : 指定したファイルを読み込んで配列を返す。
// note  : 改行で区切る。
// ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ----
function readTextFileToArray(file) {

    var array;

    var rawFile = new XMLHttpRequest();
    rawFile.open("GET", file, false);
    rawFile.onreadystatechange = function ()
    {
        if(rawFile.readyState === 4)
        {
            if(rawFile.status === 200 || rawFile.status == 0)
            {
                var allText = rawFile.responseText;
                // alert(allText);

                array = allText.split(/[\r\n]+/);
            }
        }
    }
    rawFile.send(null);

    return array;
}
```

___
### ③FlashingText.js
html ページ内のテキストを点滅させるクラスです。

```
// ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ----
// breif : 点滅テキストクラス
// note  :
// ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ----
class FlashingText {

    // ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ----
    // breif : コンストラクタ
    // note  : 引数 id     -> htmlのIDタグ
    //              msg    -> IDタグに表示させるメッセージ
    //              onTime -> 点灯時間[mm秒]
    //              offTime-> 消灯時間[mm秒]
    //              flag   -> 0:表示, 1:非表示    
    // ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ----
    constructor(id, msg, onTime, offTime, flag) {

        this.id = id;
        this.msg = msg;
        this.onTime = onTime;
        this.offTime = offTime;
        this.flag = flag;

        this.isFlashing = 1;    // 1:点滅させる　0:点滅させない
    }

    // setter
    setMsg(msg) { this.msg = msg; }
    setIsFlashing(isFlashing) {
        this.isFlashing = isFlashing;
        // alert("isFlashing:" + this.isFlashing);
    }

    // ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ----
    // breif : テキストを点滅させる
    // note  :
    // ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ----
    flash() {

        // 指定IDのテキストを取得
        var element = document.getElementById(this.id);
        var interval;

    	if(this.flag == 0){

    		// テキストを表示
    		element.innerHTML = this.msg;
    		this.flag = 1;
    		interval = this.onTime;
    	}
    	else{

    		// テキストを非表示
    		element.innerHTML = "";
    		this.flag = 0;
    		interval = this.offTime;
    	}

        if(this.isFlashing == 1 || (this.isFlashing == 0 && this.flag == 0)) {
            var self = this;
        	setTimeout(function() {self.flash();}, interval);
        }
    }
}
```

___
### ④Typing.js
ポイントは、キーボード番号に対する文字列を辞書型で定義していることです。
さらに、CapsLockの状態で小文字/大文字状態が変わってしまうため、CapsLockのON/OFFそれぞれを別変数で定義しているのです。

また、このクラスでは対応する文字列のキーボードを色付けする関数が含まれています（setactive メソッド）。


文字列を引数として読み込むことで、htmlのタグ内classをもとに対応するキーボードを判断しています。
そのため、このクラスは表示させる html（キーボードを表示させる部分）と連携させる必要があるのですが、
別ファイルで管理するとメンテ面でもよろしくないため、
このクラスからキーボードのhtmlを表示させるように工夫しました。

```
// ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ----
// breif : タイピングクラス
// note  :
// ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ----

var codelist = {8:["BackSpase","BackSpace"], 9:["Tab","Tab"], 13:["Enter","Enter"], 16:["Shift",""], 17:["Ctrl",""]
                , 20:["","caps"], 32:[" "," "], 45:["-","="], 48:["0",""], 49:["1","!"], 50:["2",'"'], 51:["3","#"]
                , 52:["4","$"], 53:["5","%"], 54:["6","&"], 55:["7","'"], 56:["8","("], 57:["9",")"], 58:[":","*"]
                , 59:[";","+"], 61:[";","+"], 64:["@","`"], 65:["a","A"], 66:["b","B"], 67:["c","C"], 68:["d","D"]
                , 69:["e","E"], 70:["f","F"], 71:["g","G"], 72:["h","H"], 73:["i","I"], 74:["j","J"], 75:["k","K"]
                , 76:["l","L"], 77:["m","M"], 78:["n","N"], 79:["o","O"], 80:["p","P"], 81:["q","Q"], 82:["r","R"]
                , 83:["s","S"], 84:["t","T"], 85:["u","U"], 86:["v","V"], 87:["w","W"], 88:["x","X"], 89:["y","Y"]
                , 90:["z","Z"], 92:["\\","_"], 96:["0",""], 97:["1",""], 98:["2",""], 99:["3",""], 100:["4",""], 101:["5",""]
                , 102:["6",""], 103:["7",""], 104:["8",""], 105:["9",""], 107:[";","+"], 109:["-","="], 160:["^","~"]
                , 173:["-","="], 186:[":","*"], 187:[";","+"], 188:[",","＜"], 189:["-","="], 190:[".","＞"], 191:["/","?"]
                , 192:["@","`"], 219:["[","{"], 220:["\\","_"], 221:["]","}"], 222:["^","~"], 226:["\\","_"], 222:["^","~"]
                , 240:["英数",""], 244:["半/全",""]};

var capslist = {8:["BackSpase","BackSpace"], 9:["Tab","Tab"], 13:["Enter","Enter"], 16:["Shift",""], 17:["Ctrl",""]
                , 20:["","caps"], 32:[" "," "], 45:["-","="], 48:["0",""], 49:["1","!"], 50:["2",'"'], 51:["3","#"]
                , 52:["4","$"], 53:["5","%"], 54:["6","&"], 55:["7","'"], 56:["8","("], 57:["9",")"], 58:[":","*"]
                , 59:[";","+"], 61:[";","+"], 64:["@","`"], 65:["A","a"], 66:["B","b"], 67:["C","c"], 68:["D","d"]
                , 69:["E","e"], 70:["F","f"], 71:["G","g"], 72:["H","h"], 73:["I","i"], 74:["J","j"], 75:["K","k"]
                , 76:["L","l"], 77:["M","m"], 78:["N","n"], 79:["O","o"], 80:["P","p"], 81:["Q","q"], 82:["R","r"]
                , 83:["S","s"], 84:["T","t"], 85:["U","u"], 86:["V","v"], 87:["W","w"], 88:["X","x"], 89:["Y","y"]
                , 90:["Z","z"], 92:["\\","_"], 96:["0",""], 97:["1",""], 98:["2",""], 99:["3",""], 100:["4",""], 101:["5",""]
                , 102:["6",""], 103:["7",""], 104:["8",""], 105:["9",""], 107:[";","+"], 109:["-","="], 160:["^","~"]
                , 173:["-","="], 186:[":","*"], 187:[";","+"], 188:[",","＜"], 189:["-","="], 190:[".","＞"], 191:["/","?"]
                , 192:["@","`"], 219:["[","{"], 220:["\\","_"], 221:["]","}"], 222:["^","~"], 226:["\\","_"], 222:["^","~"]
                , 240:["英数",""], 244:["半/全",""]};

var leftcode = {"!":"", '"':"", "#":"", "$":"", "%":"", "&":"", "Q":"", "W":"", "E":"", "R":"", "T":"", "A":"", "S":""
                , "D":"", "F":"", "G":"", "Z":"", "X":"", "C":"", "V":"", "B":""};

var leftcaps = {"!":"", '"':"", "#":"", "$":"", "%":"", "&":"", "q":"", "w":"", "e":"", "r":"", "t":"", "a":"", "s":""
                , "d":"", "f":"", "g":"", "z":"", "x":"", "c":"", "v":"", "b":""};

var eachactive = {"\\":"220", "|":"220", "_":"226", ";":"187", "+":"187", ":":"186", "*":"186"};

var keyboardHTML = ''
            + '<table id="keyboard">'
            + '<tr class="tr0">'
            + '<td colspan="30" class="col30">'
            + '<span class="col key code027">Esc</span>'
            + '<span class="col key">F1</span>'
            + '<span class="col key">F2</span>'
            + '<span class="col key">F3</span>'
            + '<span class="col key">F4</span>'
            + '<span class="col key">F5</span>'
            + '<span class="col key">F6</span>'
            + '<span class="col key">F7</span>'
            + '<span class="col key">F8</span>'
            + '<span class="col key">F9</span>'
            + '<span class="col key">F10</span>'
            + '<span class="col key">F11</span>'
            + '<span class="col key">F12</span>'
            + '<span class="col key">Num</span>'
            + '<span class="col key">Prt</span>'
            + '<span class="col key">Ins</span>'
            + '<span class="col key">Del</span>'
            + '</td>'
            + '</tr>'
            + '<tr class="tr1">'
            + '<td colspan="1" class="col1 key"><div class="table code244">半</div></td>'
            + '<td colspan="2" class="col2 key"><div class="table code049">1<span class="subkey">!</span></div></td>'
            + '<td colspan="2" class="col2 key"><div class="table code050">2<span class="subkey">”</span></div></td>'
            + '<td colspan="2" class="col2 key"><div class="table code051">3<span class="subkey">#</span></div></td>'
            + '<td colspan="2" class="col2 key"><div class="table code052">4<span class="subkey">$</span></div></td>'
            + '<td colspan="2" class="col2 key"><div class="table code053">5<span class="subkey">%</span></div></td>'
            + '<td colspan="2" class="col2 key"><div class="table code054">6<span class="subkey">&</span></div></td>'
            + '<td colspan="2" class="col2 key"><div class="table code055">7<span class="subkey">’</span></div></td>'
            + '<td colspan="2" class="col2 key"><div class="table code056">8<span class="subkey">（</span></div></td>'
            + '<td colspan="2" class="col2 key"><div class="table code057">9<span class="subkey">）</span></div></td>'
            + '<td colspan="2" class="col2 key"><div class="table code048">0</div></td>'
            + '<td colspan="2" class="col2 key"><div class="table code189 code109 code173 code045">-<span class="subkey">=</span></div></td>'
            + '<td colspan="2" class="col2 key"><div class="table code222 code160">^<span class="subkey">~</span></div></td>'
            + '<td colspan="2" class="col2 key"><div class="table code220">\<span class="subkey">|</span></div></td>'
            + '<td colspan="3" class="col3 key"><div class="table code008">Back</div></td>'
            + '</tr>'
            + '<tr class="tr2">'
            + '<td colspan="2" class="col2 key"><div class="table code009">Tab</div></td>'
            + '<td colspan="2" class="col2 key"><div class="table code081">Q</div></td>'
            + '<td colspan="2" class="col2 key"><div class="table code087">W</div></td>'
            + '<td colspan="2" class="col2 key"><div class="table code069">E</div></td>'
            + '<td colspan="2" class="col2 key"><div class="table code082">R</div></td>'
            + '<td colspan="2" class="col2 key"><div class="table code084">T</div></td>'
            + '<td colspan="2" class="col2 key"><div class="table code089">Y</div></td>'
            + '<td colspan="2" class="col2 key"><div class="table code085">U</div></td>'
            + '<td colspan="2" class="col2 key"><div class="table code073">I</div></td>'
            + '<td colspan="2" class="col2 key"><div class="table code079">O</div></td>'
            + '<td colspan="2" class="col2 key"><div class="table code080">P</div></td>'
            + '<td colspan="2" class="col2 key"><div class="table code192 code064">@<span class="subkey">`</span></div></td>'
            + '<td colspan="2" class="col2 key"><div class="table code219">[<span class="subkey">{</span></div></td>'
            + '<td colspan="1" class="col1 key"></td>'
            + '<td colspan="3" rowspan="2" class="col3 key"><div class="table code013 row2">Enter</div></td>'
            + '</tr>'
            + '<tr class="tr3">'
            + '<td colspan="3" class="col3 key"><div class="table code240 code020">英数</div></td>'
            + '<td colspan="2" class="col2 key"><div class="table code065">A</div></td>'
            + '<td colspan="2" class="col2 key"><div class="table code083">S</div></td>'
            + '<td colspan="2" class="col2 key"><div class="table code068">D</div></td>'
            + '<td colspan="2" class="col2 key"><div class="table code070">F</div></td>'
            + '<td colspan="2" class="col2 key"><div class="table code071">G</div></td>'
            + '<td colspan="2" class="col2 key"><div class="table code072">H</div></td>'
            + '<td colspan="2" class="col2 key"><div class="table code074">J</div></td>'
            + '<td colspan="2" class="col2 key"><div class="table code075">K</div></td>'
            + '<td colspan="2" class="col2 key"><div class="table code076">L</div></td>'
            + '<td colspan="2" class="col2 key"><div class="table code187 code107 code059">;<span class="subkey">+</span></div></td>'
            + '<td colspan="2" class="col2 key"><div class="table code186 code058">:<span class="subkey">*</span></div></td>'
            + '<td colspan="2" class="col2 key"><div class="table code221">]<span class="subkey">}</span></div></td>'
            + '</tr>'
            + '<tr class="tr4">'
            + '<td colspan="4" class="col4 key"><div class="table code016 code020">Shift</div></td>'
            + '<td colspan="2" class="col2 key"><div class="table code090">Z</div></td>'
            + '<td colspan="2" class="col2 key"><div class="table code088">X</div></td>'
            + '<td colspan="2" class="col2 key"><div class="table code067">C</div></td>'
            + '<td colspan="2" class="col2 key"><div class="table code086">V</div></td>'
            + '<td colspan="2" class="col2 key"><div class="table code066">B</div></td>'
            + '<td colspan="2" class="col2 key"><div class="table code078">N</div></td>'
            + '<td colspan="2" class="col2 key"><div class="table code077">M</div></td>'
            + '<td colspan="2" class="col2 key"><div class="table code188">,<span class="subkey"><</span></div></td>'
            + '<td colspan="2" class="col2 key"><div class="table code190">.<span class="subkey">></span></div></td>'
            + '<td colspan="2" class="col2 key"><div class="table code191">/<span class="subkey">?</span></div></td>'
            + '<td colspan="2" class="col2 key"><div class="table code226">\<span class="subkey">_</span></div></td>'
            + '<td colspan="2" class="col2 key"><div class="table">↑</div></td>'
            + '<td colspan="2" class="col2 key"><div class="table code016">Shift</div></td>'
            + '</tr>'
            + '<tr class="tr5">'
            + '<td colspan="2" class="col2 key"><div class="table">Ctrl</div></td>'
            + '<td colspan="2" class="col2 key"><div class="table">Fn</div></td>'
            + '<td colspan="2" class="col2 key"><div class="table">Win</div></td>'
            + '<td colspan="2" class="col2 key"><div class="table">Alt</div></td>'
            + '<td colspan="2" class="col2 key"><div class="table">無</div></td>'
            + '<td colspan="5" class="col5 key"><div class="table code032"></div></td>'
            + '<td colspan="2" class="col2 key"><div class="table">変</div></td>'
            + '<td colspan="2" class="col2 key"><div class="table">か</div></td>'
            + '<td colspan="2" class="col2 key"><div class="table">Alt</div></td>'
            + '<td colspan="1" class="col1 key"><div class="table">App</div></td>'
            + '<td colspan="2" class="col2 key"><div class="table">Ctrl</div></td>'
            + '<td colspan="2" class="col2 key"><div class="table">←</div></td>'
            + '<td colspan="2" class="col2 key"><div class="table">↓</div></td>'
            + '<td colspan="2" class="col2 key"><div class="table">→</div></td>'
            + '</tr>'
            + '</table>';

class Typing {

    // ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ----
    // breif : コンストラクタ
    // note  :
    // ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ----
    constructor() {

        this.capslock = ""
    }

    // ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ----
    // breif : キーボードのhtmlを挿入
    // note  :
    // ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ----
    insertKeyboard(id) {
        document.getElementById(id).innerHTML = keyboardHTML;
    }

    // ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ----
    // breif : 入力した文字を取得
    // note  :
    // ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ----
    checkWord(event) {

        var keycode, shiftcode, chara

		keycode = event.keyCode;
        shiftcode = event.shiftKey;

        if(this.capslock==1){
            chara = this.getcapschar(keycode,shiftcode);  
        }else{
            chara = this.getchar(keycode,shiftcode);
        }

        return chara;
    }

    // ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ----
    // breif : CapsLockを判定して取得
    // note  :
    // ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ----
    checkCapsLock(onKeyUpEvent) {

        if(onKeyUpEvent.getModifierState("CapsLock") == false) {
            console.log("CapsLock OFF");
            this.capslock = 0;
        }
        else {
            console.log("CapsLock ON");
            this.capslock = 1;
        }

        return this.capslock;
    }

    // ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ----
    // breif : キーコードとシフト押下状態から文字を取得
    // note  : CapsLock OFF
    // ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ----
    getchar(keycode, shiftcode){
        var chara;

        if(keycode in codelist){
            if(shiftcode){
                   chara = codelist[keycode][1];
            }else{
                   chara = codelist[keycode][0];
            }
        }else{
            chara = "";
        }
        return chara;
    }

    // ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ----
    // breif : キーコードとシフト押下状態から文字を取得
    // note  : CapsLock ON
    // ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ----
    getcapschar(keycode, shiftcode){
        var chara;

        if(keycode in capslist){
            if(shiftcode){
                   chara = capslist[keycode][1];
            }else{
                   chara = capslist[keycode][0];
            }
        }else{
            chara = "";
        }
        return chara;
    }

    // ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ----
    // breif : キーボードの色状態を更新
    // note  :
    // ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ----
    active(statement, numOfStatement){

        var ichi = numOfStatement;          // 問題文の位置（何番目）
        var mondai = statement;             // 問題文
        var mondailen = statement.length;   // 問題文の長さ

        var left, list;

        this.resetactive();
        if(this.capslock==1){
            // CapsLock:ON
            left=leftcaps;
            list=capslist;
        }else{
            // CapsLock:OFF
            left=leftcode;
            list=codelist;
        }
        if(ichi!=mondailen){
            for(var i in list){
                if(list[i][0]==mondai.charAt(ichi)){
                    if(mondai.charAt(ichi) in eachactive){
                            this.setactive("code"+('00'+eachactive[mondai.charAt(ichi)]).slice(-3),0);
                    }else{
                            this.setactive("code"+('00'+i).slice(-3),0);
                    }
                }else if(list[i][1]==mondai.charAt(ichi)){
                    if(mondai.charAt(ichi) == "_"){
                            this.setactive("code226",0);
                    }else if(mondai.charAt(ichi) == "|"){
                            this.setactive("code220",0);
                    }else{
                            this.setactive("code"+('00'+i).slice(-3),0);
                    }
                    if(list[i][1] in left){
                            this.setactive("code016",1);
                    }else{
                            this.setactive("code016",0);
                    }
                }
            }
        }
    }

    // ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ----
    // breif : 対象のキーを色付けする。
    // note  : targetClass -> 色付けするクラス
    //         targetNo    -> 色付けする対象が複数ある場合の何番目かどうか
    // ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ----
    setactive(targetClass,targetNo){
        var allElements;
        var elementname;

        var foundElements = new Array();
        if (document.all){
            allElements = document.all;
        }else {
            allElements = document.getElementsByTagName("*");
        }
        var elementslen;
        var j=0;
        for(var i=0,elementslen=allElements.length;i<elementslen;i++){
            elementname = allElements[i].className;
            if(elementname.indexOf(targetClass,0) > -1) {
                foundElements[j] = allElements[i];
                j++;
            }
        }
        foundElements[targetNo].style.background="#00ffff";
    }

    // ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ----
    // breif : キーの色付けを解除する。
    // note  :
    // ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ----
    resetactive(){
        var allElements;
        var elementname;

        if (document.all){
            allElements = document.all;
        }else {
            allElements = document.getElementsByTagName("*");
        }
        var elementslen;
        for(var i=0,elementslen=allElements.length;i<elementslen;i++){
            elementname = allElements[i].className;
            if(elementname.indexOf("table",0) > -1) {
                allElements[i].style.background="#ffffff";
            }
        }
    }
}
```

以上です。
