记录一下在POWER BI中如何使用Drilldown choropleth插件实现地图下钻功能。

一、准备工作：

1.获取Drilldown choropleth插件

2.Geojson格式的形状地图，分为省级和市级，可从[阿里云](http://datav.aliyun.com/tools/atlas/#&lat=33.50475906922609&lng=104.2822265625&zoom=4) 中分别获取并合并

3.合并地图，以及geojson转topojson格式：https://mapshaper.org/

二、为了实现下钻效果，先来处理地图:

1.从阿里云获得多个单独的形状地图后，首先需要合并，这里需要打开[mapshaper](https://mapshaper.org/)
一次导入多个json文件只需同时选中并拖拽上传，点击右上角console，在左侧输入代码 -merge-layers target=* force （注意这里代码开头的-不可忽略），可以看到所有的图层已经合并，此时可以导出geojson地图。

2.由于省级的地图和市级的地图之间没有关键字作为连接，需要先在geojson文件中添加关键字，再转换成topojson格式。
  这里采用的是python中的geopandas库，以省级地图为例，需要先准备province_connect.csv作为对应关系，其中两列分别是province和province_connect.代码如下:

      import geopandas as gpd
      import pandas as pd
      
      # 1. 读取已合并的 GeoJSON，以省级为例
      gdf = gpd.read_file(r"C:\Users\Documents\Data tools\待整理\中国_省.geojson")
      # 2. 读取映射表 CSV
      df_map = pd.read_csv(r"C:\Users\Downloads\province_connect.csv")
      # 3. 将映射表转为字典，方便映射
      mapping = dict(zip(df_map["province"], df_map["province_connect"]))
      # 4. 新增属性列
      gdf["province"] = gdf["name"].map(mapping)
      # 5. 将结果写回 GeoJSON
      gdf.to_file(r"C:\Users\Downloads\geojson_province_merged_with_province.geojson", driver="GeoJSON")
      
      print("Done!")

  同理，在市级的地图转换前，需要准备city_connect.csv,其中包含city与province_connect，得到市级的geojson地图文件。
  
3.在[mapshaper](https://mapshaper.org/)中导入文件，并转换为topojson格式，导出备用。

三、topojson的网络地址

为了在Power BI中实现地图的调用，需要把topojson文件存储在网络环境中，如GitHub/SharePoint等等，此处采用的是GitHub。

1.在GitHub中新建一个仓库，并将得到的省级和市级topojson地图上传。

2.在GitHub中注意代码设置为public才能通过powerbi读取，点击raw按钮获取链接，并通过jsDelivr加速访问，获得的链接如下：

省级：https://cdn.jsdelivr.net/gh/rainemeow/china_geojson_map/topojson_province_merged_with_province.json
市级：https://cdn.jsdelivr.net/gh/rainemeow/china_geojson_map/topojson_city_merged_with_province.json

四、Power BI操作:

1.按图相应设置Drilldown choropleth中的参数，注意Level 1为省级链接，Level 2为市级链接，ID 1为省级的连接词，也就是提前在地图中添加的关键词"province"
![image](https://github.com/user-attachments/assets/b5dfe297-90c1-4a42-a41a-ac2a56f7a659)

2.想要实现钻取功能，需要注意点击↓图标，开启钻取，效果如下
![image](https://github.com/user-attachments/assets/f177d2fa-1d80-40e3-bd12-9d04a7166f22)

3.填充颜色设置在Format-Data colors

![image](https://github.com/user-attachments/assets/5e8e38e8-f597-47e2-9afb-9ef6e1766b47)


未完待续……
