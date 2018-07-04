Google S2 Geometry Library
----
useful links:
- [S2Geometry](http://s2geometry.io/)
- [google/s2geometry](https://github.com/google/s2geometry), c++ implementations with a python api supported by swig, a repo on github
- [高效的多维空间点索引算法 — Geohash 和 Google S2](https://halfrost.com/go_spatial_search/), an introdutioin to geohash and s2 in Chinese. 
- [Google S2 with Python & Jupyter](https://blog.nobugware.com/post/2018/google-s2-python-jupyter/). [python-visualization/folium] is used to visualize a map demo application


## installation
Refer to [S2 Installation](http://s2geometry.io/about/platforms). 

Some tricks:
- Prerequisite setting up: as python support wanted on ubuntu 16.04 system, I mainly followed the linux part. Make sure SWIG library is installed for python support.
- Compiling:
    - `DGTEST_ROOT` is the root directory for GoogleTest, `/usr/src/gtest` on Linux systems
    ```shell
    $ cmake -DWITH_GFLAGS=ON -WITH_GTEST=ON -DGTEST_ROOT=/usr/src/gtest ..
    ```
    
    - Make sure the `PythonInterp` and `PythonLibs` version consistent. There may still be some binding issues of the CMake setting files of the repo currently, c.f., [#6](https://github.com/google/s2geometry/issues/6) and [#34](https://github.com/google/s2geometry/pull/34). If following happens where the interpreter version is `2.7.12` and the library is `3.6.5`, `pywraps2` module will not be imported correctly 
    ```shell
    $ make
    ...
    -- Found PythonInterp: /usr/bin/python (found version "2.7.12") 
    -- Found PythonLibs: /usr/lib/x86_64-linux-gnu/libpython3.6m.so (found version "3.6.5") 
    ...
    ```
    One possible way to solvle this is to create a virtual environment for python and then do `make`.
    ```shell
    $ source ~/py-env/bin/activate
    ```
    Then do `make test` and `pywraps2_test` will pass.
    
    - C.f., [libs2.so import error solution on stckoverflow](https://stackoverflow.com/questions/45439754/importerror-libs2-so-cannot-open-shared-object-file-no-such-file-or-directory). 
    Because of ubuntu system, change the `CMAKE_INSTALL_PREFIX:PATH=/usr` in the `CMakeCache.txt` file after do `make`, then do `sudo make install`.
    Copy module files to the virtual environment.
    ```shell
    $ cp /usr/lib/python3.6/site-packages/pywraps2.py ~/py-env/lib/python3.6/site-packages
    $ cp /usr/lib/python3.6/site-packages/_pywraps2.so ~/py-env/lib/python3.6/site-packages
    ```
    Then `pywraps2` can be imported in the virtual environment.
