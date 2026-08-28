# hokuyo_navigation2

`hokuyo_navigation2` は、北陽電機製の 3D LiDAR・RTK-GNSS一体型センサ RSF 専用の ROS2 ベースの屋内外対応ナビゲーションシステムのパッケージ群です。3D-SLAM, 自己位置推定、ROS 2 Navigation Stack (Nav2) を連携させ、高精度な自律移動を実現します。また、直感的な操作を可能にするWebベースのGUIを用いることで、マッピングからナビゲーションまでの一連の操作をブラウザから簡単に行うことができます。

![hokuyo_navigation2](Image/hokuyo_navigation2.png)

![expo_nav](Image/expo_navigation.png)

![hokuyo_navigation2_gui](Image/hokuyo_navigation2_gui_pc.png)

![expo2025](Image/expo2025.png)

![map_viewer](Image/Map_viewer.png)

## 主な機能

- **ロボットとセンサノードの起動**
  - モータドライバ・センサのROS2ノードの起動
  **本パッケージはモータドライバとして [icart_mini_driver_ros2](https://github.com/hokuyo-rd/icart_mini_driver_ros2)を使用したサンプルとなっています。使用するモータドライバに合わせて[このスクリプト](https://github.com/hokuyo-rd-release/hokuyo_navigation2/blob/release/scripts/navigation/nav_common.sh)のlaunch_motor_driver関数を変更してください。**
- **3D SLAMと2Dウェイポイントファイル出力の同時実行**:
  - `hokuyo_lio` を用いた高精度なLiDAR慣性オドメトリ（LIO）と3D点群マップ生成。
  - ROS Bagから`lio_raw`（軌跡ベース）または`p2o`（点群ベース）の3Dマップ（`.pcd`）を作成。
  - 3D点群マップ生成と同時にウェイポイント作成
  - 3D点群マップをNav2用の2Dグリッドマップ（`.pgm`, `.yaml`）へ変換。

- **ナビゲーション [`hokuyo_navigation2`](https://github.com/hokuyo-rd-release/hokuyo_navigation2)**:
  - `hokuyo_rsf`を利用したリアルタイム3D自己位置推定を用いた自律走行
  - 3D点群マップと`simple_fastlio_localization` を利用したリアルタイム自己位置推定を用いた自律走行
  - Nav2 (Navigation2) スタックと連携し、指定されたウェイポイントに沿った自律走行。
  - 単一マップ走行および複数マップを連続して走行するマルチマップナビゲーションに対応。

- **ブラウザベースの統合GUI [`hokuyo_navigation2_gui`](https://github.com/hokuyo-rd-release/hokuyo_navigation2_gui)**:
  - **プロセス実行**: データ取得、マッピング、ナビゲーションの各プロセスをブラウザから起動。
  - **ファイル管理**: マップ、ウェイポイント、設定ファイルなどをブラウザ上で管理（作成、名前変更、削除）。
  - **高機能エディタ**:
    - **Map Viewer**: 3D/2Dマップとウェイポイントを視覚化し、GUI上で直感的にウェイポイントを編集（追加、移動、回転、属性変更）。
    - **CSV Editor**: マルチマップ走行シナリオをテーブル形式で簡単に編集。

## 使い方

[hokuyo_navigation2 ユーザーマニュアル](https://github.com/hokuyo-rd-release/hokuyo_navigation2_users_manual/blob/0a2411ed30dbcc8be91f021d0c6ca1f0b338de7c/hokuyo_navigation2_usersmanual_v1.0.0.pdf) ドキュメントバージョン 1.0.0 を公開しました。
本ドキュメントバージョンで説明する hokuyo_navigation2 のソフトウェアバージョンは 1.0.0 となります。
ROS 2 Humble / Ubuntu 22.04 (`release` ブランチ) と ROS 2 Jazzy / Ubuntu 24.04 (`jazzy` ブランチ) の
**両方の構成を併記**しており、以下の全手順を収録しています。

1. システム構成
2. セットアップ（環境構築）
3. GUIアプリケーションの起動
4. コンフィグファイルの設定
5. データの取得
6. マッピング
7. 経路設計
8. 2D地図への変換
9. 自律走行
10. ファイル管理とデータ変換ツール
11. トラブルシューティング（エラー番号 E-101 〜 E-610）

付録として、パッケージリファレンス（全サブモジュールの詳細）、設定・パラメータ早見表、用語集を収録しています。

> **旧版**: ドキュメントバージョン 0.1.0 は [doc/hokuyo_navigation2_usersmanual_v0.1.0.pdf](doc/hokuyo_navigation2_usersmanual_v0.1.0.pdf)、
> 1.0.0 は [doc](doc) にあります。

### マニュアルのビルド

マニュアルは [Typst](https://typst.app/) で記述されています。ソースは `doc/hokuyo_navigation2_users_manual/` にあります。

```bash
# 日本語フォント (原ノ味フォント) が必要です
cd doc/hokuyo_navigation2_users_manual
typst compile hokuyo_navigation2_usersmanual_v1.1.0.typ
```

## セットアップ

ROS2 Version: Jazzy (Ubuntu 24.04 LTS / Python 3.12)

> **注意**: 本ブランチ (`jazzy`) は ROS 2 Jazzy / Ubuntu 24.04 LTS 用の手順です。
> ROS 2 Humble / Ubuntu 22.04 で使用する場合は `release` ブランチを参照してください。

```bash
sudo apt-get update 
sudo apt-get install -y tree xdotool wmctrl zenity bc python3-pip

# 1. build ros2 packages
cd <YOUR_ROS2_WORKSPACE>/src
git clone https://github.com/Hokuyo-aut/hokuyo_rsf.git
git clone --recursive -b jazzy https://github.com/Hokuyo-aut/hokuyo_navigation2.git
cd <YOUR_ROS2_WORKSPACE>
rosdep update
rosdep install --from-paths src/hokuyo_navigation2 --ignore-src -r -y
sudo apt-get install ros-jazzy-tf-transformations ros-jazzy-joint-state-publisher ros-jazzy-robot-state-publisher

# 下記エラーが出力されますが無視して進めて下さい。
# ERROR: the following packages/stacks could not have their rosdep keys resolved to system dependencies:
# waypoint_manager: Cannot locate rosdep definition for [move_base_msgs] → 使用しません。
# hokuyo_navigation2: Cannot locate rosdep definition for [tf-transformations] → 後の手順でインストールします。
# fix2xyz: Cannot locate rosdep definition for [PROJ] → 後からの手順でインストールします。

# 2. Python パッケージ
# Ubuntu 24.04 (Python 3.12) は PEP 668 により外部管理環境となっているため、
# オプション無しで pip3 install するとインストールに失敗します。
#   error: externally-managed-environment
# 下記のように --user --break-system-packages を付けてユーザ環境 (~/.local) へインストールしてください。
cd <YOUR_ROS2_WORKSPACE>/src/hokuyo_navigation2
pip3 install --user --break-system-packages -r requirements.txt

# ~/.local/bin に PATH が通っていない場合は追加してください。
# echo 'export PATH=$HOME/.local/bin:$PATH' >> ~/.bashrc && source ~/.bashrc

# 3. Motor_driver if you need.
# ypspurのインストール
mkdir ~/hokuyo_lib
cd ~/hokuyo_lib
sudo apt-get install libmodbus-dev
git clone https://github.com/hokuyo-rd-release/yp-spur.git
cd yp-spur
mkdir build
cd build
cmake ..
make
sudo make install

# モータドライバインストールの確認
# 端末 1
# ypspur-coordinator -d /dev/ttyUSB0 --blvr -p <PATH_TO_YOUR_PARAM_FILE>/wizurg_lio.param

# 端末 2
# cd ~/colcon_ws/src/yp-spur/build/sample
# ./run-test

# icart_mini_driver_ros2のインストール
cd <YOUR_ROS2_WORKSPACE>/src
git clone https://github.com/hokuyo-rd-release/icart_mini_driver_ros2.git
colcon build --symlink-install --packages-select icart_mini_driver

# 3. hokuyo_slam_ros2 build
sudo apt-get install libsqlite3-dev sqlite3 libeigen3-dev qtbase5-dev clang qtcreator libqt5x11extras5-dev

## proj
# Ubuntu 24.04 の apt には proj 9.4.0 (libproj-dev) しか無いため、9.4.1 をソースからビルドして
# /usr/local へインストールします。
cd ~/hokuyo_lib
wget https://download.osgeo.org/proj/proj-9.4.1.tar.gz
tar -zxvf proj-9.4.1.tar.gz
cd proj-9.4.1
mkdir build
cd build
cmake ..
cmake --build .
sudo cmake --build . --target install

## pcl 1.14.1 ビルドに時間がかかります。
# Ubuntu 24.04 の apt には pcl 1.14.0 (libpcl-dev) しか無いため、1.14.1 をソースからビルドして
# /opt/pcl へインストールし、CMAKE_PREFIX_PATH で参照します。
cd ~/hokuyo_lib
wget https://github.com/PointCloudLibrary/pcl/releases/download/pcl-1.14.1/source.tar.gz -O pcl.tar.gz
tar -xvf pcl.tar.gz
cd pcl
cmake -Bbuild -DCMAKE_INSTALL_PREFIX=/opt/pcl .
cmake --build build
sudo cmake --install build
export CMAKE_PREFIX_PATH=$CMAKE_PREFIX_PATH:/opt/pcl

## hokuyo_slam_ros2
cd <YOUR_ROS2_WORKSPACE>/src/hokuyo_navigation2/hokuyo_slam_ros2
export CMAKE_PREFIX_PATH=$CMAKE_PREFIX_PATH:/opt/pcl

mkdir build
cmake -Bbuild . && cmake --build build

# 4. colcon build
cd <YOUR_ROS2_WORKSPACE>/src/hokuyo_navigation2/hokuyo_navigation2
mkdir map/ rosbag/ waypoints/
cd <YOUR_ROS2_WORKSPACE>
colcon build
colcon build --symlink-install --packages-select hokuyo_navigation2 lio_nav2_bringup simple_fastlio_localization

# 5. firewall
sudo ufw allow 5050/tcp
sudo ufw allow 5050
sudo ufw allow 5000/tcp
sudo ufw allow 5000
sudo ufw allow 5001/tcp
sudo ufw allow 5001
sudo ufw allow 8000/tcp
sudo ufw allow 8000
sudo ufw allow 9000/tcp
sudo ufw allow 9000
sudo ufw allow 9090/tcp
sudo ufw allow 9090
sudo ufw allow 10940/tcp
sudo ufw allow 10940
sudo ufw allow 7400:7800/udp
sudo ufw enable
```

## 実行方法

サーバーの起動

```bash
cd <YOUR_ROS2_WORKSPACE>/src/hokuyo_navigation2/hokuyo_navigation2
./scripts/start_server.sh
```

Webブラウザで `http://<ホストマシンのIPアドレス>:5050` にアクセスします。GUIの指示に従い、マッピングやナビゲーションを実行してください。詳細は [`hokuyo_navigation2_gui`](https://github.com/hokuyo-rd/hokuyo_navigation2_gui) のドキュメントを参照してください。

## 停止方法

サーバーの停止

```bash
cd <YOUR_ROS2_WORKSPACE>/src/hokuyo_navigation2/hokuyo_navigation2
./scripts/stop_server.sh
```
