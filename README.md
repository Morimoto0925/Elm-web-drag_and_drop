# Elm-web-drag_and_drop
Elm(プログラミング言語)を使ったwebブラウザ上でのドラッグ&ドロップについて

**Case**文でそれぞれ別々に処理し、**model.drag**と書くことで**Move**の場合の処理でも使えるようになっている

```
module Main exposing(..)
import Browser
import Html exposing (..)
import Html.Attributes exposing (..)
import Html.Events exposing (..)
import Json.Decode exposing (..)

main = Browser.sandbox
    { init=init
    , view=view
    , update=update}

type alias Model = {x:Int, y:Int, drag:Bool}--変数の作成

init : Model
init = {x = 0, y = 0, drag = False}--変数の初期化

type Msg = Down
         | Up
         | Move Int Int

update : Msg -> Model -> Model
update msg model =
  case msg of
    Down -> {model | drag = True}--マウスダウンを受け取ったらdragフラグをTrueに
    Up -> {model | drag = False}--マウスアップを受け取ったらdragフラグをFalseに
    Move x y ->
      if model.drag then --ドラッグ中ならoffset座標を元に表示する場所を変える
          {model | x = x - 50, y = y - 50}
      else model --ドラッグ中ではないなら座標を維持する

view : Model -> Html Msg
view model =
  --親要素
  div[ style "position" "relative"
    , style "background" ""
    , style "height" "100%"
    , style "width" "100%"
    , onMouseMove Move
    , onMouseUp Up
  ]
  [ div --四角形
    [ style "position" "absolute"
    , style "background" "red"
    , style "top" (String.fromInt model.y ++ "px")
    , style "left" (String.fromInt model.x ++ "px")
    , style "height" "100px"
    , style "width" "100px"
    , onMouseDown Down --四角形の中でのみマウスダウンを呼び出せるようにする
    ]
    []
  ]

--マウスの座標を返す関数
onMouseMove : (Int -> Int -> msg) -> Html.Attribute msg
onMouseMove f = --client座標を返す
  on "mousemove" (map2 f (field "clientX" int) (field "clientY" int))
```

