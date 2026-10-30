import argparse  # Lets you set model options with arguments when running a script.
import matplotlib.pyplot as plt  # Lets the program draw and save the graphs.

YEARS = 30  # Sets how many years the model will simulate.
KELP_CAPACITY = 100.0  # Sets the maximum kelp abundance index.
URCHIN_CAPACITY = 100.0  # Sets the maximum urchin abundance index.
STARTING_KELP = 30.0  # Sets the initial kelp index for this example.
STARTING_URCHINS = 70.0  # Sets the initial urchin index for this example.
KELP_GROWTH_RATE = 0.40  # Controls how quickly kelp grows when there is space.
URCHIN_GROWTH_RATE = 0.15  # Controls how quickly urchins reproduce when food is available.
URCHIN_DEATH_RATE = 0.02  # Represents natural urchin deaths each year.
GRAZING_RATE = 0.75  # Controls how strongly urchins reduce kelp through grazing.
REMOVAL_PER_DIVER = 0.015  # Assumes each diver removes 1.5% of the current urchin index per year.
MAX_REMOVAL_FRACTION = 0.60  # Caps total modeled removal at 60% per year.

def simulate(removal_fraction, scenario_name, debug):  # Runs one version of the model.
    kelp = STARTING_KELP  # Sets kelp to its starting index.
    urchins = STARTING_URCHINS  # Sets urchins to their starting index.
    years = [0]  # Saves the starting year for the graph.
    kelp_history = [kelp]  # Saves the starting kelp value.
    urchin_history = [urchins]  # Saves the starting urchin value.

    if debug:  # Checks whether debug output was requested.
        print(f"\nDebug output: {scenario_name}")  # Labels the debug information.

    for year in range(1, YEARS + 1):  # Repeats the model once for each year.
        kelp_growth = KELP_GROWTH_RATE * kelp * (1 - kelp / KELP_CAPACITY)  # Calculates new kelp growth.
        grazing_loss = GRAZING_RATE * kelp * (urchins / URCHIN_CAPACITY)  # Calculates kelp lost to grazing.
        kelp = kelp + kelp_growth - grazing_loss  # Updates kelp after growth and grazing.
        kelp = max(0.0, min(KELP_CAPACITY, kelp))  # Keeps kelp between zero and its maximum index.

        food_available = kelp / KELP_CAPACITY  # Estimates how much kelp is available as food.
        urchin_births = URCHIN_GROWTH_RATE * urchins * food_available * (1 - urchins / URCHIN_CAPACITY)  # Estimates new urchins.
        natural_deaths = URCHIN_DEATH_RATE * urchins  # Estimates urchins lost to natural mortality.
        urchins = urchins + urchin_births - natural_deaths  # Updates urchins before diver removal.
        urchins = max(0.0, min(URCHIN_CAPACITY, urchins))  # Keeps urchins within the model's limits.
        urchins_removed = urchins * removal_fraction  # Calculates the portion removed by volunteer divers.
        urchins = urchins - urchins_removed  # Subtracts the modeled removal from the urchin index.

        years.append(year)  # Saves the year for the graph.
        kelp_history.append(kelp)  # Saves this year's kelp index.
        urchin_history.append(urchins)  # Saves this year's urchin index.

        if debug:  # Prints yearly values only when debug mode is on.
            print(f"Year {year:2d} | Kelp index: {kelp:6.2f} | Urchin index: {urchins:6.2f} | Removed: {urchins_removed:5.2f}")  # Shows the yearly model results.

    return years, kelp_history, urchin_history  # Returns the results so they can be graphed.

