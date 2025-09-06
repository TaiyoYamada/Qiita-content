# React NativeはiOS 26.0で動かない？

## はじめに

React Native でアプリを開発する中で、シミュレータのOSバージョンによって動作が異なる現象を確認しました。なお、執筆時点（2025年9月6日現在）において、iOS 26.0 はまだベータ版であることに留意してください。


## 開発環境

* macOS: 26.0 beta
* Xcode: 16.4
* React Native: 0.81
* Expo SDK 53
* iOS シミュレータ: 26.0 / 18.6


## 検証結果

* **iOS 26.0** → アプリ起動直後にクラッシュ。
* **iOS 18.6** → 問題なく起動し、正常に動作。


## 考察

Apple公式から明確に「JITを廃止した」とのアナウンスは現時点ではありません。ただし、開発者コミュニティでは **iOS 26 でレガシーJITランタイムが制限され、非公式のJavaScriptCore統合も利用できなくなっている** という報告が複数出ています。これが今回の挙動に影響している可能性が高いと考えられます。

参考情報：

* [iOS 26でReact Nativeが動かない問題について](https://blog.stackademic.com/ios-26-breaks-your-react-native-app-heres-what-to-do-27b617770c81)
* [Tools4Hackの記事](https://tools4hack.santalab.me/news-apple-completely-killed-enable-jit-solution-in-ios26.html)
* [Redditでのユーザー報告](https://www.reddit.com/r/EmulationOniOS/comments/1l7eit9/jit_no_longer_work_on_ios_26/)

## 回避策

* **安定性を重視する場合**: iOS 18.x シミュレータを利用する。
* **最新環境を試す場合**: 非公式ツールを利用する方法もあるが、推奨はできない。


## まとめ

* React Nativeは**iOS 26.0シミュレータでは動作しなかった**。
* **iOS 18.6では正常に動作**した。
* 背景には**iOS 26でのJIT制限**が関わっている可能性が高い。
* 現状では**iOS 18.xシミュレータを利用するのが無難**。

iOS 26はまだベータ版なので、正式リリースやReact Nativeのアップデート次第で状況は変わるかもしれません。今後の動向を注視しながら、開発環境を柔軟に選んでいくのがよさそうです。
