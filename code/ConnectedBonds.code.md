# ConnectedBonds.groovy
**Source code:**
```groovy
@Grab(group='org.openscience.cdk', module='cdk-bundle', version='2.9')

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
    " has bond(s)"
  for (bond in ethanol.getConnectedBondsList(atom)) {
    print " " + bond.getOrder()
  }
  println ""
}
```
**Output:**
```plain
C has bond(s) SINGLE
C has bond(s) SINGLE SINGLE
O has bond(s) SINGLE
```
