# Kelp and Sea Urchin Population Simulation

## Project overview

This Python project explores how sea urchin populations and kelp abundance might change over time, and how volunteer divers removing sea urchins could influence those changes. It compares two hypothetical scenarios: one without diver removal and one with volunteer divers removing a percentage of sea urchins each year.

Kelp forests provide habitat for many marine organisms, while sea urchins graze on kelp. The simulation is intended as an educational example of how population dynamics, grazing, and restoration efforts can be represented with a computational model. **It is not a prediction of actual kelp forest recovery or a model calibrated to field data.**

## What the program does

- Asks the user to choose the simulation duration, starting kelp abundance, starting sea urchin population, grazing strength, and removal effectiveness per diver.
- Accepts the number of volunteer divers through a prompt or the optional `--divers` argument.
- Models annual kelp growth, sea urchin grazing, sea urchin reproduction, natural mortality, and diver removal.
- Compares a baseline scenario (no removal) with a volunteer-diver scenario using the same starting conditions.
- Optionally prints year-by-year results in debug mode.
- Displays two comparison plots and saves them as `kelp_urchin_population_model.png`.

## Requirements

- Python 3
- Matplotlib

The script also uses `argparse`, which is included in Python's standard library.

Install Matplotlib with:

```bash
python -m pip install matplotlib
```

## Running the simulation

Save the Python code as `kelp_urchin_model.py`, then run:

```bash
python kelp_urchin_model.py
```

The program asks for model parameters before starting the simulation. You can also provide the number of volunteer divers from the command line:

```bash
python kelp_urchin_model.py --divers 3
```

To print the results for every simulated year:

```bash
python kelp_urchin_model.py --divers 3 --debug
```

**Note:** Even when using command-line arguments, the script still asks for the simulation duration, starting populations, grazing rate, and removal effectiveness. The displayed defaults in those prompts are suggestions; the current code requires you to enter a value rather than pressing Enter to accept a default. If `--debug` is not specified, the program asks whether to enable debug output.

### Example inputs

| Input | Example value |
| --- | ---: |
| Simulation duration | 30 years |
| Starting kelp abundance index | 30 |
| Starting sea urchin population | 10,000 |
| Grazing rate | 0.75 |
| Annual removal fraction per diver | 0.015 (1.5%) |
| Volunteer divers | 3 |

With these settings, the assumed annual removal fraction is **4.5%**, subject to the model's maximum annual removal fraction of 60%. This fraction is applied to the sea urchin population after modeled births and natural deaths each year.

## How the model works

The simulation advances in one-year steps. During each step:

1. Kelp grows according to a logistic growth equation.
2. Sea urchin grazing reduces kelp abundance.
3. Sea urchins reproduce based partly on the available kelp.
4. Natural mortality reduces the sea urchin population.
5. In the diver scenario, volunteers remove a specified fraction of the remaining urchins.
6. The program records the updated kelp and sea urchin values for plotting.

Both kelp abundance and sea urchin population are restricted to the range from zero to their respective modeled capacities. The kelp value is an **abundance index**, not a measured kelp count or biomass estimate.

### Model parameters

| Parameter | Value in code | Meaning |
| --- | ---: | --- |
| `KELP_CAPACITY` | 1,000 | Maximum kelp abundance index |
| `URCHIN_CAPACITY` | 100,000 | Maximum modeled sea urchin population |
| `KELP_GROWTH_RATE` | 0.40 | Annual kelp growth coefficient |
| `URCHIN_GROWTH_RATE` | 0.15 | Annual sea urchin reproduction coefficient |
| `URCHIN_DEATH_RATE` | 0.02 | Annual natural mortality coefficient |
| `MAX_REMOVAL_FRACTION` | 0.60 | Maximum annual diver removal fraction |

The simulation duration, starting values, grazing coefficient, and removal fraction per diver are supplied by the user when the program runs.

## Understanding the output

The program creates a figure with two plots:

- **Sea urchin population:** compares modeled sea urchin abundance with and without diver removal.
- **Kelp abundance:** compares modeled kelp abundance with and without diver removal.

The dashed gray line represents the scenario without volunteer removal. The colored line represents the scenario with divers. The figure is saved in the working directory as:

```text
kelp_urchin_population_model.png
```

If debug output is enabled, the terminal also shows each year's kelp index, estimated sea urchin population, and estimated number of urchins removed for both scenarios.

## Limitations

This is a simplified, hypothetical model. Its parameters are illustrative rather than estimates derived from field observations. It does not explicitly account for factors such as ocean temperature, disease, predators, recruitment from outside the modeled area, habitat differences, or changes in diver effort over time.

The model assumes each diver removes a fixed percentage of the sea urchins remaining each year, up to a combined maximum of 60%. Actual removal effectiveness would depend on many additional factors.

**Important code detail:** `KELP_CAPACITY` is set to 1,000, although the kelp plot's y-axis label currently says `Kelp abundance index (0–100)`. The numerical scale used by the simulation and graph is 0–1,000. Updating that label in the Python code would make the graph more accurate.

## Project purpose

This project demonstrates how Python can be used to explore ecological interactions and compare possible conservation strategies. By changing the starting conditions and diver-removal assumptions, users can investigate how different inputs influence the model's outcomes.

The results should be interpreted as **illustrative simulations**, not evidence that a particular number of divers will produce a specific restoration outcome in a real kelp forest.
