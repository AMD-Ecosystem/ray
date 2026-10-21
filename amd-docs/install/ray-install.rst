.. meta::
  :description: installing Ray for ROCm
  :keywords: installation instructions, Docker, AMD, ROCm, Ray

.. _ray-on-rocm-installation:

********************************************************************
Ray on ROCm installation
********************************************************************

System requirements
====================================================================

To use Ray `2.59.0 <https://github.com/AMD-Ecosystem/ray/tree/release/2.59.0.amd0>`__ on ROCm, you need the following prerequisites:

- **ROCm version:** `10.1.0 <https://rocm.docs.amd.com/en/docs-10.1.0/>`__
- **Operating system:** Ubuntu 24.04
- **GPU platform:** AMD Instinct™ MI300X, MI325X, MI350X and MI355X
- **PyTorch:** `2.13.0 <https://github.com/ROCm/pytorch/tree/release/2.13>`__
- **Python:** `3.14 <https://www.python.org/downloads/release/python-3148/>`__
- **vLLM:** `0.29.0 <https://github.com/vllm-project/vllm/releases/tag/v0.29.0>`__

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

      git clone https://github.com/AMD-Ecosystem/ray.git -b release/2.59.0.amd0

2. Build the Docker container using the Dockerfile in the ``ray/docker`` directory:

   .. code-block:: bash

      cd ray
      docker build -f docker/Dockerfile.rocm -t my-rocm-ray .

3. Launch and connect to the container:

   .. code-block:: bash

      docker run --rm -it \
         --device=/dev/dri \
         --device=/dev/kfd \
         -p 8265:8265 \
         --group-add video \
         --cap-add=SYS_PTRACE \
         --security-opt seccomp=unconfined \
         --privileged \
         -v $HOME/.ssh:/root/.ssh \
         -v $HOME:$HOME \
         --shm-size 128G \
         -w $PWD \
         --name rocm_verl \
         my-rocm-ray \
         /bin/bash

   .. note::

      The ``--shm-size`` parameter allocates shared memory for the container. It can be adjusted based on your system's resources.

   .. warning::

      ``--privileged``, ``--cap-add=SYS_PTRACE``, and ``--security-opt seccomp=unconfined`` reduce container isolation.
      The command also mounts ``$HOME`` and ``$HOME/.ssh`` into the container.
      Use this configuration on a trusted development host.
      On a shared or production host, drop the flags that your workload does not require.

4. Verify the installed Ray version:

   .. code-block:: bash

      pip3 freeze | grep ray

   Expected output:

   .. code-block::

      ray @ https://s3-us-west-2.amazonaws.com/ray-wheels/master/c50286910448ba6005a85da1e22eca94bba38428/ray-3.0.0.dev0-cp314-cp314-manylinux2014_x86_64.whl#sha256=d6ae2b51cb649f0c4869cbd7d8688380fb26c61158d9b997e5657dfe97ae7b48

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
      | AMD-SMI            27.1.0+7da43026                                           |
      | amdgpu Version:    6.19.14.31400100                                          |
      | ROCm Version:      10.1.0                                                    |
      | VBIOS Version:     00160474                                                  |
      | FW PLDM:           01.25.04.02                                               |
      | Platform:          Linux Baremetal                                           |
      |-------------------------------------+----------------------------------------|
      | BDF                        GPU-Name | Mem-Uti   Temp   UEC       Power-Usage |
      | GPU  HIP-ID  OAM-ID  Partition-Mode | GFX-Uti    Fan               Mem-Usage |
      |=====================================+========================================|
      | 0000:09:00.0    AMD Instinct MI325X | 0 %      54 °C   0          139/1000 W |
      |   0       1       3        SPX/NPS1 | 0 %        N/A           283/262128 MB |
      |-------------------------------------+----------------------------------------|
      | 0000:1b:00.0    AMD Instinct MI325X | 0 %      64 °C   0          163/1000 W |
      |   1       3       2        SPX/NPS1 | 0 %        N/A           284/262128 MB |
      |-------------------------------------+----------------------------------------|
      | 0000:6b:00.0    AMD Instinct MI325X | 0 %      63 °C   0          171/1000 W |
      |   2       2       1        SPX/NPS1 | 0 %        N/A           283/262128 MB |
      |-------------------------------------+----------------------------------------|
      | 0000:79:00.0    AMD Instinct MI325X | 0 %      51 °C   0          142/1000 W |
      |   3       0       0        SPX/NPS1 | 0 %        N/A           283/262128 MB |
      |-------------------------------------+----------------------------------------|
      | 0000:89:00.0    AMD Instinct MI325X | 0 %      53 °C   0          138/1000 W |
      |   4       5       7        SPX/NPS1 | 0 %        N/A           283/262128 MB |
      |-------------------------------------+----------------------------------------|
      | 0000:9a:00.0    AMD Instinct MI325X | 0 %      62 °C   0          167/1000 W |
      |   5       7       6        SPX/NPS1 | 0 %        N/A           283/262128 MB |
      |-------------------------------------+----------------------------------------|
      | 0000:ea:00.0    AMD Instinct MI325X | 0 %      65 °C   0          173/1000 W |
      |   6       6       5        SPX/NPS1 | 0 %        N/A           283/262128 MB |
      |-------------------------------------+----------------------------------------|
      | 0000:f9:00.0    AMD Instinct MI325X | 0 %      55 °C   0          151/1000 W |
      |   7       4       4        SPX/NPS1 | 0 %        N/A           283/262128 MB |
      +-------------------------------------+----------------------------------------+
      +------------------------------------------------------------------------------+
      | Processes:                                                                   |
      |  GPU      PID  Process Name     GTT_MEM  VRAM_MEM  MEM_USAGE   CU %     SDMA |
      |==============================================================================|
      |  No running processes found                                                  |
      +------------------------------------------------------------------------------+

2. Install the required version of Ray with ROCm support using pip:

   .. code-block:: bash

      pip install -U \
         "ray[default,serve] @ https://s3-us-west-2.amazonaws.com/ray-wheels/master/c50286910448ba6005a85da1e22eca94bba38428/ray-3.0.0.dev0-cp314-cp314-manylinux2014_x86_64.whl"

3. Verify the installed Ray version:

   .. code-block:: bash

      pip3 freeze | grep ray

   Expected output:

   .. code-block::

      ray @ https://s3-us-west-2.amazonaws.com/ray-wheels/master/c50286910448ba6005a85da1e22eca94bba38428/ray-3.0.0.dev0-cp314-cp314-manylinux2014_x86_64.whl#sha256=d6ae2b51cb649f0c4869cbd7d8688380fb26c61158d9b997e5657dfe97ae7b48

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

   Expected output (for example, on the MI325X node):

   .. code-block:: shell-session

      {'object_store_memory': 20401094656.0, 'memory': 2411852390400.0, 'CPU': 384.0, 'node:10.194.129.123': 1.0, 'GPU': 8.0, 'accelerator_type:AMD-Instinct-MI325X-OAM': 1.0, 'node:__internal_head__': 1.0}
