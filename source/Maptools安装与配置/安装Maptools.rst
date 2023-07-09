安装 |name| 
==================

|name| 也是一个Python的包，但由于 |name| 仍在开发阶段，没有对应的发行版本，所以需要从源代码工程安装。

点击 `这里 <https://github.com/MiskaMoska/NVCIM-COMM>`_ 进入 |name| 的github主页，下载工程到本地。

如果你访问不了github，可以从gitee安装，点击 `这里 <https://gitee.com/cao-wenxu/nvcim-comm>`_ 进入 |name| 的gitee主页，这时注意选择cboost分支，然后下载工程并解压。

在Windows中安装 |name| 
------------------------

打开Powershell或cmd，进入刚下载的工程的根目录（包含有`setup.py`文件的那个目录）运行安装命令：

.. code-block::

    python setup.py install

即可完成 |name| 的安装。

在Linux/WSL中安装 |name| 
----------------------------

进入刚下载的工程的根目录（包含有 `setup.py` 文件的那个目录）运行安装命令：

.. code:: shell

    make maptools 

.. warning::

    编译报错?

    虽然 |name| 是一个python包，但是它里面的toksim模块的底层是用C++写的，所以在运行 `make  Maptools` 时会首先自动调用编译器编译toksim模块，在Linux/WSL环境下，默认选择gcc编译器，在Windows环境下，默认选择MSVC编译器，但亲测MSVC编译目前尚存在一些问题，所以当在Windows下安装 |name| 时，我们特意屏蔽了toksim的编译，也就是说在Windows下toksim就不能使用了，但是 |name| 中的其他模块和功能是能够正常使用的，而在Linux/WSL中， |name| 的所有功能均能够正常使用。

    因此，一旦在安装的过程中遇到编译错误，请检查自己的gcc是否能够正常工作.

.. note::

    访问权限报错?

    在make的过程，由于安装的包需要被写入python的site-packages目录，如果site-packages目录没有访问权限，会产生报错，这时建议根据报错提示找到site-packages目录的路径，然后使用chown和chmod修改该目录的所有者和权限

.. important::

    为了检查是否安装成功, 运行:

    .. code-block:: shell

        python

    进入python的交互界面，然后运行:

    .. code-block:: python

        import maptools 

    若不报错，则说明 |name| 安装成功.