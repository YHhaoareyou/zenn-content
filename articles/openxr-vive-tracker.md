---
title: "OpenXRを使ったUnity XRプロジェクトでViveトラッカーを使う"
emoji: "🐯"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Unity", "OpenXR", "HTC Vive Tracker", "VR", "XR"]
published: true
---

# はじめに

[この動画](https://youtu.be/Tw5LbNLW9Q4)で紹介された手順を、自分で実際に一通り従って設定していて、結果を確かめました。せっかくなので、自分が設定しながら撮っていたスクショを添付し、この日本語の記事にさせていただきました。

ちなみに[こちらのこりんさん](https://framesynthesis.jp/tech/unity/xr/#viveトラッカーを使用するには)も同じ内容を大ざっぱに共有されています。

## なぜSteamVR PluginではなくOpenXR Pluginを使うか

OpenXRはSteamVRを含むさまざまなプラットフォームに対応しており、これからVRを含むXRの開発の標準規格になるという見込みです。

ViveトラッカーをUnityプロジェクトで使用するには、今はまだSteamVRを介する必要がありますが、Unity上で使うPluginについては、SteamVR Pluginを諦めて先にOpenXR Pluginに移行した方が、今後プロジェクトの拡張や他のプラットフォームへの対応が楽になるのではないかと思います。

\* もう一つの原因は普通に、自分がSteamVRのInput Binding（下の画像）が少し使いづらく、しかもなぜか結構効かなくなって再設定しないといけないことが多かったからです...今回紹介させていただく方法の方が便利だと思いますし、設定が効かなくなったりすることもありません。

![](/images/openxr-vive-tracker/steamvr-input-binding.jpg)

# 事前準備

- **OpenXR Plugin**と**XR Interaction Toolkit**がすでに入っているUnityプロジェクト
  - OpenXR PluginとXR Interaction Toolkitの導入については[こりんさんの記事](https://framesynthesis.jp/tech/unity/xr/)を参考していただければと思います。
- SteamVR対応のVRデバイスがPCに繋がっている
  - 今回はMeta Quest 2を使用するため、Oculus LinkでPCに繋げています。
- SteamVRが起動され、OpenXRのランタイムとして設定されている
  - ![](/images/openxr-vive-tracker/preparation-01.png)
- Viveトラッカーがペアリングされている

# ViveトラッカーのInteraction Profileを追加

[こちらの書き込み](https://forum.unity.com/threads/openxr-and-openvr-together.1113136/#post-7803057)に貼ってある `HTCViveTrackerProfile.cs` をダウンロードし、Unityプロジェクトに入れましょう。「はじめに」で言及した動画の概要欄にもリンクが貼ってあります。

![HTCViveTrackerProfileが貼ってある書き込み](/images/openxr-vive-tracker/profile-script-01.png)

そしたら、 `Edit > Project Settings > XR Plug-in Management > OpenXR > Interaction Profiles` の「＋」を押したら `HTC Vive Tracker Profile` が選択肢一覧に表示されると思います。それを選択してInteraction Profilesに加えましょう。

![HTC Vive Tracker ProfileをOpenXRのInteraction Profilesに追加する](/images/openxr-vive-tracker/profile-script-02.png)

# トラッカーの役割設定

SteamVRの設定画面を開いて、`コントローラ > Viveトラッカーの管理`で、今ペアリングされているトラッカーに役割をつけましょう（例えば「腰」として設定します）。

![トラッカーに役割をつける](/images/openxr-vive-tracker/tracker-role-01.png =500x)

# トラッカーの情報が読まれているか確認

Playボタンを押しましょう。そして下のスクショのようにメニューを辿って`Input Debugger`を開きましょう。

![Input Debuggerを開く](/images/openxr-vive-tracker/input-debugger-01.png =300x100)

下のスクショのように、全役割のトラッカーが表示されますが、腰のトラッカーのところをダブルクリックすると、位置と回転が読まれているのを確認できます。

![Input Debugger画面](/images/openxr-vive-tracker/input-debugger-02.png =300x100)

もし読まれていないようであれば、Unityを再起動してみてください。

# Input Actionsでトラッカーの情報を使えるようにする

## Input Actions Assetを新規作成

![Input Actions Assetを作成](/images/openxr-vive-tracker/input-actions-01.png =300x100)

トラッカーの役割ごとInput Actions Assetを1つずつ作成してもいいし、全役割を1つのInput Actions Assetにまとめてもいいと思います。

今回は腰のトラッカー1つしかないため、普通にInput Actions Assetを1つ作成します。

## Actionsを作成

そしたらAction Mapを作成し、Actionを2つ作りましょう：**Position**と**Rotation**。

![トラッカーのPositionとRotationに対応するActionsを作成](/images/openxr-vive-tracker/input-actions-02.png)

## Position

Positionを選択し、パネルの右側で以下のように設定しましょう：

```
Action Type: Value
Control Type: Vector3
```

![](/images/openxr-vive-tracker/input-actions-03.png)

下の<No Binding>のところをクリックし、パネルの右側の`Binding > Path`をクリックし、`XR Tracker > HTC Vive Tracker (OpenXR) > HTC Vive Tracker (OpenXR) (役割) > devicePosition`を選択しましょう。

![](/images/openxr-vive-tracker/input-actions-04.png)
![](/images/openxr-vive-tracker/input-actions-05.png)
![](/images/openxr-vive-tracker/input-actions-06.png =300x100)
![](/images/openxr-vive-tracker/input-actions-07.png)

## Rotation

Rotationを選択し、パネルの右側で以下のように設定しましょう：

```
Action Type: Value
Control Type: Quaternion
```

下の<No Binding>のところをクリックし、パネルの右側の`Binding > Path`をクリックし、`XR Tracker > HTC Vive Tracker (OpenXR) > HTC Vive Tracker (OpenXR) (役割) > deviceRotation`を選択しましょう。

![](/images/openxr-vive-tracker/input-actions-08.png)

最後にセーブし忘れずに！

## Input Action ManagerにActions Asset追加する

XR Interaction Toolkitを使って既に開発が進んだプロジェクトだと、シーンの中にInput Action Managerが既に作られたと思いますが、作ったばかりのプロジェクトであれば、空のGameObjectを作成し、コンポーネント`Input Action Manager`を付与し、`Action Assets`に先程作成したトラッカー用のActions Assetを追加しましょう

![](/images/openxr-vive-tracker/input-actions-09.png)

これでトラッカーの位置・回転を、UnityのInput Actionで扱えるようになりました！

スクリプトを書いて自由に利用してもいいし、以下のように`Tracked Pose Driver`を使い、単純にシーンの中のGameObjectの位置・回転をトラッカーと同期させることもできます。

# GameObjectとトラッカーの同期

何かのGameObjectを作って（例えばCube）、コンポーネント`Tracked Pose Driver`を付与しましょう。

![](/images/openxr-vive-tracker/track-pose-01.png)

**Position Input**と**Rotation Input**のそれぞれの**Use Reference**チェックを入れて、**Reference**をそれぞれ先程作ったPositionとRotationのActionに設定しましょう。

![](/images/openxr-vive-tracker/track-pose-02.png)

# 動かしてみよう

Playボタンを押して、トラッカーを手に持って色々動かし、Cubeが同様に動くか確かめましょう！

# おわりに

今回はOpenXRを使ったUnity XRプロジェクトでViveトラッカーの情報を扱う方法について紹介させていただきました。

今はViveトラッカーを使うにはまだSteamVRを介する必要がありますが、今後SteamVRを介さなくて済むようになるのかなと勝手に期待したりています。

現状だと、OpenXRを使って何かクロスプラットフォームのXRプロジェクトを開発するとき、「SteamVRを使用する場合だけのオプション機能」としてViveトラッカーを使って、他のプラットフォームだとViveトラッカーを使わずに済む、というようなコンテンツは作れるのではないかと思います。

次の記事はいつになるか分かりませんが、[UltimateXR](https://www.ultimatexr.io)についての記事になると思います。それではまた！