[[Ellieで動かす]](https://ellie-app.com/bPW5YTrRhvMa1)

このコードでは、四角形の中でクリックした時のみ図形が動き、常に図形の中心にカーソルが来るように動作する

## クリックした場所を保存(?)できるようにする
動作としては、図形をクリックした際のカーソルの位置を記憶して、カーソルと図形の相対位置が変わらないようにしたい

そこで**offset座標**を使う
javascriptのマウスイベントで使われる座標系には、client、screen、**offset**、pageなどがある
それぞれの座標が画面、ウィンドウ、要素内のどこを参照しているのかは[[マウスイベントで取得されるカーソル座標パラメータの整理]](https://qiita.com/yukiB/items/31a9e9e600dfb1f34f76)ここが分かりやすい

・**onMouseOffset**関数を作成して、四角形のoffsetXYを保存
・**Move**ではカーソルとの相対位置をoffsetを使って求めるようにする

## 微妙にずれる
offsetを使っても、ドラッグした際にほんの少しだけ図形が右下にズレてしまう
理由がわからないが常に右下にズレるだけなので、プログラム側で直接座標を左上にズラして対応する

## とりあえず完成

```
module Main exposing(..)
import Browser
import Html exposing (..)
import Html.Attributes exposing (..)
import Html.Events exposing (..)
import Json.Decode exposing (..)

main = Browser.sandbox
    { init=init
    , view=view
    , update=update}

type alias Model = {x:Int, y:Int, x_offset:Int, y_offset:Int, drag:Bool}--変数の作成

init : Model
init = {x = 0, y = 0, x_offset = 0, y_offset = 0, drag = False}--変数の初期化

type Msg = Down
         | Up
         | Move Int Int
         | Offset Int Int

update : Msg -> Model -> Model
update msg model =
  case msg of
    Down -> {model | drag = True}--マウスダウンを受け取ったらdragフラグをTrueに
    Up -> {model | drag = False}--マウスアップを受け取ったらdragフラグをFalseに
    Move x y ->
      if model.drag then --ドラッグ中ならoffset座標を元に表示する場所を変える
          {model | x = x - model.x_offset, y = y - model.y_offset}
      else model --ドラッグ中ではないなら座標を維持する
    Offset x y ->
      if (model.drag==False) then 
          {model | x_offset = x+6, y_offset = y+6}--ドラッグ時に少しだけずれてしまうので微調整(+6)
      else model

view : Model -> Html Msg
view model =
  --親要素
  div[ style "position" "relative"
    , style "background" ""
    , style "height" "100%"
    , style "width" "100%"
    , onMouseMove Move
    , onMouseUp Up
  ]
  [ div --四角形
    [ style "position" "absolute"
    , style "background" "red"
    , style "top" (String.fromInt model.y ++ "px")
    , style "left" (String.fromInt model.x ++ "px")
    , style "height" "100px"
    , style "width" "100px"
    , onMouseOffset Offset
    , onMouseDown Down --四角形の中でのみマウスダウンを呼び出せるようにする
    ]
    []
  ]

--マウスの座標を返す関数
onMouseMove : (Int -> Int -> msg) -> Html.Attribute msg
onMouseMove f = --client座標を返す
  on "mousemove" (map2 f (field "clientX" int) (field "clientY" int))
 
onMouseOffset : (Int -> Int -> msg) -> Html.Attribute msg
onMouseOffset f = --offset座標を返す
  on "mousemove" (map2 f (field "offsetX" int) (field "offsetY" int))
```
:::

[[Ellieで動かす]](https://ellie-app.com/bPNvvrw5HtWa1)

マウスを速く動かすと図形が付いてこないことがある
動作が重い？

## マウスを高速で動かしてもついてくるようにする
色々弄って試してみたところ、親要素の **height** と **width** を **〇〇%** ではなく**〇〇vh(vw)** で指定すると滑らかに動くようになった（なんで？）

%指定とviewport基準の指定との違いについては[[知らないと損！CSSのvh/vwの使いこなしでレスポンシブなサイト制作が捗る]](https://www.webprofessional.jp/css-viewport-units-quick-start/)に書いてある

| 指定方法 | 基準になるもの    |
|:-----: |:---------------:|
| %      | 親要素の幅を100分割 |
| vh,vw  | viewportの寸法を100分割 |

でも動作が重くなる理由はわからない…
再び弄ってみると、**width**は%指定にしても問題なく動くことがわかった
**height**を%指定した時だけ重くなるらしい

[[[CSS]ビューポート(vw, vh)とパーセント(%)、レスポンシブに適した単位の賢い使い分け方法]](https://coliss.com/articles/build-websites/operation/css/viewport-vs-percentage-units-by-ire.html)より、
> 要素をページの高さいっぱいにする時は、パーセントよりビューポートの単位「vh」が適しています。
> 
> パーセントで定義された要素のサイズは親要素に基づくため、親要素が高さに何らかの影響を受けている必要があります。これを実現するには、html要素を親要素として要素を配置するか、なんらかのハックをしなければできないでしょう
> 
> しかし、「vh」を使用すると簡単にできます。

とのこと
幅(width)に関してはむしろ **%指定** の方が適しているらしい

heightを%指定で参照すると、背景の親要素を作っていない為、恐らく無の空間を参照してしまい範囲0になっていると思われる
なので、図形からカーソルが離れたとたんに動かなくなってしまう

修正箇所
```javascript
  --親要素
  div[ style "position" "relative"
    , style "background" ""
    , style "height" "100vh"
    , style "width" "100%"
    , onMouseMove Move
    , onMouseUp Up
  ]
```
