# Hollow 1.1 (c) 2009.  Bosco Ho and Franz Gruswitz 
Following installation and usage partly written, partly adapted by jnoh2
### Installation
```
conda install -n hollow python=2.7 pip numpy -c condaforge
python setup.py install --user --install-scripts [path to installation]
```

### Use (Automated)
```
hollow -g 0.2 -o [outpath] [inpath]
```

### Use (Constrained)
```
hollow -g 0.2 -o [outpath] -c [constraintpath] [inpath]
```
Constraint File (spherical)
```
{
    'type': 'sphere',          # 'cylinder' or 'sphere'
    'remove_asa_shell': True,  # True or False
    'radius': 13.0,      

    'chain1': 'A',             
    'res_num1': 107,     
    'atom1': 'CD1',      
}
```
Constraint File (cylindrical)
```
{
    'type': 'cylinder',          # 'cylinder' or 'sphere'
    'remove_asa_shell': False,   # True or False

    'radius': 10.0,      

    'chain1': 'F',              
    'res_num1': 57,             
    'atom1': 'CE1',             
    'axis_offset1': 0,          
                                
    'chain2': 'G',              
    'res_num2': 185,            
    'atom2': 'CA',              
    'axis_offset2': -3,
  }
```

## Content prior to fork
Hollow generates fake atoms that identifies voids, pockets, channels and depressions in a protein structure specified in the PDB format. 

There are two effective modes to run this program: an automated mode that explicitly deduces the molecular surface, and a constrained mode that works in a pre-specified volume.

To see the options, type in the command line: 
  
  python hollow.py
  
The program requires Python 2.4 or higher. 

Please refer to the website for illustrated tutorials:

  1) The interior pathways of myoglobin:
       http://hollow.sourceforge.net/myoglobin.html.

  2) The channel surface of a phosphate proin:
       http://hollow.sourceforge.net/channel.html.


# Automated Surface Detection Mode

In the automated (also the default) mode, a grid is constructed over the entire protein. To analyze large proteins, due to the large size of the resultant grid, use only coarse grid spacings of 1.0 angstroms or 0.5 angstroms. 

The automated mode requires several intermediate calculations. The Accessible Surface Area is first calculated to define surface atoms. Then we sweep a large ball (of say 8.0 angstroms) over all the surface atoms to remove grid points near the surface of the protein. The remaining grid points found under the ball define depressions. This is the costliest part of the calculation.




# Constrained Mode

In the constrained mode, either a sphere or a cylinder is specified in a separate file. Grid points are constructed within these constraints, and thus, there is no need for the costly calculation that sweeps over the entire surface. 

The point of having a constrained mode is that once you identify an interior region of interest from a previous calculation in the Default Mode using a coarse grid, you can identify a smaller region of interest and then set up a very fine grid (eg 0.25 angstroms) around this region.

We also label the occupancy with 1.0 if the sphere is within the accessible surface of the protein, otherwise we label the occupancy with 0.0 if the sphere is outside the accessible surface of the protein. This allows the visualization of funnel-like surfaces for channel proteins.

Here's a sample spherical constraint file. The format is a Python dictionary:

  {
    'type': 'sphere',          # 'cylinder' or 'sphere'
    'remove_asa_shell': True,  # True or False
    'radius': 13.0,      

    'chain1': 'A',             
    'res_num1': 107,     
    'atom1': 'CD1',      
  }

The 'type' refers to whether the constraint is in the shape of a sphere or a cylinder. 

The boolean value 'remove_asa_shell' tells the program whether we want to do the outer shell calculation. If this is True then the fake atoms outside the Accessible Surface will be removed. If False then the fake atoms will follow the shape of the sphere (or cylinder) as it projects out of the exterior of the protein.

The atom around which the spherical constraint is centered is denoted by 'chain1', 'res_num1' and 'atom1'. 

Here's a sample cylinder constraint file:

  {
    'type': 'cylinder',          # 'cylinder' or 'sphere'
    'remove_asa_shell': False,   # True or False

    'radius': 10.0,      

    'chain1': 'F',              
    'res_num1': 57,             
    'atom1': 'CE1',             
    'axis_offset1': 0,          
                                
    'chain2': 'G',              
    'res_num2': 185,            
    'atom2': 'CA',              
    'axis_offset2': -3,
  }

