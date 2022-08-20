# HydrogenDepletedGraph.groovy
**Source code:**
```groovy
@Grab(group='org.openscience.cdk', module='cdk-bundle', version='2.7.1')

import org.openscience.cdk.interfaces.*;
import org.openscience.cdk.*;
import org.openscience.cdk.atomtype.*;
import org.openscience.cdk.config.*;
import org.openscience.cdk.tools.*;
import org.openscience.cdk.tools.manipulator.*;
import javax.vecmath.Point3d;

molecule = new AtomContainer();
carbon = new Atom(Elements.CARBON);
carbon.setImplicitHydrogenCount(4);
molecule.addAtom(carbon);
```
**Output:**
```plain
```
