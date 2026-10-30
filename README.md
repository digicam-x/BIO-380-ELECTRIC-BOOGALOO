# BIO-380-ELECTRIC-BOOGALOO


import argparse  # Lets you change settings when you run the script.
import matplotlib.pyplot as plt  # Lets the script create the graph.
YEARS = 30  # Sets how many years the model will simulate.
KELP_CAPACITY = 100.0  # Sets the maximum kelp abundance index.
URCHIN_CAPACITY = 60.0  # Sets the maximum urchin abundance index.
STARTING_KELP = 45.0  # Sets the kelp index at the beginning of the model.
STARTING_URCHINS = 50.0  # Sets the urchin index at the beginning of the model.
KELP_GROWTH_RATE = 0.45  # Controls how quickly kelp grows when there is room.
URCHIN_GROWTH_RATE = 0.20  # Controls urchin reproduction when food is available.
URCHIN_DEATH_RATE = 0.08  # Represents natural urchin deaths each year.
GRAZING_RATE = 0.90  # Controls how strongly urchins reduce kelp through grazing.

def simulate(removal_fraction, scenario_name, debug):  # Runs one version of the model.
    kelp = STARTING_KELP  # Starts kelp at the chosen initial index.
    urchins = STARTING_URCHINS  # Starts urchins at the chosen initial index.
    years = [0]  # Stores the starting year for the graph.
    kelp_history = [kelp]  # Stores the starting kelp value.
    urchin_history = [urchins]  # Stores the starting urchin value.

    if debug:  # Checks whether debug mode was turned on.
        print(f"\nDebug output: {scenario_name}")  # Labels the values printed below.

    for year in range(1, YEARS + 1):  # Repeats the model once for each year.
        kelp_growth = KELP_GROWTH_RATE * kelp * (1 - kelp / KELP_CAPACITY)  # Calculates new kelp growth.
        grazing_loss = GRAZING_RATE * kelp * (urchins / 100)  # Calculates kelp eaten by urchins.
        kelp = kelp + kelp_growth - grazing_loss  # Updates kelp after growth and grazing.
        kelp = max(0.0, min(KELP_CAPACITY, kelp))  # Keeps kelp between zero and its capacity.

        food_available = kelp / KELP_CAPACITY  # Estimates how much kelp is available to urchins.
        urchin_births = URCHIN_GROWTH_RATE * urchins * food_available * (1 - urchins / URCHIN_CAPACITY)  # Estimates new urchins.
        natural_deaths = URCHIN_DEATH_RATE * urchins  # Estimates urchins lost to natural mortality.
        urchins = urchins + urchin_births - natural_deaths  # Updates urchins before removal.
        urchins = urchins * (1 - removal_fraction)  # Removes the selected fraction of urchins.
        urchins = max(0.0, min(URCHIN_CAPACITY, urchins))  # Keeps urchins between zero and their capacity.

        years.append(year)  # Saves the year for the graph.
        kelp_history.append(kelp)  # Saves this year's kelp value.
        urchin_history.append(urchins)  # Saves this year's urchin value.

        if debug:  # Prints yearly values only when debug mode is on.
            print(f"Year {year:2d} | Kelp index: {kelp:6.2f} | Urchin index: {urchins:6.2f}")  # Shows the model's yearly state.

    return years, kelp_history, urchin_history  # Sends the results back for graphing.

def main():  # Defines the steps the program runs when started.
    parser = argparse.ArgumentParser(description="Simulate kelp and urchin changes over time.")  # Creates the command-line options.
    parser.add_argument(  # Adds the adjustable urchin-removal setting.
        "--removal-percent",  # Names the setting used when running the script.
        type=float,  # Reads the setting as a number.
        default=20,  # Uses 20 percent removal if no value is entered.
        help="Annual percent of urchins removed; for example, 20 means 20 percent.",  # Explains the setting.
    )  # Finishes adding the removal setting.
    parser.add_argument("--debug", action="store_true", help="Print each year's model values.")  # Adds the optional debug switch.
    args, _ = parser.parse_known_args()  # Reads the settings entered when the script starts.

    if not 0 <= args.removal_percent <= 100:  # Checks that the removal percentage is valid.
        parser.error("Enter a removal percentage from 0 to 100.")  # Stops and explains the valid range.

    removal_fraction = args.removal_percent / 100  # Converts a percent such as 20 into 0.20.
    years, kelp_baseline, urchin_baseline = simulate(0, "No removal", args.debug)  # Runs the baseline with no removal.
    _, kelp_managed, urchin_managed = simulate(removal_fraction, f"{args.removal_percent:g}% removal", args.debug)  # Runs the chosen removal scenario.

    figure, (kelp_axis, urchin_axis) = plt.subplots(2, 1, figsize=(9, 7), sharex=True)  # Creates two graph panels with the same time axis.
    kelp_axis.plot(years, kelp_baseline, linestyle="--", label="No removal")  # Draws baseline kelp values.
    kelp_axis.plot(years, kelp_managed, label=f"{args.removal_percent:g}% annual removal")  # Draws kelp values with the chosen removal rate.
    kelp_axis.set_ylabel("Kelp abundance index")  # Labels the kelp graph's vertical axis.
    kelp_axis.set_ylim(0, KELP_CAPACITY)  # Keeps the kelp graph within its model limits.
    kelp_axis.legend()  # Shows which line represents each scenario.
    kelp_axis.grid(True, alpha=0.3)  # Adds a light grid to help read kelp values.

    urchin_axis.plot(years, urchin_baseline, linestyle="--", label="No removal")  # Draws baseline urchin values.
    urchin_axis.plot(years, urchin_managed, label=f"{args.removal_percent:g}% annual removal")  # Draws urchin values with the chosen removal rate.
    urchin_axis.set_xlabel("Year")  # Labels the graph's horizontal axis.
    urchin_axis.set_ylabel("Urchin abundance index")  # Labels the urchin graph's vertical axis.
    urchin_axis.set_ylim(0, URCHIN_CAPACITY)  # Keeps the urchin graph within its model limits.
    urchin_axis.legend()  # Shows which line represents each scenario.
    urchin_axis.grid(True, alpha=0.3)  # Adds a light grid to help read urchin values.

    figure.suptitle("Kelp Forest Comeback: No Removal vs. Urchin Removal")  # Adds a title for both graphs.
    figure.tight_layout(rect=[0, 0, 1, 0.95])  # Adjusts spacing so the title and graphs fit.
    figure.savefig("kelp_urchin_comparison.png", dpi=200)  # Saves the graph for your presentation.
    plt.show()  # Displays the graph at the end of the script.

if __name__ == "__main__":  # Checks whether this file was run directly.
    main()  # Starts the program.
    
