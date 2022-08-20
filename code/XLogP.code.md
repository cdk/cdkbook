# XLogP.groovy
**Source code:**
```groovy
@Grab(group='org.openscience.cdk', module='cdk-bundle', version='2.7.1')

import org.openscience.cdk.*;
import org.openscience.cdk.templates.*;
import org.openscience.cdk.tools.*;
import org.openscience.cdk.tools.manipulator.*;
import org.openscience.cdk.qsar.descriptors.molecular.*;
import org.openscience.cdk.qsar.result.*;

adder = CDKHydrogenAdder.getInstance(
  DefaultChemObjectBuilder.getInstance()
);
oxazone = MoleculeFactory.makeOxazole();
benzene = MoleculeFactory.makeBenzene();
// add explicit hydrogens ...
AtomContainerManipulator.percieveAtomTypesAndConfigureAtoms(oxazone)
adder.addImplicitHydrogens(oxazone)
AtomContainerManipulator.convertImplicitToExplicitHydrogens(oxazone)
AtomContainerManipulator.percieveAtomTypesAndConfigureAtoms(benzene)
adder.addImplicitHydrogens(benzene)
AtomContainerManipulator.convertImplicitToExplicitHydrogens(benzene)
descriptor = new XLogPDescriptor()
println "LogP of oxazone: " +
  ((DoubleResult)descriptor.calculate(oxazone).value)
  .doubleValue()
println "LogP of benzene: " +
  ((DoubleResult)descriptor.calculate(benzene).value)
  .doubleValue()
```
**Output:**
```plain
LogP of oxazone: -0.14800000000000002
LogP of benzene: 2.082
```
