# CREST calculation interface
Submits CREST calculations on the input molecule and returns conformers to the .conformer attribute

The crest calculator object is created with the CREST function

```
from workflowV2.software.CREST import CREST
from workflowV2 import molecule
from workflowV2.calculator import Run

mol = molecule.SmilesToMol('CCCCCCCCCC')
calc = CREST(mol,jobname='confsearch',runtype='confsearch') 
result = Run(calc)
```

## Crest calculation options

### Required arguments
`mol` - the molecule object to compute\
`jobname` - name of the calculation (this names the directory and files for the calculation)\
`runtype` - what type of calculation is to be run:
  - `'confsearch'`

### Job related arguments
`nproc` - number of processors\
`mem` - memory in GB\
`time` - time limit for the job\
`partition` - partition for the job

### Calculation specifications
Most Crest keywords for the command line are availible for use [Crest manual](https://xtb-docs.readthedocs.io/en/latest/crest.html)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; **General formatting** \
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Command line options can be specified like:\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; `calc = CREST(mol,runtype='confsearch',gbsa='water')`\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; where the `keyword = 'some option'` is added to the command line as `keyword=some option`

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Similarly, we could modify the `ewin` energy window and run in Crest's `nci` mode with:\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; `calc = CREST(mol,runtype='confsearch',gbsa='water',ewin=500,nci=True)`\


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; **Dealing with Constraints** \
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; If the input Mol object has constraints defined in the `.constraints` attribute, \
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; the CREST calculator will automatically write a .c constraint file and use `-cinp` on the command line to  \
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; apply the appropriate atom constraints. 

### Misc. Arguments
There are a few arguments that fall outside of any of the above categories the user should still be aware of:\
`delete` - This argument takes a list and should specify globbed matches for files to be deleted. \
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; For example, the default is `delete=['METADYN*','MRMSD','NORMMD*','*.tmp','wbo']` which will delete the METADYN, MRMSD, NORMMD, .tmp and wbo files/directories if present. \
`max_confs` - This argument requests that the workflow code only parses the first `max_confs` conformers, whether or not CREST has generated more than what is specified.\
`try_count` - This argument should very rarely be set by the user. \
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; It is for internally keeping track of how many times a job has been resubmitted by the failure handler and there are no existing use cases for the user themselves to specify it directly. 


## Crest Results
The results of Crest calculations are stored in the updated Mol object that is returned by the calculator. 


### Direclty updated Mol attributes
`mol.conformers` (list) - Updated with the conformers generated by CREST, ranked by their energy
`mol.warnings` (list) - Any calculation warnings are put in a list in the warnings attribute. These warnings are used to resubmit failed calculations

### Updated in Mol tags attribute
`mol.conformers[0]['origin'] = 'CREST'` (string) - Each of the generated conformers is tagged with the origin `'CREST'`

