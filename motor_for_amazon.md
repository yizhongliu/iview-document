# 1 Running Mode

Mirror Projector app support three running mode:

## 1.1 Free Control Mode

Android phone App can control the mirror projector like a remote control which can rotate mirror, play video and image, and do the geometric correction.

## 1.2 Path Planning Mode

Users can use Android App to plan a moving path of the mirror projector. In this mode, the projector will not response to commands of free control mode

## 1.3 Auto Running Mode

Mirror Projector will auto run according to the path saved in path planning mode. In this mode , the mirror projector would not response to any command from phone except stop auto run.



# 2 Socket Data Format

## 2.1 Free Control Mode

### 2.1.1  Control motor move

```python
{
  "type":"Control"
  "action":"Move"
  "arg": {
	"angle" :  0   // int (0, 90, 180, 270)
  }
}
```



2.1.2  Show video or image

```python
{
  "type":"Control"
  "action":"Show"
  "arg": {
	"rotation":0           //rotation degree 0-360
	"url": "test.jpg"     // file name
	"imgTime":5000   //image display time(ms)
    "keystone":0  // -40 - 40
  }
}
```



2.1.3  Change rotation and keystone while media file is playing(rotation is not support for video) 

```python
{
  "type":"Control"
  "action":"SetParam"
  "arg": {
	"rotation":0    //image rotation degree 0-360
    "keystone":0  // -40 - 40
  }
}
```



## 2.2 Path Planning Mode

2.2.1 Enter path planning mode. When received this command, the mirror will move to initialization location. The projector will not response to any command while it is moving  

```python
{
  "type":"PathPlanning"
  "action":"Start"
}
```

2.2.2 Leave path planning mode and save path name

```python
{
  "type":"PathPlanning"
  "action":"Stop"
  "arg": {
	"pathName":"aaaa"
   }
}
```

2.2.3  Control motor move

```python
{
  "type":"PathPlanning"
  "action":"Move"
  "arg": {
	"angle" :  0   // int (0, 90, 180, 270)
  }
}
```

2.2.4 Show video or image, this command will not be save in planning path

```python
{
  "type":"PathPlanning"
  "action":"Preview"
  "arg": {
	"rotation":0           //rotation degree 0-360
	"url": "test.jpg"     // file name
	"imgTime":5000   //image display time(ms)
    "keystone":0  // -40 - 40
  }
}
```

2.2.5  Change rotation and keystone while media file is playing(rotation is not support for video) , this command will not be saved in planning path

```python
{
  "type":"PathPlanning"
  "action":"SetParam"
  "arg": {
	"rotation":0    //image rotation degree 0-360
    "keystone":0  // -40 - 40
  }
}
```

2.2.6  Set video/image display in the current location. This command will be save the planning path, but media file will not be played.

```python
{
  "type":"PathPlanning"
  "action":"Show"
  "arg": {
	"rotation":0           //rotation degree 0-360
	"url": "test.jpg"     // file name
	"imgTime":5000   //image display time(ms)
    "keystone":0  // -40 - 40
  }
}
```

2.2.7 Leave path planning mode without saving planning path

```python
{
  "type":"PathPlanning"
  "action":"Cancel"
}
```

## 2.3 Auto Running Mode

2.3.1 Start auto running

```python
{
  "type":"AutoRunning"
  "action":"Start"
  "arg": {
	"pathName":"aaaa"
   }
}
```

2.3.2 Stop auto running

```python
{
  "type":"AutoRunning"
  "action":"Stop"
}
```



# 3 HTTP data format

The port 8093 is fixed.



3.1 Get file list in the projector

Curl request

```bash
curl -X GET \
  http://192.168.0.139:8093
```

Http response

```python
{
    "files":["1565146851237.jpg","logo","dayin",".kspdf","baidu",".BD_SAPI_CACHE",".zp",".x_o_b_d",".g_m_o_bs",".g_b_d_v","TbsReaderTemp","news_article","1562634565913_logo.png","1565147488884.jpg","GfanPlatform",".um",".uxx",".stats",".qmt","QTDownloadRadio",".cc",".a.dat","cn.net.gfan.portal","360","BaiduNetdisk","userexperience","Mprinter",".lm_device","1558526323360_logo.png"]
}
```



3. 2 Get the thumbnail of media file

Curl request

```bash
curl -X GET \
  http://192.168.0.102:8093/test3.mp4 \
  -H 'cache-control: no-cache' \
  -H 'resize: 200,100'
```

http response:

file stream



3.3 Get the path list saved in the projector

Curl request

```bash
curl -X GET \
  http://192.168.0.139:8093/getPathlist
```

http response

```json
{"pathlist":["aaa", "bbb", "ccc"]}
```



3.4 Delete path list

Curl request

```bash
curl -X POST \
  http://192.168.0.139:8093/delPathlist \
  -H 'Content-Type: application/json' \
  -d '{
    "pathlist": ["aaa", "bbb"]
}'
```

http response

```json
{"pathlist":["ccc"]}
```



3.5 Get the state of projector

Curl reques

```bash
curl -X GET \
  http://192.168.0.139:8093/getDeviceState
```

http response

```json
{
   "RunningState":"autorunning",
   "PathName": "aaa"  //may be null
}
```

parameters

|                    | RunningState |
| ------------------ | ------------ |
| Free control mode  | control      |
| Path planning mode | pathplanning |
| Auto running mode  | autorunning  |

