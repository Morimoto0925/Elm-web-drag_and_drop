# Elm-web-drag_and_drop
Elm(プログラミング言語)を使ったwebブラウザ上でのドラッグ&ドロップについて

## 環境
Windows10 Pro
Elmのバージョン:0.19
オンライン開発環境:[Ellie](https://ellie-app.com/new)

## Elmとは
Elm作者による[公式ガイド](https://guide.elm-lang.jp/)より
ElmはJavaScriptにコンパイルできる関数型プログラミング言語のこと

## コード

```javascript
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
    , style "height" "100vh"
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

[[Ellieで動かす]](https://ellie-app.com/bXc2njdSd92a1)

## 説明
### main
The Elm Architectureより
>Model — アプリケーションの状態

>View — 状態を HTML に変換する方法

>Update — メッセージを使って状態を更新する方法

### update
```javascript
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
```
viewから渡されたメッセージに対応した処理を行う
**Offset**では、図形の描画点からマウスまでの相対距離(図形内のどこをクリックしたか)を保存しておく
これによって、図形の端を掴んだまま移動させたり、中央を掴んだまま移動させることが出来る
それでもドラッグ時に若干ずれるので、xy座標+6で微調整している(環境差あるかも)

### view
```javascript
view : Model -> Html Msg
view model =
  --親要素
  div[ style "position" "relative"
    , style "background" ""
    , style "height" "100vh"
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
```
赤い四角形を表示させる部分と、その親要素となる背景の設定を行っている
また、それぞれの要素内ではonMouseMoveなどの関数を呼び出している

親要素は **height** は「vh」で指定し、 **width** は「%」で指定すると動作が滑らかになる

%指定とviewport基準の指定との違いについては[[知らないと損！CSSのvh/vwの使いこなしでレスポンシブなサイト制作が捗る]](https://www.webprofessional.jp/css-viewport-units-quick-start/)に書いてある

| 指定方法 | 基準になるもの    |
|:-----: |:---------------:|
| %      | 親要素の幅を100分割 |
| vh,vw  | viewportの寸法を100分割 |

#### ビューポートの解説
[[[CSS]ビューポート(vw, vh)とパーセント(%)、レスポンシブに適した単位の賢い使い分け方法]](https://coliss.com/articles/build-websites/operation/css/viewport-vs-percentage-units-by-ire.html)より、
> 要素をページの高さいっぱいにする時は、パーセントよりビューポートの単位「vh」が適しています。
> 
> パーセントで定義された要素のサイズは親要素に基づくため、親要素が高さに何らかの影響を受けている必要があります。これを実現するには、html要素を親要素として要素を配置するか、なんらかのハックをしなければできないでしょう
> 
> しかし、「vh」を使用すると簡単にできます。

とのこと
幅(width)に関してはむしろ **%指定** の方が適しているらしい

heightを%指定で参照すると、背景の親要素を作っていない為、恐らく大きさの指定されていない空間を参照してしまい範囲0になっていると思われる
なので、赤い四角からカーソルが離れると動作しなくなる

### onMouseあれこれ
```javascript
onMouseMove : (Int -> Int -> msg) -> Html.Attribute msg
onMouseMove f = --client座標を返す
  on "mousemove" (map2 f (field "clientX" int) (field "clientY" int))
 
onMouseOffset : (Int -> Int -> msg) -> Html.Attribute msg
onMouseOffset f = --offset座標を返す
  on "mousemove" (map2 f (field "offsetX" int) (field "offsetY" int))
```

マウスのclient座標かoffset座標を返す関数(関数を作らずにdiv内に直接書くこともできる)
