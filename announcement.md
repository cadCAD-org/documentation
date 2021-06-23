## cadCAD will be releasing Production and Beta builds
cadCAD is moving towards a release model of Production and Beta builds. Production represents the latest stable build and Beta is the release candidate undergoing testing. Our aim is to introduce more stability for models already built in the cadCAD ecosystem as we introduce fixes and updates. We also aim to increase the quality of each release with a wider scope of testing with broad participation. As we mature towards this release process, we are first inviting users of the Labs Platform Private Alpha to participate on this first iteration.

User Acceptance Testing (UAT) of the Labs Platform Private Alpha will use the version of cadCAD in the staging branch, which represents a provisional cadCAD Beta. Please contact Chris Frazier or Chris Catoya(@Drcatman) to participate.

## cadCAD Beta
This version addresses of some of the open cadCAD GitHub issues that will be resolved in the next release. 

### Addressed GitHub Issues
	•	cadCAD parallelized simulations run in series #242 (Implemented / Benchmark Pending)
	•	First PSUB at first timestep has a timestep equals to 0 instead of 1 #250 (Test Passing)
	•	Unexpected behaviour when simulating multiple configurations #248 (Test Passing)
	•	ValueError gives mis-leading error message if catching a non-related ValueError #257 (Test Passing)

### In Progress GitHub Issues 
	•	cadCAD parallelized simulations run in series #242 (Benchmark)
	•	Jupyter lab and Jupyter notebook not recognizing cadCAD module #252

### ToDo:
	•	Update https://github.com/cadCAD-org/demos with changes in forked https://github.com/JEJodesty/demos

## Labs Platform UAT
The release of this cadCAD Beta coincides with extended cadCAD functionality specific to and bounded to the Labs Platform. It will be necessary to install the cadCAD Beta release and update Labs models for the model simulations to run on the platform 

### Steps required for a Labs conversion:
	•	Install the cadCAD Beta release from dev branch to run a Labs-ready project locally
	•	cd into your exsisting cadCAD project's root directory 
	•	Download and place cadCAD-0.4.24-py3-none-any.whl in your project's root directory OR wget https://github.com/cadCAD-org/cadCAD/blob/dev/dist/cadCAD-0.4.24-py3-none-any.whl 
	•	pip install cadCAD-0.4.24-py3-none-any.whl 
	•	Make necessary cadCAD Changes: see CHANGELOG
	•	pip freeze > requirements.txt and delete dependencies not needed for running simulations in your project
	•	Create labs.py in root directory: Example
	◦	import cadCAD.configuration.Experiment() object the you simulation you want to run on Labs Example: `from demos.Agent_Based_Modeling.prey_predator_abm.model.config import exp` 
	◦	Add model_dir variable to specify the directory you model(s) are written within (Example: model_dir = 'demos') 
	•	Add __init__.py to all directories under the one specified with model_dir
	•	Update all relative imports (relative paths) to absolute imports (full paths): https://realpython.com/absolute-vs-relative-python-imports/
	•	add model_id to your append_model() for every model configuration that is appended

