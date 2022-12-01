# A CUDA-based Solver for Linear Social MDP

This repository hosts the code for the paper [_Linear Social MDPs_](https://linear-social-mdp.github.io). The solver targets the two-agent grid world scenario in the paper and uses C++ with CUDA for high performance.

## Building

The build requirements:

- CMake with version at least 3.22 (older versions may work but not tested);
- A C++ compiler that supports C++20. Both Clang 12 and GCC 10 have been tested to work. MSVC should work too. But do not use GCC 11 because to our knowledge it conflicts with nvcc;
- A CUDA-capable GPU (an RTX 3080 would be decent). CUDA toolkit version at least 11.6 (older versions may work but not tested);
- The following libraries should be available to CMake: nlohmann JSON, fmt, SFML, and FTXUI. SFML is necessary only if you define `RENDER_WITH_SFML` in `visualize.cpp`; FTXUI is necessary only if `RENDER_WITH_FTXUI` is defined. We recommend managing these dependencies with vcpkg.

Build it the standard way you would for normal CMake projects. No extra flags are needed, though you might some extra configuration if you are running the code on a server and using your own copy of CUDA / compilers (when the system ones are too old).

## Configuration

By default, the programs reads its configuration from `config.json` under the current working directory. This can be changed by passing the path to the config json as a command line argument, i.e. `./SMDP-CUDA ../my-config/config.json`.

It is recommended to create a new config based on the sample `config.json` under this repository. The format:

```jsonc
{
  "nSides": 10, // Ths side length of the grid world, 10x10 here.
  "nIterations": 50, // The # of value iterations for one solve.
  "gamma": 0.95, // Gamma as in the conventional MDP formulation.
  "tau": 5, // Controls how rational an agent thinks the other agent is.
  "allGoals": [
    // Positions of all goals. Coordinates are 0-based.
    [1, 2], // [row, col]
    [1, 6]
  ],
  "agents": [
    // Two agents, A and B.
    {
      "goal": 0, // The goal as an index to the allGoals array above.
      "level": 2, // The level of the agent.
      "chi": 0, // The social goal, only useful if level = 2.
      "forcePGE": false, // Whether to force physical goal estimation (PGE) on this agent. By
      "forceSGE": false // default PGE & SGE are enabled only on-demand.
    },
    {
      "goal": 1,
      "level": 1,
      "chi": 1,
      "forcePGE": false,
      "forceSGE": false
    }
  ],
  // The initial state, i.e. the initial positions of [A, B, A's resource, B's resource].
  "initialState": [9, 2, 9, 5, 4, 5, 4, 7]
}
```

## Running the Solver

To run the solver, just run the binary you get from the build from **the root directory of the project**. E.g. assuming you followed the instructions in `README.md`, you should run
```bash
./build/SMDP_CUDA
```

**Note:** Please create a `records` folder under this repository manually before your first run of the solver:
```bash
mkdir records
```
This is where all the record JSON files will end up.

## Plotting
After running the solver a playback JSON file will be saved (you can see the file name and path at the end).

To create plots for a specific run, `cd` in to `analysis` and run `python plot.py --record <path_to_record> --output <output_directory>`.
For example: 
```bash
python plot.py --record ../records/A020B112G1216I92954547y0.95t5.json --output output
```
Creates plots of record `A020B112G1216I92954547y0.95t5.json` under the folder `output`.