def main():  # Runs the model setup, simulations, and graph.
    parser = argparse.ArgumentParser(description="Model kelp and urchin abundance with volunteer diver removal.")  # Creates command-line options.
    parser.add_argument("--divers", type=int, default=None, help="Number of volunteer divers.")  # Adds an optional diver-number argument.
    parser.add_argument("--debug", action="store_true", help="Print each year's model values.")  # Adds an optional debug argument.
    args, _ = parser.parse_known_args()  # Reads arguments while ignoring Jupyter's extra kernel arguments.

    if args.divers is None:  # Checks whether a diver number was supplied as an argument.
        while True:  # Repeats the prompt until a valid whole number is entered.
            try:  # Tries to read the diver number.
                divers = int(input("How many volunteer divers? Enter a whole number: "))  # Prompts for the number of divers.
                if divers < 0:  # Checks that the number is not negative.
                    print("Please enter zero or a positive number.")  # Explains the valid input.
                    continue  # Shows the prompt again.
                break  # Continues once the number is valid.
            except ValueError:  # Handles text or other invalid entries.
                print("Please enter a whole number, such as 5.")  # Explains the expected input.
    else:  # Uses the diver number supplied as an argument.
        divers = args.divers  # Saves the argument as the number of divers.
        if divers < 0:  # Checks that the argument is not negative.
            parser.error("The number of divers cannot be negative.")  # Stops with a clear error message.

    debug = args.debug  # Turns on debug output if --debug was used.
    if not debug:  # Prompts for debug mode if the argument was not used.
        debug_answer = input("Turn on debug output? (y/n): ").strip().lower()  # Asks whether to print yearly details.
        debug = debug_answer in ("y", "yes")  # Turns debug mode on for a yes answer.

    removal_fraction = min(divers * REMOVAL_PER_DIVER, MAX_REMOVAL_FRACTION)  # Converts divers into a capped annual removal fraction.
    print(f"\nModeling {divers} volunteer divers.")  # Reports the number of divers entered.
    print(f"Assumed annual removal: {removal_fraction * 100:.1f}% of the current urchin index.")  # Reports the model's removal assumption.
    print("Starting values are illustrative abundance indexes, not real population counts.")  # Clarifies what the model values mean.

    years, kelp_baseline, urchin_baseline = simulate(0.0, "No diver removal", debug)  # Runs the comparison with no removal.
    _, kelp_with_divers, urchin_with_divers = simulate(removal_fraction, f"{divers} volunteer divers", debug)  # Runs the scenario with the entered number of divers.

    figure, (urchin_axis, kelp_axis) = plt.subplots(2, 1, figsize=(10, 7), sharex=True)  # Creates two graph panels with a shared year axis.
    urchin_axis.plot(years, urchin_baseline, linestyle="--", color="gray", label="No diver removal")  # Draws urchins in the no-removal scenario.
    urchin_axis.plot(years, urchin_with_divers, color="darkorange", label=f"{divers} volunteer divers")  # Draws urchins in the diver-removal scenario.
    urchin_axis.set_ylabel("Urchin abundance index (0–100)")  # Labels the urchin graph's vertical axis.
    urchin_axis.set_ylim(0, URCHIN_CAPACITY)  # Keeps the urchin graph within its index limits.
    urchin_axis.legend()  # Shows which line represents each scenario.
    urchin_axis.grid(True, alpha=0.3)  # Adds a light grid to help read the graph.

    kelp_axis.plot(years, kelp_baseline, linestyle="--", color="gray", label="No diver removal")  # Draws kelp in the no-removal scenario.
    kelp_axis.plot(years, kelp_with_divers, color="seagreen", label=f"{divers} volunteer divers")  # Draws kelp in the diver-removal scenario.
    kelp_axis.set_xlabel("Years into the simulation")  # Labels the graph's horizontal axis.
    kelp_axis.set_ylabel("Kelp abundance index (0–100)")  # Labels the kelp graph's vertical axis.
    kelp_axis.set_ylim(0, KELP_CAPACITY)  # Keeps the kelp graph within its index limits.
    kelp_axis.legend()  # Shows which line represents each scenario.
    kelp_axis.grid(True, alpha=0.3)  # Adds a light grid to help read the graph.

    figure.suptitle("Illustrative Kelp and Urchin Model")  # Adds a title to the figure.
    figure.tight_layout(rect=[0, 0, 1, 0.95])  # Adjusts spacing so the title and graphs fit.
    figure.savefig("kelp_urchin_volunteer_model.png", dpi=200)  # Saves the graph for your presentation.
    plt.show()  # Displays the graphs.

if __name__ == "__main__":  # Checks whether the file or notebook cell is being run directly.
    main()  # Starts the model.

if __name__ == "__main__":  # Checks whether this file was run directly.
    main()  # Starts the program.
    
