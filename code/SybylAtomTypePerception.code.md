# SybylAtomTypePerception.groovy
**Source code:**
```groovy
@Grab(group='org.openscience.cdk', module='cdk-bundle', version='2.9')

import org.openscience.cdk.interfaces.*;
import org.openscience.cdk.*;
import org.openscience.cdk.atomtype.*;
import org.openscience.cdk.config.*;
import org.openscience.cdk.tools.manipulator.*;
import javax.vecmath.Point3d;

molecule = new AtomContainer();
atom = new Atom(Elements.CARBON);
molecule.addAtom(atom);
matcher = SybylAtomTypeMatcher.getInstance(
  DefaultChemObjectBuilder.getInstance()
);
type = matcher.findMatchingAtomType(molecule, atom);
AtomTypeManipulator.configure(atom, type);
println "Atom type: $type.atomTypeName"
```
**Output:**
```plain
Atom type: C.3
```
