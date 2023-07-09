执行Tile映射 (逻辑映射)
========================

Tile映射指根据每个Tile的规格、尤其是Xbar的规格，将设备算子图的每个算子任务切分到一个或多个Tile中，并得到每个Tile的配置信息以及所有Tile之间的通信关系，这些信息通过 :py:data:`CTG` 表示.

经过Tile映射后，可以获得当前算法所需的Tile的数量以及每个Tile的基本配置，但是这时的Tile是逻辑Tile (:py:data:`LogicalTile`)，并没有被绑定至硬件中的物理的Tile (:py:data:`PhysicalTile`), 所以Tile映射又称作逻辑映射.

通过 :py:data:`TileMapper` 实现Tile映射
-------------------------------------------

Tile映射的全部流程都被集成到 :py:data:`TileMapper` 这个 `class` 中, 用户只需要创建 :py:data:`TileMapper` 对象并调用其 :py:data:`TileMapper.run_map` 方法便可实现整个Tile映射流程.

下面是一个例子:

.. code-block:: python

    # 代码接上回
    ... 

    # 创建Tile映射器，设置Xbar尺寸为 64 × 320
    tm = TileMapper(oc.device_graph, 64, 64*5, **config)

    # 执行Tile映射
    tm.run_map()

    # 打印Tile映射信息
    tm.print_config()

    # 获得映射得到的CTG
    ctg = tm.ctg

    # 保存Tile映射得到的CTG图
    ctg.plot_ctg(direction='UD')

.. note::

    在创建 :py:data:`TileMapper` 时, 需要传入的位置参数有:

    + 模型转换得到的 :py:data:`DeviceGraph` 类型的设备算子图
    + Tile中Xbar的尺寸

    详见 :py:data:`TileMapper.__init__`.

Tile映射结果说明
----------------

+ Tile映射得到的 :py:data:`CTG` 被存放在 :py:data:`TileMapper.ctg` 属性中, 供后续映射和仿真使用.

+ 作出的 :py:data:`CTG` 图被保存 `./mapsave/your-mapname/ctg` 中.

+ Tile映射报告被打印至终端.

.. important:: 

    + :py:data:`CTG` 是极为重要的一个中间级表示, 它包含了每个逻辑Tile的配置信息 :py:data:`CTG.dicts`, 逻辑Tile之间的所有通信连接 (:py:data:`CTG.cast_comms`, :py:data:`CTG.merge_comms`, :py:data:`CTG.gather_comms`) 及连接图 (:py:data:`CTG.graph`).
    + 在作出的 :py:data:`CTG` 图中, 红色, 蓝色和紫色的连接分别表示cast, merge和gather三种数据流.
    + 在系统仿真时, :py:data:`CalcuSim` 和 :py:data:`TokSim` 都是基于 :py:data:`CTG` 运行的.
    + :py:data:`CTG` 中的每个逻辑Tile采用一个四维数据表示, 详见 :py:data:`LogicalTile`.