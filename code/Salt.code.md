# Salt.groovy
**Source code:**
```groovy
@Grab(group='org.openscience.cdk', module='cdk-bundle', version='2.9')

import org.openscience.cdk.interfaces.*;
import org.openscience.cdk.*;

salt = new AtomContainer();
sodium = new Atom("Na");
sodium.setFormalCharge(+1);
chloride = new Atom("Cl");
chloride.setFormalCharge(-1);
salt.addAtom(sodium);
salt.addAtom(chloride);
```
**Output:**
```plain
```
