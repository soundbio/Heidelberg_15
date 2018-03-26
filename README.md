## Random notes

### Problems encountered

####  `tleap` tool force fields input file

The original file intended for use by the `MAWS.py` code (`leaprc.ff12SB`) does not show up within the Amber data files (on the server located at `/home/jpuser/anaconda3/pkgs/ambermini-16.16.0-7/dat/leap/`) but we found one that does exist (`leaprc.constph`) and was able to point the code to that as an alternative, which appears to have enabled it to get to the next stage in the code. We will likely need to identify the proper force fields file and install that to the approriate directory (`/home/jpuser/anaconda3/pkgs/ambermini-16.16.0-7/dat/leap/cmd/`) in case the differences in what assumptions are made by that file could drive miscalculations in the MAWS code.

#### Issues with the input protein file

The particular protein file we were attempting to work with at the beginning (`data/1aoc.pdb`) still gave us errors after we fixed the above issue. The line Python wasn't performing (based on seeing `FileNotFound` errors for `DGN.prmtop`, etc.) was `saveamberparm union DGN.prmtop DGN.inpcrd` within the `tleap` tool (also applies to any of the four nucleotides: `DGN`, `DAN`, `DTN`, `DCN`). We tried running the steps the MAWS code was running directly within the `tleap` command line tool to see what was going on. Here's what we ran:

```bash
cd $HOME/jupyter/apdesign/apdesign
tleap -f leaprc.constph

# within the tleap tool:
ligand = loadpdb data/1aoc.pdb
```

Here is the brunt of the errors we got at first:

```
. . .
Creating new UNIT for residue: SO4 sequence: 351
One sided connection. Residue:  missing connect0 atom.
Created a new atom named: O1 within residue: .R<SO4 351>
Created a new atom named: O2 within residue: .R<SO4 351>
Created a new atom named: O3 within residue: .R<SO4 351>
Created a new atom named: O4 within residue: .R<SO4 351>
Created a new atom named: S within residue: .R<SO4 351>
  total atoms in file: 2943
  Leap added 3054 missing atoms according to residue templates:
       12 Heavy
       3042 H / lone pairs
  The file contained 5 atoms not in residue templates
```

After various bits of tinkering, we removed the lines that looked most related (based on the `S` and `O1`/`O2`/etc, all appearing to be part of a SO4 molecule attached to the protein):

```
. . . 
HETATM 2744  O1  SO4 B 600     -21.502   5.846  59.822  1.00 51.93      C    O  
HETATM 2745  O2  SO4 B 600     -19.868   4.612  61.066  1.00 57.56      C    O  
HETATM 2746  O3  SO4 B 600     -20.606   3.861  59.003  1.00 58.13      C    O1-
HETATM 2747  O4  SO4 B 600     -19.262   5.634  59.278  1.00 60.11      C    O  
HETATM 2748  S   SO4 B 600     -20.365   5.000  59.814  1.00 54.95      C    S  
. . . 
```


When re-running after these were removed, we got different errors:

```
. . .
  Added missing heavy atom: .R<THR 194>.A<CG2 7>
  Added missing heavy atom: .R<THR 194>.A<OG1 11>
  Added missing heavy atom: .R<CPHE 350>.A<OXT 21>
Illegal CONECT record in pdb file
Illegal CONECT record in pdb file
Illegal CONECT record in pdb file
Illegal CONECT record in pdb file
Illegal CONECT record in pdb file
  total atoms in file: 2938
  Leap added 3055 missing atoms according to residue templates:
       13 Heavy
       3042 H / lone pairs
```

Scrolling further through the `1aoc.pdb` file, we found and removed these lines:

```
CONECT 2744 2748
CONECT 2745 2748
CONECT 2746 2748
CONECT 2747 2748
CONECT 2748 2744 2745 2746 2747
```

After all of that, we got the `tleap` part "working" (whether it is appropriately modeling the target protein is a question to investigate later, we mainly want to debug the later steps of the `MAWS.py` code). Here's the full set of commands run within the `tleap` tool (lines starting with `> ` are commands fed to the tool; somebody knowledgeable on the protein side of things might be able to help us understand whether this is doing things appropriately with respect to the force field file change and the modifications of the .pdb file itself):

