---
title: "Simulating a Cyberattack Against the Texas Natural Gas Industry with ExaGO"
date: 2024-05-05
draft: false
---

*TLDR; View the pretty graph [here](https://exago.ned.vc). View a brief summary of our research [here](/tx-gas-report.pdf).*

# Introduction

The past semester I conducted research with Utah State University's Center for Anticipatory Intelligence, examining the resilience of the Texas natural gas industry. Alongside interviews with five subject matter experts from Baker Tilly, ZGlobal, Vistra Corp, and Pacific Northwest National Lab, we utilized [ExaGO](https://github.com/pnnl/ExaGO) to simulate the impact of cyberattacks on Texas critical infrastructure. This blog post will focus on the technical side of our simulations, rather than our overall analysis and findings. If you're interested in that analysis, feel free to check out [this brief summary](/tx-gas-report.pdf).

To get started, we'll need to install some software:

* [ExaGO](https://github.com/pnnl/ExaGO/), check out my [previous guide](https://ned.vc/posts/exago-for-dummies/)
* [MATLAB](https://www.mathworks.com/products/matlab.html), often available for free through university licensing agreements
* [MATPOWER](https://matpower.org/)

We will also need to [download sythetic grid data](https://electricgrids.engr.tamu.edu/electric-grid-test-cases/activsg2000/) to manipulate. We'll use an example based on the Texas Interconnection. A big thank you to the researchers at Texas A&M for their work.

After the the downloaded is finished, unzip it. I like to place the files in their own folder so I don't clutter my home directory.

```
unzip ACTIVSg2000.zip -d ~/ACTIVSg2000
```

# Prepare the Data

With MATLAB and MATPOWER installed, open the `ACTIVSg2000` directory. We'll use a simple MATLAB script to randomly turn off a percentage of the total natural gas power plants, simulating a sophisticated cyberattack on a significant natural gas pipeline or multiple gas-fired power plants.

```
mpc = case_ACTIVSg2000;
ng_indices = find(strcmp(mpc.genfuel, 'ng'));

% Calculate the number of generators to turn off
num_to_turn_off = round(length(ng_indices) * 0.5); 

% Randomly select indices of generators to turn off
turn_off_indices = ng_indices(randperm(length(ng_indices), num_to_turn_off));

% Set status of selected generators to 0 (offline)
mpc.gen(turn_off_indices, 8) = 0;

savecase('my_new_texas_case.m', mpc);
```

Run that script, and now you should have your scenario saved as `my_new_texas_case.m`

# Run ExaGO

A brief explanation of the command we're about to run. `opflow` finds the optimal AC power flow in the overall system, using some really sophisticated math that is way above my pay grade. We include the flag `-opflow_include_loadloss_variables` because with half of the natual gas plants turned off, we are necessarily going to have to shut off power to (i.e. [brownout](https://en.wikipedia.org/wiki/Brownout_(electricity))) over burdened areas. If we don't include this flag, opflow will fail because the problem is impossible to solve without shedding load. `-netfile` and `-gicfile` provide information on grid infrastructure and geospatial location, respectively. Finally, we save the ouput in JSON format to be used in the visualization later.

```
source spack/share/spack/setup-env.sh
cd ~/ExaGO
opflow -opflow_include_loadloss_variables -netfile ~/ACTIVSg2000/my_new_texas_case.m -gicfile ~/ACTIVSg2000/ACTIVSg2000_GIC_data.gic -save_output -opflow_output_format JSON 
```

After that's finished running, you should have a new file named `opflowout.json` in the `ExaGO` directory.

# Visualize the results

It's time to get to the fun part, visualizing our results!

One slightly annoying bug we'll need to deal with first, there will be an extra comma in opflowout.json. I'm not sure why this is the case and [I've opened a bug report](https://github.com/pnnl/ExaGO/issues/134). Let's edit `opflowout.json` real quick.

```
vim opflowout.json
# Shift+G to jump to the end of the file
# Delete the extra comma
# :wq to save and exit
cp opflowout.json ~/ExaGO/viz/data
```

The remaining instructions are basically copy+pasted from the [ExaGO visualization](https://github.com/pnnl/ExaGO/blob/develop/viz/README.md) and [Node.js](https://nodejs.org/en/download/package-manager) documentation.

```
# installs NVM (Node Version Manager)
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash

# download and install Node.js
nvm install 16.13.0
```

One last step, open a new terminal window:

```
cd ~/ExaGO/viz
npm install --legacy-peer-deps
python3 geninputfile.py opflowout.json
npm start
```

Hooray! Your visualization should be available at http://localhost:8080. If you want to see my example, take a look at https://exago.ned.vc (please be patient as it loads, ExaGO was never really meant to be exposed to the public internet so I've got a really janky reverse proxy serving the files). Make sure to check the boxes for `Voltage` and `Generation Power` to see the pretty colors.

I encourage you to mess around and have fun with the MATLAB files and ExaGO, turning off different power plants and testing out different grids. There are a number of other datasets available from Texas A&M researchers, available [here](https://electricgrids.engr.tamu.edu/electric-grid-test-cases/). Although this project has made me sleep a little bit more uneasy at night knowing how vulnerable our critical infrastructure is, it's been very fun to geek out and learn about the confluence of national security, operational technology, and my home state. Thanks for reading!