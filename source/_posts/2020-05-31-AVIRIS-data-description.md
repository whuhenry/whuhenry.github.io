---
title: AVIRIS数据描述简介
date: 2020-05-31 17:38:42
categories: 高光谱
tags:
	- 高光谱
	- 数据集
	- AVIRIS
typora-root-url: 2020-05-31-AVIRIS-data-description
mathjax: true
---

{% note info %}

AVIRIS数据可以通过[AVIRIS Data Protal](https://aviris.jpl.nasa.gov/dataportal/)网站进行检索和下载。该网站提供了2种检索方式，一种是基于地图的检索，另一种是基于元数据列表的检索，元数据列表通过Excel文件格式提供。当确定需要下载的数据后，可以通过官方的FTP服务器进行下载，官方提供L1级数据（辐亮度数据）和L2级数据（反射率数据）下载，下载后的文件为GZIP格式压缩包。具体数据组织和元数据描述方法如下。
{% endnote %}

<!-- more -->

![AVIRIS Data Protal ↑](image-20200603110118989.png)

## 1. 数据文件组成

根据AVIRIS的官方数据说明文档[^1]，AVIRIS数据可以通过FTP服务器下载，以GZIP压缩包格式提供每景数据。L1级和L2级分别提供独立的压缩包。

### 1.1.  L1级数据

L1级数据，也就是经过正射校正的辐亮度图像，以如下的目录结构组织每景数据：

```
/fyymmddtnnpnnrnnrdn_v
|   
|   *ortho.readme（数据说明文件，也就是官方文档）
|   AVIRIS_OrthoProcessing_Info.txt（AVIRS的正射校正详细说明文件）
|   fmmyyddtnnpnnrnnrdn_v_*gain（辐亮度到16bit整型变换的系数，将原始数据除以该系数，得到辐亮度图像，单位μW/cm2/nm/sr）
|   fmmyyddtnnpnnrnnrdn_v_*rcc（辐亮度矫正系数）
|   fmmyyddtnnpnnrnnrdn_v_*spc（光谱矫正文件）
|   fmmyyddtnnpnnrnnrdn_v_*eph（WGS-84/NAD83 UTM坐标系下的位置信息，x,y,z坐标系统）
|   fmmyyddtnnpnnrnnrdn_v_*lonlat_eph（WGS-84坐标系下的经纬度和高程数据）
|   fmmyyddtnnpnnrnnrdn_v_*obs（观测几何参数和光照条件RAW数据文件）
|   fmmyyddtnnpnnrnnrdn_v_*obs.hdr（OBS图像头文件）
|   fmmyyddtnnpnnrnnrdn_v_*obs_ort（使用*_ort_glt查找表变换后的观测几何参数和光照条件RAW数据文件）
|   fmmyyddtnnpnnrnnrdn_v_*obs_ort.hdr（OBS_ORT图像头文件）
|   fmmyyddtnnpnnrnnrdn_v_*ort.plog（通用数据处理信息）
|   fmmyyddtnnpnnrnnrdn_v_*ort_glt（地理几何查找表文件）
|   fmmyyddtnnpnnrnnrdn_v_*ort_glt.hdr（GLT图像头文件）
|   fmmyyddtnnpnnrnnrdn_v_*ort_igm（输入几何文件）
|   fmmyyddtnnpnnrnnrdn_v_*ort_igm.hdr（IMG图像头文件）
|   fmmyyddtnnpnnrnnrdn_v_*ort_img（正射辐亮度图像）
|   fmmyyddtnnpnnrnnrdn_v_*ort_img.hdr（ORT_IMG图像头文件）
```

### 1.2. L2级数据

L2级数据，也就是经过大气校正的反射率数据，以如下的目录结构组织每景数据：

```
/fyymmddtnnpnnrnn_rfl
|   
|   fmmyyddtnnpnnrnnrdn_corr_vn（经过正射校正和大气校正的反射率数据）
|   fmmyyddtnnpnnrnnrdn_corr_vn.hdr（CORR图像头文件）
|   fmmyyddtnnpnnrnnrdn_h2o_v1（反演得到的水蒸气柱浓度和液态水以及并光学路径吸收）
|   fmmyyddtnnpnnrnnrdn_h2o_v1.hdr（H2O图像头文件）
|   *README_vn.txt（README文件，包含了L2数据产品的详细说明）
```

## 1. 数据元数据描述列说明

AVIRIS-Classic 和 AVIRIS-NG具有几乎完全相同的元数据描述方法，元数据具体项及其含义如下

| 参数名称               | 参数说明                                                     |
| ---------------------- | ------------------------------------------------------------ |
| Name                   | 数据名称，对于AVIRIS-Classic，数据名称格式形如`fyymmddtnnpnnrnn`，其中`yymmdd`表示该数据的采集日期、`tnn`表示保留名称，用来和层级记录在VLDS磁带上的数据进行转换、`pnn`电源循环数（永远为00），`rnn`为飞行编号（通常一次飞行不同航带为依次递增编号，如r01，r02等等）；对于AVIRIS-NG数据，数据名称则是形如`ang{YYYYMMDD}t{******}`，同样表示该数据的采集日期和编号等信息 |
| Site Name              | 站点名称，通常表明数据采集目标地区名称以及摄站编号等信息     |
| NASA Log               | NASA log文件名称                                             |
| Investigator           | 数据采集负责人名称                                           |
| Comments               | 数据的一些注释信息，可能包含了数据采集高度、航速以及数据中云含量以及分布等信息 |
| Flight Scene           | 为飞行场景编号，具体形式为`{Name}_{Scene}`                   |
| Date                   | 数据采集日期                                                 |
| RDN Ver                | 辐亮度版本号                                                 |
| Scene                  | 场景编号                                                     |
| GEO Ver                | 几何处理版本（目前全部为ort，意为正射影像）                  |
| YY                     | 数据采集年份简写，形如06,11等                                |
| Tape                   | 保留项，为了与层级记录在磁带上的数据兼容，新的数据该项通常为00 |
| Flight ID              | 形如`fyymmddtnn`表明飞行日期                                 |
| Flight                 | 形如`yymmdd`，为飞行日期                                     |
| Run                    | 飞行编号，与名称中`rnn`对应                                  |
| Year                   | 数据采集年份全称，形如2006,2011等                            |
| Month                  | 数据采集月份                                                 |
| Day                    | 数据采集日期                                                 |
| UTC Hour               | 数据采集小时号（UTC时间，所以经常出现为19、20点采集）        |
| UTC Minute             | 数据采集分钟点                                               |
| Pixel Size             | 像素地面分辨率（单位是m），主要分为两种，一种低空飞行，分辨率在3~5m左右，另一种为高空飞行，分辨率在14~16m左右 |
| Rotation               | 数据倾斜角度，，                                             |
| Number of Lines        | 扫描线数量（图像高）                                         |
| Number of Samples      | 每条扫描线像素数（图像宽）                                   |
| Solar Elevation        | 太阳高度角                                                   |
| Solar Azimuth          | 太阳方位角                                                   |
| Mean Scene Elevation   | 场景平均高程                                                 |
| Min Scene Elevation    | 场景最小高程                                                 |
| Max Scene Elevation    | 场景最大高程                                                 |
| File Size (GB)         | 未压缩图像大小（GB为单位）                                   |
| Gzip File Size (GB)    | Gzip格式图像压缩包大小（GB为单位）                           |
| File Size (Bytes)      | 未压缩图像大小（Byte为单位）                                 |
| Gzip File Size (Bytes) | Gzip格式图像压缩包大小（Byte为单位）                         |
| Download Name          | 下载的图像压缩包名称，一般形式为`{Name}.tar.gz`              |
| link_kml_overlay       | 整景图像覆盖区域的kml文件下载链接                            |
| link_kml_outline       | 整景图像中实际拍摄到的目标区域的kml文件下载链接（相当于将图像中的黑边去除掉的有效图像区域） |
| link_rgb               | 合成三波段可视图像下载地址（HTTP协议）                       |
| link_rgb_small         | 合成三波段可视图像小样下载地址（HTTP协议）                   |
| link_ftp               | 辐亮度数据（L1B）压缩包的FTP下载地址                         |
| link_log               | 日志文件下载地址                                             |
| Lon1                   | 数据外包矩形顶点1经度                                        |
| Lon2                   | 数据外包矩形顶点2经度                                        |
| Lon3                   | 数据外包矩形顶点3经度                                        |
| Lon4                   | 数据外包矩形顶点4经度                                        |
| Lat1                   | 数据外包矩形顶点1纬度                                        |
| Lat2                   | 数据外包矩形顶点2纬度                                        |
| Lat3                   | 数据外包矩形顶点3纬度                                        |
| Lat4                   | 数据外包矩形顶点4纬度                                        |
| kml_poly               | KML语言格式记录的数据外包矩形                                |
| HTML Link CH4          | 空                                                           |
| HTML Link Reflectance  | 反射率数据（L2）压缩包FTP下载地址（只有部分数据生成了反射率数据） |

[^1]: [AVIRIS Distribution Document 20160614](https://aviris.jpl.nasa.gov/dataportal/20170911_AV_Download.readme)