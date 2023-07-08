core
==========

`core` 目录中存放着Maptools的核心数据类型, 数据结构和中间级表示等.

数据类型
---------

.. py:data:: LogicalTile
   :value: Tuple[int, int, int, int]

   逻辑Tile, 以 `[layer, cluster, block, index]` 的格式表示, 其中:

   + `layer` 表示该Tile负责的层ID；
   + `cluster` 表示该Tile所在的 `cluster` 的ID， `cluster` 为某一层中负责计算相同的卷积输出通道范围的Tile组成的一个集合；
   + `block` 表示该Tile所在的 `block` 的ID， `block` 为某一个 `cluster` 中负责计算相同的卷积输入通道范围的Tile组成的一个集合；
   + `index` 表示该Tile在所在 `block` 中的索引.

.. py:data:: PhysicalTile
   :value: Tuple[int, int]

   物理Tile, 以 `[x_pos, y_pos]` 的格式表示, 其中:

   + `x_pos` 表示该Tile的x坐标;
   + `y_pos` 表示该Tile的y坐标.

.. py:data:: ArbitaryEdge
   :value: Tuple[PhysicalTile, PhysicalTile]

   任意边, 可以是任意两个Tile之间的边


.. py:data:: MeshEdge
   :value: Tuple[PhysicalTile, PhysicalTile]

   Mesh边, 必须是Mesh网络中的有效边


.. py:class:: NNModelArch(enum.Enum)

   神经网络模型架构

   .. attribute:: VGG 

      VGG结构

   .. attribute:: RESNET  

      ResNet结构

   .. attribute:: GOOGLENET

      GoogleNet结构

   .. attribute:: YOLO_V3   

      YOLOv3结构

   .. attribute:: SQUEEZENET
      
      SqueezeNet结构


.. py:class:: DLEMethod(enum.Enum)

   :py:class:`DLE` 方法

   .. attribute:: REVERSE_S

      按照Reverse-S的顺序进行映射


.. py:class:: DREMethod(enum.Enum)

   :py:class:`DRE` 方法

   .. attribute:: DyXY
    
      按照DYXY路由进行多播规划 (包含随机因素)

   .. attribute:: RPM
    
      采用RPM多播规划方法

   .. attribute:: OCR
    
      采用OCR多播规划方法


数据结构
---------

.. py:data:: OperatorConfig
   :value: Dict[str, Any]
   
   算子配置信息
   
.. csv-table:: `OperatorConfig` 中的配置信息
    :header: "Key", "类型", "说明"
    :widths: 15, 10, 30

    "op_type",              `str`,                                      "当前算子的操作类型"
    "conv_kernel_size",     "`List[int, int]`",                         "卷积核尺寸, `[Height, Width]`"
    "conv_pads",            "`List[int, int, int, int]`",               "卷积padding尺寸, `[Top, Right, Bottom, Left]`"
    "conv_input_size",      "`List[int, int]`",                         "卷积输入尺寸, 不含padding, `[Height, Width]`"
    "conv_output_size",     "`List[int, int]`",                         "卷积输出尺寸, `[Height, Width]`"
    "conv_strides",         "`List[int, int]`",                         "卷积滑动步长, `[Height, Width]`"
    "conv_weights",         "`str`",                                    "卷积权重的指针, 作为 :py:class:`ModelParams` 的 `key` 使用"
    "conv_bias",            "`str`",                                    "卷积偏置的指针, 作为 :py:class:`ModelParams` 的 `key` 使用"
    "act_mode",             "`Literal['Relu','PRelu','HardSigmoid']`",  "激活类型"
    "pool_mode",            "`Literal['MaxPool','AveragePool']`",       "池化类型"
    "pool_kernel_size",     "`List[int, int]`",                         "池化窗口尺寸, `[Height, Width]`"
    "pool_pads",            "`List[int, int, int, int]`",               "池化padding尺寸, `[Top, Right, Bottom, Left]`"
    "pool_input_size",      "`List[int, int]`",                         "池化输入尺寸, 不含padding, `[Height, Width]`"
    "pool_output_size",     "`List[int, int]`",                         "池化输出尺寸, `[Height, Width]`"
    "pool_strides",         "`List[int, int]`",                         "池化滑动步长, `[Height, Width]`" 
    "resize_scales",        "`List[1, 1, int, int]`",                   "上采样系数, `[Batch, Channel, Height, Width]`"
    "conv_quant_config",    ":py:class:`OperatorQuantConfig`",          "卷积量化配置信息"
    "act_quant_config",     ":py:class:`OperatorQuantConfig`",          "激活量化配置信息"
    "sum_quant_config",     ":py:class:`OperatorQuantConfig`",          "加和量化配置信息"
    

.. note::

    `OperatorConfig` 变量并不一定拥有上述所有的 `key`,
    比如当该算子不执行池化操作时, 则对应的 `OperatorConfig` 不包含与池化有关的 `key`.

.. important::

    操作类型是由 `Conv` (卷积), `Add` (加和) , `Act` (激活), `Pool` (池化), `Bias` (偏置) 和 `Rsz` (上采样) 五个基本操作组合而成的，比如 `Conv-Act-Pool`， `Conv-Pool`， `Conv-Add-Act-Bias`，当然也可以是单独的 `Conv`。
    不管四个基本操作的组合顺序如何，其在硬件上的执行顺序永远是 `Conv->Bias->Add->Act->Pool->Rsz`。 
    
    op_type至少包含 `Conv`，也就是说，每个算子都要执行卷积运算。


.. py:data:: TileConfig
   :value: Dict[str, Any]
   
   Tile配置信息, 与算子配置信息类似, 但增加了一些额外的与Tile有关的配置信息.

.. csv-table:: `TileConfig` 相对于 `OperatorConfig` 增加的配置信息
    :header: "Key", "类型", "说明"
    :widths: 15, 10, 30

    "xbar_icfg",            :py:data:`InputChannelConfig`,              "Tile中的Xbar输入通道配置"
    "xbar_ocfg",            :py:data:`OutputChannelConfig`,             "Tile中的Xbar输出通道配置"
    "xbar_num_ichan",       "`int`",                                    "Tile中的Xbar负责的卷积输入通道的个数"
    "xbar_num_ochan",       "`int`",                                    "Tile中的Xbar负责的卷积输出通道的个数"
    "box_idx",              "`int`",                                    "box索引, 专门用于Concat算子的处理"
    "'tqc",                 ":py:data:`TileQuantConfig`",               "Tile量化配置信息"

.. attention:: 

    注意Tile的 `op_type` 与其所负责的算子的 `op_type` 并不一定相同, 因为同一个算子可能被切分部署到多个Tile, 每个Tile负责的操作可能会有一些细微的差别.

   
.. py:data:: ModelParams
   :value: Dict[str, numpy.ndarray]
   
   模型参数, 主要包括卷积权重和卷积偏置.

.. py:class:: OperatorQuantConfig(object)
    
    算子量化配置信息

.. py:class:: TileQuantConfig(object)
    
    Tile量化配置信息


中间级表示
----------

.. py:class:: OperatorGraph(object)

    算子图的基类

.. py:class:: OriginGraph(object)

    原始模型算子图, 在 :py:class:`OperatorGraph` 的基础上扩展了图分配方法, 将原始模型算子图分配到主机算子图和设备算子图.

.. py:class:: HostGraph(object)

    主机算子图

.. py:class:: DeviceGraph(object)

    设备算子图

.. py:class:: CTG(object)

    通信追踪图 (Communication Trace Graph)

.. py:class:: ACG(object)

    架构特征图 (Architecture Characterization Graph)
