# ConnectedAtoms.groovy
**Source code:**
```groovy
@Grab(group='org.openscience.cdk', module='cdk-bundle', version='2.11')

import org.openscience.cdk.interfaces.*;
import org.openscience.cdk.*;

IAtom atom1 = new Atom("C")
IAtom atom2 = new Atom("C")
IAtom atom3 = new Atom("O")
IBond bond1 = new Bond(atom1, atom2, IBond.Order.SINGLE);
IBond bond2 = new Bond(atom2, atom3, IBond.Order.SINGLE);

IAtomContainer ethanol = new AtomContainer();
ethanol.addAtom(atom1);
ethanol.addAtom(atom2);
ethanol.addAtom(atom3);
ethanol.addBond(bond1);
ethanol.addBond(bond2);
for (atom in ethanol.atoms()) {
  print atom.getSymbol() +
    " is connected to "
  for (neighbor in ethanol.getConnectedAtomsList(atom)) {
    print neighbor.getSymbol() + " "
  }
  println ""
}
```
**Output:**
```plain
C is connected to C 
C is connected to C O 
O is connected to C 
```
