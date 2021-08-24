# grid-monai

Grid.ai Session and Experiments with [Monai](https://monai.io) [tutorials](https://github.com/Project-MONAI/tutorials).
The MONAI framework is the open-source foundation being created by Project MONAI. MONAI is a freely available, community-supported, PyTorch-based framework for deep learning in healthcare imaging. It provides domain-optimized foundational capabilities for developing healthcare imaging training workflows in a native PyTorch paradigm.

# Grid.ai Session with Jupyter Notebook

## ssh into Grid.is session
  
```bash
grid ssh-keys add lit_key ~/.ssh/id_ed25519.pub
grid session ssh rslee-t2-xlarge # rslee-t2-medium fails and requires non-cached data loader (double check)
```

- setup Conda environment
```bash
conda create --yes --name monai python=3.8
conda activate monai # note you may get prompt to run `conda init bash && exit`
pip install ipykernel # allow usage with Jupyter notebook
python -m ipykernel install --user --name=monai # show as monai in Jupyter notebook
```

- Setup Linux SAR

SAR allows monitoring of CPU, RAM, IO, Network utilization.

```bash
sudo sed -ibak 's/ENABLED=\"false\"/ENABLED=\"true\"/g'  /etc/default/sysstat
sudo sed -ibak 's/^\(.*\) \(command -v debian-sa1\)/\* \* \* \* \* \2/g' /etc/cron.d/sysstat 
sudo service sysstat restart
sar 5 # confirm sar is running and Ctl-C to break
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
ipython profile create
cat > ~/.ipython/profile_default/startup/oo-monai.py <<EOF
import os
#os.environ['WANDB_MODE'] = "offline"
os.environ['WANDB_NOTEBOOK_NAME'] = "monai" # should be notebook file
os.environ['WANDB_API_KEY'] = "xxx"
EOF
```

## Setup Monai tutorial

- Download Monai tutorial
```bash
pip install monai
pip install -U pip
pip install -U matplotlib
pip install -U notebook
git clone https://github.com/Project-MONAI/tutorials.git
pip install -r tutorials/requirements.txt
```

- Run tutorial
```bash
cd tutorials/
```

- fix up "" in the leading directory that is interpreted as "/"
```bash
for f in $(find . -name "*.py" -print); do
  sed -ibak 's/data_path = os.sep.join(\[\"\"\,/data_path = os.sep.join(\[\"\.\"\,/g' $f
done
git reset # revert if not correct git reset --hard 
M	3d_classification/ignite/densenet_evaluation_array.py
M	3d_classification/ignite/densenet_evaluation_dict.py
M	3d_classification/ignite/densenet_training_array.py
M	3d_classification/ignite/densenet_training_dict.py
M	3d_classification/torch/densenet_evaluation_array.py
M	3d_classification/torch/densenet_evaluation_dict.py
M	3d_classification/torch/densenet_training_array.py
M	3d_classification/torch/densenet_training_dict.py
M	pathology/tumor_detection/ignite/camelyon_train_evaluate_nvtx_profiling.py
M	pathology/tumor_detection/ignite/profiling_camelyon_pipeline.ipynb
find . -name "*.pybak" -print -exec rm {} \;
```

- run examples that will download data and create checkpoint files required to run some of noteboooks
```bash
./runexamples.sh
```

- Setup Monai environ vars
```bash
# for python scripts
cat >> ~/.bashrc <<EOF
export MONAI_DATA_DIR="$(pwd)/workspace"
EOF
# for notebooks
ipython profile create
cat > ~/.ipython/profile_default/startup/oo-monai.py <<EOF
import os
os.environ['MONAI_DATA_DIR'] = "$(pwd)/workspace"
EOF
```