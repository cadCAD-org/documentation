# cadCAD.jl Updates

## June 25, 2021

cadCAD.jl, as the Julia implementation is called, was written based on the micro-spec and with performance in mind. 
The goal of this version is to decrease the simulation time of computationally expensive system models, and as a consequence, 
to lower the time to discovery of complex models. It is important to note that although the software is 100% compliant to the micro-spec, 
the latter is general enough to not restrict an engine developer to use the full extent of the syntax of his/hers language of choice. 
In the case of Julia, for example, we used its powerful metaprogramming capabilities, expressive type system and straightforward parallelization 
macros to build a high performance cadCAD engine with fewer lines of code than it would take on other compiled languages. This, in turn, leads to 
higher maintainability and easier onboarding of new contributors. As for future directions, our goal is to make the Julia engine execute Python 
system models, in order to provide acceleration to current cadCAD simulations.

Areas of focus:
- Validation of the engine through Julia system models
- Use PyCall.jl to execute Python system models
