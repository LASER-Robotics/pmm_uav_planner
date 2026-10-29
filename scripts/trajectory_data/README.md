# PMM trajectory output

The standalone PMM demo writes sampled trajectories here when `export.sampled_trajectory` is enabled in its planner YAML. `export.sampled_trajectory_file` sets the filename.

Each CSV has no header and uses these columns:

```text
t, p_x, p_y, p_z, v_x, v_y, v_z, a_x, a_y, a_z
```

Time is in seconds, position in metres, velocity in m/s and acceleration in m/s². Run `scripts/plot_trajectory.py` or `scripts/plot_profiles.py` from the repository root with the CSV path to visualize the result. See the [planner README](../../README.md) for example commands.
