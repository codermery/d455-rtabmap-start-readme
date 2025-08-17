# RTAB-Map ile Intel RealSense D455 Kamera Kullanımı (ROS 2)

Bu doküman, Intel RealSense D455 kamera kullanılarak **RTAB-Map** ile RGB-D tabanlı haritalama yapılmasını anlatır.
ROS 2 ortamında kamera sürücüsü, transform yayıncısı ve RTAB-Map başlatma adımları aşağıda listelenmiştir.

---

## Gereksinimler

* ROS 2 (Humble veya uyumlu sürüm)
* `realsense2_camera` paketi
* `rtabmap_ros` (veya `rtabmap_launch`)
* `tf2_ros`

---

## Adımlar

### 1. RealSense Kamerasını Başlat

Aşağıdaki komut ile D455 kamera sürücüsü çalıştırılır.
`align_depth:=true` parametresi, depth verisini renkli görüntüyle hizalar.

```bash
ros2 launch realsense2_camera rs_launch.py camera_name:=d455
veya
ros2 launch realsense2_camera rs_d455_pointcloud_launch.py camera_name:=d455
```

Bu komut ile aşağıdaki topic’ler yayınlanır:

* `/camera/d455/color/image_raw`
* `/camera/d455/depth/image_rect_raw`
* `/camera/d455/color/camera_info`

---

### 2. Statik Transform Yayınla

Kamerayı `base_link` koordinat sistemine bağlamak için statik transform yayınlanır.

```bash
  ros2 run tf2_ros static_transform_publisher 0 0 0 0 0 0 base_link d455_link
```

Burada:

* `0 0 0 0 0 0` → konum ve oryantasyon (x, y, z, roll, pitch, yaw)
* `base_link` → robotun ana gövde çerçevesi
* `d455_link` → kameraya ait çerçeve

---

### 3. RTAB-Map Başlat

RTAB-Map’i başlatmak için:

```bash
ros2 launch rtabmap_launch rtabmap.launch.py \
  rtabmap_args:="--delete_db_on_start" \
  rgb_topic:=/camera/d455/color/image_raw \
  depth_topic:=/camera/d455/depth/image_rect_raw \
  camera_info_topic:=/camera/d455/color/camera_info \
  approx_sync:=true \
  qos_image:=2 \
  qos_camera_info:=2
```

Parametreler:

* `--delete_db_on_start` → her başlatmada eski veritabanını siler.
* `rgb_topic` → kameranın renkli görüntü topic’i.
* `depth_topic` → kameranın derinlik görüntü topic’i.
* `camera_info_topic` → kamera kalibrasyon bilgileri.
* `approx_sync:=true` → RGB ve depth verilerini yaklaşık zaman senkronizasyonu ile eşler.
* `qos_image`, `qos_camera_info` → kalite servis (QoS) ayarları.

---

## Çalıştırma Akışı

1. Kamerayı başlat
2. Statik transform yayınla
3. RTAB-Map ile haritalama başlat

Bu üç adım ile D455 kamera kullanılarak RTAB-Map üzerinde haritalama yapılabilir.

---

## Notlar

* RTAB-Map çıktısını görmek için **RViz2** kullanabilirsin:

  ```bash
  ros2 run rviz2 rviz2
  ```
* Kameranın doğru çalıştığını test etmek için:

  ```bash
  ros2 topic hz /camera/d455/color/image_raw
  ```
