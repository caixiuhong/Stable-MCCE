# MCCE tools reference
<small><i>Page last updated on: {{ git_revision_date }}</i></small>

## Improving and helping runs
---
### step by step mcce run scripts
MCCE involves 4 main steps:

1. Step 1: Convert raw PDB file to MCCE PDB file, reformat terminal residues, and strip off surface water and ions.
2. Step 2: Make side chain conformers, including rotamers and ionization conformers.
3. Step 3: Compute self-energy of conformers and pairwise energy lookup table.
4. Step 4: Monte Carlo sampling to evaluate favorable side chain position and ionization at various conditions.

While the control of these 4 steps is fully configured in run.prm and executed by mcce command, the 4 steps can be carried out by python wrapper scripts without configuring run.prm.

####  Step 1: prepare structure
```
usage: step1.py [-h] [--norun] [--noter] [-e /path/to/mcce] [-u Key=Value]
                [-d epsilon] [--dry]
                pdb

Run mcce step 1, premcce to format PDB file to MCCE PDB format.

positional arguments:
  pdb

optional arguments:
  -h, --help        show this help message and exit
  --norun           Create run.prm but do not run step 1
  --noter           Do not label terminal residues (for making ftpl).
  -e /path/to/mcce  mcce executable location, default to "mcce"
  -u Key=Value      User customized variables
  -d epsilon        protein dielectric constant for delphi, default to 4.0
  --dry             Delete all water molecules.
```

####  Step 2: make conformers
```
usage: step2.py [-h] [--norun] [-d epsilon] [-e /path/to/mcce] [-u Key=Value]
                [-l level]

Run mcce step 2, make side chain conformers from step1_out.pdb.

optional arguments:
  -h, --help        show this help message and exit
  --norun           Create run.prm but do not run step 2
  -d epsilon        dielectric constant for optimizing conformers
  -e /path/to/mcce  mcce executable location, default to "mcce"
  -u Key=Value      User customized variables
  -l level          conformer level 1: quick, 2: medium, 3: comprehensive
```

####  Step 3: energy calculation
```
usage: step3.py [-h] [-c start end] [-d epsilon] [-e /path/to/mcce]
                [-f tmp folder] [-p processes] [-r] [-u Key=Value]
                [-x /path/to/delphi] [--norun]

Run mcce step 3, energy calculations, with multiple threads.

optional arguments:
  -h, --help          show this help message and exit
  -c start end        starting and ending conformer, default to 1 and 9999
  -d epsilon          protein dielectric constant for delphi, default to 4.0
  -e /path/to/mcce    mcce executable location, default to "mcce"
  -f tmp folder       delphi temporary folder, default to /tmp
  -p processes        run mcce with number of processes, default to 1
  -r                  refresh opp files and head3.lst without running delphi
  -u Key=Value        User customized variables
  -x /path/to/delphi  delphi executable location, default to "delphi"
  --norun             Create run.prm but do not run step 3
```

step3.py is able to run in multiple threads.

####  Step 4: Monte Carlo sampling
```
usage: step4.py [-h] [--norun] [-i initial ph/eh] [-d interval] [-n steps]
                [--xts] [--ms] [-e /path/to/mcce] [-t ph or eh] [-u Key=Value]

Run mcce step 4, Monte Carlo sampling to simulate a titration.

optional arguments:
  -h, --help        show this help message and exit
  --norun           Create run.prm but do not run step 4
  -i initial ph/eh  Initial pH/Eh of titration
  -d interval       titration interval in pJ or mV
  -n steps          number of steps of titration
  --xts             Enable entropy correction, default is false
  --ms              Enable microstate output
  -e /path/to/mcce  mcce executable location, default to "mcce"
  -t ph or eh       titration type, pH or Eh
  -u Key=Value      User customized variables
```

#### Step 5: titration fitting
Step 5 is optional. It writes net charge of each residue in sum_crg2.out and fits the titration curve in pK2.out.
```
usage: step5.py [-h]

Run mcce step 5, generate net charge, fit titration curve.

optional arguments:
  -h, --help  show this help message and exit
```

#### Step 6: hydrogen bond
Step 6 is for hydrogen bond network analysis. It requires step 4 to output microstates.

```
usage: step6.py [-h] [--norun] [-e /path/to/mcce] [-u Key=Value]

Run step 6, hydrogen bond analysis, requires microstate output from step 4.

optional arguments:
  -h, --help        show this help message and exit
  --norun           Create run.prm but do not run step 6
  -e /path/to/mcce  mcce executable location, default to "mcce"
  -u Key=Value      User customized variables
```

Example:
```
step4.py --ms
step6.py
```

