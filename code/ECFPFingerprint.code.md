# ECFPFingerprint.groovy
**Source code:**
```groovy
@Grab(group='org.openscience.cdk', module='cdk-bundle', version='2.7.1')

import org.openscience.cdk.interfaces.*;
import org.openscience.cdk.templates.*;
import org.openscience.cdk.*;
import org.openscience.cdk.fingerprint.*;
import org.openscience.cdk.tools.*;
import org.openscience.cdk.tools.manipulator.*;

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
//adder = CDKHydrogenAdder.getInstance(
//  ethanol.getBuilder()
//)
//AtomContainerManipulator.percieveAtomTypesAndConfigureAtoms(ethanol)
//adder.addImplicitHydrogens(ethanol)
fingerprinter = new CircularFingerprinter(
  CircularFingerprinter.CLASS_ECFP6
);
println "ethanol: " +
  fingerprinter.getBitFingerprint(ethanol).
    asBitSet()
```
**Output:**
```plain
ethanol: {152, 325, 625, 740, 947, 993}
```
