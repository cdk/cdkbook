# IterateAtomsGroovy.groovy
**Source code:**
```groovy
@Grab(group='org.openscience.cdk', module='cdk-bundle', version='2.9')

import org.openscience.cdk.interfaces.*;
import org.openscience.cdk.silent.SilentChemObjectBuilder;

IChemObjectBuilder builder = SilentChemObjectBuilder.getInstance();
IAtomContainer molecule = builder.newInstance(IAtomContainer.class);
molecule.addAtom(builder.newInstance(IAtom.class, "C"));
molecule.addAtom(builder.newInstance(IAtom.class, "O"));
molecule.addAtom(builder.newInstance(IAtom.class, "C"));

for (atom in molecule.atoms()) {
   println atom.getSymbol()
}
```
**Output:**
```plain
C
O
C
```
