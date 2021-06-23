## cadCAD will be releasing Production and Beta builds
cadCAD is moving towards a release model of Production and Beta builds. Production represents the latest stable build and Beta is the release candidate undergoing testing. Our aim is to introduce more stability for models already built in the cadCAD ecosystem as we introduce fixes and updates. We also aim to increase the quality of each release with a wider scope of testing with broad participation. As we mature towards this release process, we are first inviting users of the Labs Platform Private Alpha to participate on this first iteration.

User Acceptance Testing (UAT) of the Labs Platform Private Alpha will use the version of cadCAD in the `staging` branch, which represents a provisional cadCAD Beta. Please contact Chris Frazier or Chris Catoya(@Drcatman) to participate.

## cadCAD Beta
This version addresses of some of the open cadCAD GitHub issues that will be resolved in the next release. 

### GitHub Issues Addressed by [Merged Pull Request](https://github.com/cadCAD-org/cadCAD/pull/274): 
1. cadCAD parallelized simulations run in series
	* https://github.com/cadCAD-org/cadCAD/issues/242
2. test the validity of result counts between what is specified by the API and the the expected results 
* GitHub Issue: #248
3. A/B test of result count between releases
* GitHub Issue: #248 
4. we need Jupyter Server to recognizing cadCAD module
* Github Issue: #252 
5. ValueError gives mis-leading error message if catching a non-related ValueError
* GitHub Issue: #257

### ToDo:
* Update https://github.com/cadCAD-org/demos with changes to system models

## Labs Platform UAT
The release of this cadCAD Beta coincides with extended cadCAD functionality specific to and bounded to the Labs Platform. It will be necessary to install the cadCAD Beta release and update Labs models for the model simulations to run on the platform 

### Steps required for a Labs conversion:
- [ ] Install `cadCAD` pre-release from `staging` branch to run a Labs-ready project locally
* **Option 1:** Build From Source
* [git clone](https://git-scm.com/docs/git-clone) repo from [`staging`](https://github.com/cadCAD-org/cadCAD/tree/staging)
* `cd` into project's root directory and execute the code block below:
	```
	pip install -r requirements.txt
	python setup.py bdist_wheel
	pip install dist/*.whl
	```
* **Option 2:** Download Package
** `cd` into your exsisting cadCAD project's root directory
** Download and place [`cadCAD-0.4.25-py3-none-any.whl`](https://github.com/cadCAD-org/cadCAD/blob/staging/dist/cadCAD-0.4.25-py3-none-any.whl) in your project's root directory OR `wget https://github.com/cadCAD-org/cadCAD/blob/staging/dist/cadCAD-0.4.25-py3-none-any.whl`
** `pip install cadCAD-0.4.25-py3-none-any.whl`
- [ ] Make neccessary cadCAD Changes: see [CHANGELOG](https://github.com/cadCAD-org/cadCAD/blob/staging/CHANGELOG.md)
- [ ] Create `labs.py` in root directory: [Example](https://github.com/JEJodesty/demos/blob/main/labs.py) 
  - [ ] import `cadCAD.configuration.Experiment()` object the you simulation you want to run on Labs
        Example: `from demos.Agent_Based_Modeling.prey_predator_abm.model.config import exp`
  - [ ] Add `model_dir` variable to specify the directory your model(s) are written within (Example: `model_dir = 'demos'`)
- [ ] Add `__init__.py` to all directories under the one specified with `model_dir`
- [ ] Update all relative imports (relative paths) to absolute imports (full paths): https://realpython.com/absolute-vs-relative-python-imports/
- [ ] add `model_id` to your `append_model()` for every model configuration that is appended
- [ ] Add all imported dependencies under the directory your model(s) are written to `requirements.txt` (this is the same directory you assigned to `model_dir`)