#### Combine first 4 steps
The first 4 steps are essential steps.

``` bash
#!/bin/bash
step1.py input.pdb
step2.py
step3.py
step4.py
```

---
### energies.py
*Run step3 with multiple threads.*

**Syntax:**
```
energies.py [-h] [-p processes] [-r] [-c start end] [-e /path/to/mcce]
```

**Optional arguments:**

Argument | Description
---|---
-h, --help  |      show this help message and exit
-p processes |     run mcce with number of processes, default to 1
-r          |     refresh opp files and head3.lst without running delphi
-c start end |     starting and ending conformer, default to 1 and 9999
-e /path/to/mcce | mcce executable location, default to "mcce"

**Examples:**

```
energies.py -p 6 
```
This command runs step 3 in 6 threads.

```
energies.py -r 
```
This command runs step 3 just to refresh head3.lst and vdw energy.

```
energies.py -c 1 100 -p 4 
```
This command calculates conformer from 1 to 100 in 5 threads.

**Note**

After this command, the lines:
```
1       delphi start conformer number, 0 based            (PBE_START)
99999   delphi end conformer number, self included        (PBE_END)
```
in run.prm file might be altered.

---
### getpdb
*Download a pdb file by its PDB ID.*

**Syntax:**
```
getpbd pdbid [saved name]
```

**Example:**
```
getpdb 1akk
Inquiring the remote file 1AKK.pdb ...
Saving as 1AKK.pdb ...
Download completed. 
```

Or 
```
getpdb 1akk prot.pdb
Inquiring the remote file 1AKK.pdb ...
Saving as prot.pdb ...
Download completed. 
```

to save as specified name.


---
### translate_step2.py
*Translate old atom names in step2_out to new PDB v3 names.*

**Syntax:**
```
translate_step2.py step2_out.pdb
```

Hydrogen names in PDB v3 are significantly different from PDB v2. Due to the randomness in step 2, 
it is sometimes desired to preserve step2_out.pdb. This tool translates the names so that step2_out.pdb so that it 
can be used by new mcce.

This program prints the translated content on screen. So the user is responsible to make backup of the step2_out.pdb 
file and redirect the outcome to a new step2_out.pdb.  
  
**Example:**
```
cp step2_out.pdb step2_out.pdb.bak
translate_step2.py step2_out.pdb.bak > step2_out.pdb
```


## Result Analysis
---
### state.py
*Analyze how a microstate energy is composed.*

**Syntax:**
```
state.py
```

**Required files:**

 * run.prm
 * head3.lst
 * extra.tpl for scaling factors that are not equal to 1.
 * energies/\*.opp for pairwise interactions 
 * microstate file named "states"
 
The microstate file is in a fixed format. The first line is like:
```
T=298.15, ph=7.0, eh=0.0
```

It specifies the the environment variables which affects the energy calculation. Variables are separated by ",
". The variable names are **not** case sensitive. 

 * T: Temperature. Room temperature is 298.15 K
 * ph: pH value.
 * eh: Eh value, the electric potential for potential titration.

The rest of the lines are microstates. One line per state. Each line consists the index number of conformers (0 
based). Numbers can be separated by either comma or space.
 
**Example:**

Make a states file:

```

```

Run 
```
state.py
```

---
### mfe.py
*Analyze ionization free energy of a residue. It tells you why a residue has that pKa and what factors played a role.*

**Syntax**
```
mfe.py [-h] [-p pH/Eh] [-x TS_correction] [-c cutoff] residue
```

 * residue: residue ID as in pK.out
 * pH/Eh: pH value at which ionization energy is calculated. Default is the pKa (midpoint) where dG = 0.
 * cut_off: Report only pairwise interaction bigger than this value.
 * TS_correction: "t" to include entropy in G, "f" not to include, "r" (default) will look for run.prm. If entropy correction was turned on in step 4, mfe should not include this term as entropy has been "removed".

**Required files:**

 * run.prm
 * head3.lst
 * extra.tpl for scaling factors that are not equal to 1.
 * energies/\*.opp for pairwise interactions 

**Example:**

Check titration result in pKa.out:
```
cat pK.out
... 
ASP-A0044_        3.947
...
```

ASP-A0052 is the residue_ID. Its calculated pKa is 2.275. At this point, the free energy of reaction from ASP neutral to ASP ionized should be close to 0.

