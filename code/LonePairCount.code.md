# LonePairCount.groovy
**Source code:**
```groovy
@Grab(group='org.openscience.cdk', module='cdk-bundle', version='2.9')

import org.openscience.cdk.interfaces.*;
import org.openscience.cdk.*;

IAtom atom1 = new Atom("H")
IAtom atom2 = new Atom("H")
IAtom atom3 = new Atom("O")
IBond bond1 = new Bond(atom1, atom2, IBond.Order.SINGLE)
IBond bond2 = new Bond(atom2, atom3, IBond.Order.SINGLE)
IAtomContainer water = new AtomContainer()
water.addAtom(atom1)
water.addAtom(atom2)
water.addAtom(atom3)
water.addBond(bond1)
water.addBond(bond2)
water.addLonePair(new LonePair(atom3))
water.addLonePair(new LonePair(atom3))
for (atom in water.atoms()) {
  println atom.getSymbol() + " has " +
    water.getConnectedLonePairsCount(atom) +
    " lone pairs" 
}
```
**Output:**
```plain
H has 0 lone pairs
H has 0 lone pairs
O has 2 lone pairs
```
