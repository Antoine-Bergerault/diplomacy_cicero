# CICERO for the SAILING Lab

The following is adapted for an MBZUAI HPC cluster using Qlustar OS.

### Accessing the cluster

1. Use a VPN client (e.g., FortiClient VPN) to join the MBZUAI network. Credentials are provided by the lab.
2. SSH to the cluster using your user credentials. Keep the connection open and use the shell to perform the following commands.

### Note on SLURM

SLURM is the workload manager, both CPU nodes and GPU nodes are available. The home directory is not meant to be used for running jobs. Please use the lustre file system (`/l/users/<user_id>`).

### Installation

Most of the code of the project implemented in Python with some parts in C++. The snippet below show how to install and build all required components within a conda environment on Ubuntu system. You would need C++ compiler with C++11 support. The original code was compiled using gcc 9.4.

```
# Move to the working directory
cd <working directory> # ex: cd /l/users/<user_id>/

# Clone the repository with submodules:
git clone --recursive https://github.com/Antoine-Bergerault/diplomacy_cicero.git diplomacy_cicero

# Create and activate a conda environment
conda create --yes -n cicero python=3.7
conda activate cicero

# Request a GPU node via SLURM
# Note: requesting 1 GPU will give faster access, it is sufficient for the installation.
srun -t 0:30:00 -N1 --gres=gpu:1 --pty /bin/bash -l

# Install pytorch, pybind11
conda install --yes pytorch=1.7.1 torchvision cudatoolkit=11.0 -c pytorch
conda install --yes pybind11

# Install go for boringssl in grpc
conda install --yes go protobuf=3.19.1

# Move cudNN to $CONDA_PREFIX/lib if necessary
# Note: cudNN should already be installed

# Get and compile glog
wget https://github.com/google/glog/archive/refs/tags/v0.6.0.tar.gz
tar -xf v0.6.0.tar.gz
cd glog-0.6.0
make

# Make glog available to conda
cp build/*.so $CONDA_PREFIX/lib
cp src/glog/*.h $CONDA_PREFIX/include/glog
cp build/glog/*.h $CONDA_PREFIX/include/glog

cd ../diplomacy_cicero

# Download all decrypted models
# Important: these are licensed materials, information on the original repository.
# The url is known by the SAILING Lab
wget <url of models directory>

# Install python requirements
pip install -r requirements.txt

# Local pip installs
pip install -e ./thirdparty/github/fairinternal/postman/nest/
pip install -e ./thirdparty/github/fairinternal/postman/postman/
pip install -e . -vv

# Check correct pytorch/torch installations
conda list torch # should not display "cpu_only"
python -c "import torch; print(torch.cuda.is_available())" # should print "True"

# Make sure Caffe2 will find cudNN
export PATH=$CONDA_PREFIX/lib:$PATH
export PATH=$CONDA_PREFIX/include:$PATH

# Make
make

# Run unit tests
make test_fast
```

After each pull it's recommended to run `make` to re-compile internal C++ and protobuf code.

### Simulating a basic game

In order to verify the installation, it is recommended to check if a basic simulation of the game can run. More specifically, we recommend the following:

````shell
python run.py --adhoc --cfg conf/c01_ag_cmp/cmp.prototxt Iagent_one=agents/cicero.prototxt use_shared_agent=1 power_one=TURKEY
````

This is a heavy task, we tried to run it using different sets of resources. For SLURM, we recommend requesting at least 1 GPU and 100GB of memory per CPU. As an example, here is how to request 3 GPUs for 20 hours and 120GB of memory per CPU.

````shell
srun -t 20:00:00 --mem-per-cpu=120000 -N1 --gres=gpu:3 --pty /bin/bash -l
````

We are currently running experiments to make this simulation easier. As for now, the above command has low chances of success, due to restricitions on resources accesses and on VPN connections.

## Licenses

See the [original Cicero repository](https://github.com/facebookresearch/diplomacy_cicero).

### References

- (Cicero) Meta Fundamental AI Research Diplomacy Team (FAIR)†, et al. ["Human-level play in the game of Diplomacy by combining language models with strategic reasoning."](https://www.science.org/doi/abs/10.1126/science.ade9097) Science 378.6624 (2022): 1067-1074.

- (Diplodocus) Bakhtin, Anton, et al. ["Mastering the Game of No-Press Diplomacy via Human-Regularized Reinforcement Learning and Planning."](https://arxiv.org/abs/2210.05492) arXiv preprint arXiv:2210.05492 (2022).

- (R2C2) Shuster, Kurt, et al. ["Language models that seek for knowledge: Modular search & generation for dialogue and prompt completion."](https://arxiv.org/abs/2203.13224) arXiv preprint arXiv:2203.13224 (2022).
