## Issue:

```cmd
 File "src/latebind.pyx", line 39, in OpenGL_accelerate.latebind.LateBind.__call__
  File "src/wrapper.pyx", line 303, in OpenGL_accelerate.wrapper.Wrapper.__call__
  File "src/wrapper.pyx", line 88, in OpenGL_accelerate.wrapper.CArgCalculator.c_call
  File "src/wrapper.pyx", line 69, in OpenGL_accelerate.wrapper.CArgCalculatorElement.c_call
  File "src/wrapper.pyx", line 64, in OpenGL_accelerate.wrapper.CArgCalculatorElement.c_call
  File "src/arraydatatype.pyx", line 355, in OpenGL_accelerate.arraydatatype.SizedOutputOrInput.c_call
  File "src/arraydatatype.pyx", line 224, in OpenGL_accelerate.arraydatatype.ArrayDatatype.c_zeros
  File "src/arraydatatype.pyx", line 69, in OpenGL_accelerate.arraydatatype.HandlerRegistry.c_get_output_handler
  File "src/arraydatatype.pyx", line 80, in OpenGL_accelerate.arraydatatype.HandlerRegistry.c_handler_by_plugin_name
  File "D:\miniconda\envs\vtuber\lib\site-packages\OpenGL\plugins.py", line 18, in load
    return importByName( self.import_path )
  File "D:\miniconda\envs\vtuber\lib\site-packages\OpenGL\plugins.py", line 45, in importByName
    module = __import__( ".".join(moduleName), {}, {}, moduleName)
  File "D:\miniconda\envs\vtuber\lib\site-packages\OpenGL\arrays\numpymodule.py", line 28, in <module>
    from OpenGL_accelerate.numpy_formathandler import NumpyHandler
  File "src/numpy_formathandler.pyx", line 1, in init OpenGL_accelerate.numpy_formathandler
ValueError: ('numpy.dtype size changed, may indicate binary incompatibility. Expected 96 from C header, got 88 from PyObject', 1, <OpenGL.platform.baseplatform.glGenTextures object at 0x000001ECC04F2D40>)
```

```cmd
ValueError: ('numpy.dtype size changed, may indicate binary incompatibility. Expected 96 from C header, got 88 from PyObject', 1, <OpenGL.platform.baseplatform.glGenTextures object at 0x0000018644ED2DC0>)
```



## 环境:

```cmd
Python 3.10.14 | packaged by conda-forge | (main, Mar 20 2024, 12:40:08) [MSC v.1938 64 bit (AMD64)] on win32
Type "help", "copyright", "credits" or "license" for more information.
>>> import numpy
>>> numpy.__version__
'2.0.1'
>>> import OpenGL
>>> OpenGL.__version__
'3.1.7'
>>>
```



## 解决:

```python
pip install oldest-supported-numpy
```