```
-I: Adding /home/jpuser/anaconda3/bin/../dat/leap/prep to search path.
-I: Adding /home/jpuser/anaconda3/bin/../dat/leap/lib to search path.
-I: Adding /home/jpuser/anaconda3/bin/../dat/leap/parm to search path.
-I: Adding /home/jpuser/anaconda3/bin/../dat/leap/cmd to search path.
-f: Source leaprc.constph.

Welcome to LEaP!
(no leaprc in search path)
Sourcing: /home/jpuser/anaconda3/bin/../dat/leap/cmd/leaprc.constph
----- Source: /home/jpuser/anaconda3/bin/../dat/leap/cmd/oldff/leaprc.ff10
----- Source of /home/jpuser/anaconda3/bin/../dat/leap/cmd/oldff/leaprc.ff10 done
Log file: ./leap.log
Loading parameters: /home/jpuser/anaconda3/bin/../dat/leap/parm/parm10.dat
Reading title:
PARM99 + frcmod.ff99SB + frcmod.parmbsc0 + OL3 for RNA
Loading library: /home/jpuser/anaconda3/bin/../dat/leap/lib/amino10.lib
Loading library: /home/jpuser/anaconda3/bin/../dat/leap/lib/aminoct10.lib
Loading library: /home/jpuser/anaconda3/bin/../dat/leap/lib/aminont10.lib
Could not open file phosphoaa10.lib: not found
Could not open database: phosphoaa10.lib
Loading library: /home/jpuser/anaconda3/bin/../dat/leap/lib/nucleic10.lib
Loading library: /home/jpuser/anaconda3/bin/../dat/leap/lib/atomic_ions.lib
Loading library: /home/jpuser/anaconda3/bin/../dat/leap/lib/solvents.lib
Loading library: /home/jpuser/anaconda3/bin/../dat/leap/lib/constph.lib
Loading library: /home/jpuser/anaconda3/bin/../dat/leap/lib/all_prot_nucleic10.lib
Loading library: /home/jpuser/anaconda3/bin/../dat/leap/lib/cph_nucleic_caps.lib
Loading parameters: /home/jpuser/anaconda3/bin/../dat/leap/parm/frcmod.constph
Reading force field modification type file (frcmod)
Reading title:
Force field modifcations for titrations
Loading parameters: /home/jpuser/anaconda3/bin/../dat/leap/parm/frcmod.protonated_nucleic
Reading force field modification type file (frcmod)
Reading title:
Force field modifications for protonated nucleic acids
Using H(N)-modified Bondi radii
> ligand = loadpdb data/1aoc.pdb
Loading PDB file: ./data/1aoc.pdb
 (starting new molecule for chain B)
 (starting new molecule for chain A)
 (starting new molecule for chain B)
  Added missing heavy atom: .R<CPHE 175>.A<OXT 21>
  Added missing heavy atom: .R<LEU 191>.A<CG 8>
  Added missing heavy atom: .R<LEU 191>.A<CD1 10>
  Added missing heavy atom: .R<LEU 191>.A<CD2 14>
  Added missing heavy atom: .R<ARG 193>.A<CG 8>
  Added missing heavy atom: .R<ARG 193>.A<CD 11>
  Added missing heavy atom: .R<ARG 193>.A<NE 14>
  Added missing heavy atom: .R<ARG 193>.A<CZ 16>
  Added missing heavy atom: .R<ARG 193>.A<NH1 17>
  Added missing heavy atom: .R<ARG 193>.A<NH2 20>
  Added missing heavy atom: .R<THR 194>.A<CG2 7>
  Added missing heavy atom: .R<THR 194>.A<OG1 11>
  Added missing heavy atom: .R<CPHE 350>.A<OXT 21>
  total atoms in file: 2938
  Leap added 3055 missing atoms according to residue templates:
       13 Heavy
       3042 H / lone pairs
> DGN = sequence {DGN}
> union = combine { ligand DGN }
> saveamberparm union DGN.prmtop DGN.inpcrd
Checking Unit.
WARNING: The unperturbed charge of the unit: 10.000000 is not zero.

 -- ignoring the warning.

Building topology.
Building atom parameters.
Building bond parameters.
Building angle parameters.
Building proper torsion parameters.
Building improper torsion parameters.
 total 1186 improper torsions applied
Building H-Bond parameters.
Incorporating Non-Bonded adjustments.
Not Marking per-residue atom chain types.
Marking per-residue atom chain types.
  (Residues lacking connect0/connect1 -
   these don't have chain types marked:

        res     total affected

        CPHE    2
        NALA    2
        WAT     195
  )
 (no restraints)
```

#### Issues with the multiprocessing code

Once we got things running smoothly within the `tleap` tool directly, we modified the code to point to the new force fields file (added `forcefield_name = 'leaprc.constph'` after the original to overwrite that variable) and ran the code on the the modified protein file:

```
source activate iGEM
cd $HOME/jupyter/apdesign/apdesign
export AMBERHOME=/home/jpuser/anaconda3/pkgs/ambermini-16.16.0-7/
python src/MAWS.py ./cache/ data/1aoc.pdb -f pdb
```

This got further into running the code, but seemed to randomly be crashing somewhere within the multiprocessing piece. After much debugging (adding in various print lines within the Amber code here: `/home/jpuser/anaconda3/envs/iGEM/lib/python3.6/site-packages/simtk/openmm/app/internal/amber_file_parser.py`), we noticed that the multiprocessing threads were arbitrarily stopping their parsing of the .prmtop files before getting to the end. We broke the multiprocessing logic and instead setup the code to run through the four nucleotides sequentially (edits mainly made to the `loop()` function in `MAWS.py`, adding a for loop). We will reimplement parallelization in the code after we fully debug the rest of it (and likely refactor most of it in general).

### "Working" example

After all the debugging above, we got the code to a state where it appears to be churning through the Monte Carlo simluations (we see `montetest1.pdb` files popping up), which is the next level of "working" for the time we were able to put into this today. Since it's running on a small EC2 instance, it's going to take a while to get through all the calcs, so we're letting it run in the background and will revisit it when we next have time to dive into this.
