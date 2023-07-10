执行NoC映射 (物理映射)
======================

NoC映射的核心任务是根据片上网络的具体规格, 将 :py:data:`CTG` 中的每个逻辑Tile一一绑定到物理Tile, 完成逻辑-物理映射, 并根据物理Tile间的通信关系规划片上网络的路由路径, 因此又称作物理映射.

NoC映射流程介绍
----------------

NoC映射包括两个主要步骤:

+ 布局 (Layout)
    将 :py:data:`CTG` 中的每个逻辑Tile一一绑定到物理Tile, 不同的绑定方式会产生不同的Tile布局方式和通信分布方式.

+ 布线 (Routing)
    在布局结果上根据 :py:data:`CTG` 中的通信连接关系在两两物理Tile间规划路由路径.

.. important::

    + 虽然Tile映射背后的机理很复杂, 但是其流程简单, 而且计算方式是确定性的, 不需要太大的计算量, 所以仅靠一个 :py:data:`TileMapper` 便可实现.
    + 然而, NoC映射流程较多, 而且涉及到智能布局布线, 需要依靠复杂的智能优化算法, 占用庞大的计算量, 因此在 |name| 中, 设置了一个单独的模块 :ref:`nlrt` 来实现布局布线引擎, 并将 :ref:`nlrt` 的全部功能封装至 :py:data:`NocMapper` 中以方便用户使用.

通过 :py:data:`NocMapper` 实现NoC映射
-------------------------------------------

NoC映射的全部流程都被集成到 :py:data:`NocMapper` 这个 `class` 中, 用户只需要创建 :py:data:`NocMapper` 对象并调用其 :py:data:`NocMapper.run_map` 方法便可实现整个NoC映射流程.

下面是一个例子:

.. code-block:: python

    # 代码接上回
    ... 

    # 创建Tile阵列拓扑图, 设置阵列规模
    acg = ACG(6, 8)

    # 创建物理映射器
    nm = NocMapper(ctg, acg, **config)

    # 执行智能布局布线
    nm.run_layout()
    nm.run_routing()

    # 保存布局布线图
    nm.save_layout()
    nm.save_routing()

.. note:: 

    在创建 :py:data:`NocMapper` 时, 需要传入的位置参数有:

    + Tile映射得到的 :py:data:`CTG`
    + 表征NoC拓扑的 :py:data:`ACG`
    + NoC或Tile阵列的尺寸

    另外, 还可以通过一些关键字参数指定 :ref:`layout_engine` 和 :ref:`routing_engine`.
    详见 :py:data:`NocMapper.__init__`.


NoC映射结果说明
----------------

+ 布局结果被存放在 :py:data:`NocMapper.layout` 属性中, 作出的布局图被保存至 `./mapsave/your-mapname/layout` 目录.
+ 布线结果被保存在 :py:data:`NocMapper.routing` 属性中, 作出的布线图被保存至 `./mapsave/your-mapname/routing` 目录.

.. attention::

    为什么保存的布线图只有cast_path和merge_path, gather_path哪去了呢?

    在执行布局布线时, :py:data:`CTG` 中的cast数据流和gather数据流被合并到一起后进行布线规划, 
    merge数据流其实并不参与布线规划, 因为merge数据流采用一个单独的动态归约网络进行传输.

    所以保存目录中的 `cast_path.png` 才是真正的布线结果, 它包含了 :py:data:`CTG` 中的cast数据流和gather数据流. 
    `routing.gv.pdf` 和 `cast_path.png` 描述的是同一个东西, 只不过前者侧重展示通信冲突, 后者侧重展示路由路径, .




