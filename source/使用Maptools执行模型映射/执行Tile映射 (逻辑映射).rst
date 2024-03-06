执行Tile映射 (逻辑映射)
========================

Tile映射指根据每个Tile的规格, 尤其是Xbar的规格, 将设备算子图的每个算子任务切分到一个或多个Tile中, 并得到每个Tile的配置信息以及所有Tile之间的通信关系, 这些信息通过 :py:data:`CTG` 表示.

经过Tile映射后, 可以获得当前算法所需的Tile的数量以及每个Tile的基本配置, 但是这时的Tile是逻辑Tile (:py:data:`LogicalTile`), 并没有被绑定至硬件中的物理的Tile (:py:data:`PhysicalTile`), 所以Tile映射又称作逻辑映射.

通过 :py:data:`TileMapper` 实现Tile映射
-------------------------------------------

Tile映射的全部流程都被集成到 :py:data:`TileMapper` 这个 `class` 中, 用户只需要创建 :py:data:`TileMapper` 对象并调用其 :py:data:`TileMapper.run_map` 方法便可实现整个Tile映射流程.

下面是一个例子:

.. code-block:: python

    # 代码接上回
    ... 

    # 创建Tile映射器, 设置每个Tile内的Xbar尺寸为 256 × 1280
    tm = TileMapper(oc.device_graph, 256, 256*5, **config)

    # 执行Tile映射
    tm.run_map()

    # 打印Tile映射信息
    tm.report_config()

    # 获得映射得到的CTG
    ctg = tm.ctg

    # 保存Tile映射得到的CTG图
    ctg.plot_ctg(direction='UD')


.. important::

    在创建 :py:data:`TileMapper` 时, 需要传入的位置参数有:

    + 模型转换得到的 :py:data:`DeviceGraph` 类型的设备算子图
    + Tile中Xbar的尺寸

    详见 :py:data:`TileMapper.__init__`.


.. attention::

    在创建Tile映射器时, 需要根据模型中卷积层的规模合理设置Xbar的尺寸, 若Xbar设置得太大, 可能会导致很多卷积层只能占用Xbar一部分, 从而导致Xbar资源利用率低; 若Xbar设置得太小, 可能会占用太多的Tile, 而片上网络提供的节点数是有一定的物理限制的, 一般来说一个拥有100个节点的片上网络就已经很大了.

    每次映射占用的Tile数量会在调用 :py:data:`TileMapper.print_config` 方法时被打印到终端 (最后一行), 建议每次映射后检查占用的Tile数量是否合理.

    当然, 实际的存算一体芯片中的Xbar尺寸肯定是确定的, 所以Xbar尺寸的设置也是一个双向的权衡问题, 需要综合考虑目标模型的卷积层规模, 资源利用率以及片上网络的节点限制.


.. note::

    为什么上面的脚本中设置的Xbar的宽和高不相等, 而且还差了5倍?

    这涉及到卷积层之间的通道传递特性, 当前卷积层的输出通道数等于下一个卷积层的输入通道数, 这意味着, 从整个神经网络的层面来看, 输出通道和输入通道其实是同一个东西. 
    
    根据Xbar的计算模式, 可以得到Xbar的高=输入向量长度=输入通道数×卷积核大小, Xbar的宽=输出向量长度=输出通道数, 如果将Xbar的宽和高设置相等, 其能承担的输入通道数量必然会小于输出通道数量, 导致Tile对输入输出向量占用能力的不平衡, 为了实现平衡, 必须在输入向量维度上花费更多的Tile, 但是我们完全可以在Tile内部做平衡, 做法就是增加每个Xbar的高, 这样有利于减小Tile的总数量, 节约片上网络硬件开销.

    那么每个Xbar的高应该增加到多大呢? 根据上面的算法, 最优的情况应该是将每个Xbar的高宽比设置为卷积核大小 (比如3×3卷积核, 高宽比应该设置为9), 但是不同模型不同卷积层的卷积和尺寸是变化的, 为了在这种变化的条件中取一个折衷, 现有的存算一体芯片大多将这个比值设置为4.5, |name| 暂时还不支持小数高宽比, 所以我们暂且建议将高宽比设置为5.


Tile映射结果说明
----------------

+ Tile映射得到的 :py:data:`CTG` 被存放在 :py:data:`TileMapper.ctg` 属性中, 供后续映射和仿真使用.

+ 作出的 :py:data:`CTG` 图被保存至 `./mapsave/your-mapname/ctg` 目录中.

+ Tile映射报告被打印至终端.

.. important:: 

    + :py:data:`CTG` 是极为重要的一个中间级表示, 它包含了每个逻辑Tile的配置信息 :py:data:`CTG.dicts`, 逻辑Tile之间的所有通信连接 (:py:data:`CTG.cast_comms`, :py:data:`CTG.merge_comms`, :py:data:`CTG.gather_comms`) 及连接图 (:py:data:`CTG.graph`).
    + 在作出的 :py:data:`CTG` 图中, 红色, 蓝色和紫色的连接分别表示cast, merge和gather三种数据流.
    + 在系统仿真时, :py:data:`CalcuSim` 和 :py:data:`TokSim` 都是基于 :py:data:`CTG` 运行的.
    + :py:data:`CTG` 中的每个逻辑Tile采用一个四维数据表示, 详见 :py:data:`LogicalTile`.
