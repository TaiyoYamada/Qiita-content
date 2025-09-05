# Mac お掃除コマンド集（Xcode開発者向け）

---

Mac のストレージを圧迫しがちなキャッシュや古いシミュレーターを、ターミナルからお掃除するためのコマンドまとめです。**自分用メモ**ですが、同じように Xcode を使う方にも役立つと思います。


## 1. キャッシュ削除

```bash
rm -rf ~/Library/Caches/*
```

ユーザーキャッシュを削除（アプリ再起動で再生成されるので安心）。
※ 一部システムキャッシュは `Operation not permitted` で保護されて消せません。


## 2. ゴミ箱を空にする

```bash
rm -rf ~/.Trash/*
```


## 3. Homebrew のお掃除
古いバージョンのライブラリを削除。安全。
```bash
brew cleanup
```

プレビューだけ見たい場合：

```bash
brew cleanup -n
```


## 4. Docker のお掃除

```bash
docker system prune -a
```

使ってないコンテナ・イメージを削除。

## 5. Xcode ビルドキャッシュ削除

```bash
rm -rf ~/Library/Developer/Xcode/DerivedData/*
```
プロジェクトごとのビルドキャッシュを削除。再ビルド時に再生成される。


## 6. iOS シミュレーター整理

### 一覧を確認

* ランタイム一覧

```bash
xcrun simctl list runtimes
```

* デバイス一覧

```bash
xcrun simctl list devices
```

### 不要なデバイス削除

```bash
xcrun simctl delete <device-uuid>
```

### Unavailable デバイスを一括削除

```bash
xcrun simctl delete unavailable
```

### 不要ランタイム削除

```bash
xcrun simctl delete runtime com.apple.CoreSimulator.SimRuntime.iOS-XX-X
```

#### 例：

```bash
xcrun simctl delete runtime com.apple.CoreSimulator.SimRuntime.iOS-26-0
```


## 7. 大掃除（自己責任）

```bash
sudo rm -rf /Library/Caches/*
```
システムキャッシュまで消す。基本は不要。


## まとめ


* ストレージに余裕が欲しいときは「シミュレーターの整理」が一番効く。
* SIP（システム保護）のおかげで消せないキャッシュは安全なので安心。
