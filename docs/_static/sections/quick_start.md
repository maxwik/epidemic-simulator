# Quick start guide



## Getting the files

Go to the private bitbucket repository and ‘clone’ the repository by clicking on ‘clone’ on the top right of the page and follow the instructions. You can do this in a folder of your choice.

```bash
$ ~/tutorial/markov_simuls > python3 -m venv edaDev
$ ~/tutorial/markov_simuls > source edaDev/bin/activate
(edaDev) $ ~/tutorial/markov_simuls >
```

The entire tutorial will be based on a specific state of the repository: ``3bb22c3``

```bash
$ ~/tutorial > cd markov_simuls
$ ~/tutorial/markov_simuls > git checkout 3bb22c3
```

## Repository structure

The repository has three main components that we would be using which are in three different folders:

- ``staticInst``: This folder contains the python code that generates the city once it is provided the demographics/employment etc. information.
- ``cpp_simulator``: This contains the C++ code for the simulator
- ``simulator``: This contains the javascript version of the simulator 
- ``cpp_simulator_calibration``: This contains some python scripts that is used for calibrating some of the values.


We will first do the city building.

## Building the synthetic city

In order to run the city building script, we would need to get a few python packages. We recommend using a virtual environment to install the packages, since it isolates the pacakges to be installed from your system-wide python setup. More infomration is available in [setting up the python environment](/sections/instantiation/setting_up_py_env.md) page.

```bash
(edaDev) $ ~/tutorial/markov_simuls > pip install -r staticInst/requirements.txt
```

With these packages installed, we are now ready to actually build the city.

The demographics, employment etc. data is present in the folder ``staticInst/data/base/[city name]``. We will work with Mumbai and generate the city.

```bash
(edaDev) $ ~/tutorial/markov_simuls > cd staticInst
(edaDev) $ ~/tutorial/markov_simuls/staticInst > python CityGen.py -h
usage: CityGen.py [-h] [-n N] [-i I] [-o O] [--validate] [-s S]

Create mini-city for COVID-19 simulation

optional arguments:
  -h, --help  show this help message and exit
  -n N        target population
  -i I        input folder
  -o O        output folder
  --validate  script for validation plots on
  -s S        [for debug] restore random seed from folder
```

With the syntax in mind, let us go ahead and build the city (of 150k size).

```bash
(edaDev) $ ~/tutorial/markov_simuls/staticInst > python CityGen.py -n 150000 -i data/base/mumbai -o data/mumbai_150k --validate
input_folder: data/base/mumbai
output_folder: data/mumbai_150k

(Distance kernel parameters provided.)
createHouses                  ...	done. (6242 ms)
populateHouses                ...	done. (22332 ms)
assignSchools                 ...	done. (287 ms)
assignWorkplaces              ...	done. (6630 ms)

City: mumbai
Population: 150071
Number of wards: 48
Has slums: True

Number of houses: 33259
Number of schools: 107
Number of workplaces: 2899
Number of workers: 60606

dump_files                    ...	done. (1239 ms)
validate_ages                 ...	done. (452 ms)
validate_householdsizes       ...	done. (158 ms)
validate_schoolsizes          ...	done. (162 ms)
validate_workplacesizes       ...	done. (801 ms)
validate_commutedistances     ...	done. (1819 ms)
```


We now have a 150k version of Mumbai instantiated in ``staticInst/data/mumbai_150k``.


## Simulating a test run

Now that we have generated a city, we can go ahead and execute a test run to see if everything is working fine.

```bash
(edaDev) $ ~/tutorial/markov_simuls/staticInst > cd ../cpp-simulator
(edaDev) $ ~/tutorial/markov_simuls/cpp-simulator > make all
```


**Note for Mac users**: The above make command will complain saying that there is no option fopenmp as Mac does not by default have that. In that case, instead run the following command:

```bash
(edaDev) $ ~/tutorial/markov_simuls/cpp-simulator > make all -f Makefile_np
```

We can now execute a test run. Let us first create the directory where we want to store the outputs of the simulation.

```bash
(edaDev) $ ~/tutorial/markov_simuls/cpp-simulator > mkdir -p outputs/mumbai_test_run
```

… and now the actual simulation

```bash
(edaDev) $ ~/tutorial/markov_simuls/cpp-simulator > ./drive_simulator --NUM_DAYS 10 --SEED_FIXED_NUMBER --INIT_FIXED_NUMBER_INFECTED 100 --INTERVENTION 0 --input_directory ../staticInst/data/mumbai_150k --output_directory outputs/mumbai_test_run
```

You should now have a whole bunch of files in the ``cpp-simulator/outputs/mumbai_test_run`` folder.