For the cylinder constraint, 'chain1' 'res1' and 'atom1' refers to the center of the one end of the cylinder, whilst 'chain2' 'res2' 'atom2' refers to the center at the other end of the cylinder. The length of the cylinder is inferred from the inter-atomic distance between these two atoms. The 'axis_offset1' and 'axis_offset2' allows the length of the cylinder to be adjusted along the cylinder aixs.

It is also possible to add a 'brick' (or 'box') constraint [added by OB in 1.1-ob1]:

 {
    'type': 'brick',	
    'remove_asa_shell': False,   # True or False

    # lower left front corner of the orthorhombic box
    'chain1': 'F',              
    'res_num1': 57,             
    'atom1': 'CE1',             
    'offset1': (0,0,-5.0),     # extend in z direction
                                
    # upper right back corner
    'chain2': 'G',              
    'res_num2': 185,            
    'atom2': 'CA',              
    'offset2': (-5,0,10),      # shrink in x and extend in z
 }   


# Default options
  
Default values for various parameters are stored in hollow.txt. If you use the program a lot, you might want to fine tune these options.
                        
  {
    'grid_spacing': 1.0,
    'interior_probe': 1.444,
    'surface_probe': 8.0,
    'bfactor_probe': 0.0,
    'res_type': 'HOH',
    'atom_type': 'O',
    'atom_field': 'ATOM'
  }

The 'grid_spacing' determines how detailed the resultant fake atoms will be in the output PDB file. Since the program is written in standard Python, the object that holds the information about the grid runs across memory constraints. That is why it is suggested that a medium resolution of 1.0 angstrom is used in a first spacing. Another problem is that a finer grid will generate a lot of fake atoms. This runs into the problem of displaying the atoms in the protein viewer. 

Fortunately, we have found that the structures at 1.0 and 0.5 angstroms are adequate for identifying most of the regions of interest. And a fine grid of 0.2 angstrom has more than enough detail, when used with constraints.

The definition of the accessible surface involves rolling a surface probe over the atomic surface defined by the van der Waals radius of the atoms. The 'interior probe' defines the size of the surface probe.

In the automatic analysis mode, we want to generate fake atoms in clefts. We start the calculation with a cubic grid of fake atoms and systematically eliminate fake atoms that like within the protein structure. However, we need to also eliminate fake atoms outside the accessible surface of the protein as well, whilst allowing fake atoms to fill in surface clefts. To do that, we need to define a wrapping surface around the protein structure, that wraps around the accessible surface but contains spaces corresponding to surface clefts. We do this by defining a large exterior surface probe through 'surface_probe' (default is 8.0 angstroms).

Three other options are also given, and these relate to the chemistry of the fake atoms in the resultant PDB file. There is also the choice of whether the atoms are written as 'ATOM' or 'HETATM' in the PDB file. The default is 'ATOM' even though water is typically written out as 'HETATM' in order to exploit PyMol. In PyMol, molecular surfaces are only generated for polymers, labeled with 'ATOM'. You can't generate molecular surfaces for 'HETATM' entries.



# Atomic radii
  
In order to calculate the accessible surface area, we need the atomic radii. In the program, a set of standard atomic radii are read from the radii.txt. Edit this file to add or change radii for different elements. If the element is not defined, we give it a default of 1.8 angstroms (identified as element '.' in the radii.txt).



# B-factors

We also calculate appropriate B-factors of every fake atom, by averaging over the heavy protein atoms around each fake atom. This is controlled by the command-line option 'BFACTOR_PROBE'.



# Works with PyMol

We developed this program with output designed to be easily viewed and manipulated with PyMol, an open-source protein viewer. By default, the hollow spheres are stored with the "ATOM" field as water oxygen molecules. Pymol can draw the molecular surface of overlapping fake water molecules as it interprets "ATOM" as if the atoms belong to a pseudo polymer.



# Use in IDLE

Hollow can also be imported as a PYTHON module. This allows hollow to be used in the PYTHON command-line, for example:

   import hollow
   hollow.make_hollow_spheres(
      '1abc.pdb', 
      output_pdb='hollow.pdb', 
      grid_spacing=0.1, 
      constraint_file="my_constraint")
  
the parameters to the make_hollow_spheres function are:

  def make_hollow_spheres(
      pdb, 
      out_pdb="",
      grid_spacing=defaults.grid_spacing,
      size_interior_probe=defaults.interior_probe,
      is_skip_waters=defaults.is_skip_waters,
      size_surface_probe=defaults.surface_probe,
      constraint_file="", 
      size_bfactor_probe=defaults.bfactor_probe):

