# Modification to Volumetric Grasping Network

Including new objects that simulate recyclables and added more robust grasp dataset generation

## Initially from:

```
@inproceedings{breyer2020volumetric,
 title={Volumetric Grasping Network: Real-time 6 DOF Grasp Detection in Clutter},
 author={Breyer, Michel and Chung, Jen Jen and Ott, Lionel and Roland, Siegwart and Juan, Nieto},
 booktitle={Conference on Robot Learning},
 year={2020},
}
```

## Installation

The following instructions were tested with `python3.8` on Ubuntu 20.04. A ROS installation is only required for visualizations and interfacing hardware. Simulations and network training should work just fine without. The [Robot Grasping](#robot-grasping) section describes the setup for robotic experiments in more details.

OpenMPI is optionally used to distribute the data generation over multiple cores/machines.

```
sudo apt install libopenmpi-dev
```

Clone the repository into the `src` folder of a catkin workspace.

```
git clone https://github.com/ethz-asl/vgn
```

Create and activate a new virtual environment.

```
cd /path/to/vgn
python3 -m venv --system-site-packages .venv
source .venv/bin/activate
```

Install the Python dependencies within the activated virtual environment.

```
pip install -r requirements.txt
```

Build and source the catkin workspace,

```
catkin build vgn
source /path/to/catkin_ws/devel/setup.zsh
```

or alternatively install the project locally in "editable" mode using `pip`.

```
pip install -e .
```

Finally, download the data folder [here](https://drive.google.com/file/d/1MysYHve3ooWiLq12b58Nm8FWiFBMH-bJ/view?usp=sharing), then unzip and place it in the repo's root.

## Data Generation

Generate raw synthetic grasping trials using the [pybullet](https://github.com/bulletphysics/bullet3) physics simulator.

```
python scripts/generate_data.py data/raw/foo --scene pile --object-set blocks [--num-grasps=...] [--sim-gui]
```

* `python scripts/generate_data.py -h` prints a list with all the options.
* `mpirun -np <num-workers> python ...` will run multiple simulations in parallel.

The script will create the following file structure within `data/raw/foo`:

* `grasps.csv` contains the configuration, label, and associated scene for each grasp,
* `scenes/<scene_id>.npz` contains the synthetic sensor data of each scene.

Clean the generated grasp configurations using the `data.ipynb` notebook.

Finally, generate the voxel grids/grasp targets required to train VGN.

```
python scripts/construct_dataset.py data/raw/foo data/datasets/foo
```

* Samples of the dataset can be visualized with the `vis_sample.py` script and `vgn.rviz` configuration. The script includes the option to apply a random affine transform to the input/target pair to check the data augmentation procedure.

## Network Training

```
python scripts/train_vgn.py --dataset data/datasets/foo [--augment]
```

Training and validation metrics are logged to TensorBoard and can be accessed with

```
tensorboard --logdir data/runs
```

## Simulated Grasping

Run simulated clutter removal experiments.

```
python scripts/sim_grasp.py --model data/models/vgn_conv.pth [--sim-gui] [--rviz]
```

* `python scripts/sim_grasp.py -h` prints a complete list of optional arguments.
* To detect grasps using GPD, you first need to install and launch the [`gpd_ros`](https://github.com/atenpas/gpd_ros) node (`roslaunch vgn gpd.launch`).

Use the `clutter_removal.ipynb` notebook to compute metrics and visualize failure cases of an experiment.