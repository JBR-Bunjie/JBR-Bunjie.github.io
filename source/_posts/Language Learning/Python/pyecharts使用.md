## install： pip install pyecharts

## Normal Process

1. import dependencies

```python
from pyecharts.charts import * # 引入所有图表类
from pyecharts.components import Table # 引入table组件
from pyecharts import options as opts # 引入options并rename
from pyecharts.commons.utils import JsCode # 引入原生的js代码

from snapshot_selenium import snapshot # 使用snapshot_selenium渲染图片
from pyecharts.render import make_snapshot

import random
import datetime
...
```

2. 对对象添加配置项：

   几乎所有配置项都会在

   + 全局配置项：通过配置全局项，可以更好的设置个性化图表，提供功能组件和更多信息

     > 1. 在定义类的详细内容时使用：set_global_opts（function）来进行设定
     >
     > 2. 常用全局配置：
     >
     >    >
     >    >
     >    >```python
     >    >init_opts= # 这是一个例外：这个需要在Bar() 即新建类时调用，用于设定width等值
     >    >
     >    >title_opts=
     >    >toolbox_opts=
     >    >tooltip_opts=
     >    >visualmap_opts=
     >    >```
     >    >
     >    >
     
   + 系列配置项：

## Basic Charts

### Calendar：日历图

```python
1. class pyecharts.charts.Calendar
	class Calendar(init_opts: opts.InitOpts = opts.InitOpts())
2. func pyeachrts.charts.Calendar.add
	yaxis_data: Sequence, # example: [(key, value), ...]
    calendar_opts=opts.CalendarOpts(
        range_="2017",
    ),
3. 
```

### Funnel：漏斗图

```python
# 系列数据项，格式为 [(key1, value1), (key2, value2)]
    data_pair: Sequence,
```

### Gauge：仪表盘

```python
# 系列数据项，格式为 [(key1, value1), (key2, value2)]
    data_pair: Sequence,
```

### Graph：关系图

```python
    # 关系图节点数据项列表，参考 `opts.GraphNode`
    nodes: Sequence[Union[opts.GraphNode, dict]],

    # 关系图节点间关系数据项列表，参考 `opts.GraphLink`
    links: Sequence[Union[opts.GraphLink, dict]],

    # 关系图节点分类的类目列表，参考 `opts.GraphCategory`
    categories: Union[Sequence[Union[opts.GraphCategory, dict]], None] = None,
```

### Liquid: 水流图

```python
# 系列数据，格式为 [value1, value2, ....]
    data: Sequence # normally： value1 > value2 > ...
```

### Parallel: 平行坐标系

```python
# 轴定义：
parallel_axis = [{"dim": 0, "name": "Price"},
    {"dim": 1, "name": "Net Weight"},
    {"dim": 2, "name": "Amount"},
    {   "dim": 3,
        "name": "Score",
        "type": "category",
        "data": ["Excellent", "Good", "OK", "Bad"],},]
# 数据准备
data = [(12.99, 100, 82, "Good"), (9.99, 80, 77, "OK"), (20, 120, 60, "Excellent")]
(
    Parallel(init_opts=opts.InitOpts(width="1400px", height="800px"))
    .add_schema(schema=parallel_axis)
    .add(
        series_name="",
        data=data
    )
    .render()
)
```

### Pie

```python
# 系列数据项，格式为 [(key1, value1), (key2, value2)]
data_pair: types.Sequence[types.Union[types.Sequence, opts.PieItem, dict]],
```

### Polar：极坐标系

```python
# 对数据存在一个add_schema来定义轴
 def add_schema(
    radiusaxis_opts: Union[opts.RadiusAxisOpts, dict] = opts.RadiusAxisOpts(),
    angleaxis_opts: Union[opts.AngleAxisOpts, dict] = opts.AngleAxisOpts(),
)
# 数据本身仍在add中引入
    # 系列数据项
    data: Sequence, # 数据形式是数组：[(key, value1), value2...]       
    # 注意一个类型选项
    type_="bar" # 支持ChartType.SCATTER, ChartType.LINE, ChartType.BAR，ChartType.EFFECT_SCATTER
默认是极坐标形式的散点图

    Polar()
    .add_schema(
        radiusaxis_opts=opts.RadiusAxisOpts(type_="category"),
        angleaxis_opts=opts.AngleAxisOpts(is_clockwise=True, max_=10),
    )
    .add("A", [("周一", 1), 2, 3, 4, 3, 5, 1], type_="bar")
    .set_global_opts(title_opts=opts.TitleOpts(title="Polar-RadiusAxis"))
    .set_series_opts(label_opts=opts.LabelOpts(is_show=True))
    .render()
```

### Radar

