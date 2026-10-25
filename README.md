# rabbits_ws
ATTRACTS向け開発ロボット用のROS2パッケージ群の環境構築用リポジトリ

[![CI](https://github.com/rabbits-robotics/rabbits_ws/actions/workflows/ci.yml/badge.svg)](https://github.com/rabbits-robotics/rabbits_ws/actions/workflows/ci.yml)

## 事前準備
### RaspberryPi5+Ubuntu24.04+m.2SSDBoot
- これを参照
  - https://raspida.com/how2boot-nvme-ubuntu/

### ROS2 Jazzyのインストール
- これを参照
  - https://www.techlife-hacking.com/?p=2857

### windows10からubuntu24.04へのRDP接続
- これを参照
  - https://qiita.com/hitobb/items/b44e87e04133dc845265
  - https://engrmemo.jp/linux/ubuntu-remote-desktop/#google_vignette
  - https://knowledge.f5.si/open.knowledge/view/51
  - https://qiita.com/Olykoek/items/9956e375d06c69de4243

#### 他の方法
やったら追記、誰かやったら追記して欲しい

## 使い方
- `git clone git@github.com:rabbits-robotics/rabbits_ws.git`
- `cd rabbits_ws`
- `vcs import src < default_package_list.repos`
- `colcon build`

## 開発環境（Dev Container）

Windows / macOS / Linux のどれでも、Docker と VS Code があれば CI と同じ環境（`ros:jazzy-ros-base` + colcon / rosdep / vcstool / gtest / lint）で開発できます。

### 準備（Windows）
1. 管理者の PowerShell で `wsl --install -d Ubuntu-24.04` を実行し、再起動する
2. [Docker Desktop](https://www.docker.com/products/docker-desktop/) をインストールし、Settings → Resources → WSL integration で Ubuntu-24.04 を有効にする
3. VS Code に拡張機能 [Dev Containers](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers) を入れる
4. （任意）コンテナ内から push したい場合は、Windows の OpenSSH Authentication Agent を起動して `ssh-add` で鍵を登録しておく（Dev Containers がエージェントをコンテナへ転送する）

### 使い方
1. このリポジトリを VS Code で開き、コマンドパレットから **Dev Containers: Reopen in Container** を選ぶ
2. 初回はイメージのビルド後に `vcs import src < default_package_list.repos` と `rosdep install` が自動で走る。SSH 鍵が無ければ `ci.repos`（HTTPS）に自動で切り替わる。`src/` にすでにあるパッケージはそのまま使う
3. ターミナルで `colcon build` → `colcon test` → `colcon test-result --verbose`

### GUI（rviz2 など）

Docker Desktop はホスト側の WSLg を `/run/desktop/mnt/host/wslg` に見せているので、devcontainer.json でそれをコンテナにマウントし、`DISPLAY=:0` で X11 として表示します。描画はソフトウェア OpenGL です（`LIBGL_ALWAYS_SOFTWARE=1`）。

```bash
# ロボットモデルを rviz2 で表示（joint_state_publisher_gui 付き）
ros2 launch rabbits_description display.launch.py use_gui:=true

# rviz2 だけ
rviz2
```

通常の `ros2 run` / `ros2 launch` / `ros2 topic` はコンテナ内のターミナルでそのまま使えます。`colcon build` のあとは新しいターミナルを開くか `source install/setup.bash` を実行してください。

### USB 機器（シリアル変換機、ゲームパッド）をコンテナから使う（Windows）

Docker Desktop は WSL2 の VM で動くので、USB 機器を [usbipd-win](https://github.com/dorssel/usbipd-win) で WSL2 に渡すとコンテナからも見える。devcontainer.json は `--privileged` と `/dev` のマウントで機器を見せている。

1. 管理者 PowerShell で `winget install --interactive --exact dorssel.usbipd-win`。usbipd が機器を渡す先として WSL の配布が 1 つ要る（`wsl --install -d Ubuntu`。ROS 2 はコンテナ内で動かすので Ubuntu 側には何も入れない）
2. 機器を挿して `usbipd list` で BUSID を確認し、管理者 PowerShell で `usbipd bind --busid <BUSID>` → `usbipd attach --wsl --busid <BUSID>`（attach は挿し直すたびに必要。`--auto-attach` を付けるとそのウィンドウを占有して挿し直しに自動で追従する。attach には WSL の配布が 1 つ動いている必要があるので、先に `wsl -d Ubuntu` を開いておく）
3. `wsl -d Ubuntu -e ls -l /dev/ttyUSB0 /dev/input/js0` で見えることを確認してから、Dev Container を開く（開いていた場合は Rebuild / Reopen）
4. コンテナ内で `ros2 launch rabbits_bridge robot_bridge.launch.py config:=...`（YAML の `device` を `/dev/ttyUSB0` に）と `ros2 launch rabbits_interface gamepad.launch.py`

WSL2 のカーネル（6.x）には FTDI / CP210x / CH341 / PL2303 と joydev のドライバが入っている。シリアル変換機は 3.3 V のものを使う（ESP32-P4 は 5 V 非対応）。

### 注意

- HTTPS で取得したパッケージはそのままでは push できない。push するなら SSH 鍵を転送した上で `git remote set-url origin git@github.com:rabbits-robotics/<パッケージ名>.git` に切り替える
- Linux ホストで使う場合は、devcontainer.json の WSLg 用マウントをホストの `/tmp/.X11-unix` に差し替え、`--privileged` は外して `--device` で必要な機器だけ渡す
- Jetson 実機との DDS 通信は Docker Desktop では制約があるため、実機側で行う

## その他
- このリポジトリで使っている`vcs`について
  - https://memoteki.net/archives/3296
- UbuntuにGitをインストール
  - https://qiita.com/tommy_g/items/771ac45b89b02e8a5d64
- ubuntuで日本語入力を有効にする
  - https://lilaboc.work/archives/29007972.html

## legacy
## 事前準備
### Ubuntu22.04のインストール
#### WSLを用いた方法
- Ubuntu22.04のインストールはこれを参照
  - https://zenn.dev/sunazukin/articles/8fe49e9dee7b7c
- ubuntu-desktopのインストールはこれを参照
  - https://qiita.com/Akaaku/items/31428b3478a1b58196a6

### ROS2 Humbleのインストール
- これを参照
  - https://qiita.com/porizou1/items/5dd915402e2990e4d95f