# PMM Planner for LASER

`pmm_planner` generates minimum-time trajectories for the LASER UAV System (LUS). Its C++ interface takes a commanded start state and a goal or waypoint list, then returns reference states with their relative times.

The planning manager calls this library when `planner: pmm` is selected. PMM handles trajectory generation; the manager handles mission execution, and control interpolates the references onto its prediction horizon. PMM does not check for obstacles.

## Build and use

From a ROS 2 workspace containing the LUS dependencies:

```bash
colcon build --packages-up-to laser_uav_managers
source install/setup.bash
```

Select `planner: pmm` in the planning-manager YAML before startup. That launch loads `params/<UAV_TYPE>.yaml`; use `pmm_planner_file` to select another file.

Send navigation goals and waypoint lists to `planning_manager`, as described in [LASER UAV Managers](https://github.com/LASER-Robotics/laser_uav_managers). The PMM library itself has no ROS subscriptions, publishers or services.

## C++ interface

Include `<pmm_planner/pmm_planner.hpp>` and construct `pmm_planner::PmmPlanner` with the vehicle and planner settings.

| Method | Purpose |
| --- | --- |
| `generateTrajectory(start, goal, speed, use_speed)` | Plans from a `laser_msgs/msg/ReferenceState` to one `PoseWithHeading`. |
| `generateTrajectory(start, waypoints, speed)` | Plans through a vector of `PoseWithHeading` waypoints. |
| `getTrajectory()` | Returns the sampled `ReferenceState` vector, including the final state. |
| `getTrajectoryTimes()` | Returns the corresponding relative sample times. |
| `setMass()` | Updates the vehicle mass used by the planner. |

The returned state and time arrays belong together. The planner has no execution clock or controller timestep. The planning manager supplies the calibrated mass before generating a segment.

## Configuration

The vehicle files in [params/](params/) use the `pmm_planner` key under `ros__parameters`.

| Parameter suffix | Meaning |
| --- | --- |
| `multirotor_parameters.max_accel` | Maximum thrust-to-mass acceleration in m/s². |
| `multirotor_parameters.max_vel` | Maximum trajectory speed in m/s. |
| `multirotor_parameters.default_vel` | Default mission speed in m/s when no override is requested. |
| `ltd_opt.use_drag` | Enables drag in limited-thrust decomposition. |
| `ltd_opt.thrust_decomp_acc_precision` | Convergence precision for thrust decomposition. |
| `ltd_opt.thrust_decomp_max_iter` | Maximum thrust-decomposition iterations. |
| `first_vel_opt.alpha`, `alpha_reduction_factor`, `alpha_min_threshold`, `max_iter` | Step size, reduction and stopping settings for the first velocity-optimization pass. |
| `second_vel_opt.run` | Enables the second optimization pass. |
| `second_vel_opt.alpha`, `alpha_reduction_factor`, `alpha_min_threshold`, `max_iter` | Equivalent settings for the second pass. |
| `time.dt_precision` | Trajectory-duration improvement threshold in seconds. |
| `time.sampling_step` | Output sample interval in seconds; supplied value is `0.01`. Keep it consistent with the planning setup. |

The vehicle model additionally needs mass, motor count, inertia and `G1`, provided by the manager's airframe file. Match acceleration and velocity limits to the vehicle calibration.

## Standalone demo

The original solver demo is available as `ros2 run pmm_planner main`. From the repository directory, run it with the standalone configuration and waypoints:

```bash
ros2 run pmm_planner main config/planner/planner_config.yaml config/waypoints/race.yaml
```

This demo uses the YAML structure in `config/planner/`, which differs from the manager-facing files in `params/`. It takes vehicle limits under `uav`, optimizer settings under `thrust_decomposition` and `velocity_optimization`, and output settings under `export`.

For a standalone build without the ROS integration:

```bash
make
./main config/planner/planner_config.yaml config/waypoints/race.yaml
```

Set `export.sampled_trajectory: true` to write a headerless CSV in `scripts/trajectory_data/`. `export.sampling_step` sets the interval and `export.sampled_trajectory_file` sets the filename. Columns are time, position, velocity and acceleration:

```text
t, p_x, p_y, p_z, v_x, v_y, v_z, a_x, a_y, a_z
```

Time is in seconds, position in metres, velocity in m/s and acceleration in m/s². With NumPy and Matplotlib installed, plot a saved result from the repository root:

```bash
python3 scripts/plot_trajectory.py scripts/trajectory_data/sampled_trajectory.csv
python3 scripts/plot_profiles.py scripts/trajectory_data/sampled_trajectory.csv
```

## Reference and license

K. Teissing, M. Novosad, R. Penicka and M. Saska, "Real-Time Planning of Minimum-Time Trajectories for Agile UAV Flight," IEEE Robotics and Automation Letters, 9(11), pp. 10351-10358, 2024. DOI: 10.1109/LRA.2024.3471388. [Paper](https://ieeexplore.ieee.org/document/10700666), [PDF](https://arxiv.org/pdf/2409.16074), [flight video](https://youtu.be/wArd536Amro).

```bibtex
@ARTICLE{teissing2024pmm,
  author={Teissing, Krystof and Novosad, Matej and Penicka, Robert and Saska, Martin},
  journal={IEEE Robotics and Automation Letters},
  title={Real-time Planning of Minimum-time Trajectories for Agile UAV Flight},
  year={2024},
  volume={9},
  number={11},
  pages={10351-10358},
  doi={10.1109/LRA.2024.3471388}
}
```

The solver retains its [GPL-3.0 license](LICENSE), copyright 2024 K. Teissing, M. Novosad, R. Pěnička and M. Saska, Faculty of Electrical Engineering, Czech Technical University in Prague. The LASER integration retains its [BSD 3-Clause notice](LICENSE.LASER). This is research software with no guarantee of fitness for a particular purpose.