```python
v1 = [(4300, 10000, 28000)]
v2 = [(5000, 14000, 28000)]
(Radar(init_opts=opts.InitOpts(width="1280px", height="720px", bg_color="#CCCCCC"))
    .add_schema(
        schema=[
            opts.RadarIndicatorItem(name="销售（sales）", max_=6500),
            opts.RadarIndicatorItem(name="研发（Development）", max_=52000),
            opts.RadarIndicatorItem(name="市场（Marketing）", max_=25000),
        ],
        splitarea_opt=opts.SplitAreaOpts(
            is_show=True, areastyle_opts=opts.AreaStyleOpts(opacity=1)
        ),
        textstyle_opts=opts.TextStyleOpts(color="#fff"),
    )
    .add(
        series_name="预算分配（Allocated Budget）",
        data=v1,
        linestyle_opts=opts.LineStyleOpts(color="#CD0000"),
    )
    .add(...)
    .set_series_opts(label_opts=opts.LabelOpts(is_show=False))
    .set_global_opts(
        title_opts=opts.TitleOpts(title="基础雷达图"), 
    )
    .render())

    # 系列名称，用于 tooltip 的显示，legend 的图例筛选。
    series_name: str,
```

### Sankey：桑基图

```python
nodes = [{"name": "category1"},
    {"name": "category2"},
    {"name": "category3"},
    {"name": "category4"},
    {"name": "category5"},
    {"name": "category6"},]
links = [{"source": "category1", "target": "category2", "value": 10},
    {"source": "category2", "target": "category3", "value": 15},
    {"source": "category3", "target": "category4", "value": 20},
    {"source": "category5", "target": "category6", "value": 25},]
(Sankey().add(
    series_name="sankey",
    nodes=nodes,
    links=links,
    pos_bottom=50
    # linestyle_opt=opts.LineStyleOpts(opacity=0.2, curve=0.5, color="source"),
    # label_opts=opts.LabelOpts(position="right"),
).set_global_opts(title_opts=opts.TitleOpts(title="Sankey-基本示例")).render())
```

### Sunburst：旭日图

```python
 # 数据项1：
    data = [
    {
        "name": "Flora",
        "itemStyle": {"color": "#da0d68"},
        "children": [
            {"name": "Black Tea", "value": 1, "itemStyle": {"color": "#975e6d"}},
            {
                "name": "Floral",
                "itemStyle": {"color": "#e0719c"},
                "children": [{...},],
            },
        ],
    },
    {...}
    # 数据项2。
    data_pair: Sequence,
data = [
    opts.SunburstItem(
        name="Grandpa",
        children=[
            opts.SunburstItem(
                name="Uncle Leo",
                value=15,
                children=[...],
            ),
            opts.SunburstItem(...),
        ],),
    opts.SunburstItem(),
]
sunburst = (
    Sunburst(init_opts=opts.InitOpts(width="1000px", height="600px"))
    .add(series_name="", data_pair=data, radius=[0, "90%"])
    .set_global_opts(title_opts=opts.TitleOpts(title="Sunburst-基本示例"))
    .set_series_opts(label_opts=opts.LabelOpts(formatter="{b}"))
    .render("basic_sunburst.html")
)
```

### ThemeRiver

### WordCloud

## 直角坐标系图表

```python
几乎所有表的数据要求特征相同，一般为xaxis赋字符串列表，而使用一条或多条yaxis来展示值
1. markline
markline_opts=opts.MarkLineOpts(data=[opts.MarkLineItem(type_="average")]) # 
```

### 柱状图

```python
    # 系列名称，用于 tooltip 的显示，legend 的图例筛选。
    series_name: str,
    # 系列数据
    x/y_axis: Sequence[Numeric, opts.BarItem, dict],
```

### 热力图

```python
# Y 坐标轴数据
yaxis_data: types.Sequence[types.Union[opts.HeatMapItem, dict]] # 例: Faker.week
# 系列数据项
value: [[0, 0, 18], ...]types.Sequence[types.Union[opts.HeatMapItem, dict]]
```

### 箱形图

```python
v1 = [[850, 740, 900, 1070, 930, 850, 950, 980, 980, 880, 1000, 980],]
c = Boxplot()
c.add_xaxis(["expr1"])
c.add_yaxis("A", c.prepare_data(v1))
c.set_global_opts(title_opts=opts.TitleOpts(title="BoxPlot-基本示例"))
c.render()
```

### 散点图

```python
data = [[10.0, 8.04],...]
x_data = [d[0] for d in data]
y_data = [d[1] for d in data]
# 可能需要把global options中各轴的type设置为value
tooltip_opts=opts.TooltipOpts(is_show=True)
```

### 折线图

```python
    Line()
    .add_xaxis(Faker.choose())
    .add_yaxis("商家A", Faker.values(), areastyle_opts=opts.AreaStyleOpts(opacity=0.5))
    .add_yaxis("商家B", Faker.values(), areastyle_opts=opts.AreaStyleOpts(opacity=0.5))
    .set_global_opts(title_opts=opts.TitleOpts(title="Line-面积图"))
    .render()
```

### 层叠图

```python
line.overlap(scatter).render() # line = Line()... scatter = Scatter()...
```
