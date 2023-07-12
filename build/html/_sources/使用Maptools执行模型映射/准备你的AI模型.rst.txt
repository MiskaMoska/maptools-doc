准备你的AI模型
=================

|name| 端到端映射的起点是onnx模型, onnx是一种神经网络通用的算子图格式, 它包含了能够支持神经网络运行的全部信息.
几乎所有的深度学习框架都支持onnx格式, 因此,  |name| 拥有较强的通用性.

获取onnx模型
----------------

通过云盘获得onnx模型
~~~~~~~~~~~~~~~~~~~~

点击 `这里 <https://pan.baidu.com/s/1IeAx3D7Die9q7LmOjIEC-A?pwd=ktv2>`_ 获取现成的onnx模型.


自己动手导出onnx模型
~~~~~~~~~~~~~~~~~~~~

授人以鱼不如授人以渔, 若要将自己训练好的模型导出到的onnx文件, 可以使用torch的onnx工具, 下面展示了一个导出torchvision官方提供的resnet18模型到onnx文件的例子. 

编写python脚本: 

.. code-block:: python

    import torch
    import torchvision as tv
    import torch.onnx as ox

    model = tv.models.resnet18() #可以换成你自己的模型

    ox.export(
        model,
        torch.randn([1,3,1000,224]), #输入数据尺寸
        "onnx_models/resnet18.onnx", #输出路径, 自己定义
        opset_version=11,
        input_names=["input"],
        output_names=["output"]
    )

接下来需要使用 `onnxsim` 工具进行模型简化, 比如将卷积层和BN融合. 

运行下面的命令安装 `onnxsim` : 

.. code-block:: shell

    pip install onnx-simplifier==0.4.17

.. important::

    注意这里必须指定安装 `onnxsim` 的0.4.17版本, 因为其他版本亲测有bug, 而0.4.17是比较稳定的

运行下面的命令进行模型简化: 

.. code-block:: shell

    onnxsim a.onnx b.onnx

其中 `a.onnx` 是待简化的模型路径,  `b.onnx` 是简化后的模型的保存路径

.. warning::

    切记在对一个onnx模型进行映射之前, 一定要先让你的onnx模型文件过一遍 `onnxsim` ,  `onnxsim` 不仅能够简化模型结构, 还能补全每层的尺寸信息, 只有经过简化和补全的onnx模型才是 |name| 所能识别的有效输入

从github上获取onnx模型
~~~~~~~~~~~~~~~~~~~~~~

onnx官方提供了一些常见神经网络的onnx模型, 发布在github上, 点击 `这里 <https://github.com/onnx/models>`_ 下载, 记得下载得到的模型同样要使用 `onnxsim` 进行简化.

.. note::

    注意从github上获取的onnx模型并不一定能用于 |name| 映射, 即使经过了onnxsim简化, 因为这些模型的算子图结构可能会千奇百怪, 可能会超出 |name| 能够识别的范围, 
    所以建议在执行模型映设置前先使用 `netron` 工具检查onnx模型的结构是否合理, 请参照 :ref:`netron_check`.

.. _netron_check:

查看onnx模型
------------

可以使用 `netron` 查看onnx模型的结构和参数,  `netron` 是一款神经网络可视化工具, 点击 `这里 <https://netron.app/>`_ 使用 `netron` 的在线版, 然后选择你要查看的onnx模型即可. 





