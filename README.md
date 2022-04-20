# grid-monai

Grid.ai Session and Experiments with [Monai](https://monai.io) [tutorials](https://github.com/Project-MONAI/tutorials).
The MONAI framework is the open-source foundation being created by Project MONAI. MONAI is a freely available, community-supported, PyTorch-based framework for deep learning in healthcare imaging. It provides domain-optimized foundational capabilities for developing healthcare imaging training workflows in a native PyTorch paradigm.

# Grid.ai Session with Jupyter Notebook

## ssh into Grid.is session
  
```bash
grid ssh-keys add lit_key ~/.ssh/id_ed25519.pub
grid session --instance_type 4_cpu_8gb ssh monai # t2.medium fails and requires non-cached data loader (double check)
tmux # 
```

- setup Conda environment
```bash
export CONDA_NAME=monai
conda create --yes --name $CONDA_NAME python=3.8
conda activate $CONDA_NAME # note you may get prompt to run `conda init bash && exit`
pip install ipykernel # allow usage with Jupyter notebook
python -m ipykernel install --user --name=$CONDA_NAME # show conda env in Jupyter notebook
ipython profile create
```

## Setup Monai tutorial

- Download Monai tutorial
```bash
# pip install monai
pip install -U pip matplotlib notebook sklearn # used by some of the notebooks although not listed in the docs
pip install -r https://raw.githubusercontent.com/Project-MONAI/MONAI/dev/requirements-dev.txt

git clone https://github.com/Project-MONAI/tutorials.git monai-tutorials
```

- download data and create checkpoint files required to run some of notebooks

This will take about an hour to finish.  use `screen` or `tmux` if connecting remote.
```bash
cd monai-tutorials/
./runexamples.sh
```

- Setup Monai environ vars
```bash
# for python scripts
cat >> ~/.bashrc <<EOF
export MONAI_DATA_DIRECTORY="$(pwd)/workspace"
EOF
# for notebooks
cat > ~/.ipython/profile_default/startup/00-monai.py <<EOF
import os
os.environ['MONAI_DATA_DIRECTORY'] = "$(pwd)/workspace"
EOF
```

# Optional for enhanced observability

- Setup [wandb](https://docs.wandb.ai/guides/track/advanced/environment-variables)

```bash
pip install wandb
# for python scripts
cat >> ~/.bashrc <<EOF
#export WANDB_MODE="offline"
export WANDB_NOTEBOOK_NAME="monai"
export WANDB_MODE="xxx"
EOF

# for notebooks
cat > ~/.ipython/profile_default/startup/00-wanb.py <<EOF
import os
#os.environ['WANDB_MODE'] = "offline"
os.environ['WANDB_NOTEBOOK_NAME'] = "monai" # should be notebook file
os.environ['WANDB_API_KEY'] = "xxx"
EOF
```

- fix to run on CPU as well
```bash
3d_segmentation/torch/unet_evaluation_dict.py
```


- Setup Linux SAR

SAR allows monitoring of CPU, RAM, IO, Network utilization.

```bash
sudo sed -ibak -e "s#SA_DIR=.*#SA_DIR=${HOME}#g" /etc/sysstat/sysstat 
diff /etc/sysstat/sysstat /etc/sysstat/sysstatbak

sudo cp /etc/sysstat/sysstatbak /etc/sysstat/sysstat

sudo sed -ibak -e 's/ENABLED=\"false\"/ENABLED=\"true\"/g'  /etc/default/sysstat
diff /etc/default/sysstat /etc/default/sysstatbak
sudo sed -ibak -e 's#^5-55/10#\*/1#g' /etc/cron.d/sysstat 
diff /etc/cron.d/sysstat /etc/cron.d/sysstatbak


PATH=$PATH:/usr/lib/sysstat:/usr/sbin:/usr/sbin:/usr/bin:/sbin:/bin

sudo service sysstat restart
sar 5 # confirm sar is running and Ctl-C to break
sar -f $HOME/sa07
```

- Setup dstat
```bash
sudo apt-get install dstat #install dstat
pip install nvidia-ml-py #install Python NVIDIA Management Library
pip install pynvml 
wget https://raw.githubusercontent.com/datumbox/dstat/master/plugins/dstat_nvidia_gpu.py
sudo mv dstat_nvidia_gpu.py /usr/share/dstat/ #move file to the plugins directory of dstat
dstat -a --nvidia-gpu # confirm dstat is running
```

- nvidia-smi dstat
sudo nvidia-smi daemon -d 60 -p $HOME