Run 
```
$ mfe.py -c 0.01 ASP-A0044_ 
Residue ASP-A0044_ pKa/Em=3.947
=================================
Terms          pH     meV    Kcal
---------------------------------
vdw0        -0.00   -0.27   -0.01
vdw1        -0.01   -0.59   -0.01
tors        -0.15   -8.63   -0.20
ebkb        -0.67  -38.66   -0.91
dsol         0.26   15.35    0.36
offset      -0.62  -36.17   -0.85
pH&pK0       0.80   46.61    1.10
Eh&Em0       0.00    0.00    0.00
-TS          0.58   33.46    0.79
residues    -0.19  -11.14   -0.26
*********************************
TOTAL       -0.00   -0.05   -0.00  sum_crg
*********************************
NTRA0003_   -0.03   -1.66   -0.04    1.00
ASPA0022_    0.02    0.95    0.02   -0.86
ASNA0023_   -0.01   -0.68   -0.02    0.00
LYSA0027_   -0.01   -0.77   -0.02    1.00
LYSA0039_   -0.14   -8.23   -0.19    1.00
ASNA0041_   -0.01   -0.62   -0.01    0.00
LYSA0049_   -0.04   -2.10   -0.05    1.00
ASPA0054_    0.01    0.69    0.02   -0.95
GLUA0073_    0.01    0.79    0.02   -0.99
ASPA0075_    0.05    2.85    0.07   -1.00
TYRA0078_    0.02    1.05    0.02   -0.00
SERA0080_    0.02    1.19    0.03    0.00
ARGA0083_   -0.04   -2.51   -0.06    1.00
ARGA0087_   -0.03   -1.49   -0.04    1.00
HISA0102_   -0.02   -1.02   -0.02    1.00
=================================
```

You can do mfe calculation at pH other than mid-point.

---
### mfe_all.py
Perform mfe analysis on all ionizable residues.

**Syntax**
```
mfe_all.py [-h] [-p pH/Eh] [-x TS_correction] [-f format] [-o filename]
```

 * pH/Eh: pH value at which ionization energy is calculated. Default is the pKa (midpoint) where dG = 0.
 * TS_correction: "t" to include entropy in G, "f" not to include, "r" (default) will look for run.prm. If entropy correction was turned on in step 4, mfe should not include this term as entropy has been "removed".
 * format: output file format, xlsx, csv, or html. The default is excel format xlsx.
 * filename: output file name. The default is mfe_all.\[ext\]

**Required files:**

 * run.prm
 * head3.lst
 * extra.tpl for scaling factors that are not equal to 1.
 * energies/\*.opp for pairwise interactions

**Example:**
```
mfe_all.py -f html
MFE output saved in file mfe_all.html
```

---
### sanity.py
*Quick check on the charge state of ionizable residues abd report unusual charge.*

**Syntax**
```
sanity.py
```

**Required files:**

 * run.prm
 * sum_crg.out

**Example:**
```
$ sanity.py 
TYR-A0013_: charge  0.00 expected near -1.0 at pH=12.0
HIS+A0018_: charge  0.00 expected near  1.0 at pH= 0.0
TYR-A0024_: charge  0.00 expected near -1.0 at pH=12.0
LYS+A0062_: charge  0.55 expected near  0.0 at pH=13.0
GLU-A0073_: charge -0.74 expected near  0.0 at pH= 2.0
ASP-A0075_: charge -1.00 expected near  0.0 at pH= 0.0
TYR-A0078_: charge  0.00 expected near -1.0 at pH=12.0
TYR-A0090_: charge -0.09 expected near -1.0 at pH=12.0
ASP-A0093_: charge -0.83 expected near  0.0 at pH= 1.0
TYR-A0097_: charge -0.17 expected near -1.0 at pH=12.0
TYR-A0103_: charge  0.00 expected near -1.0 at pH=12.0
```
---

## Parameter file preparation

## Connecting to other programs

---
### striph2o.py
*This tool strips away specified surface molecules layer by layer until all left are buried*

**Syntax**
```
usage: striph2o.py [-h] [-c RES [RES ...]] [-s exposure] -f inputfile
                   [-o outputfile]

Strip off exposed cofactors like water and ions based on solvent accessible
surface area.

optional arguments:
  -h, --help        show this help message and exit
  -c RES [RES ...]  Specify cofactor names to strip off, default is HOH.
  -s exposure       Fraction exposure threshold to be cut. Default is 0.05.
  -f inputfile      Input file name.
  -o outputfile     Output file name, default is inputfile name with extension
                    .stripped.
```

This program does not only strip off water, it also strip off any molecule as named in the command line. So one can use it to delete ions and lipid bilayers in the input structure.

**Required files:**

 * input pdb file

**Output files**

* input_file-stripped.pdb for stripped pdb file unless otherwise named
* input_file.acc for residue solvent accessibility


**Example:**
```
striph2o.py -f frame32_50.pdb -c HOH CLA SOD BGA POP DLP
```
---
