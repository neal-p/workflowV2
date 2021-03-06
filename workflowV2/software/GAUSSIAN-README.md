# GAUSSIAN calculation interface
Submits gaussian calculations on the input molecule

The gaussian calculator object is created with the GAUSSIAN function

```
from workflowV2.software.GAUSSIAN import GAUSSIAN
from workflowV2 import molecule
from workflowV2.calculator import Run

mol = molecule.SmilesToMol('C1=CC=CC=C1')
calc = GAUSSIAN(mol,jobname='benzene_opt',runtype='opt_freq',method='HF/6-31G') 
result = Run(calc)
```

## Gaussian calculation options

### Required arguments
`mol` - the molecule object to compute\
`jobname` - name of the calculation (this names the directory and files for the calculation)\
`runtype` - what type of calculation is to be run:
  - `'sp'`
  - `'opt'`
  - `'freq'`
  - `'opt_freq'`
  - `'irc'`

`method` - the gaussian method to use\
  for example, a dft calculation should give the `functional/basis set`\
  a semi-empirical calculation should give the name of the semi-empirical method like `'PM7'`

### Job related arguments
`nproc` - number of processors\
`mem` - memory in GB\
`time` - time limit for the job\
`partition` - partition for the job

### Calculation specifications
Most gaussian keywords for the route line are availible for use [gaussian manual](http://gaussian.com/man/)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; **General formatting** \
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Route line options can be specified like:\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; `calc = GAUSSIAN(mol,runtype='opt_freq',method='B3LYP/6-31G',opt='calcfc')`\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; where the `keyword = 'some option'` is added to the route line as `keyword=(some option)`
&nbsp; \
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Another example would be to add implicit solvent with gaussian's scrf keyword\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; `calc = GAUSSIAN(mol,runtype='opt_freq',method='HF/6-31G',opt='calcfc',scrf='iefpcm,solvent=toluene')`\
&nbsp; \
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Finally, we can add empirical dispersion correction with
```
calc = GAUSSIAN(mol,runtype='opt_freq',opt='calcfc'
                   method='HF/6-31G',empiricaldispersion='GD3bj',scrf='iefpcm,solvent=toluene'
                   )
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; (it's often useful to split up the function call to multiple lines since complex calcuations can have lengthy route lines)\
&nbsp; \
&nbsp; \
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; **Dealing with Constraints** \
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; If the input Mol object has constraints defined in the `.constraints` attribute, \
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; the GAUSSIAN calculator will automatically add `modredundant` to the `opt` keyword,  \
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; apply the appropriate atom constraints after the input section. \
&nbsp; \
&nbsp; \
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; **Reading from chk** \
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Additionally, keywords for reading from chk files are allowed, but the `oldchk` should be specified \
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; `calc = GAUSSIAN(mol,runtype='opt_freq',method='HF/6-31G',oldchk='/home/benzene.chk',geom='check',guess='read')` \
&nbsp; \
&nbsp; \
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; **Advanced options** \
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Lastly, there are two special keywords for advanced use: \
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; - `afterinput` - This allows the user to specify additional input for after the input coordinates. \
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; This can be used for specifying custom solvents \
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; - `advanced_route_opts` - This is a catch-all keyword that will put anything passed to it directly on the route line, exactly as written: \
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; `advanced_route_opts='banana formcheck'` is added to the route line as `banana formcheck` 

### Misc. Arguments
There are a few arguments that fall outside of any of the above categories the user should still be aware of:\
`delete` - This argument takes a list and should specify globbed matches for files to be deleted. \
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; For example, the default is `delete=['Gau*']` which will delete the Gau-#####.* intermediate files if present. \
`TS` - This argument does not affect the calculation setup whatsoever. It merely suppresses the warning for a single negative frequency if set to `True`.\
`try_count` - This argument should very rarely be set by the user. \
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; It is for internally keeping track of how many times a job has been resubmitted by the failure handler and there are no existing use cases for the user themselves to specify it directly. 


## Gaussian Results
The results of gaussian calculations are stored in the updated Mol object that is returned by the calculator. 


### Direclty updated Mol attributes
`mol.energy` (float) - Updated with the calculated energy with the following priority
  1. Free energy
  2. Excited State Total Energy
  3. Electronic energy

`mol.coords` (np array of x,y,z coordinate floats) - Updated with the coordinates. \
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; This also updates all of the dependencies (`mol.xyz`, `mol.xyzstring`, etc)\
`mol.warnings` - Any calculation warnings are put in a list in the warnings attribute. These warnings are used to resubmit failed calculations\

### Updated in Mol tags attribute
`mol.tags['chk']` (string) - Name of the calculation checkpoint file. This can be used to read the checkpoint into subsequent calculations

### Updated in Mol properties attribute
Most of the additional calculation information is stored here\
There is also some redundancy with the energies parsed above 

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; **Energies** \
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; `mol.properties['electronic_energy']`(float) - Electronic energy\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; `mol.properties['optimization_energies']` (list of floats) - Electronic energy at each optimization step\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; `mol.properties['thermal_correction']` (float) - thermal energy correction\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; `mol.properties['enthalpy_correction']` (float) - enthalpy correction\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; `mol.properties['free_energy_correction']` (float) - free energy correction\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; `mol.properties['zero_point_corrected_energy']` (float) - zero point energy corrected energy\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; `mol.properties['thermal_energy']` (float) - thermal energy\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; `mol.properties['enthalpy']` (float) - enthalpy\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; `mol.properties['free_energy']` (float) - free energy

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; **Orbitals** \
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; `mol.properties['homo']` (float) - HOMO orbital energy\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; `mol.properties['lumo']` (float) - LUMO orbital energy\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; `mol.properties['occupied_orbitals']` (list of floats) - Occupied orbital energies\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; `mol.properties['unoccupied_orbitals']` (list of floats) - Unoccupied orbital energies

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; **Coordinates** \
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; `mol.properties['optimization_xyzs']` (list of strings) - a list of XYZ files, as strings, for each optimization geometry

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; **Vibrations** \
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; `mol.properties['frequencies']` (np array of floats) - vibrational frequencies

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; **Excited state properties** \
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; `mol.properties['excited_state_energy']` (float) - excited state energy (for the state of interest)\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; `mol.properties['excited_states']` (list of dicts) - a list containing a dictionary for each excited state with the following keys:\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;   - `'character'` - i.e. Singlet, triplet\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;   - `'energy'` - energy in eVs\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;   - `'wavelength'` - wavelength in nm\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;   - `'f'` - oscillator strength\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;   - `'transitions'` - a tuple of the (orbital_transitions,contribution)


