.. meta::
  :description: installing Ray for ROCm
  :keywords: installation instructions, Docker, AMD, ROCm, Ray

.. _ray-on-rocm-installation:

********************************************************************
Ray on ROCm installation
********************************************************************

System requirements
====================================================================

To use Ray `2.58.0 <https://github.com/AMD-Ecosystem/ray/tree/release/2.58.0>`__, you need the following prerequisites:

- **ROCm version:** `10.0.0 <https://rocm.docs.amd.com/en/docs-10.0.0/>`__
- **Operating system:** Ubuntu 24.04
- **GPU platform:** AMD Instinct™ MI300X, MI325X, and MI355X
- **PyTorch:** `2.12.0 <https://github.com/ROCm/pytorch/tree/release/2.12>`__
- **Python:** `3.14 <https://www.python.org/downloads/release/python-3147>`__
- **vLLM:** `0.27.0 <https://github.com/vllm-project/vllm/releases/tag/v0.27.0>`__

Install Ray
================================================================================

To install Ray on ROCm, you have the following options:

* :ref:`build-ray-rocm-docker-image`
* :ref:`install-rocm-ray-bare-metal`
* :ref:`build-rocm-ray-from-source`

.. _build-ray-rocm-docker-image:

Build your own Docker image
--------------------------------------------------------------------------------------

1. Clone the `https://github.com/AMD-Ecosystem/ray <https://github.com/AMD-Ecosystem/ray>`__ repository:

   .. code-block:: bash

      git clone https://github.com/AMD-Ecosystem/ray.git -b release/2.58.0

2. Build the Docker container using the Dockerfile in the ``ray/docker`` directory:

   .. code-block:: bash

      cd ray
      docker build -f docker/Dockerfile.rocm -t my-rocm-ray .

3. Launch and connect to the container:

   .. code-block:: bash

      docker run --rm -it --device /dev/dri --device /dev/kfd -p 8265:8265 --group-add video \
      --cap-add SYS_PTRACE --security-opt seccomp=unconfined --privileged -v $HOME/.ssh:/root/.ssh \
      -v $HOME:$HOME --shm-size 128G -w $PWD --name rocm_verl \
      my-rocm-ray /bin/bash

   .. note::

      The ``--shm-size`` parameter allocates shared memory for the container. It can be adjusted based on your system's resources.

4. Verify the installed Ray version:

   .. code-block:: bash

      pip3 freeze | grep ray

   Expected output:

   .. code-block::

      ray==2.58.0

.. _install-rocm-ray-bare-metal:

Install Ray on bare metal or a custom container
--------------------------------------------------------------------------------------

Follow these steps if you prefer to install ROCm manually on your host system or in a custom container.

1. Install ROCm. Follow the `ROCm installation guide <https://rocm.docs.amd.com/en/latest/deploy/linux/quick_start.html>`_ to install ROCm on your system.

   Once installed, verify your ROCm installation using:

   .. code-block:: bash

      amd-smi

   Expected output:

   .. code-block:: bash

      +------------------------------------------------------------------------------+
      | AMD-SMI            27.0.0+6b0e43f3                                           |
      | amdgpu Version:    7.1.0.0                                                   |
      | ROCm Version:      10.0.0                                                    |
      | VBIOS Version:     00182096                                                  |
      | FW PLDM:           00.25.06.05                                               |
      | Platform:          Linux Baremetal                                           |
      |-------------------------------------+----------------------------------------|
      | BDF                        GPU-Name | Mem-Uti   Temp   UEC       Power-Usage |
      | GPU  HIP-ID  OAM-ID  Partition-Mode | GFX-Uti    Fan               Mem-Usage |
      |=====================================+========================================|
      | 0000:1b:00.0    AMD Instinct MI300X | 0 %      43 °C   0           157/750 W |
      |   0       0       1        SPX/NPS1 | 0 %        N/A           285/196592 MB |
      |-------------------------------------+----------------------------------------|
      | 0000:3d:00.0    AMD Instinct MI300X | 0 %      43 °C   0           154/750 W |
      |   1       1       0        SPX/NPS1 | 0 %        N/A           285/196592 MB |
      |-------------------------------------+----------------------------------------|
      | 0000:4e:00.0    AMD Instinct MI300X | 0 %      42 °C   0           156/750 W |
      |   2       2       3        SPX/NPS1 | 0 %        N/A           285/196592 MB |
      |-------------------------------------+----------------------------------------|
      | 0000:5f:00.0    AMD Instinct MI300X | 0 %      43 °C   0           152/750 W |
      |   3       3       2        SPX/NPS1 | 0 %        N/A           285/196592 MB |
      |-------------------------------------+----------------------------------------|
      | 0000:9d:00.0    AMD Instinct MI300X | 0 %      43 °C   0           154/750 W |
      |   4       4       6        SPX/NPS1 | 0 %        N/A           286/196592 MB |
      |-------------------------------------+----------------------------------------|
      | 0000:bd:00.0    AMD Instinct MI300X | 0 %      44 °C   0           151/750 W |
      |   5       5       7        SPX/NPS1 | 0 %        N/A           285/196592 MB |
      |-------------------------------------+----------------------------------------|
      | 0000:cd:00.0    AMD Instinct MI300X | 0 %      43 °C   0           155/750 W |
      |   6       6       5        SPX/NPS1 | 0 %        N/A           285/196592 MB |
      |-------------------------------------+----------------------------------------|
      | 0000:dd:00.0    AMD Instinct MI300X | 0 %      40 °C   0           152/750 W |
      |   7       7       4        SPX/NPS1 | 0 %        N/A           285/196592 MB |
      +-------------------------------------+----------------------------------------+
      +------------------------------------------------------------------------------+
      | Processes:                                                                   |
      |  GPU      PID  Process Name     GTT_MEM  VRAM_MEM  MEM_USAGE   CU %     SDMA |
      |==============================================================================|
      |  No running processes found                                                  |
      +------------------------------------------------------------------------------+
   
2. Install the required version of Ray with ROCm support using pip:

   .. code-block:: bash

      pip install -U ray[default,serve]==2.58.0

3. Verify the installed Ray version:

   .. code-block:: bash

      pip3 freeze | grep ray
   
   Expected output:

   .. code-block::

      ray==2.58.0

.. _build-rocm-ray-from-source:

Build Ray from source
--------------------------------------------------------------------------------------

Follow the `Building Ray from source guide <https://docs.ray.io/en/latest/ray-contribute/development.html>`__ 
to build Ray with ROCm support from source.

.. _ray-verify-installation:

Test the Ray installation
======================================================================================

Ray unit tests are optional for validating your installation if you used a
prebuilt Docker image from AMD ROCm Docker Hub. To run unit tests manually and
validate your installation fully, follow these steps:

1. After launching the container, test whether Ray detects ROCm devices as expected.

   .. code-block:: bash
   
      python3 -c "import ray; ray.init(); print(ray.cluster_resources())"

2. If the setup is successful, the output should list all available ROCm devices.

   Expected output (for example, on the MI300X node):

   .. code-block:: shell-session

      {'memory': 1420360912896.0, 'GPU': 8.0, 'accelerator_type:AMD-Instinct-MI300X-OAM': 1.0, 'node:10.7.39.110': 1.0, 'CPU': 384.0, 'node:__internal_head__': 1.0, 'object_store_memory': 200000000000.0}

