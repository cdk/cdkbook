# InputFromStringReader.groovy
**Source code:**
```groovy
@Grab(group='org.openscience.cdk', module='cdk-bundle', version='2.7.1')

import java.net.UnknownHostException;
import org.openscience.cdk.*
import org.openscience.cdk.io.*
import org.openscience.cdk.silent.*
import org.openscience.cdk.tools.manipulator.*


String bf3 = "4\n" +
"Bortrifluorid\n" +
"B    0.0000    0.0000    0.0000\n" +
"F    1.0000    0.0000    0.0000\n" +
"F   -0.5000   -0.8660    0.0000\n" +
"F   -0.5000    0.8660    0.0000\n";
reader = new XYZReader(
  new StringReader(bf3)
)
chemfile = reader.read(new ChemFile())
mol = ChemFileManipulator.getAllAtomContainers(chemfile)
  .get(0)
println "Atom count: $mol.atomCount"
assert mol.atomCount == 4
```
**Output:**
```plain
Atom count: 4
```
